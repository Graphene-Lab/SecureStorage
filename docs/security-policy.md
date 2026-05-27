# SecureStorage – Security Policy

## Scope

SecureStorage is a cryptographic library. Security is its primary concern. This document describes the security model, supported versions, and the vulnerability disclosure process.

## Security Model

- **AES-256** encryption of all persisted data.
- Key is derived from `hash(domain + MachineName + UserName)` — bound to the machine and OS user by default.
- Optional hardware-backed key storage (TPM, Secure Enclave) via delegate injection.
- Domain isolation: data from one domain is cryptographically inaccessible to other domains.
- No network communication: the library is entirely local.

## Reporting a Vulnerability

1. **Do not** open a public GitHub issue.
2. Contact the maintainer privately (see repository contacts).
3. Include: description, affected version, reproduction steps, potential impact.
4. Allow up to **90 days** for a fix before public disclosure.

## Supported Versions

Only the latest published NuGet version is supported. Update promptly.

## Known Limitations

- The default key derivation ties data to a specific machine + OS user. If the machine name or user account changes, stored data cannot be recovered without a backup of the key.
- Hardware-backed key storage is only as secure as the hardware module provided by the caller.
- The `encrypted: false` constructor option removes all encryption. **Never use in production.**
