# WebAssemblyAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyAsmPrinter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file contains a printer that converts from our internal representation of machine-dependent LLVM code to the WebAssembly assembly language.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyAsmPrinter.cpp`，主要负责 WebAssembly 后端的汇编与 MC 发射逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyAsmPrinter.cpp - WebAssembly LLVM assembly writer ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-13

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains a printer that converts from our internal
/// representation of machine-dependent LLVM code to the WebAssembly assembly
/// language.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 14-20

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyAsmPrinter.h"
#include "MCTargetDesc/WebAssemblyMCAsmInfo.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "MCTargetDesc/WebAssemblyTargetStreamer.h"
#include "TargetInfo/WebAssemblyTargetInfo.h"
```
- **EN**: Pulls in direct dependencies required by this assembly / MC emission logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该汇编与 MC 发射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 21-27

```cpp
#include "Utils/WebAssemblyTypeUtilities.h"
#include "WebAssembly.h"
#include "WebAssemblyMCInstLower.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblyRegisterInfo.h"
#include "WebAssemblyRuntimeLibcallSignatures.h"
#include "WebAssemblyTargetMachine.h"
```
- **EN**: Pulls in direct dependencies required by this assembly / MC emission logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该汇编与 MC 发射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 28-34

```cpp
#include "WebAssemblyUtilities.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/ADT/SmallSet.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/BinaryFormat/Wasm.h"
#include "llvm/CodeGen/Analysis.h"
```
- **EN**: Pulls in direct dependencies required by this assembly / MC emission logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该汇编与 MC 发射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 35-41

```cpp
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineModuleInfoImpls.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DebugInfoMetadata.h"
#include "llvm/IR/GlobalVariable.h"
```
- **EN**: Pulls in direct dependencies required by this assembly / MC emission logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该汇编与 MC 发射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 42-48

```cpp
#include "llvm/IR/Metadata.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCSectionWasm.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCSymbolWasm.h"
```
- **EN**: Pulls in direct dependencies required by this assembly / MC emission logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该汇编与 MC 发射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 49-55

```cpp
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this assembly / MC emission logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该汇编与 MC 发射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 56-59

```cpp
#define DEBUG_TYPE "asm-printer"

extern cl::opt<bool> WasmKeepRegisters;
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 60-61

```cpp
//===----------------------------------------------------------------------===//
// Helpers.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Helpers.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Helpers.”。

### Lines 62-99

```cpp
//===----------------------------------------------------------------------===//

MVT WebAssemblyAsmPrinter::getRegType(unsigned RegNo) const {
  const TargetRegisterInfo *TRI = Subtarget->getRegisterInfo();
  const TargetRegisterClass *TRC = MRI->getRegClass(RegNo);
  for (MVT T : {MVT::i32, MVT::i64, MVT::f32, MVT::f64, MVT::v16i8, MVT::v8i16,
                MVT::v4i32, MVT::v2i64, MVT::v4f32, MVT::v2f64, MVT::v8f16})
    if (TRI->isTypeLegalForClass(*TRC, T))
      return T;
  LLVM_DEBUG(errs() << "Unknown type for register number: " << RegNo);
  llvm_unreachable("Unknown register type");
  return MVT::Other;
}

std::string WebAssemblyAsmPrinter::regToString(const MachineOperand &MO) {
  Register RegNo = MO.getReg();
  assert(RegNo.isVirtual() &&
         "Unlowered physical register encountered during assembly printing");
  assert(!MFI->isVRegStackified(RegNo));
  unsigned WAReg = MFI->getWAReg(RegNo);
  assert(WAReg != WebAssembly::UnusedReg);
  return '$' + utostr(WAReg);
}

WebAssemblyTargetStreamer *WebAssemblyAsmPrinter::getTargetStreamer() {
  MCTargetStreamer *TS = OutStreamer->getTargetStreamer();
  return static_cast<WebAssemblyTargetStreamer *>(TS);
}

// Emscripten exception handling helpers
//
// This converts invoke names generated by LowerEmscriptenEHSjLj to real names
// that are expected by JavaScript glue code. The invoke names generated by
// Emscripten JS glue code are based on their argument and return types; for
// example, for a function that takes an i32 and returns nothing, it is
// 'invoke_vi'. But the format of invoke generated by LowerEmscriptenEHSjLj pass
// contains a mangled string generated from their IR types, for example,
// "__invoke_void_%struct.mystruct*_int", because final wasm types are not
```
- **EN**: Implements helper routine(s) `getRegType`, `getRegisterInfo`, `getRegClass` for this portion of the WebAssembly backend assembly / MC emission logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `getRegType`, `getRegisterInfo`, `getRegClass`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 100-137

```cpp
// available in the IR pass. So we convert those names to the form that
// Emscripten JS code expects.
//
// Refer to LowerEmscriptenEHSjLj pass for more details.

// Returns true if the given function name is an invoke name generated by
// LowerEmscriptenEHSjLj pass.
static bool isEmscriptenInvokeName(StringRef Name) {
  if (Name.front() == '"' && Name.back() == '"')
    Name = Name.substr(1, Name.size() - 2);
  return Name.starts_with("__invoke_");
}

// Returns a character that represents the given wasm value type in invoke
// signatures.
static char getInvokeSig(wasm::ValType VT) {
  switch (VT) {
  case wasm::ValType::I32:
    return 'i';
  case wasm::ValType::I64:
    return 'j';
  case wasm::ValType::F32:
    return 'f';
  case wasm::ValType::F64:
    return 'd';
  case wasm::ValType::V128:
    return 'V';
  case wasm::ValType::FUNCREF:
    return 'F';
  case wasm::ValType::EXTERNREF:
    return 'X';
  case wasm::ValType::EXNREF:
    return 'E';
  default:
    llvm_unreachable("Unhandled wasm::ValType enum");
  }
}
```
- **EN**: Implements helper routine(s) `isEmscriptenInvokeName`, `front`, `back` for this portion of the WebAssembly backend assembly / MC emission logic.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `isEmscriptenInvokeName`, `front`, `back`。

### Lines 138-153

```cpp
// Given the wasm signature, generate the invoke name in the format JS glue code
// expects.
static std::string getEmscriptenInvokeSymbolName(wasm::WasmSignature *Sig) {
  assert(Sig->Returns.size() <= 1);
  std::string Ret = "invoke_";
  if (!Sig->Returns.empty())
    for (auto VT : Sig->Returns)
      Ret += getInvokeSig(VT);
  else
    Ret += 'v';
  // Invokes' first argument is a pointer to the original function, so skip it
  for (unsigned I = 1, E = Sig->Params.size(); I < E; I++)
    Ret += getInvokeSig(Sig->Params[I]);
  return Ret;
}
```
- **EN**: Implements helper routine(s) `getEmscriptenInvokeSymbolName`, `size`, `empty` for this portion of the WebAssembly backend assembly / MC emission logic.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `getEmscriptenInvokeSymbolName`, `size`, `empty`。

### Lines 154-155

```cpp
//===----------------------------------------------------------------------===//
// WebAssemblyAsmPrinter Implementation.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssemblyAsmPrinter Implementation.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssemblyAsmPrinter Implementation.”。

### Lines 156-193

```cpp
//===----------------------------------------------------------------------===//

MCSymbolWasm *WebAssemblyAsmPrinter::getMCSymbolForFunction(
    const Function *F, wasm::WasmSignature *Sig, bool &InvokeDetected) {
  MCSymbolWasm *WasmSym = nullptr;

  const bool EnableEmEH =
      WebAssembly::WasmEnableEmEH || WebAssembly::WasmEnableEmSjLj;
  if (EnableEmEH && isEmscriptenInvokeName(F->getName())) {
    assert(Sig);
    InvokeDetected = true;
    if (Sig->Returns.size() > 1) {
      std::string Msg =
          "Emscripten EH/SjLj does not support multivalue returns: " +
          std::string(F->getName()) + ": " +
          WebAssembly::signatureToString(Sig);
      report_fatal_error(Twine(Msg));
    }
    WasmSym = static_cast<MCSymbolWasm *>(
        GetExternalSymbolSymbol(getEmscriptenInvokeSymbolName(Sig)));
  } else {
    WasmSym = static_cast<MCSymbolWasm *>(getSymbol(F));
  }
  return WasmSym;
}

void WebAssemblyAsmPrinter::emitGlobalVariable(const GlobalVariable *GV) {
  if (GV->hasCommonLinkage()) {
    OutContext.reportError(SMLoc(),
                           "common symbols are not yet implemented for Wasm: " +
                               getSymbol(GV)->getName());
    return;
  }

  if (!WebAssembly::isWasmVarAddressSpace(GV->getAddressSpace())) {
    AsmPrinter::emitGlobalVariable(GV);
    return;
  }
```
- **EN**: Implements helper routine(s) `getMCSymbolForFunction`, `isEmscriptenInvokeName`, `getName` for this portion of the WebAssembly backend assembly / MC emission logic.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `getMCSymbolForFunction`, `isEmscriptenInvokeName`, `getName`。

### Lines 194-231

```cpp

  assert(!GV->isThreadLocal());
  auto *Sym = static_cast<MCSymbolWasm *>(getSymbol(GV));
  if (!Sym->getType()) {
    SmallVector<MVT, 1> VTs;
    Type *GlobalVT = GV->getValueType();
    // Function-specific subtargets are not needed here: WebAssembly
    // coalesces features before isel, so use the TargetMachine's
    // module-wide subtarget to compute legal value types.
    auto &WasmTM = static_cast<const WebAssemblyTargetMachine &>(TM);
    const WebAssemblySubtarget *ST = WasmTM.getSubtargetImpl();
    const WebAssemblyTargetLowering &TLI = *ST->getTargetLowering();
    computeLegalValueVTs(TLI, GV->getParent()->getContext(),
                         GV->getDataLayout(), GlobalVT, VTs);

    WebAssembly::wasmSymbolSetType(Sym, GlobalVT, VTs);
  }

  emitVisibility(Sym, GV->getVisibility(), !GV->isDeclaration());
  emitSymbolType(Sym);
  if (GV->hasInitializer()) {
    assert(getSymbolPreferLocal(*GV) == Sym);
    emitLinkage(GV, Sym);
    OutStreamer->emitLabel(Sym);
    // TODO: Actually emit the initializer value.  Otherwise the global has the
    // default value for its type (0, ref.null, etc).
    OutStreamer->addBlankLine();
  }
}

MCSymbol *WebAssemblyAsmPrinter::getOrCreateWasmSymbol(StringRef Name) {
  auto *WasmSym = static_cast<MCSymbolWasm *>(GetExternalSymbolSymbol(Name));
  // May be called multiple times, so early out.
  if (WasmSym->getType())
    return WasmSym;

  const WebAssemblySubtarget &Subtarget = getSubtarget();
```
- **EN**: Implements helper routine(s) `isThreadLocal`, `getSymbol`, `getType` for this portion of the WebAssembly backend assembly / MC emission logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `isThreadLocal`, `getSymbol`, `getType`。 子目标特性裁剪会影响这里的行为。

### Lines 232-269

```cpp
  // Except for certain known symbols, all symbols used by CodeGen are
  // functions. It's OK to hardcode knowledge of specific symbols here; this
  // method is precisely there for fetching the signatures of known
  // Clang-provided symbols.
  if (Name == "__stack_pointer" || Name == "__tls_base" ||
      Name == "__memory_base" || Name == "__table_base" ||
      Name == "__tls_size" || Name == "__tls_align") {
    bool Mutable =
        Name == "__stack_pointer" || Name == "__tls_base";
    WasmSym->setType(wasm::WASM_SYMBOL_TYPE_GLOBAL);
    WasmSym->setGlobalType(wasm::WasmGlobalType{
        uint8_t(Subtarget.hasAddr64() ? wasm::WASM_TYPE_I64
                                      : wasm::WASM_TYPE_I32),
        Mutable});
    return WasmSym;
  }

  if (Name.starts_with("GCC_except_table")) {
    WasmSym->setType(wasm::WASM_SYMBOL_TYPE_DATA);
    return WasmSym;
  }

  SmallVector<wasm::ValType, 4> Returns;
  SmallVector<wasm::ValType, 4> Params;
  if (Name == "__cpp_exception" || Name == "__c_longjmp") {
    WasmSym->setType(wasm::WASM_SYMBOL_TYPE_TAG);
    WasmSym->setExternal(true);

    // Currently both C++ exceptions and C longjmps have a single pointer type
    // param. For C++ exceptions it is a pointer to an exception object, and for
    // C longjmps it is pointer to a struct that contains a setjmp buffer and a
    // longjmp return value. We may consider using multiple value parameters for
    // longjmps later when multivalue support is ready.
    wasm::ValType AddrType =
        Subtarget.hasAddr64() ? wasm::ValType::I64 : wasm::ValType::I32;
    Params.push_back(AddrType);
  } else { // Function symbols
    WasmSym->setType(wasm::WASM_SYMBOL_TYPE_FUNCTION);
```
- **EN**: Implements helper routine(s) `setType`, `setGlobalType`, `uint8_t` for this portion of the WebAssembly backend assembly / MC emission logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `setType`, `setGlobalType`, `uint8_t`。 子目标特性裁剪会影响这里的行为。

### Lines 270-307

```cpp
    WebAssembly::getLibcallSignature(Subtarget, Name, Returns, Params);
  }
  auto Signature = OutContext.createWasmSignature();
  Signature->Returns = std::move(Returns);
  Signature->Params = std::move(Params);
  WasmSym->setSignature(Signature);

  return WasmSym;
}

void WebAssemblyAsmPrinter::emitSymbolType(const MCSymbolWasm *Sym) {
  std::optional<wasm::WasmSymbolType> WasmTy = Sym->getType();
  if (!WasmTy)
    return;

  switch (*WasmTy) {
  case wasm::WASM_SYMBOL_TYPE_GLOBAL:
    getTargetStreamer()->emitGlobalType(Sym);
    break;
  case wasm::WASM_SYMBOL_TYPE_TAG:
    getTargetStreamer()->emitTagType(Sym);
    break;
  case wasm::WASM_SYMBOL_TYPE_TABLE:
    getTargetStreamer()->emitTableType(Sym);
    break;
  default:
    break; // We only handle globals, tags and tables here
  }
}

void WebAssemblyAsmPrinter::emitDecls(const Module &M) {
  if (signaturesEmitted)
    return;
  signaturesEmitted = true;

  // Normally symbols for globals get discovered as the MI gets lowered,
  // but we need to know about them ahead of time. This will however,
  // only find symbols that have been used. Unused symbols from globals will
```
- **EN**: Implements helper routine(s) `getLibcallSignature`, `createWasmSignature`, `move` for this portion of the WebAssembly backend assembly / MC emission logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `getLibcallSignature`, `createWasmSignature`, `move`。 子目标特性裁剪会影响这里的行为。

### Lines 308-345

```cpp
  // not be found here.
  MachineModuleInfoWasm &MMIW = MMI->getObjFileInfo<MachineModuleInfoWasm>();
  for (StringRef Name : MMIW.MachineSymbolsUsed) {
    auto *WasmSym = static_cast<MCSymbolWasm *>(getOrCreateWasmSymbol(Name));
    if (WasmSym->isFunction()) {
      // TODO(wvo): is there any case where this overlaps with the call to
      // emitFunctionType in the loop below?
      getTargetStreamer()->emitFunctionType(WasmSym);
    }
  }

  for (auto &It : OutContext.getSymbols()) {
    // Emit .globaltype, .tagtype, or .tabletype declarations for extern
    // declarations, i.e. those that have only been declared (but not defined)
    // in the current module
    auto Sym = static_cast<MCSymbolWasm *>(It.getValue().Symbol);
    if (Sym && !Sym->isDefined())
      emitSymbolType(Sym);
  }

  DenseSet<MCSymbol *> InvokeSymbols;
  for (const auto &F : M) {
    if (F.isIntrinsic())
      continue;

    // Emit function type info for all functions. This will emit duplicate
    // information for defined functions (which already have function type
    // info emitted alongside their definition), but this is necessary in
    // order to enable the single-pass WebAssemblyAsmTypeCheck to succeed.
    SmallVector<MVT, 4> Results;
    SmallVector<MVT, 4> Params;
    computeSignatureVTs(F.getFunctionType(), &F, F, TM, Params, Results);
    // At this point these MCSymbols may or may not have been created already
    // and thus also contain a signature, but we need to get the signature
    // anyway here in case it is an invoke that has not yet been created. We
    // will discard it later if it turns out not to be necessary.
    auto Signature = signatureFromMVTs(OutContext, Results, Params);
    bool InvokeDetected = false;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "not be found here.". Notable symbols in this range include `getOrCreateWasmSymbol`, `isFunction`, `TODO`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“not be found here.”。 该区间中较显眼的符号包括 `getOrCreateWasmSymbol`, `isFunction`, `TODO`。

### Lines 346-383

```cpp
    auto *Sym = getMCSymbolForFunction(&F, Signature, InvokeDetected);

    // Multiple functions can be mapped to the same invoke symbol. For
    // example, two IR functions '__invoke_void_i8*' and '__invoke_void_i32'
    // are both mapped to '__invoke_vi'. We keep them in a set once we emit an
    // Emscripten EH symbol so we don't emit the same symbol twice.
    if (InvokeDetected && !InvokeSymbols.insert(Sym).second)
      continue;

    Sym->setType(wasm::WASM_SYMBOL_TYPE_FUNCTION);
    if (!Sym->getSignature()) {
      Sym->setSignature(Signature);
    }

    getTargetStreamer()->emitFunctionType(Sym);

    if (F.hasFnAttribute("wasm-import-module")) {
      StringRef Name =
          F.getFnAttribute("wasm-import-module").getValueAsString();
      Sym->setImportModule(OutContext.allocateString(Name));
      getTargetStreamer()->emitImportModule(Sym, Name);
    }
    if (F.hasFnAttribute("wasm-import-name")) {
      // If this is a converted Emscripten EH/SjLj symbol, we shouldn't use
      // the original function name but the converted symbol name.
      StringRef Name =
          InvokeDetected
              ? Sym->getName()
              : F.getFnAttribute("wasm-import-name").getValueAsString();
      Sym->setImportName(OutContext.allocateString(Name));
      getTargetStreamer()->emitImportName(Sym, Name);
    }

    if (F.hasFnAttribute("wasm-export-name")) {
      auto *Sym = static_cast<MCSymbolWasm *>(getSymbol(&F));
      StringRef Name = F.getFnAttribute("wasm-export-name").getValueAsString();
      Sym->setExportName(OutContext.allocateString(Name));
      getTargetStreamer()->emitExportName(Sym, Name);
```
- **EN**: Implements helper routine(s) `getMCSymbolForFunction`, `insert`, `setType` for this portion of the WebAssembly backend assembly / MC emission logic.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `getMCSymbolForFunction`, `insert`, `setType`。

### Lines 384-421

```cpp
    }
  }
}

void WebAssemblyAsmPrinter::emitEndOfAsmFile(Module &M) {
  // This is required to emit external declarations (like .functypes) when
  // no functions are defined in the compilation unit and therefore,
  // emitDecls() is not called until now.
  emitDecls(M);

  // When a function's address is taken, a TABLE_INDEX relocation is emitted
  // against the function symbol at the use site.  However the relocation
  // doesn't explicitly refer to the table.  In the future we may want to
  // define a new kind of reloc against both the function and the table, so
  // that the linker can see that the function symbol keeps the table alive,
  // but for now manually mark the table as live.
  for (const auto &F : M) {
    if (!F.isIntrinsic() && F.hasAddressTaken()) {
      MCSymbolWasm *FunctionTable =
          WebAssembly::getOrCreateFunctionTableSymbol(OutContext, Subtarget);
      OutStreamer->emitSymbolAttribute(FunctionTable, MCSA_NoDeadStrip);
      break;
    }
  }

  for (const auto &G : M.globals()) {
    if (!G.hasInitializer() && G.hasExternalLinkage() &&
        !WebAssembly::isWasmVarAddressSpace(G.getAddressSpace()) &&
        G.getValueType()->isSized()) {
      uint16_t Size = G.getGlobalSize(M.getDataLayout());
      OutStreamer->emitELFSize(getSymbol(&G),
                               MCConstantExpr::create(Size, OutContext));
    }
  }

  if (const NamedMDNode *Named = M.getNamedMetadata("wasm.custom_sections")) {
    for (const Metadata *MD : Named->operands()) {
      const auto *Tuple = dyn_cast<MDTuple>(MD);
```
- **EN**: Implements helper routine(s) `emitEndOfAsmFile`, `declarations`, `emitDecls` for this portion of the WebAssembly backend assembly / MC emission logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `emitEndOfAsmFile`, `declarations`, `emitDecls`。 子目标特性裁剪会影响这里的行为。

### Lines 422-459

```cpp
      if (!Tuple || Tuple->getNumOperands() != 2)
        continue;
      const MDString *Name = dyn_cast<MDString>(Tuple->getOperand(0));
      const MDString *Contents = dyn_cast<MDString>(Tuple->getOperand(1));
      if (!Name || !Contents)
        continue;

      OutStreamer->pushSection();
      std::string SectionName = (".custom_section." + Name->getString()).str();
      MCSectionWasm *MySection =
          OutContext.getWasmSection(SectionName, SectionKind::getMetadata());
      OutStreamer->switchSection(MySection);
      OutStreamer->emitBytes(Contents->getString());
      OutStreamer->popSection();
    }
  }

  EmitProducerInfo(M);
  EmitTargetFeatures(M);
  EmitFunctionAttributes(M);
}

void WebAssemblyAsmPrinter::EmitProducerInfo(Module &M) {
  llvm::SmallVector<std::pair<std::string, std::string>, 4> Languages;
  if (const NamedMDNode *Debug = M.getNamedMetadata("llvm.dbg.cu")) {
    llvm::SmallSet<StringRef, 4> SeenLanguages;
    for (size_t I = 0, E = Debug->getNumOperands(); I < E; ++I) {
      const auto *CU = cast<DICompileUnit>(Debug->getOperand(I));
      StringRef Language =
          dwarf::LanguageString(CU->getSourceLanguage().getUnversionedName());

      Language.consume_front("DW_LANG_");
      if (SeenLanguages.insert(Language).second)
        Languages.emplace_back(Language.str(), "");
    }
  }

  llvm::SmallVector<std::pair<std::string, std::string>, 4> Tools;
```
- **EN**: Implements helper routine(s) `getNumOperands`, `getOperand`, `pushSection` for this portion of the WebAssembly backend assembly / MC emission logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `getNumOperands`, `getOperand`, `pushSection`。 子目标特性裁剪会影响这里的行为。

### Lines 460-497

```cpp
  if (const NamedMDNode *Ident = M.getNamedMetadata("llvm.ident")) {
    llvm::SmallSet<StringRef, 4> SeenTools;
    for (size_t I = 0, E = Ident->getNumOperands(); I < E; ++I) {
      const auto *S = cast<MDString>(Ident->getOperand(I)->getOperand(0));
      std::pair<StringRef, StringRef> Field = S->getString().split("version");
      StringRef Name = Field.first.trim();
      StringRef Version = Field.second.trim();
      if (SeenTools.insert(Name).second)
        Tools.emplace_back(Name.str(), Version.str());
    }
  }

  int FieldCount = int(!Languages.empty()) + int(!Tools.empty());
  if (FieldCount != 0) {
    MCSectionWasm *Producers = OutContext.getWasmSection(
        ".custom_section.producers", SectionKind::getMetadata());
    OutStreamer->pushSection();
    OutStreamer->switchSection(Producers);
    OutStreamer->emitULEB128IntValue(FieldCount);
    for (auto &Producers : {std::make_pair("language", &Languages),
            std::make_pair("processed-by", &Tools)}) {
      if (Producers.second->empty())
        continue;
      OutStreamer->emitULEB128IntValue(strlen(Producers.first));
      OutStreamer->emitBytes(Producers.first);
      OutStreamer->emitULEB128IntValue(Producers.second->size());
      for (auto &Producer : *Producers.second) {
        OutStreamer->emitULEB128IntValue(Producer.first.size());
        OutStreamer->emitBytes(Producer.first);
        OutStreamer->emitULEB128IntValue(Producer.second.size());
        OutStreamer->emitBytes(Producer.second);
      }
    }
    OutStreamer->popSection();
  }
}

void WebAssemblyAsmPrinter::EmitTargetFeatures(Module &M) {
```
- **EN**: Implements helper routine(s) `getNamedMetadata`, `getNumOperands`, `getOperand` for this portion of the WebAssembly backend assembly / MC emission logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `getNamedMetadata`, `getNumOperands`, `getOperand`。 子目标特性裁剪会影响这里的行为。

### Lines 498-535

```cpp
  struct FeatureEntry {
    uint8_t Prefix;
    std::string Name;
  };

  // Read target features and linkage policies from module metadata
  SmallVector<FeatureEntry, 4> EmittedFeatures;
  auto EmitFeature = [&](std::string Feature) {
    std::string MDKey = (StringRef("wasm-feature-") + Feature).str();
    Metadata *Policy = M.getModuleFlag(MDKey);
    if (Policy == nullptr)
      return;

    FeatureEntry Entry;
    Entry.Prefix = 0;
    Entry.Name = Feature;

    if (auto *MD = cast<ConstantAsMetadata>(Policy))
      if (auto *I = cast<ConstantInt>(MD->getValue()))
        Entry.Prefix = I->getZExtValue();

    // Silently ignore invalid metadata
    if (Entry.Prefix != wasm::WASM_FEATURE_PREFIX_USED &&
        Entry.Prefix != wasm::WASM_FEATURE_PREFIX_DISALLOWED)
      return;

    EmittedFeatures.push_back(Entry);
  };

  for (const SubtargetFeatureKV &KV : WebAssemblyFeatureKV) {
    EmitFeature(KV.Key);
  }
  // This pseudo-feature tells the linker whether shared memory would be safe
  EmitFeature("shared-mem");

  // This is an "architecture", not a "feature", but we emit it as such for
  // the benefit of tools like Binaryen and consistency with other producers.
  // FIXME: Subtarget is null here, so can't Subtarget->hasAddr64() ?
```
- **EN**: Implements helper routine(s) `StringRef`, `str`, `getModuleFlag` for this portion of the WebAssembly backend assembly / MC emission logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `StringRef`, `str`, `getModuleFlag`。 子目标特性裁剪会影响这里的行为。

### Lines 536-573

```cpp
  if (M.getDataLayout().getPointerSize() == 8) {
    // Can't use EmitFeature since "wasm-feature-memory64" is not a module
    // flag.
    EmittedFeatures.push_back({wasm::WASM_FEATURE_PREFIX_USED, "memory64"});
  }

  if (EmittedFeatures.size() == 0)
    return;

  // Emit features and linkage policies into the "target_features" section
  MCSectionWasm *FeaturesSection = OutContext.getWasmSection(
      ".custom_section.target_features", SectionKind::getMetadata());
  OutStreamer->pushSection();
  OutStreamer->switchSection(FeaturesSection);

  OutStreamer->emitULEB128IntValue(EmittedFeatures.size());
  for (auto &F : EmittedFeatures) {
    OutStreamer->emitIntValue(F.Prefix, 1);
    OutStreamer->emitULEB128IntValue(F.Name.size());
    OutStreamer->emitBytes(F.Name);
  }

  OutStreamer->popSection();
}

void WebAssemblyAsmPrinter::EmitFunctionAttributes(Module &M) {
  auto V = M.getNamedGlobal("llvm.global.annotations");
  if (!V)
    return;

  // Group all the custom attributes by name.
  MapVector<StringRef, SmallVector<MCSymbol *, 4>> CustomSections;
  const ConstantArray *CA = cast<ConstantArray>(V->getOperand(0));
  for (Value *Op : CA->operands()) {
    auto *CS = cast<ConstantStruct>(Op);
    // The first field is a pointer to the annotated variable.
    Value *AnnotatedVar = CS->getOperand(0)->stripPointerCasts();
    // Only annotated functions are supported for now.
```
- **EN**: Implements helper routine(s) `getDataLayout`, `getPointerSize`, `push_back` for this portion of the WebAssembly backend assembly / MC emission logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `getDataLayout`, `getPointerSize`, `push_back`。 子目标特性裁剪会影响这里的行为。

### Lines 574-611

```cpp
    if (!isa<Function>(AnnotatedVar))
      continue;
    auto *F = cast<Function>(AnnotatedVar);

    // The second field is a pointer to a global annotation string.
    auto *GV = cast<GlobalVariable>(CS->getOperand(1)->stripPointerCasts());
    StringRef AnnotationString;
    getConstantStringInfo(GV, AnnotationString);
    auto *Sym = static_cast<MCSymbolWasm *>(getSymbol(F));
    CustomSections[AnnotationString].push_back(Sym);
  }

  // Emit a custom section for each unique attribute.
  for (const auto &[Name, Symbols] : CustomSections) {
    MCSectionWasm *CustomSection = OutContext.getWasmSection(
        ".custom_section.llvm.func_attr.annotate." + Name, SectionKind::getMetadata());
    OutStreamer->pushSection();
    OutStreamer->switchSection(CustomSection);

    for (auto &Sym : Symbols) {
      OutStreamer->emitValue(
          MCSymbolRefExpr::create(Sym, WebAssembly::S_FUNCINDEX, OutContext),
          4);
    }
    OutStreamer->popSection();
  }
}

void WebAssemblyAsmPrinter::emitConstantPool() {
  emitDecls(*MMI->getModule());
  assert(MF->getConstantPool()->getConstants().empty() &&
         "WebAssembly disables constant pools");
}

void WebAssemblyAsmPrinter::emitJumpTableInfo() {
  // Nothing to do; jump tables are incorporated into the instruction stream.
}
```
- **EN**: Implements helper routine(s) `getOperand`, `stripPointerCasts`, `getConstantStringInfo` for this portion of the WebAssembly backend assembly / MC emission logic.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `getOperand`, `stripPointerCasts`, `getConstantStringInfo`。

### Lines 612-649

```cpp
void WebAssemblyAsmPrinter::emitFunctionBodyStart() {
  const Function &F = MF->getFunction();
  SmallVector<MVT, 1> ResultVTs;
  SmallVector<MVT, 4> ParamVTs;
  computeSignatureVTs(F.getFunctionType(), &F, F, TM, ParamVTs, ResultVTs);

  auto Signature = signatureFromMVTs(OutContext, ResultVTs, ParamVTs);
  auto *WasmSym = static_cast<MCSymbolWasm *>(CurrentFnSym);
  WasmSym->setSignature(Signature);
  WasmSym->setType(wasm::WASM_SYMBOL_TYPE_FUNCTION);

  getTargetStreamer()->emitFunctionType(WasmSym);

  // Emit the function index.
  if (MDNode *Idx = F.getMetadata("wasm.index")) {
    assert(Idx->getNumOperands() == 1);

    getTargetStreamer()->emitIndIdx(AsmPrinter::lowerConstant(
        cast<ConstantAsMetadata>(Idx->getOperand(0))->getValue()));
  }

  SmallVector<wasm::ValType, 16> Locals;
  valTypesFromMVTs(MFI->getLocals(), Locals);
  getTargetStreamer()->emitLocal(Locals);

  AsmPrinter::emitFunctionBodyStart();
}

void WebAssemblyAsmPrinter::emitInstruction(const MachineInstr *MI) {
  LLVM_DEBUG(dbgs() << "EmitInstruction: " << *MI << '\n');
  WebAssembly_MC::verifyInstructionPredicates(MI->getOpcode(),
                                              Subtarget->getFeatureBits());

  switch (MI->getOpcode()) {
  case WebAssembly::ARGUMENT_i32:
  case WebAssembly::ARGUMENT_i32_S:
  case WebAssembly::ARGUMENT_i64:
  case WebAssembly::ARGUMENT_i64_S:
```
- **EN**: Implements helper routine(s) `emitFunctionBodyStart`, `getFunction`, `computeSignatureVTs` for this portion of the WebAssembly backend assembly / MC emission logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `emitFunctionBodyStart`, `getFunction`, `computeSignatureVTs`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 650-687

```cpp
  case WebAssembly::ARGUMENT_f32:
  case WebAssembly::ARGUMENT_f32_S:
  case WebAssembly::ARGUMENT_f64:
  case WebAssembly::ARGUMENT_f64_S:
  case WebAssembly::ARGUMENT_v16i8:
  case WebAssembly::ARGUMENT_v16i8_S:
  case WebAssembly::ARGUMENT_v8i16:
  case WebAssembly::ARGUMENT_v8i16_S:
  case WebAssembly::ARGUMENT_v4i32:
  case WebAssembly::ARGUMENT_v4i32_S:
  case WebAssembly::ARGUMENT_v2i64:
  case WebAssembly::ARGUMENT_v2i64_S:
  case WebAssembly::ARGUMENT_v4f32:
  case WebAssembly::ARGUMENT_v4f32_S:
  case WebAssembly::ARGUMENT_v2f64:
  case WebAssembly::ARGUMENT_v2f64_S:
  case WebAssembly::ARGUMENT_v8f16:
  case WebAssembly::ARGUMENT_v8f16_S:
  case WebAssembly::ARGUMENT_externref:
  case WebAssembly::ARGUMENT_externref_S:
  case WebAssembly::ARGUMENT_funcref:
  case WebAssembly::ARGUMENT_funcref_S:
  case WebAssembly::ARGUMENT_exnref:
  case WebAssembly::ARGUMENT_exnref_S:
    // These represent values which are live into the function entry, so there's
    // no instruction to emit.
    break;
  case WebAssembly::FALLTHROUGH_RETURN: {
    // These instructions represent the implicit return at the end of a
    // function body.
    if (isVerbose()) {
      OutStreamer->AddComment("fallthrough-return");
      OutStreamer->addBlankLine();
    }
    break;
  }
  case WebAssembly::COMPILER_FENCE:
    // This is a compiler barrier that prevents instruction reordering during
```
- **EN**: Implements helper routine(s) `isVerbose`, `AddComment`, `addBlankLine` for this portion of the WebAssembly backend assembly / MC emission logic.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `isVerbose`, `AddComment`, `addBlankLine`。

### Lines 688-725

```cpp
    // backend compilation, and should not be emitted.
    break;
  case WebAssembly::CATCH:
  case WebAssembly::CATCH_S:
  case WebAssembly::CATCH_REF:
  case WebAssembly::CATCH_REF_S:
  case WebAssembly::CATCH_ALL:
  case WebAssembly::CATCH_ALL_S:
  case WebAssembly::CATCH_ALL_REF:
  case WebAssembly::CATCH_ALL_REF_S:
    // These are pseudo instructions to represent catch clauses in try_table
    // instruction to simulate block return values.
    break;
  default: {
    WebAssemblyMCInstLower MCInstLowering(OutContext, *this);
    MCInst TmpInst;
    MCInstLowering.lower(MI, TmpInst);
    EmitToStreamer(*OutStreamer, TmpInst);
    break;
  }
  }
}

bool WebAssemblyAsmPrinter::PrintAsmOperand(const MachineInstr *MI,
                                            unsigned OpNo,
                                            const char *ExtraCode,
                                            raw_ostream &OS) {
  // First try the generic code, which knows about modifiers like 'c' and 'n'.
  if (!AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, OS))
    return false;

  if (!ExtraCode) {
    const MachineOperand &MO = MI->getOperand(OpNo);
    switch (MO.getType()) {
    case MachineOperand::MO_Immediate:
      OS << MO.getImm();
      return false;
    case MachineOperand::MO_Register:
```
- **EN**: Implements helper routine(s) `MCInstLowering`, `lower`, `EmitToStreamer` for this portion of the WebAssembly backend assembly / MC emission logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `MCInstLowering`, `lower`, `EmitToStreamer`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 726-763

```cpp
      // FIXME: only opcode that still contains registers, as required by
      // MachineInstr::getDebugVariable().
      assert(MI->getOpcode() == WebAssembly::INLINEASM);
      OS << regToString(MO);
      return false;
    case MachineOperand::MO_GlobalAddress:
      PrintSymbolOperand(MO, OS);
      return false;
    case MachineOperand::MO_ExternalSymbol:
      GetExternalSymbolSymbol(MO.getSymbolName())->print(OS, MAI);
      printOffset(MO.getOffset(), OS);
      return false;
    case MachineOperand::MO_MachineBasicBlock:
      MO.getMBB()->getSymbol()->print(OS, MAI);
      return false;
    default:
      break;
    }
  }

  return true;
}

bool WebAssemblyAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
                                                  unsigned OpNo,
                                                  const char *ExtraCode,
                                                  raw_ostream &OS) {
  // The current approach to inline asm is that "r" constraints are expressed
  // as local indices, rather than values on the operand stack. This simplifies
  // using "r" as it eliminates the need to push and pop the values in a
  // particular order, however it also makes it impossible to have an "m"
  // constraint. So we don't support it.

  return AsmPrinter::PrintAsmMemoryOperand(MI, OpNo, ExtraCode, OS);
}

char WebAssemblyAsmPrinter::ID = 0;
```
- **EN**: Implements helper routine(s) `getDebugVariable`, `getOpcode`, `regToString` for this portion of the WebAssembly backend assembly / MC emission logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `getDebugVariable`, `getOpcode`, `regToString`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 764-772

```cpp
INITIALIZE_PASS(WebAssemblyAsmPrinter, "webassembly-asm-printer",
                "WebAssembly Assmebly Printer", false, false)

// Force static initialization.
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeWebAssemblyAsmPrinter() {
  RegisterAsmPrinter<WebAssemblyAsmPrinter> X(getTheWebAssemblyTarget32());
  RegisterAsmPrinter<WebAssemblyAsmPrinter> Y(getTheWebAssemblyTarget64());
}
```
- **EN**: Implements helper routine(s) `INITIALIZE_PASS`, `LLVMInitializeWebAssemblyAsmPrinter`, `X` for this portion of the WebAssembly backend assembly / MC emission logic.
- **CN**: 这里实现了 WebAssembly 后端该部分汇编与 MC 发射逻辑所需的辅助例程 `INITIALIZE_PASS`, `LLVMInitializeWebAssemblyAsmPrinter`, `X`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Assembly / MC emission logic / 汇编与 MC 发射逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyAsmPrinter.h`
- `MCTargetDesc/WebAssemblyMCAsmInfo.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `MCTargetDesc/WebAssemblyTargetStreamer.h`
- `TargetInfo/WebAssemblyTargetInfo.h`
- `Utils/WebAssemblyTypeUtilities.h`
- `WebAssembly.h`
- `WebAssemblyMCInstLower.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblyRegisterInfo.h`
- `WebAssemblyRuntimeLibcallSignatures.h`
- `WebAssemblyTargetMachine.h`
- `WebAssemblyUtilities.h`
- `llvm/ADT/MapVector.h`
- `llvm/ADT/SmallSet.h`
- `llvm/ADT/StringExtras.h`
- `llvm/Analysis/ValueTracking.h`
- `llvm/BinaryFormat/Wasm.h`
- `llvm/CodeGen/Analysis.h`
- `llvm/CodeGen/AsmPrinter.h`
- `llvm/CodeGen/MachineConstantPool.h`
- `llvm/CodeGen/MachineInstr.h`
- `llvm/CodeGen/MachineModuleInfoImpls.h`
- `llvm/IR/DataLayout.h`
- `llvm/IR/DebugInfoMetadata.h`
- `llvm/IR/GlobalVariable.h`
- `llvm/IR/Metadata.h`
- `llvm/IR/Module.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCSectionWasm.h`
- `llvm/MC/MCStreamer.h`
- `llvm/MC/MCSymbol.h`
- `llvm/MC/MCSymbolWasm.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/Compiler.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
