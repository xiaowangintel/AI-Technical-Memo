# HexagonVectorCombine.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonVectorCombine.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon vector combine and canonicalization.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及 HVX/向量处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===-- HexagonVectorCombine.cpp ------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // HexagonVectorCombine is a utility class implementing a variety of functions
     9: // that assist in vector-based optimizations.
    10: //
    11: // AlignVectors: replace unaligned vector loads and stores with aligned ones.
    12: // HvxIdioms: recognize various opportunities to generate HVX intrinsic code.
    13: //===----------------------------------------------------------------------===//
    14: 
    15: #include "llvm/ADT/APInt.h"
    16: #include "llvm/ADT/ArrayRef.h"
    17: #include "llvm/ADT/DenseMap.h"
    18: #include "llvm/ADT/MapVector.h"
    19: #include "llvm/ADT/STLExtras.h"
    20: #include "llvm/ADT/SmallVector.h"
    21: #include "llvm/Analysis/AliasAnalysis.h"
    22: #include "llvm/Analysis/AssumeBundleQueries.h"
    23: #include "llvm/Analysis/AssumptionCache.h"
    24: #include "llvm/Analysis/InstSimplifyFolder.h"
    25: #include "llvm/Analysis/InstructionSimplify.h"
    26: #include "llvm/Analysis/ScalarEvolutionExpressions.h"
    27: #include "llvm/Analysis/TargetLibraryInfo.h"
    28: #include "llvm/Analysis/ValueTracking.h"
    29: #include "llvm/Analysis/VectorUtils.h"
    30: #include "llvm/CodeGen/TargetPassConfig.h"
    31: #include "llvm/CodeGen/ValueTypes.h"
    32: #include "llvm/IR/Dominators.h"
    33: #include "llvm/IR/IRBuilder.h"
    34: #include "llvm/IR/IntrinsicInst.h"
    35: #include "llvm/IR/Intrinsics.h"
    36: #include "llvm/IR/IntrinsicsHexagon.h"
    37: #include "llvm/IR/Metadata.h"
    38: #include "llvm/IR/PatternMatch.h"
    39: #include "llvm/InitializePasses.h"
    40: #include "llvm/Pass.h"
    41: #include "llvm/Support/CommandLine.h"
    42: #include "llvm/Support/KnownBits.h"
    43: #include "llvm/Support/MathExtras.h"
    44: #include "llvm/Support/raw_ostream.h"
    45: #include "llvm/Target/TargetMachine.h"
    46: #include "llvm/Transforms/Utils/Local.h"
    47: 
    48: #include "Hexagon.h"
    49: #include "HexagonSubtarget.h"
    50: #include "HexagonTargetMachine.h"
    51: 
    52: #include <algorithm>
    53: #include <deque>
    54: #include <map>
    55: #include <optional>
    56: #include <set>
    57: #include <utility>
    58: #include <vector>
    59: 
    60: #define DEBUG_TYPE "hexagon-vc"
    61: 
    62: // This is a const that represents default HVX VTCM page size.
    63: // It is boot time configurable, so we probably want an API to
    64: // read it, but for now assume 128KB
    65: #define DEFAULT_HVX_VTCM_PAGE_SIZE 131072
    66: 
    67: using namespace llvm;
    68: 
    69: namespace {
    70: cl::opt<bool> DumpModule("hvc-dump-module", cl::Hidden);
    71: cl::opt<bool> VAEnabled("hvc-va", cl::Hidden, cl::init(true)); // Align
    72: cl::opt<bool> VIEnabled("hvc-vi", cl::Hidden, cl::init(true)); // Idioms
    73: cl::opt<bool> VADoFullStores("hvc-va-full-stores", cl::Hidden);
    74: 
    75: cl::opt<unsigned> VAGroupCountLimit("hvc-va-group-count-limit", cl::Hidden,
    76:                                     cl::init(~0));
    77: cl::opt<unsigned> VAGroupSizeLimit("hvc-va-group-size-limit", cl::Hidden,
    78:                                    cl::init(~0));
    79: cl::opt<unsigned>
    80:     MinLoadGroupSizeForAlignment("hvc-ld-min-group-size-for-alignment",
    81:                                  cl::Hidden, cl::init(4));
    82: 
    83: class HexagonVectorCombine {
    84: public:
    85:   HexagonVectorCombine(Function &F_, AliasAnalysis &AA_, AssumptionCache &AC_,
    86:                        DominatorTree &DT_, ScalarEvolution &SE_,
    87:                        TargetLibraryInfo &TLI_, const TargetMachine &TM_)
    88:       : F(F_), DL(F.getDataLayout()), AA(AA_), AC(AC_), DT(DT_),
    89:         SE(SE_), TLI(TLI_),
    90:         HST(static_cast<const HexagonSubtarget &>(*TM_.getSubtargetImpl(F))) {}
    91: 
    92:   bool run();
    93: 
    94:   // Common integer type.
    95:   IntegerType *getIntTy(unsigned Width = 32) const;
    96:   // Byte type: either scalar (when Length = 0), or vector with given
    97:   // element count.
    98:   Type *getByteTy(int ElemCount = 0) const;
    99:   // Boolean type: either scalar (when Length = 0), or vector with given
   100:   // element count.
   101:   Type *getBoolTy(int ElemCount = 0) const;
   102:   // Create a ConstantInt of type returned by getIntTy with the value Val.
   103:   ConstantInt *getConstInt(int Val, unsigned Width = 32) const;
   104:   // Get the integer value of V, if it exists.
   105:   std::optional<APInt> getIntValue(const Value *Val) const;
   106:   // Is Val a constant 0, or a vector of 0s?
   107:   bool isZero(const Value *Val) const;
   108:   // Is Val an undef value?
   109:   bool isUndef(const Value *Val) const;
   110:   // Is Val a scalar (i1 true) or a vector of (i1 true)?
   111:   bool isTrue(const Value *Val) const;
   112:   // Is Val a scalar (i1 false) or a vector of (i1 false)?
   113:   bool isFalse(const Value *Val) const;
   114: 
   115:   // Get HVX vector type with the given element type.
   116:   VectorType *getHvxTy(Type *ElemTy, bool Pair = false) const;
   117: 
   118:   enum SizeKind {
   119:     Store, // Store size
   120:     Alloc, // Alloc size
   121:   };
   122:   int getSizeOf(const Value *Val, SizeKind Kind = Store) const;
   123:   int getSizeOf(const Type *Ty, SizeKind Kind = Store) const;
   124:   int getTypeAlignment(Type *Ty) const;
   125:   size_t length(Value *Val) const;
   126:   size_t length(Type *Ty) const;
   127: 
   128:   Value *simplify(Value *Val) const;
   129: 
   130:   Value *insertb(IRBuilderBase &Builder, Value *Dest, Value *Src, int Start,
   131:                  int Length, int Where) const;
   132:   Value *vlalignb(IRBuilderBase &Builder, Value *Lo, Value *Hi,
   133:                   Value *Amt) const;
   134:   Value *vralignb(IRBuilderBase &Builder, Value *Lo, Value *Hi,
   135:                   Value *Amt) const;
   136:   Value *concat(IRBuilderBase &Builder, ArrayRef<Value *> Vecs) const;
   137:   Value *vresize(IRBuilderBase &Builder, Value *Val, int NewSize,
   138:                  Value *Pad) const;
   139:   Value *rescale(IRBuilderBase &Builder, Value *Mask, Type *FromTy,
   140:                  Type *ToTy) const;
   141:   Value *vlsb(IRBuilderBase &Builder, Value *Val) const;
   142:   Value *vbytes(IRBuilderBase &Builder, Value *Val) const;
   143:   Value *subvector(IRBuilderBase &Builder, Value *Val, unsigned Start,
   144:                    unsigned Length) const;
   145:   Value *sublo(IRBuilderBase &Builder, Value *Val) const;
   146:   Value *subhi(IRBuilderBase &Builder, Value *Val) const;
   147:   Value *vdeal(IRBuilderBase &Builder, Value *Val0, Value *Val1) const;
   148:   Value *vshuff(IRBuilderBase &Builder, Value *Val0, Value *Val1) const;
   149: 
   150:   Value *createHvxIntrinsic(IRBuilderBase &Builder, Intrinsic::ID IntID,
   151:                             Type *RetTy, ArrayRef<Value *> Args,
   152:                             ArrayRef<Type *> ArgTys = {},
   153:                             ArrayRef<Value *> MDSources = {}) const;
   154:   SmallVector<Value *> splitVectorElements(IRBuilderBase &Builder, Value *Vec,
   155:                                            unsigned ToWidth) const;
   156:   Value *joinVectorElements(IRBuilderBase &Builder, ArrayRef<Value *> Values,
   157:                             VectorType *ToType) const;
   158: 
   159:   std::optional<int> calculatePointerDifference(Value *Ptr0, Value *Ptr1) const;
   160: 
   161:   unsigned getNumSignificantBits(const Value *V,
   162:                                  const Instruction *CtxI = nullptr) const;
   163:   KnownBits getKnownBits(const Value *V,
   164:                          const Instruction *CtxI = nullptr) const;
   165: 
   166:   bool isSafeToClone(const Instruction &In) const;
   167: 
   168:   template <typename T = std::vector<Instruction *>>
   169:   bool isSafeToMoveBeforeInBB(const Instruction &In,
   170:                               BasicBlock::const_iterator To,
   171:                               const T &IgnoreInsts = {}) const;
   172: 
   173:   // This function is only used for assertions at the moment.
   174:   [[maybe_unused]] bool isByteVecTy(Type *Ty) const;
   175: 
   176:   Function &F;
   177:   const DataLayout &DL;
   178:   AliasAnalysis &AA;
   179:   AssumptionCache &AC;
   180:   DominatorTree &DT;
   181:   ScalarEvolution &SE;
   182:   TargetLibraryInfo &TLI;
   183:   const HexagonSubtarget &HST;
   184: 
   185: private:
   186:   Value *getElementRange(IRBuilderBase &Builder, Value *Lo, Value *Hi,
   187:                          int Start, int Length) const;
   188: };
   189: 
   190: class AlignVectors {
   191:   // This code tries to replace unaligned vector loads/stores with aligned
   192:   // ones.
   193:   // Consider unaligned load:
   194:   //   %v = original_load %some_addr, align <bad>
   195:   //   %user = %v
   196:   // It will generate
   197:   //      = load ..., align <good>
   198:   //      = load ..., align <good>
   199:   //      = valign
   200:   //      etc.
```
- EN: It imports headers such as llvm/ADT/APInt.h, llvm/ADT/ArrayRef.h, llvm/ADT/DenseMap.h, llvm/ADT/MapVector.h, ... (42 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonVectorCombine, SizeKind, AlignVectors, which carry the state or API of this component.
- CN: 这里引入了 llvm/ADT/APInt.h, llvm/ADT/ArrayRef.h, llvm/ADT/DenseMap.h, llvm/ADT/MapVector.h, ... (42 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonVectorCombine, SizeKind, AlignVectors 等类型，用来承载该组件的状态或接口。

### Lines 201-400 / 第 201-400 行

```cpp
   201:   //   %synthesize = combine/shuffle the loaded data so that it looks
   202:   //                 exactly like what "original_load" has loaded.
   203:   //   %user = %synthesize
   204:   // Similarly for stores.
   205: public:
   206:   AlignVectors(const HexagonVectorCombine &HVC_) : HVC(HVC_) {}
   207: 
   208:   bool run();
   209: 
   210: private:
   211:   using InstList = std::vector<Instruction *>;
   212:   using InstMap = DenseMap<Instruction *, Instruction *>;
   213: 
   214:   struct AddrInfo {
   215:     AddrInfo(const AddrInfo &) = default;
   216:     AddrInfo &operator=(const AddrInfo &) = default;
   217:     AddrInfo(const HexagonVectorCombine &HVC, Instruction *I, Value *A, Type *T,
   218:              Align H)
   219:         : Inst(I), Addr(A), ValTy(T), HaveAlign(H),
   220:           NeedAlign(HVC.getTypeAlignment(ValTy)) {}
   221: 
   222:     // XXX: add Size member?
   223:     Instruction *Inst;
   224:     Value *Addr;
   225:     Type *ValTy;
   226:     Align HaveAlign;
   227:     Align NeedAlign;
   228:     int Offset = 0; // Offset (in bytes) from the first member of the
   229:                     // containing AddrList.
   230:   };
   231:   using AddrList = std::vector<AddrInfo>;
   232: 
   233:   struct InstrLess {
   234:     bool operator()(const Instruction *A, const Instruction *B) const {
   235:       return A->comesBefore(B);
   236:     }
   237:   };
   238:   using DepList = std::set<Instruction *, InstrLess>;
   239: 
   240:   struct MoveGroup {
   241:     MoveGroup(const AddrInfo &AI, Instruction *B, bool Hvx, bool Load)
   242:         : Base(B), Main{AI.Inst}, Clones{}, IsHvx(Hvx), IsLoad(Load) {}
   243:     MoveGroup() = default;
   244:     Instruction *Base; // Base instruction of the parent address group.
   245:     InstList Main;     // Main group of instructions.
   246:     InstList Deps;     // List of dependencies.
   247:     InstMap Clones;    // Map from original Deps to cloned ones.
   248:     bool IsHvx;        // Is this group of HVX instructions?
   249:     bool IsLoad;       // Is this a load group?
   250:   };
   251:   using MoveList = std::vector<MoveGroup>;
   252: 
   253:   struct ByteSpan {
   254:     // A representation of "interesting" bytes within a given span of memory.
   255:     // These bytes are those that are loaded or stored, and they don't have
   256:     // to cover the entire span of memory.
   257:     //
   258:     // The representation works by picking a contiguous sequence of bytes
   259:     // from somewhere within a llvm::Value, and placing it at a given offset
   260:     // within the span.
   261:     //
   262:     // The sequence of bytes from llvm:Value is represented by Segment.
   263:     // Block is Segment, plus where it goes in the span.
   264:     //
   265:     // An important feature of ByteSpan is being able to make a "section",
   266:     // i.e. creating another ByteSpan corresponding to a range of offsets
   267:     // relative to the source span.
   268: 
   269:     struct Segment {
   270:       // Segment of a Value: 'Len' bytes starting at byte 'Begin'.
   271:       Segment(Value *Val, int Begin, int Len)
   272:           : Val(Val), Start(Begin), Size(Len) {}
   273:       Segment(const Segment &Seg) = default;
   274:       Segment &operator=(const Segment &Seg) = default;
   275:       Value *Val; // Value representable as a sequence of bytes.
   276:       int Start;  // First byte of the value that belongs to the segment.
   277:       int Size;   // Number of bytes in the segment.
   278:     };
   279: 
   280:     struct Block {
   281:       Block(Value *Val, int Len, int Pos) : Seg(Val, 0, Len), Pos(Pos) {}
   282:       Block(Value *Val, int Off, int Len, int Pos)
   283:           : Seg(Val, Off, Len), Pos(Pos) {}
   284:       Block(const Block &Blk) = default;
   285:       Block &operator=(const Block &Blk) = default;
   286:       Segment Seg; // Value segment.
   287:       int Pos;     // Position (offset) of the block in the span.
   288:     };
   289: 
   290:     int extent() const;
   291:     ByteSpan section(int Start, int Length) const;
   292:     ByteSpan &shift(int Offset);
   293:     SmallVector<Value *, 8> values() const;
   294: 
   295:     int size() const { return Blocks.size(); }
   296:     Block &operator[](int i) { return Blocks[i]; }
   297:     const Block &operator[](int i) const { return Blocks[i]; }
   298: 
   299:     std::vector<Block> Blocks;
   300: 
   301:     using iterator = decltype(Blocks)::iterator;
   302:     iterator begin() { return Blocks.begin(); }
   303:     iterator end() { return Blocks.end(); }
   304:     using const_iterator = decltype(Blocks)::const_iterator;
   305:     const_iterator begin() const { return Blocks.begin(); }
   306:     const_iterator end() const { return Blocks.end(); }
   307:   };
   308: 
   309:   std::optional<AddrInfo> getAddrInfo(Instruction &In) const;
   310:   bool isHvx(const AddrInfo &AI) const;
   311:   // This function is only used for assertions at the moment.
   312:   [[maybe_unused]] bool isSectorTy(Type *Ty) const;
   313: 
   314:   Value *getPayload(Value *Val) const;
   315:   Value *getMask(Value *Val) const;
   316:   Value *getPassThrough(Value *Val) const;
   317: 
   318:   Value *createAdjustedPointer(IRBuilderBase &Builder, Value *Ptr, Type *ValTy,
   319:                                int Adjust,
   320:                                const InstMap &CloneMap = InstMap()) const;
   321:   Value *createAlignedPointer(IRBuilderBase &Builder, Value *Ptr, Type *ValTy,
   322:                               int Alignment,
   323:                               const InstMap &CloneMap = InstMap()) const;
   324: 
   325:   Value *createLoad(IRBuilderBase &Builder, Type *ValTy, Value *Ptr,
   326:                     Value *Predicate, int Alignment, Value *Mask,
   327:                     Value *PassThru, ArrayRef<Value *> MDSources = {}) const;
   328:   Value *createSimpleLoad(IRBuilderBase &Builder, Type *ValTy, Value *Ptr,
   329:                           int Alignment,
   330:                           ArrayRef<Value *> MDSources = {}) const;
   331: 
   332:   Value *createStore(IRBuilderBase &Builder, Value *Val, Value *Ptr,
   333:                      Value *Predicate, int Alignment, Value *Mask,
   334:                      ArrayRef<Value *> MDSources = {}) const;
   335:   Value *createSimpleStore(IRBuilderBase &Builder, Value *Val, Value *Ptr,
   336:                            int Alignment,
   337:                            ArrayRef<Value *> MDSources = {}) const;
   338: 
   339:   Value *createPredicatedLoad(IRBuilderBase &Builder, Type *ValTy, Value *Ptr,
   340:                               Value *Predicate, int Alignment,
   341:                               ArrayRef<Value *> MDSources = {}) const;
   342:   Value *createPredicatedStore(IRBuilderBase &Builder, Value *Val, Value *Ptr,
   343:                                Value *Predicate, int Alignment,
   344:                                ArrayRef<Value *> MDSources = {}) const;
   345: 
   346:   DepList getUpwardDeps(Instruction *In, Instruction *Base) const;
   347:   bool createAddressGroups();
   348:   MoveList createLoadGroups(const AddrList &Group) const;
   349:   MoveList createStoreGroups(const AddrList &Group) const;
   350:   bool moveTogether(MoveGroup &Move) const;
   351:   template <typename T>
   352:   InstMap cloneBefore(BasicBlock::iterator To, T &&Insts) const;
   353: 
   354:   void realignLoadGroup(IRBuilderBase &Builder, const ByteSpan &VSpan,
   355:                         int ScLen, Value *AlignVal, Value *AlignAddr) const;
   356:   void realignStoreGroup(IRBuilderBase &Builder, const ByteSpan &VSpan,
   357:                          int ScLen, Value *AlignVal, Value *AlignAddr) const;
   358:   bool realignGroup(const MoveGroup &Move);
   359:   Value *makeTestIfUnaligned(IRBuilderBase &Builder, Value *AlignVal,
   360:                              int Alignment) const;
   361: 
   362:   using AddrGroupMap = MapVector<Instruction *, AddrList>;
   363:   AddrGroupMap AddrGroups;
   364: 
   365:   friend raw_ostream &operator<<(raw_ostream &OS, const AddrList &L);
   366:   friend raw_ostream &operator<<(raw_ostream &OS, const AddrInfo &AI);
   367:   friend raw_ostream &operator<<(raw_ostream &OS, const MoveGroup &MG);
   368:   friend raw_ostream &operator<<(raw_ostream &OS, const MoveList &L);
   369:   friend raw_ostream &operator<<(raw_ostream &OS, const ByteSpan::Block &B);
   370:   friend raw_ostream &operator<<(raw_ostream &OS, const ByteSpan &BS);
   371:   friend raw_ostream &operator<<(raw_ostream &OS, const AddrGroupMap &AG);
   372:   friend raw_ostream &operator<<(raw_ostream &OS, const AddrList &L);
   373:   friend raw_ostream &operator<<(raw_ostream &OS, const AddrInfo &AI);
   374:   friend raw_ostream &operator<<(raw_ostream &OS, const MoveGroup &MG);
   375:   friend raw_ostream &operator<<(raw_ostream &OS, const MoveList &L);
   376:   friend raw_ostream &operator<<(raw_ostream &OS, const ByteSpan::Block &B);
   377:   friend raw_ostream &operator<<(raw_ostream &OS, const ByteSpan &BS);
   378:   friend raw_ostream &operator<<(raw_ostream &OS, const AddrGroupMap &AG);
   379: 
   380:   const HexagonVectorCombine &HVC;
   381: };
   382: 
   383: [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
   384:                                          const AlignVectors::AddrGroupMap &AG) {
   385:   OS << "Printing AddrGroups:"
   386:      << "\n";
   387:   for (auto &It : AG) {
   388:     OS << "\n\tInstruction: ";
   389:     It.first->dump();
   390:     OS << "\n\tAddrInfo: ";
   391:     for (auto &AI : It.second)
   392:       OS << AI << "\n";
   393:   }
   394:   return OS;
   395: }
   396: 
   397: [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
   398:                                          const AlignVectors::AddrList &AL) {
   399:   OS << "\n *** Addr List: ***\n";
   400:   for (auto &AG : AL) {
```
- EN: It declares types such as AddrInfo, InstrLess, MoveGroup, ByteSpan, ... (6 total), which carry the state or API of this component. It declares or implements routines such as AlignVectors, run, AddrInfo, comesBefore, ... (34 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVectorCombine, showing how the code connects to sibling backend components.
- CN: 这里声明了 AddrInfo, InstrLess, MoveGroup, ByteSpan, ... (6 total) 等类型，用来承载该组件的状态或接口。 这里声明或实现了 AlignVectors, run, AddrInfo, comesBefore, ... (34 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVectorCombine，说明了它与同级后端组件的连接关系。

### Lines 401-600 / 第 401-600 行

```cpp
   401:     OS << "\n *** Addr Group: ***\n";
   402:     OS << AG;
   403:     OS << "\n";
   404:   }
   405:   return OS;
   406: }
   407: 
   408: [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
   409:                                          const AlignVectors::AddrInfo &AI) {
   410:   OS << "Inst: " << AI.Inst << "  " << *AI.Inst << '\n';
   411:   OS << "Addr: " << *AI.Addr << '\n';
   412:   OS << "Type: " << *AI.ValTy << '\n';
   413:   OS << "HaveAlign: " << AI.HaveAlign.value() << '\n';
   414:   OS << "NeedAlign: " << AI.NeedAlign.value() << '\n';
   415:   OS << "Offset: " << AI.Offset;
   416:   return OS;
   417: }
   418: 
   419: [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
   420:                                          const AlignVectors::MoveList &ML) {
   421:   OS << "\n *** Move List: ***\n";
   422:   for (auto &MG : ML) {
   423:     OS << "\n *** Move Group: ***\n";
   424:     OS << MG;
   425:     OS << "\n";
   426:   }
   427:   return OS;
   428: }
   429: 
   430: [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
   431:                                          const AlignVectors::MoveGroup &MG) {
   432:   OS << "IsLoad:" << (MG.IsLoad ? "yes" : "no");
   433:   OS << ", IsHvx:" << (MG.IsHvx ? "yes" : "no") << '\n';
   434:   OS << "Main\n";
   435:   for (Instruction *I : MG.Main)
   436:     OS << "  " << *I << '\n';
   437:   OS << "Deps\n";
   438:   for (Instruction *I : MG.Deps)
   439:     OS << "  " << *I << '\n';
   440:   OS << "Clones\n";
   441:   for (auto [K, V] : MG.Clones) {
   442:     OS << "    ";
   443:     K->printAsOperand(OS, false);
   444:     OS << "\t-> " << *V << '\n';
   445:   }
   446:   return OS;
   447: }
   448: 
   449: [[maybe_unused]] raw_ostream &
   450: operator<<(raw_ostream &OS, const AlignVectors::ByteSpan::Block &B) {
   451:   OS << "  @" << B.Pos << " [" << B.Seg.Start << ',' << B.Seg.Size << "] ";
   452:   if (B.Seg.Val == reinterpret_cast<const Value *>(&B)) {
   453:     OS << "(self:" << B.Seg.Val << ')';
   454:   } else if (B.Seg.Val != nullptr) {
   455:     OS << *B.Seg.Val;
   456:   } else {
   457:     OS << "(null)";
   458:   }
   459:   return OS;
   460: }
   461: 
   462: [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
   463:                                          const AlignVectors::ByteSpan &BS) {
   464:   OS << "ByteSpan[size=" << BS.size() << ", extent=" << BS.extent() << '\n';
   465:   for (const AlignVectors::ByteSpan::Block &B : BS)
   466:     OS << B << '\n';
   467:   OS << ']';
   468:   return OS;
   469: }
   470: 
   471: class HvxIdioms {
   472: public:
   473:   enum DstQualifier {
   474:     Undefined = 0,
   475:     Arithmetic,
   476:     LdSt,
   477:     LLVM_Gather,
   478:     LLVM_Scatter,
   479:     HEX_Gather_Scatter,
   480:     HEX_Gather,
   481:     HEX_Scatter,
   482:     Call
   483:   };
   484: 
   485:   HvxIdioms(const HexagonVectorCombine &HVC_) : HVC(HVC_) {
   486:     auto *Int32Ty = HVC.getIntTy(32);
   487:     HvxI32Ty = HVC.getHvxTy(Int32Ty, /*Pair=*/false);
   488:     HvxP32Ty = HVC.getHvxTy(Int32Ty, /*Pair=*/true);
   489:   }
   490: 
   491:   bool run();
   492: 
   493: private:
   494:   enum Signedness { Positive, Signed, Unsigned };
   495: 
   496:   // Value + sign
   497:   // This is to keep track of whether the value should be treated as signed
   498:   // or unsigned, or is known to be positive.
   499:   struct SValue {
   500:     Value *Val;
   501:     Signedness Sgn;
   502:   };
   503: 
   504:   struct FxpOp {
   505:     unsigned Opcode;
   506:     unsigned Frac; // Number of fraction bits
   507:     SValue X, Y;
   508:     // If present, add 1 << RoundAt before shift:
   509:     std::optional<unsigned> RoundAt;
   510:     VectorType *ResTy;
   511:   };
   512: 
   513:   auto getNumSignificantBits(Value *V, Instruction *In) const
   514:       -> std::pair<unsigned, Signedness>;
   515:   auto canonSgn(SValue X, SValue Y) const -> std::pair<SValue, SValue>;
   516: 
   517:   auto matchFxpMul(Instruction &In) const -> std::optional<FxpOp>;
   518:   auto processFxpMul(Instruction &In, const FxpOp &Op) const -> Value *;
   519: 
   520:   auto processFxpMulChopped(IRBuilderBase &Builder, Instruction &In,
   521:                             const FxpOp &Op) const -> Value *;
   522:   auto createMulQ15(IRBuilderBase &Builder, SValue X, SValue Y,
   523:                     bool Rounding) const -> Value *;
   524:   auto createMulQ31(IRBuilderBase &Builder, SValue X, SValue Y,
   525:                     bool Rounding) const -> Value *;
   526:   // Return {Result, Carry}, where Carry is a vector predicate.
   527:   auto createAddCarry(IRBuilderBase &Builder, Value *X, Value *Y,
   528:                       Value *CarryIn = nullptr) const
   529:       -> std::pair<Value *, Value *>;
   530:   auto createMul16(IRBuilderBase &Builder, SValue X, SValue Y) const -> Value *;
   531:   auto createMulH16(IRBuilderBase &Builder, SValue X, SValue Y) const
   532:       -> Value *;
   533:   auto createMul32(IRBuilderBase &Builder, SValue X, SValue Y) const
   534:       -> std::pair<Value *, Value *>;
   535:   auto createAddLong(IRBuilderBase &Builder, ArrayRef<Value *> WordX,
   536:                      ArrayRef<Value *> WordY) const -> SmallVector<Value *>;
   537:   auto createMulLong(IRBuilderBase &Builder, ArrayRef<Value *> WordX,
   538:                      Signedness SgnX, ArrayRef<Value *> WordY,
   539:                      Signedness SgnY) const -> SmallVector<Value *>;
   540: 
   541:   bool matchMLoad(Instruction &In) const;
   542:   bool matchMStore(Instruction &In) const;
   543:   Value *processMLoad(Instruction &In) const;
   544:   Value *processMStore(Instruction &In) const;
   545:   std::optional<uint64_t> getAlignment(Instruction &In, Value *ptr) const;
   546:   std::optional<uint64_t>
   547:   getAlignmentImpl(Instruction &In, Value *ptr,
   548:                    SmallPtrSet<Value *, 16> &Visited) const;
   549:   std::optional<uint64_t> getPHIBaseMinAlignment(Instruction &In,
   550:                                                  PHINode *PN) const;
   551: 
   552:   // Vector manipulations for Ripple
   553:   bool matchScatter(Instruction &In) const;
   554:   bool matchGather(Instruction &In) const;
   555:   Value *processVScatter(Instruction &In) const;
   556:   Value *processVGather(Instruction &In) const;
   557: 
   558:   VectorType *HvxI32Ty;
   559:   VectorType *HvxP32Ty;
   560:   const HexagonVectorCombine &HVC;
   561: 
   562:   friend raw_ostream &operator<<(raw_ostream &, const FxpOp &);
   563: };
   564: 
   565: [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
   566:                                          const HvxIdioms::FxpOp &Op) {
   567:   static const char *SgnNames[] = {"Positive", "Signed", "Unsigned"};
   568:   OS << Instruction::getOpcodeName(Op.Opcode) << '.' << Op.Frac;
   569:   if (Op.RoundAt.has_value()) {
   570:     if (Op.Frac != 0 && *Op.RoundAt == Op.Frac - 1) {
   571:       OS << ":rnd";
   572:     } else {
   573:       OS << " + 1<<" << *Op.RoundAt;
   574:     }
   575:   }
   576:   OS << "\n  X:(" << SgnNames[Op.X.Sgn] << ") " << *Op.X.Val << "\n"
   577:      << "  Y:(" << SgnNames[Op.Y.Sgn] << ") " << *Op.Y.Val;
   578:   return OS;
   579: }
   580: 
   581: } // namespace
   582: 
   583: namespace {
   584: 
   585: template <typename T> T *getIfUnordered(T *MaybeT) {
   586:   return MaybeT && MaybeT->isUnordered() ? MaybeT : nullptr;
   587: }
   588: template <typename T> T *isCandidate(Instruction *In) {
   589:   return dyn_cast<T>(In);
   590: }
   591: template <> LoadInst *isCandidate<LoadInst>(Instruction *In) {
   592:   return getIfUnordered(dyn_cast<LoadInst>(In));
   593: }
   594: template <> StoreInst *isCandidate<StoreInst>(Instruction *In) {
   595:   return getIfUnordered(dyn_cast<StoreInst>(In));
   596: }
   597: 
   598: // Forward other erase_ifs to the LLVM implementations.
   599: template <typename Pred, typename T> void erase_if(T &&container, Pred p) {
   600:   llvm::erase_if(std::forward<T>(container), p);
```
- EN: It opens namespaces (namespace) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HvxIdioms, DstQualifier, Signedness, SValue, ... (5 total), which carry the state or API of this component. It defines declarative TableGen records like HvxIdioms; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as printAsOperand, HvxIdioms, getIntTy, getHvxTy, ... (23 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（namespace），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HvxIdioms, DstQualifier, Signedness, SValue, ... (5 total) 等类型，用来承载该组件的状态或接口。 这里定义了 HvxIdioms 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 printAsOperand, HvxIdioms, getIntTy, getHvxTy, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 601-800 / 第 601-800 行

```cpp
   601: }
   602: 
   603: } // namespace
   604: 
   605: // --- Begin AlignVectors
   606: 
   607: // For brevity, only consider loads. We identify a group of loads where we
   608: // know the relative differences between their addresses, so we know how they
   609: // are laid out in memory (relative to one another). These loads can overlap,
   610: // can be shorter or longer than the desired vector length.
   611: // Ultimately we want to generate a sequence of aligned loads that will load
   612: // every byte that the original loads loaded, and have the program use these
   613: // loaded values instead of the original loads.
   614: // We consider the contiguous memory area spanned by all these loads.
   615: //
   616: // Let's say that a single aligned vector load can load 16 bytes at a time.
   617: // If the program wanted to use a byte at offset 13 from the beginning of the
   618: // original span, it will be a byte at offset 13+x in the aligned data for
   619: // some x>=0. This may happen to be in the first aligned load, or in the load
   620: // following it. Since we generally don't know what the that alignment value
   621: // is at compile time, we proactively do valigns on the aligned loads, so that
   622: // byte that was at offset 13 is still at offset 13 after the valigns.
   623: //
   624: // This will be the starting point for making the rest of the program use the
   625: // data loaded by the new loads.
   626: // For each original load, and its users:
   627: //   %v = load ...
   628: //   ... = %v
   629: //   ... = %v
   630: // we create
   631: //   %new_v = extract/combine/shuffle data from loaded/valigned vectors so
   632: //            it contains the same value as %v did before
   633: // then replace all users of %v with %new_v.
   634: //   ... = %new_v
   635: //   ... = %new_v
   636: 
   637: auto AlignVectors::ByteSpan::extent() const -> int {
   638:   if (size() == 0)
   639:     return 0;
   640:   int Min = Blocks[0].Pos;
   641:   int Max = Blocks[0].Pos + Blocks[0].Seg.Size;
   642:   for (int i = 1, e = size(); i != e; ++i) {
   643:     Min = std::min(Min, Blocks[i].Pos);
   644:     Max = std::max(Max, Blocks[i].Pos + Blocks[i].Seg.Size);
   645:   }
   646:   return Max - Min;
   647: }
   648: 
   649: auto AlignVectors::ByteSpan::section(int Start, int Length) const -> ByteSpan {
   650:   ByteSpan Section;
   651:   for (const ByteSpan::Block &B : Blocks) {
   652:     int L = std::max(B.Pos, Start);                       // Left end.
   653:     int R = std::min(B.Pos + B.Seg.Size, Start + Length); // Right end+1.
   654:     if (L < R) {
   655:       // How much to chop off the beginning of the segment:
   656:       int Off = L > B.Pos ? L - B.Pos : 0;
   657:       Section.Blocks.emplace_back(B.Seg.Val, B.Seg.Start + Off, R - L, L);
   658:     }
   659:   }
   660:   return Section;
   661: }
   662: 
   663: auto AlignVectors::ByteSpan::shift(int Offset) -> ByteSpan & {
   664:   for (Block &B : Blocks)
   665:     B.Pos += Offset;
   666:   return *this;
   667: }
   668: 
   669: auto AlignVectors::ByteSpan::values() const -> SmallVector<Value *, 8> {
   670:   SmallVector<Value *, 8> Values(Blocks.size());
   671:   for (int i = 0, e = Blocks.size(); i != e; ++i)
   672:     Values[i] = Blocks[i].Seg.Val;
   673:   return Values;
   674: }
   675: 
   676: // Turn a requested integer alignment into the effective Align to use.
   677: // If Requested == 0 -> use ABI alignment of the value type (old semantics).
   678: // 0 means "ABI alignment" in old IR.
   679: static Align effectiveAlignForValueTy(const DataLayout &DL, Type *ValTy,
   680:                                       int Requested) {
   681:   if (Requested > 0)
   682:     return Align(static_cast<uint64_t>(Requested));
   683:   return Align(DL.getABITypeAlign(ValTy).value());
   684: }
   685: 
   686: auto AlignVectors::getAddrInfo(Instruction &In) const
   687:     -> std::optional<AddrInfo> {
   688:   if (auto *L = isCandidate<LoadInst>(&In))
   689:     return AddrInfo(HVC, L, L->getPointerOperand(), L->getType(),
   690:                     L->getAlign());
   691:   if (auto *S = isCandidate<StoreInst>(&In))
   692:     return AddrInfo(HVC, S, S->getPointerOperand(),
   693:                     S->getValueOperand()->getType(), S->getAlign());
   694:   if (auto *II = isCandidate<IntrinsicInst>(&In)) {
   695:     Intrinsic::ID ID = II->getIntrinsicID();
   696:     switch (ID) {
   697:     case Intrinsic::masked_load:
   698:       return AddrInfo(HVC, II, II->getArgOperand(0), II->getType(),
   699:                       II->getParamAlign(0).valueOrOne());
   700:     case Intrinsic::masked_store:
   701:       return AddrInfo(HVC, II, II->getArgOperand(1),
   702:                       II->getArgOperand(0)->getType(),
   703:                       II->getParamAlign(1).valueOrOne());
   704:     }
   705:   }
   706:   return std::nullopt;
   707: }
   708: 
   709: auto AlignVectors::isHvx(const AddrInfo &AI) const -> bool {
   710:   return HVC.HST.isTypeForHVX(AI.ValTy);
   711: }
   712: 
   713: auto AlignVectors::getPayload(Value *Val) const -> Value * {
   714:   if (auto *In = dyn_cast<Instruction>(Val)) {
   715:     Intrinsic::ID ID = 0;
   716:     if (auto *II = dyn_cast<IntrinsicInst>(In))
   717:       ID = II->getIntrinsicID();
   718:     if (isa<StoreInst>(In) || ID == Intrinsic::masked_store)
   719:       return In->getOperand(0);
   720:   }
   721:   return Val;
   722: }
   723: 
   724: auto AlignVectors::getMask(Value *Val) const -> Value * {
   725:   if (auto *II = dyn_cast<IntrinsicInst>(Val)) {
   726:     switch (II->getIntrinsicID()) {
   727:     case Intrinsic::masked_load:
   728:       return II->getArgOperand(1);
   729:     case Intrinsic::masked_store:
   730:       return II->getArgOperand(2);
   731:     }
   732:   }
   733: 
   734:   Type *ValTy = getPayload(Val)->getType();
   735:   if (auto *VecTy = dyn_cast<VectorType>(ValTy))
   736:     return Constant::getAllOnesValue(HVC.getBoolTy(HVC.length(VecTy)));
   737:   return Constant::getAllOnesValue(HVC.getBoolTy());
   738: }
   739: 
   740: auto AlignVectors::getPassThrough(Value *Val) const -> Value * {
   741:   if (auto *II = dyn_cast<IntrinsicInst>(Val)) {
   742:     if (II->getIntrinsicID() == Intrinsic::masked_load)
   743:       return II->getArgOperand(2);
   744:   }
   745:   return UndefValue::get(getPayload(Val)->getType());
   746: }
   747: 
   748: auto AlignVectors::createAdjustedPointer(IRBuilderBase &Builder, Value *Ptr,
   749:                                          Type *ValTy, int Adjust,
   750:                                          const InstMap &CloneMap) const
   751:     -> Value * {
   752:   if (auto *I = dyn_cast<Instruction>(Ptr))
   753:     if (Instruction *New = CloneMap.lookup(I))
   754:       Ptr = New;
   755:   return Builder.CreatePtrAdd(Ptr, HVC.getConstInt(Adjust), "gep");
   756: }
   757: 
   758: auto AlignVectors::createAlignedPointer(IRBuilderBase &Builder, Value *Ptr,
   759:                                         Type *ValTy, int Alignment,
   760:                                         const InstMap &CloneMap) const
   761:     -> Value * {
   762:   auto remap = [&](Value *V) -> Value * {
   763:     if (auto *I = dyn_cast<Instruction>(V)) {
   764:       for (auto [Old, New] : CloneMap)
   765:         I->replaceUsesOfWith(Old, New);
   766:       return I;
   767:     }
   768:     return V;
   769:   };
   770:   Value *AsInt = Builder.CreatePtrToInt(Ptr, HVC.getIntTy(), "pti");
   771:   Value *Mask = HVC.getConstInt(-Alignment);
   772:   Value *And = Builder.CreateAnd(remap(AsInt), Mask, "and");
   773:   return Builder.CreateIntToPtr(
   774:       And, PointerType::getUnqual(ValTy->getContext()), "itp");
   775: }
   776: 
   777: auto AlignVectors::createLoad(IRBuilderBase &Builder, Type *ValTy, Value *Ptr,
   778:                               Value *Predicate, int Alignment, Value *Mask,
   779:                               Value *PassThru,
   780:                               ArrayRef<Value *> MDSources) const -> Value * {
   781:   // Predicate is nullptr if not creating predicated load
   782:   if (Predicate) {
   783:     assert(!Predicate->getType()->isVectorTy() &&
   784:            "Expectning scalar predicate");
   785:     if (HVC.isFalse(Predicate))
   786:       return UndefValue::get(ValTy);
   787:     if (!HVC.isTrue(Predicate)) {
   788:       Value *Load = createPredicatedLoad(Builder, ValTy, Ptr, Predicate,
   789:                                          Alignment, MDSources);
   790:       return Builder.CreateSelect(Mask, Load, PassThru);
   791:     }
   792:     // Predicate == true here.
   793:   }
   794:   assert(!HVC.isUndef(Mask)); // Should this be allowed?
   795:   if (HVC.isZero(Mask))
   796:     return PassThru;
   797: 
   798:   Align EffA = effectiveAlignForValueTy(HVC.DL, ValTy, Alignment);
   799:   if (HVC.isTrue(Mask))
   800:     return createSimpleLoad(Builder, ValTy, Ptr, EffA.value(), MDSources);
```
- EN: It declares or implements routines such as std::min, std::max, emplace_back, Values, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 std::min, std::max, emplace_back, Values, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801: 
   802:   Instruction *Load =
   803:       Builder.CreateMaskedLoad(ValTy, Ptr, EffA, Mask, PassThru, "mld");
   804:   LLVM_DEBUG(dbgs() << "\t[Creating masked Load:] "; Load->dump());
   805:   propagateMetadata(Load, MDSources);
   806:   return Load;
   807: }
   808: 
   809: auto AlignVectors::createSimpleLoad(IRBuilderBase &Builder, Type *ValTy,
   810:                                     Value *Ptr, int Alignment,
   811:                                     ArrayRef<Value *> MDSources) const
   812:     -> Value * {
   813:   Align EffA = effectiveAlignForValueTy(HVC.DL, ValTy, Alignment);
   814:   Instruction *Load = Builder.CreateAlignedLoad(ValTy, Ptr, EffA, "ald");
   815:   propagateMetadata(Load, MDSources);
   816:   LLVM_DEBUG(dbgs() << "\t[Creating Load:] "; Load->dump());
   817:   return Load;
   818: }
   819: 
   820: auto AlignVectors::createPredicatedLoad(IRBuilderBase &Builder, Type *ValTy,
   821:                                         Value *Ptr, Value *Predicate,
   822:                                         int Alignment,
   823:                                         ArrayRef<Value *> MDSources) const
   824:     -> Value * {
   825:   assert(HVC.HST.isTypeForHVX(ValTy) &&
   826:          "Predicates 'scalar' vector loads not yet supported");
   827:   assert(Predicate);
   828:   assert(!Predicate->getType()->isVectorTy() && "Expectning scalar predicate");
   829:   Align EffA = effectiveAlignForValueTy(HVC.DL, ValTy, Alignment);
   830:   assert(HVC.getSizeOf(ValTy, HVC.Alloc) % EffA.value() == 0);
   831: 
   832:   if (HVC.isFalse(Predicate))
   833:     return UndefValue::get(ValTy);
   834:   if (HVC.isTrue(Predicate))
   835:     return createSimpleLoad(Builder, ValTy, Ptr, EffA.value(), MDSources);
   836: 
   837:   auto V6_vL32b_pred_ai = HVC.HST.getIntrinsicId(Hexagon::V6_vL32b_pred_ai);
   838:   // FIXME: This may not put the offset from Ptr into the vmem offset.
   839:   return HVC.createHvxIntrinsic(Builder, V6_vL32b_pred_ai, ValTy,
   840:                                 {Predicate, Ptr, HVC.getConstInt(0)}, {},
   841:                                 MDSources);
   842: }
   843: 
   844: auto AlignVectors::createStore(IRBuilderBase &Builder, Value *Val, Value *Ptr,
   845:                                Value *Predicate, int Alignment, Value *Mask,
   846:                                ArrayRef<Value *> MDSources) const -> Value * {
   847:   if (HVC.isZero(Mask) || HVC.isUndef(Val) || HVC.isUndef(Mask))
   848:     return UndefValue::get(Val->getType());
   849:   assert(!Predicate || (!Predicate->getType()->isVectorTy() &&
   850:                         "Expectning scalar predicate"));
   851:   if (Predicate) {
   852:     if (HVC.isFalse(Predicate))
   853:       return UndefValue::get(Val->getType());
   854:     if (HVC.isTrue(Predicate))
   855:       Predicate = nullptr;
   856:   }
   857:   // Here both Predicate and Mask are true or unknown.
   858: 
   859:   if (HVC.isTrue(Mask)) {
   860:     if (Predicate) { // Predicate unknown
   861:       return createPredicatedStore(Builder, Val, Ptr, Predicate, Alignment,
   862:                                    MDSources);
   863:     }
   864:     // Predicate is true:
   865:     return createSimpleStore(Builder, Val, Ptr, Alignment, MDSources);
   866:   }
   867: 
   868:   // Mask is unknown
   869:   if (!Predicate) {
   870:     Instruction *Store =
   871:         Builder.CreateMaskedStore(Val, Ptr, Align(Alignment), Mask);
   872:     propagateMetadata(Store, MDSources);
   873:     return Store;
   874:   }
   875: 
   876:   // Both Predicate and Mask are unknown.
   877:   // Emulate masked store with predicated-load + mux + predicated-store.
   878:   Value *PredLoad = createPredicatedLoad(Builder, Val->getType(), Ptr,
   879:                                          Predicate, Alignment, MDSources);
   880:   Value *Mux = Builder.CreateSelect(Mask, Val, PredLoad);
   881:   return createPredicatedStore(Builder, Mux, Ptr, Predicate, Alignment,
   882:                                MDSources);
   883: }
   884: 
   885: auto AlignVectors::createSimpleStore(IRBuilderBase &Builder, Value *Val,
   886:                                      Value *Ptr, int Alignment,
   887:                                      ArrayRef<Value *> MDSources) const
   888:     -> Value * {
   889:   Align EffA = effectiveAlignForValueTy(HVC.DL, Val->getType(), Alignment);
   890:   Instruction *Store = Builder.CreateAlignedStore(Val, Ptr, EffA);
   891:   LLVM_DEBUG(dbgs() << "\t[Creating store:] "; Store->dump());
   892:   propagateMetadata(Store, MDSources);
   893:   return Store;
   894: }
   895: 
   896: auto AlignVectors::createPredicatedStore(IRBuilderBase &Builder, Value *Val,
   897:                                          Value *Ptr, Value *Predicate,
   898:                                          int Alignment,
   899:                                          ArrayRef<Value *> MDSources) const
   900:     -> Value * {
   901:   Align EffA = effectiveAlignForValueTy(HVC.DL, Val->getType(), Alignment);
   902:   assert(HVC.HST.isTypeForHVX(Val->getType()) &&
   903:          "Predicates 'scalar' vector stores not yet supported");
   904:   assert(Predicate);
   905:   if (HVC.isFalse(Predicate))
   906:     return UndefValue::get(Val->getType());
   907:   if (HVC.isTrue(Predicate))
   908:     return createSimpleStore(Builder, Val, Ptr, EffA.value(), MDSources);
   909: 
   910:   assert(HVC.getSizeOf(Val, HVC.Alloc) % EffA.value() == 0);
   911:   auto V6_vS32b_pred_ai = HVC.HST.getIntrinsicId(Hexagon::V6_vS32b_pred_ai);
   912:   // FIXME: This may not put the offset from Ptr into the vmem offset.
   913:   return HVC.createHvxIntrinsic(Builder, V6_vS32b_pred_ai, nullptr,
   914:                                 {Predicate, Ptr, HVC.getConstInt(0), Val}, {},
   915:                                 MDSources);
   916: }
   917: 
   918: auto AlignVectors::getUpwardDeps(Instruction *In, Instruction *Base) const
   919:     -> DepList {
   920:   BasicBlock *Parent = Base->getParent();
   921:   assert(In->getParent() == Parent &&
   922:          "Base and In should be in the same block");
   923:   assert(Base->comesBefore(In) && "Base should come before In");
   924: 
   925:   DepList Deps;
   926:   std::deque<Instruction *> WorkQ = {In};
   927:   while (!WorkQ.empty()) {
   928:     Instruction *D = WorkQ.front();
   929:     WorkQ.pop_front();
   930:     if (D != In)
   931:       Deps.insert(D);
   932:     for (Value *Op : D->operands()) {
   933:       if (auto *I = dyn_cast<Instruction>(Op)) {
   934:         if (I->getParent() == Parent && Base->comesBefore(I))
   935:           WorkQ.push_back(I);
   936:       }
   937:     }
   938:   }
   939:   return Deps;
   940: }
   941: 
   942: auto AlignVectors::createAddressGroups() -> bool {
   943:   // An address group created here may contain instructions spanning
   944:   // multiple basic blocks.
   945:   AddrList WorkStack;
   946: 
   947:   auto findBaseAndOffset = [&](AddrInfo &AI) -> std::pair<Instruction *, int> {
   948:     for (AddrInfo &W : WorkStack) {
   949:       if (auto D = HVC.calculatePointerDifference(AI.Addr, W.Addr))
   950:         return std::make_pair(W.Inst, *D);
   951:     }
   952:     return std::make_pair(nullptr, 0);
   953:   };
   954: 
   955:   auto traverseBlock = [&](DomTreeNode *DomN, auto Visit) -> void {
   956:     BasicBlock &Block = *DomN->getBlock();
   957:     for (Instruction &I : Block) {
   958:       auto AI = this->getAddrInfo(I); // Use this-> for gcc6.
   959:       if (!AI)
   960:         continue;
   961:       auto F = findBaseAndOffset(*AI);
   962:       Instruction *GroupInst;
   963:       if (Instruction *BI = F.first) {
   964:         AI->Offset = F.second;
   965:         GroupInst = BI;
   966:       } else {
   967:         WorkStack.push_back(*AI);
   968:         GroupInst = AI->Inst;
   969:       }
   970:       AddrGroups[GroupInst].push_back(*AI);
   971:     }
   972: 
   973:     for (DomTreeNode *C : DomN->children())
   974:       Visit(C, Visit);
   975: 
   976:     while (!WorkStack.empty() && WorkStack.back().Inst->getParent() == &Block)
   977:       WorkStack.pop_back();
   978:   };
   979: 
   980:   traverseBlock(HVC.DT.getRootNode(), traverseBlock);
   981:   assert(WorkStack.empty());
   982: 
   983:   // AddrGroups are formed.
   984:   // Remove groups of size 1.
   985:   AddrGroups.remove_if([](auto &G) { return G.second.size() == 1; });
   986:   // Remove groups that don't use HVX types.
   987:   AddrGroups.remove_if([&](auto &G) {
   988:     return llvm::none_of(
   989:         G.second, [&](auto &I) { return HVC.HST.isTypeForHVX(I.ValTy); });
   990:   });
   991: 
   992:   LLVM_DEBUG(dbgs() << AddrGroups);
   993:   return !AddrGroups.empty();
   994: }
   995: 
   996: auto AlignVectors::createLoadGroups(const AddrList &Group) const -> MoveList {
   997:   // Form load groups.
   998:   // To avoid complications with moving code across basic blocks, only form
   999:   // groups that are contained within a single basic block.
  1000:   unsigned SizeLimit = VAGroupSizeLimit;
```
- EN: It declares or implements routines such as CreateMaskedLoad, dump, propagateMetadata, effectiveAlignForValueTy, ... (27 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 CreateMaskedLoad, dump, propagateMetadata, effectiveAlignForValueTy, ... (27 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:   if (SizeLimit == 0)
  1002:     return {};
  1003: 
  1004:   auto tryAddTo = [&](const AddrInfo &Info, MoveGroup &Move) {
  1005:     assert(!Move.Main.empty() && "Move group should have non-empty Main");
  1006:     if (Move.Main.size() >= SizeLimit)
  1007:       return false;
  1008:     // Don't mix HVX and non-HVX instructions.
  1009:     if (Move.IsHvx != isHvx(Info))
  1010:       return false;
  1011:     // Leading instruction in the load group.
  1012:     Instruction *Base = Move.Main.front();
  1013:     if (Base->getParent() != Info.Inst->getParent())
  1014:       return false;
  1015:     // Check if it's safe to move the load.
  1016:     if (!HVC.isSafeToMoveBeforeInBB(*Info.Inst, Base->getIterator()))
  1017:       return false;
  1018:     // And if it's safe to clone the dependencies.
  1019:     auto isSafeToCopyAtBase = [&](const Instruction *I) {
  1020:       return HVC.isSafeToMoveBeforeInBB(*I, Base->getIterator()) &&
  1021:              HVC.isSafeToClone(*I);
  1022:     };
  1023:     DepList Deps = getUpwardDeps(Info.Inst, Base);
  1024:     if (!llvm::all_of(Deps, isSafeToCopyAtBase))
  1025:       return false;
  1026: 
  1027:     Move.Main.push_back(Info.Inst);
  1028:     llvm::append_range(Move.Deps, Deps);
  1029:     return true;
  1030:   };
  1031: 
  1032:   MoveList LoadGroups;
  1033: 
  1034:   for (const AddrInfo &Info : Group) {
  1035:     if (!Info.Inst->mayReadFromMemory())
  1036:       continue;
  1037:     if (LoadGroups.empty() || !tryAddTo(Info, LoadGroups.back()))
  1038:       LoadGroups.emplace_back(Info, Group.front().Inst, isHvx(Info), true);
  1039:   }
  1040: 
  1041:   // Erase groups smaller than the minimum load group size.
  1042:   unsigned LoadGroupSizeLimit = MinLoadGroupSizeForAlignment;
  1043:   erase_if(LoadGroups, [LoadGroupSizeLimit](const MoveGroup &G) {
  1044:     return G.Main.size() < LoadGroupSizeLimit;
  1045:   });
  1046: 
  1047:   // Erase HVX groups on targets < HvxV62 (due to lack of predicated loads).
  1048:   if (!HVC.HST.useHVXV62Ops())
  1049:     erase_if(LoadGroups, [](const MoveGroup &G) { return G.IsHvx; });
  1050: 
  1051:   LLVM_DEBUG(dbgs() << "LoadGroups list: " << LoadGroups);
  1052:   return LoadGroups;
  1053: }
  1054: 
  1055: auto AlignVectors::createStoreGroups(const AddrList &Group) const -> MoveList {
  1056:   // Form store groups.
  1057:   // To avoid complications with moving code across basic blocks, only form
  1058:   // groups that are contained within a single basic block.
  1059:   unsigned SizeLimit = VAGroupSizeLimit;
  1060:   if (SizeLimit == 0)
  1061:     return {};
  1062: 
  1063:   auto tryAddTo = [&](const AddrInfo &Info, MoveGroup &Move) {
  1064:     assert(!Move.Main.empty() && "Move group should have non-empty Main");
  1065:     if (Move.Main.size() >= SizeLimit)
  1066:       return false;
  1067:     // For stores with return values we'd have to collect downward dependencies.
  1068:     // There are no such stores that we handle at the moment, so omit that.
  1069:     assert(Info.Inst->getType()->isVoidTy() &&
  1070:            "Not handling stores with return values");
  1071:     // Don't mix HVX and non-HVX instructions.
  1072:     if (Move.IsHvx != isHvx(Info))
  1073:       return false;
  1074:     // For stores we need to be careful whether it's safe to move them.
  1075:     // Stores that are otherwise safe to move together may not appear safe
  1076:     // to move over one another (i.e. isSafeToMoveBefore may return false).
  1077:     Instruction *Base = Move.Main.front();
  1078:     if (Base->getParent() != Info.Inst->getParent())
  1079:       return false;
  1080:     if (!HVC.isSafeToMoveBeforeInBB(*Info.Inst, Base->getIterator(), Move.Main))
  1081:       return false;
  1082:     Move.Main.push_back(Info.Inst);
  1083:     return true;
  1084:   };
  1085: 
  1086:   MoveList StoreGroups;
  1087: 
  1088:   for (auto I = Group.rbegin(), E = Group.rend(); I != E; ++I) {
  1089:     const AddrInfo &Info = *I;
  1090:     if (!Info.Inst->mayWriteToMemory())
  1091:       continue;
  1092:     if (StoreGroups.empty() || !tryAddTo(Info, StoreGroups.back()))
  1093:       StoreGroups.emplace_back(Info, Group.front().Inst, isHvx(Info), false);
  1094:   }
  1095: 
  1096:   // Erase singleton groups.
  1097:   erase_if(StoreGroups, [](const MoveGroup &G) { return G.Main.size() <= 1; });
  1098: 
  1099:   // Erase HVX groups on targets < HvxV62 (due to lack of predicated loads).
  1100:   if (!HVC.HST.useHVXV62Ops())
  1101:     erase_if(StoreGroups, [](const MoveGroup &G) { return G.IsHvx; });
  1102: 
  1103:   // Erase groups where every store is a full HVX vector. The reason is that
  1104:   // aligning predicated stores generates complex code that may be less
  1105:   // efficient than a sequence of unaligned vector stores.
  1106:   if (!VADoFullStores) {
  1107:     erase_if(StoreGroups, [this](const MoveGroup &G) {
  1108:       return G.IsHvx && llvm::all_of(G.Main, [this](Instruction *S) {
  1109:                auto MaybeInfo = this->getAddrInfo(*S);
  1110:                assert(MaybeInfo.has_value());
  1111:                return HVC.HST.isHVXVectorType(
  1112:                    EVT::getEVT(MaybeInfo->ValTy, false));
  1113:              });
  1114:     });
  1115:   }
  1116: 
  1117:   return StoreGroups;
  1118: }
  1119: 
  1120: auto AlignVectors::moveTogether(MoveGroup &Move) const -> bool {
  1121:   // Move all instructions to be adjacent.
  1122:   assert(!Move.Main.empty() && "Move group should have non-empty Main");
  1123:   Instruction *Where = Move.Main.front();
  1124: 
  1125:   if (Move.IsLoad) {
  1126:     // Move all the loads (and dependencies) to where the first load is.
  1127:     // Clone all deps to before Where, keeping order.
  1128:     Move.Clones = cloneBefore(Where->getIterator(), Move.Deps);
  1129:     // Move all main instructions to after Where, keeping order.
  1130:     ArrayRef<Instruction *> Main(Move.Main);
  1131:     for (Instruction *M : Main) {
  1132:       if (M != Where)
  1133:         M->moveAfter(Where);
  1134:       for (auto [Old, New] : Move.Clones)
  1135:         M->replaceUsesOfWith(Old, New);
  1136:       Where = M;
  1137:     }
  1138:     // Replace Deps with the clones.
  1139:     for (int i = 0, e = Move.Deps.size(); i != e; ++i)
  1140:       Move.Deps[i] = Move.Clones[Move.Deps[i]];
  1141:   } else {
  1142:     // Move all the stores to where the last store is.
  1143:     // NOTE: Deps are empty for "store" groups. If they need to be
  1144:     // non-empty, decide on the order.
  1145:     assert(Move.Deps.empty());
  1146:     // Move all main instructions to before Where, inverting order.
  1147:     ArrayRef<Instruction *> Main(Move.Main);
  1148:     for (Instruction *M : Main.drop_front(1)) {
  1149:       M->moveBefore(Where->getIterator());
  1150:       Where = M;
  1151:     }
  1152:   }
  1153: 
  1154:   return Move.Main.size() + Move.Deps.size() > 1;
  1155: }
  1156: 
  1157: template <typename T>
  1158: auto AlignVectors::cloneBefore(BasicBlock::iterator To, T &&Insts) const
  1159:     -> InstMap {
  1160:   InstMap Map;
  1161: 
  1162:   for (Instruction *I : Insts) {
  1163:     assert(HVC.isSafeToClone(*I));
  1164:     Instruction *C = I->clone();
  1165:     C->setName(Twine("c.") + I->getName() + ".");
  1166:     C->insertBefore(To);
  1167: 
  1168:     for (auto [Old, New] : Map)
  1169:       C->replaceUsesOfWith(Old, New);
  1170:     Map.insert(std::make_pair(I, C));
  1171:   }
  1172:   return Map;
  1173: }
  1174: 
  1175: auto AlignVectors::realignLoadGroup(IRBuilderBase &Builder,
  1176:                                     const ByteSpan &VSpan, int ScLen,
  1177:                                     Value *AlignVal, Value *AlignAddr) const
  1178:     -> void {
  1179:   LLVM_DEBUG(dbgs() << __func__ << "\n");
  1180: 
  1181:   Type *SecTy = HVC.getByteTy(ScLen);
  1182:   int NumSectors = (VSpan.extent() + ScLen - 1) / ScLen;
  1183:   bool DoAlign = !HVC.isZero(AlignVal);
  1184:   BasicBlock::iterator BasePos = Builder.GetInsertPoint();
  1185:   BasicBlock *BaseBlock = Builder.GetInsertBlock();
  1186: 
  1187:   ByteSpan ASpan;
  1188:   auto *True = Constant::getAllOnesValue(HVC.getBoolTy(ScLen));
  1189:   auto *Undef = UndefValue::get(SecTy);
  1190: 
  1191:   // Created load does not have to be "Instruction" (e.g. "undef").
  1192:   SmallVector<Value *> Loads(NumSectors + DoAlign, nullptr);
  1193: 
  1194:   // We could create all of the aligned loads, and generate the valigns
  1195:   // at the location of the first load, but for large load groups, this
  1196:   // could create highly suboptimal code (there have been groups of 140+
  1197:   // loads in real code).
  1198:   // Instead, place the loads/valigns as close to the users as possible.
  1199:   // In any case we need to have a mapping from the blocks of VSpan (the
  1200:   // span covered by the pre-existing loads) to ASpan (the span covered
```
- EN: It declares or implements routines such as assert, front, isSafeToMoveBeforeInBB, getUpwardDeps, ... (27 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 assert, front, isSafeToMoveBeforeInBB, getUpwardDeps, ... (27 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:   // by the aligned loads). There is a small problem, though: ASpan needs
  1202:   // to have pointers to the loads/valigns, but we don't have these loads
  1203:   // because we don't know where to put them yet. We find out by creating
  1204:   // a section of ASpan that corresponds to values (blocks) from VSpan,
  1205:   // and checking where the new load should be placed. We need to attach
  1206:   // this location information to each block in ASpan somehow, so we put
  1207:   // distincts values for Seg.Val in each ASpan.Blocks[i], and use a map
  1208:   // to store the location for each Seg.Val.
  1209:   // The distinct values happen to be Blocks[i].Seg.Val = &Blocks[i],
  1210:   // which helps with printing ByteSpans without crashing when printing
  1211:   // Segments with these temporary identifiers in place of Val.
  1212: 
  1213:   // Populate the blocks first, to avoid reallocations of the vector
  1214:   // interfering with generating the placeholder addresses.
  1215:   for (int Index = 0; Index != NumSectors; ++Index)
  1216:     ASpan.Blocks.emplace_back(nullptr, ScLen, Index * ScLen);
  1217:   for (int Index = 0; Index != NumSectors; ++Index) {
  1218:     ASpan.Blocks[Index].Seg.Val =
  1219:         reinterpret_cast<Value *>(&ASpan.Blocks[Index]);
  1220:   }
  1221: 
  1222:   // Multiple values from VSpan can map to the same value in ASpan. Since we
  1223:   // try to create loads lazily, we need to find the earliest use for each
  1224:   // value from ASpan.
  1225:   DenseMap<void *, Instruction *> EarliestUser;
  1226:   auto isEarlier = [](Instruction *A, Instruction *B) {
  1227:     if (B == nullptr)
  1228:       return true;
  1229:     if (A == nullptr)
  1230:       return false;
  1231:     assert(A->getParent() == B->getParent());
  1232:     return A->comesBefore(B);
  1233:   };
  1234:   auto earliestUser = [&](const auto &Uses) {
  1235:     Instruction *User = nullptr;
  1236:     for (const Use &U : Uses) {
  1237:       auto *I = dyn_cast<Instruction>(U.getUser());
  1238:       assert(I != nullptr && "Load used in a non-instruction?");
  1239:       // Make sure we only consider users in this block, but we need
  1240:       // to remember if there were users outside the block too. This is
  1241:       // because if no users are found, aligned loads will not be created.
  1242:       if (I->getParent() == BaseBlock) {
  1243:         if (!isa<PHINode>(I))
  1244:           User = std::min(User, I, isEarlier);
  1245:       } else {
  1246:         User = std::min(User, BaseBlock->getTerminator(), isEarlier);
  1247:       }
  1248:     }
  1249:     return User;
  1250:   };
  1251: 
  1252:   for (const ByteSpan::Block &B : VSpan) {
  1253:     ByteSpan ASection = ASpan.section(B.Pos, B.Seg.Size);
  1254:     for (const ByteSpan::Block &S : ASection) {
  1255:       auto &EU = EarliestUser[S.Seg.Val];
  1256:       EU = std::min(EU, earliestUser(B.Seg.Val->uses()), isEarlier);
  1257:     }
  1258:   }
  1259: 
  1260:   LLVM_DEBUG({
  1261:     dbgs() << "ASpan:\n" << ASpan << '\n';
  1262:     dbgs() << "Earliest users of ASpan:\n";
  1263:     for (auto &[Val, User] : EarliestUser) {
  1264:       dbgs() << Val << "\n ->" << *User << '\n';
  1265:     }
  1266:   });
  1267: 
  1268:   auto createLoad = [&](IRBuilderBase &Builder, const ByteSpan &VSpan,
  1269:                         int Index, bool MakePred) {
  1270:     Value *Ptr =
  1271:         createAdjustedPointer(Builder, AlignAddr, SecTy, Index * ScLen);
  1272:     Value *Predicate =
  1273:         MakePred ? makeTestIfUnaligned(Builder, AlignVal, ScLen) : nullptr;
  1274: 
  1275:     // If vector shifting is potentially needed, accumulate metadata
  1276:     // from source sections of twice the load width.
  1277:     int Start = (Index - DoAlign) * ScLen;
  1278:     int Width = (1 + DoAlign) * ScLen;
  1279:     return this->createLoad(Builder, SecTy, Ptr, Predicate, ScLen, True, Undef,
  1280:                             VSpan.section(Start, Width).values());
  1281:   };
  1282: 
  1283:   auto moveBefore = [this](BasicBlock::iterator In, BasicBlock::iterator To) {
  1284:     // Move In and its upward dependencies to before To.
  1285:     assert(In->getParent() == To->getParent());
  1286:     DepList Deps = getUpwardDeps(&*In, &*To);
  1287:     In->moveBefore(To);
  1288:     // DepList is sorted with respect to positions in the basic block.
  1289:     InstMap Map = cloneBefore(In, Deps);
  1290:     for (auto [Old, New] : Map)
  1291:       In->replaceUsesOfWith(Old, New);
  1292:   };
  1293: 
  1294:   // Generate necessary loads at appropriate locations.
  1295:   LLVM_DEBUG(dbgs() << "Creating loads for ASpan sectors\n");
  1296:   for (int Index = 0; Index != NumSectors + 1; ++Index) {
  1297:     // In ASpan, each block will be either a single aligned load, or a
  1298:     // valign of a pair of loads. In the latter case, an aligned load j
  1299:     // will belong to the current valign, and the one in the previous
  1300:     // block (for j > 0).
  1301:     // Place the load at a location which will dominate the valign, assuming
  1302:     // the valign will be placed right before the earliest user.
  1303:     Instruction *PrevAt =
  1304:         DoAlign && Index > 0 ? EarliestUser[&ASpan[Index - 1]] : nullptr;
  1305:     Instruction *ThisAt =
  1306:         Index < NumSectors ? EarliestUser[&ASpan[Index]] : nullptr;
  1307:     if (auto *Where = std::min(PrevAt, ThisAt, isEarlier)) {
  1308:       Builder.SetInsertPoint(Where);
  1309:       Loads[Index] =
  1310:           createLoad(Builder, VSpan, Index, DoAlign && Index == NumSectors);
  1311:       // We know it's safe to put the load at BasePos, but we'd prefer to put
  1312:       // it at "Where". To see if the load is safe to be placed at Where, put
  1313:       // it there first and then check if it's safe to move it to BasePos.
  1314:       // If not, then the load needs to be placed at BasePos.
  1315:       // We can't do this check proactively because we need the load to exist
  1316:       // in order to check legality.
  1317:       if (auto *Load = dyn_cast<Instruction>(Loads[Index])) {
  1318:         if (!HVC.isSafeToMoveBeforeInBB(*Load, BasePos))
  1319:           moveBefore(Load->getIterator(), BasePos);
  1320:       }
  1321:       LLVM_DEBUG(dbgs() << "Loads[" << Index << "]:" << *Loads[Index] << '\n');
  1322:     }
  1323:   }
  1324: 
  1325:   // Generate valigns if needed, and fill in proper values in ASpan
  1326:   LLVM_DEBUG(dbgs() << "Creating values for ASpan sectors\n");
  1327:   for (int Index = 0; Index != NumSectors; ++Index) {
  1328:     ASpan[Index].Seg.Val = nullptr;
  1329:     if (auto *Where = EarliestUser[&ASpan[Index]]) {
  1330:       Builder.SetInsertPoint(Where);
  1331:       Value *Val = Loads[Index];
  1332:       assert(Val != nullptr);
  1333:       if (DoAlign) {
  1334:         Value *NextLoad = Loads[Index + 1];
  1335:         assert(NextLoad != nullptr);
  1336:         Val = HVC.vralignb(Builder, Val, NextLoad, AlignVal);
  1337:       }
  1338:       ASpan[Index].Seg.Val = Val;
  1339:       LLVM_DEBUG(dbgs() << "ASpan[" << Index << "]:" << *Val << '\n');
  1340:     }
  1341:   }
  1342: 
  1343:   for (const ByteSpan::Block &B : VSpan) {
  1344:     ByteSpan ASection = ASpan.section(B.Pos, B.Seg.Size).shift(-B.Pos);
  1345:     Value *Accum = UndefValue::get(HVC.getByteTy(B.Seg.Size));
  1346:     Builder.SetInsertPoint(cast<Instruction>(B.Seg.Val));
  1347: 
  1348:     // We're generating a reduction, where each instruction depends on
  1349:     // the previous one, so we need to order them according to the position
  1350:     // of their inputs in the code.
  1351:     std::vector<ByteSpan::Block *> ABlocks;
  1352:     for (ByteSpan::Block &S : ASection) {
  1353:       if (S.Seg.Val != nullptr)
  1354:         ABlocks.push_back(&S);
  1355:     }
  1356:     llvm::sort(ABlocks,
  1357:                [&](const ByteSpan::Block *A, const ByteSpan::Block *B) {
  1358:                  return isEarlier(cast<Instruction>(A->Seg.Val),
  1359:                                   cast<Instruction>(B->Seg.Val));
  1360:                });
  1361:     for (ByteSpan::Block *S : ABlocks) {
  1362:       // The processing of the data loaded by the aligned loads
  1363:       // needs to be inserted after the data is available.
  1364:       Instruction *SegI = cast<Instruction>(S->Seg.Val);
  1365:       Builder.SetInsertPoint(&*std::next(SegI->getIterator()));
  1366:       Value *Pay = HVC.vbytes(Builder, getPayload(S->Seg.Val));
  1367:       Accum =
  1368:           HVC.insertb(Builder, Accum, Pay, S->Seg.Start, S->Seg.Size, S->Pos);
  1369:     }
  1370:     // Instead of casting everything to bytes for the vselect, cast to the
  1371:     // original value type. This will avoid complications with casting masks.
  1372:     // For example, in cases when the original mask applied to i32, it could
  1373:     // be converted to a mask applicable to i8 via pred_typecast intrinsic,
  1374:     // but if the mask is not exactly of HVX length, extra handling would be
  1375:     // needed to make it work.
  1376:     Type *ValTy = getPayload(B.Seg.Val)->getType();
  1377:     Value *Cast = Builder.CreateBitCast(Accum, ValTy, "cst");
  1378:     Value *Sel = Builder.CreateSelect(getMask(B.Seg.Val), Cast,
  1379:                                       getPassThrough(B.Seg.Val), "sel");
  1380:     B.Seg.Val->replaceAllUsesWith(Sel);
  1381:   }
  1382: }
  1383: 
  1384: auto AlignVectors::realignStoreGroup(IRBuilderBase &Builder,
  1385:                                      const ByteSpan &VSpan, int ScLen,
  1386:                                      Value *AlignVal, Value *AlignAddr) const
  1387:     -> void {
  1388:   LLVM_DEBUG(dbgs() << __func__ << "\n");
  1389: 
  1390:   Type *SecTy = HVC.getByteTy(ScLen);
  1391:   int NumSectors = (VSpan.extent() + ScLen - 1) / ScLen;
  1392:   bool DoAlign = !HVC.isZero(AlignVal);
  1393: 
  1394:   // Stores.
  1395:   ByteSpan ASpanV, ASpanM;
  1396: 
  1397:   // Return a vector value corresponding to the input value Val:
  1398:   // either <1 x Val> for scalar Val, or Val itself for vector Val.
  1399:   auto MakeVec = [](IRBuilderBase &Builder, Value *Val) -> Value * {
  1400:     Type *Ty = Val->getType();
```
- EN: It declares or implements routines such as emplace_back, assert, comesBefore, dyn_cast<Instruction>, ... (27 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 emplace_back, assert, comesBefore, dyn_cast<Instruction>, ... (27 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:     if (Ty->isVectorTy())
  1402:       return Val;
  1403:     auto *VecTy = VectorType::get(Ty, 1, /*Scalable=*/false);
  1404:     return Builder.CreateBitCast(Val, VecTy, "cst");
  1405:   };
  1406: 
  1407:   // Create an extra "undef" sector at the beginning and at the end.
  1408:   // They will be used as the left/right filler in the vlalign step.
  1409:   for (int Index = (DoAlign ? -1 : 0); Index != NumSectors + DoAlign; ++Index) {
  1410:     // For stores, the size of each section is an aligned vector length.
  1411:     // Adjust the store offsets relative to the section start offset.
  1412:     ByteSpan VSection =
  1413:         VSpan.section(Index * ScLen, ScLen).shift(-Index * ScLen);
  1414:     Value *Undef = UndefValue::get(SecTy);
  1415:     Value *Zero = Constant::getNullValue(SecTy);
  1416:     Value *AccumV = Undef;
  1417:     Value *AccumM = Zero;
  1418:     for (ByteSpan::Block &S : VSection) {
  1419:       Value *Pay = getPayload(S.Seg.Val);
  1420:       Value *Mask = HVC.rescale(Builder, MakeVec(Builder, getMask(S.Seg.Val)),
  1421:                                 Pay->getType(), HVC.getByteTy());
  1422:       Value *PartM = HVC.insertb(Builder, Zero, HVC.vbytes(Builder, Mask),
  1423:                                  S.Seg.Start, S.Seg.Size, S.Pos);
  1424:       AccumM = Builder.CreateOr(AccumM, PartM);
  1425: 
  1426:       Value *PartV = HVC.insertb(Builder, Undef, HVC.vbytes(Builder, Pay),
  1427:                                  S.Seg.Start, S.Seg.Size, S.Pos);
  1428: 
  1429:       AccumV = Builder.CreateSelect(
  1430:           Builder.CreateICmp(CmpInst::ICMP_NE, PartM, Zero), PartV, AccumV);
  1431:     }
  1432:     ASpanV.Blocks.emplace_back(AccumV, ScLen, Index * ScLen);
  1433:     ASpanM.Blocks.emplace_back(AccumM, ScLen, Index * ScLen);
  1434:   }
  1435: 
  1436:   LLVM_DEBUG({
  1437:     dbgs() << "ASpanV before vlalign:\n" << ASpanV << '\n';
  1438:     dbgs() << "ASpanM before vlalign:\n" << ASpanM << '\n';
  1439:   });
  1440: 
  1441:   // vlalign
  1442:   if (DoAlign) {
  1443:     for (int Index = 1; Index != NumSectors + 2; ++Index) {
  1444:       Value *PrevV = ASpanV[Index - 1].Seg.Val, *ThisV = ASpanV[Index].Seg.Val;
  1445:       Value *PrevM = ASpanM[Index - 1].Seg.Val, *ThisM = ASpanM[Index].Seg.Val;
  1446:       assert(isSectorTy(PrevV->getType()) && isSectorTy(PrevM->getType()));
  1447:       ASpanV[Index - 1].Seg.Val = HVC.vlalignb(Builder, PrevV, ThisV, AlignVal);
  1448:       ASpanM[Index - 1].Seg.Val = HVC.vlalignb(Builder, PrevM, ThisM, AlignVal);
  1449:     }
  1450:   }
  1451: 
  1452:   LLVM_DEBUG({
  1453:     dbgs() << "ASpanV after vlalign:\n" << ASpanV << '\n';
  1454:     dbgs() << "ASpanM after vlalign:\n" << ASpanM << '\n';
  1455:   });
  1456: 
  1457:   auto createStore = [&](IRBuilderBase &Builder, const ByteSpan &ASpanV,
  1458:                          const ByteSpan &ASpanM, int Index, bool MakePred) {
  1459:     Value *Val = ASpanV[Index].Seg.Val;
  1460:     Value *Mask = ASpanM[Index].Seg.Val; // bytes
  1461:     if (HVC.isUndef(Val) || HVC.isZero(Mask))
  1462:       return;
  1463:     Value *Ptr =
  1464:         createAdjustedPointer(Builder, AlignAddr, SecTy, Index * ScLen);
  1465:     Value *Predicate =
  1466:         MakePred ? makeTestIfUnaligned(Builder, AlignVal, ScLen) : nullptr;
  1467: 
  1468:     // If vector shifting is potentially needed, accumulate metadata
  1469:     // from source sections of twice the store width.
  1470:     int Start = (Index - DoAlign) * ScLen;
  1471:     int Width = (1 + DoAlign) * ScLen;
  1472:     this->createStore(Builder, Val, Ptr, Predicate, ScLen,
  1473:                       HVC.vlsb(Builder, Mask),
  1474:                       VSpan.section(Start, Width).values());
  1475:   };
  1476: 
  1477:   for (int Index = 0; Index != NumSectors + DoAlign; ++Index) {
  1478:     createStore(Builder, ASpanV, ASpanM, Index, DoAlign && Index == NumSectors);
  1479:   }
  1480: }
  1481: 
  1482: auto AlignVectors::realignGroup(const MoveGroup &Move) -> bool {
  1483:   LLVM_DEBUG(dbgs() << "Realigning group:\n" << Move << '\n');
  1484: 
  1485:   // TODO: Needs support for masked loads/stores of "scalar" vectors.
  1486:   if (!Move.IsHvx)
  1487:     return false;
  1488: 
  1489:   // Return the element with the maximum alignment from Range,
  1490:   // where GetValue obtains the value to compare from an element.
  1491:   auto getMaxOf = [](auto Range, auto GetValue) {
  1492:     return *llvm::max_element(Range, [&GetValue](auto &A, auto &B) {
  1493:       return GetValue(A) < GetValue(B);
  1494:     });
  1495:   };
  1496: 
  1497:   AddrList &BaseInfos = AddrGroups[Move.Base];
  1498: 
  1499:   // Conceptually, there is a vector of N bytes covering the addresses
  1500:   // starting from the minimum offset (i.e. Base.Addr+Start). This vector
  1501:   // represents a contiguous memory region that spans all accessed memory
  1502:   // locations.
  1503:   // The correspondence between loaded or stored values will be expressed
  1504:   // in terms of this vector. For example, the 0th element of the vector
  1505:   // from the Base address info will start at byte Start from the beginning
  1506:   // of this conceptual vector.
  1507:   //
  1508:   // This vector will be loaded/stored starting at the nearest down-aligned
  1509:   // address and the amount of the down-alignment will be AlignVal:
  1510:   //   valign(load_vector(align_down(Base+Start)), AlignVal)
  1511: 
  1512:   std::set<Instruction *> TestSet(Move.Main.begin(), Move.Main.end());
  1513:   AddrList MoveInfos;
  1514: 
  1515:   llvm::copy_if(
  1516:       BaseInfos, std::back_inserter(MoveInfos),
  1517:       [&TestSet](const AddrInfo &AI) { return TestSet.count(AI.Inst); });
  1518: 
  1519:   // Maximum alignment present in the whole address group.
  1520:   const AddrInfo &WithMaxAlign =
  1521:       getMaxOf(MoveInfos, [](const AddrInfo &AI) { return AI.HaveAlign; });
  1522:   Align MaxGiven = WithMaxAlign.HaveAlign;
  1523: 
  1524:   // Minimum alignment present in the move address group.
  1525:   const AddrInfo &WithMinOffset =
  1526:       getMaxOf(MoveInfos, [](const AddrInfo &AI) { return -AI.Offset; });
  1527: 
  1528:   const AddrInfo &WithMaxNeeded =
  1529:       getMaxOf(MoveInfos, [](const AddrInfo &AI) { return AI.NeedAlign; });
  1530:   Align MinNeeded = WithMaxNeeded.NeedAlign;
  1531: 
  1532:   // Set the builder's insertion point right before the load group, or
  1533:   // immediately after the store group. (Instructions in a store group are
  1534:   // listed in reverse order.)
  1535:   Instruction *InsertAt = Move.Main.front();
  1536:   if (!Move.IsLoad) {
  1537:     // There should be a terminator (which store isn't, but check anyways).
  1538:     assert(InsertAt->getIterator() != InsertAt->getParent()->end());
  1539:     InsertAt = &*std::next(InsertAt->getIterator());
  1540:   }
  1541: 
  1542:   IRBuilder Builder(InsertAt->getParent(), InsertAt->getIterator(),
  1543:                     InstSimplifyFolder(HVC.DL));
  1544:   Value *AlignAddr = nullptr; // Actual aligned address.
  1545:   Value *AlignVal = nullptr;  // Right-shift amount (for valign).
  1546: 
  1547:   if (MinNeeded <= MaxGiven) {
  1548:     int Start = WithMinOffset.Offset;
  1549:     int OffAtMax = WithMaxAlign.Offset;
  1550:     // Shift the offset of the maximally aligned instruction (OffAtMax)
  1551:     // back by just enough multiples of the required alignment to cover the
  1552:     // distance from Start to OffAtMax.
  1553:     // Calculate the address adjustment amount based on the address with the
  1554:     // maximum alignment. This is to allow a simple gep instruction instead
  1555:     // of potential bitcasts to i8*.
  1556:     int Adjust = -alignTo(OffAtMax - Start, MinNeeded.value());
  1557:     AlignAddr = createAdjustedPointer(Builder, WithMaxAlign.Addr,
  1558:                                       WithMaxAlign.ValTy, Adjust, Move.Clones);
  1559:     int Diff = Start - (OffAtMax + Adjust);
  1560:     AlignVal = HVC.getConstInt(Diff);
  1561:     assert(Diff >= 0);
  1562:     assert(static_cast<decltype(MinNeeded.value())>(Diff) < MinNeeded.value());
  1563:   } else {
  1564:     // WithMinOffset is the lowest address in the group,
  1565:     //   WithMinOffset.Addr = Base+Start.
  1566:     // Align instructions for both HVX (V6_valign) and scalar (S2_valignrb)
  1567:     // mask off unnecessary bits, so it's ok to just the original pointer as
  1568:     // the alignment amount.
  1569:     // Do an explicit down-alignment of the address to avoid creating an
  1570:     // aligned instruction with an address that is not really aligned.
  1571:     AlignAddr =
  1572:         createAlignedPointer(Builder, WithMinOffset.Addr, WithMinOffset.ValTy,
  1573:                              MinNeeded.value(), Move.Clones);
  1574:     AlignVal =
  1575:         Builder.CreatePtrToInt(WithMinOffset.Addr, HVC.getIntTy(), "pti");
  1576:     if (auto *I = dyn_cast<Instruction>(AlignVal)) {
  1577:       for (auto [Old, New] : Move.Clones)
  1578:         I->replaceUsesOfWith(Old, New);
  1579:     }
  1580:   }
  1581: 
  1582:   ByteSpan VSpan;
  1583:   for (const AddrInfo &AI : MoveInfos) {
  1584:     VSpan.Blocks.emplace_back(AI.Inst, HVC.getSizeOf(AI.ValTy),
  1585:                               AI.Offset - WithMinOffset.Offset);
  1586:   }
  1587: 
  1588:   // The aligned loads/stores will use blocks that are either scalars,
  1589:   // or HVX vectors. Let "sector" be the unified term for such a block.
  1590:   // blend(scalar, vector) -> sector...
  1591:   int ScLen = Move.IsHvx ? HVC.HST.getVectorLength()
  1592:                          : std::max<int>(MinNeeded.value(), 4);
  1593:   assert(!Move.IsHvx || ScLen == 64 || ScLen == 128);
  1594:   assert(Move.IsHvx || ScLen == 4 || ScLen == 8);
  1595: 
  1596:   LLVM_DEBUG({
  1597:     dbgs() << "ScLen:  " << ScLen << "\n";
  1598:     dbgs() << "AlignVal:" << *AlignVal << "\n";
  1599:     dbgs() << "AlignAddr:" << *AlignAddr << "\n";
  1600:     dbgs() << "VSpan:\n" << VSpan << '\n';
```
- EN: It declares or implements routines such as VectorType::get, CreateBitCast, section, UndefValue::get, ... (32 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 VectorType::get, CreateBitCast, section, UndefValue::get, ... (32 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:   });
  1602: 
  1603:   if (Move.IsLoad)
  1604:     realignLoadGroup(Builder, VSpan, ScLen, AlignVal, AlignAddr);
  1605:   else
  1606:     realignStoreGroup(Builder, VSpan, ScLen, AlignVal, AlignAddr);
  1607: 
  1608:   for (auto *Inst : Move.Main)
  1609:     Inst->eraseFromParent();
  1610: 
  1611:   return true;
  1612: }
  1613: 
  1614: auto AlignVectors::makeTestIfUnaligned(IRBuilderBase &Builder, Value *AlignVal,
  1615:                                        int Alignment) const -> Value * {
  1616:   auto *AlignTy = AlignVal->getType();
  1617:   Value *And = Builder.CreateAnd(
  1618:       AlignVal, ConstantInt::get(AlignTy, Alignment - 1), "and");
  1619:   Value *Zero = ConstantInt::get(AlignTy, 0);
  1620:   return Builder.CreateICmpNE(And, Zero, "isz");
  1621: }
  1622: 
  1623: auto AlignVectors::isSectorTy(Type *Ty) const -> bool {
  1624:   if (!HVC.isByteVecTy(Ty))
  1625:     return false;
  1626:   int Size = HVC.getSizeOf(Ty);
  1627:   if (HVC.HST.isTypeForHVX(Ty))
  1628:     return Size == static_cast<int>(HVC.HST.getVectorLength());
  1629:   return Size == 4 || Size == 8;
  1630: }
  1631: 
  1632: auto AlignVectors::run() -> bool {
  1633:   LLVM_DEBUG(dbgs() << "\nRunning HVC::AlignVectors on " << HVC.F.getName()
  1634:                     << '\n');
  1635:   if (!createAddressGroups())
  1636:     return false;
  1637: 
  1638:   LLVM_DEBUG({
  1639:     dbgs() << "Address groups(" << AddrGroups.size() << "):\n";
  1640:     for (auto &[In, AL] : AddrGroups) {
  1641:       for (const AddrInfo &AI : AL)
  1642:         dbgs() << "---\n" << AI << '\n';
  1643:     }
  1644:   });
  1645: 
  1646:   bool Changed = false;
  1647:   MoveList LoadGroups, StoreGroups;
  1648: 
  1649:   for (auto &G : AddrGroups) {
  1650:     llvm::append_range(LoadGroups, createLoadGroups(G.second));
  1651:     llvm::append_range(StoreGroups, createStoreGroups(G.second));
  1652:   }
  1653: 
  1654:   LLVM_DEBUG({
  1655:     dbgs() << "\nLoad groups(" << LoadGroups.size() << "):\n";
  1656:     for (const MoveGroup &G : LoadGroups)
  1657:       dbgs() << G << "\n";
  1658:     dbgs() << "Store groups(" << StoreGroups.size() << "):\n";
  1659:     for (const MoveGroup &G : StoreGroups)
  1660:       dbgs() << G << "\n";
  1661:   });
  1662: 
  1663:   // Cumulative limit on the number of groups.
  1664:   unsigned CountLimit = VAGroupCountLimit;
  1665:   if (CountLimit == 0)
  1666:     return false;
  1667: 
  1668:   if (LoadGroups.size() > CountLimit) {
  1669:     LoadGroups.resize(CountLimit);
  1670:     StoreGroups.clear();
  1671:   } else {
  1672:     unsigned StoreLimit = CountLimit - LoadGroups.size();
  1673:     if (StoreGroups.size() > StoreLimit)
  1674:       StoreGroups.resize(StoreLimit);
  1675:   }
  1676: 
  1677:   for (auto &M : LoadGroups)
  1678:     Changed |= moveTogether(M);
  1679:   for (auto &M : StoreGroups)
  1680:     Changed |= moveTogether(M);
  1681: 
  1682:   LLVM_DEBUG(dbgs() << "After moveTogether:\n" << HVC.F);
  1683: 
  1684:   for (auto &M : LoadGroups)
  1685:     Changed |= realignGroup(M);
  1686:   for (auto &M : StoreGroups)
  1687:     Changed |= realignGroup(M);
  1688: 
  1689:   return Changed;
  1690: }
  1691: 
  1692: // --- End AlignVectors
  1693: 
  1694: // --- Begin HvxIdioms
  1695: 
  1696: auto HvxIdioms::getNumSignificantBits(Value *V, Instruction *In) const
  1697:     -> std::pair<unsigned, Signedness> {
  1698:   unsigned Bits = HVC.getNumSignificantBits(V, In);
  1699:   // The significant bits are calculated including the sign bit. This may
  1700:   // add an extra bit for zero-extended values, e.g. (zext i32 to i64) may
  1701:   // result in 33 significant bits. To avoid extra words, skip the extra
  1702:   // sign bit, but keep information that the value is to be treated as
  1703:   // unsigned.
  1704:   KnownBits Known = HVC.getKnownBits(V, In);
  1705:   Signedness Sign = Signed;
  1706:   unsigned NumToTest = 0; // Number of bits used in test for unsignedness.
  1707:   if (isPowerOf2_32(Bits))
  1708:     NumToTest = Bits;
  1709:   else if (Bits > 1 && isPowerOf2_32(Bits - 1))
  1710:     NumToTest = Bits - 1;
  1711: 
  1712:   if (NumToTest != 0 && Known.Zero.ashr(NumToTest).isAllOnes()) {
  1713:     Sign = Unsigned;
  1714:     Bits = NumToTest;
  1715:   }
  1716: 
  1717:   // If the top bit of the nearest power-of-2 is zero, this value is
  1718:   // positive. It could be treated as either signed or unsigned.
  1719:   if (unsigned Pow2 = PowerOf2Ceil(Bits); Pow2 != Bits) {
  1720:     if (Known.Zero.ashr(Pow2 - 1).isAllOnes())
  1721:       Sign = Positive;
  1722:   }
  1723:   return {Bits, Sign};
  1724: }
  1725: 
  1726: auto HvxIdioms::canonSgn(SValue X, SValue Y) const
  1727:     -> std::pair<SValue, SValue> {
  1728:   // Canonicalize the signedness of X and Y, so that the result is one of:
  1729:   //   S, S
  1730:   //   U/P, S
  1731:   //   U/P, U/P
  1732:   if (X.Sgn == Signed && Y.Sgn != Signed)
  1733:     std::swap(X, Y);
  1734:   return {X, Y};
  1735: }
  1736: 
  1737: // Match
  1738: //   (X * Y) [>> N], or
  1739: //   ((X * Y) + (1 << M)) >> N
  1740: auto HvxIdioms::matchFxpMul(Instruction &In) const -> std::optional<FxpOp> {
  1741:   using namespace PatternMatch;
  1742:   auto *Ty = In.getType();
  1743: 
  1744:   if (!Ty->isVectorTy() || !Ty->getScalarType()->isIntegerTy())
  1745:     return std::nullopt;
  1746: 
  1747:   unsigned Width = cast<IntegerType>(Ty->getScalarType())->getBitWidth();
  1748: 
  1749:   FxpOp Op;
  1750:   Value *Exp = &In;
  1751: 
  1752:   // Fixed-point multiplication is always shifted right (except when the
  1753:   // fraction is 0 bits).
  1754:   auto m_Shr = [](auto &&V, auto &&S) {
  1755:     return m_CombineOr(m_LShr(V, S), m_AShr(V, S));
  1756:   };
  1757: 
  1758:   uint64_t Qn = 0;
  1759:   if (Value *T; match(Exp, m_Shr(m_Value(T), m_ConstantInt(Qn)))) {
  1760:     Op.Frac = Qn;
  1761:     Exp = T;
  1762:   } else {
  1763:     Op.Frac = 0;
  1764:   }
  1765: 
  1766:   if (Op.Frac > Width)
  1767:     return std::nullopt;
  1768: 
  1769:   // Check if there is rounding added.
  1770:   uint64_t CV;
  1771:   if (Value *T;
  1772:       Op.Frac > 0 && match(Exp, m_Add(m_Value(T), m_ConstantInt(CV)))) {
  1773:     if (CV != 0 && !isPowerOf2_64(CV))
  1774:       return std::nullopt;
  1775:     if (CV != 0)
  1776:       Op.RoundAt = Log2_64(CV);
  1777:     Exp = T;
  1778:   }
  1779: 
  1780:   // Check if the rest is a multiplication.
  1781:   if (match(Exp, m_Mul(m_Value(Op.X.Val), m_Value(Op.Y.Val)))) {
  1782:     Op.Opcode = Instruction::Mul;
  1783:     // FIXME: The information below is recomputed.
  1784:     Op.X.Sgn = getNumSignificantBits(Op.X.Val, &In).second;
  1785:     Op.Y.Sgn = getNumSignificantBits(Op.Y.Val, &In).second;
  1786:     Op.ResTy = cast<VectorType>(Ty);
  1787:     return Op;
  1788:   }
  1789: 
  1790:   return std::nullopt;
  1791: }
  1792: 
  1793: auto HvxIdioms::processFxpMul(Instruction &In, const FxpOp &Op) const
  1794:     -> Value * {
  1795:   assert(Op.X.Val->getType() == Op.Y.Val->getType());
  1796: 
  1797:   auto *VecTy = dyn_cast<VectorType>(Op.X.Val->getType());
  1798:   if (VecTy == nullptr)
  1799:     return nullptr;
  1800:   auto *ElemTy = cast<IntegerType>(VecTy->getElementType());
```
- EN: It opens namespaces (PatternMatch) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as realignStoreGroup, getType, CreateAnd, ConstantInt::get, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（PatternMatch），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 realignStoreGroup, getType, CreateAnd, ConstantInt::get, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:   unsigned ElemWidth = ElemTy->getBitWidth();
  1802: 
  1803:   // TODO: This can be relaxed after legalization is done pre-isel.
  1804:   if ((HVC.length(VecTy) * ElemWidth) % (8 * HVC.HST.getVectorLength()) != 0)
  1805:     return nullptr;
  1806: 
  1807:   // There are no special intrinsics that should be used for multiplying
  1808:   // signed 8-bit values, so just skip them. Normal codegen should handle
  1809:   // this just fine.
  1810:   if (ElemWidth <= 8)
  1811:     return nullptr;
  1812:   // Similarly, if this is just a multiplication that can be handled without
  1813:   // intervention, then leave it alone.
  1814:   if (ElemWidth <= 32 && Op.Frac == 0)
  1815:     return nullptr;
  1816: 
  1817:   auto [BitsX, SignX] = getNumSignificantBits(Op.X.Val, &In);
  1818:   auto [BitsY, SignY] = getNumSignificantBits(Op.Y.Val, &In);
  1819: 
  1820:   // TODO: Add multiplication of vectors by scalar registers (up to 4 bytes).
  1821: 
  1822:   Value *X = Op.X.Val, *Y = Op.Y.Val;
  1823:   IRBuilder Builder(In.getParent(), In.getIterator(),
  1824:                     InstSimplifyFolder(HVC.DL));
  1825: 
  1826:   auto roundUpWidth = [](unsigned Width) -> unsigned {
  1827:     if (Width <= 32 && !isPowerOf2_32(Width)) {
  1828:       // If the element width is not a power of 2, round it up
  1829:       // to the next one. Do this for widths not exceeding 32.
  1830:       return PowerOf2Ceil(Width);
  1831:     }
  1832:     if (Width > 32 && Width % 32 != 0) {
  1833:       // For wider elements, round it up to the multiple of 32.
  1834:       return alignTo(Width, 32u);
  1835:     }
  1836:     return Width;
  1837:   };
  1838: 
  1839:   BitsX = roundUpWidth(BitsX);
  1840:   BitsY = roundUpWidth(BitsY);
  1841: 
  1842:   // For elementwise multiplication vectors must have the same lengths, so
  1843:   // resize the elements of both inputs to the same width, the max of the
  1844:   // calculated significant bits.
  1845:   unsigned Width = std::max(BitsX, BitsY);
  1846: 
  1847:   auto *ResizeTy = VectorType::get(HVC.getIntTy(Width), VecTy);
  1848:   if (Width < ElemWidth) {
  1849:     X = Builder.CreateTrunc(X, ResizeTy, "trn");
  1850:     Y = Builder.CreateTrunc(Y, ResizeTy, "trn");
  1851:   } else if (Width > ElemWidth) {
  1852:     X = SignX == Signed ? Builder.CreateSExt(X, ResizeTy, "sxt")
  1853:                         : Builder.CreateZExt(X, ResizeTy, "zxt");
  1854:     Y = SignY == Signed ? Builder.CreateSExt(Y, ResizeTy, "sxt")
  1855:                         : Builder.CreateZExt(Y, ResizeTy, "zxt");
  1856:   };
  1857: 
  1858:   assert(X->getType() == Y->getType() && X->getType() == ResizeTy);
  1859: 
  1860:   unsigned VecLen = HVC.length(ResizeTy);
  1861:   unsigned ChopLen = (8 * HVC.HST.getVectorLength()) / std::min(Width, 32u);
  1862: 
  1863:   SmallVector<Value *> Results;
  1864:   FxpOp ChopOp = Op;
  1865:   ChopOp.ResTy = VectorType::get(Op.ResTy->getElementType(), ChopLen, false);
  1866: 
  1867:   for (unsigned V = 0; V != VecLen / ChopLen; ++V) {
  1868:     ChopOp.X.Val = HVC.subvector(Builder, X, V * ChopLen, ChopLen);
  1869:     ChopOp.Y.Val = HVC.subvector(Builder, Y, V * ChopLen, ChopLen);
  1870:     Results.push_back(processFxpMulChopped(Builder, In, ChopOp));
  1871:     if (Results.back() == nullptr)
  1872:       break;
  1873:   }
  1874: 
  1875:   if (Results.empty() || Results.back() == nullptr)
  1876:     return nullptr;
  1877: 
  1878:   Value *Cat = HVC.concat(Builder, Results);
  1879:   Value *Ext = SignX == Signed || SignY == Signed
  1880:                    ? Builder.CreateSExt(Cat, VecTy, "sxt")
  1881:                    : Builder.CreateZExt(Cat, VecTy, "zxt");
  1882:   return Ext;
  1883: }
  1884: 
  1885: inline bool HvxIdioms::matchScatter(Instruction &In) const {
  1886:   IntrinsicInst *II = dyn_cast<IntrinsicInst>(&In);
  1887:   if (!II)
  1888:     return false;
  1889:   return (II->getIntrinsicID() == Intrinsic::masked_scatter);
  1890: }
  1891: 
  1892: inline bool HvxIdioms::matchGather(Instruction &In) const {
  1893:   IntrinsicInst *II = dyn_cast<IntrinsicInst>(&In);
  1894:   if (!II)
  1895:     return false;
  1896:   return (II->getIntrinsicID() == Intrinsic::masked_gather);
  1897: }
  1898: 
  1899: inline bool HvxIdioms::matchMLoad(Instruction &In) const {
  1900:   IntrinsicInst *II = dyn_cast<IntrinsicInst>(&In);
  1901:   if (!II)
  1902:     return false;
  1903:   return (II->getIntrinsicID() == Intrinsic::masked_load);
  1904: }
  1905: 
  1906: inline bool HvxIdioms::matchMStore(Instruction &In) const {
  1907:   IntrinsicInst *II = dyn_cast<IntrinsicInst>(&In);
  1908:   if (!II)
  1909:     return false;
  1910:   return (II->getIntrinsicID() == Intrinsic::masked_store);
  1911: }
  1912: 
  1913: Instruction *locateDestination(Instruction *In, HvxIdioms::DstQualifier &Qual);
  1914: 
  1915: // Binary instructions we want to handle as users of gather/scatter.
  1916: inline bool isArithmetic(unsigned Opc) {
  1917:   switch (Opc) {
  1918:   case Instruction::Add:
  1919:   case Instruction::Sub:
  1920:   case Instruction::Mul:
  1921:   case Instruction::And:
  1922:   case Instruction::Or:
  1923:   case Instruction::Xor:
  1924:   case Instruction::AShr:
  1925:   case Instruction::LShr:
  1926:   case Instruction::Shl:
  1927:   case Instruction::UDiv:
  1928:     return true;
  1929:   }
  1930:   return false;
  1931: }
  1932: 
  1933: // TODO: Maybe use MemoryLocation for this. See getLocOrNone above.
  1934: inline Value *getPointer(Value *Ptr) {
  1935:   assert(Ptr && "Unable to extract pointer");
  1936:   if (isa<AllocaInst>(Ptr) || isa<Argument>(Ptr) || isa<GlobalValue>(Ptr))
  1937:     return Ptr;
  1938:   if (isa<LoadInst>(Ptr) || isa<StoreInst>(Ptr))
  1939:     return getLoadStorePointerOperand(Ptr);
  1940:   if (IntrinsicInst *II = dyn_cast<IntrinsicInst>(Ptr)) {
  1941:     if (II->getIntrinsicID() == Intrinsic::masked_store)
  1942:       return II->getOperand(1);
  1943:   }
  1944:   return nullptr;
  1945: }
  1946: 
  1947: static Instruction *selectDestination(Instruction *In,
  1948:                                       HvxIdioms::DstQualifier &Qual) {
  1949:   Instruction *Destination = nullptr;
  1950:   if (!In)
  1951:     return Destination;
  1952:   if (isa<StoreInst>(In)) {
  1953:     Destination = In;
  1954:     Qual = HvxIdioms::LdSt;
  1955:   } else if (IntrinsicInst *II = dyn_cast<IntrinsicInst>(In)) {
  1956:     if (II->getIntrinsicID() == Intrinsic::masked_gather) {
  1957:       Destination = In;
  1958:       Qual = HvxIdioms::LLVM_Gather;
  1959:     } else if (II->getIntrinsicID() == Intrinsic::masked_scatter) {
  1960:       Destination = In;
  1961:       Qual = HvxIdioms::LLVM_Scatter;
  1962:     } else if (II->getIntrinsicID() == Intrinsic::masked_store) {
  1963:       Destination = In;
  1964:       Qual = HvxIdioms::LdSt;
  1965:     } else if (II->getIntrinsicID() ==
  1966:                Intrinsic::hexagon_V6_vgather_vscattermh) {
  1967:       Destination = In;
  1968:       Qual = HvxIdioms::HEX_Gather_Scatter;
  1969:     } else if (II->getIntrinsicID() == Intrinsic::hexagon_V6_vscattermh_128B) {
  1970:       Destination = In;
  1971:       Qual = HvxIdioms::HEX_Scatter;
  1972:     } else if (II->getIntrinsicID() == Intrinsic::hexagon_V6_vgathermh_128B) {
  1973:       Destination = In;
  1974:       Qual = HvxIdioms::HEX_Gather;
  1975:     }
  1976:   } else if (isa<ZExtInst>(In)) {
  1977:     return locateDestination(In, Qual);
  1978:   } else if (isa<CastInst>(In)) {
  1979:     return locateDestination(In, Qual);
  1980:   } else if (isa<CallInst>(In)) {
  1981:     Destination = In;
  1982:     Qual = HvxIdioms::Call;
  1983:   } else if (isa<GetElementPtrInst>(In)) {
  1984:     return locateDestination(In, Qual);
  1985:   } else if (isArithmetic(In->getOpcode())) {
  1986:     Destination = In;
  1987:     Qual = HvxIdioms::Arithmetic;
  1988:   } else {
  1989:     LLVM_DEBUG(dbgs() << "Unhandled destination : " << *In << "\n");
  1990:   }
  1991:   return Destination;
  1992: }
  1993: 
  1994: // This method attempts to find destination (user) for a given intrinsic.
  1995: // Given that these are produced only by Ripple, the number of options is
  1996: // limited. Simplest case is explicit store which in fact is redundant (since
  1997: // HVX gater creates its own store during packetization). Nevertheless we need
  1998: // to figure address where we storing. Other cases are more complicated, but
  1999: // still few.
  2000: Instruction *locateDestination(Instruction *In, HvxIdioms::DstQualifier &Qual) {
```
- EN: It declares or implements routines such as getBitWidth, getNumSignificantBits, Builder, PowerOf2Ceil, ... (27 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 getBitWidth, getNumSignificantBits, Builder, PowerOf2Ceil, ... (27 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001:   Instruction *Destination = nullptr;
  2002:   if (!In)
  2003:     return Destination;
  2004:   // Get all possible destinations
  2005:   SmallVector<Instruction *> Users;
  2006:   // Iterate over the uses of the instruction
  2007:   for (auto &U : In->uses()) {
  2008:     if (auto *UI = dyn_cast<Instruction>(U.getUser())) {
  2009:       Destination = selectDestination(UI, Qual);
  2010:       if (Destination)
  2011:         Users.push_back(Destination);
  2012:     }
  2013:   }
  2014:   // Now see which of the users (if any) is a memory destination.
  2015:   for (auto *I : Users)
  2016:     if (getPointer(I))
  2017:       return I;
  2018:   return Destination;
  2019: }
  2020: 
  2021: // The two intrinsics we handle here have GEP in a different position.
  2022: inline GetElementPtrInst *locateGepFromIntrinsic(Instruction *In) {
  2023:   assert(In && "Bad instruction");
  2024:   IntrinsicInst *IIn = dyn_cast<IntrinsicInst>(In);
  2025:   assert((IIn && (IIn->getIntrinsicID() == Intrinsic::masked_gather ||
  2026:                   IIn->getIntrinsicID() == Intrinsic::masked_scatter)) &&
  2027:          "Not a gather Intrinsic");
  2028:   GetElementPtrInst *GEPIndex = nullptr;
  2029:   if (IIn->getIntrinsicID() == Intrinsic::masked_gather)
  2030:     GEPIndex = dyn_cast<GetElementPtrInst>(IIn->getOperand(0));
  2031:   else
  2032:     GEPIndex = dyn_cast<GetElementPtrInst>(IIn->getOperand(1));
  2033:   return GEPIndex;
  2034: }
  2035: 
  2036: // Given the intrinsic find its GEP argument and extract base address it uses.
  2037: // The method relies on the way how Ripple typically forms the GEP for
  2038: // scatter/gather.
  2039: static Value *locateAddressFromIntrinsic(Instruction *In) {
  2040:   GetElementPtrInst *GEPIndex = locateGepFromIntrinsic(In);
  2041:   if (!GEPIndex) {
  2042:     LLVM_DEBUG(dbgs() << "  No GEP in intrinsic\n");
  2043:     return nullptr;
  2044:   }
  2045:   Value *BaseAddress = GEPIndex->getPointerOperand();
  2046:   auto *IndexLoad = dyn_cast<LoadInst>(BaseAddress);
  2047:   if (IndexLoad)
  2048:     return IndexLoad;
  2049: 
  2050:   auto *IndexZEx = dyn_cast<ZExtInst>(BaseAddress);
  2051:   if (IndexZEx) {
  2052:     IndexLoad = dyn_cast<LoadInst>(IndexZEx->getOperand(0));
  2053:     if (IndexLoad)
  2054:       return IndexLoad;
  2055:     IntrinsicInst *II = dyn_cast<IntrinsicInst>(IndexZEx->getOperand(0));
  2056:     if (II && II->getIntrinsicID() == Intrinsic::masked_gather)
  2057:       return locateAddressFromIntrinsic(II);
  2058:   }
  2059:   auto *BaseShuffle = dyn_cast<ShuffleVectorInst>(BaseAddress);
  2060:   if (BaseShuffle) {
  2061:     IndexLoad = dyn_cast<LoadInst>(BaseShuffle->getOperand(0));
  2062:     if (IndexLoad)
  2063:       return IndexLoad;
  2064:     auto *IE = dyn_cast<InsertElementInst>(BaseShuffle->getOperand(0));
  2065:     if (IE) {
  2066:       auto *Src = IE->getOperand(1);
  2067:       IndexLoad = dyn_cast<LoadInst>(Src);
  2068:       if (IndexLoad)
  2069:         return IndexLoad;
  2070:       auto *Alloca = dyn_cast<AllocaInst>(Src);
  2071:       if (Alloca)
  2072:         return Alloca;
  2073:       if (isa<Argument>(Src)) {
  2074:         return Src;
  2075:       }
  2076:       if (isa<GlobalValue>(Src)) {
  2077:         return Src;
  2078:       }
  2079:     }
  2080:   }
  2081:   LLVM_DEBUG(dbgs() << "  Unable to locate Address from intrinsic\n");
  2082:   return nullptr;
  2083: }
  2084: 
  2085: static Type *getIndexType(Value *In) {
  2086:   if (!In)
  2087:     return nullptr;
  2088: 
  2089:   if (isa<LoadInst>(In) || isa<StoreInst>(In))
  2090:     return getLoadStoreType(In);
  2091: 
  2092:   if (IntrinsicInst *II = dyn_cast<IntrinsicInst>(In)) {
  2093:     if (II->getIntrinsicID() == Intrinsic::masked_load)
  2094:       return II->getType();
  2095:     if (II->getIntrinsicID() == Intrinsic::masked_store)
  2096:       return II->getOperand(0)->getType();
  2097:   }
  2098:   return In->getType();
  2099: }
  2100: 
  2101: static Value *locateIndexesFromGEP(Value *In) {
  2102:   if (!In)
  2103:     return nullptr;
  2104:   if (isa<LoadInst>(In))
  2105:     return In;
  2106:   if (IntrinsicInst *II = dyn_cast<IntrinsicInst>(In)) {
  2107:     if (II->getIntrinsicID() == Intrinsic::masked_load)
  2108:       return In;
  2109:     if (II->getIntrinsicID() == Intrinsic::masked_gather)
  2110:       return In;
  2111:   }
  2112:   if (auto *IndexZEx = dyn_cast<ZExtInst>(In))
  2113:     return locateIndexesFromGEP(IndexZEx->getOperand(0));
  2114:   if (auto *IndexSEx = dyn_cast<SExtInst>(In))
  2115:     return locateIndexesFromGEP(IndexSEx->getOperand(0));
  2116:   if (auto *BaseShuffle = dyn_cast<ShuffleVectorInst>(In))
  2117:     return locateIndexesFromGEP(BaseShuffle->getOperand(0));
  2118:   if (auto *IE = dyn_cast<InsertElementInst>(In))
  2119:     return locateIndexesFromGEP(IE->getOperand(1));
  2120:   if (auto *cstDataVector = dyn_cast<ConstantDataVector>(In))
  2121:     return cstDataVector;
  2122:   if (auto *GEPIndex = dyn_cast<GetElementPtrInst>(In))
  2123:     return GEPIndex->getOperand(0);
  2124:   return nullptr;
  2125: }
  2126: 
  2127: // Given the intrinsic find its GEP argument and extract offsetts from the base
  2128: // address it uses.
  2129: static Value *locateIndexesFromIntrinsic(Instruction *In) {
  2130:   GetElementPtrInst *GEPIndex = locateGepFromIntrinsic(In);
  2131:   if (!GEPIndex) {
  2132:     LLVM_DEBUG(dbgs() << "  No GEP in intrinsic\n");
  2133:     return nullptr;
  2134:   }
  2135:   Value *Indexes = GEPIndex->getOperand(1);
  2136:   if (auto *IndexLoad = locateIndexesFromGEP(Indexes))
  2137:     return IndexLoad;
  2138: 
  2139:   LLVM_DEBUG(dbgs() << "  Unable to locate Index from intrinsic\n");
  2140:   return nullptr;
  2141: }
  2142: 
  2143: // Because of aukward definition of many Hex intrinsics we often have to
  2144: // reinterprete HVX native <64 x i16> as <32 x i32> which in practice is a NOP
  2145: // for all use cases, so this only exist to make IR builder happy.
  2146: inline Value *getReinterpretiveCast_i16_to_i32(const HexagonVectorCombine &HVC,
  2147:                                                IRBuilderBase &Builder,
  2148:                                                LLVMContext &Ctx, Value *I) {
  2149:   assert(I && "Unable to reinterprete cast");
  2150:   Type *NT = HVC.getHvxTy(HVC.getIntTy(32), false);
  2151:   std::vector<unsigned> shuffleMask;
  2152:   for (unsigned i = 0; i < 64; ++i)
  2153:     shuffleMask.push_back(i);
  2154:   Constant *Mask = llvm::ConstantDataVector::get(Ctx, shuffleMask);
  2155:   Value *CastShuffle =
  2156:       Builder.CreateShuffleVector(I, I, Mask, "identity_shuffle");
  2157:   return Builder.CreateBitCast(CastShuffle, NT, "cst64_i16_to_32_i32");
  2158: }
  2159: 
  2160: // Recast <128 x i8> as <32 x i32>
  2161: inline Value *getReinterpretiveCast_i8_to_i32(const HexagonVectorCombine &HVC,
  2162:                                               IRBuilderBase &Builder,
  2163:                                               LLVMContext &Ctx, Value *I) {
  2164:   assert(I && "Unable to reinterprete cast");
  2165:   Type *NT = HVC.getHvxTy(HVC.getIntTy(32), false);
  2166:   std::vector<unsigned> shuffleMask;
  2167:   for (unsigned i = 0; i < 128; ++i)
  2168:     shuffleMask.push_back(i);
  2169:   Constant *Mask = llvm::ConstantDataVector::get(Ctx, shuffleMask);
  2170:   Value *CastShuffle =
  2171:       Builder.CreateShuffleVector(I, I, Mask, "identity_shuffle");
  2172:   return Builder.CreateBitCast(CastShuffle, NT, "cst128_i8_to_32_i32");
  2173: }
  2174: 
  2175: // Create <32 x i32> mask reinterpreted as <128 x i1> with a given pattern
  2176: inline Value *get_i32_Mask(const HexagonVectorCombine &HVC,
  2177:                            IRBuilderBase &Builder, LLVMContext &Ctx,
  2178:                            unsigned int pattern) {
  2179:   std::vector<unsigned int> byteMask;
  2180:   for (unsigned i = 0; i < 32; ++i)
  2181:     byteMask.push_back(pattern);
  2182: 
  2183:   return Builder.CreateIntrinsic(
  2184:       HVC.getBoolTy(128), HVC.HST.getIntrinsicId(Hexagon::V6_vandvrt),
  2185:       {llvm::ConstantDataVector::get(Ctx, byteMask), HVC.getConstInt(~0)},
  2186:       nullptr);
  2187: }
  2188: 
  2189: Value *HvxIdioms::processVScatter(Instruction &In) const {
  2190:   auto *InpTy = dyn_cast<VectorType>(In.getOperand(0)->getType());
  2191:   assert(InpTy && "Cannot handle no vector type for llvm.scatter/gather");
  2192:   unsigned InpSize = HVC.getSizeOf(InpTy);
  2193:   auto *F = In.getFunction();
  2194:   LLVMContext &Ctx = F->getContext();
  2195:   auto *ElemTy = dyn_cast<IntegerType>(InpTy->getElementType());
  2196:   assert(ElemTy && "llvm.scatter needs integer type argument");
  2197:   unsigned ElemWidth = HVC.DL.getTypeAllocSize(ElemTy);
  2198:   LLVM_DEBUG({
  2199:     unsigned Elements = HVC.length(InpTy);
  2200:     dbgs() << "\n[Process scatter](" << In << ")\n" << *In.getParent() << "\n";
```
- EN: It declares or implements routines such as selectDestination, locateGepFromIntrinsic, assert, dyn_cast<IntrinsicInst>, ... (34 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonVectorCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 selectDestination, locateGepFromIntrinsic, assert, dyn_cast<IntrinsicInst>, ... (34 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonVectorCombine，说明了它与同级后端组件的连接关系。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201:     dbgs() << "  Input type(" << *InpTy << ") elements(" << Elements
  2202:            << ") VecLen(" << InpSize << ") type(" << *ElemTy << ") ElemWidth("
  2203:            << ElemWidth << ")\n";
  2204:   });
  2205: 
  2206:   IRBuilder Builder(In.getParent(), In.getIterator(),
  2207:                     InstSimplifyFolder(HVC.DL));
  2208: 
  2209:   auto *ValueToScatter = In.getOperand(0);
  2210:   LLVM_DEBUG(dbgs() << "  ValueToScatter   : " << *ValueToScatter << "\n");
  2211: 
  2212:   if (HVC.HST.getVectorLength() != InpSize) {
  2213:     LLVM_DEBUG(dbgs() << "Unhandled vector size(" << InpSize
  2214:                       << ") for vscatter\n");
  2215:     return nullptr;
  2216:   }
  2217: 
  2218:   // Base address of indexes.
  2219:   auto *IndexLoad = locateAddressFromIntrinsic(&In);
  2220:   if (!IndexLoad)
  2221:     return nullptr;
  2222:   LLVM_DEBUG(dbgs() << "  IndexLoad        : " << *IndexLoad << "\n");
  2223: 
  2224:   // Address of destination. Must be in VTCM.
  2225:   auto *Ptr = getPointer(IndexLoad);
  2226:   if (!Ptr)
  2227:     return nullptr;
  2228:   LLVM_DEBUG(dbgs() << "  Ptr              : " << *Ptr << "\n");
  2229:   // Indexes/offsets
  2230:   auto *Indexes = locateIndexesFromIntrinsic(&In);
  2231:   if (!Indexes)
  2232:     return nullptr;
  2233:   LLVM_DEBUG(dbgs() << "  Indexes          : " << *Indexes << "\n");
  2234:   Value *CastedDst = Builder.CreateBitOrPointerCast(Ptr, Type::getInt32Ty(Ctx),
  2235:                                                     "cst_ptr_to_i32");
  2236:   LLVM_DEBUG(dbgs() << "  CastedDst        : " << *CastedDst << "\n");
  2237:   // Adjust Indexes
  2238:   auto *cstDataVector = dyn_cast<ConstantDataVector>(Indexes);
  2239:   Value *CastIndex = nullptr;
  2240:   if (cstDataVector) {
  2241:     // Our indexes are represented as a constant. We need it in a reg.
  2242:     Type *IndexVectorType = HVC.getHvxTy(HVC.getIntTy(32), false);
  2243:     AllocaInst *IndexesAlloca = Builder.CreateAlloca(IndexVectorType);
  2244:     [[maybe_unused]] auto *StoreIndexes =
  2245:         Builder.CreateStore(cstDataVector, IndexesAlloca);
  2246:     LLVM_DEBUG(dbgs() << "  StoreIndexes     : " << *StoreIndexes << "\n");
  2247:     CastIndex =
  2248:         Builder.CreateLoad(IndexVectorType, IndexesAlloca, "reload_index");
  2249:   } else {
  2250:     if (ElemWidth == 2)
  2251:       CastIndex = getReinterpretiveCast_i16_to_i32(HVC, Builder, Ctx, Indexes);
  2252:     else
  2253:       CastIndex = Indexes;
  2254:   }
  2255:   LLVM_DEBUG(dbgs() << "  Cast index       : " << *CastIndex << ")\n");
  2256: 
  2257:   if (ElemWidth == 1) {
  2258:     // v128i8 There is no native instruction for this.
  2259:     // Do this as two Hi/Lo gathers with masking.
  2260:     Type *NT = HVC.getHvxTy(HVC.getIntTy(32), false);
  2261:     // Extend indexes. We assume that indexes are in 128i8 format - need to
  2262:     // expand them to Hi/Lo 64i16
  2263:     Value *CastIndexes = Builder.CreateBitCast(CastIndex, NT, "cast_to_32i32");
  2264:     auto V6_vunpack = HVC.HST.getIntrinsicId(Hexagon::V6_vunpackub);
  2265:     auto *UnpackedIndexes = Builder.CreateIntrinsic(
  2266:         HVC.getHvxTy(HVC.getIntTy(32), true), V6_vunpack, CastIndexes, nullptr);
  2267:     LLVM_DEBUG(dbgs() << "  UnpackedIndexes  : " << *UnpackedIndexes << ")\n");
  2268: 
  2269:     auto V6_hi = HVC.HST.getIntrinsicId(Hexagon::V6_hi);
  2270:     auto V6_lo = HVC.HST.getIntrinsicId(Hexagon::V6_lo);
  2271:     [[maybe_unused]] Value *IndexHi =
  2272:         HVC.createHvxIntrinsic(Builder, V6_hi, NT, UnpackedIndexes);
  2273:     [[maybe_unused]] Value *IndexLo =
  2274:         HVC.createHvxIntrinsic(Builder, V6_lo, NT, UnpackedIndexes);
  2275:     LLVM_DEBUG(dbgs() << "  UnpackedIndHi    : " << *IndexHi << ")\n");
  2276:     LLVM_DEBUG(dbgs() << "  UnpackedIndLo    : " << *IndexLo << ")\n");
  2277:     // Now unpack values to scatter
  2278:     Value *CastSrc =
  2279:         getReinterpretiveCast_i8_to_i32(HVC, Builder, Ctx, ValueToScatter);
  2280:     LLVM_DEBUG(dbgs() << "  CastSrc          : " << *CastSrc << ")\n");
  2281:     auto *UnpackedValueToScatter = Builder.CreateIntrinsic(
  2282:         HVC.getHvxTy(HVC.getIntTy(32), true), V6_vunpack, CastSrc, nullptr);
  2283:     LLVM_DEBUG(dbgs() << "  UnpackedValToScat: " << *UnpackedValueToScatter
  2284:                       << ")\n");
  2285: 
  2286:     [[maybe_unused]] Value *UVSHi =
  2287:         HVC.createHvxIntrinsic(Builder, V6_hi, NT, UnpackedValueToScatter);
  2288:     [[maybe_unused]] Value *UVSLo =
  2289:         HVC.createHvxIntrinsic(Builder, V6_lo, NT, UnpackedValueToScatter);
  2290:     LLVM_DEBUG(dbgs() << "  UVSHi            : " << *UVSHi << ")\n");
  2291:     LLVM_DEBUG(dbgs() << "  UVSLo            : " << *UVSLo << ")\n");
  2292: 
  2293:     // Create the mask for individual bytes
  2294:     auto *QByteMask = get_i32_Mask(HVC, Builder, Ctx, 0x00ff00ff);
  2295:     LLVM_DEBUG(dbgs() << "  QByteMask        : " << *QByteMask << "\n");
  2296:     [[maybe_unused]] auto *ResHi = Builder.CreateIntrinsic(
  2297:         Type::getVoidTy(Ctx), Intrinsic::hexagon_V6_vscattermhq_128B,
  2298:         {QByteMask, CastedDst, HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE),
  2299:          IndexHi, UVSHi},
  2300:         nullptr);
  2301:     LLVM_DEBUG(dbgs() << "  ResHi            : " << *ResHi << ")\n");
  2302:     return Builder.CreateIntrinsic(
  2303:         Type::getVoidTy(Ctx), Intrinsic::hexagon_V6_vscattermhq_128B,
  2304:         {QByteMask, CastedDst, HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE),
  2305:          IndexLo, UVSLo},
  2306:         nullptr);
  2307:   } else if (ElemWidth == 2) {
  2308:     Value *CastSrc =
  2309:         getReinterpretiveCast_i16_to_i32(HVC, Builder, Ctx, ValueToScatter);
  2310:     LLVM_DEBUG(dbgs() << "  CastSrc        : " << *CastSrc << ")\n");
  2311:     return Builder.CreateIntrinsic(
  2312:         Type::getVoidTy(Ctx), Intrinsic::hexagon_V6_vscattermh_128B,
  2313:         {CastedDst, HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE), CastIndex,
  2314:          CastSrc},
  2315:         nullptr);
  2316:   } else if (ElemWidth == 4) {
  2317:     return Builder.CreateIntrinsic(
  2318:         Type::getVoidTy(Ctx), Intrinsic::hexagon_V6_vscattermw_128B,
  2319:         {CastedDst, HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE), CastIndex,
  2320:          ValueToScatter},
  2321:         nullptr);
  2322:   } else {
  2323:     LLVM_DEBUG(dbgs() << "Unhandled element type for vscatter\n");
  2324:     return nullptr;
  2325:   }
  2326: }
  2327: 
  2328: Value *HvxIdioms::processVGather(Instruction &In) const {
  2329:   [[maybe_unused]] auto *InpTy =
  2330:       dyn_cast<VectorType>(In.getOperand(0)->getType());
  2331:   assert(InpTy && "Cannot handle no vector type for llvm.gather");
  2332:   [[maybe_unused]] auto *ElemTy =
  2333:       dyn_cast<PointerType>(InpTy->getElementType());
  2334:   assert(ElemTy && "llvm.gather needs vector of ptr argument");
  2335:   auto *F = In.getFunction();
  2336:   LLVMContext &Ctx = F->getContext();
  2337:   LLVM_DEBUG(dbgs() << "\n[Process gather](" << In << ")\n"
  2338:                     << *In.getParent() << "\n");
  2339:   LLVM_DEBUG(dbgs() << "  Input type(" << *InpTy << ") elements("
  2340:                     << HVC.length(InpTy) << ") VecLen(" << HVC.getSizeOf(InpTy)
  2341:                     << ") type(" << *ElemTy << ") Access alignment("
  2342:                     << *In.getOperand(1) << ") AddressSpace("
  2343:                     << ElemTy->getAddressSpace() << ")\n");
  2344: 
  2345:   // TODO: Handle masking of elements.
  2346:   assert(dyn_cast<VectorType>(In.getOperand(2)->getType()) &&
  2347:          "llvm.gather needs vector for mask");
  2348:   IRBuilder Builder(In.getParent(), In.getIterator(),
  2349:                     InstSimplifyFolder(HVC.DL));
  2350: 
  2351:   // See who is using the result. The difference between LLVM and HVX vgather
  2352:   // Intrinsic makes it impossible to handle all cases with temp storage. Alloca
  2353:   // in VTCM is not yet supported, so for now we just bail out for those cases.
  2354:   HvxIdioms::DstQualifier Qual = HvxIdioms::Undefined;
  2355:   Instruction *Dst = locateDestination(&In, Qual);
  2356:   if (!Dst) {
  2357:     LLVM_DEBUG(dbgs() << "  Unable to locate vgather destination\n");
  2358:     return nullptr;
  2359:   }
  2360:   LLVM_DEBUG(dbgs() << "  Destination    : " << *Dst << " Qual(" << Qual
  2361:                     << ")\n");
  2362: 
  2363:   // Address of destination. Must be in VTCM.
  2364:   auto *Ptr = getPointer(Dst);
  2365:   if (!Ptr) {
  2366:     LLVM_DEBUG(dbgs() << "Could not locate vgather destination ptr\n");
  2367:     return nullptr;
  2368:   }
  2369: 
  2370:   // Result type. Assume it is a vector type.
  2371:   auto *DstType = cast<VectorType>(getIndexType(Dst));
  2372:   assert(DstType && "Cannot handle non vector dst type for llvm.gather");
  2373: 
  2374:   // Base address for sources to be loaded
  2375:   auto *IndexLoad = locateAddressFromIntrinsic(&In);
  2376:   if (!IndexLoad)
  2377:     return nullptr;
  2378:   LLVM_DEBUG(dbgs() << "  IndexLoad      : " << *IndexLoad << "\n");
  2379: 
  2380:   // Gather indexes/offsets
  2381:   auto *Indexes = locateIndexesFromIntrinsic(&In);
  2382:   if (!Indexes)
  2383:     return nullptr;
  2384:   LLVM_DEBUG(dbgs() << "  Indexes        : " << *Indexes << "\n");
  2385: 
  2386:   Instruction *Gather = nullptr;
  2387:   Type *NT = HVC.getHvxTy(HVC.getIntTy(32), false);
  2388:   if (Qual == HvxIdioms::LdSt || Qual == HvxIdioms::Arithmetic) {
  2389:     // We fully assume the address space is in VTCM. We also assume that all
  2390:     // pointers in Operand(0) have the same base(!).
  2391:     // This is the most basic case of all the above.
  2392:     unsigned OutputSize = HVC.getSizeOf(DstType);
  2393:     auto *DstElemTy = cast<IntegerType>(DstType->getElementType());
  2394:     unsigned ElemWidth = HVC.DL.getTypeAllocSize(DstElemTy);
  2395:     LLVM_DEBUG(dbgs() << "  Buffer type    : " << *Ptr->getType()
  2396:                       << "  Address space ("
  2397:                       << Ptr->getType()->getPointerAddressSpace() << ")\n"
  2398:                       << "  Result type    : " << *DstType
  2399:                       << "\n  Size in bytes  : " << OutputSize
  2400:                       << " element type(" << *DstElemTy
```
- EN: It declares or implements routines such as Builder, getOperand, LLVM_DEBUG, locateAddressFromIntrinsic, ... (30 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 Builder, getOperand, LLVM_DEBUG, locateAddressFromIntrinsic, ... (30 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2401-2600 / 第 2401-2600 行

```cpp
  2401:                       << ")\n  ElemWidth      : " << ElemWidth << " bytes\n");
  2402: 
  2403:     auto *IndexType = cast<VectorType>(getIndexType(Indexes));
  2404:     assert(IndexType && "Cannot handle non vector index type for llvm.gather");
  2405:     unsigned IndexWidth = HVC.DL.getTypeAllocSize(IndexType->getElementType());
  2406:     LLVM_DEBUG(dbgs() << "  IndexWidth(" << IndexWidth << ")\n");
  2407: 
  2408:     // Intrinsic takes i32 instead of pointer so cast.
  2409:     Value *CastedPtr = Builder.CreateBitOrPointerCast(
  2410:         IndexLoad, Type::getInt32Ty(Ctx), "cst_ptr_to_i32");
  2411:     // [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, ...]
  2412:     // int_hexagon_V6_vgathermh       [... , llvm_v16i32_ty]
  2413:     // int_hexagon_V6_vgathermh_128B  [... , llvm_v32i32_ty]
  2414:     // int_hexagon_V6_vgathermhw      [... , llvm_v32i32_ty]
  2415:     // int_hexagon_V6_vgathermhw_128B [... , llvm_v64i32_ty]
  2416:     // int_hexagon_V6_vgathermw       [... , llvm_v16i32_ty]
  2417:     // int_hexagon_V6_vgathermw_128B  [... , llvm_v32i32_ty]
  2418:     if (HVC.HST.getVectorLength() == OutputSize) {
  2419:       if (ElemWidth == 1) {
  2420:         // v128i8 There is no native instruction for this.
  2421:         // Do this as two Hi/Lo gathers with masking.
  2422:         // Unpack indexes. We assume that indexes are in 128i8 format - need to
  2423:         // expand them to Hi/Lo 64i16
  2424:         Value *CastIndexes =
  2425:             Builder.CreateBitCast(Indexes, NT, "cast_to_32i32");
  2426:         auto V6_vunpack = HVC.HST.getIntrinsicId(Hexagon::V6_vunpackub);
  2427:         auto *UnpackedIndexes =
  2428:             Builder.CreateIntrinsic(HVC.getHvxTy(HVC.getIntTy(32), true),
  2429:                                     V6_vunpack, CastIndexes, nullptr);
  2430:         LLVM_DEBUG(dbgs() << "  UnpackedIndexes : " << *UnpackedIndexes
  2431:                           << ")\n");
  2432: 
  2433:         auto V6_hi = HVC.HST.getIntrinsicId(Hexagon::V6_hi);
  2434:         auto V6_lo = HVC.HST.getIntrinsicId(Hexagon::V6_lo);
  2435:         [[maybe_unused]] Value *IndexHi =
  2436:             HVC.createHvxIntrinsic(Builder, V6_hi, NT, UnpackedIndexes);
  2437:         [[maybe_unused]] Value *IndexLo =
  2438:             HVC.createHvxIntrinsic(Builder, V6_lo, NT, UnpackedIndexes);
  2439:         LLVM_DEBUG(dbgs() << "  UnpackedIndHi   : " << *IndexHi << ")\n");
  2440:         LLVM_DEBUG(dbgs() << "  UnpackedIndLo   : " << *IndexLo << ")\n");
  2441:         // Create the mask for individual bytes
  2442:         auto *QByteMask = get_i32_Mask(HVC, Builder, Ctx, 0x00ff00ff);
  2443:         LLVM_DEBUG(dbgs() << "  QByteMask       : " << *QByteMask << "\n");
  2444:         // We use our destination allocation as a temp storage
  2445:         // This is unlikely to work properly for masked gather.
  2446:         auto V6_vgather = HVC.HST.getIntrinsicId(Hexagon::V6_vgathermhq);
  2447:         [[maybe_unused]] auto GatherHi = Builder.CreateIntrinsic(
  2448:             Type::getVoidTy(Ctx), V6_vgather,
  2449:             {Ptr, QByteMask, CastedPtr,
  2450:              HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE), IndexHi},
  2451:             nullptr);
  2452:         LLVM_DEBUG(dbgs() << "  GatherHi        : " << *GatherHi << ")\n");
  2453:         // Rematerialize the result
  2454:         [[maybe_unused]] Value *LoadedResultHi = Builder.CreateLoad(
  2455:             HVC.getHvxTy(HVC.getIntTy(32), false), Ptr, "temp_result_hi");
  2456:         LLVM_DEBUG(dbgs() << "  LoadedResultHi : " << *LoadedResultHi << "\n");
  2457:         // Same for the low part. Here we use Gather to return non-NULL result
  2458:         // from this function and continue to iterate. We also are deleting Dst
  2459:         // store below.
  2460:         Gather = Builder.CreateIntrinsic(
  2461:             Type::getVoidTy(Ctx), V6_vgather,
  2462:             {Ptr, QByteMask, CastedPtr,
  2463:              HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE), IndexLo},
  2464:             nullptr);
  2465:         LLVM_DEBUG(dbgs() << "  GatherLo        : " << *Gather << ")\n");
  2466:         Value *LoadedResultLo = Builder.CreateLoad(
  2467:             HVC.getHvxTy(HVC.getIntTy(32), false), Ptr, "temp_result_lo");
  2468:         LLVM_DEBUG(dbgs() << "  LoadedResultLo : " << *LoadedResultLo << "\n");
  2469:         // Now we have properly sized bytes in every other position
  2470:         // B b A a c a A b B c f F g G h H is presented as
  2471:         // B . b . A . a . c . a . A . b . B . c . f . F . g . G . h . H
  2472:         // Use vpack to gather them
  2473:         auto V6_vpackeb = HVC.HST.getIntrinsicId(Hexagon::V6_vpackeb);
  2474:         [[maybe_unused]] auto Res = Builder.CreateIntrinsic(
  2475:             NT, V6_vpackeb, {LoadedResultHi, LoadedResultLo}, nullptr);
  2476:         LLVM_DEBUG(dbgs() << "  ScaledRes      : " << *Res << "\n");
  2477:         [[maybe_unused]] auto *StoreRes = Builder.CreateStore(Res, Ptr);
  2478:         LLVM_DEBUG(dbgs() << "  StoreRes       : " << *StoreRes << "\n");
  2479:       } else if (ElemWidth == 2) {
  2480:         // v32i16
  2481:         if (IndexWidth == 2) {
  2482:           // Reinterprete 64i16 as 32i32. Only needed for syntactic IR match.
  2483:           Value *CastIndex =
  2484:               getReinterpretiveCast_i16_to_i32(HVC, Builder, Ctx, Indexes);
  2485:           LLVM_DEBUG(dbgs() << "  Cast index: " << *CastIndex << ")\n");
  2486:           // shift all i16 left by 1 to match short addressing mode instead of
  2487:           // byte.
  2488:           auto V6_vaslh = HVC.HST.getIntrinsicId(Hexagon::V6_vaslh);
  2489:           Value *AdjustedIndex = HVC.createHvxIntrinsic(
  2490:               Builder, V6_vaslh, NT, {CastIndex, HVC.getConstInt(1)});
  2491:           LLVM_DEBUG(dbgs()
  2492:                      << "  Shifted half index: " << *AdjustedIndex << ")\n");
  2493: 
  2494:           auto V6_vgather = HVC.HST.getIntrinsicId(Hexagon::V6_vgathermh);
  2495:           // The 3rd argument is the size of the region to gather from. Probably
  2496:           // want to set it to max VTCM size.
  2497:           Gather = Builder.CreateIntrinsic(
  2498:               Type::getVoidTy(Ctx), V6_vgather,
  2499:               {Ptr, CastedPtr, HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE),
  2500:                AdjustedIndex},
  2501:               nullptr);
  2502:           for (auto &U : Dst->uses()) {
  2503:             if (auto *UI = dyn_cast<Instruction>(U.getUser()))
  2504:               dbgs() << "    dst used by: " << *UI << "\n";
  2505:           }
  2506:           for (auto &U : In.uses()) {
  2507:             if (auto *UI = dyn_cast<Instruction>(U.getUser()))
  2508:               dbgs() << "    In used by : " << *UI << "\n";
  2509:           }
  2510:           // Create temp load from result in case the result is used by any
  2511:           // other instruction.
  2512:           Value *LoadedResult = Builder.CreateLoad(
  2513:               HVC.getHvxTy(HVC.getIntTy(16), false), Ptr, "temp_result");
  2514:           LLVM_DEBUG(dbgs() << "  LoadedResult   : " << *LoadedResult << "\n");
  2515:           In.replaceAllUsesWith(LoadedResult);
  2516:         } else {
  2517:           dbgs() << "    Unhandled index type for vgather\n";
  2518:           return nullptr;
  2519:         }
  2520:       } else if (ElemWidth == 4) {
  2521:         if (IndexWidth == 4) {
  2522:           // v32i32
  2523:           auto V6_vaslh = HVC.HST.getIntrinsicId(Hexagon::V6_vaslh);
  2524:           Value *AdjustedIndex = HVC.createHvxIntrinsic(
  2525:               Builder, V6_vaslh, NT, {Indexes, HVC.getConstInt(2)});
  2526:           LLVM_DEBUG(dbgs()
  2527:                      << "  Shifted word index: " << *AdjustedIndex << ")\n");
  2528:           Gather = Builder.CreateIntrinsic(
  2529:               Type::getVoidTy(Ctx), Intrinsic::hexagon_V6_vgathermw_128B,
  2530:               {Ptr, CastedPtr, HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE),
  2531:                AdjustedIndex},
  2532:               nullptr);
  2533:         } else {
  2534:           LLVM_DEBUG(dbgs() << "    Unhandled index type for vgather\n");
  2535:           return nullptr;
  2536:         }
  2537:       } else {
  2538:         LLVM_DEBUG(dbgs() << "    Unhandled element type for vgather\n");
  2539:         return nullptr;
  2540:       }
  2541:     } else if (HVC.HST.getVectorLength() == OutputSize * 2) {
  2542:       // This is half of the reg width, duplicate low in high
  2543:       LLVM_DEBUG(dbgs() << "    Unhandled half of register size\n");
  2544:       return nullptr;
  2545:     } else if (HVC.HST.getVectorLength() * 2 == OutputSize) {
  2546:       LLVM_DEBUG(dbgs() << "    Unhandle twice the register size\n");
  2547:       return nullptr;
  2548:     }
  2549:     // Erase the original intrinsic and store that consumes it.
  2550:     // HVX will create a pseudo for gather that is expanded to gather + store
  2551:     // during packetization.
  2552:     Dst->eraseFromParent();
  2553:   } else if (Qual == HvxIdioms::LLVM_Scatter) {
  2554:     // Gather feeds directly into scatter.
  2555:     LLVM_DEBUG({
  2556:       auto *DstInpTy = cast<VectorType>(Dst->getOperand(1)->getType());
  2557:       assert(DstInpTy && "Cannot handle no vector type for llvm.scatter");
  2558:       unsigned DstInpSize = HVC.getSizeOf(DstInpTy);
  2559:       unsigned DstElements = HVC.length(DstInpTy);
  2560:       auto *DstElemTy = cast<PointerType>(DstInpTy->getElementType());
  2561:       assert(DstElemTy && "llvm.scatter needs vector of ptr argument");
  2562:       dbgs() << "  Gather feeds into scatter\n  Values to scatter : "
  2563:              << *Dst->getOperand(0) << "\n";
  2564:       dbgs() << "  Dst type(" << *DstInpTy << ") elements(" << DstElements
  2565:              << ") VecLen(" << DstInpSize << ") type(" << *DstElemTy
  2566:              << ") Access alignment(" << *Dst->getOperand(2) << ")\n";
  2567:     });
  2568:     // Address of source
  2569:     auto *Src = getPointer(IndexLoad);
  2570:     if (!Src)
  2571:       return nullptr;
  2572:     LLVM_DEBUG(dbgs() << "  Src            : " << *Src << "\n");
  2573: 
  2574:     if (!isa<PointerType>(Src->getType())) {
  2575:       LLVM_DEBUG(dbgs() << "    Source is not a pointer type...\n");
  2576:       return nullptr;
  2577:     }
  2578: 
  2579:     Value *CastedSrc = Builder.CreateBitOrPointerCast(
  2580:         Src, Type::getInt32Ty(Ctx), "cst_ptr_to_i32");
  2581:     LLVM_DEBUG(dbgs() << "  CastedSrc: " << *CastedSrc << "\n");
  2582: 
  2583:     auto *DstLoad = locateAddressFromIntrinsic(Dst);
  2584:     if (!DstLoad) {
  2585:       LLVM_DEBUG(dbgs() << "  Unable to locate DstLoad\n");
  2586:       return nullptr;
  2587:     }
  2588:     LLVM_DEBUG(dbgs() << "  DstLoad  : " << *DstLoad << "\n");
  2589: 
  2590:     Value *Ptr = getPointer(DstLoad);
  2591:     if (!Ptr)
  2592:       return nullptr;
  2593:     LLVM_DEBUG(dbgs() << "  Ptr      : " << *Ptr << "\n");
  2594:     Value *CastIndex =
  2595:         getReinterpretiveCast_i16_to_i32(HVC, Builder, Ctx, IndexLoad);
  2596:     LLVM_DEBUG(dbgs() << "  Cast index: " << *CastIndex << ")\n");
  2597:     // Shift all i16 left by 1 to match short addressing mode instead of
  2598:     // byte.
  2599:     auto V6_vaslh = HVC.HST.getIntrinsicId(Hexagon::V6_vaslh);
  2600:     Value *AdjustedIndex = HVC.createHvxIntrinsic(
```
- EN: It declares or implements routines such as cast<VectorType>, assert, getTypeAllocSize, LLVM_DEBUG, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 cast<VectorType>, assert, getTypeAllocSize, LLVM_DEBUG, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2601-2800 / 第 2601-2800 行

```cpp
  2601:         Builder, V6_vaslh, NT, {CastIndex, HVC.getConstInt(1)});
  2602:     LLVM_DEBUG(dbgs() << "  Shifted half index: " << *AdjustedIndex << ")\n");
  2603: 
  2604:     return Builder.CreateIntrinsic(
  2605:         Type::getVoidTy(Ctx), Intrinsic::hexagon_V6_vgathermh_128B,
  2606:         {Ptr, CastedSrc, HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE),
  2607:          AdjustedIndex},
  2608:         nullptr);
  2609:   } else if (Qual == HvxIdioms::HEX_Gather_Scatter) {
  2610:     // Gather feeds into previously inserted pseudo intrinsic.
  2611:     // These could not be in the same packet, so we need to generate another
  2612:     // pseudo that is expanded to .tmp + store V6_vgathermh_pseudo
  2613:     // V6_vgathermh_pseudo (ins IntRegs:$_dst_, s4_0Imm:$Ii, IntRegs:$Rt,
  2614:     // ModRegs:$Mu, HvxVR:$Vv)
  2615:     if (isa<AllocaInst>(IndexLoad)) {
  2616:       auto *cstDataVector = dyn_cast<ConstantDataVector>(Indexes);
  2617:       if (cstDataVector) {
  2618:         // Our indexes are represented as a constant. We need THEM in a reg.
  2619:         // This most likely will not work properly since alloca gives us DDR
  2620:         // stack location. This will be fixed once we teach compiler about VTCM.
  2621:         AllocaInst *IndexesAlloca = Builder.CreateAlloca(NT);
  2622:         [[maybe_unused]] auto *StoreIndexes =
  2623:             Builder.CreateStore(cstDataVector, IndexesAlloca);
  2624:         LLVM_DEBUG(dbgs() << "  StoreIndexes   : " << *StoreIndexes << "\n");
  2625:         Value *LoadedIndex =
  2626:             Builder.CreateLoad(NT, IndexesAlloca, "reload_index");
  2627:         AllocaInst *ResultAlloca = Builder.CreateAlloca(NT);
  2628:         LLVM_DEBUG(dbgs() << "  ResultAlloca   : " << *ResultAlloca << "\n");
  2629: 
  2630:         Value *CastedSrc = Builder.CreateBitOrPointerCast(
  2631:             IndexLoad, Type::getInt32Ty(Ctx), "cst_ptr_to_i32");
  2632:         LLVM_DEBUG(dbgs() << "  CastedSrc      : " << *CastedSrc << "\n");
  2633: 
  2634:         Gather = Builder.CreateIntrinsic(
  2635:             Type::getVoidTy(Ctx), Intrinsic::hexagon_V6_vgathermh_128B,
  2636:             {ResultAlloca, CastedSrc,
  2637:              HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE), LoadedIndex},
  2638:             nullptr);
  2639:         Value *LoadedResult = Builder.CreateLoad(
  2640:             HVC.getHvxTy(HVC.getIntTy(16), false), ResultAlloca, "temp_result");
  2641:         LLVM_DEBUG(dbgs() << "  LoadedResult   : " << *LoadedResult << "\n");
  2642:         LLVM_DEBUG(dbgs() << "  Gather         : " << *Gather << "\n");
  2643:         In.replaceAllUsesWith(LoadedResult);
  2644:       }
  2645:     } else {
  2646:       // Address of source
  2647:       auto *Src = getPointer(IndexLoad);
  2648:       if (!Src)
  2649:         return nullptr;
  2650:       LLVM_DEBUG(dbgs() << "  Src      : " << *Src << "\n");
  2651: 
  2652:       Value *CastedSrc = Builder.CreateBitOrPointerCast(
  2653:           Src, Type::getInt32Ty(Ctx), "cst_ptr_to_i32");
  2654:       LLVM_DEBUG(dbgs() << "  CastedSrc: " << *CastedSrc << "\n");
  2655: 
  2656:       auto *DstLoad = locateAddressFromIntrinsic(Dst);
  2657:       if (!DstLoad)
  2658:         return nullptr;
  2659:       LLVM_DEBUG(dbgs() << "  DstLoad  : " << *DstLoad << "\n");
  2660:       auto *Ptr = getPointer(DstLoad);
  2661:       if (!Ptr)
  2662:         return nullptr;
  2663:       LLVM_DEBUG(dbgs() << "  Ptr      : " << *Ptr << "\n");
  2664: 
  2665:       Gather = Builder.CreateIntrinsic(
  2666:           Type::getVoidTy(Ctx), Intrinsic::hexagon_V6_vgather_vscattermh,
  2667:           {Ptr, CastedSrc, HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE),
  2668:            Indexes},
  2669:           nullptr);
  2670:     }
  2671:     return Gather;
  2672:   } else if (Qual == HvxIdioms::HEX_Scatter) {
  2673:     // This is the case when result of a gather is used as an argument to
  2674:     // Intrinsic::hexagon_V6_vscattermh_128B. Most likely we just inserted it
  2675:     // ourselves. We have to create alloca, store to it, and replace all uses
  2676:     // with that.
  2677:     AllocaInst *ResultAlloca = Builder.CreateAlloca(NT);
  2678:     Value *CastedSrc = Builder.CreateBitOrPointerCast(
  2679:         IndexLoad, Type::getInt32Ty(Ctx), "cst_ptr_to_i32");
  2680:     LLVM_DEBUG(dbgs() << "  CastedSrc      : " << *CastedSrc << "\n");
  2681:     Value *CastIndex =
  2682:         getReinterpretiveCast_i16_to_i32(HVC, Builder, Ctx, Indexes);
  2683:     LLVM_DEBUG(dbgs() << "  Cast index     : " << *CastIndex << ")\n");
  2684: 
  2685:     Gather = Builder.CreateIntrinsic(
  2686:         Type::getVoidTy(Ctx), Intrinsic::hexagon_V6_vgathermh_128B,
  2687:         {ResultAlloca, CastedSrc, HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE),
  2688:          CastIndex},
  2689:         nullptr);
  2690:     Value *LoadedResult = Builder.CreateLoad(
  2691:         HVC.getHvxTy(HVC.getIntTy(16), false), ResultAlloca, "temp_result");
  2692:     LLVM_DEBUG(dbgs() << "  LoadedResult   : " << *LoadedResult << "\n");
  2693:     In.replaceAllUsesWith(LoadedResult);
  2694:   } else if (Qual == HvxIdioms::HEX_Gather) {
  2695:     // Gather feeds to another gather but already replaced with
  2696:     // hexagon_V6_vgathermh_128B
  2697:     if (isa<AllocaInst>(IndexLoad)) {
  2698:       auto *cstDataVector = dyn_cast<ConstantDataVector>(Indexes);
  2699:       if (cstDataVector) {
  2700:         // Our indexes are represented as a constant. We need it in a reg.
  2701:         AllocaInst *IndexesAlloca = Builder.CreateAlloca(NT);
  2702: 
  2703:         [[maybe_unused]] auto *StoreIndexes =
  2704:             Builder.CreateStore(cstDataVector, IndexesAlloca);
  2705:         LLVM_DEBUG(dbgs() << "  StoreIndexes   : " << *StoreIndexes << "\n");
  2706:         Value *LoadedIndex =
  2707:             Builder.CreateLoad(NT, IndexesAlloca, "reload_index");
  2708:         AllocaInst *ResultAlloca = Builder.CreateAlloca(NT);
  2709:         LLVM_DEBUG(dbgs() << "  ResultAlloca   : " << *ResultAlloca
  2710:                           << "\n  AddressSpace: "
  2711:                           << ResultAlloca->getAddressSpace() << "\n";);
  2712: 
  2713:         Value *CastedSrc = Builder.CreateBitOrPointerCast(
  2714:             IndexLoad, Type::getInt32Ty(Ctx), "cst_ptr_to_i32");
  2715:         LLVM_DEBUG(dbgs() << "  CastedSrc      : " << *CastedSrc << "\n");
  2716: 
  2717:         Gather = Builder.CreateIntrinsic(
  2718:             Type::getVoidTy(Ctx), Intrinsic::hexagon_V6_vgathermh_128B,
  2719:             {ResultAlloca, CastedSrc,
  2720:              HVC.getConstInt(DEFAULT_HVX_VTCM_PAGE_SIZE), LoadedIndex},
  2721:             nullptr);
  2722:         Value *LoadedResult = Builder.CreateLoad(
  2723:             HVC.getHvxTy(HVC.getIntTy(16), false), ResultAlloca, "temp_result");
  2724:         LLVM_DEBUG(dbgs() << "  LoadedResult   : " << *LoadedResult << "\n");
  2725:         LLVM_DEBUG(dbgs() << "  Gather         : " << *Gather << "\n");
  2726:         In.replaceAllUsesWith(LoadedResult);
  2727:       }
  2728:     }
  2729:   } else if (Qual == HvxIdioms::LLVM_Gather) {
  2730:     // Gather feeds into another gather
  2731:     errs() << " Underimplemented vgather to vgather sequence\n";
  2732:     return nullptr;
  2733:   } else
  2734:     llvm_unreachable("Unhandled Qual enum");
  2735: 
  2736:   return Gather;
  2737: }
  2738: 
  2739: // Go through all PHI incomming values and find minimal alignment for non GEP
  2740: // members.
  2741: std::optional<uint64_t> HvxIdioms::getPHIBaseMinAlignment(Instruction &In,
  2742:                                                           PHINode *PN) const {
  2743:   if (!PN)
  2744:     return std::nullopt;
  2745: 
  2746:   SmallVector<Value *, 16> Worklist;
  2747:   SmallPtrSet<Value *, 16> Visited;
  2748:   uint64_t minPHIAlignment = Value::MaximumAlignment;
  2749:   Worklist.push_back(PN);
  2750: 
  2751:   while (!Worklist.empty()) {
  2752:     Value *V = Worklist.back();
  2753:     Worklist.pop_back();
  2754:     if (!Visited.insert(V).second)
  2755:       continue;
  2756: 
  2757:     if (PHINode *PN = dyn_cast<PHINode>(V)) {
  2758:       for (unsigned i = 0; i < PN->getNumIncomingValues(); ++i) {
  2759:         Worklist.push_back(PN->getIncomingValue(i));
  2760:       }
  2761:     } else if (isa<GetElementPtrInst>(V)) {
  2762:       // Ignore geps for now.
  2763:       continue;
  2764:     } else {
  2765:       Align KnownAlign = getKnownAlignment(V, HVC.DL, &In, &HVC.AC, &HVC.DT);
  2766:       if (KnownAlign.value() < minPHIAlignment)
  2767:         minPHIAlignment = KnownAlign.value();
  2768:     }
  2769:   }
  2770:   if (minPHIAlignment != Value::MaximumAlignment)
  2771:     return minPHIAlignment;
  2772:   return std::nullopt;
  2773: }
  2774: 
  2775: // Helper function to discover alignment for a ptr.
  2776: std::optional<uint64_t> HvxIdioms::getAlignment(Instruction &In,
  2777:                                                 Value *ptr) const {
  2778:   SmallPtrSet<Value *, 16> Visited;
  2779:   return getAlignmentImpl(In, ptr, Visited);
  2780: }
  2781: 
  2782: std::optional<uint64_t>
  2783: HvxIdioms::getAlignmentImpl(Instruction &In, Value *ptr,
  2784:                             SmallPtrSet<Value *, 16> &Visited) const {
  2785:   LLVM_DEBUG(dbgs() << "[getAlignment] for : " << *ptr << "\n");
  2786:   // Prevent infinite recursion
  2787:   if (!Visited.insert(ptr).second)
  2788:     return std::nullopt;
  2789:   // Try AssumptionCache.
  2790:   Align KnownAlign = getKnownAlignment(ptr, HVC.DL, &In, &HVC.AC, &HVC.DT);
  2791:   // This is the most formal and reliable source of information.
  2792:   if (KnownAlign.value() > 1) {
  2793:     LLVM_DEBUG(dbgs() << "  VC align(" << KnownAlign.value() << ")\n");
  2794:     return KnownAlign.value();
  2795:   }
  2796: 
  2797:   // If it is a PHI try to iterate through inputs
  2798:   if (PHINode *PN = dyn_cast<PHINode>(ptr)) {
  2799:     // See if we have a common base to which we know alignment.
  2800:     auto baseAlignmentOpt = getPHIBaseMinAlignment(In, PN);
```
- EN: It declares or implements routines such as LLVM_DEBUG, V6_vgathermh_pseudo, dyn_cast<ConstantDataVector>, CreateAlloca, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 LLVM_DEBUG, V6_vgathermh_pseudo, dyn_cast<ConstantDataVector>, CreateAlloca, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 2801-3000 / 第 2801-3000 行

```cpp
  2801:     if (!baseAlignmentOpt)
  2802:       return std::nullopt;
  2803: 
  2804:     uint64_t minBaseAlignment = *baseAlignmentOpt;
  2805:     // If it is 1, there is no point to keep on looking.
  2806:     if (minBaseAlignment == 1)
  2807:       return 1;
  2808:     // No see if all other incomming phi nodes are just loop carried constants.
  2809:     uint64_t minPHIAlignment = minBaseAlignment;
  2810:     LLVM_DEBUG(dbgs() << "  It is a PHI with(" << PN->getNumIncomingValues()
  2811:                       << ")nodes and min base aligned to (" << minBaseAlignment
  2812:                       << ")\n");
  2813:     for (unsigned i = 0; i < PN->getNumIncomingValues(); ++i) {
  2814:       Value *IV = PN->getIncomingValue(i);
  2815:       // We have already looked at all other values.
  2816:       if (!isa<GetElementPtrInst>(IV))
  2817:         continue;
  2818:       uint64_t MemberAlignment = Value::MaximumAlignment;
  2819:       if (auto res = getAlignment(*PN, IV))
  2820:         MemberAlignment = *res;
  2821:       else
  2822:         return std::nullopt;
  2823:       // Adjust total PHI alignment.
  2824:       if (minPHIAlignment > MemberAlignment)
  2825:         minPHIAlignment = MemberAlignment;
  2826:     }
  2827:     LLVM_DEBUG(dbgs() << "  total PHI alignment(" << minPHIAlignment << ")\n");
  2828:     return minPHIAlignment;
  2829:   }
  2830: 
  2831:   if (auto *GEP = dyn_cast<GetElementPtrInst>(ptr)) {
  2832:     auto *GEPPtr = GEP->getPointerOperand();
  2833:     // Only if this is the induction variable with const offset
  2834:     // Implicit assumption is that induction variable itself is a PHI
  2835:     if (&In == GEPPtr) {
  2836:       APInt Offset(HVC.DL.getPointerSizeInBits(
  2837:                        GEPPtr->getType()->getPointerAddressSpace()),
  2838:                    0);
  2839:       if (GEP->accumulateConstantOffset(HVC.DL, Offset)) {
  2840:         LLVM_DEBUG(dbgs() << "  Induction GEP with const step of ("
  2841:                           << Offset.getZExtValue() << ")\n");
  2842:         return Offset.getZExtValue();
  2843:       }
  2844:     }
  2845:   }
  2846: 
  2847:   return std::nullopt;
  2848: }
  2849: 
  2850: Value *HvxIdioms::processMStore(Instruction &In) const {
  2851:   [[maybe_unused]] auto *InpTy =
  2852:       dyn_cast<VectorType>(In.getOperand(0)->getType());
  2853:   assert(InpTy && "Cannot handle no vector type for llvm.masked.store");
  2854: 
  2855:   LLVM_DEBUG(dbgs() << "\n[Process mstore](" << In << ")\n"
  2856:                     << *In.getParent() << "\n");
  2857:   LLVM_DEBUG(dbgs() << "  Input type(" << *InpTy << ") elements("
  2858:                     << HVC.length(InpTy) << ") VecLen(" << HVC.getSizeOf(InpTy)
  2859:                     << ") type(" << *InpTy->getElementType() << ") of size("
  2860:                     << InpTy->getScalarSizeInBits() << ")bits\n");
  2861:   auto *CI = dyn_cast<CallBase>(&In);
  2862:   assert(CI && "Expected llvm.masked.store to be a call");
  2863:   Align HaveAlign = CI->getParamAlign(1).valueOrOne();
  2864: 
  2865:   uint64_t KA = 1;
  2866:   if (auto res = getAlignment(In, In.getOperand(1))) // ptr operand
  2867:     KA = *res;
  2868:   LLVM_DEBUG(dbgs() << "  HaveAlign(" << HaveAlign.value() << ") KnownAlign("
  2869:                     << KA << ")\n");
  2870:   // Normalize 0 -> ABI alignment of the stored value type (operand 0).
  2871:   Type *ValTy = In.getOperand(0)->getType();
  2872:   Align EffA =
  2873:       (KA > 0) ? Align(KA) : Align(HVC.DL.getABITypeAlign(ValTy).value());
  2874: 
  2875:   if (EffA < HaveAlign)
  2876:     return nullptr;
  2877: 
  2878:   // Attach/replace the param attribute on pointer param #1.
  2879:   AttrBuilder AttrB(CI->getContext());
  2880:   AttrB.addAlignmentAttr(EffA);
  2881:   CI->setAttributes(
  2882:       CI->getAttributes().addParamAttributes(CI->getContext(), 1, AttrB));
  2883:   return CI;
  2884: }
  2885: 
  2886: Value *HvxIdioms::processMLoad(Instruction &In) const {
  2887:   [[maybe_unused]] auto *InpTy = dyn_cast<VectorType>(In.getType());
  2888:   assert(InpTy && "Cannot handle non vector type for llvm.masked.store");
  2889:   LLVM_DEBUG(dbgs() << "\n[Process mload](" << In << ")\n"
  2890:                     << *In.getParent() << "\n");
  2891:   LLVM_DEBUG(dbgs() << "  Input type(" << *InpTy << ") elements("
  2892:                     << HVC.length(InpTy) << ") VecLen(" << HVC.getSizeOf(InpTy)
  2893:                     << ") type(" << *InpTy->getElementType() << ") of size("
  2894:                     << InpTy->getScalarSizeInBits() << ")bits\n");
  2895:   auto *CI = dyn_cast<CallBase>(&In);
  2896:   assert(CI && "Expected to be a call to llvm.masked.load");
  2897:   // The pointer is operand #0, and its param attribute index is also 0.
  2898:   Align HaveAlign = CI->getParamAlign(0).valueOrOne();
  2899: 
  2900:   // Compute best-known alignment KA from analysis.
  2901:   uint64_t KA = 1;
  2902:   if (auto res = getAlignment(In, In.getOperand(0))) // ptr operand
  2903:     KA = *res;
  2904: 
  2905:   // Normalize 0 → ABI alignment of the loaded value type.
  2906:   Type *ValTy = In.getType();
  2907:   Align EffA =
  2908:       (KA > 0) ? Align(KA) : Align(HVC.DL.getABITypeAlign(ValTy).value());
  2909:   if (EffA < HaveAlign)
  2910:     return nullptr;
  2911:   LLVM_DEBUG(dbgs() << "  HaveAlign(" << HaveAlign.value() << ") KnownAlign("
  2912:                     << KA << ")\n");
  2913: 
  2914:   // Attach/replace the param attribute on pointer param #0.
  2915:   AttrBuilder AttrB(CI->getContext());
  2916:   AttrB.addAlignmentAttr(EffA);
  2917:   CI->setAttributes(
  2918:       CI->getAttributes().addParamAttributes(CI->getContext(), 0, AttrB));
  2919:   return CI;
  2920: }
  2921: 
  2922: auto HvxIdioms::processFxpMulChopped(IRBuilderBase &Builder, Instruction &In,
  2923:                                      const FxpOp &Op) const -> Value * {
  2924:   assert(Op.X.Val->getType() == Op.Y.Val->getType());
  2925:   auto *InpTy = cast<VectorType>(Op.X.Val->getType());
  2926:   unsigned Width = InpTy->getScalarSizeInBits();
  2927:   bool Rounding = Op.RoundAt.has_value();
  2928: 
  2929:   if (!Op.RoundAt || *Op.RoundAt == Op.Frac - 1) {
  2930:     // The fixed-point intrinsics do signed multiplication.
  2931:     if (Width == Op.Frac + 1 && Op.X.Sgn != Unsigned && Op.Y.Sgn != Unsigned) {
  2932:       Value *QMul = nullptr;
  2933:       if (Width == 16) {
  2934:         QMul = createMulQ15(Builder, Op.X, Op.Y, Rounding);
  2935:       } else if (Width == 32) {
  2936:         QMul = createMulQ31(Builder, Op.X, Op.Y, Rounding);
  2937:       }
  2938:       if (QMul != nullptr)
  2939:         return QMul;
  2940:     }
  2941:   }
  2942: 
  2943:   assert(Width >= 32 || isPowerOf2_32(Width)); // Width <= 32 => Width is 2^n
  2944:   assert(Width < 32 || Width % 32 == 0);       // Width > 32 => Width is 32*k
  2945: 
  2946:   // If Width < 32, then it should really be 16.
  2947:   if (Width < 32) {
  2948:     if (Width < 16)
  2949:       return nullptr;
  2950:     // Getting here with Op.Frac == 0 isn't wrong, but suboptimal: here we
  2951:     // generate a full precision products, which is unnecessary if there is
  2952:     // no shift.
  2953:     assert(Width == 16);
  2954:     assert(Op.Frac != 0 && "Unshifted mul should have been skipped");
  2955:     if (Op.Frac == 16) {
  2956:       // Multiply high
  2957:       if (Value *MulH = createMulH16(Builder, Op.X, Op.Y))
  2958:         return MulH;
  2959:     }
  2960:     // Do full-precision multiply and shift.
  2961:     Value *Prod32 = createMul16(Builder, Op.X, Op.Y);
  2962:     if (Rounding) {
  2963:       Value *RoundVal =
  2964:           ConstantInt::get(Prod32->getType(), 1ull << *Op.RoundAt);
  2965:       Prod32 = Builder.CreateAdd(Prod32, RoundVal, "add");
  2966:     }
  2967: 
  2968:     Value *ShiftAmt = ConstantInt::get(Prod32->getType(), Op.Frac);
  2969:     Value *Shifted = Op.X.Sgn == Signed || Op.Y.Sgn == Signed
  2970:                          ? Builder.CreateAShr(Prod32, ShiftAmt, "asr")
  2971:                          : Builder.CreateLShr(Prod32, ShiftAmt, "lsr");
  2972:     return Builder.CreateTrunc(Shifted, InpTy, "trn");
  2973:   }
  2974: 
  2975:   // Width >= 32
  2976: 
  2977:   // Break up the arguments Op.X and Op.Y into vectors of smaller widths
  2978:   // in preparation of doing the multiplication by 32-bit parts.
  2979:   auto WordX = HVC.splitVectorElements(Builder, Op.X.Val, /*ToWidth=*/32);
  2980:   auto WordY = HVC.splitVectorElements(Builder, Op.Y.Val, /*ToWidth=*/32);
  2981:   auto WordP = createMulLong(Builder, WordX, Op.X.Sgn, WordY, Op.Y.Sgn);
  2982: 
  2983:   auto *HvxWordTy = cast<VectorType>(WordP.front()->getType());
  2984: 
  2985:   // Add the optional rounding to the proper word.
  2986:   if (Op.RoundAt.has_value()) {
  2987:     Value *Zero = Constant::getNullValue(WordX[0]->getType());
  2988:     SmallVector<Value *> RoundV(WordP.size(), Zero);
  2989:     RoundV[*Op.RoundAt / 32] =
  2990:         ConstantInt::get(HvxWordTy, 1ull << (*Op.RoundAt % 32));
  2991:     WordP = createAddLong(Builder, WordP, RoundV);
  2992:   }
  2993: 
  2994:   // createRightShiftLong?
  2995: 
  2996:   // Shift all products right by Op.Frac.
  2997:   unsigned SkipWords = Op.Frac / 32;
  2998:   Constant *ShiftAmt = ConstantInt::get(HvxWordTy, Op.Frac % 32);
  2999: 
  3000:   for (int Dst = 0, End = WordP.size() - SkipWords; Dst != End; ++Dst) {
```
- EN: It declares or implements routines such as LLVM_DEBUG, getNumIncomingValues, getIncomingValue, getPointerOperand, ... (33 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 LLVM_DEBUG, getNumIncomingValues, getIncomingValue, getPointerOperand, ... (33 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 3001-3200 / 第 3001-3200 行

```cpp
  3001:     int Src = Dst + SkipWords;
  3002:     Value *Lo = WordP[Src];
  3003:     if (Src + 1 < End) {
  3004:       Value *Hi = WordP[Src + 1];
  3005:       WordP[Dst] = Builder.CreateIntrinsic(HvxWordTy, Intrinsic::fshr,
  3006:                                            {Hi, Lo, ShiftAmt},
  3007:                                            /*FMFSource*/ nullptr, "int");
  3008:     } else {
  3009:       // The shift of the most significant word.
  3010:       WordP[Dst] = Builder.CreateAShr(Lo, ShiftAmt, "asr");
  3011:     }
  3012:   }
  3013:   if (SkipWords != 0)
  3014:     WordP.resize(WordP.size() - SkipWords);
  3015: 
  3016:   return HVC.joinVectorElements(Builder, WordP, Op.ResTy);
  3017: }
  3018: 
  3019: auto HvxIdioms::createMulQ15(IRBuilderBase &Builder, SValue X, SValue Y,
  3020:                              bool Rounding) const -> Value * {
  3021:   assert(X.Val->getType() == Y.Val->getType());
  3022:   assert(X.Val->getType()->getScalarType() == HVC.getIntTy(16));
  3023:   assert(HVC.HST.isHVXVectorType(EVT::getEVT(X.Val->getType(), false)));
  3024: 
  3025:   // There is no non-rounding intrinsic for i16.
  3026:   if (!Rounding || X.Sgn == Unsigned || Y.Sgn == Unsigned)
  3027:     return nullptr;
  3028: 
  3029:   auto V6_vmpyhvsrs = HVC.HST.getIntrinsicId(Hexagon::V6_vmpyhvsrs);
  3030:   return HVC.createHvxIntrinsic(Builder, V6_vmpyhvsrs, X.Val->getType(),
  3031:                                 {X.Val, Y.Val});
  3032: }
  3033: 
  3034: auto HvxIdioms::createMulQ31(IRBuilderBase &Builder, SValue X, SValue Y,
  3035:                              bool Rounding) const -> Value * {
  3036:   Type *InpTy = X.Val->getType();
  3037:   assert(InpTy == Y.Val->getType());
  3038:   assert(InpTy->getScalarType() == HVC.getIntTy(32));
  3039:   assert(HVC.HST.isHVXVectorType(EVT::getEVT(InpTy, false)));
  3040: 
  3041:   if (X.Sgn == Unsigned || Y.Sgn == Unsigned)
  3042:     return nullptr;
  3043: 
  3044:   auto V6_vmpyewuh = HVC.HST.getIntrinsicId(Hexagon::V6_vmpyewuh);
  3045:   auto V6_vmpyo_acc = Rounding
  3046:                           ? HVC.HST.getIntrinsicId(Hexagon::V6_vmpyowh_rnd_sacc)
  3047:                           : HVC.HST.getIntrinsicId(Hexagon::V6_vmpyowh_sacc);
  3048:   Value *V1 =
  3049:       HVC.createHvxIntrinsic(Builder, V6_vmpyewuh, InpTy, {X.Val, Y.Val});
  3050:   return HVC.createHvxIntrinsic(Builder, V6_vmpyo_acc, InpTy,
  3051:                                 {V1, X.Val, Y.Val});
  3052: }
  3053: 
  3054: auto HvxIdioms::createAddCarry(IRBuilderBase &Builder, Value *X, Value *Y,
  3055:                                Value *CarryIn) const
  3056:     -> std::pair<Value *, Value *> {
  3057:   assert(X->getType() == Y->getType());
  3058:   auto VecTy = cast<VectorType>(X->getType());
  3059:   if (VecTy == HvxI32Ty && HVC.HST.useHVXV62Ops()) {
  3060:     SmallVector<Value *> Args = {X, Y};
  3061:     Intrinsic::ID AddCarry;
  3062:     if (CarryIn == nullptr && HVC.HST.useHVXV66Ops()) {
  3063:       AddCarry = HVC.HST.getIntrinsicId(Hexagon::V6_vaddcarryo);
  3064:     } else {
  3065:       AddCarry = HVC.HST.getIntrinsicId(Hexagon::V6_vaddcarry);
  3066:       if (CarryIn == nullptr)
  3067:         CarryIn = Constant::getNullValue(HVC.getBoolTy(HVC.length(VecTy)));
  3068:       Args.push_back(CarryIn);
  3069:     }
  3070:     Value *Ret = HVC.createHvxIntrinsic(Builder, AddCarry,
  3071:                                         /*RetTy=*/nullptr, Args);
  3072:     Value *Result = Builder.CreateExtractValue(Ret, {0}, "ext");
  3073:     Value *CarryOut = Builder.CreateExtractValue(Ret, {1}, "ext");
  3074:     return {Result, CarryOut};
  3075:   }
  3076: 
  3077:   // In other cases, do a regular add, and unsigned compare-less-than.
  3078:   // The carry-out can originate in two places: adding the carry-in or adding
  3079:   // the two input values.
  3080:   Value *Result1 = X; // Result1 = X + CarryIn
  3081:   if (CarryIn != nullptr) {
  3082:     unsigned Width = VecTy->getScalarSizeInBits();
  3083:     uint32_t Mask = 1;
  3084:     if (Width < 32) {
  3085:       for (unsigned i = 0, e = 32 / Width; i != e; ++i)
  3086:         Mask = (Mask << Width) | 1;
  3087:     }
  3088:     auto V6_vandqrt = HVC.HST.getIntrinsicId(Hexagon::V6_vandqrt);
  3089:     Value *ValueIn =
  3090:         HVC.createHvxIntrinsic(Builder, V6_vandqrt, /*RetTy=*/nullptr,
  3091:                                {CarryIn, HVC.getConstInt(Mask)});
  3092:     Result1 = Builder.CreateAdd(X, ValueIn, "add");
  3093:   }
  3094: 
  3095:   Value *CarryOut1 = Builder.CreateCmp(CmpInst::ICMP_ULT, Result1, X, "cmp");
  3096:   Value *Result2 = Builder.CreateAdd(Result1, Y, "add");
  3097:   Value *CarryOut2 = Builder.CreateCmp(CmpInst::ICMP_ULT, Result2, Y, "cmp");
  3098:   return {Result2, Builder.CreateOr(CarryOut1, CarryOut2, "orb")};
  3099: }
  3100: 
  3101: auto HvxIdioms::createMul16(IRBuilderBase &Builder, SValue X, SValue Y) const
  3102:     -> Value * {
  3103:   Intrinsic::ID V6_vmpyh = 0;
  3104:   std::tie(X, Y) = canonSgn(X, Y);
  3105: 
  3106:   if (X.Sgn == Signed) {
  3107:     V6_vmpyh = HVC.HST.getIntrinsicId(Hexagon::V6_vmpyhv);
  3108:   } else if (Y.Sgn == Signed) {
  3109:     // In vmpyhus the second operand is unsigned
  3110:     V6_vmpyh = HVC.HST.getIntrinsicId(Hexagon::V6_vmpyhus);
  3111:   } else {
  3112:     V6_vmpyh = HVC.HST.getIntrinsicId(Hexagon::V6_vmpyuhv);
  3113:   }
  3114: 
  3115:   // i16*i16 -> i32 / interleaved
  3116:   Value *P =
  3117:       HVC.createHvxIntrinsic(Builder, V6_vmpyh, HvxP32Ty, {Y.Val, X.Val});
  3118:   // Deinterleave
  3119:   return HVC.vshuff(Builder, HVC.sublo(Builder, P), HVC.subhi(Builder, P));
  3120: }
  3121: 
  3122: auto HvxIdioms::createMulH16(IRBuilderBase &Builder, SValue X, SValue Y) const
  3123:     -> Value * {
  3124:   Type *HvxI16Ty = HVC.getHvxTy(HVC.getIntTy(16), /*Pair=*/false);
  3125: 
  3126:   if (HVC.HST.useHVXV69Ops()) {
  3127:     if (X.Sgn != Signed && Y.Sgn != Signed) {
  3128:       auto V6_vmpyuhvs = HVC.HST.getIntrinsicId(Hexagon::V6_vmpyuhvs);
  3129:       return HVC.createHvxIntrinsic(Builder, V6_vmpyuhvs, HvxI16Ty,
  3130:                                     {X.Val, Y.Val});
  3131:     }
  3132:   }
  3133: 
  3134:   Type *HvxP16Ty = HVC.getHvxTy(HVC.getIntTy(16), /*Pair=*/true);
  3135:   Value *Pair16 =
  3136:       Builder.CreateBitCast(createMul16(Builder, X, Y), HvxP16Ty, "cst");
  3137:   unsigned Len = HVC.length(HvxP16Ty) / 2;
  3138: 
  3139:   SmallVector<int, 128> PickOdd(Len);
  3140:   for (int i = 0; i != static_cast<int>(Len); ++i)
  3141:     PickOdd[i] = 2 * i + 1;
  3142: 
  3143:   return Builder.CreateShuffleVector(
  3144:       HVC.sublo(Builder, Pair16), HVC.subhi(Builder, Pair16), PickOdd, "shf");
  3145: }
  3146: 
  3147: auto HvxIdioms::createMul32(IRBuilderBase &Builder, SValue X, SValue Y) const
  3148:     -> std::pair<Value *, Value *> {
  3149:   assert(X.Val->getType() == Y.Val->getType());
  3150:   assert(X.Val->getType() == HvxI32Ty);
  3151: 
  3152:   Intrinsic::ID V6_vmpy_parts;
  3153:   std::tie(X, Y) = canonSgn(X, Y);
  3154: 
  3155:   if (X.Sgn == Signed) {
  3156:     V6_vmpy_parts = Intrinsic::hexagon_V6_vmpyss_parts;
  3157:   } else if (Y.Sgn == Signed) {
  3158:     V6_vmpy_parts = Intrinsic::hexagon_V6_vmpyus_parts;
  3159:   } else {
  3160:     V6_vmpy_parts = Intrinsic::hexagon_V6_vmpyuu_parts;
  3161:   }
  3162: 
  3163:   Value *Parts = HVC.createHvxIntrinsic(Builder, V6_vmpy_parts, nullptr,
  3164:                                         {X.Val, Y.Val}, {HvxI32Ty});
  3165:   Value *Hi = Builder.CreateExtractValue(Parts, {0}, "ext");
  3166:   Value *Lo = Builder.CreateExtractValue(Parts, {1}, "ext");
  3167:   return {Lo, Hi};
  3168: }
  3169: 
  3170: auto HvxIdioms::createAddLong(IRBuilderBase &Builder, ArrayRef<Value *> WordX,
  3171:                               ArrayRef<Value *> WordY) const
  3172:     -> SmallVector<Value *> {
  3173:   assert(WordX.size() == WordY.size());
  3174:   unsigned Idx = 0, Length = WordX.size();
  3175:   SmallVector<Value *> Sum(Length);
  3176: 
  3177:   while (Idx != Length) {
  3178:     if (HVC.isZero(WordX[Idx]))
  3179:       Sum[Idx] = WordY[Idx];
  3180:     else if (HVC.isZero(WordY[Idx]))
  3181:       Sum[Idx] = WordX[Idx];
  3182:     else
  3183:       break;
  3184:     ++Idx;
  3185:   }
  3186: 
  3187:   Value *Carry = nullptr;
  3188:   for (; Idx != Length; ++Idx) {
  3189:     std::tie(Sum[Idx], Carry) =
  3190:         createAddCarry(Builder, WordX[Idx], WordY[Idx], Carry);
  3191:   }
  3192: 
  3193:   // This drops the final carry beyond the highest word.
  3194:   return Sum;
  3195: }
  3196: 
  3197: auto HvxIdioms::createMulLong(IRBuilderBase &Builder, ArrayRef<Value *> WordX,
  3198:                               Signedness SgnX, ArrayRef<Value *> WordY,
  3199:                               Signedness SgnY) const -> SmallVector<Value *> {
  3200:   SmallVector<SmallVector<Value *>> Products(WordX.size() + WordY.size());
```
- EN: It declares or implements routines such as CreateAShr, joinVectorElements, assert, getIntrinsicId, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 CreateAShr, joinVectorElements, assert, getIntrinsicId, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 3201-3400 / 第 3201-3400 行

```cpp
  3201: 
  3202:   // WordX[i] * WordY[j] produces words i+j and i+j+1 of the results,
  3203:   // that is halves 2(i+j), 2(i+j)+1, 2(i+j)+2, 2(i+j)+3.
  3204:   for (int i = 0, e = WordX.size(); i != e; ++i) {
  3205:     for (int j = 0, f = WordY.size(); j != f; ++j) {
  3206:       // Check the 4 halves that this multiplication can generate.
  3207:       Signedness SX = (i + 1 == e) ? SgnX : Unsigned;
  3208:       Signedness SY = (j + 1 == f) ? SgnY : Unsigned;
  3209:       auto [Lo, Hi] = createMul32(Builder, {WordX[i], SX}, {WordY[j], SY});
  3210:       Products[i + j + 0].push_back(Lo);
  3211:       Products[i + j + 1].push_back(Hi);
  3212:     }
  3213:   }
  3214: 
  3215:   Value *Zero = Constant::getNullValue(WordX[0]->getType());
  3216: 
  3217:   auto pop_back_or_zero = [Zero](auto &Vector) -> Value * {
  3218:     if (Vector.empty())
  3219:       return Zero;
  3220:     auto Last = Vector.back();
  3221:     Vector.pop_back();
  3222:     return Last;
  3223:   };
  3224: 
  3225:   for (int i = 0, e = Products.size(); i != e; ++i) {
  3226:     while (Products[i].size() > 1) {
  3227:       Value *Carry = nullptr; // no carry-in
  3228:       for (int j = i; j != e; ++j) {
  3229:         auto &ProdJ = Products[j];
  3230:         auto [Sum, CarryOut] = createAddCarry(Builder, pop_back_or_zero(ProdJ),
  3231:                                               pop_back_or_zero(ProdJ), Carry);
  3232:         ProdJ.insert(ProdJ.begin(), Sum);
  3233:         Carry = CarryOut;
  3234:       }
  3235:     }
  3236:   }
  3237: 
  3238:   SmallVector<Value *> WordP;
  3239:   for (auto &P : Products) {
  3240:     assert(P.size() == 1 && "Should have been added together");
  3241:     WordP.push_back(P.front());
  3242:   }
  3243: 
  3244:   return WordP;
  3245: }
  3246: 
  3247: auto HvxIdioms::run() -> bool {
  3248:   bool Changed = false;
  3249: 
  3250:   for (BasicBlock &B : HVC.F) {
  3251:     for (auto It = B.rbegin(); It != B.rend(); ++It) {
  3252:       if (auto Fxm = matchFxpMul(*It)) {
  3253:         Value *New = processFxpMul(*It, *Fxm);
  3254:         // Always report "changed" for now.
  3255:         Changed = true;
  3256:         if (!New)
  3257:           continue;
  3258:         bool StartOver = !isa<Instruction>(New);
  3259:         It->replaceAllUsesWith(New);
  3260:         RecursivelyDeleteTriviallyDeadInstructions(&*It, &HVC.TLI);
  3261:         It = StartOver ? B.rbegin()
  3262:                        : cast<Instruction>(New)->getReverseIterator();
  3263:         Changed = true;
  3264:       } else if (matchGather(*It)) {
  3265:         Value *New = processVGather(*It);
  3266:         if (!New)
  3267:           continue;
  3268:         LLVM_DEBUG(dbgs() << "  Gather : " << *New << "\n");
  3269:         // We replace original intrinsic with a new pseudo call.
  3270:         It->eraseFromParent();
  3271:         It = cast<Instruction>(New)->getReverseIterator();
  3272:         RecursivelyDeleteTriviallyDeadInstructions(&*It, &HVC.TLI);
  3273:         Changed = true;
  3274:       } else if (matchScatter(*It)) {
  3275:         Value *New = processVScatter(*It);
  3276:         if (!New)
  3277:           continue;
  3278:         LLVM_DEBUG(dbgs() << "  Scatter : " << *New << "\n");
  3279:         // We replace original intrinsic with a new pseudo call.
  3280:         It->eraseFromParent();
  3281:         It = cast<Instruction>(New)->getReverseIterator();
  3282:         RecursivelyDeleteTriviallyDeadInstructions(&*It, &HVC.TLI);
  3283:         Changed = true;
  3284:       } else if (matchMLoad(*It)) {
  3285:         Value *New = processMLoad(*It);
  3286:         if (!New)
  3287:           continue;
  3288:         LLVM_DEBUG(dbgs() << "  MLoad : " << *New << "\n");
  3289:         Changed = true;
  3290:       } else if (matchMStore(*It)) {
  3291:         Value *New = processMStore(*It);
  3292:         if (!New)
  3293:           continue;
  3294:         LLVM_DEBUG(dbgs() << "  MStore : " << *New << "\n");
  3295:         Changed = true;
  3296:       }
  3297:     }
  3298:   }
  3299: 
  3300:   return Changed;
  3301: }
  3302: 
  3303: // --- End HvxIdioms
  3304: 
  3305: auto HexagonVectorCombine::run() -> bool {
  3306:   if (DumpModule)
  3307:     dbgs() << "Module before HexagonVectorCombine\n" << *F.getParent();
  3308: 
  3309:   bool Changed = false;
  3310:   if (HST.useHVXOps()) {
  3311:     if (VAEnabled)
  3312:       Changed |= AlignVectors(*this).run();
  3313:     if (VIEnabled)
  3314:       Changed |= HvxIdioms(*this).run();
  3315:   }
  3316: 
  3317:   if (DumpModule) {
  3318:     dbgs() << "Module " << (Changed ? "(modified)" : "(unchanged)")
  3319:            << " after HexagonVectorCombine\n"
  3320:            << *F.getParent();
  3321:   }
  3322:   return Changed;
  3323: }
  3324: 
  3325: auto HexagonVectorCombine::getIntTy(unsigned Width) const -> IntegerType * {
  3326:   return IntegerType::get(F.getContext(), Width);
  3327: }
  3328: 
  3329: auto HexagonVectorCombine::getByteTy(int ElemCount) const -> Type * {
  3330:   assert(ElemCount >= 0);
  3331:   IntegerType *ByteTy = Type::getInt8Ty(F.getContext());
  3332:   if (ElemCount == 0)
  3333:     return ByteTy;
  3334:   return VectorType::get(ByteTy, ElemCount, /*Scalable=*/false);
  3335: }
  3336: 
  3337: auto HexagonVectorCombine::getBoolTy(int ElemCount) const -> Type * {
  3338:   assert(ElemCount >= 0);
  3339:   IntegerType *BoolTy = Type::getInt1Ty(F.getContext());
  3340:   if (ElemCount == 0)
  3341:     return BoolTy;
  3342:   return VectorType::get(BoolTy, ElemCount, /*Scalable=*/false);
  3343: }
  3344: 
  3345: auto HexagonVectorCombine::getConstInt(int Val, unsigned Width) const
  3346:     -> ConstantInt * {
  3347:   return ConstantInt::getSigned(getIntTy(Width), Val);
  3348: }
  3349: 
  3350: auto HexagonVectorCombine::isZero(const Value *Val) const -> bool {
  3351:   if (auto *C = dyn_cast<Constant>(Val))
  3352:     return C->isNullValue();
  3353:   return false;
  3354: }
  3355: 
  3356: auto HexagonVectorCombine::getIntValue(const Value *Val) const
  3357:     -> std::optional<APInt> {
  3358:   if (auto *CI = dyn_cast<ConstantInt>(Val))
  3359:     return CI->getValue();
  3360:   return std::nullopt;
  3361: }
  3362: 
  3363: auto HexagonVectorCombine::isUndef(const Value *Val) const -> bool {
  3364:   return isa<UndefValue>(Val);
  3365: }
  3366: 
  3367: auto HexagonVectorCombine::isTrue(const Value *Val) const -> bool {
  3368:   return Val == ConstantInt::getTrue(Val->getType());
  3369: }
  3370: 
  3371: auto HexagonVectorCombine::isFalse(const Value *Val) const -> bool {
  3372:   return isZero(Val);
  3373: }
  3374: 
  3375: auto HexagonVectorCombine::getHvxTy(Type *ElemTy, bool Pair) const
  3376:     -> VectorType * {
  3377:   EVT ETy = EVT::getEVT(ElemTy, false);
  3378:   assert(ETy.isSimple() && "Invalid HVX element type");
  3379:   // Do not allow boolean types here: they don't have a fixed length.
  3380:   assert(HST.isHVXElementType(ETy.getSimpleVT(), /*IncludeBool=*/false) &&
  3381:          "Invalid HVX element type");
  3382:   unsigned HwLen = HST.getVectorLength();
  3383:   unsigned NumElems = (8 * HwLen) / ETy.getSizeInBits();
  3384:   return VectorType::get(ElemTy, Pair ? 2 * NumElems : NumElems,
  3385:                          /*Scalable=*/false);
  3386: }
  3387: 
  3388: auto HexagonVectorCombine::getSizeOf(const Value *Val, SizeKind Kind) const
  3389:     -> int {
  3390:   return getSizeOf(Val->getType(), Kind);
  3391: }
  3392: 
  3393: auto HexagonVectorCombine::getSizeOf(const Type *Ty, SizeKind Kind) const
  3394:     -> int {
  3395:   auto *NcTy = const_cast<Type *>(Ty);
  3396:   switch (Kind) {
  3397:   case Store:
  3398:     return DL.getTypeStoreSize(NcTy).getFixedValue();
  3399:   case Alloc:
  3400:     return DL.getTypeAllocSize(NcTy).getFixedValue();
```
- EN: It declares or implements routines such as push_back, Constant::getNullValue, back, pop_back, ... (35 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonVectorCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 push_back, Constant::getNullValue, back, pop_back, ... (35 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonVectorCombine，说明了它与同级后端组件的连接关系。

### Lines 3401-3600 / 第 3401-3600 行

```cpp
  3401:   }
  3402:   llvm_unreachable("Unhandled SizeKind enum");
  3403: }
  3404: 
  3405: auto HexagonVectorCombine::getTypeAlignment(Type *Ty) const -> int {
  3406:   // The actual type may be shorter than the HVX vector, so determine
  3407:   // the alignment based on subtarget info.
  3408:   if (HST.isTypeForHVX(Ty))
  3409:     return HST.getVectorLength();
  3410:   return DL.getABITypeAlign(Ty).value();
  3411: }
  3412: 
  3413: auto HexagonVectorCombine::length(Value *Val) const -> size_t {
  3414:   return length(Val->getType());
  3415: }
  3416: 
  3417: auto HexagonVectorCombine::length(Type *Ty) const -> size_t {
  3418:   auto *VecTy = dyn_cast<VectorType>(Ty);
  3419:   assert(VecTy && "Must be a vector type");
  3420:   return VecTy->getElementCount().getFixedValue();
  3421: }
  3422: 
  3423: auto HexagonVectorCombine::simplify(Value *V) const -> Value * {
  3424:   if (auto *In = dyn_cast<Instruction>(V)) {
  3425:     SimplifyQuery Q(DL, &TLI, &DT, &AC, In);
  3426:     return simplifyInstruction(In, Q);
  3427:   }
  3428:   return nullptr;
  3429: }
  3430: 
  3431: // Insert bytes [Start..Start+Length) of Src into Dst at byte Where.
  3432: auto HexagonVectorCombine::insertb(IRBuilderBase &Builder, Value *Dst,
  3433:                                    Value *Src, int Start, int Length,
  3434:                                    int Where) const -> Value * {
  3435:   assert(isByteVecTy(Dst->getType()) && isByteVecTy(Src->getType()));
  3436:   int SrcLen = getSizeOf(Src);
  3437:   int DstLen = getSizeOf(Dst);
  3438:   assert(0 <= Start && Start + Length <= SrcLen);
  3439:   assert(0 <= Where && Where + Length <= DstLen);
  3440: 
  3441:   int P2Len = PowerOf2Ceil(SrcLen | DstLen);
  3442:   auto *Poison = PoisonValue::get(getByteTy());
  3443:   Value *P2Src = vresize(Builder, Src, P2Len, Poison);
  3444:   Value *P2Dst = vresize(Builder, Dst, P2Len, Poison);
  3445: 
  3446:   SmallVector<int, 256> SMask(P2Len);
  3447:   for (int i = 0; i != P2Len; ++i) {
  3448:     // If i is in [Where, Where+Length), pick Src[Start+(i-Where)].
  3449:     // Otherwise, pick Dst[i];
  3450:     SMask[i] =
  3451:         (Where <= i && i < Where + Length) ? P2Len + Start + (i - Where) : i;
  3452:   }
  3453: 
  3454:   Value *P2Insert = Builder.CreateShuffleVector(P2Dst, P2Src, SMask, "shf");
  3455:   return vresize(Builder, P2Insert, DstLen, Poison);
  3456: }
  3457: 
  3458: auto HexagonVectorCombine::vlalignb(IRBuilderBase &Builder, Value *Lo,
  3459:                                     Value *Hi, Value *Amt) const -> Value * {
  3460:   assert(Lo->getType() == Hi->getType() && "Argument type mismatch");
  3461:   if (isZero(Amt))
  3462:     return Hi;
  3463:   int VecLen = getSizeOf(Hi);
  3464:   if (auto IntAmt = getIntValue(Amt))
  3465:     return getElementRange(Builder, Lo, Hi, VecLen - IntAmt->getSExtValue(),
  3466:                            VecLen);
  3467: 
  3468:   if (HST.isTypeForHVX(Hi->getType())) {
  3469:     assert(static_cast<unsigned>(VecLen) == HST.getVectorLength() &&
  3470:            "Expecting an exact HVX type");
  3471:     return createHvxIntrinsic(Builder, HST.getIntrinsicId(Hexagon::V6_vlalignb),
  3472:                               Hi->getType(), {Hi, Lo, Amt});
  3473:   }
  3474: 
  3475:   if (VecLen == 4) {
  3476:     Value *Pair = concat(Builder, {Lo, Hi});
  3477:     Value *Shift =
  3478:         Builder.CreateLShr(Builder.CreateShl(Pair, Amt, "shl"), 32, "lsr");
  3479:     Value *Trunc =
  3480:         Builder.CreateTrunc(Shift, Type::getInt32Ty(F.getContext()), "trn");
  3481:     return Builder.CreateBitCast(Trunc, Hi->getType(), "cst");
  3482:   }
  3483:   if (VecLen == 8) {
  3484:     Value *Sub = Builder.CreateSub(getConstInt(VecLen), Amt, "sub");
  3485:     return vralignb(Builder, Lo, Hi, Sub);
  3486:   }
  3487:   llvm_unreachable("Unexpected vector length");
  3488: }
  3489: 
  3490: auto HexagonVectorCombine::vralignb(IRBuilderBase &Builder, Value *Lo,
  3491:                                     Value *Hi, Value *Amt) const -> Value * {
  3492:   assert(Lo->getType() == Hi->getType() && "Argument type mismatch");
  3493:   if (isZero(Amt))
  3494:     return Lo;
  3495:   int VecLen = getSizeOf(Lo);
  3496:   if (auto IntAmt = getIntValue(Amt))
  3497:     return getElementRange(Builder, Lo, Hi, IntAmt->getSExtValue(), VecLen);
  3498: 
  3499:   if (HST.isTypeForHVX(Lo->getType())) {
  3500:     assert(static_cast<unsigned>(VecLen) == HST.getVectorLength() &&
  3501:            "Expecting an exact HVX type");
  3502:     return createHvxIntrinsic(Builder, HST.getIntrinsicId(Hexagon::V6_valignb),
  3503:                               Lo->getType(), {Hi, Lo, Amt});
  3504:   }
  3505: 
  3506:   if (VecLen == 4) {
  3507:     Value *Pair = concat(Builder, {Lo, Hi});
  3508:     Value *Shift = Builder.CreateLShr(Pair, Amt, "lsr");
  3509:     Value *Trunc =
  3510:         Builder.CreateTrunc(Shift, Type::getInt32Ty(F.getContext()), "trn");
  3511:     return Builder.CreateBitCast(Trunc, Lo->getType(), "cst");
  3512:   }
  3513:   if (VecLen == 8) {
  3514:     Type *Int64Ty = Type::getInt64Ty(F.getContext());
  3515:     Value *Lo64 = Builder.CreateBitCast(Lo, Int64Ty, "cst");
  3516:     Value *Hi64 = Builder.CreateBitCast(Hi, Int64Ty, "cst");
  3517:     Value *Call = Builder.CreateIntrinsic(Intrinsic::hexagon_S2_valignrb,
  3518:                                           {Hi64, Lo64, Amt},
  3519:                                           /*FMFSource=*/nullptr, "cup");
  3520:     return Builder.CreateBitCast(Call, Lo->getType(), "cst");
  3521:   }
  3522:   llvm_unreachable("Unexpected vector length");
  3523: }
  3524: 
  3525: // Concatenates a sequence of vectors of the same type.
  3526: auto HexagonVectorCombine::concat(IRBuilderBase &Builder,
  3527:                                   ArrayRef<Value *> Vecs) const -> Value * {
  3528:   assert(!Vecs.empty());
  3529:   SmallVector<int, 256> SMask;
  3530:   std::vector<Value *> Work[2];
  3531:   int ThisW = 0, OtherW = 1;
  3532: 
  3533:   Work[ThisW].assign(Vecs.begin(), Vecs.end());
  3534:   while (Work[ThisW].size() > 1) {
  3535:     auto *Ty = cast<VectorType>(Work[ThisW].front()->getType());
  3536:     SMask.resize(length(Ty) * 2);
  3537:     std::iota(SMask.begin(), SMask.end(), 0);
  3538: 
  3539:     Work[OtherW].clear();
  3540:     if (Work[ThisW].size() % 2 != 0)
  3541:       Work[ThisW].push_back(UndefValue::get(Ty));
  3542:     for (int i = 0, e = Work[ThisW].size(); i < e; i += 2) {
  3543:       Value *Joined = Builder.CreateShuffleVector(
  3544:           Work[ThisW][i], Work[ThisW][i + 1], SMask, "shf");
  3545:       Work[OtherW].push_back(Joined);
  3546:     }
  3547:     std::swap(ThisW, OtherW);
  3548:   }
  3549: 
  3550:   // Since there may have been some undefs appended to make shuffle operands
  3551:   // have the same type, perform the last shuffle to only pick the original
  3552:   // elements.
  3553:   SMask.resize(Vecs.size() * length(Vecs.front()->getType()));
  3554:   std::iota(SMask.begin(), SMask.end(), 0);
  3555:   Value *Total = Work[ThisW].front();
  3556:   return Builder.CreateShuffleVector(Total, SMask, "shf");
  3557: }
  3558: 
  3559: auto HexagonVectorCombine::vresize(IRBuilderBase &Builder, Value *Val,
  3560:                                    int NewSize, Value *Pad) const -> Value * {
  3561:   assert(isa<VectorType>(Val->getType()));
  3562:   auto *ValTy = cast<VectorType>(Val->getType());
  3563:   assert(ValTy->getElementType() == Pad->getType());
  3564: 
  3565:   int CurSize = length(ValTy);
  3566:   if (CurSize == NewSize)
  3567:     return Val;
  3568:   // Truncate?
  3569:   if (CurSize > NewSize)
  3570:     return getElementRange(Builder, Val, /*Ignored*/ Val, 0, NewSize);
  3571:   // Extend.
  3572:   SmallVector<int, 128> SMask(NewSize);
  3573:   std::iota(SMask.begin(), SMask.begin() + CurSize, 0);
  3574:   std::fill(SMask.begin() + CurSize, SMask.end(), CurSize);
  3575:   Value *PadVec = Builder.CreateVectorSplat(CurSize, Pad, "spt");
  3576:   return Builder.CreateShuffleVector(Val, PadVec, SMask, "shf");
  3577: }
  3578: 
  3579: auto HexagonVectorCombine::rescale(IRBuilderBase &Builder, Value *Mask,
  3580:                                    Type *FromTy, Type *ToTy) const -> Value * {
  3581:   // Mask is a vector <N x i1>, where each element corresponds to an
  3582:   // element of FromTy. Remap it so that each element will correspond
  3583:   // to an element of ToTy.
  3584:   assert(isa<VectorType>(Mask->getType()));
  3585: 
  3586:   Type *FromSTy = FromTy->getScalarType();
  3587:   Type *ToSTy = ToTy->getScalarType();
  3588:   if (FromSTy == ToSTy)
  3589:     return Mask;
  3590: 
  3591:   int FromSize = getSizeOf(FromSTy);
  3592:   int ToSize = getSizeOf(ToSTy);
  3593:   assert(FromSize % ToSize == 0 || ToSize % FromSize == 0);
  3594: 
  3595:   auto *MaskTy = cast<VectorType>(Mask->getType());
  3596:   int FromCount = length(MaskTy);
  3597:   int ToCount = (FromCount * FromSize) / ToSize;
  3598:   assert((FromCount * FromSize) % ToSize == 0);
  3599: 
  3600:   auto *FromITy = getIntTy(FromSize * 8);
```
- EN: It declares or implements routines such as llvm_unreachable, getABITypeAlign, length, dyn_cast<VectorType>, ... (32 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonVectorCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm_unreachable, getABITypeAlign, length, dyn_cast<VectorType>, ... (32 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonVectorCombine，说明了它与同级后端组件的连接关系。

### Lines 3601-3800 / 第 3601-3800 行

```cpp
  3601:   auto *ToITy = getIntTy(ToSize * 8);
  3602: 
  3603:   // Mask <N x i1> -> sext to <N x FromTy> -> bitcast to <M x ToTy> ->
  3604:   // -> trunc to <M x i1>.
  3605:   Value *Ext = Builder.CreateSExt(
  3606:       Mask, VectorType::get(FromITy, FromCount, /*Scalable=*/false), "sxt");
  3607:   Value *Cast = Builder.CreateBitCast(
  3608:       Ext, VectorType::get(ToITy, ToCount, /*Scalable=*/false), "cst");
  3609:   return Builder.CreateTrunc(
  3610:       Cast, VectorType::get(getBoolTy(), ToCount, /*Scalable=*/false), "trn");
  3611: }
  3612: 
  3613: // Bitcast to bytes, and return least significant bits.
  3614: auto HexagonVectorCombine::vlsb(IRBuilderBase &Builder, Value *Val) const
  3615:     -> Value * {
  3616:   Type *ScalarTy = Val->getType()->getScalarType();
  3617:   if (ScalarTy == getBoolTy())
  3618:     return Val;
  3619: 
  3620:   Value *Bytes = vbytes(Builder, Val);
  3621:   if (auto *VecTy = dyn_cast<VectorType>(Bytes->getType()))
  3622:     return Builder.CreateTrunc(Bytes, getBoolTy(getSizeOf(VecTy)), "trn");
  3623:   // If Bytes is a scalar (i.e. Val was a scalar byte), return i1, not
  3624:   // <1 x i1>.
  3625:   return Builder.CreateTrunc(Bytes, getBoolTy(), "trn");
  3626: }
  3627: 
  3628: // Bitcast to bytes for non-bool. For bool, convert i1 -> i8.
  3629: auto HexagonVectorCombine::vbytes(IRBuilderBase &Builder, Value *Val) const
  3630:     -> Value * {
  3631:   Type *ScalarTy = Val->getType()->getScalarType();
  3632:   if (ScalarTy == getByteTy())
  3633:     return Val;
  3634: 
  3635:   if (ScalarTy != getBoolTy())
  3636:     return Builder.CreateBitCast(Val, getByteTy(getSizeOf(Val)), "cst");
  3637:   // For bool, return a sext from i1 to i8.
  3638:   if (auto *VecTy = dyn_cast<VectorType>(Val->getType()))
  3639:     return Builder.CreateSExt(Val, VectorType::get(getByteTy(), VecTy), "sxt");
  3640:   return Builder.CreateSExt(Val, getByteTy(), "sxt");
  3641: }
  3642: 
  3643: auto HexagonVectorCombine::subvector(IRBuilderBase &Builder, Value *Val,
  3644:                                      unsigned Start, unsigned Length) const
  3645:     -> Value * {
  3646:   assert(Start + Length <= length(Val));
  3647:   return getElementRange(Builder, Val, /*Ignored*/ Val, Start, Length);
  3648: }
  3649: 
  3650: auto HexagonVectorCombine::sublo(IRBuilderBase &Builder, Value *Val) const
  3651:     -> Value * {
  3652:   size_t Len = length(Val);
  3653:   assert(Len % 2 == 0 && "Length should be even");
  3654:   return subvector(Builder, Val, 0, Len / 2);
  3655: }
  3656: 
  3657: auto HexagonVectorCombine::subhi(IRBuilderBase &Builder, Value *Val) const
  3658:     -> Value * {
  3659:   size_t Len = length(Val);
  3660:   assert(Len % 2 == 0 && "Length should be even");
  3661:   return subvector(Builder, Val, Len / 2, Len / 2);
  3662: }
  3663: 
  3664: auto HexagonVectorCombine::vdeal(IRBuilderBase &Builder, Value *Val0,
  3665:                                  Value *Val1) const -> Value * {
  3666:   assert(Val0->getType() == Val1->getType());
  3667:   int Len = length(Val0);
  3668:   SmallVector<int, 128> Mask(2 * Len);
  3669: 
  3670:   for (int i = 0; i != Len; ++i) {
  3671:     Mask[i] = 2 * i;           // Even
  3672:     Mask[i + Len] = 2 * i + 1; // Odd
  3673:   }
  3674:   return Builder.CreateShuffleVector(Val0, Val1, Mask, "shf");
  3675: }
  3676: 
  3677: auto HexagonVectorCombine::vshuff(IRBuilderBase &Builder, Value *Val0,
  3678:                                   Value *Val1) const -> Value * { //
  3679:   assert(Val0->getType() == Val1->getType());
  3680:   int Len = length(Val0);
  3681:   SmallVector<int, 128> Mask(2 * Len);
  3682: 
  3683:   for (int i = 0; i != Len; ++i) {
  3684:     Mask[2 * i + 0] = i;       // Val0
  3685:     Mask[2 * i + 1] = i + Len; // Val1
  3686:   }
  3687:   return Builder.CreateShuffleVector(Val0, Val1, Mask, "shf");
  3688: }
  3689: 
  3690: auto HexagonVectorCombine::createHvxIntrinsic(IRBuilderBase &Builder,
  3691:                                               Intrinsic::ID IntID, Type *RetTy,
  3692:                                               ArrayRef<Value *> Args,
  3693:                                               ArrayRef<Type *> ArgTys,
  3694:                                               ArrayRef<Value *> MDSources) const
  3695:     -> Value * {
  3696:   auto getCast = [&](IRBuilderBase &Builder, Value *Val,
  3697:                      Type *DestTy) -> Value * {
  3698:     Type *SrcTy = Val->getType();
  3699:     if (SrcTy == DestTy)
  3700:       return Val;
  3701: 
  3702:     // Non-HVX type. It should be a scalar, and it should already have
  3703:     // a valid type.
  3704:     assert(HST.isTypeForHVX(SrcTy, /*IncludeBool=*/true));
  3705: 
  3706:     Type *BoolTy = Type::getInt1Ty(F.getContext());
  3707:     if (cast<VectorType>(SrcTy)->getElementType() != BoolTy)
  3708:       return Builder.CreateBitCast(Val, DestTy, "cst");
  3709: 
  3710:     // Predicate HVX vector.
  3711:     unsigned HwLen = HST.getVectorLength();
  3712:     Intrinsic::ID TC = HwLen == 64 ? Intrinsic::hexagon_V6_pred_typecast
  3713:                                    : Intrinsic::hexagon_V6_pred_typecast_128B;
  3714:     return Builder.CreateIntrinsic(TC, {DestTy, Val->getType()}, {Val},
  3715:                                    /*FMFSource=*/nullptr, "cup");
  3716:   };
  3717: 
  3718:   Function *IntrFn =
  3719:       Intrinsic::getOrInsertDeclaration(F.getParent(), IntID, ArgTys);
  3720:   FunctionType *IntrTy = IntrFn->getFunctionType();
  3721: 
  3722:   SmallVector<Value *, 4> IntrArgs;
  3723:   for (int i = 0, e = Args.size(); i != e; ++i) {
  3724:     Value *A = Args[i];
  3725:     Type *T = IntrTy->getParamType(i);
  3726:     if (A->getType() != T) {
  3727:       IntrArgs.push_back(getCast(Builder, A, T));
  3728:     } else {
  3729:       IntrArgs.push_back(A);
  3730:     }
  3731:   }
  3732:   StringRef MaybeName = !IntrTy->getReturnType()->isVoidTy() ? "cup" : "";
  3733:   CallInst *Call = Builder.CreateCall(IntrFn, IntrArgs, MaybeName);
  3734: 
  3735:   MemoryEffects ME = Call->getAttributes().getMemoryEffects();
  3736:   if (!ME.doesNotAccessMemory() && !ME.onlyAccessesInaccessibleMem())
  3737:     propagateMetadata(Call, MDSources);
  3738: 
  3739:   Type *CallTy = Call->getType();
  3740:   if (RetTy == nullptr || CallTy == RetTy)
  3741:     return Call;
  3742:   // Scalar types should have RetTy matching the call return type.
  3743:   assert(HST.isTypeForHVX(CallTy, /*IncludeBool=*/true));
  3744:   return getCast(Builder, Call, RetTy);
  3745: }
  3746: 
  3747: auto HexagonVectorCombine::splitVectorElements(IRBuilderBase &Builder,
  3748:                                                Value *Vec,
  3749:                                                unsigned ToWidth) const
  3750:     -> SmallVector<Value *> {
  3751:   // Break a vector of wide elements into a series of vectors with narrow
  3752:   // elements:
  3753:   //   (...c0:b0:a0, ...c1:b1:a1, ...c2:b2:a2, ...)
  3754:   // -->
  3755:   //   (a0, a1, a2, ...)    // lowest "ToWidth" bits
  3756:   //   (b0, b1, b2, ...)    // the next lowest...
  3757:   //   (c0, c1, c2, ...)    // ...
  3758:   //   ...
  3759:   //
  3760:   // The number of elements in each resulting vector is the same as
  3761:   // in the original vector.
  3762: 
  3763:   auto *VecTy = cast<VectorType>(Vec->getType());
  3764:   assert(VecTy->getElementType()->isIntegerTy());
  3765:   unsigned FromWidth = VecTy->getScalarSizeInBits();
  3766:   assert(isPowerOf2_32(ToWidth) && isPowerOf2_32(FromWidth));
  3767:   assert(ToWidth <= FromWidth && "Breaking up into wider elements?");
  3768:   unsigned NumResults = FromWidth / ToWidth;
  3769: 
  3770:   SmallVector<Value *> Results(NumResults);
  3771:   Results[0] = Vec;
  3772:   unsigned Length = length(VecTy);
  3773: 
  3774:   // Do it by splitting in half, since those operations correspond to deal
  3775:   // instructions.
  3776:   auto splitInHalf = [&](unsigned Begin, unsigned End, auto splitFunc) -> void {
  3777:     // Take V = Results[Begin], split it in L, H.
  3778:     // Store Results[Begin] = L, Results[(Begin+End)/2] = H
  3779:     // Call itself recursively split(Begin, Half), split(Half+1, End)
  3780:     if (Begin + 1 == End)
  3781:       return;
  3782: 
  3783:     Value *Val = Results[Begin];
  3784:     unsigned Width = Val->getType()->getScalarSizeInBits();
  3785: 
  3786:     auto *VTy = VectorType::get(getIntTy(Width / 2), 2 * Length, false);
  3787:     Value *VVal = Builder.CreateBitCast(Val, VTy, "cst");
  3788: 
  3789:     Value *Res = vdeal(Builder, sublo(Builder, VVal), subhi(Builder, VVal));
  3790: 
  3791:     unsigned Half = (Begin + End) / 2;
  3792:     Results[Begin] = sublo(Builder, Res);
  3793:     Results[Half] = subhi(Builder, Res);
  3794: 
  3795:     splitFunc(Begin, Half, splitFunc);
  3796:     splitFunc(Half, End, splitFunc);
  3797:   };
  3798: 
  3799:   splitInHalf(0, NumResults, splitInHalf);
  3800:   return Results;
```
- EN: It declares or implements routines such as getIntTy, CreateSExt, CreateBitCast, CreateTrunc, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonVectorCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getIntTy, CreateSExt, CreateBitCast, CreateTrunc, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonVectorCombine，说明了它与同级后端组件的连接关系。

### Lines 3801-4000 / 第 3801-4000 行

```cpp
  3801: }
  3802: 
  3803: auto HexagonVectorCombine::joinVectorElements(IRBuilderBase &Builder,
  3804:                                               ArrayRef<Value *> Values,
  3805:                                               VectorType *ToType) const
  3806:     -> Value * {
  3807:   assert(ToType->getElementType()->isIntegerTy());
  3808: 
  3809:   // If the list of values does not have power-of-2 elements, append copies
  3810:   // of the sign bit to it, to make the size be 2^n.
  3811:   // The reason for this is that the values will be joined in pairs, because
  3812:   // otherwise the shuffles will result in convoluted code. With pairwise
  3813:   // joins, the shuffles will hopefully be folded into a perfect shuffle.
  3814:   // The output will need to be sign-extended to a type with element width
  3815:   // being a power-of-2 anyways.
  3816:   SmallVector<Value *> Inputs(Values);
  3817: 
  3818:   unsigned ToWidth = ToType->getScalarSizeInBits();
  3819:   unsigned Width = Inputs.front()->getType()->getScalarSizeInBits();
  3820:   assert(Width <= ToWidth);
  3821:   assert(isPowerOf2_32(Width) && isPowerOf2_32(ToWidth));
  3822:   unsigned Length = length(Inputs.front()->getType());
  3823: 
  3824:   unsigned NeedInputs = ToWidth / Width;
  3825:   if (Inputs.size() != NeedInputs) {
  3826:     // Having too many inputs is ok: drop the high bits (usual wrap-around).
  3827:     // If there are too few, fill them with the sign bit.
  3828:     Value *Last = Inputs.back();
  3829:     Value *Sign = Builder.CreateAShr(
  3830:         Last, ConstantInt::get(Last->getType(), Width - 1), "asr");
  3831:     Inputs.resize(NeedInputs, Sign);
  3832:   }
  3833: 
  3834:   while (Inputs.size() > 1) {
  3835:     Width *= 2;
  3836:     auto *VTy = VectorType::get(getIntTy(Width), Length, false);
  3837:     for (int i = 0, e = Inputs.size(); i < e; i += 2) {
  3838:       Value *Res = vshuff(Builder, Inputs[i], Inputs[i + 1]);
  3839:       Inputs[i / 2] = Builder.CreateBitCast(Res, VTy, "cst");
  3840:     }
  3841:     Inputs.resize(Inputs.size() / 2);
  3842:   }
  3843: 
  3844:   assert(Inputs.front()->getType() == ToType);
  3845:   return Inputs.front();
  3846: }
  3847: 
  3848: auto HexagonVectorCombine::calculatePointerDifference(Value *Ptr0,
  3849:                                                       Value *Ptr1) const
  3850:     -> std::optional<int> {
  3851:   // Try SCEV first.
  3852:   const SCEV *Scev0 = SE.getSCEV(Ptr0);
  3853:   const SCEV *Scev1 = SE.getSCEV(Ptr1);
  3854:   const SCEV *ScevDiff = SE.getMinusSCEV(Scev0, Scev1);
  3855:   if (auto *Const = dyn_cast<SCEVConstant>(ScevDiff)) {
  3856:     APInt V = Const->getAPInt();
  3857:     if (V.isSignedIntN(8 * sizeof(int)))
  3858:       return static_cast<int>(V.getSExtValue());
  3859:   }
  3860: 
  3861:   struct Builder : IRBuilder<> {
  3862:     Builder(BasicBlock *B) : IRBuilder<>(B->getTerminator()) {}
  3863:     ~Builder() {
  3864:       for (Instruction *I : llvm::reverse(ToErase))
  3865:         I->eraseFromParent();
  3866:     }
  3867:     SmallVector<Instruction *, 8> ToErase;
  3868:   };
  3869: 
  3870: #define CallBuilder(B, F)                                                      \
  3871:   [&](auto &B_) {                                                              \
  3872:     Value *V = B_.F;                                                           \
  3873:     if (auto *I = dyn_cast<Instruction>(V))                                    \
  3874:       B_.ToErase.push_back(I);                                                 \
  3875:     return V;                                                                  \
  3876:   }(B)
  3877: 
  3878:   auto Simplify = [this](Value *V) {
  3879:     if (Value *S = simplify(V))
  3880:       return S;
  3881:     return V;
  3882:   };
  3883: 
  3884:   auto StripBitCast = [](Value *V) {
  3885:     while (auto *C = dyn_cast<BitCastInst>(V))
  3886:       V = C->getOperand(0);
  3887:     return V;
  3888:   };
  3889: 
  3890:   Ptr0 = StripBitCast(Ptr0);
  3891:   Ptr1 = StripBitCast(Ptr1);
  3892:   if (!isa<GetElementPtrInst>(Ptr0) || !isa<GetElementPtrInst>(Ptr1))
  3893:     return std::nullopt;
  3894: 
  3895:   auto *Gep0 = cast<GetElementPtrInst>(Ptr0);
  3896:   auto *Gep1 = cast<GetElementPtrInst>(Ptr1);
  3897:   if (Gep0->getPointerOperand() != Gep1->getPointerOperand())
  3898:     return std::nullopt;
  3899:   if (Gep0->getSourceElementType() != Gep1->getSourceElementType())
  3900:     return std::nullopt;
  3901: 
  3902:   Builder B(Gep0->getParent());
  3903:   int Scale = getSizeOf(Gep0->getSourceElementType(), Alloc);
  3904: 
  3905:   // FIXME: for now only check GEPs with a single index.
  3906:   if (Gep0->getNumOperands() != 2 || Gep1->getNumOperands() != 2)
  3907:     return std::nullopt;
  3908: 
  3909:   Value *Idx0 = Gep0->getOperand(1);
  3910:   Value *Idx1 = Gep1->getOperand(1);
  3911: 
  3912:   // First, try to simplify the subtraction directly.
  3913:   if (auto *Diff = dyn_cast<ConstantInt>(
  3914:           Simplify(CallBuilder(B, CreateSub(Idx0, Idx1)))))
  3915:     return Diff->getSExtValue() * Scale;
  3916: 
  3917:   KnownBits Known0 = getKnownBits(Idx0, Gep0);
  3918:   KnownBits Known1 = getKnownBits(Idx1, Gep1);
  3919:   APInt Unknown = ~(Known0.Zero | Known0.One) | ~(Known1.Zero | Known1.One);
  3920:   if (Unknown.isAllOnes())
  3921:     return std::nullopt;
  3922: 
  3923:   Value *MaskU = ConstantInt::get(Idx0->getType(), Unknown);
  3924:   Value *AndU0 = Simplify(CallBuilder(B, CreateAnd(Idx0, MaskU)));
  3925:   Value *AndU1 = Simplify(CallBuilder(B, CreateAnd(Idx1, MaskU)));
  3926:   Value *SubU = Simplify(CallBuilder(B, CreateSub(AndU0, AndU1)));
  3927:   int Diff0 = 0;
  3928:   if (auto *C = dyn_cast<ConstantInt>(SubU)) {
  3929:     Diff0 = C->getSExtValue();
  3930:   } else {
  3931:     return std::nullopt;
  3932:   }
  3933: 
  3934:   Value *MaskK = ConstantInt::get(MaskU->getType(), ~Unknown);
  3935:   Value *AndK0 = Simplify(CallBuilder(B, CreateAnd(Idx0, MaskK)));
  3936:   Value *AndK1 = Simplify(CallBuilder(B, CreateAnd(Idx1, MaskK)));
  3937:   Value *SubK = Simplify(CallBuilder(B, CreateSub(AndK0, AndK1)));
  3938:   int Diff1 = 0;
  3939:   if (auto *C = dyn_cast<ConstantInt>(SubK)) {
  3940:     Diff1 = C->getSExtValue();
  3941:   } else {
  3942:     return std::nullopt;
  3943:   }
  3944: 
  3945:   return (Diff0 + Diff1) * Scale;
  3946: 
  3947: #undef CallBuilder
  3948: }
  3949: 
  3950: auto HexagonVectorCombine::getNumSignificantBits(const Value *V,
  3951:                                                  const Instruction *CtxI) const
  3952:     -> unsigned {
  3953:   return ComputeMaxSignificantBits(V, DL, &AC, CtxI, &DT);
  3954: }
  3955: 
  3956: auto HexagonVectorCombine::getKnownBits(const Value *V,
  3957:                                         const Instruction *CtxI) const
  3958:     -> KnownBits {
  3959:   return computeKnownBits(V, DL, &AC, CtxI, &DT);
  3960: }
  3961: 
  3962: auto HexagonVectorCombine::isSafeToClone(const Instruction &In) const -> bool {
  3963:   if (In.mayHaveSideEffects() || In.isAtomic() || In.isVolatile() ||
  3964:       In.isFenceLike() || In.mayReadOrWriteMemory()) {
  3965:     return false;
  3966:   }
  3967:   if (isa<CallBase>(In) || isa<AllocaInst>(In))
  3968:     return false;
  3969:   return true;
  3970: }
  3971: 
  3972: template <typename T>
  3973: auto HexagonVectorCombine::isSafeToMoveBeforeInBB(const Instruction &In,
  3974:                                                   BasicBlock::const_iterator To,
  3975:                                                   const T &IgnoreInsts) const
  3976:     -> bool {
  3977:   auto getLocOrNone =
  3978:       [this](const Instruction &I) -> std::optional<MemoryLocation> {
  3979:     if (const auto *II = dyn_cast<IntrinsicInst>(&I)) {
  3980:       switch (II->getIntrinsicID()) {
  3981:       case Intrinsic::masked_load:
  3982:         return MemoryLocation::getForArgument(II, 0, TLI);
  3983:       case Intrinsic::masked_store:
  3984:         return MemoryLocation::getForArgument(II, 1, TLI);
  3985:       }
  3986:     }
  3987:     return MemoryLocation::getOrNone(&I);
  3988:   };
  3989: 
  3990:   // The source and the destination must be in the same basic block.
  3991:   const BasicBlock &Block = *In.getParent();
  3992:   assert(Block.begin() == To || Block.end() == To || To->getParent() == &Block);
  3993:   // No PHIs.
  3994:   if (isa<PHINode>(In) || (To != Block.end() && isa<PHINode>(*To)))
  3995:     return false;
  3996: 
  3997:   if (!mayHaveNonDefUseDependency(In))
  3998:     return true;
  3999:   bool MayWrite = In.mayWriteToMemory();
  4000:   auto MaybeLoc = getLocOrNone(In);
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares types such as Builder, which carry the state or API of this component. It declares or implements routines such as assert, Inputs, getScalarSizeInBits, front, ... (32 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明了 Builder 等类型，用来承载该组件的状态或接口。 这里声明或实现了 assert, Inputs, getScalarSizeInBits, front, ... (32 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 4001-4108 / 第 4001-4108 行

```cpp
  4001: 
  4002:   auto From = In.getIterator();
  4003:   if (From == To)
  4004:     return true;
  4005:   bool MoveUp = (To != Block.end() && To->comesBefore(&In));
  4006:   auto Range =
  4007:       MoveUp ? std::make_pair(To, From) : std::make_pair(std::next(From), To);
  4008:   for (auto It = Range.first; It != Range.second; ++It) {
  4009:     const Instruction &I = *It;
  4010:     if (llvm::is_contained(IgnoreInsts, &I))
  4011:       continue;
  4012:     // assume intrinsic can be ignored
  4013:     if (auto *II = dyn_cast<IntrinsicInst>(&I)) {
  4014:       if (II->getIntrinsicID() == Intrinsic::assume)
  4015:         continue;
  4016:     }
  4017:     // Parts based on isSafeToMoveBefore from CoveMoverUtils.cpp.
  4018:     if (I.mayThrow())
  4019:       return false;
  4020:     if (auto *CB = dyn_cast<CallBase>(&I)) {
  4021:       if (!CB->hasFnAttr(Attribute::WillReturn))
  4022:         return false;
  4023:       if (!CB->hasFnAttr(Attribute::NoSync))
  4024:         return false;
  4025:     }
  4026:     if (I.mayReadOrWriteMemory()) {
  4027:       auto MaybeLocI = getLocOrNone(I);
  4028:       if (MayWrite || I.mayWriteToMemory()) {
  4029:         if (!MaybeLoc || !MaybeLocI)
  4030:           return false;
  4031:         if (!AA.isNoAlias(*MaybeLoc, *MaybeLocI))
  4032:           return false;
  4033:       }
  4034:     }
  4035:   }
  4036:   return true;
  4037: }
  4038: 
  4039: auto HexagonVectorCombine::isByteVecTy(Type *Ty) const -> bool {
  4040:   if (auto *VecTy = dyn_cast<VectorType>(Ty))
  4041:     return VecTy->getElementType() == getByteTy();
  4042:   return false;
  4043: }
  4044: 
  4045: auto HexagonVectorCombine::getElementRange(IRBuilderBase &Builder, Value *Lo,
  4046:                                            Value *Hi, int Start,
  4047:                                            int Length) const -> Value * {
  4048:   assert(0 <= Start && size_t(Start + Length) < length(Lo) + length(Hi));
  4049:   SmallVector<int, 128> SMask(Length);
  4050:   std::iota(SMask.begin(), SMask.end(), Start);
  4051:   return Builder.CreateShuffleVector(Lo, Hi, SMask, "shf");
  4052: }
  4053: 
  4054: // Pass management.
  4055: 
  4056: namespace {
  4057: class HexagonVectorCombineLegacy : public FunctionPass {
  4058: public:
  4059:   static char ID;
  4060: 
  4061:   HexagonVectorCombineLegacy() : FunctionPass(ID) {}
  4062: 
  4063:   StringRef getPassName() const override { return "Hexagon Vector Combine"; }
  4064: 
  4065:   void getAnalysisUsage(AnalysisUsage &AU) const override {
  4066:     AU.setPreservesCFG();
  4067:     AU.addRequired<AAResultsWrapperPass>();
  4068:     AU.addRequired<AssumptionCacheTracker>();
  4069:     AU.addRequired<DominatorTreeWrapperPass>();
  4070:     AU.addRequired<ScalarEvolutionWrapperPass>();
  4071:     AU.addRequired<TargetLibraryInfoWrapperPass>();
  4072:     AU.addRequired<TargetPassConfig>();
  4073:     FunctionPass::getAnalysisUsage(AU);
  4074:   }
  4075: 
  4076:   bool runOnFunction(Function &F) override {
  4077:     if (skipFunction(F))
  4078:       return false;
  4079:     AliasAnalysis &AA = getAnalysis<AAResultsWrapperPass>().getAAResults();
  4080:     AssumptionCache &AC =
  4081:         getAnalysis<AssumptionCacheTracker>().getAssumptionCache(F);
  4082:     DominatorTree &DT = getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  4083:     ScalarEvolution &SE = getAnalysis<ScalarEvolutionWrapperPass>().getSE();
  4084:     TargetLibraryInfo &TLI =
  4085:         getAnalysis<TargetLibraryInfoWrapperPass>().getTLI(F);
  4086:     auto &TM = getAnalysis<TargetPassConfig>().getTM<HexagonTargetMachine>();
  4087:     HexagonVectorCombine HVC(F, AA, AC, DT, SE, TLI, TM);
  4088:     return HVC.run();
  4089:   }
  4090: };
  4091: } // namespace
  4092: 
  4093: char HexagonVectorCombineLegacy::ID = 0;
  4094: 
  4095: INITIALIZE_PASS_BEGIN(HexagonVectorCombineLegacy, DEBUG_TYPE,
  4096:                       "Hexagon Vector Combine", false, false)
  4097: INITIALIZE_PASS_DEPENDENCY(AAResultsWrapperPass)
  4098: INITIALIZE_PASS_DEPENDENCY(AssumptionCacheTracker)
  4099: INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
  4100: INITIALIZE_PASS_DEPENDENCY(ScalarEvolutionWrapperPass)
  4101: INITIALIZE_PASS_DEPENDENCY(TargetLibraryInfoWrapperPass)
  4102: INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
  4103: INITIALIZE_PASS_END(HexagonVectorCombineLegacy, DEBUG_TYPE,
  4104:                     "Hexagon Vector Combine", false, false)
  4105: 
  4106: FunctionPass *llvm::createHexagonVectorCombineLegacyPass() {
  4107:   return new HexagonVectorCombineLegacy();
  4108: }
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonVectorCombineLegacy, which carry the state or API of this component. It defines declarative TableGen records like HexagonVectorCombineLegacy; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getIterator, end, std::make_pair, getLocOrNone, ... (29 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonVectorCombineLegacy 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonVectorCombineLegacy 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getIterator, end, std::make_pair, getLocOrNone, ... (29 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制
- pattern-driven instruction selection / 基于模式的指令选择
- intrinsic mapping / Intrinsic 映射

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/ADT/APInt.h, llvm/ADT/ArrayRef.h, llvm/ADT/DenseMap.h, llvm/ADT/MapVector.h, llvm/ADT/STLExtras.h, llvm/ADT/SmallVector.h, llvm/Analysis/AliasAnalysis.h, llvm/Analysis/AssumeBundleQueries.h, llvm/Analysis/AssumptionCache.h, llvm/Analysis/InstSimplifyFolder.h, ... (42 total)`
- Hexagon symbols / Hexagon 符号: `HexagonVectorCombine, HexagonSubtarget, HexagonTargetMachine, HexagonVectorCombineLegacy`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
