# X86AsmPrinter.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86AsmPrinter.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces and shared data structures for assembly emission in the core X86 backend. / 为X86 后端核心中的汇编输出声明接口与共享数据结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86AsmPrinter.h - X86 implementation of AsmPrinter ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_X86_X86ASMPRINTER_H
#define LLVM_LIB_TARGET_X86_X86ASMPRINTER_H

#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/FaultMaps.h"
#include "llvm/CodeGen/StackMaps.h"
#include "llvm/Passes/CodeGenPassBuilder.h"

// Implemented in X86MCInstLower.cpp
namespace {
  class X86MCInstLower;
}

namespace llvm {
class MCCodeEmitter;
class MCStreamer;
class X86Subtarget;
class TargetMachine;

class LLVM_LIBRARY_VISIBILITY X86AsmPrinter : public AsmPrinter {
public:
  static char ID;
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include X86MCInstLower, MCCodeEmitter, MCStreamer, X86Subtarget, TargetMachine, LLVM_LIBRARY_VISIBILITY. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 X86MCInstLower, MCCodeEmitter, MCStreamer, X86Subtarget, TargetMachine, LLVM_LIBRARY_VISIBILITY。这些内容定义了实现文件所依赖的契约。

### Lines 31-60: Type declarations and shared abstractions / 类型声明与共享抽象
```cpp

private:
  const X86Subtarget *Subtarget = nullptr;
  FaultMaps FM;
  std::unique_ptr<MCCodeEmitter> CodeEmitter;
  bool EmitFPOData = false;
  bool ShouldEmitWeakSwiftAsyncExtendedFramePointerFlags = false;
  bool IndCSPrefix = false;
  bool EnableImportCallOptimization = false;
  bool SplitChainedAtEndOfBlock = false;

  enum ImportCallKind : unsigned {
    IMAGE_RETPOLINE_AMD64_IMPORT_BR = 0x02,
    IMAGE_RETPOLINE_AMD64_IMPORT_CALL = 0x03,
    IMAGE_RETPOLINE_AMD64_INDIR_BR = 0x04,
    IMAGE_RETPOLINE_AMD64_INDIR_CALL = 0x05,
    IMAGE_RETPOLINE_AMD64_INDIR_BR_REX = 0x06,
    IMAGE_RETPOLINE_AMD64_CFG_BR = 0x08,
    IMAGE_RETPOLINE_AMD64_CFG_CALL = 0x09,
    IMAGE_RETPOLINE_AMD64_CFG_BR_REX = 0x0A,
    IMAGE_RETPOLINE_AMD64_SWITCHTABLE_FIRST = 0x010,
    IMAGE_RETPOLINE_AMD64_SWITCHTABLE_LAST = 0x01F,
  };
  struct ImportCallInfo {
    MCSymbol *CalleeSymbol;
    ImportCallKind Kind;
  };
  DenseMap<MCSection *, std::vector<ImportCallInfo>>
      SectionToImportedFunctionCalls;

```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include ImportCallKind, ImportCallInfo. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 ImportCallKind, ImportCallInfo。这些内容定义了实现文件所依赖的契约。

### Lines 61-90: Type declarations and shared abstractions / 类型声明与共享抽象
```cpp
  // This utility class tracks the length of a stackmap instruction's 'shadow'.
  // It is used by the X86AsmPrinter to ensure that the stackmap shadow
  // invariants (i.e. no other stackmaps, patchpoints, or control flow within
  // the shadow) are met, while outputting a minimal number of NOPs for padding.
  //
  // To minimise the number of NOPs used, the shadow tracker counts the number
  // of instruction bytes output since the last stackmap. Only if there are too
  // few instruction bytes to cover the shadow are NOPs used for padding.
  class StackMapShadowTracker {
  public:
    void startFunction(MachineFunction &MF) {
      this->MF = &MF;
    }
    void count(const MCInst &Inst, const MCSubtargetInfo &STI,
               MCCodeEmitter *CodeEmitter);

    // Called to signal the start of a shadow of RequiredSize bytes.
    void reset(unsigned RequiredSize) {
      RequiredShadowSize = RequiredSize;
      CurrentShadowSize = 0;
      InShadow = true;
    }

    // Called before every stackmap/patchpoint, and at the end of basic blocks,
    // to emit any necessary padding-NOPs.
    void emitShadowPadding(MCStreamer &OutStreamer, const MCSubtargetInfo &STI);
  private:
    const MachineFunction *MF = nullptr;
    bool InShadow = false;

```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include StackMapShadowTracker. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 StackMapShadowTracker。这些内容定义了实现文件所依赖的契约。

### Lines 91-120: Comments and explanatory notes / 注释与说明性文字
```cpp
    // RequiredShadowSize holds the length of the shadow specified in the most
    // recently encountered STACKMAP instruction.
    // CurrentShadowSize counts the number of bytes encoded since the most
    // recently encountered STACKMAP, stopping when that number is greater than
    // or equal to RequiredShadowSize.
    unsigned RequiredShadowSize = 0, CurrentShadowSize = 0;
  };

  StackMapShadowTracker SMShadowTracker;

  // All instructions emitted by the X86AsmPrinter should use this helper
  // method.
  //
  // This helper function invokes the SMShadowTracker on each instruction before
  // outputting it to the OutStream. This allows the shadow tracker to minimise
  // the number of NOPs used for stackmap padding.
  void EmitAndCountInstruction(MCInst &Inst);
  void LowerSTACKMAP(const MachineInstr &MI);
  void LowerPATCHPOINT(const MachineInstr &MI, X86MCInstLower &MCIL);
  void LowerSTATEPOINT(const MachineInstr &MI, X86MCInstLower &MCIL);
  void LowerFAULTING_OP(const MachineInstr &MI, X86MCInstLower &MCIL);
  void LowerPATCHABLE_OP(const MachineInstr &MI, X86MCInstLower &MCIL);

  void LowerTlsAddr(X86MCInstLower &MCInstLowering, const MachineInstr &MI);

  // XRay-specific lowering for X86.
  void LowerPATCHABLE_FUNCTION_ENTER(const MachineInstr &MI,
                                     X86MCInstLower &MCIL);
  void LowerPATCHABLE_RET(const MachineInstr &MI, X86MCInstLower &MCIL);
  void LowerPATCHABLE_TAIL_CALL(const MachineInstr &MI, X86MCInstLower &MCIL);
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 121-150: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp
  void LowerPATCHABLE_EVENT_CALL(const MachineInstr &MI, X86MCInstLower &MCIL);
  void LowerPATCHABLE_TYPED_EVENT_CALL(const MachineInstr &MI,
                                       X86MCInstLower &MCIL);

  void LowerFENTRY_CALL(const MachineInstr &MI, X86MCInstLower &MCIL);

  // KCFI specific lowering for X86.
  uint32_t MaskKCFIType(uint32_t Value);
  void EmitKCFITypePadding(const MachineFunction &MF, bool HasType = true);
  void LowerKCFI_CHECK(const MachineInstr &MI);

  // Address sanitizer specific lowering for X86.
  void LowerASAN_CHECK_MEMACCESS(const MachineInstr &MI);

  // Choose between emitting .seh_ directives and .cv_fpo_ directives.
  void EmitSEHInstruction(const MachineInstr *MI);

  void PrintSymbolOperand(const MachineOperand &MO, raw_ostream &O) override;
  void PrintOperand(const MachineInstr *MI, unsigned OpNo, raw_ostream &O);
  void PrintModifiedOperand(const MachineInstr *MI, unsigned OpNo,
                            raw_ostream &O, StringRef Modifier = {});
  void PrintPCRelImm(const MachineInstr *MI, unsigned OpNo, raw_ostream &O);
  void PrintLeaMemReference(const MachineInstr *MI, unsigned OpNo,
                            raw_ostream &O, StringRef Modifier = {});
  void PrintMemReference(const MachineInstr *MI, unsigned OpNo, raw_ostream &O,
                         StringRef Modifier = {});
  void PrintIntelMemReference(const MachineInstr *MI, unsigned OpNo,
                              raw_ostream &O, StringRef Modifier = {});
  const MCSubtargetInfo *getIFuncMCSubtargetInfo() const override;
  void emitMachOIFuncStubBody(Module &M, const GlobalIFunc &GI,
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 151-180: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp
                              MCSymbol *LazyPointer) override;
  void emitMachOIFuncStubHelperBody(Module &M, const GlobalIFunc &GI,
                                    MCSymbol *LazyPointer) override;

  void emitCallInstruction(const llvm::MCInst &MCI);
  void maybeEmitNopAfterCallForWindowsEH(const MachineInstr *MI);

  // Emits a label to mark the next instruction as being relevant to Import Call
  // Optimization.
  void emitLabelAndRecordForImportCallOptimization(ImportCallKind Kind);

public:
  X86AsmPrinter(TargetMachine &TM, std::unique_ptr<MCStreamer> Streamer);

  StringRef getPassName() const override {
    return "X86 Assembly Printer";
  }

  const X86Subtarget &getSubtarget() const { return *Subtarget; }

  void emitStartOfAsmFile(Module &M) override;

  void emitEndOfAsmFile(Module &M) override;

  void emitInstruction(const MachineInstr *MI) override;

  void emitInlineAsmEnd(const MCSubtargetInfo &StartInfo,
                        const MCSubtargetInfo *EndInfo,
                        const MachineInstr *MI) override;

```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 181-210: Type declarations and shared abstractions / 类型声明与共享抽象
```cpp
  void emitBasicBlockEnd(const MachineBasicBlock &MBB) override;

  bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                       const char *ExtraCode, raw_ostream &O) override;
  bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNo,
                             const char *ExtraCode, raw_ostream &O) override;

  bool doInitialization(Module &M) override {
    SMShadowTracker.reset(0);
    SM.reset();
    FM.reset();
    return AsmPrinter::doInitialization(M);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
  void emitFunctionBodyStart() override;
  void emitFunctionBodyEnd() override;
  void emitKCFITypeId(const MachineFunction &MF) override;

  bool shouldEmitWeakSwiftAsyncExtendedFramePointerFlags() const override {
    return ShouldEmitWeakSwiftAsyncExtendedFramePointerFlags;
  }

  std::function<ProfileSummaryInfo *(Module &)> GetPSI;
  std::function<StaticDataProfileInfo *(Module &)> GetSDPI;
};

class X86AsmPrinterBeginPass
    : public OptionalPassInfoMixin<X86AsmPrinterBeginPass> {
public:
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include X86AsmPrinterBeginPass. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 X86AsmPrinterBeginPass。这些内容定义了实现文件所依赖的契约。

### Lines 211-229: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
  PreservedAnalyses run(Module &M, ModuleAnalysisManager &MAM);
};

class X86AsmPrinterPass : public RequiredPassInfoMixin<X86AsmPrinterPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
  // AsmPrinter needs to run regardless of optimization level.
};

class X86AsmPrinterEndPass
    : public OptionalPassInfoMixin<X86AsmPrinterEndPass> {
public:
  PreservedAnalyses run(Module &M, ModuleAnalysisManager &MAM);
};

} // end namespace llvm

#endif
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include X86AsmPrinterPass, X86AsmPrinterEndPass. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 X86AsmPrinterPass, X86AsmPrinterEndPass。这些内容定义了实现文件所依赖的契约。

## Key Concepts / 关键概念
- Primary topic: assembly emission. / 核心主题：汇编输出。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86MCInstLower, MCCodeEmitter, MCStreamer, X86Subtarget, TargetMachine, LLVM_LIBRARY_VISIBILITY. / 重要符号：X86MCInstLower, MCCodeEmitter, MCStreamer, X86Subtarget, TargetMachine, LLVM_LIBRARY_VISIBILITY。
- The file emphasizes declarations, interfaces, and reusable helpers. / 该文件以声明、接口和可复用辅助逻辑为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: llvm/CodeGen/AsmPrinter.h, llvm/CodeGen/FaultMaps.h, llvm/CodeGen/StackMaps.h, llvm/Passes/CodeGenPassBuilder.h. / 直接包含：llvm/CodeGen/AsmPrinter.h, llvm/CodeGen/FaultMaps.h, llvm/CodeGen/StackMaps.h, llvm/Passes/CodeGenPassBuilder.h。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
- Implementation files in the same subsystem rely on these declarations. / 同一子系统中的实现文件依赖这些声明。
