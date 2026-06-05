# LVLGen.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/LVLGen.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements VE-specific helper generation for vector-length or vector-lane related operations.
  - **CN**: 实现 VE 中与向量长度或向量通道相关操作的辅助生成逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- LVLGen.cpp - LVL instruction generator ----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-14
```cpp

#include "VE.h"
#include "VESubtarget.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/TargetInstrInfo.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `VE.h`, `VESubtarget.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VE.h`, `VESubtarget.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`。

### Lines 15-23
```cpp
using namespace llvm;

#define DEBUG_TYPE "lvl-gen"

namespace {
struct LVLGen : public MachineFunctionPass {
  const TargetInstrInfo *TII;
  const TargetRegisterInfo *TRI;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 24-28
```cpp
  static char ID;
  LVLGen() : MachineFunctionPass(ID) {}
  bool runOnMachineBasicBlock(MachineBasicBlock &MBB);
  bool runOnMachineFunction(MachineFunction &F) override;

```
- **EN**: Implements logic around `LVLGen`, `runOnMachineBasicBlock`, `runOnMachineFunction`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LVLGen`, `runOnMachineBasicBlock`, `runOnMachineFunction` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 29-33
```cpp
  unsigned getVL(const MachineInstr &MI);
  int getVLIndex(unsigned Opcode);
};
char LVLGen::ID = 0;

```
- **EN**: Implements logic around `getVL`, `getVLIndex`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getVL`, `getVLIndex` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 34-40
```cpp
} // end of anonymous namespace

FunctionPass *llvm::createLVLGenPass() { return new LVLGen; }

int LVLGen::getVLIndex(unsigned Opcode) {
  const MCInstrDesc &MCID = TII->get(Opcode);

```
- **EN**: Implements logic around `createLVLGenPass`, `getVLIndex`, `get`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createLVLGenPass`, `getVLIndex`, `get` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 41-47
```cpp
  // If an instruction has VLIndex information, return it.
  if (HAS_VLINDEX(MCID.TSFlags))
    return GET_VLINDEX(MCID.TSFlags);

  return -1;
}

```
- **EN**: Implements logic around `GET_VLINDEX`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `GET_VLINDEX` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 48-54
```cpp
// returns a register holding a vector length. NoRegister is returned when
// this MI does not have a vector length.
unsigned LVLGen::getVL(const MachineInstr &MI) {
  int Index = getVLIndex(MI.getOpcode());
  if (Index >= 0)
    return MI.getOperand(Index).getReg();

```
- **EN**: Implements logic around `getVL`, `getVLIndex`, `getOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getVL`, `getVLIndex`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 55-61
```cpp
  return VE::NoRegister;
}

bool LVLGen::runOnMachineBasicBlock(MachineBasicBlock &MBB) {
#define RegName(no)                                                            \
  (MBB.getParent()->getSubtarget<VESubtarget>().getRegisterInfo()->getName(no))

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 62-68
```cpp
  bool Changed = false;
  bool HasRegForVL = false;
  unsigned RegForVL;

  for (MachineBasicBlock::iterator I = MBB.begin(); I != MBB.end();) {
    MachineBasicBlock::iterator MI = I;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 69-78
```cpp
    // Check whether MI uses a vector length operand.  If so, we prepare for VL
    // register.  We would like to reuse VL register as much as possible.  We
    // also would like to keep the number of LEA instructions as fewer as
    // possible.  Therefore, we use a regular scalar register to hold immediate
    // values to load VL register.  And try to reuse identical scalar registers
    // to avoid new LVLr instructions as much as possible.
    unsigned Reg = getVL(*MI);
    if (Reg != VE::NoRegister) {
      LLVM_DEBUG(dbgs() << "Vector instruction found: ");
      LLVM_DEBUG(MI->dump());
```
- **EN**: Implements logic around `getVL`; this block applies conditional target rules.
- **CN**: 围绕 `getVL` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 79-83
```cpp
      LLVM_DEBUG(dbgs() << "Vector length is " << RegName(Reg) << ". ");
      LLVM_DEBUG(dbgs() << "Current VL is "
                        << (HasRegForVL ? RegName(RegForVL) : "unknown")
                        << ". ");

```
- **EN**: Implements logic around `RegName`.
- **CN**: 围绕 `RegName` 实现具体逻辑。

### Lines 84-93
```cpp
      if (!HasRegForVL || RegForVL != Reg) {
        // Use VL, but a different value in a different scalar register.
        // So, generate new LVL instruction just before the current instruction.
        LLVM_DEBUG(dbgs() << "Generate a LVL instruction to load "
                          << RegName(Reg) << ".\n");
        BuildMI(MBB, I, MI->getDebugLoc(), TII->get(VE::LVLr)).addReg(Reg);
        HasRegForVL = true;
        RegForVL = Reg;
        Changed = true;
      } else {
```
- **EN**: Implements logic around `RegName`, `BuildMI`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `RegName`, `BuildMI` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 94-103
```cpp
        LLVM_DEBUG(dbgs() << "Reuse current VL.\n");
      }
    }
    // Check the update of a given scalar register holding an immediate value
    // for VL register.  Also, a call doesn't preserve VL register.
    if (HasRegForVL) {
      if (MI->definesRegister(RegForVL, TRI) ||
          MI->modifiesRegister(RegForVL, TRI) ||
          MI->killsRegister(RegForVL, TRI) || MI->isCall()) {
        // The latest VL is needed to be updated, so disable HasRegForVL.
```
- **EN**: Implements logic around `modifiesRegister`, `killsRegister`; this block applies conditional target rules.
- **CN**: 围绕 `modifiesRegister`, `killsRegister` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 104-109
```cpp
        LLVM_DEBUG(dbgs() << RegName(RegForVL) << " is needed to be updated: ");
        LLVM_DEBUG(MI->dump());
        HasRegForVL = false;
      }
    }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 110-114
```cpp
    ++I;
  }
  return Changed;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 115-119
```cpp
bool LVLGen::runOnMachineFunction(MachineFunction &F) {
  LLVM_DEBUG(dbgs() << "********** Begin LVLGen **********\n");
  LLVM_DEBUG(dbgs() << "********** Function: " << F.getName() << '\n');
  LLVM_DEBUG(F.dump());

```
- **EN**: Implements logic around `runOnMachineFunction`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineFunction` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 120-125
```cpp
  bool Changed = false;

  const VESubtarget &Subtarget = F.getSubtarget<VESubtarget>();
  TII = Subtarget.getInstrInfo();
  TRI = Subtarget.getRegisterInfo();

```
- **EN**: Implements logic around `getSubtarget<VESubtarget>`, `getInstrInfo`, `getRegisterInfo`.
- **CN**: 围绕 `getSubtarget<VESubtarget>`, `getInstrInfo`, `getRegisterInfo` 实现具体逻辑。

### Lines 126-135
```cpp
  for (MachineBasicBlock &MBB : F)
    Changed |= runOnMachineBasicBlock(MBB);

  if (Changed) {
    LLVM_DEBUG(dbgs() << "\n");
    LLVM_DEBUG(F.dump());
  }
  LLVM_DEBUG(dbgs() << "********** End LVLGen **********\n");
  return Changed;
}
```
- **EN**: Implements logic around `runOnMachineBasicBlock`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineBasicBlock` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

## Key Concepts / 关键概念

- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VE.h`, `VESubtarget.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/TargetInstrInfo.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_VLINDEX`
