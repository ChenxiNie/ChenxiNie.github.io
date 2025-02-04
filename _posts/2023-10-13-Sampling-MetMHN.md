---
title: 'Modeling Metastatic Progression With Sampling-MetMHN'
date: 2023-10-13
permalink: /posts/2023/10/Sampling-MetMHN/
tags:
  - Cancer progression models
  - Metastasis 
  - MCMC Sampling
---


We're excited to introduce **Sampling-MetMHN**, a fast approximate cancer progression model that models both primary tumors and metastases. To the best of our knowledge, it is one of the first cancer progression models that manages to model the genetic progression of primary tumors and metastases simultaneously. 

Metastatic cancer progression 
=================

The term metastatic cancer refers to the systematic spread of cancer from the initial tumor site to different areas of the body. We often refer to the tumor in the initial tumor site as the primary tumor and the tumor that colonized in other areas as metastasis or secondary tumor. 

Metastatic cancer progression is highly selective. In cancer patients, large numbers of cancer cells are released into the circulation system on a daily basis. However, only less than 0.1% of tumor cells establish metastatic secondary tumors. 

Apart from being highly selective, metastatic cancer progression is organ-specific as well. This organ-specific aspect of metastatic progression is captured by the famous seed and soil hypotheses by Paget in 1889. Paget noticed that, in nature, when plants go to seed, their seeds are carried in all directions, but the seed can only grow if it lands on congenial soil. Drawing an analogy from this observation, Paget suggests in the seed and soil hypothesis that cancer cells from the primary tumor (the seed) can spread to different parts of the body. However, these cancer cells can only succeed in colonizing if the microenvironment on which they land is pleasant. 

Existing models of metastatic cancer progression
=======

Existing models of metastatic cancer progression can be categorized into two classes. The first category models metastatic cancer progression as a growth model and these models do not focus on the genomic side of metastatic cancer progression. The second category is genomic models, where the interactions of genomic events are considered. One particular subtype of the genomic model is the logical model where they utilize regulatory networks that are constructed from published literature. These regulatory networks describe the interactions between nodes of the model (genomic events). Using these regulatory networks, these models define a list of specific logical rules for each genomic event. Although these models are able to model interactions between genomic events, they face scalability issues. If the model needs to be expanded to include additional events, we would have to consider updating every one of the logical rules manually. Moreover, potentially essential interactions between nodes can be missed due to a lack of literature support. 

In an effort to overcome the drawbacks of the existing models, we propose Sampling-MetMHN, a genomic model that builds on existing cancer progression models developed for primary tumors but adapted to be able to model both the primary tumors and metastases. It is both scalable and able to model the interactions of genomic events. 

MetMHN: A cancer progression model that works on metastasis
==================

## The base: Mutual Hazard Networks

Sampling-MetMHN is based on Mutual Hazard Networks (MHN). The original paper describing MHN in detail can be found in the key reference section below. MHN is a state-of-the-art cancer progression model that models the progression of one single tumor. It models tumor progression as a continuous time Markov process. It is assumed that every tumor starts from a mutation-free state and evolves according to a Markov Chain until it reaches a state where all of the genomic events have happened. 

![Alt text](https://github.com/ChenxiNie/ChenxiNie.github.io/blob/master/images/Markov_Chain_MHN.png?raw=true)

In an MHN with $n$ genomic events, each datapoint is encoded as a bitstring of length $n$. The $i$=th bit encodes the presence (1) or the absence (0) of the $i$-th event. The table below illustrates an example datapoint in MHN where mutations 1, 2, and 5 are present and mutations 3 and 4 are absent. 

| Mut 1 | Mut 2 | Mut 3 | Mut 4 | Mut 5 | ... |
|-------|-------|-------|-------|-------|-----|
| 1     | 1     | 0     | 0     | 1     | ... |


## From MHN to MetMHN: Enabling metastasis in MHN framework 

Metastatic cancer typically starts with one primary tumor and then the primary tumor goes through a series of steps to form an initial metastatic tumor at a distant site. Based on this observation, we define the seeding event as a representation of the steps that the primary tumor takes to successfully form an initial metastatic tumor. We assume that prior to the successful seeding event, the primary tumor and the (potential) metastasis evolve jointly as one physical entity and every mutation that occurs in the primary tumor also occurs in the (potential) metastasis. Once the seeding event has happened, the primary tumor and the metastasis are physically separated and are assumed to evolve independently. 

![](https://raw.githubusercontent.com/ChenxiNie/ChenxiNie.github.io/e5bb3732515d30968d809134d23e935deacbc84d/images/CTMC_MetMHN.svg)

Similar to MHN, in MetMHN, each datapoint is encoded as a bitstring as well. However, to include both the primary tumor and the metastasis, we have to double the length of the bitstring. Additionally, we added one bit for the seeding event. Thus, in MetMHN with $n$ events, the final length of that datapoint's bitstring is $2n+1$. 


![](https://raw.githubusercontent.com/ChenxiNie/ChenxiNie.github.io/60d391ce993db4d5d691326e965f53efebe33b02/images/Met_MHN_State_Space.svg)

## From MetMHN to Sampling-MetMHN: a Markov Chain Monte Carlo Sampler for MetMHN 

When using first-order optimization algorithms, the majority of time is devoted to calculating the gradient of marginal log-likelihood with respect to parameters. Here, we propose a MCMC based sampling algorithm to calculate the derivative of MetMHN efficiently. The idea of this method closely follows the work of Gotovos et al. (See key reference below) in which they developed a similar sampling algorithm for the original MHN. 

The input of Sampling-MetMHN is a collection of bitstrings representing each patient's genotypes. These bitstrings tell us about which mutations have happened in the patients. What it does **NOT** tell us is the chronological order of these mutations. For example, let us consider a patient with a genotype $11\|00\|1$. This genotype tells us that mutation 1 has happened to both the primary tumor and the metastasis. What it does not tell us is the specific order of mutation 1. It could come from either of the three scenarios. 

1. Mutation 1 happened before the seeding event, then the seeding event happened 

2. The seeding event happened, then mutation 1 happened to the primary tumor, and then mutation 1 happened to the metastasis. 

3. The seeding event happened, then mutation 1 happened to the metastasis, and finally mutation 1 happened to the primary tumor.
   
   
For simplification, we refer to the set $V$ as the set formed by all mutations that have happened as indicated by the bitstring. We then refer to the chronologically known scenarios corresponding to set $V$ as **sequences** and denote them as $\sigma$. For example, the genotype $11\|00\|1$ indicates that set $V$ is the set formed by {mutation 1 in the primary tumor, mutation 1 in the metastasis, the seeding event}, and there are three sequences corresponding to this set $V$, $\sigma_1$, $\sigma_2$, $\sigma_3$ corresponding to the three scenarios above. 

The probability of observing set $V$ is a sum of the probability of observing its corresponding sequences as shown below. 

$$p(V;\theta) = \sum_{i} p(\sigma_i;\theta)$$

$\theta$ represents the model parameters of Sampling-MetMHN. Then the log-derivative of $p(V;\theta)$ is given by the following equation.

$$\nabla_\theta\log p(V;\theta) = \frac{1}{p(V;\theta)}\nabla_\theta p(V;\theta) = \frac{1}{p(V;\theta)}\sum_{i}\nabla_\theta p(\sigma_i;\theta) = \sum_{i} \frac{p(\sigma_i;\theta)}{p(V;\theta)}\nabla_\theta \log p(\sigma_i;\theta)$$

The probability of observing a chronologically known sequence $\sigma_i$ and its log derivative are already given by Gotovos et al.

The problem of calculating the log-derivative using the above equation is the number of sequences corresponding to a set $V$. For a set of size $\|V\|$, we have to sum over $\|V\|!$ terms! This is feasible for only very small set sizes. However, the last term of the above equation suggests that the log derivative of $p(V;\theta)$ is a sum of the log derivative of every possible sequence, weighted by the probability of observing that sequence given set $V$. Based on this observation, a stochastic approximation to the above equation can be done by first sample $M$ sequences $\sigma_1, \sigma_2,..., \sigma_M$ from distribution $p(\cdot\|V;\theta) = \frac{p(\cdot;\theta)}{p(V;\theta)}$, and then calculates the approximated log derivative of $p(V;\theta)$ by 

$$\nabla_\theta \log p(V;\theta) \approx \frac{1}{M} \sum_{i=1}^M \nabla_\theta \log p(\sigma_i;\theta)$$

We used the Metropolis-Hasting algorithm to sample from the distribution $p(\cdot\|S;\theta)$. In this algorithm, at every time step, we first propose a new sequence $\sigma_{new}$ from some proposal distribution $Q$ based on our current sequence $\sigma$ and $\theta$. Then we make a transition to $\sigma_{new}$ with probability $p_{accept}$ given by 

$$p_{accept} = \min \left(1, \frac{p(\sigma_{new}|V;\theta)Q(\sigma|\sigma_{new};\theta)}{p(\sigma|V;\theta)Q(\sigma_{new}|\sigma;\theta)}\right)$$

With this Metropolis-Hasting sampler, we can calculate the log-derivative of Sampling-MetMHN efficiently.

Results
========

Since we are only able to calculate the log derivative of Sampling-MetMHN, we implemented an AdaGrad optimizer with L1 penalty for Sampling-MetMHN and used that optimizer to learn the final output -- the $\theta$ matrix. 

## Performance 

Sampling-MetMHN was implemented using C++ and we ran Sampling-MetMHN with different numbers of genomic events on a 16-inch 2019 MacBook Pro with a 2.3 GHz 8-core i9 processor. For each run, 59 data points were generated using the Gillespie algorithm, and for each data point, 50 samples were drawn to calculate its log derivative. We ran 500 iterations of AdaGrad for each run. The time to finish 500 iterations of AdaGrad is documented in the table below. 


| n               | 5  | 10  | 15  | 20  | 25  |
|-----------------|----|-----|-----|-----|-----|
| Time in seconsd | 28 | 106 | 230 | 280 | 430 |


## Simulation 

We tested whether Sampling-MetMHN is able to recover the distribution of the state space $S$ given the input data $\mathcal{D}$, 

In this simulation test, we modeled $n = 8$ genomic events. We randomly filled 90% of the off-diagonal entries with standard log-normal distribution and the other 10% with 0. We then sampled from a standard normal distribution to fill the diagonal entries. To simulate the passenger mutations, we also included two additional independent events where the two events do not interact with all other 8 genomic events and have only their base rates. 

With this ground truth $\theta$ matrix, we again used the Gillespie algorithm and generated 800 samples as the input dataset $\mathcal{D}$ of Sampling-MetMHN. We then trained a Sampling-MetMHN using these 800 samples. The output matrix of Sampling-MetMHN is denoted $\hat{\theta}$.

To compare the distribution of state space $S$ given the dataset $\mathcal{D}$, we used the same Gillespie algorithm and generated 50,000 sample genotypes for both the ground truth theta ($\theta$) and the inferred theta $\hat{\theta}$. Then, for both $\theta$ and $\hat{\theta}$, we calculated the frequency of every possible genotype for $n=10$ events using these $50,000$ samples. The result $freq_\theta$ and ${freq}_{\hat{\theta}}$ are summarized in the table below. 

| genotype                     | 00\|00\|...\|00\|0 | 00\|00\|...\|00\|1 | ...  | 11\|11\|...\|11\|1 |
|------------------------------|--------------------|--------------------|------|--------------------|
| ${freq}_\theta$         | 0.024              | 1.19e-5            | .... | 0.0032             |
| ${freq}_{\hat{\theta}}$ | 0.015              | 2e-6               | ...  | 0.00577            |


We then move on to plot the direct difference between each entry, i.e. 
$d_i = freq_\theta - freq_{\hat\theta}$, where $i \in$ {all possible genotype}. As can be seen from the figure below, the maximum difference between the two distributions is below 0.020.

![](https://github.com/ChenxiNie/ChenxiNie.github.io/blob/master/images/simulation_distribution_comparison.png?raw=true)

## Real Dataset

We then applied Sampling-MetMHN to a pancreatic cancer dataset with 35 genomic events. This dataset contains 76 patients where both their primary tumors and metastases are sequenced, 123 patients who did not suffer from metastases, and 998 patients who had metastasis, however, only the primary tumor is sequenced and the genotype of the metastasis is unknown. For simplicity, we call the 76 patients with both tumors sequenced, *paired patients*, and 123 patients with only primary tumor, *primary only patients*. 

In the perfect setting, we would also be able to obtain the 998 patients' metastasis genotype, making in total 1074 *paired patients*. However, since Sampling-MetMHN requires the genotypes of both tumors, we can only include 76 *paired patients* in our real dataset. This means that we have a biased dataset where the ratio between *paired patients* and *priamry only patients* is skewed and *paired patients* are massively underrepresented. To counteract this skewed ratio, we introduce a weight of $998 / (998 + 123) = 0.89$ for every *paired patient* and a weight of $1 - 0.89 = 0.11$ for every *primary only patient* when calculating the log derivative. 

The output of Sampling-MetMHN is shown below. The most important observation here is that Sampling-MetMHN is able to recover the underlying data structure of this dataset. 

In the picture below, we see a mutual exclusive effect of the genomic event denoted as "Mut.KRAS" (single nucleotide variant (SNV) in KRAS) and "Mut.MEN1" (SNV in MEN1). This observation reassures us that we have correctly captured the underlying structure of this dataset. The dataset is constructed using two subtypes of pancreatic cancers, pancreatic adenocarcinomas (PAAD) and pancreatic neuroendocrine tumors (PANET). These two subtypes of pancreatic cancers originated from different cell types found in the pancreas, making their genomic profiles distinct from each other. Pancreatic adenocarcinomas are driven by mutations in gene KRAS and pancreatic neuroendocrine tumors are characterized by mutations in genes MEN1, DAXX, and ATRX. These two distinct genomic profiles mean that mutations of KRAS would not have any selective advantage in PANET while mutations in MEN1 give no selective advantage to PAAD. This is correctly captured by the mutual exclusive effect of KRAS and MEN1. Moreover, the promotion of DAXX and ATRX by MEN1 is observed. (See entry (Mut.DAXX, Mut.MEN1) and entry (Mut.ATRX, Mut.MEN1))

![](https://github.com/ChenxiNie/ChenxiNie.github.io/blob/master/images/theta_real.png?raw=true)

Discussion
==========

## Identifiability Problem

In the figure below, we show the output $\hat\theta$ from the simulation study on the left and the ground truth $\theta$ on the right. As can be seen from the figure, although we are able to recover the distribution $S$, there is a discrepancy between the inferred theta ($\hat\theta$) and the ground truth theta ($\theta$). This identifiability issue means that interpreting the inter genomic effects inferred by Sampling-MetMHN causally should only be done with caution. 

The identifiability problem is common among the MHN class of models and is not limited to Sampling-MetMHN. Solving this identifiability problem requires further research.

![](https://github.com/ChenxiNie/ChenxiNie.github.io/blob/master/images/Identifiability_issue.png?raw=true)

## Data availability 

Although much effort has been poured into cancer genomics, the paired samples of both primary tumors and metastases remain sparse. In our dataset, only 76 patients have both primary tumor and metastasis sequenced, while there are additionally 998 patients with both primary tumors and metastasis but only the primary tumor got sequenced. Should we add these 998 patients to the dataset, we would be more confident about our model output. 

## Simplifying assumptions

To fit the metastatic process into the MHN frameworks, we have to make some simplifying assumptions.

First, we ignored intratumor heterogeneity. In this model, we represent each tumor using only one genotype. However, in reality, distinct tumor cell populations within the same specimen are of great importance and contribute to therapeutic resistance, recurrence, as well as metastatic seeding. 

Second, after a successful seeding event, the primary tumor and the metastasis are assumed to be independent of each other. In reality, the primary tumor is not only the source of metastatic cells but also modulates host responses to these cells, leading to enhancement or inhibition of metastasis.

Conclusion
==========

In this work, we propose a fast approximate MHN framework that models both primary tumors and metastases. It shares a similar mathematical framework as the original MHN but modifies the underlying CTMC to account for the independent evolution of primary tumor and metastasis after the seeding event. To the best of our knowledge, Sampling-MetMHN is one of the first cancer progression models that manages to model the genetic progression of primary tumors and metastases simultaneously.

Key References
==========
* The original MHN paper: Rudolf Schill, Stefan Solbrig, Tilo Wettig, Rainer Spang, Modelling cancer progression using Mutual Hazard Networks, Bioinformatics, Volume 36, Issue 1, January 2020, Pages 241–249, https://doi.org/10.1093/bioinformatics/btz513

* Gotovos' MCMC sampler for the original MHN: Gotovos, A., Burkholz, R., Quackenbush, J., and Jegelka, S. (2021). Scaling up continuous-time markov chains helps resolve underspecification. Advances in Neural Information Processing Systems, 34:14580–14592.


