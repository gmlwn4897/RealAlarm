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



floatingActionButton을 추가하기 위해서 alarm_activity_main.xml 레이아웃에 코드를 추가한다.
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




floatingActionButton을 누르면 알림을 설정할 수 있는 레이아웃으로 넘어가도록 다음과 같은 코드를 추가한다.

~~~java
  @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_alarm,container,false);
        editText = (EditText)view.findViewById(R.id.editText);
        timePicker = (TimePicker)view.findViewById(R.id.timepicker);
        recyclerView = view.findViewById(R.id.recyclerView);
        recyclerView.setHasFixedSize(true);
        final RecyclerView.LayoutManager layoutManager = new LinearLayoutManager(getActivity());
        recyclerView.setLayoutManager(layoutManager);
        firebaseFirestore = FirebaseFirestore.getInstance();
        alarmManager = (AlarmManager)getActivity().getSystemService(Context.ALARM_SERVICE);
        view.findViewById(R.id.floatingActionButton).setOnClickListener(onClickListener);
        alarmUpdate();
        return view;
    }

    View.OnClickListener onClickListener = new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            myStartActivity(SettingAlarm.class);
        }
    };
~~~




알림설정을 하는 layout코드는 다음과 같다.
~~~java
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".AlarmMainActivity">


    <EditText
        android:id="@+id/editText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:layout_marginBottom="16dp"
        android:hint="복용해야할 약 이름을 입력해주세요."
        app:layout_constraintBottom_toTopOf="@+id/timepicker"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.446"
        app:layout_constraintStart_toStartOf="parent" />

    <TimePicker
        android:id="@+id/timepicker"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:timePickerMode="spinner"
        app:layout_constraintBottom_toTopOf="@id/btnset"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_chainStyle="packed" />

    <Button
        android:id="@+id/btnset"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:text="저장"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintHorizontal_chainStyle="spread"
        app:layout_constraintLeft_toRightOf="@+id/btncancel"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/timepicker" />

    <Button
        android:id="@+id/btncancel"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:text="취소"
        app:layout_constraintHorizontal_chainStyle="spread"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toLeftOf="@+id/btnset"
        app:layout_constraintTop_toBottomOf="@id/timepicker" />



</androidx.constraintlayout.widget.ConstraintLayout>
~~~



알림을 설정하고 저장버튼을 누르면 setAlarm메소드로 가고 알림리스트 창으로 가게한다.
~~~java
View.OnClickListener onClickListener = new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // 프래그먼트 사용을 위해 transaction 정의
            if(v.getId() ==R.id.btnset){
                setAlarm();
               replaceFragment(fragmentAlarm);

           }
        }
    };
~~~

timepicker와 edittext를 이용해서 알림을 설정하면 다음코드와 같이 저장되고 uploader메소드로 넘어가면서 firebase에 저장이 되도록한다.
~~~
private void setAlarm() {//알림 설정

        hourtime = timePicker.getCurrentHour().toString();
        minutetime = timePicker.getCurrentMinute().toString();
        notificationText = drugEditText.getText().toString();


        final String timetimes = hourtime + minutetime;
        //int hourtext = Integer.parseInt(hourtime);

         String times = hourtime+minutetime;

        int hourtest = Integer.parseInt(hourtime);
        int minutetest = Integer.parseInt(minutetime);

        String hourtext = "";
        String minutetext = "";

        String realTime = "";

        if (hourtime.length() > 0 && minutetime.length() > 0) {
            if (hourtest > 11 && hourtest < 24) {
                ampm = "오후";
                realTime = String.valueOf(hourtest - 12);
            } else {
                ampm = "오전";
                realTime = String.valueOf(hourtest);
            }


            if (hourtest < 10) {
                hourtext = " " + realTime + ":";

            } else {
                hourtext = realTime + ":";
            }
            if (minutetest < 10) {
                minutetext = "0" + minutetest;
            } else {
                minutetext = minutetime;
            }


            alarmInfo = new AlarmInfo(hourtime, minutetime, notificationText, ampm,times);
            uploader(alarmInfo);

            /*
            if (Build.VERSION.SDK_INT < Build.VERSION_CODES.O) {
                Toast.makeText(this, "버전을 확인해주세요.", Toast.LENGTH_SHORT).show();
                return;
            }
             */



        }
    }

    //저장 버튼을 누르면 hour,minute,drugtext를 파이어베이스에 넘어감
    //DB에 업로드 되는 코드
    private void uploader(final AlarmInfo alarmInfos) {

            firebaseFirestore = FirebaseFirestore.getInstance();
            final DocumentReference documentReference = modifyAlarm == null ? firebaseFirestore.collection("AlarmDemo").document()
                    : firebaseFirestore.collection("AlarmDemo").document(modifyAlarm.getId());
            // final DocumentReference documentReference = firebaseFirestore.collection("AlarmDemo").document(times);


            //Log.e("log : ",alarmInfo.getId().toString());
            documentReference.set(alarmInfos.getAlarmInfo())
                .addOnSuccessListener(new OnSuccessListener<Void>() {
                    @Override
                    public void onSuccess(Void aVoid) {
                        Log.e(TAG, "id");
                    }
                })
                .addOnFailureListener(new OnFailureListener() {
                @Override
                public void onFailure(@NonNull Exception e) {
                    Log.e(TAG, "Error", e);
                }
            });




        firebaseFirestore = FirebaseFirestore.getInstance();
        firebaseFirestore.collection("AlarmDemo").get()
                .addOnCompleteListener(new OnCompleteListener<QuerySnapshot>() {
                    @Override
                    public void onComplete(@NonNull Task<QuerySnapshot> task) {

                        if(task.isSuccessful()){
                            for (QueryDocumentSnapshot documentSnapshot : task.getResult()){
                                if((alarmInfos.getHour()+alarmInfos.getMinute()).equals(documentSnapshot.getData().get("times"))){
                                   // cancelId = getIntent().getStringExtra("cancelId");
                                    if (modifyAlarm != null){
                                        Log.e("getHour+getMinute ==>",alarmInfos.getHour()+alarmInfos.getMinute());
                                        Log.e("documentSnapshot",documentSnapshot.getData().get("times").toString());

                                        Intent intent = new Intent(getApplicationContext(), AlarmReceiver.class);
                                        intent.putExtra("drug", documentSnapshot.getData().get("drugtext").toString());
                                        intent.putExtra("id",alarmInfo2);


                                        PendingIntent pIntent = PendingIntent.getBroadcast(getApplicationContext(), Integer.parseInt(alarmInfo2),intent, PendingIntent.FLAG_UPDATE_CURRENT);
                                        alarmManager.cancel(pIntent);
                                        pIntent.cancel();

                                        Log.e("수정text : ",documentSnapshot.getData().get("drugtext").toString());
                                        Log.e("수정 id : ", documentSnapshot.getData().get("hour").toString()+documentSnapshot.getData().get("minute").toString());
                                        final Calendar calendar = Calendar.getInstance();

                                        calendar.set(Calendar.HOUR_OF_DAY, Integer.parseInt(alarmInfos.getHour()));
                                        Log.e("hourtime : ", documentSnapshot.getData().get("hour").toString());
                                        calendar.set(Calendar.MINUTE, Integer.parseInt(documentSnapshot.getData().get("minute").toString()));
                                        Log.e("minutetime : ", documentSnapshot.getData().get("minute").toString());
                                        calendar.set(Calendar.SECOND, 0);

                                        long intervalTime = 1000 * 24 * 60 * 60;
                                        long currentTime = System.currentTimeMillis();

                                        if (currentTime > calendar.getTimeInMillis()) {
                                            //알림설정한 시간이 이미 지나간 시간이라면 하루뒤로 알림설정하도록함.
                                            calendar.setTimeInMillis(calendar.getTimeInMillis() + intervalTime);
                                        }

                                        PendingIntent pIntents = PendingIntent.getBroadcast(getApplicationContext(), Integer.parseInt(alarmInfo2), intent, 0);
                                        alarmManager.set(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), pIntents);
                                        Toast.makeText(getApplicationContext(), "알림이 수정되었습니다.", Toast.LENGTH_SHORT).show();

                                    }
                                    else if (cancelId != null){
                                        Log.e("cancel : ","cancel");
                                        //alarmManager = (AlarmManager)getSystemService(Context.ALARM_SERVICE);
                                        Intent intents = new Intent(getApplicationContext(), AlarmReceiver.class);
                                        PendingIntent pIntent = PendingIntent.getBroadcast(getApplicationContext(), Integer.parseInt(cancelId),intents, PendingIntent.FLAG_UPDATE_CURRENT);
                                        alarmManager.cancel(pIntent);
                                        pIntent.cancel();

                                        Toast.makeText(getApplicationContext(), "알림이 삭제되었습니다.", Toast.LENGTH_SHORT).show();

                                        Log.e("ERROR : ","ERROR");

                                    }
                                    else {
                                            firedrugtext = documentSnapshot.getData().get("drugtext").toString();

                                            Log.e("확인확인", firedrugtext);
                                            notificationId = documentSnapshot.getData().get("times").toString();
                                            Log.e("확인확인", notificationId);

                                            Intent intent = new Intent(getApplicationContext(), AlarmReceiver.class);
                                            intent.putExtra("drug", firedrugtext);
                                            intent.putExtra("id", notificationId);

                                            final Calendar calendar = Calendar.getInstance();

                                            calendar.set(Calendar.HOUR_OF_DAY, Integer.parseInt(documentSnapshot.getData().get("hour").toString()));
                                            Log.e("hourtime : ", documentSnapshot.getData().get("hour").toString());
                                            calendar.set(Calendar.MINUTE, Integer.parseInt(documentSnapshot.getData().get("minute").toString()));
                                            Log.e("minutetime : ", documentSnapshot.getData().get("minute").toString());
                                            calendar.set(Calendar.SECOND, 0);

                                            long intervalTime = 1000 * 24 * 60 * 60;
                                            long currentTime = System.currentTimeMillis();

                                            if (currentTime > calendar.getTimeInMillis()) {
                                                //알림설정한 시간이 이미 지나간 시간이라면 하루뒤로 알림설정하도록함.
                                                calendar.setTimeInMillis(calendar.getTimeInMillis() + intervalTime);
                                            }

                                            PendingIntent pIntent = PendingIntent.getBroadcast(getApplicationContext(), Integer.parseInt(hourtime+minutetime), intent, 0);
                                            alarmManager.set(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), pIntent);
                                            Toast.makeText(getApplicationContext(), "알림이 설정되었습니다.", Toast.LENGTH_SHORT).show();
                                    }
                                }
                            }

                        }

                    }

                });
    }
~~~



firebase에 저장된 알림데이터를 가져와서 알림을 저장한다.
notification을 이용하여 알림을 하기 위해서 pendingIntent를 사용하는데, 여러개의 푸시알림을 위해서 pendingIntent에 들아가는 requestcode값이 각각 달라야한다. 따라서 이것을 구분해주기 위해서 설정한 알림시간의 시값+분값을 requestcode로 설정해주었고, intent를 이용하여 AlarmReceiver.class에 약이름과 requestcode값을 넘겨주었다.



