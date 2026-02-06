---
layout: post
title: "The Due Diligence Checklist: a compass for engineering autonomy"
description: "A practical framework to move from 'task-completer' to intentional engineer. A personal checklist for coding with context and asking better questions in the AI era"
categories: misc
youtubeId:
image: https://github.com/user-attachments/assets/3d0f48df-5cea-4130-8dcc-81d4cf234e89
---

Over the last few years, I have been synthesizing a personal checklist for software development. It is a fusion of direct feedback from Tech Leads, observations of teams, and operational playbooks — especially from [JetRockets](https://www.linkedin.com/company/jetrockets/) — combined with my own background as a professor and academic researcher. This combination of experiences allowed me to solidify a mini framework

Originating in law and business, "Due Diligence" refers to the comprehensive appraisal of a business or situation prior to signing a contract. It is the rigorous investigation required to ensure that all parties know exactly what they are getting into, minimizing risks and surprises. In an era where AI can plan, code, and review faster than ever, our responsibility to verify increases. We must be prepared to take full ownership of the results and evaluate precisely what we are delivering.

In Software Engineering, we can apply this concept to the daily "contracts" we make: the code we ship to production and the questions we ask our colleagues. It is about applying rigor, context, and documentation to the acts of coding and asking.

To ensure that we are not just "task-completers," but intentional engineers, I propose applying this personal audit before every task.

## Phase 1: The Build Checklist (Coding with Context)

Before writing a single line of code, understand exactly what you are getting into.

#### The "Why"

[ ] Business Goal: Do I truly understand the user problem or business value this task resolves? If I can't explain why a feature exists, I am not ready to code it.

#### Software Archaeology

[ ] Pattern Matching: Have I searched for similar features already implemented in the project? Reusing existing patterns ensures consistency.

[ ] External References: If no internal pattern exists, did I research reliable external sources and adapt them to our specific context?

[ ] Architectural Decisions: Have I studied why the current code is written this way? I need to understand the legacy choices before I can introduce new ones.

#### The Blueprint

[ ] Integration: How will the solution plug into the existing project?

[ ] Harmony: Will the approach clash with the codebase’s style? Most of the time goal is for the code to look like it has always belonged there.

💡 The Innovation Caveat: Sometimes, the goal is to break the pattern. Companies hire engineers to think and act, not merely to reproduce—that is how technical diversity and evolution happen. However, swimming against the current requires stronger muscles. If you deviate from the established architecture, your "Due Diligence" must be doubled. Be prepared to defend why the new approach is necessary and superior.

#### The Self-Audit

[ ] Goal Check: Does the solution actually solve the initial problem?

[ ] Quality Control: Is the code clean, documented, and compliant with project standards?

[ ] Monitoring & Safety: How will I detect possible problems in production? Do I have a plan to revert changes if necessary?

## Phase 2: The "Unstuck" Checklist (Asking with Purpose)

Sometimes, despite our best efforts, we hit a wall. Let's reflect a little before reaching out for help.

#### The Solo Mission (Before Asking)

[ ] Consult the Documentation: Have I actually read the manual and READMEs?

[ ] The External Search: Have I checked Stack Overflow or forums for similar errors?

[ ] The Git History: Have I run a git blame to see if a recent change caused the issue?

#### The Evidence Package (Formulating the Question)

When reaching out to a peer, you shouldn't just ask for the solution; you must present your findings.

Contextualize: "It’s not working" is not a question. I must explain the specific gap between expectation and reality. Document the Journey:

- "I tried X, hoping for Y, but got Z."
- "I looked at the docs for library A, but they seem outdated."

By documenting the effort, we prove respect for the team's time. You are not asking them to do the work; you are asking them to unblock the specific obstacle you couldn't clear yourself.

## The Result: A Compass, Not a Cage

In the reality of our daily work—filled with calls, tight deadlines, and urgent bug fixes—I don't religiously check every box for every single commit. That would be paralyzing.

Instead, I keep this checklist within eyesight. It serves as my anchor.

When everything is flowing, I might skip a step. But when I feel stuck, when the solution isn't clear, or when I simply run out of ideas, I return to these lists. They force me to pause, breathe, and realign with the process. The idea follows a simple principle: autonomy isn't about being perfect all the time; it's about knowing where to look when you get lost.

<img width="1024" height="683" alt="image" src="https://github.com/user-attachments/assets/3d0f48df-5cea-4130-8dcc-81d4cf234e89" />

Image: "Compass" by Johannes Ko. [Open Verse, Creative Commons 2.0](https://openverse.org/image/6dbecb6a-a18a-4f0e-ad33-b8d13da98753)
