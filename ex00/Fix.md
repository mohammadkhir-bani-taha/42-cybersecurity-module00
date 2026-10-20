# Exercise 00 — How to Prevent XSS

## Root cause
The application treats user input as HTML by assigning it to `innerHTML`, and also creates a script element from the same untrusted input. Both behaviors allow an attacker-controlled value to become executable code.

## Fix 1 — Use `textContent` for plain text

**Vulnerable:**
```javascript
document.getElementById("output").innerHTML = "<b>" + userInput + "</b>";
```

**Safer:**
```javascript
const output = document.getElementById("output");
output.textContent = userInput;
```

`textContent` inserts the value as text instead of parsing it as HTML.

## Fix 2 — Never turn user input into JavaScript

**Remove this vulnerable block entirely:**
```javascript
var script = document.createElement("script");
script.textContent = userInput;
document.body.appendChild(script);
```

User-provided text should not be evaluated or appended as executable code.

## Fix 3 — Safe implementation of the form handler

Replace the current `displayText` function with:

```javascript
function displayText() {
    const userInput = document.getElementById("inputText").value;
    const output = document.getElementById("output");

    const bold = document.createElement("b");
    bold.textContent = userInput;
    output.replaceChildren(bold);
}
```

This preserves the intended bold formatting while treating the input as text.

## Fix 4 — Defense in depth

- Apply context-appropriate output encoding whenever displaying untrusted data.
- If rich HTML is genuinely required, use a maintained, well-configured HTML sanitizer rather than directly trusting user input.
- Use a restrictive Content Security Policy (CSP) to reduce the impact of script injection. CSP is an additional defense and does not replace safe DOM operations.
- In real applications, set session cookies with `HttpOnly`, `Secure` (over HTTPS), and an appropriate `SameSite` attribute. These flags do not remove an XSS vulnerability, but can reduce some consequences. JavaScript cannot create an `HttpOnly` cookie; it must be set by the server.
- Avoid dangerous APIs and sinks such as `eval`, unsafe `innerHTML`, and dynamic script creation from user data.

## How to verify the fix
1. Run the local application with the safe handler.
2. Submit `<img src=x onerror=alert('XSS')>`.
3. Confirm that the payload is displayed literally as text, with **no** popup.
4. Check that the form still displays normal text correctly.

## References
- OWASP Cross Site Scripting Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP DOM based XSS Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html
- MDN Content Security Policy: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP
