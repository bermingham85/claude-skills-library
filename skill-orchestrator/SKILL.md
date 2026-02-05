---
name: skill-orchestrator
description: Master coordinator that routes requests to appropriate skills and chains multi-skill workflows. Detects intent and triggers the right skill or skill sequence.
---

# Skill Orchestrator

Master skill that detects user intent and routes to the appropriate skill(s), handling multi-skill workflows when needed.

## When to Activate

This skill is always passively monitoring. It activates to:
- Route ambiguous requests to the right skill
- Chain multiple skills for complex workflows
- Resolve conflicts between skills
- Track skill usage patterns

## Skill Registry

| Skill | Triggers On | Webhook |
|-------|-------------|---------|
| document-control | Save artifacts, organize files | /webhook/claude-to-warp |
| character-manager | Character creation, updates | /webhook/jesse-novel-trigger |
| transaction-categorizer | Financial data, expenses | /webhook/transaction/categorize |
| world-building | Backstory, lore, arcs | /webhook/jesse-novel-trigger |
| prompt-library | Save/retrieve prompts | /webhook/claude-to-warp |

## Intent Detection

### Single-Skill Patterns

| Pattern | Route To |
|---------|----------|
| "save this [prompt/document/file]" | document-control |
| "categorize these transactions" | transaction-categorizer |
| "create a character" | character-manager → world-building |
| "save this prompt" | prompt-library |
| "what's [character]'s backstory" | world-building |

### Multi-Skill Workflows

| Workflow | Skill Chain |
|----------|-------------|
| New character creation | character-manager → world-building → document-control |
| Story chapter development | world-building → character-manager → document-control |
| Financial month-end | transaction-categorizer → document-control |
| Prompt refinement | prompt-library (retrieve) → prompt-library (save improved) |

## Process

1. **Parse user intent** from message

2. **Match to skill(s)**:
   - Single skill: Route directly
   - Multiple skills: Determine sequence and dependencies

3. **Execute skill chain**:
   ```
   For each skill in chain:
     - Gather required context
     - Execute skill
     - Pass output to next skill if needed
   ```

4. **Report results** from all skills involved

## Workflow Definitions

### New Character Workflow
```yaml
name: new-character
trigger: "create a new character for Jesse"
steps:
  1:
    skill: character-manager
    action: create_entry
    output: character_id
  2:
    skill: world-building
    action: generate_backstory
    input: character_id
  3:
    skill: world-building
    action: generate_arc
    input: character_id
  4:
    skill: document-control
    action: save_to_notion
    input: all_generated_content
```

### Monthly Finance Workflow
```yaml
name: monthly-finance
trigger: "process this month's transactions"
steps:
  1:
    skill: transaction-categorizer
    action: categorize_all
    output: categorized_transactions
  2:
    skill: document-control
    action: save_report
    input: categorized_transactions
```

## Guidelines

- Default to single most relevant skill
- Only chain skills when workflow clearly requires it
- Pass minimal necessary context between skills
- Report which skills were used

## Conflict Resolution

When multiple skills could handle a request:
1. Check for explicit user preference
2. Use most specific skill (character-manager over document-control for character data)
3. Ask user if genuinely ambiguous

## Example

```
User: "Create a new antagonist for Jesse - a local guard (police officer) with a dark secret"

Orchestrator detects: New character creation workflow

1. → character-manager: Create entry
   - Name: TBD (suggest Irish name)
   - Role: Antagonist
   - Occupation: Garda Síochána
   
2. → world-building: Generate backstory
   - Incorporate Garda career path
   - Dark secret thread
   - Connection to Jesse
   
3. → world-building: Generate arc
   - Antagonist arc type
   - Collision points with Jesse
   
4. → document-control: Save all
   - Character database
   - Notion story bible

Report: "Created Sergeant Ciarán Doyle - 12 years Garda, stationed in [town].
Dark secret: [generated]. Full profile saved to character database and Notion.
Want me to develop his first scene with Jesse?"
```

## Meta-Skill Functions

- **Skill Discovery**: List available skills and their capabilities
- **Usage Analytics**: Track which skills used most (for optimization)
- **Gap Detection**: Identify requests that don't match any skill (suggests new skills)
