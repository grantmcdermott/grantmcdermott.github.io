---
title: Fast geospatial operations with data.table, geos & co.
excerpt: 'Use all your favourites'
date: '2022-01-14'
slug: fast-geospatial-datatable-geos
toc: true
tags:
  - R
  - spatial
  - data.table
  - sf
  - geos
---



## Motivation

This blog post pulls together various tips and suggestions that I've left
around the place. My main goal is to show you how **data.table** can be used
in geospatial workflows, as well as some additional libraries for doing
high-performance spatial work in R.

If you're the type of person who likes to load everything at once, here are the
R libraries and theme settings that I'll be using in this post. (Don't worry if
not: I'll be loading them again in the relevant sections below to underscore why
I'm calling a specific library.)


{% highlight r %}
## Data wrangling
library(dplyr)
library(data.table) ## NB: dev version! data.table::update.dev.pkg()

## Geospatial
library(sf)
library(geos)

## Plotting
library(ggplot2)
theme_set(theme_minimal())

## Benchmarking
library(microbenchmark)
{% endhighlight %}

## data.table + sf workflows

Everyone who does spatial work in R is familiar with the wonderful **sf**
package. You know, this one:


{% highlight r %}
library(sf)
library(ggplot2)

## Grab the North Carolina shapefile that come bundled with sf
nc_shapefile = system.file("shape/nc.shp", package = "sf")
nc = st_read(nc_shapefile)
{% endhighlight %}



{% highlight text %}
## Reading layer `nc' from data source `/usr/lib/R/library/sf/shape/nc.shp' using driver `ESRI Shapefile'
## Simple feature collection with 100 features and 14 fields
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -84.32 ymin: 33.88 xmax: -75.46 ymax: 36.59
## Geodetic CRS:  NAD27
{% endhighlight %}



{% highlight r %}
## Quick plot
ggplot(nc) + geom_sf()
{% endhighlight %}

![plot of chunk nc](/figure/posts/2022-01-14-fast-geospatial-datatable-geos/nc-1.png)

The revolutionary idea of **sf** was (is) that it allowed you to treat 
spatial objects as regular data frames, so you can do things like this:


{% highlight r %}
library(dplyr)

nc |>
  group_by(region = ifelse(CNTY_ID<=1980, 'high', 'low')) |>
  summarise(geometry = st_union(geometry)) |>
  ggplot(aes(fill=region)) + 
  geom_sf()
{% endhighlight %}

![plot of chunk dplyr_union](/figure/posts/2022-01-14-fast-geospatial-datatable-geos/dplyr_union-1.png)

In the above code chunk, I'm using the powerful **dplyr** package to do a 
grouped aggregation on our North Carolina data object. The aggregation itself is
pretty silly---i.e. divide the state into two hemispheres---but the same idea
extends to virtually all of **dplyr's** capabilities. It makes for a very 
potent and flexible combination that has driven an awful lot of R-based spatial
work in recent years.

At the same time, there's another powerful data wrangling library in R: 
**data.table**. This post is not going to rehash the (mostly pointless) debates 
about which of **dplyr** or **data.table** is 
better.^[Use what you want people.] 
But I think it's fair to say that the latter offers incredible 
performance that makes it a must-use library for a lot of people, including
myself. And yet it seems to me that many **data.table** users aren't aware that 
you can use it for spatial operations in exactly the same way.

If you're following along on your own computer, make sure to grab the 
development version (v1.14.3) before continuing:


{% highlight r %}
## Assuming you have data.table on your system already
data.table::update.dev.pkg()

## Use the below instead if you don't have data.table installed already
# remotes::install_github("Rdatatable/data.table")
{% endhighlight %}

Okay, let's create a "data.table" version of our `nc` object and take a 
quick look at the first few rows and some columns.


{% highlight r %}
library(data.table)

nc_dt = as.data.table(nc)
nc[1:3, c('NAME', 'CNTY_ID', 'geometry')]
{% endhighlight %}



{% highlight text %}
## Simple feature collection with 3 features and 2 fields
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -81.74 ymin: 36.23 xmax: -80.44 ymax: 36.59
## Geodetic CRS:  NAD27
##        NAME CNTY_ID                       geometry
## 1      Ashe    1825 MULTIPOLYGON (((-81.47 36.2...
## 2 Alleghany    1827 MULTIPOLYGON (((-81.24 36.3...
## 3     Surry    1828 MULTIPOLYGON (((-80.46 36.2...
{% endhighlight %}

At this point, I have to briefly back up to say that the reason I wanted you to
grab the development version of **data.table** is that it "pretty prints" the
columns by default. This not only includes the columns types and keys (if you've
set any), but also the special `sfc_MULTIPLOYGON` list columns which is where
the **sf** magic is hiding. It's a small cosmetic change that nonetheless
underscores the integration between these two 
packages.^[There are many other [killer features](https://rdatatable.gitlab.io/data.table/news/index.html#unreleased-data-table-v1-14-3-in-development-) that **data.table** v1.14.3 is set to introduce. I might write up a list of my favourites once the new version hits CRAN. In the meantime, if any DT devs are reading this, _please pretty please_ can we include these two PRs ([1](https://github.com/Rdatatable/data.table/pull/4163), [2](https://github.com/Rdatatable/data.table/pull/4883)) into the next release before then.]

Just like we did with **dplyr** earlier, we can now do grouped spatial 
operations on this object using **data.table**'s concise syntax:


{% highlight r %}
nc_dt[, 
      .(geometry = st_union(geometry)), 
      by = .(region = ifelse(CNTY_ID<=1980, 'high', 'low'))] |> 
  ggplot(aes(geometry=geometry, fill=region)) + 
  geom_sf() +
  labs(caption = "Now brought to you by data.table")
{% endhighlight %}

![plot of chunk dt_union](/figure/posts/2022-01-14-fast-geospatial-datatable-geos/dt_union-1.png)

Okay, I'll admit that there are a few tiny tweaks we need to make to the plot
call. Unlike with the non-**data.table** workflow, this time we have to specify
the geometry aesthetic with `aes(geometry=geometry, ...)`. Otherwise,
**ggplot2** won't know what do with this object. The other difference is that it
doesn't automatically recognise the CRS (i.e. "NAD27"), so the projection is a
little off. Again, however, that information is contained with the `geometry`
column of our `nc_dt` object. It just requires that we provide this information
to our plot call explicitly.


{% highlight r %}
## Grab CRS from the geometry column
crs = st_crs(nc_dt$geometry)

## Update our previous plot
last_plot() + 
  coord_sf(crs=crs) +
  labs(caption = "Now with the right projection too")
{% endhighlight %}

![plot of chunk dt_union_crs](/figure/posts/2022-01-14-fast-geospatial-datatable-geos/dt_union_crs-1.png)

Plotting tweaks aside, I don't want to lose sight of the main point of this
post, namely: **sf** and **data.table** play perfectly well together. You
can do (grouped) spatial operations and aggregations inside the latter, exactly
how you would any other data wrangling task. So if you love **data.table's**
performance and syntax, then by all means continue using it for your spatial
work too. Speaking of performance...

## Speeding things up with geos

As great as **sf** is, even its most ardent proponents will admit that it can
drag a bit when it comes to large geospatial computations. Now that's not to say
it is objectively "slow". But I've found it does lag behind **geopandas**, for example, 
once I start working with really big geospatial files. Luckily, there's a new
kid in town that offers big performance gains and plays very well with the 
workflow I demonstrated above.

Dewey Dunnington and Edzer Pebesma's
[**geos**](https://paleolimbot.github.io/geos/index.html) package covers all of
the same geospatial operations as **sf**. However, it directly wraps the 
underlying `GEOS` API, which is all written in C and is thus extremely performant.
Here's a simple example, where we calculate the centroid of each North Carolina
county.


{% highlight r %}
library(geos)           ## For geos operations  
library(microbenchmark) ## For benchmarking

## Create a geos geometry object
nc_geos = nc |> as_geos_geometry()

## Benchmark
microbenchmark(
  sf = nc$geometry |> st_centroid(),
  geos = nc_geos |> geos_centroid(), 
  times = 5
  )
{% endhighlight %}



{% highlight text %}
## Unit: microseconds
##  expr    min     lq   mean median     uq    max neval cld
##    sf 6683.6 6737.4 7079.3 7224.0 7357.4 7394.4     5   b
##  geos  102.5  105.8  132.1  115.3  140.6  196.5     5  a
{% endhighlight %}

A couple of things worth noting. First, the executing functions are very similar 
(`st_centroid()` vs `geos_centroid()`). Second, the **geos** command executes
orders of magnitude faster than the **sf** equivalent. Third, we have to do an
explicit `as_geos_geometry()` coercion before we can perform any **geos** 
operations on the resulting object. 

That last point seems the most mundane. (_Why aren't you talking more about how
crazy fast **geos** is?!_) But it's important since it underscores a key 
difference between the two packages and why the developers view them as 
complements. Unlike **sf**, which treats spatial objects as data frames, 
**geos** only preserves the geometry attributes. Take a look:


{% highlight r %}
head(nc_geos)
{% endhighlight %}



{% highlight text %}
## <geos_geometry[6] with CRS=NAD27>
## [1] <MULTIPOLYGON [-81.741 36.234...-81.240 36.590]>
## [2] <MULTIPOLYGON [-81.348 36.365...-80.903 36.573]>
## [3] <MULTIPOLYGON [-80.966 36.234...-80.435 36.565]>
## [4] <MULTIPOLYGON [-76.330 36.073...-75.773 36.557]>
## [5] <MULTIPOLYGON [-77.901 36.163...-77.075 36.556]>
## [6] <MULTIPOLYGON [-77.218 36.230...-76.707 36.556]>
{% endhighlight %}

Gone are all those extra columns containing information about county names,
FIPS codes, population numbers, etc. etc. We're just left the necessary 
information to do high-performance spatial operations.

### Quick aside on plotting geos objects

Because we've dropped all of the **sf** / data frame attributes, we can't use 
**ggplot2** to plot anymore. But we can use the base R plotting method:


{% highlight r %}
plot(nc_geos, col = "gray90")
plot(geos_centroid(nc_geos), pch = 21, col = 'red', bg = 'red', add = TRUE)
{% endhighlight %}

![plot of chunk nc_geos_baseplot](/figure/posts/2022-01-14-fast-geospatial-datatable-geos/nc_geos_baseplot-1.png)

Actually, that's not quite true, since an alternative is convert it back into an
**sf** object with `st_as_sf()` and then call **ggplot2**. This is particularly
useful because you can hand off some heavy calculation to **geos** before
bringing it back to **sf** for any additional functionality. Again, the 
developers of these packages designed them to act as complements.


{% highlight r %}
ggplot() +
  geom_sf(data = nc) +
  geom_sf(data = nc_geos |> geos_centroid() |> st_as_sf(), 
          col = "red")
{% endhighlight %}

![plot of chunk nc_geos_ggplot](/figure/posts/2022-01-14-fast-geospatial-datatable-geos/nc_geos_ggplot-1.png)

Okay, back to the main post...

### Back to the data.table + geos workflow

Finally, we get to the _pièce de résistance_ of today's post. The fact that 
`as_geos_geometry()` creates a GEOS geometry (i.e. list) object---rather than
preserving all of the data frame attributes---is a actually a good thing for our
**data.table** workflow. Why? Well, because we can just include this GEOS
geometry object as a regular column inside our 
data.table.^[Yes, yes. I know you can include a (list) column of data frames within a data.table. But just bear with me for the moment.] 
This means that you can do grouped spatial operations
inside that data.table and thus **combine the power of data.table and geos**.

(Same for tibbles, but we'll get to that.)

Let's prove that this idea works by creating a GEOS column in our data.table. 
I'll creatively call this column `geo`, but really you could call it anything
you want (including overwriting the existing `geometry` column).


{% highlight r %}
nc_dt[, geo := as_geos_geometry(geometry)]
nc_dt[1:3, c('NAME', 'CNTY_ID', 'geo')] ## Print a few rows/columns
{% endhighlight %}



{% highlight text %}
##         NAME CNTY_ID                                              geo
##       <char>   <num>                                  <geos_geometry>
## 1:      Ashe    1825 <MULTIPOLYGON [-81.741 36.234...-81.240 36.590]>
## 2: Alleghany    1827 <MULTIPOLYGON [-81.348 36.365...-80.903 36.573]>
## 3:     Surry    1828 <MULTIPOLYGON [-80.966 36.234...-80.435 36.565]>
{% endhighlight %}

GEOS column in hand, we can manipulate or plot it directly from within the 
data.table. For example, we can recreate our previous centroid plot.


{% highlight r %}
plot(nc_dt[, geo], col = "gray90")
plot(nc_dt[, geos_centroid(geo)], pch = 21, col = 'red', bg = 'red', add = TRUE)
{% endhighlight %}

![plot of chunk dt_geos_centroid](/figure/posts/2022-01-14-fast-geospatial-datatable-geos/dt_geos_centroid-1.png)

And here's how we could replicate our earlier "hemisphere" plot:


{% highlight r %}
nc_dt[,
      .(geo = geo |> geos_make_collection() |> geos_unary_union()),
      .(region = ifelse(CNTY_ID<=1980, 'high', 'low'))
      ][, geo] |>
  plot()
{% endhighlight %}

![plot of chunk geos_union](/figure/posts/2022-01-14-fast-geospatial-datatable-geos/geos_union-1.png)

Okay, this time around the translation from the equivalent **sf** code isn't as 
direct. We have one step (`st_union()`) vs. two 
(`geos_make_collection() |> geos_unary_union()`). The first `geo_unary_union()`
step is clear enough. But it's the second `geos_make_collection()`step that's
key for our aggregating task. We have to tell **geos** to treat everything
within the same group (the `by = ` part) as a collective. This extra step 
becomes very natural after you've done it a few times and is a small price to
pay for the resulting performance boost. 

Speaking of which, it's nearly time for some final benchmarks. The only extra
thing I want to do first is, as promised, include a **tibble**/**dplyr**
equivalent. The exact same concepts and benefits carry over here, for those of
you that prefer the tidyverse syntax and 
workflow.^[The important thing is that you _explicitly_ convert it to a tibble. Leaving it as an **sf** object won't yield the same speed benefits.]


{% highlight r %}
nc_tibble = tibble::as_tibble(nc) |> 
  mutate(geo = as_geos_geometry(geometry))
{% endhighlight %}

### Benchmarks

For this final set of benchmarks, I'm going to horserace the same grouped
aggregation that we've been using throughout.


{% highlight r %}
microbenchmark(
  
  sf_tidy = nc |>
    group_by(region = ifelse(CNTY_ID<=1980, 'high', 'low')) |>
    summarise(geometry = st_union(geometry)),
  
  sf_dt = nc_dt[, 
                .(geometry = st_union(geometry)), 
                by = .(region = ifelse(CNTY_ID<=1980, 'high', 'low'))],
  
  geos_tidy = nc_tibble |>  
    group_by(region = ifelse(CNTY_ID<=1980, 'high', 'low')) |>
    summarise(geo = geos_unary_union(geos_make_collection(geo))),
  
  geos_dt = nc_dt[,
                  .(geo = geos_unary_union(geos_make_collection(geo))),
                  .(region = ifelse(CNTY_ID<=1980, 'high', 'low'))],
  
  times = 5
)
{% endhighlight %}



{% highlight text %}
## Unit: milliseconds
##       expr    min     lq   mean median     uq    max neval  cld
##    sf_tidy 102.93 103.20 104.42 103.62 105.15 107.18     5    d
##      sf_dt  95.79  96.81  97.33  96.90  96.90 100.24     5   c 
##  geos_tidy  15.25  15.62  15.86  15.76  16.18  16.47     5  b  
##    geos_dt  11.82  12.08  12.17  12.26  12.33  12.34     5 a
{% endhighlight %}

**Result:** A 10x speed-up. Nice! While the dataset that we're using here is too
small to notice in real-time, those same performance benefits will be evident 
with big geospatial tasks too.

## Conclusion

My takeaways:

1. It's fine to treat **sf** objects as **data.tables** (or vice versa) if 
that's your preferred workflow. Just remember to specify the geometry column.

2. For large (or small!) geospatial tasks, give the **geos** package a go. 
It integrates very well with both **data.table** and the **tidyverse**, and the 
high-performance benefits carry over to both ecosystems.


PS. There are more exciting high-performance geospatial developments on the 
way in R (as well as other languages) 
like [geoarrow](https://github.com/paleolimbot/geoarrow).
