
  - [ggplot2-extension-cookbook](#ggplot2-extension-cookbook)
  - [Preface and acknowledgements](#preface-and-acknowledgements)
  - [Getting started](#getting-started)
  - [geom\_\*: writing new definitions for where and how of marks on
    ggplots](#geom_-writing-new-definitions-for-where-and-how-of-marks-on-ggplots)
      - [geom\_text\_coordinate: **1:1:1**](#geom_text_coordinate-111)
          - [Step 0: use base ggplot2](#step-0-use-base-ggplot2)
          - [Step 1: compute](#step-1-compute)
          - [Step 2: pass to ggproto
            object](#step-2-pass-to-ggproto-object)
          - [Step 3. Write user facing
            function.](#step-3-write-user-facing-function)
      - [geom\_xy\_means: **n:1:1**](#geom_xy_means-n11)
          - [Step 0. Use base ggplot2](#step-0-use-base-ggplot2-1)
          - [Step 1. Write compute
            function](#step-1-write-compute-function)
          - [Step 2. Define Stat, pasing in
            compute](#step-2-define-stat-pasing-in-compute)
          - [Step 3. Write user-facing
            function](#step-3-write-user-facing-function-1)
          - [Step 4.](#step-4)
      - [geom\_chull: **N:1:n**](#geom_chull-n1n)
          - [Step 1 and 2](#step-1-and-2)
          - [Step 3. Write user-facing geom\_/stat\_
            Function(s)](#step-3-write-user-facing-geom_stat_-functions)
          - [Step 4. Try out/test/ enjoy](#step-4-try-outtest-enjoy)
      - [geom\_circle: **1:1:n**, new: defining compute\_panel in
        ggproto](#geom_circle-11n-new-defining-compute_panel-in-ggproto)
          - [Step 0. Do it with base
            ggplot2](#step-0-do-it-with-base-ggplot2)
          - [Step 1. Compute](#step-1-compute-1)
          - [Step 2. Pass to ggproto](#step-2-pass-to-ggproto)
          - [Step 3. Write geom\_\* or
            stat\_\*](#step-3-write-geom_-or-stat_)
          - [Step 4: Enjoy (test)](#step-4-enjoy-test)
      - [geom\_us\_state: **1:1:n**](#geom_us_state-11n)
      - [geom\_ggcirclepack: **1:1:n,
        interdependance**](#geom_ggcirclepack-11n-interdependance)
      - [geom\_ols: **n:k:w;
        interdependence**](#geom_ols-nkw-interdependence)
      - [geom\_county: **1:1:1 via geometry
        sf**](#geom_county-111-via-geometry-sf)
      - [geom\_candlestick summarize first, then interdependence
        …](#geom_candlestick-summarize-first-then-interdependence-)
      - [geom\_pie: **n-\>1:1:1**](#geom_pie-n-111)
      - [geom\_wedge: **n-\>1:1:n**](#geom_wedge-n-11n)
  - [stat\_\* layers: keeping flexible via stat\_\*
    functions](#stat_-layers-keeping-flexible-via-stat_-functions)
      - [stat\_chull](#stat_chull)
      - [stat\_waterfall: **1:1:1;
        interdependence**](#stat_waterfall-111-interdependence)
          - [Step 0](#step-0)
          - [Steps 1 and 2](#steps-1-and-2)
          - [Step 3](#step-3)
          - [Step 4](#step-4-1)
  - [borrowing compute](#borrowing-compute)
      - [geom\_smoothfit: **1:1:1** ggproto piggybacking on
        compute…](#geom_smoothfit-111-ggproto-piggybacking-on-compute)
  - [add default aesthetics](#add-default-aesthetics)
      - [geom\_barlab: Adding defaults to existing stats via ggproto
        editing](#geom_barlab-adding-defaults-to-existing-stats-via-ggproto-editing)
  - [modified start points;
    ggverbatim(),](#modified-start-points-ggverbatim)
      - [ggverbatim()](#ggverbatim)
  - [ggedgelist()](#ggedgelist)
  - [theme\_chalkboard()](#theme_chalkboard)
  - [geom-led extension](#geom-led-extension)
      - [ggscatterplot](#ggscatterplot)
  - [wrapping fiddly functions (annotate and
    theme)](#wrapping-fiddly-functions-annotate-and-theme)
  - [make it a package: ggtedious *formal
    testing*](#make-it-a-package-ggtedious-formal-testing)

<!-- README.md is generated from README.Rmd. Please edit that file -->

# ggplot2-extension-cookbook

<!-- badges: start -->

<!-- badges: end -->

*The ggplot2 Extension Cookbook* aims to provide ggplot2 some extension
strategies in a consistent and accessible way.

We group the content by extension type, provide demonstrations of their
use, and then link (for now) to the code that defines them. In that
material, I’ll try to stick to a formula to orient you to the ggplot2
extension:

  - Step 0: get job done with ‘base’ ggplot2
  - Step 1: Write a function for the ‘compute’
  - Step 2: Pass the compute to ggproto object
  - Step 3: Pass ggproto to a user-facing function for use in a ggplot()
    pipeline
  - Step 4: Try out/test/enjoy\!

Regarding stat\_‘s vs. geom\_’s, I take a geoms-first approach, because
they are more commonly used. I suspect this is they are more concrete
descriptive of what the creator envisions for her plot, whereas stat\_\*
may feel a be more ’adverbial’ and nebulous in their description of
rendered output. Consider that ggplot(mtcars, aes(wt, mpg)) +
stat\_identity() and ggplot(mtcars, aes(wt, mpg)) + geom\_point() render
identical ggplot objects, but the later might feel more descriptive of
the resultant plot. Furthermore, when it comes to excitement about
ggplot2 extension packages, new geoms really rule the day, see
(<https://albert-rapp.de/posts/ggplot2-tips/20_ggplot_extensions/ggplot_extensions>;
And slide 38 in C. Scherer’s ‘Favorite ggplot2 extensions’
<https://www.cedricscherer.com/slides/RLadiesTunis-2021-favorite-ggplot-extensions.pdf>)

Finally, most of the code is at the ‘R for data science level, and not
’Advanced R’ level, which hopefully will afford greater reach. Object
oriented programming (OOP) gets top billing in many extension materials,
but many folks that *are* fluent in ggplot2 might *not* know much about
OOP. So, I try to see what can be accomplished with little emphasis on
OOP and ggroto.

*Currently, I’m experimenting with a ratio typology that you’ll see in
the section titles. The idea is to think about how the input data
relates to the mark we see on the plot and in turn how the mark’s
information is stored in the the ggplot2 object. This is really new, and
I’m unsure of how productive or precise it can be…*

Overall, I think the resources in thisggplot2 extension cookbook are
aligned with the findings in [‘10 Things Software Developers Should
Learn about
Learning’](https://cacm.acm.org/magazines/2024/1/278891-10-things-software-developers-should-learn-about-learning/fulltext)

# Preface and acknowledgements

I attended Thomas Lin Pederson’s January 2020 talk ‘Extending your
ability to extend ggplot2’ seated on the floor of a packed out ballroom.
The talk had the important central message - “you can be a ggplot2
extender”. And since then, I aimed to be an extender. I hope that this
*ggplot2 Extension Cookbook* will help you on your extender journey and,
especially if you are fluent in R and ggplot2, it says to you “you can
be a ggplot2 extender”.

I’ve been using ggplot2 since about 2017, and really enjoyed the user
interface. In general, the syntax does a fantastic job at letting users
compose their plots bit-by-bit, closely resembling to how you might
sketch out a plot on a notepad or blackboard, and describing your
decisions as you go to yourself or a colleague. Or as Thomas Lin
Pederson has also said, ‘ggplot2 lets you *speak your plot into
existence*’. And perhaps a little less eloquently by Hadley Wickham’s,
the ggplot2 author, “This is what I’m thinking; your the computer, now
go and do it\!” (my paraphrase of the author talking about how he
thought data viz should feel as a statistical consultant – well before
ggplot2 existed).

But there were pain points when using ‘base’ ggplot2; for me, this was
mostly when a geom didn’t exist for doing some compute in the
background, but maybe wanted that compute done over and over, and it
would be a slough to do it over and over upstream to the plotting
environment. This pre-computation problem felt manageable in classroom
setting which I was in through early 2020 but when I moved to a
primarily analytic role at West Point — where the volume of analysis was
simply higher and turn around times faster — I felt the problem much
more acutely. (Overnight, I went from weak preference for geom\_col - to
strong preference for geom\_bar\!) Extension seemed to offer the
solution to the problem and I was more motivated in my analyst role.

However, I experienced about a year of failure and frustration when
first entering the extension space. If I weren’t so convinced of the
efficiency gains that it would eventually yield, I’d likely have given
up. Recognizing the substancial hurdles for even long time R and ggplot2
users, I think there is space for more ggplot2 extension reference
materials, such as the recipes in the *ggplot2 Extension Cookbook*.

I’m grateful for several expediences and the efforts of others that have
refined these new materials. First, just after getting my own feet wet
in extension, I had the chance to work on extension with students in the
context of independent studies. Our focus was the same type of extension
that Pederson demonstrated – a geom that used a Stat to do some
computational work, and then inherited the rest of its behavior from a
more primitive geom.

Working with first and second year undergrad students meant that I had
to think about and formulate workflow - where reference material would
even be accessibility to very new R users. As veterans of just one or
two stats classes that used R and ggplot2, what would they find familiar
and accessible? What might we be able to de-emphasize ggproto and OOP in
R which they hadn’t covered and still succeed? Working with students, I
had a wonderful chance to refine extension workflow futher.

The following steps emerged and persisted into our tutorial formula:

  - Step 0: get job done with ‘base’ ggplot2
  - Step 1: Write a function for the ‘compute’
  - Step 2: Pass the compute to ggproto
  - Step 3: Pass ggproto to stat/geom/facet function
  - Step 4: Try out/test/enjoy\!

Taking on new R users to try to do something only just felt like I was
wrapping my head around was a leap of faith. I was very impressed with
what the students were able to accomplish during a single semester, and
breathed a sign of relief.

I wondered how the strategy would perform with experienced R and ggplot2
users. Being an academic, I wanted to assess further and I went down the
route of devising a tutorial \[with assistance from independent study
student Morgan Brown\] and formally getting feedback on it via focus
groups and a survey, after refining the tutorial.

I did some research on ggplot2 extension among ggplot2 and R ‘super
users’ and have found that the perhaps this community is under-served,
but with the right materials, more folks could get into ggplot2
extension.

I fielded ‘easy geom recipes’ with a group of statistics educators,
conducting a survey on the resource and also getting feedback via a
focus group.

Among my favorite quotation from the focus groups is something that
validated the efforts but also challenged me:

> it was … easy. And I felt empowered as a result of that…. But you
> know, like, my problem isn’t gonna be that easy.

To that participant, I’d say ‘Sometimes it *is* that easy’. But he is
right, that often times I come to an extension problem and am suprised
that the strategy that I think is going to work doesn’t, or at least not
without a little fiddling.

The [feedback on the
easy-geom-recipes](https://github.com/EvaMaeRey/easy-geom-recipes) was
collected in March 2023.

To try to make those experiences valuable to others, I follow the
‘recipe’ formula as much as possible so that as strategies morph, one
still recognizes ‘where we are’ at a high-level in the process.

Now to serve the community and for use in this *ggplot2 Extension
Cookbook*, I’m using an in-README documentation strategy for my ggplot2
extension packages so that the development stories are in one place
instead of scattered between package files.

So, I’m interested in simply developing new, useful extensions.

This is made possible via the readme2pkg R package and in turn the knitr
package written by Yihui Xie, whose tools (literate programming,
xaringan and other tools) had already played a huge role in my
relationship with ggplot2. Before arriving at this solution, I felt that
I’d need to divide my energy between moving forward in my own extension
development journey and educator. I’m grateful that with the magic of
knitr::knitr\_code$get() I allow code to sit in the README narrative and
also send it to the required .R directory to serve in the package.

At present, I’ll just show examples of functionality, and then link to
the READMEs for further investigation of the specific recipes/strategies
used.

I’m personally grateful to other ggplot2 extenders and R enthusiasts
that have supported this journey.

I’m also grateful to the ggplot2 development team .

I’m also indebted to my Department of Mathematics and Dean Data Cell
colleagues at West Point, for sitting through some talks (some
extemporaneous and muddled) where I tried to articulate my ggplot2
extension dreams.

Finally, to Winston Chang, who gets top billing in the ggplot2 extension
vignette along with your ggproto, I hope you won’t mind the general
approach here which experiments with making ggproto as ignorable as
possible for OOP noobs. I also hope to meet you someday and hear more
about the early days of ggproto, maybe at ggplot2 extenders meetup as a
special guest, perhaps January 2025.

And finally, finally to Hadley Wickham and Leland Wilkinson having
incredible insights and acting on them.

# Getting started

For best results, I’d recommend *diving* in by actually creating some
geoms as prompted in the ‘easy geom recipes’ tutorial using the rendered
[tutorial](https://evamaerey.github.io/easy-geom-recipes/easy_geom_recipes_compute_group.html)
or [text .Rmd
file](https://raw.githubusercontent.com/EvaMaeRey/easy-geom-recipes/main/easy_geom_recipes_compute_group.Rmd).
The ‘easy recipes’ contain 3 fully worked examples, and 3 exercises that
extend the lessons in the examples.

Having completed these exercises, you’ll have lived geom creations from
start to finish, will be well oriented to the consistent patterns I use,
to the extent possible, throughout the cookbook.

# geom\_\*: writing new definitions for where and how of marks on ggplots

This section tackles creating new geom\_\* layers. The strategy is to
look at compute that you’d do without extension capabilities (Step 0),
and then create a Stat for that (Step 1 & 2), and then compose a
user-facing function, which inherits other behavior from a more
primitive geom (Step 3), so that ggplot2 can do compute for you in the
background (Step 4).

## geom\_text\_coordinate: **1:1:1**

  - for each row in the input dataframe …
  - we’ll perceive a single mark
  - which will be defined by as single row in the internal dataframe

### Step 0: use base ggplot2

``` r
library(tidyverse)
library(ggxmean)

cars |>
  mutate(coords = 
           paste0("(", speed, ",", dist, ")")) |>
  ggplot() + 
  aes(x = speed, y = dist) + 
  geom_point() + 
  geom_text(aes(label = coords), 
            check_overlap = T,
            hjust = 0,
            vjust = 0)
```

![](man/figures/unnamed-chunk-3-1.png)<!-- -->

Step 0.b But we might like the concise syntax…

``` r
ggplot(cars) + 
  aes(x = speed, y = dist) + 
  geom_point() + 
  geom_text_coordinate(
            check_overlap = T,
            hjust = 0,
            vjust = 0)
```

### Step 1: compute

``` r
compute_group_coordinates <- function(data, scales) {

data |>
    mutate(label = 
             paste0("(", data$x, ", ", data$y, ")"))
}

cars %>% 
  rename(x = speed, y = dist) %>% 
  compute_group_coordinates() %>% 
  head()
#>   x  y   label
#> 1 4  2  (4, 2)
#> 2 4 10 (4, 10)
#> 3 7  4  (7, 4)
#> 4 7 22 (7, 22)
#> 5 8 16 (8, 16)
#> 6 9 10 (9, 10)
```

### Step 2: pass to ggproto object

``` r
StatCoordinate <- ggplot2::ggproto(
  `_class` = "StatCoordinate",
  `_inherit` = ggplot2::Stat,
  required_aes = c("x", "y"),
  compute_group = compute_group_coordinates
  )
```

### Step 3. Write user facing function.

A user-facing geom\_\* function will use the gggplot2::layer function
under the hood. geom\_layer is actually an exported function in ggplot2
and can be used directly in ggplot() pipelines as shown below; however
for convenience, the layer() function usually wrapped to have a fixed
stat or fixed geom. In geom\_text\_coordinate, because the use-scope is
so narrow, both the stat and geom are ‘hard-coded’ in the layer.

``` r
# part 3.0 use ggplot2::layer which requires specifying Geom and Stat
ggplot(data = cars) + 
  aes(x = speed, y = dist) + 
  geom_point() + 
  ggplot2::layer(
    stat = StatCoordinate,
    geom = ggplot2::GeomText,
    position = "identity"
    )
```

![](man/figures/unnamed-chunk-7-1.png)<!-- -->

``` r

# part b. create geom_* user-facing function using g
geom_text_coordinate <- function(mapping = NULL, 
                                 data = NULL,
                                 position = "identity", 
                                 show.legend = NA,
                                 inherit.aes = TRUE, 
                                 na.rm = FALSE,
                                 ...) {
  ggplot2::layer(
    stat = StatCoordinate,
    geom = ggplot2::GeomText, 
    mapping = mapping,
    data = data,
    position = position,
    show.legend = show.legend,
    inherit.aes = inherit.aes,
    params = list(na.rm = na.rm, ...)
  )
}
```

## geom\_xy\_means: **n:1:1**

*many rows from a dataset: will be summarized and visualized by as
single mark: the mark will be defined by one row of data*

### Step 0. Use base ggplot2

``` r
mtcar_xy_means <- mtcars |>
  summarize(wt_mean = mean(wt),
            mpg_mean = mean(mpg))

ggplot(mtcars) + 
  aes(x = wt, y = mpg) + 
  geom_point() + 
  geom_point(data = mtcar_xy_means,
             aes(x = wt_mean, y = mpg_mean),
             size = 8)
```

![](man/figures/unnamed-chunk-8-1.png)<!-- -->

### Step 1. Write compute function

``` r
compute_group_means <- function(data, scales){
  
  data %>% 
    summarise(x = mean(x),
              y = mean(y))
  
}
```

### Step 2. Define Stat, pasing in compute

``` r
StatXymean <- ggplot2::ggproto("StatXymean",
                               ggplot2::Stat,
                               compute_group = compute_group_means,
                               required_aes = c("x", "y")
)
```

### Step 3. Write user-facing function

``` r
geom_xy_means <- function(mapping = NULL, 
                          data = NULL,
                          position = "identity", 
                          na.rm = FALSE, 
                          show.legend = NA,
                          inherit.aes = TRUE, ...) {

  ggplot2::layer(
    stat = StatXymean, 
    geom = ggplot2::GeomPoint, 
    data = data, 
    mapping = mapping,
    position = position, 
    show.legend = show.legend, 
    inherit.aes = inherit.aes,
    params = list(na.rm = na.rm, ...)
  )

}
```

### Step 4.

``` r
ggplot(mtcars) + 
  aes(x = wt, y = mpg) + 
  geom_point() + 
  geom_xy_means(size = 8)
```

![](man/figures/unnamed-chunk-12-1.png)<!-- -->

``` r

last_plot() +
  aes()
```

![](man/figures/unnamed-chunk-12-2.png)<!-- -->

## geom\_chull: **N:1:n**

This example uses the chull function in R, which ‘computes the subset of
points which lie on the convex hull of the set of points specified.’ In
layman’s terms if you had a bunch of nails hammered into a board and put
a rubber-band around them, the convex hull would be defined by the
subset of nails touching the rubberband.

I’m especially excited to include this example, reworked using the Step
0-4 approach, because ultimately looking at the ggplot2 extension
vignette on stat\_chull and geom\_chull was the beginning of layer
extension unlocking for me.
<https://ggplot2.tidyverse.org/articles/extending-ggplot2.html#creating-a-new-stat>

``` r
library(tidyverse)
chull_row_ids <- chull(mtcars$wt, mtcars$mpg)
chull_row_ids
#>  [1] 17 16 15 24  7 29 21  3 28 20 18
mtcars_chull_subset <- mtcars %>% slice(chull_row_ids)

ggplot(mtcars) + 
  aes(x = wt, y = mpg) + 
  geom_point() + 
  geom_polygon(data = mtcars_chull_subset, 
               alpha = .3, 
               color = "black")
```

![](man/figures/unnamed-chunk-13-1.png)<!-- -->

### Step 1 and 2

``` r
# Step 1
compute_group_c_hull <- function(data, scales){
  
  chull_row_ids <- chull(data$x, data$y)
  
  data %>% slice(chull_row_ids)
  
}
```

Below, we see that the dataset is reduced to 11 rows which constitute
the convex hull perimiter.

``` r
mtcars %>% # 32 rows
  rename(x = wt, y = mpg) %>% 
  compute_group_c_hull() # 11 rows
#>                        y cyl  disp  hp drat     x  qsec vs am gear carb
#> Chrysler Imperial   14.7   8 440.0 230 3.23 5.345 17.42  0  0    3    4
#> Lincoln Continental 10.4   8 460.0 215 3.00 5.424 17.82  0  0    3    4
#> Cadillac Fleetwood  10.4   8 472.0 205 2.93 5.250 17.98  0  0    3    4
#> Camaro Z28          13.3   8 350.0 245 3.73 3.840 15.41  0  0    3    4
#> Duster 360          14.3   8 360.0 245 3.21 3.570 15.84  0  0    3    4
#> Ford Pantera L      15.8   8 351.0 264 4.22 3.170 14.50  0  1    5    4
#> Toyota Corona       21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
#> Datsun 710          22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
#> Lotus Europa        30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
#> Toyota Corolla      33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
#> Fiat 128            32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
```

``` r
# Step 2
StatChull <- ggproto(`_class` = "StatChull",
                     `_inherit` = ggplot2::Stat,
                     compute_group = compute_group_c_hull,
                     required_aes = c("x", "y"))
```

### Step 3. Write user-facing geom\_/stat\_ Function(s)

``` r
geom_chull <- function(mapping = NULL, 
                        data = NULL,
                        position = "identity", 
                        na.rm = FALSE, 
                        show.legend = NA,
                        inherit.aes = TRUE, ...) {

  ggplot2::layer(
    stat = StatChull, 
    geom = ggplot2::GeomPolygon, 
    data = data, mapping = mapping,
    position = position, 
    show.legend = show.legend, 
    inherit.aes = inherit.aes,
    params = list(na.rm = na.rm, ...)
  )

}
```

### Step 4. Try out/test/ enjoy

``` r
ggplot(data = mtcars) + 
  aes(x = wt, y = mpg) + 
  geom_point() + 
  geom_chull(alpha = .3)
```

![](man/figures/unnamed-chunk-18-1.png)<!-- -->

``` r

last_plot() + 
  aes(color = factor(am),
      fill = factor(am))
```

![](man/figures/unnamed-chunk-18-2.png)<!-- -->

-----

## geom\_circle: **1:1:n**, new: defining compute\_panel in ggproto

*a single row in a dataframe: will be visualized by a single mark : the
mark will be defined by many-row in an internal dataframe*

for each row in the dataframe, a single geometry is visualized, but each
geom is defined by many rows…

### Step 0. Do it with base ggplot2

``` r
library(tidyverse)

data.frame(x0 = 0:1, y0 = 0:1, r = 1:2/3) %>% 
  mutate(join_var = 1) %>% 
  mutate(group = row_number()) %>% 
  left_join(tibble(z = 1:5, join_var = 1),
            multiple = "all") %>% 
  mutate(around = 2*pi*z/max(z-1)) %>% 
  mutate(x = x0 + cos(around)*r,
         y = y0 + sin(around)*r) %>% 
  ggplot() + 
  aes(x, y, label = z) +
  geom_text() +
  geom_path(aes(group = group))
#> Joining with `by = join_by(join_var)`
```

![](man/figures/unnamed-chunk-43-1.png)<!-- -->

### Step 1. Compute

``` r
compute_panel_equilateral <- function(data, scales, n = 15){
  
  data %>% 
    mutate(join_var = 1, 
           group = row_number()) %>% 
  left_join(tibble(z = 1:(n + 1), join_var = 1),
            multiple = "all") %>% 
  mutate(around = 2*pi*z/max(z-1)) %>% 
  mutate(x = x0 + cos(around)*r,
         y = y0 + sin(around)*r) 
  
}

tibble(x0 = 1:2, y0 = 1:2, r = 1 ) %>% 
  compute_panel_equilateral()
#> Joining with `by = join_by(join_var)`
#> # A tibble: 32 × 9
#>       x0    y0     r join_var group     z around      x     y
#>    <int> <int> <dbl>    <dbl> <int> <int>  <dbl>  <dbl> <dbl>
#>  1     1     1     1        1     1     1  0.419 1.91   1.41 
#>  2     1     1     1        1     1     2  0.838 1.67   1.74 
#>  3     1     1     1        1     1     3  1.26  1.31   1.95 
#>  4     1     1     1        1     1     4  1.68  0.895  1.99 
#>  5     1     1     1        1     1     5  2.09  0.5    1.87 
#>  6     1     1     1        1     1     6  2.51  0.191  1.59 
#>  7     1     1     1        1     1     7  2.93  0.0219 1.21 
#>  8     1     1     1        1     1     8  3.35  0.0219 0.792
#>  9     1     1     1        1     1     9  3.77  0.191  0.412
#> 10     1     1     1        1     1    10  4.19  0.500  0.134
#> # ℹ 22 more rows
```

### Step 2. Pass to ggproto

``` r
StatEquilateral <- ggproto(
  `_class` = "StatAround", 
  `_inherit` = ggplot2::Stat,
  compute_panel = compute_panel_equilateral,
                      required_aes = c("x0", "y0", "r")
  #,
                      # default_aes = aes(group = row)
                      )
```

### Step 3. Write geom\_\* or stat\_\*

``` r
geom_equilateral <- function(
  mapping = NULL,
  data = NULL,
  position = "identity",
  na.rm = FALSE,
  show.legend = NA,
  inherit.aes = TRUE, ...) {
  ggplot2::layer(
    stat = StatEquilateral,  # proto object from Step 2
    geom = ggplot2::GeomPolygon,  # inherit other behavior
    data = data,
    mapping = mapping,
    position = position,
    show.legend = show.legend,
    inherit.aes = inherit.aes,
    params = list(na.rm = na.rm, ...)
  )
}
```

### Step 4: Enjoy (test)

``` r
data.frame(x0 = 0:1, y0 = 0:1, r = 1:2/3) %>% 
  ggplot() + 
  aes(x0 = x0, y0 = y0, r = r) + 
  geom_equilateral() + 
  aes(fill = r)
#> Joining with `by = join_by(join_var)`
```

![](man/figures/unnamed-chunk-47-1.png)<!-- -->

``` r

diamonds %>% 
  slice_sample(n = 80) %>% 
  ggplot() + 
  aes(x0 = cut %>% as.numeric, y0 = carat  , r = clarity %>% as.numeric()/20) + 
  geom_equilateral(alpha = .2) + 
  aes(fill = after_stat(r)) +
  coord_equal()
#> Joining with `by = join_by(join_var)`
```

![](man/figures/unnamed-chunk-47-2.png)<!-- -->

``` r

cars %>% 
  ggplot() + 
  aes(x0 = speed, y0 = dist, r = speed/dist) + 
  geom_equilateral()
#> Joining with `by = join_by(join_var)`
```

![](man/figures/unnamed-chunk-47-3.png)<!-- -->

``` r
  
cars %>% 
  sample_n(12) %>%  
  ggplot() + 
  aes(x0 = speed, y0 = dist, r = 3) + 
  geom_equilateral(color = "black") +
  coord_equal()
#> Joining with `by = join_by(join_var)`
```

![](man/figures/unnamed-chunk-47-4.png)<!-- -->

``` r

last_plot() + 
  aes(alpha = speed > 15)
#> Warning: Using alpha for a discrete variable is not advised.
#> Joining with `by = join_by(join_var)`
```

![](man/figures/unnamed-chunk-47-5.png)<!-- -->

``` r

last_plot() + 
  aes(linetype = dist > 20)
#> Warning: Using alpha for a discrete variable is not advised.
#> Joining with `by = join_by(join_var)`
```

![](man/figures/unnamed-chunk-47-6.png)<!-- -->

``` r

last_plot() + 
  aes(fill = speed > 20)
#> Warning: Using alpha for a discrete variable is not advised.
#> Joining with `by = join_by(join_var)`
```

![](man/figures/unnamed-chunk-47-7.png)<!-- -->

``` r

last_plot() + 
  facet_wrap(~ dist > 40)
#> Warning: Using alpha for a discrete variable is not advised.
#> Joining with `by = join_by(join_var)`
#> Joining with `by = join_by(join_var)`
```

![](man/figures/unnamed-chunk-47-8.png)<!-- -->

## geom\_us\_state: **1:1:n**

## geom\_ggcirclepack: **1:1:n, interdependance**

*a many-row geom for each row of the input data frame, with
interdependence between input observations.*

## geom\_ols: **n:k:w; interdependence**

*between-group computation*

## geom\_county: **1:1:1 via geometry sf**

*a geom defined by an sf geometry column*

## geom\_candlestick summarize first, then interdependence …

## geom\_pie: **n-\>1:1:1**

## geom\_wedge: **n-\>1:1:n**

# stat\_\* layers: keeping flexible via stat\_\* functions

## stat\_chull

Rather than defining geom functions, you might instead write stat\_\*
functions which can be used with a variety of geoms. Let’s contrast
geom\_chull and stat\_chull below.

``` r
geom_chull <- function(mapping = NULL, 
                        data = NULL,
                        position = "identity", 
                        na.rm = FALSE, 
                        show.legend = NA,
                        inherit.aes = TRUE, ...) {

  ggplot2::layer(
    stat = StatChull, 
    geom = ggplot2::GeomPolygon, 
    data = data, mapping = mapping,
    position = position, 
    show.legend = show.legend, 
    inherit.aes = inherit.aes,
    params = list(na.rm = na.rm, ...)
  )

}


stat_chull <- function(mapping = NULL, 
                       geom = ggplot2::GeomPolygon, 
                       data = NULL,
                       position = "identity", 
                       na.rm = FALSE, 
                       show.legend = NA,
                       inherit.aes = TRUE, ...) {

  ggplot2::layer(
    stat = StatChull, 
    geom = geom, 
    data = data, 
    mapping = mapping,
    position = position, 
    show.legend = show.legend, 
    inherit.aes = inherit.aes,
    params = list(na.rm = na.rm, ...)
  )

}
```

The construction is almost identical. However, in the stat version, the
geom is flexible because it can be user defined, instead of being
hard-coded in the function. Its use allows you to go in different visual
directions, but might have a higher cognative load.

``` r
p <- ggplot(data = mtcars) + 
  aes(x = wt, y = mpg) + 
  geom_point() 

p +
  stat_chull(alpha = .3)
```

![](man/figures/unnamed-chunk-26-1.png)<!-- -->

``` r

p +
  stat_chull(geom = "point",
             color = "red",
             size = 4)
```

![](man/figures/unnamed-chunk-26-2.png)<!-- -->

``` r

p + 
  stat_chull(geom = "text",
             label = "c-hull point",
             hjust = 0)
```

![](man/figures/unnamed-chunk-26-3.png)<!-- -->

``` r

# shows stat does not well-serve "path" geom
p + 
  stat_chull(geom = "path",
             label = "c-hull point",
             hjust = 0)
#> Warning in stat_chull(geom = "path", label = "c-hull point", hjust = 0):
#> Ignoring unknown parameters: `label` and `hjust`
```

![](man/figures/unnamed-chunk-26-4.png)<!-- -->

## stat\_waterfall: **1:1:1; interdependence**

Because the stat\_\* functions might require more cognitively from the
user, aliasing might be a good idea, creating one or more geoms\_\* the
stat layer.

The function geom\_waterfall() is exported from the ggwaterfall package,
which is being developed at github.com/EvaMaeRey/ggwaterfall

*One-row geom for each row in input dataset; geom interdependence*

A waterfall plot displays inflows and outflows that occur as a result of
events as well as the balance across multiple events. It is typically
displayed as a series of rectangles. Because the net change is displayed
(cumulative change), there is interdependence between the geometries on
our plot – where one rectangle ends, the next in the series begins.

In this example we’ll see how to alias the stat to a geom user-facing
function (stat\_waterfall -\> geom\_waterfall), and also how to change
the geom to allow for additional convenient user-facing functions
(stat\_waterfall -\> geom\_waterfall\_label). We prep to create
geom\_waterfall label by using the default\_aes slot in in the ggproto
step.

``` r
library(ggwaterfall)
library(tidyverse)
flow_df <- data.frame(
  event = c("Sales", "Refunds", "Payouts", "Court Losses", 
            "Court Wins", "Contracts", "Fees"),
           change = c(6400, -1100, 
                      -100, -4200, 3800, 
                      1400, -2800)) |> 
  mutate(event = factor(event))

flow_df
#>          event change
#> 1        Sales   6400
#> 2      Refunds  -1100
#> 3      Payouts   -100
#> 4 Court Losses  -4200
#> 5   Court Wins   3800
#> 6    Contracts   1400
#> 7         Fees  -2800

flow_df |> 
  ggplot() +
  geom_hline(yintercept = 0) +
  aes(change = change, 
      x = event) + # event in order
  geom_waterfall() + 
  geom_waterfall_label() + 
  scale_y_continuous(expand = expansion(.1)) + 
  scale_fill_manual(values = c("springgreen4", "darkred"))
```

![](man/figures/unnamed-chunk-27-1.png)<!-- -->

The strategy to create geom waterfall follows the standard four steps.

### Step 0

For ‘step 0’, we base ggplot2 to accomplish this task, and actually
pretty closely follow Hadley Wickham’s short paper that tackles a
waterfall plot with ggplot2.
<https://vita.had.co.nz/papers/ggplot2-wires.pdf>

### Steps 1 and 2

Then, we bundle up this computation into a function (step 1), called
compute\_panel\_waterfall. This function then define the compute\_panel
element in the ggproto object (step 2). We want the computation done
*panel-wise* because of the interdependence between the events, which
run along the x axis. Group-wise computation (the defining
compute\_group element), would fail us, as the cross-event
interdependence would not be preserved.

``` r
compute_panel_waterfall <- function(data, scales, width = .90){
  
  data %>% 
  mutate(x_scale = x) %>% 
  mutate(x_pos = x %>% as.numeric()) %>% 
  arrange(x_pos) %>% 
  mutate(balance = cumsum(c(0, 
                            change[-nrow(.)]))) %>% 
  mutate(direction = factor(sign(change))) %>% 
  mutate(xmin = x_pos - width/2,
         xmax = x_pos + width/2,
         ymin = balance,
         ymax = balance + change) %>% 
  mutate(x = x_pos) %>% 
  mutate(y = ymax) %>% 
  mutate(gain_loss = ifelse(direction == 1, "gain", "loss"))
  
}


### Step 1.1 Test compute 

# flow_df %>% 
#   rename(x = event) %>% 
#   compute_panel_waterfall() 



## Step 2. Pass compute to ggproto 

StatWaterfall <- ggplot2::ggproto(`_class` = "StatWaterfall", 
                         `_inherit` = ggplot2::Stat,
                         required_aes = c("change", "x"),
                         compute_panel = compute_panel_waterfall,
                         default_aes = ggplot2::aes(label = ggplot2::after_stat(change),
                                           fill = ggplot2::after_stat(gain_loss),
                                           vjust = ggplot2::after_stat((direction == -1) %>%
                                                                as.numeric)))
```

### Step 3

In step 3, we define stat\_waterfall, passing along StatWaterfall to
create a ggplot2 layer function. We include a standard set of arguments,
and we set the geom to ggplot2::GeomRect.

``` r
stat_waterfall <- function(geom = ggplot2::GeomRect, 
  mapping = NULL,
  data = NULL,
  position = "identity",
  na.rm = FALSE,
  show.legend = NA,
  inherit.aes = TRUE, ...) {
  ggplot2::layer(
    stat = StatWaterfall,  # proto object from step 2
    geom = geom,  # inherit other behavior
    data = data,
    mapping = mapping,
    position = position,
    show.legend = show.legend,
    inherit.aes = inherit.aes,
    params = list(na.rm = na.rm, ...)
  )
}

geom_waterfall <- stat_waterfall


geom_waterfall_label <- function(..., lineheight = .8){
  stat_waterfall(geom = "text", 
                 lineheight = lineheight, ...)}
```

### Step 4

In Step 4, we get to try out the functionality.

``` r
flow_df |> 
  ggplot() +
  geom_hline(yintercept = 0) +
  aes(change = change, 
      x = event) + # event in order
  geom_waterfall() + 
  geom_waterfall_label() + 
  scale_y_continuous(expand = expansion(.1)) + 
  scale_fill_manual(values = c("springgreen4", "darkred"))

last_plot() + 
  aes(x = fct_reorder(event, change))

last_plot() + 
  aes(x = fct_reorder(event, abs(change)))
```

<img src="man/figures/unnamed-chunk-30-1.png" width="33%" /><img src="man/figures/unnamed-chunk-30-2.png" width="33%" /><img src="man/figures/unnamed-chunk-30-3.png" width="33%" />

The final plot shows that while there are some convenience defaults for
label and fill, these can be over-ridden.

``` r
last_plot() + 
  aes(label = ifelse(change > 0, "gain", "loss")) + 
  aes(fill = NULL)
```

![](man/figures/unnamed-chunk-31-1.png)<!-- -->

# borrowing compute

geom\_smoothfit: **1:1:1** highjacking (?) an existing stats’s
computational powers

## geom\_smoothfit: **1:1:1** ggproto piggybacking on compute…

n:1:80 is geom\_smooth default.

``` r
library(ggsmoothfit)
```

# add default aesthetics

## geom\_barlab: Adding defaults to existing stats via ggproto editing

# modified start points; ggverbatim(),

## ggverbatim()

``` r
library(ggverbatim)
```

# ggedgelist()

``` r
library(ggedgelist)
```

# theme\_chalkboard()

``` r
#library(ggchalkboard)
```

# geom-led extension

## ggscatterplot

# wrapping fiddly functions (annotate and theme)

``` r
library(ggstamp)
```

``` r
#library(ggcons) # 
```

# make it a package: ggtedious *formal testing*

This is a placeholder for the ggtedious workshop, yet to be completed.

``` r
#library(ggtedius)
```
