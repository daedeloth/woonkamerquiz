# woonkamerquiz.be

Teaser landing page for De Quizfabriek. Static: one HTML file with inline CSS
and JS, a vector logo, and a share image. No build step and no dependencies.
The only external request is the Matomo tracker on `stats.catlab.eu`.

```
www/
  index.html          the page
  logo.svg            De Quizfabriek, viewBox trimmed to the artwork
  og.png              1200x630 share preview
  apple-touch-icon.png
  CNAME               custom domain, must ship inside the Pages artifact
.github/workflows/
  pages.yml           publishes www/ to GitHub Pages on push to main
CLAUDE.md             notes for anyone (or anything) editing this repo
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

## Analytics

Matomo, self-hosted on `stats.catlab.eu`, site id 41, in the `<head>` of
`www/index.html`. It is the one third-party request the page makes.

## Deploy

GitHub Actions publishes `www/` to GitHub Pages on every push to `main`
(`.github/workflows/pages.yml`), and can be re-run by hand from the Actions tab.
There are no deploy credentials: `deploy-pages` authenticates with an OIDC token
minted during the run, so there is nothing to leak or rotate.

`www/CNAME` holds the custom domain. It has to stay inside `www/` -- Pages only
keeps the domain if the CNAME ships in the uploaded artifact.

Both `woonkamerquiz.be` and `www.woonkamerquiz.be` work. Pages serves whichever
hostname is in `CNAME` and permanently redirects the other to it, so the apex is
canonical and `www` redirects to it. Swapping the direction means editing
`CNAME`, not DNS. The certificate GitHub issues covers both.

Pages does not serve a private repository on a free plan, so this repository is
public.

### DNS

Cloudflare fronts the domain, with GitHub Pages as the origin.

| Record | Value |
| --- | --- |
| `woonkamerquiz.be` A | `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153` |
| `www` CNAME | `daedeloth.github.io` |

All four A records, not one: browsers fail over to the next address only if it
was published, so a single record is a single point of failure for no saving.

If the certificate ever has to be reissued, set those records to DNS-only (grey
cloud) first -- GitHub cannot complete the challenge through the Cloudflare
proxy -- then re-enable the proxy with SSL mode **Full**.
