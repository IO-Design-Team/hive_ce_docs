# What are boxes?

All data stored in Hive is organized in boxes. A box can be compared to a table in SQL, but it does not have a structure and can contain anything.

For a small app, a single box might be enough. For more advanced problems, boxes are a great way to organize your data. Boxes can also be encrypted to store sensitive data.

## Open Box

Before you can use a box, you have to open it. For regular boxes, this loads all of its data from the local storage into memory for immediate access.

```dart
final box = await Hive.openBox<E>('testBox');
```

| Parameter          | Description                                                                                                                                                                                                       |
| :----------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name               | The name of the box specifies the storage location and is used to check if a box already exists. It is **case-insensitive**.                                                                                      |
| encryptionKey      | The key has to be a byte array with length 32 and is used to encrypt and decrypt all values in the box.                                                                                                           |
| keyComparator      | By default, keys are sorted lexicographically. This parameters allows you to provide a custom sorting order.                                                                                                      |
| compactionStrategy | Specify your own rules for automatic compaction.                                                                                                                                                                  |
| crashRecovery      | If your app is killed while a write operations is running, the last entry might be corrupted. This entry is deleted automatically when the app starts again. If you don't want this behavior, you can disable it. |
| path               | By default, boxes are stored in the directory given to `Hive.init()`. With this parameter you can specify the location where the box should be stored.                                                            |
| bytes              | Instead of using a file as backend, you can provide the box in binary form and open an in-memory box.                                                                                                             |
| `E`                | The optional type parameter specifies the type of the values in the box.                                                                                                                                          |

If the box is already open, it is returned immediately and all supplied parameters are ignored.

Once you obtained a box instance, you can read, write, and delete entries.

## Get open box

Hive stores a reference to all open boxes. If you want to get an already opened box, you can use

```dart
final box = Hive.box('myBox');
```

This method is especially useful for Flutter apps because you don't need to pass the box between widgets.

## Close box

If you don't need a box again, you should close it. All cached keys and values of the box will be dropped from memory and the box file is closed after all active read and write operations finished.

?> It is perfectly fine to leave a box open for the runtime of the app. If you need a box again in the future, just leave it open.

```dart
final box = await Hive.openBox('myBox');
await box.put('hello', 'world');
await box.close();
```

Before your application exits, you should call `Hive.close()` to close all open boxes. Don't worry if the app is killed before you close Hive, it doesn't matter.

## Type parameter: Box&lt;E&gt;

When you open a box, you can specify that it may only contain values of a specific type. For example a user box could be opened like this:

```dart
final box = await Hive.openBox<User>('users');

box.add(User());

box.add(5); // Compile time error
```

This box can also contain subtypes of `User`.

It is important that you provide the same type parameter to `Hive.box()`. You cannot open the same box multiple times with different type parameters.

```dart
await Hive.openBox<User>('users');

Hive.box<User>('users'); // OK

Hive.box('users'); // ERROR
Hive.box<Dog>('users'); // ERROR
```

!> Generic type parameters like Box<List<int>> are unsupported due to Dart limitations.
