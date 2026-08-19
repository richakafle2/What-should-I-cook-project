# What Should I Cook? 🍳

A lightweight Flask web app that helps indecisive cooks pick a recipe. Users take a short quiz — pick a cuisine, a cook time, or just ask for a random surprise — and the app returns a curated list of matching recipes, each linking out to the original recipe page.

## How it works

The app ships with a hard-coded catalog of **74 recipes**, spanning 6 cuisines and 4 time ranges. On the quiz page, the user can:

- Pick a **cuisine** (Italian, Japanese, Chinese, Indian, American, or Mexican), and/or
- Pick a **cook time** (0–30 min, 30 min–1 hr, 1–2 hrs, or 2+ hrs), or
- Skip both and hit **"Chef's Choices"** for 3 random recipes instead.

The server matches the selected filters (or combination of both) against the recipe catalog and renders a results page listing the matching recipe names as clickable links to the original source (AllRecipes, Food Network, Taste of Home, Just One Cookbook, Serious Eats, etc.).

## Project structure

```
What-should-I-cook-project-master/
├── web.py                  # Flask app: routes, recipe data, and matching logic
├── templates/               # Jinja2 HTML templates
│   ├── layout.html          #   Base layout (page shell, stylesheet link, HOME button)
│   ├── home.html            #   Landing page ("/") — title + link to the quiz
│   ├── quizpage.html        #   Quiz form ("/quizpage") — cuisine, time, chef's choice
│   └── results.html         #   Results page ("/results") — renders matched recipes
├── static/
│   ├── style.css            # All page styling (colors, layout, buttons, form)
│   ├── home.png             # Background/hero image for the home page
│   ├── example.png          # Background/hero image for the quiz page
│   └── results.png          # Background/hero image for the results page
├── requirements.txt          # Pinned Python/pip dependencies
├── Pipfile                   # Pipenv config (pins Python 3.8.3)
├── Procfile                  # Heroku-style process file (`gunicorn web:app`)
├── .replit                   # Legacy Replit run configuration
├── main.sh                   # Leftover Replit boilerplate script (not used by the app)
└── .gitignore                # Ignores node_modules/ and package-lock.json
```

### File-by-file notes

**`web.py`** — the entire application lives in this one file:
- `name_list`, `Link_list`, `time_list`, `cuisine_list` — four parallel lists (74 entries each) that make up the recipe catalog. Each index `i` across all four lists describes one recipe: its display name, its source URL, a time-bucket code (`ta`/`tb`/`tc`/`td`), and a cuisine code (`ca`–`cf`).
- `find_recipe(cuisine, cook_time, chefs_choice)` — the matching logic:
  - If `chefs_choice` is set, it shuffles the indices and returns 3 random recipes, ignoring the other filters.
  - Otherwise, it scans the catalog and collects recipes where the cuisine matches, the time matches, or both match (whichever filters were actually selected).
- Three Flask routes:
  - `GET /` → renders `home.html`
  - `GET /quizpage` → renders `quizpage.html`
  - `GET/POST /results` → reads the submitted form fields (`cuisine`, `cook_time`, `random`), calls `find_recipe`, and renders `results.html` with the matches (or a "No recipes found." message if nothing was selected).

**`templates/layout.html`** — shared page frame that every other template extends via `{% block content %}`. Links the stylesheet and renders a persistent HOME button.

**`templates/home.html`** — simple landing page with the app title and a button that navigates to `/quizpage`.

**`templates/quizpage.html`** — the quiz form. Radio buttons for cuisine (`name="cuisine"`), cook time (`name="cook_time"`), and a "Chef's Choices" option (`name="random"`), submitted via POST to `/results`.

**`templates/results.html`** — loops over the `results`/`recipe_name` lists passed in from Flask and renders each as a numbered, clickable link (opens in a new tab). Shows a "Back to Quiz" button.

**`static/style.css`** — all visual styling: antique-white color scheme, full-bleed background images per page, button and form styling, and a two-column flex layout on the quiz page (form on the left, instructions on the right).

**`requirements.txt`** — pinned dependencies for a `pip install` based setup (older Flask/Werkzeug/Jinja2 versions, plus `gunicorn` for production serving).

**`Pipfile`** — for anyone using `pipenv` instead of raw `pip`; pins Python `3.8.3`.

**`Procfile`** — tells a Heroku-style host to run the app with `gunicorn web:app`.

**`.replit`** and **`main.sh`** — leftover configuration from when this project was scaffolded/hosted on Replit. `main.sh` just echoes `"Hello World"` and `.replit` doesn't actually launch `web.py` — neither is needed to run the app locally or on Heroku; they can be updated or removed for other deployment targets.

## Requirements

- Python 3.8+ (the project was built against 3.8.3; newer 3.x should also work)
- pip (or pipenv, if you prefer to use the included `Pipfile`)

## Setup

1. Clone or download the project, then move into it:
   ```bash
   cd What-should-I-cook-project-master
   ```
2. (Recommended) create and activate a virtual environment:
   ```bash
   python3 -m venv venv
   source venv/bin/activate      # on Windows: venv\Scripts\activate
   ```
3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Running the app

**Local development** (Flask's built-in server, with debug reload):
```bash
python web.py
```
This starts the server on `http://0.0.0.0:5000`. Open **http://localhost:5000** in your browser.

**Production-style serving** (matches the `Procfile`, useful for Heroku or any gunicorn-compatible host):
```bash
gunicorn web:app --log-file=-
```

> Note: `requirements.txt` pins quite old versions of Flask/Jinja2/Werkzeug (2020-era). If you hit install errors on a modern Python version, either use a Python 3.8 environment to match the original pins, or relax the version pins in `requirements.txt` and re-test the app (the code itself uses only basic Flask/Jinja2 features, so newer versions should work fine).

## Using the web app

1. Open the home page and click **Recipe Quiz**.
2. On the quiz page:
   - Select a cuisine, a cook time, both, or neither.
   - Or select **Chef's Choices** to skip the filters entirely and get 3 random picks.
   - Click **Submit**.
3. The results page lists every matching recipe as a clickable link — clicking one opens the original recipe source in a new tab.
4. Click **Back to Quiz** (or **HOME**) to try again with different answers.

If no filters and no "Chef's Choices" option are selected, the results page shows **"No recipes found."**

## Known limitations

- The recipe catalog is hard-coded directly in `web.py` rather than loaded from a database or config file, so adding/editing recipes means editing the source lists directly (and keeping all four lists in sync by index).
- The `id` attributes on the quiz page's radio inputs aren't all unique (several share the same `id` across different `<label for="...">` targets), which is a minor HTML/accessibility issue but doesn't affect functionality since the form logic relies on `name`/`value`, not `id`.
- `.replit` and `main.sh` are unused leftovers from earlier Replit-based hosting and can be safely ignored, updated, or removed depending on where you deploy.
- Pinned dependency versions in `requirements.txt` are several years old; consider updating them (and re-testing) before using this in a new environment.

## Tech stack

- **Backend:** Python, Flask
- **Templating:** Jinja2
- **Frontend:** Plain HTML/CSS (no JS framework)
- **Production server:** Gunicorn
- **Recipe sources:** AllRecipes, Food Network, Taste of Home, Just One Cookbook, Serious Eats, RecipeTin Eats, The Woks of Life, and other public recipe sites (linked, not hosted, by this app)
