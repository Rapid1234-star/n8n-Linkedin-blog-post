# 📰 Blog to LinkedIn

A new blog post goes in. A LinkedIn caption and a custom illustration come out. The workflow only picks an article it has not posted before, then stops on a preview until you say otherwise.

The sheet remembers every article. Monday-you does not.

`testMode` starts as `true`, so the first run will not publish.

## 🎬 Demo

<video src="https://github.com/Rapid1234-star/n8n-Linkedin-blog-post/raw/main/demo.mp4" controls width="720"></video>

## 🗺️ How it runs

![One new article becomes one LinkedIn post](./flow.svg)

1. **Read** the blog feed.
2. **Skip** anything already in the Google Sheet.
3. **Write** the caption with Groq. Gemini tries if Groq fails. If both fail, a short fallback uses the article title and summary.
4. **Draw** a clean illustration with Cloudflare.
5. **Preview** it in the browser.
6. **Publish** to LinkedIn and log the article. This happens only after you set `testMode` to `false` in **Prepare Blog Data**.

## ✍️ The writing prompts

The caption prompts in **Generate Post - Groq**, **Generate Post - Gemini**, and **Generate Post - Ollama** are already engineered for a short, practical LinkedIn post. Open any of those nodes and edit the prompt if you want a different voice, length, or industry. The image brief is in **Generate Image Concept** and **Generate Image Concept - Gemini Fallback**. Same idea: it is ready, and you can still tune it.

## 🧩 What to fill in

Import `Wix Blog to LinkedIn v91.4 FINAL.json`. Replace each placeholder, then save.

### Blog feed

In **RSS Read**, replace `https://your-website.com/blog-feed.xml`.

- Wix: `https://your-site.com/blog-feed.xml`
- WordPress: `https://your-site.com/feed`
- Ghost: `https://your-site.com/rss`

Open that URL in a browser first. You should see article titles, not a normal webpage.

### LinkedIn ids

In **Prepare Blog Data**:

- `YOUR_PERSONAL_URN` is the id inside `urn:li:person:YOUR_PERSONAL_URN`. After LinkedIn is connected in n8n, call `GET https://api.linkedin.com/v2/userinfo` and use the `sub` value.
- `YOUR_COMPANY_URN` is the number in `https://www.linkedin.com/company/YOUR_COMPANY_URN/admin/`.

`postAsOrg` is `true`, so the post goes to the company page. Set it to `false` to post from the personal profile.

### Google Sheet

Make a sheet tab named `Sheet1` with these headers in row 1:

`dedupeKey` · `latestTitle` · `latestLink` · `processedAt`

The id is the text between `/d/` and `/edit` in the sheet URL. Paste it over `YOUR_SHEET_ID` in **Google Sheets Read Dedupe** and **Google Sheets Append Dedupe**.

### Cloudflare account id

In **Cloudflare Generate Image**, replace `YOUR_CLOUDFLARE_ACCOUNT_ID` in the URL. In the Cloudflare dashboard it is the id after `dash.cloudflare.com/`. That is the account id, not the API token.

### Company name

Leave `companyNameOverride` blank in **Prepare Blog Data**. The name comes from the article link, so `acme.com` becomes Acme. Set it only when the domain is wrong, such as `hpe.com` when the name should be `HP`.

The caption is written from the article, not from the company name.

## 🔑 API keys

None of these keys are in the JSON file. Create them in each service, then attach them in n8n when the node asks for a credential.

| Service | Create it here | Attach it in n8n |
| --- | --- | --- |
| Groq | [console.groq.com](https://console.groq.com/keys) | Groq credential on both Groq chat models |
| Gemini | [Google AI Studio](https://aistudio.google.com/apikey) | Gemini credential on both Gemini chat models |
| Cloudflare | Cloudflare dashboard → API tokens | Header Auth on **Cloudflare Generate Image**. Header name `Authorization`, value `Bearer YOUR_CLOUDFLARE_API_TOKEN`. The token needs Workers AI. |
| LinkedIn | A LinkedIn developer app, then the n8n LinkedIn credential | OAuth on the post and image upload nodes. This is a login, not a key pasted into the file. |
| Google Sheets | The n8n Google Sheets credential | OAuth on the read and append nodes. The Google account must be able to edit the sheet. |

Ollama is optional. Leave it unconnected if you only use Groq and Gemini.
