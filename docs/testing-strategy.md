# SecureStorage – Testing Strategy

## Overview

SecureStorage is a cryptographic persistence library. Tests must verify:

1. **Encryption/decryption round-trips** – data can be stored and retrieved correctly.
2. **Domain isolation** – a domain cannot read data written by another domain.
3. **Machine binding** – changing machine/user identity changes the derived key.
4. **Hardware delegate** – custom key store delegates are called correctly.
5. **Type coverage** – all supported scalar types round-trip correctly.

---

## 1. Values Round-Trip Tests

```csharp
[Fact]
public void Values_String_RoundTrip()
{
	var storage = new Storage("TestDomain_" + Guid.NewGuid());
	storage.Values.Set("key", "hello world");
	Assert.Equal("hello world", storage.Values.Get<string>("key"));
}

[Theory]
[InlineData(42)]
[InlineData(-1)]
[InlineData(int.MaxValue)]
public void Values_Int_RoundTrip(int value)
{
	var storage = new Storage("TestDomain_" + Guid.NewGuid());
	storage.Values.Set("n", value);
	Assert.Equal(value, storage.Values.Get<int>("n"));
}
```

---

## 2. ObjectStorage Round-Trip Tests

```csharp
[Fact]
public void ObjectStorage_ComplexObject_RoundTrip()
{
	var storage = new Storage("TestDomain_" + Guid.NewGuid());
	var original = new MyData { Name = "Alice", Score = 99 };

	storage.ObjectStorage.Save("alice", original);
	var loaded = storage.ObjectStorage.Load<MyData>("alice");

	Assert.Equal(original.Name, loaded.Name);
	Assert.Equal(original.Score, loaded.Score);
}
```

---

## 3. Domain Isolation Test

```csharp
[Fact]
public void DomainA_Cannot_Read_DomainB_Data()
{
	var a = new Storage("DomainA_" + Guid.NewGuid());
	var b = new Storage("DomainB_" + Guid.NewGuid());

	a.Values.Set("secret", "domainA_value");

	// Domain B has a different key; it should not find Domain A's key
	Assert.Null(b.Values.Get<string>("secret"));
}
```

---

## 4. Hardware Delegate Test

```csharp
[Fact]
public void HardwareDelegate_IsCalledForKeyStorage()
{
	var setCalls = new List<(string key, string value)>();
	var getCalls = new List<string>();
	var store    = new Dictionary<string, string>();

	var storage = new Storage(
		domain: "HWTest_" + Guid.NewGuid(),
		getSecureKeyValue: key => { getCalls.Add(key); return store.GetValueOrDefault(key); },
		setSecureKeyValue: (key, value) => { setCalls.Add((key, value)); store[key] = value; }
	);

	storage.Values.Set("pin", "1234");
	_ = storage.Values.Get<string>("pin");

	Assert.NotEmpty(setCalls);
	Assert.NotEmpty(getCalls);
}
```

---

## 5. Delete and Missing Key Tests

```csharp
[Fact]
public void Values_Delete_RemovesEntry()
{
	var storage = new Storage("Del_" + Guid.NewGuid());
	storage.Values.Set("x", "v");
	storage.Values.Delete("x");
	Assert.Null(storage.Values.Get<string>("x"));
}

[Fact]
public void Values_MissingKey_ReturnsDefault()
{
	var storage = new Storage("Miss_" + Guid.NewGuid());
	Assert.Equal(0, storage.Values.Get<int>("nonexistent", defaultValue: 0));
}
```

---

## Test Coverage Goals

| Area | Target |
|---|---|
| Values (all types) | 100 % |
| ObjectStorage | ≥ 90 % |
| DataStorage | ≥ 90 % |
| Domain isolation | 100 % |
| Hardware delegate wiring | 100 % |

---

## Running Tests

```powershell
dotnet test ..\..\SecureStorage\
```
