# Types and structs

### Primitive types
* `Height` - 64-bit unsigned integer (`uint64_t`), denotes blockchain height
* `Amount` - 64-bit unsigned integer (`uint64_t`), denotes value in basic uints (groths)
* `AssetID` - 32-bit integer (`uint32_t`), used as the asset identifier. Value `0` is reserved for _Beam_.
* `Timestamp` - 64-bit unsigned integer (`uint64_t`), UTC time in seconds.

### Opaque types
* `PubKey` - 33-byte long representation of a public key (secp256k1 group element, i.e. point).
* `ShaderID` - 32-byte long shader ID (derived from its compiled bytecode)
* `ContractID` - 32-byte long contract ID
* `HashObj` - opaque handle, represents a hash processor object managed by BVM.
* `Secp_scalar` - opaque handle, represents a secp256k1 scalar object managed by BVM.
* `Secp_point` - opaque handle, represents a secp256k1 scalar object managed by BVM.

### Others
* [FundsChange](BVM-FundsChange.md)
* [SigRequest](BVM-SigRequest.md)
* `BlockHeader`
* [KeyTag](BVM-KeyTag.md)
* `Merkle`


# Common functions (for both Contract and App shaders)

## General
* [Halt](BVM-Halt.md)
* [get_Height](BVM-get_Height.md)
* [get_HdrInfo](BVM-get_HdrInfo.md)
* [get_HdrFull](BVM-get_HdrFull.md)
* [get_RulesCfg](BVM-get_RulesCfg.md)
* [Write](BVM-Write.md)

## Raw memory utility functions

* [Memcpy](BVM-Memcpy.md)
* [Memset](BVM-Memset.md)
* [Memcmp](BVM-Memcmp.md)
* [Memis0](BVM-Memis0.md)
* [Strlen](BVM-Strlen.md)
* [Strcmp](BVM-Strcmp.md)

## Memory allocation
	
* [StackAlloc](BVM-StackAlloc.md)
* [StackFree](BVM-StackFree.md)
* [Heap_Alloc](BVM-Heap_Alloc.md)
* [Heap_Free](BVM-Heap_Free.md)

## Hash functions:

* [HashCreateSha256](BVM-HashCreateSha256.md)
* [HashCreateBlake2b](BVM-HashCreateBlake2b.md)
* [HashCreateKeccak](BVM-HashCreateKeccak.md)
* [HashWrite](BVM-HashWrite.md)
* [HashGetValue](BVM-HashGetValue.md)
* [HashFree](BVM-HashFree.md)
* [HashClone](BVM-HashClone.md)
* [VerifyBeamHashIII](BVM-VerifyBeamHashIII.md)

## secp256k1 (elliptic curve cryptography) functions:
* [Secp_Scalar_alloc](BVM-Secp_Scalar_alloc.md)
* [Secp_Scalar_free](BVM-Secp_Scalar_free.md)
* [Secp_Scalar_import](BVM-Secp_Scalar_import.md)
* [Secp_Scalar_export](BVM-Secp_Scalar_export.md)
* [Secp_Scalar_neg](BVM-Secp_Scalar_neg.md)
* [Secp_Scalar_add](BVM-Secp_Scalar_add.md)
* [Secp_Scalar_mul](BVM-Secp_Scalar_mul.md)
* [Secp_Scalar_inv](BVM-Secp_Scalar_inv.md)
* [Secp_Scalar_set](BVM-Secp_Scalar_set.md)
* [Secp_Point_alloc](BVM-Secp_Point_alloc.md)
* [Secp_Point_free](BVM-Secp_Point_free.md)
* [Secp_Point_Import](BVM-Secp_Point_Import.md)
* [Secp_Point_Export](BVM-Secp_Point_Export.md)
* [Secp_Point_neg](BVM-Secp_Point_neg.md)
* [Secp_Point_add](BVM-Secp_Point_add.md)
* [Secp_Point_mul](BVM-Secp_Point_mul.md)
* [Secp_Point_IsZero](BVM-Secp_Point_IsZero.md)
* [Secp_Point_mul_G](BVM-Secp_Point_mul_G.md)
* [Secp_Point_mul_J](BVM-Secp_Point_mul_J.md)
* [Secp_Point_mul_H](BVM-Secp_Point_mul_H.md)

# Contract-only functions

## General
* [LoadVar](BVM-LoadVar.md) 
* [LoadVarEx](BVM-LoadVarEx.md)
* [SaveVar](BVM-SaveVar.md)
* [EmitLog](BVM-EmitLog.md)
* [UpdateShader](BVM-UpdateShader.md)

## Cross-contract
* [CallFar](BVM-CallFar.md)
* [get_CallDepth](BVM-get_CallDepth.md)
* [get_CallerCid](BVM-get_CallerCid.md)
* [RefAdd](BVM-RefAdd.md)
* [RefRelease](BVM-RefRelease.md)

## Funds and signature management
* [AddSig](BVM-AddSig.md)
* [FundsLock](BVM-FundsLock.md)
* [FundsUnlock](BVM-FundsUnlock.md)

## Asset management
* [AssetCreate](BVM-AssetCreate.md)
* [AssetEmit](BVM-AssetEmit.md)
* [AssetDestroy](BVM-AssetDestroy.md)


# Application-only functions

## Variable management
* [Vars_Enum](BVM-Vars_Enum.md)
* [Vars_MoveNext](BVM-Vars_MoveNext.md)
* [Vars_Close](BVM-Vars_Close.md)
* [VarGetProof](BVM-VarGetProof.md)
## Log management
* [Logs_Enum](BVM-Logs_Enum.md)
* [Logs_MoveNext](BVM-Logs_MoveNext.md)
* [Logs_Close](BVM-Logs_Close.md)
* [LogGetProof](BVM-LogGetProof.md)
## Key management
* [DerivePk](BVM-DerivePk.md)
## Application JSON response
* [DocAddGroup](BVM-DocAddGroup.md)
* [DocCloseGroup](BVM-DocCloseGroup.md)
* [DocAddText](BVM-DocAddText.md)
* [DocAddNum32](BVM-DocAddNum32.md)
* [DocAddNum64](BVM-DocAddNum64.md)
* [DocAddArray](BVM-DocAddArray.md)
* [DocCloseArray](BVM-DocCloseArray.md)
* [DocAddBlob](BVM-DocAddBlob.md)
# Application parameters
* [DocGetText](BVM-DocGetText.md)
* [DocGetNum32](BVM-DocGetNum32.md)
* [DocGetNum64](BVM-DocGetNum64.md)
* [DocGetBlob](BVM-DocGetBlob.md)
## Transaction management
* [SelectContext](BVM-SelectContext.md)
* [GenerateKernel](BVM-GenerateKernel.md)
* [GenerateKernelAdvanced](BVM-GenerateKernelAdvanced.md)
* [GenerateRandom](BVM-GenerateRandom.md)
## Nonce management
* [SlotInit](BVM-SlotInit.md)
* [get_SlotImage](BVM-get_SlotImage.md)
* [get_SlotImageEx](BVM-get_SlotImageEx.md)
* [get_BlindSk](BVM-get_BlindSk.md)
* [get_Pk](BVM-get_Pk.md)
* [get_PkEx](BVM-get_PkEx.md)
## Communication
* [Comm_Listen](BVM-Comm_Listen.md)
* [Comm_Send](BVM-Comm_Send.md)
* [Comm_Read](BVM-Comm_Read.md)
* [Comm_WaitMsg](BVM-Comm_WaitMsg.md)

# C++ API
```C++

// Common functions
void Write(const void* pData, uint32_t nData, uint32_t iStream);
void* Memcpy(void* pDst, const void* pSrc, uint32_t size);
void* Memset(void* pDst, uint8_t val, uint32_t size);
int32_t Memcmp(const void* p1, const void* p2, uint32_t size);
uint8_t Memis0(const void* p, uint32_t size);
uint32_t Strlen(const char* sz);
int32_t Strcmp(const char* sz1, const char* sz2);
void* StackAlloc(uint32_t size);
void StackFree(uint32_t size);
void* Heap_Alloc(uint32_t size);
void Heap_Free(void* pPtr);
void Halt();
void HashWrite(HashObj* pHash, const void* p, uint32_t size);
void HashGetValue(HashObj* pHash, void* pDst, uint32_t size);
void HashFree(HashObj* pHash);
HashObj* HashClone(HashObj* pHash);
Height get_Height();
void get_HdrInfo(BlockHeader::Info& hdr);
void get_HdrFull(BlockHeader::Full& hdr);
Height get_RulesCfg(Height h, HashValue& res);
HashObj* HashCreateSha256();
HashObj* HashCreateBlake2b(const void* pPersonal, uint32_t nPersonal, uint32_t nResultSize);
HashObj* HashCreateKeccak(uint32_t nBits);
Secp_scalar* Secp_Scalar_alloc();
void Secp_Scalar_free(Secp_scalar& s);
uint8_t Secp_Scalar_import(Secp_scalar& s, const Secp_scalar_data& data);
void Secp_Scalar_export(const Secp_scalar& s, Secp_scalar_data& data);
void Secp_Scalar_neg(Secp_scalar& dst, const Secp_scalar& src);
void Secp_Scalar_add(Secp_scalar& dst, const Secp_scalar& a, const Secp_scalar& b);
void Secp_Scalar_mul(Secp_scalar& dst, const Secp_scalar& a, const Secp_scalar& b);
void Secp_Scalar_inv(Secp_scalar& dst, const Secp_scalar& src);
void Secp_Scalar_set(Secp_scalar& dst, uint64_t val);
Secp_point* Secp_Point_alloc();
void Secp_Point_free(Secp_point& p);
uint8_t Secp_Point_Import(Secp_point& p, const PubKey& pk);
void Secp_Point_Export(const Secp_point& p, PubKey& pk);
void Secp_Point_neg(Secp_point& dst, const Secp_point& src);
void Secp_Point_add(Secp_point& dst, const Secp_point& a, const Secp_point& b);
void Secp_Point_mul(Secp_point& dst, const Secp_point& p, const Secp_scalar& s);
uint8_t Secp_Point_IsZero(const Secp_point& p);
void Secp_Point_mul_G(Secp_point& dst, const Secp_scalar& s);
void Secp_Point_mul_J(Secp_point& dst, const Secp_scalar& s);
void Secp_Point_mul_H(Secp_point& dst, const Secp_scalar& s, AssetID aid);
uint8_t VerifyBeamHashIII(const void* pInp, uint32_t nInp, const void* pNonce, uint32_t nNonce, const void* pSol, uint32_t nSol);

// Contract only functions
uint32_t LoadVar(const void* pKey, uint32_t nKey, void* pVal, uint32_t nVal, uint8_t nType);
uint32_t SaveVar(const void* pKey, uint32_t nKey, const void* pVal, uint32_t nVal, uint8_t nType);
uint32_t EmitLog(const void* pKey, uint32_t nKey, const void* pVal, uint32_t nVal, uint8_t nType);
void CallFar(const ContractID& cid, uint32_t iMethod, void* pArgs, uint32_t nArgs, uint8_t bInheritContext);
uint32_t get_CallDepth();
void get_CallerCid(uint32_t iCaller, ContractID& cid);
void UpdateShader(const void* pVal, uint32_t nVal);
void LoadVarEx(void* pKey, uint32_t& nKey, uint32_t nKeyBufSize, void* pVal, uint32_t& nVal, uint8_t nType, uint8_t nSearchFlag);
void AddSig(const PubKey& pubKey);
void FundsLock(AssetID aid, Amount amount);
void FundsUnlock(AssetID aid, Amount amount);
uint8_t RefAdd(const ContractID& cid);
uint8_t RefRelease(const ContractID& cid);
AssetID AssetCreate(const void* pMeta, uint32_t nMeta);
uint8_t AssetEmit(AssetID aid, Amount amount, uint8_t bEmit);
uint8_t AssetDestroy(AssetID aid);

// Application-only functions
void SelectContext(uint8_t bDependent, uint32_t nChargeNeeded);
uint32_t Vars_Enum(const void* pKey0, uint32_t nKey0, const void* pKey1, uint32_t nKey1);
uint8_t Vars_MoveNext(uint32_t iSlot, void* pKey, uint32_t& nKey, void* pVal, uint32_t& nVal, uint8_t nRepeat);
void Vars_Close(uint32_t iSlot);
uint32_t VarGetProof(const void* pKey, uint32_t nKey, const void** ppVal, uint32_t* pnVal, const Merkle::Node** ppProof);
uint32_t Logs_Enum(const void* pKey0, uint32_t nKey0, const void* pKey1, uint32_t nKey1, const HeightPos* pPosMin, const HeightPos* pPosMax);
uint8_t Logs_MoveNext(uint32_t iSlot, void* pKey, uint32_t& nKey, void* pVal, uint32_t& nVal, HeightPos& pos, uint8_t nRepeat);
void Logs_Close(uint32_t iSlot);
uint32_t LogGetProof(const HeightPos& pos, const Merkle::Node** ppProof);
void DerivePk(PubKey& pubKey, const void* pID, uint32_t nID);
void DocAddGroup(const char* szID);
void DocCloseGroup();
void DocAddText(const char* szID, const char* val);
void DocAddNum32(const char* szID, uint32_t val);
void DocAddNum64(const char* szID, uint64_t val);
void DocAddArray(const char* szID);
void DocCloseArray();
void DocAddBlob(const char* szID, const void* pBlob, uint32_t nBlob);
uint32_t DocGetText(const char* szID, char* szRes, uint32_t nLen);
uint8_t DocGetNum32(const char* szID, uint32_t* pOut);
uint8_t DocGetNum64(const char* szID, uint64_t* pOut);
uint32_t DocGetBlob(const char* szID, void* pOut, uint32_t nLen);
void GenerateKernel(const ContractID* pCid, uint32_t iMethod, const void* pArg, uint32_t nArg, const FundsChange* pFunds, uint32_t nFunds, const SigRequest* pSig, uint32_t nSig, const char* szComment, uint32_t nCharge);
void GenerateRandom(void* pBuf, uint32_t nSize);
void get_SlotImage(Secp_point& res, uint32_t iSlot);
void SlotInit(const void* pExtraSeed, uint32_t nExtraSeed, uint32_t iSlot);
void get_Pk(Secp_point& res, const void* pID, uint32_t nID);
void get_BlindSk(Secp_scalar& res, const void* pID, uint32_t nID, const Secp_scalar& mul, uint32_t iSlot);
void GenerateKernelAdvanced(const ContractID* pCid, uint32_t iMethod, const void* pArg, uint32_t nArg, const FundsChange* pFunds, uint32_t nFunds, const PubKey* pSig, uint32_t nSig, const char* szComment, uint32_t nCharge, Height hMin, Height hMax, const PubKey& ptFullBlind, const PubKey& ptFullNonce, const Secp_scalar_data& skForeignSig, uint32_t iSlotBlind, uint32_t iSlotNonce, Secp_scalar_data* pChallenges);
void get_SlotImageEx(Secp_point& res, const Secp_point& gen, uint32_t iSlot);
void get_PkEx(Secp_point& res, const Secp_point& gen, const void* pID, uint32_t nID);
void Comm_Listen(const void* pID, uint32_t nID, uint32_t nCookie);
void Comm_Send(const PubKey& pkRemote, const void* pBuf, uint32_t nSize);
uint32_t Comm_Read(void* pBuf, uint32_t nSize, uint32_t* pCookie, uint8_t bKeep);
void Comm_WaitMsg(uint32_t nTimeout_ms);
```