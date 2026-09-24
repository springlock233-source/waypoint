# Waypoint 路书

A single-file road-trip planner for comparing multi-stop routes and discussing them with friends.
Open `index.html` in a browser. No build step, no backend. State lives in memory, so a refresh resets it.

Sample data: a 7-day National Day self-drive trip from Changsha through southern Hunan and northern Guangxi (1–7 Oct 2026).

## Views (keys 1–4)

1. **路线树 Route Tree** – a mind-map of possible routes branching from the origin. Click a node to make the path from the root to that node the active plan. A side panel compares every complete route by nights, travel time, km and chosen / TBD attractions.
2. **行程与地图 Itinerary & Map** – the active plan's stops and legs, a schematic SVG map, and a per-stop time budget.
3. **站点景点 Stop Detail** – attraction cards for one stop with photo upload, hours and a go / TBD / skip decision.
4. **总结导出 Summary** – a print-styled itinerary sheet. "导出 PDF" calls the browser print dialog.

## Travel times

Drive and walk legs are routed with the public [OSRM](https://project-osrm.org) server (no API key) and cached per edge. Train, bus, plane and boat legs use a speed × detour estimate. A leg with a hand-entered time keeps it, and the itinerary view shows the routed time beside it with a one-click "改用路线时间" to adopt it. If routing is unavailable the app falls back to a straight-line estimate marked with `*`.

## Data model

```js
state = {
  title, view, startDate,
  rootId, activeLeafId, currentPlaceId,
  places: { [id]: { id, name, lat, lng, days, note,
                    attractions: [{ id, name, desc, hours, decision: 'go'|'tbd'|'skip', img }] } },
  nodes:  { [id]: { id, placeId, parentId, mode, minutes /* null = auto */, children: [ids] } }
}
```

A place can appear in several tree nodes, so attractions and decisions are shared across branches.
