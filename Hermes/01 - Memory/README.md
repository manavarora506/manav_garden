# Pillar 1 — Memory

Memory is the small, durable context that Hermes carries across sessions. Every time a session starts, these files are loaded so Hermes always knows what's going on.

## Key Files

### user.md
Who you are, your style, your preferences, and things you don't like.
- Your background and role
- Communication style preferences
- Things Hermes should never do
- Tools and workflows you use

### memory.md
Your environment, active projects, and business context.
- Projects you're currently working on
- Ongoing goals and deadlines
- Key people and relationships
- Important facts about your life/work

## Rules
- Save durable preferences and facts to memory
- Use session search for old conversations (stored in SQLite)
- Do NOT store secrets or temporary task status in memory files
- Tell Hermes "chuck that in the memory" when something important comes up
