# Experiment APIs — Detailed Guide

Experiment APIs are custom WebExtension APIs bundled with your add-on. They
execute in Thunderbird's parent process and have unrestricted access to
Thunderbird internals. They are the primary escape hatch when built-in
WebExtension APIs don't cover your use case.

## When to Use

- You need access to Thunderbird internals not exposed via WebExtension APIs
- You need to inject UI elements into core Thunderbird windows
- You need to interact with Thunderbird's native preferences, file system, or network stack
- You need to bridge between WebExtension code and Thunderbird's internal services

## Architecture

Experiments have three components:

```
api/MyAPI/
├── schema.json       # API surface definition (types, functions, events)
└── implementation.js # Parent-side implementation (Thunderbird main process)
```

An optional child implementation can also be provided for content-process logic,
but this is rarely needed since Thunderbird doesn't fully use multi-process.

### Registration in manifest.json

```json
"experiment_apis": {
  "MyAPI": {
    "schema": "api/MyAPI/schema.json",
    "parent": {
      "scopes": ["addon_parent"],
      "paths": [["MyAPI"]],
      "script": "api/MyAPI/implementation.js",
      "events": ["startup"]
    }
  }
}
```

Key properties:
- **`scopes`**: Typically `["addon_parent"]`. Determines which process the API loads in.
- **`paths`**: The namespace path. `[["MyAPI"]]` means `browser.MyAPI` or `messenger.MyAPI`.
- **`script`**: Implementation file path relative to extension root.
- **`events`**: `["startup"]` ensures the API loads at add-on startup (required for `onStartup()`).

### Schema File (schema.json)

The schema describes the API surface — types, functions, events, and their
signatures. Here's a minimal example:

```json
[
  {
    "namespace": "MyAPI",
    "functions": [
      {
        "name": "doSomething",
        "type": "function",
        "async": true,
        "parameters": [
          {
            "name": "param1",
            "type": "string"
          }
        ]
      }
    ],
    "events": [
      {
        "name": "onSomethingHappened",
        "type": "function",
        "parameters": [
          {
            "name": "data",
            "type": "object",
            "properties": {
              "key": { "type": "string" }
            }
          }
        ]
      }
    ]
  }
]
```

Rules:
- Functions that return a value **must** be marked `"async": true`.
- Events use the standard WebExtension event pattern (`.addListener()`, `.removeListener()`).
- Types can be `string`, `number`, `boolean`, `object` (with `properties`), `array` (with `items`), `any`, or `function`.

### Parent Implementation (implementation.js)

```javascript
var { ExtensionCommon } = ChromeUtils.importESModule(
  "resource://gre/modules/ExtensionCommon.sys.mjs"
);
var { ExtensionParent } = ChromeUtils.importESModule(
  "resource://gre/modules/ExtensionParent.sys.mjs"
);

var MyAPI = class extends ExtensionCommon.ExtensionAPI {
  getAPI(context) {
    // context provides: context.extension, context.cloneScope,
    // context.callOnClose(), etc.

    return {
      MyAPI: {
        async doSomething(param1) {
          // Access Thunderbird internals freely here
          console.log("MyAPI called with:", param1);
          return { result: "done" };
        }
      }
    };
  }

  onStartup() {
    // Called when the add-on loads (if events: ["startup"] is set)
  }

  onShutdown(isAppShutdown) {
    // Called when add-on is unloaded
    // CRITICAL: Invalidate startup cache on non-app shutdown
    if (!isAppShutdown) {
      Services.obs.notifyObservers(null, "startupcache-invalidate", null);
    }
  }
};
```

**Critical rules:**

1. **Invalidate cache on shutdown**: When the add-on is disabled/updated (not app shutdown), you **must** call:
   ```javascript
   Services.obs.notifyObservers(null, "startupcache-invalidate", null);
   ```
   Failure to do this causes cached Experiment APIs to persist across updates.

2. **No global variables**: Use API properties or closures instead. Global variables in Experiment implementations can collide across add-ons.

3. **Per-context API instances**: `getAPI(context)` is called once per WebExtension context. Use `context.callOnClose(callback)` for per-context cleanup.

## Context and Scope

### Accessing WebExtension Scope from Experiment

```javascript
// From parent implementation, get background page scope:
const webextScope = Array.from(extension.views).find(
  view => view.viewType === "background"
).xulBrowser.contentWindow.wrappedJSObject;
```

Note: This relies on single-process architecture and may break in future versions.

### Cloning and Scope Boundaries

Since Experiments run in a privileged scope, data crossing the boundary
needs careful handling:

**Experiment → WebExtension:**
Simple data structures (strings, numbers, plain objects, arrays) pass through
automatically via structured clone.

**WebExtension → Experiment access to privileged objects:**
Blocked by Xray vision. Workarounds:
- Use `Components.utils.cloneInto(value, context.cloneScope)` to copy objects into the unprivileged scope
- Use `context.wrapPromise(promise)` for async results (auto-clones)
- Use constructors from `context.cloneScope` directly
- Wrap async functions before cloning (they return Promises in the function's own scope)

**Experiment → WebExtension access to WebExtension objects:**
Use `Components.utils.waiveXrays(value)` to opt out of Xray vision.

## Complex Experiments with Multiple Files

For larger experiments, use `ChromeUtils.importESModule()` with resource:// URLs:

```javascript
// Register a resource:// namespace pointing to your modules directory
// This requires a resource registration step, typically done in onStartup()

const extension = ExtensionParent.GlobalManager.getExtension(
  "your-extension-id@domain.com"
);
const query = extension.manifest.version; // Cache-busting on update

var { MyModule } = ChromeUtils.importESModule(
  "resource://myaddon/MyModule.sys.mjs?" + query
);
```

The version query string ensures modules reload when the add-on updates (system modules can't be unloaded once loaded).

## Child Implementation (Rarely Needed)

A child implementation runs in the content process:
```json
"child": {
  "scopes": ["addon_child"],
  "paths": [["MyAPI"]],
  "script": "api/MyAPI/child.js"
}
```

Child implementations are useful when:
- You need to pass functions or custom class instances between processes
- You need better performance for frequently-called APIs
- Thunderbird's multi-process architecture evolves

Since Thunderbird currently runs mostly single-process, child implementations
are rarely required.

## Existing Shared Experiments

Before building your own, check if one of these already solves your problem:

| Experiment | Repository | Purpose |
|------------|-----------|---------|
| **CustomUI** | rsjtdrjgfuzkfg/thunderbird-experiments | iframe-based UI extension points |
| **LegacyCSS** | thunderbird/webext-support | Load custom CSS into Thunderbird windows |
| **LegacyPrefs** | thunderbird/webext-support | Access Thunderbird system preferences |
| **FileSystem** | thunderbird/webext-support | Access files in the profile folder |
| **NotificationBox** | thunderbird/webext-experiments | In-app notification banners |
| **Calendar** | thunderbird/webext-experiments | Draft calendar APIs |
| **TCP** | rsjtdrjgfuzkfg/thunderbird-experiments | TCP client based on ArrayBuffers |
| **Runtime.onDisable** | rsjtdrjgfuzkfg/thunderbird-experiments | Cleanup on disable/uninstall |
| **ComposeMessageHeaders** | gruemme/tb-api-compose_message_headers | Add headers to new messages |

## Debugging Experiment APIs

1. **Browser Console** (Ctrl+Shift+J): Shows `console.log` output from parent scope. Also shows syntax errors in implementation files.
2. **Cache Invalidation**: After modifying `implementation.js`, run `startupcache-invalidate` and restart.
3. **Schema errors**: Check the browser console for errors mentioning "schema" or "extension".
4. **Reload**: The "Reload" button in Debug Add-ons reloads the extension but does NOT clear the Experiment API cache. Always restart Thunderbird after Experiment changes.

## Tools

- **Experiment Generator**: https://darktrojan.github.io/experiments/ — Generates boilerplate schema + implementation files.
- **Built-in API implementations**: Reference implementations live in `comm/mail/components/extensions/parent/` in the Thunderbird source tree.
- **ExtensionCommon.sys.mjs**: The low-level extension framework code has detailed documentation in comments.
