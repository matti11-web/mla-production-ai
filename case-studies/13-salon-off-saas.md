# Taking a One-Person Salon Off a Paid SaaS

> **In business terms**
>
> - **Problem:** A one-person dog grooming salon paid a monthly subscription for booking software that did not fit how it works. The software had no concept of a dog, so it priced every breed as a separate service.
> - **What changed:** Custom software built around the dog, the coat and the owner's own price list. It ran on real data four days after the first commit.
> - **Result:** The salon has worked in it every day since mid-September. 136 appointments booked in it so far, 29 of them online by clients. The build took four days. The switch took seven more weeks.

A one-person grooming salon ran its agenda, client file and online booking on a paid salon SaaS. I replaced it with software built for this one business. This is what the build was, and why the build was the short part.

**Status:** First commit 23 July 2026. Running on the real database 27 July 2026. Daily use in the salon since 14 September 2026. Figures measured on 28 October 2026. The old subscription is not confirmed cancelled at the time of writing.

---

## Why replace it at all

The subscription cost €45 a month, plus €0.15 per SMS. The price was not the problem. The fit was.

The SaaS has no concept of a dog. It knows clients and services. So the salon priced its work as one service per breed: 84 services, many with the breed name misspelled, because the misspelled service *was* the price. A returning client could not book "the usual for Max". They had to find their breed in a list of 84.

The owner wrote down what she wanted from the screen she works in all day. Eleven points. Search from the first letter. A client file with only name, phone, email and dogs. A dog file with only what matters during a groom: coat length in millimetres, and notes like "bites" or "needs a muzzle". Very little typing. Nothing in that list is exotic. The SaaS just was not shaped around it.

## The four days

The first commit landed late on 23 July. The app ran on the real database on 27 July.

| Date | What landed |
|---|---|
| 23 Jul | Research, roadmap with acceptance criteria per work item, data model |
| 24 Jul | Extraction from the old system, a new price model, idempotent import |
| 25 Jul | The real database, agenda, client and dog files, booking, checkout and a daily receipts book |
| 26 Jul | Smarter search, offline-capable app |
| 27 Jul | Deploy, login screen, rate limiting on login |

38 pull requests merged by the end of 27 July, each through CI. 134 merged by 25 October 2026.

Stack: Cloudflare Workers, D1 (SQLite at the edge), React, TypeScript, Astro for the public site, Playwright for screenshots and checks.

## The price model was the real design work

The old price list carried 84 per-breed services. I replaced it with a grid: five coat groups (smooth, curly, clipped, hand-stripped, woolly) by size. 16 cells of that grid carry a price. Those prices come from the owner's own internal service list, not from me.

Seventy breeds then had to land in a cell. The owner delegated that: "the price gives away the category and the size." That is a rule, so I applied it. Each breed went to the cell closest to the median of its real receipts, from 255 receipt lines, with a minimum of two receipts. Seven breeds moved because their receipts disagreed with their name.

One nuance stayed visible. The price reveals the **size**, not the **coat group**. €70 appears in three groups. Where the data could not decide, the choice is written down next to the breed, separate from the decided ones.

The import also surfaced errors that clients could already see on the public price list. One breed with no name carried a price of nine digits. Hand-stripping a wire-haired dachshund was priced at €9.

## Three bugs that only real data showed

**The clock had no time zone.** The old system exports Brussels wall-clock time with no offset. Adding a `Z` would have moved every appointment one or two hours, depending on daylight saving time. The import converts it properly, with tests on both sides of each DST switch.

**76 appointments pointed at clients that no longer existed.** They belonged to 21 clients who were missing from the client export. A plain foreign key would have dropped them silently. The appointment rows still carried a name and a phone number, so the import rebuilt the client from those.

**The re-import would have erased the owner's corrections.** The import is idempotent, so the final switch is just a re-run for the new data. But every re-run set name, coat length and status back to the old system's values. Every row now carries a `manually_edited` flag, and the import never overwrites a row that has it.

## Green tests, nothing running

By 26 July the first two phases were code-complete. Nothing was deployed. The database existed, but there was no Worker and no site. Code-complete is not running.

Then it was deployed, and I opened it myself. **There was no login screen.** The API had a login endpoint from the first days, and its tests passed. The app had never had a screen that called it. Every request returned 401, and the app showed an empty agenda. That looks like a quiet Monday, not like an error.

The second check that mattered: I logged in with a deliberately wrong password. The answer was "wrong password", not a server error. That one answer proves the user row exists and the password hash ran on the real runtime. The local test runtime had passed the same path for days while it was broken on the real one. Testing with a wrong password also keeps the real one out of every log.

## The switch took seven weeks

Software running on day 4 did not mean the salon used it. The real switch needed things no pull request could do:

- the salon's own domain moved off the old website builder, with mail records kept intact;
- a sending domain authenticated for confirmation mails;
- the owner logged in on her own phone, on the new address;
- a final delta import of everything booked in the old system in the meantime.

Daily use started on 14 September 2026.

Then the first leak. A week later the owner said a client had booked in the old system. The cause was my own site: every booking button on the new website still pointed at the old booking page. Three online bookings had landed there after go-live. **Two of them were not in the new system.** One fix redirected the buttons. A second fix made the site build fail if the old booking address appears anywhere in it.

## Where it stands (28 October 2026)

| Measure | Value |
|---|---|
| Clients in the file | 713 |
| Dogs in the file | 795 |
| Appointments booked in the new system since 15 September | 136 |
| …of which booked online by clients | 29 |
| …of which booked in the salon | 107 |
| Merged pull requests | 134 |

## What generalises

- **The build is the short part.** Four days to working software, seven weeks to daily use. The gap was domains, accounts, a phone, and the last import, not code.
- **Model the business, not the old tool's workaround.** 84 misspelled services were the old tool's shape, not the salon's. The salon's shape was coat by size.
- **Delegation with a rule is a decision, not a question.** When the owner gave a rule ("the price gives away the size"), applying it was the job. Sending eleven questions back would have stalled it.
- **Open the deployed thing yourself.** Green tests did not show a missing login screen. One person opening the app did.
- **Close the old door from your own side.** The leak came from my own links, not from the old system.

---

## What's NOT in this case study

- The salon's name, location, website or anyone working there
- The name of the SaaS it replaced
- Client names, dog names, phone numbers, or any record from either system
- Revenue, prices beyond the examples above, and the owner's price list
- Repository paths and code

---

**Built:** 23–27 July 2026. **Daily use since:** 14 September 2026. **Figures measured:** 28 October 2026. **Old subscription:** not confirmed cancelled at the time of writing. I will add a dated status line when it is, not rewrite the tense.
