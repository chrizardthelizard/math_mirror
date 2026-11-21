# AGENTS.md

## Project shape
- Pelican static site ("Math Mirror"). Source of truth is `content/`; generated site is `output/`; the live site is GitHub Pages branch `gh-pages`, not `main`.
- Theme is `themes/blue-penguin-dark` inside the `themes/` git submodule (upstream `getpelican/pelican-themes`). Theme changes belong in the submodule pin, not local edits that the parent repo won't carry.

## Environment / setup
- No `requirements.txt`, `pyproject.toml`, lockfile, or test/lint config. Dependencies live in conda env `pelican` (Python 3.14, Pelican 4.11); the default `python` has no Pelican. Use `conda activate pelican` or `conda run -n pelican ...` before building.
- `tasks.py` imports `invoke`, which is not installed in any local conda env — the `invoke` tasks are unusable as-is. Use the direct Pelican commands below.
- `.gitmodules` points `themes` at a local Windows path (`.\themes\`), so a fresh clone cannot init it normally. Override: `git -c submodule.themes.url=https://github.com/getpelican/pelican-themes.git submodule update --init themes`.

## Build / preview / publish
- Build: `pelican content -o output -s pelicanconf.py`
- Rebuild on change: `pelican -r content -o output -s pelicanconf.py`
- Preview at localhost:8000: `pelican -l content -o output -s pelicanconf.py`
- `DELETE_OUTPUT_DIRECTORY = True` in both configs: every build wipes `output/`. That directory is tracked on `main` only from the initial commit and is not maintained there, so expect large diffs — don't commit generated output with source changes unless intentional.
- `Makefile` targets `public/`, not the deployed `output/`; plain `make html` does not refresh the site.
- `invoke gh_pages` in `tasks.py` is set to branch `main`, but the deployed branch is `gh-pages`. Use the documented flow instead: `pelican content -o output -s pelicanconf.py` → `ghp-import output -b gh-pages` → `git push origin gh-pages` (`ghp-import` is installed in the `pelican` env).

## Content conventions
- Articles: `content/blogs/*.md` with `Title`/`Date`/`Category`/`Author`/`Summary` headers; pages: `content/pages/*.md`; static assets: `content/images/` and `content/files/`. `content/drafts/` is absent from `ARTICLE_PATHS`, so builds ignore it — that's where unfinished posts live.
- Homepage is `content/pages/home.md`: a hidden page with `Save_as: index.html` and an empty `Url:`. The article list lives at `/blog/` via `INDEX_SAVE_AS = 'blog/index.html'`; `MENU_INTERNAL_PAGES` supplies the nav link.
- Article URLs are root-level slugs (`content/blogs/post_2_pinns.md` → `/pinns.html`), so image links are written site-root-relative (`images/foo.png`). Match filename case exactly; GitHub Pages is case-sensitive even though local Windows isn't.
- Keep `RELATIVE_URLS = False` and `SITEURL` fixed to `https://chrizardthelizard.github.io/math_mirror/`; earlier commits repeatedly fixed broken image paths after switching to relative URLs.
- In pages, `{attach}/images/file.pdf` copies the file next to the generated page (`/pages/file.pdf`), not to `/images/`.
