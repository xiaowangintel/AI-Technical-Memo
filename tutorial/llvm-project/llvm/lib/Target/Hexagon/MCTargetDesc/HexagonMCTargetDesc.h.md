# HexagonMCTargetDesc.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCTargetDesc.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file provides Hexagon specific target descriptions.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===-- HexagonMCTargetDesc.h - Hexagon Target Descriptions -----*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file provides Hexagon specific target descriptions.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #ifndef LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONMCTARGETDESC_H
    14: #define LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONMCTARGETDESC_H
    15: 
    16: #include "llvm/MC/MCRegisterInfo.h"
    17: #include "llvm/Support/CommandLine.h"
    18: #include <cstdint>
    19: 
    20: #define Hexagon_POINTER_SIZE 4
    21: 
    22: #define Hexagon_PointerSize (Hexagon_POINTER_SIZE)
    23: #define Hexagon_PointerSize_Bits (Hexagon_POINTER_SIZE * 8)
    24: #define Hexagon_WordSize Hexagon_PointerSize
    25: #define Hexagon_WordSize_Bits Hexagon_PointerSize_Bits
```
- EN: It imports headers such as llvm/MC/MCRegisterInfo.h, llvm/Support/CommandLine.h, cstdint, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCTargetDesc, Hexagon_POINTER_SIZE, Hexagon_PointerSize, Hexagon_PointerSize_Bits, ... (6 total), showing how the code connects to sibling backend components.
- CN: 这里引入了 llvm/MC/MCRegisterInfo.h, llvm/Support/CommandLine.h, cstdint 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCTargetDesc, Hexagon_POINTER_SIZE, Hexagon_PointerSize, Hexagon_PointerSize_Bits, ... (6 total)，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```cpp
    26: 
    27: // allocframe saves LR and FP on stack before allocating
    28: // a new stack frame. This takes 8 bytes.
    29: #define HEXAGON_LRFP_SIZE 8
    30: 
    31: // Normal instruction size (in bytes).
    32: #define HEXAGON_INSTR_SIZE 4
    33: 
    34: // Maximum number of words and instructions in a packet.
    35: #define HEXAGON_PACKET_SIZE 4
    36: #define HEXAGON_MAX_PACKET_SIZE (HEXAGON_PACKET_SIZE * HEXAGON_INSTR_SIZE)
    37: // Minimum number of instructions in an end-loop packet.
    38: #define HEXAGON_PACKET_INNER_SIZE 2
    39: #define HEXAGON_PACKET_OUTER_SIZE 3
    40: // Maximum number of instructions in a packet before shuffling,
    41: // including a compound one or a duplex or an extender.
    42: #define HEXAGON_PRESHUFFLE_PACKET_SIZE (HEXAGON_PACKET_SIZE + 3)
    43: 
    44: // Name of the global offset table as defined by the Hexagon ABI
    45: #define HEXAGON_GOT_SYM_NAME "_GLOBAL_OFFSET_TABLE_"
    46: 
    47: namespace llvm {
    48: 
    49: struct InstrStage;
    50: class FeatureBitset;
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as InstrStage, FeatureBitset, which carry the state or API of this component. It defines declarative TableGen records like FeatureBitset; these records are consumed by TableGen instead of executed directly.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 InstrStage, FeatureBitset 等类型，用来承载该组件的状态或接口。 这里定义了 FeatureBitset 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 51-75 / 第 51-75 行

```cpp
    51: class MCAsmBackend;
    52: class MCCodeEmitter;
    53: class MCContext;
    54: class MCInstrInfo;
    55: class MCObjectTargetWriter;
    56: class MCRegisterInfo;
    57: class MCSubtargetInfo;
    58: class MCTargetOptions;
    59: class Target;
    60: class Triple;
    61: class StringRef;
    62: 
    63: extern cl::opt<bool> HexagonDisableCompound;
    64: extern cl::opt<bool> HexagonDisableDuplex;
    65: extern const InstrStage HexagonStages[];
    66: 
    67: MCInstrInfo *createHexagonMCInstrInfo();
    68: MCRegisterInfo *createHexagonMCRegisterInfo(StringRef TT);
    69: 
    70: namespace Hexagon_MC {
    71:   StringRef selectHexagonCPU(StringRef CPU);
    72: 
    73:   FeatureBitset completeHVXFeatures(const FeatureBitset &FB);
    74:   /// Create a Hexagon MCSubtargetInfo instance. This is exposed so Asm parser,
    75:   /// etc. do not need to go through TargetRegistry.
```
- EN: It opens namespaces (Hexagon_MC) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as MCAsmBackend, MCCodeEmitter, MCContext, MCInstrInfo, ... (11 total), which carry the state or API of this component. It defines declarative TableGen records like MCAsmBackend, MCCodeEmitter, MCContext, MCInstrInfo, MCObjectTargetWriter, ... (11 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as createHexagonMCInstrInfo, createHexagonMCRegisterInfo, selectHexagonCPU, completeHVXFeatures, translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（Hexagon_MC），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 MCAsmBackend, MCCodeEmitter, MCContext, MCInstrInfo, ... (11 total) 等类型，用来承载该组件的状态或接口。 这里定义了 MCAsmBackend, MCCodeEmitter, MCContext, MCInstrInfo, MCObjectTargetWriter, ... (11 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 createHexagonMCInstrInfo, createHexagonMCRegisterInfo, selectHexagonCPU, completeHVXFeatures 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 76-100 / 第 76-100 行

```cpp
    76:   MCSubtargetInfo *createHexagonMCSubtargetInfo(const Triple &TT, StringRef CPU,
    77:                                                 StringRef FS);
    78:   MCSubtargetInfo const *getArchSubtarget(MCSubtargetInfo const *STI);
    79:   void addArchSubtarget(MCSubtargetInfo const *STI,
    80:                         StringRef FS);
    81:   unsigned GetELFFlags(const MCSubtargetInfo &STI);
    82: 
    83:   llvm::ArrayRef<MCPhysReg> GetVectRegRev();
    84: 
    85:   std::optional<unsigned> getHVXVersion(const FeatureBitset &Features);
    86: 
    87:   unsigned getArchVersion(const FeatureBitset &Features);
    88:   } // namespace Hexagon_MC
    89: 
    90: MCCodeEmitter *createHexagonMCCodeEmitter(const MCInstrInfo &MCII,
    91:                                           MCContext &MCT);
    92: 
    93: MCAsmBackend *createHexagonAsmBackend(const Target &T,
    94:                                       const MCSubtargetInfo &STI,
    95:                                       const MCRegisterInfo &MRI,
    96:                                       const MCTargetOptions &Options);
    97: 
    98: std::unique_ptr<MCObjectTargetWriter>
    99: createHexagonELFObjectWriter(uint8_t OSABI, StringRef CPU);
   100: 
```
- EN: It opens namespaces (Hexagon_MC) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as createHexagonMCSubtargetInfo, getArchSubtarget, addArchSubtarget, GetELFFlags, ... (10 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include Hexagon_MC, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（Hexagon_MC），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 createHexagonMCSubtargetInfo, getArchSubtarget, addArchSubtarget, GetELFFlags, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 Hexagon_MC，说明了它与同级后端组件的连接关系。

### Lines 101-122 / 第 101-122 行

```cpp
   101: unsigned HexagonGetLastSlot();
   102: unsigned HexagonConvertUnits(unsigned ItinUnits, unsigned *Lanes);
   103: 
   104: } // End llvm namespace
   105: 
   106: // Define symbolic names for Hexagon registers.  This defines a mapping from
   107: // register name to register number.
   108: //
   109: #define GET_REGINFO_ENUM
   110: #include "HexagonGenRegisterInfo.inc"
   111: 
   112: // Defines symbolic names for the Hexagon instructions.
   113: //
   114: #define GET_INSTRINFO_ENUM
   115: #define GET_INSTRINFO_SCHED_ENUM
   116: #define GET_INSTRINFO_MC_HELPER_DECLS
   117: #include "HexagonGenInstrInfo.inc"
   118: 
   119: #define GET_SUBTARGETINFO_ENUM
   120: #include "HexagonGenSubtargetInfo.inc"
   121: 
   122: #endif // LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONMCTARGETDESC_H
```
- EN: It imports headers such as HexagonGenRegisterInfo.inc, HexagonGenInstrInfo.inc, HexagonGenSubtargetInfo.inc, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as HexagonGetLastSlot, HexagonConvertUnits, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 HexagonGenRegisterInfo.inc, HexagonGenInstrInfo.inc, HexagonGenSubtargetInfo.inc 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 HexagonGetLastSlot, HexagonConvertUnits 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/MC/MCRegisterInfo.h, llvm/Support/CommandLine.h, cstdint, HexagonGenRegisterInfo.inc, HexagonGenInstrInfo.inc, HexagonGenSubtargetInfo.inc`
- Hexagon symbols / Hexagon 符号: `HexagonMCTargetDesc, Hexagon_POINTER_SIZE, Hexagon_PointerSize, Hexagon_PointerSize_Bits, Hexagon_WordSize, Hexagon_WordSize_Bits, HexagonDisableCompound, HexagonDisableDuplex, HexagonStages, Hexagon_MC, ... (15 total)`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
