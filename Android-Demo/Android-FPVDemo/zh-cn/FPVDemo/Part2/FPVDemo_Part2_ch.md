# DJI Mobile Android SDK 教程

## 如何创建一个航拍相机App: 第二部分

在第一部分中，我们实现了一个基本的FPV视图，你可以在这个视图上看到飞行器摄像头的视频画面。在本教程中，我们使用Phantom 3 Professional进行演示，你会学到如何在app中添加拍照和录像功能，我们开始吧！

### 1. 实现拍照功能

我们在FPVActivity中实现了方法**private void captureAction()**来进行拍照。当按下按钮“Capture”, 该方法将被调用，将拍下一张照片存储在飞机上的SD卡上。**private void captureAction()**方法的代码如下，

~~~java
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
                        
                    }); // Execute the startTakePhoto API if successfully setting the camera mode as
                    	// Camera_Capture_Mode 
                } else {
                    handler.sendMessage(handler.obtainMessage(SHOWTOAST, result)); 
                    // Show the error when setting fails
                }
                
            }
            
        });
                   
    }
~~~

在方法**private void captureAction()**中有两步，第一步是设置相机模式；第二步是设置相机拍照模式并拍照。我们把第二步的相关代码放在第一步的回调接口的**onResult()**方法里面，这样可以保证成功运行完第一步后再执行第二步。

编译并运行工程，连接飞行器试一下拍照功能，当在按下"capture"按钮后，屏幕闪了一下就说明拍照功能可以使用了。


### 2. 实现录像功能

与实现拍照功能类似，我们实现了方法**private void recordAction()**来进行录像功能。当按下“Record”按钮,该方法将被调用，飞机上的相机开始录像。代码如下，
 
~~~java
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
~~~ 
  
### 3. 实现停止录像功能

在FPVActivity中，方法**private void stopRecord()** 被实现用来停止录像。当按下按钮“Stop recording”,该方法将被调用，飞机上的相机将停止录像。代码如下，

~~~java
	 // function for stopping recording
    private void stopRecord(){
    // Call the API
        DJIDrone.getDjiCamera().stopRecord(new DJIExecuteResultCallback(){

            @Override
            public void onResult(DJIError mErr)
            {
                
                String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
                handler.sendMessage(handler.obtainMessage(SHOWTOAST, result));
// Show the return of the API calling
            }
            
        });
    }

~~~

现在，我们可以编译并运行工程，检查我们开始录像，停止录像功能了。如果一切正常，你将看到类似如下的截屏，
![recordVideoScreenShot](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/recordVideo.png)

恭喜你！你的FPV航拍App已经大功告成，你现在可以用它来控制Inspire 1的相机了。

### 4. 现在要怎么做?

你可以从这里下载到本教程的最终Demo工程（与第一部分是同一个工程）: <https://github.com/DJI-Mobile-SDK/Android-FPVDemo-Part2>

你已经完成了整篇教程的学习: 学会如何使用DJI Mobile SDK来开发app，展示飞行器相机的FPV视图，控制DJI 飞行器的相机进行**拍照**和**录像**操作，这两项功能经常被使用到，也是一款航拍app的基本功能点。但是，要开发一款很酷的航拍app，你还有很长的一段路要走。像预览SD卡中的照片和视频，展示飞机的OSD数据等等。请继续关注我们后续的教程，希望你喜欢！
