# Dynamic Website Scraping API: How Does It Actually Work? Which Plan Should You Choose? Is It Worth It? — ScraperAPI Complete Guide (Includes JavaScript Rendering, Credit Costs, All Plans Compared, and Free Trial Info)

So here's a situation a lot of developers run into.

You fire up your scraper, point it at a site, and get back… nothing useful. Maybe an empty `<div>`. Maybe a loading spinner frozen mid-spin. The page looks perfectly fine in your browser, but your script is staring at a skeleton. This isn't a bug in your code. This is what happens when you try to scrape a **dynamic website** with a plain HTTP request.

Welcome to the JavaScript problem — the one that quietly breaks a huge chunk of web scraping projects.

**What Is Dynamic Website Scraping, and Why Is It Hard?**

There's a clean dividing line in how websites work. Old-school sites sent you a fully-formed HTML page — every word, price, and product listing was right there in the response. Your scraper just had to read it.

Modern sites don't work that way. A React or Next.js app sends you a nearly empty HTML shell, and then JavaScript runs in the browser to fetch data and build the page. The stuff you actually want — the prices, reviews, search results — arrives *after* the initial response, sometimes hundreds of milliseconds later, sometimes after a button click or a scroll.

A static scraper (one that just sends a GET request and parses the response) never sees any of that. It gets the shell and exits. That's why scraping dynamic content requires something closer to a real browser.

**Three ways developers usually handle this:**

1. **Run a headless browser yourself** — Puppeteer, Playwright, Selenium. They spin up a full Chromium or Firefox instance, execute JavaScript, and let you interact with the rendered page. Powerful, but resource-heavy. Every machine that runs your scraper needs a full browser installed, and you're still responsible for proxies, CAPTCHA solving, IP rotation, and keeping the whole thing from getting blocked.

2. **Find the underlying API** — Some dynamic sites load their data from a separate backend endpoint (you can spot these in the Network tab of DevTools). If you can find and replicate those requests, you skip the browser entirely. But this only works on cooperative sites, and it breaks whenever the site changes its API.

3. **Use a dynamic website scraping API** — Hand the problem off entirely. You send a URL, the API spins up a headless browser in the cloud, handles proxy rotation, solves CAPTCHAs, waits for JavaScript to render, and returns the finished HTML (or parsed JSON). Your code stays simple. The infrastructure problem is someone else's.

Option three is where **ScraperAPI** comes in — and it's worth understanding in detail before you start throwing credits at it.

---

**What ScraperAPI Actually Does**

ScraperAPI is a cloud-based web scraping infrastructure service. You make a single HTTP request to their API with your target URL and some parameters, and they handle the rest: rotating through a pool of 40 million+ IPs across 50+ countries, running headless Chromium when you need JavaScript rendering, solving CAPTCHAs, managing retries, and returning the page content.

👉 [Try ScraperAPI free — 1,000 credits/month, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

The pitch is simple: instead of building and maintaining your own scraping infrastructure, you pay per successful request. You only get billed when the scrape actually works — failed requests (anything outside a 200 or 404 status) don't cost you anything.

Founded in 2018, ScraperAPI now processes around 36 billion API requests per month and serves over 10,000 companies, including Deloitte, Sony, and Alibaba. It's a mature platform with solid documentation and broad language support (Python, Node.js, PHP, Ruby, Java, and others all have code examples in the docs).

---

**The JavaScript Rendering Feature — What It Does and What It Costs**

This is the part most relevant to dynamic website scraping, so let's go through it properly.

By default, ScraperAPI sends a standard HTTP request and returns the raw HTML response — no JavaScript execution. If your target is a simple static page, this is fine and costs 1 credit per request.

To scrape a dynamic, JavaScript-rendered page, you add `render=true` to your request:

python
import requests

payload = {
    'api_key': 'YOUR_API_KEY',
    'url': 'https://www.example.com/',
    'render': 'true'
}

response = requests.get('https://api.scraperapi.com', params=payload)
print(response.text)


With `render=true`, ScraperAPI routes your request through a Chromium headless browser instance, executes the JavaScript on the page, waits for it to finish, and returns the fully rendered HTML. For pages where content loads after a delay, you can add `wait_for_selector` to tell the API to wait for a specific element to appear before returning:

python
payload = {
    'api_key': 'YOUR_API_KEY',
    'url': 'https://www.example.com/',
    'render': 'true',
    'wait_for_selector': '#product-list'
}


For more complex interactions — filling out search forms, clicking buttons, scrolling through infinite-scroll pages — ScraperAPI supports a full **Rendering Instruction Set**. You pass a JSON array of browser actions (click, input, scroll, wait, loop, wait_for_selector, wait_for_event) as a header. Here's an example that types a search term, clicks submit, and waits for results:

python
import requests
import json

url = 'https://api.scraperapi.com/'

headers = {
    'x-sapi-api_key': 'YOUR_API_KEY',
    'x-sapi-render': 'true',
    'x-sapi-instruction_set': json.dumps([
        {"type": "input", "selector": {"type": "css", "value": "#searchInput"}, "value": "product name"},
        {"type": "click", "selector": {"type": "css", "value": "#search-form button[type='submit']"}},
        {"type": "wait_for_selector", "selector": {"type": "css", "value": "#results-container"}}
    ])
}

payload = {'url': 'https://www.target-site.com'}
response = requests.get(url, params=payload, headers=headers)
print(response.text)


The instruction set supports `click`, `input`, `scroll` (to a pixel value, to top, to bottom, or to an element), `wait` (by seconds), `wait_for_event` (domcontentloaded, networkidle, stabilize, etc.), `wait_for_selector`, and `loop` (for repeating action sequences — useful for infinite scroll). Keep instruction sets to 3–4 actions where possible; longer sequences add latency and increase timeout risk.

**One important cost note:** enabling `render=true` adds **+10 credits** per request on top of the base cost. And base cost itself varies by domain — more on that in the next section.

---

**The Credit System — The Part That Actually Determines Your Real Cost**

Here's the thing that catches almost every new ScraperAPI user off guard. The plan headline says "100,000 credits." That number is accurate. But a credit is not always a request.

The actual credit cost of each request depends on two things:

**1. The domain you're scraping** (applied automatically, you don't choose this):

| Target Domain Type | Base Credits per Request |
|---|---|
| Normal websites (blogs, news, etc.) | 1 credit |
| E-commerce (Amazon, eBay, Walmart) | 5 credits |
| Search Engines (Google, Bing) | 25 credits |
| Social/Professional (LinkedIn) | 30 credits |

**2. Parameters you add** (applied only when explicitly set, except anti-bot bypass which is automatic):

| Parameter | Extra Credits |
|---|---|
| `render=true` (JavaScript rendering) | +10 credits |
| `premium=true` (premium proxies) | +10 credits |
| `screenshot=true` | +10 credits |
| `ultra_premium=true` | +30 credits |
| Anti-bot bypass (Cloudflare, DataDome, PerimeterX) | +10 credits (auto-detected) |
| `premium=true` + `render=true` combined | +25 credits (not +20 — it stacks non-linearly) |
| `ultra_premium=true` + `render=true` combined | +75 credits (not +40) |

That last point deserves emphasis. Combining `ultra_premium` with `render` costs 75 credits total per request — not 40 as you might expect. On the Hobby plan (100,000 credits/month), that works out to roughly 1,333 actual requests for heavily-protected dynamic pages. The $49/month plan suddenly looks very different from the headline number.

**What this means for dynamic website scraping specifically:**

If your target is a modern JavaScript-heavy site that also uses Cloudflare protection, you're looking at a minimum of 1 (base) + 10 (render) + 10 (auto-detected Cloudflare bypass) = **21 credits per request** on a regular site, and significantly more on e-commerce or SERP targets.

The good news: **you're only billed for successful requests**. If ScraperAPI fails to deliver a 200 or 404 response, the credits aren't consumed. This is a meaningful protection against runaway costs from infrastructure failures.

---

**All Plans, All Prices: The Complete ScraperAPI Tier Breakdown**

ScraperAPI's lineup currently spans eight tiers. May 2026 saw the addition of two new Growth plans (Professional and Advanced) for teams that need higher volume without going full Enterprise. Annual billing gets you a 10% discount across all paid plans.

| Plan | Monthly Price | Annual Price (per mo) | Credits/Month | Concurrent Threads | Geotargeting | Pay-As-You-Go | Get Started |
|---|---|---|---|---|---|---|---|
| **Free** | $0 | — | 1,000 | 5 | None | No |  [Start free (no card needed)](https://www.scraperapi.com/?fp_ref=coupons) |
| **Hobby** | $49/mo | $44.10/mo | 100,000 | 20 | US & EU only | No |  [Get Hobby Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Startup** | $149/mo | $134.10/mo | 1,000,000 | 50 | US & EU only | No |  [Get Startup Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Business** | $299/mo | $269.10/mo | 3,000,000 | 100 | Global (50+ countries) | No |  [Get Business Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Scaling** | $475/mo | $427.50/mo | 5,000,000 | 200 | Global | Yes |  [Get Scaling Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Professional** *(New Growth)* | $975/mo | $877.50/mo | 10,500,000 (+250K bonus) | 300 | Global | Yes |  [Get Professional Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Advanced** *(New Growth)* | $1,975/mo | $1,777.50/mo | 21,500,000 (+500K bonus) | 500 | Global | Yes |  [Get Advanced Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Enterprise** | Custom | Custom | 22M+ | 500+ | Global | Yes |  [Contact Sales for Enterprise](https://www.scraperapi.com/?fp_ref=coupons) |

**A few non-obvious things about this table:**

- **Geotargeting is gated.** Hobby and Startup are US & EU proxies only. If you need requests to appear to come from Australia, Japan, or anywhere else, you need Business or higher.
- **Pay-As-You-Go only kicks in at Scaling ($475/mo).** Below that, running out of credits mid-month means you're done until the next billing cycle — no overflow option.
- **Credits don't roll over.** Unused credits reset at each renewal. If you over-buy, you lose the difference.
- **DataPipeline (no-code scheduled scraping) uses a different, higher credit schedule** — basic requests cost 6 credits instead of 1 through DataPipeline vs. the standard API. Worth knowing before you set up automated pipelines.
- **Analytics history** is capped at 30 days on Hobby/Startup and extends to 6 months on Business and above.

---

**Which Plan Should You Pick for Dynamic Website Scraping?**

The right answer depends entirely on what you're scraping and how often.

**Start with Free or the 7-day trial** (5,000 credits, no card required) regardless of which plan you're planning to buy. Run your real target URLs through the trial and watch the credit consumption in the dashboard. This is the only way to get an accurate estimate of your monthly cost.

👉 [Start your free 5,000-credit trial here — no credit card needed](https://www.scraperapi.com/?fp_ref=coupons)

**Hobby ($49/mo)** works well for personal projects, prototypes, or monitoring a small number of pages on standard sites. If you're scraping basic HTML pages without render, 100,000 credits is genuinely a lot of requests. If you're scraping Amazon with JS rendering, it's about 4,500–5,000 successful scrapes. That might be enough for a side project; it's probably not enough for a production workflow.

**Startup ($149/mo)** is the sweet spot for small SaaS products, freelancers running scraping pipelines for clients, or teams building internal data tools. One million credits covers a solid amount of dynamic content scraping at reasonable concurrency (50 threads). Still US & EU geotargeting only.

**Business ($299/mo)** is where it gets serious: global geotargeting unlocks, concurrency doubles to 100 threads, and analytics history extends. This is the right tier for production workflows where reliability and geographic coverage matter.

**Scaling and above** — if you're at this level, the key unlock is Pay-As-You-Go overflow. You won't hit a hard stop mid-month, and the per-credit rate improves as volume increases.

---

**Where ScraperAPI Performs Well — and Where It Doesn't**

For a dynamic website scraping API, success rate on real targets matters more than any feature list. Independent benchmarks (Scrapeway, April 2026) show ScraperAPI is genuinely strong in some categories and surprisingly weak in others.

**Strong performers:**

- **Zillow**: 100% success rate, 10.5s average response
- **Amazon**: 98% success rate — the structured data endpoints are particularly reliable, returning 18+ parsed fields including pricing, reviews, BSR, variants, and seller info
- **Etsy**: 99% success rate
- **Walmart**: 93%
- **Google SERPs**: Works well, though at 25 credits/request the costs add up fast

**Weak or zero performance:**

- **Instagram, Twitter/X, Booking.com**: 0% success rate in independent tests
- **Realtor.com**: 12% — surprisingly low for a real estate site given the Zillow performance
- **Login-required sites**: ScraperAPI's terms explicitly prohibit scraping content behind authentication walls. Session persistence via `session_number` is supported for maintaining the same IP across requests, but actual login flows (form filling, 2FA, etc.) are not

The overall average success rate sits around 62–63%, slightly above the industry average of ~58–59%. Response times average 5–7 seconds, faster than the industry average of 9.8 seconds.

---

**Structured Data Endpoints — Useful Shortcut for Supported Sites**

Beyond raw HTML scraping, ScraperAPI offers 18 structured data endpoints that return clean, parsed JSON for specific platforms:

- **Amazon** (3 endpoints): Product by ASIN, search results, competitor offers — 18+ fields including price, ratings, images, descriptions, BSR, reviews
- **Google** (5 endpoints): SERP, Shopping, Maps, News, Jobs
- **Walmart** (4 endpoints): Product, Search, Category, Reviews
- **eBay** (2 endpoints): Product, Search
- **Redfin** (4 endpoints): Search, Agent Details, Rental Properties, For Sale listings

These endpoints are available on all plans, including the Free tier. For teams without the development time to build and maintain custom parsers, the structured data endpoints can save significant effort — though they do cost more credits per request than raw scraping.

---

**What Real Users Say**

Trustpilot sits at **4.5/5 (43 reviews)**. Capterra shows **4.6/5 (62 reviews)** with a notable sub-rating breakdown: Ease of Use at **4.9/5** and Value for Money at **4.5/5**. G2 lands at **4.4/5 (16 reviews)**.

The consistent praise across all platforms: clean documentation, simple integration (you can drop ScraperAPI into existing code as a proxy replacement with minimal changes), and responsive support.

The consistent complaints: the credit multiplier math is confusing until you really sit down and work through it, and performance varies substantially by target — teams who selected plans based on the headline credit number without accounting for multipliers found themselves burning through credits much faster than expected.

> *"Easy to use out of the box. You can start scraping in minutes."* — Capterra (4.9/5 Ease of Use)

> *"Breakdown of credit costs can be confusing."* — John S., Founder, Capterra (Feb 2025)

---

**Async Scraping — For When You're Moving Volume**

One more feature worth flagging for larger dynamic website scraping workflows: the **Async Scraper Service**. Instead of sending a request and waiting synchronously for a response (which blocks your code), you submit jobs to `async.scraperapi.com/jobs`, get back a job ID, and poll for the result when it's ready.

This is particularly useful for scraping dynamic pages at scale, since `render=true` requests can take several seconds each. With async mode, you can submit thousands of jobs in parallel without keeping thousands of connections open:

python
import requests

r = requests.post(
    url='https://async.scraperapi.com/jobs',
    json={
        'apiKey': 'YOUR_API_KEY',
        'render': 'true',
        'url': 'https://www.example.com/product/12345'
    }
)
print(r.text)  # Returns job ID


Async mode supports all the same parameters as the synchronous API, including `render=true`, rendering instruction sets, `wait_for_selector`, geotargeting, and premium proxies.

---

**Quick Tips Before You Commit**

A few practical things that save headaches:

- **Use the dashboard's Domain Cost Estimator** before scraping at scale. It shows you the exact credit cost for a specific URL so you can sanity-check your budget before running 50,000 requests.
- **Domain-based costs are automatic.** Amazon always costs 5 credits. Google always costs 25. You don't opt in — it's applied as soon as ScraperAPI detects the domain. Anti-bot bypass costs are also added automatically when detected.
- **Parameters that cost zero extra:** `wait_for_selector`, `country_code`, `session_number`, `device_type`, `output_format`, `keep_headers=true`, `autoparse=true`. These are free to use.
- **No proactive usage alerts.** ScraperAPI's dashboard shows your consumption, but it won't email you when credits are running low. Check it manually, especially in the first month.
- **7-day no-questions-asked refund policy.** If it doesn't work for your use case, you can get your money back.

---

**The Bottom Line**

Dynamic website scraping is genuinely hard. JavaScript-rendered pages, anti-bot systems, IP blocks, CAPTCHA challenges, infinite scroll, form interactions — the infrastructure problem alone can eat weeks of engineering time if you try to solve it yourself.

ScraperAPI is a solid, mature option for developers who want to outsource that infrastructure. The JavaScript rendering is real and works well on major platforms. The structured data endpoints for Amazon and Google are particularly strong. The 40M+ IP pool is substantial. And you only pay for successful requests.

The main thing to understand going in: the headline credit number is not your request limit. It's your credit budget. Run the math for your specific targets and feature flags before choosing a plan — the actual cost per useful scrape can be anywhere from 1 credit to 75+, depending on what you're hitting.

The free trial makes it easy enough to test before spending anything. Start there.

👉 [Get 5,000 free trial credits — no credit card needed](https://www.scraperapi.com/?fp_ref=coupons)
