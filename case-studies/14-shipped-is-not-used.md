# Shipped Is Not Used: Measuring Adoption After the Merge

> **In business terms**
>
> - **Problem:** A feature can be merged, deployed and verified in production, and still never be used. The pull request does not tell you.
> - **What changed:** After shipping, I query the real database and the real usage data. I treat that result as the status of the feature, not the merge.
> - **Result:** One tool that looked finished had one record, and that record was my own test. A tutorial series had grown past its own length limit without anyone noticing. Daily use by the sales team was measured, not assumed. Each result changed what I did next.

I build internal software for a building company with AI. Merging is cheap now. That makes it easy to confuse "shipped" with "done". This case study shows three measurements I took after the merge, and what each one changed.

**Status:** Plan wrapper measured on 3 August 2026. Tutorial series measured on 3 August 2026. Daily use measured on 21 October 2026, over the 30 days from 21 September to 20 October.

---

## Case one: a tool with one record

The Hub has a tool that wraps supplier drawings with a company cover page and a standard text appendix. A project manager uses it to send plans to a client.

The last series of pull requests ran from #450 to #455. All of them merged. Both database migrations ran in production and I checked them on content. There were no open pull requests and no loose branches. By every engineering measure, the work was finished.

Then I counted rows in the production database:

| Table | Rows |
|---|---|
| Wraps made | 1 |
| Standard plan texts | 0 |

The one wrap was my own test. The tool had never had a real user. With zero standard texts, it produced an empty appendix.

This had happened before in the same project. An older text library never got a screen to edit it. Its appendix stayed empty for months, and nobody saw that as a defect. The lesson I wrote down: **what can only be edited with hand-written SQL does not get edited.** That is why one of the last pull requests added an editing screen for the plan texts.

What the count changed: I did not close the project. The old engine still runs in production, and one work item stays open to remove it. It stays open until real users make real wraps.

## Case two: a tutorial series past its limit

The CRM has a tutorial series for the sales team: 14 videos, in Dutch and French. The goal was that a sales rep can learn the CRM alone. The series had a limit of 20 minutes in total.

An earlier note had warned that a 14th video would almost certainly break the limit. The 14th video was added anyway. Nobody checked the total after that.

I measured all 14 files on 3 August 2026. The Dutch series runs 20:20. The French series runs 20:22. Both are over the limit. The warning came true and was never noticed.

What the measurement changed: the question became a decision for me, not a background fact. The options are to drop the limit, shorten a video, or remove the 14th video.

The usage measurement below adds a second fact. On 21 October 2026, the academy module in the Hub was close to unused. A series that exists and is linked is still not a series that is watched.

## Case three: daily use by the sales team

I wanted to know if the sales team really works in the Hub and the CRM, or only logs in. The apps record visible-tab time and page views per user per day. I query them read-only.

Measured on 21 October 2026, over the last 30 days:

- All 7 active sales reps use the CRM 4.0 to 4.6 workdays per week.
- The median is 27 minutes per rep per active day.
- Use jumped from about 2.5 to about 4.4 days per rep per week around 1 September 2026. That was the cutoff date for the old way of working.
- In the Hub, sales mainly use the calculators and the pre-analysis tool.
- Meetings, academy, checklists and the document vault were close to unused.

The limits of this data are part of the result. Minutes come from a heartbeat every 30 seconds while the tab is visible. A tab left open still counts. A page view is not an action. So I read these figures as "people are there", not as "people do the work there".

What the measurement changed: daily use of the CRM is now measured, not assumed. The modules with almost no use are a separate list. For those, the next question is why, not what to build next.

## The pattern

In all three cases, the code was not the problem. The merge was clean and production was correct. What was missing was a real person using the thing. Only a count after the merge showed that.

The salon case (case 13) shows the same gap from the other side: the build took four days, the switch to daily use took seven weeks.

## What generalises

- **The status of a feature is a database count, not a merge.** "Merged and verified in production" described the plan tool well. It still had one record.
- **Count rows before you close a project.** One query on the production tables took minutes. It kept a project open that I was ready to close.
- **A warning in a note is not a check.** The tutorial limit was written down and still broken. A limit only holds if something measures it again.
- **No editing screen means no data.** If only a developer can fill a table, the table stays empty.
- **State the limits of your usage data.** Visible-tab minutes show presence, not work. Say that next to the number.
- **Measure, then decide what to build.** Real use moved my next step away from more features and toward asking users why.

---

## What's NOT in this case study

- Names of colleagues, sales reps or clients
- Per-person usage figures
- CRM conversion or adoption rates
- Prices, margins or revenue
- Internal URLs, repository paths, table schemas and code

---

**Plan wrapper and tutorial series measured:** 3 August 2026. **Daily use measured:** 21 October 2026, over 21 September to 20 October 2026. **Plan wrapper:** no real user on 3 August 2026. **Last updated:** 28 October 2026.
