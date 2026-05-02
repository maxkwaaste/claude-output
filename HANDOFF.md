# Handoff: Claude Output Archive - Olive Branch Selection

## Status

Max is choosing an olive branch decoration style for `~/ClaudeCode/claude-output/index.html`. An interactive evaluator with 10 variations is ready at `olive-eval.html`. Max needs to open it, pick a number, and tell you which one(s) to apply.

## What's been done

### Bug fixes (committed, pushed to `maxkwaaste/claude-output`)
Three interactive bugs in index.html were fixed:

1. **Date sort tiebreaker** (line ~871): `b.date.localeCompare(a.date) || a.title.localeCompare(b.title)` -- all entries shared the same date so sort did nothing
2. **Search sort hijack removed** (line ~941): The render function used to force sort to "relevance" whenever a search was active. Now it just enables the relevance option without forcing it.
3. **Clear-all expanded** (line ~1100): Now also resets `searchQuery`, `searchInput.value`, `clearBtn`, `sortMode` back to `date-desc`, and the sort dropdown value.

### Olive evaluator (`olive-eval.html`)
Uses real CC0-licensed SVG paths from OpenClipart (Laurel-Wreath.svg, viewBox `0 0 1475.4 801.7`). The wreath has a left branch (stem path3035 + ~14 leaf paths) and a mirrored right branch (stem path3862 + ~14 leaf paths). All paths are `fill="#000000"` in the source.

The 10 variations:
1. Full Wreath, Small -- 200px, olive green `#5c6b3c`, 30% opacity, scaleY(-1)
2. Full Wreath, Large -- 400px, olive green, 25% opacity
3. Half Drape -- cropped viewBox showing only y>400 region, 300px, 35% opacity
4. Stems Only -- just path3035 + path3862, no leaves, 350px, 40% opacity
5. Gold Wreath -- 300px, gold `#b8943e`, 20% opacity
6. Dark Olive, Tight -- 180px, dark olive `#3d4a24`, 50% opacity
7. Wide Ceremonial -- 500px, 15% opacity watermark
8. Top Leaves Only -- cropped to y<200, 250px, 40% opacity
9. Silhouette Badge -- 120px inside circle stroke, 60% opacity
10. Split Drape -- left/right branches separated with gold dot between, 300px total, 35% opacity

## What to do next

### When Max picks a variation number

Replace two sections in `index.html`:

**1. CSS (lines ~69-79):** Replace the `.olive-branches` CSS block:
```css
/* ─── Roman olive branches ─── */
.olive-branches {
  display: flex; justify-content: center; align-items: flex-start;
  padding: 10px 0 0; gap: 6px; opacity: 0.4;
}
.olive-branches svg { width: 80px; height: 32px; }
.olive-branches svg path { fill: var(--olive); }
.olive-branches .dot {
  width: 5px; height: 5px; border-radius: 50%; background: var(--gold-500);
  margin-top: 12px; flex-shrink: 0;
}
```

**2. HTML (lines ~472-505):** Replace the `<div class="olive-branches">...</div>` block with the chosen variation's SVG markup from `olive-eval.html`.

To extract the right SVG: in `olive-eval.html`, find `.olive-var-N` (where N is the chosen number). Copy the SVG element inside it. Wrap it in the `.olive-branches` div in index.html. Adjust the CSS to match the variation's sizing/opacity.

### SVG source reference

The complete SVG paths are from `https://openclipart.org/download/203594/Laurel-Wreath.svg` (CC0 public domain). The raw file is also saved locally at `~/ClaudeCode/claude-output/laurel-wreath-openclipart.svg`. The evaluator file `olive-eval.html` contains all 10 variations with the full untruncated path data -- use it as the source of truth.

Key rendering notes:
- Original wreath has stems going UP. Apply `transform="scaleY(-1)"` for draping-down effect.
- Left branch: 1 stem path + 14 leaf paths + 2 bottom-curl paths (26 paths total)
- Right branch: mirror of left (26 paths total)
- All paths use `fill` only, no strokes, in the source

### Dark mode consideration

The current index.html dark mode overrides `--olive` and `--olive-light` CSS variables. If the chosen variation uses hardcoded fill colors (like `#5c6b3c`), convert them to `var(--olive)` so dark mode still works. The dark mode section is around line ~440 in the CSS (`body.dark`).

### After applying

- Commit and push to `maxkwaaste/claude-output`
- Can optionally delete `olive-eval.html` and the raw SVG files (`green-laurel-wreath.svg`, `laurel-wreath-openclipart.svg`, `olive-wreath-wikimedia.svg`) since they're just working files

## Skills and workflows that interact with index.html

The `output-page` skill (`~/.claude/skills/output-page/SKILL.md`) auto-adds entries to the `#index-data` JSON array in `index.html` whenever a new output page is generated. It does NOT touch the header/olive branches area -- those are purely visual and won't conflict.

The `design-evaluator` skill (`~/.claude/skills/design-evaluator/SKILL.md`) is the general workflow for producing these toggle-panel evaluator files. It was used to create the original `design-eval.html` for the index page improvements.

## Repo state

- Repo: `maxkwaaste/claude-output` (private)
- Branch: `main`
- Last commit: `03b33ef` -- "Fix interactive bugs in index.html + add olive branch evaluator with real CC0 SVG paths"
- Working directory: `~/ClaudeCode/claude-output/`
- Untracked files: `.claude-flow/`, `green-laurel-wreath.svg`, `laurel-wreath-openclipart.svg`, `olive-wreath-wikimedia.svg`
- Modified (unstaged): `2026-04-12-tts-evaluation.html` (unrelated, separate work)
