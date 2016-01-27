---
layout: post
title:  "On 'Limitations of GCTA...'"
date:   2015-12-30 17:21:14
categories: heritability
---

The recent paper of {% cite Krishna-Kumar:2015 %} [[pdf](http://www.pnas.org/content/early/2015/12/17/1520109113.full.pdf)] makes a provocative claim challenging the GREML model (implemented in the software GCTA), which has been widely used to estimate components of heritability: "_Here, we show that GCTA applied to current SNP data cannot produce reliable or stable estimates of heritability_". This paper continues an important discussion on the robustness of GREML, but I believe the empirical claims about bias and reliability are overstated and should not cast doubt on properly computed GREML estimates. 

**What does GCTA do?** The SKK paper does a good job of summarizing the assumptions and goals of GCTA. In brief, GCTA uses genetic data from unrelated individuals to estimate the variance in phenotype that can be explained by genotyped SNPs (often called SNP-heritability, or h2g). It assumes a model where phenotypes are drawn from a multi-variate normal distribution with variance modeled by a genetic relatedness matrix (GRM) and an environmental term (the identity matrix). It directly estimates entries in the GRM as the covariance of a pair of individuals over all genotyped SNPs; it uses the REML algorithm to identify the maximum likelihood estimate of the variance attributed to the GRM; and it uses the Hessian of the likelihood to approximate the corresponding standard error. GCTA and related papers are landmarks in genetics, but the criticisms apply to all methods that use this combination of GRM and the REML algorithm, so I will refer to the general approach as GREML to distinguish between model and implementation.

**Why is this a big deal?** Geneticists are generally interested in understanding the "genetic architecture" of a trait: the distribution/relationship between trait causing effect-sizes and genetic variants. How much of a trait is due to rare or common variants? How much is due to variants that can be tagged by a genotyping array, or an exome array, or coding SNPs? There are lots of ways to answer these questions, but the primary advantage of the GREML approach is that it provides estimates that are neither biased by sample size nor dependent on individually significant associations. This means that the seminal finding of [_Yang et al. 2010 Nature Genetics_] that common variants explain 45% of height is a conclusion about the trait, not about the study, and continues to be relevant to study design in 2015. Hundreds of such analyses have since been carried out, and if the estimates are shown to be unreliable then they can no longer be used to make accurate predictions about the future and lose their value.

**What do Krishna Kumar et al. show?** The SKK paper demonstrates a relationship between the GREML likelihood and the singular value decomposition of the GRM, which allows them to make general theoretical claims about the instability of GREML estimates. This is an important advance; GREML is mostly treated as a black-box algorithm for estimating one very important parameter, and more intuition on its stability and expected performance is an important contribution. However, the paper also makes empirical claims about the instability of GREML, concluding that the estimates are biased when all model assumptions are met as well as in analyses of real traits. Here, I believe the paper draws conclusions that are not supported by the data and overstates the instability of the GREML approach. In this post, I will focus on "Case 1" from the paper, where the GREML assumptions are fully satisfied but resulting estimates are purported to be biased.

**What is bias?** A biased estimator is one where the estimate does not converge to the underlying population parameter with increasing information. In the case of GREML, this could mean that (A) the h2g estimate is systematically and significantly different from the true h2g in the population; or (B) the estimated standard error is systematically different from the true sampling error of estimate. Scenario A is a big problem for drawing conclusions from current estimates. Scenario B is sub-optimal but is only a big problem if the bias is an underestimate - that is, we think the estimate is more precise than it truly is. The SKK paper does not explicitly state what kind of bias they identify, but "Case 1" only focuses on the standard error of the estimate (specifically: "_More than half of these estimates lie outside the 95% confidence interval_") and therefore I believe their focus is on Scenario B. Here, I re-analyzed the SKK simulations to quantify this bias.

**Simulations from the SKK model.** The SKK paper describes a simulation framework that satisfies the GREML assumptions: 50,000 SNPs for 2,000 individuals are randomly generated; SNP effect-sizes are independent and identically distributed (the infinitesimal model); and there is no population structure. I replicated the SKK simulations (see details and code below) and re-plotted their Figure 2. Specifically, I generated 50,000 random SNPs (MAF=0.5); 2,000 individuals; and an infinitesimal phenotype with h2g = 0.75. The GREML estimate of h2g for this sample was 0.60 (se 0.15), within the noise of the SKK simulation estimate of 0.69 (se 0.15). I then performed 500 down-samples of the data to 5,000 random SNPs, recomputed the GRM, and re-estimated h2g. Following the SKK format, I plot the overall estimated h2g per-SNP (0.69/50,000) and the density of the down-sampled h2g per-SNP (down-sampled h2g/5,000) below:

![down-sampled heritability estimates]({{ site.baseurl }}/images/plot_krishna_kumar_pnas.svg)

First, the down-sampled estimates are roughly symmetric and centered at the full-sample estimate (solid line in the middle). The average down-sampled estimate is 1.17e-5 (se 4.0e-7) and not significantly different from the full-sample estimate of 1.21e-5. **Therefore, the GREML estimate appears to be unbiased in the SKK simulation.** Second, each down-sampled GREML estimate also computes an approximate 95% confidence interval, and I have plotted that (averaged over 500 runs) with dotted lines. This estimated confidence interval (CI) is wider than the empirical 95% interval of down-sampled estimates (blue shaded region). **Therefore, the GREML confidence interval appears to be accurate or mildly conservative in the SKK simulation.** The standard error is a measure of the expected sampling standard deviation of the estimate, or (for an unbiased estimator) the interval where the true underlying parameter is expected to lie - both of which are confirmed to be accurate in these simulations.

Finally, the estimated confidence interval from all 50,000 SNPs is shown in dashed lines. This is the estimate the SKK paper finds to be much lower than the down-sampled variance (red arrows in SKK Fig. 2), and that observation is confirmed here. There is no justification provided for why the estimated CI from a 50k SNP GRM should be compared to the variance of estimates from a 5k SNP GRM. Both the MLE and the likelihood surface is specific to the genetic data used for estimation and make no guarantees about sub-sampled data. Indeed, this very point motivates the SKK paper: "The MLEs produced by GCTA depend on the properties of the GRM matrix". In the context of linear regression (where the dual of the GRM is used) the SKK analysis is like comparing the standard error of a regression coefficient computed in 50k individuals to the variance in the estimate computed in random 5k sub-groups. The down-sampling strategy proposed by SKK is therefore not an assessment of bias, and re-analysis of their simulations here shows that the GREML estimates are indeed unbiased.

**Other work**. Naturally, the first hurdle a method should clear is that it works under the model assumptions. Several independent analyses have shown this to be the case for GREML, including the work of {% cite Speed:2012 %} (see Fig.1 "ALL") and {% cite Zhou:2013 %} (see Fig.1B,D). Importantly, Speed et al. elegantly showed how _violations_ to the model assumptions on LD can lead to bias, and producing an optimal solution continues to be an open challenge. Lastly, the developers of GCTA have recently released a [statistical power calculator](http://cnsgenomics.com/shiny/gctaPower/) to compute the expected standard error of h2g given sample parameters. For a sample size of 2,000, h2g of 0.75, and default GRM variance (2e-5) the expected standard error is 0.15 and matches what was observed in the full-sample simulations. For a sample size of 2,000, h2g of 0.75/10 and down-sampled GRM variance of 10*2e-5, the expected standard error is 0.05, also matching what was observed in the down-sampled simulations. In general, GREML bias under model violations (see also {% cite Golan:2014 %}) is an important area of research, and the unique perspective provided by the SKK paper is useful. But the method has been thoroughly reproduced when model assumptions are satisfied and the results of "Case 1" in {% cite Krishna-Kumar:2015 %} do not cast doubt on previous simulations and theory.

**References**

{% bibliography --cited %}

*****

# Footnotes

* The SKK paper contains a typo on pg.3, which states that the MLE has SD 1.51/50,000 = 3.1e-6. In fact, 1.51/50,000 = 3.1e-5, which would have fully covered the variance they observed. I assumed the intended computation is 0.151/50,000 = 3.0e-6, which is consistent with the computation on pg.4.

* The SKK paper correctly notes that GREML assumes each SNP to have a random, i.i.d effect-size, but their simulation framework includes 5,000 non-causal SNPs which violates this assumption. For clarity, I've simulated every SNP to be causal, but in practice this assumption does not matter much unless the fraction of causal variants is very low. Below I've run a separate simulation where 10% of the SNPs are causal (more realistic and aggressive than the SKK set-up where 90% are causal) and the previous conclusions hold. See Speed et al. and Zhou et al. for additional torture tests.

![down-sampled heritability estimates]({{ site.baseurl }}/images/plot_krishna_kumar_pnas.noninf.svg)

* Fig. 2 of the SKK paper is bounded at zero. While a variance estimate <0 in real data is un-intuitive, it is important to allow the MLE to go below zero when assessing bias in simulations. In the simplest case where the truth is zero, an estimator that's bounded at zero will always appear biased. This likely explains the asymmetric distribution observed.

# Simulation details

The code for this simulation framework has been made available at [https://github.com/sashagusev/SKK-REML-sim](https://github.com/sashagusev/SKK-REML-sim). The simulation framework is written in R and divided into three main components which rely on GREML functions I have implemented in the `func_reml.R` file. These functions use GCTA-like convergence criteria, and should give nearly identical outputs to running GCTA directly. The workflow is: (1) Generate the full-sample genotypes, GRM, and phenotype and estimate full-sample h2g; (2) Load the data from (1), down-sample to 5,000 SNPs and re-estimate heritability 500 times; (3) Collate and plot the results.

## (1) Full-sample simulation

Code to simulate the genotypes and estimate heritability is in the file `sim.R` and assumes `func_reml.R` exists in the same directory. This script will generate a "sim.rbin" file which is necessary for step (2). The code is outlined below.

Load GREML functions and define the parameters:
 
~~~ R
source('func_reml.R')

# reproducibility!
set.seed(1234)

N = 2000
P = 50000
h2 = 0.75
~~~

Generate genotype matrix with MAF = 0.5, and kinship:

~~~ R
Z = matrix(rbinom(N*P,2,0.5),nrow=N,ncol=P)

# standardize
for ( i in 1:P ) {
Z[,i] = (Z[,i] - mean(Z[,i]))/sd(Z[,i])
if ( i %% 1000 == 0 ) cat(i,'\n',file=stderr())
}

# generate kinship
A = Z %*% t(Z) / P
~~~

Generate an infinitesimal, heritable phenotype:

~~~ R
# 1. SNP effects
u = rnorm(P,0,1)
# 2. genetic value
g = Z %*% u
g = (g - mean(g))/sd(g)
# 3. add environmental noise
y = sqrt(h2) * g + rnorm(N,0,sqrt(1-h2))
y = (y - mean(y))/sd(y)

# save data
save(N,P,h2,y,g,u,A,Z,file="sim.rbin")
~~~

Estimate the h2g of the phenotype using GREML:

~~~ R
# func_reml requires a list of GRMs
K = list()
K[[1]] = A
# see func_reml.R for details, this runs GCTA-style GREML
reml = aiML(K,y,c(0.5,0.5))
cat( "estimate" , reml$h2 , reml$se[1] , '\n' )
~~~

Final output is `"estimate 0.603424 0.150628"` and is also in the file `sim.out`.

## (2) Down-sample simulation

Code to down-sample the data and re-estimate heritability is in `run_random.R`, which must be run using the command:

~~~ bash
R --slave --args $SEED < run_random.R
~~~

The concatenated output from 500 runs (with seeds 1-500) is in the file `run_random.out`.

## (3) Plotting

Code to generate the above figures is in `plot.R` which assumes `sim.out` and `run_random.out` both exist.
