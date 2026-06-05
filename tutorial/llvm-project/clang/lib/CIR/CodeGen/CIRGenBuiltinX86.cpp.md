# CIRGenBuiltinX86.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `clang/lib/CIR/CodeGen/CIRGenBuiltinX86.cpp`
- **Repository**: `/root/xw/llvm-project` (`llvm-project`)
- **Purpose (EN)**: This contains code to emit x86/x86_64 Builtin calls as CIR or a function call to be later resolved.
- **Purpose (CN)**: 实现与 `CIRGenBuiltinX86` 相关的 CIR 代码生成支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
   1: //===----------------------------------------------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This contains code to emit x86/x86_64 Builtin calls as CIR or a function
  10: // call to be later resolved.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 14-33
```cpp
  14: #include "CIRGenBuilder.h"
  15: #include "CIRGenFunction.h"
  16: #include "CIRGenModule.h"
  17: #include "mlir/IR/Attributes.h"
  18: #include "mlir/IR/BuiltinAttributes.h"
  19: #include "mlir/IR/Location.h"
  20: #include "mlir/IR/Types.h"
  21: #include "mlir/IR/ValueRange.h"
  22: #include "clang/Basic/Builtins.h"
  23: #include "clang/Basic/TargetBuiltins.h"
  24: #include "clang/CIR/Dialect/IR/CIRAttrs.h"
  25: #include "clang/CIR/Dialect/IR/CIRTypes.h"
  26: #include "clang/CIR/MissingFeatures.h"
  27: #include "llvm/ADT/Sequence.h"
  28: #include "llvm/Support/ErrorHandling.h"
  29: #include <string>
  30: 
  31: using namespace clang;
  32: using namespace clang::CIRGen;
  33: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `CIRGenBuilder.h`, `CIRGenFunction.h`, `CIRGenModule.h`, `Attributes.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `CIRGenBuilder.h`, `CIRGenFunction.h`, `CIRGenModule.h`, `Attributes.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 34-43
```cpp
  34: // OG has unordered comparison as a form of optimization in addition to
  35: // ordered comparison, while CIR doesn't.
  36: //
  37: // This means that we can't encode the comparison code of UGT (unordered
  38: // greater than), at least not at the CIR level.
  39: //
  40: // The boolean shouldInvert compensates for this.
  41: // For example: to get to the comparison code UGT, we pass in
  42: // emitVectorFCmp (OLE, shouldInvert = true) since OLE is the inverse of UGT.
  43: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 44-61
```cpp
  44: // There are several ways to support this otherwise:
  45: // - register extra CmpOpKind for unordered comparison types and build the
  46: // translation code for
  47: //    to go from CIR -> LLVM dialect. Notice we get this naturally with
  48: //    shouldInvert, benefiting from existing infrastructure, albeit having to
  49: //    generate an extra `not` at CIR).
  50: // - Just add extra comparison code to a new VecCmpOpKind instead of
  51: // cluttering CmpOpKind.
  52: // - Add a boolean in VecCmpOp to indicate if it's doing unordered or ordered
  53: // comparison
  54: // - Just emit the intrinsics call instead of calling this helper, see how the
  55: // LLVM lowering handles this.
  56: static mlir::Value emitVectorFCmp(CIRGenFunction &cgf, const CallExpr &expr,
  57:                                   llvm::SmallVector<mlir::Value> &ops,
  58:                                   cir::CmpOpKind pred, bool shouldInvert) {
  59:   CIRGenFunction::CIRGenFPOptionsRAII FPOptsRAII(cgf, &expr);
  60:   // TODO(cir): Add isSignaling boolean once emitConstrainedFPCall implemented
  61:   assert(!cir::MissingFeatures::emitConstrainedFPCall());
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitVectorFCmp`, `FPOptsRAII`, `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitVectorFCmp`、`FPOptsRAII`、`assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 62-69
```cpp
  62:   clang::CIRGen::CIRGenBuilderTy &builder = cgf.getBuilder();
  63:   mlir::Value cmp = builder.createVecCompare(cgf.getLoc(expr.getExprLoc()),
  64:                                              pred, ops[0], ops[1]);
  65:   mlir::Value bitCast = builder.createBitcast(
  66:       shouldInvert ? builder.createNot(cmp) : cmp, ops[0].getType());
  67:   return bitCast;
  68: }
  69: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 70-75
```cpp
  70: static mlir::Value getMaskVecValue(CIRGenBuilderTy &builder, mlir::Location loc,
  71:                                    mlir::Value mask, unsigned numElems) {
  72:   auto maskTy = cir::VectorType::get(
  73:       builder.getSIntNTy(1), cast<cir::IntType>(mask.getType()).getWidth());
  74:   mlir::Value maskVec = builder.createBitcast(mask, maskTy);
  75: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getMaskVecValue`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getMaskVecValue`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 76-83
```cpp
  76:   // If we have less than 8 elements, then the starting mask was an i8 and
  77:   // we need to extract down to the right number of elements.
  78:   if (numElems < 8) {
  79:     SmallVector<mlir::Attribute, 4> indices;
  80:     mlir::Type i32Ty = builder.getSInt32Ty();
  81:     for (auto i : llvm::seq<unsigned>(0, numElems))
  82:       indices.push_back(cir::IntAttr::get(i32Ty, i));
  83: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 84-88
```cpp
  84:     maskVec = builder.createVecShuffle(loc, maskVec, maskVec, indices);
  85:   }
  86:   return maskVec;
  87: }
  88: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 89-111
```cpp
  89: // Builds the VecShuffleOp for pshuflw and pshufhw x86 builtins.
  90: //
  91: // The vector is split into lanes of 8 word elements (16 bits). The lower or
  92: // upper half of each lane, controlled by `isLow`, is shuffled in the following
  93: // way: The immediate is truncated to 8 bits, separated into 4 2-bit fields. The
  94: // i-th field's value represents the resulting index of the i-th element in the
  95: // half lane after shuffling. The other half of the lane remains unchanged.
  96: static cir::VecShuffleOp emitPshufWord(CIRGenBuilderTy &builder,
  97:                                        const mlir::Value vec,
  98:                                        const mlir::Value immediate,
  99:                                        const mlir::Location loc,
 100:                                        const bool isLow) {
 101:   uint32_t imm = CIRGenFunction::getZExtIntValueFromConstOp(immediate);
 102: 
 103:   auto vecTy = cast<cir::VectorType>(vec.getType());
 104:   unsigned numElts = vecTy.getSize();
 105: 
 106:   unsigned firstHalfStart = isLow ? 0 : 4;
 107:   unsigned secondHalfStart = 4 - firstHalfStart;
 108: 
 109:   // Splat the 8-bits of immediate 4 times to help the loop wrap around.
 110:   imm = (imm & 0xff) * 0x01010101;
 111: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitPshufWord`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitPshufWord`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 112-124
```cpp
 112:   int64_t indices[32];
 113:   for (unsigned l = 0; l != numElts; l += 8) {
 114:     for (unsigned i = firstHalfStart; i != firstHalfStart + 4; ++i) {
 115:       indices[l + i] = l + (imm & 3) + firstHalfStart;
 116:       imm >>= 2;
 117:     }
 118:     for (unsigned i = secondHalfStart; i != secondHalfStart + 4; ++i)
 119:       indices[l + i] = l + i;
 120:   }
 121: 
 122:   return builder.createVecShuffle(loc, vec, ArrayRef(indices, numElts));
 123: }
 124: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 125-138
```cpp
 125: // Builds the shuffle mask for pshufd and shufpd/shufps x86 builtins.
 126: // The shuffle mask is written to outIndices.
 127: static void
 128: computeFullLaneShuffleMask(CIRGenFunction &cgf, const mlir::Value vec,
 129:                            uint32_t imm, const bool isShufP,
 130:                            llvm::SmallVectorImpl<int64_t> &outIndices) {
 131:   auto vecTy = cast<cir::VectorType>(vec.getType());
 132:   unsigned numElts = vecTy.getSize();
 133:   unsigned numLanes = cgf.cgm.getDataLayout().getTypeSizeInBits(vecTy) / 128;
 134:   unsigned numLaneElts = numElts / numLanes;
 135: 
 136:   // Splat the 8-bits of immediate 4 times to help the loop wrap around.
 137:   imm = (imm & 0xff) * 0x01010101;
 138: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `computeFullLaneShuffleMask`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `computeFullLaneShuffleMask`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 139-151
```cpp
 139:   for (unsigned l = 0; l != numElts; l += numLaneElts) {
 140:     for (unsigned i = 0; i != numLaneElts; ++i) {
 141:       uint32_t idx = imm % numLaneElts;
 142:       imm /= numLaneElts;
 143:       if (isShufP && i >= (numLaneElts / 2))
 144:         idx += numElts;
 145:       outIndices[l + i] = l + idx;
 146:     }
 147:   }
 148: 
 149:   outIndices.resize(numElts);
 150: }
 151: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 152-164
```cpp
 152: static mlir::Value emitPrefetch(CIRGenFunction &cgf, unsigned builtinID,
 153:                                 const CallExpr *e,
 154:                                 const SmallVector<mlir::Value> &ops) {
 155:   CIRGenBuilderTy &builder = cgf.getBuilder();
 156:   mlir::Location location = cgf.getLoc(e->getExprLoc());
 157:   mlir::Type voidTy = builder.getVoidTy();
 158:   mlir::Value address = builder.createPtrBitcast(ops[0], voidTy);
 159:   bool isWrite{};
 160:   int locality{};
 161: 
 162:   assert(builtinID == X86::BI_mm_prefetch || builtinID == X86::BI_m_prefetchw ||
 163:          builtinID == X86::BI_m_prefetch && "Expected prefetch builtin");
 164: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitPrefetch`, `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitPrefetch`、`assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 165-173
```cpp
 165:   if (builtinID == X86::BI_mm_prefetch) {
 166:     int hint = cgf.getSExtIntValueFromConstOp(ops[1]);
 167:     isWrite = (hint >> 2) & 0x1;
 168:     locality = hint & 0x3;
 169:   } else {
 170:     isWrite = (builtinID == X86::BI_m_prefetchw);
 171:     locality = 0x3;
 172:   }
 173: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 174-177
```cpp
 174:   cir::PrefetchOp::create(builder, location, address, locality, isWrite);
 175:   return {};
 176: }
 177: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::PrefetchOp::create`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::PrefetchOp::create`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 178-189
```cpp
 178: static mlir::Value emitX86CompressExpand(CIRGenBuilderTy &builder,
 179:                                          mlir::Location loc, mlir::Value source,
 180:                                          mlir::Value mask,
 181:                                          mlir::Value inputVector,
 182:                                          const std::string &id) {
 183:   auto resultTy = cast<cir::VectorType>(mask.getType());
 184:   mlir::Value maskValue = getMaskVecValue(
 185:       builder, loc, inputVector, cast<cir::VectorType>(resultTy).getSize());
 186:   return builder.emitIntrinsicCallOp(loc, id, resultTy,
 187:                                      mlir::ValueRange{source, mask, maskValue});
 188: }
 189: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86CompressExpand`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86CompressExpand`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 190-201
```cpp
 190: static mlir::Value
 191: emitEncodeKey(mlir::MLIRContext *context, CIRGenBuilderTy &builder,
 192:               const mlir::Location &location, mlir::ValueRange inputOperands,
 193:               mlir::Value outputOperand, std::uint8_t vecOutputCount,
 194:               const std::string &intrinsicName, std::uint8_t numResults) {
 195:   cir::VectorType resVector = cir::VectorType::get(builder.getUInt64Ty(), 2);
 196:   llvm::SmallVector<mlir::Type> members{builder.getUInt32Ty()};
 197:   llvm::append_range(members,
 198:                      llvm::SmallVector<mlir::Type>(vecOutputCount, resVector));
 199:   cir::RecordType resRecord = cir::RecordType::get(
 200:       context, members, false, false, cir::RecordType::RecordKind::Struct);
 201: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitEncodeKey`, `llvm::append_range`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitEncodeKey`、`llvm::append_range`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 202-215
```cpp
 202:   mlir::Value outputPtr =
 203:       builder.createBitcast(outputOperand, cir::PointerType::get(resVector));
 204:   mlir::Value call = builder.emitIntrinsicCallOp(location, intrinsicName,
 205:                                                  resRecord, inputOperands);
 206:   for (std::uint8_t i = 0; i < numResults; ++i) {
 207:     mlir::Value vecValue =
 208:         cir::ExtractMemberOp::create(builder, location, call, i + 1);
 209:     mlir::Value index = builder.getSInt32(i, location);
 210:     mlir::Value ptr = builder.createPtrStride(location, outputPtr, index);
 211:     builder.createStore(location, vecValue, Address{ptr, CharUnits::One()});
 212:   }
 213:   return cir::ExtractMemberOp::create(builder, location, call, 0);
 214: }
 215: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::ExtractMemberOp::create`. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::ExtractMemberOp::create`。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 216-229
```cpp
 216: static mlir::Value emitX86Select(CIRGenBuilderTy &builder, mlir::Location loc,
 217:                                  mlir::Value mask, mlir::Value op0,
 218:                                  mlir::Value op1) {
 219:   auto constOp = mlir::dyn_cast_or_null<cir::ConstantOp>(mask.getDefiningOp());
 220:   // If the mask is all ones just return first argument.
 221:   if (constOp && constOp.isAllOnesValue())
 222:     return op0;
 223: 
 224:   mask = getMaskVecValue(builder, loc, mask,
 225:                          cast<cir::VectorType>(op0.getType()).getSize());
 226: 
 227:   return cir::VecTernaryOp::create(builder, loc, mask, op0, op1);
 228: }
 229: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86Select`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86Select`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 230-243
```cpp
 230: // Helper function to extract zero-bit from a mask as a boolean
 231: static mlir::Value getMaskZeroBitAsBool(CIRGenBuilderTy &builder,
 232:                                         mlir::Location loc, mlir::Value mask) {
 233:   // Get the mask as a vector of i1 and extract bit 0
 234:   auto intTy = mlir::dyn_cast<cir::IntType>(mask.getType());
 235:   assert(intTy && "mask must be an integer type");
 236:   unsigned width = intTy.getWidth();
 237: 
 238:   auto maskVecTy = cir::VectorType::get(builder.getSIntNTy(1), width);
 239:   mlir::Value maskVec = builder.createBitcast(mask, maskVecTy);
 240: 
 241:   // Extract bit 0 from the mask vector
 242:   mlir::Value bit0 = builder.createExtractElement(loc, maskVec, uint64_t(0));
 243: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getMaskZeroBitAsBool`, `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getMaskZeroBitAsBool`、`assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 244-249
```cpp
 244:   // Convert i1 to bool for select
 245:   auto boolTy = cir::BoolType::get(builder.getContext());
 246:   return cir::CastOp::create(builder, loc, boolTy, cir::CastKind::int_to_bool,
 247:                              bit0);
 248: }
 249: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 250-253
```cpp
 250: static mlir::Value emitX86ScalarSelect(CIRGenBuilderTy &builder,
 251:                                        mlir::Location loc, mlir::Value mask,
 252:                                        mlir::Value op0, mlir::Value op1) {
 253: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86ScalarSelect`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86ScalarSelect`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 254-258
```cpp
 254:   // If the mask is all ones just return first argument.
 255:   if (auto c = mlir::dyn_cast_or_null<cir::ConstantOp>(mask.getDefiningOp()))
 256:     if (c.isAllOnesValue())
 257:       return op0;
 258: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 259-262
```cpp
 259:   mlir::Value cond = getMaskZeroBitAsBool(builder, loc, mask);
 260:   return builder.createSelect(loc, cond, op0, op1);
 261: }
 262: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 263-267
```cpp
 263: static mlir::Value emitX86MaskAddLogic(CIRGenBuilderTy &builder,
 264:                                        mlir::Location loc,
 265:                                        const std::string &intrinsicName,
 266:                                        SmallVectorImpl<mlir::Value> &ops) {
 267: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskAddLogic`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskAddLogic`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 268-277
```cpp
 268:   auto intTy = cast<cir::IntType>(ops[0].getType());
 269:   unsigned numElts = intTy.getWidth();
 270:   mlir::Value lhsVec = getMaskVecValue(builder, loc, ops[0], numElts);
 271:   mlir::Value rhsVec = getMaskVecValue(builder, loc, ops[1], numElts);
 272:   mlir::Type vecTy = lhsVec.getType();
 273:   mlir::Value resVec = builder.emitIntrinsicCallOp(
 274:       loc, intrinsicName, vecTy, mlir::ValueRange{lhsVec, rhsVec});
 275:   return builder.createBitcast(resVec, ops[0].getType());
 276: }
 277: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 278-283
```cpp
 278: static mlir::Value emitX86MaskUnpack(CIRGenBuilderTy &builder,
 279:                                      mlir::Location loc,
 280:                                      const std::string &intrinsicName,
 281:                                      SmallVectorImpl<mlir::Value> &ops) {
 282:   unsigned numElems = cast<cir::IntType>(ops[0].getType()).getWidth();
 283: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskUnpack`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskUnpack`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 284-289
```cpp
 284:   // Convert both operands to mask vectors.
 285:   mlir::Value lhs = getMaskVecValue(builder, loc, ops[0], numElems);
 286:   mlir::Value rhs = getMaskVecValue(builder, loc, ops[1], numElems);
 287: 
 288:   mlir::Type i32Ty = builder.getSInt32Ty();
 289: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 290-294
```cpp
 290:   // Create indices for extracting the first half of each vector.
 291:   SmallVector<mlir::Attribute, 32> halfIndices;
 292:   for (auto i : llvm::seq<unsigned>(0, numElems / 2))
 293:     halfIndices.push_back(cir::IntAttr::get(i32Ty, i));
 294: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 295-299
```cpp
 295:   // Extract first half of each vector. This gives better codegen than
 296:   // doing it in a single shuffle.
 297:   mlir::Value lhsHalf = builder.createVecShuffle(loc, lhs, lhs, halfIndices);
 298:   mlir::Value rhsHalf = builder.createVecShuffle(loc, rhs, rhs, halfIndices);
 299: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 300-309
```cpp
 300:   // Create indices for concatenating the vectors.
 301:   // NOTE: Operands are swapped to match the intrinsic definition.
 302:   // After the half extraction, both vectors have numElems/2 elements.
 303:   // In createVecShuffle(rhsHalf, lhsHalf, indices), indices [0..numElems/2-1]
 304:   // select from rhsHalf, and indices [numElems/2..numElems-1] select from
 305:   // lhsHalf.
 306:   SmallVector<mlir::Attribute, 64> concatIndices;
 307:   for (auto i : llvm::seq<unsigned>(0, numElems))
 308:     concatIndices.push_back(cir::IntAttr::get(i32Ty, i));
 309: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 310-315
```cpp
 310:   // Concat the vectors (RHS first, then LHS).
 311:   mlir::Value res =
 312:       builder.createVecShuffle(loc, rhsHalf, lhsHalf, concatIndices);
 313:   return builder.createBitcast(res, ops[0].getType());
 314: }
 315: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 316-323
```cpp
 316: template <typename BinOp>
 317: static mlir::Value
 318: emitX86MaskLogic(CIRGenBuilderTy &builder, mlir::Location loc,
 319:                  SmallVectorImpl<mlir::Value> &ops, bool invertLHS = false) {
 320:   unsigned numElts = cast<cir::IntType>(ops[0].getType()).getWidth();
 321:   mlir::Value lhs = getMaskVecValue(builder, loc, ops[0], numElts);
 322:   mlir::Value rhs = getMaskVecValue(builder, loc, ops[1], numElts);
 323: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskLogic`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskLogic`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 324-329
```cpp
 324:   if (invertLHS)
 325:     lhs = builder.createNot(lhs);
 326:   return builder.createBitcast(BinOp::create(builder, loc, lhs, rhs),
 327:                                ops[0].getType());
 328: }
 329: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 330-341
```cpp
 330: static mlir::Value emitX86MaskTest(CIRGenBuilderTy &builder, mlir::Location loc,
 331:                                    const std::string &intrinsicName,
 332:                                    SmallVectorImpl<mlir::Value> &ops) {
 333:   auto intTy = cast<cir::IntType>(ops[0].getType());
 334:   unsigned numElts = intTy.getWidth();
 335:   mlir::Value lhsVec = getMaskVecValue(builder, loc, ops[0], numElts);
 336:   mlir::Value rhsVec = getMaskVecValue(builder, loc, ops[1], numElts);
 337:   mlir::Type resTy = builder.getSInt32Ty();
 338:   return builder.emitIntrinsicCallOp(loc, intrinsicName, resTy,
 339:                                      mlir::ValueRange{lhsVec, rhsVec});
 340: }
 341: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskTest`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskTest`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 342-355
```cpp
 342: static mlir::Value emitX86MaskedCompareResult(CIRGenBuilderTy &builder,
 343:                                               mlir::Value cmp, unsigned numElts,
 344:                                               mlir::Value maskIn,
 345:                                               mlir::Location loc) {
 346:   if (maskIn) {
 347:     auto c = mlir::dyn_cast_or_null<cir::ConstantOp>(maskIn.getDefiningOp());
 348:     if (!c || !c.isAllOnesValue())
 349:       cmp = builder.createAnd(loc, cmp,
 350:                               getMaskVecValue(builder, loc, maskIn, numElts));
 351:   }
 352:   if (numElts < 8) {
 353:     llvm::SmallVector<mlir::Attribute> indices;
 354:     mlir::Type i64Ty = builder.getSInt64Ty();
 355: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskedCompareResult`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskedCompareResult`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 356-360
```cpp
 356:     for (unsigned i = 0; i != numElts; ++i)
 357:       indices.push_back(cir::IntAttr::get(i64Ty, i));
 358:     for (unsigned i = numElts; i != 8; ++i)
 359:       indices.push_back(cir::IntAttr::get(i64Ty, i % numElts + numElts));
 360: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 361-367
```cpp
 361:     // This should shuffle between cmp (first vector) and null (second vector)
 362:     mlir::Value nullVec = builder.getNullValue(cmp.getType(), loc);
 363:     cmp = builder.createVecShuffle(loc, cmp, nullVec, indices);
 364:   }
 365:   return builder.createBitcast(cmp, builder.getUIntNTy(std::max(numElts, 8U)));
 366: }
 367: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 368-385
```cpp
 368: // TODO: The cgf parameter should be removed when all the NYI cases are
 369: // implemented.
 370: static std::optional<mlir::Value>
 371: emitX86MaskedCompare(CIRGenBuilderTy &builder, unsigned cc, bool isSigned,
 372:                      ArrayRef<mlir::Value> ops, mlir::Location loc) {
 373:   assert((ops.size() == 2 || ops.size() == 4) &&
 374:          "Unexpected number of arguments");
 375:   unsigned numElts = cast<cir::VectorType>(ops[0].getType()).getSize();
 376:   mlir::Value cmp;
 377:   if (cc == 3) {
 378:     cmp = builder.getNullValue(
 379:         cir::VectorType::get(builder.getSIntNTy(1), numElts), loc);
 380:   } else if (cc == 7) {
 381:     cir::VectorType resultTy =
 382:         cir::VectorType::get(builder.getSIntNTy(1), numElts);
 383:     llvm::APInt allOnes = llvm::APInt::getAllOnes(1);
 384:     cmp = cir::VecSplatOp::create(
 385:         builder, loc, resultTy,
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskedCompare`, `assert`, `cir::VectorType::get`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskedCompare`、`assert`、`cir::VectorType::get`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 386-403
```cpp
 386:         builder.getConstAPInt(loc, builder.getSIntNTy(1), allOnes));
 387:   } else {
 388:     cir::CmpOpKind pred;
 389:     switch (cc) {
 390:     default:
 391:       llvm_unreachable("Unknown condition code");
 392:     case 0:
 393:       pred = cir::CmpOpKind::eq;
 394:       break;
 395:     case 1:
 396:       pred = cir::CmpOpKind::lt;
 397:       break;
 398:     case 2:
 399:       pred = cir::CmpOpKind::le;
 400:       break;
 401:     case 4:
 402:       pred = cir::CmpOpKind::ne;
 403:       break;
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 404-411
```cpp
 404:     case 5:
 405:       pred = cir::CmpOpKind::ge;
 406:       break;
 407:     case 6:
 408:       pred = cir::CmpOpKind::gt;
 409:       break;
 410:     }
 411: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 412-415
```cpp
 412:     auto resultTy = cir::VectorType::get(builder.getSIntNTy(1), numElts);
 413:     cmp = cir::VecCmpOp::create(builder, loc, resultTy, pred, ops[0], ops[1]);
 414:   }
 415: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 416-422
```cpp
 416:   mlir::Value maskIn;
 417:   if (ops.size() == 4)
 418:     maskIn = ops[3];
 419: 
 420:   return emitX86MaskedCompareResult(builder, cmp, numElts, maskIn, loc);
 421: }
 422: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 423-432
```cpp
 423: // TODO: The cgf parameter should be removed when all the NYI cases are
 424: // implemented.
 425: static std::optional<mlir::Value> emitX86ConvertToMask(CIRGenFunction &cgf,
 426:                                                        CIRGenBuilderTy &builder,
 427:                                                        mlir::Value in,
 428:                                                        mlir::Location loc) {
 429:   cir::ConstantOp zero = builder.getNullValue(in.getType(), loc);
 430:   return emitX86MaskedCompare(builder, 1, true, {in, zero}, loc);
 431: }
 432: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86ConvertToMask`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86ConvertToMask`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 433-442
```cpp
 433: static std::optional<mlir::Value> emitX86SExtMask(CIRGenBuilderTy &builder,
 434:                                                   mlir::Value op,
 435:                                                   mlir::Type dstTy,
 436:                                                   mlir::Location loc) {
 437:   unsigned numberOfElements = cast<cir::VectorType>(dstTy).getSize();
 438:   mlir::Value mask = getMaskVecValue(builder, loc, op, numberOfElements);
 439: 
 440:   return builder.createCast(loc, cir::CastKind::integral, mask, dstTy);
 441: }
 442: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86SExtMask`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86SExtMask`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 443-457
```cpp
 443: static mlir::Value emitVecInsert(CIRGenBuilderTy &builder, mlir::Location loc,
 444:                                  mlir::Value vec, mlir::Value value,
 445:                                  mlir::Value indexOp) {
 446:   unsigned numElts = cast<cir::VectorType>(vec.getType()).getSize();
 447: 
 448:   uint64_t index =
 449:       indexOp.getDefiningOp<cir::ConstantOp>().getIntValue().getZExtValue();
 450: 
 451:   index &= numElts - 1;
 452: 
 453:   cir::ConstantOp indexVal = builder.getUInt64(index, loc);
 454: 
 455:   return cir::VecInsertOp::create(builder, loc, vec, value, indexVal);
 456: }
 457: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitVecInsert`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitVecInsert`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 458-463
```cpp
 458: static mlir::Value emitX86FunnelShift(CIRGenBuilderTy &builder,
 459:                                       mlir::Location location, mlir::Value &op0,
 460:                                       mlir::Value &op1, mlir::Value &amt,
 461:                                       bool isRight) {
 462:   mlir::Type op0Ty = op0.getType();
 463: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86FunnelShift`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86FunnelShift`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 464-473
```cpp
 464:   // Amount may be scalar immediate, in which case create a splat vector.
 465:   // Funnel shifts amounts are treated as modulo and types are all power-of-2
 466:   // so we only care about the lowest log2 bits anyway.
 467:   if (amt.getType() != op0Ty) {
 468:     auto vecTy = mlir::cast<cir::VectorType>(op0Ty);
 469:     uint64_t numElems = vecTy.getSize();
 470: 
 471:     auto amtTy = mlir::cast<cir::IntType>(amt.getType());
 472:     auto vecElemTy = mlir::cast<cir::IntType>(vecTy.getElementType());
 473: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 474-486
```cpp
 474:     // If signed, cast to the same width but unsigned first to
 475:     // ensure zero-extension when casting to a bigger unsigned `vecElemeTy`.
 476:     if (amtTy.isSigned()) {
 477:       cir::IntType unsignedAmtTy = builder.getUIntNTy(amtTy.getWidth());
 478:       amt = builder.createIntCast(amt, unsignedAmtTy);
 479:     }
 480:     cir::IntType unsignedVecElemType = builder.getUIntNTy(vecElemTy.getWidth());
 481:     amt = builder.createIntCast(amt, unsignedVecElemType);
 482:     amt = cir::VecSplatOp::create(
 483:         builder, location, cir::VectorType::get(unsignedVecElemType, numElems),
 484:         amt);
 485:   }
 486: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::VectorType::get`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::VectorType::get`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 487-491
```cpp
 487:   const StringRef intrinsicName = isRight ? "fshr" : "fshl";
 488:   return builder.emitIntrinsicCallOp(location, intrinsicName, op0Ty,
 489:                                      mlir::ValueRange{op0, op1, amt});
 490: }
 491: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 492-509
```cpp
 492: static mlir::Value emitX86Muldq(CIRGenBuilderTy &builder, mlir::Location loc,
 493:                                 bool isSigned,
 494:                                 SmallVectorImpl<mlir::Value> &ops,
 495:                                 unsigned opTypePrimitiveSizeInBits) {
 496:   mlir::Type ty = cir::VectorType::get(builder.getSInt64Ty(),
 497:                                        opTypePrimitiveSizeInBits / 64);
 498:   mlir::Value lhs = builder.createBitcast(loc, ops[0], ty);
 499:   mlir::Value rhs = builder.createBitcast(loc, ops[1], ty);
 500:   if (isSigned) {
 501:     cir::ConstantOp shiftAmt =
 502:         builder.getConstant(loc, cir::IntAttr::get(builder.getSInt64Ty(), 32));
 503:     cir::VecSplatOp shiftSplatVecOp =
 504:         cir::VecSplatOp::create(builder, loc, ty, shiftAmt.getResult());
 505:     mlir::Value shiftSplatValue = shiftSplatVecOp.getResult();
 506:     // In CIR, right-shift operations are automatically lowered to either an
 507:     // arithmetic or logical shift depending on the operand type. The purpose
 508:     // of the shifts here is to propagate the sign bit of the 32-bit input
 509:     // into the upper bits of each vector lane.
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86Muldq`, `cir::VecSplatOp::create`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86Muldq`、`cir::VecSplatOp::create`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 510-525
```cpp
 510:     lhs = builder.createShift(loc, lhs, shiftSplatValue, true);
 511:     lhs = builder.createShift(loc, lhs, shiftSplatValue, false);
 512:     rhs = builder.createShift(loc, rhs, shiftSplatValue, true);
 513:     rhs = builder.createShift(loc, rhs, shiftSplatValue, false);
 514:   } else {
 515:     cir::ConstantOp maskScalar = builder.getConstant(
 516:         loc, cir::IntAttr::get(builder.getSInt64Ty(), 0xffffffff));
 517:     cir::VecSplatOp mask =
 518:         cir::VecSplatOp::create(builder, loc, ty, maskScalar.getResult());
 519:     // Clear the upper bits
 520:     lhs = builder.createAnd(loc, lhs, mask);
 521:     rhs = builder.createAnd(loc, rhs, mask);
 522:   }
 523:   return builder.createMul(loc, lhs, rhs);
 524: }
 525: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::IntAttr::get`, `cir::VecSplatOp::create`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::IntAttr::get`、`cir::VecSplatOp::create`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 526-533
```cpp
 526: // Convert f16 half values to floats.
 527: static mlir::Value emitX86CvtF16ToFloatExpr(CIRGenBuilderTy &builder,
 528:                                             mlir::Location loc,
 529:                                             llvm::ArrayRef<mlir::Value> ops,
 530:                                             mlir::Type dstTy) {
 531:   assert((ops.size() == 1 || ops.size() == 3 || ops.size() == 4) &&
 532:          "Unknown cvtph2ps intrinsic");
 533: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86CvtF16ToFloatExpr`, `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86CvtF16ToFloatExpr`、`assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 534-546
```cpp
 534:   // If the SAE intrinsic doesn't use default rounding then we can't upgrade.
 535:   if (ops.size() == 4) {
 536:     auto constOp = ops[3].getDefiningOp<cir::ConstantOp>();
 537:     assert(constOp && "Expected constant operand");
 538:     if (constOp.getIntValue().getZExtValue() != 4) {
 539:       return builder.emitIntrinsicCallOp(loc, "x86.avx512.mask.vcvtph2ps.512",
 540:                                          dstTy, ops);
 541:     }
 542:   }
 543: 
 544:   unsigned numElts = cast<cir::VectorType>(dstTy).getSize();
 545:   mlir::Value src = ops[0];
 546: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 547-552
```cpp
 547:   // Extract the subvector
 548:   if (numElts != cast<cir::VectorType>(src.getType()).getSize()) {
 549:     assert(numElts == 4 && "Unexpected vector size");
 550:     src = builder.createVecShuffle(loc, src, {0, 1, 2, 3});
 551:   }
 552: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 553-561
```cpp
 553:   // Bitcast from vXi16 to vXf16.
 554:   cir::VectorType halfTy =
 555:       cir::VectorType::get(cir::FP16Type::get(builder.getContext()), numElts);
 556: 
 557:   src = builder.createCast(cir::CastKind::bitcast, src, halfTy);
 558: 
 559:   // Perform the fp-extension
 560:   mlir::Value res = builder.createCast(cir::CastKind::floating, src, dstTy);
 561: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::VectorType::get`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::VectorType::get`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 562-566
```cpp
 562:   if (ops.size() >= 3)
 563:     res = emitX86Select(builder, loc, ops[2], res, ops[1]);
 564:   return res;
 565: }
 566: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 567-577
```cpp
 567: static mlir::Value emitX86vpcom(CIRGenBuilderTy &builder, mlir::Location loc,
 568:                                 llvm::SmallVector<mlir::Value> ops,
 569:                                 bool isSigned) {
 570:   mlir::Value op0 = ops[0];
 571:   mlir::Value op1 = ops[1];
 572: 
 573:   cir::VectorType ty = cast<cir::VectorType>(op0.getType());
 574:   cir::IntType elementTy = cast<cir::IntType>(ty.getElementType());
 575: 
 576:   uint64_t imm = CIRGenFunction::getZExtIntValueFromConstOp(ops[2]) & 0x7;
 577: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86vpcom`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86vpcom`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 578-595
```cpp
 578:   cir::CmpOpKind pred;
 579:   switch (imm) {
 580:   case 0x0:
 581:     pred = cir::CmpOpKind::lt;
 582:     break;
 583:   case 0x1:
 584:     pred = cir::CmpOpKind::le;
 585:     break;
 586:   case 0x2:
 587:     pred = cir::CmpOpKind::gt;
 588:     break;
 589:   case 0x3:
 590:     pred = cir::CmpOpKind::ge;
 591:     break;
 592:   case 0x4:
 593:     pred = cir::CmpOpKind::eq;
 594:     break;
 595:   case 0x5:
```
- **EN**: This block contains local control-flow decisions or iterative processing. A switch statement is used to dispatch behavior across enumerated cases or kinds. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 596-609
```cpp
 596:     pred = cir::CmpOpKind::ne;
 597:     break;
 598:   case 0x6:
 599:     return builder.getNullValue(ty, loc); // FALSE
 600:   case 0x7: {
 601:     llvm::APInt allOnes = llvm::APInt::getAllOnes(elementTy.getWidth());
 602:     return cir::VecSplatOp::create(
 603:         builder, loc, ty,
 604:         builder.getConstAPInt(loc, elementTy, allOnes)); // TRUE
 605:   }
 606:   default:
 607:     llvm_unreachable("Unexpected XOP vpcom/vpcomu predicate");
 608:   }
 609: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 610-621
```cpp
 610:   if ((!isSigned && elementTy.isSigned()) ||
 611:       (isSigned && elementTy.isUnsigned())) {
 612:     elementTy = elementTy.isSigned() ? builder.getUIntNTy(elementTy.getWidth())
 613:                                      : builder.getSIntNTy(elementTy.getWidth());
 614:     ty = cir::VectorType::get(elementTy, ty.getSize());
 615:     op0 = builder.createBitcast(op0, ty);
 616:     op1 = builder.createBitcast(op1, ty);
 617:   }
 618: 
 619:   return builder.createVecCompare(loc, pred, op0, op1);
 620: }
 621: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 622-628
```cpp
 622: static mlir::Value emitX86Fpclass(CIRGenBuilderTy &builder, mlir::Location loc,
 623:                                   unsigned builtinID,
 624:                                   SmallVectorImpl<mlir::Value> &ops) {
 625:   unsigned numElts = cast<cir::VectorType>(ops[0].getType()).getSize();
 626:   mlir::Value maskIn = ops[2];
 627:   ops.erase(ops.begin() + 2);
 628: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86Fpclass`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86Fpclass`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 629-646
```cpp
 629:   StringRef intrinsicName;
 630:   switch (builtinID) {
 631:   default:
 632:     llvm_unreachable("Unsupported fpclass builtin");
 633:   case X86::BI__builtin_ia32_vfpclassbf16128_mask:
 634:     intrinsicName = "x86.avx10.fpclass.bf16.128";
 635:     break;
 636:   case X86::BI__builtin_ia32_vfpclassbf16256_mask:
 637:     intrinsicName = "x86.avx10.fpclass.bf16.256";
 638:     break;
 639:   case X86::BI__builtin_ia32_vfpclassbf16512_mask:
 640:     intrinsicName = "x86.avx10.fpclass.bf16.512";
 641:     break;
 642:   case X86::BI__builtin_ia32_fpclassph128_mask:
 643:     intrinsicName = "x86.avx512fp16.fpclass.ph.128";
 644:     break;
 645:   case X86::BI__builtin_ia32_fpclassph256_mask:
 646:     intrinsicName = "x86.avx512fp16.fpclass.ph.256";
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 647-664
```cpp
 647:     break;
 648:   case X86::BI__builtin_ia32_fpclassph512_mask:
 649:     intrinsicName = "x86.avx512fp16.fpclass.ph.512";
 650:     break;
 651:   case X86::BI__builtin_ia32_fpclassps128_mask:
 652:     intrinsicName = "x86.avx512.fpclass.ps.128";
 653:     break;
 654:   case X86::BI__builtin_ia32_fpclassps256_mask:
 655:     intrinsicName = "x86.avx512.fpclass.ps.256";
 656:     break;
 657:   case X86::BI__builtin_ia32_fpclassps512_mask:
 658:     intrinsicName = "x86.avx512.fpclass.ps.512";
 659:     break;
 660:   case X86::BI__builtin_ia32_fpclasspd128_mask:
 661:     intrinsicName = "x86.avx512.fpclass.pd.128";
 662:     break;
 663:   case X86::BI__builtin_ia32_fpclasspd256_mask:
 664:     intrinsicName = "x86.avx512.fpclass.pd.256";
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 665-670
```cpp
 665:     break;
 666:   case X86::BI__builtin_ia32_fpclasspd512_mask:
 667:     intrinsicName = "x86.avx512.fpclass.pd.512";
 668:     break;
 669:   }
 670: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 671-676
```cpp
 671:   auto cmpResultTy = cir::VectorType::get(builder.getSIntNTy(1), numElts);
 672:   mlir::Value fpclass =
 673:       builder.emitIntrinsicCallOp(loc, intrinsicName, cmpResultTy, ops);
 674:   return emitX86MaskedCompareResult(builder, fpclass, numElts, maskIn, loc);
 675: }
 676: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 677-686
```cpp
 677: static mlir::Value emitX86Aes(CIRGenBuilderTy &builder, mlir::Location loc,
 678:                               llvm::StringRef intrinsicName, mlir::Type retType,
 679:                               llvm::ArrayRef<mlir::Value> ops) {
 680:   // Create return struct type and call intrinsic function.
 681:   mlir::Type vecType =
 682:       mlir::cast<cir::PointerType>(ops[0].getType()).getPointee();
 683:   cir::RecordType rstRecTy = builder.getAnonRecordTy({retType, vecType});
 684:   mlir::Value rstValueRec = builder.emitIntrinsicCallOp(
 685:       loc, intrinsicName, rstRecTy, mlir::ValueRange{ops[1], ops[2]});
 686: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Notable callable symbols here include `emitX86Aes`. It introduces or references types such as `type`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 其中值得关注的可调用符号包括 `emitX86Aes`。 它引入或引用了诸如 `type` 等类型。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 687-695
```cpp
 687:   // Extract the first return value and truncate it to 1 bit, then cast result
 688:   // to bool value.
 689:   mlir::Value flag =
 690:       cir::ExtractMemberOp::create(builder, loc, rstValueRec, /*index=*/0);
 691:   mlir::Value flagBit0 = builder.createCast(loc, cir::CastKind::integral, flag,
 692:                                             builder.getUIntNTy(1));
 693:   mlir::Value succ = builder.createCast(loc, cir::CastKind::int_to_bool,
 694:                                         flagBit0, builder.getBoolTy());
 695: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::ExtractMemberOp::create`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::ExtractMemberOp::create`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 696-716
```cpp
 696:   // Extract the second return value, store it to output address if success.
 697:   mlir::Value out =
 698:       cir::ExtractMemberOp::create(builder, loc, rstValueRec, /*index=*/1);
 699:   Address outAddr(ops[0], /*align=*/CharUnits::fromQuantity(16));
 700:   cir::IfOp::create(
 701:       builder, loc, succ, /*withElseRegion=*/true,
 702:       /*thenBuilder=*/
 703:       [&](mlir::OpBuilder &b, mlir::Location) {
 704:         builder.createStore(loc, out, outAddr);
 705:         builder.createYield(loc);
 706:       },
 707:       /*elseBuilder=*/
 708:       [&](mlir::OpBuilder &b, mlir::Location) {
 709:         mlir::Value zero = builder.getNullValue(vecType, loc);
 710:         builder.createStore(loc, zero, outAddr);
 711:         builder.createYield(loc);
 712:       });
 713: 
 714:   return cir::ExtractMemberOp::create(builder, loc, rstValueRec, /*index=*/0);
 715: }
 716: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::ExtractMemberOp::create`, `outAddr`, `cir::IfOp::create`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::ExtractMemberOp::create`、`outAddr`、`cir::IfOp::create`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 717-723
```cpp
 717: static mlir::Value emitX86Aeswide(CIRGenBuilderTy &builder, mlir::Location loc,
 718:                                   llvm::StringRef intrinsicName,
 719:                                   mlir::Type retType,
 720:                                   llvm::ArrayRef<mlir::Value> ops) {
 721:   mlir::Type vecType =
 722:       mlir::cast<cir::PointerType>(ops[1].getType()).getPointee();
 723: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86Aeswide`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86Aeswide`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 724-743
```cpp
 724:   // Create struct for return type and load input arguments, then call
 725:   // intrinsic function.
 726:   mlir::Type recTypes[9] = {retType, vecType, vecType, vecType, vecType,
 727:                             vecType, vecType, vecType, vecType};
 728:   mlir::Value arguments[9];
 729:   arguments[0] = ops[2];
 730:   for (int i = 0; i < 8; i++) {
 731:     // Loading each vector argument from input address.
 732:     cir::ConstantOp idx = builder.getUInt32(i, loc);
 733:     mlir::Value nextInElePtr =
 734:         builder.getArrayElement(loc, loc, ops[1], vecType, idx,
 735:                                 /*shouldDecay=*/false);
 736:     arguments[i + 1] =
 737:         builder.createAlignedLoad(loc, vecType, nextInElePtr,
 738:                                   /*align=*/CharUnits::fromQuantity(16));
 739:   }
 740:   cir::RecordType rstRecTy = builder.getAnonRecordTy(recTypes);
 741:   mlir::Value rstValueRec =
 742:       builder.emitIntrinsicCallOp(loc, intrinsicName, rstRecTy, arguments);
 743: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. It introduces or references types such as `for`. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 它引入或引用了诸如 `for` 等类型。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 744-752
```cpp
 744:   // Extract the first return value and truncate it to 1 bit, then cast result
 745:   // to bool value.
 746:   mlir::Value flag =
 747:       cir::ExtractMemberOp::create(builder, loc, rstValueRec, /*index=*/0);
 748:   mlir::Value flagBit0 = builder.createCast(loc, cir::CastKind::integral, flag,
 749:                                             builder.getUIntNTy(1));
 750:   mlir::Value succ = builder.createCast(loc, cir::CastKind::int_to_bool,
 751:                                         flagBit0, builder.getBoolTy());
 752: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::ExtractMemberOp::create`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::ExtractMemberOp::create`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 753-770
```cpp
 753:   // Extract other return values, store those to output address if success.
 754:   cir::IfOp::create(
 755:       builder, loc, succ, /*withElseRegion=*/true,
 756:       /*thenBuilder=*/
 757:       [&](mlir::OpBuilder &b, mlir::Location) {
 758:         for (int i = 0; i < 8; i++) {
 759:           mlir::Value out =
 760:               cir::ExtractMemberOp::create(builder, loc, rstValueRec,
 761:                                            /*index=*/i + 1);
 762:           cir::ConstantOp idx = builder.getUInt32(i, loc);
 763:           mlir::Value nextOutEleAddr =
 764:               builder.getArrayElement(loc, loc, ops[0], vecType, idx,
 765:                                       /*shouldDecay=*/false);
 766:           Address outAddr(nextOutEleAddr,
 767:                           /*align=*/CharUnits::fromQuantity(16));
 768:           builder.createStore(loc, out, outAddr);
 769:         }
 770:         builder.createYield(loc);
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::IfOp::create`, `cir::ExtractMemberOp::create`, `outAddr`. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::IfOp::create`、`cir::ExtractMemberOp::create`、`outAddr`。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 771-789
```cpp
 771:       },
 772:       /*elseBuilder=*/
 773:       [&](mlir::OpBuilder &b, mlir::Location) {
 774:         mlir::Value zero = builder.getNullValue(vecType, loc);
 775:         for (int i = 0; i < 8; i++) {
 776:           cir::ConstantOp idx = builder.getUInt32(i, loc);
 777:           mlir::Value nextOutEleAddr =
 778:               builder.getArrayElement(loc, loc, ops[0], vecType, idx,
 779:                                       /*shouldDecay=*/false);
 780:           Address outAddr(nextOutEleAddr,
 781:                           /*align=*/CharUnits::fromQuantity(16));
 782:           builder.createStore(loc, zero, outAddr);
 783:         }
 784:         builder.createYield(loc);
 785:       });
 786: 
 787:   return cir::ExtractMemberOp::create(builder, loc, rstValueRec, /*index=*/0);
 788: }
 789: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `outAddr`. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `outAddr`。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 790-801
```cpp
 790: static mlir::Value emitX86MaskedLoad(CIRGenBuilderTy &builder,
 791:                                      ArrayRef<mlir::Value> ops,
 792:                                      llvm::Align alignment,
 793:                                      mlir::Location loc) {
 794:   mlir::Type ty = ops[1].getType();
 795:   mlir::Value ptr = ops[0];
 796:   mlir::Value maskVec = getMaskVecValue(builder, loc, ops[2],
 797:                                         cast<cir::VectorType>(ty).getSize());
 798: 
 799:   return builder.createMaskedLoad(loc, ty, ptr, alignment, maskVec, ops[1]);
 800: }
 801: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskedLoad`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskedLoad`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 802-816
```cpp
 802: std::optional<mlir::Value>
 803: CIRGenFunction::emitX86BuiltinExpr(unsigned builtinID, const CallExpr *expr) {
 804:   if (builtinID == Builtin::BI__builtin_cpu_is) {
 805:     cgm.errorNYI(expr->getSourceRange(), "__builtin_cpu_is");
 806:     return mlir::Value{};
 807:   }
 808:   if (builtinID == Builtin::BI__builtin_cpu_supports) {
 809:     cgm.errorNYI(expr->getSourceRange(), "__builtin_cpu_supports");
 810:     return mlir::Value{};
 811:   }
 812:   if (builtinID == Builtin::BI__builtin_cpu_init) {
 813:     cgm.errorNYI(expr->getSourceRange(), "__builtin_cpu_init");
 814:     return mlir::Value{};
 815:   }
 816: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenFunction::emitX86BuiltinExpr`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenFunction::emitX86BuiltinExpr`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 817-826
```cpp
 817:   // Handle MSVC intrinsics before argument evaluation to prevent double
 818:   // evaluation.
 819:   assert(!cir::MissingFeatures::msvcBuiltins());
 820: 
 821:   // Find out if any arguments are required to be integer constant expressions.
 822:   assert(!cir::MissingFeatures::handleBuiltinICEArguments());
 823: 
 824:   // The operands of the builtin call
 825:   llvm::SmallVector<mlir::Value> ops;
 826: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 827-839
```cpp
 827:   // `ICEArguments` is a bitmap indicating whether the argument at the i-th bit
 828:   // is required to be a constant integer expression.
 829:   unsigned iceArguments = 0;
 830:   ASTContext::GetBuiltinTypeError error;
 831:   getContext().GetBuiltinType(builtinID, error, &iceArguments);
 832:   assert(error == ASTContext::GE_None && "Error while getting builtin type.");
 833: 
 834:   for (auto [idx, arg] : llvm::enumerate(expr->arguments()))
 835:     ops.push_back(emitScalarOrConstFoldImmArg(iceArguments, idx, arg));
 836: 
 837:   CIRGenBuilderTy &builder = getBuilder();
 838:   mlir::Type voidTy = builder.getVoidTy();
 839: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getContext`, `assert`. Loop constructs indicate repeated processing over collections or state transitions. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getContext`、`assert`。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 840-857
```cpp
 840:   switch (builtinID) {
 841:   default:
 842:     return std::nullopt;
 843:   case X86::BI_mm_clflush:
 844:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()),
 845:                                        "x86.sse2.clflush", voidTy, ops[0]);
 846:   case X86::BI_mm_lfence:
 847:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()),
 848:                                        "x86.sse2.lfence", voidTy);
 849:   case X86::BI_mm_pause:
 850:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()),
 851:                                        "x86.sse2.pause", voidTy);
 852:   case X86::BI_mm_mfence:
 853:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()),
 854:                                        "x86.sse2.mfence", voidTy);
 855:   case X86::BI_mm_sfence:
 856:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()),
 857:                                        "x86.sse.sfence", voidTy);
```
- **EN**: This block contains local control-flow decisions or iterative processing. A switch statement is used to dispatch behavior across enumerated cases or kinds. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 858-872
```cpp
 858:   case X86::BI_mm_prefetch:
 859:   case X86::BI_m_prefetch:
 860:   case X86::BI_m_prefetchw:
 861:     return emitPrefetch(*this, builtinID, expr, ops);
 862:   case X86::BI__rdtsc:
 863:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()), "x86.rdtsc",
 864:                                        builder.getUInt64Ty());
 865:   case X86::BI__builtin_ia32_rdtscp: {
 866:     mlir::Location loc = getLoc(expr->getExprLoc());
 867:     mlir::Type i64Ty = builder.getUInt64Ty();
 868:     mlir::Type i32Ty = builder.getUInt32Ty();
 869:     mlir::Type structTy = builder.getAnonRecordTy({i64Ty, i32Ty});
 870:     mlir::Value result =
 871:         builder.emitIntrinsicCallOp(loc, "x86.rdtscp", structTy);
 872: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitPrefetch`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitPrefetch`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 873-878
```cpp
 873:     // Extract and store processor_id (element 1 of the returned struct)
 874:     mlir::Value processorId =
 875:         cir::ExtractMemberOp::create(builder, loc, i32Ty, result, 1);
 876:     // ops[0] is the address to store the processor ID
 877:     builder.createStore(loc, processorId, Address{ops[0], CharUnits::One()});
 878: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Notable callable symbols here include `cir::ExtractMemberOp::create`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 其中值得关注的可调用符号包括 `cir::ExtractMemberOp::create`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 879-896
```cpp
 879:     // Return timestamp (element 0 of the returned struct)
 880:     return cir::ExtractMemberOp::create(builder, loc, i64Ty, result, 0);
 881:   }
 882:   case X86::BI__builtin_ia32_lzcnt_u16:
 883:   case X86::BI__builtin_ia32_lzcnt_u32:
 884:   case X86::BI__builtin_ia32_lzcnt_u64: {
 885:     mlir::Location loc = getLoc(expr->getExprLoc());
 886:     mlir::Value isZeroPoison = builder.getFalse(loc);
 887:     return builder.emitIntrinsicCallOp(loc, "ctlz", ops[0].getType(),
 888:                                        mlir::ValueRange{ops[0], isZeroPoison});
 889:   }
 890:   case X86::BI__builtin_ia32_tzcnt_u16:
 891:   case X86::BI__builtin_ia32_tzcnt_u32:
 892:   case X86::BI__builtin_ia32_tzcnt_u64: {
 893:     mlir::Location loc = getLoc(expr->getExprLoc());
 894:     mlir::Value isZeroPoison = builder.getFalse(loc);
 895:     return builder.emitIntrinsicCallOp(loc, "cttz", ops[0].getType(),
 896:                                        mlir::ValueRange{ops[0], isZeroPoison});
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 897-914
```cpp
 897:   }
 898:   case X86::BI__builtin_ia32_undef128:
 899:   case X86::BI__builtin_ia32_undef256:
 900:   case X86::BI__builtin_ia32_undef512:
 901:     // The x86 definition of "undef" is not the same as the LLVM definition
 902:     // (PR32176). We leave optimizing away an unnecessary zero constant to the
 903:     // IR optimizer and backend.
 904:     // TODO: If we had a "freeze" IR instruction to generate a fixed undef
 905:     //  value, we should use that here instead of a zero.
 906:     return builder.getNullValue(convertType(expr->getType()),
 907:                                 getLoc(expr->getExprLoc()));
 908:   case X86::BI__builtin_ia32_vec_ext_v4hi:
 909:   case X86::BI__builtin_ia32_vec_ext_v16qi:
 910:   case X86::BI__builtin_ia32_vec_ext_v8hi:
 911:   case X86::BI__builtin_ia32_vec_ext_v4si:
 912:   case X86::BI__builtin_ia32_vec_ext_v4sf:
 913:   case X86::BI__builtin_ia32_vec_ext_v2di:
 914:   case X86::BI__builtin_ia32_vec_ext_v32qi:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getLoc`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getLoc`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 915-925
```cpp
 915:   case X86::BI__builtin_ia32_vec_ext_v16hi:
 916:   case X86::BI__builtin_ia32_vec_ext_v8si:
 917:   case X86::BI__builtin_ia32_vec_ext_v4di: {
 918:     unsigned numElts = cast<cir::VectorType>(ops[0].getType()).getSize();
 919: 
 920:     uint64_t index = getZExtIntValueFromConstOp(ops[1]);
 921:     index &= numElts - 1;
 922: 
 923:     cir::ConstantOp indexVal =
 924:         builder.getUInt64(index, getLoc(expr->getExprLoc()));
 925: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 926-943
```cpp
 926:     // These builtins exist so we can ensure the index is an ICE and in range.
 927:     // Otherwise we could just do this in the header file.
 928:     return cir::VecExtractOp::create(builder, getLoc(expr->getExprLoc()),
 929:                                      ops[0], indexVal);
 930:   }
 931:   case X86::BI__builtin_ia32_vec_set_v4hi:
 932:   case X86::BI__builtin_ia32_vec_set_v16qi:
 933:   case X86::BI__builtin_ia32_vec_set_v8hi:
 934:   case X86::BI__builtin_ia32_vec_set_v4si:
 935:   case X86::BI__builtin_ia32_vec_set_v2di:
 936:   case X86::BI__builtin_ia32_vec_set_v32qi:
 937:   case X86::BI__builtin_ia32_vec_set_v16hi:
 938:   case X86::BI__builtin_ia32_vec_set_v8si:
 939:   case X86::BI__builtin_ia32_vec_set_v4di: {
 940:     return emitVecInsert(builder, getLoc(expr->getExprLoc()), ops[0], ops[1],
 941:                          ops[2]);
 942:   }
 943:   case X86::BI__builtin_ia32_kunpckhi:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 944-961
```cpp
 944:     return emitX86MaskUnpack(builder, getLoc(expr->getExprLoc()),
 945:                              "x86.avx512.kunpackb", ops);
 946:   case X86::BI__builtin_ia32_kunpcksi:
 947:     return emitX86MaskUnpack(builder, getLoc(expr->getExprLoc()),
 948:                              "x86.avx512.kunpackw", ops);
 949:   case X86::BI__builtin_ia32_kunpckdi:
 950:     return emitX86MaskUnpack(builder, getLoc(expr->getExprLoc()),
 951:                              "x86.avx512.kunpackd", ops);
 952:   case X86::BI_mm_setcsr:
 953:   case X86::BI__builtin_ia32_ldmxcsr: {
 954:     mlir::Location loc = getLoc(expr->getExprLoc());
 955:     Address tmp = createMemTemp(expr->getArg(0)->getType(), loc);
 956:     builder.createStore(loc, ops[0], tmp);
 957:     return builder.emitIntrinsicCallOp(loc, "x86.sse.ldmxcsr",
 958:                                        builder.getVoidTy(), tmp.getPointer());
 959:   }
 960:   case X86::BI_mm_getcsr:
 961:   case X86::BI__builtin_ia32_stmxcsr: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskUnpack`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskUnpack`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 962-979
```cpp
 962:     mlir::Location loc = getLoc(expr->getExprLoc());
 963:     Address tmp = createMemTemp(expr->getType(), loc);
 964:     builder.emitIntrinsicCallOp(loc, "x86.sse.stmxcsr", builder.getVoidTy(),
 965:                                 tmp.getPointer());
 966:     return builder.createLoad(loc, tmp);
 967:   }
 968:   case X86::BI__builtin_ia32_xsave:
 969:   case X86::BI__builtin_ia32_xsave64:
 970:   case X86::BI__builtin_ia32_xrstor:
 971:   case X86::BI__builtin_ia32_xrstor64:
 972:   case X86::BI__builtin_ia32_xsaveopt:
 973:   case X86::BI__builtin_ia32_xsaveopt64:
 974:   case X86::BI__builtin_ia32_xrstors:
 975:   case X86::BI__builtin_ia32_xrstors64:
 976:   case X86::BI__builtin_ia32_xsavec:
 977:   case X86::BI__builtin_ia32_xsavec64:
 978:   case X86::BI__builtin_ia32_xsaves:
 979:   case X86::BI__builtin_ia32_xsaves64:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 980-997
```cpp
 980:   case X86::BI__builtin_ia32_xsetbv:
 981:   case X86::BI_xsetbv: {
 982:     mlir::Location loc = getLoc(expr->getExprLoc());
 983:     StringRef intrinsicName;
 984:     switch (builtinID) {
 985:     default:
 986:       llvm_unreachable("Unexpected builtin");
 987:     case X86::BI__builtin_ia32_xsave:
 988:       intrinsicName = "x86.xsave";
 989:       break;
 990:     case X86::BI__builtin_ia32_xsave64:
 991:       intrinsicName = "x86.xsave64";
 992:       break;
 993:     case X86::BI__builtin_ia32_xrstor:
 994:       intrinsicName = "x86.xrstor";
 995:       break;
 996:     case X86::BI__builtin_ia32_xrstor64:
 997:       intrinsicName = "x86.xrstor64";
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 998-1015
```cpp
 998:       break;
 999:     case X86::BI__builtin_ia32_xsaveopt:
1000:       intrinsicName = "x86.xsaveopt";
1001:       break;
1002:     case X86::BI__builtin_ia32_xsaveopt64:
1003:       intrinsicName = "x86.xsaveopt64";
1004:       break;
1005:     case X86::BI__builtin_ia32_xrstors:
1006:       intrinsicName = "x86.xrstors";
1007:       break;
1008:     case X86::BI__builtin_ia32_xrstors64:
1009:       intrinsicName = "x86.xrstors64";
1010:       break;
1011:     case X86::BI__builtin_ia32_xsavec:
1012:       intrinsicName = "x86.xsavec";
1013:       break;
1014:     case X86::BI__builtin_ia32_xsavec64:
1015:       intrinsicName = "x86.xsavec64";
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1016-1028
```cpp
1016:       break;
1017:     case X86::BI__builtin_ia32_xsaves:
1018:       intrinsicName = "x86.xsaves";
1019:       break;
1020:     case X86::BI__builtin_ia32_xsaves64:
1021:       intrinsicName = "x86.xsaves64";
1022:       break;
1023:     case X86::BI__builtin_ia32_xsetbv:
1024:     case X86::BI_xsetbv:
1025:       intrinsicName = "x86.xsetbv";
1026:       break;
1027:     }
1028: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1029-1034
```cpp
1029:     // The xsave family of instructions take a 64-bit mask that specifies
1030:     // which processor state components to save/restore. The hardware expects
1031:     // this mask split into two 32-bit registers: EDX (high 32 bits) and
1032:     // EAX (low 32 bits).
1033:     mlir::Type i32Ty = builder.getSInt32Ty();
1034: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1035-1043
```cpp
1035:     // Mhi = (uint32_t)(ops[1] >> 32) - extract high 32 bits via right shift
1036:     cir::ConstantOp shift32 = builder.getSInt64(32, loc);
1037:     mlir::Value mhi = builder.createShift(loc, ops[1], shift32.getResult(),
1038:                                           /*isShiftLeft=*/false);
1039:     mhi = builder.createIntCast(mhi, i32Ty);
1040: 
1041:     // Mlo = (uint32_t)ops[1] - extract low 32 bits by truncation
1042:     mlir::Value mlo = builder.createIntCast(ops[1], i32Ty);
1043: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1044-1061
```cpp
1044:     return builder.emitIntrinsicCallOp(loc, intrinsicName, voidTy,
1045:                                        mlir::ValueRange{ops[0], mhi, mlo});
1046:   }
1047:   case X86::BI__builtin_ia32_xgetbv:
1048:   case X86::BI_xgetbv:
1049:     // xgetbv reads the extended control register specified by ops[0] (ECX)
1050:     // and returns the 64-bit value
1051:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()), "x86.xgetbv",
1052:                                        builder.getUInt64Ty(), ops[0]);
1053:   case X86::BI__builtin_ia32_storedqudi128_mask:
1054:   case X86::BI__builtin_ia32_storedqusi128_mask:
1055:   case X86::BI__builtin_ia32_storedquhi128_mask:
1056:   case X86::BI__builtin_ia32_storedquqi128_mask:
1057:   case X86::BI__builtin_ia32_storeupd128_mask:
1058:   case X86::BI__builtin_ia32_storeups128_mask:
1059:   case X86::BI__builtin_ia32_storedqudi256_mask:
1060:   case X86::BI__builtin_ia32_storedqusi256_mask:
1061:   case X86::BI__builtin_ia32_storedquhi256_mask:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1062-1079
```cpp
1062:   case X86::BI__builtin_ia32_storedquqi256_mask:
1063:   case X86::BI__builtin_ia32_storeupd256_mask:
1064:   case X86::BI__builtin_ia32_storeups256_mask:
1065:   case X86::BI__builtin_ia32_storedqudi512_mask:
1066:   case X86::BI__builtin_ia32_storedqusi512_mask:
1067:   case X86::BI__builtin_ia32_storedquhi512_mask:
1068:   case X86::BI__builtin_ia32_storedquqi512_mask:
1069:   case X86::BI__builtin_ia32_storeupd512_mask:
1070:   case X86::BI__builtin_ia32_storeups512_mask:
1071:   case X86::BI__builtin_ia32_storesbf16128_mask:
1072:   case X86::BI__builtin_ia32_storesh128_mask:
1073:   case X86::BI__builtin_ia32_storess128_mask:
1074:   case X86::BI__builtin_ia32_storesd128_mask:
1075:     cgm.errorNYI(expr->getSourceRange(),
1076:                  std::string("unimplemented x86 builtin call: ") +
1077:                      getContext().BuiltinInfo.getName(builtinID));
1078:     return mlir::Value{};
1079:   case X86::BI__builtin_ia32_cvtmask2b128:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1080-1097
```cpp
1080:   case X86::BI__builtin_ia32_cvtmask2b256:
1081:   case X86::BI__builtin_ia32_cvtmask2b512:
1082:   case X86::BI__builtin_ia32_cvtmask2w128:
1083:   case X86::BI__builtin_ia32_cvtmask2w256:
1084:   case X86::BI__builtin_ia32_cvtmask2w512:
1085:   case X86::BI__builtin_ia32_cvtmask2d128:
1086:   case X86::BI__builtin_ia32_cvtmask2d256:
1087:   case X86::BI__builtin_ia32_cvtmask2d512:
1088:   case X86::BI__builtin_ia32_cvtmask2q128:
1089:   case X86::BI__builtin_ia32_cvtmask2q256:
1090:   case X86::BI__builtin_ia32_cvtmask2q512:
1091:     return emitX86SExtMask(this->getBuilder(), ops[0],
1092:                            convertType(expr->getType()),
1093:                            getLoc(expr->getExprLoc()));
1094:   case X86::BI__builtin_ia32_cvtb2mask128:
1095:   case X86::BI__builtin_ia32_cvtb2mask256:
1096:   case X86::BI__builtin_ia32_cvtb2mask512:
1097:   case X86::BI__builtin_ia32_cvtw2mask128:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86SExtMask`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86SExtMask`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1098-1115
```cpp
1098:   case X86::BI__builtin_ia32_cvtw2mask256:
1099:   case X86::BI__builtin_ia32_cvtw2mask512:
1100:   case X86::BI__builtin_ia32_cvtd2mask128:
1101:   case X86::BI__builtin_ia32_cvtd2mask256:
1102:   case X86::BI__builtin_ia32_cvtd2mask512:
1103:   case X86::BI__builtin_ia32_cvtq2mask128:
1104:   case X86::BI__builtin_ia32_cvtq2mask256:
1105:   case X86::BI__builtin_ia32_cvtq2mask512:
1106:     return emitX86ConvertToMask(*this, this->getBuilder(), ops[0],
1107:                                 getLoc(expr->getExprLoc()));
1108:   case X86::BI__builtin_ia32_cvtdq2ps512_mask:
1109:   case X86::BI__builtin_ia32_cvtqq2ps512_mask:
1110:   case X86::BI__builtin_ia32_cvtqq2pd512_mask:
1111:   case X86::BI__builtin_ia32_vcvtw2ph512_mask:
1112:   case X86::BI__builtin_ia32_vcvtdq2ph512_mask:
1113:   case X86::BI__builtin_ia32_vcvtqq2ph512_mask:
1114:   case X86::BI__builtin_ia32_cvtudq2ps512_mask:
1115:   case X86::BI__builtin_ia32_cvtuqq2ps512_mask:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86ConvertToMask`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86ConvertToMask`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1116-1133
```cpp
1116:   case X86::BI__builtin_ia32_cvtuqq2pd512_mask:
1117:   case X86::BI__builtin_ia32_vcvtuw2ph512_mask:
1118:   case X86::BI__builtin_ia32_vcvtudq2ph512_mask:
1119:   case X86::BI__builtin_ia32_vcvtuqq2ph512_mask:
1120:   case X86::BI__builtin_ia32_vfmaddsh3_mask:
1121:   case X86::BI__builtin_ia32_vfmaddss3_mask:
1122:   case X86::BI__builtin_ia32_vfmaddsd3_mask:
1123:   case X86::BI__builtin_ia32_vfmaddsh3_maskz:
1124:   case X86::BI__builtin_ia32_vfmaddss3_maskz:
1125:   case X86::BI__builtin_ia32_vfmaddsd3_maskz:
1126:   case X86::BI__builtin_ia32_vfmaddsh3_mask3:
1127:   case X86::BI__builtin_ia32_vfmaddss3_mask3:
1128:   case X86::BI__builtin_ia32_vfmaddsd3_mask3:
1129:   case X86::BI__builtin_ia32_vfmsubsh3_mask3:
1130:   case X86::BI__builtin_ia32_vfmsubss3_mask3:
1131:   case X86::BI__builtin_ia32_vfmsubsd3_mask3:
1132:   case X86::BI__builtin_ia32_vfmaddph512_mask:
1133:   case X86::BI__builtin_ia32_vfmaddph512_maskz:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1134-1151
```cpp
1134:   case X86::BI__builtin_ia32_vfmaddph512_mask3:
1135:   case X86::BI__builtin_ia32_vfmaddps512_mask:
1136:   case X86::BI__builtin_ia32_vfmaddps512_maskz:
1137:   case X86::BI__builtin_ia32_vfmaddps512_mask3:
1138:   case X86::BI__builtin_ia32_vfmsubps512_mask3:
1139:   case X86::BI__builtin_ia32_vfmaddpd512_mask:
1140:   case X86::BI__builtin_ia32_vfmaddpd512_maskz:
1141:   case X86::BI__builtin_ia32_vfmaddpd512_mask3:
1142:   case X86::BI__builtin_ia32_vfmsubpd512_mask3:
1143:   case X86::BI__builtin_ia32_vfmsubph512_mask3:
1144:   case X86::BI__builtin_ia32_vfmaddsubph512_mask:
1145:   case X86::BI__builtin_ia32_vfmaddsubph512_maskz:
1146:   case X86::BI__builtin_ia32_vfmaddsubph512_mask3:
1147:   case X86::BI__builtin_ia32_vfmsubaddph512_mask3:
1148:   case X86::BI__builtin_ia32_vfmaddsubps512_mask:
1149:   case X86::BI__builtin_ia32_vfmaddsubps512_maskz:
1150:   case X86::BI__builtin_ia32_vfmaddsubps512_mask3:
1151:   case X86::BI__builtin_ia32_vfmsubaddps512_mask3:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1152-1172
```cpp
1152:   case X86::BI__builtin_ia32_vfmaddsubpd512_mask:
1153:   case X86::BI__builtin_ia32_vfmaddsubpd512_maskz:
1154:   case X86::BI__builtin_ia32_vfmaddsubpd512_mask3:
1155:   case X86::BI__builtin_ia32_vfmsubaddpd512_mask3:
1156:   case X86::BI__builtin_ia32_movdqa32store128_mask:
1157:   case X86::BI__builtin_ia32_movdqa64store128_mask:
1158:   case X86::BI__builtin_ia32_storeaps128_mask:
1159:   case X86::BI__builtin_ia32_storeapd128_mask:
1160:   case X86::BI__builtin_ia32_movdqa32store256_mask:
1161:   case X86::BI__builtin_ia32_movdqa64store256_mask:
1162:   case X86::BI__builtin_ia32_storeaps256_mask:
1163:   case X86::BI__builtin_ia32_storeapd256_mask:
1164:   case X86::BI__builtin_ia32_movdqa32store512_mask:
1165:   case X86::BI__builtin_ia32_movdqa64store512_mask:
1166:   case X86::BI__builtin_ia32_storeaps512_mask:
1167:   case X86::BI__builtin_ia32_storeapd512_mask:
1168:     cgm.errorNYI(expr->getSourceRange(),
1169:                  std::string("unimplemented X86 builtin call: ") +
1170:                      getContext().BuiltinInfo.getName(builtinID));
1171:     return {};
1172: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1173-1190
```cpp
1173:   case X86::BI__builtin_ia32_loadups128_mask:
1174:   case X86::BI__builtin_ia32_loadups256_mask:
1175:   case X86::BI__builtin_ia32_loadups512_mask:
1176:   case X86::BI__builtin_ia32_loadupd128_mask:
1177:   case X86::BI__builtin_ia32_loadupd256_mask:
1178:   case X86::BI__builtin_ia32_loadupd512_mask:
1179:   case X86::BI__builtin_ia32_loaddquqi128_mask:
1180:   case X86::BI__builtin_ia32_loaddquqi256_mask:
1181:   case X86::BI__builtin_ia32_loaddquqi512_mask:
1182:   case X86::BI__builtin_ia32_loaddquhi128_mask:
1183:   case X86::BI__builtin_ia32_loaddquhi256_mask:
1184:   case X86::BI__builtin_ia32_loaddquhi512_mask:
1185:   case X86::BI__builtin_ia32_loaddqusi128_mask:
1186:   case X86::BI__builtin_ia32_loaddqusi256_mask:
1187:   case X86::BI__builtin_ia32_loaddqusi512_mask:
1188:   case X86::BI__builtin_ia32_loaddqudi128_mask:
1189:   case X86::BI__builtin_ia32_loaddqudi256_mask:
1190:   case X86::BI__builtin_ia32_loaddqudi512_mask:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1191-1197
```cpp
1191:   case X86::BI__builtin_ia32_loadsbf16128_mask:
1192:   case X86::BI__builtin_ia32_loadsh128_mask:
1193:   case X86::BI__builtin_ia32_loadss128_mask:
1194:   case X86::BI__builtin_ia32_loadsd128_mask:
1195:     return emitX86MaskedLoad(builder, ops, llvm::Align(1),
1196:                              getLoc(expr->getExprLoc()));
1197: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskedLoad`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskedLoad`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1198-1215
```cpp
1198:   case X86::BI__builtin_ia32_loadaps128_mask:
1199:   case X86::BI__builtin_ia32_loadaps256_mask:
1200:   case X86::BI__builtin_ia32_loadaps512_mask:
1201:   case X86::BI__builtin_ia32_loadapd128_mask:
1202:   case X86::BI__builtin_ia32_loadapd256_mask:
1203:   case X86::BI__builtin_ia32_loadapd512_mask:
1204:   case X86::BI__builtin_ia32_movdqa32load128_mask:
1205:   case X86::BI__builtin_ia32_movdqa32load256_mask:
1206:   case X86::BI__builtin_ia32_movdqa32load512_mask:
1207:   case X86::BI__builtin_ia32_movdqa64load128_mask:
1208:   case X86::BI__builtin_ia32_movdqa64load256_mask:
1209:   case X86::BI__builtin_ia32_movdqa64load512_mask:
1210:     return emitX86MaskedLoad(
1211:         builder, ops,
1212:         getContext()
1213:             .getTypeAlignInChars(expr->getArg(1)->getType())
1214:             .getAsAlign(),
1215:         getLoc(expr->getExprLoc()));
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskedLoad`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskedLoad`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1216-1234
```cpp
1216: 
1217:   case X86::BI__builtin_ia32_expandloaddf128_mask:
1218:   case X86::BI__builtin_ia32_expandloaddf256_mask:
1219:   case X86::BI__builtin_ia32_expandloaddf512_mask:
1220:   case X86::BI__builtin_ia32_expandloadsf128_mask:
1221:   case X86::BI__builtin_ia32_expandloadsf256_mask:
1222:   case X86::BI__builtin_ia32_expandloadsf512_mask:
1223:   case X86::BI__builtin_ia32_expandloaddi128_mask:
1224:   case X86::BI__builtin_ia32_expandloaddi256_mask:
1225:   case X86::BI__builtin_ia32_expandloaddi512_mask:
1226:   case X86::BI__builtin_ia32_expandloadsi128_mask:
1227:   case X86::BI__builtin_ia32_expandloadsi256_mask:
1228:   case X86::BI__builtin_ia32_expandloadsi512_mask:
1229:   case X86::BI__builtin_ia32_expandloadhi128_mask:
1230:   case X86::BI__builtin_ia32_expandloadhi256_mask:
1231:   case X86::BI__builtin_ia32_expandloadhi512_mask:
1232:   case X86::BI__builtin_ia32_expandloadqi128_mask:
1233:   case X86::BI__builtin_ia32_expandloadqi256_mask:
1234:   case X86::BI__builtin_ia32_expandloadqi512_mask:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1235-1252
```cpp
1235:   case X86::BI__builtin_ia32_compressstoredf128_mask:
1236:   case X86::BI__builtin_ia32_compressstoredf256_mask:
1237:   case X86::BI__builtin_ia32_compressstoredf512_mask:
1238:   case X86::BI__builtin_ia32_compressstoresf128_mask:
1239:   case X86::BI__builtin_ia32_compressstoresf256_mask:
1240:   case X86::BI__builtin_ia32_compressstoresf512_mask:
1241:   case X86::BI__builtin_ia32_compressstoredi128_mask:
1242:   case X86::BI__builtin_ia32_compressstoredi256_mask:
1243:   case X86::BI__builtin_ia32_compressstoredi512_mask:
1244:   case X86::BI__builtin_ia32_compressstoresi128_mask:
1245:   case X86::BI__builtin_ia32_compressstoresi256_mask:
1246:   case X86::BI__builtin_ia32_compressstoresi512_mask:
1247:   case X86::BI__builtin_ia32_compressstorehi128_mask:
1248:   case X86::BI__builtin_ia32_compressstorehi256_mask:
1249:   case X86::BI__builtin_ia32_compressstorehi512_mask:
1250:   case X86::BI__builtin_ia32_compressstoreqi128_mask:
1251:   case X86::BI__builtin_ia32_compressstoreqi256_mask:
1252:   case X86::BI__builtin_ia32_compressstoreqi512_mask:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1253-1270
```cpp
1253:     cgm.errorNYI(expr->getSourceRange(),
1254:                  std::string("unimplemented X86 builtin call: ") +
1255:                      getContext().BuiltinInfo.getName(builtinID));
1256:     return mlir::Value{};
1257:   case X86::BI__builtin_ia32_expanddf128_mask:
1258:   case X86::BI__builtin_ia32_expanddf256_mask:
1259:   case X86::BI__builtin_ia32_expanddf512_mask:
1260:   case X86::BI__builtin_ia32_expandsf128_mask:
1261:   case X86::BI__builtin_ia32_expandsf256_mask:
1262:   case X86::BI__builtin_ia32_expandsf512_mask:
1263:   case X86::BI__builtin_ia32_expanddi128_mask:
1264:   case X86::BI__builtin_ia32_expanddi256_mask:
1265:   case X86::BI__builtin_ia32_expanddi512_mask:
1266:   case X86::BI__builtin_ia32_expandsi128_mask:
1267:   case X86::BI__builtin_ia32_expandsi256_mask:
1268:   case X86::BI__builtin_ia32_expandsi512_mask:
1269:   case X86::BI__builtin_ia32_expandhi128_mask:
1270:   case X86::BI__builtin_ia32_expandhi256_mask:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1271-1288
```cpp
1271:   case X86::BI__builtin_ia32_expandhi512_mask:
1272:   case X86::BI__builtin_ia32_expandqi128_mask:
1273:   case X86::BI__builtin_ia32_expandqi256_mask:
1274:   case X86::BI__builtin_ia32_expandqi512_mask: {
1275:     mlir::Location loc = getLoc(expr->getExprLoc());
1276:     return emitX86CompressExpand(builder, loc, ops[0], ops[1], ops[2],
1277:                                  "x86.avx512.mask.expand");
1278:   }
1279:   case X86::BI__builtin_ia32_compressdf128_mask:
1280:   case X86::BI__builtin_ia32_compressdf256_mask:
1281:   case X86::BI__builtin_ia32_compressdf512_mask:
1282:   case X86::BI__builtin_ia32_compresssf128_mask:
1283:   case X86::BI__builtin_ia32_compresssf256_mask:
1284:   case X86::BI__builtin_ia32_compresssf512_mask:
1285:   case X86::BI__builtin_ia32_compressdi128_mask:
1286:   case X86::BI__builtin_ia32_compressdi256_mask:
1287:   case X86::BI__builtin_ia32_compressdi512_mask:
1288:   case X86::BI__builtin_ia32_compresssi128_mask:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1289-1306
```cpp
1289:   case X86::BI__builtin_ia32_compresssi256_mask:
1290:   case X86::BI__builtin_ia32_compresssi512_mask:
1291:   case X86::BI__builtin_ia32_compresshi128_mask:
1292:   case X86::BI__builtin_ia32_compresshi256_mask:
1293:   case X86::BI__builtin_ia32_compresshi512_mask:
1294:   case X86::BI__builtin_ia32_compressqi128_mask:
1295:   case X86::BI__builtin_ia32_compressqi256_mask:
1296:   case X86::BI__builtin_ia32_compressqi512_mask: {
1297:     mlir::Location loc = getLoc(expr->getExprLoc());
1298:     return emitX86CompressExpand(builder, loc, ops[0], ops[1], ops[2],
1299:                                  "x86.avx512.mask.compress");
1300:   }
1301:   case X86::BI__builtin_ia32_gather3div2df:
1302:   case X86::BI__builtin_ia32_gather3div2di:
1303:   case X86::BI__builtin_ia32_gather3div4df:
1304:   case X86::BI__builtin_ia32_gather3div4di:
1305:   case X86::BI__builtin_ia32_gather3div4sf:
1306:   case X86::BI__builtin_ia32_gather3div4si:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1307-1324
```cpp
1307:   case X86::BI__builtin_ia32_gather3div8sf:
1308:   case X86::BI__builtin_ia32_gather3div8si:
1309:   case X86::BI__builtin_ia32_gather3siv2df:
1310:   case X86::BI__builtin_ia32_gather3siv2di:
1311:   case X86::BI__builtin_ia32_gather3siv4df:
1312:   case X86::BI__builtin_ia32_gather3siv4di:
1313:   case X86::BI__builtin_ia32_gather3siv4sf:
1314:   case X86::BI__builtin_ia32_gather3siv4si:
1315:   case X86::BI__builtin_ia32_gather3siv8sf:
1316:   case X86::BI__builtin_ia32_gather3siv8si:
1317:   case X86::BI__builtin_ia32_gathersiv8df:
1318:   case X86::BI__builtin_ia32_gathersiv16sf:
1319:   case X86::BI__builtin_ia32_gatherdiv8df:
1320:   case X86::BI__builtin_ia32_gatherdiv16sf:
1321:   case X86::BI__builtin_ia32_gathersiv8di:
1322:   case X86::BI__builtin_ia32_gathersiv16si:
1323:   case X86::BI__builtin_ia32_gatherdiv8di:
1324:   case X86::BI__builtin_ia32_gatherdiv16si: {
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1325-1342
```cpp
1325:     StringRef intrinsicName;
1326:     switch (builtinID) {
1327:     default:
1328:       llvm_unreachable("Unexpected builtin");
1329:     case X86::BI__builtin_ia32_gather3div2df:
1330:       intrinsicName = "x86.avx512.mask.gather3div2.df";
1331:       break;
1332:     case X86::BI__builtin_ia32_gather3div2di:
1333:       intrinsicName = "x86.avx512.mask.gather3div2.di";
1334:       break;
1335:     case X86::BI__builtin_ia32_gather3div4df:
1336:       intrinsicName = "x86.avx512.mask.gather3div4.df";
1337:       break;
1338:     case X86::BI__builtin_ia32_gather3div4di:
1339:       intrinsicName = "x86.avx512.mask.gather3div4.di";
1340:       break;
1341:     case X86::BI__builtin_ia32_gather3div4sf:
1342:       intrinsicName = "x86.avx512.mask.gather3div4.sf";
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 1343-1360
```cpp
1343:       break;
1344:     case X86::BI__builtin_ia32_gather3div4si:
1345:       intrinsicName = "x86.avx512.mask.gather3div4.si";
1346:       break;
1347:     case X86::BI__builtin_ia32_gather3div8sf:
1348:       intrinsicName = "x86.avx512.mask.gather3div8.sf";
1349:       break;
1350:     case X86::BI__builtin_ia32_gather3div8si:
1351:       intrinsicName = "x86.avx512.mask.gather3div8.si";
1352:       break;
1353:     case X86::BI__builtin_ia32_gather3siv2df:
1354:       intrinsicName = "x86.avx512.mask.gather3siv2.df";
1355:       break;
1356:     case X86::BI__builtin_ia32_gather3siv2di:
1357:       intrinsicName = "x86.avx512.mask.gather3siv2.di";
1358:       break;
1359:     case X86::BI__builtin_ia32_gather3siv4df:
1360:       intrinsicName = "x86.avx512.mask.gather3siv4.df";
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1361-1378
```cpp
1361:       break;
1362:     case X86::BI__builtin_ia32_gather3siv4di:
1363:       intrinsicName = "x86.avx512.mask.gather3siv4.di";
1364:       break;
1365:     case X86::BI__builtin_ia32_gather3siv4sf:
1366:       intrinsicName = "x86.avx512.mask.gather3siv4.sf";
1367:       break;
1368:     case X86::BI__builtin_ia32_gather3siv4si:
1369:       intrinsicName = "x86.avx512.mask.gather3siv4.si";
1370:       break;
1371:     case X86::BI__builtin_ia32_gather3siv8sf:
1372:       intrinsicName = "x86.avx512.mask.gather3siv8.sf";
1373:       break;
1374:     case X86::BI__builtin_ia32_gather3siv8si:
1375:       intrinsicName = "x86.avx512.mask.gather3siv8.si";
1376:       break;
1377:     case X86::BI__builtin_ia32_gathersiv8df:
1378:       intrinsicName = "x86.avx512.mask.gather.dpd.512";
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1379-1396
```cpp
1379:       break;
1380:     case X86::BI__builtin_ia32_gathersiv16sf:
1381:       intrinsicName = "x86.avx512.mask.gather.dps.512";
1382:       break;
1383:     case X86::BI__builtin_ia32_gatherdiv8df:
1384:       intrinsicName = "x86.avx512.mask.gather.qpd.512";
1385:       break;
1386:     case X86::BI__builtin_ia32_gatherdiv16sf:
1387:       intrinsicName = "x86.avx512.mask.gather.qps.512";
1388:       break;
1389:     case X86::BI__builtin_ia32_gathersiv8di:
1390:       intrinsicName = "x86.avx512.mask.gather.dpq.512";
1391:       break;
1392:     case X86::BI__builtin_ia32_gathersiv16si:
1393:       intrinsicName = "x86.avx512.mask.gather.dpi.512";
1394:       break;
1395:     case X86::BI__builtin_ia32_gatherdiv8di:
1396:       intrinsicName = "x86.avx512.mask.gather.qpq.512";
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1397-1402
```cpp
1397:       break;
1398:     case X86::BI__builtin_ia32_gatherdiv16si:
1399:       intrinsicName = "x86.avx512.mask.gather.qpi.512";
1400:       break;
1401:     }
1402: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1403-1420
```cpp
1403:     mlir::Location loc = getLoc(expr->getExprLoc());
1404:     unsigned minElts =
1405:         std::min(cast<cir::VectorType>(ops[0].getType()).getSize(),
1406:                  cast<cir::VectorType>(ops[2].getType()).getSize());
1407:     ops[3] = getMaskVecValue(builder, loc, ops[3], minElts);
1408:     return builder.emitIntrinsicCallOp(loc, intrinsicName,
1409:                                        convertType(expr->getType()), ops);
1410:   }
1411:   case X86::BI__builtin_ia32_scattersiv8df:
1412:   case X86::BI__builtin_ia32_scattersiv16sf:
1413:   case X86::BI__builtin_ia32_scatterdiv8df:
1414:   case X86::BI__builtin_ia32_scatterdiv16sf:
1415:   case X86::BI__builtin_ia32_scattersiv8di:
1416:   case X86::BI__builtin_ia32_scattersiv16si:
1417:   case X86::BI__builtin_ia32_scatterdiv8di:
1418:   case X86::BI__builtin_ia32_scatterdiv16si:
1419:   case X86::BI__builtin_ia32_scatterdiv2df:
1420:   case X86::BI__builtin_ia32_scatterdiv2di:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::min`, `convertType`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::min`、`convertType`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1421-1438
```cpp
1421:   case X86::BI__builtin_ia32_scatterdiv4df:
1422:   case X86::BI__builtin_ia32_scatterdiv4di:
1423:   case X86::BI__builtin_ia32_scatterdiv4sf:
1424:   case X86::BI__builtin_ia32_scatterdiv4si:
1425:   case X86::BI__builtin_ia32_scatterdiv8sf:
1426:   case X86::BI__builtin_ia32_scatterdiv8si:
1427:   case X86::BI__builtin_ia32_scattersiv2df:
1428:   case X86::BI__builtin_ia32_scattersiv2di:
1429:   case X86::BI__builtin_ia32_scattersiv4df:
1430:   case X86::BI__builtin_ia32_scattersiv4di:
1431:   case X86::BI__builtin_ia32_scattersiv4sf:
1432:   case X86::BI__builtin_ia32_scattersiv4si:
1433:   case X86::BI__builtin_ia32_scattersiv8sf:
1434:   case X86::BI__builtin_ia32_scattersiv8si: {
1435:     llvm::StringRef intrinsicName;
1436:     switch (builtinID) {
1437:     default:
1438:       llvm_unreachable("Unexpected builtin");
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 1439-1456
```cpp
1439:     case X86::BI__builtin_ia32_scattersiv8df:
1440:       intrinsicName = "x86.avx512.mask.scatter.dpd.512";
1441:       break;
1442:     case X86::BI__builtin_ia32_scattersiv16sf:
1443:       intrinsicName = "x86.avx512.mask.scatter.dps.512";
1444:       break;
1445:     case X86::BI__builtin_ia32_scatterdiv8df:
1446:       intrinsicName = "x86.avx512.mask.scatter.qpd.512";
1447:       break;
1448:     case X86::BI__builtin_ia32_scatterdiv16sf:
1449:       intrinsicName = "x86.avx512.mask.scatter.qps.512";
1450:       break;
1451:     case X86::BI__builtin_ia32_scattersiv8di:
1452:       intrinsicName = "x86.avx512.mask.scatter.dpq.512";
1453:       break;
1454:     case X86::BI__builtin_ia32_scattersiv16si:
1455:       intrinsicName = "x86.avx512.mask.scatter.dpi.512";
1456:       break;
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1457-1474
```cpp
1457:     case X86::BI__builtin_ia32_scatterdiv8di:
1458:       intrinsicName = "x86.avx512.mask.scatter.qpq.512";
1459:       break;
1460:     case X86::BI__builtin_ia32_scatterdiv16si:
1461:       intrinsicName = "x86.avx512.mask.scatter.qpi.512";
1462:       break;
1463:     case X86::BI__builtin_ia32_scatterdiv2df:
1464:       intrinsicName = "x86.avx512.mask.scatterdiv2.df";
1465:       break;
1466:     case X86::BI__builtin_ia32_scatterdiv2di:
1467:       intrinsicName = "x86.avx512.mask.scatterdiv2.di";
1468:       break;
1469:     case X86::BI__builtin_ia32_scatterdiv4df:
1470:       intrinsicName = "x86.avx512.mask.scatterdiv4.df";
1471:       break;
1472:     case X86::BI__builtin_ia32_scatterdiv4di:
1473:       intrinsicName = "x86.avx512.mask.scatterdiv4.di";
1474:       break;
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1475-1492
```cpp
1475:     case X86::BI__builtin_ia32_scatterdiv4sf:
1476:       intrinsicName = "x86.avx512.mask.scatterdiv4.sf";
1477:       break;
1478:     case X86::BI__builtin_ia32_scatterdiv4si:
1479:       intrinsicName = "x86.avx512.mask.scatterdiv4.si";
1480:       break;
1481:     case X86::BI__builtin_ia32_scatterdiv8sf:
1482:       intrinsicName = "x86.avx512.mask.scatterdiv8.sf";
1483:       break;
1484:     case X86::BI__builtin_ia32_scatterdiv8si:
1485:       intrinsicName = "x86.avx512.mask.scatterdiv8.si";
1486:       break;
1487:     case X86::BI__builtin_ia32_scattersiv2df:
1488:       intrinsicName = "x86.avx512.mask.scattersiv2.df";
1489:       break;
1490:     case X86::BI__builtin_ia32_scattersiv2di:
1491:       intrinsicName = "x86.avx512.mask.scattersiv2.di";
1492:       break;
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1493-1512
```cpp
1493:     case X86::BI__builtin_ia32_scattersiv4df:
1494:       intrinsicName = "x86.avx512.mask.scattersiv4.df";
1495:       break;
1496:     case X86::BI__builtin_ia32_scattersiv4di:
1497:       intrinsicName = "x86.avx512.mask.scattersiv4.di";
1498:       break;
1499:     case X86::BI__builtin_ia32_scattersiv4sf:
1500:       intrinsicName = "x86.avx512.mask.scattersiv4.sf";
1501:       break;
1502:     case X86::BI__builtin_ia32_scattersiv4si:
1503:       intrinsicName = "x86.avx512.mask.scattersiv4.si";
1504:       break;
1505:     case X86::BI__builtin_ia32_scattersiv8sf:
1506:       intrinsicName = "x86.avx512.mask.scattersiv8.sf";
1507:       break;
1508:     case X86::BI__builtin_ia32_scattersiv8si:
1509:       intrinsicName = "x86.avx512.mask.scattersiv8.si";
1510:       break;
1511:     }
1512: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1513-1518
```cpp
1513:     mlir::Location loc = getLoc(expr->getExprLoc());
1514:     unsigned minElts =
1515:         std::min(cast<cir::VectorType>(ops[2].getType()).getSize(),
1516:                  cast<cir::VectorType>(ops[3].getType()).getSize());
1517:     ops[1] = getMaskVecValue(builder, loc, ops[1], minElts);
1518: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::min`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::min`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1519-1536
```cpp
1519:     return builder.emitIntrinsicCallOp(loc, intrinsicName,
1520:                                        convertType(expr->getType()), ops);
1521:   }
1522:   case X86::BI__builtin_ia32_vextractf128_pd256:
1523:   case X86::BI__builtin_ia32_vextractf128_ps256:
1524:   case X86::BI__builtin_ia32_vextractf128_si256:
1525:   case X86::BI__builtin_ia32_extract128i256:
1526:   case X86::BI__builtin_ia32_extractf64x4_mask:
1527:   case X86::BI__builtin_ia32_extractf32x4_mask:
1528:   case X86::BI__builtin_ia32_extracti64x4_mask:
1529:   case X86::BI__builtin_ia32_extracti32x4_mask:
1530:   case X86::BI__builtin_ia32_extractf32x8_mask:
1531:   case X86::BI__builtin_ia32_extracti32x8_mask:
1532:   case X86::BI__builtin_ia32_extractf32x4_256_mask:
1533:   case X86::BI__builtin_ia32_extracti32x4_256_mask:
1534:   case X86::BI__builtin_ia32_extractf64x2_256_mask:
1535:   case X86::BI__builtin_ia32_extracti64x2_256_mask:
1536:   case X86::BI__builtin_ia32_extractf64x2_512_mask:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `convertType`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `convertType`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1537-1552
```cpp
1537:   case X86::BI__builtin_ia32_extracti64x2_512_mask: {
1538:     mlir::Location loc = getLoc(expr->getExprLoc());
1539:     cir::VectorType dstTy = cast<cir::VectorType>(convertType(expr->getType()));
1540:     unsigned numElts = dstTy.getSize();
1541:     unsigned srcNumElts = cast<cir::VectorType>(ops[0].getType()).getSize();
1542:     unsigned subVectors = srcNumElts / numElts;
1543:     assert(llvm::isPowerOf2_32(subVectors) && "Expected power of 2 subvectors");
1544:     unsigned index =
1545:         ops[1].getDefiningOp<cir::ConstantOp>().getIntValue().getZExtValue();
1546: 
1547:     index &= subVectors - 1; // Remove any extra bits.
1548:     index *= numElts;
1549: 
1550:     int64_t indices[16];
1551:     std::iota(indices, indices + numElts, index);
1552: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`, `std::iota`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`、`std::iota`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1553-1559
```cpp
1553:     mlir::Value poison =
1554:         builder.getConstant(loc, cir::PoisonAttr::get(ops[0].getType()));
1555:     mlir::Value res = builder.createVecShuffle(loc, ops[0], poison,
1556:                                                ArrayRef(indices, numElts));
1557:     if (ops.size() == 4)
1558:       res = emitX86Select(builder, loc, ops[3], res, ops[2]);
1559: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ArrayRef`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ArrayRef`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1560-1577
```cpp
1560:     return res;
1561:   }
1562:   case X86::BI__builtin_ia32_vinsertf128_pd256:
1563:   case X86::BI__builtin_ia32_vinsertf128_ps256:
1564:   case X86::BI__builtin_ia32_vinsertf128_si256:
1565:   case X86::BI__builtin_ia32_insert128i256:
1566:   case X86::BI__builtin_ia32_insertf64x4:
1567:   case X86::BI__builtin_ia32_insertf32x4:
1568:   case X86::BI__builtin_ia32_inserti64x4:
1569:   case X86::BI__builtin_ia32_inserti32x4:
1570:   case X86::BI__builtin_ia32_insertf32x8:
1571:   case X86::BI__builtin_ia32_inserti32x8:
1572:   case X86::BI__builtin_ia32_insertf32x4_256:
1573:   case X86::BI__builtin_ia32_inserti32x4_256:
1574:   case X86::BI__builtin_ia32_insertf64x2_256:
1575:   case X86::BI__builtin_ia32_inserti64x2_256:
1576:   case X86::BI__builtin_ia32_insertf64x2_512:
1577:   case X86::BI__builtin_ia32_inserti64x2_512: {
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1578-1583
```cpp
1578:     unsigned dstNumElts = cast<cir::VectorType>(ops[0].getType()).getSize();
1579:     unsigned srcNumElts = cast<cir::VectorType>(ops[1].getType()).getSize();
1580:     unsigned subVectors = dstNumElts / srcNumElts;
1581:     assert(llvm::isPowerOf2_32(subVectors) && "Expected power of 2 subvectors");
1582:     assert(dstNumElts <= 16);
1583: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1584-1587
```cpp
1584:     uint64_t index = getZExtIntValueFromConstOp(ops[2]);
1585:     index &= subVectors - 1; // Remove any extra bits.
1586:     index *= srcNumElts;
1587: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1588-1594
```cpp
1588:     llvm::SmallVector<int64_t, 16> mask(dstNumElts);
1589:     for (unsigned i = 0; i != dstNumElts; ++i)
1590:       mask[i] = (i >= srcNumElts) ? srcNumElts + (i % srcNumElts) : i;
1591: 
1592:     mlir::Value op1 =
1593:         builder.createVecShuffle(getLoc(expr->getExprLoc()), ops[1], mask);
1594: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `mask`. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `mask`。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1595-1601
```cpp
1595:     for (unsigned i = 0; i != dstNumElts; ++i) {
1596:       if (i >= index && i < (index + srcNumElts))
1597:         mask[i] = (i - index) + dstNumElts;
1598:       else
1599:         mask[i] = i;
1600:     }
1601: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 1602-1622
```cpp
1602:     return builder.createVecShuffle(getLoc(expr->getExprLoc()), ops[0], op1,
1603:                                     mask);
1604:   }
1605:   case X86::BI__builtin_ia32_pmovqd512_mask:
1606:   case X86::BI__builtin_ia32_pmovwb512_mask: {
1607:     mlir::Value Res =
1608:         builder.createIntCast(ops[0], cast<cir::VectorType>(ops[1].getType()));
1609:     return emitX86Select(builder, getLoc(expr->getExprLoc()), ops[2], Res,
1610:                          ops[1]);
1611:   }
1612:   case X86::BI__builtin_ia32_pblendw128:
1613:   case X86::BI__builtin_ia32_blendpd:
1614:   case X86::BI__builtin_ia32_blendps:
1615:   case X86::BI__builtin_ia32_blendpd256:
1616:   case X86::BI__builtin_ia32_blendps256:
1617:   case X86::BI__builtin_ia32_pblendw256:
1618:   case X86::BI__builtin_ia32_pblendd128:
1619:   case X86::BI__builtin_ia32_pblendd256: {
1620:     uint32_t imm = getZExtIntValueFromConstOp(ops[2]);
1621:     unsigned numElts = cast<cir::VectorType>(ops[0].getType()).getSize();
1622: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1623-1630
```cpp
1623:     llvm::SmallVector<mlir::Attribute, 16> indices;
1624:     // If there are more than 8 elements, the immediate is used twice so make
1625:     // sure we handle that.
1626:     mlir::Type i32Ty = builder.getSInt32Ty();
1627:     for (unsigned i = 0; i != numElts; ++i)
1628:       indices.push_back(
1629:           cir::IntAttr::get(i32Ty, ((imm >> (i % 8)) & 0x1) ? numElts + i : i));
1630: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::IntAttr::get`. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::IntAttr::get`。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1631-1648
```cpp
1631:     return builder.createVecShuffle(getLoc(expr->getExprLoc()), ops[0], ops[1],
1632:                                     indices);
1633:   }
1634:   case X86::BI__builtin_ia32_pshuflw:
1635:   case X86::BI__builtin_ia32_pshuflw256:
1636:   case X86::BI__builtin_ia32_pshuflw512:
1637:     return emitPshufWord(builder, ops[0], ops[1], getLoc(expr->getExprLoc()),
1638:                          true);
1639:   case X86::BI__builtin_ia32_pshufhw:
1640:   case X86::BI__builtin_ia32_pshufhw256:
1641:   case X86::BI__builtin_ia32_pshufhw512:
1642:     return emitPshufWord(builder, ops[0], ops[1], getLoc(expr->getExprLoc()),
1643:                          false);
1644:   case X86::BI__builtin_ia32_pshufd:
1645:   case X86::BI__builtin_ia32_pshufd256:
1646:   case X86::BI__builtin_ia32_pshufd512:
1647:   case X86::BI__builtin_ia32_vpermilpd:
1648:   case X86::BI__builtin_ia32_vpermilps:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitPshufWord`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitPshufWord`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1649-1657
```cpp
1649:   case X86::BI__builtin_ia32_vpermilpd256:
1650:   case X86::BI__builtin_ia32_vpermilps256:
1651:   case X86::BI__builtin_ia32_vpermilpd512:
1652:   case X86::BI__builtin_ia32_vpermilps512: {
1653:     const uint32_t imm = getSExtIntValueFromConstOp(ops[1]);
1654: 
1655:     llvm::SmallVector<int64_t, 16> mask(16);
1656:     computeFullLaneShuffleMask(*this, ops[0], imm, false, mask);
1657: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `mask`, `computeFullLaneShuffleMask`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `mask`、`computeFullLaneShuffleMask`。

### Lines 1658-1670
```cpp
1658:     return builder.createVecShuffle(getLoc(expr->getExprLoc()), ops[0], mask);
1659:   }
1660:   case X86::BI__builtin_ia32_shufpd:
1661:   case X86::BI__builtin_ia32_shufpd256:
1662:   case X86::BI__builtin_ia32_shufpd512:
1663:   case X86::BI__builtin_ia32_shufps:
1664:   case X86::BI__builtin_ia32_shufps256:
1665:   case X86::BI__builtin_ia32_shufps512: {
1666:     const uint32_t imm = getZExtIntValueFromConstOp(ops[2]);
1667: 
1668:     llvm::SmallVector<int64_t, 16> mask(16);
1669:     computeFullLaneShuffleMask(*this, ops[0], imm, true, mask);
1670: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `mask`, `computeFullLaneShuffleMask`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `mask`、`computeFullLaneShuffleMask`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1671-1684
```cpp
1671:     return builder.createVecShuffle(getLoc(expr->getExprLoc()), ops[0], ops[1],
1672:                                     mask);
1673:   }
1674:   case X86::BI__builtin_ia32_permdi256:
1675:   case X86::BI__builtin_ia32_permdf256:
1676:   case X86::BI__builtin_ia32_permdi512:
1677:   case X86::BI__builtin_ia32_permdf512: {
1678:     unsigned imm =
1679:         ops[1].getDefiningOp<cir::ConstantOp>().getIntValue().getZExtValue();
1680:     unsigned numElts = cast<cir::VectorType>(ops[0].getType()).getSize();
1681: 
1682:     // These intrinsics operate on 256-bit lanes of four 64-bit elements.
1683:     int64_t Indices[8];
1684: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1685-1688
```cpp
1685:     for (unsigned l = 0; l != numElts; l += 4)
1686:       for (unsigned i = 0; i != 4; ++i)
1687:         Indices[l + i] = l + ((imm >> (2 * i)) & 0x3);
1688: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 1689-1699
```cpp
1689:     return builder.createVecShuffle(getLoc(expr->getExprLoc()), ops[0],
1690:                                     ArrayRef(Indices, numElts));
1691:   }
1692:   case X86::BI__builtin_ia32_palignr128:
1693:   case X86::BI__builtin_ia32_palignr256:
1694:   case X86::BI__builtin_ia32_palignr512: {
1695:     uint32_t shiftVal = getZExtIntValueFromConstOp(ops[2]) & 0xff;
1696: 
1697:     unsigned numElts = cast<cir::VectorType>(ops[0].getType()).getSize();
1698:     assert(numElts % 16 == 0);
1699: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ArrayRef`, `assert`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ArrayRef`、`assert`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1700-1705
```cpp
1700:     // If palignr is shifting the pair of vectors more than the size of two
1701:     // lanes, emit zero.
1702:     if (shiftVal >= 32)
1703:       return builder.getNullValue(convertType(expr->getType()),
1704:                                   getLoc(expr->getExprLoc()));
1705: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1706-1714
```cpp
1706:     // If palignr is shifting the pair of input vectors more than one lane,
1707:     // but less than two lanes, convert to shifting in zeroes.
1708:     if (shiftVal > 16) {
1709:       shiftVal -= 16;
1710:       ops[1] = ops[0];
1711:       ops[0] =
1712:           builder.getNullValue(ops[0].getType(), getLoc(expr->getExprLoc()));
1713:     }
1714: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 1715-1725
```cpp
1715:     int64_t indices[64];
1716:     // 256-bit palignr operates on 128-bit lanes so we need to handle that
1717:     for (unsigned l = 0; l != numElts; l += 16) {
1718:       for (unsigned i = 0; i != 16; ++i) {
1719:         uint32_t idx = shiftVal + i;
1720:         if (idx >= 16)
1721:           idx += numElts - 16; // End of lane, switch operand.
1722:         indices[l + i] = l + idx;
1723:       }
1724:     }
1725: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 1726-1742
```cpp
1726:     return builder.createVecShuffle(getLoc(expr->getExprLoc()), ops[1], ops[0],
1727:                                     ArrayRef(indices, numElts));
1728:   }
1729:   case X86::BI__builtin_ia32_alignd128:
1730:   case X86::BI__builtin_ia32_alignd256:
1731:   case X86::BI__builtin_ia32_alignd512:
1732:   case X86::BI__builtin_ia32_alignq128:
1733:   case X86::BI__builtin_ia32_alignq256:
1734:   case X86::BI__builtin_ia32_alignq512: {
1735:     unsigned numElts = cast<cir::VectorType>(ops[0].getType()).getSize();
1736:     unsigned shiftVal =
1737:         ops[2].getDefiningOp<cir::ConstantOp>().getIntValue().getZExtValue() &
1738:         0xff;
1739: 
1740:     // Mask the shift amount to width of a vector.
1741:     shiftVal &= numElts - 1;
1742: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ArrayRef`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ArrayRef`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1743-1747
```cpp
1743:     SmallVector<mlir::Attribute, 16> indices;
1744:     mlir::Type i32Ty = builder.getSInt32Ty();
1745:     for (unsigned i = 0; i != numElts; ++i)
1746:       indices.push_back(cir::IntAttr::get(i32Ty, i + shiftVal));
1747: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1748-1764
```cpp
1748:     return builder.createVecShuffle(getLoc(expr->getExprLoc()), ops[0], ops[1],
1749:                                     indices);
1750:   }
1751:   case X86::BI__builtin_ia32_shuf_f32x4_256:
1752:   case X86::BI__builtin_ia32_shuf_f64x2_256:
1753:   case X86::BI__builtin_ia32_shuf_i32x4_256:
1754:   case X86::BI__builtin_ia32_shuf_i64x2_256:
1755:   case X86::BI__builtin_ia32_shuf_f32x4:
1756:   case X86::BI__builtin_ia32_shuf_f64x2:
1757:   case X86::BI__builtin_ia32_shuf_i32x4:
1758:   case X86::BI__builtin_ia32_shuf_i64x2: {
1759:     mlir::Value src1 = ops[0];
1760:     mlir::Value src2 = ops[1];
1761: 
1762:     unsigned imm =
1763:         ops[2].getDefiningOp<cir::ConstantOp>().getIntValue().getZExtValue();
1764: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1765-1772
```cpp
1765:     unsigned numElems = cast<cir::VectorType>(src1.getType()).getSize();
1766:     unsigned totalBits = getContext().getTypeSize(expr->getArg(0)->getType());
1767:     unsigned numLanes = totalBits == 512 ? 4 : 2;
1768:     unsigned numElemsPerLane = numElems / numLanes;
1769: 
1770:     SmallVector<mlir::Attribute, 16> indices;
1771:     mlir::Type i32Ty = builder.getSInt32Ty();
1772: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1773-1782
```cpp
1773:     for (unsigned l = 0; l != numElems; l += numElemsPerLane) {
1774:       unsigned index = (imm % numLanes) * numElemsPerLane;
1775:       imm /= numLanes;
1776:       if (l >= (numElems / 2))
1777:         index += numElems;
1778:       for (unsigned i = 0; i != numElemsPerLane; ++i) {
1779:         indices.push_back(cir::IntAttr::get(i32Ty, index + i));
1780:       }
1781:     }
1782: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1783-1800
```cpp
1783:     return builder.createVecShuffle(getLoc(expr->getExprLoc()), src1, src2,
1784:                                     indices);
1785:   }
1786:   case X86::BI__builtin_ia32_vperm2f128_pd256:
1787:   case X86::BI__builtin_ia32_vperm2f128_ps256:
1788:   case X86::BI__builtin_ia32_vperm2f128_si256:
1789:   case X86::BI__builtin_ia32_permti256:
1790:   case X86::BI__builtin_ia32_pslldqi128_byteshift:
1791:   case X86::BI__builtin_ia32_pslldqi256_byteshift:
1792:   case X86::BI__builtin_ia32_pslldqi512_byteshift:
1793:   case X86::BI__builtin_ia32_psrldqi128_byteshift:
1794:   case X86::BI__builtin_ia32_psrldqi256_byteshift:
1795:   case X86::BI__builtin_ia32_psrldqi512_byteshift:
1796:     cgm.errorNYI(expr->getSourceRange(),
1797:                  std::string("unimplemented X86 builtin call: ") +
1798:                      getContext().BuiltinInfo.getName(builtinID));
1799:     return mlir::Value{};
1800:   case X86::BI__builtin_ia32_kshiftliqi:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1801-1814
```cpp
1801:   case X86::BI__builtin_ia32_kshiftlihi:
1802:   case X86::BI__builtin_ia32_kshiftlisi:
1803:   case X86::BI__builtin_ia32_kshiftlidi: {
1804:     mlir::Location loc = getLoc(expr->getExprLoc());
1805:     unsigned shiftVal =
1806:         ops[1].getDefiningOp<cir::ConstantOp>().getIntValue().getZExtValue() &
1807:         0xff;
1808:     unsigned numElems = cast<cir::IntType>(ops[0].getType()).getWidth();
1809: 
1810:     if (shiftVal >= numElems)
1811:       return builder.getNullValue(ops[0].getType(), loc);
1812: 
1813:     mlir::Value in = getMaskVecValue(builder, loc, ops[0], numElems);
1814: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1815-1819
```cpp
1815:     SmallVector<mlir::Attribute, 64> indices;
1816:     mlir::Type i32Ty = builder.getSInt32Ty();
1817:     for (auto i : llvm::seq<unsigned>(0, numElems))
1818:       indices.push_back(cir::IntAttr::get(i32Ty, numElems + i - shiftVal));
1819: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1820-1838
```cpp
1820:     mlir::Value zero = builder.getNullValue(in.getType(), loc);
1821:     mlir::Value sv = builder.createVecShuffle(loc, zero, in, indices);
1822:     return builder.createBitcast(sv, ops[0].getType());
1823:   }
1824:   case X86::BI__builtin_ia32_kshiftriqi:
1825:   case X86::BI__builtin_ia32_kshiftrihi:
1826:   case X86::BI__builtin_ia32_kshiftrisi:
1827:   case X86::BI__builtin_ia32_kshiftridi: {
1828:     mlir::Location loc = getLoc(expr->getExprLoc());
1829:     unsigned shiftVal =
1830:         ops[1].getDefiningOp<cir::ConstantOp>().getIntValue().getZExtValue() &
1831:         0xff;
1832:     unsigned numElems = cast<cir::IntType>(ops[0].getType()).getWidth();
1833: 
1834:     if (shiftVal >= numElems)
1835:       return builder.getNullValue(ops[0].getType(), loc);
1836: 
1837:     mlir::Value in = getMaskVecValue(builder, loc, ops[0], numElems);
1838: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1839-1843
```cpp
1839:     SmallVector<mlir::Attribute, 64> indices;
1840:     mlir::Type i32Ty = builder.getSInt32Ty();
1841:     for (auto i : llvm::seq<unsigned>(0, numElems))
1842:       indices.push_back(cir::IntAttr::get(i32Ty, i + shiftVal));
1843: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1844-1861
```cpp
1844:     mlir::Value zero = builder.getNullValue(in.getType(), loc);
1845:     mlir::Value sv = builder.createVecShuffle(loc, in, zero, indices);
1846:     return builder.createBitcast(sv, ops[0].getType());
1847:   }
1848:   case X86::BI__builtin_ia32_vprotbi:
1849:   case X86::BI__builtin_ia32_vprotwi:
1850:   case X86::BI__builtin_ia32_vprotdi:
1851:   case X86::BI__builtin_ia32_vprotqi:
1852:   case X86::BI__builtin_ia32_prold128:
1853:   case X86::BI__builtin_ia32_prold256:
1854:   case X86::BI__builtin_ia32_prold512:
1855:   case X86::BI__builtin_ia32_prolq128:
1856:   case X86::BI__builtin_ia32_prolq256:
1857:   case X86::BI__builtin_ia32_prolq512:
1858:     return emitX86FunnelShift(builder, getLoc(expr->getExprLoc()), ops[0],
1859:                               ops[0], ops[1], false);
1860:   case X86::BI__builtin_ia32_prord128:
1861:   case X86::BI__builtin_ia32_prord256:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86FunnelShift`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86FunnelShift`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1862-1879
```cpp
1862:   case X86::BI__builtin_ia32_prord512:
1863:   case X86::BI__builtin_ia32_prorq128:
1864:   case X86::BI__builtin_ia32_prorq256:
1865:   case X86::BI__builtin_ia32_prorq512:
1866:     return emitX86FunnelShift(builder, getLoc(expr->getExprLoc()), ops[0],
1867:                               ops[0], ops[1], true);
1868:   case X86::BI__builtin_ia32_selectb_128:
1869:   case X86::BI__builtin_ia32_selectb_256:
1870:   case X86::BI__builtin_ia32_selectb_512:
1871:   case X86::BI__builtin_ia32_selectw_128:
1872:   case X86::BI__builtin_ia32_selectw_256:
1873:   case X86::BI__builtin_ia32_selectw_512:
1874:   case X86::BI__builtin_ia32_selectd_128:
1875:   case X86::BI__builtin_ia32_selectd_256:
1876:   case X86::BI__builtin_ia32_selectd_512:
1877:   case X86::BI__builtin_ia32_selectq_128:
1878:   case X86::BI__builtin_ia32_selectq_256:
1879:   case X86::BI__builtin_ia32_selectq_512:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86FunnelShift`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86FunnelShift`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1880-1897
```cpp
1880:   case X86::BI__builtin_ia32_selectph_128:
1881:   case X86::BI__builtin_ia32_selectph_256:
1882:   case X86::BI__builtin_ia32_selectph_512:
1883:   case X86::BI__builtin_ia32_selectpbf_128:
1884:   case X86::BI__builtin_ia32_selectpbf_256:
1885:   case X86::BI__builtin_ia32_selectpbf_512:
1886:   case X86::BI__builtin_ia32_selectps_128:
1887:   case X86::BI__builtin_ia32_selectps_256:
1888:   case X86::BI__builtin_ia32_selectps_512:
1889:   case X86::BI__builtin_ia32_selectpd_128:
1890:   case X86::BI__builtin_ia32_selectpd_256:
1891:   case X86::BI__builtin_ia32_selectpd_512:
1892:     return emitX86Select(builder, getLoc(expr->getExprLoc()), ops[0], ops[1],
1893:                          ops[2]);
1894:   case X86::BI__builtin_ia32_selectsh_128:
1895:   case X86::BI__builtin_ia32_selectsbf_128:
1896:   case X86::BI__builtin_ia32_selectss_128:
1897:   case X86::BI__builtin_ia32_selectsd_128: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86Select`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86Select`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1898-1915
```cpp
1898:     mlir::Location loc = getLoc(expr->getExprLoc());
1899:     mlir::Value scalar1 =
1900:         builder.createExtractElement(loc, ops[1], uint64_t(0));
1901:     mlir::Value scalar2 =
1902:         builder.createExtractElement(loc, ops[2], uint64_t(0));
1903:     mlir::Value result =
1904:         emitX86ScalarSelect(builder, loc, ops[0], scalar1, scalar2);
1905:     return builder.createInsertElement(loc, ops[1], result, uint64_t(0));
1906:   }
1907:   case X86::BI__builtin_ia32_cmpb128_mask:
1908:   case X86::BI__builtin_ia32_cmpb256_mask:
1909:   case X86::BI__builtin_ia32_cmpb512_mask:
1910:   case X86::BI__builtin_ia32_cmpw128_mask:
1911:   case X86::BI__builtin_ia32_cmpw256_mask:
1912:   case X86::BI__builtin_ia32_cmpw512_mask:
1913:   case X86::BI__builtin_ia32_cmpd128_mask:
1914:   case X86::BI__builtin_ia32_cmpd256_mask:
1915:   case X86::BI__builtin_ia32_cmpd512_mask:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86ScalarSelect`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86ScalarSelect`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1916-1933
```cpp
1916:   case X86::BI__builtin_ia32_cmpq128_mask:
1917:   case X86::BI__builtin_ia32_cmpq256_mask:
1918:   case X86::BI__builtin_ia32_cmpq512_mask:
1919:   case X86::BI__builtin_ia32_ucmpb128_mask:
1920:   case X86::BI__builtin_ia32_ucmpb256_mask:
1921:   case X86::BI__builtin_ia32_ucmpb512_mask:
1922:   case X86::BI__builtin_ia32_ucmpw128_mask:
1923:   case X86::BI__builtin_ia32_ucmpw256_mask:
1924:   case X86::BI__builtin_ia32_ucmpw512_mask:
1925:   case X86::BI__builtin_ia32_ucmpd128_mask:
1926:   case X86::BI__builtin_ia32_ucmpd256_mask:
1927:   case X86::BI__builtin_ia32_ucmpd512_mask:
1928:   case X86::BI__builtin_ia32_ucmpq128_mask:
1929:   case X86::BI__builtin_ia32_ucmpq256_mask:
1930:   case X86::BI__builtin_ia32_ucmpq512_mask: {
1931:     int64_t cc = CIRGenFunction::getZExtIntValueFromConstOp(ops[2]) & 0x7;
1932:     return emitX86MaskedCompare(builder, cc, 1, ops,
1933:                                 getLoc(expr->getExprLoc()));
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1934-1951
```cpp
1934:   }
1935:   case X86::BI__builtin_ia32_vpcomb:
1936:   case X86::BI__builtin_ia32_vpcomw:
1937:   case X86::BI__builtin_ia32_vpcomd:
1938:   case X86::BI__builtin_ia32_vpcomq:
1939:     return emitX86vpcom(builder, getLoc(expr->getExprLoc()), ops, true);
1940:   case X86::BI__builtin_ia32_vpcomub:
1941:   case X86::BI__builtin_ia32_vpcomuw:
1942:   case X86::BI__builtin_ia32_vpcomud:
1943:   case X86::BI__builtin_ia32_vpcomuq:
1944:     return emitX86vpcom(builder, getLoc(expr->getExprLoc()), ops, false);
1945:   case X86::BI__builtin_ia32_kortestcqi:
1946:   case X86::BI__builtin_ia32_kortestchi:
1947:   case X86::BI__builtin_ia32_kortestcsi:
1948:   case X86::BI__builtin_ia32_kortestcdi: {
1949:     mlir::Location loc = getLoc(expr->getExprLoc());
1950:     cir::IntType ty = cast<cir::IntType>(ops[0].getType());
1951:     mlir::Value allOnesOp =
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86vpcom`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86vpcom`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1952-1969
```cpp
1952:         builder.getConstAPInt(loc, ty, APInt::getAllOnes(ty.getWidth()));
1953:     mlir::Value orOp = emitX86MaskLogic<cir::OrOp>(builder, loc, ops);
1954:     mlir::Value cmp =
1955:         cir::CmpOp::create(builder, loc, cir::CmpOpKind::eq, orOp, allOnesOp);
1956:     return builder.createCast(cir::CastKind::bool_to_int, cmp,
1957:                               cgm.convertType(expr->getType()));
1958:   }
1959:   case X86::BI__builtin_ia32_kortestzqi:
1960:   case X86::BI__builtin_ia32_kortestzhi:
1961:   case X86::BI__builtin_ia32_kortestzsi:
1962:   case X86::BI__builtin_ia32_kortestzdi: {
1963:     mlir::Location loc = getLoc(expr->getExprLoc());
1964:     cir::IntType ty = cast<cir::IntType>(ops[0].getType());
1965:     mlir::Value allZerosOp = builder.getNullValue(ty, loc).getResult();
1966:     mlir::Value orOp = emitX86MaskLogic<cir::OrOp>(builder, loc, ops);
1967:     mlir::Value cmp =
1968:         cir::CmpOp::create(builder, loc, cir::CmpOpKind::eq, orOp, allZerosOp);
1969:     return builder.createCast(cir::CastKind::bool_to_int, cmp,
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::CmpOp::create`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::CmpOp::create`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1970-1987
```cpp
1970:                               cgm.convertType(expr->getType()));
1971:   }
1972:   case X86::BI__builtin_ia32_ktestcqi:
1973:     return emitX86MaskTest(builder, getLoc(expr->getExprLoc()),
1974:                            "x86.avx512.ktestc.b", ops);
1975:   case X86::BI__builtin_ia32_ktestzqi:
1976:     return emitX86MaskTest(builder, getLoc(expr->getExprLoc()),
1977:                            "x86.avx512.ktestz.b", ops);
1978:   case X86::BI__builtin_ia32_ktestchi:
1979:     return emitX86MaskTest(builder, getLoc(expr->getExprLoc()),
1980:                            "x86.avx512.ktestc.w", ops);
1981:   case X86::BI__builtin_ia32_ktestzhi:
1982:     return emitX86MaskTest(builder, getLoc(expr->getExprLoc()),
1983:                            "x86.avx512.ktestz.w", ops);
1984:   case X86::BI__builtin_ia32_ktestcsi:
1985:     return emitX86MaskTest(builder, getLoc(expr->getExprLoc()),
1986:                            "x86.avx512.ktestc.d", ops);
1987:   case X86::BI__builtin_ia32_ktestzsi:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskTest`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskTest`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1988-2005
```cpp
1988:     return emitX86MaskTest(builder, getLoc(expr->getExprLoc()),
1989:                            "x86.avx512.ktestz.d", ops);
1990:   case X86::BI__builtin_ia32_ktestcdi:
1991:     return emitX86MaskTest(builder, getLoc(expr->getExprLoc()),
1992:                            "x86.avx512.ktestc.q", ops);
1993:   case X86::BI__builtin_ia32_ktestzdi:
1994:     return emitX86MaskTest(builder, getLoc(expr->getExprLoc()),
1995:                            "x86.avx512.ktestz.q", ops);
1996:   case X86::BI__builtin_ia32_kaddqi:
1997:     return emitX86MaskAddLogic(builder, getLoc(expr->getExprLoc()),
1998:                                "x86.avx512.kadd.b", ops);
1999:   case X86::BI__builtin_ia32_kaddhi:
2000:     return emitX86MaskAddLogic(builder, getLoc(expr->getExprLoc()),
2001:                                "x86.avx512.kadd.w", ops);
2002:   case X86::BI__builtin_ia32_kaddsi:
2003:     return emitX86MaskAddLogic(builder, getLoc(expr->getExprLoc()),
2004:                                "x86.avx512.kadd.d", ops);
2005:   case X86::BI__builtin_ia32_kadddi:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86MaskTest`, `emitX86MaskAddLogic`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86MaskTest`、`emitX86MaskAddLogic`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2006-2023
```cpp
2006:     return emitX86MaskAddLogic(builder, getLoc(expr->getExprLoc()),
2007:                                "x86.avx512.kadd.q", ops);
2008:   case X86::BI__builtin_ia32_kandqi:
2009:   case X86::BI__builtin_ia32_kandhi:
2010:   case X86::BI__builtin_ia32_kandsi:
2011:   case X86::BI__builtin_ia32_kanddi:
2012:     return emitX86MaskLogic<cir::AndOp>(builder, getLoc(expr->getExprLoc()),
2013:                                         ops);
2014:   case X86::BI__builtin_ia32_kandnqi:
2015:   case X86::BI__builtin_ia32_kandnhi:
2016:   case X86::BI__builtin_ia32_kandnsi:
2017:   case X86::BI__builtin_ia32_kandndi:
2018:     return emitX86MaskLogic<cir::AndOp>(builder, getLoc(expr->getExprLoc()),
2019:                                         ops, /*invertLHS=*/true);
2020:   case X86::BI__builtin_ia32_korqi:
2021:   case X86::BI__builtin_ia32_korhi:
2022:   case X86::BI__builtin_ia32_korsi:
2023:   case X86::BI__builtin_ia32_kordi:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2024-2041
```cpp
2024:     return emitX86MaskLogic<cir::OrOp>(builder, getLoc(expr->getExprLoc()),
2025:                                        ops);
2026:   case X86::BI__builtin_ia32_kxnorqi:
2027:   case X86::BI__builtin_ia32_kxnorhi:
2028:   case X86::BI__builtin_ia32_kxnorsi:
2029:   case X86::BI__builtin_ia32_kxnordi:
2030:     return emitX86MaskLogic<cir::XorOp>(builder, getLoc(expr->getExprLoc()),
2031:                                         ops, /*invertLHS=*/true);
2032:   case X86::BI__builtin_ia32_kxorqi:
2033:   case X86::BI__builtin_ia32_kxorhi:
2034:   case X86::BI__builtin_ia32_kxorsi:
2035:   case X86::BI__builtin_ia32_kxordi:
2036:     return emitX86MaskLogic<cir::XorOp>(builder, getLoc(expr->getExprLoc()),
2037:                                         ops);
2038:   case X86::BI__builtin_ia32_knotqi:
2039:   case X86::BI__builtin_ia32_knothi:
2040:   case X86::BI__builtin_ia32_knotsi:
2041:   case X86::BI__builtin_ia32_knotdi: {
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2042-2059
```cpp
2042:     cir::IntType intTy = cast<cir::IntType>(ops[0].getType());
2043:     unsigned numElts = intTy.getWidth();
2044:     mlir::Value resVec =
2045:         getMaskVecValue(builder, getLoc(expr->getExprLoc()), ops[0], numElts);
2046:     return builder.createBitcast(builder.createNot(resVec), ops[0].getType());
2047:   }
2048:   case X86::BI__builtin_ia32_kmovb:
2049:   case X86::BI__builtin_ia32_kmovw:
2050:   case X86::BI__builtin_ia32_kmovd:
2051:   case X86::BI__builtin_ia32_kmovq: {
2052:     // Bitcast to vXi1 type and then back to integer. This gets the mask
2053:     // register type into the IR, but might be optimized out depending on
2054:     // what's around it.
2055:     cir::IntType intTy = cast<cir::IntType>(ops[0].getType());
2056:     unsigned numElts = intTy.getWidth();
2057:     mlir::Value resVec =
2058:         getMaskVecValue(builder, getLoc(expr->getExprLoc()), ops[0], numElts);
2059:     return builder.createBitcast(resVec, ops[0].getType());
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getMaskVecValue`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getMaskVecValue`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2060-2077
```cpp
2060:   }
2061:   case X86::BI__builtin_ia32_sqrtsh_round_mask:
2062:   case X86::BI__builtin_ia32_sqrtsd_round_mask:
2063:   case X86::BI__builtin_ia32_sqrtss_round_mask:
2064:     cgm.errorNYI(expr->getSourceRange(),
2065:                  std::string("unimplemented X86 builtin call: ") +
2066:                      getContext().BuiltinInfo.getName(builtinID));
2067:     return mlir::Value{};
2068:   case X86::BI__builtin_ia32_sqrtph512:
2069:   case X86::BI__builtin_ia32_sqrtps512:
2070:   case X86::BI__builtin_ia32_sqrtpd512: {
2071:     mlir::Location loc = getLoc(expr->getExprLoc());
2072:     mlir::Value arg = ops[0];
2073:     return cir::SqrtOp::create(builder, loc, arg.getType(), arg).getResult();
2074:   }
2075:   case X86::BI__builtin_ia32_pmuludq128:
2076:   case X86::BI__builtin_ia32_pmuludq256:
2077:   case X86::BI__builtin_ia32_pmuludq512: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2078-2095
```cpp
2078:     unsigned opTypePrimitiveSizeInBits =
2079:         cgm.getDataLayout().getTypeSizeInBits(ops[0].getType());
2080:     return emitX86Muldq(builder, getLoc(expr->getExprLoc()), /*isSigned*/ false,
2081:                         ops, opTypePrimitiveSizeInBits);
2082:   }
2083:   case X86::BI__builtin_ia32_pmuldq128:
2084:   case X86::BI__builtin_ia32_pmuldq256:
2085:   case X86::BI__builtin_ia32_pmuldq512: {
2086:     unsigned opTypePrimitiveSizeInBits =
2087:         cgm.getDataLayout().getTypeSizeInBits(ops[0].getType());
2088:     return emitX86Muldq(builder, getLoc(expr->getExprLoc()), /*isSigned*/ true,
2089:                         ops, opTypePrimitiveSizeInBits);
2090:   }
2091:   case X86::BI__builtin_ia32_pternlogd512_mask:
2092:   case X86::BI__builtin_ia32_pternlogq512_mask:
2093:   case X86::BI__builtin_ia32_pternlogd128_mask:
2094:   case X86::BI__builtin_ia32_pternlogd256_mask:
2095:   case X86::BI__builtin_ia32_pternlogq128_mask:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2096-2113
```cpp
2096:   case X86::BI__builtin_ia32_pternlogq256_mask:
2097:   case X86::BI__builtin_ia32_pternlogd512_maskz:
2098:   case X86::BI__builtin_ia32_pternlogq512_maskz:
2099:   case X86::BI__builtin_ia32_pternlogd128_maskz:
2100:   case X86::BI__builtin_ia32_pternlogd256_maskz:
2101:   case X86::BI__builtin_ia32_pternlogq128_maskz:
2102:   case X86::BI__builtin_ia32_pternlogq256_maskz:
2103:     cgm.errorNYI(expr->getSourceRange(),
2104:                  std::string("unimplemented X86 builtin call: ") +
2105:                      getContext().BuiltinInfo.getName(builtinID));
2106:     return mlir::Value{};
2107:   case X86::BI__builtin_ia32_vpshldd128:
2108:   case X86::BI__builtin_ia32_vpshldd256:
2109:   case X86::BI__builtin_ia32_vpshldd512:
2110:   case X86::BI__builtin_ia32_vpshldq128:
2111:   case X86::BI__builtin_ia32_vpshldq256:
2112:   case X86::BI__builtin_ia32_vpshldq512:
2113:   case X86::BI__builtin_ia32_vpshldw128:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2114-2131
```cpp
2114:   case X86::BI__builtin_ia32_vpshldw256:
2115:   case X86::BI__builtin_ia32_vpshldw512:
2116:     return emitX86FunnelShift(builder, getLoc(expr->getExprLoc()), ops[0],
2117:                               ops[1], ops[2], false);
2118:   case X86::BI__builtin_ia32_vpshrdd128:
2119:   case X86::BI__builtin_ia32_vpshrdd256:
2120:   case X86::BI__builtin_ia32_vpshrdd512:
2121:   case X86::BI__builtin_ia32_vpshrdq128:
2122:   case X86::BI__builtin_ia32_vpshrdq256:
2123:   case X86::BI__builtin_ia32_vpshrdq512:
2124:   case X86::BI__builtin_ia32_vpshrdw128:
2125:   case X86::BI__builtin_ia32_vpshrdw256:
2126:   case X86::BI__builtin_ia32_vpshrdw512:
2127:     // Ops 0 and 1 are swapped.
2128:     return emitX86FunnelShift(builder, getLoc(expr->getExprLoc()), ops[1],
2129:                               ops[0], ops[2], true);
2130:   case X86::BI__builtin_ia32_reduce_fadd_pd512:
2131:   case X86::BI__builtin_ia32_reduce_fadd_ps512:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86FunnelShift`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86FunnelShift`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2132-2149
```cpp
2132:   case X86::BI__builtin_ia32_reduce_fadd_ph512:
2133:   case X86::BI__builtin_ia32_reduce_fadd_ph256:
2134:   case X86::BI__builtin_ia32_reduce_fadd_ph128: {
2135:     assert(!cir::MissingFeatures::fastMathFlags());
2136:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()),
2137:                                        "vector.reduce.fadd", ops[0].getType(),
2138:                                        mlir::ValueRange{ops[0], ops[1]});
2139:   }
2140:   case X86::BI__builtin_ia32_reduce_fmul_pd512:
2141:   case X86::BI__builtin_ia32_reduce_fmul_ps512:
2142:   case X86::BI__builtin_ia32_reduce_fmul_ph512:
2143:   case X86::BI__builtin_ia32_reduce_fmul_ph256:
2144:   case X86::BI__builtin_ia32_reduce_fmul_ph128: {
2145:     assert(!cir::MissingFeatures::fastMathFlags());
2146:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()),
2147:                                        "vector.reduce.fmul", ops[0].getType(),
2148:                                        mlir::ValueRange{ops[0], ops[1]});
2149:   }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2150-2167
```cpp
2150:   case X86::BI__builtin_ia32_reduce_fmax_pd512:
2151:   case X86::BI__builtin_ia32_reduce_fmax_ps512:
2152:   case X86::BI__builtin_ia32_reduce_fmax_ph512:
2153:   case X86::BI__builtin_ia32_reduce_fmax_ph256:
2154:   case X86::BI__builtin_ia32_reduce_fmax_ph128: {
2155:     assert(!cir::MissingFeatures::fastMathFlags());
2156:     cir::VectorType vecTy = cast<cir::VectorType>(ops[0].getType());
2157:     return builder.emitIntrinsicCallOp(
2158:         getLoc(expr->getExprLoc()), "vector.reduce.fmax",
2159:         vecTy.getElementType(), mlir::ValueRange{ops[0]});
2160:   }
2161:   case X86::BI__builtin_ia32_reduce_fmin_pd512:
2162:   case X86::BI__builtin_ia32_reduce_fmin_ps512:
2163:   case X86::BI__builtin_ia32_reduce_fmin_ph512:
2164:   case X86::BI__builtin_ia32_reduce_fmin_ph256:
2165:   case X86::BI__builtin_ia32_reduce_fmin_ph128: {
2166:     assert(!cir::MissingFeatures::fastMathFlags());
2167:     cir::VectorType vecTy = cast<cir::VectorType>(ops[0].getType());
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2168-2185
```cpp
2168:     return builder.emitIntrinsicCallOp(
2169:         getLoc(expr->getExprLoc()), "vector.reduce.fmin",
2170:         vecTy.getElementType(), mlir::ValueRange{ops[0]});
2171:   }
2172:   case X86::BI__builtin_ia32_rdrand16_step:
2173:   case X86::BI__builtin_ia32_rdrand32_step:
2174:   case X86::BI__builtin_ia32_rdrand64_step:
2175:   case X86::BI__builtin_ia32_rdseed16_step:
2176:   case X86::BI__builtin_ia32_rdseed32_step:
2177:   case X86::BI__builtin_ia32_rdseed64_step: {
2178:     llvm::StringRef intrinsicName;
2179:     switch (builtinID) {
2180:     default:
2181:       llvm_unreachable("Unsupported intrinsic!");
2182:     case X86::BI__builtin_ia32_rdrand16_step:
2183:       intrinsicName = "x86.rdrand.16";
2184:       break;
2185:     case X86::BI__builtin_ia32_rdrand32_step:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2186-2201
```cpp
2186:       intrinsicName = "x86.rdrand.32";
2187:       break;
2188:     case X86::BI__builtin_ia32_rdrand64_step:
2189:       intrinsicName = "x86.rdrand.64";
2190:       break;
2191:     case X86::BI__builtin_ia32_rdseed16_step:
2192:       intrinsicName = "x86.rdseed.16";
2193:       break;
2194:     case X86::BI__builtin_ia32_rdseed32_step:
2195:       intrinsicName = "x86.rdseed.32";
2196:       break;
2197:     case X86::BI__builtin_ia32_rdseed64_step:
2198:       intrinsicName = "x86.rdseed.64";
2199:       break;
2200:     }
2201: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2202-2209
```cpp
2202:     mlir::Location loc = getLoc(expr->getExprLoc());
2203:     mlir::Type randTy = cast<cir::PointerType>(ops[0].getType()).getPointee();
2204:     llvm::SmallVector<mlir::Type, 2> resultTypes = {randTy,
2205:                                                     builder.getUInt32Ty()};
2206:     cir::RecordType resRecord =
2207:         cir::RecordType::get(&getMLIRContext(), resultTypes, false, false,
2208:                              cir::RecordType::RecordKind::Struct);
2209: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::RecordType::get`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::RecordType::get`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2210-2215
```cpp
2210:     mlir::Value call =
2211:         builder.emitIntrinsicCallOp(loc, intrinsicName, resRecord);
2212:     mlir::Value rand =
2213:         cir::ExtractMemberOp::create(builder, loc, randTy, call, 0);
2214:     builder.CIRBaseBuilderTy::createStore(loc, rand, ops[0]);
2215: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::ExtractMemberOp::create`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::ExtractMemberOp::create`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2216-2233
```cpp
2216:     return cir::ExtractMemberOp::create(builder, loc, builder.getUInt32Ty(),
2217:                                         call, 1);
2218:   }
2219:   case X86::BI__builtin_ia32_addcarryx_u32:
2220:   case X86::BI__builtin_ia32_addcarryx_u64:
2221:   case X86::BI__builtin_ia32_subborrow_u32:
2222:   case X86::BI__builtin_ia32_subborrow_u64:
2223:     cgm.errorNYI(expr->getSourceRange(),
2224:                  std::string("unimplemented X86 builtin call: ") +
2225:                      getContext().BuiltinInfo.getName(builtinID));
2226:     return mlir::Value{};
2227:   case X86::BI__builtin_ia32_fpclassps128_mask:
2228:   case X86::BI__builtin_ia32_fpclassps256_mask:
2229:   case X86::BI__builtin_ia32_fpclassps512_mask:
2230:   case X86::BI__builtin_ia32_vfpclassbf16128_mask:
2231:   case X86::BI__builtin_ia32_vfpclassbf16256_mask:
2232:   case X86::BI__builtin_ia32_vfpclassbf16512_mask:
2233:   case X86::BI__builtin_ia32_fpclassph128_mask:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2234-2249
```cpp
2234:   case X86::BI__builtin_ia32_fpclassph256_mask:
2235:   case X86::BI__builtin_ia32_fpclassph512_mask:
2236:   case X86::BI__builtin_ia32_fpclasspd128_mask:
2237:   case X86::BI__builtin_ia32_fpclasspd256_mask:
2238:   case X86::BI__builtin_ia32_fpclasspd512_mask:
2239:     return emitX86Fpclass(builder, getLoc(expr->getExprLoc()), builtinID, ops);
2240:   case X86::BI__builtin_ia32_vp2intersect_q_512:
2241:   case X86::BI__builtin_ia32_vp2intersect_q_256:
2242:   case X86::BI__builtin_ia32_vp2intersect_q_128:
2243:   case X86::BI__builtin_ia32_vp2intersect_d_512:
2244:   case X86::BI__builtin_ia32_vp2intersect_d_256:
2245:   case X86::BI__builtin_ia32_vp2intersect_d_128: {
2246:     unsigned numElts = cast<cir::VectorType>(ops[0].getType()).getSize();
2247:     mlir::Location loc = getLoc(expr->getExprLoc());
2248:     StringRef intrinsicName;
2249: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitX86Fpclass`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitX86Fpclass`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2250-2267
```cpp
2250:     switch (builtinID) {
2251:     default:
2252:       llvm_unreachable("Unexpected builtin");
2253:     case X86::BI__builtin_ia32_vp2intersect_q_512:
2254:       intrinsicName = "x86.avx512.vp2intersect.q.512";
2255:       break;
2256:     case X86::BI__builtin_ia32_vp2intersect_q_256:
2257:       intrinsicName = "x86.avx512.vp2intersect.q.256";
2258:       break;
2259:     case X86::BI__builtin_ia32_vp2intersect_q_128:
2260:       intrinsicName = "x86.avx512.vp2intersect.q.128";
2261:       break;
2262:     case X86::BI__builtin_ia32_vp2intersect_d_512:
2263:       intrinsicName = "x86.avx512.vp2intersect.d.512";
2264:       break;
2265:     case X86::BI__builtin_ia32_vp2intersect_d_256:
2266:       intrinsicName = "x86.avx512.vp2intersect.d.256";
2267:       break;
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 2268-2274
```cpp
2268:     case X86::BI__builtin_ia32_vp2intersect_d_128:
2269:       intrinsicName = "x86.avx512.vp2intersect.d.128";
2270:       break;
2271:     }
2272: 
2273:     auto resVector = cir::VectorType::get(builder.getBoolTy(), numElts);
2274: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2275-2278
```cpp
2275:     cir::RecordType resRecord =
2276:         cir::RecordType::get(&getMLIRContext(), {resVector, resVector}, false,
2277:                              false, cir::RecordType::RecordKind::Struct);
2278: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2279-2288
```cpp
2279:     mlir::Value call = builder.emitIntrinsicCallOp(
2280:         getLoc(expr->getExprLoc()), intrinsicName, resRecord,
2281:         mlir::ValueRange{ops[0], ops[1]});
2282:     mlir::Value result =
2283:         cir::ExtractMemberOp::create(builder, loc, resVector, call, 0);
2284:     result = emitX86MaskedCompareResult(builder, result, numElts, nullptr, loc);
2285:     Address addr = Address(
2286:         ops[2], clang::CharUnits::fromQuantity(std::max(1U, numElts / 8)));
2287:     builder.createStore(loc, result, addr);
2288: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::ExtractMemberOp::create`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::ExtractMemberOp::create`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2289-2306
```cpp
2289:     result = cir::ExtractMemberOp::create(builder, loc, resVector, call, 1);
2290:     result = emitX86MaskedCompareResult(builder, result, numElts, nullptr, loc);
2291:     addr = Address(ops[3],
2292:                    clang::CharUnits::fromQuantity(std::max(1U, numElts / 8)));
2293:     builder.createStore(loc, result, addr);
2294:     return mlir::Value{};
2295:   }
2296:   case X86::BI__builtin_ia32_vpmultishiftqb128:
2297:   case X86::BI__builtin_ia32_vpmultishiftqb256:
2298:   case X86::BI__builtin_ia32_vpmultishiftqb512:
2299:   case X86::BI__builtin_ia32_vpshufbitqmb128_mask:
2300:   case X86::BI__builtin_ia32_vpshufbitqmb256_mask:
2301:   case X86::BI__builtin_ia32_vpshufbitqmb512_mask:
2302:   case X86::BI__builtin_ia32_cmpeqps:
2303:   case X86::BI__builtin_ia32_cmpeqpd:
2304:     return emitVectorFCmp(*this, *expr, ops, cir::CmpOpKind::eq,
2305:                           /*shouldInvert=*/false);
2306:   case X86::BI__builtin_ia32_cmpltps:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `clang::CharUnits::fromQuantity`, `emitVectorFCmp`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `clang::CharUnits::fromQuantity`、`emitVectorFCmp`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2307-2324
```cpp
2307:   case X86::BI__builtin_ia32_cmpltpd:
2308:     return emitVectorFCmp(*this, *expr, ops, cir::CmpOpKind::lt,
2309:                           /*shouldInvert=*/false);
2310:   case X86::BI__builtin_ia32_cmpleps:
2311:   case X86::BI__builtin_ia32_cmplepd:
2312:     return emitVectorFCmp(*this, *expr, ops, cir::CmpOpKind::le,
2313:                           /*shouldInvert=*/false);
2314:   case X86::BI__builtin_ia32_cmpunordps:
2315:   case X86::BI__builtin_ia32_cmpunordpd:
2316:     return emitVectorFCmp(*this, *expr, ops, cir::CmpOpKind::uno,
2317:                           /*shouldInvert=*/false);
2318:   case X86::BI__builtin_ia32_cmpneqps:
2319:   case X86::BI__builtin_ia32_cmpneqpd:
2320:     return emitVectorFCmp(*this, *expr, ops, cir::CmpOpKind::ne,
2321:                           /*shouldInvert=*/false);
2322:   case X86::BI__builtin_ia32_cmpnltps:
2323:   case X86::BI__builtin_ia32_cmpnltpd:
2324:     return emitVectorFCmp(*this, *expr, ops, cir::CmpOpKind::lt,
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitVectorFCmp`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitVectorFCmp`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2325-2342
```cpp
2325:                           /*shouldInvert=*/true);
2326:   case X86::BI__builtin_ia32_cmpnleps:
2327:   case X86::BI__builtin_ia32_cmpnlepd:
2328:     return emitVectorFCmp(*this, *expr, ops, cir::CmpOpKind::le,
2329:                           /*shouldInvert=*/true);
2330:   case X86::BI__builtin_ia32_cmpordps:
2331:   case X86::BI__builtin_ia32_cmpordpd:
2332:     return emitVectorFCmp(*this, *expr, ops, cir::CmpOpKind::uno,
2333:                           /*shouldInvert=*/true);
2334:   case X86::BI__builtin_ia32_cmpph128_mask:
2335:   case X86::BI__builtin_ia32_cmpph256_mask:
2336:   case X86::BI__builtin_ia32_cmpph512_mask:
2337:   case X86::BI__builtin_ia32_cmpps128_mask:
2338:   case X86::BI__builtin_ia32_cmpps256_mask:
2339:   case X86::BI__builtin_ia32_cmpps512_mask:
2340:   case X86::BI__builtin_ia32_cmppd128_mask:
2341:   case X86::BI__builtin_ia32_cmppd256_mask:
2342:   case X86::BI__builtin_ia32_cmppd512_mask:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitVectorFCmp`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitVectorFCmp`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2343-2360
```cpp
2343:   case X86::BI__builtin_ia32_vcmpbf16512_mask:
2344:   case X86::BI__builtin_ia32_vcmpbf16256_mask:
2345:   case X86::BI__builtin_ia32_vcmpbf16128_mask:
2346:   case X86::BI__builtin_ia32_cmpps:
2347:   case X86::BI__builtin_ia32_cmpps256:
2348:   case X86::BI__builtin_ia32_cmppd:
2349:   case X86::BI__builtin_ia32_cmppd256:
2350:   case X86::BI__builtin_ia32_cmpeqss:
2351:   case X86::BI__builtin_ia32_cmpltss:
2352:   case X86::BI__builtin_ia32_cmpless:
2353:   case X86::BI__builtin_ia32_cmpunordss:
2354:   case X86::BI__builtin_ia32_cmpneqss:
2355:   case X86::BI__builtin_ia32_cmpnltss:
2356:   case X86::BI__builtin_ia32_cmpnless:
2357:   case X86::BI__builtin_ia32_cmpordss:
2358:   case X86::BI__builtin_ia32_cmpeqsd:
2359:   case X86::BI__builtin_ia32_cmpltsd:
2360:   case X86::BI__builtin_ia32_cmplesd:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2361-2383
```cpp
2361:   case X86::BI__builtin_ia32_cmpunordsd:
2362:   case X86::BI__builtin_ia32_cmpneqsd:
2363:   case X86::BI__builtin_ia32_cmpnltsd:
2364:   case X86::BI__builtin_ia32_cmpnlesd:
2365:   case X86::BI__builtin_ia32_cmpordsd:
2366:     cgm.errorNYI(expr->getSourceRange(),
2367:                  std::string("unimplemented X86 builtin call: ") +
2368:                      getContext().BuiltinInfo.getName(builtinID));
2369:     return {};
2370:   case X86::BI__builtin_ia32_vcvtph2ps_mask:
2371:   case X86::BI__builtin_ia32_vcvtph2ps256_mask:
2372:   case X86::BI__builtin_ia32_vcvtph2ps512_mask: {
2373:     mlir::Location loc = getLoc(expr->getExprLoc());
2374:     return emitX86CvtF16ToFloatExpr(builder, loc, ops,
2375:                                     convertType(expr->getType()));
2376:   }
2377:   case X86::BI__builtin_ia32_cvtneps2bf16_128_mask: {
2378:     mlir::Location loc = getLoc(expr->getExprLoc());
2379:     cir::VectorType resTy = cast<cir::VectorType>(convertType(expr->getType()));
2380: 
2381:     cir::VectorType inputTy = cast<cir::VectorType>(ops[0].getType());
2382:     unsigned numElts = inputTy.getSize();
2383: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2384-2385
```cpp
2384:     mlir::Value mask = getMaskVecValue(builder, loc, ops[2], numElts);
2385: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2386-2390
```cpp
2386:     SmallVector<mlir::Value, 3> args;
2387:     args.push_back(ops[0]);
2388:     args.push_back(ops[1]);
2389:     args.push_back(mask);
2390: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2391-2408
```cpp
2391:     return builder.emitIntrinsicCallOp(
2392:         loc, "x86.avx512bf16.mask.cvtneps2bf16.128", resTy, args);
2393:   }
2394:   case X86::BI__builtin_ia32_cvtneps2bf16_256_mask:
2395:   case X86::BI__builtin_ia32_cvtneps2bf16_512_mask: {
2396:     mlir::Location loc = getLoc(expr->getExprLoc());
2397:     cir::VectorType resTy = cast<cir::VectorType>(convertType(expr->getType()));
2398:     StringRef intrinsicName;
2399:     if (builtinID == X86::BI__builtin_ia32_cvtneps2bf16_256_mask) {
2400:       intrinsicName = "x86.avx512bf16.cvtneps2bf16.256";
2401:     } else {
2402:       assert(builtinID == X86::BI__builtin_ia32_cvtneps2bf16_512_mask);
2403:       intrinsicName = "x86.avx512bf16.cvtneps2bf16.512";
2404:     }
2405: 
2406:     mlir::Value res = builder.emitIntrinsicCallOp(loc, intrinsicName, resTy,
2407:                                                   mlir::ValueRange{ops[0]});
2408: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2409-2426
```cpp
2409:     return emitX86Select(builder, loc, ops[2], res, ops[1]);
2410:   }
2411:   case X86::BI__cpuid:
2412:   case X86::BI__cpuidex: {
2413:     mlir::Location loc = getLoc(expr->getExprLoc());
2414:     mlir::Value subFuncId = builtinID == X86::BI__cpuidex
2415:                                 ? ops[2]
2416:                                 : builder.getConstInt(loc, sInt32Ty, 0);
2417:     cir::CpuIdOp::create(builder, loc, /*cpuInfo=*/ops[0],
2418:                          /*functionId=*/ops[1], /*subFunctionId=*/subFuncId);
2419:     return mlir::Value{};
2420:   }
2421:   case X86::BI__emul:
2422:   case X86::BI__emulu:
2423:   case X86::BI__mulh:
2424:   case X86::BI__umulh:
2425:   case X86::BI_mul128:
2426:   case X86::BI_umul128: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::CpuIdOp::create`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::CpuIdOp::create`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2427-2446
```cpp
2427:     cgm.errorNYI(expr->getSourceRange(),
2428:                  std::string("unimplemented X86 builtin call: ") +
2429:                      getContext().BuiltinInfo.getName(builtinID));
2430:     return mlir::Value{};
2431:   }
2432:   case X86::BI__faststorefence: {
2433:     cir::AtomicFenceOp::create(
2434:         builder, getLoc(expr->getExprLoc()),
2435:         cir::MemOrder::SequentiallyConsistent,
2436:         cir::SyncScopeKindAttr::get(&getMLIRContext(),
2437:                                     cir::SyncScopeKind::System));
2438:     return mlir::Value{};
2439:   }
2440:   case X86::BI__shiftleft128:
2441:   case X86::BI__shiftright128: {
2442:     // Flip low/high ops and zero-extend amount to matching type.
2443:     // shiftleft128(Low, High, Amt) -> fshl(High, Low, Amt)
2444:     // shiftright128(Low, High, Amt) -> fshr(High, Low, Amt)
2445:     std::swap(ops[0], ops[1]);
2446: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `cir::AtomicFenceOp::create`, `std::swap`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`cir::AtomicFenceOp::create`、`std::swap`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2447-2453
```cpp
2447:     // Zero-extend shift amount to i64 if needed
2448:     auto amtTy = mlir::cast<cir::IntType>(ops[2].getType());
2449:     cir::IntType i64Ty = builder.getUInt64Ty();
2450: 
2451:     if (amtTy != i64Ty)
2452:       ops[2] = builder.createIntCast(ops[2], i64Ty);
2453: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2454-2471
```cpp
2454:     const StringRef intrinsicName =
2455:         (builtinID == X86::BI__shiftleft128) ? "fshl" : "fshr";
2456:     return builder.emitIntrinsicCallOp(
2457:         getLoc(expr->getExprLoc()), intrinsicName, i64Ty,
2458:         mlir::ValueRange{ops[0], ops[1], ops[2]});
2459:   }
2460:   case X86::BI_ReadWriteBarrier:
2461:   case X86::BI_ReadBarrier:
2462:   case X86::BI_WriteBarrier: {
2463:     cir::AtomicFenceOp::create(
2464:         builder, getLoc(expr->getExprLoc()),
2465:         cir::MemOrder::SequentiallyConsistent,
2466:         cir::SyncScopeKindAttr::get(&getMLIRContext(),
2467:                                     cir::SyncScopeKind::SingleThread));
2468:     return mlir::Value{};
2469:   }
2470:   case X86::BI_AddressOfReturnAddress: {
2471:     mlir::Location loc = getLoc(expr->getExprLoc());
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::AtomicFenceOp::create`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::AtomicFenceOp::create`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2472-2489
```cpp
2472:     mlir::Value addr =
2473:         cir::AddrOfReturnAddrOp::create(builder, loc, allocaInt8PtrTy);
2474:     return builder.createCast(loc, cir::CastKind::bitcast, addr, voidPtrTy);
2475:   }
2476:   case X86::BI__stosb:
2477:   case X86::BI__ud2:
2478:   case X86::BI__int2c:
2479:   case X86::BI__readfsbyte:
2480:   case X86::BI__readfsword:
2481:   case X86::BI__readfsdword:
2482:   case X86::BI__readfsqword:
2483:   case X86::BI__readgsbyte:
2484:   case X86::BI__readgsword:
2485:   case X86::BI__readgsdword:
2486:   case X86::BI__readgsqword: {
2487:     cgm.errorNYI(expr->getSourceRange(),
2488:                  std::string("unimplemented X86 builtin call: ") +
2489:                      getContext().BuiltinInfo.getName(builtinID));
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::AddrOfReturnAddrOp::create`, `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::AddrOfReturnAddrOp::create`、`std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2490-2497
```cpp
2490:     return mlir::Value{};
2491:   }
2492:   case X86::BI__builtin_ia32_encodekey128_u32: {
2493:     return emitEncodeKey(&getMLIRContext(), builder, getLoc(expr->getExprLoc()),
2494:                          {ops[0], ops[1]}, ops[2], 6, "x86.encodekey128", 3);
2495:   }
2496:   case X86::BI__builtin_ia32_encodekey256_u32: {
2497: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2498-2502
```cpp
2498:     return emitEncodeKey(&getMLIRContext(), builder, getLoc(expr->getExprLoc()),
2499:                          {ops[0], ops[1], ops[2]}, ops[3], 7,
2500:                          "x86.encodekey256", 4);
2501:   }
2502: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2503-2520
```cpp
2503:   case X86::BI__builtin_ia32_aesenc128kl_u8:
2504:   case X86::BI__builtin_ia32_aesdec128kl_u8:
2505:   case X86::BI__builtin_ia32_aesenc256kl_u8:
2506:   case X86::BI__builtin_ia32_aesdec256kl_u8: {
2507:     llvm::StringRef intrinsicName;
2508:     switch (builtinID) {
2509:     default:
2510:       llvm_unreachable("Unexpected builtin");
2511:     case X86::BI__builtin_ia32_aesenc128kl_u8:
2512:       intrinsicName = "x86.aesenc128kl";
2513:       break;
2514:     case X86::BI__builtin_ia32_aesdec128kl_u8:
2515:       intrinsicName = "x86.aesdec128kl";
2516:       break;
2517:     case X86::BI__builtin_ia32_aesenc256kl_u8:
2518:       intrinsicName = "x86.aesenc256kl";
2519:       break;
2520:     case X86::BI__builtin_ia32_aesdec256kl_u8:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 2521-2524
```cpp
2521:       intrinsicName = "x86.aesdec256kl";
2522:       break;
2523:     }
2524: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2525-2542
```cpp
2525:     return emitX86Aes(builder, getLoc(expr->getExprLoc()), intrinsicName,
2526:                       convertType(expr->getType()), ops);
2527:   }
2528:   case X86::BI__builtin_ia32_aesencwide128kl_u8:
2529:   case X86::BI__builtin_ia32_aesdecwide128kl_u8:
2530:   case X86::BI__builtin_ia32_aesencwide256kl_u8:
2531:   case X86::BI__builtin_ia32_aesdecwide256kl_u8: {
2532:     llvm::StringRef intrinsicName;
2533:     switch (builtinID) {
2534:     default:
2535:       llvm_unreachable("Unexpected builtin");
2536:     case X86::BI__builtin_ia32_aesencwide128kl_u8:
2537:       intrinsicName = "x86.aesencwide128kl";
2538:       break;
2539:     case X86::BI__builtin_ia32_aesdecwide128kl_u8:
2540:       intrinsicName = "x86.aesdecwide128kl";
2541:       break;
2542:     case X86::BI__builtin_ia32_aesencwide256kl_u8:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2543-2549
```cpp
2543:       intrinsicName = "x86.aesencwide256kl";
2544:       break;
2545:     case X86::BI__builtin_ia32_aesdecwide256kl_u8:
2546:       intrinsicName = "x86.aesdecwide256kl";
2547:       break;
2548:     }
2549: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2550-2565
```cpp
2550:     return emitX86Aeswide(builder, getLoc(expr->getExprLoc()), intrinsicName,
2551:                           convertType(expr->getType()), ops);
2552:   }
2553:   case X86::BI__builtin_ia32_vfcmaddcph512_mask:
2554:   case X86::BI__builtin_ia32_vfmaddcph512_mask:
2555:   case X86::BI__builtin_ia32_vfcmaddcsh_round_mask:
2556:   case X86::BI__builtin_ia32_vfmaddcsh_round_mask:
2557:   case X86::BI__builtin_ia32_vfcmaddcsh_round_mask3:
2558:   case X86::BI__builtin_ia32_vfmaddcsh_round_mask3:
2559:   case X86::BI__builtin_ia32_prefetchi:
2560:     cgm.errorNYI(expr->getSourceRange(),
2561:                  std::string("unimplemented X86 builtin call: ") +
2562:                      getContext().BuiltinInfo.getName(builtinID));
2563:     return mlir::Value{};
2564:   }
2565: }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

## Key Concepts / 关键概念

- **Clang AST integration / Clang AST 集成**: Reads semantic information from Clang AST data structures. 从 Clang AST 数据结构读取语义信息。
- **MLIR/CIR integration / MLIR/CIR 集成**: Bridges Clang logic to MLIR/CIR operations, attributes, or types. 将 Clang 逻辑连接到 MLIR/CIR 的操作、属性或类型。
- **CIR dialect usage / CIR 方言使用**: Manipulates CIR-specific types, attributes, and operations. 操作 CIR 专用的类型、属性和操作。
- **CIR generation pipeline / CIR 生成流水线**: Participates in the lowering pipeline from Clang semantics to CIR constructs. 参与从 Clang 语义到 CIR 构造的 lowering 流水线。
- **`type` / `type`**: `type` is a prominent symbol in this file and helps define its structure or behavior. `type` 是该文件中的关键符号之一，用来界定其结构或行为。
- **`for` / `for`**: `for` is a prominent symbol in this file and helps define its structure or behavior. `for` 是该文件中的关键符号之一，用来界定其结构或行为。

## Dependencies / 依赖关系

- **Clang / Clang**: `clang/Basic/Builtins.h`, `clang/Basic/TargetBuiltins.h`, `clang/CIR/Dialect/IR/CIRAttrs.h`, `clang/CIR/Dialect/IR/CIRTypes.h`, `clang/CIR/MissingFeatures.h`
- **LLVM / LLVM**: `llvm/ADT/Sequence.h`, `llvm/Support/ErrorHandling.h`
- **MLIR / MLIR**: `mlir/IR/Attributes.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/Location.h`, `mlir/IR/Types.h`, `mlir/IR/ValueRange.h`
- **StdLib/Other / 标准库/其他**: `CIRGenBuilder.h`, `CIRGenFunction.h`, `CIRGenModule.h`, `string`
