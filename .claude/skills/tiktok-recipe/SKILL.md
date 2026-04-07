---
name: tiktok-recipe
description: >
  Extracts a structured recipe from a TikTok video URL. Use when the user provides a TikTok
  link and wants to turn it into a recipe. Handles scraping video description, captions, and
  on-screen text, then formats the result as a clean markdown recipe file. Triggers on:
  "tiktok recipe", "make a recipe from this tiktok", "extract recipe from", any TikTok URL
  (tiktok.com or vm.tiktok.com) combined with recipe intent.
---

# TikTok Recipe Skill

## Workflow

### Step 1: Extract content from TikTok

Use `mcp__Claude_in_Chrome__navigate` to open the TikTok URL, then `mcp__Claude_in_Chrome__get_page_text` to extract all visible text.

Capture:
- Video title / caption (usually the richest source of recipe info)
- Ingredient list (often in the caption or pinned comments)
- Any steps mentioned in description or on-screen text
- Creator name (for attribution if needed)

If the page requires interaction (e.g., cookie banner), use `mcp__Claude_in_Chrome__find` and `mcp__Claude_in_Chrome__javascript_tool` to dismiss it, then re-extract.

**Fallback**: If Chrome tools are unavailable or return empty content, use `WebFetch` on the URL directly.

### Step 2: Parse the recipe

From the extracted text, identify:
- **Title** — dish name
- **Servings** — if mentioned; otherwise omit
- **Timing** — prep/cook time if mentioned; otherwise omit
- **Ingredients** — with quantities and units; group logically if the dish has multiple components
- **Method** — ordered steps; infer logical order if steps are scattered or implied
- **Notes** — tips, substitutions, or finishing touches mentioned

If critical information is missing (e.g., quantities not given), make a reasonable culinary inference and note it with *(estimated)*.

### Step 3: Format and save

Format as a markdown file matching the project's recipe style (see below). Save to the recipes directory using `Write`.

Suggest a filename in kebab-case from the dish title (e.g., `spicy-korean-corn-dogs.md`).

---

## Recipe Markdown Format

Match the style of existing recipes in the project:

```markdown
# [Dish Title]
*Serves [N]*

---

## [Component or Section Name]

### Ingredients
- [quantity] [unit] [ingredient]
- ...

### Method
[Prose instructions. Write in second person, present tense. Detailed and precise — specify
temperatures, times, visual cues. One paragraph per logical stage.]

---

## [Next Component]

### Ingredients
- ...

### Method
...

---

## Assembly

1. [Step]
2. [Step]
```

**Key formatting rules:**
- Use `---` horizontal rules between sections
- Group ingredients under `### Ingredients` and steps under `### Method` within each component
- If the dish is simple (one component), use a single ingredients + method block — no subsections needed
- Write method as flowing prose, not a numbered list (reserve numbered lists for multi-step assembly)
- Include sensory cues: colour, texture, smell, sound — not just timings

---

## Handling Incomplete Content

| Situation | Action |
|-----------|--------|
| No quantities given | Estimate from common ratios; mark with *(estimated)* |
| Steps out of order | Reorder logically; note "steps inferred from video" at top |
| No servings info | Omit the *Serves N* line |
| Video is not a recipe | Tell the user clearly; don't fabricate |
| Multiple dishes in one video | Ask user which to extract, or extract all with separate files |
