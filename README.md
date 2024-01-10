
  - [ggplot2-extension-cookbook](#ggplot2-extension-cookbook)
  - [Preface and acknowledgements](#preface-and-acknowledgements)
  - [geom and stat layers](#geom-and-stat-layers)
      - [geom\_coordinates: **1:1:1**](#geom_coordinates-111)
      - [geom\_xy\_means: **n:1:1**](#geom_xy_means-n11)
      - [geom\_circle: **1:1:n**](#geom_circle-11n)
      - [geom\_us\_state: **1:1:n**](#geom_us_state-11n)
      - [geom\_ggcirclepack: **1:1:n,
        interdependance**](#geom_ggcirclepack-11n-interdependance)
      - [geom\_ols: **n:k:w;
        interdependence**](#geom_ols-nkw-interdependence)
      - [geom\_county: **1:1:1 via geometry
        sf**](#geom_county-111-via-geometry-sf)
      - [geom\_waterfall: **1:1:1;
        interdependence**](#geom_waterfall-111-interdependence)
          - [Step 0](#step-0)
          - [Steps 1 and 2](#steps-1-and-2)
          - [Step 3](#step-3)
          - [Step 4](#step-4)
  - [summarize first, then interdependence
    …](#summarize-first-then-interdependence-)
      - [geom\_pie: **n-\>1:1:1**](#geom_pie-n-111)
      - [geom\_wedge: **n-\>1:1:n**](#geom_wedge-n-11n)
  - [repurposing an existing stats](#repurposing-an-existing-stats)
      - [geom\_smoothfit: ggproto
        piggybacking](#geom_smoothfit-ggproto-piggybacking)
      - [geom\_barlab: Adding defaults to existing stats via ggproto
        editing](#geom_barlab-adding-defaults-to-existing-stats-via-ggproto-editing)
  - [modifying ggplot() function (data
    transformation)](#modifying-ggplot-function-data-transformation)
  - [modifying ggraph() function](#modifying-ggraph-function)
  - [defining theme…](#defining-theme)
  - [pseudo extension?](#pseudo-extension)
      - [geom-led funtions.](#geom-led-funtions)
  - [wrapping fiddly functions (annotate and
    theme)](#wrapping-fiddly-functions-annotate-and-theme)
  - [ggtedious *formal testing*](#ggtedious-formal-testing)

<!-- README.md is generated from README.Rmd. Please edit that file -->

# ggplot2-extension-cookbook

<!-- badges: start -->

<!-- badges: end -->

*The ggplot2 Extension Cookbook* aims to provide ggplot2 extension
strategies in a consistent and accessible way.

We group the content by extension type, provide demonstrations of their
use, and then link (for now) to the code that defines them. In that
material, I’ll try to stick to a formula to orient you to the ggplot2
extension:

  - Step 0: get job done with ‘base’ ggplot2
  - Step 1: Write a function for the ‘compute’
  - Step 2: Pass the compute to ggproto
  - Step 3: Pass ggproto to a user-facing function for use in a ggplot()
    pipeline
  - Step 4: Try out/test/enjoy\!

Furthermore, we’ll try to orient with a ratio typology that you’ll see
in the section titles.

Finally, most of the code is at the ‘R for data science level, and not
’Advanced R’ level, which hopefully will afford greater reach.

# Preface and acknowledgements

I attended Thomas Lin Pederson’s January 2020 talk ‘Extending your
ability to extend ggplot2’ seated on the floor of a packed out ballroom.
The talk had the important central message - “you can be a ggplot2
extender”. And since then, I aimed to be an extender. I hope *ggplot2
Extension Cookbook* will help you in your extender journey and, if you
are fluent in R and ggplot2, also says to you “you can be a ggplot2
extender”.

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
more primitive geom. Working with first and second year undergrad
students meant that I had to think about and formulate workflow - where
reference material would even be accessibility to very new R users. As
veterans of just one or two stats classes that used R and ggplot2, what
would they find familiar and accessible? What might we be able to
de-emphasize and still succeed? I’m grateful to these This worked with
students and I had a chance to refine it further.

The following steps emerged and persisted into our tutorial formula:

  - Step 0: get job done with ‘base’ ggplot2
  - Step 1: Write a function for the ‘compute’
  - Step 2: Pass the compute to ggproto
  - Step 3: Pass ggproto to stat/geom/facet function
  - Step 4: Try out/test/enjoy\!

Impressed that … Novice -\> extender, in 4 months.

I wondered how the strategy would perform with experienced R and ggplot2
users. Being an academic, further assessment sounded attractive and I
went down the route of devising a tutorial \[with assistance from
independent study student Morgan Brown\] and formally getting feedback
on it via focus groups and a survey, after refining the tutorial.

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

The feedback on the easy-geom-recipes was collected in March 2023.

So, I’m interested in simply developing new, useful extensions.

To try to make those experiences valuable to others, I follow the
‘recipe’ formula as much as possible so that as strategies morph, one
still recognizes where we are at a high-level in the process.

Now to serve the community and for use in this *ggplot2 Extension
Cookbook*, I’m using an in-README documentation strategy for my ggplot2
extension packages so that the development stories are in one place
instead of scattered between package files.

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

# geom and stat layers

<data:n> definding geom:geom

## geom\_coordinates: **1:1:1**

*a row in the input dataframe will display a single geometry, which will
be defined by as single row in the internal data*

``` r
ggxmean::geom_text_coordinate()
```

Note: in this situation, you can also get away with delayed aesthetic
evaluation. See also redefaulting.

## geom\_xy\_means: **n:1:1**

*many rows from a dataset: will be summarized and visualized by as
single mark: the mark will be defined by one row of data*

``` r
ggxmean::geom_xy_means()
```

## geom\_circle: **1:1:n**

*a single row in a dataframe: will be visualized by a single mark: the
mark will be defined by many-row in an internal dataframe*

for each row in the dataframe, a single geometry is visualized, but each
geom is defined by many rows…

## geom\_us\_state: **1:1:n**

``` r
library(ggcircle)
```

## geom\_ggcirclepack: **1:1:n, interdependance**

*a many-row geom for each row of the input data frame, with
interdependence between input observations.*

``` r
library(ggcirclepack)
```

## geom\_ols: **n:k:w; interdependence**

*between-group computation*

``` r
library(ggols)
```

## geom\_county: **1:1:1 via geometry sf**

*a geom defined by an sf geometry column*

``` r
library(ggnorthcarolina)
```

## geom\_waterfall: **1:1:1; interdependence**

*One-row geom for each row in input dataset; geom interdependence*

A waterfall plot displays inflows and outflows that occur as a result of
events as well as the balance across multiple events. It is typically
displayed as a series of rectangles. Because the net change is displayed
(cumulative change), there is interdependence between the geometries on
our plot – where one rectangle ends, the next in the series begins.

``` r
library(ggwaterfall)
library(tidyverse)
#> ── Attaching core tidyverse packages ─────────────────── tidyverse 2.0.0.9000 ──
#> ✔ dplyr     1.1.0          ✔ readr     2.1.4     
#> ✔ forcats   1.0.0          ✔ stringr   1.5.0     
#> ✔ ggplot2   3.4.4.9000     ✔ tibble    3.2.1     
#> ✔ lubridate 1.9.2          ✔ tidyr     1.3.0     
#> ✔ purrr     1.0.1          
#> ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
#> ✖ dplyr::filter() masks stats::filter()
#> ✖ dplyr::lag()    masks stats::lag()
#> ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
flow_df <- data.frame(event = c(
                     "Sales", 
                     "Refunds",
                     "Payouts", 
                     "Court Losses", 
                     "Court Wins", 
                     "Contracts", 
                     "Fees"),
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

![](man/figures/unnamed-chunk-8-1.png)<!-- -->

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
#' Title
#'
#' @param data 
#' @param scales 
#' @param width 
#'
#' @return
#' @export
#'
#' @examples
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

### Step 4

In Step 4, we get to try out the functionality.

# summarize first, then interdependence …

``` r
library(ggcandlestick)
```

## geom\_pie: **n-\>1:1:1**

## geom\_wedge: **n-\>1:1:n**

# repurposing an existing stats

## geom\_smoothfit: ggproto piggybacking

``` r
library(ggsmoothfit)
```

## geom\_barlab: Adding defaults to existing stats via ggproto editing

# modifying ggplot() function (data transformation)

``` r
library(ggverbatim)
```

# modifying ggraph() function

``` r
library(ggedgelist)
```

# defining theme…

``` r
library(ggchalkboard)
```

# pseudo extension?

## geom-led funtions.

# wrapping fiddly functions (annotate and theme)

``` r
library(ggstamp)
```

``` r
library(ggcons) # 
```

# ggtedious *formal testing*

``` r
library(ggtedius)
```
