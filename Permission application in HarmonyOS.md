<h2 id="zA9a6"><font style="color:rgb(0, 0, 0) !important;">TokenID</font></h2>
<font style="color:rgba(0, 0, 0, 0.85) !important;">The system uses TokenID (Token identity) as the unique identifier of the application. The permission management service manages the AT (Access Token) information of the application through the TokenID of the application, including the application identity APP ID, sub-user ID, application clone index information, application APL, application permission authorization status, etc. When using resources, the system will obtain the permission authorization status information of the corresponding application through the TokenID as the unique identity identifier mapping, and perform authentication accordingly to control the resource access behavior of the application.</font>

  


<font style="color:rgba(0, 0, 0, 0.85) !important;">It should be noted that the system supports the multi-user feature and the application clone feature. The same application will have its own AT under different sub-users and different application clones, and the TokenIDs of these ATs are also different.</font>

<h2 id="p4M92"><font style="color:rgb(0, 0, 0) !important;">Authorization Methods</font></h2>
<font style="color:rgba(0, 0, 0, 0.85) !important;">According to different authorization methods, permission types can be divided into system_grant (system authorization) and user_grant (user authorization).</font>

  


1. <font style="color:rgba(0, 0, 0, 0.85) !important;">If the system_grant permission is applied for in the application, the system will automatically grant the corresponding permission to the application when the user installs the application.</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">For user_grant permissions, not only do you need to apply for permissions in the installation package, but you also need to request user authorization by sending a pop-up window during the dynamic operation of the application. Only after the user manually allows the authorization can the application truly obtain the corresponding permission and successfully access the target object of the operation.</font>

<h3 id="UoJYc"><font style="color:rgb(0, 0, 0) !important;">system_grant Permissions</font></h3>
**<font style="color:rgba(0, 0, 0, 0.85);">Configure</font>**

```plain
ohos.permission.INTERNET
Allows the use of the Internet network.

ohos.permission.USE_BLUETOOTH
Allows the application to view the Bluetooth configuration.

ohos.permission.GET_BUNDLE_INFO
Allows querying the basic information of the application.

ohos.permission.PREPARE_APP_TERMINATE
Allows the application to perform custom pre-termination actions before closing.

ohos.permission.PRINT 
Allows the application to obtain the capabilities of the printing framework.

ohos.permission.DISCOVER_BLUETOOTH
Allows the application to configure the local Bluetooth, find and pair with remote devices.

ohos.permission.ACCELEROMETER
Allows the application to read the data of the accelerometer sensor.

ohos.permission.ACCESS_BIOMETRIC
Allows the application to use biometric identification capabilities for identity authentication.

ohos.permission.ACCESS_NOTIFICATION_POLICY
Allows the application to access the notification policy on this device.

ohos.permission.GET_NETWORK_INFO
Allows the application to obtain data network information.

ohos.permission.GET_WIFI_INFO
Allows the application to obtain Wi-Fi information.

ohos.permission.GYROSCOPE
Allows the application to read the data of the gyroscope sensor.

ohos.permission.KEEP_BACKGROUND_RUNNING
Allows the Service Ability to continue running in the background.

ohos.permission.NFC_CARD_EMULATION
Allows the application to implement the card emulation function.

ohos.permission.NFC_TAG
Allows the application to read and write Tag cards.

ohos.permission.PRIVACY_WINDOW
Allows the application to set the window as a privacy window and prohibits screenshot and screen recording.

ohos.permission.PUBLISH_AGENT_REMINDER
Allows the application to use the background agent reminder.

ohos.permission.SET_NETWORK_INFO
Allows the application to configure the data network.

ohos.permission.SET_WIFI_INFO
Allows the application to configure Wi-Fi devices.

ohos.permission.VIBRATE
Allows the application to control the motor vibration.

ohos.permission.CLEAN_BACKGROUND_PROCESSES
Allows the application to clean up related background processes according to the package name.

ohos.permission.COMMONEVENT_STICKY
Allows the application to publish sticky public events.

ohos.permission.MODIFY_AUDIO_SETTINGS
Allows the application to modify the audio settings.

ohos.permission.RUNNING_LOCK
Allows the application to obtain a running lock to ensure the continuous running of the application in the background.

ohos.permission.SET_WALLPAPER**
Allows the application to set the wallpaper.

ohos.permission.ACCESS_CERT_MANAGER
Allows the application to perform operations such as querying certificates and private credentials.

ohos.permission.hsdr.HSDR_ACCESS
Allows the application to access the security detection and response framework.
```

<h3 id="OUKEh"><font style="color:rgb(0, 0, 0) !important;">user_grant Permissions</font></h3>
**<font style="color:rgba(0, 0, 0, 0.85);">Must</font>**

```plain
ohos.permission.CAMERA
Allows the application to use the camera.

ohos.permission.READ_MEDIA
Allows the application to read the media file information in the user's external storage.

ohos.permission.WRITE_MEDIA
Allows the application to read and write the media file information in the user's external storage.

ohos.permission.APPROXIMATELY_LOCATION
Allows the application to obtain the approximate location information of the device.
ohos.permission.LOCATION
Allows the application to obtain the location information of the device.

ohos.permission.MICROPHONE
Allows the application to use the microphone.

ohos.permission.READ_CALENDAR
Allows the application to read the calendar information.

ohos.permission.READ_HEALTH_DATA
Allows the application to read the user's health data.

ohos.permission.WRITE_CALENDAR
Allows the application to add, remove or change calendar events.

ohos.permission.ACCESS_BLUETOOTH
Allows the application to access the Bluetooth and use Bluetooth capabilities, such as pairing and connecting to peripheral devices.

ohos.permission.MEDIA_LOCATION
Allows the application to access the geographical location information in the user's media files.

ohos.permission.APP_TRACKING_CONSENT
Allows the application to read the open anonymous device identifier.

ohos.permission.ACTIVITY_MOTION
Allows the application to read the user's motion status.

ohos.permission.DISTRIBUTED_DATASYNC
Allows data exchange between different devices.
```

<h2 id="uYi4B"><font style="color:rgb(0, 0, 0) !important;">Declaration of Permissions</font></h2>
<font style="color:rgba(0, 0, 0, 0.85) !important;">When an application applies for permissions, it needs to declare the required permissions one by one in the configuration file of the project. Otherwise, the application will not be able to obtain authorization.</font>

  


<font style="color:rgba(0, 0, 0, 0.85) !important;">The application needs to declare the permissions in the</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">requestPermissions</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">tag of the</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">module.json5</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">configuration file.</font>

  


| **<font style="color:rgb(0, 0, 0) !important;">Attribute</font>** | **<font style="color:rgb(0, 0, 0) !important;">Description</font>** | **<font style="color:rgb(0, 0, 0) !important;">Value Range</font>** |
| :--- | :--- | :--- |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">name</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">Required, fill in the name of the permission to be used.</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">Must be a system-defined permission</font> |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">reason</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">Optional. When the applied permission is a user_grant permission, this field is required to describe the reason for applying for the permission.</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>**<font style="color:rgb(0, 0, 0) !important;">Note:</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">This field is used for application listing verification. When the applied permission is a user_grant permission, it is required and needs to be adapted to multiple languages.</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">Use string resource reference. The format is $string: ***</font> |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">usedScene</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">Optional. When the applied permission is a user_grant permission, this field is required. Describe that the usage scenario of the permission is composed of abilities and when. Among them, abilities can be configured as multiple UIAbility components, and when represents the call timing.</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>**<font style="color:rgb(0, 0, 0) !important;">Note:</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">It is optional by default. When the applied permission is a user_grant permission, the abilities tag is required, and the when tag is optional.</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">abilities: The name of the UIAbility or ExtensionAbility component. when: inuse (when in use), always (always).</font> |


  


**<font style="color:rgba(0, 0, 0, 0.85);">module.json5</font>**

```plain
{
  "module" : {
    // ...
    "requestPermissions":[
      {
        "name" : "ohos.permission.PERMISSION1",
        "reason": "$string:reason",
        "usedScene": {
          "abilities": [
            "FormAbility"
          ],
          "when":"inuse"
        }
      },
      {
        "name" : "ohos.permission.PERMISSION2",
        "reason": "$string:reason",
        "usedScene": {
          "abilities": [
            "FormAbility"
          ],
          "when":"always"
        }
      }
    ]
  }
}
```

<h3 id="xlPV7"><font style="color:rgb(0, 0, 0) !important;">Specification for the Copywriting Content of the Reason for Using Permissions</font></h3>
1. <font style="color:rgba(0, 0, 0, 0.85) !important;">Keep the sentences concise and do not add redundant delimiters.</font>**<font style="color:rgb(0, 0, 0) !important;">Suggested sentence pattern</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">: Used for something.</font>**<font style="color:rgb(0, 0, 0) !important;">Example</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">: Used for scanning codes and taking photos.</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">The string of the purpose description is recommended to be less than 72 characters (that is, 36 Chinese characters, and it is displayed as about two lines in the UI interface). It cannot exceed 256 characters to ensure the experience of multi-language adaptation.</font>
3. <font style="color:rgba(0, 0, 0, 0.85) !important;">If not written, the default application reason will be displayed.</font>

<h2 id="XmV9i"><font style="color:rgb(0, 0, 0) !important;">Requesting Authorization from the User</font></h2>
<font style="color:rgba(0, 0, 0, 0.85) !important;">When an application needs to access user privacy information or use system capabilities, such as obtaining location information, accessing the calendar, taking photos or recording videos using the camera, etc., it should request authorization from the user. These permissions are user_grant permissions.</font>

<h3 id="DNSNh"><font style="color:rgb(0, 0, 0) !important;">Steps to Apply for user_grant Permissions</font></h3>
1. <font style="color:rgba(0, 0, 0, 0.85) !important;">In the configuration file, declare the permissions that the application needs to request.</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">Associate the target object for which the application needs to apply for permissions with the corresponding target permission, so that the user can clearly know which operations require the user to grant the specified permission to the application.</font>
3. <font style="color:rgba(0, 0, 0, 0.85) !important;">When running the application, when the user triggers the access to the target object, the interface should be called to accurately trigger the dynamic authorization pop-up window. The internal interface will check whether the current user has authorized the permissions required by the application. If the current user has not granted the permissions required by the application, the interface will pop up a dynamic authorization pop-up window to request authorization from the user.</font>
4. <font style="color:rgba(0, 0, 0, 0.85) !important;">Check the user's authorization result, and confirm that the user has authorized before proceeding to the next operation.</font>

<h3 id="iOgES"><font style="color:rgb(0, 0, 0) !important;">Constraints and Limitations</font></h3>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Every time an operation that requires a target permission is executed, the application must check whether it already has the permission.</font><font style="color:rgba(0, 0, 0, 0.85) !important;">To check whether the user has granted a specific permission to your application, you can use the</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>[<font style="color:rgb(9, 105, 218);">checkAccessToken()</font>](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-abilityaccessctrl-0000001820880529#ZH-CN_TOPIC_0000001820880529__checkaccesstoken9)<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">function. This method will return</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>[<font style="color:rgb(9, 105, 218);">PERMISSION_GRANTED</font>](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-abilityaccessctrl-0000001820880529#ZH-CN_TOPIC_0000001820880529__grantstatus)<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">or</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>[<font style="color:rgb(9, 105, 218);">PERMISSION_DENIED</font>](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-abilityaccessctrl-0000001820880529#ZH-CN_TOPIC_0000001820880529__grantstatus)<font style="color:rgba(0, 0, 0, 0.85) !important;">.</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Before accessing an interface protected by a target permission each time, you need to use the</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>[<font style="color:rgb(9, 105, 218);">requestPermissionsFromUser()</font>](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-abilityaccessctrl-0000001820880529#ZH-CN_TOPIC_0000001820880529__requestpermissionsfromuser9)<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">interface to request the corresponding permission.</font><font style="color:rgba(0, 0, 0, 0.85) !important;">The user may cancel the application's permission through the system settings after dynamically granting the permission, so the previously granted authorization status cannot be persisted.</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">The authorization of user_grant permissions should be based on the principle of being known and controllable by the user. The application needs to actively call the system's interface for dynamically applying for permissions during runtime. The system pop-up window is authorized by the user. The user can identify the rationality of the application's application for the corresponding sensitive permission based on the context of the application's running scenario, and thus make the correct choice.</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">The system does not encourage frequently popping up windows to disturb the user. If the user refuses the authorization, the pop-up window cannot be triggered again. The application needs to guide the user to manually grant the permission in the "Settings" interface of the system application.</font>

<h4 id="G06Qg"><font style="color:rgb(0, 0, 0) !important;">1. Check Whether a Single Permission is Authorized</font></h4>
**<font style="color:rgba(0, 0, 0, 0.85);">plaintext</font>**

```plain
/**
 * Verify whether the current permission has been authorized
 * @param permission
 * @returns
 */
async checkAccessToken(permission: Permissions) {
  let atManager = abilityAccessCtrl.createAtManager();
  let grantStatus: abilityAccessCtrl.GrantStatus = abilityAccessCtrl.GrantStatus.PERMISSION_DENIED;

  // Get the accessTokenID of the application
  let tokenID: number = 0;
  try {
    let bundleInfo: bundleManager.BundleInfo = await bundleManager.getBundleInfoForSelf(bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION);
    let appInfo: bundleManager.ApplicationInfo = bundleInfo.appInfo;
    tokenID = appInfo.accessTokenId;
  } catch (err) {
    console.error(`getBundleInfoForSelf failed, error: ${err}`);
  }

  // Verify whether the application is granted the permission
  try {
    grantStatus = await atManager.checkAccessToken(tokenID, permission);
  } catch (err) {
    console.error(`checkAccessToken failed, error: ${err}`);
  }
  return grantStatus;
}
```

<h4 id="Qf9W2"><font style="color:rgb(0, 0, 0) !important;">2. Batch Check of Permissions</font></h4>
**<font style="color:rgba(0, 0, 0, 0.85);">plaintext</font>**

```plain
/**
 * Request permissions
 * @param permissions
 * @param context
 */
requestPermissions(permissions: Array<Permissions>, context: common.UIAbilityContext): void {
  let atManager: abilityAccessCtrl.AtManager = abilityAccessCtrl.createAtManager();
  // requestPermissionsFromUser will determine the authorization status of the permission to decide whether to pop up a window
  atManager.requestPermissionsFromUser(context, permissions).then((data) => {
    let grantStatus: Array<number> = data.authResults;
    let length: number = grantStatus.length;
    for (let i = 0; i < length; i++) {
      if (grantStatus[i] === 0) {
        // User authorization, can continue to access the target operation

      } else {
        // User refuses authorization, prompt the user that authorization is required to access the function of the current page, and guide the user to open the corresponding permission in the system settings
        return;
      }
    }
    // Authorization successful
  }).catch((err: BusinessError) => {
    console.error(`Failed to request permissions from user. Code is ${err.code}, message is ${err.message}`);
  })
}
```

<h4 id="sf7YF"><font style="color:rgb(0, 0, 0) !important;">3. Batch Application for Permissions</font></h4>
**<font style="color:rgba(0, 0, 0, 0.85);">plaintext</font>**

```plain
/**
 * Pass in the permissions to be checked here, and you can also pass in an array of permissions Array<Permissions>
 * @param permission
 * @returns
 */
async checkAndRequestPermissions(permissions: Array<Permissions>, context: common.UIAbilityContext): Promise<void> {
  for (let i = 0; i < permissions.length; i++) {
    let grantStatus: abilityAccessCtrl.GrantStatus = await this.checkAccessToken(permissions[i]);
    if (grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) {
      // Already authorized, can continue to access the target operation

    } else {
      // Apply for permission
      this.requestPermissions(this.permissions, context)
    }
  }

}
```

<h4 id="G62CY"><font style="color:rgb(0, 0, 0) !important;">4. How to Use the Application for Permissions</font></h4>
**<font style="color:rgba(0, 0, 0, 0.85);">plaintext</font>**

```plain
aboutToAppear(): void {
  const context: common.UIAbilityContext = getContext(this) as common.UIAbilityContext
  this.checkAndRequestPermissions(this.permissions, context)
}
```

