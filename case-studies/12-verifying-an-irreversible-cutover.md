# Verifying an Irreversible Cutover

How I built the evidence to switch a ~1,000-page company website onto a new stack, when the switch itself is a DNS change and there is no diff afterwards.

**Status:** Verification complete and green. **The cutover itself is scheduled for 26 August 2026 and had not been executed at time of writing.** This case study is about the proof, not the outcome — publishing it as a completed migration would be the exact failure mode it argues against.

---

## The shape of the problem

An SME's public website moved from a legacy PHP CMS to a static-site generator with a headless CMS behind it, served from the edge. Around **1,027 public paths**, built up over years by several agencies, in two languages.

The rebuild is the easy half. The hard half is this: cutover is a DNS change. The moment it happens, the old site stops being observable. You cannot compare afterwards, because "afterwards" is the thing you were supposed to compare against. Every question you did not ask before the switch becomes unanswerable at the exact moment it becomes urgent.

So the deliverable was not a website. It was a battery of gates that had to be green *while both systems still existed*.

## The gates

### 1. Path parity — does every URL still resolve?

Enumerate every live URL from the old system, resolve each against the new one, and require an explicit outcome per path: matched, intentionally redirected, or intentionally gone. No path is allowed to have no decision attached to it.

This is the boring gate and it is the one that saves you. A 404 on a page nobody remembers is still a page some customer bookmarked.

### 2. Content parity — is the same content actually there?

Per path, compare the sequence of content blocks between old and new. Not a text diff — a structural one. A text diff on a rebuilt site is noise; every whitespace and markup change fires. The sequence of blocks is the thing that has to survive.

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

## What I would tell anyone facing the same switch

- **Split the irreversible step into the smallest possible atom, and verify each precursor on its own.** Moving DNS control, changing the origin, and decommissioning the old host are three separate events. Doing them as one heroic Tuesday means a rollback has to undo all three.
- **Run the gate against the artifact you will actually ship.** Not the branch, not staging, not "it's basically the same build". This is the single most transferable line in this write-up.
- **Every visual outlier gets a written explanation or a fix.** There is no third bucket. The moment "known difference" becomes a shrug instead of a sentence, the gate stops working.
- **Structural comparison misses attribute-level loss.** Block sequences matched perfectly while 110 buttons were gone. Any comparison you build has a blind spot; the useful question is which one, and what second check covers it.
- **Verification tools are read-only, and separate from the tools that write.** A read path that shares a binary with a write path will eventually be run in the wrong mode by a tired person at 23:00.
- **The old system does not get patched after cutover.** Nobody maintains a CMS they have replaced. If the old origin stays reachable at all, it needs a firewall in front of it, not good intentions.

---

## What I'm not publishing

- The domain, the hosting providers, the agencies involved, and any record values
- Repository paths, script internals, and the runbook itself
- Traffic, conversion, and commercial figures

---

**Verification built and green:** July–August 2026. **Cutover scheduled:** 26 August 2026 — not executed at the time this was written. I will update the status line rather than quietly rewrite the tense.
