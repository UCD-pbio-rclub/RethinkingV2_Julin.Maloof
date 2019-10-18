---
title: "Untitled"
author: "Julin N Maloof"
date: "10/18/2019"
output: 
  html_document: 
    keep_md: yes
---



## 1. 
Consider the data(Wines2012) data table. These data are expert ratings
of 20 different French and American wines by 9 different French and American judges. Your goal is to model score, the subjective rating assigned by
each judge to each wine. I recommend standardizing it.
In this first problem, consider only variation among judges and wines.
Construct index variables of judge and wine and then use these index variables to construct a linear regression model. Justify your priors. You should
end up with 9 judge parameters and 20 wine parameters. Use ulam instead of
quap to build this model, and be sure to check the chains for convergence. If
you’d rather build the model directly in Stan or PyMC3, go ahead. I just want
you to use Hamiltonian Monte Carlo instead of quadratic approximation.
How do you interpret the variation among individual judges and individual wines? Do you notice any patterns, just by plotting the differences?
Which judges gave the highest/lowest ratings? Which wines were rated worst/
best on average?


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
## ── Attaching packages ───────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ tibble  2.1.3     ✔ purrr   0.3.2
## ✔ tidyr   1.0.0     ✔ dplyr   0.8.3
## ✔ readr   1.3.1     ✔ stringr 1.4.0
## ✔ tibble  2.1.3     ✔ forcats 0.4.0
```

```
## ── Conflicts ──────────────────────────────────── tidyverse_conflicts() ──
## ✖ tidyr::extract() masks rstan::extract()
## ✖ dplyr::filter()  masks stats::filter()
## ✖ dplyr::lag()     masks stats::lag()
## ✖ purrr::map()     masks rethinking::map()
```

```r
data("Wines2012")
head(Wines2012)
```

```
##             judge flight wine score wine.amer judge.amer
## 1 Jean-M Cardebat  white   A1    10         1          0
## 2 Jean-M Cardebat  white   B1    13         1          0
## 3 Jean-M Cardebat  white   C1    14         0          0
## 4 Jean-M Cardebat  white   D1    15         0          0
## 5 Jean-M Cardebat  white   E1     8         1          0
## 6 Jean-M Cardebat  white   F1    13         1          0
```





```r
Wines2012$score_std <- scale(Wines2012$score)
wlist <- with(Wines2012,
              list(judge=as.numeric(judge),
                   wine=as.numeric(wine),
                   score = score_std))
```


```r
m1 <- ulam(alist(
  score ~ dnorm(mu, sigma),
  mu <- a_j[judge] + a_w[wine],
  a_j[judge] ~ dnorm(0,1),
  a_w[wine] ~ dnorm(0,1),
  sigma ~ dexp(1)),
  data = wlist,
  chains = 4,
  cores = 4,
  iter=2000)
```

are the priors reasonable?

```r
priors <- extract.prior(m1)
```

```
## 
## SAMPLING FOR MODEL 'cf1f3dd733966f44a73175906cfe0fc6' NOW (CHAIN 1).
## Chain 1: 
## Chain 1: Gradient evaluation took 2.1e-05 seconds
## Chain 1: 1000 transitions using 10 leapfrog steps per transition would take 0.21 seconds.
## Chain 1: Adjust your expectations accordingly!
## Chain 1: 
## Chain 1: 
## Chain 1: Iteration:    1 / 2000 [  0%]  (Warmup)
## Chain 1: Iteration:  200 / 2000 [ 10%]  (Warmup)
## Chain 1: Iteration:  400 / 2000 [ 20%]  (Warmup)
## Chain 1: Iteration:  600 / 2000 [ 30%]  (Warmup)
## Chain 1: Iteration:  800 / 2000 [ 40%]  (Warmup)
## Chain 1: Iteration: 1000 / 2000 [ 50%]  (Warmup)
## Chain 1: Iteration: 1001 / 2000 [ 50%]  (Sampling)
## Chain 1: Iteration: 1200 / 2000 [ 60%]  (Sampling)
## Chain 1: Iteration: 1400 / 2000 [ 70%]  (Sampling)
## Chain 1: Iteration: 1600 / 2000 [ 80%]  (Sampling)
## Chain 1: Iteration: 1800 / 2000 [ 90%]  (Sampling)
## Chain 1: Iteration: 2000 / 2000 [100%]  (Sampling)
## Chain 1: 
## Chain 1:  Elapsed Time: 0.161622 seconds (Warm-up)
## Chain 1:                0.144254 seconds (Sampling)
## Chain 1:                0.305876 seconds (Total)
## Chain 1:
```

```r
str(priors)
```

```
## List of 3
##  $ a_j  : num [1:1000, 1:9] 1.385 0.8738 0.0718 -0.2941 1.1683 ...
##  $ a_w  : num [1:1000, 1:20] 0.0569 -0.4817 1.1051 -0.1848 -0.6394 ...
##  $ sigma: num [1:1000(1d)] 1.368 2.269 0.548 1.967 2.127 ...
##  - attr(*, "source")= chr "ulam prior: 1000 samples from fit"
```

each "score" would be the sum of a wine and a judge, so we could just run through some of these

```r
data <- data.frame(judge=sample(1:9, 1000, replace = TRUE), 
                   wine=sample(1:20, 1000, replace = TRUE))
prior.score <- link(m1,data,post=priors)
dens(prior.score)
```

![](Chapter9_moreHW_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
dens(wlist$score)
```

![](Chapter9_moreHW_files/figure-html/unnamed-chunk-5-2.png)<!-- -->

Reasonable...


```r
precis(m1, depth=2)
```

```
##                 mean         sd        5.5%       94.5%    n_eff      Rhat
## a_j[1]  -0.311374149 0.25868020 -0.71306672  0.10097874 1088.055 1.0055044
## a_j[2]   0.227296668 0.25566538 -0.18156173  0.63898242 1058.779 1.0051585
## a_j[3]   0.217272567 0.26039104 -0.18757344  0.63625369 1112.269 1.0031609
## a_j[4]  -0.601668728 0.25626146 -1.00786845 -0.19338309 1112.620 1.0034044
## a_j[5]   0.875269289 0.25448701  0.47067283  1.27249429 1128.456 1.0042310
## a_j[6]   0.521957421 0.25755485  0.10321584  0.93230277 1094.311 1.0055603
## a_j[7]   0.141502769 0.25762665 -0.27306718  0.54870223 1106.151 1.0061085
## a_j[8]  -0.729053251 0.26131784 -1.13995836 -0.30900922 1175.131 1.0045622
## a_j[9]  -0.384170566 0.25365061 -0.78444814  0.01772697 1080.699 1.0043986
## a_w[1]   0.144865498 0.32295115 -0.36189348  0.65819897 1656.587 1.0036035
## a_w[2]   0.108268328 0.32413838 -0.40838439  0.62118687 1457.170 1.0034815
## a_w[3]   0.287503083 0.31900071 -0.23386727  0.79219075 1615.916 1.0027388
## a_w[4]   0.571476954 0.32350188  0.05620140  1.08616391 1604.962 1.0041292
## a_w[5]  -0.124463345 0.32654790 -0.64148118  0.39079064 1762.938 1.0017649
## a_w[6]  -0.370449978 0.32441413 -0.89694937  0.14724277 1609.655 1.0025833
## a_w[7]   0.309215777 0.32539965 -0.20141142  0.83352464 1634.999 1.0021387
## a_w[8]   0.281938863 0.32416512 -0.23140119  0.78288901 1508.574 1.0026142
## a_w[9]   0.093247164 0.32099816 -0.42292742  0.60535838 1649.769 1.0019528
## a_w[10]  0.128400866 0.31847880 -0.38452027  0.62935684 1495.907 1.0043828
## a_w[11] -0.007056486 0.32894335 -0.52419639  0.52147167 1617.422 1.0017948
## a_w[12] -0.027831763 0.32707445 -0.55225505  0.50177324 1690.641 1.0030443
## a_w[13] -0.097958634 0.33250246 -0.61531054  0.44268636 1643.077 1.0027336
## a_w[14]  0.013642336 0.31805228 -0.49340218  0.51615271 1649.092 1.0025658
## a_w[15] -0.213134148 0.31549180 -0.71007723  0.30012339 1778.809 1.0021408
## a_w[16] -0.196410280 0.32998272 -0.71728711  0.32642559 1623.016 1.0020252
## a_w[17] -0.140530801 0.32064865 -0.64197642  0.37251621 1648.993 1.0036142
## a_w[18] -0.871005660 0.32675205 -1.39495843 -0.34636314 1797.444 1.0015572
## a_w[19] -0.166982041 0.32587066 -0.68645674  0.33957391 1618.338 1.0020548
## a_w[20]  0.400851841 0.32280151 -0.09913348  0.91536171 1625.625 1.0041934
## sigma    0.852464850 0.04770495  0.78106660  0.93198191 4252.734 0.9993189
```

```r
plot(precis(m1, depth=2))
```

![](Chapter9_moreHW_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
traceplot(m1, ask=FALSE)
```

```
## Waiting to draw page 2 of 2
```

![](Chapter9_moreHW_files/figure-html/unnamed-chunk-6-2.png)<!-- -->![](Chapter9_moreHW_files/figure-html/unnamed-chunk-6-3.png)<!-- -->

```r
trankplot(m1, ask=FALSE)
```

```
## Waiting to draw page 2 of 2
```

![](Chapter9_moreHW_files/figure-html/unnamed-chunk-6-4.png)<!-- -->![](Chapter9_moreHW_files/figure-html/unnamed-chunk-6-5.png)<!-- -->

More varitaion in judges than in wines.  Only 2 wines seem to be different from mean, but maybe 4 judges.

## 2. Now consider three features of the wines and judges:
(1) flight: Whether the wine is red or white.
(2) wine.amer: Indicator variable for American wines.
(3) judge.amer: Indicator variable for American judges.
Use indicator or index variables to model the influence of these features on
the scores. Omit the individual judge and wine index variables from Problem
1. Do not include interaction effects yet. Again use ulam, justify your priors,
and be sure to check the chains. What do you conclude about the differences
among the wines and judges? Try to relate the results to the inferences in
Problem 1.


```r
wlist <- with(Wines2012,
              list(score = score_std,
                   red=as.numeric(flight),
                   wine_amer=wine.amer+1,
                   judge_amer=judge.amer+1
              ))

m2 <- ulam(alist(
  score ~ dnorm(mu, sigma),
  mu <- a_red[red] + a_w_amer[wine_amer] + a_j_amer[judge_amer],
  a_red[red] ~ dnorm(0,1),
  a_w_amer[wine_amer] ~ dnorm(0,1),
  a_j_amer[judge_amer] ~ dnorm(0,1),
  sigma ~ dexp(1)),
  data = wlist,
  chains = 4,
  cores = 4,
  iter=2000)
```


```r
precis(m2, depth=2)
```

```
##                     mean         sd       5.5%     94.5%    n_eff
## a_red[1]    -0.013837579 0.58033401 -0.9390828 0.9118710 1563.083
## a_red[2]    -0.009211218 0.58266477 -0.9353479 0.9012773 1549.073
## a_w_amer[1]  0.099478498 0.57595476 -0.8039962 1.0258874 1561.699
## a_w_amer[2] -0.092427316 0.57657547 -0.9917831 0.8380773 1597.149
## a_j_amer[1] -0.106337469 0.58260725 -1.0281332 0.8322306 1557.502
## a_j_amer[2]  0.137316025 0.57931400 -0.7722742 1.0715112 1543.665
## sigma        1.000078896 0.05434088  0.9167102 1.0908165 2280.467
##                  Rhat
## a_red[1]    1.0022656
## a_red[2]    1.0020357
## a_w_amer[1] 1.0024212
## a_w_amer[2] 1.0023756
## a_j_amer[1] 1.0012898
## a_j_amer[2] 1.0014110
## sigma       0.9996553
```

```r
plot(precis(m2, depth=2))
```

![](Chapter9_moreHW_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
traceplot(m2, ask=FALSE)
trankplot(m2, ask=FALSE)
```

![](Chapter9_moreHW_files/figure-html/unnamed-chunk-8-2.png)<!-- -->![](Chapter9_moreHW_files/figure-html/unnamed-chunk-8-3.png)<!-- -->

No effect of flight, or country of origin for wine or judge.

## 3. Now consider two-way interactions among the three features. 
You should end up with three different interaction terms in your model. These will be
easier to build, if you use indicator variables. Again use ulam, justify your
priors, and be sure to check the chains. Explain what each interaction means.
Be sure to interpret the model’s predictions on the outcome scale (mu, the
expected score), not on the scale of individual parameters. You can use link
to help with this, or just use your knowledge of the linear model instead.
What do you conclude about the features and the scores? Can you relate
the results of your model(s) to the individual judge and wine inferences from
Problem 1?


```r
m3 <- ulam(alist(
  score ~ dnorm(mu, sigma),
  mu <- a_red[red] + 
    a_w_amer[wine_amer] + 
    a_j_amer[judge_amer] + 
    i_red_w_amer[red]*(wine_amer-1) + # 0 for french wines, 1 for american wines
    i_red_j_amer[red]*(wine_amer-1) + # 0 for french wines, 1 for american wines
    i_w_j_amer[wine_amer]*(judge_amer-1), # 0 for french judges, 1 for american judges
  a_red[red] ~ dnorm(0,1),
  a_w_amer[wine_amer] ~ dnorm(0,1),
  a_j_amer[judge_amer] ~ dnorm(0,1),
  i_red_w_amer[red] ~ dnorm(0,.5),
  i_red_j_amer[red] ~ dnorm(0,.5),
  i_w_j_amer[wine_amer] ~ dnorm(0,.5),
  sigma ~ dexp(1)),
  data = wlist,
  chains = 4,
  cores = 4,
  iter=2000)
```


```r
precis(m3, depth=2)
```

```
##                        mean         sd       5.5%     94.5%    n_eff
## a_red[1]         0.15084405 0.61329641 -0.8208611 1.1257459 1997.274
## a_red[2]        -0.16041570 0.61523691 -1.1446059 0.8192677 2068.840
## a_w_amer[1]      0.08295765 0.62139499 -0.8883045 1.0734465 2456.092
## a_w_amer[2]     -0.04452207 0.65914289 -1.1010758 1.0059321 2753.426
## a_j_amer[1]     -0.13233893 0.59296388 -1.0556851 0.7983070 2263.414
## a_j_amer[2]      0.10309759 0.62705115 -0.8877506 1.0807400 2216.259
## i_red_w_amer[1] -0.14690159 0.43056677 -0.8503944 0.5345541 3533.231
## i_red_w_amer[2]  0.12830999 0.44018639 -0.5770087 0.8289469 3793.645
## i_red_j_amer[1] -0.12786075 0.42448201 -0.8065648 0.5473183 4266.392
## i_red_j_amer[2]  0.12938571 0.42890099 -0.5438034 0.8362311 4262.969
## i_w_j_amer[1]    0.07174188 0.36987999 -0.5148888 0.6677050 3164.292
## i_w_j_amer[2]   -0.02992628 0.36213781 -0.6052239 0.5620533 3222.826
## sigma            0.99317524 0.05429978  0.9114066 1.0823954 5333.055
##                      Rhat
## a_red[1]        1.0005516
## a_red[2]        1.0006566
## a_w_amer[1]     1.0002122
## a_w_amer[2]     1.0003251
## a_j_amer[1]     1.0006075
## a_j_amer[2]     0.9998733
## i_red_w_amer[1] 1.0009447
## i_red_w_amer[2] 0.9994889
## i_red_j_amer[1] 1.0002320
## i_red_j_amer[2] 0.9995806
## i_w_j_amer[1]   1.0000186
## i_w_j_amer[2]   1.0019278
## sigma           0.9991032
```

```r
plot(precis(m3, depth=2))
```

![](Chapter9_moreHW_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
traceplot(m3, ask=FALSE)
trankplot(m3, ask=FALSE)
```

![](Chapter9_moreHW_files/figure-html/unnamed-chunk-10-2.png)<!-- -->![](Chapter9_moreHW_files/figure-html/unnamed-chunk-10-3.png)<!-- -->

I don't see anything inersting here:

* a_red: score of white or red wines
* a_w_amer: score of french or american wines
* a_j_amer: score of french or american judges
* i_red_w_amer: interaction effect of red on french vs american wines (the amount that a white or red score shifts when a  wine is from America 
* i_red_j_amer: interaction effect of red on french vs american wines (the amount that a white or red score shifts when a  judge is from America )
* i_w_j_amer: interaction effect of wine and judge country.  The amount that the score shifts when an american or french wine is judged by an american.


