# Next.js - The @ path alias
The _@/_ prefix is a path alias that maps to the root of your project. Instead of writing a relative path like _../../../auth_, you can always write _@/auth_ regardless of how deeply nested the importing file is.

The alias is defined in _tsconfig.json_ user _compilerOptions_:
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

## Related
- [[Next.js]]