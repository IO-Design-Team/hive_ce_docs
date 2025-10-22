# Generate adapters

The [hive_ce_generator](https://pub.dev/packages/hive_ce_generator) package can automatically generate `TypeAdapter`s for almost any class. You can even generate adapters for external packages such as a generated OpenAPI client.

1. To generate `TypeAdapter`s for classes, create the file `lib/hive/hive_adapters.dart`
2. Create a `GenerateAdapters` annotation and add `AdapterSpec`s for each class you want to generate an adapter for
3. Run the build task `dart run build_runner build`
4. Register the generated adapters by calling the generated `Hive.registerAdapters()` method

## Generated files

### hive_adapters.g.dart

This file contains the generated adapter classes.

### hive_adapters.g.yaml

The Hive schema is a generated yaml file that contains the information necessary to incrementally update the generated TypeAdapters as your model classes evolve.

!> There will be a lot of churn in this file during initial development. Make sure to delete `hive_adapters.g.yaml` and regenerate before the first real deployment of your application to reclaim unused field indices.

Some migrations may require manual modifications to the Hive schema file. One example is class/field renaming. Without manual intervention, the generator will see both an added and removed class/field. To resolve this, manually rename the class/field in the schema.

### hive_registrar.g.dart

This file contains an extension method to register all generated adapters.

### Example

Model class:

```dart
class Person {
  final String name;
  final int age;
  final List<Person> friends;

  const Person({
    required this.name,
    required this.age,
    required this.friends,
  });
}
```

`lib/hive/hive_adapters.dart`:

```dart
import 'package:hive_ce/hive.dart';
import 'person.dart';

@GenerateAdapters([AdapterSpec<Person>()])
part 'hive_adapters.g.dart';
```

Adapter registration:

```dart
import 'package:hive_ce/hive.dart';
import 'package:your_package/hive/hive_registrar.g.dart';

void main() {
  Hive
    ..init('.')
    ..registerAdapters();
}
```

The above code generates and registers an adapter class called `PersonAdapter`.

## Updating a class

If an existing class needs to be changed – for example, you'd like the class to have a new field – but you'd still like to read objects written with the old adapter, don't worry! It is straightforward to update generated adapters without breaking any of your existing code. Just remember the following rules:

- If you add new fields, any objects written by the "old" adapter can still be read by the new adapter. These fields are just ignored. Similarly, objects written by your new code can be read by your old code: the new field is ignored when parsing.
- Fields can be renamed, but this requires manually modifying the field key in the `hive_adapters.g.yaml` file
- Fields can be removed
- Changing the type of a field is not supported. You should create a new one instead.
- You have to provide a default value in the constructor for new non-nullable fields

## Enums

Generating an adapter for enums works exactly as it does for classes.

For updating the enum, the same rules apply as above.

## Default value

You can provide default values to properties and fields by providing a default value in the constructor.

```dart
class Customer {
  final double balance;

  const Customer({
    this.balance = 0,
  });
}
```
