
Searching GitHub via the API
----------------------------

A "five minute exercise" triggered by a twitter conversation with [@pssguy](https://github.com/pssguy) re: his [Shiny app for inspecting someone's GitHub issues](https://mytinyshinys.shinyapps.io/githubAnalyses/).

Goal: do this issue search via the GitHub API:

<https://github.com/issues?q=is%3Aissue+author%3Atimelyportfolio+is%3Aopen>

### curl

In the browser, I see 194 open issues (I wrote this 2016-02-11).

I can get them from the API at the command line with this:

``` bash
curl -o out.json https://api.github.com/search/issues?q=is:issue+author:timelyportfolio+state:open
head out.json
```

    ##   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
    ##                                  Dload  Upload   Total   Spent    Left  Speed
    ## 
      0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
      0 71147    0   267    0     0    445      0  0:02:39 --:--:--  0:02:39   445
    100 71147  100 71147    0     0  81011      0 --:--:-- --:--:-- --:--:-- 81033
    ## {
    ##   "total_count": 194,
    ##   "incomplete_results": false,
    ##   "items": [
    ##     {
    ##       "url": "https://api.github.com/repos/hadley/svglite/issues/58",
    ##       "repository_url": "https://api.github.com/repos/hadley/svglite",
    ##       "labels_url": "https://api.github.com/repos/hadley/svglite/issues/58/labels{/name}",
    ##       "comments_url": "https://api.github.com/repos/hadley/svglite/issues/58/comments",
    ##       "events_url": "https://api.github.com/repos/hadley/svglite/issues/58/events",

### gh

Here's the relevant GitHub API endpoint:

<https://developer.github.com/v3/search/#search-issues>

We can't just provide the search terms as params to `gh()`. They need to be pre-processed into a query string. This little exercise exposed some problems:

-   with `gh`: This structure of what this endpoint returns is [not compatible with the current approach to page traversal](https://github.com/gaborcsardi/gh/issues/33).
-   and maybe `httr` as well: [This issue](https://github.com/hadley/httr/issues/335) about the encoding/decoding of `+` may be part of the story here.

But that can all be worked around!

#### Retrieve issues via API search

``` r
## devtools::install_github("gaborcsardi/gh")
library(gh)
library(purrr)

author <- "timelyportfolio"
state <- "open"
is <- "issue"

search_q <- list(author = author, state = state, is = is)
(search_q <- paste(names(search_q), search_q, sep = ":", collapse = " "))
```

    ## [1] "author:timelyportfolio state:open is:issue"

``` r
res <- gh("/search/issues", q = search_q, .limit = Inf) 
## OK this is not ideal but we can work with it
str(res, max.level = 1)
```

    ## List of 21
    ##  $ total_count       : int 194
    ##  $ incomplete_results: logi FALSE
    ##  $ items             :List of 30
    ##  $ NA                : int 194
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 194
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 194
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 194
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 194
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 194
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 14
    ##  - attr(*, "method")= chr "GET"
    ##  - attr(*, "response")=List of 25
    ##   ..- attr(*, "class")= chr [1:2] "insensitive" "list"
    ##  - attr(*, ".send_headers")= Named chr [1:2] "application/vnd.github.v3+json" "https://github.com/gaborcsardi/whoami"
    ##   ..- attr(*, "names")= chr [1:2] "Accept" "User-Agent"
    ##  - attr(*, "class")= chr [1:2] "gh_response" "list"

`res` now contains what we need, in a terribly awkward form, because `gh`'s current approach to traversing pages is not prepared for the unexpected behavior of the API's search endpoint :confused:. It returns something quite different from the other endpoints.

#### Extract info and put it in a data frame

Let's dig out what we need. I display the top of a data frame with one row per issue and, for now, issue title and it's browser URL.

``` r
good_stuff <- res %>% 
  keep(is_list) %>% # this happens to retain what we want = pages of results
  flatten()         # get one big list of issues
df <- good_stuff %>%
 map_df(`[`, c("title", "html_url")) # extract other bits as needed here!
df %>%                               # just for display purposes
  dplyr::transmute(title = substr(title, 1, 30),
                   html_url = paste0("...", substr(html_url, 20, 45), "..."))
```

    ## Source: local data frame [194 x 2]
    ## 
    ##                             title                         html_url
    ##                             (chr)                            (chr)
    ## 1  performance hit with svgstring   ...hadley/svglite/issues/58...
    ## 2  fix editSVG to work with new i   ...hadley/svglite/issues/56...
    ## 3                       accordion ...timelyportfolio/buildingwi...
    ## 4                   love the idea ...w8r/leaflet-schematic/issu...
    ## 5             add d3-lasso-plugin   ...juba/scatterD3/issues/19...
    ## 6  consider/add another xml viewe  ...hrbrmstr/xmlview/issues/3...
    ## 7  link to listviewer issue reque  ...hrbrmstr/xmlview/issues/1...
    ## 8             resizable component ...timelyportfolio/buildingwi...
    ## 9               doSlide or swiper ...timelyportfolio/buildingwi...
    ## 10 allow non-date data with d3kit ...timelyportfolio/timelineR/...
    ## ..                            ...                              ...
