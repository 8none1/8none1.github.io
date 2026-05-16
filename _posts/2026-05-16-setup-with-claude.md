---
layout: post
title: "Writing setup docs for an AI: a pattern for small, complex projects"
date: 2026-05-16
tags: [ai, claude, claude-code, github, software-engineering, open-source, setup]
social-share: true
excerpt: "I built a small podcast publisher this weekend, hit the usual nine-step README problem, and ended up doing something different — writing a setup playbook addressed to an AI assistant instead of to a human. Here's why I think this is going to be the right shape of documentation for a lot of hobby projects on GitHub."
---

## TL;DR

I built a small personal-scale tool ([Bookcast](https://github.com/8none1/bookcast)) — drop audiobooks into a Google Drive folder, get RSS feeds you can subscribe to in any podcast app. The setup involves a Google Cloud project, an OAuth client, the Apps Script editor, and `clasp`. That's not hard, but it's the kind of nine-step process where every step has a "unless your account is in Advanced Protection, then do X instead" branch.

My first instinct was to write a long README. I did, and it works. But while I was writing the README I noticed something: I'd already done the install once *with* Claude Code, and that experience had been roughly 50× nicer than any README would be. So I wrote a second file — `SETUP_WITH_CLAUDE.md` — addressed not to a human reader, but to an AI assistant. The human reader just types "Claude, walk me through `SETUP_WITH_CLAUDE.md`" and lets the AI run the install for them.

This is, I think, the right shape of documentation for a lot of hobby projects on GitHub. Here's the thinking.

## What I was building

[Bookcast](https://github.com/8none1/bookcast) is a serverless audiobook publisher. The user drops MP3s into a Google Drive folder, structured one-subfolder-per-book. A single Google Apps Script web app reads that folder on demand, generates an HTML index page with a card per book (cover art, title, author, Subscribe button), and serves per-book RSS feeds that any podcast app — Overcast, Pocket Casts, Castro, AntennaPod — can subscribe to.

There is no server to maintain, no database, no GitHub Actions runner. Everything runs inside Google's infrastructure. The whole codebase is a few hundred lines of JavaScript executed lazily on each request. The security model is URL obscurity: the deployment URL is a long unguessable string, individual file URLs are also unguessable, and the Drive folder itself is never publicly shared. I share the URL with my partner and a few family members and that's it.

The full design rationale is in the repo's [DESIGN.md](https://github.com/8none1/bookcast/blob/main/DESIGN.md). For this post the relevant bit is just: *the code is small but the install isn't*.

## The install nightmare

To get a fresh Bookcast running on a new machine, here's what has to happen:

1. Pick a Google account
2. Create a Drive folder, drop in audiobooks structured a particular way.
3. Create a new Google Cloud project.
4. Enable the Apps Script API and Drive API on that project.
5. Toggle the user-level Apps Script API at `script.google.com/home/usersettings`.
6. If your account is in Google's Advanced Protection Program, create your own OAuth client; otherwise skip ahead.
7. Install `clasp`, log in to it (possibly with the custom OAuth client).
8. `clasp create-script`, `clasp push` (with `--force` because clasp 3.x prompts interactively on manifest changes, which silently aborts in non-interactive shells).
9. Create a gitignored `Local.js` file with your Drive folder ID, run a `setupOnce()` function from the Apps Script editor.
10. Run a `runDiagnostics()` to confirm Drive access works.
11. `clasp deploy` and save the deployment URL somewhere private.
12. Optionally run `prewarmAll()` to absorb the one-time per-chapter `setSharing` cost.

Plus a dozen small "by the way" decisions: which licence, what to name your project, whether to also push to a personal GitHub remote, what to do when a particular `gh` flag isn't recognised by your version of the CLI, why the *first* feed fetch takes 30 seconds and the *second* takes 1.4 seconds.

It's not difficult, it's just *bitty*. Every step has a branch. Every branch has its own failure modes. And the failure messages from the various tools involved are some of the least helpful in the business ("Invalid container file type", "Skipping push.", "User has not enabled the Apps Script API" — when the API toggle is in a totally different place from where you'd expect).

This is the actual shape of personal-scale software in 2026. There's almost always a cloud component, an auth flow, a CLI tool, and a configuration step that varies per user. The "I cloned the repo and ran `make`" era was a beautiful brief moment between "compile your own kernel" and "configure six SaaS products to talk to each other".

## What I was going to do before

A long README. I actually wrote one — it's in the repo at [README.md](https://github.com/8none1/bookcast/blob/main/README.md) and weighs in at about 1,800 words. It walks through every step above with shell snippets and screenshots.

It's fine. It would also be a slog for anyone who isn't me to actually execute. Specifically, it has all the README anti-patterns I've experienced as an *installer* of other people's projects:

- **Branching is hard for a reader to navigate.** "If you have gcloud, do X; if not, do Y." Easy to write, easy to miss while skimming, infuriating when you realise three steps later that you took the wrong branch.
- **Copy-paste fails silently.** A `clasp push` that "Skipped push" because of a manifest prompt looks like success to a reader who isn't looking for it. They get a confusing error five steps later when nothing has actually deployed.
- **The bits that need interactive UI are hardest to document.** "Now go to the Apps Script editor, open the Local.gs file in the sidebar, select setupOnce from the function dropdown, click Run, and approve the auth prompt." That's three sentences to describe a thing that takes about 90 seconds to *do* — and if the user clicks the wrong menu, they're stuck.
- **Failure recovery isn't documented because you can't anticipate every failure.** Real installs hit weird stuff: a stale `~/.clasprc.json` from a previous project, a `gh` version that doesn't take a flag the README assumes, a Google account that's in Advanced Protection. The README either becomes a wiki of every possible failure or it leaves the user to figure those out themselves.

Setup scripts (the `curl https://... | bash` pattern) solve some of this but introduce their own problems: brittleness to environment changes, inability to handle interactive flows like OAuth, and a small security disaster waiting to happen because users have learned to pipe untrusted scripts into a shell.

## The lightbulb moment

I'd already done the install once, working with Claude Code. The experience was *night and day* compared to either a README or a setup script. Claude:

- Checked which Google account `gcloud` was authenticated as before doing anything Drive-related.
- Created the Cloud project, enabled APIs, set the ADC quota project — all without me typing the commands.
- Hit Advanced Protection on my account, nearly recognised what was happening, suggested the OAuth-client workaround, and walked me through the GCP console steps that *can't* be automated.
- Spotted the `clasp create-script --type webapp` failure (clasp 3.x rejects `webapp`; needs `standalone`), worked out the new flag, retried.
- Caught the `clasp push` skipping silently (manifest changed, non-interactive prompt), added `--force`, retried.
- Caught the `XmlService.addNamespaceDeclaration` bug in my Feed code, fixed it, redeployed.
- Caught the `/a/whizzy.org/` URL form Apps Script injects for Workspace-domain users, stripped it in code, redeployed in place to preserve the URL I'd already saved.
- Curl-tested every change against the deployment URL.

After the install was done, I sat looking at the README I'd started and thought: *who is this for?* The audience for written setup docs in 2026 is increasingly an AI assistant standing between the user and the install steps. The user opens Claude Code or Cursor or Copilot Workspace, points it at the repo, and says "set this up for me." Whatever document the AI reads first is the *real* installation script — and I'd been writing it for the wrong audience.

## SETUP_WITH_CLAUDE.md

So I wrote a second file: [SETUP_WITH_CLAUDE.md](https://github.com/8none1/bookcast/blob/main/SETUP_WITH_CLAUDE.md). It's addressed directly to the AI assistant. The first line says:

> **If you are a human:** ignore this file; follow [README.md](./README.md) instead.
>
> **If you are an AI assistant:** this is your playbook.

Then it tells the assistant, in order: what to verify before starting, what to ask the user, what to execute autonomously, the hard rules it must never violate (never share Drive folders, never commit certain values, never make the repo public without explicit user confirmation), the ten install steps with their happy-path commands and their known failure modes, and a clear definition of *done*.

The bit that surprised me was how much shorter it could be than a human-facing README, despite covering the same ground. The AI doesn't need screenshots — it can read the command output and adapt. It doesn't need exhaustive "you might hit this error" lists — it can read the error message and respond. It doesn't need the introductory "what is this project" preamble — that's in DESIGN.md and the AI can read that too if needed. The playbook is a set of *prompts* for the assistant, not a set of *instructions* for a human.

A user who clones the repo and types "open SETUP_WITH_CLAUDE.md and walk me through it" into Claude Code gets an install experience that adapts to their machine, their Google account, their preferences. The AI handles the branching. The user just answers questions and clicks the browser links the AI sends them.

## Why this is going to be the right shape for a lot of GitHub projects

Three reasons:

**It maps to how people are actually using AI now.** Anyone writing software in 2026 is also using an AI coding assistant. That assistant is increasingly the medium through which they encounter and install *other* people's projects, too. Writing setup docs as if the install audience is exclusively human ignores that the most common install medium for hobby projects is going to be "an AI assistant operated by the user." A `SETUP_WITH_CLAUDE.md` (or `AGENTS.md`, or whatever name wins) is a first-class citizen of that workflow.

**One document, many failure modes.** The hardest part of a README is the long tail of failures. With an AI playbook, you describe the *intent* of each step and a few common failures — the AI handles novel failures by reading the actual error and reasoning about it. You write less, and the install becomes more robust at the same time. That trade is rare.

**Interactive steps stop being a hostile UX.** OAuth flows, "go to this UI and click that button," waiting for an API toggle to propagate — these are torture to express in a README and natural to express in an AI playbook. ("Tell the user to open this URL, then wait for them to confirm before continuing.") The interactive bits, which are the *worst* part of any modern install, become some of the *easiest* parts of the AI playbook.

The pattern doesn't replace READMEs — humans without AI assistants still need them — but it sits alongside them and shifts the centre of gravity. For small, complex hobby projects on GitHub I suspect this is going to be the dominant install mode within a year or two.

## The security angle, which I think is real

The natural worry about handing an AI shell access on your machine is: how do I know what it's going to do? With curl-bash, you don't (you can `curl` and read it first, but most people don't). With a long README, you do, but you're also responsible for executing the commands yourself, and the failure modes for "human executes the wrong command" are well-documented.

`SETUP_WITH_CLAUDE.md` is interesting because it's *both* the source of truth for what the AI will do *and* a human-readable document the user can audit before letting the AI loose. Before you point Claude at it you can read it yourself in a minute and see exactly what it's intended to do: create a GCP project named like this, enable these APIs, create a `Local.js` with this shape, run these editor functions. The AI may invoke different *specific* tools to accomplish those steps, but the playbook bounds the work and the AI's own safety rails handle the rest.

That makes AI-driven setup, in principle, *more* auditable than a setup script. The setup script is opaque until you read it — and once you've read it, you have to trust that nothing in the chain has tampered with it between your read and your execution. The playbook + AI combination is auditable at read time *and* the AI gives you a running narration of what it's doing at execute time.

There are a few security primitives I encoded in Bookcast's playbook that I think are general:

- **Hard rules at the top.** "Never apply `setSharing` to a folder." "Never commit the Drive folder ID." The AI reads these once and respects them across the whole session. A human reading a README would skim the rules section.
- **Where secrets go.** The playbook explicitly says which values are sensitive and where they're permitted to live (PropertiesService at runtime; gitignored `Local.js` at rest). The AI then enforces this — it won't, e.g., add a generated `Local.js` to git.
- **What requires the user's explicit consent.** Making the GitHub repo public is irreversible-ish; the playbook says the AI must ask the user explicitly before running `gh repo edit --visibility public`. (Claude Code's built-in safety classifier *also* blocks this without explicit consent, so there are two layers of seatbelt here, which is appropriate.)

The general principle: the playbook is a security contract you write *once*, in plain English, that the AI then enforces *consistently* across the install. A human installer might forget the rule by step 8. The AI won't.

## When does this pattern fit?

Not every project benefits from this. A project where install is `npm install && npm start` doesn't need an AI playbook — the README's first three lines are sufficient. A project with no install at all (a single-file Bash script, say) similarly doesn't need one.

The pattern earns its keep when install has:

- Multi-platform setup (your machine + a cloud service + maybe a CLI tool + maybe a browser flow).
- Auth or credential bootstrap that varies per user.
- Branching paths (account type, OS, presence of supporting tools).
- Interactive steps that can't be scripted (GCP console clicks, OAuth approvals).
- Configuration the user must hand-supply (folder IDs, account emails, etc.).

That description fits an enormous slice of hobby projects on GitHub today — anything involving Google services, AWS, Azure, Discord bots, home automation, mail integrations, smart home devices, Cloudflare Workers. Pretty much any project in the modern "personal-scale software" category.

## Closing thought

I'd love to see this pattern (or some convergent version of it — `AGENTS.md`, `AI_SETUP.md`, whatever) become a standard part of small project repos. The barrier to entry for personal-scale software has been climbing for years, and most of that climb is in install complexity rather than the software itself. AI assistants are the first install technology in a long time that actually pushes that barrier *down* instead of up.

If you've shipped a small personal project recently and found yourself writing a setup README that felt longer than the project deserved, try writing the AI-facing version too. You may find — as I did — that the AI version is the one that does the most actual work.

---

*[Bookcast is on GitHub](https://github.com/8none1/bookcast) under MIT. The setup playbook is [`SETUP_WITH_CLAUDE.md`](https://github.com/8none1/bookcast/blob/main/SETUP_WITH_CLAUDE.md). If you try this pattern on one of your own projects I'd be interested to hear how it went.*
