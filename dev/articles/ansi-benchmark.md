# ANSI function benchmarks

\$output function (x, options) { if (class == “output” && output_asis(x,
options)) return(x) hook.t(x, options\[\[paste0(“attr.”, class)\]\],
options\[\[paste0(“class.”, class)\]\]) } \<bytecode: 0x55c6a70d5dc0\>
\<environment: 0x55c6a7c019c0\>

## Introduction

Often we can use the corresponding base R function as a baseline. We
also compare to the fansi package, where it is possible.

## Data

In cli the typical use case is short string scalars, but we run some
benchmarks longer strings and string vectors as well.

``` r

library(cli)
library(fansi)
options(cli.unicode = TRUE)
options(cli.num_colors = 256)
```

``` r

ansi <- format_inline(
  "{col_green(symbol$tick)} {.code print(x)} {.emph emphasised}"
)
```

``` r

plain <- ansi_strip(ansi)
```

``` r

vec_plain <- rep(plain, 100)
vec_ansi <- rep(ansi, 100)
vec_plain6 <- rep(plain, 6)
vec_ansi6 <- rep(plain, 6)
```

``` r

txt_plain <- paste(vec_plain, collapse = " ")
txt_ansi <- paste(vec_ansi, collapse = " ")
```

``` r

uni <- paste(
  "\U0001f477\u200d\u2640\ufe0f",
  "\U0001f477\U0001f3fb",
  "\U0001f477\u200d\u2640\ufe0f",
  "\U0001f477\U0001f3fb",
  "\U0001f477\U0001f3ff\u200d\u2640\ufe0f"
)
vec_uni <- rep(uni, 100)
txt_uni <- paste(vec_uni, collapse = " ")
```

## ANSI functions

### `ansi_align()`

``` r

bench::mark(
  ansi  = ansi_align(ansi, width = 20),
  plain = ansi_align(plain, width = 20), 
  base  = format(plain, width = 20),
  check = FALSE
)
```

``` fansi
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 ansi        20.91µs   23.7µs    41397.    99.6KB     41.4
#> 2 plain       21.04µs   23.8µs    36160.        0B     39.8
#> 3 base         5.92µs    6.9µs   141170.    48.6KB     28.2
```

``` r

bench::mark(
  ansi  = ansi_align(ansi, width = 20, align = "right"),
  plain = ansi_align(plain, width = 20, align = "right"), 
  base  = format(plain, width = 20, justify = "right"),
  check = FALSE
)
```

``` fansi
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 ansi        21.07µs  23.93µs    37938.        0B     45.6
#> 2 plain       21.22µs  23.96µs    41106.        0B     49.4
#> 3 base         6.93µs   8.02µs   121334.        0B     48.6
```

### `ansi_chartr()`

``` r

bench::mark(
  ansi  = ansi_chartr("abc", "XYZ", ansi),
  plain = ansi_chartr("abc", "XYZ", plain),
  base  = chartr("abc", "XYZ", plain),
  check = FALSE
)
```

``` fansi
#> # A tibble: 3 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 ansi        52.12µs  57.61µs    16998.   77.03KB     34.1
#> 2 plain        41.3µs   44.9µs    21796.    8.91KB     32.7
#> 3 base         1.24µs   1.29µs   730124.        0B      0
```

### `ansi_columns()`

``` r

bench::mark(
  ansi  = ansi_columns(vec_ansi6, width = 120),
  plain = ansi_columns(vec_plain6, width = 120),
  check = FALSE
)
```

``` fansi
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 ansi          142µs    156µs     6193.   33.24KB     34.1
#> 2 plain         142µs    154µs     6406.    1.09KB     24.9
```

### `ansi_has_any()`

``` r

bench::mark(
  cli_ansi        = ansi_has_any(ansi),
  fansi_ansi      = has_sgr(ansi),
  cli_plain       = ansi_has_any(plain),
  fansi_plain     = has_sgr(plain),
  cli_vec_ansi    = ansi_has_any(vec_ansi),
  fansi_vec_ansi  = has_sgr(vec_ansi),
  cli_vec_plain   = ansi_has_any(vec_plain),
  fansi_vec_plain = has_sgr(vec_plain),
  cli_txt_ansi    = ansi_has_any(txt_ansi),
  fansi_txt_ansi  = has_sgr(txt_ansi),
  cli_txt_plain   = ansi_has_any(txt_plain),
  fansi_txt_plain = has_sgr(vec_plain),
  check = FALSE
)
```

``` fansi
#> # A tibble: 12 × 6
#>    expression           min   median `itr/sec` mem_alloc `gc/sec`
#>    <bch:expr>      <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#>  1 cli_ansi          3.25µs   3.81µs   252724.    9.27KB     25.3
#>  2 fansi_ansi       15.31µs  17.14µs    54934.    4.18KB     22.0
#>  3 cli_plain         3.17µs   3.66µs   265778.        0B     26.6
#>  4 fansi_plain       15.4µs  17.32µs    56785.      688B     22.7
#>  5 cli_vec_ansi      4.14µs   4.62µs   211825.      448B     21.2
#>  6 fansi_vec_ansi   20.37µs  23.02µs    42474.    5.02KB     21.2
#>  7 cli_vec_plain     4.22µs   4.93µs   200152.      448B     20.0
#>  8 fansi_vec_plain  20.06µs  22.36µs    43859.    5.02KB     17.6
#>  9 cli_txt_ansi      3.17µs   3.65µs   266927.        0B     26.7
#> 10 fansi_txt_ansi   15.54µs  17.65µs    55625.      688B     22.3
#> 11 cli_txt_plain     3.83µs   4.31µs   227742.        0B     22.8
#> 12 fansi_txt_plain  19.75µs   22.2µs    43996.    5.02KB     22.0
```

### `ansi_html()`

This is typically used with longer text.

``` r

bench::mark(
  cli   = ansi_html(txt_ansi),
  fansi = sgr_to_html(txt_ansi, classes = TRUE),
  check = FALSE
)
```

``` fansi
#> # A tibble: 2 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 cli          31.2µs   32.6µs    29660.    22.7KB     8.90
#> 2 fansi        59.6µs   63.9µs    13773.    55.3KB     6.16
```

### `ansi_nchar()`

``` r

bench::mark(
  cli_ansi        = ansi_nchar(ansi),
  fansi_ansi      = nchar_sgr(ansi),
  base_ansi       = nchar(ansi),
  cli_plain       = ansi_nchar(plain),
  fansi_plain     = nchar_sgr(plain),
  base_plain      = nchar(plain),
  cli_vec_ansi    = ansi_nchar(vec_ansi),
  fansi_vec_ansi  = nchar_sgr(vec_ansi),
  base_vec_ansi   = nchar(vec_ansi),
  cli_vec_plain   = ansi_nchar(vec_plain),
  fansi_vec_plain = nchar_sgr(vec_plain),
  base_vec_plain  = nchar(vec_plain),
  cli_txt_ansi    = ansi_nchar(txt_ansi),
  fansi_txt_ansi  = nchar_sgr(txt_ansi),
  base_txt_ansi   = nchar(txt_ansi),
  cli_txt_plain   = ansi_nchar(txt_plain),
  fansi_txt_plain = nchar_sgr(txt_plain),
  base_txt_plain  = nchar(txt_plain),
  check = FALSE
)
```

``` fansi
#> # A tibble: 18 × 6
#>    expression           min   median `itr/sec` mem_alloc `gc/sec`
#>    <bch:expr>      <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#>  1 cli_ansi          3.66µs   4.27µs   224908.        0B    22.5 
#>  2 fansi_ansi       38.27µs  42.76µs    22969.   38.84KB    18.4 
#>  3 base_ansi       521.08ns 591.04ns  1441593.        0B   144.  
#>  4 cli_plain         3.58µs   4.17µs   234113.        0B     0   
#>  5 fansi_plain       37.8µs  41.36µs    23129.      688B    20.8 
#>  6 base_plain      510.02ns 559.03ns  1651783.        0B     0   
#>  7 cli_vec_ansi     14.92µs  16.27µs    60305.      448B     6.03
#>  8 fansi_vec_ansi   48.65µs  52.57µs    18636.    5.02KB    16.8 
#>  9 base_vec_ansi     8.91µs   11.5µs    89716.      448B     0   
#> 10 cli_vec_plain     14.4µs  15.28µs    55072.      448B     5.51
#> 11 fansi_vec_plain  44.22µs  48.02µs    20453.    5.02KB    19.0 
#> 12 base_vec_plain    5.07µs   5.98µs   169254.      448B     0   
#> 13 cli_txt_ansi     14.89µs   15.7µs    62812.        0B     6.28
#> 14 fansi_txt_ansi   45.52µs  48.51µs    19989.      688B    16.6 
#> 15 base_txt_ansi     8.64µs  10.76µs    92922.        0B     0   
#> 16 cli_txt_plain    13.97µs  14.94µs    66073.        0B     6.61
#> 17 fansi_txt_plain   39.1µs  42.81µs    22904.      688B    20.6 
#> 18 base_txt_plain    4.87µs   5.88µs   171441.        0B     0
```

``` r

bench::mark(
  cli_ansi        = ansi_nchar(ansi, type = "width"),
  fansi_ansi      = nchar_sgr(ansi, type = "width"),
  base_ansi       = nchar(ansi, "width"),
  cli_plain       = ansi_nchar(plain, type = "width"),
  fansi_plain     = nchar_sgr(plain, type = "width"),
  base_plain      = nchar(plain, "width"),
  cli_vec_ansi    = ansi_nchar(vec_ansi, type = "width"),
  fansi_vec_ansi  = nchar_sgr(vec_ansi, type = "width"),
  base_vec_ansi   = nchar(vec_ansi, "width"),
  cli_vec_plain   = ansi_nchar(vec_plain, type = "width"),
  fansi_vec_plain = nchar_sgr(vec_plain, type = "width"),
  base_vec_plain  = nchar(vec_plain, "width"),
  cli_txt_ansi    = ansi_nchar(txt_ansi, type = "width"),
  fansi_txt_ansi  = nchar_sgr(txt_ansi, type = "width"),
  base_txt_ansi   = nchar(txt_ansi, "width"),
  cli_txt_plain   = ansi_nchar(txt_plain, type = "width"),
  fansi_txt_plain = nchar_sgr(txt_plain, type = "width"),
  base_txt_plain  = nchar(txt_plain, type = "width"),
  check = FALSE
)
```

``` fansi
#> # A tibble: 18 × 6
#>    expression           min   median `itr/sec` mem_alloc `gc/sec`
#>    <bch:expr>      <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#>  1 cli_ansi          4.29µs   5.16µs   155560.        0B    15.6 
#>  2 fansi_ansi       38.08µs  41.15µs    23836.      688B    21.5 
#>  3 base_ansi        770.9ns 861.01ns  1102582.        0B     0   
#>  4 cli_plain         4.33µs   5.09µs   192191.        0B    19.2 
#>  5 fansi_plain      37.48µs  40.76µs    24038.      688B    22.2 
#>  6 base_plain      631.09ns 711.18ns  1333284.        0B     0   
#>  7 cli_vec_ansi     17.62µs  18.87µs    52652.      448B     5.27
#>  8 fansi_vec_ansi    49.8µs  53.37µs    17946.    5.02KB    16.8 
#>  9 base_vec_ansi    26.09µs  28.88µs    34436.      448B     0   
#> 10 cli_vec_plain    17.94µs  18.87µs    52476.      448B     5.25
#> 11 fansi_vec_plain   44.6µs  48.21µs    19994.    5.02KB    17.0 
#> 12 base_vec_plain   14.36µs  14.69µs    67397.      448B     0   
#> 13 cli_txt_ansi     17.77µs  18.54µs    53140.        0B    10.6 
#> 14 fansi_txt_ansi   47.11µs  50.71µs    19336.      688B    16.6 
#> 15 base_txt_ansi    28.57µs  30.69µs    32316.        0B     0   
#> 16 cli_txt_plain    17.51µs  18.28µs    53991.        0B     5.40
#> 17 fansi_txt_plain  40.86µs  44.52µs    22098.      688B    19.9 
#> 18 base_txt_plain   15.99µs  16.51µs    59855.        0B     0
```

### `ansi_simplify()`

Nothing to compare here.

``` r

bench::mark(
  cli_ansi      = ansi_simplify(ansi),
  cli_plain     = ansi_simplify(plain),
  cli_vec_ansi  = ansi_simplify(vec_ansi),
  cli_vec_plain = ansi_simplify(vec_plain),
  cli_txt_ansi  = ansi_simplify(txt_ansi),
  cli_txt_plain = ansi_simplify(txt_plain),
  check = FALSE
)
```

``` fansi
#> # A tibble: 6 × 6
#>   expression         min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>    <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 cli_ansi        3.56µs   4.11µs   237066.        0B    23.7 
#> 2 cli_plain       3.35µs   3.85µs   254680.        0B    25.5 
#> 3 cli_vec_ansi   17.17µs  18.28µs    54234.      848B     5.42
#> 4 cli_vec_plain   5.51µs   6.09µs   160908.      848B    16.1 
#> 5 cli_txt_ansi   16.39µs  17.52µs    54262.        0B     0   
#> 6 cli_txt_plain   3.92µs   4.47µs   217635.        0B    21.8
```

### `ansi_strip()`

``` r

bench::mark(
  cli_ansi        = ansi_strip(ansi),
  fansi_ansi      = strip_sgr(ansi),
  cli_plain       = ansi_strip(plain),
  fansi_plain     = strip_sgr(plain),
  cli_vec_ansi    = ansi_strip(vec_ansi),
  fansi_vec_ansi  = strip_sgr(vec_ansi),
  cli_vec_plain   = ansi_strip(vec_plain),
  fansi_vec_plain = strip_sgr(vec_plain),
  cli_txt_ansi    = ansi_strip(txt_ansi),
  fansi_txt_ansi  = strip_sgr(txt_ansi),
  cli_txt_plain   = ansi_strip(txt_plain),
  fansi_txt_plain = strip_sgr(txt_plain),
  check = FALSE
)
```

``` fansi
#> # A tibble: 12 × 6
#>    expression           min   median `itr/sec` mem_alloc `gc/sec`
#>    <bch:expr>      <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#>  1 cli_ansi          13.8µs   15.4µs    63584.        0B     25.4
#>  2 fansi_ansi        14.2µs   15.9µs    60734.    7.24KB     24.3
#>  3 cli_plain         13.8µs   15.4µs    62972.        0B     25.2
#>  4 fansi_plain         14µs   15.9µs    61479.      688B     24.6
#>  5 cli_vec_ansi      19.3µs     21µs    46727.      848B     18.7
#>  6 fansi_vec_ansi    28.1µs   30.4µs    32404.    5.41KB     16.2
#>  7 cli_vec_plain     14.8µs   16.3µs    59891.      848B     24.0
#>  8 fansi_vec_plain   19.1µs   21.1µs    46595.    4.59KB     18.6
#>  9 cli_txt_ansi      18.7µs   20.7µs    47306.        0B     18.9
#> 10 fansi_txt_ansi    23.3µs   25.2µs    38789.    5.12KB     15.5
#> 11 cli_txt_plain     14.1µs   15.8µs    61809.        0B     24.7
#> 12 fansi_txt_plain   14.8µs   16.3µs    60018.      688B     30.0
```

### `ansi_strsplit()`

``` r

bench::mark(
  cli_ansi        = ansi_strsplit(ansi, "i"),
  fansi_ansi      = strsplit_sgr(ansi, "i"),
  base_ansi       = strsplit(ansi, "i"),
  cli_plain       = ansi_strsplit(plain, "i"),
  fansi_plain     = strsplit_sgr(plain, "i"),
  base_plain      = strsplit(plain, "i"),
  cli_vec_ansi    = ansi_strsplit(vec_ansi, "i"),
  fansi_vec_ansi  = strsplit_sgr(vec_ansi, "i"),
  base_vec_ansi   = strsplit(vec_ansi, "i"),
  cli_vec_plain   = ansi_strsplit(vec_plain, "i"),
  fansi_vec_plain = strsplit_sgr(vec_plain, "i"),
  base_vec_plain  = strsplit(vec_plain, "i"),
  cli_txt_ansi    = ansi_strsplit(txt_ansi, "i"),
  fansi_txt_ansi  = strsplit_sgr(txt_ansi, "i"),
  base_txt_ansi   = strsplit(txt_ansi, "i"),
  cli_txt_plain   = ansi_strsplit(txt_plain, "i"),
  fansi_txt_plain = strsplit_sgr(txt_plain, "i"),
  base_txt_plain  = strsplit(txt_plain, "i"),
  check = FALSE
)
```

``` fansi
#> # A tibble: 18 × 6
#>    expression           min   median `itr/sec` mem_alloc `gc/sec`
#>    <bch:expr>      <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#>  1 cli_ansi         65.91µs  71.67µs    13627.  104.86KB    23.3 
#>  2 fansi_ansi       58.65µs  65.33µs    15074.  106.35KB    23.2 
#>  3 base_ansi         2.38µs   2.62µs   371907.      224B     0   
#>  4 cli_plain        67.99µs  73.31µs    12781.    8.09KB    22.9 
#>  5 fansi_plain      58.13µs  61.91µs    15755.    9.62KB    23.3 
#>  6 base_plain        2.08µs   2.25µs   429113.        0B     0   
#>  7 cli_vec_ansi      3.67ms   3.83ms      258.  823.77KB    25.8 
#>  8 fansi_vec_ansi  559.95µs 604.85µs     1523.  846.81KB    27.0 
#>  9 base_vec_ansi    84.57µs  89.03µs    10826.    22.7KB     4.14
#> 10 cli_vec_plain     3.56ms   3.69ms      269.  823.77KB    26.0 
#> 11 fansi_vec_plain 528.21µs 572.54µs     1719.  845.98KB    31.8 
#> 12 base_vec_plain   54.31µs  59.48µs    16643.      848B     6.10
#> 13 cli_txt_ansi      1.68ms   1.77ms      556.    63.6KB     0   
#> 14 fansi_txt_ansi  879.92µs 909.61µs     1096.   35.05KB     2.02
#> 15 base_txt_ansi    72.29µs  78.93µs    12515.   18.47KB     4.08
#> 16 cli_txt_plain     1.29ms   1.32ms      756.    63.6KB     2.02
#> 17 fansi_txt_plain  276.5µs 287.87µs     3433.    30.6KB     4.08
#> 18 base_txt_plain   45.03µs  48.17µs    20367.   11.05KB     4.07
```

### `ansi_strtrim()`

``` r

bench::mark(
  cli_ansi        = ansi_strtrim(ansi, 10),
  fansi_ansi      = strtrim_sgr(ansi, 10),
  base_ansi       = strtrim(ansi, 10),
  cli_plain       = ansi_strtrim(plain, 10),
  fansi_plain     = strtrim_sgr(plain, 10),
  base_plain      = strtrim(plain, 10),
  cli_vec_ansi    = ansi_strtrim(vec_ansi, 10),
  fansi_vec_ansi  = strtrim_sgr(vec_ansi, 10),
  base_vec_ansi   = strtrim(vec_ansi, 10),
  cli_vec_plain   = ansi_strtrim(vec_plain, 10),
  fansi_vec_plain = strtrim_sgr(vec_plain, 10),
  base_vec_plain  = strtrim(vec_plain, 10),
  cli_txt_ansi    = ansi_strtrim(txt_ansi, 10),
  fansi_txt_ansi  = strtrim_sgr(txt_ansi, 10),
  base_txt_ansi   = strtrim(txt_ansi, 10),
  cli_txt_plain   = ansi_strtrim(txt_plain, 10),
  fansi_txt_plain = strtrim_sgr(txt_plain, 10),
  base_txt_plain  = strtrim(txt_plain, 10),
  check = FALSE
)
```

``` fansi
#> # A tibble: 18 × 6
#>    expression           min   median `itr/sec` mem_alloc `gc/sec`
#>    <bch:expr>      <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#>  1 cli_ansi          70.3µs   76.9µs    12648.   33.84KB    25.4 
#>  2 fansi_ansi        27.7µs   31.3µs    30983.   31.42KB    21.7 
#>  3 base_ansi        621.1ns    681ns  1377855.     4.2KB     0   
#>  4 cli_plain         69.8µs   78.5µs    12387.        0B    23.4 
#>  5 fansi_plain       27.5µs   30.8µs    31511.      872B    22.1 
#>  6 base_plain         571ns  631.1ns  1484902.        0B     0   
#>  7 cli_vec_ansi     141.1µs  149.1µs     6592.   16.73KB    14.7 
#>  8 fansi_vec_ansi    66.1µs   69.4µs    14091.    5.59KB    11.7 
#>  9 base_vec_ansi     20.3µs   21.5µs    45967.      848B     0   
#> 10 cli_vec_plain    114.2µs  121.3µs     8161.   16.73KB    14.8 
#> 11 fansi_vec_plain   56.3µs   59.5µs    16463.    5.59KB    12.6 
#> 12 base_vec_plain    16.4µs   17.1µs    57684.      848B     5.77
#> 13 cli_txt_ansi      74.2µs   80.1µs    11469.        0B    21.3 
#> 14 fansi_txt_ansi    27.3µs   29.9µs    32834.      872B    23.0 
#> 15 base_txt_ansi    631.1ns    691ns  1345195.        0B     0   
#> 16 cli_txt_plain     69.3µs   75.3µs    12926.        0B    25.2 
#> 17 fansi_txt_plain   26.8µs   29.4µs    33375.      872B    23.4 
#> 18 base_txt_plain     581ns  611.1ns  1493256.        0B   149.
```

### `ansi_strwrap()`

This function is most useful for longer text, but it is often called for
short text in cli, so it makes sense to benchmark that as well.

``` r

bench::mark(
  cli_ansi        = ansi_strwrap(ansi, 30),
  fansi_ansi      = strwrap_sgr(ansi, 30),
  base_ansi       = strwrap(ansi, 30),
  cli_plain       = ansi_strwrap(plain, 30),
  fansi_plain     = strwrap_sgr(plain, 30),
  base_plain      = strwrap(plain, 30),
  cli_vec_ansi    = ansi_strwrap(vec_ansi, 30),
  fansi_vec_ansi  = strwrap_sgr(vec_ansi, 30),
  base_vec_ansi   = strwrap(vec_ansi, 30),
  cli_vec_plain   = ansi_strwrap(vec_plain, 30),
  fansi_vec_plain = strwrap_sgr(vec_plain, 30),
  base_vec_plain  = strwrap(vec_plain, 30),
  cli_txt_ansi    = ansi_strwrap(txt_ansi, 30),
  fansi_txt_ansi  = strwrap_sgr(txt_ansi, 30),
  base_txt_ansi   = strwrap(txt_ansi, 30),
  cli_txt_plain   = ansi_strwrap(txt_plain, 30),
  fansi_txt_plain = strwrap_sgr(txt_plain, 30),
  base_txt_plain  = strwrap(txt_plain, 30),
  check = FALSE
)
```

``` fansi
#> # A tibble: 18 × 6
#>    expression           min   median `itr/sec` mem_alloc `gc/sec`
#>    <bch:expr>      <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#>  1 cli_ansi        191.17µs 207.05µs    4751.     6.18KB     23.4
#>  2 fansi_ansi       49.47µs  54.83µs   17950.    97.33KB     20.9
#>  3 base_ansi        16.99µs  19.08µs   51285.         0B     20.5
#>  4 cli_plain       120.47µs  129.6µs    7595.         0B     23.0
#>  5 fansi_plain      48.22µs  54.08µs   17983.       872B     23.1
#>  6 base_plain       13.84µs  15.42µs   61899.         0B     18.6
#>  7 cli_vec_ansi     20.17ms  20.84ms      46.8   94.67KB     42.9
#>  8 fansi_vec_ansi  125.91µs 135.36µs    7169.     7.25KB     10.3
#>  9 base_vec_ansi     1.16ms   1.22ms     801.    48.18KB     23.6
#> 10 cli_vec_plain    12.17ms  12.69ms      77.4    2.48KB     37.2
#> 11 fansi_vec_plain 102.54µs 110.06µs    7970.     6.42KB     10.5
#> 12 base_vec_plain  880.32µs  920.4µs    1055.     47.4KB     21.6
#> 13 cli_txt_ansi     13.48ms  14.19ms      68.3    4.27MB     11.8
#> 14 fansi_txt_ansi  121.41µs 126.84µs    7137.     6.77KB     10.4
#> 15 base_txt_ansi   683.04µs 745.47µs    1281.   582.06KB     16.2
#> 16 cli_txt_plain   671.39µs 712.26µs    1365.   369.84KB     15.6
#> 17 fansi_txt_plain  95.48µs 101.21µs    9712.     2.51KB     12.4
#> 18 base_txt_plain  462.73µs 494.76µs    1978.   367.31KB     18.4
```

### `ansi_substr()`

``` r

bench::mark(
  cli_ansi        = ansi_substr(ansi, 2, 10),
  fansi_ansi      = substr_sgr(ansi, 2, 10),
  base_ansi       = substr(ansi, 2, 10),
  cli_plain       = ansi_substr(plain, 2, 10),
  fansi_plain     = substr_sgr(plain, 2, 10),
  base_plain      = substr(plain, 2, 10),
  cli_vec_ansi    = ansi_substr(vec_ansi, 2, 10),
  fansi_vec_ansi  = substr_sgr(vec_ansi, 2, 10),
  base_vec_ansi   = substr(vec_ansi, 2, 10),
  cli_vec_plain   = ansi_substr(vec_plain, 2, 10),
  fansi_vec_plain = substr_sgr(vec_plain, 2, 10),
  base_vec_plain  = substr(vec_plain, 2, 10),
  cli_txt_ansi    = ansi_substr(txt_ansi, 2, 10),
  fansi_txt_ansi  = substr_sgr(txt_ansi, 2, 10),
  base_txt_ansi   = substr(txt_ansi, 2, 10),
  cli_txt_plain   = ansi_substr(txt_plain, 2, 10),
  fansi_txt_plain = substr_sgr(txt_plain, 2, 10),
  base_txt_plain  = substr(txt_plain, 2, 10),
  check = FALSE
)
```

``` fansi
#> # A tibble: 18 × 6
#>    expression           min   median `itr/sec` mem_alloc `gc/sec`
#>    <bch:expr>      <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#>  1 cli_ansi          3.59µs   4.22µs   230138.   25.09KB    23.0 
#>  2 fansi_ansi       39.99µs   44.6µs    22100.   28.48KB    22.1 
#>  3 base_ansi       621.08ns 681.03ns  1357164.        0B     0   
#>  4 cli_plain         3.73µs    4.3µs   226710.        0B    22.7 
#>  5 fansi_plain      38.76µs  44.78µs    21964.    1.98KB    19.9 
#>  6 base_plain      579.98ns 630.97ns  1454949.        0B     0   
#>  7 cli_vec_ansi      13.7µs  14.79µs    66399.     1.7KB    13.3 
#>  8 fansi_vec_ansi   58.17µs  62.85µs    15675.    8.86KB    14.9 
#>  9 base_vec_ansi     3.66µs   4.14µs   239709.      848B     0   
#> 10 cli_vec_plain    12.56µs  13.46µs    73342.     1.7KB     7.33
#> 11 fansi_vec_plain  55.19µs  58.73µs    16788.    8.86KB    14.8 
#> 12 base_vec_plain    3.59µs   4.01µs   244664.      848B    24.5 
#> 13 cli_txt_ansi      3.67µs   4.13µs   236186.        0B    23.6 
#> 14 fansi_txt_ansi   38.33µs  43.77µs    22164.    1.98KB    20.0 
#> 15 base_txt_ansi     2.82µs   2.94µs   317008.        0B     0   
#> 16 cli_txt_plain     4.14µs   4.76µs   203870.        0B    20.4 
#> 17 fansi_txt_plain  39.75µs  45.03µs    21807.    1.98KB    21.8 
#> 18 base_txt_plain    1.95µs   2.03µs   462331.        0B     0
```

### `ansi_tolower()` , `ansi_toupper()`

``` r

bench::mark(
  cli_ansi        = ansi_tolower(ansi),
  base_ansi       = tolower(ansi),
  cli_plain       = ansi_tolower(plain),
  base_plain      = tolower(plain),
  cli_vec_ansi    = ansi_tolower(vec_ansi),
  base_vec_ansi   = tolower(vec_ansi),
  cli_vec_plain   = ansi_tolower(vec_plain),
  base_vec_plain  = tolower(vec_plain),
  cli_txt_ansi    = ansi_tolower(txt_ansi),
  base_txt_ansi   = tolower(txt_ansi),
  cli_txt_plain   = ansi_tolower(txt_plain),
  base_txt_plain  = tolower(txt_plain),
  check = FALSE
)
```

``` fansi
#> # A tibble: 12 × 6
#>    expression          min   median `itr/sec` mem_alloc `gc/sec`
#>    <bch:expr>     <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#>  1 cli_ansi        50.02µs  54.46µs    18090.    12.1KB    16.6 
#>  2 base_ansi      842.03ns 901.05ns  1084058.        0B     0   
#>  3 cli_plain       39.62µs  43.44µs    22553.    8.91KB    15.8 
#>  4 base_plain     621.08ns 661.12ns  1416704.        0B   142.  
#>  5 cli_vec_ansi     2.43ms   2.55ms      390.  838.95KB    22.5 
#>  6 base_vec_ansi   47.62µs  49.42µs    19928.      848B     0   
#>  7 cli_vec_plain    1.33ms   1.38ms      721.  817.08KB    26.4 
#>  8 base_vec_plain  25.42µs  26.76µs    37007.      848B     0   
#>  9 cli_txt_ansi     7.77ms   8.05ms      117.   114.6KB     4.17
#> 10 base_txt_ansi   45.54µs  47.28µs    21104.        0B     0   
#> 11 cli_txt_plain  146.53µs 152.64µs     6462.   18.34KB     6.12
#> 12 base_txt_plain  25.19µs  25.53µs    38875.        0B     0
```

### `ansi_trimws()`

``` r

bench::mark(
  cli_ansi        = ansi_trimws(ansi),
  base_ansi       = trimws(ansi),
  cli_plain       = ansi_trimws(plain),
  base_plain      = trimws(plain),
  cli_vec_ansi    = ansi_trimws(vec_ansi),
  base_vec_ansi   = trimws(vec_ansi),
  cli_vec_plain   = ansi_trimws(vec_plain),
  base_vec_plain  = trimws(vec_plain),
  cli_txt_ansi    = ansi_trimws(txt_ansi),
  base_txt_ansi   = trimws(txt_ansi),
  cli_txt_plain   = ansi_trimws(txt_plain),
  base_txt_plain  = trimws(txt_plain),
  check = FALSE
)
```

``` fansi
#> # A tibble: 12 × 6
#>    expression          min   median `itr/sec` mem_alloc `gc/sec`
#>    <bch:expr>     <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#>  1 cli_ansi        48.53µs  54.05µs    17378.        0B    25.2 
#>  2 base_ansi         8.3µs   9.51µs   101809.        0B    20.4 
#>  3 cli_plain       47.93µs  52.85µs    18459.        0B    25.2 
#>  4 base_plain       8.16µs   9.15µs   106810.        0B    21.4 
#>  5 cli_vec_ansi    96.56µs 104.68µs     9162.     7.2KB    12.4 
#>  6 base_vec_ansi   29.14µs  33.46µs    29783.    1.66KB     5.96
#>  7 cli_vec_plain    86.5µs  98.41µs     9943.     7.2KB    14.5 
#>  8 base_vec_plain  25.83µs  29.21µs    34169.    1.66KB     6.84
#>  9 cli_txt_ansi    84.63µs  90.14µs    10968.        0B    14.4 
#> 10 base_txt_ansi   20.03µs  21.98µs    45051.        0B     9.01
#> 11 cli_txt_plain   76.66µs  81.91µs    12078.        0B    16.6 
#> 12 base_txt_plain  16.88µs  18.35µs    53516.        0B    10.7
```

## UTF-8 functions

### `utf8_nchar()`

``` r

bench::mark(
  cli        = utf8_nchar(uni, type = "chars"),
  base       = nchar(uni, "chars"),
  cli_vec    = utf8_nchar(vec_uni, type = "chars"),
  base_vec   = nchar(vec_uni, "chars"),
  cli_txt    = utf8_nchar(txt_uni, type = "chars"),
  base_txt   = nchar(txt_uni, "chars"),
  check = FALSE
)
```

``` fansi
#> # A tibble: 6 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 cli          4.14µs   4.72µs   206652.        0B    20.7 
#> 2 base       530.97ns 589.99ns  1571510.        0B     0   
#> 3 cli_vec     12.03µs  12.94µs    76042.      448B     7.60
#> 4 base_vec     6.83µs   7.35µs   134843.      448B     0   
#> 5 cli_txt     12.82µs  13.89µs    71148.        0B     7.12
#> 6 base_txt     7.78µs   8.05µs   121831.        0B     0
```

``` r

bench::mark(
  cli        = utf8_nchar(uni, type = "width"),
  base       = nchar(uni, "width"),
  cli_vec    = utf8_nchar(vec_uni, type = "width"),
  base_vec   = nchar(vec_uni, "width"),
  cli_txt    = utf8_nchar(txt_uni, type = "width"),
  base_txt   = nchar(txt_uni, "width"),
  check = FALSE
)
```

``` fansi
#> # A tibble: 6 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 cli          4.24µs   4.84µs   201545.        0B    40.3 
#> 2 base       792.09ns 871.14ns  1082846.        0B     0   
#> 3 cli_vec     15.34µs  16.23µs    60814.      448B     6.08
#> 4 base_vec    29.23µs  30.26µs    32766.      448B     0   
#> 5 cli_txt     14.87µs  15.77µs    62637.        0B     6.26
#> 6 base_txt     51.1µs  52.79µs    18715.        0B     0
```

``` r

bench::mark(
  cli        = utf8_nchar(uni, type = "codepoints"),
  base       = nchar(uni, "chars"),
  cli_vec    = utf8_nchar(vec_uni, type = "codepoints"),
  base_vec   = nchar(vec_uni, "chars"),
  cli_txt    = utf8_nchar(txt_uni, type = "codepoints"),
  base_txt   = nchar(txt_uni, "chars"),
  check = FALSE
)
```

``` fansi
#> # A tibble: 6 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 cli          4.56µs   5.19µs   187862.        0B    18.8 
#> 2 base       540.98ns 600.12ns  1550581.        0B     0   
#> 3 cli_vec     10.85µs  11.79µs    83464.      448B     8.35
#> 4 base_vec     6.83µs   7.42µs   132644.      448B    13.3 
#> 5 cli_txt     11.55µs   13.1µs    75826.        0B     7.58
#> 6 base_txt     7.48µs   7.91µs   124660.        0B     0
```

### `utf8_substr()`

``` r

bench::mark(
  cli        = utf8_substr(uni, 2, 10),
  base       = substr(uni, 2, 10),
  cli_vec    = utf8_substr(vec_uni, 2, 10),
  base_vec   = substr(vec_uni, 2, 10),
  cli_txt    = utf8_substr(txt_uni, 2, 10),
  base_txt   = substr(txt_uni, 2, 10),
  check = FALSE
)
```

``` fansi
#> # A tibble: 6 × 6
#>   expression      min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 cli          3.44µs   3.94µs   247550.    22.2KB    24.8 
#> 2 base       621.08ns 661.01ns  1383640.        0B   138.  
#> 3 cli_vec     15.91µs  16.55µs    59664.     1.7KB     5.97
#> 4 base_vec     4.78µs   4.93µs   199764.      848B     0   
#> 5 cli_txt      3.46µs   3.95µs   247470.        0B    24.7 
#> 6 base_txt      2.6µs   2.81µs   345017.        0B     0
```

## Session info

``` r

sessioninfo::session_info()
```

``` fansi
#> ─ Session info ──────────────────────────────────────────────────────
#>  setting  value
#>  version  R version 4.6.1 (2026-06-24)
#>  os       Ubuntu 24.04.5 LTS
#>  system   x86_64, linux-gnu
#>  ui       X11
#>  language en
#>  collate  C.UTF-8
#>  ctype    C.UTF-8
#>  tz       UTC
#>  date     2026-10-13
#>  pandoc   3.8.3 @ /opt/hostedtoolcache/pandoc/3.8.3/x64/ (via rmarkdown)
#>  quarto   NA
#> 
#> ─ Packages ──────────────────────────────────────────────────────────
#>  package     * version    date (UTC) lib source
#>  bench         1.1.4      2025-01-16 [1] RSPM
#>  bslib         0.12.0     2026-08-04 [1] RSPM
#>  cachem        1.1.0      2024-05-16 [1] RSPM
#>  cli         * 3.6.6.9000 2026-10-13 [1] local
#>  codetools     0.2-20     2024-03-31 [3] CRAN (R 4.6.1)
#>  desc          1.4.3      2023-12-10 [1] RSPM
#>  digest        0.6.39     2025-11-19 [1] RSPM
#>  evaluate      1.0.5      2025-08-27 [1] RSPM
#>  fansi       * 1.0.7      2025-11-19 [1] RSPM
#>  fastmap       1.2.0      2024-05-15 [1] RSPM
#>  fs            2.1.0      2026-04-18 [1] RSPM
#>  glue          1.8.1      2026-04-17 [1] RSPM
#>  htmltools     0.5.9      2025-12-04 [1] RSPM
#>  htmlwidgets   1.6.4      2023-12-06 [1] RSPM
#>  jquerylib     0.1.4      2021-04-26 [1] RSPM
#>  jsonlite      2.0.0      2025-03-27 [1] RSPM
#>  knitr         1.52       2026-09-06 [1] RSPM
#>  lifecycle     1.0.5      2026-01-08 [1] RSPM
#>  magrittr      2.0.5      2026-04-04 [1] RSPM
#>  otel          0.2.0      2025-08-29 [1] RSPM
#>  pillar        1.11.1     2025-09-17 [1] RSPM
#>  pkgconfig     2.0.3      2019-09-22 [1] RSPM
#>  pkgdown       2.2.1      2026-07-07 [1] any (@2.2.1)
#>  profmem       0.7.0      2025-05-02 [1] RSPM
#>  R6            2.6.1      2025-02-15 [1] RSPM
#>  ragg          1.5.2      2026-03-23 [1] RSPM
#>  rlang         1.3.0      2026-07-05 [1] RSPM
#>  rmarkdown     2.32       2026-09-01 [1] RSPM
#>  sass          0.4.10     2025-04-11 [1] RSPM
#>  sessioninfo   1.2.4      2026-06-04 [1] RSPM
#>  systemfonts   1.3.2      2026-03-05 [1] RSPM
#>  textshaping   1.0.5      2026-03-06 [1] RSPM
#>  tibble        3.3.1      2026-01-11 [1] RSPM
#>  utf8          1.2.6      2025-06-08 [1] RSPM
#>  vctrs         0.7.3      2026-04-11 [1] RSPM
#>  xfun          0.61       2026-09-16 [1] RSPM
#>  yaml          2.3.12     2025-12-10 [1] RSPM
#> 
#>  [1] /home/runner/work/_temp/Library
#>  [2] /opt/R/4.6.1/lib/R/site-library
#>  [3] /opt/R/4.6.1/lib/R/library
#>  * ── Packages attached to the search path.
#> 
#> ─────────────────────────────────────────────────────────────────────
```
