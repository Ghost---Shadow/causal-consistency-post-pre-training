# Future work: 13 November 2026

> **Status: untested.** Nothing in this file has been run. The dataset columns are guesses about where to look first. They are not results. Read the [disclaimer](README.md#disclaimer).

This file records a design discussion from 13 November 2026. It makes the idea of [`hand-off.md`](hand-off.md) more general. Do the experiments of `hand-off.md` (E0 to E4) first. This file is for the work after them.

## 1. What "post-pre-training" means

Post-pre-training is a training stage between pre-training and task fine-tuning.

- It is not pre-training. Each step needs a model that can already answer a prompt such as "Why?". A model with random weights cannot.
- It is not fine-tuning in the usual sense. It uses no task data and no labels. The data is the pre-training corpus.
- The training pairs come from the model's own answers. Where possible, the loss stays the usual next-token loss.

The why / therefore cycle is one objective of this type. The sections below list others.

## 2. The general principle

Why / therefore is a pair of inverse relations. But "inverse" is not the important property. The important property is that **two computations must agree**, and their disagreement is the loss. No label is necessary, because each computation supervises the other.

In why / therefore, one computation is "do nothing" (X stays X) and the other is "why, then therefore". Other graphs give other objectives. Section 5 lists ten graph shapes. Sections 3 and 4 give examples of the first two.

## 3. Shape 1, inversion: other cycles

The pattern is a forward question, a backward question that inverts it, and the original line as the target.

| Cycle | Forward | Backward | Dataset that could benefit most | Reason |
|---|---|---|---|---|
| Why / therefore | "Why?" | "Therefore?" | COPA, e-CARE | Both ask for the cause or the effect of a premise. This is the cycle itself as a test. |
| General / specific | "What is this an example of?" | "Give an example." | CommonsenseQA | Its questions come from ConceptNet, with many IsA relations. |
| Premise / conclusion | "What must be true for this?" | "What follows from that?" | EntailmentBank, ProofWriter | Multi-step deduction where each step must hold in both directions. Near to Deductive Closure Training. |
| Claim / evidence | "What evidence supports this?" | "What does this evidence show?" | FEVER | The task is to connect a claim to its evidence. |
| Negation | "State the opposite." | "State the opposite." | Negated LAMA, CondaQA | Both show that models ignore negation. |
| Relation inversion | "A is the parent of B." | "B is the child of whom?" | The reversal curse dataset (Berglund et al.) | It measures this failure directly. |
| Part / whole | "What is this a part of?" | "Name a part of it." | LAMA, ConceptNet subset | It has PartOf and HasA probes. |
| Question / answer | "What question does this line answer?" | "Answer the question." | SQuAD | It is used for question generation and for answering, so both directions have a score. CycleInstruct uses this cycle to make instruction data. |
| Before / after | "What happened before this?" | "What happened next?" | MC-TACO, ROCStories (Story Cloze) | Temporal order and the next event in a story. Before is not always a cause, so this is different from why / therefore. |
| Goal / step | "What is the goal of this step?" | "What is a step toward that goal?" | The wikiHow goal-step dataset | It has goal inference and step inference as separate tasks. |
| Problem / solution | "What problem does this solve?" | "How do you solve that problem?" | GSM8K, StrategyQA | RevThink reports gains from backward questions on this type of task. |
| Summary / expansion | "Summarise." | "Expand." | XSum | One-sentence summaries, so the middle step is short and the identity shortcut is blocked. The cycle is lossy: the target is the meaning, not the exact tokens. |
| Translation | "Translate to French." | "Translate to English." | FLORES, low-resource pairs | This is back-translation. It helps most where parallel data is small. |
| Code / description | "What does this code do?" | "Write the code." | HumanEval, MBPP | Tests give an external check of the cycle. |

Notes:

- **Strongest match:** relation inversion and the reversal curse dataset. The dataset measures the exact gap that the cycle trains, and the answer is exact, so the paraphrase shortcut is small.
- **Most general:** general / specific. Many corpus lines have no cause, but almost every line is an example of something.
- **Only external check:** code / description. A cause cannot be verified. Code can be run.
- **Weakest match:** part / whole. No benchmark known to the author is primarily about part-whole relations.
- **Old benchmarks:** COPA and Story Cloze are small, and a modern model is near the ceiling. A toy-scale model would show a difference more clearly.
- **Mixed cycles:** a longer loop (why, generalise, give an example, therefore) is harder to close with a shortcut. But the one-to-many problem of section 6 grows with each step. Test single cycles first.

## 4. Shape 2, commutative: two paths to one point

The source of this shape is question-answering evaluation of summary faithfulness (the QAGS / FEQA / QuestEval family; the exact paper that started the discussion was not identified):

1. Generate questions from the summary.
2. Answer each question with the source document.
3. Answer each question with the summary.
4. If the two answers agree, the summary is faithful.

```txt
document ──────────────── answer question ──→ answer A
document ──→ summarise ── answer question ──→ answer B
                                              A should equal B
```

Those papers use this as an evaluation. The proposal is to make it a training objective.

**How to make it trainable**

- **Teacher path and student path.** Generate answer A with the document path, with no gradient. Use A as the next-token target for the summary path. The loss stays the usual next-token loss.
- **Distribution match.** Minimise the KL divergence between the answer distributions of the two paths. This is a larger change, because it is not a next-token loss on text.
- **Gradient into the summariser.** "Write a summary from which the questions get the same answers" is a signal for faithfulness. The sampling step in the middle blocks the gradient, so this needs a policy-gradient method or a soft relaxation.

**Examples**

| Path 1 | Path 2 | What agreement trains | Dataset that could benefit most | Reason |
|---|---|---|---|---|
| Answer from the document | Answer from the summary | Faithful summaries | FRANK, the XSum faithfulness annotations | Both have human labels for factual errors in summaries. |
| Answer directly | Answer with step-by-step reasoning | Reasoning distilled into the direct answer | GSM8K | The gap between the direct answer and the reasoned answer is large there. |
| Answer the question | Answer a paraphrase of the question | Robustness to phrasing | ParaRel | It measures agreement across paraphrases of one fact. |
| Answer in English | Translate, answer, translate back | The same knowledge in all languages | mLAMA, XQuAD | They ask the same facts in many languages. |
| Answer a two-hop question directly | Answer the two sub-questions and compose | Composition | HotpotQA, 2WikiMultihopQA | Two-hop questions with known sub-facts. |
| Answer with a retrieved document | Answer from memory | Knowledge moved into the weights | Natural Questions (open-book against closed-book) | The same questions have both settings. |
| Predict the output of code | Run the code | Execution semantics | CRUXEval | It tests output prediction, and the interpreter is an exact teacher. |
| Premises in order A, B | Premises in order B, A | Order invariance | ProofWriter | The premises of a proof can be shuffled with no change to the answer. |

**Which path is the teacher?** In most rows, one path has more information or more computation (the full document, the reasoning steps, the retrieved text, the interpreter). That path is the teacher, and the gradient goes only to the other path. The paraphrase row and the order row are symmetric, so they have the collapse risk of section 6.

## 5. Ten graph shapes

Only shapes with a different graph are listed. The grouping and the names are from the discussion, not from a paper.

| # | Name | Rule | Shape | Example | Dataset that could benefit most | Reason |
|---|---|---|---|---|---|---|
| 1 | Inversion | g(f(x)) = x | Loop of two edges | Why / therefore | The reversal curse dataset | See section 3. |
| 2 | Commutative | f(x) = h(g(x)) | Two paths, one end point | Answer from document = answer from summary | FRANK | See section 4. |
| 3 | Idempotence | f(f(x)) = f(x) | Self-loop that stops | Correct a corrected text. The second pass must change nothing. | CoNLL-2014, BEA-2019 (grammar correction) | A corrected sentence is a fixed point by definition, and over-correction is a known error there. |
| 4 | Transitivity | A→B and B→C gives A→C | Triangle | If A is larger than B and B is larger than C, then A is larger than C. Also: translation through a pivot language = direct translation. | CLUTRR | It tests composition of kinship relations along a chain. |
| 5 | Split and merge | f(a + b) = f(a) ⊕ f(b) | Fan-out, then fan-in | Summary of a document = summary of the summaries of its chapters. | BookSum, arXiv / PubMed summarisation | Long documents with a natural division into parts. |
| 6 | Equivariance | f(T(x)) = T(f(x)) | Square | Rename "Alice" to "Bob" in the question. The answer must change in the same way and in no other way. | GSM-Symbolic | It changes names and numbers in GSM8K problems and reports a fall in accuracy. |
| 7 | Exclusion | f(x) ≠ f(not x) | Edge that pushes apart | A statement and its negation must not both get "true". | Negated LAMA, CondaQA | Both measure negation errors directly. |
| 8 | Coherence | The parts sum to the whole | Star | P(A) + P(not A) = 1. | The consistency checks of Fluri et al. 2023; calibration error on MMLU | Both measure whether the stated probabilities obey the rules of probability. |
| 9 | Monotonicity | More in gives no less out | Ordered chain | P(A and B) ≤ P(A). | The consistency checks of Fluri et al. 2023 | They include monotonicity checks on forecasts. |
| 10 | Self-prediction | predict(f(x)) = f(x) | Model points at itself | Ask "Will you be correct?", then compare with the real answer. | TriviaQA with stated confidence | Short answers with an exact check, so stated confidence can be compared with accuracy. |

**Three families**

- **Equality** (1 to 6): two computations must agree. The loss is the next-token loss with one side as the target.
- **Disequality** (7): two computations must disagree. This needs a contrastive loss.
- **Inequality and sum** (8, 9): the constraint is on probabilities, not on text. The loss is on the output distribution.

Self-prediction (10) is separate. It is an equality, but one side is a statement about the model, not about the world.

**Priority**

1. **Idempotence.** The cheapest. One prompt, no second relation. A good corpus line is already a fixed point of "correct this text", which gives an anchor.
2. **Exclusion.** The only shape that works against collapse. Add it as a partner to any equality shape.
3. **Transitivity.** A small form of composition. A toy model with no pre-trained prior does not compose with no explicit steps, so the gap should be largest there.
4. **Self-prediction.** It trains calibration, which no cycle on world knowledge can do. It is the most speculative.

## 6. Problems that the shapes share

- **One-to-many.** The backward step has many correct answers. "Give an example of a mammal" does not have to return "dog". The loss then punishes correct answers. Shapes with exact inverses (relation inversion, translation, negation) have less of this problem.
- **The identity shortcut.** The model can pass X through the middle step almost unchanged. The middle step needs a constraint, for example a limit on token overlap with X.
- **Collapse.** An equality shape with no fixed anchor has a trivial solution: the same empty answer on both paths. Why / therefore has an anchor, because the target X is real corpus text. Idempotence is the clearest risk: "output nothing" is a perfect fixed point.
- **Benchmark mismatch.** A benchmark has one gold answer, but a one-to-many cycle has many correct answers. The objective could improve the model and still show no gain on the benchmark.
- **Split and merge is only approximately true.** A summary of chapter summaries loses the links between chapters. A hard equality would train the model to ignore those links.
- **Stated probability against token probability.** Coherence and monotonicity need probabilities. "I am 70% sure" and the token probability are different things, and the choice changes the objective.

## 7. Related work to check first

These are from memory. No literature search was made for sections 4 and 5. Do the search before any claim of novelty.

- QAGS, FEQA, QuestEval: question-answering evaluation of summary faithfulness. Evaluation only.
- Consistency regularisation (UDA, R-Drop): agreement of two paths, but the paths are noise or dropout, not different reasoning routes.
- Self-consistency (Wang et al. 2022): a vote across reasoning paths at inference time. No training.
- Deductive Closure Training: trains on the model's own consistent set of statements.
- Contrast-Consistent Search (Burns et al. 2022): exclusion and coherence on a statement and its negation. It trains a probe, not the model.
- Fluri et al. 2023, "Evaluating superhuman models with consistency checks": negation, paraphrase, monotonicity and Bayes-rule checks. Evaluation only.
- Idempotent Generative Network (Shocher et al. 2023): trains an image model with f(f(x)) = f(x).
- CycleInstruct, RevThink, the reversal curse, back-translation: see `hand-off.md`.

## 8. Suggested experiments

Each one needs a prediction and a stop rule before the run, as in `hand-off.md`.

- **F1. Relation inversion at toy scale.** A synthetic set of facts with a known inverse for each relation. Train on one direction, run the cycle, test the other direction. Prediction: the cycle closes part of the reversal gap. Stop rule: if the gap is the same with and without the cycle, drop inversion for exact relations.
- **F2. Idempotence with and without exclusion.** Train "correct this text" with the fixed-point loss. Measure the collapse rate (empty or constant output). Then add an exclusion partner and measure again. Prediction: collapse with no partner, less collapse with one.
- **F3. Teacher / student summary faithfulness.** The diagram of section 4 with the document path as the teacher. Measure on FRANK. Control: the same number of training steps on plain summarisation.
- **F4. Transitivity at toy scale.** Train on A→B and B→C. Test A→C with and without the triangle loss.
