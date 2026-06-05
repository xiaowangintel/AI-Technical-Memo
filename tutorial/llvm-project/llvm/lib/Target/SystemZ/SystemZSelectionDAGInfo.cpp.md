# SystemZSelectionDAGInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZSelectionDAGInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZSelectionDAGInfo.cpp - SystemZ SelectionDAG Info -----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the SystemZSelectionDAGInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "SystemZSelectionDAGInfo.h"
  14: #include "SystemZTargetMachine.h"
  15: #include "llvm/CodeGen/SelectionDAG.h"
  16: 
  17: #define GET_SDNODE_DESC
  18: #include "SystemZGenSDNodeInfo.inc"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZSelectionDAGInfo.h`, `SystemZTargetMachine.h`, `SelectionDAG.h`, `SystemZGenSDNodeInfo.inc` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZSelectionDAGInfo.h`, `SystemZTargetMachine.h`, `SelectionDAG.h`, `SystemZGenSDNodeInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: 
  20: using namespace llvm;
  21: 
  22: #define DEBUG_TYPE "systemz-selectiondag-info"
  23: 
  24: SystemZSelectionDAGInfo::SystemZSelectionDAGInfo()
  25:     : SelectionDAGGenTargetInfo(SystemZGenSDNodeInfo) {}
  26: 
  27: const char *SystemZSelectionDAGInfo::getTargetNodeName(unsigned Opcode) const {
  28:   switch (static_cast<SystemZISD::NodeType>(Opcode)) {
  29:   case SystemZISD::GET_CCMASK:
  30:     return "SystemZISD::GET_CCMASK";
  31:   }
  32: 
  33:   return SelectionDAGGenTargetInfo::getTargetNodeName(Opcode);
  34: }
  35: 
  36: static unsigned getMemMemLenAdj(unsigned Op) {
```
- **EN**: The range implements or declares functions including `SelectionDAGGenTargetInfo`, `getMemMemLenAdj`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 这一段实现或声明了 `SelectionDAGGenTargetInfo`, `getMemMemLenAdj` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   return Op == SystemZISD::MEMSET_MVC ? 2 : 1;
  38: }
  39: 
  40: static SDValue createMemMemNode(SelectionDAG &DAG, const SDLoc &DL, unsigned Op,
  41:                                 SDValue Chain, SDValue Dst, SDValue Src,
  42:                                 SDValue LenAdj, SDValue Byte) {
  43:   SDVTList VTs = Op == SystemZISD::CLC ? DAG.getVTList(MVT::i32, MVT::Other)
  44:                                        : DAG.getVTList(MVT::Other);
  45:   SmallVector<SDValue, 6> Ops;
  46:   if (Op == SystemZISD::MEMSET_MVC)
  47:     Ops = { Chain, Dst, LenAdj, Byte };
  48:   else
  49:     Ops = { Chain, Dst, Src, LenAdj };
  50:   return DAG.getNode(Op, DL, VTs, Ops);
  51: }
  52: 
  53: // Emit a mem-mem operation after subtracting one (or two for memset) from
  54: // size, which will be added back during pseudo expansion. As the Reg case
```
- **EN**: The range implements or declares functions including `createMemMemNode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `createMemMemNode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55: // emitted here may be converted by DAGCombiner into having an Imm length,
  56: // they are both emitted the same way.
  57: static SDValue emitMemMemImm(SelectionDAG &DAG, const SDLoc &DL, unsigned Op,
  58:                              SDValue Chain, SDValue Dst, SDValue Src,
  59:                              uint64_t Size, SDValue Byte = SDValue()) {
  60:   unsigned Adj = getMemMemLenAdj(Op);
  61:   assert(Size >= Adj && "Adjusted length overflow.");
  62:   SDValue LenAdj = DAG.getConstant(Size - Adj, DL, Dst.getValueType());
  63:   return createMemMemNode(DAG, DL, Op, Chain, Dst, Src, LenAdj, Byte);
  64: }
  65: 
  66: static SDValue emitMemMemReg(SelectionDAG &DAG, const SDLoc &DL, unsigned Op,
  67:                              SDValue Chain, SDValue Dst, SDValue Src,
  68:                              SDValue Size, SDValue Byte = SDValue()) {
  69:   int64_t Adj = getMemMemLenAdj(Op);
  70:   SDValue LenAdj = DAG.getNode(ISD::ADD, DL, MVT::i64,
  71:                                DAG.getZExtOrTrunc(Size, DL, MVT::i64),
  72:                                DAG.getSignedConstant(0 - Adj, DL, MVT::i64));
```
- **EN**: The range implements or declares functions including `emitMemMemImm`, `emitMemMemReg`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `emitMemMemImm`, `emitMemMemReg` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   return createMemMemNode(DAG, DL, Op, Chain, Dst, Src, LenAdj, Byte);
  74: }
  75: 
  76: SDValue SystemZSelectionDAGInfo::EmitTargetCodeForMemcpy(
  77:     SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Dst, SDValue Src,
  78:     SDValue Size, Align Alignment, bool IsVolatile, bool AlwaysInline,
  79:     MachinePointerInfo DstPtrInfo, MachinePointerInfo SrcPtrInfo) const {
  80:   if (IsVolatile)
  81:     return SDValue();
  82: 
  83:   if (auto *CSize = dyn_cast<ConstantSDNode>(Size))
  84:     return emitMemMemImm(DAG, DL, SystemZISD::MVC, Chain, Dst, Src,
  85:                          CSize->getZExtValue());
  86: 
  87:   return emitMemMemReg(DAG, DL, SystemZISD::MVC, Chain, Dst, Src, Size);
  88: }
  89: 
  90: // Handle a memset of 1, 2, 4 or 8 bytes with the operands given by
```
- **EN**: The range implements or declares functions including `SystemZSelectionDAGInfo::EmitTargetCodeForMemcpy`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZSelectionDAGInfo::EmitTargetCodeForMemcpy` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91: // Chain, Dst, ByteVal and Size.  These cases are expected to use
  92: // MVI, MVHHI, MVHI and MVGHI respectively.
  93: static SDValue memsetStore(SelectionDAG &DAG, const SDLoc &DL, SDValue Chain,
  94:                            SDValue Dst, uint64_t ByteVal, uint64_t Size,
  95:                            Align Alignment, MachinePointerInfo DstPtrInfo) {
  96:   uint64_t StoreVal = ByteVal;
  97:   for (unsigned I = 1; I < Size; ++I)
  98:     StoreVal |= ByteVal << (I * 8);
  99:   return DAG.getStore(
 100:       Chain, DL, DAG.getConstant(StoreVal, DL, MVT::getIntegerVT(Size * 8)),
 101:       Dst, DstPtrInfo, Alignment);
 102: }
 103: 
 104: SDValue SystemZSelectionDAGInfo::EmitTargetCodeForMemset(
 105:     SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Dst,
 106:     SDValue Byte, SDValue Size, Align Alignment, bool IsVolatile,
 107:     bool AlwaysInline, MachinePointerInfo DstPtrInfo) const {
 108:   EVT PtrVT = Dst.getValueType();
```
- **EN**: The range implements or declares functions including `memsetStore`, `SystemZSelectionDAGInfo::EmitTargetCodeForMemset`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `memsetStore`, `SystemZSelectionDAGInfo::EmitTargetCodeForMemset` 等函数。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 109-126 / 第 109-126 行
```cpp
 109: 
 110:   if (IsVolatile)
 111:     return SDValue();
 112: 
 113:   auto *CByte = dyn_cast<ConstantSDNode>(Byte);
 114:   if (auto *CSize = dyn_cast<ConstantSDNode>(Size)) {
 115:     uint64_t Bytes = CSize->getZExtValue();
 116:     if (Bytes == 0)
 117:       return SDValue();
 118:     if (CByte) {
 119:       // Handle cases that can be done using at most two of
 120:       // MVI, MVHI, MVHHI and MVGHI.  The latter two can only be
 121:       // used if ByteVal is all zeros or all ones; in other cases,
 122:       // we can move at most 2 halfwords.
 123:       uint64_t ByteVal = CByte->getZExtValue();
 124:       if (ByteVal == 0 || ByteVal == 255
 125:               ? Bytes <= 16 && llvm::popcount(Bytes) <= 2
 126:               : Bytes <= 4) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:         unsigned Size1 = Bytes == 16 ? 8 : llvm::bit_floor(Bytes);
 128:         unsigned Size2 = Bytes - Size1;
 129:         SDValue Chain1 = memsetStore(DAG, DL, Chain, Dst, ByteVal, Size1,
 130:                                      Alignment, DstPtrInfo);
 131:         if (Size2 == 0)
 132:           return Chain1;
 133:         Dst = DAG.getNode(ISD::ADD, DL, PtrVT, Dst,
 134:                           DAG.getConstant(Size1, DL, PtrVT));
 135:         DstPtrInfo = DstPtrInfo.getWithOffset(Size1);
 136:         SDValue Chain2 =
 137:             memsetStore(DAG, DL, Chain, Dst, ByteVal, Size2,
 138:                         std::min(Alignment, Align(Size1)), DstPtrInfo);
 139:         return DAG.getNode(ISD::TokenFactor, DL, MVT::Other, Chain1, Chain2);
 140:       }
 141:     } else {
 142:       // Handle one and two bytes using STC.
 143:       if (Bytes <= 2) {
 144:         SDValue Chain1 =
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:             DAG.getStore(Chain, DL, Byte, Dst, DstPtrInfo, Alignment);
 146:         if (Bytes == 1)
 147:           return Chain1;
 148:         SDValue Dst2 = DAG.getNode(ISD::ADD, DL, PtrVT, Dst,
 149:                                    DAG.getConstant(1, DL, PtrVT));
 150:         SDValue Chain2 = DAG.getStore(Chain, DL, Byte, Dst2,
 151:                                       DstPtrInfo.getWithOffset(1), Align(1));
 152:         return DAG.getNode(ISD::TokenFactor, DL, MVT::Other, Chain1, Chain2);
 153:       }
 154:     }
 155:     assert(Bytes >= 2 && "Should have dealt with 0- and 1-byte cases already");
 156: 
 157:     // Handle the special case of a memset of 0, which can use XC.
 158:     if (CByte && CByte->getZExtValue() == 0)
 159:       return emitMemMemImm(DAG, DL, SystemZISD::XC, Chain, Dst, Dst, Bytes);
 160: 
 161:     return emitMemMemImm(DAG, DL, SystemZISD::MEMSET_MVC, Chain, Dst, SDValue(),
 162:                          Bytes, DAG.getAnyExtOrTrunc(Byte, DL, MVT::i32));
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   }
 164: 
 165:   // Variable length
 166:   if (CByte && CByte->getZExtValue() == 0)
 167:     // Handle the special case of a variable length memset of 0 with XC.
 168:     return emitMemMemReg(DAG, DL, SystemZISD::XC, Chain, Dst, Dst, Size);
 169: 
 170:   return emitMemMemReg(DAG, DL, SystemZISD::MEMSET_MVC, Chain, Dst, SDValue(),
 171:                        Size, DAG.getAnyExtOrTrunc(Byte, DL, MVT::i32));
 172: }
 173: 
 174: // Convert the current CC value into an integer that is 0 if CC == 0,
 175: // greater than zero if CC == 1 and less than zero if CC >= 2.
 176: // The sequence starts with IPM, which puts CC into bits 29 and 28
 177: // of an integer and clears bits 30 and 31.
 178: static SDValue addIPMSequence(const SDLoc &DL, SDValue CCReg,
 179:                               SelectionDAG &DAG) {
 180:   SDValue IPM = DAG.getNode(SystemZISD::IPM, DL, MVT::i32, CCReg);
```
- **EN**: The range implements or declares functions including `addIPMSequence`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `addIPMSequence` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   SDValue SHL = DAG.getNode(ISD::SHL, DL, MVT::i32, IPM,
 182:                             DAG.getConstant(30 - SystemZ::IPM_CC, DL, MVT::i32));
 183:   SDValue SRA = DAG.getNode(ISD::SRA, DL, MVT::i32, SHL,
 184:                             DAG.getConstant(30, DL, MVT::i32));
 185:   return SRA;
 186: }
 187: 
 188: std::pair<SDValue, SDValue> SystemZSelectionDAGInfo::EmitTargetCodeForMemcmp(
 189:     SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Src1,
 190:     SDValue Src2, SDValue Size, const CallInst *CI) const {
 191:   SDValue CCReg;
 192:   // Swap operands to invert CC == 1 vs. CC == 2 cases.
 193:   if (auto *CSize = dyn_cast<ConstantSDNode>(Size)) {
 194:     uint64_t Bytes = CSize->getZExtValue();
 195:     assert(Bytes > 0 && "Caller should have handled 0-size case");
 196:     CCReg = emitMemMemImm(DAG, DL, SystemZISD::CLC, Chain, Src2, Src1, Bytes);
 197:   } else
 198:     CCReg = emitMemMemReg(DAG, DL, SystemZISD::CLC, Chain, Src2, Src1, Size);
```
- **EN**: The range implements or declares functions including `SystemZSelectionDAGInfo::EmitTargetCodeForMemcmp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZSelectionDAGInfo::EmitTargetCodeForMemcmp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 199-216 / 第 199-216 行
```cpp
 199:   Chain = CCReg.getValue(1);
 200:   return std::make_pair(addIPMSequence(DL, CCReg, DAG), Chain);
 201: }
 202: 
 203: std::pair<SDValue, SDValue> SystemZSelectionDAGInfo::EmitTargetCodeForMemchr(
 204:     SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Src,
 205:     SDValue Char, SDValue Length, MachinePointerInfo SrcPtrInfo) const {
 206:   // Use SRST to find the character.  End is its address on success.
 207:   EVT PtrVT = Src.getValueType();
 208:   SDVTList VTs = DAG.getVTList(PtrVT, MVT::i32, MVT::Other);
 209:   Length = DAG.getZExtOrTrunc(Length, DL, PtrVT);
 210:   Char = DAG.getZExtOrTrunc(Char, DL, MVT::i32);
 211:   Char = DAG.getNode(ISD::AND, DL, MVT::i32, Char,
 212:                      DAG.getConstant(255, DL, MVT::i32));
 213:   SDValue Limit = DAG.getNode(ISD::ADD, DL, PtrVT, Src, Length);
 214:   SDValue End = DAG.getNode(SystemZISD::SEARCH_STRING, DL, VTs, Chain,
 215:                             Limit, Src, Char);
 216:   SDValue CCReg = End.getValue(1);
```
- **EN**: The range implements or declares functions including `SystemZSelectionDAGInfo::EmitTargetCodeForMemchr`.
- **CN**: 这一段实现或声明了 `SystemZSelectionDAGInfo::EmitTargetCodeForMemchr` 等函数。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   Chain = End.getValue(2);
 218: 
 219:   // Now select between End and null, depending on whether the character
 220:   // was found.
 221:   SDValue Ops[] = {
 222:       End, DAG.getConstant(0, DL, PtrVT),
 223:       DAG.getTargetConstant(SystemZ::CCMASK_SRST, DL, MVT::i32),
 224:       DAG.getTargetConstant(SystemZ::CCMASK_SRST_FOUND, DL, MVT::i32), CCReg};
 225:   End = DAG.getNode(SystemZISD::SELECT_CCMASK, DL, PtrVT, Ops);
 226:   return std::make_pair(End, Chain);
 227: }
 228: 
 229: std::pair<SDValue, SDValue> SystemZSelectionDAGInfo::EmitTargetCodeForStrcpy(
 230:     SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Dest,
 231:     SDValue Src, MachinePointerInfo DestPtrInfo, MachinePointerInfo SrcPtrInfo,
 232:     bool isStpcpy, const CallInst *CI) const {
 233:   SDVTList VTs = DAG.getVTList(Dest.getValueType(), MVT::Other);
 234:   SDValue EndDest = DAG.getNode(SystemZISD::STPCPY, DL, VTs, Chain, Dest, Src,
```
- **EN**: The range implements or declares functions including `SystemZSelectionDAGInfo::EmitTargetCodeForStrcpy`.
- **CN**: 这一段实现或声明了 `SystemZSelectionDAGInfo::EmitTargetCodeForStrcpy` 等函数。

### Lines 235-252 / 第 235-252 行
```cpp
 235:                                 DAG.getConstant(0, DL, MVT::i32));
 236:   return std::make_pair(isStpcpy ? EndDest : Dest, EndDest.getValue(1));
 237: }
 238: 
 239: std::pair<SDValue, SDValue> SystemZSelectionDAGInfo::EmitTargetCodeForStrcmp(
 240:     SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Src1,
 241:     SDValue Src2, MachinePointerInfo Op1PtrInfo, MachinePointerInfo Op2PtrInfo,
 242:     const CallInst *CI) const {
 243:   SDVTList VTs = DAG.getVTList(Src1.getValueType(), MVT::i32, MVT::Other);
 244:   // Swap operands to invert CC == 1 vs. CC == 2 cases.
 245:   SDValue Unused = DAG.getNode(SystemZISD::STRCMP, DL, VTs, Chain, Src2, Src1,
 246:                                DAG.getConstant(0, DL, MVT::i32));
 247:   SDValue CCReg = Unused.getValue(1);
 248:   Chain = Unused.getValue(2);
 249:   return std::make_pair(addIPMSequence(DL, CCReg, DAG), Chain);
 250: }
 251: 
 252: // Search from Src for a null character, stopping once Src reaches Limit.
```
- **EN**: The range implements or declares functions including `SystemZSelectionDAGInfo::EmitTargetCodeForStrcmp`.
- **CN**: 这一段实现或声明了 `SystemZSelectionDAGInfo::EmitTargetCodeForStrcmp` 等函数。

### Lines 253-270 / 第 253-270 行
```cpp
 253: // Return a pair of values, the first being the number of nonnull characters
 254: // and the second being the out chain.
 255: //
 256: // This can be used for strlen by setting Limit to 0.
 257: static std::pair<SDValue, SDValue> getBoundedStrlen(SelectionDAG &DAG,
 258:                                                     const SDLoc &DL,
 259:                                                     SDValue Chain, SDValue Src,
 260:                                                     SDValue Limit) {
 261:   EVT PtrVT = Src.getValueType();
 262:   SDVTList VTs = DAG.getVTList(PtrVT, MVT::i32, MVT::Other);
 263:   SDValue End = DAG.getNode(SystemZISD::SEARCH_STRING, DL, VTs, Chain,
 264:                             Limit, Src, DAG.getConstant(0, DL, MVT::i32));
 265:   Chain = End.getValue(2);
 266:   SDValue Len = DAG.getNode(ISD::SUB, DL, PtrVT, End, Src);
 267:   return std::make_pair(Len, Chain);
 268: }
 269: 
 270: std::pair<SDValue, SDValue> SystemZSelectionDAGInfo::EmitTargetCodeForStrlen(
```
- **EN**: The range implements or declares functions including `getBoundedStrlen`.
- **CN**: 这一段实现或声明了 `getBoundedStrlen` 等函数。

### Lines 271-284 / 第 271-284 行
```cpp
 271:     SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Src,
 272:     const CallInst *CI) const {
 273:   EVT PtrVT = Src.getValueType();
 274:   return getBoundedStrlen(DAG, DL, Chain, Src, DAG.getConstant(0, DL, PtrVT));
 275: }
 276: 
 277: std::pair<SDValue, SDValue> SystemZSelectionDAGInfo::EmitTargetCodeForStrnlen(
 278:     SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Src,
 279:     SDValue MaxLength, MachinePointerInfo SrcPtrInfo) const {
 280:   EVT PtrVT = Src.getValueType();
 281:   MaxLength = DAG.getZExtOrTrunc(MaxLength, DL, PtrVT);
 282:   SDValue Limit = DAG.getNode(ISD::ADD, DL, PtrVT, Src, MaxLength);
 283:   return getBoundedStrlen(DAG, DL, Chain, Src, Limit);
 284: }
```
- **EN**: The range implements or declares functions including `SystemZSelectionDAGInfo::EmitTargetCodeForStrnlen`.
- **CN**: 这一段实现或声明了 `SystemZSelectionDAGInfo::EmitTargetCodeForStrnlen` 等函数。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `SystemZSelectionDAGInfo.h`
- `SystemZTargetMachine.h`
- `llvm/CodeGen/SelectionDAG.h`
- `SystemZGenSDNodeInfo.inc`
