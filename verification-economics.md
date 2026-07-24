---
layout: article
title: "The Economics of Verification: How AI Agents Are Repriced as Generation Gets Cheaper"
description: "As generating answers becomes cheaper, verification becomes a core constraint on the value of AI agent systems."
---

> As generating answers becomes cheaper, the scarce capability is no longer simply producing an answer. It is knowing whether that answer deserves to be trusted. Verification is moving from a supporting component of agent systems to a core constraint on their value. Yet an easily overlooked trap remains: many systems that look collaborative are, in practice, betting on an uncalibrated signal.

## 1. The Same Accuracy Gain Can Come From Two Different Mechanisms

Multi-agent systems have acquired a compelling product narrative: let several models deliberate, vote, or check one another, and the resulting answer will be more reliable.

But an accuracy improvement does not explain where the improvement came from. At least two distinct mechanisms can produce the same headline result.

The first is **candidate coverage**. Different agents produce different approaches, increasing the chance that the candidate pool contains a correct answer. Diversity and independence may genuinely matter here.

The second is **candidate selection**. A correct answer already exists in the pool, and the system uses tests, a scorer, or another model to select it. Here, the decisive question is not how many participants are involved, but whether the judge is reliable.

Both mechanisms can raise final accuracy, but they imply different system designs and cost structures. The first expands the search space; the second realizes upside already present in the pool. Unless they are separated, it is easy to credit “multi-agent collaboration” for work actually done by a verifier. It is equally easy to assume that adding more rounds of deliberation will keep improving reliability when the verification signal itself is weak.

## 2. What a Fixed Candidate Pool Reveals

In the OracleGap experiments, I isolated the second mechanism by holding candidate pools fixed. The study covers LiveCodeBench, MATH, and GPQA-Diamond. For each task, the candidate set remains unchanged while the signal used to select among candidates varies. The question is how much of the pool’s potential improvement each mechanism can actually realize.

This setup separates three quantities:

- the performance of the first candidate;
- the theoretical performance of a perfect selector over the pool;
- the performance achieved by a real selector or verifier.

The pool-level oracle shows how much improvement is available. The distance between a real mechanism and that oracle shows how much opportunity the mechanism leaves unrealized.

Final accuracy alone, however, is still insufficient. An aggressive verifier may repair many initially wrong answers while also overturning many answers that were already correct. A conservative verifier may cover fewer cases while causing almost no harm. Three quantities therefore need to be tracked together:

- **Gross recovery:** how many initially wrong answers the verifier fixes;
- **Harm:** how many initially correct answers the verifier overturns;
- **Net capture:** how much oracle headroom remains after subtracting harm from gross recovery.

The **net oracle capture rate** below is net gain divided by oracle headroom. In count form, it is the number of net fixes divided by the number of recoverable cases. It is not the same as gross recovery rate.

On LiveCodeBench, different verification signals exhibit sharply different error structures. A same-model LLM selector recovers more cases but also causes substantially more harm. Model-generated tests that are actually executed have lower coverage but almost never damage an initially correct answer. High-quality public tests combine stronger recovery with near-zero harm. A verifier’s value therefore cannot be inferred from how often it checks an answer or how many failures it repairs. What matters is the net result of recovery and harm.

**Table 1. Benefit and risk profiles of verification signals on LiveCodeBench**  
N = 2,888. First-candidate accuracy is 72.33%. The oracle upper bound is 84.07%, leaving 11.74 percentage points of oracle headroom, or 339 recoverable cases.

| Verification signal | Gross recoveries | Harms | Net fixes | Net gain (95% CI) | Net oracle capture |
|---|---:|---:|---:|---:|---:|
| Executed model-generated tests | 80 | 2 | 78 | +2.70 pp [2.02, 3.43] | 23.0% |
| Public tests (signal-quality reference) | 235 | 0 | 235 | +8.14 pp [6.99, 9.36] | 69.3% |
| Same-model LLM selection | 199 | 98 | 101 | +3.50 pp [2.26, 4.73] | 29.8% |
| Cross-model LLM selection | 175 | 118 | 57 | +1.97 pp [0.74, 3.20] | 16.8% |

The public-test condition is not a fully deployable verifier. It serves as a signal-quality reference in the diagnostic experiment. Its purpose is not to imply that a deployed product can access ground-truth answers in advance. Rather, it shows that even when two workflows both appear to “run tests and select a candidate,” the quality of those tests can radically change how much oracle headroom is captured.

Table 1 also shows why executability does not mechanically determine net gain. Model-generated tests are extremely low-harm but have limited coverage. The same-model LLM selector recovers more cases but causes much more harm. Their net gains are not statistically distinguishable in this experiment: the paired difference is +0.80 percentage points, with a 95% confidence interval of [-0.38, 2.01]. Executability produces a distinctly low-harm error profile, not an automatic advantage in net accuracy.

GPQA-Diamond illustrates the opposite boundary. In the studied configuration, the candidate pool offers only 18 recoverable cases beyond the first candidate. The selectors repair very few errors while overturning more answers that were already correct, producing negative net gains.

**Table 2. Negative net gains on GPQA-Diamond**  
N = 594. First-candidate accuracy is 47.64%. Oracle headroom is 3.03 percentage points, corresponding to 18 recoverable cases and an oracle upper bound of 50.67%.

| Verification signal | Gross recoveries | Harms | Net fixes | Net gain |
|---|---:|---:|---:|---:|
| Same-model LLM selection | 5 | 15 | -10 | -1.68 pp |
| Cross-model LLM selection | 6 | 20 | -14 | -2.36 pp |

> **Statistical note:** The main LiveCodeBench experiment uses task-cluster hierarchical bootstrap confidence intervals over three seeds. In the paper, GPQA is reported as a descriptive boundary analysis, with per-seed ranges and explicit denominators rather than the same bootstrap procedure. Table 2 therefore retains descriptive point estimates instead of mixing two statistical reporting regimes in a single column.

The GPQA result is not merely a case of a weak verifier. The pool itself contains almost no meaningful variation. In 87.54% of pools, all five candidates give the same answer; the mean number of unique answers per task is only 1.138. Five nominal generations are often five copies of the same answer. If the pool contains no better option, even a perfect selection strategy has nothing to recover. Overriding the first answer then creates opportunities for harm without creating comparable opportunities for repair.

The negative result is therefore a direct expression of the oracle-gap mechanism. Collaboration depends not only on the task, but also on whether a particular model and sampling configuration generate meaningfully different candidates that sometimes include the correct answer. On the same task, switching to a smaller 9B model compresses recoverable mass further, to 0.67%, while the share of completely answer-identical pools rises to 94.44%. This comparison shows that the oracle gap is a joint property of the task, model, and sampling configuration, not an immutable property of a benchmark.

The 35B experiment still contains only 18 recoverable cases, so it should be read as a mechanism-revealing boundary case for this model, pool, and selection setup—not as a universal law about knowledge-intensive question answering.

The selection-stage accounting can be summarized by a simple identity:

> **Net verification gain = correct answers recovered - correct answers harmed.**

This is more informative than the number of agents in a system. The fixed-pool evidence does not show that independence is worthless. It shows that **once the candidate pool is fixed, selection gain is governed first by recoverable mass, signal coverage, conditional selection quality, and harm**. Whether independence improves candidate generation and coverage is a separate question that must be measured separately.

> **Methods and reproducibility:** N denotes the number of evaluation observations. “First candidate” corresponds to `sample0` in the experiment records, and percentage points refer to absolute differences in accuracy. The paper and repository disclose the model combinations, number of candidates per task, verifier activation and fallback rules, dataset versions, and bootstrap procedures. The repository includes a frozen `numbers.json`, run-label and model provenance, per-example records, a reproducibility guide, checksums, and numerical assertion scripts. The paper’s numerical claims can be audited without making model API calls. This article uses LiveCodeBench and GPQA-Diamond to illustrate the positive and negative boundaries; complete MATH results and experimental details appear in the paper.

## 3. Verification Is Not a Switch. It Is a Cost Curve.

Verification is often described as a binary distinction between “decidable” and “undecidable” tasks. Real systems lie on a continuum.

At one end is low-cost verification with strong ground truth. Code can compile, unit tests can run, numerical results can be substituted back into an equation, and structured records can be cross-checked. The verification signal directly touches an objective constraint and is usually cheap, fast, and repeatable.

At the other end is high-cost verification with weak or delayed ground truth. Does a codebase still contain a hidden security vulnerability? Has a legal opinion missed a decisive condition? Is a medical recommendation safe for this particular patient? Did a long-running agent deviate from the user’s intent in an obscure branch of its execution trace? These questions are not absolutely unanswerable. Reliable judgment simply requires more context, more time, more expensive expertise, and sometimes observation of outcomes that arrive much later.

The economics of a system therefore depend less on whether a problem is “decidable” in the abstract than on:

- the cost of obtaining a reliable judgment;
- the distance between the available signal and the eventual outcome;
- how long it takes for ground truth to arrive;
- the asymmetric costs of false positives and false negatives;
- whether the system can abstain or escalate when confidence is insufficient.

Signal fidelity cannot be reduced to a single accuracy number. Two verifiers can have identical average accuracy while differing radically in harm rate, calibration, or performance on high-risk cases. In security, law, and medicine, those differences often matter more than the average score.

## 4. The Real Allocation Problem Is Generation Budget Versus Verification Budget

From this perspective, a multi-agent system does not face a simple choice about whether to add another model. It faces a budget-allocation problem.

More generation budget can increase candidate coverage and raise the pool-level oracle ceiling. More verification budget attempts to capture that ceiling. Both sides exhibit diminishing returns. More candidates do not increase the probability of a correct answer proportionally, and more rounds of checking do not automatically make the verification signal more accurate.

Whether verification is worth funding depends on a fuller accounting:

> **Expected value of corrections - expected cost of harm - cost of verification.**

When a task offers cheap, executable feedback, verification is more likely to produce positive returns. When reliable judgment depends on experts, delayed outcomes, or complex context, verification can still be valuable, but the system must also support calibration, abstention, human escalation, and clear responsibility boundaries. Otherwise, “more checking” merely replaces one wrong answer with another wrong answer that sounds more authoritative.

A strong agent architecture should therefore not impose the same collaboration protocol on every task. It should first locate the task on the verification cost curve, then decide whether to expand the candidate pool, run tools, invoke a specialized verifier, request human review, or stop when the available evidence is insufficient.

## 5. The Value-Control Point in the Agent Economy Is Shifting

Generation remains important, but it is diffusing rapidly. Model capabilities are becoming more widely available, unit inference prices continue to fall, and the gap between open and closed models keeps changing. Together, these trends compress the premium attached to merely generating a plausible-looking answer.

As candidate answers become less scarce, system value increasingly depends on three questions: Can the system select reliably? Can it assume accountability? Can it demonstrate that it did not quietly fail inside a real workflow?

Low-cost verification with strong ground truth will also become commoditized. Compilers, test frameworks, rule-based checks, and structured reconciliation can be reused, and their marginal cost often falls with automation. Harder to replicate are the verification capabilities required in high-risk settings with weak ground truth: domain expertise, adversarial evaluation, process auditing, long-term feedback data, institutional credibility, and a willingness to assume responsibility for mistakes.

A cautious but consequential industry thesis follows:

> **The value-control point in the agent economy is shifting from “who can generate” to “who can verify at a credible cost and assume accountability.”**

This does not mean profits will automatically flow to the hardest-to-verify domains. High verification cost often comes with poor scalability, greater liability, and longer delivery cycles. Willingness to pay may rise while profit does not. Durable premiums are more likely to accrue to teams that can turn expensive, ambiguous judgment into a repeatable institution. Such teams will possess not only better models, but also feedback loops, audit processes, escalation protocols, and ways to price responsibility.

Today, those capabilities may appear in training environments and high-quality evaluations. Next, they may emerge in agent-behavior audits, review layers for professional workflows, risk underwriting, and protocols that assign rights and responsibilities across agents. The exact institutional form remains uncertain, but the underlying questions will persist: Who verifies? Who bears the cost? What evidence is sufficient to act? Who is responsible when the judgment is wrong?

## 6. The Evidentiary Boundary of This Argument

The empirical study currently covers code generation, mathematical reasoning, and graduate-level multiple-choice questions, using a limited set of model families. Because the fixed-pool design intentionally isolates candidate selection, it cannot establish whether independence creates value through candidate generation, search trajectories, or knowledge coverage.

The further claim that the agent economy’s value-control point is shifting is an industry thesis, not a conclusion established by the experiments themselves. It still requires evidence from real workflows: how much cost is spent on generation, review, and error handling; what forms of reliability customers will pay for; and how liability costs affect the margins of verification services.

The deeper difficulty is that verifier reliability may itself be hard to measure when ground truth is weak or delayed. If one model can only be evaluated by another model, the verification problem may simply have been moved back one layer. Building external feedback, delayed-outcome records, and accountability trails may matter more than adding another “review agent.”

## 7. Three Questions to Ask of Any Collaborative Agent System

“More agents produce better answers” is a convenient slogan, but it is not a sufficient design principle.

The next time an agent system claims an accuracy improvement from collaboration, the useful questions are not how many roles it added or how many rounds the agents debated. They are:

1. Did the additional agents increase the probability that the candidate pool contained a correct answer?
2. How many initially wrong answers did the verifier recover, and how many initially correct answers did it harm?
3. After subtracting harm and verification cost, how much oracle headroom did the system actually capture?

A team that can answer these questions separately knows whether its system is collaborating, verifying, or merely placing a more elaborate bet.

As generation gets cheaper, reliability does not automatically become cheaper with it. Knowing when to trust, when to stop, and who is willing to stand behind the judgment may become the hardest capability in the agent economy to replicate.

## Paper and Reproducibility Materials

- Paper: [Oracle Gap and Signal Fidelity: A Fixed-Pool Diagnostic for Test-Time Collaboration](https://arxiv.org/abs/2607.17531)
- Code, data, and audit scripts: [OracleGap GitHub repository](https://github.com/AmGarfield/OracleGap)
- Reproduction instructions: [Reproducibility Guide](https://github.com/AmGarfield/OracleGap/blob/main/docs/REPRODUCIBILITY.md)
