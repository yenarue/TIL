신경망 학습
====

## 학습
훈련 데이터로부터 **가중치 매개변수의 최적값을 자동으로 획득**하는 것

- 목표 : 손실함수의 결과값을 작게 만드는 가중치 매개변수를 찾는 것!
- 손실함수 : 신경망이 학습을 할 수 있도록 하는 지표.  



### 데이터 주도 학습

사람의 개입없이 데이터만으로 규칙을 찾아내는 것!

특징(feature)을 추출한다 -> 특징의 패턴을 기계학습으로 학습한다

- 특징 (feature) : 입력 데이터에서 본질적인 중요한 데이터를 뽑아낸 것



## 기계학습과 딥러닝의 차이

![](/Users/yenarue/OneDrive/Developer/TIL/DataScience/images/human_to_machine.png)

- 기계학습 : 특징을 추출한 데이터에서 규칙을 찾아내는 것은 기계가 하지만 **특징을 정의하는 것은 사람이 직접 설계해야 한다. (중요)**
- 딥러닝 : 그 특징까지도 기계가 스스로 학습한다! (=종단간 기계학습, end-to-end machine learning)



## 훈련 데이터와 시험 데이터

훈련 데이터에 포함 되어있지 않는 데이터도 잘 풀어내는지 (**범용능력**) 확인하기 위해 데이터셋을 다음과 같이 나눈다.

- 훈련 데이터 (training data) : 이 데이터만 사용하여 학습하면서 최적의 매개변수를 찾는다
- 시험 데이터 (test data) : 그 이후 시험 데이터를 사용하여 위에서 훈련된 모델의 성능을 평가한다

### 오버피팅 (overfitting)

훈련 데이터가 지역성을 가지는 경우. 학습이 특정 데이터셋에만 지나치게 최적화된 상태.

### 딥러닝의 최종목표

**범용 능력을 획득**하고 **오버피팅을 지양**한다



## 손실 함수 (loss function)

**신경망 성능의 나쁨을 나타내는 지표.** 이 지표를 기준으로 최적의 매개변수 값을 찾는다.

E : error

y : output

t : training data

### 손실함수의 종류 - 평균 제곱 오차 (mean squared error, MSE)

![](/Users/yenarue/OneDrive/Developer/TIL/DataScience/images/mse.png)

손실함수의 결과 값이 클수록 오차가 크다 = 학습이 잘 되지 않았다는 뜻

```python
def mean_squared_error(y, t):
    return 0.5 * np.sum((y - t) ** 2)

def main():
    t = [0, 0, 1, 0, 0, 0, 0, 0, 0, 0] # 원-핫 인코딩
    
    # 정답을 추측한 경우 (인덱스 2 : 0.6)
	y = [0.1, 0.05, 0.6, 0.0, 0.05, 0.1, 0.0, 0.1, 0.0, 0.0]
	print(mean_squared_error(np.array(y), np.array(t)))       ## 0.09750000000000003
    
    # 오답의 경우 (인덱스 2 : 0.1)
	y = [0.1, 0.05, 0.1, 0.0, 0.05, 0.1, 0.0, 0.6, 0.0, 0.0]
	print(mean_squared_error(np.array(y), np.array(t)))       ## 0.5975
```



### 손실함수의 종류 - 교차 엔트로피 오차 (cross entropy error, CEE)

![](/Users/yenarue/OneDrive/Developer/TIL/DataScience/images/cee.png)

손실함수의 결과 값이 0에서 멀어질수록 오차가 크다.

- 자연로그 (밑이 e인 log)

![자연로그](https://amsi.org.au/ESA_Senior_Years/imageSenior/3h_3.png)

위의 자연로그 그래프에서 볼 수 있듯이, x가 1인 경우 y가 1이다.

CEE는 자연로그를 사용하여, 정답일 확률 값 y가 커질수록 0에 가까운 결과 값이 나온다. (y=1일때는 값이 0이 나온다.) 반대로 정답일 확률 값(y)이 작아질수록 오차가 급감한다.

```python
def cross_entropy_error(y, t):
    delta = 1e-7 # np.log(x)의 매개변수 x가 0이 되지 않게 하기 위함. (마이너스 무한대(-inf)방지)
    return -np.sum(t * np.log(y + delta))

def main():
    t = [0, 0, 1, 0, 0, 0, 0, 0, 0, 0]
    
    # 정답을 제대로 추측한 경우 (인덱스 2 : 0.6)
	y = [0.1, 0.05, 0.6, 0.0, 0.05, 0.1, 0.0, 0.1, 0.0, 0.0]
	print(cross_entropy_error(np.array(y), np.array(t)))       ## 0.510825457099338
    
    # 오답의 경우 (인덱스 2 : 0.1)
	y = [0.1, 0.05, 0.1, 0.0, 0.05, 0.1, 0.0, 0.6, 0.0, 0.0]
	print(cross_entropy_error(np.array(y), np.array(t)))       ## 2.302584092994546
    
    # 오답의 경우2 (인덱스 2 : 0.2)
	y = [0, 0.05, 0.2, 0.0, 0.05, 0.1, 0.0, 0.6, 0.0, 0.0]
	print(cross_entropy_error(np.array(y), np.array(t)))		## 1.6094374124342252

    # 오답의 경우3 (인덱스 2 : 0.05)
	y = [0.15, 0.05, 0.05, 0.0, 0.05, 0.1, 0.0, 0.6, 0.0, 0.0]
	print(cross_entropy_error(np.array(y), np.array(t)))		## 2.9957302735559908
```

정답인 y 인덱스 2의 값(=인덱스 2가 정답일 확률)에 따라 CEE의 결과 값이 결정되는 것을 알 수 있다.



### [MSE vs CEE](http://fbsight.com/t/loss-function/114616)

- MSE
  - Regression 에 주로 사용된다.
  - [Gradient Vanishing](https://www.youtube.com/watch?v=Y1Z7YWr5PQE) 문제로 학습속도가 느리다. [#1](https://www.youtube.com/watch?v=QAhlSzRLSos) [#2](https://www.youtube.com/watch?v=SKMpmAOUa2Q)
- CEE
  - 시그모이드나 소프트맥스 함수를 사용하는 Classification 시에는 학습속도 측면에서 더 빠르다.
  - 확률을 통하여 계산하기 때문에 분류, 원핫인코딩 처럼 Output을 확률값으로 줄 수 있다면 CEE를 사용하면 좋다. (그 반대는 MSE)



## 미니배치