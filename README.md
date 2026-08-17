# Drum Time! 🥁

A single-file, installable drum machine for teaching kids rhythm. No build step,
no dependencies — `index.html` is the whole app, sounds included (Web Audio,
synthesised at runtime).

**Live:** https://jimtan123.github.io/drum-time/

## Running it locally

Open `index.html` in a browser. That's it. The service worker only registers
over http(s), so for a full PWA test serve the folder:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

## What it does

- **Drums:** hi-hat, snare, tom, bass, clap, cowbell — pick which rows show
- **Hi-hat cells have three states:** off → closed (yellow ✨) → open (orange 💫).
  An open hat rings until the next hi-hat cell of any kind closes it, so a
  following 16th chokes it short and a whole beat lets it wash out.
- **Grid resolution:** quarter, eighth, or 16th notes
- **Loop length:** 1 or 2 bars. Growing to 2 copies bar 1 into bar 2 so the
  groove keeps going and the second bar can be turned into a fill.
- Metronome (click or spoken "1 e and a"), count-in, tempo 40–160bpm,
  nursery-rhyme melodies over the beat, 3 save slots, and a lock for kids

Switching resolution matches cells by where they sit in the bar, so a hit keeps
its real timing. Coarser grids drop hits that no longer have a cell to land on —
an open hat on the "a" disappears in eighth-note mode. That's intended.

## Two things that will waste your time if you forget them

**1. Bump the service-worker cache on every change.** `sw.js` starts with

```js
const CACHE = 'drumtime-v6';
```

Change the file, bump the number. Phones and installed home-screen copies will
otherwise keep serving the old version, and in standalone PWA mode there is no
refresh button to force it. If a device is stuck: open the site in the browser
(not the home-screen icon) with a `?v=2` cache-buster, or delete the icon and
re-add it.

**2. GitHub Pages builds can wedge during a GitHub Actions outage.** Pages
"legacy" builds run on Actions infrastructure, so githubstatus.com can show
`Pages: operational` while nothing deploys. The signature is a build stuck at
`status: building` with `duration: 0` — it never errors, so anything watching
for a failure sees only silence. Check it with:

```bash
gh api repos/jimtan123/drum-time/pages/builds \
  --jq '.[0:3][] | "\(.status) \(.commit[0:7]) dur=\(.duration)"'
```

A healthy build takes ~35–50s. Once Actions is back to operational, request a
fresh build — the wedged one does not recover on its own:

```bash
gh api -X POST repos/jimtan123/drum-time/pages/builds
```

Observed 2026-08-17: a build sat at `building`/`duration=0` for 80 minutes
through an Actions `major_outage`, then completed in 49s on a rebuild request
once Actions recovered.
