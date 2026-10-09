# wandrly-export

A Claude skill for planning trips that land in [Wandrly](https://wandrly.ai), and for reviewing trips that are already there.

## What it does

The skill works in three modes, picked from what you ask for:

| Mode | When | Result |
|---|---|---|
| **A: Plan a new trip** | "Plan me 4 days in Rome in September…" | A day-by-day summary, then an hourly schedule, then the trip itself |
| **B: Fill the gaps** | You hand it a `.wandrly` export that has your confirmed bookings | It plans only the missing days and stops, around the bookings you already have |
| **C: Review an existing trip** | "Check day 3 of my trip, something doesn't add up" | A side-by-side current-vs-proposed comparison; nothing changes until you approve |

Every plan goes through the same checkpoints: it asks what it needs to know, shows the summary, then the hourly schedule, and delivers only after you confirm.

## How it delivers

- **With the Wandrly MCP connector** (Claude on the web or desktop, Claude Code): the trip is added straight to your account. Mode C applies an approved reorder directly, one day at a time.
- **Without it** (any other AI tool, or if you prefer a file): it writes a `.wandrly` file. Import it in Wandrly as a new trip, or use **Merge into trip** to add it to one you already have.

## What it can and cannot change

New items always land as **drafts** for you to review in the app. For bookings that are already confirmed:

- **Booking details are never changed**: dates, prices, confirmation codes and booked times stay exactly as they are.
- **A confirmed hotel or transport card can be moved within its own day**, for example a check-in that ended up mid-morning moved to the evening. It is never moved to another day.
- **Flights never move**, and nothing is moved past a flight.
- **A locked time** (a tour slot, a restaurant table) changes only if you approve that specific event. Through the connector, Claude has to ask you first. With a file, the merge screen lists each locked time with a checkbox, and only the ones you tick change.

These rules are enforced by the Wandrly server, not only by the skill.

## File format

A `.wandrly` file is JSON with a `trip` object and a single `events` array holding every flight, accommodation, transport and place, each identified by its `record_type`. Dates and positions of accommodation and transport live in their `timeline_slots`. The full format and planning rules are in [`SKILL.md`](wandrly-export/SKILL.md).

## Installation

### Claude Code

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/ronit-brosh/wandrly-skills.git /tmp/wandrly-skills-install
cp -rn /tmp/wandrly-skills-install/wandrly-export ~/.claude/skills/
rm -rf /tmp/wandrly-skills-install
```

Re-running is safe: `cp -rn` skips files that already exist. To update, delete `~/.claude/skills/wandrly-export` first.

### Claude on the web or desktop

Upload the `wandrly-export` folder as a skill in Claude's settings. Re-upload it after every update; it does not refresh on its own.

## Requirements

- Claude Pro, Max, Team or Enterprise, with Code Execution enabled
- A Wandrly account; the MCP connector is optional (see the AI tools page in Wandrly)
