# Version Compatibility Reference

Thunderbird releases align with Firefox ESR branches. Each major version
upgrade introduces changes that may break extensions, especially those
using Experiment APIs.

## Version Timeline

| Thunderbird | Based on Firefox | Status | Key Changes |
|-------------|-----------------|--------|-------------|
| 151 ESR | 151 ESR | Current | Latest ESR |
| 128 ESR | 128 ESR | Supported | MV3 support, ESM modules |
| 115 (Supernova) | 115 ESR | Supported | Major UI rewrite |
| 102 ESR | 102 ESR | End of Life | Compose API expansion |
| 91 ESR | 91 ESR | End of Life | WebExtension API milestone |
| 78 ESR | 78 ESR | End of Life | Initial WebExtension support |
| 68 ESR | 68 ESR | End of Life | Legacy bootstrapped extensions |

## TB 151 Changes

- Built on Firefox 151 ESR
- Continuing ESM migration of internal modules
- `strict_max_version` should be set to `"151.*"` when using Experiment APIs

## TB 128 Changes

- **Manifest V3 support added** (in addition to MV2)
  - `browser_action` → `action` in MV3
  - `background.service_worker` supported (experimental)
- **ESM module migration**: Internal Firefox modules use `.sys.mjs` extension
  - `ChromeUtils.import()` → `ChromeUtils.importESModule()`
  - Module paths now end in `.sys.mjs` (e.g., `ExtensionCommon.sys.mjs`)
- **`strict_min_version`** should be `"128.0"` for new extensions

## TB 115 (Supernova) Changes

- **Major UI rewrite**: Core Thunderbird windows restructured
- **CustomUI experiments may break**: The `messengercompose.xhtml` DOM structure changed significantly
  - `messageEditor` element still exists but parent hierarchy may differ
  - Settings dialog redesigned
- **Cards View**: New message list layout
- **Unified toolbar**: New customization system
- **`strict_max_version`**: If using CustomUI experiments, test thoroughly against TB 115+

### Adapting CustomUI Experiments for Supernova

Reference: `https://developer.thunderbird.net/add-ons/updating/tb115/adapt-to-changes-in-thunderbird-103-115`

Key changes:
- Compose window: `#composeContentBox` replaced with new layout
- Some XUL elements replaced with HTML equivalents
- Toolbar customization API changed
- Themes may need updates

## TB 102 Changes

- **Compose API**: Added `setComposeDetails()`, `getComposeDetails()` improvements
- **X- header support**: The standard compose API now supports adding custom headers (previously needed the ComposeMessageHeaders experiment)
- **Spaces toolbar**: New vertical toolbar introduced (experimental)
- **Address book**: Major redesign began

## TB 91 Changes

- **WebExtension API milestone**: Many APIs moved from experimental to stable
- **New APIs**: `composeScripts`, `messageDisplayScripts`, `messages`, `addressBooks`
- **Permissions model**: Refined permission requirements for existing APIs

## TB 78 Changes

- **Transition to MailExtensions**: First version where legacy overlay/bootstrapped extensions were deprecated
- **New APIs**: `compose`, `mailTabs`, `messageDisplay`
- **Manifest V2** became the standard

## Manifest V3 in Thunderbird

MV3 support was added in Thunderbird 128 (Beta 110). Key differences from MV2:

| Feature | MV2 | MV3 |
|---------|-----|-----|
| `browser_action` | ✓ | ✗ (use `action`) |
| `action` | ✗ | ✓ |
| `content_security_policy` | String | Object (split by type) |
| `web_accessible_resources` | Array of strings | Array of objects with `resources` and `matches` |
| `optional_permissions` | ✓ | ✗ (not yet supported in TB) |
| `background.scripts` | ✓ | ✓ |
| `background.service_worker` | ✗ | Experimental |

Currently, MV2 is recommended for broader compatibility. MV3 is still
maturing in Thunderbird and some features are incomplete.

## Setting Version Constraints

### Without Experiment APIs
```json
"browser_specific_settings": {
  "gecko": {
    "id": "my-ext@domain.com",
    "strict_min_version": "128.0"
    // strict_max_version can be omitted
  }
}
```

### With Experiment APIs
```json
"browser_specific_settings": {
  "gecko": {
    "id": "my-ext@domain.com",
    "strict_min_version": "128.0",
    "strict_max_version": "151.*"  // Pin to a specific branch
  }
}
```

Always pin `strict_max_version` when using Experiment APIs because:
- Internal Thunderbird APIs your experiment calls may change between versions
- The `.sys.mjs` module paths may change
- XUL/HTML element IDs or hierarchies may be reorganized

When a new Thunderbird ESR releases, test your extension and update `strict_max_version`.
