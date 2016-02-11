
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
