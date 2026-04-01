# NOBL Body Condition Score App
## Deployment Guide — Netlify + Shopify

---

## What's in this package

```
nobl-bcs/
├── bcs-app.html                  ← Main app (paste into Shopify)
├── netlify.toml                  ← Netlify configuration
├── netlify/
│   └── functions/
│       └── analyze.js            ← Secure API proxy (keeps your key hidden)
└── README.md
```

---

## Step 1 — Deploy the proxy to Netlify

The proxy keeps your Anthropic API key out of the browser.

### 1a. Create a GitHub repo
- Go to github.com → New repository
- Name it `nobl-bcs-proxy` (or anything you like)
- Upload the entire contents of this folder

### 1b. Connect to Netlify
1. Go to app.netlify.com → "Add new site" → "Import an existing project"
2. Choose GitHub and select your repo
3. Build settings: leave all blank (no build command needed)
4. Click **Deploy site**

### 1c. Add your API key (secret — never share this)
1. In Netlify, go to **Site configuration → Environment variables**
2. Click **Add a variable**
3. Key: `ANTHROPIC_API_KEY`
4. Value: your Anthropic API key (starts with `sk-ant-...`)
5. Click **Save**

### 1d. Redeploy
- Go to **Deploys** → click **Trigger deploy** → **Deploy site**
- Your proxy URL will be something like:
  `https://your-site-name.netlify.app/.netlify/functions/analyze`

---

## Step 2 — Update the API endpoint in bcs-app.html

Open `bcs-app.html` and find this line near the top of the `<script>` section:

```javascript
const API_ENDPOINT = "/.netlify/functions/analyze";
```

Replace it with your full Netlify proxy URL:

```javascript
const API_ENDPOINT = "https://your-site-name.netlify.app/.netlify/functions/analyze";
```

---

## Step 3 — Add to Shopify

### Option A — Custom page (recommended)
1. In Shopify admin, go to **Online Store → Pages**
2. Click **Add page**
3. Set the title (e.g. "Body Condition Score")
4. In the content editor, click the `<>` (HTML) button
5. Paste the **entire contents** of `bcs-app.html`
6. Save the page

### Option B — Theme section
1. In Shopify admin, go to **Online Store → Themes → Customize**
2. Add a new **Custom HTML** section
3. Paste the contents of `bcs-app.html`

### Option C — Dedicated landing page via theme code
1. Go to **Online Store → Themes → Edit code**
2. Under **Templates**, click **Add a new template**
3. Choose `page` type, name it `bcs`
4. Replace the template content with the HTML from `bcs-app.html`
5. Assign the template to your BCS page

---

## How the photo AI analysis works

1. User uploads a side photo (and optionally a top-down photo)
2. Photos are converted to base64 in the browser — **they never touch your server**
3. The base64 image data is sent to your Netlify proxy along with questionnaire answers
4. The proxy forwards everything to Claude's vision API (claude-sonnet-4-20250514)
5. Claude analyzes the photos + answers and returns:
   - Specific visual observations from the photos
   - Estimated weight to lose or gain (in lbs and kg)
   - Realistic timeline to reach BCS 5
   - Personalized guidance for that dog
6. Results display on screen in seconds

---

## Shopify-specific notes

- **CORS**: The Netlify proxy is configured to allow requests from any origin, so Shopify pages can call it freely
- **No Shopify app needed**: This is pure HTML/JS — no app installation required
- **Theme conflicts**: If your Shopify theme loads CSS globally that affects inputs or buttons, you may need to scope the styles. Wrap all CSS selectors with `.nobl-bcs-app` and wrap the outer div with `<div class="nobl-bcs-app">` if needed
- **Mobile**: The app is fully responsive down to 375px

---

## Customization

| What               | Where in bcs-app.html                    |
|--------------------|------------------------------------------|
| Brand name/logo    | `.nobl-header` section at top of HTML    |
| Colors             | `:root` CSS variables at top of `<style>`|
| API endpoint URL   | `const API_ENDPOINT = "..."` in script   |
| Vet disclaimer     | `.vet-note` paragraph at bottom          |
| Recommendations    | `const RECS = { ... }` object in script  |

---

## Troubleshooting

**"API key not configured on server"**
→ You haven't added `ANTHROPIC_API_KEY` to Netlify environment variables yet, or you need to redeploy after adding it.

**Photos aren't being analyzed**
→ Check browser console for CORS errors. Make sure `API_ENDPOINT` is set to the full Netlify URL, not the relative path.

**Results show without photo insights**
→ The AI response JSON may have failed to parse. Check the browser console for errors and make sure your proxy is returning a 200 response.

**Shopify strips my HTML**
→ Some Shopify themes sanitize HTML in the page editor. Use the theme code editor (Option C above) to embed the full file instead.
