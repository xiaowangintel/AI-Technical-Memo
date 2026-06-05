# HexagonSelectionDAGInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonSelectionDAGInfo.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file implements the HexagonSelectionDAGInfo class.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===-- HexagonSelectionDAGInfo.cpp - Hexagon SelectionDAG Info -----------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file implements the HexagonSelectionDAGInfo class.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "HexagonSelectionDAGInfo.h"
    14: #include "HexagonTargetMachine.h"
    15: #include "llvm/CodeGen/SelectionDAG.h"
    16: 
    17: #define GET_SDNODE_DESC
    18: #include "HexagonGenSDNodeInfo.inc"
    19: 
    20: using namespace llvm;
    21: 
    22: #define DEBUG_TYPE "hexagon-selectiondag-info"
    23: 
    24: HexagonSelectionDAGInfo::HexagonSelectionDAGInfo()
    25:     : SelectionDAGGenTargetInfo(HexagonGenSDNodeInfo) {}
```
- EN: It imports headers such as HexagonSelectionDAGInfo.h, HexagonTargetMachine.h, llvm/CodeGen/SelectionDAG.h, HexagonGenSDNodeInfo.inc, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonSelectionDAGInfo::HexagonSelectionDAGInfo, translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonSelectionDAGInfo.h, HexagonTargetMachine.h, llvm/CodeGen/SelectionDAG.h, HexagonGenSDNodeInfo.inc 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonSelectionDAGInfo::HexagonSelectionDAGInfo 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 26-50 / 第 26-50 行

```cpp
    26: 
    27: const char *HexagonSelectionDAGInfo::getTargetNodeName(unsigned Opcode) const {
    28: #define CASE(NAME)                                                             \
    29:   case HexagonISD::NAME:                                                       \
    30:     return "HexagonISD::" #NAME
    31: 
    32:   // These nodes don't have corresponding entries in *.td files yet.
    33:   switch (static_cast<HexagonISD::NodeType>(Opcode)) {
    34:     CASE(CALLR);
    35:     CASE(VROR);
    36:     CASE(D2P);
    37:     CASE(P2D);
    38:     CASE(V2Q);
    39:     CASE(Q2V);
    40:     CASE(TL_EXTEND);
    41:     CASE(TL_TRUNCATE);
    42:     CASE(TYPECAST);
    43:     CASE(ISEL);
    44:   }
    45: #undef CASE
    46: 
    47:   return SelectionDAGGenTargetInfo::getTargetNodeName(Opcode);
    48: }
    49: 
    50: void HexagonSelectionDAGInfo::verifyTargetNode(const SelectionDAG &DAG,
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as HexagonSelectionDAGInfo::getTargetNodeName, CASE, SelectionDAGGenTargetInfo::getTargetNodeName, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSelectionDAGInfo, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 HexagonSelectionDAGInfo::getTargetNodeName, CASE, SelectionDAGGenTargetInfo::getTargetNodeName 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSelectionDAGInfo, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 51-75 / 第 51-75 行

```cpp
    51:                                                const SDNode *N) const {
    52:   switch (N->getOpcode()) {
    53:   default:
    54:     break;
    55:   case HexagonISD::VALIGNADDR:
    56:     // invalid number of operands; expected 1, got 2
    57:   case HexagonISD::VINSERTW0:
    58:     // operand #1 must have type i32, but has type v4i8/v2i16
    59:     return;
    60:   }
    61: 
    62:   SelectionDAGGenTargetInfo::verifyTargetNode(DAG, N);
    63: }
    64: 
    65: SDValue HexagonSelectionDAGInfo::EmitTargetCodeForMemcpy(
    66:     SelectionDAG &DAG, const SDLoc &dl, SDValue Chain, SDValue Dst, SDValue Src,
    67:     SDValue Size, Align Alignment, bool isVolatile, bool AlwaysInline,
    68:     MachinePointerInfo DstPtrInfo, MachinePointerInfo SrcPtrInfo) const {
    69:   ConstantSDNode *ConstantSize = dyn_cast<ConstantSDNode>(Size);
    70:   if (AlwaysInline || Alignment < Align(4) || !ConstantSize)
    71:     return SDValue();
    72: 
    73:   uint64_t SizeVal = ConstantSize->getZExtValue();
    74:   if (SizeVal < 32 || (SizeVal % 8) != 0)
    75:     return SDValue();
```
- EN: It declares or implements routines such as SelectionDAGGenTargetInfo::verifyTargetNode, HexagonSelectionDAGInfo::EmitTargetCodeForMemcpy, dyn_cast<ConstantSDNode>, getZExtValue, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonISD, HexagonSelectionDAGInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 SelectionDAGGenTargetInfo::verifyTargetNode, HexagonSelectionDAGInfo::EmitTargetCodeForMemcpy, dyn_cast<ConstantSDNode>, getZExtValue 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonISD, HexagonSelectionDAGInfo，说明了它与同级后端组件的连接关系。

### Lines 76-100 / 第 76-100 行

```cpp
    76: 
    77:   // Special case aligned memcpys with size >= 32 bytes and a multiple of 8.
    78:   //
    79:   const TargetLowering &TLI = *DAG.getSubtarget().getTargetLowering();
    80:   TargetLowering::ArgListTy Args;
    81:   Type *ArgTy = DAG.getDataLayout().getIntPtrType(*DAG.getContext());
    82:   Args.emplace_back(Dst, ArgTy);
    83:   Args.emplace_back(Src, ArgTy);
    84:   Args.emplace_back(Size, ArgTy);
    85: 
    86:   RTLIB::LibcallImpl SpecialMemcpyImpl = DAG.getLibcalls().getLibcallImpl(
    87:       RTLIB::HEXAGON_MEMCPY_LIKELY_ALIGNED_MIN32BYTES_MULT8BYTES);
    88:   if (SpecialMemcpyImpl == RTLIB::Unsupported)
    89:     return SDValue();
    90: 
    91:   const MachineFunction &MF = DAG.getMachineFunction();
    92:   bool LongCalls = MF.getSubtarget<HexagonSubtarget>().useLongCalls();
    93:   unsigned Flags = LongCalls ? HexagonII::HMOTF_ConstExtended : 0;
    94: 
    95:   CallingConv::ID CC =
    96:       DAG.getLibcalls().getLibcallImplCallingConv(SpecialMemcpyImpl);
    97: 
    98:   TargetLowering::CallLoweringInfo CLI(DAG);
    99:   CLI.setDebugLoc(dl)
   100:       .setChain(Chain)
```
- EN: It declares or implements routines such as getSubtarget, getDataLayout, emplace_back, getLibcalls, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSubtarget, getDataLayout, emplace_back, getLibcalls, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 101-110 / 第 101-110 行

```cpp
   101:       .setLibCallee(
   102:           CC, Type::getVoidTy(*DAG.getContext()),
   103:           DAG.getTargetExternalSymbol(
   104:               SpecialMemcpyImpl, TLI.getPointerTy(DAG.getDataLayout()), Flags),
   105:           std::move(Args))
   106:       .setDiscardResult();
   107: 
   108:   std::pair<SDValue, SDValue> CallResult = TLI.LowerCallTo(CLI);
   109:   return CallResult.second;
   110: }
```
- EN: It declares or implements routines such as setLibCallee, LowerCallTo, translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 setLibCallee, LowerCallTo 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- MC-layer target description / MC 层目标描述
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制
- calling convention encoding / 调用约定编码

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonSelectionDAGInfo.h, HexagonTargetMachine.h, llvm/CodeGen/SelectionDAG.h, HexagonGenSDNodeInfo.inc`
- Hexagon symbols / Hexagon 符号: `HexagonSelectionDAGInfo, HexagonTargetMachine, HexagonGenSDNodeInfo, HexagonISD, HexagonSubtarget, HexagonII`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
