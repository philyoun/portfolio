Analyst Internship Test
=======================

이번에 스타트업 analyst internship에 지원하게 되었다. <br /> 사전과제를 받았고, 근 이틀을 온전히 여기에 쏟아부었다. <br /> 그냥 제출하고 끝내기엔 아쉬우니 이렇게 시간 조금만 투자해서 남겨두자. <br />

<style>
p.comment {
background-color: #DBDBDB;
padding: 10px;
border: 1px solid black;
margin-left: 25px;
border-radius: 5px;
}

</style>
문제
----

<p class="comment">
본 스프레드시트는 가상의 공연별 티켓 판매량 데이터입니다. <br /> 1) 아래 설명을 참고하여 'ticket\_sales'를 예측 할 수 있는 'predictive model'을 만들어주세요. <br /> 2) feature engineering 및 modeling 과정을 설명해주세요. 자유로운 방식으로 전달해주시면 됩니다. <br /> 3) 결과물을 송부하실 때는 아래 내용을 반드시 포함해주세요: feature engineering 및 modeling의 전체 코드, 과정 및 평가 방식에 대한 설명 <br /> "주요 식별 데이터는 마스킹되었고, 수치형 데이터의 경우 암호화 되었습니다. 분석에는 영향이 없으므로 별도의 복호없이 사용하시면 됩니다." <br /> <strong>concert\_list</strong> <br /> artist 아티스트 <br /> continent 공연 대륙 <br /> city 공연 도시 <br /> closing\_date "예측 분석 마감일 (해당 공연 예측에 사용되는 데이터는 언제나 마감일 기준 과거의 데이터여야 합니다.) <br /> \* 예) closing\_date = '2019-09-09'일 경우, 2019년 9월 9일 이전의 데이터를 사용하여 예측 분석해야함" <br /> make 공연에 대한 MMT의 make 카운트 <br /> population 공연 해당 국가의 특정 연령층 인구 수 <br /> gdp 공연 해당 국가의 gdp <br /> tastemaker\_count 공연 해당 국가의 '아무 공연'에 make를 누른 사람의 수 <br /> ticket\_sales 공연 티켓 판매량 <br /> <br /> <strong>artist\_list</strong> <br /> artist 아티스트 <br /> gender 아티스트의 성별 ( 0 : 남성, 1 : 여성, 2 : 혼성 ) <br /> <br /> <strong>vlive\_data</strong> 아티스트의 V Live 데이터 <br /> artist 아티스트 <br /> upload\_date 비디오의 업로드 날짜 <br /> follower 아티스트의 V LIVE 팔로워 수 <br /> playtime 비디오의 재생 시간 ( 총 재생된 시간이 아닌, Video 자체의 길이 ) <br /> view\_count 비디오의 총 재생 횟수 <br /> like\_count 비디오의 '좋아요' 개수 <br /> comment\_count 비디오의 댓글 수 <br /> <br /> <strong>mv\_data</strong> 아티스트의 뮤직비디오 데이터 <br /> artist 아티스트 <br /> upload\_date 뮤직비디오의 업로드 날짜 <br /> view\_count 뮤직비디오 재생 횟수 <br /> like\_count 뮤직비디오의 '좋아요' 개수 <br /> dislike\_count 뮤직비디오의 '싫어요' 개수 <br /> comment\_count 뮤직비디오의 댓글 수 <br /> <br /> <strong>twitter\_data</strong> 아티스트의 트위터 데이터 <br /> artist 아티스트 <br /> upload\_date 트윗의 업로드 날짜 <br /> follower 아티스트의 트위터 팔로워 수 <br /> total\_tweet 아티스트가 작성한 총 트윗 개수 <br /> like\_count 해당 트윗의 '좋아요' 개수 <br /> retweet\_count 해당 트윗의 리트윗 수 <br /> comment\_count 해당 트윗의 댓글 수 <br />
</p>
언제까지 링크가 존재할지 모르겠지만, <https://bit.ly/2n69uPE> 으로 데이터를 다운받을 수 있다.

단상
----

이 작업을 하면서, 학교다닐적 과제 생각이 많이 났다. EDA(Exploratory Data Analysis, 탐색적자료분석)수업에서 과제를 받으면, 데이터를 살펴보는데만 시간이 엄청나게 걸린다. 분석 자체는 웬만한 경우에 있어, 코드 몇 줄 걸리지 않는다. 물론, 그 코드 자체를 충분히 잘 알고 있어야 제대로 쓰는 것이지만, 프로그램이 코드를 실행해주기 때문에 코드 몇 줄짜리다.

제일 시간이 많이 걸리는 부분은 데이터 전처리Data Processing와 EDA다.

자기자신과 관련이 없는 이런 데이터들을 주의깊게 살펴볼 사람은 거의 없겠지만, 일단 남겨두자.

Data, Library Import
--------------------

``` r
library(tidyverse)
library(lubridate)
library(neuralnet)

setwd("C:/Users/Phil2/Downloads")
concert <- read_csv("data_concert.csv")[-10]
artist <- read_csv("data_artist.csv")
vlive <- read_csv("data_vlive.csv")
mv <- read_csv("data_mv.csv")
twitter <- read_csv("data_twitter.csv")
concert
```

    ## # A tibble: 155 x 9
    ##    artist continent city  closing_date  make population   gdp
    ##    <chr>      <dbl> <chr> <chr>        <dbl>      <dbl> <dbl>
    ##  1 aaaa3~         1 76f3~ 2015.11.5    1.17      0.144  2.31 
    ##  2 aaaa3~         1 8d00~ 2015.11.5    1.02      0.0445 2.19 
    ##  3 aaaa3~         1 4cba~ 2015.11.5    1.10      0.0865 1.01 
    ##  4 aaaa3~         1 4d34~ 2015.11.5    1.59      0.0850 0.679
    ##  5 6db3d~         2 cf2c~ 2015.11.22   1.25      0.0524 2.85 
    ##  6 aaaa3~         1 6504~ 2015.11.18   1.17      0.545  1.90 
    ##  7 bdce4~         2 f485~ 2015.12.9    0.808     0.279  0.468
    ##  8 bdce4~         2 daf6~ 2015.12.9    0.772     2.31   0.185
    ##  9 bdce4~         2 5e86~ 2015.12.9    0.539     0.639  0.320
    ## 10 145a1~         1 ccb9~ 2016.3.13    1.14      0.538  1.91 
    ## # ... with 145 more rows, and 2 more variables: tastemaker_count <dbl>,
    ## #   ticket_sales <dbl>

``` r
artist
```

    ## # A tibble: 22 x 2
    ##    artist   gender
    ##    <chr>     <dbl>
    ##  1 aaaa33d9      0
    ##  2 6db3de5a      0
    ##  3 bdce474a      0
    ##  4 145a1215      0
    ##  5 550d7ae2      0
    ##  6 62a22f5c      2
    ##  7 8f4a1945      1
    ##  8 69821cdd      1
    ##  9 9813d312      0
    ## 10 f6450680      1
    ## # ... with 12 more rows

``` r
vlive
```

    ## # A tibble: 8,223 x 7
    ##    artist upload_date follower playtime view_count like_count comment_count
    ##    <chr>  <chr>          <dbl>    <dbl>      <dbl>      <dbl>         <dbl>
    ##  1 bdce4~ 2017.4.2      0.923    0.161      0.171      0.0754       0.0351 
    ##  2 62a22~ 2017.2.11     0.422    0.0764     0.101      0.0169       0.0176 
    ##  3 f6450~ 2016.4.25     0.0861   2.75       0.108      0.0150       0.120  
    ##  4 f6450~ 2017.12.7     0.0861   0.0726     0.0367     0.0103       0.0141 
    ##  5 368e6~ 2017.8.20     0.156    1.23       0.405      0.208        0.188  
    ##  6 368e6~ 2018.5.25     0.156    1.39       0.654      0.312        0.886  
    ##  7 8f4a1~ 2015.6.22     0.143    1.10       0.316      0.0354       0.414  
    ##  8 8f4a1~ 2017.6.7      0.143    2.62       0.0661     0.0426       0.129  
    ##  9 69821~ 2016.1.15     1.28     0.655      0.353      0.0290       0.291  
    ## 10 69821~ 2017.11.16    1.28     0.0634     0.167      0.0227       0.00773
    ## # ... with 8,213 more rows

``` r
mv
```

    ## # A tibble: 203 x 6
    ##    artist   upload_date view_count like_count dislike_count comment_count
    ##    <chr>    <chr>            <dbl>      <dbl>         <dbl>         <dbl>
    ##  1 145a1215 2017.12.20       0.964      1.70          0.838         1.64 
    ##  2 145a1215 2017.8.23        1.40       2.15          1.14          1.88 
    ##  3 786d89ee 2018.4.20        1.08       2.42          2.11          1.96 
    ##  4 f4055a26 2018.3.21        1.44       3.50          1.81          5.02 
    ##  5 145a1215 2017.1.25        0.816      1.62          0.558         1.38 
    ##  6 786d89ee 2018.1.23        1.90       3.77          2.42          3.69 
    ##  7 f4055a26 2017.10.4        1.17       2.49          0.774         2.37 
    ##  8 a9fba206 2017.10.11       0.263      0.834         0.142         0.531
    ##  9 145a1215 2016.9.8         1.25       2.29          0.976         1.26 
    ## 10 786d89ee 2017.8.23        0.982      1.79          0.750         1.41 
    ## # ... with 193 more rows

``` r
twitter
```

    ## # A tibble: 20,001 x 7
    ##    artist upload_date follower total_tweet like_count retweet_count
    ##    <chr>  <chr>          <dbl>       <dbl>      <dbl>         <dbl>
    ##  1 145a1~ 2018.6.17       1.68        2.05      3.91          2.89 
    ##  2 145a1~ 2018.6.16       1.68        2.05      4.71          3.70 
    ##  3 145a1~ 2018.6.14       1.68        2.05      3.74          3.17 
    ##  4 145a1~ 2018.6.19       1.68        2.05      3.96          2.85 
    ##  5 145a1~ 2018.6.16       1.68        2.05      3.99          3.17 
    ##  6 145a1~ 2018.6.21       1.68        2.05      0.686         0.320
    ##  7 145a1~ 2018.6.21       1.68        2.05      0.775         0.358
    ##  8 145a1~ 2018.6.20       1.68        2.05      0.814         0.361
    ##  9 145a1~ 2018.6.19       1.68        2.05      1.89          1.39 
    ## 10 145a1~ 2018.6.17       1.68        2.05      3.42          2.57 
    ## # ... with 19,991 more rows, and 1 more variable: comment_count <dbl>

그런데 작업을 하다보니, 꽤나 중대한 오류를 발견했다. `concert` 테이블의 `closing_date()`를 항상 염두에 두어야 하는데, 날짜에서 2016.6.4 &lt; 2016.9.6은 참이지만, 2016.6.4 &lt; 2016.10.8은 거짓이다. 06.04 &lt; 10.08는 참으로 인식을 하니, 날짜 부분에 수정이 필요하다.
