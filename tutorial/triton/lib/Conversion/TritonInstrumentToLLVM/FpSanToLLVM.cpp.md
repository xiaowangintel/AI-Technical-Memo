# FpSanToLLVM.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Conversion/TritonInstrumentToLLVM/FpSanToLLVM.cpp`
- **Purpose / 作用:** **EN:** Converts Triton instrumentation logic forFP sanitizer into LLVM-level form. **CN:** 把与 FP San To LLVM 相关的 TritonInstrument 逻辑转换为 LLVM 层表示。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
   1: #include "mlir/Conversion/LLVMCommon/Pattern.h"
   2: #include "mlir/IR/TypeUtilities.h"
   3: #include "triton/Conversion/TritonGPUToLLVM/PatternTritonGPUOpToLLVM.h"
   4: #include "triton/Conversion/TritonGPUToLLVM/Utility.h"
   5: #include "triton/Dialect/TritonInstrument/IR/Dialect.h"
   6: #include "llvm/ADT/APFloat.h"
   7: #include "llvm/ADT/bit.h"
   8: #include <cassert>
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`PatternTritonGPUOpToLLVM.h`, `Utility.h`, `Dialect.h`) provide domain-specific IR/support, MLIR headers (`Pattern.h`, `TypeUtilities.h`) provide rewriting and analysis infrastructure, LLVM headers (`APFloat.h`, `bit.h`) supply low-level utilities, and standard/library headers (`cassert`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`PatternTritonGPUOpToLLVM.h`, `Utility.h`, `Dialect.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`Pattern.h`, `TypeUtilities.h`）提供重写与分析基础设施，LLVM 头文件（`APFloat.h`, `bit.h`）提供底层工具，而标准/通用库头文件（`cassert`）提供通用能力。
### Lines 10-11

```cpp
  10: using namespace mlir;
  11: using namespace mlir::triton;
```

- **EN:** Introduces namespace aliases/imports (`mlir`, `mlir::triton`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`, `mlir::triton`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 13-13

```cpp
  13: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 15-15

```cpp
  15: namespace tti = mlir::triton::instrument;
```

- **EN:** Opens or closes the namespace nesting for tti, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 tti 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 17-23

```cpp
  17: uint64_t invOddU64(uint64_t a) {
  18:   assert((a & 1) == 1);
  19:   uint64_t x = 2 - a;
  20:   for (unsigned correctBits = 2; correctBits < 64; correctBits *= 2)
  21:     x *= 2 - a * x;
  22:   return x;
  23: }
```

- **EN:** Defines `invOddU64`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `invOddU64`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 25-31

```cpp
  25: uint64_t getOneBitPattern(FloatType floatTy) {
  26:   llvm::APFloat one(1.0);
  27:   bool losesInfo = false;
  28:   one.convert(floatTy.getFloatSemantics(), llvm::APFloat::rmNearestTiesToEven,
  29:               &losesInfo);
  30:   return one.bitcastToAPInt().getZExtValue();
  31: }
```

- **EN:** Defines accessor/helper `getOneBitPattern` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getOneBitPattern`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 33-44

```cpp
  33: struct PayloadMixConfig {
  34:   unsigned bitWidth;
  35:   unsigned shift;
  36:   uint64_t signMask;
  37:   uint64_t magMask;
  38:   uint64_t mulA;
  39:   uint64_t mulAInv;
  40:   uint64_t mulBPos;
  41:   uint64_t mulBNeg;
  42:   uint64_t mulBPosInv;
  43:   uint64_t mulBNegInv;
  44: };
```

- **EN:** Defines `PayloadMixConfig`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `PayloadMixConfig`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 46-50

```cpp
  46: PayloadMixConfig getPayloadMixConfig(FloatType floatTy) {
  47:   unsigned bitWidth = floatTy.getWidth();
  48:   assert(bitWidth > 1 && bitWidth <= 64);
  49:   uint64_t signMask = uint64_t{1} << (bitWidth - 1);
  50:   uint64_t magMask = signMask - 1;
```

- **EN:** Defines accessor/helper `getPayloadMixConfig` that exposes or updates operation state in a compact, reusable way. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义访问器/辅助函数 `getPayloadMixConfig`，以紧凑且可复用的方式读取或更新操作状态。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 52-55

```cpp
  52:   uint64_t oneBits = getOneBitPattern(floatTy);
  53:   assert(oneBits != 0 && "expected non-zero 1.0 bit pattern");
  54:   unsigned shift = llvm::countr_zero(oneBits);
  55:   assert(shift != 0 && "expected even 1.0 bit pattern");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 57-62

```cpp
  57:   // we firstly multiply by an arbitrary odd constant to mix from low
  58:   // bits to high whilst remaining invertible:
  59:   uint64_t mulA = 922291u & magMask;
  60:   uint64_t oneMixed = (oneBits * mulA) & magMask;
  61:   oneMixed ^= oneMixed >> shift;
  62:   assert((oneMixed & 1) == 1 && "expected odd mixed 1.0");
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 64-80

```cpp
  64:   // the second multiplier is chosen so that the entire payload mixing
  65:   // operation maps the float 1.0 to the integer 1:
  66:   uint64_t mulBPos = invOddU64(oneMixed) & magMask;
  67:   uint64_t mulBNeg = (mulBPos * magMask) & magMask;
  68:   return PayloadMixConfig{
  69:       bitWidth,
  70:       shift,
  71:       signMask,
  72:       magMask,
  73:       mulA,
  74:       invOddU64(mulA) & magMask,
  75:       mulBPos,
  76:       mulBNeg,
  77:       invOddU64(mulBPos) & magMask,
  78:       invOddU64(mulBNeg) & magMask,
  79:   };
  80: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 82-87

```cpp
  82: Value createUIntConstant(ConversionPatternRewriter &rewriter, Location loc,
  83:                          Type intTy, uint64_t value) {
  84:   auto intType = cast<IntegerType>(intTy);
  85:   auto attr = IntegerAttr::get(intType, llvm::APInt(intType.getWidth(), value));
  86:   return LLVM::ConstantOp::create(rewriter, loc, intType, attr);
  87: }
```

- **EN:** Defines helper `createUIntConstant` that computes or constructs intermediate data used by the surrounding transformation.
- **CN:** 这里定义辅助函数 `createUIntConstant`，用于计算或构造外围变换所需的中间数据。
### Lines 89-103

```cpp
  89: Value selectUIntConstantOnSign(ConversionPatternRewriter &rewriter,
  90:                                Location loc, Value signSource,
  91:                                uint64_t signMaskValue,
  92:                                uint64_t nonNegativeValue,
  93:                                uint64_t negativeValue) {
  94:   TritonLLVMOpBuilder b(loc, rewriter);
  95:   auto intTy = signSource.getType();
  96:   Value signMask = createUIntConstant(rewriter, loc, intTy, signMaskValue);
  97:   Value zero = createUIntConstant(rewriter, loc, intTy, 0u);
  98:   Value sign = b.and_(signSource, signMask);
  99:   Value isNeg = b.icmp_ne(sign, zero);
 100:   Value nonNeg = createUIntConstant(rewriter, loc, intTy, nonNegativeValue);
 101:   Value neg = createUIntConstant(rewriter, loc, intTy, negativeValue);
 102:   return b.select(isNeg, neg, nonNeg);
 103: }
```

- **EN:** Defines `selectUIntConstantOnSign`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `selectUIntConstantOnSign`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 105-111

```cpp
 105: Value xorShiftRight(ConversionPatternRewriter &rewriter, Location loc, Value v,
 106:                     unsigned shift) {
 107:   TritonLLVMOpBuilder b(loc, rewriter);
 108:   Value shiftValue = createUIntConstant(rewriter, loc, v.getType(), shift);
 109:   Value shifted = b.lshr(v, shiftValue);
 110:   return b.xor_(v, shifted);
 111: }
```

- **EN:** Defines `xorShiftRight`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `xorShiftRight`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 113-118

```cpp
 113: Value inverseXorShiftRight(ConversionPatternRewriter &rewriter, Location loc,
 114:                            Value v, const PayloadMixConfig &cfg) {
 115:   for (unsigned shift = cfg.shift; shift < cfg.bitWidth; shift *= 2)
 116:     v = xorShiftRight(rewriter, loc, v, shift);
 117:   return v;
 118: }
```

- **EN:** Defines `inverseXorShiftRight`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `inverseXorShiftRight`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 120-137

```cpp
 120: Value mixFloatToInt(ConversionPatternRewriter &rewriter, Location loc, Value u,
 121:                     FloatType floatTy) {
 122:   TritonLLVMOpBuilder b(loc, rewriter);
 123:   PayloadMixConfig cfg = getPayloadMixConfig(floatTy);
 124:   Value signFlip =
 125:       selectUIntConstantOnSign(rewriter, loc, u, cfg.signMask, 0, cfg.signMask);
 126:   Value x = b.xor_(u, signFlip);
 127:   Value mulA = createUIntConstant(rewriter, loc, u.getType(), cfg.mulA);
 128:   Value magMask = createUIntConstant(rewriter, loc, u.getType(), cfg.magMask);
 129:   Value yMul = b.mul(x, mulA);
 130:   Value y = b.and_(yMul, magMask);
 131:   Value z = xorShiftRight(rewriter, loc, y, cfg.shift);
 132:   Value mulB = selectUIntConstantOnSign(rewriter, loc, u, cfg.signMask,
 133:                                         cfg.mulBPos, cfg.mulBNeg);
 134:   Value wMul = b.mul(z, mulB);
 135:   Value w = b.and_(wMul, magMask);
 136:   return b.xor_(w, signFlip);
 137: }
```

- **EN:** Defines `mixFloatToInt`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mixFloatToInt`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 139-156

```cpp
 139: Value unmixIntToFloat(ConversionPatternRewriter &rewriter, Location loc,
 140:                       Value v, FloatType floatTy) {
 141:   TritonLLVMOpBuilder b(loc, rewriter);
 142:   PayloadMixConfig cfg = getPayloadMixConfig(floatTy);
 143:   Value signFlip =
 144:       selectUIntConstantOnSign(rewriter, loc, v, cfg.signMask, 0, cfg.signMask);
 145:   Value w = b.xor_(v, signFlip);
 146:   Value magMask = createUIntConstant(rewriter, loc, v.getType(), cfg.magMask);
 147:   Value mulBInv = selectUIntConstantOnSign(rewriter, loc, v, cfg.signMask,
 148:                                            cfg.mulBPosInv, cfg.mulBNegInv);
 149:   Value zMul = b.mul(w, mulBInv);
 150:   Value z = b.and_(zMul, magMask);
 151:   Value y = inverseXorShiftRight(rewriter, loc, z, cfg);
 152:   Value mulAInv = createUIntConstant(rewriter, loc, v.getType(), cfg.mulAInv);
 153:   Value xMul = b.mul(y, mulAInv);
 154:   Value x = b.and_(xMul, magMask);
 155:   return b.xor_(x, signFlip);
 156: }
```

- **EN:** Defines `unmixIntToFloat`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `unmixIntToFloat`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 158-164

```cpp
 158: Value bitcastIfNeeded(ConversionPatternRewriter &rewriter, Location loc,
 159:                       Value value, Type dstTy) {
 160:   if (value.getType() == dstTy)
 161:     return value;
 162:   TritonLLVMOpBuilder b(loc, rewriter);
 163:   return b.bitcast(value, dstTy);
 164: }
```

- **EN:** Defines `bitcastIfNeeded`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `bitcastIfNeeded`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 166-168

```cpp
 166: struct ExperimentalFPSanEmbedOpConversion
 167:     : public ConvertOpToLLVMPattern<tti::ExperimentalFPSanEmbedOp> {
 168:   using ConvertOpToLLVMPattern::ConvertOpToLLVMPattern;
```

- **EN:** Defines `ExperimentalFPSanEmbedOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ExperimentalFPSanEmbedOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 170-175

```cpp
 170:   LogicalResult
 171:   matchAndRewrite(tti::ExperimentalFPSanEmbedOp op, OpAdaptor adaptor,
 172:                   ConversionPatternRewriter &rewriter) const override {
 173:     auto loc = op.getLoc();
 174:     auto floatTy = cast<FloatType>(getElementTypeOrSelf(op.getVal().getType()));
 175:     Type intTy = rewriter.getIntegerType(floatTy.getWidth());
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。
### Lines 177-181

```cpp
 177:     SmallVector<Value> resultVals;
 178:     for (Value elem : unpackLLElements(loc, adaptor.getVal(), rewriter)) {
 179:       Value raw = bitcastIfNeeded(rewriter, loc, elem, intTy);
 180:       resultVals.push_back(mixFloatToInt(rewriter, loc, raw, floatTy));
 181:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 183-188

```cpp
 183:     Value result = packLLElements(loc, getTypeConverter(), resultVals, rewriter,
 184:                                   op.getType());
 185:     rewriter.replaceOp(op, result);
 186:     return success();
 187:   }
 188: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 190-192

```cpp
 190: struct ExperimentalFPSanUnembedOpConversion
 191:     : public ConvertOpToLLVMPattern<tti::ExperimentalFPSanUnembedOp> {
 192:   using ConvertOpToLLVMPattern::ConvertOpToLLVMPattern;
```

- **EN:** Defines `ExperimentalFPSanUnembedOpConversion`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `ExperimentalFPSanUnembedOpConversion`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 194-199

```cpp
 194:   LogicalResult
 195:   matchAndRewrite(tti::ExperimentalFPSanUnembedOp op, OpAdaptor adaptor,
 196:                   ConversionPatternRewriter &rewriter) const override {
 197:     auto loc = op.getLoc();
 198:     auto floatTy = cast<FloatType>(getElementTypeOrSelf(op.getType()));
 199:     Type resultElemTy = getTypeConverter()->convertType(floatTy);
```

- **EN:** Defines `matchAndRewrite`, the core MLIR rewrite callback. It checks whether the source operation matches the pattern and then materializes the lowered replacement. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这里定义 `matchAndRewrite`，即 MLIR 模式重写的核心回调。它先检查源操作是否匹配模式，再构造降级后的替代结果。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 201-205

```cpp
 201:     SmallVector<Value> resultVals;
 202:     for (Value elem : unpackLLElements(loc, adaptor.getVal(), rewriter)) {
 203:       Value raw = unmixIntToFloat(rewriter, loc, elem, floatTy);
 204:       resultVals.push_back(bitcastIfNeeded(rewriter, loc, raw, resultElemTy));
 205:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 207-212

```cpp
 207:     Value result = packLLElements(loc, getTypeConverter(), resultVals, rewriter,
 208:                                   op.getType());
 209:     rewriter.replaceOp(op, result);
 210:     return success();
 211:   }
 212: };
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. It finishes by replacing the original operation with the lowered form. Type mapping is delegated to the LLVM type converter or related conversion helpers.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 它会在转换成功后用降级后的形式替换原始操作。 类型映射委托给 LLVM 类型转换器或相关转换辅助函数。
### Lines 214-214

```cpp
 214: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 216-220

```cpp
 216: void mlir::triton::populateFpSanToLLVMPatterns(LLVMTypeConverter &typeConverter,
 217:                                                RewritePatternSet &patterns) {
 218:   patterns.add<ExperimentalFPSanEmbedOpConversion,
 219:                ExperimentalFPSanUnembedOpConversion>(typeConverter);
 220: }
```

- **EN:** Defines `mlir::triton::populateFpSanToLLVMPatterns`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `mlir::triton::populateFpSanToLLVMPatterns`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。

## Key Concepts / 关键概念
- **EN:** The file focuses on lowering fp san to llvm related IR into a more target-oriented representation.
  **CN:** 本文件重点是把与 FP San To LLVM 相关的 IR 降级为更面向目标的表示。
- **EN:** MLIR dialect conversion patterns drive per-op lowering to LLVM-compatible IR.
  **CN:** MLIR 方言转换模式负责按操作粒度把 IR 降级为 LLVM 兼容表示。
- **EN:** Type conversion bridges Triton/MLIR types to LLVM-level data layouts.
  **CN:** 类型转换负责把 Triton/MLIR 类型映射到 LLVM 层的数据布局。
- **EN:** Pattern rewriting is the main mechanism for canonicalization and lowering in this file.
  **CN:** 模式重写是本文件进行规范化与降级转换的核心机制。
- **EN:** Conversion rewrites replace source dialect ops with target dialect values while preserving semantics.
  **CN:** 转换式重写通过替换源方言操作来保持语义并生成目标方言值。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Conversion/TritonGPUToLLVM/PatternTritonGPUOpToLLVM.h`, `triton/Conversion/TritonGPUToLLVM/Utility.h`, `triton/Dialect/TritonInstrument/IR/Dialect.h`
- **MLIR headers / MLIR 头文件:** `mlir/Conversion/LLVMCommon/Pattern.h`, `mlir/IR/TypeUtilities.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/APFloat.h`, `llvm/ADT/bit.h`
- **Standard/library headers / 标准或通用库头文件:** `cassert`
- **Primary APIs used / 主要 API:** `PatternRewriter`, `ConversionPatternRewriter`, `LLVMTypeConverter`, `OpBuilder`
- **Lowering role / 降级角色:** The implementation depends on source-dialect semantics being valid and emits forms expected by later LLVM or GPU stages. / 该实现依赖源方言语义已经正确，并输出供后续 LLVM 或 GPU 阶段使用的形式。
