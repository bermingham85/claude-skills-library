---
name: document-control
description: Automatically saves documents, prompts, scripts, and other artifacts to organized locations via n8n webhook. Triggers when substantial reusable content is created.
---

# Document Control Skill

When you create substantial, reusable content (prompts, templates, scripts, documentation), this skill automatically saves it to the correct location via the n8n document control system.

## When to Activate

Activate when ANY of these occur:
- A markdown document is created (>100 words)
- A JSON/YAML configuration is generated
- A reusable prompt is created (for Midjourney, ComfyUI, ElevenLabs, etc.)
- A script or code file is generated
- User says "save this" or "template this"

## Process

1. **Detect content type** from keywords and context:
   - `midjourney`, `--ar`, `--v` → prompt/midjourney
   - `comfyui`, `workflow`, `nodes` → workflow/comfyui
   - `character`, `backstory`, `traits` → character/animation
   - `transaction`, `VAT`, `category` → financial/quickbooks
   - `elevenlabs`, `voice`, `SSML` → prompt/elevenlabs

2. **Generate metadata**:
   ```json
   {
     "type": "prompt|character|workflow|script|documentation",
     "category": "midjourney|comfyui|elevenlabs|animation|financial",
     "suggested_name": "descriptive-kebab-case-name",
     "tags": ["relevant", "tags"],
     "project": "project-name-if-applicable"
   }
   ```

3. **Call n8n webhook**:
   ```
   POST http://localhost:5678/webhook/claude-to-warp
   {
     "action": "save_document",
     "content": "<the document content>",
     "metadata": { ... }
   }
   ```

4. **Report result**: Brief confirmation of where file was saved

## Destination Routing (handled by n8n)

| Type | Category | Destination |
|------|----------|-------------|
| prompt | midjourney | `C:\Users\bermi\prompts\midjourney\` |
| prompt | comfyui | `C:\Users\bermi\prompts\comfyui\` |
| prompt | elevenlabs | `C:\Users\bermi\prompts\elevenlabs\` |
| character | * | `C:\Users\bermi\Projects\animation-agent\characters\` |
| workflow | n8n | `C:\Users\bermi\Projects\bermech-n8n-workflows\` |
| script | * | `C:\Users\bermi\Projects\{project}\scripts\` |

## Do NOT

- Save trivial or one-off content
- Ask "should I save this?" - determine automatically
- Save files directly - always use the webhook
- Duplicate existing content

## Example

```
User: "Create a Midjourney prompt for Grimlo in a winter outfit"

You:
1. Create the prompt content
2. Detect: type=prompt, category=midjourney
3. POST to webhook with content + metadata
4. Say: "Here's your prompt. Saved: prompts/midjourney/grimlo-winter-outfit.md"
```
