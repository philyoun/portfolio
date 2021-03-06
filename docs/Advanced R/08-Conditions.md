8.1 Introduction
----------------

**condition** 시스템은, <br /> 함수 작성자function author가 뭔가 잘못되고 있다는 걸 알려주고, 함수 사용자가 해결할게 있다는 걸 알려주는, <br /> the author of a function to indicate that something unusual is happening, and the user of that function to deal with it <br /> 한 쌍의 도구들tools을 제공한다.

여기서 conditions는 조건문의 컨디션이 아니라, 상태라는 뜻의 컨디션으로 쓰였다. <br />

함수 작성자는 컨디션들conditions을, <br />   `stop()`(에러에 대해), `warning()`(경고에 대해), `message()`(메세지에 대해)와 같은 함수들을 이용해 <br /> 시그널signal해주고,

함수 사용자는 `tryCatch()`나 `withCallingHandlers()`와 같은 함수들을 이용해 컨디션들을 다룬다handle. <br />

컨디션 시스템을 이해하는 것은 중요하다. <br /> 왜냐하면 작성자나 사용자 둘 다의 입장에서 종종 다룰 것이기 때문. <br /> 너가 함수를 만들 때는 컨디션 시그널링을, 함수를 호출할 때는 함수가 보내는 시그널된 컨디션을 다뤄야함.

R은 Common Lisp의 아이디어를 기반으로 한 매우 강력한pwerful 컨디션 시스템을 제공한다. <br /> R의 Object-oriented 프로그래밍과 마찬가지로, 인기있는 프로그래밍 언어들과는 조금 달라서, 잘못 이해하기가 쉽다. <br /> 그리고 효율적으로 사용할 수 있게끔 설명해놓은게 별로 없기도 하다. <br /> 역사적으로, 이건 몇 안 되는 사람들만 이것의 power을 완전히 사용했다는 뜻이다. <br /> 이 챕터의 목표는 이 상황을 개선하는 것. <br /> R의 컨디션 시스템에 대한 핵심 아이디어를 이해하고, <br />   너의 코드를 더 강하게 만드는 여러가지 실용적인 도구들tools에 대해 배울 것.

이 챕터를 작성하는데 있어, 2가지 자료가 매우 유용했다. <br /> 컨디션 시스템에 대한 영감과 동기를 더 배우고 싶어, 직접 읽어보고 싶을 수도 있겠다.

-   [A propotype of a conditions system for R](http://homepage.stat.uiowa.edu/~luke/R/exceptions/simpcond.html) <br />
-   [Beyond exception handling: conditions and restarts](http://www.gigamonkeys.com/book/beyond-exception-handling-conditions-and-restarts.html)

이 아이디어들을 고안implement해 놓은 C 코드에 대해서 알아보는 것도 도움이 된다. <br /> 어떻게 작동하는지를 이해해보는데 관심이 있다면, [my notes](https://gist.github.com/hadley/4278d0a6d3a10e42533d59905fbed0ac)를 읽어보는 것도 유용할거다.

### Quiz

이 챕터를 스킵하고 싶은가? 아래의 문제들을 대답할 수 있다면, 그래도 된다. <br /> [Section 8.7]()에서 답을 찾을 수 있음.

1.  컨디션condition의 3가지 가장 중요한 타입들은?

2.  코드 블락에서 에러들을 무시하고 싶다면 어떤 함수를 이용해야할까? <br /> What function do you use to ignore errors in block of code?

3.  `tryCatch()`와 `withCallingHandlers()`간의 가장 중요한 차이는 무엇인가?

4.  커스텀custom 에러 오브젝트를 만들어하고 싶을 이유는 무엇일까?

### Outline

-   Section 8.2에서는 컨디션들을 시그널링하는데 쓸 기본적인 툴들을 소개해줌 <br /> 그리고 각 타입을 언제 사용하는 것이 적절한지에 대해 다룸.

-   Section 8.3에서는 컨디션들을 다루는데 있어 가장 간단한 툴들에 대해 가르쳐줌. <br /> `try()`나 `supressMessages()`와 같은 함수들은 컨디션들을 삼키고swallow, 컨디션들이 탑 레벨에 뜨는 것을 방지?

그러니깐 8.2는 함수 작성자의 입장, 8.3은 함수 사용자의 입장

-   Section 8.4에서는 컨디션 오브젝트에 대해 소개하고, 컨디션 핸들링의 2가지 기본적인 툴들. <br /> 에러 컨디션들에 대해선 `tryCatch()`, 나머지에 대해선 `withCallingHandlers()`

-   Section 8.5에서는 이미 만들어져있는 컨디션 오브젝트를 유용한 데이터 저장을 위해 어떻게 확장시킬건지. <br /> 더 좋은 결정을 내리기 위해 컨디션 핸들러가 이용할 수 있는.

-   Section 8.6에서는 이전 sections에서 배웠던, low-level 툴들을 사용한 실용적인 응용으로 챕터를 마무리.

### 8.1.1 Prerequisites

컨디션 시그널링과 핸들링을 하는데 있어, base R 함수들뿐 아니라 rlang의 함수들을 좀 이용해야 한다.

``` r
library(rlang)
```

------------------------------------------------------------------------

8.2 Signalling conditions
-------------------------

코드에서 시그널할 수 있는 3개의 컨디션들이 있다. : 에러, 경고, 메세지

-   에러errors가 가장 심각한거다. <br /> 함수가 계속될 방법이 없다는 걸 알려주고, 실행이 멈춘다.

-   경고warnings는 에러와 메세지 사이에 있다. <br /> 보통 뭔가가 잘못되었지만, 함수가 적어도 부분적으로 복구 가능했다는 걸 알려준다. <br /> indicate that something has gone wrong but the function has been able to at least partially recover.

-   메세지messages가 가장 순한 것이다. <br /> 이건, 유저들 대신에, 어떠한 행동action이 실행되었다는 걸 알려주는 방법이다.

interactive하게만 생성될 수 있는 마지막 컨디션이 있다. interrupt <br /> interrupt는 Esc 키, Ctrl+Break 혹은 Ctrl+C(윈도우냐 맥이냐에 따라)를 눌러서 실행을 방해interrupt했다는 것을 알려준다.

컨디션들을 보통 크게 표시된다. <br /> R의 인터페이스에 따라, 진한 글씨체로 혹은 빨간 색으로. <br /> 에러들은 항상, <br />   ①에러는 "Error"로, ②경고는 "Warning" 혹은 "Warning message"로, ③메세지는 아무것도 없이 시작하기 때문에, <br /> 구별할 수 있다.

``` r
stop("This is what an error looks like")
## Error in eval(expr, envir, enclos): This is what an error looks like
warning("This is what warning looks like")
## Warning: This is what warning looks like
message("This is what a message looks like")
## This is what a message looks like
```

다음의 3개 sections에서 에러, 경고, 메세지를 자세하게 설명한다.

### 8.2.1 Errors

base R에서, 에러들은 `stop()`을 통해 시그널되거나 던져thrown진다.

``` r
f <- function() g()
g <- function() h()
h <- function() stop("This is an error!")

f()
## Error in h(): This is an error!
```

디폴트로, 에러 메세지는 call을 포함한다. 하지만 보통 별로 쓸모가 없다. <br /> 그리고 `traceback()`을 통해 쉽게 얻을 수 있는 정보를 요약한다. <br /> 그래서, `call. = FALSE`를 사용하는 것이 좋은 습관이라고 생각한다.

``` r
h <- function() stop("This is an error!", call. = FALSE)
f()
## Error: This is an error!
```

rlang 패키지에서, `stop()`과 같은 역할을 하는 것은 `rlang::abort()`다. <br /> 얘는 이걸 자동으로 한다. `call. = FALSE`를 자동으로 한다고. <br /> `abort()`를 챕터 내내 쓸 것인데, 가장 강력한 기능인, <br />   추가적인 메타데이터를 추가add additional metadata하는 기능은 마지막에 배울 것이다.

``` r
h <- function() abort("This is an error!")
f()
## Error: This is an error!
```

주의사항: `stop()`은 여러 개의 인풋들을 붙일 수 있다. `abort()`는 못 함. <br /> abort를 사용해 복잡한 에러 메세지를 만들고 싶다면, `glue::glue()` 사용할 것을 추천한다. <br /> 이럼, [Section 8.5]()에서 배울, `abort()`의 유용한 기능들에, 다른 인자들arguments 사용하는걸 가능케 해준다.

가장 좋은 에러 메세지는, 무엇이 잘못되었는지를 말해주고, <br />   해결하기 위해서는 어떻게 해야할지 방향을 알려주는 것. <br /> 좋은 에러 메세지를 작성하는 것은 힘들다. <br /> 왜냐하면 보통 에러는, 사용자가 흠이 있는flawed 모델을 넣었을 때 발생하는데, <br /> 개발자가, 사용자가 함수에 대해 어떻게 잘못 생각했을지를 상상하는 것이 어렵기에, <br />   알맞은 방향으로 조종하기 힘들기 때문. <br /> 말나온김에, tidyverse 스타일 가이드는 유용한 몇 개의 일반적인 원리들을 설명해준다. <br /> <http://style.tidyverse.org/error-messages.html>

### 8.2.2 Warnings

`warning()`으로 시그널되는 경고들은, 에러들보단 약한 것이다. <br /> 얘들은, 뭔가 잘못되었지만, 코드는 복구된 다음 계속 되었다는 것을 시그널해준다. <br /> 에러들과는 다르게, 하나의 함수 호출에 여러 개의 경고들을 가질 수 있다.

``` r
fw <- function() {
    cat("1\n")
    warning("W1")
    cat("2\n")
    warning("W2")
    cat("3\n")
    warning("W3")
}
```

디폴트로, 컨트롤control이 최상위 레벨로 돌아올 때만 경고들은 캐시되고 프린트된다. <br />
warnings() are cached and printed only when control returns to the top level.

``` r
fw()
## 1
## Warning in fw(): W1
## 2
## Warning in fw(): W2
## 3
## Warning in fw(): W3
```

이 행동을 warn 옵션을 이용해서 컨트롤할 수 있다.

-   경고들이 즉시 나오게 하려면, `options(warn = 1)`으로 설정

-   경고들을 에러들로 바꾸려면, `options(warn = 2)`으로 설정 <br /> 이게 경고를 디버그하기 가장 쉬운 방법이다. <br /> 왜냐하면 에러라면 `traceback()`과 같은 도구를 이용해 출처source를 찾을 수 있기 때문.

-   디폴트 행동을 복원하려면, `options(warn = 0)`으로 설정

하나씩 해보면 무슨 소리인지 금방 이해할 수 있다.

`stop()`과 마찬가지로, `warning()`도 call 인자argument를 받는다. <br /> 이건 좀 더 유용하긴한데(경고들은 보통 출처source에서 좀 더 멀기 때문), <br />   그래도 난 여전히 `call. = FALSE`를 이용해 억제suppress해놓는다. <br /> `rlang::abort()`와 마찬가지로, rlang은 `warning()`의 동일한 역할로 `rlang::warn()`이 있다. <br /> 이것도 `call.`을 디폴트로 억제해놓는다.

경고는 메세지(이거에 대해 알아야한다)와 에러(이걸 고쳐야한다) 사이에 위치하기에, <br />   언제 경고를 사용해야하는지 정확한 충고를 주는게 힘들다. <br /> 일반적으로, 절제해서 사용해라. <br /> 왜냐하면 경고는, output이 많다면 놓치기가 쉽고, <br />   명백하게 잘못된 input이 있는데 함수가 너무 쉽게 복구되지 않도록 원하기 때문. <br /> 개인적인 의견으로, base R은 경고를 너무 많이 쓰는 것 같다. <br /> 그리고 이 중 많은 것들이 에러인게 더 나을 것 같다. <br /> 예를 들어, 다음의 경고들은 에러인게 더 낫다.

``` r
formals(1)
## Warning in formals(fun): argument is not a function
## NULL
file.remove("this-file-doesn't exist")
## Warning in file.remove("this-file-doesn't exist"): 파일 'this-file-doesn't
## exist'을 지울 수 없습니다, 그 이유는 'No such file or directory'입니다
## [1] FALSE
lag(1:3, k = 1.5)
## Warning in lag.default(1:3, k = 1.5): 'k' is not an integer
## [1] 1 2 3
## attr(,"tsp")
## [1] -1  1  1
```

경고가 확실하게 더 적절한 경우는 2개의 경우밖에 없다.

1.  오래된 코드가 작업을 계속 할 수는 있는 함수를 **비난deprecate**하고 싶을 때.(경고를 무시하는 것은 괜찮) <br /> 하지만 새로운 함수를 사용하도록 권장하고 싶다.할 때. <br /> When you deprecate a function you want to allow older to continue to work(so ignoring the warning is OK) <br /> but you want to encourage the user to switch to a new function.

2.  문제를 복구recover할 수 있다고 상당히 확신reasonably certain할 때. <br /> 만약에 문제 해결이 가능하다고 100% 확신한다면, 아무런 메세지가 필요없을 것이다. <br /> 만약 문제 해결할 수 있을지 확신할 수 없다면, 에러를 내도록 하는게 낫다.

다른 경우에는 경고를 절제해서 쓰고, 에러가 적절하지 않을지 신중하게 고려해봐라.

### 8.2.3 Messages

`message()`로 시그널되는 메세지는, 정보를 준다informational. <br /> 메세지는, 사용자 대신에 R이 뭔가를 했다는걸 알려준다. <br /> 좋은 메세지들은 중도를 지킨다.: <br />   사용자에게 어떻게 되어가고 있는지 딱 적당한 정보를 주지만, <br />   압도되지는 않을 정도만 준다.

`message()`는 즉시 디스플레이되며, `call.` 인자argument는 받지 않는다.

``` r
fm <- function() {
    cat("1\n")
    message("M1")
    cat("2\n")
    message("M2")
    cat("3\n")
    message("M3")
}

fm()
## 1
## M1
## 2
## M2
## 3
## M3
```

메세지를 사용하기 좋은 때는, <br /> - 디폴트 인자argument가 사소하지 않은 양의 계산을 필요로 하고, <br />   사용자에게 어떤 값이 이용되었는지 알려주고 싶을 때. <br /> 예를 들어, ggplot2는 `bindwidth`를 공급해주지 않았을 때, 몇 개의 bins를 사용했는지를 보고한다.

-   일차적으로 다른 작업side-effects을 위해 호출되는 함수들. <br /> 예를 들어, 디스크에 파일을 저장write하거나, 웹 API를 호출하거나, 데이터베이스에 저장할 때, <br /> 일정한 상태 메세지를 통해 무엇이 일어나고 있는지 사용자에게 말해주는 것은 유용하다.

-   중간 결과물intermediate output 없이 긴 running 프로세스를 시작하려 할 때. <br /> progress bar([progress](https://github.com/r-lib/progress))가 더 낫긴하지만, 메세지로 시작해보기 좋다.

-   패키지를 작성할 때, 너의 패키지가 로드되었다는걸 표시할 메세지를 원할 수 있다.(예를 들어, `.onAttach()` 안에서처럼) <br /> 여기서는 `packageStartupMessage()`를 사용해야만 한다.

일반적으로, 메세지를 만드는 모든 함수들은, 어떻게든 억제suppress할 방법이 필요하다. <br /> 예를 들어 `quiet = TRUE`와도 같은 인자argument. <br /> `suppressMessages()`를 이용해 모든 메세지들을 억제하는 것이 가능하다. <br /> 하지만 곧 배우게 될 텐데, 좀 더 정제된 컨트롤을 주는 것이 낫다.

`message()`와 긴밀하게 연결되어 있는 `cat()`을 비교하는 것은 중요하다. <br /> 사용법과 결과물을 보면, 상당히 비슷해보인다.

``` r
cat("Hi!\n")
## Hi!
message("Hi!")
## Hi!
```

하지만, `cat()`과 `message()`의 목적이 다르다. <br /> 함수의 첫 번째 목적이, 콘솔에 프린트할 때는 `cat()`을 사용한다. `print()`나 `str()` 메소드와 같이. <br /> 주요 목적은 다른 것인데, 콘솔에 프린트하게끔 사이드 채널side-channel을 원할 때는 `message()`를 사용. <br /> 즉, `cat()`은 사용자가 뭔가 프린트되도록 요청ask할 때. <br /> `message()`는 개발자가 뭔가 프린트되도록 선택elect할 때. <br />

### 8.2.4 Exercises

------------------------------------------------------------------------

8.3 Ignoring conditions
-----------------------

R에서 컨디션을 다루는 가장 간단한 방법은, 무시ignore하는 것이다.

-   에러는 `try()`로 무시 <br />
-   경고는 `suppressWarnings()`로 무시 <br />
-   메세지는 `suppressMessages()`로 무시

알고있는 어떤 한 타입의 컨디션은 억제하면서 다른 컨디션들은 통과되도록 할 수가 없기 때문에, <br />   이 함수들은 사용하기가 어렵다. <br /> ??? <br /> 이 문제에 대해선 나중에 다루어볼 것이다. <br /> These functions are heavy handed as you can't use them to suppress a single type of condition <br />   that you know about, while allowing everything else to pass through.

`try()`는 실행이, 에러가 일어난 뒤에도 계속될 수 있게끔 허용해준다. <br /> 보통 실행한 함수가 에러를 내면, 함수는 즉시 종료되며, 값을 return하지 않는다.

``` r
f1 <- function(x) {
    log(x)
    10
}

f1("x")
## Error in log(x): 수학함수에 숫자가 아닌 인자가 전달되었습니다
```

하지만, 에러를 만드는 statement을 `try()`로 감싸놓으면, 에러 메세지는 표시되지만 실행은 계속된다.

``` r
f2 <- function(x) {
    try(log(x))
    10
}

f2("a")
## Error in log(x) : 수학함수에 숫자가 아닌 인자가 전달되었습니다
## [1] 10
```

`try()`의 결과물을 저장하고, <br />   성공했는지 실패했는지에 따라 다른 행동action을 하도록 하는 것도 가능은 하다. <br /> 하지만 추천하지는 않는다. <br /> 대신에, `tryCatch()`를 사용하거나 높은 차원의higher-level helper를 사용하는게 낫다. <br /> 곧 배울 것이다.

간단하지만 유용한 패턴은, 호출 안에다가 할당을 하는 것이다. <br /> 이럼 코드가 성공하지 못할 경우, 사용될 디폴트값이 정의하도록 해준다. <br /> argument는 calling env에서 evaluate되지, 함수 안에서 evaluate되는게 아니기 때문에 그렇다. <br /> ([Section 6.5.1](https://blog-for-phil.readthedocs.io/en/latest/Advanced%20R/06-Functions/#651-promises)을 다시 보자.)

``` r
default <- NULL
try(default <- read.csv("possibly-bad-input.csv"), silent = TRUE)
```

`suppressWarnings()`랑 `suppressMessages()`는 모든 경고랑 메세지를 억제suppress한다. <br /> 에러와는 다르게, 메세지랑 경고는 실행을 종료하지 않으며, <br />   하나의 블락 안에 여러 개의 경고들이랑 메세지들을 가질 수 있다.

``` r
suppressWarnings({
    warning("Uhoh!")
    warning("Another warning")
    1
})
## [1] 1

suppressMessages({
    message("Hello there")
    2
})
## [1] 2

suppressWarnings({
    message("You can still see me")
    3
})
## You can still see me
## [1] 3
```

------------------------------------------------------------------------

8.4 Handling conditions
-----------------------

모든 컨디션들은 각각의 디폴트 행동behaviour이 있다: <br />   에러는 실행을 멈추고 최상위 레벨top level로 return하며, <br />   경고는 캡쳐되고 다 합쳐서 표시된다.displayed in aggregate <br />   메세지는 즉시 표시된다.

컨디션 **핸들러handler**는, 디폴트 행동을 일시적으로 덮어씌우거나 보충할 수 있도록 해준다. <br /> override or supplement

두 개의 함수들, `tryCatch()`나 `withCallingHandlers()`는 핸들러들을 등록register할 수 있도록 해준다. <br /> 핸들러라는건, 시그널된 컨디션을 단 하나의 인자argument로 받는 함수들. <br /> 등록 함수registration function들은 같은 기본 형식을 갖는다.

``` r
tryCatch(
    error = function(cnd) {
        # 에러가 발생했을 때 실행될 코드
    },

    핸들러가 active할 때 실행될 코드
)

withCallingHandlers(
    warning = function(cnd) {
        # 경고가 시그널되었을 때 실행되는 코드
    },
    message = function(cnd) {
        # 메세지가 시그널되었을 때 실행되는 코드
    },
    
    핸들러가 active할 때 실행될 코드
)
```

각각이 만드는 핸들러들의 타입은 다르다.

-   `tryCatch()`는 exiting 핸들러를 정의한다. <br /> 컨디션이 핸들되고나면, 컨트롤control은 `tryCatch()`가 호출된 context로 돌아간다. <br /> 이래서 `tryCatch()`가 에러나 방해interrupt를 다룰 때 더 적합하다. <br /> 왜냐하면, 이것들은 어쨋거나 항상 exit해야되기 때문.

-   `withCallingHandlers()`는 calling 핸들러를 정의한다. <br /> 컨디션이 캡쳐되고나면, 컨트롤control은 컨디션이 시그널 되었던 context로 돌아간다. <br /> after the condition is captured control returns to the context where the condition was signalled. <br /> 이래서 `withCallingHandlers()`는 에러가 아닌 컨디션들을 다룰 때 더 적합하다.

하지만 이 핸들러들을 배우고 사용하기 전에, 컨디션 오브젝트들objects에 대해 다뤄봐야한다. <br /> 이것들은 컨디션을 시그널할 때 암묵적implicit으로 만들어지지만, 핸들러 안에서는 명백하다.

### 8.4.1 Condition objects

이 때까지는 컨디션들을 시그널하기만 했고, 안보는데서 만들어지는 오브젝트들에 대해선 알아보지 않았다. <br /> 컨디션 오브젝트를 보는 가장 쉬운 방법은, 시그널된 컨디션에서 하나 캐치catch해보는 것. <br /> 그게 `rlang::catch_cnd()`이 하는 것이다.

``` r
cnd <- catch_cnd(stop("An error"))
str(cnd)
## List of 2
##  $ message: chr "An error"
##  $ call   : language force(expr)
##  - attr(*, "class")= chr [1:3] "simpleError" "error" "condition"
```

만들어져 있는built-in 컨디션은, 2개의 원소들elements을 갖고 있는 리스트다.

-   `message`는, 사용자에게 보여줄 텍스트를 가지고 있는, 길이 하나짜리 캐릭터 벡터. <br /> 메세지를 추출하기 위해서는, `conditionMessage(cnd)`를 사용해라.

-   `call`은, 컨디션을 불러 일으키는 호출. the call which triggers the condition. <br /> 위의 예에서처럼, 우린 call을 사용하지 않기 때문에, 이건 보통 `NULL`일 것이다. <br /> 이걸 추출하기 위해서는, `conditionCall(cnd)`을 사용해라

커스텀 컨디션들은 다른 요소들components을 가지고 있을수도 있다. <br /> [Section 8.5]()에서 다뤄볼 것.

또한 컨디션들은 `class` 특성을 가지고 있다. 이게 컨디션들을 S3 오브젝트들로 만들어준다. <br /> Chapter 13에서 S3을 다뤄볼 때까지 언급하지 않을 것이지만, 다행히, 컨디션 오브젝트들은 꽤나 단순하다. <br /> 가장 중요한 점은 `class` 특성은 캐릭터 벡터이며, 이게 어떤 핸들러들이 컨디션을 매치할 것인지를 결정한다는 것. <br />   it determines which handlers will match the condition.

### 8.4.2 Exiting handlers

`tryCatch()`는 exititing 핸들러를 등록register한다. <br /> tryCatch() registers exiting handlers. <br /> 보통 에러 컨디션을 다룰 때 사용하는데, <br />   에러났을 때 디폴트 행동behaviour을 바꿀 수 있도록 해준다. <br /> 예를 들어, 다음의 코드는 에러를 내는 대신, `NA`를 return할 것이다.

``` r
f3 <- function(x) {
    tryCatch(
        error = function(cnd) NA,
        log(x)
    )
}

f3("x")
## [1] NA
```

만약 아무런 컨디션들이 시그널되지 않거나, 시그널된 컨디션의 클래스가 핸들러 이름을 매치하지 않는다면, <br /> 코드는 정상적으로 실행된다.

``` r
tryCatch(
    error = function(cnd) 10,
    1 + 1
)
## [1] 2

tryCatch(
    error = function(cnd) 10,
    {
        message("Hi")
        1 + 1
    }
)
## Hi
## [1] 2
```

`tryCatch()`로 셋업한 핸들러들은 **exiting** 핸들러라고 불린다. <br /> 왜냐하면 컨디션이 시그널되고나면, 컨트롤은 핸들러에게 패스되며, 오리지널 코드로 절대 돌아오지 않는다. <br /> 한 마디로 하면, 코드를 exit한다는 뜻임.

``` r
tryCatch(
    message = function(cnd) "There",
    {
        message("Here")
        stop("This code is never run!")
    }
)
## [1] "There"
```

이 경우에는 "Here"라는 메세지가 감지되었으니깐, 위로 올라가서 "There"이 나오고, <br /> 오리지널 코드로는 절대 돌아오지 않아서, `stop()`의 것들은 evaluate되지 않는다는 것. <br />

보호된 코드protected code는 `tryCatch()`의 env에서 evaluate된다. <br /> 하지만 핸들러 코드는 그렇지 않다. 왜냐하면 핸들러들도 함수이기 때문. <br /> 이건 기억하는게 중요하다. 특히나 parent env에 있는 오브젝트들을 수정하려 할 때.

핸들러 함수들은, 컨디션 오브젝트라는 하나의 인자로 호출된다. <br /> 나는 이 인자를 관습에 따라 `cnd`라고 부른다. <br /> 이 값은, base 컨디션들에 대해선 조금만 유용하다. 왜냐하면 정보가 적기 때문. <br /> 곧 보게될텐데, 이건 커스텀 컨디션들을 만들 때 특히나 더 유용하다.

``` r
path <- tempfile()
tryCatch(
    {
        writeLines("Hi!", path)
        # ...
    },
    finally = {
        # 항상 실행됨
        unlink(path)
    }
)
```

### 8.4.3 Calling handlers

`tryCatch()`로 셋업된 핸들러들은 exiting handlers라고 부른다. <br /> 왜냐하면 한 번 컨디션이 잡히고나면 코드를 exit하기 때문. <br /> 반대로, `withCallingHandlers()`는 **calling** handlers를 셋업한다. <br />   핸들러가 return하고 난 뒤에도 코드 실행은 정상적으로 계속된다. <br /> 이래서 `withCallingHandelrs()`는 에러가 아닌 컨디션들과 자연스럽게 짝을 짓는다. <br /> exiting handler와 calling handler는 handler라는 단어를 좀 다른 의미로 사용한다.

-   exiting handler는, 문제를 다루는handle 것처럼, 시그널을 handle한다. 문제가 사라지도록 함. <br />
-   calling handler는, 차를 핸들handle하는 것처럼, 시그널을 handle한다. 차는 여전히 남아있음.

아래의 `tryCatch()`와 `withCallingHandlers()`의 결과물을 비교해보자. <br /> 첫 번째 예에서는 메세지가 프린트되지 않는다. <br /> 왜냐하면 exiting handler가 완료되고나면 코드가 종료되기 때문.

두 번째 예에서는 메세지가 프린트된다. <br /> 왜냐하면 calling handler는 exit하지 않기 때문.

``` r
tryCatch(
    message = function(cnd) cat("Caught a message!\n"),
    {
        message("Someone there?")
        message("Why, yes!")
    }
)
## Caught a message!
```

``` r
withCallingHandlers(
    message = function(cnd) cat("Caught a message!\n"),
    {
        message("Someone there?")
        message("Why, yes!")
    }
)
## Caught a message!
## Someone there?
## Caught a message!
## Why, yes!
```

핸들러들은 차례대로 적용되기 때문에applied in order, 무한 루프에 갇힐 걱정을 하지 않아도 된다. <br /> 다음의 예에서, 핸들러에 의해 시그널된 `message()`는 잡히지 않는다.

``` r
withCallingHandlers(
    message = function(cnd) message("Second message"),
    message("First message")
)
## Second message
## First message
```

그러니깐 메세지가 감지되어서 메세지가 나오는데 이게 또 감지되지 않을까 걱정할 수 있는데, 그렇지 않다는 것. <br /> (하지만 여러개의 핸들러가 있을 때에는 조심해라. 핸들러에 의해 캡쳐된 시그널을, 다른 핸들러가 또 시그널할 수 있음. <br /> 순서를 철저하게 잘 생각해야 한다.)

calling handler의 return 값은 무시된다. <br /> 왜냐하면 핸들러가 완료되더라도 코드는 계속해서 실행되기 때문.

그럼 return값은 어디로 가는지? <br /> 그래서 calling handlers는 side-effect를 위해서 쓸 때가 가장 효과적이라는 것.

calling handler의 고유한, 중요 side-effect는 시그널을 **muffle**할 수 있는 것. <br /> 디폴트로, 컨디션은 parent handler까지 계속해서 전달propagate된다. <br /> 디폴트 핸들러까지 전달됨.(혹은 exiting 핸들러가 있다면, 거기까지) <br /> 다음의 예를 보자.

``` r
# 메세지를 만드는 디폴트 핸들러까지 올라감bubble up
withCallingHandlers(
    message = function(cnd) cat("Level 2\n"),
    withCallingHandlers(
        message = function(cnd) cat("Level 1\n"),
        message("Hello")
    )
)
## Level 1
## Level 2
## Hello
```

``` r
# tryCatch까지 올라감bubble up
tryCatch(
    message = function(cnd) cat("Level 2\n"),
    withCallingHandlers(
        message = function(cnd) cat("Level 1\n"),
        message("Hello")
    )
)
## Level 1
## Level 2
```

올라가는 걸bubble up 방지하고 싶지만 코드 블락의 나머지들은 여전히 실행하고 싶다면, <br /> `rlang::cnd_muffle()`을 이용해서 명백하게 muffle할 수는 있다.

``` r
withCallingHandlers(
    message = function(cnd) {
        cat("Level 2\n")
        cnd_muffle(cnd)
    },
    withCallingHandlers(
        message = function(cnd) cat("Level 1\n"),
        message("Hello")
    )
)
## Level 1
## Level 2
```

원래는, <br /> Level 1 <br /> Level 2 <br /> Hello <br /> 이렇게 나와야하는데

`cnd_muffle(cnd)` 이 Level 2 다음에 있으니깐, Hello는 안 나옴 <br />

``` r
withCallingHandlers(
    message = function(cnd) {
        cat("Level 2\n")
    },
    withCallingHandlers(
        message = function(cnd) {
            cat("Level 1\n")
            cnd_muffle(cnd)
        },
        message("Hello")
    )
)
## Level 1
```

그러니깐 디폴트로 컨디션은 parent handler까지 계속해서 전달되는데, <br /> 이걸 `cnd_muffle()`을 통해서 방해muffle할 수 있다는거군.

### 8.4.4 Call stacks

section을 끝내기전에, exiting 핸들러의 콜 스택과 calling 핸들러간의 콜 스택간의 차이점을 알아보자. <br /> 이 차이점들이 그리 중요하지는 않은데, 가끔 유용할 수 있기 때문에 여기 넣어놨다.

`lobstr::cst()`을 사용하는 작은 예를 이용해서 차이점을 살펴보는게 가장 쉽다.

``` r
f <- function() g()
g <- function() h()
h <- function() message("!")
```

calling 핸들러는 호출된다. 컨디션을 시그널하는 호출call의 문맥에 따라. <br /> Calling handlers are called in the context of the call that signalled the condition.

``` r
withCallingHandlers(f(), message = function(cnd) {
    lobstr::cst()
    cnd_muffle(cnd)
})
##      x
##   1. +-base::withCallingHandlers(...)
##   2. +-global::f()
##   3. | \-global::g()
##   4. |   \-global::h()
##   5. |     \-base::message("!")
##   6. |       +-base::withRestarts(...)
##   7. |       | \-base:::withOneRestart(expr, restarts[[1L]])
##   8. |       |   \-base:::doWithOneRestart(return(expr), restart)
##   9. |       \-base::signalCondition(cond)
##  10. \-(function (cnd) ...
##  11.   \-lobstr::cst()
```

exiting 핸들러는 호출된다. `tryCatch()`를 호출하는 문맥에 따라. <br /> Exiting handlers are called in the context of the call to tryCatch().

``` r
tryCatch(f(), message = function(cnd) lobstr::cst())
##     x
##  1. \-base::tryCatch(f(), message = function(cnd) lobstr::cst())
##  2.   \-base:::tryCatchList(expr, classes, parentenv, handlers)
##  3.     \-base:::tryCatchOne(expr, names, parentenv, handlers[[1L]])
##  4.       \-value[[3L]](cond)
##  5.         \-lobstr::cst()
```

### 8.4.5 Exercises

------------------------------------------------------------------------

8.5 Custom conditions
---------------------

R에서 에러를 핸들링하는 것의 문제 중 하나는, <br />   대부분의 함수들이 이미 만들어져 있는built-in 컨디션들을 생성한다는 것. <br /> 이것들은 `message`랑 `call`만을 가지고 있음. <br /> 이말인즉슨, 특정한 타입의 에러를 감지하고 싶다면, 에러 메세지의 텍스트 가지고서만 작업을 할 수 있다는 것. <br /> 이건 에러가 나기 쉽다. 메세지가 시간이 지남에 따라 달라질뿐 아니라, 메세지가 다른 언어들로 번역이 될 수도 있기 때문.

다행히도 R은 강력한 기능이 있다. 물론 별로 사용이 덜 되긴하지만. <br /> 추가적인 메타데이터를 가질 수 있는 커스텀 컨디션을 만들 수 있다. <br /> 커스텀 컨디션을 만드는 것은 base R에서는 성가시다. <br /> 하지만 `rlang::abort()`는 커스텀 `.subClass`랑 추가적인 메타데이터를 줄 수 있어서 매우 쉽다.

다음의 예는, 기본적인 패턴을 보여준다. <br /> 커스텀 컨디션에 대해서는, 다음의 콜 구조 사용할 것을 추천한다. <br /> R의 유연한 인자 매칭flexible argument matching을 이용해서, <br />   에러 타입의 이름이 첫 번째로 나오고, <br />   사용자가 보게 되는 텍스트가 그 다음으로 나오고,user facing text <br />   커스텀 메타데이터custom metadata가 그 다음으로 나온다.

``` r
abort(
    "error_not_found",
    message = "Path `blah.csv` not found",
    path = "blah.csv"
)
## Error: Path `blah.csv` not found
```

커스텀 컨디션들은 인터랙티브하게 사용할 때는 평범한 컨디션들같이 작동한다. <br /> 하지만 핸들러들이 좀 더 많은걸 할 수 있게해줌.

### 8.5.1 Motivation

`base::log()`를 이용해 좀 더 자세하게 알아보자. <br /> 유효하지 않은 인자들arguments로 인해 에러가 나올 때, 너무 불친절하다.

``` r
log(letters)
## Error in log(letters): 수학함수에 숫자가 아닌 인자가 전달되었습니다
log(1:10, base = letters)
## Error in log(1:10, base = letters): 수학함수에 숫자가 아닌 인자가 전달되었습니다
```

어떤 인자argument가 문제였는지를 알려주고,(`x`가 문제인지 `base`가 문제인지) <br />   어떤 input이 문제였는지를 알려줌으로써 좀 더 명백해질 수 있다고 생각한다.

``` r
my_log <- function(x, base = exp(1)) {
  if (!is.numeric(x)) {
    abort(paste0(
      "`x` must be a numeric vector; not ", typeof(x), "."
    ))
  }
  
  if (!is.numeric(base)) {
    abort(paste0(
      "`base` must be a numeric vector; not ", typeof(base), "."
    ))
  }
  
  base::log(x, base = base)
}
```

이러고 나면,

``` r
my_log(letters)
## Error: `x` must be a numeric vector; not character.
my_log(1:10, base = letters)
## Error: `base` must be a numeric vector; not character.
```

이제 최소한, 어떤 인자가 잘못되었는지는 알게 됐다. <br /> 인터랙티브한 사용 측면에서, 이건 발전이다. <br /> 왜냐하면 에러 메세지가, 사용자로 하게끔 정확한 수정으로 유도했기 때문. <br /> 하지만, 프로그램적으로 에러를 핸들하기 원했다면, 나아진 건 없는거다. <br /> they're no better if you want to programmatically handle the errors <br /> 에러에 대한 모든 유용한 메타데이터는, 하나의 스트링으로 축약됐다.

### 8.5.2 Signalling

이 상황을 발전시키기 위한 기초적 구조infrastructure를 만들어보자. <br /> 잘못된 인자argument에 대한 커스텀 `abort()`를 공급하는 것에서 시작해볼거다. <br /> 이건 너무 일반화시킨 예제이긴한데, 다른 함수들에서도 나타나는 일반적인 패턴을 반영한다. <br /> 이 패턴은 꽤나 단순하다.

사용자user를 위해선, `glue::glue()`를 이용해 나이스한 에러 메세지를 만들어준다. <br /> 개발자developer를 위해선, 컨디션 콜condition call에 있는 메타데이터를 저장한다.

``` r
abort_bad_argument <- function(arg, must, not = NULL) {
    msg <- glue::glue("`{arg}` must {must}")
    if (!is.null(not)) {
        not <- typeof(not)
        msg <- glue::glue("{msg}; not {not}.")
    }

    abort("error_bad_argument",
        message = msg,
        arg = arg,
        must = must,
        not = not
    )
}
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
<strong>base R에서는</strong> <br /> rlang을 쓰지않고 커스텀 에러를 만들고 싶다면, 컨디션 오브젝트를 직접 만들고by hand, <br /> 아래처럼, <code>stop()</code>에게 전달해주면 된다.

``` r
stop_custom <- function(.subClass, message, call = NULL, ...) {
    err <- structure(
        list(
            message = message,
            call = call,
            ...
        ),
        class = c(.subClass, "error", "condition")
    )
    stop(err)
}

err <- catch_cnd(
    stop_custom("error_new", "This is a custom error", x = 10)
)

class(err)

err$x
```

</p>
이제, 이 새로운 헬퍼를 이용해서 `my_log()`를 다시 써볼 수 있다.

``` r
my_log <- function(x, base = exp(1)) {
    if (!is.numeric(x)) {
        abort_bad_argument("x", must = "be numeric", not = x)
    }
    if (!is.numeric(base)) {
        abort_bad_argument("base", must = "be numeric", not = base)
    }

    base::log(x, base = base)
}
```

`my_log()` 자체는 이제 좀 길어졌는데, 하지만 훨씬 의미를 많이 전달한다. <br /> 그리고 에러 메세지가, 함수들에 걸쳐, 잘못된 인자들에 대해 일관성 있게 되었다. <br /> 이전과 같이, 인터랙티브 에러 메세지를 내준다.

``` r
my_log(letters)
## Error: `x` must be numeric; not character.
my_log(1:10, base = letters)
## Error: `base` must be numeric; not character.
```

### 8.5.3 Hanlding

이렇게 구조화된 컨디션 오브젝트는, 프로그램하기가 훨씬 쉬워진다. <br /> 이 기능을, 니가 만든 함수를 테스트할 때 써볼 수 있다. <br /> 유닛 테스팅unit testing은 이 책의 주제가 아니지만, 기본은 이해하기 쉽다.(자세하게 알고 싶다면, R packages 참고) <br /> (간단히 찾아보니, 프로그래밍에서 소스 코드의 특정 모듈이, 의도된 대로 정확히 작동하는지 검증하는 절차) <br /> 다음의 코드는 에러를 캡쳐하고, 우리가 기대한 구조를 가지고 있다는 걸 보여준다.

``` r
library(testthat)
err <- catch_cnd(my_log("a"))
expect_s3_class(err, "error_bad_argument")
expect_equal(err$arg, "x")
expect_equal(err$not, "character")
```

(위에 있는 라이브러리랑 함수들은 저자Hadley가 직접 만든 것들이다. <br /> `my_log("a")`가 내놓은 컨디션을 캐치한 것을 바탕으로, 이것저것 원하는대로 되었는지 유닛 테스팅을 한 것.)

또는, `tryCatch()`의 `error_bad_argument` 클래스를 사용해서, 특정한 에러를 handle하도록할 수도 있다.

``` r
tryCatch(
    error_bad_argument = function(cnd) "bad_argument",
    error = function(cnd) "other error",
    my_log("a")
)
## [1] "bad_argument"
```

`tryCatch()`를 여러 개의 handlers와 커스텀 클래스들과 함께 사용할 때, <br />   시그널의 클래스 벡터를 매치하는 첫 번째 핸들러가 호출된다. 최선의 매치가 아니라. <br /> the first handler to match any class in the signal's class vector is called. <br /> 이러한 이유에서, 가장 특정한 핸들러를 맨 첨에 놓아야한다. <br /> 다음의 코드는 원하는대로 되질 않는다.

``` r
tryCatch(
    error = function(cnd) "other error",
    error_bad_argument = function(cnd) "bad_argument",
    my_log("a")
)
## [1] "other error"
```

### 8.5.4 Exercises

------------------------------------------------------------------------

8.6 Applications
----------------

R의 컨디션 시스템에 대한 기본적인 툴들을 다 배워봤으니, 몇몇 응용들을 해보자. <br /> 이 section의 목표는, `tryCatch()`나 `withCallingHandlers()`의 모든 가능한 사용법에 대해 배우는게 아니라, <br />   자주 일어나는 몇몇 일반적인 패턴들을 그려illustrate보는 것. <br /> 이러고나면 너의 창조적인 주스를 흐르게 해서(ㅋㅋ), 새로운 문제를 만났을 때, 유용한 해결법을 생각해낼 수 있을 것이다. <br /> Hopefully these will get your creative juices flowing, so when you encounter a new problem you can come up with a useful solution.

<details> <summary>...</summary> 중학교 때인가, 영어 선생이 이런 말을 했었다. <br /> '영어는 그림을 그리는 언어라고 한다. 그만큼 표현이 풍부하다.' <br /> 당시에는 말도 안 된다고 생각했다. 무슨 ㅋㅋ 우리나라 말도 얼마나 풍부한 표현이 많고 시적인 표현도 많고 좋은 문학 작품도 많고.. <br /> (물론 접한 영문학은 많지 않지만(..이건 지금도..)) <br /> 그런데 그로부터 10년 이상이 지나고, 그 동안 여러가지 우리나라 문학이나, 수천 수만가지 영어 텍스트들을 접하고 나니, <br /> 영어가 더 표현력이 풍부한 것 같다. <br /> 시나 소설 같은 문학적인 분야에선 몰라도, 일상적인 표현에서는 영어가 더 표현력이 풍성하다. <br /> 머릿속에 당장 떠오르는 표현으로도, Sounds good! 을 우리나라 말로 번역하려하면 '좋아요'밖에 없다. <br /> 뭐 이건 너무 단편적일지라도, 경험적으로, 당시 영어 선생에게 동의할 수 밖에 없다. </details> <br /> <br />

들어가기 전에, 이미 한 번 다 봐본 내가 정리를 해봤다.

1.  Failure values, 에러가 발생했을 때, 에러 대신에 지정해놓은 디폴값을 반환하도록. <br />
2.  Success and failure values, 1번을 더 확장해서, 성공했을 때는 그 값을, 실패했을 땐, 지정해놓은 값을 반환하도록. <br />
3.  Resignal, 맨 처음 봤을 때는, resign의 명사형인가? 생각했는데, 그게 아니고, re - signal이다. <br /> warning이 나왔을 때, 이걸 error로 바꿔서 return하도록. <br />
4.  Record, 무슨 무슨 값들이 나왔는지 기록하는 것. <br />
5.  No default behaviour, 이건 설명하기가 좀 힘듬. 읽어봐야함.

### 8.6.1 Failure values

에러 핸들러로부터 반환return되는 값에 따라, 단순하지만 유용한 몇 개의 `tryCatch()` 패턴들이 있다. <br /> 가장 간단한 케이스는, 에러가 발생할 때, 디폴트 값에다가 wrapper을 씌우는 것이다.

``` r
fail_with <- function(expr, value = NULL) {
    tryCatch(
        error = function(cnd) value,
        expr
    )
}

fail_with(log(10), NA_real_)
## [1] 2.302585
fail_with(log("x"), NA_real_)
## [1] NA
```

좀 더 복잡한 응용은, `base::try()`다. <br /> 아래에, `try2()`는 `base::try()`의 에센스만 뽑았다. <br /> 원래의 `try()`는, `tryCatch()`를 사용하지 않은 경우 에러 메세지를 더 보여줘야 되어서, 더 복잡하다.

``` r
try2 <- function(expr, silent = FALSE) {
    tryCatch(
        error = function(cnd) {
            msg <- conditionMessage(cnd)
            if (!silent) {
                message("Error: ", msg)
            }
            structure(msg, class = "try-error")
        },
        expr
    )
}

try2(1)
## [1] 1
try2(stop("Hi"))
## Error: Hi
## [1] "Hi"
## attr(,"class")
## [1] "try-error"
try2(stop("Hi"), silent = TRUE)
## [1] "Hi"
## attr(,"class")
## [1] "try-error"
```

### 8.6.2 Success and failure values

이 패턴을 확장시켜서, 만약 코드가 성공적으로 evaluate되면 어떤 값을 return하도록(`success_val`) 하고, <br />   실패하면 다른 값을 return하도록(`error_val`) 할 수 있다.

이 패턴은 하나의 작은 트릭만 있으면 된다: <br />   사용자 공급 코드를 evaluate하고, `success_val` <br />   만약 코드가 에러를 내면, `success_val`로는 가지 못하고, 대신에 `error_val`을 return한다.

``` r
foo <- function(expr) {
    tryCatch(
        error = function(cnd) error_val,
        {
            expr
            success_val
        }
    )
}
```

이걸 사용해서 expression이 실패했는지를 알 수 있다.

``` r
does_error <- function(expr) {
    tryCatch(
        error = function(cnd) TRUE,
        {
            expr
            FALSE
        }
    )
}
```

혹은 어떠한 컨디션이라도 캡쳐하는데도 쓸 수 있다. `rlang::catch_cnd()`처럼

``` r
catch_cnd <- function(expr) {
    tryCatch(
        condition = function(cnd) cnd,
        {
            expr
            NULL
        }
    )
}
```

`try()`의 변형variant을 만드는데 있어 이 패턴을 사용할 수도 있다. <br /> `try()`의 문제 중 하나는, 코드가 성공했는지 아닌지를 판단하는게 상당히 어렵다는 것. <br /> 오브젝트를 특정한 클래스와 함께 return하는 것보다는, <br />   result와 error라는 2개의 요소component를 갖고 있는 리스트를 return하는 것이 더 나은 것 같다.

``` r
safety <- function(expr) {
    tryCatch(
        error = function(cnd) {
            list(result = NULL, error = cnd)
        },
        list(result = expr, expr = NULL)
    )
}
str(safety(1 + 10))
## List of 2
##  $ result: num 11
##  $ expr  : NULL
str(safety(stop("Error!")))
## List of 2
##  $ result: NULL
##  $ error :List of 2
##   ..$ message: chr "Error!"
##   ..$ call   : language doTryCatch(return(expr), name, parentenv, handler)
##   ..- attr(*, "class")= chr [1:3] "simpleError" "error" "condition"
```

이건 `purrr::safely()`와 밀접한 관련이 있다. <br /> 이건 function operator인데, [Section 11.2.1]()에서 배우게 될 것이다.

### 8.6.3 Resignal

컨디션이 시그널되었을 때 디폴트 값을 return하는 것과 마찬가지로, <br />   핸들러들은 좀 더 정보가 많은 에러 메세지들을 만드는데 사용될 수 있다. <br /> 하나 간단한 응용은, 하나의 코드 블락에 대해 `options(warn = 2)`과 같이 작동하는 함수를 만드는 것. <br /> 아이디어는 간단하다: 에러를 만들어냄으로써 경고를 다룬다. <br /> we handle warnings by throwing an error.

``` r
warning2error <- function(expr) {
    withCallingHandlers(
        warning = function(cnd) abort(conditionMessage(cnd)),
        expr
    )
}
```

``` r
warning2error({
  x <- 2 ^ 4
  warn("Hello")
})
## Error: Hello
```

성가신 메세지의 소스를 찾으려고 할 때, 비슷한 함수를 작성할 수 있다. <br /> Section 22.6에서 더 볼 수 있다.

### 8.6.4 Record

다른 흔한 패턴은, 나중의 조사investigation를 위해 컨디션들을 기록record해두는 것. <br /> 여기서 문제는, calling 핸들러들은 side-effects를 위해서 호출되기에 값을 return받을 수 없다. <br /> 하지만 대신에 몇몇 오브젝트를 수정해줘야 한다.

``` r
catch_cnds <- function(expr) {
    conds <- list()
    add_cond <- function(cnd) {
        conds <<- append(conds, list(cnd))
        cnd_muffle(cnd)
    }

    withCallingHandlers(
        message = add_cond,
        warning = add_cond,
        expr
    )

    conds
}
```

``` r
catch_cnds({
    inform("a")
    warn("b")
    inform("c")
})
## [[1]]
## <message: a>
## 
## [[2]]
## <warning: b>
## 
## [[3]]
## <message: c>
```

근데, 또 에러를 캡쳐하기 원한다면 어떻게 해야할까? <br /> `tryCatch()`안의 `withCallingHandlers()`를 감싸야한다. <br /> 만약 에러가 발생한다면, 그게 마지막 컨디션이 될 것이다.

``` r
catch_cnds <- function(expr) {
    conds <- list()
    add_cond <- function(cnd) {
        conds <<- append(conds, list(cnd))
        cnd_muffle(cnd)
    }

    tryCatch(
        error = function(cnd) {
            conds <<- append(conds, list(cnd))
        },
        withCallingHandlers(
            message = add_cond,
            warning = add_cond,
            expr
        )
    )

    conds
}
```

``` r
catch_cnds({
    inform("a")
    warn("b")
    abort("c")
})
## [[1]]
## <message: a>
## 
## [[2]]
## <warning: b>
## 
## [[3]]
## <error/rlang_error>
## c
## Backtrace:
##   1. rmarkdown::render(...)
##  19. global::catch_cnds(...)
##  24. base::withCallingHandlers(...)
```

이것이, knitr을 지원하는 evaluate 패키지([Wickham and Xie 2018](https://github.com/r-lib/evaluate))의 핵심 아이디어다. <br /> 모든 아웃풋을 특별한 데이터 구조로 캡쳐해서, 나중에 replay될 수 있도록 해준다. <br /> 전체적으로는, evaluate 패키지가 여기있는 것들보다 훨씬 복잡하다. <br /> plots랑 text 아웃풋들도 다뤄야하기 때문.

### 8.6.5 No default behaviour

마지막으로 유용한 패턴은, `message`나 `warning` 혹은 `error`로부터 inherit하지 않는 컨디션을 시그널하는 것. <br /> 이 말은, 디폴트 행동behaviour이 없기 때문에, 컨디션이 아무런 효과effect가 없다는 것. <br /> 사용자가 따로 딱 정해주지 않는 이상. <br /> 예를 들어, 컨디션들에 따른 로깅 시스템을 상상해볼 수 있다.

``` r
log <- function(message, level = c("info", "error", "fatal")) {
    level <- match.arg(level)
    signal(message, "log", level = level)
}
```

다음과 같이, <br /> `log()`를 호출할 때, 컨디션은 시그널되지만, 아무런 디폴트 핸들러가 없기 때문에, 아무일도 일어나지 않는다.

``` r
log("This code was run")
```

logging을 activate하기 위해서, log 컨디션을 이용해 무언가를 해주는 핸들러가 필요하다. <br /> 밑에다, 파일에다가 logging 메세지들을 기록하는, `record_log()` 함수를 정의해놨다.

``` r
record_log <- function(expr, path = stdout()) {
    withCallingHandlers(
        log = function(cnd) {
            cat(
            "[", cnd$level, "] ", cnd$message, "\n", sep = "",
            file = path, append = TRUE
            )
        },
        expr
    )
}

record_log(log("Hello"))
## [info] Hello
```

다른 함수를 layering해서, 몇몇 logging 레벨들을 선택적으로 억눌러suppress주는 것도 가능하다.

``` r
ignore_log_levels <- function(expr, levels) {
    withCallingHandlers(
        log = function(cnd) {
            if (cnd$level %in% levels) {
                cnd_muffle(cnd)
            }
        },
        expr
    )
}

record_log(ignore_log_levels(log("Hello"), "info"))
```

<p class="comment">
<strong>base R에서는</strong> <br /> 컨디션 오브젝트를 손으로 만들고, <code>signalCondition()</code>, <code>cnd\_muffle()</code>로 시그널하려하면 안 될 것이다. <br /> 대신에 muffle restart라는걸 다음과 같이 정의해야한다. <br />
</p>
``` r
withRestarts(signalCondition(cond), muffle = function() NULL)
```

<p class="comment">
restarts는 현재로선 이 책의 범위를 넘지만, 3판쯤에는 포함될거라 생각한다.
</p>
### 8.6.6 Exercises

------------------------------------------------------------------------

8.7 Quiz answers
----------------

1.  `error`, `warning`, `message`

2.  `try()` 혹은 `tryCatch()`

3.  `tryCatch()`는 wrapped code의 실행을 끝내는, exiting handler를 만듬. <br /> `withCallingHandlers()`는 wrapped code의 실행에 영향을 주지 않는, calling handler를 만듬.

4.  왜냐하면, `tryCatch()`를 이용해서 특정한 타입의 에러를 캡쳐할수도 있다. <br /> 반면에, 단순히 에러 문자열error strings를 비교에 기대는 것은 메세지가 translate될 수 있어 위험할 수 있음
