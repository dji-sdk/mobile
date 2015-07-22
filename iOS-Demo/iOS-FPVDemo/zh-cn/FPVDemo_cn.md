# DJI Mobile iOS SDK 教程


## 如何创建一个航拍相机App: 第一部分

### 1. 下载SDK


你可以从[这里](https://dev.dji.com/cn/products/sdk/mobile-sdk/downloads)下载到最新的SDK 

开发包内容包括：

- 相机、云台、地面站、操纵杆等各模块的demo程序（地面站和操纵杆仅Level 2 开发者可使用）
- API 说明文档
- lib库文件

支持平台：支持iOS 6.1 及以上版本

### 2. 解压SDK并导入到自己的工程

   1. 先拷贝DJISDK.framework到您的工程目录下，再将DJISDK.framework拖到您工程”Frameworks”的目录下，如下图所示:

   ![ImportSDK](images/importSDK.png)
   
   2. 从Xcode的左侧导航栏选择当前工程，进入右侧的 Build Phases -> Link Binary With Libraries. 点击底部的 "+" 按钮添加 libstdc++.6.0.9.dylib 和 libz.dylib 到你的工程中. 编译SDK时需要用到这些动态库。

   3. 由于iOS SDK中使用了C++进行开发，故为了在工程中使用SDK，需要修改工程其中一个实现文件的后缀名为"**.mm**". 这里我们修改 "**AppDelegate.m**" 文件, 修改为 "**AppDelegate.mm**".

**重要提示**: APP如需支持Inspire 1/Phantom 3 Professional飞行器, 需要添加MFI通信支持。

   添加方法：在工程的Supporting Files文件夹下的plist文件添加MFI协议名称，如下图所示：  
   
   ![MFI](images/MFIProtocol.png)
   
### 3. 激活 SDK

3.1 在 **AppDelegate.mm** 文件中导入SDK头文件:

```objc
   #import <DJISDK/DJISDK.h>
```

3.2 实现 **DJIAppManagerDelegate** 的委托方法，并且在注册App成功后，发送通知，如下图所示：

```objc
@interface AppDelegate ()<DJIAppManagerDelegate>

@end

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    NSString *appKey = @"Enter Your App Key";
    [DJIAppManager registerApp:appKey withDelegate:self];
    
    return YES;
}

#pragma mark DJIAppManagerDelegate Method
-(void)appManagerDidRegisterWithError:(int)error
{
    NSString* message = @"Register App Successed!";
    if (error != RegisterSuccess) {
        message = @"Register App Failed!";
    }else
    {
        [[NSNotificationCenter defaultCenter] postNotificationName:@"RegisterAppSuccess" object:nil];
    }
    
    UIAlertView* alertView = [[UIAlertView alloc] initWithTitle:@"Register App" message:message delegate:nil cancelButtonTitle:@"OK" otherButtonTitles:nil];	
    [alertView show];
}
```

---
**注意**: 你可以在[SDK网站上](https://dev.dji.com/cn/user/mobile-sdk)创建属于你自己App的**App Key**, 如下图所示:

![AppKey](images/AppKey_CN.png)

另外, **App Key** 是和工程的 **Bundle Identifier**相关联的. 所以如果Bundle Identifier不正确的话，你就不能在多个不同工程里面使用同一个App Key。

---

如果注册App失败, 你可以通过检查以下委托方法的 `error` 变量值来寻找原因:

```objc
-(void)appManagerDidRegisterWithError:(int)error;
```

APP KEY 激活失败码如下所示:
 
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


3.3 现在运行你的Xcode工程, 如果一切顺利, 你可以看到 "Register App Successed!" 的提示！


### 4. 实现FPV视图功能
4.1 我们使用 FFMPEG 解码库 (http://ffmpeg.org) 对视频流进行解码. 你可以在下载好的SDK开发包中找到 `VideoPreviewer` 文件夹. 将它拷贝到 Xcode 工程的文件夹中, 然后像下图所示添加到工程导航栏的`thirdParty`文件夹下:
  
 ![AppKey](images/ffmpegImport.png)
 
4.2 接下来来到XCode -> Project -> Build Phases -> Link Binary With Libraries中, 点击`+` 添加`libiconv.dylib`动态库文件. 然后在Build Settings中的Header Search Paths 添加FFMPEG的 include 文件夹路径. 同时在Library Search Paths中添加 FFMPEG的 lib 文件夹的路径，如下图所示:
  
  ![HeaderSearchPath](images/headerSearchPath.png)
  ![LibrarySearchPath](images/librarySearchPath.png)
  
4.3 创建 `DJICameraViewController` 并在 `Main.storyboard` 中设置它为`Root ViewController`, 添加一个 UIView到该viewController中, 设置它的 IBOutlet 为`fpvPreviewView`, 然后在底部添加两个 UIButton 和一个 UISegmentedControl 控件, 设置好它们的IBOutlets 和 IBActions，如下图所示:
  
  ![Storyboard](images/Storyboard.png)
  
  打开 `DJICameraViewController.m` 文件 导入 `DJISDK` 和 `VideoPreviewer` 头文件, 然后创建 `DJIDrone` 和 `DJICamera` 实例变量，如下所示实现它们的委托方法:
  
```objc
#import <DJISDK/DJISDK.h>
#import "VideoPreviewer.h"

@interface DJICameraViewController ()<DJICameraDelegate, DJIDroneDelegate>
{
    DJIDrone *_drone;
    DJICamera* _camera;
}

```
4.4 在`ViewDidLoad` 函数中初始化 DJIDrone 实例变量 然后设置它的类型为 `DJIDrone_Inspire` (你可以根据你的飞机机型输入), 设置它的delegate 并启动 Video Previewer. 同时, 添加一个 NSNotificationCenter 的监听来检查 `RegisterAppSuccess` 消息:
  
```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(registerAppSuccess:) name:@"RegisterAppSuccess" object:nil];
    _drone = [[DJIDrone alloc] initWithType:DJIDrone_Inspire];
    _drone.delegate = self;
    _camera = _drone.camera;
    _camera.delegate = self;
    
    [[VideoPreviewer instance] start];
    
}

- (void)registerAppSuccess:(NSNotification *)notification
{
    
    NSLog(@"registerAppSuccess");
    [_drone connectToDrone];
    [_camera startCameraSystemStateUpdates];
    
}

```
  
接着, 在`viewWillAppear`方法中设置 `fpvPreviewView` 实例变量为 `VideoPreviewer` 的视图，以展示视频流画面，然后在`viewWillDisappear` 方法中重置为`nil`:
 
```objc
- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    
    [_drone connectToDrone];
    [_camera startCameraSystemStateUpdates];
    [[VideoPreviewer instance] setView:self.fpvPreviewView];
    
}

- (void)viewWillDisappear:(BOOL)animated
{
    
    [super viewWillDisappear:animated];
    [_camera stopCameraSystemStateUpdates];
    [_drone disconnectToDrone];
    [[VideoPreviewer instance] setView:nil];
    
}
```
  
  最后, 实现 DJICameraDelegate 方法:
  
```objc
#pragma mark - DJICameraDelegate

-(void) camera:(DJICamera*)camera didReceivedVideoData:(uint8_t*)videoBuffer length:(int)length
{
    uint8_t* pBuffer = (uint8_t*)malloc(length);
    memcpy(pBuffer, videoBuffer, length);
    [[VideoPreviewer instance].dataQueue push:pBuffer length:length];
}

-(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState
{
    if (!systemState.isTimeSynced) { //Only for Phantom 2 Vision/Phantom 2 Vision+ to check camera time
        [_camera syncTime:nil];
    }
    if (systemState.isUSBMode) { //Only for Phantom 2 Vision/Phantom 2 Vision+ to keep cameraMode when systemState is under USBMode
        [_camera setCamerMode:CameraCameraMode withResultBlock:Nil];
    }
    
}

-(void) droneOnConnectionStatusChanged:(DJIConnectionStatus)status
{
    if (status == ConnectionSuccessed) {
        NSLog(@"Connection Successed");
    }
    else if(status == ConnectionStartConnect)
    {
        NSLog(@"Start Reconnect");
    }
    else if(status == ConnectionBroken)
    {
        NSLog(@"Connection Broken");
    }
    else if (status == ConnectionFailed)
    {
        NSLog(@"Connection Failed");
    }
}

```
   `-(void) camera:(DJICamera*)camera didReceivedVideoData:(uint8_t*)videoBuffer length:(int)length` 委托方法用来发送视频流信息给**VideoPreviewer**进行解码.
   
   `-(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState` 委托方法用来获取相机的状态信息, 它会被频繁调用, 所以你可以在这个委托方法中更新你的App界面状态和相机参数设置.
   
   `-(void) droneOnConnectionStatusChanged:(DJIConnectionStatus)status` 委托方法用来检查飞行器的连接状态.
   
4.5 编译运行你的工程, 检查下一切是否正常. 如果你可以看到类似以下截屏画面, 那么你就可以准备启动你的航拍飞机，享受飞机摄像机上的FPV画面了!
  
  ![Screenshot](images/Screenshot.jpg)
  
### 5. 连接飞行器
完成以上步骤后, 现在就可以连接你的移动设备到DJI飞行器上，检查是否获取到FPV画面，以下是连接指引：

* 连接 Inspire 1, Phantom 3 Professional or Phantom 3 Advanced:

  **1**. 首先启动遥控器电源, 再启动你的飞行器
  
  **2**. 使用苹果的数据线将移动设备连接到遥控器上
  
  **3**. 如果弹出“是否信任该设备？”对话框，选择信任
  
  **4**. 使用app来控制飞机的相机


* 连接 DJI Phantom 2 Vision+ or Phantom 2 Vision:

  **1**. 首先启动遥控器电源, 再启动你的飞行器
  
  **2**. 启动Wi-Fi中继器电源
  
  **3**. 在你的移动设备上启动 WIFI ，然后连接到名字类似Phantom-xxxxxx (xxxxxx是你的中继器SSID数字)的WIFI网络上
  
  **4**. 使用app来控制飞机的相机
  
  
### 6. 享受FPV视图

如果你可以在app中看到飞机的视频流，那么恭喜，你已经完成了第一部分教程的内容了！下图是app的截屏：

  ![fpv](images/fpv.jpg)


### 7. 现在要怎么做?

   你可以从这里下载到本教程的Demo工程:<https://github.com/DJI-Mobile-SDK/FPVDemo-Part1.git>
   
   你已经学会了如何配置DJI Mobile SDK的iOS开发环境，并成功用它开发app来展示飞行器相机的FPV画面。在接下来的教程中，我们会在此基础上添加拍照和录像功能. 请关注我们第二部分的教程，希望你喜欢！
   
---
   
## 如何创建一个航拍相机App: 第二部分

#####在第一部分中, 我们实现了一个基本的FPV视图，你可以在这个视图上看到飞行器摄像头的视频画面. 在本教程中, 我们使用Inspire 1进行演示, 你会学到如何在app中添加拍照和录像功能, 我们开始吧!

### 1. 实现拍照功能

添加以下代码到 `captureAction` 方法中:

```objc
- (IBAction)captureAction:(id)sender {
    
    [_camera startTakePhoto:CameraSingleCapture withResult:^(DJIError *error) {
        if (error.errorCode != ERR_Successed) {
            NSLog(@"Take Photo Error : %@", error.errorDescription);
        }
    }];
}
```
   调用`DJICamera`的以下方法:
   
   `-(void) startTakePhoto:(CameraCaptureMode)captureMode withResult:(DJIExecuteResultBlock)block;`
   
  `CameraCaptureMode` 有以下四种类型: 
  
```objc
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
```
  
  它们提供了多种方式进行拍照, `CameraSingleCapture` 使用起来很简单, 你不需要在调用`startTakePhoto`方法之前，设置其它参数. 对于 `CameraMultiCapture`, 你需要使用 `DJICamera.h`中的`-(void) setMultiCaptureCount:(CameraMultiCaptureCount)count withResultBlock:(DJIExecuteResultBlock)block`方法来设置多拍参数，并且在block中检查是否设置成功，然后才能调用`startTakePhoto`方法。
  
  更多信息, 请查看 `DJICamera.h` 和 `DJICameraSettingsDef.h` 文件。
    
---

**注意**: 因为 DJICamera 有多个子类: DJIInspireCamera, DJIPhantom3AdvancedCamera, DJIPhantomCamera等等, 你需要找到对应的方法来设置拍照参数. 例如, Inspire 1支持CameraAEBCapture 模式, 所以你应该在`DJIInspireCamera.h`文件中寻找AEB的设置方法，而不是`DJICamera.h`.

---

  这里我们设置拍照模式为 `CameraSingleCapture`. 你可以在block中的`DJIError` 变量检查拍照结果。
  
  编译运行你的工程，尝试下拍照功能，如果屏幕在你点击`拍照`按钮后，闪烁了一下，表示拍照成功！到此，你已经完成了拍照功能。  
  
### 2. 实现录像功能
  
##### 1. 切换相机模式Camera Mode
   在实现录像功能之前，我们需要先切换相机模式。   
   我们先检查下`DJICameraSettingsDef.h`文件：
   
```objc
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
```
   如你所见, 有5种`CameraWorkMode`, 这里我们只使用 `CameraWorkModeCapture` 和 `CameraWorkModeRecord` 两种相机模式. 因为这里使用的是Inspire 1机型, 所以我们要用`DJIInspireCamera.h`文件中的
   
   `-(void) setCameraWorkMode:(CameraWorkMode)mode withResult:(DJIExecuteResultBlock)block` 方法来切换相机模式。
   
   打开 `Main.storyboard`，为UISegmented Control控件添加一个 IBOutlet, 命名为 `changeWorkModeSegmentControl`. 还记得第一部分教程中DJICameraDelegate 的一个委托方法吗？
      
   `-(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState;`
   
   我们可以用这个委托方法，在切换`CameraWorkModeCapture` 和 `CameraWorkModeRecord` 相机模式时，更新segmented control的状态。
      
```objc
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

```

 然后我们按以下方式实现 `changeWorkModeAction` 方法:
 
```objc
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

```
 这里添加了两个UIAlertView， 在设置相机模式失败时提醒用户。
  
##### 2. 添加录像按钮

  首先, 我们需要一个布尔值来保存录像的状态，还需要一个UILabel来展示录像时间. 我们`Main.storyboard`， 然后拖动一个UILabel控件到屏幕的顶部，设置好它的Autolayout，并在`DJICameraViewController.m`文件中创建一个名字为`currentRecordTimeLabel`的IBOutlet. 接着，为录像按钮创建一个名字为 `recordBtn` 的IBOutlet。
  
  然后在`DJICameraViewController`的类拓展中添加布尔参数 `isRecording`， 确保在`viewDidLoad`方法中隐藏掉`currentRecordTimeLabel`。我们可以在以下委托方法中更新 `isRecording` 的值 和 `currentRecordTimeLabel` 的文本：
   
```objc
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
   
```
   
   因为 `currentRecordingTime` 的值是以秒为单位计算的, 所以我们需要将它转换成`mm:ss`的格式，如下所示:
   
```objc
- (NSString *)formattingSeconds:(int)seconds
{
    NSDate *date = [NSDate dateWithTimeIntervalSince1970:seconds];
    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
    [formatter setDateFormat:@"mm:ss"];
    [formatter setTimeZone:[NSTimeZone timeZoneForSecondsFromGMT:0]];
    
    NSString *formattedTimeString = [formatter stringFromDate:date];
    return formattedTimeString;
}
```
   
   现在, 我们可以编译运行工程，检查下刚做好的功能. 你可以尝试下 **录像** 和 **切换相机工作模式** 功能, 如果一切顺利，你会看到以下App截屏画面：
   
   ![Screenshot](images/record_screenshot.jpg)
   
   恭喜你! 你的FPV航拍App已经大功告成，你现在可以用它来控制Inspire 1的相机了。

###3. 现在要做什么?

   你可以从这里下载到本教程的最终Demo工程:<https://github.com/DJI-Mobile-SDK/FPVDemo-Part2.git>
   
   你已经完成了整篇教程的学习: 学会如何使用DJI Mobile SDK来开发app，展示飞行器相机的FPV视图，控制DJI 飞行器的相机进行**拍照**和**录像**操作，这两项功能经常被使用到，也是一款航拍app的基本功能点。但是，要开发一款很酷的航拍app，你还有很长的一段路要走。像预览SD卡中的照片和视频，展示飞机的OSD数据等等。
   
   请继续关注我们后续的教程，希望你喜欢！    
