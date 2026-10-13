# cli progress bar benchmark

## Introduction

We make sure that the timer is not `TRUE`, by setting it to ten hours.

``` r

library(cli)
# 10 hours
cli:::cli_tick_set(10 * 60 * 60 * 1000)
cli_tick_reset()
#> NULL
`__cli_update_due`
#> [1] FALSE
```

## R benchmarks

### The timer

``` r

fun <- function() NULL
ben_st <- bench::mark(
  `__cli_update_due`,
  fun(),
  .Call(ccli_tick_reset),
  interactive(),
  check = FALSE
)
ben_st
#> # A tibble: 4 × 6
#>   expression                  min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>             <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 __cli_update_due              0      1ns    1.86e8        0B        0
#> 2 fun()                      80ns   90.1ns    7.11e6        0B        0
#> 3 .Call(ccli_tick_reset)     70ns   80.1ns    1.16e7        0B        0
#> 4 interactive()                 0   10.1ns    9.76e7        0B        0
```

``` r

ben_st2 <- bench::mark(
  if (`__cli_update_due`) foobar()
)
ben_st2
#> # A tibble: 1 × 6
#>   expression                    min median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>                 <bch:> <bch:>     <dbl> <bch:byt>    <dbl>
#> 1 if (`__cli_update_due`) f… 20.1ns 30.2ns 28838561.        0B        0
```

### `cli_progress_along()`

``` r

seq <- 1:100000
ta <- cli_progress_along(seq)
bench::mark(seq[[1]], ta[[1]])
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 seq[[1]]     70.1ns   80.1ns 11099813.        0B        0
#> 2 ta[[1]]        80ns   90.1ns  9624512.        0B        0
```

#### `for` loop

This is the baseline:

``` r

f0 <- function(n = 1e5) {
  x <- 0
  seq <- 1:n
  for (i in seq) {
    x <- x + i %% 2
  }
  x
}
```

With progress bars:

``` r

fp <- function(n = 1e5) {
  x <- 0
  seq <- 1:n
  for (i in cli_progress_along(seq)) {
    x <- x + seq[[i]] %% 2
  }
  x
}
```

Overhead per iteration:

``` r

ben_taf <- bench::mark(f0(), fp())
ben_taf
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0()         14.8ms     15ms      66.4    21.6KB     553.
#> 2 fp()         16.8ms   16.8ms      59.4    82.5KB     455.
(ben_taf$median[2] - ben_taf$median[1]) / 1e5
#> [1] 18.3ns
```

``` r

ben_taf2 <- bench::mark(f0(1e6), fp(1e6))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_taf2
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+06)     165ms    175ms      5.14        0B     44.5
#> 2 fp(1e+06)     165ms    168ms      5.91     1.9KB     27.6
(ben_taf2$median[2] - ben_taf2$median[1]) / 1e6
#> [1] 1ns
```

``` r

ben_taf3 <- bench::mark(f0(1e7), fp(1e7))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_taf3
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+07)     1.68s    1.68s     0.596        0B     28.6
#> 2 fp(1e+07)     1.72s    1.72s     0.582     1.9KB     27.3
(ben_taf3$median[2] - ben_taf3$median[1]) / 1e7
#> [1] 4.08ns
```

``` r

ben_taf4 <- bench::mark(f0(1e8), fp(1e8))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_taf4
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+08)     16.6s    16.6s    0.0604        0B     31.6
#> 2 fp(1e+08)     17.2s    17.2s    0.0582     1.9KB     30.3
(ben_taf4$median[2] - ben_taf4$median[1]) / 1e8
#> [1] 6.3ns
```

#### Mapping with `lapply()`

This is the baseline:

``` r

f0 <- function(n = 1e5) {
  seq <- 1:n
  ret <- lapply(seq, function(x) {
    x %% 2
  })
  invisible(ret)
}
```

With an index vector:

``` r

f01 <- function(n = 1e5) {
  seq <- 1:n
  ret <- lapply(seq_along(seq), function(i) {
    seq[[i]] %% 2
  })
  invisible(ret)
}
```

With progress bars:

``` r

fp <- function(n = 1e5) {
  seq <- 1:n
  ret <- lapply(cli_progress_along(seq), function(i) {
    seq[[i]] %% 2
  })
  invisible(ret)
}
```

Overhead per iteration:

``` r

ben_tam <- bench::mark(f0(), f01(), fp())
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_tam
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0()         55.8ms   61.6ms     11.8      781KB     21.6
#> 2 f01()        68.7ms   73.1ms     11.8      781KB     16.8
#> 3 fp()         82.9ms   94.4ms      9.51     783KB     11.4
(ben_tam$median[3] - ben_tam$median[1]) / 1e5
#> [1] 328ns
```

``` r

ben_tam2 <- bench::mark(f0(1e6), f01(1e6), fp(1e6))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_tam2
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+06)     606ms    606ms      1.65    7.63MB     6.60
#> 2 f01(1e+06)    930ms    930ms      1.08    7.63MB     4.30
#> 3 fp(1e+06)     947ms    947ms      1.06    7.63MB     6.33
(ben_tam2$median[3] - ben_tam2$median[1]) / 1e6
#> [1] 341ns
(ben_tam2$median[3] - ben_tam2$median[2]) / 1e6
#> [1] 17.5ns
```

#### Mapping with purrr

This is the baseline:

``` r

f0 <- function(n = 1e5) {
  seq <- 1:n
  ret <- purrr::map(seq, function(x) {
    x %% 2
  })
  invisible(ret)
}
```

With index vector:

``` r

f01 <- function(n = 1e5) {
  seq <- 1:n
  ret <- purrr::map(seq_along(seq), function(i) {
    seq[[i]] %% 2
  })
  invisible(ret)
}
```

With progress bars:

``` r

fp <- function(n = 1e5) {
  seq <- 1:n
  ret <- purrr::map(cli_progress_along(seq), function(i) {
    seq[[i]] %% 2
  })
  invisible(ret)
}
```

Overhead per iteration:

``` r

ben_pur <- bench::mark(f0(), f01(), fp())
ben_pur
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0()         46.2ms   46.4ms      21.3    1.44MB     12.2
#> 2 f01()        54.1ms     55ms      18.1   781.3KB     18.1
#> 3 fp()         57.9ms   66.1ms      15.4  783.26KB     11.5
(ben_pur$median[3] - ben_pur$median[1]) / 1e5
#> [1] 196ns
(ben_pur$median[3] - ben_pur$median[2]) / 1e5
#> [1] 111ns
```

``` r

ben_pur2 <- bench::mark(f0(1e6), f01(1e6), fp(1e6))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_pur2
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+06)  666.22ms 666.22ms     1.50     7.63MB     4.50
#> 2 f01(1e+06) 817.81ms 817.81ms     1.22     7.63MB     4.89
#> 3 fp(1e+06)     2.19s    2.19s     0.458    7.63MB     1.83
(ben_pur2$median[3] - ben_pur2$median[1]) / 1e6
#> [1] 1.52µs
(ben_pur2$median[3] - ben_pur2$median[2]) / 1e6
#> [1] 1.37µs
```

### `ticking()`

``` r

f0 <- function(n = 1e5) {
  i <- 0
  x <- 0 
  while (i < n) {
    x <- x + i %% 2
    i <- i + 1
  }
  x
}
```

``` r

fp <- function(n = 1e5) {
  i <- 0
  x <- 0 
  while (ticking(i < n)) {
    x <- x + i %% 2
    i <- i + 1
  }
  x
}
```

``` r

ben_tk <- bench::mark(f0(), fp())
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_tk
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0()        15.28ms  15.75ms    62.7      39.3KB     3.92
#> 2 fp()          2.27s    2.27s     0.442   100.8KB     5.30
(ben_tk$median[2] - ben_tk$median[1]) / 1e5
#> [1] 22.5µs
```

### Traditional API

``` r

f0 <- function(n = 1e5) {
  x <- 0
  for (i in 1:n) {
    x <- x + i %% 2
  }
  x
}
```

``` r

fp <- function(n = 1e5) {
  cli_progress_bar(total = n)
  x <- 0
  for (i in 1:n) {
    x <- x + i %% 2
    cli_progress_update()
  }
  x
}
```

``` r

ff <- function(n = 1e5) {
  cli_progress_bar(total = n)
  x <- 0
  for (i in 1:n) {
    x <- x + i %% 2
    if (`__cli_update_due`) cli_progress_update()
  }
  x
}
```

``` r

ben_api <- bench::mark(f0(), ff(), fp())
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_api
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0()        14.14ms  14.49ms    61.5      18.7KB     7.93
#> 2 ff()        18.57ms  18.85ms    48.6      27.6KB     5.83
#> 3 fp()          1.27s    1.27s     0.788    25.1KB     4.73
(ben_api$median[3] - ben_api$median[1]) / 1e5
#> [1] 12.6µs
(ben_api$median[2] - ben_api$median[1]) / 1e5
#> [1] 43.6ns
```

``` r

ben_api2 <- bench::mark(f0(1e6), ff(1e6), fp(1e6))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_api2
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+06)   147.9ms  151.9ms    6.58          0B     8.23
#> 2 ff(1e+06)   188.4ms  189.1ms    5.28      1.91KB     5.28
#> 3 fp(1e+06)     12.5s    12.5s    0.0798    1.91KB     4.95
(ben_api2$median[3] - ben_api2$median[1]) / 1e6
#> [1] 12.4µs
(ben_api2$median[2] - ben_api2$median[1]) / 1e6
#> [1] 37.2ns
```

## C benchmarks

Baseline function:

``` c
SEXP test_baseline() {
  int i;
  int res = 0;
  for (i = 0; i < 2000000000; i++) {
    res += i % 2;
  }
  return ScalarInteger(res);
}
```

Switch + modulo check:

``` c
SEXP test_modulo(SEXP progress) {
  int i;
  int res = 0;
  int progress_ = LOGICAL(progress)[0];
  for (i = 0; i < 2000000000; i++) {
    if (i % 10000 == 0 && progress_) cli_progress_set(R_NilValue, i);
    res += i % 2;
  }
  return ScalarInteger(res);
}
```

cli progress bar API:

``` c
SEXP test_cli() {
  int i;
  int res = 0;
  SEXP bar = PROTECT(cli_progress_bar(2000000000, NULL));
  for (i = 0; i < 2000000000; i++) {
    if (CLI_SHOULD_TICK) cli_progress_set(bar, i);
    res += i % 2;
  }
  cli_progress_done(bar);
  UNPROTECT(1);
  return ScalarInteger(res);
}
```

``` c
SEXP test_cli_unroll() {
  int i = 0;
  int res = 0;
  SEXP bar = PROTECT(cli_progress_bar(2000000000, NULL));
  int s, final, step = 2000000000 / 100000;
  for (s = 0; s < 100000; s++) {
    if (CLI_SHOULD_TICK) cli_progress_set(bar, i);
    final = (s + 1) * step;
    for (i = s * step; i < final; i++) {
      res += i % 2;
    }
  }
  cli_progress_done(bar);
  UNPROTECT(1);
  return ScalarInteger(res);
}
```

``` r

library(progresstest)
ben_c <- bench::mark(
  test_baseline(),
  test_modulo(),
  test_cli(),
  test_cli_unroll()
)
ben_c
#> # A tibble: 4 × 6
#>   expression             min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>        <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 test_baseline()      449ms    454ms      2.20    2.08KB        0
#> 2 test_modulo()        754ms    754ms      1.33    2.24KB        0
#> 3 test_cli()           491ms    492ms      2.03   24.11KB        0
#> 4 test_cli_unroll()    454ms    456ms      2.19    3.58KB        0
(ben_c$median[3] - ben_c$median[1]) / 2000000000
#> [1] 1ns
```

## Display update

We only update the display a fixed number of times per second.
(Currently maximum five times per second.)

Let’s measure how long a single update takes.

### Iterator with a bar

``` r

cli_progress_bar(total = 100000)
bench::mark(cli_progress_update(force = TRUE), max_iterations = 10000)
#> ■                                  0% | ETA:  3m
#> ■                                  0% | ETA:  1h
#> ■                                  0% | ETA: 47m
#> ■                                  0% | ETA: 37m
#> ■                                  0% | ETA: 31m
#> ■                                  0% | ETA: 26m
#> ■                                  0% | ETA: 23m
#> ■                                  0% | ETA: 21m
#> ■                                  0% | ETA: 19m
#> ■                                  0% | ETA: 18m
#> ■                                  0% | ETA: 17m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 13m
#> ■                                  0% | ETA: 13m
#> ■                                  0% | ETA: 12m
#> ■                                  0% | ETA: 12m
#> ■                                  0% | ETA: 12m
#> ■                                  0% | ETA: 11m
#> ■                                  0% | ETA: 11m
#> ■                                  0% | ETA: 11m
#> ■                                  0% | ETA: 11m
#> ■                                  0% | ETA: 10m
#> ■                                  0% | ETA: 10m
#> ■                                  0% | ETA: 10m
#> ■                                  0% | ETA: 10m
#> ■                                  0% | ETA: 10m
#> ■                                  0% | ETA: 10m
#> ■                                  0% | ETA:  9m
#> ■                                  0% | ETA:  9m
#> ■                                  0% | ETA:  9m
#> ■                                  0% | ETA:  9m
#> ■                                  0% | ETA:  9m
#> ■                                  0% | ETA:  9m
#> ■                                  0% | ETA:  9m
#> ■                                  0% | ETA:  9m
#> ■                                  0% | ETA:  9m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  8m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  7m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> ■                                  0% | ETA:  6m
#> # A tibble: 1 × 6
#>   expression                    min median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>                 <bch:> <bch:>     <dbl> <bch:byt>    <dbl>
#> 1 cli_progress_update(force… 3.09ms  3.2ms      301.    1.42MB     4.10
cli_progress_done()
```

### Iterator without a bar

``` r

cli_progress_bar(total = NA)
bench::mark(cli_progress_update(force = TRUE), max_iterations = 10000)
#> ⠙ 1 done (740/s) | 2ms
#> ⠹ 2 done (126/s) | 16ms
#> ⠸ 3 done (151/s) | 20ms
#> ⠼ 4 done (169/s) | 24ms
#> ⠴ 5 done (182/s) | 28ms
#> ⠦ 6 done (192/s) | 32ms
#> ⠧ 7 done (200/s) | 36ms
#> ⠇ 8 done (207/s) | 39ms
#> ⠏ 9 done (212/s) | 43ms
#> ⠋ 10 done (216/s) | 47ms
#> ⠙ 11 done (220/s) | 50ms
#> ⠹ 12 done (224/s) | 54ms
#> ⠸ 13 done (227/s) | 58ms
#> ⠼ 14 done (229/s) | 62ms
#> ⠴ 15 done (231/s) | 65ms
#> ⠦ 16 done (233/s) | 69ms
#> ⠧ 17 done (235/s) | 73ms
#> ⠇ 18 done (237/s) | 77ms
#> ⠏ 19 done (238/s) | 80ms
#> ⠋ 20 done (240/s) | 84ms
#> ⠙ 21 done (241/s) | 88ms
#> ⠹ 22 done (242/s) | 91ms
#> ⠸ 23 done (243/s) | 95ms
#> ⠼ 24 done (244/s) | 99ms
#> ⠴ 25 done (245/s) | 103ms
#> ⠦ 26 done (246/s) | 106ms
#> ⠧ 27 done (247/s) | 110ms
#> ⠇ 28 done (248/s) | 114ms
#> ⠏ 29 done (241/s) | 121ms
#> ⠋ 30 done (240/s) | 126ms
#> ⠙ 31 done (239/s) | 130ms
#> ⠹ 32 done (238/s) | 135ms
#> ⠸ 33 done (237/s) | 140ms
#> ⠼ 34 done (236/s) | 145ms
#> ⠴ 35 done (237/s) | 148ms
#> ⠦ 36 done (238/s) | 152ms
#> ⠧ 37 done (238/s) | 156ms
#> ⠇ 38 done (239/s) | 159ms
#> ⠏ 39 done (240/s) | 163ms
#> ⠋ 40 done (240/s) | 167ms
#> ⠙ 41 done (241/s) | 171ms
#> ⠹ 42 done (241/s) | 174ms
#> ⠸ 43 done (242/s) | 178ms
#> ⠼ 44 done (243/s) | 182ms
#> ⠴ 45 done (242/s) | 186ms
#> ⠦ 46 done (243/s) | 190ms
#> ⠧ 47 done (243/s) | 194ms
#> ⠇ 48 done (244/s) | 198ms
#> ⠏ 49 done (244/s) | 201ms
#> ⠋ 50 done (244/s) | 205ms
#> ⠙ 51 done (245/s) | 209ms
#> ⠹ 52 done (245/s) | 213ms
#> ⠸ 53 done (246/s) | 216ms
#> ⠼ 54 done (246/s) | 220ms
#> ⠴ 55 done (246/s) | 224ms
#> ⠦ 56 done (246/s) | 228ms
#> ⠧ 57 done (246/s) | 232ms
#> ⠇ 58 done (247/s) | 236ms
#> ⠏ 59 done (247/s) | 240ms
#> ⠋ 60 done (247/s) | 243ms
#> ⠙ 61 done (247/s) | 247ms
#> ⠹ 62 done (247/s) | 251ms
#> ⠸ 63 done (248/s) | 255ms
#> ⠼ 64 done (248/s) | 259ms
#> ⠴ 65 done (248/s) | 263ms
#> ⠦ 66 done (248/s) | 267ms
#> ⠧ 67 done (248/s) | 271ms
#> ⠇ 68 done (248/s) | 274ms
#> ⠏ 69 done (248/s) | 278ms
#> ⠋ 70 done (248/s) | 282ms
#> ⠙ 71 done (249/s) | 286ms
#> ⠹ 72 done (249/s) | 290ms
#> ⠸ 73 done (249/s) | 294ms
#> ⠼ 74 done (249/s) | 298ms
#> ⠴ 75 done (249/s) | 302ms
#> ⠦ 76 done (249/s) | 306ms
#> ⠧ 77 done (249/s) | 309ms
#> ⠇ 78 done (250/s) | 313ms
#> ⠏ 79 done (250/s) | 317ms
#> ⠋ 80 done (250/s) | 321ms
#> ⠙ 81 done (250/s) | 325ms
#> ⠹ 82 done (250/s) | 328ms
#> ⠸ 83 done (250/s) | 332ms
#> ⠼ 84 done (250/s) | 336ms
#> ⠴ 85 done (245/s) | 347ms
#> ⠦ 86 done (245/s) | 352ms
#> ⠧ 87 done (244/s) | 357ms
#> ⠇ 88 done (244/s) | 362ms
#> ⠏ 89 done (243/s) | 367ms
#> ⠋ 90 done (243/s) | 371ms
#> ⠙ 91 done (243/s) | 375ms
#> ⠹ 92 done (243/s) | 378ms
#> ⠸ 93 done (244/s) | 382ms
#> ⠼ 94 done (244/s) | 386ms
#> ⠴ 95 done (244/s) | 390ms
#> ⠦ 96 done (244/s) | 393ms
#> ⠧ 97 done (245/s) | 397ms
#> ⠇ 98 done (245/s) | 401ms
#> ⠏ 99 done (245/s) | 405ms
#> ⠋ 100 done (245/s) | 408ms
#> ⠙ 101 done (245/s) | 412ms
#> ⠹ 102 done (245/s) | 416ms
#> ⠸ 103 done (245/s) | 420ms
#> ⠼ 104 done (246/s) | 424ms
#> ⠴ 105 done (246/s) | 428ms
#> ⠦ 106 done (246/s) | 432ms
#> ⠧ 107 done (246/s) | 436ms
#> ⠇ 108 done (246/s) | 439ms
#> ⠏ 109 done (246/s) | 443ms
#> ⠋ 110 done (246/s) | 447ms
#> ⠙ 111 done (246/s) | 451ms
#> ⠹ 112 done (246/s) | 455ms
#> ⠸ 113 done (246/s) | 459ms
#> ⠼ 114 done (246/s) | 463ms
#> ⠴ 115 done (247/s) | 467ms
#> ⠦ 116 done (247/s) | 471ms
#> ⠧ 117 done (247/s) | 475ms
#> ⠇ 118 done (247/s) | 479ms
#> ⠏ 119 done (247/s) | 483ms
#> ⠋ 120 done (247/s) | 487ms
#> ⠙ 121 done (247/s) | 491ms
#> ⠹ 122 done (247/s) | 494ms
#> ⠸ 123 done (247/s) | 498ms
#> ⠼ 124 done (247/s) | 502ms
#> ⠴ 125 done (247/s) | 506ms
#> ⠦ 126 done (247/s) | 510ms
#> ⠧ 127 done (248/s) | 514ms
#> # A tibble: 1 × 6
#>   expression                    min median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>                 <bch:> <bch:>     <dbl> <bch:byt>    <dbl>
#> 1 cli_progress_update(force… 3.63ms 3.83ms      257.     266KB     4.15
cli_progress_done()
```
