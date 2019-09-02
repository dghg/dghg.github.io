---
layout: post
title: "[CS231N] Visualizing and Understanding"
date: 2019-08-19 00:00:00
excerpt: "이 글은 CS231n 12강을 정리한 것입니다. "  
tags:
- CS231N
categories:
- 공부
---
## Table of Contents
1. [Visualizing CNN](#visual)

### 0. 도입
CS231N 12강에서는 전에 배웠던 Convolution 네트워크의 Layer들이 어떤 특성을 가지는지, 또한 각 Layer들을 visualizing 하면 어떻게 되는지 등을 공부합니다.  
  
    
### 1. Visualizing CNN<a name="visual"></a>
CNN의 공통적인 특성은 바로 Input이 raw pixel 이고, output이 labeling score라는 점입니다. 이 외에는 각 모델마다 사용하는 filter size도 다르고, layer의 갯수도 다 다르기 때문에 일반화해서 분석하기는 힘듭니다.
  
##### 1. 1st Layer 
첫번째로, Input과 바로 연결된 1st layer를 분석했습니다. Alexnet, ResNet 등 다양한 종류의 모델이 공통적으로 가지는 특성은 다음과 같습니다.  
![first-layer](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/1-visual.PNG)
모든 모델에서 공통적으로 edge와 color 등이 표현됩니다. 이것은 사람이 객체를 인지할 때 visual system의 행동과 비슷하다고 합니다.

##### 2. 2nd, 3rd Layer
두번째로, 1st Layer와 바로 연결된 2nd, 3rd Layer를 분석했습니다. 하지만 visual적으로 유의미한 결과는 얻지 못하였습니다. 

##### 3. Last Layer(FC Layer)
세번째로 마지막 layer인 FC Layer를 분석해 보았습니다. 여기서는 Alexnet 모델과, Nearest Neighbor 방식을 이용하여 분석을 진행하였습니다.  
Alexnet의 last layer는 4096의 feature-vector로 구성되어 있습니다. 이 값들을 5개의 nearest-neighbor로 정렬 하면 다음과 같은 결과가 나옵니다.  
![last-layer](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/2-visual.PNG)
공통적으로 찾을 수 있는 점은, Nearest Neighbor인 데이터들이 모두 같은 label을 가지고 있다는 것입니다. 여기서 흥미로운 점은, 코끼리의 경우 test-image와 3rd nearest negihbor 의 이미지를 비교해 보면, 코끼리의 서있는 방향도 다르고 pixel값도 매우 다르게 보입니다. 하지만  모델은 이 둘을 nearest-neighbor로 취급합니다. 이를 통해 last layer의 값들이 유의미하다는 것을 알 수 있습니다.  
  
Last Layer를 분석하는 두번째 방법으로, Dimensionality Reduction을 사용하였습니다. 이를 통해 4096개의 vector를 2D안에 표현할 수 있게 됩니다. 여기서는 reduction 방법으로 t-SNE를 사용하였습니다.  
이를 이용해 이미지들을 2D 좌표계에 표현한 결과, 다음과 같은 결과가 나왔습니다. 
![last-layer2](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/3-visual.PNG)
왼쪽 아래에 초록색의 이미지들이 몰려있고 이미지들을 clustering 해줍니다.  

  
  
##### 4. Visualzing Activation Map
activation map을 분석하는 경우 몇몇 case에서는 유의미한 결과가 나타납니다. 여기선 예시로 Alexnet을 사용합니다.  
예를 들어, Alexnet의 conv5의 activation map은 $$ 128 \times 13 \ times 13 $$ 이고 이것의 feature-map들을 살펴보다보면, 사람의 얼굴과 비슷한 뉴런이 보이기도 합니다. 

##### 5. Maximally Activating patches
위와 마찬가지로, 128개의 $$ 13 \ times 13 $$ activation map중 임의로 선택한 channel을 관찰합니다.( 여기서는 17th channel)  
이후 이미지를 네트워크에 넣은 뒤, 해당 channel의 activation값이 큰 순서대로 visualizing 합니다. 이를 보면 해당 뉴런이 찾고 있는 것을 알 수 있습니다. 
![maximally-activating](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/4-visual.PNG)  

##### 6. Occlusion Experiments
이번 실험에서는, 이미지의 어떤 부분이 output을 결정하는데 중요한 역할을 하는지 알아보는 실험입니다. 이를 위해 이미지의 특정 부분을 block하고, 이에 대한 output의 확률 분포를 관찰합니다.  
![occlusion](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/5-visual.PNG)  