# AxisInfo.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Analysis/AxisInfo.h`
- **EN:** Declares analysis helpers centered on `AxisInfo`.
- **CN:** 声明围绕 `AxisInfo` 的分析辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_ANALYSIS_AXISINFO_H
   2: #define TRITON_ANALYSIS_AXISINFO_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-5
```cpp
   4: #include "mlir/Analysis/DataFlow/SparseAnalysis.h"
   5: #include "llvm/Support/raw_ostream.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Analysis/DataFlow/SparseAnalysis.h and llvm/Support/raw_ostream.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Analysis/DataFlow/SparseAnalysis.h and llvm/Support/raw_ostream.h。

### Lines 7-8
```cpp
   7: #include "mlir/Support/LLVM.h"
   8: #include "triton/Analysis/Utility.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Support/LLVM.h and triton/Analysis/Utility.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Support/LLVM.h and triton/Analysis/Utility.h。

### Lines 10-10
```cpp
  10: #include <optional>
```
**EN:** This block imports the direct dependencies needed here, including <optional>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 <optional>。

### Lines 12-12
```cpp
  12: namespace mlir::triton {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir::triton.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir::triton 下。

### Lines 14-16
```cpp
  14: //===----------------------------------------------------------------------===//
  15: // AxisInfo
  16: //===----------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===----------------------------------------------------------------------===// AxisInfo ===----------------------------------------------------------------------===//.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 18-21
```cpp
  18: /// This lattice value represents known information on the axes of a lattice.
  19: class AxisInfo {
  20: public:
  21:   typedef SmallVector<int64_t> DimVectorT;
```
**EN:** This block introduces `AxisInfo`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `AxisInfo`。

### Lines 23-24
```cpp
  23: public:
  24:   AxisInfo() : AxisInfo({}, {}, {}) {}
```
**EN:** This block declares or defines callable APIs such as AxisInfo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 AxisInfo 等可调用 API，用来封装这里提供的核心行为。

### Lines 26-28
```cpp
  26:   AxisInfo(ArrayRef<int64_t> contiguity, ArrayRef<int64_t> divisibility,
  27:            ArrayRef<int64_t> constancy)
  28:       : AxisInfo(contiguity, divisibility, constancy, std::nullopt) {}
```
**EN:** This block declares or defines callable APIs such as AxisInfo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 AxisInfo 等可调用 API，用来封装这里提供的核心行为。

### Lines 30-36
```cpp
  30:   AxisInfo(ArrayRef<int64_t> contiguity, ArrayRef<int64_t> divisibility,
  31:            ArrayRef<int64_t> constancy, std::optional<int64_t> constantValue)
  32:       : contiguity(contiguity), divisibility(divisibility),
  33:         constancy(constancy), constantValue(constantValue) {
  34:     assert(divisibility.size() == contiguity.size());
  35:     assert(constancy.size() == contiguity.size());
  36:   }
```
**EN:** This block declares or defines callable APIs such as AxisInfo, contiguity, divisibility, constancy, constantValue, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 AxisInfo, contiguity, divisibility, constancy, constantValue, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 38-61
```cpp
  38:   // contiguity[d] is the length of the shortest sequence of contiguous integers
  39:   // along dimension d.
  40:   //
  41:   // If we have an array of N elements with a contiguity value C, then the array
  42:   // can be divided into a list of N/C sequences of C contiguous elements.
  43:   // Since we have N = 2^k, C must be a power of two.
  44:   //
  45:   // For example, the 2D array
  46:   //
  47:   //   [[10, 11, 12, 13, 18, 19, 20, 21],
  48:   //    [20, 21, 22, 23, 28, 29, 30, 31]]
  49:   //
  50:   // has contiguity [1, 4], and
  51:   //
  52:   //   [[12, 16, 20, 24],
  53:   //    [13, 17, 21, 25],
  54:   //    [14, 18, 22, 26],
  55:   //    [15, 19, 23, 27],
  56:   //    [18, 22, 26, 30],
  57:   //    [19, 23, 27, 31]]
  58:   //
  59:   // has contiguity [2, 1].
  60:   int64_t getContiguity(size_t dim) const { return contiguity[dim]; }
  61:   const DimVectorT &getContiguity() const { return contiguity; }
```
**EN:** This block declares or defines callable APIs such as getContiguity, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getContiguity 等可调用 API，用来封装这里提供的核心行为。

### Lines 63-86
```cpp
  63:   // divisibility[d] is the largest power of two that divides the first element
  64:   // of all groups of length contiguity[d] along dimension d.
  65:   //
  66:   // For example,
  67:   //
  68:   //   [[10, 11, 12, 13, 18, 19, 20, 21],
  69:   //    [20, 21, 22, 23, 28, 29, 30, 31]]
  70:   //
  71:   //  has divisibility [1, 2], and
  72:   //
  73:   //    [[12, 16, 20, 24],
  74:   //     [13, 17, 21, 25],
  75:   //     [14, 18, 22, 26],
  76:   //     [15, 19, 23, 27]]
  77:   //
  78:   // has divisibility [4, 1].
  79:   //
  80:   // On the other hand,
  81:   //
  82:   //   [0, 1, 2, 0, 4, 5, 6, 7]
  83:   //
  84:   // has divisibility 1 because its contiguity is 1.
  85:   int64_t getDivisibility(size_t dim) const { return divisibility[dim]; }
  86:   const DimVectorT &getDivisibility() const { return divisibility; }
```
**EN:** This block declares or defines callable APIs such as getDivisibility, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getDivisibility 等可调用 API，用来封装这里提供的核心行为。

### Lines 88-105
```cpp
  88:   // constancy[d] is the length of the shortest sequence of repeating integers
  89:   // along dimension d.
  90:   //
  91:   // This is particularly useful to infer the contiguity of operations (e.g.
  92:   // add) involving a constant.
  93:   //
  94:   // If we have an array of N elements, with a constancy value C, then the array
  95:   // can be divided into a list of N/C sequences of C elements with the same
  96:   // value.  Since we have N = 2^k, C must be a power of two.
  97:   //
  98:   // For example
  99:   //
 100:   //   [[8, 8, 8, 8, 12, 12, 12, 12],
 101:   //    [16, 16, 16, 16, 20, 20, 20, 20]]
 102:   //
 103:   // has constancy [1, 4].
 104:   int64_t getConstancy(size_t dim) const { return constancy[dim]; }
 105:   const DimVectorT &getConstancy() const { return constancy; }
```
**EN:** This block declares or defines callable APIs such as getConstancy, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getConstancy 等可调用 API，用来封装这里提供的核心行为。

### Lines 107-107
```cpp
 107:   int getRank() const { return contiguity.size(); }
```
**EN:** This block declares or defines callable APIs such as getRank and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getRank and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 109-109
```cpp
 109:   std::optional<int64_t> getConstantValue() const { return constantValue; }
```
**EN:** This block declares or defines callable APIs such as getConstantValue, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getConstantValue 等可调用 API，用来封装这里提供的核心行为。

### Lines 111-115
```cpp
 111:   static void initPessimisticStateFromFunc(int argNumber,
 112:                                            FunctionOpInterface funcOp,
 113:                                            DimVectorT *contiguity,
 114:                                            DimVectorT *divisibility,
 115:                                            DimVectorT *constancy);
```
**EN:** This block declares or defines callable APIs such as initPessimisticStateFromFunc, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 initPessimisticStateFromFunc 等可调用 API，用来封装这里提供的核心行为。

### Lines 117-117
```cpp
 117:   static void initDimVectorFromHint(Attribute attr, DimVectorT *vec);
```
**EN:** This block declares or defines callable APIs such as initDimVectorFromHint, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 initDimVectorFromHint 等可调用 API，用来封装这里提供的核心行为。

### Lines 119-123
```cpp
 119:   bool operator==(const AxisInfo &other) const {
 120:     return contiguity == other.contiguity &&
 121:            divisibility == other.divisibility && constancy == other.constancy &&
 122:            constantValue == other.constantValue;
 123:   }
```
**EN:** This block stores supporting state such as constantValue, which other APIs in the file consume.
**CN:** 该代码块声明了 constantValue 等支撑状态，供本文件中的其他 API 使用。

### Lines 125-125
```cpp
 125:   static AxisInfo getPessimisticValueState(Value value);
```
**EN:** This block declares or defines callable APIs such as getPessimisticValueState, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPessimisticValueState 等可调用 API，用来封装这里提供的核心行为。

### Lines 127-128
```cpp
 127:   // The gcd of both arguments for each dimension
 128:   static AxisInfo join(const AxisInfo &lhs, const AxisInfo &rhs);
```
**EN:** This block declares or defines callable APIs such as join, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 join 等可调用 API，用来封装这里提供的核心行为。

### Lines 130-144
```cpp
 130:   void print(raw_ostream &os) const {
 131:     auto print = [&](StringRef name, DimVectorT vec) {
 132:       os << name << " = [";
 133:       llvm::interleaveComma(vec, os);
 134:       os << "]";
 135:     };
 136:     print("contiguity", contiguity);
 137:     print(", divisibility", divisibility);
 138:     print(", constancy", constancy);
 139:     os << ", constant_value = ";
 140:     if (constantValue)
 141:       os << *constantValue;
 142:     else
 143:       os << "<none>";
 144:   }
```
**EN:** This block declares or defines callable APIs such as print and interleaveComma, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 print and interleaveComma 等可调用 API，用来封装这里提供的核心行为。

### Lines 146-149
```cpp
 146: private:
 147:   DimVectorT contiguity;
 148:   DimVectorT divisibility;
 149:   DimVectorT constancy;
```
**EN:** This block stores supporting state such as contiguity, divisibility, and constancy, which other APIs in the file consume.
**CN:** 该代码块声明了 contiguity, divisibility, and constancy 等支撑状态，供本文件中的其他 API 使用。

### Lines 151-153
```cpp
 151:   // The constant value of the lattice if we can infer it.
 152:   std::optional<int64_t> constantValue;
 153: };
```
**EN:** This block stores supporting state such as constantValue, which other APIs in the file consume.
**CN:** 该代码块声明了 constantValue 等支撑状态，供本文件中的其他 API 使用。

### Lines 155-158
```cpp
 155: class AxisInfoVisitor {
 156: public:
 157:   AxisInfoVisitor() = default;
 158:   virtual ~AxisInfoVisitor() = default;
```
**EN:** This block introduces `AxisInfoVisitor`, the main class/struct defined here. Within the declaration, methods such as ~AxisInfoVisitor expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `AxisInfoVisitor`。 其中 ~AxisInfoVisitor 等方法构成了它的主要接口。

### Lines 160-162
```cpp
 160:   bool isContiguousDim(const AxisInfo &info, ArrayRef<int64_t> shape, int dim) {
 161:     return info.getContiguity(dim) == shape[dim];
 162:   }
```
**EN:** This block declares or defines callable APIs such as isContiguousDim and getContiguity, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isContiguousDim and getContiguity 等可调用 API，用来封装这里提供的核心行为。

### Lines 164-166
```cpp
 164:   bool isConstantDim(const AxisInfo &info, ArrayRef<int64_t> shape, int dim) {
 165:     return info.getConstancy(dim) == shape[dim];
 166:   }
```
**EN:** This block declares or defines callable APIs such as isConstantDim and getConstancy, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isConstantDim and getConstancy 等可调用 API，用来封装这里提供的核心行为。

### Lines 168-170
```cpp
 168:   virtual AxisInfo
 169:   getAxisInfo(Operation *op,
 170:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) = 0;
```
**EN:** This block declares or defines callable APIs such as getAxisInfo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getAxisInfo 等可调用 API，用来封装这里提供的核心行为。

### Lines 172-173
```cpp
 172:   virtual bool match(Operation *op) = 0;
 173: };
```
**EN:** This block declares or defines callable APIs such as match, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 match 等可调用 API，用来封装这里提供的核心行为。

### Lines 175-180
```cpp
 175: class AxisInfoVisitorList {
 176: public:
 177:   template <typename... Ts, typename = std::enable_if_t<sizeof...(Ts) != 0>>
 178:   void append() {
 179:     (visitors.emplace_back(std::make_unique<Ts>()), ...);
 180:   }
```
**EN:** This block introduces `AxisInfoVisitorList`, the main class/struct defined here. Within the declaration, methods such as append and emplace_back expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `AxisInfoVisitorList`。 其中 append and emplace_back 等方法构成了它的主要接口。

### Lines 182-188
```cpp
 182:   AxisInfo apply(Operation *op,
 183:                  ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) {
 184:     for (auto &visitor : visitors)
 185:       if (visitor->match(op))
 186:         return visitor->getAxisInfo(op, operands);
 187:     return AxisInfo();
 188:   }
```
**EN:** This block declares or defines callable APIs such as apply, match, getAxisInfo, and AxisInfo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 apply, match, getAxisInfo, and AxisInfo 等可调用 API，用来封装这里提供的核心行为。

### Lines 190-192
```cpp
 190: private:
 191:   std::vector<std::unique_ptr<AxisInfoVisitor>> visitors;
 192: };
```
**EN:** This block stores supporting state such as visitors, which other APIs in the file consume.
**CN:** 该代码块声明了 visitors 等支撑状态，供本文件中的其他 API 使用。

### Lines 194-197
```cpp
 194: class AxisInfoAnalysis : public dataflow::SparseForwardDataFlowAnalysis<
 195:                              dataflow::Lattice<AxisInfo>> {
 196: protected:
 197:   AxisInfoVisitorList visitors;
```
**EN:** This block introduces `AxisInfoAnalysis`, the main class/struct defined here. It also inherits behavior from a base type.
**CN:** 该代码块引入了此文件的核心类/结构体 `AxisInfoAnalysis`。 它还通过继承复用基类能力。

### Lines 199-199
```cpp
 199:   void setToEntryState(dataflow::Lattice<AxisInfo> *lattice) override;
```
**EN:** This block declares or defines callable APIs such as setToEntryState, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setToEntryState 等可调用 API，用来封装这里提供的核心行为。

### Lines 201-204
```cpp
 201:   void visitNonControlFlowArguments(
 202:       Operation *op, const RegionSuccessor & /*successor*/,
 203:       ValueRange /*nonSuccessorInputs*/,
 204:       ArrayRef<dataflow::Lattice<AxisInfo> *> argLattices) override;
```
**EN:** This block declares or defines callable APIs such as visitNonControlFlowArguments, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 visitNonControlFlowArguments 等可调用 API，用来封装这里提供的核心行为。

### Lines 206-208
```cpp
 206:   void
 207:   visitForOpInductionVar(scf::ForOp op,
 208:                          ArrayRef<dataflow::Lattice<AxisInfo> *> argLattices);
```
**EN:** This block declares or defines callable APIs such as visitForOpInductionVar, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 visitForOpInductionVar 等可调用 API，用来封装这里提供的核心行为。

### Lines 210-213
```cpp
 210: public:
 211:   AxisInfoAnalysis(DataFlowSolver &solver);
 212:   using dataflow::SparseForwardDataFlowAnalysis<
 213:       dataflow::Lattice<AxisInfo>>::getLatticeElement;
```
**EN:** This block declares or defines callable APIs such as AxisInfoAnalysis, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 AxisInfoAnalysis 等可调用 API，用来封装这里提供的核心行为。

### Lines 215-218
```cpp
 215:   LogicalResult
 216:   visitOperation(Operation *op,
 217:                  ArrayRef<const dataflow::Lattice<AxisInfo> *> operands,
 218:                  ArrayRef<dataflow::Lattice<AxisInfo> *> results) override;
```
**EN:** This block declares or defines callable APIs such as visitOperation, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 visitOperation 等可调用 API，用来封装这里提供的核心行为。

### Lines 220-222
```cpp
 220:   static AxisInfoAnalysis *loadDefaultAnalysis(DataFlowSolver *solver);
 221:   using LoadCallback = decltype(&AxisInfoAnalysis::loadDefaultAnalysis);
 222: };
```
**EN:** This block declares or defines callable APIs such as loadDefaultAnalysis, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 loadDefaultAnalysis 等可调用 API，用来封装这里提供的核心行为。

### Lines 224-260
```cpp
 224: // Module level axis info analysis based on the call graph, assuming that we do
 225: // not have recursive functions.
 226: //
 227: // Since each function will be called multiple times, we need to calculate the
 228: // axis info based on the axis info of all the callers.  In the future, we can
 229: // perform optimization using function cloning so that each call site will have
 230: // unique axis info.
 231: using AxisInfoMapT = DenseMap<Value, AxisInfo>;
 232: class ModuleAxisInfoAnalysis : public CallGraph<AxisInfoMapT> {
 233: public:
 234:   // AxisInfoAnalysis::LoadCallback loads the per-function analysis pass into
 235:   // the DataFlowSolver. This allows passes derived from AxisInfoAnalysis to
 236:   // re-use the module level analysis framework.
 237:   explicit ModuleAxisInfoAnalysis(ModuleOp moduleOp,
 238:                                   AxisInfoAnalysis::LoadCallback loadAnalysis =
 239:                                       AxisInfoAnalysis::loadDefaultAnalysis)
 240:       : CallGraph<AxisInfoMapT>(moduleOp) {
 241:     SmallVector<FunctionOpInterface> funcs;
 242:     walk<WalkOrder::PreOrder, WalkOrder::PostOrder>(
 243:         // Pre-order edge walk callback
 244:         [](CallOpInterface callOp, FunctionOpInterface funcOp) {},
 245:         // Post-order node walk callback
 246:         [&](FunctionOpInterface funcOp) {
 247:           funcs.push_back(funcOp);
 248:           funcMap.try_emplace(funcOp, AxisInfoMapT{});
 249:         });
 250:     SetVector<FunctionOpInterface> sortedFuncs(funcs.begin(), funcs.end());
 251:     SymbolTableCollection symbolTable;
 252:     for (auto funcOp : llvm::reverse(sortedFuncs)) {
 253:       initialize(funcOp, loadAnalysis);
 254:       funcOp.walk([&](CallOpInterface callOp) {
 255:         auto callee = dyn_cast<FunctionOpInterface>(
 256:             callOp.resolveCallableInTable(&symbolTable));
 257:         update(callOp, callee);
 258:       });
 259:     }
 260:   }
```
**EN:** This block declares or defines callable APIs such as ModuleAxisInfoAnalysis, push_back, try_emplace, sortedFuncs, begin, end, reverse, and initialize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ModuleAxisInfoAnalysis, push_back, try_emplace, sortedFuncs, begin, end, reverse, and initialize 等可调用 API，用来封装这里提供的核心行为。

### Lines 262-274
```cpp
 262:   AxisInfo *getAxisInfo(Value value) {
 263:     auto funcOp =
 264:         value.getParentRegion()->getParentOfType<FunctionOpInterface>();
 265:     auto *axisInfoMap = getFuncData(funcOp);
 266:     if (!axisInfoMap) {
 267:       return nullptr;
 268:     }
 269:     auto it = axisInfoMap->find(value);
 270:     if (it == axisInfoMap->end()) {
 271:       return nullptr;
 272:     }
 273:     return &(it->second);
 274:   }
```
**EN:** This block declares or defines callable APIs such as getAxisInfo, getParentRegion, getFuncData, find, and end, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getAxisInfo, getParentRegion, getFuncData, find, and end 等可调用 API，用来封装这里提供的核心行为。

### Lines 276-277
```cpp
 276:   unsigned getContiguity(Value value);
 277:   unsigned getAlignment(Value value);
```
**EN:** This block declares or defines callable APIs such as getContiguity and getAlignment, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getContiguity and getAlignment 等可调用 API，用来封装这里提供的核心行为。

### Lines 279-290
```cpp
 279:   // Overloads of the above methods but have separated elementBitWidth to
 280:   // calculate the contiguity. These are useful for computing axis info when
 281:   // lowering to hardware intrinsics that require a scalar/warp-uniform base ptr
 282:   // with separate per lane offsets like AMD buffer operations.
 283:   //
 284:   // As a concrete example, instead of a single tensor<128x64x!tt.ptr<f16>>
 285:   // value, now we have two separate values: !tt.ptr<f16> for the base pointer
 286:   // and tensor<128x64xi32> for the offset. For such cases, we want to compute
 287:   // the contiguity on the offsets but use the pointee element type bit width
 288:   // instead of the offset element type bit width for alignment
 289:   unsigned getContiguity(Value offsetsValue, unsigned elementBitWidth);
 290:   unsigned getAlignment(Value offsetsValue, unsigned elementBitWidth);
```
**EN:** This block declares or defines callable APIs such as getContiguity and getAlignment, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getContiguity and getAlignment 等可调用 API，用来封装这里提供的核心行为。

### Lines 292-292
```cpp
 292:   unsigned getMaskAlignment(Value mask);
```
**EN:** This block declares or defines callable APIs such as getMaskAlignment, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getMaskAlignment 等可调用 API，用来封装这里提供的核心行为。

### Lines 294-298
```cpp
 294: private:
 295:   void initialize(FunctionOpInterface funcOp, AxisInfoAnalysis::LoadCallback);
 296:   void update(CallOpInterface callOp, FunctionOpInterface funcOp);
 297: };
 298: } // namespace mlir::triton
```
**EN:** This block declares or defines callable APIs such as initialize and update, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 initialize and update 等可调用 API，用来封装这里提供的核心行为。

### Lines 300-300
```cpp
 300: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** axis reasoning  
  **CN:** 轴信息推理
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** interfaces  
  **CN:** 接口
- **EN:** attributes  
  **CN:** 属性
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Analysis/DataFlow/SparseAnalysis.h`
  - `llvm/Support/raw_ostream.h`
  - `mlir/Support/LLVM.h`
  - `triton/Analysis/Utility.h`
- **System or external includes / 系统或外部依赖:**
  - `<optional>`
