# CSKYConstantPoolValue.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYConstantPoolValue.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file implements the CSKY specific constantpool value class.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYConstantPoolValue.cpp - CSKY constantpool value ---------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the CSKY specific constantpool value class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CSKYConstantPoolValue.h"
  14: #include "llvm/ADT/FoldingSet.h"
  15: #include "llvm/CodeGen/MachineBasicBlock.h"
  16: #include "llvm/IR/Constant.h"
  17: #include "llvm/IR/Constants.h"
  18: #include "llvm/IR/GlobalValue.h"
  19: #include "llvm/IR/Type.h"
  20: #include "llvm/Support/raw_ostream.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: using namespace llvm;
  22: 
  23: //===----------------------------------------------------------------------===//
  24: // CSKYConstantPoolValue
  25: //===----------------------------------------------------------------------===//
  26: 
  27: CSKYConstantPoolValue::CSKYConstantPoolValue(Type *Ty, CSKYCP::CSKYCPKind Kind,
  28:                                              unsigned PCAdjust,
  29:                                              CSKYCP::CSKYCPModifier Modifier,
  30:                                              bool AddCurrentAddress,
  31:                                              unsigned ID)
  32:     : MachineConstantPoolValue(Ty), Kind(Kind), PCAdjust(PCAdjust),
  33:       Modifier(Modifier), AddCurrentAddress(AddCurrentAddress), LabelId(ID) {}
  34: 
  35: const char *CSKYConstantPoolValue::getModifierText() const {
  36:   switch (Modifier) {
  37:   case CSKYCP::ADDR:
  38:     return "ADDR";
  39:   case CSKYCP::GOT:
  40:     return "GOT";
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as MachineConstantPoolValue contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 MachineConstantPoolValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 41-60

```cpp
  41:   case CSKYCP::GOTOFF:
  42:     return "GOTOFF";
  43:   case CSKYCP::PLT:
  44:     return "PLT";
  45:   case CSKYCP::TLSIE:
  46:     return "TLSIE";
  47:   case CSKYCP::TLSLE:
  48:     return "TLSLE";
  49:   case CSKYCP::TLSGD:
  50:     return "TLSGD";
  51:   case CSKYCP::NO_MOD:
  52:     return "";
  53:   }
  54:   llvm_unreachable("Unknown modifier!");
  55: }
  56: 
  57: int CSKYConstantPoolValue::getExistingMachineCPValue(MachineConstantPool *CP,
  58:                                                      Align Alignment) {
  59:   llvm_unreachable("Shouldn't be calling this directly!");
  60: }
```

- EN: Function bodies or method definitions such as getExistingMachineCPValue contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: getExistingMachineCPValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 61-80

```cpp
  61: 
  62: void CSKYConstantPoolValue::addSelectionDAGCSEId(FoldingSetNodeID &ID) {
  63:   ID.AddInteger(LabelId);
  64:   ID.AddInteger(PCAdjust);
  65:   ID.AddInteger(Modifier);
  66: }
  67: 
  68: void CSKYConstantPoolValue::print(raw_ostream &O) const {
  69:   if (Modifier)
  70:     O << "(" << getModifierText() << ")";
  71:   if (PCAdjust)
  72:     O << " + " << PCAdjust;
  73: }
  74: 
  75: //===----------------------------------------------------------------------===//
  76: // CSKYConstantPoolConstant
  77: //===----------------------------------------------------------------------===//
  78: 
  79: CSKYConstantPoolConstant::CSKYConstantPoolConstant(
  80:     const Constant *C, Type *Ty, CSKYCP::CSKYCPKind Kind, unsigned PCAdjust,
```

- EN: Function bodies or method definitions such as addSelectionDAGCSEId, print contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: addSelectionDAGCSEId, print 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81:     CSKYCP::CSKYCPModifier Modifier, bool AddCurrentAddress, unsigned ID)
  82:     : CSKYConstantPoolValue(Ty, Kind, PCAdjust, Modifier, AddCurrentAddress,
  83:                             ID),
  84:       CVal(C) {}
  85: 
  86: CSKYConstantPoolConstant *CSKYConstantPoolConstant::Create(
  87:     const Constant *C, CSKYCP::CSKYCPKind Kind, unsigned PCAdjust,
  88:     CSKYCP::CSKYCPModifier Modifier, bool AddCurrentAddress, unsigned ID) {
  89:   return new CSKYConstantPoolConstant(C, C->getType(), Kind, PCAdjust, Modifier,
  90:                                       AddCurrentAddress, ID);
  91: }
  92: 
  93: CSKYConstantPoolConstant *CSKYConstantPoolConstant::Create(
  94:     const Constant *C, Type *Ty, CSKYCP::CSKYCPKind Kind, unsigned PCAdjust,
  95:     CSKYCP::CSKYCPModifier Modifier, bool AddCurrentAddress, unsigned ID) {
  96:   return new CSKYConstantPoolConstant(C, Ty, Kind, PCAdjust, Modifier,
  97:                                       AddCurrentAddress, ID);
  98: }
  99: 
 100: const GlobalValue *CSKYConstantPoolConstant::getGV() const {
```

- EN: Function bodies or method definitions such as CSKYConstantPoolValue contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: CSKYConstantPoolValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 101-120

```cpp
 101:   assert(isa<GlobalValue>(CVal) && "CVal should be GlobalValue");
 102:   return cast<GlobalValue>(CVal);
 103: }
 104: 
 105: const BlockAddress *CSKYConstantPoolConstant::getBlockAddress() const {
 106:   assert(isa<BlockAddress>(CVal) && "CVal should be BlockAddress");
 107:   return cast<BlockAddress>(CVal);
 108: }
 109: 
 110: const Constant *CSKYConstantPoolConstant::getConstantPool() const {
 111:   return CVal;
 112: }
 113: 
 114: int CSKYConstantPoolConstant::getExistingMachineCPValue(MachineConstantPool *CP,
 115:                                                         Align Alignment) {
 116:   return getExistingMachineCPValueImpl<CSKYConstantPoolConstant>(CP, Alignment);
 117: }
 118: 
 119: void CSKYConstantPoolConstant::addSelectionDAGCSEId(FoldingSetNodeID &ID) {
 120:   ID.AddPointer(CVal);
```

- EN: Function bodies or method definitions such as getExistingMachineCPValue, addSelectionDAGCSEId contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: getExistingMachineCPValue, addSelectionDAGCSEId 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 121-140

```cpp
 121: 
 122:   CSKYConstantPoolValue::addSelectionDAGCSEId(ID);
 123: }
 124: 
 125: void CSKYConstantPoolConstant::print(raw_ostream &O) const {
 126:   O << CVal->getName();
 127:   CSKYConstantPoolValue::print(O);
 128: }
 129: 
 130: //===----------------------------------------------------------------------===//
 131: // CSKYConstantPoolSymbol
 132: //===----------------------------------------------------------------------===//
 133: 
 134: CSKYConstantPoolSymbol::CSKYConstantPoolSymbol(Type *Ty, const char *S,
 135:                                                unsigned PCAdjust,
 136:                                                CSKYCP::CSKYCPModifier Modifier,
 137:                                                bool AddCurrentAddress)
 138:     : CSKYConstantPoolValue(Ty, CSKYCP::CPExtSymbol, PCAdjust, Modifier,
 139:                             AddCurrentAddress),
 140:       S(strdup(S)) {}
```

- EN: Function bodies or method definitions such as print, CSKYConstantPoolValue contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: print, CSKYConstantPoolValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 141-160

```cpp
 141: 
 142: CSKYConstantPoolSymbol *
 143: CSKYConstantPoolSymbol::Create(Type *Ty, const char *S, unsigned PCAdjust,
 144:                                CSKYCP::CSKYCPModifier Modifier) {
 145:   return new CSKYConstantPoolSymbol(Ty, S, PCAdjust, Modifier, false);
 146: }
 147: 
 148: int CSKYConstantPoolSymbol::getExistingMachineCPValue(MachineConstantPool *CP,
 149:                                                       Align Alignment) {
 150: 
 151:   return getExistingMachineCPValueImpl<CSKYConstantPoolSymbol>(CP, Alignment);
 152: }
 153: 
 154: void CSKYConstantPoolSymbol::addSelectionDAGCSEId(FoldingSetNodeID &ID) {
 155:   ID.AddString(S);
 156:   CSKYConstantPoolValue::addSelectionDAGCSEId(ID);
 157: }
 158: 
 159: void CSKYConstantPoolSymbol::print(raw_ostream &O) const {
 160:   O << S;
```

- EN: Function bodies or method definitions such as Create, getExistingMachineCPValue, addSelectionDAGCSEId contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: Create, getExistingMachineCPValue, addSelectionDAGCSEId 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 161-180

```cpp
 161:   CSKYConstantPoolValue::print(O);
 162: }
 163: 
 164: //===----------------------------------------------------------------------===//
 165: // CSKYConstantPoolMBB
 166: //===----------------------------------------------------------------------===//
 167: 
 168: CSKYConstantPoolMBB::CSKYConstantPoolMBB(Type *Ty, const MachineBasicBlock *Mbb,
 169:                                          unsigned PCAdjust,
 170:                                          CSKYCP::CSKYCPModifier Modifier,
 171:                                          bool AddCurrentAddress)
 172:     : CSKYConstantPoolValue(Ty, CSKYCP::CPMachineBasicBlock, PCAdjust, Modifier,
 173:                             AddCurrentAddress),
 174:       MBB(Mbb) {}
 175: 
 176: CSKYConstantPoolMBB *CSKYConstantPoolMBB::Create(Type *Ty,
 177:                                                  const MachineBasicBlock *Mbb,
 178:                                                  unsigned PCAdjust) {
 179:   return new CSKYConstantPoolMBB(Ty, Mbb, PCAdjust, CSKYCP::ADDR, false);
 180: }
```

- EN: Function bodies or method definitions such as CSKYConstantPoolValue contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: CSKYConstantPoolValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 181-200

```cpp
 181: 
 182: int CSKYConstantPoolMBB::getExistingMachineCPValue(MachineConstantPool *CP,
 183:                                                    Align Alignment) {
 184:   return getExistingMachineCPValueImpl<CSKYConstantPoolMBB>(CP, Alignment);
 185: }
 186: 
 187: void CSKYConstantPoolMBB::addSelectionDAGCSEId(FoldingSetNodeID &ID) {
 188:   ID.AddPointer(MBB);
 189:   CSKYConstantPoolValue::addSelectionDAGCSEId(ID);
 190: }
 191: 
 192: void CSKYConstantPoolMBB::print(raw_ostream &O) const {
 193:   O << "BB#" << MBB->getNumber();
 194:   CSKYConstantPoolValue::print(O);
 195: }
 196: 
 197: //===----------------------------------------------------------------------===//
 198: // CSKYConstantPoolJT
 199: //===----------------------------------------------------------------------===//
 200: 
```

- EN: Function bodies or method definitions such as getExistingMachineCPValue, addSelectionDAGCSEId, print contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: getExistingMachineCPValue, addSelectionDAGCSEId, print 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 201-220

```cpp
 201: CSKYConstantPoolJT::CSKYConstantPoolJT(Type *Ty, int JTIndex, unsigned PCAdj,
 202:                                        CSKYCP::CSKYCPModifier Modifier,
 203:                                        bool AddCurrentAddress)
 204:     : CSKYConstantPoolValue(Ty, CSKYCP::CPJT, PCAdj, Modifier,
 205:                             AddCurrentAddress),
 206:       JTI(JTIndex) {}
 207: 
 208: CSKYConstantPoolJT *
 209: CSKYConstantPoolJT::Create(Type *Ty, int JTI, unsigned PCAdj,
 210:                            CSKYCP::CSKYCPModifier Modifier) {
 211:   return new CSKYConstantPoolJT(Ty, JTI, PCAdj, Modifier, false);
 212: }
 213: 
 214: int CSKYConstantPoolJT::getExistingMachineCPValue(MachineConstantPool *CP,
 215:                                                   Align Alignment) {
 216:   return getExistingMachineCPValueImpl<CSKYConstantPoolJT>(CP, Alignment);
 217: }
 218: 
 219: void CSKYConstantPoolJT::addSelectionDAGCSEId(FoldingSetNodeID &ID) {
 220:   ID.AddInteger(JTI);
```

- EN: Function bodies or method definitions such as CSKYConstantPoolValue, Create, getExistingMachineCPValue contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: CSKYConstantPoolValue, Create, getExistingMachineCPValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 221-227

```cpp
 221:   CSKYConstantPoolValue::addSelectionDAGCSEId(ID);
 222: }
 223: 
 224: void CSKYConstantPoolJT::print(raw_ostream &O) const {
 225:   O << "JTI#" << JTI;
 226:   CSKYConstantPoolValue::print(O);
 227: }
```

- EN: Function bodies or method definitions such as print contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: print 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

## Key Concepts / 关键概念

- SelectionDAG lowering / SelectionDAG 降级
- Basic block level transformation / 基本块级转换

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYConstantPoolValue.h`, `llvm/ADT/FoldingSet.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/Type.h`, `llvm/Support/raw_ostream.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Local companions / 本地配套文件: `CSKYConstantPoolValue.h`
