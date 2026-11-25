# ColdIgniter User Guide

The user guide of [ColdIgniter](https://github.com/ColdIgniter/coldigniter), a CFML port of CodeIgniter 4.

The pages are Markdown files in `guide/`. [Zensical](https://zensical.org/) builds them into a static site, and a Cloudflare Worker with static assets only serves the site at `https://coldigniter.dev/user_guide/`.

## Files

| File | What it is |
|---|---|
| `guide/` | The pages, in the same folders and with the same file names as the CodeIgniter 4 guide. |
| `guide/images/` | The logo (`logo.svg`, and `logo-dark.svg` for the dark colours) and the favicon. |
| `guide/stylesheets/extra.css` | The colours and the look of the header and the menu. |
| `mkdocs.yml` | The settings of the site and the menu. |
| `requirements.txt` | The version of the generator. |
| `wrangler.jsonc` | The settings of the Worker. |

## Build

```sh
pip install -r requirements.txt
zensical build --clean
```

The site is written to `dist/user_guide/`. Every page is a file with its own address, as in the CodeIgniter guide: `installation/index.html`, `general/controllers.html`. The tracked file `dist/_redirects` sends a folder address to its `index.html`; add two lines there for every new folder that has an `index.md`. The folder `user_guide/` is there on purpose: the Worker serves a file at the same path as it has in `dist/`.

## Look at it on your own machine

```sh
zensical serve
```

To run it as the Worker does:

```sh
npx wrangler@4 dev
```

Then open `http://localhost:8787/user_guide/`.

## Deploy

The workflow `.github/workflows/user_guide.yml` builds the guide for every pull request. On `trunk` a second job also deploys, when the repository variable `USER_GUIDE_DEPLOY` is `true`. That job runs in the GitHub environment `cloudflare`, which holds the secrets `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID`.

By hand:

```sh
npx wrangler@4 deploy
```

## The pages

The menu in `mkdocs.yml` follows the table of contents of the CodeIgniter 4 guide: the same sections, in the same order, with the same page titles and file paths. So `incoming/routing` in their guide is `incoming/routing.html` here. "CodeIgniter" is "ColdIgniter" in the titles.

A page that is not written yet is a stub: the title, one sentence and the note "This page is not written yet.". A page for a part of CodeIgniter 4 that is not ported yet says "Not ported yet". Left out on purpose: the change logs and upgrade pages of single CodeIgniter versions (the page "Change logs" points to `CHANGELOG.md` of ColdIgniter), and pages about Composer.

## A new page

1. Add a Markdown file in `guide/`.
2. Add it to `nav` in `mkdocs.yml`. The `index.md` of a folder goes first in its section, without a title: the section title in the menu then opens that page. List the pages of the folder under "In this section" in that `index.md`.
3. For a new folder with an `index.md`, add the two lines to `dist/_redirects`.

## License

MIT
