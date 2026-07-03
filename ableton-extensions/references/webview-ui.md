# Webview Modal Dialogs

Custom UI is an HTML page shown in a modal webview. While it's open the extension is suspended and Live's UI is blocked. Use it for input/forms; use a **progress dialog** (patterns reference) for background work with feedback.

## API

```ts
showModalDialog(url: string, width: number, height: number): Promise<string>
```
Supported URL schemes: `file:`, `data:`, `https:`, `http://localhost`. The Promise resolves with the string the page sends back (see the bridge).

The common approach inlines the HTML as a `data:` URL so nothing needs hosting:

```ts
import modalInterface from "./interface.html";   // esbuild inlines this as a string

const result = await context.ui.showModalDialog(
  `data:text/html,${encodeURIComponent(modalInterface)}`, 360, 240);
const { name } = JSON.parse(result);
```

## esbuild HTML inlining

Two pieces make `import html from "./interface.html"` typecheck and bundle:

1. build.ts loader: `loader: { ".html": "text" }` (see project-setup).
2. A type shim, `src/html.d.ts`:
```ts
declare module "*.html" {
  const content: string;
  export default content;
}
```

## The host message bridge

The webview runs in its own environment. To return a value and close, the page posts a message named `close_and_send` whose `params` is an array containing a **single string** (conventionally stringified JSON). Feature-detect the platform: WebKit (macOS) vs WebView2 (Windows).

```html
<script>
  const isWebKit = window.webkit?.messageHandlers?.live;
  const isWebView2 = window.chrome?.webview;

  function sendMessage(message) {
    if (isWebKit) window.webkit.messageHandlers.live.postMessage(message);
    else if (isWebView2) window.chrome.webview.postMessage(message);
  }

  function closeWithResult(result) {
    sendMessage({ method: "close_and_send", params: [JSON.stringify(result)] });
  }
</script>
```

`showModalDialog` resolves with that single string. Keep both platform branches — an extension may run on either OS.

## Passing data *into* the webview

There's no live channel; bake data into the HTML before encoding, or append query params to the data URL and read them in the page:

```ts
const html = template.replace("__INITIAL__", JSON.stringify(initialData));
await context.ui.showModalDialog(`data:text/html,${encodeURIComponent(html)}`, 360, 240);
```

## Native look — Live dark theme

Match Live's chrome. The example dialog defines these custom properties (HSL source values fed through `oklch(from …)`), uses the `AbletonSansSmall` font at `~11.5px`, and classes `.alx-input` / `.alx-button` / `.alx-label`:

```css
:root {
  --p-live-ui-bg: hsl(0, 0%, 21%);
  --p-live-control-bg: hsl(0, 0%, 16%);
  --p-live-input-bg: hsl(0, 0%, 12%);
  --p-live-text-primary: hsl(0, 0%, 71%);
  --p-live-text-secondary: hsl(0, 0%, 41%);
  --p-live-control-border: hsl(0, 0%, 7%);
  --p-live-accent-primary: hsl(31, 100%, 67%);   /* Live's signature orange/yellow */
}
html {
  background: hsl(0,0%,21%);
  color: hsl(0,0%,71%);
  font-family: "AbletonSansSmall", sans-serif;
  font-size: 11.5px; font-weight: 500;
  -webkit-font-smoothing: antialiased;
}
```

Handy affordances from the example: focus the primary input on `DOMContentLoaded`; bind `Enter` to submit and `Escape` to cancel (send a null/empty result).

## Design-guide rules

- Visual hierarchy should reflect sonic importance; group related parameters; use whitespace.
- Reflect processing order top-to-bottom / left-to-right.
- **Disable** controls (`:disabled`) rather than hiding them.
- Button labels are **verbs**; keep a consistent "Cancel" label.
- Radio group for ≤5 options; a `select` for more/longer labels; toggles/checkboxes for on/off (not buttons).
- Live's signature yellow/blue signal interactivity; aim for WCAG contrast.
