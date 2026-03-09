# Genora Lite - Photo Edit Pro

## What This Is
AI-powered photo editor web app using Google's Gemini 3 Pro Image model (`gemini-3-pro-image-preview`). Users upload a photo, describe edits in natural language, and get AI-edited results with face/identity preservation.

## Architecture
Single-page static HTML app + Vercel serverless function. No build step, no framework.

```
genora-lite/
├── index.html          # Full app UI (HTML + CSS + JS, all-in-one)
├── api/
│   └── edit.js         # Vercel serverless function - proxies Gemini API calls
├── .env                # Local env file (GEMINI_API_KEY) - git-ignored
├── .gitignore
└── photo-editor (6).html  # Original source file (reference only, not deployed)
```

## Key Design Decisions

### Server-side API key
The Gemini API key is **never exposed to the frontend**. The frontend calls `/api/edit` which proxies to Google's Gemini API using `process.env.GEMINI_API_KEY`. This was an intentional change from the original file which had client-side API key input.

### No build system
This is a zero-config static site on Vercel. The `index.html` is served directly. The only server-side piece is the `api/edit.js` serverless function.

### Body size limit
`api/edit.js` has a 20MB body size limit configured to handle large base64-encoded images. However, Vercel's infrastructure enforces a ~4.5MB limit on serverless payloads, so images are compressed client-side before sending.

### Client-side image compression
Uploaded images are resized to max 2048px and JPEG-compressed at 85% quality before sending to the API. This keeps payloads under Vercel's limit without affecting download quality (Gemini generates fresh output at the selected resolution).

### Single-turn editing
Each edit sends only the latest image (original or last edited) plus the new prompt. Conversation history is not accumulated across turns to avoid payload bloat. Download quality is unaffected — edited images are served exactly as Gemini returns them.

## How the App Works
1. User uploads a photo (resized to max 2048px, JPEG-compressed at 85% client-side)
2. User types editing instructions (e.g., "change the dress to red")
3. Frontend sends `{ contents, resolution }` to `/api/edit` (single-turn: latest image + prompt only)
4. Serverless function forwards to Gemini API with the server-side API key
5. Response (with generated image) is returned to frontend
6. Subsequent edits use the last edited image as input (no history accumulation)
7. Resolution options: 2K (default) or 4K

## Prompt Engineering
Every user prompt is enhanced with extensive face/identity preservation rules before being sent to Gemini. This includes instructions for facial identity, color/composition, lighting, and quality preservation.

## Deployment
- **GitHub**: https://github.com/DamilolaEmmnauel/genora-lite
- **Vercel**: https://genora-lite.vercel.app
- **Auto-deploy**: Pushes to `main` trigger automatic Vercel deployments
- **Env var**: `GEMINI_API_KEY` must be set in Vercel project settings

## Common Tasks
- To update the app: edit `index.html` and/or `api/edit.js`, commit, push to `main`
- To change the Gemini model: update `modelName` in `api/edit.js`
- To adjust prompt enhancement: edit the `enhancedPrompt` template string in `index.html`
