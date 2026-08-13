# Contacts App Introduction

[![](https://img.shields.io/badge/author-%40Reprevise-blue)](https://github.com/Reprevise)

In this tutorial, we will be building a fully functional app that stores your contacts in under 230 lines of code!

We will be using immutable model classes and enums. Hive CE generates the adapters for them with `GenerateAdapters`, so you do not need `@HiveType` or `@HiveField` on every field. Let's get started!

## Source Code

Here's the source: https://github.com/IO-Design-Team/hive_ce_samples/tree/master/contacts

## Setup

First we create a new Flutter project:

```
flutter create contacts
```

## Dependencies

Add Hive CE and the generator. Adapters are generated with [`GenerateAdapters`](/custom-objects/generate_adapters.md).

```yaml
environment:
  sdk: ^3.4.0

dependencies:
  flutter:
    sdk: flutter
  hive_ce: latest
  hive_ce_flutter: latest

dev_dependencies:
  hive_ce_generator: latest
  build_runner: latest
  flutter_test:
    sdk: flutter
```

## Models and Enums

Keep models immutable: `final` fields and a `const` constructor. Hive CE does not need annotations on the class.

One field is a `Relationship` enum. Hive can store that the same way it stores the `Contact` class.

```dart
class Contact {
  const Contact({
    required this.name,
    required this.age,
    required this.phoneNumber,
    required this.relationship,
  });

  final String name;
  final int age;
  final String phoneNumber;
  final Relationship relationship;
}

enum Relationship { family, friend }

const relationshipString = <Relationship, String>{
  Relationship.family: 'Family',
  Relationship.friend: 'Friend',
};
```

## Generate adapters

Create `lib/hive/hive_adapters.dart` and list each type in `@GenerateAdapters`. Type IDs and field indexes live in the generated schema, not on the model.

```dart
import 'package:hive_ce/hive_ce.dart';
import '../contact.dart';

@GenerateAdapters([
  AdapterSpec<Contact>(),
  AdapterSpec<Relationship>(),
])
part 'hive_adapters.g.dart';
```

Then run:

```shell
dart run build_runner build --delete-conflicting-outputs
```

That generates:

- `hive_adapters.g.dart` — the adapter classes
- `hive_adapters.g.yaml` — the schema (check this in)
- `hive_registrar.g.dart` — `Hive.registerAdapters()`

!> Do not edit `hive_adapters.g.dart` or `hive_registrar.g.dart`. If a class or field is renamed, update `hive_adapters.g.yaml` as described [here](/custom-objects/generate_adapters.md).

## Initialization

Initialize Hive, register every generated adapter in one call, then open the box before `runApp()`.

```dart
import 'package:flutter/material.dart';
import 'package:hive_ce_flutter/hive_ce_flutter.dart';
import 'package:contacts/hive/hive_registrar.g.dart';
import 'package:contacts/contact.dart';

const contactsBoxName = 'contacts';

void main() async {
  await Hive.initFlutter();
  Hive.registerAdapters();
  await Hive.openBox<Contact>(contactsBoxName);
  runApp(const MyApp());
}
```

## Main App Structure

All of the UI code (except for the form) is in one widget, `MyApp`. It uses a `ValueListenableBuilder` so the list rebuilds when the box changes.

If the box is empty, we show a short message. Otherwise a `ListView.builder` reads each contact with `Box.getAt()`. A long press on a card asks to delete it. The FAB opens the `AddContact` screen.

!> Since we're not storing the keys ourselves, we're using `Box.getAt()` instead of `Box.get()`.

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Contacts App',
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Contacts App with Hive'),
        ),
        body: ValueListenableBuilder(
          valueListenable: Hive.box<Contact>(contactsBoxName).listenable(),
          builder: (context, Box<Contact> box, _) {
            if (box.values.isEmpty) {
              return const Center(
                child: Text('No contacts'),
              );
            }
            return ListView.builder(
              itemCount: box.values.length,
              itemBuilder: (context, index) {
                final currentContact = box.getAt(index)!;
                final relationship =
                    relationshipString[currentContact.relationship];
                return Card(
                  clipBehavior: Clip.antiAlias,
                  child: InkWell(
                    onLongPress: () {
                      showDialog(
                        context: context,
                        barrierDismissible: true,
                        builder: (context) => AlertDialog(
                          content: Text(
                            'Do you want to delete ${currentContact.name}?',
                          ),
                          actions: <Widget>[
                            TextButton(
                              child: const Text('No'),
                              onPressed: () => Navigator.of(context).pop(),
                            ),
                            TextButton(
                              child: const Text('Yes'),
                              onPressed: () async {
                                await box.deleteAt(index);
                                if (context.mounted) {
                                  Navigator.of(context).pop();
                                }
                              },
                            ),
                          ],
                        ),
                      );
                    },
                    child: Padding(
                      padding: const EdgeInsets.all(8.0),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: <Widget>[
                          const SizedBox(height: 5),
                          Text(currentContact.name),
                          const SizedBox(height: 5),
                          Text(currentContact.phoneNumber),
                          const SizedBox(height: 5),
                          Text('Age: ${currentContact.age}'),
                          const SizedBox(height: 5),
                          Text('Relationship: $relationship'),
                          const SizedBox(height: 5),
                        ],
                      ),
                    ),
                  ),
                );
              },
            );
          },
        ),
        floatingActionButton: Builder(
          builder: (context) {
            return FloatingActionButton(
              child: const Icon(Icons.add),
              onPressed: () {
                Navigator.of(context).push(
                  MaterialPageRoute(builder: (context) => AddContact()),
                );
              },
            );
          },
        ),
      ),
    );
  }
}
```

!> If you're copying the code, the `Builder` widget is required to make `Navigator.of(context)` work so don't remove it!

## Creating the form

The user needs to be able to create a contact so let's build a form for them to use.

This is a very simple form that does _not_ contain any validation whatsoever.

The fields we have are:

- Contact Name
- Contact Age
- Contact Phone
- Contact Relationship

Feel free to add as many fields as you like!

?> Note that we have an undefined method `onFormSubmit()`. We will take care of that in the next part.

?> Also note the `formKey` variable. This is used for form validation. For more information, click [here](https://flutter.dev/docs/cookbook/forms/validation).

```dart
class AddContact extends StatefulWidget {
  AddContact({super.key});

  final formKey = GlobalKey<FormState>();

  @override
  State<AddContact> createState() => _AddContactState();
}

class _AddContactState extends State<AddContact> {
  String name = '';
  int age = 0;
  String phoneNumber = '';
  Relationship? relationship;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: SingleChildScrollView(
          child: Form(
            key: widget.formKey,
            child: Padding(
              padding: const EdgeInsets.all(8.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: <Widget>[
                  TextFormField(
                    autofocus: true,
                    initialValue: '',
                    decoration: const InputDecoration(
                      labelText: 'Name',
                    ),
                    onChanged: (value) {
                      setState(() {
                        name = value;
                      });
                    },
                  ),
                  TextFormField(
                    keyboardType: TextInputType.number,
                    initialValue: '',
                    maxLength: 3,
                    decoration: const InputDecoration(
                      labelText: 'Age',
                    ),
                    onChanged: (value) {
                      setState(() {
                        age = int.parse(value);
                      });
                    },
                  ),
                  TextFormField(
                    keyboardType: TextInputType.phone,
                    initialValue: '',
                    decoration: const InputDecoration(
                      labelText: 'Phone',
                    ),
                    onChanged: (value) {
                      setState(() {
                        phoneNumber = value;
                      });
                    },
                  ),
                  DropdownButton<Relationship>(
                    items: relationshipString.keys.map((Relationship value) {
                      return DropdownMenuItem<Relationship>(
                        value: value,
                        child: Text(relationshipString[value]!),
                      );
                    }).toList(),
                    value: relationship,
                    hint: const Text('Relationship'),
                    onChanged: (value) {
                      setState(() {
                        relationship = value;
                      });
                    },
                  ),
                  OutlinedButton(
                    child: const Text('Submit'),
                    onPressed: onFormSubmit,
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

## Submitting the form

We have a form and a submit button for it, great! But we can't do anything with it right now. Let's fix that.

I've added a `onFormSubmit()` method that will take care of the submit process and add data to our box.

!> After you add form validation, check to see if the form is valid before adding data to the box. Click [here](https://flutter.dev/docs/cookbook/forms/validation) for more information.

```dart
class _AddContactState extends State<AddContact> {
  // ...

  void onFormSubmit() {
    final selected = relationship;
    if (selected == null) return;
    final contactsBox = Hive.box<Contact>(contactsBoxName);
    contactsBox.add(
      Contact(
        name: name,
        age: age,
        phoneNumber: phoneNumber,
        relationship: selected,
      ),
    );
    Navigator.of(context).pop();
  }

  @override
  Widget build(BuildContext context) {
    // ...
  }
}
```

## Deleting a contact

Uh oh, you have too many contacts and now you have to delete some. How do we do that?

Remember that `InkWell` widget above the `Card`? We're going to use the `onLongPress` callback to open a dialog that asks the user whether or not they would like to delete the selected contact.

!> We're using `Box.deleteAt()` instead of `Box.delete()` because we're using auto-incrementing keys to store the contacts. Read more [here](/basics/auto_increment.md).

```dart
// inside of `InkWell` widget
onLongPress: () {
  showDialog(
    context: context,
    barrierDismissible: true,
    builder: (context) => AlertDialog(
      content: Text(
        'Do you want to delete ${currentContact.name}?',
      ),
      actions: <Widget>[
        TextButton(
          child: const Text('No'),
          onPressed: () => Navigator.of(context).pop(),
        ),
        TextButton(
          child: const Text('Yes'),
          onPressed: () async {
            await box.deleteAt(index);
            if (context.mounted) {
              Navigator.of(context).pop();
            }
          },
        ),
      ],
    ),
  );
},
```

## The End

Congratulations, you have finished this tutorial where you have built a fully functional Contacts app. Feel free to change the UI to make it more beautiful than I did and add more fields for more information!
