Activity你真的熟悉吗？看了才知道
字数3812 阅读1896 评论3 喜欢29

学过android的人都知道，activity是最常用的四大组件之一，但你真的了解透彻activity了吗？接下来，本人将从activity的正常和异常生命周期、启动模式、IntentFilter匹配原则、activity的过渡动画等方面做个总结。
一、 activity的生命周期

正常生命周期
1.正常打开单个Activity，然后退出应用：

这种情况是最普通的状况，Activity的生命周期会按照上图从上到下的方式走。即：onCreate --> onStart --> onResume --> 运行--> 按返回键结束程序--> onPause-->onStop-->onDestory
2.打开一个Activity A，然后再打开另一个Activity B

对于A：onCreate --> onStart --> onResume --> A运行 --> A发出打开B的Intent --> onPause-->B可见-->onStop
此时，会打开B，B同样会经历一个完整的Activity生命周期。等B结束，A再度可见的时候，A会经历：onRestart-->onStart-->onResume

    注意：B这个Activity是在A的onPause执行后才变成可见状态的，所以为了不影响B的显示，最好不要在onPause里执行一些耗时操作，可以考虑将这些操作放到onStop里，这时B已经可见了。

异常情况生命周期
情况1.资源相关的系统配置发生改变

资源相关的系统配置发生改变，举个例子。当前Activity处于竖屏状态的时候突然转成横屏，系统配置发生了改变，Activity就会销毁并且重建，其onPause, onStop, onDestory均会被调用。因为实在异常情况下终止的，所以系统会调用onSaveInstanceState来保存当前Activity状态。这个方法是在onStop之前，与onPause没有固定的时序关系。当Activity重建的时候系统会把onSaveInstanceState所保存的Bundle作为对象传递给onRestoreInstanceState和onCreate方法。
情况2：资源内存不足导致低优先级Activity被杀死

Activity优先级
前台Activity——正在和用户交互的Activity，优先级最高
可见但非前台Activity——Activity中弹出的对话框导致Activity可见但无法交互
后台Activity——已经被暂停的Activity，优先级最低
系统内存不足是，会按照以上顺序杀死Activity，并通过onSaveInstanceState和onRestoreInstanceState这两个方法来存储和恢复数据。
二、activity的启动模式

四种启动模式分别是standard（标准模式）、singleTop(栈顶复用模式)、singleTask(栈内复用模式)、singleInstance（单实例模式 - 加强的singleTask模式）
standard

    系统默认启动模式
    不论存在与否，都会重新创建一个新的实例
    多实例实现，谁启动了这个Activity、那么这个Activity就运行在启动它那个Activity所在栈

singleTop

    判断需要启动的Activity是否为任务栈栈顶 ，如果是，则不会重新创建，如果不是，则会重新创建
    不重新创建时候，该Activity的 onNewIntent(Intent intent) 方法会被回调，通过该方法的参数，可以取出当前请求的信息；
    系统可能会杀死该Activity，杀死之后，启动情况与第一次启动相同，所以有必要在onCreate与onNewIntent方法中调用同一个处理数据的方法

        运用场景：常运用于通知栏弹出Notification，点击Notification跳转到指定的Activity，设置singleTop模式

singleTask

    判断Activity所需任务栈内是否已经存在，如果存在，就把该Activity切换到栈顶（会导致在它之上的都会出栈）
    如果所需任务栈都不存在，就会先创建任务栈再创建该Activity
    可以理解为 顶置Activity+singleTop 的模式

        运用场景：可用来退出整个应用。主界面activity设为singleTas模式，要退出应用时转到主activity,从而将主activity之上的activity都清除，然后重写主activity的onNewIntent()方法，在里面加上finish(),即可退出所有activity。这种模式还适用于做浏览器、微博之类的应用

singleInstance

    拥有singleTask的所有特性之外，此模式Activity只能单独地位于一个新的任务栈中
    也就是，Activity启动之后，就会独自在一个新的任务栈中，下次肯定不会重新创建该Activity，除非被系统杀死

        运用场景：这种模式常运用于需要与程序分离的界面，如在SetupWizard（安装向导）中调用紧急呼叫就是适用这种模式

三、Intent和Intent-filter的匹配规则
1、Intent和Intent Filter的介绍

Intent是抽象的数据结构，包含了一系列描述某个操作的数据，使得程序在运行时可以在程序中不同组件间通信或启动不同的应用程序。
可以通过startActivity(Intent)启动一个Activity,sendBroadcast(Intent))发送广播发送给感兴趣的BroadcastReceiver组件,startService(android.content.Intent))启动service，bindService()绑定服务。
Intent Filter顾名思义就是Intent的过滤器，组件通过定义Intent Filter可以决定哪些隐式Intent可以和该组件进行通讯
Intent分为隐式(implicit)Intent和显式(explicit)Intent。
显式Intent通常用于在程序内部组件间通信，已经明确的定义目标组件的信息，所以不需要系统决策哪个目标组件处理Intent，如下

Intent intent =new Intent(CRListDemo.this, GoogleMapDemo.class);
startActivity(intent);

其中CRListDemo和GoogleMapDemo都是用户自定义的组件，
隐式Intent不指明目标组件的class，只定义希望的Action及Data等相关信息，由系统决定使用哪个目标组件。如发送短信
2.Intent和Intent-filter的匹配规则

Android系统通过对Intent和目标组件在AndroidManifest文件中定义的（也可以在程序中定义Intent Filter）进行匹配决定和哪个目标组件通讯。如果某组件未定义则只能通过显式的Intent进行通信。Intent的三个属性用于对目标组件选取的决策，分别是Action、Data（Uri和Data Type）、Category。匹配规则如下

    action匹配规则：要求intent中的action 存在 且 必须和过滤规则中的其中一个相同 区分大小写；
    category匹配规则：系统会默认加上一个android.intent.category.DEAFAULT，所以intent中可以不存在category，但如果存在就必须匹配其中一个；
    data匹配规则：data由两部分组成，mimeType和URI，要求和action相似。如果没有指定URI，URI但默认值为content和file（schema）

3.利用Intent调用其他常见程序

a. 发送短信

Uri uri = Uri.parse("smsto:15800000000");
Intent i=newIntent(Intent.ACTION_SENDTO, uri);
i.putExtra("sms_body", "The SMS text");
startActivity(i);

b. 打电话

Uri dial = Uri.parse("tel:15800000000");
Intent i=newIntent(Intent.ACTION_DIAL, dial);
startActivity(i);

c. 发送邮件

Uri email = Uri.parse("mailto:abc@126.com;def@126.com");
Intent i=newIntent(Intent.ACTION_SENDTO, email);
startActivity(i);

d. 拍照

Intent i =newIntent(MediaStore.ACTION_IMAGE_CAPTURE);
String folderPath= Environment.getExternalStorageDirectory().getAbsolutePath() + File.separator + "AndroidDemo" +File.separator;
String filePath= folderPath + System.currentTimeMillis() + ".jpg";newFile(folderPath).mkdirs();
File camerFile=newFile(filePath);
i.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(camerFile));
startActivityForResult(i,1);

e. 浏览网页

Uri web = Uri.parse("http://www.google.com");
Intent i=newIntent(Intent.ACTION_VIEW, web);
startActivity(i);

f. 查看联系人

Intent i =newIntent();
i.setAction(Intent.ACTION_GET_CONTENT);
i.setType("vnd.android.cursor.item/phone");
startActivityForResult(i,1);

四、activity过渡动画的五种实现
1.使用overridePendingTransition方法实现Activity跳转动画

overridePendingTransition方法是Activity中提供的Activity跳转动画方法，通过该方法可以实现Activity跳转时的动画效果，简单例子如下：

Intent intent =newIntent(MainActivity.this, SecondActivity.class);
startActivity(intent);
overridePendingTransition(R.anim.slide_in_left, R.anim.slide_in_left);

    注意：overridePendingTransition在startActivity或者是finish方法立刻执行才有效

2、使用style的方式定义Activity的切换动画

（1）定义Application的style

<!-- 系统Application定义 -->
<application 
Android:allowBackup="true"
Android:icon="@mipmap/ic_launcher" 
Android:label="@string/app_name" 
Android:supportsRtl="true" 
Android:theme="@style/AppTheme">

（2）定义具体的AppTheme样式
其中这里的windowAnimationStyle就是我们定义Activity切换动画的style。而@anim/slide_in_top就是我们定义的动画文件，也就是说通过为Appliation设置style，然后为windowAnimationStyle设置动画文件就可以全局的为Activity的跳转配置动画效果。

<!-- Base application theme. --> 
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
 <!-- Customize your theme here. --> 
<item name="colorPrimary">@color/colorPrimary</item>
<item name="colorPrimaryDark">@color/colorPrimaryDark</item> 
<item name="colorAccent">@color/colorAccent</item>
<item name="Android:windowAnimationStyle">@style/activityAnim</item>

</style><!-- 使用style方式定义activity切换动画 -->
 <style name="activityAnim">
 <item name="Android:activityOpenEnterAnimation">@anim/slide_in_top</item>
 <item name="Android:activityOpenExitAnimation">@anim/slide_in_top</item> </style>

而在windowAnimationStyle中存在四种动画：

    activityOpenEnterAnimation
    用于设置打开新的Activity并进入新的Activity展示的动画
    activityOpenExitAnimation
    用于设置打开新的Activity并销毁之前的Activity展示的动画
    activityCloseEnterAnimation
    用于设置关闭当前Activity进入上一个Activity展示的动画
    activityCloseExitAnimation
    用于设置关闭当前Activity时展示的动画

3.使用ActivityOptions切换动画实现Activity跳转动画

通过overridePendingTransition方法基本上可以满足我们日常中对Activity跳转动画的需求了，但MD风格出来之后，overridePendingTransition这种老旧、生硬的方式怎么能适合我们的MD风格的App呢？google在新的sdk中给我们提供了另外一种Activity的过度动画——ActivityOptions。并且提供了兼容包——ActivityOptionsCompat。ActivityOptionsCompat是一个静态类，提供了相应的Activity跳转动画效果，通过其可以实现不少炫酷的动画效果。
（1）在跳转的Activity中设置contentFeature

@Override protected void onCreate(Bundle savedInstanceState) { 
super.onCreate(savedInstanceState); 
// 设置contentFeature,可使用切换动画 
getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS); 
Transition explode = TransitionInflater.from(this).inflateTransition(Android.R.transition.explode);
getWindow().setEnterTransition(explode); 
setContentView(R.layout.activity_three); }

（2）在startActivity执行跳转逻辑的时候调用startActivity的重写方法，执行ActivityOptions.makeSceneTransitionAnimation方法

/** * 点击按钮,实现Activity的跳转操作 * 通过Android5.0及以上代码的方式实现activity的跳转动画 */
button3.setOnClickListener(new View.OnClickListener() { 
@Override public void onClick(View v) { 
Intent intent = new Intent(MainActivity.this, ThreeActivity.class); 
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(MainActivity.this).toBundle()); } });


activity跳转动画效果
（四）使用ActivityOptions之后内置的动画效果通过style的方式

这种方式其实就是通过style的方式展示和使用ActivityOptions过度动画，下面是实现通过定义style方式定义过度动画的步骤：
（1）编写过度动画文件

<explode xmlns:Android="http://schemas.Android.com/apk/res/Android" 
Android:duration="300" />

首先我们需要在Application项目res目录下新建一个transition目录，然后创建资源文件，然后使用这些系统自带的过渡动画效果，这里设置了过度时长为300ms。
（2）定义style文件

<!-- Base application theme. --> 
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar"> 
<!-- Customize your theme here. --> 
<item name="colorPrimary">@color/colorPrimary</item>
<item name="colorPrimaryDark">@color/colorPrimaryDark</item> 
<item name="colorAccent">@color/colorAccent</item>
<item name="Android:windowEnterTransition">@transition/activity_explode</item>
<item name="Android:windowExitTransition">@transition/activity_explode</item> 
</style>

在Application的style文件中添加：

<item name="Android:windowEnterTransition">@transition/activity_explode</item>
<item name="Android:windowExitTransition">@transition/activity_explode</item>

并指定过渡动画效果为我们刚刚定义的过渡动画文件。
（3）执行跳转逻辑
点击按钮,实现Activity的跳转操作 * 通过Android5.0及以上style的方式实现activity的跳转动画

button4.setOnClickListener(new View.OnClickListener() 
{ @Override public void onClick(View v) {
 /** * 调用ActivityOptions.makeSceneTransitionAnimation实现过度动画 */ 
Intent intent = new Intent(MainActivity.this, FourActivity.class); 
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(MainActivity.this).toBundle()); 
}
 });

这样执行之后也可以展示出Activity跳转过度动画了，其和通过代码方式实现的效果是类似的，而且这种动画效果是全局的。
（五）使用ActivityOptions动画共享组件的方式实现跳转Activity动画

这里的共享组件动画效果是指将前面一个Activity的某个子View与后面一个Activity的某个子View之间有过渡效果，即在这种过度效果下实现Activity的跳转操作。那么如何实现两个组件View之间实现过渡效果呢？
（1）定义共享组件
在Activity a中的button按钮点击transitionName属性：

<Button Android:id="@+id/button5" 
Android:layout_width="match_parent" 
Android:layout_height="wrap_content" 
Android:layout_below="@+id/button4" 
Android:layout_marginTop="10dp" 
Android:layout_marginRight="10dp" 
Android:layout_marginLeft="10dp" Android:text="组件过度动画" 
Android:background="@color/colorPrimary" 
Android:transitionName="shareNames" />

在Activity b的布局文件中为组件定义transitionName属性，这样这两个组件相当于有了过度对应关系，这里需要注意的是这两个组件的transitionName属性的值必须是相同的。

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
xmlns:Android="http://schemas.Android.com/apk/res/Android" 
Android:id="@+id/activity_second" 
Android:layout_width="match_parent" 
Android:layout_height="match_parent" 
Android:gravity="center_horizontal" Android:orientation="vertical" 
Android:transitionName="shareNames" > <TextView 
Android:layout_width="match_parent" 
Android:layout_height="match_parent" 
Android:background="@color/colorAccent" 
Android:layout_marginTop="10dp" 
Android:layout_marginBottom="10dp" />
</LinearLayout>

（2）调用startActivity执行跳转动画
点击按钮,实现Activity的跳转操作 * 通过Android5.0及以上共享组件的方式实现activity的跳转动画

 button5.setOnClickListener(new View.OnClickListener() { 
@Override public void onClick(View v) { 
Intent intent = new Intent(MainActivity.this, FiveActivity.class); 
startActivity(intent, 
ActivityOptions.makeSceneTransitionAnimation(MainActivity.this, button5, "shareNames").toBundle()); } 
});

需要说明的是这里调用的ActivityOptions.makeSceneTransitionAnimation方法，传递了三个参数，其中第一个参数为context对象，第二个参数为启动Activity的共享组件，第三个参数为启动Activity的共享组件transitionName属性值。
这样经过调用之后我们就实现了从Activity a跳转到Activity b的时候a中的组件到b中组件的过度效果。

这里写图片描述

    过渡动画总结

        overridePendingTransition方法从Android2.0开始，基本上能够覆盖我们activity跳转动画的需求；
        ActivityOptions API是在Android5.0开始的，可以实现一些炫酷的动画效果，更加符合MD风格；ActivityOptions还可以实现两个Activity组件之间的过度动画；
        过渡动画可参考github项目地址：实现activity跳转动画的五种方式

