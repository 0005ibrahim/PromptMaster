# PromptMaster

A course landing page with a live "Prompt Analyzer" — a Q&A box where visitors can ask
anything about AI prompting or prompt engineering and get a real answer from Claude.

```
promptmaster/
├── frontend/   static site (HTML/CSS/JS) — open in a browser or serve statically
└── backend/    Node/Express API that calls the Anthropic API on the frontend's behalf
```

The frontend never talks to Anthropic directly. It calls your own backend at
`/api/ask`, and the backend holds the API key and forwards the question to Claude.
This keeps your key out of the browser entirely.

## 1. Get an Anthropic API key

You need your own key to make this work — there isn't one built in.

1. Go to **https://console.anthropic.com/** and sign in or create an account.
2. Open **Settings → API Keys** (or **Get API Keys** from the dashboard).
3. Click **Create Key**, name it something like `promptmaster-dev`, and copy the value
   shown (it starts with `sk-ant-`). You won't be able to see it again after closing the dialog.
4. Anthropic API usage is billed separately from any Claude.ai subscription — check
   **console.anthropic.com/settings/billing** to add a payment method or view current credit,
   since a fresh account may need billing set up before requests succeed.

Treat this key like a password:
- Never paste it into the frontend code, a commit, or a public chat.
- Only put it in `backend/.env` (see below), which is already excluded via `.gitignore`.
- If a key ever leaks, revoke it immediately from the same API Keys page and issue a new one.

## 2. Run the backend

```bash
cd backend
npm install
cp .env.example .env
```

Open `backend/.env` and paste your key in:

```
ANTHROPIC_API_KEY=sk-ant-your-real-key-here
```

Then start the server:

```bash
npm start
```

You should see:

```
PromptMaster backend running on http://localhost:3001
```

Visit `http://localhost:3001/api/health` — it should report `{"ok":true,"hasKey":true}`.
If `hasKey` is `false`, the `.env` file isn't being picked up (check it's in `backend/`,
not the project root, and that the key line has no quotes or extra spaces).

## 3. Run the frontend

Simplest option — just open the file directly:

```bash
open frontend/index.html      # macOS
# or double-click frontend/index.html in your file explorer
```

Or serve it so relative paths behave exactly like a real deployment:

```bash
cd frontend
npx serve .
```

The frontend calls the backend at `http://localhost:3001` by default. If you run the
backend on a different port or host, set this before the page's own script runs — add
a line like this near the top of `frontend/index.html`, just before the closing
`</body>` tag's `<script>` block:

```html
<script>window.PROMPTMASTER_API_BASE = 'http://localhost:4000';</script>
```

## Using it

Type a question about prompting or AI engineering into the Prompt Analyzer box on the
hero section (or tap one of the example chips), and hit the arrow or press Enter.
The backend sends it to Claude and the answer streams into the widget. Questions are
kept to a running history (session-only, click one to re-ask it) and you can copy the
latest Q&A pair with the share icon.

The backend's system prompt keeps answers scoped to prompting/AI-engineering topics —
off-topic questions get a polite redirect instead of a real answer.

## Deploying for real visitors

Running both parts on your own laptop is fine for testing, but if this goes live on a
real domain:

- Deploy `backend/` somewhere that can hold an environment variable securely (Render,
  Fly.io, a small VPS, etc.) and set `ANTHROPIC_API_KEY` there — never in the repo.
- Update `window.PROMPTMASTER_API_BASE` in the frontend to point at that backend's public URL.
- Add rate limiting (e.g. `express-rate-limit`) in front of `/api/ask` so the key can't be
  drained by automated traffic — the current server has none.
- Serve the frontend as static files from any host (Netlify, Vercel, S3, GitHub Pages, etc.).

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| "Failed to reach the AI service" | Backend isn't running, or `ANTHROPIC_API_KEY` is missing/invalid |
| CORS error in browser console | Backend isn't running on the URL set in `PROMPTMASTER_API_BASE` |
| `hasKey: false` at `/api/health` | `.env` missing, misnamed, or not in `backend/` |
| Answers seem cut off | `max_tokens` in `backend/server.js` is capped at 500 — raise it if needed |
