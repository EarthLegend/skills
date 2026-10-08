# Changelog

One entry per release, written by the publishing script in LGND's source repository.

## lgnd-geo 0.1.4 (2026-10-08)

- Skills: `find-in-area`, `geo-basics`, `selecting-index`
- Source commit: `4e2cf96`
- New `geo-basics`: how to answer with LGND Geo, from deciding what imagery can answer to reporting only what the imagery check confirmed.
- New `selecting-index`: choosing the index (naip or s2), the level and the dates for a question.
- `find-in-area` now leaves the choice of index to `selecting-index`.

## lgnd-geo 0.1.3 (2026-10-08)

- Skills: `find-in-area`
- Renamed the `map-in-area` skill to `find-in-area`: call it as `/lgnd-geo:find-in-area`. Nothing else changed.

## lgnd-geo 0.1.2 (2026-10-08)

- Skills: `map-in-area`
- First public release: map-in-area maps every place of one kind in an area, with an approximate count.
