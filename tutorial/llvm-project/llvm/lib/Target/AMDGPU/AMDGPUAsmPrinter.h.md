# AMDGPUAsmPrinter.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUAsmPrinter.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUAsmPrinter in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUAsmPrinter 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===-- AMDGPUAsmPrinter.h - Print AMDGPU assembly code ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// AMDGPU Assembly printer class.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUASMPRINTER_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUASMPRINTER_H

#include "AMDGPUMCResourceInfo.h"
#include "AMDGPUResourceUsageAnalysis.h"
#include "MCTargetDesc/AMDGPUTargetStreamer.h"
#include "SIProgramInfo.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/CodeGen/AsmPrinter.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 24-44: Declares class AMDGPUMachineFunctionInfo
```cpp
namespace llvm {

class AMDGPUMachineFunctionInfo;
class AMDGPUResourceUsageAnalysis;
class AMDGPUTargetStreamer;
class MCCodeEmitter;
class MCOperand;
class MCResourceInfo;

namespace AMDGPU {
struct MCKernelDescriptor;
struct AMDGPUMCKernelCodeT;
namespace HSAMD {
class MetadataStreamer;
}
} // namespace AMDGPU

class AMDGPUAsmPrinter final : public AsmPrinter {
public:
  static char ID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUMachineFunctionInfo`, `AMDGPUResourceUsageAnalysis`, `AMDGPUTargetStreamer`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUMachineFunctionInfo`, `AMDGPUResourceUsageAnalysis`, `AMDGPUTargetStreamer`。

### Lines 45-67: Declares initializeTargetID
```cpp
private:
  unsigned CodeObjectVersion;
  void initializeTargetID(const Module &M);

  const AMDGPUResourceUsageAnalysisWrapperPass::FunctionResourceInfo
      *ResourceUsage;

  MCResourceInfo RI;

  SIProgramInfo CurrentProgramInfo;

  std::unique_ptr<AMDGPU::HSAMD::MetadataStreamer> HSAMetadataStream;

  MCCodeEmitter *DumpCodeInstEmitter = nullptr;

  // When appropriate, add a _dvgpr$ symbol.
  void emitDVgprSymbol(MachineFunction &MF);

  void getSIProgramInfo(SIProgramInfo &Out, const MachineFunction &MF);
  void getAmdKernelCode(AMDGPU::AMDGPUMCKernelCodeT &Out,
                        const SIProgramInfo &KernelInfo,
                        const MachineFunction &MF) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 68-91: Defines EmitProgramInfoSI
```cpp
  /// Emit register usage information so that the GPU driver
  /// can correctly setup the GPU state.
  void EmitProgramInfoSI(const MachineFunction &MF,
                         const SIProgramInfo &KernelInfo);
  void EmitPALMetadata(const MachineFunction &MF,
                       const SIProgramInfo &KernelInfo);
  void emitPALFunctionMetadata(const MachineFunction &MF);
  void emitCommonFunctionComments(const MCExpr *NumVGPR, const MCExpr *NumAGPR,
                                  const MCExpr *TotalNumVGPR,
                                  const MCExpr *NumSGPR,
                                  const MCExpr *ScratchSize, uint64_t CodeSize,
                                  const AMDGPUMachineFunctionInfo *MFI);
  void emitResourceUsageRemarks(const MachineFunction &MF,
                                const SIProgramInfo &CurrentProgramInfo,
                                bool isModuleEntryFunction, bool hasMAIInsts);

  const MCExpr *getAmdhsaKernelCodeProperties(const MachineFunction &MF) const;

  AMDGPU::MCKernelDescriptor
  getAmdhsaKernelDescriptor(const MachineFunction &MF,
                            const SIProgramInfo &PI) const;

  void initTargetStreamer(Module &M);

```
**EN:** This section contains concrete logic for EmitProgramInfoSI. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 EmitProgramInfoSI 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 92-115: Declares emitAMDGPUInfo
```cpp
  void emitAMDGPUInfo(Module &M);
  void collectCallEdge(const MachineInstr &MI);

  SetVector<std::pair<MCSymbol *, MCSymbol *>> DirectCallEdges;

  SmallVector<AMDGPU::FuncInfo, 8> FunctionInfos;

  SmallString<128> getMCExprStr(const MCExpr *Value);

  /// Attempts to replace the validation that is missed in getSIProgramInfo due
  /// to MCExpr being unknown. Invoked during doFinalization such that the
  /// MCResourceInfo symbols are known.
  void validateMCResourceInfo(Function &F);

public:
  explicit AMDGPUAsmPrinter(TargetMachine &TM,
                            std::unique_ptr<MCStreamer> Streamer);

  StringRef getPassName() const override;

  const MCSubtargetInfo* getGlobalSTI() const;

  AMDGPUTargetStreamer* getTargetStreamer() const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 116-138: Declares doInitialization
```cpp
  bool doInitialization(Module &M) override;
  bool doFinalization(Module &M) override;
  bool runOnMachineFunction(MachineFunction &MF) override;

  /// Wrapper for MCInstLowering.lowerOperand() for the tblgen'erated
  /// pseudo lowering.
  bool lowerOperand(const MachineOperand &MO, MCOperand &MCOp) const;

  /// Lower the specified LLVM Constant to an MCExpr.
  /// The AsmPrinter::lowerConstantof does not know how to lower
  /// addrspacecast, therefore they should be lowered by this function.
  const MCExpr *lowerConstant(const Constant *CV, const Constant *BaseCV,
                              uint64_t Offset) override;

  /// tblgen'erated driver function for lowering simple MI->MC pseudo
  /// instructions.
  bool lowerPseudoInstExpansion(const MachineInstr *MI, MCInst &Inst);

  /// Implemented in AMDGPUMCInstLower.cpp
  void emitInstruction(const MachineInstr *MI) override;

  void emitFunctionBodyStart() override;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 139-158: Declares endFunction
```cpp
  void endFunction(const MachineFunction *MF);

  void emitImplicitDef(const MachineInstr *MI) const override;

  void emitFunctionEntryLabel() override;

  void emitBasicBlockStart(const MachineBasicBlock &MBB) override;

  void emitGlobalVariable(const GlobalVariable *GV) override;

  void emitStartOfAsmFile(Module &M) override;

  void emitEndOfAsmFile(Module &M) override;

  bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                       const char *ExtraCode, raw_ostream &O) override;

protected:
  void getAnalysisUsage(AnalysisUsage &AU) const override;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 159-166: Preprocessor guards and macros
```cpp
  std::vector<std::string> DisasmLines, HexLines;
  size_t DisasmLineMaxLen;
  bool IsTargetStreamerInitialized;
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_AMDGPUASMPRINTER_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUMachineFunctionInfo`, `AMDGPUResourceUsageAnalysis`, `AMDGPUTargetStreamer`, `MCCodeEmitter`, `MCOperand`, `MCResourceInfo`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUMCResourceInfo.h"`
- `"AMDGPUResourceUsageAnalysis.h"`
- `"MCTargetDesc/AMDGPUTargetStreamer.h"`
- `"SIProgramInfo.h"`
- `"llvm/ADT/SetVector.h"`
- `"llvm/CodeGen/AsmPrinter.h"`
