---
name: selecting-index
description: Choose the LGND Geo index, level and time window for a question before searching or exploring: naip or s2, single observations or cells, which dates. Use when planning LGND Geo calls, and again when results come back empty, cloudy or at the wrong scale.
---

# Choosing an index, a level and dates

The server's instructions list each index's coverage, dates, levels and periods. This skill
is about choosing among them.

## Can the index see the thing?

Compare the thing's size with what one place of the index covers:

- **naip**: 1 m pixels; a place is about 150 m across.
- **s2**: 10 m pixels; a place is about 1.2 km across.

- Things under about 10 m don't show on their own on either index. Look for their setting
  instead: a parking lot, not cars; a marina, not boats.
- A thing much narrower than a place is under-found: a 30 m airstrip on naip, a lake under
  about 3 km² on s2.
- Small things (buildings, panels, pads, pens) need naip. A thing under 1 km across outside
  the contiguous US is beyond both indexes: say so rather than search.
- Big things that fill a place (mines, large lakes, forests, burn scars) work on s2, anywhere.

## Does the index cover the place and the time?

- Outside the contiguous US, only s2.
- For dates after naip's newest, only s2.
- naip has about two photos per place. A naip change is the step between them, so it says
  *that* a place changed between two years, not *when*.
- s2 is monthly. Use it to date a change, or to see a change naip's two photos can't
  separate. Seasons change s2 every year: compare the same months, or use a period of a
  year or more.

## Can you search it the way you want?

- naip takes words (`text`). s2 takes none: it needs an example of the thing, a node id
  (`like`) or points for a classifier.
- To get an s2 example, use a place you know is the thing, or ask the person for one, and
  check it with `reasoning_imagery_check` before you search from it.
- A thing found on naip can be followed through time on s2 at the same coordinates with
  `discover_time_zoom`, as long as it is big enough to show at 10 m.

## Which level?

- **Observations** (single images of single places) for finding and checking things.
- **Cells** to survey an area, to rank where change happened, or to follow an area through
  time. A cell is read as one square, so a cell level finds only things that fill it.
- Start coarse over a large area and zoom in where the cells say to look.

## Which dates on s2?

- Check `cloud` before trusting an s2 result; a cloudy month is not a change.
- `discover_area_view` with `period` "month" shows each month's cloud over an area. Choose
  a window of a few clear months, and give every call for the question the same window.
- Bright ground (tailings, salt flats) and water can read as cloud.

## Neither index fits

Say so, and say what would: finer imagery, newer imagery, or another source. Don't answer
from an index that can't see the thing.
