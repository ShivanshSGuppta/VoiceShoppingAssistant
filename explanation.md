# Explanation (Detailed)

This document explains **how the solution is structured**, **how voice + NLP works**, and how each required feature is implemented.

## 1) High-level architecture

Everything is client-side for a fast assessment demo.

- **UI layer (React)**
  - `VoiceBar`: microphone control + live transcript + manual fallback input.
  - `ShoppingList`: categorized list with quantity controls and “purchased” toggles.
  - `SearchPanel`: voice-activated search + filters + availability toggles.
  - `SuggestionsPanel`: history suggestions + seasonal suggestions + substitutes.
  - `ActionLog` + `Toasts`: real-time feedback.

- **Domain layer (pure functions)**
  - `utils/nlp.ts`: rule-based intent recognition + entity extraction.
  - `utils/history.ts`: purchase-history capture + suggestion generation.
  - `data/*`: catalog, categories, seasonal mapping, substitute mapping.

- **Persistence**
  - `utils/storage.ts`: localStorage read/write helpers.
  - Keys:
    - `svca:list:v1`: shopping list items
    - `svca:history:v1`: purchase timestamps per item
    - `svca:availability:v1`: product availability overrides
    - `svca:lang:v1`: selected speech recognition language

Why this design?
- Keeps the code **clean and testable**: NLP/history logic is isolated from UI.
- Meets the 8-hour constraint: no backend required, but the design still resembles a typical separation of concerns.

## 2) Data model

### Shopping list item

`src/components/ShoppingList.tsx`

```ts
export type ListItem = {
  id: string
  name: string
  normalized: string
  qty: number
  category: Category
  addedAt: number
  updatedAt: number
  purchasedAt?: number
}
```

Notes:
- `normalized` is a sanitized lowercase version used for matching.
- `category` is computed from keywords (`data/categories.ts`).
- `purchasedAt` enables “history-driven” suggestions.

### Purchase history

`src/utils/history.ts`

```ts
export type HistoryStore = Record<string, number[]> // normalized -> timestamps
```

This is enough to compute a repurchase interval using simple statistics.

### Catalog & availability overrides

`src/data/productCatalog.ts`

- `CATALOG` is a static dataset suitable for a demo.
- Availability can be toggled at runtime:

```ts
export type AvailabilityOverrides = Record<string, boolean>
```

This allows substitutes to be demonstrated reliably.

## 3) Voice recognition

`src/components/VoiceBar.tsx`

- Uses `window.SpeechRecognition || window.webkitSpeechRecognition`.
- Runs in **continuous** mode with **interim results**.
- When a final transcript arrives, it calls `onUtterance(finalText)`.

Fallback:
- If SpeechRecognition is unavailable, the UI raises a compatibility toast and you can use typed commands.

Key UX behaviors:
- Shows **Listening / Idle** status.
- Shows interim transcript live.
- Displays “Last command” as final transcript.

## 4) NLP / command understanding

`src/utils/nlp.ts`

This is a **rule-based intent parser** (lightweight NLP) that covers common shopping assistant actions.

### Supported intents

- `ADD_ITEM`: “Add milk”, “I need apples”, “Buy bananas”, “Add 2 bottles of water”
- `REMOVE_ITEM`: “Remove milk”, “Delete bread”, “Drop eggs”
- `UPDATE_QTY`: “Change oranges to 5”, “Set milk to 2”
- `SEARCH`: “Find organic apples under $5”, “Search toothpaste by Colgate under $5”
- `CLEAR`: “Clear list”, “Empty list”
- `HELP`: “Help”, “Commands”

### Multilingual support

The SpeechRecognition language is user-selectable.

The intent parser contains a minimal verb dictionary for:
- English (`en-*`)
- Spanish (`es-*`)
- Hindi (`hi-*`, Devanagari + romanization)

Example Hindi phrases (supported as demo patterns):
- “मुझे चाहिए दूध” (add)
- “दूध हटाओ” (remove)

### Quantity extraction

`src/utils/numbers.ts`

Rules:
- Prefer digits: `\b(\d{1,3})\b`
- Otherwise match common word numbers (English + basic Spanish):
  - “two”, “three”, …
  - “dos”, “tres”, …

### Search constraint extraction

For SEARCH intent:
- Price: `under|below|less than $5` → `maxPriceUsd`
- Brand: `brand Colgate` or `by Colgate` → `brand`
- Tags: “organic”, “herbal”, “vegan”, “gluten free” → `tags`

This design is intentionally deterministic and transparent (good for interviews). If this were production, you could replace `parseUtterance()` with a small on-device model or a hosted LLM.

## 5) Shopping list management

Implemented in `App.tsx` + `ShoppingList.tsx`:

- **Add / remove / update qty**
  - Via voice intents or UI buttons.
- **Categorization**
  - `categorize(normalizedName)` uses keyword mapping.
- **Quantity management**
  - Extracted from voice, editable via UI `+ / −`.
- **Basic error handling**
  - When an item can’t be found to remove/update, a toast appears.

## 6) Smart suggestions

### A) History-based “running low”

When the user marks an item as purchased:
- Append timestamp into `HistoryStore[item]`.
- Compute average days between purchases (`avgIntervalDays`).
- Suggest items not currently on the list when:
  - `daysSinceLastPurchase >= avgInterval * 1.2`

This mimics a real “replenishment” model in a simple, explainable way.

### B) Seasonal suggestions

`src/data/seasonal.ts`

A month→items mapping emits suggestions like:
- “Oranges (citrus season)”
- “Spinach (winter greens)”

In production, this could be replaced by:
- A geo-aware seasonality provider
- Retail promotion feeds

### C) Substitutes

- The catalog includes a “Mark unavailable” toggle.
- If an unavailable product matches the user’s add/search term:
  - The app proposes substitutes from `data/substitutes.ts`.

Example:
- If “Whole Milk” is unavailable, suggest “Almond milk / oat milk / lactose free milk”.

## 7) Voice-activated search

- SEARCH intent populates `lastSearch`.
- `SearchPanel` reads `lastSearch` and immediately applies filters:
  - term
  - brand
  - maxPriceUsd
  - tags

Users can also manually use filters without voice.

## 8) UI/UX & feedback

- **Toasts** provide confirmations + errors.
- **ActionLog** shows real-time interpreted actions (good for demos).
- **Loading state**: “Updating…” pill (short simulated delay) after list/history changes.
- Mobile responsive layout (single column below ~980px).

## 9) Known limitations (acceptable for assessment)

- SpeechRecognition availability varies by browser/OS.
- NLP is rule-based, not a statistical model (by design for explainability).
- Seasonal and pricing signals are simulated (static mapping + catalog dataset).

## 10) If you want to extend this into production

- Replace rule-based NLP with:
  - on-device intent classifier OR
  - a small hosted model (plus privacy review)
- Replace demo catalog with a real product search backend.
- Add user profiles + preference learning.
- Add analytics + A/B testing for suggestion quality.

---

**Where to look in code**

- Speech + UI wiring: `src/components/VoiceBar.tsx`, `src/App.tsx`
- NLP: `src/utils/nlp.ts`, `src/utils/numbers.ts`
- Suggestions: `src/utils/history.ts`, `src/data/seasonal.ts`, `src/data/substitutes.ts`
- Search: `src/components/SearchPanel.tsx`, `src/data/productCatalog.ts`
