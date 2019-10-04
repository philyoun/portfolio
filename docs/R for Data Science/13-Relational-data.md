13 Relational data
==================

13.1 Introduction
-----------------

데이터 분석을 하는데 있어, 하나만의 table을 쓰는 경우는 거의 없다. <br /> 보통 여러 개의 테이블들이 있고, 니가 궁금한 것에 대해 대답을 하기 위해선, 이것들을 잘 조합combine해야됨. <br /> 종합해서, 이 multiple tables of data를 relational data라고 부른다. <br /> 그냥 개별적인 데이터셋들이 아니라, relations가 중요하기 때문이다.

Relations라는건, 항상 2개의 테이블에서 정의되는거다. <br /> 모든 relations들은 이 간단한 아이디어에서 만들어진 것. <br /> 3개 이상의 테이블에서의 relations라는건 항상 2개씩의 relations의 특성임. 확장의 개념으로 생각하자. <br /> the relations of three or more tables / are always a property of the relations between each pair.

Relational data에 대해 다루기 위해서는, pairs of tables에 사용되는 verbs를 알아야됨. <br /> 3가지의 families of verbs가 있음.

-   Mutating joins: 하나의 데이터 프레임에 새로운 변수들을 추가하는 것. <br />     어떤 새로운 변수? 다른 데이터 프레임에서, 매칭이 되는 관측치에 대한 변수 <br />     which add new variables to one data frame / from matching observations in another. <br />
-   Filtering joins: 하나의 데이터 프레임에서 관측치를 필터링 하는 것임. <br />     어떤 기준으로? 다른 데이터 프레임에, 그 관측치에 대한 정보가 있냐없냐에 따라 <br />     which filter observations from one data frame / based on whether or not they match an observation in the other table.
-   Set Operations: 관측치를 set elements였던 것처럼 다루는 것. <br />     which treat observations as if they were set elements.

Relational data를 찾을 수 있는 가장 흔한 장소는 relational database management system(RDBMS)이다. <br /> 거의 모든 현대 데이터베이스들을 관통하는 단어. <br /> 만약에 데이터베이스를 다루어본 적이 있다면, 아마 분명히 SQL을 썼을 것이다. <br /> 그렇다면, 이 챕터에서 비슷한 개념을 찾을 수 있을 것이다. 물론 표현은 dplyr에선 좀 다르겠지만. <br /> 일반적으로, dplyr은 SQL보다 좀 더 사용하기 쉽다. 왜냐하면 데이터 분석에 특화되어있기 때문에. <br /> 보통의 데이터 분석 작업을 쉽게 만들어 주는데, 데이터 분석에 잘 이용되지 않는 다른 것들은 하기가 좀 힘들어지긴 한다. <br /> it makes common data analysis operations easier, at the expense of making it more difficult to do other things to do other things that aren't commonly needed for data analysis.

### 13.1.1 Prerequisites

dplyr에서 two-table verbs를 사용함으로써, `nycflights13`의 relational data를 다루어보자. <br />

``` r
library(tidyverse)
```

    ## Warning: package 'tidyverse' was built under R version 3.5.2

    ## -- Attaching packages ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- tidyverse 1.2.1 --

    ## √ ggplot2 3.1.0     √ purrr   0.2.5
    ## √ tibble  2.1.3     √ dplyr   0.7.8
    ## √ tidyr   0.8.2     √ stringr 1.3.1
    ## √ readr   1.3.1     √ forcats 0.4.0

    ## Warning: package 'ggplot2' was built under R version 3.5.2

    ## Warning: package 'tibble' was built under R version 3.5.3

    ## Warning: package 'readr' was built under R version 3.5.2

    ## Warning: package 'forcats' was built under R version 3.5.2

    ## -- Conflicts ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(nycflights13)
```

    ## Warning: package 'nycflights13' was built under R version 3.5.2

13.2 nycflights13
-----------------

relational data를 배우기 위해서 nycflights13 패키지를 사용할 것이다. <br /> 이 패키지는 4개의 tibbles를 갖고 있다. <br /> 이 tibbles는 data transformation을 할 때 썼던, `flights`라는 테이블과 연관이 있는.

`airlines`: 항공사 이름을 단축된 코드로 볼 수 있도록 해준다.

``` r
airlines
```

    ## # A tibble: 16 x 2
    ##    carrier name                       
    ##    <chr>   <chr>                      
    ##  1 9E      Endeavor Air Inc.          
    ##  2 AA      American Airlines Inc.     
    ##  3 AS      Alaska Airlines Inc.       
    ##  4 B6      JetBlue Airways            
    ##  5 DL      Delta Air Lines Inc.       
    ##  6 EV      ExpressJet Airlines Inc.   
    ##  7 F9      Frontier Airlines Inc.     
    ##  8 FL      AirTran Airways Corporation
    ##  9 HA      Hawaiian Airlines Inc.     
    ## 10 MQ      Envoy Air                  
    ## 11 OO      SkyWest Airlines Inc.      
    ## 12 UA      United Air Lines Inc.      
    ## 13 US      US Airways Inc.            
    ## 14 VX      Virgin America             
    ## 15 WN      Southwest Airlines Co.     
    ## 16 YV      Mesa Airlines Inc.

`airports`: 각 공항에 대한 정보를 준다. faa airport code로 확인할 수 있는.

``` r
airports
```

    ## # A tibble: 1,458 x 8
    ##    faa   name                    lat    lon   alt    tz dst   tzone        
    ##    <chr> <chr>                 <dbl>  <dbl> <int> <dbl> <chr> <chr>        
    ##  1 04G   Lansdowne Airport      41.1  -80.6  1044    -5 A     America/New_~
    ##  2 06A   Moton Field Municipa~  32.5  -85.7   264    -6 A     America/Chic~
    ##  3 06C   Schaumburg Regional    42.0  -88.1   801    -6 A     America/Chic~
    ##  4 06N   Randall Airport        41.4  -74.4   523    -5 A     America/New_~
    ##  5 09J   Jekyll Island Airport  31.1  -81.4    11    -5 A     America/New_~
    ##  6 0A9   Elizabethton Municip~  36.4  -82.2  1593    -5 A     America/New_~
    ##  7 0G6   Williams County Airp~  41.5  -84.5   730    -5 A     America/New_~
    ##  8 0G7   Finger Lakes Regiona~  42.9  -76.8   492    -5 A     America/New_~
    ##  9 0P2   Shoestring Aviation ~  39.8  -76.6  1000    -5 U     America/New_~
    ## 10 0S9   Jefferson County Intl  48.1 -123.    108    -8 A     America/Los_~
    ## # ... with 1,448 more rows

`planes`: 각 비행기 정보를 준다. tailnum로 확인할 수 있는.

``` r
planes
```

    ## # A tibble: 3,322 x 9
    ##    tailnum  year type       manufacturer  model  engines seats speed engine
    ##    <chr>   <int> <chr>      <chr>         <chr>    <int> <int> <int> <chr> 
    ##  1 N10156   2004 Fixed win~ EMBRAER       EMB-1~       2    55    NA Turbo~
    ##  2 N102UW   1998 Fixed win~ AIRBUS INDUS~ A320-~       2   182    NA Turbo~
    ##  3 N103US   1999 Fixed win~ AIRBUS INDUS~ A320-~       2   182    NA Turbo~
    ##  4 N104UW   1999 Fixed win~ AIRBUS INDUS~ A320-~       2   182    NA Turbo~
    ##  5 N10575   2002 Fixed win~ EMBRAER       EMB-1~       2    55    NA Turbo~
    ##  6 N105UW   1999 Fixed win~ AIRBUS INDUS~ A320-~       2   182    NA Turbo~
    ##  7 N107US   1999 Fixed win~ AIRBUS INDUS~ A320-~       2   182    NA Turbo~
    ##  8 N108UW   1999 Fixed win~ AIRBUS INDUS~ A320-~       2   182    NA Turbo~
    ##  9 N109UW   1999 Fixed win~ AIRBUS INDUS~ A320-~       2   182    NA Turbo~
    ## 10 N110UW   1999 Fixed win~ AIRBUS INDUS~ A320-~       2   182    NA Turbo~
    ## # ... with 3,312 more rows

`weather`: NYC 공항에서의 시간별 날씨를 준다.

``` r
weather
```

    ## # A tibble: 26,115 x 15
    ##    origin  year month   day  hour  temp  dewp humid wind_dir wind_speed
    ##    <chr>  <dbl> <dbl> <int> <int> <dbl> <dbl> <dbl>    <dbl>      <dbl>
    ##  1 EWR     2013     1     1     1  39.0  26.1  59.4      270      10.4 
    ##  2 EWR     2013     1     1     2  39.0  27.0  61.6      250       8.06
    ##  3 EWR     2013     1     1     3  39.0  28.0  64.4      240      11.5 
    ##  4 EWR     2013     1     1     4  39.9  28.0  62.2      250      12.7 
    ##  5 EWR     2013     1     1     5  39.0  28.0  64.4      260      12.7 
    ##  6 EWR     2013     1     1     6  37.9  28.0  67.2      240      11.5 
    ##  7 EWR     2013     1     1     7  39.0  28.0  64.4      240      15.0 
    ##  8 EWR     2013     1     1     8  39.9  28.0  62.2      250      10.4 
    ##  9 EWR     2013     1     1     9  39.9  28.0  62.2      260      15.0 
    ## 10 EWR     2013     1     1    10  41    28.0  59.6      260      13.8 
    ## # ... with 26,105 more rows, and 5 more variables: wind_gust <dbl>,
    ## #   precip <dbl>, pressure <dbl>, visib <dbl>, time_hour <dttm>

다이어그램으로 이 테이블들 간의 관계를 표시했는데, ![그림1](https://d33wubrfki0l68.cloudfront.net/245292d1ea724f6c3fd8a92063dcd7bfb9758d02/5751b/diagrams/relational-nycflights.png)

다이어그램은 좀 복잡하긴한데, 필드에 나가서 보게 될 것에 비하면 간단한 편이다. <br /> 이러한 다이어그램을 이해하는 것의 key는, 각 relation은 한 쌍의 테이블만을 고려한다는 것. <br /> The key to understanding diagrams like this / is to remember / each relation always concerns a pair of tables. <br /> 다 이해할 필요는 없고, 테이블 간의 (니가 관심있는)chain of relations만 이해해라.

이 nycflights13에 관해선, <br /> `flights`는 `planes`와 하나의 변수인, `tailnum`을 통해 연결된다. <br /> `flights`는 `airlines`와 `carrier` 변수를 통해 연결된다. <br /> `flights`는 `airports`와 두 가지 방법, `origin`과 `dest` 변수들을 통해 연결된다. <br /> `flights`는 `weather`와, `origin`(출발지), `year`, `month`, `day` 그리고 `hour`를 통해 연결된다.

13.3 Keys
---------

2개의 tables를 연결시켜주는 변수들을, **keys**라고 부른다. <br /> keys는 관측치를 unique하게 identify해주는 변수(혹은 변수들). <br /> 간단한 케이스들에서는, 하나의 변수가 관측치를 identify하기에 충분하다.
