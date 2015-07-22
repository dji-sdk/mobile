# DJI Mobile iOS SDK Tutorial


## How to create a Camera Application: Part 2/2

In [Part 1](../Part1/FPVDemo_Part1_en.md), we have implemented the basic FPV view which allowed us to see the live video stream from the aircraft's camera. In this tutorial, we use the Inspire 1 as an example to show you how to add photo taking and video recording features in the app, let's get started!

### 1. Implement the Capture function

Add the following codes to the **captureAction** IBAction method:

~~~objc
- (IBAction)captureAction:(id)sender {
    
    [_camera startTakePhoto:CameraSingleCapture withResult:^(DJIError *error) {
        if (error.errorCode != ERR_Successed) {
            NSLog(@"Take Photo Error : %@", error.errorDescription);
        }
    }];
}
~~~
   Just call the following method of **DJICamera**:
   
   -(void) startTakePhoto:(CameraCaptureMode)captureMode withResult:(DJIExecuteResultBlock)block;
   
  There are 4 types of **CameraCaptureMode**: 
  
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
  
  These enum values give you mutiple ways to capture photos, **CameraSingleCapture** is easy to use because you do not need to set any param before calling **startTakePhoto** method, unlike the other modes. For **CameraMultiCapture**, you need to use the
  **-(void) setMultiCaptureCount:(CameraMultiCaptureCount)count withResultBlock:(DJIExecuteResultBlock)block;** method in **DJICamera.h** header file to set the **captureCount** param and check if the take photo action succeed in the block before calling **startTakePhoto** method.
  
  For more infos, you can check **DJICamera.h** and **DJICameraSettingsDef.h** files.
    
---
##### Note: Since DJICamera has several subclasses: DJIInspireCamera, DJIPhantom3AdvancedCamera, DJIPhantomCamera, etc, you should find the corresponding methods when you want to set the params. For example, CameraAEBCapture mode is supported in Inspire 1, so you should find the AEB setting method in DJIInspireCamera.h file rather than in DJICamera.h file.

---

  Here we set the capture mode to **CameraSingleCapture**. You can check the capture result from the **DJIError** instance in the block.
  
  Build and run your project and then try the capture function. If the screen flash after your press the **capture** button, your capture fuction should be working.
  
  
### 2. Implement the Record function
  
#### 1. Switching Camera Mode
   Before we implementing the record function, we need to switch the camera work mode firstly.
   
   Let's check the **DJICameraSettingsDef.h** file.
   
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
   You can see above that there are 5 types of **CameraWorkMode**. Since we are using the Inspire 1 as an example, **CameraWorkModeCapture** and **CameraWorkModeRecord** are used as follows:
   
   -(void) setCameraWorkMode:(CameraWorkMode)mode withResult:(DJIExecuteResultBlock)block; method inside **DJIInspireCamera.h** file to switch camera work mode.
   
   Open **Main.storyboard** and add an IBOutlet for the UISegmented Control called "changeWorkModeSegmentControl". Remember the delegate method of DJICameraDelegate in **Tutorial Part 1**?
   
   -(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState;
   
   We can update the state of the segmented control when switching between **CameraWorkModeCapture** and **CameraWorkModeRecord** using the above delegate method.
   
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
 Now we can implement the **changeWorkModeAction** method as follows:
 
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
 Here we add two UIAlertViews to get a warning when the user set CameraWorkMode failed.
 
#### 2. Add Record Action

  Firstly, we need a BOOL variable to save the status of the record action and a UILabel to show the current record time. So let's go to **Main.storyboard** and drag a UILabel on top of the screen, set up the Autolayout for it and create an IBOutlet named "**currentRecordTimeLabel**" to the **DJICameraViewController.m** file. Moreover, create an IBOutlet called "**recordBtn**" for the Record Button.
  
  Then add a BOOL variable **isRecording** in the class extension of **DJICameraViewController**. Be sure to hide **currentRecordTimeLabel** in the **viewDidLoad** method. We can update the text values for **isRecording** and **currentRecordTimeLabel**'s text value in the following delegate method.
   
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
   
   Because the value of **currentRecordingTime** is counted in seconds, so we need to convert it to "mm:ss" format like this:
   
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
   
   Now, we can build and run the project and check the functions. You can try to play with the **Record** and **Switch Camera WorkMode** functions, if everything is going well, you should see the screenshot like this:
   
   ![Screenshot](../../images/record_screenshot.jpg)
   
   Congratulations! Your Aerial FPV iOS app is complete, you can now use this app to control the camera of your Inspire 1. 


### 3. Where To Go From Here?

   You can download the final project for this tutorial here: <https://github.com/DJI-Mobile-SDK/iOS-FPVDemo-Part2.git>
   
   You’ve come a long way in this tutorial: you’ve learned how to use DJI Mobile SDK to show the FPV View from the aircraft's camera and control the camera of DJI's Aircraft. These are the most basic and common features in a typical drone mobile app: **Capture** and **Record**. However, if you want to create a drone app that is more fancy, you still have a long way to go. More advanced features would include previewing the photo and video in the SD Card, showing the OSD data of the aircraft and so on. Hope you enjoy this tutorial, and stay tuned for our next one!