# AGENTS

Operating contract for AI agents and automation helpers working in this project.

## Mission

- This project **is a Templetry template**. Everything in it is rendered into somebody else's repository, so a mistake here is copied into every project generated from it.

## Core Rules

- **No templating language in source files.** Values arrive by identity renaming (`TemplateApp` becomes the project's name, with its casings) and by directives inside ordinary comments (`# tpl:if feature`). A source file must stay valid, compilable source.
- **The template is a real project that builds.** Not a skeleton with placeholders. If it cannot build, it is not ready — that is the bar the whole catalog is held to.
- **Declare, do not hardcode.** Variables, features, presets and the taxonomy all live in `template.yml`. The engine knows nothing about any framework and must not need to.
- **Every feature combination has to work.** A feature that only builds when another is on needs `requires`, not a comment.
- **Three environment profiles** — development, staging, production — using this ecosystem's own mechanism, never a Templetry-shaped one.

## Safe Change Workflow

1. Read `GUIDE.md` before changing the manifest — it explains each field and the rules behind it.
2. Render the template and build the output; that is the only proof that a change works.
3. Review the diff with git before committing.

## Required checks before finishing

```sh templetry:checks
templetry verify --template .
```

## This project came from a template

Four facts you cannot infer from the code in front of you:

- **Never hand-edit `.templetry-answers.yml`.** It records what generated this project. Editing it makes the next update merge against a state that never existed.
- **Before writing a capability by hand, run `templetry pieces`.** Auth, RBAC, audit trails, API keys and whole CRUD resources may already exist as pieces for this template. Adopting one is `templetry add <name>`, and it brings its own tests.
- **`templetry update` pulls improvements from the template** through a three-way merge that keeps your edits. Use it instead of copying files from the template by hand.
- **Directives like `tpl:if` belong to the template, not here.** If you find one in this project, it is a rendering bug worth reporting — do not try to interpret it.
