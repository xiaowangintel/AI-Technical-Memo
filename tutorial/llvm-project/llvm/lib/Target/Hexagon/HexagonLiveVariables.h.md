# HexagonLiveVariables.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonLiveVariables.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file implements the Hexagon specific LiveVariables analysis pass.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===----------------- HexagonLiveVariables.h ---------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // Hexagon Live Variable Analysis
     9: // This file implements the Hexagon specific LiveVariables analysis pass.
    10: // 1. Computes the live variables by analyzing the use-defs.
    11: //      - The use-def specifiers are 'assumed' to be correct for each operand.
    12: // 2. Re-calculates the MBB numbers to that they are in sequence.
    13: // TODO: Mark dead instructions.
    14: // TODO: Provide APIs like the target independent Liveness Analysis so that
    15: //       other passes can reuse the liveness information.
    16: //===----------------------------------------------------------------------===//
    17: 
    18: #ifndef HEXAGON_LIVEVARIABLES_H
    19: #define HEXAGON_LIVEVARIABLES_H
    20: 
    21: #include "Hexagon.h"
    22: #include "HexagonInstrInfo.h"
    23: 
    24: #include "llvm/ADT/BitVector.h"
    25: #include "llvm/ADT/DenseMap.h"
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It imports headers such as Hexagon.h, HexagonInstrInfo.h, llvm/ADT/BitVector.h, llvm/ADT/DenseMap.h, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里引入了 Hexagon.h, HexagonInstrInfo.h, llvm/ADT/BitVector.h, llvm/ADT/DenseMap.h 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 26-50 / 第 26-50 行

```cpp
    26: #include "llvm/ADT/SmallPtrSet.h"
    27: #include "llvm/ADT/SmallVector.h"
    28: #include "llvm/ADT/StringRef.h"
    29: #include "llvm/CodeGen/MachineBasicBlock.h"
    30: #include "llvm/CodeGen/MachineFunctionPass.h"
    31: #include "llvm/CodeGen/MachineInstr.h"
    32: #include "llvm/CodeGen/TargetRegisterInfo.h"
    33: #include <algorithm>
    34: #include <cstdint>
    35: #include <list>
    36: 
    37: class HexagonLiveVariablesImpl;
    38: 
    39: namespace llvm {
    40: 
    41: typedef std::pair<BitVector, BitVector> UseDef_t; // (Use, Def)
    42: typedef DenseMap<MachineBasicBlock *, UseDef_t> MBBUseDef_t;
    43: typedef DenseMap<const MachineInstr *, UseDef_t> MIUseDef_t;
    44: 
    45: // List of intervals [From, To).
    46: typedef std::list<std::pair<int64_t, int64_t>> IntervalList_t;
    47: // Intervals stored in indexed form.
    48: typedef SmallVector<IntervalList_t, 0> IndexedLiveIntervals_t;
    49: 
    50: class HexagonLiveVariables : public MachineFunctionPass {
```
- EN: It imports headers such as llvm/ADT/SmallPtrSet.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, ... (10 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonLiveVariablesImpl, HexagonLiveVariables, which carry the state or API of this component. It defines declarative TableGen records like HexagonLiveVariablesImpl, HexagonLiveVariables; these records are consumed by TableGen instead of executed directly.
- CN: 这里引入了 llvm/ADT/SmallPtrSet.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, ... (10 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonLiveVariablesImpl, HexagonLiveVariables 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonLiveVariablesImpl, HexagonLiveVariables 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 51-75 / 第 51-75 行

```cpp
    51: public:
    52:   typedef MachineBasicBlock::const_instr_iterator MICInstIterType;
    53: 
    54:   static char ID; // Pass identification, replacement for typeid
    55:   bool HLVComplete;
    56:   HexagonLiveVariables();
    57: 
    58:   bool runOnMachineFunction(MachineFunction &MF) override;
    59:   void getAnalysisUsage(AnalysisUsage &AU) const override;
    60:   StringRef getPassName() const override {
    61:     return "Hexagon Live Variables Analysis";
    62:   }
    63: 
    64:   /// recalculate - recalculates the liveness from scratch. It is like
    65:   /// calling the runOnMachineFunction.
    66:   void recalculate(MachineFunction &MF);
    67: 
    68:   /// updateLocalLiveness - update only kill flags of operands.
    69:   /// Assumes that global liveness is correct.
    70:   bool updateLocalLiveness(MachineFunction &Fn);
    71: 
    72:   /// updateLocalLiveness - update only kill flags of operands in MBB.
    73:   /// Assumes that global liveness is correct.
    74:   /// This is useful when a local transformation modifies MIs,
    75:   /// which only changes the local liveness.
```
- EN: It declares or implements routines such as HexagonLiveVariables, runOnMachineFunction, getAnalysisUsage, getPassName, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonLiveVariables, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonLiveVariables, runOnMachineFunction, getAnalysisUsage, getPassName, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariables，说明了它与同级后端组件的连接关系。

### Lines 76-100 / 第 76-100 行

```cpp
    76:   bool updateLocalLiveness(MachineBasicBlock *MBB, bool updateBundle);
    77: 
    78:   /// incrementalUpdate - update the liveness when \p MIDelta is moved from
    79:   /// \p From to \p To.
    80:   /// @note: This is extremely fragile now. It 'assumes' that the other
    81:   /// successor(s) of \p To do not use Defs of MIDelta.
    82:   bool incrementalUpdate(MICInstIterType MIDelta, MachineBasicBlock *From,
    83:                          MachineBasicBlock *To);
    84:   // addNewMI - update internal data-structures of Live Variable Analysis.
    85:   void addNewMI(MachineInstr *MI, MachineBasicBlock *MBB);
    86: 
    87:   /// addNewMBB - inform the LiveVariable Analysis that new MBB has been added.
    88:   /// update the liveness of this new MBB.
    89:   /// @note MBB should be empty. If we want to add an MI, add it after calling
    90:   /// this function.
    91:   void addNewMBB(MachineBasicBlock *MBB);
    92: 
    93:   /// @brief Constructs use-defs of \p MBB by analyzing each MachineOperand.
    94:   /// Collects relevant information so that global liveness can be updated.
    95:   void constructUseDef(MachineBasicBlock *MBB);
    96: 
    97:   bool isLiveOut(const MachineBasicBlock *MBB, unsigned Reg) const;
    98:   const BitVector &getLiveOuts(const MachineBasicBlock *MBB) const;
    99: 
   100:   // Returns true when \p Reg is used within [MIBegin, MIEnd)
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as updateLocalLiveness, successor, addNewMI, addNewMBB, ... (7 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 updateLocalLiveness, successor, addNewMI, addNewMBB, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-125 / 第 101-125 行

```cpp
   101:   // @note: MIBegin and MIEnd should be from same MBB
   102:   // @note: It returns just the first use found in the range.
   103:   // The Use is closest to MIEnd.
   104:   // Takes care of aliases as well.
   105:   bool
   106:   isUsedWithin(MICInstIterType MIBegin, MICInstIterType MIEnd, unsigned Reg,
   107:                MICInstIterType &Use,
   108:                SmallPtrSet<MachineInstr *, 2> *ExceptionsList = nullptr) const;
   109:   // Returns true when \p Reg id defined within [MIBegin, MIEnd)
   110:   // @note: MIBegin and MIEnd should be from same MBB
   111:   // The Def is closest to MIEnd.
   112:   // Takes care of aliases as well.
   113:   bool isDefinedWithin(MICInstIterType MIBegin, MICInstIterType MIEnd,
   114:                        unsigned Reg, MICInstIterType &Def) const;
   115:   bool isDefLiveIn(const MachineInstr *MI, const MachineBasicBlock *MBB) const;
   116:   MBBUseDef_t &getMBBUseDefs();
   117:   MIUseDef_t &getMIUseDefs();
   118: 
   119:   /// Returns the linear distance (as per layout) of \p MI from the Function.
   120:   /// \p BufferPerMBB is to allow some room for .falign (if added later).
   121:   unsigned getDistanceBetween(const MachineBasicBlock *From,
   122:                               const MachineBasicBlock *To,
   123:                               unsigned BufferPerMBB = HEXAGON_INSTR_SIZE) const;
   124: 
   125:   // recalculate the distance map.
```
- EN: It declares or implements routines such as isUsedWithin, isDefinedWithin, isDefLiveIn, getMBBUseDefs, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 isUsedWithin, isDefinedWithin, isDefLiveIn, getMBBUseDefs, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 126-134 / 第 126-134 行

```cpp
   126:   void regenerateDistanceMap(const MachineFunction &Fn);
   127: 
   128: private:
   129:   std::unique_ptr<HexagonLiveVariablesImpl> HLV;
   130: };
   131: 
   132: } // namespace llvm
   133: 
   134: #endif
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as regenerateDistanceMap, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonLiveVariablesImpl, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 regenerateDistanceMap 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariablesImpl，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, llvm/ADT/BitVector.h, llvm/ADT/DenseMap.h, llvm/ADT/SmallPtrSet.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, ... (14 total)`
- Hexagon symbols / Hexagon 符号: `HexagonLiveVariables, HexagonInstrInfo, HexagonLiveVariablesImpl`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
