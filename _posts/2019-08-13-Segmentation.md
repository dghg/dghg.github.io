---
layout: post
title: "[CS231N] Detection and Segmentation"
date: 2019-08-13 00:00:00
excerpt: "이 글은 CS231n 11강을 정리한 것입니다. "  
tags:
- CS231N
- 딥러닝
- CNN
- 공부
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

### 2. Classification + Localization<a name="classf"></a>
### 3. Object Detection<a name="obj"></a>
### 4. Instance Segmentation<a name="inst"></a>