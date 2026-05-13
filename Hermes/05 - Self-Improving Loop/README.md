# Pillar 5 — Self-Improving Loop

Hermes gets better the more you use it — but only when you actively close the loop. Automatic does not mean magic.

## The Loop
```
Do Work → Agent Learns → Save to Memory/Skills → Search Past Sessions → Do Better Work
```

1. **Do the work** — have a conversation, complete a task
2. **Agent learns** — Hermes extracts preferences and patterns
3. **Persist** — save useful things to `user.md`, `memory.md`, or as a new skill
4. **Search** — Hermes can query past sessions in SQLite when old context matters
5. **Repeat** — every cycle makes Hermes more tailored to you

## How to Close the Loop
- Say "save that to memory" when something important comes up
- Say "turn this into a skill" after completing a repeatable workflow
- Correct Hermes explicitly — it will update its soul and memory based on feedback
- After complex work, ask: "what should we save or turn into a skill from this session?"

## The Context File (agents.md)
- Like `CLAUDE.md` — the overall project goal and structure
- Use this when coding or running terminal-based projects with Hermes
- Tells Hermes the structure, purpose, and rules of a specific project

## Signs the Loop is Working
- Hermes stops asking you things it already knows
- Skills get noticeably better over time
- Crons run without needing adjustment
- Responses feel more "you"
