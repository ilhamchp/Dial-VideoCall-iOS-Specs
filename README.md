# DialVideoCall Specs
Dial VideoCall library using OpenVidu on iOS

# How to Install
### Step 1. Add the Podspec to your Podfile
Add this code below in your Podfile. Make sure you use the latest release version!
```ruby
platform :ios, '12.1'       # Add this, note: Minimum target required is iOS 12.1
source 'https://github.com/CocoaPods/Specs.git'   # Add this, note: Default CocoaPods Specs
source 'https://github.com/ilhamchp/Dial-VideoCall-iOS-Specs.git'   # Add this, note: DialVideoCall Specs

target 'DemoApps' do
  use_frameworks!

  pod 'DialVideoCall', '~>0.1.0'    # Add this, note: Always use latest version!

end
```

### Step 2. Ask for permission
To ensure all feature works, define for **`camera`** and **`microphone`** usage before starting video call.
In your `Info.plist` add key and define your own value like on the table below.
| Key | Value |
| --- | ----- |
| `NSCameraUsageDescription` | This app need to use camera for video call feature |
| `NSMicrophoneUsageDescription` | This app need to use microphone for video call feature |

### Step 3. Add Actionable Notification
To allow users accept / decline incoming video call from notification, you need to add actionable notification.
On your `AppDelegate`, add `import DialVideoCall`. Then, add this function below :
```swift
private func registerCustomActions() {
    let accept = UNNotificationAction(
      identifier: DialVideoCall.CALL_ACCEPT,
      title: "Accept Call",
      options: UNNotificationActionOptions.foreground
    )
  
    let reject = UNNotificationAction(
      identifier: DialVideoCall.CALL_DECLINE,
      title: "Decline Call",
      options: [UNNotificationActionOptions.foreground, UNNotificationActionOptions.destructive]
    )
  
    let category = UNNotificationCategory(
      identifier: DialVideoCall.VIDEO_CALL,
      actions: [accept, reject],
      intentIdentifiers: []
    )
    UNUserNotificationCenter.current()
        .setNotificationCategories([category])
}
```

And call it on

```swift
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    registerCustomActions()   // Call it
    Messaging.messaging().apnsToken = deviceToken
}
```


### Step 3. Handle Incoming Video Call
On your `AppDelegate` add this function below 

```Swift
private func goToVcallPage(callData vcmodel: VideoCallModel)
{
    var window:UIWindow
    if #available(iOS 13, *) {
        window = UIApplication.shared.windows.first { $0.isKeyWindow }!
    } else {
        window = UIApplication.shared.keyWindow!
    }
   
    let storyboard = UIStoryboard(name: "Main", bundle: nil)
    if #available(iOS 13.0, *) {
        let yourVC = storyboard.instantiateViewController(identifier: "ViewController") as ViewController
        yourVC.vcmodel = vcmodel
        window.rootViewController = yourVC
    } else {
        // Handle with your own code for iOS < 13
    }
}
```

Handle incoming video call on user tap actionable notification
```swift
func userNotificationCenter(
    _ center: UNUserNotificationCenter,
    didReceive response: UNNotificationResponse,
    withCompletionHandler completionHandler: @escaping () -> Void
 ) {
    let userInfo = response.notification.request.content.userInfo
    print(userInfo)
    if let sessionFrom = userInfo["session_from"] as? String, let sessionTo = userInfo["session_to"] as? String{
        goToVcallPage(callData: VideoCallModel(from: sessionFrom, to: sessionTo, status: response.actionIdentifier))
    }
    completionHandler()
}
```

Handle incoming video call on user tap notification
```swift
func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?
) -> Bool {
    guard let options = launchOptions,
      let remoteNotif = options[UIApplication.LaunchOptionsKey.remoteNotification] as? [String: Any] else {return true}
    let aps = remoteNotif["aps"] as? [String: Any]
    if let sessionFrom = aps!["sessionFrom"] as? String, let sessionTo = aps!["sessionTo"] as? String {
        goToVcallPage(callData: VideoCallModel(from: sessionFrom, to: sessionTo))
    }
    return true
}
```

### Step 4. Process Video Call Data from AppDelegate on ViewController
On your `ViewController` add some code like example class below :

```swift
class ViewController: UIViewController {
  
    var vcmodel: VideoCallModel? = nil    // Add this line
    override func viewDidLoad() {
        super.viewDidLoad()
    }
    
    // Add this function
    override func viewDidAppear(_ animated: Bool) {
        if vcmodel != nil {
            startVideoCall(vcmodel: vcmodel!)
            vcmodel = nil
        }
    }
    
    // Add this function
    private func startVideoCall(vcmodel: VideoCallModel){
        let vcUI = DialVideoCall.startVideoCall(vcall: vcmodel, isProduction: false)
        if(vcUI != nil){
            present(vcUI!, animated: true, completion: nil)
        }
    }
    
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

}
```

### Step 5. (OPTIONAL) Add Custom Notification Sound on iOS
To add custom notification sound on iOS platform, you can check this [link](https://stackoverflow.com/a/54104535).

# API
We provide two , to handle production and development:

<details>
  <summary>Production</summary>
  
  ### Function Call
  `DialVideoCall.startVideoCall(vcall: vcmodel, isProduction: true)`
  
  Call This function when you want to start video call in production server
  
  ### Production URL
  https://call.dial.id/softphone/clicktovcallov/?clientSid=AAAA&apikey=4k535D14LVc4LlPR0d12345&from=BBBB&to=CCCC&fcm_server_key=DDDD&fcm_token=EEEE#CCCC
  
  Change following parameter using your configuration:
  1. clientSid
  2. from
  3. to
  4. fcm_server_key
  5. fcm_token
  
  At the end of parameter, after the `#`, fill it with `to` session name
</details>

<details>
  <summary>Development</summary>
  
  ### Function Call
  `DialVideoCall.startVideoCall(vcall: vcmodel, isProduction: false)`
  
  Call This function when you want to start video call in development server
  
  ### Development URL
  https://devcall.dial.id/softphone/clicktovcall/?clientSid=AAAA&apikey=4k535D14LVc4Ll12345&from=BBBB&to=CCCC&fcm_server_key=DDDD&fcm_token=EEEE#CCCC
  
  Change following parameter using your configuration:
  1. clientSid
  2. from
  3. to
  4. fcm_server_key
  5. fcm_token
  
  At the end of parameter, after the `#`, fill it with `to` session name
</details>


# Troubleshoot

### Cloning takes a lot of time
`Cloning spec repo 'cocoapods' from 'https://github.com/CocoaPods/Specs.git'`

If the cloning process take a lot of time / forever. You can use this command to install the pod :

`pod install --verbose --no-repo-update`


### Failed to clone (authentication problem)
If you fail to clone caused by authentication problem, you can solve it by following this [link](https://stackoverflow.com/questions/68775869/support-for-password-authentication-was-removed-please-use-a-personal-access-to/68781050#68781050)

### Minimum target
`Specs satisfying the ‘DialVideoCall’ dependency were found, but they required a higher minimum deployment target.`


If this problem appear, make sure that your minimum `Podfile`, `Project Deployment Target` and `Pods Deployment Target` is iOS 12.1

### Bitcode Error
`'/Users/macbookpro2019/Documents/Cahya/iOS/dependencies/DialVideoCall/Example/Pods/GoogleWebRTC/Frameworks/frameworks/WebRTC.framework/WebRTC' does not contain bitcode. You must rebuild it with bitcode enabled (Xcode setting ENABLE_BITCODE), obtain an updated library from the vendor, or disable bitcode for this target. file '/Users/macbookpro2019/Documents/Cahya/iOS/dependencies/DialVideoCall/Example/Pods/GoogleWebRTC/Frameworks/frameworks/WebRTC.framework/WebRTC' for architecture arm64`

In case this problem appear, you can see this table below

| Location | Bitcode |
| -------- | ------ |
| Project {ProjectName] | Enable |
| Targets {ProjectName} | Disable |
| Targets {ProjectName}_Tests | Disable |
| Targets {ProjectName}_UI_Tests | Disable |
| Project Pods |  Enable |
| Targets DialVideoCall | Disable |
| Targets DialVideoCall-DialVideoCall | Disable |
| Targets GoogleWebRTC | Disable |


And do the following steps until all location from the table configured:
1. Open the `Location` configuration based on the table
2. Go to `Build Settings`
3. Find `Enable Bitcode` under `Build Option`
4. Set `Enable Bitcode` based on the table
