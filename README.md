# 🥗 Gut Tracker

A personal health tracking app for logging meals, gut symptoms, morning check-ins and weight. Built to help identify food intolerances, irritants and patterns over time.

Runs entirely in the browser. Data lives in your own Google Sheet. No servers, no subscriptions, no third-party databases.

---

## Features

- **Meal logging** — photo + ingredient input, Claude AI auto-detects meal name, calories, macros and irritant categories
- **Irritant tagging** — 14 categories including histamine, gluten, dairy, FODMAPs, nightshades, oxalates, lectins, salicylates, sulphur, caffeine and more
- **Symptom tracking** — bloating score, symptom types (gas, cramps, heaviness, nausea, reflux, low energy, energy crash, weakness, migraine), linked to the triggering meal with timestamp
- **Morning check-in** — overall feeling, energy, bloating, sleep quality, Bristol stool type, hours slept — auto-linked to previous day's meals
- **Daily targets** — calorie and macro progress bars with remaining amounts, personalised for recomposition goal
- **7-day history** — bar chart showing daily calorie % of target
- **Weight log** — with time-of-day context (morning weight flagged as most accurate)
- **Dark mode** — automatic, follows system preference
- **PWA-ready** — add to phone home screen, works like a native app

---

## Tech stack

| Layer | Tool |
|---|---|
| Frontend | Plain HTML + CSS + JS, hosted on GitHub Pages |
| Database | Google Sheets via Apps Script web app |
| AI analysis | Claude API (Sonnet) — vision + text |
| Hosting | GitHub Pages (free, auto-deploys on commit) |

No frameworks. No build step. No dependencies. One `index.html` file.

---

## Setup

### 1. Claude API key
1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Create an API key
3. Paste it into the app's Settings screen — stored locally on your device only, never sent anywhere except Anthropic's API

### 2. Google Sheets backend
1. Create a new Google Sheet
2. Go to **Extensions → Apps Script**
3. Paste the contents of `apps-script.js` (see below), replacing all existing code
4. Click **Deploy → New deployment → Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Copy the deployment URL
6. Paste it into `index.html` as the value of `SHEET_URL_DEFAULT`

> ⚠️ When updating the Apps Script, always use **Deploy → Manage deployments → edit (pencil icon) → New version**. This keeps the same URL. Creating a "New deployment" generates a new URL every time.

### 3. GitHub Pages
1. Fork or clone this repo
2. Go to repo **Settings → Pages**
3. Source: **Deploy from branch → main → / (root)**
4. Your app will be live at `https://YOUR-USERNAME.github.io/gut-tracker`

### 4. Add to phone home screen
- **iPhone (Safari):** Share → Add to Home Screen
- **Android (Chrome):** Menu (⋮) → Add to Home Screen

---

## Google Sheets structure

The Apps Script automatically creates these sheets on first use:

### Meals
| Column | Description |
|---|---|
| Schema | Version of the data schema that wrote this row |
| ID | UUID — used to link symptoms back to a meal |
| Timestamp | ISO 8601 datetime |
| Meal Name | Auto-detected or manually entered |
| Calories | kcal |
| Protein (g) | grams |
| Carbs (g) | grams |
| Fat (g) | grams |
| Mood | calm / neutral / rushed / stressed / happy |
| Ingredients | Raw text entered by user |
| Notes | Free text |
| Irritants | JSON — e.g. `{"gluten":["pasta"],"fodmap":["onion","garlic"]}` |
| Symptom Score | 1–5 bloating scale |
| Symptom Type | Comma-separated symptom list |
| Symptom Time | ISO 8601 — when symptoms started |
| Symptom Notes | Free text |

### Morning Log
| Column | Description |
|---|---|
| Schema | Version |
| ID | UUID |
| Timestamp | ISO 8601 |
| Overall feeling | 1–5 |
| Energy level | 1–5 |
| Bloating | 1–5 |
| Bristol type | 1–7 (Bristol Stool Scale) |
| Stool notes | Free text |
| Sleep quality | 1–5 |
| Sleep hours | Decimal e.g. 7.25 = 7h 15min |
| Morning notes | Free text |
| Linked meal IDs | Comma-separated UUIDs of previous day's meals |

### Weight
| Column | Description |
|---|---|
| Schema | Version |
| Timestamp | ISO 8601 |
| Weight (kg) | Decimal |
| Time of day | morning / midday / evening |
| Notes | Free text |

### Errors
Automatically populated when the Apps Script catches an unhandled exception. Useful for debugging.

---

## Daily targets

Personalised for recomposition (lose fat, preserve/build muscle):

| | Target |
|---|---|
| Calories | 2,100 kcal |
| Protein | 160g |
| Carbs | 190g |
| Fat | 65g |

Based on: Male · 173cm · 74.4kg · 33yo · lightly active · recomp goal.
Calculated using Mifflin-St Jeor BMR → TDEE (×1.375) → mild deficit (−350 kcal).
Protein set at 2.1g/kg for muscle preservation.

To change targets, update the `GOALS` constant in `index.html`:
```javascript
const GOALS = { cal:2100, pro:160, carb:190, fat:65 };
```

---

## Irritant categories tracked

| Category | Key triggers |
|---|---|
| Histamine | Aged cheese, wine, vinegar, fermented foods, cured meats, tomatoes, avocado, leftovers |
| Mast cell | Alcohol, citrus, strawberries, shellfish, additives, artificial colours, spicy food |
| Gluten | Wheat, barley, rye, spelt, bread, pasta, soy sauce |
| Dairy | Milk, cheese, butter, cream, yogurt, whey, casein |
| FODMAPs | Onion, garlic, beans, lentils, apples, honey, mushrooms, cauliflower |
| Nightshades | Tomatoes, peppers, eggplant, potatoes, paprika, chilli |
| Oxalate | Spinach, almonds, beets, dark chocolate, sweet potato |
| Lectins | Beans, lentils, peanuts, whole grains, soy, cashews |
| Salicylates | Berries, grapes, oranges, herbs, spices, honey, almonds |
| Sulphur | Wine, dried fruit, garlic, onion, eggs, cruciferous veg |
| High fat | Fried food, fatty cuts, full-fat dairy, large amounts of oil |
| Seed oils | Sunflower, canola, soybean, corn oil, most packaged snacks |
| Caffeine | Coffee, black tea, green tea, energy drinks, dark chocolate |
| High sugar | Refined sugar, white bread, sugary drinks, pastries, fruit juice |

---

## Security notes

- The Claude API key is stored in `localStorage` on your device only — it is never committed to this repo and never sent anywhere except Anthropic's API over HTTPS
- The Apps Script URL is public but only accepts structured JSON payloads with strict validation — free text fields are truncated, numbers are range-checked, timestamps are validated
- The Google Sheet should remain public (anyone can call the Apps Script) — do not store sensitive personal data beyond what the app collects
- An `Errors` sheet logs any server-side exceptions for debugging

---

## Roadmap

- [ ] Analysis tab — correlate irritants with symptoms across all meals
- [ ] Weekly report — summary of patterns, top symptom triggers
- [ ] Elimination protocol tracker — mark foods as currently excluded
- [ ] Export to CSV / PDF for sharing with a nutritionist or doctor

---

## Bristol Stool Scale reference

| Type | Description | Meaning |
|---|---|---|
| 1 | Separate hard lumps | Constipated |
| 2 | Lumpy sausage | Constipated |
| 3 | Cracked sausage | Normal |
| 4 | Smooth sausage | Ideal |
| 5 | Soft blobs | Normal |
| 6 | Fluffy, mushy pieces | Loose |
| 7 | Entirely liquid | Diarrhoea |
