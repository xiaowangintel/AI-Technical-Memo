# HexagonAsmPrinter.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonAsmPrinter.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): XRay implementation
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===- HexagonAsmPrinter.h - Print machine code -----------------*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // Hexagon Assembly printer class.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONASMPRINTER_H
    14: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONASMPRINTER_H
    15: 
    16: #include "HexagonSubtarget.h"
    17: #include "llvm/CodeGen/AsmPrinter.h"
    18: #include "llvm/CodeGen/MachineFunction.h"
    19: #include "llvm/MC/MCStreamer.h"
    20: #include <utility>
    21: 
    22: namespace llvm {
    23: 
    24: class MachineInstr;
    25: class MCInst;
```
- EN: It imports headers such as HexagonSubtarget.h, llvm/CodeGen/AsmPrinter.h, llvm/CodeGen/MachineFunction.h, llvm/MC/MCStreamer.h, ... (5 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as MachineInstr, MCInst, which carry the state or API of this component.
- CN: 这里引入了 HexagonSubtarget.h, llvm/CodeGen/AsmPrinter.h, llvm/CodeGen/MachineFunction.h, llvm/MC/MCStreamer.h, ... (5 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 MachineInstr, MCInst 等类型，用来承载该组件的状态或接口。

### Lines 26-50 / 第 26-50 行

```cpp
    26: class raw_ostream;
    27: class TargetMachine;
    28: 
    29:   class HexagonAsmPrinter : public AsmPrinter {
    30:   public:
    31:     static char ID;
    32: 
    33:   private:
    34:     const HexagonSubtarget *Subtarget = nullptr;
    35: 
    36:     void emitAttributes();
    37: 
    38:   public:
    39:     explicit HexagonAsmPrinter(TargetMachine &TM,
    40:                                std::unique_ptr<MCStreamer> Streamer)
    41:         : AsmPrinter(TM, std::move(Streamer), ID) {}
    42: 
    43:     bool runOnMachineFunction(MachineFunction &Fn) override {
    44:       Subtarget = &Fn.getSubtarget<HexagonSubtarget>();
    45:       const bool Modified = AsmPrinter::runOnMachineFunction(Fn);
    46:       // Emit the XRay table for this function.
    47:       emitXRayTable();
    48: 
    49:       return Modified;
    50:     }
```
- EN: It declares types such as raw_ostream, TargetMachine, HexagonAsmPrinter, which carry the state or API of this component. It defines declarative TableGen records like raw_ostream, TargetMachine, HexagonAsmPrinter; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as emitAttributes, HexagonAsmPrinter, runOnMachineFunction, getSubtarget<HexagonSubtarget>, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 raw_ostream, TargetMachine, HexagonAsmPrinter 等类型，用来承载该组件的状态或接口。 这里定义了 raw_ostream, TargetMachine, HexagonAsmPrinter 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 emitAttributes, HexagonAsmPrinter, runOnMachineFunction, getSubtarget<HexagonSubtarget>, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 51-75 / 第 51-75 行

```cpp
    51: 
    52:     StringRef getPassName() const override {
    53:       return "Hexagon Assembly Printer";
    54:     }
    55: 
    56:     bool isBlockOnlyReachableByFallthrough(const MachineBasicBlock *MBB)
    57:           const override;
    58: 
    59:     void emitInstruction(const MachineInstr *MI) override;
    60: 
    61:     //===------------------------------------------------------------------===//
    62:     // XRay implementation
    63:     //===------------------------------------------------------------------===//
    64:     // XRay-specific lowering for Hexagon.
    65:     void LowerPATCHABLE_FUNCTION_ENTER(const MachineInstr &MI);
    66:     void LowerPATCHABLE_FUNCTION_EXIT(const MachineInstr &MI);
    67:     void LowerPATCHABLE_TAIL_CALL(const MachineInstr &MI);
    68:     void LowerPATCHABLE_EVENT_CALL(const MachineInstr &MI, bool Typed);
    69:     void EmitSled(const MachineInstr &MI, SledKind Kind);
    70: 
    71:     void HexagonProcessInstruction(MCInst &Inst, const MachineInstr &MBB);
    72: 
    73:     void printOperand(const MachineInstr *MI, unsigned OpNo, raw_ostream &O);
    74:     bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
    75:                          const char *ExtraCode, raw_ostream &OS) override;
```
- EN: It declares or implements routines such as getPassName, isBlockOnlyReachableByFallthrough, emitInstruction, LowerPATCHABLE_FUNCTION_ENTER, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonProcessInstruction, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getPassName, isBlockOnlyReachableByFallthrough, emitInstruction, LowerPATCHABLE_FUNCTION_ENTER, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonProcessInstruction，说明了它与同级后端组件的连接关系。

### Lines 76-84 / 第 76-84 行

```cpp
    76:     bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNo,
    77:                                const char *ExtraCode, raw_ostream &OS) override;
    78:     void emitStartOfAsmFile(Module &M) override;
    79:     void emitEndOfAsmFile(Module &M) override;
    80:   };
    81: 
    82: } // end namespace llvm
    83: 
    84: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONASMPRINTER_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as PrintAsmMemoryOperand, emitStartOfAsmFile, emitEndOfAsmFile, translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 PrintAsmMemoryOperand, emitStartOfAsmFile, emitEndOfAsmFile 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonSubtarget.h, llvm/CodeGen/AsmPrinter.h, llvm/CodeGen/MachineFunction.h, llvm/MC/MCStreamer.h, utility`
- Hexagon symbols / Hexagon 符号: `HexagonAsmPrinter, HexagonSubtarget, HexagonProcessInstruction`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
