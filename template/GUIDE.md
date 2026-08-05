# Template author's guide

The rules that make a Templetry template work, distilled from building the official catalog.

## The golden rule: your template compiles

A template is a **real, working project** with a canonical identity — never a skeleton of broken placeholders. If your stack builds, your template must build as-is. That is what lets CI verify every render.

## Identity archaeology

Writing the manifest is mostly finding **every form your identity takes**. Check for: the dotted package (`com.example.base`), the bare token (plugin ids, storage dirs), the PascalCase name, the spaced human label ("Sample Project" in strings/resources), the slash form in docs (`com/example/base`), and lockfiles. The loop that finds them fast:

```sh
templetry render --template . --out /tmp/out --set "project_name=Probe App"
grep -ri "sample" /tmp/out    # any survivor is a missing identity entry
```

## The three mechanisms

1. **Identity map** — global renames in content and paths. Longest-first, dotted entries also rewrite their slash form.
2. **Directives** (`tpl:if key`, `tpl:if !key`, `tpl:endif`, `tpl:var key literal`) — inside comments, so files stay valid. See `hello.py` for a live example. JSON has no comments: that's what patches are for.
3. **Patches** — RFC 6902 operations on JSON files, declared per feature. See `config.json` + the `extras` feature.

## Features: additive only

A feature adds/removes files and blocks over one skeleton. If a variant is *the same code arranged differently*, it's a different **form** (its own directory in a parent repo), not a feature.

## Publish and catalog

1. Push this template to a repo (or a form directory inside a parent repo).
2. Verify CI stays green (`.github/workflows/verify.yml` renders and checks leftovers — add a real build step for your stack).
3. List it in a registry: your own `registry.json` (schema v2) served anywhere — users point the CLI (`--registry`) or the desktop app (Settings → Registry URL) at it.
