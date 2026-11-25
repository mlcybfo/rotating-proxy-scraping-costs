# Rotating proxy for web scraping: datacenter or residential, build or buy, and what a plan actually costs per 1,000 pages?

If your crawler pulls more than a few hundred pages from one domain, a rotating proxy for web scraping is what separates a usable dataset from a wall of 403s. A single IP requesting 50 pages in five seconds gets throttled or blocked; a handful of IPs taking turns usually doesn't. The decisions that follow are narrower than most guides make them: which IP type to pay for, how much rotation logic you want to own, and how a provider's billing maps onto the sites you actually hit.

> **Quick pick:** start on ScraperAPI's free plan. It gives you 1,000 API credits a month with 5 concurrent connections, plus 5,000 credits during your first 7 days, which is enough to measure what your target domains cost per request. If your targets are ordinary pages at 1 credit each, Hobby at **$49/month** covers 100,000 of them. 👉 [See the current plans and prices](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Every ScraperAPI plan side by side

Prices below are the monthly rates on ScraperAPI's pricing page. Annual billing is **10% lower**, and Enterprise is quoted per account rather than listed.

| Plan | API credits / month | Monthly price | What it fits | Start |
| --- | --- | --- | --- | --- |
| Free | 1,000 (5 concurrent) | $0 | Testing cost per domain, not volume | [Free plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Hobby | 100,000 (20 threads) | $49 | Side projects, a few thousand pages a day | [Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | 1,000,000 | $149 | One production crawler on standard pages | [Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | 3,000,000 | $299 | Mixed targets including Amazon and SERPs | [Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | 5,000,000 | $475 | Team-scale pipelines, multiple domains | [Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | 10,500,000 | $975 | Heavy volume with bot-protected sites | [Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | 21,500,000 | $1,975 | Continuous crawling, large backfills | [Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Quote | Committed volume, negotiated terms | [Talk to sales](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

Two billing rules matter more than the sticker price. Credits reset every cycle and **do not roll over**, so buying a tier above your real month wastes money. Annual plans are the exception: you get the full year's credits upfront and can spend them on any timeline.

## How many pages your credits actually cover

Not every request costs the same. A standard page is 1 credit. Amazon is 5. Google and Bing, including subdomains, are 25. LinkedIn is 30. Any page sitting behind Cloudflare, DataDome or PerimeterX adds **10 credits** on top when ScraperAPI has to bypass it.

That turns the plan table into something different from what it looks like at first glance:

| Target | Credits per request | Requests on Hobby (100k) | Cost per 1,000 requests on Hobby |
| --- | --- | --- | --- |
| Standard page | 1 | 100,000 | $0.49 |
| Amazon | 5 | 20,000 | $2.45 |
| Cloudflare-protected page | 11 | ~9,090 | ~$5.39 |
| Google / Bing | 25 | 4,000 | $12.25 |
| LinkedIn | 30 | 3,333 | $14.70 |

The same 100,000 credits are worth 100,000 ecommerce category pages or 4,000 SERPs. On Business the per-credit price drops to roughly $0.0001, which puts a standard page near $0.10 per thousand and a Google request near $2.49 per thousand. Before you pick a tier, run your top URLs through the Domain Cost Estimator in the dashboard and set a `max_cost` per scrape so no single request quietly eats your budget.

## Why one IP cannot carry a crawl

Rate limiting is usually counted per IP, not per account. Send 50 requests in five seconds from one address and the server has a clear signal to throttle you or serve a CAPTCHA. Spread those same requests across three IPs and the pattern disappears below the noise floor. That is the whole mechanism.

The threshold where this becomes non-negotiable is lower than people expect. Anything past a few hundred pages per run against one host, or any recurring schedule, needs address rotation or it will degrade within days. This is also why a rotating proxy pool helps beyond blocking: geo-distributed IPs let you request region-specific pricing and content from the same script.

## Datacenter first, residential last

Datacenter IPs are fast, cheap and stable, but their ownership is traceable to the hosting provider, so an anti-bot team can block entire ranges at once. Residential IPs come from consumer ISPs and carry real geography, which makes them very hard to flag as non-human traffic. They also cost far more, because sourcing them is harder.

The sensible test order runs opposite to most sales pages: **try datacenter IPs against your target first, and only move to residential when datacenter gets blocked at scale or when you need a specific location.** Many sites never check IP ownership at all, and you would be paying a premium for nothing. Apify's engineers argue the same thing publicly: keep residential as the fallback, not the default.

There are subtypes worth knowing. Mobile IPs and static residential IPs are both sold under the residential umbrella; mobile is the hardest to block and the most expensive, static residential suits logins and long-lived sessions where the IP must not change.

Free proxy lists are the one option to rule out. They have a long record of abusing or reselling user traffic, including logins and payment details, and they die fast enough that you rebuild your list weekly.

## Three traps in a do-it-yourself rotator

Writing `random.choice(proxies)` takes a minute. Making it survive a week takes real work, and these are the three places it breaks.

**Same subnet in a row.** A random pick will happily hand you two IPs from the same /24, and many rate limiters read the third octet of the IPv4 address. Track the last subnet used and skip any candidate that matches it. Rotating by ASN or country, not just address, covers the cases where a limiter looks at the IP owner instead.

**Dead proxies left in the pool.** Proxies get blocked and later recover. Keep a dict of failures with a timestamp and bench an IP for a cooldown period, 30 seconds is a reasonable start, before letting it back in. Without this, your success rate decays as the pool fills with corpses.

**Rotating on every single request.** Constant IP hopping is itself a signal. Sophisticated defenses expect a visitor to keep an address for a page and its assets, then move on. Session control, where one IP is pinned for a controlled number of requests, mimics that behavior. Crawlee's `SessionPool` does this out of the box, with something like 100 concurrent human-like sessions.

None of this is hard individually. It is the part that keeps breaking at 2 a.m., and it is what you are buying when you rent a scraping API instead of running a proxy list. 👉 [Skip the rotator build and start on ScraperAPI](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## What ScraperAPI actually does with your request

You send a URL; ScraperAPI routes it through its pools and returns the page. Rotation is controlled by query parameters, which is where most people stop reading and then get surprised later:

- `render=true` runs JavaScript in a headless browser before returning the HTML.
- `country_code=us` pins the exit IP to a country (the parameter accepts standard two-letter codes).
- `premium=true` forces the request through the residential pool.
- `session_number=123` makes the request sticky. Pass the same integer and you keep the same IP.

**Sessions expire 15 minutes after the last usage.** Any long job that assumes its sticky session persists will silently start landing on fresh IPs an hour in. If a workflow needs continuity, reissue the same session number before the window closes, or open a new integer deliberately and treat it as a new visitor.

The other behavior worth understanding is the standard pool's cost logic. ScraperAPI's standard rotating pools are datacenter-first with residential proxies as automatic backups, so a request only escalates to residential when the datacenter attempt fails. You get residential coverage on hard targets without paying residential rates across the board. Setting `premium=true` deliberately opts out of that optimization, which is the right call only when you already know the datacenter path fails. 👉 [Test your session-heavy targets on the free plan](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## How the alternatives compare

Three names come up constantly if you keep shopping. All are capable; the differences are billing model and control surface.

**Oxylabs** runs rotating residential proxies with country, city and ASN-level targeting, and its Web Scraper API applies rotation internally. The targeting granularity is finer than a country code, which matters for local pricing and ad verification work.

**ScrapFly** states its scraping API draws on 190M+ proxies across 92+ countries, and lets you set the origin country and choose a residential pool per request. New accounts get 1,000 free credits, comparable to ScraperAPI's entry test allowance.

**Bright Data** handles stickiness through a session ID embedded in the proxy username, so one credential string covers both rotation and session pinning. It is the most configuration-heavy of the three.

If your work is a few hundred thousand standard pages a month, all four land in a similar range and the deciding factor is usually which dashboard you can stand. Where ScraperAPI separates itself is the credit model: you can compute the cost of a specific domain before committing, which most per-GB residential pricing does not let you do.

## FAQ

**Are 1,000 free credits enough to test anything?**
For standard pages, yes: 1,000 requests is a real sample. If your targets are Google or Bing, it is 40 requests; behind Cloudflare, about 90. The 5,000 credits in your first 7 days give you room to measure several domains before you commit.

**Why are residential proxies so much more expensive than datacenter ones?**
The IPs come from real consumer devices through ISPs, which is expensive to source and maintain. Datacenter IPs are rented from hosting providers at a fraction of the cost. You are paying for the fact that a residential address is nearly indistinguishable from a normal visitor.

**Do I need a new session for every request?**
No. Sessions exist to keep an IP stable across a sequence of related requests, like a paginated crawl or a checkout flow. Rotating per request is fine for independent pages; pinning a session is better when the site tracks state.

**Is proxy rotation legal?**
In the vast majority of countries, requesting public HTML through a proxy is legal. The exception worth checking is countries with network censorship, where using a proxy to reach blocked sites can itself be illegal. Rules differ by jurisdiction, so this is general information rather than advice for your situation.

## Which plan to pick, and what to confirm before you pay

Match the tier to the target, not to the page count:

- **Standard pages, under ~3,000 a day:** Hobby at $49. You will not outgrow it quickly.
- **Amazon, or a mix that includes bot-protected sites:** Business at $299. At 5 to 11 credits per request, Hobby's 100,000 credits disappear in days.
- **Google, Bing or LinkedIn at any volume:** budget 25 to 30 credits per request and size up accordingly. This is where a "cheap" plan stops being cheap.
- **Multiple domains with unpredictable volume:** Scaling or Professional, with pay-as-you-go enabled and a spending cap set so a runaway crawl cannot bill past your limit.
- **Committed annual volume:** the annual discount is real, and credits arrive upfront for the full year, which removes the monthly rollover problem.

Two things to confirm before checkout. First, run your actual URLs through the Domain Cost Estimator so the credit math is based on your targets rather than the standard-page rate. Second, know what happens at the limit: exceeding your concurrency returns a 429 rather than a surprise charge, credit exhaustion pauses the account unless auto-upgrade or pay-as-you-go is on, and there is a 7-day refund window if the service does not fit.

👉 [Pick a plan and run your first 1,000 requests](https://www.scraperapi.com/pricing/?fp_ref=coupons)
