# Transitive Hive dependencies

If you are using a package that depends on Hive v2, you can use the following workaround to force it to use Hive CE:

```yaml
dependencies:
  # Depend on hive_ce to prevent resolving breaking versions
  hive_ce: latest

dependency_overrides:
  hive:
    git:
      url: https://github.com/IO-Design-Team/hive_ce
      ref: 18f92b53295e9eb77ebd4830d905a72cd404a126
      path: overrides/hive
```
