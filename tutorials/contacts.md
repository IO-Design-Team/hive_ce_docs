# Contacts App Introduction

[![](https://img.shields.io/badge/author-%40Reprevise-blue)](https://github.com/Reprevise)

In this tutorial, we will build a fully functional app that stores your contacts.

We will be storing immutable model classes and enums directly in Hive. No more converting all of your models to JSON and your enums to strings. Let's get started!

## Source Code

Here's the source: https://github.com/IO-Design-Team/hive_ce_samples/tree/master/contacts

## Setup

First we create a new Flutter project:

```shell
flutter create contacts
```

## Dependencies

Now we add Hive and the tools needed to [generate `TypeAdapters`](/custom-objects/generate_adapters.md) automatically:

```shell
flutter pub add hive_ce hive_ce_flutter dev:hive_ce_generator dev:build_runner
```

## Models and Enums

We need a model class and an enum. The `Contact` class stores the information for a contact, and the `Relationship` enum defines how you know that person.

Models stored in Hive should be immutable. All of the fields are `final`, and the constructor has a parameter for every field. The generated adapter uses this constructor to create objects when reading them from the box.

The `Relationship` enum also carries a display label, so we don't need a separate conversion method or messy `if` statements.

`lib/contact.dart`:

```dart
enum Relationship {
  family('Family'),
  friend('Friend');

  final String label;

  const Relationship(this.label);
}

class Contact {
  final String name;
  final int age;
  final String phoneNumber;
  final Relationship relationship;

  const Contact({
    required this.name,
    required this.age,
    required this.phoneNumber,
    required this.relationship,
  });
}
```

?> Notice that there are no Hive annotations on the models. They are plain Dart classes.

## Generating adapters

Hive needs a `TypeAdapter` for every custom type it stores. Instead of writing them by hand, we tell the generator which types we want adapters for with the `GenerateAdapters` annotation.

`lib/hive/hive_adapters.dart`:

```dart
import 'package:contacts/contact.dart';
import 'package:hive_ce/hive_ce.dart';

@GenerateAdapters([AdapterSpec<Contact>(), AdapterSpec<Relationship>()])
part 'hive_adapters.g.dart';
```

Now run the build task:

```shell
dart run build_runner build
```

This generates three files in `lib/hive`:

- `hive_adapters.g.dart` contains the `ContactAdapter` and `RelationshipAdapter` classes
- `hive_adapters.g.yaml` is the Hive schema, which keeps track of type IDs and field indices as your models evolve
- `hive_registrar.g.dart` contains the `registerAdapters()` extension method

!> The `hive_adapters.g.yaml` file must be checked into version control. Read more [here](/custom-objects/generate_adapters.md).

!> Do _not_ modify the generated adapters. Rerun `build_runner` whenever you change a model.

## Initialization

Now we initialize Hive, register the generated adapters and open the box in the `main()` function.

`lib/main.dart`:

```dart
import 'package:contacts/contact.dart';
import 'package:contacts/hive/hive_registrar.g.dart';
import 'package:flutter/material.dart';
import 'package:hive_ce_flutter/hive_ce_flutter.dart';

const contactsBoxName = 'contacts';

void main() async {
  await Hive.initFlutter();
  Hive.registerAdapters();
  await Hive.openBox<Contact>(contactsBoxName);
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(title: 'Contacts App', home: ContactsScreen());
  }
}
```

?> We open the box in the `main()` method, so we can later use `Hive.box()` synchronously anywhere in the app.

## Listing contacts

The `ContactsScreen` widget shows all of the stored contacts. It uses a `StreamBuilder` with `box.watch()` to rebuild whenever the box changes.

If the box is empty, we show a `Text` widget notifying the user that they don't have any contacts. Otherwise we use a `ListView.builder` to show a `Card` for each contact.

We also use a `FloatingActionButton` to navigate the user to the `AddContact` screen.

!> Since we're not storing the keys ourselves, we're using `Box.getAt()` instead of `Box.get()`.

```dart
class ContactsScreen extends StatelessWidget {
  const ContactsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final box = Hive.box<Contact>(contactsBoxName);
    return Scaffold(
      appBar: AppBar(title: const Text('Contacts App with Hive')),
      body: StreamBuilder(
        stream: box.watch(),
        builder: (context, snapshot) {
          if (box.isEmpty) {
            return const Center(child: Text('No contacts'));
          }
          return ListView.builder(
            itemCount: box.length,
            itemBuilder: (context, index) {
              final contact = box.getAt(index)!;
              return Card(
                clipBehavior: Clip.antiAlias,
                child: InkWell(
                  onLongPress: () {
                    // We'll get back to this later
                  },
                  child: Padding(
                    padding: const EdgeInsets.all(8),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      spacing: 5,
                      children: [
                        Text(contact.name),
                        Text(contact.phoneNumber),
                        Text('Age: ${contact.age}'),
                        Text('Relationship: ${contact.relationship.label}'),
                      ],
                    ),
                  ),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: const Icon(Icons.add),
        onPressed: () => Navigator.of(
          context,
        ).push(MaterialPageRoute(builder: (context) => const AddContact())),
      ),
    );
  }
}
```

This is a very simple layout and I challenge you to improve upon it and make the app look gorgeous!

## Creating the form

The user needs to be able to create a contact, so let's build a form with a field for each property of `Contact`. Each field has a validator so the user can't submit an incomplete contact.

When the form is submitted, we create a new `Contact` and `add()` it to the box. Hive assigns it an auto-increment key. Read more [here](/basics/auto_increment.md).

?> For more information about form validation, click [here](https://docs.flutter.dev/cookbook/forms/validation).

```dart
class AddContact extends StatefulWidget {
  const AddContact({super.key});

  @override
  State<AddContact> createState() => _AddContactState();
}

class _AddContactState extends State<AddContact> {
  final formKey = GlobalKey<FormState>();

  String name = '';
  String age = '';
  String phoneNumber = '';
  Relationship? relationship;

  void onFormSubmit() {
    if (!formKey.currentState!.validate()) return;

    Hive.box<Contact>(contactsBoxName).add(
      Contact(
        name: name,
        age: int.parse(age),
        phoneNumber: phoneNumber,
        relationship: relationship!,
      ),
    );
    Navigator.of(context).pop();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Add Contact')),
      body: Form(
        key: formKey,
        child: ListView(
          padding: const EdgeInsets.all(8),
          children: [
            TextFormField(
              autofocus: true,
              decoration: const InputDecoration(labelText: 'Name'),
              validator: (value) =>
                  value == null || value.isEmpty ? 'Enter a name' : null,
              onChanged: (value) => name = value,
            ),
            TextFormField(
              keyboardType: TextInputType.number,
              maxLength: 3,
              decoration: const InputDecoration(labelText: 'Age'),
              validator: (value) => int.tryParse(value ?? '') == null
                  ? 'Enter a valid age'
                  : null,
              onChanged: (value) => age = value,
            ),
            TextFormField(
              keyboardType: TextInputType.phone,
              decoration: const InputDecoration(labelText: 'Phone'),
              validator: (value) => value == null || value.isEmpty
                  ? 'Enter a phone number'
                  : null,
              onChanged: (value) => phoneNumber = value,
            ),
            DropdownButtonFormField<Relationship>(
              hint: const Text('Relationship'),
              items: [
                for (final relationship in Relationship.values)
                  DropdownMenuItem(
                    value: relationship,
                    child: Text(relationship.label),
                  ),
              ],
              validator: (value) =>
                  value == null ? 'Select a relationship' : null,
              onChanged: (value) => relationship = value,
            ),
            const SizedBox(height: 16),
            OutlinedButton(
              onPressed: onFormSubmit,
              child: const Text('Submit'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## Deleting a contact

Uh oh, you have too many contacts and now you have to delete some. How do we do that?

Remember that `InkWell` widget inside the `Card`? We're going to use the `onLongPress` callback to open a dialog that asks the user whether they would like to delete the selected contact.

!> We're using `Box.deleteAt()` instead of `Box.delete()` because we're using auto-increment keys to store the contacts. Read more [here](/basics/auto_increment.md).

```dart
// inside of the `InkWell` widget
onLongPress: () => showDialog(
  context: context,
  builder: (context) => AlertDialog(
    content: Text('Do you want to delete ${contact.name}?'),
    actions: [
      TextButton(
        child: const Text('No'),
        onPressed: () => Navigator.of(context).pop(),
      ),
      TextButton(
        child: const Text('Yes'),
        onPressed: () {
          box.deleteAt(index);
          Navigator.of(context).pop();
        },
      ),
    ],
  ),
),
```

## The End

Congratulations, you have finished this tutorial where you have built a fully functional Contacts app. Feel free to change the UI to make it more beautiful than I did and add more fields for more information!

?> If you add a field to `Contact` later, rerun `build_runner`. Existing contacts can still be read as long as the new field is nullable or has a default value in the constructor. Read more [here](/custom-objects/generate_adapters.md#updating-a-class).
