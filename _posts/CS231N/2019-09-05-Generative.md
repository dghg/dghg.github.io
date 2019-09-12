---
layout: post
title: "[CS231N] Generative Models"
date: 2019-09-05 00:00:00
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
5. [Generative Adversial Network](#gan)
6. [요약](#sum)
  

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
*training data $$p_{data}(x)$$가 주어졌을 때 같은 분포를 보이는 새로운 sample $$p_{model}(x)$$를 생성*
![genearative model](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/1-gen.PNG)  
  
##### Why Generative Model?
Time-series data를 이용한 경우, Simulation이나 Planning, Reinforce Learning에 적용할 수 있습니다.  
또한 Generative Model을 이용하면 데이터의 숨겨진 새로운 feature를 찾을 수도 있습니다.

##### Generative Model의 분류
![genearative model](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/2-gen.PNG)  
  

크게 Explicit density와 Implicit density로 분류합니다.  
첫번째로 Explicit의 경우 명시적으로  $$p_{model}(x)$$를 정의합니다. 강의에서는 예시로 PixelRNN/CNN, Variational AutoEncoder를 다룹니다.  
반대로 Implicit의 경우 반대로 명시적인 정의 없이 $$p_{model}(x)$$를 샘플링합니다.  
강의에서는 예시로 GAN을 다룹니다.  
  
### 3. PixelRNN/CNN<a name="pixel"></a> 

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
VAE는 숨겨진 변수 **z**에 관한 식으로 확률분포함수를 정의합니다.  
바로 직접적으로 optimize할 수 없으므로, 대신 lower bound를 구해 최적화합니다.  
![VAE](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/4-gen.PNG)  
  
##### AutoEncoder
VAE의 원리가 되는 AutoEncoder는, unlabeled 데이터인 $$X$$로부터 $$z$$라는 feature를 뽑아냅니다.(Encoder) 이때 의미있는 feature을 얻기 위해  $$x$$보다 낮은 차원의 $$z$$를 뽑아냅니다.  
이후, Decoder를 이용해 $$z$$로부터 $$x \hat $$을 생성합니다.
![VAE](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/5-gen.PNG)  
  
여기서 AutoEncoder의 Encoder부분은 *Supervised Model*의 초기화에도 사용할 수 있습니다.   
  
VAE도 마찬가지로 Encoder와 Decoder로 구성되어 있습니다.  
첫번째로 Encoder의 역할은, $$x$$가 주어졌을 때 숨겨진 변수인 $$z$$에 대한 확률분포인 $$p_{\phi}(z|x)$$를 찾는 것입니다. 하지만 우리는 이런 확률분포가 무엇인지 모르기 때문에 간단한 확률분포인 **Gaussian Distribution**인 **$$q_{\phi}(z|x)$$** 로 대체합니다.  
  
두번째로 Decoder는, $$z$$가 주어졌을 때 새로운 $$x$$를 만들어 내는것입니다.$$(p_{\theta}(x|z))$$
  
이제 이 두 확률분포를 이용해 Maximum Likelihood를 구합니다.  
![VAE](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/6-gen.PNG)  
  
  
![VAE](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/7-gen.PNG)  


### 5. Generative Adversial Network<a name="gan"></a>
마지막으로 알아볼 GAN은 지금까지 알아본 모델들과 달리 Implicit Density 입니다. 여기서는 2-player의 게임 이론으로 접근합니다.  
  
##### Training GAN : Two-player game
GAN는 *Discriminator, Generator*라는 두개의 네트워크로 구성됩니다.   
*Generator*의 역할은 새로운 이미지를 생성하고, *Discriminator*를 속이는 것입니다. 반대로 *Discriminator*는 진짜 이미지인지, *Generator*에서 생성된 이미지인지 구별하는게 목적입니다. 이 두 네트워크의 상호작용을 통해, 서로 공동으로 학습시킬 수 있습니다.  
![GAN](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/8-gen.PNG) 

GAN은 다음과 같은 minmax 문제를 푸는것으로 학습이 진행됩니다.  
$$D$$는 *Discriminator*, $$G$$는 *Generator*를 의미하고, $$p_{data}(x)$$는 실제 데이터의 확률분포, $$p_{z}(z)$$는 노이즈 샘플링을 통해 생성한 데이터를 의미합니다.  
  
![GAN](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/9-gen.PNG)  
  
여기서 $$D$$라는 함수는, Input으로 들어온 데이터가 진짜이면 1, 아니면 0을 output으로 내놓습니다. 
첫번째로, $$D$$에 관해 함수를 최대화하는 것을 생각할 때, $$logD_{\theta_{d}}(x)$$와 $$log(1-D_{\theta_{d}}(G(z)))$$가 1이 되어야 하므로 $$x$$는 1로, $$G(z)$$는 0으로 학습이 진행되어야 합니다. 이는 *gradient ascent*방법으로 진행할 수 있습니다. 

두번째로, $$G$$에 관해 함수를 최소화하는 것을 생각할 때, 첫번째 항은 관련이 없으니 $$log(1-D_{\theta_{d}}(G(z)))$$를 최소화 해야합니다. 이는 $$D(G(z))$$가 1이 되어야함을 의미하고, 이는 Generator가 Discriminator를 속이는 데이터를 만들어야 함을 의미합니다.  이는 *gradient descent*방법으로 진행할 수 있습니다.  
  
하지만 실제로 이런 방법으로 *Generator*를 optimizing하는것은 *loss landscape*라는 상황 때문에 어렵게 됩니다. 
![GAN](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/10-gen.PNG)  
  
$$D(G(z))$$가 0에 가까울 때(학습이 필요할 때) *gradient*가 0에 가깝고, $$D(G(z))$$가 1에 가까울 때(이미 학습이 완료됐을 때) *gradient*가 커져 학습에 어려움이 있습니다.  
이런 문제를 해결하고자 *Generator*는 다음과 같은 새로운 objective function을 사용합니다.  
![GAN](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/11-gen.PNG) 
  
GAN은 이런 objective 함수를 사용하여 두 네트워크를 번갈아가며 학습시키는데, 이 때 학습하는 비율은 1:1 또는 불균형하게 설정합니다.  

### 6. 요약

### References
1. [CS231N : Lecture 13](https://www.youtube.com/watch?v=5WoItGTWV54&list=PLC1qU-LWwrF64f4QKQT-Vg5Wr4qEE1Zxk&index=14&t=0s)  
2. [Variational AutoEncoder](https://datascienceschool.net/view-notebook/c5248de280a64ae2a96c1d4e690fdf79/)  
3. [논문으로 짚어보는 딥러닝의 맥](https://www.edwith.org/deeplearningchoi/lecture/15846/)
