---
name: why-therefore
description: Run the why/therefore consistency ritual on a draft answer before committing to it. For each load-bearing claim, abduce its cause ("why?"), then check blind that the cause alone leads back to the claim and not to a plausible wrong alternative ("therefore?"), and revise every claim that fails. Use this whenever the user asks to "run the ritual", asks for a why/therefore or causal-consistency check, says "are you sure?", "sanity-check this" or "double-check your reasoning", and also when an answer rests on a chain of causal or factual claims where a self-contradiction would be costly, such as a root-cause analysis, a diagnosis, an explanation of why something happened, a plan with dependencies, or a summary of evidence.
---

# Why/Therefore ritual

"Why" is the inverse of "therefore". If a claim is something you really hold, it must survive a round trip: from the claim to its cause, and from the cause back to the claim. A claim that cannot make the trip back is a claim you wrote, not a claim you can support.

This skill is the in-context form of a training idea (a cycle-consistency loss between "why" and "therefore"). Nothing is trained here. The cycle runs on your draft answer, and each failure makes you revise the draft.

Status: this ritual is untested. No one has measured that it improves answers. Treat it as a structured self-check, not as proof.

## The steps

### 1. Draft, then list the claims

Write your answer as a draft. Do not send it. List the load-bearing claims, one sentence each. A claim is load-bearing if the answer changes when the claim is false. Three to seven claims is the usual number. More than that usually means the claims are too small.

### 2. Why: abduce a cause for each claim

For each claim, write one sentence that says why it is true.

Do not reuse the content words of the claim in the cause. If the cause repeats the claim, step 4 becomes a word match and tests nothing. "The build fails because the build is broken" will pass every check and tell you nothing.

### 3. Write a hard distractor for each claim

A distractor is the answer that a careless version of you could have given. It has the same topic and the same form as the claim, but different substance: a different culprit, the opposite direction, a different number.

An easy distractor makes the test worthless. "The cache is stale" against "Tim got a promotion" is not a test. "The cache is stale" against "The cache key is wrong" is a test.

### 4. Therefore: go back from the cause, blind

The question is: given only the cause, which follows, the claim or the distractor?

The reader of the cause must not know which option is the original. You wrote the claim, so you cannot do this step honestly with the claim in view.

- If you can start a subagent, use one for each claim, or one for the full batch. Give it only the cause and the two options in a random order, labelled A and B. Ask: "Which statement follows from this cause: A, B, or neither? Answer with one letter and one line of reasoning." Do not give it the user's question, your draft, or a hint.
- If you cannot start a subagent, do the step yourself as honestly as you can. Read only the cause line. Write what follows from it before you look at the options. Then compare.

### 5. Read the verdicts

- **Pass:** the reader picked the claim.
- **Fail:** the reader picked the distractor, or "neither".

A fail means the cause does not support the claim. There are two possible reasons. The cause is wrong, so go back to step 2 and find the real one. Or the claim is wrong or too strong, so change it, hedge it, or remove it.

Do not reword the cause until the reader is forced to pick the claim. That makes the ritual agree with itself and teaches you nothing. If you cannot find a cause that leads back to the claim without reusing its words, the claim is the problem.

### 6. Go to the root

For each claim that passed, ask "why?" again on its cause. Stop after three levels, or when you reach bedrock. Bedrock is something given: a fact from the user, a file or a source that you read, a definition.

Then compare the roots. Chains about the same subject must end in roots that can all be true at the same time. Two roots that contradict each other mean that the answer holds two views of the world. Resolve the contradiction before you answer.

A chain that never reaches bedrock is a claim with no support in this conversation. Say so in the answer, or go and get the evidence.

### 7. Paraphrase probe

Restate the user's question in two other ways. If your answer changes with the wording, the answer depends on the phrasing and not on the facts. Find out which reading the user means, or give an answer that holds for all of them.

### 8. Rewrite

Write the final answer from the claims that survived. Evidence in the conversation (files, tool results, sources, facts from the user) always outranks the result of the cycle. The cycle checks that you agree with yourself. It does not check that you are right.

## What to show the user

By default, show only the final answer and one line that says what the ritual changed, for example: "Check: 5 claims, 1 revised (the cache claim had no cause that led back to it)."

If the user asked for the ritual by name, also show the ledger:

| # | Claim | Why (cause) | Distractor | Verdict | Action |
|---|---|---|---|---|---|
| 1 | ... | ... | ... | pass / fail | kept / revised / removed |

Then list the roots from step 6 and any contradiction you found.

## Example

Question: "Why did the plants in my garden wilt?" The draft says: "They wilted from lack of water."

- Claim: The plants wilted from lack of water.
- Why: There has been no rain for three weeks and the hose was not used.
- Distractor: The plants wilted from too much fertiliser.
- Therefore, blind: from "no rain for three weeks and the hose was not used", the reader picks the claim. Pass.
- Root: "no rain for three weeks" is a fact from the user. Bedrock.

Now suppose the cause had been "Plants need the right conditions to grow". From that cause alone, the reader cannot choose between the claim and the distractor. That is a fail: the draft had a conclusion but no reason behind it. The fix is to ask the user about watering and fertiliser, not to assert the first guess.

## When not to use it

- Simple lookups and one-step answers. The ritual costs more than a mistake does.
- Creative writing, brainstorming and opinion, where there is no claim to check.
- Pure mathematics or code behaviour. A proof check or running the code is a better test than a cycle of sentences.

## Limits

- Untested. See the status note at the top.
- Consistent is not the same as correct. A wrong belief that is held consistently passes every step.
- If you do step 4 yourself, you know the answer already, and the test is weaker.
- The ritual adds model calls and time. Use the batch form of step 4 when there are many claims.
