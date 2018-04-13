# Firebase Android Codelab
이 글은 Google Developer codelab이 영문으로 되어있어 보기힘들었던 내 자신을 위해 내 입맛대로 보기쉽게 번역한 글입니다.

## [4. Firebase console 프로젝트 만들기](https://codelabs.developers.google.com/codelabs/firebase-android/#3)
1. [Firebase console](https://console.firebase.google.com) 로 이동하세요.
2. ** Create New Proejct **를 선택하고 "FriendlyChat"이라는 이름으로 프로젝트를 만드세요.

### Android 앱 연결하기
1. 새 프로젝트의 개요 화면에서 **Android 앱에 Firebase 추가**를 클릭하세요.

2. 코드랩 패키지명을 입력하세요 : `com.google.firebase.codelab.friendlychat`

3. 서명된 키스토어의 SHA1를 입력하세요. 만약 표준 디버그 키스토어를 사용하고 있다면 아래의 커멘드를 입력하여 SHA1 hash값을 확인해보세요
```bash
$ keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore -list -v -storepass android
```
> **Note:** 디버그 키스토어 (debug.keysotre)는 보통 <home>/.android/debug.keystore에 위치합니다. [여기서](https://developers.google.com/android/guides/client-auth) SHA1를 찾는 좀 더 디테일한 방법을 찾아보세요.

### google-services.json 파일을 앱에 추가하기
패키지명과 SHA1를 추가하고 계속 버튼을 선택하면, 브라우저가 자동으로 설정 파일을 다운로드 합니다. 설정 파일에는 우리 앱을 위한 필수적인 Firebase metadata가 모두 들어있어요. google-service.json 파일을 복사해서 안드로이드 프로젝트 내 `app` 디렉토리에 붙여넣으세요. 

### google-services 플러그인을 앱에 추가하기
google-servies 플러그인은 google-services.json 파일을 사용하여 앱이 firebase를 사용할 수 있도록 설정합니다. 아래의 문장이 이미 `app/build.gradle`에 추가되어있어야 해요.
```groovy
apply plugin: 'com.google.gms.google-services'
```

### Gradle Sync 맞추기
모든 의존성들이 프로젝트에서 사용 가능한 것인지 확실하게 확인하기 위해, gradle 관련 파일들을 프로젝트와 동기화 해야 한다. Android Studio 툴바에서 ![Sync Project with Gradle Files](https://codelabs.developers.google.com/codelabs/firebase-android/img/19a637a9cffe4b32.png) 버튼을 클릭하자. 

> **Note: **이번 코드랩에서는 여기에서만 딱 한번 Gradle Sync를 하게 됩니다.
