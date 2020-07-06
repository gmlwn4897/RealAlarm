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
>>##### 2-5-3 알림삭제

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

SettingAlarm.java
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
notification을 이용하여 알림을 하기 위해서 pendingIntent를 사용하는데, 여러개의 푸시알림을 위해서 pendingIntent에 들아가는 requestcode값이 각각 달라야한다.
따라서 이것을 구분해주기 위해서 설정한 알림시간의 시값+분값을 requestcode로 설정해주었고, intent를 이용하여 AlarmReceiver.class에 약이름과 requestcode값을 넘겨주었다.



AlarmReceiver.class
~~~java
public class AlarmReceiver extends BroadcastReceiver {
    String notificationid;
    AlarmManager alarmManager;
    AlarmInfo alarmInfo2;
    Intent mainIntent;
    Context contexts;
    String cancelId;
    String str;
    int i = 0;


    //받아서 푸쉬알림 해주는 부분.
    @Override
    public void onReceive(Context context, Intent intent) {
        this.contexts = context;
        notificationid = intent.getStringExtra("id");
        String text = intent.getStringExtra("drug");
        str = notificationid.toString();


        Log.e("약번호 넘어오자...", String.valueOf(notificationid));
        Log.e("약이름 넘어오자...",text);


        //푸쉬알람 해주는 부분
        mainIntent = new Intent(context, MainActivity.class);
        PendingIntent contentIntent = PendingIntent.getActivity(context, Integer.parseInt(notificationid),mainIntent, PendingIntent.FLAG_UPDATE_CURRENT);

        NotificationManager notificationManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);

        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, "drugId");

        if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.N_MR1) {

            Toast.makeText(context, "누가버전", Toast.LENGTH_SHORT).show();
            builder.setSmallIcon(R.drawable.ic_drug_icon);

            builder.setAutoCancel(true)
                    .setWhen(System.currentTimeMillis())
                    .setContentTitle("약쏙")
                    .setContentText(text + "을(를) 복용할시간에요:)")
                    .setPriority(Notification.PRIORITY_DEFAULT)
                    .setContentIntent(contentIntent)
                    .setContentInfo("INFO")
                    .setDefaults(Notification.DEFAULT_VIBRATE);


            if (notificationManager != null) {


                PowerManager powerManager = (PowerManager) context.getSystemService(Context.POWER_SERVICE);
                PowerManager.WakeLock wakeLock = powerManager.newWakeLock(PowerManager.FULL_WAKE_LOCK | PowerManager.ACQUIRE_CAUSES_WAKEUP | PowerManager.ON_AFTER_RELEASE, "My:Tag"
                );
                wakeLock.acquire(5000);
                notificationManager.notify(Integer.parseInt(notificationid), builder.build());

            }
        }


        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {

            Toast.makeText(context, "오레오 이상", Toast.LENGTH_SHORT).show();

            builder.setSmallIcon(R.drawable.ic_drug_icon);

            String channelId = "drug";
            String chaanelName = "약쏙";
            String description = "매일 정해진 시간에 알림합니다. ";
            int importance = NotificationManager.IMPORTANCE_HIGH;

            NotificationChannel channel = new NotificationChannel(channelId, chaanelName, importance);
            channel.setDescription(description);

            if (notificationManager.getNotificationChannel(channelId) == null) {
                notificationManager.createNotificationChannel(channel);
            }

            builder.setSmallIcon(R.drawable.ic_drug_icon);

            builder.setAutoCancel(true)
                    .setWhen(System.currentTimeMillis())
                    .setContentTitle("약쏙")
                    .setContentText(text + "을(를) 복용할시간에요:)")
                    .setPriority(Notification.PRIORITY_DEFAULT)
                    .setContentIntent(contentIntent)
                    .setContentInfo("INFO")
                    .setDefaults(Notification.DEFAULT_VIBRATE);


            //if(notificationManager !=null){


            PowerManager powerManager = (PowerManager) context.getSystemService(Context.POWER_SERVICE);
            PowerManager.WakeLock wakeLock = powerManager.newWakeLock(PowerManager.FULL_WAKE_LOCK | PowerManager.ACQUIRE_CAUSES_WAKEUP | PowerManager.ON_AFTER_RELEASE, "My:Tag"
            );
            wakeLock.acquire(5000);
            notificationManager.notify(Integer.parseInt(notificationid), builder.build());

            //}

        }
    }
}
~~~


settingAlarm.java에서 넘겨준 약이름을 받아와서 설정한 notification알림에 builder를 이용하여 contentText에 자기가 설정한 약이름을 띄울수있게 했다.



오레오 이상부터는 channelId를 필수로 써야하기 때문에 오레오 이전버전과, 오레오 이상버전의 notification알림을 if문을 이용해서 각각 설정해 주었다. 


알림이 설정되면 adapter를 이용해서 알림리스트에 설정한 알림이 뜨도록 했다.


alarmUpdate는 firebase에 저장된 알림데이터들을 ArrayList에 저장하여 MyAdapter로 넘겨준다. 
~~~java
private void alarmUpdate(){
        firebaseFirestore.collection("AlarmDemo").orderBy("hour", Query.Direction.ASCENDING).get()
                .addOnCompleteListener(new OnCompleteListener<QuerySnapshot>() {
                    @Override
                    public void onComplete(@NonNull Task<QuerySnapshot> task) {
                        if(task.isSuccessful()){
                            alarmList = new ArrayList<>();
                            alarmList.clear();
                            for(QueryDocumentSnapshot document : task.getResult()){
                                alarmList.add(new AlarmInfo(
                                        document.getData().get("hour").toString(),
                                        document.getData().get("minute").toString(),
                                        document.getData().get("drugtext").toString(),
                                        document.getData().get("ampm").toString(),
                                        document.getId()
                                ));
                            }
                            myAdapter = new MyAdapter(getActivity(), alarmList);
                            myAdapter.setOnAlarmListener(onAlarmListener);
                            recyclerView.setAdapter(myAdapter);
                            myAdapter.notifyDataSetChanged();
                        }else {
                            Log.d(TAG, "Error : ",task.getException());
                        }
                    }
                });
    }
~~~

MyAdapter.java
~~~java
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder> {
    private ArrayList<AlarmInfo> mDataset;
    private Activity activity;
    private OnAlarmListener onAlarmListener;
    private Button modifybtn;
    private Button deletebtn;
    AlarmManager alarmManager;


    static class MyViewHolder extends RecyclerView.ViewHolder {
        CardView cardView;
        MyViewHolder(Activity activity, CardView v, AlarmInfo alarmInfo) {
            super(v);
            cardView = v;
        }
    }

    public int getItemViewType(int position){
        return position;
    }

    public void setOnAlarmListener(OnAlarmListener onAlarmListener){
        this.onAlarmListener = onAlarmListener;
    }
    MyAdapter(Activity activity, ArrayList<AlarmInfo> mDataset){
        this.mDataset = mDataset;
        this.activity = activity;
    }


    @NonNull
    @Override
    public MyAdapter.MyViewHolder onCreateViewHolder(@NonNull ViewGroup parent, final int viewType) {

        final CardView cardView = (CardView) LayoutInflater.from(parent.getContext()).inflate(R.layout.item_alarm, parent, false);
        final MyViewHolder myViewHolder = new MyViewHolder(activity, cardView, mDataset.get(viewType));
        modifybtn = cardView.findViewById(R.id.modifybtn);
        deletebtn = cardView.findViewById(R.id.deletebtn);

        //수정버튼 클릭시
        modifybtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                onAlarmListener.onModify(myViewHolder.getAdapterPosition());
                //myStartActivity(SettingAlarm.class);
            }
        });

        //삭제버튼 클릭시
        deletebtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                onAlarmListener.onDelete(myViewHolder.getAdapterPosition());
                myStartActivity(MainActivity.class);
            }
        });

        return myViewHolder;
    }

    @Override
    public void onBindViewHolder(@NonNull final MyAdapter.MyViewHolder holder, final int position) {

        final CardView cardView = holder.cardView;
        TextView hourText = cardView.findViewById(R.id.hourText);
        hourText.setText(mDataset.get(position).getHour());
        Log.e("확인확인", String.valueOf(mDataset.get(position).getHour()));

        TextView minuteText = cardView.findViewById(R.id.minuteText);
        minuteText.setText(mDataset.get(position).getMinute());
        Log.e("getMinute", String.valueOf(mDataset.get(position).getMinute()));

        TextView drugText = cardView.findViewById(R.id.drug_text);
        drugText.setText(mDataset.get(position).getDrugText());
        Log.e("getDrugText",mDataset.get(position).getDrugText());

        TextView ampmText = cardView.findViewById(R.id.ampmText);
        ampmText.setText(mDataset.get(position).getAmpm());
        Log.e("getAmpm", mDataset.get(position).getAmpm());

        //modifybtn = cardView.findViewById(R.id.modifybtn);
        //deletebtn = cardView.findViewById(R.id.deletebtn);
    }
    private void myStartActivity (Class c, AlarmReceiver alarmReceiver){//intent를 이용하여 id 값을 전달해줄것임.
        Intent intent = new Intent(activity, c);
        intent.putExtra("alarmInfo", (Parcelable) alarmReceiver);//앞에는 key값, 뒤에는 실제 값
        //id값을 보내주면 WritePostActivity에서 받아서 쓸것임
        activity.startActivity(intent);
    }
    private void myStartActivity(Class c) {//화면 전환을 위한 메서드를 함수로 정의함
        Intent intent = new Intent(activity, c);
        activity.startActivityForResult(intent, 1);
    }
    @Override
    public int getItemCount() {
        return (mDataset !=null ? mDataset.size() :0);
    }


}
~~~



>>2-5-2 푸시알림
#####

~~~java
NotificationChannel channel = new NotificationChannel(channelId, chaanelName, importance);
            channel.setDescription(description);

            if (notificationManager.getNotificationChannel(channelId) == null) {
                notificationManager.createNotificationChannel(channel);
            }

            builder.setSmallIcon(R.drawable.ic_drug_icon);

            builder.setAutoCancel(true)
                    .setWhen(System.currentTimeMillis())
                    .setContentTitle("약쏙")
                    .setContentText(text + "을(를) 복용할시간에요:)")
                    .setPriority(Notification.PRIORITY_DEFAULT)
                    .setContentIntent(contentIntent)
                    .setContentInfo("INFO")
                    .setDefaults(Notification.DEFAULT_VIBRATE);


            //if(notificationManager !=null){


            PowerManager powerManager = (PowerManager) context.getSystemService(Context.POWER_SERVICE);
            PowerManager.WakeLock wakeLock = powerManager.newWakeLock(PowerManager.FULL_WAKE_LOCK | PowerManager.ACQUIRE_CAUSES_WAKEUP | PowerManager.ON_AFTER_RELEASE, "My:Tag"
            );
            wakeLock.acquire(5000);
            notificationManager.notify(Integer.parseInt(notificationid), builder.build());
~~~


ic_drug_icon.png
<img src="https://user-images.githubusercontent.com/62935657/86551712-d9dbc800-bf80-11ea-8deb-d7e3d49fb645.png" width="10%"></img>
를 넣어 푸시알림이 왔을때 위와 같은 icon이 뜨도록 설정했다.



setDefaults(Notification.DEFAULT_VIBRATE); 를 이용하여 푸시알림이 왔을때 진동이 울리게 했다.


~~~java
if (notificationManager != null) {


                PowerManager powerManager = (PowerManager) context.getSystemService(Context.POWER_SERVICE);
                PowerManager.WakeLock wakeLock = powerManager.newWakeLock(PowerManager.FULL_WAKE_LOCK | PowerManager.ACQUIRE_CAUSES_WAKEUP | PowerManager.ON_AFTER_RELEASE, "My:Tag"
                );
                wakeLock.acquire(5000);
                notificationManager.notify(Integer.parseInt(notificationid), builder.build());

            }
~~~
다음과 같은 코드를 이용하여 화면이 꺼져있을때, 화면이 켜지면서 푸시알림이 보일수 있도록 했다.



>>2-5-3 알림삭제
alarmlistener 인터페이스를 만들어서 삭제버튼을 눌렀을때, fragmentAlarm.java로 가서, 알림이 삭제가 될수 있도록 한다.
~~~java
OnAlarmListener onAlarmListener = new OnAlarmListener() {//인터페이스인 OnPostListener를 가져와서 구현해줌
        @Override
        public void onDelete(final int position) {//MainAdapter에 넘겨주기 위한 메서드 작성

            id = alarmList.get(position).getId();//document의 id에 맞게 지워주기 위해 id값을 얻어옴
            firebaseFirestore.collection("AlarmDemo").document(id).delete()//그 id에 맞는 값들을 지워줌
                    .addOnSuccessListener(new OnSuccessListener<Void>() {
                        @Override
                        public void onSuccess(Void aVoid) {//성공시
                            alarmUpdate();

                            Intent intent = new Intent(getActivity(), AlarmReceiver.class);
                            // intent.putExtra("cancelId", alarmList.get(position).getHour()+alarmList.get(position).getMinute());
                            //cancelId = intent.getStringExtra("cancelId");
                            //if (cancelId != null){
                            Log.e("cancel : ","cancel");
                            //alarmManager = (AlarmManager)getSystemService(Context.ALARM_SERVICE);
                            // Intent intent = new Intent(getApplicationContext(), AlarmReceiver.class);
                            PendingIntent pIntent = PendingIntent.getBroadcast(getActivity(),
                                    Integer.parseInt(alarmList.get(position).getHour()+alarmList.get(position).getMinute()),intent, PendingIntent.FLAG_UPDATE_CURRENT);
                            alarmManager.cancel(pIntent);
                            pIntent.cancel();
                            Toast.makeText(getActivity(), "알림이 삭제되었습니다.", Toast.LENGTH_SHORT).show();
                            Log.e("ERROR : ","ERROR");

                            Log.e("DB삭제 성공 : ","성공");

                        }
                    }).addOnFailureListener(new OnFailureListener(){
                @Override
                public void onFailure(@NonNull Exception e) {//실패시
                    startToast("게시글 삭제에 실패하였습니다.");
                }
            });
        }
~~~


        
        
        


