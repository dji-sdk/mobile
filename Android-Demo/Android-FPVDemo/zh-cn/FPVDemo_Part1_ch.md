# DJI Mobile Android SDK 教程

## 如何创建一个航拍相机App: 第一部分

### 1.准备工作

(1) 下载SDK:
你可以从这里下载到最新的SDK: <http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads>

(2)你可以从这里下载开发者固件，并升级对应飞行器(Phantom 3 Professional, Phantom 3 Advanced 或者 Inspire 1)的固件<http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads>

可以依据文档<http://download.dji-innovations.com/downloads/phantom_3/cn/How_to_Update_Firmware_cn.pdf>里的飞行器固件升级步骤升级。

### 2.解压SDK并导入Lib到自己的工程
(1) 解压SDK包，导入文件夹"Lib"到你的eclipse，把它添加为你的工程的库(把鼠标放在你的工程名上右击->选择“Properties”->选择"Android").
![setLib](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_importLib.png)

(2) 检查库是否已成功导入

查看“Android Private Libraries”，里面有如下jar包时，说明导入成功。

![checkLib](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_CheckLib.png)



### 3.编写显示FPV视频的代码

(1) 激活SDK: 

在工程的AndroidManifest.xml文件中添加以下meta-data元素配置用来激活的APP KEY。
![appKeyMetaData](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_appKeyMetaData.png)

请使用你在DJI开发者网站<http://dev.dji.com>上申请的APP KEY填入**android:value=""**。当申请APP KEY的时候，需要在标识码处填入你的工程的包名。
![appKey](https://github.com/mrjzhao/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/zh-cn/images/appKey_cn.png)
 
在开始调用SDK APIs之前，需要添加以下代码来进行激活验证，

~~~java
	new Thread(){
		public void run(){
			try{
				DJIDrone.checkPermission(getApplicationContext(), new DJIGeneralListener(){
					@Override
					public void onGetPermissionResult(int result){
						if(result == 0) {
							// show success
							Log.e(TAG, "onGetPermissionResult ="+result);
							Log.e(TAG, "onGetPermissionResultDescription="+DJIError.getCheckPermissionErrorDescription(result));
						}else {
							// show error									
							Log.e(TAG, "onGetPermissionResult ="+result);
							Log.e(TAG, "onGetPermissionResultDescription="+DJIError.getCheckPermissionErrorDescription(result));
						}
					}
				});
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}.start();
~~~

只有在激活成功之后，SDK APIs才能被调用成功。激活结果的返回码和相应的描述请见下表，若遇到问题不能成功，请先检查以下几点：1. 在申请APP KEY时，是否是使用工程的包名填入标识码；2. 是否有连接网络； 3. 所使用的APP KEY的最大装机量是否使用完。 若仍未能解决激活问题，可以发邮件给我们Mobile SDK邮箱:<sdk@dji.com>

返回错误码  		 | 错误描述 
------------- | -------------
0   | 验证成功
-1  | 没有网络连接
-2  | 无效的app key
-3  | 获取授权数据超时
-4  | 设备uuid不匹配
-5  | 工程包名和申请app key时的标识码不匹配
-6  | App key被禁
-7  | 激活量已经到达最大装机量
-8  | App key的使用平台不正确
-9  | App key不存在
-10 | App key没有权限
-11 | 服务器解析失败
-12 | 服务器获取设备uuid错误
-13 | 服务器app包名不对
-14 | 服务器解析激活数据失败
-15 | AES 256加密不支持
-16 | AES 256加密失败
-17 | 获取设备uuid失败
-18 | 空的App keyEmpty app key
-1000 | 服务器错误 


(2) 添加对Android Open Accessory (AOA)的支持:

DJI的遥控器最新的固件将需要app端实现对AOA的支持。在Demo程序中，我们提供了对AOA支持的一个样例。在AndroidManifest.xml文件中，使用DJIAoaActivity作为主activity，即是打开app时第一个执行的activity。并添加intent-filter和meta-data。以及在**AndroidManifest.xml**中添加 **uses-feature android:name="android.hardware.usb.accessory" android:required="false"**, **uses-feature android:name="android.hardware.usb.host" android:required="false"**， 和在**application**标签下添加 **uses-library android:name="com.android.future.usb.accessory"**。

~~~xml
	...
	
	<uses-feature android:name="android.hardware.usb.accessory" android:required="false" />
	<uses-feature android:name="android.hardware.usb.host" android:required="false" />
	<application
		android:label="@string/app_name"
		android:theme="@style/AppTheme">
		
		<uses-library android:name="com.android.future.usb.accessory" />
	...
		<activity
			android:name=".DJIAoaActivity"
			android:configChanges="orientation|screenSize|keyboardHidden|keyboard"
			android:screenOrientation="sensorLandscape" >
			
			<intent-filter>
				<action android:name="android.intent.action.MAIN" />
				<category android:name="android.intent.category.LAUNCHER" />
			</intent-filter>
			
			<intent-filter>
				<action android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED" />
			</intent-filter>
			
			<meta-data
				android:name = "android.hardware.usb.action.USB_ACCESSORY_ATTACHED"
				android:resource = "@xml/accessory_filter" />
		</activity>
~~~

DJIAoaActivity中有添加如下代码支持AOA,
 
~~~java
	private static boolean isStarted = false;
	...
	@Override
	protected void onCreate(Bundle savedInstanceState){
		super.onCreate(savedInstanceState);
		setContentView(new View(this));
		
		if (isStarted) {
			//Do nothing
		}else {
			isStarted = true;
			ServiceManager.getInstance();
			UsbAccessoryService.registerAoaReceiver(this); 
			Intent intent = new Intent(DJIAoaActivity.this, FPVActivity.class);
			startActivity(intent);
		}
		
		Intent aoaIntent = getIntent();
		if(aoaIntent != null) {
			String action = aoaIntent.getAction();
			if (action==UsbManager.ACTION_USB_ACCESSORY_ATTACHED || action == Intent.ACTION_MAIN){
				Intent attachedIntent = new Intent();
				attachedIntent.setAction(DJIUsbAccessoryReceiver.ACTION_USB_ACCESSORY_ATTACHED);
				sendBroadcast(attachedIntent);
			}
		}
		finish();
	}
	...
~~~ 

以上代码让程序在打开时运行Service支持AOA连接遥控器，然后我们还需要在每一个Activity运行**onPause()**或是**onResume()**回调接口时，相应的暂停或是继续该Service。你可以通过让Activity继承Demo程序中的**DemoBaseActivity**来实现。**DemoBaseActivity**中的代码片段如下，

~~~java
	...
	@Override
	protected void onResume(){
		super.onResume();
		ServiceManager.getInstance().pauseService(false); // Resume the service
	} 
	
	@Override
	protected void onPause() {
		super.onPause();
		ServiceManager.getInstance().pauseService(true); // Pause the service
	}
~~~	
	
(3) 在APP上实现FPV实时视频显示
	
(a) 在使用SDK APIs之前，我们先需要根据连接的飞机的类型来初始化SDK APIs.使用DJIDrone类里的**public static boolean initWithType(Context mContext, DJIDroneType type)**方法来初始化。

~~~java
	@Override
	protected void onCreate(Bundle savedInstanceState){
		
		...
		DroneCode = 1; 
		onInitSDK(DroneCode);
		...
	}
	
	private void onInitSDK(int type){
		switch(type){
			case 0: {
				DJIDrone.initWithType(this.getApplicationContext(), DJIDroneType.DJIDrone_Vision);
				// The SDK initiation for Phantom 2 Vision or Vision Plus 
				break;
			}
			case 1: {
				DJIDrone.initWithType(this.getApplicationContext(), DJIDroneType.DJIDrone_Inspire1); 
				// The SDK initiation for Inspire 1 or Phantom 3 Professional.
				break;
			}
			case 2: {
				DJIDrone.initWithType(this.getApplicationContext(), DJIDroneType.DJIDrone_Phantom3_Advanced);
				// The SDK initiation for Phantom 3 Advanced
				break;
			}
			case 3: {
				DJIDrone.initWithType(this.getApplicationContext(), DJIDroneType.DJIDrone_M100);
				// The SDK initiation for Matrice 100.
				break;
			}
			default:{
				break;
			}
	}
	
	...
~~~	

(b) 在初始化SDK APIs之后，我需要使用DJIDrone类里面的**public static boolean connectToDrone()**方法连接飞机，

~~~java
	DJIDrone.connectToDrone(); // Connect to the drone
~~~	

(c) 现在我们可以实例化一个视频数据的回调接口**DJIReceivedVideoDataCallBack()**, 然后调用API **public public void setReceivedVideoDataCallBack(DJIReceivedVideoDataCallBack mReceivedVideoDataCallBack)**来获取实时视频数据（raw H264格式）。用户可以实现自己的代码去处理该视频数据。这里，我们使用DJI SDK提供的解码器来解码该视频数据，并把解码出来的视频显示在**SurfaceView**。

在 layout **activity_fpv.xml**里面添加surfaceview, 该layout是**FPVActivity**的界面文件，

~~~xml
	<dji.sdk.widget.DjiGLSurfaceView
		android:id="@+id/DjiSurfaceView_02"
		android:layout_width="fill_parent"
		android:layout_height="fill_parent" />
~~~
		
添加代码把视频数据发送到类**DjiGLSurfaceView**实例化的一个对象，该对象对其进行解码并进行显示。		

~~~java
	...
	
	private DjiGLSurfaceView mDjiGLSurfaceView;
	
	...
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_fpv);
		
		...
		
		mDjiGLSurfaceView = (DjiGLSurfaceView)findViewById(R.id.DjiSurfaceView_02);
		mDjiGLSurfaceView.start();
		mReceivedVideoDataCallBack = new DJIReceivedVideoDataCallBack(){
			@Override
			public void onResult(byte[] videoBuffer, int size){
				mDjiGLSurfaceView.setDataToDecoder(videoBuffer, size);
			}
		};
		DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(mReceivedVideoDataCallBack);
		
		...
		
		
	}
~~~	

**注意**：需要先调用实例化对象**mDjiGLSurfaceView**的**public boolean start()**方法，然后调用**DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(mReceivedVideoDataCallBack)**把视频数据传递给**mDjiGLSurfaceView**解码并显示视频。

当显示视频的activity关闭时，你需要先调用**DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(null)**停止传递视频数据给**mDjiGLSurfaceView**, 然后再释放**mDjiSurfaceView**。相关代码如下，

~~~java
	...
	
	@Override
	protected void onDestroy() {
		if (DJIDrone.getDjiCamera() != null) {
			DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(null);
		}
		mDjiGLSurfaceView.destroy();
		
		...
	}
~~~

你应该注意调用**mDjiGLSurfaceView**的方法**public boolean start()**和设置视频数据回调接口给**mDjiGLSurfaceView**传递视频数据的顺序，以及释放**mDjiGLSurfaceView**和停止传递数据给它的顺序。

(d) 编译并运行你的工程，检查一切是否正常。如果在运行后看到移动端出现如下界面，你就可以开始用你自己的APP连接飞机，并享受飞机实时传回航拍视频的乐趣了！
![afterCompileScreenShot](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/afterComplileScreenShot.png)

### 4. 连接飞行器
完成以上步骤后, 现在就可以连接你的移动设备到DJI飞行器上，检查是否获取到FPV画面，以下是连接指引：

* 连接 Inspire 1, Phantom 3 Professional 或者 Phantom 3 Advanced:

1. 首先启动遥控器电源, 再启动你的飞行器

2. 使用USB数据线将移动设备连接到遥控器上，当弹出为USB设备选择一个应用时，选择你自己开发的应用

3. 当弹出“是否允许应用连接USB设备”时，选择允许。

4. 点击“OK”当激活成功。

5. 飞行器相机的实时视频就会显示在你的移动设备上。

* 连接 DJI Phantom 2 Vision+ or Phantom 2 Vision:

1. 首先启动遥控器电源, 再启动你的飞行器

2. 确认移动设备有互联网连接，打开应用激活DJI SDK。当激活成功点击“OK”。

3. 启动Wi-Fi中继器电源

4. 在你的移动设备上启动 WIFI ，然后连接到名字类似Phantom-xxxxxx (xxxxxx是你的中继器SSID数字)的WIFI网络上

5. 飞行器相机的实时视频就会显示在你的移动设备上

### 5.享受FPV视图
如果你可以在app中看到飞机的视频流，那么恭喜，你已经完成了第一部分教程的内容了！下图是app的截屏：
![runAppScreenShot](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/runAppScreenShot.png)

### 6.现在要怎么做?

你可以从这里下载到本教程的Demo工程：<https://github.com/DJI-Mobile-SDK/Android-FPVDemo-Part1.git>

你已经学会了如何配置DJI Mobile SDK的Android开发环境，并成功用它开发app来展示飞行器相机的FPV画面。在接下来的教程中，我们会在此基础上添加拍照和录像功能. 请关注我们第二部分的教程，希望你喜欢！
