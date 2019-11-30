# GCP - Natural Language API

Google Cloud Platform 의 Natural Language API 를 사용해보자.

여기서는 감성분석(Sentiment Analysis) API를 사용 해 볼 것이다.

## Pricing

https://cloud.google.com/natural-language/pricing?hl=ko

* 한 단위. : 1000자

* 월 5000 건 이하의 경우에는 무료다!

음.. 테스트는 맘껏 해봐도 되겠다.

## Get Started

#### 1. Google Natural Language API 설정하기

Google Cloud Console 에 들어가서 Google Cloud Natural Language API 를 사용 설정한다.

#### 2. gcloud sdk 설치/로그인하기

1. [Cloud SDK](https://cloud.google.com/sdk/) 에 접속하여 각자의 os 에 맞는 버전을 다운로드한다.

2. 임의 위치에 파일압축을 푼다

3. 환경변수 PATH 에 해당 위치를 설정한다
  ```bash
  GOOGLE_CLOUD_SDK_PATH=[google-cloud-sdk를 넣어둔 경로]
  PATH=$PATH:$GOOGLE_CLOUD_SDK_PATH/bin
  ```

4. 환경변수 적용 후 설치를 여부 확인한다
  ```bash
  $ gcloud --version
  Google Cloud SDK 245.0.0
  bq 2.0.43
  core 2019.05.03
  gsutil 4.38
  ```

5. 계정/프로젝트와 연결하기 위해 `gcloud` 초기화를 진행한다 

  ```bash
  $ gcloud init
  Welcome! This command will take you through the configuration of gcloud.
  
  Your current configuration has been set to: [default]
  
  You can skip diagnostics next time by using the following flag:
    gcloud init --skip-diagnostics
  
  Network diagnostic detects and fixes local network connection issues.
  Checking network connection...done.                                            
  Reachability Check passed.
  Network diagnostic passed (1/1 checks passed).
  
  You must log in to continue. Would you like to log in (Y/n)?   Y
  ### 브라우저에서 구글 로그인 진행
  
  You are logged in as: [yena.kim@formakers.net].
  
  Pick cloud project to use: 
   [1] 플젝이름1
   [2] ds-nlp-sentiment
   [3] Create a new project
  Please enter numeric choice or text value (must exactly match list 
  item) 2
  
  Your current project has been set to: [ds-nlp-sentiment].
  
  Not setting default zone/region (this feature makes it easier to use
  [gcloud compute] by setting an appropriate default value for the
  --zone and --region flag).
  See https://cloud.google.com/compute/docs/gcloud-compute section on how to set
  default compute region and zone manually. If you would like [gcloud init] to be
  able to do this for you the next time you run it, make sure the
  Compute Engine API is enabled for your project on the
  https://console.developers.google.com/apis page.
  
  Created a default .boto configuration file at [/Users/yenarue/.boto]. See this file and
  [https://cloud.google.com/storage/docs/gsutil/commands/config] for more
  information about configuring Google Cloud Storage.
  Your Google Cloud SDK is configured and ready to use!
  
  * Commands that require authentication will use yena.kim@formakers.net by default
  * Commands will reference project `ds-nlp-sentiment` by default
  Run `gcloud help config` to learn how to change individual settings
  
  This gcloud configuration is called [default]. You can create additional configurations if you work with multiple accounts and/or projects.
  Run `gcloud topic configurations` to learn more.
  
  Some things to try next:
  
  * Run `gcloud --help` to see the Cloud Platform services you can interact with. And run `gcloud help COMMAND` to get help on any gcloud command.
  * Run `gcloud topic --help` to learn about advanced features of the SDK like arg files and output formatting
  ```

6. 만약 설치가 되어있다면 그냥 로그인을 한다

   ```bash
   $ gcloud auth login
   ```

#### 3. 서비스 계정 키 다운받기

1. Google Cloud Console -> `IAM 및 관리자` -> `서비스 계정` 

2. 특정 서비스 계정의 `작업` 메뉴를 선택하여 `키 만들기` 를 진행한다.

3. 키 유형 `JSON` 으로 선택 -> `만들기`

4. 키 파일이 로컬에 다운로드 된다.

   **[주의] 해당 키는 단 한번만 다운로드 가능하므로 보관 주의 요망**

#### 4. 서비스 계정 키 설정하기

다운받은 서비스 계정 키 파일의 보관 위치를 `GOOGLE_APPLICATION_CREDENTIALS` 환경변수에 셋팅한다

```bash
export GOOGLE_APPLICATION_CREDENTIALS=해당키 파일의 경로/key.json
```

프로젝트에서 사용하고 싶은 경우에는 해당 키파일을 프로젝트 루트 하위에 놓고 Run Configuration에 해당 위치를 설정하도록 하자.

### Python Library Install

https://cloud.google.com/natural-language/docs/reference/libraries

```bash
$ pip install --upgrade google-api-core
$ pip install --upgrade google-cloud-language
$ pip install --upgrade google-api-python-client
```



## 감정분석 (Sentiment Analysis)

https://cloud.google.com/natural-language/docs/basics?hl=ko#interpreting_sentiment_analysis_values

### Response 구성

* score : 부정적 (-1.0) ~ 긍정적(1.0)
* magnitude : 전반적인 감정의 강도 (0.0~)
  * 텍스트 내의 감정 표현(긍정/부정 모두)이 모두 반영 된다.
  * 정규화 된 값이 아니기 때문에 길이가 길수록 값이 커진다.

### 돌려보기

일단 간단한 게임 피드백 1줄짜리를 콘솔에서 넣어보았다.

```bash
$ gcloud ml language analyze-sentiment --content="Enjoy your vacation!"
{
  "documentSentiment": {
    "magnitude": 0.9,
    "score": 0.9
  },
  "language": "en",
  "sentences": [
    {
      "sentiment": {
        "magnitude": 0.9,
        "score": 0.9
      },
      "text": {
        "beginOffset": 0,
        "content": "Enjoy your vacation"
      }
    }
  ]
}
```

```bash
$ gcloud ml language analyze-sentiment --content="게임 플레이가 원활 하지 않아 조금 아쉬웠습니다."
{
  "documentSentiment": {
    "magnitude": 0.4,
    "score": -0.4
  },
  "language": "ko",
  "sentences": [
    {
      "sentiment": {
        "magnitude": 0.4,
        "score": -0.4
      },
      "text": {
        "beginOffset": 0,
        "content": "\uac8c\uc784 \ud50c\ub808\uc774\uac00 \uc6d0\ud65c\ud558\uc9c0 \uc54a\uc544 \uc870\uae08 \uc544\uc26c\uc6e0\uc2b5\ub2c8\ub2e4."
      }
    }
  ]
}
$ gcloud ml language analyze-sentiment --content="게임 플레이가 원활 하지 않았지만 그래도 스토리가 좋아서 계속 했습니다."
{
  "documentSentiment": {
    "magnitude": 0.7,
    "score": 0.7
  },
  "language": "ko",
  "sentences": [
    {
      "sentiment": {
        "magnitude": 0.7,
        "score": 0.7
      },
      "text": {
        "beginOffset": 0,
        "content": "\uac8c\uc784 \ud50c\ub808\uc774\uac00 \uc6d0\ud65c\ud558\uc9c0 \uc54a\uc558\uc9c0\ub9cc \uadf8\ub798\ub3c4 \uc2a4\ud1a0\ub9ac\uac00 \uc88b\uc544\uc11c \uacc4\uc18d \ud588\uc2b5\ub2c8\ub2e4."
      }
    }
  ]
}
```

### Python 프로젝트에서 돌려보기

https://cloud.google.com/natural-language/docs/analyzing-sentiment?hl=ko#language-sentiment-string-python

사용하게 되면 파이썬 프로젝트에서 사용하게 될 거라, 파이썬 프로젝트에서 호출해봤다.

```python
import sys
from google.cloud import language_v1
from google.cloud.language_v1 import enums
import six

def sample_analyze_sentiment(content):
    # [START language_sentiment_text_core]

    client = language_v1.LanguageServiceClient()

    # content = 'Your text to analyze, e.g. Hello, world!'

    if isinstance(content, six.binary_type):
        content = content.decode('utf-8')

    type_ = enums.Document.Type.PLAIN_TEXT
    document = {'type': type_, 'content': content}

    response = client.analyze_sentiment(document)
    sentiment = response.document_sentiment
    print('Score: {}'.format(sentiment.score))
    print('Magnitude: {}'.format(sentiment.magnitude))
```

#### 유지의견

```python
sample_analyze_sentiment("무엇보다도 타격감이 엄청 좋은 거 같아요. 조작도 쉽고 하니까 누구나 재미있게 플레이를 할 수 있을 거 같아요. '태그액션' 이라는 장르가 사실 익숙한 장르가 아니다 보니까 조금은 이상했었는데 이렇게나 개성있는 게임을 플레이를 하니 정말 좋았습니다. RPG 게임이 정말 지루하다고 생각했습니다. 하지만 이렇게 스테이지 형식의 게임으로 플레이를 해보니 정말 재미있네요. 신박해서 정말 좋았습니다. 때리다보면 귀도 즐겁네요!")

# Score: 0.5
# Magnitude: 3.5999999046325684
```

score 0.5, Magnitude 3.6 정도로 전반적으로 긍정적이라 볼 수 있다

#### 개선의견

```python
sample_analyze_sentiment("첫 인상 ) UI가 너무 간단하다 못해 퀄리티가 떨어져보여요. 아무리 게임이 재밌다고 하더라도 UI보고 약간 꺼려할 거 같아요. 약간만 바꾸면 좋을 듯 합니다. 처음하는 사람은 이게 뭐지 싶을 거 같아요. 플레이 도중 ) 타격감도 좋고 게임 다 신기하고 뭔가 엄청 신박해서 좋은데 보스까지 처치하고 나서 메인 메뉴로 이동이 없어서 조금 아쉬워요! 그거 말고는 저는 다 좋았던 거 같아요. UI도 처음에는 왜 이렇게 놨을까 하고 고민을 조금 했는데 방치형이랑 스테이지 형식의 게임을 조금 섞어서 만든 거처럼 그래서 그랬던거구나 싶었습니다.")

# Score: 0.10000000149011612
# Magnitude: 3.5
```

score 0.1, Magnitude 3.5 정도로 중립에 가깝다고 볼 수 있다.

우리 유저들의 피드백 자체가 워낙 정중한 편이라, 개선의견에  **부정적인 표현 + 긍정적인 표현**이 공존해서 중립이나 긍정이 나오는 경우가 많을 것 같다.

> 아이디어 : 각 문장별로 감정을 표시해주면 어떨까? 네거티브는 파란 색, 파지티브는 밝은 색

#### 전체 피드백 vs 피드백 한 문장씩 평균

Magnitude 값이 그냥 길이가 길수록 증가하다보니 불명확해보여서 한문장씩 끊어서 보내면 어떻게 나올까 궁금해졌다.

##### 직접 잘라보기

```python
import re
from google.cloud import language_v1
from google.cloud.language_v1 import enums
import six

def sample_analyze_sentiment(content):
    # [START language_sentiment_text_core]

    client = language_v1.LanguageServiceClient()

    # content = 'Your text to analyze, e.g. Hello, world!'

    if isinstance(content, six.binary_type):
        content = content.decode('utf-8')

    type_ = enums.Document.Type.PLAIN_TEXT
    document = {'type': type_, 'content': content}

    response = client.analyze_sentiment(document)
    sentiment = response.document_sentiment
    print('Score: {}'.format(sentiment.score))
    print('Magnitude: {}'.format(sentiment.magnitude))

    return sentiment.score, sentiment.magnitude

# feedback = "무엇보다도 타격감이 엄청 좋은 거 같아요. 조작도 쉽고 하니까 누구나 재미있게 플레이를 할 수 있을 거 같아요. '태그액션' 이라는 장르가 사실 익숙한 장르가 아니다 보니까 조금은 이상했었는데 이렇게나 개성있는 게임을 플레이를 하니 정말 좋았습니다. RPG 게임이 정말 지루하다고 생각했습니다. 하지만 이렇게 스테이지 형식의 게임으로 플레이를 해보니 정말 재미있네요. 신박해서 정말 좋았습니다. 때리다보면 귀도 즐겁네요!"
feedback = "첫 인상 ) UI가 너무 간단하다 못해 퀄리티가 떨어져보여요. 아무리 게임이 재밌다고 하더라도 UI보고 약간 꺼려할 거 같아요. 약간만 바꾸면 좋을 듯 합니다. 처음하는 사람은 이게 뭐지 싶을 거 같아요. 플레이 도중 ) 타격감도 좋고 게임 다 신기하고 뭔가 엄청 신박해서 좋은데 보스까지 처치하고 나서 메인 메뉴로 이동이 없어서 조금 아쉬워요! 그거 말고는 저는 다 좋았던 거 같아요. UI도 처음에는 왜 이렇게 놨을까 하고 고민을 조금 했는데 방치형이랑 스테이지 형식의 게임을 조금 섞어서 만든 거처럼 그래서 그랬던거구나 싶었습니다."
sentences = re.split("[.!?~]\s+", feedback)
print(sentences)
print('문장 개수 : {}'.format(len(sentences)))

total_score, total_magnitude = 0, 0
for sentence in sentences:
    print("")
    print(sentence)
    score, magnitude = sample_analyze_sentiment(sentence)
    total_score += score
    total_magnitude += magnitude

print("=========================")
print('평균 Score : {}'.format(total_score / len(sentences)))
print('평균 Magnitude : {}'.format(total_score / len(sentences)))
print("=========================")
print("한꺼번에 구했을 때 : ")
sample_analyze_sentiment(feedback)

# =========================
# 평균 Score : 0.14285714392151153
# 평균 Magnitude : 0.14285714392151153
# =========================
# 한꺼번에 구했을 때 : 
# Score: 0.10000000149011612
# Magnitude: 3.5
```

스코어는 조금 증가하고 Magnitude 는 좀 더 정규화 되었다. 

이렇게 비교하고 보니, 그냥 길수록 Magnitude 가 증가하는 게 자연스러워 보이기도 하다.. .그만큼 의견이 많다는 거니까.... 강하다고 볼 수 있지 않나....... 흐으으음 고민된다.

> 아이디어) 성실의 여부를 Magnitude 로 활용할수도 있겠다 싶다. 길이가 짧아도 강하게 적으면 이게 올라가니까.

##### API 내에 있는 `sentences` 필드 활용하기

* 테스트 결과 문서 : https://docs.google.com/spreadsheets/d/1NLMUXenYXHiu4MUEPwCwIOsbTo6ZLSraSOuEFnH03y0/

API Response 구조를 다시 한번 살펴보니, 문장을 알아서 끊어서 보내주고 있더라!

```python
def sample_analyze_sentiment(content):
  ...생략
  return doc_sentiment.score, doc_sentiment.magnitude, response.sentences
  
print("한꺼번에 구했을 때 : ")
score, magnitude, doc_sentences = sample_analyze_sentiment(feedback)
sentiments = map(lambda sen: sen.sentiment, doc_sentences)
total_doc_score, total_doc_magnitude = 0, 0
for sentiment in sentiments:
    print("")
    print(sentiment)
    total_doc_score += sentiment.score
    total_doc_magnitude += sentiment.magnitude

print(total_doc_score / len(doc_sentences))
print(total_doc_magnitude / len(doc_sentences))

# 한꺼번에 구했을 때 : 
# Score: 0.10000000149011612
# Magnitude: 3.5
# 
# 생략..
#
# 0.114285716
# 0.457142864
```

* 일부 문장들의 경우 score, magnitude 값이 없기도 하다..;; 뭐지?;;;;
* 값이 있는 경우, 직접 문장을 잘라서 api 를 쐈을 경우와 거의 유사한 값이 나왔다. (이 정도면  뭐 거의 같다고 볼 수 있음)
  * 이 정도면 그냥 직접 자르지 말고, API 에서 보내준 걸 그대로 쓰자
  * 그대로 쓰는게 호출 횟수도 줄어들고, 단위도 줄어들어 비용도 더 절약될 듯

### 평가

이 API의 평가는 어떻게 하는걸까? 이 api의 신뢰도를 어떻게 믿지.... 흠 

