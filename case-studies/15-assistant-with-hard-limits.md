# An AI Assistant With Its Limits in Code, Not in the Prompt

> **In business terms**
>
> - **Problem:** People in a few group chats ask questions while I am busy. An assistant that answers on its own saves time. An assistant that answers on its own can also quote a price, leak a password or speak for me.
> - **What changed:** A WhatsApp assistant, Cato, that replies by itself only in allowlisted chats. Every limit that matters is a check in code. The prompt describes the limits too, but nothing depends on the model obeying it.
> - **Result:** The model can only propose a reply. Code decides if the reply goes out, in what form, from which number, and with what content removed. A failed check never means silence: the person hears that Cato cannot help, and I get a note.

Cato is my AI assistant. Since mid-October it answers in five group chats without asking me first, from a family chat to small dev chats. This case is about the part that makes that acceptable. It is not about the answers. It is about what the code refuses to let through.

**Status:** First build 15 October 2026, in dry-run. Live the same evening. Fix sessions added 25 October 2026. Code measured on 28 October 2026, as it is on disk, including edits not yet committed. The main script is 1,119 lines. The test file has 91 tests.

---

## Why the prompt is not the safety layer

A prompt is a request. The model follows it most of the time. "Most of the time" is fine for tone. It is not fine for "never quote a price" in a chat where a customer could read along.

So I split the work. The model does one thing: it reads the message and returns a label, a draft reply and a one-line reason. Everything after that is ordinary code. The code does not ask the model if the reply is safe. It checks the reply itself.

## Two switches before anything is sent

Dry-run is the default. A reply only goes out when two independent switches are both on.

- The config file of the assistant must say it is live.
- My own global rules file must contain a line that names the script as a standing exception to my approval rule.

If either one is missing, every reply goes to an audit log and nowhere else. A command-line flag forces dry-run whatever the config says. Replaying old messages for a test is always dry-run.

The second switch is deliberate. The rule that allows Cato to reply without my approval lives in the same file as the rule that forbids it everywhere else. If I delete the exception from my rules, the code stops sending. The policy and the behaviour cannot drift apart.

The fix sessions further down have a third switch of the same kind. They need their own line in that rules file as well.

## Who it listens to

Cato only reads chats on an allowlist in its config. In most of them it answers only when a message names it, or replies to Cato within a set number of hours after its last post. Two chats are marked "always on": every message reaches the model, and the model may also choose to ignore it.

Before the model sees anything, a family filter runs. It drops messages from family members, except in the one chat where I allowed them on purpose. Cato also skips its own messages and any text that carries its signature. That last check exists because of a real incident on the first day, described below.

Two more limits run in code. A chat gets at most six replies per hour. A pause file, or a fixed stop command from my own number, stops all replies at once.

## The hard stops

After the model returns a draft, three pattern checks run on both the incoming message and the draft.

- **Banned topics:** a currency sign, an amount with a currency, or words for price, discount, invoice, payment, password or bank account number. Dutch, French and English. "Grand Prix" is excluded, because a racing chat says it often.
- **Private topics:** credentials, secrets, finances, bank details and similar words.
- **Secret-like strings:** text that looks like a key or a token.

Any hit turns the reply into a handover. The model's own text is thrown away. The person gets a fixed sentence: Cato cannot do this here, and Matthias has it. I get a note with the reason. An unknown label or an empty reply also becomes a handover.

The patterns are crude on purpose. They block some harmless messages. A question about a login screen in a dev chat can trip the password check. I accept that. A false stop costs me one reply by hand. A missed stop could cost much more.

The greeting and the signature are added by code, not written by the model. So every reply says it comes from an AI assistant, in the same words, in the language of the chat.

## It can read, it cannot act

Cato has no tool to send mail, change a calendar or edit a file. The decision call has no tools at all. It only gets a small knowledge pack per chat and the last messages of that chat.

Some chats allow a deeper look at code or documents. That runs as a separate read-only session. It can only read, search and list files, and only inside the folders that chat lists. Secret files, key files and environment files are denied by name. I tested that a read outside the listed folders is refused.

Replies go out through a local bridge for Cato's own phone number. My own number is not a route the script knows.

## The day-one incident

On the first day, before the responder was live, a reply signed as Cato went out from my own account, not from Cato's number. That broke my own rule. Nothing harmful was said. But it showed that a signature in the text proves nothing about the sender.

Two changes came from it. The responder sends only through Cato's own bridge. And Cato ignores any message that carries a Cato signature, so it cannot end up answering a copy of itself.

## Bug reports that become pull requests, not deploys

In the dev chats that allow it, a bug report from an allowed sender can start a fix session. This is the only place where the assistant changes code. The limits are again in code.

- The session works in a fresh git worktree, outside my workspace.
- The editing model can run the checks, but cannot commit, push, deploy or call GitHub. Those commands are denied.
- After it finishes, the script checks the result itself: no forbidden paths, a maximum diff size, and the type check, lint and test suite run again.
- Only then does the script commit, push a branch and open a pull request. It never merges and never deploys.
- One fix runs per repository at a time. If a chat lists several repositories and the report does not point clearly at one, no fix starts. The code does not guess.

A person still reviews and merges every change.

## What it must say

The prompt asks Cato to say plainly when it cannot do something. The code backs that up. A handover always produces a reply that says what Cato cannot do and that I have it. There is one exception. If nobody addressed Cato by name, a "cannot" in a busy group is noise, so the code stays silent there and only writes the log.

## What generalises

- **Let the model propose, let code decide.** The model returns a label and a draft. Sending, framing and blocking are plain code.
- **Tie the switch to the policy.** The code reads the same rules file where the exception is written down. Delete the rule and the behaviour stops.
- **Check the input as well as the output.** If a question is about money, the answer is a handover, even when the draft looks clean.
- **Prefer crude stops with a fallback.** A regex that blocks too much is acceptable when every block still tells the person what happened.
- **Give the assistant its own identity.** Its own number and a fixed signature added by code. A signature written by a model proves nothing.
- **Make code changes end at a pull request.** The machine can prepare a fix. A person merges it.

---

## What's NOT in this case study

- Chat names, group names, members, phone numbers and family details
- Message volume, reply counts or accuracy figures. I have not verified them.
- The exact patterns, prompts, config and code
- Model names, keys, paths and the bridge setup
- The content of any message or reply

---

**Built:** 15 October 2026 (responder) and 25 October 2026 (fix sessions). **Code measured:** 28 October 2026. **Volume and accuracy:** not measured at the time of writing. I will add a dated status line when they are, not rewrite the tense.
