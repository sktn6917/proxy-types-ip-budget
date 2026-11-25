# Web Scraping Proxy: Which Type to Buy, How Many IPs You Need, and When a Scraping API Beats Running Your Own Pool

There are two ways to keep a scraper running past a site's rate limits. You rent proxy IPs and build the rotation, retry and ban-detection logic yourself, or you send requests through a scraping API that already runs a rotating pool behind one endpoint and bills per page. Residential traffic runs $1.40 to $8 per GB across the vendors compared below; a managed scraping API starts at $49 a month.

Most people searching for a web scraping proxy are choosing between those two paths, and the decision comes down to three numbers: how many pages you pull per month, how hard the target fights back, and what an hour of your engineering time costs.

> **Quick pick:** if your targets sit behind Cloudflare, DataDome or PerimeterX and you would rather not operate a proxy pool, ScraperAPI's Hobby plan starts at $49/month and folds rotating proxies, JS rendering and bot-protection bypass into a per-request price. The trial runs 7 days with 5,000 credits and no credit card. 👉 [Check the current plans and what one credit covers](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## ScraperAPI's plans, with the numbers that decide the bill

ScraperAPI prices in credits, and one credit buys one standard page request. That unit is the whole story of your monthly cost, because harder pages consume more of it.

| Plan | Monthly credits | Concurrent threads | Price | Get started |
| --- | --- | --- | --- | --- |
| Free trial | 5,000 for 7 days | 5 | $0, no card required | [Start the trial](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Hobby | Entry allowance | Scales with plan | $49/month | [See Hobby pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | 1,000,000 | Scales with plan | $149/month | [See Startup pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | $475/month, or $427.50/month billed annually ($5,130/year, saves $570) | [See Scaling pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | 21,500,000, plus a limited-time 500K bonus | 500 | $1,975/month, or $1,777.50/month billed annually ($21,330/year, saves $2,370) | [See Advanced pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | Quote | [Request Enterprise details](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

Two tiers sit between the ones priced above. Business fills the gap between Startup and Scaling, and Professional sits between Scaling and Advanced; the pricing page lists their credit allowances. Annual billing takes 10% off across the board, which is where the annual figures in the table come from.

Three plan mechanics are worth knowing before you pick a tier, because they change what "out of credits" means:

- Going over your concurrency limit returns a 429 and the request is turned away rather than queued. There is no surcharge for it.
- Auto-upgrade is opt-in on Hobby, Startup, Business, Scaling and Professional. When credits run dry, the account moves up a tier instead of stalling.
- Pay-as-you-go, at a fixed per-credit rate up to a cap you set, is available on Scaling, Professional, Advanced and Enterprise.

## Residential, ISP and datacenter: the job each type actually does

Datacenter IPs are cheap and fast and get flagged. They are assigned commercially through cloud servers rather than ISPs, so they carry a standing "probably automated" reputation, and sharing them across users makes it worse. They work well against high-volume targets with light protection.

Residential IPs carry real ISP associations and a much better trust score, at the cost of unstable infrastructure and per-gigabyte billing. ISP proxies, sometimes sold as static residential, try to keep the reliability of a datacenter with the trust of a residential address. Mobile IPs score highest of all and cost accordingly; most scraping projects do not need them.

The matching rule that Bright Data publishes for its own customers is a reasonable default for anyone: protected, anti-bot-heavy sites take residential; high-volume lenient targets take datacenter; logged-in or long-running sessions take ISP; and hard targets that throw CAPTCHAs need an unblocker or scraping API on top of whichever proxy type you land on.

One budget detail gets skipped often. Very few target sites support IPv6, so in practice you buy IPv4 addresses, and IPv4 runs 3 to 10 times more expensive per address because the supply is fixed. If your target happens to answer over IPv6, your pool budget drops sharply.

## How many IPs, and the two bills you will be comparing

A widely cited rule of thumb from 2018: one IP can handle about 500 requests per hour, so 100,000 URLs per hour needs 200 IPs, or 400 to 600 with a 2x to 3x safety margin. Treat that as a historical magnitude, not a current standard. Rate-limit thresholds vary enormously between sites today, and the only reliable method is measuring your own 429 and block rates after a week of traffic.

Cost comparisons go wrong when people add up numbers from different billing models. Three models are in play here:

**Per gigabyte.** Residential proxies are sold this way, at $1.40 to $8 per GB depending on vendor and committed volume. This is the model where browser automation quietly doubles your bill, because Puppeteer, Playwright and Selenium pull images, fonts and scripts you never parse. Heavy HTML sites cost more bandwidth than sites that fill their pages with background requests.

**Per IP or per month.** Datacenter and ISP proxies usually price this way. Published ranges put datacenter pools at roughly $50 to $100 per month for 10 to 50 IPs, and residential services at $80 to $500 per month depending on traffic.

**Per request.** Scraping APIs price this way, and the per-request price is not fixed. That is the subject of the next section, and it matters more than which tier you buy.

A 2018 rule of thumb and a 2026 price list cannot be reconciled into one number. Pick the billing model that matches how you will actually consume the resource, then compare within it.

## Playwright proxy setup, and three failures that eat an afternoon

Playwright takes a proxy as a launch argument. You pass a `proxy` dictionary to `chromium.launch()` and every request from that browser instance goes through it. The `server` field accepts `http://`, `https://` and `socks5://`, and a bare `host:port` string is treated as HTTP. Credentials go in separate `username` and `password` fields in the same dictionary.

That last point is not a style preference. The team at Scrapfly documented their own test results, and three of them will cost you real debugging time:

1. **A wrong password produces a 30-second timeout, not a 407.** `page.goto()` waits, then raises a timeout error. Playwright never surfaces the proxy authentication failure, so the error message points at the page rather than the credentials.
2. **Credentials embedded in the server URL do not work.** The cURL-style `username:password@host:port` form is ignored, and you get the same 30-second timeout.
3. **Sticky sessions need provider-side configuration.** A fixed proxy gateway does not guarantee a fixed exit IP. For an IP-bound login or a checkout flow, keep the task in one browser context and turn on the provider's sticky-session setting so new connections retain the same exit address. Scrapfly notes that their two-context test only demonstrated cookie isolation, not whether an IP change invalidates a login.

Two more behaviors are worth building around. Each browser context can take its own proxy, and contexts do not see each other's traffic, so one browser can run several sessions with different exit IPs simultaneously. And a proxy only moves the traffic; the browser still reports `HeadlessChrome` and keeps your machine's timezone unless you set them, which is enough for fingerprinting scripts to catch you even with a clean residential IP.

To confirm a proxy is live, load `ipinfo.io` or `httpbin.dev/anything` inside the browser and check the returned origin against your own IP. Note that the check has to run inside the browser: a `requests` call in the same script bypasses Playwright's proxy settings entirely.

If that list reads like a week of work you would rather not spend, that is the argument for a managed endpoint. 👉 [See how the request-level pricing compares with running your own pool](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Why the plan matters less than the multiplier

This is the part that decides whether a scraping API is cheap or expensive, and it is buried in a pricing FAQ rather than on the plan cards.

| Request type | Credits per request |
| --- | --- |
| Standard page | 1 |
| Amazon, Walmart, eBay | 5 |
| Google, Bing and subdomains | 25 |
| LinkedIn | 30 |
| Bot-protection bypass (Cloudflare, DataDome, PerimeterX/HUMAN, Akamai) | +10 |
| JS rendering (`render=true`) | +10 |
| Premium proxies (`premium=true`) | +10 |
| Screenshot | +10 |
| Premium proxies plus JS rendering | 25 |
| Ultra Premium (`ultra_premium=true`) | 30 |
| Ultra Premium plus JS rendering | 75 |

Geotargeting, sessions, device type, output format and auto parsing add nothing. Every response carries an `sa-credit-cost` header showing what that request actually cost, and you can price any URL in advance with the Domain Cost Estimator in the dashboard or cap it with `max_cost`.

Run the arithmetic on the Startup tier and the spread becomes obvious. At $149 for 1,000,000 credits, plain HTML pages come out around $0.15 per 1,000 pages. Pages that need premium proxies plus rendering cost 25 credits each, so the same plan covers 40,000 of them, or about $3.73 per 1,000. Ultra Premium with rendering at 75 credits works out to roughly $11.18 per 1,000 pages.

An upgrade from Startup to the next tier buys you more credits at a marginally better rate. Cutting rendering down to the pages that genuinely need it buys you 25 times more coverage on the same budget. Do the second thing first.

## The proxy vendors, side by side

These are published list prices for residential traffic, per gigabyte, plus network size. The figures come from a Bright Data comparison page, which also ranks Bright Data first; treat the list prices as starting points and the ranking as marketing.

| Provider | Residential IPs | Residential from | Other pricing | Notes |
| --- | --- | --- | --- | --- |
| Bright Data | 400M+ | $2.50/GB | Datacenter from $0.90/IP, ISP from $1.30/IP | Largest network, bundles unblocker and scraping browser |
| Decodo | 115M+ | $2/GB on volume | — | Budget all-rounder with a site unblocker |
| Oxylabs | 175M+ | $2.50/GB on volume | Entry plans at $6/GB | Strong enterprise tooling, higher entry price, strict vetting |
| SOAX | 155M+ | $2/GB on volume | $90 starter plan | Precise geo-targeting, no pay-as-you-go |
| IPRoyal | 34M+ | $1.75/GB | Pay-as-you-go at $7/GB | Purchased traffic does not expire |
| Webshare | 80M+ | $1.40/GB at high volume | — | Instant self-serve signup, email-only support |
| Rayobyte | Under 1M | $3/GB | — | Positions on ethical sourcing |

Network size is not the same as usable inventory. A 400M-address pool and a 34M-address pool can both serve a price-monitoring job in one country, and the smaller one may be cheaper.

## Questions that come up before checkout

**Can I just use free proxies?** No, and the failure modes are specific. Free proxy lists run at 5% to 15% working at any given moment, get shared across thousands of users so they are blocked almost immediately, offer no geographic targeting, and in the worst case log your traffic or inject malware. Zyte's guidance is blunter than most: stay away from public proxies entirely. If cost is the constraint, use a vendor's free tier or trial instead.

**HTTP or SOCKS5?** SOCKS5 is marginally faster, more stable and more secure. HTTP is supported by more providers and more client libraries. In day-to-day scraping the difference rarely shows up, and Playwright handles both, defaulting a bare `host:port` to HTTP.

**Why do residential proxies break long sessions?** Residential pools rotate by design, and a mid-flow IP change breaks a logged-in session or an in-progress cart. The fix is provider-side session stickiness, not a fixed gateway address, because a single gateway can still hand out different exit IPs.

**Do I still need to respect robots.txt?** Yes. A proxy changes where your requests appear to come from, not what you are allowed to do. Throttle your rate, identify the crawler with a clear user agent where appropriate, check `robots.txt` and the sitemap before the first request, and do not push harder than the site can take.

## Which one to pick

If you are pulling a few hundred thousand pages a month from sites without aggressive bot protection, a cheap datacenter pool plus your own rotation logic is the lower bill. If most of your targets sit behind Cloudflare or DataDome, add up what 25 to 75 credits per page does to your monthly total on the cheapest tier that fits, then compare it against a residential proxy invoice plus the engineering hours.

Concretely: under 100,000 requests a month on plain HTML, take the entry paid tier and stop reading. Above 1 million requests a month, or any meaningful share of JS-rendered pages, do the credit math on Startup and Scaling before you commit, and turn on `max_cost` so a single expensive URL cannot burn a day of quota.

Two things to confirm before you pay anyone: whether the plan renews at the same rate you signed up at, and whether a refund or trial window covers the first month while you measure real success rates against your own targets. ScraperAPI's trial gives you 7 days and 5,000 credits with no card attached, which is enough to run your hardest URL list through it.

👉 [Start the free trial and price your own target list](https://www.scraperapi.com/pricing/?fp_ref=coupons)
