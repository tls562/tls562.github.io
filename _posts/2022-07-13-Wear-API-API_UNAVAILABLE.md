---
title: "Wearable API statusCode=API_UNAVAILABLE error"
categories:
  - android
tags:
  - kotlin
---


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