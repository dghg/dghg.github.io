---
layout: post
title: "[CS231N] Generative Models"
date: 2019-08-19 00:00:00
excerpt: "이 글은 CS231n 13강을 정리한 것입니다. "  
tags:
- CS231N
categories:
- 공부
---

## Table of Contents
1. [Supervised와 Unsupervised learning](#sup)
2. [Generative Model란 무엇인가](#gen)
3. [Pixel RNN/CNN](#pixel)
4. [Variational AutoEncoder](#vae)  
  

### 1. Supervised와 Unsupervised learning<a name="sup"></a>  
  
##### Supervised Learning
cs231n에서 살펴본 대부분의 모델들은 *Supervised Learning*입니다.  
여기서는 Data가 $$(x,y)$$형태로 존재하고 $$y$$는 **label**로 여러 형태를 가질 수 있습니다. ImageNet 에서는 $$y$$가 이미지의 class가 됩니다.  
*Supervised Learning*의 목적은 주어진 $$x$$로부터 $$y$$값을 예측하는 함수 $$f$$를 학습하는 것입니다. 이를 위해 CNN, RNN 등 여러 형태의 모델을 배웠습니다. 
  
  
##### Unsupervised Learning
이제부터 살펴볼 *Unsupervised Learning*의 특징은 바로 Data에 **label**이 존재하지 않는다는 것입니다. 이러한 이유로 학습 데이터를 수집하는게 *Supervised Learning*에 비해 적은 노력을 필요로합니다.  
*Unsupervised Learning*의 목적은 바로 데이터의 **hidden structure**를 찾는 것입니다. 이를 수행하는 예시로는 Clustering, feature learning, density estimation 등이 있습니다.  
  
  
### 2. Generative Model란 무엇인가<a name="gen"></a>  

강의에서는 *Generative Model*을 다음과 같이 설명하고 있습니다.  
*training data $$p_{data}(x)$$가 주어졌을 때 같은 분포를 보이는 새로운 sample $$p_{model}(x)$$를 생성**
![genearative model](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/1-gen.PNG)  
  
##### Why Generative Model?
Time-series data를 이용한 경우, Simulation이나 Planning, Reinforce Learning에 적용할 수 있습니다.  
또한 Generative Model을 이용하면 데이터의 숨겨진 새로운 feature를 찾을 수도 있습니다.

##### Generative Model의 분류
![genearative model](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/2-gen.PNG) 
크게 Explicit density와 Implicit density로 분류합니다.  
첫번째로 Explicit의 경우 명시적으로  $$p_{model}(x)$$를 정의합니다.  
강의에서는 예시로 PixelRNN/CNN, Variational AutoEncoder를 다룹니다.  
반대로 Implicit의 경우 반대로 명시적인 정의 없이 $$p_{model}(x)$$를 샘플링합니다.  
강의에서는 예시로 GAN을 다룹니다.  
  
### 3. PixelRNN/<a name="pixel"></a> 

##### 1) FVBN(Fully Visible Belief Network)
처음으로 살펴볼 FVBN은 Explicit model입니다. chain-rule을 사용하여 크기가 n인 1-d 이미지 $$x$$의 확률을 n개의 곱으로 나타냅니다. 
![FVBN](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/3-gen.PNG) 
이 모델은 n개의 곱으로 나타내기 때문에 매우 느리고 복잡합니다. 이를 해결하기 위해 Neural Network를 도입합니다.  
  
##### 2) PixelRNN
이미지를 upper-left corner에서부터 생성해나갑니다. 이전 pixel과 LSTM을 이용하여 다음 pixel과 dependecy를 유지합니다.  
잘 되지만, 순서대로 데이터를 생성해나가야 하기 때문에 굉장히 느립니다.  
##### 3) PixelCNN
PixelRNN과 마찬가지로 corner부터 이미지를 생성해나갑니다. 여기서는 이전 pixel 여러개와 CNN을 이용하여 다음 pixel과 dependency를 유지합니다.  
training 과정에서 병렬 연산을 수행해 PixelRNN보다는 빠르지만, 이미지 생성과정은 역시 sequential하기 때문에 느립니다.

### 4. Variational AutoEncoder<a name="vae"></a> 