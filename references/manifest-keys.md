# Supported Manifest Keys — Complete Reference

## Standard Keys (MV2 & MV3)

| Key | MV2 | MV3 | Description |
|-----|-----|-----|-------------|
| `manifest_version` | ✓ | ✓ | **Required.** `2` or `3` (MV3 since TB 128) |
| `name` | ✓ | ✓ | **Required.** Extension display name. Localizable. |
| `version` | ✓ | ✓ | **Required.** Dotted number string. |
| `description` | ✓ | ✓ | Short description in Add-ons Manager. Localizable. |
| `author` | ✓ | ✓ | Developer name. Overridden by `developer.name`. Localizable. |
| `homepage_url` | ✓ | ✓ | Extension home page URL. Localizable. |
| `developer` | ✓ | ✓ | `{ name, url }` object. Overrides `author` and `homepage_url`. |
| `icons` | ✓ | ✓ | Map of size→path. Sizes: 16, 32, 48, 64, 128, 512. Supports PNG and SVG. |
| `default_locale` | ✓ | ✓ | Required if `_locales/` exists. Identifies default language. |
| `short_name` | ✓ | ✓ | ≤12 characters. Used where full name is too long. Localizable. |
| `background` | ✓ | ✓ | Background page/scripts. Use `"type": "module"` for ES modules. |
| `options_ui` | ✓ | ✓ | `{ page, open_in_tab?, browser_style? }` |
| `permissions` | ✓ | ✓ | Array of permission strings. |
| `commands` | ✓ | ✓ | Keyboard shortcuts for extension actions. Supports `suggested_key` with platform variants. |
| `content_scripts` | ✓ | ✗ | Inject into web pages (not email messages). Use composeScripts/messageDisplayScripts for messages. |
| `content_security_policy` | ✓ | ✗ | CSP string restricting script/object sources. |
| `optional_permissions` | ✓ | ✗ | Runtime-requestable permissions. Not supported when Experiment APIs are present. |
| `web_accessible_resources` | ✓ | ✗ | Resources exposed to web pages. |
| `user_scripts` | ✓ | ✗ | Custom methods for user scripts API. |
| `dictionaries` | ✓ | ✓ | Locale→dictionary extension mapping. |
| `protocol_handlers` | ✓ | ✓ | Web-based protocol handler registration. |

## Thunderbird-Specific Keys

| Key | MV2 | MV3 | Description |
|-----|-----|-----|-------------|
| `browser_action` | ✓ | ✗ | Button in main toolbar. Renamed to `action` in MV3. |
| `action` | ✗ | ✓ | MV3 equivalent of `browser_action`. |
| `compose_action` | ✓ | ✓ | Button in compose window toolbar. Supports `default_area: "formattoolbar"`. |
| `message_display_action` | ✓ | ✓ | Button in message display toolbar. |
| `cloud_file` | ✓ | ✓ | Cloud file upload provider configuration. |
| `theme` | ✓ | ✓ | Static theme definition for Thunderbird UI. |
| `theme_experiment` | ✓ | ✓ | Experimental theme properties. |

### Action Button Configuration

All three action types (`browser_action`, `compose_action`, `message_display_action`) support:

```json
{
  "default_title": "Tooltip text",
  "default_icon": "path/to/icon.png",
  "default_popup": "popup.html",
  "default_area": "formattoolbar",
  "browser_style": true,
  "theme_icons": [
    { "dark": "icons/dark.svg", "light": "icons/light.svg", "size": 16 }
  ]
}
```

- `default_popup`: Clicking opens this HTML page in a popup
- `default_area`: Where to place the button (`"formattoolbar"`, `"maintoolbar"`, etc.)
- `theme_icons`: Dark/light theme icon variants (compose_action only)
- All three can also function as menu-type action buttons (dropdown instead of popup)

## Deprecated / Renamed

| Key | Notes |
|-----|-------|
| `applications` | Deprecated. Use `browser_specific_settings` instead. MV2 only. |

## `browser_specific_settings.gecko` Properties

```json
{
  "browser_specific_settings": {
    "gecko": {
      "id": "extension@example.com",
      "strict_min_version": "128.0",
      "strict_max_version": "151.*"
    }
  }
}
```

- `id`: **Required for publishing.** Use email-style format (`extension@your-domain.com`) or UUID in curly braces. Cannot be changed once published.
- `strict_min_version`: Minimum Thunderbird version. Set to the oldest you've tested.
- `strict_max_version`: Maximum version. Use `"151.*"` format to cap at a branch. Important when using Experiment APIs since internal APIs may change. Omit for pure WebExtension add-ons.
