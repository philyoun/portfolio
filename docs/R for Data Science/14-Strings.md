14 Strings
================

14.1 Introduction
-----------------

이 chapter에서는, R에서 string 조작manipulation을 어떻게 해야할지 알려줄 것이다. <br /> 어떻게 strings가 작동하는지를 배우고, 어떻게 손으로 만들어낼 수 있는지도 배우지만, <br /> 이 chapter의 핵심은 regular expression, 줄여서 regexps라고 부르는 것이다.

Regexp는 유용하다. 왜냐하면, <br /> strings는 보통 구조가 없거나 좀 덜한(structured or semi-structured) 데이터를 가지고 있는데, <br />     regexp는 strings의 패턴을 묘사해주는, 간결한concise 언어이기 때문. <br /> 처음 regexp를 볼 때면, 고양이가 키보드 위를 걸어간 것 같겠지만, 이해하고 나면 말이 될 것이다.

(이 글에서는, Regular expression = regexp = 정규표현식 3단어 모두 혼용해서 씁니당)

### 14.1.1 Prerequisites

이 chapter에서는 string 조작을 위해, stringr 패키지를 사용할 것이다. <br /> 항상 텍스트 데이터를 다루는게 아니라서, stringr은 tidyverse의 핵심 부분이 아니기에, <br /> 따로 explicit하게 로드를 해줘야한다. <br /> (라고 되어 있지만 `library(tidyverse)`하면 stringr 패키지도 로드된다.)

``` r
library(tidyverse)
library(stringr)
```

------------------------------------------------------------------------

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

### 14.2.1 String Length

base R에는 strings를 작업할 수 있는 여러가지 함수들이 있다. <br /> 근데 이것들은 일관적이지 않아서inconsistent 기억을 하기가 힘들다. <br /> 그렇기 때문에 사용하지 않을거다.

대신에 stringr에 있는 함수들을 쓸 것이다. <br /> 이 함수들은 전부다 str\_로 시작하기 때문에 직관적인 이름들을 가지고 있다. <br /> 예를 들어, `str_length()`는 해당 string의 길이가 얼마나 되는지를 알려준다.

``` r
str_length(c("a", "R for data science", NA))
## [1]  1 18 NA
```

RStudio에서 작업을 하면 더 편리하다. <br /> `str_`라고 치기만 하면 자동완성autocomplete이 되기 때문에, 다른 stringr 함수들을 볼 수가 있다. <br /> ![그림1](https://d33wubrfki0l68.cloudfront.net/7d1defbecac1e73595c3841f2753a09734dcb0be/0b58f/screenshots/stringr-autocomplete.png)

### 14.2.2 Combining strings

2개 이상의 string들을 결합하고 싶다면, `str_c()`를 사용하자.

``` r
str_c("x", "y")
## [1] "xy"
str_c("x", "y", "z")
## [1] "xyz"
```

`sep` 인자argument를 사용해서 어떻게 분리를 할지 컨트롤할 수 있다.

``` r
str_c("x", "y", sep = ", ")
## [1] "x, y"
```

R의 다른 대부분의 함수들과 마찬가지로, 결측값missing values는 위험할 수 있다. <br /> 그냥 `"NA"` 그대로 프린트하고 싶다면, `str_replace_na()`을 사용하자.

``` r
x <- c("abc", NA)
str_c("|-", x, "-|")
## [1] "|-abc-|" NA
str_c("|-", str_replace_na(x), "-|")
## [1] "|-abc-|" "|-NA-|"
```

위에서 볼 수 있듯, `str_c()`는 벡터화vectorized되고, 길이가 안 맞는게 있다면 recycle을 해서 맞춘다.

``` r
str_c("prefix-", c("a", "b", "c"), "-suffix")
## [1] "prefix-a-suffix" "prefix-b-suffix" "prefix-c-suffix"
```

길이 0짜리 오브젝트들은 조용하게 드랍된다. Objects of length 0 are silently dropped. <br /> 이건 `if` 문과 함께 쓸 때 특히나 유용하다.

``` r
name <- "Hadley"
time_of_day <- "morning"
birthday <- FALSE

str_c(
    "Good ", time_of_day, " ", name, 
    if (birthday) " and HAPPY BIRTHDAY",
    "."
)
## [1] "Good morning Hadley."
```

strings의 벡터를 하나로 축소collapse하고 싶다면, `collapse` 인자argument를 사용하자.

``` r
str_c(c("x", "y", "z"), collapse = ", ")
## [1] "x, y, z"
```

### 14.2.3 Subsetting strings

`str_sub()`를 사용해서, string의 일부를 추출해낼 수 있다. <br /> string과 마찬가지로, `str_sub()`도 substring의 포지션을 알려주는, `start`와 `end` 인자들arguments을 받는다.

``` r
x <- c("Apple", "Banana", "Pear")
str_sub(x, 1, 3)
## [1] "App" "Ban" "Pea"

str_sub(x, -3, -1)
## [1] "ple" "ana" "ear"
```

`str_sub()`는 string이 너무 짧아도 상관없다는 걸 인지하시라. <br /> 가능한만큼만 return한다.

``` r
str_sub("a", 1, 5)
## [1] "a"
```

strings를 수정하기 위해서, `str_sub()`를 assignment form으로 써줄 수 있다. <br /> 위의 "Apple", "Banana", "Pear"의 첫 글자를 소문자로 수정하고 싶다고 치자.

``` r
str_sub(x, 1, 1) <- str_to_lower(str_sub(x, 1, 1))
x
## [1] "apple"  "banana" "pear"
```

이렇게도 수정을 할 수가 있다는 말.

### 14.2.4 Locales

위에서 텍스트를 소문자로 바꿀 때 `str_to_lower()`을 사용했다. <br /> 마찬가지로 `str_to_upper()`와 `str_to_title()`도 있다. <br /> 하지만, 이렇게 바꾸는건 생각보다 복잡하다. <br /> 왜냐하면 각 언어마다 바꾸는게 좀 다른 룰을 가지고 있기 때문. <br /> 그럴 때, locale을 정해줌으로써 룰을 정할 수 있다.

``` r
# 터키는 2개의 i를 갖고 있다. 점이 있는 것과 없는 것. 그래서 대문자화하면 다르게 된다.
str_to_upper(c("i", "ı"))
## [1] "I" "I"
str_to_upper(c("i", "ı"), locale = "tr")
## [1] "<U+0130>" "I"
```

locale은 ISO 639 언어코드로 정해져있다. 언어 코드는 2, 3개의 줄임말로 되어있다는거. <br /> tr, kr 등등.. 어떤 언어의 코드를 모르겠다면, [Wikipedia](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)에 리스트가 나와 있다. <br /> locale 부분을 비워두면, operating system에 있는 current locale대로 사용한다. (우리나라를 찾아보니 kr이 아니고 ko로 쓰네.)

locale에 영향을 받는 또다른 중요한 operation은 sorting이다. <br /> base R의 `order()`과 `sort()` 함수들은 현재 locale을 이용해서 strings를 정렬sort한다. <br /> 다른 컴퓨터들간에도 변함없는 결과를 얻고 싶다면robust behavior, <br />     `str_sort()`와 `str_order()`을 사용해서 `locale` 인자argument를 컨트롤해라.

``` r
x <- c("apple", "eggplant", "banana")

str_sort(x, locale = "en")
## [1] "apple"    "banana"   "eggplant"

str_sort(x, locale = "haw")
## [1] "apple"    "eggplant" "banana"
```

### 14.2.5 Exercises

------------------------------------------------------------------------

14.3 Matching patterns with regular expressions
-----------------------------------------------

Regexps(이하 정규표현식)는 string에 있는 패턴을 묘사할 수 있도록 해주는, 매우 간결한terse 언어다. <br /> 처음엔 낯설지만, 이해하고 나면 매우 유용하다.

regular expressions(이하 정규표현식)를 배우기 위해서, `str_view()`와 `str_view_all()`를 사용할거다. <br /> 이 함수들은 character vector와 정규표현식을 받고, 어떻게 매치가 되는지를 보여줄거다.

매우 간단한 정규표현식에서 시작해서, 점점 더 복잡해져갈 것이다. <br /> 이 패턴 매칭을 마스터하고 나면, 다양한 stringr 함수들에 어떻게 적용을 해야하는지 알 수 있다.

### 14.3.1 Basic matches

가장 간단한 패턴 매칭은 exact strings다.

``` r
x <- c("apple", "banana", "pear")
str_view(x, "an")
```

![14.3.1-1](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.1-1.png?raw=true)

복잡함을 한 단계 올려서, `.`이라는건 any character라는 뜻이다.

``` r
str_view(x, ".a.")
```

![14.3.1-2](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.1-2.png?raw=true)

하지만 만약 "`.`"이 any character을 매치시켜주는거라 할 때, 진짜 `.`을 매치하고 싶다면? <br /> escape을 사용하면 된다. <br /> strings와 마찬가지로, 정규표현식도 특별한 행동을 escape하기 위해, `\`라는 백슬래쉬를 쓴다.

그래서 `.`을 매치하고 싶다면, `\.`을 사용하면 된다. 하지만 이러면 문제가 생긴다. <br /> 정규표현식을 표현하기 위해서 strings를 사용했는데, `\`는 strings에서 symbol을 escape하는데도 사용. <br /> 그래서 `\.`라는 정규표현식을 만들고 싶다면, `\\.`을 사용.

``` r
dot <- "\\."

writeLines(dot)
## \.
```

``` r
str_view(c("abc", "a.c", "bef"), "a\\.c")
```

![14.3.1-3](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.1-3.png?raw=true)

진짜 `\`를 매치하고 싶다면? escape를 해야하니깐, `\\`라는 정규표현식을 만들어야됨. <br /> 그런데 string을 이용해서 정규표현식을 만들어야하니깐 `\\\` <br /> 근데 이걸 또 escape해야하니깐 하나 더. `\\\\`

그래서 `\` 하나 매치시키기 위해서, `"\\\\"`, 즉 4번의 백슬래쉬가 필요하다는 것.

``` r
x <- "a\\b"
writeLines(x)
## a\b
```

``` r
str_view(x, "\\\\")
```

![14.3.1-4](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.1-4.png?raw=true)

여기서는, 정규표현식은 `\.`로, 이 정규표현식을 나타내는 strings를 `"\\."`로 쓸 것이다.

연습문제까지 다 해보진 않는데, 여기 chapter에서는 좀 해봐야될 것 같다.

<br /> <br /> <br />

<details> <summary>14.3.1.1 Exercises</summary> 1. 왜 다음의 각각은 `\`를 match하지 못하는지를 설명해봐라. : `"\"`, `"\\"`, `"\\\"`

1.  `"'\`를 어떻게 match할 건지?

``` r
x <- "asdf\"'\\"
writeLines(x)
str_view(x, "\"\'\\\\")
# str_view(x, "\"'\\\\") 이것도 되네
```

![14.3.1-5](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.1-5.png?raw=true)

1.  어떤 패턴이 `\..\..\..`라는 정규표현식에 match가 될지? 이걸 어떻게 string으로 표현할건지? <br /> (진짜 .) + any character + (진짜 .) + any character + (진짜 .) + any character 같은 정규표현식을 match가능.

string으로는 `"\\..\\..\\.."` 이렇게 써야겠지?

``` r
str_view(".q.w.e", "\\..", "\\..\\..\\..")
```

![14.3.1-6](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.1-6.png?raw=true)

</details>

<br /> <br /> <br /> <br />

### 14.3.2 Anchors

디폴트로, 정규표현식은 string의 어떠한 부분이나 매치할 수 있다. <br /> string의 start나 end를 매치시키도록 고정anchor할 수 있다. <br /> - `^` : string의 start를 매치 - `$` : string의 end를 매치

``` r
x <- c("apple", "banana", "pear")
str_view(x, "^a")
```

![14.3.2-1](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-1.png?raw=true)

``` r
str_view(x, "a$")
```

![14.3.2-2](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-2.png?raw=true)

뭐가 뭔지 기억하기 쉽도록, try this mnemonic: if you begin with power(`^`), you end up with money(`$`). <br /> (...이게 왜 기억하기 쉽단건지 이해불가;)

오직 complete string만을 매치하도록 정규표현식을 강제할 수 있는데, `^`랑 `$`를 둘 다 이용해 고정anchor할 수 있다.

``` r
x <- c("apple pie", "apple", "apple cake")
str_view(x, "apple")
```

![14.3.2-3](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-3.png?raw=true)

``` r
str_view(x, "^apple$")
```

![14.3.2-4](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-4.png?raw=true)

`\b`로 boundary 매치를 시킬수도 있다. <br /> 나는(Hadley는) R에서 이걸 자주 사용하지는 않는데, RStudio에서는 검색할 때 가끔 사용한다. <br /> 예를 들어, `\bsum\b`를 사용해서, `summarise`, `summary`, `rowsum` 등등이 검색되는걸 피할 수 있다.

<br /> <br /> <br />

<details> <summary>14.3.2.1 Exercises</summary> 1. `"$^$"` 이 자체를 match하고 싶다면? string 사이에 `$^$`가 있을수도 있는건데, 딱 이것만을 match하고 싶다면.

``` r
x <- c("$^$", "ab$^$sfas")
str_view(x, "^\\$\\^\\$$")
```

![14.3.2-5](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-5.png?raw=true)

1.  `stringr::words`에는 단어들이 잔뜩 있다. 다음의 각 조건을 맞는 단어들을 찾아보라. 1번. "y"로 시작,

``` r
str_view(words, "^y", match = TRUE)
```

![14.3.2-6](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-6.png?raw=true)

2번. "x"로 끝

``` r
str_view(words, "x$", match = TRUE)
```

![14.3.2-7](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-7.png?raw=true)

3번. 정확하게 3개의 알파벳으로 구성(`str_length()` 쓰지 말고!)

``` r
str_view(words, "^...$", match = TRUE)
```

4번. 7개 이상의 알파벳들로 구성

``` r
str_view(words, ".......", match = TRUE)
```

3번, 4번은 너무 길어서 결과를 포함하지 않겠다.

해당이 되는 것들만 출력이 되길 원한다면, 위와 같이 `match = TRUE` 옵션을 넣으면 된다. </details>

<br /> <br /> <br />

### 14.3.3 Character classes and alternatives

### 14.3.4 Repetition

### 14.3.5 Grouping and backreferences