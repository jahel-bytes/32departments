# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

"Broken Boots Road Trip": a pixel-art browser game, a fan game inspired by Broken Boots Travel (brokenbootstravel.com). Ginna (yellow jacket, gray beanie, brown "broken" boots) rides her black AKT scooter Kali across all 32 departments of Colombia. Department facts, taglines and "best thing to do" come from the site's 32-departments pages. `filmed: 1` marks departments Ginna has actually documented. The rest use our own tips (postcard label "Traveler tip").

## Running / checking

- No build, no dependencies, no tests. Everything is one file: `index.html` (inline CSS + one IIFE script). Google Fonts (Press Start 2P, VT323) is the only external resource.
- Run: open `index.html` in a browser, or `python3 -m http.server` and visit it.
- Syntax check (the only "lint"): extract the script and run `node --check`:
  `python3 -c "import re;open('/tmp/g.js','w').write(re.search(r'<script>(.*)</script>',open('index.html',encoding='utf-8').read(),re.S).group(1))" && node --check /tmp/g.js`
- The file has no `<!doctype>`, `<html>` or `<body>` on purpose. It is Artifact-publish friendly (the publisher adds the skeleton). So keep `<meta charset="utf-8">` first and the global `[hidden]{display:none!important}` rule. Without them the file breaks when opened locally (mangled regex, visible touch pad).

## Architecture (all in `index.html`)

- **Rendering:** a 480×270 canvas (`W`,`H`) scaled with `image-rendering:pixelated`. `fit()` sets CSS var `--s`, and all DOM UI sizes use `calc(N * var(--u))` so the overlay scales with the canvas. Canvas draws the world. DOM (`#ui`) draws text-heavy panels: title, HUD, prompt, dialog, postcard, passport, fail, ending, and the `#twist` challenge banner.
- **Pixel art is procedural.** There are no image assets. Sprites are built at startup with `spr(w,h,fn,outlineColor)` plus `outline()` (auto 1px outline), cached in `PROP` (`prop(key)`, `key` may be `name:variant`) and `LM` (`landmark(key,d)`). `PROP2` holds newer sprites (movers, canoe, maloca…); `prop()` falls through to it. Riders live in `CHARS` (palette + flags: `hs` hair style, `hat`, `slv`, `pat`, `beard`, `glass`, `veh:'bike'`, `cat`). `setChar(C)` rebuilds `RIDER[0..3]` (`riderFrame(f,C)` + `riderHead`), the map sprite `MINI` (`miniFrame`) and the dialog portrait (`drawPortrait`), and updates rider-name text. The in-canvas text is a custom 3×5 font (`txt`/`txt0` with a scale arg; `tw_`/`tw` measures width). Beware: local variables named `tw` (the challenge object) shadow the width helper, which is why `tw_` exists.
- **Map (`buildMap`)** is rasterized once:
  - Hand-typed lat/lon polygons (`COL` = Colombia, `SE`/`PAN` = neighbors) → `CLS` (0 sea, 1 Colombia, 2 other land).
  - Voronoi over each department's seed + capital → `DIDX` (department per pixel).
  - Ridge polylines + peaks → elevation → hillshade.
  - BFS sea distance → shallow water.
  - Rivers, labels.
  - Projection is `MX/MY` at `S=32` px/degree. Walking is allowed only on `CLS===1`. San Andrés is reachable only via the ferry (F at Bolívar/San Andrés).
- **Ride scene:** a side-scroller.
  - Parallax layers (`buildLayer(spec,…)`, kinds: mountains, hills, sea, sea7, shore, dunes, canopy, flat, plain, mesas) are prebuilt as `LW=1536`-wide periodic canvases, using periodic noise `rn()`.
  - Grass strip, props, landmark, road, obstacles, items and rider are drawn into the offscreen `FG` canvas, tinted by time of day (`TINT`), then composited.
  - Glow effects (headlight, lights, fireflies, darkness overlay `drawDark`) are drawn after the tint.
- **Per-department configuration is layered:**
  1. `RAW`/`DEPTS`: facts, capital, pin/seed coords, `biome`, and optional `lm` (landmark), `fx`, `time`, `houses`, `noSea`, `farSnow`.
  2. `BIOMES[biome]`: sky, far/mid layer specs, ground colors, road type, default props, obstacles, item.
  3. `SCENE[id]`: per-department prop list and far/mid spec overrides (paddies, flood, patch, river…), merged over the biome.
  4. `DX[id]` (between the `/* DX-BEGIN */` … `/* DX-END */` markers, one `DX.<id>={...}` block per department): the department's own art, overriding the layers above.
     - `lm()` is the end-of-ride landmark (used instead of `d.lm`); `lmFx(g,x,y,t,r)` animates it.
     - `back()` is a distant set piece drawn between the far and mid layers (`backY`, `backGap`, `backPar`).
     - `props` holds new roadside sprites (keys prefixed `<id>_`, looked up by `prop()` via `DXP`), and `scene` is the roadside pool.
     - `far`/`mid` are merged into the layer specs, and `time` optionally overrides the time of day.
     - Blocks must stay self-contained, with no top-level declarations, because they share the IIFE scope.
  5. `TW[id]`: the department's challenge (`k` is one of boat, gaps, movers, dark, zones, bumps, surf, rocks, wind, mirage, fog, foam, vents, arches, timer, quota, plus params). `MOVER` holds moving-hazard stats, and `OBS` holds obstacle hitboxes (`h`, `w`, optional `fx`).
- **Ride lifecycle:**
  - `startRide(d, demo)` builds `R`. Difficulty `k = lvl/31` comes from stamps already collected and scales length (2200→3800), base speed, spacing and challenge intensity.
  - `genCourse(R)` lays out obstacles, items, gaps, zones, surfaces, fog, arches and vents from a seeded `rng` (so each department's layout is deterministic). Landslides, wind and foam use `Math.random` at runtime.
  - `updateRide` handles physics, collisions, `hurt()`, `failRide(title,msg)` and `arrive()`. The postcard appears, unless the Caldas quota isn't met.
  - `drawRideScene` draws the frame, and `drawRideHud` draws the in-canvas HUD.
  - The title screen reuses `startRide(meta, true)` as a demo, with no challenge.
- **State machine:** the global `state` is one of title, select (rider picker, reached from title or C on the map), dialog, map, passport, ferry, ride, postcard, fail, ending, art. `setState()` toggles DOM panels. `transition(mid)` does the iris wipe; game updates pause while `TR` is set. Input goes through `K` (held) and `JP` (just pressed, cleared each frame) via `KEYMAP` and touch buttons (`data-k`).
- **Art gallery (`state` art):** `#art` is a scrolling full-page overlay outside `#wrap`, opened with G (title, map), the title's `#artBtn`, or `#art` in the URL (`art/index.html` redirects there for `/art`). `ART_BUILD[tab]` builds a tab on first open from the live sprite functions (`charSpr`, `landmark`, `prop`, `MAPC`). Backdrops call `startRide(d,true)` + `drawRideScene()` per department, copy the frame, then restore `R`. They are queued in `ART.queue` and painted one per tick by `artPump` only while the gallery is open. New props, movers, items or landmarks show up automatically; give them a readable name in `ART_NAME` (retired `d.lm` landmarks are listed in `ART_LM`).
- **Persistence:** `localStorage['bbt32']` holds `{v:[visited ids], intro, end, c:rider id}` (`SAVE`), wrapped in try/catch.
- **Audio:** a WebAudio cumbia loop (`sched`) plus `SFX.*`. It starts on first input, M toggles music, and it suspends while the tab is hidden.

## Testing approach used so far

With no test suite, QA was done by loading a copy of the page with a `window.__G` hook injected before the final `document.fonts` line. The game's `requestAnimationFrame` was replaced so frames can be stepped synchronously, and a scripted autopilot jumped obstacles and gaps, held ← in zones, and held → on the timer. Every department was run at level 0 and level 31, checking for a postcard with all 3 boots. Canvas frames were saved via `toDataURL()` for visual review.

Background browser tabs pause `requestAnimationFrame` and throttle timers, so drive the loop manually instead of waiting on real time. Keep such hooks out of the committed `index.html`.
