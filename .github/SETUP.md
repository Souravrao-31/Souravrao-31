# Profile README — maintenance notes

## Why the cards broke (Aug 2026)

The README used to point at the **public** instances of two widget services.
Both went down, and they fail silently as broken images:

| Widget | Old host | Status when checked |
| --- | --- | --- |
| Stats + Top Languages | `github-readme-stats.vercel.app` | `503 DEPLOYMENT_PAUSED` — over Vercel free-tier quota |
| Trophies | `github-profile-trophy.vercel.app` | `402 Payment Required` — billing suspended |
| Streak | `github-readme-streak-stats.herokuapp.com` | still 200, but Heroku free tier died in 2022 — living on borrowed time |
| Contribution graph | `github-readme-activity-graph.vercel.app` | fine — GitHub's image proxy had just cached an old failure |

These are shared public deployments. Thousands of profiles hammer them, they
blow past the hosting quota, and every README pointing at them goes blank.
Nothing was wrong with the account or the markup.

## What it points at now

Working mirrors of the same projects, all verified returning `200`:

- Stats / languages / repo pins → `github-readme-stats-sigma-five.vercel.app`
- Trophies → `github-trophies.vercel.app`
- Streak → `streak-stats.demolab.com` (the maintained domain)
- Contribution graph → `github-readme-activity-graph.vercel.app`

Mirrors are still someone else's free hosting. They can hit the same wall.
The two workflows below remove that risk entirely.

---

## Workflow 1 — `snake.yml` (no setup required)

Renders the contribution grid as a snake animation and commits the SVG to an
`output` branch. Uses the built-in `GITHUB_TOKEN`.

1. Push this repo.
2. **Actions** tab → **Generate Snake** → **Run workflow**.
3. Once it turns green, uncomment the snake block in `README.md`
   (search for `snake-dark.svg`).

After that it refreshes daily at 03:00 UTC.

---

## Workflow 2 — `metrics.yml` (needs one token)

Generates the stats, language breakdown, and an isometric contribution
calendar as SVGs committed **into this repo**, so the README stops depending
on any third party.

**This step is optional.** The mirror URLs in the README already work — this
just removes the dependency on them.

### Where does `METRICS_TOKEN` come from?

Nowhere — you create it. `METRICS_TOKEN` is simply the name this repo stores
your personal access token under, so `metrics.yml` can read it as
`${{ secrets.METRICS_TOKEN }}`. Rename it in both places if you prefer.

Why a PAT instead of the built-in `GITHUB_TOKEN`: that one is scoped to this
single repo, so it cannot read account-wide commit counts, stars, or language
totals. `snake.yml` only needs this repo's data, which is why it needs no setup.

### 1. Create the token

Open <https://github.com/settings/tokens/new?scopes=public_repo,read:user&description=metrics>
(that link pre-selects the correct scopes), then:

| Field | Value |
| --- | --- |
| Note | `metrics` |
| Expiration | **No expiration** — otherwise the workflow silently starts failing the day it lapses |
| Scopes | `public_repo` and `read:user` **only** — this token just reads public stats |

Click **Generate token** and copy the `ghp_…` string. GitHub displays it once;
navigate away and you'll have to regenerate it.

### 2. Store it in this repo

Open <https://github.com/Souravrao-31/Souravrao-31/settings/secrets/actions/new>

- **Name:** `METRICS_TOKEN` — exact, case-sensitive
- **Secret:** paste the token
- **Add secret**

### 3. Run it

**Actions** tab → **Generate Metrics** → **Run workflow**.

If Actions are disabled on the repo, enable them first under
**Settings → Actions → General → Allow all actions**.

**Then swap the README over** — replace the mirror URLs in the Dashboard
section with the locally generated files:

```markdown
<img src="./metrics/overview.svg"  alt="GitHub Stats"/>
<img src="./metrics/languages.svg" alt="Top Languages"/>
<img src="./metrics/calendar.svg"  alt="Contribution Calendar"/>
```

Those are plain files in your own repo. They cannot 503.

---

## Fallback — self-host `github-readme-stats`

If you'd rather keep the original card design and own the deployment:

1. Fork <https://github.com/anuraghazra/github-readme-stats>
2. Import the fork at <https://vercel.com/new> (free Hobby plan is enough)
3. Add an env var `PAT_1` = a classic PAT with `public_repo` scope
4. Deploy, then replace `github-readme-stats-sigma-five.vercel.app` in the
   README with your own `*.vercel.app` domain

Your own instance serves one profile instead of thousands, so it never
exhausts its quota.

---

## Gotcha: GitHub caches images aggressively

GitHub proxies every external image through `camo.githubusercontent.com` and
caches the result — including failures. After changing a URL the new image is
usually picked up right away, but a *previously broken* URL can stay broken in
your browser for a while. To force a refresh, append a throwaway query param
(`&v=2`) or purge with:

```bash
curl -s -X PURGE "https://camo.githubusercontent.com/<hash>"
```

## Gotcha: some badge logos no longer exist

Shields.io pulls icons from Simple Icons, which has **removed** several brand
icons after trademark requests — including **LinkedIn, AWS, Amazon, and
OpenAI**. `?logo=linkedin` is not a typo; the icon is simply gone and the badge
renders text-only. Those badges intentionally omit the `logo=` parameter so
they don't look half-broken. Also note `css3` was renamed to `css`.
