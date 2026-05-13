# Pillar 4 — Crons

Crons turn Hermes from reactive into proactive. Schedule automations in plain English and Hermes handles the rest — no infrastructure required.

## How Crons Work
- Tell Hermes in natural language: "every morning at 6am do X"
- Each cron runs in a fresh isolated session (no inherited context)
- Results are sent back to your original chat (Telegram, Discord, etc.)
- Can update local files or trigger other actions after running

## Useful Flags
- `context_from` — pipe one job's output into another job
- `work_dir` — run tools from a specific project folder
- `--no-agent` — just run a script, skip the agent loop

## Safety Rules
- Cron sessions cannot recursively create more cron jobs
- Prompts must be self-contained

## My Active Crons
<!-- Document your crons here -->

| Name | Schedule | Description |
|------|----------|-------------|
|      |          |             |

## Ideas
<!-- Crons to set up -->
- [ ] Morning briefing (calendar + email + weather) → Telegram
- [ ] Daily Obsidian summary
- [ ] Weekly review digest
- [ ] GitHub PR/issue monitor
