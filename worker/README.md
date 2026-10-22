# Votes to Win memo service

A Cloudflare Worker (free plan is plenty) that writes the strategist's memo for buyers with the Claude API. The browser sends the plan's numbers and the buyer's unlock code here; the Worker checks the code, asks Claude for the memo, and relays Claude's event stream back untouched (the page turns it into text). The API key never leaves the Worker.

Why the Worker doesn't parse the stream itself: the free plan allows 10 ms of CPU per request, and parsing a thousand streamed events in the Worker used about 40 ms, which got memos cut off mid-sentence. Passing the body through costs almost nothing.

What keeps the API credit safe:

- Only requests from the Votes to Win site are accepted (`ALLOWED_ORIGINS`).
- The unlock code must hash to `UNLOCK_HASH`, the same check the page makes.
- Each visitor gets at most 3 memos a minute, and each memo is capped at `MAX_TOKENS`.
- Keep the Claude Console organization with no purchased credit and no auto-reload. Then the monthly API credit is a hard ceiling: if it runs out, memos stop until the next cycle and nothing is charged.

## Setup (once, about 15 minutes)

1. **Claude API key.** On claude.ai, Settings > Billing > API credits > Link organization (pick Josh's Individual Org). Then open platform.claude.com, API keys, and create a key named `votes-to-win-memo`. Copy it; it's shown once.
2. **Cloudflare account.** Sign up free at dash.cloudflare.com if you don't have one.
3. **Create the Worker from this repo.** Dashboard > Workers & Pages > Create > Import a repository > connect GitHub > choose `jatreat/votes-to-win`. Settings for the build:
   - Project name: `votes-to-win-memo`
   - Root directory: `worker`
   - Build command: leave empty
   - Deploy command: `npx wrangler deploy`
   Save and deploy. It redeploys on every push to `main`.
4. **Add the secret.** On the Worker, Settings > Variables and Secrets > Add > type Secret, name `ANTHROPIC_API_KEY`, paste the key. Save. (The other settings are already in `wrangler.toml`.)
5. **Check it.** Open the Worker's URL (`https://votes-to-win-memo.<your-subdomain>.workers.dev`). It should answer `{"ok":true,...}`.
6. **Turn the memo on.** In `index.html`, set `CONFIG.memoApi` to that URL (no trailing slash) and push. The memo section appears for buyers; nothing changes for anyone else. `CONFIG.memoFrom` is an optional launch moment (ISO 8601, UTC): until then the section stays hidden even with `memoApi` set.

To turn it off, set `memoApi` back to `""`.

## Local run

```
cd worker
npm install
printf 'ANTHROPIC_API_KEY=sk-ant-...\nALLOWED_ORIGINS=https://jatreat.github.io,http://127.0.0.1:8080\n' > .dev.vars
npx wrangler dev
```

`.dev.vars` is ignored by git. For a test page, serve the repo root on `127.0.0.1:8080` with `memoApi` set to `http://127.0.0.1:8788`.

## Changing the memo

The instructions Claude follows are in `SYSTEM_PROMPT` in `src/index.js`: nonpartisan, operations only (field, volunteers, money, budget, timeline, risks), fixed headings, 650–900 words. Edit and push; the Worker redeploys.
