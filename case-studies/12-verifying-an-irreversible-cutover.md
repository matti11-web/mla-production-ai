# Verifying an Irreversible Cutover

> **In business terms**
>
> - **Problem:** The company website is the channel for most of the company's won sales. It has around 1,000 public pages in two languages. A move to a new platform is a DNS switch. After the switch, the old site can no longer be compared. A page lost in the move is lost search traffic and lost leads, and nobody sees it until sales drop.
> - **What changed:** Before the switch, automated checks compared every page between old and new: the address, the content and the look. Checking about 1,000 pages by hand is not possible in practice. The checks did it on every run.
> - **Result:** The switch happened on 28 August 2026. Before it, the checks found 110 missing call-to-action buttons on 102 pages in the build that was going to ship. That defect was fixed before the switch. The way back to the old site was a single edge rule that takes seconds, not a DNS change that takes hours.

This case study leads with pages at risk, not with money saved. I have no verified figure for the cost of the old platform against the new hosting. I have no verified search traffic figures from before and after the switch. The money figure here is the share of sales that depends on the site, which is what a failed cutover puts at risk.

**Status (28 October 2026):** The cutover ran on 28 August 2026 at 13:14 UTC, two days after the planned date. The site has run on the new stack since then. It has not been rolled back. The 110 missing buttons were closed on 18 August 2026, before the switch. A full re-import of the production data and a new measurement then found zero real differences between staging and production. This case study is about the proof before the switch, not about the results after it.

---

## What was at risk

- **Sales.** The website is the channel for most of the company's won sales. The contact and quote forms are the main way leads arrive.
- **Pages.** 1,025 public URLs at the time of the switch: 513 in Dutch and 512 in French. Each one can be bookmarked, linked from another site, or ranked in search.
- **Search position.** The domain has about 20 years of search history. Search engines do not reset that history when the domain stays the same. The risk is in the execution: a changed or missing URL loses its ranking.
- **Effort.** The plan for the whole migration was about 242 hours over 36 work items. A cutover that fails and has to be repeated burns a large part of that again.

## The shape of the problem

An SME's public website moved from a legacy PHP CMS to a static-site generator with a headless CMS behind it, served from the edge. Around **1,027 public paths**, built up over years by several agencies, in two languages.

The rebuild is the easy half. The hard half is this: cutover is a DNS change. The moment it happens, the old site stops being observable. You cannot compare afterwards, because "afterwards" is the thing you were supposed to compare against. Every question you did not ask before the switch becomes unanswerable at the exact moment it becomes urgent.

So the deliverable was not a website. It was a battery of gates that had to be green *while both systems still existed*.

## The gates

### 1. Path parity — does every URL still resolve?

Enumerate every live URL from the old system, resolve each against the new one, and require an explicit outcome per path: matched, intentionally redirected, or intentionally gone. No path is allowed to have no decision attached to it.

This is the boring gate and it is the one that saves you. A 404 on a page nobody remembers is still a page some customer bookmarked.

### 2. Content parity — is the same content actually there?

Per path, compare the sequence of content blocks between old and new. Not a text diff — a structural one. A text diff on a rebuilt site is noise. Every whitespace and markup change fires. The sequence of blocks is the thing that has to survive.

### 3. Visual parity — a sampled pixel diff, with every outlier explained

Render matched pairs on both systems and compare images. Sixty desktop pairs, sampled across page types rather than taken from the top of the list.

The rule that made this gate useful: **an outlier is not allowed to be waved away.** Eight pairs exceeded the difference threshold. Six of those eight traced to a single known cause — the live site uses a fixed-height carousel where the rebuild uses a responsive grid, an intended design change. That explanation had to be written down and matched against each affected pair individually. The remaining two were real defects and were fixed.

"Mostly the same, the rest is probably fine" is not a gate. It is a gate-shaped feeling.

### 4. DNS control moved *before*, and separately from, the cutover

Nameservers moved to the new provider well ahead of the cutover date, with the origin still pointing at the old host. Nothing about the live site changed. What changed is who can change it.

That move got its own verification: **53 of 53 (name, type) record pairs identical to the pre-move snapshot**, zero records lost, partial or missing, origin address unchanged, and the mail path tested down to the SMTP banner — because the fastest way to turn a website migration into a company-wide incident is to drop an MX record nobody was looking at.

The verification tool is read-only and is a *separate program* from the one that creates records. Same reason you do not put the safety catch on the trigger.

## The finding that mattered

Every gate above ran green — against staging.

The cutover ships from production.

When the same comparison was pointed at the production build, it surfaced **110 call-to-action buttons missing across 102 pages**. The content-block sequences were identical on all 1,027 paths, which is precisely why the defect was invisible to the structural gate: the blocks were all present, and a property inside them was not.

Staging and production had drifted. Every green result I had was a true statement about an artifact I was not going to ship.

That became a hard precondition in the cutover runbook: **re-run the full battery against the production build, and treat a staging-only pass as no pass at all.** It cost a day. It would have cost a week of "why did our contact conversions drop after the relaunch" if it had been found later — or never, which is worse.

## What generalises

- **Split the irreversible step into the smallest possible atom, and verify each precursor on its own.** Moving DNS control, changing the origin, and decommissioning the old host are three separate events. Doing them as one heroic Tuesday means a rollback has to undo all three.
- **Run the gate against the artifact you will actually ship.** Not the branch, not staging, not "it is basically the same build". This is the single most transferable line in this write-up.
- **Every visual outlier gets a written explanation or a fix.** There is no third bucket. The moment "known difference" becomes a shrug instead of a sentence, the gate stops working.
- **Structural comparison misses attribute-level loss.** Block sequences matched perfectly while 110 buttons were gone. Any comparison you build has a blind spot. The useful question is which one, and what second check covers it.
- **Verification tools are read-only, and separate from the tools that write.** A read path that shares a binary with a write path will eventually be run in the wrong mode by a tired person at 23:00.
- **The old system does not get patched after cutover.** Nobody maintains a CMS they have replaced. If the old origin stays reachable at all, it needs a firewall in front of it, not good intentions.

---

## What's NOT in this case study

- The domain, the hosting providers, the agencies involved, and any record values
- Repository paths, script internals, and the runbook itself
- Traffic and conversion figures, and absolute revenue figures (only the share of sales that the site carries is given)

---

**Verification built:** July–August 2026. **Cutover:** 28 August 2026, 13:14 UTC. **Last updated:** 28 October 2026.
