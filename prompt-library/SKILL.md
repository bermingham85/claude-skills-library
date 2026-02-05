---
name: prompt-library
description: Saves effective prompts to organized library and retrieves them on request. Routes through n8n to persist to Google Drive with proper categorization and tagging.
---

# Prompt Library Skill

Captures, categorizes, and retrieves effective prompts across all AI tools (Midjourney, ChatGPT, Claude, ComfyUI, etc.).

## When to Activate

Activate when ANY of these occur:
- User says "save this prompt" or similar
- User asks for prompts they've used before
- Keywords: prompt library, save prompt, prompt template
- User creates a particularly effective prompt worth saving
- User asks "what prompts do I have for X"

## Prompt Categories

| Category | Tools | Description |
|----------|-------|-------------|
| image-generation | Midjourney, DALL-E, Stable Diffusion | Visual creation prompts |
| animation | ComfyUI, LivePortrait, MuseTalk | Video/animation prompts |
| writing | Claude, ChatGPT | Creative writing, editing |
| coding | Claude, ChatGPT, Copilot | Code generation, debugging |
| analysis | Claude, ChatGPT | Data analysis, research |
| automation | n8n, scripting | Workflow automation |
| business | Various | Marketing, email, proposals |

## Prompt Schema

```yaml
prompt:
  title: "Descriptive name"
  content: "The actual prompt text"
  tool: "midjourney|chatgpt|claude|comfyui|etc"
  category: "image-generation|writing|coding|etc"
  tags: ["tag1", "tag2"]
  variables: ["{{style}}", "{{subject}}"]  # Replaceable parts
  effectiveness: 1-5  # User rating
  notes: "When to use, tips"
  created: "2026-02-05"
  last_used: "2026-02-05"
```

## Process

### Saving a Prompt

1. **Extract prompt** from conversation

2. **Determine category and tool** from context

3. **Identify variables** (parts user would typically change)

4. **Trigger n8n webhook**:
   ```
   POST http://localhost:5678/webhook/claude-to-warp
   {
     "action": "save_document",
     "content": "prompt content here",
     "metadata": {
       "type": "prompt",
       "category": "image-generation",
       "tool": "midjourney",
       "suggested_name": "cinematic-portrait-v2",
       "tags": ["portrait", "cinematic", "lighting"],
       "variables": ["{{subject}}", "{{mood}}"]
     }
   }
   ```

5. **Confirm**: "Saved 'cinematic-portrait-v2' to prompt library (image-generation/midjourney)"

### Retrieving Prompts

1. **Parse request** for category, tool, or keywords

2. **Query library** (via webhook or local search)

3. **Present matches** with key details

4. **Offer to fill variables** if user selects one

## Storage Structure

```
Google Drive/AI_Assets/Prompts/
├── image-generation/
│   ├── midjourney/
│   │   ├── cinematic-portrait-v2.md
│   │   └── product-photography.md
│   └── stable-diffusion/
├── writing/
│   ├── claude/
│   └── chatgpt/
├── coding/
└── _index.json  # Searchable index
```

## Guidelines

- Auto-detect category when possible
- Identify reusable variables (mark as {{variable}})
- Rate effectiveness if user provides feedback
- Link related prompts
- Track which prompts get reused (popularity)

## Examples

### Save Example
```
User: "That Midjourney prompt worked great, save it"

You:
1. Extract last MJ prompt from context
2. Identify as image-generation/midjourney
3. Find variables: {{subject}}, {{style}}
4. POST to webhook
5. Say: "Saved to prompt library:
   - Name: dramatic-portrait-lighting
   - Category: image-generation/midjourney
   - Variables: {{subject}}, {{style}}
   - Tags: portrait, dramatic, lighting"
```

### Retrieve Example
```
User: "What prompts do I have for product photography?"

You:
1. Search library for "product photography"
2. Say: "Found 3 prompts:
   1. **product-hero-shot** (Midjourney) ⭐⭐⭐⭐⭐
      'Product photography of {{product}}, studio lighting...'
   2. **lifestyle-product** (Midjourney) ⭐⭐⭐⭐
   3. **ecommerce-white-bg** (Stable Diffusion) ⭐⭐⭐
   
   Want me to fill in variables for any of these?"
```

## Integration

- **Document Control**: Prompts saved via standard document flow
- **Google Drive**: Persisted with folder structure
- **Index**: Searchable JSON index updated on each save
