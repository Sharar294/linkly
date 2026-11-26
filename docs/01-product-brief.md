# Linkly — Product Brief

Nov 25, 2026

## How to use this brief

This brief is written the way a product manager would hand it to an engineering team: it describes the product, the users and what the business cares about, but it does not list requirements for you. Your job is to extract them.

1. Read the whole brief once without taking notes.
2. On a second pass, pull out every **functional requirement** (something the system must do) and every **non-functional requirement** (how well it must do it: latency, availability, durability, security, scale, cost).
3. Turn vague statements into measurable ones. "Redirects should feel instant" is not a requirement until it has a number.
4. Do the back-of-envelope math in the traffic section yourself. The brief gives inputs, not answers.
5. List the ambiguities you find. Some are planted on purpose, and a few stakeholder wishes conflict with each other. Decide how you would resolve each one and write it down as an assumption.
6. Mark what is in scope for v1 and what you would defer.

A good result is a 2–3 page requirements doc with numbered requirements (FR-1, NFR-1, …) that you can trace later into your design doc, tests, SLOs and alerts.

## Background and vision

Linkly is a link-shortening service for small businesses, creators and community organizers who share links on social media, in printed flyers, in SMS campaigns and on QR codes. They want links that are short, look trustworthy and tell them whether their campaigns are working.

The product has two halves. The **public side** is the redirect: anyone in the world who clicks or scans a Linkly link must land on the right page, every time, quickly. The **customer side** is where people create links, manage them and see how they perform.

The redirect is the product. A customer may forgive a slow dashboard, but a broken link on 5,000 printed flyers cannot be recalled, and it makes the customer look bad in front of their own audience. Leadership's guiding line is: *"A Linkly link that worked yesterday must work today, and in five years."*

Linkly is launching as a new product. A small team (you) will build v1, run it in production and carry the pager for it.

## Who uses Linkly

| Persona | Who they are | What they want from Linkly |
| --- | --- | --- |
| **Maya, the café owner** | Runs one coffee shop. Prints QR codes on menus, table tents and window posters. Not technical. | Set up a link in under a minute, never think about it again. Change where a printed QR code points (seasonal menu) without reprinting. |
| **Dev, the creator** | Posts on Instagram and TikTok to 200k followers. Shares 5–10 links a week. | Branded, memorable aliases like `linkly.to/devs-gear`. See which posts drive clicks, and see it soon after posting. |
| **Priya, the nonprofit organizer** | Runs SMS campaigns to 40,000 volunteers before events. | Links that work the moment the text goes out, when everyone taps at once. Some links should stop working after the event. |
| **The visitor** | Anyone who clicks or scans a link. Never signs up. Often on a phone, often on a weak mobile connection. | Get to the destination fast, and not be sent somewhere dangerous. |
| **Sam, Linkly's support person** | Handles customer emails and abuse reports. | Find any link and its owner quickly. Disable a malicious link within minutes. |

## User journeys

**Journey 1: Maya prints a menu QR code.** Maya pastes the URL of her online menu into Linkly and gets back a short link. She downloads it as a QR code and sends it to the printer. Three months later her menu moves to a new website. She opens Linkly, edits the destination of the same short link, and every printed QR code now goes to the new site. She never wants to see an error page because she forgot to renew something.

**Journey 2: Dev launches a gear video.** Dev creates `linkly.to/devs-gear` before posting. If someone else already has that alias, he wants to be told immediately and offered alternatives. He posts the video at 6 p.m. Within a couple of minutes he checks Linkly to see whether people are clicking, and the next morning he checks which countries and which referrers (Instagram vs. TikTok) the clicks came from. He does not need per-second accuracy, but he gets annoyed if the numbers look frozen.

**Journey 3: Priya's SMS blast.** At 9:00 a.m. Priya's SMS tool sends one Linkly link to 40,000 phones. Most recipients tap within the first few minutes. Every one of them must land on the sign-up page. The link is set to expire at midnight after the event; after that, visitors should see a friendly "this link has expired" page, not an error.

**Journey 4: A malicious link is reported.** A visitor emails support saying a Linkly link leads to a fake bank login page. Sam looks up the link, sees who created it and disables it. From then on, anyone who visits it sees a warning page. Sam also wants to see whether the same account created other links.

**Journey 5: A developer integrates Linkly.** A small e-commerce shop wants to create short links automatically for every new product. They use Linkly's public API with an API key. They create a few hundred links a day, sometimes in bursts when they import a catalog.

## Feature details

**Creating links.** A customer submits a long URL and gets a short link back. Short codes are generated automatically unless the customer chooses a custom alias. Generated codes should be short enough to type from a printed flyer, and the business does not want competitors to be able to estimate how many links Linkly has by looking at codes. Custom aliases are case-insensitive, 3–30 characters, and can only use letters, numbers and hyphens. Some words are reserved because the product uses them (`api`, `admin`, `login`, `stats` and similar). Destination URLs must be valid `http` or `https` URLs, and the product team has said "no linking to ourselves," because redirect loops have caused problems at other companies.

**Redirecting.** Visiting a short link sends the visitor to the destination. Customers often change destinations (see Maya), and every visit must be counted for analytics. Unknown codes show a branded "link not found" page. Expired links show an "expired" page. Disabled links show a warning page. None of these should look like the site is broken.

**Managing links.** Customers can list their links, search them, edit the destination, set or remove an expiry date, and delete a link. Leadership is uneasy about deletion: if a deleted code is ever reused by someone else, old flyers would suddenly point to a stranger's site.

**Analytics.** For each link, customers see total clicks, clicks over time (hourly for the last 7 days, daily for the last year), top countries, top referrers and device type (mobile or desktop). Raw click data older than one year can be summarized. Legal has asked that visitor IP addresses are not stored in raw form.

**Accounts and API.** Customers sign in to manage their own links and can only see and change their own links. The public API lets customers do anything the dashboard can do, authenticated by an API key. Free-tier customers can create up to 50 links a month. The API must not let one customer's bulk import slow down everyone else.

**Abuse handling.** Support can disable any link and suspend any account. New destinations should be checked against a known-malicious URL list before the link is created. Every support action must be recorded: who did what, to which link, and when.

## Traffic and growth expectations

The business team's forecast is below. Use it to estimate requests per second, peak load, storage and cache size. Treat these as planning inputs, not guarantees.

| Metric | At launch | In 2 years |
| --- | --- | --- |
| Active customers | 10,000 | 500,000 |
| New links created per day | 50,000 | 2,000,000 |
| Redirects per day | 5,000,000 | 300,000,000 |
| Dashboard and API reads per day | 200,000 | 10,000,000 |

Things the business team mentioned in passing:

- Traffic is uneven. Evenings in North America and Europe are busiest, and a single viral post or SMS blast can send tens of thousands of clicks to one link within minutes.
- Roughly 20% of links receive about 80% of all clicks. Most links get a handful of clicks in their first week and almost none after that.
- A typical destination URL is about 100 characters, but some marketing URLs with tracking parameters exceed 2,000 characters.
- Links never expire unless the customer sets an expiry. Nothing is ever cleaned up automatically.
- Visitors are global, though most customers are in North America and Europe at launch.

## What stakeholders care about

These are things people said in kickoff meetings, in their own words. They are deliberately vague. Turning them into measurable targets, and deciding between them where they conflict, is part of the exercise.

- **CEO:** "Links can never go down. If a customer's QR code fails during their grand opening, we lose them forever."
- **Head of Product:** "Redirects need to feel instant, even on a phone in a subway. Nobody waits for a link shortener."
- **Head of Product:** "The dashboard can be a bit slower. Nobody minds a second or so when they're looking at charts."
- **Analytics lead:** "Every single click has to be counted. Customers pay us for the numbers, so the numbers must be exact."
- **Engineering manager:** "Counting clicks must never slow down a redirect. If analytics is having a bad day, links still have to work."
- **Customer success:** "When Maya changes her menu link, the new destination should work right away. She'll test it by scanning the QR code while she's on the phone with us."
- **Security lead:** "Nobody should be able to see or change another customer's links. API keys and passwords must never be stored in plain text, and we need to know about abuse within an hour."
- **Legal:** "We need to honor a customer's request to delete their account and data within 30 days. Don't store anything about visitors we don't need."
- **Finance:** "Infrastructure for v1 should stay under about $500 a month at launch. Don't build for Google scale on day one, but don't paint us into a corner either."
- **On-call engineer (you, in the future):** "I want to know something is wrong before a customer emails us, and I don't want to be paged at 3 a.m. for things that can wait until morning."
- **Head of Product, again:** "We'll want to add custom domains (like `go.mayascafe.com`) next year, so keep that in mind."

## Constraints, non-goals and open questions

**Constraints**

- One small team builds and operates the system, so it must be simple to deploy, debug and run on call.
- It runs on one cloud provider or Kubernetes, in one region at launch.
- Infrastructure budget as Finance stated above.

**Not in v1**

- Custom domains (planned for next year)
- Teams, shared workspaces and roles beyond "customer" and "support"
- Billing and paid plans (the free-tier limit still applies)
- A mobile app
- A/B testing or rotating one short link between several destinations

**Questions nobody has answered yet**

Answer these yourself as written assumptions in your requirements doc. You will find more as you read.

- When two customers request the same custom alias at the same moment, who gets it?
- When a customer deletes a link, does the code ever become available again?
- Should the same long URL shortened twice by the same customer return the same short link or two different ones?
- If the malicious-URL check service is down, do we block link creation or allow it and check later?
- How "exact" do click counts need to be, given the engineering manager's and analytics lead's statements?
- Should a bot or link-preview crawler (like a chat app generating a preview) count as a click?
- What does "never go down" mean in practice for a single-region system with a $500 budget?
