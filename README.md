# wandrly-skills

Claude skills for the [Wandrly](https://wandrly.app) travel app.

These skills let Claude export trip itineraries directly into Wandrly's native format — so anything you plan in a conversation can land in the app in one click.

## Available skills

| Skill | What it does |
|-------|-------------|
| [wandrly-export](./wandrly-export/) | Exports a planned itinerary to a `.wandrly` file, ready to import into the app |

## Installation

### Claude Code
```bash
npx clawhook@latest install github:ronit-brosh/wandrly-skills/wandrly-export
```

### Manual
```bash
mkdir -p ~/.claude/skills
git clone https://github.com/ronit-brosh/wandrly-skills.git /tmp/wandrly-skills
cp -r /tmp/wandrly-skills/wandrly-export ~/.claude/skills/
```

## Requirements

- Claude Pro, Max, Team, or Enterprise
- Code Execution enabled in settings

## License

MIT
