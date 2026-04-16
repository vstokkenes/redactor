# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in Redactor, please report it responsibly.

**Do not open a public issue.** Instead, email [vidar@stokken.es](mailto:vidar@stokken.es) with:

- Description of the vulnerability
- Steps to reproduce
- Potential impact

You should receive a response within 72 hours. Fixes for confirmed vulnerabilities will be released as soon as possible.

## Scope

This policy covers the `redactor.html` file and its client-side functionality. Since the tool runs entirely in the browser with no server component, the primary attack surface is:

- XSS or DOM injection via crafted input
- Weaknesses in the encryption implementation (AES-256-GCM / PBKDF2)
- ReDoS via user-supplied regex patterns
- Data leakage through clipboard, localStorage, or exports

## Design Principles

- **Zero network requests** — enforced by `Content-Security-Policy: default-src 'none'`
- **No external dependencies** — single HTML file, no CDN resources
- **Web Crypto API only** — no custom crypto implementations
