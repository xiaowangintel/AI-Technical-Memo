# HexagonMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonMachineFunctionInfo.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Declares a Hexagon backend component.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //=- HexagonMachineFunctionInfo.h - Hexagon machine function info -*- C++ -*-=//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONMACHINEFUNCTIONINFO_H
    10: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONMACHINEFUNCTIONINFO_H
    11: 
    12: #include "llvm/CodeGen/MIRYamlMapping.h"
    13: #include "llvm/CodeGen/MachineFunction.h"
    14: #include <map>
    15: 
    16: namespace llvm {
    17: 
    18: namespace yaml {
    19: struct HexagonFunctionInfo;
    20: } // end namespace yaml
    21: 
    22: namespace Hexagon {
    23: 
    24:     const unsigned int StartPacket = 0x1;
    25:     const unsigned int EndPacket = 0x2;
```
- EN: It imports headers such as llvm/CodeGen/MIRYamlMapping.h, llvm/CodeGen/MachineFunction.h, map, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, yaml, Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonFunctionInfo, which carry the state or API of this component.
- CN: 这里引入了 llvm/CodeGen/MIRYamlMapping.h, llvm/CodeGen/MachineFunction.h, map 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, yaml, Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonFunctionInfo 等类型，用来承载该组件的状态或接口。

### Lines 26-50 / 第 26-50 行

```cpp
    26: 
    27: } // end namespace Hexagon
    28: 
    29: /// Hexagon target-specific information for each MachineFunction.
    30: class HexagonMachineFunctionInfo : public MachineFunctionInfo {
    31:   // SRetReturnReg - Some subtargets require that sret lowering includes
    32:   // returning the value of the returned struct in a register. This field
    33:   // holds the virtual register into which the sret argument is passed.
    34:   unsigned SRetReturnReg = 0;
    35:   Register StackAlignBaseReg = 0;    // Aligned-stack base register
    36:   int VarArgsFrameIndex;
    37:   int RegSavedAreaStartFrameIndex;
    38:   int FirstNamedArgFrameIndex;
    39:   int LastNamedArgFrameIndex;
    40:   bool HasClobberLR = false;
    41:   bool HasEHReturn = false;
    42:   std::map<const MachineInstr*, unsigned> PacketInfo;
    43:   virtual void anchor();
    44: 
    45: public:
    46:   HexagonMachineFunctionInfo() = default;
    47: 
    48:   HexagonMachineFunctionInfo(const Function &F,
    49:                              const TargetSubtargetInfo *STI) {}
    50: 
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonMachineFunctionInfo, which carry the state or API of this component. It defines declarative TableGen records like HexagonMachineFunctionInfo; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as anchor, HexagonMachineFunctionInfo, translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonMachineFunctionInfo 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonMachineFunctionInfo 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 anchor, HexagonMachineFunctionInfo 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-75 / 第 51-75 行

```cpp
    51:   MachineFunctionInfo *
    52:   clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
    53:         const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
    54:       const override;
    55: 
    56:   void initializeBaseYamlFields(const yaml::HexagonFunctionInfo &YamlMFI);
    57: 
    58:   unsigned getSRetReturnReg() const { return SRetReturnReg; }
    59:   void setSRetReturnReg(unsigned Reg) { SRetReturnReg = Reg; }
    60: 
    61:   void setVarArgsFrameIndex(int v) { VarArgsFrameIndex = v; }
    62:   int getVarArgsFrameIndex() { return VarArgsFrameIndex; }
    63: 
    64:   void setRegSavedAreaStartFrameIndex(int v) { RegSavedAreaStartFrameIndex = v;}
    65:   int getRegSavedAreaStartFrameIndex() { return RegSavedAreaStartFrameIndex; }
    66: 
    67:   void setFirstNamedArgFrameIndex(int v) { FirstNamedArgFrameIndex = v; }
    68:   int getFirstNamedArgFrameIndex() { return FirstNamedArgFrameIndex; }
    69: 
    70:   void setLastNamedArgFrameIndex(int v) { LastNamedArgFrameIndex = v; }
    71:   int getLastNamedArgFrameIndex() { return LastNamedArgFrameIndex; }
    72: 
    73:   void setStartPacket(MachineInstr* MI) {
    74:     PacketInfo[MI] |= Hexagon::StartPacket;
    75:   }
```
- EN: It declares or implements routines such as clone, initializeBaseYamlFields, getSRetReturnReg, setSRetReturnReg, ... (13 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonFunctionInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 clone, initializeBaseYamlFields, getSRetReturnReg, setSRetReturnReg, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonFunctionInfo，说明了它与同级后端组件的连接关系。

### Lines 76-100 / 第 76-100 行

```cpp
    76:   void setEndPacket(MachineInstr* MI)   {
    77:     PacketInfo[MI] |= Hexagon::EndPacket;
    78:   }
    79:   bool isStartPacket(const MachineInstr* MI) const {
    80:     auto It = PacketInfo.find(MI);
    81:     return It != PacketInfo.end() && (It->second & Hexagon::StartPacket);
    82:   }
    83:   bool isEndPacket(const MachineInstr* MI) const {
    84:     auto It = PacketInfo.find(MI);
    85:     return It != PacketInfo.end() && (It->second & Hexagon::EndPacket);
    86:   }
    87:   void setHasClobberLR(bool v) { HasClobberLR = v;  }
    88:   bool hasClobberLR() const { return HasClobberLR; }
    89: 
    90:   bool hasEHReturn() const { return HasEHReturn; };
    91:   void setHasEHReturn(bool H = true) { HasEHReturn = H; };
    92: 
    93:   void setStackAlignBaseReg(Register R) { StackAlignBaseReg = R; }
    94:   Register getStackAlignBaseReg() const { return StackAlignBaseReg; }
    95: };
    96: 
    97: namespace yaml {
    98: 
    99: /// Hexagon-specific MachineFunction properties for YAML serialization.
   100: struct HexagonFunctionInfo final : public yaml::MachineFunctionInfo {
```
- EN: It opens namespaces (yaml) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonFunctionInfo, which carry the state or API of this component. It declares or implements routines such as setEndPacket, isStartPacket, find, end, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（yaml），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonFunctionInfo 等类型，用来承载该组件的状态或接口。 这里声明或实现了 setEndPacket, isStartPacket, find, end, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-121 / 第 101-121 行

```cpp
   101:   StringValue StackAlignBaseReg;
   102: 
   103:   HexagonFunctionInfo() = default;
   104:   HexagonFunctionInfo(const llvm::HexagonMachineFunctionInfo &MFI,
   105:                       const TargetRegisterInfo &TRI);
   106: 
   107:   void mappingImpl(yaml::IO &YamlIO) override;
   108:   ~HexagonFunctionInfo() override = default;
   109: };
   110: 
   111: template <> struct MappingTraits<HexagonFunctionInfo> {
   112:   static void mapping(IO &YamlIO, HexagonFunctionInfo &MFI) {
   113:     YamlIO.mapOptional("stackAlignBaseReg", MFI.StackAlignBaseReg);
   114:   }
   115: };
   116: 
   117: } // end namespace yaml
   118: 
   119: } // end namespace llvm
   120: 
   121: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONMACHINEFUNCTIONINFO_H
```
- EN: It opens namespaces (yaml, llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonFunctionInfo, mappingImpl, mapping, mapOptional, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonFunctionInfo, HexagonMachineFunctionInfo, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（yaml, llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonFunctionInfo, mappingImpl, mapping, mapOptional 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonFunctionInfo, HexagonMachineFunctionInfo，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- VLIW packetization / VLIW 成包
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制
- ABI and stack-frame lowering / ABI 与栈帧下沉

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/CodeGen/MIRYamlMapping.h, llvm/CodeGen/MachineFunction.h, map`
- Hexagon symbols / Hexagon 符号: `HexagonMachineFunctionInfo, HexagonFunctionInfo`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
