Coursera의 유명 강의, Andrew Ng 교수님의 Machine Learning 를 들으며 정리한 개인 학습자료입니다.



# W1-2. Supervised Learning - Linear Regression with one variable

지도학습 (Supervised Learning) 중의 하나인 선형 회귀 (Linear Regression) 에 대해서 알아보자.

# Parameter Learning

이제 자동적으로 파라미터(세타0,1)와 비용함수 J의 최소값을 찾아주는 알고리즘에 대해 알아보자.

## Gradient Descent (기울기 하강)

**함수의 최소값을 구하는 반복적 알고리즘**

선형회귀에서만 사용되는 알고리즘은 아니다. 기계학습의 모든 곳에서 사용됨. 함수들의 최소값을 구할 때 사용.

여기서는 비용함수 J의 최소값을 구하기 위해 기울기 하강 알고리즘을 사용할 예정



세타0, 세타1에 초기값(일반적으로는 (0,0)을 셋팅하고 세타0과 세타1을 조금씩 바꿔가면서 J(세타0, 세타1)을 줄여나간다.



아래 공식을 수렴할 때 까지 반복하면 세타0, 세타1을 업데이트 시킨다.

<공식 스샷>

* `:=` assignment (할당 기호)

* `a := b` b의 값을 a에 덮어씌운다 (assign)

* `a = b` a와 b는 같다 (Truth Assertion, boolean)

* alpha : learning rate (학습률, 훈련비율)
  * 얼마나 큰 걸음으로 내려갈 것인가
  * 늘 양수 (음수가 되면 최소값에서 멀어짐 = Gradient Descent 가 하려는 일을 반대로 하게 됨 => 노답 - [출처](https://towardsdatascience.com/frequently-asked-questions-on-learning-rate-6defb4e45d2e))

* 미분계수

세타0과 세타1은 동시에 대입된다.

왜 동시에 바꿔야 하는가? 둘중 하나가 먼저 업데이트되면 나머지 하나를 계산할 때 업데이트된 값이 대입되어버리기 때문이다.

## Deep Dive to Gradient Descent

앞서 살펴본 파라미터를 업데이트하는 공식에서 학습률(Learning Rate)과 미분계수(Derivative Term) 부분을 자세히 살펴보자.

쉽고 간단하게 설명하기 위해 세타1만 가지고 함수를 최소화시켜보겠다.

### 미분계수

그 점의 탄젠트 값을 구하는 것

함수의 탄젠트 값 = 빨간색 선의 기울기 값 = 미분계수



예시를 보자: 

기울기가 양수값 = 미분계수가 양수

업데이트될 세타1 =  (세타1 - 알파 * 양수)

세타1가 왼쪽으로 이동하며 최소값을 향해 다가간다



반대쪽에서 다가가는 예시를 보자 :

기울기가 음수값 (negative slope) = 미분계수가 음수

업데이트될 세타1 = 세타 - 알파 * 음수

세타가 오른쪽으로 이동하며 최소값을 향해 다가간다



### Learning Rate (학습률)

학습률 alpha가 너무 작거나 너무 크면 어떻게 될까?

* 너무 작으면 : 너무 조금씩 움직이기 때문에 연산을 자주 해야 하고 너무 느림

* 너무 크면 :  너무 크게 움직이면 계속해서 최소값에서 멀어지게 됨. 방향 전환 자체를 실패하게 될 수 있음.

파라미터가 지역 최소값이면 기울기(slope)가 0이되어 세타1이 변하지 않게 된다.



그렇다면 학습률을 매 턴마다 조정해야할까?

그래프 참고) 미분계수가 매우 가파르다가 턴이 진행될수록 기울기가 덜 가파르게 된다.

지역 최소값에 가까워질수록 더 작은 거리를 이동하게 된다. 최소값에 가까워진다는 것은 미분계수가 0이 되는 것을 의미하기 때문이다. 지역최소값에 가까워질수록 미분계수도 작아지고 하강기울기는 더 조금씩 이동하게 된다.-> 학습률 (alpha)의 값을 줄일 필요가 없다.



## Gradient Descent For Linear Regression

그럼 이제 선형회귀에서 기울기 하강(Gradient Descent)이 사용되는 경우에 대해서 알아보도록 하자. 

앞서 이야기했듯이 Gradient Descent은 함수의 결과값을 최소로 만들기 위해 사용하는 알고리즘이다. 선형회귀의 핵심은 비용함수 J의 값이 최소가 되는 파라미터를 구하는 것이므로 이 경우에 Gradient Descent가 아주 유용하게 사용된다.



<세타0, 세타1에 대한 Gradient Descent 알고리즘 공식>

여기서도 동시에 넣어줘야 함



<Gradient Descent를 적용하는 그래프 예시와 비용함수 그래프 예시 비교>

왼쪽 그래프에서 Gradient Descent 를 적용하게되면 지역 최소값을 찾아 가게됨

하지만 오른쪽의 선형회귀의 비용함수의 그래프는 항상 아래쪽으로 볼록한 함수임

=> 지역 최소값이 존재하지 않고 전역 최소값만 존재함 => Gradient Descent 적용시 언제나 전역 최소값을 찾게됨 => 만세!



예시를 살펴보자

초기값 : 세타0 = 900, 세타1 = -0.1

h(x) = 900 - 0.1x

전역적 최소값이 가설과 일치할 때 자료와 잘 일치하게 된다.



이것이 바로 기울기 하강이다. 예측하는데 사용할 수 있게 되었다 :-) 

지금까지 살펴본 기울기 하강 알고리즘의 또 다른 명칭은 Batch Gradient Descent 이다.



### 집단 기울기 하강 (Batch Gradient Descent)

기계학습에서 쓰는 용어.

* Batch : Each step of gradient descent uses all the training examples
* This method looks at every example in the entire training set on every step, and is called **batch gradient descent**.

가끔 하강 기울기가 집단의 값과 다를때도 있다. 전체적인 훈련집단을 보지 말고 훈련집합의 작은 부분직합을 보도록 하자.



### 비용함수 J의 최소값을 구하는 또 다른 방법 : 반복 최소 이승법

하강기울기 반복없이 비용함수 J의 최소값을 구하는 방법 => 반복 최소 이승법

=> 자료의 범위나 크기가 너무 거대해서 하강 기울기 알고리즘을 사용할 수 없을 때 사용

