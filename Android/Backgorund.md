백그라운드 작업 처리
=====
안드로이드가 버전이 업그레이드 되면서 계속 백그라운드 작업에 제한을 두고있다.

어휴 이거 테스트하다보니 대기시간들때문에 시간이 훅훅 가네...

## 해결?방법

* 알림 채널을 이용한 포어그라운드 서비스 이용
* JobScheduler 이용


### JobScheduler

```logcat
08-01 16:09:25.757 31880-31880/packagename D/DETECTAPP: [MainActivity] Job button onClick!
08-01 16:09:25.771 31880-31880/packagename D/DETECTAPP: [MainActivity] jobSchedule result=1
08-01 16:09:27.176 31880-31880/packagename D/DETECTAPP: [MainActivity] Periodic Job button onClick!
08-01 16:09:27.176 31880-31880/packagename W/JobInfo: Specified interval for 1002 is +3s0ms. Clamped to +15m0s0ms
08-01 16:09:27.177 31880-31880/packagename W/JobInfo: Specified flex for 1002 is +3s0ms. Clamped to +5m0s0ms
08-01 16:09:27.178 31880-31880/packagename D/DETECTAPP: [MainActivity] jobSchedule result=1
08-01 16:09:27.178 31880-31880/packagename D/DETECTAPP: [MainActivity] jobSchedule result=1
08-01 16:09:30.778 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@67e9973
    [CollectUsageDataJobService] [1001] isOverrideDeadlineExpired!
08-01 16:13:15.829 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@45e1a2e
08-01 16:14:53.881 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@7507f5c
08-01 16:17:22.031 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@b5ab3a
08-01 16:19:27.273 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@6143548
08-01 16:19:58.964 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@1c34106
08-01 16:21:30.843 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@3db5df4
08-01 16:21:30.862 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@3db5df4
08-01 16:23:38.374 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@bc26792
08-01 16:27:38.447 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@74a6560
08-01 16:30:03.431 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@91b6ade
08-01 16:35:40.914 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@ae0778c
08-01 16:46:23.556 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@edf7ddb

```

### `Periodic`

```java
JobInfo jobInfo = new JobInfo.Builder(CollectUsageDataJobService.PERIODIC_JOB_ID, new ComponentName(context, CollectUsageDataJobService.class))
                            .setPeriodic(3000L, 3000L)
//                          .setPersisted(true)
                            .setRequiresCharging(true)
                            .build();
```

```logcat
08-01 16:19:27.273 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@6143548
//30
08-01 16:19:58.964 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@1c34106
//90 : 1분 30초
08-01 16:21:30.843 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@3db5df4
//120 : 2분
08-01 16:23:38.374 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@bc26792
//240 : 4분
08-01 16:27:38.447 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@74a6560
//480 : 8분...
08-01 16:35:40.914 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@ae0778c
```

## 배터리 충전 해제 후 재 충전시
거의 바로 반응옴

```logcat
08-01 17:17:39.691 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1002] onStartJob : packagename.CollectUsageDataJobService@c388c84
08-01 17:18:24.010 31880-31880/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@f1d20a2

```


## 일반잡 vs interval vs periodic

```logcat
08-01 17:58:22.476 9526-9526/packagename D/DETECTAPP: [PackageStatusService] onCreate
08-01 17:58:36.806 9526-9526/packagename D/DETECTAPP: [MainActivity] Job button onClick!
08-01 17:58:36.822 9526-9526/packagename D/DETECTAPP: [MainActivity] jobSchedule result=1
08-01 17:58:37.270 9526-9526/packagename D/DETECTAPP: [MainActivity] Periodic Job button onClick!
08-01 17:58:37.270 9526-9526/packagename W/JobInfo: Specified interval for 1002 is +3s0ms. Clamped to +15m0s0ms
08-01 17:58:37.271 9526-9526/packagename W/JobInfo: Specified flex for 1002 is +3s0ms. Clamped to +5m0s0ms
08-01 17:58:37.271 9526-9526/packagename D/DETECTAPP: [MainActivity] jobSchedule result=1
08-01 17:58:37.875 9526-9526/packagename D/DETECTAPP: [MainActivity] Job button onClick!
08-01 17:58:37.910 9526-9526/packagename D/DETECTAPP: [MainActivity] jobSchedule result=1
08-01 17:58:39.858 9526-9526/packagename D/DETECTAPP: [CollectUsageDataJobService] [1000] onStartJob : packagename.CollectUsageDataJobService@7a1e0c0
08-01 17:58:40.951 9526-9526/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@ff4c53e
08-01 17:59:13.486 9526-9526/packagename D/DETECTAPP: [CollectUsageDataJobService] [1000] onStartJob : packagename.CollectUsageDataJobService@b2448ec
08-01 17:59:13.488 9526-9526/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@b2448ec
08-01 18:00:41.596 9526-9526/packagename D/DETECTAPP: [CollectUsageDataJobService] [1000] onStartJob : packagename.CollectUsageDataJobService@6f3f74a
08-01 18:00:41.599 9526-9526/packagename D/DETECTAPP: [CollectUsageDataJobService] [1001] onStartJob : packagename.CollectUsageDataJobService@6f3f74a

```

### 조건 미충족+타임오버시 제대로 실행되는애는...
* interval+deadline만 정상적으로 실행됨

```logcat
08-01 20:35:57.676 5770-5770/packagename D/DETECTAPP: [PackageStatusService] onCreate
08-01 20:35:59.530 5770-5770/packagename D/DETECTAPP: [MainActivity] Job button onClick!
08-01 20:35:59.551 5770-5770/packagename D/DETECTAPP: [MainActivity] jobSchedule result=1
08-01 20:36:00.010 5770-5770/packagename D/DETECTAPP: [MainActivity] Periodic Job button onClick!
08-01 20:36:00.011 5770-5770/packagename W/JobInfo: Specified interval for 1002 is +3s0ms. Clamped to +15m0s0ms
    Specified flex for 1002 is +3s0ms. Clamped to +5m0s0ms
08-01 20:36:00.011 5770-5770/packagename W/JobInfo: Specified flex for 1002 is +3s0ms. Clamped to +5m0s0ms
08-01 20:36:00.018 5770-5770/packagename D/DETECTAPP: [MainActivity] jobSchedule result=1
08-01 20:36:00.495 5770-5770/packagename D/DETECTAPP: [MainActivity] Job button onClick!
08-01 20:36:00.496 5770-5770/packagename D/DETECTAPP: [MainActivity] jobSchedule result=1
08-01 20:36:04.541 5770-5770/packagename D/DETECTAPP: [SingleJobService] [1001] onStartJob : packagename.SingleJobService@dc3fc00
    [SingleJobService] [1001] isOverrideDeadlineExpired!
```

* Deadline으로 인해 실행된 경우에는 조건들의 변화 모두 무시됨. 즉, 무조건 실행되는 것이 보장되는 듯.

```logcat
// onStartJob()에서 1분동안 멈추는 코드를 작성함
08-02 12:30:24.033 14713-14713/packagename D/DETECTAPP: [SingleJobService] [1001] onStartJob : packagename.SingleJobService@4f0e63
08-02 12:30:24.034 14713-14713/packagename D/DETECTAPP: [SingleJobService] [1001] isOverrideDeadlineExpired!
08-02 12:30:24.034 14713-14713/packagename D/DETECTAPP: [SingleJobService] [1001] 멈춰
// $ adb shell dumpsys battery unplug 을 하였으나 Stop되지 않고 정상종료됨
08-02 12:31:24.037 14713-14713/packagename D/DETECTAPP: [SingleJobService] [1001] 풀려
```

??? 이건 왜 정상종료하지??;;;

```logcat
08-02 12:41:53.459 14713-14713/packagename D/DETECTAPP: [SingleJobService] [1002] onStartJob : packagename.SingleJobService@13a15b7
08-02 12:41:53.459 14713-14713/packagename D/DETECTAPP: [SingleJobService] [1002] 멈춰
// $ adb shell dumpsys battery unplug 을 하였으나 Stop되지 않고 정상종료됨(????)
08-02 12:42:53.461 14713-14713/packagename D/DETECTAPP: [SingleJobService] [1002] 풀려
```



## 주의사항
* JobScheduler는 Doze모드에서 작동하지 않는다.
> [참고](https://stackoverflow.com/questions/38344220/job-scheduler-not-running-on-android-n)

## 참고자료
* https://medium.com/til-kotlin-ko/android-o%EC%97%90%EC%84%9C%EC%9D%98-%EB%B0%B1%EA%B7%B8%EB%9D%BC%EC%9A%B4%EB%93%9C-%EC%B2%98%EB%A6%AC%EB%A5%BC-%EC%9C%84%ED%95%9C-jobintentservice-250af2f7783c