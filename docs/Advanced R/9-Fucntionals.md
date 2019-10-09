9 Functionals
=============

9.1 Introduction
----------------

코드가 더 믿을만하려면, 더 투명해야 한다. <br /> 특히나, nested conditions랑 loop는 의심을 갖고 보아야 한다. <br /> 복잡한 control flows는 프로그래머를 혼란스럽게 한다. <br /> 복잡한 코드는 종종 버그를 숨기고 있다. <br /> To become significantly more reliable, code must become more transparent. <br /> In particular, nested conditions and loops must be viewed with great suspicion. <br /> Complicated control flows confuse programmers. <br /> Messy code often hides bugs.

*자 그래서 functional 무엇이냐? function과는 어떻게 다른 것인가?* <br /> functional은, input을 함수로 받고, output을 벡터로 return하는 함수. <br /> A functional is a function / that takes a function as an input / and returns a vector as output.

간단한 Functional을 소개해보자. <br /> 1000개의 random uniform numbers를 input으로 넣은 함수를 호출call하는 functional.

``` r
randomise <- function(f) f(runif(1e3))
randomise(mean)
## [1] 0.4862418
randomise(mean)
## [1] 0.4842393
randomise(sum)
## [1] 503.4041
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
```

    ## [[1]]
    ## [1] 3
    ## 
    ## [[2]]
    ## [1] 6
    ## 
    ## [[3]]
    ## [1] 9

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
왜 이 함수를 `map()`이라고 할까? <br /> 지도가 아니라 수학적인 map, "given set의 각 element를, 두 번째 set의 하나 혹은 이상의 elements로 association시키는 연산operation"이라는 뜻에서 map()이라고 부르는 것이다. <br /> (그리고 "Map"이라는 단어는 스펠링이 짧아서 기초적인 building block이 되기에 좋다.)
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

실제 `purrr::map()`은 몇 가지 차이점이 있다. <br /> C언어로 써서 성능을 조금이라도 더 짜냈고, 이름을 보존하고, Section 9.2.2에서 배울 몇 가지 지름길들을 지원한다.

<p class="comment">
`map()`과 동등한 base 함수는 `lapply()`다. <br /> 유일한 차이는 `lapply()`는 밑에서 배우게 될 helpers를 지원하지 않는다는 것이다. <br /> 그래서 만약에 purrr에서 `map()`만 쓸 것이라면, 추가적인 의존성additional dependency를 스킵하고 `lapply()`를 쓰면 된다.
</p>
### 9.2.1 Producing atomic vectors
