# SecureStorage – Developer Guide

## Installation

```bash
dotnet add package SecureStorage
```

## Initialization

```csharp
using SecureStorage;

// Create a storage instance (domain = logical namespace)
var storage = new Storage(domain: "MyApplication");
```

For hardware-backed key storage:

```csharp
var storage = new Storage(
	domain: "MyApplication",
	getSecureKeyValue: key => MyHSM.Read(key),
	setSecureKeyValue: (key, value) => MyHSM.Write(key, value)
);
```

`Storage` must be instantiated **once per domain** per process lifetime. Creating a second instance with the same domain throws an exception.

---

## Values – Named Scalars

```csharp
// Write
storage.Values.Set("username", "alice");
storage.Values.Set("loginCount", 42);
storage.Values.Set("lastLogin", DateTime.UtcNow);

// Read
string name  = storage.Values.Get<string>("username");
int    count = storage.Values.Get<int>("loginCount", defaultValue: 0);

// Delete
storage.Values.Delete("username");
```

Supported types: `string`, `int`, `long`, `bool`, `DateTime`, `byte[]`, `decimal`.

---

## ObjectStorage – Serialized Objects

```csharp
// Save any serializable class
var contact = new Contact { Name = "Alice", PublicKey = keyBytes };
storage.ObjectStorage.Save("alice", contact);

// Load
var loaded = storage.ObjectStorage.Load<Contact>("alice");

// Delete
storage.ObjectStorage.Delete("alice");

// List all keys of a type
IEnumerable<string> keys = storage.ObjectStorage.Keys<Contact>();
```

Objects are serialized with `System.Text.Json` (or `XmlSerializer` fallback) and then AES-256 encrypted.

---

## DataStorage – Raw Bytes

```csharp
// Save
storage.DataStorage.Save("encryptionKey", keyBytes);

// Load
byte[] key = storage.DataStorage.Load("encryptionKey");

// Delete
storage.DataStorage.Delete("encryptionKey");
```

---

## Encryption-Disabled Mode

Encryption can be disabled (not recommended outside testing):

```csharp
var storage = new Storage(domain: "TestDomain", encrypted: false);
```

---

## Thread Safety

All public methods on `Storage`, `Values`, `ObjectStorage`, and `DataStorage` are thread-safe.

---

## Building

```powershell
dotnet build ..\..\SecureStorage\SecureStorage.csproj
```
