---
layout: post
title: "Batch Normalization(2015)"
date: 2019-07-29 00:00:00
excerpt: "Batch Normalization 정리와 구현"  
tags:
- Study
categories:
- 공부
---
## Table of Contents 
1. [Abstract](#abstract)
2. [Toward Reducing Internal Covariate Shift](#reducing)
3. [Batch Normalization Approach](#appr)
4. [BN In CNN](#cnn)
5. [Effect of BN](#effect)
6. [Experiment](#exp)


### 1. Abstract <a name="abstract"></a>
Deep한 모델을 학습할 때 , 이전 Layer들의 parameter들이 업데이트 됨에 따라 Input의 분포가 계속 변화할 수 있습니다. 이러한 문제는 특정 구간에서 미분값이 0에 가까워지는 활성함수(Sigmoid 등)를 쓸 때 Gradient vanishing 현상으로 인해 training 속도를 늦출 수 있습니다. *Batch Normalization* 방법은 입력의 분포를 normalize 함으로써 regularizer, Dropout 등의 의존도를 낮출 수 있습니다.

#### Internal Covariate shift
이전 Layer들의 parameter 변화로 Input 분포가 바뀌는 것을 여기선 **Internal Covariate shift** 라고 부릅니다.  
이러한 현상은 Gradient Vanishing&Exploding의 원인이 되어, 학습속도를 늦출 수 있습니다. 기존에는 ReLU(x) = max(0,x)나 careful initialization( Bengio & Glorot, 2010; Saxe et al. 2013), 작은 learning rate를 사용함으로써 막을 수 있었는데, 이 논문에서는 Batch Normalization이라는 새로운 방법을 제시합니다.  

### 2. Towards Reducing Internal Covariate Shift<a name="reducing"></a>
Internal Covariate Shift를 막는 방법중 하나는, Input을 mean=0, variance=1를 가지는 정규분포로 정규화함으로써 해결할 수 있고, 이를 "Whitening"이라 부릅니다. 이러한 방식을 사용함으로써 Input의 분포를 고정할 수 있고 Internal Covariate Shift를 막을 수 있습니다.  
그러나, 이러한 whitening 과정은 최적화 과정과 무관하게 진행됨에 따라, 특정 parameter를 계속해서 증가시킬 수 있습니다. 논문에서는 이러한 결과를 실험으로 관찰하였습니다.  
예를 들어, Input이 u고 paramter가 b인 레이어 **x = u + b**를 zero-centered 되게 Normalize( **x = x- E(x)** )하고 parameter를 업데이트 하는 과정을 살펴보면,  
b' = b + ∆b 이고 **x' = u + b - E(u + b + ∆b) = u + b - E(u+b) **로 output이 업데이트 전과 달라진게 없게 됩니다. 이러한 방식은 b가 무한하게 증가하는 방향으로 학습이 진행되게 됩니다.  
이런 현상이 발생하는 이유는 바로 Backpropagation 과정에서 Normalization이 존재한다는 것을 반영하지 않기 때문입니다. Backpropagation 과정에서 Normalization 과정까지 고려한다면, whitening 방식은 계산량에 있어 very-expensive 하기 때문에 이를 더욱 더 간단하게 한 새로운 방법을 찾았습니다.

### 3. Batch Normalization Approach<a name="appr"></a>
Batch Normalization 레이어는 기존 d-dimension 데이터를 1) mean 0, variance 1로 Normalize 하고, 이렇게 normalize한 데이터의 출력은 직선에 가까워질 수 있으므로, 2) 새로운 parameter를 이용해(scaling, shifting) mapping 시킵니다.
![ALG1](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/3.PNG)   
이러한 방식을 사용한다면, BN layer는 모두 미분가능하므로 chain-rule을 사용할 수 있습니다. 
![CHRULE](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/4.PNG)  

### 4. Training & Inference With BN
Training 할 때는 자연스럽게 Mini-batch의 평균과 분산을 이용해 normalize하지만, Infenrence의 경우에는 다른 방법을 사용합니다.  
이때는 다음과 같이이동평균(moving-average)과 unbiased-estimator인 표본분산을 사용해 이 값으로 Normalize 합니다.  
![INFwBN](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/5.PNG) 

### 5. Batch Normalization In CNN<a name="cnn"></a>
일반적인 Network와 달리 CNN에서는 다른 방법을 이용해야 합니다.  
Network의 출력인 z 는 **z = g(Wu+b)** 처럼 표현됩니다. 하지만 CNN에서 BN을 사용할 경우 b 값이 shifting factor 에 의해 대체될 수 있으므로, b 를 없애고 **z = g(BN(Wu))** 형태로 사용합니다  
또한, Convolution 성질을 유지시키기 위해 각 channel 마다 mean과 variance를 구해 Batch Normalization을 적용합니다.   
예를 들어 Channel이 m이고 feature map이 p\*q인 경우 batch-size는 m\*p\*q가 되고, 이에 대해 mean과 variance를 구한 후 channel size인 m개에 대해 scaling, shifting 변수를 학습합니다.

### 6. Effect of Batch Normalization<a name="effect"></a>
*첫번째로, 높은 learning rate를 사용할 수 있습니다.*    
  
높은 learning rate는 parameter들의 scale을 증가시키는 경향이 있고, 큰 parmeter scale은 학습 시 gradient가 explosion/vanishing 하게 됩니다. 반면에 Batch Normalization을 적용하면 parameter의 scale에 영향을 받지 않게 됩니다. **BN(Wu) = BN((aW)u)** 따라서 학습 과정을 안정화하고, 빠른 학습을 가능하게 합니다.

*두번째로, model을 regularize 할 수 있습니다.*  
  
자체적인 regularizer 효과로, 기존에 사용하던 기법인 weight decay나 Dropout 없이도 overfitting을 막을 수 있습니다.

### 7. Experiment<a name="exp"></a>
간단하게 MNIST 데이터셋을 이용하여 실험 해보았습니다. 아무런 전처리 없이  
공통적으로 128의 Batch Size, 30번의 EPOCH, weight 초기화의 경우 mean=0, variance=0.01의 정규분포로 초기화하였습니다.  
BN을 적용하지 않은 Network의 경우,   
 1) 2개의 Convolution Layer(32개의 3\*3, 64개의 3\*3)와 2개의 2 by 2 Maxpooling,  
   2개의 FC Layer(크기 1024와 10) 1개의 Dropout layer  
  
BN을 적용한 Network는 위에서 Dropout Layer만 제거한 이후  
1) 0.0025의 learning rate로 학습  
2) 0.01의 learning rate로 학습  
과정을 거쳤습니다. 

간단하게 30개의 EPOCH을 거친 후 Test Set에서의 Loss 결과는 다음과 같습니다.  
![INFwBN](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/8.PNG) 
이를 통해, 단지 normalization만 해줬을 뿐이지만 모델의 성능이 향상 된 것을 알 수 있습니다. 이외에도 weight 초기값을 mean=10, variance=1로 초기화하고 학습을 진행했을 때도 LOSS값이 기존 네트워크와 비슷하게 수렴합니다. 이를 통해 normalize만 해줬을 뿐이지만 네트워크 성능이 크게 향상된 것을 알 수 있습니다.  
  
### 8. Conclusion
요약하자면, Batch Normalization 기법은  
**Input의 분포를 안정화 시키기 위해,(Internal Covariate Shift를 줄이기 위해)  
1) 네트워크에 들어가는 Input을 Normalize 해준다.  
2) 여기다 scaling/shifting factor를 학습시킨다.**  
로 정리할 수 있습니다.  
  
이를 통해 learning rate를 높일 수 있고, Dropout같은 regularizer의 의존도를 줄여 빠른 모델 학습이 가능하게 합니다. 