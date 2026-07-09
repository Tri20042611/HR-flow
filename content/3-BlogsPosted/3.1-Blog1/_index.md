---
title: "Blog 1"
date: 2026-07-06
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# I've Been Using Kiro for 2 Weeks — Let Me Tell You About It

If you're already using Cursor or Copilot, come sit down — there's a new kid in town that I think is worth talking about. It's called **Kiro**, made by AWS. No sponsored post, just my genuine take because I found it genuinely different from the other AI coding tools out there.

## What even is Kiro?

Put simply: Cursor or Copilot is "chat with AI, get code." Kiro is different — it makes you write the script first, like planning a wedding: no spec, no work.

Honestly, my first reaction was: "It's just a small demo, why do I have to write all this paperwork?" But after a week of using it, I got it — writing the spec first and then building saves a lot of breakage. Back when I used Cursor, I'd code for half a day before realizing "oh, I picked the wrong database" — then I'd have to rip everything apart. Kiro pushes that pain earlier, into the spec phase, when fixing it costs way less.

## How the workflow actually works

**Step 1:** I type "Build me a Google login feature" — in Vietnamese if I want, no problem.

**Step 2:** It generates a `requirements.md` file — listing what users need to do, what conditions count as passing. Reading it, I suddenly see where I was being vague.

**Step 3:** It generates `design.md` — architecture diagrams, which database, which APIs, how to handle errors. I read it, edit, approve.

**Step 4:** It breaks things down into `tasks.md` — each task is one thing, with markers for what's required vs optional so I can pick and choose.

**Step 5:** It codes each task one by one, ticking off what's done. Commits automatically when done.

What I love: edit the spec and it updates the code automatically. Last time I added two lines to `requirements.md` saying "oh right, we need a tagging feature" — 5 minutes later it had updated the design, tasks, and code. I didn't even have time to take a sip of water.

## Cool stuff I've been using

**Steering files** — this is what I wished I had when using Cursor. You write a markdown file that sets the rules for AI to follow, and it remembers across every session. Like: "Lambda must have timeout under 30s, S3 must have versioning on, UI in Vietnamese but code identifiers in English" — it remembers everything, no repeat reminders. You attach the rule file to specific files, unrelated files don't load it, so no quota waste.

**Hooks** — you set up triggers like "on save TypeScript file, auto-run eslint," "on save template.yaml, auto-run sam validate." It's like having a senior dev review your code the moment you save. Bonus: hooks don't count against your quota — small checks run free.

**MCP** — plug in MCP servers and Kiro can call real AWS. I often ask "what's wrong with this Lambda" — it digs into CloudWatch logs and pulls the output for me. Or "which S3 buckets are public" — it calls `s3:GetBucketPublicAccessBlock` and returns results. No tab-switching to the console, saves brain cells.

## The not-so-great parts

Free tier is 50 chat messages per month — fun to play with, but not enough for a real project. I'm currently on the Pro trial at $20.

It's slower than Cursor for small quick tasks. If you need to whip up a 30-minute demo for your boss, chat mode Cursor is still faster.

You get used to it and it feels great, but it's still young — sometimes it blanks. If I edit code manually and forget to tell it to update the spec, two days later it codes right on top of my changes — gotta remember to refresh.

## Who is this for?

My take:

- **Backend devs on AWS, IaC, multi-service projects** — Kiro was made for you. Spec + steering + hooks is a really solid combo (personal opinion, I'm a newbie in this space too).
- **Quick demos for friends, one-session MVP** — stick with Cursor, don't force Kiro.
- **Teams that want everyone following the same coding style and security standards** — Kiro's steering files solve this, instead of pasting rules in README that everyone ignores.

## How to get started?

Download from kiro.dev, create a `.kiro/steering/` folder, drop in a few markdown rule files, create a `.kiro/hooks/` folder, drop in a few JSON trigger files — done. 10 minutes and you're running.

Still on the fence? Start with the free tier. 50 messages is enough to decide if it's worth upgrading.

## Wrapping up

I'm not saying Kiro kills Cursor or anything because they're different tools. I'm just saying: if you're doing serious AWS work and still using AI as "chat then paste code," you're kind of missing out. Kiro changes how you work with AI — from "ask and receive" to "plan together then build."

## Side note: how I met Kiro

I actually discovered Kiro through a friend, Anh Nghị, on a Saturday evening — that was my first time hearing about it and learning how to use it. Back then I was a complete newbie who thought AWS was mostly about clicking around on the web. It was actually Kiro that showed me a whole new world: "SAM." Until next time, folks.

![Kiro dashboard with three modes: Spec, Vibe, Agent](/images/kiro-dashboard.png)
