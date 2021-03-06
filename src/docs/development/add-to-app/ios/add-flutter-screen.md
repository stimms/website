---
title: Adding a Flutter screen to an iOS app
short-title: Add a Flutter screen
description: Learn how to add a single Flutter screen to your existing iOS app.
---

This guide describes how to add a single Flutter screen to an existing iOS app.

## Start a FlutterEngine and FlutterViewController

To launch a Flutter screen from an existing iOS, you start a
[`FlutterEngine`]({{site.api}}/objcdoc/Classes/FlutterEngine.html)
and a [`FlutterViewController`]({{site.api}}/objcdoc/Classes/FlutterViewController.html).

{{site.alert.secondary}}
  The `FlutterEngine` serves as a host to the Dart VM and your Flutter runtime,
  and the `FlutterViewController` attaches to a `FlutterEngine` to pass UIKit
  input events into Flutter and to display frames rendered by the `FlutterEngine`.
{{site.alert.end}}

The `FlutterEngine` may have the same lifespan as your `FlutterViewController`
or outlive your `FlutterViewController`.

{{site.alert.tip}}
It's generally recommended to pre-warm a long-lived `FlutterEngine` for your
application because:

- The first frame will appear faster when showing the `FlutterViewController`.
- Your Flutter and Dart state will outlive one `FlutterViewController`.
- Your application and your plugins can interact with Flutter and your Dart
  logic before showing the UI.
{{site.alert.end}}

See [Loading sequence and performance](/docs/development/add-to-app/performance)
for more analysis on the latency and memory trade-offs of pre-warming an engine.

### Create a FlutterEngine

The proper place to create a `FlutterEngine` is specific to your host app. As an example, we
demonstrate creating a `FlutterEngine`, exposed as a property, on app startup in
the app delegate.

{% samplecode engine %}
{% sample Objective-C %}
**In `AppDelegate.h`:**

<?code-excerpt "AppDelegate.h" title?>
```objectivec
@import UIKit;
@import Flutter;

@interface AppDelegate : FlutterAppDelegate // More on the FlutterAppDelegate below.
@property (nonatomic,strong) FlutterEngine *flutterEngine;
@end
```

**In `AppDelegate.m`:**

<?code-excerpt "AppDelegate.m" title?>
```objectivec
#import <FlutterPluginRegistrant/GeneratedPluginRegistrant.h> // Used to connect plugins.

#import "AppDelegate.h"

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary<UIApplicationLaunchOptionsKey, id> *)launchOptions {
  self.flutterEngine = [[FlutterEngine alloc] initWithName:@"my flutter engine"];
  // Runs the default Dart entrypoint with a default Flutter route.
  [self.flutterEngine run];
  [GeneratedPluginRegistrant registerWithRegistry:self.flutterEngine];
  return [super application:application didFinishLaunchingWithOptions:launchOptions];
}

@end
```
{% sample Swift %}
**In `AppDelegate.swift`:**

<?code-excerpt "AppDelegate.swift" title?>
```swift
import UIKit
import Flutter
import FlutterPluginRegistrant // Used to connect plugins.

@UIApplicationMain
class AppDelegate: FlutterAppDelegate { // More on the FlutterAppDelegate.
  lazy var flutterEngine = FlutterEngine(name: "my flutter engine")

  override func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Runs the default Dart entrypoint with a default Flutter route.
    flutterEngine.run();
    GeneratedPluginRegistrant.register(with: self.flutterEngine);
    return super.application(application, didFinishLaunchingWithOptions: launchOptions);
  }
}
```
{% endsamplecode %}

### Show a FlutterViewController with your FlutterEngine

The following example shows a generic ViewController with a UIButton hooked to
present a [`FlutterViewController`]({{site.api}}/objcdoc/Classes/FlutterViewController.html).
The `FlutterViewController` uses the `FlutterEngine` instance created in the
`AppDelegate`.

{% samplecode vc %}
{% sample Objective-C %}
<?code-excerpt "ViewController.m" title?>
```objectivec
@import Flutter;
#import "AppDelegate.h"
#import "ViewController.h"

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];

    // Make a button to call the showFlutter function when pressed.
    UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
    [button addTarget:self
               action:@selector(showFlutter)
     forControlEvents:UIControlEventTouchUpInside];
    [button setTitle:@"Show Flutter!" forState:UIControlStateNormal];
    button.backgroundColor = UIColor.blueColor;
    button.frame = CGRectMake(80.0, 210.0, 160.0, 40.0);
    [self.view addSubview:button];
}

- (void)showFlutter {
    FlutterEngine *flutterEngine =
        ((AppDelegate *)UIApplication.sharedApplication.delegate).flutterEngine;
    FlutterViewController *flutterViewController =
        [[FlutterViewController alloc] initWithEngine:flutterEngine nibName:nil bundle:nil];
    [self presentViewController:flutterViewController animated:YES completion:nil];
}
@end
```
{% sample Swift %}
<?code-excerpt "ViewController.swift" title?>
```swift
import UIKit
import Flutter

class ViewController: UIViewController {
  override func viewDidLoad() {
    super.viewDidLoad()

    // Make a button to call the showFlutter function when pressed.
    let button = UIButton(type:UIButton.ButtonType.custom)
    button.addTarget(self, action: #selector(showFlutter), for: .touchUpInside)
    button.setTitle("Show Flutter!", for: UIControl.State.normal)
    button.frame = CGRect(x: 80.0, y: 210.0, width: 160.0, height: 40.0)
    button.backgroundColor = UIColor.blue
    self.view.addSubview(button)
  }

  @objc func showFlutter() {
    let flutterEngine = (UIApplication.shared.delegate as! AppDelegate).flutterEngine
    let flutterViewController =
        FlutterViewController(engine: flutterEngine, nibName: nil, bundle: nil)
    present(flutterViewController, animated: true, completion: nil)
  }
}
```
{% endsamplecode %}

Now, you have a Flutter screen embedded in your iOS app.

{{site.alert.note}}
Using the previous example, the default `main()` entrypoint function of your
default Dart library would run when calling `run` on the
`FlutterEngine` created in the `AppDelegate`.
{{site.alert.end}}

### _Alternatively_ - Create a FlutterViewController with an implicit FlutterEngine

As an alternative to the previous example, you can let the
`FlutterViewController` implicitly create its own `FlutterEngine` without
pre-warming one ahead of time.

This is not recommended because creating a `FlutterEngine` on-demand could
introduce a noticeable latency between when the `FlutterViewController` is
presented and when it renders its first frame. This could, however, be
useful if the Flutter screen is rarely shown, when there are no good
heuristics to determine when the Dart VM should be started, and when Flutter
doesn't need to persist state between view controllers.

To let the `FlutterViewController` present without an existing `FlutterEngine`,
omit the `FlutterEngine` construction, and create the
`FlutterViewController` without an engine reference.

{% samplecode no-engine-vc %}
{% sample Objective-C %}
<?code-excerpt "ViewController.m" title?>
```objectivec
// Existing code omitted.
- (void)showFlutter {
  FlutterViewController *flutterViewController =
      [[FlutterViewController alloc] initWithProject:nil nibName:nil bundle:nil];
  [self presentViewController:flutterViewController animated:YES completion:nil];
}
@end
```
{% sample Swift %}
<?code-excerpt "ViewController.swift" title?>
```swift
// Existing code omitted.
func showFlutter() {
  let flutterViewController = FlutterViewController(project: nil, nibName: nil, bundle: nil)
  present(flutterViewController, animated: true, completion: nil)
}
```
{% endsamplecode %}

See [Loading sequence and performance](/docs/development/add-to-app/performance)
for more explorations on latency and memory usage.

## Using the FlutterAppDelegate

Letting your application's `UIApplicationDelegate` subclass `FlutterAppDelegate`
is recommended but not required.

The `FlutterAppDelegate` performs functions such as:

- Forwarding application callbacks such as [`openURL`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623112-application)
  to plugins such as [local_auth](https://pub.dev/packages/local_auth).
- Forwarding status bar taps (which can only be detected in the AppDelegate) to
  Flutter for scroll-to-top behavior.

If your app delegate can't directly make `FlutterAppDelegate` a subclass,
make your app delegate implement the `FlutterAppLifeCycleProvider` protocol in
order to make sure your plugins receive the necessary callbacks. Otherwise,
plugins that depend on these events may have undefined behavior.

For instance:

<?code-excerpt "AppDelegate.h" title?>
```objectivec
@import Flutter;
@import UIKit;
@import FlutterPluginRegistrant;

@interface AppDelegate : UIResponder <UIApplicationDelegate, FlutterAppLifeCycleProvider>
@property (strong, nonatomic) UIWindow *window;
@property (nonatomic,strong) FlutterEngine *flutterEngine;
@end
```

The implementation should delegate mostly to a `FlutterPluginAppLifeCycleDelegate`:

<?code-excerpt "AppDelegate.m" title?>
```objectivec
@interface AppDelegate ()
@property (nonatomic, strong) FlutterPluginAppLifeCycleDelegate* lifeCycleDelegate;
@end

@implementation AppDelegate

- (instancetype)init {
    if (self = [super init]) {
        _lifeCycleDelegate = [[FlutterPluginAppLifeCycleDelegate alloc] init];
    }
    return self;
}

- (BOOL)application:(UIApplication*)application
didFinishLaunchingWithOptions:(NSDictionary<UIApplicationLaunchOptionsKey, id>*))launchOptions {
    self.flutterEngine = [[FlutterEngine alloc] initWithName:@"io.flutter" project:nil];
    [self.flutterEngine runWithEntrypoint:nil];
    [GeneratedPluginRegistrant registerWithRegistry:self.flutterEngine];
    return [_lifeCycleDelegate application:application didFinishLaunchingWithOptions:launchOptions];
}

// Returns the key window's rootViewController, if it's a FlutterViewController.
// Otherwise, returns nil.
- (FlutterViewController*)rootFlutterViewController {
    UIViewController* viewController = [UIApplication sharedApplication].keyWindow.rootViewController;
    if ([viewController isKindOfClass:[FlutterViewController class]]) {
        return (FlutterViewController*)viewController;
    }
    return nil;
}

- (void)touchesBegan:(NSSet*)touches withEvent:(UIEvent*)event {
    [super touchesBegan:touches withEvent:event];

    // Pass status bar taps to key window Flutter rootViewController.
    if (self.rootFlutterViewController != nil) {
        [self.rootFlutterViewController handleStatusBarTouches:event];
    }
}

- (void)application:(UIApplication*)application
didRegisterUserNotificationSettings:(UIUserNotificationSettings*)notificationSettings {
    [_lifeCycleDelegate application:application
didRegisterUserNotificationSettings:notificationSettings];
}

- (void)application:(UIApplication*)application
didRegisterForRemoteNotificationsWithDeviceToken:(NSData*)deviceToken {
    [_lifeCycleDelegate application:application
didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}

- (void)application:(UIApplication*)application
didReceiveRemoteNotification:(NSDictionary*)userInfo
fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler {
    [_lifeCycleDelegate application:application
       didReceiveRemoteNotification:userInfo
             fetchCompletionHandler:completionHandler];
}

- (BOOL)application:(UIApplication*)application
            openURL:(NSURL*)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey, id>*)options {
    return [_lifeCycleDelegate application:application openURL:url options:options];
}

- (BOOL)application:(UIApplication*)application handleOpenURL:(NSURL*)url {
    return [_lifeCycleDelegate application:application handleOpenURL:url];
}

- (BOOL)application:(UIApplication*)application
            openURL:(NSURL*)url
  sourceApplication:(NSString*)sourceApplication
         annotation:(id)annotation {
    return [_lifeCycleDelegate application:application
                                   openURL:url
                         sourceApplication:sourceApplication
                                annotation:annotation];
}

- (void)application:(UIApplication*)application
performActionForShortcutItem:(UIApplicationShortcutItem*)shortcutItem
  completionHandler:(void (^)(BOOL succeeded))completionHandler NS_AVAILABLE_IOS(9_0) {
    [_lifeCycleDelegate application:application
       performActionForShortcutItem:shortcutItem
                  completionHandler:completionHandler];
}

- (void)application:(UIApplication*)application
handleEventsForBackgroundURLSession:(nonnull NSString*)identifier
  completionHandler:(nonnull void (^)(void))completionHandler {
    [_lifeCycleDelegate application:application
handleEventsForBackgroundURLSession:identifier
                  completionHandler:completionHandler];
}

- (void)application:(UIApplication*)application
performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler {
    [_lifeCycleDelegate application:application performFetchWithCompletionHandler:completionHandler];
}

- (void)addApplicationLifeCycleDelegate:(NSObject<FlutterPlugin>*)delegate {
    [_lifeCycleDelegate addDelegate:delegate];
}
@end
```

## Launch options

The examples demonstrate running Flutter using the default launch settings.

In order to customize your Flutter runtime, you can also specify the Dart entrypoint, library, and route.

### Dart entrypoint

Calling `run` on a `FlutterEngine`, by default, runs the `main()` Dart function
of your `lib/main.dart` file.

You can also run a different entrypoint function by using [`runWithEntrypoint`]({{site.api}}/objcdoc/Classes/FlutterEngine.html#/c:objc(cs)FlutterEngine(im)runWithEntrypoint:)
with an `NSString` specifying a different Dart function.

{{site.alert.note}}
Dart entrypoint functions other than `main()` must be annotated with the following in order to not be [tree-shaken](https://en.wikipedia.org/wiki/Tree_shaking) away when compiling:

<?code-excerpt "main.dart" title?>
```dart
@pragma('vm:entry-point')
void myOtherEntrypoint() { ... };
```
{{site.alert.end}}

### Dart library

In addition to specifying a Dart function, you can specify an entrypoint
function in a specific file.

For instance the following runs `myOtherEntrypoint()` in `lib/other_file.dart` instead of `main()` in
`lib/main.dart`:

{% samplecode entrypoint-library %}
{% sample Objective-C %}
<?code-excerpt "Objective-C" title?>
```objectivec
[flutterEngine runWithEntrypoint:@"myOtherEntrypoint" libraryURI:@"other_file.dart"];
```
{% sample Swift %}
<?code-excerpt "Swift" title?>
```swift
flutterEngine.run(withEntrypoint: "myOtherEntrypoint", libraryURI: "other_file.dart")
```
{% endsamplecode %}


### Route

An initial route can be set for your Flutter [`WidgetsApp`]({{site.api}}/flutter/widgets/WidgetsApp-class.html)
when constructing the engine.

{% samplecode initial-route %}
{% sample Objective-C %}
<?code-excerpt "Creating engine" title?>
```objectivec
FlutterEngine *flutterEngine =
    [[FlutterEngine alloc] initWithName:@"my flutter engine"];
[[flutterEngine navigationChannel] invokeMethod:@"setInitialRoute"
                                      arguments:@"/onboarding"];
[flutterEngine run];
```
{% sample Swift %}
<?code-excerpt "Creating engine" title?>
```swift
let flutterEngine = FlutterEngine(name: "my flutter engine")
flutterEngine.navigationChannel.invokeMethod("setInitialRoute", arguments:"/onboarding")
flutterEngine.run()
```
{% endsamplecode %}

This code sets your `dart:ui`'s [`window.defaultRouteName`]({{site.api}}/flutter/dart-ui/Window/defaultRouteName.html)
to `"/onboarding"` instead of `"/"`.

{{site.alert.warning}}
`"setInitialRoute"` on the `navigationChannel` must be called before running your
`FlutterEngine` in order for Flutter's first frame to use the desired
route.

Specifically, this must be called before running the Dart entrypoint.  The
entrypoint may lead to a series of events where
[`runApp`]({{site.api}}/flutter/widgets/runApp.html) builds a
Material/Cupertino/WidgetsApp which implicitly creates a
[Navigator]({{site.api}}/flutter/widgets/Navigator-class.html) which
may read `window.defaultRouteName` when the
[`NavigatorState`]({{site.api}}/flutter/widgets/NavigatorState-class.html) is
first initialized.

Setting the initial route after running the engine doesn't have an effect.
{{site.alert.end}}

{{site.alert.tip}}
In order to imperatively change your current Flutter route from the platform
side after the `FlutterEngine` is already running, use [pushRoute]({{site.api}}/objcdoc/Classes/FlutterViewController.html#/c:objc(cs)FlutterViewController(im)pushRoute:)
or [popRoute]({{site.api}}/objcdoc/Classes/FlutterViewController.html#/c:objc(cs)FlutterViewController(im)popRoute)
on the `FlutterViewController`.

To pop the iOS route from the Flutter side, call [`SystemNavigator.pop()`]({{site.api}}/flutter/services/SystemNavigator/pop.html).
{{site.alert.end}}

See [Navigation and routing](/docs/development/ui/navigation) for more about Flutter's routes.

### Other

The preious example only illustrates a few ways to customize how a Flutter
instance is initiated. Using [platform channels](/docs/development/platform-integration/platform-channels),
you're free to push data or prepare your Flutter environment in any way you'd
like, before presenting the Flutter UI via a `FlutterViewController`.
