# Landscape scan: what already exists

First pass at the prior art, done before writing any templates. The point is to
find where the crowded parts are so we don't rebuild them.

## The crowded part: "learn by building a project"

This space is saturated and the incumbents are good and free.

| Resource | What it is | Why it matters to us |
| --- | --- | --- |
| [project-based-learning](https://github.com/practical-tutorials/project-based-learning) | Curated index of build-it tutorials across ~25 languages. 281k stars. | Any flat list of "build a X" links we make is a worse copy of this. |
| [build-your-own-x](https://github.com/codecrafters-io/build-your-own-x) | Rebuild Git, Docker, Redis from scratch. Maintained by CodeCrafters. | Owns the "reimplement the tool" niche. |
| [CodeCrafters](https://codecrafters.io/) | Paid, tested challenges in the build-your-own-x format for experienced engineers. | Proof that graded, executable projects can be a business. |
| [Exercism](https://exercism.org/) | Language fundamentals and idioms, mentored. | Owns the "think in the language" layer below projects. |
| [The Odin Project](https://www.theodinproject.com/) / [Full Stack Open](https://fullstackopen.com/) | Full free job-oriented web curricula. | The bar for a complete linear path. |
| [roadmap.sh](https://roadmap.sh/) | Role-based skill graphs. | Already does role to skill list. Don't redo it. |

The recurring criticism of these is not quality but shape: they are linear and
built for an average learner who does not exist. That is a real opening, but a
narrow one.

## The thin part: concept anchored to real code at scale

Our stated idea is different from the list above. It runs the other direction:
start from one concept and show how it appears in production systems, not start
from a project and hope the concept lands.

What exists near this, and what is missing:

- [grep.app](https://grep.app/) and [Sourcegraph](https://sourcegraph.com/) can
  search across a million public repositories. They give raw hits with no
  sequencing, no annotation, and no answer to "why is it written this way."
- Academic work has catalogued worked examples pulled from open-source projects.
  The systematic mapping study on example-based learning in software engineering
  education (arXiv:2503.18080) found five studies specifically using OSS-derived
  worked examples, and describes a portal for cataloguing them. It is a research
  prototype, not something a learner uses.
- Nobody appears to ship a maintained, curated ladder from toy usage to
  production usage for a specific concept.

This is the gap worth taking.

## The constraint we have to design around

Dropping a beginner into a large real codebase to learn a basic construct is the
approach with the most evidence against it. Kirschner, Sweller and Clark
(Educational Psychologist 41(2), 75–86, 2006) argue that minimally guided
instruction fails for novices because free exploration of a complex environment
consumes working memory that should be going to the concept. The same literature
supports the worked-example effect: novices learn more from being shown a solved
case than from finding one.

The design consequence is that "see the for loop in a big system" cannot be rung
one. It has to be a late rung on a ladder, and each rung has to strip away
context the learner does not yet have.

Proposed rungs per concept:

1. Minimal correct usage, five lines, no dependencies.
2. Idiomatic usage as a working developer would write it, with the naive version
   shown alongside for contrast.
3. A real excerpt from a named repository, with everything not related to the
   concept stubbed out, plus a note on what was removed.
4. The unedited file in its repository, with a reading path and the specific
   line numbers that matter.
5. The failure mode: what breaks at scale, and what production code does instead.

Rung 5 is where most of the value is and where every existing resource stops.

## A caution about the "for loop" example

Checking the premise honestly: in large production code the explicit loop is
often the thing that is absent. Production Python leans on comprehensions and
vectorised operations; production JavaScript leans on map, filter and reduce;
production Go keeps the loop but the body is mostly error handling. So a truthful
"for loop at scale" lesson frequently reads as an explanation of why the loop was
not written.

That is a better lesson than the one implied by the framing, but it means the
concept inventory has to be derived from what recurs in real repositories, not
copied from a textbook table of contents.

## The job-description axis

Job postings are marketing documents. They are keyword-stuffed, inconsistent
between companies, and inflate requirements. Deriving a concept inventory from
them by hand will not scale, and doing it with a model produces plausible mush.

Use an existing taxonomy as the spine instead:

- [Lightcast Open Skills](https://lightcast.io/resources/blog/open-skills-taxonomy)
  — open, 32,000+ skills, built from job postings, updated on a two-week cycle.
  Closest fit to "what employers actually ask for."
- [O*NET](https://www.onetonline.org/) — US occupational data, stable, coarse.
- [ESCO](https://esco.ec.europa.eu/) — EU, multilingual, maps skills to
  qualifications. Relevant if Arabic or multilingual output is ever in scope.

Our contribution is the mapping from a taxonomy skill to a concept ladder, not a
new taxonomy.

## If the audience is models rather than people

If part of the goal is producing training or evaluation material for AI models,
that is a different project with different prior art:

- [SWE-bench](https://www.swebench.com/) — 2,294 tasks from 12 Python
  repositories, real GitHub issues resolved in full repository context.
- SWE-rebench (arXiv:2505.20411, arXiv:2602.23866) — automated, continuously
  updated, decontaminated task collection, now language-agnostic, built for
  reinforcement learning environments.

Competing here requires executable environments and verified tests, not prose.
It should not be mixed into the human-facing work under one roof.

## Open questions

1. Who reads the output: a human learner, a model in training, or an agent that
   scaffolds a project from a template?
2. One role and one language for the first slice, or breadth from the start?
3. How do we keep excerpts from rotting when upstream repositories change?
