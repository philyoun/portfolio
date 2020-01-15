18.1 Introduction
-----------------

언어language를 계산하기 위해서는, 먼저 그것의 구조에 대해 이해할 필요가 있다. <br /> 이러기 위해서는 새로운 단어, 새로운 툴들, R 코드에 대해 생각하는 새로운 방법들이 필요. <br /> 첫 번째로, operation과 그 결과에 대해 구분해야한다.

변수 `x`에 10을 곱하고 그 결과를 `y`라는 새로운 변수에 저장하는 걸 해보자. <br /> `x`가 무엇인지 정의되지 않았기 때문에, 당연히 작동 안 한다.

``` r
y <- x * 10
## Error in eval(expr, envir, enclos): 객체 'x'를 찾을 수 없습니다
```

코드를 실행시키지 않고, 이 코드의 의도intent만 캡쳐만 할 수 있다면 좋을 것이다. <br /> It would be nice / if we could capture the intent of the code / without executing it. <br /> 다른 말로, 어떻게 action의 묘사description(코드 의도)와 action 그 자체(코드 실행)를 구분할 수 있을까? <br /> 한 가지 방법은 `rlang::expr()`을 쓰는 것

``` r
z <- rlang::expr(y <- x * 10)
z
## y <- x * 10
```

`expr()`은 expression을 반환한다. 코드 구조를 evaluating하지 않고 캡쳐한 오브젝트

expression을 가지고 있다면, `base::eval()`을 통해서 evaluate할 수 있다.

``` r
x <- 4
eval(z)
y
## [1] 40
```

이 장에서는, expressions 밑에 있는 데이터 구조에 대해 알아볼 것. <br /> 이 것에 대해 마스터하게 되면, <br />   캡쳐된 코드를 점검inspect하고 수정modify할 수 있고, 코드로 코드를 만들 수 있다. <br /> generate code with code.

`expr()`에 대해서는 19장에서, `eval()`에 대해서는 20장에서 다룸.

### Outline

-   Section 18.2에서는, abstract syntax tree(AST)의 아이디어를 소개하고, <br />   모든 R 코드 밑에 있는 나무 모양tree like 구조를 보여준다.

-   Section 18.3에서는, AST를 뒷받침하는 데이터 구조의 디테일들을 알아볼 것. <br /> constants, symbols, calls 이것들이 다 합쳐 expressions라고 알려져있다.

-   Section 18.4에서는, parsing을 다룬다. <br /> 이건 코드 안에 있는, 일렬의 캐릭터들을, AST로 변환하는 행동. <br /> 그리고 이 parsing을 이용해, R 문법의 몇몇 디테일들을 알아본다.

-   Section 18.5에서는, 어떻게 재귀recursive 함수를 사용해 <br />   언어를 계산하고, expressions로 계산하는 함수를 작성할 수 있는지를 보여줌. <br /> compute on the language, writing functions that compute with expressions.

-   Section 18.6에서는, 3개의 더 특화된 데이터 구조들에 대해 알아볼 것. <br /> pairlists, missing arguments, expression vectors

### Prerequisites

Chapter 17의 메타프로그래밍 개요를 보고 오자. <br /> 그래야 좀 더 동기motivation이나 기본 단어들을 알 수 있다. <br /> expressions를 캡쳐하고 계산하기 위해 rlang 패키지가 필요하고, <br />   이걸 시각화하기 위해 lobstr 패키지가 필요하다.

``` r
library(rlang)
library(lobstr)
```

------------------------------------------------------------------------

18.2 Abstract syntax trees
--------------------------

expressions는 **abstract syntax trees**(ASTs)라고도 부른다. 왜냐하면 코드 구조가 위계질서hierarchical가 있고, 나무와 같이 자연스럽게 표현할 수 있기 때문. 이 나무 구조를 이해하는 것은 expressions를 점검inspect하고 수정modify하는데 있어 중요하다.

앞으로 tree라는 걸 나무라고도 하고, 트리라고도 하고, tree라고도 하겠다. 일관성 있게 번역을 하고 싶은데, 그보다 더 중요한 건 이해가 잘 되게끔 하는거라 이렇게 하겠다. 경우에 따라 나무라고 하면, 이질감이 든다고 해야하나... 뭘 쓰든 원문은 tree라는 걸 인지하자.

### 18.2.1 Drawing

ASTs를 그리는데 있어, 몇 가지 관습convention을 소개하겠다. 중요 요소들components을 다 보여주는 함수 호출, f(x, "y", 1)를 불러보는걸로 시작해보자. 두 가지 방법으로 이 tree들을 그릴건데,

-   OmniGraffle이라는 프로그램을 통해 만든 그림으로. ![Figure 18.1](https://d33wubrfki0l68.cloudfront.net/d50d61f9b2b8d935a45fd91ea142c7c0a29d5d23/525f5/diagrams/expressions/simple.png)

-   `lobstr::ast()`를 이용해서.

``` r
lobstr::ast(f(x, "y", 1))
## o-f 
## +-x 
## +-"y" 
## \-1
```

2가지 접근법 모두 최대한 같은 관습들을 공유하고 있다.

-   트리의 잎사귀들leaves은 `f`나 `x` 같은 symbols이거나, `1`이나 `"y"` 같은 constants다. Symbols는 둥근 모서리의 보라색 네모로 그렸고, Constants는 검은색 네모로 그렸다. Strings랑 symbols는 쉽게 헷갈리기 때문에, strings는 항상 따옴표quotes로 묶여있다.

-   트리의 가지들branches은 call objects다. 이건 함수 호출function calls을 표현한다. 오렌지색 네모로 그렸다. 첫 번째 새끼child인 `f`는 호출되는 함수다. 두 번째 그리고 그 다음 새끼들children인 `x`, `"y"`, `1`은 그 함수의 인자들arguments이다.

`ast()`를 호출하면 색이 나온다. 근데 이 책에서 나타나지는 않는다. 복잡한 기술적인 문제들이 있음.

위의 예제에는, 하나의 함수 호출만을 가지고 있었기 때문에, shallow tree를 만들었다. 많은 expressions는 많은 호출들을 가지고 있기 때문에, 트리들이 multiple levels를 가진다. 예를 들어서, `f(g(1, 2), h(3, 4, i()))`의 AST를 봐보자.

![Figure 18.2](https://d33wubrfki0l68.cloudfront.net/9e269a7eb3509ae2e9f3fa9583ff2195b947cc53/d5886/diagrams/expressions/complicated.png)

``` r
lobstr::ast(f(g(1, 2), h(3, 4, i())))
## o-f 
## +-o-g 
## | +-1 
## | \-2 
## \-o-h 
##   +-3 
##   +-4 
##   \-o-i
```

다이어그램은 그냥 왼쪽에서 오른쪽으로 읽으면 되고, lobstr로 그린 것은 위에서 아래로 읽으면 된다. 트리의 뎁스depth는 함수 호출의 nesting으로 결정된다. 또한 이게 evaluation order도 결정하는데, 왜냐하면 evaluation은 보통 깊은 곳에서부터 얕은 곳으로 진행되기 때문. 하지만 꼭 그런 것만도 아닌게, lazy evaluation([Section 6.5](https://blog-for-phil.readthedocs.io/en/latest/Advanced%20R/06-Functions/#65-lazy-evaluation)) 때문.

또, `i()`를 보면, 아무런 인자들이 없는 함수 호출인데, 여전히 하나의 leaf, symbol `i`를 갖고 있는 branch라는 걸 인지하자.

### 18.2.2 Non-code components

무엇이 **abstract** syntax trees를 만드는지 궁금할 수 있다. 왜 abstract라고 불리냐면, 코드의 중요한 구조적인 디테일만 캡쳐하지, 띄어쓰기나 코멘트는 안하기 때문.

``` r
ast(
  f(x,   y) # important!
)
## o-f 
## +-x 
## \-y
```

띄어쓰기가 AST에 영향을 주는 경우가 하나 있긴 함.

``` r
lobstr::ast(y <- x)
## o-`<-` 
## +-y 
## \-x
lobstr::ast(y < -x)
## o-`<` 
## +-y 
## \-o-`-` 
##   \-x
```

### 18.2.3 Infix calls

R의 모든 호출들calls은 트리 형식form으로 쓸 수 있다. 왜냐하면 어떠한 호출도 prefix form으로 쓸 수 있기 때문.([Section 6.8.1](https://blog-for-phil.readthedocs.io/en/latest/Advanced%20R/06-Functions/#681-rewriting-to-prefix-form)) `y <- x * 10`을 다시 예로 들어보자. 여긴 어떤 함수들이 호출된걸까? `f(x, 1)` 만큼 파악하기 쉽진 않다. 왜냐하면 그 expression은 2개의 infix 호출들을 포함하고 있기 때문. `<-`와 `*`. 그래서 다음의 두 코드들은 동일한 것이다.

``` r
y <- x * 10
`<-`(y, `*`(x, 10))
```

둘 다 동일한 AST를 갖는다. ![Figure 18.3](https://d33wubrfki0l68.cloudfront.net/e32631051094207bc971e4352744db7ba6f8aac1/6f551/diagrams/expressions/prefix.png)

``` r
lobstr::ast(y <- x * 10)
## o-`<-` 
## +-y 
## \-o-`*` 
##   +-x 
##   \-10
```

AST 간에는 정말 아무런 차이가 없으며, 만약에 prefix 호출로 expression을 생성하더라도, R은 여전히 infix form으로 print할 것이다.

``` r
expr(`<-`(y, `*`(x, 10)))
## y <- x * 10
```

infix 연산자operators가 적용되는 순서는, 연산자 우선순위operator precedence라고 불리는 규칙들에 의해 결정govern되며, Section 18.4.1에서 `lobstr::ast()`를 이용해 탐구해볼 것이다.

### 18.2.4 Exercises

------------------------------------------------------------------------

18.3 Expressions
----------------

------------------------------------------------------------------------

18.4 Parsing and grammar
------------------------

------------------------------------------------------------------------

18.5 Walking AST with recursive functions
-----------------------------------------

------------------------------------------------------------------------

18.6 Specialised data structures
--------------------------------
