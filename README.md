# Adaptive Booking (BDR tool)

Static booking app for GitHub Pages. Deployed HTML uses a **password gate**: the plaintext password is stored only as a GitHub Actions secret; the site compares a SHA-256 hash in the page (injected at deploy time).

**Limits:** Anyone can view the published JavaScript and brute-force a weak password, or clear `sessionStorage`. This keeps casual visitors out; it is **not** the same as server-side authentication. For stronger protection, use something like [Cloudflare Access](https://www.cloudflare.com/zero-trust/products/access/) in front of a custom domain.

## One-time GitHub setup

1. Create a new repository on GitHub and push this project (see below).
2. **Settings → Secrets and variables → Actions → New repository secrets**
   - `SITE_ACCESS_PASSWORD` — password your team uses on the site (for the gate).

   Chili Piper round-robin URLs live in **`index.html`** (the `ROUTES` object). Edit and commit when those links change.
3. **Settings → Pages**
   - **Source:** GitHub Actions (not “Deploy from a branch”).
4. Push to `main`. The **Deploy GitHub Pages** workflow runs, injects the hash into `index.html` in the build only, and publishes the artifact. Your site will be at:

   `https://<username>.github.io/<repo-name>/`

   (GitHub shows the exact URL in the workflow run and in Pages settings.)

## Push this folder to GitHub

```bash
cd "/path/to/Show Booking"
git init
git add index.html .github README.md
git commit -m "Add Adaptive Booking for GitHub Pages"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```

Include `adaptive-booking-cursor-prompt.md` in the repo only if you want it in version control.

## Local development

The **`__GH_PAGES_ACCESS_SHA256__`** placeholder in `index.html` is replaced only in CI with the SHA-256 of `SITE_ACCESS_PASSWORD`. For local testing, leave it as-is to disable the gate, or run the same `printf` / `sed` step as in `.github/workflows/pages.yml` if you want to test the gate locally.

```bash
printf '%s' 'your-password' | shasum -a 256
```

## Changing the password

Update the `SITE_ACCESS_PASSWORD` secret in GitHub, then re-run the workflow (e.g. push an empty commit or **Actions → Deploy GitHub Pages → Run workflow**).
