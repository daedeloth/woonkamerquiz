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
  CNAME               custom domain, must ship inside the Pages artifact
.github/workflows/
  pages.yml           publishes www/ to GitHub Pages on push to main
.buildpacks           dokku/heroku-buildpack-nginx (fallback host)
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

GitHub Actions publishes `www/` to GitHub Pages on every push to `main`
(`.github/workflows/pages.yml`), and can be re-run by hand from the Actions tab.
There are no deploy credentials: `deploy-pages` authenticates with an OIDC token
minted during the run, so there is no key to leak or rotate.

`www/CNAME` holds the custom domain. It has to stay inside `www/` -- the domain
is only preserved if the CNAME ships in the uploaded artifact.

Both `woonkamerquiz.be` and `www.woonkamerquiz.be` work. Pages serves whichever
is in `CNAME` and permanently redirects the other to it, so the apex is
canonical and `www` redirects to it. Swapping the direction means changing
`CNAME`, not the DNS. (Today both hostnames answer `200` independently, which
is duplicate content with no canonical -- the redirect is an improvement.)
The certificate GitHub issues covers both.

Pages does not serve a private repository on a free plan, so this repository is
public.

### Cutover

The domain is proxied through Cloudflare, with Dokku as the origin. Pages is not
live until DNS moves:

1. Merge, and confirm the Pages deploy is green and the `github.io` URL serves
   the page.
2. In Cloudflare, replace the origin A records for `woonkamerquiz.be` with the
   GitHub Pages addresses -- `185.199.108.153`, `185.199.109.153`,
   `185.199.110.153`, `185.199.111.153` -- and add a CNAME for `www` pointing
   at `daedeloth.github.io` (the user, not the repository path).
3. Set both records to DNS-only (grey cloud). GitHub cannot issue its
   certificate through the Cloudflare proxy.
4. Wait for the certificate, then tick **Enforce HTTPS** in the repository's
   Pages settings.
5. Re-enable the Cloudflare proxy if you want it, with SSL mode **Full**.

Until step 2, the live site is still served by Dokku.

### Dokku (previous host, kept as fallback)

`.buildpacks` and `.static` are still here, so `git push dokku main` works if
Pages is ever unavailable. The buildpack serves `/app/www` by default and leaves
an existing `www/` alone, so this layout needs no configuration. (`NGINX_ROOT`
is only for a document root *nested inside* `www/`; setting it to `www` here
would resolve to `/app/www/www` and serve nothing.)

```sh
dokku apps:create woonkamerquiz
dokku domains:set woonkamerquiz woonkamerquiz.be
git remote add dokku dokku@<dokku-host>:woonkamerquiz
git push dokku main
```

TLS on the Dokku side, if you ever fall back to it:

```sh
dokku letsencrypt:set woonkamerquiz email <you>@example.com
dokku letsencrypt:enable woonkamerquiz
```

Once the Pages cutover is done and settled, this section and the two buildpack
files can go.
