# DJI Mobile iOS SDK 教程


## 如何创建一个航拍相机App: 第二部分

#####在[第一部分](../Part1/FPVDemo_Part1_ch.html)中, 我们实现了一个基本的FPV视图，你可以在这个视图上看到飞行器摄像头的视频画面. 在本教程中, 我们使用Inspire 1进行演示, 你会学到如何在app中添加拍照和录像功能, 我们开始吧!**

### 1. 实现拍照功能

添加以下代码到 **captureAction** 方法中:

~~~objc
- (IBAction)captureAction:(id)sender {
    
    [_camera startTakePhoto:CameraSingleCapture withResult:^(DJIError *error) {
        if (error.errorCode != ERR_Successed) {
            NSLog(@"Take Photo Error : %@", error.errorDescription);
        }
    }];
}
~~~
   调用**DJICamera**的以下方法:
   
   -(void) startTakePhoto:(CameraCaptureMode)captureMode withResult:(DJIExecuteResultBlock)block;
   
  **CameraCaptureMode** 有以下四种类型: 
  
~~~objc
  /**
 *  Camera capture mode
 */
    typedef NS_ENUM(uint8_t, CameraCaptureMode){
    /**
     *  Single capture
     */
    CameraSingleCapture,
    /**
     *  Multiple capture
     */
    CameraMultiCapture,
    /**
     *  Continuous capture
     */
    CameraContinousCapture,
    /**
     *  AEB capture. Support in Inspire 1/Phantom3 professional/Phantom3 Advanced
     */
    CameraAEBCapture,
};
~~~
  
  它们提供了多种方式进行拍照, **CameraSingleCapture** 使用起来很简单, 你不需要在调用**startTakePhoto**方法之前，设置其它参数. 对于 **CameraMultiCapture**, 你需要使用 **DJICamera.h**的  
  **-(void) setMultiCaptureCount:(CameraMultiCaptureCount)count withResultBlock:(DJIExecuteResultBlock)block** 方法来设置多拍参数，并且在block中检查是否设置成功，然后才能调用**startTakePhoto**方法。
  
  更多信息, 请查看 **DJICamera.h** 和 **DJICameraSettingsDef.h** 文件。
    
---
######注意: 因为 DJICamera 有多个子类: DJIInspireCamera, DJIPhantom3AdvancedCamera, DJIPhantomCamera等等, 你需要找到对应的方法来设置拍照参数. 例如, Inspire 1支持CameraAEBCapture 模式, 所以你应该在DJIInspireCamera.h文件中寻找AEB的设置方法，而不是DJICamera.h.

---

  这里我们设置拍照模式为 **CameraSingleCapture**. 你可以在block中的**DJIError** 变量检查拍照结果。
  
  编译运行你的工程，尝试下拍照功能，如果屏幕在你点击**拍照**按钮后，闪烁了一下，表示拍照成功！到此，你已经完成了拍照功能。  
  
### 2. 实现录像功能
  
##### 1. 切换相机模式Camera Mode
   在实现录像功能之前，我们需要先切换相机模式。   
   我们先检查下**DJICameraSettingsDef.h**文件：
   
~~~objc
   /**
 *  Camera work mode. Used in Inspire/Phantom3 professional/Phantom3 Advanced
 */
    typedef NS_ENUM(uint8_t, CameraWorkMode){
    /**
     *  Capture mode. In this mode, user could do capture action only.
     */
    CameraWorkModeCapture                   = 0x00,
    /**
     *  Record mode. In this mode, user could do record action only.
     */
    CameraWorkModeRecord                    = 0x01,
    /**
     *  Playback mode. In this mode, user could preview photos or videos and delete the file.
     */
    CameraWorkModePlayback                  = 0x02,
    /**
     *  Download mode. In this mode, user could download the selected file from SD card
     */
    CameraWorkModeDownload                  = 0x03,
    /**
     *  Unknown
     */
    CameraWorkModeUnknown                   = 0xFF
};
~~~
   如你所见, 有5种**CameraWorkMode**, 这里我们只使用 **CameraWorkModeCapture** 和 **CameraWorkModeRecord** 两种相机模式. 因为这里使用的是Inspire 1机型, 所以我们要用**DJIInspireCamera.h**文件中的
   
   -(void) setCameraWorkMode:(CameraWorkMode)mode withResult:(DJIExecuteResultBlock)block 方法来切换相机模式。
   
   打开 **Main.storyboard**，为UISegmented Control控件添加一个 IBOutlet, 命名为 "changeWorkModeSegmentControl". 还记得**第一部分教程**中DJICameraDelegate 的一个委托方法吗？
      
   -(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState;
   
   我们可以用这个委托方法，在切换**CameraWorkModeCapture** 和 **CameraWorkModeRecord** 相机模式时，更新segmented control的状态。
      
~~~objc
-(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState
{
    if (_drone.droneType == DJIDrone_Inspire) {
        
        //Update UISegmented Control's state        
        if (systemState.workMode == CameraWorkModeCapture) {
            [self.changeWorkModeSegmentControl setSelectedSegmentIndex:0];
        }else if (systemState.workMode == CameraWorkModeRecord){
            [self.changeWorkModeSegmentControl setSelectedSegmentIndex:1];
        }
    }
}

~~~

 然后我们按以下方式实现 **changeWorkModeAction** 方法:
 
~~~objc
- (IBAction)changeWorkModeAction:(id)sender {
    
    DJIInspireCamera* inspireCamera = (DJIInspireCamera*)_camera;
    __weak DJICameraViewController *weakSelf = self;

    UISegmentedControl *segmentControl = (UISegmentedControl *)sender;
    if (segmentControl.selectedSegmentIndex == 0) { //CaptureMode
        
        [inspireCamera setCameraWorkMode:CameraWorkModeCapture withResult:^(DJIError *error) {
            
            if (error.errorCode != ERR_Successed) {
                UIAlertView *errorAlert = [[UIAlertView alloc] initWithTitle:@"Set CameraWorkModeCapture Failed" message:error.errorDescription delegate:weakSelf cancelButtonTitle:@"OK" otherButtonTitles:nil];
                [errorAlert show];
            }
            
        }];
        
    }else if (segmentControl.selectedSegmentIndex == 1){ //RecordMode
    
        [inspireCamera setCameraWorkMode:CameraWorkModeRecord withResult:^(DJIError *error) {
            
            if (error.errorCode != ERR_Successed) {
                UIAlertView *errorAlert = [[UIAlertView alloc] initWithTitle:@"Set CameraWorkModeRecord Failed" message:error.errorDescription delegate:weakSelf cancelButtonTitle:@"OK" otherButtonTitles:nil];
                [errorAlert show];
            }
        
        }];
        
    }
    
}

~~~
 这里添加了两个UIAlertView， 在设置相机模式失败时提醒用户。
  
##### 2. 添加录像按钮

  首先, 我们需要一个布尔值来保存录像的状态，还需要一个UILabel来展示录像时间. 我们**Main.storyboard**， 然后拖动一个UILabel控件到屏幕的顶部，设置好它的Autolayout，并在**DJICameraViewController.m**文件中创建一个名字为"**currentRecordTimeLabel**"的IBOutlet. 接着，为录像按钮创建一个名字为 "**recordBtn**" 的IBOutlet。
  
  然后在**DJICameraViewController**的类拓展中添加布尔参数 **isRecording** . 确保在**viewDidLoad**方法中隐藏掉**currentRecordTimeLabel**. 我们可以在以下委托方法中更新 **isRecording** 的值 和 **currentRecordTimeLabel** 的文本：
   
~~~objc
-(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState
{
    if (_drone.droneType == DJIDrone_Inspire) {
        
        self.isRecording = systemState.isRecording;
        
        [self.currentRecordTimeLabel setHidden:!self.isRecording];
        [self.currentRecordTimeLabel setText:[self formattingSeconds:systemState.currentRecordingTime]];
        
        if (self.isRecording) {
            [self.recordBtn setTitle:@"Stop Record" forState:UIControlStateNormal];
        }else
        {
            [self.recordBtn setTitle:@"Start Record" forState:UIControlStateNormal];
        }
        
        //Update UISegmented Control's state
        if (systemState.workMode == CameraWorkModeCapture) {
            [self.changeWorkModeSegmentControl setSelectedSegmentIndex:0];
        }else if (systemState.workMode == CameraWorkModeRecord){
            [self.changeWorkModeSegmentControl setSelectedSegmentIndex:1];
        }
       
    }
}
   
~~~
   
   因为 **currentRecordingTime** 的值是以秒为单位计算的, 所以我们需要将它转换成“mm:ss” 的格式，如下所示:
   
~~~objc
- (NSString *)formattingSeconds:(int)seconds
{
    NSDate *date = [NSDate dateWithTimeIntervalSince1970:seconds];
    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
    [formatter setDateFormat:@"mm:ss"];
    [formatter setTimeZone:[NSTimeZone timeZoneForSecondsFromGMT:0]];
    
    NSString *formattedTimeString = [formatter stringFromDate:date];
    return formattedTimeString;
}
~~~
   
   现在, 我们可以编译运行工程，检查下刚做好的功能. 你可以尝试下 **录像** 和 **切换相机工作模式** 功能, 如果一切顺利，你会看到以下App截屏画面：
   
   ![Screenshot](../../images/record_screenshot.jpg)
   
   恭喜你! 你的FPV航拍App已经大功告成，你现在可以用它来控制Inspire 1的相机了。

###3. 现在要做什么?

   你可以从这里下载到本教程的最终Demo工程:<https://github.com/DJI-Mobile-SDK/FPVDemo-Part2.git>
   
   你已经完成了整篇教程的学习: 学会如何使用DJI Mobile SDK来开发app，展示飞行器相机的FPV视图，控制DJI 飞行器的相机进行**拍照**和**录像**操作，这两项功能经常被使用到，也是一款航拍app的基本功能点。但是，要开发一款很酷的航拍app，你还有很长的一段路要走。像预览SD卡中的照片和视频，展示飞机的OSD数据等等。请继续关注我们后续的教程，希望你喜欢！    