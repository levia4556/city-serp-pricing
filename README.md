# City SERP Pricing: Geekflare Cheaper Cost Analysis Actually Shows How Local Results Vary — Which City Provider Delivers Accurate Data For Developers? (Full Pricing Breakdown - Free Tier)

When you need to pull Google's search results for a specific city—organic listings, local packs, news, images, **People Also Ask** (PAA) blocks, and the new Google AI Overview—most APIs either charge per-page, hide the credit math, or force you into a "pay-as-you-go" nightmare that explodes the bill the moment you add a new city. Geekflare built its Search API to give you **predictable credit consumption**, **city-level geo-targeting** across 65,000+ locations, and a single, honest pricing table that lets you size a production pipeline without a surprise invoice. This README walks you through everything a backend engineer needs to know: how the API works, quick-start code, credit calculations, plan selection, and a side-by-side look at the three biggest competitors. All numbers, URLs, and credit rules are taken directly from Geekflare's official docs, so you can trust the math and focus on building, not on chasing billing support.

---  

## How Geekflare Approaches City-Level SERP API Without Multiplier Surprises

Geekflare's Search endpoint (`POST https://api.geekflare.com/search`) is deliberately simple: **four mutually exclusive modes** that each have a fixed total credit cost. There is no hidden "base-2-plus-extra" fee; you pay exactly what the mode says.

| Mode | What you get | Total credits per call |
|------|--------------|------------------------|
| **Standard web search** | Basic organic results only | **2** |
| **Full Google SERP JSON** (`serp: true`) | Structured JSON for page 1: organic results, ads, Google AI Overview, **People Also Ask**, related searches, local pack, news, images, popular products | **2** |
| **Search + Scrape** (`scrape: true`) | Everything in SERP mode **plus** raw HTML of the top 1-10 result pages (JS rendering, CAPTCHA handling, anti-bot bypass included) | **4** |
| **Grounded AI Answer** (`groundedAnswer: true`) | AI-generated answer with inline citations, plus the full SERP payload | **5** |

All modes share the same **city-level geo-targeting** syntax:  

```json
{
  "city": "Austin,Texas,United States",
  "device": "desktop"
}
```

You can also specify a plain ISO country (`location`) or a device type (`mobile`). No extra credit charge for filters like `source`, `time`, `category`, `includeDomains`, or `excludeDomains`. Output can be `json`, `markdown`, or `html`—again at no extra cost.

**Rate limits** are enforced per-second, not per-connection: 1 req/s on Free, 5 req/s on Starter, 10 req/s on Growth, 25 req/s on Business, and up to **1,000 req/s** on Enterprise. This makes it easy to calculate throughput: a Growth plan can safely fire 10 requests every second, consuming at most 50 credits per second in SERP + Scrape mode (10 req × 4 credits).

Geekflare also guarantees **failed-request credit immunity**: any 5xx error or capture that returns no data costs **0 credits**. Alerts fire at 90% usage, and the system pauses at 100% until you add a credit pack or upgrade—no hidden throttling.

---

## Quickstart Implementation You Can Drop Into Your Repo

Below are two minimal, runnable examples. The first uses plain **cURL** with the Managed Cloud Platform (MCP) JSON relay, which lets you keep your API key out of source control. The second shows the same request using the official **Node SDK** (`@geekflare/api-node`). Both target a city-level SERP request for "best coffee shops" in **Portland, Oregon, United States**, pulling the full SERP JSON (including PAA) and the top-3 result pages.

### 1️⃣ cURL via MCP

```bash
curl -X POST https://mcp.geekflare.com/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://api.geekflare.com/search",
    "method": "POST",
    "headers": {
      "x-api-key": "YOUR_GEKKFLARE_API_KEY"
    },
    "body": {
      "query": "best coffee shops",
      "city": "Portland,Oregon,United States",
      "device": "desktop",
      "serp": true,
      "scrape": true,
      "maxResults": 3,
      "output": "json"
    }
  }'
```

*What happens*: MCP forwards the request to the Search API, adds the required `x-api-key` header, and returns the raw JSON. The call consumes **4 credits** (SERP + Scrape). If you run this 1,000 times in a month, you'll spend 4,000 credits—well inside the Starter tier's 10,000-credit allowance.

### 2️⃣ Node SDK Example

```js
const { GeekflareClient } = require('@geekflare/api-node');

const client = new GeekflareClient({
  apiKey: process.env.GEEKFLARE_API_KEY   // keep it out of repo
});

(async () => {
  const resp = await client.search({
    query: 'best coffee shops',
    city: 'Portland,Oregon,United States',
    device: 'desktop',
    serp: true,
    scrape: true,
    maxResults: 3,
    output: 'json'
  });
  console.log(JSON.stringify(resp, null, 2));
})();
```

The SDK abstracts the HTTP layer, but the credit consumption is identical: **4 credits** per call. Both examples illustrate the exact parameters you need for city-level SERP extraction, including the PAA block that lives under `data.paa`.

---

## Companion Free Tools and Workflow Integrations

Geekflare's ecosystem is built for engineers who don't want to reinvent the wheel. Here's what's available out of the box:

| Category | Offering | How to use |
|----------|----------|------------|
| **SDKs** | Python (`pip install geekflare-api`), Node/TS (`npm install @geekflare/api-node`) | Import the client, set `GEekflare_API_KEY` env var, call `client.search()` |
| **MCP Server** | Hosted at `https://mcp.geekflare.com/mcp` (OAuth protected) or local `npx -y @geekflare/mcp` | 18 built-in tools (`search`, `webScrape`, `screenshot`, …) accessible via a single JSON endpoint |
| **AI Frameworks** | LangChain (`pip install langchain-geekflare`), LlamaIndex (`pip install llama-index-tools-geekflare`), Vercel AI SDK (`@geekflare/ai-sdk`) | Plug the Search tool into your LLM pipelines for grounded answers |
| **No-Code** | n8n node, Zapier integration, Make.com connector, Google Sheets add-on (named only) | Drag-and-drop "Geekflare Search" into a workflow, map `city` and `query` fields |
| **Free Utilities** | SERP Checker, URL-to-PDF, Webpage Screenshot, Webpage-to-Markdown | Quick sanity checks before you fire the paid API; all hosted on `geekflare.com/tools/` |

All of these integrations point to the same credit pool, so you can mix and match without double-billing. For example, you could use the free SERP Checker to validate query syntax, then hand off to the paid Search API for the full JSON payload.

---

## The Credit Math Nobody Puts in the Headline

Credit budgeting is where most APIs trip you up. Below we compare Geekflare's fixed-cost model to three well-known competitors, using realistic monthly workloads.

**Example Workloads**

| Monthly volume | Queries (city-level SERP) | Credits needed (SERP + Scrape) |
|----------------|--------------------------|--------------------------------|
| 1,000 queries  | 1,000                     | 4,000 credits |
| 5,000 queries  | 5,000                     | 20,000 credits |
| 20,000 queries | 20,000                    | 80,000 credits |
| 100,000 queries| 100,000                   | 400,000 credits |

**Geekflare Cost Breakdown (per-credit rates)**

| Plan | Monthly price | Credits included | Effective $/1K credits (SERP + Scrape)  | Link |
|------|---------------|------------------|---------------------------------------- | --- |
| Free | $0            | 500              | N/A (exceeds quickly)  | [Start Free](https://geekflare.com/pricing/) |
| Starter | $19 | 10,000 | $7.60/1K  | [Get Starter](https://geekflare.com/pricing/) |
| Growth | $69 (or $58 billed yearly) | 100,000 | $2.76/1K  | [Get Growth](https://geekflare.com/pricing/) |
| Business | $349 | 1,000,000 | $0.70/1K  | [Get Business](https://geekflare.com/pricing/) |

**Scenario 1 – 5,000 city-level SERP + scrape calls**  
- Credits: 20,000 → fits comfortably in **Growth** (100,000 credits).  
- Cost: $69/mo (or $58/mo if you commit yearly). No need for extra packs.

**Scenario 2 – 20,000 calls**  
- Credits: 80,000 → still inside **Growth**.  
- If you prefer a tighter budget, you could stay on **Starter** and buy a **Blaze** pack (50,000 credits for $50). That gives you 60,000 + 10,000 = 70,000 credits, still short, so you'd need an additional **Spark** pack ($10 for 5,000 credits). Total spend: $19 + $50 + $10 = **$79** (no monthly plan upgrade).  

**Scenario 3 – 100,000 calls**  
- Credits: 400,000 → exceeds **Growth**.  
- **Business** plan (1,000,000 credits) covers it for $349, which works out to **$0.70/1K**—the cheapest per-credit rate across all tiers.  

**Competitor Credit Multipliers**

| Vendor | Free credits | Pay-as-you-go rate (per 1K) | Typical cost for 20,000 SERP + Scrape calls |
|--------|--------------|----------------------------|--------------------------------------------|
| **Tavily** | 1,000 | $8/1K | 80,000 credits |
| **SerpApi** | 250 | $15/1K (minimum) | 40,000 credits |
| **Exa** | 0 | $7/1K search + $1/1K content + $5/1K answer (worst case) | 40,000 credits (note loss of SERP fields like PAA) |

Geekflare's **Growth** tier at $2.76/1K (SERP + Scrape) is cheaper than the cheapest competitor for the same credit volume, while also delivering the full Google SERP JSON—including PAA, AI Overview, and local packs—that the others either omit or charge extra for.

---

## What Each Geekflare Subscription Tier Includes

| Tier | Price (monthly) | Credits / month | Req/s | Team members | Log retention | Proxy access | Priority support |
|------|-----------------|----------------|------|--------------|---------------|--------------|------------------|
| **Free** | $0 | 500 | 1 | 1 | 7 days | **No** proxy | – |
| **Starter** | $19 | 10,000 | 5 | 3 | 30 days | Rotating proxy (+4 credits) | Email |
| **Growth** | $69 / $58 *yearly* | 100,000 | 10 | 5 | 30 days | Rotating proxy (+4 credits) | Priority email |
| **Business** | $349 | 1,000,000 | 25 | 25 | 90 days | Rotating proxy (+4 credits) | Priority email |
| **Enterprise** | Custom | Custom | Up to 1,000 | Custom | Custom | Rotating proxy (+4 credits) | Dedicated support |

*Key policies*  

* **No credit rollover** – credits reset on the first of each billing cycle.  
* **90% usage alert** – you get an email when you cross 90% of your quota.  
* **Hard stop at 100%** – requests are paused until you upgrade or add a credit pack.  
* **Refund guarantee** – cancel within 7 days and get a full refund if you used <10% of your credits.  
* **Failed request exemption** – 5xx errors and captures that return no data cost 0 credits.  

**One-time credit packs** (stackable, valid 1 year, no subscription required):  

| Pack | Price | Credits | Approx. $/1K credits |
|------|-------|---------|----------------------|
| Spark | $10 | 5,000 | $2.00 |
| Blaze | $50 | 50,000 | $1.00 |
| Inferno | $100 | 100,000 | $1.00 |
| Supernova | $200 | 200,000 | $1.00 |

You can buy a pack at any time and the credits are added to your monthly pool, letting you stay on a lower-tier plan while handling occasional spikes.

---

## Mapping Workloads: Which Geekflare Plan Is Right for You?

Below are three common production scenarios. All numbers are **credits**, not dollars, so you can mix plans and packs without doing arithmetic on hidden fees.

| Scenario | Monthly queries | Credits needed (SERP + Scrape) | Recommended tier + packs |
|----------|----------------|--------------------------------|--------------------------|
| **Local SEO dashboard** – 500 keywords across 100 cities, daily refresh | 50,000 | 200,000 | **Growth** (100 K) + **Blaze** pack (50 K) + **Spark** pack (5 K) → 155 K credits, still under 200 K, leaving headroom |
| **Real-time city-level price monitor** – 2,000 keywords, hourly updates | 144,000 | 576,000 | **Business** (1 M) – comfortably covered, no extra packs needed |
| **Ad-hoc research spikes** – occasional batch of 10,000 queries | 10,000 | 40,000 | **Starter** (10 K) + **Blaze** pack (50 K) → 60 K credits, enough for the spike and regular traffic |

The **Growth** plan is the sweet spot for most SaaS products that need city-level SERP data at scale. If you anticipate occasional bursts, a **Blaze** pack adds 50,000 credits for $50—far cheaper than upgrading to Business for a short-term need.

---

## Annual vs Monthly Billing — Plus One-Time Credit Packs

Geekflare offers a **2-month free** incentive when you switch to annual billing. The Growth tier drops from **$69/mo** to **$58/mo** (billed once per year). This is a **flat $10/mo** discount, not a percentage-based calculation, and the price stays locked for the year.

*Why you might choose monthly*:  
- You need flexibility to test a new city-level feature without a year-long commitment.  
- Your credit consumption is highly variable and you want to avoid over-provisioning.

*Why you might choose annual*:  
- Predictable spend and a guaranteed two-month discount.  
- No need to worry about monthly renewal churn.

If you're on a Free or Starter plan and hit a sudden traffic surge, buying a **Spark** ($10 for 5,000 credits) or **Blaze** ($50 for 50,000 credits) pack instantly lifts your quota without changing the subscription. Packs never expire within the year and can be stacked—so a Growth plan plus two Blaze packs gives you 200,000 extra credits for $100.

---

## An Independent Side-by-Side Look at Leading Providers

| Feature / Vendor | **Geekflare** | **Tavily** | **SerpApi** | **Exa** |
|------------------|---------------|------------|-------------|---------|
| Free tier credits | 500 | 1,000 | 250 | 0 |
| Full Google SERP JSON (incl. PAA, AI Overview) | ✅ (2 credits) | ❌ (no full SERP fields) | ✅ (multi-engine) | ❌ (semantic only) |
| City-level geo-targeting (65 k+ cities) | ✅ | ✅ (limited) | ✅ (multi-engine) | ✅ (semantic) |
| Rate limit (req/s) | 1 / 5 / 10 / 25 / up to 1,000 (Enterprise) | 5 req/s | 5 req/s (paid) | 10 req/s |
| Credit-based pricing (Growth tier) | $2.76/1K (SERP + Scrape) | $8/1K | $15/1K | $7/1K (search) + $1/1K (content) + $5/1K (answer) |
| Proxy access | Rotating proxy (+4 credits) on paid plans | No proxy (user-managed) | Built-in rotating proxy | No proxy |
| Refund policy | 7-day, <10% usage | No guarantee | No guarantee | No guarantee |
| Uptime SLA | 99.9% (GCP + Cloudflare) | vary by plan — see vendor pricing | vary by plan — see vendor pricing | vary by plan — see vendor pricing |

**Takeaway** – Geekflare gives you the *full Google SERP* (including PAA and AI Overview) at a predictable credit cost, with city-level targeting and built-in proxy support. The competitors either charge per-query at a much higher rate, lack PAA data, or require you to stitch together multiple services to reach the same feature set.

---

## Five Things That Separate Reliable APIs From Credit Traps

1. **Fixed-cost credit model** – No "base + extra" hidden fees. Geekflare's modes have a single credit total per request.  
2. **Transparent rate limits** – Expressed as requests per second, not "concurrent connections". This lets you calculate throughput accurately.  
3. **Free-tier sanity check** – 500 credits let you prototype without a card; you know exactly when you'll need to upgrade.  
4. **Fail-safe credit exemption** – 5xx errors and empty captures cost zero credits, protecting you from accidental bill spikes.  
5. **Clear rollover policy** – Credits reset monthly, so you never accrue a hidden surplus that later disappears.  

Any provider that hides per-feature pricing, does not offer rollover (credits reset on the 1st), or charges extra for basic SERP fields falls into the "credit trap" category.

---

## When Geekflare Is the Right Call (and When It Isn't)

**Right call** if you:  

* Need **city-level SERP data** with PAA, AI Overview, and local packs in a single JSON payload.  
* Want **predictable pricing** that scales linearly with usage.  
* Require **built-in rotating proxy** for reliable Google scraping.  
* Value a **high reliability record** (99.9% uptime, 4.9 overall rating across G2, Google, and Product Hunt).  

**Not the best fit** if you:  

* Need **deep pagination** beyond page 1 (Geekflare only returns the first page).  
* Want **non-Google engines** (Bing, DuckDuckGo, etc.) or specialized verticals like Flights, Hotels, or Trends.  
* Require **real-time streaming** of raw HTML for every result (you'll need to combine with a separate scraping tool).  

In those edge cases, you might layer Geekflare with another service (e.g., a dedicated map API) or choose a multi-engine provider, but for the vast majority of city-level SERP use cases Geekflare offers the most cost-effective and feature-complete solution.

---

## Frequently Asked Questions

<details><summary>How many credits does a standard search cost?</summary>

A standard web search (organic results only) costs **2 credits** per call. The same 2-credit cost applies to the full SERP JSON mode (`serp: true`).  
</details>

<details><summary>Can I mix different modes in a single request?</summary>

No. Each request must choose one mode (`serp`, `scrape`, or `groundedAnswer`). The credit cost shown in the table is the total for that mode; there is no additive cost.  
</details>

<details><summary>Do failed requests consume credits?</summary>

No. Any 5xx server error or a capture that returns no data is free. This protects you from accidental over-billing during outages.  
</details>

<details><summary>Is there a way to get higher rate limits without moving to Enterprise?</summary>

The highest tier before Enterprise is **Business**, which offers **25 req/s**. If you need more, contact Geekflare for a custom Enterprise agreement that can go up to **1,000 req/s**.  
</details>

<details><summary>How do credit packs interact with my monthly quota?</summary>

Credit packs are added to your monthly pool and are valid for one year. They do not affect your subscription price; they simply increase the number of credits you can spend before hitting the 100% pause.  
</details>

---

Explore plans on [Geekflare Pricing](https://geekflare.com/pricing/), read the [Search API Docs](https://docs.geekflare.com/guides/search.md), or test the [Geekflare Search API](https://geekflare.com/search/) on the card-free Free tier (500 credits/month).
