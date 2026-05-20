# Duraclean
Cleaning solutions 

## Website files

Use `index.html` for the GitHub Pages website, `styles.css` for styling, and the `assets/` folder for local images. Use `all-in-one.html` when you need a self-contained demo file without separate CSS/assets.

## Single-file version

Open [`all-in-one.html`](all-in-one.html) for a self-contained version with embedded CSS, admin studio, Supabase guide, and before/after assets.

## GitHub Pages compatibility

The website uses relative asset paths (for example `assets/...`) so it works when deployed from a GitHub Pages repository subpath. The `.nojekyll` file is included to make GitHub Pages serve static files exactly as packaged.


## If you see a `diff --git` page instead of the website

That means you are viewing the pull request diff or a patch page, not the deployed GitHub Pages site. After merging, open the Pages URL from **Settings → Pages** or from the **Deploy static site to GitHub Pages** workflow summary. The deployed website entry file is `index.html`; do not copy the PR diff text into the browser as the site.

This repository includes `.github/workflows/pages.yml`, which publishes the static files to GitHub Pages automatically after pushes to `main` or `master`.
