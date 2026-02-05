# Claude Skills Research & Format Specification

## Official Format (December 2025)

### Location
- **Personal skills**: `~/.claude/skills/` (Windows: `C:\Users\bermi\.claude\skills\`)
- **Project skills**: `.claude/skills/` in project root
- **OpenAI Codex**: `~/.codex/skills/` (same format)

### Structure
```
skill-name/
├── SKILL.md          # Required - main instructions
├── reference.md      # Optional - detailed docs
├── examples.md       # Optional - usage examples
└── scripts/          # Optional - helper scripts
    └── helper.py
```

### SKILL.md Format
```markdown
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
context: inline|fork              # Optional: inline (default) or fork to subagent
agent: Explore|Plan|general       # Optional: which subagent to use
disable-model-invocation: true    # Optional: prevent auto-invocation
allowed-tools:                    # Optional: tools granted without approval
  - Bash(specific_command:*)
  - Read
  - Write
---

# My Skill Name

[Instructions that Claude will follow when this skill is active]

## Process
1. Step one
2. Step two

## Guidelines
- Guideline 1
- Guideline 2

## Examples
- Example usage 1
- Example usage 2
```

### Key Concepts

1. **Model-Invoked**: Claude automatically decides when to use skills based on context
2. **Progressive Disclosure**: Only ~100 tokens for metadata scan, <5k when activated
3. **Slash Commands**: `/skill-name` invokes directly (merged with old commands system)
4. **Subagents**: Can fork to Explore, Plan, or custom agents

### Invocation Types

| Type | Description | Use Case |
|------|-------------|----------|
| Reference | Knowledge applied to current work | Conventions, patterns, style guides |
| Task | Step-by-step instructions | Deployments, commits, code generation |

### Context Options

| Context | Behavior |
|---------|----------|
| `inline` (default) | Runs in current conversation |
| `fork` | Runs in separate subagent, returns summary |

### Best Practices

1. Keep SKILL.md under 500 lines
2. Move detailed reference to separate files
3. Use clear, specific descriptions
4. Include examples for complex workflows
5. Use `disable-model-invocation: true` for tasks that should only be user-invoked

## Integration with n8n

Skills can call webhooks via scripts:
```python
# scripts/webhook.py
import requests
import json
import sys

def call_webhook(action, data):
    payload = {"action": action, "data": data}
    response = requests.post(
        "http://localhost:5678/webhook/claude-to-warp",
        json=payload
    )
    return response.json()

if __name__ == "__main__":
    action = sys.argv[1]
    data = json.loads(sys.argv[2])
    result = call_webhook(action, data)
    print(json.dumps(result))
```

Then in SKILL.md:
```markdown
## Process
1. Gather metadata about the document
2. Run: `python scripts/webhook.py save_document '{"content": "...", "type": "prompt"}'`
3. Report the result
```

## Sources

- https://code.claude.com/docs/en/skills (Official Claude Code docs)
- https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview (API docs)
- https://github.com/anthropics/skills (Official examples)
- https://claude.com/blog/skills (Announcement blog)
- https://skillsmp.com/ (Community marketplace)

## Last Updated
2026-02-05
