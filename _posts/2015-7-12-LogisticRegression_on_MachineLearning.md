---
layout: post
category: MachineLearning
title: 로지스틱 회귀(Logistic Regression)
tagline: by Pigbrain
tags: [MachineLearning]
---

<!--more-->

#회귀 ?#
* 두 변수 x와 y와의 관계에 적합한 선
* 회귀가 직선인 경우에는 회귀 직선이라 한다
<img src="/assets/themes/Snail/img/LogisticRegression/regression.png" alt="">  

#로지스틱 회귀#   
**-종속변수가 이항형 문제(유효한 범주의 개수가 두개인 경우)를 분류할 때 사용**

* 장점
	* 계산 비용이 적고 구현이 쉽다
	* 결과 해석을 위한 표현이 쉽다
* 단점
	* 언더피팅(Underfitting)경향이 있어 정확도가 낮게 나올 수 있다 

#배경지식#
* [시그모이드 함수](http://pigbrain.github.io/math/2015/07/10/SigmoidFunction_on_Math/)
  

#분류방법#
 * 각각의 속성에 가중치를 곱한 다음 서로 더한 값을 구한다
 * 더한 값을 시그모이드 함수에 넣고 0에서 1 사이의 수를 구한다
 * 이 수가 0.5 이상이면 1로 분류하고, 0.5이하이면 0으로 분류한다

<img src="/assets/themes/Snail/img/LogisticRegression/logistic_regression.png" alt="">  

#회귀 계수(가중치)를 찾기 위한 방법# 
* 시그모이드 함수의 입력은 z이며 z는 다음과 같이 주어진다
<img src="/assets/themes/Snail/img/LogisticRegression/input_z.png" alt=""> 	
	* 벡터 x는 입력 데이터
	* 가장 좋은 계수 w(가중치)를 찾기 위하여 기울기상승(gradient ascent)이라는 최적화 방법을 이용
* 기울기 상승은 함수에서 최대 지점을 찾고자 할 때, 이 지점으로 이동하는 가장 좋은 방법이 기울기의 방향에 있다는 생각을 기반으로 한다
	* 기울기는 ▽기호를 사용
	* 함수f(x, y)의 기울기는 아래와 같다
<img src="/assets/themes/Snail/img/LogisticRegression/gradient.png" alt="">
	* f(x, y)는 정의가 필요하며 계산이 이뤄지는 지점 근처에서 미분이 가능해야 한다
<img src="/assets/themes/Snail/img/LogisticRegression/gradient_descent.png" alt="">
	* 기울기 상승 알고리즘은 아래와 같이 벡터로 표현 가능
		* 매개변수 a에 의해 크기가 결정
<img src="/assets/themes/Snail/img/LogisticRegression/gradeint_vector.png" alt="">

#용어#
* 오버피팅(Overfitting)과 언더피팅(Underfitting)
	* 훈련 데이터 집합에만 과하게 적합되어 검증 데이터 집합이나 다른 입력에 대해 부정확한 결과를 내는 것을 오버피팅이라 한다
	* 위와 반대로 모호한 결과를 내는 것을 언더피팅이라 한다.
<img src="/assets/themes/Snail/img/LogisticRegression/overfitting_underfitting.png" alt="">

* [기울기 하강](https://ko.wikipedia.org/wiki/%EA%B2%BD%EC%82%AC_%ED%95%98%EA%B0%95%EB%B2%95)

#참고#
* http://terms.naver.com/entry.nhn?docId=2323285&cid=42419&categoryId=42419
* http://blog.secmem.org/647
* https://people.cs.pitt.edu/~milos/courses/cs2710/lectures/Class22.pdf