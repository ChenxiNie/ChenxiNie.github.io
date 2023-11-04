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





