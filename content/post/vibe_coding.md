+++
date = '2026-01-23T13:37:29-08:00'
title = 'Vide Coding with Claude Code'

image = "/post/images/claude_code.jpg"
tags = ["programming", "claude"]
categories = ["Project"]
+++


# Vibe Coding, But for Real: Building with Claude Code and Multi-Agent AI

## 1. Vibe Coding (What I Thought It Was vs What I Tried)

Before this project, I had already been using AI a lot while coding, but mostly in a pretty shallow way. It was basically a better search engine. I’d ask it how something worked, grab a small snippet, tweak it, and move on. Helpful, but not game-changing. It never really felt like part of my workflow more so like a shortcut when I got stuck.

I had also tried coding agents before. When Gemini CLI first came out and was free, I played around with it a bit. But I didn’t really put any effort into setting it up properly. I just used it straight out of the box, didn’t give it any instructions, didn’t define how I wanted it to behave, and didn’t add things like a `GEMINI.md` file. Unsurprisingly, the results were pretty underwhelming. It felt random, and I quickly wrote it off as not that useful.

At the same time, I kept seeing people do genuinely impressive things with Claude. Not just writing snippets, but actually building features, refactoring code, and managing larger changes. That got my attention. It felt like there was something I was missing, not just a better model.

So I decided to take a more intentional approach. Instead of treating the AI like a black box, I wanted to see what would happen if I actually customized it. That meant using things like Claude Code, defining behavior with `claude.md` files, assigning skills, and generally trying to treat the agent more like a developer than a tool.

The goal was to see if, with the right setup and structure, vibe coding could turn into something more capable and whether all the hype around Claude Code was actually justified.


## 2. The Project (Why I Finally Built It)

The project I used as a testbed is a **local-first personal financial dashboard**. It tracks portfolios, spending data, and historical performance, and it includes an AI assistant that can answer questions about your financial situation.

This is a project I’ve wanted to build for a long time. The reason I never did was simple. It felt like too much effort for the value it would provide. Between backend logic, frontend UI (especially this), data handling, and security concerns, it always felt like one of those ideas that’s cool in theory but not worth the time investment.

Vibe coding changed that cost benefit analysis.

With AI helping carry some of the load, this went from “probably not worth it” to “actually doable.” The dashboard itself still took work, but AI made it realistic to build something I would have otherwise kept on a todo list forever.

That’s why I chose it. It’s complex enough to expose where this approach works and where it doesn’t.


## 3. Security and Privacy Were Non-Negotiable

Security was a huge concern from day one. The idea of having API access to all of my financial accounts made me uncomfortable, especially in a project that involved heavy AI usage. Even if it’s technically possible to do this securely, it felt like adding unnecessary risk.

So I took a strict approach:
- No stored credentials
- No direct bank or brokerage APIs
- No cloud-hosted AI models

Instead, the dashboard relies on manually provided data (like CSVs) and **local-only AI** using Ollama. Everything runs on my machine, and nothing sensitive ever leaves it.

This decision shaped the entire project. It limited some convenience, but it also made the system easier to reason about and trust. If something breaks, it breaks locally. If something leaks, it’s my fault not an API key sitting somewhere I forgot about.


## 4. The AI Inside the Dashboard (Not the One Writing Code)

It’s important to separate two different uses of AI in this project.

The first is **AI helping me build the project**, which I’ll get to later.  
The second is **the AI agent that exists inside the dashboard itself**.

The dashboard includes an AI assistant that can answer questions about your financial situation. The key here is the context. The agent is given structured access to things like portfolio holdings, spending patterns, historical snapshots, and trends over time.

On top of that, it can optionally use web search to pull in general market or asset-level context. The key detail here is that the web search augments *external knowledge*, not personal data. Your financial information stays local, while public information comes from outside.

That combination turned out to be really powerful. The AI isn’t guessing or speaking in generalities it’s responding based on your actual numbers. At the same time, it isn’t sending those numbers anywhere. It’s a good example of how AI can be genuinely helpful without being invasive.

## 5. Using Claude Code to Actually Build the System

To build all of this, I relied heavily on **Claude Code**. This felt very different from using AI as a normal chat assistant. Instead of asking isolated questions, I was delegating real chunks of work.

Claude Code felt less like autocomplete and more like working with a developer who could take initiative, but still needed clear direction and review. When that direction was missing, things got messy fast. When it was clear, the results were honestly impressive.

That realization led me to experiment with a more structured, multi-agent approach.


## 6. Multi-Agent Vibe Coding with Git Branches

One of the most effective setups I used was a **multi-agent workflow** built around Git branches.

I split work across three sub-agents, each focused on a specific area like frontend changes, backend logic, or AI-related code. Each agent worked on its own branch and handled a narrow set of tasks instead of touching everything at once.

On top of that, I had a supervising agent whose job was to review changes, approve merges, or push back with feedback. That agent didn’t write much code itself it mostly acted as a reviewer.

This structure mattered more than I expected. It reduced chaos, made mistakes easier to undo, and prevented the codebase from drifting in random directions. It felt a lot closer to managing a small team than “vibe coding” in the usual sense.


## 7. CLAUDE.md Files (And Borrowing from Others)

The thing that made this setup work was `CLAUDE.md`.

These files define how Claude should behave inside a project. At first, mine was pretty barebones. Over time, I started expanding it and honestly, I didn’t do that alone. I borrowed ideas from other people’s `CLAUDE.md` files that I found on GitHub, stealing bits and pieces that reflected good coding practices, architectural discipline, and clear boundaries.

Things like:
- What parts of the codebase are off-limits
- How to think about changes vs refactors
- When to ask questions instead of guessing
- How strict to be about structure and readability

This turned out to be one of the highest-leverage things I did. The better the instructions got, the better the AI behaved. Without them, the AI felt unpredictable. With them, it felt surprisingly consistent.


## 8. Claude Code Skills and Frontend Design

One Claude Code feature that stood out was **skills**, especially the `/frontend-design` skill.

By the time I used it, my UI technically worked but looked very “vibe coded.” Everything was there, but everything looked average. Instead of rewriting logic or removing features, I constrained the skill very tightly. The instructions were basically: don’t delete anything, don’t change data, just improve layout, spacing, and visual hierarchy.

That constraint made a huge difference. The UI didn’t magically become perfect, but it went from looking hacked together to looking like something someone actually designed.

### Original UI
![Alt text](/post/images/old_ui_redacted.png)

### Refined UI After Frontend-Design Skill
![Alt text](/post/images/new_ui1_redacted.png)
![Alt text](/post/images/new_ui2_redacted.png)
![Alt text](/post/images/new_ui3_redacted.png)
![Alt text](/post/images/new_ui4_redacted.png)
![Alt text](/post/images/new_ui5_redacted.png)
![Alt text](/post/images/new_ui6_redacted.png)
![Alt text](/post/images/new_ui7.png)

NOTE: Some features were added after the `/frontend-design` UI update.


## 9. The Token Problem (It Adds Up Fast)

This approach is effective, but it burns tokens fast.

Running multiple agents, keeping large context windows, and iterating often adds up quickly. I definitely noticed how fast usage climbed, especially when agents were working in parallel or reviewing large diffs. I could burn through all my tokens in 5 minutes if I wan't careful. 

Even so, the tradeoff felt worth it. The speed, the learning, and the quality of the end result outweighed the cost for me but it’s not something to ignore. This style of development is powerful, but it isn’t free.


## 10. Closing Thoughts

This project made one thing very clear to me: the biggest limitation in AI-assisted development isn’t the model. It’s how well you can structure work, give instructions, and review output.

Vibe coding can work, but only when it’s paired with clear boundaries, good documentation, and thoughtful review.