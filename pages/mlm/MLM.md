---
title: Multilevel Models
keywords: 
last_updated: Sept 28, 2016
tags: 
summary: "MLMs (also known as hierarchical linear models, nested data models, mixed models, random coefficient, random-effects models, random parameter models, or split-plot designs) are statistical models of parameters that vary at more than one level."
sidebar: main_sidebar
permalink: MLM.html
folder: mlm
---

## What are Multilevel Models?

Multilevel models (also known as hierarchical linear models, nested data models, mixed models, random coefficient, random-effects models, random parameter models, or split-plot designs) are statistical models of parameters that vary at more than one level. An example could be a model of student performance that contains measures for individual students as well as measures for classrooms within which the students are grouped. These models can be seen as generalizations of linear models (in particular, linear regression), although they can also extend to non-linear models. These models became much more popular after sufficient computing power and software became available.


\\[ \mathbf{X} = \mathbf{Z} \mathbf{P^\mathsf{T}} \\]





Multilevel models are particularly appropriate for research designs where data for participants are organized at more than one level (i.e., nested data). The units of analysis are usually individuals (at a lower level) who are nested within contextual/aggregate units (at a higher level). While the lowest level of data in multilevel models is usually an individual, repeated measurements of individuals may also be examined. As such, multilevel models provide an alternative type of analysis for univariate or multivariate analysis of repeated measures. Individual differences in growth curves may be examined (see growth model). Furthermore, multilevel models can be used as an alternative to ANCOVA, where scores on the dependent variable are adjusted for covariates (i.e., individual differences) before testing treatment differences. Multilevel models are able to analyze these experiments without the assumptions of homogeneity-of-regression slopes that is required by ANCOVA.

Multilevel models can be used on data with many levels, although 2-level models are the most common and the rest of this article deals only with these. The dependent variable must be examined at the lowest level of analysis


