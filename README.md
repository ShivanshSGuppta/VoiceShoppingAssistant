# Shopping Voice Assistant

A voice-first shopping list app built for a placement assessment.

This project focuses on practical voice interactions: speak naturally, let the app interpret intent, and manage your list with minimal typing. It also demonstrates suggestion logic, product search, and graceful fallback when voice APIs are not available.

## What this project demonstrates

- Voice command handling with the Web Speech API (`SpeechRecognition` / `webkitSpeechRecognition`)
- Intent parsing for add, remove, update quantity, search, clear, and help
- Multi-language speech input support through language selection (English/Hindi/Spanish tuned in parser)
- Smart suggestions from purchase history + month-based seasonal recommendations
- Substitute recommendation flow when an item is marked unavailable
- Local-first persistence using `localStorage` (no backend required)
- Real-time UX feedback via action logs, toasts, and loading indicators

## Core features

### 1. Voice + text commands
- Click `Listen`, speak your command, and get live transcript feedback.
- If voice is unsupported or blocked, use the text input fallback.
- Example commands:
  - `Add 2 bottles of water`
  - `Remove milk`
  - `Change oranges to 5`
  - `Find organic apples under $5`

### 2. Shopping list management
- Auto-categorizes items (Produce, Dairy, Pantry, etc.)
- Increment/decrement quantity directly from the list
- Mark purchased / unmark purchased
- Remove items instantly

### 3. Smart suggestions
- **History-based**: tracks purchase timestamps and suggests “running low” items based on average repurchase interval
- **Seasonal**: month-based quick picks
- **Substitutes**: if a product is unavailable, suggests alternatives (e.g., milk -> almond milk)

### 4. Voice-activated search panel
- Supports filters from voice intent:
  - term
  - brand (`brand X` / `by X`)
  - max price (`under $5`)
  - tags (organic, herbal, vegan, gluten free, whole wheat)
- Also supports manual filters from UI
- Allows toggling product availability to test substitute behavior

## Tech stack

- React 18 + TypeScript
- Vite 5
- ESLint 9
- Browser Web Speech API
- `localStorage` for persistence

## Run locally

```bash
npm install
npm run dev
```

Then open the Vite URL (usually `http://localhost:5173`) and allow microphone access.

## Build for production

```bash
npm run build
npm run preview
```

Deploy the generated `dist/` folder to any static host (Firebase Hosting, Netlify, Vercel, etc.).

## Project structure

```text
src/
  components/      # Voice bar, list, suggestions, search, toasts, action log
  data/            # Catalog, categories, substitutes, seasonal mappings
  utils/           # NLP parsing, quantity extraction, history, storage, string helpers
  App.tsx          # Main orchestration and state
```

## Known limitations

- Voice recognition depends on browser support and mic permissions.
- NLP is rule-based (intentionally lightweight), not an ML model.
- Data is local to the browser session/profile (no sync across devices).
- Seasonal and substitute logic are demo datasets by design.

## Why this is assessment-ready

The app highlights end-to-end product thinking: input handling, intent parsing, state management, UX feedback, and edge-case fallbacks. It is compact, understandable, and easy to demo live.
