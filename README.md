# redis hosting: Vultr's hourly-billed Valkey plans, a price check against cheaper managed hosts, and how to pick the right tier

Most people searching for redis hosting want the same thing: an instance of Redis, or something that speaks its protocol, that someone else patches, backs up, and keeps online — without paying enterprise-platform money for it. It's a crowded market, and the prices are all over the place. One host sells 1 GB of managed Redis for $1.49/month. The official Redis cloud won't sell you a dedicated deployment for less than $200/month. Somewhere in between sits Vultr, and its pricing model is different from almost everyone else's.

This article walks through Vultr's managed caching service — which runs Valkey, the Redis-protocol-compatible fork — with the full plan list, an explanation of how hourly billing changes the math for short-lived projects, and an honest comparison with the cheaper niche hosts you've probably already seen. Short version: Vultr is not the cheapest option for a tiny hobby cache, and pretending otherwise would waste your time. Where it earns its price is flexibility, scale, and the ability to pay for exactly the hours you use.

> **Quick picks**
>
> - Most small-to-medium apps: **Vulcan — 1 GB RAM, $15/month**, billed hourly with the bill capped at the monthly rate. New Vultr accounts get $100 in credit for their first 30 days, which covers the first month of this plan several times over.
> - Cache-only workload on a tighter budget: Vulcan Budget, also 1 GB, at $10/month.
> - You want the absolute cheapest managed Redis that exists: niche hosts like Kerit sell 1 GB with persistence and daily backups for $1.49/month. Genuinely cheaper than anything Vultr offers, on a much smaller platform.
>
> 👉 [Create a Vultr account and put the trial credit toward your first cache](https://bit.ly/VulTR)

## Vultr's "Redis" is actually Valkey — here's why that doesn't break anything

Vultr sells this product under the name Managed Databases for Caching, and the engine behind it is Valkey. If you opened the Vultr console expecting the word Redis and got a different name instead, that's the entire explanation — there's no separate "Redis product" hiding elsewhere on the platform.

Some background, because you'll run into this everywhere in 2026: Redis changed its license in 2024 to a pair of non-open-source options, and the open-source community forked the last permissive version into Valkey, now hosted under the Linux Foundation. Valkey keeps the Redis wire protocol. Practically, that means your existing code connects to Vultr's service the same way it would connect to Redis: ioredis, node-redis, redis-py, go-redis, Lettuce — all of them work without changes. Job libraries built on Redis, like BullMQ or Celery or Sidekiq, work too. You swap in a connection string and carry on.

Every plan, cheap or expensive, ships with the same management layer:

- **Automatic daily backups**, restored into a fresh instance rather than overwritten in place
- **TLS for connections in transit**, with access over the public internet or locked to a private VPC network
- **Configurable eviction policies** — LRU, LFU, TTL-based, or no-eviction — set from the console
- **Read replicas** if you need them, each billed at the same rate as the primary
- **Metrics** for memory, connections, and throughput in the Vultr dashboard
- **32 data center locations** to choose from, so the cache can sit near your app servers

That last point matters more than it sounds. A cache is only as good as its round-trip time, and co-locating your compute and your cache in the same VPC on Vultr gets you low-latency, private-network traffic between them.

👉 [See the full feature list on Vultr's managed database for caching page](https://www.vultr.com/products/managed-databases-for-caching/?ref=7268283)

## Every Vultr caching plan and what it costs

Vultr currently lists 14 managed caching plans, split into a standard line and a Budget line. Prices below are from Vultr's pricing page at the time of writing; every plan is billed hourly, with the total capped at the monthly price.

**Standard plans:**

| Plan | RAM | Monthly price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Vulcan | 1 GB | $15/mo | Hourly, capped monthly | [ Deploy Vulcan](https://www.vultr.com/pricing/?ref=7268283) |
| Opteron | 2 GB | $30/mo | Hourly, capped monthly | [ Deploy Opteron](https://www.vultr.com/pricing/?ref=7268283) |
| Interlagos | 4 GB | $60/mo | Hourly, capped monthly | [ Deploy Interlagos](https://www.vultr.com/pricing/?ref=7268283) |
| Talos | 8 GB | $115/mo | Hourly, capped monthly | [ Deploy Talos](https://www.vultr.com/pricing/?ref=7268283) |
| EPYC 7402 | 16 GB | $230/mo | Hourly, capped monthly | [ Deploy EPYC 7402](https://www.vultr.com/pricing/?ref=7268283) |
| EPYC 7452 | 24 GB | $345/mo | Hourly, capped monthly | [ Deploy EPYC 7452](https://www.vultr.com/pricing/?ref=7268283) |
| EPYC 7502 | 32 GB | $460/mo | Hourly, capped monthly | [ Deploy EPYC 7502](https://www.vultr.com/pricing/?ref=7268283) |

**Budget plans** — same RAM, roughly a third off the price, with the CPU allocation cut in half:

| Plan | RAM | Monthly price | Get it |
| --- | --- | --- | --- |
| Vulcan Budget | 1 GB | $10/mo | [ Deploy Vulcan Budget](https://www.vultr.com/pricing/?ref=7268283) |
| Opteron Budget | 2 GB | $20/mo | [ Deploy Opteron Budget](https://www.vultr.com/pricing/?ref=7268283) |
| Interlagos Budget | 4 GB | $40/mo | [ Deploy Interlagos Budget](https://www.vultr.com/pricing/?ref=7268283) |
| Talos Budget | 8 GB | $75/mo | [ Deploy Talos Budget](https://www.vultr.com/pricing/?ref=7268283) |
| EPYC 7402 Budget | 16 GB | $155/mo | [ Deploy EPYC 7402 Budget](https://www.vultr.com/pricing/?ref=7268283) |
| EPYC 7452 Budget | 24 GB | $230/mo | [ Deploy EPYC 7452 Budget](https://www.vultr.com/pricing/?ref=7268283) |
| EPYC 7502 Budget | 32 GB | $305/mo | [ Deploy EPYC 7502 Budget](https://www.vultr.com/pricing/?ref=7268283) |

The hourly billing deserves its own paragraph, because none of the niche competitors in this article offer it. Vulcan's $15/month works out to roughly $0.022 per hour. Run a cache for a one-week load test and you'll pay about $3.70, then destroy the instance and stop paying entirely. Run it all month and the bill caps at $15. Compare that with the typical monthly-subscription model, where even a three-day project costs a full month at Kerit (money-back window is only three days there) or a fixed $9.99 at Galaxy. For steady workloads, hourly billing changes nothing. For anything temporary, spiky, or experimental, it's a genuine cost difference.

One note on the Budget line: a Redis cache is usually memory- and network-bound rather than CPU-bound, which is why the halved CPU allocation is a reasonable trade for a third off the price. If your workload is heavy on Lua scripting or complex sorted-set operations, stick to the standard line. For plain caching and session storage, Budget is hard to argue with.

👉 [Check Vultr's current caching prices and regional rates](https://www.vultr.com/pricing/?ref=7268283)

## How Vultr's prices stack up against other managed Redis hosts

Here's the honest part. On raw sticker price for small plans, Vultr loses to several of the niche hosts. What follows is the comparison with the numbers each service currently publishes — judge for yourself which end of the trade-off you're on.

| Service | Cheapest paid plan | RAM included | Billing model | Worth knowing |
| --- | --- | --- | --- | --- |
| Kerit Cloud | $1.49/mo | 1 GB | Monthly | Persistence (RDB + AOF) and daily backups included; 3-day money-back guarantee |
| Galaxy | $9.99/mo | 512 MB | Monthly | Zero egress fees; database runs on the same network as your app |
| Layerbase | Free | 5 GB storage, 2 databases | Free tier, then $5 or $15/mo flat | Free tier sleeps after 15 minutes idle; paid plans stay awake |
| openredis | $12/mo | 50 MB | Monthly | Every plan includes a master + replica pair with automatic failover; hourly backups stored in S3 |
| Redis Cloud (official) | ~$5/mo (from $0.007/hr) | From 250 MB | Hourly, shared infrastructure | 30 MB free tier; dedicated Pro plans start at a $200/month minimum |

A few conclusions worth drawing from those numbers, because the headline prices hide a lot.

Per gigabyte, Vultr's mid-tier plans are far more competitive than its entry plans suggest. openredis charges $135/month for 1.7 GB on its Large plan; Vultr's Talos gives you 8 GB for $115. At the top end the gap widens further — openredis's Giant plan is $750/month for 10 GB, while Vultr's EPYC 7452 Budget covers 24 GB for $230. To be fair to openredis, every one of its plans includes an automatic-failover replica, which Vultr only provides if you add one and pay for it. If master-plus-replica redundancy is non-negotiable and you're staying small, that built-in pair is real value.

At the small end, Kerit is simply cheaper, full stop. Its $1.49 Standard plan includes RDB + AOF persistence and daily backups with 7-day retention — features Vultr also includes, at ten times the price. The reasons to pay Vultr's $15 instead are the ones Kerit can't offer: 32 regions, hourly billing with no monthly commitment, dedicated resources, tight VPC integration if your servers already run on Vultr, and the trial credit that makes the first month effectively free.

The official Redis Cloud service is the interesting middle case. Its Essentials tier is also hourly-billed, from $0.007/hour, and scales from 250 MB up to 100 GB — but on shared infrastructure, and the moment you want a dedicated deployment (the Pro tier), you're at a $200/month minimum. Vultr fills that gap with dedicated plans at $10–$460/month, which is exactly the range where a lot of real production caches live.

## Which plan fits your workload

Match your situation to one of these and you're done deciding.

**A side project or learning sandbox.** Honesty first: Vultr's cheapest plan is $10/month, and Layerbase's free tier (5 GB, sleeps when idle) or Kerit's $0.85 Hobby plan cost less. Use Vultr here only if you're already in its ecosystem and want everything on one bill and one console.

**Production cache for an app already running on Vultr.** This is the sweet spot. Put a Vulcan or Vulcan Budget instance in the same region as your compute, attach it to your private network, and your cache reads and writes travel over the internal network instead of the public internet. The $100 trial credit will cover a full month while you benchmark it.

**Short-lived projects, load tests, seasonal traffic.** Hourly billing with a monthly cap exists for exactly this. A two-week campaign cache on Vulcan costs about $7.50 total. No monthly plan anywhere else in this comparison comes close to that granularity.

**Job queues and real-time leaderboards in the 4–8 GB range.** Interlagos ($60) or Talos ($115) standard, or their Budget versions at $40 and $75. Queues are more tolerant of Budget's reduced CPU than you'd think, but if your streams have thousands of consumer-group reads per second, take the standard line.

**16 GB and up.** Compare EPYC 7452 Budget ($230 for 24 GB) against anything the niche hosts sell in that bracket and the decision makes itself. If you also need multi-region active-active replication and enterprise compliance features, the official Redis Cloud Pro tier is the alternative worth evaluating — at its $200/month entry point.

## redis hosting on Vultr: FAQ

**Is Vultr's managed database real Redis?**
It's Valkey, the Linux Foundation fork of Redis created after Redis's 2024 license change. It speaks the Redis wire protocol, so every mainstream client library and Redis-based tool connects without code changes. For caching, sessions, queues, pub/sub, and sorted-set leaderboards, it behaves exactly as you'd expect.

**Do I need to change my application code to use it?**
No. Copy the connection string from the Vultr console into ioredis, node-redis, redis-py, go-redis, or Lettuce, and connect over TLS. Existing BullMQ, Celery, and Sidekiq setups work against it as-is.

**What happens when the memory fills up?**
You choose the eviction policy from the console — LRU, LFU, TTL-based, or no-eviction — and you can move to a bigger plan whenever you need more RAM. For a cache, LRU is the usual default; for a queue, no-eviction is safer because silently dropping keys breaks consumers.

**How do backups work?**
Every managed caching plan takes automatic daily backups, and restores land in a fresh instance rather than overwriting your live database. That means a botched restore can't take down production.

**Can I try it before committing to a monthly price?**
Yes, twice over. Vultr bills hourly with the bill capped at the monthly rate, and new accounts get $100 in credit valid for 30 days. Between the two, you can run a Vulcan instance for weeks without paying anything out of pocket.

## Before you click sign-up

Three quick checks that will save you a support ticket later.

First, size the plan on your worst day, not your average one. Cache evictions are silent, and the symptom — your primary database suddenly taking load — shows up at the worst possible time. If you're within 20% of a plan's RAM on a busy day, buy the next tier up; the hourly model makes downsizing later painless.

Second, pick the region where your application servers live, and if they're already on Vultr, attach the database to your private network. A cache 200 milliseconds away from your app is worse than no cache.

Third, remember the Budget trade: same RAM, half the CPU, about a third off. Fine for caching and sessions, worth skipping for script-heavy or high-throughput queue workloads.

And if you'd rather start on a $1–$2/month niche plan and migrate later, that's a defensible path too — the connection string is portable, so switching hosts later is an afternoon, not a project. But if your app already lives on Vultr, or you want hourly billing and room to grow from 1 GB to 32 GB on one platform, the decision is easy:

👉 [Sign up for Vultr and deploy your first managed cache with the trial credit](https://bit.ly/VulTR)
