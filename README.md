# 📡 Trend Scout

Find trending dropshipping products every day — powered by Claude AI + live web search.

## What it does
- Searches Google Trends, Reddit, TikTok, Amazon in real time
- Scores each product by viral velocity (trend score)
- Shows category, competition level, price range, and why it's trending
- Caches results for 3 hours so it's instant to reopen
- Auto-refreshes daily via GitHub Actions

## Setup (5 minutes)

### 1. Fork / clone this repo
```bash
git clone https://github.com/YOUR_USERNAME/trend-scout.git
cd trend-scout
npm install
npm run dev
```

### 2. Deploy to GitHub Pages
1. Push to GitHub
2. Go to **Settings → Pages**
3. Under "Source", select **GitHub Actions**
4. Push any change to `main` — it auto-deploys

Your app will be live at: `https://YOUR_USERNAME.github.io/trend-scout/`

### 3. Enable daily auto-refresh
The `.github/workflows/deploy.yml` already includes a cron job that rebuilds the site every day at 8am UTC (3am CST). No extra setup needed — it runs automatically once GitHub Pages is enabled.

> **Note:** The app uses the Anthropic API. This is handled by the Claude.ai artifact proxy when running inside Claude. If you want to run it standalone outside Claude.ai, you'll need to add your own `ANTHROPIC_API_KEY` to the fetch headers.

## Customization
- Change the cron schedule in `.github/workflows/deploy.yml`
- Edit the AI prompt in `src/App.jsx` (`fetchTrends` function) to focus on specific niches
- Adjust `CACHE_TTL` to control how often it re-fetches

## Stack
- React + Vite
- Anthropic Claude API (claude-sonnet-4) with web_search tool
- Unsplash for product images
- GitHub Pages for hosting
- GitHub Actions for CI/CD
