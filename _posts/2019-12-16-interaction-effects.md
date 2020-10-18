---
title: "Marginal effects and interaction terms"
excerpt: "Quickly get the full marginal effect of interaction terms in R (and other software)"
tags: [interaction terms, marginal effects]
# toc: true
# toc_label: "Jump to:"
comments: true
---

I recently [tweeted](https://twitter.com/grant_mcdermott/status/1202084676439085056?s=20){:target="_blank"} one of my favourite R tricks for getting the full marginal effect(s) of interaction terms. The short version is that, instead of writing your model as `lm(y ~ f1 * x2)`, you write it as `lm(y ~ f1 / x2)`. Here's an example using everyone's favourite mtcars dataset.

First, partial marginal effects with the standard `f1 * x2` interaction syntax.

```r
summary(lm(mpg ~ factor(am) * wt, data = mtcars))
#> 
#> Call:
#> lm(formula = mpg ~ factor(am) * wt, data = mtcars)
#> 
#> Residuals:
#>     Min      1Q  Median      3Q     Max 
#> -3.6004 -1.5446 -0.5325  0.9012  6.0909 
#> 
#> Coefficients:
#>                Estimate Std. Error t value Pr(>|t|)    
#> (Intercept)     31.4161     3.0201  10.402 4.00e-11 ***
#> factor(am)1     14.8784     4.2640   3.489  0.00162 ** 
#> wt              -3.7859     0.7856  -4.819 4.55e-05 ***
#> factor(am)1:wt  -5.2984     1.4447  -3.667  0.00102 ** 
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 2.591 on 28 degrees of freedom
#> Multiple R-squared:  0.833,  Adjusted R-squared:  0.8151 
#> F-statistic: 46.57 on 3 and 28 DF,  p-value: 5.209e-11
```

Second, full marginal effects with the trick `f1 / x2` interaction syntax.

```r
summary(lm(mpg ~ factor(am) / wt, data = mtcars))
#> 
#> Call:
#> lm(formula = mpg ~ factor(am)/wt, data = mtcars)
#> 
#> Residuals:
#>     Min      1Q  Median      3Q     Max 
#> -3.6004 -1.5446 -0.5325  0.9012  6.0909 
#> 
#> Coefficients:
#>                Estimate Std. Error t value Pr(>|t|)    
#> (Intercept)     31.4161     3.0201  10.402 4.00e-11 ***
#> factor(am)1     14.8784     4.2640   3.489  0.00162 ** 
#> factor(am)0:wt  -3.7859     0.7856  -4.819 4.55e-05 ***
#> factor(am)1:wt  -9.0843     1.2124  -7.493 3.68e-08 ***
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 2.591 on 28 degrees of freedom
#> Multiple R-squared:  0.833,  Adjusted R-squared:  0.8151 
#> F-statistic: 46.57 on 3 and 28 DF,  p-value: 5.209e-11
```

To get the full marginal effect of `factor(am)1:wt` in the first case, I have to manually sum up the coefficients on the constituent parts (i.e. `factor(am)1=14.8784` + `factor(am)1:wt=-5.2984`). In the second case, I get the full marginal effect of **&minus;9.0843** immediately in the model summary. Not only that, but the correct standard errors, p-values, etc. are also automatically calculated for me. (If you don't remember, manually calculating SEs for multiplicative interaction terms is a [huge](http://mattgolder.com/wp-content/uploads/2015/05/standarderrors1.png){:target="_blank"} [pain](http://mattgolder.com/wp-content/uploads/2015/05/standarderrors2.png){:target="_blank"}. And that's before we even get to complications like standard error clustering.)

Note that the `lm(y ~ f1 / x2)` syntax is actually shorthand for the more verbose `lm(y ~ f1 + f1:x2)`. I'll get back to this point further below, but I wanted to flag the expanded syntax as important because it demonstrates why this trick "works". The key idea is to drop the continuous variable parent term (here: `x2`) from the regression. This forces all of the remaining child terms relative to the same base. It's also why this trick can easily be adapted to, say, Julia or Stata (see [here](https://twitter.com/paulgp/status/1202085605116665856){:target="_blank"}).

So far, so good. It's a great trick that has saved me a bunch of time (say nothing of likely user-error) that I recommend to everyone. Yet, I was prompted to write a separate blog post after being asked whether this trick a) works for higher-order interactions, and b) other non-linear models like logit? The answer in both cases is a happy "Yes!".

## Dealing with higher-order interactions

Let's consider a threeway interaction, since this will demonstrate the general principle for higher-order interactions. First, as a matter of convenience, I'll create a new dataset so that I don't have to keep specifying the factor variables.

```r
library(tidyverse)
df = 
  mtcars %>%
  mutate(vs = factor(vs), am = factor(am))
```

Now, we run a threeway interaction and view the (naive, partial) marginal effects.

```r
fit1 = lm(mpg ~ vs * am * wt, data = df) 
## Naive coeficients
summary(fit1)
#> 
#> Call:
#> lm(formula = mpg ~ vs * am * wt, data = df)
#> 
#> Residuals:
#>     Min      1Q  Median      3Q     Max 
#> -3.3055 -1.7152 -0.7279  1.3504  5.3624 
#> 
#> Coefficients:
#>             Estimate Std. Error t value Pr(>|t|)    
#> (Intercept)  25.0594     4.0397   6.203 2.07e-06 ***
#> vs1           6.4677    10.1440   0.638   0.5298    
#> am1          17.3041     7.7041   2.246   0.0342 *  
#> wt           -2.4389     0.9689  -2.517   0.0189 *  
#> vs1:am1      -4.7049    12.9763  -0.363   0.7201    
#> vs1:wt       -0.9372     3.0560  -0.307   0.7617    
#> am1:wt       -5.4749     2.4667  -2.220   0.0362 *  
#> vs1:am1:wt    1.0833     4.4419   0.244   0.8094    
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 2.469 on 24 degrees of freedom
#> Multiple R-squared:  0.8701, Adjusted R-squared:  0.8322 
#> F-statistic: 22.96 on 7 and 24 DF,  p-value: 3.533e-09
```

Say we are interested in the full marginal effect of the threeway interaction `vs1:am1:wt`. Even summing the correct parent coefficients is a potentially error-prone process of thinking through the underlying math (which terms are excluded from the partial derivative, etc.) And don't even get me started on the standard errors...

Now, it should be said that there _are_ several existing tools for obtaining this number that don't require us working through everything by hand. Here I'll use my favourite such tool &mdash; the [**margins**](https://cran.r-project.org/web/packages/margins/vignettes/Introduction.html){:target="_blank"} package &mdash; to save me the mental arithmetic.
```r
library(margins)
## Evaluate the marginal effect of `wt` at vs = 1 and am = 1
fit1 %>%
  margins(
    variables = "wt",
    at = list(vs = "1", am = "1")
    ) %>%
  summary()
#>  factor     vs     am     AME     SE       z      p    lower   upper
#>      wt 1.0000 1.0000 -7.7676 2.2903 -3.3916 0.0007 -12.2565 -3.2788
```

We now at least see that the full (average) marginal effect is **&minus;7.7676**. Still, while this approach works well in the present example, we can also begin to see some downsides. It requires extra coding steps and comes with its own specialised syntax. Moreover, underneath the hood, **margins** relies on a numerical delta method that can dramatically increase computation time and memory use for even moderately sized real-world problems. (Is your dataset bigger than 1 GB? [Good luck](https://github.com/leeper/margins/issues/130){:target="_blank"}.) Another practical problem is that **margins** may not even support your model class. (See [here](https://github.com/leeper/margins/issues/101){:target="_blank"}.)

So, what about the alternative? Does our little syntax trick work here too? The good news is that, yes, it's just as simple as it was before.

```r
fit2 = lm(mpg ~ vs / am / wt, data = df)
summary(fit2)
#> 
#> Call:
#> lm(formula = mpg ~ vs/am/wt, data = df)
#> 
#> Residuals:
#>     Min      1Q  Median      3Q     Max 
#> -3.3055 -1.7152 -0.7279  1.3504  5.3624 
#> 
#> Coefficients:
#>             Estimate Std. Error t value Pr(>|t|)    
#> (Intercept)  25.0594     4.0397   6.203 2.07e-06 ***
#> vs1           6.4677    10.1440   0.638  0.52978    
#> vs0:am1      17.3041     7.7041   2.246  0.03417 *  
#> vs1:am1      12.5993    10.4418   1.207  0.23934    
#> vs0:am0:wt   -2.4389     0.9689  -2.517  0.01891 *  
#> vs1:am0:wt   -3.3761     2.8983  -1.165  0.25552    
#> vs0:am1:wt   -7.9138     2.2685  -3.489  0.00190 ** 
#> vs1:am1:wt   -7.7676     2.2903  -3.392  0.00241 ** 
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 2.469 on 24 degrees of freedom
#> Multiple R-squared:  0.8701, Adjusted R-squared:  0.8322 
#> F-statistic: 22.96 on 7 and 24 DF,  p-value: 3.533e-09
```

Again, we get the full marginal effect of **&minus;7.7676** (and correct SE of 2.2903) directly in the model object. Much easier, isn't it?

Where this approach really shines is in combination with plotting. Say, after extracting the coefficients with `broom::tidy()`. Model results are usually much easier to interpret visually, but this is precisely where we want to depict full marginal effects to our reader. In the below example, we immediately get a sense of how automatic transmission exacerbates the impact of vehicle weight on MPG, while the conditional impact of engine shape is more ambiguous. In contrast, I invite you to try the same plot on the `fit1` object and see if you can easily make sense of it. I certainly can't.

```r
library(broom)
library(hrbrthemes) ## theme(s) I like

tidy(fit2, conf.int = T) %>%
  filter(grepl("wt", term)) %>%
  ## Optional regexp work to make plot look nicier  
  mutate(
    am = ifelse(grepl("am1", term), "Automatic", "Manual"),
    vs = ifelse(grepl("vs1", term), "V-shaped", "Straight"),
    x_lab = paste(am, vs, sep="\n")
    ) %>%
  ggplot(aes(x=x_lab, y=estimate, ymin=conf.low, ymax=conf.high)) +
  geom_pointrange() +
  geom_hline(yintercept = 0, col = "orange") +
  labs(
    x = NULL, y = "Marginal effect (Δ in MPG : Δ in '000 lbs)",
    title = " Marginal effect of vehicle weight on MPG", 
    subtitle = "Conditional on transmission type and engine shape"
    ) +
  theme_ipsum() 
```

![](https://i.imgur.com/0aFoZcy.png)


## Aside: Specifying (parent) terms as fixed effects

On the subject of speed, recall that the `lm(y ~ f1 / x2)` syntax is equivalent to the more verbose `lm(y ~ f1 + f1:x2)`. This verbose syntax provides a clue for greatly reducing computation time for large models; particularly those with factor variables that contain many levels. We simply need specify the parent factor terms as _fixed effects_ (using a specialised library like [**lfe**](https://cran.r-project.org/web/packages/lfe/index.html) or [**fixest**](https://github.com/lrberge/fixest/wiki)). Going back to our introductory twoway interaction example, you would thus write the model as follows. 

```r
library(fixest)
feols(mpg ~ am:wt | am, data = df)

## Another option...
library(lfe)
felm(mpg ~ am:wt | am, data = df)
``` 

(I'll let you confirm for yourself that running either of the above models gives the correct &minus;9.0843 figure from before.)

In case you're wondering, the verbose equivalent for the `f1 / f2 / x3` threeway interaction is `f1 + f2 + f1:f2 + f1:f2:x3`. So we can use the same FE approach for this more complicated case as follows:

```r
## Option 1 using verbose base lm(). Not run.
# summary(lm(mpg ~ vs + am + vs:am + vs:am:wt, data = df))

## Option 2 using lfe::felm(). Also not run.
# summary(felm(mpg ~ vs:am:wt | vs + am + vs:am, data = df))

## Option 3 using fixest::feols()
feols(mpg ~ vs:am:wt | vs + am + vs^am, data = df)
#> OLS estimation, Dep. Var.: mpg
#> Observations: 32 
#> Fixed-effects: vs: 2,  am: 2,  vs^am: 4
#> Standard-errors: Clustered (vs) 
#>            Estimate Std. Error       z value  Pr(>|z|)    
#> vs0:am0:wt  -2.4389   1.01e-15 -2.407093e+15 < 2.2e-16 ***
#> vs1:am0:wt  -3.3761   1.03e-15 -3.286044e+15 < 2.2e-16 ***
#> vs0:am1:wt  -7.9138   3.15e-16 -2.514714e+16 < 2.2e-16 ***
#> vs1:am1:wt  -7.7676   1.60e-16 -4.843029e+16 < 2.2e-16 ***
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> Log-likelihood: -69.72   Adj. R2: 0.81694 
#>                        R2-Within: 0.56653
```

There's our desired **&minus;7.7676** coefficient again. This time, however, we also get the added bonus of clustered standard errors (which are switched on by default in `fixest::feols()`'s print method).

**Caveat:** The above example implicitly presumes that you don't really care about the coefficients on the parent term(s), since these are swept away by the underlying fixed-effect procedures. That is clearly not going to be desireable in every case. But, in practice, I often find that it is a perfectly acceptable trade-off for models that I am running. (For example, when I am trying to remove general calender artefacts like monthly effects.)

## Other model classes

The last thing I want to demonstrate quickly is that our little trick carries over neatly to other model classes to. Say, that ~~old workhorse of non-linear stats~~ hot! new! machine! learning! classifier: logit models. Again, I'll let you run these to confirm for yourself:

```r
## Tired
summary(glm(am ~ vs * wt, family = binomial, data = df))
## Wired
summary(glm(am ~ vs / wt, family = binomial, data = df))
```

Okay, I confess: That last code chunk was a trick to see who was staying awake during statistics class. I mean, it will correctly sum the coefficient values. But we all know that the raw coefficient values on generalised linear models like logit cannot be interepreted as marginal effects, regardless of whether there are interactions or not. Instead, we need to convert them via an appropriate link function. In R, the [**mfx**](https://cran.r-project.org/web/packages/mfx/index.html){:target="_blank"} package will do this for us automatically. My real point, then, is to say that we can combine the link function (via **mfx**) and our syntax trick (in the case of interaction terms). This makes a suprisingly complicated problem much easier to handle.

``` r
library(mfx)

## Broke
mfx::logitmfx(am ~ vs * wt, data = df)
#> Call:
#> mfx::logitmfx(formula = am ~ vs * wt, data = df)
#> 
#> Marginal Effects:
#>            dF/dx Std. Err.        z   P>|z|    
#> vs1    -0.994682  0.045074 -22.0680 < 2e-16 ***
#> wt     -1.268124  0.639812  -1.9820 0.04748 *  
#> vs1:wt  0.494044  0.719052   0.6871 0.49203    
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> dF/dx is for discrete change for the following variables:
#> 
#> [1] "vs1"

## Woke
mfx::logitmfx(am ~ vs / wt, data = df)
#> Call:
#> mfx::logitmfx(formula = am ~ vs/wt, data = df)
#> 
#> Marginal Effects:
#>            dF/dx Std. Err.        z   P>|z|    
#> vs1    -0.994682  0.045074 -22.0680 < 2e-16 ***
#> vs0:wt -1.268124  0.639812  -1.9820 0.04748 *  
#> vs1:wt -0.774081  0.673267  -1.1497 0.25025    
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> dF/dx is for discrete change for the following variables:
#> 
#> [1] "vs1"
```

## Conclusion

We don't always want the full marginal effect of an interaction term. Indeed, there are times where we are specifically interested in evaluating the partial marginal effect. (In a difference-in-differences model, for example.) But in many other cases, the full marginal effect of the interaction terms is _exactly_ what we want. The `lm(y ~ f1 / x2)` syntax trick (and its equivalents) is a really useful shortcut to remember in these cases.

**PS.** In case, I didn't make it clear: This trick works best when your interaction contains at most one continuous variable. (This is the parent "x" term that gets left out in all of the above examples.) You can still use it when you have more than one continuous variable, but it will implicitly force one of them to zero. Factor variables, on the other hand, get forced relative to the same base (here: the intercept), which is what we want.

**Update.** Subsequent to posting this, I was made aware of this nice [SO answer](https://stackoverflow.com/questions/32616762/defining-an-infix-operator-for-use-within-a-formula/32682826#32682826){:target="_blank"} that treads similar ground. I like the definitional contrast between factors that are "crossed" versus those that are "nested".