# Open Remote URL in Sidepanel

## Chrome Version Requirement

Chrome 140.0.7289.0+ introduces support for opening remote URLs in the side panel.

```json
{
  "minimum_chrome_version": "140.0.7289.0"
}
```

## Two-Step Process

Starting with Chrome 140.0.7289.0, you can open **remote URLs** in the side panel using a two-step process:

### Step 1: Open local sidepanel with URL params

```typescript
// utils.ts - Create path with URL params
export const SIDE_PANEL_PATH = chrome.runtime.getURL('src/sidepanel/sidepanel.html');

export function createSidePanelPath(tabId: number, url: string) {
  const sidePanelURL = new URL(SIDE_PANEL_PATH);
  sidePanelURL.searchParams.set('tabId', tabId.toString());
  sidePanelURL.searchParams.set('url', url.toString());
  return sidePanelURL.toString();
}

export async function openSidePanelURL(tabId: number, url: string) {
  const path = createSidePanelPath(tabId, url);
  // Don't await setOptions() - keep user gesture for open()
  chrome.sidePanel.setOptions({
    path,
    enabled: true,
    tabId
  });
  return chrome.sidePanel.open({ tabId });
}
```

### Step 2: Redirect to remote URL from within sidepanel

In your sidepanel HTML/JS (after loading the local sidepanel):

```typescript
const winURL = new URL(location.href);
const tabId = parseInt(winURL.searchParams.get('tabId') || '0');
const url = winURL.searchParams.get('url');

function redirect() {
  if (!tabId || !url) return;
  // Set the remote URL as the sidepanel path
  chrome.sidePanel.setOptions({ tabId, enabled: true, path: url });
}

// Call redirect() when appropriate
redirect();
```

The sidepanel will navigate to the remote URL and display it in the side panel.

## Manifest Requirements

```json
{
  "permissions": ["sidePanel"],
  "minimum_chrome_version": "140.0.7289.0"
}
```

Key points:
- **`sidePanel` permission is required**
- **`side_panel` key is NOT needed** - configure sidepanel programmatically
- **`minimum_chrome_version`**: Set to `'140.0.7289.0'` for remote URL support

## vite.config.ts Setup

Add sidepanel.html to build inputs:

```typescript
import { resolve } from 'path';
import { crx } from '@crxjs/vite-plugin';
import manifest from './src/manifest.config';

export default defineConfig(({ command }) => ({
  build: {
    rollupOptions: {
      input: {
        sidepanel: resolve(__dirname, './src/sidepanel/sidepanel.html')
      }
    }
  },
  plugins: [crx({ manifest })]
}));
```

## Reference

- https://developer.chrome.com/docs/extensions/reference/api/sidePanel/
