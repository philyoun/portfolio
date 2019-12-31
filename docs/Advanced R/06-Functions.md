6.1 Introduction
----------------

------------------------------------------------------------------------

6.2 Function fundamentals
-------------------------

R에서 함수를 이해하기 위해서는, 2개의 아이디어를 이해해야 한다. <br /> 1. 함수는 3개의 요소components로 쪼갤 수 있다. <br /> 요소arguments, 본체body 그리고 environment <br /> 얘는 그냥 env라고 해야지.

모든 규칙들rules은 예외가 있다. <br /> 그리고 이 1번의 경우엔, "원시primitive" base 함수들이 있다. <br /> 이것들은 온전히 C 언어로만 고안implement되었다.

1.  함수Function는 오브젝트다. 벡터vector도 오브젝트이듯이.

### 6.2.1 Function components

함수는 3가지 파트가 있다. <br /> 1. `formals()`: 어떻게 함수를 호출할건지 컨트롤할 수 있는 요소arguments들의 리스트

1.  `body()`: 함수 안의 코드. the code inside the function

2.  `environment()`: 어떻게 함수가, 이름들과 연관되어 있는 값을 찾는지, 이걸 결정하는 데이터 구조

formals와 body는, 함수를 만들 때 명백하게 정해줘야explicitly specify하는데, <br /> environment는, 어디에 만드냐에 따라 implicit하게 specify 된다.

function env는 항상 존재한다. <br /> 하지만, 함수가 global env에 정의되어 있지 않을때에만 프린트된다. <br /> it is only printed when the function isn't defined in the global env.

``` r
f02 <- function(x, y) {
    # A comment
    x + y
}

formals(f02)
## $x
## 
## 
## $y
body(f02)
## {
##     x + y
## }
environment(f02)
## <environment: R_GlobalEnv>
```

함수를, 아래의 다이어그램처럼 그려볼 것이다. <br /> 왼쪽의 검은 점은 env다. 오른쪽 2개의 블락들blocks은 함수의 인자들arguments이다. <br /> body는 안 그릴 것이다. <br /> 왜냐하면 보통 양이 많고, 함수의 shape을 이해하는데 별 도움이 안 되기 때문. <br /> ![Figure 6.1](https://d33wubrfki0l68.cloudfront.net/de34ef3939642ec68b2b78dc310f3baa22d12106/ac3f3/diagrams/functions/components.png){ width = 50% }

R의 모든 오브젝트들과 마찬가지로, 함수들도 몇 개든 상관없이 추가적인 특성들`attributes()`을 가질 수 있다. <br /> base R에 의해 사용된 하나의 특성은 `srcref`인데, source reference의 줄임말이다. <br /> 함수를 만드는데 사용된 소스코드를 알려준다.

`srcref`는 프린팅하는데 사용된다. <br /> 왜냐하면, `body()`와는 다르게, 코드 코멘트들이랑 다른 포매팅들도 갖고 있기 때문.

``` r
attr(f02, "srcref")
## function(x, y) {
##  # A comment
##  x + y
## }
```

### 6.2.2 Primitive functions

함수는 3개의 파트를 갖는다는 이 규칙에는, 하나의 예외가 있다. <br /> 원시primitive 함수들, `sum()`이나 `[`와 같은 것들은, C 코드를 직접적으로 호출한다.

``` r
sum
## function (..., na.rm = FALSE)  .Primitive("sum")
`[`
## .Primitive("[")
```

이것들은, 타입 `builtin`이나 타입 `special`을 갖고 있다.

``` r
typeof(sum)
## [1] "builtin"
typeof(`[`)
## [1] "special"
```

이 함수들은 애초에 R이 아닌, C로 존재하기 때문에, <br />   이것들의 `formals()`, `body()`, `environment()`는 전부 `NULL`이다.

``` r
formals(sum)
## NULL
body(sum)
## NULL
environment(sum)
## NULL
```

원시 함수들은 base 패키지에서만 발견된다. <br /> 분명한 퍼포먼스적인 이점이 있지만, 대가가 있다. <br /> 더 작성하기가 힘들다. harder to write. <br /> 이러한 이유에서, R-코어팀은 다른 옵션이 없는게 아니라면, 이 방법을 쓰지 않는다.

### 6.2.3 First-class functions

R 함수가, 그 자체로 오브젝트라는걸 이해하는건 매우 중요하다. <br /> 이건 보통 "first-class functions"라고 부르는 언어 속성language property다.

많은 다른 언어들과는 다르게, 함수를 정의하고 이름 붙이는데 다른 특별한 문법syntax이 없다. <br /> 그냥 `function()`을 이용해서 함수 오브젝트function object를 만들고, `<-`를 이용해서 이름 붙이면 된다.

<details> <summary>느낌</summary> 프로그래밍 언어가 퍼스트클래스 함수를 지원하면, 변수에 함수를 할당도 할 수 있고, 인자로써 다른 함수에 전달할 수도 있고, 함수의 리턴값으로도 쓸 수 있고. </details> <br /> <br />

``` r
f01 <- function(x) {
  sin(1/ x ^ 2)
}
```

<img src="https://d33wubrfki0l68.cloudfront.net/5db72a270ade61a321dfc2519e6fb0f56370609e/807cb/diagrams/functions/first-class.png" alt="Figure 6.2" style="width:50.0%" />

거의 항상, 함수를 만들고 나면 이름을 붙이겠지만, 이 이름을 붙이는 binding step이 꼭 요구되는 건 아니다. <br /> 이름을 안 붙이기로 결정했다면, **익명 함수anonymous function**을 만든 것이다. <br /> 이름을 꼭 붙여야 할 필요가 없는 경우라면, 상당히 유용하다.

``` r
lapply(mtcars, function(x) length(unique(x)))
Filter(function(x) !is.numeric(x), mtcars)
integrate(function(x) sin(x) ^ 2, 0, pi)
```

마지막 옵션은, 리스트에다가 함수들을 넣는 것이다. <br /> (아니 리스트에다 함수 넣는 것도 되는건 처음 알았네)

``` r
funs <- list(
    half = function(x) x / 2,
    double = function(x) x * 2
)

funs$half(10)
## [1] 5
funs$double(10)
## [1] 20
```

R에서, 종종 **closures**라는 함수를 볼 것이다. <br /> 이건, R 함수들이 자기 자신의 env를 캡쳐한다는 사실을 반영한 것이다. <br /> [Section 7.4.2](https://blog-for-phil.readthedocs.io/en/latest/Advanced%20R/07-Environments/#742-the-function-environment)에서 더 배우게 될 것이다.

### 6.2.4 Invoking a function

보통 함수를, 함수 이름에다 괄호를 열고, 인자들arguments을 넣고, 괄호를 닫는 식으로 호출한다. <br /> 예를 들어서, `mean(1:10, na.rm = TRUE)` 이렇게. <br /> 그런데 만약에 데이터 구조에 인자들을 이미 갖고 있는 경우에는 어떻게 할 수 있을까? <br /> 예를 들어서,

``` r
args <- list(1:10, na.rm = TRUE)
```

이렇게 갖고 인자들을 갖고 있는 것임.

`do.call()`을 쓰면 된다. <br /> 이 함수는 2개의 인자들arguments을 받는다. <br /> 하나는 호출할 함수 이름, 다른 하나는 함수 인자들을 가지고 있는 리스트.

``` r
do.call(mean, args)
## [1] 5.5
```

이 아이디어를 Section 19.6에서 다시 볼 것이다.

### 6.2.5 Exercises

------------------------------------------------------------------------

6.3 Function composition
------------------------

함수 합성.

base R은, 여러 개의 함수 호출을 합성하는데 있어, 2가지 방법을 제공한다. <br /> 예를 들어, `sqrt()`와 `mean()`을 바탕으로, 모표준편차population standard deviation를 계산하고 싶다치자.

``` r
square <- function(x) x ^ 2
deviation <- function(x) x - mean(x)
```

① 함수 호출들을 중첩nest시킬 수도 있고,

``` r
x <- runif(100)
sqrt(mean(square(deviation(x))))
## [1] 0.2879764
```

② 아니면 중간중간 결과물들을 변수로 저장할 수도 있다.

``` r
out <- deviation(x)
out <- square(out)
out <- mean(out)
out <- sqrt(out)
out
## [1] 0.2879764
```

위 2개는 base R이고, <br /> ③ magrittr 패키지([Bache and Wickham 2014](https://magrittr.tidyverse.org/))는 3번째 옵션을 제공한다. <br /> 이항 연산자binary operator인 `%>%`는, 파이프pipe라고 부르고, "and then"이라고 발음한다.

``` r
library(magrittr)

x %>%
    deviation() %>%
    square() %>%
    mean() %>%
    sqrt()
## [1] 0.2879764
```

`x %>% f()`는, `f(x)`와 같은 것이다. <br /> `x %>% f(y)`는, `f(x, y)`와 같은 것이다. <br /> 파이프를 사용하면 낮은 수준의 데이터 흐름이 아니라, 높은 수준의 함수 구성에 집중할 수 있다. <br /> 초점은 수정 된 것(명사)이 아니라, 수행중인 것(동사)에 있다. <br /> The pipe allows you to focus on the high-level composition of functions rather than the low-level flow of data; <br /> the focus is on what's being done(the verbs), rather than on what's being modified(the nouns). <br /> 이러한 스타일은 하스켈이나 F\#에서는 흔하다. <br /> 이게 magrittr을 만드는데 있어 영감이 되었고, Forth나 Factor라는 프로그래밍 언어의 디폴트 스타일이다. <br /> (둘 다 이번에 처음 알게 된 프로그래밍 언어다.)

위에 소개한 3개의 옵션들은 각각 장단점이 있다.

1.  Nesting은, (`f(g(x))` 같은) 간결하고, 짧은 시퀀스에 최적화되어있다. <br /> 하지만 길이가 길어질수록 읽기가 어려워진다. 왜냐하면 안에서부터 밖으로, 오른쪽에서부터 왼쪽으로 읽어야하기 때문. <br /> 결과적으로, 인자들arguments이 퍼지면서 Dagwood sandwich 문제를 발생시킬 수 있다. <br /> 별 대단한 문제는 아니고, 그냥 길어짐에 따라 함수랑 인자들이랑 거리가 멀어진다. 이게 진짜 다임.

2.  중간중간 결과물을 저장하는 것은, (`y <- f(x); g(y)` 이런 식) <br /> 중간 오브젝트들intermediate objects에 이름을 붙여줘야 한다. <br /> 만약 이 오브젝트들이 중요하다면 강점이 될 수 있겠는데, 그렇지 않다면 약점이다.

3.  Piping은, (`x %>% f() %>% g()`) 그냥 그대로 읽으면 된다는 점에서 강점을 갖고 있다. <br /> 하던대로 왼쪽에서 오른쪽으로 읽으면 되고, 중간 오브젝트들에 이름을 붙일 필요도 없다. <br /> 하지만 하나의 오브젝트만을 선형 변환 시퀀스linear sequence of transformation로 사용할 수 있다. <br /> 그리고 magrittr이라는 3번째 패키지를 필요로 하고, 독자가 piping을 알고 있어야 한다는 문제가 있다.

대부분의 코드는 위 3가지 스타일의 조합을 사용한다. <br /> 그때그때 필요에 따라 3개 이것저것 쓴다. <br /> 그래도, Piping은 데이터 분석 코드에 좀 더 흔하다. <br /> 분석이라는게 하나의 오브젝트(예를 들어 데이터 프레임이나 plot)에 변형 시퀀스를 적용하는 것이다 보니깐. <br /> 패키지들에는 piping을 별로 안 쓴다. <br /> 이게 나쁜 아이디어라서가 아니라, 별로 내추럴하지 않아서.

------------------------------------------------------------------------

6.4 Lexical scoping
-------------------

Chapter 2에서, 할당assignment에 대해 배웠다. <br /> 이름name에다가 값value을 binding하는 행동. <br /> 여기서는 **scoping**에 대해 다룰 것인데, 이름과 연관associate된 값을 찾는 행동임.

scoping의 기본적인 룰은 꽤나 직관적이다. <br /> 대놓고 배우지는 않았더라도, 모르는 사이에 이미 어느 정도 알고 있을수도 있다. <br /> 예를 들어, 다음의 코드는 10과 20 중 어떤 값을 return할까?

``` r
x <- 10
g01 <- function() {
  x <- 20
  x
}

g01()
```

이 섹션에서는, scoping의 형식적인 룰들과 사소한 디테일들에 대해 배울 것이다. <br /> scoping에 대해 깊이 이해하고 나면, 좀 더 advanced function programming 툴들을 사용할 수 있을 것이고, <br /> R 코드를 다른 언어들로 번역할 수 있는 툴들을 작성할 수 있게 해준다.

R은 **lexical scoping**을 사용한다. <br /> 함수가 어떻게 정의되었는지를 바탕으로 이름name의 값value을 찾아본다. <br />   어떻게 호출되었는지가 아니라. <br /> R looks up the values of names based on how a function is defined, not how it is called.

여기서 "Lexical"은 word나 vocabulary라는 뜻이 아니다. <br /> 이건 기술적인 CS 단어다. <br />   scoping rule이, run-time 구조가 아닌 parse-time을 사용한다는. <br /> It’s a technical CS term that tells us that the scoping rules use a parse-time, rather than a run-time structure.

<details> <summary>parse-time run-time</summary> 여기서 parse-time이랑 run-time이 무슨 뜻인지 한참 찾아봤는데, <br />   parse-time이라는건 위에 how a function is defined, 함수가 어떻게 정의되었는지와 관련이, <br />   run-time이라는건 위에 how a function is called, 함수가 어떻게 호출되었는지와 관련이 있음. <br /> 그래서 lazy evaluation같이, 받아만 놓고 evaluate는 호출되었을 때만 하면 그게 run-time이랑 연관이, <br />   입력한 즉시 evaluate가 되는 그런건 parse-time이랑 연관이 있는듯. </details> <br /> <br />

R의 lexical scoping은, 4개의 주요한 규칙들이 있다. <br /> 1. Name masking <br /> 2. Functions versus variables <br /> 3. A fresh start <br /> 4. Dynamic lookup

### 6.4.1 Name masking

lexical scoping의 기본 원리, <br />   함수 안에서 정의된 이름name들은, 밖에서 정의된 이름들을 가린다.mask <br /> 그러니깐 밖에서 정의된 이름들이 함수 안에서 정의된 걸로 덮어씌워진다는 것. <br /> 하지만, 덮어씌운다고 한다면 override라고 했을텐데 mask라고 했으니 '가린다'라고 번역했다. <br /> 다음의 예를 보자.

``` r
x <- 10
y <- 20
g02 <- function() {
  x <- 1
  y <- 2
  c(x, y)
}

g02()
## [1] 1 2
```

만약 이름이 함수 안에 정의되어 있지 않으면, R은 한 레벨 위를 찾아본다.

``` r
x <- 2
g03 <- function() {
  y <- 1
  c(x, y)
}

g03()
## [1] 2 1
```

그리고 이건 이전의 y값을 바꾸지는 않음

``` r
y
## [1] 20
```

어떤 함수가 다른 함수 안에서 정의되어 있다해도, 같은 규칙이 적용된다. <br /> 먼저, R은 현재 함수의 안에서 찾아보고, <br /> 다음으로 함수가 정의된 곳을 찾아보고(없으면 한 레벨 위씩 올라가서 global env까지), <br /> 마지막으로 다른 로드된 패키지들에서 찾아본다.

다음의 코드는 어떤 결과물이 나올지를 예상해보자.

``` r
x <- 1
g04 <- function() {
    y <- 2
    i <- function() {
      z <- 3
      c(x, y, z)
    }
    i()
}

g04()
```

같은 규칙이, 다른 함수들로 만들어진 함수들에도 적용된다. <br /> 난 이걸 찍어낸 함수manufactured function라고 부른다. <br /> 이건 10장의 주제다.

### 6.4.2 Function versus variables

R에서는, 함수도 일반적인 오브젝트이다. <br /> 이 말인즉슨, 위에서 설명했던 scoping rule이 함수에도 똑같이 적용된다는 말이다.

``` r
g07 <- function(x) x + 1
g08 <- function() {
  g07 <- function(x) x + 100
  g07(10)
}

g08()
## [1] 110
```

하지만, 만약에 함수와, 함수가 아닌 것이, 똑같은 이름을 갖는다면,(물론 둘은 서로 다른 env에 있어야겠지만) <br />   이 규칙을 적용하는 것이 조금은 더 복잡해진다. <br /> However, when a function and a non-function share the same name (they must, of course, reside in different environments), <br /> applying these rules gets a little more complicated.

함수 호출에서 이름을 사용할 때, R은 그 값을 찾는데 있어 함수가 아닌 오브젝트들은 애초에 무시한다. <br /> 예를 들어 아래의 코드에서, `g09`는 2개의 다른 값들을 갖는다.

``` r
g09 <- function(x) x + 100
g10 <- function() {
  g09 <- 10
  g09(g09)
}

g10()
## [1] 110
```

그러니깐 `g09()`를 찾는데 있어 함수가 아니면 애초에 고려를 하지도 않아서 함수 안의 10의 값을 갖는 `g09`를 제끼고, <br /> 함수 밖의 `g09()`라는 함수를 잘 찾는 것.

물론 분명히 말하건대, 다른 것들에 대해 같은 이름을 사용하는 것은 헷갈리고, 피하는 것이 가장 좋다!

### 6.4.3 A fresh start

함수 호출invocation을 여러 번 하는데 있어, 값들values에는 무슨 일이 일어날까? <br /> What happens to values between invocations of a function? <br /> 아래의 예를 보자. 아래의 함수를 처음으로 실행하면 무슨 값을 얻을까? 두 번째로 실행할 때는? <br /> (`exists()`를 본 적이 없다면, 그 이름으로 된 변수가 존재한다면 `TRUE`를 return하고 아니면 `FALSE`를 return)

``` r
g11 <- function() {
  if (!exists("a")) {
    a <- 1
  } else {
    a <- a + 1
  }
  a
}

g11()
g11()
```

`g11()`이 항상 같은 값을 return한다는 것에 놀랄 수도 있다. <br /> 함수가 호출될 때마다, 실행을 호스팅하기 위해 새로운 env가 만들어지기 때문. <br /> This happens because every time a function is called a new environment is created to host its execution. <br /> 저번 실행 때 무슨 일이 일어났는지를, 함수가 말해줄 방법은 없다는 것. <br /> 각 호출은 완전히 독립적이다. each invocation is completely independent. <br /> 이걸 [Section 10.2.4](https://blog-for-phil.readthedocs.io/en/latest/Advanced%20R/10-Function-Factories/#102-factory-fundamentals)에서 다룰 것이다.

<details> <summary>call과 invocation?</summary> call도 호출이고 invocation도 호출이라고 번역을 하긴했는데, 분명 차이가 있을거 같아서 찾아봤다. <br /> javaScript에서는 구별이 확실히 되는거같은데, <br />   call a function이라고 하면 직접적으로 실행을 하는 것이고, <br />   invoke a function이라고 하면 간접적으로 실행을 하는 것인가보다. <br /> Section 6.2.4 Invoking a function에서, <code>do.call()</code>처럼 <code>mean()</code>을 호출하는 것도 일종의 간접적인 방법으로 실행하는 거라고 생각해봐도 되겠다. </details> <br /> <br />

### 6.4.4 Dynamic lookup

Lexical scoping은 언제가 아니라, 어디서 값을 찾아볼지를 정하는 것이다. <br /> Lexical scoping determines where, but not when to look for values.

R은 함수가 만들어졌을 때가 아니라, 실행될 때 값을 찾아본다. <br /> R looks for values when the function is run, not when the function is created.

실행될 때, 그리고 어디서. <br /> 이 2개를 종합해보면, 함수의 output은 함수 env 외부 오브젝트들에 따라 달라질 수 있다는 것. <br /> Together, these two properties tell us that the output of a function can differ depending on the objects outside the function's environment.

``` r
g12 <- function() x + 1
x <- 15
g12()
## [1] 16

x <- 20
g12()
## [1] 21
```

이러한 행동은 꽤 짜증날 수 있다. <br /> 코드에 스펠링 실수를 했다면, 함수를 생성할 때 아무런 에러 메세지를 얻지 못한다. <br /> 그리고 실수를 하지 않았어도, global env에 정의된 변수들에 따라, 함수를 실행할 때 아무런 에러 메세지를 얻지 못할수도 있다.

이러한 문제를 감지하기 위해, `codetools::findGlobals()`를 사용하자. <br /> 이 함수는 함수 내의 모든 외부 종속성들dependencies(unbound symbol)을 나열한다. <br /> This function lists all the external dependencies (unbound symbols) within a function:

<details> <summary>unbound symbols?</summary> 그러니깐 name에는 value가 associate되어있는게 일반적인데, value가 없는 name, 예를 들어 <code>+</code>를 unbound symbol이라고 하는듯 </details> <br /> <br />

``` r
codetools::findGlobals(g12)
## [1] "+" "x"
```

이 문제를 해결하기 위해서, 함수의 env를 `emptyenv()`로 manual하게 바꿀 수 있다. <br /> 아무것도 없는 env임.

``` r
environment(g12) <- emptyenv()
g12()
## Error in x + 1: 함수 "+"를 찾을 수 없습니다
```

문제와 해결법을 보고나면, 왜 이런 원치않아보이는 행동이 존재하는지를 알게 된다. <br /> R은 하나부터 끝까지, 뭘 찾든간에, lexical scoping에 의존하고 있다. <br /> `mean()`과 같이 명백해보이는 것들에서부터 시작해서, 좀 덜 명백해보이는 `+`나 `{` 같은 것들까지. <br /> 이것은 R의 scoping rule에 좀 아름다운 단순함을 부여한다.

### 6.4.5 Exercises

------------------------------------------------------------------------

6.5 Lazy evaluation
-------------------

R에서, 함수 인자들function arguments은 **lazily evaluated**된다. <br /> 접근되었을 때만 evaluate된다는 것. <br /> 예를 들어, 이 코드는 에러가 나오지 않는다. 왜냐하면 `x`는 전혀 사용되지 않았기 때문.

``` r
h01 <- function(x) {
  10
}

h01(stop("This is an error!"))
## [1] 10
```

그러니깐, 원래 `stop()` 함수는 에러를 내는 함수임. <br /> 그래서 `h01(stop("This is an error!"))` 하면 에러가 나와야 할 것 같지만, <br /> `h01()`이라는 함수는 `x`를 받는데 body에는 `x`가 없으니 evaluate할 필요가 없어서, <br /> 에러가 안 나온다. 이런 얘기.

이건, <br />   함수 인자들에, 필요할 때만 evaluate되는, 잠재적으로 값비싼 계산을 포함시키는 것과 같은 일을 할 수 있기 때문에, <br /> 중요한 기능feature이다.

### 6.5.1 Promises

lazy evaluation은, **promise**라고 부르는 데이터 구조data structure로 작동power된다. <br /> promise는, 덜 흔하게는 thunk라고 부른다. <br /> 이건 R을 흥미로운 프로그래밍 언어로 만드는 기능 중 하나다. <br /> (Section 20.3에서 promises에 대해 다시 다룰 것이다.)

promise는 3개의 요소components들을 가지고 있다. <br /> ① expression, `x + y`와 같은, delayed computation을 발생시킨다. <br /> ② environment는 expression이 evaluate되는 장소다. 즉, environment는 함수가 호출되는 곳이다. <br /> 그래서 다음의 함수는 101이 아닌, 11을 return한다.

``` r
y <- 10
h02 <- function(x) {
  y <- 100
  x + 1
}

h02(y)
## [1] 11
```

그리고, 함수 호출 안에서 할당assignment을 하면, 변수는 함수 안이 아닌, 밖에서 bound된다. <br /> `h02(y <- 1000)`는 무슨 값이 나올까? 변수가 함수 안이 아닌 밖에서 bound된다고 했으니,

``` r
y <- 1000
h02 <- function(x) {
    y <- 100
    x + 1
}

h02(y)
```

와 같은 것이다. <br /> 즉, `h02(y <- 1000)`은 1001이 나온다.

``` r
h02(y <- 1000)
## [1] 1001
y
## [1] 1000
```

③ value는 promise가 처음으로 접근되었을 때 계산되고 캐시cache되는 것. <br /> expression이 특정한 env에서 evaluate되었을 때, <br />   value는, promise가 처음 access되었을 때 계산되고 캐시cache된다. <br /> A value, which is computed and cached the first time a promise is accessed when <br /> the expression is evaluated in the specified environment.

이래서 promise는 최대 한 번 evaluate되고, 다음의 예에서 "Calculating..."은 한 번만 보게 된다.

``` r
double <- function(x) {
  message("Calculating...")
  x * 2
}

h03 <- function(x) {
  c(x, x)
}

h03(double(20))
## Calculating...
## [1] 40 40
```

R 코드로는 promises를 조작manipulate할 수 없다. <br /> promises는 마치 퀀텀 상태quantum state와 비슷하다. <br /> R 코드로 이걸 검사inspect해보려고 하면, 즉시 evaluation이 실행되어서, promise가 사라지게 될 것이다. <br /> 나중에 Section 20.3에서 quosure에 대해 배울 것이다. <br /> 얘는 promise를 R 오브젝트로 바꿔서, expression과 environment를 쉽게 검사해볼 수 있다.

### 6.5.2 Default arguments

lazy evaluation 덕분에, 디폴트 값들이 다른 인자들arguments로 정의될 수 있다. <br /> 심지어는, 나중에 함수 안에서 정의될 변수들variables로도 디폴트 값을 정의할 수 있다. <br /> 다음의 예를 보면 이해된다.

``` r
h04 <- function(x = 1, y = x * 2, z = a + b) {
    a <- 10
    b <- 100

    c(x, y, z)
}

h04()
## [1]   1   2 110
```

여기서 보면, `y`는 `x`라는 다른 인자로 정의되었고, `z`는 함수 안에서 정의될 변수들 `a`, `b`로 정의되었다.

많은 base R 함수들이 이러한 테크닉을 사용하지만, 나는 이걸 추천하지 않는다. <br /> 이러면 코드가 더 이해하기 힘들기 때문. <br /> 무엇이 return될지 예상하기 위해, <br />   정확히 어떤 순서로 디폴트 인자들arguments이 evaluate되는지를 알아야한다.

그런데, <br /> 디폴트 인자들arguments이 evaluate되는 env와, 사용자가 공급한 인자들이 evaluate되는 env가 살짝 다르다. <br /> The evaluation env is slightly different for default and user supplied arguments. <br /> 디폴트 인자들은 함수 안에서 evaluate된다.

그래서 보기엔 똑같아 보이는 호출들도 다른 값들을 return할 수 있다. <br /> 다음의 극단적인 예를 보자.

``` r
h05 <- function(x = ls()) {
  a <- 1
  x
}
```

``` r
h05()
## [1] "a" "x"
```

이렇게 디폴트 인자들의 경우에는, `ls()`가 함수 안에서 evaluate된다.

``` r
h05(ls())
##  [1] "args"      "deviation" "double"    "f01"       "f02"      
##  [6] "funs"      "g02"       "g03"       "g07"       "g08"      
## [11] "g09"       "g10"       "g12"       "h01"       "h02"      
## [16] "h03"       "h04"       "h05"       "out"       "square"   
## [21] "x"         "y"
```

이렇게 사용자 공급 인자들의 경우에는, `ls()`가 global env에서 evaluate된다.

### 6.5.3 Missing arguments

인자들 값이 user에서 오는지 디폴트에서 오는지는, `missing()`을 이용해서 알 수 있다.

``` r
h06 <- function(x = 10) {
    list(missing(x), x)
}

str(h06())
## List of 2
##  $ : logi TRUE
##  $ : num 10
str(h06(10))
## List of 2
##  $ : logi FALSE
##  $ : num 10
```

`TRUE`라면 디폴트, `FALSE`라면 사용자 공급.

하지만, `missing()`은 덜 사용하는 것이 제일 좋다. missing() is best used sparingly, however. <br /> `sample()` 함수를 예로 들어보자. <br /> 몇 개의 인자들arguments이 요구되는가?

``` r
args(sample)
## function (x, size, replace = FALSE, prob = NULL) 
## NULL
```

`x`와 `size`는 꼭 필요한 것처럼 보인다. <br /> 하지만 만약 `size`를 주지 않으면, `sample()`은 디폴트를 제공하기 위해 `missing()`을 이용한다. <br /> (`sample`을 콘솔 창에 그냥 입력해보면, `missing(size)`가 등장하는 것을 볼 수 있음.) <br /> 이 함수를 내가 다시 써본다면, `size`가 꼭 필요한 것은 아니지만 값을 넣을 수는 있다고, explicit하게 `NULL`을 써 줄 것이다.

``` r
sample <- function(x, size = NULL, replace = FALSE, prob = NULL) {
  if (is.null(size)) {
    size <- length(x)
  }
  
  x[sample.int(length(x), size, replace = replace, prob = prob)]
}
```

만약 여기서 `%||%`라는 삽입 연산자infix operator를 사용해서, 위에 새롭게 쓴 `sample()`을 더 간단하게 만들 수 있다. <br /> (삽입 연산자라는건 기호의 왼쪽 오른쪽에 뭐가 있다는 거임. 그러니깐 `3 + 4`처럼. `+`도 infix operator임. Section 6.8에서 배우게 됨.) <br /> 이 기호는, 간단하게 말해, `NULL`이라면 오른쪽을 사용하고, `NULL`이 아니라면 왼쪽을 사용한다는 뜻이다.

``` r
`%||%` <- function(lhs, rhs) {
  if (!is.null(lhs)) {
    lhs
  } else {
    rhs
  }
}

sample <- function(x, size = NULL, replace = replace, prob = NULL) {
  size <- size %||% length(x)
  x[sample.int(length(x), size, replace = replace, prob = prob)]
}
```

lazy evaluation 덕분에, 불필요한 계산에 대해 걱정할 필요가 없다. <br /> `%||%`은 `size`가 `NULL`이 아닐때만 evaluate될 것이다.

### 6.5.4 Exercises

------------------------------------------------------------------------

6.6 `...` (dot-dot-dot)
-----------------------

함수들은 특별한 인자argument인 `...`(dot-dot-dot이라고 읽음)을 가질 수 있다. <br /> 이것과 함께라면, 함수는 몇 개든 추가적인 인자들arguments을 가질 수 있다. <br /> 다른 프로그래밍 언어들에서는, 이러한 타입의 인자를 종종 varargs(variable arguments의 줄임말)라고 부른다. <br /> 그리고 이 인자를 사용하는 함수를 variadic이라고 부른다.

다른 함수에게 추가적인 인자들을 전달하는데 `...`을 사용할 수도 있다.

``` r
i01 <- function(y, z) {
  list(y = y, z = z)
}

i02 <- function(x, ...) {
  i01(...)
}

str(i02(x = 1, y = 2, z = 3))
## List of 2
##  $ y: num 2
##  $ z: num 3
```

`..N`으로 시작하는, 특별한 형식form을 사용하면, `...`에서 해당하는 elements를, 포지션position으로 refer해줌.

``` r
i03 <- function(...) {
  list(first = ..1, third = ..3)
}

str(i03(1, 2, 3))
## List of 2
##  $ first: num 1
##  $ third: num 3

i03 <- function(...) {
  list(first = ..1, second = ..3)
}

str(i03(1, 2, 3))
## List of 2
##  $ first : num 1
##  $ second: num 3
```

이것보다 더 유용한건 `list(...)`인데, 얘는 arguments를 evaluate하고 list에 저장하는 것.

``` r
i04 <- function(...) {
    list(...)
}
str(i04(a = 1, b = 2))
## List of 2
##  $ a: num 1
##  $ b: num 2
```

`rlang::list2()`도 확인해봐라. 얘는 쪼개는 것splicing과, 뒤에 쓸데없이 콤마가 붙는 것을 무시하는 것도 지원한다. <br /> 예를 들어, `list(a = 1, b = 2, )` 이렇게 뒤에 ,가 붙어도 무시하고 해준다는 것.

그리고 `rlang::enquos()`는 unevaluated 인자들arguments도 캡쳐해준다. <br /> 얘는 [quasiquotation]()의 주제다.

음... `...`의 2가지 주요한 사용법이 있다.

-   만약에 지금 함수가 다른 함수를 argument로 받는데, <br />   후자의 함수에 추가적인 인자들additional arguments을 전해주고 싶다. 할 때. <br /> 아래의 예에서는, `lapply()`가 `...`를 사용한다. `mean()`이라는 함수에 `na.rm`을 전달하기 위해서.

``` r
x <- list(c(1, 3, NA), c(4, NA, 6))
str(lapply(x, mean, na.rm = TRUE))
## List of 2
##  $ : num 2
##  $ : num 5
```

그러니까, `lapply()`라는 함수가 `mean()`이라는 함수를 argument로 받는데, <br />   `mean()`이라는 함수에 `na.rm`이라는 추가 인자를 전달하기 위해서 `...`을 사용한다. 이 뜻. <br /> `lapply`를 보면 `...`이라는 걸 argument로 받고 있는걸 볼 수 있음.

``` r
lapply
## function (X, FUN, ...) 
## {
##     FUN <- match.fun(FUN)
##     if (!is.vector(X) || is.object(X)) 
##         X <- as.list(X)
##     .Internal(lapply(X, FUN))
## }
## <bytecode: 0x000000001358c038>
## <environment: namespace:base>
```

이 테크닉에 대해 [Section 9.2.3](https://blog-for-phil.readthedocs.io/en/latest/Advanced%20R/09-Functionals/#92-my-first-functional-map)에서 다시 다룰 것이다.

-   만약 너의 함수가 S3 제네릭generic이라면, <br />   메소드들methods이 임의의 추가 인자들arbitrary extra arguments을 받을 수 있게끔 해주고 싶을 수 있다.

예를 들어서, `print()`라는 함수를 예로 들어보자. <br /> 어떠한 타입의 오브젝트를 print하냐에 따라, 다양한 옵션들이 있기 때문에, <br />   어떤 인자들을 받을건지 미리 다 정해놓을 수는 없다. <br /> 그리고 `...`가 각각의 메소드들이 그때그때마다 다른 인자들arguments를 받을 수 있게끔 허락해준다. <br /> 아래의 예를 보면 그때그때 주는 인자들arguments이 다른 걸 볼 있다. <br /> `...`이 이걸 가능하게 해주는 것임. <br /> `print`를 콘솔 창에 쳐보면, 위의 `lapply`와 마찬가지로 `...`가 있는 걸 볼 수 있을 것이다.

``` r
print(factor(letters), max.levels = 4)

print(y ~ x, showEnv = TRUE)
```

이 `...`의 사용법에 대해서는 Section 13.4.3에서 다시 다룰 것이다.

`...`을 사용하는 것에는 2가지 단점이 있다.

-   다른 함수에 인자들arguments을 전달하기 위해 사용할 때에는, <br />   이 인자들이 어디로 가는지 사용자에게 잘 설명해줘야 한다. <br /> 그래서 `lapply()`나 `plot()`같은 함수들로 무엇을 할 수 있는지 이해하기가 힘들어진다.

-   오타난 인자misspelled argument임에도 에러가 발생하지 않는다. 오타를 알아차리기 힘들게 된다.

``` r
sum(1, 2, NA, na_rm = TRUE)
## [1] NA
```

### 6.6.1 Exercises

------------------------------------------------------------------------

6.7 Exiting a function
----------------------

대부분의 함수는 두 가지 방법 중 하나로 exit한다. <br /> ①성공을 나타내는, 값value을 return하거나, 혹은 ②실패를 나타내는, 에러를 나타낸다. <br /> 이 section에서는, <br />   1. 값을 반환하는 것에 대해 다루고(implicit versus explicit, visible versus invisible), <br />   2. 에러에 대해 간략하게 다루어보며, <br />   3. exit handlers를 소개한다. 함수를 exit할 때 코드를 실행하게 해준다.

### 6.7.1 Implicit versus explicit returns

함수가 값value을 return할 수 있는 2가지 방법이 있다. <br /> - Implicit하게, 즉, 마지막으로 evaluate된 expression이 return되는 값이 되는 것임.

``` r
j01 <- function(x) {
    if (x < 10) {
        0
    } else {
        10
    }
}

j01(5)
## [1] 0
j01(15)
## [1] 10
```

-   Explicit하게, 즉, `return()`을 호출해서 쓰는 것임.

``` r
j02 <- function(x) {
    if (x < 10) {
        return(0)
    } else {
        return(10)
    }
}

j02(5)
## [1] 0
j02(15)
## [1] 10
```

### 6.7.2 Invisible values

대부분의 함수들은 눈에 보이게 return한다. <br /> 그냥 interactive context에다가 함수를 호출하면, 결과물을 출력한다. <br /> (콘솔에다가 함수를 호출하면 결과물이 나온단 소리임)

``` r
j03 <- function() 1
j03()
## [1] 1
```

그러나, 마지막 값last value에다가 `invisible()`을 씌워서, 자동적으로 프린트되는 것을 막을 수 있다.

``` r
j04 <- function() invisible(1)
j04()
```

이 값이 진짜로 존재한다는 걸 증명하려면, explicit하게 print하던가, 괄호로 감싸주면 된다.

``` r
print(j04())
## [1] 1


(j04())
## [1] 1
```

혹은, `withVisible()`을 사용해, value랑 보이는지 안보이는지 visibility flag도 return하게끔 할 수 있다.

``` r
str(withVisible(j04()))
## List of 2
##  $ value  : num 1
##  $ visible: logi FALSE
```

invisible하게 return하는 가장 흔한 함수는, &lt;-다.

``` r
a <- 2
(a <- 2)
## [1] 2
```

이게 체인 할당chain assignments이 가능한 이유다.

``` r
a <- b <- c <- d <- 2
```

일반적으로, side effect 때문에 호출되는 모든 함수들은, invisible value를 return해야 한다. <br /> 그런 함수들의 예를 들자면 `<-`, `print()`, `plot()` 같은 것들, <br /> 그리고 return하는 invisible value는 보통 첫 번째 인자의 값.

### 6.7.3 Errors

함수가 할당된 작업assigned task을 완수할 수 없다면, `stop()`과 함께 에러가 나온다. <br /> `stop()`은 즉시 함수의 실행을 종료한다.

``` r
j05 <- function() {
    stop("I'm an error")
    return(10)
}

j05()
## Error in j05(): I'm an error
```

에러는 뭔가가 잘못되었다는 걸 알려주며, 유저가 문제를 해결하게끔 강제한다. <br /> C, Go, Rust 같은 몇몇 언어들은 문제들을 알려주는 특별한 return 값들이 있다. <br /> 그러나 R에서는 항상 에러가 나와야 한다. <br /> 8장에서 에러들과 이것들을 어떻게 다루어야 하는지 배울 것이다.

### 6.7.4 Exit handlers

가끔 함수는 global state에 임시적인 변화들changes을 필요로 할 수 있다. <br /> 하지만 이러한 변화들changes을 나중에 치우는게clean-up 힘들 수 있다.(만약 에러가 생긴다면? 어떻게 복구할건지?) <br /> 어떻게 함수가 exit되던간에, <br />   이러한 변화들이 취소undone되고 global state가 복구되었다는걸 보장하기 위해서는, <br />   exit handler를 셋업하기 위해 `on.exit()`을 사용하자.

다음의 간단한 예는, 함수가 정상적으로 exit되든 에러로 exit되든, <br />   exit handler가 실행된다는 것을 보여준다.

``` r
j06 <- function(x) {
    cat("Hello\n")
    on.exit(cat("Goodbye!\n"), add = TRUE)

    if(x) {
        return(10)
    } else {
        stop("Error")
    }
}

j06(TRUE)
## Hello
## Goodbye!
## [1] 10

j06(FALSE)
## Hello
## Error in j06(FALSE): Error
## Goodbye!
```

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
<code>on.exit()</code>을 사용할 때는 <code>add = TRUE</code>로 설정해놓자. <br /> 만약 그렇지 않으면, 각 <code>on.exit()</code>가 이전의 exit handler를 덮어쓸 것이다. <br /> 디폴트가 <code>add = FALSE</code>라서 그럼. <br /> 그래서 한 개의 handler만 쓰더라도, <code>add = TRUE</code>로 설정하는 것은 좋은 습관이다. <br /> 그래야 나중에 exit handlers를 추가하더라도 문제가 생기지 않을 것이다.
</p>
`on.exit()`는 유용하다. <br /> 왜냐하면 다음과 같이, 클린업clean-up을 필요로 하는 코드 바로 다음에, 클린업 코드를 놓으면 되기 때문.

``` r
cleanup <- function(dir, code) {
  old_dir <- setwd(dir)
  on.exit(setwd(old_dir), add = TRUE)
  
  old_opt <- options(stringsAsFactors = FALSE)
  on.exit(options(old_opt), add = TRUE)
}
```

위 코드가 의미하는 것은, 함수 안에서는 새롭게 경로를 설정했다가(`old_dir <- setwd(dir)`), <br /> `on.exit(setwd(old_dir), add = TRUE)`로 함수가 끝나고 난 뒤에는 원래의 경로로 복구undone하고(clean-up),

함수 안에서는 데이터 프레임이나 행렬을 읽는 기본 옵션을 바꿨다가(`old_opt <- options(stringsAsFactors = FALSE)`), <br /> `on.exit(options(old_opt), add = TRUE)`로 함수가 끝나고 난 뒤에는 원래의 옵션으로 복구undone하는 것(clean-up).

함수 안에서 `setwd()` 같은 것으로 새롭게 경로를 설정하면, global state에서도 경로가 바뀐다. <br /> 직접해서 확인해봐도 됨. ㅇㅇ

lazy evaluation과 함께 써서, 변경된 env에서 코드 블락을 실행시키는, 매우 유용한 패턴을 만들 수 있다.

``` r
with_dir <- function(dir, code) {
  old <- setwd(dir)
  on.exit(setwd(old), add = TRUE)
  
  force(code)
}

getwd()
## [1] "C:/Users/Phil2/Documents/portfolio/docs/Advanced R"
with_dir("~", getwd())
## [1] "C:/Users/Phil2/Documents"
```

`code`라고만 써도 evluation이 강제 되기 때문에 `force()`를 쓰는 것이 꼭 필요하지는 않다. <br /> 하지만, `force()`를 사용하면, 우리가 의도적으로 실행을 강제한다는게 매우 명백해진다. <br /> `force()`의 다른 사용법들에 대해서는 10장에서 배울 것이다.

withr 패키지는([Hester et al. 2018](http://withr.r-lib.org/)), 임시적인 state를 셋업해주는 여러가지 다른 함수들을 제공한다.

3.4나 그 이전의 버전 R은, `on.exit()` expression들은 항상 만들어진 순서대로 실행된다.

``` r
j08 <- function() {
  on.exit(message("a"), add = TRUE)
  on.exit(message("b"), add = TRUE)
}

j08()
## a
## b
```

위처럼 actions가 특정한 순서대로 일어나야 한다면, 클린업을 하는 것이 까다로워진다. <br /> 전형적으로 최근에 추가된 expression이 먼저 실행되기를 원하는 경우엔. <br /> This can make cleanup a little tricky if some actions need to happen in a specific order; <br /> typically you want the most recent added expression to be first. <br /> 이걸 3.5 혹은 이후 버전의 R에서는, `after = FALSE` 옵션을 이용해서 컨트롤할 수 있다.

``` r
j09 <- function() {
  on.exit(message("a"), add = TRUE, after = FALSE)
  on.exit(message("b"), add = TRUE, after = FALSE)
}

j09()
## b
## a
```

### 6.7.5 Exercises

------------------------------------------------------------------------

6.8 Function forms
------------------
