#iOS 9 Day by Day
#6. iPad Multitasking

iOS 9에서 큰 변화 중 하나는 Multitasking이다. Multitasking은 사용자가 한 화면에 하나 이상의 앱을 사용할 수 있게 한다. 두 가지 형태로 동작한다. 그 두가지는 Slide Over와 Split View이다.

##Slide Over View

![The new iOS Slide Over View](images/slideOver.png)

화면의 오른쪽 끝에서 왼쪽으로 쓸어넘기기를 하면 앱의 목록을 보여주는 Slide over view가 나타나고 이 좁은 창에 보여질 앱을 선택할 수 있다. 이 좁은 창은 실행중인 앱 위에 나타나며, 창 왼쪽의 입력은 불가능한 상태가 된다.

##Split View

![The new iOS Split View](images/split.png)

Split view를 보려면 Slide Over view에 있는 수직 구분자를 왼쪽으로 더 당겨야 한다. 두 앱 화면 사이에 있는 수직 구분자를 움직여서 앱 화면의 크기를 조정한다. Split View가 활성화 되었을 때, foreground 앱과 background 앱으로 나눠지지 않는다. 두 앱 모두 foreground 앱이다.

Split view는 현재 iPad Air 2에서만 가능하다.

##Enabling Multitasking in Your App

Xcode 7에서 생성된 새로운 프로젝트는 기본으로 Multitasking을 사용할 수 있다. 그러나 기존 응용프로그램을 가지고 있다면, 직접 활성화해야 한다. iOS 9 SDK를 사용하는 경우 이를 활성화하는 몇 단계 과정이 있다.

1. 앱의 모든 사용자 인터페이스의 모든 화면 방향을 활성화 시킨다.
2. Launch Storyboard를 사용한다.

###Opting Out
If your app already does the above things, then multitasking will be enabled when it is built with the iOS 9 SDK. If you want to opt out of this behaviour, specify the `UIRequiredFullscreen` key in your `info.plist ` file. 

###The Importance of Auto Layout
Auto Layout was first introduced in iOS 6, and gives you a way to lay out your UI by specifying constraints rather than fixed positions. Adaptive Layout was introduced in iOS8, which takes Auto Layout to the next level by allowing you to specify different constraints based on different size classes. Size classes identify a relative amount of display space for the height and for the width of your app's window.

Due to the nature of multitasking, there are a few issues that you'll have to take into consideration when compiling your app with the iOS 9 SDK.

###Don't Use UIInterfaceOrientation any more!
Conceptually, this doesn't work any more if your app supports multitasking. If you have a multitasking app and you are checking the current UIInterfaceOrientation, you can't be sure that your app is running in full screen. If your app is the front app in SplitView and the iPad is landscape, then even though it is larger vertically than horizontally, it will still return UIInterfaceOrientationPortrait.

Sometimes you will still need to modify your interface based on size of the app's window though. So how can we do that? The answer is to use `traitCollection.horizontalSizeClass`. This gives you the Size Class information about your interface, which you can use to conditionally position views in your app.

###Size Change Transition Events

Previously, events such as `willRotateToInterfaceOrientation` and `didRotateToInterfaceOrientation` were the recommended way to make changes to your application when the screen rotated. In iOS 8, Apple introduced `willTransitionToTraitCollection` and `viewWillTransitionToSize`. These methods become even more important in iOS 9 with the introduction of multitasking. To check whether your interface is portrait or landscape, which you still may wish to do, you can manually compare the width to the height.

###Responding to Keyboard Events

In the past, the only time your app would be effected by the keyboard was when it was opened by your app itself. Now, it's possible to have a keyboard appear on top of your app, even though a user did not open it from your app.

![The keyboard covering two apps in iOS 9](images/keyboard.png)

In some cases, you may be fine with the keyboard appearing on top of your app. However, if it obstructs an important piece of your UI then your users may be obstructed. In this situation, you should respond to one of the `UIKeyboard` notifications that have been around for a long time now. `WillShow`, `DidShow`, `WillHide`, `DidHide`, `WillChangeFrame` and `DidChangeFrame` notifications should give you the ability to do this. These events will fire in **both** apps that are present on screen.

###Other Considerations

The changes you will have to make aren't just visual. Previously, apps could rely on being the only app running in the foreground. You had sole access to the vast majority of system resources such as the CPU, GPU and memory. However, this has now changed. If a user has split view or slide over view active, and, at the same time, is watching a video in the new iOS 9 [picture in picture mode](https://developer.apple.com/library/prerelease/ios/documentation/WindowsViews/Conceptual/AdoptingMultitaskingOniPad/QuickStartForPictureInPicture.html), then these resources must be shared between three applications.

> *For best user experience, the system tightly manages resource usage and terminates apps that are using more than their fair share* - Apple iOS 9 Multitasking Documentation

You should therefore profile and heavily test your applications on different variations of iPad so that you are confident that your application is as efficient as it can be and is not using resources that it does not need.

##Further Reading
iOS 9의 새로운 multitasking 기능에 대한 더 많은 정보를 원한다면, iOS 개발자 라이브러리의 가이드에서 [Adopting Multitasking On iPad](https://developer.apple.com/library/prerelease/ios/documentation/WindowsViews/Conceptual/AdoptingMultitaskingOniPad/index.html) 살펴 보라. WWDC 205 세션 [Continuous Integration and Code Coverage in Xcode](https://developer.apple.com/videos/wwdc/2015/?id=410)을 보는 것을 추천한다. 이 글에서 우리가 만들고 설명했던 프로젝트를 시도하려고 한다면, [GitHub](https://github.com/shinobicontrols/iOS9-day-by-day/tree/master/05-CodeCoverage) 에서  찾을 수 있다는 것을 잊지 마시라.
