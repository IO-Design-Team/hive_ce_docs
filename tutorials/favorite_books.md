# Favorite Books Introduction

[![](https://img.shields.io/badge/author-%40Reprevise-blue)](https://github.com/Reprevise)

In this tutorial we will build a simple app which stores the user's favorite books. It features a list of popular books and data persistence all with Hive in under 100 lines of code!

## Source Code

Here's the source: https://github.com/IO-Design-Team/hive_ce_samples/tree/master/favorite_books

Below you can find the final code.

```dart
import 'package:flutter/material.dart';
import 'package:hive_ce_flutter/hive_ce_flutter.dart';

const favoritesBox = 'favorite_books';
const books = [
  'Harry Potter',
  'To Kill a Mockingbird',
  'The Hunger Games',
  'The Giver',
  'Brave New World',
  'Unwind',
  'World War Z',
  'The Lord of the Rings',
  'The Hobbit',
  'Moby Dick',
  'War and Peace',
  'Crime and Punishment',
  'The Adventures of Huckleberry Finn',
  'Catch-22',
  'The Sound and the Fury',
  'The Grapes of Wrath',
  'Heart of Darkness',
];

void main() async {
  await Hive.initFlutter();
  await Hive.openBox<String>(favoritesBox);
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  Box<String> get favoriteBooksBox => Hive.box<String>(favoritesBox);

  Widget getIcon(int index) {
    if (favoriteBooksBox.containsKey(index)) {
      return const Icon(Icons.favorite, color: Colors.red);
    }
    return const Icon(Icons.favorite_border);
  }

  void onFavoritePress(int index) {
    if (favoriteBooksBox.containsKey(index)) {
      favoriteBooksBox.delete(index);
      return;
    }
    favoriteBooksBox.put(index, books[index]);
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Favorite Books',
      home: Scaffold(
        appBar: AppBar(title: const Text('Favorite Books')),
        body: StreamBuilder(
          stream: favoriteBooksBox.watch(),
          builder: (context, snapshot) {
            return ListView.builder(
              itemCount: books.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text(books[index]),
                  trailing: IconButton(
                    icon: getIcon(index),
                    onPressed: () => onFavoritePress(index),
                  ),
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

## Setup

First we create a new Flutter project:

```shell
flutter create favorite_books
```

## Dependencies

We can then go ahead and add `hive_ce` and `hive_ce_flutter` to the project:

```shell
flutter pub add hive_ce hive_ce_flutter
```

## Initialization

I've defined a `const` variable to hold our `Box` name. Inside the `main()` function, we initialize Hive and open up the box. We also call `runApp()` to allow Flutter to build our app.

```dart
import 'package:flutter/material.dart';
import 'package:hive_ce_flutter/hive_ce_flutter.dart';

const favoritesBox = 'favorite_books';

void main() async {
  await Hive.initFlutter();
  await Hive.openBox<String>(favoritesBox);
  runApp(const MyApp());
}
```

## Our Data

For this app, we get data from a list of strings but you can easily do the same with an external API!

Each of the items in our list has a secret number called an index that Dart auto assigns for us, remember this for later! The list's index starts counting at 0, not 1!

```dart
const books = [
  // book name, index
  'Harry Potter', // 0
  'To Kill a Mockingbird', // 1
  'The Hunger Games', // 2
  'The Giver', // 3
  'Brave New World', // 4
  'Unwind', // 5
  'World War Z', // 6
  'The Lord of the Rings', // etc...
  'The Hobbit',
  'Moby Dick',
  'War and Peace',
  'Crime and Punishment',
  'The Adventures of Huckleberry Finn',
  'Catch-22',
  'The Sound and the Fury',
  'The Grapes of Wrath',
  'Heart of Darkness',
];
```

## MyApp Widget

Here's the `MyApp` class that we call inside of `runApp()`. We have some undefined functions and variables but we'll take care of those later.

The `MyApp` widget has a `Scaffold` which has a `StreamBuilder`. It listens to `box.watch()` and rebuilds the list every time our box changes.

Inside that builder is a `ListView` that holds all of the books in a `ListTile`.

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  // ...

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Favorite Books',
      home: Scaffold(
        appBar: AppBar(title: const Text('Favorite Books')),
        body: StreamBuilder(
          stream: favoriteBooksBox.watch(),
          builder: (context, snapshot) {
            return ListView.builder(
              itemCount: books.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text(books[index]),
                  trailing: IconButton(
                    icon: getIcon(index),
                    onPressed: () => onFavoritePress(index),
                  ),
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

## Getting the box

Before we can do anything with the box, we have to get it. We already opened the box when we initialized Hive. The great thing about Hive is that you can get boxes anywhere, you don't have to pass the box down from widget to widget. Just call `Hive.box()`. It's a synchronous method so no messy `async await` stuff. All of the values in the box are already in memory so we can access them instantly.

```dart
class MyApp extends StatelessWidget {
  // ...
  Box<String> get favoriteBooksBox => Hive.box<String>(favoritesBox);
  // ...
}
```

## Writing to the Box

Inside of `onFavoritePress`, we react to the favorite icon being pressed inside of the `ListTile` widget. Here, we're checking if our box already contains the book index and delete it if so because you can't favorite a book twice. If it doesn't contain the index, then it will be put inside the Hive box and the list will be rebuilt because we are listening to changes in the box using `box.watch()`.

!> In this scenario, the `putAt` function won't work as our `favoriteBooksBox`'s index is different from our book list's index! The `putAt` function is useful for updating data if you know the index it's in inside the Hive box.

```dart
class MyApp extends StatelessWidget {
  // ...
  void onFavoritePress(int index) {
    if (favoriteBooksBox.containsKey(index)) {
      favoriteBooksBox.delete(index);
      return;
    }
    favoriteBooksBox.put(index, books[index]);
  }
  // ...
}
```

## Reading from the Box

So, we added items to the box but there's still one more issue! How does the user know if a book is already favorited?

We are going to change the icon and its color if the book's index number is in the Hive box using the function below.

All the function does is return an `Icon` widget if our box contains the book index. Remember, the data stored in a Hive box is stored in a key-value store like a `Map`.

```dart
class MyApp extends StatelessWidget {
  // ...
  Widget getIcon(int index) {
    if (favoriteBooksBox.containsKey(index)) {
      return const Icon(Icons.favorite, color: Colors.red);
    }
    return const Icon(Icons.favorite_border);
  }
  // ...
}
```

## The End

Congratulations, you have finished this tutorial where you have built a fully functional app that saves your favorite books. Feel free to change the UI to make it more beautiful than I did and add more features like the ability to filter out those you did or didn't like.
