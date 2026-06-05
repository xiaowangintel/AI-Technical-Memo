# VNCoercion.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/VNCoercion.cpp` | `llvm/lib/Transforms/Utils/VNCoercion.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements / Return true if coerceAvailableValueToLoadType will succeed. within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 VNCoercion 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-47

```cpp
#include "llvm/Transforms/Utils/VNCoercion.h"
#include "llvm/Analysis/ConstantFolding.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/IntrinsicInst.h"

#define DEBUG_TYPE "vncoerce"

using namespace llvm;
using namespace VNCoercion;

static bool isFirstClassAggregateOrScalableType(Type *Ty) {
  return Ty->isStructTy() || Ty->isArrayTy() || Ty->isScalableTy();
}

/// Return true if coerceAvailableValueToLoadType will succeed.
bool VNCoercion::canCoerceMustAliasedValueToLoad(Value *StoredVal, Type *LoadTy,
                                                 Function *F) {
  Type *StoredTy = StoredVal->getType();
  if (StoredTy == LoadTy)
    return true;

  const DataLayout &DL = F->getDataLayout();
  TypeSize MinStoreSize = DL.getTypeSizeInBits(StoredTy);
  TypeSize LoadSize = DL.getTypeSizeInBits(LoadTy);
  if (isa<ScalableVectorType>(StoredTy) && isa<ScalableVectorType>(LoadTy) &&
      MinStoreSize == LoadSize)
    return true;

  // If the loaded/stored value is a first class array/struct, don't try to
  // transform them. We need to be able to bitcast to integer. For scalable
  // vectors forwarded to fixed-sized vectors @llvm.vector.extract is used.
  if (isa<ScalableVectorType>(StoredTy) && isa<FixedVectorType>(LoadTy)) {
    if (StoredTy->getScalarType() != LoadTy->getScalarType())
      return false;

    // If it is known at compile-time that the VScale is larger than one,
    // use that information to allow for wider loads.
    const auto &Attrs = F->getAttributes().getFnAttrs();
    unsigned MinVScale = Attrs.getVScaleRangeMin();
    MinStoreSize =
        TypeSize::getFixed(MinStoreSize.getKnownMinValue() * MinVScale);
  } else if (isFirstClassAggregateOrScalableType(LoadTy) ||
             isFirstClassAggregateOrScalableType(StoredTy)) {
    return false;
  }

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 48-88

```cpp
  // The store size must be byte-aligned to support future type casts.
  if (llvm::alignTo(MinStoreSize, 8) != MinStoreSize)
    return false;

  // The store has to be at least as big as the load.
  if (!TypeSize::isKnownGE(MinStoreSize, LoadSize))
    return false;

  bool StoredNI = DL.isNonIntegralPointerType(StoredTy->getScalarType());
  bool LoadNI = DL.isNonIntegralPointerType(LoadTy->getScalarType());
  // Don't coerce non-integral pointers to integers or vice versa.
  if (StoredNI != LoadNI) {
    // As a special case, allow coercion of memset used to initialize
    // an array w/null.  Despite non-integral pointers not generally having a
    // specific bit pattern, we do assume null is zero.
    if (auto *CI = dyn_cast<Constant>(StoredVal))
      return CI->isNullValue();
    return false;
  } else if (StoredNI && LoadNI &&
             StoredTy->getPointerAddressSpace() !=
                 LoadTy->getPointerAddressSpace()) {
    return false;
  }

  // The implementation below uses inttoptr for vectors of unequal size; we
  // can't allow this for non integral pointers. We could teach it to extract
  // exact subvectors if desired.
  if (StoredNI && (StoredTy->isScalableTy() || MinStoreSize != LoadSize))
    return false;

  if (StoredTy->isTargetExtTy() || LoadTy->isTargetExtTy())
    return false;

  return true;
}

/// If we saw a store of a value to memory, and
/// then a load from a must-aliased pointer of a different type, try to coerce
/// the stored value.  LoadedTy is the type of the load we want to replace.
/// IRB is IRBuilder used to insert new instructions.
///
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 89-132

```cpp
/// If we can't do it, return null.
Value *VNCoercion::coerceAvailableValueToLoadType(Value *StoredVal,
                                                  Type *LoadedTy,
                                                  IRBuilderBase &Helper,
                                                  Function *F) {
  assert(canCoerceMustAliasedValueToLoad(StoredVal, LoadedTy, F) &&
         "precondition violation - materialization can't fail");
  const DataLayout &DL = F->getDataLayout();
  if (auto *C = dyn_cast<Constant>(StoredVal))
    StoredVal = ConstantFoldConstant(C, DL);

  // If this is already the right type, just return it.
  Type *StoredValTy = StoredVal->getType();

  // If this is a scalable vector forwarded to a fixed vector load, create
  // a @llvm.vector.extract instead of bitcasts.
  if (isa<ScalableVectorType>(StoredVal->getType()) &&
      isa<FixedVectorType>(LoadedTy)) {
    return Helper.CreateIntrinsic(LoadedTy, Intrinsic::vector_extract,
                                  {StoredVal, Helper.getInt64(0)});
  }

  TypeSize StoredValSize = DL.getTypeSizeInBits(StoredValTy);
  TypeSize LoadedValSize = DL.getTypeSizeInBits(LoadedTy);

  // If the store and reload are the same size, we can always reuse it.
  if (StoredValSize == LoadedValSize) {
    // Pointer to Pointer -> use bitcast.
    if (StoredValTy->isPtrOrPtrVectorTy() && LoadedTy->isPtrOrPtrVectorTy()) {
      StoredVal = Helper.CreateBitCast(StoredVal, LoadedTy);
    } else {
      // Convert source pointers to integers, which can be bitcast.
      if (StoredValTy->isPtrOrPtrVectorTy()) {
        StoredValTy = DL.getIntPtrType(StoredValTy);
        StoredVal = Helper.CreatePtrToInt(StoredVal, StoredValTy);
      }

      Type *TypeToCastTo = LoadedTy;
      if (TypeToCastTo->isPtrOrPtrVectorTy())
        TypeToCastTo = DL.getIntPtrType(TypeToCastTo);

      if (StoredValTy != TypeToCastTo)
        StoredVal = Helper.CreateBitCast(StoredVal, TypeToCastTo);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 133-174

```cpp
      // Cast to pointer if the load needs a pointer type.
      if (LoadedTy->isPtrOrPtrVectorTy())
        StoredVal = Helper.CreateIntToPtr(StoredVal, LoadedTy);
    }

    if (auto *C = dyn_cast<ConstantExpr>(StoredVal))
      StoredVal = ConstantFoldConstant(C, DL);

    return StoredVal;
  }
  // If the loaded value is smaller than the available value, then we can
  // extract out a piece from it.  If the available value is too small, then we
  // can't do anything.
  assert(!StoredValSize.isScalable() &&
         TypeSize::isKnownGE(StoredValSize, LoadedValSize) &&
         "canCoerceMustAliasedValueToLoad fail");

  // Convert source pointers to integers, which can be manipulated.
  if (StoredValTy->isPtrOrPtrVectorTy()) {
    StoredValTy = DL.getIntPtrType(StoredValTy);
    StoredVal = Helper.CreatePtrToInt(StoredVal, StoredValTy);
  }

  // Convert vectors and fp to integer, which can be manipulated.
  if (!StoredValTy->isIntegerTy()) {
    StoredValTy = IntegerType::get(StoredValTy->getContext(), StoredValSize);
    StoredVal = Helper.CreateBitCast(StoredVal, StoredValTy);
  }

  // If this is a big-endian system, we need to shift the value down to the low
  // bits so that a truncate will work.
  if (DL.isBigEndian()) {
    uint64_t ShiftAmt = DL.getTypeStoreSizeInBits(StoredValTy).getFixedValue() -
                        DL.getTypeStoreSizeInBits(LoadedTy).getFixedValue();
    StoredVal = Helper.CreateLShr(
        StoredVal, ConstantInt::get(StoredVal->getType(), ShiftAmt));
  }

  // Truncate the integer to the right size now.
  Type *NewIntTy = IntegerType::get(StoredValTy->getContext(), LoadedValSize);
  StoredVal = Helper.CreateTruncOrBitCast(StoredVal, NewIntTy);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 175-220

```cpp
  if (LoadedTy != NewIntTy) {
    // If the result is a pointer, inttoptr.
    if (LoadedTy->isPtrOrPtrVectorTy())
      StoredVal = Helper.CreateIntToPtr(StoredVal, LoadedTy);
    else
      // Otherwise, bitcast.
      StoredVal = Helper.CreateBitCast(StoredVal, LoadedTy);
  }

  if (auto *C = dyn_cast<Constant>(StoredVal))
    StoredVal = ConstantFoldConstant(C, DL);

  return StoredVal;
}

/// This function is called when we have a memdep query of a load that ends up
/// being a clobbering memory write (store, memset, memcpy, memmove).  This
/// means that the write *may* provide bits used by the load but we can't be
/// sure because the pointers don't must-alias.
///
/// Check this case to see if there is anything more we can do before we give
/// up.  This returns -1 if we have to give up, or a byte number in the stored
/// value of the piece that feeds the load.
static int analyzeLoadFromClobberingWrite(Type *LoadTy, Value *LoadPtr,
                                          Value *WritePtr,
                                          uint64_t WriteSizeInBits,
                                          const DataLayout &DL) {
  // If the loaded/stored value is a first class array/struct, or scalable type,
  // don't try to transform them. We need to be able to bitcast to integer.
  if (isFirstClassAggregateOrScalableType(LoadTy))
    return -1;

  int64_t StoreOffset = 0, LoadOffset = 0;
  Value *StoreBase =
      GetPointerBaseWithConstantOffset(WritePtr, StoreOffset, DL);
  Value *LoadBase = GetPointerBaseWithConstantOffset(LoadPtr, LoadOffset, DL);
  if (StoreBase != LoadBase)
    return -1;

  uint64_t LoadSize = DL.getTypeSizeInBits(LoadTy).getFixedValue();

  if ((WriteSizeInBits & 7) | (LoadSize & 7))
    return -1;
  uint64_t StoreSize = WriteSizeInBits / 8; // Convert to bytes.
  LoadSize /= 8;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 221-264

```cpp
  // If the Load isn't completely contained within the stored bits, we don't
  // have all the bits to feed it.  We could do something crazy in the future
  // (issue a smaller load then merge the bits in) but this seems unlikely to be
  // valuable.
  if (StoreOffset > LoadOffset ||
      StoreOffset + int64_t(StoreSize) < LoadOffset + int64_t(LoadSize))
    return -1;

  // Okay, we can do this transformation.  Return the number of bytes into the
  // store that the load is.
  return LoadOffset - StoreOffset;
}

/// This function is called when we have a
/// memdep query of a load that ends up being a clobbering store.
int VNCoercion::analyzeLoadFromClobberingStore(Type *LoadTy, Value *LoadPtr,
                                               StoreInst *DepSI,
                                               const DataLayout &DL) {
  auto *StoredVal = DepSI->getValueOperand();

  // Cannot handle reading from store of first-class aggregate or scalable type.
  if (isFirstClassAggregateOrScalableType(StoredVal->getType()))
    return -1;

  if (!canCoerceMustAliasedValueToLoad(StoredVal, LoadTy, DepSI->getFunction()))
    return -1;

  Value *StorePtr = DepSI->getPointerOperand();
  uint64_t StoreSize =
      DL.getTypeSizeInBits(DepSI->getValueOperand()->getType()).getFixedValue();
  return analyzeLoadFromClobberingWrite(LoadTy, LoadPtr, StorePtr, StoreSize,
                                        DL);
}

/// This function is called when we have a
/// memdep query of a load that ends up being clobbered by another load.  See if
/// the other load can feed into the second load.
int VNCoercion::analyzeLoadFromClobberingLoad(Type *LoadTy, Value *LoadPtr,
                                              LoadInst *DepLI,
                                              const DataLayout &DL) {
  // Cannot handle reading from store of first-class aggregate or scalable type.
  if (isFirstClassAggregateOrScalableType(DepLI->getType()))
    return -1;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 265-306

```cpp
  if (!canCoerceMustAliasedValueToLoad(DepLI, LoadTy, DepLI->getFunction()))
    return -1;

  Value *DepPtr = DepLI->getPointerOperand();
  uint64_t DepSize = DL.getTypeSizeInBits(DepLI->getType()).getFixedValue();
  return analyzeLoadFromClobberingWrite(LoadTy, LoadPtr, DepPtr, DepSize, DL);
}

int VNCoercion::analyzeLoadFromClobberingMemInst(Type *LoadTy, Value *LoadPtr,
                                                 MemIntrinsic *MI,
                                                 const DataLayout &DL) {
  // If the mem operation is a non-constant size, we can't handle it.
  ConstantInt *SizeCst = dyn_cast<ConstantInt>(MI->getLength());
  if (!SizeCst)
    return -1;
  uint64_t MemSizeInBits = SizeCst->getZExtValue() * 8;

  // If this is memset, we just need to see if the offset is valid in the size
  // of the memset..
  if (const auto *Memset = dyn_cast<MemSetInst>(MI)) {
    if (DL.isNonIntegralPointerType(LoadTy->getScalarType())) {
      auto *CI = dyn_cast<ConstantInt>(Memset->getValue());
      if (!CI || !CI->isZero())
        return -1;
    }
    return analyzeLoadFromClobberingWrite(LoadTy, LoadPtr, MI->getDest(),
                                          MemSizeInBits, DL);
  }

  // If we have a memcpy/memmove, the only case we can handle is if this is a
  // copy from constant memory.  In that case, we can read directly from the
  // constant memory.
  MemTransferInst *MTI = cast<MemTransferInst>(MI);

  Constant *Src = dyn_cast<Constant>(MTI->getSource());
  if (!Src)
    return -1;

  GlobalVariable *GV = dyn_cast<GlobalVariable>(getUnderlyingObject(Src));
  if (!GV || !GV->isConstant() || !GV->hasDefinitiveInitializer())
    return -1;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 307-351

```cpp
  // See if the access is within the bounds of the transfer.
  int Offset = analyzeLoadFromClobberingWrite(LoadTy, LoadPtr, MI->getDest(),
                                              MemSizeInBits, DL);
  if (Offset == -1)
    return Offset;

  // Otherwise, see if we can constant fold a load from the constant with the
  // offset applied as appropriate.
  unsigned IndexSize = DL.getIndexTypeSizeInBits(Src->getType());
  if (ConstantFoldLoadFromConstPtr(Src, LoadTy, APInt(IndexSize, Offset), DL))
    return Offset;
  return -1;
}

static Value *getStoreValueForLoadHelper(Value *SrcVal, unsigned Offset,
                                         Type *LoadTy, IRBuilderBase &Builder,
                                         const DataLayout &DL) {
  LLVMContext &Ctx = SrcVal->getType()->getContext();

  // If two pointers are in the same address space, they have the same size,
  // so we don't need to do any truncation, etc. This avoids introducing
  // ptrtoint instructions for pointers that may be non-integral.
  if (SrcVal->getType()->isPointerTy() && LoadTy->isPointerTy() &&
      cast<PointerType>(SrcVal->getType())->getAddressSpace() ==
          cast<PointerType>(LoadTy)->getAddressSpace()) {
    return SrcVal;
  }

  // Return scalable values directly to avoid needing to bitcast to integer
  // types, as we do not support non-zero Offsets.
  if (isa<ScalableVectorType>(LoadTy)) {
    assert(Offset == 0 && "Expected a zero offset for scalable types");
    return SrcVal;
  }

  // For the case of a scalable vector being forwarded to a fixed-sized load,
  // only equal element types are allowed and a @llvm.vector.extract will be
  // used instead of bitcasts.
  if (isa<ScalableVectorType>(SrcVal->getType()) &&
      isa<FixedVectorType>(LoadTy)) {
    assert(Offset == 0 &&
           SrcVal->getType()->getScalarType() == LoadTy->getScalarType());
    return SrcVal;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 352-392

```cpp
  uint64_t StoreSize =
      (DL.getTypeSizeInBits(SrcVal->getType()).getFixedValue() + 7) / 8;
  uint64_t LoadSize = (DL.getTypeSizeInBits(LoadTy).getFixedValue() + 7) / 8;
  // Compute which bits of the stored value are being used by the load.  Convert
  // to an integer type to start with.
  if (SrcVal->getType()->isPtrOrPtrVectorTy())
    SrcVal =
        Builder.CreatePtrToInt(SrcVal, DL.getIntPtrType(SrcVal->getType()));
  if (!SrcVal->getType()->isIntegerTy())
    SrcVal =
        Builder.CreateBitCast(SrcVal, IntegerType::get(Ctx, StoreSize * 8));

  // Shift the bits to the least significant depending on endianness.
  unsigned ShiftAmt;
  if (DL.isLittleEndian())
    ShiftAmt = Offset * 8;
  else
    ShiftAmt = (StoreSize - LoadSize - Offset) * 8;
  if (ShiftAmt)
    SrcVal = Builder.CreateLShr(SrcVal,
                                ConstantInt::get(SrcVal->getType(), ShiftAmt));

  if (LoadSize != StoreSize)
    SrcVal = Builder.CreateTruncOrBitCast(SrcVal,
                                          IntegerType::get(Ctx, LoadSize * 8));
  return SrcVal;
}

Value *VNCoercion::getValueForLoad(Value *SrcVal, unsigned Offset, Type *LoadTy,
                                   Instruction *InsertPt, Function *F) {
  const DataLayout &DL = F->getDataLayout();
#ifndef NDEBUG
  TypeSize MinSrcValSize = DL.getTypeStoreSize(SrcVal->getType());
  TypeSize LoadSize = DL.getTypeStoreSize(LoadTy);
  if (MinSrcValSize.isScalable() && !LoadSize.isScalable())
    MinSrcValSize =
        TypeSize::getFixed(MinSrcValSize.getKnownMinValue() *
                           F->getAttributes().getFnAttrs().getVScaleRangeMin());
  assert((MinSrcValSize.isScalable() || Offset + LoadSize <= MinSrcValSize) &&
         "Expected Offset + LoadSize <= SrcValSize");
  assert((!MinSrcValSize.isScalable() ||
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 393-432

```cpp
          (Offset == 0 && TypeSize::isKnownLE(LoadSize, MinSrcValSize))) &&
         "Expected offset of zero and LoadSize <= SrcValSize");
#endif
  IRBuilder<> Builder(InsertPt);
  SrcVal = getStoreValueForLoadHelper(SrcVal, Offset, LoadTy, Builder, DL);
  return coerceAvailableValueToLoadType(SrcVal, LoadTy, Builder, F);
}

Constant *VNCoercion::getConstantValueForLoad(Constant *SrcVal, unsigned Offset,
                                              Type *LoadTy,
                                              const DataLayout &DL) {
#ifndef NDEBUG
  unsigned SrcValSize = DL.getTypeStoreSize(SrcVal->getType()).getFixedValue();
  unsigned LoadSize = DL.getTypeStoreSize(LoadTy).getFixedValue();
  assert(Offset + LoadSize <= SrcValSize);
#endif
  return ConstantFoldLoadFromConst(SrcVal, LoadTy, APInt(32, Offset), DL);
}

/// This function is called when we have a
/// memdep query of a load that ends up being a clobbering mem intrinsic.
Value *VNCoercion::getMemInstValueForLoad(MemIntrinsic *SrcInst,
                                          unsigned Offset, Type *LoadTy,
                                          Instruction *InsertPt,
                                          const DataLayout &DL) {
  LLVMContext &Ctx = LoadTy->getContext();
  uint64_t LoadSize = DL.getTypeSizeInBits(LoadTy).getFixedValue() / 8;
  IRBuilder<> Builder(InsertPt);

  // We know that this method is only called when the mem transfer fully
  // provides the bits for the load.
  if (MemSetInst *MSI = dyn_cast<MemSetInst>(SrcInst)) {
    // memset(P, 'x', 1234) -> splat('x'), even if x is a variable, and
    // independently of what the offset is.
    Value *Val = MSI->getValue();
    if (LoadSize != 1)
      Val =
          Builder.CreateZExtOrBitCast(Val, IntegerType::get(Ctx, LoadSize * 8));
    Value *OneElt = Val;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 433-476

```cpp
    // Splat the value out to the right number of bits.
    for (unsigned NumBytesSet = 1; NumBytesSet != LoadSize;) {
      // If we can double the number of bytes set, do it.
      if (NumBytesSet * 2 <= LoadSize) {
        Value *ShVal = Builder.CreateShl(
            Val, ConstantInt::get(Val->getType(), NumBytesSet * 8));
        Val = Builder.CreateOr(Val, ShVal);
        NumBytesSet <<= 1;
        continue;
      }

      // Otherwise insert one byte at a time.
      Value *ShVal =
          Builder.CreateShl(Val, ConstantInt::get(Val->getType(), 1 * 8));
      Val = Builder.CreateOr(OneElt, ShVal);
      ++NumBytesSet;
    }

    return coerceAvailableValueToLoadType(Val, LoadTy, Builder,
                                          InsertPt->getFunction());
  }

  // Otherwise, this is a memcpy/memmove from a constant global.
  MemTransferInst *MTI = cast<MemTransferInst>(SrcInst);
  Constant *Src = cast<Constant>(MTI->getSource());
  unsigned IndexSize = DL.getIndexTypeSizeInBits(Src->getType());
  return ConstantFoldLoadFromConstPtr(Src, LoadTy, APInt(IndexSize, Offset),
                                      DL);
}

Constant *VNCoercion::getConstantMemInstValueForLoad(MemIntrinsic *SrcInst,
                                                     unsigned Offset,
                                                     Type *LoadTy,
                                                     const DataLayout &DL) {
  LLVMContext &Ctx = LoadTy->getContext();
  uint64_t LoadSize = DL.getTypeSizeInBits(LoadTy).getFixedValue() / 8;

  // We know that this method is only called when the mem transfer fully
  // provides the bits for the load.
  if (MemSetInst *MSI = dyn_cast<MemSetInst>(SrcInst)) {
    auto *Val = dyn_cast<ConstantInt>(MSI->getValue());
    if (!Val)
      return nullptr;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 477-487

```cpp
    Val = ConstantInt::get(Ctx, APInt::getSplat(LoadSize * 8, Val->getValue()));
    return ConstantFoldLoadFromConst(Val, LoadTy, DL);
  }

  // Otherwise, this is a memcpy/memmove from a constant global.
  MemTransferInst *MTI = cast<MemTransferInst>(SrcInst);
  Constant *Src = cast<Constant>(MTI->getSource());
  unsigned IndexSize = DL.getIndexTypeSizeInBits(Src->getType());
  return ConstantFoldLoadFromConstPtr(Src, LoadTy, APInt(IndexSize, Offset),
                                      DL);
}
```
- EN: This region continues the VNCoercion implementation with local helper logic centered on Val, ConstantInt, Ctx, APInt.
- CN: 这一段延续了 VNCoercion 的主体实现，围绕 Val, ConstantInt, Ctx, APInt 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `isFirstClassAggregateOrScalableType` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`isFirstClassAggregateOrScalableType` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/IRBuilder.h`, `llvm/IR/IntrinsicInst.h`, `llvm/Transforms/Utils/VNCoercion.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/IRBuilder.h`, `llvm/IR/IntrinsicInst.h`, `llvm/Transforms/Utils/VNCoercion.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Runtime analysis dependencies: `DataLayout`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
