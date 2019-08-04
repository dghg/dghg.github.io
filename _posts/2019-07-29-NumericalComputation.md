---
layout: post
title: "Numerical Computation"
date: 2019-07-28 00:00:00
excerpt: "이 글은 Deep Learning Book 4장을 정리한 것입니다. "  
tags:
- ML
- DEEPLearning
categories:
- 공부
---


 

Machine Learning 알고리즘은 매우 많은 계산량을 필요로 합니다. 근본적인 이유는 함수를 최소화&최대화 하는 값들을 찾기 위해 analytically 한 방법보다는 Iterative한 과정을 통해 찾기 때문입니다. 이러한 과정을 제한된 메모리의 컴퓨터로 수행하기 때문에, 필연적으로 수를 정확히 표현하지 못하게 되고, 각종 오류(Overflow 등)이 발생하게 됩니다

### 1. Overflow and Underflow
근본적인 문제는 제한된 bit로 실수(real number)를 표현하기 때문에 발생합니다.  