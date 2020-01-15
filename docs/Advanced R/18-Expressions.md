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

이 장에서는, expressions 밑에 있는 데이터 구조에 대해 알아볼 것. <br /> 이 것에 대해 마스터하게 되면, <br />   캡쳐된 코드를 검사inspect하고 수정modify할 수 있고, 코드로 코드를 만들 수 있다. <br /> generate code with code.

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

### 18.2.1 Drawing

### 18.2.2 Non-code components

### 18.2.3 Infix calls

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
