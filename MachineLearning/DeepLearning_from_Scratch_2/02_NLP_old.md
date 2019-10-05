02_자연어와 단어의 분산 표현
====

자연어 처리의 전처리와 딥러닝이 등장하기 이전의 고전적인 방법을 소개한다.

## 자연어 처리 (NLP)

* 자연어 (Natural Language) : 우리가 평소에 쓰는 말
* 자연어 처리 (Natural Language Processing) : 우리의 말을 컴퓨터에게 이해시키기 위한 기술

### 자연어의 특징

* 부드러움, 유연함
  * 같은 의미도 다양한 문장으로 표현 가능
  * 같은 단어도 시대에 따라 뜻이 변하기도 함
  * 새로운 단어가 나타나기도 함

* 단어
  * 의미의 최소 단위

### 단어를 이해시키자
언어의 최소단위라고 볼 수 있는 단어를 이해시키는 것이 중요

* 시소러스(thesaurus, 유의어 사전)를 활용한 기법
* 통계 정보로 부터 단어를 표현하는 통계 기반 기법
* 신경망을 활용한 추론 기반 기법 (word2vec)



## 시소러스

유의어들이 한 그룹으로 분류되어 있는 사전.

![](./images/fig 2-1.png)

자연어 처리 시의 시소러스에는 **단어 사이의 상하 관계**와 **전체/부분 관계** 등의 세부 관계들까지 정의하여 단어들의 관계를 그래프로 표현할 수 있다. 이 단어 네트워크 그래프를 통해 단어 사이의 관계를 컴퓨터에게 가르칠 수 있다.

![](./images/fig 2-2.png)

### [WordNet](https://wordnet.princeton.edu/)

자연어 처리 분야에서 가장 유명한 시소러스.

프린스턴 대학교에서 1985년부터 구축하기 시작한 전통 있는 시소러스.

* 유의어/단어 네트워크
* 단어 사이의 유사도 측정

#### WordNet 맛보기

---추후 진행 후 업데이트하기---

### 문제점

사람이 수작업으로 레이블링하는 방식의 결점이 그대로 존재한다.

* 시대 변화에 대응하기 어렵다 (자연어의 유연함에 대처하기 어려움)

  * > 내 생각 : 그럼 어반딕셔너리의 경우에는..??? WordNet도 위키류 문서들처럼 널리 퍼트리면 해소되는 문제 아닐까?

* 엄청난 인적 리소스 (인건비)

* 단어의 미묘한 차이를 표현할 수 없다

  * 뜻이 비슷한 단어들 끼리 묶다보니 미묘한 차이를 극복하기가 어려움 (의미는 같지만 용법이 다른 경우도 포함)



## 통계 기반 기법

corpus로 부터 '단어의 의미'를 자동으로 추출하는 방법.

* 말뭉치 (corpus, 코퍼스) : 대량의 텍스트 데이터. 특히, 자연어 처리 연구나 앱을 위해 수집된 텍스트 데이터를 뜻한다.

### 1. 말뭉치 전처리하기

텍스트 데이터를 단어로 분할하고 그 분할된 단어들을 단어 ID 목록으로 변환해보자

```python
# 일단 말뭉치대신 연습삼아 한 문장을 전처리 해보자.
>>> text = "You say goodbye and I say hello."
>>> text = text.lower()
>>> text = text.replace('.', ' .')      # 정규표현식을 사용하는 것이 더 좋음
>>> text
'you say goodbye and i say hello .'
>>> words = text.split(' ')
>>> words
['you', 'say', 'goodbye', 'and', 'i', 'say', 'hello', '.']
```

단어를 텍스트 그대로 조작하기에는 불편하므로 ID를 부여해보자

```python
>>> word_to_id = {}
>>> id_to_word = {}
>>> for word in words:
...     if word not in word_to_id:
...             new_id = len(word_to_id)
...             word_to_id[word] = new_id
...             id_to_word[new_id] = word
... 
>>> word_to_id
{'you': 0, 'say': 1, 'goodbye': 2, 'and': 3, 'i': 4, 'hello': 5, '.': 6}
>>> id_to_word
{0: 'you', 1: 'say', 2: 'goodbye', 3: 'and', 4: 'i', 5: 'hello', 6: '.'}
>>> id_to_word[1]
'say'
>>> word_to_id['hello']
5
```

인덱싱 활용을 위해 단어 리스트를 단어 ID 리스트로 변경해보자

```python
>>> import numpy as np
>>> corpus = [word_to_id[w] for w in words]		# 파이썬의 내포(comprehension) 표기법
>>> corpus = np.array(corpus)
>>> corpus
array([0, 1, 2, 3, 4, 1, 5, 6])
```

위의 내용들을 하나의 함수로 작성하여 사용하면 편리하다.

```python
import numpy as np

def preprocess(text):
    text = text.lower()
    text = text.replace('.', ' .')
    words = text.split(' ')

    word_to_id = {}
    id_to_word = {}
    for word in words:
        if word not in word_to_id:
            new_id = len(word_to_id)
            word_to_id[word] = new_id
            id_to_word[new_id] = word

    corpus = np.array([word_to_id[w] for w in words])

    return corpus, word_to_id, id_to_word
```

말뭉치를 다루기 위한 준비는 끝!

### 2. 단어의 분산 표현 (Distributional Representation)

색깔을 색깔의 이름보다 RGB 값으로 표현하는 것이 더 간결하고 관계성 파악이 수월하듯이 단어에도 벡터표현을 적용하면 어떨까?

* 단어의 분산표현 : 단어를 고정 길이의 밀집 벡터(dense vector)로 표현한다. 대부분의 원소가 0이 아닌 실수인 벡터.

### 3. 분포 가설 (Distributional Hypothesis)

자연어 처리의 역사에서 단어를 벡터로 표현하고자 할 때, 근간을 두었던 아이디어/가설.

* 분포 가설 : '단어의 의미는 주변 단어에 의해 형성된다' 라는 가설
  * 단어 자체에는 의미가 없다. 그 단어가 사용된 **맥락(context, 컨텍스트)**이 의미를 형성한다.

![](./images/fig 2-3.png)

* 맥락 (context) : 특정 단어를 중심에 둔 그 주변 단어를 말한다.
* 윈도우 크기 (window size) : 확인할 주변 단어의 개수
  * 방향은 언어나 상황에 따라 다르게 설정할 수 있음. (단방향/양방향)

### 4. 동시발생 행렬 (Co-Occurrence Matrix)

특정 단어 주변에 어떤 단어가 몇번이나 등장하는지를 세어보는 방법이다.

윈도우 크기를 1로 생각하고 샘플 문장이었던 "you say goodbye and i say hello." 를 예로 들어 살펴보자.

등장한 단어를 카운팅하여 벡터로 표현하는 것이다.

* `you` : 주변에 있는 단어는 `say` 뿐이다. => [0, 1, 0, 0, 0, 0, 0]

  ![](./images/fig 2-4.png)

  ![](./images/fig 2-5.png)

* `say` : 총 2번 등장하기 때문에 주변에 있는 단어는 총 4개이다. => [1, 0, 1, 0, 1, 0, 1]

  ![](./images/fig 2-6.png)

모든 단어를 위와 같은 방식으로 카운팅 해보면 다음과 같은 벡터 표가 만들어진다.
![](./images/fig 2-7.png)

이는 **모든 단어에 대해 동시발생하는 단어를 표에 정리한 것**으로서, 이를 **동시발생 행렬(co-occurrence matrix)**이라고 부른다. 모든 문장에 적용 할 수 있도록 일반화시켜 코드로 작성해보자.

```python
def create_co_matrix(corpus, vocab_size, window_size=1):
    corpus_size = len(corpus)
    # 0으로 채워진 2차원의 배열로 초기화한다
    co_matrix = np.zeros((vocab_size, vocab_size), dtype=np.int32)

    # 각 단어에 대한
    for idx, word_id in enumerate(corpus):
      	# 주변 단어 카운팅
        for i in range(1, window_size + 1):
            left_idx = idx - i
            right_idx = idx + i
            
            if left_idx >= 0:             # 바운더리 체크
                left_word_id = corpus[left_idx]
                co_matrix[word_id, left_word_id] += 1
            
            if right_idx < corpus_size:   # 바운더리 체크
                right_word_id = corpus[right_idx]
                co_matrix[word_id, right_word_id] += 1
                
    return co_matrix
```

### 5. 벡터 간 유사도

동시발생 행렬을 활용하여 구해낸 각 단어간 벡터 데이터의 벡터 간 유사도를 구해보자.

* 벡터의 유사도를 측정하는 방법 
  * 벡터의 내적 활용
  
  * 유클리드 거리
  
  * 코사인 유사도 (cosine similarity) : 두 벡터가 가리키는 방향이 얼마나 비슷한가 (같으면 1 반대면 -1)
   ![](./images/e 2-1.png)
   
   * 분자 : 벡터의 내적
   * 분모 : 벡터의 노름(norm)
   
   ```python
   # eps(epsilon) : 아주 작은 값. 0으로 나누는 오류를 피하기 위해 추가된 변수
   def cos_similarity(x, y, eps=1e-8):
       normalizedX = x / (np.sqrt(np.sum(x ** 2)) + eps)
       normalizedY = y / (np.sqrt(np.sum(y ** 2)) + eps)
       return np.dot(normalizedX, normalizedY)
   ```

코사인 유사도를 이용하여 이전 샘플 문장에서  `you` 와 `I` 의 유사도를 구해보자.

```python
from common.util import preprocess, create_co_matrix, cos_similarity

text = 'You say goodbye and I say hello'
corpus, word_to_id, id_to_word = preprocess(text)
vocab_size = len(word_to_id)
C = create_co_matrix(corpus, vocab_size)

c0 = C[word_to_id['you']]
c1 = C[word_to_id['i']]
print(cos_similarity(c0, c1)) # 0.7071067691154799
```

`you` 와 `I` 의 유사도는 0.7로 계산되었다. -1~1 범주임을 고려하면 꽤 높은 유사도를 보이는 것이다.

### 6. 유사 단어의 랭킹 표시

이번에는 특정 단어와 비슷한 단어를 유사도 순으로 출력하는 랭킹 함수를 만들어보자.

