19.1 Introduction
-----------------

이제 R 코드의 tree 구조에 대해 이해를 했으니, `expr()`나 `ast()`가 work하는 원리에 대해 알아보자: 바로 quotation. tidy evaluation에서, 모든 quoting 함수는 사실 quasiquoting 함수다. 왜냐하면 unquoting 또한 지원하기 때문.

quotation은 unevaluated expression을 캡처링하는 act인데 비해, unquotation은 quoted expression이었을 부분을 선택적으로 evaluate하는 그런 기능. 이걸 다 합쳐서 quasiquotation이라고 함. quasiquotation은 어떤 함수를 create하는 것을 쉽게 만들어준다. 어떤 함수? 함수 작성자author가 만든 코드와 함수 사용자user가 만든 코드를 결합combine해주는 함수. 이게 넓은 범위의 어려운 문제들을 해결해준다.

quasiquotation은 tidy evaluation의 3가지 기둥 중 하나다. 나머지 둘인 quosures와 data mask에 대해서는 Chapter 20에서 배울 것.

홀로 사용했을 때에는, quasiquotation은 가장 유용한 프로그래밍. 특히나 코드를 만드는데generating 있어서. 하지만 다른 테크닉들과 결합되면, tidy evaluation은 data analysis를 위한 강력한 툴이 된다.

### Outline

-   Section 19.2에서는 quasiquotation의 개발 동기가 되는 함수, `cement()`에 대해 알아봄. `paste()`와 비슷하게 작동하지만, 자동으로 함수 인자들을 따옴표quote해주기 때문에, 직접 안 해도 된다.

-   Section 19.3에서는 expressions를 quote하는 도구를 준다. 그 expression을 작성자가 만들었든 사용자가 만들었든, rlang을 사용하든 base R을 사용하든.

-   Section 19.4에서는, rlang quoting 함수와 base quoting 함수 간의 가장 큰 차이점을 소개한다.: `!!`로 unquoting하는 것과 `!!!`로 unquoting하는 것.

-   Section 19.5에서는, quoting하는 동작을 비활성화disable하는데 base R 함수들이 사용하는, 3개의 주요 non-quoting 테크닉들을 다룬다.

-   Section 19.6에서는, `!!!`을 사용할 수 있는 또 다른 장소, `...`을 받는 함수에 대해 알아본다. 또, 특별한 연산자인 `:=`를 소개하는데, 얘는 인자 이름argument names을 다이나믹하게 바꿀 수 있도록 해준다.

-   Section 19.7에서는, 몇몇 코드 생성을 자연적으로 필요로 하는 문제를 해결하는데 있어, quoting의 실용적인 사용법 몇 개를 보여준다.

-   Section 19.8에서는, quasiquotation에 대해 짧은 역사로 마무리. 혹시나 누군가 궁금해할까봐...

### Prerequisites

동기 부여에 대한 전체적인 개요와 기본적인 단어들을 위해, [Chapter 17](https://blog-for-phil.readthedocs.io/en/latest/Advanced%20R/17-Big-Picture/), metaprogramming overview를 읽고 오자. 그리고 [Section 18.3](https://blog-for-phil.readthedocs.io/en/latest/Advanced%20R/18-Expressions/#183-expressions)에 설명되어 있는, expression의 tree 구조에 친숙해야 한다.

코드 측면에서는, 대부분의 경우, rlang에 있는 툴들을 사용할 것이다. 하지만 chapter의 끝 부분에서, purrr과 연동해 강력한 응용을 보게 될 것이다.

``` r
library(rlang)
library(purrr)
```

### Related work

quoting 함수들은, Lisp **macros**와 깊은 관련이 있다. 하지만 macros는 보통 컴파일-타임에 실행되는데, R에는 이게 존재하지 않는다. 그리고 항상 input이랑 output이 AST다. 이걸 R에다가 구현하려는 접근법으로는 Lumley([2001](https://www.r-project.org/doc/Rnews/Rnews_2001-3.pdf))를 참고해라. quoting 함수들은, 소수만 아는, Lisp의 fexprs와 더 밀접한 연관이 있는데, 이 함수들은, 모든 인자들을 디폴트로 quote한다. 이 용어들은, 다른 프로그래밍 언어에서 연관된 작업related work을 찾아볼 때, 유용하다.

------------------------------------------------------------------------

19.2 Motivation
---------------

unquoting의 필요성을 보여주는, 구체적인 예시와 함께 시작해보자. 결국엔 이게 quasiquotation으로 연결 words를 joining함으로써 strings를 만든다고 상상해보자.

``` r
paste("Good", "morning", "Hadley")
## [1] "Good morning Hadley"
paste("Good", "afternoon", "Alice")
## [1] "Good afternoon Alice"
```

이렇게 따옴표quotes를 다 넣는건 지루하다. 그냥 생 단어bare words를 사용하고 싶다. 그러고 싶다면, 다음의 함수를 사용하면 된다. (각 조각들에 대해선 걱정하지마라. 나중에 배울 것이다.)

``` r
cement <- function(...) {
    args <- ensyms(...)
    paste(purrr::map(args, as_string), collapse = " ")
}
```

``` r
cement(Good, morning, Hadley)
## [1] "Good morning Hadley"
cement(Good, afternoon, Alice)
## [1] "Good afternoon Alice"
```

형식적으로, 이 함수는 모든 인풋들을 따옴표quote해준다. 자동적으로 각 인자argument에다가 따옴표를 넣어준다고 생각할 수도 있는데, 그건 정확한 사실은 아니다. 왜냐하면 중간 결과물들이 expression이지, string이 아니기 때문. 하지만 여전히 유용한 근사치approximations이긴하다. 그리고 이게 "quote"라는 용어의 근본적인 의미기도 하고.

이 함수는 더 이상 따옴표를 하나하나 안 쳐도 된다는 점에서 nice하다. 하지만 문제는, variables를 사용하고 싶을 때 발생한다. `paste()` 때는 variables 사용하기 쉬웠다. 그냥 따옴표를 안치면 됐으니깐.

``` r
name <- "Hadley"
time <- "morning"

paste("Good", time, name)
## [1] "Good morning Hadley"
```

당연히, `cement()`에서는 안 된다. 왜냐하면 모든 input들을 자동적으로 quote하기 때문.

``` r
cement(Good, time, name)
## [1] "Good time name"
```

input을 명백하게explicitly, unquote할 방법이 필요하다. `cement()`라는 함수가 자동적인 따옴표 치는 걸 막아줄 그런 방법. 여기선 , `time`과 `name`을, `Good`과는 다르게 다루어줘야 한다.

quasiquotation은 이걸 할 수 있는 방법을 준다. `!!`, unquote라고 불리는 기본적인 도구. 뱅뱅이라고 부름. 이건 quoting 함수에게 quote하지 말 것을 전달해준다.

``` r
cement(Good, !!time, !!name)
## [1] "Good morning Hadley"
```

`cement()`와 `paste()`를 직접적으로 비교하는건 유용하다. `paste()`는 인자들arguments을 evaluate하기 때문에, quote가 필요하면 해줘야 함. `cement()`는 인자들arguments을 quote하기 때문에, unquote가 필요하면 해줘야 한다.

``` r
paste("Good", time, name)
cement(Good, !!time, !!name)
```

### 19.2.1 Vocabulary

quote된 인자들arguments과, evaluate된 인자들을 구분하는 것은 중요하다.

-   evaluate된 인자들은 R의 usual evaluation rule을 따른다.

-   quote된 인자들은 함수에 의해 캡처된 것으로, 몇몇 커스텀 방법으로 처리됨.

`paste()`는 모든 인자들을 evaluate하고, `cement()`는 모든 인자들을 quote한다.

어떤 인자가 quote된건지 evaluate된건지 확신이 없다면, 코드를 함수 밖에서 실행해보라. 만약에 작동하지 않는다거나, 다른 행동을 한다면, 인자는 quote된 것. 예를 들어서, `library()`에서 첫 번째 인자는 quote일까, evaluate일까?

``` r
# 작동됨
library(MASS)

# 안 됨
MASS
## Error in eval(expr, envir, enclos): 객체 'MASS'를 찾을 수 없습니다
#> Error ...
```

그러니깐 quote되어 있는 것임.

인자가 quote된건지 evaluate된건지에 대해 말하는 것은, 함수가 non-standard evaluation(NSE)를 사용하는것인지 아닌지를, 좀 더 정확하게 나타내는 것.

난 가끔, 하나 혹은 여러 개의 인자들을 quotes하는 함수들을, 짧게 quoting function이라고 부를 것이다. 하지만 일반적으로, quoted 인자들에 대해 얘기할 것이다. 왜냐하면 거기서부터 차이가 발생하는 것이기 때문.

### 19.2.2 Exercises

------------------------------------------------------------------------

19.3 Quoting
------------

------------------------------------------------------------------------

19.4 Unquoting
--------------

------------------------------------------------------------------------

19.5 Non-quoting
----------------

------------------------------------------------------------------------

19.6 `...` (dot-dot-dot)
------------------------

------------------------------------------------------------------------

19.7 Case studies
-----------------

------------------------------------------------------------------------

19.8 History
------------
