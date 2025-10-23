# Isolates

Hive CE supports concurrent access across multiple isolates through the `IsolatedHive` interface

## The problem

The normal `Hive` interface is not safe to use across multiple isolates. Concurrent writes are almost guaranteed to corrupt box data.

?> Hive CE will print a warning in most cases when attempting to use `Hive` across multiple isolates

## Examples of multi-isolate usage

You may be using multiple isolates without even realizing it. Here are some common use-cases that result in code running in multiple isolates:

- A Flutter desktop app with multiple windows
- Running background tasks with [flutter_workmanager](https://pub.dev/packages/workmanager), [background_fetch](https://pub.dev/packages/background_fetch), etc
- Push notification processing

## IsolatedHive

`IsolatedHive` allows you to safely interact with boxes in a multi-isolate environment by maintaining its own separate isolate for `Hive` operations

!> If you are using `IsolatedHive`, you MUST use it everywhere in place of the normal `Hive` interface

`IsolatedHive` has a very similar API to `Hive`, but there are some key differences:

- Most methods are asynchronous due to isolate communication
- `IsolatedHive` does not support `HiveObject` or `HiveList`
- Isolate communication does add some overhead

?> On web, `IsolatedHive` directly calls `Hive` since web does not support isolates

## Usage

```dart
await IsolatedHive.initFlutter();
IsolatedHive.registerAdapters(); // if using generated adapters
final box = await IsolatedHive.openBox('box');
await box.put('key', 'value');
print(await box.get('key')); // reading is async
```

?> `IsolatedHive` does not have out of the box support for running in a Dart-only environment

## Example

See an example of a multi-window Flutter app using `IsolatedHive` [here](https://github.com/Rexios80/hive_ce_multiwindow)
