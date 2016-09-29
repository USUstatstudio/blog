---
title: Finch Text Chapter 3 example of 2 level models
keywords: 
last_updated: Sept 29, 2016
tags: 
summary: "testing"
sidebar: main_sidebar
permalink: Finch_ch3_Fit_2_Levels.html
folder: mlm
---

# Knits APA style documents (from GitHub, not CRAN)
    library("papaja")       

    # Pander is the middle-man between R markdown and LaTe
    library("pander")
    panderOptions("table.split.table", Inf)
    panderOptions("round", 2)
    panderOptions("keep.trailing.zeros", TRUE)
    panderOptions("table.emphasize.rownames", FALSE)
    panderOptions("table.alignment.rownames", "left")
    panderOptions("missing", "")

    # work smarter, not harder with the tidyverse
    library("tidyverse")    # loads the CORE packages
    library("magrittr")     # more pipe operators to code easier

    # get data in
    library("haven")        # read in SPSS dataset
    library("labelled")     # deal with factors from SPSS, through haven
    library("readxl")       # for excel files (xls, xlsx)

    # display result tables
    library("furniture")    # nice table1() descriptives - 9/20/16 now on CRAN!
    library("car")          # Companion to Applied Regression
    library("stargazer")    # display nice tables: summary & regression
    library("texreg")       # Convert Regression Output to LaTeX or HTML Tables

    # plots: exploratory, results, and residuals
    library("corrplot")     # correlograms
    library("gpairs")       # generalized pairs plots
    library("GGally")       # extends ggplot2, speciallized plots
    library("ggthemes")     # custom themes for ggplot2
    library("RColorBrewer") # nice color palettes for plots
    library("gridExtra")    # place ggplots together as one plot

    # fit multilevel models, and work with output
    library("nlme")         # non-linear mixed-effects models
    library("lme4")         # Linear, generalized linear, & nonlinear mixed models
    library("lmerTest")     # Tests on lmer objects
    library("HLMdiag")      # Diagnostic Tools for for nlme & lmer4
    library("sjPlot")       # Plotting lme4 results
    library("mlmRev")       # example datasets for multilevel models

<!-- ========================================================= -->
The `Achieve` Dataset
=====================

<!-- ========================================================= -->
The datasets for this textbook may be downloaded from the website:
<http://www.mlminr.com/data-sets/>. I was unable to find any
documentation on this dataset in the book or online, so I contacted the
authors. There were unable to provide much either, but based on visual
inspection designated the class of *factor* to thoes vairables that seem
to represent categorical quantities. The labels for gender and class
size are relative to the frequencies in the journal article the authors
did point me to *(although the samples sizes do not match up)*.

    achieve <- read_sav("Achieve.sav")

    achieve <- achieve %>% 
      mutate(id       = to_factor(id),
             region   = to_factor(region),
             corp     = to_factor(corp),
             school   = to_factor(school),
             class    = to_factor(class),
             gender   = to_factor(gender) %>% 
                           factor(labels = c("Female", "Male")),
             classize = to_factor(classize) %>% 
                           factor(labels = c("12-17", "18-21", 
                                             "22-26", ">26")))

<!-- ========================================================= -->
Sample Structure
----------------

<!-- ========================================================= -->
It is obvious that the sample is hiarchical in nature. The nesting
starts with `students` (level 1) nested within `class` (level 2), which
are further nested within `school` (level 3), `corp` (level 4), and
finally `region` (level 5). For this chapter we will only focus on two
levels: **students** are the units on which the outcome is measured and
**schools** are the units in which they are nested.

    # number of...regions = 9
    num_region <- achieve %>% group_by(region) %>% 
                  summarize(1) %>% nrow
    num_region

    ## [1] 9

    # number of...corps = 60
    num_corp <- achieve %>% group_by(region, corp) %>% 
                summarize(1) %>% nrow
    num_corp

    ## [1] 60

    # number of...schools = 160 
    num_school <- achieve %>% group_by(region, corp, school) %>% 
                  summarize(1) %>% nrow
    num_school 

    ## [1] 160

    # number of...classes = 568
    num_class <- achieve %>% group_by(region, corp, school, class) %>% 
                 summarize(1) %>% nrow
    num_class 

    ## [1] 568

    # number of...students = 10320
    num_id <- achieve %>% nrow
    num_id

    ## [1] 10320

There are 10320 pupils nested within 160 schools.

<!-- ========================================================= -->
Summarize Descriptive Statistics
================================

<!-- ========================================================= -->
<!-- ========================================================= -->
Using `stargazer()` from the `stargazer` package
------------------------------------------------

<!-- ========================================================= -->
Most posters, journal articles, and reports start with a table of
descriptive statistics. Since it tends to come first, this type of table
is often refered to as *Table 1*. The `stargazer()` function can be used
to create such a table, but only for the entire dataset (Hlavac 2015). I
haven't been able to find a way to get it to summarize subsamples and
compare them in the standard format.

    # summarize the numeric variables with stargazer
    achieve %>% 
      select(classize, gender, 
             geread, gevocab, senroll, age) %>% 
      data.frame %>% 
      stargazer(title  = "Summary of Some Numeric Variables",
                header = FALSE)

\begin{table}[!htbp] \centering 
  \caption{Summary of Some Numeric Variables} 
  \label{} 
\begin{tabular}{@{\extracolsep{5pt}}lccccc} 
\\[-1.8ex]\hline 
\hline \\[-1.8ex] 
Statistic & \multicolumn{1}{c}{N} & \multicolumn{1}{c}{Mean} & \multicolumn{1}{c}{St. Dev.} & \multicolumn{1}{c}{Min} & \multicolumn{1}{c}{Max} \\ 
\hline \\[-1.8ex] 
geread & 10,320 & 4.341 & 2.332 & 0.000 & 12.000 \\ 
gevocab & 10,320 & 4.494 & 2.368 & 0.000 & 11.200 \\ 
senroll & 10,320 & 533.415 & 154.797 & 115 & 916 \\ 
age & 10,320 & 107.529 & 5.060 & 82 & 135 \\ 
\hline \\[-1.8ex] 
\end{tabular} 
\end{table}

<!-- ========================================================= -->
Using `table1()` from the `furniture` package
---------------------------------------------

<!-- ========================================================= -->
Our colleages Tyson Barrett and Emily Brignone have authored the
**furniture** package which is now available on CRAN (Barrett and
Brignone 2016). It includes the extremely useful function `table1()`[1]
which simplifies the common task of creating a stratified, comparative
table of descriptive statistics. Full documentation can be accessed by
executing `?furniture::table1`.

    table1(achieve,
           geread, gevocab, age, 
           var_names      = c("Reading Score",  
                              "Vocabulary Score",
                              "Age (in months)"),    # override names
           splitby        = ~ gender,                # var to divide sample by
           test           = TRUE,                    # test groups different?
           format_output  = "pvalues",               # display p-vals for diff
           output_type    = "latex",                 # output for latex
           align          = c("l", "r", "r", "r"),   # column alignment
           caption        = "Compare pupils, by genders")  # title

\begin{table}

\caption{Compare pupils, by genders}
\centering
\begin{tabular}[t]{lrrr}
\toprule
  & Female & Male & P-Value\\
\midrule
Observations & 5143 & 5177 & \\
Reading Score &  &  & 0.218\\
 & 4.37 (2.35) & 4.31 (2.31) & \\
Vocabulary Score &  &  & <.001\\
 & 4.58 (2.41) & 4.41 (2.32) & \\
\addlinespace
Age (in months) &  &  & <.001\\
 & 107.14 (4.95) & 107.92 (5.13) & \\
\bottomrule
\end{tabular}
\end{table}

<!-- ========================================================= -->
Visually Exploring the Data
===========================

<!-- ========================================================= -->
<!-- ========================================================= -->
Level One Plots: Disaggregate or ignore higher levels
-----------------------------------------------------

<!-- ========================================================= -->
For a first look, its useful to plot all the data points on a single
scatterplot as displayed in Figure . Due to the large sample size, many
points end up being plotted on top of or very near each other
(*overplotted*). When this is the case, it can be useful to use
`geom_hex()`[2] rather than `geom_point()` so the color saturation of
the hexigons convey the number of points at that location (Wickham
2009).

    achieve %>% 
      ggplot(aes(x = gevocab, y = geread))+
      stat_binhex(colour = "grey85", na.rm  = TRUE) +    # outlines
      scale_fill_gradientn(colors   = c("grey","black"), # fill color extremes
                           name     = "Frequency",       # legend title
                           na.value = NA) +              # color for count = 0
      theme_bw() +
      labs(x = "Vocabulary Score", y = "Reading Score") +
      scale_x_continuous(breaks = seq(from = 0, to = 12, by = 2)) + 
      scale_y_continuous(breaks = seq(from = 0, to = 12, by = 2)) 

![Association between vocabulary and reading scores
](Finch_ch3_Fit_2_Levels_files/figure-markdown_strict/unnamed-chunk-6-1.png)

<!-- ========================================================= -->
Multilevel plots: illustrate two nested levels
----------------------------------------------

<!-- ========================================================= -->
Up to this point, all investigation of this dataset has been only at the
pupil level and any nesting or clustering within schools has been
ignored. Plotting is a good was to start to get an idea of the
school-to-school variability. Figure displays four handpicked school to
illustrate the degreen of school-to-school variability in the
association between vocab and reading scores.

    achieve %>% 
      #filter(school %in% levels(achieve$school)[1:9]) %>% 
      filter(school %in% c(1321, 6181, 6197, 6823)) %>% 
      ggplot(aes(x = gevocab,
                 y = geread))+
      geom_count() +
      geom_smooth(method = "lm") +
      theme_bw() +
      labs(x = "Vocabulary Score",
           y = "Reading Score") +
      scale_x_continuous(breaks = seq(from = 0, to = 12, by = 3)) + 
      scale_y_continuous(breaks = seq(from = 0, to = 12, by = 3)) +
      facet_wrap(~ school, labeller = "label_both")

![Association between vocabulary and reading scores, for 4 hand selected
schools
](Finch_ch3_Fit_2_Levels_files/figure-markdown_strict/unnamed-chunk-7-1.png)

Another way to explore the school-to-school variability is to plot the
linear model fit independently to each of the schools. Figure displays
only the smooth lines without the standard error bands or the raw data
in the form of points or hexagons.

    achieve %>% 
      ggplot(aes(x       = gevocab,
                 y       = geread,
                 group   = school)) +
      geom_smooth(method = "lm",
                  se     = FALSE,
                  size   = 0.3) +
      theme_bw() +
      labs(x = "Vocabulary Score",
           y = "Reading Score") +
      scale_x_continuous(breaks = seq(from = 0, to = 12, by = 2)) + 
      scale_y_continuous(breaks = seq(from = 0, to = 12, by = 3)) 

![Spaghetti plot: association between vocabulary and reading scores, one
line per school
](Finch_ch3_Fit_2_Levels_files/figure-markdown_strict/unnamed-chunk-8-1.png)

Due to the high number of schools, Figure resembles a hairball and is
hard to deduce much about individual schools. By using the
`facet_grid()` layer, we can seperate the schools out so better see
school-to-school variability. It also allows investigation of higher
level predictors, such as the school's SES (median split) and class size
as seen in Figure .

    achieve %>% 
      mutate(ses2 = ntile(ses, 2) %>% 
               factor(labels = c("SES: Lower Half", 
                                 "SES: Upper Half"))) %>% 
      ggplot(aes(x       = gevocab,
                 y       = geread,
                 group   = school)) +
      geom_smooth(method = "lm",
                  se     = FALSE,
                  size   = 0.3) +
      theme_bw() +
      labs(x = "Vocabulary Score",
           y = "Reading Score") +
      scale_x_continuous(breaks = seq(from = 0, to = 12, by = 3)) + 
      scale_y_continuous(breaks = seq(from = 0, to = 12, by = 3)) +
      facet_grid(classize ~ ses2)

![Spaghetti plot: association between vocabulary and reading scores, one
line per school, divided by SES median split and class size
](Finch_ch3_Fit_2_Levels_files/figure-markdown_strict/unnamed-chunk-9-1.png)

<!-- ========================================================= -->
Disaggregating: with `lm()` in Base R
-------------------------------------

<!-- ========================================================= -->
    # linear model - ignores school (for reference only)
    fit_read_lm_0 <- lm(formula = geread ~ 1,   
                    data    = achieve)

    texreg(fit_read_lm_0, 
           custom.model.names = "LR",
           digits        = 4,
           float.pos     = "hb",
           caption       = "Linear Regression for comparison, ignores clustering",
           caption.above = TRUE,
           label         = "tab:LR")

\begin{table}[hb]
\caption{Linear Regression for comparison, ignores clustering}
\begin{center}
\begin{tabular}{l c }
\hline
 & LR \\
\hline
(Intercept) & $4.3408^{***}$ \\
            & $(0.0230)$     \\
\hline
R$^2$       & 0.0000         \\
Adj. R$^2$  & 0.0000         \\
Num. obs.   & 10320          \\
RMSE        & 2.3319         \\
\hline
\multicolumn{2}{l}{\scriptsize{$^{***}p<0.001$, $^{**}p<0.01$, $^*p<0.05$}}
\end{tabular}
\label{tab:LR}
\end{center}
\end{table}
As is seen in table we can see...

<!-- ========================================================= -->
Fitting Multilevel Models
=========================

<!-- ========================================================= -->
<!-- ========================================================= -->
Step 1) No Explanatory variables - random intercepts
----------------------------------------------------

<!-- ========================================================= -->
A so called *Empty Model* only includes random intercepts. No
independent variables are involved, other the grouping or clustering
variable that designates how *level 1* units are nested within *level 2*
units. For a cross-sectional study design this would be the grouping
variables, where as for longitudinal or repeated measures designs this
would be the subject identifier. This **nested structure** variable
should be set to have class `factor`.

    # mulitlevel model - treats school as Random
    fit_read_0re <- lmer(geread ~ 1 + (1 | school) , 
                          data = achieve,
                          REML = TRUE)

    fit_read_0ml <- lmer(geread ~ 1 + (1 | school), 
                          data = achieve,
                          REML = FALSE)

    texreg(list(fit_read_lm_0, fit_read_0re, fit_read_0ml), 
           custom.model.names = c("LR", "M0 w/REML", "M0 w/ML"),
           float.pos     = "!hbt",
           caption       = "Linear regression vs. multilevel regression",
           caption.above = TRUE,
           label         = "tab:LR_M0")

\begin{table}[!hbt]
\caption{Linear regression vs. multilevel regression}
\begin{center}
\begin{tabular}{l c c c }
\hline
 & LR & M0 w/REML & M0 w/ML \\
\hline
(Intercept)             & $4.34^{***}$ & $4.31^{***}$ & $4.31^{***}$ \\
                        & $(0.02)$     & $(0.05)$     & $(0.05)$     \\
\hline
R$^2$                   & 0.00         &              &              \\
Adj. R$^2$              & 0.00         &              &              \\
Num. obs.               & 10320        & 10320        & 10320        \\
RMSE                    & 2.33         &              &              \\
AIC                     &              & 46274.31     & 46270.34     \\
BIC                     &              & 46296.03     & 46292.06     \\
Log Likelihood          &              & -23134.15    & -23132.17    \\
Num. groups: school     &              & 160          & 160          \\
Var: school (Intercept) &              & 0.39         & 0.39         \\
Var: Residual           &              & 5.05         & 5.05         \\
\hline
\multicolumn{4}{l}{\scriptsize{$^{***}p<0.001$, $^{**}p<0.01$, $^*p<0.05$}}
\end{tabular}
\label{tab:LR_M0}
\end{center}
\end{table}
Notice that the estimate for the intercept is nearly the same in the
linear regression and intercept only models (see Table ), but the
standard errors are quite different. When there is clustering in sample,
the result of ignoring it is under estimation of the standard errors and
over stating the significance of associations. This table was made with
the `texreg()` function in the self named package (Leifeld 2013). I tend
to prefer this display over `stargazer()`.

### Estimate ICC

$$
\\rho = \\frac{\\sigma^2\_{u0}}{\\sigma^2\_{u0} + \\sigma^2\_e}
$$

    # extract the random term's variance-covariance estimates 
    fit_read_0re_VarCorr <- fit_read_0re %>% VarCorr %>% data.frame
    fit_read_0re_VarCorr %>% apa_table(caption = "M0: variance-covariance matrix",
                                       placement = "h!")

<center>
Table. *M0: variance-covariance matrix*
</center>
<table>
<thead>
<tr class="header">
<th align="left">grp</th>
<th align="left">var1</th>
<th align="left">var2</th>
<th align="right">vcov</th>
<th align="right">sdcor</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">school</td>
<td align="left">(Intercept)</td>
<td align="left">NA</td>
<td align="right">0.3915154</td>
<td align="right">0.6257119</td>
</tr>
<tr class="even">
<td align="left">Residual</td>
<td align="left">NA</td>
<td align="left">NA</td>
<td align="right">5.0450083</td>
<td align="right">2.2461096</td>
</tr>
</tbody>
</table>

    # variances in the residuals, attributed to random noise
    fit_read_0re_sigma2_e <- 
      fit_read_0re_VarCorr %>% 
      filter(grp == "Residual") %>% 
      select(vcov) %>% 
      as.double
    fit_read_0re_sigma2_e 

\[1\] 5.045008

    # variances in the intercepts, attributed to the grouping (schools)
    fit_read_0re_sigma2_u0 <- 
      fit_read_0re_VarCorr %>% 
      filter(grp == "school") %>% 
      select(vcov) %>% 
      as.double
    fit_read_0re_sigma2_u0 

\[1\] 0.3915154

    # Calculate the estimated ICC
    rho <- fit_read_0re_sigma2_u0 / 
                  (fit_read_0re_sigma2_u0 + fit_read_0re_sigma2_e)
    rho

\[1\] 0.07201576

On page 45 of the Finch textbook, the authors substituted standard
deviations into the formula, rather than variances. The mistake is
listed on their webpage errata (<http://www.mlminr.com/errata>). With
the correct values, we find that the estimate for the ICC is 0.0720158.
While this is somewhat of a low correlation, it definately warrents the
includion of the school effect in regresion models.

<!-- ========================================================= -->
Step 2) Lower-level explanatory variables - fixed, ML
-----------------------------------------------------

<!-- ========================================================= -->
**Variance Component** models (steps 2 and 3) - decompose the INTERCEPT
variance into different variance compondents for each level. The
regression intercepts are assumed to varry ACROSS the groups, while the
slopes are assumed fixed.

\begin{center}
If only level 1 predictors and random intercepts   
MLM \textit{(ML)} $\approx$ ANCOVA \textit{OLS}
\end{center}
Fixed effects selection should come prior to random effects. You should
use *Maximum Likelihood (ML)* estimation when fitting these models.

    # add pupil's vocab score as a fixed effects predictor
    fit_read_1ml <- lmer(geread ~ gevocab + (1 | school), 
                         data = achieve,
                         REML = FALSE)

    texreg(list(fit_read_0ml, fit_read_1ml), 
           custom.model.names = c("M0", "M1"),
           custom.coef.names  = c("(Intercept)", 
                                  "Vocab Score"),
           digits        = 4,
           float.pos     = "!hbt",
           caption       = "Adding a level 1 fixed effect (fit with ML)",
           caption.above = TRUE,
           label         = "tab:M0_M1")

\begin{table}[!hbt]
\caption{Adding a level 1 fixed effect (fit with ML)}
\begin{center}
\begin{tabular}{l c c }
\hline
 & M0 & M1 \\
\hline
(Intercept)             & $4.3068^{***}$ & $2.0231^{***}$ \\
                        & $(0.0548)$     & $(0.0492)$     \\
Vocab Score             &                & $0.5130^{***}$ \\
                        &                & $(0.0084)$     \\
\hline
AIC                     & 46270.3388     & 43132.4318     \\
BIC                     & 46292.0643     & 43161.3991     \\
Log Likelihood          & -23132.1694    & -21562.2159    \\
Num. obs.               & 10320          & 10320          \\
Num. groups: school     & 160            & 160            \\
Var: school (Intercept) & 0.3885         & 0.0987         \\
Var: Residual           & 5.0450         & 3.7661         \\
\hline
\multicolumn{3}{l}{\scriptsize{$^{***}p<0.001$, $^{**}p<0.01$, $^*p<0.05$}}
\end{tabular}
\label{tab:M0_M1}
\end{center}
\end{table}
    # Likelihood Ratio Test (LRT)
    lrt_0vs1 <- anova(fit_read_0ml, fit_read_1ml, 
                      model.names = c("M0", "M1")) %>% data.frame

    lrt_0vs1 %>% apa_table(caption = "Compare M0 and M1 via the LRT (fit with ML)",
                           placement = "h!")

<center>
Table. *Compare M0 and M1 via the LRT (fit with ML)*
</center>
<table>
<thead>
<tr class="header">
<th align="left"></th>
<th align="right">Df</th>
<th align="right">AIC</th>
<th align="right">BIC</th>
<th align="right">logLik</th>
<th align="right">deviance</th>
<th align="right">Chisq</th>
<th align="right">Chi.Df</th>
<th align="right">Pr..Chisq.</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">M0</td>
<td align="right">3</td>
<td align="right">46270.34</td>
<td align="right">46292.06</td>
<td align="right">-23132.17</td>
<td align="right">46264.34</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
</tr>
<tr class="even">
<td align="left">M1</td>
<td align="right">4</td>
<td align="right">43132.43</td>
<td align="right">43161.40</td>
<td align="right">-21562.22</td>
<td align="right">43124.43</td>
<td align="right">3139.907</td>
<td align="right">1</td>
<td align="right">0</td>
</tr>
</tbody>
</table>

Since models 1 and 2 are nested models, only differing by the the
inclusion or exclusion of the fixed effects predictor `gevocab`, AND
both models were fit via Maximum Likelihood, we can compare the model
fit may be compared via the *Likilihood-Ratio Test (LRT)*. The
*Likelihood Ratio* value *(L. Ratio)* is found by subtracting the two
model's `-2 * logLik` values. Significance is judged by the Chi Squared
distribution, using the difference in the number of parameters fit as
the degrees of freedom.

Conclusion: A student's vocabulary score is positively associated with
tehir reading score, *χ*<sup>2</sup>(1) = 3139.9070209, *p* = 0[3].

### Level 1 *R*<sup>2</sup>

**Snijders and Bosker Formual** Finch (page 47), proportion of variance
in the outcome explained by predictor on level one[4]

\begin{table}[h!]
\centering
\begin{tabular}{c|cc}
\hline
 Variances   & residuals           & intercepts         \\  \hline
Model 1 (M1) & $\sigma^2_{e-M1}$   & $\sigma^2_{u0-M1}$ \\  
Model 0 (M0) & $\sigma^2_{e-M0}$   & $\sigma^2_{u0-M0}$ \\  \hline
\end{tabular}
\end{table}
$$
SB: R^2\_1 = 1 - \\frac{\\sigma^2\_{e-M1} + \\sigma^2\_{u0-M1}}
                     {\\sigma^2\_{e-M0} + \\sigma^2\_{u0-M0}}
$$

    # fit with REML
    fit_read_1re <- lmer(geread ~ gevocab + (1 | school), 
                         data = achieve,
                         REML = TRUE)

    # extract the random term's variance-covariance estimates 
    fit_read_1re_VarCorr <- fit_read_1re %>% VarCorr %>% data.frame
    fit_read_1re_VarCorr %>% apa_table(caption = "M1: variance-covariance matrix",
                                       placement = "h!")

<center>
Table. *M1: variance-covariance matrix*
</center>
<table>
<thead>
<tr class="header">
<th align="left">grp</th>
<th align="left">var1</th>
<th align="left">var2</th>
<th align="right">vcov</th>
<th align="right">sdcor</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">school</td>
<td align="left">(Intercept)</td>
<td align="left">NA</td>
<td align="right">0.0997786</td>
<td align="right">0.3158775</td>
</tr>
<tr class="even">
<td align="left">Residual</td>
<td align="left">NA</td>
<td align="left">NA</td>
<td align="right">3.7664703</td>
<td align="right">1.9407396</td>
</tr>
</tbody>
</table>

    # variances in the residuals, attributed to random noise
    fit_read_1re_sigma2_e <- 
      fit_read_1re_VarCorr %>% 
      filter(grp == "Residual") %>% 
      select(vcov) %>% 
      as.double
    fit_read_1re_sigma2_e 

\[1\] 3.76647

    # variances in the intercepts, attributed to the grouping (schools)
    fit_read_1re_sigma2_u0 <- 
      fit_read_1re_VarCorr %>% 
      filter(grp == "school") %>% 
      select(vcov) %>% 
      as.double
    fit_read_1re_sigma2_u0 

\[1\] 0.09977858

    # r-squared for the level one 
    fit_read_1re_r2_1sb <-  1 - (fit_read_1re_sigma2_e + fit_read_1re_sigma2_u0) /
                                (fit_read_0re_sigma2_e + fit_read_0re_sigma2_u0)
    fit_read_1re_r2_1sb

\[1\] 0.288838

This tells us that the global effect of vocabulary scores accounts for
28.8838028% of the variance in the reading scores above and belond that
accounted for due to school-to-school variability (the null model).

### Level 1 *R*<sup>2</sup>

**Raudenbush and Bryk Approximate Formula**, Hox (page 71)

    # Raudenbush and Bryk approximate R-squared
    # "proportion of variance explained by the first level predictor(s)"
    fit_read_1re_r2_1rb <- (fit_read_0re_sigma2_e - fit_read_1re_sigma2_e) /
                            fit_read_0re_sigma2_e
    fit_read_1re_r2_1rb

\[1\] 0.2534263

### Level 2 *R*<sup>2</sup>, proportion of variance in the outcome explained by predictor

$$
R^2\_2 = 1 - \\frac{\\frac{\\sigma^2\_{e-M1}}{B} + \\sigma^2\_{u0-M1}}
                 {\\frac{\\sigma^2\_{e-M0}}{B} + \\sigma^2\_{u0-M0}}
$$

*B* is the average size of the Level 2 units (schools):

    # average sample cluster size
    B <- num_id / num_school
    B

\[1\] 64.5

    # r-squared for the level two 
    fit_read_1re_r2_2sb <-  1 - (fit_read_1re_sigma2_e/B + fit_read_1re_sigma2_u0) /
                                (fit_read_0re_sigma2_e/B + fit_read_0re_sigma2_u0)
    fit_read_1re_r2_2sb

\[1\] 0.6632691

Part of investigating lower level explanatory variables, is checking for
interactions between these variables. The interaction between fixed
effects is also considered to be a fixed effect, so we need to employ
*Maximum Likelihood* estimation to compare nested models.

    fit_read_4ml <- lmer(geread ~ gevocab + age + (1 | school), 
                         data = achieve,
                         REML = FALSE)

    fit_read_5ml <- lmer(geread ~ gevocab + age + gevocab*age + (1 | school), 
                         data = achieve,
                         REML = FALSE)

    texreg(list(fit_read_4ml, fit_read_5ml), 
           custom.model.names = c("M4", "M5"),
           custom.coef.names  = c("(Intercept)", 
                                  "Vocab Score", 
                                  "Age (months)", 
                                  "Vocab x Age"),
           float.pos     = "!hbt",
           caption       = "Level 1 fixed effect interaction (fit with ML)",
           caption.above = TRUE,
           label         = "tab:M4_M5")

\begin{table}[!hbt]
\caption{Level 1 fixed effect interaction (fit with ML)}
\begin{center}
\begin{tabular}{l c c }
\hline
 & M4 & M5 \\
\hline
(Intercept)             & $3.00^{***}$ & $5.19^{***}$  \\
                        & $(0.42)$     & $(0.87)$      \\
Vocab Score             & $0.51^{***}$ & $-0.03$       \\
                        & $(0.01)$     & $(0.19)$      \\
Age (months)            & $-0.01^{*}$  & $-0.03^{***}$ \\
                        & $(0.00)$     & $(0.01)$      \\
Vocab x Age             &              & $0.01^{**}$   \\
                        &              & $(0.00)$      \\
\hline
AIC                     & 43128.82     & 43122.57      \\
BIC                     & 43165.03     & 43166.02      \\
Log Likelihood          & -21559.41    & -21555.28     \\
Num. obs.               & 10320        & 10320         \\
Num. groups: school     & 160          & 160           \\
Var: school (Intercept) & 0.10         & 0.10          \\
Var: Residual           & 3.76         & 3.76          \\
\hline
\multicolumn{3}{l}{\scriptsize{$^{***}p<0.001$, $^{**}p<0.01$, $^*p<0.05$}}
\end{tabular}
\label{tab:M4_M5}
\end{center}
\end{table}
    # Likelihood Ratio Test (LRT)
    lrt_4vs5 <- anova(fit_read_4ml, fit_read_5ml, 
                      model.names = c("M4", "M5")) %>% data.frame

    lrt_4vs5 %>% apa_table(caption = "Compare M4 and M5 via the LRT (fit with ML)",
                           placement = "h!")

<center>
Table. *Compare M4 and M5 via the LRT (fit with ML)*
</center>
<table>
<thead>
<tr class="header">
<th align="left"></th>
<th align="right">Df</th>
<th align="right">AIC</th>
<th align="right">BIC</th>
<th align="right">logLik</th>
<th align="right">deviance</th>
<th align="right">Chisq</th>
<th align="right">Chi.Df</th>
<th align="right">Pr..Chisq.</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">M4</td>
<td align="right">5</td>
<td align="right">43128.82</td>
<td align="right">43165.03</td>
<td align="right">-21559.41</td>
<td align="right">43118.82</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
</tr>
<tr class="even">
<td align="left">M5</td>
<td align="right">6</td>
<td align="right">43122.57</td>
<td align="right">43166.02</td>
<td align="right">-21555.28</td>
<td align="right">43110.57</td>
<td align="right">8.251354</td>
<td align="right">1</td>
<td align="right">0.0040722</td>
</tr>
</tbody>
</table>

Conclusion: A effect of a student's vocabulary score is dependent on
their age, *χ*<sup>2</sup>(1) = 8.2513535, *p* = 0.0040722[5].

<!-- ========================================================= -->
Step 3) Higher-level explanatory variables - fixed, ML
------------------------------------------------------

<!-- ========================================================= -->
School enrollment (`senroll`) applies to each school as a whole. When a
variable is measured at a higher level, all units in the same group have
the same value. In this case, all student in the same school have the
same value for `senroll`.

    fit_read_2ml <- lmer(geread ~ gevocab + age + gevocab*age + 
                                  senroll + (1 | school), 
                       data = achieve,
                       REML = FALSE)

    texreg(list(fit_read_0ml, fit_read_1ml, fit_read_5ml, fit_read_2ml), 
           custom.model.names = c("M0", "M1", "M5", "M2"),
           custom.coef.names  = c("(Intercept)", 
                                  "Vocab Score", 
                                  "Age (months)",
                                  "Vocab x Age",
                                  "School Enrollment"),
           float.pos     = "!hbt",
           caption       = "Adding a level 2 fixed effect (fit with ML)",
           caption.above = TRUE,
           label         = "tab:M0_M1_M2")

\begin{table}[!hbt]
\caption{Adding a level 2 fixed effect (fit with ML)}
\begin{center}
\begin{tabular}{l c c c c }
\hline
 & M0 & M1 & M5 & M2 \\
\hline
(Intercept)             & $4.31^{***}$ & $2.02^{***}$ & $5.19^{***}$  & $5.24^{***}$  \\
                        & $(0.05)$     & $(0.05)$     & $(0.87)$      & $(0.87)$      \\
Vocab Score             &              & $0.51^{***}$ & $-0.03$       & $-0.03$       \\
                        &              & $(0.01)$     & $(0.19)$      & $(0.19)$      \\
Age (months)            &              &              & $-0.03^{***}$ & $-0.03^{***}$ \\
                        &              &              & $(0.01)$      & $(0.01)$      \\
Vocab x Age             &              &              & $0.01^{**}$   & $0.01^{**}$   \\
                        &              &              & $(0.00)$      & $(0.00)$      \\
School Enrollment       &              &              &               & $-0.00$       \\
                        &              &              &               & $(0.00)$      \\
\hline
AIC                     & 46270.34     & 43132.43     & 43122.57      & 43124.31      \\
BIC                     & 46292.06     & 43161.40     & 43166.02      & 43175.01      \\
Log Likelihood          & -23132.17    & -21562.22    & -21555.28     & -21555.16     \\
Num. obs.               & 10320        & 10320        & 10320         & 10320         \\
Num. groups: school     & 160          & 160          & 160           & 160           \\
Var: school (Intercept) & 0.39         & 0.10         & 0.10          & 0.10          \\
Var: Residual           & 5.05         & 3.77         & 3.76          & 3.76          \\
\hline
\multicolumn{5}{l}{\scriptsize{$^{***}p<0.001$, $^{**}p<0.01$, $^*p<0.05$}}
\end{tabular}
\label{tab:M0_M1_M2}
\end{center}
\end{table}
    # Likelihood Ratio Test (LRT)
    lrt_5vs2 <- anova(fit_read_0ml, fit_read_1ml, fit_read_5ml, fit_read_2ml, 
                      model.names = c("M0", "M1", "M5", "M2")) %>% data.frame

    lrt_5vs2 %>% apa_table(caption = "Compare M0, M1, M5, and M2 via the LRT (fit with ML)",
                           placement = "h!")

<center>
Table. *Compare M0, M1, M5, and M2 via the LRT (fit with ML)*
</center>
<table>
<thead>
<tr class="header">
<th align="left"></th>
<th align="right">Df</th>
<th align="right">AIC</th>
<th align="right">BIC</th>
<th align="right">logLik</th>
<th align="right">deviance</th>
<th align="right">Chisq</th>
<th align="right">Chi.Df</th>
<th align="right">Pr..Chisq.</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">M0</td>
<td align="right">3</td>
<td align="right">46270.34</td>
<td align="right">46292.06</td>
<td align="right">-23132.17</td>
<td align="right">46264.34</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
</tr>
<tr class="even">
<td align="left">M1</td>
<td align="right">4</td>
<td align="right">43132.43</td>
<td align="right">43161.40</td>
<td align="right">-21562.22</td>
<td align="right">43124.43</td>
<td align="right">3139.9070209</td>
<td align="right">1</td>
<td align="right">0.0000000</td>
</tr>
<tr class="odd">
<td align="left">M5</td>
<td align="right">6</td>
<td align="right">43122.57</td>
<td align="right">43166.02</td>
<td align="right">-21555.28</td>
<td align="right">43110.57</td>
<td align="right">13.8630471</td>
<td align="right">2</td>
<td align="right">0.0009765</td>
</tr>
<tr class="even">
<td align="left">M2</td>
<td align="right">7</td>
<td align="right">43124.31</td>
<td align="right">43175.01</td>
<td align="right">-21555.16</td>
<td align="right">43110.31</td>
<td align="right">0.2548234</td>
<td align="right">1</td>
<td align="right">0.6136991</td>
</tr>
</tbody>
</table>

<!-- ========================================================= -->
Step 4) Explanatory variables predict Slopes - random, REML
-----------------------------------------------------------

<!-- ========================================================= -->
**Random Coefficient** models - decompose the SLOPE variance BETWEEN
groups.

The fixed effect of the predictor captures the overall association it
has with the outcome (intercept), while the random effect of the
predictor captures the group-to-group variation in the association
(slope)[6].

    fit_read_1re <- lmer(geread ~ gevocab + (1 | school), 
                         data = achieve,
                         REML = TRUE)

    #fit_read_3re <- lmer(geread ~ gevocab + (gevocab | school), 
    #                     data = achieve,
    #                     REML = TRUE)         # failed to converge :(

    fit_read_3re <- lme(fixed  = geread ~ gevocab,
                        random = ~gevocab | school, 
                        data = achieve)

    texreg(list(fit_read_1re, fit_read_3re), 
           custom.model.names = c("M1", "M3"),
           custom.coef.names  = c("(Intercept)", 
                                  "Vocab Score"),
           float.pos     = "!hbt",
           caption       = "Adding a level 2 random effect (fit with REML)",
           caption.above = TRUE,
           label         = "tab:M1_M3_M4")

\begin{table}[!hbt]
\caption{Adding a level 2 random effect (fit with REML)}
\begin{center}
\begin{tabular}{l c c }
\hline
 & M1 & M3 \\
\hline
(Intercept)             & $2.02^{***}$ & $2.01^{***}$ \\
                        & $(0.05)$     & $(0.06)$     \\
Vocab Score             & $0.51^{***}$ & $0.52^{***}$ \\
                        & $(0.01)$     & $(0.01)$     \\
\hline
AIC                     & 43145.20     & 43004.85     \\
BIC                     & 43174.17     & 43048.30     \\
Log Likelihood          & -21568.60    & -21496.43    \\
Num. obs.               & 10320        & 10320        \\
Num. groups: school     & 160          &              \\
Var: school (Intercept) & 0.10         &              \\
Var: Residual           & 3.77         &              \\
Num. groups             &              & 160          \\
\hline
\multicolumn{3}{l}{\scriptsize{$^{***}p<0.001$, $^{**}p<0.01$, $^*p<0.05$}}
\end{tabular}
\label{tab:M1_M3_M4}
\end{center}
\end{table}
    # Likelihood Ratio Test (LRT)
    lrt_1vs3 <- anova(fit_read_1re, fit_read_3re, 
                      refit=FALSE, 
                      model.names = c("M1", "M3")) %>% data.frame

    lrt_1vs3 %>% apa_table(caption = "Compare M1 and M3 via the LRT (fit with RML)",
                           placement = "h!")

<center>
Table. *Compare M1 and M3 via the LRT (fit with RML)*
</center>
<table>
<thead>
<tr class="header">
<th align="left"></th>
<th align="right">Sum.Sq</th>
<th align="right">Mean.Sq</th>
<th align="right">NumDF</th>
<th align="right">DenDF</th>
<th align="right">F.value</th>
<th align="right">Pr..F.</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">gevocab</td>
<td align="right">14134.07</td>
<td align="right">14134.07</td>
<td align="right">1</td>
<td align="right">9800.844</td>
<td align="right">3752.605</td>
<td align="right">0</td>
</tr>
</tbody>
</table>

You can use the Chi-squared LRT test based on deviances even though we
fit our modesl with REML, since the models only differe in terms of
random effects, but have the same fixed effects.

<!-- ========================================================= -->
Step 5) Cross-Level interactions between explanatory variables - fixed, ML
--------------------------------------------------------------------------

<!-- ========================================================= -->
Remember that an interaction beween fixed effects is also fixed.

    fit_read_6ml_r <- lmer(geread ~ gevocab + senroll + (1 | school), 
                           data = achieve,
                           REML = FALSE)

    fit_read_6ml <- lmer(geread ~ gevocab + senroll + gevocab*senroll + (1 | school), 
                           data = achieve,
                           REML = FALSE)

    texreg(list(fit_read_6ml_r, fit_read_6ml), 
           custom.model.names = c("M6 w/o inter", "M6 w/inter"),
           custom.coef.names  = c("(Intercept)", "Vocab Score", 
                                  "School Enroll", "Vocab x Enroll"),
           float.pos     = "!hbt",
           caption       = "Cross Level Interaction (fit with ML)",
           caption.above = TRUE,
           label         = "tab:M6")

\begin{table}[!hbt]
\caption{Cross Level Interaction (fit with ML)}
\begin{center}
\begin{tabular}{l c c }
\hline
 & M6 w/o inter & M6 w/inter \\
\hline
(Intercept)             & $2.07^{***}$ & $1.75^{***}$ \\
                        & $(0.11)$     & $(0.17)$     \\
Vocab Score             & $0.51^{***}$ & $0.59^{***}$ \\
                        & $(0.01)$     & $(0.03)$     \\
School Enroll           & $-0.00$      & $0.00$       \\
                        & $(0.00)$     & $(0.00)$     \\
Vocab x Enroll          &              & $-0.00^{*}$  \\
                        &              & $(0.00)$     \\
\hline
AIC                     & 43134.18     & 43129.82     \\
BIC                     & 43170.39     & 43173.28     \\
Log Likelihood          & -21562.09    & -21558.91    \\
Num. obs.               & 10320        & 10320        \\
Num. groups: school     & 160          & 160          \\
Var: school (Intercept) & 0.10         & 0.10         \\
Var: Residual           & 3.77         & 3.76         \\
\hline
\multicolumn{3}{l}{\scriptsize{$^{***}p<0.001$, $^{**}p<0.01$, $^*p<0.05$}}
\end{tabular}
\label{tab:M6}
\end{center}
\end{table}
    # Likelihood Ratio Test (LRT)
    lrt_6 <- anova(fit_read_6ml_r, fit_read_6ml, 
                      model.names = c("WIthout", "With")) %>% data.frame

    lrt_6 %>% apa_table(caption = "Interaction between pupil vocab and school enrollment via the LRT (fit with ML)",
                           placement = "h!")

<center>
Table. *Interaction between pupil vocab and school enrollment via the
LRT (fit with ML)*
</center>
<table>
<thead>
<tr class="header">
<th align="left"></th>
<th align="right">Df</th>
<th align="right">AIC</th>
<th align="right">BIC</th>
<th align="right">logLik</th>
<th align="right">deviance</th>
<th align="right">Chisq</th>
<th align="right">Chi.Df</th>
<th align="right">Pr..Chisq.</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">WIthout</td>
<td align="right">5</td>
<td align="right">43134.18</td>
<td align="right">43170.39</td>
<td align="right">-21562.09</td>
<td align="right">43124.18</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
</tr>
<tr class="even">
<td align="left">With</td>
<td align="right">6</td>
<td align="right">43129.82</td>
<td align="right">43173.28</td>
<td align="right">-21558.91</td>
<td align="right">43117.82</td>
<td align="right">6.352515</td>
<td align="right">1</td>
<td align="right">0.0117215</td>
</tr>
</tbody>
</table>

<!-- ========================================================= -->
Centering Predictors
--------------------

<!-- ========================================================= -->
Centering variables measured on the lowest level only involves
subtacting the mean from every value. The spread or standard deviation
is not changed.

    # Grand Mean Centering
    achieve <- achieve %>% 
      mutate(gevocab_c = gevocab - mean(gevocab),
             age_c     = age     - mean(age),
             senroll_c = senroll - mean(senroll))

    # view the first few rows
    achieve %>% 
      select(id, school, 
             gevocab, gevocab_c, age, age_c, senroll, senroll_c) %>% 
      head %>% data.frame %>% 
      stargazer(summary  = FALSE,
                rownames = FALSE,
                header   = FALSE)

\begin{table}[!htbp] \centering 
  \caption{} 
  \label{} 
\begin{tabular}{@{\extracolsep{5pt}} cccccccc} 
\\[-1.8ex]\hline 
\hline \\[-1.8ex] 
id & school & gevocab & gevocab\_c & age & age\_c & senroll & senroll\_c \\ 
\hline \\[-1.8ex] 
1 & 767 & $3.100$ & $$-$1.394$ & $104$ & $$-$3.529$ & $463$ & $$-$70.415$ \\ 
2 & 767 & $2.800$ & $$-$1.694$ & $106$ & $$-$1.529$ & $463$ & $$-$70.415$ \\ 
3 & 767 & $1.700$ & $$-$2.794$ & $112$ & $4.471$ & $463$ & $$-$70.415$ \\ 
4 & 767 & $2.100$ & $$-$2.394$ & $109$ & $1.471$ & $463$ & $$-$70.415$ \\ 
5 & 767 & $2.400$ & $$-$2.094$ & $107$ & $$-$0.529$ & $463$ & $$-$70.415$ \\ 
6 & 767 & $2.400$ & $$-$2.094$ & $108$ & $0.471$ & $463$ & $$-$70.415$ \\ 
\hline \\[-1.8ex] 
\end{tabular} 
\end{table}
    # compare the summary statistcs
    achieve %>% 
      select(gevocab, gevocab_c, age, age_c, senroll, senroll_c) %>% 
      data.frame %>% 
      stargazer(flip     = TRUE,
                header   = FALSE)

\begin{table}[!htbp] \centering 
  \caption{} 
  \label{} 
\begin{tabular}{@{\extracolsep{5pt}}lcccccc} 
\\[-1.8ex]\hline 
\hline \\[-1.8ex] 
Statistic & gevocab & gevocab\_c & age & age\_c & senroll & senroll\_c \\ 
\hline \\[-1.8ex] 
N & 10,320 & 10,320 & 10,320 & 10,320 & 10,320 & 10,320 \\ 
Mean & 4.494 & $-$0.000 & 107.529 & 0.000 & 533.415 & 0.000 \\ 
St. Dev. & 2.368 & 2.368 & 5.060 & 5.060 & 154.797 & 154.797 \\ 
Min & 0.000 & $-$4.494 & 82 & $-$25.529 & 115 & $-$418.415 \\ 
Max & 11.200 & 6.706 & 135 & 27.471 & 916 & 382.585 \\ 
\hline \\[-1.8ex] 
\end{tabular} 
\end{table}
### Centered variables in a model, with an interaction

    fit_read_4ml <- lmer(geread ~ gevocab + age + (1 | school), 
                       data = achieve,
                       REML = FALSE)

    fit_read_5ml <- lmer(geread ~ gevocab + age + gevocab*age + (1 | school), 
                       data = achieve,
                       REML = FALSE)

    fit_read_5ml_c <- lmer(geread ~ gevocab_c + age_c + gevocab_c*age_c + (1 | school), 
                       data = achieve,
                       REML = FALSE)

    texreg(list(fit_read_4ml, fit_read_5ml, fit_read_5ml_c), 
           custom.model.names = c("M4 w/o centering",
                                  "M5 w/o centering", 
                                  "M5 w/centering"),
           custom.coef.names  = c("(Intercept)", 
                                  "Vocab Score", 
                                  "Age (months)", 
                                  "Vocab x Age",
                                  "Vocab Score", 
                                  "Age (months)", 
                                  "Vocab x Age"),
           float.pos     = "!hbt",
           caption       = "With and without mean centering predictors (fit with ML)",
           caption.above = TRUE,
           label         = "tab:M4_M5_M5c")

\begin{table}[!hbt]
\caption{With and without mean centering predictors (fit with ML)}
\begin{center}
\begin{tabular}{l c c c }
\hline
 & M4 w/o centering & M5 w/o centering & M5 w/centering \\
\hline
(Intercept)             & $3.00^{***}$ & $5.19^{***}$  & $4.33^{***}$ \\
                        & $(0.42)$     & $(0.87)$      & $(0.03)$     \\
Vocab Score             & $0.51^{***}$ & $-0.03$       & $0.51^{***}$ \\
                        & $(0.01)$     & $(0.19)$      & $(0.01)$     \\
Age (months)            & $-0.01^{*}$  & $-0.03^{***}$ & $-0.01$      \\
                        & $(0.00)$     & $(0.01)$      & $(0.00)$     \\
Vocab x Age             &              & $0.01^{**}$   & $0.01^{**}$  \\
                        &              & $(0.00)$      & $(0.00)$     \\
\hline
AIC                     & 43128.82     & 43122.57      & 43122.57     \\
BIC                     & 43165.03     & 43166.02      & 43166.02     \\
Log Likelihood          & -21559.41    & -21555.28     & -21555.28    \\
Num. obs.               & 10320        & 10320         & 10320        \\
Num. groups: school     & 160          & 160           & 160          \\
Var: school (Intercept) & 0.10         & 0.10          & 0.10         \\
Var: Residual           & 3.76         & 3.76          & 3.76         \\
\hline
\multicolumn{4}{l}{\scriptsize{$^{***}p<0.001$, $^{**}p<0.01$, $^*p<0.05$}}
\end{tabular}
\label{tab:M4_M5_M5c}
\end{center}
\end{table}

References
==========

Barrett, Tyson, and Emily Brignone. 2016. *Furniture: Furniture for
Health, Behavioral, and Social Scientists*.

Hlavac, Marek. 2015. *Stargazer: Well-Formatted Regression and Summary
Statistics Tables*. Cambridge, USA: Harvard University.
<http://CRAN.R-project.org/package=stargazer>.

Leifeld, Philip. 2013. “texreg: Conversion of Statistical Model Output
in R to LaTeX and HTML Tables.” *Journal of Statistical Software* 55
(8): 1–24. <http://www.jstatsoft.org/v55/i08/>.

Wickham, Hadley. 2009. *Ggplot2: Elegant Graphics for Data Analysis*.
Springer-Verlag New York. <http://ggplot2.org>.

[1] All of the other tables we have created with `stargazer()` float to
the bottom of the page, after the code chunk is echoed, but these tables
float to the top of the page, before the code chunk. I have emailed the
package author determine how to best override this default behavior.

[2] I had to manually install the package `hexbin` for the `geom_hex()`
to run.

[3] The p-value should NEVER be equal to zero. Instead it should be
reported as *p* &lt; .001. I will look into how we can automate this
substitution.

[4] This formula also apprears in the errata. The subscripts in the
denominator of the fraction should be for model 0, not model 1. They did
substitute in the correct values.

[5] The p-value should NEVER be equal to zero. Instead it should be
reported as *p* &lt; .001. I will look into how we can automate this
substitution.

[6] A variable can be fit as BOTH a fixed and random effect.
