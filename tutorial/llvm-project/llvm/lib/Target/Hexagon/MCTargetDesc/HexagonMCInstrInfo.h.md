# HexagonMCInstrInfo.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCInstrInfo.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Declares Hexagon instruction semantics, helpers, and packetization-aware queries.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。 重点涉及指令语义与选择。 重点涉及调度或成包。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonMCInstrInfo.cpp - Utility functions on Hexagon MCInsts ------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // Utility functions for Hexagon specific MCInst queries
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #ifndef LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONMCINSTRINFO_H
    14: #define LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONMCINSTRINFO_H
    15: 
    16: #include "llvm/ADT/SmallVector.h"
    17: #include "llvm/ADT/StringRef.h"
    18: #include "llvm/ADT/iterator.h"
    19: #include "llvm/ADT/iterator_range.h"
    20: #include "llvm/MC/MCInst.h"
    21: #include "llvm/Support/MathExtras.h"
    22: #include <cstddef>
    23: #include <cstdint>
    24: 
    25: namespace llvm {
    26: 
    27: class HexagonMCChecker;
    28: class MCContext;
    29: class MCExpr;
    30: class MCInstrDesc;
    31: class MCInstrInfo;
    32: class MCRegisterInfo;
    33: class MCSubtargetInfo;
    34: 
    35: class DuplexCandidate {
    36: public:
    37:   unsigned packetIndexI, packetIndexJ, iClass;
    38: 
    39:   DuplexCandidate(unsigned i, unsigned j, unsigned iClass)
    40:       : packetIndexI(i), packetIndexJ(j), iClass(iClass) {}
    41: };
    42: 
    43: namespace Hexagon {
    44: 
    45: class PacketIterator
    46:     : public llvm::iterator_facade_base<
    47:           PacketIterator, std::forward_iterator_tag, const MCInst> {
    48:   MCInstrInfo const &MCII;
    49:   MCInst::const_iterator BundleCurrent;
    50:   MCInst::const_iterator BundleEnd;
```
- EN: It imports headers such as llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/ADT/iterator.h, llvm/ADT/iterator_range.h, ... (8 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonMCChecker, MCContext, MCExpr, MCInstrDesc, ... (9 total), which carry the state or API of this component.
- CN: 这里引入了 llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/ADT/iterator.h, llvm/ADT/iterator_range.h, ... (8 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonMCChecker, MCContext, MCExpr, MCInstrDesc, ... (9 total) 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:   MCInst::const_iterator DuplexCurrent;
    52:   MCInst::const_iterator DuplexEnd;
    53: 
    54: public:
    55:   PacketIterator(MCInstrInfo const &MCII, MCInst const &Inst);
    56:   PacketIterator(MCInstrInfo const &MCII, MCInst const &Inst, std::nullptr_t);
    57: 
    58:   PacketIterator &operator++();
    59:   MCInst const &operator*() const;
    60:   bool operator==(PacketIterator const &Other) const;
    61: };
    62: 
    63: } // end namespace Hexagon
    64: 
    65: namespace HexagonMCInstrInfo {
    66: 
    67: constexpr size_t innerLoopOffset = 0;
    68: constexpr int64_t innerLoopMask = 1 << innerLoopOffset;
    69: 
    70: constexpr size_t outerLoopOffset = 1;
    71: constexpr int64_t outerLoopMask = 1 << outerLoopOffset;
    72: 
    73: // do not reorder memory load/stores by default load/stores are re-ordered
    74: // and by default loads can be re-ordered
    75: constexpr size_t memReorderDisabledOffset = 2;
    76: constexpr int64_t memReorderDisabledMask = 1 << memReorderDisabledOffset;
    77: 
    78: constexpr size_t splitNoMemOrderOffset = 3;
    79: constexpr int64_t splitNoMemorderMask = 1 << splitNoMemOrderOffset;
    80: 
    81: constexpr size_t noShuffleOffset = 4;
    82: constexpr int64_t noShuffleMask = 1 << noShuffleOffset;
    83: 
    84: constexpr size_t bundleInstructionsOffset = 1;
    85: 
    86: void addConstant(MCInst &MI, uint64_t Value, MCContext &Context);
    87: void addConstExtender(MCContext &Context, MCInstrInfo const &MCII, MCInst &MCB,
    88:                       MCInst const &MCI);
    89: 
    90: // Returns a iterator range of instructions in this bundle
    91: iterator_range<Hexagon::PacketIterator>
    92: bundleInstructions(MCInstrInfo const &MCII, MCInst const &MCI);
    93: iterator_range<MCInst::const_iterator> bundleInstructions(MCInst const &MCI);
    94: 
    95: // Returns the number of instructions in the bundle
    96: size_t bundleSize(MCInst const &MCI);
    97: 
    98: // Put the packet in to canonical form, compound, duplex, pad, and shuffle
    99: bool canonicalizePacket(MCInstrInfo const &MCII, MCSubtargetInfo const &STI,
   100:                         MCContext &Context, MCInst &MCB,
```
- EN: It opens namespaces (Hexagon, HexagonMCInstrInfo) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as PacketIterator, addConstant, addConstExtender, bundleInstructions, ... (5 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（Hexagon, HexagonMCInstrInfo），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 PacketIterator, addConstant, addConstExtender, bundleInstructions, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:                         HexagonMCChecker *Checker,
   102:                         bool AttemptCompatibility = false);
   103: bool IsABranchingInst(MCInstrInfo const &MCII, MCSubtargetInfo const &STI,
   104:                       MCInst const &I);
   105: 
   106: // Create a duplex instruction given the two subinsts
   107: MCInst *deriveDuplex(MCContext &Context, unsigned iClass, MCInst const &inst0,
   108:                      MCInst const &inst1);
   109: MCInst deriveExtender(MCInstrInfo const &MCII, MCInst const &Inst,
   110:                       MCOperand const &MO);
   111: 
   112: // Convert this instruction in to a duplex subinst
   113: MCInst deriveSubInst(MCInst const &Inst);
   114: 
   115: // Return the extender for instruction at Index or nullptr if none
   116: MCInst const *extenderForIndex(MCInst const &MCB, size_t Index);
   117: void extendIfNeeded(MCContext &Context, MCInstrInfo const &MCII, MCInst &MCB,
   118:                     MCInst const &MCI);
   119: 
   120: // Return memory access size in bytes
   121: unsigned getMemAccessSize(MCInstrInfo const &MCII, MCInst const &MCI);
   122: 
   123: // Return memory access size
   124: unsigned getAddrMode(MCInstrInfo const &MCII, MCInst const &MCI);
   125: 
   126: MCInstrDesc const &getDesc(MCInstrInfo const &MCII, MCInst const &MCI);
   127: 
   128: // Return which duplex group this instruction belongs to
   129: unsigned getDuplexCandidateGroup(MCInst const &MI);
   130: 
   131: // Return a list of all possible instruction duplex combinations
   132: SmallVector<DuplexCandidate, 8>
   133: getDuplexPossibilties(MCInstrInfo const &MCII, MCSubtargetInfo const &STI,
   134:                       MCInst const &MCB);
   135: unsigned getDuplexRegisterNumbering(MCRegister Reg);
   136: 
   137: MCExpr const &getExpr(MCExpr const &Expr);
   138: 
   139: // Return the index of the extendable operand
   140: unsigned short getExtendableOp(MCInstrInfo const &MCII, MCInst const &MCI);
   141: 
   142: // Return a reference to the extendable operand
   143: MCOperand const &getExtendableOperand(MCInstrInfo const &MCII,
   144:                                       MCInst const &MCI);
   145: 
   146: // Return the implicit alignment of the extendable operand
   147: unsigned getExtentAlignment(MCInstrInfo const &MCII, MCInst const &MCI);
   148: 
   149: // Return the number of logical bits of the extendable operand
   150: unsigned getExtentBits(MCInstrInfo const &MCII, MCInst const &MCI);
```
- EN: It declares or implements routines such as IsABranchingInst, deriveDuplex, deriveExtender, deriveSubInst, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCChecker, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 IsABranchingInst, deriveDuplex, deriveExtender, deriveSubInst, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCChecker，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151: 
   152: // Check if the extendable operand is signed.
   153: bool isExtentSigned(MCInstrInfo const &MCII, MCInst const &MCI);
   154: 
   155: // Return the max value that a constant extendable operand can have
   156: // without being extended.
   157: int getMaxValue(MCInstrInfo const &MCII, MCInst const &MCI);
   158: 
   159: // Return the min value that a constant extendable operand can have
   160: // without being extended.
   161: int getMinValue(MCInstrInfo const &MCII, MCInst const &MCI);
   162: 
   163: // Return instruction name
   164: StringRef getName(MCInstrInfo const &MCII, MCInst const &MCI);
   165: 
   166: // Return the operand index for the new value.
   167: unsigned short getNewValueOp(MCInstrInfo const &MCII, MCInst const &MCI);
   168: 
   169: // Return the operand that consumes or produces a new value.
   170: MCOperand const &getNewValueOperand(MCInstrInfo const &MCII, MCInst const &MCI);
   171: unsigned short getNewValueOp2(MCInstrInfo const &MCII, MCInst const &MCI);
   172: MCOperand const &getNewValueOperand2(MCInstrInfo const &MCII,
   173:                                      MCInst const &MCI);
   174: 
   175: // Return the Hexagon ISA class for the insn.
   176: unsigned getType(MCInstrInfo const &MCII, MCInst const &MCI);
   177: 
   178: /// Return the resources used by this instruction
   179: unsigned getCVIResources(MCInstrInfo const &MCII,
   180:                                       MCSubtargetInfo const &STI,
   181:                                       MCInst const &MCI);
   182: 
   183: /// Return the slots used by the insn.
   184: unsigned getUnits(MCInstrInfo const &MCII, MCSubtargetInfo const &STI,
   185:                   MCInst const &MCI);
   186: unsigned getOtherReservedSlots(MCInstrInfo const &MCII,
   187:                                MCSubtargetInfo const &STI, MCInst const &MCI);
   188: bool hasDuplex(MCInstrInfo const &MCII, MCInst const &MCI);
   189: 
   190: // Does the packet have an extender for the instruction at Index
   191: bool hasExtenderForIndex(MCInst const &MCB, size_t Index);
   192: 
   193: bool hasImmExt(MCInst const &MCI);
   194: 
   195: // Return whether the instruction is a legal new-value producer.
   196: bool hasNewValue(MCInstrInfo const &MCII, MCInst const &MCI);
   197: bool hasNewValue2(MCInstrInfo const &MCII, MCInst const &MCI);
   198: bool hasTmpDst(MCInstrInfo const &MCII, MCInst const &MCI);
   199: bool hasHvxTmp(MCInstrInfo const &MCII, MCInst const &MCI);
   200: unsigned iClassOfDuplexPair(unsigned Ga, unsigned Gb);
```
- EN: It declares or implements routines such as isExtentSigned, getMaxValue, getMinValue, getName, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 isExtentSigned, getMaxValue, getMinValue, getName, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```cpp
   201: 
   202: int64_t minConstant(MCInst const &MCI, size_t Index);
   203: template <unsigned N, unsigned S>
   204: bool inRange(MCInst const &MCI, size_t Index) {
   205:   return isShiftedUInt<N, S>(minConstant(MCI, Index));
   206: }
   207: template <unsigned N, unsigned S>
   208: bool inSRange(MCInst const &MCI, size_t Index) {
   209:   return isShiftedInt<N, S>(minConstant(MCI, Index));
   210: }
   211: template <unsigned N> bool inRange(MCInst const &MCI, size_t Index) {
   212:   return isUInt<N>(minConstant(MCI, Index));
   213: }
   214: 
   215: // Return the instruction at Index
   216: MCInst const &instruction(MCInst const &MCB, size_t Index);
   217: bool isAccumulator(MCInstrInfo const &MCII, MCInst const &MCI);
   218: 
   219: // Returns whether this MCInst is a wellformed bundle
   220: bool isBundle(MCInst const &MCI);
   221: 
   222: // Return whether the insn is an actual insn.
   223: bool isCanon(MCInstrInfo const &MCII, MCInst const &MCI);
   224: bool isCofMax1(MCInstrInfo const &MCII, MCInst const &MCI);
   225: bool isCofRelax1(MCInstrInfo const &MCII, MCInst const &MCI);
   226: bool isCofRelax2(MCInstrInfo const &MCII, MCInst const &MCI);
   227: bool isCompound(MCInstrInfo const &MCII, MCInst const &MCI);
   228: 
   229: // Return whether the instruction needs to be constant extended.
   230: bool isConstExtended(MCInstrInfo const &MCII, MCInst const &MCI);
   231: bool isCVINew(MCInstrInfo const &MCII, MCInst const &MCI);
   232: 
   233: // Is this double register suitable for use in a duplex subinst
   234: bool isDblRegForSubInst(MCRegister Reg);
   235: 
   236: // Is this a duplex instruction
   237: bool isDuplex(MCInstrInfo const &MCII, MCInst const &MCI);
   238: 
   239: // Can these instructions be duplexed
   240: bool isDuplexPair(MCInst const &MIa, MCInst const &MIb);
   241: 
   242: // Can these duplex classes be combine in to a duplex instruction
   243: bool isDuplexPairMatch(unsigned Ga, unsigned Gb);
   244: 
   245: // Return true if the insn may be extended based on the operand value.
   246: bool isExtendable(MCInstrInfo const &MCII, MCInst const &MCI);
   247: 
   248: // Return whether the instruction must be always extended.
   249: bool isExtended(MCInstrInfo const &MCII, MCInst const &MCI);
   250: 
```
- EN: It declares or implements routines such as minConstant, inRange, S>, inSRange, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 minConstant, inRange, S>, inSRange, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 251-300 / 第 251-300 行

```cpp
   251: /// Return whether it is a floating-point insn.
   252: bool isFloat(MCInstrInfo const &MCII, MCInst const &MCI);
   253: 
   254: bool isHVX(MCInstrInfo const &MCII, MCInst const &MCI);
   255: 
   256: // Returns whether this instruction is an immediate extender
   257: bool isImmext(MCInst const &MCI);
   258: 
   259: // Returns whether this bundle is an endloop0
   260: bool isInnerLoop(MCInst const &MCI);
   261: 
   262: // Is this an integer register
   263: bool isIntReg(MCRegister Reg);
   264: 
   265: // Is this register suitable for use in a duplex subinst
   266: bool isIntRegForSubInst(MCRegister Reg);
   267: bool isMemReorderDisabled(MCInst const &MCI);
   268: 
   269: // Return whether the insn is a new-value consumer.
   270: bool isNewValue(MCInstrInfo const &MCII, MCInst const &MCI);
   271: /// Return true if the operand is a new-value store insn.
   272: bool isNewValueStore(MCInstrInfo const &MCII, MCInst const &MCI);
   273: bool isOpExtendable(MCInstrInfo const &MCII, MCInst const &MCI, unsigned short);
   274: 
   275: // Can these two instructions be duplexed
   276: bool isOrderedDuplexPair(MCInstrInfo const &MCII, MCInst const &MIa,
   277:                          bool ExtendedA, MCInst const &MIb, bool ExtendedB,
   278:                          bool bisReversable, MCSubtargetInfo const &STI);
   279: 
   280: // Returns whether this bundle is an endloop1
   281: bool isOuterLoop(MCInst const &MCI);
   282: 
   283: // Return whether this instruction is predicated
   284: bool isPredicated(MCInstrInfo const &MCII, MCInst const &MCI);
   285: bool isPredicateLate(MCInstrInfo const &MCII, MCInst const &MCI);
   286: bool isPredicatedNew(MCInstrInfo const &MCII, MCInst const &MCI);
   287: 
   288: // Return whether the predicate sense is true
   289: bool isPredicatedTrue(MCInstrInfo const &MCII, MCInst const &MCI);
   290: 
   291: // Return true if this is a scalar predicate register.
   292: bool isPredReg(MCRegisterInfo const &MRI, MCRegister Reg);
   293: 
   294: // Returns true if the Ith operand is a predicate register.
   295: bool isPredRegister(MCInstrInfo const &MCII, MCInst const &Inst, unsigned I);
   296: 
   297: // Return whether the insn is a prefix.
   298: bool isPrefix(MCInstrInfo const &MCII, MCInst const &MCI);
   299: 
   300: // Return whether the insn is solo, i.e., cannot be in a packet.
```
- EN: It declares or implements routines such as isFloat, isHVX, isImmext, isInnerLoop, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 isFloat, isHVX, isImmext, isInnerLoop, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-350 / 第 301-350 行

```cpp
   301: bool isSolo(MCInstrInfo const &MCII, MCInst const &MCI);
   302: 
   303: /// Return whether the insn can be packaged only with A and X-type insns.
   304: bool isSoloAX(MCInstrInfo const &MCII, MCInst const &MCI);
   305: 
   306: /// Return whether the insn can be packaged only with an A-type insn in slot #1.
   307: bool isRestrictSlot1AOK(MCInstrInfo const &MCII, MCInst const &MCI);
   308: bool isRestrictNoSlot1Store(MCInstrInfo const &MCII, MCInst const &MCI);
   309: bool isSubInstruction(MCInst const &MCI);
   310: bool isVector(MCInstrInfo const &MCII, MCInst const &MCI);
   311: bool mustExtend(MCExpr const &Expr);
   312: bool mustNotExtend(MCExpr const &Expr);
   313: 
   314: // Returns true if this instruction requires a slot to execute.
   315: bool requiresSlot(MCSubtargetInfo const &STI, MCInst const &MCI);
   316: 
   317: 
   318: // Returns true if \a MCB would require endloop padding.
   319: bool LoopNeedsPadding(MCInst const &MCB);
   320: 
   321: unsigned packetSize(StringRef CPU);
   322: 
   323: // Returns the maximum number of slots available in the given
   324: // subtarget's packets.
   325: unsigned packetSizeSlots(MCSubtargetInfo const &STI);
   326: 
   327: // Returns the number of slots consumed by this packet, considering duplexed
   328: // and compound instructions.
   329: unsigned slotsConsumed(MCInstrInfo const &MCII, MCSubtargetInfo const &STI,
   330:                        MCInst const &MCI);
   331: 
   332: // Pad the bundle with nops to satisfy endloop requirements.
   333: void padEndloop(MCInst &MCI, MCContext &Context);
   334: class PredicateInfo {
   335: public:
   336:   PredicateInfo() : Operand(0), PredicatedTrue(false) {}
   337:   PredicateInfo(MCRegister Register, unsigned Operand, bool PredicatedTrue)
   338:       : Register(Register), Operand(Operand), PredicatedTrue(PredicatedTrue) {}
   339:   bool isPredicated() const;
   340:   MCRegister Register;
   341:   unsigned Operand;
   342:   bool PredicatedTrue;
   343: };
   344: PredicateInfo predicateInfo(MCInstrInfo const &MCII, MCInst const &MCI);
   345: bool prefersSlot3(MCInstrInfo const &MCII, MCInst const &MCI);
   346: 
   347: // Replace the instructions inside MCB, represented by Candidate
   348: void replaceDuplex(MCContext &Context, MCInst &MCI, DuplexCandidate Candidate);
   349: 
   350: bool s27_2_reloc(MCExpr const &Expr);
```
- EN: It declares types such as PredicateInfo, which carry the state or API of this component. It defines declarative TableGen records like PredicateInfo; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as isSolo, isSoloAX, isRestrictSlot1AOK, isRestrictNoSlot1Store, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 PredicateInfo 等类型，用来承载该组件的状态或接口。 这里定义了 PredicateInfo 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 isSolo, isSoloAX, isRestrictSlot1AOK, isRestrictNoSlot1Store, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 351-384 / 第 351-384 行

```cpp
   351: // Marks a bundle as endloop0
   352: void setInnerLoop(MCInst &MCI);
   353: void setMemReorderDisabled(MCInst &MCI);
   354: void setMustExtend(MCExpr const &Expr, bool Val = true);
   355: void setMustNotExtend(MCExpr const &Expr, bool Val = true);
   356: void setS27_2_reloc(MCExpr const &Expr, bool Val = true);
   357: 
   358: // Marks a bundle as endloop1
   359: void setOuterLoop(MCInst &MCI);
   360: 
   361: // Would duplexing this instruction create a requirement to extend
   362: bool subInstWouldBeExtended(MCInst const &potentialDuplex);
   363: unsigned SubregisterBit(MCRegister Consumer, MCRegister Producer,
   364:                         MCRegister Producer2);
   365: 
   366: bool IsVecRegSingle(MCRegister VecReg);
   367: bool IsVecRegPair(MCRegister VecReg);
   368: bool IsReverseVecRegPair(MCRegister VecReg);
   369: bool IsSingleConsumerRefPairProducer(MCRegister Producer, MCRegister Consumer);
   370: 
   371: /// Returns an ordered pair of the constituent register ordinals for
   372: /// each of the elements of \a VecRegPair.  For example, Hexagon::W0 ("v0:1")
   373: /// returns { 0, 1 } and Hexagon::W1 ("v3:2") returns { 3, 2 }.
   374: std::pair<unsigned, unsigned> GetVecRegPairIndices(MCRegister VecRegPair);
   375: 
   376: // Attempt to find and replace compound pairs
   377: void tryCompound(MCInstrInfo const &MCII, MCSubtargetInfo const &STI,
   378:                  MCContext &Context, MCInst &MCI);
   379: 
   380: } // end namespace HexagonMCInstrInfo
   381: 
   382: } // end namespace llvm
   383: 
   384: #endif // LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONMCINSTRINFO_H
```
- EN: It opens namespaces (HexagonMCInstrInfo, llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as setInnerLoop, setMemReorderDisabled, setMustExtend, setMustNotExtend, ... (14 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（HexagonMCInstrInfo, llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 setInnerLoop, setMemReorderDisabled, setMustExtend, setMustNotExtend, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- VLIW packetization / VLIW 成包
- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/ADT/iterator.h, llvm/ADT/iterator_range.h, llvm/MC/MCInst.h, llvm/Support/MathExtras.h, cstddef, cstdint`
- Hexagon symbols / Hexagon 符号: `HexagonMCInstrInfo, HexagonMCChecker`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
