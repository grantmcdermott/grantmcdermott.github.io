---
title: Software
permalink: /software/
header:
  image: /assets/images/tree-bw-crop2.jpg
---

Here is a collection of software tools that I have written and serve as primary maintainer for.

### R

- [**etwfe**](https://www.grantmcdermott.com/etwfe/) Convenience functions for implementing Extended Two-Way Fixed Effect regressions _a la_ Wooldridge ([2021](doi:10.2139/ssrn.3906345), [2022](doi:10.2139/ssrn.4183726)).
- [**ggfixest**](https://www.grantmcdermott.com/ggfixest/) Provides dedicated `ggplot2` methods for `fixest` objects. (Formerly called "ggiplot".)
- [**lfe2fixest**](https://www.grantmcdermott.com/lfe2fixest) A simple package that automatically converts R scripts with `lfe:felm()` calls into their `fixest::feols()` equivalents.
- [**parttree**](https://grantmcdermott.com/parttree/) A package for visualizing decision tree partitions in R. Useful as a pedagogical tool, where learners are trying to understand how a decision tree partitioned the data.
- [**ritest**](https://www.grantmcdermott.com/ritest/) Fast randomization inference on R model objects. A port of the `-ritest-` Stata routine by Simon Heß.
- [**tinyplot**](https://grantmcdermott.com/tinyplot/) Lightweight extension of the base R graphics system, with support for automatic grouping, legends, facets, and various other enhancements. (Formerly called "plot2".)

In addition to the above, I am a fairly active contributor to other R packages for which I am not the primary maintainer. You can take a look at the activity log on my [GitHub page](https://github.com/grantmcdermott) for more. Finally, I have also posted a number of [gists](https://gist.github.com/grantmcdermott) over the years, usually after being nerd-sniped by some prompt on social media. Examples include a (fast) [Bayesian bootstrap](https://gist.github.com/grantmcdermott/7d8f9ea20d2bbf54d3366f5a72482ad9), [stacked regression](https://gist.github.com/apoorvalal/db934b9b9b8ec849c5aac914a6a2ca57?permalink_comment_id=4064285#gistcomment-4064285), etc.

### Quarto / R Markdown

- [**revealjs-clean**](https://github.com/grantmcdermott/quarto-revealjs-clean) A minimal and elegant presentation theme for Quarto reveal.js, inspired by modern Beamer templates. 
- [**lecturenotes**](https://grantmcdermott.com/lecturenotes) A personalised R Markdown template that I use for writing my lecture notes and academic papers. Takes care of various inconsistencies that arise when you want to export (i.e. "knit") that same .Rmd file to both HTML and PDF. (2023 update: I recommend switching to Quarto instead of R Markdown.)

### Websites

I am a (co-)maintainer of various websites and webpages that collectively aim to improve the accessibility of data science software. These include:

- [**CRAN Econometrics Task View**](https://cran.r-project.org/web/views/Econometrics.html) 
- [**DiD**](https://asjadnaqvi.github.io/DiD/) (Difference-in-Differences)
- [**LOST**](https://lost-stats.github.io/) (Library of Statistical Techniques)
- [**Stata2R**](https://stata2r.github.io/)

### Miscellaneous

A random selection of other guides, scripts, and tools:

- [**arch-tips**](https://github.com/grantmcdermott/arch-tips) A fairly detailed changelog and collection of customization tips for Arch Linux.
- [**Causal Inference: The Mixtape (Fast Forward ed.)**](https://github.com/grantmcdermott/mixtape_learnr-ff/) A lean 'n mean reworking of the code accompanying _The Mixtape_ book, which focuses on performance and concision (both the code itself and package dependencies). 
- [**codespaces-r2u**](https://github.com/grantmcdermott/codespaces-r2u) A minimal(ish) Codespaces environment for R. Provides a fully-functioning R environment up in the cloud (running on GitHub servers) at the click of a button.
- [**containerit-demo**](https://github.com/grantmcdermott/containerit-demo) Simple demo for automating a Docker image build from your R environment, using the neat containerit package.
- [**hidpi.sh**](https://gist.github.com/grantmcdermott/fa3c90179f7b5613dcf267745bf07081) Shell script for fixing HiDPI scaling issues on Linux. Good for automating an otherwise laborious process following system or library updates.
- [**open-access-fishery**](https://grantmcdermott.shinyapps.io/open-access-fishery/) An interactive Shiny app for exploring open-access fishery dynamics. Good for an introductory resource economics class.
- [**renv-rspm**](https://github.com/grantmcdermott/renv-rspm) An example repo that demonstrates my recommonded approach to creating reproducible environments in R. Includes a video link. (Update: This approach should be enabled automatically with the latest release of )
- [**tikz-examples**](https://github.com/grantmcdermott/tikzexamples) Some examples of how to draw TikZ and PGFPlots figures in LaTeX, with a focus on environmental economics topics (e.g. a negative externality with deadweight loss).