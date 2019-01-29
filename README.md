# AndroidAlarmClockApp


# Screenshots 
<img src="/screenshots/screenshot1.png" width="220" height="400"> <img src="/screenshots/screenshot2.png" width="220" height="400"> 

Notification 
<img src="/screenshots/screenshot3_noti.png" > 

# Step to re-crearte 

1. Create a new project in Android Studio (Choose the Empty Activity)
2. Navigate to app/res/layout/activity_main.xml
3. Open the "Text" view and the replace the xml code with the below code or Change the ConstraintLayout to RelativeLayout. Add TimePicker, ToggleButton and a TextView into your layout

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TimePicker
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/alarmTimePicker"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true" />


    <ToggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Alarm On/Off"
        android:id="@+id/alarmToggle"
        android:layout_centerHorizontal="true"
        android:layout_below="@+id/alarmTimePicker"
        android:onClick="onToggleClicked" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:text=""
        android:id="@+id/alarmText"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="20dp"
        android:layout_below="@+id/alarmToggle" />

</RelativeLayout>
```
4. Create a new Java file name it **AlarmService.java** in the following path app/java/"The first folder"/
5. Replace the code in that file with the below code ("Dont remove the first line starting with package")

```
import android.app.IntentService;
import android.app.NotificationManager;
import android.app.PendingIntent;
import android.content.Context;
import android.content.Intent;
import android.support.v4.app.NotificationCompat;
import android.util.Log;

public class AlarmService extends IntentService {
    private NotificationManager alarmNotificationManager;

    public AlarmService() {
        super("AlarmService");
    }

    @Override
    public void onHandleIntent(Intent intent) {
        sendNotification("Wake Up! Wake Up!");
    }

    private void sendNotification(String msg) {
        Log.d("AlarmService", "Preparing to send notification...: " + msg);
        alarmNotificationManager = (NotificationManager) this
                .getSystemService(Context.NOTIFICATION_SERVICE);

        PendingIntent contentIntent = PendingIntent.getActivity(this, 0,
                new Intent(this, MainActivity.class), 0);

        NotificationCompat.Builder alamNotificationBuilder = new NotificationCompat.Builder(
                this).setContentTitle("Alarm").setSmallIcon(R.mipmap.ic_launcher)
                .setStyle(new NotificationCompat.BigTextStyle().bigText(msg))
                .setContentText(msg);


        alamNotificationBuilder.setContentIntent(contentIntent);
        alarmNotificationManager.notify(1, alamNotificationBuilder.build());
        Log.d("AlarmService", "Notification sent.");
    }
}
```
6.  Create a new Java file name it **AlarmReceiver.java** in the following path app/java/"The first folder"/
7.  Replace the code in that file with the below code ("Dont remove the first line starting with package")
```
import android.app.Activity;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.media.Ringtone;
import android.media.RingtoneManager;
import android.net.Uri;
import android.support.v4.content.WakefulBroadcastReceiver;

public class AlarmReceiver extends WakefulBroadcastReceiver {

    @Override
    public void onReceive(final Context context, Intent intent) {
        //this will update the UI with message
        MainActivity inst = MainActivity.instance();
        inst.setAlarmText("Alarm! Wake up! Wake up!");

        //this will sound the alarm tone
        //this will sound the alarm once, if you wish to
        //raise alarm in loop continuously then use MediaPlayer and setLooping(true)
        Uri alarmUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_ALARM);
        if (alarmUri == null) {
            alarmUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
        }
        Ringtone ringtone = RingtoneManager.getRingtone(context, alarmUri);
        ringtone.play();

        //this will send a notification message
        ComponentName comp = new ComponentName(context.getPackageName(),
                AlarmService.class.getName());
        startWakefulService(context, (intent.setComponent(comp)));
        setResultCode(Activity.RESULT_OK);
    }
}
```

8. Open the java file for the main activity by opening the MainActivity.java from the following path app/java/"The first folder"/MainActivity.java
9. Add the below code before the OnCreate Method 
```
    AlarmManager alarmManager;
    private PendingIntent pendingIntent;
    private TimePicker alarmTimePicker;
    private static MainActivity inst;
    private TextView alarmTextView;

    public static MainActivity instance() {
        return inst;
    }

    @Override
    public void onStart() {
        super.onStart();
        inst = this;
    }
```
10. Inside of the OnCreate Method add the following code 
```
        alarmTimePicker = (TimePicker) findViewById(R.id.alarmTimePicker);
        alarmTextView = (TextView) findViewById(R.id.alarmText);
        ToggleButton alarmToggle = (ToggleButton) findViewById(R.id.alarmToggle);
        alarmManager = (AlarmManager) getSystemService(ALARM_SERVICE);
```
11. After the OnCreate Method add the following code 
```
 public void onToggleClicked(View view) {
        if (((ToggleButton) view).isChecked()) {
            Log.d("MyActivity", "Alarm On");
            Calendar calendar = Calendar.getInstance();
            calendar.set(Calendar.HOUR_OF_DAY, alarmTimePicker.getCurrentHour());
            calendar.set(Calendar.MINUTE, alarmTimePicker.getCurrentMinute());
            Intent myIntent = new Intent(MainActivity.this, AlarmReceiver.class);
            pendingIntent = PendingIntent.getBroadcast(MainActivity.this, 0, myIntent, 0);
            alarmManager.set(AlarmManager.RTC, calendar.getTimeInMillis(), pendingIntent);
        } else {
            alarmManager.cancel(pendingIntent);
            setAlarmText("");
            Log.d("MyActivity", "Alarm Off");
        }
    }

    public void setAlarmText(String alarmText) {
        alarmTextView.setText(alarmText);
    }
```

12. Open the **AndroidManifest.xml** and add the following code before ```<application android:..... ```
The bellow is permission that will allow the app to keep the screen awake 
```
    <uses-permission android:name="android.permission.WAKE_LOCK" />

```
13. In the same file **AndroidManifest.xml**  After the last ```</activity> ``` and before ```</application>``` add the code bellow 
The code bellow is a Service that will run in the background
```
          <service
            android:name=".AlarmService"
            android:enabled="true" />
        <receiver android:name=".AlarmReceiver" />
```

Thank you!
