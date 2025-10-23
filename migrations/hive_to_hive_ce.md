# Hive to Hive CE

The `hive_ce` package is a drop in replacement for Hive v2. Make the following replacements in your project:

pubspec.yaml

```yaml
# old dependencies
dependencies:
  hive: ^2.0.0
  hive_flutter: ^1.0.0

dev_dependencies:
  hive_generator: ^1.0.0

# new dependencies
dependencies:
  hive_ce: latest
  hive_ce_flutter: latest

dev_dependencies:
  hive_ce_generator: latest
```

Dart files

```dart
// old imports
import 'package:hive/hive.dart';
import 'package:hive_flutter/hive_flutter.dart';

// new imports
import 'package:hive_ce/hive.dart';
import 'package:hive_ce_flutter/hive_flutter.dart';
```

!> Data from Hive v4 IS NOT compatible with Hive CE
