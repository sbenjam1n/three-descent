# Q-001 — AI Asset Pipeline (Implementation Index)

**Index Version:** 1.0
**Queue item:** [Q-001](../QUEUE.md) — AI asset pipeline, Phase 0 onward
**Status:** PLANNED

---

## EDIT PROTOCOL

This is a living document. When updating this index:

- **Never overwrite; edit, update, extract or append.** Existing section numbers are stable references.
- **Increment the version** in the header (`Index Version:`) and the filename (`Q-001-ai-asset-pipeline-vX.X.md`).
- **Append new sections** after the last numbered section. Do not renumber existing sections.
- **Log changes** in the Version History with date, version, and summary of what changed.
- **Mark deprecated content** with `[DEPRECATED as of vX.X]` inline rather than deleting it.
- **Phase list updates:** add new items to existing phases or append new phases. Do not remove existing items; mark completed items with `[x]`.

## Table of Contents

1. Goal & Scope
2. Background & Constraints
3. Architecture
4. Phase List
5. Files & Surfaces
6. Risks & Open Questions
7. Verification
8. Version History

---

## 1. Goal & Scope

Build a reusable harness that lets AI-enhanced textures (upscaled and/or
generated) override the original palette-indexed PIG bitmaps at render time,
without changing geometry, UVs, or game logic. Model it on the GZDoom hi-res
texture-pack workflow: extract → enhance offline → drop in keyed overrides.

In scope: wall/segment textures, sprites (powerups/explosions), robot/model
polygon textures, projectiles. Out of scope (later phases): HUD/cockpit/fonts,
flat-shaded gouraud model polys (no texture).

## 2. Background & Constraints

- Rendering is **resolution-agnostic**: every texture is built from
  `bm.width`/`bm.height` read from the bitmap header; UVs are normalized; no
  hardcoded 64×64 in render logic (only the placeholder bitmap at
  `piggy.js:197`).
- All palette-index → RGBA conversion is funneled through **four** sites:
  `render.js:58`, `polyobj.js:622`, `powerup.js:96`, `laser.js:297`.
- Transparency is encoded as palette index 255 (transparent) and 254
  (super-transparent); AI steps destroy these, so alpha must be derived from the
  original, never trusted from the AI output.
- Overlay merging (`texmerge.js`) composites `tmap_num2` onto a base **in
  palette-index space** at runtime — incompatible with naive RGBA upscales.
- Wall textures tile (`RepeatWrapping`) and must stay seamless.

## 3. Architecture

Three stages:

1. **Extract** (in-repo node tool): walk the PIG, write
   `assets/original/<index>_<name>.png` (RGBA, alpha from 255/254) + a
   `manifest.json` (index, name, w, h, flags, category).
2. **Enhance** (offline, run by user): folder-in/folder-out →
   `assets/hires/<index>_<name>.png`. Faithful path = Real-ESRGAN 4× (alpha
   split + recombine); generative path = img2img with tile/structure control,
   seamless mode, original alpha reused as a hard mask.
3. **Override loader** (runtime): load `manifest.json` at startup; at each of
   the four funnels, if a hi-res override exists for the bitmap index, build the
   texture from the PNG instead of the palette path (cached). Fall back to the
   palette path when no override is present.

Overlays/texmerge are deferred (Phase 3) via **offline pre-merge**: enumerate
the finite set of `(base, overlay, rotation)` combos, composite at original res
as today, then upscale the merged result and key the override at the
`mergedTextureCache` level.

## 4. Phase List

- **Phase 0 — Harness** (this item's MVP)
  - [ ] `tools/extract-assets.mjs`: dump PIG bitmaps → `assets/original/` PNGs.
  - [ ] Emit `assets/manifest.json` (index, name, w, h, flags, category).
  - [ ] `src/assetoverride.js`: async manifest + override-texture loader + cache.
  - [ ] Wire override check into the 4 funnels (`render`, `polyobj`, `powerup`, `laser`).
  - [ ] Graceful fallback to palette path when an override is missing.
- **Phase 1 — Faithful upscale**
  - [ ] Real-ESRGAN 4× batch over `assets/original/` → `assets/hires/`.
  - [ ] Alpha-mask handling (split/threshold/recombine).
  - [ ] Verify on GitHub Pages.
- **Phase 2 — Generative replacement (subset)**
  - [ ] Pick an asset class; img2img with tile/depth control, seamless.
  - [ ] Alpha reused from original; tiling check.
- **Phase 3 — Overlays / texmerge**
  - [ ] Enumerate used `(base, overlay, rotation)` combos across levels.
  - [ ] Offline pre-merge → upscale → override at merged-cache key.
- **Phase 4 — HUD / cockpit / fonts**
  - [ ] Decide per-asset-class approach (fonts are special).

## 5. Files & Surfaces

- New: `tools/extract-assets.mjs`, `src/assetoverride.js`, `assets/` (`original/`, `hires/`, `manifest.json`).
- Edit (override hook): `src/render.js` (~58), `src/polyobj.js` (~622), `src/powerup.js` (~96), `src/laser.js` (~297).
- Reference only: `src/piggy.js` (bitmap/palette access), `src/texmerge.js` (Phase 3).

## 6. Risks & Open Questions

- **texmerge in palette space** — biggest wrinkle; mitigated by Phase-3 offline pre-merge.
- **Alpha fidelity** — AI blurs masks; derive alpha from original (the universal lesson from DOOM packs).
- **Tiling for generated assets** — requires seamless mode; non-issue for upscales.
- **VRAM / load time** — 4× across all bitmaps multiplies memory and async load; may need lazy/streamed loading or a per-class budget.
- **Open:** faithful-upscale vs generate-new vs both (affects manifest metadata). Default: keep both paths open.
- **Open:** PNG overrides shipped in-repo vs fetched from an assets host.

## 7. Verification

- Extract: PNG count == PIG bitmap count; spot-check a known texture renders identically to the palette path when no override exists.
- Override: with an override present, the hi-res texture appears; with it absent, the palette path is used (no regressions).
- Visual: side-by-side on GitHub Pages across a level with walls, sprites, and robots.

## 8. Version History

| Date | Version | Summary |
| --- | --- | --- |
| 2026-06-27 | 1.0 | Initial index created from chat research (opendoom/neural-upscale feasibility). |

---

**Edit Protocol** is stated at the top of this document. Rules: never overwrite,
always append, increment the version in the header and filename, log changes in
the Version History section, mark deprecated content inline rather than deleting
it, and do not renumber existing sections.
