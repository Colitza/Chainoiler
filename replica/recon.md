# Recon map: OsmAnd (Android)

Scope: offline maps and turn-by-turn navigation, plus motorcycle trip recording
(GPX tracks, trip stats, distance).
For: motorcyclists, starting as a personal tool (tie-in: feed ridden distance to
the Chainoiler).
Date: 2026-10-22

> **How this map was built.** In this session, live page fetches to osmand.net
> and the Play Store failed (DNS blocked in the cloud container), and only web
> search worked. So the screens, flows and settings below come from public
> knowledge of the OsmAnd Android app and its user guide, and nothing was
> captured live. Rows marked *medium* or *guess* should be checked against
> the docs and your own phone before `/replica-architect`. Put your own
> screenshots in `replica/screens/` named by screen ID (`S01.png` ...).

## Sources

| # | source | URL | notes |
| --- | --- | --- | --- |
| 1 | user guide (help center) | https://docs.osmand.net/docs/user | most complete feature and settings list |
| 2 | first steps | https://docs.osmand.net/docs/user/start-with/first-steps | onboarding, first map download, links to nav and trip recording |
| 3 | map downloads | https://docs.osmand.net/docs/user/personal/maps-resources | regions, update, delete (path to verify) |
| 4 | navigation | https://docs.osmand.net/docs/user/navigation | route planning, guidance, voice |
| 5 | trip recording plugin | https://docs.osmand.net/docs/user/plugins/trip-recording | GPX recording, intervals, notification |
| 6 | purchases (Android) | https://docs.osmand.net/docs/user/purchases/android | what Maps+ and Pro gate |
| 7 | Play Store listing | https://play.google.com/store/apps/details?id=net.osmand | screenshots, pitch, ratings |
| 8 | Wikipedia | https://en.wikipedia.org/wiki/OsmAnd | history, licence (GPLv3 app, ODbL data) |
| 9 | your own phone | — | install the free app and drive every state; screenshots go in `replica/screens/` |

Clean-room note: OsmAnd's app source is public (GPLv3). Do **not** read it for
this clone. Work from the docs and the app's behaviour only. OpenStreetMap
data is ODbL and can be used directly, with attribution.

## Core loop

Download your region once, then search a place, get a motorcycle route and
ride it with turn-by-turn voice guidance with no signal, while the trip is
recorded to a track with distance and stats.

## Screens

| ID | screen | route / how to reach | purpose | key components | states seen |
| --- | --- | --- | --- | --- | --- |
| S01 | Onboarding / first map download | first launch | grant location, detect region, download first map | region card, size label, progress bar, skip | no location permission, offline, downloading, done |
| S02 | Main map | app home | the map with your position and the main controls | map canvas, my-location button, zoom +/-, compass, menu button, search button, directions button, top widgets | no map downloaded (world overview only), GPS searching, following, free pan, night mode |
| S03 | Main menu (drawer) | menu button on S02 | entry to everything else | list rows with icons, profile switcher | — |
| S04 | Maps & resources | S03 → Maps & resources | download, update and delete regions | region tree (continent → country → region), size, update badge, download button, storage bar | not downloaded, downloading, downloaded, update available, low storage, free download limit reached |
| S05 | Search | search button on S02 | find addresses, places, categories, coordinates | search field, category grid, history list, result list with distance | empty (history), typing, no results, region not downloaded |
| S06 | Point context menu | tap a point / pick a search result | act on a place | bottom sheet: name, address, distance; Directions, Save, Share, More | collapsed, expanded, saved |
| S07 | Route preparation | Directions on S02 or S06 | set start/end/waypoints and the profile, see the route | from/to fields, add-waypoint, profile chips (car, motorcycle, bike, foot), route params button, route line on map, distance/time summary, Start | calculating, calculated, no route found, missing map along the route |
| S08 | Route parameters | gear on S07 | routing options for the profile | toggles: avoid motorways, avoid tolls, avoid unpaved roads, avoid ferries; prefer/avoid options; voice on/off | — |
| S09 | Route details | swipe up on S07 summary | turn list and elevation | turn-by-turn list, elevation graph, road surface breakdown | — |
| S10 | Navigation (in progress) | Start on S07 | guide the ride | next-turn widget with distance, lane guidance, street name, speed, speed limit, ETA / distance / time remaining, mute, stop | on route, off route (recalculating), arrived, GPS lost, screen-off with voice |
| S11 | Trip recording controls | REC widget on S02 or the notification | start, pause, stop and save the trip | REC widget showing distance/time, sheet: start/pause/save/discard, foreground notification | idle, recording, paused, saving |
| S12 | My places: tracks | S03 → My places → Tracks | list recorded and imported tracks | folders, track rows (name, date, distance, duration), import GPX | empty, filled |
| S13 | Track details | tap a track on S12 | review a ride | track on map, stats (distance, duration, moving time, avg/max speed, elevation up/down), speed and elevation graphs, rename, share GPX, delete | — |
| S14 | My places: favourites | S03 → My places → Favourites | saved places in groups | group list, colour/icon per group, rows with distance | empty, filled |
| S15 | Configure profile | S03 → Settings → profile | per-profile settings | navigation type (motorcycle), voice language, units, screen behaviour during nav, map orientation | — |
| S16 | Configure map | layers button / S03 | map style and layers | style picker (e.g. touring), layers: favourites, tracks, POI; day/night | — |
| S17 | Trip recording settings | S03 → Plugins → Trip recording | how GPS is logged | logging interval, min displacement, min accuracy, auto-split on gap, save automatically | — |
| S18 | Plan a route | S03 → Plan a route | draw a route by tapping, save as GPX, navigate it later | point list, snap-to-road toggle, distance, undo/redo, save | empty, editing |
| S19 | Purchases / upgrade | download limit, menu | Maps+ / Pro paywall | plan cards, price, restore | — skip in clone |

## Flows

```
F01 Install and get a map onto the phone
    S01 -> S04 (region) -> S02
    happy path clicks: 3 (allow location, download, done)
    edge: no location permission, no Wi-Fi (size warning), low storage, download interrupted

F02 Find a place and navigate to it on the motorcycle
    S02 -> S05 -> S06 -> S07 (motorcycle profile) -> S10
    happy path clicks: 5 (search, type, pick, Directions, Start)
    edge: region missing along the route, no route found, start GPS without fix,
          off-route recalculation offline, arrival, app backgrounded / screen off

F03 Plan a route with waypoints and options
    S02 -> S07 -> add waypoint -> S08 -> S09 -> Start (S10)
    happy path clicks: 6
    edge: reorder waypoints, avoid-unpaved makes the route impossible, very long route

F04 Record a ride
    S02 -> S11 start -> ride -> S11 stop/save -> S13
    happy path clicks: 3 (REC, start, save)
    edge: GPS gap in a tunnel (auto-split), app killed by battery saver,
          recording while navigating (F02 running), phone reboot mid-ride

F05 Review and share a past ride
    S03 -> S12 -> S13 -> share GPX
    happy path clicks: 4
    edge: very long track (graph performance), track with no elevation

F06 Save and reuse a favourite place
    S06 Save -> S14 -> pick -> S07
    happy path clicks: 3
    edge: duplicate name, group management

F07 Update or delete downloaded maps
    S03 -> S04 -> update / delete
    happy path clicks: 3
    edge: map in use by an active route, partial update failure

F08 Set up the motorcycle profile
    S03 -> S15 -> navigation type, voice, units -> S08 defaults
    happy path clicks: ~5
```

## Components

| component | variants | states | used on |
| --- | --- | --- | --- |
| Map canvas | vector, night, touring style | loading tiles, no data, following, free pan, rotated | S02, S06, S07, S10, S13, S18 |
| Floating map button | my location, zoom in/out, compass, layers, directions, search, menu | default, pressed, active (following), disabled | S02, S10 |
| Map widget (info) | speed, distance-to-go, time-to-go, ETA, altitude, trip distance, REC | default, highlighted, tap to cycle | S02, S10, S11 |
| Next-turn panel | large, small (second turn), lanes | normal, imminent, off route | S10 |
| Bottom sheet | collapsed, half, full | dragging | S06, S07, S09, S11 |
| Search field + result row | address, POI, coordinate, history | typing, empty, no results | S05 |
| Category grid tile | — | default, pressed | S05 |
| Region row | continent, country, region | not downloaded, downloading (progress), downloaded, update available | S04, S01 |
| Progress bar | determinate, indeterminate | — | S01, S04, S07 |
| Profile chip | car, motorcycle, bicycle, pedestrian | selected, unselected | S07, S03 |
| Toggle row | — | on, off, disabled | S08, S15, S16, S17 |
| Stat block | distance, time, speed, elevation | — | S11, S13 |
| Line graph | elevation, speed | empty, filled, scrubbing | S09, S13 |
| List row with icon + distance | favourite, track, POI | — | S05, S12, S14 |
| Foreground notification | recording, navigating | running, paused | S10, S11 |
| Dialog | confirm, warning (size/Wi-Fi), error | — | everywhere |
| Snackbar / toast | info, undo | — | everywhere |

## Inferred data model

```
MapRegion     id, name, parent_id, kind (continent|country|region), file_size,
              version_date, bbox
              evidence: S04 region tree, sizes, update badges
              confidence: high

DownloadedMap region_id, local_path, version_date, size_on_disk, status
              (downloading|ready|outdated|failed)
              evidence: S04 states, F07
              confidence: high

Place         (read-only, from map data) id, name, category, lat, lon, address
              evidence: S05 results, S06 context menu
              confidence: high

Favourite     id, name, lat, lon, group_id, description, created_at
FavouriteGroup id, name, color, icon, visible
              evidence: S14, S06 Save
              confidence: high

Profile       id, name, base (car|motorcycle|bicycle|pedestrian), routing_type,
              routing_params (json: avoid_motorways, avoid_tolls, avoid_unpaved,
              avoid_ferries, ...), voice_language, units, map_style, color, icon
              evidence: S07 chips, S08, S15
              confidence: high (params list: medium)

Route         id, profile_id, start, end, waypoints[] (ordered lat/lon/name),
              distance_m, duration_s, geometry, instructions[]
              evidence: S07, S09, F03
              confidence: high (persisted only as GPX/plan: medium)

Instruction   index, type (turn left|right|roundabout exit n|...), street_name,
              distance_m, lanes
              evidence: S09 list, S10 next-turn panel
              confidence: high

Track         id, name, folder, created_at, source (recorded|imported|planned),
              file_path (GPX), stats (json: distance_m, duration_s, moving_s,
              avg_speed, max_speed, ele_up, ele_down)
TrackSegment  track_id, index
TrackPoint    segment_id, lat, lon, ele, time, speed, hdop
              evidence: S12, S13, S17 (auto-split = segments), GPX export
              confidence: high

RecordingSettings  interval_s, min_displacement_m, min_accuracy_m,
              auto_split_gap_s, auto_save
              evidence: S17
              confidence: medium (exact option names to verify)

TripOdometer  (clone addition) total_distance_m, since_last_oil_m
              evidence: none in OsmAnd; for the Chainoiler tie-in
              confidence: guess
```

Relationships: MapRegion 1-n MapRegion (tree), MapRegion 1-0..1 DownloadedMap,
FavouriteGroup 1-n Favourite, Profile 1-n Route, Route 1-n Instruction,
Track 1-n TrackSegment 1-n TrackPoint.

Everything lives on the device. No accounts in the core loop (OsmAnd's cloud
sync is a paid add-on and out of scope).

## Feature matrix

See `features.csv`. Must: 21, should: 12, could: 8, skip: 7.

## Out of scope (cannot or should not be cloned)

- **OsmAnd's prepared map files (`.obf`) and their download servers.** These
  belong to them. The clone builds its own offline data from OpenStreetMap
  extracts (ODbL, attribution required).
- **OsmAnd's code, voice prompts, icons, map styles, name and logo.** Make your
  own or use openly licensed ones.
- **OsmAnd Cloud / backup sync.** It's their service, and a personal tool doesn't
  need it.
- **Wikipedia and Wikivoyage offline content.** That's a different licence and
  outside the slice.
- **Weather, live traffic and hourly map updates (Pro).** These need data feeds,
  so they're out of the slice.
- **Contour lines and hillshade (Maps+).** They could be built later from
  public SRTM data, but they're not in this slice.
- **The Play Store paywall (S19).** It's a personal tool, so there's nothing
  to sell.

## Size

Screens 19 (18 to build), flows 8, entities 11.

Hard parts:
1. **Offline vector map data and rendering.** You need a pipeline from OSM
   extracts to per-region files on the phone, plus a renderer. Use existing
   open engines, such as MapLibre Native with offline vector tiles, rather
   than writing your own.
2. **Offline routing with a motorcycle profile.** You need an on-device
   engine (open options include GraphHopper and BRouter), routing graphs
   built per region, a custom motorcycle cost profile, and fast offline
   recalculation when you go off route.
3. **Turn-by-turn guidance that survives the background.** This covers
   voice prompts (Android TTS), a foreground service, screen-off guidance,
   and recording a track during navigation without Android's battery
   savers killing it.

Size: **L** if it's built on open engines, **XL** if the map and routing
engines are written from scratch, which would need a rescope. Suggested
phases:
- Phase 1: trip recording, the track list and the Chainoiler odometer.
  This is an **S–M** job and works on a plain online map.
- Phase 2: offline map downloads and search.
- Phase 3: motorcycle routing and navigation.
