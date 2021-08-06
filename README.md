# R 语言和 Stata 中的数据转换（透视、长宽数据转换）

R 语言我们主要是以 gather/spread 和 pivot_longer/pivot_wider，Stata 我们主要使用 gather/spread 命令（我在课程附件中准备了 tidy 安装包，可以使用 net install 安装）。因为 Stata 的 reshape long 和 reshape wide 看起来很强大，实际上对数据的规范程度要求较多，使用起来并不方便，所以我就不再推荐使用了。

## 宽数据转长数据

### 列名为字符串

#### 使用 R 语言

这是一份宽数据：

```r
library(tidyverse)
read_csv('relig_income.csv') -> relig_income
relig_income

#> # A tibble: 18 x 11
#>    religion `<$10k` `$10-20k` `$20-30k` `$30-40k` `$40-50k` `$50-75k` `$75-100k`
#>    <chr>      <dbl>     <dbl>     <dbl>     <dbl>     <dbl>     <dbl>      <dbl>
#>  1 Agnostic      27        34        60        81        76       137        122
#>  2 Atheist       12        27        37        52        35        70         73
#>  3 Buddhist      27        21        30        34        33        58         62
#>  4 Catholic     418       617       732       670       638      1116        949
#>  5 Don’t …       15        14        15        11        10        35         21
#>  6 Evangel…     575       869      1064       982       881      1486        949
#>  7 Hindu          1         9         7         9        11        34         47
#>  8 Histori…     228       244       236       238       197       223        131
#>  9 Jehovah…      20        27        24        24        21        30         15
#> 10 Jewish        19        19        25        25        30        95         69
#> 11 Mainlin…     289       495       619       655       651      1107        939
#> 12 Mormon        29        40        48        51        56       112         85
#> 13 Muslim         6         7         9        10         9        23         16
#> 14 Orthodox      13        17        23        32        32        47         38
#> 15 Other C…       9         7        11        13        13        14         18
#> 16 Other F…      20        33        40        46        49        63         46
#> 17 Other W…       5         2         3         4         2         7          3
#> 18 Unaffil…     217       299       374       365       341       528        407
#> # … with 3 more variables: $100-150k <dbl>, >150k <dbl>,
#> #   Don't know/refused <dbl>
```

这个数据展示了各种信仰群体不同收入层次的人数分布。我们可以使用 tidyr 包的 gather() 或者 pivot_longer() 函数将它转换成长数据：

使用 gather()：

```r
relig_income %>% 
  gather(!religion, key = "income", value = "count")

#> # A tibble: 180 x 3
#>    religion                income count
#>    <chr>                   <chr>  <dbl>
#>  1 Agnostic                <$10k     27
#>  2 Atheist                 <$10k     12
#>  3 Buddhist                <$10k     27
#>  4 Catholic                <$10k    418
#>  5 Don’t know/refused      <$10k     15
#>  6 Evangelical Prot        <$10k    575
#>  7 Hindu                   <$10k      1
#>  8 Historically Black Prot <$10k    228
#>  9 Jehovah's Witness       <$10k     20
#> 10 Jewish                  <$10k     19
#> # … with 170 more rows
```

或者：

```r
relig_income %>% 
  gather(2:ncol(.), key = "income", value = "count")
#> # A tibble: 180 x 3
#>    religion                income count
#>    <chr>                   <chr>  <dbl>
#>  1 Agnostic                <$10k     27
#>  2 Atheist                 <$10k     12
#>  3 Buddhist                <$10k     27
#>  4 Catholic                <$10k    418
#>  5 Don’t know/refused      <$10k     15
#>  6 Evangelical Prot        <$10k    575
#>  7 Hindu                   <$10k      1
#>  8 Historically Black Prot <$10k    228
#>  9 Jehovah's Witness       <$10k     20
#> 10 Jewish                  <$10k     19
#> # … with 170 more rows
```

使用 pivot_longer()：

```r
relig_income %>% 
  pivot_longer(!religion, names_to = "income", values_to = "count")

#> # A tibble: 180 x 3
#>    religion income             count
#>    <chr>    <chr>              <dbl>
#>  1 Agnostic <$10k                 27
#>  2 Agnostic $10-20k               34
#>  3 Agnostic $20-30k               60
#>  4 Agnostic $30-40k               81
#>  5 Agnostic $40-50k               76
#>  6 Agnostic $50-75k              137
#>  7 Agnostic $75-100k             122
#>  8 Agnostic $100-150k            109
#>  9 Agnostic >150k                 84
#> 10 Agnostic Don't know/refused    96
#> # … with 170 more rows
```

或者：

```r
relig_income %>% 
  pivot_longer(2:ncol(.), names_to = "income", values_to = "count")
#> # A tibble: 180 x 3
#>    religion income             count
#>    <chr>    <chr>              <dbl>
#>  1 Agnostic <$10k                 27
#>  2 Agnostic $10-20k               34
#>  3 Agnostic $20-30k               60
#>  4 Agnostic $30-40k               81
#>  5 Agnostic $40-50k               76
#>  6 Agnostic $50-75k              137
#>  7 Agnostic $75-100k             122
#>  8 Agnostic $100-150k            109
#>  9 Agnostic >150k                 84
#> 10 Agnostic Don't know/refused    96
#> # … with 170 more rows
```

#### 使用 Stata

我们先读取这个文件：

```stata
clear all
import delimited using relig_income.csv, clear varnames(nonames)
```

![](https://mdniceczx.oss-cn-beijing.aliyuncs.com/image_20210806105525.png)

因为 Stata 的变量要求比较多，所以不能直接设置第一行数据作为变量名，如果调整起来太麻烦了，所以我们就不折腾这个了，使用下面这样的方法更好：

```stata
clear all
import delimited using relig_income.csv, clear varnames(nonames)
* 直接 gather
gather v2-v11
keep if v1 == "religion"
drop v1
ren value income
save varnames, replace 

* 再次导入 relig_income.csv
import delimited using relig_income.csv, clear varnames(nonames)
gather v2-v11
merge m:1 variable using varnames
ren v1 religion
drop variable
ren value count
drop _m
drop if religion == "religion"
destring, replace 
order religion income count
```

### 列名中含有数值

#### 使用 R 语言

例如这份数据：

```r
read_csv('billboard.csv') -> billboard
billboard

#> # A tibble: 317 x 79
#>    artist   track   date.entered   wk1   wk2   wk3   wk4   wk5   wk6   wk7   wk8
#>    <chr>    <chr>   <date>       <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#>  1 2 Pac    Baby D… 2000-02-26      87    82    72    77    87    94    99    NA
#>  2 2Ge+her  The Ha… 2000-09-02      91    87    92    NA    NA    NA    NA    NA
#>  3 3 Doors… Krypto… 2000-04-08      81    70    68    67    66    57    54    53
#>  4 3 Doors… Loser   2000-10-21      76    76    72    69    67    65    55    59
#>  5 504 Boyz Wobble… 2000-04-15      57    34    25    17    17    31    36    49
#>  6 98^0     Give M… 2000-08-19      51    39    34    26    26    19     2     2
#>  7 A*Teens  Dancin… 2000-07-08      97    97    96    95   100    NA    NA    NA
#>  8 Aaliyah  I Don'… 2000-01-29      84    62    51    41    38    35    35    38
#>  9 Aaliyah  Try Ag… 2000-03-18      59    53    38    28    21    18    16    14
#> 10 Adams, … Open M… 2000-08-26      76    76    74    69    68    67    61    58
#> # … with 307 more rows, and 68 more variables: wk9 <dbl>, wk10 <dbl>,
#> #   wk11 <dbl>, wk12 <dbl>, wk13 <dbl>, wk14 <dbl>, wk15 <dbl>, wk16 <dbl>,
#> #   wk17 <dbl>, wk18 <dbl>, wk19 <dbl>, wk20 <dbl>, wk21 <dbl>, wk22 <dbl>,
#> #   wk23 <dbl>, wk24 <dbl>, wk25 <dbl>, wk26 <dbl>, wk27 <dbl>, wk28 <dbl>,
#> #   wk29 <dbl>, wk30 <dbl>, wk31 <dbl>, wk32 <dbl>, wk33 <dbl>, wk34 <dbl>,
#> #   wk35 <dbl>, wk36 <dbl>, wk37 <dbl>, wk38 <dbl>, wk39 <dbl>, wk40 <dbl>,
#> #   wk41 <dbl>, wk42 <dbl>, wk43 <dbl>, wk44 <dbl>, wk45 <dbl>, wk46 <dbl>,
#> #   wk47 <dbl>, wk48 <dbl>, wk49 <dbl>, wk50 <dbl>, wk51 <dbl>, wk52 <dbl>,
#> #   wk53 <dbl>, wk54 <dbl>, wk55 <dbl>, wk56 <dbl>, wk57 <dbl>, wk58 <dbl>,
#> #   wk59 <dbl>, wk60 <dbl>, wk61 <dbl>, wk62 <dbl>, wk63 <dbl>, wk64 <dbl>,
#> #   wk65 <dbl>, wk66 <lgl>, wk67 <lgl>, wk68 <lgl>, wk69 <lgl>, wk70 <lgl>,
#> #   wk71 <lgl>, wk72 <lgl>, wk73 <lgl>, wk74 <lgl>, wk75 <lgl>, wk76 <lgl>
```

这份数据从第四列开始，列名都是 wk + 数字的格式，我们也可以使用 gather() 或者 pivot_longer() 函数将其转为长数据：

使用 gather()：

```r
billboard %>% 
  gather(starts_with("wk"), key = "week", value = "rank", na.rm = T)
```

使用 pivot_longer()：

```r
billboard %>% 
  pivot_longer(
    cols = starts_with("wk"), 
    names_to = "week", 
    values_to = "rank",
    values_drop_na = TRUE
  )
#> # A tibble: 5,307 x 5
#>    artist         track                   date.entered week   rank
#>    <chr>          <chr>                   <date>       <chr> <dbl>
#>  1 2 Pac          Baby Don't Cry (Keep... 2000-02-26   wk1      87
#>  2 2Ge+her        The Hardest Part Of ... 2000-09-02   wk1      91
#>  3 3 Doors Down   Kryptonite              2000-04-08   wk1      81
#>  4 3 Doors Down   Loser                   2000-10-21   wk1      76
#>  5 504 Boyz       Wobble Wobble           2000-04-15   wk1      57
#>  6 98^0           Give Me Just One Nig... 2000-08-19   wk1      51
#>  7 A*Teens        Dancing Queen           2000-07-08   wk1      97
#>  8 Aaliyah        I Don't Wanna           2000-01-29   wk1      84
#>  9 Aaliyah        Try Again               2000-03-18   wk1      59
#> 10 Adams, Yolanda Open My Heart           2000-08-26   wk1      76
#> # … with 5,297 more rows
```

另外我们也注意到 week 变量的值实际上仅保留数字即可，pivot_longer() 函数可以直接实现：

```r
billboard %>% 
  pivot_longer(
    cols = starts_with("wk"), 
    names_to = "week", 
    names_prefix = "wk",
    names_transform = list(week = as.integer),
    values_to = "rank",
    values_drop_na = TRUE,
  )
#> # A tibble: 5,307 x 5
#>    artist  track                   date.entered week   rank
#>    <chr>   <chr>                   <date>       <chr> <dbl>
#>  1 2 Pac   Baby Don't Cry (Keep... 2000-02-26   wk1      87
#>  2 2 Pac   Baby Don't Cry (Keep... 2000-02-26   wk2      82
#>  3 2 Pac   Baby Don't Cry (Keep... 2000-02-26   wk3      72
#>  4 2 Pac   Baby Don't Cry (Keep... 2000-02-26   wk4      77
#>  5 2 Pac   Baby Don't Cry (Keep... 2000-02-26   wk5      87
#>  6 2 Pac   Baby Don't Cry (Keep... 2000-02-26   wk6      94
#>  7 2 Pac   Baby Don't Cry (Keep... 2000-02-26   wk7      99
#>  8 2Ge+her The Hardest Part Of ... 2000-09-02   wk1      91
#>  9 2Ge+her The Hardest Part Of ... 2000-09-02   wk2      87
#> 10 2Ge+her The Hardest Part Of ... 2000-09-02   wk3      92
#> # … with 5,297 more rows
```

或者这样也可以：

```r
billboard %>% 
  pivot_longer(
    cols = starts_with("wk"), 
    names_to = "week", 
    names_transform = list(week = readr::parse_number),
    values_to = "rank",
    values_drop_na = TRUE,
  )
#> # A tibble: 5,307 x 5
#>    artist  track                   date.entered  week  rank
#>    <chr>   <chr>                   <date>       <int> <dbl>
#>  1 2 Pac   Baby Don't Cry (Keep... 2000-02-26       1    87
#>  2 2 Pac   Baby Don't Cry (Keep... 2000-02-26       2    82
#>  3 2 Pac   Baby Don't Cry (Keep... 2000-02-26       3    72
#>  4 2 Pac   Baby Don't Cry (Keep... 2000-02-26       4    77
#>  5 2 Pac   Baby Don't Cry (Keep... 2000-02-26       5    87
#>  6 2 Pac   Baby Don't Cry (Keep... 2000-02-26       6    94
#>  7 2 Pac   Baby Don't Cry (Keep... 2000-02-26       7    99
#>  8 2Ge+her The Hardest Part Of ... 2000-09-02       1    91
#>  9 2Ge+her The Hardest Part Of ... 2000-09-02       2    87
#> 10 2Ge+her The Hardest Part Of ... 2000-09-02       3    92
#> # … with 5,297 more rows
```

如果你使用 gather() 函数则需要再增加一步：

```r
billboard %>% 
  gather(starts_with("wk"), key = "week", value = "rank", na.rm = T) %>% 
  mutate(week = str_remove_all(week, "wk"),
         week = as.integer(week))
#> # A tibble: 5,307 x 5
#>    artist         track                   date.entered  week  rank
#>    <chr>          <chr>                   <date>       <int> <dbl>
#>  1 2 Pac          Baby Don't Cry (Keep... 2000-02-26       1    87
#>  2 2Ge+her        The Hardest Part Of ... 2000-09-02       1    91
#>  3 3 Doors Down   Kryptonite              2000-04-08       1    81
#>  4 3 Doors Down   Loser                   2000-10-21       1    76
#>  5 504 Boyz       Wobble Wobble           2000-04-15       1    57
#>  6 98^0           Give Me Just One Nig... 2000-08-19       1    51
#>  7 A*Teens        Dancing Queen           2000-07-08       1    97
#>  8 Aaliyah        I Don't Wanna           2000-01-29       1    84
#>  9 Aaliyah        Try Again               2000-03-18       1    59
#> 10 Adams, Yolanda Open My Heart           2000-08-26       1    76
#> # … with 5,297 more rows
```

#### 使用 Stata 

同样，我们还是先使用 import delimited 读取：

```stata
import delimited using billboard.csv, clear
tostring _all, replace 
gather wk*
ren variable week
ren value rank
replace week = subinstr(week, "wk", "", .)
drop if rank == "NA"
destring _all, replace 
gen date = date(dateentered, "YMD")
format date %tdCY-N-D
drop dateentered
ren date dateentered
```

结果：

![](https://mdniceczx.oss-cn-beijing.aliyuncs.com/image_20210806105655.png)

### 列名中包含多个变数

例如 who 数据：

```r
read_csv('who.csv') -> who
who
#> # A tibble: 7,240 x 60
#>    country  iso2  iso3   year new_sp_m014 new_sp_m1524 new_sp_m2534 new_sp_m3544
#>    <chr>    <chr> <chr> <dbl>       <dbl>        <dbl>        <dbl>        <dbl>
#>  1 Afghani… AF    AFG    1980          NA           NA           NA           NA
#>  2 Afghani… AF    AFG    1981          NA           NA           NA           NA
#>  3 Afghani… AF    AFG    1982          NA           NA           NA           NA
#>  4 Afghani… AF    AFG    1983          NA           NA           NA           NA
#>  5 Afghani… AF    AFG    1984          NA           NA           NA           NA
#>  6 Afghani… AF    AFG    1985          NA           NA           NA           NA
#>  7 Afghani… AF    AFG    1986          NA           NA           NA           NA
#>  8 Afghani… AF    AFG    1987          NA           NA           NA           NA
#>  9 Afghani… AF    AFG    1988          NA           NA           NA           NA
#> 10 Afghani… AF    AFG    1989          NA           NA           NA           NA
#> # … with 7,230 more rows, and 52 more variables: new_sp_m4554 <dbl>,
#> #   new_sp_m5564 <dbl>, new_sp_m65 <dbl>, new_sp_f014 <dbl>,
#> #   new_sp_f1524 <dbl>, new_sp_f2534 <dbl>, new_sp_f3544 <dbl>,
#> #   new_sp_f4554 <dbl>, new_sp_f5564 <dbl>, new_sp_f65 <dbl>,
#> #   new_sn_m014 <dbl>, new_sn_m1524 <dbl>, new_sn_m2534 <dbl>,
#> #   new_sn_m3544 <dbl>, new_sn_m4554 <dbl>, new_sn_m5564 <dbl>,
#> #   new_sn_m65 <dbl>, new_sn_f014 <dbl>, new_sn_f1524 <dbl>,
#> #   new_sn_f2534 <dbl>, new_sn_f3544 <dbl>, new_sn_f4554 <dbl>,
#> #   new_sn_f5564 <dbl>, new_sn_f65 <dbl>, new_ep_m014 <dbl>,
#> #   new_ep_m1524 <dbl>, new_ep_m2534 <dbl>, new_ep_m3544 <dbl>,
#> #   new_ep_m4554 <dbl>, new_ep_m5564 <dbl>, new_ep_m65 <dbl>,
#> #   new_ep_f014 <dbl>, new_ep_f1524 <dbl>, new_ep_f2534 <dbl>,
#> #   new_ep_f3544 <dbl>, new_ep_f4554 <dbl>, new_ep_f5564 <dbl>,
#> #   new_ep_f65 <dbl>, newrel_m014 <dbl>, newrel_m1524 <dbl>,
#> #   newrel_m2534 <dbl>, newrel_m3544 <dbl>, newrel_m4554 <dbl>,
#> #   newrel_m5564 <dbl>, newrel_m65 <dbl>, newrel_f014 <dbl>,
#> #   newrel_f1524 <dbl>, newrel_f2534 <dbl>, newrel_f3544 <dbl>,
#> #   newrel_f4554 <dbl>, newrel_f5564 <dbl>, newrel_f65 <dbl>
```

这是个宽数据，它的变量名包含三个变数，`new_*_**`，对于这样的数据集我们可以这样：

```r
who %>% pivot_longer(
  cols = new_sp_m014:newrel_f65,
  names_to = c("diagnosis", "gender", "age"), 
  names_pattern = "new_?(.*)_(.)(.*)",
  values_to = "count",
  values_drop_na = TRUE
)
#> # A tibble: 76,046 x 8
#>    country     iso2  iso3   year diagnosis gender age   count
#>    <chr>       <chr> <chr> <dbl> <chr>     <chr>  <chr> <dbl>
#>  1 Afghanistan AF    AFG    1997 sp        m      014       0
#>  2 Afghanistan AF    AFG    1997 sp        m      1524     10
#>  3 Afghanistan AF    AFG    1997 sp        m      2534      6
#>  4 Afghanistan AF    AFG    1997 sp        m      3544      3
#>  5 Afghanistan AF    AFG    1997 sp        m      4554      5
#>  6 Afghanistan AF    AFG    1997 sp        m      5564      2
#>  7 Afghanistan AF    AFG    1997 sp        m      65        0
#>  8 Afghanistan AF    AFG    1997 sp        f      014       5
#>  9 Afghanistan AF    AFG    1997 sp        f      1524     38
#> 10 Afghanistan AF    AFG    1997 sp        f      2534     36
#> # … with 76,036 more rows
```

这个正则表达式很简单：需要注意这几个限定符的区别：

字符 |	描述
:-:|:-:
* |	匹配前面的子表达式零次或多次。例如，zo* 能匹配 "z" 以及 "zoo"。* 等价于{0,}。
+ |	匹配前面的子表达式一次或多次。例如，'zo+' 能匹配 "zo" 以及 "zoo"，但不能匹配 "z"。+ 等价于 {1,}。
? |	匹配前面的子表达式零次或一次。例如，"do(es)?" 可以匹配 "do" 、 "does" 中的 "does" 、 "doxy" 中的 "do" 。? 等价于 {0,1}。
{n} |	n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "Bob" 中的 'o'，但是能匹配 "food" 中的两个 o。
{n,} |	n 是一个非负整数。至少匹配n 次。例如，'o{2,}' 不能匹配 "Bob" 中的 'o'，但能匹配 "foooood" 中的所有 o。'o{1,}' 等价于 'o+'。'o{0,}' 则等价于 'o*'。
{n,m} |	m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次。例如，"o{1,3}" 将匹配 "fooooood" 中的前三个 o。'o{0,1}' 等价于 'o?'。请注意在逗号和两个数之间不能有空格。

关于正则表达式的详细使用方法大家可以参考这个：https://www.runoob.com/regexp/regexp-syntax.html 不过常用的正则表达式没几个，大家也可以遇到一个记一个。

使用 pivot_longer() 函数的 names_transform 参数可以实现转换后的变量类型转换：

```r
who %>% pivot_longer(
  cols = new_sp_m014:newrel_f65,
  names_to = c("diagnosis", "gender", "age"), 
  names_pattern = "new_?(.*)_(.)(.*)",
  names_transform = list(
    gender = ~ readr::parse_factor(.x, levels = c("f", "m")),
    age = ~ readr::parse_factor(
      .x,
      levels = c("014", "1524", "2534", "3544", "4554", "5564", "65"), 
      ordered = TRUE
    )
  ),
  values_to = "count",
  values_drop_na = TRUE
)

#> # A tibble: 76,046 x 8
#>    country     iso2  iso3   year diagnosis gender age   count
#>    <chr>       <chr> <chr> <dbl> <chr>     <fct>  <ord> <dbl>
#>  1 Afghanistan AF    AFG    1997 sp        m      014       0
#>  2 Afghanistan AF    AFG    1997 sp        m      1524     10
#>  3 Afghanistan AF    AFG    1997 sp        m      2534      6
#>  4 Afghanistan AF    AFG    1997 sp        m      3544      3
#>  5 Afghanistan AF    AFG    1997 sp        m      4554      5
#>  6 Afghanistan AF    AFG    1997 sp        m      5564      2
#>  7 Afghanistan AF    AFG    1997 sp        m      65        0
#>  8 Afghanistan AF    AFG    1997 sp        f      014       5
#>  9 Afghanistan AF    AFG    1997 sp        f      1524     38
#> 10 Afghanistan AF    AFG    1997 sp        f      2534     36
#> # … with 76,036 more rows
```

大家可以注意到，这个 pivot_\* 函数的功能好多，越来越复杂了，所以这也是我不推荐使用的原因（我平常主要用 gather 和 spread），这是因为功能越多的函数记忆起来越困难，如果实在懒得记忆，不如把简单的函数灵活组合使用，例如上面的代码使用 gather 也可以实现：

```r
who %>% 
  gather(new_sp_m014:newrel_f65, key = "var", value = "count") %>% 
  mutate(diagnosis = str_match(var, "new_?(.*)_(.)(.*)")[,2],
           gender = str_match(var, "new_?(.*)_(.)(.*)")[,3],
           age = str_match(var, "new_?(.*)_(.)(.*)")[,4]) %>% 
  select(-var) %>% 
  subset(!is.na(count))
#> # A tibble: 76,046 x 8
#>    country     iso2  iso3   year count diagnosis gender age  
#>    <chr>       <chr> <chr> <dbl> <dbl> <chr>     <chr>  <chr>
#>  1 Afghanistan AF    AFG    1997     0 sp        m      014  
#>  2 Afghanistan AF    AFG    1998    30 sp        m      014  
#>  3 Afghanistan AF    AFG    1999     8 sp        m      014  
#>  4 Afghanistan AF    AFG    2000    52 sp        m      014  
#>  5 Afghanistan AF    AFG    2001   129 sp        m      014  
#>  6 Afghanistan AF    AFG    2002    90 sp        m      014  
#>  7 Afghanistan AF    AFG    2003   127 sp        m      014  
#>  8 Afghanistan AF    AFG    2004   139 sp        m      014  
#>  9 Afghanistan AF    AFG    2005   151 sp        m      014  
#> 10 Afghanistan AF    AFG    2006   193 sp        m      014  
#> # … with 76,036 more rows
```

#### 使用 Stata

使用 Stata 的 gather 命令同样可以完成这个操作：

```stata
import delimited using who.csv, clear 
gather new*
drop if value == "NA"
gen diagnosis = ustrregexs(1) if ustrregexm(var, "new_?(.*)_(.)(.*)")
gen gender = ustrregexs(2) if ustrregexm(var, "new_?(.*)_(.)(.*)")
gen age = ustrregexs(3) if ustrregexm(var, "new_?(.*)_(.)(.*)")
drop var
ren val count
```

### 每行包含多种观测值

例如我们看这个数据：

```r
read_csv('family.csv') -> family
family

#> # A tibble: 5 x 5
#>   family dob_child1 dob_child2 gender_child1 gender_child2
#>    <dbl> <date>     <date>             <dbl>         <dbl>
#> 1      1 1998-11-26 2000-01-29             1             2
#> 2      2 1996-06-22 NA                     2            NA
#> 3      3 2002-07-11 2004-04-05             2             2
#> 4      4 2004-10-10 2009-08-27             1             1
#> 5      5 2000-12-05 2005-02-28             2             1
```

这个数据框的三四列是孩子的出生日期，五六列是孩子的性别，我们如何将这个数据框转成长数据呢？

```r
family %>% 
  pivot_longer(
    !family, 
    names_to = c(".value", "child"), 
    names_sep = "_", 
    values_drop_na = TRUE
  )
#> # A tibble: 9 x 4
#>   family child  dob        gender
#>    <dbl> <chr>  <date>      <dbl>
#> 1      1 child1 1998-11-26      1
#> 2      1 child2 2000-01-29      2
#> 3      2 child1 1996-06-22      2
#> 4      3 child1 2002-07-11      2
#> 5      3 child2 2004-04-05      2
#> 6      4 child1 2004-10-10      1
#> 7      4 child2 2009-08-27      1
#> 8      5 child1 2000-12-05      2
#> 9      5 child2 2005-02-28      1
```

有点绕了，我们再用 gather() 函数试试：

```r
family %>% 
  gather(!family, key = "key", value = "val") %>% 
  separate(key, into = c("key1", "child"), sep = "_") %>% 
  spread(key1, val) %>% 
  dplyr::filter(!is.na(gender)) %>% 
  mutate(dob = lubridate::as_date(dob))
#> # A tibble: 9 x 4
#>   family child  dob        gender
#>    <dbl> <chr>  <date>      <dbl>
#> 1      1 child1 1998-11-26      1
#> 2      1 child2 2000-01-29      2
#> 3      2 child1 1996-06-22      2
#> 4      3 child1 2002-07-11      2
#> 5      3 child2 2004-04-05      2
#> 6      4 child1 2004-10-10      1
#> 7      4 child2 2009-08-27      1
#> 8      5 child1 2000-12-05      2
#> 9      5 child2 2005-02-28      1
```

这样就可以了。

我们再看一下另外一个案例：

```r
read_csv('anscombe.csv') -> anscombe
anscombe

#> # A tibble: 11 x 8
#>       x1    x2    x3    x4    y1    y2    y3    y4
#>    <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#>  1    10    10    10     8  8.04  9.14  7.46  6.58
#>  2     8     8     8     8  6.95  8.14  6.77  5.76
#>  3    13    13    13     8  7.58  8.74 12.7   7.71
#>  4     9     9     9     8  8.81  8.77  7.11  8.84
#>  5    11    11    11     8  8.33  9.26  7.81  8.47
#>  6    14    14    14     8  9.96  8.1   8.84  7.04
#>  7     6     6     6     8  7.24  6.13  6.08  5.25
#>  8     4     4     4    19  4.26  3.1   5.39 12.5 
#>  9    12    12    12     8 10.8   9.13  8.15  5.56
#> 10     7     7     7     8  4.82  7.26  6.42  7.91
#> 11     5     5     5     8  5.68  4.74  5.73  6.89
```

这个数据框包含两类变量，可以这样转换成长数据：

```r
anscombe %>% 
  pivot_longer(everything(), 
    names_to = c(".value", "set"), 
    names_pattern = "(.)(.)"
  ) %>% 
  arrange(set, x, y)
#> # A tibble: 44 x 3
#>    set       x     y
#>    <chr> <dbl> <dbl>
#>  1 1         4  4.26
#>  2 1         5  5.68
#>  3 1         6  7.24
#>  4 1         7  4.82
#>  5 1         8  6.95
#>  6 1         9  8.81
#>  7 1        10  8.04
#>  8 1        11  8.33
#>  9 1        12 10.8 
#> 10 1        13  7.58
#> # … with 34 more rows
```

使用 gather 函数可以这样：

```r
anscombe %>% 
  mutate(id = row.names(.)) %>% 
  gather(!id, key = "key", value = "val") %>% 
  mutate(var = str_match(key, "(.)(.)")[,2],
         set = str_match(key, "(.)(.)")[,3]) %>% 
  select(-key) %>% 
  spread(var, val) %>% 
  arrange(set, x, y) %>% 
  select(-id)
#> # A tibble: 44 x 3
#>    set       x     y
#>    <chr> <dbl> <dbl>
#>  1 1         4  4.26
#>  2 1         5  5.68
#>  3 1         6  7.24
#>  4 1         7  4.82
#>  5 1         8  6.95
#>  6 1         9  8.81
#>  7 1        10  8.04
#>  8 1        11  8.33
#>  9 1        12 10.8 
#> 10 1        13  7.58
#> # … with 34 more rows
```

再看这个例子：

```r
read_csv('pnl.csv') -> pnl
pnl

#> # A tibble: 4 x 7
#>       x     a     b      y1     y2    z1    z2
#>   <dbl> <dbl> <dbl>   <dbl>  <dbl> <dbl> <dbl>
#> 1     1     1     0 -0.202  -0.359     3    -2
#> 2     2     1     1  0.0766  0.253     3    -2
#> 3     3     0     1 -0.919  -0.324     3    -2
#> 4     4     0     1 -1.11   -1.37      3    -2
```

转换成长数据：

```r
pnl %>% 
  pivot_longer(
    !c(x, a, b), 
    names_to = c(".value", "time"), 
    names_pattern = "(.)(.)"
  )
#> # A tibble: 8 x 6
#>       x     a     b time        y     z
#>   <dbl> <dbl> <dbl> <chr>   <dbl> <dbl>
#> 1     1     1     0 1     -0.202      3
#> 2     1     1     0 2     -0.359     -2
#> 3     2     1     1 1      0.0766     3
#> 4     2     1     1 2      0.253     -2
#> 5     3     0     1 1     -0.919      3
#> 6     3     0     1 2     -0.324     -2
#> 7     4     0     1 1     -1.11       3
#> 8     4     0     1 2     -1.37      -2
```

使用 gather 函数也不复杂：

```r
pnl %>% 
  gather(!c(x, a, b), key = "key", value = "val") %>% 
  mutate(var = str_match(key, "(.)(.)")[,2],
         time = str_match(key, "(.)(.)")[,3]) %>% 
  select(-key) %>% 
  spread(var, val)
#> # A tibble: 8 x 6
#>       x     a     b time        y     z
#>   <dbl> <dbl> <dbl> <chr>   <dbl> <dbl>
#> 1     1     1     0 1     -0.202      3
#> 2     1     1     0 2     -0.359     -2
#> 3     2     1     1 1      0.0766     3
#> 4     2     1     1 2      0.253     -2
#> 5     3     0     1 1     -0.919      3
#> 6     3     0     1 2     -0.324     -2
#> 7     4     0     1 1     -1.11       3
#> 8     4     0     1 2     -1.37      -2
```

#### 使用 Stata 

我们再用 Stata 完成上面的几个例子：

1. family.csv

```stata
import delimited using family.csv, clear 
tostring _all, replace 
gather *_*
split var, parse(_)
keep family val variable1 variable2
ren variable2 child
spread var val
drop if gender == "NA"
destring, replace 
gen date = date(dob, "YMD")
format date %tdCY-N-D
drop dob
ren date dob
```

2. anscombe.csv

```stata
import delimited using anscombe.csv, clear
gen id = _n
gather x* y*
gen var = substr(variable, 1, 1)
gen set = substr(variable, 2, 2)
drop variable
spread var val
drop id
destring, replace 
```

3. pnl.csv

```stata
import delimited using pnl.csv, clear
gather y* z*
gen var = substr(variable, 1, 1)
gen timer = substr(variable, 2, 2)
spread var val
drop var
destring, replace 
```

## 长数据转换成宽数据

例如 fish_encounters 数据：

```r
read_csv('fish_encounters.csv') -> fish_encounters
fish_encounters

#> # A tibble: 114 x 3
#>     fish station  seen
#>    <dbl> <chr>   <dbl>
#>  1  4842 Release     1
#>  2  4842 I80_1       1
#>  3  4842 Lisbon      1
#>  4  4842 Rstr        1
#>  5  4842 Base_TD     1
#>  6  4842 BCE         1
#>  7  4842 BCW         1
#>  8  4842 BCE2        1
#>  9  4842 BCW2        1
#> 10  4842 MAE         1
#> # … with 104 more rows
```

使用 pivot_wider 可以很方便的将它转换成宽数据：

```r
fish_encounters %>% 
  pivot_wider(
    names_from = station, 
    values_from = seen,
    values_fill = 0
  )

#> # A tibble: 19 x 12
#>     fish Release I80_1 Lisbon  Rstr Base_TD   BCE   BCW  BCE2  BCW2   MAE   MAW
#>    <dbl>   <dbl> <dbl>  <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#>  1  4842       1     1      1     1       1     1     1     1     1     1     1
#>  2  4843       1     1      1     1       1     1     1     1     1     1     1
#>  3  4844       1     1      1     1       1     1     1     1     1     1     1
#>  4  4845       1     1      1     1       1     0     0     0     0     0     0
#>  5  4847       1     1      1     0       0     0     0     0     0     0     0
#>  6  4848       1     1      1     1       0     0     0     0     0     0     0
#>  7  4849       1     1      0     0       0     0     0     0     0     0     0
#>  8  4850       1     1      0     1       1     1     1     0     0     0     0
#>  9  4851       1     1      0     0       0     0     0     0     0     0     0
#> 10  4854       1     1      0     0       0     0     0     0     0     0     0
#> 11  4855       1     1      1     1       1     0     0     0     0     0     0
#> 12  4857       1     1      1     1       1     1     1     1     1     0     0
#> 13  4858       1     1      1     1       1     1     1     1     1     1     1
#> 14  4859       1     1      1     1       1     0     0     0     0     0     0
#> 15  4861       1     1      1     1       1     1     1     1     1     1     1
#> 16  4862       1     1      1     1       1     1     1     1     1     0     0
#> 17  4863       1     1      0     0       0     0     0     0     0     0     0
#> 18  4864       1     1      0     0       0     0     0     0     0     0     0
#> 19  4865       1     1      1     0       0     0     0     0     0     0     0
```

使用 spread 命令也可以：

```r
fish_encounters %>% 
  spread(station, seen, fill = 0)

#> # A tibble: 19 x 12
#>     fish Base_TD   BCE  BCE2   BCW  BCW2 I80_1 Lisbon   MAE   MAW Release  Rstr
#>    <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>  <dbl> <dbl> <dbl>   <dbl> <dbl>
#>  1  4842       1     1     1     1     1     1      1     1     1       1     1
#>  2  4843       1     1     1     1     1     1      1     1     1       1     1
#>  3  4844       1     1     1     1     1     1      1     1     1       1     1
#>  4  4845       1     0     0     0     0     1      1     0     0       1     1
#>  5  4847       0     0     0     0     0     1      1     0     0       1     0
#>  6  4848       0     0     0     0     0     1      1     0     0       1     1
#>  7  4849       0     0     0     0     0     1      0     0     0       1     0
#>  8  4850       1     1     0     1     0     1      0     0     0       1     1
#>  9  4851       0     0     0     0     0     1      0     0     0       1     0
#> 10  4854       0     0     0     0     0     1      0     0     0       1     0
#> 11  4855       1     0     0     0     0     1      1     0     0       1     1
#> 12  4857       1     1     1     1     1     1      1     0     0       1     1
#> 13  4858       1     1     1     1     1     1      1     1     1       1     1
#> 14  4859       1     0     0     0     0     1      1     0     0       1     1
#> 15  4861       1     1     1     1     1     1      1     1     1       1     1
#> 16  4862       1     1     1     1     1     1      1     0     0       1     1
#> 17  4863       0     0     0     0     0     1      0     0     0       1     0
#> 18  4864       0     0     0     0     0     1      0     0     0       1     0
#> 19  4865       0     0     0     0     0     1      1     0     0       1     0
```

使用 Stata 也可以轻松完成上面的操作：

```stata
import delimited using fish_encounters.csv, clear
spread station seen
mvencode BCE - Rstr, mv(. = 0)
```

### 数据透视：加总

pivot_wider 可以在长宽转换的同时还可以进行简单的加总运行，例如：

```r
read_csv('warpbreaks.csv') -> warpbreaks
warpbreaks

#> # A tibble: 54 x 3
#>    wool  tension breaks
#>    <chr> <chr>    <dbl>
#>  1 A     L           26
#>  2 A     L           30
#>  3 A     L           54
#>  4 A     L           25
#>  5 A     L           70
#>  6 A     L           52
#>  7 A     L           51
#>  8 A     L           26
#>  9 A     L           67
#> 10 A     M           18
#> # … with 44 more rows
```

生成数据透视表：

```r
warpbreaks %>% 
  pivot_wider(
    names_from = wool, 
    values_from = breaks,
    values_fn = list(breaks = mean)
  )
#> # A tibble: 3 x 3
#>   tension     A     B
#>   <chr>   <dbl> <dbl>
#> 1 L        44.6  28.2
#> 2 M        24    28.8
#> 3 H        24.6  18.8
```

这样我们就可以看到每种 wool 和 tension 的 breaks 的均值了。

使用 summarise 函数和 spread 函数也可以实现：

```r
warpbreaks %>% 
  group_by(wool, tension) %>% 
  summarise(breaks = mean(breaks, na.rm = T)) %>% 
  spread(wool, breaks)
#> # A tibble: 3 x 3
#>   tension     A     B
#>   <chr>   <dbl> <dbl>
#> 1 H        24.6  18.8
#> 2 L        44.6  28.2
#> 3 M        24    28.8
```

#### 使用 Stata 完成

```stata
import delimited using warpbreaks.csv, clear
collapse breaks, by(wool tension)
spread wool breaks
```

### 从多个变量生成列名

例如这个数据：

```r
read_csv('production.csv') -> production
production

#> # A tibble: 45 x 4
#>    product country  year production
#>    <chr>   <chr>   <dbl>      <dbl>
#>  1 A       AI       2000     -0.184
#>  2 A       AI       2001     -0.210
#>  3 A       AI       2002      1.37 
#>  4 A       AI       2003     -2.03 
#>  5 A       AI       2004      1.98 
#>  6 A       AI       2005     -2.32 
#>  7 A       AI       2006      0.292
#>  8 A       AI       2007     -0.352
#>  9 A       AI       2008     -1.37 
#> 10 A       AI       2009      0.524
#> # … with 35 more rows
```

我们想把 product 和 country 变量组合起来转换成宽数据：

```r
production %>% pivot_wider(
  names_from = c(product, country), 
  values_from = production
)
#> # A tibble: 15 x 4
#>     year    A_AI    B_AI    B_EI
#>    <dbl>   <dbl>   <dbl>   <dbl>
#>  1  2000 -0.184   1.54    0.158 
#>  2  2001 -0.210  -1.46   -2.09  
#>  3  2002  1.37   -1.24   -1.97  
#>  4  2003 -2.03   -0.0630  0.0431
#>  5  2004  1.98   -0.424   0.329 
#>  6  2005 -2.32   -1.55   -0.402 
#>  7  2006  0.292   1.95   -1.12  
#>  8  2007 -0.352  -1.00   -0.610 
#>  9  2008 -1.37    0.717   0.586 
#> 10  2009  0.524  -1.41   -0.0954
#> 11  2010 -0.0431 -0.376  -1.22  
#> 12  2011  0.0384 -0.869   0.121 
#> 13  2012  0.554   0.876  -0.166 
#> 14  2013  0.527  -0.641  -0.111 
#> 15  2014  0.482  -0.472   1.22
```

```r
production %>% pivot_wider(
  names_from = c(product, country), 
  values_from = production,
  names_sep = ".",
  names_prefix = "prod."
)
#> # A tibble: 15 x 4
#>     year prod.A.AI prod.B.AI prod.B.EI
#>    <dbl>     <dbl>     <dbl>     <dbl>
#>  1  2000   -0.184     1.54      0.158 
#>  2  2001   -0.210    -1.46     -2.09  
#>  3  2002    1.37     -1.24     -1.97  
#>  4  2003   -2.03     -0.0630    0.0431
#>  5  2004    1.98     -0.424     0.329 
#>  6  2005   -2.32     -1.55     -0.402 
#>  7  2006    0.292     1.95     -1.12  
#>  8  2007   -0.352    -1.00     -0.610 
#>  9  2008   -1.37      0.717     0.586 
#> 10  2009    0.524    -1.41     -0.0954
#> 11  2010   -0.0431   -0.376    -1.22  
#> 12  2011    0.0384   -0.869     0.121 
#> 13  2012    0.554     0.876    -0.166 
#> 14  2013    0.527    -0.641    -0.111 
#> 15  2014    0.482    -0.472     1.22
```

```r
production %>% pivot_wider(
  names_from = c(product, country), 
  values_from = production,
  names_glue = "prod_{product}_{country}"
)
#> # A tibble: 15 x 4
#>     year prod_A_AI prod_B_AI prod_B_EI
#>    <dbl>     <dbl>     <dbl>     <dbl>
#>  1  2000   -0.184     1.54      0.158 
#>  2  2001   -0.210    -1.46     -2.09  
#>  3  2002    1.37     -1.24     -1.97  
#>  4  2003   -2.03     -0.0630    0.0431
#>  5  2004    1.98     -0.424     0.329 
#>  6  2005   -2.32     -1.55     -0.402 
#>  7  2006    0.292     1.95     -1.12  
#>  8  2007   -0.352    -1.00     -0.610 
#>  9  2008   -1.37      0.717     0.586 
#> 10  2009    0.524    -1.41     -0.0954
#> 11  2010   -0.0431   -0.376    -1.22  
#> 12  2011    0.0384   -0.869     0.121 
#> 13  2012    0.554     0.876    -0.166 
#> 14  2013    0.527    -0.641    -0.111 
#> 15  2014    0.482    -0.472     1.22
```

使用 spread 函数也可以：

```r
production %>% 
  unite("var", product:country, sep = "_", remove = T) %>% 
  spread(var, production)
#> # A tibble: 15 x 4
#>     year    A_AI    B_AI    B_EI
#>    <dbl>   <dbl>   <dbl>   <dbl>
#>  1  2000 -0.184   1.54    0.158 
#>  2  2001 -0.210  -1.46   -2.09  
#>  3  2002  1.37   -1.24   -1.97  
#>  4  2003 -2.03   -0.0630  0.0431
#>  5  2004  1.98   -0.424   0.329 
#>  6  2005 -2.32   -1.55   -0.402 
#>  7  2006  0.292   1.95   -1.12  
#>  8  2007 -0.352  -1.00   -0.610 
#>  9  2008 -1.37    0.717   0.586 
#> 10  2009  0.524  -1.41   -0.0954
#> 11  2010 -0.0431 -0.376  -1.22  
#> 12  2011  0.0384 -0.869   0.121 
#> 13  2012  0.554   0.876  -0.166 
#> 14  2013  0.527  -0.641  -0.111 
#> 15  2014  0.482  -0.472   1.22
```

### 使用 Stata

```stata
import delimited using production.csv, clear
unite product country, gen(var) sep(_)
drop product country
spread var production
```

------------

<h4 align="center">

Code of Conduct

</h4>

<h6 align="center">

Please note that this project is released with a [Contributor Code of
Conduct](CODE_OF_CONDUCT.md).<br>By participating in this project you
agree to abide by its terms.

</h6>

<h4 align="center">

License

</h4>

<h6 align="center">

MIT © 微信公众号 RStata

</h6>
