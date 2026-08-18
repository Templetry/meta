# Environment profiles

Replace this directory with your ecosystem's own mechanism — see section 9 of
[GUIDE.md](../GUIDE.md).

`development`, `staging` and `production`, always those three names. What the
files look like depends entirely on the stack: `.env.<profile>` for Node,
Python, Go and Rust; `appsettings.{Environment}.json` for ASP.NET;
`application-{profile}.yml` for Spring; build configurations for Xcode;
product flavors for Android.

Whatever the shape, three things have to be true before it counts as done:

1. The three sources differ in something observable, not just in a name.
2. The app reads the active one through a single typed accessor, validated
   as early as the platform allows.
3. A test asserts that loading a named profile yields that profile's values.

And nothing committed here may be a secret. Gitignore a local override for
the values one machine needs.
