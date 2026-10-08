---
name: find-in-area
description: Find every place of one kind in an area with an LGND Geo classifier, a wall-to-wall map and count where a search gives the best few. Use when the person wants all of a kind of place in an area, or how many there are.
---

# Finding all of one thing in an area

A search finds the best few matches. A classifier labels every place in an area, so it can map and count all of one kind. Ask for the area if the person names none.

## Words used here

- **Target**: the kind of thing the person wants found, such as solar farms.
- **Place**: one image tile of an index. The classifier gives each place one label, for what covers most of it.
- **Site**: one real target, such as one solar farm. It can cover several neighbouring places.
- **Point**: an example given to the build as `{"lat": ..., "lon": ...}`. It stands for the whole place it falls in.
- **Node id**: the id of one image of a place, such as `naip/obs:250109666`, in the `node` column of the tools' results. A place has one per image.
- **Check**: a `reasoning_imagery_check` call. It tests a **claim** (a short description, such as "a football stadium with stands") against each place's image. The build's `check` field is something else: how many of a label's examples fall under their own label.

## Choose the index

- **naip**: the contiguous US, 2020-2024, places about 150 m across (about 40 to a km²), searchable in words. Use it when some targets are under 1 km across.
- **s2**: the world's land, monthly from 2017, places about 1.2 km across. Use it outside the US, when every target fills a place (mines, lakes over about 3 km², forests, burn scars), or for dates after 2024.
- A target much narrower than a place is under-found on either index: a 30 m grass airstrip on naip, a lake under 3 km² on s2. A target under 1 km across outside the US is beyond both. Tell the person.

## Reading the tools

`reasoning_imagery_check`:

- It checks the first six places of a call. The places that pass are in `supported_at`, and `shows` says what each rejected place shows.
- Its verdict depends on the claim's wording as well as the image, most of all on s2's 10 m images. The first time you use a claim, include two controls among the six places:
  - a place you know is the target (before your first check, the top match);
  - one you know is not: a look-alike once you have one (a highway, for runways), otherwise a town centre from `discover_area_view` (outside the area is fine). A town centre is easy to reject, so swap in a look-alike as soon as you have one; a claim that passes nearly every match is too loose.
- Reword the claim if the known-not control passes, or if a target you know fails. Describe what every target shares, with its colour if that is unusual (an algae-green lake), and no more: "on a cleared pad beside a gravel road" passed both controls, then failed real wind turbines.
- The claim is judged one place at a time, so a site's edge places, which show only part of the target, fail it. An edge place beside a site's other places whose `shows` names part of the target (one end of a stadium, a turbine's blade or shadow) belongs to that site, whatever the verdict. A target that covers ground (a lake, a forest) owns a place only where it covers most of it.
- When a verdict looks wrong, check the opposite claim on the same places ("a highway", not "a runway"). If both pass, check a claim about the one feature that tells them apart (the stands, for a stadium).
- Name kinds for what they are from above, whatever `shows` calls them.
- A place said to be "blurry", "too dark" or under "thick clouds" tells you nothing about the claim: drop it. A whole call that comes back `unchecked` means one image failed: check in smaller groups to find it.

`toolbox_classifier_apply`:

- Pass `limit` 200. The response has a grid and a list of places. One grid character covers about three places on naip and one on s2 and shows their majority, so small targets and `?` places often don't show in it: read places from the list.
- The list's rows are shared among the labels, clearest first within each label. A label with more places than its share (about 200 divided by the number of labels) loses its least clear places from the list. In a 6 km naip box, background and look-alikes always do, so their `?` places and their places beside the target go unlisted until step 5's small boxes.
- A place that fits no label shows as `?` in the grid, and in the list under its nearest label with `fit` "poor". `counts` leave out these places (`fits_no_label`) and, on s2, cloudy ones (`cloudy`), though some were the target.
- A marginal call has a margin under 0.05. A `fit` of "close" can still be wrong.
- Read the legend each time: label letters change between builds.

Errors: retry a call that fails with `CLASSIFIER_CONFLICT`, `SERVICE_UNAVAILABLE`, `strabo 500` (even when it comes under `REQUEST_REFUSED`) or a bare "Server returned an error response", after a few minutes if it persists. Builds take one to five minutes, and a conflict can come at the end of one.

## Steps

### 1. Find positives

- Call `search_place_find` in the area with `limit` 50:
  - on naip, with `text` describing the target. A place matches once per image year, while the build uses the newest: once the first matches show the years, pass `start` for the newest year the whole area has (near a state line it can differ).
  - on s2, choose `start` and `end` first (step 4) and give every call the same. Search `like` a checked place of the target that you know or the person names. If neither of you knows one, ask. `discover_area_view` at level `observations` over a box up to 3 km across lists its places' node ids.
- Check matches from each cluster of matches, and from across the ranking, not only the top. Lower matches that fail are hard negatives for step 2.
- The search can miss whole sites. Also search any part of the area the matches leave empty, and around targets you know of, and check a few of those matches.

**Done when** 5 or more places pass the check, from as many of the sites you found as you can.

### 2. Find negatives

Look-alikes (hard negatives), kinds the classifier could take for the target:

- Use the places that step 1's checks rejected, except a site's edge places (whose `shows` names part of the target).
- On naip, search for each look-alike in words that don't also describe the target ("sports fields", not "running track", for stadiums). Check those matches with the target's claim: the rejected places are examples of the kind they show, and the supported ones are more positives.
- On s2, put points on look-alikes you know in the area (tailings and towns, for mines).
- A kind with fewer than 5 places: search for more, or merge it with a similar kind.
- Keep at least one look-alike label. With only the target and background, the target takes the look-alikes, and no `?` shows.

Background (soft negatives), what the rest of the area is made of:

- About a dozen points spread evenly over the area, away from the target's sites. They need no check: the build's `check` field catches most that land on something else.
- A few points on the area's common kinds least like the target, such as forest, fields or water (unless the target's places contain it, as a lake's or a dam's do). On naip a text search finds them.
- Without background, the target takes most of the area, and the build reports nothing wrong.

**Done when** you have a background label and at least one look-alike label, each with 5 or more examples.

### 3. Build the classifier

Call `toolbox_classifier_build` with a label for the target, one for each look-alike kind, and one for background.

- Give every example as a point. A point uses its place's newest clear image, which is the one the apply labels. A `{"text": ...}` example builds but never wins a place.
- The build lists each example's node id. Check again any target example whose node id differs from the one you checked: a place can pass on one image and fail on another, even of the same date.
- A `check` field short of full means one of that label's examples falls under another label or under none (a stray). Find it by applying to that label's examples as `nodes`, then check it:
  - it is what another label says: move it to that label;
  - it is what its own label says: keep it;
  - it is neither: give its kind a label.
- Replace failed or moved examples with other places that passed the check, or drop them while the label keeps 5. A build takes about a minute: make every fix, then build again once.

**Done when** `method` is `lsvm` and every `check` field is full, apart from strays you checked and kept, or after that one rebuild: a stray left then stays, and you say so. A "close" `labels_apart` is fine, whatever the build says. `centroid` means some label has under 5 examples: it reports the same `check` fields but mislabels far more and shows no `?`.

### 4. Apply over the area

- Call `toolbox_classifier_apply` at level `observations`, in boxes under 2,000 places: about 6 km across on naip, 50 km on s2. A refusal names the count.
- Split a box whose target places outgrow the target's share of the list, unless the target covers large areas (lakes, forests): then take its sites' sizes from the grid.
- A county on naip is hundreds of boxes: agree the area with the person first.
- Cell levels read each cell as one square and find only targets that fill it.
- On s2:
  - Give the searches, the build and the applies the same `start` and `end`, about three months with clear images. `discover_area_view` at a cell level with `period` "month" shows each month's cloud. A place has an image a month, and a call over 10,000 images fails.
  - The apply reads each place's newest image in the window, cloudy or not. If many target places come back `c` (cloudy), end the window just after a clear date and build again, then check the examples again: their images change.
  - Bright ground (tailings, salt flats) and water can read as cloud, and the check can pass a cloudy image as the target: check cloudy target places with the opposite claim too, which names the clouds.

**Done when** every box of the area is applied.

### 5. Verify across the distribution

Check places from across the classifier's range, not only the target's clearest:

- the target's marginal calls;
- places labelled as the target inside look-alike areas (in towns, for solar farms);
- `?` places near the target;
- other labels' places beside its sites, which hold its misses.

List them by applying over a small box of under 200 places (about 2 km on naip, 16 km on s2), around sites with marginal calls or `?` first. A `?` may be the target at its edges, a kind common in the area, or a form of the target the claim can't separate from it (a solar farm under construction, a drinking-water plant): ask the person whether it counts.

**Add a label and build again** only when the checks find false sites (target-labelled places away from any real site) of one kind that has no label yet, named as it looks from above (electrical substations, plant nurseries, industrial sites), and they are a quarter or more of the target-labelled sites you checked. In tests a new label for such a kind helped; more examples for a kind that already has a label, or a label for wrong places at a real site's edge (its parking lot, its shoreline, the dirt road beside a turbine), dropped real targets and gained nothing, so report those instead. Give the kind a label of 5 or more checked places (search for more if needed), go back to step 3, and do this once. Afterwards, apply to every box again, check the places that joined or left the target's `counts`, and keep the ones that left that the checks support.

**Done when** each group above is checked at about five sites (all, if fewer), those with marginal calls or `?` first, and you have built again at most once. Stop at about 10 check calls in this step, apart from those for a new label, and say which groups you didn't reach.

## Answer

Answer from the map and the checks you have made. Make no more checks only to sharpen the count: offer them instead (last bullet). Keep it short: the count first, then the sites (with more than about 20, say where they lie and offer the full list), then a few lines on what is uncertain.

- **Sites**: group the target's places into sites, across box edges too. Places a gap of one or two apart can be one target (a solar farm in two blocks), and touching places can be several (a chain of lakes): split or join them using what you know, or say what you couldn't tell. Give each site's position, size in places and one node id.
- **Count**: the number of sites is the number of targets. A small, tall target (a wind turbine, a mast) can spread with its shadow over up to four touching places: count each group of touching places as one, and say the count is low where such targets stand close, since their groups touch. When the person gives a size, measure each site in places and name the sites near the line or cut by the area's edge.
- **Area**, only when the person asks how much ground, or the target covers ground (a lake, a burn scar): the target's `counts` summed over the final classifier's boxes, plus places the checks supported outside them, minus places the checks rejected other than a site's edge places, times a place's area. Call it approximate.
- Say which sites the checks support and which are only labelled, what may be under-found, and what you left for the person to decide.
- **Offer a checked count** when the count is uncertain, with its cost (one check call per six places), and make it only if the person asks. To split groups of touching places, check every place of each group of three or more with a claim naming the part each target has once, at its centre, and its shadow doesn't (a turbine's hub, where its blades join): each place that passes is one target, and two touching places that pass are one.
