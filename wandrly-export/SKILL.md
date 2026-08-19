---
name: wandrly-export
description: >
  ALWAYS use this skill when the user asks to plan a trip, itinerary, or travel schedule —
  even if they don't mention wandrly or export. Also use when the user provides a .wandrly
  file and wants to add planning on top of existing bookings. At the end of every trip
  planning response, append one line offering to deliver it — adding it straight to the
  user's Wandrly account when the Wandrly MCP tools are available, otherwise a .wandrly
  file. Use this skill also when the user explicitly asks to export, save, download, or
  import a trip.
---

# Wandrly Export Skill

> **TEMPORARY — measuring what enrichment costs.** Before you start writing,
> ask the user ONE short question and wait for the answer:
>
> *"לחפש את המקומות בגוגל קודם (איטי יותר — דקות של חיפושים לפני שאני מתחיל לכתוב),
> או לשלוח רק שמות ולתת לוונדרלי לפתור אחרי הייבוא (מהיר)?"*
>
> If they choose to look places up, ignore the rest of this section for that run
> and enrich as described at the bottom.
>
> **Report the time either way**, so the two runs can be compared:
> - **With MCP** — call `get_planning_rules` ONCE before you start writing, purely to
>   get its `START_TOKEN` (you already have the format from this skill, so ignore the
>   rest of what it returns). Pass the token verbatim as `started_at` to `import_plan`
>   / `merge_plan`; the response comes back with `generation_time` measured off the
>   server clock. Quote that number.
> - **Writing a file** — there is no server clock to read, so say plainly that the
>   time is your own estimate rather than a measurement.
>
> Remove this block once the comparison is done.

---

⚠️ **CRITICAL**: All events (flights, accommodation, transport, places) **must** go inside a single `"events"` array. Never output separate `"hotels"`, `"flights"`, `"commutes"`, or `"places"` keys.

Turns trip data discussed in the conversation into a valid `.wandrly` plan, then
delivers it — imported straight into the user's account when the Wandrly MCP
tools are available, or written as a file when they aren't. See **Delivery** at
the end; the file is the fallback, not the goal.

---

## Mode A — Planning from scratch

Use this mode when the user has no existing bookings and wants to plan a new trip.

### Before planning — gather missing info

**Before starting to plan any trip**, check if the following required details are present in the conversation. If any are missing, ask for them all in a single message before proceeding:

- **Destination** — where are they going? (city/country)
- **Origin** — where are they flying from? (city or airport code)
- **Travel dates** — when is the trip? (start and end dates or duration)
- **Number of travelers** — how many people?
- **Traveler ages / type** — adults, kids (ages?), seniors? (important for activity and pace suggestions)
- **Budget** — approximate total budget or per-person budget (and currency)?
- **Trip style** — e.g. relaxed sightseeing, adventure, food-focused, family with young kids, romantic, etc.

Ask only for the fields that are genuinely missing. If enough context is provided to make reasonable assumptions, state the assumptions and proceed. **Do not ask for info already provided.**

### Planning principles

- **Route optimization is mandatory** — always plan the most logical geographic flow. Never jump back and forth between distant areas on the same day or across consecutive days.
- **Long drives** — any drive over ~2 hours should include a suggested stop (scenic spot, town, attraction) along the route. Mention it explicitly in the day plan.
- **Pacing** — match the daily intensity to the traveler profile (family with kids, seniors, active travelers, etc.).

### Planning flow — MANDATORY for all inputs including pre-made itineraries

**Even if the user provides a ready-made itinerary or detailed travel plan**, the planning flow below is mandatory. Do NOT skip steps. A pre-made plan is treated as raw material, not as a confirmed schedule.

#### Step 1 — Day-by-day summary (always required)

Present a concise day-by-day summary of the proposed itinerary. For each day list: date, main destinations, key activities, accommodation. Do not include specific times yet.

Ask the user to confirm, adjust, or comment on the day structure before moving on.
- In Hebrew: *"האם המבנה הזה נראה נכון? יש ימים שתרצי לשנות?"*
- In English: *"Does this structure look right? Any days you'd like to adjust?"*

**Wait for explicit confirmation before proceeding to Step 2.**

#### Step 2 — Hourly schedule (required after Step 1 confirmed)

Only after the user confirms the day structure: present a full hourly schedule with specific times per activity for every day. Include travel times between locations, meal breaks, and realistic pacing for the traveler profile.

Ask the user to confirm the hourly schedule.
- In Hebrew: *"האם לוח הזמנים הזה נראה טוב? רוצה לשנות משהו?"*
- In English: *"Does this schedule work for you? Anything to adjust?"*

**Wait for explicit confirmation before proceeding to export.**

#### Step 3 — Delivery trigger (required after Step 2 confirmed)

Only after the hourly schedule is confirmed: ask how to deliver it. **Offer what
you can actually do** — if the Wandrly MCP tools are available, offer the import,
not a file. Promising a file when you could have written straight to the account
is the failure mode this step exists to prevent.

With MCP tools available:
- In Hebrew: *"רוצה שאוסיף את הטיול לחשבון הוונדרלי שלך?"*
- In English: *"Want me to add this trip to your Wandrly account?"*

Without them:
- In Hebrew: *"רוצה שאייצא את המסלול לקובץ wandrly?"*
- In English: *"Want me to export this itinerary as a .wandrly file?"*

Wait for confirmation ("כן", "yes", "בטח"), then follow **Delivery** below.

**Do NOT skip to delivery directly, even if the user provided a complete ready-made plan with times.**

---

## Mode B — Enrich an existing .wandrly file

Use this mode when the user shares a `.wandrly` file (by uploading it or pasting its JSON) and asks to add planning content on top of existing bookings.

### Step B-1 — Parse and display the skeleton

Read the provided `.wandrly` JSON. Identify all **anchors** — events that have a `reservation` or `confirmation_code` field in their `data`. These are real confirmed bookings and must never be moved or modified.

Display a compact timeline to the user showing what is already booked:

```
📅 Trip skeleton from your .wandrly file:

✈️ [date] Flight [from_airport]→[to_airport] [departure time] — CONFIRMED (PNR: XXX)
🏨 [checkin date–checkout date] Hotel in [city] — CONFIRMED
🚗 [pickup date] Car rental pickup [address] — CONFIRMED
...
🕳️ [date range] — nothing planned yet
```

Mark each confirmed item as **[CONFIRMED — cannot be moved]**. List any gaps (days or half-days with no content) explicitly.

Then ask: *"What would you like to add? (places to visit, hotel placeholder for the gap, intermediate car rental, etc.)"*

Wait for the user's planning prompt before proceeding.

### Step B-2 — Fill the gaps (day-by-day summary)

Based on what the user asked for, propose what to add **only in the gaps**. Never suggest moving or replacing confirmed items. For each gap, propose:
- Places to visit (with approximate duration)
- Hotel placeholder if overnight is needed and none exists
- Car rental or transport if logical

Present as a day-by-day summary of **new additions only**, anchored against the confirmed items. Ask the user to confirm the structure.

### Step B-3 — Hourly schedule for new items only

After confirming the structure: present a full hourly schedule for the newly added items. Confirmed bookings are shown as fixed reference points. Ask the user to confirm before exporting.

### Step B-4 — Deliver (merge mode)

Build the complete plan containing:

1. **All original confirmed events** — copied verbatim from the input file. Their `data` fields including `reservation` and `confirmation_code` are preserved exactly. In their `timeline_slots`, set `"is_pinned": true`.
2. **All new placeholder events** — generated by you. No `reservation` or `confirmation_code`. Places use `"visit_time": null` and `"is_pinned": false`.

Set the output `source` to:
```json
{ "generator": "ai", "tool": "claude", "model": "claude-sonnet-4-6", "ingestion": "file", "method": "skill" }
```

Then deliver it per **Delivery** below. Merging is the whole point of Mode B, so:

- **With MCP tools** — use `merge_plan` with the target trip id. It skips anything
  whose `confirmation_code` is already in the DB, so confirmed bookings are never
  duplicated or overwritten. Note that merge mode is exempt from the trip-overlap
  check, so a `trip_overlap` error here means you used `import_plan` by mistake.
- **Without MCP tools** — write the file and tell the user: *"Import with the Merge
  button in the app — it will add the new places/hotels without touching your
  existing bookings."*

---

## Output format

The file is a single JSON object. Follow this example exactly — field names, nesting, and structure are authoritative. Leave unknown fields as `null`, never invent data.

```json
{
  "wandrly_version": 1,
  "source": { "generator": "ai", "tool": "claude", "model": "claude-sonnet-4-6", "ingestion": "file", "method": "skill" },
  "exported_at": "YYYY-MM-DDTHH:MM:SS",
  "trip": {
    "name": "Trip name",
    "destination": "City / Region",
    "start_date": "YYYY-MM-DD",
    "end_date": "YYYY-MM-DD",
    "status": "draft",
    "notes": null,
    "ai_tips": [
      "Practical general tip about the destination (transport, culture, money, weather, etc.)",
      "Another general tip relevant to this specific trip"
    ]
  },
  "passengers": [],
  "events": [
    {
      "record_type": "flight",
      "date": "YYYY-MM-DD",
      "sort_order": 1,
      "data": {
        "airline": null, "flight_number": null,
        "from_city": "Origin city", "from_airport": "AAA",
        "departure": "YYYY-MM-DDTHH:MM:SS",
        "to_city": "Destination city", "to_airport": "BBB",
        "arrival": "YYYY-MM-DDTHH:MM:SS",
        "cost": null, "currency": null,
        "class_of_service": null, "bag_carry_on_kg": null, "bag_checked_kg": null, "notes": null
      }
    },
    {
      "record_type": "accommodation",
      "timeline_slots": [
        { "slot": "start", "date": "YYYY-MM-DD", "time": "15:00", "sort_order": 3, "is_pinned": false, "duration_minutes": null },
        { "slot": "end",   "date": "YYYY-MM-DD", "time": "11:00", "sort_order": 1, "is_pinned": false, "duration_minutes": null }
      ],
      "data": {
        "hotel_name": "Hotel in [city/area]", "hotel_address": null,
        "destination": "City", "provider": null,
        "total_price": null, "currency": null,
        "room_type": null, "is_non_refundable": null, "notes": null
      }
    },
    {
      "record_type": "transport",
      "timeline_slots": [
        { "slot": "start", "date": "YYYY-MM-DD", "time": "HH:MM", "sort_order": 2, "is_pinned": false, "duration_minutes": null, "address": "Pickup address", "lat": null, "lng": null },
        { "slot": "end",   "date": "YYYY-MM-DD", "time": "HH:MM", "sort_order": 2, "is_pinned": false, "duration_minutes": null, "address": "Dropoff address", "lat": null, "lng": null }
      ],
      "data": {
        "category": "car_rental", "company": null,
        "vehicle_type": null, "cost": null, "currency": null, "notes": null
      }
    },
    {
      "record_type": "place",
      "date": "YYYY-MM-DD",
      "sort_order": 4,
      "data": {
        "name": "Place name", "type": "attraction",
        "area": null,
        "duration_minutes": 60, "visit_time": null,
        "priority": "preferred", "status": "planned",
        "cost_estimate": null, "currency": null,
        "notes": "One specific practical tip for this place — best time to visit, what not to miss, local insider info, booking requirement, etc."
      }
    }
  ],
  "tasks": [
    { "title": "Task description", "due_date": "YYYY-MM-DD", "is_done": false }
  ]
}
```

## Field definitions

All allowed fields per event type:

**flight** (`date` + `sort_order` at event level):
`airline`, `flight_number`, `from_city`, `from_airport`, `departure` (datetime), `to_city`, `to_airport`, `arrival` (datetime), `cost`, `currency`, `class_of_service`, `bag_carry_on_kg`, `bag_checked_kg`, `notes`

**accommodation** (`timeline_slots` only — no `date`/`sort_order` at event level):
`hotel_name`, `hotel_address`, `destination`, `provider`, `total_price`, `currency`, `room_type`, `is_non_refundable`, `notes`
Slots (`start`/`end`): `date`, `time` (HH:MM), `sort_order`, `is_pinned`, `duration_minutes`

**transport** (`timeline_slots` only — no `date`/`sort_order` at event level):
`category` (car_rental/taxi/bus/train/ferry/other), `company`, `vehicle_type`, `cost`, `currency`, `notes`
Slots (`start`/`end`): `date`, `time` (HH:MM), `sort_order`, `is_pinned`, `duration_minutes`, `address`, `lat`, `lng`

**place** (`date` + `sort_order` at event level):
`name`, `type`, `area`, `duration_minutes`, `visit_time` (always null), `priority` (preferred/optional), `status` (always "planned"), `cost_estimate`, `currency`, `notes` (**always fill** — one specific tip for this place)

**Never emit `address`, `lat`, `lng`, `google_place_id` or `image_urls` for a place or hotel.** Wandrly resolves them against Google itself on the first trip load. Looking them up costs minutes of silent tool calls before a single line of the plan is written, burns Google quota twice, and produces exactly what the server produces anyway.

## ⚠️ ABSOLUTE RULE — never break this under any circumstance

The JSON must contain exactly one key for all trip events: **`"events"`** — a single array containing flights, accommodations, transport, and places. **Never** create separate top-level keys like `"hotels"`, `"flights"`, `"commutes"`, `"places"`, `"accommodations"`. Each event is identified by its `"record_type"` field. Any output that uses separate arrays instead of `"events"` is invalid and must not be written.

## Rules

1. **`events` is the only array for all trip events** — flights, accommodations, transport, places all go inside it. Never create separate top-level keys like `"hotels"`, `"flights"`, `"commutes"`, `"places"`.
2. `accommodation` and `transport` have **no `date` or `sort_order`** at the event level — only inside `timeline_slots`.
3. `accommodation` data has **no** `checkin_date`, `checkout_date`, `checkin_time`, `checkout_time`.
4. `transport` data has **no** `departure_datetime`, `arrival_datetime`, `departure_address`, `departure_lat/lng`, `arrival_address`, `arrival_lat/lng` — those go in the slot's `address`/`lat`/`lng`.
5. `visit_time` on places is always `null`.
6. `cost`/`total_price` always `null` — never guess prices.
7. **Flights — `departure` and `arrival` must always be full datetime strings (`YYYY-MM-DDTHH:MM:SS`), never `null`.** If the exact time is unknown, estimate based on typical flight duration for the route (e.g. TLV→MUC ≈ 4.5h, TLV→JFK ≈ 12h, TLV→BKK ≈ 11h). Always include a realistic departure time and compute arrival accordingly.
8. `passengers` — include only if names were mentioned, otherwise `[]`.
9. `tasks` — generate actionable pre-trip tasks (bookings, tickets, checks). `is_done` always `false`.
10. `trip.ai_tips` — **always populate** with 3–6 practical general tips about the destination (transport, culture, cash/card, weather, safety, tipping customs, etc.). These appear as trip-level tips in the app.
11. `place.data.notes` — **always populate** for every place with one specific practical tip: best time to visit, what to skip, how long to really spend, insider advice, booking requirement, hidden gem detail, etc. Never leave `notes` as `null` for a place.
12. Write names/places in the same language the user used in the conversation. `hotel_name` = "Hotel in [city/area]" (e.g. "Hotel in Kotor") — put the specific suggested hotel name in `notes`. No suffixes like "nights 1-2", "last night", etc.
13. `source.tool` is always `"claude"`. `source.model` = your current model version (e.g. `"claude-sonnet-4-6"`). `method` is always `"skill"`.
14. `trip.name` — use a hyphen (`-`) to join locations, never "and" / "ו-". Example: `"Munich - Berlin"`, not `"Munich and Berlin"` or `"Munich ו-Berlin"`.
15. **Mode B only**: confirmed events from the input file are copied verbatim into the output `events` array with their `reservation`/`confirmation_code` intact. New AI-generated events have no `reservation` or `confirmation_code`.

## Never look places up — Wandrly resolves them

**Do not call `places_search` (or any Google lookup) while planning.** Emit each
place with its `name` only. No `address`, no `lat`/`lng`, no `google_place_id`,
no `image_urls`.

Wandrly resolves all of it itself — `_enrich_places_from_google` fills in coords,
address, rating and photos on the first trip load, within seconds. Looking them
up yourself adds MINUTES of silent tool calls before a single character of the
plan is written, burns Google quota twice, and produces exactly what the server
produces anyway. The user sits watching an idle screen for work that was never
needed.

### If the user asked for enrichment (the slow path, for comparison only)

Batch the places into as few `places_search` calls as possible (up to 10 queries
per call) and populate `address`, `lat`, `lng` and `google_place_id`. Leave
`image_urls` null — the app fills photos in regardless. Tell the user this run is
enriching first, so the wait is expected.

This holds for BOTH delivery paths — MCP import and a written file. A file
without coordinates is correct: the app fills them in the moment it is imported.

## Delivery — import to the account when you can, file only when you can't

Once the plan is confirmed, it has to reach the user's Wandrly account. There are
two ways, and **the file is the fallback, not the default.**

### If Wandrly MCP tools are available → import directly

When tools named `import_plan` / `merge_plan` / `validate_plan` are available,
use them. Do NOT write a file and ask the user to import it by hand — that is a
manual step they explicitly do not want.

**Say what you are doing before you start.** Writing a three-week plan is thousands
of tokens and takes minutes, during which the user sees nothing — no progress bar
exists for it. A plan that is already reviewed and approved makes this worse: from
the user's side the work looks finished, so silence reads as a hang. Post one line
first — *"בונה את הקובץ ומייבא לחשבון, זה ייקח דקה או שתיים"* / *"Building and
importing now, this takes a minute or two"* — and for a long trip, split the import
by leg (`import_plan` for the first, `merge_plan` for the rest) so something lands
early instead of everything landing at the end.

1. `validate_plan` — always first. Fix every error before continuing.
2. `import_plan` for a new trip, or `merge_plan` to add onto an existing one
   (merge is the right call whenever confirmed bookings already live in the trip
   — it skips anything whose `confirmation_code` is already in the DB).
3. Report what landed: trip name, counts, and that everything is a DRAFT.

Everything written this way lands as drafts by design — the server enforces it.
Confirmed bookings are never created, modified, or resolved through this channel.
Say so plainly; it is a guarantee, not a limitation to apologize for.

**Report import errors verbatim — never guess at the cause.** The two real ones:

- **`trip_overlap` (409)** — the dates collide with an existing trip. Wandrly
  forbids two trips sharing ANY day, even a single boundary day. The error lists
  the conflicting trips by name and dates: show them and ask whether to change
  the dates, merge into that trip, or delete it. This is the single most common
  failure and it is trivially fixable — but only if the user is told what it is.
- **401 / expired token** — the connection needs re-authorizing. OAuth access
  tokens are short-lived; a retry usually mints a fresh one.

- **`This token is missing the 'trips:write-draft' scope`** — real and distinct
  from the two above. The connection is read-only: the tools still LIST (tool
  availability says nothing about granted scopes), so this only surfaces on the
  first write. Fix: reconnect Wandrly to re-consent with write access. Never
  "solve" it by writing a file instead — say what happened, since only the user
  can re-authorize.

Report the server's message verbatim. Do not translate a 409 or a 401 into "no
write access", and do not assume write access from the presence of a tool — a
wrong guess sends the user off fixing a connection that was never broken, and
that is what pushed a real user into a file-download dead end after a full plan
had already been built and approved.

### Only if MCP tools are NOT available → write the file

- Filename: `<destination>-<year>.wandrly` (e.g. `munich-2026.wandrly`)
- Path: `/mnt/user-data/outputs/<filename>`
- Format: valid JSON, UTF-8, pretty-printed (2-space indent)

After writing, call `present_files` and confirm how many confirmed events were
carried over and how many new items were added.
Tell the user to import it via the app (with the **Merge** button if the trip
already has confirmed bookings).
