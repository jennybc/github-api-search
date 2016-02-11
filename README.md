
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

The structure of what's returned is problematic (`total_count`, `incomplete_results`, `items`) and won't play well with `gh`'s approach to de-pagination. But since we get a ridiculously small number of results, we aren't able to see that problem yet here. It almost feels like the search terms mean something different via API vs browser?

What if I try one of the search examples from the API docs? Basically success. We do seem to retrieve the relevant results and we get a demo of how the search results don't play well with `gh`'s pagination.

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
