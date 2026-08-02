---
title: "How I actually build with Claude: design big, then walk away"
slug: "design-big-delegate-walk-away"
date: 2026-08-02
draft: false
tags: ["claude-code", "ai-agents", "workflow"]
threadX: "https://x.com/ubermuda/status/2083934358407110955"
---

The most useful shift in how I work with Claude Code wasn't a prompt trick. It was starting to actually trust it, the way you trust someone you delegate to, instead of treating it like a fancy autocomplete.

Which moved where the work happens. It's not in the typing anymore. It's in the brief.

## Design first, and design it properly

I spend the real effort up front, with Claude, before any code exists. For a big feature that's an actual design conversation: what we're building, the approach, the edge cases, the parts I actually care about getting right. I'll go back and forth until the plan is something I'd be comfortable handing to a competent coworker. Sometimes I'll line up several smaller features in one sitting the same way.

That conversation is where I find the stuff I hadn't thought about: the edge case that changes the whole approach, the assumption that doesn't hold. Half the value is being forced to say out loud what I actually want.

The other half is saying no. It throws far more ideas at me than I'd have generated alone, and most of them I reject, but rejecting them is how the vision gets sharp. I end up with a clearer picture of what I want precisely because I've been made to rule out a bunch of stuff I didn't.

## Then I leave

Once the design is solid, I hand it off and go do something else. That's the whole point of getting the design right: I don't have to hover. I'm not pair-programming. I'm a manager who wrote a clear plan and went to lunch.

## Why the design step is the whole game

The failure mode I see everywhere is someone typing "build feature X," then watching it go sideways and correcting it in real time. That's worse than doing it yourself. The upfront design is what lets you walk away; skip it and you're babysitting.

So when people ask how I get Claude to build big things unsupervised, I don't have a prompt to hand them. Three things do the work. I did the thinking first, together with it. I accept that what comes back will deviate from the plan somewhat, because expecting a perfect match is how you end up hovering again. And I review the deliverables, which is where I catch the deviation that actually matters and correct it.

That middle one is the part people skip. Unsupervised work never comes back exactly the way you wrote it down. You decide up front how much drift you can live with, and you keep a step at the end to catch the rest.

## The thing I'm experimenting with

Lately I've been tinkering with running a few of these at once, in separate git worktrees, so multiple features can build in parallel while I'm away. It also works inside a single plan, parallelizing tasks that don't depend on each other rather than whole features.

It's still rough, though not in the way I expected. The harness does bind an agent's writes to its worktree, so they can't just wander off and edit whatever they like. What bites is quieter. A plain subagent inherits whatever worktree the parent session is currently in, not the one you launched it for, so its work either gets rejected or lands in the wrong tree. And a tool that does its own file access outside the harness ignores the binding completely. Mine is bound to the main checkout, so it'll cheerfully write there from inside a worktree, leaving my branch untouched and the main tree dirty. On top of that, a worktree only counts as independent once it has its own isolated resources: its own database, its own URL, all of it.

What I think fixes it is Claude Code's worktree hooks. `WorktreeCreate` and `WorktreeRemove` would cover the lifecycle: a worktree an agent creates isn't a working app until something provisions it, and nothing tears it down afterwards, so right now that part is all manual. A `PreToolUse` hook would help with the bypass, since the harness already blocks a normal out-of-worktree edit and what's left is the tools that go around it.

## The part that's still broken

Reviewing. And it's broken at both ends.

Reviewing an AI-generated plan is already a problem on its own, before any code exists. Then it goes and does the work, and now I have to review that too. Either way I'm staring at something long (a 2000-line plan, or the diff that came out of it) and my real options are "read every line" or "looks good, ship it." The better my brief was, the more I trust it, and the *less* I actually check. Which is a great way to walk something bad straight into main.

## So, the loop

Design big with Claude, delegate, go live my life, come back and review. The parallel-worktree thing is an experiment I don't fully trust yet. The review step is the open problem. That's roughly where I'm at.

If you delegate to Claude the same way, how do you handle the coming-back part (the review)? That's the piece still costing me the most.
