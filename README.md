# Redactor

A client-side text redaction tool that runs entirely in your browser. No data ever leaves your machine.

## What is this?

Redactor replaces sensitive information in text with consistent, reversible tokens. Paste in logs, configs, support tickets, or any text containing personal data — and get back a safely shareable version where names, IPs, emails, and other identifiers are replaced with tokens like `REDACTED-001`.

## Use cases

- **Support tickets** — Share customer cases with third parties without exposing personal data
- **Log analysis** — Redact server logs before posting in Slack, Jira, or forums
- **Configuration review** — Redact credentials and internal hostnames from config files before sharing
- **Incident reports** — Strip PII from post-mortems and incident timelines
- **Development & testing** — Create redacted datasets from production data
- **Compliance** — Help meet GDPR/privacy requirements when sharing text across teams or organizations
- **Consulting** — Safely share client infrastructure details in proposals or documentation
- **Accessibility** — People with reading or writing difficulties (e.g. dyslexia) can redact sensitive details before using tools like Google Translate, AI assistants, or spell-checkers — getting the language help they need without exposing personal data to third-party services

## Features

### Data types

| Type | Example input | How to add |
|------|--------------|------------|
| Plain text | `John Smith` | Type directly |
| IPv4 address | `192.168.1.1` | Type or auto-detect |
| CIDR range | `10.0.0.0/8` | Matches all IPs in range |
| Email | `user@example.com` | Type or auto-detect |
| FQDN | `api.internal.corp` | Type or auto-detect |
| UUID | `550e8400-e29b-...` | Type or auto-detect |
| Key:value | `key:password` | Prefix with `key:` — matches values after `=`, `:`, `=>` |
| Regex | `regex:\b\d{11}\b` | Prefix with `regex:` for custom patterns |

### Auto-detection

Toggle in settings to automatically find and redact:
- IPv4 addresses (on by default)
- Email addresses
- Hostnames / FQDNs
- UUIDs

### Built-in presets

- Norwegian SSN (11-digit personal numbers)
- Phone numbers (Norwegian format)
- API keys (common prefixes like `sk_`, `api_`, `token_`)

### Sessions

Each session maintains a consistent mapping between original values and tokens. The same input always produces the same token within a session.

- Create multiple named sessions (per case, per client, etc.)
- View the full token-to-original mapping table
- Export/import sessions as JSON to share with colleagues for restoration
- Statistics showing match counts per term

### Export & import

- **Terms** — Export your term list as a password-encrypted `.enc` file. A password is always required — exported files may end up in email, shared drives, or chat, so they are never stored in plain text.
- **Sessions** — Export session mappings for colleagues who need to restore the original text
- **Encrypted export** — AES-256-GCM encryption with PBKDF2 key derivation (600,000 iterations)
- Import auto-detects encrypted vs. plain files and prompts for the password when needed

### Other

- Undo / redo
- Swap input and output
- Drag-and-drop file loading
- Keyboard shortcuts (press `?` to see all)
- Term grouping and search/filter (filter by type: `regex`, `cidr`, `ip`, etc.)
- Add multiple comma-separated terms at once (regex-safe: commas inside `{1,2}` or `[a,b]` are preserved)
- Customizable token prefix
- Dark and light theme

## Security

- **100% client-side** — all processing happens in the browser
- **Zero network requests** — no external scripts, fonts, APIs, or tracking
- **Strict CSP** — `default-src 'none'` Content Security Policy
- **Mandatory encrypted export** — term exports are always password-protected (AES-256-GCM), since exported files may be shared via email or other channels
- **Optional encrypted storage** — AES-256-GCM encryption for localStorage data
- **Secure password input** — all password prompts use modal dialogs with masked input (no browser `prompt()`)
- **Brute-force protection** — exponential backoff on failed unlock attempts (lockout after 5 failures)
- **ReDoS protection** — user-supplied regex patterns are validated with structural analysis and time-based checks to prevent catastrophic backtracking
- **Single HTML file** — easy to audit, no build step, no dependencies

## Languages

English, Norwegian Bokmål, Swedish, and Danish — with full UI translations.

## Getting started

1. Open `redactor.html` in any modern browser
2. Add terms you want to redact (or enable auto-detection in settings)
3. Paste text in the input area
4. Press **Redact** (or `Ctrl+Enter`)
5. Copy the redacted output

That's it. No installation, no server, no account.

## License

MIT
