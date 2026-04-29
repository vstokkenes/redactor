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

### Term types

Terms are added in the term input field. The type is detected automatically from a prefix (e.g. `person:`, `project:`) or from the value itself (e.g. a bare `192.168.0.0/24` becomes a CIDR). The prefix is stripped before storage.

You can add several terms at once by separating them with commas — commas inside `{}` and `[]` are preserved (regex-safe).

#### Generic

| Prefix | Example | Notes |
|--------|---------|-------|
| *(none)* | `secretword` | Plain text match |
| `regex:` | `regex:\b\d{11}\b` | Custom pattern; validated against catastrophic backtracking |
| `key:` | `key:password` | Matches the value after `=`, `:` or `=>` delimiters |

#### People

| Prefix | Example | Notes |
|--------|---------|-------|
| `person:` | `person:John Smith` | Full name |
| `person-first:` | `person-first:John` | First name only |
| `person-last:` | `person-last:Smith` | Last name only |

#### Organizations & projects

| Prefix | Example | Notes |
|--------|---------|-------|
| `org:` | `org:Acme Corp` | External company / organization |
| `corp:` | `corp:Platform Team` | Internal team or department name |
| `project:` | `project:Bluebird` | Project name or codename |

#### Network

| Prefix | Example | Notes |
|--------|---------|-------|
| `ip:` | `ip:192.168.1.1` | IPv4 (also auto-detected from bare values) |
| `ipv6:` | `ipv6:2001:db8::1` | IPv6 (auto-detect optional in Settings) |
| `cidr:` | `cidr:10.0.0.0/8` | Matches all IPs in range; auto-detected from bare values |
| `mac:` | `mac:aa:bb:cc:dd:ee:ff` | MAC address (auto-detect optional in Settings) |
| `fqdn:` | `fqdn:api.internal.corp` | Fully qualified domain name (auto-detect optional) |
| `domain:` | `domain:internal.corp` | Domain name |
| `server:` | `server:db01` | Server / hostname |
| `url:` | `url:https://example.com/x` | Full HTTP/HTTPS URL; validated via URL API |
| `email:` | `email:user@example.com` | Email (auto-detect optional in Settings) |

#### Identifiers

| Prefix | Example | Notes |
|--------|---------|-------|
| `phone:` | `phone:+47 12345678` | Phone number |
| `account:` | `account:ACC-12345` | Account identifier |
| `iban:` | `iban:NO9386011117947` | Bank account; mod-97 checksum validated |
| `national-id:` | `national-id:1234567` | Generic national ID (non-Norwegian) |
| `dob:` | `dob:1985-04-12` | Date of birth — accepts `YYYY-MM-DD` or `DD.MM.YYYY` |
| `hash:` | `hash:5d41402a...` | Hex digest; validates length (MD5/SHA-1/256/512 etc.) |
| `location:` | `location:Oslo` | Place name |

### Auto-detection

Toggle in Settings to automatically find and redact common patterns without listing each value as a term:

- IPv4 addresses (on by default)
- Email addresses
- Hostnames / FQDNs
- UUIDs
- IPv6 addresses
- MAC addresses

### Built-in presets

Toggle in Settings:

- Norwegian SSN (11-digit personal numbers)
- Phone numbers (Norwegian format)
- API keys (common prefixes like `sk_`, `pk_`, `api_`, `token_`, `secret_`)

### Pseudonymization

Optional mode (toggle in Settings) — instead of replacing values with `<REDACTED-NNN>` tokens, redactor substitutes realistic fake values:

- `person:` → fake names from embedded dictionaries
- `ip:` → RFC 5737 TEST-NET addresses (`198.51.100.x`, `203.0.113.x`)
- `ipv6:` → RFC 3849 documentation range (`2001:db8::…`)
- `mac:` → locally administered, unicast MAC
- `email:` → fake address using RFC 2606 domain
- `fqdn:`/`server:` → fake hostname + fake domain
- `iban:` → fake Norwegian IBAN
- `dob:` → fake birth date
- `hash:` → random hex of the same length as the original
- `org:`/`corp:`/`project:`/`location:`/`phone:`/`account:` → values from embedded dictionaries

Fake values are deterministic per token within a session — the same token always renders the same fake value, so the output stays internally consistent. The mapping is stored in the session, so you can still restore the original text afterwards.

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
- **Optional encrypted storage** — AES-256-GCM encryption for localStorage data (off by default — see Limitations)
- **Secure password input** — all password prompts use modal dialogs with masked input (no browser `prompt()`)
- **Brute-force protection** — exponential backoff on failed unlock attempts (lockout after 5 failures)
- **ReDoS protection** — user-supplied regex patterns are validated with structural analysis and time-based checks to prevent catastrophic backtracking
- **Single HTML file** — easy to audit, no build step, no dependencies

> **Client-side is not isolated.** Browser extensions, devtools, screen sharing, clipboard managers, and browser sync can still observe input and output while the tab is open. Treat the browser environment itself as part of the trust boundary.

## Limitations

### Context can re-identify what redaction tokens hide

Redactor replaces *identifiers*, not *context*. Even when every name, IP, and email is correctly tokenized, the surrounding text can still reveal what was redacted:

- A redacted person's name next to their unique job title, employer, and city is often still identifiable
- A redacted IP address surrounded by ASN / routing details narrows back down to a small set
- A redacted customer name accompanied by an unmistakable description of their product or contract terms is effectively un-redacted

The tool also cannot detect what it does not know about: encoded values (Base64, URL-encoded), Unicode homoglyphs that survive NFKC normalization, deeply nested structures, or sensitive information you have not added as a term and which is not covered by auto-detection or built-in presets.

### Session exports contain the originals in plaintext

Session JSON exports include the full token-to-original mapping. Sharing a session export with a colleague to let them de-redact the text is equivalent to sharing the underlying sensitive data — handle the file with the same care as the raw input.

### Pseudonymized values look real

In pseudonymization mode the substitutes (RFC 5737 IPs, fake IBANs, fake names, fake hostnames) are designed to be valid-looking — they fit each type's syntax so the redacted text remains usable. They are not real anonymization: the session still maps them back to the originals, and a recipient unaware of the mode may treat the fake values as truthful data. Always tell recipients when output is pseudonymized.

### localStorage is unencrypted by default

Terms, sessions, and settings persist in browser localStorage across visits. Encryption is opt-in (toggle in Settings — AES-256-GCM with PBKDF2 key derivation). On a shared device, anyone with access to the same browser profile can read every term and session unless encryption is enabled or you clear the data when finished.

### Always review the redacted output before sharing

Redactor is a convenience aid, not a guarantee of anonymity, and should not be the sole safeguard for compliance-critical workflows.

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
