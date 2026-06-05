# TargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/TargetMachine.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines the target machine, data layout, pass pipeline hooks, and subtarget creation logic.
  - **CN**: 定义目标机器、数据布局、Pass 流水线钩子以及子目标创建逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- TargetMachine.cpp - General Target Information ---------------------==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the general parts of a Target machine.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-29
```cpp

#include "llvm/Target/TargetMachine.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/Mangler.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/CodeGen.h"
#include "llvm/Target/TargetLoweringObjectFile.h"
using namespace llvm;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Target/TargetMachine.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalValue.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Target/TargetMachine.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalValue.h`。

### Lines 30-38
```cpp
cl::opt<bool> llvm::NoKernelInfoEndLTO(
    "no-kernel-info-end-lto",
    cl::desc("remove the kernel-info pass at the end of the full LTO pipeline"),
    cl::init(false), cl::Hidden);

//---------------------------------------------------------------------------
// TargetMachine Class
//

```
- **EN**: Implements logic around `NoKernelInfoEndLTO`, `desc`, `init`.
- **CN**: 围绕 `NoKernelInfoEndLTO`, `desc`, `init` 实现具体逻辑。

### Lines 39-48
```cpp
TargetMachine::TargetMachine(const Target &T, StringRef DataLayoutString,
                             const Triple &TT, StringRef CPU, StringRef FS,
                             const TargetOptions &Options)
    : TheTarget(T), DL(DataLayoutString), TargetTriple(TT),
      TargetCPU(std::string(CPU)), TargetFS(std::string(FS)), AsmInfo(nullptr),
      MRI(nullptr), MII(nullptr), STI(nullptr), RequireStructuredCFG(false),
      O0WantsFastISel(false), Options(Options) {}

TargetMachine::~TargetMachine() = default;

```
- **EN**: Implements logic around `TargetMachine`, `TheTarget`, `TargetCPU`, `MRI`, ....
- **CN**: 围绕 `TargetMachine`, `TheTarget`, `TargetCPU`, `MRI`, ... 实现具体逻辑。

### Lines 49-59
```cpp
Expected<std::unique_ptr<MCStreamer>>
TargetMachine::createMCStreamer(raw_pwrite_stream &Out,
                                raw_pwrite_stream *DwoOut,
                                CodeGenFileType FileType, MCContext &Ctx) {
  return nullptr;
}

bool TargetMachine::isLargeGlobalValue(const GlobalValue *GVal) const {
  if (getTargetTriple().getArch() != Triple::x86_64)
    return false;

```
- **EN**: Implements logic around `createMCStreamer`, `isLargeGlobalValue`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createMCStreamer`, `isLargeGlobalValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 60-71
```cpp
  // Remaining logic below is ELF-specific. For other object file formats where
  // the large code model is mostly used for JIT compilation, just look at the
  // code model.
  if (!getTargetTriple().isOSBinFormatELF())
    return getCodeModel() == CodeModel::Large;

  auto *GO = GVal->getAliaseeObject();

  // Be conservative if we can't find an underlying GlobalObject.
  if (!GO)
    return true;

```
- **EN**: Implements logic around `getCodeModel`, `getAliaseeObject`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getCodeModel`, `getAliaseeObject` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 72-88
```cpp
  auto *GV = dyn_cast<GlobalVariable>(GO);

  auto IsPrefix = [](StringRef Name, StringRef Prefix) {
    return Name.consume_front(Prefix) && (Name.empty() || Name[0] == '.');
  };

  // Functions/GlobalIFuncs are only large under the large code model.
  if (!GV) {
    // Handle explicit sections as we do for GlobalVariables with an explicit
    // section, see comments below.
    if (GO->hasSection()) {
      StringRef Name = GO->getSection();
      return IsPrefix(Name, ".ltext");
    }
    return getCodeModel() == CodeModel::Large;
  }

```
- **EN**: Implements logic around `dyn_cast<GlobalVariable>`, `consume_front`, `getSection`, `IsPrefix`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `dyn_cast<GlobalVariable>`, `consume_front`, `getSection`, `IsPrefix`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 89-100
```cpp
  if (GV->isThreadLocal())
    return false;

  // For x86-64, we treat an explicit GlobalVariable small code model to mean
  // that the global should be placed in a small section, and ditto for large.
  if (auto CM = GV->getCodeModel()) {
    if (*CM == CodeModel::Small)
      return false;
    if (*CM == CodeModel::Large)
      return true;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 101-110
```cpp
  // Treat all globals in explicit sections as small, except for the standard
  // large sections of .lbss, .ldata, .lrodata. This reduces the risk of linking
  // together small and large sections, resulting in small references to large
  // data sections. The code model attribute overrides this above.
  if (GV->hasSection()) {
    StringRef Name = GV->getSection();
    return IsPrefix(Name, ".lbss") || IsPrefix(Name, ".ldata") ||
           IsPrefix(Name, ".lrodata");
  }

```
- **EN**: Implements logic around `getSection`, `IsPrefix`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getSection`, `IsPrefix` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 111-128
```cpp
  // Respect large data threshold for medium and large code models.
  if (getCodeModel() == CodeModel::Medium ||
      getCodeModel() == CodeModel::Large) {
    if (!GV->getValueType()->isSized())
      return true;
    // Linker defined start/stop symbols can point to arbitrary points in the
    // binary, so treat them as large.
    if (GV->isDeclaration() && (GV->getName() == "__ehdr_start" ||
                                GV->getName().starts_with("__start_") ||
                                GV->getName().starts_with("__stop_")))
      return true;
    // Linkers do not currently support PT_GNU_RELRO for SHF_X86_64_LARGE
    // sections; that would require the linker to emit more than one
    // PT_GNU_RELRO because large sections are discontiguous by design, and most
    // ELF dynamic loaders do not support that (bionic appears to support it but
    // glibc/musl/FreeBSD/NetBSD/OpenBSD appear not to). With current linkers
    // these sections will end up in .ldata which results in silently disabling
    // RELRO. If this ever gets supported by downstream components in the future
```
- **EN**: Implements logic around `getCodeModel`, `getName`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getCodeModel`, `getName` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 129-139
```cpp
    // we could add an opt-in flag for moving these sections to .ldata.rel.ro
    // which would trigger the creation of a second PT_GNU_RELRO.
    if (!GV->isDeclarationForLinker() &&
        TargetLoweringObjectFile::getKindForGlobal(GV, *this)
            .isReadOnlyWithRel())
      return false;
    const DataLayout &DL = GV->getDataLayout();
    uint64_t Size = GV->getGlobalSize(DL);
    return Size == 0 || Size > LargeDataThreshold;
  }

```
- **EN**: Implements logic around `getKindForGlobal`, `isReadOnlyWithRel`, `getDataLayout`, `getGlobalSize`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getKindForGlobal`, `isReadOnlyWithRel`, `getDataLayout`, `getGlobalSize` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 140-148
```cpp
  return false;
}

bool TargetMachine::isPositionIndependent() const {
  return getRelocationModel() == Reloc::PIC_;
}

/// Reset the target options based on the function's attributes.
/// setFunctionAttributes should have made the raw attribute value consistent
```
- **EN**: Implements logic around `isPositionIndependent`, `getRelocationModel`; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `isPositionIndependent`, `getRelocationModel` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 149-160
```cpp
/// with the command line flag if used.
//
// FIXME: This function needs to go away for a number of reasons:
// a) global state on the TargetMachine is terrible in general,
// b) these target options should be passed only on the function
//    and not on the TargetMachine (via TargetOptions) at all.
void TargetMachine::resetTargetOptions(const Function &F) const {
#define RESET_OPTION(X, Y)                                              \
  do {                                                                  \
    Options.X = F.getFnAttribute(Y).getValueAsBool();     \
  } while (0)

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 161-178
```cpp
  RESET_OPTION(NoSignedZerosFPMath, "no-signed-zeros-fp-math");
}

/// Returns the code generation relocation model. The choices are static, PIC,
/// and dynamic-no-pic.
Reloc::Model TargetMachine::getRelocationModel() const { return RM; }

uint64_t TargetMachine::getMaxCodeSize() const {
  switch (getCodeModel()) {
  case CodeModel::Tiny:
    return llvm::maxUIntN(10);
  case CodeModel::Small:
  case CodeModel::Kernel:
  case CodeModel::Medium:
    return llvm::maxUIntN(31);
  case CodeModel::Large:
    return llvm::maxUIntN(64);
  }
```
- **EN**: Implements logic around `RESET_OPTION`, `getRelocationModel`, `getMaxCodeSize`, `maxUIntN`; this block uses `switch`-based dispatch; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `RESET_OPTION`, `getRelocationModel`, `getMaxCodeSize`, `maxUIntN` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，映射 fixup 或重定位。

### Lines 179-196
```cpp
  llvm_unreachable("Unhandled CodeModel enum");
}

/// Get the IR-specified TLS model for Var.
static TLSModel::Model getSelectedTLSModel(const GlobalValue *GV) {
  switch (GV->getThreadLocalMode()) {
  case GlobalVariable::NotThreadLocal:
    llvm_unreachable("getSelectedTLSModel for non-TLS variable");
    break;
  case GlobalVariable::GeneralDynamicTLSModel:
    return TLSModel::GeneralDynamic;
  case GlobalVariable::LocalDynamicTLSModel:
    return TLSModel::LocalDynamic;
  case GlobalVariable::InitialExecTLSModel:
    return TLSModel::InitialExec;
  case GlobalVariable::LocalExecTLSModel:
    return TLSModel::LocalExec;
  }
```
- **EN**: Implements logic around `llvm_unreachable`, `getSelectedTLSModel`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `llvm_unreachable`, `getSelectedTLSModel` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 197-214
```cpp
  llvm_unreachable("invalid TLS model");
}

bool TargetMachine::shouldAssumeDSOLocal(const GlobalValue *GV) const {
  const Triple &TT = getTargetTriple();
  Reloc::Model RM = getRelocationModel();

  // According to the llvm language reference, we should be able to
  // just return false in here if we have a GV, as we know it is
  // dso_preemptable.  At this point in time, the various IR producers
  // have not been transitioned to always produce a dso_local when it
  // is possible to do so.
  //
  // As a result we still have some logic in here to improve the quality of the
  // generated code.
  if (!GV)
    return false;

```
- **EN**: Implements logic around `llvm_unreachable`, `shouldAssumeDSOLocal`, `getTargetTriple`, `getRelocationModel`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `llvm_unreachable`, `shouldAssumeDSOLocal`, `getTargetTriple`, `getRelocationModel` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 215-223
```cpp
  // If the IR producer requested that this GV be treated as dso local, obey.
  if (GV->isDSOLocal())
    return true;

  if (TT.isOSBinFormatCOFF()) {
    // DLLImport explicitly marks the GV as external.
    if (GV->hasDLLImportStorageClass())
      return false;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 224-232
```cpp
    // On MinGW, variables that haven't been declared with DLLImport may still
    // end up automatically imported by the linker. To make this feasible,
    // don't assume the variables to be DSO local unless we actually know
    // that for sure. This only has to be done for variables; for functions
    // the linker can insert thunks for calling functions from another DLL.
    if (TT.isOSCygMing() && GV->isDeclarationForLinker() &&
        isa<GlobalVariable>(GV))
      return false;

```
- **EN**: Implements logic around `isa<GlobalVariable>`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isa<GlobalVariable>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 233-242
```cpp
    // Don't mark 'extern_weak' symbols as DSO local. If these symbols remain
    // unresolved in the link, they can be resolved to zero, which is outside
    // the current DSO.
    if (GV->hasExternalWeakLinkage())
      return false;

    // Every other GV is local on COFF.
    return true;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 243-251
```cpp
  if (TT.isOSBinFormatGOFF())
    return true;

  if (TT.isOSBinFormatMachO()) {
    if (RM == Reloc::Static)
      return true;
    return GV->isStrongDefinitionForLinker();
  }

```
- **EN**: Implements logic around `isStrongDefinitionForLinker`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `isStrongDefinitionForLinker` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 252-265
```cpp
  assert(TT.isOSBinFormatELF() || TT.isOSBinFormatWasm() ||
         TT.isOSBinFormatXCOFF());
  return false;
}

bool TargetMachine::useEmulatedTLS() const { return Options.EmulatedTLS; }
bool TargetMachine::useTLSDESC() const { return Options.EnableTLSDESC; }

TLSModel::Model TargetMachine::getTLSModel(const GlobalValue *GV) const {
  bool IsPIE = GV->getParent()->getPIELevel() != PIELevel::Default;
  Reloc::Model RM = getRelocationModel();
  bool IsSharedLibrary = RM == Reloc::PIC_ && !IsPIE;
  bool IsLocal = shouldAssumeDSOLocal(GV);

```
- **EN**: Implements logic around `assert`, `isOSBinFormatXCOFF`, `useEmulatedTLS`, `useTLSDESC`, ...; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `assert`, `isOSBinFormatXCOFF`, `useEmulatedTLS`, `useTLSDESC`, ... 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 266-278
```cpp
  TLSModel::Model Model;
  if (IsSharedLibrary) {
    if (IsLocal)
      Model = TLSModel::LocalDynamic;
    else
      Model = TLSModel::GeneralDynamic;
  } else {
    if (IsLocal)
      Model = TLSModel::LocalExec;
    else
      Model = TLSModel::InitialExec;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 279-291
```cpp
  // If the user specified a more specific model, use that.
  TLSModel::Model SelectedModel = getSelectedTLSModel(GV);
  if (SelectedModel > Model)
    return SelectedModel;

  return Model;
}

TargetTransformInfo
TargetMachine::getTargetTransformInfo(const Function &F) const {
  return TargetTransformInfo(F.getDataLayout());
}

```
- **EN**: Implements logic around `getSelectedTLSModel`, `getTargetTransformInfo`, `TargetTransformInfo`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getSelectedTLSModel`, `getTargetTransformInfo`, `TargetTransformInfo` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 292-304
```cpp
void TargetMachine::getNameWithPrefix(SmallVectorImpl<char> &Name,
                                      const GlobalValue *GV, Mangler &Mang,
                                      bool MayAlwaysUsePrivate) const {
  if (MayAlwaysUsePrivate || !GV->hasPrivateLinkage()) {
    // Simple case: If GV is not private, it is not important to find out if
    // private labels are legal in this case or not.
    Mang.getNameWithPrefix(Name, GV, false);
    return;
  }
  const TargetLoweringObjectFile *TLOF = getObjFileLowering();
  TLOF->getNameWithPrefix(Name, GV, *this);
}

```
- **EN**: Implements logic around `getNameWithPrefix`, `getObjFileLowering`; this block applies conditional target rules.
- **CN**: 围绕 `getNameWithPrefix`, `getObjFileLowering` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 305-315
```cpp
MCSymbol *TargetMachine::getSymbol(const GlobalValue *GV) const {
  const TargetLoweringObjectFile *TLOF = getObjFileLowering();
  // XCOFF symbols could have special naming convention.
  if (MCSymbol *TargetSymbol = TLOF->getTargetSymbol(GV, *this))
    return TargetSymbol;

  SmallString<128> NameStr;
  getNameWithPrefix(NameStr, GV, TLOF->getMangler());
  return TLOF->getContext().getOrCreateSymbol(NameStr);
}

```
- **EN**: Implements logic around `getSymbol`, `getObjFileLowering`, `getNameWithPrefix`, `getContext`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getSymbol`, `getObjFileLowering`, `getNameWithPrefix`, `getContext` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 316-330
```cpp
TargetIRAnalysis TargetMachine::getTargetIRAnalysis() const {
  // Since Analysis can't depend on Target, use a std::function to invert the
  // dependency.
  return TargetIRAnalysis(
      [this](const Function &F) { return this->getTargetTransformInfo(F); });
}

std::pair<int, int> TargetMachine::parseBinutilsVersion(StringRef Version) {
  if (Version == "none")
    return {INT_MAX, INT_MAX}; // Make binutilsIsAtLeast() return true.
  std::pair<int, int> Ret;
  if (!Version.consumeInteger(10, Ret.first) && Version.consume_front("."))
    Version.consumeInteger(10, Ret.second);
  return Ret;
}
```
- **EN**: Implements logic around `getTargetIRAnalysis`, `TargetIRAnalysis`, `getTargetTransformInfo`, `parseBinutilsVersion`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getTargetIRAnalysis`, `TargetIRAnalysis`, `getTargetTransformInfo`, `parseBinutilsVersion`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

## Key Concepts / 关键概念

- **Target configuration / 目标配置**:
  - **EN**: Owns data layout, passes, and per-function subtarget selection
  - **CN**: 管理数据布局、Pass 与按函数的子目标选择
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/Target/TargetMachine.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Mangler.h`, `llvm/IR/Module.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCStreamer.h` ... (+3 more)
- **LLVM subsystems / LLVM 子系统**: MC, IR, Support
