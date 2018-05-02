Etc
====
과거에 작성했던 원노트 백업용 문서.

* Touch Event Parent Intercept : http://stackoverflow.com/questions/20590236/scrollview-with-children-view-how-to-intercept-scroll-condItionally

# Network Libs

| 종류/레이어 | 라이브러리 예시 |
|---------------------|-----------------------------------------------|
| [AsyncImgerLoaders] | Picasso  NetworkImageView/ImageLoader |
| [Network Libs] | Retrofit  Volley  Netty |
| [Http Stack] | OkHttp  UrlConnection  ApacheHttp |

* Picasso, Retrofit, OkHttp : http://square.github.io/

# LunchMode 
//FLAG_ACTIVITY_NEW_TASK 플래그를 설정하지 않는 한, 기존 task에 Activity Instance가 들어감
1. Standard
2. SingleTop : 액티비티 스택의 top에 있는 클래스의 인스턴스가 top으로 또 쌓이려고하면 새로 생성되지 않고 이전 액티비티가 onNewIntent()를 콜백하며 재실행됨.
    * 액티비티가 상속되었을 경우는 어떻게 될까?
    > Similarly, if you navigate up to an activity on the current stack, the behavior is determined by the parent activity's launch mode. If the parent activity has launch mode singleTop (or the up intent contains FLAG_ACTIVITY_CLEAR_TOP), the parent is brought to the top of the stack, and its state is preserved. The navigation intent is received by the parent activity's onNewIntent() method. If the parent activity has launch mode standard (and the up intent does not containFLAG_ACTIVITY_CLEAR_TOP), the current activity and its parent are both popped off the stack, and a new instance of the parent activity is created to receive the navigation intent.

    * 하위는 권장X, 불안정한 듯. os자체 문제인 듯
    * SingleTask와 SingleInstance가 걸려있는 액티비티들은 한 태스크에서만 실행가능해짐.
    * 즉, 언제나 액티비티 스택의 루트가 됨…….
    > In contrast, "singleTask" and "singleInstance" activities can only begin a task. They are always at the root of the activity stack. Moreover, the device can hold only one instance of the activity at a time — only one such task.

3. SingleTask : 단 하나의 Task만이 존재.
    * 그 Task안에 여러 Activity Instance가 쌓일 수 있음

4. SingleInstance : 단 하나의 Instance만이 존재
    * Instance는 Task에 종속적이므로 Activity가 불릴 때마다 새로운 Task와 1:1 매칭이 되어 생겨난다. 즉 Instance호출 시마다 Task가 함께 하나씩 증가함

5. 참고자료
    * Activity 중복 실행 방지관련 질문 글의 댓글 토론 : 
    http://www.androidpub.com/796480
    * LaunchMode 간단 정리 : 
    http://darksilber.tistory.com/entry/launchMode-%EC%86%8D%EC%84%B1

# Resource Merge
* PPT 자료 만들었음.
* BuildType -> Flavor -> Main -> Dependencies