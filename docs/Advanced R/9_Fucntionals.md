9 Functionals
=============

9.1 Introduction
----------------

코드가 더 믿을만하려면, 더 투명해야 한다. 특히나, nested conditions랑 loop는 의심을 갖고 보아야 한다. 복잡한 control flows는 프로그래머를 혼란스럽게 한다. 복잡한 코드는 종종 버그를 숨기고 있다. To become significantly more reliable, code must become more transparent. In particular, nested conditions and loops must be viewed with great suspicion. Complicated control flows confuse programmers. Messy code often hides bugs.

*자 그래서 functional 무엇이냐? function과는 어떻게 다른 것인가?* functional은, input을 함수로 받고, output을 벡터로 return하는 함수. A functional is a function / that takes a function as an input / and returns a vector as output.

간단한 Functional을 소개해보자. 1000개의 random uniform numbers를 input으로 넣은 함수를 호출call하는 functional.

``` r
knitr::opts_chunk$set(warning = FALSE)
randomise <- function(f) f(runif(1e3))
randomise(mean)
```

    ## [1] 0.4981153

``` r
randomise(mean)
```

    ## [1] 0.5048043

``` r
randomise(sum)
```

    ## [1] 490.3197
