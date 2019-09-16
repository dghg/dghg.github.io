---
layout: post
title: "GoogLeNet(Inception v1)을 알아보자"
date: 2019-09-12 00:00:00
excerpt: "이 글은 GoogLeNet을 다룬 논문인 Going Deeper with Convolutions를 읽고 정리한 글입니다."  
tags:
- Study
categories:
- 공부
---

## Table of Contents 
1. [Introduction](#intro)
2. [Motivation](#motiv) 
3. [Architectural Design](#des)

### 0. 도입
*GoogLeNet*은 이름에서 알 수 있듯이 google에서 만든 모델이고, ILSVRC 2014 classification에서 가장 좋은 정확도를 달성했습니다.

### 1. Introduction<a name="intro"></a>
최근 *Deep Learning*과 *Convolutional Network*의 발전으로 Object Classification 분야의 성과가 놀랍도록 향상되었습니다.  
여기서 살펴볼 *GoogLeNet*의 경우 2012년의 *AlexNet*보다 12배 적은 파라미터 수로 더 정확한 결과를 보여줬습니다. *GoogLeNet*는 정확도도 중요하지만, 그것보다 **계산의 효율**을 중심으로 설계된 모델입니다. 최근 모바일과 임베디드 분야의 발전으로, 메모리 사용의 효율성이 중요해 졌습니다. 이를 위해 *GoogLeNet*는 **Inception**이라는 구조를 활용해 계산의 효율성을 높였습니다.

### 2. Motivation<a name="motiv"></a>
모델의 성능을 높이기 위해서 가장 좋은 방법은, 모델을 Deep하게, 즉 Width와 Depth를 늘리는 것입니다. 하지만 이런 방법은 두가지 문제를 안고 있습니다.  
##### 1. Overfitting에 취약하다
제한된 학습 데이터 안에서 모델의 파라미터 수가 많아질수록, 모델은 Overfitting에 취약하게 됩니다. 파라미터의 수를 줄이기 위해서 *Fully Connected Layer*를 없애고, *1x1 Filter*를 사용하게 됩니다.
##### 2. 계산량이 매우 많다
딥러닝 모델의 경우 sparsity가 높아야 성능이 좋은 경우가 많습니다. 즉 필요없는 연산을 하는 경우가 많은데, 이를 줄여야 합니다. 그러나 실제로는 dense matrix 연산의 경우가 훨씬 빠르다고 합니다. 이런 점을 활용한 구조가 바로 **Inception** 구조입니다.  
  
이러한 문제를 해결하기 위해 *GoogLeNet*은 다음과 같은 방식들을 사용하였습니다.

### 3. Architectural Design<a name="des"></a>
  
##### 1. 1x1 Convolution
NiN(Network in Network) 에서처럼 1x1 Convolution을 사용하였습니다. *GoogLeNet*에서 이런 Convolution을 사용한 이유는, 계산량을 줄이기 위해서입니다.  

![Without 1x1](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/1-google.PNG)  
  
  
첫번째로 1x1 Convolution을 사용하지 않은 경우입니다. 48개의 5x5 convolution을 사용해 연산을 수행했을 떄, 계산량은 (14×14×48)×(5×5×480) = **112.9M**이 됩니다.  

![With 1x1](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/2-google.PNG)  
  
반대로 1x1 Convolution을 사용했을때, 총 계산량은 **5.3M**으로 사용하지 않았을 때와 큰 차이가 발생합니다.



### References
1. [Going Deeper With Convolutions](https://www.cv-foundation.org/openaccess/content_cvpr_2015/papers/Szegedy_Going_Deeper_With_2015_CVPR_paper.pdf)  
2. [Review : GoogLeNet(Inception v1)](https://medium.com/coinmonks/paper-review-of-googlenet-inception-v1-winner-of-ilsvlc-2014-image-classification-c2b3565a64e7)