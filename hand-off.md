# Hand-off: 10 November 2026

For the person (or agent) who starts the experiments in this repository.

## 1. Read this first

**Nothing is tested.** The idea is from 2021. It exists as two slide decks, one blog post and one Claude skill. This repository starts empty, apart from this file, the README and a copy of the skill.

The one piece of "evidence" is two playground examples from the original deck. Both can be position bias (section 3, item 1). Do not build on them before experiment E0.

## 2. The idea in ten lines

- An LLM is trained on next-token prediction. That rewards consistency inside one context window. It does not connect what the model says in different documents.
- "Why" is the inverse of "therefore". "I went to a donut shop" -why-> "I wanted to eat donuts" -why-> "I was hungry", and back with "therefore".
- **Objective (a stage after pre-training):**
  1. Pick a line from the pre-training corpus.
  2. Ask the model "Why?" and record the answer (abduction).
  3. Give the answer, ask "Therefore?", and offer the original line and a distractor line.
  4. Train with next-token loss on the original line.
- The cycle also runs in the other direction: therefore, then why, then therefore.
- It is "post-pre-training" because step 2 needs a model that can already answer "why", and because the stage uses no task data and no labels.
- **2026 revision (proposals):** distractor from the same document, cycle loss plus the standard language-model loss as an anchor, a consistency filter on the abduced causes, chains of "why" steps to a root cause.
- **In-context form:** the same cycle on one draft answer, with no change to the weights. See `skill/why-therefore/SKILL.md`.

## 3. Possible errata (from a review in November 2026)

In the evidence:

1. **Position bias.** In both playground examples the model picked the first option in the list. That alone can explain the two failures.
2. **Sampling noise.** Two different answers about a favourite president can come from sampling at a non-zero temperature. They do not prove two beliefs.
3. **Earlier work.** ATOMIC and COMET (2019) generate "intent" and "effect" relations. Abductive NLI (2020) and round-trip consistency for question answering (2019) are also before 2021.
4. **A special case.** Repeated "why" converges in the Three Little Pigs because there is one character with one motive. In general the chains branch, loop, or do not end.

In the objective:

5. **"Why" is not a true inverse.** One effect has many causes, and one cause has many effects. A hard distractor from the same document can also be a valid effect.
6. **Lines with no cause.** A "why" for a definition or a description is a rationalisation. The model then trains on causes that it invented.
7. **Unclear loss target.** In the playground screenshots the answer is one letter. A loss on one letter gives about one bit for each example.
8. **Test mismatch.** Paraphrase consistency on SQuAD measures a property that the cycle does not train.

Also from later papers: a consistency loss with no anchor can collapse to one constant answer, and a reconstruction step can use shared words in place of the causal relation.

## 4. Experiments, in priority order

State the prediction and the stop rule before each run.

### E0. Measure the backward-step error rate correctly

This comes first. It decides if there is a signal to train on.

- **Setup:** a current open model, no training. At least 500 corpus lines. For each line: abduce a cause, then score the original line against a distractor.
- **Controls:** score each option by the log-likelihood of its full text, not by a letter. Run both option orders. Use temperature 0 for the abduction, or several samples with the spread reported.
- **Distractor types:** a random corpus line, a line from the same document, a line on the same topic from a different document.
- **Prediction:** with a random distractor and the position control, the error rate is low (a few percent). With a same-document distractor it is much higher.
- **Stop rule:** if the error rate is near zero for all three distractor types, the model is already cycle-consistent on this test and the objective has nothing to fix. If it is high only for same-document distractors, check erratum 5 by hand on 50 failures before you call them errors.

### E1. How many lines have a cause?

- **Setup:** 200 random corpus lines. Label each as "has a cause that a reader can state" or not. Two labellers, or one human and one model with agreement reported.
- **Why:** this sets the fraction of the corpus that the objective can use (erratum 6), and it gives a filter for E2 and E3.
- **Stop rule:** none. The number is an input to the design.

### E2. Toy-scale training test with a known causal graph

- **Setup:** a synthetic corpus made by full enumeration from a small causal graph. Each "document" states a few edges. Some cause-to-effect paths never appear inside one document. Pre-train a small model on the corpus, then compare:
  - A: more pre-training steps (the control for extra compute),
  - B: the cycle loss alone,
  - C: the cycle loss plus the language-model loss.
- **Measure:** exact-match accuracy on cross-document questions (paths that no single document states), on reversed questions, and on in-document questions as the accuracy guard.
- **Prediction:** C is better than A on cross-document and reversed questions, with no loss on in-document questions. B collapses or loses in-document accuracy.
- **Stop rule:** if C is not better than A at an equal number of gradient steps, the objective adds nothing at toy scale. Do not go to E3 before you understand why.
- **Host:** `C:\Users\soura\Desktop\me-gpt` is a from-scratch GPT of about 100M parameters with a workflow for this type of test (combinatorial data, exact-match checks on generated text). It has no pre-training stage today, so E2 must add one on the synthetic corpus.

### E3. Fine-tune a small pre-trained model

- **Setup:** a 1B to 3B open model with LoRA. Baseline: Deductive Closure Training (arXiv 2401.08574), which is the closest published method. Then the cycle objective with the E0 scoring, the E1 filter and same-document distractors.
- **Measure:** the Reversal Curse test set (arXiv 2309.12288), CREAK fact verification, a paraphrase-consistency set, and a general accuracy guard.
- **Stop rule:** decide the smallest gain that matters before the run. A method that is not better than Deductive Closure Training is not worth a write-up.

### E4. Test the in-context skill

- **Setup:** a set of questions whose answers rest on a chain of claims. Run each with the skill and without it.
- **Measure:** accuracy, the number of claims that the ritual changed, and the cost in tokens. Read the transcripts: check that the blind step was blind.
- **Stop rule:** if accuracy does not change and the cost at least doubles, mark the skill as not useful and say so in the gist.

## 5. Design decisions to fix before E2

- **Loss target:** the full text of the original line, conditioned on the abduced cause. Not an option letter.
- **Distractor:** where it comes from, and how to exclude a distractor that is also a valid effect.
- **Anchor weight:** the ratio of the cycle loss to the language-model loss.
- **Filter:** which abduced causes are dropped before the loss (for example a cause that reuses the words of the line).
- **Direction:** train both cycle directions, or one.

## 6. Where things are

| Item | Location |
|---|---|
| 2026 slide deck | https://docs.google.com/presentation/d/1gSwNj3VmGG-11UbirovTkRNc0zRZEkNEETYd88vZfYs/edit?usp=sharing |
| Original slide deck | https://docs.google.com/presentation/d/1QZony2o_EFzHmStWyfw4Ivyu5-OQvMuOY8ullP1i52s/edit?usp=sharing |
| Blog post | https://ad-absurdum.me/blog/causal-consistency-training |
| Skill (gist) | https://gist.github.com/Ghost---Shadow/409687db6d711a9b9a09417fe30852cc |
| Paper notes, 9 files | `C:\Users\soura\Desktop\Dump\digest\Consistency\` |

Closest published work, with what each one lacks:

| Paper | Lacks |
|---|---|
| Deductive Closure Training, arXiv 2401.08574 | Goes forward only; no inverse step |
| Reasoning to Learn from Latent Thoughts (BoLT), arXiv 2503.18866 | The cycle is generative; no distractor test |
| RevThink, arXiv 2411.19865 | Needs a teacher model and question-answer pairs |
| Cycle-Instruct, arXiv 2508.16100 | Pairs instructions with responses, not causes with effects |
| Time-Reversal (arXiv 2412.02626), LEDOM (ACL 2026) | Reverse the token order, not the meaning |
| Maieutic Prompting, arXiv 2205.11822 | Inference only; nothing is trained |

The paper notes were written mostly from automatic summaries of the papers. Check a number against the paper before you cite it.

## 7. A note on the date

The author dates the idea to 2021. One example in the original deck quotes a news line from December 2022, so that example was added later.

## 8. After E0 to E4

[`future-work.md`](future-work.md) (13 November 2026) makes the idea more general: other cycles, two-path objectives and eight more graph shapes, with a candidate dataset for each. It is also untested. Do not start it before E0.
