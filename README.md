# Accelerated Jam Tracks — acceleratedjamtracks.com

The website for the **Accelerated Jam Tracks** YouTube channel: the logo and a responsive
grid of jam-track videos. Plain HTML, CSS, and a little JavaScript — no framework, no build
step, no database, no tracking. The grid is generated in the browser from
`public/videos.json`, so adding a video is a one-line data edit.

Hosted on **Cloudflare Workers** (static assets) and deployed automatically from GitHub:
push to `main`, and the live site updates.

## Project structure

```text
AJT Website/
├── public/                 # everything here is served as-is at the site root
│   ├── index.html          # page markup + the small script that builds the grid
│   ├── styles.css          # all styling
│   ├── videos.json         # ← the video list (edit this to add / remove videos)
│   ├── ajt-logo.png        # white logo, transparent background
│   ├── favicon.ico
│   ├── favicon-32.png
│   └── apple-touch-icon.png
├── wrangler.jsonc          # Cloudflare Workers config (serves ./public)
├── package.json
├── .gitignore
└── README.md
```

## Adding a video (the normal update)

This is built so you can hand an AI coding agent a single instruction:

> **Add this video to Accelerated Jam Tracks: `<YouTube URL>`**

The agent (or you) then:

1. Open `public/videos.json`.
2. Add one object to the array with the video's `title` and `youtubeUrl`. Newest videos
   appear first automatically (by `publishedAt`), so there's nothing to reorder.
3. *(Optional)* Preview locally — `npm install`, then `npm run dev`, and open the URL it
   prints.
4. Commit and push to `main`:
   ```bash
   git add public/videos.json
   git commit -m "Add <title>"
   git push
   ```
5. Cloudflare deploys automatically within a minute or two. Done.

### `videos.json` format

Each entry supports these fields:

| Field          | Required | Notes                                                                                   |
| -------------- | -------- | --------------------------------------------------------------------------------------- |
| `title`        | yes      | Shown on the card, e.g. `"Hey Joe"`.                                                     |
| `youtubeUrl`   | yes      | Full watch URL. `watch?v=…`, `youtu.be/…`, and `shorts/…` all work.                     |
| `tempoText`    | no       | Small line under the title, e.g. `"50–85 BPM"`.                                          |
| `publishedAt`  | no       | `YYYY-MM-DD`. Controls ordering (newest first).                                         |
| `thumbnailUrl` | no       | Override only. If omitted, the YouTube thumbnail is fetched automatically.               |
| `sortOrder`    | no       | Manual override. If **any** entry has one, the whole list sorts by it (ascending) instead of by date. |

Minimal example — this is all that's needed to add a video:

```json
{
  "title": "Crazy Train",
  "tempoText": "120–160 BPM",
  "youtubeUrl": "https://www.youtube.com/watch?v=XXXXXXXXXXX",
  "publishedAt": "2026-09-10"
}
```

The thumbnail is pulled from YouTube automatically (`maxresdefault`, falling back to
`hqdefault`) using the video ID in `youtubeUrl`, so you normally don't set `thumbnailUrl`.

## Preview locally

```bash
npm install
npm run dev     # wrangler prints a local URL, usually http://localhost:8787
```

Resize the window (or use the browser's device toolbar at ~390px) to check the phone
layout: three cards per row on desktop, two on tablets, one on phones, and no sideways
scrolling.

## How deployment works

- The site is a Cloudflare **Worker** named `accelerated-jam-tracks` that serves the
  `public/` folder as static files (see `wrangler.jsonc`).
- It's connected to this GitHub repo via **Workers Builds**: every push to `main` runs a
  deploy (`npx wrangler deploy`) — no dashboard needed for normal updates.
- `acceleratedjamtracks.com` points at this Worker, with HTTPS handled by Cloudflare.

### One-time setup: existing Worker → Git auto-deploy

If the Git connection isn't wired up yet:

1. **GitHub** — create a repository and push this folder to it (`main` branch).
2. **Cloudflare** — dashboard → **Workers & Pages** → **accelerated-jam-tracks** →
   **Settings** → **Builds** → connect the GitHub repo (authorize the Cloudflare app when
   prompted). Set **production branch** = `main`, **build command** = *(leave empty)*,
   **deploy command** = `npx wrangler deploy`.
3. Push a commit to `main` and confirm it deploys. From then on, every push deploys.

A manual deploy is always available with `npm run deploy` (after `npx wrangler login` once).
