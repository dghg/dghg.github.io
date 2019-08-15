---
layout: post
title: "[CS231N] Detection and Segmentation"
date: 2019-08-13 00:00:00
excerpt: "이 글은 CS231n 11강을 정리한 것입니다. "  
tags:
- CS231N
categories:
- 공부
---
## Table of Contents
1. [Semantic Segmentation](#semantic)
2. [Classification + Localization](#classf)
3. [Object Detection](#obj)
4. [Instance Segmentation](#inst)

### 0. 도입  
CS231N 11강에서는 기존에 공부한 Image-Classification을 이용해, 다양한 Computer Vision Task에 적용하는 방법을 배웁니다. Computer Vision Task에는 모든 Pixel 값을 classify하는 Semantic Segmentation, 분류에 더해 객체에 bounding-box를 적용할 수 있는 Localization, 또한 Single Object가 아닌 여러개의 객체를 판별할 수 있는 Object-Detection과 Instance Segmentation에 대해 공부합니다.  
  
  
### 1. Semantic Segmentation<a name="semantic"></a>
첫번째로 Semantic-Segmentation은, 이미지 내에 존재하는 모든 pixel에 대해 (Background, Sky, Object, ..) 분류하는 것입니다. 첫번째로 도입되는 아이디어는 바로 이미지를 쪼개 일일히 분류하는 Sliding Window 방식입니다.  
#### Sliding Window
  
이미지 내에서 patch들을 추출해 CNN에 넣어 classification을 진행합니다. 하지만 이런 방식은 **1) 계산량이 매우 많다**, **2) overlap되는 patch를 재사용하지 않아 매우 비효율적이다** 라는 단점이 있습니다.  
  
#### Fully CNN    
  
이 방식은 입력이 3\*H\*W이고 score가 C\*H\*W의 사이즈를 가지는 네트워크로 설계하는 것입니다.(C: class의 수 ) 이렇게 설계함으로써 각 pixel 마다 score가 최대인 class로 분류를 하면 됩니다.  
그러나 이런 방식도, 네트워크 내부에서 H\*W 사이즈를 유지하기 위해 드는 계산량이 많아 사용하지 않고, 다음에 등장하는 **Downsampling, Upsampling** 방식의 CNN을 사용합니다.  
  
#### Downsampling, Upsampling
  이전에 배웠던 pooling, stride를 이용하여 이미지 와 레이어들을 Upsampling, Downsampling 합니다.   
downsampling의 경우는 pooling 등을 이용하면 되는데, 여기서 문제는 바로 upsampling을 어떻게 하냐는 것입니다.  
  
##### 방법1. Nearest Neighbor Upsampling
![nn](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/10-seg.PNG)  
사진과 같이, 주변의 값들을 모두 Input 값으로 채웁니다.
  
##### 방법2. bed of nails
![bon](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/11-seg.PNG)
사진과 같이, 모두 0으로 채웁니다.
  
##### 방법3. Max unpooling
![max](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/12-seg.PNG)
Maxpooling 과정에서, 가장 큰 원소의 위치를 저장해둡니다. 이후 unpooling 시 그 위치를 사용해 upsampling을 진행합니다.

##### 방법4. Transpose Convolution
값 1개를 이용해 n개의 값을 얻어낼 수 있게 filter를 사용해  upsampling을 합니다.  
![trans](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/13-seg.PNG)
이렇듯 filter를 이용해 input 값 하나를 여러개로 대응시킬 수 있습니다. ( $$ R \mapsto R^k $$)  
또한 이런 방법을 matrix 관점에서 살펴볼 수 있습니다.  
Input이 $$ [x, y, z] $$ 이고 Filter가 $$ [a, b, c, d] $$,  인 1D Convolution을 생각해 보면, (stride=1) 다음과 같이 나타낼 수 있습니다.  
$$ \\\begin{matrix} 0 $ x & y & z $ 0 \\ 0 & 0 & a $ b $ c \\ 0 & a & b $ c $ d \\a $ b $ c $ d $ 0 \\b $ c $ d $ 0 $ 0 \\ \\\end{matrix} $$  
  
이를 Matrix의 곱셈으로 바라본다면 다음과 같이 나타낼 수 있습니다.

$$
X\vec(a)=
    \begin{pmatrix}  
    x & y & z $ 0 $ 0 $ 0 \\ 
    0 & x & y $ z $ 0 $ 0 \\ 
    0 & 0 & x $ y $ z $ 0 \\ 
    0 $ 0 $ 0 $ x $ y $ z \\
    \end{pmatrix}
    \\\begin{pmatrix}
    0 \\ 
    a \\ 
    b \\ 
    c \\
    d \\
    0 \\
    \\\end{pmatrix}  
    =
    \\\begin{pmatrix}
    ay+bz \\ 
    ax+by+cz \\ 
    bx+cy+dz \\ 
    cx+dy \\
    \\\end{pmatrix}      
$$

  

이러한 방식으로 downsampe, upsample을 통해 모델을 구성하여 Semantic Segmentation을 수행합니다.
![seg](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/14-seg.PNG)  
  
  
    
### 2. Classification + Localization<a name="classf"></a>
Localization은, 객체가 어디에 존재하는지 영역을 bounding box로 나타내는 과정입니다. 기본적인 아이디어는 바로 기존 모델에 bounding box의 좌표를 구하는 과정을 regression 문제로 생각해 추가하는 것입니다. 
![local](https://github.com/dghg/dghg.github.io/raw/master/_posts/img/15-seg.PNG)  
이렇듯 최종 LOSS에 Softmax와 L2 Loss를 weighted sum 한 값을 사용하게 됩니다. 여기서 weight는 하이퍼파라미터로, 어떻게 설정하느냐에 따라 LOSS 값에 큰 영향을 미치게 됩니다. 
  
#### Human Pose Estimation
사람의 특성을 이용한 Human pose estimation에도 사용할 수 있습니다.  

### 3. Object Detection<a name="obj"></a>  
Object Detection은 기존과 달리 image 내에 있는 모든 객체를 찾아내는 것입니다. 모델의 출력으로 각각의 object마다 bounding box를 만들게 됩니다.  
#### As Regression ?  
Localization의 기본 아이디어처럼 각각의 object마다 bounding box를 regression 방식으로 구할 수 있습니다. 하지만 이런 방식은 모델의 입력인 이미지가 매번 달라져 출력의 갯수도 항상 달라진다는 것입니다.

#### Sliding Window
Image를 조각내 CNN의 입력으로 넣어 classification을 진행합니다 .하지만 이런 방식은 계산량이 매우 많습니다.

#### Region Proposals  
물체가 있을만한 Region을 Selective-Search를 이용해 찾아냅니다. 

#### R-CNN  
위에서 소개한 Selective Search 방식으로 물체가 있을만한 부분인 ROI(Regions Of Interests)를 찾아냅니다.(~2K) 이후 CNN에 입력으로 넣기 위해 fixed-size square로 region을 조절합니다.  
이러한 방식은 Sliding-Window 방식에 비해 낫긴 하지만, 학습시간이 오래걸리고 또한 Inference 과정에도 시간이 오래걸립니다.(47s in VGG16)  

#### Fast R-CNN
ROI를 설정하는 대신, 전체 Image를 우선 CNN에 입력으로 넣고, 출력인  Feature Map 에서 ROI를 선정합니다. 
### 4. Instance Segmentation<a name="inst"></a>