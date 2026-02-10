# Blood Bowl Card Creator

A free, browser-based tool for creating custom Blood Bowl player cards, special cards, and playbook cards. No installation, no sign-up — just open and start designing.

## Features

### Player Cards
- Full stat customization (MA, ST, AG, PA, AV)
- Roster and Star Player card types
- All six 2025 skill categories (Agility, Devious, General, Mutation, Passing, Strength) with primary/secondary selection
- Custom player image upload with position and scale controls
- Skills, traits, position name, and GP cost fields
- Optional card border overlay

### Special Cards
- Custom timing, duration, and effect sections
- Adjustable font size
- Flexible text layout for house rules and abilities

### Playbook Cards
- Objectives and actions sections
- Background customization
- 59 preloaded play designs

### Export & Persistence
- **PNG export** — print-ready card images (822x1122px)
- **JSON export/import** — save, share, and reload card designs
- **Auto-save** — cards persist in your browser between sessions via localStorage

## Getting Started

### Online

Visit the [GitHub Pages site](https://graylikeme.github.io/bloodbowl-card-creator/) — no setup required.

### Local Development

**Prerequisites:** Ruby with Jekyll (`gem install jekyll bundler`)

```bash
git clone https://github.com/graylikeme/bloodbowl-card-creator.git
cd bloodbowl-card-creator
jekyll serve --livereload
```

Open [http://localhost:4000](http://localhost:4000).

> **Note:** Jekyll is needed to process `_includes/` templates. Without it, you can still run a static server (`python -m http.server 8000`) but the includes won't render.

## Usage

1. Select a card type from the navigation bar (Players, Special Cards, or Plays)
2. Fill in the form fields on the right — the canvas preview updates in real time
3. Upload a player image and adjust position/scale as needed
4. Click **Save as Image** to download a PNG, or **Save as JSON** to export the card data
5. To reload a design, use the **Load from JSON** file picker

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Markup | HTML5 + Jekyll templating |
| Styling | Bootstrap 4 (Bootswatch Darkly/Flatly) |
| JavaScript | Vanilla ES6 + jQuery 3.x |
| Rendering | HTML5 Canvas API |
| Fonts | 6 custom font families (Brothers, Franklin Gothic, Built Titling, Bank Gothic, Frutiger, Indie Flower) |
| Hosting | GitHub Pages |

No build pipeline, no npm, no webpack, no TypeScript. Just HTML, CSS, and JS.

## Project Structure

```
├── index.html              # Player cards page
├── special.html            # Special cards page
├── playbook.html           # Playbook cards page
├── instructions.html       # Usage instructions
├── _includes/
│   ├── tabs/               # Tab content templates
│   ├── card/               # Player card form sections
│   ├── special/            # Special card form sections
│   └── plays/              # Playbook card form sections
├── assets/
│   ├── js/
│   │   ├── card.js         # Player card rendering (~950 lines)
│   │   ├── special.js      # Special card rendering (~790 lines)
│   │   └── plays.js        # Playbook card rendering (~990 lines)
│   ├── css/main.css        # Font-face declarations + custom styles
│   ├── img/                # Card frames, backgrounds, stat numbers, 600+ team logos
│   ├── fonts/              # 28 font files across 6 families
│   └── vendors/            # jQuery, Bootstrap, Popper.js
└── assets/plays.json       # 59 preloaded playbook designs
```

## How It Works

Each card type has its own JS file following the same architecture:

```
User Input (forms) → onAnyChange() → readControls() → data object → render() → Canvas
                                                                                   ↓
                                                                            saveLatestFighterData() → localStorage
```

On page load, `loadLatestFighterData()` restores the last card from localStorage.

The canvas renders in layers: background texture, player image, card frame, text elements, stat numbers, and optional border overlay.

## Contributing

Contributions are welcome! The codebase is intentionally simple — vanilla JS with no framework — to keep the barrier to entry low.

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test manually across browsers (Chrome, Firefox, Safari)
5. Submit a pull request

See [CLAUDE.md](CLAUDE.md) for detailed architecture documentation and guides for common changes (adding form fields, skill categories, assets, etc.).

## License

This project is open source. See the repository for license details.
