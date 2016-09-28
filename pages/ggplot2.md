---
title: Data Visualization: Easily Exploring Your Data in R Using ggplot2
keywords: R, ggplot2
last_updated: Sept 28, 2016
tags: [R]
summary: "ggplot2"
sidebar: main_sidebar
permalink: ggplot2.html
folder: ggplot2
---

## Why Use ggplot2 in R?

The first step to any data analysis project should be to explore your data.  The easiest and fastest way to do this is through graphics.  Although you can create plots in many programs including base R, the ggplot2 package is fairly quick to learn and makes life easier in at least ten ways (see Many Mejia’s post with examples)

* Flexibility: It can do quick-and-dirty AND complex plots, so you only need one system
* Handsome default settings: the default colors & other aesthetics are nice…much better than Excel
* You don’t have to think hard: no need to specify outer or inner margins among other things
* You can save your plots (or the beginnings of a plot) as an object
* Multivariate exploration is greatly simplified through faceting and coloring
* Easily build plots in layers to tell a more complete story
* Let your plots evolve (or devolve) with minimal changes to code
* Make layered histograms and other cool plots
* It’s not that hard and takes very little code
* The documentation and support is great


## What you will be able to do after this workshop

* Change the structure of your data frame to make certain plot s possible. The reshape2 packages make it quick to move form “wide form” (i.e. “flat”) to “long form” (also known as a dynaset).
* Create plots for 1, 2, 3, or more variables at a time using the ggplot2
* Use another variable to subset, color, or facet (break into panels) the plot.
* Change or add titles, labels, legends, and themes.
* Save a plot as a pdf, jpeg, png, bmp, or other type of file, including changing the size and resolution.

## Required Prerequisite

Participants should have some basic familiarity in R (R-Studio) and have the software running on the laptop they bring to this workshop.  Completion of the Stat Studio “Intro to R: For Absolute Beginners” workshop held last November is sufficient.  IF you missed it you can find all the materials at http://cehs.usu.edu/research/Stat Studio/intro2r or there are many other similar resources online.  
Download the following packages from CRAN:  ggplot2, dplyr, tidyr, reshape2

{% include links.html %}
