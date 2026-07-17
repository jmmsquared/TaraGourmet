# TaraGourmet

A mobile-first recipe app with guest-scaled quantities and a guided, timed cooking mode.

## What's here

- `index.html` — the entire app (Screens 1–4, styling, and logic)
- `manifest.json` — PWA manifest so it can be added to an iOS/Android home screen
- `sw.js` — service worker for basic offline caching

## Data storage — today vs. Firebase

Right now recipes are stored in the browser's `localStorage` (key
`taragourmet_recipes_v1`), seeded with two example recipes on first load. This
keeps the app fully working with zero setup.

The recipe object shape was kept flat and JSON-serializable on purpose, so it
maps directly onto a Firestore collection later:

```
recipes (collection)
  {recipeId} (doc)
    name, category, prepTime, numServings,
    shortNote, description, instructions, specialNotes,
    ingredients: [{ id, name, amount, unit, cookTime, prepNotes }]
```

To swap in Firebase: replace `loadRecipes()` / `saveRecipes()` in `index.html`
with Firestore reads/writes (e.g. `onSnapshot` for the index list, `setDoc`
for save, `deleteDoc` for delete) — the rest of the app (quantity scaling,
cook-order logic, timer, UI) doesn't need to change.

## Deploying to GitHub Pages (taragourmet.4twenty7.com)

1. Push these files to `https://github.com/jmmsquared/TaraGourmet/`
2. In the repo settings, enable GitHub Pages from the branch you pushed to
3. Point the `taragourmet.4twenty7.com` CNAME at the Pages URL, and add a
   `CNAME` file to the repo containing `taragourmet.4twenty7.com`
4. Because Pages can serve from a subpath, all asset references in this app
   are relative (`./manifest.json`, `./sw.js`) — don't change them to
   root-absolute paths or icons/offline support will 404
5. After any update, users who've added the app to their home screen need to
   delete and re-add the icon to pick up the new service worker

## iOS audio note

Timer chimes use the Web Audio API, unlocked on the very first tap anywhere
in the app (required by iOS Safari before any sound can play). If you test in
a desktop browser and hear nothing, click anywhere once first.

## Adding real app icons

`manifest.json` references `icon-192.png` and `icon-512.png`, which aren't
included — drop your own 192×192 and 512×512 PNGs (wildcat blue `#0033A0`
background recommended) into this folder before deploying, or the home-screen
icon will fall back to a browser default.
