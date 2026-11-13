# Causal-consistency post-pre-training objective

> **Status: untested.** No experiment has been run. Start with [`hand-off.md`](hand-off.md).

A training stage that runs after pre-training and asks a language model to agree with itself across documents.

"Why" is the inverse of "therefore". Pick a line from the pre-training corpus, ask the model "Why?", then give its answer back, ask "Therefore?", and train the model to recover the original line against a distractor. The loss is plain next-token prediction.

- [`hand-off.md`](hand-off.md): the idea, the possible errata and the experiment plan
- [`skill/why-therefore/SKILL.md`](skill/why-therefore/SKILL.md): an in-context form of the same cycle, as a Claude skill (also untested)
- Blog post: https://ad-absurdum.me/blog/causal-consistency-training
- Slides: https://docs.google.com/presentation/d/1gSwNj3VmGG-11UbirovTkRNc0zRZEkNEETYd88vZfYs/edit?usp=sharing
- [`future-work.md`](future-work.md): other cycles, two-path (commutative) objectives and eight more graph shapes, each with the dataset that could benefit most

## Citation

GitHub shows a "Cite this repository" button from [`CITATION.cff`](CITATION.cff). The BibTeX form is:

```bibtex
@misc{nanda2026causalconsistency,
  author       = {Nanda, Souradeep},
  title        = {Causal-consistency post-pre-training objective},
  year         = {2026},
  howpublished = {\url{https://github.com/Ghost---Shadow/causal-consistency-post-pre-training}},
  note         = {Untested proposal.}
}
```

## Disclaimer

- **Nothing here is tested.** No model was trained and no benchmark was run. Each claim about a benefit is a hypothesis.
- **The dataset columns are guesses.** They show where to look first. They are not predictions with evidence.
- **The related work is from memory.** The paper names, years and findings were not checked against the papers, and no full literature search was made. Some of these ideas could already be published. Search before you claim novelty, and I claim no priority.
- **The text was written with an AI assistant.** The idea and the direction of the discussion are mine. The notes, tables and experiment plans were drafted with Claude and I have not verified each line.
- **The names of the graph shapes are an informal grouping.** They are not standard terms from a paper.
- **The skill is untested too.** It can make a model state causes that are false but sound correct.
- **This is personal work.** It does not represent an employer or a university.
- The MIT license applies to all files in this repository. As the license says, there is no warranty.

## License

[MIT](LICENSE)
