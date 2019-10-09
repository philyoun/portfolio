9 Functionals
=============

9.1 Introduction
----------------

코드가 더 믿을만하려면, 더 투명해야 한다. <br /> 특히나, nested conditions랑 루프는 의심을 갖고 보아야 한다. <br /> 복잡한 control flows는 프로그래머를 혼란스럽게 한다. <br /> 복잡한 코드는 종종 버그를 숨기고 있다. <br /> To become significantly more reliable, code must become more transparent. <br /> In particular, nested conditions and loops must be viewed with great suspicion. <br /> Complicated control flows confuse programmers. <br /> Messy code often hides bugs.

**자 그래서 functional 무엇이냐? function과는 어떻게 다른 것인가?** <br /> functional은, input을 함수로 받고, output을 벡터로 return하는 함수. <br /> A functional is a function / that takes a function as an input / and returns a vector as output.

간단한 Functional을 소개해보자. <br /> 1000개의 random uniform numbers를 input으로 넣은 함수를 호출call하는 functional.

``` r
randomise <- function(f) f(runif(1e3))
randomise(mean)
## [1] 0.4889273
randomise(mean)
## [1] 0.489791
randomise(sum)
## [1] 506.9031
```

이미 functional을 사용해봤을 수 있다. <br /> for 루프문을 대신하기 위해 base R의 `lapply()`, `apply()`나 `tapply()` 혹은 purrr의 `map()`을 써봤을거다. <br /> 혹은 수학적 functional인 `integrate()`나 `optim()`을 써봤을 수도 있다.

functional을 주 용도는 for 루프문 대신해서 쓰는 것이다. <br /> for 루프는 많은 사람들이 느리다고 믿기 때문에, R에서 평판이 안 좋다. <br /> 하지만 for 루프의 진정한 단점은 for 루프가 매우 유연하다는 것이다. <br /> 루프를 사용하면 반복한다는 의도를 전달할 수 있지만, 그 결과물로 무엇을 해야하는지는 알려주지 않는다. <br /> a loop conveys that you’re iterating, but not what should be done with the results.

`repeat` 대신에 `while`이, `while` 대신에는 `for`을 사용하는 것이 낫듯이,(Section 5.3.2) <br /> `for`대신에 functional을 사용하는 것이 낫다. <br /> 각 functional은 특정한 작업에 잘 맞추어져있기 때문에, functional을 보면 왜 쓰는지를 바로 볼 수 있다.

만약에 니가 loop문에 익숙한 유저라면, functionals로 스위칭하는 것은 그냥 패턴 매칭 연습하는 것이다. <br /> for 루프를 보고, 알맞은 기본형 functional을 매치시키면 된다. <br /> 만약에 그런 것이 존재하지 않는다면, 이미 존재하는 functional을 변형해서 찾으려고 노력하지 마라. <br /> 그냥 for 루프로 남겨둬라!(만약에 같은 루프를 두 번 이상 반복해야 한다면, 자신의 functional을 쓸 생각도 해봐야 한다.)

### Prerequisites

이 chapter에서는, purrr package에 있는 functional에 집중할 것이다. <br /> 일관적인 인터페이스를 갖고 있어서, base에 있는 것들보다 핵심 아이디어를 더 이해하기 쉽게 한다. <br /> 몇 년에 걸쳐 유기적으로 개발된 것이다. <br /> base R의 동일한 것들과, 앞으로도 비교대조하겠다. <br /> 그리고 purrr에는 없는 base functionals에 대한 논의로 chapter 마무리를 하겠다.

``` r
library(purrr)
```

9.2 My first functional: `map()`
--------------------------------

가장 기초적인 functional은 `purrr::map()`이다. <br /> a vector와 a function을 받고, 벡터의 각 element에 대해 함수를 호출한다. <br /> 그리고나서 결과물을 리스트로 return한다. <br /> 즉, `map(1:3, f)`는 `list(f(1), f(2), f(3))`과 같은 것이다.

``` r
triple <- function(x) x * 3
map(1:3, triple)
## [[1]]
## [1] 3
## 
## [[2]]
## [1] 6
## 
## [[3]]
## [1] 9
```

시각적으로 나타내면, ![그림1](https://d33wubrfki0l68.cloudfront.net/f0494d020aa517ae7b1011cea4c4a9f21702df8b/2577b/diagrams/functionals/map.png)

<style>
p.comment {
background-color: #DBDBDB;
padding: 10px;
border: 1px solid black;
margin-left: 25px;
border-radius: 5px;
}
</style>
<p class="comment">
왜 이 함수를 <code>map()</code>이라고 할까? <br /> 지도가 아니라 수학적인 map, "given set의 각 element를, 두 번째 set의 하나 혹은 이상의 elements로 association시키는 연산operation"이라는 뜻에서 <code>map()</code>이라고 부르는 것이다. <br /> (그리고 "Map"이라는 단어는 스펠링이 짧아서 기초적인 building block이 되기에 좋다.)
</p>
`map()`의 implementation은 꽤나 간단하다. <br /> input과 같은 길이의 리스트를 만들어놓고, 리스트의 각각을 for 루프를 사용해서 채운다. <br /> implementation의 핵심은 코드 몇 줄 되지 않는다.

``` r
simple_map <- function(x, f, ...) {
  out <- vector("list", length(x))
  for (i in seq_along(x)) {
    out[[i]] <- f(x[[i]], ...)
  }
  out
}
```

실제 `purrr::map()`은 이렇게 만들어지진 않았다. 몇 가지 차이점이 있다. <br /> C언어로 써서 성능을 조금이라도 더 짜냈고, 이름을 보존하고, Section 9.2.2에서 배울 몇 가지 shorcuts들을 지원한다.

<p class="comment">
<code>map()</code>과 동등한 base 함수는 <code>lapply()</code>다. <br /> 유일한 차이는 <code>lapply()</code>는 밑에서 배우게 될 helpers를 지원하지 않는다는 것이다. <br /> 그래서 만약에 purrr에서 <code>map()</code>만 쓸 것이라면, 추가적인 의존성additional dependency를 스킵하고 <code>lapply()</code>를 쓰면 된다.
</p>
### 9.2.1 Producing atomic vectors

`map()`은 리스트를 return하는데, map family가 가장 일반적일 수 있게 해주는 거다. <br /> 왜냐하면 그 어떤 것이라도 리스트에 넣을 수 있기 때문.

그런데 더 간단한 데이터 구조data structure가 가능한데 꼭 리스트를 return하게 하는 것은 불편. <br /> 그렇기 때문에 4가지의 더 특정한 변형들이 있다. `map_lgl()`, `map_int()`, `map_dbl()`, `map_chr()` <br /> 각각은 특정화된specified 타입의 벡터를 return해준다.

1.  `map_chr()`은 항상 character vector를 return한다.

``` r
map_chr(mtcars, typeof)
##      mpg      cyl     disp       hp     drat       wt     qsec       vs 
## "double" "double" "double" "double" "double" "double" "double" "double" 
##       am     gear     carb 
## "double" "double" "double"
```

2. `map_lgl()`은 항상 logical vector를 return한다.

``` r
map_lgl(mtcars, is.double)
##  mpg  cyl disp   hp drat   wt qsec   vs   am gear carb 
## TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
```

3. `map_int()`은 항상 integer vector를 return한다.

``` r
n_unique <- function(x) length(unique(x))
map_int(mtcars, n_unique)
##  mpg  cyl disp   hp drat   wt qsec   vs   am gear carb 
##   25    3   27   22   22   29   30    2    2    3    6
```

4. `map_dbl()`은 항상 double vector를 return한다.

``` r
map_dbl(mtcars, mean)
##        mpg        cyl       disp         hp       drat         wt 
##  20.090625   6.187500 230.721875 146.687500   3.596563   3.217250 
##       qsec         vs         am       gear       carb 
##  17.848750   0.437500   0.406250   3.687500   2.812500
```

purrr은, 접미사suffix만 봐도 output이 무엇이 될지를 알 수 있도록 했다. <br /> `_dbl`이면 double vector가 output이겠구나! 하게끔.

모든 `map_*()`함수들은 어떠한 타입의 벡터든 input으로 받을 수 있다. <br /> `mtcars`는 데이터 프레임이고, 데이터 프레임은 같은 길이의 벡터들을 갖고 있는 리스트들이다. <br /> 벡터 때와 마찬가지로 그림으로 그려서 표현해보면, ![그림2](https://d33wubrfki0l68.cloudfront.net/12f6af8404d9723dff9cc665028a35f07759299d/d0d9a/diagrams/functionals/map-list.png)

모든 map 함수들은 input과 같은 길이를 갖는 output 벡터를 return해야한다. <br /> 이 말인즉슨, `.f`의 각 호출call이 single value를 return해야한다는 것이다. <br /> 만약 그렇지 않다면, 에러가 나온다.

``` r
pair <- function(x) c(x, x)
map_dbl(1:2, pair)
```

    ## Error: Result 1 is not a length 1 atomic vector

이 때의 에러는, `.f`가 잘못된 타입일 때 얻게되는 에러와 같다.

``` r
map_dbl(1:2, as.character)
## Error: Can't coerce element 1 from a character to a double
```

위 2가지의 경우에 있어, 그냥 `map()`을 쓰는게 낫다. <br /> 왜냐하면 `map()`은 어떠한 타입의 output이든 다 받아주기 때문이다. <br /> 이러고나면 output이 어떻게 생겼는지를 알 수 있고, 뭘 해야할지를 알게 된다.

``` r
map(1:2, pair)
## [[1]]
## [1] 1 1
## 
## [[2]]
## [1] 2 2
map(1:2, as.character)
## [[1]]
## [1] "1"
## 
## [[2]]
## [1] "2"
```

<p class="comment">
<strong>base R에서는</strong> <br /> base R은 atomic vector를 return하는 2개의 apply 함수들을 가지고 있다. <br /> <code>sapply()</code>랑 <code>vapply()</code>. <br /> <code>sapply()</code>는 비추하는데, 왜냐하면 얘는 results를 간단화simplify시키려고 노력을 하기 때문에, <br /> list를 return할 수도, vector를 return할 수도, matrix를 return할 수도 있기 때문이다. <br /> 이러면 프로그램을 하기가 힘들어지고, non-interactive 셋팅에서는 피해야할 일이다. <br /> <code>vapply()</code>는 <code>FUN.VALUE</code>라는 template를 제공해서, output shape를 describe해주기 때문에 더 안전하다. <br /> 만약 purrr을 이용하고 싶지 않다면, <code>sapply()</code>가 아닌, <code>vapply()</code>를 항상 이용하기를 권한다. <br /> <code>vapply()</code>의 단점으로는, 너무 장황하다는 것이다.verbosity <br /> 예를 들어, <code>map\_dbl(x, mean, na.rm = TRUE)</code>를 base R로 써보려고 하면, <br /> <code>vapply(x, mean, na.rm = TRUE, FUN.VALUE = double(1))</code>이다.
</p>
### 9.2.2 Anonymous functions and shortcuts

이미 존재하는 함수와 `map()`을 쓰는 것 대신에, inline anonymous 함수를 쓸 수도 있다.(Section 6.2.3) <br /> (anonymous 함수라는게 대단한 것은 아니고, 이름도 붙일 필요 없는 그냥 한 줄짜리 간단한 함수.)

``` r
map_dbl(mtcars, function(x) length(unique(x)))
##  mpg  cyl disp   hp drat   wt qsec   vs   am gear carb 
##   25    3   27   22   22   29   30    2    2    3    6
```

anonymous 함수는 굉장히 유용하지만, 문법syntax이 좀 길다. 그래서 purrr은 shortcut을 제공한다.

``` r
map_dbl(mtcars, ~ length(unique(.x)))
##  mpg  cyl disp   hp drat   wt qsec   vs   am gear carb 
##   25    3   27   22   22   29   30    2    2    3    6
```

모든 함수들이 `~`(twiddle이라고 발음함)로 시작하는 formula를 함수function로 번역해주기 때문에, 이게 가능하다. <br /> `as_mapper()`를 사용해서 무엇이 일어나고 있는지를 확인할 수 있다.

``` r
as_mapper(~ length(unique(.x)))
## <lambda>
## function (..., .x = ..1, .y = ..2, . = ..1) 
## length(unique(.x))
## attr(,"class")
## [1] "rlang_lambda_function" "function"
```

이 함수 argument는 유별나보이지만quirky, <br /> 하나의 argument를 갖고 있는 함수에 대해서는 `.`를 통해서 refer을, <br /> 두개의 arguments를 갖고 있는 함수에 대해서는 `.x`와 `.y`를 통해서 refer을, <br /> 그 이상의 arguments를 갖고 있는 함수에 대해서는 `..1`, `..2`, `..3` 등등을 통해 refer을 할 수 있도록 해준다.

`.`는 하위 호환성backward compatibility를 위해서 남겨두었지만, 이걸 이용하는 건 추천하지 않는다. <br /> 왜냐하면 magrittr의 pipe와 쉽게 헷갈리기 때문이다.

다음의 shortcut은 random data를 만드는데 있어 굉장히 유용하다.

``` r
x <- map(1:3, ~ runif(2))
str(x)
## List of 3
##  $ : num [1:2] 0.0223 0.7066
##  $ : num [1:2] 0.495 0.552
##  $ : num [1:2] 0.969 0.708
```

짧고 간단한 함수들에 사용할 걸 대비해 이 사용법을 익혀두자. <br /> 한 줄이 넘어갈 정도로 길어지거나 `{}`을 사용할 정도가 되면, name을 붙여줄 때가 된 것이다.

벡터vector에서 element를 추출하는데에도 shortcut이 있는데, `purrr::pluck()`으로 power된다. <br /> element를 이름으로 추출할 수도, 몇 번째 position에 있느냐를 바탕으로 숫자로 추출할 수도, 리스트에서 둘 다를 이용해서 추출할 수도 있다. <br /> 이건 JSON에서 종종 일어나는, deeply nested list랑 작업할 때 편하다.

``` r
x <- list(
  list(-1, x = 1, y = c(2), z = "a"),
  list(-2, x = 4, y = c(5, 6), z = "b"),
  list(-3, x = 8, y = c(9, 10, 11))
)

# 이름으로 추출하기
map_dbl(x, "x")
## [1] 1 4 8

# position으로 추출하기
map_dbl(x, 1)
## [1] -1 -2 -3

# 둘 다 이용해서 추출하기
map_dbl(x, list("y", 1))
## [1] 2 5 9

# 만약에 해당하는 곳에 component가 없다면 에러가 나옴
map_chr(x, "z")
## Error: Result 3 is not a length 1 atomic vector

# 디폴트를 제공해놓으면 그 값이 나옴
map_chr(x, "z", .default = NA)
## [1] "a" "b" NA
```

<p class="comment">
<strong>base R에서는</strong> <br /> <code>lapply()</code>와 같은 base R 함수들도, 이름을 string으로 니가 제공해줄 수는 있다. <br /> 하지만 별로 유용하지는 않은게, <code>lapply(x, "f")</code>는 거의 항상 <code>lapply(x, f)</code>와 같고, <br /> 타이핑만 늘어날 뿐이기 때문이다.
</p>
### 9.2.3 Passing argument with `...`

니가 호출하는 함수에다가 추가적인 인자들additional arguments을 패스해놓는 것은 종종 유용하다. <br /> 예를 들어, `mean()`이라는 함수에다가 `na.rm = TRUE`를 패스하고 싶다고 치자. <br /> 하나의 방법은 anonymous 함수를 이용하는 것이다.

``` r
x <- list(1:5, c(1:10, NA))
map_dbl(x, ~ mean(.x, na.rm = TRUE))
## [1] 3.0 5.5
```

하지만 map 함수들은 `...`를 패스해줄 수 있기 때문에, 더 간단한 form이 가능하다.

``` r
map_dbl(x, mean, na.rm = TRUE)
## [1] 3.0 5.5
```

그림을 보면 더 쉽게 이해가 가능하다. <br /> `f` 다음에 나오는 어떠한 arguments건 간에, `map()`은 데이터에다가 `f()`를 하고, **그 다음에** 넣어준다. ![그림3](https://d33wubrfki0l68.cloudfront.net/e1b3536a7556aef348f546a79277125c419a5fdc/0c0a1/diagrams/functionals/map-arg.png)

중요한 점은, 이 패스되는 arguments들은 decompose되지 않는다는 것이다. <br /> `map()`에서 첫 번째로 넣어지는 arguments들은 decompose되었는데, 패스되는 additional arguments들은 decompose안된다. <br /> 그림을 보고 제대로 이해하자. <br /> ![그림4](https://d33wubrfki0l68.cloudfront.net/a468c847ea8aca9a6131492e1e7431f418259eaf/ce4e0/diagrams/functionals/map-arg-recycle.png)

additional arguments도 decompose되는 것은 Section 9.4.2와 Section 9.4.5에서 배울 것이다. map variants에서.

`map()`에다가 이렇게 additional arguments를 패스해놓는 것이랑, anonymous 함수를 이용해서 거기에다가 extra argument를 넣어놓는 것이랑은 미묘하게 다르다. <br /> 전자는 `map()`이 실행될 때 한 번만 evaluate되고, <br /> 후자는 `f()`가 실행될 때마다 새롭게 evaluate된다. `map()`이 불러질 때 한번만 evaluate되는 것이 아니라. <br /> 다음의 예를 보면 이해가 된다.

``` r
plus <- function(x, y) x + y

x <- c(0, 0, 0, 0)
map_dbl(x, plus, runif(1))
## [1] 0.8441357 0.8441357 0.8441357 0.8441357
map_dbl(x, ~ plus(.x, runif(1)))
## [1] 0.03506259 0.38196867 0.71835723 0.37133646
```

### 9.2.4 Argument names

여기서는 argument names를 써주자 하는 것, 그리고 왜`x`나 `f` 대신에 `.x`, `.f`를 사용하는지를 알게 된다.

다이어그램에서는, 전반적인 구조를 이해하라고 argument names를 다 생략해놨었다. <br /> 하지만 작성하는 코드에서는, 읽기 쉬게끔, full names를 다 써놓기를 권장한다. <br /> 예를 들어, `map(x, mean, 0.1)`은 완벽하게 유효한valid 코드이지만, `mean(x[[1]], 0.1)`이란 뜻이고, <br /> 읽는 사람은 `mean()`의 두 번째 argument는 `trim`이란 걸 기억해야 한다. <br /> 읽는 사람이 불필요하게 고생하지 않도록, `map(x, mean, trim = 0.1)`이라고 써주자. <br />

이게 `map()`의 arguments들이 좀 이상해보이는 이유다. <br /> `x`, `f`라고 안 쓰고, `.x`, `.f`라고 쓴다. <br /> 왜 이렇게 하는지 예를 들어서 보여주겠다. <br /> 위에 `map()`이 어떻게 implement되었는지를 간단하게 보인, `simple_map()`을 다시 봐보자.

``` r
simple_map <- function(x, f, ...) {
  out <- vector("list", length(x))
  for (i in seq_along(x)) {
    out[[i]] <- f(x[[i]], ...)
  }
  out
}
```

그리고 `bootstrap_summary()`라는 함수가 있다.

``` r
bootstrap_summary <- function(x, f) {
  f(sample(x, replace = TRUE))
}
```

``` r
simple_map(mtcars, bootstrap_summary, f = mean)
```

    ## Error in mean.default(x[[i]], ...): 'trim' must be numeric of length one

왜 이런 에러가 발생하는걸까? <br /> `simple_map()`을 호출한다는건 `simple_map(x = mtcars, f = mean, bootstrap_summary)`이랑 같은 것이고, <br /> named 매칭이랑 positional 매칭이랑 충돌한다.

purrr 함수들은 이러한 충돌을, 흔히 사용하는 `x`, `f` 대신에 `.x`, `.f`를 사용함으로써, 가능성을 줄였다. <br /> 물론, 이러한 테크닉은 완벽하진 않다. (여전히 `.x`, `.f`를 이용하는 함수도 있을테니깐) <br /> 하지만 이러면 99퍼센트는 문제를 회피할 수 있다. <br /> 여전히 남은 1퍼센트의 경우에 있어서는 anonymous 함수를 사용하자.

<p class="comment">
<strong>base R에서는</strong> <br /> <code>...</code>를 사용하는 base 함수들은, 이런 원치 않는 argument 매칭을 피하기 위해 여러가지 naming convention을 사용한다. <br /> 1. apply 함수들은 대부분 대문자를 사용함. ex) <code>X</code>, <code>FUN</code> <br /> 2. <code>transform()</code>는 좀 더 이국적인exotic 접두사prefix를 사용한다. <code>\_</code> <br /> 이러면 이름이 더 비문법적non-syntatic이 되기 때문에, <code>`</code>로 감싸줘야한다. Section 2.2.1에서 나왔던대로. <br /> 이러면 원치않은 매칭이 일어날 일이 매우 줄어든다. <br /> 3. 다른 functionals들,`uniroot()`나`optim()\`은 충돌을 피하기 위한 아무런 노력도 하지 않지만, 특별히 만들어진 함수들을 사용하기 때문에, <br /> 충돌이 일어날 일이 적다.
</p>
### 9.2.5 Varying another argument

이 때까지, map()의 첫 번째 argument는 항상 function의 첫 번째 argument가 되었다. <br /> 하지만, 만약에 첫 번째 argument는 일정constant하고, different argument가 변하기vary를 바란다면? <br /> 그림으로 따지자면 다음과 같이. ![그림5](https://d33wubrfki0l68.cloudfront.net/6d0b927ba5266f886cc721ae090afcc5e872a748/f8636/diagrams/functionals/map-arg-flipped.png "width:30px")

직접적으로 하는 방법은 없다. 하지만 대신에, 2가지 트릭을 이용해 할 수 있다. <br /> 이 예를 좀 묘사illustrate해보자면, 좀 일반적이지 않은 그런 값들이 있는 벡터가 있고, <br />     mean을 구하는데 있어, triming에다가 다른 값들을 넣어 탐구를 해보고 싶다치자. <br /> 이 경우에, `mean()`의 first argument는 일정하고constant, `trim`이라는 second argument가 vary하기를 원하는 거다.

``` r
trims <- c(0, 0.1, 0.2, 0.5)
x <- rcauchy(1000)
```

1.  가장 간단한 방법은, argument order를 rearrange하는 anonymous function을 사용하는 것.

``` r
map_dbl(trims, ~ mean(x, trim = .x))
## [1] -7.24640036  0.05732821  0.05482457  0.03404175
```

그런데 얘는 x와 .x를 둘 다 사용하고 있기 때문에, 헷갈린다. <br /> `~` 사용하는 걸 포기함으로써 좀 더 깔끔하게 만들 수 있다.

``` r
map_dbl(trims, function(trim) mean(x, trim = trim))
## [1] -7.24640036  0.05732821  0.05482457  0.03404175
```

2. 너무 알고 있는게 많아서, R의 flexible argument 매칭 룰을 사용할 수도 있다. <br /> 예를 들어, `mean(x, trim = 0.1)`을 `mean(0.1, x = x)`라고 쓸 수도 있는데, 이렇게 `map_dbl()`을 call할 수 있다.

``` r
map_dbl(trims, mean, x = x)
## [1] -7.24640036  0.05732821  0.05482457  0.03404175
```

그런데 이 방법technique은 추천하지 않는다. <br /> 왜냐하면, 독자가 .f의 argument order와 R의 argument matching rules를 둘 다 이해하고 있다는 가정 하에 하는 것이기 때문.

3. Section 9.4.5에서 배울 `pmap()`을 쓰는 방법도 있다.

9.3 Purrr style
---------------

더 많은 map 변형들variants을 탐구해보기 전에, <br /> 상당히 현실적인 문제들을, 여러 개의 purrr 함수들을 사용해서 어떻게 해결하는지 봐보자.

이 toy example에서, 각 subgroup에 대한 model을 fitting하고, 모델의 계수를 추출해보자. <br /> `mtcars` 데이터를 cylinders의 개수에 따라, `base::split()` 함수를 이용해, 그룹으로 쪼갤 것이다.

``` r
by_cyl <- split(mtcars, mtcars$cyl)
```

이러면 3개의 데이터 프레임들을 갖고 있는 list가 생긴다. 각각 4, 6, 8개의 cylinders를 갖고 있는 차들.

이제 각각에 대해 linear model을 fit하고, 두 번째 계수, 즉, slope를 추출하고 싶다고 하자. <br /> 다음은 purrr을 통해 어떻게 해야할지를 보여준다.

``` r
by_cyl %>% 
  map(~ lm(mpg ~ wt, data = .x)) %>% 
  map(coef) %>% 
  map_dbl(2) # intercept를 얻고 싶었다면 1을 넣겠지.
##         4         6         8 
## -5.647025 -2.780106 -2.192438
```

(`%>%`라는 pipe를 본 적이 없다면, Section 6.3에 설명되어 있다.)

각 line이 하나의 step을 요약encapsulate하고 있기 때문에, 쉽게 이 functional 뭐하고 있는지를 구분할 수 있고, <br /> purrrr helpers들이 우리가 각 step에서 뭘 해야하는지를 간결하게 묘사하고 있다. <br /> 그래서 위 코드가 쉽다. <br /> and the purrr helpers allow us / to very concisely describe / what to do in each step.

base R로는 이 문제를 어떻게 풀 수 있을까? <br /> 분명히 위의 각 purrr 함수를 동등한 base 함수로 대체할 수 있다.

``` r
by_cyl %>%
    lapply(function(data) lm(mpg ~ wt, data = data)) %>%
    lapply(coef) %>%
    vapply(function(x) x[[2]], double(1))
##         4         6         8 
## -5.647025 -2.780106 -2.192438
```

하지만, pipe를 쓰고 있기 때문에 진짜 base R만을 쓴 것은 아니다. <br /> 순수하게 base R로만 하려고 하면, 중간과정을 넣어야하고, 각 step에서 뭔가를 조금 더 해야한다.

``` r
models <- lapply(by_cyl, function(data) lm(mpg ~ wt, data = data))
vapply(models, function(x) coef(x)[[2]], double(1))
##         4         6         8 
## -5.647025 -2.780106 -2.192438
```

혹은, 물론 for 루프를 사용할 수도 있다.

``` r
intercepts <- double(length(by_cyl))
for (i in seq_along(by_cyl)) {
    model <- lm(mpg ~ wt, data = by_cyl[[i]])
    intercepts[[i]] <- coef(model)[[2]]
}
intercepts
## [1] -5.647025 -2.780106 -2.192438
```

purrr에서, apply 함수로, for 루프로, 옮겨가면서 더 많은 iteration을 해야한다는 걸 볼 수 있다. <br /> purrr에서는 map(), map(), map\_dbl()으로 총 3번 iterate <br /> apply 함수들은 lapply(), vapply()로 총 2번 iterate <br /> for 루프문을 이용했을 때는 한 번 iterate

각 스텝이 더 많고(줄이 많고), 간단할수록 코드를 더 이해하기 쉽고 나중에 수정하기도 편해서 좋다.

9.4 Map variants
----------------

map()의 주요 변형primary variants으로는 총 23개가 있다. <br /> 이 때까지 5개를 배웠고, (`map()`, `map_lgl()`, `map_int()`, `map_dbl()`, `map_chr()`) <br /> 이 말인즉슨 18개 남았다는 뜻. <br /> 하지만 다 따로 배워야하는 건 아니고, 5개의 새로운 아이디어만 배우면 되게끔 purrr을 디자인했다.

-   output이 input과 같은 타입이 되도록, `modify()`
-   2개의 inputs에 대해 iterate하도록, `map2()`
-   index를 이용해서 iterate하도록, `imap()`
-   아무것도 return하지 않도록, `walk()`
-   몇 개의 input이 되든 상관없이 iterate하도록, `pmap()`

map family의 함수들은, orthogonal한 input과 output을 갖고 있는데, <br /> 이 말인즉슨 모든 family를 inputs를 행에, outputs를 열에 두고서, 행렬matrix로 조직organise해볼 수 있다는 거다.

row에 있는 아이디어를 이해하고 나면, 칼럼에 있는 것과 결합시킬 수 있다. <br /> column에 있는 아이디어를 이해하고 나면, row에 있는 것과 결합시킬 수 있다.

|                      |   List   | Atomic            | Same type   |    Nothing|
|----------------------|:--------:|-------------------|-------------|----------:|
| One argument         |  `map()` | `map_lgl()`, ...  | `modify()`  |   `walk()`|
| Two arguments        | `map2()` | `map2_lgl()`, ... | `modify2()` |  `walk2()`|
| One argument + index | `imap()` | `imap_lgl()`, ... | `imodify()` |  `iwalk()`|
| N arguments          | `pmap()` | `pmap_lgl()`, ... | -           |  `pwalk()`|
