# Welcome to the Windows Bridge for iOS project
## Getting started
To use the Windows Bridge for iOS you'll need:

- Windows 10
- Visual Studio 2015 with Windows developer tools
- And the SDK itself: winobjc-0.1-preview.zip [LINK TBD]

If you don't have Visual Studio, you can download Visual Studio Community 2015 for free [here](https://go.microsoft.com/fwlink/p/?LinkId=534599).

## Build the SDK
If you'd like to build the SDK from source, you can use the following process:

1. Install Visual Studio
2. Clone the repo

    `git clone --recursive https://github.com/microsoft/winobjc`
3. Navigate into the **build** directory of the repo and double-click on **build.sln** to open the project solution in Visual Studio.
4. Set the build type to be **Release**.
5. Right-click on the **package** project under the **Solution** folder and select build.
6. The SDK, **winobjc.zip**, will be created in the *build/SDKPackage/Release* directory.

## Install the SDK
Installing the SDK is simple, just extract the winobjc.zip file to an appropriate location on your PC. The only restriction is that this path must not contain any spaces.

## Using the SDK
There are two ways to use the SDK. The first is to use the **vsimporter** tool to import your existing Xcode project into a Visual Studio solution and the second is to instead add Objective-C functionality to a new or existing Visual Studio project.

### Using vsimporter
The **vsimporter** tool enables you to import your Xcode project into a new Visual Studio Universal Windows Platform (UWP) app project with Objective-C support.

To use the tool:

1. Download the SDK, **winobjc-0.1-preview.zip**, and extract the files to a directory (for example `c:\winobjc`)
2. From a command prompt, navigate to the directory containing your Xcode project, for example `c:\winobjc\samples\WOCCatalog`
3. At the command prompt, run **vsimporter.exe**

    `c:\winobjc\samples\WOCCatalog> ..\..\bin\vsimporter.exe`
4.	A Visual Studio solution file is created in your current directory, double click this file to open your project in Visual Studio.
5.	Press **Ctrl-F5** to build your app and run it on your PC.

You can also pass the **-i** option at the command line to run the **vsimporter** tool in interactive mode. Interactive mode lets you see and select the specific configurations of the Xcode project that you wish to import. By default **vsimporter** creates a Visual Studio solution that targets Windows 10. If you'd like to target Windows 8.1 (Phone or Store), use the **-format** option.

For help running **vsimporter**, use the **-help** option at the command line to see the full set of supported options.

#### Known issues with vsimporter
1. `–f-no-arc-objc` is not supported (`–f-no` is unsupported for any option, as this is normally interpreted by a clang driver). To work around the issue:
  * Right click on the relevant file in the Visual Studio solution explorer
  * Select **Properties**
  * Select  **clang**

      ![FileProperties1](https://github.com/Microsoft/WinObjC/wiki/images/FileProperties1.png)
  * Set **Enable Objective-C ARC** to **No**

      ![FileProperties2](https://github.com/Microsoft/WinObjC/wiki/images/FileProperties2.png)

2. Certain build stages are ignored (although they will be logged in the console window output):
  - Shell scripts
  - Header copy stage
  - Copy file stage
3.	Absolute paths in projects may be problematic, relative paths are preferable
4.	Windows Linker flags differ from OS X
5.	Framework search paths are ignored
6.	Custom build rules are ignored
7.	Data models and asset catalogs are not currently supported

### Add Objective-C support to a new (or existing) Visual Studio project
Rather than importing an existing Xcode project, you can choose to instead add Objective-C functionality to a new or existing VS project using the following process:

1. Download the SDK, **winobjc-0.1-preview.zip**, and extract the files to a directory.
2. Open a new or existing UWP app project in Visual Studio.
3. Within the Visual Studio solution, right-click on the project.
2. Select **Build Dependencies > Build Customizations…**.

	![ProjectBuildDependencies](https://github.com/Microsoft/WinObjC/wiki/images/ProjectBuildDependencies.png)

3. Select **Find Existing**.
4. Navigate to the directory where you extracted the SDK.
5. Navigate into the **msvc** directory.
6. Select the file **starboard.targets**.
7. Ensure **starboard.targets** has a check box next to it in the "Build Customizations..." dialog.

That's it, now you can create .m and .mm files within a UWP app project.

## Visual Studio project properties
With the correct *.targets* file loaded, the **Properties** panel for your project (in Visual Studio) presents you with several new categories of options. Some key options are listed below.

### Item type
Right click on a file in the Visual Studio solutions explorer, and click on **Properties**. Under the **General** section, you should see a field called **Item Type**.

![FileProperties](https://github.com/Microsoft/WinObjC/wiki/images/FileProperties.png)

This field tells Visual Studio about the type of each file in a project. By default, Visual Studio should automatically set .m and .mm files to be of type *Clang source* (this ensures that they are built with the correct compiler – clang rather than cl). Similarly, c++/cx files (.cpp extension) should continue to be set as *C/C++ compiler* type.

Another important item type is *Starboard Resource*. Items set to this type are copied into the app package, which is useful for application assets.

These types should be set automatically, but you can use the menu to override the default behavior or correct any issues.

### Clang
In the Visual Studio solution explorer, right-click on your project and click **Properties**. You should a set of properties under *Clang*.

![ProjectProperties](https://github.com/Microsoft/WinObjC/wiki/images/ProjectProperties.png)

This process can be used to see/edit properties related to building .m and .mm files. For example, you can see whether or not ARC is enabled or specify the #include paths that are relevant to building these files.

## Using Windows APIs from Objective-C
There are a few different ways to leverage Windows APIs from your Objective-C app. Specifically, you can either add .cpp files to your project and, from there, leverage c++/cx or you can alternatively use Windows APIs directly from Objective-C.

To support direct usage of Windows APIs, the `include/Platform/Windows 8.1/UWP` directory contains a set of header files, each of which maps to a Windows namespace. For example, *WSystem.h* maps to the `Windows::System` namespace and this header file contains an Objective-C projection of the APIs in that namespace. Similarly, classes are prefixed with the letters that constitute their containing namespace, so `Windows::System::Launcher` is presented as an Objective-C `@interface` named *WSLauncher*.

The underlying implementations for these Objective-C projections use the Windows Runtime Library and are located in the libraries in `lib/Windows 8.1/x86`.

To see a detailed example of directly calling Windows APIs in Objective-C, take a look at [TBD SAMPLE].

### Async APIs
Asynchronous operations are supported through Objective-C via callbacks to blocks. For example, the Objective-C signature for *launchUriAsync* is:

`+ (void)launchUriAsync:(WFUri *)uri success:(void (^)(BOOL))success failure:(void (^)(NSError*))failure;`

The important parts are the `success:` and `failure:` parameters, these blocks are invoked with the given parameters when the async operation succeeds or fails respectively. All async APIs are exposed to Objective-C using this same pattern.

### Constructors
Looking at the async example again, you can see that it requires a *WFUri*. This is defined in *WFoundation.h* and you can create one using the following method in *WFUri*:

`+ (WFUri *)createUri:(NSString *)uri;`

In c++ the *Uri* class has a normal constructor of `Uri(String^ uri)`. However, in Objective-C constructors are mapped to static *create* methods. When constructors take multiple parameters, the parameter names are hard-coded into the Objective-C function signature:

`+ (WFUri *)createWithRelativeUri:(NSString *)baseUri relativeUri:(NSString *)relativeUri;`

### Type interoperability
Looking again at the *createUri* constructor, notice that it takes an *NSString* as a parameter. This usage is possible, because the interop layer allows you treat *HSTRINGs* as *NSStrings* and vice versa.

Similarly, looking at the `Windows::Networking::Connectivity::ProxyConfiguration` class in c++, you can see the ProxyUris property:

```
property IVectorView<Uri^>^ ProxyUris {
    IVectorView<Uri^>^ get();}
```
This property is used to access a list (an *IVectorView*) of *ProxyUris*. When projected into Objective-C, the property appears as follows:

`@property (readonly) NSArray * /*WFUri*/  proxyUris;`

The *IVectorView* is mapped into an *NSArray* and, therefore, this API may be used from Objective-C by using the built in *NSArray* type.

Interoperability for other types (enumerators, maps, etc.) is still under development.

### Event registration
Similarly to async functionality, event handlers can be provided using Objective-C blocks. For an example of this, see **winobjc/samples/WOCCatalog**:

```
[xamlWebView addLoadCompletedEvent: ^void(RTObject * sender, WUXNNavigationEventArgs * e)
{
    _welcomeLabel.text = e.uri.rawUri;
}];
```
## XAML interoperability
Even though your app might be using UIKit, WinObjC uses the XAML compositor to manage your views and perform animations on them. This functionality is implemented by tying CALayers to XAML elements. For most standard use cases, such as getting your iOS/UIKit/OpenGL app running, this implementation detail doesn't really matter. You can continue to use UIKit and CoreAnimation as you’re used to.

However, when combined with Windows API interoperability, this approach makes certain use cases, such as mixing-and-matching UIKit and XAML components, simple and straight forward. You can, for example, create a XAML webview and place it in your UIKit based application:

```
// Create the Windows::Xaml::Control::WebView and set the destination URL
WXCWebView *xamlWebView = [WXCWebView create];
[xamlWebView navigate: [WFUri createUri: @"http://www.bing.ca"]];

// Create a CALayer and add the Xaml control to that layer
CALayer *nativeLayer = [CALayer new];
nativeLayer.masksToBounds = TRUE;
[nativeLayer setBackgroundColor: [UIColor greenColor]];
[nativeLayer setFrame: CGRectMake(80, 200, 450, 250)];

//  Set contents of the CALayer to be a native Xaml Element
[nativeLayer setContentsElement: xamlWebView];

xamlWebView.Width = 850;
xamlWebView.Height = 550;

// Add the layer to the containing window
[[_mainWindow layer] addSublayer: nativeLayer];
```
See **winobjc/samples/WOCCatalog** for more extensive use of this functionality.

##Modify an iOS app for Windows
While the Windows Bridge for iOS provides support for Objective-C within a UWP app, there are some differences between the iOS and Windows platform that might necessitate code modifications or additions.

###Screen size and resolution
Windows supports a wide variety of form factors and screen sizes. For a good user experience, your app should be aware of and respond to the configuration on which it's run. That means, at startup your app needs to specify:
- How large (width/height) it should be
- What magnification it should render at
- How window resize should be handled

To specify these behaviors, simply create a *Category* for your UIApplication in your AppDelegate called *UIApplicationInitialStartupMode*

`@interface UIApplication(UIApplicationInitialStartupMode)`

You can then add a method *setStartupDisplayMode* that is called at startup and define the appropriate settings using a *WOCDisplayMode* object.

`+(void) setStartupDisplayMode: (WOCDisplayMode*) mode { ... }`

There are a number of properties exposed by the *WOCDisplayMode* object, but for simplicity the *DisplayPreset* property can be used to set behavior to several predefined configurations.
- **WOCDisplayPresetPhone320x480**: Configures a phone app with 320x480 resolution
- **WOCDisplayPresetTablet768x1024**: Configures a tablet app at 768x1024
- **WOCDisplayPresetNative**: Configures app to use the size and aspect ratio of the containing window
- **WOCDisplayPresetNative2x**: Configures app to use the size and aspect ratio of the containing window, as well as apply a 2x magnification

If you'd like to instead manually configure properties of the **WOCDisplayMode** object, the following are available.

- **autoMagnification**: This property can be used, along with a *fixedWidth* and *fixedHeight*, to handle window size changes by simply "zooming" in/out. The pixel width/height of your app will remain constant (as specified below) but the magnification will change with window size.

- **sizeUIWindowToFit**: This property resizes the application windows along with the containing window. If this is set to **FALSE** then the application will remain at the specified (original) size irrespective of its containing window.

- **fixedWidth**: This property specifies the width, in pixels, of the application. If set to "0" then the width of the application will follow the width of the containing window.

- **fixedHeight**: This property specifies the height, in pixels, of the application. If set to "0" then the height of the application will follow the height of the containing window.

- **fixedAspectRatio**: If this property is set to 0 then the app has no fixed aspect ratio. However, if it has a non-zero value, then the provided aspect ratio will be maintained throughout any resizing operations.

- **magnification**: If this property is set to 0 then the application may zoom in/out as the window is resized. However, if set to a non-zero value then the magnification is fixed and this may be used to increase app size on high resolution displays.

For a detailed walkthrough of handling screen resolution, take a look at DisplayModeViewController.m and AppDelegate.m in the **WOCCatalog** sample. To see the complete interface, you can view *include/UIKit/UIApplication.h*.


##Contributions
There are many ways that you can contribute to the WinObjC project:
-	Submit a bug
-	Verify fixes for bugs
-	Submit a code fix for a bug
-	Submit a feature request
-	Submit a unit test
-	Tell others about the WinObjC project
-	Tell the developers how much you appreciate the project

### Pull requests
You will need to sign a [Contribution License Agreement](https://cla.microsoft.com/) (CLA) before submitting your pull request. To complete the CLA, you will need to submit the request via the form and then electronically sign the CLA when you receive an email containing a link to the document.

This process needs to only be done once for any Microsoft open source project.

### Contributing to README and Wiki
You do not need to sign a Contribution License Agreement if you are just contributing to the README or the Wiki. However, by submitting a contribution to the README or the Wiki, you are contributing it under the [Creative Commons CC0 1.0 Universal Public Domain Dedication](http://creativecommons.org/publicdomain/zero/1.0/).

###Problems or questions?
You can reach our dev team in a variety of ways:
- Tweet @WindowsDev and mark your questions with **#winobjc**
- Post questions to Stack Overflow and tag them with **winobjc**
-	Visit the #winobjc channel on IRC (webchat.freenode.net)


## What's still under development?
As this project is still under active development, there are a few features that are not yet built out:
1.	Autolayout
2.	Storyboard support
3.	MapKit
4.	AssetsLibrary
5.	AddressBook
6.	Ads
7.	Objective-C annotations
8.	Media Capture and Playback
9.	ARM support (x86 only today)
10.	Compiler optimizations will not work and will likely crash clang, debug builds only
