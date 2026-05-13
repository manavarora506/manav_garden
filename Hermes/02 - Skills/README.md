# Pillar 2 — Skills

Skills are procedural memory — reusable playbooks for how to do a task well. Think of them as recipes: consistent, repeatable, improvable over time.

## How Skills Work
- Each skill is a `.md` file with YAML front matter that tells Hermes when to use it
- Hermes reads the front matter to decide if the skill is relevant, then loads the full skill only if needed (progressive disclosure — no context bloat)
- Skills are auto-created by Hermes when it notices you doing something repeatedly
- Skills improve as you give feedback

## Skill Structure
```yaml
---
name: skill-name
description: what this skill does
triggers: [when to use it]
---
# Steps
...
```

## Resources
- Community Skills Hub: 520+ community skills available
- Install a skill by sending Hermes the skill URL and saying "install this skill"
- Anthropic official skills available (canvas design, frontend design, skill creator, etc.)

## My Custom Skills
<!-- Add links to your custom skills here -->
