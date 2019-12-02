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

locale은 ISO 639 언어코드로 정해져있다. 언어 코드는 2, 3개의 줄임말로 되어있다는거. <br /> tr, kr 등등.. 어떤 언어의 코드를 모르겠다면, [Wikipedia](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)에 리스트가 나와 있다. <br /> locale 부분을 비워두면, operating system에 있는 current locale대로 사용한다. <br /> (우리나라를 찾아보니 kr이 아니고 ko로 쓰네.)

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

#### 14.3.1.1 Exercises

1.  왜 다음의 각각은 `\`를 match하지 못하는지 생각해보자. : `"\"`, `"\\"`, `"\\\"` <br /> <br />
2.  `"'\`를 어떻게 match할 수 있을지? <br /> <br />
3.  어떤 패턴이 `\..\..\..`라는 정규표현식에 match가 될지? 그리고 이걸 어떻게 string으로 표현할건지?

<details> <summary>14.3.1.1 Exercises sol</summary> 1. <br /> <br /> 2.

``` r
x <- "asdf\"'\\"
writeLines(x)
## asdf"'\
```

``` r
str_view(x, "\"\'\\\\")
# str_view(x, "\"'\\\\") 이것도 되네
```

<img src="https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.1-5.png?raw=true" alt="14.3.1-5"> <br /> <br /> 3. (진짜 .) + any character + (진짜 .) + any character + (진짜 .) + any character 같은 정규표현식을 match가능. <br /> <br /> string으로는 <code>"\\..\\..\\.."</code> 이렇게 써야겠지? <br />

``` r
str_view(".a.b.c", "\\..\\..\\..")
```

<img src="https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.1-6.png?raw=true" alt="14.3.1-6"> <br /> </details>

<br /> <br /> <br /> <br />

### 14.3.2 Anchors

디폴트로, 정규표현식은 string의 어떠한 부분이나 매치할 수 있다. <br /> string의 start나 end를 매치시키도록 고정anchor할 수 있다. <br /> - `^` : string의 start를 매치 <br /> - `$` : string의 end를 매치

``` r
x <- c("apple", "banana", "pear")
str_view(x, "^a")
```

![14.3.2-1](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-1.png?raw=true)

``` r
str_view(x, "a$")
```

![14.3.2-2](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-2.png?raw=true)

뭐가 뭔지 기억하기 쉽도록, <br /> try this mnemonic: if you begin with power(`^`), you end up with money(`$`). <br /> (...이게 왜 기억하기 쉽단건지 이해불가;;)

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

#### 14.3.2.1 Exercises

1.  `"$^$"` 이 자체를 match하고 싶다면? string 사이에 <code>$^$</code>가 있을수도 있는건데, 그게 아니라 딱 이것만을 match하고 싶다면 string으로 어떻게 써야할까? <br /> <br />
2.  `stringr::words`에는 단어들이 잔뜩 있다. 다음의 각 조건을 맞는 단어들을 찾아보라. <br />    1번. "y"로 시작하는 단어들? <br />    2번. "x"로 끝나는 단어들? <br />    3번. 정확하게 3개의 알파벳으로 구성(`str_length()` 쓰지 말고!)된 단어들? <br />    4번. 7개 이상의 알파벳들로 구성된 단어들? <br /> 해당되는 단어들은 많기 때문에, `str_view()`에 `match` argument를 사용해서 매칭되는 단어들만 표시하도록 할 수 있다.

<details> <summary>14.3.2.1 Exercises sol</summary> 1.

``` r
x <- c("$^$", "ab$^$sfas")
str_view(x, "^\\$\\^\\$$")
```

<img src="https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-5.png?raw=true" alt="14.3.2-5"> <br /> <br /> 2. <br /> 1번.

``` r
str_view(words, "^y", match = TRUE)
```

<img src="https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-6.png?raw=true" alt="14.3.2-6">

2번.

``` r
str_view(words, "x$", match = TRUE)
```

<img src="https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.2-7.png?raw=true" alt="14.3.2-7">

3번.

``` r
str_view(words, "^...$", match = TRUE)
```

4번.

``` r
str_view(words, ".......", match = TRUE)
```

3번, 4번은 너무 길어서 결과는 생략. </details> <br /> <br />

### 14.3.3 Character classes and alternatives

딱 하나만의 캐릭터를 match시켜주는게 아니라, 여러 개의 캐릭터를 match시켜주는, 특별한 패턴들이 많다. <br /> 이미 `.`를 봤는데, 새로운 라인에 있지 않은 any character들을 매치시켜줌. <br /> 4개의 다른 유용한 툴들이 있다.

-   `\d`: 어떠한 숫자를 매치시켜줌
-   `\s`: 어떠한 공백이나 매치시켜줌(띄어쓰기, 탭, 새로운 라인)
-   `[abc]`: a, b, 혹은 c를 매치
-   `[^abc]`: a, b, c를 제외한 아무거나 매치

`\d`, `\s`를 포함한 정규표현식을 만들기 위해서는, string에서 `\`를 escape해야되므로, `"\\d"`, `"\\s"`로 타이핑해야된다는 것을 꼭 기억.

특히 저 3번째 저건, escape를 쓰지 않고 하나의 metacharacter를 정규표현식에 넣을 수 있도록 해준다. <br /> 이걸 character class라고 부른다. <br /> 그리고 많은 사람들이 이게 더 읽기 쉽다고 느낀다. 다음의 예를 보자.

``` r
str_view(c("abc", "a.c", "a*c", "a c"), "a[.]c")
```

![14.3.3-1](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.3-1.png?raw=true)

``` r
str_view(c("abc", "a.c", "a*c", "a c"), ".[*]c")
```

![14.3.3-2](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.3-2.png?raw=true)

``` r
str_view(c("abc", "a.c", "a*c", "a c"), "a[ ]")
```

![14.3.3-3](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.3-3.png?raw=true)

대부분의 정규표현식 metacharacters에 대해(다는 아니고) 작동한다. `$`, `.`, `|`, `?`, `*`, `+`, `(`, `)`, `[`, `{` <br /> 하지만, 몇몇 캐릭터들은 character class 안에서도 특별한 뜻을 가져서, 백슬래쉬 escape와 함께 써줘야 한다. `]`, `\`, `^`, `=`

하나 이상의 패턴 중에서 pick하고 싶다면, alternation을 쓸 수 있다. <br /> 예를 들어서, `abc|d..f`라고 하면, "abc"나 "deaf"를 match한다. <br /> `|` 는 우선순위가 낮아서, `abc|xyz`라고 하면, `abc` 혹은 `xyz`를 match하지, `abcyz` 혹은 `abxyz`를 match하는게 아니다. <br /> 수학적 표현과 마찬가지로, 우선순위가 좀 헷갈린다면, 괄호를 써서 명확하게 할 수 있다.

``` r
str_view(c("grey", "gray"), "gr(e|a)y")
```

![14.3.3-4](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.3-4.png?raw=true)

#### 14.3.3.1 Exercises

<br /> <br /> <details> <summary>14.3.3.1 Exercises sol</summary> </details> <br /> <br /> <br />

### 14.3.4 Repetition

몇 번이나 나오는지를 컨트롤하는 것이 다음 단계.

-   `?` : 0 or 1
-   `+` : 1 or 그 이상
-   `*` : 0 or 그 이상

``` r
x <- "1888 is the longest year in Roman numerals: MDCCCLXXXVIII"
str_view(x, "CC?")
```

![14.3.4-1](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.4-1.png?raw=true)

``` r
str_view(x, "CC+")
```

![14.3.4-2](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.4-2.png?raw=true)

``` r
str_view(x, "C[LX]+")
```

![14.3.4-3](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.4-3.png?raw=true)

이 operators의 우선순위는 높다. 그래서 다음과 같이 쓸 수도 있다. <br /> 미국-영국 스펠링에 대한 match를 위해, `colou?r`처럼. <br /> 이 말인즉슨, 대부분의 사용법에서는 괄호가 필요할 거란 뜻이다. 예를 들어 `bana(na)+`같이.

몇 번이나 나올지를 정해줄 수도 있다.

-   `{n}` : 정확히 n번
-   `{n, }` : n번 or 그 이상
-   `{, m}` : 최대 m번
-   `{n, m}` : n번과 m번 사이

``` r
str_view(x, "C{2}")
```

![14.3.4-4](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.4-4.png?raw=true)

``` r
str_view(x, "C{2,}")
```

![14.3.4-5](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.4-5.png?raw=true)

``` r
str_view(x, "C{2,3}")
```

![14.3.4-6](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.4-6.png?raw=true)

디폴트로, 이러한 match들은 "greedy"하다. 가능한 가장 긴 string을 match한다. <br /> 이걸 "lazy"하게 만들 수 있다. 가능한 가장 짧은 string을 match하도록. 끝에 `?`를 붙여서. <br /> 정규표현식의 고급특성인데, 존재한다는걸 알아두면 좋다.

``` r
str_view(x, "C{2,3}?")
```

![14.3.4-7](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.4-7.png?raw=true)

``` r
str_view(x, "C[LX]+?")
```

![14.3.4-8](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.4-8.png?raw=true)

#### 14.3.4.1 Exercises

<br /> <br /> <details> <summary>14.3.4.1 Exercises sol</summary> </details> <br /> <br /> <br />

### 14.3.5 Grouping and backreferences

복잡한 표현을 명확하게하는 방법으로 괄호를 쓰는 걸 배웠다. <br /> 괄호는 또한, 번호가 매겨진 capturing group을 만드는데도 쓸 수 있다.

capturing group은 괄호 안의 정규표현식 부분과 일치하는, 문자열 부분을 저장한다. <br /> 그리고 이렇게 capturing group 뒤에다가, `\1`, `\2`와 같은 역참조backreference를 통해, 이전에 일치했던 것과 동일한 텍스트를 refer하도록 할 수 있다. <br /> A capturing group stores the part of the string matched by the part of the regular expression inside the parentheses. <br /> You can refer to the same text as previously matched by a capturing group with backreferences, like `\1`, `\2` etc.

이거 이해하느라 한참 걸렸는데, 어려운거 아니다. <br /> 그런데, 말로 하면 도대체 뭔소린지를 모르겠다. 예제를 빠르게 보자.

``` r
str_view(fruit, "(..)\\1", match = TRUE)
```

![14.3.5-1](https://github.com/philyoun/portfolio/blob/master/docs/R%20for%20Data%20Science/14-Strings_files/14.3.5-1.png?raw=true)

그러니깐 뒤에 나오는 `\\1`은 앞에 나온 `(..)`를 refer하는거다. <br /> 그래서 즉, repeated pair of letters가 있는 걸 찾아준다.

이건 특히나 exercises를 보면서 예제들을 보며 이해를 해보자.

#### 14.3.5.1 Exercises

1.  다음의 expressions가 무엇을 match할지를 설명해보자. <br />    1번. `(.)\1\1` <br />    2번. `"(.)(.)\\2\\1"` <br />    3번. `(..)\1` <br />    4번. `"(.).\\1.\\1"` <br />    5번. `"(.)(.)(.).*\\3\\2\\1"` <br />

어떤건 `\1`이고 어떤건 `\\1`이다. 일부러 저자가 이렇게 써놨는데, 잘 생각해보자.

2. 다음의 각 조건을 match해주는 정규표현식을 구성해보자. <br />    1번. 하나의 같은 문자로 시작하고 끝나야하려면? <br />    2번. 반복되는 한 쌍의 문자를 갖고 있으려면? 예를 들어, "church"는 "ch"가 두 번 나온다. <br />    3번. 똑같은 문자가 최소 3번 나오려면? 예를 들어, "eleven"에서 "e"는 3번 나온다. <br />

<br /> <br />

<details> <summary>14.3.5.1 Exercises sol</summary> 1. <br /> 1번. 똑같은 문자가 3번 연속으로 나와야함. 예를 들어, "aaa" <br /> 2번. 한 쌍의 문자들이, 상반된 순서로 나와야한다. 그러니깐, "abba" 같이. <br /> 3번. 두 개의 문자들이 연속으로 2번 나와야함. "a1a1" <br /> 4번. 원하는 문자 하나 + any character + 처음 그 문자 하나 + any character + 처음 그 문자 하나. <br /> 예를 들어, "abaca", "b8b.b" <br /> 5번. 이제 이걸 이해하면 다 이해했다고 볼 수 있다. <br /> 처음 (.)은 <code>\\1</code>로, 두 번째 (.)는 <code>\\2</code>로, 세 번째 (.)는 <code>\\3</code>으로 refer되는 거다. <br /> 그리고 가운데는 .\*니깐 0 or more. <br /> 그래서 예를 들어보면, "abccba", "abc1cba" 혹은 "abcsgasgddsadgsdgcba" 다 되는 것.

1.  <br /> 1번. <code>`^(.).*\1$`</code>라고 생각했는데, "a"는 포함이 안 된다. "a"도 조건에 만족하는 단어인데, 꼭 시작과 끝을 명시해놔서 안 잡히는듯. <br /> 정답은 <code>'^(.)((.\*$)|\\1?$)\`</code> <br /> <br /> <br /> 2번.

<br />

</details> <br /> <br /> <br />

14.4 Tools
----------
