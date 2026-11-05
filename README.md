# QBiC-website

Custom front-end code and content for the QBiC website (Quantitative Biology Center, University of
Tübingen).

## What this repository is

The live website is hosted in **WordPress**. This repository is not a deployable site and not the
source of truth for WordPress itself: it is a GitHub copy of the custom code and data that has been
uploaded to WordPress. The workflow is:

1. edit the code/data here,
2. copy the changed files or snippets into WordPress (manually),
3. keep the repository and the live site in sync.

WordPress core, the theme, the plugins and the remaining page content are not part of this
repository.

## Contents

- `wp-content/custom-code/pages/<page>/` – one standalone HTML page plus its script per page
  (`about-us`, `jobs`, `research`, `teams`). Each file contains the final WordPress markup in a
  commented-out `<!-- WP -->` block at the bottom; the active markup uses relative paths for local
  testing.
- `wp-content/custom-code/shared/css/` – `styles.css` for the standalone pages, `stylesWP.css` for
  the WordPress variant.
- `wp-content/custom-code/shared/js/utils.js` – shared email-link handler (emails are base64-encoded
  in `data-encoded-mail` and decoded on click).
- `wp-content/custom-code/data/` – JSON content (`json/teamsData.json`, `json/papersData.json`),
  member images (`images/people/`) and PDFs (`pdfs/`).

## Local preview

There is no build step and no dependencies to install. Because the scripts reference absolute
`/wp-content/...` paths, serve the repository root over HTTP rather than opening the files directly:

```
python3 -m http.server
```

Then open e.g. `http://localhost:8000/wp-content/custom-code/pages/teams/teams.html`.

## Tests and linting

None configured.

See `AGENTS.md` for conventions, data formats and constraints.
