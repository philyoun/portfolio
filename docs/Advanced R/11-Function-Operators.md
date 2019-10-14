11 Function Operators
=====================

11.1 Introduction
-----------------

이번 Chatper에서는 function operators에 대해서 배울 것이다. <br /> **function operator**는, **하나 이상의 functions를 input**으로 받아서, **function을 output**으로 return하는 function.

(기억을 복기해보면, <br /> functionals는 function을 input으로, vector를 output으로. <br /> function factories는 vector를 input으로, function을 output으로.)

다음의 예는, `chatty()`라는 간단한 function operator를 보여준다. <br /> function을 wrap해서, 첫 번째 argument를 print하는 새로운 function을 만든다. <br /> 이러면 `map_int()`와 같은 functionals가 어떻게 작동하는지 볼 수 있는 창window를 만들어준다.

``` r
chatty <- function(f) {
  force(f)
  
  function(x, ...) {
    res <- f(x, ...)
    cat("Processing ", x, "\n", sep = "")
    res
  }
}

f <- function(x) x ^ 2
s <- c(3, 2, 1)

purrr::map_dbl(s, chatty(f))
## Processing 3
## Processing 2
## Processing 1
## [1] 9 4 1
```

function operators는 function factory와 밀접한 관련이 있음 사실 그냥 function을 input으로 받는 function factory임.

factories와 마찬가지로, 이게 없다고 못할 건 하나도 없음. Like factories, there's nothing you can't do without them. 하지만 복잡함을 factor out할 수 있도록 해줌. 그래서 너의 코드를 좀 더 readable, reusable하게끔.

function operators는 보통 functionals와 pair된다. for 루프를 사용한다면, function operator는 조금의 이득을 위해 복잡함만 늘리기 때문에 쓸 이유가 없음. 하지만 functional과 같이 쓰면, 꽤 이득을 볼 수 있다. 11.3에서 그 예를 보게 된다.

Python에 친숙하다면, decorators는 function operators의 다른 이름일 뿐이다.

### Outline

Section 11.2에서는 두 개의 매우 유용한, 존재하는 function operator를 소개하고, real problem을 solve하는데 사용하는 법을 보여줌. `safely()`와 `memoise()`.

Section 11.3 function operators로 해결할 수 있는 문제를 보여준다: 많은 웹 페이지 다운.

### Prerequisites

function operator는 일종의 function factory다. 그래서 최소한 Section 6.2에 익숙해 있어야한다

Chapter 9에서 배운 purrr의 몇몇 functionals을 쓸 것이고, purrr의 몇몇 function operators에 대해서도 배울 것이다. 그리고 memoise 패키지에 있는 `memoise()` 연산자operator도 쓸 것이다.

``` r
library(purrr)
library(memoise)
```

11.2 Existing function operators
--------------------------------

흔하게 발생하는 문제를 해결해줄 2개의 매우 유용한 function operators가 있다. 그리고 이것들은 function operators가 뭘하는지에 대해 감을 잡게 해줄 것이다. `purrr::safely()`, `memoise::memoise()`

### 11.2.1 Capturing errors with `purrr::safely()`

for 루프의 한 가지 장점은 iteration이 실패하더라도,     그 실패전까지의 모든 결과물들에 access할 수 있다는 것이다. 다음의 예를 보자.

``` r
x <- list(
    c(0.512, 0.165, 0.717),
    c(0.064, 0.781, 0.427),
    c(0.890, 0.785, 0.495),
    "oops"
)

out <- rep(NA_real_, length(x))
for(i in seq_along(x)) {
  out[[i]] <- sum(x[[i]])
}
## Error in sum(x[[i]]): 인자의 'type' (character)이 올바르지 않습니다

out
## [1] 1.394 1.272 2.170    NA
```

같은 것을 functional가지고 한다면, 아무런 output을 얻을 수 없다. 그래서 어디가 문제인지 알아내기가 힘들다.

``` r
map_dbl(x, sum)
## Error in .Primitive("sum")(..., na.rm = na.rm): 인자의 'type' (character)이 올바르지 않습니다
```

`purrr::safely()`는 이 문제를 해결하는데 도움을 준다. `safely()`는 function operator다. 에러를 데이터로 변환하도록, 함수를 transform하는. `safely()` is a function operator that transforms a function / to turn errors into data. 그러니깐, 출력되는 에러를 데이터로 변환해준다. 이게 무슨 소리인지는 밑의 예를 보면 금방 이해가 된다. 어려운 얘기 아니다. (이게 가능하도록 해주는 기본 아이디어를 Section 8.6.2에서 배울 수 있다.)

``` r
safe_sum <- safely(sum)
safe_sum
## function (...) 
## capture_error(.f(...), otherwise, quiet)
## <bytecode: 0x00000000180d2ac0>
## <environment: 0x00000000180d2628>
```

다른 function operators와 마찬가지로, safely()는 function을 받고, wrapped function을 return한다. 이렇게 얻은 wrapped function도, 평소와 같이 호출하면 된다.

``` r
safe_sum(x[[1]])
## $result
## [1] 1.394
## 
## $error
## NULL
str(safe_sum(x[[1]]))
## List of 2
##  $ result: num 1.39
##  $ error : NULL
safe_sum(x[[4]])
## $result
## NULL
## 
## $error
## <simpleError in .Primitive("sum")(..., na.rm = na.rm): 인자의 'type' (character)이 올바르지 않습니다>
str(safe_sum(x[[4]]))
## List of 2
##  $ result: NULL
##  $ error :List of 2
##   ..$ message: chr "인자의 'type' (character)이 올바르지 않습니다"
##   ..$ call   : language .Primitive("sum")(..., na.rm = na.rm)
##   ..- attr(*, "class")= chr [1:3] "simpleError" "error" "condition"
```

`safely()`를 이용해서 transform된 함수가, 항상 `results`와 `error`라는 2개의 elements를 갖고 있는 리스트를, 돌려주는걸 볼 수 있다. 만약 function이 성공적으로 실행되면, `error`는 `NULL`이 되고, `result`는 result를 contain하고 있다. 만약 function이 실패하면, `result`는 `NULL`이 되고, `error`는 error를 contain하고 있다.

이제, safely()를 functional과 함께 사용해보자.

``` r
out <- map(x, safely(sum))
str(out)
## List of 4
##  $ :List of 2
##   ..$ result: num 1.39
##   ..$ error : NULL
##  $ :List of 2
##   ..$ result: num 1.27
##   ..$ error : NULL
##  $ :List of 2
##   ..$ result: num 2.17
##   ..$ error : NULL
##  $ :List of 2
##   ..$ result: NULL
##   ..$ error :List of 2
##   .. ..$ message: chr "인자의 'type' (character)이 올바르지 않습니다"
##   .. ..$ call   : language .Primitive("sum")(..., na.rm = na.rm)
##   .. ..- attr(*, "class")= chr [1:3] "simpleError" "error" "condition"
```

여기서 output은, 좀 불편한 form으로 되어 있다. 4개의 리스트들이 있고, 각각은 result와 error를 가지고 있는 리스트다. 이 output을 좀 더 쉽게 만들 수 있음. purrr::transpose()를 사용해서 뒤집어놓음으로써. We can make the output easier to use / by turning it "inside-out" with purrr::transpose() 그러면 result들의 리스트와, error들의 리스트를 얻게 된다.

``` r
out <- transpose(map(x, safely(sum)))
str(out)
## List of 2
##  $ result:List of 4
##   ..$ : num 1.39
##   ..$ : num 1.27
##   ..$ : num 2.17
##   ..$ : NULL
##  $ error :List of 4
##   ..$ : NULL
##   ..$ : NULL
##   ..$ : NULL
##   ..$ :List of 2
##   .. ..$ message: chr "인자의 'type' (character)이 올바르지 않습니다"
##   .. ..$ call   : language .Primitive("sum")(..., na.rm = na.rm)
##   .. ..- attr(*, "class")= chr [1:3] "simpleError" "error" "condition"
```

위에서는 List of 4였는데, List of 2가 된 것을 볼 수 있다.

이제 잘 작업이 된 결과물들을 쉽게 찾을 수 있다. 혹은, fail한 경우에는 그 input도 찾을 수 있고.

``` r
# 에러가 있는지 없는지를 index로 만들기
ok <- map_lgl(out$error, is.null)
ok 
## [1]  TRUE  TRUE  TRUE FALSE

x[!ok] # fail한 경우에 input 찾기
## [[1]]
## [1] "oops"

out$result[ok] # 작동한 결과물들을 찾기
## [[1]]
## [1] 1.394
## 
## [[2]]
## [1] 1.272
## 
## [[3]]
## [1] 2.17
```

예를 들어, 데이터 프레임들을 모아놓은 리스트가 있고, 각각의 데이터 프레임에 GLM(Generalised Linear Model)을 fitting하는 상황을 상상해보자. GLM은 가끔 optimisation 문제 때문에 fail할 수 있는데, 여전히 일단 다 시도해본다음에, 나중에 fail한 것만 따로 봐보고 싶다고치자.

``` r
fit_model <- function(df) {
    glm(y ~ x1 + x2 * x3, data = df)
}

models <- transpose(map(datasets, safely(fit_model)))

# 위에서 했던 것과 같이, 에러가 있는지 없는지를 index로 만들기
ok <- map_lgl(models$error, is.null)

# 어떤 데이터셋이 converge하는데 실패했는지?
datasets[!ok]

# converge에 성공한 모델들은 어떻게 생겼는지?
models[ok]
```

저자 생각엔, 이게 functionals와 function operators를 결합한 훌륭한 예다. safely()는 간결하게 표현하도록 해준다. 일반적인 데이터 분석 문제를 해결하는데 필요한 것을. safely() lets you succinctly express / what you need / to solve a common data analysis problem.

purrr에는 비슷한 맥락의 3개 다른 function operators가 있다. - `possibly()`: error가 있을 때 디폴트 값을 return해준다. 얘는 에러 발생 여부를 알 수 있는 방법이 없기 때문에, 확실한 sentinel 값(NA와 같은)이 있을 때만 쓰자

-   `quietly()`: output, 메세지, warning side-effects를 output의 각 component로 만들어준다. 그러니깐, `output`, `message`, `warning`으로 넣어준다.

-   `auto_browse()`: 에러가 있을 때, 함수 안에서 `browser()`을 자동으로 실행해준다.

더 자세하게 알고 싶다면 documentation을 보자.

### 11.2.2 Caching computations with `memoise::memoise()`

계산을 caching해놓을 수 있다. 이러면 복잡한 계산을 두 번 할 필요가 없다.

또 다른 편리한 function operator는 memoise::memoise()이다. 이건 함수를 **memoise**한다. function은 이전의 inputs를 기억하고, cached results를 return하는 것. memoisation은 classic computer science에서, memory와 speed의 tradeoff하는 것의 한 가지 예다. memoise된 함수는 더 빨리 실행된다. memory를 사용해서. speed를 얻은 것.

expensive operation을 simulate하는 toy function과 함께, 이 아이디어에 대해 알아보자.

``` r
slow_function <- function(x) {
    Sys.sleep(1)
    x * 10 * runif(1)
}

system.time(print(slow_function(1)))
## [1] 7.387341
##    user  system elapsed 
##       0       0       1
system.time(print(slow_function(1)))
## [1] 4.83196
##    user  system elapsed 
##       0       0       1
```

이 함수를 memoise하고 나면, 우리가 새로운 argument와 함께 호출할 때는 느리다. 하지만 이전에 해봤던 arguments와 함께 호출할 때면, 즉시 나온다. 이전에 계산된 값을 바로 되찾아온다retrieve.

``` r
fast_function <- memoise::memoise(slow_function)
system.time(print(fast_function(1)))
## [1] 2.900197
##    user  system elapsed 
##       0       0       1

system.time(print(fast_function(1))) # 해봤던 arguments니깐 즉시 나옴.
## [1] 2.900197
##    user  system elapsed 
##    0.03    0.00    0.03

system.time(print(fast_function(2))) # 이러면 새로운 argument니깐 다시 느려지고.
## [1] 5.514712
##    user  system elapsed 
##       0       0       1
```

이런 memoisation의 그나마 현실적인 이용으로는 피보나치 수열 계산이 있겠다. 피보나치 수열은 recursive하게 정의되는데, f(0) = 0, f(1) = 1로 정의되어 있고, f(n) = f(n-1) + f(n-2)이다.
