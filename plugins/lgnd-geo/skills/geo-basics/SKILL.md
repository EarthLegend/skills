---
name: geo-basics
description: Read before calling any LGND Geo tool (search_place_find, search_change_find, search_similar_change_find, discover_*, toolbox_classifier_*, reasoning_imagery_check). How to choose between published data, live sources and imagery, and how to report only what the imagery check confirmed.
---

# Answering with LGND Geo

Geo's tools find, compare and label places from embeddings of imagery. They are good at
*where* and *what it looks like*. They are not a register of facts, and they are not live. The
best answers pair them with what you can find elsewhere.

## 1. Decide what imagery can and can't answer

- **Counts, inventories and official figures** ("how many power plants in Ohio", "the
  largest mines in Chile"): find an authoritative list first: USGS, EIA, USDA, OpenStreetMap,
  Copernicus EMS, a government register. Then use Geo to locate the listed sites, check them
  against imagery, and find ones the list misses. Say which number came from where.
- **Live conditions** (ship positions, river gauges, alerts, throughput, prices): use live
  sources. Imagery can confirm or locate, but its newest date is weeks or years old.
- **Where, what, and whether it changed**: this is what Geo is for. Start there.

## 2. Report what was checked, and only that

- **Say the newest date** behind every finding. "As of the 2023 NAIP photo" is an answer;
  "currently" is not.
- **Lead with coordinates and node ids** for every finding (`naip/obs:123`, 33.45, -112.07),
  so the person can look for themselves.
- Run `reasoning_imagery_check` on each claim before you state it. Report as **confirmed**
  only the places in its `supported_at`. Say how many places were checked and how many were
  rejected, and what the rejected ones showed instead. It checks the first six places it is
  given; places you didn't check are *candidates*, and say so.
- A search result is a lead, not a finding. Search returns the best matches, never "all of
  them": don't count search results as a total. To find or count every place of one kind in
  an area, use the `find-in-area` skill.

## 3. Read the numbers for what they are

- **`similarity` is closeness in embedding space**; use it only to rank.
- **Areas from classifier counts are estimates.** Call them approximate, and note the close
  calls (lowercase letters) and the targets that fit no label (`?`).
- A classifier is only as good as its examples. Report how distinct its labels came out.

## 4. Keep a soft call budget

Plan the calls before the first one (the server's own instructions say how). Then stop and
answer once more calls aren't changing the result. When the last few calls haven't moved
the answer, answer with what you have and say what's uncertain.
