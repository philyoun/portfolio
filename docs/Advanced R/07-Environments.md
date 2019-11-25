7 Environments
==============

``` r
library(rlang)
```

7.4 Special Environments
------------------------

대부분의 env들은 너가 만드는게 아니라, R이 만든다. <br /> 이 섹션에서는 package env부터 시작해서 중요한 env들에 대해 배운다. <br /> 그리고 함수function가 만들어질 때 함수에 bound되는, function env에 대해 배우고, <br /> 함수가 호출될 때마다 생겼다가 없어지는, ephemeral execution env에 대해 배운다. <br /> 1. package env, 2. function env, 3. execution env

마지막으로, package와 function env가 namespaces를 지원support하기 위해 어떻게 interact하는지, <br />     namespaces는 user가 어떤 다른 패키지들을 load했던 간에 패키지가 항상 같은 방식으로 행동하게끔 한다.

### 7.4.1 Package env와 search path

library()나 require()를 통해서 attach한 패키지들은 global env의 parents가 된다.<br /> immediate parent는 가장 최근에 attach한 패키지, 그리고 그 바로 위 parent는 2번 째로 최근에 attach한 패키지..이런 식 ![그림1](https://d33wubrfki0l68.cloudfront.net/038b2da4f5db1d2a8acaf4ee1e7d08d04ab36ebc/ac22a/diagrams/environments/search-path.png)

이런 식으로 parents를 거슬러 올라가다보면, 패키지들이 attach된 순서를 볼 수 있다. 이걸 **search path**라고 부르는데, <br />     이 env들에 있는 모든 오브젝트들을 top-level interactive workspace에서부터 찾을 수 있기 때문.<br />     because all objects in these environments / can be found from the top-level interactive workspace.

이 env들의 이름들을, base::search() 혹은 env 그 자체들을 rlang::search\_envs()를 통해서 확인할 수 있다.

``` r
search()
```

    ##  [1] ".GlobalEnv"        "package:rlang"     "package:stats"    
    ##  [4] "package:graphics"  "package:grDevices" "package:utils"    
    ##  [7] "package:datasets"  "package:methods"   "Autoloads"        
    ## [10] "package:base"

``` r
search_envs()
```

    ##  [[1]] $ <env: global>
    ##  [[2]] $ <env: package:rlang>
    ##  [[3]] $ <env: package:stats>
    ##  [[4]] $ <env: package:graphics>
    ##  [[5]] $ <env: package:grDevices>
    ##  [[6]] $ <env: package:utils>
    ##  [[7]] $ <env: package:datasets>
    ##  [[8]] $ <env: package:methods>
    ##  [[9]] $ <env: Autoloads>
    ## [[10]] $ <env: package:base>

search path의 마지막 2개 env들은 항상 같다.

-   Autoloads env는 delayed bindings를 이용해서 메모리를 save한다. 어떻게? 패키지 오브젝트들을 필요할 때만 로딩하는 방식으로.

-   package:base 혹은 그냥 base라고 하는 base env는, base 패키지의 env다. <br /> 이건 다른 패키지들의 로딩을 시동할 수 있어야하기 때문에 특별하다. <br /> It is special because / it has to be able to bootstrap / the loading of all other packages. <br /> 이 base env는, base\_env()를 통해서 직접적으로 access할 수 있다.

library()를 통해서 다른 패키지를 로딩할 때, global env의 parent env가 다음과 같이 변한다. pkg:d가 추가된 것. ![그림2](https://d33wubrfki0l68.cloudfront.net/7c87a5711e92f0269cead3e59fc1e1e45f3667e9/0290f/diagrams/environments/search-path-2.png)

### 7.4.2 The function environment

함수function는, 그게 만들어질 때, current env를 bind한다. <br /> A function binds the current environment when it is created. <br /> 이걸 **function env**라고 부르는데, lexical scoping에 사용된다. <br /> 컴퓨터 언어에서는, 자신의 env를 캡쳐하는 함수들을 **closures**라고 부르는데, R에서는 함수가 자기자신의 env를 항상 bind, 그래서 R's documentation에서는 function이랑 closures랑 혼용해서 사용되는 것이다.

이 function env는 `fn_env()`를 통해서 얻을 수 있다.

``` r
y <- 1
f <- function(x) x + y
fn_env(f)
```

    ## <environment: R_GlobalEnv>

<details> <summary>base R</summary> 함수 `f`의 env를 access하고 싶다면 environment(f)를 사용해라. </details>

다이어그램에서는, 함수를 다음과 같이 env를 bind하고 있는 '반원이 붙은 네모'로 그릴 것이다. <br /> In diagrams, I'll draw a function as a rectangle with a rounded end that binds an environment. ![그림3](https://d33wubrfki0l68.cloudfront.net/cd8208b418ecbaf6ace1b6453b93fdf628173e01/68d59/diagrams/environments/binding.png)

이 경우에 `f()`는, `f`라는 이름을 함수에 bind하는 env를, bind한다. <br /> In this case, `f()` binds the environment that binds the name `f` to the function.

하지만 항상 이런건 아니다. 다음의 예를 보자. <br /> `g()`는 global env를 binds하고 있고, `g`는 새로운 env `e`에 bound되어 있다. <br /> ![그림4](https://d33wubrfki0l68.cloudfront.net/cd32bb2bc59dcfa579b0415ebac271f24c6a85fd/cde86/diagrams/environments/binding-2.png)

binding하는 것과 bound되는 것은 미묘하지만 분명한 차이가 있다. <br /> 전자는 우리가 `g`를 어떻게 찾느냐 하는 것이고, 후자는 `g`가 그것의 변수들을 어떻게 찾느냐 하는 것임.

\*함수 `g`는 global env에서 우리가 찾는 것이고, `g`의 변수들이 있다면 e라는 env안에서 찾는 것.

### 7.4.3 Namespaces

위의 다이어그램을 보면, 어떤 패키지들을 로드시키냐에 따라 패키지의 parent env가 달라진다. <br /> 그럼 걱정이 된다. 패키지들이 다른 순서로 로드되어 있으면 패키지가 다른 함수를 찾는게 아닐까? <br /> **namespaces**의 목표는 이런 일이 생기지 않도록 하는 것이다. 그리고 어떤 패키지들이 attach되었던간에 같은 방식으로 작동하도록.

예를 들어서, sd()를 봐보자.

``` r
sd
```

    ## function (x, na.rm = FALSE) 
    ## sqrt(var(if (is.vector(x) || is.factor(x)) x else as.double(x), 
    ##     na.rm = na.rm))
    ## <bytecode: 0x0000000015aee308>
    ## <environment: namespace:stats>

sd()는 var()의 관점으로 정의되어 있다. sd() is defined in terms of var(). <br /> 그래서 만약에 global env에서나 혹은 다른 attach된 패키지 안의, var()이라고 불리는 어떤 함수에 의해, <br />     sd()의 결과가 영향받지 않을까 걱정할 수 있다. <br /> so you might worry that the result of sd() / would be affected / by any function called var() <br />     either in the global env, or in one of the other attached packages. <details> <summary>예를 들어,</summary>

``` r
sd(1:2)
```

    ## [1] 0.7071068

이 값을, `var`을 새롭게 정의해놓는다면 바뀌지 않을까? 하고 걱정할 수 있음.

``` r
var <- function(x) x
var(1)
```

    ## [1] 1

이제 `var`이라는 함수는 받은 그대로를 출력하는 함수

그래도 여전히 `sd()`는 바뀌지 않는다.

``` r
sd(1:2)
```

    ## [1] 0.7071068

</details> <br /> <br /> <br /> <br />

R은 앞서 설명한 함수 대(對) binding env를 사용해서, 이러한 문제를 피한다. <br /> R avoids this problem by taking advantage of the function versus binding env described above.

패키지에 있는 모든 함수들은, 한 쌍의 env와 결합associate되어 있다. <br /> package env와 namespace env.

1.  package env는 패키지에 대한 external interface. <br /> The package env is th external interface to the package. <br /> R user가 어떻게 attach된 패키지에서, 혹은 ::를 이용해서 함수를 찾는지. <br /> It's how you, the R user, find a function in an attached package or with ::. <br /> package env의 parents는 search path에 의해 결정된다. <br />     즉, 패키지가 어떤 순서로 attach되었는지에 따라, package env의 parents가 결정된다.

2.  namespace env는 패키지에 대한 internal interface. <br /> package env가, 우리가 어떻게 함수를 찾는지를 컨트롤한다면, <br /> namespace env는 어떻게 그 함수가 그 안의 변수를 찾는지를 컨트롤.

정리해보면, package env는 우리가 함수를 찾을 때 쓰는 것이고, namespace env는 함수가 그 안의 변수를 찾을 때 쓰는 것이고. <br /> 근데 그렇다면, 어떤 함수가 다른 함수들을 찾을 수는 없는 것 아닌가? <br /> 내가 함수를 찾을 수는 있고, 함수가 그 안의 변수들을 찾을 수는 있는데, <br /> 함수가 다른 함수들을 찾을 수는 없잖아?

그래서 <br /> package env에 있는 모든 binding들은 namespace env에도 있다. <br /> 이렇게 모든 함수들이 패키지 안의 다른 함수들을 사용할 수 있는 것. <br /> 하지만 몇몇 binding들은 namespace env에서만 출현occur한다. <br /> 이것들은 internal 혹은 non-exported 오브젝트들이라고 알려져있는데, 이것들 때문에 <br /> user가 internal implementation을 감출 수 있는 것hide이다.

이걸 그림으로 나타내보면, ![그림5](https://d33wubrfki0l68.cloudfront.net/d4fc3ef4f21f2cb0cd065933cba3005cc4b0ea3c/4c4b3/diagrams/environments/namespace-bind.png)

다음으로, 모든 namespace env는 같은 set의 ancestors를 갖는다. <br /> - 각 namespace는 imports env를 갖는다. <br /> 패키지에 이용된 모든 함수들에 대한 bindings를 갖고 있는 env. <br /> imports env는 패키지 개발자에 의해, NAMESPACE 파일로 컨트롤된다.

-   모든 base 함수들을 explicit하게 importing하는 것은 귀찮다. <br /> 그래서 imports env의 parent는 base namespace. <br /> base namespace는 base env와 같은 bindings를 갖고 있는데, 다른 parent를 갖는다.

-   base namespace의 parent는 global env다. 이 말인즉슨, binding이 imports env에서 정의되지 않았다면, 패키지는 평소와 같은 방법으로 찾아볼 거라는 것. <br /> This means that if binding isn't defined in the imports env / the package will look for it in the usual way. <br /> 이건 보통 나쁜 방법이기 때문에, `R CMD check`가 자동적으로 이러한 코드에 대해서 경고한다. <br /> S3 메소드 디스패치가 작동하는 방법 때문에 필요했던 역사적인 이유가 있다.

위 3가지 논의를 그림으로 정리해보면, ![그림6](https://d33wubrfki0l68.cloudfront.net/3184a9827ac2c26c60f65680157241819f55e754/542c2/diagrams/environments/namespace-env.png)

그리고 이걸 전부다 종합해서, `sd()`의 예를 설명해보면, ![그림7](https://d33wubrfki0l68.cloudfront.net/fbbfd3b49bdbd3ca1913043233d48454ec27f14e/ae75a/diagrams/environments/namespace.png)

그래서 `sd()`가 `var`의 값을 찾아볼 때, 항상 패키지 user가 아닌, 패키지 developer가 결정해놓은 env의 sequence들을 찾아가게 된다. <br /> 그래서 package 코드는 user가 어떤 패키지들을 attach시켜놨던간에 항상 같은 방식으로 작동하도록 보장받는 것이다.

위 그림을 보면 알다시피 패키지와 namespace env간에 직접적이 링크direct link는 없다. function env를 통해서만 링크는 정의된다.

### 7.4.4 Execution environments

마지막으로 다뤄야할 중요한 주제는 execution env다. <br /> 다음의 함수를 처음 실행시켜보면 무엇이 나올까? 2번째로 실행시켜보면?

``` r
g <- function(x) {
  if (!env_has(current_env(), "a")) {
    message("Defining a")
    a <- 1
  } else {
    a <- a + 1
  }
  a
}
```

계속 읽기전에 한 번 생각해보자.

``` r
g(10)
```

    ## Defining a

    ## [1] 1

``` r
g(10)
```

    ## Defining a

    ## [1] 1

이 함수는 같은 값을 계속 return하는데, Section 6.4.3에서 다루었던 fresh start principle 때문이다. <br /> 함수가 호출될 때마다 host execution에 새로운 env가 생긴다. <br /> 이건 execution env라고 부르고, 이것의 parent는 function env다.

좀 더 간단한 예와 함께 이 과정을 설명해보자. <br /> execution env는 function env를 통해 찾을 수 있다.

``` r
h <- function(x) {
  a <- 2
  x + a
}

y <- h(1)
```

![그림8](https://d33wubrfki0l68.cloudfront.net/862b3606a4a218cc98739b224521b649eeac6082/5d3e9/diagrams/environments/execution.png)

우리가 `y <- h(1)`이라고 함수를 호출하면, 1에서처럼, execution env가 생겨서 `x`에다가 1을 assign. <br /> 그리고 2에서처럼, 이 execution env안에서 `a`에다가 2를 assign. <br /> 그리고나서 3에서처럼, execution env는 사라지고, y에다가 3을 return하면서 함수가 complete.

그림에서, execution env의 parent가 function env라는 것을 확인할 수 있다.

execution env는 보통 ephemeral하다. 쓰고나면 없어진다. <br /> 함수가 완료되고 나면, env는 garbage collected된다. <br /> 몇 가지 방법으로 이걸 더 오래남께끔 할 수는 있다. <br /> 첫 번째는 explicit하게 return하는 것.

``` r
h2 <- function(x) {
  a <- x * 2
  current_env()
}

e <- h2(x = 10)
env_print(e)
```

    ## <environment: 00000000186F6870>
    ## parent: <environment: global>
    ## bindings:
    ##  * a: <dbl>
    ##  * x: <dbl>

``` r
fn_env(h2)
```

    ## <environment: R_GlobalEnv>

여기서도 `h2`의 execution env의 parent가 global env라는 걸 볼 수 있다. function env가 global env이기 때문.

두 번째 방법은 함수같이, env가 binding된 object를 return하도록 하는 것. <br /> Another way to capture it is to return an object with a binding to that environment, like a function. <br /> 다음의 예는 function factory를 사용해서 이 아이디어를 illustrate한다. <br /> `plus()`라는 function factory를 이용해서, `plus_one()`이라는 함수를 만들어볼 것임.

``` r
plus <- function(x) {
  function(y) x + y
}

plus_one <- plus(1)
plus_one
```

    ## function(y) x + y
    ## <environment: 0x0000000018f5b080>

다이어그램을 보면, `plus_one()`의 enclosing env가 `plus()`의 execution env라서 조금 복잡하다. ![그림9](https://d33wubrfki0l68.cloudfront.net/853b74c3293fae253c978b73c55f3d0531d746c5/6ffd5/diagrams/environments/closure.png)

우리가 `plus_one()`을 호출하면 무슨 일이 일어나는지? <br /> plus\_one()의 execution env는, 캡쳐된 plus()의 execution env를 parent로 가질 것이다. <br /> What happens when we call plus\_one()? <br /> Its execution environment will have / the captured execution env of plus() as its parent. <br /> 그래서 plus()의 execution env가 더 오래 남아있다. ![그림10](https://d33wubrfki0l68.cloudfront.net/66676485e6a22c807c19b0c54c8fda6bd1292531/3526e/diagrams/environments/closure-call.png)

function factory에 대해서는 Section 10.2에서 자세하게 배운다.