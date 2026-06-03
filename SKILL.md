---
name: thunderbird-extension
description: >
  Build, debug, and modify Thunderbird MailExtensions (add-ons). Use this skill
  whenever the user needs to create a new Thunderbird extension, modify an
  existing one, fix compatibility issues, work with manifest.json, add
  Experiment APIs, set up compose/browser/message_display actions, register
  composeScripts or messageDisplayScripts, use Thunderbird-specific APIs like
  accounts, folders, messages, or troubleshoot extension loading/runtime
  problems. Also use when the user mentions Thunderbird add-ons, .xpi files,
  web-ext build, mail extension, or MailExtension — even if they don't say
  "extension" or "add-on" explicitly, but are clearly working inside a
  Thunderbird extension project or want to add features to Thunderbird.
---

# Thunderbird MailExtension Development

This skill covers the full lifecycle of Thunderbird extension development:
project structure, manifest configuration, API usage, Experiment APIs, debugging,
building, and compatibility handling.

## Quick Reference

The following reference files contain exhaustive tables and schemas.
Read them when you need specific details:

- **`references/manifest-keys.md`** — Every supported manifest key, MV2 vs MV3,
  Thunderbird-specific keys, and permission requirements.
- **`references/webextension-apis.md`** — Complete list of supported
  Thunderbird-specific APIs and Firefox-compatible APIs, with required
  permissions.
- **`references/experiments.md`** — How to write Experiment APIs (schema,
  parent/child implementation, lifecycle, scope handling, debugging).
- **`references/version-updates.md`** — Breaking changes per Thunderbird
  version (TB 68 through TB 151).

## Extension Structure

A minimal Thunderbird extension:

```
my-extension/
├── manifest.json          # Required: extension configuration
├── background.js          # Background script (or background.html)
├── _locales/              # Optional: localized strings
│   └── en/
│       └── messages.json
├── options/               # Optional: options page
│   └── options.html
├── images/                # Icons
└── api/                   # Optional: Experiment APIs
    └── MyExperiment/
        ├── schema.json
        └── implementation.js
```

### manifest.json Essentials

```json
{
  "manifest_version": 2,
  "name": "My Extension",
  "version": "1.0",
  "description": "What it does",
  "author": "Your Name",
  "browser_specific_settings": {
    "gecko": {
      "id": "my-extension@my-domain.com",
      "strict_min_version": "128.0",
      "strict_max_version": "151.*"
    }
  },
  "icons": { "64": "images/icon.png" },
  "background": { "scripts": ["background.js"], "type": "module" },
  "options_ui": { "page": "options/options.html" },
  "permissions": ["storage"]
}
```

Key rules:
- **manifest_version**: Use `2` (stable, widely supported) or `3` (since TB 128). MV3 in Thunderbird is still maturing.
- **id**: Must be an email-style ID (`something@your-domain.com`) or a UUID in curly braces. Cannot be changed once published.
- **strict_min_version**: Set to the oldest Thunderbird version you support.
- **strict_max_version**: Only needed if you use Experiment APIs (they may break across versions). Use `"151.*"` to target a specific branch.

## Development Workflow

### Building

```bash
# Install dependencies
npm install

# Build with web-ext
npx web-ext build -s extension -n my-extension.xpi

# Output: web-ext-artifacts/my-extension.xpi
```

The `web-ext` tool packages everything under the source directory (`-s`) into a
signed or unsigned `.xpi` file.

### Installing for Testing

1. Open Thunderbird → Add-ons Manager (Tools → Add-ons and Themes)
2. Click the gear icon → "Debug Add-ons"
3. Click "Load Temporary Add-on" → select your `manifest.json`
4. Or: "Install Add-on From File" → select the `.xpi`

Temporary loading allows reloading without restarting Thunderbird.

### Hot Reload

During development, use the debug add-ons page to reload:
- Click "Reload" on your extension's debug card
- This re-reads the manifest and scripts without a Thunderbird restart
- For Experiment API changes: you must restart Thunderbird (JS caching)

### Debugging

- **Background scripts / extension pages**: Open "Debug Add-ons" → click "Inspect" on your extension. This opens a dedicated developer toolbox.
- **Experiment APIs (parent scope)**: Use the global browser console (Ctrl+Shift+J) or developer toolbox (Ctrl+Shift+I). Enable "Show Content Messages" in the toolbox settings to see logs from popups and content scripts.
- **Content scripts (composeScripts, messageDisplayScripts)**: These execute in the content process. Logs appear in the associated tab's console. Use the global browser console with content messages enabled.

### Forcing Experiment API Cache Invalidation

When you modify Experiment API implementation files, Thunderbird may cache the
old version. Purge the cache by running this in the console before restarting:

```javascript
Services.obs.notifyObservers(null, "startupcache-invalidate", null);
```

Or launch Thunderbird with `-purgecaches` as a command-line option.

## Common Patterns

### Adding a Toolbar Button

**Compose window button** (for email composition features):

```json
"compose_action": {
  "default_title": "My Button",
  "default_icon": "images/icon.svg",
  "default_popup": "popup.html"
}
```

**Main window button** (for global features):

```json
"browser_action": {
  "default_title": "My Button",
  "default_icon": "images/icon.svg",
  "default_popup": "popup.html"
}
```

**Message display button** (for per-message features):

```json
"message_display_action": {
  "default_title": "My Button",
  "default_icon": "images/icon.svg",
  "default_popup": "popup.html"
}
```

In MV3, `browser_action` is renamed to `action`.

Each action has a corresponding API (`browser.composeAction`, `browser.browserAction`,
`browser.messageDisplayAction`) for programmatic control.

### Injecting Scripts into Compose/Message Windows

Thunderbird supports content scripts in compose and message display windows,
but they must be registered programmatically (not via manifest):

```javascript
// In background script
messenger.composeScripts.register({
  js: [{ file: "compose-content.js" }],
  css: [{ file: "compose-styles.css" }]
});

messenger.messageDisplayScripts.register({
  js: [{ file: "message-display.js" }]
});
```

These scripts have restricted API access — they can use `runtime`, `i18n`,
`storage`, and `menus.getTargetElement`. For anything else, communicate with
the background script via `runtime.sendMessage`.

### Communicating Between Scripts

Background → Content (compose/messageDisplay):
```javascript
// Background
const tabs = await messenger.tabs.query({ windowType: "messageCompose" });
await messenger.tabs.sendMessage(tabs[0].id, { action: "doSomething" });
```

Content → Background:
```javascript
// Content script
const result = await messenger.runtime.sendMessage({ action: "getData" });
```

Background → Preview iframe (via Experiment APIs):
Use the Experiment API's messaging bridge. For CustomUI-based experiments,
`messageManager.sendAsyncMessage` and `addMessageListener` provide the transport.

### Working with Thunderbird-Specific APIs

Thunderbird adds APIs not present in Firefox. The key ones:

| API | Permission | Purpose |
|-----|-----------|---------|
| `messenger.accounts` | `accountsRead` | List/manage email accounts |
| `messenger.folders` | `accountsFolders` | Browse folder hierarchy |
| `messenger.messages` | `messagesRead`, `messagesMove` | List/move/delete messages |
| `messenger.compose` | `compose` | Open/manage compose windows |
| `messenger.mailTabs` | `accountsFolders` | Access the main mail tab |
| `messenger.addressBooks` | `addressBooks` | Read/write address books |
| `messenger.cloudFile` | manifest key only | Cloud file upload providers |

Always use the `messenger.` namespace (not `browser.`) for Thunderbird-specific
APIs. Both namespaces are available — `browser.` gives access to standard
WebExtension APIs, `messenger.` to Thunderbird ones.

### Using Experiment APIs

Experiment APIs are custom APIs bundled with your extension to access
Thunderbird internals not exposed through standard WebExtension APIs.

**Important**: Including any Experiment API replaces individual permission
prompts with a single "full, unrestricted access" prompt. Optional permissions
are unsupported when Experiment APIs are present.

Registration in manifest.json:
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

The implementation file receives an `ExtensionAPI` class:
```javascript
var MyAPI = class extends ExtensionCommon.ExtensionAPI {
  getAPI(context) {
    return {
      MyAPI: {
        async myMethod(param) {
          // Implementation in parent scope
          return result;
        }
      }
    };
  }
};
```

For detailed Experiment API guidance, read `references/experiments.md`.

## Version Compatibility

Thunderbird releases align with Firefox ESR branches. Each major version
upgrade can introduce breaking changes:

- **TB 128 (ESR)**: Added MV3 support, ESM module migration (`.sys.mjs`)
- **TB 115 (Supernova)**: Major UI rewrite, CustomUI experiments need adaptation
- **TB 102**: Many compose API changes, `X-` header support added
- **TB 91**: WebExtension API milestone, many new APIs added

Read `references/version-updates.md` for the complete version history and
breaking changes.

When setting `strict_max_version`:
- Without Experiment APIs: usually safe to omit or set broadly
- With Experiment APIs: pin to a specific branch (e.g., `"151.*"`) because
  internal APIs your experiment depends on may change

## Troubleshooting

### Extension won't load / "corrupt" warning
- Check JSON syntax in manifest.json (trailing commas are invalid)
- Verify all file paths referenced in the manifest exist
- Ensure the `id` in `browser_specific_settings.gecko` is present and unique
- Check `strict_min_version` is not higher than your Thunderbird version

### Experiment API not working
- Run `startupcache-invalidate` and restart Thunderbird
- Check the browser console (Ctrl+Shift+J) for errors in the implementation
- Verify `paths` in manifest match the namespace used in code (e.g., `browser.MyAPI`)
- Ensure `events: ["startup"]` is set if you need `onStartup()`

### Content script not loading
- composeScripts/messageDisplayScripts must be registered via the API (not manifest)
- Content scripts are limited to `runtime`, `i18n`, `storage`, and `menus.getTargetElement`
- Use `runtime.sendMessage` for access to other APIs

### Button not appearing in toolbar
- Verify the manifest key name matches your Thunderbird version (MV2: `browser_action`, MV3: `action`)
- Ensure icon files exist at the specified paths
- Check `default_area` if specified (e.g., `"formattoolbar"` for compose_action)

## External Resources

- **API Reference**: https://webextension-api.thunderbird.net/ — official API docs
- **Developer Docs**: https://developer.thunderbird.net/add-ons/mailextensions
- **Example Extensions**: https://github.com/thunderbird/webext-examples
- **Shared Experiments**: https://github.com/thunderbird/webext-experiments
- **Community Support**: https://discuss.thunderbird.net/groups/addons
- **Experiment Generator**: https://darktrojan.github.io/experiments/
