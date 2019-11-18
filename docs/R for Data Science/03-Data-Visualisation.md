3 Data Visualisation
================

3 Data Visualisation
====================

3.1 Introduction
----------------

> "간단한 그래프가 다른 어떤 도구보다 데이터 애널리스트에게 많은 정보를 전달해준다." - 존 튜키 <br /> “The simple graph has brought more information to the data analyst’s mind than any other device.” — John Tukey

이 chapter에서는 ggplot2를 이용해, 데이터를 어떻게 visualise할지 알려준다. <br /> R에는 그래프를 만들어주는 몇 가지 시스템들이 있지만, ggplot2는 가장 우아하고 다재다능한 것이다.

ggplot2는 **그래픽의 문법**을 구현implement했다. <br /> 그래프를 만들고 묘사하는데 있어, 일관적인 시스템을 정립했음. <br /> ggplot2와 함께라면, 하나의 시스템만을 배워서 더 빠르게, 그리고 많은 곳에 응용할 수 있다.

시작하기 전에, ggplot2의 이론적인 근거들을 더 배우고 싶다면, <br /> [The Layered Grammar of Graphics](http://vita.had.co.nz/papers/layered-grammar.pdf) 이걸 읽기를 추천한다.

### 3.1.1 Prerequisites

이 chapter에서는, tidyverse의 핵심 멤버 중 하나인, ggplot2에 집중할 것이다. <br /> 데이터셋을 접근하고, 필요한 함수를 사용하기 위해서, tidyverse를 로드하자.

``` r
library(tidyverse)
```

    ## -- Attaching packages -------------------------------------------------------------------------- tidyverse 1.2.1 --

    ## √ ggplot2 3.1.0     √ purrr   0.2.5
    ## √ tibble  2.1.3     √ dplyr   0.7.8
    ## √ tidyr   0.8.2     √ stringr 1.3.1
    ## √ readr   1.3.1     √ forcats 0.4.0

    ## -- Conflicts ----------------------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

이 코드 한줄로 핵심 tidyverse를 로드할 수 있다. <br /> 거의 모든 데이터 분석에서 사용할 패키지들이다. (실제로 없으면 안 되는 수준이다.) <br /> base R 혹은 로드한 다른 패키지들의 함수와, 이름이 같아서 충돌하는지도 알려준다.

만약에 이 코드를 실행했는데, "there is no package called 'tidyverse'"라는 에러 메시지가 나오면, <br /> 먼저 설치를 해야한다. 그리고 나서 `library()`를 다시 실행해야 한다.

``` r
install.packages("tidyverse")
library(tidyverse)
```

패키지 설치는 처음 한 번만 하면 되는데, 패키지를 로드 하는 것은 세션을 시작할때마다 해줘야 한다. <br /> (독자가 beginner라고 가정하고 설명을 해놓은듯..)

어떤 함수가 어디에서 왔는지를 명시해주고 싶다면, <br />     `package::function()`이라는 특별한 형식으로 사용해줘야 한다. <br /> 예를 들어, `ggplot2::ggplot()`라고 하면, <br />     우리가 ggplot2라는 패키지에서 `ggplot()`이라는 함수를 사용하고 있다는 걸 뜻한다.