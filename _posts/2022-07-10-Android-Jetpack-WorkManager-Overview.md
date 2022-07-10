---
title: "remote device/server와 데이터 동기화 구현 - WorkManager"
categories:
  - android
tags:
  - kotlin
  - android
  - jetpack
  - workManager
  - dataSync
---

올해 4월에 WorkManager가 릴리즈 되었다.  

현재 개인적으로 진행하고 있는 프로젝트가 wear와 mobile간 data sync가 필요한 부분이 있다.  

삭제된 데이터를 확인한 후 wear로 해당 파일을 삭제하도록 요청해야하고, 신규로 추가되었으나, 미처 wear로 전송하지 못한 데이터를 지속적으로 체크하고, 그 각각의 work를 queue로 관리하여, work 가능한 시점에 queue에 있는 작업들을 순차적으로 수행해야 했다.
또한, 사용자가 중도 정지도 가능하게 말이다. 

이런 logic을 직접 구현하려면, 참 까다롭고 골치가 아플텐데, 열심히 검색하다보니, 내가 원하던 딱 그 작업을 위한 jetpack이 존재했다! 

이런 식의 동작은 server 혹은 remote device와 항시 보장되진 않은 connection 혹은 device 환경에서 필수로 추가해야 할 작업들이다.   

얼마나 좋은 세상인가, 알고리즘을 최적화를 직접하지 않아도 최적의 인터페이스를 제공하고 있다. 

심지어 이 엄청난 WorkManager는 배터리 빵빵 시, 네트워크 빵빵 시 작업 수행 같은 조건도 처리할 수 있다. 

그리고 작업 중 작업이 실패하는 상황에서도 앱 재시작 시 이어서 동작할 수 있도록 지원한다. 

~~~kotlin
 continuation = WorkManager.getInstance(context)
            .beginUniqueWork(
                Constants.IMAGE_MANIPULATION_WORK_NAME,
                ExistingWorkPolicy.REPLACE,
                OneTimeWorkRequest.from(CleanupWorker::class.java)
            ).thenMaybe<WaterColorFilterWorker>(waterColor)
            .thenMaybe<GrayScaleFilterWorker>(grayScale)
            .thenMaybe<BlurEffectFilterWorker>(blur)
            .then(
                if (save) {
                    workRequest<SaveImageToGalleryWorker>(tag = Constants.TAG_OUTPUT)
                } else /* upload */ {
                    workRequest<UploadWorker>(tag = Constants.TAG_OUTPUT)
                }
            )
 ~~~   

작업 제약 조건 배터리 상태 혹은 네트워크 환경에 따라 작업 수행 여부를 조정할 수도 있게 되어 있다.  

*OneTimeWorkRequestBuilder* 생성 시 아래와 같은 방식으로 간단히 설정할 수 있다. 
~~~kotlin
val constraints = Constraints.Builder()
        .setRequiresCharging(true)
        .build()

// Add WorkRequest to save the image to the filesystem
val save = OneTimeWorkRequestBuilder<SaveImageToFileWorker>()
        .setConstraints(constraints)
        .addTag(TAG_OUTPUT)
        .build()
continuation = continuation.then(save)

// Actually start the work
continuation.enqueue()
 ~~~  

 WorkContinuation를 생성 후, 
 user input혹은 작업이 필요할 때마다 continuation.enqueue() 호출을 통해, queue로 작업을 관리하고 순차적으로 수행하게 된다.

~~~kotlin
internal fun cancelWork() {
        workManager.cancelUniqueWork(IMAGE_MANIPULATION_WORK_NAME)
    }
~~~  

취소도 가능하다.   

나와있는 샘플에는 *OneTimeWork*에 대한 수행이나, 아래 처럼 *"workManager 작업 유형 3가지"* 즉시, 장기 실행, 지연 가능하게 설정 가능하다. 

열심히 잘~~ 활용해봐야겠다.  사용법도 크게 어려운 부분은 없는 것 같다. 
아래 링크에 달아놓은 offitial site에서 예제로 나와 있는 이미지 변환 앱을 구동시켜보고 코드를 살펴보면 쉽게 이해 가능하다. 


workManager 작업 유형 3가지  

- 즉시: 즉시 시작하고 곧 완료해야 하는 작업입니다. 신속하게 처리될 수 있습니다.  
- 장기 실행: 더 오래(10분 이상이 될 수 있음) 실행될 수 있는 작업입니다.  
- 지연 가능: 나중에 시작하며 주기적으로 실행될 수 있는 예약된 작업입니다.

<p align="center">
<img src="/assets/jetpack_workmanager.jpg">  
</p>

유연한 재시도 정책  

경우에 따라 작업이 실패하기도 합니다. WorkManager는 구성 가능한 지수 백오프 정책을 비롯해 유연한 재시도 정책을 제공합니다.

작업 체이닝   

복잡한 관련 작업의 경우 직관적인 인터페이스를 사용하여 개별 작업을 함께 체이닝하면 순차적으로 실행할 작업과 동시에 실행할 작업을 제어할 수 있습니다.  




참고 링크  
  
[Official documentation](https://developer.android.com/topic/libraries/architecture/workmanager/)

[Release notes](https://developer.android.com/jetpack/androidx/releases/work)

