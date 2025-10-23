# BoxCollection

`BoxCollections` are a set of boxes which can be similarly used as normal boxes, except of that
they dramatically improve speed on web. They support opening and closing all boxes of a collection
at once and more efficiently store data in indexed DB on web.

Aside, they also expose Transactions which can be used to speed up tremendous numbers of database
transactions on web.

On `dart:io` platforms, there is no performance gain by BoxCollections or Transactions. Only
BoxCollections might be useful for some box hierarchy and development experience.

Custom objects must be json serializable in order to be used with BoxCollections.

?> For an object to be json serializable, it must implement a `toJson` method and a `fromJson` constructor. These can be generated using the [json_serializable](https://pub.dev/packages/json_serializable) package.

## Usage

```dart
// Create a box collection
final collection = await BoxCollection.open(
  // Name of your database
  'MyFirstFluffyBox',
  // Names of your boxes
  {'cats', 'dogs'},
  // Path where to store your boxes (Only used in Flutter / Dart IO)
  path: './',
  // Key to encrypt your boxes (Only used in Flutter / Dart IO)
  key: HiveCipherImpl(),
);

// Open your boxes. Optional: Give it a type.
final catsBox = await collection.openBox<Map>('cats');

// Put something in
await catsBox.put('fluffy', {'name': 'Fluffy', 'age': 4});
await catsBox.put('loki', {'name': 'Loki', 'age': 2});

// Get values of type (immutable) Map?
final loki = await catsBox.get('loki');
print('Loki is ${loki?['age']} years old.');

// Returns a List of values
final cats = await catsBox.getAll(['loki', 'fluffy']);
print(cats);

// Returns a List<String> of all keys
final allCatKeys = await catsBox.getAllKeys();
print(allCatKeys);

// Returns a Map<String, Map> with all keys and entries
final catMap = await catsBox.getAllValues();
print(catMap);

// delete one or more entries
await catsBox.delete('loki');
await catsBox.deleteAll(['loki', 'fluffy']);

// ...or clear the whole box at once
await catsBox.clear();

// Speed up write actions with transactions
await collection.transaction(
  () async {
    await catsBox.put('fluffy', {'name': 'Fluffy', 'age': 4});
    await catsBox.put('loki', {'name': 'Loki', 'age': 2});
    // ...
  },
  boxNames: ['cats'], // By default all boxes become blocked.
  readOnly: false,
);
```
