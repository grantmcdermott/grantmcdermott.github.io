---
title: 'Simulations remix: Turn up the base'
excerpt: 'Faster still, with base R'
date: '2021-07-08'
slug: simulations-remix-turn-up-the-base
toc: false
mathjax: true
tags:
  - R
  - Econometrics
---



*Note: I apologise for the title of this post and will show myself the door shortly.*

I wanted to quickly follow up on my last post about [efficient simulations in R]({{ site.url }}/efficient-simulations-in-r). If you recall, in that post we used **data.table** and some other tricks to run 40,000 regressions (i.e. 20k simulations with 2 regressions each) in just over 2 seconds. The question before us today is: **Can we go even faster using only base R?** And it turns out that the answer is, yes, we can.

My motivation for a follow-up is partially the result of [this very nice post](https://elbersb.com/public/posts/interaction_simulation/) by Benjamin Elbers, who replicates my simulation using Julia. In so doing, he demonstrates some of Julia's killer features; most notably the fact that we don't need to think about vectorisation --- e.g. when creating the data --- since Julia's compiler will take care of that for us automatically.[^1] But Ben also does another interesting thing, which is to show the speed gains that come from defining our own (super lean) regression function. He uses Cholesky decomposition and it's fairly straightforward to do the same thing in R. ([Here](https://www.theissaclee.com/post/linearqrandchol/) is a nice tutorial by Issac Lee.) 

I was halfway on my way to doing this myself when I stumbled on a [totally different post](https://rpubs.com/maechler/fast_lm) by R Core member, Martin Maechler. Therein he introduces the **`.lm.fit()`** function (note the leading dot), which incurs even less overhead than the `lm.fit()` function I mentioned in my last post. I'm slightly embarrassed to say I had never heard about it until now[^2], but a quick bit of testing "confirms" Martin's more rigorous benchmarks: `.lm.fit` yields a consistent 30-40% improvement over even `lm.fit`.

Now, it would be trivial to amend my previous simulation script to slot in `.lm.fit()` and re-run the benchmarks. But I thought I'd make this a bit more interesting by redoing the whole thing using only base R. (I'll load the **parallel** package, but that comes bundled with the [base distribution](https://stackoverflow.com/a/9705725/4115816) so hardly counts as cheating.) Here's the full script with benchmarks for both sequential and parallel implementations at the bottom.


{% highlight r %}
# Data generation ---------------------------------------------------------

gen_data = function(sims=1) {
  
  ## Total time periods in the the panel = 500
  tt = 500
  
  ## x1 covariates
  x1_A = 1 + rnorm(tt*sims, 0, 1)
  x1_B = 1/4 + rnorm(tt*sims, 0, 1)
  x1 = c(x1_A, x1_B)
  
  ## Add second, nested x2 covariates for each country
  x2_A = 1 + x1_A + rnorm(tt*sims, 0, 1)
  x2_B = 1 + x1_B + rnorm(tt*sims, 0, 1)
  x2 = c(x2_A, x2_B)
  
  ## Outcomes (notice different slope coefs for x2_A and x2_B)
  y_A = x1_A + 1*x2_A + rnorm(tt*sims, 0, 1)
  y_B = x1_B + 2*x2_B + rnorm(tt*sims, 0, 1)
  y = c(y_A, y_B)
  
  ## Group variables (id and sim)
  id = as.factor(c(rep('A', length(x1_A)), rep('B', length(x1_B))))
  sim = rep(rep(1L:sims, each = tt), times = length(levels(id)))
  
  ## Demeaned covariates
  x1_dmean = x1 - ave(x1, list(sim, id), FUN = mean)
  x2_dmean = x2 - ave(x2, list(sim, id), FUN = mean)
  
  ## Bind in a matrix
  mat = cbind('sim' = sim, 
              'id' = id,
              'y' = y,
              'intercept' = 1, 
              'x1' = x1, 
              'x2' = x2, 
              'x1:x2' = x1*x2, 
              'x1_dmean:x2_dmean' = x1_dmean * x2_dmean)
  
  ## Set order i.t.o simulations
  mat = mat[order(mat[, 'sim']), ]
  
  return(mat)
}

## How many simulations do we want?
n_sims = 2e4

## Generate them all as one big matrix
d = gen_data(n_sims)

## Create index list (for efficient subsetting of the large data matrix)
## Note that each simulation is (2*500=)1000 rows long.
ii = lapply(1L:n_sims-1, function(i) 1L:1e3L + rep(1e3L*(i), each=1e3L))

# Benchmarks --------------------------------------------------------------

library(microbenchmark) ## For high-precision timing
library(parallel)
n_cores = detectCores()


## Convenience function for running the two regressions and extracting the 
## interaction coefficients of interest (saves having to retype everything).
## The key bit is the .lm.fit() function.
get_coefs = 
  function(dat) {
    level = coef(.lm.fit(dat[, c('intercept', 'x1', 'x2', 'x1:x2', 'id')], 
                         dat[, 'y']))[4]
    dmean = coef(.lm.fit(dat[, c('intercept', 'x1', 'x2', 'x1_dmean:x2_dmean', 'id')], 
                         dat[, 'y']))[4]
    return(cbind(level, dmean))
  }

## Run the benchmarks for both sequential and parallel versions
microbenchmark(
  
  sims_sequential = lapply(1:n_sims, 
                           function(i) {
                             index = ii[[i]]
                             get_coefs(d[index, ])
                             }),
  
  sims_parallel = mclapply(1:n_sims, 
                           function(i) {
                             index = ii[[i]]
                             get_coefs(d[index, ])
                             }, 
                           mc.cores = n_cores
                           ),

  times = 1
  )
{% endhighlight %}



{% highlight text %}
## Unit: milliseconds
##             expr    min     lq   mean median     uq    max neval
##  sims_sequential 2471.1 2471.1 2471.1 2471.1 2471.1 2471.1     1
##    sims_parallel  836.1  836.1  836.1  836.1  836.1  836.1     1
{% endhighlight %}

There you have it. Down to _less than a second_ for a simulation involving 40,000 regressions using only base R.[^3] On a laptop, no less. Just incredibly impressive. 

**Conclusion:** No grand conclusion today... except a sincere note of gratitude to the R Core team (and Julia devs and so many other OSS maintainers) for providing us with such an incredible base to build from. 

P.S. Achim Zeileis (who else?) has another great tip for speeding up simulations where the experimental design is fixed [here](https://twitter.com/AchimZeileis/status/1413407892556947465).

[^1]: In this house, we stan both R and Julia.

[^2]: I think Dirk Eddelbuettel had mentioned it to me, but I hadn't grokked the difference.

[^3]: Interestingly enough, this knitted R markdown version is a bit slower than when I run the script directly in my R console. But we're really splitting hairs now. (As an aside: I won't bother plotting the results, but you're welcome to run the simulation yourself and confirm that it yields the same insights as my previous post.)
