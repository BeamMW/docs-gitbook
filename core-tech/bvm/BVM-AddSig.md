# Env::AddSig

```C++
void AddSig(const PubKey& pubKey);
```
Adds the specified public key to the list of signatures mandatory for the current transaction kernel. Halts BVM execution and failes transaction if the specified key is not valid or the kernel is not signed with the private key fo the given public key.

## Parameters
* `pubKey `  : public key

## Return value
* none

## Notes
* Zero key (a.k.a. point at infinity) is NOT a valid key

## Example 