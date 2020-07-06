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
##### floatingActionButton
floatingActionButton을 추가하기 위해서 gradle에 다음과 같은 코드를 추가한다.
~~~java
dependencies {
    implementation "com.google.android.material:material:1.1.0"
}
~~~
floatingActionButton을 추가하기 위해서 fragment_alarm.xml 레이아웃에 코드를 추가한다.
~~~java
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:id="@+id/content">

        <TextView
            android:id="@+id/alarmView"
            android:layout_width="match_parent"
            android:layout_height="80dp"
            android:gravity="center"
            android:paddingTop="10dp"
            android:text="ALARM"
            android:textSize="60sp"/>

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerView"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:paddingTop="100dp">


        </androidx.recyclerview.widget.RecyclerView>

        <com.google.android.material.floatingactionbutton.FloatingActionButton
            android:id="@+id/floatingActionButton"
            android:layout_height="80dp"
            android:layout_width="80dp"
            android:layout_alignParentRight="true"
            android:layout_alignParentBottom="true"
            android:layout_marginRight="10dp"
            android:layout_marginBottom="10dp"
            android:clickable="true"
            android:src="@drawable/ic_add_black_24dp"
            android:backgroundTint="#A8A8A8"/>



</FrameLayout>
~~~





