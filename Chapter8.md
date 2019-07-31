---
title: "Chapter8"
author: "Julin N Maloof"
date: "7/30/2019"
output: 
  html_document: 
    keep_md: yes
---



## 7E1
_For each of the causal relationships below, name a hypothetical third variable that would lead to an interaction effect._

_(1) Bread dough rises because of yeast_
Temperature

_(2) Education leads to higher income._
Job? Race? Gender?

_(3) Gasoline makes a car go._
Key? Spark?

## 7M3

_In parts of North America, ravens depend upon wolves for their food. This is because ravens are carnivorous but cannot usually kill or open carcasses of prey. Wolves however can and do kill and tear open animals, and they tolerate ravens co-feeding at their kills. This species relationship is generally described as a “species interaction.” Can you invent a hypothetical set of data on raven population size in which this relationship would manifest as a statistical interaction? Do you think the biological interaction could be linear? Why or why not?_

Ravens ~ Wolves + Prey + Wolves:Prey

## 7H3

_Use the tomato.csv (attached) data set and evaluate whether hypocotyl length ("hyp") is affected by shade ("trt"), species ("species") and their interaction._


```r
d <- read_csv("Tomato.csv") %>%
  select(hyp, trt, species) %>%
  na.omit()
```

```
## Parsed with column specification:
## cols(
##   .default = col_double(),
##   shelf = col_character(),
##   col = col_character(),
##   acs = col_character(),
##   trt = col_character(),
##   date = col_character(),
##   species = col_character(),
##   who = col_character()
## )
```

```
## See spec(...) for full column specifications.
```

```r
d
```

```
## # A tibble: 1,008 x 3
##      hyp trt   species        
##    <dbl> <chr> <chr>          
##  1  19.5 H     S. pennellii   
##  2  31.3 H     S. peruvianum  
##  3  56.6 H     S. peruvianum  
##  4  35.2 H     S. chilense    
##  5  35.3 H     S. chilense    
##  6  28.7 H     S. chmielewskii
##  7  33.1 H     S. habrochaites
##  8  42.1 H     S. pennellii   
##  9  32.7 H     S. peruvianum  
## 10  34.3 H     S. peruvianum  
## # … with 998 more rows
```

Make a plot

```r
d %>%
  group_by(species,trt) %>%
  summarize(mean=mean(hyp), 
            sem=sd(hyp)/sqrt(n()), 
            ymax=mean+sem,
            ymin=mean-sem) %>%
  ggplot(aes(x=species, y=mean, ymax=ymax, ymin=ymin, fill=trt)) +
  geom_col(position = "dodge") +
  geom_errorbar(position = position_dodge(width=.9), width=.5)
```

![](Chapter8_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

make indices for the factors:

```r
d <- d %>%
  mutate(species_i = as.numeric(as.factor(species)),
         trt_i = as.numeric(as.factor(trt))-1)
```


fit non interaction model

```r
m1 <- quap(flist = alist(
  hyp ~ dnorm(mu, sigma),
  mu <- a[species_i] + b*trt_i,
  a[species_i] ~ dnorm(25, 5),
  b ~ dnorm(0, 5),
  sigma ~ dexp(1)),
  data=d, start=list(b=0, sigma=3))
```

check the priors

```r
prior <- extract.prior(m1)
str(prior)
```

```
## List of 3
##  $ b    : num [1:1000(1d)] 4.66 2.92 -3.09 11.38 2.15 ...
##  $ sigma: num [1:1000(1d)] 0.318 1.561 2.741 1.865 0.885 ...
##  $ a    : num [1:1000, 1:5] 27.7 24.3 32.1 22.9 30.3 ...
##  - attr(*, "source")= chr "quap prior: 1000 samples from m1"
```

```r
d2 <- expand.grid(species_i=1:5, trt_i=0:1)
prior.pred <- link(m1, post=prior, data=d2)
colnames(prior.pred) <- str_c("species", d2$species_i, "_trt", d2$trt_i)
prior.pred %>% as_tibble() %>%
  gather() %>%
  ggplot(aes(x=key, y=value)) +
  geom_violin() +
  theme(axis.text.x = element_text(angle=90))
```

![](Chapter8_files/figure-html/unnamed-chunk-5-1.png)<!-- -->



```r
precis(m1, depth=2)
```

```
##            mean        sd      5.5%     94.5%
## b      5.416368 0.5581268  4.524373  6.308362
## sigma  8.978821 0.1986589  8.661325  9.296316
## a[1]  31.901419 0.6791482 30.816009 32.986830
## a[2]  29.215099 0.6579336 28.163594 30.266604
## a[3]  28.461990 0.6477592 27.426746 29.497234
## a[4]  25.801557 0.8273469 24.479296 27.123817
## a[5]  35.557151 0.6683728 34.488962 36.625340
```

now the interaction model:

```r
m2 <- quap(flist = alist(
  hyp ~ dnorm(mu, sigma),
  mu <- a[species_i] + b*trt_i + b_int[species_i]*trt_i,
  a[species_i] ~ dnorm(25, 5),
  b ~ dnorm(0, 5),
  b_int[species_i] ~ dnorm(0, 1),
  sigma ~ dexp(1)),
  data=d, start=list(b=0, sigma=3), control=list(maxit=500))
```


```r
precis(m2, depth=2)
```

```
##                mean        sd       5.5%      94.5%
## b         5.4861578 0.7116510  4.3488020  6.6235135
## sigma     8.9407338 0.1982683  8.6238628  9.2576049
## a[1]     31.6545982 0.7595104 30.4407538 32.8684426
## a[2]     29.6950552 0.7387040 28.5144636 30.8756469
## a[3]     28.8110147 0.7167357 27.6655326 29.9564969
## a[4]     25.1357198 0.9286241 23.6515992 26.6198405
## a[5]     35.3012930 0.7498117 34.1029491 36.4996368
## b_int[1]  0.4274292 0.8264991 -0.8934759  1.7483343
## b_int[2] -1.0089823 0.8195295 -2.3187487  0.3007841
## b_int[3] -0.8156731 0.8196950 -2.1257041  0.4943579
## b_int[4]  1.1803883 0.8657556 -0.2032564  2.5640329
## b_int[5]  0.4362538 0.8224924 -0.8782480  1.7507555
```


```r
compare(m1,m2)
```

```
##        WAIC    pWAIC   dWAIC     weight       SE      dSE
## m2 7303.341 9.058034 0.00000 0.95206672 58.70979       NA
## m1 7309.318 7.686942 5.97765 0.04793328 58.61851 2.588609
```


```r
m3 <- lm(hyp ~ species*trt, data=d)
summary(m3)
```

```
## 
## Call:
## lm(formula = hyp ~ species * trt, data = d)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -29.113  -5.556  -0.733   4.749  36.706 
## 
## Coefficients:
##                             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                  31.5282     0.8907  35.395  < 2e-16 ***
## speciesS. chmielewskii       -0.9555     1.2393  -0.771  0.44090    
## speciesS. habrochaites       -2.0670     1.2139  -1.703  0.08892 .  
## speciesS. pennellii          -7.9579     1.4636  -5.437 6.81e-08 ***
## speciesS. peruvianum          3.7546     1.2507   3.002  0.00275 ** 
## trtL                          6.3654     1.2507   5.090 4.29e-07 ***
## speciesS. chmielewskii:trtL  -3.4551     1.7316  -1.995  0.04627 *  
## speciesS. habrochaites:trtL  -2.9548     1.7322  -1.706  0.08836 .  
## speciesS. pennellii:trtL      3.1772     2.0092   1.581  0.11413    
## speciesS. peruvianum:trtL    -0.1131     1.7486  -0.065  0.94846    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 8.996 on 998 degrees of freedom
## Multiple R-squared:  0.1889,	Adjusted R-squared:  0.1816 
## F-statistic: 25.83 on 9 and 998 DF,  p-value: < 2.2e-16
```


