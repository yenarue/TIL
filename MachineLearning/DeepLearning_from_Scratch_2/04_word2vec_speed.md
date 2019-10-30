word2vec 속도 개선
====

앞서 익힌 word2vec 모델 중 하나인 CBOW는 어휘량이 많아질수록 계산량이 크게 늘어난다는 단점을 가지고 있다.

이제부터는 속도를 개선시킬 수 있는 방법인 **'Embedding'** 과 **'네거티브 샘플링'** 에 대해 알아보겠다.

## CBOW 의 문제점 살펴보기

![](./images/fig 4-2.png)

위 그림에서 예상 가능하듯, 아래 두 부분에서 계산 병목현상이 발생한다:

* 원핫표현으로 이루어진 입력층과 가중치 행렬 Win의 곱 계산
  * 어휘수가 많아질 수록 원핫 벡터의 크기도 커진다.
  * Embedding 계층 도입으로 해결
* 은닉층과 가중치 행렬 Wout의 곱 및 Softmax 계층의 계산
  * 네거티브 샘플링으로 해결

## 개선하기 - 1. Embedding 계층

먼저 기존 입력층과 가중치 행렬의 연산을 살펴보자:

![](./images/fig 4-3.png)

정말 거대한 연산을 진행하지만, 자세히 살펴보면, 이 연산은 결국 **행렬의 특정 행을 추출해내는 연산**일 뿐이다. 즉, 원핫 표현으로의 변환 & MatMul 계층의 행렬곱 계산이 굳이 필요없다. 그냥 바로 인덱스로 접근하면 훨씬 성능이 개선될 것임을 알 수 있다.

Embedding 계층은 **가중치 매개변수(행렬)로 부터 특정 단어(단어 ID)에 해당하는 행(벡터)를 추출하는 계층**을 뜻한다. 즉, 단어의 분산표현이 저장되는 계층이다.

> Embedding : word embedding. 단어의 밀집벡터 표현. 단어의 분산표현.

그럼, 행렬에서 특정 행을 추출해내는 코드를 구현해보자:

```python
>>> import numpy as np
>>> W = np.arange(21).reshape(7, 3)
>>> W
array([[ 0,  1,  2],
       [ 3,  4,  5],
       [ 6,  7,  8],
       [ 9, 10, 11],
       [12, 13, 14],
       [15, 16, 17],
       [18, 19, 20]])
# 특정 행 추출하기
>>> W[2]
array([6, 7, 8])
>>> W[5]
array([15, 16, 17])

# 여러 행을 한꺼번에 추출하기
>>> idx = np.array([1, 0, 3, 0])
>>> W[idx]
array([[ 3,  4,  5],
       [ 0,  1,  2],
       [ 9, 10, 11],
       [ 0,  1,  2]])
```

위와 같이 이전과 같은 복잡한 연산 없이 간단한 인덱싱 접근으로 빠르게 추출되는 것을 알 수 있다.

### Embedding 계층 구현하기

그럼 이제 Embedding 계층을 구현해보자!

![](./images/fig 4-4.png)

위에서 살펴본 연산이 바로 Embedding 계층의 순전파이다. 이는 행렬의 특정 행만 뽑아낸 것으로서 아무런 변형 없이 그대로 값을 흘려보낸 행위이다. 그러므로 이의 역전파도 그냥 전해진 기울기를 그대로 다음층으로 흘려보내주면 된다.

이를 코드로 옮겨보면 다음과 같다 : 

```python
class Embedding:
    def __init__(self, W):
        self.params = [W]
        self.grads = [np.zeros_like(W)]
        self.idx = None
        
    def forward(self, idx):
        W, = self.params
        self.idx = idx
        out = W[idx]
        return out
    
    def backward(self, dout):
        dW, = self.grads
        dW[...] = 0			# dW의 shape는 유지하고 내용물을 0으로 채워넣는다
        dW[self.idx] = dout	# dout = 앞층에서 전해져온 기울기
        return None
```

그런데 현재 역전파 구현에 문제가 있다. 인덱스에 중복된 값이 들어올 경우에 이전의 dout이 마지막으로 나타난 값의 dout으로 덮어씌워진다는 점이다.

![](./images/fig 4-5.png)

덮어씌워지는 문제를 해결하기 위해서는 단순 할당을 더해지는 연산으로 변경해야 한다:

```python
    def backward(self, dout):
        dW, = self.grads
        dW[...] = 0
        
        # np.add.at(dW, self.idx, dout) 가 더 빠르지만 원리를 보여주기 위해..
        for i, word_id in enumerate(self.idx):
            dW[word_id] += dout[i]
            
        return None
```

구현 끝! 이제 입력층의 MatMul 계층을 Embedding 계층으로 변경해주면 효율적인 메모리 사용 및 계산 최적화가 가능해진다! :-D

## 개선하기 - 2. 네거티브 샘플링 (부정적 샘플링)