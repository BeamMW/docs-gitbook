```C++
struct SigRequest
{
	const void* m_pID;
	uint32_t m_nID;
};
```

Describes the source material (opaque buffer), from which the user wallet generates the secret key.

* `m_pID` - pointer to the key material buffer
* `m_nID` - size in bytes of the key material

Application shaders can use arbitrary key materials, such as concatenated strings, `ContractID`, some context info and etc. The wallet would generate a unique secret key that is derived from its internal secret and the key material provided.

The secret key itself is not revealed to the shader, but it can get the appropriate public key that corresponds to it via `DerivePk` method.

The app shader should specify an array of `SigRequest` structures in a call to `GenerateKernel`, according to keys that contract shader would require during the method invocation (see `AddSig` for more info). By such the wallet signs the contract invocation kernel appropriately.
