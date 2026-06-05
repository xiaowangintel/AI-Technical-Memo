# SystemZRegisterInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZRegisterInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines or implements target register information for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责定义或实现目标寄存器信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZRegisterInfo.h - SystemZ register information ----*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZREGISTERINFO_H
  10: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZREGISTERINFO_H
  11: 
  12: #include "SystemZ.h"
  13: #include "llvm/CodeGen/TargetFrameLowering.h"
  14: #include "llvm/CodeGen/TargetRegisterInfo.h"
  15: 
  16: #define GET_REGINFO_HEADER
  17: #include "SystemZGenRegisterInfo.inc"
  18: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZ.h`, `TargetFrameLowering.h`, `TargetRegisterInfo.h`, `SystemZGenRegisterInfo.inc` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZ.h`, `TargetFrameLowering.h`, `TargetRegisterInfo.h`, `SystemZGenRegisterInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: namespace llvm {
  20: 
  21: class LiveIntervals;
  22: 
  23: namespace SystemZ {
  24: // Return the subreg to use for referring to the even and odd registers
  25: // in a GR128 pair.  Is32Bit says whether we want a GR32 or GR64.
  26: inline unsigned even128(bool Is32bit) {
  27:   return Is32bit ? subreg_l32 : subreg_h64;
  28: }
  29: inline unsigned odd128(bool Is32bit) {
  30:   return Is32bit ? subreg_ll32 : subreg_l64;
  31: }
  32: 
  33: // Reg should be a 32-bit GPR.  Return true if it is a high register rather
  34: // than a low register.
  35: inline bool isHighReg(unsigned int Reg) {
  36:   if (SystemZ::GRH32BitRegClass.contains(Reg))
```
- **EN**: This block declares or refines TableGen records such as `LiveIntervals`. The range implements or declares functions including `even128`, `odd128`, `isHighReg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `LiveIntervals` 等 TableGen 记录。 这一段实现或声明了 `even128`, `odd128`, `isHighReg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 37-54 / 第 37-54 行
```cpp
  37:     return true;
  38:   assert(SystemZ::GR32BitRegClass.contains(Reg) && "Invalid GRX32");
  39:   return false;
  40: }
  41: } // end namespace SystemZ
  42: 
  43: /// A SystemZ-specific class detailing special use registers
  44: /// particular for calling conventions.
  45: /// It is abstract, all calling conventions must override and
  46: /// define the pure virtual member function defined in this class.
  47: class SystemZCallingConventionRegisters {
  48: 
  49: public:
  50:   /// \returns the register that keeps the return function address.
  51:   virtual int getReturnFunctionAddressRegister() = 0;
  52: 
  53:   /// \returns the register that keeps the
  54:   /// stack pointer address.
```
- **EN**: This block declares or refines TableGen records such as `SystemZCallingConventionRegisters`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 该代码块声明或细化了 `SystemZCallingConventionRegisters` 等 TableGen 记录。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   virtual int getStackPointerRegister() = 0;
  56: 
  57:   /// \returns the register that keeps the
  58:   /// frame pointer address.
  59:   virtual int getFramePointerRegister() = 0;
  60: 
  61:   /// \returns an array of all the callee saved registers.
  62:   virtual const MCPhysReg *
  63:   getCalleeSavedRegs(const MachineFunction *MF) const = 0;
  64: 
  65:   /// \returns the mask of all the call preserved registers.
  66:   virtual const uint32_t *getCallPreservedMask(const MachineFunction &MF,
  67:                                                CallingConv::ID CC) const = 0;
  68: 
  69:   /// \returns the offset to the locals area.
  70:   virtual int getCallFrameSize() = 0;
  71: 
  72:   /// \returns the stack pointer bias.
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target register information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   virtual int getStackPointerBias() = 0;
  74: 
  75:   /// Destroys the object. Bogus destructor allowing derived classes
  76:   /// to override it.
  77:   virtual ~SystemZCallingConventionRegisters() = default;
  78: };
  79: 
  80: /// XPLINK64 calling convention specific use registers
  81: /// Particular to z/OS when in 64 bit mode
  82: class SystemZXPLINK64Registers : public SystemZCallingConventionRegisters {
  83: public:
  84:   int getReturnFunctionAddressRegister() final { return SystemZ::R7D; };
  85: 
  86:   int getStackPointerRegister() final { return SystemZ::R4D; };
  87: 
  88:   int getFramePointerRegister() final { return SystemZ::R8D; };
  89: 
  90:   int getAddressOfCalleeRegister() { return SystemZ::R6D; };
```
- **EN**: This block declares or refines TableGen records such as `SystemZXPLINK64Registers`. The range implements or declares functions including `getAddressOfCalleeRegister`.
- **CN**: 该代码块声明或细化了 `SystemZXPLINK64Registers` 等 TableGen 记录。 这一段实现或声明了 `getAddressOfCalleeRegister` 等函数。

### Lines 91-108 / 第 91-108 行
```cpp
  91: 
  92:   int getADARegister() { return SystemZ::R5D; }
  93: 
  94:   const MCPhysReg *getCalleeSavedRegs(const MachineFunction *MF) const final;
  95: 
  96:   const uint32_t *getCallPreservedMask(const MachineFunction &MF,
  97:                                        CallingConv::ID CC) const final;
  98: 
  99:   int getCallFrameSize() final { return 128; }
 100: 
 101:   int getStackPointerBias() final { return 2048; }
 102: 
 103:   /// Destroys the object. Bogus destructor overriding base class destructor
 104:   ~SystemZXPLINK64Registers() override = default;
 105: };
 106: 
 107: /// ELF calling convention specific use registers
 108: /// Particular when on zLinux in 64 bit mode
```
- **EN**: The range implements or declares functions including `getADARegister`.
- **CN**: 这一段实现或声明了 `getADARegister` 等函数。

### Lines 109-126 / 第 109-126 行
```cpp
 109: class SystemZELFRegisters : public SystemZCallingConventionRegisters {
 110: public:
 111:   int getReturnFunctionAddressRegister() final { return SystemZ::R14D; };
 112: 
 113:   int getStackPointerRegister() final { return SystemZ::R15D; };
 114: 
 115:   int getFramePointerRegister() final { return SystemZ::R11D; };
 116: 
 117:   const MCPhysReg *getCalleeSavedRegs(const MachineFunction *MF) const final;
 118: 
 119:   const uint32_t *getCallPreservedMask(const MachineFunction &MF,
 120:                                        CallingConv::ID CC) const final;
 121: 
 122:   int getCallFrameSize() final { return SystemZMC::ELFCallFrameSize; }
 123: 
 124:   int getStackPointerBias() final { return 0; }
 125: 
 126:   /// Destroys the object. Bogus destructor overriding base class destructor
```
- **EN**: This block declares or refines TableGen records such as `SystemZELFRegisters`.
- **CN**: 该代码块声明或细化了 `SystemZELFRegisters` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   ~SystemZELFRegisters() override = default;
 128: };
 129: 
 130: struct SystemZRegisterInfo : public SystemZGenRegisterInfo {
 131: public:
 132:   SystemZRegisterInfo(unsigned int RA, unsigned int HwMode);
 133: 
 134:   /// getPointerRegClass - Return the register class to use to hold pointers.
 135:   /// This is currently only used by LOAD_STACK_GUARD, which requires a non-%r0
 136:   /// register, hence ADDR64.
 137:   const TargetRegisterClass *
 138:   getPointerRegClass(unsigned Kind = 0) const override {
 139:     return &SystemZ::ADDR64BitRegClass;
 140:   }
 141: 
 142:   /// getCrossCopyRegClass - Returns a legal register class to copy a register
 143:   /// in the specified class to or from. Returns NULL if it is possible to copy
 144:   /// between a two registers of the specified class.
```
- **EN**: It introduces interface types such as `SystemZRegisterInfo`, shaping how other backend components interact with this file. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `SystemZRegisterInfo` 等接口类型，定义了其他后端组件与本文件交互的方式。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   const TargetRegisterClass *
 146:   getCrossCopyRegClass(const TargetRegisterClass *RC) const override;
 147: 
 148:   bool getRegAllocationHints(Register VirtReg, ArrayRef<MCPhysReg> Order,
 149:                              SmallVectorImpl<MCPhysReg> &Hints,
 150:                              const MachineFunction &MF, const VirtRegMap *VRM,
 151:                              const LiveRegMatrix *Matrix) const override;
 152: 
 153:   // Override TargetRegisterInfo.h.
 154:   bool requiresRegisterScavenging(const MachineFunction &MF) const override {
 155:     return true;
 156:   }
 157:   bool requiresFrameIndexScavenging(const MachineFunction &MF) const override {
 158:     return true;
 159:   }
 160:   const MCPhysReg *getCalleeSavedRegs(const MachineFunction *MF) const override;
 161:   const uint32_t *getCallPreservedMask(const MachineFunction &MF,
 162:                                        CallingConv::ID CC) const override;
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target register information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   const uint32_t *getNoPreservedMask() const override;
 164:   BitVector getReservedRegs(const MachineFunction &MF) const override;
 165:   bool eliminateFrameIndex(MachineBasicBlock::iterator MI,
 166:                            int SPAdj, unsigned FIOperandNum,
 167:                            RegScavenger *RS) const override;
 168: 
 169:   /// SrcRC and DstRC will be morphed into NewRC if this returns true.
 170:  bool shouldCoalesce(MachineInstr *MI,
 171:                       const TargetRegisterClass *SrcRC,
 172:                       unsigned SubReg,
 173:                       const TargetRegisterClass *DstRC,
 174:                       unsigned DstSubReg,
 175:                       const TargetRegisterClass *NewRC,
 176:                       LiveIntervals &LIS) const override;
 177: 
 178:   Register getFrameRegister(const MachineFunction &MF) const override;
 179: };
 180: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-183 / 第 181-183 行
```cpp
 181: } // end namespace llvm
 182: 
 183: #endif
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target register information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Frame lowering**: Builds stack frames, callee-save handling, and prologue/epilogue sequences. / 构建栈帧、被调用者保存寄存器处理以及序言/尾声序列。

## Dependencies / 依赖关系
- `SystemZ.h`
- `llvm/CodeGen/TargetFrameLowering.h`
- `llvm/CodeGen/TargetRegisterInfo.h`
- `SystemZGenRegisterInfo.inc`
