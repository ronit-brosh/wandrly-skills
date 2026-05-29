---
name: wandrly-export
description: >
  Export a planned trip to a .wandrly file — the Wandrly app's native JSON format.
  Use this skill whenever the user asks to export, save, or download a trip as a .wandrly file,
  or asks to "create a wandrly file", "export to wandrly", or "save my trip for the app".
  Also trigger when a trip itinerary has just been planned in conversation and the user says
  things like "now export it", "make a file from this", or "I want to import this into my app".
  IMPORTANT: also trigger at the end of any trip planning session where a multi-day itinerary
  has just been presented — even if the user didn't ask. In that case, do NOT export automatically;
  instead ask: "רוצה שאייצא את המסלול לקובץ wandrly?" and wait for confirmation before proceeding.
  The skill reads the Wandrly schema, maps all available trip data from the conversation,
  and produces a valid .wandrly file for download.
---

# Wandrly Export Skill

Produces a valid `.wandrly` export file from trip data discussed in the conversation.

## When to trigger

Two modes:

**Explicit request** — user asks to export/save/download. Proceed directly.

**End of itinerary planning** — a multi-day itinerary was just presented (map shown,
day-by-day schedule complete). In this case: append a single short line at the end of
the response asking if the user wants a wandrly file. In Hebrew if the conversation is
in Hebrew. Example: *"רוצה שאייצא את המסלול לקובץ wandrly?"*
Do NOT export yet — wait for a "yes" / "כן" / "בטח" type confirmation.

## Schema reference

Before writing any output, read the schema:
`references/wandrly-schema.json`

The schema is the authoritative definition of all fields, types, and constraints.
Use it to validate your output mentally before writing the file.

## Output file

- Filename: `<destination>-<year>.wandrly` (e.g. `naples-2026.wandrly`, `tokyo-2027.wandrly`)
- Path: `/mnt/user-data/outputs/<filename>`
- Format: valid JSON, UTF-8, pretty-printed (2-space indent)

## Mapping process

Work through each section in order. For each section, extract what's available
from the conversation. Leave fields `null` if unknown — never invent data.

### 1. Root fields
- `wandrly_version`: always `1`
- `exported_at`: current UTC datetime in ISO 8601

### 2. `trip`
Pull destination, name, start/end dates, status, notes from conversation context.

### 3. `hotels`
One entry per booked/chosen hotel. Include: name, address, lat/lng if mentioned,
check-in/out dates and times, price, notes, rating if available.
`action`: `"booking"` if booked, `null` if just planned.

### 4. `flights`
One row per flight segment. Include airline, flight number, IATA codes,
departure/arrival datetimes (local, no timezone), class, seat if known.

### 5. `commutes`
Car rentals and transfers. Include trains, buses, taxis if they have
a defined departure/arrival with address/datetime.

### 6. `places`
One entry per attraction, restaurant, museum, etc.
- `type`: `"attraction"` / `"restaurant"` / `"museum"` / `"bar"` / etc.
- `priority`: `"preferred"` for must-do, `"optional"` for nice-to-have
- `status`: `"planned"` for all new entries
- `visit_date` + `visit_time`: fill from the itinerary schedule
- `sort_order`: sequential integer within each day
- `is_pinned`: `true` if a specific time was pinned, `false` otherwise
- `cost_estimate`: per-person estimate in EUR (or relevant currency) if mentioned
- Fill `notes` with practical tips from the conversation (wait times, booking advice, etc.)

### 7. `timeline_order`
One entry per scheduled event per day, in chronological order.
- `event_type`: one of `place`, `hotel_checkin`, `hotel_checkout`,
  `commute_pickup`, `commute_dropoff`, `airport_checkin`, `flight`
- `ref`: for places → place name (exact match). For hotels → hotel name.
  For commutes without confirmation codes → `null`.
- `sort_order`: starts at 1 per day

### 8. `expenses`
Aggregate cost estimates by category (attractions, food, transport, accommodation).
Use amounts discussed in the conversation. Mark `expense_date: null` for estimates
that span the whole trip.

### 9. `tasks`
Generate actionable pre-trip tasks from the conversation:
- Reservations that need to be made (restaurants, attractions with advance tickets)
- Bookings to confirm (hotel, flights)
- Practical checks (opening days, closures, transit tickets)
Set `is_done: false` for all. Set `due_date` to a reasonable date before the trip.

## Quality checks before writing

1. All required fields present: `wandrly_version`, `trip.name`
2. All dates in `YYYY-MM-DD` format, all datetimes in `YYYY-MM-DDTHH:MM:SS`
3. `timeline_order` refs match place names exactly
4. No invented data — unknown fields stay `null`
5. `sort_order` in `places` and `timeline_order` is sequential per day

## After writing

Call `present_files` with the output path so the user can download the file.
Confirm how many places, hotels, tasks were exported.
