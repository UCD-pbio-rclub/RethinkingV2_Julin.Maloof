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
## rethinking (Version 1.91)
```

```
## 
## Attaching package: 'rethinking'
```

```
## The following object is masked from 'package:stats':
## 
##     rstudent
```

```r
library(tidyverse)
```

```
## ── Attaching packages ────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ tibble  2.1.3     ✔ purrr   0.3.2
## ✔ tidyr   1.0.0     ✔ dplyr   0.8.3
## ✔ readr   1.3.1     ✔ stringr 1.4.0
## ✔ tibble  2.1.3     ✔ forcats 0.4.0
```

```
## ── Conflicts ───────────────────────────────────── tidyverse_conflicts() ──
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
## a[1] -1.532508 0.06444880 -1.632609 -1.429007 1391.192 1.000347
## a[2] -1.736481 0.08059295 -1.861345 -1.609508 1408.391 0.999907
```

look at differences in award rate

```r
post <- extract.samples(m11.1) 

# relative scale
precis(data.frame(rel_dif=exp(post$a[,2]-post$a[,1])))
```

```
##             mean         sd      5.5%     94.5%    histogram
## rel_dif 0.819823 0.08459473 0.6929348 0.9663226 ▁▁▂▃▇▇▅▂▂▁▁▁
```

```r
#absolute scale
precis(data.frame(prob_dif=inv_logit(post$a[,2])-inv_logit(post$a[,1])))
```

```
##                 mean         sd        5.5%        94.5%  histogram
## prob_dif -0.02777275 0.01392101 -0.04992741 -0.004561252 ▁▁▁▃▇▇▅▂▁▁
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

```
## Warning: Tail Effective Samples Size (ESS) is too low, indicating posterior variances and tail quantiles may be unreliable.
## Running the chains for more iterations may help. See
## http://mc-stan.org/misc/warnings.html#tail-ess
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
## a[1] -1.1949377 0.4250615 -1.9106403 -0.5450413 313.4236 1.007643
## a[2] -1.3330912 0.4266675 -2.0415787 -0.6705171 317.6503 1.007483
## b[1]  0.1875881 0.4586430 -0.5354389  0.9546452 388.6041 1.006099
## b[2] -0.1612351 0.4564259 -0.8624993  0.5908838 340.6427 1.006258
## b[3]  0.1594095 0.4859282 -0.6303840  0.9449390 413.2674 1.007005
## b[4] -0.3823160 0.4421644 -1.0728511  0.3466631 333.7426 1.007602
## b[5] -0.3557451 0.4503802 -1.0667063  0.4007917 356.8286 1.006539
## b[6] -0.4181544 0.4570857 -1.1364828  0.3143766 370.1610 1.006931
## b[7] -0.1480132 0.4433855 -0.8395630  0.5892443 331.4896 1.008476
## b[8] -0.6054813 0.4304089 -1.2703783  0.1128159 325.6695 1.008019
## b[9] -0.4866842 0.4377664 -1.1800209  0.2556796 333.5162 1.007413
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
## a[1]     -0.9703198 0.2063782 -1.3133473 -0.651717725 4195.513 1.0002867
## a[2]     -1.3365192 0.1874059 -1.6421945 -1.042822314 4794.462 0.9999530
## a[3]     -0.9988477 0.2534362 -1.4076838 -0.603679991 4410.247 1.0004452
## a[4]     -1.5550444 0.1393841 -1.7872395 -1.337566267 3415.893 1.0001374
## a[5]     -1.5290669 0.1660396 -1.7997799 -1.273160465 4843.586 1.0006353
## a[6]     -1.5878935 0.2113591 -1.9333473 -1.266777964 4668.769 0.9995471
## a[7]     -1.3152324 0.1514526 -1.5575925 -1.080030725 4150.811 1.0000268
## a[8]     -1.7822967 0.1107202 -1.9589638 -1.602346515 4031.535 0.9998967
## a[9]     -1.6628401 0.1353235 -1.8821914 -1.446011916 4466.892 0.9996882
## b_female -0.1628451 0.1040575 -0.3305328  0.001692421 3060.192 1.0001048
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
##                 mean         sd      5.5%    94.5%      histogram
## rel_female 0.8543403 0.08930556 0.7185408 1.001694 ▁▁▁▂▅▇▇▅▂▁▁▁▁▁
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
## [1,] 0.2286231 0.2330686
## [2,] 0.2919495 0.2543404
## [3,] 0.2876541 0.2183053
## [4,] 0.2675664 0.2456986
## [5,] 0.3310425 0.2917223
## [6,] 0.1999016 0.1804065
```


```r
precis(list(abs_female=pred[,2] - pred[,1]))
```

```
##                   mean         sd        5.5%       94.5%      histogram
## abs_female -0.03090697 0.01980371 -0.06341506 0.000359746 ▁▁▁▂▃▅▇▇▅▃▁▁▁▁
```

Women do 3% worse when accounting for overall differences in award rate between departments, although confidence interval touches 0

Can I do this from posterior directly?


```r
post <- extract.samples(m11.3)
str(post)
```

```
## List of 2
##  $ a       : num [1:4000, 1:9] -1.216 -0.886 -0.907 -1.007 -0.703 ...
##  $ b_female: num [1:4000(1d)] 0.025 -0.19 -0.369 -0.115 -0.184 ...
##  - attr(*, "source")= chr "ulam posterior: 4000 samples from m11.3"
```

```r
# again I should just be able to look at one discipline
precis(list(abd_female=inv_logit(post$a[,1]) -inv_logit(post$a[,1]-post$b_female)))
```

```
##                   mean         sd        5.5%        94.5% histogram
## abd_female -0.03410005 0.02254414 -0.07208479 0.0003595087  ▁▁▂▅▇▅▁▁
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
##                  mean        sd        5.5%       94.5%    n_eff      Rhat
## a[1]     -1.010921615 0.2278512 -1.38223593 -0.65282997 5402.254 0.9993256
## a[2]     -1.388708883 0.2016403 -1.71314378 -1.07317367 4934.253 1.0004456
## a[3]     -1.004996806 0.2588242 -1.42153743 -0.59966690 6123.852 0.9997873
## a[4]     -1.730433359 0.1766835 -2.01586234 -1.45387665 4485.614 0.9993227
## a[5]     -1.611276361 0.1866676 -1.91510194 -1.32222738 4562.826 1.0002278
## a[6]     -1.836190777 0.2573554 -2.25388521 -1.43761175 4513.120 0.9992847
## a[7]     -1.180127451 0.1838976 -1.47458898 -0.89030220 4997.066 0.9993159
## a[8]     -1.721646885 0.1301432 -1.93064101 -1.51545012 5091.197 0.9994332
## a[9]     -1.500000474 0.1588282 -1.75350209 -1.24638522 4820.574 0.9998348
## b_female -0.077821910 0.2135595 -0.42135367  0.26504151 1787.766 1.0024157
## inter[1]  0.007323555 0.3560325 -0.56578134  0.57361629 4010.904 1.0005386
## inter[2]  0.147615977 0.3473272 -0.40516437  0.69195086 4087.727 1.0003160
## inter[3] -0.066573295 0.4289275 -0.75863717  0.61332304 5420.749 1.0001893
## inter[4]  0.315585452 0.2957178 -0.15297066  0.80418860 2683.559 1.0000075
## inter[5]  0.239788346 0.3387887 -0.31119351  0.76623066 3634.264 1.0008047
## inter[6]  0.468172731 0.3476877 -0.08495996  1.01480003 2899.835 0.9998721
## inter[7] -0.442067868 0.3060302 -0.93077176  0.03823043 3032.648 1.0007641
## inter[8] -0.226644499 0.2661432 -0.65723187  0.19943788 2373.664 1.0012194
## inter[9] -0.436383200 0.2886657 -0.89496703  0.03142262 2649.194 1.0014621
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

## 10H3
_The data contained in library(MASS);data(eagles) are records of salmon pirating at- tempts by Bald Eagles in Washington State. See ?eagles for details. While one eagle feeds, some- times another will swoop in and try to steal the salmon from it. Call the feeding eagle the “victim” and the thief the “pirate.” Use the available data to build a binomial GLM of successful pirating attempts._


```r
library(MASS)
```

```
## 
## Attaching package: 'MASS'
```

```
## The following object is masked from 'package:dplyr':
## 
##     select
```

```r
data(eagles)
?eagles
eagles
```

```
##    y  n P A V
## 1 17 24 L A L
## 2 29 29 L A S
## 3 17 27 L I L
## 4 20 20 L I S
## 5  1 12 S A L
## 6 15 16 S A S
## 7  0 28 S I L
## 8  1  4 S I S
```


```r
eagleslist <- with(eagles,
                   list(y=y,
                        n=n,
                        pirate_large=ifelse(P=="L",1,0),
                        pirate_adult=ifelse(A=="A",1,0),
                        victim_large=ifelse(V=="L",1,0)))
str(eagleslist)
```

```
## List of 5
##  $ y           : int [1:8] 17 29 17 20 1 15 0 1
##  $ n           : int [1:8] 24 29 27 20 12 16 28 4
##  $ pirate_large: num [1:8] 1 1 1 1 0 0 0 0
##  $ pirate_adult: num [1:8] 1 1 0 0 1 1 0 0
##  $ victim_large: num [1:8] 1 0 1 0 1 0 1 0
```


```r
m10h3q <- quap(alist(y ~ dbinom(n, p),
                    logit(p) <- alpha + 
                      b_pirate_large*pirate_large +
                      b_pirate_adult*pirate_adult +
                      b_victim_large*victim_large,
                    alpha ~ dnorm(0,10),
                    c(b_pirate_large, b_pirate_adult, b_victim_large) ~ dnorm(0,5)),
              data=eagleslist)
```


```r
precis(m10h3q)
```

```
##                      mean        sd       5.5%     94.5%
## alpha           0.5915456 0.6622748 -0.4668975  1.649989
## b_pirate_large  4.2418208 0.8960187  2.8098099  5.673832
## b_pirate_adult  1.0814174 0.5339215  0.2281077  1.934727
## b_victim_large -4.5926128 0.9613955 -6.1291084 -3.056117
```

```r
pairs(m10h3q)
```

![](Chapter11_files/figure-html/unnamed-chunk-26-1.png)<!-- -->


```r
m10h3stan <- ulam(alist(y ~ dbinom(n, p),
                    logit(p) <- alpha + 
                      b_pirate_large*pirate_large +
                      b_pirate_adult*pirate_adult +
                      b_victim_large*victim_large,
                    alpha ~ dnorm(0,10),
                    c(b_pirate_large, b_pirate_adult, b_victim_large) ~ dnorm(0,5)),
              data=eagleslist,
              chains = 4,
              cores = 4,
              iter = 2000,
              log_lik = TRUE)
```


```r
precis(m10h3stan)
```

```
##                      mean        sd       5.5%     94.5%    n_eff
## alpha           0.6637335 0.6819248 -0.3916924  1.791591 1877.224
## b_victim_large -5.0337388 1.0582749 -6.8364324 -3.482865 1748.239
## b_pirate_adult  1.1334794 0.5532767  0.2564207  2.013208 2002.102
## b_pirate_large  4.6091540 0.9790200  3.1716305  6.254428 1583.631
##                     Rhat
## alpha          1.0008236
## b_victim_large 1.0006981
## b_pirate_adult 0.9996616
## b_pirate_large 1.0005021
```

```r
pairs(m10h3stan)
```

![](Chapter11_files/figure-html/unnamed-chunk-28-1.png)<!-- -->

_(b) Now interpret the estimates. If the quadratic approximation turned out okay, then it’s okay to use the map estimates. Otherwise stick to map2stan estimates. Then plot the posterior predictions. Compute and display both (1) the predicted probability of success and its 89% interval for each row (i) in the data, as well as (2) the predicted success count and its 89% interval. What different information does each type of posterior prediction provide?_

posterior are not entirely Gaussian, stick with Stan

Get predictions


```r
pred <- link(m10h3stan)
head(pred)
```

```
##           [,1]      [,2]      [,3]      [,4]       [,5]      [,6]
## [1,] 0.7399914 0.9992990 0.6046788 0.9986965 0.01094923 0.8472191
## [2,] 0.9107439 0.9997830 0.7442789 0.9992397 0.02903942 0.9310561
## [3,] 0.8431150 0.9997014 0.6749662 0.9992277 0.02814966 0.9474959
## [4,] 0.8350484 0.9994329 0.5999719 0.9980884 0.01668798 0.8552393
## [5,] 0.6702704 0.9937269 0.5844871 0.9909600 0.03953052 0.7623200
## [6,] 0.8925297 0.9993329 0.5585012 0.9956368 0.03301445 0.8603102
##             [,7]      [,8]
## [1,] 0.005914587 0.7487637
## [2,] 0.008458784 0.7939012
## [3,] 0.011068478 0.8745798
## [4,] 0.005002875 0.6364086
## [5,] 0.027691801 0.6893868
## [6,] 0.005173585 0.4840315
```

```r
summary(pred)
```

```
##        V1               V2               V3               V4        
##  Min.   :0.4536   Min.   :0.9673   Min.   :0.2567   Min.   :0.8765  
##  1st Qu.:0.7391   1st Qu.:0.9964   1st Qu.:0.4953   1st Qu.:0.9888  
##  Median :0.7948   Median :0.9982   Median :0.5589   Median :0.9945  
##  Mean   :0.7877   Mean   :0.9972   Mean   :0.5575   Mean   :0.9914  
##  3rd Qu.:0.8433   3rd Qu.:0.9992   3rd Qu.:0.6204   3rd Qu.:0.9975  
##  Max.   :0.9626   Max.   :1.0000   Max.   :0.8572   Max.   :0.9999  
##        V5                  V6               V7                  V8        
##  Min.   :0.0005083   Min.   :0.5331   Min.   :0.0001153   Min.   :0.1512  
##  1st Qu.:0.0214519   1st Qu.:0.7956   1st Qu.:0.0070465   1st Qu.:0.5487  
##  Median :0.0407875   Median :0.8548   Median :0.0135190   Median :0.6568  
##  Mean   :0.0534568   Mean   :0.8425   Mean   :0.0185353   Mean   :0.6455  
##  3rd Qu.:0.0718257   3rd Qu.:0.9015   3rd Qu.:0.0251496   3rd Qu.:0.7509  
##  Max.   :0.3311130   Max.   :0.9886   Max.   :0.1718715   Max.   :0.9729
```
These are the probability of successful pirate for each of the 8 rows in the table.


```r
pred_obs <- as_tibble(cbind(eagles, 
                            mean_prob=colMeans(pred),
                            low.89=apply(pred, 2, HPDI)[1,],
                            high.89=apply(pred, 2, HPDI)[2,]
)) %>%
  mutate(observed_prob=y/n,
         pred_success=mean_prob*n,
         pred_low=low.89*n,
         pred_high=high.89*n,
         label=str_c("P: ", P, ", A: ", A, ", V: ", V)) %>%
  dplyr::select(label, everything())
pred_obs
```

```
## # A tibble: 8 x 13
##   label     y     n P     A     V     mean_prob  low.89 high.89
##   <chr> <int> <int> <fct> <fct> <fct>     <dbl>   <dbl>   <dbl>
## 1 P: L…    17    24 L     A     L        0.788  6.69e-1  0.901 
## 2 P: L…    29    29 L     A     S        0.997  9.94e-1  1.000 
## 3 P: L…    17    27 L     I     L        0.558  4.17e-1  0.703 
## 4 P: L…    20    20 L     I     S        0.991  9.81e-1  1.000 
## 5 P: S…     1    12 S     A     L        0.0535 6.53e-4  0.109 
## 6 P: S…    15    16 S     A     S        0.843  7.31e-1  0.955 
## 7 P: S…     0    28 S     I     L        0.0185 3.20e-4  0.0381
## 8 P: S…     1     4 S     I     S        0.645  4.22e-1  0.868 
## # … with 4 more variables: observed_prob <dbl>, pred_success <dbl>,
## #   pred_low <dbl>, pred_high <dbl>
```


```r
pred_obs %>%
  ggplot(aes(x=label)) +
  geom_pointrange(aes(y=mean_prob, ymin=low.89, ymax=high.89), color="blue", fill="blue") +
  geom_point(aes(y=observed_prob)) +
  theme(axis.text.x = element_text(angle=90))
```

![](Chapter11_files/figure-html/unnamed-chunk-31-1.png)<!-- -->


```r
pred_obs %>%
  ggplot(aes(x=label)) +
  geom_pointrange(aes(y=pred_success, ymin=pred_low, ymax=pred_high), color="blue", fill="blue") +
  geom_point(aes(y=y)) +
  theme(axis.text.x = element_text(angle=90))
```

![](Chapter11_files/figure-html/unnamed-chunk-32-1.png)<!-- -->

Overall the fit looks pretty good.  Size of pirate and victim are both pretty importnat; age is less important.

Try with interaction


```r
m10h3stan_int <- ulam(alist(y ~ dbinom(n, p),
                    logit(p) <- alpha + 
                      b_pirate_large*pirate_large +
                      b_pirate_adult*pirate_adult +
                      b_victim_large*victim_large +
                      bpsa*pirate_large*pirate_adult,
                    alpha ~ dnorm(0,10),
                    c(b_pirate_large, b_pirate_adult, b_victim_large) ~ dnorm(0,5),
                    bpsa ~ dnorm(0,2.5)),
              data=eagleslist,
              chains = 4,
              cores = 4,
              iter = 2000,
              log_lik = TRUE)
```


```r
precis(m10h3stan_int)
```

```
##                     mean        sd      5.5%      94.5%     n_eff     Rhat
## alpha          -0.481587 0.9779743 -2.113790  1.0037005 1055.2746 1.004794
## b_victim_large -5.216552 1.1649907 -7.163285 -3.6131459 1394.6153 1.002421
## b_pirate_adult  2.972590 1.1454467  1.216403  4.8829315  866.6952 1.003420
## b_pirate_large  6.168537 1.3828670  4.125637  8.5053373  879.5320 1.001457
## bpsa           -2.408138 1.2406177 -4.424368 -0.4730717  916.7005 1.003078
```

```r
pairs(m10h3stan_int)
```

![](Chapter11_files/figure-html/unnamed-chunk-34-1.png)<!-- -->


```r
compare(m10h3stan, m10h3stan_int)
```

```
##                   WAIC    pWAIC   dWAIC     weight       SE      dSE
## m10h3stan_int 21.93145 2.535442 0.00000 0.98641948 5.801133       NA
## m10h3stan     30.50234 5.065915 8.57089 0.01358052 7.710244 2.906001
```

Interaction model fits better.  Lets compare the predictions.

```r
pred_int <- link(m10h3stan_int)
head(pred)
```

```
##           [,1]      [,2]      [,3]      [,4]       [,5]      [,6]
## [1,] 0.7399914 0.9992990 0.6046788 0.9986965 0.01094923 0.8472191
## [2,] 0.9107439 0.9997830 0.7442789 0.9992397 0.02903942 0.9310561
## [3,] 0.8431150 0.9997014 0.6749662 0.9992277 0.02814966 0.9474959
## [4,] 0.8350484 0.9994329 0.5999719 0.9980884 0.01668798 0.8552393
## [5,] 0.6702704 0.9937269 0.5844871 0.9909600 0.03953052 0.7623200
## [6,] 0.8925297 0.9993329 0.5585012 0.9956368 0.03301445 0.8603102
##             [,7]      [,8]
## [1,] 0.005914587 0.7487637
## [2,] 0.008458784 0.7939012
## [3,] 0.011068478 0.8745798
## [4,] 0.005002875 0.6364086
## [5,] 0.027691801 0.6893868
## [6,] 0.005173585 0.4840315
```

```r
summary(pred)
```

```
##        V1               V2               V3               V4        
##  Min.   :0.4536   Min.   :0.9673   Min.   :0.2567   Min.   :0.8765  
##  1st Qu.:0.7391   1st Qu.:0.9964   1st Qu.:0.4953   1st Qu.:0.9888  
##  Median :0.7948   Median :0.9982   Median :0.5589   Median :0.9945  
##  Mean   :0.7877   Mean   :0.9972   Mean   :0.5575   Mean   :0.9914  
##  3rd Qu.:0.8433   3rd Qu.:0.9992   3rd Qu.:0.6204   3rd Qu.:0.9975  
##  Max.   :0.9626   Max.   :1.0000   Max.   :0.8572   Max.   :0.9999  
##        V5                  V6               V7                  V8        
##  Min.   :0.0005083   Min.   :0.5331   Min.   :0.0001153   Min.   :0.1512  
##  1st Qu.:0.0214519   1st Qu.:0.7956   1st Qu.:0.0070465   1st Qu.:0.5487  
##  Median :0.0407875   Median :0.8548   Median :0.0135190   Median :0.6568  
##  Mean   :0.0534568   Mean   :0.8425   Mean   :0.0185353   Mean   :0.6455  
##  3rd Qu.:0.0718257   3rd Qu.:0.9015   3rd Qu.:0.0251496   3rd Qu.:0.7509  
##  Max.   :0.3311130   Max.   :0.9886   Max.   :0.1718715   Max.   :0.9729
```
These are the probability of successful pirate for each of the 8 rows in the table.


```r
pred_obs_int <- as_tibble(cbind(pred_obs, 
                            mean_prob_int=colMeans(pred_int),
                            low.89_int=apply(pred_int, 2, HPDI)[1,],
                            high.89_int=apply(pred_int, 2, HPDI)[2,]
)) %>%
  mutate(pred_success_int=mean_prob_int*n,
         pred_low_int=low.89_int*n,
         pred_high_int=high.89_int*n)
pred_obs_int
```

```
## # A tibble: 8 x 19
##   label     y     n P     A     V     mean_prob  low.89 high.89
##   <chr> <int> <int> <fct> <fct> <fct>     <dbl>   <dbl>   <dbl>
## 1 P: L…    17    24 L     A     L        0.788  6.69e-1  0.901 
## 2 P: L…    29    29 L     A     S        0.997  9.94e-1  1.000 
## 3 P: L…    17    27 L     I     L        0.558  4.17e-1  0.703 
## 4 P: L…    20    20 L     I     S        0.991  9.81e-1  1.000 
## 5 P: S…     1    12 S     A     L        0.0535 6.53e-4  0.109 
## 6 P: S…    15    16 S     A     S        0.843  7.31e-1  0.955 
## 7 P: S…     0    28 S     I     L        0.0185 3.20e-4  0.0381
## 8 P: S…     1     4 S     I     S        0.645  4.22e-1  0.868 
## # … with 10 more variables: observed_prob <dbl>, pred_success <dbl>,
## #   pred_low <dbl>, pred_high <dbl>, mean_prob_int <dbl>,
## #   low.89_int <dbl>, high.89_int <dbl>, pred_success_int <dbl>,
## #   pred_low_int <dbl>, pred_high_int <dbl>
```


```r
pred_obs_int %>%
  ggplot(aes(x=label)) +
  geom_pointrange(aes(y=mean_prob, ymin=low.89, ymax=high.89), color="blue", position = position_nudge(x=-.1)) +
  geom_pointrange(aes(y=mean_prob_int, ymin=low.89_int, ymax=high.89_int), color="red", position = position_nudge(x=.1)) +
  geom_point(aes(y=observed_prob)) +
  theme(axis.text.x = element_text(angle=90))
```

![](Chapter11_files/figure-html/unnamed-chunk-38-1.png)<!-- -->


```r
pred_obs_int %>%
  ggplot(aes(x=label)) +
  geom_pointrange(aes(y=pred_success, ymin=pred_low, ymax=pred_high), color="blue", position = position_nudge(x=-.1)) +
  geom_pointrange(aes(y=pred_success_int, ymin=pred_low_int, ymax=pred_high_int), color="red", position = position_nudge(x=.1)) +
  geom_point(aes(y=y)) +
  theme(axis.text.x = element_text(angle=90))
```

![](Chapter11_files/figure-html/unnamed-chunk-39-1.png)<!-- -->

Fits better!

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
salamanders %>% dplyr::select(SALAMAN, PCTCOVER, FORESTAGE) %>% ggpairs()
```

![](Chapter11_files/figure-html/unnamed-chunk-41-1.png)<!-- -->



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
## alpha    0.7997838 0.1014382 0.6376660 0.9619015
## beta_pct 0.6535396 0.1064787 0.4833661 0.8237132
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
## alpha    0.7997838 0.1014382 0.6376660 0.9619015
## beta_pct 0.6535396 0.1064787 0.4833661 0.8237132
```


```r
precis(m10h4.1.stan)
```

```
##               mean        sd      5.5%     94.5%    n_eff     Rhat
## alpha    0.6610284 0.1232501 0.4540424 0.8440612 743.9957 1.001986
## beta_pct 0.9023013 0.1418832 0.6912867 1.1440361 913.8161 1.003632
```

somewhat similar


```r
pairs(m10h4.1.stan)
```

![](Chapter11_files/figure-html/unnamed-chunk-47-1.png)<!-- -->

```r
trankplot(m10h4.1.stan)
traceplot(m10h4.1.stan)
```

![](Chapter11_files/figure-html/unnamed-chunk-47-2.png)<!-- -->![](Chapter11_files/figure-html/unnamed-chunk-47-3.png)<!-- -->

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
## 1     1      13       85       316            0.727  3.75   3.25    4.32
## 2     2      11       86        88            0.755  3.85   3.32    4.42
## 3     3      11       90       548            0.867  4.26   3.62    4.90
## 4     4       9       88        64            0.811  4.04   3.43    4.63
## 5     5       8       89        43            0.839  4.15   3.54    4.77
## 6     6       7       83       368            0.671  3.56   3.08    4.09
```


```r
pred_obs %>%
  mutate(observed=SALAMAN,
         predicted=mu) %>%
  ggplot(aes(x=PCTCOVER)) +
  geom_point(aes(y=observed), color="black") +
  geom_pointrange(aes(y=predicted, ymin=low.89, ymax=high.89), color="blue")
```

![](Chapter11_files/figure-html/unnamed-chunk-49-1.png)<!-- -->

Lots of scatter at high PCTCOVER.


```r
pred_obs %>%
  mutate(observed=SALAMAN,
         predicted=mu) %>%
  ggplot(aes(observed,predicted)) +
  geom_point()
```

![](Chapter11_files/figure-html/unnamed-chunk-50-1.png)<!-- -->


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
##               mean         sd       5.5%     94.5%     n_eff      Rhat
## alpha    0.6593298 0.11965579  0.4659972 0.8419865 1016.3714 1.0005350
## beta_f   0.0172658 0.09028251 -0.1323880 0.1550326 1335.3071 1.0025495
## beta_pct 0.8897137 0.15163408  0.6513304 1.1341934  967.3081 0.9990162
```


```r
pairs(m10h4.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-53-1.png)<!-- -->

```r
trankplot(m10h4.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-53-2.png)<!-- -->

```r
traceplot(m10h4.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-53-3.png)<!-- -->

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
##  1     1      13       85       316            0.727            0.761  3.76
##  2     2      11       86        88            0.755           -0.418  3.79
##  3     3      11       90       548            0.867            1.96   4.38
##  4     4       9       88        64            0.811           -0.542  3.98
##  5     5       8       89        43            0.839           -0.650  4.08
##  6     6       7       83       368            0.671            1.03   3.60
##  7     7       6       83       200            0.671            0.161  3.54
##  8     8       6       91        71            0.895           -0.505  4.29
##  9     9       5       88        42            0.811           -0.655  3.97
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

![](Chapter11_files/figure-html/unnamed-chunk-55-1.png)<!-- -->


```r
compare(m10h4.1.stan, m10h4.2)
```

```
##                  WAIC    pWAIC    dWAIC   weight       SE      dSE
## m10h4.1.stan 215.0667 4.055079 0.000000 0.852044 24.58128       NA
## m10h4.2      218.5682 6.420401 3.501447 0.147956 25.13470 1.219609
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
## alpha       0.9107869 0.1821801  0.62454831  1.19990467 388.1683 1.004211
## beta_pct_f -0.5088073 0.2708950 -0.94385352 -0.07895737 370.0958 1.001334
## beta_f      0.4156641 0.2315581  0.05218414  0.79801988 360.4152 1.002120
## beta_pct    0.5838703 0.2367277  0.20434173  0.95797008 372.1728 1.002778
```


```r
pairs(m10h4.3)
```

![](Chapter11_files/figure-html/unnamed-chunk-59-1.png)<!-- -->

```r
trankplot(m10h4.3)
traceplot(m10h4.3)
```

![](Chapter11_files/figure-html/unnamed-chunk-59-2.png)<!-- -->![](Chapter11_files/figure-html/unnamed-chunk-59-3.png)<!-- -->

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
##  3     3      11       90       548            0.867            1.96   3.99
##  4     4       9       88        64            0.811           -0.542  4.02
##  5     5       8       89        43            0.839           -0.650  4.13
##  6     6       7       83       368            0.671            1.03   4.00
##  7     7       6       83       200            0.671            0.161  3.74
##  8     8       6       91        71            0.895           -0.505  4.32
##  9     9       5       88        42            0.811           -0.655  4.03
## 10    10       5       90       551            0.867            1.98   3.99
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

![](Chapter11_files/figure-html/unnamed-chunk-61-1.png)<!-- -->


```r
compare(m10h4.1.stan, m10h4.2)
```

```
##                  WAIC    pWAIC    dWAIC   weight       SE      dSE
## m10h4.1.stan 215.0667 4.055079 0.000000 0.852044 24.58128       NA
## m10h4.2      218.5682 6.420401 3.501447 0.147956 25.13470 1.219609
```

## Week6 PDF # 3 

_The data in data(Primates301) were first introduced at the end of Chapter 7. In this problem, you will consider how brain size is associated with social learning._

_There are three parts._

_First, model the number of observations of social_learning for each species as a function of the log brain size. Use a Poisson distribution for the social_learning outcome variable. Interpret the resulting posterior._


```r
data("Primates301")
p <- Primates301 %>% 
  dplyr::select(genus, species, social_learning, brain, research_effort) %>%
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
  dplyr::select(social_learning, l_brain_std, l_research_effort_std) %>%
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

![](Chapter11_files/figure-html/unnamed-chunk-64-1.png)<!-- -->


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

```
## Warning: Tail Effective Samples Size (ESS) is too low, indicating posterior variances and tail quantiles may be unreliable.
## Running the chains for more iterations may help. See
## http://mc-stan.org/misc/warnings.html#tail-ess
```


```r
precis(m3)
```

```
##                 mean        sd      5.5%      94.5%    n_eff     Rhat
## alpha      -1.046116 0.1112733 -1.223817 -0.8678478 359.6889 1.007003
## beta_brain  2.704313 0.0745816  2.583570  2.8220920 351.7492 1.005476
```

each std deviation increase in log brain size causes a 14.8797317 fold increase in social learning


```r
pairs(m3)
```

![](Chapter11_files/figure-html/unnamed-chunk-67-1.png)<!-- -->

```r
trankplot(m3)
traceplot(m3)
```

![](Chapter11_files/figure-html/unnamed-chunk-67-2.png)<!-- -->![](Chapter11_files/figure-html/unnamed-chunk-67-3.png)<!-- -->


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

![](Chapter11_files/figure-html/unnamed-chunk-69-1.png)<!-- -->


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
## alpha      -1.5729360 0.13698835 -1.7930772 -1.3516361 745.0240 1.002845
## beta_brain  0.4533455 0.07939778  0.3250634  0.5785741 792.4698 1.004110
## beta_r      1.9475900 0.07980390  1.8169763  2.0717989 702.4556 1.000889
```

The brain size effect is now much smaller, and research effort has a large effect

each std deviation increase in log brain size is associated with a 1.5683122 fold increase in social learning

each std deviation increase in log research effort is associated with a 6.958751 fold increase in social learning



```r
pairs(m3.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-72-1.png)<!-- -->

```r
trankplot(m3.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-72-2.png)<!-- -->

```r
traceplot(m3.2)
```

![](Chapter11_files/figure-html/unnamed-chunk-72-3.png)<!-- -->


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

![](Chapter11_files/figure-html/unnamed-chunk-74-1.png)<!-- -->


```r
pred_obs %>%
  ggplot(aes(x=research_effort)) +
  geom_point(aes(y=social_learning)) +
  geom_pointrange(aes(y=predicted, ymin=low.89, ymax=high.89), color="blue", alpha=.5) +
  scale_x_log10()
```

![](Chapter11_files/figure-html/unnamed-chunk-75-1.png)<!-- -->


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

![](Chapter11_files/figure-html/unnamed-chunk-76-1.png)<!-- -->


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

![](Chapter11_files/figure-html/unnamed-chunk-77-1.png)<!-- -->


