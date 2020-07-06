# RealAlarm
스마트 모바일 프로그래밍 약쏙 최종 보고서
===================================
목차   
-----


### 1.소개
>#### 1-1 주제선정이유
>#### 1-2 앱 개발중 사용한 기능
>#### 1-3 앱 개발로 얻는효과

### 2. 기능 구현
>#### 2-5 복용시간 알림
>>##### 2-5-1 알림설정
>>##### 2-5-2 푸시알림
>>##### 2-5-4 다중알림
>>##### 2-5-5 알림삭제

## 2. 기능구현

>### 2-5 복용시간 알림
알림을 설정했을 때 firebase에 데이터를 저장을 하기 위해서 firebase와 연동을 해야한다. 

>>2-5-1 알림설정
##### firebase연동
firebae와 연동하기 위해서 gradle에 다음과 같은 코드를 추가한다.
~~~java
apply plugin: 'com.google.gms.google-services'

dependencies{
implementation 'com.google.firebase:firebase-analytics:17.4.3'
    implementation 'com.google.firebase:firebase-auth:19.3.1'
    implementation 'com.google.firebase:firebase-firestore:21.4.3'
}
~~~

    

