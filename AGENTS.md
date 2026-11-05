# Project instructions

## Project overview
- Purpose: static front-end code and content for the QBiC website (Quantitative Biology Center,
  University of Tübingen).
- Hosting model: the live website is hosted in WordPress. This GitHub repository is only a copy of
  the custom code and data that has been uploaded to WordPress; it is edited here and then uploaded/
  synced to WordPress manually. There is no automated deploy from the repository.
- Only the custom code is kept here. WordPress core, theme, plugins, server configuration and the
  remaining site content are NOT in this repository and cannot be inspected from it.
- Main languages / frameworks: HTML, CSS, vanilla JavaScript (ES modules), JSON data. No framework,
  no bundler, no package manager, no build step.
- Important directories:
  - `wp-content/custom-code/pages/<page>/` – one standalone HTML page + script per page
    (`about-us`, `jobs`, `research`, `teams`).
  - `wp-content/custom-code/shared/css/` – `stylesWP.css` is the **authoritative, up-to-date**
    stylesheet (it is what the live WordPress site uses). `styles.css` is the older standalone/local
    copy and may lag behind. Add or change styles in `stylesWP.css` first.
  - `wp-content/custom-code/shared/js/utils.js` – shared email-link handler.
  - `wp-content/custom-code/data/json/` – `teamsData.json`, `papersData.json` (content is data-driven).
  - `wp-content/custom-code/data/images/people/` – member photos as `<Name>.webp`, originals in
    `subdir original/` as `.jpg`; `templateMember.webp` is the fallback.
  - `wp-content/custom-code/data/pdfs/` – job-advert PDFs embedded by `jobs.html`.

## Environment and setup
- Supported runtime versions: browsers only (modern, ES modules + `fetch`). No runtime version is
  pinned in the repository.
- Dependency installation: none for the site itself (Google Fonts and Material Symbols are loaded
  from CDNs in the CSS).
- Python: not used by the site. A local `.venv/` (Python 3.13.18, `numpy`,
  `opencv-python-headless`, `pillow`) existed but was deleted as unused, and `.venv/` is now in
  `.gitignore`. Recreate it only if a one-off script needs it
  (`python3 -m venv .venv && .venv/bin/pip install pillow`); no script in the repository uses it.
- Configuration / environment variables: none in the repository.
- External services: the public website domain is referenced in `teamsScript.js`
  (`https://www.info.qbic.uni-tuebingen.de/#kontakt`); data DOIs/URLs come from `papersData.json`.
- Git remote: `https://github.com/qbicsoftware/QBiC-website.git`, default branch `main`.

## Development workflow
- Tests: none. There is no test framework, no CI config and no test files.
- Linting / formatting: none configured (no ESLint/Prettier/editorconfig). Match the surrounding
  style by hand: 2 spaces in HTML/JSON, 4 spaces in `teamsScript.js`/`utils.js`,
  4 spaces in CSS.
- Build / validation: none. Changes are validated by loading the page in a browser.
- Small test command: serve the repository root over HTTP and open a page, e.g.
  `python3 -m http.server` then `http://localhost:8000/wp-content/custom-code/pages/teams/teams.html`.
  Only reproduces `styles.css`; not the live WordPress rendering.
- Deployment: manual, via WordPress. No CI/CD, no push-to-deploy, nothing is uploaded automatically
  from git. Edit code/data here, then copy the changed snippets/assets into WordPress by hand so that
  the repository and the live site stay in sync. Each page HTML file keeps the final WordPress
  markup/paths in a commented-out `<!-- WP -->` block at the bottom; the active markup above it uses
  relative paths for local testing. Keep both variants in sync when editing
  (commit history: "updated wordpress copy paste").

## Project-specific conventions
- Architecture: every page is a thin HTML skeleton plus one script that fetches its JSON and builds
  the DOM with `document.createElement`; no templating layer.
- Paths: scripts and shared imports use absolute site-root paths
  (`/wp-content/custom-code/...`), so pages must be served from a root that contains `wp-content`.
  Local `<link>`/`<script>` tags use relative paths (`../../shared/css/styles.css`).
- Naming / style:
  - Files: `papersData.json`, `teamsData.json` (camelCase + `Data`); page scripts
    `teamsScript.js`, `aboutUsScript.js` (camelCase + `Script`); HTML files kebab-case
    (`about-us.html`).
  - JS: camelCase functions/variables (`displayTeams`, `filteredPapers`), JSDoc blocks above
    functions, module-level `//` comments explaining non-obvious behaviour.
  - CSS: kebab-case ids/classes (`team-grid`, `member-image`), layout values in `:root` custom
    properties (`--red-primary`, `--gap-between-members`). Many selectors carry the comment
    `/* don't change name without checking js*/` – those names are relied on by the scripts.
  - Stylesheet precedence: `stylesWP.css` wins. A rule only in `styles.css` may not apply on the
    live site, and a rule only in `stylesWP.css` will not show in the local preview – check both.
- Data formats:
  - `teamsData.json`: object keyed by team id (`head`, `biopm`, ...); each team has `name`,
    optional `description`, and `members` keyed `team_member_<n>` with `name`, `title`,
    `image` (basename, no extension), `mail` (base64), optional `phone`, optional `break`
    (grid column start for a forced row break).
  - `papersData.json`: array of `{title, authors[], journal, publisher, doi, url, year, volume,
    issue, pp}`; years currently span 2012–2025.
  - Member images: resolved at runtime by trying `webp`, `jpg`, `jpeg`, `png`, falling back to
    `templateMember.<ext>`.
  - Email addresses are base64-encoded in `data-encoded-mail` and only decoded into a `mailto:` link
    on click (`shared/js/utils.js`) to hinder scraping. Never put plain addresses in the HTML.
- Adding content: adding a paper requires updating `papersData.json` and, if the year is new, the
  hard-coded year list in `research.html` (and its WP copy). New member photos need the `webp` file
  plus, by convention, the original under `images/people/original/`.

## Important constraints
- Files or interfaces that must not change: CSS ids/classes flagged `don't change name without
  checking js` (they are queried by the scripts); the `data-encoded-mail` / `.email-link`
  contract shared by every page.
- Expensive operations requiring approval: converting/resizing image assets, converting the large
  `papersData.json`, and anything that touches the live WordPress site.
- Security / privacy considerations: keep emails base64-encoded in the DOM; do not commit real
  credentials or `.env` files (none exist today); external links use
  `target="_blank" rel="noopener noreferrer"`.
- Known issue (do not "fix" silently): `pages/research/researchScript.js` registers a second
  `download-json` click listener that calls an undefined `downloadJson()`, which throws on click.
  Also, every page's `<title>` is "Research Papers", copied from the research page.
- Not covered anywhere in the repo: how WordPress itself is administered (site is hosted in
  WordPress and updated manually – see Deployment above).
