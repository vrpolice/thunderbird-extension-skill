# Supported WebExtension APIs — Complete Reference

## Thunderbird-Specific MailExtension APIs

These APIs are unique to Thunderbird. Use the `messenger.` namespace to access them.

| API | Required Permission / Key | Purpose |
|-----|--------------------------|---------|
| `accounts` | `accountsRead` | List/manage email accounts and identities |
| `addressBooks` | `addressBooks` | Read/write address books and contacts |
| `browserAction` | `browser_action` manifest key | Control the main toolbar button (MV2 only; use `action` in MV3) |
| `cloudFile` | `cloudFile` manifest key | Cloud file upload providers |
| `commands` | `commands` manifest key | Keyboard shortcuts |
| `compose` | `compose` | Open/manage compose windows, get/set compose details |
| `composeAction` | `compose_action` manifest key | Control the compose toolbar button |
| `composeScripts` | `compose` | Register scripts injected into compose windows |
| `contacts` | `addressBooks` | Address book contact operations |
| `folders` | `accountsFolders` | Browse/manage email folder hierarchy |
| `mailingLists` | `addressBooks` | Mailing list operations |
| `mailTabs` | `accountsFolders` (for setting displayed folder only) | Access main mail tab, set displayed folder |
| `menus` | `menus` (+ optional: `menus.overrideContext`, `accountsRead`, `messagesRead`, `activeTab`) | Add/modify context menus |
| `messageDisplay` | `messagesRead` | Access displayed message data |
| `messageDisplayAction` | `message_display_action` manifest key | Control the message display toolbar button |
| `messageDisplayScripts` | `messagesModify` | Register scripts injected into message display windows |
| `messages` | `messagesRead`, `messagesMove`, `accountsRead` | List, move, delete, tag messages |
| `tabs` | Varies: `tabs`, `activeTab`, `compose`, or `messageModify` | Extended tabs API supporting compose/message tabs |
| `theme` | `theme` manifest key | Theme management |
| `windows` | `tabs` | Extended windows API with `messageCompose` window type |

### Extended `tabs` API

Thunderbird's `tabs` API extends the standard Firefox version with:
- `windowType`: `"messageCompose"`, `"messageDisplay"`, `"mail"`, `"addressBook"`, `"calendar"`
- `messenger.tabs.sendMessage(tabId, message)` — works on compose and message display tabs
- `messenger.tabs.query({ windowType: "messageCompose" })` — find compose tabs

### Extended `windows` API

Thunderbird's `windows` API supports:
- `windowTypes`: `"messageCompose"`, `"messageDisplay"`, `"normal"`, `"calendar"`, `"addressBook"`
- `messenger.windows.getAll({ populate: true, windowTypes: ["messageCompose"] })` — list all compose windows with tab details

## Standard Firefox WebExtension APIs (Known to Work in Thunderbird)

These standard APIs are confirmed working in Thunderbird:

| API | Required Permission | Notes |
|-----|---------------------|-------|
| `browserSettings` | `browserSettings` | Browser-level settings |
| `clipboard` | `clipboardWrite` / `clipboardRead` | Clipboard access |
| `contentScripts` | Host permissions (URL patterns) | Web page content scripts only (not mail messages) |
| `cookies` | `cookies` + host permissions | Cookie access |
| `dns` | `dns` | DNS resolution |
| `downloads` | `downloads` (+ `downloads.open`) | File downloads |
| `extension` | None | Extension metadata (`getURL`, `getViews`, etc.) |
| `i18n` | None | Internationalization |
| `identity` | `identity` | OAuth identity flows |
| `idle` | `idle` | Idle state detection |
| `notifications` | `notifications` | System notifications |
| `permissions` | None | Runtime permission management |
| `pkcs11` | `pkcs11` | Cryptographic token management |
| `privacy` | `privacy` | Privacy settings |
| `proxy` | `proxy` + host permissions | Proxy configuration |
| `runtime` | None | Messaging, lifecycle, manifest access |
| `storage` | `storage` / `unlimitedStorage` | Local/sync storage |
| `management` | `management` | Access other installed add-ons |
| `userScripts` | None | Custom user scripts (web page tabs only) |
| `webNavigation` | `webNavigation` | Navigation events in web tabs |
| `webRequest` | `webRequest` (+ `webRequestBlocking`) | HTTP request interception |

## APIs NOT Supported in Thunderbird

The following standard Firefox APIs are notably absent and should not be relied upon:
- `alarms` — Not available in Thunderbird
- `bookmarks` — Not meaningful in a mail client
- `browsingData` — Not available
- `contextualIdentities` — Not available
- `devtools` — Not available
- `find` — Not available
- `geckoProfiler` — Not available
- `history` — Not available
- `pageAction` — Not available
- `sessions` — Not available
- `sidebarAction` — Not available
- `tabHide` — Not available
- `topSites` — Not available
- `search` — Not available
- `unifiedExtensions` — Not available

## Content Script API Restrictions

Scripts loaded via `composeScripts` or `messageDisplayScripts` can ONLY use:
- `runtime.connect()`, `runtime.getManifest()`, `runtime.getURL()`
- `runtime.onConnect`, `runtime.onMessage`, `runtime.sendMessage`
- `i18n.getMessage()`, `i18n.getAcceptLanguages()`, `i18n.getUILanguage()`, `i18n.detectLanguage()`
- `menus.getTargetElement()`
- `storage.*`

For anything else: communicate with the background script via `runtime.sendMessage`.

## CloudFile Management Script Restrictions

Scripts loaded from a CloudFile `management_url` can ONLY use:
- `cloudFile.*`
- `extension.*`
- `i18n.*`
- `runtime.*`
- `storage.*`

## `messenger.` vs `browser.` Namespace

Thunderbird provides both namespaces:
- `browser.*` → Standard WebExtension APIs (Firefox-compatible)
- `messenger.*` → Thunderbird-specific MailExtension APIs

Both namespaces can access common APIs like `runtime`, `storage`, `i18n`, `menus`, `windows`, `tabs`. However, for Thunderbird-specific features (accounts, folders, messages, compose), only `messenger.` works.
