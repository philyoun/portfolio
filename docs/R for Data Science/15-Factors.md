15 Factors
==========

15.1 Introduction
-----------------

R에서, factors는 categorical 변수들을 작업하기 위해서 쓰인다.  이 변수들이란? fixed, known set of possible values. 가능한 값들이 미리 정해져있고, 알려져 있는 것 또한, character vectors를 non-alphabetical order로 표현하고 싶을 때 유용하다.

역사적으로, factors가 characters보다 더 작업하기 쉽다. 그래서 base R의 많은 함수들은, characters를 자동적으로 factors로 바꾼다. 이 말인즉슨, factors가 불쑥 튀어나와 별 도움이 안 되는 경우가 있다. 그런 일은 tidyverse에서 일어나지 않는다. 그냥 factors가 정말로 유용한 상황에만 집중하면 된다.

15.1.1 Prerequisites
--------------------

factors를 다루기 위해서, forcats 패키지를 사용할 것이다. categorical variables을 다루는데 필요한 도구들을 제공해줌. **for cat**egorical이랍시고 이렇게 패키지 이름을 만들어 놓은듯.

factors를 작업할 수 있는 넓은 범위의 helper들을 제공해준다. forcats는 tidyverse에 포함이 안 되어 있기 때문에, 따로 로드를 해줘야 한다.

``` r
library(tidyverse)
library(forcats)
```

### 15.1.2 Learning more

factors에 대해 더 배우고 싶다면, Amelia McNamara와 Nicholas Horton가 쓴 [Wrangling categorical data in R](https://peerj.com/preprints/3163/)을 참고할 것. 뭐 역사적인 것들도 설명해주고, base R 방법들이랑 tidyverse간에 비교도 해주고. 이 문서의 초기 버전은 forcats 패키지를 개발하고 발전시키는데 도움을 줬다. 고맙댄다! \* \* \* \#\# 15.2 Creating factors 다음과 같이 month를 기록한 변수variable가 있다고 치자.

``` r
x1 <- c("Dec", "Apr", "Jan", "Mar")
```

이렇게 저장하는데 있어 string을 사용하면, 2개의 문제가 생긴다. 1) 12개의 가능한 month들이 있는데, 만약에 오타가 나면 아무것도 널 지켜주지 않는다.

``` r
x2 <- c("Dec", "Apr", "Jam", "Mar")
```

1.  그리고 유용하도록 sort할 수가 없다.

``` r
sort(x1)
## [1] "Apr" "Dec" "Jan" "Mar"
```

이러한 문제를, factor로 해결할 수 있다. factor를 만들기 위해서는, '가능한 levels의 리스트'를 만드는 것에서부터 시작해야 한다.

``` r
month_levels <- c("Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec")
```

이렇게 가능한 levels의 리스트...라기보단 벡터인데, 이걸 만들고,

이제 factor를 만들 수 있다.

``` r
y1 <- factor(x1, levels = month_levels)
y1
## [1] Dec Apr Jan Mar
## Levels: Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
sort(y1)
## [1] Jan Mar Apr Dec
## Levels: Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
```

그럼 위의 2가지 문제들이 어떻게 해결되는지를 보자. 1) 오타가 난다면? set에 없는 값들은 조용하게 `NA`로 변환된다. silently converted to `NA`

``` r
y2 <- factor(x2, levels = month_levels)
y2
## [1] Dec  Apr  <NA> Mar 
## Levels: Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
```

1.  유용하게 sort도 가능하다.

``` r
sort(y1)
## [1] Jan Mar Apr Dec
## Levels: Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
```

오타의 경우에 있어서 set에 없는 값들에 대해, warning을 받고 싶다면, `readr::parse_factor()`를 쓸 수 있다.

``` r
y2 <- parse_factor(x2, levels = month_levels)
## Warning: 1 parsing failure.
## row col           expected actual
##   3  -- value in level set    Jam
```

만약에 levels를 생략하면, levels를 알파벳 순으로, 데이터에서 취한다.

``` r
factor(x1)
## [1] Dec Apr Jan Mar
## Levels: Apr Dec Jan Mar
```

가끔, 나타나는 순으로 levels 순서를 지정하고 싶을 때가 있다. 이걸 factor를 만들 때, levels에다가 `unique(x)`로 설정해놓거나, 혹은, factor를 만든 이후에, `fct_inorder()`로 할 수도 있다.

``` r
f1 <- factor(x1, levels = unique(x1))
f1
## [1] Dec Apr Jan Mar
## Levels: Dec Apr Jan Mar

f2 <- x1 %>% factor() %>% fct_inorder()
f2
## [1] Dec Apr Jan Mar
## Levels: Dec Apr Jan Mar
```

만약 가능한 levels의 셋을 직접적으로 접근해야할 필요가 있다면, `levels()`를 사용하자.

``` r
levels(f2)
## [1] "Dec" "Apr" "Jan" "Mar"
```

------------------------------------------------------------------------

15.3 General Social Survey
--------------------------