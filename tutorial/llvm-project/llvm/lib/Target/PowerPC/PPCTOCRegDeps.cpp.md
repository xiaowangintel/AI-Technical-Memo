# PPCTOCRegDeps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCTOCRegDeps.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCTOCRegDeps.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCTOCRegDeps.cpp - Add Extra TOC Register Dependencies -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-44

```cpp
//===----------------------------------------------------------------------===//
//
// When resolving an address using the ELF ABI TOC pointer, two relocations are
// generally required: one for the high part and one for the low part. Only
// the high part generally explicitly depends on r2 (the TOC pointer). And, so,
// we might produce code like this:
//
// .Ltmp526:
//         addis 3, 2, .LC12@toc@ha
// .Ltmp1628:
//         std 2, 40(1)
//         ld 5, 0(27)
//         ld 2, 8(27)
//         ld 11, 16(27)
//         ld 3, .LC12@toc@l(3)
//         rldicl 4, 4, 0, 32
//         mtctr 5
//         bctrl
//         ld 2, 40(1)
//
// And there is nothing wrong with this code, as such, but there is a linker bug
// in binutils (https://sourceware.org/bugzilla/show_bug.cgi?id=18414) that will
// misoptimize this code sequence to this:
//         nop
//         std     r2,40(r1)
//         ld      r5,0(r27)
//         ld      r2,8(r27)
//         ld      r11,16(r27)
//         ld      r3,-32472(r2)
//         clrldi  r4,r4,32
//         mtctr   r5
//         bctrl
//         ld      r2,40(r1)
// because the linker does not know (and does not check) that the value in r2
// changed in between the instruction using the .LC12@toc@ha (TOC-relative)
// relocation and the instruction using the .LC12@toc@l(3) relocation.
// Because it finds these instructions using the relocations (and not by
// scanning the instructions), it has been asserted that there is no good way
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "When resolving an address using the ELF ABI TOC pointer, two relocations are". Notable symbols in this range include `r2`, `l`, `binutils`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“When resolving an address using the ELF ABI TOC pointer, two relocations are”。 该区间中较显眼的符号包括 `r2`, `l`, `binutils`。

### Lines 45-60

```cpp
// to detect the change of r2 in between. As a result, this bug may never be
// fixed (i.e. it may become part of the definition of the ABI). GCC was
// updated to add extra dependencies on r2 to instructions using the @toc@l
// relocations to avoid this problem, and we'll do the same here.
//
// This is done as a separate pass because:
//  1. These extra r2 dependencies are not really properties of the
//     instructions, but rather due to a linker bug, and maybe one day we'll be
//     able to get rid of them when targeting linkers without this bug (and,
//     thus, keeping the logic centralized here will make that
//     straightforward).
//  2. There are ISel-level peephole optimizations that propagate the @toc@l
//     relocations to some user instructions, and so the exta dependencies do
//     not apply only to a fixed set of instructions (without undesirable
//     definition replication).
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "to detect the change of r2 in between. As a result, this bug may never be". Notable symbols in this range include `fixed`, `bug`, `instructions`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“to detect the change of r2 in between. As a result, this bug may never be”。 该区间中较显眼的符号包括 `fixed`, `bug`, `instructions`。

### Lines 61-67

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCInstrInfo.h"
#include "PPCTargetMachine.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/Statistic.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 68-74

```cpp
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/Support/ErrorHandling.h"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 75-112

```cpp
#define DEBUG_TYPE "ppc-toc-reg-deps"

namespace {
  // PPCTOCRegDeps pass - For simple functions without epilogue code, move
  // returns up, and create conditional returns, to avoid unnecessary
  // branch-to-blr sequences.
  struct PPCTOCRegDeps : public MachineFunctionPass {
    static char ID;
    PPCTOCRegDeps() : MachineFunctionPass(ID) {}

  protected:
    bool hasTOCLoReloc(const MachineInstr &MI) {
      if (MI.getOpcode() == PPC::LDtocL || MI.getOpcode() == PPC::ADDItocL8 ||
          MI.getOpcode() == PPC::LWZtocL)
        return true;

      for (const MachineOperand &MO : MI.operands()) {
        if (MO.getTargetFlags() == PPCII::MO_TOC_LO)
          return true;
      }

      return false;
    }

    bool processBlock(MachineBasicBlock &MBB) {
      bool Changed = false;

      const bool isPPC64 =
          MBB.getParent()->getSubtarget<PPCSubtarget>().isPPC64();
      const unsigned TOCReg = isPPC64 ? PPC::X2 : PPC::R2;

      for (auto &MI : MBB) {
        if (!hasTOCLoReloc(MI))
          continue;

        MI.addOperand(MachineOperand::CreateReg(TOCReg,
                                                false  /*IsDef*/,
                                                true  /*IsImp*/));
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 113-142

```cpp
        Changed = true;
      }

      return Changed;
    }

public:
    bool runOnMachineFunction(MachineFunction &MF) override {
      bool Changed = false;

      for (MachineBasicBlock &B : llvm::make_early_inc_range(MF))
        if (processBlock(B))
          Changed = true;

      return Changed;
    }

    void getAnalysisUsage(AnalysisUsage &AU) const override {
      MachineFunctionPass::getAnalysisUsage(AU);
    }
  };
}

INITIALIZE_PASS(PPCTOCRegDeps, DEBUG_TYPE,
                "PowerPC TOC Register Dependencies", false, false)

char PPCTOCRegDeps::ID = 0;
FunctionPass*
llvm::createPPCTOCRegDepsPass() { return new PPCTOCRegDeps(); }
```
- **EN**: Implements helper routine(s) `runOnMachineFunction`, `make_early_inc_range`, `processBlock` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `runOnMachineFunction`, `make_early_inc_range`, `processBlock`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Object format integration / 目标文件格式集成
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCInstrInfo.h`
- `PPCTargetMachine.h`
- `llvm/ADT/STLExtras.h`
- `llvm/ADT/Statistic.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstr.h`
- `llvm/Support/ErrorHandling.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
