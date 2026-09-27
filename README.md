# Broken Boots Road Trip

A pixel-art browser game. Ride Ginna's black AKT scooter, Kali, across all 32 departments of Colombia and collect a passport stamp in each one.

**Play:** https://32departments.vercel.app

Fan game inspired by [Broken Boots Travel](https://brokenbootstravel.com). Department facts, taglines and "best thing to do" come from the site's 32-departments pages.

## How to play

Ride around the map of Colombia and pull up to a department's flag to start its ride. Each ride is a side-scroller with its own scenery, landmark and challenge (river boats, fog, wind, floods, dark jungle nights…). Reach the end with your boots intact to earn the postcard and the stamp. The ride gets longer and faster as your passport fills up.

| Key | Action |
| --- | --- |
| Arrows / WASD | Move on the map · set the pace while riding |
| Space | Jump |
| Enter / E | Explore a department · confirm |
| P / Tab | Passport |
| C | Choose a rider |
| F | Take the ferry to San Andrés |
| M | Music on/off |
| Esc | Back |

Touch controls appear on phones and tablets. Progress is saved in your browser.

## Run locally

No build and no dependencies. The whole game is one file, `index.html`.

```bash
python3 -m http.server
# open http://localhost:8000
```

Or open `index.html` directly in a browser.

## Deploy

Hosted on Vercel as a static site. `.vercelignore` keeps repo-only files off the site.

```bash
vercel deploy --prod
```

## Development notes

See [CLAUDE.md](CLAUDE.md) for the architecture: canvas rendering, procedural sprites, map generation, per-department config and the ride lifecycle.
