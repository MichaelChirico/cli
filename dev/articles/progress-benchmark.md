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
#> 1 __cli_update_due              0   10.1ns 88112241.        0B        0
#> 2 fun()                  130.04ns  160.1ns  4535275.        0B        0
#> 3 .Call(ccli_tick_reset) 110.01ns    130ns  7425193.        0B        0
#> 4 interactive()            8.96ns   11.1ns 58989592.        0B        0
```

``` r

ben_st2 <- bench::mark(
  if (`__cli_update_due`) foobar()
)
ben_st2
#> # A tibble: 1 × 6
#>   expression                    min median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>                  <bch> <bch:>     <dbl> <bch:byt>    <dbl>
#> 1 if (`__cli_update_due`) fo…  40ns 50.1ns 20211449.        0B        0
```

### `cli_progress_along()`

``` r

seq <- 1:100000
ta <- cli_progress_along(seq)
bench::mark(seq[[1]], ta[[1]])
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 seq[[1]]      130ns    150ns  6363301.        0B        0
#> 2 ta[[1]]       140ns    160ns  5655147.        0B        0
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
#> 1 f0()         22.1ms   22.2ms      45.1    21.6KB     406.
#> 2 fp()         24.7ms     25ms      40.0    82.5KB     340.
(ben_taf$median[2] - ben_taf$median[1]) / 1e5
#> [1] 28.4ns
```

``` r

ben_taf2 <- bench::mark(f0(1e6), fp(1e6))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_taf2
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+06)     249ms    250ms      3.99        0B     35.9
#> 2 fp(1e+06)     265ms    267ms      3.75     1.9KB     31.8
(ben_taf2$median[2] - ben_taf2$median[1]) / 1e6
#> [1] 16.6ns
```

``` r

ben_taf3 <- bench::mark(f0(1e7), fp(1e7))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_taf3
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+07)     2.44s    2.44s     0.409        0B     19.2
#> 2 fp(1e+07)     2.54s    2.54s     0.394     1.9KB     18.5
(ben_taf3$median[2] - ben_taf3$median[1]) / 1e7
#> [1] 9.19ns
```

``` r

ben_taf4 <- bench::mark(f0(1e8), fp(1e8))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_taf4
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+08)     23.8s    23.8s    0.0420        0B     22.1
#> 2 fp(1e+08)     25.5s    25.5s    0.0392     1.9KB     20.3
(ben_taf4$median[2] - ben_taf4$median[1]) / 1e8
#> [1] 17ns
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
#> 1 f0()         93.9ms    108ms      7.63     781KB    16.8 
#> 2 f01()         105ms    109ms      9.24     781KB     9.24
#> 3 fp()        127.2ms    133ms      6.67     783KB    11.7
(ben_tam$median[3] - ben_tam$median[1]) / 1e5
#> [1] 247ns
```

``` r

ben_tam2 <- bench::mark(f0(1e6), f01(1e6), fp(1e6))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_tam2
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+06)     1.08s    1.08s     0.924    7.63MB     6.47
#> 2 f01(1e+06)    1.92s    1.92s     0.522    7.63MB     4.17
#> 3 fp(1e+06)     2.48s    2.48s     0.404    7.63MB     2.42
(ben_tam2$median[3] - ben_tam2$median[1]) / 1e6
#> [1] 1.39µs
(ben_tam2$median[3] - ben_tam2$median[2]) / 1e6
#> [1] 558ns
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
#> 1 f0()         78.3ms   78.9ms      12.6    1.44MB     5.04
#> 2 f01()        93.3ms   93.9ms      10.6   781.3KB     2.12
#> 3 fp()         95.9ms     96ms      10.4  783.26KB     6.94
(ben_pur$median[3] - ben_pur$median[1]) / 1e5
#> [1] 171ns
(ben_pur$median[3] - ben_pur$median[2]) / 1e5
#> [1] 21.5ns
```

``` r

ben_pur2 <- bench::mark(f0(1e6), f01(1e6), fp(1e6))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_pur2
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+06)  892.09ms 892.09ms     1.12     7.63MB     2.24
#> 2 f01(1e+06)     1.1s     1.1s     0.908    7.63MB     1.82
#> 3 fp(1e+06)     1.16s    1.16s     0.865    7.63MB     1.73
(ben_pur2$median[3] - ben_pur2$median[1]) / 1e6
#> [1] 264ns
(ben_pur2$median[3] - ben_pur2$median[2]) / 1e6
#> [1] 53.7ns
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
#> 1 f0()        22.76ms  22.85ms    42.6      39.3KB     1.94
#> 2 fp()          4.04s    4.04s     0.248   100.8KB     1.98
(ben_tk$median[2] - ben_tk$median[1]) / 1e5
#> [1] 40.1µs
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
#> 1 f0()        21.52ms  21.75ms    43.0      18.7KB     1.95
#> 2 ff()        32.14ms  32.25ms    29.7      27.6KB     3.96
#> 3 fp()          2.24s    2.24s     0.446    25.1KB     1.78
(ben_api$median[3] - ben_api$median[1]) / 1e5
#> [1] 22.2µs
(ben_api$median[2] - ben_api$median[1]) / 1e5
#> [1] 105ns
```

``` r

ben_api2 <- bench::mark(f0(1e6), ff(1e6), fp(1e6))
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
ben_api2
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 f0(1e+06)   219.8ms  222.4ms    4.44          0B     2.96
#> 2 ff(1e+06)   327.1ms    329ms    3.04      1.91KB     1.52
#> 3 fp(1e+06)     22.8s    22.8s    0.0439    1.91KB     1.76
(ben_api2$median[3] - ben_api2$median[1]) / 1e6
#> [1] 22.6µs
(ben_api2$median[2] - ben_api2$median[1]) / 1e6
#> [1] 107ns
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
#> 1 test_baseline()   622.62ms 622.62ms     1.61     2.08KB        0
#> 2 test_modulo()        1.25s    1.25s     0.802    2.24KB        0
#> 3 test_cli()           1.25s    1.25s     0.803   24.11KB        0
#> 4 test_cli_unroll() 623.95ms 623.95ms     1.60     3.58KB        0
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
#> ■                                  0% | ETA:  4m
#> ■                                  0% | ETA:  2h
#> ■                                  0% | ETA:  1h
#> ■                                  0% | ETA:  1h
#> ■                                  0% | ETA:  1h
#> ■                                  0% | ETA: 46m
#> ■                                  0% | ETA: 41m
#> ■                                  0% | ETA: 37m
#> ■                                  0% | ETA: 35m
#> ■                                  0% | ETA: 32m
#> ■                                  0% | ETA: 30m
#> ■                                  0% | ETA: 29m
#> ■                                  0% | ETA: 27m
#> ■                                  0% | ETA: 26m
#> ■                                  0% | ETA: 25m
#> ■                                  0% | ETA: 24m
#> ■                                  0% | ETA: 23m
#> ■                                  0% | ETA: 23m
#> ■                                  0% | ETA: 22m
#> ■                                  0% | ETA: 22m
#> ■                                  0% | ETA: 21m
#> ■                                  0% | ETA: 21m
#> ■                                  0% | ETA: 20m
#> ■                                  0% | ETA: 20m
#> ■                                  0% | ETA: 19m
#> ■                                  0% | ETA: 19m
#> ■                                  0% | ETA: 19m
#> ■                                  0% | ETA: 18m
#> ■                                  0% | ETA: 18m
#> ■                                  0% | ETA: 18m
#> ■                                  0% | ETA: 18m
#> ■                                  0% | ETA: 18m
#> ■                                  0% | ETA: 17m
#> ■                                  0% | ETA: 17m
#> ■                                  0% | ETA: 17m
#> ■                                  0% | ETA: 17m
#> ■                                  0% | ETA: 17m
#> ■                                  0% | ETA: 17m
#> ■                                  0% | ETA: 17m
#> ■                                  0% | ETA: 17m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 16m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 15m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 14m
#> ■                                  0% | ETA: 14m
#> # A tibble: 1 × 6
#>   expression                    min median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>                 <bch:> <bch:>     <dbl> <bch:byt>    <dbl>
#> 1 cli_progress_update(force… 6.45ms 6.54ms      148.    1.42MB     2.03
cli_progress_done()
```

### Iterator without a bar

``` r

cli_progress_bar(total = NA)
bench::mark(cli_progress_update(force = TRUE), max_iterations = 10000)
#> ⠙ 1 done (481/s) | 3ms
#> ⠹ 2 done (68/s) | 30ms
#> ⠸ 3 done (81/s) | 38ms
#> ⠼ 4 done (89/s) | 45ms
#> ⠴ 5 done (95/s) | 53ms
#> ⠦ 6 done (100/s) | 61ms
#> ⠧ 7 done (104/s) | 68ms
#> ⠇ 8 done (106/s) | 76ms
#> ⠏ 9 done (109/s) | 84ms
#> ⠋ 10 done (110/s) | 91ms
#> ⠙ 11 done (112/s) | 99ms
#> ⠹ 12 done (113/s) | 106ms
#> ⠸ 13 done (115/s) | 114ms
#> ⠼ 14 done (116/s) | 122ms
#> ⠴ 15 done (117/s) | 129ms
#> ⠦ 16 done (117/s) | 137ms
#> ⠧ 17 done (118/s) | 144ms
#> ⠇ 18 done (119/s) | 152ms
#> ⠏ 19 done (120/s) | 160ms
#> ⠋ 20 done (120/s) | 167ms
#> ⠙ 21 done (121/s) | 175ms
#> ⠹ 22 done (121/s) | 182ms
#> ⠸ 23 done (122/s) | 190ms
#> ⠼ 24 done (122/s) | 198ms
#> ⠴ 25 done (122/s) | 205ms
#> ⠦ 26 done (123/s) | 213ms
#> ⠧ 27 done (123/s) | 220ms
#> ⠇ 28 done (123/s) | 228ms
#> ⠏ 29 done (123/s) | 236ms
#> ⠋ 30 done (124/s) | 243ms
#> ⠙ 31 done (124/s) | 251ms
#> ⠹ 32 done (124/s) | 258ms
#> ⠸ 33 done (124/s) | 266ms
#> ⠼ 34 done (125/s) | 274ms
#> ⠴ 35 done (125/s) | 281ms
#> ⠦ 36 done (125/s) | 289ms
#> ⠧ 37 done (125/s) | 296ms
#> ⠇ 38 done (125/s) | 304ms
#> ⠏ 39 done (125/s) | 312ms
#> ⠋ 40 done (126/s) | 319ms
#> ⠙ 41 done (126/s) | 327ms
#> ⠹ 42 done (126/s) | 335ms
#> ⠸ 43 done (126/s) | 342ms
#> ⠼ 44 done (126/s) | 350ms
#> ⠴ 45 done (126/s) | 357ms
#> ⠦ 46 done (126/s) | 365ms
#> ⠧ 47 done (126/s) | 373ms
#> ⠇ 48 done (126/s) | 380ms
#> ⠏ 49 done (127/s) | 388ms
#> ⠋ 50 done (125/s) | 400ms
#> ⠙ 51 done (125/s) | 408ms
#> ⠹ 52 done (125/s) | 416ms
#> ⠸ 53 done (125/s) | 425ms
#> ⠼ 54 done (125/s) | 433ms
#> ⠴ 55 done (125/s) | 442ms
#> ⠦ 56 done (125/s) | 450ms
#> ⠧ 57 done (125/s) | 458ms
#> ⠇ 58 done (124/s) | 467ms
#> ⠏ 59 done (124/s) | 475ms
#> ⠋ 60 done (124/s) | 483ms
#> ⠙ 61 done (125/s) | 491ms
#> ⠹ 62 done (125/s) | 498ms
#> ⠸ 63 done (125/s) | 506ms
#> ⠼ 64 done (125/s) | 513ms
#> ⠴ 65 done (125/s) | 521ms
#> ⠦ 66 done (125/s) | 528ms
#> # A tibble: 1 × 6
#>   expression                    min median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>                 <bch:> <bch:>     <dbl> <bch:byt>    <dbl>
#> 1 cli_progress_update(force… 7.51ms 7.59ms      130.     265KB     2.02
cli_progress_done()
```
