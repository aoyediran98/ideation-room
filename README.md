# Ideation Room

A reusable A/B test ideation app for CRO-led sessions with marketing, product, design and engineering in the room.

Flow per session: **Context** (objective, constraints, concerns; page URL, HTML or copy; ad imagery and screenshots) → **Insights** (anyone uploads quant CSVs, qual verbatims or opinions; AI generates sourced insight cards; imagery is analysed for message match) → **Ideas & votes** (everyone adds rough ideas, optionally anonymous and claimable later; blind dot-voting, revealed by the facilitator) → **Hypotheses** (AI neatens and merges ideas into the standard form with metric, guardrail and surface; all editable) → **Prioritise** (PXL-style objective criteria + ease + vote share, with visible weights; AI can suggest criteria, facilitator confirms) → **Roadmap** (sequenced into monthly slots, one test per surface at a time; Markdown and CSV export).

Facilitator rights come from a key in the creator's link; the join link has none. Participants identify by name and team. Everything is stored server-side (Upstash Redis) so sessions can be reopened and edited after the meeting. Live updates by polling every 5 seconds.

## Stack
`public/index.html` (the app) · `api/store.js` (Redis ops) · `api/ai.js` (Claude) · `api/upload.js` + `api/blob.js` (images: Vercel Blob if configured, else stored in Redis).

## Run locally
```
npm install
cp .env.example .env.local   # ANTHROPIC_API_KEY at minimum
npx vercel dev
```
Without Redis the store is in-memory and resets on restart (fine for trying it).

## Deploy
See `.claude/skills/ideation/SKILL.md`. Live: https://ideation-theta.vercel.app
