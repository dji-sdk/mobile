# DJI Mobile Android SDK Tutorial

## How to create a Camera Application: Part 1

### 1.Preparation

1.1 Download the SDK:

Download the Mobile SDK for Android from the [developer download site](http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads)

1.2 Update the firmware of your drone (Phantom 3 Professional, Phantom 3 Advanced or Inspire 1):

The latest developer firmware can also be downloaded from our [developer download site](http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads), 
and we also provide a reference manual about [How to update the Aircraft Firmware](http://download.dji-innovations.com/downloads/phantom_3/en/How_to_Update_Firmware_en.pdf) to help you update your drone.

### 2.Unzip the SDK package and import Lib

2.1 Unzip the SDK package. 

Import the folder **Lib** into eclipse, add it as a library for your own project (Right click on your project->Select "**Properties**"->Select "**Android**").

![setLib](../../images/1_importLib.png)

2.2 Locate the imported library.

![checkLib](../../images/1_CheckLib.png)

### 3.Implementing for FPV View

3.1 Activate the SDK: 

Add the highlighted meta-data elements into your **AndroidManifest** for activation.

![appKeyMetaData](../../images/1_appKeyMetaData.png)

Input the APP KEY that you have applied from <http://dev.dji.com>. Note that the Identification Code is identical to your project's package name.

![appKey](../../images/1_appKey.png)
 
Add the following codes before calling the SDK APIs,
```java
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
						// show errors
						Log.e(TAG, "onGetPermissionResult ="+result);
						Log.e(TAG, "onGetPermissionResultDescription="+DJIError.getCheckPermissionErrorDescription(result));
					}
				}
			});
		}
	}
}
```

You are required to complete the activation procedure before your first time of using SDK API and you can check the error codes about the APP KEY activation result. 

If you failed acitvation your project, please refer to the following instructions for troubleshooting.

1. Esnure you have access to the internet;
2. Ensure the project package name is identical to the Identification Code when applying the APP KEY; 
3. Ensure that APP KEY has not reach its installed capacity limit.

result  	  | Description 
------------- | -------------
0   | Check permission successful
-1  | Cannot connect to Internet
-2  | Invalid app key
-3  | Get permission data timeout
-4  | Device uuid not match
-5  | Project package name does not match the app 	   key's identification code
-6  | App key is forbidden
-7  | Activated device number is up to the maximum 		available one
-8  | App key's platform is not correct
-9  | App key does not exist
-10 | App key has no permission
-11 | Server parser failed
-12 | Error in server obtaining uuid
-13 | Server app package name abnormal
-14 | Server parsing activation data failed
-15 | AES 256 encryption unsupported
-16 | AES 256 encryption failed
-17 | Get device uuid failed
-18 | Empty app key
-1000 | Server error 

If you have further questions, contact our mobile SDK support by sending emails to <sdk@dji.com>

3.2 Add the Android Open Accessory (AOA) support:

AOA is required to support the new remote controller from DJI.

Modify **AndroidManifest.xml** to set **DJIAoaActivity** as the main activity, which is served the entry of application. 
And add `uses-feature android:name="android.hardware.usb.accessory"`, `android:required="false"`, `uses-feature android:name="android.hardware.usb.host"`, `android:required="false"` in **AndroidManifest.xml**. 
Add `uses-library android:name="com.android.future.usb.accessory"` under element application.

```xml


<uses-feature android:name="android.hardware.usb.accessory" android:required="false" />
<uses-feature android:name="android.hardware.usb.host" android:required="false" />
<application
	android:label="@string/app_name"
	android:theme="@style/AppTheme">
	
	<uses-library android:name="com.android.future.usb.accessory" />

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
```

Added the following code in `DJIAoaActivity` to enable the AOA support.
```java
private static boolean isStarted = false;

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
```

Add the following codes to pause or resume the AOA data connection service when the `onPause()` or `onResume()` lifecycle callback is called. Set the `DemoBaseActivity` as the base activity, which contains the following codes:

```java
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
```
	
3.3 Implementing the FPV View
	
(a) Initiate the SDK API according to the type of the drone.  
```java
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
```	

(b) Connect to the drone using the following codes:

````java
	DJIDrone.connectToDrone(); // Connect to the drone
```

(c) Implement the FPV view by using the SDK API `public void setReceivedVideoDataCallBack(DJIReceivedVideoDataCallBack mReceivedVideoDataCallBack)` to obtain the live preview video data(raw H264 format). The video data can then be processed by using codes for specific purpose. Here, we use the decoder provided by DJI to decode the video data and display them to the live view using `SurfaceView`.

Add the `surfaceview` in the layout `activity_fpv.xml` in the sample code, which is used as the `FPVActivity`'s view,

```xml
	<dji.sdk.widget.DjiGLSurfaceView
		android:id="@+id/DjiSurfaceView_02"
		android:layout_width="fill_parent"
		android:layout_height="fill_parent" />
```	
	
Add the code to send video data to `DjiGLSurfaceView` for decoding and displaying. 

```java
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
```	
**Note**:

`mDjiGLSurfaceView` should be started first, follow by calling the  `DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(mReceivedVideoDataCallBack)` to send the video data to `mDjiGLSurfaceView` for decoding and displaying.

After activity is closed, you should first call `DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(null)` to reset the callback and then destroy `mDjiGLSurfaceView` to complete the life cycle.

```java
...

@Override
protected void onDestroy() {
	if (DJIDrone.getDjiCamera() != null) {
		DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(null);
	}
	mDjiGLSurfaceView.destroy();
	
	...
}
```

**Note**:

Be aware of the sequence of the start of `mDjiGLSurfaceView` and the setting of the callback, and the sequence of the destroy of `mDjiGLSurfaceView` and setting the callback using null.

3.4 Build and run the project:

Check if everything is running as desired.
If the follow screen can be seen, it means that you have succesfully created an simple app by using the SDK APIs.
![afterCompileScreenShot](../../images/afterComplileScreenShot.png)

### 4.Connect to your DJI Drones

After you build and run the project successfully, you can now connect your mobile device to your drone to check the FPV View. Follow the instruction to check your FPV View app:

#### Connect to a DJI Inspire 1 or Phantom 3 Professional/Advanced:

1. Turn on your remote controller, then turn on your drone.

2. Connect your mobile device to the remote controller using a USB cable. Tap your own app and a message windows of "Choose an app for the USB device" prompts.

3. Tap "OK" when the messagte window prompts "Allow the app to access the USB accessory".

4. Tap "OK" when the activation alert displays.

5. Now you start using FPV View app. 

#### Connect to a DJI Phantom 2 Vision+ or Phantom 2 Vision:
	
1. Turn on your remote controller, then turn on your drone.

2. Ensure that the mobile device have access to the Internet. Tap the app to activate and select "OK" when the activation is done.

3. Turn on the Wi-Fi range extender

4. Turn on the Wi-Fi on your mobile device and connect to the Wi-Fi network of Phantom-xxxxxx (where xxxxxx is your range extender’s SSID number)

5. Now you start using FPV View app.

### 5.Check the result of FPV View

If you can see the live video stream in the app, congratulations! You can move on to the Part 2 of the tutorial now:
![runAppScreenShot](../../images/runAppScreenShot.png)

### 6.Where to Go From Here?

You can download the demo project for this tutorial from here: <https://github.com/DJI-Mobile-SDK/Android-FPVDemo-Part1.git>

You’ve learned how to setup the DJI Mobile SDK's developmengt environment and use it to show the FPV view from the drone's camera. 

We will add the capture and record functions in the app in next part of this tutorial. 

Hope you enjoy it!

---

## How to create a Camera Application: Part 2

In Part 1, you have implemented a basic FPV view application, which allows you to see the live video stream from drone's camera. In part 2, we use Inspire 1 as an example to show how to add photo taking and video recording features in your app. Let's get started!

### 1. Implement the capture function

The `private void captureAction()` function is used to take photos. Whenever the `Capture` button is pressed, the function will be called to taking a photo. 

Following is the code to implement this function:

```java
// function for taking photo
private void captureAction(){
       
    CameraMode cameraMode = CameraMode.Camera_Capture_Mode;
    // Set the cameraMode as Camera_Capture_Mode. All the available modes can be seen in
    // DJICameraSettingsTypeDef.java
    DJIDrone.getDjiCamera().setCameraMode(cameraMode, new DJIExecuteResultCallback(){

        @Override
        public void onResult(DJIError mErr)
        {
            
            String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
            if (mErr.errorCode == DJIError.RESULT_OK) {
                CameraCaptureMode photoMode = CameraCaptureMode.Camera_Single_Capture; 
                // Set the camera capture mode as Camera_Single_Capture. All the available modes 
                // can be seen in DJICameraSettingsTypeDef.java
                
                DJIDrone.getDjiCamera().startTakePhoto(photoMode, new DJIExecuteResultCallback(){
                    @Override
                    public void onResult(DJIError mErr)
                    {
                        
                        String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
                        handler.sendMessage(handler.obtainMessage(SHOWTOAST, result));  // display the returned message in the callback               
                    }
                        
                }); 
                // Execute the startTakePhoto API if setting the camera mode as Camera_Capture_Mode successfully.
            } else {
                handler.sendMessage(handler.obtainMessage(SHOWTOAST, result)); 
                // Show the error when setting fails
            }
                
        }
            
    });
                   
}
```

The `captureAction()` method consists of two parts - the first part sets the camera mode, and the second part sets the camera capture mode and takes a photo. We presents the code for the second step in `onResult()` to make sure the second part runs after the first part has been successfully executed.

Build and run the project and try the capture function, if the screen flashes after your press the capture button, it shows that capture feature is functioning. 

### 2. Implement the recording function

Similar to the implementation of the capture function, the `private void recordAction()` function is implemented for recording videos. When a user presses the "Record" button, the function will be called and the app starts recording video.

Following is the code for this function:

```java
// function for starting recording
private void recordAction(){
    // Set the cameraMode as Camera_Record_Mode.
    CameraMode cameraMode = CameraMode.Camera_Record_Mode;
    DJIDrone.getDjiCamera().setCameraMode(cameraMode, new DJIExecuteResultCallback(){
		 
        @Override
        public void onResult(DJIError mErr)
        {
            
            String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
            if (mErr.errorCode == DJIError.RESULT_OK) {
                   
                //Call the startRecord API
                DJIDrone.getDjiCamera().startRecord(new DJIExecuteResultCallback(){

                    @Override
                    public void onResult(DJIError mErr)
                    {
                        
                        String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
                        handler.sendMessage(handler.obtainMessage(SHOWTOAST, result));  // display the returned message in the callback               
                         
                    }
                        
                }); // Execute the startTakePhoto API
            } else {
                handler.sendMessage(handler.obtainMessage(SHOWTOAST, result));
            }
                
        }
        
    });
        
}
```

### 3. Implement the stopping recording function

The `private void stopRecord()` function is used to stop recording video. 
When a user presses the "Stop recording" button, this function will be called and the app stops recording video. 

Following is the code for this function:

```java
// function for stopping recording
private void stopRecord(){
    // Call the API
    DJIDrone.getDjiCamera().stopRecord(new DJIExecuteResultCallback(){
        @Override
        public void onResult(DJIError mErr)
        {
            
            String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
            handler.sendMessage(handler.obtainMessage(SHOWTOAST, result));

        }
            
    });
   
}

```

Now, we can build and run the project to verify the functions. You may try out the Record function. If a obtain screenshoot similar to the scrrenshot as folloing.

Then congratualtions! Your Aerial FPV Android app is complete. You can use this app to control the camera of your Inspire 1.

![recordVideoScreenShot](../../images/recordVideo.png)

### 4.Where to Go From Here?

You can download the final project for this tutorial here: <https://github.com/DJI-Mobile-SDK/Android-FPVDemo-Part2.git>

You’ve come a long way in this tutorial: you’ve learned how to use the DJI Mobile SDK to show the FPV view of the aircraft's camera and control the camera of a DJI platform. These are the most basic and common features in a typical drone mobile app: `Capture` and `Record`. 

However, if you want to create a drone app that is more fancy, you still have a long way to go. More advanced features would include previewing the photo and video in the SD Card, showing the OSD data of the aircraft and so on. 

Hope you enjoy this tutorial, and stay tuned for our next one!

