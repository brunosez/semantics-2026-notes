# semantics-2026-notes

Bilingual (EN/FR) Quarto site holding my notes from **SEMANTiCS 2026**, Ghent.
Each talk gets one page, marked up with `schema.org` JSON-LD.

Published at <https://brunosez.github.io/semantics-2026-notes>.

---

## 1. One-time setup

```bash
# 1. Create the repo on GitHub, then:
git init
git add .
git commit -m "Scaffold SEMANTiCS 2026 notes site"
git branch -M main
git remote add origin git@github.com:brunosez/semantics-2026-notes.git
git push -u origin main
```

Then in the GitHub repo: **Settings → Pages → Build and deployment → Source:
`GitHub Actions`**. The workflow in `.github/workflows/publish.yml` takes it
from there on every push to `main`.

All URLs are already set to the `brunosez` account. If you ever rename the repo
or move it, these are the places to update:

- `_quarto.yml` → `site-url`, `repo-url`, and the GitHub navbar icon
- `about.qmd` / `fr/apropos.qmd` → the "open an issue" links
- the `isPartOf.url` field in each talk's JSON-LD

```bash
grep -rn 'brunosez' . --include='*.qmd' --include='*.yml'
```

## 2. Local preview

```bash
quarto preview        # live-reloading server
quarto render         # one-off build into _site/
```

Install Quarto from <https://quarto.org/docs/get-started/> if you haven't.

## 3. Adding a talk

```bash
cp talks/_template.qmd talks/2026-09-15-keynote-day1.qmd
cp fr/talks/_modele.qmd fr/talks/2026-09-15-keynote-jour1.qmd
```

Fill the YAML header, write the four sections, update the JSON-LD block to
match. Files beginning with `_` are never rendered, so the templates stay out
of the published site.

Keep the EN and FR filenames parallel (same date, matching slug). Nothing
enforces it — it's what keeps the two trees navigable by hand.

Delete the two `*keynote-day1*` / `*keynote-jour1*` example files once you have
real entries; every value in them is a placeholder.

## 4. Category vocabulary

Pick from this list rather than inventing a new tag each time — otherwise the
category filter on the index page becomes useless. Add to the list
deliberately.

| English (`talks/`)      | French (`fr/talks/`)            |
| ----------------------- | ------------------------------- |
| `keynote`               | `keynote`                       |
| `data-governance`       | `gouvernance-des-donnees`       |
| `data-quality`          | `qualite-des-donnees`           |
| `ontology`              | `ontologie`                     |
| `knowledge-graphs`      | `graphes-de-connaissances`      |
| `graphrag`              | `graphrag`                      |
| `llm`                   | `llm`                           |
| `kb-generation`         | `generation-de-bc`              |
| `industry`              | `industrie`                     |
| `research`              | `recherche`                     |

## 5. Structured data

Each talk page carries a `<script type="application/ld+json">` block with a
`@graph` of four nodes:

| Node          | Type          | Role                                                       |
| ------------- | ------------- | ---------------------------------------------------------- |
| `#talk`       | `Event`       | the talk, linked to the conference via `superEvent`         |
| `#speaker`    | `Person`      | the speaker, with `affiliation`                             |
| conference    | `Event`       | SEMANTiCS 2026, shared `@id` across all pages               |
| `#notes`      | `BlogPosting` | your notes, `about` the talk, `author` you                  |

The conference node uses the same absolute `@id` on every page, so a crawler
that merges the pages gets one conference node with many `superEvent` children
rather than N duplicates.

Validate before publishing:

- <https://validator.schema.org/> — paste a rendered page URL or its HTML
- Google Rich Results Test, if you care about search appearance

Quick local check that every block is valid JSON. Note it scans `_site/`, not
the `.qmd` sources — the sources contain `{{< var … >}}` shortcodes, which are
not valid JSON until Quarto resolves them, so run `quarto render` first.

```bash
quarto render
python3 - <<'PY'
import json, re, pathlib, sys
bad = n = 0
for p in pathlib.Path('_site').rglob('*.html'):
    for m in re.findall(r'<script type="application/ld\+json">(.*?)</script>', p.read_text(), re.S):
        n += 1
        try:
            json.loads(m)
        except json.JSONDecodeError as e:
            bad += 1
            print(f"{p}: {e}")
print(f"{n} block(s) checked — " + ("OK" if not bad else f"{bad} INVALID"))
sys.exit(1 if bad else 0)
PY
```

## 6. Things to verify yourself

I have not verified these against the live services — check them rather than
trusting the scaffold:

- **Conference dates and venue.** `_variables.yml` now says 15–17 September
  2026 at *Music Center de Bijloke*, Bijlokekaai 7, 9000 Ghent. I took this
  from the conference website, not from your own records — **you were there,
  so check it**, in particular the exact spelling of the venue (the Dutch name
  is *Muziekcentrum De Bijloke*) and whether your day 1 was the 15th.
  Everything derives from this one file, so a correction here fixes every page.
- **GitHub Action versions.** `checkout@v4`, `configure-pages@v5`,
  `upload-pages-artifact@v3`, `deploy-pages@v4`, `quarto-actions/setup@v2` were
  current as far as I know, but these move. If the workflow fails on a
  deprecated action, bump the major version.
- **Quarto listing options.** The index pages use `type: table` deliberately.
  I tested this on Quarto 1.6.39: the `default` listing layout **silently
  ignores** custom fields, so `speaker` never appears; `table` renders it as a
  sortable column. If you switch back to `default` for the nicer card layout,
  expect to lose the speaker column and fold that information into the
  `description` instead.

## Licence

Notes: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
Linked slides and abstracts belong to their respective speakers.
