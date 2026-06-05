# InjectPointerSigningFixups.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/ExpressionParser/Clang/InjectPointerSigningFixups.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: On arm64e, Clang emits ConstantPtrAuth expressions in global initializers to represent signed pointers. These are normally resolved by the dynamic linker, but LLDB's JIT does not run the linker, so they must be resolved manually. This pass replaces each ConstantPtrAuth in a global initializer with the unsigned pointer and emits a constructor function that signs the pointer at runtime using the ptrauth intrinsics.
  - **CN**: 实现与 `InjectPointerSigningFixups` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-21
```cpp
//
// On arm64e, Clang emits ConstantPtrAuth expressions in global initializers
// to represent signed pointers. These are normally resolved by the dynamic
// linker, but LLDB's JIT does not run the linker, so they must be resolved
// manually. This pass replaces each ConstantPtrAuth in a global initializer
// with the unsigned pointer and emits a constructor function that signs the
// pointer at runtime using the ptrauth intrinsics.
//
// Example: given "static int (*fp)(int, int) = &mul;", Clang emits:
//
//   @fp = internal global ptr ptrauth (ptr @mul, i32 0)
//
// This pass transforms it into:
//
```
- **EN**: Documents the surrounding declarations, example workflow, or design constraints for this LLDB component.
- **CN**: 说明周边声明、示例流程，或记录该 LLDB 组件的设计约束。

### Lines 22-35
```cpp
//   @fp = internal global ptr @mul
//   @llvm.global_ctors = appending global [1 x { i32, ptr, ptr }]
//       [{ i32, ptr, ptr } { i32 0, ptr @ptrauth.sign, ptr null }]
//
//   define internal void @ptrauth.sign() {
//     %1 = load ptr, ptr @fp, align 8
//     %2 = ptrtoint ptr %1 to i64
//     %3 = call i64 @llvm.ptrauth.sign(i64 %2, i32 0, i64 0)
//     %4 = inttoptr i64 %3 to ptr
//     store ptr %4, ptr @fp, align 8
//     ret void
//   }
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the surrounding declarations, example workflow, or design constraints for this LLDB component.
- **CN**: 说明周边声明、示例流程，或记录该 LLDB 组件的设计约束。

### Lines 36-45
```cpp

#include "InjectPointerSigningFixups.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/TargetParser/Triple.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `InjectPointerSigningFixups.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `InjectPointerSigningFixups.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`。

### Lines 46-53
```cpp
using namespace llvm;

namespace {
struct ExprStep {
  ConstantExpr *CE;
  unsigned OperandIdx;
};

```
- **EN**: Introduces declarations for `ExprStep`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ExprStep` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 54-60
```cpp
struct PtrAuthFixup {
  GlobalVariable *GV;
  ConstantPtrAuth *CPA;
  /// ConstantAggregate types are walekd via GEP indices.
  SmallVector<unsigned> GEPPath;
  /// ConstantExpr types are traversed via ExprStep (ConstantExpr + Operand
  /// index).
```
- **EN**: Introduces declarations for `PtrAuthFixup`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `PtrAuthFixup` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 61-69
```cpp
  SmallVector<ExprStep> ExprPath;
  PtrAuthFixup(GlobalVariable *GV, ConstantPtrAuth *CPA,
               const SmallVectorImpl<unsigned> &GEPPath,
               const SmallVectorImpl<ExprStep> &ExprPath)
      : GV(GV), CPA(CPA), GEPPath(GEPPath.begin(), GEPPath.end()),
        ExprPath(ExprPath.begin(), ExprPath.end()) {}
};
} // namespace

```
- **EN**: Implements logic around `PtrAuthFixup`, `GV`, `ExprPath`.
- **CN**: 围绕 `PtrAuthFixup`, `GV`, `ExprPath` 实现具体逻辑。

### Lines 70-83
```cpp
/// Recursively walk a constant looking for ConstantPtrAuth expressions.
static void findPtrAuth(Constant *C, GlobalVariable &GV,
                        SmallVectorImpl<unsigned> &GEPPath,
                        SmallVectorImpl<ExprStep> &ExprPath,
                        SmallVectorImpl<PtrAuthFixup> &Fixups) {
  if (auto *CPA = dyn_cast<ConstantPtrAuth>(C)) {
    Fixups.emplace_back(&GV, CPA, GEPPath, ExprPath);
    return;
  }
  if (isa<ConstantAggregate>(C)) {
    for (unsigned I = 0, E = C->getNumOperands(); I != E; ++I) {
      if (auto *COp = dyn_cast<Constant>(C->getOperand(I))) {
        GEPPath.push_back(I);
        findPtrAuth(COp, GV, GEPPath, ExprPath, Fixups);
```
- **EN**: Implements logic around `findPtrAuth`, `dyn_cast`, `emplace_back`, `isa`, and 2 more symbols.
- **CN**: 围绕 `findPtrAuth`, `dyn_cast`, `emplace_back`, `isa`, and 2 more symbols 实现具体逻辑。

### Lines 84-97
```cpp
        GEPPath.pop_back();
      }
    }
    return;
  }

  if (auto *CE = dyn_cast<ConstantExpr>(C)) {
    for (unsigned I = 0, E = C->getNumOperands(); I != E; ++I) {
      if (auto *COp = dyn_cast<Constant>(C->getOperand(I))) {
        ExprPath.push_back({CE, I});
        findPtrAuth(COp, GV, GEPPath, ExprPath, Fixups);
        ExprPath.pop_back();
      }
    }
```
- **EN**: Implements logic around `pop_back`, `dyn_cast`, `getNumOperands`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `pop_back`, `dyn_cast`, `getNumOperands`, `push_back`, and 1 more symbols 实现具体逻辑。

### Lines 98-108
```cpp
  }
}

namespace lldb_private {

Error InjectPointerSigningFixupCode(llvm::Module &M,
                                    ExecutionPolicy execution_policy) {
  // If we cannot execute fixups, don't insert them.
  if (execution_policy == eExecutionPolicyNever)
    return Error::success();

```
- **EN**: Introduces declarations for `lldb_private`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `lldb_private` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 109-122
```cpp
  llvm::Triple T(M.getTargetTriple());

  // Bail out if we don't need pointer signing fixups.
  if (!T.isArm64e())
    return Error::success();

  // Collect all ConstantPtrAuth expressions in global initializers.
  SmallVector<PtrAuthFixup> Fixups;
  for (auto &G : M.globals()) {
    if (!G.hasInitializer())
      continue;
    SmallVector<unsigned> GEPPath;
    SmallVector<ExprStep> ExprPath;
    findPtrAuth(G.getInitializer(), G, GEPPath, ExprPath, Fixups);
```
- **EN**: Implements logic around `T`, `isArm64e`, `success`, `globals`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `T`, `isArm64e`, `success`, `globals`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 123-136
```cpp
  }

  if (Fixups.empty())
    return Error::success();

  // Set up types and intrinsics.
  auto &Ctx = M.getContext();
  Type *Int32Ty = Type::getInt32Ty(Ctx);
  Type *IntPtrTy = Type::getInt64Ty(Ctx);
  Function *BlendIntrinsic =
      Intrinsic::getOrInsertDeclaration(&M, Intrinsic::ptrauth_blend);
  Function *SignIntrinsic =
      Intrinsic::getOrInsertDeclaration(&M, Intrinsic::ptrauth_sign);

```
- **EN**: Implements logic around `empty`, `success`, `getContext`, `getInt32Ty`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `empty`, `success`, `getContext`, `getInt32Ty`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 137-143
```cpp
  // Create the fixup function.
  Function *FixupFn =
      Function::Create(FunctionType::get(Type::getVoidTy(Ctx), false),
                       GlobalValue::InternalLinkage, "ptrauth.sign", &M);
  FixupFn->insert(FixupFn->end(), BasicBlock::Create(Ctx));
  IRBuilder<> B(&FixupFn->back());

```
- **EN**: Implements logic around `Create`, `insert`, `B`.
- **CN**: 围绕 `Create`, `insert`, `B` 实现具体逻辑。

### Lines 144-153
```cpp
  for (auto &Fixup : Fixups) {
    GlobalVariable *GV = Fixup.GV;
    ConstantPtrAuth *CPA = Fixup.CPA;

    // Null pointers must remain zero.
    if (isa<ConstantPointerNull>(CPA->getPointer())) {
      CPA->replaceAllUsesWith(CPA->getPointer());
      continue;
    }

```
- **EN**: Implements logic around `isa`, `replaceAllUsesWith`.
- **CN**: 围绕 `isa`, `replaceAllUsesWith` 实现具体逻辑。

### Lines 154-166
```cpp
    // Build a GEP to the location of the ConstantPtrAuth (or the expression
    // path to the ConstantPtrAuth) within the global.
    Value *Loc;
    if (Fixup.GEPPath.empty()) {
      Loc = GV;
    } else {
      SmallVector<Value *> GEPValues;
      GEPValues.push_back(ConstantInt::get(Int32Ty, 0));
      for (unsigned Idx : Fixup.GEPPath)
        GEPValues.push_back(ConstantInt::get(Int32Ty, Idx));
      Loc = B.CreateGEP(GV->getValueType(), GV, GEPValues);
    }

```
- **EN**: Implements logic around `empty`, `push_back`, `CreateGEP`; this block supports expression parsing, wrapping, or debug-time code generation.
- **CN**: 围绕 `empty`, `push_back`, `CreateGEP` 实现具体逻辑；该代码块支持表达式解析、包装或调试期代码生成。

### Lines 167-174
```cpp
    Type *PtrTy = CPA->getType();

    // Compute the discriminator, blending with the address if needed.
    Value *Disc = CPA->getDiscriminator();
    if (CPA->hasAddressDiscriminator())
      Disc = B.CreateCall(BlendIntrinsic,
                          {B.CreatePointerCast(Loc, IntPtrTy), Disc});

```
- **EN**: Implements logic around `getType`, `getDiscriminator`, `hasAddressDiscriminator`, `CreateCall`, and 1 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `getType`, `getDiscriminator`, `hasAddressDiscriminator`, `CreateCall`, and 1 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 175-182
```cpp
    if (!Fixup.ExprPath.empty()) {
      // The CPA is wrapped in a ConstantExpr chain. Sign the CPA's pointer
      // directly and re-evaluate the expr chain.
      Value *SignedPtr = B.CreateCall(
          SignIntrinsic, {B.CreatePointerCast(CPA->getPointer(), IntPtrTy),
                          CPA->getKey(), Disc});
      Value *Result = B.CreateIntToPtr(SignedPtr, PtrTy);

```
- **EN**: Implements logic around `empty`, `CreateCall`, `CreatePointerCast`, `getKey`, and 1 more symbols.
- **CN**: 围绕 `empty`, `CreateCall`, `CreatePointerCast`, `getKey`, and 1 more symbols 实现具体逻辑。

### Lines 183-196
```cpp
      for (auto &Step : llvm::reverse(Fixup.ExprPath)) {
        Instruction *I = Step.CE->getAsInstruction();
        I->setOperand(Step.OperandIdx, Result);
        B.Insert(I);
        Result = I;
      }
      B.CreateStore(Result, Loc);
    } else {
      // There is no expression chain. Load and sign the pointer directly.
      Value *RawPtr = B.CreateLoad(PtrTy, Loc);
      Value *SignedPtr =
          B.CreateCall(SignIntrinsic, {B.CreatePointerCast(RawPtr, IntPtrTy),
                                       CPA->getKey(), Disc});
      B.CreateStore(B.CreateBitOrPointerCast(SignedPtr, PtrTy), Loc);
```
- **EN**: Implements logic around `reverse`, `getAsInstruction`, `setOperand`, `Insert`, and 4 more symbols; this block supports expression parsing, wrapping, or debug-time code generation; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `reverse`, `getAsInstruction`, `setOperand`, `Insert`, and 4 more symbols 实现具体逻辑；该代码块支持表达式解析、包装或调试期代码生成，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 197-204
```cpp
    }
    // Replace the ConstantPtrAuth in the initializer with the unsigned pointer.
    CPA->replaceAllUsesWith(CPA->getPointer());
  }

  // Close off the fixup function.
  B.CreateRetVoid();

```
- **EN**: Implements logic around `replaceAllUsesWith`, `CreateRetVoid`.
- **CN**: 围绕 `replaceAllUsesWith`, `CreateRetVoid` 实现具体逻辑。

### Lines 205-212
```cpp
  // Update the global ctors list to call the pointer fixup function first.
  auto *UInt8PtrTy = PointerType::getUnqual(Ctx);
  StructType *CtorType =
      StructType::get(Ctx, {Int32Ty, FixupFn->getType(), UInt8PtrTy});
  Constant *PtrFixupCtor =
      ConstantStruct::get(CtorType, {ConstantInt::get(Int32Ty, 0), FixupFn,
                                     Constant::getNullValue(UInt8PtrTy)});

```
- **EN**: Implements logic around `getUnqual`, `get`, `getNullValue`.
- **CN**: 围绕 `getUnqual`, `get`, `getNullValue` 实现具体逻辑。

### Lines 213-226
```cpp
  const char *LLVMGlobalCtorsName = "llvm.global_ctors";
  GlobalVariable *OldCtorList = M.getNamedGlobal(LLVMGlobalCtorsName);
  SmallVector<Constant *> CtorListArgs;
  CtorListArgs.push_back(PtrFixupCtor);

  if (OldCtorList) {
    // If the old ctors list has any uses then bail out: we do not know how to
    // rewrite them.
    if (OldCtorList->getNumUses() != 0) {
      std::string ErrStr;
      raw_string_ostream S(ErrStr);
      S << "Global ctors variable has users, so can not be rewritten to "
           "include pointer fixups: '"
        << *OldCtorList << "'";
```
- **EN**: Implements logic around `getNamedGlobal`, `push_back`, `getNumUses`, `S`.
- **CN**: 围绕 `getNamedGlobal`, `push_back`, `getNumUses`, `S` 实现具体逻辑。

### Lines 227-233
```cpp
      return make_error<StringError>(S.str(), inconvertibleErrorCode());
    }

    for (auto &Op : OldCtorList->getInitializer()->operands())
      CtorListArgs.push_back(cast<Constant>(Op.get()));
  }

```
- **EN**: Implements logic around `make_error`, `getInitializer`, `push_back`.
- **CN**: 围绕 `make_error`, `getInitializer`, `push_back` 实现具体逻辑。

### Lines 234-245
```cpp
  ArrayType *CtorListType = ArrayType::get(CtorType, CtorListArgs.size());
  Constant *CtorListInit = ConstantArray::get(CtorListType, CtorListArgs);

  GlobalVariable *NewCtorList = new GlobalVariable(
      M, CtorListType, false, GlobalValue::AppendingLinkage, CtorListInit);

  if (OldCtorList) {
    NewCtorList->takeName(OldCtorList);
    OldCtorList->eraseFromParent();
  } else
    NewCtorList->setName(LLVMGlobalCtorsName);

```
- **EN**: Implements logic around `get`, `GlobalVariable`, `takeName`, `eraseFromParent`, and 1 more symbols.
- **CN**: 围绕 `get`, `GlobalVariable`, `takeName`, `eraseFromParent`, and 1 more symbols 实现具体逻辑。

### Lines 246-249
```cpp
  return Error::success();
}

} // namespace lldb_private
```
- **EN**: Implements logic around `success`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `success` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **User expressions / 用户表达式**:
  - **EN**: Wraps and evaluates source snippets inside the debuggee context.
  - **CN**: 在被调试进程上下文中包装并求值源码片段。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `InjectPointerSigningFixups.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h`, `llvm/Support/raw_ostream.h`, `llvm/TargetParser/Triple.h`
- **Subsystem categories / 子系统类别**: shared LLVM infrastructure / 共享 LLVM 基础设施 (5), LLVM support-library helpers / LLVM Support 库辅助组件 (1), LLVM target and ABI parsing helpers / LLVM 目标与 ABI 解析辅助组件 (1)
