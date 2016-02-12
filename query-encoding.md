
*Taking a closer look at the query encoding issue. Eventually led to comments on [iss \#335 encoding/decoding '+' as space (rather than literal '+' sign)](https://github.com/hadley/httr/issues/335) and a [pull request](https://github.com/hadley/httr/pull/337).*

Example of advanced Github search in the browser (purposefully chose example with small number of issues -- this is not about pagination):

[https://github.com/search?utf8=✓&q=type%3Aissue+author%3Ahadley+repo%3Arstudio%2Frstudioapi+state%3Aopen&type=Issues&ref=searchresults](https://github.com/search?utf8=✓&q=type%3Aissue+author%3Ahadley+repo%3Arstudio%2Frstudioapi+state%3Aopen&type=Issues&ref=searchresults)

I see 4 issues.

Get them via API from the command line

``` bash
curl -o hadley-rstudioapi.json https://api.github.com/search/issues?q=type:issue+author:hadley+repo:rstudio/rstudioapi+state:open
head hadley-rstudioapi.json
#>   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
#>                                  Dload  Upload   Total   Spent    Left  Speed
#> 
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100  8118  100  8118    0     0  14316      0 --:--:-- --:--:-- --:--:-- 14342
#> {
#>   "total_count": 4,
#>   "incomplete_results": false,
#>   "items": [
#>     {
#>       "url": "https://api.github.com/repos/rstudio/rstudioapi/issues/17",
#>       "repository_url": "https://api.github.com/repos/rstudio/rstudioapi",
#>       "labels_url": "https://api.github.com/repos/rstudio/rstudioapi/issues/17/labels{/name}",
#>       "comments_url": "https://api.github.com/repos/rstudio/rstudioapi/issues/17/comments",
#>       "events_url": "https://api.github.com/repos/rstudio/rstudioapi/issues/17/events",
```

Again, 4 issues.

How to get them from R? First, assemble all terms in a list.

``` r
library(gh)
library(purrr)
library(httr)
suppressPackageStartupMessages(library(dplyr))

type <- "issue"
author <- "hadley"
repo <- "rstudio/rstudioapi"
state <- "open"
(search_terms <- lst(type, author, repo, state))
#> $type
#> [1] "issue"
#> 
#> $author
#> [1] "hadley"
#> 
#> $repo
#> [1] "rstudio/rstudioapi"
#> 
#> $state
#> [1] "open"
```

Function to create query string with `sep` between field and value and `collapse` between field-value pairs.

``` r
make_query <- function(terms, sep = ":", collapse = " ") {
  paste(names(terms), terms, sep = sep, collapse = collapse)
}
```

Form query with `sep` set to `+` and \` \`.

``` r
(query_plus <- make_query(search_terms, collapse = "+"))
#> [1] "type:issue+author:hadley+repo:rstudio/rstudioapi+state:open"
(query_space <- make_query(search_terms, collapse = " "))
#> [1] "type:issue author:hadley repo:rstudio/rstudioapi state:open"
```

Use `gh` with both queries.

``` r
## space works
res <- gh("/search/issues", q = query_space, .limit = Inf) 
res$total_count
#> [1] 4
#jsonlite::toJSON(res$items[[1]], pretty = TRUE, auto_unbox = TRUE)

## `+` does not work
res <- gh("/search/issues", q = query_plus, .limit = Inf) 
#> condition in gh("/search/issues", q = query_plus, .limit = Inf): GitHub API error: 422 Unprocessable Entity
#>   Validation Failed
```

Use `httr` with both queries.

``` r
## space works if passed through modify_url
(url <- modify_url("https://api.github.com", path = "search/issues", 
                   query = list(q = query_space)))
#> [1] "https://api.github.com/search/issues?q=type%3Aissue%20author%3Ahadley%20repo%3Arstudio%2Frstudioapi%20state%3Aopen"
x <- GET(url)
status_code(x)
#> [1] 200
length(content(x)$items)
#> [1] 4

## `+` does not work when passed through modify_url
(url <- modify_url("https://api.github.com", path = "search/issues", 
                   query = list(q = query_plus)))
#> [1] "https://api.github.com/search/issues?q=type%3Aissue%2Bauthor%3Ahadley%2Brepo%3Arstudio%2Frstudioapi%2Bstate%3Aopen"
x <- GET(url)
status_code(x)
#> [1] 422

## space used literally does not work, of course
(url <- paste0("https://api.github.com/search/issues?q=", query_space))
#> [1] "https://api.github.com/search/issues?q=type:issue author:hadley repo:rstudio/rstudioapi state:open"
x <- GET(url)
status_code(x)
#> [1] 400

## `+` used literally does work
(url <- paste0("https://api.github.com/search/issues?q=", query_plus))
#> [1] "https://api.github.com/search/issues?q=type:issue+author:hadley+repo:rstudio/rstudioapi+state:open"
x <- GET(url)
status_code(x)
#> [1] 200
length(content(x)$items)
#> [1] 4
```
