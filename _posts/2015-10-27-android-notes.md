---
toc: true
toc_sticky: true
title: Android notes
category: notes
tags: android mobile sdk
image: android.png
---

This is a collection of notes I took during the **Mobile Programming** course at my University.

## Project

The project needs to follow a specific structure to be compiled and packaged correctly by the [Android Software Development Kit](https://developer.android.com/sdk/index.html).

At a higher level, we subdivide it in modules, such as:

* **Android application modules**: with code, resources, and configurations. These will be transformed into an *apk* file that will be installed to devices.
* **test modules**: containing [Junit](https://junit.org) tests.
* **library modules**: which are code shared with other projects. These will be included in the *apk* as well.

### Files

* `build/`: application build artifacts.
* `libs/`: private libraries.
* `src/`: the source code.
  * `androidTest/`: Java tests.
  * `main/java/com.project.app`: Java code such as *activities*.
  * `main/assets/`: store for *game assets*.
  * `main/res/`: various resources.
    * `anim/`: XML files compiled into *animation objects*.
    * `color/`: XML files with colors.
    * `drawable/`: bitmap files (PNG, JPEG, GIF), 9-Patch, and XML describing *drawables*.
    * `mipmap/`: Application icons.
    * `layout/`: XML files compiled into *layouts*.
    * `menu/`: XML files defining *menus*.
    * `raw/`: for various assets, like *mp3* or *ogg* files.
    * `values/`: XML files defining generic values.
    * `xml/`: other XML files.
  * `AndroidManifest.xml`: main file describing the applications and all of its components.
* `.gitignore/`: files to be ignored from [git](https://git-scm.com) versioning.
* `app.iml/`: *IntelliJ IDEA* config file.
* `build.gradle`: *build* config file.
* `proguard-rules.pro`: [ProGuard](https://proguard.sourceforge.net) config file.

## Localization

When the user launches an app, Android automatically selects and loads the right resources according to the device configuration.

In the `res/values/string.xml` files are stored strings that are used by default, though it is possible to specify alternative resources using _qualifiers_ for different languages.

For instance, to [localize](https://en.wikipedia.org/wiki/Language_localisation) text content in italian, we need to create an alternative `string.xml` file under the `res/values-it/` folder.

## Testing

It is possible to test our apps on an **emulator** or a reald Android **device**.

The number of _smartphones_ on the market is huge and continuously growing. Nevertheless every device is different and deciding which ones to test could be challenging.

It would be ideal to support as much devices as possible. We could buy a certain number of devices and test our app under real conditions, but it would definitely be more expensive that using the emulator.

The emulator might not allow us to see how apps work on real devices, but it does not hurt from a financial perspective.

When developing on a device, it is still good practive to use the emulator to test our apps on configurations that are different than those of real devices available to us. Even though the emulator can not test all settings, it can verify that the apps work correctly on different Android versions, or screen size and orientations.

A mix between real device testing and emulator testing could guarantee a good **test coverage** at a reasonable price.

## Layout

A _layout_ is the visual structure of a **user interface** and it can be defined in two ways:

* **XML**: Android provides XML tags corresponding to classes of the [View](https://developer.android.com/reference/android/view/package-summary.html).
* **Runtime**: you can create [View](https://developer.android.com/reference/android/view/View.html) and [ViewGroup](https://developer.android.com/reference/android/view/ViewGroup.html) objects programmatically.

We can use one or both methods to declare and manage the graphic interface of our application. For example, we could declare some layout in XML, and later add code to modify at _runtime_ the state of the elements on the screen.

The benefit of using XML files is the **separation** between presentation and code controlling the behaviour of the application. The description of the user interface is external to the logic code, enabling us to modify it without needing to recompile. Moreover, declaring the layout in XML makes it easier to visualize the structure of the user interface and it simplifies _debugging_.

### View

A View is basically a rectangle with **width** and **height**, and a position expressed as **left** and **top**. The unit is the pixel. We can also distinguish two pairs of width and height: _measured_ width and height relative to its parent, and _drawing_ width and height which define the size when drawing on screen.

Drawing a layout involves two steps: **measure** and **layout**. The measure step is implemented in [`measure(int, int)`](https://developer.android.com/reference/android/view/View.html#measure%28int,%20int%29) which is a _top-down traversal_ of the view-tree where their dimensions are calculated. The second step is implemented in [`layout(int, int, int,
int)`](https://developer.android.com/reference/android/view/View.html#layout%28int,%20int,%20int,%20int%29), top-down as well, where every parent is responsible to move their children using values calculated during the measure step.

## ViewGroup

### Linear Layout

A linear layout is a ViewGroup which aligns their children **horizontally** or **vertically** (`android:orientation`).

### Relative Layout

A relative layout is a ViewGroup which shows children with **relative** positions relative to their siblings (_left-of_, _below_, …) or parent (_bottom_, _left_, _center_, …).

## List View

A list view is a ViewGroup showing a **list of elements**. The elements are added automatically using an [Adapter](https://developer.android.com/reference/android/widget/Adapter.html), which takes the content of a source, like an _array_ or a _database_, and converts each element into a View to be inserted into the list.

### Grid View

A grid view is a ViewGroup which shows a **grid of elements** which, as the list view, is populated through an Adapter.

### Adapter View

When the content is **dynamic**, we can use an [AdapterView](https://developer.android.com/reference/android/widget/AdapterView.html) subclass to populate it at _runtime_.
An AdapterView subclass uses an Adapter, which behaves like a mediator between the data source and the layout. The Adapter receives data and converts each entry into a View that can be added to the layout.

Android provides some Adapter subclasses for some cases:

* **ArrayAdapter**: useful when the data source is an _array_. By default, it creates a View for every element of the array by calling `toString()` and puts it into a [TextView](https://developer.android.com/reference/android/widget/TextView.html).

```java
ArrayAdapter<String> adapter = new ArrayAdapter<>(
    this, android.R.layout.simple_list_item1, myStringArray);
ListView listView = (ListView) findViewById(R.id.listview);
listView.setAdapter(adapter);
```
* **SimpleCursorAdapter**: useful when data come from a [Cursor](https://developer.android.com/reference/android/database/Cursor.html).

## Toast

A Toast provides a simple **feedback** for an operation in a little _popup_. It occupies only the spaces required by the message and the current Activity remains visible and interactive. Toasts vanishes automatically after a _timeout_. We can create them with the static method [`Toast.makeText()`](https://developer.android.com/reference/android/widget/Toast.html#makeText%28android.content.Context,%20int,%20int%29), which takes three arguments (_context_, _message_ and _duration_), and can show with the `show()` method.

```java
Toast.makeText(context, "text", Toast.LENGTH_SHORT)
```

We can position with the [`setGravity()`](https://developer.android.com/reference/android/widget/Toast.html#setGravity%28int,%20int,%20int%29) method which accepts three arguments (_gravity constant_, _x offset_, and _y offset_).

```java
toast.setGravity(Gravity.TOP | Gravity.LEFT, 0, 0);
```

It is also possible to create a custom layout for toasts in XML and use it by passing it to the toast through the `setView(View)` method.
