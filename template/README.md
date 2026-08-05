# new-template-name

NEW_TEMPLATE_DESCRIPTION

A [Templetry](https://github.com/Templetry) template. Generated with `templetry init meta/template` — replace the sample project files with your real stack and keep the manifest honest.

## Try it

```sh
templetry render --template . --out /tmp/demo --set "project_name=Demo App" --feature extras
```

## Where things live

| File | Teaches |
|---|---|
| `template.yml` | Your manifest: variables, identity map, features |
| `hello.py` | Directives in action (`tpl:if`) and canonical identity in code |
| `config.json` | A patch target (JSON has no comments) |
| `extras/` | Feature-gated files |
| `GUIDE.md` | The author's rules — read this first |
| `.github/workflows/verify.yml` | Render + leftover check on every push |
