# TargetLoweringObjectFile.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/TargetLoweringObjectFile.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements shared object-file lowering helpers for target backends.
  - **CN**: 实现各目标后端共用的目标文件 lowering 辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- llvm/Target/TargetLoweringObjectFile.cpp - Object File Info -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements classes used to handle lowerings specific to common
// object file formats.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 13-30
```cpp

#include "llvm/Target/TargetLoweringObjectFile.h"
#include "llvm/BinaryFormat/Dwarf.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/Mangler.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/SectionKind.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Target/TargetMachine.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Target/TargetLoweringObjectFile.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Target/TargetLoweringObjectFile.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`。

### Lines 31-39
```cpp
#include "llvm/Target/TargetOptions.h"
using namespace llvm;

//===----------------------------------------------------------------------===//
//                              Generic Code
//===----------------------------------------------------------------------===//

/// Initialize - this method must be called before any actual lowering is
/// done.  This specifies the current context for codegen, and gives the
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Target/TargetOptions.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Target/TargetOptions.h`。

### Lines 40-48
```cpp
/// lowering implementations a chance to set up their default sections.
void TargetLoweringObjectFile::Initialize(MCContext &ctx,
                                          const TargetMachine &TM) {
  // `Initialize` can be called more than once.
  delete Mang;
  Mang = new Mangler();
  initMCObjectFileInfo(ctx, TM.isPositionIndependent(),
                       TM.getCodeModel() == CodeModel::Large);

```
- **EN**: Implements logic around `Initialize`, `Mangler`, `initMCObjectFileInfo`, `getCodeModel`; this block works at the MC layer.
- **CN**: 围绕 `Initialize`, `Mangler`, `initMCObjectFileInfo`, `getCodeModel` 实现具体逻辑；这一段工作在 MC 层。

### Lines 49-59
```cpp
  // Reset various EH DWARF encodings.
  PersonalityEncoding = LSDAEncoding = TTypeEncoding = dwarf::DW_EH_PE_absptr;
  CallSiteEncoding = dwarf::DW_EH_PE_uleb128;

  this->TM = &TM;
}

TargetLoweringObjectFile::~TargetLoweringObjectFile() {
  delete Mang;
}

```
- **EN**: Implements logic around `~TargetLoweringObjectFile`.
- **CN**: 围绕 `~TargetLoweringObjectFile` 实现具体逻辑。

### Lines 60-68
```cpp
unsigned TargetLoweringObjectFile::getCallSiteEncoding() const {
  // If target does not have LEB128 directives, we would need the
  // call site encoding to be udata4 so that the alternative path
  // for not having LEB128 directives could work.
  if (!getContext().getAsmInfo().hasLEB128Directives())
    return dwarf::DW_EH_PE_udata4;
  return CallSiteEncoding;
}

```
- **EN**: Implements logic around `getCallSiteEncoding`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getCallSiteEncoding` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 69-81
```cpp
static bool isNullOrUndef(const Constant *C) {
  // Check that the constant isn't all zeros or undefs.
  if (C->isNullValue() || isa<UndefValue>(C))
    return true;
  if (!isa<ConstantAggregate>(C))
    return false;
  for (const auto *Operand : C->operand_values()) {
    if (!isNullOrUndef(cast<Constant>(Operand)))
      return false;
  }
  return true;
}

```
- **EN**: Implements logic around `isNullOrUndef`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isNullOrUndef` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 82-92
```cpp
static bool isSuitableForBSS(const GlobalVariable *GV) {
  const Constant *C = GV->getInitializer();

  // Must have zero initializer.
  if (!isNullOrUndef(C))
    return false;

  // Leave constant zeros in readonly constant sections, so they can be shared.
  if (GV->isConstant())
    return false;

```
- **EN**: Implements logic around `isSuitableForBSS`, `getInitializer`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isSuitableForBSS`, `getInitializer` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 93-101
```cpp
  // If the global has an explicit section specified, don't put it in BSS.
  if (GV->hasSection())
    return false;

  // Otherwise, put it in BSS!
  return true;
}

/// IsNullTerminatedString - Return true if the specified constant (which is
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 102-110
```cpp
/// known to have a type that is an array of 1/2/4 byte elements) ends with a
/// nul value and contains no other nuls in it.  Note that this is more general
/// than ConstantDataSequential::isString because we allow 2 & 4 byte strings.
static bool IsNullTerminatedString(const Constant *C) {
  // First check: is we have constant array terminated with zero
  if (const ConstantDataSequential *CDS = dyn_cast<ConstantDataSequential>(C)) {
    uint64_t NumElts = CDS->getNumElements();
    assert(NumElts != 0 && "Can't have an empty CDS");

```
- **EN**: Implements logic around `IsNullTerminatedString`, `getNumElements`, `assert`; this block applies conditional target rules.
- **CN**: 围绕 `IsNullTerminatedString`, `getNumElements`, `assert` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 111-120
```cpp
    if (CDS->getElementAsInteger(NumElts-1) != 0)
      return false; // Not null terminated.

    // Verify that the null doesn't occur anywhere else in the string.
    for (uint64_t i = 0; i != NumElts - 1; ++i)
      if (CDS->getElementAsInteger(i) == 0)
        return false;
    return true;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 121-131
```cpp
  // Another possibility: [1 x i8] zeroinitializer
  if (isa<ConstantAggregateZero>(C))
    return cast<ArrayType>(C->getType())->getNumElements() == 1;

  return false;
}

MCSymbol *TargetLoweringObjectFile::getSymbolWithGlobalValueBase(
    const GlobalValue *GV, StringRef Suffix, const TargetMachine &TM) const {
  assert(!Suffix.empty());

```
- **EN**: Implements logic around `cast<ArrayType>`, `getSymbolWithGlobalValueBase`, `assert`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `cast<ArrayType>`, `getSymbolWithGlobalValueBase`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 132-144
```cpp
  SmallString<60> NameStr;
  NameStr += GV->getDataLayout().getInternalSymbolPrefix();
  TM.getNameWithPrefix(NameStr, GV, *Mang);
  NameStr.append(Suffix.begin(), Suffix.end());
  return getContext().getOrCreateSymbol(NameStr);
}

MCSymbol *TargetLoweringObjectFile::getCFIPersonalitySymbol(
    const GlobalValue *GV, const TargetMachine &TM,
    MachineModuleInfo *MMI) const {
  return TM.getSymbol(GV);
}

```
- **EN**: Implements logic around `getDataLayout`, `getNameWithPrefix`, `append`, `getContext`, ...; this block returns target-specific results.
- **CN**: 围绕 `getDataLayout`, `getNameWithPrefix`, `append`, `getContext`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 145-154
```cpp
void TargetLoweringObjectFile::emitPersonalityValue(
    MCStreamer &Streamer, const DataLayout &, const MCSymbol *Sym,
    const MachineModuleInfo *MMI) const {}

void TargetLoweringObjectFile::emitCGProfileMetadata(MCStreamer &Streamer,
                                                     Module &M) const {
  MCContext &C = getContext();
  SmallVector<Module::ModuleFlagEntry, 8> ModuleFlags;
  M.getModuleFlagsMetadata(ModuleFlags);

```
- **EN**: Implements logic around `emitPersonalityValue`, `emitCGProfileMetadata`, `getContext`, `getModuleFlagsMetadata`; this block works at the MC layer.
- **CN**: 围绕 `emitPersonalityValue`, `emitCGProfileMetadata`, `getContext`, `getModuleFlagsMetadata` 实现具体逻辑；这一段工作在 MC 层。

### Lines 155-164
```cpp
  MDNode *CGProfile = nullptr;

  for (const auto &MFE : ModuleFlags) {
    StringRef Key = MFE.Key->getString();
    if (Key == "CG Profile") {
      CGProfile = cast<MDNode>(MFE.Val);
      break;
    }
  }

```
- **EN**: Implements logic around `getString`, `cast<MDNode>`; this block applies conditional target rules.
- **CN**: 围绕 `getString`, `cast<MDNode>` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 165-177
```cpp
  if (!CGProfile)
    return;

  auto GetSym = [this](const MDOperand &MDO) -> MCSymbol * {
    if (!MDO)
      return nullptr;
    auto *V = cast<ValueAsMetadata>(MDO);
    const Function *F = cast<Function>(V->getValue()->stripPointerCasts());
    if (F->hasDLLImportStorageClass())
      return nullptr;
    return TM->getSymbol(F);
  };

```
- **EN**: Implements logic around `cast<ValueAsMetadata>`, `cast<Function>`, `getSymbol`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `cast<ValueAsMetadata>`, `cast<Function>`, `getSymbol` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 178-194
```cpp
  for (const auto &Edge : CGProfile->operands()) {
    MDNode *E = cast<MDNode>(Edge);
    const MCSymbol *From = GetSym(E->getOperand(0));
    const MCSymbol *To = GetSym(E->getOperand(1));
    // Skip null functions. This can happen if functions are dead stripped after
    // the CGProfile pass has been run.
    if (!From || !To)
      continue;
    uint64_t Count = cast<ConstantAsMetadata>(E->getOperand(2))
                         ->getValue()
                         ->getUniqueInteger()
                         .getZExtValue();
    Streamer.emitCGProfileEntry(MCSymbolRefExpr::create(From, C),
                                MCSymbolRefExpr::create(To, C), Count);
  }
}

```
- **EN**: Implements logic around `cast<MDNode>`, `GetSym`, `cast<ConstantAsMetadata>`, `getValue`, ...; this block applies conditional target rules.
- **CN**: 围绕 `cast<MDNode>`, `GetSym`, `cast<ConstantAsMetadata>`, `getValue`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 195-212
```cpp
void TargetLoweringObjectFile::emitPseudoProbeDescMetadata(
    MCStreamer &Streamer, Module &M,
    std::function<void(MCStreamer &Streamer)> COMDATSymEmitter) const {
  NamedMDNode *FuncInfo = M.getNamedMetadata(PseudoProbeDescMetadataName);
  if (!FuncInfo)
    return;

  // Emit a descriptor for every function including functions that have an
  // available external linkage. We may not want this for imported functions
  // that has code in another thinLTO module but we don't have a good way to
  // tell them apart from inline functions defined in header files. Therefore
  // we put each descriptor in a separate comdat section and rely on the
  // linker to deduplicate.
  auto &C = getContext();
  for (const auto *Operand : FuncInfo->operands()) {
    const auto *MD = cast<MDNode>(Operand);
    auto *GUID = mdconst::extract<ConstantInt>(MD->getOperand(0));
    auto *Hash = mdconst::extract<ConstantInt>(MD->getOperand(1));
```
- **EN**: Implements logic around `emitPseudoProbeDescMetadata`, `function<void`, `getNamedMetadata`, `getContext`, ...; this block applies conditional target rules.
- **CN**: 围绕 `emitPseudoProbeDescMetadata`, `function<void`, `getNamedMetadata`, `getContext`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 213-223
```cpp
    auto *Name = cast<MDString>(MD->getOperand(2));
    auto *S = C.getObjectFileInfo()->getPseudoProbeDescSection(
        TM->getFunctionSections() ? Name->getString() : StringRef(),
        Hash->getZExtValue());

    Streamer.switchSection(S);

    // emit COFF COMDAT symbol.
    if (COMDATSymEmitter)
      COMDATSymEmitter(Streamer);

```
- **EN**: Implements logic around `cast<MDString>`, `getObjectFileInfo`, `getFunctionSections`, `getZExtValue`, ...; this block uses `switch`-based dispatch; applies conditional target rules.
- **CN**: 围绕 `cast<MDString>`, `getObjectFileInfo`, `getFunctionSections`, `getZExtValue`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则。

### Lines 224-234
```cpp
    Streamer.emitInt64(GUID->getZExtValue());
    Streamer.emitInt64(Hash->getZExtValue());
    Streamer.emitULEB128IntValue(Name->getString().size());
    Streamer.emitBytes(Name->getString());
  }
}

static bool containsConstantPtrAuth(const Constant *C) {
  if (isa<ConstantPtrAuth>(C))
    return true;

```
- **EN**: Implements logic around `emitInt64`, `emitULEB128IntValue`, `emitBytes`, `containsConstantPtrAuth`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `emitInt64`, `emitULEB128IntValue`, `emitBytes`, `containsConstantPtrAuth` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 235-244
```cpp
  if (isa<BlockAddress>(C) || isa<GlobalValue>(C))
    return false;

  for (const Value *Op : C->operands())
    if (containsConstantPtrAuth(cast<Constant>(Op)))
      return true;

  return false;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 245-253
```cpp
/// getKindForGlobal - This is a top-level target-independent classifier for
/// a global object.  Given a global variable and information from the TM, this
/// function classifies the global in a target independent manner. This function
/// may be overridden by the target implementation.
SectionKind TargetLoweringObjectFile::getKindForGlobal(const GlobalObject *GO,
                                                       const TargetMachine &TM){
  assert(!GO->isDeclarationForLinker() &&
         "Can only be used for global definitions");

```
- **EN**: Implements logic around `getKindForGlobal`, `assert`.
- **CN**: 围绕 `getKindForGlobal`, `assert` 实现具体逻辑。

### Lines 254-264
```cpp
  // Functions are classified as text sections.
  if (isa<Function>(GO))
    return SectionKind::getText();

  // Basic blocks are classified as text sections.
  if (isa<BasicBlock>(GO))
    return SectionKind::getText();

  // Global variables require more detailed analysis.
  const auto *GVar = cast<GlobalVariable>(GO);

```
- **EN**: Implements logic around `getText`, `cast<GlobalVariable>`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getText`, `cast<GlobalVariable>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 265-277
```cpp
  // Handle thread-local data first.
  if (GVar->isThreadLocal()) {
    if (isSuitableForBSS(GVar) && !TM.Options.NoZerosInBSS) {
      // Zero-initialized TLS variables with local linkage always get classified
      // as ThreadBSSLocal.
      if (GVar->hasLocalLinkage()) {
        return SectionKind::getThreadBSSLocal();
      }
      return SectionKind::getThreadBSS();
    }
    return SectionKind::getThreadData();
  }

```
- **EN**: Implements logic around `getThreadBSSLocal`, `getThreadBSS`, `getThreadData`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getThreadBSSLocal`, `getThreadBSS`, `getThreadData` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 278-291
```cpp
  // Variables with common linkage always get classified as common.
  if (GVar->hasCommonLinkage())
    return SectionKind::getCommon();

  // Most non-mergeable zero data can be put in the BSS section unless otherwise
  // specified.
  if (isSuitableForBSS(GVar) && !TM.Options.NoZerosInBSS) {
    if (GVar->hasLocalLinkage())
      return SectionKind::getBSSLocal();
    else if (GVar->hasExternalLinkage())
      return SectionKind::getBSSExtern();
    return SectionKind::getBSS();
  }

```
- **EN**: Implements logic around `getCommon`, `getBSSLocal`, `getBSSExtern`, `getBSS`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getCommon`, `getBSSLocal`, `getBSSExtern`, `getBSS` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 292-309
```cpp
  // Global variables with '!exclude' should get the exclude section kind if
  // they have an explicit section and no other metadata.
  if (GVar->hasSection())
    if (MDNode *MD = GVar->getMetadata(LLVMContext::MD_exclude))
      if (!MD->getNumOperands())
        return SectionKind::getExclude();

  // If the global is marked constant, we can put it into a mergable section,
  // a mergable string section, or general .data if it contains relocations.
  if (GVar->isConstant()) {
    // If the initializer for the global contains something that requires a
    // relocation, then we may have to drop this into a writable data section
    // even though it is marked const.
    const Constant *C = GVar->getInitializer();
    if (!C->needsRelocation()) {
      // If the global is required to have a unique address, it can't be put
      // into a mergable section: just drop it into the general read-only
      // section instead.
```
- **EN**: Implements logic around `getExclude`, `getInitializer`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getExclude`, `getInitializer` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 310-325
```cpp
      if (!GVar->hasGlobalUnnamedAddr())
        return SectionKind::getReadOnly();

      // If initializer is a null-terminated string, put it in a "cstring"
      // section of the right width.
      if (ArrayType *ATy = dyn_cast<ArrayType>(C->getType())) {
        if (IntegerType *ITy =
              dyn_cast<IntegerType>(ATy->getElementType())) {
          if ((ITy->getBitWidth() == 8 || ITy->getBitWidth() == 16 ||
               ITy->getBitWidth() == 32) &&
              IsNullTerminatedString(C)) {
            if (ITy->getBitWidth() == 8)
              return SectionKind::getMergeable1ByteCString();
            if (ITy->getBitWidth() == 16)
              return SectionKind::getMergeable2ByteCString();

```
- **EN**: Implements logic around `getReadOnly`, `dyn_cast<IntegerType>`, `getBitWidth`, `IsNullTerminatedString`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getReadOnly`, `dyn_cast<IntegerType>`, `getBitWidth`, `IsNullTerminatedString`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 326-343
```cpp
            assert(ITy->getBitWidth() == 32 && "Unknown width");
            return SectionKind::getMergeable4ByteCString();
          }
        }
      }

      // Otherwise, just drop it into a mergable constant section.  If we have
      // a section for this size, use it, otherwise use the arbitrary sized
      // mergable section.
      switch (
          GVar->getDataLayout().getTypeAllocSize(C->getType())) {
      case 4:  return SectionKind::getMergeableConst4();
      case 8:  return SectionKind::getMergeableConst8();
      case 16: return SectionKind::getMergeableConst16();
      case 32: return SectionKind::getMergeableConst32();
      default:
        return SectionKind::getReadOnly();
      }
```
- **EN**: Implements logic around `assert`, `getMergeable4ByteCString`, `getDataLayout`, `getMergeableConst4`, ...; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `assert`, `getMergeable4ByteCString`, `getDataLayout`, `getMergeableConst4`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 344-360
```cpp

    } else {
      // The dynamic linker always needs to fix PtrAuth relocations up.
      if (containsConstantPtrAuth(C))
        return SectionKind::getReadOnlyWithRel();

      // In static, ROPI and RWPI relocation models, the linker will resolve
      // all addresses, so the relocation entries will actually be constants by
      // the time the app starts up.  However, we can't put this into a
      // mergable section, because the linker doesn't take relocations into
      // consideration when it tries to merge entries in the section.
      Reloc::Model ReloModel = TM.getRelocationModel();
      if (ReloModel == Reloc::Static || ReloModel == Reloc::ROPI ||
          ReloModel == Reloc::RWPI || ReloModel == Reloc::ROPI_RWPI ||
          !C->needsDynamicRelocation())
        return SectionKind::getReadOnly();

```
- **EN**: Implements logic around `getReadOnlyWithRel`, `getRelocationModel`, `needsDynamicRelocation`, `getReadOnly`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getReadOnlyWithRel`, `getRelocationModel`, `needsDynamicRelocation`, `getReadOnly` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 361-370
```cpp
      // Otherwise, the dynamic linker needs to fix it up, put it in the
      // writable data.rel section.
      return SectionKind::getReadOnlyWithRel();
    }
  }

  // Okay, this isn't a constant.
  return SectionKind::getData();
}

```
- **EN**: Implements logic around `getReadOnlyWithRel`, `getData`; this block returns target-specific results.
- **CN**: 围绕 `getReadOnlyWithRel`, `getData` 实现具体逻辑；这一段返回目标相关结果。

### Lines 371-379
```cpp
/// This method computes the appropriate section to emit the specified global
/// variable or function definition.  This should not be passed external (or
/// available externally) globals.
MCSection *TargetLoweringObjectFile::SectionForGlobal(
    const GlobalObject *GO, SectionKind Kind, const TargetMachine &TM) const {
  // Select section name.
  if (GO->hasSection())
    return getExplicitSectionGlobal(GO, Kind, TM);

```
- **EN**: Implements logic around `SectionForGlobal`, `getExplicitSectionGlobal`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `SectionForGlobal`, `getExplicitSectionGlobal` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 380-389
```cpp
  if (auto *GVar = dyn_cast<GlobalVariable>(GO)) {
    auto Attrs = GVar->getAttributes();
    if ((Attrs.hasAttribute("bss-section") && Kind.isBSS()) ||
        (Attrs.hasAttribute("data-section") && Kind.isData()) ||
        (Attrs.hasAttribute("relro-section") && Kind.isReadOnlyWithRel()) ||
        (Attrs.hasAttribute("rodata-section") && Kind.isReadOnly()))  {
       return getExplicitSectionGlobal(GO, Kind, TM);
    }
  }

```
- **EN**: Implements logic around `getAttributes`, `hasAttribute`, `getExplicitSectionGlobal`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getAttributes`, `hasAttribute`, `getExplicitSectionGlobal` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 390-402
```cpp
  // Use default section depending on the 'type' of global
  return SelectSectionForGlobal(GO, Kind, TM);
}

/// This method computes the appropriate section to emit the specified global
/// variable or function definition. This should not be passed external (or
/// available externally) globals.
MCSection *
TargetLoweringObjectFile::SectionForGlobal(const GlobalObject *GO,
                                           const TargetMachine &TM) const {
  return SectionForGlobal(GO, getKindForGlobal(GO, TM), TM);
}

```
- **EN**: Implements logic around `SelectSectionForGlobal`, `SectionForGlobal`; this block returns target-specific results.
- **CN**: 围绕 `SelectSectionForGlobal`, `SectionForGlobal` 实现具体逻辑；这一段返回目标相关结果。

### Lines 403-413
```cpp
MCSection *TargetLoweringObjectFile::getSectionForJumpTable(
    const Function &F, const TargetMachine &TM) const {
  return getSectionForJumpTable(F, TM, /*JTE=*/nullptr);
}

MCSection *TargetLoweringObjectFile::getSectionForJumpTable(
    const Function &F, const TargetMachine &TM,
    const MachineJumpTableEntry *JTE) const {
  Align Alignment(1);
  return getSectionForConstant(F.getDataLayout(), SectionKind::getReadOnly(),
                               /*C=*/nullptr, Alignment, &F);
```
- **EN**: Implements logic around `getSectionForJumpTable`, `Alignment`, `getSectionForConstant`; this block returns target-specific results.
- **CN**: 围绕 `getSectionForJumpTable`, `Alignment`, `getSectionForConstant` 实现具体逻辑；这一段返回目标相关结果。

### Lines 414-423
```cpp
}

bool TargetLoweringObjectFile::shouldPutJumpTableInFunctionSection(
    bool UsesLabelDifference, const Function &F) const {
  // In PIC mode, we need to emit the jump table to the same section as the
  // function body itself, otherwise the label differences won't make sense.
  // FIXME: Need a better predicate for this: what about custom entries?
  if (UsesLabelDifference)
    return true;

```
- **EN**: Implements logic around `shouldPutJumpTableInFunctionSection`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `shouldPutJumpTableInFunctionSection` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 424-432
```cpp
  // We should also do if the section name is NULL or function is declared
  // in discardable section
  // FIXME: this isn't the right predicate, should be based on the MCSection
  // for the function.
  return F.isWeakForLinker();
}

/// Given a mergable constant with the specified size and relocation
/// information, return a section that it should be placed in.
```
- **EN**: Implements logic around `isWeakForLinker`; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `isWeakForLinker` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 433-441
```cpp
MCSection *TargetLoweringObjectFile::getSectionForConstant(
    const DataLayout &DL, SectionKind Kind, const Constant *C, Align &Alignment,
    const Function *F) const {
  if (Kind.isReadOnly() && ReadOnlySection != nullptr)
    return ReadOnlySection;

  return DataSection;
}

```
- **EN**: Implements logic around `getSectionForConstant`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getSectionForConstant` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 442-453
```cpp
MCSection *TargetLoweringObjectFile::getSectionForConstant(
    const DataLayout &DL, SectionKind Kind, const Constant *C, Align &Alignment,
    const Function *F, StringRef SectionPrefix) const {
  // Fallback to `getSectionForConstant` without `SectionPrefix` parameter if it
  // is empty.
  if (SectionPrefix.empty())
    return getSectionForConstant(DL, Kind, C, Alignment, F);
  report_fatal_error(
      "TargetLoweringObjectFile::getSectionForConstant that "
      "accepts SectionPrefix is not implemented for the object file format");
}

```
- **EN**: Implements logic around `getSectionForConstant`, `report_fatal_error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getSectionForConstant`, `report_fatal_error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 454-464
```cpp
MCSection *TargetLoweringObjectFile::getSectionForMachineBasicBlock(
    const Function &F, const MachineBasicBlock &MBB,
    const TargetMachine &TM) const {
  return nullptr;
}

MCSection *TargetLoweringObjectFile::getUniqueSectionForFunction(
    const Function &F, const TargetMachine &TM) const {
  return nullptr;
}

```
- **EN**: Implements logic around `getSectionForMachineBasicBlock`, `getUniqueSectionForFunction`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getSectionForMachineBasicBlock`, `getUniqueSectionForFunction` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 465-473
```cpp
/// getTTypeGlobalReference - Return an MCExpr to use for a
/// reference to the specified global variable from exception
/// handling information.
const MCExpr *TargetLoweringObjectFile::getTTypeGlobalReference(
    const GlobalValue *GV, unsigned Encoding, const TargetMachine &TM,
    MachineModuleInfo *MMI, MCStreamer &Streamer) const {
  const MCSymbolRefExpr *Ref =
      MCSymbolRefExpr::create(TM.getSymbol(GV), getContext());

```
- **EN**: Implements logic around `getTTypeGlobalReference`, `create`; this block returns target-specific results.
- **CN**: 围绕 `getTTypeGlobalReference`, `create` 实现具体逻辑；这一段返回目标相关结果。

### Lines 474-491
```cpp
  return getTTypeReference(Ref, Encoding, Streamer);
}

const MCExpr *TargetLoweringObjectFile::
getTTypeReference(const MCSymbolRefExpr *Sym, unsigned Encoding,
                  MCStreamer &Streamer) const {
  switch (Encoding & 0x70) {
  default:
    report_fatal_error("We do not support this DWARF encoding yet!");
  case dwarf::DW_EH_PE_absptr:
    // Do nothing special
    return Sym;
  case dwarf::DW_EH_PE_pcrel: {
    // Emit a label to the streamer for the current position.  This gives us
    // .-foo addressing.
    MCSymbol *PCSym = getContext().createTempSymbol();
    Streamer.emitLabel(PCSym);
    const MCExpr *PC = MCSymbolRefExpr::create(PCSym, getContext());
```
- **EN**: Implements logic around `getTTypeReference`, `report_fatal_error`, `getContext`, `emitLabel`, ...; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `getTTypeReference`, `report_fatal_error`, `getContext`, `emitLabel`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 492-502
```cpp
    return MCBinaryExpr::createSub(Sym, PC, getContext());
  }
  }
}

const MCExpr *TargetLoweringObjectFile::getDebugThreadLocalSymbol(const MCSymbol *Sym) const {
  // FIXME: It's not clear what, if any, default this should have - perhaps a
  // null return could mean 'no location' & we should just do that here.
  return MCSymbolRefExpr::create(Sym, getContext());
}

```
- **EN**: Implements logic around `createSub`, `getDebugThreadLocalSymbol`, `create`; this block returns target-specific results.
- **CN**: 围绕 `createSub`, `getDebugThreadLocalSymbol`, `create` 实现具体逻辑；这一段返回目标相关结果。

### Lines 503-507
```cpp
void TargetLoweringObjectFile::getNameWithPrefix(
    SmallVectorImpl<char> &OutName, const GlobalValue *GV,
    const TargetMachine &TM) const {
  Mang->getNameWithPrefix(OutName, GV, /*CannotUsePrivateLabel=*/false);
}
```
- **EN**: Implements logic around `getNameWithPrefix`.
- **CN**: 围绕 `getNameWithPrefix` 实现具体逻辑。

## Key Concepts / 关键概念

- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/Target/TargetLoweringObjectFile.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Mangler.h`, `llvm/IR/Module.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h` ... (+6 more)
- **LLVM subsystems / LLVM 子系统**: MC, IR, Support
