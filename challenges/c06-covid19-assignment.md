COVID-19
================
Christopher Nie
2025-03-12

- [Grading Rubric](#grading-rubric)
  - [Individual](#individual)
  - [Submission](#submission)
- [The Big Picture](#the-big-picture)
- [Get the Data](#get-the-data)
  - [Navigating the Census Bureau](#navigating-the-census-bureau)
    - [**q1** Load Table `B01003` into the following tibble. Make sure
      the column names are
      `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.](#q1-load-table-b01003-into-the-following-tibble-make-sure-the-column-names-are-id-geographic-area-name-estimatetotal-margin-of-errortotal)
  - [Automated Download of NYT Data](#automated-download-of-nyt-data)
    - [**q2** Visit the NYT GitHub repo and find the URL for the **raw**
      US County-level data. Assign that URL as a string to the variable
      below.](#q2-visit-the-nyt-github-repo-and-find-the-url-for-the-raw-us-county-level-data-assign-that-url-as-a-string-to-the-variable-below)
- [Join the Data](#join-the-data)
  - [**q3** Process the `id` column of `df_pop` to create a `fips`
    column.](#q3-process-the-id-column-of-df_pop-to-create-a-fips-column)
  - [**q4** Join `df_covid` with `df_q3` by the `fips` column. Use the
    proper type of join to preserve *only* the rows in
    `df_covid`.](#q4-join-df_covid-with-df_q3-by-the-fips-column-use-the-proper-type-of-join-to-preserve-only-the-rows-in-df_covid)
- [Analyze](#analyze)
  - [Normalize](#normalize)
    - [**q5** Use the `population` estimates in `df_data` to normalize
      `cases` and `deaths` to produce per 100,000 counts \[3\]. Store
      these values in the columns `cases_per100k` and
      `deaths_per100k`.](#q5-use-the-population-estimates-in-df_data-to-normalize-cases-and-deaths-to-produce-per-100000-counts-3-store-these-values-in-the-columns-cases_per100k-and-deaths_per100k)
  - [Guided EDA](#guided-eda)
    - [**q6** Compute some summaries](#q6-compute-some-summaries)
    - [**q7** Find and compare the top
      10](#q7-find-and-compare-the-top-10)
  - [Self-directed EDA](#self-directed-eda)
    - [**q8** Drive your own ship: You’ve just put together a very rich
      dataset; you now get to explore! Pick your own direction and
      generate at least one punchline figure to document an interesting
      finding. I give a couple tips & ideas
      below:](#q8-drive-your-own-ship-youve-just-put-together-a-very-rich-dataset-you-now-get-to-explore-pick-your-own-direction-and-generate-at-least-one-punchline-figure-to-document-an-interesting-finding-i-give-a-couple-tips--ideas-below)
    - [Ideas](#ideas)
    - [Aside: Some visualization
      tricks](#aside-some-visualization-tricks)
    - [Geographic exceptions](#geographic-exceptions)
- [Notes](#notes)

*Purpose*: In this challenge, you’ll learn how to navigate the U.S.
Census Bureau website, programmatically download data from the internet,
and perform a county-level population-weighted analysis of current
COVID-19 trends. This will give you the base for a very deep
investigation of COVID-19, which we’ll build upon for Project 1.

<!-- include-rubric -->

# Grading Rubric

<!-- -------------------------------------------------- -->

Unlike exercises, **challenges will be graded**. The following rubrics
define how you will be graded, both on an individual and team basis.

## Individual

<!-- ------------------------- -->

| Category | Needs Improvement | Satisfactory |
|----|----|----|
| Effort | Some task **q**’s left unattempted | All task **q**’s attempted |
| Observed | Did not document observations, or observations incorrect | Documented correct observations based on analysis |
| Supported | Some observations not clearly supported by analysis | All observations clearly supported by analysis (table, graph, etc.) |
| Assessed | Observations include claims not supported by the data, or reflect a level of certainty not warranted by the data | Observations are appropriately qualified by the quality & relevance of the data and (in)conclusiveness of the support |
| Specified | Uses the phrase “more data are necessary” without clarification | Any statement that “more data are necessary” specifies which *specific* data are needed to answer what *specific* question |
| Code Styled | Violations of the [style guide](https://style.tidyverse.org/) hinder readability | Code sufficiently close to the [style guide](https://style.tidyverse.org/) |

## Submission

<!-- ------------------------- -->

Make sure to commit both the challenge report (`report.md` file) and
supporting files (`report_files/` folder) when you are done! Then submit
a link to Canvas. **Your Challenge submission is not complete without
all files uploaded to GitHub.**

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

*Background*:
[COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019) is
the disease caused by the virus SARS-CoV-2. In 2020 it became a global
pandemic, leading to huge loss of life and tremendous disruption to
society. The New York Times (as of writing) publishes up-to-date data on
the progression of the pandemic across the United States—we will study
these data in this challenge.

*Optional Readings*: I’ve found this [ProPublica
piece](https://www.propublica.org/article/how-to-understand-covid-19-numbers)
on “How to understand COVID-19 numbers” to be very informative!

# The Big Picture

<!-- -------------------------------------------------- -->

We’re about to go through *a lot* of weird steps, so let’s first fix the
big picture firmly in mind:

We want to study COVID-19 in terms of data: both case counts (number of
infections) and deaths. We’re going to do a county-level analysis in
order to get a high-resolution view of the pandemic. Since US counties
can vary widely in terms of their population, we’ll need population
estimates in order to compute infection rates (think back to the
`Titanic` challenge).

That’s the high-level view; now let’s dig into the details.

# Get the Data

<!-- -------------------------------------------------- -->

1.  County-level population estimates (Census Bureau)
2.  County-level COVID-19 counts (New York Times)

## Navigating the Census Bureau

<!-- ------------------------- -->

**Steps**: Our objective is to find the 2018 American Community
Survey\[1\] (ACS) Total Population estimates, disaggregated by counties.
To check your results, this is Table `B01003`.

1.  Go to [data.census.gov](data.census.gov).
2.  Scroll down and click `View Tables`.
3.  Apply filters to find the ACS **Total Population** estimates,
    disaggregated by counties. I used the filters:

- `Topics > Populations and People > Counts, Estimates, and Projections > Population Total`
- `Geography > County > All counties in United States`

5.  Select the **Total Population** table and click the `Download`
    button to download the data; make sure to select the 2018 5-year
    estimates.
6.  Unzip and move the data to your `challenges/data` folder.

- Note that the data will have a crazy-long filename like
  `ACSDT5Y2018.B01003_data_with_overlays_2020-07-26T094857.csv`. That’s
  because metadata is stored in the filename, such as the year of the
  estimate (`Y2018`) and my access date (`2020-07-26`). **Your filename
  will vary based on when you download the data**, so make sure to copy
  the filename that corresponds to what you downloaded!

### **q1** Load Table `B01003` into the following tibble. Make sure the column names are `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.

*Hint*: You will need to use the `skip` keyword when loading these data!

``` r
## TASK: Load the census bureau data with the following tibble name
df_pop <- read_csv("data/ACSDT5Y2018.B01003-Data.csv", col_names = TRUE)
```

    ## New names:
    ## Rows: 3221 Columns: 5
    ## ── Column specification
    ## ──────────────────────────────────────────────────────── Delimiter: "," chr
    ## (4): GEO_ID, NAME, B01003_001E, B01003_001M lgl (1): ...5
    ## ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    ## Specify the column types or set `show_col_types = FALSE` to quiet this message.
    ## • `` -> `...5`

``` r
new_colnames <- df_pop %>% slice(1) %>% unlist()
df_pop <- df_pop %>% slice(-1)
colnames(df_pop) <- new_colnames
colnames(df_pop)[1] <- "id"
df_pop <- df_pop %>% select(-ncol(df_pop))
df_pop
```

    ## # A tibble: 3,220 × 4
    ##    id            `Geographic Area Name` `Estimate!!Total` Margin of Error!!Tot…¹
    ##    <chr>         <chr>                  <chr>             <chr>                 
    ##  1 0500000US010… Autauga County, Alaba… 55200             *****                 
    ##  2 0500000US010… Baldwin County, Alaba… 208107            *****                 
    ##  3 0500000US010… Barbour County, Alaba… 25782             *****                 
    ##  4 0500000US010… Bibb County, Alabama   22527             *****                 
    ##  5 0500000US010… Blount County, Alabama 57645             *****                 
    ##  6 0500000US010… Bullock County, Alaba… 10352             *****                 
    ##  7 0500000US010… Butler County, Alabama 20025             *****                 
    ##  8 0500000US010… Calhoun County, Alaba… 115098            *****                 
    ##  9 0500000US010… Chambers County, Alab… 33826             *****                 
    ## 10 0500000US010… Cherokee County, Alab… 25853             *****                 
    ## # ℹ 3,210 more rows
    ## # ℹ abbreviated name: ¹​`Margin of Error!!Total`

*Note*: You can find information on 1-year, 3-year, and 5-year estimates
[here](https://www.census.gov/programs-surveys/acs/guidance/estimates.html).
The punchline is that 5-year estimates are more reliable but less
current.

## Automated Download of NYT Data

<!-- ------------------------- -->

ACS 5-year estimates don’t change all that often, but the COVID-19 data
are changing rapidly. To that end, it would be nice to be able to
*programmatically* download the most recent data for analysis; that way
we can update our analysis whenever we want simply by re-running our
notebook. This next problem will have you set up such a pipeline.

The New York Times is publishing up-to-date data on COVID-19 on
[GitHub](https://github.com/nytimes/covid-19-data).

### **q2** Visit the NYT [GitHub](https://github.com/nytimes/covid-19-data) repo and find the URL for the **raw** US County-level data. Assign that URL as a string to the variable below.

``` r
## TASK: Find the URL for the NYT covid-19 county-level data
url_counties <- "https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-counties.csv"
```

Once you have the url, the following code will download a local copy of
the data, then load the data into R.

``` r
## NOTE: No need to change this; just execute
## Set the filename of the data to download
filename_nyt <- "./data/nyt_counties.csv"

## Download the data locally
curl::curl_download(
        url_counties,
        destfile = filename_nyt
      )

## Loads the downloaded csv
df_covid <- read_csv(filename_nyt)
```

    ## Rows: 2502832 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (3): county, state, fips
    ## dbl  (2): cases, deaths
    ## date (1): date
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

You can now re-run the chunk above (or the entire notebook) to pull the
most recent version of the data. Thus you can periodically re-run this
notebook to check in on the pandemic as it evolves.

*Note*: You should feel free to copy-paste the code above for your own
future projects!

# Join the Data

<!-- -------------------------------------------------- -->

To get a sense of our task, let’s take a glimpse at our two data
sources.

``` r
## NOTE: No need to change this; just execute
df_pop %>% glimpse
```

    ## Rows: 3,220
    ## Columns: 4
    ## $ id                       <chr> "0500000US01001", "0500000US01003", "0500000U…
    ## $ `Geographic Area Name`   <chr> "Autauga County, Alabama", "Baldwin County, A…
    ## $ `Estimate!!Total`        <chr> "55200", "208107", "25782", "22527", "57645",…
    ## $ `Margin of Error!!Total` <chr> "*****", "*****", "*****", "*****", "*****", …

``` r
df_covid %>% glimpse
```

    ## Rows: 2,502,832
    ## Columns: 6
    ## $ date   <date> 2020-01-21, 2020-01-22, 2020-01-23, 2020-01-24, 2020-01-24, 20…
    ## $ county <chr> "Snohomish", "Snohomish", "Snohomish", "Cook", "Snohomish", "Or…
    ## $ state  <chr> "Washington", "Washington", "Washington", "Illinois", "Washingt…
    ## $ fips   <chr> "53061", "53061", "53061", "17031", "53061", "06059", "17031", …
    ## $ cases  <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
    ## $ deaths <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …

To join these datasets, we’ll need to use [FIPS county
codes](https://en.wikipedia.org/wiki/FIPS_county_code).\[2\] The last
`5` digits of the `id` column in `df_pop` is the FIPS county code, while
the NYT data `df_covid` already contains the `fips`.

### **q3** Process the `id` column of `df_pop` to create a `fips` column.

``` r
## TASK: Create a `fips` column by extracting the county code
df_q3 <- df_pop %>% 
  mutate(fips = str_sub(id,-5))
```

Use the following test to check your answer.

``` r
## NOTE: No need to change this
## Check known county
assertthat::assert_that(
              (df_q3 %>%
              filter(str_detect(`Geographic Area Name`, "Autauga County")) %>%
              pull(fips)) == "01001"
            )
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

### **q4** Join `df_covid` with `df_q3` by the `fips` column. Use the proper type of join to preserve *only* the rows in `df_covid`.

``` r
## TASK: Join df_covid and df_q3 by fips.
df_q4 <- df_q3 %>% 
  right_join(df_covid, by = "fips")
df_q4
```

    ## # A tibble: 2,502,832 × 10
    ##    id      `Geographic Area Name` `Estimate!!Total` Margin of Error!!Tot…¹ fips 
    ##    <chr>   <chr>                  <chr>             <chr>                  <chr>
    ##  1 050000… Autauga County, Alaba… 55200             *****                  01001
    ##  2 050000… Autauga County, Alaba… 55200             *****                  01001
    ##  3 050000… Autauga County, Alaba… 55200             *****                  01001
    ##  4 050000… Autauga County, Alaba… 55200             *****                  01001
    ##  5 050000… Autauga County, Alaba… 55200             *****                  01001
    ##  6 050000… Autauga County, Alaba… 55200             *****                  01001
    ##  7 050000… Autauga County, Alaba… 55200             *****                  01001
    ##  8 050000… Autauga County, Alaba… 55200             *****                  01001
    ##  9 050000… Autauga County, Alaba… 55200             *****                  01001
    ## 10 050000… Autauga County, Alaba… 55200             *****                  01001
    ## # ℹ 2,502,822 more rows
    ## # ℹ abbreviated name: ¹​`Margin of Error!!Total`
    ## # ℹ 5 more variables: date <date>, county <chr>, state <chr>, cases <dbl>,
    ## #   deaths <dbl>

Use the following test to check your answer.

``` r
## NOTE: No need to change this
if (!any(df_q4 %>% pull(fips) %>% str_detect(., "02105"), na.rm = TRUE)) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Your df_q4 contains a row for the Hoonah-Angoon Census Area (AK),",
    "which is not in df_covid. You used the incorrect join type.",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
if (any(df_q4 %>% pull(fips) %>% str_detect(., "78010"), na.rm = TRUE)) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Your df_q4 does not include St. Croix, US Virgin Islands,",
    "which is in df_covid. You used the incorrect join type.",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

For convenience, I down-select some columns and produce more convenient
column names.

``` r
## NOTE: No need to change; run this to produce a more convenient tibble
df_data <-
  df_q4 %>%
  select(
    date,
    county,
    state,
    fips,
    cases,
    deaths,
    population = `Estimate!!Total`
  )
```

# Analyze

<!-- -------------------------------------------------- -->

Now that we’ve done the hard work of loading and wrangling the data, we
can finally start our analysis. Our first step will be to produce county
population-normalized cases and death counts. Then we will explore the
data.

## Normalize

<!-- ------------------------- -->

### **q5** Use the `population` estimates in `df_data` to normalize `cases` and `deaths` to produce per 100,000 counts \[3\]. Store these values in the columns `cases_per100k` and `deaths_per100k`.

``` r
## TASK: Normalize cases and deaths
df_normalized <-
  df_data %>%
  mutate(
    population = as.numeric(population),
    cases_per100k = (cases / population)*100000, 
    deaths_per100k = (deaths / population)*100000
    
    )
```

You may use the following test to check your work.

``` r
## NOTE: No need to change this
## Check known county data
if (any(df_normalized %>% pull(date) %>% str_detect(., "2020-01-21"))) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Date 2020-01-21 not found; did you download the historical data (correct),",
    "or just the most recent data (incorrect)?",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
if (any(df_normalized %>% pull(date) %>% str_detect(., "2022-05-13"))) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Date 2022-05-13 not found; did you download the historical data (correct),",
    "or a single year's data (incorrect)?",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
## Check datatypes
assertthat::assert_that(is.numeric(df_normalized$cases))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$deaths))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$population))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$cases_per100k))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$deaths_per100k))
```

    ## [1] TRUE

``` r
## Check that normalization is correct
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(cases_per100k) - 0.127) < 1e-3
            )
```

    ## [1] TRUE

``` r
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(deaths_per100k) - 0) < 1e-3
            )
```

    ## [1] TRUE

``` r
print("Excellent!")
```

    ## [1] "Excellent!"

## Guided EDA

<!-- ------------------------- -->

Before turning you loose, let’s complete a couple guided EDA tasks.

### **q6** Compute some summaries

Compute the mean and standard deviation for `cases_per100k` and
`deaths_per100k`. *Make sure to carefully choose **which rows** to
include in your summaries,* and justify why!

``` r
############################
# isolate cases
############################
df_normalized_NAremoved <- df_normalized %>% 
  filter(!is.na(cases_per100k)) %>% 
  filter(is.finite(cases_per100k))
  
lower_bound <- quantile(df_normalized_NAremoved$cases_per100k, 0.25) - 1.5*IQR(df_normalized_NAremoved$cases_per100k)
upper_bound <- quantile(df_normalized_NAremoved$cases_per100k, 0.75) + 1.5*IQR(df_normalized_NAremoved$cases_per100k)

df_cases100k <- 
  df_normalized_NAremoved %>% 
  filter(population >= 10000) %>% # smaller populations disproportionally affect the cases per 100k
  filter(cases_per100k >= lower_bound & cases_per100k <= upper_bound)# outliers affect mean and sd

############################
# isolate deaths
############################


df_normalized_NAremoved <- df_normalized %>% 
  filter(!is.na(deaths_per100k)) %>% 
  filter(is.finite(deaths_per100k))
  
lower_bound <- quantile(df_normalized_NAremoved$deaths_per100k, 0.25) - 1.5*IQR(df_normalized_NAremoved$deaths_per100k)
upper_bound <- quantile(df_normalized_NAremoved$deaths_per100k, 0.75) + 1.5*IQR(df_normalized_NAremoved$deaths_per100k)

df_deaths100k <- 
  df_normalized_NAremoved %>% 
  filter(population >= 10000) %>% # smaller populations disproportionally affect the cases per 100k
  filter(deaths_per100k >= lower_bound & deaths_per100k <= upper_bound)# outliers affect mean and sd


mean_sd <- tibble(
  `data` = c("Cases", "Deaths"), 
  `mean` = c(mean(df_cases100k$cases_per100k),mean(df_deaths100k$deaths_per100k)),
  `sd` = c(sd(df_cases100k$cases_per100k), sd(df_deaths100k$deaths_per100k))
)

mean_sd
```

    ## # A tibble: 2 × 3
    ##   data    mean    sd
    ##   <chr>  <dbl> <dbl>
    ## 1 Cases  9828. 8281.
    ## 2 Deaths  165.  142.

``` r
## TASK: Compute mean and sd for cases_per100k and deaths_per100k
```

- Which rows did you pick? Why?
  - I filtered out any NA values. This appears to be necessary for the
    `mean` and `sd` function to work.
  - I filtered out the outliers, as outliers tend to have an extremely
    strong impact on mean and standard deviations.
  - I filtered out the counties with populations with less than 10000,
    as a small change disproportionately affects the the cases per 100k.
    For example, a single case in a county with a population of 1,000
    would result in 100 cases per 100000. Although mathematically, this
    is the normalization doing its job, I felt like it would affect the
    mean and standard deviations, so I decided to remove it.

### **q7** Find and compare the top 10

Find the top 10 counties in terms of `cases_per100k`, and the top 10 in
terms of `deaths_per100k`. Report the population of each county along
with the per-100,000 counts. Compare the counts against the mean values
you found in q6. Note any observations.

``` r
## TASK: Find the top 10 max cases_per100k counties; report populations as well

top_cases <- df_normalized %>% 
  arrange(desc(cases_per100k)) %>% 
  distinct(county, .keep_all = TRUE) %>% 
  head(10)

top_deaths <- df_normalized %>% 
  arrange(desc(deaths_per100k)) %>% 
  distinct(county, .keep_all = TRUE) %>% 
  head(10)

top_cases
```

    ## # A tibble: 10 × 9
    ##    date       county           state fips  cases deaths population cases_per100k
    ##    <date>     <chr>            <chr> <chr> <dbl>  <dbl>      <dbl>         <dbl>
    ##  1 2022-05-12 Loving           Texas 48301   196      1        102       192157.
    ##  2 2022-05-11 Chattahoochee    Geor… 13053  7486     22      10767        69527.
    ##  3 2022-05-11 Nome Census Area Alas… 02180  6245      5       9925        62922.
    ##  4 2022-05-11 Northwest Arcti… Alas… 02188  4837     13       7734        62542.
    ##  5 2022-05-13 Crowley          Colo… 08025  3347     30       5630        59449.
    ##  6 2022-05-11 Bethel Census A… Alas… 02050 10362     41      18040        57439.
    ##  7 2022-03-30 Dewey            Sout… 46041  3139     42       5779        54317.
    ##  8 2022-05-12 Dimmit           Texas 48127  5760     51      10663        54019.
    ##  9 2022-05-12 Jim Hogg         Texas 48247  2648     22       5282        50133.
    ## 10 2022-05-11 Kusilvak Census… Alas… 02158  4084     14       8198        49817.
    ## # ℹ 1 more variable: deaths_per100k <dbl>

``` r
top_deaths
```

    ## # A tibble: 10 × 9
    ##    date       county           state fips  cases deaths population cases_per100k
    ##    <date>     <chr>            <chr> <chr> <dbl>  <dbl>      <dbl>         <dbl>
    ##  1 2022-02-19 McMullen         Texas 48311   166      9        662        25076.
    ##  2 2022-04-27 Galax city       Virg… 51640  2551     78       6638        38430.
    ##  3 2022-03-10 Motley           Texas 48345   271     13       1156        23443.
    ##  4 2022-04-20 Hancock          Geor… 13141  1577     90       8535        18477.
    ##  5 2022-04-19 Emporia city     Virg… 51595  1169     55       5381        21725.
    ##  6 2022-04-27 Towns            Geor… 13281  2396    116      11417        20986.
    ##  7 2022-02-14 Jerauld          Sout… 46073   404     20       2029        19911.
    ##  8 2022-03-04 Loving           Texas 48301   165      1        102       161765.
    ##  9 2022-02-03 Robertson        Kent… 21201   570     21       2143        26598.
    ## 10 2022-05-05 Martinsville ci… Virg… 51690  3452    124      13101        26349.
    ## # ℹ 1 more variable: deaths_per100k <dbl>

``` r
## TASK: Find the top 10 deaths_per100k counties; report populations as well
```

**Observations**:

- The largest values are way above the mean, with the maximum
  `cases_per100k` almost 19x that of the mean, and triple the value of
  the next highest `cases_per100k`. It is well outside of the 3 standard
  deviations that make up the IQR. Thus, it is an outlier. This number
  is probably because of the small population of 102. This would be an
  example of an outlier / small population bias that we removed for the
  previous question. Another interesting observation we can make is that
  there are more cases than the population, which means that the
  `cases_per100k` is actually greater than `100000`. Another observation
  about the `cases_per100k` is that all values on the list is outside
  the outlier. However, of the ten counties listed, only three of them
  pass the minimum population of 10000 to be included in the mean. Thus,
  the fact that they are outliers is not due to their population size.
- The maximum `deaths_per100k` was almost 10x that of the mean. All of
  their populations are smaller than `10000`, so we filtered these out
  for the mean.
- When did these “largest values” occur?
  - All of these largest values occurred in the year 2022, with all of
    the highest `cases_per100k` in May. The highest `deaths_per100k`
    occurred between February and July

## Self-directed EDA

<!-- ------------------------- -->

### **q8** Drive your own ship: You’ve just put together a very rich dataset; you now get to explore! Pick your own direction and generate at least one punchline figure to document an interesting finding. I give a couple tips & ideas below:

### Ideas

<!-- ------------------------- -->

- Look for outliers.
- Try web searching for news stories in some of the outlier counties.
- Investigate relationships between county population and counts.
- Do a deep-dive on counties that are important to you (e.g. where you
  or your family live).
- Fix the *geographic exceptions* noted below to study New York City.
- Your own idea!

**DO YOUR OWN ANALYSIS HERE**

``` r
df_q8 <- df_normalized %>%
  filter(county == "San Francisco" | county == "Los Angeles") %>% 
  pivot_longer(
    cols = c(cases_per100k, deaths_per100k),
    names_to = "metric",
    values_to = "value"
  )

ggplot(df_q8, aes(x = date, y = value, color = county)) +
  geom_line(linewidth = 1) +
  facet_wrap(~metric, scales = "free_y", ncol = 1, strip.position = "left") 
```

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

- I am from San Francisco, California, so I wanted to explore San
  Francisco’s response to COVID-19.

- Los Angeles and San Francisco have virtually the same shape in both
  `cases_per100k` and `deaths_per100k` . It almost looks like Los
  Angeles is simply a scaled version of the San Francisco graphs. They
  even have the same peak locations. However, it is important to
  remember that Los Angeles has a much higher population than San
  Francisco. Thus, in practice, the actual cases and deaths is much
  higher than that of San Francisco.

- This shape similarity made me wonder if this was because they were
  both population centers of California.

- Thus, I wanted to look over other counties in California.

``` r
samples <- c("Los Angeles", "Merced", "Alameda", "San Diego", "Riverside", "San Diego", "Yolo", "Santa Barbara", "San Francisco")
df_q8_1 <- df_normalized %>%
  filter(county %in% samples) %>% 
  pivot_longer(
    cols = c(cases_per100k, deaths_per100k),
    names_to = "metric",
    values_to = "value"
  )

ggplot(df_q8_1, aes(x = date, y = value, color = county)) +
  geom_line(linewidth = 1) +
  facet_wrap(~metric, scales = "free_y", ncol = 1, strip.position = "left") +
  scale_color_brewer(palette="Accent")
```

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

- I looked at the counties where the various Universities of California
  (UC) campuses are located. I chose not to graph UC Irvine and UC Santa
  Cruz because their graphs had a lot of noise. Most noticeably, all of
  these locations can be considered major metropolitan areas. We can see
  that most of the `cases_per100k` look like a scaling factor was
  applied to some other city’s graph.

- However, the `death_per100k` looks quite different for each county.
  According to one source \[1\], LA’s high death rate and case rate
  might be due to factors such as “people per bedroom” rather than just
  “people per square foot”. In other words, another metric to measure
  overcrowded households. The other top contenders for `deaths_per100k`
  is Merced and Riverside. It is interesting to see that they are also
  among the highest for `cases_per100k`. These are two primarily
  college-based areas. Thus, it is possible that, due to student housing
  situations, they also have higher “people per bedroom” compared to the
  other counties with UC’s, which are more independent of the students
  at the UC.

- I wanted to see if the same `cases_per100k` trends could be seen
  across smaller counties of California.

``` r
samples <- c("San Francisco", "Solano", "Sonoma", "Stanislaus", "Glenn", "Lassen", "Madera")
df_q8_1 <- df_normalized %>%
  filter(county %in% samples) %>% 
  pivot_longer(
    cols = c(cases_per100k, deaths_per100k),
    names_to = "metric",
    values_to = "value"
  )

ggplot(df_q8_1, aes(x = date, y = value, color = county)) +
  geom_line(linewidth = 1) +
  facet_wrap(~metric, scales = "free_y", ncol = 1, strip.position = "left") +
  scale_color_brewer(palette="Accent")
```

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

- Again, we see similar trends for `cases_per100k` for peak locations.
  The variability in `death_per100k` shows that it is not too related to
  `cases_per100k`, and that there are many other factors in play to
  determine it.

- Next, I want to see if the `cases_per100k` was due to the COVID-19
  strands, or if it was unique to California. Thus, I looked outside of
  California.

``` r
samples <- c("Solano", "Ada", "Spokane", "Washtenaw", "Prince William", "Nueces", "Onondaga", "Monterey")
df_q8_1 <- df_normalized %>%
  filter(county %in% samples) %>% 
  pivot_longer(
    cols = c(cases_per100k, deaths_per100k),
    names_to = "metric",
    values_to = "value"
  )

ggplot(df_q8_1, aes(x = date, y = value, color = county)) +
  geom_line(linewidth = 1) +
  facet_wrap(~metric, scales = "free_y", ncol = 1, strip.position = "left") +
  scale_color_brewer(palette="Accent")
```

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

- I chose counties with a population of ~400000 (mid size county). For
  context, Needham, MA is part of the Norfolk county which has a
  population of ~700000

  - Ada, ID

  - Monterey, CA

  - Nueces, TX

  - Onondaga, NY

  - Prince William, VA

  - Solano, CA

  - Spokane, WA

  - Washtenaw, MI

- Looking at the `cases_per100k` graph, we can see a lot of variation.
  Whereas the time history of the various California counties looked
  almost parallel despite the varying population sizes, the time history
  of the various counties (with similar population sizes) contains a lot
  of intersection and back-and-forth crossings.

- Thus, it looks like California’s similarities for `cases_per100k`
  across counties was due to the fact that it was in the same state. The
  actual cause can be anything from policies to travel patterns or
  access to healthcare / ability.

``` r
df_normalized %>% 
  filter(state == "California" & date == "2020-08-23") %>% 
  arrange(desc(population))
```

    ## # A tibble: 58 × 9
    ##    date       county         state  fips   cases deaths population cases_per100k
    ##    <date>     <chr>          <chr>  <chr>  <dbl>  <dbl>      <dbl>         <dbl>
    ##  1 2020-08-23 Los Angeles    Calif… 06037 231695   5545   10098052         2294.
    ##  2 2020-08-23 San Diego      Calif… 06073  36603    660    3302833         1108.
    ##  3 2020-08-23 Orange         Calif… 06059  45954    897    3164182         1452.
    ##  4 2020-08-23 Riverside      Calif… 06065  49944    927    2383286         2096.
    ##  5 2020-08-23 San Bernardino Calif… 06071  45035    691    2135413         2109.
    ##  6 2020-08-23 Santa Clara    Calif… 06085  15688    225    1922200          816.
    ##  7 2020-08-23 Alameda        Calif… 06001  16744    234    1643700         1019.
    ##  8 2020-08-23 Sacramento     Calif… 06067  15515    234    1510023         1027.
    ##  9 2020-08-23 Contra Costa   Calif… 06013  12663    169    1133247         1117.
    ## 10 2020-08-23 Fresno         Calif… 06019  23197    226     978130         2372.
    ## # ℹ 48 more rows
    ## # ℹ 1 more variable: deaths_per100k <dbl>

- \[1\] Schmidt, Grayson. *Why does LA have so many COVID-19 cases,* USC
  Today. <a href="#0"
  class="uri">https://today.usc.edu/los-angeles-covid-19-cases-density-housing-overcrowding-usc-experts/</a>

### Aside: Some visualization tricks

<!-- ------------------------- -->

These data get a little busy, so it’s helpful to know a few `ggplot`
tricks to help with the visualization. Here’s an example focused on
Massachusetts.

``` r
## NOTE: No need to change this; just an example
df_normalized %>%
  filter(
    state == "Massachusetts", # Focus on Mass only
    !is.na(fips), # fct_reorder2 can choke with missing data
  ) %>%

  ggplot(
    aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k))
  ) +
  geom_line() +
  scale_y_log10(labels = scales::label_number()) +
  scale_color_discrete(name = "County") +
  theme_minimal() +
  labs(
    x = "Date",
    y = "Cases (per 100,000 persons)"
  )
```

![](c06-covid19-assignment_files/figure-gfm/ma-example-1.png)<!-- -->

*Tricks*:

- I use `fct_reorder2` to *re-order* the color labels such that the
  color in the legend on the right is ordered the same as the vertical
  order of rightmost points on the curves. This makes it easier to
  reference the legend.
- I manually set the `name` of the color scale in order to avoid
  reporting the `fct_reorder2` call.
- I use `scales::label_number_si` to make the vertical labels more
  readable.
- I use `theme_minimal()` to clean up the theme a bit.
- I use `labs()` to give manual labels.

### Geographic exceptions

<!-- ------------------------- -->

The NYT repo documents some [geographic
exceptions](https://github.com/nytimes/covid-19-data#geographic-exceptions);
the data for New York, Kings, Queens, Bronx and Richmond counties are
consolidated under “New York City” *without* a fips code. Thus the
normalized counts in `df_normalized` are `NA`. To fix this, you would
need to merge the population data from the New York City counties, and
manually normalize the data.

# Notes

<!-- -------------------------------------------------- -->

\[1\] The census used to have many, many questions, but the ACS was
created in 2010 to remove some questions and shorten the census. You can
learn more in [this wonderful visual
history](https://pudding.cool/2020/03/census-history/) of the census.

\[2\] FIPS stands for [Federal Information Processing
Standards](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards);
these are computer standards issued by NIST for things such as
government data.

\[3\] Demographers often report statistics not in percentages (per 100
people), but rather in per 100,000 persons. This is [not always the
case](https://stats.stackexchange.com/questions/12810/why-do-demographers-give-rates-per-100-000-people)
though!
