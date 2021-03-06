---
layout: post
category: MachineLearning
title: Regularization
tagline: by Pigbrain
tags: [MachineLearning]
---

<!--more-->
  
**-특정 θ의 값을 변경하여 Overfitting을 방지하기 위한 방법** 
<img src="/assets/themes/Snail/img/MachineLearning/Regularization/function.png" alt="">  
<br>
  
# Cost Function    
* 아래와 함수는 Overfitting되어 있다고 가정  
<img src="/assets/themes/Snail/img/MachineLearning/Regularization/function.png" alt="">  
  
* 함수에서 θ3, θ4를 없애지 않고 영향도를 줄이기 위하여 아래와 같이 Cost Funciton을 변경  
	* 임의의 상수를 곱한 θ3, θ4를 더해줌으로써  θ3, θ4가 가져야할 값을 줄일 수 있  
<img src="/assets/themes/Snail/img/MachineLearning/Regularization/costfunction_ex.png" alt="">  

* 특정 θ값이 아닌 전체 θ값들에 위와 같은 방법으로 Regularization 시키기 위해서는 아래의 비용 함수를 이용  
	* λ  
		* Regularization Parameter  
		* θ값들을 얼마나 많이 변경할지 결정하는 변수  
<img src="/assets/themes/Snail/img/MachineLearning/Regularization/costfunction.png" alt="">  
<br>

# Regularization을 적용한 Linear Regression    
* Gradient Descent  
	* θj의 함수에서 빨간색으로 표시된 부분은 항상 1보다 작다  
		* θj의 값은 계산 과정에서 계속해서 줄어든다  
<img src="/assets/themes/Snail/img/MachineLearning/Regularization/linearRegression_GradeintDescent.png" alt="">  

* Normal Equation  
	* L행렬은 (n+1)×(n+1)차원이 되어야 한다
<img src="/assets/themes/Snail/img/MachineLearning/Regularization/linearRegression_NormalEquation.png" alt="">  
<br>

# Regularization을 적용한 Logistic Regression     
* Cost Function  
<img src="/assets/themes/Snail/img/MachineLearning/Regularization/logisticRegression_CostFunction.png" alt="">  
  
* Gradient Descent  
<img src="/assets/themes/Snail/img/MachineLearning/Regularization/logisticRegression_GradeintDescent.png" alt="">  
  
  
# 참고  
* https://en.wikipedia.org/wiki/Regularization_(mathematics)  
* https://share.coursera.org/wiki/index.php/ML:Regularization  