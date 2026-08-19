---
title: "Vibecode on a stack you understand"
slug: "vibecode-a-stack-you-understand"
date: 2026-08-19
draft: false
tags: ["ai-agents", "vibecoding", "stack", "workflow"]
threadX: "https://x.com/ubermuda/status/2090082088951087110"
threadBluesky: "https://bsky.app/profile/ubermuda.xyz/post/3mtgwofvkh22z"
---

The most useful thing I learned building with AI this year is slightly counterintuitive: vibecode on a stack you understand. Not the one the model supposedly writes best. The one *you* know cold.

I got there the annoying way, by trying the alternatives first.

## What I tried

Full vibecode from scratch with Laravel or Symfony. PHP, worked fine.

Full vibecode from scratch with Node. This one degraded. The code got worse the further in I went, each round took longer than the one before it, and I could never quite tell whether what came back was good or just plausible. At some point it stopped being sustainable and I dropped it.

Eventually I settled on PHP + Symfony, a stack I know properly, and built my own template on top of it.

## It's not a language thing

You might get the impression I'm trashing Node here. I'm not. Plenty of people ship excellent things in Node. The difference wasn't the language. It was me.

I know Symfony well. I know what good looks like in it. So when I brief an agent I can be precise, and when it hands something back I can tell in a few seconds whether it's right or confidently wrong. In Node from scratch I had neither of those. I couldn't write a sharp brief, and I couldn't grade the output. Which meant I was reduced to babysitting and hoping, which is the worst way to work with an agent.

## You can only delegate what you can evaluate

This is the whole thing, so I'll say it plainly: delegation only works if you can check the work. The moment you can't tell good output from plausible garbage, the AI stops being leverage and turns into a very fast way to generate code you don't understand.

Your judgment is the bottleneck, not the model's training data. Picking the stack the model "writes best" optimizes the wrong side of that equation. Pick the stack where *your* judgment is sharpest, and the model immediately gets more useful, because now you can steer it and catch it when it's wrong.

## So I picked what I know

PHP + Symfony. Boring, unfashionable, and I can read its output at a glance. That last part is worth more than the model being marginally more fluent in something trendier. I would rather have a slightly less "AI-native" stack that I can review in my sleep than a fashionable one where I'm flying blind.

Then I built my own template on top, a skeleton that encodes how I work, so every project starts from my standards instead of a blank page.

## The takeaway

If you're choosing a stack to vibecode in, don't pick the one the internet says the AI is best at. Pick the one you can review in your sleep. The bottleneck is you, and that's good news, because it's the part you can make better.
