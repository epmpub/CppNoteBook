**To remove an old Boost version in vcpkg, use the special `boost-uninstall` port.** This is the recommended and cleanest way because Boost consists of many split packages (`boost-xxx`).

### Recommended command (Classic mode)

```bash
vcpkg install boost-uninstall
```

Or with `--recurse` if needed:

```bash
vcpkg install boost-uninstall --recurse
```

This special port is designed specifically to clean up **all** Boost-related packages (including old versions).

### Alternative ways

1. **Remove specific outdated packages** (useful if you just want to clean old versions):

   ```bash
   vcpkg remove --outdated
   ```

2. **Remove all Boost packages manually** (if `boost-uninstall` doesn't cover everything):

   ```bash
   vcpkg remove boost-uninstall --recurse
   ```

   Or target patterns (works in many cases):

   ```bash
   vcpkg remove boost*:* --recurse
   ```

### In Manifest mode (`vcpkg.json`)

- Add `"boost-uninstall"` temporarily to your dependencies, run `vcpkg install`, then remove it again.
- Or simply remove `"boost"` (and any `boost-*` overrides) from `vcpkg.json` and run `vcpkg install`.

### Additional cleanup tips

- Run `vcpkg list | grep boost` to see what Boost versions/packages are currently installed.
- After removal, you can install the desired Boost version with `vcpkg install boost` (or a specific version via overrides/baselines).
- Clean build trees/cache if you have disk space issues: `vcpkg remove --outdated` or manually delete folders under `buildtrees/boost*` and `packages/boost*`.

The `boost-uninstall` method is by far the most reliable for cleaning old Boost installations.

Let me know your vcpkg mode (Classic or Manifest) and triplet (e.g. `x64-windows`) if you need more specific commands!