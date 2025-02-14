# Full device screen sharing

Full device screen sharing allows your support agents to view screens from applications outside of your own. This is often useful where support agents need to check the state of system settings, or need to see the user navigate between multiple applications.

If you do not want this feature, you may completely disable "Full Device upgrade" in your [account settings](../account-configuration.md).&#x20;

Full device screen sharing is supported by all SDKs. Check below to see if any extra steps are required for your platform:

{% tabs %}
{% tab title="Web" %}
{% hint style="info" %}
Redaction, annotation, and remote control are disabled when in full device mode via our Web SDK. If you require full desktop redaction, annotation, and remote control, please see our [MacOS](../../sdk-installation/macos.md) and [Windows](../../sdk-installation/windows.md) SDKs, which will require a downloaded utility on the end-user's computer.
{% endhint %}

The Cobrowse.io SDK for Web includes an optional "full device" toggle in the bottom right when running an active session. This feature enables the end-user to share their entire screen on their laptop/desktop, or select individual apps or browser tabs to share (browser dependent).

This is great if the end-user navigates to browser tabs outside of your website, where the Web SDK's JavaScript snippet is not installed, or if the end-user needs to share a PDF, or their entire desktop.&#x20;

No extra integration work is required to use full device mode via our Web SDK.

{% hint style="warning" %}
Due to browser limitations, this feature is not available on IE11, or in the mobile browsers such as Mobile Chrome and Mobile Safari. Please see our native [Android](../../sdk-installation/android.md) and [iOS](../../sdk-installation/ios.md) SDKs for [full device capabilities](./) on mobile.&#x20;
{% endhint %}
{% endtab %}

{% tab title="iOS" %}
The Cobrowse.io SDK for iOS allows full device screen capture, but this requires adding a Broadcast Extension when integrating the iOS SDK. Please follow the directions below.&#x20;

**Add a Broadcast Extension target**

1. Open your Xcode project
2. Navigate to File > New > Target...
3. Pick "Broadcast Upload Extension"
4. Enter a name for the target
5. Uncheck "Include UI Extension"
6. Create the target, noting its bundle ID
7. Change the target SDK of your Broadcast Extension to iOS 12.0 or higher _(Note: it should still work as far back to iOS 11.0)_

**Set up Keychain Sharing**

Your app and the app extension you created above need to share some secrets via the iOS Keychain. They do this using their own Keychain group so they are isolated from the rest of your apps Keychain.

In **both** your **app target** and your **extension target** add a Keychain Sharing entitlement for the "io.cobrowse" keychain group.

**Add the bundle ID to your plist**

Take the bundle ID of the **extension** you created above, and add the following entry in your apps `Info.plist` (_Note:_ **not** in the extensions `Info.plist`), replacing the bundle ID below with your own:

```markup
<key>CBIOBroadcastExtension</key>
<string>your.app.extension.bundle.ID.here</string>
```

**Add `CobrowseIOExtension` dependency to your project**

The app extension needs a dependency on the CobrowseIO app extension framework. It is available for installation via several dependency managers.

{% tabs %}
{% tab title="SPM" %}
**Add the new package dependency to your project**

```
https://github.com/cobrowseio/cobrowse-sdk-ios-binary.git
```

Make sure your **app target** uses `CobrowseIO` package and your **extension target** uses `CobrowseIOExtension` package, respectively:

<img src="../../.gitbook/assets/xcode_spm_dependency_structure.png" height="400" />

{% hint style="info" %}
Xcode 13.3 and newer might not copy `CobrowseIOExtension.framework` extension dependency into resulting IPA builds. If that happens to you, follow the steps below:

1. Add a new script build phase to your **app target**:

    <img src="../../.gitbook/assets/xcode_add_new_run_script.png" height="320" />

2. Configure the new script:

    a) Set the phase name you like (e.g. _Copy Cobrowse.io broadcast extension framework_)

    b) Set _Shell_ to `/usr/bin/ruby`

    c) Copy and paste the script content:
    ```ruby
    require 'fileutils'

    cbioExt = "CobrowseIOAppExtension.framework"
    cbioExtFrameworkSource = "#{ENV['TARGET_BUILD_DIR']}/#{cbioExt}"
    cbioExtFramework = "#{ENV['BUILT_PRODUCTS_DIR']}/#{ENV['FRAMEWORKS_FOLDER_PATH']}/#{cbioExt}"

    if (!Dir.exist?(cbioExtFramework))
        FileUtils.copy_entry cbioExtFrameworkSource, cbioExtFramework
        FileUtils.rm_rf "#{cbioExtFramework}/Headers"
        FileUtils.rm_rf "#{cbioExtFramework}/Modules"
    end

    if (ENV['PLATFORM_NAME'] == 'iphoneos')
        codeSignIdentityForItems = ENV['EXPANDED_CODE_SIGN_IDENTITY_NAME']
        if (!codeSignIdentityForItems || codeSignIdentityForItems.length == 0)
            codeSignIdentityForItems = ENV['CODE_SIGN_IDENTITY']
        end

        `codesign --force --verbose --sign "#{codeSignIdentityForItems}" "#{cbioExtFramework}"`
    end
    ```

    d) Uncheck _"For install builds only"_, _"Based on dependency analysis"_, _"Show environment variables in build log"_, and _"Use discovery dependency file"_:

    <img src="../../.gitbook/assets/xcode_spm_copy_extension_script.png" height="560" />

{% endhint %}
{% endtab %}
{% tab title="CocoaPods" %}
**Add the new target to your Podfile**

Add the following to your Podfile, replacing the target name with you own extensions target name:

```ruby
# Replace YourExtensionTargetName with your extension target name
target 'YourExtensionTargetName' do
    pod 'CobrowseIO/Extension', '~>2'
end
```

{% hint style="info" %}
By default CocoaPods links both `CobrowseIO.framework` and `CobrowseIOExtension.framework` with **your app target** which leads to several warnings shown at runtime, such as _Class CBIOSession is implemented in both CobrowseIOAppExtension.framework and CobrowseIO.framework. One of the two will be used. Which one is undefined._ To get rid of these warnings, add the following script to your Podfile:

```ruby
post_install do |installer|
  # Replace YourAppTargetName with your app target name
  target = 'YourAppTargetName'
  aggregate_target = installer.aggregate_targets.find { |aggregate_target| aggregate_target.name == "Pods-#{target}" }
  aggregate_target.xcconfigs.each do |config_name, config_file|
    config_file.frameworks.delete('CobrowseIOAppExtension')
    xcconfig_path = aggregate_target.xcconfig_path(config_name)
    config_file.save_as(xcconfig_path)
  end
end
```
{% endhint %}
_Make sure to run `pod install` after updating your Podfile_
{% endtab %}
{% endtabs %}


**Implement the extension**

Xcode will have added `SampleHandler.m` and `SampleHandler.h` (or `SampleHander.swift`) files as part of the target you created earlier. Replace the content of the files with the following:

#### Swift

```swift
import CobrowseIOAppExtension

class SampleHandler: CobrowseIOReplayKitExtension {

}
```

#### Objective C

```objectivec
// SampleHandler.h

@import CobrowseIOAppExtension;

@interface SampleHandler : CobrowseIOReplayKitExtension

@end
```

```objectivec
// SampleHandler.m

#import "SampleHandler.h"

@implementation SampleHandler

@end
```

**Build and run your app**

You're now ready to build and run your app. The full device capability is only available on physical devices, it will not work in the iOS Simulator.

If you've set everything up properly, after clicking the blue circular icon you should see the following screen to select your Broadcast Extension.&#x20;

<img src="../../.gitbook/assets/broadcast_extension_example.png" title="Select your Broadcast Extension from the list" height="560" />

### Troubleshooting

If full device screen capture on iOS is not working, please check the following:

* Please verify you are testing on a physical device, and not the iOS simulator.
* Please verify you have added the Bundle Id of your Broadcast Extension to your main app's Info.plist as described in our documentation. If you have not, then no options will appear in the list after clicking the blue circular record button.&#x20;
* Please verify you are not running any other screen recording or screen mirroring software at the same time, as this will interfere.&#x20;
{% endtab %}

{% tab title="Android" %}
The Cobrowse.io SDK for Android will allow full device screen capture, including home screen, device settings, and everything else, just by toggling "full device mode" during an active session.

No extra integration work is required to use full device mode via our Android SDK.

#### Notes for unattended access

For unattended full device access, we strongly recommend:

* Please initiate sessions with push notifications, rather than our default sockets. This will enable unattended access, even when your app has been backgrounded a long time, or force closed. More info at [Initiate sessions with push](../initiate-sessions-with-push.md).
* Please turn off "Require User Consent" prompts at [https://cobrowse.io/dashboard/settings](https://cobrowse.io/dashboard/settings). Otherwise, a user must always accept the consent prompt on the device before a session can start.
* Be wary of battery optimization policies. On some devices you may need to add your app to a battery optimization whitelist to prevent it from killing the push notifications. More info here: [https://dontkillmyapp.com/](https://dontkillmyapp.com/)
* This requires enabling the Accessibility Service when integrating the Android SDK. Please see the [full device remote control docs](full-device-remote-control.md).

### Troubleshooting

* If the screen is black during full device screen capture, please make sure your views are not marked as secure. More info here: [https://developer.android.com/reference/android/view/WindowManager.LayoutParams#FLAG\_SECURE](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#FLAG\_SECURE)
* If you are using Android Enterprise, please ensure your enterprise settings do not disallow screen capture.
* If you get `compile error android:foregroundServiceType not found`, please update your Android project to use `compileSdkVersion 29`.&#x20;
{% endtab %}

{% tab title="React Native" %}
{% hint style="info" %}
Please follow the iOS and Android documentation to implement full device capabilities on React Native.
{% endhint %}



### Troubleshooting

* For React Native on iOS, some clients have reported that Xcode does not automatically create the _\{{extensionname\}}.entitlements_ file in the extension directory, which is necessary for the "io.cobrowse" keychain sharing to work.&#x20;
{% endtab %}

{% tab title="Xamarin.iOS" %}
{% hint style="info" %}
Please review the iOS documentation for full device capabilities first.&#x20;

This documentation specific to Xamarin.iOS is supplementary, and covers the differences only.&#x20;
{% endhint %}

**Add a Broadcast Extension project**

In Visual Studio for Mac:

1. Open your Xamarin solution
2. Right click on the solution, select Add > New Project...
3. Navigate iOS > Extension
4. Pick "Broadcast Upload Extension"
5. Enter a name for the target, e.g. `YourApp.iOS.BroadcastUploadExtension`
6. Select your iOS app to add the extension to
7. Create the location for the extension and press "Create"
8. Visual Studio for Mac will create two extension projects for you: `YourApp.iOS.BroadcastUploadExtension` and `YourApp.iOS.BroadcastUploadExtensionUI`. The second project is not required and you can safely delete it.
9. Change the target SDK of your Broadcast Extension target to at least iOS 10.0

**Set up Keychain Sharing**

Your app and the app extension you created above need to share some secrets via the iOS Keychain. They do this using their own Keychain group so they are isolated from the rest of your apps Keychain.

In **both** your **iOS app** and your **extension project** add a Keychain Sharing entitlement for the "io.cobrowse" keychain group.

**Add the bundle ID to your plist**

Take the bundle ID of the **extension** you created above, and add the following entry in your apps `Info.plist` (_Note:_ **not** in the extensions `Info.plist`), replacing the bundle ID below with your own:

```markup
<key>CBIOBroadcastExtension</key>
<string>your.app.extension.bundle.ID.here</string>
```

**Add the Cobrowse.io AppExtension NuGet**

The app extension needs a dependency on the CobrowseIO app extension framework. Add the following NuGet to the **extension project** (not to the iOS app project):

* [![CobrowseIO.AppExtension.iOS NuGet](https://img.shields.io/nuget/v/CobrowseIO.AppExtension.iOS.svg?label=CobrowseIO.AppExtension.iOS)](https://www.nuget.org/packages/CobrowseIO.AppExtension.iOS/)

**Implement the extension**

Visual Studio will have added `SampleHandler.cs` file as part of the extension project you created earlier. Replace the content of the file with the following:

```csharp
using Xamarin.CobrowseIO.AppExtension;

[Register("SampleHandler")]
public class SampleHandler : CobrowseIOReplayKitExtension
{
    protected SampleHandler(IntPtr handle) : base(handle)
    {
    }
}
```

**Make sure Info.plist points to the correct class**

Open Info.plist of the extension project and make sure that `NSExtension` section looks like this:

```markup
<plist version="1.0">
<dict>
    ...
    <key>NSExtension</key>
    <dict>
        <key>NSExtensionPointIdentifier</key>
        <string>com.apple.broadcast-services-upload</string>
        <key>NSExtensionPrincipalClass</key>
        <string>SampleHandler</string>
        <key>RPBroadcastProcessMode</key>
        <string>RPBroadcastProcessModeSampleBuffer</string>
    </dict>
</dict>
</plist>
```
{% endtab %}

{% tab title="Xamarin.Android" %}
{% hint style="info" %}
Please see the Android documentation for full device capabilities.&#x20;
{% endhint %}
{% endtab %}

{% tab title="macOS" %}
The Cobrowse.io SDK for macOS will allow full device screen capture, including home screen, device settings, and everything else, just by toggling "full device mode" during an active session.

No extra integration work is required to use full device mode via our macOS SDK.
{% endtab %}

{% tab title="Windows" %}
Full device screen share by default, no extra integration needed.
{% endtab %}
{% endtabs %}

**Are you looking to customize the full device consent prompt?** See these docs:

{% content-ref url="../customize-the-interface/full-device-consent-dialog.md" %}
[full-device-consent-dialog.md](../customize-the-interface/full-device-consent-dialog.md)
{% endcontent-ref %}
