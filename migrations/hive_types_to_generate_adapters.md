# HiveTypes to GenerateAdapters

If you already have model classes with `HiveType` and `HiveField` annotations, you can take the following steps to migrate to the new [GenerateAdapters](/custom-objects/generate_adapters.md) annotation:

1. Convert all default values to [constructor parameter defaults](/custom-objects/generate_adapters.md#default-value)
2. Run `dart run hive_ce_generator:migrate`. This will generate `lib/hive/hive_adapters.dart` and `lib/hive/hive_adapters.g.yaml`.
3. Remove all explicit `HiveType` and `HiveField` annotations from your model classes
4. Run the `build_runner`
