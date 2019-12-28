6.1 Introduction
----------------

------------------------------------------------------------------------

6.2 Function fundamentals
-------------------------

R에서 함수를 이해하기 위해서는, 2개의 아이디어를 이해해야 한다. 1. 함수는 3개의 요소components로 쪼갤 수 있다. 요소arguments, 본체body 그리고 environment 얘는 그냥 env라고 해야지.

모든 규칙들rules은 예외가 있다. 그리고 이 1번의 경우엔, "원시primitive" base 함수들이 있다. 이것들은 온전히 C 언어로만 고안implement되었다.

1.  함수Function는 오브젝트다. 벡터vector도 오브젝트이듯이.

### 6.2.1 Function components

함수는 3가지 파트가 있다. 1. `formals()`: 어떻게 함수를 호출할건지 컨트롤할 수 있는 요소arguments들의 리스트

1.  `body()`: 함수 안의 코드. the code inside the function

2.  `environment()`: 어떻게 함수가, 이름들과 연관되어 있는 값을 찾는지, 이걸 결정하는 데이터 구조

formals와 body는, 함수를 만들 때 명백하게 정해줘야explicitly specify하는데, environment는, 어디에 만드냐에 따라 implicit하게 specify 된다.

function env는 항상 존재한다. 하지만, 함수가 global env에 정의되어 있지 않을때에만 프린트된다. it is only printed when the function isn't defined in the global env.

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

함수를, 아래의 다이어그램처럼 그려볼 것이다. 왼쪽의 검은 점은 env다. 오른쪽 2개의 블락들blocks은 함수의 인자들arguments이다. body는 안 그릴 것이다. 왜냐하면 보통 양이 많고, 함수의 shape을 이해하는데 도움이 안 되기 때문. ![Figure 6.1](https://d33wubrfki0l68.cloudfront.net/de34ef3939642ec68b2b78dc310f3baa22d12106/ac3f3/diagrams/functions/components.png){ width = 50% }

R의 모든 오브젝트들과 마찬가지로, 함수들도 몇 개든 상관없이 추가적인 특성들`attributes()`을 가질 수 있다. base R에 의해 사용된 하나의 특성은 `srcref`인데, source reference의 줄임말이다. 함수를 만드는데 사용된 소스코드를 알려준다.

`srcref`는 프린팅하는데 사용된다. 왜냐하면, `body()`와는 다르게, 코드 코멘트들이랑 다른 포매팅들도 갖고 있기 때문.

``` r
attr(f02, "srcref")
## function(x, y) {
##  # A comment
##  x + y
## }
```

### 6.2.2 Primitive functions

함수는 3개의 파트를 갖는다는 이 규칙에는, 하나의 예외가 있다. 원시primitive 함수들, `sum()`이나 `[`와 같은 것들은, C 코드를 직접적으로 호출한다.

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

이 함수들은 애초에 R이 아닌, C로 존재하기 때문에,   이것들의 `formals()`, `body()`, `environment()`는 전부 `NULL`이다.

``` r
formals(sum)
## NULL
body(sum)
## NULL
environment(sum)
## NULL
```

원시 함수들은 base 패키지에서만 발견된다. 분명한 퍼포먼스적인 이점이 있지만, 대가가 있다. 더 작성하기가 힘들다. harder to write. 이러한 이유에서, R-코어팀은 다른 옵션이 없는게 아니라면, 이 방법을 쓰지 않는다.

### 6.2.3 First-class functions

R 함수가, 그 자체로 오브젝트라는걸 이해하는건 매우 중요하다. 이건 보통 "first-class functions"라고 부르는 언어 속성language property다.

많은 다른 언어들과는 다르게, 함수를 정의하고 이름 붙이는데 다른 특별한 문법syntax이 없다. 그냥 `function()`을 이용해서 함수 오브젝트function object를 만들고, `<-`를 이용해서 이름 붙이면 된다.

<details> <summary>느낌</summary> 프로그래밍 언어가 퍼스트클래스 함수를 지원하면, 변수에 함수를 할당도 할 수 있고, 인자로써 다른 함수에 전달할 수도 있고, 함수의 리턴값으로도 쓸 수 있고. </details> <br /> <br />

``` r
f01 <- function(x) {
  sin(1/ x ^ 2)
}
```

<img src="https://d33wubrfki0l68.cloudfront.net/5db72a270ade61a321dfc2519e6fb0f56370609e/807cb/diagrams/functions/first-class.png" alt="Figure 6.2" style="width:50.0%" />

거의 항상, 함수를 만들고 나면 이름을 붙이겠지만, 이 이름을 붙이는 binding step이 꼭 요구되는 건 아니다. 이름을 안 붙이기로 결정했다면, **익명 함수anonymous function**을 만든 것이다. 이름을 꼭 붙여야 할 필요가 없는 경우라면, 상당히 유용하다.

``` r
lapply(mtcars, function(x) length(unique(x)))
Filter(function(x) !is.numeric(x), mtcars)
integrate(function(x) sin(x) ^ 2, 0, pi)
```

마지막 옵션은, 리스트에다가 함수들을 넣는 것이다. (아니 리스트에다 함수 넣는 것도 되는건 처음 알았네)

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

R에서, 종종 **closures**라는 함수를 볼 것이다. 이건, R 함수들이 자기 자신의 env를 캡쳐한다는 사실을 반영한 것이다. [Section 7.4.2](https://blog-for-phil.readthedocs.io/en/latest/Advanced%20R/07-Environments/#742-the-function-environment)에서 더 배우게 될 것이다.

### 6.2.4 Invoking a function

보통 함수를, 함수 이름에다 괄호를 열고, 인자들arguments을 넣고, 괄호를 닫는 식으로 호출한다. 예를 들어서, `mean(1:10, na.rm = TRUE)` 이렇게. 그런데 만약에 데이터 구조에 인자들을 이미 갖고 있는 경우에는 어떻게 할 수 있을까? 예를 들어서,

``` r
args <- list(1:10, na.rm = TRUE)
```

이렇게 갖고 인자들을 갖고 있는 것임.

`do.call()`을 쓰면 된다. 이 함수는 2개의 인자들arguments을 받는다. 하나는 호출할 함수 이름, 다른 하나는 함수 인자들을 가지고 있는 리스트.

``` r
do.call(mean, args)
## [1] 5.5
```

이 아이디어를 Section 19.6에서 다시 볼 것이다.

### 6.2.5 Exercises

------------------------------------------------------------------------

6.3 Function composition
------------------------
