# Votes to Win

Free win number calculator for candidates and campaign managers, with a paid full campaign plan (weekly door and phone goals, budget, call time, and a week-by-week calendar to Election Day).

Live site: https://jatreat.github.io/votes-to-win/

The whole app is one static `index.html`. Settings (price, Stripe payment link, support email, memo service URL) live in the `CONFIG` block near the top of the script.

## Strategist's memo (optional)

Buyers can have a written strategist's memo added to their plan. The site stays static: the memo comes from a small Cloudflare Worker in [`worker/`](worker/) that holds the Claude API key and streams the memo back. The section only appears once `CONFIG.memoApi` is set to the Worker's URL; leave it empty and the site is exactly as before. Setup steps are in [`worker/README.md`](worker/README.md).
