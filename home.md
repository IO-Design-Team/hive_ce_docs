# Quick Start

## Add to project

Add the following to your `pubspec.yaml`:

```yaml
dependencies:
  hive_ce: latest
  hive_ce_flutter: latest

dev_dependencies:
  hive_ce_generator: latest
  build_runner: latest
```

## Initialize

Initializes Hive with a valid directory in your app files. You can also provide a subdirectory:

```dart
await Hive.initFlutter();
```

?> Use `Hive.init()` for non-Flutter apps.

## Open a Box

All of your data is stored in boxes.

```dart
final box = await Hive.openBox('testBox');
```

?> You may call `box('testBox')` to get the singleton instance of an already opened box.

## Read & Write

Hive supports all primitive types, `List`, `Set`, `Map`, `DateTime`, `Duration`, `BigInt` and `Uint8List`. Any object can be stored using [TypeAdapters](/custom-objects/generate_adapters.md).

```dart
import 'package:hive_ce/hive.dart';

void main() async {
  Hive.init('somePath') // not needed in browser

  final box = await Hive.openBox('testBox');

  box.put('name', 'David');

  print('Name: ${box.get('name')}');
}
```
