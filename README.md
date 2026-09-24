# Waypoint 路书

A single-file road-trip planner for comparing multi-stop routes and discussing them with friends.
No build step, no backend.

**Live:** https://springlock233-source.github.io/waypoint/ (GitHub Pages, updates on every push to `main`). Or open `index.html` locally.

Sample data: a 7-day National Day self-drive trip from Changsha through southern Hunan and northern Guangxi (1–7 Oct 2026).

## Views (keys 1–4)

1. **路线树 Route Tree** – a mind-map of possible routes branching from the origin. Click a node to make the path from the root to that node the active plan. The side panel compares every complete route by nights, travel time, km, chosen / TBD attractions and budget. Branches can also **merge**: "＋ 分支 → 汇入已有站点" connects a stop to an existing later stop, so everything after it is shared instead of duplicated. Each incoming connection keeps its own travel mode and time, and every distinct start-to-end path still appears in the comparison.
2. **行程与地图 Itinerary & Map** – the active plan's stops and legs, a real map (Leaflet) with the routed roads drawn on it, and a per-stop time and money summary.
3. **站点景点 Stop Detail** – attraction cards for one stop with photo upload, hours, ticket price and a go / TBD / skip decision. Hotel per night and food per day live in the side panel.
4. **总结导出 Summary** – a print-styled itinerary sheet with a budget breakdown and a static map of the route (map tiles stitched on a canvas in the browser, so it prints as one image). "导出 PDF" calls the browser print dialog.

## Saving and sharing

- The trip autosaves on every keystroke to **localStorage** and **IndexedDB**; on load the newest valid copy wins. localStorage matters because Chrome may delete a site's IndexedDB when the disk is nearly full, but leaves localStorage alone. Photos that don't fit in localStorage stay in IndexedDB only. The app also asks Chrome for persistent storage.
- The top bar shows the save state; click it for details and **history versions** (one every 10 minutes, plus one before every import, reset or restore), each restorable.
- If a saved copy can't be read it is set aside as `waypoint.state.unreadable` instead of being overwritten. A second open tab follows the first one's edits, so closing it can't overwrite newer work.
- **导出** downloads the trip as a JSON file; **导入** loads one. That is how you send a plan to a friend and get their edits back. Photos are embedded in the file, so it grows with the number of images.
- **⚙ 设置 → 重置为示例行程** restores the sample.

## Map data

Everything works without a key using **OpenStreetMap** tiles, the public **OSRM** router for driving legs and **Nominatim** for place search. Stored coordinates are always WGS-84.

Paste a **高德 Web服务 Key** in ⚙ 设置 and the app switches to 高德 tiles, 高德 driving and walking routes (with tolls) and 高德 POI search. The key stays in this browser's localStorage and is never written to the exported JSON. Coordinates are converted between WGS-84 and GCJ-02 at the boundary.

Getting a key: [console.amap.com](https://console.amap.com) → 应用管理 → 创建新应用 → 添加 Key → 服务平台 choose **Web服务**. An individual developer currently gets 150,000 basic LBS calls (routing, geocoding) and 5,000 search calls per month for free.

## Travel times and budget

- Driving legs use the routed duration and road distance. Other modes use a speed × detour estimate. A leg with a hand-typed time keeps it and shows the routed time beside it with a one-click "改用路线时间". Legs marked `*` fall back to a straight-line estimate because routing was unavailable.
- Budget per route = driving km × cost/km (set in 设置) + 高德 tolls when routed + hotel × nights + food × days + tickets of chosen attractions.

## Data model

```js
state = {
  title, view, startDate, costPerKm,
  rootId, activePath: [nodeIds], currentPlaceId,
  places: { [id]: { id, name, lat, lng, days, note, hotel, food,
                    attractions: [{ id, name, desc, hours, ticket, decision: 'go'|'tbd'|'skip', img }] } },
  nodes:  { [id]: { id, placeId,
                    parents: { [parentNodeId]: { mode, minutes /* null = routed */ } },   // several = merged branches
                    parentId,        // which parent the node hangs under in the tree layout
                    children: [ids] } }
}
```

The route graph is a DAG: a node may have several parents (merged branches), and the active plan is an explicit path of node ids. Files saved in the older single-parent format (`parentId`/`mode`/`minutes` on the node, `activeLeafId`) are migrated on load. A place can appear in several nodes, so attractions and decisions are shared across branches.
