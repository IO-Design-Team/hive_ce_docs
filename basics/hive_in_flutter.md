# Hive in Flutter

Hive supports all platforms supported by Flutter.

## Initialize Flutter Apps

Before you can open a box, Hive needs to know where it can store its data. Android and iOS have very strict rules for allowed directories. You can use the `path_provider` package to get a valid directory.

The `hive_ce_flutter` package provides the `Hive.initFlutter()` extension method which handles everything for you.

## Listening for changes

If you want your widgets to refresh based on the data stored in Hive, you can use `box.watch()` with `StreamBuilder`.

```dart
import 'package:flutter/material.dart';

import 'package:hive_ce/hive.dart';
import 'package:hive_ce_flutter/hive_flutter.dart';

void main() async {
  await Hive.initFlutter();
  await Hive.openBox('settings');
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  final box = Hive.box('settings');

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Demo Settings',
      home: Scaffold(
        body: StreamBuilder(
          stream: box.watch(),
          builder: (context, snapshot) {
            return Center(
              child: Switch(
                value: box.get('darkMode', defaultValue: false),
                onChanged: (val) {
                  box.put('darkMode', val);
                },
              ),
            );
          },
        ),
      ),
    );
  }
}
```

Each time the value associated with `darkMode` changes, the `builder` is called, and the `Switch` widget refreshed.

### Listening for specific keys

It is good practice to refresh widgets only if necessary. If a widget only depends on specific keys in your box, you can filter the stream:

```dart
StreamBuilder(
  stream: Hive.box('settings').watch().where((event) => {'firstKey', 'secondKey'}.contains(event.key)),
  builder: (context, snapshot) {
    // build widget
  },
)
```

## Async calls

Many of the Hive methods like `put()` or `delete()` are asynchronous. Handling async calls in Flutter is not fun because you have to use `FutureBuilder` or `StreamBuilder` and handle exceptions.

The good news is that you don't have to await all the `Future`s returned by Hive. Changes are applied instantly, even if the `Future` has not finished yet. If you want to make sure that a write operation is successful, just await its `Future`.
