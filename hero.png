import { useState, useEffect } from "react"
import "./App.css"

const CACHE_KEY = "trend_scout_cache"
const CACHE_TTL = 3 * 60 * 60 * 1000 // 3 hours

function getCache() {
  try {
    const raw = localStorage.getItem(CACHE_KEY)
    if (!raw) return null
    const { ts, data } = JSON.parse(raw)
    if (Date.now() - ts > CACHE_TTL) return null
    return data
  } catch { return null }
}

function setCache(data) {
  try {
    localStorage.setItem(CACHE_KEY, JSON.stringify({ ts: Date.now(), data }))
  } catch {}
}

async function fetchTrends(forceRefresh = false) {
  if (!forceRefresh) {
    const cached = getCache()
    if (cached) return cached
  }

  const prompt = `You are a dropshipping product researcher. Search the web RIGHT NOW for trending products people are buying online today. 

Look at:
- Google Trends rising searches today
- Reddit r/dropship, r/entrepreneur, r/flipping hot posts this week  
- TikTok Shop trending products (search for recent articles/posts about this)
- Amazon Movers & Shakers
- Any viral product videos or "where to buy" searches spiking

Return EXACTLY a JSON array of 12 trending products. No markdown, no explanation, ONLY valid JSON.

Format:
[
  {
    "name": "Product name",
    "category": "Category",
    "trendScore": 92,
    "momentum": "🔥 Exploding" | "📈 Rising" | "👀 Emerging",
    "whyTrending": "One sentence: why this is trending right now",
    "searchSignal": "e.g. +340% search volume this week",
    "avgPrice": "$XX-$XX",
    "competition": "Low" | "Medium" | "High",
    "source": "Where you saw this trending (e.g. TikTok, Reddit, Google Trends)",
    "imageQuery": "a short 3-5 word Google image search query to find a clean product photo"
  }
]

Score trendScore 1-100 based on viral velocity. Sort by trendScore descending. Be specific and real — use actual products you find trending TODAY, not generic examples.`

  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      model: "claude-sonnet-4-20250514",
      max_tokens: 4000,
      tools: [{ type: "web_search_20250305", name: "web_search" }],
      messages: [{ role: "user", content: prompt }]
    })
  })

  const data = await response.json()
  
  const textBlock = data.content?.find(b => b.type === "text")
  if (!textBlock) throw new Error("No response from AI")
  
  let json = textBlock.text.trim()
  json = json.replace(/```json\n?/g, "").replace(/```\n?/g, "").trim()
  
  const startIdx = json.indexOf("[")
  const endIdx = json.lastIndexOf("]")
  if (startIdx !== -1 && endIdx !== -1) {
    json = json.slice(startIdx, endIdx + 1)
  }
  
  const products = JSON.parse(json)
  setCache(products)
  return products
}

const UNSPLASH_QUERIES = {
  default: "product ecommerce minimal white background"
}

function getImageUrl(query) {
  const encoded = encodeURIComponent(query + " product white background")
  return `https://source.unsplash.com/400x300/?${encoded}`
}

function ScoreBadge({ score }) {
  const color = score >= 85 ? "#e8533a" : score >= 70 ? "#e8933a" : "#3a8ee8"
  return (
    <div style={{
      background: color,
      color: "white",
      borderRadius: "20px",
      padding: "2px 10px",
      fontSize: "13px",
      fontWeight: 600,
      display: "inline-flex",
      alignItems: "center",
      gap: "4px"
    }}>
      {score}
    </div>
  )
}

function CompetitionDot({ level }) {
  const colors = { Low: "#22c55e", Medium: "#f59e0b", High: "#ef4444" }
  return (
    <span style={{ display: "inline-flex", alignItems: "center", gap: "5px" }}>
      <span style={{
        width: "7px", height: "7px", borderRadius: "50%",
        background: colors[level] || "#888",
        display: "inline-block"
      }} />
      {level}
    </span>
  )
}

function ProductCard({ product, index }) {
  const [imgSrc, setImgSrc] = useState(null)
  const [imgLoaded, setImgLoaded] = useState(false)

  useEffect(() => {
    const query = encodeURIComponent((product.imageQuery || product.name) + " product")
    setImgSrc(`https://source.unsplash.com/400x260/?${query}`)
  }, [product])

  return (
    <div className="product-card" style={{ animationDelay: `${index * 60}ms` }}>
      <div className="card-img-wrap">
        {imgSrc && (
          <img
            src={imgSrc}
            alt={product.name}
            onLoad={() => setImgLoaded(true)}
            onError={() => {
              setImgSrc(`https://source.unsplash.com/400x260/?${encodeURIComponent(product.category + " product")}`)
            }}
            style={{
              width: "100%", height: "100%", objectFit: "cover",
              opacity: imgLoaded ? 1 : 0,
              transition: "opacity 0.4s ease"
            }}
          />
        )}
        {!imgLoaded && (
          <div style={{
            position: "absolute", inset: 0,
            background: "linear-gradient(135deg, #f5f5f5 0%, #e8e8e8 100%)",
            display: "flex", alignItems: "center", justifyContent: "center",
            color: "#bbb", fontSize: "32px"
          }}>
            📦
          </div>
        )}
        <div className="card-momentum">{product.momentum}</div>
        <div className="card-score-badge">
          <ScoreBadge score={product.trendScore} />
        </div>
      </div>

      <div className="card-body">
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", gap: "8px", marginBottom: "6px" }}>
          <h3 style={{ margin: 0, fontSize: "15px", fontWeight: 600, lineHeight: 1.3, color: "var(--text-primary)" }}>{product.name}</h3>
        </div>

        <div style={{ fontSize: "12px", color: "var(--text-muted)", marginBottom: "8px", fontWeight: 500 }}>
          {product.category}
        </div>

        <p style={{ fontSize: "13px", color: "var(--text-secondary)", margin: "0 0 10px", lineHeight: 1.5 }}>
          {product.whyTrending}
        </p>

        <div style={{
          background: "var(--signal-bg)",
          borderRadius: "6px",
          padding: "6px 10px",
          fontSize: "12px",
          color: "var(--signal-text)",
          fontWeight: 500,
          marginBottom: "10px"
        }}>
          📊 {product.searchSignal}
        </div>

        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", fontSize: "12px" }}>
          <div style={{ color: "var(--text-muted)" }}>
            💰 <span style={{ color: "var(--text-primary)", fontWeight: 500 }}>{product.avgPrice}</span>
          </div>
          <div style={{ color: "var(--text-muted)" }}>
            Competition: <CompetitionDot level={product.competition} />
          </div>
        </div>

        <div style={{ borderTop: "1px solid var(--border)", marginTop: "10px", paddingTop: "8px", fontSize: "11px", color: "var(--text-muted)" }}>
          via {product.source}
        </div>
      </div>
    </div>
  )
}

function FilterBar({ filter, setFilter, sortBy, setSortBy }) {
  const categories = ["All", "🔥 Exploding", "📈 Rising", "👀 Emerging"]
  const sorts = [
    { label: "Trend score", value: "score" },
    { label: "Low competition", value: "competition" },
    { label: "Price: low", value: "price" },
  ]

  return (
    <div className="filter-bar">
      <div className="filter-chips">
        {categories.map(c => (
          <button
            key={c}
            className={`chip ${filter === c ? "chip-active" : ""}`}
            onClick={() => setFilter(c)}
          >
            {c}
          </button>
        ))}
      </div>
      <select
        value={sortBy}
        onChange={e => setSortBy(e.target.value)}
        className="sort-select"
      >
        {sorts.map(s => (
          <option key={s.value} value={s.value}>{s.label}</option>
        ))}
      </select>
    </div>
  )
}

function LoadingState() {
  const steps = [
    "Searching Google Trends...",
    "Scanning Reddit communities...",
    "Checking TikTok viral products...",
    "Analyzing Amazon Movers & Shakers...",
    "Scoring trend velocity..."
  ]
  const [step, setStep] = useState(0)

  useEffect(() => {
    const iv = setInterval(() => setStep(s => (s + 1) % steps.length), 2200)
    return () => clearInterval(iv)
  }, [])

  return (
    <div style={{ textAlign: "center", padding: "80px 20px" }}>
      <div className="spinner" />
      <p style={{ color: "var(--text-secondary)", marginTop: "24px", fontSize: "15px", transition: "all 0.3s" }}>
        {steps[step]}
      </p>
      <p style={{ color: "var(--text-muted)", fontSize: "13px", marginTop: "8px" }}>
        Searching the web in real time — takes ~30 seconds
      </p>
    </div>
  )
}

function LastUpdated({ cached }) {
  const cached_ts = (() => {
    try {
      const raw = localStorage.getItem(CACHE_KEY)
      if (!raw) return null
      return JSON.parse(raw).ts
    } catch { return null }
  })()

  if (!cached_ts) return null

  const diff = Math.round((Date.now() - cached_ts) / 60000)
  const label = diff < 1 ? "just now" : diff < 60 ? `${diff}m ago` : `${Math.round(diff / 60)}h ago`

  return (
    <span style={{ fontSize: "12px", color: "var(--text-muted)" }}>
      {cached ? `Cached · updated ${label}` : `Live data · ${label}`}
    </span>
  )
}

export default function App() {
  const [products, setProducts] = useState([])
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState(null)
  const [filter, setFilter] = useState("All")
  const [sortBy, setSortBy] = useState("score")
  const [fromCache, setFromCache] = useState(false)

  async function loadTrends(force = false) {
    setLoading(true)
    setError(null)
    try {
      const cached = !force && getCache()
      const data = cached || await fetchTrends(force)
      setProducts(data)
      setFromCache(!!cached)
    } catch (e) {
      setError("Couldn't fetch trends. Check your connection and try again.")
      console.error(e)
    } finally {
      setLoading(false)
    }
  }

  useEffect(() => { loadTrends() }, [])

  const filtered = products
    .filter(p => filter === "All" || p.momentum === filter)
    .sort((a, b) => {
      if (sortBy === "score") return b.trendScore - a.trendScore
      if (sortBy === "competition") {
        const order = { Low: 0, Medium: 1, High: 2 }
        return (order[a.competition] ?? 1) - (order[b.competition] ?? 1)
      }
      if (sortBy === "price") {
        const getPrice = p => parseInt((p.avgPrice || "").replace(/[^0-9]/g, "") || "999")
        return getPrice(a) - getPrice(b)
      }
      return 0
    })

  return (
    <div className="app">
      <header className="header">
        <div className="header-inner">
          <div>
            <h1 className="logo">📡 Trend Scout</h1>
            <p className="tagline">Trending dropship products — found while you sleep</p>
          </div>
          <div style={{ display: "flex", alignItems: "center", gap: "12px", flexWrap: "wrap" }}>
            <LastUpdated cached={fromCache} />
            <button
              className="refresh-btn"
              onClick={() => loadTrends(true)}
              disabled={loading}
            >
              {loading ? "Scanning..." : "↻ Refresh"}
            </button>
          </div>
        </div>
      </header>

      <main className="main">
        {!loading && products.length > 0 && (
          <FilterBar filter={filter} setFilter={setFilter} sortBy={sortBy} setSortBy={setSortBy} />
        )}

        {loading && <LoadingState />}

        {error && (
          <div style={{
            background: "#fef2f2", border: "1px solid #fecaca", borderRadius: "10px",
            padding: "20px", textAlign: "center", color: "#dc2626", margin: "40px auto", maxWidth: "400px"
          }}>
            <p style={{ margin: 0, fontWeight: 500 }}>{error}</p>
            <button className="refresh-btn" style={{ marginTop: "12px" }} onClick={() => loadTrends(true)}>
              Try again
            </button>
          </div>
        )}

        {!loading && filtered.length > 0 && (
          <>
            <div style={{ marginBottom: "16px", fontSize: "13px", color: "var(--text-muted)" }}>
              {filtered.length} products
            </div>
            <div className="grid">
              {filtered.map((p, i) => (
                <ProductCard key={p.name + i} product={p} index={i} />
              ))}
            </div>
          </>
        )}

        {!loading && products.length > 0 && filtered.length === 0 && (
          <p style={{ textAlign: "center", color: "var(--text-muted)", padding: "60px 0" }}>
            No products match this filter right now.
          </p>
        )}
      </main>

      <footer style={{ textAlign: "center", padding: "40px 20px", color: "var(--text-muted)", fontSize: "12px" }}>
        Trend Scout · Powered by Claude AI + live web search · Data refreshes every 3 hours
      </footer>
    </div>
  )
}
