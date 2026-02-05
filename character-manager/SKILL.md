---
name: character-manager
description: Manages character database entries. When characters are mentioned, created, or described, automatically updates CHARACTER_DATABASE.json and can trigger reference image generation via n8n.
---

# Character Manager Skill

Manages the character database for animation and creative projects. Automatically detects character-related activity and maintains consistency.

## When to Activate

Activate when ANY of these occur:
- User uploads a character image
- User names a new character
- User describes character traits, backstory, or appearance
- User writes dialogue that reveals character personality
- Known character names mentioned: Grimlo, Tulio, Milka, Dr. Karen, Nana, Roopa, Mateus

## Database Location

`Projects/animation-agent/characters/CHARACTER_DATABASE.json`

## Process

### On New Character Detection

1. **Extract character data** from context:
   - Name
   - Type (human/animal/creature/puppet)
   - Physical description
   - Personality traits
   - Voice notes (if mentioned)
   - Relationships to other characters

2. **Check if character exists** in database

3. **Call n8n webhook**:
   ```
   POST http://localhost:5678/webhook/claude-to-warp
   {
     "action": "character_update",
     "character": {
       "name": "Character Name",
       "type": "human|animal|creature|puppet",
       "description": "Physical description",
       "traits": ["trait1", "trait2"],
       "flaws": ["flaw1"],
       "voice_notes": "Voice description",
       "project": "Milka Musical|Grimlo|Jesse"
     },
     "operations": ["create_entry", "update_traits"]
   }
   ```

4. **Report**: Brief confirmation of update

### On Character Image Upload

Add `"image_base64": "<base64 data>"` to payload and include `"generate_poses"` in operations to trigger 5-view reference generation.

## Character Schema

```json
{
  "name": "Character Name",
  "type": "human|animal|creature|puppet",
  "visual": {
    "description": "Physical description",
    "reference_images": {
      "front": null,
      "back": null,
      "left": null,
      "right": null,
      "expressive": null
    }
  },
  "voice": {
    "elevenlabs_voice_id": null,
    "voice_description": "Description of voice"
  },
  "personality": {
    "traits": [],
    "flaws": [],
    "social_attitude": "Description"
  },
  "background": {
    "backstory": "Brief backstory",
    "relationships": []
  },
  "projects": ["Project names"]
}
```

## Known Characters

| Name | Project | Type |
|------|---------|------|
| Grimlo | Grimlo's World | creature |
| Tulio | Milka Musical | human |
| Milka | Milka Musical | animal |
| Dr. Karen | Milka Musical | human |
| Nana | Milka Musical | human |
| Roopa | Milka Musical | human |
| Mateus | Milka Musical | human |

## Guidelines

- Load character context silently when writing about known characters
- Auto-extract traits from dialogue and description
- Update database when new info emerges
- Don't ask "should I add this?" - just do it
- Brief confirmation: "Updated Grimlo in character database"

## Example

```
User: "Write a scene where Dr. Karen threatens to quit again"

You:
1. Check database for Dr. Karen
2. Load: therapist, sarcastic, burnt-out
3. Write scene maintaining consistency
4. If new traits revealed, POST update to webhook
5. Continue without interruption
```
