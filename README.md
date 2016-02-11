
Here's what we're trying to match via API:

<https://github.com/issues?q=is%3Aissue+author%3Atimelyportfolio+is%3Aopen>

I see 194 open issues.

Here's the relevant API endpoint:

<https://developer.github.com/v3/search/#search-issues>

You can't just provide the search terms as params to `gh()`. They need to be pre-processed.

``` r
library(gh)

search_q <- list(author = "timelyportfolio", is = "open")
(search_q <- paste(names(search_q), search_q, sep = "=", collapse = "+"))
```

    ## [1] "author=timelyportfolio+is=open"

``` r
## this causes us to get an empty list !?!
## (search_q <- paste(names(search_q), search_q, sep = ":", collapse = "+"))
## (search_q <- URLencode(search_q, reserved = TRUE))
x <- gh("/search/issues", q = search_q, .limit = Inf)
str(x, max.level = 1)
```

    ## List of 3
    ##  $ total_count       : int 14
    ##  $ incomplete_results: logi FALSE
    ##  $ items             :List of 14
    ##  - attr(*, "method")= chr "GET"
    ##  - attr(*, "response")=List of 24
    ##   ..- attr(*, "class")= chr [1:2] "insensitive" "list"
    ##  - attr(*, ".send_headers")= Named chr [1:2] "application/vnd.github.v3+json" "https://github.com/gaborcsardi/whoami"
    ##   ..- attr(*, "names")= chr [1:2] "Accept" "User-Agent"
    ##  - attr(*, "class")= chr [1:2] "gh_response" "list"

``` r
length(x$items)
```

    ## [1] 14

The actual issues appear in `items`. But I only get 14! A far cry from 194.

The structure of what's returned is problematic (`total_count`, `incomplete_results`, `items`) and won't play well with pagination. But even the first page is "short", i.e. contains only 14 issues and no "next" link. So we don't even get enough results to see that problem. What's up with that?

What if I try their example? Mixed success. We can at least see that we retrieve many pages, though this does not play well with `gh`'s pagination.

``` r
search_q <- list(label = "bug", language = "python", state = "open")
search_q <- c("windows", paste(names(search_q), search_q, sep = "="))
(search_q <- paste(search_q, collapse = "+"))
```

    ## [1] "windows+label=bug+language=python+state=open"

``` r
x <- gh("/search/issues", q = search_q, .limit = Inf)
str(x, max.level = 1)
```

    ## List of 42
    ##  $ total_count       : int 394
    ##  $ incomplete_results: logi FALSE
    ##  $ items             :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 30
    ##  $ NA                : int 394
    ##  $ NA                : logi FALSE
    ##  $ NA                :List of 4
    ##  - attr(*, "method")= chr "GET"
    ##  - attr(*, "response")=List of 25
    ##   ..- attr(*, "class")= chr [1:2] "insensitive" "list"
    ##  - attr(*, ".send_headers")= Named chr [1:2] "application/vnd.github.v3+json" "https://github.com/gaborcsardi/whoami"
    ##   ..- attr(*, "names")= chr [1:2] "Accept" "User-Agent"
    ##  - attr(*, "class")= chr [1:2] "gh_response" "list"

``` r
length(x)
```

    ## [1] 42
