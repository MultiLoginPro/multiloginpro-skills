# MultiLoginPro Local API Reference

## Base URL

```
http://192.168.0.123:39633/api
```

All API calls are **HTTP POST** requests with a JSON body. Responses follow the structure:

```json
{
  "success": true,
  "result": <object|array|string|null>,
  "error": null,
  "browserNotFound": false
}
```

## Common Payload Pieces

### `searchElement`

| Field | Type | Description |
| --- | --- | --- |
| `parent` | string/null | Optional selector that scopes the search to a parent node |
| `selector` | string | CSS selector or XPath expression |
| `isXPath` | bool | Set to `true` when `selector` is XPath |
| `index` | int | 0-based match index (`-1` = all matches) |
| `searchAllIframe` | bool | Search every iframe automatically |
| `selectorForSearchIframe` | string/null | Selector for a specific iframe before searching |
 
## Endpoint Catalog

| Section | Endpoint | Purpose |
| --- | --- | --- |
| Profile | `/profile/list` | List profiles by status (`type`: 0 all, 1 opened, 2 unopened) |
| Profile | `/profile/start` | Launch a profile and return debugger ports |
| Profile | `/profile/close` | Close by `profileId` or `browserId` |
| Profile | `/profile/create` | Create a profile (name, group, proxy, OS, browser version) |
| Browser | `/browser/loadurl` | Navigate with optional new tab & timeout |
| Browser | `/browser/switchtab` | Activate a tab by URL or jump to last tab |
| Browser | `/browser/closetab` | Close a matched tab or close others/last |
| Browser | `/browser/setcookie` | Inject cookies (stringified JSON array) |
| Browser | `/browser/getcookie` | Read cookies for a specific domain |
| Browser | `/browser/getallcookies` | Dump the entire cookie jar |
| Browser | `/browser/clearcookies` | Remove all cookies |
| Browser | `/browser/getimage` | Screenshot full page or element |
| Browser | `/browser/executejs` | Run custom JavaScript (optionally inside iframe) |
| Browser | `/browser/findelement` | Locate and optionally cache an element |
| Browser | `/browser/pageloadwait` | Retry until an element appears |
| Browser | `/browser/scrapespecialtext` | Get page source/title/body/current URL |
| Browser | `/browser/scrapeonetext` | Read a single property/value |
| Browser | `/browser/scrapesometext` | Collect property values from all matches |
| Browser | `/browser/setvalue` | Type into inputs / textareas |
| Browser | `/browser/elementoperate` | Left/right/double-click |
| Browser | `/browser/keystrokesemulation` | Send keystrokes or plain text |
| Browser | `/browser/humansimulator` | Simulate real human mouse movement |

---

## Profile Endpoints

### `/profile/list`

| Name | Type | Description |
| --- | --- | --- |
| `type` | int | `0` = all profiles, `1` = opened, `2` = unopened |

**Body**
```json
{
  "type": 1
}
```

**Response (abridged)**
```json
{
  "success": true,
  "result": [
    {
      "id": "1622109562686",
      "name": "cloud_test_3",
      "group": "TEST",
      "startUrl": "https://www.baidu.com/",
      "browserId": 1956,
      "currentUrl": "https://www.baidu.com/",
      "account": null,
      "browser": {
        "browserId": 1956,
        "httpDebugAddress": "127.0.0.1:44076",
        "wsDebugAddress": "ws://127.0.0.1:44076/devtools/browser/..."
      }
    }
  ]
}
```

### `/profile/start`

| Name | Type | Description |
| --- | --- | --- |
| `profileId` | string | Target profile ID |

**Body**
```json
{
  "profileId": "1622109562686"
}
```

**Response**
```json
{
  "success": true,
  "result": {
    "browserId": 22980,
    "httpDebugAddress": "127.0.0.1:63104",
    "wsDebugAddress": "ws://127.0.0.1:63104/devtools/browser/..."
  }
}
```

### `/profile/close`

| Name | Type | Description |
| --- | --- | --- |
| `profileId` | string | Profile ID to close (optional when closing by browserId) |
| `browserId` | int | Browser instance ID |

**Body**
```json
{
  "profileId": "1622109562686",
  "browserId": 0
}
```
_or_
```json
{
  "profileId": null,
  "browserId": 22980
}
```

**Response**: success boolean payload.

### `/profile/create`

| Name | Type | Description |
| --- | --- | --- |
| `name` | string | Profile name |
| `group` | string | Group/folder |
| `proxy` | string | Proxy definition (e.g. `[http]127.0.0.1:4567`) |
| `systemOS` | int | `0` Windows, `1` macOS, `2` Linux, `3` iOS, `4` Android |
| `browserVersion` | int | `1` V127, `2` V138, `3` V139, `4` V142 |

**Body**
```json
{
  "name": "test_profile",
  "group": "TEST",
  "proxy": "[http]127.0.0.1:4567",
  "systemOS": 0,
  "browserVersion": 4
}
```

**Response**: standard success payload.

---

## Browser Navigation

### `/browser/loadurl`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser instance ID |
| `url` | string | Target URL |
| `newTab` | bool | `true` = open in new tab |
| `timeout` | int | Seconds to wait for navigation |

**Body**
```json
{
  "browserId": 22980,
  "url": "https://www.qq.com/",
  "newTab": true,
  "timeout": 30
}
```

**Response**: `"result": null` on success.

### `/browser/switchtab`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `url` | string | Substring to match the first tab |
| `lastTab` | bool | Ignore `url` and jump to last tab when `true` |

**Body**
```json
{
  "browserId": 22980,
  "url": "www.baidu.com",
  "lastTab": false
}
```

### `/browser/closetab`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `url` | string | Tab URL substring to close |
| `closeOthers` | bool | Close every other tab |
| `closeLastTab` | bool | Close the last tab directly |

**Body**
```json
{
  "browserId": 22980,
  "url": "www.baidu.com",
  "closeOthers": false,
  "closeLastTab": false
}
```

---

## Cookie Management

### `/browser/setcookie`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `cookie` | string | Stringified JSON array (Chromium DevTools cookie schema) |

**Body**
```json
{
  "browserId": 22980,
  "cookie": "[{\"domain\":\".baidu.com\",\"name\":\"BIDUPSID\",...}]"
}
```

### `/browser/getcookie`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `url` | string | Domain to filter |

**Body**
```json
{
  "browserId": 22980,
  "url": "https://www.baidu.com/"
}
```

**Result**: `result` is a JSON string containing the cookie array for that domain.

### `/browser/getallcookies`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |

**Body**
```json
{ "browserId": 22980 }
```

**Result**: JSON string array with every cookie stored in the profile.

### `/browser/clearcookies`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |

**Body**
```json
{ "browserId": 22980 }
```

---

## Visuals & JS

### `/browser/getimage`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `filePath` | string | Absolute path for the PNG file |
| `elementScreenshot` | bool | `true` = crop to element |
| `searchElement` | object | Retrieve the configuration information of the element when `elementScreenshot` is true (see `searchElement` table) |

**Body**
```json
{
  "browserId": 22980,
  "filePath": "C:\\Users\\demo\\Desktop\\shot.png",
  "elementScreenshot": true,
  "searchElement": {
    "selector": "*[id='lg']",
    "isXPath": false,
    "index": 0,
    "searchAllIframe": false
  }
}
```

### `/browser/executejs`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `js` | string | JavaScript snippet |
| `inIframe` | bool | Execute inside iframe |

**Body**
```json
{
  "browserId": 22980,
  "js": "alert('hello world')",
  "inIframe": false
}
```

---

## Element Search & Interaction

### `/browser/findelement`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `searchElement` | object | Retrieve the configuration information of the element (see `searchElement` table) |

**Body**
```json
{
  "browserId": 22980,
  "searchElement": {
    "selector": "span.soutu-btn",
    "isXPath": false,
    "index": 0,
    "searchAllIframe": false
  }
}
```

### `/browser/pageloadwait`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `searchElement` | object | Retrieve the configuration information of the element (see `searchElement` table) |
| `loop` | int | Maximum attempts |
| `sleep` | int | Seconds to wait between attempts |

**Body**
```json
{
  "browserId": 22980,
  "searchElement": {
    "selector": "input[name='q']",
    "isXPath": false,
    "index": 0,
    "searchAllIframe": false
  },
  "loop": 10,
  "sleep": 1
}
```

**Response**
```json
{
  "success": true,
  "result": true,
  "error": null,
  "browserNotFound": false
}
```

### `/browser/setvalue`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `searchElement` | object | Retrieve the configuration information of the element (see `searchElement` table) |
| `property` | string | Property to extract (e.g., `innerText`, `value`, `href`) |
| `value` | string | Text to inject |

**Body**
```json
{
  "browserId": 22980,
  "searchElement": {
    "selector": "input[name='q']",
    "isXPath": false,
    "index": 0,
    "searchAllIframe": false
  },
  "value": "hello world",
  "property": "value"
}
```

**Response**: standard success payload.
 

## Element Scraping

### `/browser/scrapeonetext`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `searchElement` | object | Retrieve the configuration information of the element (see `searchElement` table) |
| `property` | string | Property to extract (e.g., `innerText`, `value`, `href`) |

**Body**
```json
{
  "browserId": 22980,
  "searchElement": {
    "selector": "h1",
    "isXPath": false,
    "index": 0,
    "searchAllIframe": false
  },
  "property": "innerText"
}
```

**Response**: `result` contains the requested string value.

### `/browser/scrapesometext`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `searchElement` | object | Retrieve the configuration information of the elements (see `searchElement` table) |
| `property` | string | Property to capture |

**Body**
```json
{
  "browserId": 22980,
  "searchElement": {
    "selector": "div.result h3",
    "isXPath": false,
    "index": -1,
    "searchAllIframe": false
  },
  "property": "innerText"
}
```

**Response**
```json
{
  "success": true,
  "result": ["Title 1", "Title 2"],
  "error": null,
  "browserNotFound": false
}
```

### `/browser/scrapespecialtext`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `specialText` | int | `0` source, `1` title, `2` body, `3` current URL |

**Body**
```json
{
  "browserId": 22980,
  "specialText": 3
}
```

**Response**: `result` returns the requested string (e.g., current URL when `specialText` = 3).

---

## Page Utilities
 
### `/browser/keystrokesemulation`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `text` | string | Plain text to type |
| `isSpecialKey` | bool | Input specialkey(true/false） |
| `specialKey` | int | 0 - TAB,1 - ENTER,2 - UP,3 - DOWN,4 - LEFT,5 - RIGHT,6 - HOME,7 - END,8 - PAGE UP,9 - PAGE DOWN,10 - BACKSPACE,11 - DELETE,12 - ESC |
| `sendTimes` | int | Send times for specialKey |
| `minInterval` | int | Minimum interval time after inputting each character (Milliseconds) |
| `maxInterval` | int | Maximum interval time after inputting each character (Milliseconds) |

**Body**
```json
{
  "browserId": 22980,
  "key": "Enter",
  "text": null
}
```

**Response**: success payload (result is `null`).

### `/browser/elementoperate`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `searchElement` | object | Retrieve the configuration information of the element (see `searchElement` table) |
| `operateType` | int | 0 - Event，eventName,1 - Click,2 - DoubleClick,3 - MouseClick,4 - MouseRightClick,5 - MouseDoubleClick,6 - MouseMove,7 - Touch,8 - ScrollTop,9 - ScrollBottom |
| `moveTime` | int | The duration of movement when operateType is MouseMove (Seconds) |

**Body**
```json
{
  "browserId": 22980,
  "key": "Enter",
  "text": null
}
```

**Response**: success payload (result is `null`).

### `/browser/humansimulator`

| Name | Type | Description |
| --- | --- | --- |
| `browserId` | int | Browser ID |
| `selector` | string | CSS selector for finding the element area where the simulated real-user movement actions will be performed |
| `duration` | int | Duration of simulated movement (Seconds) |

**Body**
```json
{
    "browserid": 31728,
    "selector": "#lg",
    "duration": 15
}
```

**Response**: success payload (result is `null`).

---

## Example Automation Flow

1. **List profiles** with `/profile/list` and copy the desired `profileId`.
2. **Start the profile** using `/profile/start` and capture the returned `browserId`.
3. **Load a URL** via `/browser/loadurl` (set `newTab` and `timeout` as needed).
4. **Wait for DOM ready** with `/browser/pageloadwait`, then locate elements via `/browser/findelement`.
5. **Interact** using `/browser/setvalue`, `/browser/clickelement`, `/browser/inputkeyboard`.
6. **Scrape state** using `/browser/scrapeonetext` or `/browser/scrapesometext`, or export cookies with `/browser/getallcookies`.
7. **Capture evidence** via `/browser/getimage` or log page source with `/browser/scrapespecialtext`.
8. **Close the session** using `/profile/close` when automation is complete.

Keep this reference alongside `SKILL.md` for workflow guidance.
