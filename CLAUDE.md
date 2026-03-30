# CLAUDE.md

This file provides guidance for Claude Code when working with the Meal-planner repository.

## Project Overview

**Gates Family Meal Planner** — a self-contained, single-file web application for weekly family meal planning. There is no build system, no backend, no dependencies to install. The entire application lives in `index.html`.

## Running the App

Open `index.html` directly in a browser, or serve it over HTTP:

```bash
python -m http.server 8080
# then open http://localhost:8080
```

No installation, compilation, or bundling required.

## Repository Structure

```
Meal-planner/
├── index.html    # Entire application (HTML + CSS + JS, ~21 KB)
└── README.md     # Minimal title-only README
```

All HTML, CSS, and JavaScript are embedded in `index.html`. There are no separate source files, no `node_modules`, no build artifacts.

## Architecture

**Client-side only.** No server, no API, no external services at runtime.

**Data persistence:** Browser `localStorage` under the key `gatesMealPlanner`. Falls back gracefully (displays a warning banner) if localStorage is unavailable.

**In-memory state:** A single global `data` object:

```js
{
  weeks: [
    {
      startDate: "YYYY-MM-DD",  // ISO date of Monday
      days: [                    // 7 entries, Mon–Sun
        {
          breakfast: { who: [], notes: "" },
          lunch:     { who: [], notes: "" },
          dinner:    { title: "", who: [], protein: "", carbs: "", veg: "" }
        }
      ]
    }
  ]
}
```

**Rendering:** `render()` rebuilds the entire `#weeksContainer` innerHTML on every change. There is no virtual DOM or reactive framework — it is plain string interpolation via template literals.

**Auto-save:** `debouncedSave()` (800 ms debounce) calls `saveData()` which writes to localStorage. A brief toast notification confirms each save.

## Code Layout Inside `index.html`

The JavaScript is organized into clearly labelled sections (search for `// ──`):

| Section | Functions |
|---|---|
| DATA PERSISTENCE | `loadData`, `saveData`, `debouncedSave`, `showToast`, `showStorageWarning` |
| DATE HELPERS | `nextMonday`, `dateToISO`, `isoToDate`, `formatDisplay`, `addDays` |
| DATA STRUCTURE BUILDERS | `blankDay`, `blankWeek`, `initFirstWeek` |
| WEEK MANAGEMENT | `addWeek`, `confirmRemoveFirst`, `closeModal`, `removeFirstWeek` |
| DATE EDITING | `startDateEdit`, `finishDateEdit` |
| FIELD UPDATE HELPERS | `updateField`, `toggleWho` |
| RENDER | `render`, `renderWeek`, `renderDay`, `renderSimpleMeal`, `renderDinner`, `escHtml` |
| EXPORT / IMPORT | `exportData`, `importData` |

## Key Constants (hardcoded)

```js
const PEOPLE   = ['Ian', 'Cristina', 'Maya'];
const DAY_NAMES = ['Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday'];
const MONTHS   = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
```

`PEOPLE` drives all "who's eating" checkboxes. Adding a family member means adding a name here and re-rendering.

## CSS Architecture

- CSS custom properties (variables) defined in `:root` — always use these for colours, not hard-coded hex values.
- CSS is co-located with HTML in the `<style>` tag, organized by component with `/* ── SECTION ── */` comments.
- Responsive breakpoint: `@media (max-width: 700px)`.
- Typography: **Playfair Display** (headings) and **Nunito** (body), loaded from Google Fonts.

Key CSS variables:

| Variable | Usage |
|---|---|
| `--green` / `--green-light` | Primary brand colour (header, buttons, accents) |
| `--amber` / `--amber-light` | Secondary accent (dinner cards) |
| `--cream` / `--parchment` | Page/card backgrounds |
| `--text-dark` / `--text-mid` / `--text-light` | Text hierarchy |

## Coding Conventions

- **JavaScript:** ES6+, functional style (no classes). Arrow functions for callbacks. `const`/`let`, never `var`.
- **Naming:** `camelCase` for functions and variables; `UPPER_SNAKE_CASE` for module-level constants; `kebab-case` for CSS classes.
- **HTML generation:** Template literals returning HTML strings — keep them readable with consistent 2-space indentation inside the template.
- **Event handlers:** Inline `onclick`/`onchange`/`oninput` attributes passing indices (e.g., `onclick="addWeek()"`, `oninput="updateField(${wi},${di},'dinner','title',this.value)"`).
- **Security:** Always run user-supplied strings through `escHtml()` before interpolating into HTML to prevent XSS.
- **No external dependencies** should be added at runtime beyond the existing Google Fonts link.

## Testing

There is no automated test suite. Manual testing steps:

1. Open `index.html` in a browser.
2. Add/remove weeks; verify dates increment correctly.
3. Check/uncheck "who" attendance; verify persistence after page reload.
4. Export data to JSON, clear localStorage, import the JSON — verify data is restored.
5. Test on a narrow viewport (≤ 700 px) for responsive layout.

## Common Tasks

**Add a new family member**
1. Add the name to the `PEOPLE` array (line ~367).
2. All checkboxes and data structures are generated dynamically — no other changes needed.

**Change the default week start day**
- Modify `nextMonday()` in the DATE HELPERS section.

**Export/Import data format**
- JSON matching the `data` object schema above.
- Export filename: `meal-plan-YYYY-MM-DD.json`.

**LocalStorage key**
- `gatesMealPlanner` — change this constant in `loadData`/`saveData` if renaming the app.

## Git Workflow

- Development branch: `claude/create-claude-md-mjUwS`
- Push with: `git push -u origin claude/create-claude-md-mjUwS`
- There are no pre-commit hooks or linters configured.
