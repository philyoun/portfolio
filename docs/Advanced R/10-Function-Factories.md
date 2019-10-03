10 Function Factories
=====================

Function Factory는 functions를 만드는 function이다. <br /> 매우 간단한 예를 봐보자. <br /> `power1()`이라는 function factory를 이용해서, `square()`, `cube()`라는 2개의 child 함수들을 만들어보자. <br />

``` r
power1 <- function(exp){
  function(x){
    x ^ exp
  }
}

square <- power1(2)
cube <- power1(3)
```

여기서 `square()`이랑 `cube()`는 manufactured functions라고 부른다. <br /> 이 용어는 사람과 소통하기 쉽게 하기위해 만들어진거지, R의 관점에서는 다른 함수들과 다를게 없다. <br />

``` r
square(3)
```

    ## [1] 9

``` r
cube(3)
```

    ## [1] 27

Function factories를 가능케하는 개별적인 components들에 대해선 이미 다 배웠다. <br /> - Section 6.2.3에서, R's first-class functions에 대해 배웠다. <br /> R에서는 `<-`를 이용해서, 오브젝트에 이름을 bind하는 것과 같은 방식으로, 함수function에 이름을 bind할 수 있다.

-   Section 7.4.2에서, 함수가 만들어질 때, 함수를 갖고 있는 env를 캡쳐(enclose)한다는 걸 배웠다. <br /> In Section 7.4.2, you learned that a function captures(encloses) the environment / in which it is created.

-   Section 7.4.4에서, 함수가 실행될 때마다, 새로운 execution env를 만든다는 것을 배웠다. <br /> 이 env는 보통 ephemeral하지만, 여기서는 manufactured function의 enclosing env가 된다.

이 Chapter에서는, 위 3개의 불분명한 조합이 function factory로 이어진다는 것을 배울 것이다. <br /> 그리고 function factory의 사용 예제들을 visualisation과 statistics에서 볼 것이다.

3개의 주요 functional programming tools(functional, function factories and function operators)중에서, function factories를 제일 덜 쓴다. <br /> 보통, 전체 코드의 복잡성을 줄여주지는 않는다. 하지만 대신에, 좀 더 소화하기 쉬운 chunks로 복잡성을 줄여준다. <br /> Generally, they don't tend to reduce overall code complexity but instead partition complexity into more easily digested chunks. <br /> 또한 Function factories는, 11장에서 배울, 매우 유용한 function operators의 중요한 구성 요소다.
