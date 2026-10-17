# Sketchpad Introduction

This tutorial shows you how to make a sketchpad app from scratch! You can draw and Hive will save all of it.

Along the way you will see how to generate an adapter for a class you don't own, like Flutter's `Offset`.

## Source Code

Here's the source: https://github.com/IO-Design-Team/hive_ce_samples/tree/master/sketchpad

## Setup

First we create a new Flutter project:

```shell
flutter create sketchpad
```

## Dependencies

We can then go ahead and add Hive and the tools needed to [generate `TypeAdapters`](/custom-objects/generate_adapters.md):

```shell
flutter pub add hive_ce hive_ce_flutter dev:hive_ce_generator dev:build_runner
```

## The model

Every stroke the user draws is a `ColoredPath`. It stores the index of the selected color and the list of points in the stroke.

The model is immutable. We never add points to an existing `ColoredPath`. Instead, a new `ColoredPath` is created with the updated list of points.

`lib/colored_path.dart`:

```dart
import 'package:flutter/material.dart';

class ColoredPath {
  static const colors = [
    Colors.black,
    Colors.red,
    Colors.green,
    Colors.blue,
    Colors.amber,
  ];

  final int colorIndex;
  final List<Offset> points;

  const ColoredPath({required this.colorIndex, required this.points});

  Color get color => colors[colorIndex];
}
```

?> The `color` getter is not in the constructor, so the generated adapter ignores it. Only the fields passed to the constructor are stored.

## Generating adapters

Hive needs an adapter for `ColoredPath`, and since `ColoredPath` contains a list of `Offset`s, it needs an adapter for `Offset` too. `Offset` comes from Flutter, but that's no problem. The generator can create adapters for classes from other packages as long as their constructor parameters match their fields.

`lib/hive/hive_adapters.dart`:

```dart
import 'dart:ui';

import 'package:hive_ce/hive_ce.dart';
import 'package:sketchpad/colored_path.dart';

@GenerateAdapters([AdapterSpec<ColoredPath>(), AdapterSpec<Offset>()])
part 'hive_adapters.g.dart';
```

Now run the build task:

```shell
dart run build_runner build
```

!> The generated `hive_adapters.g.yaml` file must be checked into version control. Read more [here](/custom-objects/generate_adapters.md).

## Initialization

We need to initialize Hive, register the generated adapters and open the box.

`lib/main.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:hive_ce_flutter/hive_ce_flutter.dart';
import 'package:sketchpad/colored_path.dart';
import 'package:sketchpad/hive/hive_registrar.g.dart';

const sketchBox = 'sketch';

void main() async {
  await Hive.initFlutter();
  Hive.registerAdapters();
  await Hive.openBox<ColoredPath>(sketchBox);
  runApp(const DrawApp());
}

class DrawApp extends StatelessWidget {
  const DrawApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: DrawingScreen());
  }
}
```

## Painting a path

The `PathPainter` draws a single `ColoredPath` on a canvas by connecting its points with lines.

```dart
class PathPainter extends CustomPainter {
  final ColoredPath path;

  const PathPainter(this.path);

  @override
  void paint(Canvas canvas, Size size) {
    final points = path.points;
    if (points.isEmpty) return;

    final linePath = Path()..moveTo(points.first.dx, points.first.dy);
    for (final point in points.skip(1)) {
      linePath.lineTo(point.dx, point.dy);
    }

    final paint = Paint()
      ..strokeCap = StrokeCap.round
      ..isAntiAlias = true
      ..color = path.color
      ..strokeWidth = 3
      ..style = PaintingStyle.stroke;

    canvas.drawPath(linePath, paint);
  }

  @override
  bool shouldRepaint(PathPainter oldDelegate) => true;
}
```

## Drawing

The `DrawingArea` widget handles the user's gestures. While the user is drawing, the current stroke is only kept in the widget's state. When the user lifts their finger, the finished `ColoredPath` is added to the box.

```dart
class DrawingArea extends StatefulWidget {
  final int selectedColorIndex;

  const DrawingArea(this.selectedColorIndex, {super.key});

  @override
  State<DrawingArea> createState() => _DrawingAreaState();
}

class _DrawingAreaState extends State<DrawingArea> {
  var points = <Offset>[];

  ColoredPath get path =>
      ColoredPath(colorIndex: widget.selectedColorIndex, points: points);

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onPanStart: (details) => addPoint(details.localPosition),
      onPanUpdate: (details) => addPoint(details.localPosition),
      onPanEnd: (details) {
        Hive.box<ColoredPath>(sketchBox).add(path);
        setState(() => points = []);
      },
      child: CustomPaint(size: Size.infinite, painter: PathPainter(path)),
    );
  }

  void addPoint(Offset point) => setState(() => points = [...points, point]);
}
```

!> Never modify an object after it has been written to a box. That's why `addPoint()` creates a new list instead of adding to the existing one.

## The drawing screen

The `DrawingScreen` puts everything together. It uses a `StreamBuilder` with `box.watch()` to redraw the saved paths whenever the box changes.

Below the canvas is a row of color circles to select the stroke color, a button to clear the sketch, and a button to undo the last stroke. Since strokes are stored with auto-increment keys, the last stroke is always at index `box.length - 1`.

```dart
class DrawingScreen extends StatefulWidget {
  const DrawingScreen({super.key});

  @override
  State<DrawingScreen> createState() => _DrawingScreenState();
}

class _DrawingScreenState extends State<DrawingScreen> {
  var selectedColorIndex = 0;

  @override
  Widget build(BuildContext context) {
    final box = Hive.box<ColoredPath>(sketchBox);
    return Scaffold(
      body: SafeArea(
        child: StreamBuilder(
          stream: box.watch(),
          builder: (context, snapshot) => Column(
            children: [
              Expanded(
                child: Stack(
                  children: [
                    for (final path in box.values)
                      CustomPaint(
                        size: Size.infinite,
                        painter: PathPainter(path),
                      ),
                    DrawingArea(selectedColorIndex),
                  ],
                ),
              ),
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  for (var i = 0; i < ColoredPath.colors.length; i++)
                    buildColorCircle(i),
                  IconButton(
                    icon: const Icon(Icons.delete),
                    onPressed: box.isEmpty ? null : box.clear,
                  ),
                  IconButton(
                    icon: const Icon(Icons.undo),
                    onPressed: box.isEmpty
                        ? null
                        : () => box.deleteAt(box.length - 1),
                  ),
                ],
              ),
              const SizedBox(height: 20),
            ],
          ),
        ),
      ),
    );
  }

  Widget buildColorCircle(int colorIndex) {
    final selected = selectedColorIndex == colorIndex;
    return GestureDetector(
      onTap: () => setState(() => selectedColorIndex = colorIndex),
      child: ClipOval(
        child: Container(
          height: selected ? 50 : 36,
          width: selected ? 50 : 36,
          color: ColoredPath.colors[colorIndex],
        ),
      ),
    );
  }
}
```

## The End

Congratulations, you have built a sketchpad that remembers everything you draw. Try adding more colors or a stroke width option!
