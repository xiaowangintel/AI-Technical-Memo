# PlanCTA.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonNvidiaGPU/Transforms/PlanCTA.cpp`
- **Purpose / 作用:** **EN:** Implements the Plan CTA transformation or optimization pass for the TritonNvidiaGPU pipeline. **CN:** 为 TritonNvidiaGPU 编译流程实现与 Plan CTA 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
   1: /*
   2:  * Copyright (c) 2023 NVIDIA Corporation & Affiliates. All rights reserved.
   3:  *
   4:  * Permission is hereby granted, free of charge, to any person obtaining
   5:  * a copy of this software and associated documentation files
   6:  * (the "Software"), to deal in the Software without restriction,
   7:  * including without limitation the rights to use, copy, modify, merge,
   8:  * publish, distribute, sublicense, and/or sell copies of the Software,
   9:  * and to permit persons to whom the Software is furnished to do so,
  10:  * subject to the following conditions:
  11:  *
  12:  * The above copyright notice and this permission notice shall be
  13:  * included in all copies or substantial portions of the Software.
  14:  *
  15:  * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
  16:  * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
  17:  * MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
  18:  * IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 19-22

```cpp
  19:  * CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
  20:  * TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
  21:  * SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
  22:  */
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 24-24

```cpp
  24: #include <queue>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (None) provide domain-specific IR/support, MLIR headers (None) provide rewriting and analysis infrastructure, LLVM headers (None) supply low-level utilities, and standard/library headers (`queue`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（None）提供领域特定 IR/支持逻辑，MLIR 头文件（None）提供重写与分析基础设施，LLVM 头文件（None）提供底层工具，而标准/通用库头文件（`queue`）提供通用能力。
### Lines 26-32

```cpp
  26: #include "mlir/Support/LLVM.h"
  27: #include "triton/Dialect/Triton/IR/Dialect.h"
  28: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  29: #include "triton/Dialect/TritonNvidiaGPU/IR/Dialect.h"
  30: #include "triton/Dialect/TritonNvidiaGPU/Transforms/Passes.h"
  31: #include "llvm/ADT/STLExtras.h"
  32: #include "llvm/Support/ErrorHandling.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Dialect.h`, `Dialect.h`, `Dialect.h`, `Passes.h`) provide domain-specific IR/support, MLIR headers (`LLVM.h`) provide rewriting and analysis infrastructure, LLVM headers (`STLExtras.h`, `ErrorHandling.h`) supply low-level utilities, and standard/library headers (None) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Dialect.h`, `Dialect.h`, `Dialect.h`, `Passes.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`LLVM.h`）提供重写与分析基础设施，LLVM 头文件（`STLExtras.h`, `ErrorHandling.h`）提供底层工具，而标准/通用库头文件（None）提供通用能力。
### Lines 34-34

```cpp
  34: namespace ttg = mlir::triton::gpu;
```

- **EN:** Opens or closes the namespace nesting for ttg, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 ttg 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 36-38

```cpp
  36: namespace mlir {
  37: namespace triton {
  38: namespace nvidia_gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir -> triton -> nvidia_gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir -> triton -> nvidia_gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 40-41

```cpp
  40: #define GEN_PASS_DEF_TRITONGPUPLANCTAPASS
  41: #include "triton/Dialect/TritonNvidiaGPU/Transforms/Passes.h.inc"
```

- **EN:** Pulls in generated definitions from TableGen/MLIR include fragments so the handwritten code can reuse auto-generated declarations.
- **CN:** 这里引入由 TableGen/MLIR 生成的定义片段，使手写代码能够复用自动生成的声明。
### Lines 43-43

```cpp
  43: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 45-46

```cpp
  45: // TODO: use ConvertLayoutOp
  46: using CastOp = ::mlir::UnrealizedConversionCastOp;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 48-50

```cpp
  48: unsigned getNumUsers(Value value) {
  49:   return std::distance(value.user_begin(), value.user_end());
  50: }
```

- **EN:** Defines accessor/helper `getNumUsers` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getNumUsers`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 52-62

```cpp
  52: Type replaceLayout(const Type &type, const Attribute &newLayout) {
  53:   Type curType = type;
  54:   auto ptrTy = dyn_cast<triton::PointerType>(curType);
  55:   if (ptrTy)
  56:     curType = ptrTy.getPointeeType();
  57:   if (auto tensorTy = dyn_cast<RankedTensorType>(curType))
  58:     curType = tensorTy.cloneWithEncoding(newLayout);
  59:   if (ptrTy)
  60:     curType = triton::PointerType::get(curType, ptrTy.getAddressSpace());
  61:   return curType;
  62: }
```

- **EN:** Defines `replaceLayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `replaceLayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 64-81

```cpp
  64: ttg::DistributedEncodingTrait
  65: replaceCGALayout(ttg::DistributedEncodingTrait layout,
  66:                  llvm::ArrayRef<int64_t> shape, int numWarps,
  67:                  ttg::CGAEncodingAttr newCGALayout) {
  68:   if (auto blockedLayout = mlir::dyn_cast<ttg::BlockedEncodingAttr>(layout)) {
  69:     return ttg::BlockedEncodingAttr::get(
  70:         layout.getContext(), shape, blockedLayout.getSizePerThread(),
  71:         blockedLayout.getOrder(), numWarps, 32, newCGALayout);
  72:   } else if (auto sliceLayout =
  73:                  mlir::dyn_cast<ttg::SliceEncodingAttr>(layout)) {
  74:     return ttg::SliceEncodingAttr::get(
  75:         layout.getContext(), sliceLayout.getDim(),
  76:         replaceCGALayout(sliceLayout.getParent(), shape, numWarps,
  77:                          newCGALayout));
  78:   } else {
  79:     // Other layouts are generated by passes after PlanCTAPass
  80:     llvm::report_fatal_error("replaceCGALayout not implemented");
  81:     return layout;
```

- **EN:** Defines `replaceCGALayout`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `replaceCGALayout`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 82-83

```cpp
  82:   }
  83: }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 85-87

```cpp
  85: class CTAPlanner {
  86: public:
  87:   CTAPlanner();
```

- **EN:** Defines `CTAPlanner`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 89-89

```cpp
  89:   void run(triton::FuncOp &funcOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 91-95

```cpp
  91: private:
  92:   CastOp markBackward(CastOp cast) const;
  93:   CastOp markForward(CastOp cast) const;
  94:   bool isBackward(CastOp cast) const;
  95:   bool isForward(CastOp cast) const;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 97-99

```cpp
  97:   bool processDot(triton::FuncOp &funcOp);
  98:   bool processReduce(triton::FuncOp &funcOp);
  99:   void processStoreLikeOps(triton::FuncOp &funcOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 101-103

```cpp
 101:   bool propagate(CastOp cast);
 102:   bool propagateBackward(CastOp cast);
 103:   bool propagateForward(CastOp cast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 105-107

```cpp
 105:   void eraseCastOp(CastOp cast);
 106:   void eraseCastOpFromQueue(CastOp cast);
 107:   void eraseCastOpsFromQueue(llvm::ArrayRef<CastOp> casts);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 109-111

```cpp
 109:   void insertCasts(Operation *op, llvm::ArrayRef<Attribute> newOperandLayouts,
 110:                    llvm::ArrayRef<Attribute> newResultLayouts);
 111:   void eliminateAdjacentCasts(CastOp cast0, CastOp cast1);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 113-114

```cpp
 113:   bool isLoadStoreOp(Operation *op) const;
 114:   bool processLoadStore(Operation *op, Attribute layout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 116-117

```cpp
 116:   bool isElementwiseOp(Operation *op) const;
 117:   bool processElementwise(Operation *op, Attribute layout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 119-121

```cpp
 119:   bool processConstant(arith::ConstantOp constant, Attribute layout);
 120:   bool processSplat(triton::SplatOp splat, Attribute layout);
 121:   bool processMakeRange(triton::MakeRangeOp makeRange, Attribute layout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 123-125

```cpp
 123:   bool processBroadcast(triton::BroadcastOp broadcast, Attribute layout);
 124:   bool processExpandDimsBackward(triton::ExpandDimsOp expandDims,
 125:                                  ttg::DistributedEncodingTrait newResultLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 127-130

```cpp
 127:   bool processConvertLayoutBackward(ttg::ConvertLayoutOp convertLayout,
 128:                                     CastOp cast);
 129:   bool processConvertLayoutForward(ttg::ConvertLayoutOp convertLayout,
 130:                                    CastOp cast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 132-133

```cpp
 132:   bool processIfOp(scf::IfOp ifOp, int index, const Type &newType);
 133:   bool processForOp(scf::ForOp forOp, int index, const Type &newType);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 135-139

```cpp
 135:   bool processIfOpBackward(scf::IfOp ifOp, CastOp cast);
 136:   bool processForOpBackward(scf::ForOp forOp, CastOp cast);
 137:   bool processBlockArgBackward(BlockArgument arg, CastOp cast);
 138:   bool processForOpForward(scf::ForOp forOp, CastOp cast);
 139:   bool processYieldOpForward(scf::YieldOp yieldOp, CastOp cast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 141-141

```cpp
 141:   bool processOpFallback(Operation *op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 143-144

```cpp
 143:   bool processMultiUsersBackward(Value input, CastOp cast);
 144:   bool processMultiUsersForward(Value output, CastOp cast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 146-146

```cpp
 146:   void markTiled();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 148-152

```cpp
 148:   unsigned step;
 149:   unsigned stepUnchanged;
 150:   bool tiled;
 151:   std::queue<CastOp> queue;
 152: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 154-154

```cpp
 154: CTAPlanner::CTAPlanner() : step(0), stepUnchanged(0), tiled(false) {}
```

- **EN:** Defines `CTAPlanner::CTAPlanner`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::CTAPlanner`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 156-157

```cpp
 156: void CTAPlanner::run(triton::FuncOp &funcOp) {
 157:   static const unsigned maxSteps = 10000;
```

- **EN:** Defines `CTAPlanner::run`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::run`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 159-162

```cpp
 159:   auto nextStep = [&]() {
 160:     ++step;
 161:     assert(step < maxSteps && "Maximum number of steps exceeded");
 162:   };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 164-165

```cpp
 164:   processDot(funcOp);
 165:   nextStep();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 167-168

```cpp
 167:   processReduce(funcOp);
 168:   nextStep();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 170-173

```cpp
 170:   if (!tiled) {
 171:     processStoreLikeOps(funcOp);
 172:     nextStep();
 173:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 175-187

```cpp
 175:   while (!queue.empty()) {
 176:     CastOp cast = queue.front();
 177:     queue.pop();
 178:     bool changed = propagate(cast);
 179:     if (changed) {
 180:       stepUnchanged = 0;
 181:     } else {
 182:       queue.push(cast);
 183:       ++stepUnchanged;
 184:     }
 185:     nextStep();
 186:   }
 187: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 189-192

```cpp
 189: CastOp CTAPlanner::markBackward(CastOp cast) const {
 190:   cast->setAttr("direction", StringAttr::get(cast.getContext(), "backward"));
 191:   return cast;
 192: }
```

- **EN:** Defines `CTAPlanner::markBackward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::markBackward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 194-197

```cpp
 194: CastOp CTAPlanner::markForward(CastOp cast) const {
 195:   cast->setAttr("direction", StringAttr::get(cast.getContext(), "forward"));
 196:   return cast;
 197: }
```

- **EN:** Defines `CTAPlanner::markForward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::markForward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 199-201

```cpp
 199: bool CTAPlanner::isBackward(CastOp cast) const {
 200:   return cast->getAttrOfType<StringAttr>("direction") == "backward";
 201: }
```

- **EN:** Defines `CTAPlanner::isBackward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::isBackward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 203-205

```cpp
 203: bool CTAPlanner::isForward(CastOp cast) const {
 204:   return cast->getAttrOfType<StringAttr>("direction") == "forward";
 205: }
```

- **EN:** Defines `CTAPlanner::isForward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::isForward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 207-210

```cpp
 207: void CTAPlanner::markTiled() {
 208:   assert(!tiled && "CTA tiling is already determined");
 209:   tiled = true;
 210: }
```

- **EN:** Defines `CTAPlanner::markTiled`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `CTAPlanner::markTiled`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 212-227

```cpp
 212: bool CTAPlanner::processDot(triton::FuncOp &funcOp) {
 213:   // TODO: This is a naive implementation and should be refactored
 214:   auto getCTATiling = [](int64_t M, int64_t N, int64_t K,
 215:                          unsigned numCTAs) -> std::pair<unsigned, unsigned> {
 216:     // prefer a larger chunk size, at most 128; first assign splitM.
 217:     unsigned chunk_m = 128;
 218:     auto isLegal = [](unsigned chunk) { return chunk >= 64; };
 219:     unsigned splitM, splitN;
 220:     for (; isLegal(chunk_m); chunk_m /= 2) {
 221:       splitM = std::clamp<unsigned>(M / chunk_m, 1, numCTAs);
 222:       splitN = numCTAs / splitM;
 223:       if (isLegal(N / splitN)) // chunk_n;
 224:         break;
 225:     }
 226:     return {splitM, splitN};
 227:   };
```

- **EN:** Defines `CTAPlanner::processDot`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processDot`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 229-230

```cpp
 229:   funcOp.walk([&](triton::DotOp dot) {
 230:     MLIRContext *ctx = dot.getContext();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 232-234

```cpp
 232:     auto aTy = cast<RankedTensorType>(dot.getA().getType());
 233:     auto bTy = cast<RankedTensorType>(dot.getB().getType());
 234:     auto dTy = cast<RankedTensorType>(dot.getD().getType());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 236-239

```cpp
 236:     assert(isa<ttg::DotOperandEncodingAttr>(aTy.getEncoding()) &&
 237:            isa<ttg::DotOperandEncodingAttr>(bTy.getEncoding()) &&
 238:            isa<ttg::BlockedEncodingAttr>(dTy.getEncoding()) &&
 239:            "PlanCTAPass should follow immediately after CoalescePass");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 241-243

```cpp
 241:     auto aLayout = cast<ttg::DotOperandEncodingAttr>(aTy.getEncoding());
 242:     auto bLayout = cast<ttg::DotOperandEncodingAttr>(bTy.getEncoding());
 243:     auto dLayout = cast<ttg::BlockedEncodingAttr>(dTy.getEncoding());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 245-247

```cpp
 245:     unsigned M = dTy.getShape()[0];
 246:     unsigned N = dTy.getShape()[1];
 247:     unsigned K = aTy.getShape()[1];
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 249-252

```cpp
 249:     unsigned splitM, splitN;
 250:     std::tie(splitM, splitN) = getCTATiling(M, N, K, ttg::getNumCTAs(dLayout));
 251:     // FIXME: Should consider IR with more than one DotOps
 252:     markTiled();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 254-256

```cpp
 254:     OpBuilder builder(dot);
 255:     auto numThreads = ttg::lookupThreadsPerWarp(builder);
 256:     auto numWarps = ttg::lookupNumWarps(dot);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 258-266

```cpp
 258:     auto newCGALayout = ttg::CGAEncodingAttr::fromSplitParams(
 259:         ctx, {splitM, splitN}, {splitM, splitN}, {1, 0});
 260:     auto newDLayout = ttg::BlockedEncodingAttr::get(
 261:         ctx, dTy.getShape(), dLayout.getSizePerThread(), dLayout.getOrder(),
 262:         numWarps, numThreads, newCGALayout);
 263:     auto newALayout = ttg::DotOperandEncodingAttr::get(ctx, aLayout.getOpIdx(),
 264:                                                        newDLayout, 0);
 265:     auto newBLayout = ttg::DotOperandEncodingAttr::get(ctx, bLayout.getOpIdx(),
 266:                                                        newDLayout, 0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 268-270

```cpp
 268:     insertCasts(dot.getOperation(), {newALayout, newBLayout, newDLayout},
 269:                 {newDLayout});
 270:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 272-273

```cpp
 272:   return true;
 273: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 275-277

```cpp
 275: bool CTAPlanner::processReduce(triton::FuncOp &funcOp) {
 276:   ModuleOp mod = funcOp->getParentOfType<ModuleOp>();
 277:   unsigned numCTAs = ttg::TritonGPUDialect::getNumCTAs(mod);
```

- **EN:** Defines `CTAPlanner::processReduce`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processReduce`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 279-282

```cpp
 279:   funcOp.walk([&](triton::ReduceOp reduce) {
 280:     MLIRContext *context = reduce.getContext();
 281:     Value src = reduce.getOperands()[0];
 282:     unsigned axis = reduce.getAxis();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 284-286

```cpp
 284:     auto srcTy = cast<RankedTensorType>(src.getType());
 285:     auto srcShape = srcTy.getShape();
 286:     auto srcLayout = srcTy.getEncoding();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 288-291

```cpp
 288:     auto rank = srcShape.size();
 289:     auto order = ttg::getOrder(srcTy);
 290:     auto sizePerThread = ttg::getContigPerThread(srcTy);
 291:     auto CTAOrder = ttg::getCTAOrder(srcLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 293-304

```cpp
 293:     llvm::SmallVector<unsigned> CTAsPerCGA(rank, 0);
 294:     unsigned remainingCTAs = numCTAs;
 295:     for (int i = rank - 1; i >= 0; --i) {
 296:       unsigned dim = order[i];
 297:       if (dim == axis) {
 298:         CTAsPerCGA[dim] = 1;
 299:       } else {
 300:         CTAsPerCGA[dim] = std::min<unsigned>(srcShape[dim] / sizePerThread[dim],
 301:                                              remainingCTAs);
 302:         remainingCTAs /= CTAsPerCGA[dim];
 303:       }
 304:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 306-312

```cpp
 306:     for (int i = rank - 1; i >= 0; --i) {
 307:       unsigned dim = order[i];
 308:       if (dim != axis) {
 309:         CTAsPerCGA[dim] *= remainingCTAs;
 310:         break;
 311:       }
 312:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 314-314

```cpp
 314:     llvm::SmallVector<unsigned> CTASplitNum = CTAsPerCGA;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 316-322

```cpp
 316:     // If numCTAs > 1 and the only dimension is the reduced dimension, the
 317:     // loops above leave all CTAs unassigned. Put the remaining CTAs on that
 318:     // dimension so that they all collaborate in the reduction.
 319:     if (remainingCTAs > 0) {
 320:       CTAsPerCGA[order[rank - 1]] *= remainingCTAs;
 321:       CTASplitNum[order[rank - 1]] = CTAsPerCGA[order[rank - 1]];
 322:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 324-340

```cpp
 324:     auto numWarps = ttg::lookupNumWarps(reduce);
 325:     auto CGALayout = ttg::CGAEncodingAttr::fromSplitParams(
 326:         context, CTAsPerCGA, CTASplitNum, CTAOrder);
 327:     if (!tiled)
 328:       markTiled();
 329:     auto newSrcLayout =
 330:         replaceCGALayout(cast<ttg::DistributedEncodingTrait>(srcLayout),
 331:                          srcShape, numWarps, CGALayout);
 332:     unsigned numOperands = reduce.getNumOperands();
 333:     unsigned numResults = reduce.getNumResults();
 334:     SmallVector<Attribute> newSrcLayoutVec(numOperands, newSrcLayout);
 335:     Attribute newResultLayout;
 336:     if (rank > 1) {
 337:       newResultLayout =
 338:           ttg::SliceEncodingAttr::get(context, axis, newSrcLayout);
 339:     }
 340:     SmallVector<Attribute> newResultLayoutVec(numResults, newResultLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 342-345

```cpp
 342:     insertCasts(reduce.getOperation(), newSrcLayoutVec, newResultLayoutVec);
 343:   });
 344:   return true;
 345: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 347-348

```cpp
 347: void CTAPlanner::processStoreLikeOps(triton::FuncOp &funcOp) {
 348:   assert(!tiled && "CTA tiling is already determined");
```

- **EN:** Defines `CTAPlanner::processStoreLikeOps`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `CTAPlanner::processStoreLikeOps`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 350-357

```cpp
 350:   llvm::SmallVector<Operation *> stores;
 351:   funcOp.walk([&](Operation *op) {
 352:     if (llvm::isa<triton::StoreOp, triton::AtomicRMWOp, triton::AtomicCASOp,
 353:                   triton::DescriptorStoreLikeOpInterface>(op))
 354:       stores.push_back(op);
 355:   });
 356:   assert(stores.size() > 0 && "Cannot find store-like ops");
 357:   auto numWarps = ttg::lookupNumWarps(funcOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 359-376

```cpp
 359:   ttg::CGAEncodingAttr CGALayout;
 360:   for (Operation *store : stores) {
 361:     auto val = [store]() -> Value {
 362:       if (auto descStore =
 363:               dyn_cast<triton::DescriptorStoreLikeOpInterface>(store))
 364:         return descStore.getSrc();
 365:       return store->getOperand(0);
 366:     }();
 367:     if (auto tensorTy = dyn_cast<RankedTensorType>(val.getType())) {
 368:       if (!tiled) {
 369:         // Use CTA tiling of the first store-like op as global CTA tiling
 370:         CGALayout = ttg::getCGALayout(tensorTy.getEncoding());
 371:         markTiled();
 372:       }
 373:       auto newLayout = replaceCGALayout(
 374:           cast<ttg::DistributedEncodingTrait>(tensorTy.getEncoding()),
 375:           tensorTy.getShape(), numWarps, CGALayout);
 376:       processElementwise(store, newLayout);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 377-378

```cpp
 377:     }
 378:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 380-382

```cpp
 380:   if (!tiled)
 381:     markTiled();
 382: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 384-386

```cpp
 384: bool CTAPlanner::propagate(CastOp cast) {
 385:   return isBackward(cast) ? propagateBackward(cast) : propagateForward(cast);
 386: }
```

- **EN:** Defines `CTAPlanner::propagate`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::propagate`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 388-405

```cpp
 388: bool CTAPlanner::propagateBackward(CastOp cast) {
 389:   Value input = cast.getOperand(0);
 390:   Value output = cast.getResult(0);
 391:   unsigned numUsers = getNumUsers(input);
 392:   if (numUsers == 0) {
 393:     llvm::report_fatal_error("Unreachable branch");
 394:     return false;
 395:   } else if (numUsers == 1) {
 396:     Type outTy = output.getType();
 397:     if (auto ptrTy = dyn_cast<triton::PointerType>(outTy))
 398:       outTy = ptrTy.getPointeeType();
 399:     auto layout = mlir::cast<ttg::DistributedEncodingTrait>(
 400:         mlir::cast<RankedTensorType>(outTy).getEncoding());
 401:     Operation *op = input.getDefiningOp();
 402:     if (op == nullptr) {
 403:       assert(isa<BlockArgument>(input) &&
 404:              "Unexpected Value without defining op");
 405:       processBlockArgBackward(llvm::cast<BlockArgument>(input), cast);
```

- **EN:** Defines `CTAPlanner::propagateBackward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `CTAPlanner::propagateBackward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 406-423

```cpp
 406:     } else if (auto prevCast = llvm::dyn_cast<CastOp>(op)) {
 407:       eliminateAdjacentCasts(prevCast, cast);
 408:     } else if (isLoadStoreOp(op)) {
 409:       processLoadStore(op, layout);
 410:     } else if (isElementwiseOp(op)) {
 411:       processElementwise(op, layout);
 412:     } else if (auto constant = llvm::dyn_cast<arith::ConstantOp>(op)) {
 413:       processConstant(constant, layout);
 414:     } else if (auto splat = llvm::dyn_cast<triton::SplatOp>(op)) {
 415:       processSplat(splat, layout);
 416:     } else if (auto makeRange = llvm::dyn_cast<triton::MakeRangeOp>(op)) {
 417:       processMakeRange(makeRange, layout);
 418:     } else if (auto broadcast = llvm::dyn_cast<triton::BroadcastOp>(op)) {
 419:       processBroadcast(broadcast, layout);
 420:     } else if (auto expandDims = llvm::dyn_cast<triton::ExpandDimsOp>(op)) {
 421:       processExpandDimsBackward(expandDims, layout);
 422:     } else if (auto ifOp = llvm::dyn_cast<scf::IfOp>(op)) {
 423:       processIfOpBackward(ifOp, cast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 424-436

```cpp
 424:     } else if (auto forOp = llvm::dyn_cast<scf::ForOp>(op)) {
 425:       processForOpBackward(forOp, cast);
 426:     } else if (auto convertLayout = llvm::dyn_cast<ttg::ConvertLayoutOp>(op)) {
 427:       return processConvertLayoutBackward(convertLayout, cast);
 428:     } else {
 429:       // Keep original layouts. This may result in a loss of performance.
 430:       return processOpFallback(op);
 431:     }
 432:     return true;
 433:   } else {
 434:     return processMultiUsersBackward(input, cast);
 435:   }
 436: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 438-455

```cpp
 438: bool CTAPlanner::propagateForward(CastOp cast) {
 439:   Value input = cast.getOperand(0);
 440:   Value output = cast.getResult(0);
 441:   unsigned numUsers = getNumUsers(output);
 442:   if (numUsers == 0) {
 443:     cast.erase();
 444:   } else if (numUsers == 1) {
 445:     Type inTy = input.getType();
 446:     if (auto ptrTy = dyn_cast<triton::PointerType>(inTy))
 447:       inTy = ptrTy.getPointeeType();
 448:     Attribute layout = mlir::cast<RankedTensorType>(inTy).getEncoding();
 449:     Operation *op = *output.user_begin();
 450:     if (auto nextCast = llvm::dyn_cast<CastOp>(op)) {
 451:       eliminateAdjacentCasts(cast, nextCast);
 452:     } else if (isLoadStoreOp(op)) {
 453:       processLoadStore(op, layout);
 454:     } else if (isElementwiseOp(op)) {
 455:       processElementwise(op, layout);
```

- **EN:** Defines `CTAPlanner::propagateForward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::propagateForward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 456-470

```cpp
 456:     } else if (auto convertLayout = llvm::dyn_cast<ttg::ConvertLayoutOp>(op)) {
 457:       return processConvertLayoutForward(convertLayout, cast);
 458:     } else if (auto forOp = llvm::dyn_cast<scf::ForOp>(op)) {
 459:       processForOpForward(forOp, cast);
 460:     } else if (auto yieldOp = llvm::dyn_cast<scf::YieldOp>(op)) {
 461:       processYieldOpForward(yieldOp, cast);
 462:     } else {
 463:       // Keep original layouts. This may result in a loss of performance.
 464:       return processOpFallback(op);
 465:     }
 466:   } else {
 467:     processMultiUsersForward(output, cast);
 468:   }
 469:   return true;
 470: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 472-477

```cpp
 472: void CTAPlanner::eraseCastOp(CastOp cast) {
 473:   Value output = cast.getResult(0);
 474:   assert(getNumUsers(output) == 0 &&
 475:          "Cannot erase CastOp because it is still in use");
 476:   cast.erase();
 477: }
```

- **EN:** Defines `CTAPlanner::eraseCastOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `CTAPlanner::eraseCastOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 479-481

```cpp
 479: void CTAPlanner::eraseCastOpFromQueue(CastOp cast) {
 480:   eraseCastOpsFromQueue({cast});
 481: }
```

- **EN:** Defines `CTAPlanner::eraseCastOpFromQueue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::eraseCastOpFromQueue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 483-488

```cpp
 483: void CTAPlanner::eraseCastOpsFromQueue(llvm::ArrayRef<CastOp> casts) {
 484:   llvm::DenseSet<CastOp> erased;
 485:   for (CastOp cast : casts) {
 486:     eraseCastOp(cast);
 487:     erased.insert(cast);
 488:   }
```

- **EN:** Defines `CTAPlanner::eraseCastOpsFromQueue`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::eraseCastOpsFromQueue`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 490-491

```cpp
 490:   decltype(queue) tempQueue;
 491:   std::swap(queue, tempQueue);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 493-500

```cpp
 493:   // This is only a naive implementation. Should refactor with linked-list.
 494:   while (!tempQueue.empty()) {
 495:     auto cast = tempQueue.front();
 496:     tempQueue.pop();
 497:     if (!erased.contains(cast))
 498:       queue.push(cast);
 499:   }
 500: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 502-508

```cpp
 502: void CTAPlanner::insertCasts(Operation *op,
 503:                              llvm::ArrayRef<Attribute> newOperandLayouts,
 504:                              llvm::ArrayRef<Attribute> newResultLayouts) {
 505:   assert(op->getNumOperands() == newOperandLayouts.size() &&
 506:          "NumOperands mismatched");
 507:   assert(op->getNumResults() == newResultLayouts.size() &&
 508:          "NumResults mismatched");
```

- **EN:** Defines `CTAPlanner::insertCasts`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `CTAPlanner::insertCasts`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 510-511

```cpp
 510:   Location loc = op->getLoc();
 511:   OpBuilder builder(op->getContext());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 513-524

```cpp
 513:   builder.setInsertionPoint(op);
 514:   for (unsigned i = 0; i < op->getNumOperands(); ++i) {
 515:     Value operand = op->getOperand(i);
 516:     auto operandTy = operand.getType();
 517:     if (isa<RankedTensorType>(operandTy)) {
 518:       operandTy = replaceLayout(operandTy, newOperandLayouts[i]);
 519:       auto cast =
 520:           markBackward(CastOp::create(builder, loc, operandTy, operand));
 521:       op->setOperand(i, cast.getResult(0));
 522:       queue.push(cast);
 523:     }
 524:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 526-539

```cpp
 526:   builder.setInsertionPointAfter(op);
 527:   for (unsigned i = 0; i < op->getNumResults(); ++i) {
 528:     Value result = op->getResult(i);
 529:     auto resultTy = result.getType();
 530:     if (isa<RankedTensorType>(resultTy)) {
 531:       resultTy = replaceLayout(resultTy, newResultLayouts[i]);
 532:       auto cast =
 533:           markForward(CastOp::create(builder, loc, result.getType(), result));
 534:       result.setType(resultTy);
 535:       result.replaceAllUsesExcept(cast.getResult(0), cast.getOperation());
 536:       queue.push(cast);
 537:     }
 538:   }
 539: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 541-545

```cpp
 541: void CTAPlanner::eliminateAdjacentCasts(CastOp cast0, CastOp cast1) {
 542:   assert(cast0.getResult(0) == cast1.getOperand(0) &&
 543:          "The two casts are not adjacent");
 544:   assert(isForward(cast0) && isBackward(cast1) &&
 545:          "Expected pattern of adjacent casts: forward + backward");
```

- **EN:** Defines `CTAPlanner::eliminateAdjacentCasts`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `CTAPlanner::eliminateAdjacentCasts`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 547-548

```cpp
 547:   Value input = cast0.getOperand(0);
 548:   Value output = cast1.getResult(0);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 550-560

```cpp
 550:   if (input.getType() == output.getType()) {
 551:     output.replaceAllUsesWith(input);
 552:     eraseCastOpsFromQueue({cast1, cast0});
 553:   } else {
 554:     OpBuilder builder(cast1.getOperation());
 555:     auto cvt = ttg::ConvertLayoutOp::create(builder, cast1.getLoc(),
 556:                                             output.getType(), input);
 557:     output.replaceAllUsesWith(cvt.getResult());
 558:     eraseCastOpsFromQueue({cast1, cast0});
 559:   }
 560: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 562-565

```cpp
 562: bool CTAPlanner::isLoadStoreOp(Operation *op) const {
 563:   return llvm::isa<triton::LoadOp, triton::StoreOp, triton::AtomicRMWOp,
 564:                    triton::AtomicCASOp, triton::DescriptorOpInterface>(op);
 565: }
```

- **EN:** Defines `CTAPlanner::isLoadStoreOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::isLoadStoreOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 567-584

```cpp
 567: bool CTAPlanner::processLoadStore(Operation *op, Attribute layout) {
 568:   // Special logic for:
 569:   //     LoadOp -> SliceLayout
 570:   // Transform to:
 571:   //     LoadOp -> originalLayout -> ConvertLayout(DSmem) -> SliceLayout
 572:   if (auto sliceLayout = mlir::dyn_cast<ttg::SliceEncodingAttr>(layout)) {
 573:     auto dim = sliceLayout.getDim();
 574:     auto CTAsPerCGA = ttg::getCTAsPerCGA(sliceLayout.getParent());
 575:     if (CTAsPerCGA[dim] > 1) {
 576:       // Find an input or output value of LoadOp or StoreOp to get its layout
 577:       Value val =
 578:           op->getNumResults() > 0 ? op->getResult(0) : op->getOperand(0);
 579:       Attribute originalLayout =
 580:           cast<RankedTensorType>(val.getType()).getEncoding();
 581:       // Insert casts using originalLayout. Adjacent casts will be eliminated
 582:       // and generate a ConvertLayoutOp with DSmem access
 583:       return processLoadStore(op, originalLayout);
 584:     }
```

- **EN:** Defines `CTAPlanner::processLoadStore`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processLoadStore`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 585-585

```cpp
 585:   }
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 587-588

```cpp
 587:   auto CGALayout = ttg::getCGALayout(layout);
 588:   auto numWarps = ttg::lookupNumWarps(op);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 590-605

```cpp
 590:   llvm::SmallVector<Attribute> newOperandLayouts;
 591:   for (unsigned i = 0; i < op->getNumOperands(); ++i) {
 592:     auto type = op->getOperand(i).getType();
 593:     if (auto ptrTy = dyn_cast<triton::PointerType>(type))
 594:       type = ptrTy.getPointeeType();
 595:     auto tensorTy = dyn_cast<RankedTensorType>(type);
 596:     if (!tensorTy) {
 597:       newOperandLayouts.push_back(Attribute());
 598:       continue;
 599:     }
 600:     auto oldLayout =
 601:         cast<ttg::DistributedEncodingTrait>(tensorTy.getEncoding());
 602:     auto newLayout =
 603:         replaceCGALayout(oldLayout, tensorTy.getShape(), numWarps, CGALayout);
 604:     newOperandLayouts.push_back(newLayout);
 605:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 607-618

```cpp
 607:   llvm::SmallVector<Attribute> newResultLayouts;
 608:   for (unsigned i = 0; i < op->getNumResults(); ++i) {
 609:     auto type = op->getResult(i).getType();
 610:     if (auto ptrTy = dyn_cast<triton::PointerType>(type))
 611:       type = ptrTy.getPointeeType();
 612:     auto tensorTy = cast<RankedTensorType>(type);
 613:     auto oldLayout =
 614:         cast<ttg::DistributedEncodingTrait>(tensorTy.getEncoding());
 615:     auto newLayout =
 616:         replaceCGALayout(oldLayout, tensorTy.getShape(), numWarps, CGALayout);
 617:     newResultLayouts.push_back(newLayout);
 618:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 620-622

```cpp
 620:   insertCasts(op, newOperandLayouts, newResultLayouts);
 621:   return true;
 622: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 624-637

```cpp
 624: bool CTAPlanner::isElementwiseOp(Operation *op) const {
 625:   if (llvm::isa<arith::AddFOp, arith::AddIOp, arith::AndIOp, arith::CeilDivSIOp,
 626:                 arith::CeilDivUIOp, arith::DivFOp, arith::DivSIOp,
 627:                 arith::DivUIOp, arith::ExtFOp, arith::ExtSIOp, arith::ExtUIOp,
 628:                 arith::FloorDivSIOp, arith::FPToSIOp, arith::FPToUIOp,
 629:                 arith::MaximumFOp, arith::MaxNumFOp, arith::MaxSIOp,
 630:                 arith::MaxUIOp, arith::MinimumFOp, arith::MinNumFOp,
 631:                 arith::MinSIOp, arith::MinUIOp, arith::MulFOp, arith::MulIOp,
 632:                 arith::MulUIExtendedOp, arith::MulSIExtendedOp, arith::NegFOp,
 633:                 arith::OrIOp, arith::RemFOp, arith::RemSIOp, arith::RemUIOp,
 634:                 arith::ShLIOp, arith::ShRSIOp, arith::ShRUIOp, arith::SIToFPOp,
 635:                 arith::SubFOp, arith::SubIOp, arith::TruncFOp, arith::TruncIOp,
 636:                 arith::UIToFPOp, arith::XOrIOp>(op))
 637:     return true;
```

- **EN:** Defines `CTAPlanner::isElementwiseOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::isElementwiseOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 638-655

```cpp
 638:   if (llvm::isa<math::AbsFOp, math::AbsIOp, math::AtanOp, math::Atan2Op,
 639:                 math::CeilOp, math::CopySignOp, math::CosOp, math::SinOp,
 640:                 math::CountLeadingZerosOp, math::CountTrailingZerosOp,
 641:                 math::CtPopOp, math::ErfOp, math::ExpOp, math::Exp2Op,
 642:                 math::FloorOp, math::ExpM1Op, math::FmaOp, math::LogOp,
 643:                 math::Log10Op, math::Log1pOp, math::Log2Op, math::PowFOp,
 644:                 math::SqrtOp, math::RsqrtOp, math::TanhOp>(op))
 645:     return true;
 646:   if (llvm::isa<triton::IntToPtrOp, triton::PtrToIntOp, triton::BitcastOp,
 647:                 triton::FpToFpOp, triton::AddPtrOp, triton::PreciseSqrtOp,
 648:                 triton::PreciseDivFOp>(op))
 649:     return true;
 650:   if (auto externElementwiseOp = dyn_cast<triton::ExternElementwiseOp>(op))
 651:     return externElementwiseOp.getPure();
 652:   if (llvm::isa<arith::CmpIOp, arith::CmpFOp, arith::SelectOp>(op))
 653:     return true;
 654:   return false;
 655: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 657-662

```cpp
 657: bool CTAPlanner::processElementwise(Operation *op, Attribute layout) {
 658:   llvm::SmallVector<Attribute> newOperandLayouts(op->getNumOperands(), layout);
 659:   llvm::SmallVector<Attribute> newResultLayouts(op->getNumResults(), layout);
 660:   insertCasts(op, newOperandLayouts, newResultLayouts);
 661:   return true;
 662: }
```

- **EN:** Defines `CTAPlanner::processElementwise`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processElementwise`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 664-666

```cpp
 664: bool CTAPlanner::processConstant(arith::ConstantOp constant, Attribute layout) {
 665:   if (auto tensorTy = dyn_cast<RankedTensorType>(constant.getType())) {
 666:     if (auto attr = dyn_cast<SplatElementsAttr>(constant.getValue())) {
```

- **EN:** Defines `CTAPlanner::processConstant`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processConstant`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 668-675

```cpp
 668:       auto newTensorTy = tensorTy.cloneWithEncoding(layout);
 669:       constant.setValueAttr(
 670:           SplatElementsAttr::get(newTensorTy, attr.getSplatValue<Attribute>()));
 671:     }
 672:   }
 673:   insertCasts(constant.getOperation(), {}, {layout});
 674:   return true;
 675: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 677-680

```cpp
 677: bool CTAPlanner::processSplat(triton::SplatOp splat, Attribute layout) {
 678:   insertCasts(splat.getOperation(), {{}}, {layout});
 679:   return true;
 680: }
```

- **EN:** Defines `CTAPlanner::processSplat`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processSplat`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 682-686

```cpp
 682: bool CTAPlanner::processMakeRange(triton::MakeRangeOp makeRange,
 683:                                   Attribute layout) {
 684:   insertCasts(makeRange.getOperation(), {}, {layout});
 685:   return true;
 686: }
```

- **EN:** Defines `CTAPlanner::processMakeRange`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processMakeRange`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 688-692

```cpp
 688: bool CTAPlanner::processBroadcast(triton::BroadcastOp broadcast,
 689:                                   Attribute layout) {
 690:   insertCasts(broadcast.getOperation(), {layout}, {layout});
 691:   return true;
 692: }
```

- **EN:** Defines `CTAPlanner::processBroadcast`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processBroadcast`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 694-701

```cpp
 694: bool CTAPlanner::processExpandDimsBackward(
 695:     triton::ExpandDimsOp expandDims,
 696:     ttg::DistributedEncodingTrait newResultLayout) {
 697:   auto newSrcLayout = ttg::SliceEncodingAttr::get(
 698:       newResultLayout.getContext(), expandDims.getAxis(), newResultLayout);
 699:   insertCasts(expandDims.getOperation(), {newSrcLayout}, {newResultLayout});
 700:   return true;
 701: }
```

- **EN:** Defines `CTAPlanner::processExpandDimsBackward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processExpandDimsBackward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 703-713

```cpp
 703: bool CTAPlanner::processConvertLayoutBackward(
 704:     ttg::ConvertLayoutOp convertLayout, CastOp cast) {
 705:   Value src = convertLayout.getSrc();
 706:   Value result = convertLayout.getResult();
 707:   assert(getNumUsers(result) == 1 &&
 708:          "Expect to call processMultiUsersBackward first");
 709:   result.replaceAllUsesWith(src);
 710:   convertLayout.erase();
 711:   queue.push(cast);
 712:   return true;
 713: }
```

- **EN:** Defines `CTAPlanner::processConvertLayoutBackward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `CTAPlanner::processConvertLayoutBackward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 715-726

```cpp
 715: bool CTAPlanner::processConvertLayoutForward(ttg::ConvertLayoutOp convertLayout,
 716:                                              CastOp cast) {
 717:   Value src = convertLayout.getSrc();
 718:   Value result = convertLayout.getResult();
 719:   assert(getNumUsers(src) == 1 &&
 720:          "Expect to call processMultiUsersForward first");
 721:   src.setType(result.getType());
 722:   result.replaceAllUsesWith(src);
 723:   convertLayout.erase();
 724:   queue.push(cast);
 725:   return true;
 726: }
```

- **EN:** Defines `CTAPlanner::processConvertLayoutForward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `CTAPlanner::processConvertLayoutForward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 728-734

```cpp
 728: bool CTAPlanner::processIfOp(scf::IfOp ifOp, int index, const Type &newType) {
 729:   // Check index
 730:   assert(index < ifOp.getNumResults() && "Invalid result index of IfOp");
 731:   assert(index < ifOp.thenYield().getNumOperands() &&
 732:          "Invalid operand index of YieldOp");
 733:   assert(index < ifOp.elseYield().getNumOperands() &&
 734:          "Invalid operand index of YieldOp");
```

- **EN:** Defines `CTAPlanner::processIfOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `CTAPlanner::processIfOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 736-737

```cpp
 736:   Location loc = ifOp.getLoc();
 737:   OpBuilder builder(ifOp.getContext());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 739-746

```cpp
 739:   // Insert forward cast after ifOp
 740:   Value result = ifOp.getResult(index);
 741:   builder.setInsertionPointAfter(ifOp.getOperation());
 742:   auto newCast =
 743:       markForward(CastOp::create(builder, loc, result.getType(), result));
 744:   result.setType(newType);
 745:   result.replaceAllUsesExcept(newCast.getResult(0), newCast.getOperation());
 746:   queue.push(newCast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 748-755

```cpp
 748:   // Insert backward casts before yield
 749:   for (scf::YieldOp yield : {ifOp.thenYield(), ifOp.elseYield()}) {
 750:     Value yieldSrc = yield.getOperand(index);
 751:     builder.setInsertionPoint(yield.getOperation());
 752:     newCast = markBackward(CastOp::create(builder, loc, newType, yieldSrc));
 753:     yield->setOperand(index, newCast.getResult(0));
 754:     queue.push(newCast);
 755:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 757-758

```cpp
 757:   return true;
 758: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 760-763

```cpp
 760: bool CTAPlanner::processForOp(scf::ForOp forOp, int index,
 761:                               const Type &newType) {
 762:   Block *body = forOp.getBody();
 763:   auto yield = llvm::cast<scf::YieldOp>(forOp.getBody()->getTerminator());
```

- **EN:** Defines `CTAPlanner::processForOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processForOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 765-771

```cpp
 765:   // Check index
 766:   assert(index + forOp.getNumControlOperands() < forOp.getNumOperands() &&
 767:          "Invalid operand index of ForOp");
 768:   assert(index + forOp.getNumInductionVars() < body->getNumArguments() &&
 769:          "Invalid block arg index of ForOp");
 770:   assert(index < yield.getNumOperands() && "Invalid operand index of YieldOp");
 771:   assert(index < forOp.getNumResults() && "Invalid result index of IfOp");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 773-774

```cpp
 773:   Location loc = forOp.getLoc();
 774:   OpBuilder builder(forOp.getContext());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 776-783

```cpp
 776:   // Insert backward cast before forOp
 777:   OpOperand &operand =
 778:       forOp->getOpOperand(index + forOp.getNumControlOperands());
 779:   builder.setInsertionPoint(forOp.getOperation());
 780:   auto newCast =
 781:       markBackward(CastOp::create(builder, loc, newType, operand.get()));
 782:   operand.set(newCast.getResult(0));
 783:   queue.push(newCast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 785-791

```cpp
 785:   // Insert forward cast after block arg
 786:   Value arg = body->getArgument(index + forOp.getNumInductionVars());
 787:   builder.setInsertionPointToStart(body);
 788:   newCast = markForward(CastOp::create(builder, loc, arg.getType(), arg));
 789:   arg.setType(newType);
 790:   arg.replaceAllUsesExcept(newCast.getResult(0), newCast.getOperation());
 791:   queue.push(newCast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 793-798

```cpp
 793:   // Insert backward cast before yield
 794:   Value yieldSrc = yield.getOperand(index);
 795:   builder.setInsertionPoint(yield.getOperation());
 796:   newCast = markBackward(CastOp::create(builder, loc, newType, yieldSrc));
 797:   yield->setOperand(index, newCast.getResult(0));
 798:   queue.push(newCast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 800-806

```cpp
 800:   // Insert forward cast after forOp
 801:   Value result = forOp.getResult(index);
 802:   builder.setInsertionPointAfter(forOp.getOperation());
 803:   newCast = markForward(CastOp::create(builder, loc, result.getType(), result));
 804:   result.setType(newType);
 805:   result.replaceAllUsesExcept(newCast.getResult(0), newCast.getOperation());
 806:   queue.push(newCast);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 808-809

```cpp
 808:   return true;
 809: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 811-817

```cpp
 811: int findResultIndex(Operation *op, Value result) {
 812:   for (int i = 0; i < op->getNumResults(); ++i)
 813:     if (op->getResult(i) == result)
 814:       return i;
 815:   llvm::report_fatal_error("Invalid index of op result");
 816:   return -1;
 817: }
```

- **EN:** Defines `findResultIndex`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `findResultIndex`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 819-823

```cpp
 819: bool CTAPlanner::processIfOpBackward(scf::IfOp ifOp, CastOp cast) {
 820:   int index = findResultIndex(ifOp.getOperation(), cast.getOperand(0));
 821:   auto newType = cast.getResult(0).getType();
 822:   return processIfOp(ifOp, index, newType);
 823: }
```

- **EN:** Defines `CTAPlanner::processIfOpBackward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processIfOpBackward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 825-829

```cpp
 825: bool CTAPlanner::processForOpBackward(scf::ForOp forOp, CastOp cast) {
 826:   int index = findResultIndex(forOp.getOperation(), cast.getOperand(0));
 827:   auto newType = cast.getResult(0).getType();
 828:   return processForOp(forOp, index, newType);
 829: }
```

- **EN:** Defines `CTAPlanner::processForOpBackward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processForOpBackward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 831-840

```cpp
 831: bool CTAPlanner::processBlockArgBackward(BlockArgument arg, CastOp cast) {
 832:   if (auto forOp = llvm::dyn_cast<scf::ForOp>(arg.getOwner()->getParentOp())) {
 833:     int index = int(arg.getArgNumber()) - forOp.getNumInductionVars();
 834:     auto newType = cast.getResult(0).getType();
 835:     return processForOp(forOp, index, newType);
 836:   } else {
 837:     llvm::report_fatal_error("Unexpected parent op of block argument");
 838:     return true;
 839:   }
 840: }
```

- **EN:** Defines `CTAPlanner::processBlockArgBackward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processBlockArgBackward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 842-847

```cpp
 842: bool CTAPlanner::processForOpForward(scf::ForOp forOp, CastOp cast) {
 843:   int index = cast.getResult(0).use_begin()->getOperandNumber() -
 844:               forOp.getNumControlOperands();
 845:   auto newType = cast.getOperand(0).getType();
 846:   return processForOp(forOp, index, newType);
 847: }
```

- **EN:** Defines `CTAPlanner::processForOpForward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processForOpForward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 849-859

```cpp
 849: bool CTAPlanner::processYieldOpForward(scf::YieldOp yieldOp, CastOp cast) {
 850:   int index = cast.getResult(0).use_begin()->getOperandNumber();
 851:   auto newType = cast.getOperand(0).getType();
 852:   if (auto ifOp = llvm::dyn_cast<scf::IfOp>(yieldOp->getParentOp()))
 853:     return processIfOp(ifOp, index, newType);
 854:   else if (auto forOp = llvm::dyn_cast<scf::ForOp>(yieldOp->getParentOp()))
 855:     return processForOp(forOp, index, newType);
 856:   else
 857:     llvm::report_fatal_error("Unexpected parent op of YieldOp");
 858:   return true;
 859: }
```

- **EN:** Defines `CTAPlanner::processYieldOpForward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processYieldOpForward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 861-863

```cpp
 861: bool CTAPlanner::processOpFallback(Operation *op) {
 862:   Location loc = op->getLoc();
 863:   OpBuilder builder(op->getContext());
```

- **EN:** Defines `CTAPlanner::processOpFallback`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processOpFallback`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 865-875

```cpp
 865:   builder.setInsertionPoint(op);
 866:   for (unsigned i = 0; i < op->getNumOperands(); ++i) {
 867:     Value operand = op->getOperand(i);
 868:     auto operandTy = operand.getType();
 869:     if (isa<RankedTensorType>(operandTy)) {
 870:       auto cast =
 871:           markBackward(CastOp::create(builder, loc, operandTy, operand));
 872:       op->setOperand(i, cast.getResult(0));
 873:       queue.push(cast);
 874:     }
 875:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 877-886

```cpp
 877:   builder.setInsertionPointAfter(op);
 878:   for (unsigned i = 0; i < op->getNumResults(); ++i) {
 879:     Value result = op->getResult(i);
 880:     auto resultTy = result.getType();
 881:     if (isa<RankedTensorType>(resultTy)) {
 882:       auto cast = markForward(CastOp::create(builder, loc, resultTy, result));
 883:       result.replaceAllUsesExcept(cast.getResult(0), cast.getOperation());
 884:       queue.push(cast);
 885:     }
 886:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 888-889

```cpp
 888:   return true;
 889: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 891-893

```cpp
 891: bool CTAPlanner::processMultiUsersBackward(Value input, CastOp cast) {
 892:   Location loc = input.getLoc();
 893:   OpBuilder builder(input.getContext());
```

- **EN:** Defines `CTAPlanner::processMultiUsersBackward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processMultiUsersBackward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 895-912

```cpp
 895:   llvm::DenseMap<Type, llvm::SmallVector<CastOp>> typeToIndices;
 896:   for (OpOperand &operand : input.getUses()) {
 897:     auto brotherCast = llvm::dyn_cast<CastOp>(operand.getOwner());
 898:     if (!brotherCast) {
 899:       if (stepUnchanged <= queue.size())
 900:         return false;
 901:       builder.setInsertionPoint(operand.getOwner());
 902:       brotherCast = markBackward(
 903:           CastOp::create(builder, loc, cast.getResult(0).getType(), input));
 904:       auto newCast = markForward(CastOp::create(builder, loc, input.getType(),
 905:                                                 brotherCast.getResult(0)));
 906:       operand.set(newCast.getResult(0));
 907:       queue.push(brotherCast);
 908:       queue.push(newCast);
 909:     }
 910:     auto type = brotherCast.getResult(0).getType();
 911:     typeToIndices[type].push_back(brotherCast);
 912:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 914-931

```cpp
 914:   bool first = true;
 915:   for (auto it : typeToIndices) {
 916:     Type &type = it.first;
 917:     llvm::SmallVector<CastOp> &casts = it.second;
 918:     Value newInput = input;
 919:     if (!first) {
 920:       if (Operation *defOp = input.getDefiningOp()) {
 921:         builder.setInsertionPointAfter(defOp);
 922:         Operation *clonedOp = builder.clone(*defOp);
 923:         newInput = clonedOp->getResult(0);
 924:       } else {
 925:         llvm::report_fatal_error("Layout conflict for block arg"); // TODO
 926:         return false;
 927:       }
 928:     }
 929:     first = false;
 930:     if (Operation *defOp = newInput.getDefiningOp()) {
 931:       builder.setInsertionPointAfter(defOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 932-947

```cpp
 932:     } else {
 933:       assert(isa<BlockArgument>(newInput) &&
 934:              "Unexpected Value without defining op");
 935:       builder.setInsertionPointToStart(
 936:           llvm::cast<BlockArgument>(newInput).getOwner());
 937:     }
 938:     auto newCast = markBackward(CastOp::create(builder, loc, type, newInput));
 939:     queue.push(newCast);
 940:     auto newResult = newCast.getResult(0);
 941:     for (CastOp &brotherCast : casts) {
 942:       brotherCast.getResult(0).replaceAllUsesWith(newResult);
 943:       eraseCastOpFromQueue(brotherCast);
 944:     }
 945:   }
 946:   return true;
 947: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 949-950

```cpp
 949: bool CTAPlanner::processMultiUsersForward(Value castResult, CastOp cast) {
 950:   Value castSrc = cast.getOperand(0);
```

- **EN:** Defines `CTAPlanner::processMultiUsersForward`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `CTAPlanner::processMultiUsersForward`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 952-954

```cpp
 952:   Location loc = cast.getLoc();
 953:   OpBuilder builder(cast.getContext());
 954:   builder.setInsertionPointAfter(cast.getOperation());
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 956-961

```cpp
 956:   while (!castResult.use_empty()) {
 957:     auto newCast = markForward(
 958:         CastOp::create(builder, loc, castResult.getType(), castSrc));
 959:     castResult.use_begin()->set(newCast.getResult(0));
 960:     queue.push(newCast);
 961:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 963-965

```cpp
 963:   eraseCastOp(cast);
 964:   return true;
 965: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 967-967

```cpp
 967: } // anonymous namespace
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 969-971

```cpp
 969: struct PlanCTAPass : public impl::TritonGPUPlanCTAPassBase<PlanCTAPass> {
 970:   void runOnOperation() override {
 971:     ModuleOp mod = getOperation();
```

- **EN:** Defines `PlanCTAPass`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `PlanCTAPass`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 973-975

```cpp
 973:     // Skip PlanCTAPass when numCTAs == 1
 974:     if (ttg::TritonGPUDialect::getNumCTAs(mod) == 1)
 975:       return;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 977-979

```cpp
 977:     mod.walk([&](triton::FuncOp funcOp) {
 978:       CTAPlanner planner;
 979:       planner.run(funcOp);
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 981-990

```cpp
 981:       // FIXME: Clone funcOp so that the IR change can be identified after
 982:       // PlanCTAPass. Without this, the change after PlanCTAPass will not be
 983:       // displayed when MLIR_ENABLE_DUMP=1. This is not reasonable and should
 984:       // be fixed later.
 985:       OpBuilder builder(funcOp);
 986:       builder.clone(*funcOp.getOperation());
 987:       funcOp.erase();
 988:     });
 989:   }
 990: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 992-994

```cpp
 992: std::unique_ptr<Pass> createTritonNvidiaGPUPlanCTAPass() {
 993:   return std::make_unique<PlanCTAPass>();
 994: }
```

- **EN:** Defines helper `createTritonNvidiaGPUPlanCTAPass` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createTritonNvidiaGPUPlanCTAPass`，用于计算或构造外围变换所需的中间数据。
### Lines 996-998

```cpp
 996: } // namespace nvidia_gpu
 997: } // namespace triton
 998: } // namespace mlir
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 1000-1010

```cpp
1000: /* TODO
1001:  * - Use ConvertLayoutOp instead of UnrealizedConversionCastOp.
1002:  * - Move PlanCTAPass to the front of CoalescePass.
1003:  * - Design better tiling strategy for DotOp and ReduceOp.
1004:  * - Consider cases where there are more than one DotOps.
1005:  * - Use better data structure for erasing CastOps from queue (linked list?).
1006:  * - Process eliminable CastOps in higher priority.
1007:  * - Fix the clone func bug in PlanCTAPass::runOnOperation.
1008:  * - Add some comments to introduce the overall idea of this pass.
1009:  * - Add some lit tests for this pass.
1010:  */
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around plan cta.
  **CN:** 核心关注点是围绕 Plan CTA 的 pass 驱动变换。
- **EN:** Tensor shape and element-type reasoning is central to the implementation.
  **CN:** 张量形状与元素类型推导是实现中的核心内容。
- **EN:** Encoding attributes describe how distributed GPU data is laid out across threads and warps.
  **CN:** 编码属性描述 GPU 分布式数据在线程与 warp 之间的布局方式。
- **EN:** The code reasons at module scope rather than only on isolated operations.
  **CN:** 代码在模块范围内进行分析，而不只是处理单个操作。
- **EN:** Hardware execution parameters such as warps and threads-per-warp affect legality and performance decisions.
  **CN:** warp 数与每个 warp 的线程数等硬件执行参数会影响合法性和性能决策。
- **EN:** Layout and encoding decisions are first-class because they determine how work and memory are distributed on the GPU.
  **CN:** 布局与编码决策是第一类问题，因为它们决定了 GPU 上工作与内存的分布方式。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/TritonGPU/IR/Dialect.h`, `triton/Dialect/TritonNvidiaGPU/IR/Dialect.h`, `triton/Dialect/TritonNvidiaGPU/Transforms/Passes.h`, `triton/Dialect/TritonNvidiaGPU/Transforms/Passes.h.inc`
- **MLIR headers / MLIR 头文件:** `mlir/Support/LLVM.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/STLExtras.h`, `llvm/Support/ErrorHandling.h`
- **Standard/library headers / 标准或通用库头文件:** `queue`
- **Generated fragments / 生成片段:** `triton/Dialect/TritonNvidiaGPU/Transforms/Passes.h.inc`
- **Primary APIs used / 主要 API:** `ModuleOp`, `RankedTensorType`, `OpBuilder`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
