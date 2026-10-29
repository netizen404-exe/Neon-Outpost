# NEON OUTPOST — Online Leaderboard Edition

This version adds a shared top-10 leaderboard to the game. Players choose a nickname and submit their final score. Scores live in Cloudflare D1, so the list is shared by visitors and devices instead of being stored only in one browser.

There are **two deployment setups** in this project. Use the Cloudflare Pages setup if your dashboard labels the project **Pages**. Use the `cloudflare-worker/` setup if it is a **Worker with static assets**. Do not deploy both to the same project.

## Option A — Cloudflare Pages

The main folder is the Pages version. It contains `index.html`, `functions/api/leaderboard.js`, and `schema.sql`.

1. Put `index.html`, the `functions/` folder, and `schema.sql` in the root of your existing Pages project/repository. Keep the `functions/` directory at the project root, not inside the public/build output directory.
2. In the Cloudflare dashboard, create a D1 database called `neon-outpost-leaderboard` (or choose an existing D1 database for this game).
3. Open that database's SQL Console and run the contents of `schema.sql` once.
4. In **Workers & Pages → your Pages project → Settings → Bindings**, add a **D1 database** binding. Set the variable name to exactly `DB`, and select the database from step 2.
5. Redeploy the Pages project after adding the binding.
6. Open `https://YOUR-SITE/api/leaderboard`. When configured, it should return JSON like `{"entries":[]}` before anyone submits a score.

**Important:** Cloudflare Pages Functions do not run if you only drag-and-drop the project in the dashboard. Use the project's Git integration or deploy the Pages project with Wrangler so the `functions/` route is deployed.

The file `functions/api/leaderboard.js` becomes the `/api/leaderboard` endpoint by Pages' file-based routing.

## Option B — Cloudflare Workers with static assets

Use the contents of `cloudflare-worker/` instead of the Pages function folder. This folder contains a Worker entry point and a `wrangler.jsonc` configuration.

If your existing site is already handled by a custom Worker with its own code or routes, do not blindly deploy this template as a replacement. Merge the `/api/leaderboard` route into your existing Worker and preserve its current configuration, or use this folder as a separate Worker only if you intend to route traffic to it.

Requirements: Node.js and Wrangler CLI access on your computer.

1. Open a terminal and change into the `cloudflare-worker/` folder.
2. Run `npx wrangler login` to connect Wrangler to your Cloudflare account.
3. Run `npx wrangler d1 create neon-outpost-leaderboard`. Copy the database ID printed by the command.
4. Edit `wrangler.jsonc`: replace `REPLACE_WITH_YOUR_D1_DATABASE_ID` with the ID from step 3. Keep the database name as `neon-outpost-leaderboard` and the binding as `DB`.
5. Run `npx wrangler d1 execute neon-outpost-leaderboard --remote --file=./schema.sql` to create the leaderboard table in the remote database.
6. Run `npx wrangler deploy` from the `cloudflare-worker/` folder.
7. Open `https://YOUR-WORKER-URL/api/leaderboard`. It should return JSON like `{"entries":[]}` before any scores exist.

If you already have a D1 database, you can reuse it by putting its database ID and name in `wrangler.jsonc`, then execute `schema.sql` against it once.

## How to check it

- `GET /api/leaderboard` should return `{ "entries": [] }` before scores are submitted.
- Complete a run, enter a nickname (use an alias, not a real name), and press **Submit score**.
- Open the leaderboard and press **Refresh**. Your score should appear.
- Open the site in a private window or on another device. The same score list should be visible there too.

## Privacy and limitations

Only the nickname and score details you submit are stored by this code; the API does not store an account, email address, or raw IP address. Nicknames are public, so do not use your real name or other private information.

This is a casual leaderboard, not a cheat-proof competition: the game runs in the visitor's browser, so a determined person could craft a fake score request. The endpoint validates formats and reasonable ranges, but it cannot prove that a submitted score was earned fairly. If the endpoint gets spammed, add a Cloudflare rate-limiting rule for `/api/leaderboard`.

## Files

- `index.html` — game and leaderboard interface for Pages
- `functions/api/leaderboard.js` — Pages API endpoint
- `schema.sql` — D1 table and ranking index
- `cloudflare-worker/` — alternative Worker-based deployment
- `LICENSE` — MIT licence
