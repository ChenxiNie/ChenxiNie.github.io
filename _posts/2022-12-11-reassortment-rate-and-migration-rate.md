---
title: 'Exploring reassortment rate and migration rate in segmented viruses'
date: 2022-12-11
permalink: /posts/2022/12/reassortment-rate-and-migration-rate/
tags:
  - Phylogenetic Models
  - Segmented Viruses 
  - Simulation Algorithms
---

In this lab rotation project, we set out to verify whether we could verify the conclusion that reassortment events are one of the main drivers for host jump events using a state-of-the-art phylogenetic model SCoRe.

Introduction
============

The genetic variation of segmented viruses is promoted by reassortment. Reassortment happens during coinfection, where different viral strains exchange segments to produce a viral progeny that contains segments from more than one parent. Additionally, influenza viruses are known to switch host species frequently, meaning that apart from reassortment, migration between host species is not uncommon. It is believed that reassortment events are one of the main drivers for host jump events. However, previously due to a lack of mathematical frameworks that take both reassortment and migration events into account, this has not yet been verified in statistical models.

Recently, SCoRe -- a model that jointly infers the migration and reassortment patterns for segmented viruses was proposed. When evaluating the model, it is suggested that there is a slight increase in reassortment rates before migration events. However, the 95% highest posterior density interval still includes no increase before migration events. During the discussion, the authors suggested that datasets with more distantly related species and span a longer time window could help improve the precision of these estimates. 

In this short lab rotation project, we developed a simulation algorithm where before each migration event, the reassortment rate is scaled by a constant scalar. This simulation algorithm would provide an assessment dataset needed to verify whether SCoRe is able to recover the elevated reassortment rate before migration events.

Simulation Algorithm
===================

Here we first formally describe an existing algorithm that does not scale the reassortment rate. Then we introduce some key changes to that algorithm so that the reassortment rate can be scaled after each migration event within a period of time.

## Structure Coalescent Network Simulation 

The generation of SCoRe networks can be modeled as a continuous backward-in-time Markov Chain (CTMC), that is characterized by *coalescent*, *migration*, *reassortment*, and *sampling* events, as shown below. To simulate this CTMC, an adaptation of the Gillespie algorithm was developed.

![](https://github.com/ChenxiNie/ChenxiNie.github.io/blob/master/images/Four%20Kinds%20of%20events.png?raw=true)

The first input of this algorithm is a list $L$, representing $n$ initial samples. Assume we are modeling $m$ types and each type is represented by a positive integer from $1$ to $m$, then the $i$-th lineage $l$ in the list can be defined by its type $I \in$ {$1, ..., m$} and the ancestral segments $C(l)$ it carries at time $t_i$ (i.e. $l = [I, C(l)]$). We also assume that all lineages in this list carry a full set of genomic segments. Additionally, we require the sampling time $t_i$ for each of the lineages in the list. 

Since we only allow for coalescent events between lineage $l$ and $l'$ if they are in the same type, the second input of the algorithm would be the pairwise coalescent rate for each type $\lambda_a$, where $a\in$ {$1, ..., m$}. The parent lineage of a coalescent event $l_p$ carries ancestral segments of both child lineages and shares their type, i.e. $l_p=[a, C(l)\cup C(l')]$. If $k_a$ is the number of lineages in type $a$, then the total coalescent rate of that type $a$ is $C_a = \lambda_a \binom{k_a}{2}$. Then since SCoRe is a CTMC, the time until the next coalescent event of type $a$ can be modeled by an exponential distribution with $C_a$.

The third input of the algorithm belongs to migration rate $\mu_{ab}$, measured per lineage and per time unit. It describes the rate at which one lineage of type $a$, i.e. $l_1 = [a, C(l_1)]$ migrates to another type b, i.e. $l_1'=[b, C(l_1)]$. The time to the next migration event where one lineage of type $a$ migrates to type $b$ is then modeled by an exponential distribution with $\mu_{ab}k_a$. Migration events do not change the segments that lineage has. 

When reassortment events happen, ancestral segments carried by the child lineage are split between its two parents $l_{p1}$ and $l_{p2}$. The two parents share the same type as the child lineage while each segment of the child lineage is assigned to $l_{p1}$ with probability $p$ and assigned to $l_{p2}$ with probability $1-p$. A reassortment event is unobservable when all the segments from the child lineage are assigned to one of its parents. Since SCoRe assumes $p = \frac{1}{2}$, this unobservable reassortment happens with probability $2 \times \left(\frac{1}{2}\right)^{\|C(l)\|}$, where $\|C(l)\|$ is the number of ancestral segments carried by the child lineage $l$. Reassortment rate $\rho$ of each type makes the fourth input of our simulation algorithm. Then the time to the next reassortment event of type $a$ can be modeled by an exponential distribution with $\rho_ak_a$.

With the introduction of the four inputs, we can describe formally the structure coalescent network simulation algorithm in the algorithm below. 

![](https://github.com/ChenxiNie/ChenxiNie.github.io/blob/master/images/Structure%20Coalescent%20Network%20Simulation%20Algorithm.png?raw=true)

## Structure Coalescent Network Simulation With Time Window

Based on the existing structure coalescent network simulation algorithm, we developed a simulation algorithm that shares the same logic but is capable of scaling the reassortment rate after each migration event (going backward in time).

The key change we made was the introduction of the "time window". Assume we observed a migration event from type $a$ to type $b$ on lineage $l$. We then define a period of time called a time window. During this time window, the reassortment rate of that lineage $l$ is scaled by a constant $S$. With the introduction of the time window, we update the definition of lineage $l$ from $l_i = [I, C(l)]$ to $l_i = [I, C(l),W_i]$ where $W_i$ is an indicator of whether this lineage is in a time window or not and if it is in a time window, $W_i$ then indicates when will this time window end. 

$$
W_i = 
\begin{cases}
-1 & \text{If } l_i \notin \text{time Window} \\
\text{end time} & \text{If } l_i \in \text{time Window}
\end{cases}
$$

Since at the sampling time, no migration event can happen, we set the initial time window $W_i$ to -1 by default and after a migration event at time $t$, we update $l_i$'s $W_i$ to $t + W_i$. 

With the introduction of the time window, we have to modify how to calculate the time until the next reassortment event. Instead of directly using $\rho_ak_a$, we have to divide $k_a$ into two parts $k_{a1}$ and $k_{a2}$ where $k_{a1}$ is the number of lineages of type $a$ that is not in a time window, and $k_{a2} = k_a - k_{a1}$ is the number of lineages of type $a$ that is currently inside a time window. Now, we then model the time until the next reassortment event using an exponential distribution of rate $\rho_ak_{a1} + \rho_aS_ak_{a2}$ where $S_a$ is the reassortment scalar of type $a$ when its lineage is in a time window. Since we do not scale migration and coalescent rate, the time until the next migration event and the time until the next coalescent event are modeled using the same distribution described above. Updated rules regarding the parent(s) of these three kinds of events are described using an example of 3 lineages shown in the figure below. 

![](https://github.com/ChenxiNie/ChenxiNie.github.io/blob/master/images/Simulation_with_Time_Window_2.jpg?raw=true)

Going backward in time we have:

1. At $t=0$, there are two samples, one human $l_1=[H, -1]$ and one avian $l_2[A, -1]$. Since we did not change the modeling of ancestral segments, $C{(l_i)}$ is omitted here for simplicity. 
2. At $t1$, the human sample migrated to avian. Thus we have to change its type from Human to Avian and then we open a time window of length $w$ and update the time window for this sample. This means we update $l_1=[H,-1]$ to $l_3=[A, t1 + w]$. We then keep track of this lineage $l_3$ and scale its reassortment rate by a constant factor $S$.
3. At $t2$, a coalescent event happened between $l_2$ and $l_3$. Here, we define that if one child lineage in the coalescent event is in a time window, then the parent lineage is also in a time window and the time window of that parent lineage is $\max(w_1,w_2)$ where $w_1$ and $w_2$ are the time windows of the two children. With that, the parent of $l_2$ and $l_3$ is a lineage with type $A$ and time window $\max(t1+w, -1) = t1+w$, i.e. $l_4 = [A, t1+w]$.
4. At $t3$, a sampling event happened and another human sample $l_5 = [H, -1]$ was introduced. 
5. At $t4$, a reassortment event happened. For reassortment events, we define that the two parent lineage shares the same time window as the child lineage. Here, lineage $l_4 = [A, t1+w]$, so its two parents $l_6$ and $l_7$ are both type A and have their time windows set to $t1+w$. 
6. At $t5$, another reassortment event happened, but this time, lineage $l_5$ is not in a time window. So, the parents of $l_5$, $l_8$ and $l_9$ are not in the time window as well. 
7. At $t6$, lineage $l8$ migrated to Avian. We have to open another time window and track lineage $l_{10}$ which has a time window of $t6+w$. 
8. At $t_1+w$, the first time window reaches an end and we move all lineages that were previously in the time window out of the time window. In this case, we set $l_6$ and $l_7$'s time windows back to -1.
9. At $t7$, another coalescent event happened, based on the definition we gave at time $t2$, we know that here lineage $l_{11}$ has type $A$ and time window $t6+w$.  

## Correctness of the algorithm

To verify the correctness of our simulation algorithm with time window, we compared our simulation algorithm to the original structure coalescent network simulation algorithm. In order to make sure that our algorithm behaves the same as the algorithm without time windows, we set the scalar constant to 1 (i.e. we do not scale the reassortment rate even if the lineage is in a time window). 

We then simulated 100,000 networks with 100 samples and 2 types. We used the same sampling time, coalescent rate, migration rate, and reassortment rate for the two algorithms and set the scalar of the time window of both types to 1. We then plotted the distribution of the network height, the total network length, the reassortment events count, and the Kolmogorov-Smirnov statistics for the above three statistics in the figure below. 


