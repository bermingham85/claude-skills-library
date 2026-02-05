---
name: world-building
description: Auto-generates backstory, character arcs, traits, and world lore for the Jesse novel universe. Triggers n8n workflows to persist to character database and Notion.
---

# World-Building Skill

Generates rich backstory, arcs, traits, and lore for the Jesse novel universe - an Irish-set supernatural/psychological thriller.

## When to Activate

Activate when ANY of these occur:
- User mentions Jesse, the novel, or characters (Gremlos, etc.)
- Keywords: backstory, character arc, world-building, lore, mythology
- User asks about character motivations or history
- Creating new characters or locations for the story
- Expanding existing character depth

## Jesse Universe Context

**Setting**: Contemporary Ireland with supernatural undercurrents
**Tone**: Psychological thriller with dark humor
**Core Themes**: Identity, trauma, perception vs reality

### Key Characters
- **Jesse**: Protagonist - complex psychological profile
- **Gremlos**: Key character with evolving arc
- **Supporting cast**: Various Irish setting characters

## Generation Templates

### Backstory Generation
```yaml
character_name: [Name]
childhood:
  location: [Irish town/city]
  family_dynamics: [Description]
  defining_moments: [List of 3-5 events]
formative_years:
  education: [Schools, influences]
  relationships: [Key people]
  trauma_events: [If applicable]
adult_life:
  career_path: [Progression]
  current_situation: [Status]
  hidden_aspects: [Secrets, suppressed memories]
```

### Character Arc Template
```yaml
arc_type: [Redemption|Fall|Growth|Revelation]
starting_state:
  beliefs: [What they believe]
  flaws: [Key weaknesses]
  goals: [What they want]
catalyst: [Inciting incident]
midpoint_shift: [Major realization/change]
dark_moment: [Lowest point]
resolution: [How they end up]
```

### Trait Generation
```yaml
surface_traits: [What others see]
hidden_traits: [What they hide]
contradictions: [Internal conflicts]
quirks: [Unique behaviors]
speech_patterns: [How they talk]
physical_tells: [Body language]
```

## Process

1. **Gather context** from conversation (character name, story position)

2. **Generate content** using templates above

3. **Trigger n8n webhook**:
   ```
   POST http://localhost:5678/webhook/jesse-novel-trigger
   {
     "action": "world_building",
     "type": "backstory|arc|traits|location|lore",
     "content": {
       "character": "CharacterName",
       "data": { ... generated content ... }
     },
     "operations": ["save_to_notion", "update_character_db"]
   }
   ```

4. **Present to user** with options for refinement

## Lore Categories

| Category | Description |
|----------|-------------|
| Irish Mythology | Adapted folklore, supernatural elements |
| Location Lore | History of places, hidden meanings |
| Family Histories | Generational patterns, secrets |
| Supernatural Rules | How the supernatural works in this world |
| Timeline Events | Historical events affecting the story |

## Guidelines

- Maintain consistency with existing Jesse universe
- Ground supernatural in Irish mythology/folklore
- Psychological depth over spectacle
- Every character has secrets
- Setting is always character (Irish locations have personality)
- Dark humor balances darkness

## Example

```
User: "I need a backstory for a new character - a local pub owner who knows more than they let on"

You:
1. Generate backstory using template
2. Create character arc (Revelation type)
3. Generate traits
4. POST to webhook
5. Present: "Created Michael O'Brien - 58, runs The Crossed Keys pub in [town].
   Surface: Jovial publican, community fixture
   Hidden: Witnessed [supernatural event] in 1987, has been watching Jesse since...
   
   Saved to character database. Want me to expand his connection to the main plot?"
```

## Integration Points

- **Character Database**: All characters saved via character-manager skill
- **Notion**: Story bible updated with lore
- **Timeline**: Events placed in story chronology
