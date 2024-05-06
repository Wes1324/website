---
title: Large screen devices
description: >
  Things to keep in mind when adapting apps to large screens.
short-title: Large screens
---

<?code-excerpt path-base="ui/adaptive_app_demos"?>

This page provides guidance on optimizing your
app to improve its behavior on large screens.

Flutter, like Android, defines [large screens][] as tablets,
foldables, and ChromeOS devices running Android. Flutter
_also_ defines large screen devices as web, desktop,
and iPads.

<b>PENDING: Mariam, do you have any specific iPad numbers?</b>

:::secondary Why do large screens matter, in particular?
Demand for large screens continues to increase.
As of January 2024,
more than [270 million active large screen][large screens]
and foldable devices run on Android.
When your app supports large screens,
it also receives other benefits.
Optimizing your app to fill the screen.
For example, it:

* Improves your app's user engagement metrics.
* Increases your app's visibility in the Play Store.
  Recent [Play Store updates][] show ratings by
  device type and indicates when an app lacks
  large screen support. 
* Ensures that your app meets iPadOS submission
  guidelines and is [accepted in the App Store][].
:::

[accepted in the App Store]: https://developer.apple.com/ipados/submit/
[large screens]: {{site.android-dev}}/guide/topics/large-screens/get-started-with-large-screens
[Play Store updates]: {{site.android-dev}}/2022/03/helping-users-discover-quality-apps-on.html

## Layout with GridView

Consider the following screenshots of an app.
The app displays its UI in a `ListView`.
The image on the left shows the app running
on a mobile device. The image on the right shows the
app running on a large screen device
_before the advice on this page was applied_.

![Sample of large screen](/assets/images/docs/ui/adaptive-responsive/large-screen.png){:width="90%"}

This is not optimal.

The [Android Large Screen App Quality Guidelines][guidelines]
and the "iOS equivalent"
say that neither text nor boxes should take up the
full screen width. How to solve this in an adaptive way?

[guidelines]: https://developer.android.com/docs/quality-guidelines/large-screen-app-quality

The answer is to either use `GridView` or the
`maxWidth` property of `BoxConstraints`.

### GridView

You can use the `GridView` widget to transform
your existing `ListView` into more reasonably-sized items.

`GridView` is similar to the [`ListView`][] widget,
but instead of handling only a list of widgets arranged linearly,
`GridView` can arrange widgets in a two-dimensional array.

`GridView` also has constructors that are similar to `ListView`.
The `ListView` default constructor maps to `GridView.count`,
and `ListView.builder` is similar to `GridView.builder`.

`GridView` has some additional constructors for more custom layouts.
To learn more, visit the [`GridView`][] API page.

[`GridView`]: {{site.api}}/flutter/widgets/GridView-class.html
[`ListView`]: {{site.api}}/flutter/widgets/ListView-class.html

For example, if your original app used a `ListView.builder`,
swap that out for a `GridView.builder`.
If your app has a large number of items,
it’s recommended to use this builder constructor to only
build the item widgets that are actually visible.

Most of the parameters in the constructor are the same between
the two widgets, so it's a straightforward swap.
However, you need to figure out what to set for the `gridDelegate`.

Flutter provides powerful premade `gridDelegates`
that you can use, namely:

[`SliverGridDelegateWith<b>FixedCrossAxisCount</b>`][]
: Lets you assign a specific number of columns to your grid.

[`SliverGridDelegateWith<b>MaxCrossAxisExtent</b>`][] 
: Lets you define a max item width.

[`SliverGridDelegateWith<b>FixedCrossAxisCount</b>`]: {{site.api}}/flutter/rendering/SliverGridDelegateWithFixedCrossAxisCount-class.html 
[`SliverGridDelegateWith<b>MaxCrossAxisExtent</b>`]:  {{site.api}}/flutter/rendering/SliverGridDelegateWithMaxCrossAxisExtent-class.html

:::
Don't use the grid delegate for these classes that lets
you set the column count directly and then hardcode
the number of columns based on whether the device
is a tablet, or whatever. 
The number of columns should be based on the size of
the window and not the size of the physical device.

This distinction is important because many mobile
devices support multi-window mode, which can
cause your app to be rendered in a space smaller than
the physical size of the display. Also, Flutter apps
can run on web and desktop, which can be sized in
many ways. <strong>For this
reason, use `MediaQuery` to get the app window size
rather than the physical device size.</strong>
:::

You can solve this scenario in one of two ways:
* Wrap the `GridView`in a `ConstrainedBox` and give
  it a `BoxConstraints` with a maximum width set.
* Use a `Container` instead of a `ConstrainedBox`
  if you want other functionality like setting the
  background color.

For choosing the maximum width value,
consider using the values recommended
by Material 3 in the [Applying layout][] guide.

[Applying layout]: https://m3.material.io/foundations/layout/applying-layout/window-size-classes

## Foldables

As mentioned previously, Android and Flutter both
recommend in their design guidance **not**
to lock screen orientation,
but some developers lock screen orientation anyway.
Be aware that this can cause problems when running your
app on a foldable device.

When running on a foldable, the app might look ok
when the device is folded. But when unfolding,
you might find the app letterboxed.

As described in the [SafeArea & MediaQuery][] page,
letterboxing means that the app's window is locked to
the center of the screen while the window is
surrounded with black.

Why can this happen?

This can happen when using `MediaQuery` to figure out
the window size for yur app. When the device is folded,
orientation is restricted to portrait mode.
Under the hood, `setPreferredOrientations` causes
Android to use a portrait compatibility mode and the app
is displayed in a letterboxed state.
In the letterboxed state, `MediaQuery` never receives
the larger window size that allows the UI to expand.

You can solve this in one of two ways:

* Support all orientations.
* Use the dimensions of the _physical display_.
  In fact, this is one of the _few_ situations where
  you would use the physical display dimensions and
  _not_ the window dimensions.

How to obtain the physical screen dimensions?

You can use the [`Display`][] API, introduced in
Flutter 3.22/Dart 3.4, which contains the size,
pixel ratio, and the refresh rate of the physical device. 

<b>PENDING: Your notes said Flutter 3.13 introduced Display, but isn't it actually Flutter 3.22?</b>

[`Display`]: https://main-api.flutter.dev/flutter/dart-ui/Display-class.html

Here is some sample code for getting a `Display` object:

```
/// AppState object.
ui.FlutterView? _view;

@override
void didChangeDependencies() {
  super.didChangeDependencies();
  _view = View.maybeOf(context);
}

void didChangeMetrics() {
  final ui.Display? display = _view?.display;
}
```

The important thing is to find the display of the
view that you care about. This is a forward-looking
API that should handle current _and_ future multi-display
and multi-view devices.

<b>PENDING: What is tricky about selecting the "right" view? It feels like this explanation is incomplete</b>

<b>PENDING: OK, the next thing covered in the slides (starting with slide 66) is an actual example that switches between Dialog implementations. This would be hard to show without providing a specific example. I think it would be VERY useful to include this in the docs. Ditto for the navigation UI (starting with slide 81) and custom layout (slide 96). Thoughts?</b>

## Adaptive input

Adding support for more screens, also means
expanding input controls.

Android guidelines describe three tiers of large format device support.

![3 tiers of large format device support](/assets/images/docs/ui/adaptive-responsive/large-screen-guidelines.png){:width="90%"}

Tier 3, the lowest level of support,
[includes support for mouse and stylus input][].

If you app uses Material 3 and its buttons and selectors,
then your app already has built-in support for
various additional input states.

But what if you have a custom widget?
Check out the [User input][] page for
guidance on adding
[input support for widgets][].

[includes support for mouse and stylus input[]: {{site.android-dev}}/developer.android.com/docs/quality-guidelines/large-screen-app-quality
[input support for widgets]: /ui/adaptive-responsive/input
