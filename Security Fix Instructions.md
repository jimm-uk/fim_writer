# FIM Writer — Security Fix Instructions

## Context
`fim_writer.html` is a single-file browser app for AI-assisted markdown writing. Users supply their own API key, base URL, and model name. The app will be served over HTTPS via Cloudflare. The goal is to mitigate security risks without adding accounts, server-side storage, or a proxy.

## Changes required

### 1. Fix XSS in the markdown preview renderer
In the `inlineMd()` function, the link regex passes `href` values through unvalidated. Sanitise it so only `http://`, `https://`, and `mailto:` schemes are allowed. Any other scheme should be replaced with `#`.

### 2. Switch to browser password manager for credential storage
Replace the current `sessionStorage`/`localStorage` approach for sensitive fields with proper `autocomplete` attributes so the browser's built-in password manager handles storage securely.

- Wrap the toolbar inputs in a `<form>` element with `autocomplete="on"`. Ensure the form does not submit (prevent default or use `type="button"` for all buttons).
- Set `autocomplete="url"` on the base URL input (`#t-url`).
- Set `autocomplete="current-password"` on the API key input (`#t-key`). The browser will treat it as a password field and offer to save it in the OS keychain.
- Set `autocomplete="off"` on the model name input (`#t-model`) — this is not sensitive and can remain in `localStorage` as-is.
- Remove the `sessionStorage.setItem` and `sessionStorage.getItem` calls for the API key entirely. The key should only ever be read from `keyEl.value` at request time, which the code already does.
- Remove the `localStorage` persistence of the base URL (`fw_url`). Do not pre-populate the field on load; let the browser autocomplete handle it.

### 3. Add a key-handling notice
Add a small, unobtrusive notice visible in the UI (e.g. below the toolbar or as a `title` attribute on the key field) stating: "Your API key is never stored by this app. Use your browser's password manager to save it securely."

## What not to change
- Do not add a server-side proxy.
- Do not add user accounts or any server-side storage.
- Do not change the core completion, summarisation, or formatting logic.
- Keep the app as a single self-contained HTML file.

