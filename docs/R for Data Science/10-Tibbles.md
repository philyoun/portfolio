10 Tibbles
================

10.1 Introduction
-----------------

이 책 내내, R의 전통적인 `data.frame` 대신, tibbles를 쓸 것이다. <br /> tibbles는 data frames이긴 한데, 좀 낡은 부분을 개선한 것. <br /> R은 꽤 오래된 언어라, 10년 20년 전에는 쓸모있었던 것들이 이제는 방해가 된다. <br /> R의 이미 존재하는 코드를 깨는 건 좀 힘들다. <br /> 그래서 대부분의 혁신은 패키지를 이용해 이루어진다. <br /> 이 chapter에서는 **tibble** 패키지에 대해 소개할 것이다. <br /> 얘는 tidyverse와 작업을 하는데 있어, 좀 더 쉽게해주는 그러한 데이터 프레임을 공급해주는 패키지. <br /> 이 책 내내 tibble과 data frame라는 단어를 번갈아가며 사용할건데, <br />   R의 built-in 데이터 프레임을 지칭하고 싶을 때에는 `data.frame`이라고 부르겠다.

이 chapter가 알려주는 것 이상으로 좀 더 배우고싶다면, `vignette("tibble")`을 사용해라.

### 10.1.1 Prerequisites

tidyverse의 핵심부분인, **tibble** 패키지를 탐구해볼 것이다.

``` r
library(tidyverse)
```

------------------------------------------------------------------------

10.2 Creating tibbles
---------------------

이 책에서, 앞으로 니가 이용할 모든 함수는 tibbles를 만들 것이다. <br /> Almost all of the functions that you’ll use in this book produce tibbles <br /> 왜냐하면 tibbles가 tidyverse의 unifying features 중 하나이기 때문이다. <br /> 다른 거의 모든 R 패키지들은 원래의 data frames를 이용하기 때문에, tibble로 변환하고 싶다면, <br /> `as_tibble()`을 쓰자.

``` r
as_tibble(iris)
## # A tibble: 150 x 5
##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species
##           <dbl>       <dbl>        <dbl>       <dbl> <fct>  
##  1          5.1         3.5          1.4         0.2 setosa 
##  2          4.9         3            1.4         0.2 setosa 
##  3          4.7         3.2          1.3         0.2 setosa 
##  4          4.6         3.1          1.5         0.2 setosa 
##  5          5           3.6          1.4         0.2 setosa 
##  6          5.4         3.9          1.7         0.4 setosa 
##  7          4.6         3.4          1.4         0.3 setosa 
##  8          5           3.4          1.5         0.2 setosa 
##  9          4.4         2.9          1.4         0.2 setosa 
## 10          4.9         3.1          1.5         0.1 setosa 
## # ... with 140 more rows
```

개별적인 벡터들individual vectors로 새로운 tibble을 만들고 싶을 때는, `tibble()`을 쓰자. <br /> 길이 하나짜리의 input은, `tibble()`이 자동적으로 recycle한다. <br />   = 그러니깐 길이가 짧으면 반복해서 길게 되도록 만들어줌. <br /> 그래서 아래와 같이, 방금 만든 variables를 refer할 수 있다.

``` r
tibble(
  x = 1:5, 
  y = 1, 
  z = x ^ 2 + y
)
## # A tibble: 5 x 3
##       x     y     z
##   <int> <dbl> <dbl>
## 1     1     1     2
## 2     2     1     5
## 3     3     1    10
## 4     4     1    17
## 5     5     1    26
```

이미 `data.frame()`과 친근하다면, `tibble()`은 더 적은 일을 한다는 것을 인지하자. <br /> tibble은 절대 inputs의 타입을 바꾸지 않는다. <br />   예를 들어, strings을 절대 factors로 바꾸지 않는다! <br /> (필자는 이런 경험이 실제로 있다. strings가 지멋대로 factors로 바뀐 경험.) <br /> variables의 이름을 바꾸지도 않고, row names를 만들지도 않고.

tibble의 column name으로, 이상한 것도 쓸 수가 있다. <br /> R의 variable name으로는, 유효하지 않은 것(aka **non-syntactic**)으로도 tibble의 칼럼 이름으로 쓸 수 있다는 것.

예를 들어서, 문자로 시작하지 않을수도 있고, 공백과 같은 흔치않은 캐릭터를 가지고 있을수도 있다. <br /> 이런 걸 variable names로 만들 때는, \`\`\`로 감싸줘야 한다.

``` r
tb <- tibble(
  `:)` = "smile", 
  ` ` = "space",
  `2000` = "number"
)
tb
## # A tibble: 1 x 3
##   `:)`  ` `   `2000`
##   <chr> <chr> <chr> 
## 1 smile space number
```

이런 변수들을 다룰 때에는, 항상 \`\`로 묶어줘야함.

다른 방법으로 tibble을 만드는 방법으로 `tribble()`이 있다. 이건 **t**ransposed tibble의 약어로, <br /> 그러니깐 보일 모습 그대로 tibble을 만든다고 생각하면 된다. 이건 대신 변수 앞에 `~`을 붙여야 됨. <br /> 양이 적은 데이터에 대해, 읽는 그대로를 tibble로 만들어준다.

``` r
tribble(
  ~x, ~y, ~z,
  #--|--|----
  "a", 2, 3.6,
  "b", 1, 8.5
)
## # A tibble: 2 x 3
##   x         y     z
##   <chr> <dbl> <dbl>
## 1 a         2   3.6
## 2 b         1   8.5
```

`#--|--|----`이 부분을 넣어줌으로써 header가 어디인지 명시해준 것. <br /> 필요없다. -가 여러개 있어도 됨.

------------------------------------------------------------------------

10.3 Tibbles vs. Data.frame
---------------------------

tibble과 전통적인 `data.frame` 사이엔 2가지 주요한 차이점이 있다. printing과 subsetting

### 10.3.1 Printing

Tibbles는 정제된refined print method를 가지고 있다. 제한적인 프린트를 해준다는 뜻. <br /> 즉, 딱 첫 10개의 줄만 보여주고, 칼럼들의 경우에는 스크린에 딱 맞게끔만 보여준다. <br /> large data에 대해서는 이게 더 일하기 쉽다. 이름과 함께, 그 열의 타입도 알려준다. <br /> `str()`로부터 빌려온 좋은 특성feature.

``` r
tibble(
    a = lubridate::now() + runif(1e3) * 86400,
    b = lubridate::today() +runif(1e3) * 30,
    c = 1:1e3,
    d = runif(1e3),
    e = sample(letters, 1e3, replace = TRUE)
)
## # A tibble: 1,000 x 5
##    a                   b              c     d e    
##    <dttm>              <date>     <int> <dbl> <chr>
##  1 2019-12-24 22:27:20 2019-12-29     1 0.761 w    
##  2 2019-12-24 10:47:35 2020-01-16     2 0.631 o    
##  3 2019-12-25 02:09:13 2019-12-25     3 0.352 n    
##  4 2019-12-24 19:15:28 2020-01-02     4 0.961 h    
##  5 2019-12-24 16:51:10 2019-12-25     5 0.552 i    
##  6 2019-12-25 06:25:14 2020-01-06     6 0.223 m    
##  7 2019-12-24 09:30:19 2019-12-31     7 0.106 t    
##  8 2019-12-25 02:03:10 2020-01-18     8 0.655 n    
##  9 2019-12-25 07:44:15 2020-01-02     9 0.150 e    
## 10 2019-12-24 21:11:05 2020-01-08    10 0.367 n    
## # ... with 990 more rows
```

그래서, 우연하게 엄청 큰 데이터를 print하더라도 너의 콘솔이 다 채워지지 않게끔 design되었다. <br /> 하지만 가끔씩은 디폴트보다 더 많은 output을 원할 수 있다.

그럴 때 쓰는 방법으로,

1.  explict하게 `print()`를 쓰고, number of rows(`n`)이랑 display of the `width`를 정해라. <br /> `width = Inf`면 모든 열들을 다 보여줄 것이다.

``` r
nycflights13::flights %>% 
  print(n = 10, width = Inf)
```

1.  아예 디폴트 print 값을 바꿀 수도 있다. <br />   1) 행과 관련해서 <br /> `options(tibble.print_max = n, tibble.print_min = m)` :이라고 쓰면, <br />   만약 `n`보다 많은 rows가 있다면, 오직 `m`개의 rows만 print해라. <br /> 항상 모든 rows를 다 프린트하길 원한다면, `options(tibble.print_min = Inf)`

  2) 열(칼럼)과 관련해서 <br /> screen 크기와 상관없이 모든 columns를 보고 싶다면, `options(tibble.width = Inf)`를 쓰자.

options들의 complete list를 보고 싶다면 `package?tibble`을 쓸 것

1.  마지막 방법은 RStudio의 만들어져있는 scrollable view를 사용해보자.

``` r
nycflights13::flights %>% 
  View()
```

### 10.3.2 Subsetting

이제껏 니가 배워왔던 도구들은 완전한 데이터 프레임과 작동해왔다. <br /> single variable을 끄집어내고 싶다면 새로운 도구가 필요하다. `$`나 `[[`. <br /> `[[`는 이름이나 position으로 가능, `$`는 이름으로만 가능하지만 타이핑을 조금 덜 친다.

``` r
df <- tibble(
    x = runif(5),
    y = rnorm(5)
)

# 이름으로 추출할 때,
df$x
## [1] 0.522401891 0.718621119 0.002673199 0.330507813 0.317680057
df[["x"]]
## [1] 0.522401891 0.718621119 0.002673199 0.330507813 0.317680057

# 포지션으로 추출할 때,
df[[1]]
## [1] 0.522401891 0.718621119 0.002673199 0.330507813 0.317680057
```

pipe 연산자를 이용해서도 할 수 있다. 대신 앞에 `.`을 붙여야한다.

``` r
df %>% .$x
## [1] 0.522401891 0.718621119 0.002673199 0.330507813 0.317680057
df %>% .[["x"]]
## [1] 0.522401891 0.718621119 0.002673199 0.330507813 0.317680057
```

데이터 프레임과 비교해서 tibbles가 좀 더 strict하다. <br /> tibble은 절대 partial matching을 하지 않고, 존재하지 않는 칼럼에 대해 access하려고 하면 warning이 뜬다. <br /> partial matching은 merge할 때 나오는 문제. 없으면 NA로 채우는데, 그것도 금지하나봄. <br /> <https://www.guru99.com/r-merge-data-frames.html#2>를 참고해보자.

10.4 Interacting with older code
--------------------------------

몇 가지 오래된 함수들은 tibbles와 작동하지 않는다. <br /> 이럴 때는, `as.data.frame()`을 사용해, tibble을 데이터 프레임으로 되돌려라.

``` r
class(as.data.frame(tb))
## [1] "data.frame"
```

오래된 함수가 tibble이랑 호환되지 않는 주요한 이유는 `[` 함수 때문이다. <br /> 이 책에서 `[`를 많이 쓰지 않는데, 왜냐하면 `dplyr::filter()`나 `dplyr::select()`가 더 명백한 코드를 주기 때문. <br /> (하지만 [vector subsetting]()에서 배우긴 할 것이다.) <br /> 기본 R 데이터 프레임으로 작업하면, `[`가 가끔은 data frame을 반환하고 가끔은 vector를 반환한다. <br /> 하지만 tibbles를 쓰면, 항상 또 다른 tibble을 반환한다.

10.5 Exercises
--------------

1.  어떻게 object가 tibble인지 아닌지를 알 수 있을까?
