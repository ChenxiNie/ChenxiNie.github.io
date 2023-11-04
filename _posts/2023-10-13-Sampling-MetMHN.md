---
title: 'Modeling Metastatic Progression With Sampling-MetMHN'
date: 2023-10-13
permalink: /posts/2023/10/Sampling-MetMHN/
tags:
  - Cancer progression models
  - Metastatisis 
  - MCMC Sampling
---


We're excited to introduce **Sampling-MetMHN**, a fast approximate MHN[1] framework that models both primary tumors and metastases. 

To the best of our of our knowledge, it is one of the first cancer progression models that manages to model the genetic progression of primary tumors and metastases simultaneously. We show that it is able to recover some of the key findings documented in pancreatic cancer literature as well as promote potentially new genomic interactions. 

Metastatic cancer progression 
=================

The term metastatic cancer refers to the systematic spread of cancer from the initial tumor site to different areas of the body[2]. We often refer to the tumor in the initial tumor site as the primary tumor and the tumor that colonized in other areas as metastasis or secondary tumor. 

Metastatic cancer progression is highly selective. In cancer patients, large numbers of cancer cells are released into the circulation system on a daily bases. However, only less than 0.1% of tumor cells establish metastatic secondary tumors[3]. 

Apart from highly selective, metastatic cancer progression is organ-specific as well. This organ-specific aspect of metastatic progression is captured by the famous seed and soil hypothesys by Paget in 1889[4]. Paget noticed that in nature, when plants go to seed, their seeds are carried in all directions, but the seed can only grow if it lands on congenial soil. Drawing an analogy from this observation, the seed and soil hypothesis suggests that cancer cells from the primary tumor (the seed) can spread to different parts of the body. However, these cancer cells can only suceed in colonizing if the microenvironment on which they land is pleasant. 

Existing models of metastatic cancer progression
=======

Existing models of metastatic cancer progression can be categorized into two classes. The first category models metastatic cancer progression as a growth model and these models do not focus on the genomic side of metastatic cancer progression. The second category is genomic models, where the interactions of genomic events are considered. One particular subtype of genomic models is the logical models where they utilize regulartory networks that are constructed from published literature. These regulartory networks describes the interactions between nodes of the model (genomic events). Using these regulartory networks, these models define a list of specific logical rules for each genomic event. Although these models are able to model interactions between genomic events, they face scalability issues. If the model needs to be expanded to include additional events, we would have to consider updating every one of the logical rules manually. Moreover, potentially essential interactions between nodes can be missed due to a lack of literature support. 

In an effort to overcome the drawbacks of the existing models, we propose Sampling-MetMHN, a genomic model that builds on existing cancer progression models developed for primary tumors but adapted to be able to model both the primary tumors and metastases. It is both scalable and able to model the interactions of genomic events. 

MetMHN: A cancer progression model that works on metastasis
==================

## The base: Mutual Hazard Networks

Sampling-MetMHN is based on Mutual Hazard Networks (MHN). MHN is a state-of-the-art cancer progression model that models the progression of one single tumor. It models tumor progression as a continuous time Markov process. It is assumed that every tumor starts from a mutation-free state and evolves according to a Markov Chain until it reaches a state where all of the genomic events have happened. 

![Alt text](https://github.com/ChenxiNie/ChenxiNie.github.io/blob/master/images/Markov_Chain_MHN.png?raw=true)

In an MHN with $n$ genomic events, each datapoint is encoded as a bitstring of length $n$. The $i$=th bit encodes the presence (1) or the absence (0) of the $i$-th event. Table below illustrate an example datapoint in MHN where mutations 1, 2, and 5 are present and mutations 3 and 4 are absent. 

| Mut 1 | Mut 2 | Mut 3 | Mut 4 | Mut 5 | ... |
|-------|-------|-------|-------|-------|-----|
| 1     | 1     | 0     | 0     | 1     | ... |


## From MHN to MetMHN: Enabling metastasis in MHN framework 

Metastatic cancer typically starts with one primary tumor and then the primary tumor goes through a series of steps to form an initial metastatic tumor at a distant site. Based on this observation, we define the seeding event as a representation of the steps that the primary tumor takes to successfully form an initial metastatic tumor. We assume that prior to the successful seeding event, the primary tumor and the (potential) metastasis evolve jointly as one physical entity and every mutation that occurs in the primary tumor also occurs in the (potential) metastasis. Once the seeding event has happened, the primary tumor and the metastasis are physically separated and are assumed to eveolve independently. 

![](https://raw.githubusercontent.com/ChenxiNie/ChenxiNie.github.io/e5bb3732515d30968d809134d23e935deacbc84d/images/CTMC_MetMHN.svg)

Similar to MHN, in MetMHN, each datapoint is encoded as a bitstring as well. However, to include both the primary tumor and the metastasis, we have to double the length of the bitstring. Additionally, we add one bit for the seeding event. Thus, in MetMHN with $n$ events, the final length of that datapoint's bitstring is $2n+1$. 


![](https://raw.githubusercontent.com/ChenxiNie/ChenxiNie.github.io/60d391ce993db4d5d691326e965f53efebe33b02/images/Met_MHN_State_Space.svg)

## From MetMHN to Sampling-MetMHN: a Markov Chain Monte Carlo Sampler for MetMHN 

When using first-order optimization algorithms, the majority of time is devoted to calculating the gradient of marginal log-likelihood with respect to parameters. Here, we propose a MCMC based sampling algorithm to calculate the derivative of MetMHN efficiently. The idea of this method  closely follows the work from Gotovos et al. in which they deveoped a similar sampling algorithm for the original MHN[5]. 

The input of Sampling-MetMHN is a collection of bitstrings representing each patient's genotypes. These bitstrings tell us about which mutations have happened in the patients. What is does **not** tell us is the chronological order of these mutations. For example, let us consider a patient with genotype $11\|00\|1$. This genotype tells us that mutation 1 has happened to both the primary tumor and the metastasis. What it does not tell us is the specific order of mutation 1. It could come from either of the three scenarios. 

1. Mutation 1 happened before the seeding event, then the seeding event happened 

2. The seeding event happened, then mutation 1 happened to the primary tumor, and then mutation 1 happened to the metastasis. 

3. The seeding event happened, then mutation 1 happened to the metastasis, finally mutation 1 happened to the primary tumor.

For similification, we refer the set $V$ as the set formed by all mutations that has happened as indicated by the bitstring.We then refer to the chronologically known scenarios corresponding to set $V$ as **sequences** and denote them as $\sigma$. For example, the genotype $11\|00\|1$ indicates that set $V$ is the set formed by $\{$ mutation 1 in the primary tumor, mutation 1 in the metastasis, the seeding event $\}$, and there are three sequences correponding to this set $V$, $\sigma_1$, $\sigma_2$, $\sigma_3$ corresponding to the three scenarios above. 

It is obvious that the probability of observing set $V$ is a sum over the probability of observing its corresponding sequences as shown below. 

$$p(V;\theta) = \sum_{i} p(\sigma_i;\theta)$$

$\theta$ represents the model parameters of Sampling-MetMHN. Then the log-derivative of $p(V;\theta)$ is given by the following equation.

$$\nabla_\theta\log p(V;\theta) = \frac{1}{p(V;\theta)}\nabla_\theta p(V;\theta) = \frac{1}{p(V;\theta)}\sum_{i}\nabla_\theta p(\sigma_i;\theta) = \sum_{i} \frac{p(\sigma_i;\theta)}{p(V;\theta)}\nabla_\theta \log p(\sigma;\theta)$$

The probability of observing a chronologically known sequence $\sigma_i$ and its log derivative are already given by Gotovos et al[5].

The problem of calculating the log-derivative using the above equation is the number of sequences corresponding to a set $V$. For a set of size $\|V\|$, we have to sum over $\|V\|!$ terms! This is feasible for only very small set size. However, the last term of the above equation suggests that the log-derivaitve of $p(V;\theta)$ is a sum of the log derivative of every possible sequence, weighted by probability of observating that sequence given set $V$. Based on this observation, a stochastic approximation to the above equation can be done by first sample $M$ sequences $\sigma_1, \sigma_2,..., \sigma_M$ from distribution $p(\cdot\|V;\theta) = \frac{p(\cdot;\theta)}{p(V;\theta)}$, and then calculates the approximated log derivative of $p(V;\theta)$ by 

$$\nabla_\theta \log p(V;\theta) \approx \frac{1}{M} \sum_{i=1}^M \nabla_\theta \log p(\sigma_i;\theta)$$

We used Metropolis-Hasting algorithm to sample from distribution $p(\cdot\|S;\theta)$ [6][7]. In this algorithm, at every time step, we first propose a new sequence $\sigma_{new}$ from some proposal distribution $Q$ based on our current sequence $\sigma$ and $\theta$. Then we make a transition to $\sigma_{new}$ with probability $p_{accept}$ given by 

$$p_{accept} = \min \left(1, \frac{p(\sigma_{new}|V;\theta)Q(\sigma|\sigma_{new};\theta)}{p(\sigma|V;\theta)Q(\sigma_{new}|\sigma;\theta)}\right)$$

With this Metropolis-Hasting sampler, we can calculate the log-derivative of Sampling-MetMHN efficiently.

Results
========

Since we are only able to calcualte the log-derivative of Sampling-MetMHN, we implemented an AdaGrad optimizer with L1 penalty for Sampling-MetMHN and used that optimizer to learn the final output -- the $\theta$ matrix. 

## Performance 

Sampling-MetMHN was implemented using C++ and we ran Sampling-MetMHN with different numbers of genomic events on a 16 inch 2019 MacBook Pro with a 2.3 GHz 8-core i9 processor. For each run, 59 data points were generated using the Gillespie algorithm, and for each data point, 50 samples were drawn to calculate its log-derivative. We ran 500 iterations of AdaGrad for each run. The time to finish 500 iterations of AdaGrad is documented in the table below. 


| n               | 5  | 10  | 15  | 20  | 25  |
|-----------------|----|-----|-----|-----|-----|
| Time in seconsd | 28 | 106 | 230 | 280 | 430 |


## Simulation 

We tested whether Sampling-MetMHN is able to recover the distribution of the state space $S$ given the input data $\mathcal{D}$, 

In this simulation test, we modeled $n = 8$ genomic events. We randomly filled 90% of the off-diagonal entries with standard log-normal distribution and the other 10% with 0. We then sampled from a standard normal distribution to fill the diagonal entries. To simulate the passenger mutations, we also included two additional independent event where the two events do not interact with all other 8 genomic events and have only their base rates. 

With this groud truth $\theta$ matrix, we again used the Gillespie algorithm and generated 800 samples as the input dataset $\mathcal{D}$ of Sampling-MetMHN. We then trained a MetMHN using these 800 samples. The output matrix of Sampling-MetMHN is denoted $\hat{\theta}$.

To compare the distribution of state space $S$ given the dataset $\mathcal{D}$, we used the same Gillespie algorithm and generated 50,000 sample genotypes for bothe the ground truth theta ($\theta$) and the inferred theta $\hat{\theta}$.Then, for both $\theta$ and $\hat{\theta}$, we calculated the frequency of every possible genotype for $n=10$ events using these $50,000$ samples. The result $freq_\theta$ and ${freq}_{\hat{\theta}}$ are summarized in the table below. 

| genotype                     | 00\|00\|...\|00\|0 | 00\|00\|...\|00\|1 | ...  | 11\|11\|...\|11\|1 |
|------------------------------|--------------------|--------------------|------|--------------------|
| ${freq}_\theta$         | 0.024              | 1.19e-5            | .... | 0.0032             |
| ${freq}_{\hat{\theta}}$ | 0.015              | 2e-6               | ...  | 0.00577            |


We then move on to plot the direct difference between each entry, i.e. 
$d_i = freq_\theta - freq_{\hat\theta}$, where $i \in$ {all possible genotype}.As can be seen from the figure below, the maximum difference between the two distributions is below 0.020.

![](https://github.com/ChenxiNie/ChenxiNie.github.io/blob/master/images/simulation_distribution_comparison.png?raw=true)

## Real Dataset

We then applied Sampling-MetMHN to a pancreatic cancer dataset with 35 genomic events. This dataset contains 76 patients where both their primary tumors and metastases are sequenced, 123 patients who did not suffer from metastases, and 998 patients who had metastasis, however, only the primary tumor is sequenced and the genotype of the metastasis is unknown. For simplicity, we call the 76 patients with both tumors sequenced, *paired patients*, and 123 patients with only primary tumor, *primary only patient*. 

In the perfect setting, we would also be able to obtain the 998 patients' metastasis genotype, making in total 1074 *paired patients*. However, since Sampling-MetMHN requires the genotypes of both tumors, we can only include 76 *paired patients* in our real dataset. This means that we have a biased dataset where the ratio between *paired patients* and *priamry only* patients is skewed and *paired patients* are massively underrepresented. To counteract this skewed ratio, we introduce a weight of $998 / (998 + 123) = 0.89$ for every *paired partient* and a weight of $1 - 0.89 = 0.11$ for every *primary only patient* when calculating the log derivative. 


References
==========
[1] Rudolf Schill, Stefan Solbrig, Tilo Wettig, Rainer Spang, Modelling cancer progression using Mutual Hazard Networks, Bioinformatics, Volume 36, Issue 1, January 2020, Pages 241–249, https://doi.org/10.1093/bioinformatics/btz513

[2] Geiger TR, Peeper DS. Metastasis mechanisms. Biochim Biophys Acta. 2009 Dec;1796(2):293-308. doi: 10.1016/j.bbcan.2009.07.006. Epub 2009 Aug 14. PMID: 19683560.

[3] Luzzi, K. J., MacDonald, I. C., Schmidt, E. E., Kerkvliet, N., Morris, V. L., Chambers, A. F.,
and Groom, A. C. (1998). Multistep nature of metastatic inefficiency: dormancy of solitary
cells after successful extravasation and limited survival of early micrometastases. The American
journal of pathology, 153(3):865–873.

[4] Paget, S. (1889). The distribution of secondary growths in cancer of the breast. The Lancet,
133(3421):571–573.

[5] Gotovos, A., Burkholz, R., Quackenbush, J., and Jegelka, S. (2021). Scaling up continuous-time markov chains helps resolve underspecification. Advances in Neural Information Processing Systems, 34:14580–14592.

[6] Metropolis, N., Rosenbluth, A. W., Rosenbluth, M. N., Teller, A. H., and Teller, E. (1953). Equation of state calculations by fast computing machines. The journal of chemical physics, 21(6):1087–1092.

[7] Hastings, W. K. (1970). Monte carlo sampling methods using markov chains and their applications.
