# Templetry parent: meta

Templates about Templetry itself. One form:

| Form | What it creates | Status |
|---|---|---|
| [`template/`](template/) | **A new Templetry template**: pre-filled manifest, author's guide, live showcase of the three mechanisms (identity, directives, patches) and a verify workflow | ✅ ready |

## Usage

```sh
templetry init meta/template --out ./my-template \
  --set "template_name=my-template" \
  --set "template_description=What it scaffolds"
```

Then read the generated `GUIDE.md`, replace the sample files with your real stack, keep the verify CI green, and list it in any registry — your own (`registry.json`, schema v2, served anywhere) or a shared one. The CLI (`--registry`) and the desktop app (Settings → Registry URL) both consume custom catalogs.
