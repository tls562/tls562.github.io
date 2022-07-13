---
published: true
title: "Wearable API statusCode=API_UNAVAILABLE error"
categories:
  - android
tags:
  - kotlin
  - wearOs
---

api level이 17보다 높음에도 API_UNAVAILABLE 문제가 발생한 경우에는 wear device에 mobile 에서 실행하고 있는 앱과 연동된 wear app이 설치되지 않았거나, wear device 자체가 연동되지 않았기 때문에 발생한다.  


java.util.concurrent.ExecutionException: com.google.android.gms.common.api.*ApiException: 17: API: Wearable.API is not available on this device. Connection failed* with: ConnectionResult{statusCode=API_UNAVAILABLE'', resolution=null, message=null} 

~~~
2022-07-12 21:08:44.740 16470-16912/com.android.aiwatchface E/AndroidRuntime: FATAL EXCEPTION: ServiceStartArguments
    Process: com.android.aiwatchface, PID: 16470
    java.util.concurrent.ExecutionException: com.google.android.gms.common.api.ApiException: 17: API: Wearable.API is not available on this device. Connection failed with: ConnectionResult{statusCode=API_UNAVAILABLE'', resolution=null, message=null}
        at com.google.android.gms.tasks.Tasks.zza(com.google.android.gms:play-services-tasks@@18.0.1:5)
        at com.google.android.gms.tasks.Tasks.await(com.google.android.gms:play-services-tasks@@18.0.1:8)
        at com.android.aiwatchface.service.DataSyncService.setupNodeId(DataSyncService.kt:94)
        at com.android.aiwatchface.service.DataSyncService.access$setupNodeId(DataSyncService.kt:38)
        at com.android.aiwatchface.service.DataSyncService$ServiceHandler.handleMessage(DataSyncService.kt:63)
        at android.os.Handler.dispatchMessage(Handler.java:106)
        at android.os.Looper.loop(Looper.java:223)
        at android.os.HandlerThread.run(HandlerThread.java:67)
     Caused by: com.google.android.gms.common.api.ApiException: 17: API: Wearable.API is not available on this device. Connection failed with: ConnectionResult{statusCode=API_UNAVAILABLE, resolution=null, message=null}
        at com.google.android.gms.common.internal.ApiExceptionUtil.fromStatus(com.google.android.gms:play-services-base@@18.0.1:3)
        at com.google.android.gms.common.internal.zap.onComplete(com.google.android.gms:play-services-base@@18.0.1:4)
        at com.google.android.gms.common.api.internal.BasePendingResult.zab(com.google.android.gms:play-services-base@@18.0.1:7)
        at 
~~~

-  수정 전
~~~kotlin
private fun setupNodeId() {
        val capabilityInfo: CapabilityInfo = Tasks.await(
            Wearable.getCapabilityClient(this)
                .getCapability(
                    WEAR_CAPABILITY,
                    CapabilityClient.FILTER_REACHABLE
                )
        )
        nodeId = pickBestNodeId(capabilityInfo.nodes)
    }
~~~

- 수정 후
~~~kotlin
private fun setupNodeId() {
        try {
            val capabilityInfo: CapabilityInfo = Tasks.await(
                Wearable.getCapabilityClient(this)
                    .getCapability(
                        WEAR_CAPABILITY,
                        CapabilityClient.FILTER_REACHABLE
                    )
            )
            nodeId = pickBestNodeId(capabilityInfo.nodes)
        } catch (e: ExecutionException) {
            e.message?.run {
                if (contains("API_UNAVAILABLE")) {
                    Log.e(TAG, "wear app is not installed")
                    //TODO make User install wear app
                }
            }
        }
    }
~~~

이 내용은 공식 디벨로퍼 사이트에 명시되어있는 사항은 아니다. 
min target sdk level이 17보다 훨씬 높은 상황에서 저런 에러가 에뮬레이터에서 지속적으로 발생하여, 
stack overflow, google blog 검색을 통해 알음알음 알려진 사실 같은 것

내가 개인적으로 테스트해봐도 wear device 연동이 되어있지 않은 단말에서 발생하는 에러가 맞는 것 같다.
내가 확인한 부분은
wear연동이 되어있지 않은 에뮬레이터에서 아래 에러 발생, 
galaxy watch 4 연동이 되어있는 내 개인 안드로이드 폰에서는 exception이 발생하지 않았다. 

이런 에러는 보통 일반적인 제너럴한 에러기 때문에, 명시적인 exception이 별도로 존재하면서 디벨로퍼 사이트에
설명이 되는게 보통 일 것 같은데, 그런 처리가 되어있지 않는게 의아하긴 하다.

아직까지 google에서 wear에 대해서 안정적인 flatform을 제공하지 못하고 있는 상황인건지, 
그것도 그럴 것이, 개발 중 문제가 발생할 때, 검색하고 찾아내기까지 참 시간이 오래걸린다. 관련 개발 블로그 글도 거의 없는 듯하다. 

앞으로 이 부분에 대해서 어떻게 개선이 되는지 지켜보고 우선은 이 글을 끝내겠다. 

업데이트 되는대로 여기에 붙이고 붙이고 붙이겠다.