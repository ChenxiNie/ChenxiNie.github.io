---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

Education
======
* B.E. in Bioinformatics, Harbin Institute of Technology, September 2016 - June 2020
* M.S. in Computational Biology and Bioinformatics, Swiss Federal Institute of Technology Zurich, September 2020 - September 2023
* Ph.D (actively looking for a Ph.D program in the field of statistic modelling in bioinformatics)

Projects
======
* March 2022 - July 2022: Long semester project at ETHz. 
  * Project title: Modelling metastatic progression using metMHN. 
  * Duties included: Designed metMHN, a cancer progression model that models both the primary tumors and the metastases' genetic progression. Developed a tensor acceleration to metMHN and implemented the model in R. Condaucted model valisation using simulations and in real-world datasets.
  * Supervisor: 
    * Professor: Prof.Dr. Niko **Beerenwinkel**
    * Ph.D candidate supervisor: Kevin **Rupp**

* March 2023 - September 2023: Master thesis project at ETHz
  * Project title: Scaling up MetMHN: a sampling based approach. 
  * Duties included: Developed a MCMC based sampling algorithm for metMHN which speed it up 100 times. Efficient implementation of Sampling-MetMHN in C++. Model validation using simulations and real-world datasets on a larger scale. 
  * Supervisor: 
    * Professor: Prof.Dr. Niko **Beerenwinkel**
    * Ph.D candidate supervisor: Kevin **Rupp**
  
* September 2022 - November 2022: Short semester project at ETHz.
  * Project title: SCoRe validation -- elevated reassortment rate before migration events. 
  * Duties included: Understand SCoRe and BEAST software. Proposed and implemented simulation algorithms to verify whether reassortment rate is elevated before host jump events using R and Java. Search for biological literature about viral reassortment rates and host jump events. 
  * Supervisor:
    * Professor: Prof.Dr. Tanja **Stadler**
    * Ph.D candidate supervisor: Ugne **Stolz**

* November 2019 - June 2020: Bachelor thesis at Harbin Institute of Technology
  * Project title: Structural Variation Detection Algorithm Based on Population Sequencing
  * Duties included: Designed a structural variation calling algorithm that utilizes existing structural variations in population databases. Implemented the model using Python and tested performace using precision and recall. 
  * Supervisor: Prof.Dr. **Liu** Bo
  
Skills
======
* Skill 1
* Skill 2
  * Sub-skill 2.1
  * Sub-skill 2.2
  * Sub-skill 2.3
* Skill 3

Publications
======
  <ul>{% for post in site.publications %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
  
Talks
======
  <ul>{% for post in site.talks %}
    {% include archive-single-talk-cv.html %}
  {% endfor %}</ul>
  
Teaching
======
  <ul>{% for post in site.teaching %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
  
Service and leadership
======
* Currently signed in to 43 different slack teams
