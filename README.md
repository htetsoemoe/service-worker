# Service Worker Demo

This project is a simple multi-page website (`index.html`, `profile.html`, `contact.html`) that uses a Service Worker (`sw.js`) to cache files and improve offline behavior.

## Project Structure

- `app.js`: Registers the Service Worker in the browser.
- `sw.js`: Handles install, activate, and fetch events.
- `index.html`, `profile.html`, `contact.html`: Pages of the site.
- `images/`: Static images used by each page.

## How It Works

### 1. Service Worker Registration

In `app.js`, the browser checks for Service Worker support and registers `./sw.js`.

```js
const registration = await navigator.serviceWorker.register("./sw.js")
```

### 2. Install Event (Pre-cache)

In `sw.js`, the `install` event:

- Calls `self.skipWaiting()` to activate the new worker faster.
- Opens cache `v1`.
- Pre-caches app shell files and page assets:
  - `/`
  - `/app.js`
  - `/index.html`
  - `/contact.html`
  - `/profile.html`
  - `/images/contactus.png`
  - `/images/home.jpg`
  - `/images/profile.jpg`

### 3. Activate Event

On `activate`, the worker:

- Calls `clients.claim()` so existing tabs are controlled immediately.
- Enables Navigation Preload (if supported), allowing the browser to start network navigation requests while the worker boots.

### 4. Fetch Event Strategy

Every request is handled by `cacheMatch(request, preloadResponsePromise)`:

1. Return cached response if found.
2. Else use navigation preload response (if available), store it in cache, and return it.
3. Else fetch from network, cache it, and return it.
4. If all fail, return fallback text response: `"Response not found!"`.

This is effectively a **cache-first strategy with network fallback**, plus navigation preload optimization.

## Run Locally

Service Workers require HTTP/HTTPS (not plain `file://`).

Example with Node:

```bash
npx serve .
```

Then open the local URL shown in terminal.

## Notes

- Cache version is currently `v1`.
- When assets change, bump cache version (for example `v2`) and optionally remove old caches during `activate`.

## About `waitUntil`

`event.waitUntil(...)` tells the browser to keep the Service Worker event alive until the passed Promise finishes.

In this project:

- In `install`, `event.waitUntil(addResourcesToCache(...))` ensures pre-caching completes before install is treated as successful.
- In `activate`, `event.waitUntil(...)` is used for `clients.claim()` and enabling navigation preload so activation tasks finish reliably.

Why it matters:

- Without `waitUntil`, the browser can terminate the event early.
- If install work is interrupted, required files may not be cached.
- If activate work is interrupted, control/preload setup may be incomplete.
