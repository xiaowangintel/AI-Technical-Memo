# HexagonMCChecker.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCChecker.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Contains the vector-pair-registers with the even number
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。 重点涉及寄存器模型。 重点涉及 HVX/向量处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===- HexagonMCChecker.h - Instruction bundle checking ---------*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This implements the checking of insns inside a bundle according to the
    10: // packet constraint rules of the Hexagon ISA.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #ifndef LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONMCCHECKER_H
    15: #define LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONMCCHECKER_H
    16: 
    17: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    18: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    19: #include "llvm/ADT/DenseMap.h"
    20: #include "llvm/Support/SMLoc.h"
    21: #include <set>
    22: #include <utility>
    23: 
    24: namespace llvm {
    25: 
```
- EN: It imports headers such as MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/DenseMap.h, llvm/Support/SMLoc.h, ... (6 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/DenseMap.h, llvm/Support/SMLoc.h, ... (6 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 26-50 / 第 26-50 行

```cpp
    26: class MCContext;
    27: class MCInst;
    28: class MCInstrInfo;
    29: class MCRegisterInfo;
    30: class MCSubtargetInfo;
    31: 
    32: /// Check for a valid bundle.
    33: class HexagonMCChecker {
    34:   MCContext &Context;
    35:   MCInst &MCB;
    36:   const MCRegisterInfo &RI;
    37:   MCInstrInfo const &MCII;
    38:   MCSubtargetInfo const &STI;
    39:   bool ReportErrors;
    40: 
    41:   /// Set of definitions: register #, if predicated, if predicated true.
    42:   using PredSense = std::pair<MCRegister, bool>;
    43:   static const PredSense Unconditional;
    44:   using PredSet = std::multiset<PredSense>;
    45:   using PredSetIterator = std::multiset<PredSense>::iterator;
    46: 
    47:   using DefsIterator = DenseMap<MCRegister, PredSet>::iterator;
    48:   DenseMap<MCRegister, PredSet> Defs;
    49: 
    50:   /// Set of weak definitions whose clashes should be enforced selectively.
```
- EN: It declares types such as MCContext, MCInst, MCInstrInfo, MCRegisterInfo, ... (6 total), which carry the state or API of this component. It defines declarative TableGen records like MCContext, MCInst, MCInstrInfo, MCRegisterInfo, MCSubtargetInfo, ... (6 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCChecker, showing how the code connects to sibling backend components.
- CN: 这里声明了 MCContext, MCInst, MCInstrInfo, MCRegisterInfo, ... (6 total) 等类型，用来承载该组件的状态或接口。 这里定义了 MCContext, MCInst, MCInstrInfo, MCRegisterInfo, MCSubtargetInfo, ... (6 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCChecker，说明了它与同级后端组件的连接关系。

### Lines 51-75 / 第 51-75 行

```cpp
    51:   using SoftDefsIterator = std::set<MCRegister>::iterator;
    52:   std::set<MCRegister> SoftDefs;
    53: 
    54:   /// Set of temporary definitions not committed to the register file.
    55:   using TmpDefsIterator = std::set<MCRegister>::iterator;
    56:   std::set<MCRegister> TmpDefs;
    57: 
    58:   /// Set of new predicates used.
    59:   using NewPredsIterator = std::set<MCRegister>::iterator;
    60:   std::set<MCRegister> NewPreds;
    61: 
    62:   /// Set of predicates defined late.
    63:   using LatePredsIterator = std::multiset<MCRegister>::iterator;
    64:   std::multiset<MCRegister> LatePreds;
    65: 
    66:   /// Set of uses.
    67:   using UsesIterator = std::set<MCRegister>::iterator;
    68:   std::set<MCRegister> Uses;
    69: 
    70:   /// Pre-defined set of read-only registers.
    71:   using ReadOnlyIterator = std::set<MCRegister>::iterator;
    72:   std::set<MCRegister> ReadOnly;
    73: 
    74:   // Contains the vector-pair-registers with the even number
    75:   // first ("v0:1", e.g.) used/def'd in this packet.
```
- EN: This range exposes interface declarations used by other Hexagon backend translation units.
- CN: 这一段暴露了供其他 Hexagon 后端编译单元使用的接口声明。

### Lines 76-100 / 第 76-100 行

```cpp
    76:   std::set<MCRegister> ReversePairs;
    77: 
    78:   void init();
    79:   void init(MCInst const &);
    80:   void initReg(MCInst const &, MCRegister, MCRegister &PredReg, bool &isTrue);
    81: 
    82:   bool registerUsed(MCRegister Register);
    83: 
    84:   /// \return a tuple of: pointer to the producer instruction or nullptr if
    85:   /// none was found, the operand index, and the PredicateInfo for the
    86:   /// producer.
    87:   std::tuple<MCInst const *, unsigned, HexagonMCInstrInfo::PredicateInfo>
    88:   registerProducer(MCRegister Register,
    89:                    HexagonMCInstrInfo::PredicateInfo Predicated);
    90: 
    91:   // Checks performed.
    92:   bool checkBranches();
    93:   bool checkPredicates();
    94:   bool checkNewValues();
    95:   bool checkRegisters();
    96:   bool checkRegistersReadOnly();
    97:   void checkRegisterCurDefs();
    98:   bool checkSolo();
    99:   bool checkShuffle();
   100:   bool checkSlots();
```
- EN: It declares or implements routines such as init, initReg, registerUsed, registerProducer, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 init, initReg, registerUsed, registerProducer, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 101-125 / 第 101-125 行

```cpp
   101:   bool checkAXOK();
   102:   bool checkHWLoop();
   103:   bool checkCOFMax1();
   104:   bool checkLegalVecRegPair();
   105:   bool checkValidTmpDst();
   106:   bool checkHVXAccum();
   107: 
   108:   static void compoundRegisterMap(unsigned &);
   109: 
   110:   bool isLoopRegister(MCRegister R) const {
   111:     return (Hexagon::SA0 == R || Hexagon::LC0 == R || Hexagon::SA1 == R ||
   112:             Hexagon::LC1 == R);
   113:   }
   114: 
   115: public:
   116:   explicit HexagonMCChecker(MCContext &Context, MCInstrInfo const &MCII,
   117:                             MCSubtargetInfo const &STI, MCInst &mcb,
   118:                             const MCRegisterInfo &ri, bool ReportErrors = true);
   119:   explicit HexagonMCChecker(HexagonMCChecker const &Check,
   120:                             MCSubtargetInfo const &STI, bool CopyReportErrors);
   121: 
   122:   bool check(bool FullCheck = true);
   123:   void reportErrorRegisters(MCRegister Register);
   124:   void reportErrorNewValue(MCRegister Register);
   125:   void reportError(SMLoc Loc, Twine const &Msg);
```
- EN: It declares or implements routines such as checkAXOK, checkHWLoop, checkCOFMax1, checkLegalVecRegPair, ... (13 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonMCChecker, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 checkAXOK, checkHWLoop, checkCOFMax1, checkLegalVecRegPair, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonMCChecker，说明了它与同级后端组件的连接关系。

### Lines 126-134 / 第 126-134 行

```cpp
   126:   void reportNote(SMLoc Loc, Twine const &Msg);
   127:   void reportError(Twine const &Msg);
   128:   void reportWarning(Twine const &Msg);
   129:   void reportBranchErrors();
   130: };
   131: 
   132: } // end namespace llvm
   133: 
   134: #endif // LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONMCCHECKER_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as reportNote, reportError, reportWarning, reportBranchErrors, translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 reportNote, reportError, reportWarning, reportBranchErrors 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/DenseMap.h, llvm/Support/SMLoc.h, set, utility`
- Hexagon symbols / Hexagon 符号: `HexagonMCChecker, HexagonMCInstrInfo, HexagonMCTargetDesc`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
