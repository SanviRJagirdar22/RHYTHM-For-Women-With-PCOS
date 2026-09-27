# Rhythm — a PCOS cycle & metabolic tracker

A React + Vite app: sign in, add whichever features you want to your home
screen from a **+ Add** button, log daily data, and get a period calendar
that predicts your next cycle from your *own* logged history (crimson
circles = logged period days, translucent red = the predicted window).

## What's actually inside

- **Auth**: username/password, stored in the browser's `localStorage`
  (password is SHA-256 hashed client-side, never stored in plain text).
  There is no server — every account only exists in the browser it was
  created in. See "About the login system" below before using this for
  anything beyond your own device.
- **Cycle prediction**: `src/utils/cyclePrediction.js` derives your period
  start dates from what you've logged, computes a recency-weighted average
  and variance of your own cycle lengths, and blends that with a short-term
  clinical-range prior until you've logged ~5 cycles of your own (see "About
  the Kaggle dataset" below for why it works this way instead of a live
  Kaggle feed).
- **12 dashboard widgets**, added/removed via **+ Add**: cycle calendar,
  non-linear cycle window estimator, ovulation classifier (BBT + cervical
  mucus), hormonal satiety slider, smart carb-pairing log, glucose-satiety
  correlation graph, second-meal effect tracker, acanthosis nigricans photo
  log (stored locally only), androgen surge alert, cystic acne monitor,
  adaptive weight contextualizer, and a cycle-phase-aware fitness adapter.
- All personal data lives in `localStorage`, namespaced per username. Nothing
  leaves the browser.

## About the login system

This demo auth is enough to let multiple people use the same deployed app
from their own browsers without seeing each other's data, and enough to
practice with. It is **not** production-grade security:
- No password reset, no email verification, no rate limiting on attempts.
- An account only exists in the browser/device it was created on — clearing
  site data deletes it, and it won't sync across devices.

For real deployment with real users, swap `src/context/AuthContext.jsx` for
a real provider — Firebase Authentication, Supabase Auth, or Auth0 all have
straightforward React SDKs and would also let you move data storage to a
real database instead of `localStorage`.

## About the Kaggle dataset request

A static front-end app with no backend cannot call `kaggle.com` at runtime —
Kaggle requires an authenticated API key and the browser has nowhere safe to
keep one. What this app does instead, and why it's actually the better fit
for a personal health tracker:

1. It predicts almost entirely from **your own** logged cycles — no dataset
   generalizes to your body better than your own history does, and personal
   cycle data never has to leave your device.
2. Until you've logged a couple of full cycles, there's a short-term fallback
   prior in `src/utils/referenceData.js`, currently set to commonly cited
   clinical ranges for cycle length/variability (illustrative, not scraped
   from a specific dataset).

If you'd still like that fallback prior trained on a real Kaggle dataset
instead of the clinical-range placeholder: `scripts/train_cycle_model.py` is
a starting point — download a PCOS/cycle dataset from Kaggle with the Kaggle
CLI, run the script against it offline, and paste the resulting numbers into
`referenceData.js`. This keeps any bulk dataset processing offline, and keeps
the shipped app itself simple and serverless.

## Run it locally in VS Code

1. Install [Node.js](https://nodejs.org) (v18 or later) if you don't have it.
2. Open this folder in VS Code: `File → Open Folder…`
3. Open a terminal in VS Code (`` Ctrl+` ``) and run:
   ```bash
   npm install
   npm run dev
   ```
4. Open the URL it prints (usually `http://localhost:5173`) in your browser.
5. Sign up with any username/password to try it — it's just your browser's
   local storage, so feel free to experiment.

## Push it to GitHub

```bash
git init
git add .
git commit -m "Initial commit: Rhythm PCOS tracker"
gh repo create rhythm --public --source=. --remote=origin --push
# no gh CLI? create an empty repo on github.com first, then:
git remote add origin https://github.com/<your-username>/rhythm.git
git branch -M main
git push -u origin main
```

## Deploy it for free (optional)

Any static host works since this is a plain Vite build. Easiest options:

**Vercel**
```bash
npm i -g vercel
vercel
```
Follow the prompts — it auto-detects Vite and deploys in under a minute.

**Netlify**
```bash
npm run build
npx netlify-cli deploy --prod --dir=dist
```

**GitHub Pages**
```bash
npm run build
npm i -D gh-pages
# add "homepage": "https://<user>.github.io/rhythm" to package.json
# add to package.json scripts: "deploy": "gh-pages -d dist"
npm run deploy
```

## Project structure

```
rhythm-app/
  src/
    context/        AuthContext (login/signup), DataContext (all logged data)
    utils/           cyclePrediction.js (the prediction model), referenceData.js
    pages/           Login.jsx, Dashboard.jsx
    components/      CalendarView.jsx, AddFeatureModal.jsx
    widgets/         one file per dashboard feature + index.js registry
  scripts/           train_cycle_model.py (optional offline Kaggle training)
```

To add a 13th feature later: create a component in `src/widgets/`, then add
one entry to `WIDGET_REGISTRY` in `src/widgets/index.js` — it will
automatically show up in the **+ Add** modal.

## A note on the health content

The classifiers and alerts in this app (ovulation, androgen-surge, cycle
prediction) are pattern-matching on the numbers you log — they are not a
diagnosis and aren't a substitute for a clinician. Treat flagged patterns as
"worth mentioning at your next appointment," not as a medical result.
