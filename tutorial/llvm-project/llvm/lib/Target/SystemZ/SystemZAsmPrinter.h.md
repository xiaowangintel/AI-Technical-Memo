# SystemZAsmPrinter.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZAsmPrinter.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将 LLVM 机器指令输出为文本汇编。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZAsmPrinter.h - SystemZ LLVM assembly printer ----*- C++ -*--===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZASMPRINTER_H
  10: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZASMPRINTER_H
  11: 
  12: #include "MCTargetDesc/SystemZTargetStreamer.h"
  13: #include "SystemZMCInstLower.h"
  14: #include "SystemZTargetMachine.h"
  15: #include "llvm/CodeGen/AsmPrinter.h"
  16: #include "llvm/CodeGen/MachineFunction.h"
  17: #include "llvm/CodeGen/StackMaps.h"
  18: #include "llvm/MC/MCInstBuilder.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZTargetStreamer.h`, `SystemZMCInstLower.h`, `SystemZTargetMachine.h`, `AsmPrinter.h`, `MachineFunction.h`, `StackMaps.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZTargetStreamer.h`, `SystemZMCInstLower.h`, `SystemZTargetMachine.h`, `AsmPrinter.h`, `MachineFunction.h`, `StackMaps.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/Support/Compiler.h"
  20: 
  21: namespace llvm {
  22: class MCStreamer;
  23: class MachineInstr;
  24: class Module;
  25: class raw_ostream;
  26: 
  27: class LLVM_LIBRARY_VISIBILITY SystemZAsmPrinter : public AsmPrinter {
  28: public:
  29:   static char ID;
  30: 
  31: private:
  32:   MCSymbol *PPA2Sym;
  33: 
  34:   SystemZTargetStreamer *getTargetStreamer() {
  35:     MCTargetStreamer *TS = OutStreamer->getTargetStreamer();
  36:     assert(TS && "do not have a target streamer");
```
- **EN**: It imports dependencies such as `Compiler.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `MCStreamer`, `MachineInstr`, `Module`, `raw_ostream`, `LLVM_LIBRARY_VISIBILITY`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 它引入了 `Compiler.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `MCStreamer`, `MachineInstr`, `Module`, `raw_ostream`, `LLVM_LIBRARY_VISIBILITY` 等 TableGen 记录。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 37-54 / 第 37-54 行
```cpp
  37:     return static_cast<SystemZTargetStreamer *>(TS);
  38:   }
  39: 
  40:   /// Call type information for XPLINK.
  41:   enum class CallType {
  42:     BASR76 = 0,   // b'x000' == BASR  r7,r6
  43:     BRAS7 = 1,    // b'x001' == BRAS  r7,ep
  44:     RESVD_2 = 2,  // b'x010'
  45:     BRASL7 = 3,   // b'x011' == BRASL r7,ep
  46:     RESVD_4 = 4,  // b'x100'
  47:     RESVD_5 = 5,  // b'x101'
  48:     BALR1415 = 6, // b'x110' == BALR  r14,r15
  49:     BASR33 = 7,   // b'x111' == BASR  r3,r3
  50:   };
  51: 
  52:   // The Associated Data Area (ADA) contains descriptors which help locating
  53:   // external symbols. For each symbol and type, the displacement into the ADA
  54:   // is stored.
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   class AssociatedDataAreaTable {
  56:   public:
  57:     using DisplacementTable =
  58:         MapVector<std::pair<const MCSymbol *, unsigned>, uint32_t>;
  59: 
  60:   private:
  61:     const uint64_t PointerSize;
  62: 
  63:     /// The mapping of name/slot type pairs to displacements.
  64:     DisplacementTable Displacements;
  65: 
  66:     /// The next available displacement value. Incremented when new entries into
  67:     /// the ADA are created.
  68:     uint32_t NextDisplacement = 0;
  69: 
  70:   public:
  71:     AssociatedDataAreaTable(uint64_t PointerSize) : PointerSize(PointerSize) {}
  72: 
```
- **EN**: This block declares or refines TableGen records such as `AssociatedDataAreaTable`. The range implements or declares functions including `AssociatedDataAreaTable`.
- **CN**: 该代码块声明或细化了 `AssociatedDataAreaTable` 等 TableGen 记录。 这一段实现或声明了 `AssociatedDataAreaTable` 等函数。

### Lines 73-90 / 第 73-90 行
```cpp
  73:     /// @brief Add a function descriptor to the ADA.
  74:     /// @param MI Pointer to an ADA_ENTRY instruction.
  75:     /// @return The displacement of the descriptor into the ADA.
  76:     uint32_t insert(const MachineOperand MO);
  77: 
  78:     /// @brief Get the displacement into associated data area (ADA) for a name.
  79:     /// If no  displacement is already associated with the name, assign one and
  80:     /// return it.
  81:     /// @param Sym The symbol for which the displacement should be returned.
  82:     /// @param SlotKind The ADA type.
  83:     /// @return The displacement of the descriptor into the ADA.
  84:     uint32_t insert(const MCSymbol *Sym, unsigned SlotKind);
  85: 
  86:     /// Get the table of GOFF displacements.  This is 'const' since it should
  87:     /// never be modified by anything except the APIs on this class.
  88:     const DisplacementTable &getTable() const { return Displacements; }
  89: 
  90:     uint32_t getNextDisplacement() const { return NextDisplacement; }
```
- **EN**: The range implements or declares functions including `getNextDisplacement`.
- **CN**: 这一段实现或声明了 `getNextDisplacement` 等函数。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   };
  92: 
  93:   AssociatedDataAreaTable ADATable;
  94: 
  95:   // Record a list of GlobalAlias associated with a GlobalObject.
  96:   // This is used for z/OS's extra-label-at-definition aliasing strategy.
  97:   // This is similar to what is done for AIX.
  98:   DenseMap<const GlobalObject *, SmallVector<const GlobalAlias *, 1>>
  99:       GOAliasMap;
 100: 
 101:   struct PPA1Info {
 102:     StringRef Name;
 103:     MCSymbol *FnEnd = nullptr;    // Symbol marking function end.
 104:     MCSymbol *PPA1 = nullptr;     // Symbol marking PPA1 begin.
 105:     MCSymbol *EPMarker = nullptr; // Symbol marking entry point.
 106:     MCSymbol *PersonalityRoutine = nullptr;
 107:     MCSymbol *GCCEH = nullptr;
 108:     int64_t OffsetFPR = 0;
```
- **EN**: It introduces interface types such as `PPA1Info`, shaping how other backend components interact with this file.
- **CN**: 它引入了 `PPA1Info` 等接口类型，定义了其他后端组件与本文件交互的方式。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     int64_t OffsetVR = 0;
 110:     uint64_t CallFrameSize = 0;
 111:     unsigned SizeOfFnParams = 0;
 112:     uint32_t FrameAndFPROffset;
 113:     uint32_t FrameAndVROffset;
 114:     uint16_t SavedGPRMask = 0;
 115:     uint16_t SavedFPRMask = 0;
 116:     uint8_t SavedVRMask = 0;
 117:     uint8_t FrameReg = 0;
 118:     uint8_t AllocaReg = 0;
 119:     bool IsVarArg = false;
 120:     bool HasStackProtector = false;
 121:   };
 122:   SmallVector<PPA1Info, 0> DeferredPPA1;
 123: 
 124:   void calculatePPA1();
 125:   void emitPPA1(PPA1Info &Info);
 126:   void emitPPA2(Module &M);
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   void emitADASection();
 128:   void emitIDRLSection(Module &M);
 129: 
 130: public:
 131:   SystemZAsmPrinter(TargetMachine &TM, std::unique_ptr<MCStreamer> Streamer)
 132:       : AsmPrinter(TM, std::move(Streamer), ID), PPA2Sym(nullptr),
 133:         ADATable(TM.getPointerSize(0)) {}
 134: 
 135:   // Override AsmPrinter.
 136:   StringRef getPassName() const override { return "SystemZ Assembly Printer"; }
 137:   void emitInstruction(const MachineInstr *MI) override;
 138:   void emitMachineConstantPoolValue(MachineConstantPoolValue *MCPV) override;
 139:   void emitXXStructorList(const DataLayout &DL, const Constant *List,
 140:                           bool IsCtor) override;
 141:   void emitEndOfAsmFile(Module &M) override;
 142:   bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
 143:                        const char *ExtraCode, raw_ostream &OS) override;
 144:   bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNo,
```
- **EN**: The range implements or declares functions including `SystemZAsmPrinter`.
- **CN**: 这一段实现或声明了 `SystemZAsmPrinter` 等函数。

### Lines 145-162 / 第 145-162 行
```cpp
 145:                              const char *ExtraCode, raw_ostream &OS) override;
 146: 
 147:   bool runOnMachineFunction(MachineFunction &MF) override {
 148:     AsmPrinter::runOnMachineFunction(MF);
 149: 
 150:     // Emit the XRay table for this function.
 151:     emitXRayTable();
 152: 
 153:     return false;
 154:   }
 155: 
 156:   bool doInitialization(Module &M) override;
 157:   void emitFunctionEntryLabel() override;
 158:   void emitFunctionBodyEnd() override;
 159:   void emitStartOfAsmFile(Module &M) override;
 160:   void emitGlobalAlias(const Module &M, const GlobalAlias &GA) override;
 161:   const MCExpr *lowerConstant(const Constant *CV,
 162:                               const Constant *BaseCV = nullptr,
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-177 / 第 163-177 行
```cpp
 163:                               uint64_t Offset = 0) override;
 164: 
 165: private:
 166:   void emitCallInformation(CallType CT);
 167:   void LowerFENTRY_CALL(const MachineInstr &MI, SystemZMCInstLower &MCIL);
 168:   void LowerSTACKMAP(const MachineInstr &MI);
 169:   void LowerPATCHPOINT(const MachineInstr &MI, SystemZMCInstLower &Lower);
 170:   void LowerPATCHABLE_FUNCTION_ENTER(const MachineInstr &MI,
 171:                                      SystemZMCInstLower &Lower);
 172:   void LowerPATCHABLE_RET(const MachineInstr &MI, SystemZMCInstLower &Lower);
 173:   void emitAttributes(Module &M);
 174: };
 175: } // end namespace llvm
 176: 
 177: #endif
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `MCTargetDesc/SystemZTargetStreamer.h`
- `SystemZMCInstLower.h`
- `SystemZTargetMachine.h`
- `llvm/CodeGen/AsmPrinter.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/StackMaps.h`
- `llvm/MC/MCInstBuilder.h`
- `llvm/Support/Compiler.h`
