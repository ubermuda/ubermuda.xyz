---
title: "Swiss cheese: five checks that fail differently"
slug: "swiss-cheese-testing"
date: 2026-08-26
draft: false
tags: ["ai-agents", "testing", "guardrails", "workflow"]
threadX: "https://x.com/ubermuda/status/2092710021570170883"
threadBluesky: "https://bsky.app/profile/ubermuda.xyz/post/3mtz66td3bc2e"
---

I mostly don't read the implementation anymore. The design is where the work happens, and if the brief was good the code that comes back is usually fine.

But "usually fine" isn't "always fine." Something has to catch the rest. Five things do. They cover different parts of the job, and every one of them misses something.

## The swiss cheese model

The model comes from accident analysis. Picture every safeguard in a system as a slice of cheese. Each slice has holes, so each one lets something through on its own. Stack a few slices and the holes rarely line up, so a failure now has to find a path straight through all of them to reach the end. Planes stay in the sky on that principle: not one perfect safeguard, but several imperfect ones whose gaps sit in different places.

The useful part is what it tells you to stop doing. You stop trying to make any single layer airtight, because you can't, and you start asking a different question about each check you add: does this fail the same way as something I already have? Two checks with the same blind spot are one check and some wasted minutes.

Here's the stack, and what each slice misses.

## Slice one: the static gate

phpstan, php-cs-fixer, phparkitect, and a pile of my own rules. Runs in seconds. Catches types, layering, dead code, convention drift, and a decent number of real bugs. A well-typed codebase says a lot about what it's meant to do, and these tools hold it to that.

What they can't do is tell me whether what it's meant to do is the right thing. A function can be perfectly typed, correctly placed, immaculately formatted, and compute the wrong output, and nothing in this layer has an opinion about it. The tools aren't broken; that's simply the category they work in.

## Slice two: unit tests

Fast, database-free tests over pure logic.

Their hole is the mocks. When I stub a collaborator, I'm writing down what I believe that collaborator does, then testing against my belief. If the real one behaves differently, the stub doesn't know and the test goes green. Unit tests verify units. They're structurally incapable of checking the seams between them, because the seams are precisely what got replaced.

Fine, as long as I remember it. The trap is staring at the coverage number and feeling safe.

## Slice three: integration tests

Tests that boot the whole kernel and talk to Postgres. Routing, Doctrine, container wiring, all of it live. Each one runs inside a transaction that gets rolled back afterwards, so they're isolated without anybody writing schema-reset code.

This is where the seams get checked. A stub can lie about what a repository returns. A Postgres query can't.

The hole: no JavaScript runs. Ever. These tests fetch server-rendered HTML and inspect it. Every Stimulus controller, every Turbo navigation, every behaviour that only exists once a browser has parsed the page is invisible to them. I can have a wall of green integration tests over a page that's comprehensively broken for anyone actually looking at it.

## Slice four: end to end

Playwright specs driving a browser against the full stack.

This is the only layer that sees what a user sees. It's also the most expensive: slow to run, fiddly to write, and the bill comes due on every single run.

So its hole is coverage, and I dug that one on purpose. End-to-end covers the paths where being broken would be worst. It doesn't cover everything and it never will, because a suite thorough enough to cover everything would take long enough that I'd stop running it. A check you skip has a hole the size of the check.

## Slice five: me

I read plans closely and diffs selectively. My hole is the obvious one: I'm not reading the implementation, which is the whole point of the exercise. I'm also inconsistent, and worse on a Friday.

The layers below exist because I'm the least reliable slice in the stack, not the most.

## When the holes line up

They do, occasionally.

There's a note in my tracker about a run where the entire end-to-end suite passed against the wrong checkout. The command that serves the app resolves to the main checkout, so a run started from a git worktree exercised main's code and reported green while gating none of the branch. I caught it because the branch happened to add visible UI and a test went looking for it. A branch that changed behaviour without changing markup would have sailed through.

That's a hole running all the way down, because the layer with the best view was aimed at a different codebase. A layered stack makes that rare. It doesn't make it impossible, and pretending otherwise is how you end up back on one slice.

## What the stack is for

Every one of these predates me delegating implementation. None of them mattered as much then, because I used to be a layer with serious coverage. I read the code. Now I don't, and the stack absorbed the work I stopped doing.

Which is the honest version of "let it cook." It isn't trust. It's five checks with uncorrelated blind spots, and a habit of asking which hole let the last one through.
