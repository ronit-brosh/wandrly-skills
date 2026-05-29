# wandrly-export

A Claude skill that exports any trip itinerary planned in conversation to a `.wandrly` file — Wandrly's native import format.

## What it does

After you plan a trip with Claude, the skill automatically asks if you'd like to save it. Say yes, and it produces a `.wandrly` file with everything mapped: hotels, places, restaurants, day-by-day timeline, pre-trip tasks, and expense estimates.

The file imports directly into the Wandrly app.

## How to use

Just plan a trip normally — no special commands needed:

> *"Plan me 4 days in Rome in September, mom with a 10-year-old. Good food, fewer queues, stay near the Pantheon."*

At the end, Claude will ask:

> *"Want me to export the itinerary as a .wandrly file?"*

Say yes and the file is ready to download.

You can also ask explicitly at any point:

> *"Export this to wandrly"* / *"Save as wandrly file"*

## Example prompts

**Plan and export in one go:**
> *"Plan a week in Japan for two in April — Tokyo, Kyoto, Hiroshima. Mid-range budget, mix of culture and food. Export to wandrly when done."*

**Export an existing conversation:**
> *"We just planned my Lisbon trip. Create a wandrly file from it."*

**Explicit export:**
> *"Export this itinerary as a .wandrly file"*

**Hebrew:**
> *"תכנן לי 5 ימים בברצלונה בסוף מאי, זוג, תקציב בינוני. תייצא לקובץ wandrly."*

## What gets exported

| Section | Contents |
|---------|----------|
| `trip` | Destination, dates, name |
| `hotels` | Name, address, check-in/out, notes |
| `flights` | Segments, airports, times |
| `commutes` | Trains, transfers, rentals |
| `places` | Every attraction and restaurant, with visit date/time, tips, priority |
| `timeline_order` | Full chronological schedule per day |
| `expenses` | Estimated costs by category |
| `tasks` | Pre-trip to-dos (reservations, bookings, checks) |

## Installation

### Claude Code
```bash
npx clawhub@latest install github:YOUR_USERNAME/wandrly-skills/wandrly-export
```

### Manual
```bash
mkdir -p ~/.claude/skills
git clone https://github.com/ronit-brosh/wandrly-skills.git /tmp/wandrly-skills
cp -r /tmp/wandrly-skills/wandrly-export ~/.claude/skills/
```

## Requirements

- Claude Pro, Max, Team, or Enterprise
- Code Execution enabled in Claude settings
