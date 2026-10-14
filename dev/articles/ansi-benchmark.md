# ANSI function benchmarks

\$output function (x, options) { if (class == “output” && output_asis(x,
options)) return(x) hook.t(x, options\[\[paste0(“attr.”, class)\]\],
options\[\[paste0(“class.”, class)\]\]) } \<bytecode: 0x55abc6f03580\>
\<environment: 0x55abc799bd60\>

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
#> 1 ansi         46.1µs   49.5µs    19545.    99.6KB     21.2
#> 2 plain        46.3µs   49.7µs    19449.        0B     22.2
#> 3 base         11.4µs   12.6µs    76783.    48.6KB     15.4
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
#> 1 ansi         48.1µs   51.7µs    18698.        0B     21.5
#> 2 plain        47.8µs   51.4µs    18829.        0B     23.5
#> 3 base         13.4µs   14.8µs    65090.        0B     26.0
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
#> 1 ansi       117.51µs 124.76µs     7736.   77.03KB     14.7
#> 2 plain       94.27µs  99.58µs     9716.    8.91KB     14.7
#> 3 base         1.88µs   2.03µs   471262.        0B     47.1
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
#> 1 ansi          345µs    367µs     2689.   33.24KB     19.1
#> 2 plain         347µs    368µs     2685.    1.09KB     21.5
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
#>  1 cli_ansi          5.85µs   6.45µs   148412.    9.27KB    29.7 
#>  2 fansi_ansi       31.07µs   34.2µs    28254.    4.18KB    25.5 
#>  3 cli_plain         5.76µs   6.36µs   151672.        0B    30.3 
#>  4 fansi_plain      30.35µs  32.75µs    28667.      688B    14.3 
#>  5 cli_vec_ansi      7.25µs   7.73µs   124019.      448B     0   
#>  6 fansi_vec_ansi   40.62µs  43.29µs    22190.    5.02KB    11.1 
#>  7 cli_vec_plain     7.79µs   8.27µs   117697.      448B    11.8 
#>  8 fansi_vec_plain  38.91µs  41.34µs    23474.    5.02KB     9.39
#>  9 cli_txt_ansi      5.84µs   6.27µs   155261.        0B    15.5 
#> 10 fansi_txt_ansi   30.84µs   32.9µs    29504.      688B    11.8 
#> 11 cli_txt_plain     6.61µs   6.98µs   139368.        0B    13.9 
#> 12 fansi_txt_plain  38.81µs  41.59µs    23329.    5.02KB    11.7
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
#> 1 cli          56.9µs   58.7µs    16604.    22.7KB     4.06
#> 2 fansi       119.1µs  125.9µs     7794.    55.3KB     4.06
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
#>  1 cli_ansi          6.91µs   7.49µs   129132.        0B    12.9 
#>  2 fansi_ansi       93.74µs  98.59µs     9788.   38.84KB     8.21
#>  3 base_ansi       911.07ns 952.04ns   979868.        0B     0   
#>  4 cli_plain         6.83µs   7.47µs   129597.        0B    13.0 
#>  5 fansi_plain      92.52µs  97.91µs     9912.      688B     8.22
#>  6 base_plain      811.07ns 852.04ns  1074410.        0B     0   
#>  7 cli_vec_ansi     29.28µs  30.24µs    32071.      448B     3.21
#>  8 fansi_vec_ansi  114.14µs 119.78µs     8033.    5.02KB     8.28
#>  9 base_vec_ansi    18.44µs  18.59µs    53061.      448B     0   
#> 10 cli_vec_plain    28.11µs  28.93µs    33840.      448B     3.38
#> 11 fansi_vec_plain 104.62µs 109.76µs     8817.    5.02KB     6.16
#> 12 base_vec_plain   10.77µs  10.86µs    89835.      448B     8.98
#> 13 cli_txt_ansi     29.41µs  30.07µs    32275.        0B     0   
#> 14 fansi_txt_ansi  105.85µs 110.67µs     8759.      688B     8.21
#> 15 base_txt_ansi    18.21µs  18.29µs    53802.        0B     0   
#> 16 cli_txt_plain    27.72µs  28.47µs    34443.        0B     3.44
#> 17 fansi_txt_plain  95.36µs 100.46µs     9648.      688B     8.21
#> 18 base_txt_plain   10.59µs   11.1µs    89498.        0B     8.95
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
#>  1 cli_ansi          8.51µs   9.21µs   104447.        0B    10.4 
#>  2 fansi_ansi       92.67µs  98.95µs     9784.      688B     8.30
#>  3 base_ansi         1.24µs   1.28µs   729879.        0B     0   
#>  4 cli_plain          8.5µs   9.26µs   104857.        0B    10.5 
#>  5 fansi_plain      92.52µs  98.47µs     9798.      688B    10.3 
#>  6 base_plain        1.01µs   1.06µs   885181.        0B     0   
#>  7 cli_vec_ansi     34.78µs  35.71µs    27408.      448B     2.74
#>  8 fansi_vec_ansi  116.15µs 122.06µs     7943.    5.02KB     6.15
#>  9 base_vec_ansi    44.07µs   44.5µs    22195.      448B     0   
#> 10 cli_vec_plain    33.51µs   34.4µs    28392.      448B     5.68
#> 11 fansi_vec_plain 107.39µs 112.41µs     8600.    5.02KB     6.15
#> 12 base_vec_plain   22.98µs  23.25µs    42388.      448B     4.24
#> 13 cli_txt_ansi     34.88µs  35.76µs    25983.        0B     2.60
#> 14 fansi_txt_ansi  107.28µs 112.54µs     8614.      688B     6.12
#> 15 base_txt_ansi    46.56µs  47.21µs    20882.        0B     2.09
#> 16 cli_txt_plain    32.84µs  33.67µs    29066.        0B     2.91
#> 17 fansi_txt_plain  97.49µs 102.71µs     9418.      688B     8.21
#> 18 base_txt_plain   24.68µs  25.31µs    38723.        0B     0
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
#> 1 cli_ansi        6.82µs   7.33µs   131732.        0B    13.2 
#> 2 cli_plain       6.34µs   6.85µs   140772.        0B    14.1 
#> 3 cli_vec_ansi    32.3µs  33.32µs    29405.      848B     2.94
#> 4 cli_vec_plain   10.4µs  11.05µs    88052.      848B     8.81
#> 5 cli_txt_ansi   31.79µs  33.12µs    29516.        0B     2.95
#> 6 cli_txt_plain   7.17µs   7.73µs   116109.        0B     0
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
#>  1 cli_ansi          25.7µs   27.5µs    34530.        0B    13.8 
#>  2 fansi_ansi        28.5µs   30.5µs    31639.    7.24KB    12.7 
#>  3 cli_plain         25.9µs   27.5µs    35172.        0B    14.1 
#>  4 fansi_plain       27.9µs   29.8µs    32245.      688B    12.9 
#>  5 cli_vec_ansi      34.7µs   36.5µs    26524.      848B    13.3 
#>  6 fansi_vec_ansi    55.9µs   58.9µs    16490.    5.41KB     6.18
#>  7 cli_vec_plain     28.5µs   30.3µs    31991.      848B    12.8 
#>  8 fansi_vec_plain   37.3µs     39µs    24333.    4.59KB     9.75
#>  9 cli_txt_ansi      33.6µs     35µs    26743.        0B    10.7 
#> 10 fansi_txt_ansi    44.1µs   45.7µs    20640.    5.12KB    10.3 
#> 11 cli_txt_plain     26.4µs   27.6µs    34663.        0B    13.9 
#> 12 fansi_txt_plain   28.8µs   30.5µs    31773.      688B    12.7
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
#>  1 cli_ansi        165.52µs 173.12µs     5556.  104.86KB    10.3 
#>  2 fansi_ansi      129.03µs 136.68µs     7124.  106.35KB    10.4 
#>  3 base_ansi         4.13µs   4.47µs   217716.      224B     0   
#>  4 cli_plain       163.25µs 170.81µs     5673.    8.09KB    10.3 
#>  5 fansi_plain     127.51µs 135.02µs     7223.    9.62KB    10.3 
#>  6 base_plain        3.61µs    3.9µs   248185.        0B     0   
#>  7 cli_vec_ansi      7.72ms   7.89ms      127.  823.77KB    13.8 
#>  8 fansi_vec_ansi    1.07ms    1.1ms      892.  846.81KB    17.3 
#>  9 base_vec_ansi   156.94µs  162.5µs     6002.    22.7KB     2.04
#> 10 cli_vec_plain     7.63ms   7.82ms      127.  823.77KB    11.2 
#> 11 fansi_vec_plain      1ms   1.04ms      960.  845.98KB    17.8 
#> 12 base_vec_plain  107.22µs 111.28µs     8757.      848B     4.06
#> 13 cli_txt_ansi      3.33ms   3.36ms      297.    63.6KB     0   
#> 14 fansi_txt_ansi    1.57ms   1.61ms      616.   35.05KB     2.02
#> 15 base_txt_ansi   133.91µs  141.4µs     7008.   18.47KB     2.02
#> 16 cli_txt_plain      2.5ms   2.55ms      392.    63.6KB     0   
#> 17 fansi_txt_plain 513.18µs 541.11µs     1817.    30.6KB     2.02
#> 18 base_txt_plain   86.73µs  89.97µs    10875.   11.05KB     2.02
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
#>  1 cli_ansi           150µs 157.27µs     6174.   33.84KB    12.5 
#>  2 fansi_ansi        55.4µs  58.89µs    16450.   31.42KB    10.4 
#>  3 base_ansi         1.06µs   1.13µs   791494.     4.2KB    79.2 
#>  4 cli_plain       148.91µs 155.68µs     6247.        0B    10.3 
#>  5 fansi_plain      55.02µs  58.72µs    16471.      872B    12.5 
#>  6 base_plain      982.08ns   1.03µs   911510.        0B     0   
#>  7 cli_vec_ansi    278.42µs 289.79µs     3383.   16.73KB     6.17
#>  8 fansi_vec_ansi  117.55µs 121.66µs     8000.    5.59KB     8.37
#>  9 base_vec_ansi    36.57µs  37.25µs    26494.      848B     0   
#> 10 cli_vec_plain   236.05µs 246.18µs     3971.   16.73KB     6.18
#> 11 fansi_vec_plain 108.83µs 113.69µs     8549.    5.59KB     8.31
#> 12 base_vec_plain   30.11µs  30.67µs    32197.      848B     0   
#> 13 cli_txt_ansi    158.41µs 165.59µs     5854.        0B    10.3 
#> 14 fansi_txt_ansi   55.09µs  58.82µs    16491.      872B    12.5 
#> 15 base_txt_ansi     1.11µs   1.17µs   814594.        0B     0   
#> 16 cli_txt_plain   146.98µs 154.96µs     6290.        0B    12.2 
#> 17 fansi_txt_plain  53.88µs  56.47µs    17157.      872B    12.5 
#> 18 base_txt_plain    1.02µs   1.07µs   886244.        0B     0
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
#>  1 cli_ansi        423.47µs 449.78µs    2218.     6.18KB    12.5 
#>  2 fansi_ansi       98.14µs 103.67µs    9370.    97.33KB    10.5 
#>  3 base_ansi        38.99µs  41.05µs   23556.         0B     9.43
#>  4 cli_plain       279.68µs 290.96µs    3357.         0B    10.3 
#>  5 fansi_plain      98.48µs 104.44µs    9278.       872B    12.4 
#>  6 base_plain       31.54µs  33.39µs   28985.         0B     8.70
#>  7 cli_vec_ansi     45.01ms  45.07ms      22.1   94.67KB    26.6 
#>  8 fansi_vec_ansi  238.04µs 247.81µs    3959.     7.25KB     4.06
#>  9 base_vec_ansi     2.28ms   2.36ms     422.    48.18KB    12.9 
#> 10 cli_vec_plain    29.35ms  29.51ms      33.8    2.48KB    14.1 
#> 11 fansi_vec_plain 193.46µs 201.31µs    4848.     6.42KB     8.35
#> 12 base_vec_plain    1.65ms   1.72ms     578.     47.4KB    12.7 
#> 13 cli_txt_ansi     27.74ms  28.07ms      35.4    4.27MB     4.43
#> 14 fansi_txt_ansi  227.64µs 236.51µs    4094.     6.77KB     6.13
#> 15 base_txt_ansi     1.29ms   1.32ms     752.   582.06KB     8.77
#> 16 cli_txt_plain     1.31ms   1.35ms     734.   369.84KB     8.68
#> 17 fansi_txt_plain 179.33µs 187.46µs    5205.     2.51KB     6.17
#> 18 base_txt_plain  867.77µs 907.17µs    1089.   367.31KB    11.0
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
#>  1 cli_ansi           6.9µs   7.51µs   127082.   25.09KB    12.7 
#>  2 fansi_ansi       79.98µs   85.2µs    11331.   28.48KB    12.5 
#>  3 base_ansi         1.06µs   1.13µs   835424.        0B     0   
#>  4 cli_plain         6.82µs   7.48µs   129191.        0B    12.9 
#>  5 fansi_plain      80.13µs  85.07µs    11374.    1.98KB    10.4 
#>  6 base_plain        1.04µs    1.1µs   852857.        0B     0   
#>  7 cli_vec_ansi     26.83µs  27.86µs    34968.     1.7KB     7.00
#>  8 fansi_vec_ansi  116.98µs 123.46µs     7866.    8.86KB     6.20
#>  9 base_vec_ansi     6.43µs   6.73µs   145292.      848B     0   
#> 10 cli_vec_plain     23.5µs  24.47µs    39844.     1.7KB     7.97
#> 11 fansi_vec_plain 112.89µs 117.99µs     8180.    8.86KB     6.20
#> 12 base_vec_plain    6.13µs   6.46µs   151364.      848B     0   
#> 13 cli_txt_ansi      6.83µs   7.53µs   126910.        0B    25.4 
#> 14 fansi_txt_ansi   80.24µs  85.14µs    11345.    1.98KB    10.4 
#> 15 base_txt_ansi      6.5µs   6.57µs   148711.        0B     0   
#> 16 cli_txt_plain     7.61µs   8.31µs   116443.        0B    11.6 
#> 17 fansi_txt_plain   79.3µs  85.03µs    11366.    1.98KB    12.3 
#> 18 base_txt_plain    4.16µs   4.22µs   232710.        0B     0
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
#>  1 cli_ansi       111.77µs 116.59µs    8290.     12.1KB     8.22
#>  2 base_ansi        1.34µs    1.4µs  694544.         0B     0   
#>  3 cli_plain       90.89µs   94.6µs   10208.     8.91KB     8.22
#>  4 base_plain       1.05µs   1.09µs  894638.         0B     0   
#>  5 cli_vec_ansi     4.16ms   4.28ms     233.   838.95KB    15.4 
#>  6 base_vec_ansi   75.48µs  76.18µs   12973.       848B     0   
#>  7 cli_vec_plain    2.35ms    2.4ms     413.   817.08KB    13.0 
#>  8 base_vec_plain  45.28µs  45.85µs   21507.       848B     0   
#>  9 cli_txt_ansi    14.91ms  15.08ms      66.2   114.6KB     4.27
#> 10 base_txt_ansi   76.47µs  76.74µs   12866.         0B     0   
#> 11 cli_txt_plain  296.39µs 308.77µs    3176.    18.34KB     2.01
#> 12 base_txt_plain  42.94µs  43.19µs   22834.         0B     0
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
#>  1 cli_ansi        110.2µs  115.4µs     8374.        0B    12.4 
#>  2 base_ansi        16.6µs   17.7µs    54687.        0B    10.9 
#>  3 cli_plain         108µs  113.3µs     8552.        0B    12.4 
#>  4 base_plain       16.4µs   17.6µs    54990.        0B    11.0 
#>  5 cli_vec_ansi      211µs  219.9µs     4439.     7.2KB     6.15
#>  6 base_vec_ansi      60µs   65.8µs    14785.    1.66KB     2.02
#>  7 cli_vec_plain     199µs  207.2µs     4697.     7.2KB     6.16
#>  8 base_vec_plain   53.3µs   59.2µs    16315.    1.66KB     4.06
#>  9 cli_txt_ansi    184.1µs  190.9µs     5100.        0B     6.12
#> 10 base_txt_ansi    41.1µs   42.4µs    22925.        0B     4.59
#> 11 cli_txt_plain   168.8µs  174.9µs     5558.        0B     8.19
#> 12 base_txt_plain   35.2µs   36.3µs    26794.        0B     5.36
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
#> 1 cli          8.14µs   8.79µs   109381.        0B    21.9 
#> 2 base       871.14ns 922.13ns  1016829.        0B     0   
#> 3 cli_vec     23.61µs  24.47µs    39908.      448B     3.99
#> 4 base_vec    11.54µs  11.66µs    83878.      448B     0   
#> 5 cli_txt     23.93µs  24.78µs    39567.        0B     3.96
#> 6 base_txt    12.52µs   12.6µs    77826.        0B     0
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
#> 1 cli          8.12µs   8.79µs   110174.        0B    11.0 
#> 2 base         1.32µs   1.38µs   685919.        0B     0   
#> 3 cli_vec     29.12µs  30.05µs    32592.      448B     3.26
#> 4 base_vec    52.59µs  53.37µs    18498.      448B     0   
#> 5 cli_txt     29.59µs  30.43µs    32198.        0B     3.22
#> 6 base_txt    86.49µs  87.08µs    11351.        0B     0
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
#> 1 cli          8.71µs   9.44µs   102147.        0B    10.2 
#> 2 base       871.02ns 922.01ns  1000223.        0B     0   
#> 3 cli_vec     19.68µs  20.56µs    47473.      448B     4.75
#> 4 base_vec    11.55µs  11.72µs    83703.      448B     8.37
#> 5 cli_txt     20.38µs  21.26µs    45929.        0B     4.59
#> 6 base_txt    12.52µs  12.59µs    78033.        0B     0
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
#> 1 cli          6.33µs   6.99µs   137702.    22.2KB    13.8 
#> 2 base         1.06µs   1.14µs   798143.        0B    79.8 
#> 3 cli_vec      29.5µs  30.44µs    32176.     1.7KB     3.22
#> 4 base_vec     8.23µs    8.5µs   115437.      848B     0   
#> 5 cli_txt      6.37µs   6.96µs   138499.        0B    13.9 
#> 6 base_txt     5.49µs   5.58µs   174932.        0B     0
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
#>  date     2026-10-14
#>  pandoc   3.8.3 @ /opt/hostedtoolcache/pandoc/3.8.3/x64/ (via rmarkdown)
#>  quarto   NA
#> 
#> ─ Packages ──────────────────────────────────────────────────────────
#>  package     * version    date (UTC) lib source
#>  bench         1.1.4      2025-01-16 [1] RSPM
#>  bslib         0.12.0     2026-08-04 [1] RSPM
#>  cachem        1.1.0      2024-05-16 [1] RSPM
#>  cli         * 3.6.6.9000 2026-10-14 [1] local
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
