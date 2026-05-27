# SecureStorage – Architecture Decision Records

## ADR-001: AES-256 as Encryption Algorithm

**Date**: 2024  
**Status**: Accepted

### Context
Multiple encryption standards exist. The chosen algorithm must be: widely audited, hardware-accelerated on modern CPUs, quantum-resistant at its key size, and used in comparable high-security products.

### Decision
Use **AES-256** (same algorithm as Bitcoin cold wallets and the US military).

### Consequences
- **Positive**: battle-tested; no known practical attack.
- **Positive**: hardware AES-NI acceleration available on all modern CPUs.
- **Positive**: 256-bit key size provides adequate margin against future quantum attacks (Grover's algorithm halves effective key length to 128 bits).
- **Negative**: if a fundamental AES break is discovered, all stored data is at risk.

---

## ADR-002: Machine-Bound Default Key Derivation

**Date**: 2024  
**Status**: Accepted

### Context
The encryption key must be derived from something available on the device without user interaction, yet not easily guessable by an attacker who obtains the encrypted files.

### Decision
Derive the default key from `hash(domain + MachineName + UserName)`. This binds the data to a specific machine and OS user account.

### Consequences
- **Positive**: no user password needed for routine operation.
- **Positive**: encrypted files copied to another machine cannot be read without the original machine's credentials.
- **Negative**: if the machine is renamed or the user account renamed/deleted, stored data cannot be recovered.

---

## ADR-003: Domain Isolation

**Date**: 2024  
**Status**: Accepted

### Context
Multiple libraries in the same application may use SecureStorage. They must not be able to read each other's data.

### Decision
Each `Storage` instance is created with a **domain string**. The domain is hashed into the key derivation, making data from different domains cryptographically isolated.

### Consequences
- **Positive**: library A cannot read library B's secrets even if both run in the same process.
- **Negative**: domain strings must be stable; changing the domain string renders existing data inaccessible.

---

## ADR-004: Optional Hardware Key Store Delegation

**Date**: 2024  
**Status**: Accepted

### Context
For high-security deployments (HSM, TPM, Secure Enclave), the encryption key should never exist in software memory.

### Decision
Accept optional `getSecureKeyValue` / `setSecureKeyValue` delegates. When provided, all key material is stored and retrieved through the hardware module; the library itself never holds the raw key.

### Consequences
- **Positive**: supports the full spectrum from pure-software to hardware-backed security.
- **Positive**: the same API is used regardless of the security level.
- **Negative**: the security guarantee depends on the correctness of the delegate implementation provided by the caller.
