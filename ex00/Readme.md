# Exercise 00 — Cross-Site Scripting (XSS)

## Objective
Identify and demonstrate an XSS vulnerability in the provided 42 Cybersecurity 0x0 local web application. The exercise specifically asks us to display the contents of a JavaScript-accessible cookie on the visible page.

## What is XSS?
Cross-Site Scripting (XSS) is a web vulnerability in which untrusted input is interpreted as executable code in a browser, instead of being displayed safely as text. An attacker may use this to alter page content or interact with information accessible to scripts in the affected origin.

## Why this application is vulnerable
The provided `pro_00/index.html` contains this code inside `displayText()`:

```javascript
var userInput = document.getElementById("inputText").value;
document.getElementById("output").innerHTML = "<b>" + userInput + "</b>";

var script = document.createElement("script");
script.textContent = userInput;
document.body.appendChild(script);
```

Two unsafe operations occur:
1. The application inserts untrusted input as HTML using `innerHTML`, enabling browser interpretation of injected elements and event handlers.
2. It creates a `<script>` element and assigns user-controlled text to `script.textContent`, making the input executable as JavaScript.

The application also sets a demonstration cookie using `document.cookie`. Because the cookie is written by JavaScript, it is accessible via `document.cookie` in the relevant document context.

## Vulnerability classification
This is a **DOM-based XSS** demonstration: client-side JavaScript reads a form value and writes it into dangerous browser sinks without safe handling. The supplied code does not show a server storing or reflecting the input, so we should not label this particular implementation Stored XSS or server-side Reflected XSS.

## Two possible exploitation scenarios

### Scenario 1 — Unauthorized content change
An attacker who can cause malicious input to be entered into the vulnerable form may execute JavaScript that changes the visible content of the page, such as displaying misleading instructions or a fake notification. This demonstrates loss of page integrity.

### Scenario 2 — Exposure of JavaScript-accessible cookies
If malicious JavaScript runs in the page's origin, it may read non-HttpOnly cookies using `document.cookie` and display their values. In an actual application, exposure could affect confidentiality, depending on what those cookies contain and whether additional protections exist. In this local exercise, the requested demonstration is to show the test cookie on the page. `HttpOnly` cookies cannot be read through `document.cookie`.

## Testing and observations
- The `<img>` element with an `onerror` JavaScript handler successfully triggered an XSS alert in the local exercise.
- An `iframe` using `srcdoc` successfully triggered an XSS alert.
- The user confirmed that the cookie-display payload using the original page's `output` element worked.
- A plain `<script>alert('XSS')</script>` injected as HTML did not cause a visible alert during the initial test; this does **not** mean the application is secure.
- The SVG test was not confirmed successful.

The full payload descriptions are in `Payloads.md`. Defensive changes are proposed in `Fix.md`.

## References
- OWASP Cross Site Scripting Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP DOM based XSS Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html
- MDN `Element.innerHTML`: https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML
