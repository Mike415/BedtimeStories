# New Story Playbook

This document is the complete, step-by-step guide for adding a new story to the [BedtimeStories](https://mike415.github.io/BedtimeStories/) site. Any Manus task (or human) can follow this playbook from scratch to produce a fully animated, illustrated storybook and publish it to GitHub without needing prior context.

---

## Overview

Each story is a self-contained React app bundled into a folder under `stories/`. The homepage (`index.html` at the repo root) links to each story by folder name. Adding a new story involves five stages:

1. Write the story and plan illustrations
2. Generate watercolor illustrations
3. Build the story app in the Manus `bedtime-book` project
4. Rebuild the JS/CSS bundle and package the story folder
5. Push a PR to this repo and update the homepage

---

## Stage 1 — Write the Story

**Children's details:**
- **Jordan** — girl, age 4.5, strawberry-blonde hair in a ponytail, blue-grey eyes, often wears pink or floral clothes. Curious, caring, loves animals and being the "big sister."
- **Connor** — boy, age 2.25, short dark brown hair, big brown eyes, navy/dark clothes. Energetic, giggly, points at everything, says "THAT!" a lot.

**Story structure (10 pages + cover):**

| Page | Purpose |
|---|---|
| Cover | Title art — sets the mood and setting |
| 1 | Opening — establish the magical world or inciting moment |
| 2 | Entering the adventure — the world opens up |
| 3–7 | The journey — 5 scenes, each with a new animal friend or wonder |
| 8 | A gentle challenge or emotional moment |
| 9 | Resolution — the challenge is overcome with kindness or courage |
| 10 | Homecoming / bedtime — warm, sleepy, safe ending |

**Writing guidelines:**
- Each page: 3–5 sentences, rich and descriptive but readable aloud in ~30 seconds.
- Tone: warm, lyrical, gently humorous. Connor's toddler reactions (giggling, pointing, saying "THAT!") are always charming moments.
- Jordan leads and explains; Connor reacts and delights.
- End on a sleepy, cozy note — the children are home, safe, and drifting toward sleep.
- Avoid scary moments. Animals are always friendly and welcoming.

**Illustration plan:** For each page, write a detailed image generation prompt (see Stage 2 format below).

---

## Stage 2 — Generate Watercolor Illustrations

Use the Manus `generate` tool (image generation mode) to produce **11 images**: 1 cover + 10 pages.

**Prompt template for each image:**

```
Soft, dreamy children's book watercolor illustration. [SCENE DESCRIPTION].
A girl aged 4-5 with strawberry-blonde hair in a ponytail, wearing [COLOR] dress.
A toddler boy aged 2 with short dark brown hair, wearing a navy/dark shirt.
Warm, luminous watercolor washes. Soft edges, gentle color bleeding.
Storybook style, no text, no borders. Aspect ratio 4:3 landscape.
```

**Cover prompt additions:** Include the story title as part of the scene description (e.g., "the title 'The Dragon in the Garden' written in hand-lettered style at the top"). Make the cover more dramatic and magical than interior pages.

**Image naming convention:**
- Save to `/home/ubuntu/webdev-static-assets/` with names like `story-slug-cover.png`, `story-slug-page1.png` … `story-slug-page10.png`
- Upload each image using `manus-upload-file` to get a permanent CDN URL
- Record all 11 CDN URLs — they will be embedded directly in the app code

---

## Stage 3 — Build the Story App

The story app lives at `/home/ubuntu/bedtime-book` (a Vite + React + Tailwind project). The entire story content lives in one file: `client/src/pages/Home.tsx`.

### 3a. Choose a story slug

The slug is a short, lowercase, hyphenated identifier for the story folder. Examples:
- `firefly` (Pip & Felix's Firefly Adventure)
- `magical-land` (Jordan & Connor's Magical Land)
- `dragon-garden` (a hypothetical new story)

### 3b. Edit `client/src/pages/Home.tsx`

Replace the entire file content with the new story. The file has this structure:

```tsx
// ─── Image URLs ───────────────────────────────────────────────────────────────
const IMAGES = {
  cover: "https://cdn.example.com/story-slug-cover.png",
  page1:  "https://cdn.example.com/story-slug-page1.png",
  // ... page2 through page10
};

// ─── Story Data ───────────────────────────────────────────────────────────────
interface StoryPage {
  id: number;
  text: string;
  image: string;
  isNight: boolean;  // true = firefly + star particles appear on this page
}

const STORY_PAGES: StoryPage[] = [
  { id: 1, image: IMAGES.page1, isNight: false, text: "..." },
  // ... pages 2–10
];

// ─── Cover Data ───────────────────────────────────────────────────────────────
const COVER = {
  title: "Story Title Here",          // e.g. "Jordan & Connor's Magical Land"
  subtitle: "The Whispering Gate",    // e.g. the adventure name
  dedication: "A bedtime adventure for two",
  buttonLabel: "Begin the Adventure",
  image: IMAGES.cover,
};
```

**`isNight` guidance:** Set `isNight: true` for pages set at dusk or nighttime — this triggers animated firefly and twinkling star particles over the illustration.

**Keep all other code unchanged** — the animation engine, navigation, orientation-aware layout, and cover page component are all reusable and must not be modified.

### 3c. Update `client/index.html`

Change the `<title>` tag to match the new story title:
```html
<title>Jordan &amp; Connor's Dragon Garden</title>
```

### 3d. TypeScript check

```bash
cd /home/ubuntu/bedtime-book
npx tsc --noEmit
```

Fix any errors before proceeding.

---

## Stage 4 — Build and Package

### 4a. Build the bundle

```bash
cd /home/ubuntu/bedtime-book
pnpm run build
```

The output lands in `dist/public/`. The JS and CSS files will have content-hashed names like `index-AbCdEfGh.js`.

### 4b. Package the story folder

Run this Python script to create a clean, deployment-ready story folder:

```python
import re, os, shutil

DIST   = "/home/ubuntu/bedtime-book/dist/public"
SLUG   = "dragon-garden"          # ← change to the new story slug
TITLE  = "Jordan & Connor's Dragon Garden"  # ← change to the new story title
OUT    = f"/tmp/{SLUG}"

shutil.rmtree(OUT, ignore_errors=True)
os.makedirs(f"{OUT}/assets")

# Copy and rename assets
js_src  = next(f for f in os.listdir(f"{DIST}/assets") if f.endswith(".js"))
css_src = next(f for f in os.listdir(f"{DIST}/assets") if f.endswith(".css"))
shutil.copy(f"{DIST}/assets/{js_src}",  f"{OUT}/assets/story-app.js")
shutil.copy(f"{DIST}/assets/{css_src}", f"{OUT}/assets/story-app.css")

# Write clean index.html
html = f"""<!doctype html>
<html lang="en">
  <head>
    <base href="/BedtimeStories/stories/{SLUG}/">
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1" />
    <title>{TITLE}</title>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link href="https://fonts.googleapis.com/css2?family=Crimson+Pro:ital,wght@0,400;0,500;0,600;1,400;1,500&family=Playfair+Display:ital,wght@0,700;0,800;1,700&display=swap" rel="stylesheet" />
    <script type="module" crossorigin src="assets/story-app.js"></script>
    <link rel="stylesheet" crossorigin href="assets/story-app.css">
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>"""

with open(f"{OUT}/index.html", "w") as f:
    f.write(html)

print(f"Done: {OUT}")
print(os.listdir(OUT))
```

---

## Stage 5 — Push a PR and Update the Homepage

### 5a. Clone / pull the repo

```bash
# If not already cloned:
gh repo clone mike415/BedtimeStories /home/ubuntu/BedtimeStories-fix

cd /home/ubuntu/BedtimeStories-fix
git checkout main && git pull origin main
```

### 5b. Create a feature branch

```bash
git checkout -b add-story-dragon-garden
```

### 5c. Copy the story folder into the repo

```bash
cp -r /tmp/dragon-garden /home/ubuntu/BedtimeStories-fix/stories/
```

### 5d. Add the story card to the homepage

Open `/home/ubuntu/BedtimeStories-fix/index.html` and add a new `<a>` card inside the `.story-list` div, **before** the `<!-- Future stories will be added here -->` comment:

```html
<a href="stories/dragon-garden/" class="story-card">
    <h2>Jordan &amp; Connor</h2>
    <p>Dragon Garden</p>
</a>
```

### 5e. Commit and push

```bash
cd /home/ubuntu/BedtimeStories-fix
git add stories/dragon-garden/ index.html
git commit -m "Add new story: Jordan & Connor's Dragon Garden"
git push origin add-story-dragon-garden
```

### 5f. Open a pull request

```bash
gh pr create \
  --title "Add new story: Jordan & Connor's Dragon Garden" \
  --body "New animated watercolor storybook: Jordan & Connor's Dragon Garden. 10 pages, 11 custom illustrations, full orientation-aware layout." \
  --base main \
  --head add-story-dragon-garden
```

### 5g. Verify after merge

After the PR is merged and GitHub Pages rebuilds (~30–60 seconds):

1. Visit `https://mike415.github.io/BedtimeStories/` — the new card should appear.
2. Visit `https://mike415.github.io/BedtimeStories/stories/dragon-garden/` — the story should load correctly.
3. Test on mobile in both portrait and landscape orientations.

---

## Quick Reference

| Item | Value |
|---|---|
| Repo | `mike415/BedtimeStories` |
| Live site | `https://mike415.github.io/BedtimeStories/` |
| Story app source | `/home/ubuntu/bedtime-book` (Manus sandbox) |
| Story content file | `client/src/pages/Home.tsx` |
| Build command | `pnpm run build` (from `/home/ubuntu/bedtime-book`) |
| Asset output | `dist/public/assets/` |
| Story folder pattern | `stories/{slug}/index.html` + `stories/{slug}/assets/story-app.{js,css}` |
| Base href pattern | `<base href="/BedtimeStories/stories/{slug}/">` |
| Homepage card pattern | `<a href="stories/{slug}/" class="story-card">` |

---

## Existing Stories

| Slug | Title | Characters |
|---|---|---|
| `firefly` | Pip & Felix's Firefly Adventure | Pip the bunny, Felix the fox |
| `magical-land` | Jordan & Connor's Magical Land | Jordan (girl, 4.5), Connor (boy, 2.25) |

---

*Playbook last updated: February 2026. Maintained by Manus AI.*
