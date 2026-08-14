# LiF Hub — frontend prototype

A frontend-only build of the LiF Interactive Engagement Hub: an events map, a master calendar,
directories for People / Groups / Organizations / Opportunities, a personal dashboard, and a
"More Features" panel so each member can build their own minimal-to-full view of the hub.

Everything that needs a real server (register for an event, sign in, connect with someone, join a
group, edit a profile) is wired to a single obvious placeholder for now. Everything that doesn't
need a server (map, filters, calendar, adding an event to your own calendar, sharing an event
link) is fully working today.

---

## 1. File structure

Paste these straight into a VS Code project with this layout:

```
lif-hub/
├── index.html              ← the whole app shell — open this
├── package.json            ← optional, just gives you `npm start`
├── README.md
├── css/
│   ├── theme.css            ← colors, fonts, spacing (design tokens only)
│   └── main.css             ← layout & every component's styling
└── js/
    ├── data.js               ← ALL sample content lives here (see §4) — this is what you swap for real API calls
    ├── state.js               ← one shared object: current view, active filters, feature toggles
    ├── utils.js                ← DOM helpers, date formatting, distance math, the backend-placeholder pattern
    ├── map.js                  ← Leaflet map + event pins
    ├── filters.js               ← the aspect wheel + full filter drawer + the filtering logic itself
    ├── eventDetail.js            ← the event detail modal
    ├── dashboard.js                ← personal dashboard sidebar
    ├── directory.js                 ← People / Groups / Organizations / Opportunities grids
    ├── calendarView.js               ← FullCalendar wrapper
    ├── customize.js                   ← the "More Features" panel
    └── app.js                          ← wires it all together — loaded last
```

No `src/`, no build output folder, no `node_modules` to commit — there's nothing to compile.

## 2. Running it

**Easiest:** double-click `index.html`. This project uses plain `<script>` tags (not ES modules),
so unlike a lot of modern frontend boilerplate it actually works straight off disk.

**Recommended anyway**, since it's closer to how it'll behave once it's live: run it through a
local server —

- VS Code: install the "Live Server" extension, right-click `index.html` → *Open with Live Server*, or
- Terminal: `npm start` (runs `serve` via `npx`, no install needed), or
- `python3 -m http.server 5500` and visit `http://localhost:5500`

One reason to prefer a server: "Use my current location" (in the distance filter) uses
`navigator.geolocation`, which some browsers restrict on plain `file://` pages. Everything else
works identically either way.

## 3. Key decisions, and why

**Vanilla JS, no framework, no build step.** Every file attaches to a single global, `window.LIF`,
so there's nothing to `npm install` and no bundler config to fight with mid-sprint. Trade-off: you
write a bit more DOM-wiring code by hand than you would in React — but there's also nothing to
break between your machine and whatever HostPapa serves it as.

**Leaflet + OpenStreetMap, not Google Maps.** The baseline code had a live Google Maps API key
embedded in it, but it's domain-restricted to the old site and isn't yours to carry over, and a
new key means billing setup. Leaflet needs no key and is free at any traffic level; OpenStreetMap's
public tiles are fine for development and normal usage, but if the hub gets heavy traffic later,
OSM's usage policy asks high-volume sites to use a paid tile provider (Mapbox, MapTiler, Stadia —
all drop-in swaps, isolated entirely to `map.js`).

**FullCalendar, pinned to v6.1.21 exactly.** The meeting transcript's action item called for
fullcalendar.io specifically. It's loaded from a CDN rather than vendored, pinned to an exact
version rather than "latest": v7.0.0 shipped on 2026-06-19 — literally the day after 6.1.21 — and
is a genuinely different architecture underneath (Preact-based, a multi-file theme system). npm's
"latest" tag now points to 7.0.2, so pulling in FullCalendar without a pin would silently hand you
the untested new major version. 6.1.21 is the last release of the older, better-documented line,
which felt like the safer choice for a team building fast without time to debug an unfamiliar
major version mid-sprint. Worth revisiting once v7 has more real-world track record.

**Chakra palette — top row only.** Every color in `css/theme.css` is the first-row hex you sent;
none of the "-2" rows were used. If you finalize those later, that file is the only place they
need to change — every component references the named variables, never raw hex.

**Aspects mapped to chakras.** The framework doc's 7 Aspects and the palette's 7 chakras are
paired in `js/data.js` (`LIF.ASPECTS`) in the order both were listed. That pairing is a judgment
call, not a spec — if your team had a different aspect-to-chakra intention, it's one array to edit
and everything (pins, badges, the aspect wheel, the calendar) updates automatically.

**Sectors use your real taxonomy.** Rather than re-deriving shorter labels from the framework doc,
`LIF.SECTORS` uses the actual 12-category tree from your production `copy_map` data
(Spirituality & Religion, Science & Technology, etc.), so this stays consistent with what's
already live.

**Map pins are events, not people**, per your note — clicking one opens the event's full detail,
never a person. Online-only events don't get a pin (a location doesn't mean much for them); they
show as a card strip under the map instead, and picking the "Online" quick filter switches
straight to the list view automatically, following the split your team landed on
Aug 13 (in-person/hybrid → map, virtual → cards).

**Minimalist by default.** On first load, only the events map/list and basic search show up.
"More Features" (top-right) reveals switches for the personal dashboard, calendar, and the
People/Groups/Organizations/Opportunities tabs, plus an "advanced filters" switch that reveals
the full filter drawer (sectors, date, time, cost, language, and the rest of the framework doc's
filter list) — the aspect wheel, search, format, and map/list toggle stay on always, since those
felt core rather than optional. This mirrors the toggle-based approach (rather than full
drag-and-drop) your team landed on for the dashboard in the same meeting.

**One placeholder pattern, used everywhere.** Every action needing a real backend — Register,
Sign In, Connect, Join Group, Express Interest, Edit Profile — calls
`LIF.util.backendPlaceholder()` in `js/utils.js`, which shows a toast explaining what would happen
and opens google.com. When your backend exists, that's the one function to change; nothing else
in the app needs to know.

**A few things are real, not placeholders**, since they don't need a server at all: downloading an
`.ics` file, an "Add to Google Calendar" link, a `mailto:` invite-a-friend link, and copying a
shareable link (`#event=<id>`, so a pasted link opens straight to that event).

**Organization-gated events.** Per the framework doc's Ashoka example, one sample event
(`evt-009`, "Ashoka Fellows Quarterly Convening") is marked `visibility: "organization"`. Since
there's no real sign-in yet, it always renders in its "signed-out" state — a members-only badge
and a locked pin, with the fuller details held back — so you can see what that gating looks like
before auth exists. A second event (`evt-005`) is hosted *with* Ashoka but kept public, matching
the doc's "public jointly created events" case.

**On the live map.co-creators.website reference:** this environment can't browse live URLs, so
this was built from the baseline code you sent (`copy_map.zip`) and the meeting transcript rather
than the live page directly. Worth noting either way: you mentioned on the call that the live
version currently has no real CSS applied, so there wasn't much visual behavior to match yet.

## 4. The test event — and the sample data around it

`js/data.js` → `LIF.EVENTS[0]` (`evt-001`, flagged `isTestEvent: true`) is the requested test
event: **"Global Co-Creation Circle,"** a hybrid networking gathering in Sedona, AZ, with a full
set of fields filled in (description, capacity, registration count, tags, aspect, sector, cost,
language, online link). Open it in the app to see every field the detail modal supports.

Eight more sample events sit alongside it — enough spread across aspects, sectors, formats, and
dates that the filters, map, and calendar all have something real to demonstrate. All of it lives
in one array; swap it for a real API response whenever the backend's ready and nothing else in the
app needs to change, since every screen reads from `LIF.EVENTS`, `LIF.PEOPLE`, `LIF.GROUPS`,
`LIF.ORGANIZATIONS`, and `LIF.OPPORTUNITIES` rather than hardcoding anything itself.

## 5. Known simplifications (fine for a first pass, worth knowing about)

- **The personal dashboard uses a hardcoded demo member** (`LIF.CURRENT_MEMBER` in `data.js`),
  clearly labeled "sample profile" in the UI. There's no real login yet — swap this object for the
  signed-in member's real data once accounts exist.
- **Filters live on the Events tab.** The calendar view reads the same filtered results, but the
  filter controls themselves only appear while you're on Events — switch there to change what's
  filtered, then flip back to Calendar.
- **No drag-and-drop dashboard layout** — just on/off switches for each widget, matching what your
  team actually decided on the call rather than the fuller drag-and-drop version that was floated
  and set aside as too time-consuming for this sprint.

## 6. Libraries used (all via CDN, no install needed)

| Library | Version | Why |
|---|---|---|
| [Leaflet](https://leafletjs.com/) | 1.9.4 | The map. No API key required. |
| [FullCalendar](https://fullcalendar.io/) | 6.1.21 (pinned) | The master calendar, per the meeting's action item. |
| [Fraunces](https://fonts.google.com/specimen/Fraunces) / [Karla](https://fonts.google.com/specimen/Karla) / [IBM Plex Mono](https://fonts.google.com/specimen/IBM+Plex+Mono) | — | Google Fonts, loaded in `index.html`. |

Map tiles are © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors.
