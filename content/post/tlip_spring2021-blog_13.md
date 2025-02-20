+++
title = "Causal inference with latent variables for unmeasured confounding"
author = "Mark van der Laan"
description = ""
tags = [
    "resources",
    "statistics",
    "targeted learning",
    "Q&A"
]
date = "2021-06-30T13:38:00+00:00"
categories = [
    "Resources",
    "Statistics",
    "Targeted Learning",
    "Q&A"
]
+++

_This post is part of our Q&A series._

A question from graduate students in our Spring 2021 offering of the new course
"Targeted Learning in Practice" at UC Berkeley:

## Question:

> Hi Mark,
>
> Statistical analyses of high-throughput sequencing data are often made
> difficult due to the presence of unmeasured sources of technical and
> biological variation. Examples of potentially unmeasured technical factors are
> the time and date when individual samples were prepared for sequencing, as
> well as which lab personnel performed the experiment. Unknown sources of
> biological variation include samples' cell-type compositions in DNA
> methylation data, and cell-cycle activity in single-cell transcriptomic data.
> If these sources of variation are not accounted for in downstream analyses,
> like differential expression analyses, the inferential procedures will at best
> be inefficient, and at worst produce invalid results.
>
> Latent variable estimation procedures have been developed to address this
> issue, and are demonstrably successful when paired with differential
> expression methods that assume a linear outcome regression. However, this
> assumption of linearity might not be reasonable, and can also lead to invalid
> inference. Instead, approaches that require less stringent assumptions about
> the data-generating process and which target meaningful parameters, like the
> TMLE of the average treatment effect, are more appropriate. As an added
> benefit, the resulting estimates from this procedure might be endowed with
> a causal interpretation if the appropriate causal assumptions hold. My main
> question is therefore the following: What complications, if any, arise with
> regard to the causal interpretation of parameter estimates generated by
> semiparametric approaches that use latent variable estimates to control for
> unmeasured covariates? Put another way: Can the exchangeability condition be
> satisfied by simply employing these estimated latent variables in our
> estimation procedure?
>
> Best,
>
> P.B.

---

## Answer:

Hi P.B.,

Thank you for your interesting question.

For concreteness, let's say we care about the causal effect of a binary
treatment variable `$A$` on an outcome `$Y$`, and that we observe i.i.d. copies
of `$(W,A,Y)$`, where `$W$` is a set of baseline covariates/confounders.
However, we consider the case that we know that the randomization assumption is
violated: `$A$` is not independent of the potential outcome `$\{Y_0, Y_1\}$`,
given `$W$`. Nonetheless, our goal is to learn `$\psi^F = \mathbb{E}Y_1 -
\mathbb{E}Y_0$`, the average treatment effect (ATE).

Firstly, we could simply estimate the statistical estimand for the ATE, that is,
`$\Psi(P_0) = \mathbb{E} [\mathbb{E} (Y \mid A = 1, W) - \mathbb{E}
(Y \mid A = 0, W)]$`, and view it as a best approximation of what we target.
Let's view the TMLE of this estimand as a baseline method. Our goal should then
be to construct an estimator that is closer to `$\psi^F$` (or an equally
interesting causal quantity) than the TMLE, but that still performs as well as
or is competitive with the TMLE when `$W$` does happen to include all
confounders.

This might be an impossible goal, but it is a fair request. That is, I am
generally very worried about these latent variable methods when they start
relying on all kinds of parametric assumptions, especially so when the
statistical model is heavily restricted as well. In that case, it will be
heavily biased even if all confounders were measured.

Whatever estimator one constructs based on latent variable-type models, it will
map the data into a number, and that number will converge to its target estimand
as sample size increases. If the statistical model is the nonparametric model or
at least a realistic model (that can be defended scientifically), this algorithm
then describes a mapping from the statistical model to the real line
`$\mathbb{R}$`, even though the authors of the algorithm had a much more
restrictive model in mind. Thus, such a latent variable estimator has a target
estimand `$\Psi_l(P_0)$` defined on a realistic statistical model. However, once
we know what the estimand is, we could develop a TMLE for this `$\Psi_l(P_0)$`
which will have superior statistical performance by being asymptotically linear
and efficient for all `$P$` in the model. By contrast, the latent variable-based
estimator will at most be consistent in general and asymptotically linear within
a small parametric model.

Presumably, `$\Psi_l(P_0)$` will be different from `$\Psi(P_0)$`, and for both
we have a well-understood TMLE. So, now everything has been boiled down to
understanding `$\Psi_l(P_0)$`, why it would be an interesting measure, and why it
would somehow be closer to `$\psi^F$` (or another causal quantity of interest)
than `$\Psi(P_0)$` is to `$\psi^F$`, for a rich enough set of possible
data-generating experiments. Therefore, I conclude that if the goal is to handle
unmeasured confounders, one needs to come up with new causal assumptions, beyond
the randomization assumption, and establish an identification result.
Identification based on having instrumental variables is an example of such
a result. One should aim to present identification assumptions that do not
restrict the statistical model beyond a model that contains the _true_
data-generating distribution `$P_0$`. There is also interesting work possible
utilizing negative controls, which has been receiving attention in the
literature. Assuming that there is an unmeasured variable `$U$` for which `$A$`
is independent of `$Y_a$` given `$(W,U)$`, and then making assumptions about
`$U$` would be another venue, but this is in many ways much more reliant on
wishful thinking than actually having a study design that yields the relevant
assumptions, such as with instrumental variables and negative controls. When
going with this latter route, one should do so without restricting the
statistical model, while most of the literature does the opposite. If one does
that, it would be of interest to understand the resulting `$\Psi_l(P_0)$` when
these causal assumptions are violated, to make sure that it still represents an
interesting measure of association, such as the case with `$\Psi(P_0)$` for the
ATE under the randomization assumption.

So, the overall simple response to your question is that one should carry out
the roadmap as laid out, for example, in my [_Epidemiology_
paper](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4077670/) with [Maya
Petersen](https://publichealth.berkeley.edu/people/maya-petersen/) or the
Targeted Learning books:

1. begin with the causal model;
2. specify the causal quantity;
3. use identification assumptions to express the resulting statistical estimand
   `$\Psi_l(P_0)$`, which equals the causal quantity if the assumptions hold;
4. commit to a VALID statistical model, one containing the true `$P_0$`
   (presumably the validity of the statistical model should not be driven by the
   stated identification assumptions);
5. decide if you want to commit to the resulting estimand `$\Psi_l(P_0)$` or
   that the identification assumptions are just too unbelievable, in which case
   the estimand is too far away from the causal quantity under realistic
   scenarios;
6. if one does commit to both the statistical model and estimand, then the
   estimation problem is defined and we can proceed with developing a TMLE and
   inference (according to a TMLE roadmap); and, finally
7. a [nonparametric _sensitivity
   analysis_](https://pubmed.ncbi.nlm.nih.gov/24246288/), e.g., as in my
   proposal with [Iván Díaz](https://www.idiaz.xyz/), investigating how the
   statistical inference is affected by the size of the causal gap, involving
   developing some understanding of what magnitude of causal gap one might
   expect.

Best Wishes,

Mark

__P.S.__, remember to write in to our blog at `vanderlaan (DOT) blog [AT]
berkeley (DOT) edu`. Interesting questions will be answered on our blog!
