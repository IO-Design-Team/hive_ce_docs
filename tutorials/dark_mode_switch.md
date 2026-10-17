# Dark Mode Switch Introduction

In this tutorial we will build a super simple app. It will have a single switch which can toggle between dark mode and light mode. We will use Hive to persist the switch state.

## Source Code

Below you can find the final code.

```dart
import 'package:flutter/material.dart';
import 'package:hive_ce_flutter/hive_ce_flutter.dart';

const darkModeBox = 'darkModeTutorial';

void main() async {
  await Hive.initFlutter();
  await Hive.openBox<bool>(darkModeBox);
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    final box = Hive.box<bool>(darkModeBox);
    return StreamBuilder(
      stream: box.watch(key: 'darkMode'),
      builder: (context, snapshot) {
        final darkMode = box.get('darkMode', defaultValue: false)!;
        return MaterialApp(
          themeMode: darkMode ? ThemeMode.dark : ThemeMode.light,
          darkTheme: ThemeData.dark(),
          home: Scaffold(
            body: Center(
              child: Switch(
                value: darkMode,
                onChanged: (value) => box.put('darkMode', value),
              ),
            ),
          ),
        );
      },
    );
  }
}
```

## Setup

First we create a new Flutter project:

```shell
flutter create dark_mode_switch
```

## Dependencies

Now we add Hive to the project:

```shell
flutter pub add hive_ce hive_ce_flutter
```

## Initialization

Now we can import `hive_ce_flutter` to initialize Hive. It also exports everything from `hive_ce`.

```dart
import 'package:flutter/material.dart';
import 'package:hive_ce_flutter/hive_ce_flutter.dart';

const darkModeBox = 'darkModeTutorial';

void main() async {
  await Hive.initFlutter();
  await Hive.openBox<bool>(darkModeBox);
  runApp(const MyApp());
}
```

?> We open the box in the `main()` method, so we can later use `Hive.box()` and avoid dealing with async code.

## Structure of the app

The following is the main structure of our app. A Material themed app with a single `Switch` in the center.

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      darkTheme: ThemeData.dark(),
      home: Scaffold(
        body: Center(
          child: Switch(value: false, onChanged: (value) {}),
        ),
      ),
    );
  }
}
```

## Persisting the Switch state

Now we read the `darkMode` entry from the box. We provide a `defaultValue` because the value will be `null` when the app starts for the first time.

Based on the `darkMode` value we set the `themeMode` of the `MaterialApp`.

When the user toggles the switch, we update the `darkMode` entry in the box.

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    final box = Hive.box<bool>(darkModeBox);
    final darkMode = box.get('darkMode', defaultValue: false)!;
    return MaterialApp(
      themeMode: darkMode ? ThemeMode.dark : ThemeMode.light,
      darkTheme: ThemeData.dark(),
      home: Scaffold(
        body: Center(
          child: Switch(
            value: darkMode,
            onChanged: (value) => box.put('darkMode', value),
          ),
        ),
      ),
    );
  }
}
```

?> We can use `Hive.box(darkModeBox)` in `MyApp` because we opened the box before this widget is used.

When you run the example, you will notice that it does not work as intended. The reason is that we don't refresh our widgets based on the changed `darkMode` value.

## Refreshing

The last step is to refresh the app when necessary. The easiest way to refresh widgets based on Hive changes is using `box.watch()` with a `StreamBuilder`. Since we only care about the `darkMode` entry, we pass the `key` parameter to only get notified about changes to that key.

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    final box = Hive.box<bool>(darkModeBox);
    return StreamBuilder(
      stream: box.watch(key: 'darkMode'),
      builder: (context, snapshot) {
        final darkMode = box.get('darkMode', defaultValue: false)!;
        return MaterialApp(
          themeMode: darkMode ? ThemeMode.dark : ThemeMode.light,
          darkTheme: ThemeData.dark(),
          home: Scaffold(
            body: Center(
              child: Switch(
                value: darkMode,
                onChanged: (value) => box.put('darkMode', value),
              ),
            ),
          ),
        );
      },
    );
  }
}
```

!> In this example we rebuild the entire app when the `darkMode` value changes. You should only refresh the necessary widgets.
