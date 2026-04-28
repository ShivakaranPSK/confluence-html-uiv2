# ERD Studio — GitHub Pages

Interactive ERD diagram viewer. The diagram auto-loads from `schema.json` in this repo.  
**To update the diagram: just commit a new `schema.json`. No code changes needed.**

---

## Repository structure

```
├── index.html      ← ERD viewer (never needs editing)
├── schema.json     ← YOUR SCHEMA — the only file pipelines update
└── README.md
```

---

## Setup (one time)

### 1. Fork or create this repo on GitHub

Push both `index.html` and `schema.json` to the `main` branch.

### 2. Enable GitHub Pages

Go to **Settings → Pages → Source** → select `main` branch → root `/` → **Save**.

Your viewer will be live at:
```
https://<your-org>.github.io/<repo-name>/
```

### 3. Embed in Confluence

In your Confluence page, insert an **iFrame macro** with:
```
URL:    https://<your-org>.github.io/<repo-name>/
Height: 750
Width:  100%
```

That's it. Every visitor to the Confluence page sees the live diagram — no uploads, no manual steps.

---

## Updating the schema

The **only file** that ever changes is `schema.json`.  
Pipeline, CI job, or human — just commit a new version:

```bash
# From your pipeline or locally
cp your-generated-schema.json schema.json
git add schema.json
git commit -m "chore: update ERD schema [auto]"
git push
```

GitHub Pages deploys in ~30 seconds. The viewer fetches `schema.json` fresh on every load (no caching), so **all Confluence visitors see the updated diagram immediately** after the next page refresh.

---

## schema.json format

```json
{
  "title": "My Schema",
  "updatedAt": "2026-04-28",
  "updatedBy": "data-pipeline",
  "domains": {
    "domain_key": {
      "label": "Tab label",
      "color": "#6366f1",
      "tables": [
        {
          "id": "table_id",
          "name": "TABLE_NAME",
          "note": "Optional description",
          "fields": [
            { "n": "ID",          "t": "pk", "dt": "varchar",  "pi": false, "masked": false },
            { "n": "CUSTOMER_ID", "t": "fk", "dt": "varchar",  "pi": false, "masked": false },
            { "n": "EMAIL",       "t": "",   "dt": "varchar",  "pi": true,  "masked": true,  "note": "PI" }
          ]
        }
      ],
      "edges": [
        { "from": "table_a", "to": "table_b", "label": "FK_COLUMN", "cardinality": "1:N" }
      ],
      "layout": {
        "table_id": [60, 60]
      }
    }
  }
}
```

### Field reference

| Field | Values | Description |
|-------|--------|-------------|
| `"n"` | string | Column name |
| `"t"` | `"pk"` \| `"fk"` \| `""` | Key type |
| `"dt"` | string | SQL datatype (`varchar`, `datetime`, `float`, …) |
| `"pi"` | true \| false | Personal Information / PII flag |
| `"masked"` | true \| false | Masked in non-production environments |
| `"note"` | string | Column description (shown in detail panel) |

### Multiple domains

Add multiple keys under `"domains"` for multi-domain ERDs. Each domain shows as a tab in the viewer.

### Layout coordinates

`"layout"` sets `[x, y]` pixel positions for each table.  
**Omit it** and use the **Auto layout** button in the viewer — it arranges tables automatically.  
Commit the resulting layout from the viewer by copying the JSON from the browser after dragging.

---

## Confluence iFrame snippet

```
{iframe:src=https://<your-org>.github.io/<repo-name>/|width=100%|height=750px|frameborder=0|scrolling=no}
```

Set height based on how many tables you have. For large schemas (10+ tables), use `900px`.

---

## Pipeline integration example

```yaml
# .github/workflows/update-erd.yml
name: Update ERD schema

on:
  push:
    paths:
      - 'data-models/**'   # trigger when your source models change

jobs:
  update-schema:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate schema.json
        run: |
          python scripts/generate_erd_schema.py > schema.json

      - name: Commit updated schema
        run: |
          git config user.name  "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add schema.json
          git diff --staged --quiet || git commit -m "chore: update ERD schema [auto] $(date -u +%Y-%m-%d)"
          git push
```

---

## Viewer features

- **Auto-loads** `schema.json` on every page open — always current
- **↺ Reload** button in status bar to refresh without page reload
- **Pan** by dragging the canvas
- **Zoom** with scroll wheel or `Ctrl/⌘ +/-`
- **Fit to view** with `Ctrl/⌘ 0` or the ⊡ button
- **Click a table** → detail panel with all columns, types, and relationships
- **Hover an edge** → relationship label appears
- **▾ Show more** → expand tables with many columns (collapses at 4+)
- **PI / Masked** column highlighting for data governance
- **Domain tabs** for multi-domain schemas
- **Export SVG** for static documentation
- **Minimap** for navigation in large diagrams
- **📖 Guide** button for inline reference
