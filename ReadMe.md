# The Anatomy of a Request

### System Design, From Scratch — Part 1 of 4

---

It's 9:47 PM on a Friday. You tap play on a show. The screen loads in under a second. You don't think about it for even a moment — you just watch.

But somewhere, in the half-second between your tap and the first frame, a small miracle happened. Your request traveled through a chain of systems that had to get *everything* right: find the right address, reach a healthy server, survive a possible crash, and come back to you before you noticed anything was wrong.

This series is about that chain. Not the buzzwords — the actual mechanics. We'll start small and only add complexity when the previous layer genuinely breaks, because that's how real systems evolve too.

By the end of this first part, you'll understand the six ideas that quietly hold up almost every product you use: Instagram, your banking app, Netflix, all of it.

---

## Why bother understanding this at all?

Most engineers meet "system design" as an interview format — a whiteboard, a 45-minute clock, a request to design something Twitter-shaped. That framing makes it feel like trivia: memorize the load balancer, memorize the cache, recite it back.

It isn't trivia. It's the difference between a product that degrades gracefully and one that falls over completely the first time it gets popular.

The good news: almost everything below is built from the same handful of ideas, reapplied at different scales. Learn the idea once, and you'll recognize it in every system you read about afterward.

---

## Where does traffic even come from?

Before a request reaches your server, it has to *find* it. Type `app.demo.com` into a browser, and before anything else happens, your device asks a question into the void: "Where does this name actually live?"

That question goes to **DNS** — the internet's phone book. DNS looks up the domain, finds the matching IP address, and hands it back. Only then does your device know where to actually send the request.

```
app.demo.com  →  DNS  →  172.16.254.254
```

Once resolved, the request lands from one of two doors, and the two doors want completely different things.

**A browser wants the whole page.** Chrome, Safari, Firefox — none of them know what your product looks like until your server tells them, in HTML, CSS, and JavaScript. It assumes a fairly stable connection, keeps you logged in with cookies, and always shows the freshest version on refresh.

**A mobile app already has the layout.** It was compiled into the app weeks ago. All it wants from your server is raw facts — JSON, not HTML. It authenticates with tokens instead of cookies, has to gracefully support users on last year's version, and assumes the connection might drop at any second.

Same request. Completely different expectations on the other end.

---

## Splitting the load: web tier vs. data tier

A single server answering requests and storing data works fine — until it doesn't, because **not everything strains at the same rate.**

As users grow, split the system into two tiers:

- **Web tier** — handles incoming requests, returns HTML or JSON. When traffic spikes, add two or three more servers behind a load balancer. Cheap, fast, repeatable.
- **Data tier** — the database. Scaling this is a different exercise: read replicas, sharding, careful migrations.

Keeping them separate means the web layer can grow freely without ever touching the database.

---

## Choosing a database: relational vs. non-relational

"But which database?" The honest answer: it depends on whether your data wants to be a **spreadsheet** or a **folder**.

**Relational databases** (PostgreSQL, MySQL, Oracle) are a perfectly organized spreadsheet — fixed tables, fixed columns, clear relationships. That rigidity buys strong consistency, exactly what you want for a bank balance or a checkout flow.

**Non-relational databases** (MongoDB, Cassandra, Redis) are an open folder — documents, key-value pairs, graphs, whatever shape the data actually has. No rigid schema, built to scale horizontally with very low latency.

> Rule of thumb: fixed shape + 100% accuracy required → relational. Flexible shape + huge volume + raw speed → non-relational.

---

## Scaling up vs. scaling out

"Code is easy. Keeping it alive when a million people show up at once — that's the actual job." There are exactly two ways to respond to that pressure.

**Vertical scaling** turns your old computer into a superhero: same machine, bigger engine. Almost no code changes required — but there's a hard ceiling, and that one machine is still a single point of failure.

**Horizontal scaling** buys more ordinary cars instead of one giant bus. Scalability becomes close to infinite, and losing one server doesn't take the system down. But now something has to stand in front of all of them, dividing the traffic.

---

## The load balancer: traffic's police officer

10,000 people hit your site at once, and you're running three servers. Send everyone to server one and it crashes, while the other two sit idle. Someone has to direct traffic.

The load balancer sits at the very front and hands off every request using one of a few algorithms:

- **Round Robin** — straight down the line: A, then B, then C, then back to A.
- **Least Connections** — whichever server is least busy right now gets the next request.
- **IP Hash** — a given user's IP always lands on the same server.

It also runs health checks constantly, pinging every server behind it to confirm it's actually alive.

---

## The hidden cracks: single points of failure

Look closely at that setup and two things stand out: everything depends on that one load balancer, and every server depends on that one database. Kill either, and the whole product goes down — even if every other component is perfectly healthy.

A **single point of failure** is any one component that, if it disappears, takes the entire system with it. The fix isn't a clever algorithm. It's refusing to let anything important exist in a quantity of one.

---

## Redundancy: always have a backup engine

A commercial airplane always flies with two engines. If one fails mid-flight, the other lands the plane safely on its own. Apply the same logic to your load balancer.

Run two instead of one. Traffic flows to the active one as normal; the passive one watches and waits. The moment the active balancer crashes, the standby takes over instantly — no one paged at 3 AM.

---

## Health checks: the heartbeat monitor

Having a backup is useless if nothing knows when to use it. This is exactly like an ICU heart-rate monitor: a steady beep confirming the patient is stable, and an instant alarm the second something changes.

Load balancers do the same thing to every server behind them — pinging on a tight loop, quietly removing any server that stops answering from the rotation until it recovers.

---

## Self-healing systems: the lizard's tail

When a lizard loses its tail, it doesn't wait for a vet. It just grows another one.

Modern cloud infrastructure does the same thing. The moment a health check flags a dead server, automation — AWS Auto Scaling and similar tools — terminates the broken instance and spins up a fresh one in its place, usually within seconds, with no engineer touching a keyboard.

---

## Putting it together: a Friday night on Netflix

Back to 9:47 PM. Here's every idea above, happening in under a second, while you're mid-episode and don't notice a thing:

**Server #45 crashes** → **Health check detects it** → **Your stream shifts to Server #46** → **A new #45 spins up in the background**

Redundancy meant a backup server was already running. The health check caught the failure in a fraction of a second and rerouted your stream before you noticed any buffering. Self-healing quietly rebuilt the failed server, so capacity never actually shrank.

That's not luck. That's the design holding under exactly the kind of pressure it was built for.

---

### Coming up in Part 2

Now that requests can survive a crash, the next problem is speed — and it starts with everything in this post hitting the database far more than it needs to. We'll cover caching, message queues, and the art of not asking the database the same question twice.

*→ [Try the interactive version of this post, with animated diagrams](#) — and grab the full source on [GitHub](#).*

---

*If this was useful, the best thing you can do is share it with one person who's about to walk into a system design interview, or just wants to understand why the apps they use don't fall over.*
