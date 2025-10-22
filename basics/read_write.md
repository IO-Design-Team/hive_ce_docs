# Reading and Writing

## Read

Reading from a box is very straightforward:

```dart
final box = Hive.box('myBox');

final name = box.get('name'); // String

final birthday = box.get('birthday'); // DateTime
```

If the key does not exist, `null` is returned. Optionally you can specify a `defaultValue` that is returned in case the key does not exist.

```dart
final height = box.get('randomKey', defaultValue: 17.5);
```

Iterables of types other than `int`, `double`, `bool`, or `String` returned by `get()` are always of type `Iterable<dynamic>` \(Maps of type `Map<dynamic, dynamic>`\). Use `iterable.cast<SomeType>()` to cast them to a specific type.

## Every object only exists once

You always get the same instance of an object from a specific key. It does not matter for primitive values since primitives are immutable, but it is essential for all other objects.

Here is an example:

```dart
import 'package:hive_ce/hive.dart';

void main() async {
  final box = await Hive.openBox('someBox');

  final initialList = ['a', 'b', 'c'];
  box.put('myList', initialList);

  final myList = box.get('myList');
  myList[0] = 'd';

  print(initialList[0]); // d
}
```

The `initialList` and `myList` are the same instance and share the same data.

In the sample above, only the cached object has been changed and not the underlying data. To persist the change, `box.put('myList', myList)` needs to be called.

## Write

Writing to a box is almost like writing to a map. All keys have to be ASCII Strings with a max length of 255 chars or unsigned 32 bit integers.

```dart
final box = Hive.box('myBox');

box.put('name', 'Paul');

box.put('friends', ['Dave', 'Simon', 'Lisa']);

box.put(123, 'test');

box.putAll({'key1': 'value1', 42: 'life'});
```

You may wonder why writing works without async code. This is one of the main strengths of Hive.  
The changes are written to the disk as soon as possible in the background but all listeners are notified immediately. If the async operation fails \(which it should not\), all listeners are notified again with the old values.  
If you want to make sure that a write operation is successful, just await its `Future`.

This works differently for [lazy boxes](/advanced/lazy_box.md): As long as the `Future` returned by `put()` did not finish, `get()` returns the old values \(or `null` if it doesn't exist\).

The following code shows the difference:

```dart
final box = await Hive.openBox('box');

box.put('key', 'value');
print(box.get('key')); // value

final lazyBox = await Hive.openLazyBox('lazyBox');

final future = lazyBox.put('key', 'value');
print(lazyBox.get('key')); // null

await future;
print(lazyBox.get('key')); // value
```

## Delete

If you want to change an existing value, you can either override it using for example `put()` or delete it:

```dart
import 'package:hive_ce/hive.dart';

void main() async {
  final box = await Hive.openBox('deleteExample');

  box.putAll({'key1': 'good', 'key2':'evil'});
  print(box.values);

  box.delete('key2');
  print(box.values);
}
```

If the key does not exist, no disk access is needed and the returned `Future` finishes immediately.

### Write null vs delete

Writing `null` is **NOT** the same as deleting a value.

```dart
import 'package:hive_ce/hive.dart';

void main() async {
  final box = await Hive.openBox('writeNullBox');

  box.put('key', 'value');

  box.put('key', null);
  print(box.containsKey('key'));

  box.delete('key');
  print(box.containsKey('key'));
}
```
