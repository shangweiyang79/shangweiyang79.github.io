---
title: Cleaning labels beat tuning the model
date: 2026-06-28
tags: [label-noise, llm-assisted, distilbert]
summary: Weeks of hyperparameter tuning bought me almost nothing. Re-examining the training labels with Gemini bought +2.2 points. The data was the model.
---

After fixing my evaluation split, I did what everyone does next: tuned. Learning rates, weighted loss, label smoothing, longer training. The gains were fractions of a point, and I kept staring at the confusion matrix wondering why *Politics* and *International Relations* bled into each other.

Then I read the misclassified rows instead of the metrics — and many of the "errors" were correct. The model was right; the label was wrong.[^1]

[^1]: The source labels came from upstream editorial metadata that was never designed to be a training target. Treating operational metadata as ground truth is a quiet, common failure.

## LLM-assisted label audit

I used Gemini as a second annotator: for each training row, it re-classified the title and tags against the same nine-category schema, and rows where it disagreed with the source label were flagged and re-examined. Corrected labels went back into training.

The result was bigger than any tuning run: DistilBERT accuracy moved from **0.731 to 0.753** (+2.2 points) with zero architecture changes.[^2]

[^2]: On the human-verified test set the effect compounds — clean training labels also made the confidence scores better calibrated, which is what makes the cascade's τ = 0.80 gate work at all.

## The uncomfortable corollary

When I scored Gemini itself against a human-verified test set, it reached 0.81 — beating my fine-tuned model. Against the *noisy* source labels it had looked worse. The benchmark had been penalizing the better model for disagreeing with wrong answers.

## What I keep from this

- **Read your errors before tuning.** Ten minutes of looking at rows told me what three weeks of sweeps could not.
- **Labels are data, and data drifts in quality.** An audit pass is a legitimate, high-leverage modeling step.
- **Never benchmark against labels you haven't verified.** Otherwise your leaderboard ranks conformity to noise, not correctness.
