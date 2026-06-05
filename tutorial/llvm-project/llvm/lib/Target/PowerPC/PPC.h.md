# PPC.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPC.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPC.h - Top-level interface for PowerPC Target.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPC.h`，主要负责 PowerPC 后端的后端的公共或内部声明。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPC.h - Top-level interface for PowerPC Target ----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
//
// This file contains the entry points for global functions defined in the LLVM
// PowerPC back-end.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file contains the entry points for global functions defined in the LLVM".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file contains the entry points for global functions defined in the LLVM”。

### Lines 12-21

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_POWERPC_PPC_H
#define LLVM_LIB_TARGET_POWERPC_PPC_H

#include "llvm/Support/CodeGen.h"

// GCC #defines PPC on Linux but we use it as our namespace name
#undef PPC
```
- **EN**: Pulls in direct dependencies required by this public or internal declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端的公共或内部声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 22-28

```cpp
namespace llvm {
class PPCRegisterBankInfo;
class PPCSubtarget;
class PPCTargetMachine;
class PassRegistry;
class FunctionPass;
class InstructionSelector;
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. Subtarget feature gating influences the behavior here.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 子目标特性裁剪会影响这里的行为。

### Lines 29-35

```cpp
class MachineInstr;
class MachineOperand;
class AsmPrinter;
class MCInst;
class MCOperand;
class ModulePass;
```
- **EN**: Declares a backend-facing type `MachineInstr`, `MachineOperand`, `AsmPrinter` and outlines the API or state that nearby code will rely on. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明面向后端的类型 `MachineInstr`, `MachineOperand`, `AsmPrinter`，并勾勒出周边代码会依赖的接口或状态。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 36-61

```cpp
#ifndef NDEBUG
  FunctionPass *createPPCCTRLoopsVerify();
#endif
  FunctionPass *createPPCLoopInstrFormPrepPass(PPCTargetMachine &TM);
  FunctionPass *createPPCTOCRegDepsPass();
  FunctionPass *createPPCEarlyReturnPass();
  FunctionPass *createPPCVSXWACCCopyPass();
  FunctionPass *createPPCVSXFMAMutatePass();
  FunctionPass *createPPCVSXSwapRemovalPass();
  FunctionPass *createPPCReduceCRLogicalsPass();
  FunctionPass *createPPCMIPeepholePass();
  FunctionPass *createPPCBranchSelectionPass();
  FunctionPass *createPPCBranchCoalescingPass();
  FunctionPass *createPPCISelDag(PPCTargetMachine &TM, CodeGenOptLevel OL);
  FunctionPass *createPPCTLSDynamicCallPass();
  FunctionPass *createPPCBoolRetToIntPass();
  FunctionPass *createPPCExpandISELPass();
  FunctionPass *createPPCPreEmitPeepholePass();
  FunctionPass *createPPCExpandAtomicPseudoPass();
  FunctionPass *createPPCCTRLoopsPass();
  ModulePass *createPPCPrepareIFuncsOnAIXPass();
  void LowerPPCMachineInstrToMCInst(const MachineInstr *MI, MCInst &OutMI,
                                    AsmPrinter &AP);
  bool LowerPPCMachineOperandToMCOperand(const MachineOperand &MO,
                                         MCOperand &OutMO, AsmPrinter &AP);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 62-87

```cpp
#ifndef NDEBUG
  void initializePPCCTRLoopsVerifyPass(PassRegistry&);
#endif
  void initializePPCLoopInstrFormPrepPass(PassRegistry&);
  void initializePPCTOCRegDepsPass(PassRegistry&);
  void initializePPCEarlyReturnPass(PassRegistry&);
  void initializePPCVSXWACCCopyPass(PassRegistry &);
  void initializePPCVSXFMAMutatePass(PassRegistry&);
  void initializePPCVSXSwapRemovalPass(PassRegistry&);
  void initializePPCReduceCRLogicalsPass(PassRegistry&);
  void initializePPCBSelPass(PassRegistry&);
  void initializePPCBranchCoalescingPass(PassRegistry&);
  void initializePPCBoolRetToIntPass(PassRegistry&);
  void initializePPCExpandISELPass(PassRegistry &);
  void initializePPCPreEmitPeepholePass(PassRegistry &);
  void initializePPCTLSDynamicCallPass(PassRegistry &);
  void initializePPCMIPeepholePass(PassRegistry&);
  void initializePPCExpandAtomicPseudoPass(PassRegistry &);
  void initializePPCCTRLoopsPass(PassRegistry &);
  void initializePPCDAGToDAGISelLegacyPass(PassRegistry &);
  void initializePPCPrepareIFuncsOnAIXPass(PassRegistry &);
  void initializePPCLinuxAsmPrinterPass(PassRegistry &);
  void initializePPCAIXAsmPrinterPass(PassRegistry &);

  extern char &PPCVSXFMAMutateID;
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `initializePPCCTRLoopsVerifyPass`, `initializePPCLoopInstrFormPrepPass`, `initializePPCTOCRegDepsPass`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `initializePPCCTRLoopsVerifyPass`, `initializePPCLoopInstrFormPrepPass`, `initializePPCTOCRegDepsPass`。

### Lines 88-98

```cpp
  ModulePass *createPPCLowerMASSVEntriesPass();
  void initializePPCLowerMASSVEntriesPass(PassRegistry &);
  extern char &PPCLowerMASSVEntriesID;

  ModulePass *createPPCGenScalarMASSEntriesPass();
  void initializePPCGenScalarMASSEntriesPass(PassRegistry &);
  extern char &PPCGenScalarMASSEntriesID;

  InstructionSelector *
  createPPCInstructionSelector(const PPCTargetMachine &, const PPCSubtarget &,
                               const PPCRegisterBankInfo &);
```
- **EN**: Declares function entry points including `createPPCLowerMASSVEntriesPass`, `initializePPCLowerMASSVEntriesPass`, `createPPCGenScalarMASSEntriesPass` that other backend components call later. Subtarget feature gating influences the behavior here.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `createPPCLowerMASSVEntriesPass`, `initializePPCLowerMASSVEntriesPass`, `createPPCGenScalarMASSEntriesPass`。 子目标特性裁剪会影响这里的行为。

### Lines 99-102

```cpp
  namespace PPCII {

  /// Target Operand Flag enum.
  enum TOF {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Target Operand Flag enum.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Target Operand Flag enum.”。

### Lines 103-128

```cpp
    //===------------------------------------------------------------------===//
    // PPC Specific MachineOperand flags.
    MO_NO_FLAG,

    /// On PPC, the 12 bits are not enough for all target operand flags.
    /// Treat all PPC target flags as direct flags. To define new flag that is
    /// combination of other flags, add new enum entry instead of combining
    /// existing flags. See example MO_GOT_TPREL_PCREL_FLAG.

    /// On a symbol operand "FOO", this indicates that the reference is actually
    /// to "FOO@plt".  This is used for calls and jumps to external functions
    /// and for PIC calls on 32-bit ELF systems.
    MO_PLT,

    /// MO_PIC_FLAG - If this bit is set, the symbol reference is relative to
    /// the function's picbase, e.g. lo16(symbol-picbase).
    MO_PIC_FLAG,

    /// MO_PCREL_FLAG - If this bit is set, the symbol reference is relative to
    /// the current instruction address(pc), e.g., var@pcrel. Fixup is VK_PCREL.
    MO_PCREL_FLAG,

    /// MO_GOT_FLAG - If this bit is set the symbol reference is to be computed
    /// via the GOT. For example when combined with the MO_PCREL_FLAG it should
    /// produce the relocation @got@pcrel. Fixup is VK_GOT_PCREL.
    MO_GOT_FLAG,
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PPC Specific MachineOperand flags.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PPC Specific MachineOperand flags.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 129-154

```cpp

    /// MO_PCREL_OPT_FLAG - If this bit is set the operand is part of a
    /// PC Relative linker optimization.
    MO_PCREL_OPT_FLAG,

    /// MO_TLSGD_FLAG - If this bit is set the symbol reference is relative to
    /// TLS General Dynamic model for Linux and the variable offset of TLS
    /// General Dynamic model for AIX.
    MO_TLSGD_FLAG,

    /// MO_TPREL_FLAG - If this bit is set, the symbol reference is relative to
    /// the thread pointer and the symbol can be used for the TLS Initial Exec
    /// and Local Exec models.
    MO_TPREL_FLAG,

    /// MO_TLSLDM_FLAG - on AIX the ML relocation type is only valid for a
    /// reference to a TOC symbol from the symbol itself, and right now its only
    /// user is the symbol "_$TLSML". The symbol name is used to decide that
    /// the R_TLSML relocation is expected.
    MO_TLSLDM_FLAG,

    /// MO_TLSLD_FLAG - If this bit is set the symbol reference is relative to
    /// TLS Local Dynamic model.
    MO_TLSLD_FLAG,

    /// MO_TLSGDM_FLAG - If this bit is set the symbol reference is relative
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "MO_PCREL_OPT_FLAG - If this bit is set the operand is part of a".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“MO_PCREL_OPT_FLAG - If this bit is set the operand is part of a”。

### Lines 155-180

```cpp
    /// to the region handle of TLS General Dynamic model for AIX.
    MO_TLSGDM_FLAG,

    /// MO_GOT_TLSGD_PCREL_FLAG - A combintaion of flags, if these bits are set
    /// they should produce the relocation @got@tlsgd@pcrel.
    /// Fix up is VK_GOT_TLSGD_PCREL
    /// MO_GOT_TLSGD_PCREL_FLAG = MO_PCREL_FLAG | MO_GOT_FLAG | MO_TLSGD_FLAG,
    MO_GOT_TLSGD_PCREL_FLAG,

    /// MO_GOT_TLSLD_PCREL_FLAG - A combintaion of flags, if these bits are set
    /// they should produce the relocation @got@tlsld@pcrel.
    /// Fix up is VK_GOT_TLSLD_PCREL
    /// MO_GOT_TLSLD_PCREL_FLAG = MO_PCREL_FLAG | MO_GOT_FLAG | MO_TLSLD_FLAG,
    MO_GOT_TLSLD_PCREL_FLAG,

    /// MO_GOT_TPREL_PCREL_FLAG - A combintaion of flags, if these bits are set
    /// they should produce the relocation @got@tprel@pcrel.
    /// Fix up is VK_GOT_TPREL_PCREL
    /// MO_GOT_TPREL_PCREL_FLAG = MO_GOT_FLAG | MO_TPREL_FLAG | MO_PCREL_FLAG,
    MO_GOT_TPREL_PCREL_FLAG,

    /// MO_LO, MO_HA - lo16(symbol) and ha16(symbol)
    MO_LO,
    MO_HA,

    MO_TPREL_LO,
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "to the region handle of TLS General Dynamic model for AIX.". Notable symbols in this range include `lo16`, `ha16`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“to the region handle of TLS General Dynamic model for AIX.”。 该区间中较显眼的符号包括 `lo16`, `ha16`。

### Lines 181-206

```cpp
    MO_TPREL_HA,

    /// These values identify relocations on immediates folded
    /// into memory operations.
    MO_DTPREL_LO,
    MO_TLSLD_LO,
    MO_TOC_LO,

    /// Symbol for VK_TLS fixup attached to an ADD instruction
    MO_TLS,

    /// MO_PIC_HA_FLAG = MO_PIC_FLAG | MO_HA
    MO_PIC_HA_FLAG,

    /// MO_PIC_LO_FLAG = MO_PIC_FLAG | MO_LO
    MO_PIC_LO_FLAG,

    /// MO_TPREL_PCREL_FLAG = MO_PCREL_FLAG | MO_TPREL_FLAG
    MO_TPREL_PCREL_FLAG,

    /// MO_TPREL_PCREL_FLAG = MO_PCREL_FLAG | MO_TLS
    MO_TLS_PCREL_FLAG,

    /// MO_GOT_PCREL_FLAG = MO_PCREL_FLAG | MO_GOT_FLAG
    MO_GOT_PCREL_FLAG,
  };
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "These values identify relocations on immediates folded".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“These values identify relocations on immediates folded”。

### Lines 207-211

```cpp
  } // end namespace PPCII

} // end namespace llvm;

#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Public or internal declarations for the backend / 后端的公共或内部声明
- SelectionDAG lowering / SelectionDAG lowering
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `llvm/Support/CodeGen.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
