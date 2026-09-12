# CLAUDE.md

Teaser landing page for De Quizfabriek, at woonkamerquiz.be.

## The constraint that shapes everything

**This is a static site on GitHub Pages. There is no server.**

GitHub Pages serves files. It does not run code. Nothing in this repository can
execute at request time, so none of the following is possible here, no matter
how it is written:

- no backend, API, database, or server-side rendering
- no form handling, no signup, no email, no contact form that posts anywhere
- no server-side redirects, rewrites, or custom headers
- no environment variables or secrets at runtime -- anything shipped is public
- no `.htaccess`, no nginx config, no middleware
- no build step: `www/` is uploaded exactly as it sits in the repository

Anything interactive has to be client-side JavaScript running in the visitor's
browser, or a request to a service hosted elsewhere. If a request seems to need
a server, say so rather than writing code that cannot run. The nearest options
are a third-party endpoint, or moving the site to a real host -- it used to run
on Dokku, and `git log` has the config if that is ever needed again.

Custom 404s do work (`www/404.html`), because Pages serves them statically.

## Layout

```
www/            everything served, uploaded verbatim by the Pages workflow
  index.html    the whole page: inline CSS, inline JS, no dependencies
  CNAME         custom domain; must stay inside www/ or Pages drops the domain
.github/workflows/pages.yml
```

## Conventions

- One file. CSS and JS stay inline in `index.html` -- do not split them out or
  add a bundler.
- No dependencies, no frameworks, no web fonts. The one external request is the
  Matomo tracker on `stats.catlab.eu` (site id 41).
- The JS is ES5-style (`var`, IIFE) and works without any transpilation. Match
  it.
- Dutch (`nl-BE`) is the page language. Copy is Dutch; code and comments are
  English.
- The countdown target (11 November 2026, 11:11 Europe/Brussels) is hardcoded in
  `www/index.html`. It hides itself until JS fills it in, so a visitor without
  JS sees the logo and button rather than a row of zeroes -- keep that
  behaviour.

## Deploying

Push to `main`; the workflow publishes `www/`. Never commit anything to `www/`
that should not be public -- it is served verbatim, and the repository is public
because Pages requires it on this plan.

See `README.md` for DNS and the apex/www redirect.
