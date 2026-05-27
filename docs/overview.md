# SecureStorage – System Overview

## Purpose

**SecureStorage** is an encryption library that protects all data persisted by an application. It prevents sensitive information from being read by other processes, malicious apps, or an attacker who gains access to the device's storage.

The library provides three storage abstractions, all transparently encrypted with AES-256:

| Class | Purpose |
|---|---|
| `DataStorage` | Raw byte arrays (binary blobs) |
| `ObjectStorage` | Serialized .NET objects (any class) |
| `Values` | Named scalar values (string, int, bool, DateTime, …) |

## Cryptographic Model

- **Algorithm**: AES-256 (same as Bitcoin).
- **Key derivation**: key = hash(domain + MachineName + UserName), bound to the machine and OS user. An optional hardware-backed key store (e.g., TPM, Secure Enclave) can be provided.
- **Domain isolation**: each `Storage` instance uses a unique domain string, so multiple applications or library instances cannot read each other's data.
- **IsolatedStorage** backed: files are stored in the .NET `IsolatedStorage` area, invisible to ordinary file browsers.

## Hardware-Backed Keys (optional)

Pass `getSecureKeyValue` / `setSecureKeyValue` delegates to `Storage` to delegate key persistence to a hardware secure element:

```csharp
var storage = new Storage(
	domain: "MyApp",
	getSecureKeyValue: key => hardwareModule.Read(key),
	setSecureKeyValue: (key, value) => hardwareModule.Write(key, value)
);
```

This enables full hardware-level encryption analogous to Self-Encrypting Drives or hardware wallets.

## Design Principles

- **Defence-in-depth**: even if OS-level encryption (BitLocker, FileVault) is present, SecureStorage adds application-level encryption that other apps cannot bypass.
- **Zero-trust other apps**: each domain is isolated; cross-domain data access is cryptographically impossible without the domain key.
- **Open source**: the encryption implementation is fully auditable; no black-box dependencies.

## Target Framework

**.NET Standard 2.1**
