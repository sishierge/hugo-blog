# Hugo Blog Healthcheck

This checklist is for `sishierge/hugo-blog` with dual delivery:
- GitHub Pages: `https://sishierge.github.io/hugo-blog/`
- Cloudflare custom domain: `https://sishierge.de5.net/`

## 1) Fast status check (2 minutes)

1. Check local branch status:
   - `git status -sb`
   - Confirm `main...origin/main` and no local pending changes.
2. Check latest Actions runs:
   - `gh run list --limit 6`
   - Ensure latest `Deploy Hugo to GitHub Pages` is `success`.
3. Compare deployed CSS fingerprint:
   - Open both homepages and check `main.min.<hash>.css` is the same.
4. Check Cloudflare latest deployment commit:
   - In Cloudflare Pages project `hugo-blog`, verify latest deployment commit equals GitHub `main` HEAD.

## 2) Expected repo configuration

- Default branch: `main`
- Branch protection on `main`: enabled
- Workflows:
  - `.github/workflows/hugo-github-pages.yml`
  - `.github/workflows/hugo-cloudflare-pages.yml`

## 3) Required GitHub Secrets / Variables

### Secrets
- `CLOUDFLARE_API_TOKEN`
- `CLOUDFLARE_ACCOUNT_ID`

### Variables
- `CLOUDFLARE_ENABLED`
  - `true`: Cloudflare upload step runs in workflow
  - `false`: Cloudflare upload step is skipped

## 4) Required Cloudflare Pages project settings

Project: `hugo-blog`

- Source: GitHub repo `sishierge/hugo-blog`
- Production branch: `main`
- Build command: `hugo`
- Build output: `public`
- Root dir: empty
- Custom domain: `sishierge.de5.net`

## 5) Common issues and fixes

### A. GitHub updated but site not updated

1. Check if latest workflow run failed:
   - `gh run list --limit 10`
2. If GitHub Pages failed, open failed log:
   - `gh run view <run_id> --log-failed`
3. If Cloudflare lagging, retry deployment in Cloudflare Pages UI.

### B. Cloudflare action fails with auth error

Symptoms:
- `Authentication failed` / `code 10000` / `apiToken not supplied`

Fix:
1. Recreate Cloudflare API Token (not Global API Key).
2. Token permissions:
   - Account: Cloudflare Pages Edit
   - Account: Account Settings Read
3. Update GitHub secret `CLOUDFLARE_API_TOKEN`.
4. Re-run workflow.

### C. Local vs production UI mismatch

Typical cause:
- Local edits inside `themes/reimu` submodule not committed upstream.

Fix:
1. Avoid relying on local submodule edits.
2. Move customizations to site-level files under:
   - `static/css/*`
   - `layouts/partials/*`
   - `config/_default/params.yml`
3. Rebuild + push.

### D. Giscus style mismatch

Check:
1. Post page references versioned CSS URLs in params:
   - `giscus_reimu_light.css?v=...`
   - `giscus_reimu_dark.css?v=...`
2. CSS responds with:
   - `Access-Control-Allow-Origin: https://giscus.app`
3. If still stale:
   - Hard refresh (`Ctrl+F5`)
   - Open in incognito window
   - Retry Cloudflare deployment

## 6) Safety recommendations

1. Rotate leaked credentials immediately.
2. Never paste secrets in chat or commit them.
3. Keep `main` protected and require review.
4. Prefer one source of truth for theme customizations (site-level overrides).

