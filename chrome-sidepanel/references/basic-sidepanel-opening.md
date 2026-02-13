# Basic Sidepanel Opening

## Opening the Side Panel

```typescript
// Bug: If you await setOptions(), the user gesture context is lost
// and open() throws "may only be called in response to a user gesture"
// Solution: Don't await setOptions()

// 1. Configure side panel for specific tab (no await!)
chrome.sidePanel.setOptions({
  tabId: tab.id,
  path: "src/sidepanel/sidepanel.html?tabId=${tab.id}",
  enabled: true,
});

// 2. Open the panel
await chrome.sidePanel.open({ tabId: tab.id });
```

## Key Patterns

- **Pass tabId via URL query param**: Use `?tabId=${tab.id}` to identify which tab the panel belongs to
- **Configure per-tab**: Call `setOptions()` before `open()` to set tab-specific path
- **Enable required**: Must set `enabled: true` before opening

## Manifest Requirements

```json
{
  "permissions": ["sidePanel"],
  "side_panel": {
    "default_path": "src/sidepanel/sidepanel.html"
  }
}
```

Use `chrome.action` without `default_popup` for direct icon click handling.

## Event Listener

In background script (service worker):

```typescript
chrome.action.onClicked.addListener(async (tab) => {
  if (tab.id) {
    chrome.sidePanel.setOptions({
      tabId: tab.id,
      path: "src/sidepanel/sidepanel.html?tabId=${tab.id}",
      enabled: true,
    });
    await chrome.sidePanel.open({ tabId: tab.id });
  }
});
```

## Retrieving tabId in Side Panel

In side panel HTML/JS:

```typescript
const tabId = parseInt(new URL(document.URL).searchParams.get("tabId") || "0");
```

## Reference

- https://developer.chrome.com/docs/extensions/reference/api/sidePanel/
- https://github.com/GoogleChrome/chrome-extensions-samples/issues/1477
