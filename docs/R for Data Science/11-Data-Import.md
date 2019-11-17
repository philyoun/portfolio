11 Data Import
================

11.1 Introduction
-----------------

R 패키지에 제공된 데이터로 작업하는건 데이터 사이언스 툴을 배우기에는 좋은 방법이지만, <br /> 언젠가 그만 배우고 니 데이터로 직접 작업하고 싶어질 때가 있을 것이다. <br /> 이 장에서는, 텍스트만 있는 네모난 파일들plain-text rectangular files을 어떻게 R로 읽을지를 배운다.

Data import의 겉만 다룰 것이지만, 많은 원리들이 다른 데이터 형식에도 적용가능하다. <br /> 다른 데이터 형식들에 대한 유용한 패키지들을 소개하며 마무리할 것.

### 11.1.1 Prerequisites

``` r
library(tidyverse)
```

**readr** 패키지를 사용해서 flat files를 로드하는 것을 배울 것이다. <br /> 이 패키지는 tidyverse의 핵심 중 하나다.

------------------------------------------------------------------------

11.2 Getting Started
--------------------

readr 패키지의 대부분 함수들은, flat files를 데이터 프레임으로 바꾸는 것과 연관되어 있다.

-   `read_csv()`는 , 로 분리된 파일을 읽고, `read_csv2()`는 ; 로 분리된 파일을 읽는다.(소수점 자리에 , 를 쓰는 나라에서 빈번) <br /> `read_tsv()`는 tab으로 분리된 파일을 읽음. <br /> `read_delim()`은 any delimiter로 분리된 파일을 읽음.

-   `read_fwf()`는 고정된 폭width으로 파일을 읽는다. <br /> width를 `fwf_widths()`로 정해주거나, `fwf_positions()`로 position을 결정해줄 수 있다. <br /> `read_table()`은 칼럼들이 띄어쓰기로 구분이 되어있는, 고정 폭 파일의 일반적인 변형을 읽어준다. <br /> `read_table()` reads a common variation of fixed width files where columns are separated by white space.

-   `read_log()`는 Apache style log files를 읽는다. <br /> ([webreadr](https://github.com/Ironholds/webreadr)도 체크해봐라. 이건 `read_log()`위에 만들어졌으며, 다른 유용한 도구들을 제공해줌.)

여튼 이 함수들은 모두 비슷한 문법Syntax을 가지고 있다. 하나만 마스터하면, 나머지도 같음. <br /> 그래서 남은 챕터 동안 read\_csv()에 집중할 것이다. <br /> csv파일이 가장 흔한 데이터 저장 형식일 뿐 아니라, 이것만 알면 readr의 다른 함수들에도 쉽게 적용가능해서.

------------------------------------------------------------------------

`read_csv()`의 첫 번째 인자argument가 제일 중요하다: 읽을 파일의 경로path to the file to read.

``` r
heights <- read_csv("data/heights.csv")
## Parsed with column specification:
## cols(
##   earn = col_double(),
##   height = col_double(),
##   sex = col_character(),
##   ed = col_double(),
##   age = col_double(),
##   race = col_character()
## )
```

`read_csv()`를 실행하면, 먼저 column specification부터 출력된다. <br /> 각 칼럼의 이름과 타입을 출력하는데, parsing a file을 할 때 다시 볼거다. readr의 중요한 부분임.

inline csv 파일을 넣어줄 수도 있다. <br /> readr을 가지고 실험해보고 싶거나, <br />     reproducible한 예제를 만들어서 다른 사람들과 공유하고 싶을 때 유용함. <br /> 아래의 예를 봐보자.

``` r
read_csv("a,b,c
1,2,3
4,5,6")
## # A tibble: 2 x 3
##       a     b     c
##   <dbl> <dbl> <dbl>
## 1     1     2     3
## 2     4     5     6
```

위의 2가지 case를 보면 알 수 있듯, `read_csv()`는 가장 첫 줄을 데이터의 이름으로 사용한다. <br /> 그러고 싶지 않다면 사용할 수 있는 2가지의 방법이 있다.

1.  처음 몇 줄에, 데이터에 대한 데이터(메타데이터)가 써져있을 때, `skip = n`을 쓰면 된다. <br /> 혹은 \#이 붙어있는 모든 줄들을 drop시키고 싶다면 `comment = "#"`를 추가

``` r
read_csv("The first line of metadata
  The second line of metadata
  x,y,z
  1,2,3", skip = 2)
## # A tibble: 1 x 3
##       x     y     z
##   <dbl> <dbl> <dbl>
## 1     1     2     3
```

``` r
read_csv("# A comment I want to skip
  x,y,z
  1,2,3", comment = "#")
## # A tibble: 1 x 3
##       x     y     z
##   <dbl> <dbl> <dbl>
## 1     1     2     3
```

1.  칼럼 이름이 없는 데이터. `col_names = FALSE`를 쓰면 된다. `header = F`와 같은 의미. <br /> 이러면 첫 줄을 headings로 다루지 않는다. 대신에 `X1`, ..., `Xn`으로 라벨한다.

``` r
read_csv("1,2,3\n4,5,6", col_names = FALSE)
## # A tibble: 2 x 3
##      X1    X2    X3
##   <dbl> <dbl> <dbl>
## 1     1     2     3
## 2     4     5     6
```

(`"\n"`은 한 줄을 추가하는 편리한 shortcut. 이것과 비슷한 것들을, string basics에서 배우게 될 것이다.)

아니면, `col_names`에다가 character vector를 패스해서, 칼럼 이름으로 쓰게할 수도 있다.

``` r
read_csv("1,2,3\n4,5,6", col_names = c("x", "y", "z"))
## # A tibble: 2 x 3
##       x     y     z
##   <dbl> <dbl> <dbl>
## 1     1     2     3
## 2     4     5     6
```

약간 수정이 필요한 또 한 가지 경우는, missing values를 `na`로 바꿔줘야할 때. <br /> 예를 들어 파일에 missing value가 na로 저장되어있지 않고 .로 표현되어 있다면, na = "." 옵션을 추가

``` r
read_csv("a,b,c\n1,2,.", na = ".")
## # A tibble: 1 x 3
##       a     b c    
##   <dbl> <dbl> <lgl>
## 1     1     2 NA
```

이제 니가 실제생활에서 만날 75%의 csv파일들은 다 어떻게 읽어야할지 안다. <br /> 앞서 배운 것들을, `read_tsv()`나 `read_fwf()`에 쉽게 적용할 수도 있다. <br /> 좀 더 어려운 파일들에 대해서는, 먼저 readr이 각 칼럼을 어떻게 parse해서 R 벡터로 바꾸는지를 배워야한다.

### 11.2.1 Compare to base R

왜 `read.csv()`를 안 쓰는지? readr의 것들이 어떤 이점이 있는지를 나열해주자면,

1.  훨씬 빠르다.(최대 10배) <br />   큰 파일에 대해서는 progress bar가 나오기 때문에, 무슨 일이 일어나고 있는지 파악가능하다. <br />   진짜 속도만 원한다면, `data.table::fread()`를 사용해보라. tidyverse에 잘 맞는다고 할 수는 없지만, 꽤나 빠를 수 있다.

2.  이 함수들은 tibbles로 만들어준다. <br />   character 벡터를 factors로 만들거나, 행의 이름을 사용하거나, 칼럼 이름을 망가뜨리지않는다. <br />   위의 저것들이 base R의 함수에선 꽤나 자주 일어나는 일이다.

3.  더 reproducible하다. <br />   base R 함수들은 작동하고 있는 시스템이나 environment variables에서 some behavior를 inherit할 수 있어서, <br />   그래서, 다른 사람의 컴퓨터에서는 니가 쓴 코드가 작동하지 않을 수도 있다.

### 11.2.2 Exercises

------------------------------------------------------------------------

11.3 Parsing a vector
---------------------

readr이 디스크에서 어떻게 파일을 읽는지 디테일하게 알기 전에, `parse_*()`에 대해 다뤄봐야한다. <br /> 이 함수들은 character vector를 받고, 좀 더 특정화된specialised 벡터(logical, integer, or date)를 반환한다.

``` r
str(parse_logical(c("TRUE", "FALSE", "NA")))
##  logi [1:3] TRUE FALSE NA
str(parse_integer(c("1", "2", "3")))
##  int [1:3] 1 2 3
str(parse_date(c("2010-01-01", "1979-10-14")))
##  Date[1:2], format: "2010-01-01" "1979-10-14"
```

위에서 input으로 쓰이는 벡터는 **character vector**라는 걸 인지하시길. <br /> 이걸 영리하게 잘 parse해준다는게 핵심ㅇㅇ

이 함수들은 그 자체로 유용하지만, readr의 중요한 building block이기도 하다. <br /> 개별적인 parser가 어떻게 작동하는지 배운 다음, <br />     다음 section에서 complete file을 parse하는 것과 어떻게 연결되는지를 확인할 것이다.

tidyverse의 모든 함수들과 마찬가지로, `parse_*()`의 함수들은 통일성이 있다. <br /> 첫 번째 인자argument는 parse할 character vector이고, `na` argument가 어떤 string을 결측값으로 받을 것인지를 정한다. <br /> 다음의 예를 보자.

``` r
parse_integer(c("1", "231", ".", "456"), na = ".")
## [1]   1 231  NA 456
```

만약 parsing이 실패한다면, wargning을 갖게 된다.

``` r
x <- parse_integer(c("123", "345", "abc", "123.45"))
## Warning: 2 parsing failures.
## row col               expected actual
##   3  -- an integer                abc
##   4  -- no trailing characters    .45
```

실패한 것들은 output에서 결측값으로 표시된다.

``` r
x
## [1] 123 345  NA  NA
## attr(,"problems")
## # A tibble: 2 x 4
##     row   col expected               actual
##   <int> <int> <chr>                  <chr> 
## 1     3    NA an integer             abc   
## 2     4    NA no trailing characters .45
```

많은 parsing failures가 있다면, complete set을 얻기 위해서 `problems()`을 사용해야 할 수도 있다. <br /> 이건 tibble을 반환하고, dplyr로 니가 조작할 수 있다.

``` r
problems(x)
## # A tibble: 2 x 4
##     row   col expected               actual
##   <int> <int> <chr>                  <chr> 
## 1     3    NA an integer             abc   
## 2     4    NA no trailing characters .45
```

parser를 이용한다는 것은 대부분, <br />     무엇이 사용가능한지 그리고 어떻게 다른 타입의 input을 다룰 건지를 이해하는 것이다. Using parsers is mostly a matter of understanding what's available and how they deal with different types of input. 8개의 특히 중요한 parser들이 있다.

1.  `parse_logical()`와 `parse_integers()`은 각각 logical과 integer를 parse. <br /> 잘못될게 없기 때문에 더 설명 안하겠다.

2.  `parse_double()`은 strict numeric parser고, `parse_number()`은 flexible numeric parser다. <br /> 니 생각보다 더 복잡하다. 왜냐하면 세상에는 숫자를 다른 방법으로 쓰는데도 있기 때문.

3.  `parse_character()`는 너무 단순simple해서 필요없을 것 같지만, 하나의 문제, character encodings때문에 중요하다.

4.  `parse_factor()`는 factors를 만든다. <br /> factors라는건, R이, 고정되어 있고 알려져있는fixed and known 카테고리컬 변수들을 표현하는데 쓰는 데이터 구조.

5.  `parse_datetime()`, `parse_date()`, `parse_time()`은 다양한 date, time 을 parse하도록 해준다. <br /> 그런데 세상엔 날짜를 쓰는 방식이 너무나도 많아서, 이것들이 가장 복잡한 것들이다.

### 11.3.1 Numbers

숫자를 parse하는건 쉽고 간단할 것 같지만, 3가지 문제가 까다롭게 만든다.

1.  세상에는 수를 다르게 쓰는 나라가 많다. <br /> 보통 소수점으로 `.`을 쓰지만, 어떤 나라에서는 `,`을 쓴다.

2.  숫자는 종종, 어떠한 context를 공급하기 위해, 다른 캐릭터들로 둘러싸여있다. <br /> 예를 들어, "$1000", "10%" 같은거.

3.  숫자는 보통 읽기 쉽게 해주는 grouping 캐릭터가 있다. 예를 들어서, "1,000,000"에서 쉼표의 역할. <br /> 그리고 이러한 grouping 캐릭터는 세상에 많은 종류가 있다.

첫 번째 문제를 해결하기 위해서, readr은 locale이라는 옵션이 있다. <br /> parsing 옵션이 그때그때마다 다를 수 있도록 해주는 object. <br /> 숫자를 parsing할 때 가장 중요한건, 소수점decimal mark이 뭔지 지정하는건데, <br />     디폴트는 `.`이지만, locale의 `decimal_mark` 인자argument를 통해 바꿀 수 있다.

``` r
parse_double("1.23")
## [1] 1.23
parse_double("1,23", locale = locale(decimal_mark = ","))
## [1] 1.23
```

readr의 디폴트 locale은 미국 중심US-centric이다. 왜냐하면 R은 보통 미국 중심이기 때문. <br /> (base R의 documentation은 미국어로 쓰였다.) <br /> 디폴트를 바꿀 수도 있나본데 다른 사람 컴퓨터에서는 작동이 안 되는 그런 문제가 있을 수 있기 때문에 추천 안함.

두 번째 문제를 해결하기 위해서, `parse_number()`를 쓴다. <br /> 얘는 숫자 전후로 나오는 non-numeric 캐릭터는 무시해준다. <br /> 통화량이나 퍼센티지를 다룰때 특히나 유용하지만, 텍스트에 있는 숫자를 추출할 때도 사용할 수 있다.

``` r
parse_number("$100")
## [1] 100
parse_number("20%")
## [1] 20
parse_number("It cost $123.45")
## [1] 123.45
```

세 번째 문제는 `parse_number()`의 locale에 있는 `grouping_mark` 인자argument를 사용해서 해결할 수 있음.

``` r
# 미국에서 쓰는 방식
parse_number("$123,456,789")
## [1] 123456789

# 유럽의 많은 국가에서 쓰는 방식
parse_number("123.456.789", locale = locale(grouping_mark = "."))
## [1] 123456789

# 스위스에서 쓰는 방식
parse_number("123'456'789", locale = locale(grouping_mark = "'"))
## [1] 123456789
```

### 11.3.2 Strings

`parse_Character()`는 간단해야할 것처럼 보인다. 인풋을 그대로 반환하면 되니깐. <br /> 하지만 인생이 그렇게 쉽지 않다. Unfortunately life isn't so simple. <br /> 왜냐하면 같은 string을 표현할 방법이 여러가지이기 때문. <br /> 뭐가 어떻게 돌아가는지 이해하기 위해서, 어떻게 컴퓨터가 strings을 표현하는지 디테일하게 알아봐야한다. <br /> `charToRaw()`를 사용해서 string의 underlying representation을 얻을 수 있다.

``` r
charToRaw("Hadley")
## [1] 48 61 64 6c 65 79
```

각 16진법 수는, 하나의 정보 바이트(ASCII 문자 하나)를 표현한다. `48`은 H, `61`은 a, 이런 식으로. <br /> 이 16진법 수를 character로 mapping하는 걸 encoding이라고 부른다. <br /> 이 인코딩은 ASCII임. 영어 캐릭터 표현에는 ASCII가 좋은데, <br />     애초에 ASCII는 American Standard Code for Information Interchange의 줄임말이기 때문.

영어가 아닌 다른 문자들에 대해서는 좀 더 복잡해진다. <br /> non-English문자들에 대해서 여러가지 기준이 있었고, <br />     정확하게 string을 번역하기 위해서는 값과 인코딩 둘 다를 알아야 했었다. <br /> 예를 들어, 가장 흔하게 쓰던 인코딩이 Latin1(서유럽 언어들에 사용되는 ISO-8859-1), Latin2(동유럽 언어들에 사용되는 ISO-8859-2) <br /> 그런데 Latin1에서 `b1`이라는 바이트는 "±"이고, Latin2에서는 "ą"이다.(...)

다행히도, 오늘날에는 거의 모든 곳에서 지원되는 단 하나의 스탠다드가 있다. UTF-8. <br /> 얘는 사람들이 쓰는 모든 character들을 인코드할 수 있고, 심지어는 emoji같은 extra symbols도 가능하다.

readr은 모든 곳에서 UTF-8을 쓴다. <br /> 읽을 때도 니 데이터가 UTF-8로 인코드되어있다고 가정하고, 쓸 때도 이걸로 쓴다. <br /> 좋은 디폴트지만, UTF-8을 이해하지 못하는 오래된 시스템에서는 깨질 수 있다. <br /> 예를 들어서,

``` r
x1 <- "El Ni\xf1o was particularly bad this year"
x2 <- "\x82\xb1\x82\xf1\x82\xc9\x82\xbf\x82\xcd"
x1
## [1] "El Ni\xf1o was particularly bad this year"
x2
## [1] "궞귪궸궭궼"
```

이 문제를 해결하기 위해, 인코딩을 `parse_character()`에다가 정해줄 수 있다.

``` r
parse_character(x1, locale = locale(encoding = "Latin1"))
## [1] "El Nino was particularly bad this year"
parse_character(x2, locale = locale(encoding = "Shift-JIS"))
## [1] "こんにちは"
```

어떻게 정확한 인코딩을 찾느냐? <br /> 재수가 좋다면, 데이터 문서에 써져 있을 것이다. <br /> 하지만 그런 경우는 별로 없다. 그럴 때는 `guess_encoding()`라는걸 한 번 써보자. <br /> 항상 잘 되는건 아니지만, 텍스트가 많은 경우에는 성공할 확률이 높다. <br /> 잘 찾을 때까지 이것저것 해보자.

``` r
guess_encoding(charToRaw(x1))
## # A tibble: 2 x 2
##   encoding   confidence
##   <chr>           <dbl>
## 1 ISO-8859-1       0.46
## 2 ISO-8859-9       0.23
guess_encoding(charToRaw(x2))
## # A tibble: 1 x 2
##   encoding confidence
##   <chr>         <dbl>
## 1 KOI8-R         0.42
```

`guess_encoding()`의 첫 번째 인자argument는 이 경우에서처럼 raw vector일수도 있고, <br /> 파일 경로file path일 수도 있다.

인코딩은 풍부하고 복잡한 주제이기 때문에, 여기서는 겉만 핥아봤다. <br /> 좀 더 알아보고 싶다면, <http://kunststube.net/encoding/>을 읽어볼 것을 추천한다.

### 11.3.3 Factors

R은 "이미 가능한 값들이 알려져 있는" 카테고리컬 변수를 표현하는데 factors를 쓴다. <br /> `parse_factor()`에다가 `levels` 옵션을 줘서 예상치 못한 값이 나왔을 때 warning이 뜰 수 있도록 하자.

``` r
fruit <- c("apple", "banana")
parse_factor(c("apple", "banana", "bananana"), levels = fruit)
## Warning: 1 parsing failure.
## row col           expected   actual
##   3  -- value in level set bananana
## [1] apple  banana <NA>  
## attr(,"problems")
## # A tibble: 1 x 4
##     row   col expected           actual  
##   <int> <int> <chr>              <chr>   
## 1     3    NA value in level set bananana
## Levels: apple banana
```

하지만 문제가 있는 entries가 많으면, 그냥 character vectors로 내버려두고, strings에서와 factors에서 배울 것들로 clean up하자.

### 11.3.4 Dates, date-times, and times

자, 다음의 3가지 parsers 중에 필요한 걸 쓰자. <br /> 1. 날짜 Dates 2. 날짜와 시간 Date-times 3. 시간 Times

다른 추가적인 인자들arguments 없이 호출해보면,

-   `parse_date()`는 네 자리로 된 year, 그리고 - or /, 그리고 month, 그리고 - or /, 그리고 day를 받을 것을 예상한다.

``` r
parse_date("2010-10-01")
## [1] "2010-10-01"
parse_date("2010/10/01")
## [1] "2010-10-01"
```

-   `parse_datetime()`은 ISO8601 date-time을 예상한다. <br /> year-month-day-hour-minute-second순으로 되어있는 국제적인 표준

``` r
parse_datetime("2010-10-01T2010")
## [1] "2010-10-01 20:10:00 UTC"
parse_datetime("20101010")
## [1] "2010-10-10 UTC"
```

이게 가장 중요한 date/time 스탠다드이고, dates, times와 작업하는 경우가 많으면, [https://en.wikipedia.org/wiki/ISO\_8601을](https://en.wikipedia.org/wiki/ISO_8601을) 읽어볼 것을 추천한다.

-   `parse_time()`은 hour, `:`, minutes, 그리고 옵션으로 `:`, seconds, 그리고 옵션으로 am/pm

``` r
library(hms)
## Warning: package 'hms' was built under R version 3.5.2
parse_time("01:10 am")
## 01:10:00
parse_time("01:10 pm")
## 13:10:00
parse_time("20:10:01")
## 20:10:01
```

base R은 time 데이터에 대한 built-in 클래스가 딱히 좋은게 없어서, hms 패키지에 있는걸 사용했다.

여기까지는 추가적인 인자들, extra arguments 없이 호출해본 것. <br /> 만약에 이 디폴트들이 당신의 데이터에는 잘 맞지 않다면, 필요에 맞게끔 format을 공급해줄 수 있다. <br /> 다음을 보자.

**Year** `%Y` (4자리) <br /> `%y` (2자리); 00-69 -&gt; 2000-2069, 70-99 -&gt; 1970-1999 (이러면 1969년같은건 표현 못한다는거네)

**Month** `%m` (2자리) <br /> `%b` ("Jan"과 같이 축약된 이름) <br /> `%B` ("January"와 같은 풀네임) <br />

**Day** `%d` (2자리) <br /> `%e` (optional leading space앞에 몇 자리 공백을 없애준다는 뜻인듯)

**Time** `%H` 0-23 hours <br /> `%I` 0-12, 대신에 `%p`와 항상 같이 쓰여야한다. <br /> `%p` AM/PM 지표Indicator <br /> `%M` minutes <br /> `%S` integer seconds <br /> `%OS` real seconds <br /> `%Z` Time zone(이름으로, 예를 들면, `America/Chicago`) <br /> 축약해서 부르는 거에 조심하자. <br /> 만약 미국인이라면, "EST"가 "Eastern Standard Time"이 아니라. <br /> 서머타임이 없는 Canadian time zone이다. (Canadian time zone that doesn't have daylight savings time) <br /> `%z` UTC에서 얼마나 차이가 나는지. 예를 들어, +0800처럼.

**Non-digits** `%.` (숫자가 아닌 하나의 캐릭터를 skip해줌)skips one non-digit character <br /> `%*` (숫자가 아닌 모든 캐릭터들을 skip해줌)skips any number of non-digits

정확한 포맷을 찾기 위한 방법으로, 몇 개의 예를 character vector로 만들어보고, parsing을 해보는거다. <br /> 예를 들어,

``` r
parse_date("01/02/15", "%m/%d/%y")
## [1] "2015-01-02"
parse_date("01/02/15", "%d/%m/%y")
## [1] "2015-02-01"
parse_date("01/02/15", "%y/%m/%d")
## [1] "2001-02-15"
```

만약 영어가 아닌 걸로, `%b`, `%B`에다가 월별 이름을 사용하고 있다면, <br />     `locale()`에다가 `lang` 인자argument를 설정해줘야 한다. `date_names_langs()`를 통해 built-in languages의 리스트를 확인해보자. <br /> 만약 당신의 언어가 포함되어 있지 않다면, `date_names()`를 통해 하나 만들수도 있다.

``` r
parse_date("1 janvier 2015", "%d %B %Y", locale = locale("fr"))
## [1] "2015-01-01"
```

### 11.3.5 Exercises

------------------------------------------------------------------------

11.4 Parsing a vector
---------------------
