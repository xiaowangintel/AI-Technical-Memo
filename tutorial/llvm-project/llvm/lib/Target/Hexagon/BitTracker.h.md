# BitTracker.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/BitTracker.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Declares Hexagon bit-level dataflow/value tracking.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- BitTracker.h ---------------------------------------------*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #ifndef LLVM_LIB_TARGET_HEXAGON_BITTRACKER_H
    10: #define LLVM_LIB_TARGET_HEXAGON_BITTRACKER_H
    11: 
    12: #include "llvm/ADT/DenseSet.h"
    13: #include "llvm/ADT/SetVector.h"
    14: #include "llvm/ADT/SmallVector.h"
    15: #include "llvm/CodeGen/MachineInstr.h"
    16: #include "llvm/CodeGen/MachineOperand.h"
    17: #include <map>
    18: #include <queue>
    19: #include <set>
    20: 
    21: namespace llvm {
    22: 
    23: class BitVector;
    24: class ConstantInt;
    25: class MachineRegisterInfo;
    26: class MachineBasicBlock;
    27: class MachineFunction;
    28: class raw_ostream;
    29: class TargetRegisterClass;
    30: class TargetRegisterInfo;
    31: 
    32: struct BitTracker {
    33:   struct BitRef;
    34:   struct RegisterRef;
    35:   struct BitValue;
    36:   struct BitMask;
    37:   struct RegisterCell;
    38:   struct MachineEvaluator;
    39: 
    40:   using BranchTargetList = SetVector<const MachineBasicBlock *>;
    41:   using CellMapType = std::map<unsigned, RegisterCell>;
    42: 
    43:   BitTracker(const MachineEvaluator &E, MachineFunction &F);
    44:   ~BitTracker();
    45: 
    46:   void run();
    47:   void trace(bool On = false) { Trace = On; }
    48:   bool has(unsigned Reg) const;
    49:   const RegisterCell &lookup(unsigned Reg) const;
    50:   RegisterCell get(RegisterRef RR) const;
```
- EN: It imports headers such as llvm/ADT/DenseSet.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, llvm/CodeGen/MachineInstr.h, ... (8 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as BitVector, ConstantInt, MachineRegisterInfo, MachineBasicBlock, ... (15 total), which carry the state or API of this component.
- CN: 这里引入了 llvm/ADT/DenseSet.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, llvm/CodeGen/MachineInstr.h, ... (8 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 BitVector, ConstantInt, MachineRegisterInfo, MachineBasicBlock, ... (15 total) 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:   void put(RegisterRef RR, const RegisterCell &RC);
    52:   void subst(RegisterRef OldRR, RegisterRef NewRR);
    53:   bool reached(const MachineBasicBlock *B) const;
    54:   void visit(const MachineInstr &MI);
    55: 
    56:   void print_cells(raw_ostream &OS) const;
    57: 
    58: private:
    59:   void visitPHI(const MachineInstr &PI);
    60:   void visitNonBranch(const MachineInstr &MI);
    61:   void visitBranchesFrom(const MachineInstr &BI);
    62:   void visitUsesOf(Register Reg);
    63: 
    64:   using CFGEdge = std::pair<int, int>;
    65:   using EdgeSetType = std::set<CFGEdge>;
    66:   using InstrSetType = std::set<const MachineInstr *>;
    67:   using EdgeQueueType = std::queue<CFGEdge>;
    68: 
    69:   // Priority queue of instructions using modified registers, ordered by
    70:   // their relative position in a basic block.
    71:   struct UseQueueType {
    72:     UseQueueType() : Uses(Dist) {}
    73: 
    74:     unsigned size() const {
    75:       return Uses.size();
    76:     }
    77:     bool empty() const {
    78:       return size() == 0;
    79:     }
    80:     MachineInstr *front() const {
    81:       return Uses.top();
    82:     }
    83:     void push(MachineInstr *MI) {
    84:       if (Set.insert(MI).second)
    85:         Uses.push(MI);
    86:     }
    87:     void pop() {
    88:       Set.erase(front());
    89:       Uses.pop();
    90:     }
    91:     void reset() {
    92:       Dist.clear();
    93:     }
    94:   private:
    95:     struct Cmp {
    96:       Cmp(DenseMap<const MachineInstr*,unsigned> &Map) : Dist(Map) {}
    97:       bool operator()(const MachineInstr *MI, const MachineInstr *MJ) const;
    98:       DenseMap<const MachineInstr*,unsigned> &Dist;
    99:     };
   100:     DenseSet<const MachineInstr*> Set; // Set to avoid adding duplicate entries.
```
- EN: It declares types such as UseQueueType, Cmp, which carry the state or API of this component. It declares or implements routines such as put, subst, reached, visit, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 UseQueueType, Cmp 等类型，用来承载该组件的状态或接口。 这里声明或实现了 put, subst, reached, visit, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-150 / 第 101-150 行

```cpp
   101:     DenseMap<const MachineInstr*,unsigned> Dist;
   102:     std::priority_queue<MachineInstr *, std::vector<MachineInstr *>, Cmp> Uses;
   103:   };
   104: 
   105:   void reset();
   106:   void runEdgeQueue(BitVector &BlockScanned);
   107:   void runUseQueue();
   108: 
   109:   const MachineEvaluator &ME;
   110:   MachineFunction &MF;
   111:   MachineRegisterInfo &MRI;
   112:   CellMapType &Map;
   113: 
   114:   EdgeSetType EdgeExec;         // Executable flow graph edges.
   115:   InstrSetType InstrExec;       // Executable instructions.
   116:   UseQueueType UseQ;            // Work queue of register uses.
   117:   EdgeQueueType FlowQ;          // Work queue of CFG edges.
   118:   DenseSet<unsigned> ReachedBB; // Cache of reached blocks.
   119:   bool Trace;                   // Enable tracing for debugging.
   120: };
   121: 
   122: // Abstraction of a reference to bit at position Pos from a register Reg.
   123: struct BitTracker::BitRef {
   124:   BitRef(unsigned R = 0, uint16_t P = 0) : Reg(R), Pos(P) {}
   125: 
   126:   bool operator== (const BitRef &BR) const {
   127:     // If Reg is 0, disregard Pos.
   128:     return Reg == BR.Reg && (Reg == 0 || Pos == BR.Pos);
   129:   }
   130: 
   131:   Register Reg;
   132:   uint16_t Pos;
   133: };
   134: 
   135: // Abstraction of a register reference in MachineOperand.  It contains the
   136: // register number and the subregister index.
   137: // FIXME: Consolidate duplicate definitions of RegisterRef
   138: struct BitTracker::RegisterRef {
   139:   RegisterRef(Register R = 0, unsigned S = 0) : Reg(R), Sub(S) {}
   140:   RegisterRef(const MachineOperand &MO)
   141:       : Reg(MO.getReg()), Sub(MO.getSubReg()) {}
   142: 
   143:   Register Reg;
   144:   unsigned Sub;
   145: };
   146: 
   147: // Value that a single bit can take.  This is outside of the context of
   148: // any register, it is more of an abstraction of the two-element set of
   149: // possible bit values.  One extension here is the "Ref" type, which
   150: // indicates that this bit takes the same value as the bit described by
```
- EN: It declares types such as BitTracker::BitRef, BitTracker::RegisterRef, which carry the state or API of this component. It declares or implements routines such as reset, runEdgeQueue, runUseQueue, BitRef, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 BitTracker::BitRef, BitTracker::RegisterRef 等类型，用来承载该组件的状态或接口。 这里声明或实现了 reset, runEdgeQueue, runUseQueue, BitRef, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```cpp
   151: // RefInfo.
   152: struct BitTracker::BitValue {
   153:   enum ValueType {
   154:     Top,    // Bit not yet defined.
   155:     Zero,   // Bit = 0.
   156:     One,    // Bit = 1.
   157:     Ref     // Bit value same as the one described in RefI.
   158:     // Conceptually, there is no explicit "bottom" value: the lattice's
   159:     // bottom will be expressed as a "ref to itself", which, in the context
   160:     // of registers, could be read as "this value of this bit is defined by
   161:     // this bit".
   162:     // The ordering is:
   163:     //   x <= Top,
   164:     //   Self <= x, where "Self" is "ref to itself".
   165:     // This makes the value lattice different for each virtual register
   166:     // (even for each bit in the same virtual register), since the "bottom"
   167:     // for one register will be a simple "ref" for another register.
   168:     // Since we do not store the "Self" bit and register number, the meet
   169:     // operation will need to take it as a parameter.
   170:     //
   171:     // In practice there is a special case for values that are not associa-
   172:     // ted with any specific virtual register. An example would be a value
   173:     // corresponding to a bit of a physical register, or an intermediate
   174:     // value obtained in some computation (such as instruction evaluation).
   175:     // Such cases are identical to the usual Ref type, but the register
   176:     // number is 0. In such case the Pos field of the reference is ignored.
   177:     //
   178:     // What is worthy of notice is that in value V (that is a "ref"), as long
   179:     // as the RefI.Reg is not 0, it may actually be the same register as the
   180:     // one in which V will be contained.  If the RefI.Pos refers to the posi-
   181:     // tion of V, then V is assumed to be "bottom" (as a "ref to itself"),
   182:     // otherwise V is taken to be identical to the referenced bit of the
   183:     // same register.
   184:     // If RefI.Reg is 0, however, such a reference to the same register is
   185:     // not possible.  Any value V that is a "ref", and whose RefI.Reg is 0
   186:     // is treated as "bottom".
   187:   };
   188:   ValueType Type;
   189:   BitRef RefI;
   190: 
   191:   BitValue(ValueType T = Top) : Type(T) {}
   192:   BitValue(bool B) : Type(B ? One : Zero) {}
   193:   BitValue(unsigned Reg, uint16_t Pos) : Type(Ref), RefI(Reg, Pos) {}
   194: 
   195:   bool operator== (const BitValue &V) const {
   196:     if (Type != V.Type)
   197:       return false;
   198:     if (Type == Ref && !(RefI == V.RefI))
   199:       return false;
   200:     return true;
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares types such as BitTracker::BitValue, ValueType, which carry the state or API of this component. It declares or implements routines such as BitValue, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明了 BitTracker::BitValue, ValueType 等类型，用来承载该组件的状态或接口。 这里声明或实现了 BitValue 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   }
   202:   bool operator!= (const BitValue &V) const {
   203:     return !operator==(V);
   204:   }
   205: 
   206:   bool is(unsigned T) const {
   207:     assert(T == 0 || T == 1);
   208:     return T == 0 ? Type == Zero
   209:                   : (T == 1 ? Type == One : false);
   210:   }
   211: 
   212:   // The "meet" operation is the "." operation in a semilattice (L, ., T, B):
   213:   // (1)  x.x = x
   214:   // (2)  x.y = y.x
   215:   // (3)  x.(y.z) = (x.y).z
   216:   // (4)  x.T = x  (i.e. T = "top")
   217:   // (5)  x.B = B  (i.e. B = "bottom")
   218:   //
   219:   // This "meet" function will update the value of the "*this" object with
   220:   // the newly calculated one, and return "true" if the value of *this has
   221:   // changed, and "false" otherwise.
   222:   // To prove that it satisfies the conditions (1)-(5), it is sufficient
   223:   // to show that a relation
   224:   //   x <= y  <=>  x.y = x
   225:   // defines a partial order (i.e. that "meet" is same as "infimum").
   226:   bool meet(const BitValue &V, const BitRef &Self) {
   227:     // First, check the cases where there is nothing to be done.
   228:     if (Type == Ref && RefI == Self)    // Bottom.meet(V) = Bottom (i.e. This)
   229:       return false;
   230:     if (V.Type == Top)                  // This.meet(Top) = This
   231:       return false;
   232:     if (*this == V)                     // This.meet(This) = This
   233:       return false;
   234: 
   235:     // At this point, we know that the value of "this" will change.
   236:     // If it is Top, it will become the same as V, otherwise it will
   237:     // become "bottom" (i.e. Self).
   238:     if (Type == Top) {
   239:       Type = V.Type;
   240:       RefI = V.RefI;  // This may be irrelevant, but copy anyway.
   241:       return true;
   242:     }
   243:     // Become "bottom".
   244:     Type = Ref;
   245:     RefI = Self;
   246:     return true;
   247:   }
   248: 
   249:   // Create a reference to the bit value V.
   250:   static BitValue ref(const BitValue &V);
```
- EN: It declares or implements routines such as is, assert, semilattice, ref, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 is, assert, semilattice, ref 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 251-300 / 第 251-300 行

```cpp
   251:   // Create a "self".
   252:   static BitValue self(const BitRef &Self = BitRef());
   253: 
   254:   bool num() const {
   255:     return Type == Zero || Type == One;
   256:   }
   257: 
   258:   operator bool() const {
   259:     assert(Type == Zero || Type == One);
   260:     return Type == One;
   261:   }
   262: 
   263:   friend raw_ostream &operator<<(raw_ostream &OS, const BitValue &BV);
   264: };
   265: 
   266: // This operation must be idempotent, i.e. ref(ref(V)) == ref(V).
   267: inline BitTracker::BitValue
   268: BitTracker::BitValue::ref(const BitValue &V) {
   269:   if (V.Type != Ref)
   270:     return BitValue(V.Type);
   271:   if (V.RefI.Reg != 0)
   272:     return BitValue(V.RefI.Reg, V.RefI.Pos);
   273:   return self();
   274: }
   275: 
   276: inline BitTracker::BitValue
   277: BitTracker::BitValue::self(const BitRef &Self) {
   278:   return BitValue(Self.Reg, Self.Pos);
   279: }
   280: 
   281: // A sequence of bits starting from index B up to and including index E.
   282: // If E < B, the mask represents two sections: [0..E] and [B..W) where
   283: // W is the width of the register.
   284: struct BitTracker::BitMask {
   285:   BitMask() = default;
   286:   BitMask(uint16_t b, uint16_t e) : B(b), E(e) {}
   287: 
   288:   uint16_t first() const { return B; }
   289:   uint16_t last() const { return E; }
   290: 
   291: private:
   292:   uint16_t B = 0;
   293:   uint16_t E = 0;
   294: };
   295: 
   296: // Representation of a register: a list of BitValues.
   297: struct BitTracker::RegisterCell {
   298:   RegisterCell(uint16_t Width = DefaultBitN) : Bits(Width) {}
   299: 
   300:   uint16_t width() const {
```
- EN: It declares types such as BitTracker::BitMask, BitTracker::RegisterCell, which carry the state or API of this component. It declares or implements routines such as self, num, bool, assert, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明了 BitTracker::BitMask, BitTracker::RegisterCell 等类型，用来承载该组件的状态或接口。 这里声明或实现了 self, num, bool, assert, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 301-350 / 第 301-350 行

```cpp
   301:     return Bits.size();
   302:   }
   303: 
   304:   const BitValue &operator[](uint16_t BitN) const {
   305:     assert(BitN < Bits.size());
   306:     return Bits[BitN];
   307:   }
   308:   BitValue &operator[](uint16_t BitN) {
   309:     assert(BitN < Bits.size());
   310:     return Bits[BitN];
   311:   }
   312: 
   313:   bool meet(const RegisterCell &RC, Register SelfR);
   314:   RegisterCell &insert(const RegisterCell &RC, const BitMask &M);
   315:   RegisterCell extract(const BitMask &M) const;  // Returns a new cell.
   316:   RegisterCell &rol(uint16_t Sh);    // Rotate left.
   317:   RegisterCell &fill(uint16_t B, uint16_t E, const BitValue &V);
   318:   RegisterCell &cat(const RegisterCell &RC);  // Concatenate.
   319:   uint16_t cl(bool B) const;
   320:   uint16_t ct(bool B) const;
   321: 
   322:   bool operator== (const RegisterCell &RC) const;
   323:   bool operator!= (const RegisterCell &RC) const {
   324:     return !operator==(RC);
   325:   }
   326: 
   327:   // Replace the ref-to-reg-0 bit values with the given register.
   328:   RegisterCell &regify(unsigned R);
   329: 
   330:   // Generate a "ref" cell for the corresponding register. In the resulting
   331:   // cell each bit will be described as being the same as the corresponding
   332:   // bit in register Reg (i.e. the cell is "defined" by register Reg).
   333:   static RegisterCell self(unsigned Reg, uint16_t Width);
   334:   // Generate a "top" cell of given size.
   335:   static RegisterCell top(uint16_t Width);
   336:   // Generate a cell that is a "ref" to another cell.
   337:   static RegisterCell ref(const RegisterCell &C);
   338: 
   339: private:
   340:   // The DefaultBitN is here only to avoid frequent reallocation of the
   341:   // memory in the vector.
   342:   static const unsigned DefaultBitN = 32;
   343:   using BitValueList = SmallVector<BitValue, DefaultBitN>;
   344:   BitValueList Bits;
   345: 
   346:   friend raw_ostream &operator<<(raw_ostream &OS, const RegisterCell &RC);
   347: };
   348: 
   349: inline bool BitTracker::has(unsigned Reg) const {
   350:   return Map.find(Reg) != Map.end();
```
- EN: It declares or implements routines such as size, assert, meet, insert, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 size, assert, meet, insert, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 351-400 / 第 351-400 行

```cpp
   351: }
   352: 
   353: inline const BitTracker::RegisterCell&
   354: BitTracker::lookup(unsigned Reg) const {
   355:   CellMapType::const_iterator F = Map.find(Reg);
   356:   assert(F != Map.end());
   357:   return F->second;
   358: }
   359: 
   360: inline BitTracker::RegisterCell
   361: BitTracker::RegisterCell::self(unsigned Reg, uint16_t Width) {
   362:   RegisterCell RC(Width);
   363:   for (uint16_t i = 0; i < Width; ++i)
   364:     RC.Bits[i] = BitValue::self(BitRef(Reg, i));
   365:   return RC;
   366: }
   367: 
   368: inline BitTracker::RegisterCell
   369: BitTracker::RegisterCell::top(uint16_t Width) {
   370:   RegisterCell RC(Width);
   371:   for (uint16_t i = 0; i < Width; ++i)
   372:     RC.Bits[i] = BitValue(BitValue::Top);
   373:   return RC;
   374: }
   375: 
   376: inline BitTracker::RegisterCell
   377: BitTracker::RegisterCell::ref(const RegisterCell &C) {
   378:   uint16_t W = C.width();
   379:   RegisterCell RC(W);
   380:   for (unsigned i = 0; i < W; ++i)
   381:     RC[i] = BitValue::ref(C[i]);
   382:   return RC;
   383: }
   384: 
   385: // A class to evaluate target's instructions and update the cell maps.
   386: // This is used internally by the bit tracker.  A target that wants to
   387: // utilize this should implement the evaluation functions (noted below)
   388: // in a subclass of this class.
   389: struct BitTracker::MachineEvaluator {
   390:   MachineEvaluator(const TargetRegisterInfo &T, MachineRegisterInfo &M)
   391:       : TRI(T), MRI(M) {}
   392:   virtual ~MachineEvaluator() = default;
   393: 
   394:   uint16_t getRegBitWidth(const RegisterRef &RR) const;
   395: 
   396:   RegisterCell getCell(const RegisterRef &RR, const CellMapType &M) const;
   397:   void putCell(const RegisterRef &RR, RegisterCell RC, CellMapType &M) const;
   398: 
   399:   // A result of any operation should use refs to the source cells, not
   400:   // the cells directly. This function is a convenience wrapper to quickly
```
- EN: It declares types such as BitTracker::MachineEvaluator, which carry the state or API of this component. It declares or implements routines such as BitTracker::lookup, find, assert, BitTracker::RegisterCell::self, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明了 BitTracker::MachineEvaluator 等类型，用来承载该组件的状态或接口。 这里声明或实现了 BitTracker::lookup, find, assert, BitTracker::RegisterCell::self, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 401-450 / 第 401-450 行

```cpp
   401:   // generate a ref for a cell corresponding to a register reference.
   402:   RegisterCell getRef(const RegisterRef &RR, const CellMapType &M) const {
   403:     RegisterCell RC = getCell(RR, M);
   404:     return RegisterCell::ref(RC);
   405:   }
   406: 
   407:   // Helper functions.
   408:   // Check if a cell is an immediate value (i.e. all bits are either 0 or 1).
   409:   bool isInt(const RegisterCell &A) const;
   410:   // Convert cell to an immediate value.
   411:   uint64_t toInt(const RegisterCell &A) const;
   412: 
   413:   // Generate cell from an immediate value.
   414:   RegisterCell eIMM(int64_t V, uint16_t W) const;
   415:   RegisterCell eIMM(const ConstantInt *CI) const;
   416: 
   417:   // Arithmetic.
   418:   RegisterCell eADD(const RegisterCell &A1, const RegisterCell &A2) const;
   419:   RegisterCell eSUB(const RegisterCell &A1, const RegisterCell &A2) const;
   420:   RegisterCell eMLS(const RegisterCell &A1, const RegisterCell &A2) const;
   421:   RegisterCell eMLU(const RegisterCell &A1, const RegisterCell &A2) const;
   422: 
   423:   // Shifts.
   424:   RegisterCell eASL(const RegisterCell &A1, uint16_t Sh) const;
   425:   RegisterCell eLSR(const RegisterCell &A1, uint16_t Sh) const;
   426:   RegisterCell eASR(const RegisterCell &A1, uint16_t Sh) const;
   427: 
   428:   // Logical.
   429:   RegisterCell eAND(const RegisterCell &A1, const RegisterCell &A2) const;
   430:   RegisterCell eORL(const RegisterCell &A1, const RegisterCell &A2) const;
   431:   RegisterCell eXOR(const RegisterCell &A1, const RegisterCell &A2) const;
   432:   RegisterCell eNOT(const RegisterCell &A1) const;
   433: 
   434:   // Set bit, clear bit.
   435:   RegisterCell eSET(const RegisterCell &A1, uint16_t BitN) const;
   436:   RegisterCell eCLR(const RegisterCell &A1, uint16_t BitN) const;
   437: 
   438:   // Count leading/trailing bits (zeros/ones).
   439:   RegisterCell eCLB(const RegisterCell &A1, bool B, uint16_t W) const;
   440:   RegisterCell eCTB(const RegisterCell &A1, bool B, uint16_t W) const;
   441: 
   442:   // Sign/zero extension.
   443:   RegisterCell eSXT(const RegisterCell &A1, uint16_t FromN) const;
   444:   RegisterCell eZXT(const RegisterCell &A1, uint16_t FromN) const;
   445: 
   446:   // Extract/insert
   447:   // XTR R,b,e:  extract bits from A1 starting at bit b, ending at e-1.
   448:   // INS R,S,b:  take R and replace bits starting from b with S.
   449:   RegisterCell eXTR(const RegisterCell &A1, uint16_t B, uint16_t E) const;
   450:   RegisterCell eINS(const RegisterCell &A1, const RegisterCell &A2,
```
- EN: It declares or implements routines such as getRef, getCell, RegisterCell::ref, value, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getRef, getCell, RegisterCell::ref, value, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 451-493 / 第 451-493 行

```cpp
   451:                     uint16_t AtN) const;
   452: 
   453:   // User-provided functions for individual targets:
   454: 
   455:   // Return a sub-register mask that indicates which bits in Reg belong
   456:   // to the subregister Sub. These bits are assumed to be contiguous in
   457:   // the super-register, and have the same ordering in the sub-register
   458:   // as in the super-register. It is valid to call this function with
   459:   // Sub == 0, in this case, the function should return a mask that spans
   460:   // the entire register Reg (which is what the default implementation
   461:   // does).
   462:   virtual BitMask mask(Register Reg, unsigned Sub) const;
   463:   // Indicate whether a given register class should be tracked.
   464:   virtual bool track(const TargetRegisterClass *RC) const { return true; }
   465:   // Evaluate a non-branching machine instruction, given the cell map with
   466:   // the input values. Place the results in the Outputs map. Return "true"
   467:   // if evaluation succeeded, "false" otherwise.
   468:   virtual bool evaluate(const MachineInstr &MI, const CellMapType &Inputs,
   469:                         CellMapType &Outputs) const;
   470:   // Evaluate a branch, given the cell map with the input values. Fill out
   471:   // a list of all possible branch targets and indicate (through a flag)
   472:   // whether the branch could fall-through. Return "true" if this information
   473:   // has been successfully computed, "false" otherwise.
   474:   virtual bool evaluate(const MachineInstr &BI, const CellMapType &Inputs,
   475:                         BranchTargetList &Targets, bool &FallsThru) const = 0;
   476:   // Given a register class RC, return a register class that should be assumed
   477:   // when a register from class RC is used with a subregister of index Idx.
   478:   virtual const TargetRegisterClass&
   479:   composeWithSubRegIndex(const TargetRegisterClass &RC, unsigned Idx) const {
   480:     if (Idx == 0)
   481:       return RC;
   482:     llvm_unreachable("Unimplemented composeWithSubRegIndex");
   483:   }
   484:   // Return the size in bits of the physical register Reg.
   485:   virtual uint16_t getPhysRegBitWidth(MCRegister Reg) const;
   486: 
   487:   const TargetRegisterInfo &TRI;
   488:   MachineRegisterInfo &MRI;
   489: };
   490: 
   491: } // end namespace llvm
   492: 
   493: #endif // LLVM_LIB_TARGET_HEXAGON_BITTRACKER_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as Reg, track, evaluate, composeWithSubRegIndex, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 Reg, track, evaluate, composeWithSubRegIndex, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/ADT/DenseSet.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, map, queue, set`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
