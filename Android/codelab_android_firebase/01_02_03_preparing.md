# Firebase Android Codelab
이 글은 Google Developer codelab이 영문으로 되어있어 보기힘들었던 내 자신을 위해 내 입맛대로 보기쉽게 번역한 글입니다.

## [1. 개요](https://codelabs.developers.google.com/codelabs/firebase-android/#0)
![동작중인 채팅 앱](images/working_friendly_chat_app.png "title" "width:300px")
[그림 : 동작중인 Friendly Chat 앱]

재잘재잘 채팅 앱 코드랩에 오신걸 환영합니다! 이 코드랩에서 당신은 Android Application을 만들기 위해서 Firebase platform을 어떻게 사용하는지 배우게 될 것 입니다. 당신은 Firebase를 이용하여 채팅 클라이언트 앱을 만들고 그것의 성능을 모니터하게 될 것입니다.

### 우리가 배우게 될 것들 
* 사용자가 로그인하도록 한다.
* ==Firebase Realtime Database==를 이용하여 데이터를 동기화 한다.
* ==Firebase Notifications==을 이용하여 Background messages를 받는다.
* ==Firebase Remote Config==를 이용하여 앱을 설정한다.
* ==Google Analytics for Firebase==를 이용하여 앱의 사용 흐름을 트래킹한다.
* ==Firebase invites==를 이용하여 사용자가 앱 설치 초대장을 보낼 수 있도록 한다.
* ==AdMob==를 이용하여 광고를 띄운다.
* ==Firebase Crash Reporting==을 이용하여 오류를 보고한다.
* ==Firebase Test Lab==을 이용하여 앱을 테스트한다.

### 배움에 앞서 준비할 것들
* Android Studio 2.1.+ 버전
* 샘플 코드 ~~[이거 말야?](https://firebase.google.com/docs/samples/?hl=ko)~~
* 테스트용 디바이스 (Android 2.3+) 혹은 에뮬레이터 (Google Play 9.8 버전 이상 설치되어 있어야 함!)
* Google 앱 6.6+ 버전 (testing Firebase App의 Step9에서만 필요함)
* 디바이스를 사용할 것이라면, 연결용 케이블이 필요합니다 (찡긋)

## [2. 샘플코드 받기](https://codelabs.developers.google.com/codelabs/firebase-android/#1)
아래의 커멘드를 입력하여 GitHub 저장소를 복사하세요.
```bash
$ git clone https://github.com/firebase/friendlychat-android
```
> "friendlychar-android" 저장소에는 샘플 프로젝트가 많이 들어있어요. 이번 코드랩에서는 두개의 프로젝트만 사용할거에요:
> - android-start : 이 코드랩을 시작할때 처음으로 사용할 코드에요
> - android : 샘플 앱을 다 만들고나서 완성된 코드에요

> **Note** : 완성된 앱을 실행시키고 싶다면 해당 패키지명과 SHA1에 대응하는 프로젝트를 Firebase console에서 생성해야 해요. 이 부분은 ['04. Create Firebase console Project'](https://codelabs.developers.google.com/codelabs/firebase-android/#3)를 참고해보세요.
> 또한, 구글을 승인된 앱 제공자로 등록해야 해요. 이건 Firebase console의 Authentication 섹션에서 진행하면 됩니다.

## [3. 스타터 앱 가져오기](https://codelabs.developers.google.com/codelabs/firebase-android/#2)
안드로이드 스튜디오를 켜고 다운받은 샘플코드의 `android-start` 디렉토리를 선택하세요.(**File > Open >** ../friendlychat-android/android-start)
자, 이제 안드로이드 스튜디오에서 android-start 프로젝트를 여세요! :-D
