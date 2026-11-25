# ScraperAPI LangChain Integration: How Do the Three Tools Work, What Does Each Request Cost in Credits, and Which Plan Does Your Agent Actually Need? (Full Plan and Credit Tables)

`langchain-scraperapi` is ScraperAPI's official LangChain package: three tool classes you install with `pip install -U langchain-scraperapi` and hand to an agent as a list. Billing runs on credits rather than calls, and the credit price of a single fetch ranges from 1 to 75 depending on the target site and the flags you set. A $49 plan therefore holds either 100,000 plain page fetches or roughly 1,333 premium rendered ones.

Below are the current plans, the full parameter list for each tool, a working tool-calling agent, and the arithmetic that decides your tier.

> **Short version for most readers:** the free plan gives you 1,000 credits a month with 5 concurrent connections, and new accounts get a 7-day trial that raises the allowance to 5,000 with no card. If your agent only reads ordinary HTML pages, Hobby at **$49/month** covers 100,000 of them. If it hits Google or Amazon regularly, run the credit math first: the same 100,000 credits buy only **4,000 Google searches**.
>
> 👉 [Start on the free plan and burn the trial credits against your real targets](https://www.scraperapi.com/?fp_ref=coupons)

## All ScraperAPI plans, side by side

ScraperAPI lists eight tiers. Prices are monthly; annual billing takes 10% off automatically and needs no code.

| Plan | Monthly | Credits/month | Concurrent threads | Geotargeting | Link |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | 1,000 | 5 | Limited | [Start free](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 | 100,000 | 20 | US & EU only | [See plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | 1,000,000 | 50 | US & EU only | [See plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | 3,000,000 | 100 | Global, 50+ countries | [See plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | 5,000,000 | 200 | Global + pay-as-you-go | [See plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | 10,500,000 | 300 | Global + pay-as-you-go | [See plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | 21,500,000 | 500 | Global + pay-as-you-go | [See plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | 22,000,000+ | 500+ | Global + custom | [See plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

Four things about this table that the plan names don't tell you:

- **Geotargeting is gated at Business.** Hobby and Startup route through US and EU proxies only. If your agent needs a German, Japanese or Brazilian IP, the entry point is **$299/month**, not $49.
- **Pay-as-you-go starts at Scaling.** Below that, an exhausted balance pauses scraping until renewal.
- **Auto-upgrade is opt-in and available from Hobby through Professional.** When credits run out, the account moves to the next tier instead of stopping. You have to switch it on.
- **Credits reset at renewal.** Unused ones don't carry into the next month, so there's no reason to overbuy "just in case".

Hitting your concurrency limit returns a 429, and there's no surcharge for it. The request is refused rather than queued.

## What one credit actually buys

This is the part that decides your plan, and it comes from ScraperAPI's own credits documentation rather than from the marketing pages.

| Request type | Credits per request |
| --- | --- |
| Normal HTML page (blog, docs, news) | 1 |
| E-commerce endpoints: Amazon, Walmart, eBay | 5 |
| SERP endpoints: Google, Bing, all subdomains | 25 |
| Social: LinkedIn | 30 |
| Bot-protection bypass: Cloudflare, Cloudflare Turnstile, DataDome, PerimeterX/Human, Akamai | +10 |
| `premium=true` | +10 |
| `render=true` | +10 |
| `screenshot=true` | +10 |
| `ultra_premium=true` | +30 |
| `premium=true` + `render=true` | 25 total |
| `ultra_premium=true` + `render=true` | 75 total |

Several parameters cost nothing extra: `country_code`, `device_type`, `session_number`, `output_format`, `keep_headers`, `autoparse`, `wait_for_selector`. Markdown output is free. JavaScript rendering is not, and that distinction matters more than any other when you're wiring up an agent.

Converted into what $49 actually delivers on the Hobby plan (100,000 credits):

| Job | Credits each | Requests you get | Cost per 1,000 |
| --- | --- | --- | --- |
| Plain HTML page | 1 | 100,000 | $0.49 |
| Amazon search, structured JSON | 5 | 20,000 | $2.45 |
| Amazon page with `render=true` | 15 | ~6,660 | ~$7.35 |
| Page behind Cloudflare | 11 | ~9,090 | ~$5.39 |
| Google SERP | 25 | 4,000 | $12.25 |
| Google SERP with render | 35 | ~2,857 | ~$17.15 |
| Ultra-premium proxy plus render | 75 | ~1,333 | ~$36.75 |

Move up to Business at $299 for 3,000,000 credits and plain pages land near **$0.10 per 1,000**, which is where ScraperAPI is genuinely competitive. The gap between $0.10 and $36.75 per thousand is the entire argument for knowing your target domains before you pick a tier.

Two tools for checking before you commit: every response carries an `sa-credit-cost` header showing exactly what that request cost, and the `max_cost` parameter caps a single scrape so a surprise 75-credit call can't happen silently. The dashboard also has a Domain Cost Estimator and an `account/urlcost` endpoint that returns the price of a URL without fetching it.

👉 [Check current plan prices against your own credit mix](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## The three tools and what each one takes

| Tool class | Use it for |
| --- | --- |
| `ScraperAPITool` | HTML, text or markdown from any URL |
| `ScraperAPIGoogleSearchTool` | Structured Google SERP data as JSON or CSV |
| `ScraperAPIAmazonSearchTool` | Structured Amazon product-search data as JSON or CSV |

**`ScraperAPITool`**

- `url` (required)
- `output_format`: `"text"` or `"markdown"`; omit it and you get raw HTML
- `country_code`: `"us"`, `"de"`, and so on
- `device_type`: `"desktop"` or `"mobile"`
- `premium`: route through premium proxies (+10 credits)
- `render`: execute JavaScript before returning content (+10 credits)
- `keep_headers`: include response headers

The integration tutorial also uses a `wait` parameter in milliseconds, for example `wait: 5000` alongside `render: True`, to give a slow page time to finish loading. It isn't in the README's parameter table, but it's the one you'll want when a rendered page comes back half-empty.

**`ScraperAPIGoogleSearchTool`**

- `query` (required)
- `output_format`: `"json"` (default) or `"csv"`
- `country_code`, `tld`, `num`, `hl`, `gl`

**`ScraperAPIAmazonSearchTool`**

- `query` (required)
- `output_format`: `"json"` (default) or `"csv"`
- `country_code`, `tld`, `page`

A real Amazon call looks like `{"query": "noise cancelling headphones", "tld": "co.uk", "page": 2}` for the second page of UK results.

One import note: the README uses `from langchain_scraperapi.tools import ScraperAPITool`, while some of ScraperAPI's own tutorial pages use `from langchain_scraperapi import ScraperAPITool`. The `.tools` path is the one in the README and the docs examples, so use that and you won't be debugging an ImportError at midnight.

## A working agent, current API

The `initialize_agent` + `AgentType.OPENAI_FUNCTIONS` pattern you'll find in older tutorials still exists in LangChain, but the maintained path is `create_tool_calling_agent` with an `AgentExecutor`. Use this:

python
import os
os.environ["SCRAPERAPI_API_KEY"] = "your-api-key"

from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_scraperapi.tools import ScraperAPITool

tools = [ScraperAPITool()]
llm = ChatOpenAI(model_name="gpt-4o", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant that can browse websites. "
               "Use ScraperAPITool to access web content."),
    ("human", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad"),
])

agent = create_tool_calling_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

print(agent_executor.invoke({"input": "Browse hackernews and summarize the top story"}))


`MessagesPlaceholder(variable_name="agent_scratchpad")` is mandatory. It's where the tool call and its result get injected between turns. Leave it out and the agent fails on the first call with a prompt-validation error, not a helpful message about the placeholder.

Swap `ChatOpenAI` for Anthropic, Cohere or a local model and the rest of the code holds; the tools are plain LangChain tools. Add the other two to the list and the same agent gains SERP and Amazon access:

python
from langchain_scraperapi.tools import (
    ScraperAPITool, ScraperAPIGoogleSearchTool, ScraperAPIAmazonSearchTool
)
tools = [ScraperAPITool(), ScraperAPIGoogleSearchTool(), ScraperAPIAmazonSearchTool()]


Prompts that exercise all three:

- "Search Google for 'Python web scraping' and give me the top 3 results."
- "Find wireless headphones on Amazon under $100."
- "Browse the GitHub homepage and summarize what's trending."

## Where agents quietly burn credits

A single `agent_executor.invoke()` is not a single request. A research prompt that searches, opens three candidate pages and re-fetches one of them because the markdown came back truncated can easily fire four or five tool calls. Budget for 3–4 calls per task and your credit estimate stops being fiction.

Add the multipliers on top and the per-task number gets uncomfortable. One Amazon research task with rendering at 15 credits a call, times four calls, is 60 credits. A 5,000-credit trial covers about 83 of those tasks, not 1,000.

Four habits that cut consumption without hurting output quality:

1. **Ask for markdown or text, not HTML.** `output_format` is free, and clean markdown means the model doesn't need a second fetch to re-extract what it missed in a wall of tags.
2. **Leave `render` off unless the page needs it.** Ten credits per call, every call, including the ones where the content was already in the initial HTML.
3. **Let the structured endpoints do the parsing.** An Amazon search call at a fixed 5 credits returns parsed JSON with more than a dozen fields. Scraping the product page yourself means 1 credit plus your own parser, plus whatever it costs when Amazon changes the layout.
4. **Cache SERP results.** At 25 credits per Google query, an agent that re-searches the same phrase across a conversation is the fastest way to drain a balance.

The same math applies to your own throughput. Twenty concurrent threads on Hobby sounds generous until each agent turn fires four calls.

## ScraperAPI vs LangChain's built-in WebBaseLoader

`WebBaseLoader` fetches a URL directly. On a static docs page or a plain blog post, that's one library call, zero credits and zero latency added by a third party. For a prototype, it's usually enough.

It breaks in two specific places: pages that build their content with JavaScript, and pages that return 403 to anything resembling a bot. It also hands back raw HTML, which your model then has to chew through. ScraperAPI's own comparison puts it plainly: lightweight loaders "fail on JavaScript-heavy and anti-bot-protected pages and hand back raw HTML." Routing through the proxy pool with rendering and CAPTCHA handling reaches pages a direct fetch can't, and returns markdown or structured JSON directly.

The decision is straightforward. Static pages, no JavaScript, no bot protection, low volume: WebBaseLoader and you're done. Anything behind Cloudflare, anything that renders client-side, anything you'd otherwise write retry and proxy logic for: the credit cost is cheaper than the maintenance.

## Mistakes that cost you an afternoon

**Timeouts.** Hard targets can take close to 70 seconds to return. If your LangChain tool timeout is the default, you'll get failures on exactly the pages you're paying the premium for. Set it to 90 seconds.

**The key isn't where you think it is.** `export SCRAPERAPI_API_KEY=...` in your shell doesn't reach a Docker container, a systemd unit or a CI job. Authentication errors on a key you know is valid almost always trace back to environment scoping.

**Tools in both lists.** The tool list goes into `create_tool_calling_agent(llm, tools, prompt)` *and* into `AgentExecutor(agent=agent, tools=tools)`. Miss the second one and the executor has nothing to call. The symptom is an agent that answers from memory and never fetches anything.

**Credits falling faster than the estimate.** Nine times out of ten it's a multiplier stack you didn't account for: an Amazon URL (5) with `render=true` (10) on a site that also triggers a bypass (10). Check the `sa-credit-cost` header on a few real requests before you blame the plan.

**Concurrency, not credits, is the ceiling.** Twenty threads on Hobby is the limit for simultaneous requests, and an agent that fans out across five URLs per turn will hit it before it hits 100,000 credits.

👉 [See which tiers include pay-as-you-go before you commit to a cap](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Who shouldn't buy this

If your agent reads static documentation, internal wikis or plain article pages, `WebBaseLoader` plus `requests` covers it for free and the integration adds a dependency you don't need.

If SERP data is your main use case, ScraperAPI is an expensive way to buy it. Google queries cost 25 credits each, which is $12.25 per 1,000 on Hobby and still over $2 per 1,000 at Business. Dedicated SERP providers sell the same data for under $1 per 1,000 queries. Use the Google tool when you're already on the platform and want the convenience, not as a primary SERP feed.

If your targets are mostly Cloudflare, DataDome or Akamai protected, the +10 bypass charge stacks on top of everything else, and providers that specialize in hard anti-bot targets usually work out cheaper per successful page. ScraperAPI's advantage is breadth: one key, one SDK, every category of target, with a predictable failure mode.

And if your volume is a few hundred pages a month, the free tier and Hobby will cover it. There's no reason to look past $49.

## Picking your tier

- **Evaluating the integration, or under 1,000 pages a month** → Free. No card, 5 concurrent connections, and the 7-day trial lifts you to 5,000 credits for a realistic test against your actual targets.
- **Agent reads ordinary web pages, under 100,000 fetches a month** → Hobby at $49. This is the plan most people reading this should buy.
- **Need non-US/EU geotargeting** → Business at $299. There's no cheaper path to a 50+ country proxy pool.
- **Amazon or eBay endpoints in regular use** → divide your monthly target count by 5 before comparing tiers. A 100,000-credit plan is a 20,000-Amazon-search plan.
- **Google SERP at any real volume** → price it against a dedicated SERP API first, then decide whether consolidation is worth the premium.
- **Unpredictable monthly volume** → Scaling at $475, because that's where pay-as-you-go and the auto-upgrade escape hatch start.
- **Running more than 100 concurrent agent threads** → Professional or Advanced; concurrency caps at 100 below Business's next tier up.

Two things to confirm on the pricing page before you enter a card: whether annual billing still carries the automatic 10% discount, and which promotional bonus credits are attached to Professional and Advanced, since those have been described as limited-time. Neither changes the per-credit math much, but both change the first invoice.

👉 [Open ScraperAPI's pricing page and start with the free tier](https://www.scraperapi.com/pricing/?fp_ref=coupons)
