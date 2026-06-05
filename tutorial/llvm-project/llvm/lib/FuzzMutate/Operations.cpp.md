# Operations.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/FuzzMutate/Operations.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements IR mutation utilities used for fuzzing and randomized testing.
  - **CN**: 实现用于模糊测试与随机化测试的 IR 变异工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
//===-- Operations.cpp ----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/FuzzMutate/Operations.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/FuzzMutate/Operations.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/FuzzMutate/Operations.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`。

### Lines 15-32
```cpp
using namespace llvm;
using namespace fuzzerop;

void llvm::describeFuzzerIntOps(std::vector<fuzzerop::OpDescriptor> &Ops) {
  Ops.push_back(binOpDescriptor(1, Instruction::Add));
  Ops.push_back(binOpDescriptor(1, Instruction::Sub));
  Ops.push_back(binOpDescriptor(1, Instruction::Mul));
  Ops.push_back(binOpDescriptor(1, Instruction::SDiv));
  Ops.push_back(binOpDescriptor(1, Instruction::UDiv));
  Ops.push_back(binOpDescriptor(1, Instruction::SRem));
  Ops.push_back(binOpDescriptor(1, Instruction::URem));
  Ops.push_back(binOpDescriptor(1, Instruction::Shl));
  Ops.push_back(binOpDescriptor(1, Instruction::LShr));
  Ops.push_back(binOpDescriptor(1, Instruction::AShr));
  Ops.push_back(binOpDescriptor(1, Instruction::And));
  Ops.push_back(binOpDescriptor(1, Instruction::Or));
  Ops.push_back(binOpDescriptor(1, Instruction::Xor));

```
- **EN**: Introduces declarations for `llvm`, `fuzzerop`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm`, `fuzzerop` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 33-44
```cpp
  Ops.push_back(cmpOpDescriptor(1, Instruction::ICmp, CmpInst::ICMP_EQ));
  Ops.push_back(cmpOpDescriptor(1, Instruction::ICmp, CmpInst::ICMP_NE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::ICmp, CmpInst::ICMP_UGT));
  Ops.push_back(cmpOpDescriptor(1, Instruction::ICmp, CmpInst::ICMP_UGE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::ICmp, CmpInst::ICMP_ULT));
  Ops.push_back(cmpOpDescriptor(1, Instruction::ICmp, CmpInst::ICMP_ULE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::ICmp, CmpInst::ICMP_SGT));
  Ops.push_back(cmpOpDescriptor(1, Instruction::ICmp, CmpInst::ICMP_SGE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::ICmp, CmpInst::ICMP_SLT));
  Ops.push_back(cmpOpDescriptor(1, Instruction::ICmp, CmpInst::ICMP_SLE));
}

```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

### Lines 45-64
```cpp
void llvm::describeFuzzerFloatOps(std::vector<fuzzerop::OpDescriptor> &Ops) {
  Ops.push_back(binOpDescriptor(1, Instruction::FAdd));
  Ops.push_back(binOpDescriptor(1, Instruction::FSub));
  Ops.push_back(binOpDescriptor(1, Instruction::FMul));
  Ops.push_back(binOpDescriptor(1, Instruction::FDiv));
  Ops.push_back(binOpDescriptor(1, Instruction::FRem));

  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_FALSE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_OEQ));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_OGT));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_OGE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_OLT));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_OLE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_ONE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_ORD));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_UNO));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_UEQ));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_UGT));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_UGE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_ULT));
```
- **EN**: Implements logic around `describeFuzzerFloatOps`, `push_back`.
- **CN**: 围绕 `describeFuzzerFloatOps`, `push_back` 实现具体逻辑。

### Lines 65-74
```cpp
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_ULE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_UNE));
  Ops.push_back(cmpOpDescriptor(1, Instruction::FCmp, CmpInst::FCMP_TRUE));
}

void llvm::describeFuzzerUnaryOperations(
    std::vector<fuzzerop::OpDescriptor> &Ops) {
  Ops.push_back(fnegDescriptor(1));
}

```
- **EN**: Implements logic around `push_back`, `describeFuzzerUnaryOperations`.
- **CN**: 围绕 `push_back`, `describeFuzzerUnaryOperations` 实现具体逻辑。

### Lines 75-87
```cpp
void llvm::describeFuzzerControlFlowOps(
    std::vector<fuzzerop::OpDescriptor> &Ops) {
  Ops.push_back(splitBlockDescriptor(1));
}

void llvm::describeFuzzerOtherOps(std::vector<fuzzerop::OpDescriptor> &Ops) {
  Ops.push_back(selectDescriptor(1));
}

void llvm::describeFuzzerPointerOps(std::vector<fuzzerop::OpDescriptor> &Ops) {
  Ops.push_back(gepDescriptor(1));
}

```
- **EN**: Implements logic around `describeFuzzerControlFlowOps`, `push_back`, `describeFuzzerOtherOps`, `describeFuzzerPointerOps`.
- **CN**: 围绕 `describeFuzzerControlFlowOps`, `push_back`, `describeFuzzerOtherOps`, `describeFuzzerPointerOps` 实现具体逻辑。

### Lines 88-99
```cpp
void llvm::describeFuzzerAggregateOps(
    std::vector<fuzzerop::OpDescriptor> &Ops) {
  Ops.push_back(extractValueDescriptor(1));
  Ops.push_back(insertValueDescriptor(1));
}

void llvm::describeFuzzerVectorOps(std::vector<fuzzerop::OpDescriptor> &Ops) {
  Ops.push_back(extractElementDescriptor(1));
  Ops.push_back(insertElementDescriptor(1));
  Ops.push_back(shuffleVectorDescriptor(1));
}

```
- **EN**: Implements logic around `describeFuzzerAggregateOps`, `push_back`, `describeFuzzerVectorOps`.
- **CN**: 围绕 `describeFuzzerAggregateOps`, `push_back`, `describeFuzzerVectorOps` 实现具体逻辑。

### Lines 100-115
```cpp
OpDescriptor llvm::fuzzerop::selectDescriptor(unsigned Weight) {
  auto buildOp = [](ArrayRef<Value *> Srcs, BasicBlock::iterator InsertPt) {
    return SelectInst::Create(Srcs[0], Srcs[1], Srcs[2], "S", InsertPt);
  };
  return {Weight,
          {boolOrVecBoolType(), matchFirstLengthWAnyType(), matchSecondType()},
          buildOp};
}

OpDescriptor llvm::fuzzerop::fnegDescriptor(unsigned Weight) {
  auto buildOp = [](ArrayRef<Value *> Srcs, BasicBlock::iterator InsertPt) {
    return UnaryOperator::Create(Instruction::FNeg, Srcs[0], "F", InsertPt);
  };
  return {Weight, {anyFloatOrVecFloatType()}, buildOp};
}

```
- **EN**: Implements logic around `selectDescriptor`, `Create`, `boolOrVecBoolType`, `fnegDescriptor`, and 1 more symbols.
- **CN**: 围绕 `selectDescriptor`, `Create`, `boolOrVecBoolType`, `fnegDescriptor`, and 1 more symbols 实现具体逻辑。

### Lines 116-135
```cpp
OpDescriptor llvm::fuzzerop::binOpDescriptor(unsigned Weight,
                                             Instruction::BinaryOps Op) {
  auto buildOp = [Op](ArrayRef<Value *> Srcs, BasicBlock::iterator InsertPt) {
    return BinaryOperator::Create(Op, Srcs[0], Srcs[1], "B", InsertPt);
  };
  switch (Op) {
  case Instruction::Add:
  case Instruction::Sub:
  case Instruction::Mul:
  case Instruction::SDiv:
  case Instruction::UDiv:
  case Instruction::SRem:
  case Instruction::URem:
  case Instruction::Shl:
  case Instruction::LShr:
  case Instruction::AShr:
  case Instruction::And:
  case Instruction::Or:
  case Instruction::Xor:
    return {Weight, {anyIntOrVecIntType(), matchFirstType()}, buildOp};
```
- **EN**: Implements logic around `binOpDescriptor`, `Create`, `anyIntOrVecIntType`; this block uses `switch`-style dispatch.
- **CN**: 围绕 `binOpDescriptor`, `Create`, `anyIntOrVecIntType` 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 136-147
```cpp
  case Instruction::FAdd:
  case Instruction::FSub:
  case Instruction::FMul:
  case Instruction::FDiv:
  case Instruction::FRem:
    return {Weight, {anyFloatOrVecFloatType(), matchFirstType()}, buildOp};
  case Instruction::BinaryOpsEnd:
    llvm_unreachable("Value out of range of enum");
  }
  llvm_unreachable("Covered switch");
}

```
- **EN**: Implements logic around `anyFloatOrVecFloatType`, `llvm_unreachable`; this block uses `switch`-style dispatch.
- **CN**: 围绕 `anyFloatOrVecFloatType`, `llvm_unreachable` 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 148-165
```cpp
OpDescriptor llvm::fuzzerop::cmpOpDescriptor(unsigned Weight,
                                             Instruction::OtherOps CmpOp,
                                             CmpInst::Predicate Pred) {
  auto buildOp = [CmpOp, Pred](ArrayRef<Value *> Srcs,
                               BasicBlock::iterator InsertPt) {
    return CmpInst::Create(CmpOp, Pred, Srcs[0], Srcs[1], "C", InsertPt);
  };

  switch (CmpOp) {
  case Instruction::ICmp:
    return {Weight, {anyIntOrVecIntType(), matchFirstType()}, buildOp};
  case Instruction::FCmp:
    return {Weight, {anyFloatOrVecFloatType(), matchFirstType()}, buildOp};
  default:
    llvm_unreachable("CmpOp must be ICmp or FCmp");
  }
}

```
- **EN**: Implements logic around `cmpOpDescriptor`, `Create`, `anyIntOrVecIntType`, `anyFloatOrVecFloatType`, and 1 more symbols; this block uses `switch`-style dispatch.
- **CN**: 围绕 `cmpOpDescriptor`, `Create`, `anyIntOrVecIntType`, `anyFloatOrVecFloatType`, and 1 more symbols 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 166-175
```cpp
OpDescriptor llvm::fuzzerop::splitBlockDescriptor(unsigned Weight) {
  auto buildSplitBlock = [](ArrayRef<Value *> Srcs,
                            BasicBlock::iterator InsertPt) {
    BasicBlock *Block = InsertPt->getParent();
    BasicBlock *Next = Block->splitBasicBlock(InsertPt, "BB");

    // If it was an exception handling block, we are done.
    if (Block->isEHPad())
      return nullptr;

```
- **EN**: Implements logic around `splitBlockDescriptor`, `getParent`, `splitBasicBlock`, `isEHPad`.
- **CN**: 围绕 `splitBlockDescriptor`, `getParent`, `splitBasicBlock`, `isEHPad` 实现具体逻辑。

### Lines 176-195
```cpp
    // Loop back on this block by replacing the unconditional forward branch
    // with a conditional with a backedge.
    if (Block != &Block->getParent()->getEntryBlock()) {
      CondBrInst::Create(Srcs[0], Block, Next,
                         Block->getTerminator()->getIterator());
      Block->getTerminator()->eraseFromParent();

      // We need values for each phi in the block. Since there isn't a good way
      // to do a variable number of input values currently, we just fill them
      // with poison.
      for (PHINode &PHI : Block->phis())
        PHI.addIncoming(PoisonValue::get(PHI.getType()), Block);
    }
    return nullptr;
  };
  SourcePred isInt1Ty{[](ArrayRef<Value *>, const Value *V) {
                        return V->getType()->isIntegerTy(1);
                      },
                      std::nullopt};
  return {Weight, {isInt1Ty}, buildSplitBlock};
```
- **EN**: Implements logic around `getParent`, `Create`, `getTerminator`, `phis`, and 2 more symbols.
- **CN**: 围绕 `getParent`, `Create`, `getTerminator`, `phis`, and 2 more symbols 实现具体逻辑。

### Lines 196-214
```cpp
}

OpDescriptor llvm::fuzzerop::gepDescriptor(unsigned Weight) {
  auto buildGEP = [](ArrayRef<Value *> Srcs, BasicBlock::iterator InsertPt) {
    // TODO: It would be better to generate a random type here, rather than
    // generating a random value and picking its type.
    Type *Ty = Srcs[1]->getType();
    auto Indices = ArrayRef(Srcs).drop_front(2);
    return GetElementPtrInst::Create(Ty, Srcs[0], Indices, "G", InsertPt);
  };
  // TODO: Handle aggregates and vectors
  // TODO: Support multiple indices.
  // TODO: Try to avoid meaningless accesses.
  SourcePred sizedType(
      [](ArrayRef<Value *>, const Value *V) { return V->getType()->isSized(); },
      std::nullopt);
  return {Weight, {sizedPtrType(), sizedType, anyIntType()}, buildGEP};
}

```
- **EN**: Implements logic around `gepDescriptor`, `getType`, `ArrayRef`, `Create`, and 2 more symbols.
- **CN**: 围绕 `gepDescriptor`, `getType`, `ArrayRef`, `Create`, and 2 more symbols 实现具体逻辑。

### Lines 215-234
```cpp
static uint64_t getAggregateNumElements(Type *T) {
  assert(T->isAggregateType() && "Not a struct or array");
  if (isa<StructType>(T))
    return T->getStructNumElements();
  return T->getArrayNumElements();
}

static SourcePred validExtractValueIndex() {
  auto Pred = [](ArrayRef<Value *> Cur, const Value *V) {
    if (auto *CI = dyn_cast<ConstantInt>(V))
      if (!CI->uge(getAggregateNumElements(Cur[0]->getType())))
        return true;
    return false;
  };
  auto Make = [](ArrayRef<Value *> Cur, ArrayRef<Type *> Ts) {
    std::vector<Constant *> Result;
    auto *Int32Ty = Type::getInt32Ty(Cur[0]->getContext());
    uint64_t N = getAggregateNumElements(Cur[0]->getType());
    // Create indices at the start, end, and middle, but avoid dups.
    Result.push_back(ConstantInt::get(Int32Ty, 0));
```
- **EN**: Introduces declarations for `or`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `or` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 235-254
```cpp
    if (N > 1)
      Result.push_back(ConstantInt::get(Int32Ty, N - 1));
    if (N > 2)
      Result.push_back(ConstantInt::get(Int32Ty, N / 2));
    return Result;
  };
  return {Pred, Make};
}

OpDescriptor llvm::fuzzerop::extractValueDescriptor(unsigned Weight) {
  auto buildExtract = [](ArrayRef<Value *> Srcs,
                         BasicBlock::iterator InsertPt) {
    // TODO: It's pretty inefficient to shuffle this all through constants.
    unsigned Idx = cast<ConstantInt>(Srcs[1])->getZExtValue();
    return ExtractValueInst::Create(Srcs[0], {Idx}, "E", InsertPt);
  };
  // TODO: Should we handle multiple indices?
  return {Weight, {anyAggregateType(), validExtractValueIndex()}, buildExtract};
}

```
- **EN**: Implements logic around `push_back`, `extractValueDescriptor`, `cast`, `Create`, and 1 more symbols.
- **CN**: 围绕 `push_back`, `extractValueDescriptor`, `cast`, `Create`, and 1 more symbols 实现具体逻辑。

### Lines 255-269
```cpp
static SourcePred matchScalarInAggregate() {
  auto Pred = [](ArrayRef<Value *> Cur, const Value *V) {
    if (auto *ArrayT = dyn_cast<ArrayType>(Cur[0]->getType()))
      return V->getType() == ArrayT->getElementType();

    auto *STy = cast<StructType>(Cur[0]->getType());
    for (int I = 0, E = STy->getNumElements(); I < E; ++I)
      if (STy->getTypeAtIndex(I) == V->getType())
        return true;
    return false;
  };
  auto Make = [](ArrayRef<Value *> Cur, ArrayRef<Type *>) {
    if (auto *ArrayT = dyn_cast<ArrayType>(Cur[0]->getType()))
      return makeConstantsWithType(ArrayT->getElementType());

```
- **EN**: Implements logic around `matchScalarInAggregate`, `dyn_cast`, `getType`, `cast`, and 3 more symbols.
- **CN**: 围绕 `matchScalarInAggregate`, `dyn_cast`, `getType`, `cast`, and 3 more symbols 实现具体逻辑。

### Lines 270-289
```cpp
    std::vector<Constant *> Result;
    auto *STy = cast<StructType>(Cur[0]->getType());
    for (int I = 0, E = STy->getNumElements(); I < E; ++I)
      makeConstantsWithType(STy->getTypeAtIndex(I), Result);
    return Result;
  };
  return {Pred, Make};
}

static SourcePred validInsertValueIndex() {
  auto Pred = [](ArrayRef<Value *> Cur, const Value *V) {
    if (auto *CI = dyn_cast<ConstantInt>(V))
      if (CI->getBitWidth() == 32) {
        Type *Indexed = ExtractValueInst::getIndexedType(Cur[0]->getType(),
                                                         CI->getZExtValue());
        return Indexed == Cur[1]->getType();
      }
    return false;
  };
  auto Make = [](ArrayRef<Value *> Cur, ArrayRef<Type *> Ts) {
```
- **EN**: Implements logic around `cast`, `getNumElements`, `makeConstantsWithType`, `validInsertValueIndex`, and 5 more symbols.
- **CN**: 围绕 `cast`, `getNumElements`, `makeConstantsWithType`, `validInsertValueIndex`, and 5 more symbols 实现具体逻辑。

### Lines 290-303
```cpp
    std::vector<Constant *> Result;
    auto *Int32Ty = Type::getInt32Ty(Cur[0]->getContext());
    auto *BaseTy = Cur[0]->getType();
    int I = 0;
    while (Type *Indexed = ExtractValueInst::getIndexedType(BaseTy, I)) {
      if (Indexed == Cur[1]->getType())
        Result.push_back(ConstantInt::get(Int32Ty, I));
      ++I;
    }
    return Result;
  };
  return {Pred, Make};
}

```
- **EN**: Implements logic around `getInt32Ty`, `getType`, `getIndexedType`, `push_back`.
- **CN**: 围绕 `getInt32Ty`, `getType`, `getIndexedType`, `push_back` 实现具体逻辑。

### Lines 304-315
```cpp
OpDescriptor llvm::fuzzerop::insertValueDescriptor(unsigned Weight) {
  auto buildInsert = [](ArrayRef<Value *> Srcs, BasicBlock::iterator InsertPt) {
    // TODO: It's pretty inefficient to shuffle this all through constants.
    unsigned Idx = cast<ConstantInt>(Srcs[2])->getZExtValue();
    return InsertValueInst::Create(Srcs[0], Srcs[1], {Idx}, "I", InsertPt);
  };
  return {
      Weight,
      {anyAggregateType(), matchScalarInAggregate(), validInsertValueIndex()},
      buildInsert};
}

```
- **EN**: Implements logic around `insertValueDescriptor`, `cast`, `Create`, `anyAggregateType`.
- **CN**: 围绕 `insertValueDescriptor`, `cast`, `Create`, `anyAggregateType` 实现具体逻辑。

### Lines 316-334
```cpp
OpDescriptor llvm::fuzzerop::extractElementDescriptor(unsigned Weight) {
  auto buildExtract = [](ArrayRef<Value *> Srcs,
                         BasicBlock::iterator InsertPt) {
    return ExtractElementInst::Create(Srcs[0], Srcs[1], "E", InsertPt);
  };
  // TODO: Try to avoid undefined accesses.
  return {Weight, {anyVectorType(), anyIntType()}, buildExtract};
}

OpDescriptor llvm::fuzzerop::insertElementDescriptor(unsigned Weight) {
  auto buildInsert = [](ArrayRef<Value *> Srcs, BasicBlock::iterator InsertPt) {
    return InsertElementInst::Create(Srcs[0], Srcs[1], Srcs[2], "I", InsertPt);
  };
  // TODO: Try to avoid undefined accesses.
  return {Weight,
          {anyVectorType(), matchScalarOfFirstType(), anyIntType()},
          buildInsert};
}

```
- **EN**: Implements logic around `extractElementDescriptor`, `Create`, `anyVectorType`, `insertElementDescriptor`.
- **CN**: 围绕 `extractElementDescriptor`, `Create`, `anyVectorType`, `insertElementDescriptor` 实现具体逻辑。

### Lines 335-349
```cpp
static SourcePred validShuffleVectorIndex() {
  auto Pred = [](ArrayRef<Value *> Cur, const Value *V) {
    return ShuffleVectorInst::isValidOperands(Cur[0], Cur[1], V);
  };
  auto Make = [](ArrayRef<Value *> Cur, ArrayRef<Type *> Ts) {
    auto *FirstTy = cast<VectorType>(Cur[0]->getType());
    auto *Int32Ty = Type::getInt32Ty(Cur[0]->getContext());
    // TODO: It's straighforward to make up reasonable values, but listing them
    // exhaustively would be insane. Come up with a couple of sensible ones.
    return std::vector<Constant *>{
        PoisonValue::get(VectorType::get(Int32Ty, FirstTy->getElementCount()))};
  };
  return {Pred, Make};
}

```
- **EN**: Implements logic around `validShuffleVectorIndex`, `isValidOperands`, `cast`, `getInt32Ty`, and 1 more symbols.
- **CN**: 围绕 `validShuffleVectorIndex`, `isValidOperands`, `cast`, `getInt32Ty`, and 1 more symbols 实现具体逻辑。

### Lines 350-358
```cpp
OpDescriptor llvm::fuzzerop::shuffleVectorDescriptor(unsigned Weight) {
  auto buildShuffle = [](ArrayRef<Value *> Srcs,
                         BasicBlock::iterator InsertPt) {
    return new ShuffleVectorInst(Srcs[0], Srcs[1], Srcs[2], "S", InsertPt);
  };
  return {Weight,
          {anyVectorType(), matchFirstType(), validShuffleVectorIndex()},
          buildShuffle};
}
```
- **EN**: Implements logic around `shuffleVectorDescriptor`, `ShuffleVectorInst`, `anyVectorType`.
- **CN**: 围绕 `shuffleVectorDescriptor`, `ShuffleVectorInst`, `anyVectorType` 实现具体逻辑。

## Key Concepts / 关键概念

- **IR mutation / IR 变异**:
  - **EN**: Builds randomized but valid IR edits for fuzzing workflows.
  - **CN**: 为模糊测试流程构造随机但有效的 IR 变换。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/FuzzMutate/Operations.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`
- **Subsystem categories / 子系统类别**: LLVM IR core abstractions / LLVM IR 核心抽象 (4)
