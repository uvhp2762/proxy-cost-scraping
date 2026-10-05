# http proxies: how to pick ones that don't die mid-scrape, what they really cost per GB, and when free lists are worth using

Search "http proxies" and you'll get two very different kinds of results: GitHub repos dumping thousands of IP:port lines, and vendor pages trying to sell you bandwidth. Both are answering the same question — where do I get an HTTP proxy? — but they're aimed at completely different problems. The repo is fine if you need one request through one random IP. It falls apart the moment you need 20,000 requests against a site that actually checks who's knocking.

This is a walkthrough of what HTTP proxies are, why the free route fails at scale, what to look at when you're paying, and how the pricing on one provider — DataImpulse — compares across its four proxy types. Prices below come from the provider's own published rates and third-party reviews from September 2026.

## What an HTTP proxy actually is (and the HTTPS part everyone skips)

An HTTP proxy is a server that sits between your client and the site you're hitting. Your request goes to the proxy, the proxy forwards it, the response comes back through the proxy. The target site sees the proxy's IP instead of yours.

The detail that trips people up: HTTP and HTTPS are handled differently. For plain `http://` destinations, the proxy can forward the request directly and even cache responses. For `https://` destinations, the client sends an HTTP `CONNECT` request and the proxy opens a raw TCP tunnel — it can't read or cache what's inside, because it's TLS. That's why free proxy lists advertise "HTTPS support" as a separate feature. It's the same proxy port, different handshake.

Practical consequence: a proxy that's transparent for HTTP traffic may fail entirely on HTTPS if it doesn't allow `CONNECT`, and CONNECT support is exactly what operators of sketchy free proxies often disable, because tunneling gives them nothing to scrape.

## Why the free HTTP proxy list route collapses

The free lists are real and actively maintained. Some are genuinely impressive operations — HProxy claims 780,000+ collected proxies with live re-checking every couple of minutes [3]. Others publish smaller verified sets: one GitHub repo was showing 165 working HTTP proxies at its last hourly refresh, alongside 34 SOCKS4 and 440 SOCKS5 entries [4].

Look closer at the numbers and the picture changes. One collector publishes its own success verification rate: 0.69% [6]. Databay's list had 4,136 HTTP entries that passed an HTTPS request with certificate validation at their last check [5] — but "at last check" is doing a lot of work in that sentence. The maintainers themselves say it plainly: an endpoint can stop working between checks, and you should re-fetch rather than cache.

Three specific failure modes, all of which show up in production:

**They die constantly.** Monosans' README says it outright — a proxy that worked at the top of the hour may be gone by the next update [4]. If your scraper hardcodes a list, you're rebuilding it hourly and still eating failures.

**They're shared with everyone else.** The same IPs appear in dozens of public lists at once. Any site with a halfway decent IP reputation check has already blocked them, which is why the working fraction is so low.

**The operator can see everything.** The monosans README carries an unusually blunt warning: whoever runs the proxy can log and modify anything passing through it. Never send credentials or tokens over a list proxy, and stick to HTTPS so the operator can't read or rewrite responses [4].

That last point is the one people ignore. Free HTTP proxies are fine for fetching a public page you don't care about. They are not fine for anything involving a login, an API key, an e-commerce account, or a competitor you're scraping repeatedly.

## What to check before paying for HTTP proxies

Once you're past the free tier, the label price is almost irrelevant. Five things drive the actual bill:

1. **Does it support CONNECT/HTTPS, and on which ports?** Some providers give you HTTP on one port and SOCKS5 on another, and that's it.
2. **What happens to unused traffic?** This is the biggest hidden cost. Monthly plans that reset mean you pay for GB you never consumed. Pay-as-you-go with non-expiring traffic means you don't.
3. **How is geo-targeting priced?** Country-level is usually included. City, state, ZIP, and ASN are frequently paid add-ons — and some providers bill them at a multiplier, not a flat fee.
4. **What's the concurrency limit?** A provider that caps you at 50 threads will bottleneck a large crawl no matter how cheap the bandwidth is.
5. **What's the cost per *successful* request?** A rate of $1/GB with a 70% success rate costs more in real terms than $1.40/GB at 99%. This is the number that actually belongs in your budget.

That last framing — cost per successful request — is the one most comparison pages skip, because it makes cheap-looking providers look expensive.

## Where DataImpulse sits

DataImpulse runs its own IP pool rather than reselling another provider's, which is the reason its rates sit where they do. The public numbers: residential at **$1/GB**, datacenter at **$0.50/GB**, mobile at **$2/GB**, premium residential at **$5/GB** — all pay-as-you-go, with traffic that never expires and no subscription required [2][9].

The pool is advertised at 90M+ ethically sourced IPs across 195 countries, with a published success rate of 99.51% and a 4.8/5 rating on G2 [9]. Coverage by product type differs more than the marketing suggests: 214 locations for residential, 210 for premium residential, 191 for mobile, and 123 for datacenter [11].

Protocol support, which matters for the exact thing you're searching for:

| Setting | Value |
| --- | --- |
| HTTP/HTTPS rotating | `gw.dataimpulse.com:823` |
| SOCKS5 rotating | `gw.dataimpulse.com:824` |
| Sticky sessions | Ports 10000–20000, 1–120 minutes per IP |
| Default sticky interval | 30 minutes if unset or set to 0 |
| Max concurrent threads | 2,000 (scalable on request) |
| Protocols | HTTP, HTTPS, SOCKS5 |

Source: [7][10]

Rotating means the IP changes on every request — what you want for high-volume crawling. Sticky pins one IP to one port for up to two hours, which is what you want when a site behaves better with session continuity, like a logged-in workflow or a multi-step checkout flow.

👉 [Start with 5 GB of residential HTTP proxies for $5](https://bit.ly/dataimPulse)

## Full plan and pricing comparison

All four product lines, with the tiers DataImpulse publishes as of September 2026. Traffic never expires on any of them, and no subscription is required.

| Proxy type | Package | Traffic | Price | Effective rate | Billing |
| --- | --- | --- | --- | --- | --- |
| Residential | Intro | 5 GB | $5 | $1.00/GB | Pay-as-you-go  [Get started](https://bit.ly/dataimPulse) |
| Residential | Advanced | 1 TB | $800 | $0.80/GB | Pay-as-you-go  [See the Advanced tier](https://bit.ly/dataimPulse) |
| Residential | Custom | 5 TB+ | From $0.70/GB | Negotiated | Custom  [Request pricing](https://bit.ly/dataimPulse) |
| Datacenter | Intro | 10 GB | $5 | $0.50/GB | Pay-as-you-go  [Get started](https://bit.ly/dataimPulse) |
| Datacenter | Volume | 100 GB | $50 | $0.50/GB | Pay-as-you-go  [See datacenter plans](https://bit.ly/dataimPulse) |
| Datacenter | Volume | 1 TB | $450 | $0.45/GB | Pay-as-you-go  [See datacenter plans](https://bit.ly/dataimPulse) |
| Datacenter | Custom | 5 TB+ | From $2,250 | Negotiated | Custom  [Request pricing](https://bit.ly/dataimPulse) |
| Mobile | Intro | 2.5 GB | $5 | $2.00/GB | Pay-as-you-go  [Get started](https://bit.ly/dataimPulse) |
| Mobile | Volume | 25 GB | $50 | $2.00/GB | Pay-as-you-go  [See mobile plans](https://bit.ly/dataimPulse) |
| Mobile | Volume | 1 TB | $1,600 | $1.60/GB | Pay-as-you-go  [See mobile plans](https://bit.ly/dataimPulse) |
| Mobile | Custom | 5 TB+ | From $8,000 | Negotiated | Custom  [Request pricing](https://bit.ly/dataimPulse) |
| Premium residential | Intro | 1 GB | $5 | $5.00/GB | Pay-as-you-go  [Get started](https://bit.ly/dataimPulse) |
| Premium residential | Volume | 10 GB | $50 | $5.00/GB | Pay-as-you-go  [See premium plans](https://bit.ly/dataimPulse) |
| Premium residential | Custom | 5 TB+ | From $20,000 | Negotiated | Custom  [Request pricing](https://bit.ly/dataimPulse) |

Rates and tiers as published by DataImpulse and cross-checked against AIMultiple's September 2026 update [1][2]. Datacenter plans include 99.9% uptime and randomized subnet access [1]. Volume discounts on mobile and premium residential only kick in at the 1 TB tier — below that, you're paying the flat rate [1].

## The targeting surcharge worth knowing before you buy

This is the detail that turns a $1/GB plan into a $2/GB one, and it's easy to miss.

Country selection or exclusion is included in the base price. ASN exclusion is included too. But state, city, ZIP, and ASN selection are billed at **double the standard rate** on residential plans [1]. So a scrape targeting ZIP codes in Ohio doesn't cost $1/GB — it costs $2/GB.

On datacenter proxies, state/city/ZIP/ASN targeting appears to be included at no extra charge [1]. AIMultiple flags this asymmetry explicitly and suggests confirming current billing with support before you build a budget around it. Worth doing, since the difference is 2x on your largest cost line.

If your project genuinely needs city-level precision, factor it in from the start. If it doesn't — and plenty of scraping jobs work fine at country level — you're leaving money on the table by not checking.

## Which plan fits which job

The pricing structure makes the decision mostly mechanical:

**Residential at $1/GB** is the default for anything hitting protected targets: e-commerce product pages, SERPs, social platforms, price monitoring. Real home-broadband IPs mean the target site sees an ordinary user, which is the whole point. For most projects this is where you start.

**Datacenter at $0.50/GB** is half the price and much faster, and it's the right call when the target doesn't aggressively block server IPs. Common use: bulk fetching your own infrastructure, testing, or scraping sites with light protection. Paying residential rates for datacenter-grade work is just burning budget.

**Mobile at $2/GB** is for the hard targets — mobile app data, and sites where carrier-grade NAT means thousands of real users share one IP, making blocks impractical. It's the most expensive per GB for a reason. Use it only when residential is actually failing.

**Premium residential at $5/GB** buys speed, higher uptime, a dedicated account manager, and all targeting options with no surcharge [1]. The math works out if you'd otherwise be paying the 2x targeting multiplier plus your own time on failed requests — 5x the base rate is harder to justify if you're a country-level scrape with no support needs.

One structural note: DataImpulse sells raw proxy connections, not a managed scraping API. TechRadar's review points this out as the trade-off — you write your own request handling, parsing, retries, and CAPTCHA logic [7]. In exchange you get the proxy layer at a low per-GB rate and full control over the stack. There are integration guides for Scrapy, Puppeteer, Selenium, Playwright, AdsPower, Multilogin, Shadowrocket, and Zapier, plus code snippets in Python, Node.js, PHP, C#, Go, Ruby, and cURL [7]. If you want a turnkey scraper with no code, this isn't that product.

👉 [Compare all DataImpulse proxy types and pick your plan](https://bit.ly/dataimPulse)

## Setting it up

The HTTP part is genuinely boring, which is a compliment. Point your client at the gateway and authenticate:

bash
curl -x http://USERNAME:PASSWORD@gw.dataimpulse.com:823 https://example.com


In Python with `requests`, the same thing:

python
proxies = {
    "http": "http://USERNAME:PASSWORD@gw.dataimpulse.com:823",
    "https": "http://USERNAME:PASSWORD@gw.dataimpulse.com:823",
}
r = requests.get("https://example.com", proxies=proxies, timeout=30)


Two things to watch. First, note that both the `http` and `https` keys point at the HTTP proxy port — that's normal, `CONNECT` handles the TLS tunnel. Second, keep certificate verification on. A proxy that requires you to disable TLS checks is a proxy that can read your traffic, and that's true whether you paid for it or not.

For sticky sessions, swap port 823 for any port in 10000–20000. The IP binds to that port for up to 120 minutes, defaulting to 30 if you don't specify [10]. Running multiple threads on different sticky ports is how you parallelize a session-bound workflow.

New accounts get a 7-day refund window, which is the practical way to measure your own cost per successful request against your actual target list instead of trusting anyone's benchmark [1].

## Quick answers

**Are free HTTP proxies illegal to use?** Not inherently. What matters is what you do with them — bypassing rate limits, scraping behind logins, or accessing content you're not authorized to access can breach terms of service or laws depending on jurisdiction. The bigger practical risk is that the proxy operator can log and modify your traffic.

**HTTP proxy or SOCKS5?** For general web scraping, HTTP is fine and marginally easier to configure. SOCKS5 works at a lower level and handles non-HTTP protocols and raw TCP tunnels. DataImpulse runs both on one gateway — 823 for HTTP, 824 for SOCKS5 — so you can switch without changing providers [10].

**Do I need rotating or sticky IPs?** Rotating for high-volume crawling and anything where a fresh IP per request prevents rate limiting. Sticky when a site ties session state to an IP and breaking that mid-flow causes errors.

**What does "traffic never expires" actually change?** It means unused GB stay in your account indefinitely rather than resetting monthly. For workloads with irregular volume — a big crawl one month, almost nothing the next — that's the difference between paying for what you use and paying for a ceiling you don't hit [8].

**Is $1/GB actually cheap?** Against the 2026 market, yes. Fair ranges sit around $1–8/GB for residential, with $3–4/GB mid-market and $5–8/GB enterprise pricing [2]. Premium residential, ISP/static, and mobile all sit higher, which is why picking the right proxy *type* for the job matters more than chasing the lowest headline rate.

The short version: if you need one page through one random IP, the free lists still work. If you need HTTP proxies that hold up across thousands of requests, the money goes to whichever provider gets you the lowest cost per successful request — and that number starts with the per-GB rate, not the plan description.
