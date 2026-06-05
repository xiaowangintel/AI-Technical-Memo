# HexagonInstrInfo.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonInstrInfo.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): HexagonInstrInfo.h - Hexagon Instruction Information -----*- C++
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。 重点涉及指令语义与选择。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonInstrInfo.h - Hexagon Instruction Information -----*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file contains the Hexagon implementation of the TargetInstrInfo class.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONINSTRINFO_H
    14: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONINSTRINFO_H
    15: 
    16: #include "MCTargetDesc/HexagonBaseInfo.h"
    17: #include "llvm/ADT/ArrayRef.h"
    18: #include "llvm/ADT/SmallVector.h"
    19: #include "llvm/CodeGen/MachineBasicBlock.h"
    20: #include "llvm/CodeGen/TargetInstrInfo.h"
    21: #include "llvm/CodeGen/ValueTypes.h"
    22: #include "llvm/CodeGenTypes/MachineValueType.h"
    23: #include <cstdint>
    24: #include <vector>
    25: 
    26: #include "HexagonRegisterInfo.h"
    27: 
    28: #define GET_INSTRINFO_HEADER
    29: #include "HexagonGenInstrInfo.inc"
    30: 
    31: namespace llvm {
    32: 
    33: class HexagonSubtarget;
    34: class MachineBranchProbabilityInfo;
    35: class MachineFunction;
    36: class MachineInstr;
    37: class MachineOperand;
    38: class TargetRegisterInfo;
    39: 
    40: class HexagonInstrInfo : public HexagonGenInstrInfo {
    41:   const HexagonRegisterInfo RegInfo;
    42:   const HexagonSubtarget &Subtarget;
    43: 
    44:   enum BundleAttribute {
    45:     memShufDisabledMask = 0x4
    46:   };
    47: 
    48:   virtual void anchor();
    49: 
    50: public:
```
- EN: It imports headers such as MCTargetDesc/HexagonBaseInfo.h, llvm/ADT/ArrayRef.h, llvm/ADT/SmallVector.h, llvm/CodeGen/MachineBasicBlock.h, ... (11 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonSubtarget, MachineBranchProbabilityInfo, MachineFunction, MachineInstr, ... (8 total), which carry the state or API of this component.
- CN: 这里引入了 MCTargetDesc/HexagonBaseInfo.h, llvm/ADT/ArrayRef.h, llvm/ADT/SmallVector.h, llvm/CodeGen/MachineBasicBlock.h, ... (11 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonSubtarget, MachineBranchProbabilityInfo, MachineFunction, MachineInstr, ... (8 total) 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:   explicit HexagonInstrInfo(const HexagonSubtarget &ST);
    52: 
    53:   const HexagonRegisterInfo &getRegisterInfo() const { return RegInfo; }
    54: 
    55:   /// TargetInstrInfo overrides.
    56: 
    57:   /// If the specified machine instruction is a direct
    58:   /// load from a stack slot, return the virtual or physical register number of
    59:   /// the destination along with the FrameIndex of the loaded stack slot.  If
    60:   /// not, return 0.  This predicate must return 0 if the instruction has
    61:   /// any side effects other than loading from the stack slot.
    62:   Register isLoadFromStackSlot(const MachineInstr &MI,
    63:                                int &FrameIndex) const override;
    64: 
    65:   /// If the specified machine instruction is a direct
    66:   /// store to a stack slot, return the virtual or physical register number of
    67:   /// the source reg along with the FrameIndex of the loaded stack slot.  If
    68:   /// not, return 0.  This predicate must return 0 if the instruction has
    69:   /// any side effects other than storing to the stack slot.
    70:   Register isStoreToStackSlot(const MachineInstr &MI,
    71:                               int &FrameIndex) const override;
    72: 
    73:   /// Check if the instruction or the bundle of instructions has
    74:   /// load from stack slots. Return the frameindex and machine memory operand
    75:   /// if true.
    76:   bool hasLoadFromStackSlot(
    77:       const MachineInstr &MI,
    78:       SmallVectorImpl<const MachineMemOperand *> &Accesses) const override;
    79: 
    80:   /// Check if the instruction or the bundle of instructions has
    81:   /// store to stack slots. Return the frameindex and machine memory operand
    82:   /// if true.
    83:   bool hasStoreToStackSlot(
    84:       const MachineInstr &MI,
    85:       SmallVectorImpl<const MachineMemOperand *> &Accesses) const override;
    86: 
    87:   /// Analyze the branching code at the end of MBB, returning
    88:   /// true if it cannot be understood (e.g. it's a switch dispatch or isn't
    89:   /// implemented for a target).  Upon success, this returns false and returns
    90:   /// with the following information in various cases:
    91:   ///
    92:   /// 1. If this block ends with no branches (it just falls through to its succ)
    93:   ///    just return false, leaving TBB/FBB null.
    94:   /// 2. If this block ends with only an unconditional branch, it sets TBB to be
    95:   ///    the destination block.
    96:   /// 3. If this block ends with a conditional branch and it falls through to a
    97:   ///    successor block, it sets TBB to be the branch destination block and a
    98:   ///    list of operands that evaluate the condition. These operands can be
    99:   ///    passed to other TargetInstrInfo methods to create new branches.
   100:   /// 4. If this block ends with a conditional branch followed by an
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as HexagonInstrInfo, getRegisterInfo, isLoadFromStackSlot, isStoreToStackSlot, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonSubtarget, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 HexagonInstrInfo, getRegisterInfo, isLoadFromStackSlot, isStoreToStackSlot, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonSubtarget, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   ///    unconditional branch, it returns the 'true' destination in TBB, the
   102:   ///    'false' destination in FBB, and a list of operands that evaluate the
   103:   ///    condition.  These operands can be passed to other TargetInstrInfo
   104:   ///    methods to create new branches.
   105:   ///
   106:   /// Note that removeBranch and insertBranch must be implemented to support
   107:   /// cases where this method returns success.
   108:   ///
   109:   /// If AllowModify is true, then this routine is allowed to modify the basic
   110:   /// block (e.g. delete instructions after the unconditional branch).
   111:   bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
   112:                      MachineBasicBlock *&FBB,
   113:                      SmallVectorImpl<MachineOperand> &Cond,
   114:                      bool AllowModify) const override;
   115: 
   116:   /// Remove the branching code at the end of the specific MBB.
   117:   /// This is only invoked in cases where analyzeBranch returns success. It
   118:   /// returns the number of instructions that were removed.
   119:   unsigned removeBranch(MachineBasicBlock &MBB,
   120:                         int *BytesRemoved = nullptr) const override;
   121: 
   122:   /// Insert branch code into the end of the specified MachineBasicBlock.
   123:   /// The operands to this method are the same as those
   124:   /// returned by analyzeBranch.  This is only invoked in cases where
   125:   /// analyzeBranch returns success. It returns the number of instructions
   126:   /// inserted.
   127:   ///
   128:   /// It is also invoked by tail merging to add unconditional branches in
   129:   /// cases where analyzeBranch doesn't apply because there was no original
   130:   /// branch to analyze.  At least this much must be implemented, else tail
   131:   /// merging needs to be disabled.
   132:   unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
   133:                         MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
   134:                         const DebugLoc &DL,
   135:                         int *BytesAdded = nullptr) const override;
   136: 
   137:   /// Analyze loop L, which must be a single-basic-block loop, and if the
   138:   /// conditions can be understood enough produce a PipelinerLoopInfo object.
   139:   std::unique_ptr<PipelinerLoopInfo>
   140:   analyzeLoopForPipelining(MachineBasicBlock *LoopBB) const override;
   141: 
   142:   /// Return true if it's profitable to predicate
   143:   /// instructions with accumulated instruction latency of "NumCycles"
   144:   /// of the specified basic block, where the probability of the instructions
   145:   /// being executed is given by Probability, and Confidence is a measure
   146:   /// of our confidence that it will be properly predicted.
   147:   bool isProfitableToIfCvt(MachineBasicBlock &MBB, unsigned NumCycles,
   148:                            unsigned ExtraPredCycles,
   149:                            BranchProbability Probability) const override;
   150: 
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as block, removeBranch, insertBranch, analyzeLoopForPipelining, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 block, removeBranch, insertBranch, analyzeLoopForPipelining, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   /// Second variant of isProfitableToIfCvt. This one
   152:   /// checks for the case where two basic blocks from true and false path
   153:   /// of a if-then-else (diamond) are predicated on mutually exclusive
   154:   /// predicates, where the probability of the true path being taken is given
   155:   /// by Probability, and Confidence is a measure of our confidence that it
   156:   /// will be properly predicted.
   157:   bool isProfitableToIfCvt(MachineBasicBlock &TMBB,
   158:                            unsigned NumTCycles, unsigned ExtraTCycles,
   159:                            MachineBasicBlock &FMBB,
   160:                            unsigned NumFCycles, unsigned ExtraFCycles,
   161:                            BranchProbability Probability) const override;
   162: 
   163:   /// Return true if it's profitable for if-converter to duplicate instructions
   164:   /// of specified accumulated instruction latencies in the specified MBB to
   165:   /// enable if-conversion.
   166:   /// The probability of the instructions being executed is given by
   167:   /// Probability, and Confidence is a measure of our confidence that it
   168:   /// will be properly predicted.
   169:   bool isProfitableToDupForIfCvt(MachineBasicBlock &MBB, unsigned NumCycles,
   170:                                  BranchProbability Probability) const override;
   171: 
   172:   /// Emit instructions to copy a pair of physical registers.
   173:   ///
   174:   /// This function should support copies within any legal register class as
   175:   /// well as any cross-class copies created during instruction selection.
   176:   ///
   177:   /// The source and destination registers may overlap, which may require a
   178:   /// careful implementation when multiple copy instructions are required for
   179:   /// large registers. See for example the ARM target.
   180:   void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator I,
   181:                    const DebugLoc &DL, Register DestReg, Register SrcReg,
   182:                    bool KillSrc, bool RenamableDest = false,
   183:                    bool RenamableSrc = false) const override;
   184: 
   185:   /// Store the specified register of the given register class to the specified
   186:   /// stack frame index. The store instruction is to be added to the given
   187:   /// machine basic block before the specified machine instruction. If isKill
   188:   /// is true, the register operand is the last use and must be marked kill.
   189:   void storeRegToStackSlot(
   190:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register SrcReg,
   191:       bool isKill, int FrameIndex, const TargetRegisterClass *RC, Register VReg,
   192:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
   193: 
   194:   /// Load the specified register of the given register class from the specified
   195:   /// stack frame index. The load instruction is to be added to the given
   196:   /// machine basic block before the specified machine instruction.
   197:   void loadRegFromStackSlot(
   198:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
   199:       Register DestReg, int FrameIndex, const TargetRegisterClass *RC,
   200:       Register VReg, unsigned SubReg = 0,
```
- EN: It declares or implements routines such as else, isProfitableToDupForIfCvt, copyPhysReg, storeRegToStackSlot, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 else, isProfitableToDupForIfCvt, copyPhysReg, storeRegToStackSlot 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```cpp
   201:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
   202: 
   203:   /// This function is called for all pseudo instructions
   204:   /// that remain after register allocation. Many pseudo instructions are
   205:   /// created to help register allocation. This is the place to convert them
   206:   /// into real instructions. The target can edit MI in place, or it can insert
   207:   /// new instructions and erase MI. The function should return true if
   208:   /// anything was changed.
   209:   bool expandPostRAPseudo(MachineInstr &MI) const override;
   210: 
   211:   /// Get the base register and byte offset of a load/store instr.
   212:   bool getMemOperandsWithOffsetWidth(
   213:       const MachineInstr &LdSt,
   214:       SmallVectorImpl<const MachineOperand *> &BaseOps, int64_t &Offset,
   215:       bool &OffsetIsScalable, LocationSize &Width,
   216:       const TargetRegisterInfo *TRI) const override;
   217: 
   218:   /// Reverses the branch condition of the specified condition list,
   219:   /// returning false on success and true if it cannot be reversed.
   220:   bool reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond)
   221:         const override;
   222: 
   223:   /// Insert a noop into the instruction stream at the specified point.
   224:   void insertNoop(MachineBasicBlock &MBB,
   225:                   MachineBasicBlock::iterator MI) const override;
   226: 
   227:   /// Returns true if the instruction is already predicated.
   228:   bool isPredicated(const MachineInstr &MI) const override;
   229: 
   230:   /// Return true for post-incremented instructions.
   231:   bool isPostIncrement(const MachineInstr &MI) const override;
   232: 
   233:   /// Convert the instruction into a predicated instruction.
   234:   /// It returns true if the operation was successful.
   235:   bool PredicateInstruction(MachineInstr &MI,
   236:                             ArrayRef<MachineOperand> Cond) const override;
   237: 
   238:   /// Returns true if the first specified predicate
   239:   /// subsumes the second, e.g. GE subsumes GT.
   240:   bool SubsumesPredicate(ArrayRef<MachineOperand> Pred1,
   241:                          ArrayRef<MachineOperand> Pred2) const override;
   242: 
   243:   /// If the specified instruction defines any predicate
   244:   /// or condition code register(s) used for predication, returns true as well
   245:   /// as the definition predicate(s) by reference.
   246:   bool ClobbersPredicate(MachineInstr &MI, std::vector<MachineOperand> &Pred,
   247:                          bool SkipDead) const override;
   248: 
   249:   /// Return true if the specified instruction can be predicated.
   250:   /// By default, this returns true for every instruction with a
```
- EN: It declares or implements routines such as expandPostRAPseudo, getMemOperandsWithOffsetWidth, reverseBranchCondition, insertNoop, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 expandPostRAPseudo, getMemOperandsWithOffsetWidth, reverseBranchCondition, insertNoop, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 251-300 / 第 251-300 行

```cpp
   251:   /// PredicateOperand.
   252:   bool isPredicable(const MachineInstr &MI) const override;
   253: 
   254:   /// Test if the given instruction should be considered a scheduling boundary.
   255:   /// This primarily includes labels and terminators.
   256:   bool isSchedulingBoundary(const MachineInstr &MI,
   257:                             const MachineBasicBlock *MBB,
   258:                             const MachineFunction &MF) const override;
   259: 
   260:   /// Measure the specified inline asm to determine an approximation of its
   261:   /// length.
   262:   unsigned getInlineAsmLength(
   263:     const char *Str,
   264:     const MCAsmInfo &MAI,
   265:     const TargetSubtargetInfo *STI = nullptr) const override;
   266: 
   267:   /// Allocate and return a hazard recognizer to use for this target when
   268:   /// scheduling the machine instructions after register allocation.
   269:   ScheduleHazardRecognizer*
   270:   CreateTargetPostRAHazardRecognizer(const InstrItineraryData *II,
   271:                                      const ScheduleDAG *DAG) const override;
   272: 
   273:   /// For a comparison instruction, return the source registers
   274:   /// in SrcReg and SrcReg2 if having two register operands, and the value it
   275:   /// compares against in CmpValue. Return true if the comparison instruction
   276:   /// can be analyzed.
   277:   bool analyzeCompare(const MachineInstr &MI, Register &SrcReg,
   278:                       Register &SrcReg2, int64_t &Mask,
   279:                       int64_t &Value) const override;
   280: 
   281:   /// Compute the instruction latency of a given instruction.
   282:   /// If the instruction has higher cost when predicated, it's returned via
   283:   /// PredCost.
   284:   unsigned getInstrLatency(const InstrItineraryData *ItinData,
   285:                            const MachineInstr &MI,
   286:                            unsigned *PredCost = nullptr) const override;
   287: 
   288:   /// Create machine specific model for scheduling.
   289:   DFAPacketizer *
   290:   CreateTargetScheduleState(const TargetSubtargetInfo &STI) const override;
   291: 
   292:   // Sometimes, it is possible for the target
   293:   // to tell, even without aliasing information, that two MIs access different
   294:   // memory addresses. This function returns true if two MIs access different
   295:   // memory addresses and false otherwise.
   296:   bool
   297:   areMemAccessesTriviallyDisjoint(const MachineInstr &MIa,
   298:                                   const MachineInstr &MIb) const override;
   299: 
   300:   /// For instructions with a base and offset, return the position of the
```
- EN: It declares or implements routines such as isPredicable, isSchedulingBoundary, getInlineAsmLength, CreateTargetPostRAHazardRecognizer, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 isPredicable, isSchedulingBoundary, getInlineAsmLength, CreateTargetPostRAHazardRecognizer, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-350 / 第 301-350 行

```cpp
   301:   /// base register and offset operands.
   302:   bool getBaseAndOffsetPosition(const MachineInstr &MI, unsigned &BasePos,
   303:                                 unsigned &OffsetPos) const override;
   304: 
   305:   /// If the instruction is an increment of a constant value, return the amount.
   306:   bool getIncrementValue(const MachineInstr &MI, int &Value) const override;
   307: 
   308:   /// getOperandLatency - Compute and return the use operand latency of a given
   309:   /// pair of def and use.
   310:   /// In most cases, the static scheduling itinerary was enough to determine the
   311:   /// operand latency. But it may not be possible for instructions with variable
   312:   /// number of defs / uses.
   313:   ///
   314:   /// This is a raw interface to the itinerary that may be directly overridden
   315:   /// by a target. Use computeOperandLatency to get the best estimate of
   316:   /// latency.
   317:   std::optional<unsigned> getOperandLatency(const InstrItineraryData *ItinData,
   318:                                             const MachineInstr &DefMI,
   319:                                             unsigned DefIdx,
   320:                                             const MachineInstr &UseMI,
   321:                                             unsigned UseIdx) const override;
   322: 
   323:   /// Decompose the machine operand's target flags into two values - the direct
   324:   /// target flag value and any of bit flags that are applied.
   325:   std::pair<unsigned, unsigned>
   326:   decomposeMachineOperandsTargetFlags(unsigned TF) const override;
   327: 
   328:   /// Return an array that contains the direct target flag values and their
   329:   /// names.
   330:   ///
   331:   /// MIR Serialization is able to serialize only the target flags that are
   332:   /// defined by this method.
   333:   ArrayRef<std::pair<unsigned, const char *>>
   334:   getSerializableDirectMachineOperandTargetFlags() const override;
   335: 
   336:   /// Return an array that contains the bitmask target flag values and their
   337:   /// names.
   338:   ///
   339:   /// MIR Serialization is able to serialize only the target flags that are
   340:   /// defined by this method.
   341:   ArrayRef<std::pair<unsigned, const char *>>
   342:   getSerializableBitmaskMachineOperandTargetFlags() const override;
   343: 
   344:   bool isTailCall(const MachineInstr &MI) const override;
   345:   bool isAsCheapAsAMove(const MachineInstr &MI) const override;
   346: 
   347:   // Return true if the instruction should be sunk by MachineSink.
   348:   // MachineSink determines on its own whether the instruction is safe to sink;
   349:   // this gives the target a hook to override the default behavior with regards
   350:   // to which instructions should be sunk.
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as getBaseAndOffsetPosition, getIncrementValue, getOperandLatency, decomposeMachineOperandsTargetFlags, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 getBaseAndOffsetPosition, getIncrementValue, getOperandLatency, decomposeMachineOperandsTargetFlags, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 351-400 / 第 351-400 行

```cpp
   351:   bool shouldSink(const MachineInstr &MI) const override;
   352: 
   353:   /// HexagonInstrInfo specifics.
   354: 
   355:   Register createVR(MachineFunction *MF, MVT VT) const;
   356:   MachineInstr *findLoopInstr(MachineBasicBlock *BB, unsigned EndLoopOp,
   357:                               MachineBasicBlock *TargetBB,
   358:                               SmallPtrSet<MachineBasicBlock *, 8> &Visited) const;
   359: 
   360:   bool isAbsoluteSet(const MachineInstr &MI) const;
   361:   bool isAccumulator(const MachineInstr &MI) const;
   362:   bool isAddrModeWithOffset(const MachineInstr &MI) const;
   363:   bool isBaseImmOffset(const MachineInstr &MI) const;
   364:   bool isComplex(const MachineInstr &MI) const;
   365:   bool isCompoundBranchInstr(const MachineInstr &MI) const;
   366:   bool isConstExtended(const MachineInstr &MI) const;
   367:   bool isDeallocRet(const MachineInstr &MI) const;
   368:   bool isDependent(const MachineInstr &ProdMI,
   369:                    const MachineInstr &ConsMI) const;
   370:   bool isDotCurInst(const MachineInstr &MI) const;
   371:   bool isDotNewInst(const MachineInstr &MI) const;
   372:   bool isDuplexPair(const MachineInstr &MIa, const MachineInstr &MIb) const;
   373:   bool isEndLoopN(unsigned Opcode) const;
   374:   bool isExpr(unsigned OpType) const;
   375:   bool isExtendable(const MachineInstr &MI) const;
   376:   bool isExtended(const MachineInstr &MI) const;
   377:   bool isFloat(const MachineInstr &MI) const;
   378:   bool isHVXMemWithAIndirect(const MachineInstr &I,
   379:                              const MachineInstr &J) const;
   380:   bool isIndirectCall(const MachineInstr &MI) const;
   381:   bool isIndirectL4Return(const MachineInstr &MI) const;
   382:   bool isJumpR(const MachineInstr &MI) const;
   383:   bool isJumpWithinBranchRange(const MachineInstr &MI, unsigned offset) const;
   384:   bool isLateSourceInstr(const MachineInstr &MI) const;
   385:   bool isLoopN(const MachineInstr &MI) const;
   386:   bool isMemOp(const MachineInstr &MI) const;
   387:   bool isNewValue(const MachineInstr &MI) const;
   388:   bool isNewValue(unsigned Opcode) const;
   389:   bool isNewValueInst(const MachineInstr &MI) const;
   390:   bool isNewValueJump(const MachineInstr &MI) const;
   391:   bool isNewValueJump(unsigned Opcode) const;
   392:   bool isNewValueStore(const MachineInstr &MI) const;
   393:   bool isNewValueStore(unsigned Opcode) const;
   394:   bool isOperandExtended(const MachineInstr &MI, unsigned OperandNum) const;
   395:   bool isPredicatedNew(const MachineInstr &MI) const;
   396:   bool isPredicatedNew(unsigned Opcode) const;
   397:   bool isPredicatedTrue(const MachineInstr &MI) const;
   398:   bool isPredicatedTrue(unsigned Opcode) const;
   399:   bool isPredicated(unsigned Opcode) const;
   400:   bool isPredicateLate(unsigned Opcode) const;
```
- EN: It declares or implements routines such as shouldSink, createVR, findLoopInstr, isAbsoluteSet, ... (37 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 shouldSink, createVR, findLoopInstr, isAbsoluteSet, ... (37 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 401-450 / 第 401-450 行

```cpp
   401:   bool isPredictedTaken(unsigned Opcode) const;
   402:   bool isPureSlot0(const MachineInstr &MI) const;
   403:   bool isRestrictNoSlot1Store(const MachineInstr &MI) const;
   404:   bool isSaveCalleeSavedRegsCall(const MachineInstr &MI) const;
   405:   bool isSignExtendingLoad(const MachineInstr &MI) const;
   406:   bool isSolo(const MachineInstr &MI) const;
   407:   bool isSpillPredRegOp(const MachineInstr &MI) const;
   408:   bool isTC1(const MachineInstr &MI) const;
   409:   bool isTC2(const MachineInstr &MI) const;
   410:   bool isTC2Early(const MachineInstr &MI) const;
   411:   bool isTC4x(const MachineInstr &MI) const;
   412:   bool isToBeScheduledASAP(const MachineInstr &MI1,
   413:                            const MachineInstr &MI2) const;
   414:   bool isHVXVec(const MachineInstr &MI) const;
   415:   bool isValidAutoIncImm(const EVT VT, const int Offset) const;
   416:   bool isValidOffset(unsigned Opcode, int Offset,
   417:                      const TargetRegisterInfo *TRI, bool Extend = true) const;
   418:   bool isVecAcc(const MachineInstr &MI) const;
   419:   bool isVecALU(const MachineInstr &MI) const;
   420:   bool isVecUsableNextPacket(const MachineInstr &ProdMI,
   421:                              const MachineInstr &ConsMI) const;
   422:   bool isZeroExtendingLoad(const MachineInstr &MI) const;
   423: 
   424:   bool addLatencyToSchedule(const MachineInstr &MI1,
   425:                             const MachineInstr &MI2) const;
   426:   bool canExecuteInBundle(const MachineInstr &First,
   427:                           const MachineInstr &Second) const;
   428:   bool doesNotReturn(const MachineInstr &CallMI) const;
   429:   bool hasEHLabel(const MachineBasicBlock *B) const;
   430:   bool hasNonExtEquivalent(const MachineInstr &MI) const;
   431:   bool hasPseudoInstrPair(const MachineInstr &MI) const;
   432:   bool hasUncondBranch(const MachineBasicBlock *B) const;
   433:   bool mayBeCurLoad(const MachineInstr &MI) const;
   434:   bool mayBeNewStore(const MachineInstr &MI) const;
   435:   bool producesStall(const MachineInstr &ProdMI,
   436:                      const MachineInstr &ConsMI) const;
   437:   bool producesStall(const MachineInstr &MI,
   438:                      MachineBasicBlock::const_instr_iterator MII) const;
   439:   bool predCanBeUsedAsDotNew(const MachineInstr &MI, Register PredReg) const;
   440:   bool PredOpcodeHasJMP_c(unsigned Opcode) const;
   441:   bool predOpcodeHasNot(ArrayRef<MachineOperand> Cond) const;
   442: 
   443:   unsigned getAddrMode(const MachineInstr &MI) const;
   444:   MachineOperand *getBaseAndOffset(const MachineInstr &MI, int64_t &Offset,
   445:                                    LocationSize &AccessSize) const;
   446:   SmallVector<MachineInstr*,2> getBranchingInstrs(MachineBasicBlock& MBB) const;
   447:   unsigned getCExtOpNum(const MachineInstr &MI) const;
   448:   HexagonII::CompoundGroup
   449:   getCompoundCandidateGroup(const MachineInstr &MI) const;
   450:   unsigned getCompoundOpcode(const MachineInstr &GA,
```
- EN: It declares or implements routines such as isPredictedTaken, isPureSlot0, isRestrictNoSlot1Store, isSaveCalleeSavedRegsCall, ... (37 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isPredictedTaken, isPureSlot0, isRestrictNoSlot1Store, isSaveCalleeSavedRegsCall, ... (37 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

### Lines 451-500 / 第 451-500 行

```cpp
   451:                              const MachineInstr &GB) const;
   452:   int getDuplexOpcode(const MachineInstr &MI, bool ForBigCore = true) const;
   453:   int getCondOpcode(int Opc, bool sense) const;
   454:   int getDotCurOp(const MachineInstr &MI) const;
   455:   int getNonDotCurOp(const MachineInstr &MI) const;
   456:   int getDotNewOp(const MachineInstr &MI) const;
   457:   int getDotNewPredJumpOp(const MachineInstr &MI,
   458:                           const MachineBranchProbabilityInfo *MBPI) const;
   459:   int getDotNewPredOp(const MachineInstr &MI,
   460:                       const MachineBranchProbabilityInfo *MBPI) const;
   461:   int getDotOldOp(const MachineInstr &MI) const;
   462:   HexagonII::SubInstructionGroup getDuplexCandidateGroup(const MachineInstr &MI)
   463:                                                          const;
   464:   short getEquivalentHWInstr(const MachineInstr &MI) const;
   465:   unsigned getInstrTimingClassLatency(const InstrItineraryData *ItinData,
   466:                                       const MachineInstr &MI) const;
   467:   bool getInvertedPredSense(SmallVectorImpl<MachineOperand> &Cond) const;
   468:   unsigned getInvertedPredicatedOpcode(const int Opc) const;
   469:   int getMaxValue(const MachineInstr &MI) const;
   470:   unsigned getMemAccessSize(const MachineInstr &MI) const;
   471:   int getMinValue(const MachineInstr &MI) const;
   472:   short getNonExtOpcode(const MachineInstr &MI) const;
   473:   bool getPredReg(ArrayRef<MachineOperand> Cond, Register &PredReg,
   474:                   unsigned &PredRegPos, RegState &PredRegFlags) const;
   475:   short getPseudoInstrPair(const MachineInstr &MI) const;
   476:   short getRegForm(const MachineInstr &MI) const;
   477:   unsigned getSize(const MachineInstr &MI) const;
   478:   uint64_t getType(const MachineInstr &MI) const;
   479:   InstrStage::FuncUnits getUnits(const MachineInstr &MI) const;
   480: 
   481:   MachineBasicBlock::instr_iterator expandVGatherPseudo(MachineInstr &MI) const;
   482: 
   483:   /// getInstrTimingClassLatency - Compute the instruction latency of a given
   484:   /// instruction using Timing Class information, if available.
   485:   unsigned nonDbgBBSize(const MachineBasicBlock *BB) const;
   486:   unsigned nonDbgBundleSize(MachineBasicBlock::const_iterator BundleHead) const;
   487: 
   488:   void immediateExtend(MachineInstr &MI) const;
   489:   bool invertAndChangeJumpTarget(MachineInstr &MI,
   490:                                  MachineBasicBlock *NewTarget) const;
   491:   void genAllInsnTimingClasses(MachineFunction &MF) const;
   492:   bool reversePredSense(MachineInstr &MI) const;
   493:   unsigned reversePrediction(unsigned Opcode) const;
   494:   bool validateBranchCond(const ArrayRef<MachineOperand> &Cond) const;
   495: 
   496:   void setBundleNoShuf(MachineBasicBlock::instr_iterator MIB) const;
   497:   bool getBundleNoShuf(const MachineInstr &MIB) const;
   498: 
   499:   // When TinyCore with Duplexes is enabled, this function is used to translate
   500:   // tiny-instructions to big-instructions and vice versa to get the slot
```
- EN: It declares or implements routines such as getDuplexOpcode, getCondOpcode, getDotCurOp, getNonDotCurOp, ... (34 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getDuplexOpcode, getCondOpcode, getDotCurOp, getNonDotCurOp, ... (34 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

### Lines 501-550 / 第 501-550 行

```cpp
   501:   // consumption.
   502:   void changeDuplexOpcode(MachineBasicBlock::instr_iterator MII,
   503:                           bool ToBigInstrs) const;
   504:   void translateInstrsForDup(MachineFunction &MF,
   505:                              bool ToBigInstrs = true) const;
   506:   void translateInstrsForDup(MachineBasicBlock::instr_iterator MII,
   507:                              bool ToBigInstrs) const;
   508:   bool useMachineCombiner() const override { return true; }
   509:   bool isAssociativeAndCommutative(const MachineInstr &Inst,
   510:                                    bool Invert) const override;
   511: 
   512:   // Addressing mode relations.
   513:   short changeAddrMode_abs_io(short Opc) const;
   514:   short changeAddrMode_io_abs(short Opc) const;
   515:   short changeAddrMode_io_pi(short Opc) const;
   516:   short changeAddrMode_io_rr(short Opc) const;
   517:   short changeAddrMode_pi_io(short Opc) const;
   518:   short changeAddrMode_rr_io(short Opc) const;
   519:   short changeAddrMode_rr_ur(short Opc) const;
   520:   short changeAddrMode_ur_rr(short Opc) const;
   521: 
   522:   short changeAddrMode_abs_io(const MachineInstr &MI) const {
   523:     return changeAddrMode_abs_io(MI.getOpcode());
   524:   }
   525:   short changeAddrMode_io_abs(const MachineInstr &MI) const {
   526:     return changeAddrMode_io_abs(MI.getOpcode());
   527:   }
   528:   short changeAddrMode_io_rr(const MachineInstr &MI) const {
   529:     return changeAddrMode_io_rr(MI.getOpcode());
   530:   }
   531:   short changeAddrMode_rr_io(const MachineInstr &MI) const {
   532:     return changeAddrMode_rr_io(MI.getOpcode());
   533:   }
   534:   short changeAddrMode_rr_ur(const MachineInstr &MI) const {
   535:     return changeAddrMode_rr_ur(MI.getOpcode());
   536:   }
   537:   short changeAddrMode_ur_rr(const MachineInstr &MI) const {
   538:     return changeAddrMode_ur_rr(MI.getOpcode());
   539:   }
   540: 
   541:   MCInst getNop() const override;
   542:   bool isQFPMul(const MachineInstr *MF) const;
   543: 
   544:   // Check if the instruction uses a qf32/qf16 operand.
   545:   // 'Index' specifies the input operand number (1..3). If 0, check any.
   546:   bool usesQF32Operand(MachineInstr *MI, unsigned Index = 0) const;
   547:   bool usesQF16Operand(MachineInstr *MI, unsigned Index = 0) const;
   548:   bool usesQFOperand(MachineInstr *MI, unsigned Index = 0) const;
   549: 
   550:   // Check if the instruction has a qf32/qf16 output.
```
- EN: It declares or implements routines such as changeDuplexOpcode, translateInstrsForDup, useMachineCombiner, isAssociativeAndCommutative, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 changeDuplexOpcode, translateInstrsForDup, useMachineCombiner, isAssociativeAndCommutative, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 551-565 / 第 551-565 行

```cpp
   551:   bool isQFP32Instr(MachineInstr *MI) const;
   552:   bool isQFP16Instr(MachineInstr *MI) const;
   553:   bool isQFPInstr(MachineInstr *MI) const;
   554: };
   555: 
   556: /// \brief Create RegSubRegPair from a register MachineOperand
   557: inline TargetInstrInfo::RegSubRegPair
   558: getRegSubRegPair(const MachineOperand &O) {
   559:   assert(O.isReg());
   560:   return TargetInstrInfo::RegSubRegPair(O.getReg(), O.getSubReg());
   561: }
   562: 
   563: } // end namespace llvm
   564: 
   565: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONINSTRINFO_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as isQFP32Instr, isQFP16Instr, isQFPInstr, getRegSubRegPair, ... (6 total), translating Hexagon-specific policy into reusable code paths. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 isQFP32Instr, isQFP16Instr, isQFPInstr, getRegSubRegPair, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 断言表达了该后端在当前阶段要求满足的不变量。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制
- ABI and stack-frame lowering / ABI 与栈帧下沉

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonBaseInfo.h, llvm/ADT/ArrayRef.h, llvm/ADT/SmallVector.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/TargetInstrInfo.h, llvm/CodeGen/ValueTypes.h, llvm/CodeGenTypes/MachineValueType.h, cstdint, vector, HexagonRegisterInfo.h, ... (11 total)`
- Hexagon symbols / Hexagon 符号: `HexagonInstrInfo, HexagonBaseInfo, HexagonRegisterInfo, HexagonGenInstrInfo, HexagonSubtarget, HexagonII`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
