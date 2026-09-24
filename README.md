# Waypoint 路书

A single-file road-trip planner for comparing multi-stop routes and discussing them with friends.
Open `index.html` in a browser, or host it on GitHub Pages. No build step, no backend.

Sample data: a 7-day National Day self-drive trip from Changsha through southern Hunan and northern Guangxi (1–7 Oct 2026).

## Views (keys 1–4)

1. **路线树 Route Tree** – a mind-map of possible routes branching from the origin. Click a node to make the path from the root to that node the active plan. The side panel compares every complete route by nights, travel time, km, chosen / TBD attractions and budget.
2. **行程与地图 Itinerary & Map** – the active plan's stops and legs, a real map (Leaflet) with the routed roads drawn on it, and a per-stop time and money summary.
3. **站点景点 Stop Detail** – attraction cards for one stop with photo upload, hours, ticket price and a go / TBD / skip decision. Hotel per night and food per day live in the side panel.
4. **总结导出 Summary** – a print-styled itinerary sheet with a budget breakdown. "导出 PDF" calls the browser print dialog.

## Saving and sharing

- The trip autosaves in the browser (IndexedDB), so a refresh keeps it.
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
  rootId, activeLeafId, currentPlaceId,
  places: { [id]: { id, name, lat, lng, days, note, hotel, food,
                    attractions: [{ id, name, desc, hours, ticket, decision: 'go'|'tbd'|'skip', img }] } },
  nodes:  { [id]: { id, placeId, parentId, mode, minutes /* null = routed */, children: [ids] } }
}
```

A place can appear in several tree nodes, so attractions and decisions are shared across branches.
