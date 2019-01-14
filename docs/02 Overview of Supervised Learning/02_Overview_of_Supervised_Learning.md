---
# 02 Overview of Supervised Learning
---

#![](cover1.png)

## Introduction

통계적 지도 학습의 궁극적 목적은 **input** (predictors, independent variables) 들을 사용하여 **output** (responses, dependent variables)들의 값을 예측하는 것이다.

$$Y=f(X)+\epsilon$$

Matrix Notation 의 경우 $N\times{p}$ matrix $\boldsymbol{X}$ 로 나타내게 된다. 모든 벡터들은 열벡터들로 가정되기 때문에 행렬 $\boldsymbol{X}$ 의 i번째 행은 ${x_i}^T$ 로 나타낸다.

지도학습에서의 task는 주어진 입력 벡터 $X$ 에 대한 출력값 $Y$ 에 대한 좋은 예측값 $\widehat{Y}$ 을 얻는 것이다.

회귀 문제가 아닌 분류 문제에서 이진 분류를 한다고 할 때 범주에 대한 예측값 $\widehat{G}$ 를 해당 범주에 속할 확률 $\hat{y}$ 의 값이 0.5 보다 크냐 아니냐로 결정한다. 이러한 접근 방식은 $K$-level 분류에도 적용된다. 

우리는 예측 규칙을 설계하기 위한 $N$ 개의 **training data** $(x_i,y_i)$ 또는 $(x_i,g_i)$ , $i=1,...,N$ 가 필요하다.


## Two Simple Approaches to Prediction : Least Squares and Nearest Neighbors

두 가지 단순하지만 강력한 예측 방법론이다. 바로 least squares 에 의한 선형 모델 적합과 $k$-nearest neighbor 예측 규칙이다. 

선형 모델은 구조에 대한 강력한 가정이 존재하고 안정적이지만 부정확한 예측 결과를 제공한다. $k$-nearest neighbor 방법은 구조에 대한 유연한 가정이 존재하며 종종 정확하지만 불안정한 예측 결과를 제공한다.

### Linear Models and Least Sqaures

주어진 입력 벡터 혹은 전치 행렬 ($X$ 는 열벡터) $X^T=(X_1,X_2,...,X_p)(n\times{p})$ 가 있을 때, 우리는 출력값 $Y$ 를 아래의 모델을 통해 예측한다.

$$\widehat{Y}=\hat{B_0}+\sum_{j=1}^{p}{X_j\hat{\beta_j}}$$

보통 1벡터를 $X$ 에 포함시켜 절편 추정치 $\hat{\beta_0}$ 를 계수 행렬 $\hat{\beta}$ 에 포함시키는 것이 편리하며, 선형 모델을 내적의 결과인 벡터 형태로 아래와 같이 표현할 수 있다.

$$\widehat{Y}_{(n\times{1})}=X^T_{(n\times{p})}\hat{\beta}_{(p\times{1})}$$

일반적으로 $\widehat{Y}$ 가 $K$ 벡터라고 하면 $\beta$ 는 $p\times{K}$ 의 계수 행렬이 될 것이다.

적합된 직선 $(X,\hat{Y})$은 $p+1$-차원의 입출력 공간에서의 hyperplane을 나타내게 된다. (단순선형회귀에서 회귀 직선이 1차원이듯이)

> $X$ 에 상수가 포함되어있을 경우, hyperplane은 원점을 포함하지 않는 subspace이다. 이는 $Y$-축을 $(0,\hat{\beta_0})$ 에서 자르는 affine set이다. (affine space는 원점이 없다)

**지금부터 우리는 절편이 $\widehat{\beta}$ 에 포함되어 있다고 가정한다.**

우리는 주어진 training data로 어떻게 선형 모델을 적합할 수 있을까? 다양한 방법들이 존재하지만, 가장 알려진 방법으로 **least square** 방법이 있다. 이 방법에서 우리는 residual sum of squares (RSS) 를 최소화하는 회귀 계수 $\beta$ 를 찾는다.

$$\text{RSS}(\beta)=\sum_{i=1}^{N}(y_i-x_i^T\beta)^2$$

$\text{RSS}(\beta)$ 는 모수들의 quadratic function (2차 함수) 이므로 최솟값이 항상 존재하지만, unique 하지는 않을 수 있다. 위 이차식의 해는 matrix notation으로 characterize하는 것이 가장 쉽다. 

$$\text{RSS}(\beta)=(\boldsymbol{y}-\boldsymbol{X}\beta)^T(\boldsymbol{y}-\boldsymbol{X}\beta)$$

이를 $\beta$ 에 대해 미분하면 아래의 **normal equation** 을 얻을 수 있고

$$\boldsymbol{X}^T(\boldsymbol{y}-\boldsymbol{X}\beta)=0$$

$\boldsymbol{X^TX}$ 가 non-singular 행렬이면 (역행렬이 존재하면), unique solution을 아래와 같이 얻을 수 있다.

$$\widehat{\beta}=(\boldsymbol{X^TX})^{-1}\boldsymbol{X^Ty}$$

전체 fitted surface를 $p$ 개의 모수인 $\hat{\beta}$ 들로 나타낼 수 있는 것에서 직관적으로 모델을 적합하는 데에 매우 많은 자료가 필요하지 않다는 것을 알 수 있다.

**분류 문제** 에서의 선형 모델에 대해 생각해보자. 

관측치가 나타내는 파란색 (0), 오렌지색 (1) 두 개의 색에 대한 이진 분류라고 했을 때, 범주에 속할 예측 확률 0.5 초과면 오렌지, 이하면 파란색으로 분류하게 된다. 이 때의 hyperplane, 즉 **decision boundary** 는 (이 경우 선형) $[x:x^T\hat{\beta}=0.5]$ 으로 나타나게 된다. 

이 경우에도 우리는 여러 개의 오분류 결과들을 확인할 수 있는데, 설계된 자료의 생성 과정에 대해 이야기하지 않은 상태에서 두가지 가능한 시나리오를 생각해보자.

* **Scenario 1:** 각 범주에 속하는 training data가 모평균이 다른 독립인 bivariate normal distribution 에서 생성되었다.

### Simulation

$$\text{Blue~}N_2(\begin{bmatrix}0\\1\end{bmatrix},\begin{bmatrix}1&0\\0&1\end{bmatrix})$$

$$\text{Orange~}N_2(\begin{bmatrix}1\\0\end{bmatrix},\begin{bmatrix}1&0\\0&1\end{bmatrix})$$


```r
library(tidyverse)
```

```
## Warning: package 'tidyverse' was built under R version 3.5.2
```

```
## -- Attaching packages -------------------- tidyverse 1.2.1 --
```

```
## √ ggplot2 3.1.0     √ purrr   0.2.5
## √ tibble  1.4.2     √ dplyr   0.7.7
## √ tidyr   0.8.1     √ stringr 1.3.1
## √ readr   1.1.1     √ forcats 0.3.0
```

```
## -- Conflicts ----------------------- tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
set.seed(2013122044)
## scenario 1 example
# independent bivariate normal distribution

# blue = N_2([0,1],[1,0,0,1])
blue=data.frame(X1=rnorm(100,0,1),X2=rnorm(100,1,1))
# orange = N_2([1,0],[1,0,0,1])
orange=data.frame(X1=rnorm(100,1,1),X2=rnorm(100,0,1))
# Y
Y=factor(c(rep('Blue',100),rep('Orange',100)))
# data
df=data.frame(cbind(rbind(blue,orange),Y))
# visualization
ggplot(df,aes(x=X1,y=X2))+
  geom_point(aes(color=Y))+
  scale_colour_manual(values=c('dodgerblue','orange'))+
  ggtitle('Visualization of Scenario 1')+
  theme_bw()+
  theme(plot.title=element_text(face='bold'))
```

![](02_Overview_of_Supervised_Learning_files/figure-html/unnamed-chunk-1-1.png)<!-- -->


* **Scenario 2:** 각 범주에 속하는 training data가 각 분포의 모평균 자체가 normal distribution을 따르는 10개의 low variance normal distribution의 mixture에서 생성되었다. (Gaussian Mixture)

$$\mu_{blue}\text{~}N_2(\begin{bmatrix}1\\0\end{bmatrix},\begin{bmatrix}1&0\\0&1\end{bmatrix})$$

$$\mu_{orange}\text{~}N_2(\begin{bmatrix}0\\1\end{bmatrix},\begin{bmatrix}1&0\\0&1\end{bmatrix})$$


```r
## 10 Gaussian Mixture Example
# generative model
generator=function(n){
  # parameters for bivariate mean distributions to generate 10 bivariate mean vectors
  n_mixture=10
  # additional paramters for class distributions
  sample_sd=sqrt(1/5)
  # gernerate 10 mean vectors for each class
  blue_mean_sample=data.frame(X1=rnorm(n_mixture,1,1),X2=rnorm(n_mixture,0,1))
  orange_mean_sample=data.frame(X1=rnorm(n_mixture,0,1),X2=rnorm(n_mixture,1,1))
  # blue
  idx1=sample(10,1)
  sample_blue=data.frame(X1=rnorm(n,blue_mean_sample[idx1,][[1]],sample_sd),X2=rnorm(n,blue_mean_sample[idx1,][[2]],sample_sd))
  # orange
  idx2=sample(10,1)
  sample_orange=data.frame(X1=rnorm(n,orange_mean_sample[idx2,][[1]],sample_sd),X2=rnorm(n,orange_mean_sample[idx2,][[2]],sample_sd))
  
  # final samples
  X=rbind(sample_blue,sample_orange)
  Y=factor(c(rep('Blue',100),rep('Orange',100)))
  df=data.frame(X,Y)
  return (df)
}
```



```r
# visualization for 100 samples for each class
set.seed(2013122044)
df=generator(100)
ggplot(df,aes(x=X1,y=X2))+
  geom_point(aes(color=Y))+
  scale_colour_manual(values=c('dodgerblue','orange'))+
  ggtitle('Visualization of Scenario 2')+
  theme_bw()+
  theme(plot.title=element_text(face='bold'))
```

![](02_Overview_of_Supervised_Learning_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

> Gaussian Mixture란?

Gaussian Mixture는 generative model로 가장 잘 설명될 수 있다. 먼저 어떤 Gaussian Component (ex. 10 Mixture of Bivariate Normal이면 10개 중 어떤 Bivariate Normal Random Variable을 사용할지) 를 사용할 지를 결정하는 이산형 변수를 생성하고 결정된 밀도에서 관측치를 생성하게 된다. 

각 범주에 속하는 관측치가 한 개의 Normal Random Variable을 갖는다면 선형의 decision boundary가 최적이며 겹치는 지역은 불가피하며 예측되어야 하는 미래 자료 또한 이러한 겹치는 지역에서 자유로울 수 없을 것이다. 

그렇다면 각 범주에 속하는 관측치가 여러 개의 좁게 모여 군집을 이루는 Normal Random Variable들을 갖는다면 이야기가 다르다. 최적의 decision boundary는 비선형이며 disjoint하고 보다 얻기 어려울 것이다.

### Nearest-Neighbor Methods

최근접 이웃 방법은 입력 공간 내에 존재하는 $x$ 와 가장 가까운 training set $T$ 에 있는 관측치들을 사용하여 $\widehat{Y}$ 를 형성한다.

$$\widehat{Y}(x)=\frac{1}{k}\sum_{x_i\in{N_k(x)}}y_i$$

$N_k(x)$ 는 $x$ 의 $k$ 개의 최근접 training data들로 정의된다. 우리는 여기서 거리를 정의해야하며 Euclidean distance를 가정한다. 

$$L_2\text{-norm}=\sqrt{\sum_{i=1}^{n}(x_1-x_2)^2}$$

> 즉, 가장 가까운 k개의 훈련 데이터들의 종속변수 값들을 평균내어 추정하는 것

$K$-최근접 이웃 적합에서 training data에 대한 오차는 $k$에 대해 증가함수이며, $k=1$ 인 경우 항상 0일 것이다. 

$K$-최근접 이웃 적합은 $p$개의 모수들을 가지던 least-square와 다르게 $k$의 값만이 결정한다. 

우리는 **효과적인** $k$의 값이 $N/k$ 이며 일반적으로 $p$ 보다 크고, $k$ 의 값이 증가함에 따라 감소하는 것을 확인할 수 있다.

> 이유에 대해서는, 이웃들이 겹치지 않는다면, $N/k$ 개의 이웃집단들이 있을 것이고 우리는 각 이웃집단에 대해 한 개의 모수 (a mean) 를 적합하게 된다.

또한 우리가 training set에서 $k$ 를 선택하는 기준으로 sum of squared error를 사용할 수 없다는 것 또한 자명하다 (항상 $k=1$ 을 선택하게 될 것이므로)

따라서 $K$-최근접 이웃 방법은 위에서 소개된 시나리오 중 **2번째 시나리오**에 보다 적합할 것이다. (정규 분포를 따르는 자료에 대해 decision boundary가 필요 이상으로 noisy할 것이기 때문에)

소개된 두 방법에 대해 **Least Square** 같은 경우 적합 모델(model or decision boundary) 이 **Low Variance** 그리고 **High Bias** 를 갖고 **K-최근접 이웃** 같은 경우 **High Variance** 그리고 **High Bias**를 갖는다. 

책에서는 Train 그릐고 Test Set에서의 Error Rate와 Optimal Bayes Error Rate가 $k$ 에 따라 어떻게 나타나게 되는지 보여주고 있다.

#![](knn.PNG)

**현재 사용되고 있는 많은 유명한 기법들은 이 두 가지 단순한 절차들의 변형이다.** 실제로 1-최근접 이웃 방법은 저차원 문제에서의 많은 부분에서 사용되고 있다. 

아래의 방법들이 이 단순한 절차들이 응용되고 개선된 예이다.

* **Kernel Method**는 목표 지점까지의 거리가 감소할 수록 0에 가까워지는 가중치를 사용한다.

* 고차원 공간에서 distance kernel은 특정 변수들을 강조하기 위해 변형된다.

* **Local Regression** 은 선형 모델을 locally weighted least squares를 활용하여 적합한다.

* 선형 모델을 원래의 입력값의 **Basis Expansion**으로써 임의의 복잡한 모델로의 적합이 가능하게 한다.

* **Projection Pursuit** 그리고 **Neural Network** 모델들은 비선형으로 변환된 선형 모델들의 합으로 이루어진다.

## Statistical Decision Theory

$X\in{R^p}$ 를 실수 확률 입력 벡터라고 하고 $Y\in{R}$ 를 실수 확률 출력 변수라고 하며 결합 분포를 $\text{Pr}(X,Y)$ 라고 한다면,

우리는 주어진 입력값 $X$ 를 가지고 $Y$ 를 예측하는 함수 $f(X)$ 를 찾고자 한다.

이러한 이론은 **loss function** $L(Y,f(X))$ 을 통해 예측에서의 오차를 penalize할 수 있고, 가장 널리 쓰이고 간편한 예가 바로 **squared error loss** $L(Y,f(X))=(Y-f(X))^2$ 이다.

이는 $f$를 찾기 위한 기준이 될 수 있는 **Expected (Squared) Prediction Error** 를 계산할 수 있게 한다. 

$$\text{EPE}(f)=E[(Y-f(X))^2]=\int(y-f(x))^2f(x,y)dydx$$

위 식을 $X$ 에 대해 conditioning 하면 우리는 EPE를 아래와 같이 쓸 수 있다.

$$\text{EPE}(f)=E_{X}E_{Y|X}[(Y-f(X))^2|X]$$

EPE를 pointwise하게 최소화할 수 있는 $\hat{f}(x)$ 는

$$\hat{f}(x)=\text{argmin}_cE_{Y|X}[(Y-c)^2|X=x]$$

이며, 이에 대한 해는 바로

$$f(x)=E(Y|X=x)$$ 

이며, conditional expectation이고, **regression function** 이기도 하다. 즉, 어느 지점 $X=x$ 에서든 $Y$ 에 대한 최선의 예측은 조건부 평균이다. (최선의 기준이 average squared error일 때)

**최근접 이웃** 의 경우에는 

$$\hat{f}(x)=\text{Ave}(y_i|x_i\in{N_k(x)})$$

이 된다. ($N_k(x)$ 는 training set $T$ 에서 $x$ 과 가장 가까운 $k$ 개의 관측치들의 집합)

위 식에서 두 가지의 **근사** 가 이루어지는데,

* 기댓값이 표본 데이터에 대한 평균으로 근사된다.

* 한 지점에서 conditioning 하는 것은 목표 지점에 가까운 어떤 지역에 대해 conditioning 되는 것으로 완화된다.

표본의 크기가 커질 수록 mild regularity condition들 하에서 근사되는 성질을 knn 또한 보여준다. 하지만 우리는 종종 표본 크기가 매우 크지 않고, 추가적으로 **차원** $p$ 가 커질 수록 발생하는 문제 또한 뒤에서 소개된다.

다시 선형 회귀 적합으로 돌아가, 최소제곱법과 $k$-최근접 이웃 방법 모두 조건부 기댓값을 평균을 통해 근사하는 것이라는 것을 알 수 있다. 하지만, 모델 가정에서 큰 차이가 있다.

* Least Squares 는 $f(x)$ 가 globally linear function에 의해 잘 근사될 수 있다고 가정한다. 

* $\beta=(E(XX^T))^{-1}E(XY)$

반면 

* $k$-최근접 이웃은 $f(x)$ 가 locally constant function에 의해 잘 근사될 수 있다고 가정한다.

책에서 후에 소개될 많은 현대 기법들은 Model Based 이며 엄격한 선형 모델보다 훨씬 유연하다. 

> ? Example of Additive Model

또한 loss function에는 위에서 이야기한 least square의 $L_2$ 만 있는 것이 아니다. 이를 $L_1$ 으로 대체하게 될 경우 $(E[|Y-f(X)|])$ 해는 $\hat{f}(x)=median(Y|X=x)$ , 즉 조건부 중앙값이 된다. 이는 location에 대한 다른 measure이며, 조건부 평균보다 더욱 로버스트한 추정량이다. 하지만 미분에서의 불연속 지점이 발생하기 때문에 수치적으로 널리 쓰이지 않는다.

**범주형 종속 변수** 의 경우에는 같은 개념으로 다른 loss function만을 정의해주면 된다. 우리의 loss function은 $K\times{K}$ 행렬 $\boldsymbol{L}$ 로 표현될 수 있고, (confusion matrix) $L(k,l)$ 은 $G_k$ 에 속하는 관측치를 $G_l$ 에 분류했을 때의 cost이다. 

가장 많은 경우에 **zero-one** loss function 을 사용한다.

$$\text{EPE}=E[L(G,\hat{G}(X)]$$

다시 conditioning 하여 아래와 같이 EPE를 나타낼 수 있다.

$$EPE=E_X\sum_{k=1}^{K}L(G_k,\hat{G}(X))\text{Pr}(G_k|X)$$

이 역시 EPE를 아래와 같이 최소화하는 $\hat{G}(x)$ 를 나타낼 수 있고,

$$\hat{G}(x)=\text{argmin}_{g\in{G}}\sum_{k=1}^{K}L(G_k,g)\text{Pr}(G_k|X=x)$$

0-1 loss function 의 경우 이는 아래와 같이 정리될 수 있다.

$$\hat{G}(x)=\text{argmin}_{g\in{G}}(1-\text{Pr}(g|X=x))$$

또는

$$\hat{G}(x)=G_k\text{ if Pr}(G_k|X=x)=\text{max}_{g\in{G}}\text{Pr}(g|X=x)$$

위의 해는 **Bayes Classifier** 라고 하며, 이는 우리는 이산형 조건부 분포 $\text{Pr}(G|X)$ 를 사용하여 가장 속할 확률이 높은 범주에 분류하는 것이다. 이러한 Bayes Classifier의 오차율을 **Bayes Rate** 라고 한다.

다시 $k$-NN 으로 돌아와, 이 방법이 직접적으로 이러한 해에 근사하는 것을 보일 수 있다. (majority vote, 다수결로) 

하지만 이 경우에도 $k$-NN은 지점에 대한 조건부 확률이 지점의 이웃의 조건부 확률로 완화되며 확률들이 training-sample proportion에 의해 추정된다.


## Local Methods in High Dimensions

KNN averaging 을 통해 training data 의 표본 크기가 이상적으로 커질수록 이론적인 조건부 기댓값에 근사할 수 있을까?

이러한 접근 방법과 직관은 **고차원** 에서 철저히 부숴지며 이를 **차원의 저주** 라고 한다.

$p$-차원의 단위 hypercube (테서렉트, 모든 변의 길이가 같은 도형) 에 균등하게 분포되어 있는 입력값에 대한 최근접 이웃 절차를 생각해보자. 우리가 $r/N$ 만큼을 잡아내기 위한 목표 지점에 대한 hypercubical 이웃 집단을 보낸다고 하면, 

이는 결국 $\frac{r}{\text{단위 부피}}$ 이므로 기댓변의 길이는  $e_p(r)=r^{1/p}$ 가 될 것이다. 즉, 10차원 입력 공간에서는 $e_{10}(0.01)=0.63$ , $e_{10}(0.1)=0.8$ 이다. 

즉, 이는 전체 자료의 1% 또는 10% 를 local average를 형성하기 위해 포착해야 하는데 이 경우 전체 길이의 63% 또는 80%나 되는 길이를 각 입력 변수마다 다뤄야한다는 것이다. 이러한 이웃 집단들은 더 이상 **'local'** 하지 않다.

이렇다고 $r$ 을 감소시키게 되면 더 적은 관측치들로 평균내기 때문에 적합의 분산은 커지게 된다. 

고차원에서의 sparse sampling 의 다른 결과로는 모든 표본 지점들이 표본공간의 모서리(한쪽)에 가깝다는 것이다. 

즉, 다른 관측 지점과 가깝다기 보다는 표본 공간의 경계에 더 가까워버린다는 것이다. 

> 차원이 커질 수록 그 공간 자체는 기하급수적으로 증가하므로 한쪽에 몰려있게 된다는 식의 표현인듯하다. 이는 고차원의 데이터일 수록 같은 공간을 차지하기 위해 단차원의 샘플 갯수 $N^p$ 가 필요하다는 표현과 연결된다. (차지하는 공간 = $N^{1/p}$)

이게 문제가 되는 이유는 바로 training 표본의 모서리 근처에서는 예측이 더 어렵기 때문이다. 

> One must extrapolate from neighboring sample points rather than interpolate between them (?)


#![](curseofdimension.PNG)

$U_p(-1,1)$ 에서 1000개의 training samples을 생성하여 시뮬레이션을 진행해보자. 

$X$와 $Y$의 true relationship을 아래와 같이 가정한다.

$$Y=f(X)=e^{-8||X||^2}$$

1-Nearest Neighbor 규칙을 사용하여 test point $x_0=0$ 에서의 $y_0$ 을 예측하고자 한다. 

training set을 $T$ 로 표기하여 우리는 expected prediction error at $x_0$ 을 1000개의 sample에 대해 평균내어 계산할 수 있다. 

문제가 deterministic 하므로 이는 $f(0)$ 에 대한 MSE이다.

$$\text{MSE}(x_0)=E_{T}[(f(x_0)-\hat{y}_0)^2]=$$

$$E_T[(f(x_0)-E_{T}(\hat{y}_0)+E_{T}(\hat{y}_0)-\hat{y}_0)^2]=$$

> interaction term (2XY) is 0

$$E_{T}[(\hat{y}_0-E_{T}(\hat{y}_0))^2]+(E_{T}[\hat{y}_0]-f(x_0))^2=\text{Var}_{T}(\hat{y}_0)+\text{Bias}^2(\hat{y}_0)$$

> Bias-Variance Decomposition (Always Possible and Often Useful)

### Simulation


```r
## curse of dimensionality example
set.seed(2013122044)

xgenerator=function(n,p){
  # 1000 training saples x_i generated        uniformly on [-1,1]^p
  X=data.frame(matrix(runif(n*p,-1,1),nrow=n,ncol=p))
  colnames(X)=paste0('X',1:p)
  return (X)
}

# assume that the true relationship         between X and Y is without any measurement   error
true_f=function(X){
  Y=numeric(nrow(X))
  for (i in 1:nrow(X)){
    Y[i]=exp(-8*(sum(X[i,]^2)))  
  }
  return (Y)
}

# we use 1-nn rule to predict y_0 at the    test point x_0=0

euc_dist=function(xvec,p){
  test=data.frame(t(rep(0,p)))
  return (sqrt(sum((xvec-test)^2)))
}

knnsimulator=function(n,k,p){
  X=xgenerator(n,p)
  dist=apply(X,1,euc_dist,p=p)
  knn=X[order(dist)[1:k],]
  return (knn)
}
```


```r
# True relationship in one dimension
X=xgenerator(1000,1)
f=true_f(X)
df=cbind(X,f)[order(X),]

ggplot(df,aes(x=X1,y=f))+
  geom_line()+
  labs(x='X',y='f(X)')+
  ggtitle('True Relationship in 1 Dimension')+
  theme_bw()+
  theme(plot.title=element_text(face='bold'))
```

![](02_Overview_of_Supervised_Learning_files/figure-html/unnamed-chunk-5-1.png)<!-- -->


```r
# simulation of knn (p=2,k=10)
X=xgenerator(n=1000,p=2)
knn=knnsimulator(n=1000,p=2,k=10)
test=data.frame(t(rep(0,2)))
  
ggplot(X)+
  geom_point(aes(x=X1,y=X2),color='darkgrey')+
  geom_point(data=test,aes(x=X1,y=X2,color='red'))+
  geom_point(data=knn,aes(x=X1,y=X2,color='orange'))+
  scale_color_manual(name=NULL,
            values=c('red'='red','orange'='orange'),
            labels=c('Test point','Nearest neighborhood'))+
  ggtitle('10-Nearest Neighbors in Two Dimension')+
  theme_bw()+
  theme(plot.title=element_text(face='bold'))+
  coord_fixed()
```

![](02_Overview_of_Supervised_Learning_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


```r
# simulation of distance to 1-nn vs. dimension

# (y0=1)

distance_sim=function(p,n,nsim){
  # distance
  distance=numeric(nsim)
  # yhat
  yhat=numeric(nsim)
  # test data point (x0=0)
  test=data.frame(t(rep(0,p)))
  # iteration
  for (i in 1:nsim){
    X=xgenerator(n,p)
    distance[i]=min(apply(X,1,euc_dist,p=p))
    nn=X[which.min(apply(X,1,euc_dist,p=p)),]
    yhat[i]=true_f(data.frame(nn))
  }
  # sample variance
  variance=mean((mean(yhat)-yhat)^2)
  # sample squared bias
  squared_bias=(1-mean(yhat))^2
  
  # results
  result=data.frame(Mean_Distance=mean(distance),
                    Variance=variance,
                  Squared_Bias=squared_bias,
                  MSE=variance+squared_bias)
  
  return (result)
}
```


```r
# 100 simulations for each dimension, 100 samples for 1~10 dimensions

result=distance_sim(p=1,n=500,nsim=100)
for (i in 2:10){
  result=rbind(result,distance_sim(p=i,n=500,nsim=100))
  
}


# visualizations

result$Dimension=1:10

# average distance to nearest neighbor
ggplot(result,aes(x=Dimension,y=Mean_Distance))+
  geom_point(color='tomato',size=3)+
  geom_line(color='tomato',size=1)+
  theme_bw()+
  ggtitle('Average Distance to Nearest Neighbor')+
  theme(plot.title=element_text(face='bold'))+
  labs(y='Average Distance')
```

![](02_Overview_of_Supervised_Learning_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
result2=gather(result[,-1],key,value,-Dimension,factor_key=T)


# variance, squared bias, and MSE
ggplot(result2,aes(x=Dimension,y=value,color=key))+
  geom_point(size=3)+
  geom_line(size=1)+
  theme_bw()+
  ggtitle('MSE vs. Dimension')+
  theme(plot.title=element_text(face='bold'),
        legend.title=element_blank())+
  labs(y='MSE')
```

![](02_Overview_of_Supervised_Learning_files/figure-html/unnamed-chunk-8-2.png)<!-- -->

이를 통해 차원이 커질수록 가장 가까운 이웃과의 거리 자체가 커지므로 절대적인 Bias와 Variance 또한 커지게 되는 것을 확인할 수 있다. 

**특별히** Bias term의 거리에 대한 종속성은 실제 function에 영향을 받고 그 function이 항상 저차원에만 영향을 주는 경우에는 분산이 지배적이게 된다.

### What good about the linear model?

> 적합되는 모델에 대한 가장 제약을 걸어 차원의 저주에서 벗어날 수 있다.


$$Y=X^T\beta+\epsilon$$

위와 같은 선형 모형을 가정하자. $(\epsilon\sim{N(0,\sigma^2))}$

우리는 모델을 training data에 대한 least squares 방법으로 적합하여 임의의 test point $x_0$ 에 대해 $\hat{y}_0=x^T_0\hat{\beta}$ 를 갖고 

$$\hat{y}_0=x^T_0\beta+\sum_{i=1}^{N}l_i(x_0)\epsilon_i$$

(where $l_i(x_0)$ is the $i$th element of $\boldsymbol{X(X^TX)^{-1}}x_0$)

이러한 모델 하에서 least square estimates 는 unbiased 하므로, 우리는 아래와 같이 EPE를 나타낼 수 있다.

$$\text{EPE}(x_0)=E_{y_0|x_0}E_T[(y_0-\hat{y}_0)^2]\\=Var(y_0|x_0)+E_T(\hat{y}_0-E_T\hat{y}_0)^2+(E_T[\hat{y}_0-x_0^T\beta])^2$$

$$\therefore{\sigma^2+E_T[x_0^T(\boldsymbol{(X^TX})^{-1}x_0\sigma^2]+0^2}$$

variance는 $x_0$ 에 종속적이고, $N$ 이 크고 $T$ 가 random하게 추출되었다면, $E(X)=0$ 을 가정하면 $\boldsymbol{X^TX}\to{N\text{Cov}(X)}$ 이고

$$E_{x_0}\text{EPE}(x_0)\sim{E_{x_0}[x_0^T\text{Cov}(X)^{-1}x_0\sigma^2/N+\sigma^2]}\\=\text{trace}[\text{Cov}(X)^{-1}\text{Cov}(x_0)]\sigma^2/N+\sigma^2\\=\sigma^2(p/N)+\sigma^2$$

위를 통해 expected squared predicted error가 $p$ 에 대한 선형증가함수임을 알 수 있다. (기울기 $\sigma^2/N$) 

따라서 $N$ 이 크거나 동시에 $\sigma^2$ 이 작다면, 이러한 분산의 증가는 무시할 수 있게 될 정도로 작아진다. (deterministic한 경우 0)

**즉,** 엄격한 가정 사항 하에서, 선형 모델은 unbiased 하며 무시할 정도로 분산이 작다. 하지만 이러한 가정들이 틀릴 경우에는 1-NN이 월등할 것이다. 

후술할 여러 모델들은 이러한 엄격한 선형 모델과 극도록 유연한 1NN 사이를 오가며 고차원에서의 함수의 복잡도의 지수적 증가를 피하기 위해 제안된 각자의 가정과 편향들에 대해 배울 것이다.

## Statistical Models, Supervised Learning and Function Approximation

*"finding a useful approximation of $f(x)$"*

이론적 배경 하에서 우리는 squared error loss 가 회귀 함수 $f(x)=E(Y|X=x)$ 를 도출한다고 배웠다. 

최근접 이웃 방법의 경우 이러한 조건부 기댓값에 대한 **직접적인 추정치**들로 볼 수 있고, 최소한 아래 두 개의 경우에서 오작동하는 것을 확인하였다.

* 입력 공간의 차원이 높다면 최근접 이웃들이 목표 지점에서 멀어지게 되고, 큰 오차를 낳는다.

* 특별한 구조가 존재한다고 하면 이를 추정치의 편향과 분산을 모두 줄이는데 사용할 수 있다.

### A Statistical Model for the Joint Distribution Pr(X,Y)

$$Y=f(X)+\epsilon$$

이 있다고 하자.$(E(\epsilon)=0$ and independent of $X)$

이러한 모델 $f(x)=E(Y|X=x)$ 에 대해, 실제로 조건부 분포 $Pr(Y|X)$ 는 $X$ 에 오직 조건부 평균 $f(x)$ 를 통해 의존한다.

이러한 가법 오차 (additive error) 모델은 실제 관계에 대한 유용한 근사 방법이다. 

더 나아가 여러 체계에서 입출력쌍 $(X,Y)$ 는 deterministic relationship $Y=f(x)$ 를 갖지 않을 것이다. 

일반적으로 **다른 측정되지 않은 변수들**이 $Y$ 에 영향을 미칠 것이며, 이 또한 측정 오차를 수반할 것이다. 

가법 모델은 deterministic 한 관계에서 벗어난 이러한 부분들을 오차항 $\epsilon$ 을 통해 잡아낼 수 있다고 가정한다.

특정 문제들의 경우 deterministic relationship이 성립한다. 많은 분류 문제들이 이러한 형태이며, response surface가 $R^p$ 차원에서 정의된 색칠된 지도의 형태로 생각될 수 있다.

training data는 이러한 지도 ${x_i,g_i}$ 에서 색칠된 예시들로 이루어져 있고, 우리의 목표는 다른 어떤 지점 또한 색칠할 수 있는 것이다.

이 경우 함수는 결정론적이고, randomness는 training points들의 $x$ 위치를 통해 발생한다.

> Why is it deterministic?

> 오차항에 기반한 모델들에 적합한 기술들로 이러한 문제들이 해결될 수 있는가?

위에서 서술한 관계식의 가정(오차항의 독립성, 등분산성, 정규성) 은 반드시 필요하지는 않지만, 우리가 오차 제곱을 산술평균낼 때를 생각하면 합당하다. (in $EPE$ Criterion)

이러한 모델의 경우 least squares를 모델 추정에 있어서의 기준으로 활용하는 것이 합당하다. 

**오차의 독립성 가정**을 피하기 위한 단순한 변형의 방법도 있다. 예를 들어, 우리는 $Var(Y|X=x)=\sigma(x)$ 를 만들어 평균과 분산이 모두 $X$에 의존하게 할 수 있다.

일반적으로 조건부 분포 $Pr(Y|X)$ 는 $X$에 복잡한 방법으로 종속적일 수 있지만 가법 오차 모델은 이를 **배제한다.**

가법 오차 모델은 일반적으로 질적 종속 변수 $G$ 에는 사용되지 않는다. 이 경우 목표 함수 $p(X)$ 는 조건부 밀도 $Pr(G|X)$ 이고, 이는 직접적으로 모델링될 수 있다.

예를 들어, 두 개의 범주를 갖는 자료의 경우, 자료가 독립적인 이진 시행에서 생성되었다고 가정하는 것이 합리적이다. (그 중 한 개 범주의 자료 추출 확률이 $p(X)$를 갖는)

따라서 $Y$ 가 0-1로 코딩되었다면 $E(Y|X=x)=p(x)$ 가 되지만 분산이 $x$에 **종속적**이다. $(Var(Y|X=x)=p(x)\times(1-p(x)))$

### Supervised Learning

머신 러닝 관점에서의 함수 적합 패러다임은 training set과 입출력이 존재하는 시스템에서 
학습 알고리즘을 통해 근사가 이루어진다.

이러한 학습 알고리즘은 입출력 관계 $\hat{f}$ 를 원 자료와 생성된 출력값의 차이 $y_i-\hat{f}(x_i)$ 를 통해 변형할 수 있다는 성질을 갖고 이러한 과정을 **learning by example** 이라고 한다. 

### Function Approximation

통계, 수학적 관점에서의 이러한 학습 패러다임은 **함수 근사** 이며 머신 러닝 분야 (인간의 추리 과정을 모방) 와 신경망 (뇌의 생물학적 성질 모방) 에 사용되었다. 

많은 근사 방법들은 자료에 적합하게 변형될 수 있는 모수 집합 $\theta$ 를 갖는다. 

예를 들어, 선형 모델 $f(x)=x^T\beta$ 는 $\theta$ 를 $\beta$ 로 갖는다. 

또 다른 유용한 근사기는 **linear basis expansion** 으로 표현될 수 있다. 

$$f_\theta(x)=\sum_{k=1}^{K}h_k(x)\theta_k$$

$h_k$ 는 적합한 함수의 집합 혹은 입력 벡터 $x$ 의 변환이다. 전통적 예시로 polynomial & trigonometric expansion이 있고 $h_k$ 가 $x_1^2, x_1x_2^2, \text{cos}(x_1)$ 등과 같은 경우를 말한다.

우리는 비선형 확장 또한 마주하게 될텐데, 신경망 모델에서 사용되는 sigmoid 변환이 대표적 예이다.

$$h_k(x)=\frac{1}{1+\text{exp}(-x^T\beta_k)}$$

우리는 least square로 residual sum of squares를 모수의 함수로 취급, 최소화시켜 모수들 $\theta$ 를 추정하는 데에 사용할 수 있었다.

$$\text{RSS}(\theta)=\sum_{i=1}^{N}(y_i-f_{\theta}(x_i))^2$$

선형 모델의 경우 단순한 closed-form 의 해를 최소화 문제에서 얻을 수 있다. 이는 basis function 방법론에서도 다른 어떤 잠재 모수들이 존재하지 않는한 동일하다. 

만약 다른 경우에는 해가 iterative methods 혹은 numerical optimization을 통해 얻어져야 한다.

least squares 말고도 추정에서 널리 사용되는 개념이 바로 **maximum likelihood estimation** 이다.

$$L(\theta)=\prod_{i=1}^{n}f(x_i;\theta)$$

$$l(\theta)=\sum_{i=1}^{n}\text{log}f(x_i;\theta)$$

MLE를 얻는 과정의 원리는 단순하다. $\theta$ 의 값으로 가장 합리적인 경우는 **관측된 표본을 얻을 확률이 가장 높은 경우**이다.

우리는 배웠듯이 가법 오차 모델 $Y=f_\theta(X)+\epsilon$ $(\epsilon\sim{\text{N}(0,\sigma^2)})$에 대한 least squares estimator 가 조건부 가능도를 사용한 maximum likelihood estimator와 같다는 것을 알고 있다.

$$Pr(Y|X,\theta)=N(f_\theta(X),\sigma^2)$$

따라서 추가적인 정규성 가정이 보다 제한적일 것 같아 보여도, 결과는 같다.

자료의 log-likelihood는 아래와 같고

$$l(\theta)=-\frac{N}{2}\text{log}(2\pi)-N\text{log}\sigma-\frac{1}{2\sigma^2}\sum_{i=1}^{N}(y_i-f_\theta(x_i))^2$$

$\theta$ 를 포함하는 항은 오직 마지막 항이며, 이는 $\text{RSS}(\theta)$ 에 대한 scalar negative multiplier 이다. 

보다 흥미로운 예시는 질적 종속 변수 $G$에 대한 회귀 함수 $Pr(G|X)$ 를 위한 다항 분포 likelihood 경우이다.

우리가 주어진 $X$ 에 대한 각 범주에 속할 조건부 확률을 위한 모델 $Pr(G=G_k|X=x)=p_{k,\theta}(x),\text{ }k=1,...,K$ 가 있다고 하자.

이 경우의 log-likelihood는 **(cross-entropy)** 아래와 같다.

$$l(\theta)=\sum_{i=1}^{N}\text{log}p_{g_i,\theta}(x_i)$$

## Structured Regression Models

임의의 함수 $f$ 에 대한 RSS 기준을 고려해보자. 

$$\text{RSS}(f)=\sum_{i=1}^{N}(y_i-f(x_i))^2$$

위의 식을 최소화하는 것은 무수한 많은 해들을 낳는다 : training points $(x_i,y_i)$ 를 지나는 그 어떤 함수 $\hat{f}$ 가 모두 해가 될 수 있다. 

따라서 어떤 특정 해는 test points에 대해 형편없는 예측 성능을 보여줄 것이다. 

만일 관측치 쌍의 갯수가 많다면 이러한 위험은 제한될 것이다. 

유한한 $N$ 에서 유용한 결과를 얻기 위해서 우리는 위의 최소화 문제에 제약을 걸어 보다 작은 함수의 집합을 얻어야한다. 

이러한 제약의 결정을 위한 과정은 자료의 외부에서 이루어진다. 이러한 제약들은 때때로 $f_\theta$ 의 모수적 재표현으로 이루어지거나 학습 방법 자체에서 설계된다. 

이러한 해의 제약들은 이 책의 **주요 주제**이다. 

명심해야할 것은, 그 어떠한 $f$ 에 주어지는 제약들이 실제로 해의 복잡성으로 인해 발생하는 모호함을 제거할 수는 없다는 것이다.

가능한 많은 제약들이 존재하지만, 각각은 유일 해로 이어지며, 따라서 애매함은 어떤 제약을 선택할 것이냐의 문제로 전환될 수 있다.

일반적으로 대부분의 학습 방법에서 적용하는 제약들은 제약의 **복잡도** 로 기술될 수 있다.

> 추가적 내용 존재.. 어려움

## Classes of Restricted Estimators

비모수적 회귀 기법 또는 여러 종류로 나뉘는 학습 방법의 다양성들은 적용되는 제약의 nature에 의해 결정된다. 

이러한 여러 종류의 제약들은 종종 **smoothing** 모수라고 불리는 한 개 이상의 모수들과 관련있다.

여기서는 세 가지의 넓은 의미의 분류를 소개하고 있다.

### 1. Roughness Penalty and Bayesian Methods

$$\text{PRSS}(f;\lambda)=\text{RSS}(f)+\lambda{J}(f)$$

예를 들어, 1차원 입력 벡터에 대해 유명한 **cubic smoothing spline** 방법은 아래의 penalized least-squares criterion의 해이다.

$$\text{PRSS}(f;\lambda)=\sum_{i=1}^{N}(y_i-f(x_i))^2+\lambda\int[f^{\prime\prime}(x)]^2dx$$

이 경우의 페널티항의 roughness 는 $f$ 의 2차 미분의 큰 값들에 영향을 많이 미치며, 페널티의 양은 $\lambda\ge0$ 에 의해 결정된다. 

만일 $\lambda=0$ 일 경우 페널티가 존재하지 않고, $\lambda=\infty$ 일 경우 $x$ 내 선형 함수만이 허용될 것이다.

페널티 함수 $J$ 는 어떠한 차원에서 존재하는 모든 함수에 의해 설계될 수 있으며 특별한 버전이 특별한 구조를 적용하기 위해 생성될 수 있다.

이렇게 페널티 함수, 또는 **regularization** 방법들은 우리의 이러한 종류의 함수의 smooth behavior에 대한 사전적 믿음을 반영하며, 이는 실제로 **Bayesian Framework** 내에서 적용 가능하다.

페널티 $J$ 는 **log-prior** 와 상응하며, $\text{PRSS}(f;\lambda)$ 는 **log-posterior distribution** 라고 할 수 있으며 $\text{PRSS}(f;\lambda)$ 를 최소화하는 것은 **posterior-mode** 를 찾는 것과 같다. 


### 2. Kernel Methods and Local Regression

이러한 방법론들은 local neighborhood의 nature를 정의함을 통해 회귀 함수의 추정치 또는 조건부 기댓값을 명시적으로 제공한다.

local neighborhood는 **kernel function** $K_\lambda(x_0,x)$ 를 통해 정의되며 이는 $x_0$ 주변의 $x$ 의 지점들에 가중치를 부여한다.

예를 들어, **Gaussian Kernel** 은 정규확률밀도함수에 기초한 가중치 함수를 갖고

$$K_\lambda(x_0,x)=\frac{1}{\lambda}\text{exp}(-\frac{||x-x_0||^2}{2\lambda})$$

$x_0$ 과의 **squared euclidean distance** 를 통해 멀어지는 지점들에 대해서 거리에 지수적으로 가중치를 부여하는 것이다. 

모수 $\lambda$ 는 정규확률밀도의 분산을 나타내며, 이웃의 너비를 결정한다.

커널 추정치의 가장 단순한 형태는 **Nadaraya-Watson weighted average** 이다.

$$\hat{f}(x_0)=\frac{\sum_{i=1}^{N}K_\lambda(x_0,x_i)y_i}{\sum_{i=1}^{N}K_\lambda(x_0,x_i)}$$

일반적으로 우리는 $f(x_0)$ 에 대한 local regression 추정치를 $f_\hat{\theta}(x_0)$ 로 나타내며 

$$\text{RSS}(f_\theta,x_0)=\sum_{i=1}^{N}K_\lambda(x_0,x_i)(y_i-f_\theta(x_i))^2$$

를 최소화하는 $\hat{\theta}$ 을 찾고 $f_\theta$ 는 어떤 모수적 함수이다. (ex. 저차수 다항식)

* $f_\theta(x)=\theta_0$ , 상수함수 : Nadaraya-Watson 추정치와 같은 결과를 도출.

* $f_\theta(x)=\theta_0+\theta_{1}x$ , 널리 알려진 local linear regression model 을 도출.

따라서 **최근접 이웃 방법**들은 보다 자료에 종속적인 metric을 갖는 **커널 방법**으로 생각할 수 있다.

실제로, $k$-최근접 이웃에서의 metric은 

$$K_k(x,x_0)=I(||x-x_0||\le||x_{(k)}-x_0||)$$

이며 $x_(k)$ 는 $x_0$ 와 $k$번째로 멀리 있는 관측치를 의미하고 $I(S)$ 는 집합 $S$ 의 indicator function이다.

이러한 방법들은 당연히 고차원에서 변형되어야 한다. (차원의 저주) 

### 3. Basis Functions and Dictonary Methods

이 방법론들은 익숙한 선형 또는 다항 expansion을 포함한다. 하지만 보다 중요하게 복잡하고 다양한 모델들을 포함한다. 

$f$ 에 대한 모델은 basis function의 linear expansion이며

$$f_\theta(x)=\sum_{m=1}^{M}\theta_mh_m(x)$$

$h_m$ 은 입력 벡터의 함수, linear term은 모수 $\theta$ 의 형태에 의해 결정된다.

이러한 방법론은 많은 다양한 방법론들을 다룰 수 있다. 어떤 경우 basis function들의 sequence가 미리 기술된다. (basis for polynomials in $x$ of total degree $M$)

> neural net context에서 다뤄지는 듯? 

## Model Selection and the Bias-Variane Tradeoff

모든 모델들은 **smoothing** 또는 **complexity** 모수가 결정되어야 한다.

* penalty term의 승수

* kernel의 너비

* basis function의 갯수

> Underfit = Large Bias, Small Variance, Overfit = Small Bias, Large Variance


