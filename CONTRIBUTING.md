# Working on the docs site

This repo is the source for [horchd.github.io](https://horchd.github.io)
(later: <https://horchd.xyz>). It uses [Docsify](https://docsify.js.org)
which is **runtime-only** — there is **no build step**. Edit a `.md`
file, commit, push, GitHub Pages serves the new file on the next
deploy.

## Local preview

Pick any static server. **Do not use `bun index.html`** — that tries to
bundle the page as a Vite app and fails on the CDN scripts. Docsify
needs a plain static server that just serves files as-is.

```bash
# Pythonbash
python -m http.server 8000

# bun
bunx serve

# pnpm / npm
pnpm dlx serve
npx serve

# the dedicated docsify dev server (with live reload)
bunx docsify-cli serve .
```

Then open <http://localhost:8000> (or whichever port the server picks).

## Adding a page

1. Create `your-page.md` at the repo root.
2. Add it to `_sidebar.md` so it shows up in the left nav.
3. Commit + push. The page is live within ~1 minute.

## Adding a section

`_sidebar.md` controls the left nav structure. Use markdown bullets +
indentation — see the existing entries for the syntax.

## Plugins

Loaded in `index.html`:
- Docsify core
- Search plugin (full-text in-browser search of all `.md` files)
- Prism syntax highlighting for `rust`, `toml`, `bash`, `python`

Extra languages can be added by appending another
`prism-<lang>.min.js` line to `index.html`.

## Custom domain (`horchd.xyz`)

Once DNS is set:
1. Drop a `CNAME` file at the repo root containing `horchd.xyz`.
2. At the registrar, add A records pointing to GH Pages IPs:
   `185.199.108.153`, `185.199.109.153`, `185.199.110.153`,
   `185.199.111.153`. Or a CNAME on `www` pointing at
   `horchd.github.io`.
3. In `gh repo` settings → Pages → enforce HTTPS once the cert is
   provisioned.
