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

Every later deploy is just `git push dokku main`.
