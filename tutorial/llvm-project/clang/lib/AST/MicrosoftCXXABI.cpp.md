# MicrosoftCXXABI.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/AST/MicrosoftCXXABI.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This provides C++ AST support targeting the Microsoft Visual C++ ABI.
  - **CN**: 实现 Clang AST 数据结构、语义辅助逻辑以及源码级建模支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===------- MicrosoftCXXABI.cpp - AST support for the Microsoft C++ ABI --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This provides C++ AST support targeting the Microsoft Visual C++
// ABI.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/Clang file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/Clang 标准文件头、许可证声明以及文件的高层描述。

### Lines 13-25
```cpp

#include "CXXABI.h"
#include "clang/AST/ASTContext.h"
#include "clang/AST/Attr.h"
#include "clang/AST/CXXInheritance.h"
#include "clang/AST/DeclCXX.h"
#include "clang/AST/Mangle.h"
#include "clang/AST/MangleNumberingContext.h"
#include "clang/AST/RecordLayout.h"
#include "clang/AST/Type.h"
#include "clang/Basic/DiagnosticAST.h"
#include "clang/Basic/TargetInfo.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `CXXABI.h`, `clang/AST/ASTContext.h`, `clang/AST/Attr.h`, `clang/AST/CXXInheritance.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `CXXABI.h`, `clang/AST/ASTContext.h`, `clang/AST/Attr.h`, `clang/AST/CXXInheritance.h`。

### Lines 26-37
```cpp
using namespace clang;

namespace {

/// Numbers things which need to correspond across multiple TUs.
/// Typically these are things like static locals, lambdas, or blocks.
class MicrosoftNumberingContext : public MangleNumberingContext {
  llvm::DenseMap<const Type *, unsigned> ManglingNumbers;
  unsigned LambdaManglingNumber = 0;
  unsigned StaticLocalNumber = 0;
  unsigned StaticThreadlocalNumber = 0;

```
- **EN**: Introduces declarations for `clang`, `MicrosoftNumberingContext`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `clang`, `MicrosoftNumberingContext` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 38-49
```cpp
public:
  MicrosoftNumberingContext() = default;

  unsigned getManglingNumber(const CXXMethodDecl *CallOperator) override {
    return ++LambdaManglingNumber;
  }

  unsigned getManglingNumber(const BlockDecl *BD) override {
    const Type *Ty = nullptr;
    return ++ManglingNumbers[Ty];
  }

```
- **EN**: Implements logic around `MicrosoftNumberingContext`, `getManglingNumber`; this block queries or canonicalizes Clang type-system state.
- **CN**: 围绕 `MicrosoftNumberingContext`, `getManglingNumber` 实现具体逻辑；该代码块查询或规范化 Clang 类型系统状态。

### Lines 50-60
```cpp
  unsigned getStaticLocalNumber(const VarDecl *VD) override {
    if (VD->getTLSKind())
      return ++StaticThreadlocalNumber;
    return ++StaticLocalNumber;
  }

  unsigned getManglingNumber(const VarDecl *VD,
                             unsigned MSLocalManglingNumber) override {
    return MSLocalManglingNumber;
  }

```
- **EN**: Implements logic around `getStaticLocalNumber`, `getTLSKind`, `getManglingNumber`.
- **CN**: 围绕 `getStaticLocalNumber`, `getTLSKind`, `getManglingNumber` 实现具体逻辑。

### Lines 61-70
```cpp
  unsigned getManglingNumber(const TagDecl *TD,
                             unsigned MSLocalManglingNumber) override {
    return MSLocalManglingNumber;
  }
};

class MSHIPNumberingContext : public MicrosoftNumberingContext {
  std::unique_ptr<MangleNumberingContext> DeviceCtx;

public:
```
- **EN**: Introduces declarations for `MSHIPNumberingContext`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `MSHIPNumberingContext` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 71-90
```cpp
  using MicrosoftNumberingContext::getManglingNumber;
  MSHIPNumberingContext(MangleContext *DeviceMangler) {
    DeviceCtx = createItaniumNumberingContext(DeviceMangler);
  }

  unsigned getDeviceManglingNumber(const CXXMethodDecl *CallOperator) override {
    return DeviceCtx->getManglingNumber(CallOperator);
  }

  unsigned getManglingNumber(const TagDecl *TD,
                             unsigned MSLocalManglingNumber) override {
    unsigned DeviceN = DeviceCtx->getManglingNumber(TD, MSLocalManglingNumber);
    unsigned HostN =
        MicrosoftNumberingContext::getManglingNumber(TD, MSLocalManglingNumber);
    if (DeviceN > 0xFFFF || HostN > 0xFFFF) {
      DiagnosticsEngine &Diags = TD->getASTContext().getDiagnostics();
      Diags.Report(TD->getLocation(), diag::err_ms_mangle_number_overflow);
    }
    return (DeviceN << 16) | HostN;
  }
```
- **EN**: Implements logic around `MSHIPNumberingContext`, `createItaniumNumberingContext`, `getDeviceManglingNumber`, `getManglingNumber`, and 2 more symbols; this block propagates diagnostics or recoverable errors through Clang/LLVM utilities.
- **CN**: 围绕 `MSHIPNumberingContext`, `createItaniumNumberingContext`, `getDeviceManglingNumber`, `getManglingNumber`, and 2 more symbols 实现具体逻辑；该代码块通过 Clang/LLVM 工具传播诊断信息或可恢复错误。

### Lines 91-100
```cpp
};

class MSSYCLNumberingContext : public MicrosoftNumberingContext {
  std::unique_ptr<MangleNumberingContext> DeviceCtx;

public:
  MSSYCLNumberingContext(MangleContext *DeviceMangler) {
    DeviceCtx = createItaniumNumberingContext(DeviceMangler);
  }

```
- **EN**: Introduces declarations for `MSSYCLNumberingContext`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `MSSYCLNumberingContext` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 101-114
```cpp
  unsigned getDeviceManglingNumber(const CXXMethodDecl *CallOperator) override {
    return DeviceCtx->getManglingNumber(CallOperator);
  }
};

class MicrosoftCXXABI : public CXXABI {
  ASTContext &Context;
  llvm::SmallDenseMap<CXXRecordDecl *, CXXConstructorDecl *> RecordToCopyCtor;

  llvm::SmallDenseMap<TagDecl *, DeclaratorDecl *>
      UnnamedTagDeclToDeclaratorDecl;
  llvm::SmallDenseMap<TagDecl *, TypedefNameDecl *>
      UnnamedTagDeclToTypedefNameDecl;

```
- **EN**: Introduces declarations for `MicrosoftCXXABI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `MicrosoftCXXABI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 115-133
```cpp
  // MangleContext for device numbering context, which is based on Itanium C++
  // ABI.
  std::unique_ptr<MangleContext> DeviceMangler;

public:
  MicrosoftCXXABI(ASTContext &Ctx) : Context(Ctx) {
    if (Context.getLangOpts().CUDA && Context.getAuxTargetInfo()) {
      assert(Context.getTargetInfo().getCXXABI().isMicrosoft() &&
             Context.getAuxTargetInfo()->getCXXABI().isItaniumFamily() &&
             "Unexpected combination of C++ ABIs.");
      DeviceMangler.reset(
          Context.createMangleContext(Context.getAuxTargetInfo()));
    }
    else if (Context.getLangOpts().isSYCL()) {
      DeviceMangler.reset(
          ItaniumMangleContext::create(Context, Context.getDiagnostics()));
    }
  }

```
- **EN**: Implements logic around `MicrosoftCXXABI`, `getLangOpts`, `assert`, `getAuxTargetInfo`, and 3 more symbols; this block propagates diagnostics or recoverable errors through Clang/LLVM utilities; applies ABI-sensitive symbol naming or object-model rules.
- **CN**: 围绕 `MicrosoftCXXABI`, `getLangOpts`, `assert`, `getAuxTargetInfo`, and 3 more symbols 实现具体逻辑；该代码块通过 Clang/LLVM 工具传播诊断信息或可恢复错误，并应用 ABI 敏感的符号命名或对象模型规则。

### Lines 134-143
```cpp
  MemberPointerInfo
  getMemberPointerInfo(const MemberPointerType *MPT) const override;

  CallingConv getDefaultMethodCallConv(bool isVariadic) const override {
    if (!isVariadic &&
        Context.getTargetInfo().getTriple().getArch() == llvm::Triple::x86)
      return CC_X86ThisCall;
    return Context.getTargetInfo().getDefaultCallingConv();
  }

```
- **EN**: Implements logic around `getMemberPointerInfo`, `getDefaultMethodCallConv`, `getTargetInfo`.
- **CN**: 围绕 `getMemberPointerInfo`, `getDefaultMethodCallConv`, `getTargetInfo` 实现具体逻辑。

### Lines 144-160
```cpp
  bool isNearlyEmpty(const CXXRecordDecl *RD) const override {
    llvm_unreachable("unapplicable to the MS ABI");
  }

  const CXXConstructorDecl *
  getCopyConstructorForExceptionObject(CXXRecordDecl *RD) override {
    return RecordToCopyCtor[RD];
  }

  void
  addCopyConstructorForExceptionObject(CXXRecordDecl *RD,
                                       CXXConstructorDecl *CD) override {
    assert(CD != nullptr);
    assert(RecordToCopyCtor[RD] == nullptr || RecordToCopyCtor[RD] == CD);
    RecordToCopyCtor[RD] = CD;
  }

```
- **EN**: Implements logic around `isNearlyEmpty`, `llvm_unreachable`, `getCopyConstructorForExceptionObject`, `addCopyConstructorForExceptionObject`, and 1 more symbols; this block applies ABI-sensitive symbol naming or object-model rules.
- **CN**: 围绕 `isNearlyEmpty`, `llvm_unreachable`, `getCopyConstructorForExceptionObject`, `addCopyConstructorForExceptionObject`, and 1 more symbols 实现具体逻辑；该代码块应用 ABI 敏感的符号命名或对象模型规则。

### Lines 161-174
```cpp
  void addTypedefNameForUnnamedTagDecl(TagDecl *TD,
                                       TypedefNameDecl *DD) override {
    TD = TD->getCanonicalDecl();
    DD = DD->getCanonicalDecl();
    TypedefNameDecl *&I = UnnamedTagDeclToTypedefNameDecl[TD];
    if (!I)
      I = DD;
  }

  TypedefNameDecl *getTypedefNameForUnnamedTagDecl(const TagDecl *TD) override {
    return UnnamedTagDeclToTypedefNameDecl.lookup(
        const_cast<TagDecl *>(TD->getCanonicalDecl()));
  }

```
- **EN**: Implements logic around `addTypedefNameForUnnamedTagDecl`, `getCanonicalDecl`, `getTypedefNameForUnnamedTagDecl`, `lookup`; this block maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `addTypedefNameForUnnamedTagDecl`, `getCanonicalDecl`, `getTypedefNameForUnnamedTagDecl`, `lookup` 实现具体逻辑；该代码块维护声明身份、查找或链接属性簿记。

### Lines 175-188
```cpp
  void addDeclaratorForUnnamedTagDecl(TagDecl *TD,
                                      DeclaratorDecl *DD) override {
    TD = TD->getCanonicalDecl();
    DD = cast<DeclaratorDecl>(DD->getCanonicalDecl());
    DeclaratorDecl *&I = UnnamedTagDeclToDeclaratorDecl[TD];
    if (!I)
      I = DD;
  }

  DeclaratorDecl *getDeclaratorForUnnamedTagDecl(const TagDecl *TD) override {
    return UnnamedTagDeclToDeclaratorDecl.lookup(
        const_cast<TagDecl *>(TD->getCanonicalDecl()));
  }

```
- **EN**: Implements logic around `addDeclaratorForUnnamedTagDecl`, `getCanonicalDecl`, `cast`, `getDeclaratorForUnnamedTagDecl`, and 1 more symbols; this block maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `addDeclaratorForUnnamedTagDecl`, `getCanonicalDecl`, `cast`, `getDeclaratorForUnnamedTagDecl`, and 1 more symbols 实现具体逻辑；该代码块维护声明身份、查找或链接属性簿记。

### Lines 189-198
```cpp
  std::unique_ptr<MangleNumberingContext>
  createMangleNumberingContext() const override {
    if (Context.getLangOpts().CUDA && Context.getAuxTargetInfo()) {
      assert(DeviceMangler && "Missing device mangler");
      return std::make_unique<MSHIPNumberingContext>(DeviceMangler.get());
    } else if (Context.getLangOpts().isSYCL()) {
      assert(DeviceMangler && "Missing device mangler");
      return std::make_unique<MSSYCLNumberingContext>(DeviceMangler.get());
    }

```
- **EN**: Implements logic around `createMangleNumberingContext`, `getLangOpts`, `assert`, `make_unique`.
- **CN**: 围绕 `createMangleNumberingContext`, `getLangOpts`, `assert`, `make_unique` 实现具体逻辑。

### Lines 199-218
```cpp
    return std::make_unique<MicrosoftNumberingContext>();
  }
};
}

// getNumBases() seems to only give us the number of direct bases, and not the
// total.  This function tells us if we inherit from anybody that uses MI, or if
// we have a non-primary base class, which uses the multiple inheritance model.
static bool usesMultipleInheritanceModel(const CXXRecordDecl *RD) {
  while (RD->getNumBases() > 0) {
    if (RD->getNumBases() > 1)
      return true;
    assert(RD->getNumBases() == 1);
    const CXXRecordDecl *Base =
        RD->bases_begin()->getType()->getAsCXXRecordDecl();
    if (RD->isPolymorphic() && !Base->isPolymorphic())
      return true;
    RD = Base;
  }
  return false;
```
- **EN**: Implements logic around `make_unique`, `usesMultipleInheritanceModel`, `getNumBases`, `assert`, and 2 more symbols.
- **CN**: 围绕 `make_unique`, `usesMultipleInheritanceModel`, `getNumBases`, `assert`, and 2 more symbols 实现具体逻辑。

### Lines 219-230
```cpp
}

MSInheritanceModel CXXRecordDecl::calculateInheritanceModel() const {
  if (!hasDefinition() || isParsingBaseSpecifiers())
    return MSInheritanceModel::Unspecified;
  if (getNumVBases() > 0)
    return MSInheritanceModel::Virtual;
  if (usesMultipleInheritanceModel(this))
    return MSInheritanceModel::Multiple;
  return MSInheritanceModel::Single;
}

```
- **EN**: Implements logic around `calculateInheritanceModel`, `hasDefinition`, `getNumVBases`, `usesMultipleInheritanceModel`.
- **CN**: 围绕 `calculateInheritanceModel`, `hasDefinition`, `getNumVBases`, `usesMultipleInheritanceModel` 实现具体逻辑。

### Lines 231-242
```cpp
MSInheritanceModel CXXRecordDecl::getMSInheritanceModel() const {
  MSInheritanceAttr *IA = getAttr<MSInheritanceAttr>();
  assert(IA && "Expected MSInheritanceAttr on the CXXRecordDecl!");
  return IA->getInheritanceModel();
}

bool CXXRecordDecl::nullFieldOffsetIsZero() const {
  return !inheritanceModelHasOnlyOneField(/*IsMemberFunction=*/false,
                                          getMSInheritanceModel()) ||
         (hasDefinition() && isPolymorphic());
}

```
- **EN**: Implements logic around `getMSInheritanceModel`, `getAttr`, `assert`, `getInheritanceModel`, and 3 more symbols.
- **CN**: 围绕 `getMSInheritanceModel`, `getAttr`, `assert`, `getInheritanceModel`, and 3 more symbols 实现具体逻辑。

### Lines 243-262
```cpp
MSVtorDispMode CXXRecordDecl::getMSVtorDispMode() const {
  if (MSVtorDispAttr *VDA = getAttr<MSVtorDispAttr>())
    return VDA->getVtorDispMode();
  return getASTContext().getLangOpts().getVtorDispMode();
}

// Returns the number of pointer and integer slots used to represent a member
// pointer in the MS C++ ABI.
//
// Member function pointers have the following general form;  however, fields
// are dropped as permitted (under the MSVC interpretation) by the inheritance
// model of the actual class.
//
//   struct {
//     // A pointer to the member function to call.  If the member function is
//     // virtual, this will be a thunk that forwards to the appropriate vftable
//     // slot.
//     void *FunctionPointerOrVirtualThunk;
//
//     // An offset to add to the address of the vbtable pointer after
```
- **EN**: Implements logic around `getMSVtorDispMode`, `getAttr`, `getVtorDispMode`, `getASTContext`; this block applies ABI-sensitive symbol naming or object-model rules; models C/C++ record layout and dynamic-dispatch structures.
- **CN**: 围绕 `getMSVtorDispMode`, `getAttr`, `getVtorDispMode`, `getASTContext` 实现具体逻辑；该代码块应用 ABI 敏感的符号命名或对象模型规则，并建模 C/C++ 记录布局与动态派发结构。

### Lines 263-282
```cpp
//     // (possibly) selecting the virtual base but before resolving and calling
//     // the function.
//     // Only needed if the class has any virtual bases or bases at a non-zero
//     // offset.
//     int NonVirtualBaseAdjustment;
//
//     // The offset of the vb-table pointer within the object.  Only needed for
//     // incomplete types.
//     int VBPtrOffset;
//
//     // An offset within the vb-table that selects the virtual base containing
//     // the member.  Loading from this offset produces a new offset that is
//     // added to the address of the vb-table pointer to produce the base.
//     int VirtualBaseAdjustmentOffset;
//   };
static std::pair<unsigned, unsigned>
getMSMemberPointerSlots(const MemberPointerType *MPT) {
  const CXXRecordDecl *RD = MPT->getMostRecentCXXRecordDecl();
  MSInheritanceModel Inheritance = RD->getMSInheritanceModel();
  unsigned Ptrs = 0;
```
- **EN**: Introduces declarations for `has`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `has` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 283-297
```cpp
  unsigned Ints = 0;
  if (MPT->isMemberFunctionPointer())
    Ptrs = 1;
  else
    Ints = 1;
  if (inheritanceModelHasNVOffsetField(MPT->isMemberFunctionPointer(),
                                          Inheritance))
    Ints++;
  if (inheritanceModelHasVBPtrOffsetField(Inheritance))
    Ints++;
  if (inheritanceModelHasVBTableOffsetField(Inheritance))
    Ints++;
  return std::make_pair(Ptrs, Ints);
}

```
- **EN**: Implements logic around `isMemberFunctionPointer`, `inheritanceModelHasNVOffsetField`, `inheritanceModelHasVBPtrOffsetField`, `inheritanceModelHasVBTableOffsetField`, and 1 more symbols.
- **CN**: 围绕 `isMemberFunctionPointer`, `inheritanceModelHasNVOffsetField`, `inheritanceModelHasVBPtrOffsetField`, `inheritanceModelHasVBTableOffsetField`, and 1 more symbols 实现具体逻辑。

### Lines 298-311
```cpp
CXXABI::MemberPointerInfo MicrosoftCXXABI::getMemberPointerInfo(
    const MemberPointerType *MPT) const {
  // The nominal struct is laid out with pointers followed by ints and aligned
  // to a pointer width if any are present and an int width otherwise.
  const TargetInfo &Target = Context.getTargetInfo();
  unsigned PtrSize = Target.getPointerWidth(LangAS::Default);
  unsigned IntSize = Target.getIntWidth();

  unsigned Ptrs, Ints;
  std::tie(Ptrs, Ints) = getMSMemberPointerSlots(MPT);
  MemberPointerInfo MPI;
  MPI.HasPadding = false;
  MPI.Width = Ptrs * PtrSize + Ints * IntSize;

```
- **EN**: Introduces declarations for `is`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `is` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 312-321
```cpp
  // When MSVC does x86_32 record layout, it aligns aggregate member pointers to
  // 8 bytes.  However, __alignof usually returns 4 for data memptrs and 8 for
  // function memptrs.
  if (Ptrs + Ints > 1 && Target.getTriple().isArch32Bit())
    MPI.Align = 64;
  else if (Ptrs)
    MPI.Align = Target.getPointerAlign(LangAS::Default);
  else
    MPI.Align = Target.getIntAlign();

```
- **EN**: Implements logic around `getTriple`, `getPointerAlign`, `getIntAlign`.
- **CN**: 围绕 `getTriple`, `getPointerAlign`, `getIntAlign` 实现具体逻辑。

### Lines 322-331
```cpp
  if (Target.getTriple().isArch64Bit()) {
    MPI.Width = llvm::alignTo(MPI.Width, MPI.Align);
    MPI.HasPadding = MPI.Width != (Ptrs * PtrSize + Ints * IntSize);
  }
  return MPI;
}

CXXABI *clang::CreateMicrosoftCXXABI(ASTContext &Ctx) {
  return new MicrosoftCXXABI(Ctx);
}
```
- **EN**: Implements logic around `getTriple`, `alignTo`, `CreateMicrosoftCXXABI`, `MicrosoftCXXABI`.
- **CN**: 围绕 `getTriple`, `alignTo`, `CreateMicrosoftCXXABI`, `MicrosoftCXXABI` 实现具体逻辑。

## Key Concepts / 关键概念

- **Clang AST infrastructure / Clang AST 基础设施**:
  - **EN**: Explains how Clang stores and manipulates source-level syntax and semantic entities.
  - **CN**: 说明 Clang 如何存储并操作源码级语法与语义实体。
- **AST context ownership / AST 上下文所有权**:
  - **EN**: Centralizes allocation, uniquing, and lifetime management for AST objects.
  - **CN**: 集中管理 AST 对象的分配、唯一化与生命周期。
- **Declaration hierarchy / 声明层次结构**:
  - **EN**: Covers declaration nodes, redeclaration chains, and lookup-visible entities.
  - **CN**: 涵盖声明节点、重声明链以及参与查找的实体。
- **Type-system modeling / 类型系统建模**:
  - **EN**: Captures canonical types, qualifiers, and source-facing type sugar.
  - **CN**: 刻画规范类型、限定符以及面向源码的类型语法糖。
- **Attributes / 属性机制**:
  - **EN**: Stores source-level attributes and generated metadata attached to AST entities.
  - **CN**: 存储附着在 AST 实体上的源码属性与生成元数据。
- **ABI name mangling / ABI 名字修饰**:
  - **EN**: Translates AST entities into linker-visible symbol names for different ABIs.
  - **CN**: 把 AST 实体翻译成不同 ABI 下链接器可见的符号名。
- **Record layout / 记录布局**:
  - **EN**: Computes field offsets, alignment, and object representation for records.
  - **CN**: 计算记录类型的字段偏移、对齐以及对象表示。
- **Interpreter execution / 解释执行**:
  - **EN**: Executes AST-driven constant evaluation using interpreter-like state machines.
  - **CN**: 使用解释器式状态机执行基于 AST 的常量求值。
- **Field layout / 字段布局**:
  - **EN**: Computes offsets, alignment, and packing decisions for records.
  - **CN**: 计算记录类型的偏移、对齐与打包决策。
- **Interpreter state / 解释器状态**:
  - **EN**: Keeps stacks, frames, and execution state for constexpr evaluation.
  - **CN**: 为 constexpr 求值维护栈、帧以及执行状态。

## Dependencies / 依赖关系

- **Direct LLVM/Clang includes / 直接的 LLVM/Clang 包含**: `CXXABI.h`, `clang/AST/ASTContext.h`, `clang/AST/Attr.h`, `clang/AST/CXXInheritance.h`, `clang/AST/DeclCXX.h`, `clang/AST/Mangle.h`, `clang/AST/MangleNumberingContext.h`, `clang/AST/RecordLayout.h`, `clang/AST/Type.h`, `clang/Basic/DiagnosticAST.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: Clang AST node definitions and semantic data structures / Clang AST 节点定义与语义数据结构 (8), basic Clang facilities such as source locations, identifiers, and diagnostics / Clang 基础设施，例如源码位置、标识符与诊断 (2)
