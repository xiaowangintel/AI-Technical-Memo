# M68kCollapseMOVEMPass.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kCollapseMOVEMPass.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the M68k backend.
- **用途 (CN)**: 提供 M68k 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kCollapseMOVEMPass.cpp - Expand MOVEM pass -----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// `MOVEM` is an instruction that moves multiple registers a time according to
  11: /// the given mask. Thus sometimes it's pretty expensive.
  12: /// This file contains a pass that collapses sequential MOVEM instructions into
  13: /// a single one.
  14: ///
  15: //===----------------------------------------------------------------------===//
  16: 
  17: #include "M68k.h"
  18: #include "M68kFrameLowering.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68k.h`, `M68kFrameLowering.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68k.h`, `M68kFrameLowering.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "M68kInstrInfo.h"
  20: #include "M68kMachineFunction.h"
  21: #include "M68kSubtarget.h"
  22: 
  23: #include "llvm/CodeGen/MachineFunctionPass.h"
  24: #include "llvm/CodeGen/MachineInstrBuilder.h"
  25: #include "llvm/CodeGen/MachineRegisterInfo.h"
  26: #include "llvm/IR/EHPersonalities.h"
  27: #include "llvm/IR/GlobalValue.h"
  28: #include "llvm/Support/MathExtras.h"
  29: 
  30: using namespace llvm;
  31: 
  32: #define DEBUG_TYPE "m68k-collapse-movem"
  33: #define PASS_NAME "M68k MOVEM collapser pass"
  34: 
  35: namespace {
  36: 
```
- **EN**: It imports dependencies such as `M68kInstrInfo.h`, `M68kMachineFunction.h`, `M68kSubtarget.h`, `MachineFunctionPass.h`, `MachineInstrBuilder.h`, `MachineRegisterInfo.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `M68kInstrInfo.h`, `M68kMachineFunction.h`, `M68kSubtarget.h`, `MachineFunctionPass.h`, `MachineInstrBuilder.h`, `MachineRegisterInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37: enum UpdateType { Ascending, Descending, Intermixed };
  38: 
  39: /// An abtraction of the MOVEM chain currently processing
  40: class MOVEMState {
  41:   MachineBasicBlock::iterator Begin;
  42:   MachineBasicBlock::iterator End;
  43: 
  44:   unsigned Base;
  45: 
  46:   int Start;
  47:   int Stop;
  48: 
  49:   unsigned Mask;
  50: 
  51:   enum class AccessTy { None, Load, Store };
  52:   AccessTy Access;
  53: 
  54: public:
```
- **EN**: This block declares or refines TableGen records such as `MOVEMState`.
- **CN**: 该代码块声明或细化了 `MOVEMState` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   MOVEMState()
  56:       : Begin(nullptr), End(nullptr), Base(0), Start(INT_MIN), Stop(INT_MAX),
  57:         Mask(0), Access(AccessTy::None) {}
  58: 
  59:   void setBegin(MachineBasicBlock::iterator &MI) {
  60:     assert(Begin == nullptr);
  61:     Begin = MI;
  62:   }
  63: 
  64:   void setEnd(MachineBasicBlock::iterator &MI) {
  65:     assert(End == nullptr);
  66:     End = MI;
  67:   }
  68: 
  69:   bool hasBase() const { return Base != 0; }
  70: 
  71:   unsigned getBase() const {
  72:     assert(Base);
```
- **EN**: The range implements or declares functions including `MOVEMState`, `setEnd`, `hasBase`, `getBase`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `MOVEMState`, `setEnd`, `hasBase`, `getBase` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73:     return Base;
  74:   }
  75: 
  76:   MachineBasicBlock::iterator begin() {
  77:     assert(Begin != nullptr);
  78:     return Begin;
  79:   }
  80: 
  81:   MachineBasicBlock::iterator end() {
  82:     assert(End != nullptr);
  83:     return End;
  84:   }
  85: 
  86:   unsigned getMask() const { return Mask; }
  87: 
  88:   void setBase(int Value) {
  89:     assert(!hasBase());
  90:     Base = Value;
```
- **EN**: The range implements or declares functions including `begin`, `end`, `getMask`, `setBase`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `begin`, `end`, `getMask`, `setBase` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   }
  92: 
  93:   // You need to call this before Mask update
  94:   UpdateType classifyUpdateByMask(unsigned NewMask) const {
  95:     assert(NewMask && "Mask needs to select at least one register");
  96: 
  97:     if (NewMask > Mask) {
  98:       return Ascending;
  99:     } else if (NewMask < Mask) {
 100:       return Descending;
 101:     }
 102: 
 103:     return Intermixed;
 104:   }
 105: 
 106:   bool update(int O, int M) {
 107:     if (Mask & M)
 108:       // We've already seen this register and are planning on collapsing it
```
- **EN**: The range implements or declares functions including `classifyUpdateByMask`, `update`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `classifyUpdateByMask`, `update` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 109-126 / 第 109-126 行
```cpp
 109:       // into a MOVEM. This cannot be done twice for the same register in the
 110:       // same MOVEM, so bail out now.
 111:       return false;
 112:     UpdateType Type = classifyUpdateByMask(M);
 113:     if (Type == Intermixed)
 114:       return false;
 115:     if (Start == INT_MIN) {
 116:       Start = Stop = O;
 117:       updateMask(M);
 118:       return true;
 119:     } else if (Type == Descending && O == Start - 4) {
 120:       Start -= 4;
 121:       updateMask(M);
 122:       return true;
 123:     } else if (Type == Ascending && O == Stop + 4) {
 124:       Stop += 4;
 125:       updateMask(M);
 126:       return true;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:     }
 128: 
 129:     return false;
 130:   }
 131: 
 132:   int getFinalOffset() const {
 133:     assert(
 134:         Start != INT_MIN &&
 135:         "MOVEM in control mode should increment the address in each iteration");
 136:     return Start;
 137:   }
 138: 
 139:   bool updateMask(unsigned Value) {
 140:     assert(isUInt<16>(Value) && "Mask must fit 16 bit");
 141:     assert(!(Value & Mask) &&
 142:            "This is weird, there should be no intersections");
 143:     Mask |= Value;
 144:     return true;
```
- **EN**: The range implements or declares functions including `getFinalOffset`, `updateMask`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getFinalOffset`, `updateMask` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   }
 146: 
 147:   void setLoad() { Access = AccessTy::Load; }
 148:   void setStore() { Access = AccessTy::Store; }
 149: 
 150:   bool isLoad() const { return Access == AccessTy::Load; }
 151:   bool isStore() const { return Access == AccessTy::Store; }
 152: };
 153: 
 154: /// This Pass first walks through all the MOVEM instructions
 155: /// that are chained together and record each of the
 156: /// instruction's properties like register mask and data
 157: /// access type into a `MOVEState` instance.
 158: /// Then we perform reduction / collapsing on this `MOVEMState`
 159: /// representation before creating a new `MOVEM` instruction
 160: /// based on the collapsed result, as well as removing
 161: /// redundant `MOVEM` instructions.
 162: class M68kCollapseMOVEM : public MachineFunctionPass {
```
- **EN**: This block declares or refines TableGen records such as `M68kCollapseMOVEM`. The range implements or declares functions including `setLoad`, `setStore`, `isLoad`, `isStore`.
- **CN**: 该代码块声明或细化了 `M68kCollapseMOVEM` 等 TableGen 记录。 这一段实现或声明了 `setLoad`, `setStore`, `isLoad`, `isStore` 等函数。

### Lines 163-180 / 第 163-180 行
```cpp
 163: public:
 164:   static char ID;
 165: 
 166:   const M68kSubtarget *STI;
 167:   const M68kInstrInfo *TII;
 168:   const M68kRegisterInfo *TRI;
 169:   const M68kMachineFunctionInfo *MFI;
 170:   const M68kFrameLowering *FL;
 171: 
 172:   M68kCollapseMOVEM() : MachineFunctionPass(ID) {}
 173: 
 174:   void Finish(MachineBasicBlock &MBB, MOVEMState &State) {
 175:     auto MI = State.begin();
 176:     auto End = State.end();
 177:     DebugLoc DL = MI->getDebugLoc();
 178: 
 179:     // No need to delete then add a single instruction
 180:     if (std::next(MI) == End) {
```
- **EN**: The range implements or declares functions including `M68kCollapseMOVEM`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kCollapseMOVEM` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181:       State = MOVEMState();
 182:       return;
 183:     }
 184: 
 185:     // Delete all the MOVEM instruction till the end
 186:     while (MI != End) {
 187:       auto Next = std::next(MI);
 188:       MBB.erase(MI);
 189:       MI = Next;
 190:     }
 191: 
 192:     // Add a unified one
 193:     if (State.isLoad()) {
 194:       BuildMI(MBB, End, DL, TII->get(M68k::MOVM32mp))
 195:           .addImm(State.getMask())
 196:           .addImm(State.getFinalOffset())
 197:           .addReg(State.getBase());
 198:     } else {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 199-216 / 第 199-216 行
```cpp
 199:       BuildMI(MBB, End, DL, TII->get(M68k::MOVM32pm))
 200:           .addImm(State.getFinalOffset())
 201:           .addReg(State.getBase())
 202:           .addImm(State.getMask());
 203:     }
 204: 
 205:     State = MOVEMState();
 206:   }
 207: 
 208:   bool ProcessMI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 209:                  MOVEMState &State, unsigned Mask, int Offset, unsigned Reg,
 210:                  bool IsStore = false) {
 211:     if (State.hasBase()) {
 212:       // If current Type, Reg, Offset and Mask is in proper order  then
 213:       // merge in the state
 214:       MOVEMState Temp = State;
 215:       if (State.isStore() == IsStore && State.getBase() == Reg &&
 216:           State.update(Offset, Mask)) {
```
- **EN**: The range implements or declares functions including `BuildMI`, `ProcessMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `ProcessMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-234 / 第 217-234 行
```cpp
 217:         return true;
 218:         // Otherwise we Finish processing of the current MOVEM sequance and
 219:         // start a new one
 220:       } else {
 221:         State = Temp;
 222:         State.setEnd(MI);
 223:         Finish(MBB, State);
 224:         return ProcessMI(MBB, MI, State, Mask, Offset, Reg, IsStore);
 225:       }
 226:       // If this is the first instruction is sequance then initialize the State
 227:     } else if (Reg == TRI->getStackRegister() ||
 228:                Reg == TRI->getBaseRegister() ||
 229:                Reg == TRI->getFrameRegister(*MBB.getParent())) {
 230:       State.setBegin(MI);
 231:       State.setBase(Reg);
 232:       State.update(Offset, Mask);
 233:       IsStore ? State.setStore() : State.setLoad();
 234:       return true;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:     }
 236:     return false;
 237:   }
 238: 
 239:   bool runOnMachineFunction(MachineFunction &MF) override {
 240:     STI = &MF.getSubtarget<M68kSubtarget>();
 241:     TII = STI->getInstrInfo();
 242:     TRI = STI->getRegisterInfo();
 243:     MFI = MF.getInfo<M68kMachineFunctionInfo>();
 244:     FL = STI->getFrameLowering();
 245: 
 246:     bool Modified = false;
 247: 
 248:     MOVEMState State;
 249: 
 250:     unsigned Mask = 0;
 251:     unsigned Reg = 0;
 252:     int Offset = 0;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 253-270 / 第 253-270 行
```cpp
 253: 
 254:     for (auto &MBB : MF) {
 255:       auto MI = MBB.begin(), E = MBB.end();
 256:       while (MI != E) {
 257:         // Processing might change current instruction, save next first
 258:         auto NMI = std::next(MI);
 259:         switch (MI->getOpcode()) {
 260:         default:
 261:           if (State.hasBase()) {
 262:             State.setEnd(MI);
 263:             Finish(MBB, State);
 264:             Modified = true;
 265:           }
 266:           break;
 267:         case M68k::MOVM32jm:
 268:           Mask = MI->getOperand(1).getImm();
 269:           Reg = MI->getOperand(0).getReg();
 270:           Offset = 0;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 271-288 / 第 271-288 行
```cpp
 271:           Modified |= ProcessMI(MBB, MI, State, Mask, Offset, Reg, true);
 272:           break;
 273:         case M68k::MOVM32pm:
 274:           Mask = MI->getOperand(2).getImm();
 275:           Reg = MI->getOperand(1).getReg();
 276:           Offset = MI->getOperand(0).getImm();
 277:           Modified |= ProcessMI(MBB, MI, State, Mask, Offset, Reg, true);
 278:           break;
 279:         case M68k::MOVM32mj:
 280:           Mask = MI->getOperand(0).getImm();
 281:           Reg = MI->getOperand(1).getReg();
 282:           Offset = 0;
 283:           Modified |= ProcessMI(MBB, MI, State, Mask, Offset, Reg, false);
 284:           break;
 285:         case M68k::MOVM32mp:
 286:           Mask = MI->getOperand(0).getImm();
 287:           Reg = MI->getOperand(2).getReg();
 288:           Offset = MI->getOperand(1).getImm();
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 289-306 / 第 289-306 行
```cpp
 289:           Modified |= ProcessMI(MBB, MI, State, Mask, Offset, Reg, false);
 290:           break;
 291:         }
 292:         MI = NMI;
 293:       }
 294: 
 295:       if (State.hasBase()) {
 296:         State.setEnd(MI);
 297:         Finish(MBB, State);
 298:       }
 299:     }
 300: 
 301:     return Modified;
 302:   }
 303: };
 304: 
 305: char M68kCollapseMOVEM::ID = 0;
 306: } // anonymous namespace.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 307-313 / 第 307-313 行
```cpp
 307: 
 308: INITIALIZE_PASS(M68kCollapseMOVEM, DEBUG_TYPE, PASS_NAME, false, false)
 309: 
 310: /// Returns an instance of the pseudo instruction expansion pass.
 311: FunctionPass *llvm::createM68kCollapseMOVEMPass() {
 312:   return new M68kCollapseMOVEM();
 313: }
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Frame lowering**: Builds stack frames, callee-save handling, and prologue/epilogue sequences. / 构建栈帧、被调用者保存寄存器处理以及序言/尾声序列。

## Dependencies / 依赖关系
- `M68k.h`
- `M68kFrameLowering.h`
- `M68kInstrInfo.h`
- `M68kMachineFunction.h`
- `M68kSubtarget.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/IR/EHPersonalities.h`
- `llvm/IR/GlobalValue.h`
- `llvm/Support/MathExtras.h`
