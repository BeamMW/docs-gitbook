```C++
struct FundsChange
{
	Amount m_Amount;
	AssetID m_Aid;
	uint8_t m_Consume;
};
```

Denotes the amount and the asset type that is supposed to be locked or released when the appropriate contract method is invoked. Used by the app shader as a parameter to `GenerateKernel` function.
* `m_Amount` - the amount
* `m_Aid` - asset ID. Set `0` for beams.
* `m_Consume`
   * non-zero if this amount is supposed to be spent by the user (i.e. locked by the contract)
   * `0` if this amount is supposed to be received by the user (i.e. released by the contract)

## Example
```C++
FundsChange fc;
fc.m_Amount = 5;
fc.m_Aid = 0;
fc.m_Consume = 1;

// Generate kernel for locking 5 beams by the contract with contract id cid
// Note: s_iMethod and cid are defined before
Env::GenerateKernel(&cid, s_iMethod, nullptr, 0, &fc, 1, nullptr, 0, "deposit to contract", 0);
```