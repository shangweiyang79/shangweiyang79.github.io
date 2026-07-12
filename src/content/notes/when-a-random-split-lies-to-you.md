---
title: When a random split lies to you
date: 2026-06-15
tags: [evaluation, data-leakage, nlp]
summary: My news classifier scored 0.81 on a random split and 0.75 on a chronological one. The second number is the true one — here is how I found the leak.
---

My first DistilBERT run on TaiwanPlus news titles reported 0.81 accuracy. It felt too good for a nine-class problem with short, ambiguous inputs — and it was.

## The leak

News is not i.i.d. A developing story produces a burst of near-duplicate items over a few days: same entities, same phrasing, same label.[^1] A random train/test split scatters that burst across both sides, so at test time the model is often "predicting" a headline whose sibling it already memorized during training. That is not classification skill; it is a lookup table wearing a model's clothes.

[^1]: Example from the dataset: a naval transit through the Taiwan Strait generated eleven near-identical titles in one week, all labeled *Military*.

## The fix

I rebuilt the splits chronologically — train on the past, validate and test strictly on the future, which is exactly how the model would be used in production. Accuracy dropped from 0.81 to about 0.75.[^2]

[^2]: The drop is the point. If your offline metric cannot survive a deployment-shaped split, it was never measuring deployment performance.

Nothing about the model got worse. The evaluation got honest.

## What I keep from this

- **Match the split to the deployment.** If the model will classify tomorrow's news, the test set must be tomorrow-shaped.
- **Be suspicious of numbers that feel easy.** The 6-point gap was invisible until I went looking for the reason the task felt too solvable.
- **Report the honest number.** 0.75 with a chronological split earned more trust from my newsroom stakeholders than 0.81 ever could — and it set the real baseline that the later label-cleaning and cascade work improved on.
