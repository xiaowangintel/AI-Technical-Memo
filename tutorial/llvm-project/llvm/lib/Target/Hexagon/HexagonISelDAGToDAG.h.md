# HexagonISelDAGToDAG.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonISelDAGToDAG.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Declares Hexagon DAG-to-DAG instruction selection for Hexagon.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。 重点涉及指令语义与选择。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===-- HexagonISelDAGToDAG.h -----------------------------------*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // Hexagon specific code to select Hexagon machine instructions for
     9: // SelectionDAG operations.
    10: //===----------------------------------------------------------------------===//
    11: 
    12: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONISELDAGTODAG_H
    13: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONISELDAGTODAG_H
    14: 
    15: #include "HexagonSubtarget.h"
    16: #include "HexagonTargetMachine.h"
    17: #include "llvm/CodeGen/SelectionDAG.h"
    18: #include "llvm/CodeGen/SelectionDAGISel.h"
    19: #include "llvm/Support/CodeGen.h"
    20: 
    21: #include <vector>
    22: 
    23: namespace llvm {
    24: class MachineFunction;
    25: class HexagonInstrInfo;
```
- EN: It imports headers such as HexagonSubtarget.h, HexagonTargetMachine.h, llvm/CodeGen/SelectionDAG.h, llvm/CodeGen/SelectionDAGISel.h, ... (6 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as MachineFunction, HexagonInstrInfo, which carry the state or API of this component.
- CN: 这里引入了 HexagonSubtarget.h, HexagonTargetMachine.h, llvm/CodeGen/SelectionDAG.h, llvm/CodeGen/SelectionDAGISel.h, ... (6 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 MachineFunction, HexagonInstrInfo 等类型，用来承载该组件的状态或接口。

### Lines 26-50 / 第 26-50 行

```cpp
    26: class HexagonRegisterInfo;
    27: 
    28: class HexagonDAGToDAGISel : public SelectionDAGISel {
    29:   const HexagonSubtarget *HST;
    30:   const HexagonInstrInfo *HII;
    31:   const HexagonRegisterInfo *HRI;
    32: 
    33: public:
    34:   HexagonDAGToDAGISel() = delete;
    35: 
    36:   explicit HexagonDAGToDAGISel(HexagonTargetMachine &tm,
    37:                                CodeGenOptLevel OptLevel)
    38:       : SelectionDAGISel(tm, OptLevel), HST(nullptr), HII(nullptr),
    39:         HRI(nullptr) {}
    40: 
    41:   bool runOnMachineFunction(MachineFunction &MF) override {
    42:     // Reset the subtarget each time through.
    43:     HST = &MF.getSubtarget<HexagonSubtarget>();
    44:     HII = HST->getInstrInfo();
    45:     HRI = HST->getRegisterInfo();
    46:     SelectionDAGISel::runOnMachineFunction(MF);
    47:     updateAligna();
    48:     return true;
    49:   }
    50: 
```
- EN: It declares types such as HexagonRegisterInfo, HexagonDAGToDAGISel, which carry the state or API of this component. It defines declarative TableGen records like HexagonRegisterInfo, HexagonDAGToDAGISel; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as HexagonDAGToDAGISel, runOnMachineFunction, getSubtarget<HexagonSubtarget>, getInstrInfo, ... (7 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonRegisterInfo, HexagonDAGToDAGISel, HexagonSubtarget, HexagonInstrInfo, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonRegisterInfo, HexagonDAGToDAGISel 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonRegisterInfo, HexagonDAGToDAGISel 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 HexagonDAGToDAGISel, runOnMachineFunction, getSubtarget<HexagonSubtarget>, getInstrInfo, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonRegisterInfo, HexagonDAGToDAGISel, HexagonSubtarget, HexagonInstrInfo, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 51-75 / 第 51-75 行

```cpp
    51:   bool ComplexPatternFuncMutatesDAG() const override {
    52:     return true;
    53:   }
    54:   void PreprocessISelDAG() override;
    55:   void emitFunctionEntryCode() override;
    56: 
    57:   void Select(SDNode *N) override;
    58: 
    59:   // Complex Pattern Selectors.
    60:   inline bool SelectAddrGA(SDValue &N, SDValue &R);
    61:   inline bool SelectAddrGP(SDValue &N, SDValue &R);
    62:   inline bool SelectAnyImm(SDValue &N, SDValue &R);
    63:   inline bool SelectAnyInt(SDValue &N, SDValue &R);
    64:   bool SelectAnyImmediate(SDValue &N, SDValue &R, Align Alignment);
    65:   bool SelectGlobalAddress(SDValue &N, SDValue &R, bool UseGP, Align Alignment);
    66:   bool SelectAddrFI(SDValue &N, SDValue &R);
    67:   bool DetectUseSxtw(SDValue &N, SDValue &R);
    68: 
    69:   inline bool SelectAnyImm0(SDValue &N, SDValue &R);
    70:   inline bool SelectAnyImm1(SDValue &N, SDValue &R);
    71:   inline bool SelectAnyImm2(SDValue &N, SDValue &R);
    72:   inline bool SelectAnyImm3(SDValue &N, SDValue &R);
    73: 
    74:   // Generate a machine instruction node corresponding to the circ/brev
    75:   // load intrinsic.
```
- EN: It declares or implements routines such as ComplexPatternFuncMutatesDAG, PreprocessISelDAG, emitFunctionEntryCode, Select, ... (16 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 ComplexPatternFuncMutatesDAG, PreprocessISelDAG, emitFunctionEntryCode, Select, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 76-100 / 第 76-100 行

```cpp
    76:   MachineSDNode *LoadInstrForLoadIntrinsic(SDNode *IntN);
    77:   // Given the circ/brev load intrinsic and the already generated machine
    78:   // instruction, generate the appropriate store (that is a part of the
    79:   // intrinsic's functionality).
    80:   SDNode *StoreInstrForLoadIntrinsic(MachineSDNode *LoadN, SDNode *IntN);
    81: 
    82:   void SelectFrameIndex(SDNode *N);
    83:   /// SelectInlineAsmMemoryOperand - Implement addressing mode selection for
    84:   /// inline asm expressions.
    85:   bool SelectInlineAsmMemoryOperand(const SDValue &Op,
    86:                                     InlineAsm::ConstraintCode ConstraintID,
    87:                                     std::vector<SDValue> &OutOps) override;
    88:   bool tryLoadOfLoadIntrinsic(LoadSDNode *N);
    89:   bool SelectBrevLdIntrinsic(SDNode *IntN);
    90:   bool SelectNewCircIntrinsic(SDNode *IntN);
    91:   void SelectLoad(SDNode *N);
    92:   void SelectIndexedLoad(LoadSDNode *LD, const SDLoc &dl);
    93:   void SelectIndexedStore(StoreSDNode *ST, const SDLoc &dl);
    94:   void SelectStore(SDNode *N);
    95:   void SelectSHL(SDNode *N);
    96:   void SelectIntrinsicWChain(SDNode *N);
    97:   void SelectIntrinsicWOChain(SDNode *N);
    98:   void SelectExtractSubvector(SDNode *N);
    99:   void SelectConstant(SDNode *N);
   100:   void SelectConstantFP(SDNode *N);
```
- EN: It declares or implements routines such as LoadInstrForLoadIntrinsic, store, SelectFrameIndex, SelectInlineAsmMemoryOperand, ... (17 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 LoadInstrForLoadIntrinsic, store, SelectFrameIndex, SelectInlineAsmMemoryOperand, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-125 / 第 101-125 行

```cpp
   101:   void SelectV65Gather(SDNode *N);
   102:   void SelectV65GatherPred(SDNode *N);
   103:   void SelectHVXDualOutput(SDNode *N);
   104:   void SelectAddSubCarry(SDNode *N);
   105:   void SelectVAlign(SDNode *N);
   106:   void SelectVAlignAddr(SDNode *N);
   107:   void SelectTypecast(SDNode *N);
   108:   void SelectP2D(SDNode *N);
   109:   void SelectD2P(SDNode *N);
   110:   void SelectQ2V(SDNode *N);
   111:   void SelectV2Q(SDNode *N);
   112:   void SelectFDiv(SDNode *N);
   113:   void FDiv(SDNode *N);
   114:   void FastFDiv(SDNode *N);
   115: 
   116:   // Include the declarations autogenerated from the selection patterns.
   117:   #define GET_DAGISEL_DECL
   118:   #include "HexagonGenDAGISel.inc"
   119: 
   120: private:
   121:   // This is really only to get access to ReplaceNode (which is a protected
   122:   // member). Any other members used by HvxSelector can be moved around to
   123:   // make them accessible).
   124:   friend struct HvxSelector;
   125: 
```
- EN: It imports headers such as HexagonGenDAGISel.inc, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as SelectV65Gather, SelectV65GatherPred, SelectHVXDualOutput, SelectAddSubCarry, ... (14 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonGenDAGISel, showing how the code connects to sibling backend components.
- CN: 这里引入了 HexagonGenDAGISel.inc 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 SelectV65Gather, SelectV65GatherPred, SelectHVXDualOutput, SelectAddSubCarry, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonGenDAGISel，说明了它与同级后端组件的连接关系。

### Lines 126-150 / 第 126-150 行

```cpp
   126:   SDValue selectUndef(const SDLoc &dl, MVT ResTy) {
   127:     SDNode *U = CurDAG->getMachineNode(TargetOpcode::IMPLICIT_DEF, dl, ResTy);
   128:     return SDValue(U, 0);
   129:   }
   130: 
   131:   bool keepsLowBits(const SDValue &Val, unsigned NumBits, SDValue &Src);
   132:   bool isAlignedMemNode(const MemSDNode *N) const;
   133:   bool isSmallStackStore(const StoreSDNode *N) const;
   134:   bool isPositiveHalfWord(const SDNode *N) const;
   135:   bool hasOneUse(const SDNode *N) const;
   136: 
   137:   // DAG preprocessing functions.
   138:   void PreprocessHvxISelDAG();
   139:   void ppSimplifyOrSelect0(std::vector<SDNode*> &&Nodes);
   140:   void ppAddrReorderAddShl(std::vector<SDNode*> &&Nodes);
   141:   void ppAddrRewriteAndSrl(std::vector<SDNode*> &&Nodes);
   142:   void ppHoistZextI1(std::vector<SDNode*> &&Nodes);
   143:   void ppHvxShuffleOfShuffle(std::vector<SDNode*> &&Nodes);
   144: 
   145:   void SelectHvxExtractSubvector(SDNode *N);
   146:   void SelectHvxShuffle(SDNode *N);
   147:   void SelectHvxRor(SDNode *N);
   148:   void SelectHvxVAlign(SDNode *N);
   149: 
   150:   // Function postprocessing.
```
- EN: It declares or implements routines such as selectUndef, getMachineNode, SDValue, keepsLowBits, ... (18 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 selectUndef, getMachineNode, SDValue, keepsLowBits, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 151-173 / 第 151-173 行

```cpp
   151:   void updateAligna();
   152: 
   153:   SmallDenseMap<SDNode *,int> RootWeights;
   154:   SmallDenseMap<SDNode *,int> RootHeights;
   155:   SmallDenseMap<const Value *,int> GAUsesInFunction;
   156:   int getWeight(SDNode *N);
   157:   int getHeight(SDNode *N);
   158:   SDValue getMultiplierForSHL(SDNode *N);
   159:   SDValue factorOutPowerOf2(SDValue V, unsigned Power);
   160:   unsigned getUsesInFunction(const Value *V);
   161:   SDValue balanceSubTree(SDNode *N, bool Factorize = false);
   162:   void rebalanceAddressTrees();
   163: }; // end HexagonDAGToDAGISel
   164: 
   165: class HexagonDAGToDAGISelLegacy : public SelectionDAGISelLegacy {
   166: public:
   167:   static char ID;
   168:   explicit HexagonDAGToDAGISelLegacy(HexagonTargetMachine &tm,
   169:                                      CodeGenOptLevel OptLevel);
   170: };
   171: }
   172: 
   173: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONISELDAGTODAG_H
```
- EN: It declares types such as HexagonDAGToDAGISelLegacy, which carry the state or API of this component. It defines declarative TableGen records like HexagonDAGToDAGISelLegacy; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as updateAligna, getWeight, getHeight, getMultiplierForSHL, ... (9 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, HexagonDAGToDAGISelLegacy, HexagonTargetMachine, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonDAGToDAGISelLegacy 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonDAGToDAGISelLegacy 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 updateAligna, getWeight, getHeight, getMultiplierForSHL, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel, HexagonDAGToDAGISelLegacy, HexagonTargetMachine，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonSubtarget.h, HexagonTargetMachine.h, llvm/CodeGen/SelectionDAG.h, llvm/CodeGen/SelectionDAGISel.h, llvm/Support/CodeGen.h, vector, HexagonGenDAGISel.inc`
- Hexagon symbols / Hexagon 符号: `HexagonISelDAGToDAG, HexagonSubtarget, HexagonTargetMachine, HexagonInstrInfo, HexagonRegisterInfo, HexagonDAGToDAGISel, HexagonGenDAGISel, HexagonDAGToDAGISelLegacy`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
