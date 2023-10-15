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

Metastatic cancer typically starts with one primary tumor and then the primary tumor goes through a series of steps to form an initial metastatic tumor at a distant site. Based on this observation and naming after the seed and soil hypothesis, we define the seeding event as a representation of these steps that the primary tumor takes to successfully form an initial metastatic tumor. We assume that prior to the successful seeding event, the primary tumor and the (potential) metastasis evolve jointly as one physical entity and every mutation that occurs in the primary tumor also occurs in the (potential) metastasis. Once the seeding event has happened, the primary tumor and the metastasis are physically separated and are assumed to eveolve independently. 



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