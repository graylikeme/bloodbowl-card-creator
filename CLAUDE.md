# CLAUDE.md — Blood Bowl Card Creator

## Project Overview

A client-side web application for creating custom Blood Bowl tabletop game cards. Users can design player cards, special cards, and playbook cards with full stat customization, image uploads, and PNG/JSON export. No backend — everything runs in the browser.

**Live site:** Hosted on GitHub Pages (auto-deploys from `main` branch).

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Markup | HTML5 with Jekyll templating (`_includes/`) |
| Styling | Bootstrap 4 (Bootswatch Darkly/Flatly themes), custom CSS |
| JavaScript | Vanilla ES6 + jQuery 3.x (no framework, no modules) |
| Rendering | HTML5 Canvas API (822x1122px cards, 1122x822px playbooks) |
| Storage | Browser `localStorage` for auto-save; JSON file export/import |
| Fonts | 6 custom font families in `assets/fonts/` |
| Build | Jekyll static site generator (no webpack, no npm) |
| Hosting | GitHub Pages with automatic Jekyll builds |

## Repository Structure

```
bloodbowl-card-creator/
├── index.html                  # Player cards page
├── special.html                # Special cards page
├── playbook.html               # Playbook cards page
├── instructions.html           # User instructions page
├── _includes/
│   ├── tabs/                   # Tab content templates (4 files)
│   ├── card/                   # Player card form sections
│   ├── special/                # Special card form sections
│   └── plays/                  # Playbook card form sections
├── assets/
│   ├── js/
│   │   ├── card.js             # Player card logic (~950 lines)
│   │   ├── special.js          # Special card logic (~790 lines)
│   │   └── plays.js            # Playbook card logic (~990 lines)
│   ├── css/
│   │   └── main.css            # Font-face declarations, theme imports, custom styles
│   ├── img/
│   │   ├── card/               # Player card frames, backgrounds, stat number images
│   │   ├── special/            # Special card assets
│   │   ├── plays/              # Playbook card assets
│   │   └── logos/              # 600+ team logos organized by team name
│   ├── fonts/                  # 28 font files (6 families, multiple formats)
│   ├── vendors/                # jQuery, Bootstrap, Popper.js, Bootswatch CSS
│   └── favicons/
├── memory-bank/                # Project documentation (context files)
└── assets/plays.json           # 59 preloaded playbook designs
```

## Development Setup

### Prerequisites

- Ruby with Jekyll: `gem install jekyll bundler`
- Or any static file server (but Jekyll includes won't process without Jekyll)

### Running Locally

```bash
# Recommended: Jekyll with live reload
jekyll serve --livereload
# Visit http://localhost:4000

# Alternative (includes won't be processed):
python -m http.server 8000
```

### No Build Pipeline

There is no `package.json`, no npm scripts, no webpack, no TypeScript. JavaScript is unminified vanilla ES6. CSS is hand-written (no Sass/LESS). Jekyll handles HTML templating only.

## Architecture & Key Patterns

### Data Flow

```
User Input (forms) → onAnyChange() → readControls() → data object
                                                          ↓
                                                      render() → Canvas
                                                          ↓
                                              saveLatestFighterData() → localStorage
```

On page load: `loadLatestFighterData()` → `writeControls()` → restores form state.

### Canvas Rendering Pipeline (Layer Order)

1. Background texture (`blank.png`)
2. Player image (with offset/scale transforms)
3. Card frame (`bloodbowl_frame.png` or star player variant)
4. Text elements (names, stats, skills — dynamic font sizing, word wrapping)
5. Stat number images (from `assets/img/card/numbers/`)
6. Border overlay (optional, `bloodbowl_border.png`)

### Three JS Files, Parallel Structure

Each card type (`card.js`, `special.js`, `plays.js`) follows the same pattern:

- `readControls()` — reads form DOM into a data object
- `writeControls()` — populates form DOM from a data object
- `render()` — draws the data object onto the canvas
- `defaultFighterData()` — returns a blank card data object
- `onAnyChange()` — event handler that triggers read → render → save
- `saveFighterData()` / `loadFighterData()` — JSON export/import
- `saveCardAsImage()` — PNG export via `canvas.toDataURL()`

### Player Types (card.js)

- **Roster**: Standard frame, shows position name and primary/secondary skill categories
- **Star Player**: Special frame (`bloodbowl_specialplayer_frame.png`), shows "Plays For" and "Special Rules" fields instead of position/skills

Conditional logic lives in `drawCardFrame()`:
```javascript
playerType = document.getElementById("playerType").value;
if (playerType == "star") { /* star-specific rendering */ }
else { /* roster rendering */ }
```

### Skill Category System

Six categories: **A** (Agility), **D** (Devious), **G** (General), **M** (Mutation), **P** (Passing), **S** (Strength).

Each has primary (`p_`) and secondary (`s_`) checkboxes. The comma-separated display string is built in `drawCardFrame()` (alphabetical order), then passed to `drawDevelopment(primary, secondary)` for canvas rendering.

### Storage Mechanisms

1. **localStorage** — auto-saves on every change (key: `latestCardName` + map: `fighterDataMap`)
2. **JSON export** — manual download with base64-encoded images
3. **JSON import** — FileReader API, converts base64 back to blob URLs

## How to Make Common Changes

### Adding a New Form Field

1. Add HTML input in `_includes/card/section-characteristics.html` with `oninput="onAnyChange()"`
2. Read it in `readControls()`: `data.newField = document.getElementById("newField").value;`
3. Write it in `writeControls()`: `document.getElementById("newField").value = fighterData.newField;`
4. Set default in `defaultFighterData()`: `fighterData.newField = "";`
5. Render it in `drawCardFrame()` or a new draw function

### Adding a New Skill Category

Follow the pattern from the "Devious" category addition:

1. Add primary + secondary checkboxes in `section-characteristics.html`
2. Read checkbox state in `readControls()`: `data.p_x = document.getElementById("p_x").checked;`
3. Write checkbox state in `writeControls()`
4. Append to comma-separated string in `drawCardFrame()` (maintain alphabetical order)
5. Initialize as `false` in `defaultFighterData()`

**Important:** String construction happens in `drawCardFrame()`, NOT in `drawDevelopment()`. The latter only renders pre-built strings.

### Adding New Assets

```bash
# Team logo
cp logo.png assets/img/logos/TeamName_01.png

# Card background
cp bg.png assets/img/card/new_background.png

# Fonts — add @font-face in assets/css/main.css and hidden div in HTML for preloading
```

## Code Conventions

- **No module system** — all JS is in global scope
- **Mixed DOM access** — jQuery (`$("#id")`) for some operations, vanilla `document.getElementById()` for others; both patterns are acceptable
- **Event binding** — inline `oninput="onAnyChange()"` on HTML elements
- **Canvas drawing** — direct `ctx.fillText()`, `ctx.drawImage()` calls; no abstraction layer
- **Text sizing** — dynamically reduced font size when text is too long for available width
- **Image transforms** — `offsetX`, `offsetY`, `scalePercent` properties on the data object
- **No semicolons on some lines** — code style is inconsistent; match surrounding code when editing

## Testing

There is no automated test suite. Testing is manual visual inspection:

1. Create a card with all fields populated
2. Create a card with minimal fields
3. Test both roster and star player types
4. Save/load via JSON round-trip
5. Export PNG and verify quality
6. Test with long text strings (triggers dynamic font sizing)
7. Test all skill category checkbox combinations
8. Upload various image sizes
9. Check in Chrome, Firefox, and Safari

## Known Issues

- **HTML typo**: `_includes/card/section-characteristics.html` has a double-quote on the `p_general` div: `<div class="col-sm-1"">` — cosmetic, does not affect rendering
- **Unused file**: `_includes/card/section-deployment.html` exists but is not included anywhere
- **Mobile**: Touch interactions are not optimized
- **No undo/redo**: Single card state only
- **No batch operations**: One card at a time

## Deployment

Push to `main` branch. GitHub Actions runs Jekyll build automatically and deploys to GitHub Pages. No manual deployment steps required.
