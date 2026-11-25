# scraperapi reddit: How Do You Scrape Posts and Comments, Is the Free Trial Enough, and What Do Credits Actually Cost? (Selectors, Markdown Output, Plan Notes)

ScraperAPI solves the two things that break most homemade Reddit scrapers, which are blocked IPs and JavaScript-rendered pages. You send a Reddit URL to `https://api.scraperapi.com/` with an API key, and you parse whatever HTML comes back. A new account gets **5,000 API credits during its first seven days**, then settles onto the free plan's **1,000 credits per month with 5 concurrent connections**, and paid plans include a **7-day refund window**.

What follows covers the fields ScraperAPI's own Reddit walkthrough reads, the `output_format=markdown` mode that skips HTML parsing entirely, how credits are charged per request (it isn't one credit per page everywhere), and the cases where you should skip this service and use something else.

> **Quick version:** spend the 5,000 trial credits on the subreddits you actually need before you compare plan names. Credit cost depends on the target domain and the parameters you send, so check your URLs in the dashboard's Domain Cost Estimator first. 👉 [Start the free trial and run your own subreddit through it](https://www.scraperapi.com/signup?fp_ref=coupons)

## What does ScraperAPI actually remove from a Reddit scraping job?

ScraperAPI's own guide states the problem without dressing it up: "Reddit is known for blocking scrapers from its website, making collecting data at any meaningful scale challenging." The workaround is to route every request through the API, so proxy rotation, browser rendering and retries happen on their infrastructure rather than in your script.

Concretely, three maintenance jobs disappear from your project. You stop rotating and replacing proxies when Reddit starts returning challenge pages. You stop running a headless browser for pages that only assemble content after JavaScript executes. You stop hand-writing retry and backoff logic for the requests that come back empty. Third-party tutorials from 2024 describe the same three capabilities (IP rotation, CAPTCHA handling, JavaScript rendering) as the reason to use the service on Reddit rather than raw `requests`.

What doesn't disappear is the part most tutorials skip: deciding which fields you need, keeping your parser alive as Reddit's markup moves, and validating that the data you stored is complete. A poster on r/AIToolBriefs put it well after looking into the platform. He wrote that the takeaway wasn't that it makes scraping effortless, because it doesn't; it removes infrastructure work so you spend your time on data instead of upkeep. That's the right expectation to set before you sign up.

👉 [See the Reddit scraping setup ScraperAPI documents](https://www.scraperapi.com/solutions/market-research-scraper/reddit-scraper/?fp_ref=coupons)

## Plans and credits: what to check before you pick a tier

ScraperAPI's billing documentation names eight tiers. What it states plainly is how each tier behaves when credits run out, and that behaviour matters more than the headline number: a plan that pauses mid-run costs you a day, a plan with pay-as-you-go and a spending cap doesn't. The credit allowance and price attached to each tier live on the pricing page, which is the only place they're kept current, so verify there before checkout.

| Plan | What the documentation confirms | When credits run out | Check the current price |
| --- | --- | --- | --- |
| Free | 1,000 credits/month, max 5 concurrent connections, 5,000 credits available in the first 7 days | Requests pause until the monthly renewal | [Create a free account](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Hobby | Paid tier; auto-upgrade available | Opt-in auto-upgrade to the next plan, otherwise pause | [See Hobby's allowance and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | Paid tier; auto-upgrade available | Same as Hobby | [See Startup's allowance and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | Paid tier; auto-upgrade available | Same as Hobby | [See Business's allowance and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | Auto-upgrade and pay-as-you-go | Keep scraping at a fixed per-credit rate up to a cap you set | [See Scaling's allowance and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | Auto-upgrade and pay-as-you-go | Same as Scaling | [See Professional's allowance and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | Plan changes go through sales; pay-as-you-go | Pay-as-you-go up to your monthly spending cap | [See Advanced's allowance and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Plan changes go through sales; pay-as-you-go | Same as Advanced | [See Enterprise's allowance and price](https://www.scraperapi.com/pricing?fp_ref=coupons) |

### How a credit is actually charged

This is the part that decides whether your trial lasts five days or five hours, and it's buried in the pricing FAQ rather than on the plan cards. A standard page costs 1 credit. Amazon costs 5. Google and Bing, including subdomains, cost 25. LinkedIn costs 30. Sites sitting behind bot protection such as Cloudflare, Datadome or PerimeterX add 10 credits per request when ScraperAPI has to bypass them. Parameters you add to a request can change the cost too.

Two controls follow from that. The Domain Cost Estimator in your dashboard returns the exact cost for a specific URL, so you can price a subreddit crawl before you commit. And `max_cost` lets you cap what a single scrape is allowed to spend, which stops one expensive target from quietly eating a day's allowance.

### The differences between paid tiers that don't show up on the plan card

Auto-upgrade is opt-in and available on Hobby, Startup, Business, Scaling and Professional. Without it, your account pauses at the limit. Pay-as-you-go, at a fixed per-credit rate, is available on Scaling, Professional, Advanced and Enterprise, and the monthly cap you set is a spending limit rather than an amount charged up front; usage won't exceed it unless you change the setting.

Annual plans hand you the full year's credits at the start, usable on any timeline. If you burn through them early you can renew the same annual plan before the period ends to get a fresh allocation. Cancellation is available at any time from the dashboard at no charge, and the refund policy is seven days, no questions asked.

If you landed here hunting a coupon code: I couldn't verify any code that applies at checkout, and the referral parameter on the links in this article is a tracking tag, not a discount. The concrete things you can count on are the 5,000 trial credits, the free 1,000-credit monthly tier, and the 7-day refund. 👉 [Check the current plan prices and allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## How do you scrape a Reddit post and its comments?

Send the Reddit URL through the API endpoint, then read two structures out of the returned HTML: a `shreddit-post` element for the post itself, and `div.thing[data-type="comment"]` blocks for the comments. That's the whole method behind ScraperAPI's official Python walkthrough.

1. **Pick your URLs.** The guide works with a topic page such as `https://www.reddit.com/t/valheim/` for collecting a list of posts, and an individual post URL when you want the thread.
2. **Send the request through the API.** `requests.get('https://api.scraperapi.com/', params={'api_key': ..., 'url': reddit_url})`. Every Reddit request goes through this endpoint so the anti-bot work happens upstream.
3. **Read the post attributes.** Each post lives inside an `<article class="m-0">` as a `shreddit-post` custom element. Its attributes carry `post-title`, `permalink`, `content-href`, `comment-count` and `score`; the same element supplies the author ID and name, the subreddit ID and name, and the post ID.
4. **Read the comments.** For each `div.thing[data-type="comment"]`, the author comes from `a.author`, the vote breakdown from `span.score.dislikes`, `span.score.unvoted` and `span.score.likes`, the timestamp from `time[datetime]`, and the body from `div.md p`. The guide skips any comment with no body text, which is what prevents the `NoneType` errors that otherwise kill a long crawl halfway through.
5. **Write the JSON.** `json.dump(parsed_posts, f, ensure_ascii=False, indent=2)` into `parsed_posts.json`, with comments nested inside their parent post record.

What you get is enough for most research: title, permalink, score, comment count, author, subreddit, plus a flat list of comments with author, score and timestamp. What goes missing is equally worth knowing. Comments without body text are dropped by design, and every field is pulled with a conditional, so a missing attribute becomes `null` rather than an error. Expect gaps in score and vote fields on newer or locked threads.

One caveat on the parsing code itself: those selectors are a snapshot of Reddit's page structure at the time the guide was written. `shreddit-post` is a custom element Reddit controls, so a front-end change can break your parser without any change on ScraperAPI's side. Budget an hour to re-inspect the DOM whenever your output suddenly goes empty.

## Feeding Reddit threads to an LLM without writing a parser

If the end goal is sentiment analysis or a summary rather than a database, add `output_format=markdown` to the request. The response comes back as the whole thread in clean Markdown: post title, community banner, upvotes, comment count, and the comment text itself, structured so a model can read it directly.

Gemini needs a Google API key, ScraperAPI needs its own key, and that's the entire pipeline. No BeautifulSoup, no selector maintenance, no edge cases in the DOM. For a one-off analysis of a few hundred threads this saves more time than any pricing tier difference will cost you, which is why it's worth testing during the trial rather than after you've built a parser.

## Cost per successful request beats requests per month

Comparing ScraperAPI plans by monthly request volume answers the wrong question. Three costs decide whether a Reddit pipeline is viable: the credit cost per page (which varies by domain, as above), the share of requests that come back usable, and how long a full run takes at your concurrency limit.

A commenter on r/AIToolBriefs, responding to a review of the platform, said the metric he'd watch is cost per successful request, and that pairing a scraping service with CAPTCHA solving makes a pipeline more predictable when targets fight back. A separate poster in r/bestaitools2025 made the operational version of the same argument: for batch jobs, compare failed-request rate, concurrency limits, retry behaviour and how fast you can finish a full run without partial or dirty data. His line was that a cheaper plan turning every crawl into babysitting gets expensive fast.

Applied to Reddit specifically, that means running one real crawl during the trial and measuring three numbers: credits spent, posts and comments kept after deduplication, and wall-clock time. If the failure rate is high enough that you're re-running jobs, the next tier up is usually the cheaper option.

## ScraperAPI vs a ready-made Reddit scraper

If you'd rather not maintain a parser at all, Apify's Reddit Scraper is the obvious alternative, and it works from a different angle. It reads Reddit's public JSON endpoints without login or OAuth, takes bulk start URLs and detects whether each is a post, subreddit or user profile, and supports keyword search with sort and time filters. Comment depth is capped by `maxCommentDepth` (1 to 20, default 5), subreddit listings can be sorted by hot, new, top, rising or controversial, and user profiles, submissions and comments are all available. Defaults are conservative: 10 items saved, 10 posts per page, 10 comments per page, 20 items per user, one second between requests. Proxies fall back from direct to datacenter to residential on a block, with up to three retries, and results stream into a dataset you export as JSON or CSV.

| Your situation | Better fit |
| --- | --- |
| You already have scraping code and only want the blocking and rendering handled | ScraperAPI, because you keep your own parser and swap the request layer |
| You want a finished dataset with no parser to maintain, and you're fine with a third-party platform's limits | Apify's Reddit Scraper |
| You need the thread as clean text for an LLM | ScraperAPI's `output_format=markdown`, which returns the whole thread already converted |
| You want keyword search across subreddits with sort and time windows without writing search logic | Apify's Reddit Scraper |

Neither approach makes Reddit's terms disappear. Both depend on public data, and usernames are personal data in several jurisdictions.

## FAQ

### Do I need a Reddit account or OAuth?

No. Requests go to ScraperAPI's endpoint with an `api_key` and the target `url`, so there's no Reddit login, app registration or OAuth flow in your code.

### Can I use the scraped data however I want?

ScraperAPI's blog says you're free to use the data as you like. Treat that as a vendor's position rather than legal advice. Reddit's terms still apply to the content, and if you're storing usernames, GDPR or CCPA obligations may apply depending on where you and your users are. Public data isn't the same as unregulated data.

### Will my parser break when Reddit changes its layout?

Eventually, yes. The selectors in the official guide (`article.m-0`, `shreddit-post`, `div.thing`) describe Reddit's markup at one point in time. The API request itself keeps working; the parsing code is yours to maintain. The Markdown output mode is the way to avoid that maintenance entirely.

### How many Reddit pages will 5,000 credits cover?

There's no fixed conversion. A credit cost depends on the domain and the parameters you send, and bot-protected sites add credits per request. Use the Domain Cost Estimator on your actual target URLs to get the number for your case.

### Can I get my money back?

Paid plans come with a 7-day, no-questions-asked refund, and you can cancel from the dashboard at any time without a cancellation charge. That window is long enough to run a real crawl and decide.

## Which setup fits your case

Run the trial against your real targets before you read the plan cards again. If you're doing a one-off study, a few hundred threads for a sentiment pass, or a quick check on what a subreddit thinks about a product, the 5,000 trial credits plus the free monthly 1,000 are enough to finish the work and you never need a paid tier. If you're monitoring several subreddits continuously and need runs to complete without babysitting, the decision moves to concurrency and overage behaviour: pick a tier with pay-as-you-go and a spending cap if a paused job costs you more than the credits do.

Two things to confirm before checkout: the current credit allowance and price for the tier you're considering, since those live only on the pricing page, and whether your target URLs cost more than one credit each, which the Domain Cost Estimator will tell you in about a minute.

👉 [Start with the 5,000-credit trial and price your own Reddit crawl](https://www.scraperapi.com/signup?fp_ref=coupons)
