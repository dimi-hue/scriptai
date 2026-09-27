# Slateline — AI script generator

A free-to-use tool for short-form creators: pick a niche and a runtime, get a
fresh video idea with a scene-by-scene shot list, written by AI (via Groq's
free API, running Llama 3.3).

```
scriptslate-ai/
├── index.html        the whole frontend — one page, no build step
├── api/
│   └── generate.js   serverless function that calls the Groq API
├── package.json
├── .env.example       copy to .env for local testing
└── .gitignore
```

The frontend never talks to Groq directly. It calls `/api/generate` on your
own domain, and that server-side function holds your API key. Your key is
never sent to the browser or visible in page source.

If `/api/generate` isn't reachable (no key set yet, or it errors), the page
falls back to a built-in local idea bank so the tool still works — you'll
just see "OFFLINE IDEA BANK" instead of "WRITTEN FRESH BY AI" on the script.

Each visitor's browser is limited to 3 generations (see "Usage limit" below).

## 1. Get a free Groq API key

Go to **console.groq.com**, sign up (no card required), and create a key
under API Keys. It starts with `gsk_`.

## 2. Deploy (Vercel is the easiest option — free tier works fine)

**Option A — no command line:**
1. Push this folder to a new GitHub repo.
2. Go to vercel.com → New Project → import that repo. Vercel auto-detects
   `api/generate.js` as a serverless function, no config needed.
3. Before the first deploy (or right after, then redeploy), go to
   **Project Settings → Environment Variables** and add:
   - `GROQ_API_KEY` = your key
4. Deploy. Your site is live at `your-project.vercel.app`.

**Option B — command line:**
```bash
npm i -g vercel
cd scriptslate-ai
vercel                      # follow the prompts to create the project
vercel env add GROQ_API_KEY   # paste your key when asked
vercel --prod
```

**Other hosts:** Netlify Functions and Cloudflare Pages Functions work the
same way conceptually (an HTTP function reading an env var) but expect a
slightly different file location/export signature than Vercel's `api/*.js`.
If you want to use one of those instead, say so and the function can be
adapted to match.

## 3. Point your own domain at it

In Vercel: Project Settings → Domains → add your domain, then follow the DNS
instructions it gives you (usually one A or CNAME record at your registrar).

## 4. Apply for / add AdSense

Once the site is live on your own domain with real traffic:
1. Apply at google.com/adsense if you haven't already.
2. Once approved, paste your ad unit `<script>`/`<ins>` snippets directly
   into `index.html` — natural spots are right after the `<header>`, inside
   the `.panel` (below the hint text), and after the `<footer>` content.
3. Redeploy (`vercel --prod`, or just push to GitHub if it's connected).

## Usage limit

`index.html` currently caps each visitor's browser at **3 free
generations**, stored in `localStorage`. This is a first-pass limit, not
real account-based metering — clearing browser data, using incognito, or
switching devices resets it. To change the number, edit `GENERATION_LIMIT`
near the top of the `<script>` block in `index.html`.

For anything closer to a real per-person limit, you'd need visitor accounts
(sign-up/login) and a database to track usage server-side — a meaningfully
bigger step. Worth doing once the tool has real traction, not before.

## Groq's free tier

Groq's free tier has generous but real rate limits (requests per minute and
per day) that can change over time — check **console.groq.com** for current
numbers. If you outgrow it, Groq also offers paid tiers with higher limits.

## Local testing

```bash
npm i -g vercel
cd scriptslate-ai
cp .env.example .env    # fill in your real key
vercel dev
```
This runs the whole thing (frontend + function) on `localhost` so you can
test before deploying.

## Customizing the writing

Edit `SYSTEM_PROMPT` in `api/generate.js` to change tone, add niches beyond
the ones on the frontend, adjust script length/style, or add your own
brand voice. You can also try a different Groq-hosted model by setting
`GROQ_MODEL` (see available models at console.groq.com/docs/models).
