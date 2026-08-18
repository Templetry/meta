# Template author's guide

Everything you need to build, verify and publish Templetry templates — from a single template to a full parent with multiple forms. Distilled from building the official catalog.

## 1. How Templetry works

When someone runs `templetry init <parent>/<form>` (or `render` on a local directory), the engine:

1. **Fetches** the template (a GitHub tarball or a local folder) into memory.
2. **Plans**: reads `template.yml`, validates the user's inputs (variables get defaults or fail loudly; unknown keys are errors), computes which files each feature includes/excludes, and orders the identity renames longest-first.
3. **Renders** in memory: comment directives are evaluated, identity strings are replaced in content *and* paths, JSON patches are applied, and a `.templetry-answers.yml` is added recording template, source and inputs (that file is what makes future updates possible).
4. **Writes** the result — only then is disk touched. Same inputs always produce byte-identical output.

`template.yml` is the whole API between you and the engine. Every string variable automatically gets casings: `{key}`, `{key.pascal}`, `{key.camel}`, `{key.kebab}`, `{key.snake}`, `{key.flat}`.

## 2. The golden rule: your template compiles

A template is a **real, working project** with a canonical identity — never a skeleton of broken placeholders. If your stack builds, your template must build as-is. That is what lets CI verify every render, in every ecosystem, without you hand-testing them.

## 3. The three mechanisms

1. **Identity map** — global renames in content and paths. Longest-first; dotted entries (`com.example.base`) also rewrite their slash form (`com/example/base`) in paths *and* docs.
2. **Directives** — inside comments, so files stay valid syntax: `tpl:if key` / `tpl:if !key` / `tpl:endif` alone on their line; `tpl:var key literal` attached to a code line. See `hello.py` live. JSON has no comments — that's what patches are for.
3. **Patches** — RFC 6902 operations on JSON files, declared per feature. See `config.json` + the `extras` feature.

## 4. Identity archaeology

Writing the manifest is mostly finding **every form your identity takes**: the dotted package, the bare token (plugin ids, storage folders), the PascalCase class prefix, the spaced human label in resources, the slash form in docs, lockfiles. The loop that finds them fast:

```sh
templetry render --template . --out /tmp/out --set "project_name=Probe App"
grep -ri "sample" /tmp/out    # any survivor is a missing identity entry
```

## 5. Features: the additive axis

A feature adds/removes files (globs matched against *template* paths) and applies patches, over one skeleton. Features combine freely at render time — with everything ON your template must still build (that's what CI compiles).

## 6. Parents and forms: growing beyond one template

When you want *variants* of your template, use the catalog model:

- **Parent = one repo per concept** (e.g. `you/kmp`). This very template can become a parent's first form.
- **Form = one structural variant = one subdirectory** of the parent repo, each with its own `template.yml`, each compiling on its own. Users *choose* a form; they don't combine forms.
- **Golden rule of the split**: if a variant means *adding/removing files or blocks* over the same skeleton → it's a **feature**. If it's *the same code arranged differently* (single-module vs multi-module, different layer layout) → it's a **form**.

### Steps to build a parent with forms

1. Create the parent repo and move this template into a subdirectory named after the form (e.g. `starter/`). Everything keeps working — the CLI renders subdirectories directly: `templetry render --template ./parent/starter`.
2. Add the second form as a sibling directory with its own `template.yml`. Don't share files between forms — each form is self-contained and independently buildable.
3. Give the parent a root README with a forms table (name, what it is, status `ready`/`planned`).
4. Add a root CI workflow with **one job per form × input combo**: install the `templetry` CLI from releases, render the combo, then build the output with your stack's real toolchain. Every feature combination worth supporting gets a matrix entry.
5. Resist form explosion: a new form must justify itself structurally. Features are cheap (one CI, 2^n combos); forms are projects you maintain forever.

## 7. Publish and catalog

1. Push the repo; keep verify CI green.
2. Serve a `registry.json` (schema v2) from any URL — a raw GitHub file works:

```json
{
  "schema_version": 2,
  "parents": [
    {
      "key": "mystuff", "label": "My templates",
      "repo": "you/mystuff", "ref": "main",
      "forms": [
        { "form": "starter", "name": "mystuff-starter", "path": "starter",
          "status": "ready", "description": "What it scaffolds",
          "kinds": ["backend"], "languages": ["typescript"], "frameworks": ["express"] }
      ]
    }
  ]
}
```

3. Consumers point at it: CLI `templetry list --registry <url>` / `init --registry <url>`, or the desktop app via **Settings → Registry URL**. Your catalog behaves exactly like the official one.

## 8. AI-ready docs (convention)

Projects rendered from your template will be worked on by AI agents. Ship the guidance with the template — it renames with the project identity like everything else:

1. **`AGENTS.md` at the form root**: the operating contract — mission, core rules (module boundaries, build conventions), required checks before finishing, and the safe change workflow. Keep it imperative and short; agents follow lists, not prose.
2. Optional for bigger forms: `AI_INDEX.md` (map of where things live) and `docs/ai/` (architecture guardrails, pattern recipes). The `kmp` and `android` parents show the shape.
3. Everything an agent must run to validate a change (build, lint, tests) belongs in AGENTS.md **and** in CI — the contract and its teeth.

## 9. Environment profiles (convention)

Every form in the official catalog ships `development` / `staging` / `production` ([ADR-0018](https://github.com/Templetry/wiki/blob/main/adr/0018-environment-profiles.md)). Yours should too, as a feature keyed `environments`, default on.

**Use your ecosystem's own mechanism — do not invent a Templetry-shaped one.** That is the same rule that keeps framework knowledge out of the engine: a generated project should carry configuration its own tooling understands, not a layer nobody asked for. What the catalog uses today:

| Ecosystem | Mechanism |
|---|---|
| ASP.NET | `appsettings.{Environment}.json` + `ASPNETCORE_ENVIRONMENT` |
| Spring Boot | `application-{profile}.yml` + `SPRING_PROFILES_ACTIVE` |
| Vite | `.env.{mode}`, selected with `--mode` |
| Next.js | `.env.{profile}` chosen by `APP_ENV`, inlined via `next.config.ts` |
| NestJS | `@nestjs/config` with a `validate` hook |
| Android | product flavors + `buildConfigField` |
| Xcode | build configurations + `SWIFT_ACTIVE_COMPILATION_CONDITIONS` |
| KMP | a Gradle task generating one typed object into `commonMain` |
| Node, Python, Go, Rust | `.env.<profile>` at the root, selected by `APP_ENV` |

The names are the same everywhere on purpose, so `--feature environments` means one thing across the catalog. The long spellings win because ASP.NET and Vite both key on those exact strings.

**A profile is not three empty files.** It counts as done when all three hold:

1. **Three sources exist** and differ in something observable — a log level, a cache lifetime, an error-detail flag — not just in a name string.
2. **The app reads the active one through a single typed accessor**, validated as early as the platform allows: at startup, at context refresh, or at compile time. Not scattered `getenv` calls.
3. **A test asserts that loading a named profile yields that profile's values.** Without it, the profiles are decoration that rots on the first rename.

Where a platform cannot express one of these, say so rather than faking it. iOS makes the selection a compile-time fact and turns a missing flag into `#error`; Android verifies its generated `BuildConfig` in CI because per-variant codegen makes a plain unit test awkward across flavors.

**Committed profiles carry no secrets.** They hold non-secret defaults; real credentials come from the deployment environment or a secret manager. Gitignore a local override (`.env.local`, `appsettings.Local.json`) for the values one machine needs. A template that shipped a plausible-looking secret would teach the habit of committing them.
