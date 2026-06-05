# AxisInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Analysis/AxisInfo.cpp`
- **Purpose / 作用:** **EN:** Implements the axis info analysis logic used by Triton and MLIR passes. **CN:** 实现与 Axis Info 相关的分析逻辑，供 Triton 与 MLIR 的 pass 使用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10

```cpp
   1: #include "triton/Analysis/AxisInfo.h"
   2: #include "mlir/Analysis/DataFlowFramework.h"
   3: #include "mlir/Dialect/UB/IR/UBOps.h"
   4: #include "triton/Dialect/Gluon/IR/Dialect.h"
   5: #include "triton/Dialect/Triton/IR/Dialect.h"
   6: #include "triton/Dialect/Triton/IR/Utility.h"
   7: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   8: #include "llvm/ADT/bit.h"
   9: #include "llvm/Support/Debug.h"
  10: #include "llvm/Support/raw_ostream.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`AxisInfo.h`, `Dialect.h`, `Dialect.h`, `Utility.h`, ... (+1 more)) provide domain-specific IR/support, MLIR headers (`DataFlowFramework.h`, `UBOps.h`) provide rewriting and analysis infrastructure, LLVM headers (`bit.h`, `Debug.h`, `raw_ostream.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`AxisInfo.h`, `Dialect.h`, `Dialect.h`, `Utility.h`, ... (+1 more)）提供领域特定 IR/支持逻辑，MLIR 头文件（`DataFlowFramework.h`, `UBOps.h`）提供重写与分析基础设施，LLVM 头文件（`bit.h`, `Debug.h`, `raw_ostream.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 12-12

```cpp
  12: #include <numeric>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (`numeric`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（`numeric`）提供通用能力。
### Lines 14-16

```cpp
  14: #define DEBUG_TYPE "axis-info"
  15: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  16: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 18-19

```cpp
  18: namespace mlir::triton {
  19: namespace {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton -> (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton -> (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 21-21

```cpp
  21: constexpr int64_t kMaxDivisor = highestPowOf2Divisor<int64_t>(0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 23-32

```cpp
  23: template <typename... Args> int64_t gcd(int64_t a, int64_t b, Args... args) {
  24:   if (a == 0)
  25:     return b;
  26:   if (b == 0)
  27:     return a;
  28:   if constexpr (sizeof...(args) == 0)
  29:     return std::gcd(a, b);
  30:   else
  31:     return gcd(std::gcd(a, b), args...);
  32: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 34-39

```cpp
  34: // If lhs * rhs overflows, return max value possible value for the type
  35: int64_t multiplyDivisor(int64_t lhs, int64_t rhs) {
  36:   if (lhs > kMaxDivisor / rhs)
  37:     return kMaxDivisor;
  38:   return lhs * rhs;
  39: }
```

- **EN:** Defines `multiplyDivisor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `multiplyDivisor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 41-58

```cpp
  41: int64_t getDivisibilityFromContiguity(const AxisInfo &lhs, const AxisInfo &rhs,
  42:                                       int d) {
  43:   // For example if we have the following two arrays using the selectOp:
  44:   // lhs: [[0, 1], [4, 5]]
  45:   // rhs: [[16, 17, 18, 19]]
  46:   // The resulting contiguity will be 2, while the divisibility will be 2
  47:   // because 18 is not divisible by 4.
  48:   if (lhs.getContiguity(d) == rhs.getContiguity(d) ||
  49:       lhs.getContiguity(d) == kMaxDivisor ||
  50:       rhs.getContiguity(d) == kMaxDivisor) {
  51:     // Contiguity not changed or one of them is unresolved.
  52:     // If unresolved, we can first perform a loose bound gcd since the unknown
  53:     // contiguity will be resolved in the end.
  54:     return gcd(lhs.getDivisibility(d), rhs.getDivisibility(d));
  55:   } else {
  56:     // Contiguity changed, we cannot use only divisibility.
  57:     return gcd(lhs.getDivisibility(d), rhs.getDivisibility(d),
  58:                lhs.getContiguity(d), rhs.getContiguity(d));
```

- **EN:** Defines accessor/helper `getDivisibilityFromContiguity` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDivisibilityFromContiguity`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 59-60

```cpp
  59:   }
  60: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 62-65

```cpp
  62: // Base class for all operations
  63: template <typename OpTy> class AxisInfoVisitorImpl : public AxisInfoVisitor {
  64: public:
  65:   using AxisInfoVisitor::AxisInfoVisitor;
```

- **EN:** Defines `AxisInfoVisitorImpl`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AxisInfoVisitorImpl`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 67-71

```cpp
  67:   AxisInfo
  68:   getAxisInfo(Operation *op,
  69:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) final {
  70:     return getAxisInfo(cast<OpTy>(op), operands);
  71:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 73-73

```cpp
  73:   bool match(Operation *op) final { return isa<OpTy>(op); }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 75-78

```cpp
  75:   virtual AxisInfo
  76:   getAxisInfo(OpTy op,
  77:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) = 0;
  78: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 80-84

```cpp
  80: // Binary operations
  81: template <typename OpTy>
  82: class BinaryOpVisitorImpl : public AxisInfoVisitorImpl<OpTy> {
  83: public:
  84:   using AxisInfoVisitorImpl<OpTy>::AxisInfoVisitorImpl;
```

- **EN:** Defines `BinaryOpVisitorImpl`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `BinaryOpVisitorImpl`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 86-103

```cpp
  86:   AxisInfo
  87:   getAxisInfo(OpTy op,
  88:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
  89:     auto lhsInfo = operands[0]->getValue();
  90:     auto rhsInfo = operands[1]->getValue();
  91:     auto rank = lhsInfo.getRank();
  92:     assert(isa<RankedTensorType>(op.getType()) ||
  93:            rank == 1 && "Expected ranked tensor or scalar");
  94:     assert(operands.size() == 2 && "Expected two operands");
  95:     auto constantValue = getConstantValue(op, lhsInfo, rhsInfo);
  96:     if (constantValue.has_value()) {
  97:       auto resTy = dyn_cast<RankedTensorType>(op.getType());
  98:       AxisInfo::DimVectorT constancy =
  99:           resTy ? to_vector(resTy.getShape()) : AxisInfo::DimVectorT(rank, 1);
 100:       AxisInfo::DimVectorT contiguity(rank, 1);
 101:       AxisInfo::DimVectorT divisibility(
 102:           rank, highestPowOf2Divisor<int64_t>(constantValue.value()));
 103:       return AxisInfo(contiguity, divisibility, constancy, constantValue);
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 104-114

```cpp
 104:     }
 105:     AxisInfo::DimVectorT contiguity;
 106:     AxisInfo::DimVectorT divisibility;
 107:     AxisInfo::DimVectorT constancy;
 108:     for (auto d = 0; d < rank; ++d) {
 109:       contiguity.push_back(getContiguity(op, lhsInfo, rhsInfo, d));
 110:       constancy.push_back(getConstancy(op, lhsInfo, rhsInfo, d));
 111:       divisibility.push_back(getDivisibility(op, lhsInfo, rhsInfo, d));
 112:     }
 113:     return AxisInfo(contiguity, divisibility, constancy, constantValue);
 114:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 116-120

```cpp
 116: protected:
 117:   virtual int64_t getContiguity(OpTy op, const AxisInfo &lhs,
 118:                                 const AxisInfo &rhs, int dim) {
 119:     return 1;
 120:   }
```

- **EN:** Defines accessor/helper `getContiguity` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getContiguity`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 122-125

```cpp
 122:   virtual int64_t getDivisibility(OpTy op, const AxisInfo &lhs,
 123:                                   const AxisInfo &rhs, int dim) {
 124:     return 1;
 125:   }
```

- **EN:** Defines accessor/helper `getDivisibility` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDivisibility`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 127-135

```cpp
 127:   virtual int64_t getConstancy(OpTy op, const AxisInfo &lhs,
 128:                                const AxisInfo &rhs, int dim) {
 129:     return gcd(lhs.getConstancy(dim), rhs.getConstancy(dim));
 130:   }
 131:   virtual std::optional<int64_t> getConstantValue(OpTy op, const AxisInfo &lhs,
 132:                                                   const AxisInfo &rhs) {
 133:     return {};
 134:   }
 135: };
```

- **EN:** Defines accessor/helper `getConstancy` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getConstancy`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 137-140

```cpp
 137: template <typename OpTy>
 138: class CastOpAxisInfoVisitor final : public AxisInfoVisitorImpl<OpTy> {
 139: public:
 140:   using AxisInfoVisitorImpl<OpTy>::AxisInfoVisitorImpl;
```

- **EN:** Defines `CastOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CastOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 142-147

```cpp
 142:   AxisInfo
 143:   getAxisInfo(OpTy op,
 144:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 145:     return operands[0]->getValue();
 146:   }
 147: };
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 149-153

```cpp
 149: class UnrealizedConversionCastOpAxisInfoVisitor final
 150:     : public AxisInfoVisitorImpl<mlir::UnrealizedConversionCastOp> {
 151: public:
 152:   using AxisInfoVisitorImpl<
 153:       mlir::UnrealizedConversionCastOp>::AxisInfoVisitorImpl;
```

- **EN:** Defines `UnrealizedConversionCastOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `UnrealizedConversionCastOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 155-167

```cpp
 155:   AxisInfo
 156:   getAxisInfo(mlir::UnrealizedConversionCastOp op,
 157:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 158:     auto tensorType = dyn_cast<RankedTensorType>(op.getResultTypes()[0]);
 159:     if (tensorType &&
 160:         tensorType.getRank() != operands[0]->getValue().getRank()) {
 161:       // Do not propagate AxisInfo with incorrect rank. This can cause a crash
 162:       // in future visitor applications.
 163:       return AxisInfo::getPessimisticValueState(op->getResult(0));
 164:     }
 165:     return operands[0]->getValue();
 166:   }
 167: };
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 169-172

```cpp
 169: class MakeRangeOpAxisInfoVisitor final
 170:     : public AxisInfoVisitorImpl<triton::MakeRangeOp> {
 171: public:
 172:   using AxisInfoVisitorImpl<triton::MakeRangeOp>::AxisInfoVisitorImpl;
```

- **EN:** Defines `MakeRangeOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `MakeRangeOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 174-183

```cpp
 174:   AxisInfo
 175:   getAxisInfo(triton::MakeRangeOp op,
 176:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 177:     auto start = op.getStart();
 178:     auto end = op.getEnd();
 179:     return AxisInfo(/*contiguity=*/{end - start},
 180:                     /*divisibility=*/{highestPowOf2Divisor(start)},
 181:                     /*constancy=*/{1});
 182:   }
 183: };
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 185-188

```cpp
 185: class ConstantOpAxisInfoVisitor final
 186:     : public AxisInfoVisitorImpl<arith::ConstantOp> {
 187: public:
 188:   using AxisInfoVisitorImpl::AxisInfoVisitorImpl;
```

- **EN:** Defines `ConstantOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ConstantOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 190-207

```cpp
 190:   AxisInfo
 191:   getAxisInfo(arith::ConstantOp op,
 192:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 193:     auto intAttr = dyn_cast<IntegerAttr>(op.getValue());
 194:     auto boolAttr = dyn_cast<BoolAttr>(op.getValue());
 195:     if (intAttr || boolAttr) {
 196:       int64_t value{};
 197:       if (intAttr)
 198:         value = intAttr.getValue().getSExtValue();
 199:       else
 200:         value = boolAttr.getValue() ? 1 : 0;
 201:       return AxisInfo(/*contiguity=*/{1},
 202:                       /*divisibility=*/{highestPowOf2Divisor(value)},
 203:                       /*constancy=*/{1},
 204:                       /*knownConstantValue=*/{value});
 205:     }
 206:     // TODO: generalize to dense attr
 207:     auto splatAttr = dyn_cast<SplatElementsAttr>(op.getValue());
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 208-221

```cpp
 208:     if (splatAttr && splatAttr.getElementType().isIntOrIndex()) {
 209:       int64_t value = splatAttr.template getSplatValue<APInt>().getSExtValue();
 210:       TensorType ty = cast<TensorType>(splatAttr.getType());
 211:       return AxisInfo(
 212:           /*contiguity=*/AxisInfo::DimVectorT(ty.getRank(), 1),
 213:           /*divisibility=*/
 214:           AxisInfo::DimVectorT(ty.getRank(), highestPowOf2Divisor(value)),
 215:           /*constancy=*/
 216:           AxisInfo::DimVectorT(ty.getShape().begin(), ty.getShape().end()),
 217:           /*knownConstantValue=*/{value});
 218:     }
 219:     return AxisInfo();
 220:   }
 221: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 223-225

```cpp
 223: class PoisonOpAxisInfoVisitor final : public AxisInfoVisitorImpl<ub::PoisonOp> {
 224: public:
 225:   using AxisInfoVisitorImpl::AxisInfoVisitorImpl;
```

- **EN:** Defines `PoisonOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `PoisonOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 227-232

```cpp
 227:   AxisInfo
 228:   getAxisInfo(ub::PoisonOp op,
 229:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 230:     unsigned rank = 1;
 231:     if (auto shape = dyn_cast<RankedTensorType>(op.getType()))
 232:       rank = shape.getRank();
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 234-239

```cpp
 234:     // Poison values are never accessed, thus assume optimistic values.
 235:     return AxisInfo(AxisInfo::DimVectorT(rank, kMaxDivisor),
 236:                     AxisInfo::DimVectorT(rank, kMaxDivisor),
 237:                     AxisInfo::DimVectorT(rank, kMaxDivisor));
 238:   }
 239: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 241-244

```cpp
 241: template <typename OpTy>
 242: class AddSubOpAxisInfoVisitor final : public BinaryOpVisitorImpl<OpTy> {
 243: public:
 244:   using BinaryOpVisitorImpl<OpTy>::BinaryOpVisitorImpl;
```

- **EN:** Defines `AddSubOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AddSubOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 246-263

```cpp
 246: private:
 247:   int64_t getContiguity(OpTy op, const AxisInfo &lhs, const AxisInfo &rhs,
 248:                         int dim) override {
 249:     if (isa<arith::SubIOp>(op)) {
 250:       // Case 1: If contiguity(lhs) > 1 and contiguity(rhs) > 1,
 251:       // x_t - y_t = (base_x + t) - (base_y + t) = base_x - base_y for any
 252:       // 0 <= t < min(contig_x, contig_y), so contiguity is 1.
 253:       // Case 2: If contiguity(lhs) > 1 and contiguity(rhs) == 1,
 254:       // x_t - y = (base_x + t) - base_y = base_x - base_y + t for any
 255:       // 0 <= t < contig_x,
 256:       // the contiguity depends on the constancy of rhs.
 257:       // Case 3: If contiguity(lhs) == 1 and contiguity(rhs) > 1,
 258:       // x - y_t = base_x - (base_y + t) = base_x - base_y - t for any
 259:       // 0 <= t < contig_y. The result is decreasing within the contiguous
 260:       // block, so contiguity is 1.
 261:       // Case 4: If contiguity(lhs) == 1 and contiguity(rhs) == 1,
 262:       // x - y = base_x - base_y, so contiguity is 1.
 263:       return gcd(lhs.getContiguity(dim), rhs.getConstancy(dim));
```

- **EN:** Defines accessor/helper `getContiguity` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getContiguity`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 264-279

```cpp
 264:     }
 265:     // For AddIOp and AddPtrOp
 266:     // Case 1: If contiguity(lhs) > 1 and contiguity(rhs) > 1,
 267:     // x_t + y_t = (base_x + t) + (base_y + t) = base_x + base_y + 2t for any
 268:     // 0 <= t < min(contig_x, contig_y),
 269:     // so contiguity is 1.
 270:     // Case 2: If contiguity(lhs) > 1 and contiguity(rhs) == 1,
 271:     // x_t + y = (base_x + t) + base_y = base_x + base_y + t for any
 272:     // 0 <= t < contig_x, so contiguity depends on constancy of rhs.
 273:     // Case 3: If contiguity(lhs) == 1 and contiguity(rhs) > 1,
 274:     // It's symmetric to case B.
 275:     // Case 4: If contiguity(lhs) == 1 and contiguity(rhs) == 1,
 276:     // It's trivial that contiguity is 1
 277:     return std::max(gcd(lhs.getConstancy(dim), rhs.getContiguity(dim)),
 278:                     gcd(lhs.getContiguity(dim), rhs.getConstancy(dim)));
 279:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 281-298

```cpp
 281:   int64_t getDivisibility(OpTy op, const AxisInfo &lhs, const AxisInfo &rhs,
 282:                           int dim) override {
 283:     int64_t elemSize = 1;
 284:     auto lhsDivisibility = lhs.getDivisibility(dim);
 285:     auto rhsDivisibility = rhs.getDivisibility(dim);
 286:     if constexpr (std::is_same_v<OpTy, triton::AddPtrOp>) {
 287:       //  %ptr = addptr %lhs, %rhs
 288:       // is equivalent to
 289:       //  %0 = mul %rhs, %elemSize
 290:       //  %ptr = add %lhs, %0
 291:       // The result will still be contiguous in terms of elements but not bytes
 292:       // For example:
 293:       // addptr [16] : !ptr<i32>, [0, 1, 2, 3] : i32 -> !ptr<i32>
 294:       // returns:
 295:       // [16, 20, 24, 28] : !ptr<i32>
 296:       // with element locations:
 297:       // [4, 5, 6, 7]
 298:       // It is "strided contiguous" with a divisibility of 16 bytes
```

- **EN:** Defines accessor/helper `getDivisibility` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDivisibility`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 299-316

```cpp
 299:       elemSize = std::max<int64_t>(
 300:           1, triton::getPointeeBitWidth(op.getPtr().getType()) / 8);
 301:       rhsDivisibility = multiplyDivisor(rhs.getDivisibility(dim), elemSize);
 302:     }
 303:     if (lhs.getContiguity(dim) > 1 && rhs.getContiguity(dim) > 1) {
 304:       // If both operands are contiguous, the in-group offsets are:
 305:       // Let lhs_t = base_lhs + t and rhs_t = base_rhs + t for any
 306:       // 0 <= t < min(contig_lhs, contig_rhs).
 307:       // For addition:
 308:       //   lhs_t + rhs_t = base_lhs + base_rhs + 2t
 309:       // For subtraction:
 310:       //   lhs_t - rhs_t = base_lhs - base_rhs
 311:       if constexpr (std::is_same_v<OpTy, arith::SubIOp>) {
 312:         if (lhs.getContiguity(dim) == rhs.getContiguity(dim))
 313:           return gcd(lhsDivisibility, rhsDivisibility);
 314:       }
 315:       if ((lhsDivisibility % 2 == 0 && rhsDivisibility % 2 == 0)) {
 316:         // Both even -> result divisible by 2.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 317-334

```cpp
 317:         return 2;
 318:       } else {
 319:         // At least one is odd -> the "lower bound" of divisibility is 1.
 320:         return 1;
 321:       }
 322:     } else {
 323:       // At least one operand is partially constant.
 324:       // Divisibility is defined on the *first element* of a contiguity
 325:       // group. When an operand has contiguity larger than the result
 326:       // contiguity, the "first element of a result group" can fall inside an
 327:       // operand's contiguity group, so we must clamp the operand divisibility
 328:       // accordingly (otherwise we can overestimate alignment).
 329:       if (lhs.getContiguity(dim) > 1 || rhs.getContiguity(dim) > 1) {
 330:         auto resContiguity = getContiguity(op, lhs, rhs, dim);
 331:         return gcd(lhsDivisibility, rhsDivisibility,
 332:                    multiplyDivisor(resContiguity, elemSize));
 333:       } else {
 334:         return gcd(lhsDivisibility, rhsDivisibility);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 335-337

```cpp
 335:       }
 336:     }
 337:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 339-356

```cpp
 339:   std::optional<int64_t> getConstantValue(OpTy op, const AxisInfo &lhs,
 340:                                           const AxisInfo &rhs) override {
 341:     if (lhs.getConstantValue().has_value() &&
 342:         rhs.getConstantValue().has_value()) {
 343:       if constexpr (std::is_same_v<OpTy, arith::AddIOp>) {
 344:         return {lhs.getConstantValue().value() +
 345:                 rhs.getConstantValue().value()};
 346:       } else if constexpr (std::is_same_v<OpTy, arith::SubIOp>) {
 347:         return {lhs.getConstantValue().value() -
 348:                 rhs.getConstantValue().value()};
 349:       } else if constexpr (std::is_same_v<OpTy, triton::AddPtrOp>) {
 350:         auto elemSize = std::max<int64_t>(
 351:             1, triton::getPointeeBitWidth(op.getPtr().getType()) / 8);
 352:         auto rhsValue = rhs.getConstantValue().value() * elemSize;
 353:         return {lhs.getConstantValue().value() + rhsValue};
 354:       }
 355:     }
 356:     return {};
```

- **EN:** Defines accessor/helper `getConstantValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getConstantValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 357-358

```cpp
 357:   }
 358: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 360-362

```cpp
 360: class MulIOpAxisInfoVisitor final : public BinaryOpVisitorImpl<arith::MulIOp> {
 361: public:
 362:   using BinaryOpVisitorImpl<arith::MulIOp>::BinaryOpVisitorImpl;
```

- **EN:** Defines `MulIOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `MulIOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 364-378

```cpp
 364: private:
 365:   int64_t getContiguity(arith::MulIOp op, const AxisInfo &lhs,
 366:                         const AxisInfo &rhs, int dim) override {
 367:     // lhs * 1 = lhs
 368:     auto lhsContiguity =
 369:         rhs.getConstantValue().has_value() && rhs.getConstantValue() == 1
 370:             ? lhs.getContiguity(dim)
 371:             : 1;
 372:     // 1 * rhs = rhs
 373:     auto rhsContiguity =
 374:         lhs.getConstantValue().has_value() && lhs.getConstantValue() == 1
 375:             ? rhs.getContiguity(dim)
 376:             : 1;
 377:     return std::max(lhsContiguity, rhsContiguity);
 378:   }
```

- **EN:** Defines accessor/helper `getContiguity` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getContiguity`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 380-397

```cpp
 380:   int64_t getDivisibility(arith::MulIOp op, const AxisInfo &lhs,
 381:                           const AxisInfo &rhs, int dim) override {
 382:     auto lhsDivisibility = lhs.getDivisibility(dim);
 383:     if (lhs.getContiguity(dim) > 1 && rhs.getConstantValue() != 1) {
 384:       // If the operand is contiguous, the divisibility of the
 385:       // sequence drops to 1.
 386:       // Example: [4, 5, 6, 7] (base 4 divisible by 4).
 387:       // Multiplying by 2 yields [8, 10, 12, 14] (GCD=2).
 388:       // Preserving divisibility=4 implies result align 8 (unsafe).
 389:       lhsDivisibility = 1;
 390:     }
 391:     auto rhsDivisibility = rhs.getDivisibility(dim);
 392:     if (rhs.getContiguity(dim) > 1 && lhs.getConstantValue() != 1) {
 393:       // Treat [2^n,2^n+1,...]'s divisibility as 1 instead of 2^n
 394:       rhsDivisibility = 1;
 395:     }
 396:     return multiplyDivisor(lhsDivisibility, rhsDivisibility);
 397:   }
```

- **EN:** Defines accessor/helper `getDivisibility` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDivisibility`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 399-410

```cpp
 399:   std::optional<int64_t> getConstantValue(arith::MulIOp op, const AxisInfo &lhs,
 400:                                           const AxisInfo &rhs) override {
 401:     auto lhsConst = lhs.getConstantValue();
 402:     auto rhsConst = rhs.getConstantValue();
 403:     if (lhsConst.has_value() && rhsConst.has_value())
 404:       return {lhsConst.value() * rhsConst.value()};
 405:     if ((lhsConst.has_value() && lhsConst.value() == 0) ||
 406:         (rhsConst.has_value() && rhsConst.value() == 0))
 407:       return 0;
 408:     return {};
 409:   }
 410: };
```

- **EN:** Defines accessor/helper `getConstantValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getConstantValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 412-415

```cpp
 412: template <typename OpTy>
 413: class DivOpAxisInfoVisitor final : public BinaryOpVisitorImpl<OpTy> {
 414: public:
 415:   using BinaryOpVisitorImpl<OpTy>::BinaryOpVisitorImpl;
```

- **EN:** Defines `DivOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `DivOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 417-425

```cpp
 417: private:
 418:   int64_t getContiguity(OpTy op, const AxisInfo &lhs, const AxisInfo &rhs,
 419:                         int dim) override {
 420:     // lhs / 1 = lhs
 421:     return rhs.getConstantValue().has_value() &&
 422:                    rhs.getConstantValue().value() == 1
 423:                ? lhs.getContiguity(dim)
 424:                : 1;
 425:   }
```

- **EN:** Defines accessor/helper `getContiguity` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getContiguity`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 427-442

```cpp
 427:   int64_t getConstancy(OpTy op, const AxisInfo &lhs, const AxisInfo &rhs,
 428:                        int dim) override {
 429:     auto resTy = dyn_cast<RankedTensorType>(op.getType());
 430:     auto constancy = BinaryOpVisitorImpl<OpTy>::getConstancy(op, lhs, rhs, dim);
 431:     if (!resTy)
 432:       return constancy;
 433:     auto shape = resTy.getShape();
 434:     // Case: lhs contiguous, rhs constant.
 435:     // lhs: d_lhs * k, d_lhs * k + 1, ..., d_lhs * k + n
 436:     // rhs: d_rhs * p, d_rhs * p, ..., d_rhs * p
 437:     // lhs / rhs = d_lhs * k / (d_rhs * p), (d_lhs * k + 1) / (d_rhs * p),
 438:     // ..., (d_lhs * k + n) / (d_rhs * p)
 439:     // Because d_lhs % d_rhs = 0 || d_rhs % d_lhs = 0,
 440:     // the minimal constancy is gcd(d_lhs, d_rhs).
 441:     // Since gcd(d_lhs, d_rhs) maybe > len(lhs),
 442:     // we need to use another gcd to get the actual constancy.
```

- **EN:** Defines accessor/helper `getConstancy` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getConstancy`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 443-450

```cpp
 443:     if (AxisInfoVisitor::isContiguousDim(lhs, shape, dim) &&
 444:         AxisInfoVisitor::isConstantDim(rhs, shape, dim)) {
 445:       constancy = std::max(constancy,
 446:                            gcd(lhs.getContiguity(dim), lhs.getDivisibility(dim),
 447:                                rhs.getDivisibility(dim)));
 448:     }
 449:     return constancy;
 450:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 452-469

```cpp
 452:   int64_t getDivisibility(OpTy op, const AxisInfo &lhs, const AxisInfo &rhs,
 453:                           int dim) override {
 454:     // Case 1: lhs is 0
 455:     if (lhs.getConstantValue().has_value() &&
 456:         lhs.getConstantValue().value() == 0)
 457:       return lhs.getDivisibility(dim);
 458:     // Case 2: rhs is 1
 459:     if (rhs.getConstantValue().has_value() &&
 460:         rhs.getConstantValue().value() == 1)
 461:       return lhs.getDivisibility(dim);
 462:     // Case 3: lhs has contiguity of 1 in this dimension and rhs is a power of 2
 463:     if (rhs.getConstantValue().has_value() &&
 464:         llvm::isPowerOf2_64(std::abs(rhs.getConstantValue().value())) &&
 465:         lhs.getContiguity(dim) == 1) {
 466:       int64_t absRhs = std::abs(rhs.getConstantValue().value());
 467:       return std::max<int64_t>(1, lhs.getDivisibility(dim) / absRhs);
 468:     }
 469:     // otherwise: return 1
```

- **EN:** Defines accessor/helper `getDivisibility` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDivisibility`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 470-471

```cpp
 470:     return 1;
 471:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 473-480

```cpp
 473:   std::optional<int64_t> getConstantValue(OpTy op, const AxisInfo &lhs,
 474:                                           const AxisInfo &rhs) override {
 475:     if (lhs.getConstantValue().has_value() &&
 476:         rhs.getConstantValue().has_value())
 477:       return {lhs.getConstantValue().value() / rhs.getConstantValue().value()};
 478:     return {};
 479:   }
 480: };
```

- **EN:** Defines accessor/helper `getConstantValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getConstantValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 482-485

```cpp
 482: template <typename OpTy>
 483: class RemOpAxisInfoVisitor final : public BinaryOpVisitorImpl<OpTy> {
 484: public:
 485:   using BinaryOpVisitorImpl<OpTy>::BinaryOpVisitorImpl;
```

- **EN:** Defines `RemOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `RemOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 487-503

```cpp
 487: private:
 488:   int64_t getContiguity(OpTy op, const AxisInfo &lhs, const AxisInfo &rhs,
 489:                         int dim) override {
 490:     auto resTy = dyn_cast<RankedTensorType>(op.getType());
 491:     if (!resTy)
 492:       return BinaryOpVisitorImpl<OpTy>::getContiguity(op, lhs, rhs, dim);
 493:     auto shape = resTy.getShape();
 494:     int64_t contiguity = 1;
 495:     // lhs contiguous, rhs constant
 496:     // lhs: d_lhs * k, d_lhs * k + 1, ..., d_lhs * k + n
 497:     // rhs: d_rhs * p, d_rhs * p, ..., d_rhs * p
 498:     // lhs % rhs = d_lhs * k % (d_rhs * p), (d_lhs * k + 1) % (d_rhs * p),
 499:     // ..., (d_lhs * k + n) % (d_rhs * p)
 500:     // Because d_lhs % d_rhs = 0 || d_rhs % d_lhs = 0,
 501:     // The minimal contiguity is gcd(d_lhs, d_rhs).
 502:     // Since gcd(d_lhs, d_rhs) maybe > len(lhs),
 503:     // we need to use another gcd to get the actual contiguity.
```

- **EN:** Defines accessor/helper `getContiguity` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getContiguity`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 504-510

```cpp
 504:     if (AxisInfoVisitor::isContiguousDim(lhs, shape, dim) &&
 505:         AxisInfoVisitor::isConstantDim(rhs, shape, dim)) {
 506:       contiguity = gcd(lhs.getContiguity(dim), lhs.getDivisibility(dim),
 507:                        rhs.getDivisibility(dim));
 508:     }
 509:     return contiguity;
 510:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 512-526

```cpp
 512:   int64_t getDivisibility(OpTy op, const AxisInfo &lhs, const AxisInfo &rhs,
 513:                           int dim) override {
 514:     if (rhs.getConstancy(dim) > 1) {
 515:       // lhs: d_lhs * k = gcd(d_lhs, d_rhs) * k' * k = gcd(d_lhs, d_rhs) * k''
 516:       // rhs: d_rhs * p = gcd(d_lhs, d_rhs) * p' * p = gcd(d_lhs, d_rhs) * p''
 517:       // lhs = gcd(d_lhs, d_rhs) * k'' = gcd(d_lhs, d_rhs) * d + r
 518:       // r must be divisible by gcd(d_lhs, d_rhs)
 519:       return gcd(lhs.getDivisibility(dim), rhs.getDivisibility(dim));
 520:     }
 521:     // Otherwise we shouldn't assume any divisibility.
 522:     // For example:
 523:     // lhs: [2, 2, 4, 4], rhs: [0, 1, 2, 3]
 524:     // lhs % rhs = [0, 0, 0, 1]
 525:     return 1;
 526:   };
```

- **EN:** Defines accessor/helper `getDivisibility` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDivisibility`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 528-539

```cpp
 528:   int64_t getConstancy(OpTy op, const AxisInfo &lhs, const AxisInfo &rhs,
 529:                        int dim) override {
 530:     auto constancy = BinaryOpVisitorImpl<OpTy>::getConstancy(op, lhs, rhs, dim);
 531:     auto resTy = dyn_cast<RankedTensorType>(op.getType());
 532:     if (!resTy)
 533:       return constancy;
 534:     // Case: lhs % 1 = 0
 535:     if (rhs.getConstantValue().has_value() &&
 536:         rhs.getConstantValue().value() == 1)
 537:       return resTy.getDimSize(dim);
 538:     return constancy;
 539:   }
```

- **EN:** Defines accessor/helper `getConstancy` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getConstancy`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 541-551

```cpp
 541:   std::optional<int64_t> getConstantValue(OpTy op, const AxisInfo &lhs,
 542:                                           const AxisInfo &rhs) override {
 543:     if (lhs.getConstantValue().has_value() &&
 544:         rhs.getConstantValue().has_value())
 545:       return {lhs.getConstantValue().value() % rhs.getConstantValue().value()};
 546:     else if (rhs.getConstantValue().has_value() &&
 547:              rhs.getConstantValue().value() == 1)
 548:       return {0};
 549:     return {};
 550:   }
 551: };
```

- **EN:** Defines accessor/helper `getConstantValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getConstantValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 553-556

```cpp
 553: class SplatOpAxisInfoVisitor final
 554:     : public AxisInfoVisitorImpl<triton::SplatOp> {
 555: public:
 556:   using AxisInfoVisitorImpl<triton::SplatOp>::AxisInfoVisitorImpl;
```

- **EN:** Defines `SplatOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `SplatOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 558-575

```cpp
 558:   AxisInfo
 559:   getAxisInfo(triton::SplatOp op,
 560:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 561:     Type _retTy = *op->result_type_begin();
 562:     TensorType retTy = cast<TensorType>(_retTy);
 563:     AxisInfo opInfo = operands[0]->getValue();
 564:     AxisInfo::DimVectorT contiguity;
 565:     AxisInfo::DimVectorT divisibility;
 566:     AxisInfo::DimVectorT constancy;
 567:     for (int d = 0; d < retTy.getRank(); ++d) {
 568:       contiguity.push_back(1);
 569:       divisibility.push_back(opInfo.getDivisibility(0));
 570:       constancy.push_back(retTy.getShape()[d]);
 571:     }
 572:     return AxisInfo(contiguity, divisibility, constancy,
 573:                     operands[0]->getValue().getConstantValue());
 574:   }
 575: };
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 577-579

```cpp
 577: class LoadOpAxisInfoVisitor final : public AxisInfoVisitorImpl<triton::LoadOp> {
 578: public:
 579:   using AxisInfoVisitorImpl<triton::LoadOp>::AxisInfoVisitorImpl;
```

- **EN:** Defines `LoadOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LoadOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 581-593

```cpp
 581:   AxisInfo
 582:   getAxisInfo(triton::LoadOp op,
 583:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 584:     // If pointers and mask both have constancy properties, those properties
 585:     // will also extend to output.
 586:     AxisInfo ptrInfo = operands[0]->getValue();
 587:     std::optional<AxisInfo> maskInfo;
 588:     if (operands.size() > 1) {
 589:       maskInfo = operands[1]->getValue();
 590:     }
 591:     AxisInfo::DimVectorT contiguity;
 592:     AxisInfo::DimVectorT divisibility;
 593:     AxisInfo::DimVectorT constancy;
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 595-601

```cpp
 595:     for (int d = 0; d < ptrInfo.getRank(); ++d) {
 596:       contiguity.push_back(1);
 597:       divisibility.push_back(1);
 598:       constancy.push_back(
 599:           gcd(ptrInfo.getConstancy(d),
 600:               maskInfo.has_value() ? maskInfo->getConstancy(d) : 0));
 601:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 603-605

```cpp
 603:     return AxisInfo(contiguity, divisibility, constancy);
 604:   }
 605: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 607-610

```cpp
 607: class ExpandDimsOpAxisInfoVisitor final
 608:     : public AxisInfoVisitorImpl<triton::ExpandDimsOp> {
 609: public:
 610:   using AxisInfoVisitorImpl<triton::ExpandDimsOp>::AxisInfoVisitorImpl;
```

- **EN:** Defines `ExpandDimsOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ExpandDimsOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 612-627

```cpp
 612:   AxisInfo
 613:   getAxisInfo(triton::ExpandDimsOp op,
 614:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 615:     AxisInfo opInfo = operands[0]->getValue();
 616:     AxisInfo::DimVectorT contiguity = opInfo.getContiguity();
 617:     AxisInfo::DimVectorT divisibility = opInfo.getDivisibility();
 618:     AxisInfo::DimVectorT constancy = opInfo.getConstancy();
 619:     int64_t newDivisibility = 1;
 620:     if (opInfo.getConstantValue().has_value()) {
 621:       // The tensor is constant, same as ConstantOpAxisInfoVisitor
 622:       newDivisibility = highestPowOf2Divisor(opInfo.getConstantValue().value());
 623:     } else if (opInfo.getRank()) {
 624:       // Otherwise, calculate the GCD as the new divisibility
 625:       // Treat [2^n,2^n+1,...]'s divisibility as 1 instead of 2^n
 626:       newDivisibility =
 627:           opInfo.getContiguity(0) > 1 ? 1 : opInfo.getDivisibility(0);
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 628-640

```cpp
 628:       for (int d = 1; d < opInfo.getRank(); ++d) {
 629:         newDivisibility =
 630:             gcd(newDivisibility,
 631:                 opInfo.getContiguity(d) > 1 ? 1 : opInfo.getDivisibility(d));
 632:       }
 633:     }
 634:     contiguity.insert(contiguity.begin() + op.getAxis(), 1);
 635:     divisibility.insert(divisibility.begin() + op.getAxis(), newDivisibility);
 636:     constancy.insert(constancy.begin() + op.getAxis(), 1);
 637:     return AxisInfo(contiguity, divisibility, constancy,
 638:                     operands[0]->getValue().getConstantValue());
 639:   }
 640: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 642-645

```cpp
 642: class BroadcastOpAxisInfoVisitor final
 643:     : public AxisInfoVisitorImpl<triton::BroadcastOp> {
 644: public:
 645:   using AxisInfoVisitorImpl<triton::BroadcastOp>::AxisInfoVisitorImpl;
```

- **EN:** Defines `BroadcastOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `BroadcastOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 647-664

```cpp
 647:   AxisInfo
 648:   getAxisInfo(triton::BroadcastOp op,
 649:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 650:     Type _retTy = *op->result_type_begin();
 651:     Type _opTy = *op->operand_type_begin();
 652:     TensorType retTy = cast<TensorType>(_retTy);
 653:     TensorType opTy = cast<TensorType>(_opTy);
 654:     ArrayRef<int64_t> retShape = retTy.getShape();
 655:     ArrayRef<int64_t> opShape = opTy.getShape();
 656:     AxisInfo opInfo = operands[0]->getValue();
 657:     AxisInfo::DimVectorT contiguity;
 658:     AxisInfo::DimVectorT divisibility;
 659:     AxisInfo::DimVectorT constancy;
 660:     for (int d = 0; d < retTy.getRank(); ++d) {
 661:       contiguity.push_back(opShape[d] == 1 ? 1 : opInfo.getContiguity(d));
 662:       divisibility.push_back(opInfo.getDivisibility(d));
 663:       constancy.push_back(opShape[d] == 1 ? retShape[d]
 664:                                           : opInfo.getConstancy(d));
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 665-669

```cpp
 665:     }
 666:     return AxisInfo(contiguity, divisibility, constancy,
 667:                     operands[0]->getValue().getConstantValue());
 668:   }
 669: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 671-674

```cpp
 671: class ReshapeOpAxisInfoVisitor final
 672:     : public AxisInfoVisitorImpl<triton::ReshapeOp> {
 673: public:
 674:   using AxisInfoVisitorImpl<triton::ReshapeOp>::AxisInfoVisitorImpl;
```

- **EN:** Defines `ReshapeOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ReshapeOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 676-682

```cpp
 676:   AxisInfo
 677:   getAxisInfo(triton::ReshapeOp op,
 678:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 679:     AxisInfo srcInfo = operands[0]->getValue();
 680:     auto srcTy = cast<RankedTensorType>(op.getSrc().getType());
 681:     auto dstTy = cast<RankedTensorType>(op.getType());
 682:     auto dstShape = dstTy.getShape();
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 684-693

```cpp
 684:     // Constant tensor stays constant
 685:     if (srcInfo.getConstantValue().has_value()) {
 686:       AxisInfo::DimVectorT contiguity(dstTy.getRank(), 1);
 687:       AxisInfo::DimVectorT divisibility(
 688:           dstTy.getRank(),
 689:           highestPowOf2Divisor(srcInfo.getConstantValue().value()));
 690:       AxisInfo::DimVectorT constancy(dstShape.begin(), dstShape.end());
 691:       return AxisInfo(contiguity, divisibility, constancy,
 692:                       srcInfo.getConstantValue());
 693:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 695-705

```cpp
 695:     auto srcShape = srcTy.getShape();
 696:     // `suffixProducts[d + 1]` is the flat stride of axis `d` in row-major
 697:     // order.
 698:     auto getSuffixProducts = [](ArrayRef<int64_t> shape) {
 699:       SmallVector<int64_t> suffixProducts(shape.size() + 1, 1);
 700:       for (int d = shape.size() - 1; d >= 0; --d)
 701:         suffixProducts[d] = suffixProducts[d + 1] * shape[d];
 702:       return suffixProducts;
 703:     };
 704:     auto srcSuffixProducts = getSuffixProducts(srcShape);
 705:     auto dstSuffixProducts = getSuffixProducts(dstShape);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 707-709

```cpp
 707:     AxisInfo::DimVectorT contiguity(dstTy.getRank(), 1);
 708:     AxisInfo::DimVectorT divisibility(dstTy.getRank(), 1);
 709:     AxisInfo::DimVectorT constancy(dstTy.getRank(), 1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 711-725

```cpp
 711:     for (int dstDim = 0; dstDim < dstTy.getRank(); ++dstDim) {
 712:       int64_t dstStride = dstSuffixProducts[dstDim + 1];
 713:       // Main idea:
 714:       // Let m = dstDim, and Q = dstSuffixProducts, P = srcSuffixProducts
 715:       // Q[m + 1] \in [P[i + 1], P[i]).
 716:       // This means that dimension m is splitting dimension i, so it often
 717:       // inherits the properties of this dimension
 718:       // Note that the "off by one" indexing comes from the fact that the
 719:       // stride for dimension m is in Q[m + 1].
 720:       int srcDim = 0;
 721:       for (; srcDim < srcTy.getRank(); ++srcDim) {
 722:         int64_t srcStride = srcSuffixProducts[srcDim + 1];
 723:         if (srcStride <= dstStride && dstStride < srcSuffixProducts[srcDim])
 724:           break;
 725:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 727-733

```cpp
 727:       if (srcDim == srcTy.getRank()) {
 728:         // If there are 1-sized axes at the beginning, we do not have
 729:         // dstStride < srcSuffixProducts[srcDim] but we can still reuse
 730:         // the outermost source axis.
 731:         assert(dstShape[dstDim] == 1);
 732:         srcDim = 0;
 733:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 735-738

```cpp
 735:       int64_t srcStride = srcSuffixProducts[srcDim + 1];
 736:       int64_t srcContiguity = srcInfo.getContiguity(srcDim);
 737:       int64_t srcDivisibility = srcInfo.getDivisibility(srcDim);
 738:       int64_t srcConstancy = srcInfo.getConstancy(srcDim);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 740-755

```cpp
 740:       if (srcContiguity > 1) {
 741:         // Contiguity only survives when reshape lands on the low boundary of
 742:         // the source axis. Starting inside the axis loses the unit-stride run.
 743:         if (dstStride == srcStride) {
 744:           int64_t dstContiguity = std::min(srcContiguity, dstShape[dstDim]);
 745:           contiguity[dstDim] = dstContiguity;
 746:           // If the whole contiguous run survives, the group bases are
 747:           // unchanged. When the run is truncated, later group bases can start
 748:           // inside the original run, so divisibility must be clamped
 749:           // accordingly.
 750:           divisibility[dstDim] = dstContiguity == srcContiguity
 751:                                      ? srcDivisibility
 752:                                      : std::min(srcDivisibility, dstContiguity);
 753:         }
 754:         continue;
 755:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 757-762

```cpp
 757:       int64_t constancyEnd = srcStride * srcConstancy;
 758:       if (dstStride <= constancyEnd) {
 759:         // If we land inside a constant axis, the constancy is the minimum
 760:         // between the shape and how much constancy survives.
 761:         int64_t dstConstancy =
 762:             std::min<int64_t>(dstShape[dstDim], constancyEnd / dstStride);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 764-781

```cpp
 764:         // Several constant dimensions can merge into a single constant
 765:         // dimension
 766:         int64_t remainingSize = dstShape[dstDim] / dstConstancy;
 767:         for (int dim = srcDim - 1;
 768:              srcConstancy == srcShape[srcDim] && dim >= 0 && remainingSize > 1;
 769:              --dim) {
 770:           int64_t pieceSize = std::min(srcShape[dim], remainingSize);
 771:           int64_t pieceConstancy =
 772:               std::min(srcInfo.getConstancy(dim), pieceSize);
 773:           dstConstancy *= pieceConstancy;
 774:           if (pieceConstancy < pieceSize)
 775:             break;
 776:           remainingSize /= pieceSize;
 777:         }
 778:         constancy[dstDim] = dstConstancy;
 779:       }
 780:       // Divisibility stays the same when the constant block is split
 781:       // even for constancy == 1.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 782-783

```cpp
 782:       divisibility[dstDim] = srcDivisibility;
 783:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 785-788

```cpp
 785:     return AxisInfo(contiguity, divisibility, constancy,
 786:                     srcInfo.getConstantValue());
 787:   }
 788: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 790-793

```cpp
 790: template <typename OpTy>
 791: class CmpOpAxisInfoVisitor final : public AxisInfoVisitorImpl<OpTy> {
 792: public:
 793:   using AxisInfoVisitorImpl<OpTy>::AxisInfoVisitorImpl;
```

- **EN:** Defines `CmpOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CmpOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 795-804

```cpp
 795:   AxisInfo
 796:   getAxisInfo(OpTy op,
 797:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 798:     auto resTy = dyn_cast<RankedTensorType>(op.getType());
 799:     if (!resTy)
 800:       return AxisInfo();
 801:     auto shape = resTy.getShape();
 802:     short rank = resTy.getRank();
 803:     auto lhsInfo = operands[0]->getValue();
 804:     auto rhsInfo = operands[1]->getValue();
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 806-823

```cpp
 806:     AxisInfo::DimVectorT contiguity, divisibility, constancy;
 807:     std::optional<int64_t> constantValue;
 808:     for (short d = 0; d < rank; ++d) {
 809:       int64_t constHint;
 810:       if (lhsInfo.getConstantValue().has_value() &&
 811:           rhsInfo.getConstantValue().has_value()) {
 812:         constHint = shape[d];
 813:         constantValue =
 814:             compare(getPredicate(op), lhsInfo.getConstantValue().value(),
 815:                     rhsInfo.getConstantValue().value())
 816:                 ? 1
 817:                 : 0;
 818:       } else {
 819:         // Case 1: lhs and rhs are both partial constants
 820:         constHint = gcd(lhsInfo.getConstancy(d), rhsInfo.getConstancy(d));
 821:         if ((gtPredicate(getPredicate(op)) || lePredicate(getPredicate(op))) &&
 822:             AxisInfoVisitor::isConstantDim(lhsInfo, shape, d)) {
 823:           // Case 2: lhs all constant, rhs all contiguous
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 824-841

```cpp
 824:           // NOTE:
 825:           // lhs: 4 4 4 4
 826:           // rhs: 4 5 6 7
 827:           // lhs eq rhs: 1, 0, 0, 0
 828:           // lhs ne rhs: 0, 1, 1, 1
 829:           // lhs lt rhs: 0, 1, 1, 1
 830:           // lhs le rhs: 1, 1, 1, 1
 831:           // lhs ge rhs: 1, 0, 0, 0
 832:           // lhs gt rhs: 0, 0, 0, 0
 833:           constHint = std::max(constHint, gcd(rhsInfo.getContiguity(d),
 834:                                               lhsInfo.getDivisibility(d),
 835:                                               rhsInfo.getDivisibility(d)));
 836:         } else if ((ltPredicate(getPredicate(op)) ||
 837:                     gePredicate(getPredicate(op))) &&
 838:                    AxisInfoVisitor::isConstantDim(rhsInfo, shape, d)) {
 839:           // Case 3: lhs all contiguous, rhs all constant
 840:           // NOTE
 841:           // lhs: 4 5 6 7
```

- **EN:** Defines `gePredicate`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `gePredicate`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 842-853

```cpp
 842:           // rhs: 4 4 4 4
 843:           // lhs eq rhs: 1, 0, 0, 0
 844:           // lhs ne rhs: 0, 1, 1, 1
 845:           // lhs le rhs: 1, 0, 0, 0
 846:           // lhs lt rhs: 0, 0, 0, 0
 847:           // lhs gt rhs: 0, 1, 1, 1
 848:           // lhs ge rhs: 1, 1, 1, 1
 849:           constHint = std::max(constHint, gcd(lhsInfo.getContiguity(d),
 850:                                               lhsInfo.getDivisibility(d),
 851:                                               rhsInfo.getDivisibility(d)));
 852:         }
 853:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 855-858

```cpp
 855:       constancy.push_back(constHint);
 856:       divisibility.push_back(1);
 857:       contiguity.push_back(1);
 858:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 860-861

```cpp
 860:     return AxisInfo(contiguity, divisibility, constancy, constantValue);
 861:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 863-866

```cpp
 863: private:
 864:   static arith::CmpIPredicate getPredicate(arith::CmpIOp op) {
 865:     return op.getPredicate();
 866:   }
```

- **EN:** Defines accessor/helper `getPredicate` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getPredicate`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 868-871

```cpp
 868:   static bool gtPredicate(arith::CmpIPredicate predicate) {
 869:     return predicate == arith::CmpIPredicate::sgt ||
 870:            predicate == arith::CmpIPredicate::ugt;
 871:   }
```

- **EN:** Defines `gtPredicate`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `gtPredicate`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 873-876

```cpp
 873:   static bool gePredicate(arith::CmpIPredicate predicate) {
 874:     return predicate == arith::CmpIPredicate::sge ||
 875:            predicate == arith::CmpIPredicate::uge;
 876:   }
```

- **EN:** Defines `gePredicate`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `gePredicate`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 878-881

```cpp
 878:   static bool ltPredicate(arith::CmpIPredicate predicate) {
 879:     return predicate == arith::CmpIPredicate::slt ||
 880:            predicate == arith::CmpIPredicate::ult;
 881:   }
```

- **EN:** Defines `ltPredicate`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ltPredicate`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 883-886

```cpp
 883:   static bool lePredicate(arith::CmpIPredicate predicate) {
 884:     return predicate == arith::CmpIPredicate::sle ||
 885:            predicate == arith::CmpIPredicate::ule;
 886:   }
```

- **EN:** Defines `lePredicate`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `lePredicate`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 888-905

```cpp
 888:   static bool compare(arith::CmpIPredicate predicate, int64_t lhs,
 889:                       int64_t rhs) {
 890:     switch (predicate) {
 891:     case arith::CmpIPredicate::eq:
 892:       return lhs == rhs;
 893:     case arith::CmpIPredicate::ne:
 894:       return lhs != rhs;
 895:     case arith::CmpIPredicate::slt:
 896:       return lhs < rhs;
 897:     case arith::CmpIPredicate::sle:
 898:       return lhs <= rhs;
 899:     case arith::CmpIPredicate::sgt:
 900:       return lhs > rhs;
 901:     case arith::CmpIPredicate::sge:
 902:       return lhs >= rhs;
 903:     case arith::CmpIPredicate::ult:
 904:       return (uint64_t)lhs < (uint64_t)rhs;
 905:     case arith::CmpIPredicate::ule:
```

- **EN:** Defines `compare`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `compare`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 906-916

```cpp
 906:       return (uint64_t)lhs <= (uint64_t)rhs;
 907:     case arith::CmpIPredicate::ugt:
 908:       return (uint64_t)lhs > (uint64_t)rhs;
 909:     case arith::CmpIPredicate::uge:
 910:       return (uint64_t)lhs >= (uint64_t)rhs;
 911:     default:
 912:       break;
 913:     }
 914:     llvm_unreachable("unknown comparison predicate");
 915:   }
 916: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 918-921

```cpp
 918: template <typename OpTy>
 919: class SelectOpAxisInfoVisitor final : public AxisInfoVisitorImpl<OpTy> {
 920: public:
 921:   using AxisInfoVisitorImpl<OpTy>::AxisInfoVisitorImpl;
```

- **EN:** Defines `SelectOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `SelectOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 923-929

```cpp
 923:   AxisInfo
 924:   getAxisInfo(OpTy op,
 925:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
 926:     auto condConstancy = operands[0]->getValue().getConstancy();
 927:     auto lhsInfo = operands[1]->getValue();
 928:     auto rhsInfo = operands[2]->getValue();
 929:     auto rank = lhsInfo.getRank();
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 931-948

```cpp
 931:     AxisInfo::DimVectorT contiguity, divisibility, constancy;
 932:     std::optional<int64_t> constantValue;
 933:     if (operands[0]->getValue().getConstantValue().has_value()) {
 934:       if (operands[0]->getValue().getConstantValue() == 0) {
 935:         contiguity = rhsInfo.getContiguity();
 936:         divisibility = rhsInfo.getDivisibility();
 937:         constancy = rhsInfo.getConstancy();
 938:         constantValue = rhsInfo.getConstantValue();
 939:       } else {
 940:         contiguity = lhsInfo.getContiguity();
 941:         divisibility = lhsInfo.getDivisibility();
 942:         constancy = lhsInfo.getConstancy();
 943:         constantValue = lhsInfo.getConstantValue();
 944:       }
 945:     } else {
 946:       // The condition can be either a tensor or i1.
 947:       // If i1 is used as the condition, the entire tensor of either
 948:       // lhs or rhs is selected.
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 949-966

```cpp
 949:       bool i1Cond = isa<IntegerType>(op.getOperand(0).getType());
 950:       for (auto d = 0; d < rank; ++d) {
 951:         if (i1Cond) {
 952:           constancy.push_back(
 953:               gcd(lhsInfo.getConstancy(d), rhsInfo.getConstancy(d)));
 954:           divisibility.push_back(
 955:               getDivisibilityFromContiguity(lhsInfo, rhsInfo, d));
 956:           contiguity.push_back(
 957:               gcd(lhsInfo.getContiguity(d), rhsInfo.getContiguity(d)));
 958:         } else {
 959:           constancy.push_back(gcd(lhsInfo.getConstancy(d),
 960:                                   rhsInfo.getConstancy(d), condConstancy[d]));
 961:           contiguity.push_back(gcd(lhsInfo.getContiguity(d),
 962:                                    rhsInfo.getContiguity(d), condConstancy[d]));
 963:           divisibility.push_back(
 964:               getDivisibilityFromContiguity(lhsInfo, rhsInfo, d));
 965:         }
 966:       }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 967-970

```cpp
 967:       if (lhsInfo.getConstantValue().has_value() &&
 968:           rhsInfo.getConstantValue().has_value() &&
 969:           lhsInfo.getConstantValue() == rhsInfo.getConstantValue())
 970:         constantValue = lhsInfo.getConstantValue();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 972-978

```cpp
 972:       if (constantValue.has_value()) {
 973:         auto resTy = dyn_cast<RankedTensorType>(op.getType());
 974:         assert(resTy || rank == 1);
 975:         constancy =
 976:             resTy ? to_vector(resTy.getShape()) : AxisInfo::DimVectorT(rank, 1);
 977:       }
 978:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 980-982

```cpp
 980:     return AxisInfo(contiguity, divisibility, constancy, constantValue);
 981:   }
 982: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 984-987

```cpp
 984: template <typename OpTy>
 985: class LogicalOpAxisInfoVisitor final : public BinaryOpVisitorImpl<OpTy> {
 986: public:
 987:   using BinaryOpVisitorImpl<OpTy>::BinaryOpVisitorImpl;
```

- **EN:** Defines `LogicalOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `LogicalOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 989-1006

```cpp
 989: private:
 990:   std::optional<int64_t> getConstantValue(OpTy op, const AxisInfo &lhs,
 991:                                           const AxisInfo &rhs) override {
 992:     if (lhs.getConstantValue().has_value() &&
 993:         rhs.getConstantValue().has_value()) {
 994:       if constexpr (std::is_same_v<OpTy, arith::AndIOp>) {
 995:         return {lhs.getConstantValue().value() &
 996:                 rhs.getConstantValue().value()};
 997:       } else if constexpr (std::is_same_v<OpTy, arith::OrIOp>) {
 998:         return {lhs.getConstantValue().value() |
 999:                 rhs.getConstantValue().value()};
1000:       } else if constexpr (std::is_same_v<OpTy, arith::XOrIOp>) {
1001:         return {lhs.getConstantValue().value() ^
1002:                 rhs.getConstantValue().value()};
1003:       }
1004:     }
1005:     return {};
1006:   }
```

- **EN:** Defines accessor/helper `getConstantValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getConstantValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1007-1007

```cpp
1007: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1009-1011

```cpp
1009: class ShLIOpAxisInfoVisitor final : public BinaryOpVisitorImpl<arith::ShLIOp> {
1010: public:
1011:   using BinaryOpVisitorImpl<arith::ShLIOp>::BinaryOpVisitorImpl;
```

- **EN:** Defines `ShLIOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ShLIOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1013-1021

```cpp
1013: private:
1014:   int64_t getContiguity(arith::ShLIOp op, const AxisInfo &lhs,
1015:                         const AxisInfo &rhs, int dim) override {
1016:     if (rhs.getConstantValue().has_value() &&
1017:         rhs.getConstantValue().value() == 0)
1018:       return lhs.getContiguity(dim);
1019:     else
1020:       return 1;
1021:   }
```

- **EN:** Defines accessor/helper `getContiguity` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getContiguity`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1023-1032

```cpp
1023:   int64_t getDivisibility(arith::ShLIOp op, const AxisInfo &lhs,
1024:                           const AxisInfo &rhs, int dim) override {
1025:     auto shift = rhs.getConstantValue().value_or(0);
1026:     auto lhsDivisibility = lhs.getDivisibility(dim);
1027:     if (lhs.getContiguity(dim) > 1 && shift) {
1028:       // Treat [2^n,2^n+1,...]'s divisibility as 1 instead of 2^n
1029:       lhsDivisibility = 1;
1030:     }
1031:     return multiplyDivisor(lhsDivisibility, 1ll << shift);
1032:   }
```

- **EN:** Defines accessor/helper `getDivisibility` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDivisibility`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1034-1041

```cpp
1034:   std::optional<int64_t> getConstantValue(arith::ShLIOp op, const AxisInfo &lhs,
1035:                                           const AxisInfo &rhs) override {
1036:     if (lhs.getConstantValue().has_value() &&
1037:         rhs.getConstantValue().has_value())
1038:       return {lhs.getConstantValue().value() << rhs.getConstantValue().value()};
1039:     return {};
1040:   }
1041: };
```

- **EN:** Defines accessor/helper `getConstantValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getConstantValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1043-1046

```cpp
1043: template <typename OpTy>
1044: class ShROpAxisInfoVisitor final : public BinaryOpVisitorImpl<OpTy> {
1045: public:
1046:   using BinaryOpVisitorImpl<OpTy>::BinaryOpVisitorImpl;
```

- **EN:** Defines `ShROpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ShROpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1048-1056

```cpp
1048: private:
1049:   int64_t getContiguity(OpTy op, const AxisInfo &lhs, const AxisInfo &rhs,
1050:                         int dim) override {
1051:     if (rhs.getConstantValue().has_value() &&
1052:         rhs.getConstantValue().value() == 0)
1053:       return lhs.getContiguity(dim);
1054:     else
1055:       return 1;
1056:   }
```

- **EN:** Defines accessor/helper `getContiguity` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getContiguity`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1058-1069

```cpp
1058:   int64_t getDivisibility(OpTy op, const AxisInfo &lhs, const AxisInfo &rhs,
1059:                           int dim) override {
1060:     if (!rhs.getConstantValue().has_value())
1061:       return 1;
1062:     auto shift = rhs.getConstantValue().value();
1063:     auto lhsDivisibility = lhs.getDivisibility(dim);
1064:     if (lhs.getContiguity(dim) > 1 && shift) {
1065:       // Treat [2^n,2^n+1,...]'s divisibility as 1 instead of 2^n
1066:       lhsDivisibility = 1;
1067:     }
1068:     return std::max<int64_t>(1, lhsDivisibility / (int64_t(1) << shift));
1069:   }
```

- **EN:** Defines accessor/helper `getDivisibility` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getDivisibility`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1071-1078

```cpp
1071:   std::optional<int64_t> getConstantValue(OpTy op, const AxisInfo &lhs,
1072:                                           const AxisInfo &rhs) override {
1073:     if (lhs.getConstantValue().has_value() &&
1074:         rhs.getConstantValue().has_value())
1075:       return {lhs.getConstantValue().value() >> rhs.getConstantValue().value()};
1076:     return {};
1077:   }
1078: };
```

- **EN:** Defines accessor/helper `getConstantValue` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getConstantValue`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1080-1083

```cpp
1080: template <typename OpTy>
1081: class MaxMinOpAxisInfoVisitor final : public AxisInfoVisitorImpl<OpTy> {
1082: public:
1083:   using AxisInfoVisitorImpl<OpTy>::AxisInfoVisitorImpl;
```

- **EN:** Defines `MaxMinOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `MaxMinOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1085-1102

```cpp
1085:   AxisInfo
1086:   getAxisInfo(OpTy op,
1087:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
1088:     auto lhsInfo = operands[0]->getValue();
1089:     auto rhsInfo = operands[1]->getValue();
1090:     auto rank = lhsInfo.getRank();
1091:     std::optional<int64_t> constantValue;
1092:     if (lhsInfo.getConstantValue().has_value() &&
1093:         rhsInfo.getConstantValue().has_value()) {
1094:       if constexpr (std::is_same_v<OpTy, arith::MaxSIOp> ||
1095:                     std::is_same_v<OpTy, arith::MaxUIOp>) {
1096:         constantValue = {std::max(lhsInfo.getConstantValue().value(),
1097:                                   rhsInfo.getConstantValue().value())};
1098:       } else if constexpr (std::is_same_v<OpTy, arith::MinSIOp> ||
1099:                            std::is_same_v<OpTy, arith::MinUIOp>) {
1100:         constantValue = {std::min(lhsInfo.getConstantValue().value(),
1101:                                   rhsInfo.getConstantValue().value())};
1102:       }
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1103-1119

```cpp
1103:       auto resTy = dyn_cast<RankedTensorType>(op.getType());
1104:       assert(resTy || rank == 1);
1105:       AxisInfo::DimVectorT constancy =
1106:           resTy ? to_vector(resTy.getShape()) : AxisInfo::DimVectorT(rank, 1);
1107:       AxisInfo::DimVectorT divisibility(
1108:           rank, highestPowOf2Divisor<int64_t>(constantValue.value()));
1109:       return AxisInfo(/*knownContiguity=*/AxisInfo::DimVectorT(rank, 1),
1110:                       /*knownDivisibility=*/divisibility,
1111:                       /*knownConstancy=*/constancy,
1112:                       /*constantValue=*/constantValue);
1113:     } else {
1114:       AxisInfo::DimVectorT contiguity, divisibility, constancy;
1115:       for (auto d = 0; d < rank; ++d) {
1116:         constancy.push_back(
1117:             gcd(lhsInfo.getConstancy(d), rhsInfo.getConstancy(d)));
1118:         divisibility.push_back(
1119:             getDivisibilityFromContiguity(lhsInfo, rhsInfo, d));
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1120-1126

```cpp
1120:         contiguity.push_back(
1121:             gcd(lhsInfo.getContiguity(d), rhsInfo.getContiguity(d)));
1122:       }
1123:       return AxisInfo(contiguity, divisibility, constancy, std::nullopt);
1124:     }
1125:   }
1126: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1128-1131

```cpp
1128: class TransOpAxisInfoVisitor final
1129:     : public AxisInfoVisitorImpl<triton::TransOp> {
1130: public:
1131:   using AxisInfoVisitorImpl<triton::TransOp>::AxisInfoVisitorImpl;
```

- **EN:** Defines `TransOpAxisInfoVisitor`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `TransOpAxisInfoVisitor`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1133-1138

```cpp
1133:   AxisInfo
1134:   getAxisInfo(triton::TransOp op,
1135:               ArrayRef<const dataflow::Lattice<AxisInfo> *> operands) override {
1136:     AxisInfo srcInfo = operands[0]->getValue();
1137:     auto order = op.getOrder();
1138:     auto rank = srcInfo.getRank();
```

- **EN:** Defines accessor/helper `getAxisInfo` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getAxisInfo`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1140-1143

```cpp
1140:     // Apply the transpose permutation to all axis info properties
1141:     AxisInfo::DimVectorT contiguity;
1142:     AxisInfo::DimVectorT divisibility;
1143:     AxisInfo::DimVectorT constancy;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1145-1150

```cpp
1145:     for (int d = 0; d < rank; ++d) {
1146:       int srcDim = order[d];
1147:       contiguity.push_back(srcInfo.getContiguity(srcDim));
1148:       divisibility.push_back(srcInfo.getDivisibility(srcDim));
1149:       constancy.push_back(srcInfo.getConstancy(srcDim));
1150:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1152-1155

```cpp
1152:     return AxisInfo(contiguity, divisibility, constancy,
1153:                     srcInfo.getConstantValue());
1154:   }
1155: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1157-1157

```cpp
1157: } // anonymous namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1159-1161

```cpp
1159: //===----------------------------------------------------------------------===//
1160: // AxisInfoAnalysis
1161: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 1163-1179

```cpp
1163: AxisInfoAnalysis::AxisInfoAnalysis(DataFlowSolver &solver)
1164:     : dataflow::SparseForwardDataFlowAnalysis<dataflow::Lattice<AxisInfo>>(
1165:           solver) {
1166:   // UnrealizedConversionCast:
1167:   // This is needed by TritonGPUToLLVM, to get AxisInfo when the graph is
1168:   // in the process of a PartialConversion, where UnrealizedConversionCast
1169:   // may exist
1170:   visitors.append<UnrealizedConversionCastOpAxisInfoVisitor>();
1171:   visitors.append<CastOpAxisInfoVisitor<arith::ExtSIOp>,
1172:                   CastOpAxisInfoVisitor<arith::ExtUIOp>,
1173:                   CastOpAxisInfoVisitor<arith::TruncIOp>,
1174:                   CastOpAxisInfoVisitor<triton::gpu::ConvertLayoutOp>,
1175:                   CastOpAxisInfoVisitor<triton::BitcastOp>,
1176:                   CastOpAxisInfoVisitor<triton::gluon::SetAutoLayoutOp>>();
1177:   visitors.append<MakeRangeOpAxisInfoVisitor>();
1178:   visitors.append<PoisonOpAxisInfoVisitor>();
1179:   visitors.append<ConstantOpAxisInfoVisitor>();
```

- **EN:** Defines `AxisInfoAnalysis::AxisInfoAnalysis`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AxisInfoAnalysis::AxisInfoAnalysis`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1180-1196

```cpp
1180:   visitors.append<AddSubOpAxisInfoVisitor<triton::AddPtrOp>,
1181:                   AddSubOpAxisInfoVisitor<arith::AddIOp>,
1182:                   AddSubOpAxisInfoVisitor<arith::SubIOp>>();
1183:   visitors.append<MulIOpAxisInfoVisitor>();
1184:   visitors.append<DivOpAxisInfoVisitor<arith::DivSIOp>,
1185:                   DivOpAxisInfoVisitor<arith::DivUIOp>>();
1186:   visitors.append<RemOpAxisInfoVisitor<arith::RemSIOp>,
1187:                   RemOpAxisInfoVisitor<arith::RemUIOp>>();
1188:   visitors.append<BroadcastOpAxisInfoVisitor>();
1189:   visitors.append<SplatOpAxisInfoVisitor>();
1190:   visitors.append<ExpandDimsOpAxisInfoVisitor>();
1191:   visitors.append<ReshapeOpAxisInfoVisitor>();
1192:   visitors.append<CmpOpAxisInfoVisitor<arith::CmpIOp>>();
1193:   visitors.append<LogicalOpAxisInfoVisitor<arith::AndIOp>,
1194:                   LogicalOpAxisInfoVisitor<arith::OrIOp>,
1195:                   LogicalOpAxisInfoVisitor<arith::XOrIOp>>();
1196:   visitors.append<SelectOpAxisInfoVisitor<mlir::arith::SelectOp>>();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1197-1205

```cpp
1197:   visitors.append<ShLIOpAxisInfoVisitor, ShROpAxisInfoVisitor<arith::ShRUIOp>,
1198:                   ShROpAxisInfoVisitor<arith::ShRSIOp>>();
1199:   visitors.append<MaxMinOpAxisInfoVisitor<arith::MaxSIOp>,
1200:                   MaxMinOpAxisInfoVisitor<arith::MaxUIOp>,
1201:                   MaxMinOpAxisInfoVisitor<arith::MinSIOp>,
1202:                   MaxMinOpAxisInfoVisitor<arith::MinUIOp>>();
1203:   visitors.append<LoadOpAxisInfoVisitor>();
1204:   visitors.append<TransOpAxisInfoVisitor>();
1205: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1207-1210

```cpp
1207: void AxisInfoAnalysis::setToEntryState(dataflow::Lattice<AxisInfo> *lattice) {
1208:   propagateIfChanged(lattice, lattice->join(AxisInfo::getPessimisticValueState(
1209:                                   lattice->getAnchor())));
1210: }
```

- **EN:** Defines accessor/helper `AxisInfoAnalysis::setToEntryState` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `AxisInfoAnalysis::setToEntryState`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1212-1221

```cpp
1212: void AxisInfoAnalysis::visitNonControlFlowArguments(
1213:     Operation *op, const RegionSuccessor & /*successor*/,
1214:     ValueRange /*nonSuccessorInputs*/,
1215:     ArrayRef<dataflow::Lattice<AxisInfo> *> argLattices) {
1216:   if (auto forOp = dyn_cast<scf::ForOp>(op)) {
1217:     visitForOpInductionVar(forOp, argLattices);
1218:   } else {
1219:     setAllToEntryStates(argLattices);
1220:   }
1221: }
```

- **EN:** Defines `AxisInfoAnalysis::visitNonControlFlowArguments`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AxisInfoAnalysis::visitNonControlFlowArguments`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1223-1240

```cpp
1223: LogicalResult AxisInfoAnalysis::visitOperation(
1224:     Operation *op, ArrayRef<const dataflow::Lattice<AxisInfo> *> operands,
1225:     ArrayRef<dataflow::Lattice<AxisInfo> *> results) {
1226:   // If any operands are not yet ready, skip this operation for now.
1227:   for (auto op : operands)
1228:     if (op->getValue().getRank() == 0)
1229:       return success();
1230:   AxisInfo curr = visitors.apply(op, operands);
1231:   if (curr.getRank() == 0) {
1232:     setAllToEntryStates(results);
1233:     return success();
1234:   }
1235:   // override with hint
1236:   auto newContiguity = curr.getContiguity();
1237:   auto newDivisibility = curr.getDivisibility();
1238:   auto newConstancy = curr.getConstancy();
1239:   AxisInfo::initDimVectorFromHint(op->getDiscardableAttr("tt.contiguity"),
1240:                                   &newContiguity);
```

- **EN:** Defines `AxisInfoAnalysis::visitOperation`, the transfer function that updates analysis state as each operation is visited.
- **CN:** 这里定义 `AxisInfoAnalysis::visitOperation`，即分析的传递函数：每访问一个操作就更新一次分析状态。
### Lines 1241-1251

```cpp
1241:   AxisInfo::initDimVectorFromHint(op->getDiscardableAttr("tt.divisibility"),
1242:                                   &newDivisibility);
1243:   AxisInfo::initDimVectorFromHint(op->getDiscardableAttr("tt.constancy"),
1244:                                   &newConstancy);
1245:   curr = AxisInfo(newContiguity, newDivisibility, newConstancy,
1246:                   curr.getConstantValue());
1247:   // join all lattice elements
1248:   for (auto *result : results)
1249:     propagateIfChanged(result, result->join(curr));
1250:   return success();
1251: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1253-1262

```cpp
1253: void AxisInfoAnalysis::visitForOpInductionVar(
1254:     scf::ForOp op, ArrayRef<dataflow::Lattice<AxisInfo> *> argLattices) {
1255:   ProgramPoint *programPoint = getProgramPointAfter(op);
1256:   auto *lbLattice = getLatticeElementFor(programPoint, op.getLowerBound());
1257:   auto *stepLattice = getLatticeElementFor(programPoint, op.getStep());
1258:   // If lb or step is not yet ready, skip this operation for now.
1259:   if (lbLattice->getValue().getRank() == 0 ||
1260:       stepLattice->getValue().getRank() == 0) {
1261:     return;
1262:   }
```

- **EN:** Defines `AxisInfoAnalysis::visitForOpInductionVar`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AxisInfoAnalysis::visitForOpInductionVar`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1264-1272

```cpp
1264:   AxisInfo::DimVectorT knownContiguity(1, 1);
1265:   AxisInfo::DimVectorT knownDivisibility(1, 1);
1266:   AxisInfo::DimVectorT knownConstancy(1, 1);
1267:   knownDivisibility[0] = gcd(lbLattice->getValue().getDivisibility(0),
1268:                              stepLattice->getValue().getDivisibility(0));
1269:   auto inductionVar =
1270:       AxisInfo(knownContiguity, knownDivisibility, knownConstancy);
1271:   (void)argLattices[0]->join(inductionVar);
1272: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1274-1289

```cpp
1274: void AxisInfo::initPessimisticStateFromFunc(int argNumber,
1275:                                             FunctionOpInterface funcOp,
1276:                                             DimVectorT *contiguity,
1277:                                             DimVectorT *divisibility,
1278:                                             DimVectorT *constancy) {
1279:   // list of attributes that we care about
1280:   SmallVector<std::pair<DimVectorT *, std::string>> retVecs;
1281:   retVecs.push_back({contiguity, "tt.contiguity"});
1282:   retVecs.push_back({divisibility, "tt.divisibility"});
1283:   retVecs.push_back({constancy, "tt.constancy"});
1284:   // initialize attributes one by one
1285:   for (auto [vec, attrName] : retVecs) {
1286:     Attribute attr = funcOp.getArgAttr(argNumber, attrName);
1287:     AxisInfo::initDimVectorFromHint(attr, vec);
1288:   }
1289: }
```

- **EN:** Defines `AxisInfo::initPessimisticStateFromFunc`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义 `AxisInfo::initPessimisticStateFromFunc`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 1291-1298

```cpp
1291: void AxisInfo::initDimVectorFromHint(Attribute attr, DimVectorT *vec) {
1292:   if (auto int_attr = dyn_cast_or_null<IntegerAttr>(attr))
1293:     *vec = DimVectorT(1, int_attr.getValue().getZExtValue());
1294:   if (auto dense_attr = dyn_cast_or_null<DenseElementsAttr>(attr)) {
1295:     auto vals = dense_attr.getValues<int>();
1296:     *vec = DimVectorT(vals.begin(), vals.end());
1297:   }
1298: }
```

- **EN:** Defines `AxisInfo::initDimVectorFromHint`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AxisInfo::initDimVectorFromHint`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1300-1306

```cpp
1300: /*static*/ AxisInfo AxisInfo::getPessimisticValueState(Value value) {
1301:   auto rank = 1;
1302:   if (TensorType ty = dyn_cast<TensorType>(value.getType()))
1303:     rank = ty.getRank();
1304:   if (triton::PointerType ty = dyn_cast<triton::PointerType>(value.getType()))
1305:     if (TensorType elemTy = dyn_cast<TensorType>(ty.getPointeeType()))
1306:       rank = elemTy.getRank();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1308-1310

```cpp
1308:   DimVectorT knownContiguity(rank, 1);
1309:   DimVectorT knownDivisibility(rank, 1);
1310:   DimVectorT knownConstancy(rank, 1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1312-1312

```cpp
1312:   BlockArgument blockArg = dyn_cast<BlockArgument>(value);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1314-1330

```cpp
1314:   if (blockArg && blockArg.getOwner()->isEntryBlock()) {
1315:     Operation *op = blockArg.getOwner()->getParentOp();
1316:     if (auto fun = dyn_cast<FunctionOpInterface>(op)) {
1317:       initPessimisticStateFromFunc(blockArg.getArgNumber(), fun,
1318:                                    &knownContiguity, &knownDivisibility,
1319:                                    &knownConstancy);
1320:     }
1321:   } else if (Operation *op = value.getDefiningOp()) {
1322:     // Other operations are conservatively initialized with the lowest possible
1323:     // divisibility, contiguity, and constancy unless they have specified.
1324:     AxisInfo::initDimVectorFromHint(op->getDiscardableAttr("tt.divisibility"),
1325:                                     &knownDivisibility);
1326:     AxisInfo::initDimVectorFromHint(op->getDiscardableAttr("tt.contiguity"),
1327:                                     &knownContiguity);
1328:     AxisInfo::initDimVectorFromHint(op->getDiscardableAttr("tt.constancy"),
1329:                                     &knownConstancy);
1330:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1332-1333

```cpp
1332:   return AxisInfo(knownContiguity, knownDivisibility, knownConstancy);
1333: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1335-1350

```cpp
1335: /*static*/ AxisInfo AxisInfo::join(const AxisInfo &lhs, const AxisInfo &rhs) {
1336:   // If one argument is not initialized, return the other.
1337:   if (lhs.getRank() == 0)
1338:     return rhs;
1339:   if (rhs.getRank() == 0)
1340:     return lhs;
1341:   assert(lhs.getRank() == rhs.getRank() && "Mismatched ranks");
1342:   DimVectorT contiguity;
1343:   DimVectorT divisibility;
1344:   DimVectorT constancy;
1345:   for (auto d = 0; d < lhs.getRank(); ++d) {
1346:     contiguity.push_back(gcd(lhs.getContiguity(d), rhs.getContiguity(d)));
1347:     divisibility.push_back(getDivisibilityFromContiguity(lhs, rhs, d));
1348:     constancy.push_back(gcd(lhs.getConstancy(d), rhs.getConstancy(d)));
1349:   }
1350:   std::optional<int64_t> constantValue;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1351-1356

```cpp
1351:   if (lhs.getConstantValue().has_value() &&
1352:       rhs.getConstantValue().has_value() &&
1353:       lhs.getConstantValue() == rhs.getConstantValue())
1354:     constantValue = lhs.getConstantValue();
1355:   return AxisInfo(contiguity, divisibility, constancy, constantValue);
1356: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1358-1361

```cpp
1358: AxisInfoAnalysis *
1359: AxisInfoAnalysis::loadDefaultAnalysis(DataFlowSolver *solver) {
1360:   return solver->load<AxisInfoAnalysis>();
1361: }
```

- **EN:** Defines `AxisInfoAnalysis::loadDefaultAnalysis`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `AxisInfoAnalysis::loadDefaultAnalysis`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1363-1373

```cpp
1363: unsigned ModuleAxisInfoAnalysis::getContiguity(Value value) {
1364:   auto tensorTy = dyn_cast<RankedTensorType>(value.getType());
1365:   if (!tensorTy)
1366:     return 1;
1367:   auto elemTy = tensorTy.getElementType();
1368:   // Get the pointee type if we have a tensor of ptrs to compute contiguity for
1369:   if (auto ptrTy = dyn_cast<PointerType>(elemTy)) {
1370:     elemTy = ptrTy.getPointeeType();
1371:   }
1372:   return getContiguity(value, elemTy.getIntOrFloatBitWidth());
1373: }
```

- **EN:** Defines accessor/helper `ModuleAxisInfoAnalysis::getContiguity` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ModuleAxisInfoAnalysis::getContiguity`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1375-1382

```cpp
1375: unsigned ModuleAxisInfoAnalysis::getContiguity(Value offsetsValue,
1376:                                                unsigned elementBitWidth) {
1377:   // FIXME: This is not as good as it could be, as we don't need to restrict
1378:   // the analysis to one dimension. We should determine contiguity on the
1379:   // flattenOuts() layout
1380:   auto tensorTy = cast<RankedTensorType>(offsetsValue.getType());
1381:   auto order = gpu::getOrder(tensorTy);
1382:   unsigned align = getAlignment(offsetsValue, elementBitWidth);
```

- **EN:** Defines accessor/helper `ModuleAxisInfoAnalysis::getContiguity` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ModuleAxisInfoAnalysis::getContiguity`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1384-1389

```cpp
1384:   auto uniqueContigPerThread = gpu::getContigPerThread(tensorTy);
1385:   assert(order[0] < uniqueContigPerThread.size() &&
1386:          "Unexpected uniqueContigPerThread size");
1387:   unsigned contiguity = uniqueContigPerThread[order[0]];
1388:   LDBG("getContiguity uniqueContigPerThread = " << contiguity);
1389:   contiguity = std::min(align, contiguity);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 1391-1392

```cpp
1391:   return contiguity;
1392: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1394-1397

```cpp
1394: unsigned ModuleAxisInfoAnalysis::getAlignment(Value value) {
1395:   auto tensorTy = dyn_cast<RankedTensorType>(value.getType());
1396:   if (!tensorTy)
1397:     return 1;
```

- **EN:** Defines accessor/helper `ModuleAxisInfoAnalysis::getAlignment` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ModuleAxisInfoAnalysis::getAlignment`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1399-1405

```cpp
1399:   auto elemTy = tensorTy.getElementType();
1400:   // Get the pointee type if we have a tensor of ptrs to compute contiguity for
1401:   if (auto ptrTy = dyn_cast<PointerType>(elemTy)) {
1402:     elemTy = ptrTy.getPointeeType();
1403:   }
1404:   return getAlignment(value, elemTy.getIntOrFloatBitWidth());
1405: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1407-1413

```cpp
1407: unsigned ModuleAxisInfoAnalysis::getAlignment(Value offsetsValue,
1408:                                               unsigned elementBitWidth) {
1409:   auto tensorTy = cast<RankedTensorType>(offsetsValue.getType());
1410:   auto *axisInfo = getAxisInfo(offsetsValue);
1411:   if (!axisInfo)
1412:     return 1;
1413:   auto order = gpu::getOrder(tensorTy);
```

- **EN:** Defines accessor/helper `ModuleAxisInfoAnalysis::getAlignment` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `ModuleAxisInfoAnalysis::getAlignment`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 1415-1420

```cpp
1415:   auto divisibility = axisInfo->getDivisibility(order[0]);
1416:   auto elemNumBytes = std::max<unsigned>(elementBitWidth / 8, 1);
1417:   auto elemTy = tensorTy.getElementType();
1418:   auto maxMultiple = isa<PointerType>(elemTy)
1419:                          ? std::max<int64_t>(divisibility / elemNumBytes, 1)
1420:                          : divisibility;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1422-1435

```cpp
1422:   auto maxContig = axisInfo->getContiguity(order[0]);
1423:   unsigned alignment = std::min(maxMultiple, maxContig);
1424:   LDBG("getAlignment order[0] " << order[0] << " maxContig = " << maxContig
1425:                                 << " elemNumBits = " << elementBitWidth
1426:                                 << " maxMultiple = " << maxMultiple
1427:                                 << " alignment " << alignment);
1428:   LLVM_DEBUG({
1429:     std::string axisStr;
1430:     llvm::raw_string_ostream os(axisStr);
1431:     axisInfo->print(os);
1432:     LDBG("-- " << axisStr);
1433:   });
1434:   return alignment;
1435: }
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 1437-1454

```cpp
1437: unsigned ModuleAxisInfoAnalysis::getMaskAlignment(Value mask) {
1438:   auto tensorTy = dyn_cast<RankedTensorType>(mask.getType());
1439:   if (!tensorTy)
1440:     return 1;
1441:   auto *axisInfo = getAxisInfo(mask);
1442:   if (!axisInfo)
1443:     return 1;
1444:   auto maskOrder = gpu::getOrder(tensorTy);
1445:   auto alignment = std::max<unsigned>(axisInfo->getConstancy(maskOrder[0]), 1);
1446:   LDBG("getMaskAlignment maskOrder[0] " << maskOrder[0] << " alignment "
1447:                                         << alignment);
1448:   LLVM_DEBUG({
1449:     std::string axisStr;
1450:     llvm::raw_string_ostream os(axisStr);
1451:     axisInfo->print(os);
1452:     LDBG("-- " << axisStr);
1453:   });
1454:   return alignment;
```

- **EN:** Defines debug logging helpers and compile-time tags used when tracing this pass or utility at runtime.
- **CN:** 这里定义调试日志辅助宏和编译期标签，用于在运行时跟踪该 pass 或工具。
### Lines 1455-1455

```cpp
1455: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 1457-1462

```cpp
1457: void ModuleAxisInfoAnalysis::initialize(
1458:     FunctionOpInterface funcOp, AxisInfoAnalysis::LoadCallback loadAnalysis) {
1459:   std::unique_ptr<DataFlowSolver> solver = createDataFlowSolver();
1460:   AxisInfoAnalysis *analysis = loadAnalysis(solver.get());
1461:   if (failed(solver->initializeAndRun(funcOp)))
1462:     return;
```

- **EN:** Defines `ModuleAxisInfoAnalysis::initialize`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ModuleAxisInfoAnalysis::initialize`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1464-1481

```cpp
1464:   auto *axisInfoMap = getFuncData(funcOp);
1465:   auto updateAxisInfoMap = [&](Value value) {
1466:     auto axisInfo = analysis->getLatticeElement(value)->getValue();
1467:     // If we could not determine the AxisInfo for this value, assume the
1468:     // pessimistic state.
1469:     if (axisInfo.getRank() == 0)
1470:       axisInfo = AxisInfo::getPessimisticValueState(value);
1471:     auto &valInfo = (*axisInfoMap)[value];
1472:     valInfo = AxisInfo::join(axisInfo, valInfo);
1473:   };
1474:   funcOp.walk([&](Operation *op) {
1475:     for (auto value : op->getResults()) {
1476:       updateAxisInfoMap(value);
1477:     }
1478:   });
1479:   funcOp.walk([&](Block *block) {
1480:     for (auto value : block->getArguments()) {
1481:       updateAxisInfoMap(value);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 1482-1484

```cpp
1482:     }
1483:   });
1484: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1486-1503

```cpp
1486: void ModuleAxisInfoAnalysis::update(CallOpInterface callOp,
1487:                                     FunctionOpInterface callee) {
1488:   auto caller = callOp->getParentOfType<FunctionOpInterface>();
1489:   auto *axisInfoMap = getFuncData(caller);
1490:   for (auto entry : llvm::enumerate(callOp->getOperands())) {
1491:     auto index = entry.index();
1492:     auto value = entry.value();
1493:     auto setAttrFn = [&](StringRef attrName, int64_t prevValue) {
1494:       auto curValue = kMaxDivisor;
1495:       if (callee.getArgAttrOfType<IntegerAttr>(index, attrName)) {
1496:         curValue =
1497:             callee.getArgAttrOfType<IntegerAttr>(index, attrName).getInt();
1498:       }
1499:       auto attr = IntegerAttr::get(IntegerType::get(callee.getContext(), 64),
1500:                                    gcd(prevValue, curValue));
1501:       callee.setArgAttr(index, attrName, attr);
1502:     };
1503:     auto axisInfo = axisInfoMap->lookup(value);
```

- **EN:** Defines `ModuleAxisInfoAnalysis::update`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ModuleAxisInfoAnalysis::update`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 1504-1512

```cpp
1504:     // Only scalar arguments are supported. Do not forward multi-dimensional
1505:     // AxisInfo to the callee.
1506:     if (axisInfo.getRank() != 1)
1507:       continue;
1508:     setAttrFn("tt.contiguity", axisInfo.getContiguity(0));
1509:     setAttrFn("tt.divisibility", axisInfo.getDivisibility(0));
1510:     setAttrFn("tt.constancy", axisInfo.getConstancy(0));
1511:   }
1512: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 1514-1514

```cpp
1514: } // namespace mlir::triton
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main theme is static analysis around axis info, so correctness depends on conservative fact propagation.
  **CN:** 主线是围绕 Axis Info 的静态分析，因此正确性依赖保守的信息传播。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Dataflow analysis tracks facts across operations to make conservative optimization decisions.
  **CN:** 数据流分析会跨操作跟踪事实，以做出保守但有效的优化决策。
- **EN:** Axis information captures per-dimension contiguity, divisibility, or constancy facts.
  **CN:** AxisInfo 记录逐维的连续性、可整除性或常量性等信息。
- **EN:** Layout and encoding decisions are first-class because they determine how work and memory are distributed on the GPU.
  **CN:** 布局与编码决策是第一类问题，因为它们决定了 GPU 上工作与内存的分布方式。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Analysis/AxisInfo.h`, `triton/Dialect/Gluon/IR/Dialect.h`, `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/Triton/IR/Utility.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`
- **MLIR headers / MLIR 头文件:** `mlir/Analysis/DataFlowFramework.h`, `mlir/Dialect/UB/IR/UBOps.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/bit.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`
- **Standard/library headers / 标准或通用库头文件:** `numeric`
- **Primary APIs used / 主要 API:** `RankedTensorType`
