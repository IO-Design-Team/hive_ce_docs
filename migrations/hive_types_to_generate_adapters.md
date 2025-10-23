# HiveTypes to GenerateAdapters

If you already have model classes with `HiveType` and `HiveField` annotations, you can take the following steps to migrate to the new [GenerateAdapters](/custom-objects/generate_adapters.md) annotation:

1. Convert all default values to [constructor parameter defaults](https://dart.dev/null-safety/default-initialization-parameters)
2. Add the following to your `build.yaml` file:

```yaml
targets:
  $default:
    builders:
      hive_ce_generator|hive_schema_migrator:
        enabled: true
```

3. Run the `build_runner`. This will generate `lib/hive/hive_adapters.dart` and `lib/hive/hive_adapters.g.yaml`.
4. Revert the `build.yaml` changes
5. Remove all explicit `HiveType` and `HiveField` annotations from your model classes
6. Run the `build_runner` again
