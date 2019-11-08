---
title: "Chapter 11"
author: "Julin N Maloof"
date: "10/31/2019"
output: 
  html_document: 
    keep_md: yes
---




```r
library(rethinking)
```

```
## Loading required package: rstan
```

```
## Loading required package: StanHeaders
```

```
## Loading required package: ggplot2
```

```
## rstan (Version 2.19.2, GitRev: 2e1f913d3ca3)
```

```
## For execution on a local, multicore CPU with excess RAM we recommend calling
## options(mc.cores = parallel::detectCores()).
## To avoid recompilation of unchanged Stan programs, we recommend calling
## rstan_options(auto_write = TRUE)
```

```
## Loading required package: parallel
```

```
## Loading required package: dagitty
```

```
## rethinking (Version 1.90)
```

```r
library(tidyverse)
```

```
## ── Attaching packages ──── tidyverse 1.2.1 ──
```

```
## ✔ tibble  2.1.3     ✔ purrr   0.3.2
## ✔ tidyr   1.0.0     ✔ dplyr   0.8.3
## ✔ readr   1.3.1     ✔ stringr 1.4.0
## ✔ tibble  2.1.3     ✔ forcats 0.4.0
```

```
## ── Conflicts ─────── tidyverse_conflicts() ──
## ✖ tidyr::extract() masks rstan::extract()
## ✖ dplyr::filter()  masks stats::filter()
## ✖ dplyr::lag()     masks stats::lag()
## ✖ purrr::map()     masks rethinking::map()
```

```r
library(GGally)
```

```
## Registered S3 method overwritten by 'GGally':
##   method from   
##   +.gg   ggplot2
```

```
## 
## Attaching package: 'GGally'
```

```
## The following object is masked from 'package:dplyr':
## 
##     nasa
```

0E1
10E2
10E3
10M1
 
Problems 1 and 2 at https://github.com/rmcelreath/statrethinking_winter2019/blob/master/homework/week06.pdf

## 10E1
_If an event has probability 0.35, what are the log-odds of this event?_


```r
logit(.35)
```

```
## [1] -0.6190392
```

```r
log(.35/.65)
```

```
## [1] -0.6190392
```

## 10E2
_If an event has log-odds 3.2, what is the probability of this event?_

```r
inv_logit(3.2)
```

```
## [1] 0.9608343
```

```r
exp(3.2) / (1+exp(3.2)) # algebra works!
```

```
## [1] 0.9608343
```

## 10E3
_Suppose that a coefficient in a logistic regression has value 1.7. What does this imply about the proportional change in odds of the outcome?_

The increases the probability of the event by 70%

## 10M1
_Asexplainedinthechapter,binomialdatacanbeorganizedinaggregatedanddisaggregated forms, without any impact on inference. But the likelihood of the data does change when the data are converted between the two formats. Can you explain why?_

extra parameter blah blah

## PDF 1

_The data in data(NWOGrants) are outcomes for scientific funding applications for the Netherlands Organization for Scientific Research (NWO) from 2010–2012 (see van der Lee and Ellemers doi:10.1073/pnas.1510159112). These data have a very similar structure to the UCBAdmit data discussed in Chapter 11. I want you to consider a similar question: What are the total and indirect causal effects of gender on grant awards? Consider a mediation path (a pipe) through dis- cipline. Draw the corresponding DAG and then use one or more binomial GLMs to answer the question. What is your causal interpretation? If NWO’s goal is to equalize rates of funding between the genders, what type of intervention would be most effective?_


```r
g <- dagitty("dag{
  G -> A;
  G -> D;
  D -> A
}")
coordinates(g) <- list(x=c(G=0, D=1, A=2),
                       y=c(G=0, D=1, A=0))
plot(g)
```

![](Chapter11_files/figure-html/unnamed-chunk-4-1.png)<!-- -->



```r
data("NWOGrants")
NWOGrants
```

```
##             discipline gender applications awards
## 1    Chemical sciences      m           83     22
## 2    Chemical sciences      f           39     10
## 3    Physical sciences      m          135     26
## 4    Physical sciences      f           39      9
## 5              Physics      m           67     18
## 6              Physics      f            9      2
## 7           Humanities      m          230     33
## 8           Humanities      f          166     32
## 9   Technical sciences      m          189     30
## 10  Technical sciences      f           62     13
## 11   Interdisciplinary      m          105     12
## 12   Interdisciplinary      f           78     17
## 13 Earth/life sciences      m          156     38
## 14 Earth/life sciences      f          126     18
## 15     Social sciences      m          425     65
## 16     Social sciences      f          409     47
## 17    Medical sciences      m          245     46
## 18    Medical sciences      f          260     29
```

plot it

```r
NWOGrants %>% 
  mutate(success=awards/applications) %>%
  ggplot(aes(x=discipline, y=success, color=gender, size=applications)) +
  geom_point() +
  theme(axis.text.x = element_text(angle=90, hjust = 1, vjust=0.5))
```

![](Chapter11_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

overall difference, irrespective of field

```r
d1 <- with(NWOGrants, list(g=ifelse(gender=="m",1,2),
                           applications=applications,
                           awards=awards))

m11.1 <- ulam(
  alist(awards ~ dbinom(applications, p),
        logit(p) <- a[g],
        a[g] ~ dnorm(0,1.5)),
  data=d1,
  chains = 4,
  cores = 4)
```

this is on the logit scale

```r
precis(m11.1, depth=2)
```

```
##           mean         sd      5.5%     94.5%    n_eff     Rhat
## a[1] -1.532296 0.06696433 -1.637565 -1.420580 1350.091 1.000028
## a[2] -1.740812 0.08389174 -1.870094 -1.610378 1333.340 1.000504
```

look at differences in award rate

```r
post <- extract.samples(m11.1) 

# relative scale
precis(data.frame(rel_dif=exp(post$a[,2]-post$a[,1])))
```

```
##             mean         sd      5.5%    94.5%     histogram
## rel_dif 0.816816 0.09108549 0.6791924 0.973958 ▁▁▂▃▇▇▅▂▂▁▁▁▁
```

```r
#absolute scale
precis(data.frame(prob_dif=inv_logit(post$a[,2])-inv_logit(post$a[,1])))
```

```
##                 mean         sd        5.5%        94.5%   histogram
## prob_dif -0.02834612 0.01498351 -0.05261722 -0.003616951 ▁▁▂▃▇▇▅▂▁▁▁
```
Women are 82% as likely to receive an award, translating to a reduced success rate of 3% 



## now fit a model that has a separate probability for each discipline


```r
d2 <- with(NWOGrants, list(g=ifelse(gender=="m",1,2),
                           applications=applications,
                           awards=awards, 
                           discipline=rep(1:9, each=2)))

m11.2 <- ulam(
  alist(awards ~ dbinom(applications, p),
        logit(p) <- a[g] + b[discipline],
        a[g] ~ dnorm(0,1.5),
        b[discipline] ~ dnorm(0,1.5)),
  data=d2,
  iter=2000,
  chains = 4,
  cores = 4)
```

```
## Warning: Bulk Effective Samples Size (ESS) is too low, indicating posterior means and medians may be unreliable.
## Running the chains for more iterations may help. See
## http://mc-stan.org/misc/warnings.html#bulk-ess
```


```r
pairs(m11.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

```r
precis(m11.2, depth=2)
```

```
##            mean        sd       5.5%      94.5%    n_eff     Rhat
## a[1] -1.2006456 0.4537998 -1.9075364 -0.4840414 327.6787 1.018782
## a[2] -1.3378077 0.4554483 -2.0431351 -0.6241983 338.2100 1.017836
## b[1]  0.1997746 0.4917393 -0.5938039  0.9871467 386.9325 1.013992
## b[2] -0.1531240 0.4777923 -0.9006507  0.6069762 380.8522 1.015175
## b[3]  0.1629690 0.5063306 -0.6462943  0.9770598 430.6079 1.013574
## b[4] -0.3769783 0.4636960 -1.1113154  0.3558326 345.2142 1.018064
## b[5] -0.3443805 0.4754096 -1.1024636  0.4019560 364.5607 1.015706
## b[6] -0.4202314 0.4860527 -1.1954046  0.3339823 378.7168 1.015329
## b[7] -0.1421930 0.4679858 -0.8797490  0.6020407 345.8202 1.016294
## b[8] -0.5984276 0.4601068 -1.3244547  0.1313876 340.3398 1.018420
## b[9] -0.4818142 0.4661975 -1.2301959  0.2492911 353.7825 1.015226
```

so much correlation.  Try not indexing gender. I this parameterization each discipline coefficient will be the rate for males in that discipline and then the gender coefficient will be the difference for females.


```r
d3 <- with(NWOGrants, list(g=ifelse(gender=="m",0,1),
                           applications=applications,
                           awards=awards, 
                           discipline=rep(1:9, each=2)))

m11.3 <- ulam(
  alist(awards ~ dbinom(applications, p),
        logit(p) <-  a[discipline] + b_female*g,
        a[discipline] ~ dnorm(0,1.5),
        b_female ~dnorm(0,1.5)),
  data=d3,
  iter=2000,
  chains = 4,
  cores = 4)
```



```r
precis(m11.3, depth=2)
```

```
##                mean        sd       5.5%        94.5%    n_eff      Rhat
## a[1]     -0.9746861 0.2083377 -1.3098744 -0.643431786 3831.701 0.9998193
## a[2]     -1.3336079 0.1910410 -1.6388055 -1.030912821 4606.922 1.0000673
## a[3]     -0.9956264 0.2556035 -1.4033874 -0.592566111 4175.883 0.9998506
## a[4]     -1.5584770 0.1396908 -1.7845543 -1.337323408 4361.830 0.9994781
## a[5]     -1.5279332 0.1652101 -1.7971612 -1.263913303 4208.094 1.0003499
## a[6]     -1.5887017 0.2041195 -1.9217697 -1.275838702 3622.840 0.9993471
## a[7]     -1.3189912 0.1572873 -1.5736924 -1.072298103 3292.071 0.9997634
## a[8]     -1.7824001 0.1119530 -1.9640327 -1.605250624 3377.191 0.9996266
## a[9]     -1.6592184 0.1369797 -1.8824286 -1.441558252 2958.912 0.9998627
## b_female -0.1619715 0.1067256 -0.3318247  0.007849806 2469.495 1.0004059
```

```r
pairs(m11.3)
```

![](Chapter11_files/figure-html/unnamed-chunk-13-1.png)<!-- -->
This looks much better.

look at differences in award rate.

On relative scale

```r
post <- extract.samples(m11.3, pars="b_female") 

# relative  and absolute scale
precis(list(rel_female=exp(post$b_female)))
```

```
##                 mean         sd      5.5%    94.5% histogram
## rel_female 0.8553283 0.09171534 0.7176131 1.007881 ▁▁▅▇▃▁▁▁▁
```

So women are 86% as likely to get an award, but the 89% condidence intervals cross 1

For the absolute scale I think it will probably be easier to use link


```r
# the difference between men and women will be the same for all disciplines using this model, so just get one of them.
newdat <- data.frame(g=0:1, 
                     discipline=1)
newdat
```

```
##   g discipline
## 1 0          1
## 2 1          1
```

```r
pred <- link(m11.3, data = newdat)
head(pred)
```

```
##           [,1]      [,2]
## [1,] 0.2775867 0.2402815
## [2,] 0.2583470 0.1983850
## [3,] 0.2705937 0.2285860
## [4,] 0.2650143 0.2487810
## [5,] 0.2717701 0.2103704
## [6,] 0.3085298 0.3141709
```


```r
precis(list(abs_female=pred[,2] - pred[,1]))
```

```
##                   mean         sd       5.5%       94.5%  histogram
## abs_female -0.03072858 0.02045121 -0.0630797 0.001366298 ▁▁▁▅▇▅▁▁▁▁
```

Women do 3% worse when accounting for overall differences in award rate between departments, although confidence interval touches 0

Can I do this from posterior directly?


```r
post <- extract.samples(m11.3)
str(post)
```

```
## List of 2
##  $ a       : num [1:4000, 1:9] -0.956 -1.055 -0.992 -1.02 -0.986 ...
##  $ b_female: num [1:4000(1d)] -0.1947 -0.3418 -0.2247 -0.0851 -0.337 ...
##  - attr(*, "source")= chr "ulam posterior: 4000 samples from m11.3"
```

```r
# again I should just be able to look at one discipline
precis(list(abd_female=inv_logit(post$a[,1]) -inv_logit(post$a[,1]-post$b_female)))
```

```
##                   mean         sd        5.5%       94.5%  histogram
## abd_female -0.03393459 0.02322153 -0.07215642 0.001360331 ▁▁▁▂▇▇▅▁▁▁
```

Overall I do see a reduction in award rates to women.  When we consider discipline than the signficance of this drops, but I wonder if there is still something going on...

interaction?


```r
m11.4 <- ulam(
  alist(awards ~ dbinom(applications, p),
        logit(p) <-  a[discipline] + b_female*g + inter[discipline]*g,
        a[discipline] ~ dnorm(0,1.5),
        b_female ~dnorm(0,1.5),
        inter[discipline] ~ dnorm(0,.5)),
  data=d3,
  iter=2000,
  chains = 4,
  cores = 4)
```


```r
precis(m11.4, depth=2)
```

```
##                  mean        sd        5.5%        94.5%    n_eff
## a[1]     -1.003859144 0.2323783 -1.38190458 -0.637370728 5044.918
## a[2]     -1.390813846 0.2047582 -1.72189022 -1.067906654 5400.341
## a[3]     -1.002511413 0.2619684 -1.42115626 -0.595824739 5251.879
## a[4]     -1.727191214 0.1742474 -2.01196980 -1.450076284 4290.980
## a[5]     -1.609843417 0.1813457 -1.90712802 -1.328376524 4556.343
## a[6]     -1.840685821 0.2602844 -2.27587379 -1.441632113 4177.454
## a[7]     -1.180992115 0.1778802 -1.46755397 -0.901728724 5318.549
## a[8]     -1.717119343 0.1305359 -1.92770571 -1.511426972 6055.808
## a[9]     -1.500247120 0.1599686 -1.76027720 -1.246040639 5559.588
## b_female -0.079553780 0.2031599 -0.40913646  0.241078280 1760.963
## inter[1] -0.003084928 0.3560298 -0.57710563  0.566270317 4084.913
## inter[2]  0.152817545 0.3479303 -0.41385323  0.701351291 4245.931
## inter[3] -0.064330971 0.4299350 -0.76151823  0.617709573 6135.776
## inter[4]  0.310246186 0.2879448 -0.14248981  0.769136975 2435.410
## inter[5]  0.242161757 0.3258880 -0.27498818  0.753834595 3467.792
## inter[6]  0.478290703 0.3413114 -0.06747827  1.035069556 3451.929
## inter[7] -0.434149332 0.3057509 -0.93029196  0.056219464 3390.123
## inter[8] -0.230918737 0.2544512 -0.63228696  0.172985331 2180.421
## inter[9] -0.439941628 0.2789779 -0.89674220 -0.008646316 2419.731
##               Rhat
## a[1]     0.9994468
## a[2]     1.0006243
## a[3]     1.0001560
## a[4]     0.9995334
## a[5]     0.9994826
## a[6]     0.9999905
## a[7]     1.0000666
## a[8]     0.9993370
## a[9]     0.9992794
## b_female 1.0014347
## inter[1] 1.0004490
## inter[2] 1.0005540
## inter[3] 1.0000503
## inter[4] 1.0013696
## inter[5] 1.0010111
## inter[6] 1.0013187
## inter[7] 0.9994397
## inter[8] 1.0013871
## inter[9] 1.0017528
```


```r
plot(precis(m11.4, depth=2))
```

![](Chapter11_files/figure-html/unnamed-chunk-20-1.png)<!-- -->


## 11.2

_2. Suppose that the NWO Grants sample has an unobserved confound that influences both choice of discipline and the probability of an award. One example of such a confound could be the career stage of each applicant. Suppose that in some disciplines, junior scholars apply for most of the grants. In other disciplines, scholars from all career stages compete. As a result, career stage influences discipline as well as the probability of being awarded a grant. Add these influences to your DAG from Problem 1. What happens now when you condition on discipline? Does it provide an un-confounded estimate of the direct path from gender to an award? Why or why not? Justify your answer with the back-door criterion. Hint: This is structurally a lot like the grandparents-parentschildren-neighborhoods example from a previous week. If you have trouble thinking this though, try simulating fake data, assuming your DAG is true. Then analyze it using the model from Problem 1. What do you conclude? Is it possible for gender to have a real direct causal influence but for a regression conditioning on both gender and discipline to suggest zero influence?_



```r
g <- dagitty("dag{
  G -> A;
  G -> D;
  D -> A;
  C -> A;
  C -> D;
}")
coordinates(g) <- list(x=c(G=0, D=1, A=2, C=2),
                       y=c(G=0, D=1, A=0, C=1))
plot(g)
```

![](Chapter11_files/figure-html/unnamed-chunk-21-1.png)<!-- -->

So if this is the DAG, the regression model from 1 closes the D->A, but leaves a back door from D through C to A?

# Chapter 11.2 problems

## 10E4
_Why do Poisson regressions sometimes require the use of an offset? Provide an example._

If the data have been collected (events counted) over different sampling times/ distances (exposures) we need an offset to account for this.  For example, we want to compare transposition events across the genome.  One group counted number of transposons per 100KB and the other per 1MB.

## 10M2
_If a coefficient in a Poisson regression has value 1.7, what does this imply about the change in the outcome?_

For every unit change in the predictor there will be a 5.47-fold (e^1.7) increase  in the number of counts 


```r
exp(1)
```

```
## [1] 2.718282
```

```r
exp(1+1.7)
```

```
## [1] 14.87973
```

```r
exp(1+1.7+1.7)
```

```
## [1] 81.45087
```

```r
exp(1)^1.7
```

```
## [1] 5.473947
```

## 10M3
_Explain why the logit link is appropriate for a binomial generalized linear model._

We need to transform the linear model to return a value between 0 and 1 (i.e. the probability scale)

## 10M4
_Explain why the log link is appropriate for a Poisson generalized linear model._

This keeps the outcome variable on a positive scale, required for count data.  You cannot have negative counts.  

(OK but this is true for so much stuff that we model with linear models and Gaussian distributions...)

## 10H4

_The data contained in data(salamanders) are counts of salamanders (Plethodon elongatus) from 47 different 49-m2 plots in northern California.  The column SALAMAN is the count in each plot, and the columns PCTCOVER and FORESTAGE are percent of ground cover and age of trees in the plot, respectively. You will model SALAMAN as a Poisson variable._

_(a) Model the relationship between density and percent cover, using a log-link (same as the example in the book and lecture). Use weakly informative priors of your choosing. Check the quadratic approximation again, by comparing map to map2stan. Then plot the expected counts and their 89% interval against percent cover. In which ways does the model do a good job? In which ways does it do a bad job?_


```r
data("salamanders")
head(salamanders)
```

```
##   SITE SALAMAN PCTCOVER FORESTAGE
## 1    1      13       85       316
## 2    2      11       86        88
## 3    3      11       90       548
## 4    4       9       88        64
## 5    5       8       89        43
## 6    6       7       83       368
```


```r
salamanders %>% select(SALAMAN, PCTCOVER, FORESTAGE) %>% ggpairs()
```

![](Chapter11_files/figure-html/unnamed-chunk-24-1.png)<!-- -->



```r
salamanders$pctcover_std <- scale(salamanders$PCTCOVER)
m10h4.1.quap <- quap(
  alist(SALAMAN ~ dpois(lambda),
        log(lambda) <- alpha + beta_pct*pctcover_std,
        alpha ~ dnorm(3, 0.5),
        beta_pct ~ dnorm(0, 0.2)),
  data=salamanders)
```


```r
precis(m10h4.1.quap)
```

```
##               mean        sd      5.5%     94.5%
## alpha    0.7997914 0.1014376 0.6376745 0.9619082
## beta_pct 0.6535256 0.1064780 0.4833533 0.8236980
```


```r
m10h4.1.stan <- ulam(
  alist(SALAMAN ~ dpois(lambda),
        log(lambda) <- alpha + beta_pct*pctcover_std,
        alpha ~ dnorm(3, 0.5),
        beta_pct ~ dnorm(0, 0.5)),
  chains=4,
  cores=4,
  data=salamanders,
  log_lik = TRUE)
```

```r
precis(m10h4.1.quap)
```

```
##               mean        sd      5.5%     94.5%
## alpha    0.7997914 0.1014376 0.6376745 0.9619082
## beta_pct 0.6535256 0.1064780 0.4833533 0.8236980
```


```r
precis(m10h4.1.stan)
```

```
##               mean        sd      5.5%    94.5%    n_eff     Rhat
## alpha    0.6681484 0.1205835 0.4605302 0.855159 774.7316 1.004938
## beta_pct 0.8932997 0.1456481 0.6729482 1.136029 824.7785 1.001921
```

somewhat similar


```r
pairs(m10h4.1.stan)
```

![](Chapter11_files/figure-html/unnamed-chunk-30-1.png)<!-- -->

```r
trankplot(m10h4.1.stan)
traceplot(m10h4.1.stan)
```

![](Chapter11_files/figure-html/unnamed-chunk-30-2.png)<!-- -->![](Chapter11_files/figure-html/unnamed-chunk-30-3.png)<!-- -->

plot observed and expected

```r
pred <- link(m10h4.1.stan)
pred_obs <- as_tibble(cbind(
  salamanders,
  mu=colMeans(pred),
  low.89=apply(pred,2,HPDI)[1,],
  high.89=apply(pred, 2, HPDI)[2,]))
head(pred_obs)
```

```
## # A tibble: 6 x 8
##    SITE SALAMAN PCTCOVER FORESTAGE pctcover_std[,1]    mu low.89 high.89
##   <int>   <int>    <int>     <int>            <dbl> <dbl>  <dbl>   <dbl>
## 1     1      13       85       316            0.727  3.75   3.21    4.28
## 2     2      11       86        88            0.755  3.85   3.30    4.40
## 3     3      11       90       548            0.867  4.25   3.63    4.92
## 4     4       9       88        64            0.811  4.04   3.47    4.66
## 5     5       8       89        43            0.839  4.15   3.53    4.76
## 6     6       7       83       368            0.671  3.57   3.06    4.06
```


```r
pred_obs %>%
  mutate(observed=SALAMAN,
         predicted=mu) %>%
  ggplot(aes(x=PCTCOVER)) +
  geom_point(aes(y=observed), color="black") +
  geom_pointrange(aes(y=predicted, ymin=low.89, ymax=high.89), color="blue")
```

![](Chapter11_files/figure-html/unnamed-chunk-32-1.png)<!-- -->

Lots of scatter at high PCTCOVER.


```r
pred_obs %>%
  mutate(observed=SALAMAN,
         predicted=mu) %>%
  ggplot(aes(observed,predicted)) +
  geom_point()
```

![](Chapter11_files/figure-html/unnamed-chunk-33-1.png)<!-- -->


_(b) Can you improve the model by using the other predictor, FORESTAGE? Try any models you think useful. Can you explain why FORESTAGE helps or does not help with prediction?_

additive model:


```r
salamanders$forestage_std <- scale(salamanders$FORESTAGE)
m10h4.2 <- ulam(
  alist(SALAMAN ~ dpois(lambda),
        log(lambda) <- alpha + beta_pct*pctcover_std + beta_f*forestage_std,
        alpha ~ dnorm(3, 0.5),
        c(beta_pct,beta_f) ~ dnorm(0, 0.5)),
  chains=4,
  cores=4,
  data=salamanders,
  log_lik = TRUE)
```


```r
precis(m10h4.2)
```

```
##                mean         sd       5.5%     94.5%    n_eff      Rhat
## alpha    0.65390072 0.12003524  0.4593203 0.8418163 1101.500 1.0029776
## beta_f   0.01777924 0.09579743 -0.1416433 0.1651314 1276.042 0.9994673
## beta_pct 0.89306800 0.15784466  0.6434353 1.1440172 1018.115 1.0012334
```


```r
pairs(m10h4.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-36-1.png)<!-- -->

```r
trankplot(m10h4.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-36-2.png)<!-- -->

```r
traceplot(m10h4.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-36-3.png)<!-- -->

plot observed and expected

```r
pred <- link(m10h4.2)
pred_obs <- as_tibble(cbind(
  salamanders,
  mu=colMeans(pred),
  low.89=apply(pred,2,HPDI)[1,],
  high.89=apply(pred, 2, HPDI)[2,]))
pred_obs
```

```
## # A tibble: 47 x 9
##     SITE SALAMAN PCTCOVER FORESTAGE pctcover_std[,1] forestage_std[,…    mu
##    <int>   <int>    <int>     <int>            <dbl>            <dbl> <dbl>
##  1     1      13       85       316            0.727            0.761  3.75
##  2     2      11       86        88            0.755           -0.418  3.78
##  3     3      11       90       548            0.867            1.96   4.38
##  4     4       9       88        64            0.811           -0.542  3.97
##  5     5       8       89        43            0.839           -0.650  4.07
##  6     6       7       83       368            0.671            1.03   3.59
##  7     7       6       83       200            0.671            0.161  3.53
##  8     8       6       91        71            0.895           -0.505  4.28
##  9     9       5       88        42            0.811           -0.655  3.96
## 10    10       5       90       551            0.867            1.98   4.38
## # … with 37 more rows, and 2 more variables: low.89 <dbl>, high.89 <dbl>
```


```r
pred_obs %>%
  mutate(observed=SALAMAN,
         predicted=mu) %>%
  ggplot(aes(x=PCTCOVER)) +
  geom_point(aes(y=observed), color="black") +
  geom_pointrange(aes(y=predicted, ymin=low.89, ymax=high.89), color="blue")
```

![](Chapter11_files/figure-html/unnamed-chunk-38-1.png)<!-- -->


```r
compare(m10h4.1.stan, m10h4.2)
```

```
##                  WAIC    pWAIC    dWAIC    weight       SE      dSE
## m10h4.1.stan 215.0212 3.846960 0.000000 0.8816176 24.47587       NA
## m10h4.2      219.0368 6.773962 4.015676 0.1183824 25.29934 1.452989
```

interaction model:


```r
m10h4.3 <- ulam(
  alist(SALAMAN ~ dpois(lambda),
        log(lambda) <- alpha + 
          beta_pct*pctcover_std + 
          beta_f*forestage_std +
          beta_pct_f*pctcover_std*forestage_std,
        alpha ~ dnorm(3, 0.5),
        c(beta_pct,beta_f,beta_pct_f) ~ dnorm(0, 0.5)),
  chains=4,
  cores=4,
  data=salamanders,
  log_lik = TRUE)
```

```
## Warning: Bulk Effective Samples Size (ESS) is too low, indicating posterior means and medians may be unreliable.
## Running the chains for more iterations may help. See
## http://mc-stan.org/misc/warnings.html#bulk-ess
```


```r
precis(m10h4.3)
```

```
##                  mean        sd        5.5%       94.5%    n_eff     Rhat
## alpha       0.9080070 0.1880808  0.60741822  1.20403913 393.5606 1.015805
## beta_pct_f -0.5170528 0.2884790 -0.98543635 -0.07105061 369.3854 1.012130
## beta_f      0.4199057 0.2438022  0.03573056  0.80979773 356.5620 1.014765
## beta_pct    0.5859567 0.2346886  0.20437666  0.95389616 358.6015 1.017234
```


```r
pairs(m10h4.3)
```

![](Chapter11_files/figure-html/unnamed-chunk-42-1.png)<!-- -->

```r
trankplot(m10h4.3)
traceplot(m10h4.3)
```

![](Chapter11_files/figure-html/unnamed-chunk-42-2.png)<!-- -->![](Chapter11_files/figure-html/unnamed-chunk-42-3.png)<!-- -->

plot observed and expected

```r
pred <- link(m10h4.3)
pred_obs <- as_tibble(cbind(
  salamanders,
  mu=colMeans(pred),
  low.89=apply(pred,2,HPDI)[1,],
  high.89=apply(pred, 2, HPDI)[2,]))
pred_obs
```

```
## # A tibble: 47 x 9
##     SITE SALAMAN PCTCOVER FORESTAGE pctcover_std[,1] forestage_std[,…    mu
##    <int>   <int>    <int>     <int>            <dbl>            <dbl> <dbl>
##  1     1      13       85       316            0.727            0.761  3.95
##  2     2      11       86        88            0.755           -0.418  3.84
##  3     3      11       90       548            0.867            1.96   3.96
##  4     4       9       88        64            0.811           -0.542  4.02
##  5     5       8       89        43            0.839           -0.650  4.13
##  6     6       7       83       368            0.671            1.03   3.99
##  7     7       6       83       200            0.671            0.161  3.74
##  8     8       6       91        71            0.895           -0.505  4.32
##  9     9       5       88        42            0.811           -0.655  4.03
## 10    10       5       90       551            0.867            1.98   3.95
## # … with 37 more rows, and 2 more variables: low.89 <dbl>, high.89 <dbl>
```


```r
pred_obs %>%
  mutate(observed=SALAMAN,
         predicted=mu) %>%
  ggplot(aes(x=PCTCOVER)) +
  geom_point(aes(y=observed), color="black") +
  geom_pointrange(aes(y=predicted, ymin=low.89, ymax=high.89), color="blue")
```

![](Chapter11_files/figure-html/unnamed-chunk-44-1.png)<!-- -->


```r
compare(m10h4.1.stan, m10h4.2)
```

```
##                  WAIC    pWAIC    dWAIC    weight       SE      dSE
## m10h4.1.stan 215.0212 3.846960 0.000000 0.8816176 24.47587       NA
## m10h4.2      219.0368 6.773962 4.015676 0.1183824 25.29934 1.452989
```

## Week6 PDF # 3 

_The data in data(Primates301) were first introduced at the end of Chapter 7. In this problem, you will consider how brain size is associated with social learning._

_There are three parts._

_First, model the number of observations of social_learning for each species as a function of the log brain size. Use a Poisson distribution for the social_learning outcome variable. Interpret the resulting posterior._


```r
data("Primates301")
p <- Primates301 %>% 
  select(genus, species, social_learning, brain, research_effort) %>%
  mutate(l_brain_std = scale(log(brain)),
         l_research_effort_std = scale(log(research_effort)),
         gen_spec = str_c(genus, "_", species)) %>%
  na.omit() %>%
  arrange(gen_spec)

table(p$gen_spec) %>% max() #more than one row per species?  no
```

```
## [1] 1
```

```r
head(p)
```

```
##            genus      species social_learning brain research_effort
## 1 Allenopithecus nigroviridis               0 58.02               6
## 2       Alouatta     belzebul               0 52.84              15
## 3       Alouatta       caraya               0 52.63              45
## 4       Alouatta      guariba               0 51.70              37
## 5       Alouatta     palliata               3 49.88              79
## 6       Alouatta        pigra               0 51.13              25
##   l_brain_std l_research_effort_std                    gen_spec
## 1   0.3725906           -0.73726892 Allenopithecus_nigroviridis
## 2   0.2977140           -0.03690129           Alouatta_belzebul
## 3   0.2945257            0.80282398             Alouatta_caraya
## 4   0.2802512            0.65320644            Alouatta_guariba
## 5   0.2515576            1.23298947           Alouatta_palliata
## 6   0.2713749            0.35354872              Alouatta_pigra
```


```r
p %>%
  select(social_learning, l_brain_std, l_research_effort_std) %>%
  ggpairs()
```

```
## Warning: Continuous x aesthetic -- did you forget aes(group=...)?

## Warning: Continuous x aesthetic -- did you forget aes(group=...)?
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](Chapter11_files/figure-html/unnamed-chunk-47-1.png)<!-- -->


```r
p_small <- list(social_learning=p$social_learning, l_brain_std=as.vector(p$l_brain_std))

m3 <- ulam(
  alist(social_learning ~ dpois(lambda),
        log(lambda) <- alpha + beta_brain*l_brain_std,
        alpha ~ dnorm(3, 0.5),
        beta_brain ~ dnorm(0, .5)),
  data=p_small,
  chains=4,
  cores=4,
  log_lik=T)
```

```
## Warning: Bulk Effective Samples Size (ESS) is too low, indicating posterior means and medians may be unreliable.
## Running the chains for more iterations may help. See
## http://mc-stan.org/misc/warnings.html#bulk-ess
```


```r
precis(m3)
```

```
##                 mean         sd      5.5%      94.5%    n_eff     Rhat
## alpha      -1.053739 0.11178253 -1.236772 -0.8751897 370.6354 1.010051
## beta_brain  2.710217 0.07354519  2.591235  2.8286189 355.5379 1.011461
```

each std deviation increase in log brain size causes a 14.8797317 fold increase in social learning


```r
pairs(m3)
```

![](Chapter11_files/figure-html/unnamed-chunk-50-1.png)<!-- -->

```r
trankplot(m3)
traceplot(m3)
```

![](Chapter11_files/figure-html/unnamed-chunk-50-2.png)<!-- -->![](Chapter11_files/figure-html/unnamed-chunk-50-3.png)<!-- -->


```r
pred <- link(m3)
pred_obs <- as_tibble(
  cbind(p, 
        predicted=colMeans(pred),
        low.89 = apply(pred, 2, HPDI)[1,],
        high.89 = apply(pred, 2, HPDI)[2,]))
```


```r
pred_obs %>%
  ggplot(aes(x=brain)) +
  geom_point(aes(y=social_learning)) +
  geom_pointrange(aes(y=predicted, ymin=low.89, ymax=high.89), color="blue", alpha=.5) +
  scale_x_log10()
```

![](Chapter11_files/figure-html/unnamed-chunk-52-1.png)<!-- -->


_Second, some species are studied much more than others. So the number of reported instances of social_learning could be a product of research effort. Use the research_effort variable, specifically its logarithm, as an additional predictor variable. Interpret the coefficient for log research_effort. Does this model disagree with the previous one?_


```r
p_small <- list(social_learning=p$social_learning, l_brain_std=as.vector(p$l_brain_std), l_research_effort_std=as.vector(p$l_research_effort_std))

m3.2 <- ulam(
  alist(social_learning ~ dpois(lambda),
        log(lambda) <- alpha + beta_brain*l_brain_std + beta_r*l_research_effort_std,
        alpha ~ dnorm(3, 0.5),
        beta_brain ~ dnorm(0, .5),
        beta_r ~ dnorm(0, .5)),
  data=p_small,
  chains=4,
  cores=4,
  log_lik=T)
```


```r
precis(m3.2)
```

```
##                  mean         sd       5.5%      94.5%    n_eff     Rhat
## alpha      -1.5756558 0.13865020 -1.8040054 -1.3646796 747.8917 1.002623
## beta_brain  0.4509311 0.08228079  0.3246575  0.5803572 656.6966 1.002026
## beta_r      1.9510119 0.08217136  1.8190257  2.0883603 608.8215 1.001842
```

The brain size effect is now much smaller, and research effort has a large effect

each std deviation increase in log brain size is associated with a 1.5683122 fold increase in social learning

each std deviation increase in log research effort is associated with a 6.958751 fold increase in social learning



```r
pairs(m3.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-55-1.png)<!-- -->

```r
trankplot(m3.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-55-2.png)<!-- -->

```r
traceplot(m3.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-55-3.png)<!-- -->


```r
pred <- link(m3.2)
pred_obs <- as_tibble(
  cbind(p, 
        predicted=colMeans(pred),
        low.89 = apply(pred, 2, HPDI)[1,],
        high.89 = apply(pred, 2, HPDI)[2,]))
```


```r
pred_obs %>%
  ggplot(aes(x=brain)) +
  geom_point(aes(y=social_learning)) +
  geom_pointrange(aes(y=predicted, ymin=low.89, ymax=high.89), color="blue", alpha=.5) +
  scale_x_log10()
```

![](Chapter11_files/figure-html/unnamed-chunk-57-1.png)<!-- -->


```r
pred_obs %>%
  ggplot(aes(x=research_effort)) +
  geom_point(aes(y=social_learning)) +
  geom_pointrange(aes(y=predicted, ymin=low.89, ymax=high.89), color="blue", alpha=.5) +
  scale_x_log10()
```

![](Chapter11_files/figure-html/unnamed-chunk-58-1.png)<!-- -->


```r
pred_obs %>%
  ggplot(aes(x=social_learning, y=predicted)) +
  geom_point() +
  scale_x_log10() +
  scale_y_log10()
```

```
## Warning: Transformation introduced infinite values in continuous x-axis
```

![](Chapter11_files/figure-html/unnamed-chunk-59-1.png)<!-- -->


_Third, draw a DAG to represent how you think the variables social_learning, brain, and research_effort interact. Justify the DAG with the measured associations in the two models above (and any other models you used)._

There is a backdoor from brain size to research effort


```r
g <- dagitty("dag{
  brain_size -> learning;
  brain_size -> research;
  research -> learning
}")
coordinates(g) <- list(x=c(brain_size=1, learning=1, research=2),
                       y=c(brain_size=0, learning=2, research=1))
plot(g)
```

![](Chapter11_files/figure-html/unnamed-chunk-60-1.png)<!-- -->


