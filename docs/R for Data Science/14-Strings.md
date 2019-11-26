14 Strings
================

14.1 Introduction
-----------------

이 chapter에서는, R에서 string 조작manipulation을 어떻게 해야할지 알려줄 것이다. <br /> 어떻게 strings가 작동하는지를 배우고, 어떻게 손으로 만들어낼 수 있는지도 배우지만, <br /> 이 chapter의 핵심은 regular expression, 줄여서 regexps라고 부르는 것이다.

Regexp는 유용하다. 왜냐하면, <br /> strings는 보통 구조가 없거나 좀 덜한(structured or semi-structured) 데이터를 가지고 있는데, <br />     regexp는 strings의 패턴을 묘사해주는, 정확한 언어이기 때문. <br /> 처음 regexp를 볼 때면, 고양이가 키보드 위를 걸어간 것 같겠지만, 이해하고 나면 말이 될 것이다.

(이후 Regular expression = regexp = 정규표현식 3단어 모두 혼용해서 씁니당)

### 14.1.1 Prerequisites

이 chapter에서는 string 조작을 위해, stringr 패키지를 사용할 것이다. <br /> 항상 텍스트 데이터를 다루는게 아니라서, stringr은 tidyverse의 핵심 부분이 아니기에, <br /> 따로 explicit하게 로드를 해줘야한다. <br /> (라고 되어 있지만 `library(tidyverse)`하면 stringr 패키지도 로드된다.)

``` r
library(tidyverse)
library(stringr)
```

14.2 String basics
------------------

작은 따옴표나 큰 따옴표 아무거나 써서 strings를 만들 수 있다. <br /> 다른 언어와는 다르게, 이 둘은 아무런 차이가 없다. <br /> 저자는 항상 `"`를 사용할 것을 추천한다. string이 여러 개의 `"`를 포함하고 있지 않는 이상. <br />

``` r
string1 <- "This is a string"
string2 <- 'If I want to include a "quote" inside a string, I use single quotes'
```

만약 따옴표 닫는걸 깜빡했다면, `+`라는 continuation character를 보게 될 것이다.

``` r
> "This is a string without a closing quote
+
+
+ HELP I'M STUCK
```

이런 일이 일어나면, Esc 키를(Escape 키라고 부르나 봄) 누르면 된다.

string에다가 진짜로 작은 따옴표나 큰 따옴표를 넣고 싶다면, 앞에다가 `\`를 사용해서 "escape"해야 한다.

``` r
double_quote <- "\"" # 아니면 '"'
single_quote <- '\'' # 아니면 "'"
```

이 말인즉슨, 만약에 진짜 백슬래쉬(`\`)를 넣고 싶다면, 얘도 escape해줘야 한다는 뜻. <br /> 즉, `"\\"`

프린트된 문자열 표현printed representation of string이, 그 자체로 string이 아니라는 걸 주의하자. <br /> 왜냐하면 프린트된 문자열 표현은 escapes를 보여주기 때문. <br /> string의 raw contents를 보고 싶다면, `writeLines()`를 사용해야한다.

``` r
x <- c("\"", "\\")
x
## [1] "\"" "\\"

writeLines(x)
## "
## \
```

특별한 캐릭터들이 좀 있다. <br /> 가장 흔한 걸로는 `"\n"`는 newline, `"\t"`는 tab. <br /> 완전한 리스트를 보고싶다면, `"`에 help를 해볼 것. `?'"'` 혹은 `?"'"` <br /> 가끔 `"\u00b5"`와 같은 string도 볼 수 있는데, 모든 플랫폼에서 작동하는 non-English 캐릭터를 쓰는 방법이다.

``` r
x <- "\u00b5"
x
## [1] "μ"
```

여러 개의 strings를, `c()`를 이용해 character vector에 저장할 수 있다.

``` r
c("one", "two", "three")
## [1] "one"   "two"   "three"
```
