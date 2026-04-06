# Security Review — xbookmark-cli

**Date:** 2026-04-06
**Reviewer:** Automated security audit (Claude)
**Verdict:** SAFE — No credential theft or security breaches detected

## Scope

Full codebase review covering:
- Chrome cookie extraction and handling
- All network communication endpoints
- Data storage and file operations
- Dependencies and build scripts
- Dynamic code execution patterns
- Credential management

## Findings

### 1. Chrome Cookie Extraction (`src/chrome-cookies.ts`)

- **SAFE**: Extracts only `ct0` (CSRF) and `auth_token` cookies scoped to `.x.com` / `.twitter.com`
- Uses macOS Keychain via `security` command for decryption (standard Chrome approach)
- Temporary database copies are cleaned up after use
- Cookies remain in memory — never persisted to disk or sent to external services

### 2. Network Communication

All network requests target official X/Twitter endpoints only:

| Endpoint | Purpose |
|----------|---------|
| `https://api.x.com/2/oauth2/token` | OAuth token exchange |
| `https://api.x.com/2/users/me` | Get authenticated user |
| `https://api.x.com/2/users/{id}/bookmarks` | Fetch bookmarks (API v2) |
| `https://x.com/i/api/graphql/.../Bookmarks` | GraphQL bookmark sync |
| `https://twitter.com/i/oauth2/authorize` | OAuth authorization redirect |
| Twitter media CDN URLs | Media downloads from bookmarks |

**No third-party endpoints, analytics, telemetry, or data exfiltration detected.**

### 3. Data Storage

- All data stored locally in `~/.ft-bookmarks/`
- OAuth tokens saved with `chmod 0o600` (owner-only read/write)
- Bookmark data stored as JSONL cache + SQLite FTS5 index
- No remote sync, cloud storage, or external transmission

### 4. Dependencies

Five production dependencies, all mainstream and well-audited:
- `commander` — CLI framework
- `dotenv` — .env file loading
- `sql.js` / `sql.js-fts5` — SQLite in WebAssembly
- `zod` — schema validation

No suspicious lifecycle scripts (no postinstall hooks).

### 5. Code Safety

- No `eval()`, `Function()` constructor, or dynamic code execution
- No obfuscated code or hidden base64 payloads
- LLM classification uses local `claude`/`codex` CLI (not remote APIs)
- Prompt injection defenses for untrusted bookmark text
- Parameterized SQL queries (no injection risk)

### 6. Authentication Security

- OAuth 2.0 with PKCE (code challenge + state validation)
- Local callback server binds to `127.0.0.1` only
- Bearer token is X's public client token (standard for browser-based requests)

## Conclusion

This is a legitimate, privacy-focused, local-first CLI tool. It communicates
exclusively with official X/Twitter APIs, stores all data locally, and does not
steal or exfiltrate Chrome credentials or any other sensitive information.
