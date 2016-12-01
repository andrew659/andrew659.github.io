---
layout: post
title: All you ever wanted to know about Android wake locks
date: '2016-12-01 11:34'
categories:
  - research-paper
  - performance
---

<style>
table {
	color:#333333;
	border-width: 1px;
	border-color: #666666;
	border-collapse: collapse;
  margin-bottom: 10px;
}
th {
	border-width: 1px;
	padding: 8px;
	border-style: solid;
	border-color: #666666;
	background-color: #dedede;
}
td {
	border-width: 1px;
	padding-left: 10px;
  padding-right: 10px;
  padding-top: 2px;
  padding-bottom: 2px;
	border-style: solid;
	border-color: #666666;
	background-color: #ffffff;
}
</style>
### Basics about wake locks [(skip)](#header2)
Mobile devices that are left idle can quickly fall asleep to save energy. Android [wake lock](https://developer.android.com/reference/android/os/PowerManager.html) mechanism allows developers to specify the need of keeping certain device hardware awake to run long-running tasks (e.g., large file downloading).

To use a wake lock, developers need to write the following code to specify the lock type and certain flags (see [Android API guide](https://developer.android.com/reference/android/os/PowerManager.html)):

```java
PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE);
 PowerManager.WakeLock wl = pm.newWakeLock(PowerManager.SCREEN_DIM_WAKE_LOCK, "My Tag");
 wl.acquire();
   ..screen will stay on during this section..
 wl.release();
```

In the early days, Android supports quite a few types of wake locks:

| Type                    | CPU | Screen | Keyboard |
|:------------------------|:---:|:------:|:--------:|
| partial wake lock       | on  |  off   |   off    |
| screen dim wake lock    | on  |  dim   |   off    |
| screen bright wake lock | on  | bright |   off    |
| full wake lock          | on  | bright |  bright  |

As the platform evolves, the full and screen wake locks are deprecated. To keep screen awake, Google suggests using the `FLAG_KEEP_SCREEN_ON` in activities:

```java
public class MainActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
  }
```

or declare the `android:keepScreenOn` attribute in the application layout XML files:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:keepScreenOn="true">
    ...
</RelativeLayout>
```

Then the system will manage the screen status according to the lifecycle of activity components (screen will be kept on when the activities are visible). This eases app development and avoids mistakes in wake lock uses.

### How do developers use wake locks in practice? {#header2}
In our recent [research paper](http://sccpu2.cse.ust.hk/andrewust/files/FSE2016.pdf), we studied the real uses of wake locks by analyzing 44,736 Android apps \([download](http://sccpu2.cse.ust.hk/elite/downloadApks.html)\). In particular, we studied the following research questions:

- At what program points are wake locks often acquired and released?
- What computational tasks are often protected by wake locks?
- Are there common patterns of wake locks misuses?

We made quite a few `important findings`. We discuss some examples here. Full findings available in our [paper](http://sccpu2.cse.ust.hk/andrewust/files/FSE2016.pdf) and [technical report](http://repository.ust.hk/ir/bitstream/1783.1-81288/1/wakelock_TR15.pdf).

1. Most apps acquire wake locks in [broadcast receivers](https://developer.android.com/reference/android/content/BroadcastReceiver.html).
2. Wake locks are commonly acquired and released in major lifecycle event handlers of app components \(see [activity lifecycle](https://developer.android.com/guide/components/activities.html)\), but can also be acquired and released at various other program points depending on app functionalities and specific needs.
3. Wake locks are often used to protect 13 types of critical computation, many of which are run asynchronously and could be long running (e.g., location sensing, media playing) and can bring users obvious and perceptible benefits (via certain APIs and system calls).

Networking & Communications | Logging & file I/O            | Asynchronous computation  |
UI & graphics rendering     | Inter-component communication | Data management & sharing |
System-level operations     | Media & audio                 | Security & privacy        |
Sensing operations          | Alarm & notifications         | System setting            |
Telephony services          |

We also observed eight common patterns of `wake lock misuses`. Details and examples are provided in our [paper](http://sccpu2.cse.ust.hk/andrewust/files/FSE2016.pdf).

1. `Unnecessary wake up`: wake locks are acquired too early or released too late. Consequence: energy waste.
2. `Wake lock leakage`: acquired wake locks are forgotten to be released. Consequence: energy waste.
3. `Premature lock releasing`: wake locks are released before being acquired. Consequence: app crash.
4. `Multiple lock acquisition`: wake locks are acquired too many times (e.g., when the acquiring operation resides in a frequently-invoked callback), exceeding the limit allowed by Android OS. Consequence: app crash.
5. `Inappropriate lock type`: using a wake lock with a wake level that is too high may waste energy, while using a wake lock with a wake level that is too low may cause app instability.
6. `Problematic timeout setting`: using wake locks with a too long timeout value may waste energy, while using wake locks with a too short timeout value may cause app instability.
7. `Inappropriate flags`: wake lock flags also need to be carefully set. Using Inappropriate flags \(e.g., [ON_AFTER_RELEASE](https://developer.android.com/reference/android/os/PowerManager.html#ON_AFTER_RELEASE)\) can cause energy waste.
8. `Permission error`: using wake locks without obtaining the [WAKE_LOCK](https://developer.android.com/reference/android/Manifest.permission.html#WAKE_LOCK) permission will crash an app.

To detect wake locks misuses, we also designed a static analysis tool Elite, which performs data flow analysis to infer an app's necessity of using wake locks. If you are interested, try our [prototype](http://sccpu2.cse.ust.hk/elite/tool.html).

### References
1. Yepang Liu, Chang Xu, S.C. Cheung, and Valerio Terragni. Understanding and Detecting Wake Lock Misuses for Android Applications. In Proceedings of the 24th ACM SIGSOFT International Symposium on the Foundations of Software Engineering (FSE 2016), Seattle, WA, USA, November 2016.
2. Yepang Liu, Chang Xu, S.C. Cheung, and Valerio Terragni. How Do Developers Use Wake Locks in Android Applications? A Large-Scale Empirical Study. HKUST Technical Report, HKUST-CS15-04.
