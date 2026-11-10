# Causal-consistency post-pre-training objective

> **Status: untested.** No experiment has been run. Start with [`hand-off.md`](hand-off.md).

A training stage that runs after pre-training and asks a language model to agree with itself across documents.

"Why" is the inverse of "therefore". Pick a line from the pre-training corpus, ask the model "Why?", then give its answer back, ask "Therefore?", and train the model to recover the original line against a distractor. The loss is plain next-token prediction.

- [`hand-off.md`](hand-off.md): the idea, the possible errata and the experiment plan
- [`skill/why-therefore/SKILL.md`](skill/why-therefore/SKILL.md): an in-context form of the same cycle, as a Claude skill (also untested)
- Blog post: https://ad-absurdum.me/blog/causal-consistency-training
- Slides (2026): https://docs.google.com/presentation/d/1gSwNj3VmGG-11UbirovTkRNc0zRZEkNEETYd88vZfYs/edit?usp=sharing
- Slides (original): https://docs.google.com/presentation/d/1QZony2o_EFzHmStWyfw4Ivyu5-OQvMuOY8ullP1i52s/edit?usp=sharing
