!> Use `box.watch()` with `StreamBuilder` instead.

## ValueListenable

If you want your widgets to refresh based on the data stored in Hive, you can use the `ValueListenableBuilder`. The `box.listenable()` method provides a `ValueListenable` which can also be used with the `provider` package.

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
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Demo Settings',
      home: Scaffold(
        body: ValueListenableBuilder<Box>(
          valueListenable: Hive.box('settings').listenable(),
          builder: (context, box, widget) {
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

It is good practice to refresh widgets only if necessary. If a widget only depends on specific keys in your box, you can provide the `keys` parameter:

```dart
ValueListenableBuilder<Box>(
  valueListenable: Hive.box('settings').listenable(keys: ['firstKey', 'secondKey']),
  builder: (context, box, widget) {
    // build widget
  },
)
```
