---
layout: post
title:  "Even more on 'Limitations of GCTA'"
date:   2016-02-23
categories: heritability
---

The discussion of "Limitations of GCTA..." has now grown to three articles and I encourage readers interested in the mechanics of this model to read each of them:

* [Yang et al, "Commentary on 'Limitations of GCTA'"](http://biorxiv.org/content/early/2016/01/20/036574), rebutting all empirical claims in the original manuscript.
* [Krishna-Kumar et al, "Response to Commentary"](http://biorxiv.org/content/early/2016/02/17/039594), reiterating all empirical claims in the original manuscript and looking at additional datasets.
* [Gamazon and Park, "SNP-based heritability estimation"](http://biorxiv.org/content/early/2016/02/18/040055), rebutting the fundamental mathematical claim in the original manuscript.

I greatly appreciate the passion that SK have shown in thinking and writing about this issue critically and from multiple different perspectives. However, my previous skepticism stands: there is no conclusive theoretical result showing that GCTA is biased when model assumptions are met, and the empirical results are confounded by relatedness. On the first point, I fear turning this into a pile-on so I'll leave my thoughts below the fold for those interested in the boring details. On the second point, the authors have confirmed that they indeed included genetically related individuals and moved on to a new dataset (which is of mixed ancestry, introducing a host of additional complexities).

This exchange also highlights the ups and downs of non-peer-reviewed/immediate pre-prints. On the one hand, it's awesome that this discussion is going on in a rapid, unfiltered way so geeks like myself can dig into the details as they emerge. On the other hand, having an un-answered commentary immediately go on-line places a tremendous amount of pressure on the other party to respond, which, coupled with lack of formal peer-review means the ideas are not always fully-formed. (*and yes I see the irony of complaining about not fully-formed ideas in a personal blog*). One possible advantage that pre-prints offer over the traditional commentary format is that the authors can spend several rounds hammering out differences internally and then release a more coherent point/counter-point. I think that's something that would help resolve the ongoing tension and open questions here.

---

**PS: Bias in REML estimates when assumptions are met**

The main disagreement stems from lack of clarity on what it means for REML assumptions to be met. Strictly speaking, **REML makes only one assumption**: that the phenotype is drawn from a multivariate-normal distribution with mean equal to zero and variance equal to the weighted sum of genetic relatedness matrix (GRM) and a residual. REML is then used to fit the $$h^2_g$$ parameter that maximizes the likelihood in this model. Note that no additional assumptions about the disease architecture, effect-sizes, LD, or relatedness have been made. Indeed, it is the *interpretation* of this parameter that requires additional assumptions.

Under the standard assumption that causal effect-sizes ($$\beta$$) are i.i.d (with Gaussian residuals) and all variables were standardized, this parameter corresponds to $$E[h^2_g] = \sum_j{\beta_j^2}$$ across SNPs $$j$$ typed in the GRM. Again, no assumptions are made about LD, but the disease architecture is now constrained to have uncorrelated effects. [*In the Gaussian Process formalism, this is equivalent to having a weak prior that all SNPs are causal and then letting the data shrink the causal effects/functions.*] Recall that relatedness violates this assumption by inducing correlated effect sizes.

Under the very strict (and rarely applied) assumption that the GRM from typed SNPs is an unbiased estimator of the relatedness at all SNPs, this parameter further corresponds to the total heritability ($$E[h^2_g] = h^2$$). The SK response treats this last assumption as a requirement for the model, but it is in fact only a requirement for the *interpretation* of the quantity being estimated. According to SK, $$h^2_g$$ must correspond to $$h^2$$, and they trivially show that this can be violated (for example by estimating from different chromosomes).

However, GCTA never makes this assumption, and SK haven't demonstrated that the alternative definition $$E[h^2_g] = \sum{\beta^2}$$ is invalid. In my opinion, defining $$h^2_g$$ such that it is trivially biased and ignoring an alternative, unbiased definition is not very illuminating. Moreover, I find the GCTA interpretation highly useful precisely *because* it differs from the pedigree-based $$h^2$$ quantity and estimates a meaningful property of a specific set of variants.