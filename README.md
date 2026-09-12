# woonkamerquiz.be

Teaser landing page for De Quizfabriek. Static: one HTML file with inline CSS
and JS, a vector logo, and a share image. No build step, no dependencies, no
third-party requests.

```
www/
  index.html          the page
  logo.svg            De Quizfabriek, viewBox trimmed to the artwork
  og.png              1200x630 share preview
  apple-touch-icon.png
.buildpacks           dokku/heroku-buildpack-nginx
.static               marks this as a static app for Dokku's detection
```

The countdown runs to **11 November 2026, 11:11 Europe/Brussels**, hardcoded in
`www/index.html`. It is hidden until the script fills it in (so a visitor
without JS sees the logo and the button rather than a frozen row of zeroes) and
hides itself again once that moment passes.

## Local preview

```sh
python3 -m http.server 8000 --directory www
# http://localhost:8000
```

## Deploy

The buildpack serves `/app/www` by default, and leaves an existing `www/` in the
repository alone — so this layout needs no configuration. (`NGINX_ROOT` is only
for a document root *nested inside* `www/`; setting it to `www` here would
resolve to `/app/www/www` and serve nothing.)

First time, on the Dokku host:

```sh
dokku apps:create woonkamerquiz
dokku domains:set woonkamerquiz woonkamerquiz.be
```

From this repo:

```sh
git remote add dokku dokku@<dokku-host>:woonkamerquiz
git push dokku main
```

Then, once DNS points at the host:

```sh
dokku letsencrypt:set woonkamerquiz email <you>@example.com
dokku letsencrypt:enable woonkamerquiz
```

Every later deploy is automatic: `.github/workflows/deploy.yml` pushes `main` to
Dokku on every push, and can be re-run by hand from the Actions tab. It needs
two repository settings, both already in place:

| Setting | Kind | What it is |
| --- | --- | --- |
| `DOKKU_SSH_KEY` | secret | private half of a deploy-only ed25519 keypair, authorized on the host with `dokku ssh-keys:add` |
| `DOKKU_KNOWN_HOSTS` | variable | the host's pinned ed25519 line, so the runner verifies who it is talking to |

A manual `git push dokku main` still works and is the fallback if Actions is
down.

Two things to know if you touch the workflow. The checkout needs
`fetch-depth: 0` — Dokku refuses a shallow push. And a manual re-run against an
unchanged commit is a no-op, because Dokku only rebuilds when it receives a new
commit; to force a rebuild, run `dokku ps:rebuild woonkamerquiz` on the host.

The deploy key is not scoped to this app. Dokku grants a key access to every app
on the host, so treat `DOKKU_SSH_KEY` as host-wide credentials.
