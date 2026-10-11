---
name: ideation
description: Deploy, update and extend the Ideation Room app (A/B test ideation sessions with evidence, blind voting, hypothesis neatening, PXL prioritisation and roadmap) on GitHub and Vercel with Upstash Redis. Use for /ideation, "deploy the ideation app", "add a criterion", "change the scoring".
---

# Ideation Room

Static page `public/index.html` plus four Vercel functions: `api/store.js` (Redis get/set/hash/set ops with in-memory fallback), `api/ai.js` (Claude, with image input), `api/upload.js` and `api/blob.js` (images).

## Modes
1. **Deploy** — first time (below).
2. **Update** — change a stage, the prompts, the prioritisation criteria or weights, the roadmap rules; then verify and redeploy.
3. **Explain** — the user asks how scoring or a stage works: answer from the code (`CRIT`, `DEFAULT_WEIGHTS`, `scoreOf`, `buildRoadmap`).

Prerequisites for 1 and 2: `gh auth status`, `vercel whoami`, `ANTHROPIC_API_KEY` set. If missing, give the exact fix and stop.

## Deploy
1. `npm install`; verify (below).
2. `git init && git add -A && git commit -m "Ideation Room"` then `gh repo create ideation-room --public --source=. --push`.
3. `vercel --yes`; `printf '%s' "$ANTHROPIC_API_KEY" | vercel env add ANTHROPIC_API_KEY production`.
4. Persistence is required for real sessions: ask the user to open the Vercel dashboard → project → Storage → Create Database → **Upstash for Redis** → Connect (sets `KV_REST_API_URL` / `KV_REST_API_TOKEN`). Optionally also **Blob** for image uploads (sets `BLOB_READ_WRITE_TOKEN`); without it images are stored in Redis up to ~700KB each. Wait for confirmation, then `vercel env pull .env.local` to check.
5. `vercel --prod --yes`; put the URL in README; commit; push.
6. Smoke test: open the URL, the home page pill must read "Storage: shared". Create a session, copy the join link, open it in a private window, join as a second person, add an idea, open voting, vote from both, reveal. Then test "Generate insights" with a pasted CSV.
7. Report live URL and repo URL, and remind the user that the facilitator link (with `&f=`) must not be shared.

## Data model (Redis keys)
`sessions` (set of ids) · `session:<id>` (meta: name, page, goal, facilitator, fkey, dots, status, weights, runDays) · `s:<id>:context` · hashes `s:<id>:inputs|insights|people|ideas|votes|hyps` · `s:<id>:roadmap`. Status order: setup → insights → ideation → voting → revealed → hypotheses → prioritised → planned.

## Where things live in the page
- Stages: `<section class="stage" id="st-...">` and the `#stepper` buttons; stage logic under `/* ---------- N ... ---------- */` comments.
- AI prompts: `genInsights`, `genImagery`, `genHyps`, `aiCriteria` click handlers. Keep "Return ONLY JSON" and the schema when editing.
- Prioritisation: `CRIT` (criteria list), `DEFAULT_WEIGHTS`, `scoreOf`. Adding a criterion = add to `CRIT` and `DEFAULT_WEIGHTS`, and to the `criteria` object in the `aiCriteria` prompt.
- Roadmap: `buildRoadmap` (capacity per month, one test per surface per month).
- Facilitator-only UI carries class `fac`; permission is `isFac` (link key matches `meta.fkey`).

## Verify
```
node -e 'const s=require("fs").readFileSync("public/index.html","utf8");const m=[...s.matchAll(/<script>([\s\S]*?)<\/script>/g)];require("fs").writeFileSync("/tmp/ide.js",m.map(x=>x[1]).join("\n"))' && node --check /tmp/ide.js
```
Then `npx vercel dev` and walk the flow in step 6 with two browser windows.

## Demo notes
Run a real 30-minute session with two colleagues before showing it. The three things to point at: votes are blind until reveal, every insight names its source, and the prioritisation weights are on screen so the ranking can be challenged, not just presented.
