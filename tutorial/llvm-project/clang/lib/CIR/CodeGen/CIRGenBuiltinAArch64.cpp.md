# CIRGenBuiltinAArch64.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `clang/lib/CIR/CodeGen/CIRGenBuiltinAArch64.cpp`
- **Repository**: `/root/xw/llvm-project` (`llvm-project`)
- **Purpose (EN)**: This contains code to emit ARM64 Builtin calls as CIR or a function call to be later resolved.
- **Purpose (CN)**: 实现与 `CIRGenBuiltinAArch64` 相关的 CIR 代码生成支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
   1: //===---- CIRGenBuiltinAArch64.cpp - Emit CIR for AArch64 builtins --------===//
   2: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   3: // See https://llvm.org/LICENSE.txt for license information.
   4: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   5: //
   6: //===----------------------------------------------------------------------===//
   7: //
   8: // This contains code to emit ARM64 Builtin calls as CIR or a function call
   9: // to be later resolved.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 13-19
```cpp
  13: #include "CIRGenBuilder.h"
  14: #include "CIRGenFunction.h"
  15: #include "clang/Basic/AArch64CodeGenUtils.h"
  16: #include "clang/Basic/TargetBuiltins.h"
  17: #include "clang/CIR/Dialect/IR/CIRTypes.h"
  18: #include "clang/CIR/MissingFeatures.h"
  19: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `CIRGenBuilder.h`, `CIRGenFunction.h`, `AArch64CodeGenUtils.h`, `TargetBuiltins.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `CIRGenBuilder.h`, `CIRGenFunction.h`, `AArch64CodeGenUtils.h`, `TargetBuiltins.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 20-26
```cpp
  20: // TODO(cir): once all builtins are covered, decide whether we still
  21: // need to use LLVM intrinsics or if there's a better approach to follow. Right
  22: // now the intrinsics are reused to make it convenient to encode all thousands
  23: // of them and passing down to LLVM lowering.
  24: #include "llvm/IR/Intrinsics.h"
  25: #include "llvm/IR/IntrinsicsAArch64.h"
  26: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `Intrinsics.h`, `IntrinsicsAArch64.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `Intrinsics.h`, `IntrinsicsAArch64.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 27-31
```cpp
  27: #include "mlir/IR/BuiltinTypes.h"
  28: #include "mlir/IR/Value.h"
  29: #include "clang/AST/GlobalDecl.h"
  30: #include "clang/Basic/Builtins.h"
  31: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `BuiltinTypes.h`, `Value.h`, `GlobalDecl.h`, `Builtins.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `BuiltinTypes.h`, `Value.h`, `GlobalDecl.h`, `Builtins.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 32-36
```cpp
  32: using namespace clang;
  33: using namespace clang::CIRGen;
  34: using namespace llvm;
  35: using namespace clang::aarch64;
  36: 
```
- **EN**: This block organizes symbols with namespaces or using-declarations for shorter references.
- **CN**: 该代码块通过命名空间或 using 声明组织符号并简化后续引用。

### Lines 37-46
```cpp
  37: // Generate vscale * scalingFactor
  38: static mlir::Value genVscaleTimesFactor(mlir::Location loc,
  39:                                         CIRGenBuilderTy builder,
  40:                                         mlir::Type cirTy,
  41:                                         int32_t scalingFactor) {
  42:   mlir::Value vscale = builder.emitIntrinsicCallOp(loc, "vscale", cirTy);
  43:   return builder.createNUWAMul(loc, vscale,
  44:                                builder.getUInt64(scalingFactor, loc));
  45: }
  46: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `genVscaleTimesFactor`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `genVscaleTimesFactor`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 47-50
```cpp
  47: #define SVEMAP1(NameBase, LLVMIntrinsic, TypeModifier)                         \
  48:   {#NameBase, SVE::BI__builtin_sve_##NameBase, Intrinsic::LLVMIntrinsic, 0,    \
  49:    TypeModifier}
  50: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。

### Lines 51-58
```cpp
  51: #define SVEMAP2(NameBase, TypeModifier)                                        \
  52:   {#NameBase, SVE::BI__builtin_sve_##NameBase, 0, 0, TypeModifier}
  53: static const ARMVectorIntrinsicInfo aarch64SVEIntrinsicMap[] = {
  54: #define GET_SVE_LLVM_INTRINSIC_MAP
  55: #include "clang/Basic/arm_sve_builtin_cg.inc"
  56: #undef GET_SVE_LLVM_INTRINSIC_MAP
  57: };
  58: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `arm_sve_builtin_cg.inc` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `arm_sve_builtin_cg.inc` 这样的头文件说明了该区域依赖的主要 API。

### Lines 59-62
```cpp
  59: static bool aarch64SIMDIntrinsicsProvenSorted = false;
  60: static bool aarch64SISDIntrinsicsProvenSorted = false;
  61: static bool aarch64SVEIntrinsicsProvenSorted = false;
  62: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 63-68
```cpp
  63: // Check if Builtin `builtinId` is present in `intrinsicMap`. If yes, returns
  64: // the corresponding info struct.
  65: static const ARMVectorIntrinsicInfo *
  66: findARMVectorIntrinsicInMap(ArrayRef<ARMVectorIntrinsicInfo> intrinsicMap,
  67:                             unsigned builtinID, bool &mapProvenSorted) {
  68: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Notable callable symbols here include `findARMVectorIntrinsicInMap`.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 其中值得关注的可调用符号包括 `findARMVectorIntrinsicInMap`。

### Lines 69-84
```cpp
  69: #ifndef NDEBUG
  70:   if (!mapProvenSorted) {
  71:     assert(llvm::is_sorted(intrinsicMap));
  72:     mapProvenSorted = true;
  73:   }
  74: #endif
  75: 
  76:   const ARMVectorIntrinsicInfo *info =
  77:       llvm::lower_bound(intrinsicMap, builtinID);
  78: 
  79:   if (info != intrinsicMap.end() && info->BuiltinID == builtinID)
  80:     return info;
  81: 
  82:   return nullptr;
  83: }
  84: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Notable callable symbols here include `assert`, `llvm::lower_bound`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 其中值得关注的可调用符号包括 `assert`、`llvm::lower_bound`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 85-98
```cpp
  85: //===----------------------------------------------------------------------===//
  86: //  Generic helpers
  87: //===----------------------------------------------------------------------===//
  88: // Emit an intrinsic where all operands are of the same type as the result.
  89: // Depending on mode, this may be a constrained floating-point intrinsic.
  90: static mlir::Value
  91: emitCallMaybeConstrainedBuiltin(CIRGenBuilderTy &builder, mlir::Location loc,
  92:                                 StringRef intrName, mlir::Type retTy,
  93:                                 llvm::SmallVector<mlir::Value> &ops) {
  94:   assert(!cir::MissingFeatures::emitConstrainedFPCall());
  95: 
  96:   return builder.emitIntrinsicCallOp(loc, intrName, retTy, ops);
  97: }
  98: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitCallMaybeConstrainedBuiltin`, `assert`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitCallMaybeConstrainedBuiltin`、`assert`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 99-104
```cpp
  99: static llvm::StringRef getLLVMIntrNameNoPrefix(llvm::Intrinsic::ID intrID) {
 100:   llvm::StringRef llvmIntrName = llvm::Intrinsic::getBaseName(intrID);
 101:   assert(llvmIntrName.starts_with("llvm.") && "Not an LLVM intrinsic!");
 102:   return llvmIntrName.drop_front(/*strlen("llvm.")=*/5);
 103: }
 104: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getLLVMIntrNameNoPrefix`, `assert`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getLLVMIntrNameNoPrefix`、`assert`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 105-119
```cpp
 105: //===----------------------------------------------------------------------===//
 106: //  NEON helpers
 107: //===----------------------------------------------------------------------===//
 108: /// Return true if BuiltinID is an overloaded Neon intrinsic with an extra
 109: /// argument that specifies the vector type. The additional argument is meant
 110: /// for Sema checking (see `CheckNeonBuiltinFunctionCall`) and this function
 111: /// should be kept consistent with the logic in Sema.
 112: /// TODO: Make this return false for SISD builtins.
 113: /// TODO(cir): Share this with ARM.cpp
 114: static bool hasExtraNeonArgument(unsigned builtinID) {
 115:   // Required by the headers included below, but not in this particular
 116:   // function.
 117:   [[maybe_unused]] int PtrArgNum = -1;
 118:   [[maybe_unused]] bool HasConstPtr = false;
 119: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `hasExtraNeonArgument`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `hasExtraNeonArgument`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 120-141
```cpp
 120:   // The mask encodes the type. We don't care about the actual value. Instead,
 121:   // we just check whether its been set.
 122:   uint64_t mask = 0;
 123:   switch (builtinID) {
 124: #define GET_NEON_OVERLOAD_CHECK
 125: #include "clang/Basic/arm_fp16.inc"
 126: #include "clang/Basic/arm_neon.inc"
 127: #undef GET_NEON_OVERLOAD_CHECK
 128:   // Non-neon builtins for controling VFP that take extra argument for
 129:   // discriminating the type.
 130:   case ARM::BI__builtin_arm_vcvtr_f:
 131:   case ARM::BI__builtin_arm_vcvtr_d:
 132:     mask = 1;
 133:   }
 134:   switch (builtinID) {
 135:   default:
 136:     break;
 137:   }
 138: 
 139:   return mask != 0;
 140: }
 141: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `arm_fp16.inc`, `arm_neon.inc` reveal the main APIs consumed by this region. A switch statement is used to dispatch behavior across enumerated cases or kinds. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `arm_fp16.inc`, `arm_neon.inc` 这样的头文件说明了该区域依赖的主要 API。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 142-156
```cpp
 142: static cir::VectorType getFloatNeonType(CIRGenFunction &cgf,
 143:                                         NeonTypeFlags intTypeFlags) {
 144:   int isQuad = intTypeFlags.isQuad();
 145:   switch (intTypeFlags.getEltType()) {
 146:   case NeonTypeFlags::Int16:
 147:     return cir::VectorType::get(cgf.fP16Ty, (4 << isQuad));
 148:   case NeonTypeFlags::Int32:
 149:     return cir::VectorType::get(cgf.floatTy, (2 << isQuad));
 150:   case NeonTypeFlags::Int64:
 151:     return cir::VectorType::get(cgf.doubleTy, (1 << isQuad));
 152:   default:
 153:     llvm_unreachable("Type can't be converted to floating-point!");
 154:   }
 155: }
 156: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getFloatNeonType`, `cir::VectorType::get`, `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getFloatNeonType`、`cir::VectorType::get`、`llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 157-160
```cpp
 157: static int64_t getIntValueFromConstOp(mlir::Value val) {
 158:   return val.getDefiningOp<cir::ConstantOp>().getIntValue().getSExtValue();
 159: }
 160: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getIntValueFromConstOp`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getIntValueFromConstOp`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 161-181
```cpp
 161: /// Build a constant shift amount vector of `vecTy` to shift a vector
 162: /// Here `shiftVal` is a constant integer that will be broadcast into a
 163: /// a const vector of `vecTy` which is the return value of this function
 164: /// If `neg` is true, the shift amount is negated before splatting (used
 165: /// when encoding a right shift as a left shift by a negative amount for
 166: /// intrinsics like aarch64.neon.{s,u}rshl).
 167: static mlir::Value emitNeonShiftVector(CIRGenBuilderTy &builder,
 168:                                        mlir::Value shiftVal,
 169:                                        cir::VectorType vecTy,
 170:                                        mlir::Location loc, bool neg) {
 171:   if (neg) {
 172:     int64_t shiftAmt = -getIntValueFromConstOp(shiftVal);
 173:     shiftVal = builder.getConstantInt(loc, vecTy.getElementType(), shiftAmt);
 174:   }
 175:   mlir::Type eltTy = vecTy.getElementType();
 176:   if (shiftVal.getType() != eltTy) {
 177:     shiftVal = builder.createIntCast(shiftVal, eltTy);
 178:   }
 179:   return cir::VecSplatOp::create(builder, loc, vecTy, shiftVal);
 180: }
 181: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitNeonShiftVector`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitNeonShiftVector`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 182-197
```cpp
 182: // TODO(cir): Remove `cgm` from the list of arguments once all NYI(s) are gone.
 183: template <typename Operation>
 184: static mlir::Value
 185: emitNeonCallToOp(CIRGenModule &cgm, CIRGenBuilderTy &builder,
 186:                  llvm::SmallVector<mlir::Type> argTypes,
 187:                  llvm::SmallVectorImpl<mlir::Value> &args,
 188:                  std::optional<llvm::StringRef> intrinsicName,
 189:                  mlir::Type funcResTy, mlir::Location loc,
 190:                  bool isConstrainedFPIntrinsic = false, unsigned shift = 0,
 191:                  bool rightshift = false) {
 192:   // TODO(cir): Consider removing the following unreachable when we have
 193:   // emitConstrainedFPCall feature implemented
 194:   assert(!cir::MissingFeatures::emitConstrainedFPCall());
 195:   if (isConstrainedFPIntrinsic)
 196:     cgm.errorNYI(loc, std::string("constrained FP intrinsic"));
 197: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitNeonCallToOp`, `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitNeonCallToOp`、`assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 198-215
```cpp
 198:   for (unsigned j = 0; j < argTypes.size(); ++j) {
 199:     if (isConstrainedFPIntrinsic) {
 200:       assert(!cir::MissingFeatures::emitConstrainedFPCall());
 201:     }
 202:     if (shift > 0 && shift == j) {
 203:       args[j] = emitNeonShiftVector(builder, args[j],
 204:                                     mlir::cast<cir::VectorType>(argTypes[j]),
 205:                                     loc, rightshift);
 206:     } else {
 207:       args[j] = builder.createBitcast(args[j], argTypes[j]);
 208:     }
 209:   }
 210:   if (isConstrainedFPIntrinsic) {
 211:     assert(!cir::MissingFeatures::emitConstrainedFPCall());
 212:     return nullptr;
 213:   }
 214:   if constexpr (std::is_same_v<Operation, cir::LLVMIntrinsicCallOp>) {
 215:     return Operation::create(builder, loc,
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 216-223
```cpp
 216:                              builder.getStringAttr(intrinsicName.value()),
 217:                              funcResTy, args)
 218:         .getResult();
 219:   } else {
 220:     return Operation::create(builder, loc, funcResTy, args).getResult();
 221:   }
 222: }
 223: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 224-236
```cpp
 224: // TODO(cir): Remove `cgm` from the list of arguments once all NYI(s) are gone.
 225: static mlir::Value emitNeonCall(CIRGenModule &cgm, CIRGenBuilderTy &builder,
 226:                                 llvm::SmallVector<mlir::Type> argTypes,
 227:                                 llvm::SmallVectorImpl<mlir::Value> &args,
 228:                                 llvm::StringRef intrinsicName,
 229:                                 mlir::Type funcResTy, mlir::Location loc,
 230:                                 bool isConstrainedFPIntrinsic = false,
 231:                                 unsigned shift = 0, bool rightshift = false) {
 232:   return emitNeonCallToOp<cir::LLVMIntrinsicCallOp>(
 233:       cgm, builder, std::move(argTypes), args, intrinsicName, funcResTy, loc,
 234:       isConstrainedFPIntrinsic, shift, rightshift);
 235: }
 236: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitNeonCall`, `std::move`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitNeonCall`、`std::move`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 237-247
```cpp
 237: // Computes the input vector type for a NEON pairwise widening operation (e.g.
 238: // vpaddl/vpadal). Given a result vector type, it derives the corresponding
 239: // input type by halving the element bit width and doubling the number of lanes,
 240: // while setting the signedness based on usgn.
 241: static cir::VectorType getNeonPairwiseWidenInputType(cir::VectorType resType,
 242:                                                      bool usgn) {
 243:   mlir::Type elemTy = resType.getElementType();
 244:   uint64_t resLanes = resType.getSize();
 245:   auto intTy = mlir::dyn_cast<cir::IntType>(elemTy);
 246:   assert(intTy && "vpaddl result type must be an integer vector");
 247: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getNeonPairwiseWidenInputType`, `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getNeonPairwiseWidenInputType`、`assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 248-251
```cpp
 248:   unsigned resWidth = intTy.getWidth();
 249:   assert((resWidth == 16 || resWidth == 32 || resWidth == 64) &&
 250:          "unexpected vpaddl result element width");
 251: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。

### Lines 252-259
```cpp
 252:   unsigned argWidth = resWidth / 2;
 253:   unsigned argLanes = resLanes * 2;
 254:   cir::VectorType result = cir::VectorType::get(
 255:       cir::IntType::get(resType.getContext(), argWidth, /* is_signed */ !usgn),
 256:       argLanes);
 257:   return result;
 258: }
 259: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::IntType::get`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::IntType::get`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 260-264
```cpp
 260: static mlir::Value emitCommonNeonSISDBuiltinExpr(
 261:     CIRGenFunction &cgf, const ARMVectorIntrinsicInfo &info,
 262:     llvm::SmallVectorImpl<mlir::Value> &ops, const CallExpr *expr) {
 263:   assert(info.LLVMIntrinsic && "Generic code assumes a valid intrinsic");
 264: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitCommonNeonSISDBuiltinExpr`, `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitCommonNeonSISDBuiltinExpr`、`assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 265-282
```cpp
 265:   switch (info.BuiltinID) {
 266:   case NEON::BI__builtin_neon_vcled_s64:
 267:   case NEON::BI__builtin_neon_vcled_u64:
 268:   case NEON::BI__builtin_neon_vcles_f32:
 269:   case NEON::BI__builtin_neon_vcled_f64:
 270:   case NEON::BI__builtin_neon_vcltd_s64:
 271:   case NEON::BI__builtin_neon_vcltd_u64:
 272:   case NEON::BI__builtin_neon_vclts_f32:
 273:   case NEON::BI__builtin_neon_vcltd_f64:
 274:   case NEON::BI__builtin_neon_vcales_f32:
 275:   case NEON::BI__builtin_neon_vcaled_f64:
 276:   case NEON::BI__builtin_neon_vcalts_f32:
 277:   case NEON::BI__builtin_neon_vcaltd_f64:
 278:     cgf.cgm.errorNYI(expr->getSourceRange(),
 279:                      std::string("unimplemented AArch64 builtin call: ") +
 280:                          cgf.getContext().BuiltinInfo.getName(info.BuiltinID));
 281:     break;
 282:   }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 283-287
```cpp
 283: 
 284:   llvm::StringRef llvmIntrName = getLLVMIntrNameNoPrefix(
 285:       static_cast<llvm::Intrinsic::ID>(info.LLVMIntrinsic));
 286:   mlir::Location loc = cgf.getLoc(expr->getExprLoc());
 287: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 288-305
```cpp
 288:   // The switch stmt is intended to help catch NYI cases and will be removed
 289:   // once the CIR implementation is complete. Avoid adding specialized
 290:   // code in cases - that should only be required for a handful of examples.
 291:   switch (info.BuiltinID) {
 292:   default:
 293:     cgf.cgm.errorNYI(expr->getSourceRange(),
 294:                      std::string("unimplemented AArch64 builtin call: ") +
 295:                          cgf.getContext().BuiltinInfo.getName(info.BuiltinID));
 296:     break;
 297:   case NEON::BI__builtin_neon_vminv_s8:
 298:   case NEON::BI__builtin_neon_vminvq_s8:
 299:   case NEON::BI__builtin_neon_vminv_s16:
 300:   case NEON::BI__builtin_neon_vminvq_s16:
 301:   case NEON::BI__builtin_neon_vminv_s32:
 302:   case NEON::BI__builtin_neon_vminvq_s32:
 303:   case NEON::BI__builtin_neon_vminv_u8:
 304:   case NEON::BI__builtin_neon_vminvq_u8:
 305:   case NEON::BI__builtin_neon_vminv_u16:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 306-323
```cpp
 306:   case NEON::BI__builtin_neon_vminvq_u16:
 307:   case NEON::BI__builtin_neon_vminv_u32:
 308:   case NEON::BI__builtin_neon_vminvq_u32:
 309:   case NEON::BI__builtin_neon_vminv_f32:
 310:   case NEON::BI__builtin_neon_vminvq_f32:
 311:   case NEON::BI__builtin_neon_vminvq_f64:
 312:   case NEON::BI__builtin_neon_vminnmv_f32:
 313:   case NEON::BI__builtin_neon_vminnmvq_f32:
 314:   case NEON::BI__builtin_neon_vminnmvq_f64:
 315:   case NEON::BI__builtin_neon_vabdd_f64:
 316:   case NEON::BI__builtin_neon_vabds_f32:
 317:   case NEON::BI__builtin_neon_vshld_s64:
 318:   case NEON::BI__builtin_neon_vshld_u64:
 319:   case NEON::BI__builtin_neon_vpmins_f32:
 320:   case NEON::BI__builtin_neon_vpminqd_f64:
 321:   case NEON::BI__builtin_neon_vpminnms_f32:
 322:   case NEON::BI__builtin_neon_vpminnmqd_f64:
 323:   case NEON::BI__builtin_neon_vcvts_n_f32_s32:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 324-341
```cpp
 324:   case NEON::BI__builtin_neon_vcvts_n_f32_u32:
 325:   case NEON::BI__builtin_neon_vcvts_n_s32_f32:
 326:   case NEON::BI__builtin_neon_vcvts_n_u32_f32:
 327:   case NEON::BI__builtin_neon_vcvtd_n_f64_s64:
 328:   case NEON::BI__builtin_neon_vcvtd_n_f64_u64:
 329:   case NEON::BI__builtin_neon_vcvtd_n_s64_f64:
 330:   case NEON::BI__builtin_neon_vcvtd_n_u64_f64:
 331:   case NEON::BI__builtin_neon_vaddlv_s32:
 332:   case NEON::BI__builtin_neon_vaddlv_u32:
 333:   case NEON::BI__builtin_neon_vaddlvq_s32:
 334:   case NEON::BI__builtin_neon_vaddlvq_u32:
 335:   case NEON::BI__builtin_neon_vaddv_s8:
 336:   case NEON::BI__builtin_neon_vaddv_s16:
 337:   case NEON::BI__builtin_neon_vaddv_s32:
 338:   case NEON::BI__builtin_neon_vaddv_u8:
 339:   case NEON::BI__builtin_neon_vaddv_u16:
 340:   case NEON::BI__builtin_neon_vaddv_u32:
 341:   case NEON::BI__builtin_neon_vaddv_f32:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 342-364
```cpp
 342:   case NEON::BI__builtin_neon_vaddvq_s8:
 343:   case NEON::BI__builtin_neon_vaddvq_s16:
 344:   case NEON::BI__builtin_neon_vaddvq_s32:
 345:   case NEON::BI__builtin_neon_vaddvq_s64:
 346:   case NEON::BI__builtin_neon_vaddvq_u8:
 347:   case NEON::BI__builtin_neon_vaddvq_u16:
 348:   case NEON::BI__builtin_neon_vaddvq_u32:
 349:   case NEON::BI__builtin_neon_vaddvq_u64:
 350:   case NEON::BI__builtin_neon_vaddvq_f32:
 351:   case NEON::BI__builtin_neon_vaddvq_f64:
 352:   case NEON::BI__builtin_neon_vabdh_f16:
 353:   case NEON::BI__builtin_neon_vrecpeh_f16:
 354:   case NEON::BI__builtin_neon_vrecpxh_f16:
 355:   case NEON::BI__builtin_neon_vrsqrteh_f16:
 356:   case NEON::BI__builtin_neon_vrsqrtsh_f16:
 357:     return emitNeonCall(cgf.cgm, cgf.getBuilder(),
 358:                         {cgf.convertType(expr->getArg(0)->getType())}, ops,
 359:                         llvmIntrName, cgf.convertType(expr->getType()), loc);
 360:   }
 361: 
 362:   return nullptr;
 363: }
 364: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 365-372
```cpp
 365: //===----------------------------------------------------------------------===//
 366: //  Emit-helpers
 367: //===----------------------------------------------------------------------===//
 368: static mlir::Value
 369: emitAArch64CompareBuiltinExpr(CIRGenFunction &cgf, CIRGenBuilderTy &builder,
 370:                               mlir::Location loc, mlir::Value src,
 371:                               mlir::Type retTy, const cir::CmpOpKind kind) {
 372: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitAArch64CompareBuiltinExpr`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitAArch64CompareBuiltinExpr`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 373-385
```cpp
 373:   bool scalarCmp = !isa<cir::VectorType>(src.getType());
 374:   if (!scalarCmp) {
 375:     assert(!cast<cir::VectorType>(retTy).getIsScalable() &&
 376:            "This is only intended for fixed-width vectors");
 377:     // Vector types are cast to i8 vectors. Recover original type.
 378:     src = builder.createBitcast(src, retTy);
 379:   }
 380: 
 381:   mlir::Value zero = builder.getNullValue(src.getType(), loc);
 382: 
 383:   if (!scalarCmp)
 384:     return builder.createVecCompare(loc, kind, src, zero);
 385: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 386-395
```cpp
 386:   // For scalars, cast !cir.bool to !cir.int<s, 1> so that the compare
 387:   // result is sign- rather zero-extended when casting to the output
 388:   // retType.
 389:   mlir::Value cmp = builder.createCast(
 390:       loc, cir::CastKind::bool_to_int,
 391:       builder.createCompare(loc, kind, src, zero), builder.getSIntNTy(1));
 392: 
 393:   return builder.createCast(loc, cir::CastKind::integral, cmp, retTy);
 394: }
 395: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 396-413
```cpp
 396: // TODO(cir): Remove `loc` from the list of arguments once all NYIs are gone.
 397: static cir::VectorType getNeonType(CIRGenFunction *cgf, NeonTypeFlags typeFlags,
 398:                                    mlir::Location loc,
 399:                                    bool hasLegalHalfType = true,
 400:                                    bool v1Ty = false,
 401:                                    bool allowBFloatArgsAndRet = true) {
 402:   int isQuad = typeFlags.isQuad();
 403:   switch (typeFlags.getEltType()) {
 404:   case NeonTypeFlags::Int8:
 405:   case NeonTypeFlags::Poly8:
 406:     return cir::VectorType::get(typeFlags.isUnsigned() ? cgf->uInt8Ty
 407:                                                        : cgf->sInt8Ty,
 408:                                 v1Ty ? 1 : (8 << isQuad));
 409:   case NeonTypeFlags::MFloat8:
 410:     return cir::VectorType::get(cgf->uInt8Ty, v1Ty ? 1 : (8 << isQuad));
 411:   case NeonTypeFlags::Int16:
 412:   case NeonTypeFlags::Poly16:
 413:     return cir::VectorType::get(typeFlags.isUnsigned() ? cgf->uInt16Ty
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getNeonType`, `cir::VectorType::get`. A switch statement is used to dispatch behavior across enumerated cases or kinds. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getNeonType`、`cir::VectorType::get`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 414-431
```cpp
 414:                                                        : cgf->sInt16Ty,
 415:                                 v1Ty ? 1 : (4 << isQuad));
 416:   case NeonTypeFlags::BFloat16:
 417:     if (allowBFloatArgsAndRet)
 418:       return cir::VectorType::get(cgf->getCIRGenModule().bFloat16Ty,
 419:                                   v1Ty ? 1 : (4 << isQuad));
 420:     return cir::VectorType::get(cgf->uInt16Ty, v1Ty ? 1 : (4 << isQuad));
 421:   case NeonTypeFlags::Float16:
 422:     if (hasLegalHalfType)
 423:       return cir::VectorType::get(cgf->getCIRGenModule().fP16Ty,
 424:                                   v1Ty ? 1 : (4 << isQuad));
 425:     return cir::VectorType::get(cgf->uInt16Ty, v1Ty ? 1 : (4 << isQuad));
 426:   case NeonTypeFlags::Int32:
 427:     return cir::VectorType::get(typeFlags.isUnsigned() ? cgf->uInt32Ty
 428:                                                        : cgf->sInt32Ty,
 429:                                 v1Ty ? 1 : (2 << isQuad));
 430:   case NeonTypeFlags::Int64:
 431:   case NeonTypeFlags::Poly64:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::VectorType::get`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::VectorType::get`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 432-449
```cpp
 432:     return cir::VectorType::get(typeFlags.isUnsigned() ? cgf->uInt64Ty
 433:                                                        : cgf->sInt64Ty,
 434:                                 v1Ty ? 1 : (1 << isQuad));
 435:   case NeonTypeFlags::Poly128:
 436:     // FIXME: i128 and f128 doesn't get fully support in Clang and llvm.
 437:     // There is a lot of i128 and f128 API missing.
 438:     // so we use v16i8 to represent poly128 and get pattern matched.
 439:     cgf->getCIRGenModule().errorNYI(loc, std::string("NEON type: Poly128"));
 440:     [[fallthrough]];
 441:   case NeonTypeFlags::Float32:
 442:     return cir::VectorType::get(cgf->getCIRGenModule().floatTy,
 443:                                 v1Ty ? 1 : (2 << isQuad));
 444:   case NeonTypeFlags::Float64:
 445:     return cir::VectorType::get(cgf->getCIRGenModule().doubleTy,
 446:                                 v1Ty ? 1 : (1 << isQuad));
 447:   }
 448:   llvm_unreachable("Unknown vector element type!");
 449: }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::VectorType::get`, `llvm_unreachable`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::VectorType::get`、`llvm_unreachable`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 450-460
```cpp
 450: 
 451: static mlir::Value emitNeonSplat(CIRGenBuilderTy &builder, mlir::Location loc,
 452:                                  mlir::Value v, mlir::Value lane,
 453:                                  unsigned int resEltCnt) {
 454:   assert(isa<cir::ConstantOp>(lane.getDefiningOp()) &&
 455:          "lane number is not a constant!");
 456:   int64_t laneCst = getIntValueFromConstOp(lane);
 457:   llvm::SmallVector<int64_t, 4> shuffleMask(resEltCnt, laneCst);
 458:   return builder.createVecShuffle(loc, v, shuffleMask);
 459: }
 460: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitNeonSplat`, `assert`, `shuffleMask`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitNeonSplat`、`assert`、`shuffleMask`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 461-472
```cpp
 461: /// Flip the signedness of `vecTy`'s element type, keeping the width and
 462: /// number of lanes the same. Used when a NEON intrinsic takes a shift
 463: /// amount vector that must be signed (e.g. aarch64.neon.urshl takes a
 464: /// signed amount even though the data vector is unsigned).
 465: static cir::VectorType getSignChangedVectorType(CIRGenBuilderTy &builder,
 466:                                                 cir::VectorType vecTy) {
 467:   auto elemTy = mlir::cast<cir::IntType>(vecTy.getElementType());
 468:   elemTy = elemTy.isSigned() ? builder.getUIntNTy(elemTy.getWidth())
 469:                              : builder.getSIntNTy(elemTy.getWidth());
 470:   return cir::VectorType::get(elemTy, vecTy.getSize());
 471: }
 472: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getSignChangedVectorType`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getSignChangedVectorType`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 473-483
```cpp
 473: static mlir::Value emitCommonNeonShift(CIRGenBuilderTy &builder,
 474:                                        mlir::Location loc,
 475:                                        cir::VectorType resTy,
 476:                                        mlir::Value shifTgt,
 477:                                        mlir::Value shiftAmt, bool shiftLeft) {
 478:   shiftAmt = emitNeonShiftVector(builder, shiftAmt, resTy, loc, /*neg=*/false);
 479:   return cir::ShiftOp::create(builder, loc, resTy,
 480:                               builder.createBitcast(shifTgt, resTy), shiftAmt,
 481:                               shiftLeft);
 482: }
 483: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitCommonNeonShift`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitCommonNeonShift`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 484-493
```cpp
 484: // Right-shift a vector by a constant.
 485: static mlir::Value emitNeonRShiftImm(CIRGenFunction &cgf, mlir::Value shiftVec,
 486:                                      mlir::Value shiftVal,
 487:                                      cir::VectorType vecTy, bool usgn,
 488:                                      mlir::Location loc) {
 489:   CIRGenBuilderTy &builder = cgf.getBuilder();
 490:   int64_t shiftAmt = getIntValueFromConstOp(shiftVal);
 491:   int eltSize =
 492:       cgf.cgm.getDataLayout().getTypeSizeInBits(vecTy.getElementType());
 493: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitNeonRShiftImm`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitNeonRShiftImm`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 494-510
```cpp
 494:   shiftVec = builder.createBitcast(shiftVec, vecTy);
 495:   // lshr/ashr are undefined when the shift amount is equal to the vector
 496:   // element size.
 497:   if (shiftAmt == eltSize) {
 498:     if (usgn) {
 499:       // Right-shifting an unsigned value by its size yields 0.
 500:       return builder.getZero(loc, vecTy);
 501:     }
 502:     // Right-shifting a signed value by its size is equivalent
 503:     // to a shift of size-1.
 504:     --shiftAmt;
 505:     shiftVal = builder.getConstInt(loc, vecTy.getElementType(), shiftAmt);
 506:   }
 507:   return emitCommonNeonShift(builder, loc, vecTy, shiftVec, shiftVal,
 508:                              /*shiftLeft=*/false);
 509: }
 510: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 511-515
```cpp
 511: static cir::VectorType getIntVecFromVecTy(CIRGenBuilderTy &builder,
 512:                                           cir::VectorType vecTy) {
 513:   if (!cir::isAnyFloatingPointType(vecTy.getElementType()))
 514:     return vecTy;
 515: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getIntVecFromVecTy`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getIntVecFromVecTy`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 516-523
```cpp
 516:   if (mlir::isa<cir::SingleType>(vecTy.getElementType()))
 517:     return cir::VectorType::get(builder.getSInt32Ty(), vecTy.getSize());
 518:   if (mlir::isa<cir::DoubleType>(vecTy.getElementType()))
 519:     return cir::VectorType::get(builder.getSInt64Ty(), vecTy.getSize());
 520:   llvm_unreachable(
 521:       "Unsupported element type in getVecOfIntTypeWithSameEltWidth");
 522: }
 523: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 524-530
```cpp
 524: static mlir::Value emitCommonNeonBuiltinExpr(
 525:     CIRGenFunction &cgf, unsigned builtinID, unsigned llvmIntrinsic,
 526:     unsigned altLLVMIntrinsic, const char *nameHint, unsigned modifier,
 527:     const CallExpr *expr, llvm::SmallVectorImpl<mlir::Value> &ops) {
 528:   mlir::Location loc = cgf.getLoc(expr->getExprLoc());
 529:   clang::ASTContext &ctx = cgf.getContext();
 530: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitCommonNeonBuiltinExpr`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitCommonNeonBuiltinExpr`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 531-538
```cpp
 531:   // Extract the trailing immediate argument that encodes the type discriminator
 532:   // for this overloaded intrinsic.
 533:   // TODO: Move to the parent code that takes care of argument processing.
 534:   const clang::Expr *arg = expr->getArg(expr->getNumArgs() - 1);
 535:   std::optional<llvm::APSInt> neonTypeConst = arg->getIntegerConstantExpr(ctx);
 536:   if (!neonTypeConst)
 537:     return nullptr;
 538: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 539-544
```cpp
 539:   // Determine the type of this overloaded NEON intrinsic.
 540:   NeonTypeFlags neonType(neonTypeConst->getZExtValue());
 541:   const bool isUnsigned = neonType.isUnsigned();
 542:   const bool hasLegalHalfType = cgf.getTarget().hasFastHalfType();
 543:   const bool usgn = neonType.isUnsigned();
 544: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `neonType`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `neonType`。

### Lines 545-551
```cpp
 545:   // The value of allowBFloatArgsAndRet is true for AArch64, but it should
 546:   // come from ABI info.
 547:   // TODO(cir): Use ABInfo to extract this information
 548:   const bool allowBFloatArgsAndRet = cgf.getTarget().hasFastHalfType();
 549:   // FIXME
 550:   // getTargetHooks().getABIInfo().allowBFloatArgsAndRet();
 551: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 552-557
```cpp
 552:   cir::VectorType vTy = getNeonType(&cgf, neonType, loc, hasLegalHalfType,
 553:                                     false, allowBFloatArgsAndRet);
 554:   cir::VectorType ty = vTy;
 555:   if (!ty)
 556:     return nullptr;
 557: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 558-575
```cpp
 558:   switch (builtinID) {
 559:   case NEON::BI__builtin_neon_splat_lane_v:
 560:   case NEON::BI__builtin_neon_splat_laneq_v:
 561:   case NEON::BI__builtin_neon_splatq_lane_v:
 562:   case NEON::BI__builtin_neon_splatq_laneq_v: {
 563:     uint64_t numElements = vTy.getSize();
 564:     if (builtinID == NEON::BI__builtin_neon_splatq_lane_v)
 565:       numElements *= 2;
 566:     if (builtinID == NEON::BI__builtin_neon_splat_laneq_v)
 567:       numElements /= 2;
 568:     ops[0] = cgf.getBuilder().createBitcast(loc, ops[0], vTy);
 569:     return emitNeonSplat(cgf.getBuilder(), loc, ops[0], ops[1], numElements);
 570:   }
 571:   case NEON::BI__builtin_neon_vpadd_v:
 572:   case NEON::BI__builtin_neon_vpaddq_v:
 573:   case NEON::BI__builtin_neon_vabs_v:
 574:   case NEON::BI__builtin_neon_vabsq_v:
 575:   case NEON::BI__builtin_neon_vadd_v:
```
- **EN**: This block contains local control-flow decisions or iterative processing. A switch statement is used to dispatch behavior across enumerated cases or kinds. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 576-593
```cpp
 576:   case NEON::BI__builtin_neon_vaddq_v:
 577:   case NEON::BI__builtin_neon_vaddhn_v:
 578:   case NEON::BI__builtin_neon_vcale_v:
 579:   case NEON::BI__builtin_neon_vcaleq_v:
 580:   case NEON::BI__builtin_neon_vcalt_v:
 581:   case NEON::BI__builtin_neon_vcaltq_v:
 582:   case NEON::BI__builtin_neon_vcage_v:
 583:   case NEON::BI__builtin_neon_vcageq_v:
 584:   case NEON::BI__builtin_neon_vcagt_v:
 585:   case NEON::BI__builtin_neon_vcagtq_v:
 586:     cgf.cgm.errorNYI(expr->getSourceRange(),
 587:                      std::string("unimplemented AArch64 builtin call: ") +
 588:                          ctx.BuiltinInfo.getName(builtinID));
 589:     return mlir::Value{};
 590:   case NEON::BI__builtin_neon_vceqz_v:
 591:   case NEON::BI__builtin_neon_vceqzq_v:
 592:     return emitAArch64CompareBuiltinExpr(cgf, cgf.getBuilder(), loc, ops[0],
 593:                                          vTy, cir::CmpOpKind::eq);
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `emitAArch64CompareBuiltinExpr`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`emitAArch64CompareBuiltinExpr`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 594-611
```cpp
 594:   case NEON::BI__builtin_neon_vcgez_v:
 595:   case NEON::BI__builtin_neon_vcgezq_v:
 596:   case NEON::BI__builtin_neon_vclez_v:
 597:   case NEON::BI__builtin_neon_vclezq_v:
 598:   case NEON::BI__builtin_neon_vcgtz_v:
 599:   case NEON::BI__builtin_neon_vcgtzq_v:
 600:   case NEON::BI__builtin_neon_vcltz_v:
 601:   case NEON::BI__builtin_neon_vcltzq_v:
 602:   case NEON::BI__builtin_neon_vclz_v:
 603:   case NEON::BI__builtin_neon_vclzq_v:
 604:   case NEON::BI__builtin_neon_vcvt_f32_v:
 605:   case NEON::BI__builtin_neon_vcvtq_f32_v:
 606:   case NEON::BI__builtin_neon_vcvt_f16_s16:
 607:   case NEON::BI__builtin_neon_vcvt_f16_u16:
 608:   case NEON::BI__builtin_neon_vcvtq_f16_s16:
 609:   case NEON::BI__builtin_neon_vcvtq_f16_u16:
 610:   case NEON::BI__builtin_neon_vcvt_n_f16_s16:
 611:   case NEON::BI__builtin_neon_vcvt_n_f16_u16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 612-629
```cpp
 612:   case NEON::BI__builtin_neon_vcvtq_n_f16_s16:
 613:   case NEON::BI__builtin_neon_vcvtq_n_f16_u16:
 614:     cgf.cgm.errorNYI(expr->getSourceRange(),
 615:                      std::string("unimplemented AArch64 builtin call: ") +
 616:                          ctx.BuiltinInfo.getName(builtinID));
 617:     return mlir::Value{};
 618:   case NEON::BI__builtin_neon_vcvt_n_f32_v:
 619:   case NEON::BI__builtin_neon_vcvt_n_f64_v:
 620:   case NEON::BI__builtin_neon_vcvtq_n_f32_v:
 621:   case NEON::BI__builtin_neon_vcvtq_n_f64_v: {
 622:     // The constant argument to an _n_ intrinsic always is Int32Ty.
 623:     mlir::Type cstIntTy = cgf.sInt32Ty;
 624:     llvm::StringRef llvmIntrName =
 625:         getLLVMIntrNameNoPrefix(static_cast<llvm::Intrinsic::ID>(
 626:             usgn ? llvmIntrinsic : altLLVMIntrinsic));
 627:     return emitNeonCall(cgf.getCIRGenModule(), cgf.getBuilder(),
 628:                         /*argTypes=*/{vTy, cstIntTy}, ops, llvmIntrName,
 629:                         /*funcResTy=*/getFloatNeonType(cgf, neonType), loc);
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `getLLVMIntrNameNoPrefix`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`getLLVMIntrNameNoPrefix`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 630-647
```cpp
 630:   }
 631:   case NEON::BI__builtin_neon_vcvt_n_s16_f16:
 632:   case NEON::BI__builtin_neon_vcvt_n_s32_v:
 633:   case NEON::BI__builtin_neon_vcvt_n_u16_f16:
 634:   case NEON::BI__builtin_neon_vcvt_n_u32_v:
 635:   case NEON::BI__builtin_neon_vcvt_n_s64_v:
 636:   case NEON::BI__builtin_neon_vcvt_n_u64_v:
 637:   case NEON::BI__builtin_neon_vcvtq_n_s16_f16:
 638:   case NEON::BI__builtin_neon_vcvtq_n_s32_v:
 639:   case NEON::BI__builtin_neon_vcvtq_n_u16_f16:
 640:   case NEON::BI__builtin_neon_vcvtq_n_u32_v:
 641:   case NEON::BI__builtin_neon_vcvtq_n_s64_v:
 642:   case NEON::BI__builtin_neon_vcvtq_n_u64_v: {
 643:     // The constant argument to an _n_ intrinsic always is Int32Ty.
 644:     mlir::Type cstIntTy = cgf.sInt32Ty;
 645:     llvm::StringRef llvmIntrName = getLLVMIntrNameNoPrefix(
 646:         static_cast<llvm::Intrinsic::ID>(llvmIntrinsic));
 647:     return emitNeonCall(
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 648-665
```cpp
 648:         cgf.getCIRGenModule(), cgf.getBuilder(),
 649:         /*argTypes=*/{getFloatNeonType(cgf, neonType), cstIntTy}, ops,
 650:         llvmIntrName,
 651:         /*funcResTy=*/vTy, loc);
 652:   }
 653:   case NEON::BI__builtin_neon_vcvt_s32_v:
 654:   case NEON::BI__builtin_neon_vcvt_u32_v:
 655:   case NEON::BI__builtin_neon_vcvt_s64_v:
 656:   case NEON::BI__builtin_neon_vcvt_u64_v:
 657:   case NEON::BI__builtin_neon_vcvt_s16_f16:
 658:   case NEON::BI__builtin_neon_vcvt_u16_f16:
 659:   case NEON::BI__builtin_neon_vcvtq_s32_v:
 660:   case NEON::BI__builtin_neon_vcvtq_u32_v:
 661:   case NEON::BI__builtin_neon_vcvtq_s64_v:
 662:   case NEON::BI__builtin_neon_vcvtq_u64_v:
 663:   case NEON::BI__builtin_neon_vcvtq_s16_f16:
 664:   case NEON::BI__builtin_neon_vcvtq_u16_f16:
 665:   case NEON::BI__builtin_neon_vcvta_s16_f16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 666-683
```cpp
 666:   case NEON::BI__builtin_neon_vcvta_s32_v:
 667:   case NEON::BI__builtin_neon_vcvta_s64_v:
 668:   case NEON::BI__builtin_neon_vcvta_u16_f16:
 669:   case NEON::BI__builtin_neon_vcvta_u32_v:
 670:   case NEON::BI__builtin_neon_vcvta_u64_v:
 671:   case NEON::BI__builtin_neon_vcvtaq_s16_f16:
 672:   case NEON::BI__builtin_neon_vcvtaq_s32_v:
 673:   case NEON::BI__builtin_neon_vcvtaq_s64_v:
 674:   case NEON::BI__builtin_neon_vcvtaq_u16_f16:
 675:   case NEON::BI__builtin_neon_vcvtaq_u32_v:
 676:   case NEON::BI__builtin_neon_vcvtaq_u64_v:
 677:   case NEON::BI__builtin_neon_vcvtn_s16_f16:
 678:   case NEON::BI__builtin_neon_vcvtn_s32_v:
 679:   case NEON::BI__builtin_neon_vcvtn_s64_v:
 680:   case NEON::BI__builtin_neon_vcvtn_u16_f16:
 681:   case NEON::BI__builtin_neon_vcvtn_u32_v:
 682:   case NEON::BI__builtin_neon_vcvtn_u64_v:
 683:   case NEON::BI__builtin_neon_vcvtnq_s16_f16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 684-701
```cpp
 684:   case NEON::BI__builtin_neon_vcvtnq_s32_v:
 685:   case NEON::BI__builtin_neon_vcvtnq_s64_v:
 686:   case NEON::BI__builtin_neon_vcvtnq_u16_f16:
 687:   case NEON::BI__builtin_neon_vcvtnq_u32_v:
 688:   case NEON::BI__builtin_neon_vcvtnq_u64_v:
 689:   case NEON::BI__builtin_neon_vcvtp_s16_f16:
 690:   case NEON::BI__builtin_neon_vcvtp_s32_v:
 691:   case NEON::BI__builtin_neon_vcvtp_s64_v:
 692:   case NEON::BI__builtin_neon_vcvtp_u16_f16:
 693:   case NEON::BI__builtin_neon_vcvtp_u32_v:
 694:   case NEON::BI__builtin_neon_vcvtp_u64_v:
 695:   case NEON::BI__builtin_neon_vcvtpq_s16_f16:
 696:   case NEON::BI__builtin_neon_vcvtpq_s32_v:
 697:   case NEON::BI__builtin_neon_vcvtpq_s64_v:
 698:   case NEON::BI__builtin_neon_vcvtpq_u16_f16:
 699:   case NEON::BI__builtin_neon_vcvtpq_u32_v:
 700:   case NEON::BI__builtin_neon_vcvtpq_u64_v:
 701:   case NEON::BI__builtin_neon_vcvtm_s16_f16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 702-719
```cpp
 702:   case NEON::BI__builtin_neon_vcvtm_s32_v:
 703:   case NEON::BI__builtin_neon_vcvtm_s64_v:
 704:   case NEON::BI__builtin_neon_vcvtm_u16_f16:
 705:   case NEON::BI__builtin_neon_vcvtm_u32_v:
 706:   case NEON::BI__builtin_neon_vcvtm_u64_v:
 707:   case NEON::BI__builtin_neon_vcvtmq_s16_f16:
 708:   case NEON::BI__builtin_neon_vcvtmq_s32_v:
 709:   case NEON::BI__builtin_neon_vcvtmq_s64_v:
 710:   case NEON::BI__builtin_neon_vcvtmq_u16_f16:
 711:   case NEON::BI__builtin_neon_vcvtmq_u32_v:
 712:   case NEON::BI__builtin_neon_vcvtmq_u64_v:
 713:   case NEON::BI__builtin_neon_vcvtx_f32_v:
 714:   case NEON::BI__builtin_neon_vext_v:
 715:   case NEON::BI__builtin_neon_vextq_v:
 716:   case NEON::BI__builtin_neon_vfma_v:
 717:     cgf.cgm.errorNYI(expr->getSourceRange(),
 718:                      std::string("unimplemented AArch64 builtin call: ") +
 719:                          ctx.BuiltinInfo.getName(builtinID));
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。

### Lines 720-737
```cpp
 720:     return mlir::Value{};
 721:   case NEON::BI__builtin_neon_vfmaq_v: {
 722:     // NEON intrinsic: vfmaq(accumulator, multiplicand1, multiplicand2)
 723:     // LLVM intrinsic: fma(multiplicand1, multiplicand2, accumulator)
 724:     // Reorder arguments to match LLVM fma signature
 725:     mlir::Value op0 = cgf.getBuilder().createBitcast(ops[0], ty);
 726:     mlir::Value op1 = cgf.getBuilder().createBitcast(ops[1], ty);
 727:     mlir::Value op2 = cgf.getBuilder().createBitcast(ops[2], ty);
 728:     llvm::SmallVector<mlir::Value> fmaOps = {op1, op2, op0};
 729:     return emitCallMaybeConstrainedBuiltin(cgf.getBuilder(), loc, "fma", ty,
 730:                                            fmaOps);
 731:   }
 732:   case NEON::BI__builtin_neon_vld1_v:
 733:   case NEON::BI__builtin_neon_vld1q_v:
 734:   case NEON::BI__builtin_neon_vld1_x2_v:
 735:   case NEON::BI__builtin_neon_vld1q_x2_v:
 736:   case NEON::BI__builtin_neon_vld1_x3_v:
 737:   case NEON::BI__builtin_neon_vld1q_x3_v:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 738-755
```cpp
 738:   case NEON::BI__builtin_neon_vld1_x4_v:
 739:   case NEON::BI__builtin_neon_vld1q_x4_v:
 740:   case NEON::BI__builtin_neon_vld2_v:
 741:   case NEON::BI__builtin_neon_vld2q_v:
 742:   case NEON::BI__builtin_neon_vld3_v:
 743:   case NEON::BI__builtin_neon_vld3q_v:
 744:   case NEON::BI__builtin_neon_vld4_v:
 745:   case NEON::BI__builtin_neon_vld4q_v:
 746:   case NEON::BI__builtin_neon_vld2_dup_v:
 747:   case NEON::BI__builtin_neon_vld2q_dup_v:
 748:   case NEON::BI__builtin_neon_vld3_dup_v:
 749:   case NEON::BI__builtin_neon_vld3q_dup_v:
 750:   case NEON::BI__builtin_neon_vld4_dup_v:
 751:   case NEON::BI__builtin_neon_vld4q_dup_v:
 752:   case NEON::BI__builtin_neon_vld1_dup_v:
 753:   case NEON::BI__builtin_neon_vld1q_dup_v:
 754:   case NEON::BI__builtin_neon_vld2_lane_v:
 755:   case NEON::BI__builtin_neon_vld2q_lane_v:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 756-773
```cpp
 756:   case NEON::BI__builtin_neon_vld3_lane_v:
 757:   case NEON::BI__builtin_neon_vld3q_lane_v:
 758:   case NEON::BI__builtin_neon_vld4_lane_v:
 759:   case NEON::BI__builtin_neon_vld4q_lane_v:
 760:   case NEON::BI__builtin_neon_vmovl_v:
 761:   case NEON::BI__builtin_neon_vmovn_v:
 762:   case NEON::BI__builtin_neon_vmull_v:
 763:   case NEON::BI__builtin_neon_vpadal_v:
 764:   case NEON::BI__builtin_neon_vpadalq_v:
 765:     cgf.cgm.errorNYI(expr->getSourceRange(),
 766:                      std::string("Reached code-path for ARM builtin call ") +
 767:                          ctx.BuiltinInfo.getName(builtinID) +
 768:                          "(ARM builtins are not supported ATM)");
 769:     return mlir::Value{};
 770:   case NEON::BI__builtin_neon_vpaddl_v:
 771:   case NEON::BI__builtin_neon_vpaddlq_v: {
 772:     llvm::StringRef llvmIntrName =
 773:         getLLVMIntrNameNoPrefix(static_cast<llvm::Intrinsic::ID>(
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 774-791
```cpp
 774:             usgn ? llvmIntrinsic : altLLVMIntrinsic));
 775:     return emitNeonCall(cgf.getCIRGenModule(), cgf.getBuilder(),
 776:                         /*argTypes=*/{getNeonPairwiseWidenInputType(vTy, usgn)},
 777:                         ops, llvmIntrName,
 778:                         /*funcResTy=*/vTy, loc);
 779:   }
 780:   case NEON::BI__builtin_neon_vqdmlal_v:
 781:   case NEON::BI__builtin_neon_vqdmlsl_v:
 782:   case NEON::BI__builtin_neon_vqdmulhq_lane_v:
 783:   case NEON::BI__builtin_neon_vqdmulh_lane_v:
 784:   case NEON::BI__builtin_neon_vqrdmulhq_lane_v:
 785:   case NEON::BI__builtin_neon_vqrdmulh_lane_v:
 786:   case NEON::BI__builtin_neon_vqdmulhq_laneq_v:
 787:   case NEON::BI__builtin_neon_vqdmulh_laneq_v:
 788:   case NEON::BI__builtin_neon_vqrdmulhq_laneq_v:
 789:   case NEON::BI__builtin_neon_vqrdmulh_laneq_v:
 790:   case NEON::BI__builtin_neon_vqshl_n_v:
 791:   case NEON::BI__builtin_neon_vqshlq_n_v:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 792-809
```cpp
 792:   case NEON::BI__builtin_neon_vqshlu_n_v:
 793:   case NEON::BI__builtin_neon_vqshluq_n_v:
 794:   case NEON::BI__builtin_neon_vrecpe_v:
 795:   case NEON::BI__builtin_neon_vrecpeq_v:
 796:   case NEON::BI__builtin_neon_vrsqrte_v:
 797:   case NEON::BI__builtin_neon_vrsqrteq_v:
 798:   case NEON::BI__builtin_neon_vrndi_v:
 799:   case NEON::BI__builtin_neon_vrndiq_v:
 800:     cgf.cgm.errorNYI(expr->getSourceRange(),
 801:                      std::string("unimplemented AArch64 builtin call: ") +
 802:                          ctx.BuiltinInfo.getName(builtinID));
 803:     return mlir::Value{};
 804:   case NEON::BI__builtin_neon_vrshr_n_v:
 805:   case NEON::BI__builtin_neon_vrshrq_n_v: {
 806:     llvm::StringRef intrName =
 807:         usgn ? "aarch64.neon.urshl" : "aarch64.neon.srshl";
 808:     return emitNeonCall(
 809:         cgf.cgm, cgf.getBuilder(),
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 810-827
```cpp
 810:         {ty, usgn ? getSignChangedVectorType(cgf.getBuilder(), ty) : ty}, ops,
 811:         intrName, ty, loc, /*isConstrainedFPIntrinsic=*/false,
 812:         /*shift=*/1,
 813:         /*rightshift=*/true);
 814:   }
 815:   case NEON::BI__builtin_neon_vsha512hq_u64:
 816:   case NEON::BI__builtin_neon_vsha512h2q_u64:
 817:   case NEON::BI__builtin_neon_vsha512su0q_u64:
 818:   case NEON::BI__builtin_neon_vsha512su1q_u64:
 819:     cgf.cgm.errorNYI(expr->getSourceRange(),
 820:                      std::string("unimplemented AArch64 builtin call: ") +
 821:                          ctx.BuiltinInfo.getName(builtinID));
 822:     return mlir::Value{};
 823:   case NEON::BI__builtin_neon_vshl_n_v:
 824:   case NEON::BI__builtin_neon_vshlq_n_v:
 825:     return emitCommonNeonShift(cgf.getBuilder(), loc, vTy, ops[0], ops[1],
 826:                                /*shiftLeft=*/true);
 827:   case NEON::BI__builtin_neon_vshll_n_v:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `emitCommonNeonShift`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`emitCommonNeonShift`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 828-845
```cpp
 828:   case NEON::BI__builtin_neon_vshrn_n_v:
 829:     cgf.cgm.errorNYI(expr->getSourceRange(),
 830:                      std::string("unimplemented AArch64 builtin call: ") +
 831:                          ctx.BuiltinInfo.getName(builtinID));
 832:     return mlir::Value{};
 833:   case NEON::BI__builtin_neon_vshr_n_v:
 834:   case NEON::BI__builtin_neon_vshrq_n_v:
 835:     return emitNeonRShiftImm(cgf, ops[0], ops[1], vTy, isUnsigned, loc);
 836:   case NEON::BI__builtin_neon_vst1_v:
 837:   case NEON::BI__builtin_neon_vst1q_v:
 838:   case NEON::BI__builtin_neon_vst2_v:
 839:   case NEON::BI__builtin_neon_vst2q_v:
 840:   case NEON::BI__builtin_neon_vst3_v:
 841:   case NEON::BI__builtin_neon_vst3q_v:
 842:   case NEON::BI__builtin_neon_vst4_v:
 843:   case NEON::BI__builtin_neon_vst4q_v:
 844:   case NEON::BI__builtin_neon_vst2_lane_v:
 845:   case NEON::BI__builtin_neon_vst2q_lane_v:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `emitNeonRShiftImm`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`emitNeonRShiftImm`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 846-863
```cpp
 846:   case NEON::BI__builtin_neon_vst3_lane_v:
 847:   case NEON::BI__builtin_neon_vst3q_lane_v:
 848:   case NEON::BI__builtin_neon_vst4_lane_v:
 849:   case NEON::BI__builtin_neon_vst4q_lane_v:
 850:   case NEON::BI__builtin_neon_vsm3partw1q_u32:
 851:   case NEON::BI__builtin_neon_vsm3partw2q_u32:
 852:   case NEON::BI__builtin_neon_vsm3ss1q_u32:
 853:   case NEON::BI__builtin_neon_vsm4ekeyq_u32:
 854:   case NEON::BI__builtin_neon_vsm4eq_u32:
 855:   case NEON::BI__builtin_neon_vsm3tt1aq_u32:
 856:   case NEON::BI__builtin_neon_vsm3tt1bq_u32:
 857:   case NEON::BI__builtin_neon_vsm3tt2aq_u32:
 858:   case NEON::BI__builtin_neon_vsm3tt2bq_u32:
 859:   case NEON::BI__builtin_neon_vst1_x2_v:
 860:   case NEON::BI__builtin_neon_vst1q_x2_v:
 861:   case NEON::BI__builtin_neon_vst1_x3_v:
 862:   case NEON::BI__builtin_neon_vst1q_x3_v:
 863:   case NEON::BI__builtin_neon_vst1_x4_v:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 864-881
```cpp
 864:   case NEON::BI__builtin_neon_vst1q_x4_v:
 865:   case NEON::BI__builtin_neon_vsubhn_v:
 866:   case NEON::BI__builtin_neon_vtrn_v:
 867:   case NEON::BI__builtin_neon_vtrnq_v:
 868:   case NEON::BI__builtin_neon_vtst_v:
 869:   case NEON::BI__builtin_neon_vtstq_v:
 870:   case NEON::BI__builtin_neon_vuzp_v:
 871:   case NEON::BI__builtin_neon_vuzpq_v:
 872:   case NEON::BI__builtin_neon_vxarq_u64:
 873:   case NEON::BI__builtin_neon_vzip_v:
 874:   case NEON::BI__builtin_neon_vzipq_v:
 875:   case NEON::BI__builtin_neon_vdot_s32:
 876:   case NEON::BI__builtin_neon_vdot_u32:
 877:   case NEON::BI__builtin_neon_vdotq_s32:
 878:   case NEON::BI__builtin_neon_vdotq_u32:
 879:   case NEON::BI__builtin_neon_vfmlal_low_f16:
 880:   case NEON::BI__builtin_neon_vfmlalq_low_f16:
 881:   case NEON::BI__builtin_neon_vfmlsl_low_f16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 882-899
```cpp
 882:   case NEON::BI__builtin_neon_vfmlslq_low_f16:
 883:   case NEON::BI__builtin_neon_vfmlal_high_f16:
 884:   case NEON::BI__builtin_neon_vfmlalq_high_f16:
 885:   case NEON::BI__builtin_neon_vfmlsl_high_f16:
 886:   case NEON::BI__builtin_neon_vfmlslq_high_f16:
 887:   case NEON::BI__builtin_neon_vmmlaq_s32:
 888:   case NEON::BI__builtin_neon_vmmlaq_u32:
 889:     cgf.cgm.errorNYI(expr->getSourceRange(),
 890:                      std::string("unimplemented AArch64 builtin call: ") +
 891:                          ctx.BuiltinInfo.getName(builtinID));
 892:     return mlir::Value{};
 893:   case NEON::BI__builtin_neon_vmul_v:
 894:   case NEON::BI__builtin_neon_vmulq_v:
 895:     return cgf.getBuilder().emitIntrinsicCallOp(loc, "aarch64.neon.pmul", vTy,
 896:                                                 ops);
 897:   case NEON::BI__builtin_neon_vusmmlaq_s32:
 898:   case NEON::BI__builtin_neon_vusdot_s32:
 899:   case NEON::BI__builtin_neon_vusdotq_s32:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 900-908
```cpp
 900:   case NEON::BI__builtin_neon_vbfdot_f32:
 901:   case NEON::BI__builtin_neon_vbfdotq_f32:
 902:   case NEON::BI__builtin_neon___a32_vcvt_bf16_f32:
 903:     cgf.cgm.errorNYI(expr->getSourceRange(),
 904:                      std::string("unimplemented AArch64 builtin call: ") +
 905:                          ctx.BuiltinInfo.getName(builtinID));
 906:     return mlir::Value{};
 907:   }
 908: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 909-923
```cpp
 909:   // The switch stmt is intended to help catch NYI cases and will be removed
 910:   // once the CIR implementation is complete. Avoid adding specialized
 911:   // code in cases - that should only be required for a handful of examples.
 912:   switch (builtinID) {
 913:   default:
 914:     cgf.cgm.errorNYI(expr->getSourceRange(),
 915:                      std::string("unimplemented AArch64 builtin call: ") +
 916:                          cgf.getContext().BuiltinInfo.getName(builtinID));
 917:     break;
 918:   case NEON::BI__builtin_neon_vshl_v:
 919:   case NEON::BI__builtin_neon_vshlq_v: {
 920:     llvm::StringRef llvmIntrName =
 921:         getLLVMIntrNameNoPrefix(static_cast<llvm::Intrinsic::ID>(
 922:             usgn ? llvmIntrinsic : altLLVMIntrinsic));
 923: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `getLLVMIntrNameNoPrefix`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`getLLVMIntrNameNoPrefix`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 924-932
```cpp
 924:     mlir::Value result =
 925:         emitNeonCall(cgf.getCIRGenModule(), cgf.getBuilder(),
 926:                      /*argTypes=*/{vTy, vTy}, ops, llvmIntrName,
 927:                      /*funcResTy=*/vTy, loc);
 928:     mlir::Type resultType = cgf.convertType(expr->getType());
 929:     return cgf.getBuilder().createBitcast(result, resultType);
 930:   }
 931:   }
 932: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 933-936
```cpp
 933:   // NYI
 934:   return nullptr;
 935: }
 936: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 937-945
```cpp
 937: bool CIRGenFunction::getAArch64SVEProcessedOperands(
 938:     unsigned builtinID, const CallExpr *expr, SmallVectorImpl<mlir::Value> &ops,
 939:     SVETypeFlags typeFlags) {
 940:   // Find out if any arguments are required to be integer constant expressions.
 941:   unsigned iceArguments = 0;
 942:   ASTContext::GetBuiltinTypeError error;
 943:   getContext().GetBuiltinType(builtinID, error, &iceArguments);
 944:   assert(error == ASTContext::GE_None && "Should not codegen an error");
 945: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenFunction::getAArch64SVEProcessedOperands`, `getContext`, `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenFunction::getAArch64SVEProcessedOperands`、`getContext`、`assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 946-949
```cpp
 946:   for (unsigned i = 0, e = expr->getNumArgs(); i != e; i++) {
 947:     bool isIce = iceArguments & (1 << i);
 948:     mlir::Value arg = emitScalarExpr(expr->getArg(i));
 949: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 950-958
```cpp
 950:     if (isIce) {
 951:       cgm.errorNYI(expr->getSourceRange(),
 952:                    std::string("unimplemented AArch64 builtin call: ") +
 953:                        getContext().BuiltinInfo.getName(builtinID));
 954:     }
 955: 
 956:     // FIXME: Handle types like svint16x2_t, which are currently incorrectly
 957:     // converted to i32. These should be treated as structs and unpacked.
 958: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 959-963
```cpp
 959:     ops.push_back(arg);
 960:   }
 961:   return true;
 962: }
 963: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 964-971
```cpp
 964: // Reinterpret the input predicate so that it can be used to correctly isolate
 965: // the elements of the specified datatype.
 966: mlir::Value CIRGenFunction::emitSVEPredicateCast(mlir::Value pred,
 967:                                                  unsigned minNumElts,
 968:                                                  mlir::Location loc) {
 969: 
 970:   // TODO: Handle "aarch64.svcount" once we get round to supporting SME.
 971: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenFunction::emitSVEPredicateCast`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenFunction::emitSVEPredicateCast`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 972-976
```cpp
 972:   auto retTy = cir::VectorType::get(builder.getUIntNTy(1), minNumElts,
 973:                                     /*is_scalable=*/true);
 974:   if (pred.getType() == retTy)
 975:     return pred;
 976: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 977-991
```cpp
 977:   llvm::Intrinsic::ID intID;
 978:   switch (minNumElts) {
 979:   default:
 980:     llvm_unreachable("unsupported element count!");
 981:   case 1:
 982:   case 2:
 983:   case 4:
 984:   case 8:
 985:     intID = Intrinsic::aarch64_sve_convert_from_svbool;
 986:     break;
 987:   case 16:
 988:     intID = Intrinsic::aarch64_sve_convert_to_svbool;
 989:     break;
 990:   }
 991: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 992-998
```cpp
 992:   llvm::StringRef llvmIntrName = getLLVMIntrNameNoPrefix(intID);
 993:   auto call = builder.emitIntrinsicCallOp(loc, llvmIntrName, retTy,
 994:                                           mlir::ValueRange{pred});
 995:   assert(call.getType() == retTy && "Unexpected return type!");
 996:   return call;
 997: }
 998: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 999-1009
```cpp
 999: //===----------------------------------------------------------------------===//
1000: //  SVE helpers
1001: //===----------------------------------------------------------------------===//
1002: // Get the minimum number of elements in an SVE vector for the given element
1003: // type. The actual number of elements in the vector would be an integer (power
1004: // of two) multiple of this value.
1005: static unsigned getSVEMinEltCount(clang::SVETypeFlags::EltType sveType) {
1006:   switch (sveType) {
1007:   default:
1008:     llvm_unreachable("Invalid SVETypeFlag!");
1009: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getSVEMinEltCount`, `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getSVEMinEltCount`、`llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 1010-1018
```cpp
1010:   case SVETypeFlags::EltTyInt8:
1011:     return 16;
1012:   case SVETypeFlags::EltTyInt16:
1013:     return 8;
1014:   case SVETypeFlags::EltTyInt32:
1015:     return 4;
1016:   case SVETypeFlags::EltTyInt64:
1017:     return 2;
1018: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1019-1028
```cpp
1019:   case SVETypeFlags::EltTyMFloat8:
1020:     return 16;
1021:   case SVETypeFlags::EltTyFloat16:
1022:   case SVETypeFlags::EltTyBFloat16:
1023:     return 8;
1024:   case SVETypeFlags::EltTyFloat32:
1025:     return 4;
1026:   case SVETypeFlags::EltTyFloat64:
1027:     return 2;
1028: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1029-1042
```cpp
1029:   case SVETypeFlags::EltTyBool8:
1030:     return 16;
1031:   case SVETypeFlags::EltTyBool16:
1032:     return 8;
1033:   case SVETypeFlags::EltTyBool32:
1034:     return 4;
1035:   case SVETypeFlags::EltTyBool64:
1036:     return 2;
1037:   }
1038: }
1039: 
1040: // TODO(cir): Share with OGCG
1041: constexpr unsigned sveBitsPerBlock = 128;
1042: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1043-1049
```cpp
1043: static cir::VectorType getSVEVectorForElementType(CIRGenModule &cgm,
1044:                                                   mlir::Type eltTy) {
1045:   unsigned numElts =
1046:       sveBitsPerBlock / cgm.getDataLayout().getTypeSizeInBits(eltTy);
1047:   return cir::VectorType::get(eltTy, numElts, /*is_scalable=*/true);
1048: }
1049: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getSVEVectorForElementType`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getSVEVectorForElementType`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1050-1057
```cpp
1050: //===----------------------------------------------------------------------===//
1051: //  SVE helpers
1052: //===----------------------------------------------------------------------===//
1053: std::optional<mlir::Value>
1054: CIRGenFunction::emitAArch64SVEBuiltinExpr(unsigned builtinID,
1055:                                           const CallExpr *expr) {
1056:   mlir::Type ty = convertType(expr->getType());
1057: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenFunction::emitAArch64SVEBuiltinExpr`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenFunction::emitAArch64SVEBuiltinExpr`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1058-1073
```cpp
1058:   if (builtinID >= SVE::BI__builtin_sve_reinterpret_s8_s8 &&
1059:       builtinID <= SVE::BI__builtin_sve_reinterpret_f64_f64_x4) {
1060:     cgm.errorNYI(expr->getSourceRange(),
1061:                  std::string("unimplemented AArch64 builtin call: ") +
1062:                      getContext().BuiltinInfo.getName(builtinID));
1063:     return mlir::Value{};
1064:   }
1065: 
1066:   assert(!cir::MissingFeatures::aarch64SVEIntrinsics());
1067: 
1068:   auto *builtinIntrInfo = findARMVectorIntrinsicInMap(
1069:       aarch64SVEIntrinsicMap, builtinID, aarch64SVEIntrinsicsProvenSorted);
1070: 
1071:   // The operands of the builtin call
1072:   llvm::SmallVector<mlir::Value> ops;
1073: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1074-1078
```cpp
1074:   SVETypeFlags typeFlags(builtinIntrInfo->TypeModifier);
1075:   if (!CIRGenFunction::getAArch64SVEProcessedOperands(builtinID, expr, ops,
1076:                                                       typeFlags))
1077:     return mlir::Value{};
1078: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `typeFlags`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `typeFlags`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1079-1090
```cpp
1079:   if (typeFlags.isLoad() || typeFlags.isStore() || typeFlags.isGatherLoad() ||
1080:       typeFlags.isScatterStore() || typeFlags.isPrefetch() ||
1081:       typeFlags.isGatherPrefetch() || typeFlags.isStructLoad() ||
1082:       typeFlags.isStructStore() || typeFlags.isTupleSet() ||
1083:       typeFlags.isTupleGet() || typeFlags.isTupleCreate() ||
1084:       typeFlags.isUndef())
1085:     cgm.errorNYI(expr->getSourceRange(),
1086:                  std::string("unimplemented AArch64 builtin call: ") +
1087:                      getContext().BuiltinInfo.getName(builtinID));
1088: 
1089:   mlir::Location loc = getLoc(expr->getExprLoc());
1090: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1091-1099
```cpp
1091:   // Handle built-ins for which there is a corresponding LLVM Intrinsic.
1092:   // -------------------------------------------------------------------
1093:   if (builtinIntrInfo->LLVMIntrinsic != 0) {
1094:     // Emit set FPMR for intrinsics that require it.
1095:     if (typeFlags.setsFPMR())
1096:       cgm.errorNYI(expr->getSourceRange(),
1097:                    std::string("unimplemented AArch64 builtin call: ") +
1098:                        getContext().BuiltinInfo.getName(builtinID));
1099: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 1100-1106
```cpp
1100:     // Zero-ing predication
1101:     if (typeFlags.getMergeType() == SVETypeFlags::MergeZeroExp) {
1102:       auto null = builder.getNullValue(convertType(expr->getType()),
1103:                                        getLoc(expr->getExprLoc()));
1104:       ops.insert(ops.begin(), null);
1105:     }
1106: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getLoc`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getLoc`。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 1107-1110
```cpp
1107:     if (typeFlags.getMergeType() == SVETypeFlags::MergeAnyExp)
1108:       ops.insert(ops.begin(),
1109:                  builder.getConstant(loc, cir::UndefAttr::get(ty)));
1110: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1111-1121
```cpp
1111:     // Some ACLE builtins leave out the argument to specify the predicate
1112:     // pattern, which is expected to be expanded to an SV_ALL pattern.
1113:     if (typeFlags.isAppendSVALL())
1114:       cgm.errorNYI(expr->getSourceRange(),
1115:                    std::string("unimplemented AArch64 builtin call: ") +
1116:                        getContext().BuiltinInfo.getName(builtinID));
1117:     if (typeFlags.isInsertOp1SVALL())
1118:       cgm.errorNYI(expr->getSourceRange(),
1119:                    std::string("unimplemented AArch64 builtin call: ") +
1120:                        getContext().BuiltinInfo.getName(builtinID));
1121: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 1122-1129
```cpp
1122:     // Predicates must match the main datatype.
1123:     for (mlir::Value &op : ops)
1124:       if (auto predTy = dyn_cast<cir::VectorType>(op.getType()))
1125:         if (auto cirInt = dyn_cast<cir::IntType>(predTy.getElementType()))
1126:           if (cirInt.getWidth() == 1)
1127:             op = emitSVEPredicateCast(
1128:                 op, getSVEMinEltCount(typeFlags.getEltType()), loc);
1129: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1130-1137
```cpp
1130:     // Splat scalar operand to vector (intrinsics with _n infix)
1131:     if (typeFlags.hasSplatOperand()) {
1132:       unsigned opNo = typeFlags.getSplatOperand();
1133:       ops[opNo] = cir::VecSplatOp::create(
1134:           builder, loc, getSVEVectorForElementType(cgm, ops[opNo].getType()),
1135:           ops[opNo]);
1136:     }
1137: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getSVEVectorForElementType`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getSVEVectorForElementType`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1138-1155
```cpp
1138:     if (typeFlags.isReverseCompare())
1139:       cgm.errorNYI(expr->getSourceRange(),
1140:                    std::string("unimplemented AArch64 builtin call: ") +
1141:                        getContext().BuiltinInfo.getName(builtinID));
1142:     if (typeFlags.isReverseUSDOT())
1143:       cgm.errorNYI(expr->getSourceRange(),
1144:                    std::string("unimplemented AArch64 builtin call: ") +
1145:                        getContext().BuiltinInfo.getName(builtinID));
1146:     if (typeFlags.isReverseMergeAnyBinOp() &&
1147:         typeFlags.getMergeType() == SVETypeFlags::MergeAny)
1148:       cgm.errorNYI(expr->getSourceRange(),
1149:                    std::string("unimplemented AArch64 builtin call: ") +
1150:                        getContext().BuiltinInfo.getName(builtinID));
1151:     if (typeFlags.isReverseMergeAnyAccOp() &&
1152:         typeFlags.getMergeType() == SVETypeFlags::MergeAny)
1153:       cgm.errorNYI(expr->getSourceRange(),
1154:                    std::string("unimplemented AArch64 builtin call: ") +
1155:                        getContext().BuiltinInfo.getName(builtinID));
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 1156-1163
```cpp
1156: 
1157:     // Predicated intrinsics with _z suffix.
1158:     if (typeFlags.getMergeType() == SVETypeFlags::MergeZero) {
1159:       cgm.errorNYI(expr->getSourceRange(),
1160:                    std::string("unimplemented AArch64 builtin call: ") +
1161:                        getContext().BuiltinInfo.getName(builtinID));
1162:     }
1163: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 1164-1167
```cpp
1164:     llvm::StringRef llvmIntrName = getLLVMIntrNameNoPrefix(
1165:         static_cast<llvm::Intrinsic::ID>(builtinIntrInfo->LLVMIntrinsic));
1166:     auto retTy = convertType(expr->getType());
1167: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1168-1172
```cpp
1168:     auto call = builder.emitIntrinsicCallOp(loc, llvmIntrName, retTy,
1169:                                             mlir::ValueRange{ops});
1170:     if (call.getType() == retTy)
1171:       return call;
1172: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1173-1183
```cpp
1173:     // Predicate results must be converted to svbool_t.
1174:     if (isa<mlir::VectorType>(retTy) &&
1175:         cast<mlir::VectorType>(retTy).isScalable())
1176:       cgm.errorNYI(expr->getSourceRange(),
1177:                    std::string("unimplemented AArch64 builtin call: ") +
1178:                        getContext().BuiltinInfo.getName(builtinID));
1179:     // TODO Handle struct types, e.g. svint8x2_t (update the converter first).
1180: 
1181:     llvm_unreachable("unsupported element count!");
1182:   }
1183: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Notable callable symbols here include `llvm_unreachable`. It introduces or references types such as `types`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 其中值得关注的可调用符号包括 `llvm_unreachable`。 它引入或引用了诸如 `types` 等类型。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1184-1189
```cpp
1184:   // Handle the remaining built-ins.
1185:   // -------------------------------
1186:   switch (builtinID) {
1187:   default:
1188:     return std::nullopt;
1189: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. A switch statement is used to dispatch behavior across enumerated cases or kinds. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1190-1207
```cpp
1190:   case SVE::BI__builtin_sve_svreinterpret_b:
1191:   case SVE::BI__builtin_sve_svreinterpret_c:
1192:   case SVE::BI__builtin_sve_svpsel_lane_b8:
1193:   case SVE::BI__builtin_sve_svpsel_lane_b16:
1194:   case SVE::BI__builtin_sve_svpsel_lane_b32:
1195:   case SVE::BI__builtin_sve_svpsel_lane_b64:
1196:   case SVE::BI__builtin_sve_svpsel_lane_c8:
1197:   case SVE::BI__builtin_sve_svpsel_lane_c16:
1198:   case SVE::BI__builtin_sve_svpsel_lane_c32:
1199:   case SVE::BI__builtin_sve_svpsel_lane_c64:
1200:   case SVE::BI__builtin_sve_svmov_b_z:
1201:   case SVE::BI__builtin_sve_svnot_b_z:
1202:   case SVE::BI__builtin_sve_svmovlb_u16:
1203:   case SVE::BI__builtin_sve_svmovlb_u32:
1204:   case SVE::BI__builtin_sve_svmovlb_u64:
1205:   case SVE::BI__builtin_sve_svmovlb_s16:
1206:   case SVE::BI__builtin_sve_svmovlb_s32:
1207:   case SVE::BI__builtin_sve_svmovlb_s64:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1208-1222
```cpp
1208:   case SVE::BI__builtin_sve_svmovlt_u16:
1209:   case SVE::BI__builtin_sve_svmovlt_u32:
1210:   case SVE::BI__builtin_sve_svmovlt_u64:
1211:   case SVE::BI__builtin_sve_svmovlt_s16:
1212:   case SVE::BI__builtin_sve_svmovlt_s32:
1213:   case SVE::BI__builtin_sve_svmovlt_s64:
1214:   case SVE::BI__builtin_sve_svpmullt_u16:
1215:   case SVE::BI__builtin_sve_svpmullt_u64:
1216:   case SVE::BI__builtin_sve_svpmullt_n_u16:
1217:   case SVE::BI__builtin_sve_svpmullt_n_u64:
1218:   case SVE::BI__builtin_sve_svpmullb_u16:
1219:   case SVE::BI__builtin_sve_svpmullb_u64:
1220:   case SVE::BI__builtin_sve_svpmullb_n_u16:
1221:   case SVE::BI__builtin_sve_svpmullb_n_u64:
1222: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1223-1227
```cpp
1223:   case SVE::BI__builtin_sve_svdup_n_b8:
1224:   case SVE::BI__builtin_sve_svdup_n_b16:
1225:   case SVE::BI__builtin_sve_svdup_n_b32:
1226:   case SVE::BI__builtin_sve_svdup_n_b64:
1227: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1228-1245
```cpp
1228:   case SVE::BI__builtin_sve_svdupq_n_b8:
1229:   case SVE::BI__builtin_sve_svdupq_n_b16:
1230:   case SVE::BI__builtin_sve_svdupq_n_b32:
1231:   case SVE::BI__builtin_sve_svdupq_n_b64:
1232:   case SVE::BI__builtin_sve_svdupq_n_u8:
1233:   case SVE::BI__builtin_sve_svdupq_n_s8:
1234:   case SVE::BI__builtin_sve_svdupq_n_u64:
1235:   case SVE::BI__builtin_sve_svdupq_n_f64:
1236:   case SVE::BI__builtin_sve_svdupq_n_s64:
1237:   case SVE::BI__builtin_sve_svdupq_n_u16:
1238:   case SVE::BI__builtin_sve_svdupq_n_f16:
1239:   case SVE::BI__builtin_sve_svdupq_n_bf16:
1240:   case SVE::BI__builtin_sve_svdupq_n_s16:
1241:   case SVE::BI__builtin_sve_svdupq_n_u32:
1242:   case SVE::BI__builtin_sve_svdupq_n_f32:
1243:   case SVE::BI__builtin_sve_svdupq_n_s32:
1244:   case SVE::BI__builtin_sve_svpfalse_b:
1245:   case SVE::BI__builtin_sve_svpfalse_c:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1246-1250
```cpp
1246:     cgm.errorNYI(expr->getSourceRange(),
1247:                  std::string("unimplemented AArch64 builtin call: ") +
1248:                      getContext().BuiltinInfo.getName(builtinID));
1249:     return mlir::Value{};
1250: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1251-1254
```cpp
1251:   case SVE::BI__builtin_sve_svlen_u8:
1252:   case SVE::BI__builtin_sve_svlen_s8:
1253:     return genVscaleTimesFactor(loc, builder, convertType(expr->getType()), 16);
1254: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `genVscaleTimesFactor`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `genVscaleTimesFactor`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1255-1260
```cpp
1255:   case SVE::BI__builtin_sve_svlen_u16:
1256:   case SVE::BI__builtin_sve_svlen_s16:
1257:   case SVE::BI__builtin_sve_svlen_f16:
1258:   case SVE::BI__builtin_sve_svlen_bf16:
1259:     return genVscaleTimesFactor(loc, builder, convertType(expr->getType()), 8);
1260: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `genVscaleTimesFactor`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `genVscaleTimesFactor`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1261-1265
```cpp
1261:   case SVE::BI__builtin_sve_svlen_u32:
1262:   case SVE::BI__builtin_sve_svlen_s32:
1263:   case SVE::BI__builtin_sve_svlen_f32:
1264:     return genVscaleTimesFactor(loc, builder, convertType(expr->getType()), 4);
1265: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `genVscaleTimesFactor`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `genVscaleTimesFactor`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1266-1270
```cpp
1266:   case SVE::BI__builtin_sve_svlen_u64:
1267:   case SVE::BI__builtin_sve_svlen_s64:
1268:   case SVE::BI__builtin_sve_svlen_f64:
1269:     return genVscaleTimesFactor(loc, builder, convertType(expr->getType()), 2);
1270: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `genVscaleTimesFactor`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `genVscaleTimesFactor`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1271-1288
```cpp
1271:   case SVE::BI__builtin_sve_svtbl2_u8:
1272:   case SVE::BI__builtin_sve_svtbl2_s8:
1273:   case SVE::BI__builtin_sve_svtbl2_u16:
1274:   case SVE::BI__builtin_sve_svtbl2_s16:
1275:   case SVE::BI__builtin_sve_svtbl2_u32:
1276:   case SVE::BI__builtin_sve_svtbl2_s32:
1277:   case SVE::BI__builtin_sve_svtbl2_u64:
1278:   case SVE::BI__builtin_sve_svtbl2_s64:
1279:   case SVE::BI__builtin_sve_svtbl2_f16:
1280:   case SVE::BI__builtin_sve_svtbl2_bf16:
1281:   case SVE::BI__builtin_sve_svtbl2_f32:
1282:   case SVE::BI__builtin_sve_svtbl2_f64:
1283:   case SVE::BI__builtin_sve_svset_neonq_s8:
1284:   case SVE::BI__builtin_sve_svset_neonq_s16:
1285:   case SVE::BI__builtin_sve_svset_neonq_s32:
1286:   case SVE::BI__builtin_sve_svset_neonq_s64:
1287:   case SVE::BI__builtin_sve_svset_neonq_u8:
1288:   case SVE::BI__builtin_sve_svset_neonq_u16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1289-1306
```cpp
1289:   case SVE::BI__builtin_sve_svset_neonq_u32:
1290:   case SVE::BI__builtin_sve_svset_neonq_u64:
1291:   case SVE::BI__builtin_sve_svset_neonq_f16:
1292:   case SVE::BI__builtin_sve_svset_neonq_f32:
1293:   case SVE::BI__builtin_sve_svset_neonq_f64:
1294:   case SVE::BI__builtin_sve_svset_neonq_bf16:
1295:   case SVE::BI__builtin_sve_svget_neonq_s8:
1296:   case SVE::BI__builtin_sve_svget_neonq_s16:
1297:   case SVE::BI__builtin_sve_svget_neonq_s32:
1298:   case SVE::BI__builtin_sve_svget_neonq_s64:
1299:   case SVE::BI__builtin_sve_svget_neonq_u8:
1300:   case SVE::BI__builtin_sve_svget_neonq_u16:
1301:   case SVE::BI__builtin_sve_svget_neonq_u32:
1302:   case SVE::BI__builtin_sve_svget_neonq_u64:
1303:   case SVE::BI__builtin_sve_svget_neonq_f16:
1304:   case SVE::BI__builtin_sve_svget_neonq_f32:
1305:   case SVE::BI__builtin_sve_svget_neonq_f64:
1306:   case SVE::BI__builtin_sve_svget_neonq_bf16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1307-1327
```cpp
1307:   case SVE::BI__builtin_sve_svdup_neonq_s8:
1308:   case SVE::BI__builtin_sve_svdup_neonq_s16:
1309:   case SVE::BI__builtin_sve_svdup_neonq_s32:
1310:   case SVE::BI__builtin_sve_svdup_neonq_s64:
1311:   case SVE::BI__builtin_sve_svdup_neonq_u8:
1312:   case SVE::BI__builtin_sve_svdup_neonq_u16:
1313:   case SVE::BI__builtin_sve_svdup_neonq_u32:
1314:   case SVE::BI__builtin_sve_svdup_neonq_u64:
1315:   case SVE::BI__builtin_sve_svdup_neonq_f16:
1316:   case SVE::BI__builtin_sve_svdup_neonq_f32:
1317:   case SVE::BI__builtin_sve_svdup_neonq_f64:
1318:   case SVE::BI__builtin_sve_svdup_neonq_bf16:
1319:     cgm.errorNYI(expr->getSourceRange(),
1320:                  std::string("unimplemented AArch64 builtin call: ") +
1321:                      getContext().BuiltinInfo.getName(builtinID));
1322:     return mlir::Value{};
1323:   }
1324: 
1325:   // Unreachable: All cases in the switch above return.
1326: }
1327: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1328-1332
```cpp
1328: std::optional<mlir::Value>
1329: CIRGenFunction::emitAArch64SMEBuiltinExpr(unsigned builtinID,
1330:                                           const CallExpr *expr) {
1331:   assert(!cir::MissingFeatures::aarch64SMEIntrinsics());
1332: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenFunction::emitAArch64SMEBuiltinExpr`, `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenFunction::emitAArch64SMEBuiltinExpr`、`assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1333-1338
```cpp
1333:   cgm.errorNYI(expr->getSourceRange(),
1334:                std::string("unimplemented AArch64 builtin call: ") +
1335:                    getContext().BuiltinInfo.getName(builtinID));
1336:   return mlir::Value{};
1337: }
1338: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1339-1356
```cpp
1339: // Some intrinsics are equivalent for codegen.
1340: static const std::pair<unsigned, unsigned> neonEquivalentIntrinsicMap[] = {
1341:     {
1342:         NEON::BI__builtin_neon_splat_lane_bf16,
1343:         NEON::BI__builtin_neon_splat_lane_v,
1344:     },
1345:     {
1346:         NEON::BI__builtin_neon_splat_laneq_bf16,
1347:         NEON::BI__builtin_neon_splat_laneq_v,
1348:     },
1349:     {
1350:         NEON::BI__builtin_neon_splatq_lane_bf16,
1351:         NEON::BI__builtin_neon_splatq_lane_v,
1352:     },
1353:     {
1354:         NEON::BI__builtin_neon_splatq_laneq_bf16,
1355:         NEON::BI__builtin_neon_splatq_laneq_v,
1356:     },
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1357-1374
```cpp
1357:     {
1358:         NEON::BI__builtin_neon_vabd_f16,
1359:         NEON::BI__builtin_neon_vabd_v,
1360:     },
1361:     {
1362:         NEON::BI__builtin_neon_vabdq_f16,
1363:         NEON::BI__builtin_neon_vabdq_v,
1364:     },
1365:     {
1366:         NEON::BI__builtin_neon_vabs_f16,
1367:         NEON::BI__builtin_neon_vabs_v,
1368:     },
1369:     {
1370:         NEON::BI__builtin_neon_vabsq_f16,
1371:         NEON::BI__builtin_neon_vabsq_v,
1372:     },
1373:     {
1374:         NEON::BI__builtin_neon_vcage_f16,
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1375-1392
```cpp
1375:         NEON::BI__builtin_neon_vcage_v,
1376:     },
1377:     {
1378:         NEON::BI__builtin_neon_vcageq_f16,
1379:         NEON::BI__builtin_neon_vcageq_v,
1380:     },
1381:     {
1382:         NEON::BI__builtin_neon_vcagt_f16,
1383:         NEON::BI__builtin_neon_vcagt_v,
1384:     },
1385:     {
1386:         NEON::BI__builtin_neon_vcagtq_f16,
1387:         NEON::BI__builtin_neon_vcagtq_v,
1388:     },
1389:     {
1390:         NEON::BI__builtin_neon_vcale_f16,
1391:         NEON::BI__builtin_neon_vcale_v,
1392:     },
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1393-1410
```cpp
1393:     {
1394:         NEON::BI__builtin_neon_vcaleq_f16,
1395:         NEON::BI__builtin_neon_vcaleq_v,
1396:     },
1397:     {
1398:         NEON::BI__builtin_neon_vcalt_f16,
1399:         NEON::BI__builtin_neon_vcalt_v,
1400:     },
1401:     {
1402:         NEON::BI__builtin_neon_vcaltq_f16,
1403:         NEON::BI__builtin_neon_vcaltq_v,
1404:     },
1405:     {
1406:         NEON::BI__builtin_neon_vceqz_f16,
1407:         NEON::BI__builtin_neon_vceqz_v,
1408:     },
1409:     {
1410:         NEON::BI__builtin_neon_vceqzq_f16,
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1411-1428
```cpp
1411:         NEON::BI__builtin_neon_vceqzq_v,
1412:     },
1413:     {
1414:         NEON::BI__builtin_neon_vcgez_f16,
1415:         NEON::BI__builtin_neon_vcgez_v,
1416:     },
1417:     {
1418:         NEON::BI__builtin_neon_vcgezq_f16,
1419:         NEON::BI__builtin_neon_vcgezq_v,
1420:     },
1421:     {
1422:         NEON::BI__builtin_neon_vcgtz_f16,
1423:         NEON::BI__builtin_neon_vcgtz_v,
1424:     },
1425:     {
1426:         NEON::BI__builtin_neon_vcgtzq_f16,
1427:         NEON::BI__builtin_neon_vcgtzq_v,
1428:     },
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1429-1446
```cpp
1429:     {
1430:         NEON::BI__builtin_neon_vclez_f16,
1431:         NEON::BI__builtin_neon_vclez_v,
1432:     },
1433:     {
1434:         NEON::BI__builtin_neon_vclezq_f16,
1435:         NEON::BI__builtin_neon_vclezq_v,
1436:     },
1437:     {
1438:         NEON::BI__builtin_neon_vcltz_f16,
1439:         NEON::BI__builtin_neon_vcltz_v,
1440:     },
1441:     {
1442:         NEON::BI__builtin_neon_vcltzq_f16,
1443:         NEON::BI__builtin_neon_vcltzq_v,
1444:     },
1445:     {
1446:         NEON::BI__builtin_neon_vfma_f16,
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1447-1464
```cpp
1447:         NEON::BI__builtin_neon_vfma_v,
1448:     },
1449:     {
1450:         NEON::BI__builtin_neon_vfma_lane_f16,
1451:         NEON::BI__builtin_neon_vfma_lane_v,
1452:     },
1453:     {
1454:         NEON::BI__builtin_neon_vfma_laneq_f16,
1455:         NEON::BI__builtin_neon_vfma_laneq_v,
1456:     },
1457:     {
1458:         NEON::BI__builtin_neon_vfmaq_f16,
1459:         NEON::BI__builtin_neon_vfmaq_v,
1460:     },
1461:     {
1462:         NEON::BI__builtin_neon_vfmaq_lane_f16,
1463:         NEON::BI__builtin_neon_vfmaq_lane_v,
1464:     },
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1465-1482
```cpp
1465:     {
1466:         NEON::BI__builtin_neon_vfmaq_laneq_f16,
1467:         NEON::BI__builtin_neon_vfmaq_laneq_v,
1468:     },
1469:     {NEON::BI__builtin_neon_vld1_bf16_x2, NEON::BI__builtin_neon_vld1_x2_v},
1470:     {NEON::BI__builtin_neon_vld1_bf16_x3, NEON::BI__builtin_neon_vld1_x3_v},
1471:     {NEON::BI__builtin_neon_vld1_bf16_x4, NEON::BI__builtin_neon_vld1_x4_v},
1472:     {NEON::BI__builtin_neon_vld1_bf16, NEON::BI__builtin_neon_vld1_v},
1473:     {NEON::BI__builtin_neon_vld1_dup_bf16, NEON::BI__builtin_neon_vld1_dup_v},
1474:     {NEON::BI__builtin_neon_vld1_lane_bf16, NEON::BI__builtin_neon_vld1_lane_v},
1475:     {NEON::BI__builtin_neon_vld1q_bf16_x2, NEON::BI__builtin_neon_vld1q_x2_v},
1476:     {NEON::BI__builtin_neon_vld1q_bf16_x3, NEON::BI__builtin_neon_vld1q_x3_v},
1477:     {NEON::BI__builtin_neon_vld1q_bf16_x4, NEON::BI__builtin_neon_vld1q_x4_v},
1478:     {NEON::BI__builtin_neon_vld1q_bf16, NEON::BI__builtin_neon_vld1q_v},
1479:     {NEON::BI__builtin_neon_vld1q_dup_bf16, NEON::BI__builtin_neon_vld1q_dup_v},
1480:     {NEON::BI__builtin_neon_vld1q_lane_bf16,
1481:      NEON::BI__builtin_neon_vld1q_lane_v},
1482:     {NEON::BI__builtin_neon_vld2_bf16, NEON::BI__builtin_neon_vld2_v},
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1483-1500
```cpp
1483:     {NEON::BI__builtin_neon_vld2_dup_bf16, NEON::BI__builtin_neon_vld2_dup_v},
1484:     {NEON::BI__builtin_neon_vld2_lane_bf16, NEON::BI__builtin_neon_vld2_lane_v},
1485:     {NEON::BI__builtin_neon_vld2q_bf16, NEON::BI__builtin_neon_vld2q_v},
1486:     {NEON::BI__builtin_neon_vld2q_dup_bf16, NEON::BI__builtin_neon_vld2q_dup_v},
1487:     {NEON::BI__builtin_neon_vld2q_lane_bf16,
1488:      NEON::BI__builtin_neon_vld2q_lane_v},
1489:     {NEON::BI__builtin_neon_vld3_bf16, NEON::BI__builtin_neon_vld3_v},
1490:     {NEON::BI__builtin_neon_vld3_dup_bf16, NEON::BI__builtin_neon_vld3_dup_v},
1491:     {NEON::BI__builtin_neon_vld3_lane_bf16, NEON::BI__builtin_neon_vld3_lane_v},
1492:     {NEON::BI__builtin_neon_vld3q_bf16, NEON::BI__builtin_neon_vld3q_v},
1493:     {NEON::BI__builtin_neon_vld3q_dup_bf16, NEON::BI__builtin_neon_vld3q_dup_v},
1494:     {NEON::BI__builtin_neon_vld3q_lane_bf16,
1495:      NEON::BI__builtin_neon_vld3q_lane_v},
1496:     {NEON::BI__builtin_neon_vld4_bf16, NEON::BI__builtin_neon_vld4_v},
1497:     {NEON::BI__builtin_neon_vld4_dup_bf16, NEON::BI__builtin_neon_vld4_dup_v},
1498:     {NEON::BI__builtin_neon_vld4_lane_bf16, NEON::BI__builtin_neon_vld4_lane_v},
1499:     {NEON::BI__builtin_neon_vld4q_bf16, NEON::BI__builtin_neon_vld4q_v},
1500:     {NEON::BI__builtin_neon_vld4q_dup_bf16, NEON::BI__builtin_neon_vld4q_dup_v},
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1501-1518
```cpp
1501:     {NEON::BI__builtin_neon_vld4q_lane_bf16,
1502:      NEON::BI__builtin_neon_vld4q_lane_v},
1503:     {
1504:         NEON::BI__builtin_neon_vmax_f16,
1505:         NEON::BI__builtin_neon_vmax_v,
1506:     },
1507:     {
1508:         NEON::BI__builtin_neon_vmaxnm_f16,
1509:         NEON::BI__builtin_neon_vmaxnm_v,
1510:     },
1511:     {
1512:         NEON::BI__builtin_neon_vmaxnmq_f16,
1513:         NEON::BI__builtin_neon_vmaxnmq_v,
1514:     },
1515:     {
1516:         NEON::BI__builtin_neon_vmaxq_f16,
1517:         NEON::BI__builtin_neon_vmaxq_v,
1518:     },
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1519-1536
```cpp
1519:     {
1520:         NEON::BI__builtin_neon_vmin_f16,
1521:         NEON::BI__builtin_neon_vmin_v,
1522:     },
1523:     {
1524:         NEON::BI__builtin_neon_vminnm_f16,
1525:         NEON::BI__builtin_neon_vminnm_v,
1526:     },
1527:     {
1528:         NEON::BI__builtin_neon_vminnmq_f16,
1529:         NEON::BI__builtin_neon_vminnmq_v,
1530:     },
1531:     {
1532:         NEON::BI__builtin_neon_vminq_f16,
1533:         NEON::BI__builtin_neon_vminq_v,
1534:     },
1535:     {
1536:         NEON::BI__builtin_neon_vmulx_f16,
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1537-1554
```cpp
1537:         NEON::BI__builtin_neon_vmulx_v,
1538:     },
1539:     {
1540:         NEON::BI__builtin_neon_vmulxq_f16,
1541:         NEON::BI__builtin_neon_vmulxq_v,
1542:     },
1543:     {
1544:         NEON::BI__builtin_neon_vpadd_f16,
1545:         NEON::BI__builtin_neon_vpadd_v,
1546:     },
1547:     {
1548:         NEON::BI__builtin_neon_vpaddq_f16,
1549:         NEON::BI__builtin_neon_vpaddq_v,
1550:     },
1551:     {
1552:         NEON::BI__builtin_neon_vpmax_f16,
1553:         NEON::BI__builtin_neon_vpmax_v,
1554:     },
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1555-1572
```cpp
1555:     {
1556:         NEON::BI__builtin_neon_vpmaxnm_f16,
1557:         NEON::BI__builtin_neon_vpmaxnm_v,
1558:     },
1559:     {
1560:         NEON::BI__builtin_neon_vpmaxnmq_f16,
1561:         NEON::BI__builtin_neon_vpmaxnmq_v,
1562:     },
1563:     {
1564:         NEON::BI__builtin_neon_vpmaxq_f16,
1565:         NEON::BI__builtin_neon_vpmaxq_v,
1566:     },
1567:     {
1568:         NEON::BI__builtin_neon_vpmin_f16,
1569:         NEON::BI__builtin_neon_vpmin_v,
1570:     },
1571:     {
1572:         NEON::BI__builtin_neon_vpminnm_f16,
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1573-1590
```cpp
1573:         NEON::BI__builtin_neon_vpminnm_v,
1574:     },
1575:     {
1576:         NEON::BI__builtin_neon_vpminnmq_f16,
1577:         NEON::BI__builtin_neon_vpminnmq_v,
1578:     },
1579:     {
1580:         NEON::BI__builtin_neon_vpminq_f16,
1581:         NEON::BI__builtin_neon_vpminq_v,
1582:     },
1583:     {
1584:         NEON::BI__builtin_neon_vrecpe_f16,
1585:         NEON::BI__builtin_neon_vrecpe_v,
1586:     },
1587:     {
1588:         NEON::BI__builtin_neon_vrecpeq_f16,
1589:         NEON::BI__builtin_neon_vrecpeq_v,
1590:     },
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1591-1608
```cpp
1591:     {
1592:         NEON::BI__builtin_neon_vrecps_f16,
1593:         NEON::BI__builtin_neon_vrecps_v,
1594:     },
1595:     {
1596:         NEON::BI__builtin_neon_vrecpsq_f16,
1597:         NEON::BI__builtin_neon_vrecpsq_v,
1598:     },
1599:     {
1600:         NEON::BI__builtin_neon_vrnd_f16,
1601:         NEON::BI__builtin_neon_vrnd_v,
1602:     },
1603:     {
1604:         NEON::BI__builtin_neon_vrnda_f16,
1605:         NEON::BI__builtin_neon_vrnda_v,
1606:     },
1607:     {
1608:         NEON::BI__builtin_neon_vrndaq_f16,
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1609-1626
```cpp
1609:         NEON::BI__builtin_neon_vrndaq_v,
1610:     },
1611:     {
1612:         NEON::BI__builtin_neon_vrndi_f16,
1613:         NEON::BI__builtin_neon_vrndi_v,
1614:     },
1615:     {
1616:         NEON::BI__builtin_neon_vrndiq_f16,
1617:         NEON::BI__builtin_neon_vrndiq_v,
1618:     },
1619:     {
1620:         NEON::BI__builtin_neon_vrndm_f16,
1621:         NEON::BI__builtin_neon_vrndm_v,
1622:     },
1623:     {
1624:         NEON::BI__builtin_neon_vrndmq_f16,
1625:         NEON::BI__builtin_neon_vrndmq_v,
1626:     },
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1627-1644
```cpp
1627:     {
1628:         NEON::BI__builtin_neon_vrndn_f16,
1629:         NEON::BI__builtin_neon_vrndn_v,
1630:     },
1631:     {
1632:         NEON::BI__builtin_neon_vrndnq_f16,
1633:         NEON::BI__builtin_neon_vrndnq_v,
1634:     },
1635:     {
1636:         NEON::BI__builtin_neon_vrndp_f16,
1637:         NEON::BI__builtin_neon_vrndp_v,
1638:     },
1639:     {
1640:         NEON::BI__builtin_neon_vrndpq_f16,
1641:         NEON::BI__builtin_neon_vrndpq_v,
1642:     },
1643:     {
1644:         NEON::BI__builtin_neon_vrndq_f16,
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1645-1662
```cpp
1645:         NEON::BI__builtin_neon_vrndq_v,
1646:     },
1647:     {
1648:         NEON::BI__builtin_neon_vrndx_f16,
1649:         NEON::BI__builtin_neon_vrndx_v,
1650:     },
1651:     {
1652:         NEON::BI__builtin_neon_vrndxq_f16,
1653:         NEON::BI__builtin_neon_vrndxq_v,
1654:     },
1655:     {
1656:         NEON::BI__builtin_neon_vrsqrte_f16,
1657:         NEON::BI__builtin_neon_vrsqrte_v,
1658:     },
1659:     {
1660:         NEON::BI__builtin_neon_vrsqrteq_f16,
1661:         NEON::BI__builtin_neon_vrsqrteq_v,
1662:     },
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1663-1680
```cpp
1663:     {
1664:         NEON::BI__builtin_neon_vrsqrts_f16,
1665:         NEON::BI__builtin_neon_vrsqrts_v,
1666:     },
1667:     {
1668:         NEON::BI__builtin_neon_vrsqrtsq_f16,
1669:         NEON::BI__builtin_neon_vrsqrtsq_v,
1670:     },
1671:     {
1672:         NEON::BI__builtin_neon_vsqrt_f16,
1673:         NEON::BI__builtin_neon_vsqrt_v,
1674:     },
1675:     {
1676:         NEON::BI__builtin_neon_vsqrtq_f16,
1677:         NEON::BI__builtin_neon_vsqrtq_v,
1678:     },
1679:     {NEON::BI__builtin_neon_vst1_bf16_x2, NEON::BI__builtin_neon_vst1_x2_v},
1680:     {NEON::BI__builtin_neon_vst1_bf16_x3, NEON::BI__builtin_neon_vst1_x3_v},
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1681-1698
```cpp
1681:     {NEON::BI__builtin_neon_vst1_bf16_x4, NEON::BI__builtin_neon_vst1_x4_v},
1682:     {NEON::BI__builtin_neon_vst1_bf16, NEON::BI__builtin_neon_vst1_v},
1683:     {NEON::BI__builtin_neon_vst1_lane_bf16, NEON::BI__builtin_neon_vst1_lane_v},
1684:     {NEON::BI__builtin_neon_vst1q_bf16_x2, NEON::BI__builtin_neon_vst1q_x2_v},
1685:     {NEON::BI__builtin_neon_vst1q_bf16_x3, NEON::BI__builtin_neon_vst1q_x3_v},
1686:     {NEON::BI__builtin_neon_vst1q_bf16_x4, NEON::BI__builtin_neon_vst1q_x4_v},
1687:     {NEON::BI__builtin_neon_vst1q_bf16, NEON::BI__builtin_neon_vst1q_v},
1688:     {NEON::BI__builtin_neon_vst1q_lane_bf16,
1689:      NEON::BI__builtin_neon_vst1q_lane_v},
1690:     {NEON::BI__builtin_neon_vst2_bf16, NEON::BI__builtin_neon_vst2_v},
1691:     {NEON::BI__builtin_neon_vst2_lane_bf16, NEON::BI__builtin_neon_vst2_lane_v},
1692:     {NEON::BI__builtin_neon_vst2q_bf16, NEON::BI__builtin_neon_vst2q_v},
1693:     {NEON::BI__builtin_neon_vst2q_lane_bf16,
1694:      NEON::BI__builtin_neon_vst2q_lane_v},
1695:     {NEON::BI__builtin_neon_vst3_bf16, NEON::BI__builtin_neon_vst3_v},
1696:     {NEON::BI__builtin_neon_vst3_lane_bf16, NEON::BI__builtin_neon_vst3_lane_v},
1697:     {NEON::BI__builtin_neon_vst3q_bf16, NEON::BI__builtin_neon_vst3q_v},
1698:     {NEON::BI__builtin_neon_vst3q_lane_bf16,
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1699-1716
```cpp
1699:      NEON::BI__builtin_neon_vst3q_lane_v},
1700:     {NEON::BI__builtin_neon_vst4_bf16, NEON::BI__builtin_neon_vst4_v},
1701:     {NEON::BI__builtin_neon_vst4_lane_bf16, NEON::BI__builtin_neon_vst4_lane_v},
1702:     {NEON::BI__builtin_neon_vst4q_bf16, NEON::BI__builtin_neon_vst4q_v},
1703:     {NEON::BI__builtin_neon_vst4q_lane_bf16,
1704:      NEON::BI__builtin_neon_vst4q_lane_v},
1705:     // The mangling rules cause us to have one ID for each type for
1706:     // vldap1(q)_lane and vstl1(q)_lane, but codegen is equivalent for all of
1707:     // them. Choose an arbitrary one to be handled as tha canonical variation.
1708:     {NEON::BI__builtin_neon_vldap1_lane_u64,
1709:      NEON::BI__builtin_neon_vldap1_lane_s64},
1710:     {NEON::BI__builtin_neon_vldap1_lane_f64,
1711:      NEON::BI__builtin_neon_vldap1_lane_s64},
1712:     {NEON::BI__builtin_neon_vldap1_lane_p64,
1713:      NEON::BI__builtin_neon_vldap1_lane_s64},
1714:     {NEON::BI__builtin_neon_vldap1q_lane_u64,
1715:      NEON::BI__builtin_neon_vldap1q_lane_s64},
1716:     {NEON::BI__builtin_neon_vldap1q_lane_f64,
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1717-1733
```cpp
1717:      NEON::BI__builtin_neon_vldap1q_lane_s64},
1718:     {NEON::BI__builtin_neon_vldap1q_lane_p64,
1719:      NEON::BI__builtin_neon_vldap1q_lane_s64},
1720:     {NEON::BI__builtin_neon_vstl1_lane_u64,
1721:      NEON::BI__builtin_neon_vstl1_lane_s64},
1722:     {NEON::BI__builtin_neon_vstl1_lane_f64,
1723:      NEON::BI__builtin_neon_vstl1_lane_s64},
1724:     {NEON::BI__builtin_neon_vstl1_lane_p64,
1725:      NEON::BI__builtin_neon_vstl1_lane_s64},
1726:     {NEON::BI__builtin_neon_vstl1q_lane_u64,
1727:      NEON::BI__builtin_neon_vstl1q_lane_s64},
1728:     {NEON::BI__builtin_neon_vstl1q_lane_f64,
1729:      NEON::BI__builtin_neon_vstl1q_lane_s64},
1730:     {NEON::BI__builtin_neon_vstl1q_lane_p64,
1731:      NEON::BI__builtin_neon_vstl1q_lane_s64},
1732: };
1733: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1734-1741
```cpp
1734: std::optional<mlir::Value>
1735: CIRGenFunction::emitAArch64BuiltinExpr(unsigned builtinID, const CallExpr *expr,
1736:                                        ReturnValueSlot returnValue,
1737:                                        llvm::Triple::ArchType arch) {
1738:   if (builtinID >= clang::AArch64::FirstSVEBuiltin &&
1739:       builtinID <= clang::AArch64::LastSVEBuiltin)
1740:     return emitAArch64SVEBuiltinExpr(builtinID, expr);
1741: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenFunction::emitAArch64BuiltinExpr`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenFunction::emitAArch64BuiltinExpr`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1742-1745
```cpp
1742:   if (builtinID >= clang::AArch64::FirstSMEBuiltin &&
1743:       builtinID <= clang::AArch64::LastSMEBuiltin)
1744:     return emitAArch64SMEBuiltinExpr(builtinID, expr);
1745: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 1746-1752
```cpp
1746:   if (builtinID == Builtin::BI__builtin_cpu_supports) {
1747:     cgm.errorNYI(expr->getSourceRange(),
1748:                  std::string("unimplemented AArch64 builtin call: ") +
1749:                      getContext().BuiltinInfo.getName(builtinID));
1750:     return mlir::Value{};
1751:   }
1752: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1753-1772
```cpp
1753:   switch (builtinID) {
1754:   default:
1755:     break;
1756:   case clang::AArch64::BI__builtin_arm_nop:
1757:   case clang::AArch64::BI__builtin_arm_yield:
1758:   case clang::AArch64::BI__yield:
1759:   case clang::AArch64::BI__builtin_arm_wfe:
1760:   case clang::AArch64::BI__wfe:
1761:   case clang::AArch64::BI__builtin_arm_wfi:
1762:   case clang::AArch64::BI__wfi:
1763:   case clang::AArch64::BI__builtin_arm_sev:
1764:   case clang::AArch64::BI__sev:
1765:   case clang::AArch64::BI__builtin_arm_sevl:
1766:   case clang::AArch64::BI__sevl:
1767:     cgm.errorNYI(expr->getSourceRange(),
1768:                  std::string("unimplemented AArch64 builtin call: ") +
1769:                      getContext().BuiltinInfo.getName(builtinID));
1770:     return mlir::Value{};
1771:   }
1772: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. A switch statement is used to dispatch behavior across enumerated cases or kinds. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1773-1779
```cpp
1773:   if (builtinID == clang::AArch64::BI__builtin_arm_trap) {
1774:     cgm.errorNYI(expr->getSourceRange(),
1775:                  std::string("unimplemented AArch64 builtin call: ") +
1776:                      getContext().BuiltinInfo.getName(builtinID));
1777:     return mlir::Value{};
1778:   }
1779: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1780-1786
```cpp
1780:   if (builtinID == clang::AArch64::BI__builtin_arm_get_sme_state) {
1781:     cgm.errorNYI(expr->getSourceRange(),
1782:                  std::string("unimplemented AArch64 builtin call: ") +
1783:                      getContext().BuiltinInfo.getName(builtinID));
1784:     return mlir::Value{};
1785:   }
1786: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1787-1799
```cpp
1787:   if (builtinID == clang::AArch64::BI__builtin_arm_rbit) {
1788:     cgm.errorNYI(expr->getSourceRange(),
1789:                  std::string("unimplemented AArch64 builtin call: ") +
1790:                      getContext().BuiltinInfo.getName(builtinID));
1791:     return mlir::Value{};
1792:   }
1793:   if (builtinID == clang::AArch64::BI__builtin_arm_rbit64) {
1794:     cgm.errorNYI(expr->getSourceRange(),
1795:                  std::string("unimplemented AArch64 builtin call: ") +
1796:                      getContext().BuiltinInfo.getName(builtinID));
1797:     return mlir::Value{};
1798:   }
1799: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1800-1807
```cpp
1800:   if (builtinID == clang::AArch64::BI__builtin_arm_clz ||
1801:       builtinID == clang::AArch64::BI__builtin_arm_clz64) {
1802:     cgm.errorNYI(expr->getSourceRange(),
1803:                  std::string("unimplemented AArch64 builtin call: ") +
1804:                      getContext().BuiltinInfo.getName(builtinID));
1805:     return mlir::Value{};
1806:   }
1807: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1808-1820
```cpp
1808:   if (builtinID == clang::AArch64::BI__builtin_arm_cls) {
1809:     cgm.errorNYI(expr->getSourceRange(),
1810:                  std::string("unimplemented AArch64 builtin call: ") +
1811:                      getContext().BuiltinInfo.getName(builtinID));
1812:     return mlir::Value{};
1813:   }
1814:   if (builtinID == clang::AArch64::BI__builtin_arm_cls64) {
1815:     cgm.errorNYI(expr->getSourceRange(),
1816:                  std::string("unimplemented AArch64 builtin call: ") +
1817:                      getContext().BuiltinInfo.getName(builtinID));
1818:     return mlir::Value{};
1819:   }
1820: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1821-1828
```cpp
1821:   if (builtinID == clang::AArch64::BI__builtin_arm_rint32zf ||
1822:       builtinID == clang::AArch64::BI__builtin_arm_rint32z) {
1823:     cgm.errorNYI(expr->getSourceRange(),
1824:                  std::string("unimplemented AArch64 builtin call: ") +
1825:                      getContext().BuiltinInfo.getName(builtinID));
1826:     return mlir::Value{};
1827:   }
1828: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1829-1836
```cpp
1829:   if (builtinID == clang::AArch64::BI__builtin_arm_rint64zf ||
1830:       builtinID == clang::AArch64::BI__builtin_arm_rint64z) {
1831:     cgm.errorNYI(expr->getSourceRange(),
1832:                  std::string("unimplemented AArch64 builtin call: ") +
1833:                      getContext().BuiltinInfo.getName(builtinID));
1834:     return mlir::Value{};
1835:   }
1836: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1837-1844
```cpp
1837:   if (builtinID == clang::AArch64::BI__builtin_arm_rint32xf ||
1838:       builtinID == clang::AArch64::BI__builtin_arm_rint32x) {
1839:     cgm.errorNYI(expr->getSourceRange(),
1840:                  std::string("unimplemented AArch64 builtin call: ") +
1841:                      getContext().BuiltinInfo.getName(builtinID));
1842:     return mlir::Value{};
1843:   }
1844: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1845-1852
```cpp
1845:   if (builtinID == clang::AArch64::BI__builtin_arm_rint64xf ||
1846:       builtinID == clang::AArch64::BI__builtin_arm_rint64x) {
1847:     cgm.errorNYI(expr->getSourceRange(),
1848:                  std::string("unimplemented AArch64 builtin call: ") +
1849:                      getContext().BuiltinInfo.getName(builtinID));
1850:     return mlir::Value{};
1851:   }
1852: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1853-1859
```cpp
1853:   if (builtinID == clang::AArch64::BI__builtin_arm_jcvt) {
1854:     cgm.errorNYI(expr->getSourceRange(),
1855:                  std::string("unimplemented AArch64 builtin call: ") +
1856:                      getContext().BuiltinInfo.getName(builtinID));
1857:     return mlir::Value{};
1858:   }
1859: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1860-1869
```cpp
1860:   if (builtinID == clang::AArch64::BI__builtin_arm_ld64b ||
1861:       builtinID == clang::AArch64::BI__builtin_arm_st64b ||
1862:       builtinID == clang::AArch64::BI__builtin_arm_st64bv ||
1863:       builtinID == clang::AArch64::BI__builtin_arm_st64bv0) {
1864:     cgm.errorNYI(expr->getSourceRange(),
1865:                  std::string("unimplemented AArch64 builtin call: ") +
1866:                      getContext().BuiltinInfo.getName(builtinID));
1867:     return mlir::Value{};
1868:   }
1869: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1870-1877
```cpp
1870:   if (builtinID == clang::AArch64::BI__builtin_arm_rndr ||
1871:       builtinID == clang::AArch64::BI__builtin_arm_rndrrs) {
1872:     cgm.errorNYI(expr->getSourceRange(),
1873:                  std::string("unimplemented AArch64 builtin call: ") +
1874:                      getContext().BuiltinInfo.getName(builtinID));
1875:     return mlir::Value{};
1876:   }
1877: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1878-1884
```cpp
1878:   if (builtinID == clang::AArch64::BI__clear_cache) {
1879:     cgm.errorNYI(expr->getSourceRange(),
1880:                  std::string("unimplemented AArch64 builtin call: ") +
1881:                      getContext().BuiltinInfo.getName(builtinID));
1882:     return mlir::Value{};
1883:   }
1884: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1885-1900
```cpp
1885:   if ((builtinID == clang::AArch64::BI__builtin_arm_ldrex ||
1886:        builtinID == clang::AArch64::BI__builtin_arm_ldaex) &&
1887:       getContext().getTypeSize(expr->getType()) == 128) {
1888:     cgm.errorNYI(expr->getSourceRange(),
1889:                  std::string("unimplemented AArch64 builtin call: ") +
1890:                      getContext().BuiltinInfo.getName(builtinID));
1891:     return mlir::Value{};
1892:   }
1893:   if (builtinID == clang::AArch64::BI__builtin_arm_ldrex ||
1894:       builtinID == clang::AArch64::BI__builtin_arm_ldaex) {
1895:     cgm.errorNYI(expr->getSourceRange(),
1896:                  std::string("unimplemented AArch64 builtin call: ") +
1897:                      getContext().BuiltinInfo.getName(builtinID));
1898:     return mlir::Value{};
1899:   }
1900: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1901-1909
```cpp
1901:   if ((builtinID == clang::AArch64::BI__builtin_arm_strex ||
1902:        builtinID == clang::AArch64::BI__builtin_arm_stlex) &&
1903:       getContext().getTypeSize(expr->getArg(0)->getType()) == 128) {
1904:     cgm.errorNYI(expr->getSourceRange(),
1905:                  std::string("unimplemented AArch64 builtin call: ") +
1906:                      getContext().BuiltinInfo.getName(builtinID));
1907:     return mlir::Value{};
1908:   }
1909: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1910-1917
```cpp
1910:   if (builtinID == clang::AArch64::BI__builtin_arm_strex ||
1911:       builtinID == clang::AArch64::BI__builtin_arm_stlex) {
1912:     cgm.errorNYI(expr->getSourceRange(),
1913:                  std::string("unimplemented AArch64 builtin call: ") +
1914:                      getContext().BuiltinInfo.getName(builtinID));
1915:     return mlir::Value{};
1916:   }
1917: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1918-1924
```cpp
1918:   if (builtinID == clang::AArch64::BI__getReg) {
1919:     cgm.errorNYI(expr->getSourceRange(),
1920:                  std::string("unimplemented AArch64 builtin call: ") +
1921:                      getContext().BuiltinInfo.getName(builtinID));
1922:     return mlir::Value{};
1923:   }
1924: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1925-1931
```cpp
1925:   if (builtinID == clang::AArch64::BI__break) {
1926:     cgm.errorNYI(expr->getSourceRange(),
1927:                  std::string("unimplemented AArch64 builtin call: ") +
1928:                      getContext().BuiltinInfo.getName(builtinID));
1929:     return mlir::Value{};
1930:   }
1931: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1932-1938
```cpp
1932:   if (builtinID == clang::AArch64::BI__builtin_arm_clrex) {
1933:     cgm.errorNYI(expr->getSourceRange(),
1934:                  std::string("unimplemented AArch64 builtin call: ") +
1935:                      getContext().BuiltinInfo.getName(builtinID));
1936:     return mlir::Value{};
1937:   }
1938: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1939-1945
```cpp
1939:   if (builtinID == clang::AArch64::BI_ReadWriteBarrier) {
1940:     cgm.errorNYI(expr->getSourceRange(),
1941:                  std::string("unimplemented AArch64 builtin call: ") +
1942:                      getContext().BuiltinInfo.getName(builtinID));
1943:     return mlir::Value{};
1944:   }
1945: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1946-1963
```cpp
1946:   // CRC32
1947:   Intrinsic::ID crcIntrinsicID = Intrinsic::not_intrinsic;
1948:   switch (builtinID) {
1949:   case clang::AArch64::BI__builtin_arm_crc32b:
1950:     crcIntrinsicID = Intrinsic::aarch64_crc32b;
1951:     break;
1952:   case clang::AArch64::BI__builtin_arm_crc32cb:
1953:     crcIntrinsicID = Intrinsic::aarch64_crc32cb;
1954:     break;
1955:   case clang::AArch64::BI__builtin_arm_crc32h:
1956:     crcIntrinsicID = Intrinsic::aarch64_crc32h;
1957:     break;
1958:   case clang::AArch64::BI__builtin_arm_crc32ch:
1959:     crcIntrinsicID = Intrinsic::aarch64_crc32ch;
1960:     break;
1961:   case clang::AArch64::BI__builtin_arm_crc32w:
1962:     crcIntrinsicID = Intrinsic::aarch64_crc32w;
1963:     break;
```
- **EN**: This block contains local control-flow decisions or iterative processing. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 1964-1974
```cpp
1964:   case clang::AArch64::BI__builtin_arm_crc32cw:
1965:     crcIntrinsicID = Intrinsic::aarch64_crc32cw;
1966:     break;
1967:   case clang::AArch64::BI__builtin_arm_crc32d:
1968:     crcIntrinsicID = Intrinsic::aarch64_crc32x;
1969:     break;
1970:   case clang::AArch64::BI__builtin_arm_crc32cd:
1971:     crcIntrinsicID = Intrinsic::aarch64_crc32cx;
1972:     break;
1973:   }
1974: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 1975-1981
```cpp
1975:   if (crcIntrinsicID != Intrinsic::not_intrinsic) {
1976:     cgm.errorNYI(expr->getSourceRange(),
1977:                  std::string("unimplemented AArch64 builtin call: ") +
1978:                      getContext().BuiltinInfo.getName(builtinID));
1979:     return mlir::Value{};
1980:   }
1981: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1982-1989
```cpp
1982:   // Memory Operations (MOPS)
1983:   if (builtinID == AArch64::BI__builtin_arm_mops_memset_tag) {
1984:     cgm.errorNYI(expr->getSourceRange(),
1985:                  std::string("unimplemented AArch64 builtin call: ") +
1986:                      getContext().BuiltinInfo.getName(builtinID));
1987:     return mlir::Value{};
1988:   }
1989: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 1990-2007
```cpp
1990:   // Memory Tagging Extensions (MTE) Intrinsics
1991:   Intrinsic::ID mteIntrinsicID = Intrinsic::not_intrinsic;
1992:   switch (builtinID) {
1993:   case clang::AArch64::BI__builtin_arm_irg:
1994:     mteIntrinsicID = Intrinsic::aarch64_irg;
1995:     break;
1996:   case clang::AArch64::BI__builtin_arm_addg:
1997:     mteIntrinsicID = Intrinsic::aarch64_addg;
1998:     break;
1999:   case clang::AArch64::BI__builtin_arm_gmi:
2000:     mteIntrinsicID = Intrinsic::aarch64_gmi;
2001:     break;
2002:   case clang::AArch64::BI__builtin_arm_ldg:
2003:     mteIntrinsicID = Intrinsic::aarch64_ldg;
2004:     break;
2005:   case clang::AArch64::BI__builtin_arm_stg:
2006:     mteIntrinsicID = Intrinsic::aarch64_stg;
2007:     break;
```
- **EN**: This block contains local control-flow decisions or iterative processing. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 2008-2012
```cpp
2008:   case clang::AArch64::BI__builtin_arm_subp:
2009:     mteIntrinsicID = Intrinsic::aarch64_subp;
2010:     break;
2011:   }
2012: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2013-2019
```cpp
2013:   if (mteIntrinsicID != Intrinsic::not_intrinsic) {
2014:     cgm.errorNYI(expr->getSourceRange(),
2015:                  std::string("unimplemented AArch64 builtin call: ") +
2016:                      getContext().BuiltinInfo.getName(builtinID));
2017:     return mlir::Value{};
2018:   }
2019: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2020-2033
```cpp
2020:   if (builtinID == clang::AArch64::BI__builtin_arm_rsr ||
2021:       builtinID == clang::AArch64::BI__builtin_arm_rsr64 ||
2022:       builtinID == clang::AArch64::BI__builtin_arm_rsr128 ||
2023:       builtinID == clang::AArch64::BI__builtin_arm_rsrp ||
2024:       builtinID == clang::AArch64::BI__builtin_arm_wsr ||
2025:       builtinID == clang::AArch64::BI__builtin_arm_wsr64 ||
2026:       builtinID == clang::AArch64::BI__builtin_arm_wsr128 ||
2027:       builtinID == clang::AArch64::BI__builtin_arm_wsrp) {
2028:     cgm.errorNYI(expr->getSourceRange(),
2029:                  std::string("unimplemented AArch64 builtin call: ") +
2030:                      getContext().BuiltinInfo.getName(builtinID));
2031:     return mlir::Value{};
2032:   }
2033: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2034-2042
```cpp
2034:   if (builtinID == clang::AArch64::BI_ReadStatusReg ||
2035:       builtinID == clang::AArch64::BI_WriteStatusReg ||
2036:       builtinID == clang::AArch64::BI__sys) {
2037:     cgm.errorNYI(expr->getSourceRange(),
2038:                  std::string("unimplemented AArch64 builtin call: ") +
2039:                      getContext().BuiltinInfo.getName(builtinID));
2040:     return mlir::Value{};
2041:   }
2042: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2043-2049
```cpp
2043:   if (builtinID == clang::AArch64::BI_AddressOfReturnAddress) {
2044:     cgm.errorNYI(expr->getSourceRange(),
2045:                  std::string("unimplemented AArch64 builtin call: ") +
2046:                      getContext().BuiltinInfo.getName(builtinID));
2047:     return mlir::Value{};
2048:   }
2049: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2050-2056
```cpp
2050:   if (builtinID == clang::AArch64::BI__builtin_sponentry) {
2051:     cgm.errorNYI(expr->getSourceRange(),
2052:                  std::string("unimplemented AArch64 builtin call: ") +
2053:                      getContext().BuiltinInfo.getName(builtinID));
2054:     return mlir::Value{};
2055:   }
2056: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2057-2064
```cpp
2057:   if (builtinID == clang::AArch64::BI__mulh ||
2058:       builtinID == clang::AArch64::BI__umulh) {
2059:     cgm.errorNYI(expr->getSourceRange(),
2060:                  std::string("unimplemented AArch64 builtin call: ") +
2061:                      getContext().BuiltinInfo.getName(builtinID));
2062:     return mlir::Value{};
2063:   }
2064: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2065-2074
```cpp
2065:   if (builtinID == AArch64::BI__writex18byte ||
2066:       builtinID == AArch64::BI__writex18word ||
2067:       builtinID == AArch64::BI__writex18dword ||
2068:       builtinID == AArch64::BI__writex18qword) {
2069:     cgm.errorNYI(expr->getSourceRange(),
2070:                  std::string("unimplemented AArch64 builtin call: ") +
2071:                      getContext().BuiltinInfo.getName(builtinID));
2072:     return mlir::Value{};
2073:   }
2074: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2075-2084
```cpp
2075:   if (builtinID == AArch64::BI__readx18byte ||
2076:       builtinID == AArch64::BI__readx18word ||
2077:       builtinID == AArch64::BI__readx18dword ||
2078:       builtinID == AArch64::BI__readx18qword) {
2079:     cgm.errorNYI(expr->getSourceRange(),
2080:                  std::string("unimplemented AArch64 builtin call: ") +
2081:                      getContext().BuiltinInfo.getName(builtinID));
2082:     return mlir::Value{};
2083:   }
2084: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2085-2098
```cpp
2085:   if (builtinID == AArch64::BI__addx18byte ||
2086:       builtinID == AArch64::BI__addx18word ||
2087:       builtinID == AArch64::BI__addx18dword ||
2088:       builtinID == AArch64::BI__addx18qword ||
2089:       builtinID == AArch64::BI__incx18byte ||
2090:       builtinID == AArch64::BI__incx18word ||
2091:       builtinID == AArch64::BI__incx18dword ||
2092:       builtinID == AArch64::BI__incx18qword) {
2093:     cgm.errorNYI(expr->getSourceRange(),
2094:                  std::string("unimplemented AArch64 builtin call: ") +
2095:                      getContext().BuiltinInfo.getName(builtinID));
2096:     return mlir::Value{};
2097:   }
2098: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2099-2108
```cpp
2099:   if (builtinID == AArch64::BI_CopyDoubleFromInt64 ||
2100:       builtinID == AArch64::BI_CopyFloatFromInt32 ||
2101:       builtinID == AArch64::BI_CopyInt32FromFloat ||
2102:       builtinID == AArch64::BI_CopyInt64FromDouble) {
2103:     cgm.errorNYI(expr->getSourceRange(),
2104:                  std::string("unimplemented AArch64 builtin call: ") +
2105:                      getContext().BuiltinInfo.getName(builtinID));
2106:     return mlir::Value{};
2107:   }
2108: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2109-2118
```cpp
2109:   if (builtinID == AArch64::BI_CountLeadingOnes ||
2110:       builtinID == AArch64::BI_CountLeadingOnes64 ||
2111:       builtinID == AArch64::BI_CountLeadingZeros ||
2112:       builtinID == AArch64::BI_CountLeadingZeros64) {
2113:     cgm.errorNYI(expr->getSourceRange(),
2114:                  std::string("unimplemented AArch64 builtin call: ") +
2115:                      getContext().BuiltinInfo.getName(builtinID));
2116:     return mlir::Value{};
2117:   }
2118: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2119-2126
```cpp
2119:   if (builtinID == AArch64::BI_CountLeadingSigns ||
2120:       builtinID == AArch64::BI_CountLeadingSigns64) {
2121:     cgm.errorNYI(expr->getSourceRange(),
2122:                  std::string("unimplemented AArch64 builtin call: ") +
2123:                      getContext().BuiltinInfo.getName(builtinID));
2124:     return mlir::Value{};
2125:   }
2126: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2127-2134
```cpp
2127:   if (builtinID == AArch64::BI_CountOneBits ||
2128:       builtinID == AArch64::BI_CountOneBits64) {
2129:     cgm.errorNYI(expr->getSourceRange(),
2130:                  std::string("unimplemented AArch64 builtin call: ") +
2131:                      getContext().BuiltinInfo.getName(builtinID));
2132:     return mlir::Value{};
2133:   }
2134: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2135-2141
```cpp
2135:   if (builtinID == AArch64::BI__prefetch) {
2136:     cgm.errorNYI(expr->getSourceRange(),
2137:                  std::string("unimplemented AArch64 builtin call: ") +
2138:                      getContext().BuiltinInfo.getName(builtinID));
2139:     return mlir::Value{};
2140:   }
2141: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2142-2148
```cpp
2142:   if (builtinID == AArch64::BI__hlt) {
2143:     cgm.errorNYI(expr->getSourceRange(),
2144:                  std::string("unimplemented AArch64 builtin call: ") +
2145:                      getContext().BuiltinInfo.getName(builtinID));
2146:     return mlir::Value{};
2147:   }
2148: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2149-2155
```cpp
2149:   if (builtinID == NEON::BI__builtin_neon_vcvth_bf16_f32) {
2150:     cgm.errorNYI(expr->getSourceRange(),
2151:                  std::string("unimplemented AArch64 builtin call: ") +
2152:                      getContext().BuiltinInfo.getName(builtinID));
2153:     return mlir::Value{};
2154:   }
2155: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2156-2159
```cpp
2156:   // Handle MSVC intrinsics before argument evaluation to prevent double
2157:   // evaluation.
2158:   assert(!cir::MissingFeatures::msvcBuiltins());
2159: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2160-2166
```cpp
2160:   // Some intrinsics are equivalent - if they are use the base intrinsic ID.
2161:   auto it = llvm::find_if(neonEquivalentIntrinsicMap, [builtinID](auto &p) {
2162:     return p.first == builtinID;
2163:   });
2164:   if (it != end(neonEquivalentIntrinsicMap))
2165:     builtinID = it->second;
2166: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2167-2175
```cpp
2167:   // Find out if any arguments are required to be integer constant
2168:   // expressions.
2169:   assert(!cir::MissingFeatures::handleBuiltinICEArguments());
2170:   unsigned iceArguments = 0;
2171:   ASTContext::GetBuiltinTypeError error;
2172:   getContext().GetBuiltinType(builtinID, error, &iceArguments);
2173:   assert(error == ASTContext::GE_None && "Should not codegen an error");
2174:   llvm::SmallVector<mlir::Value> ops;
2175: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`, `getContext`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`、`getContext`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2176-2193
```cpp
2176:   // Skip extra arguments used to discriminate vector types and that are
2177:   // intended for Sema checking.
2178:   bool hasExtraArg = hasExtraNeonArgument(builtinID);
2179:   unsigned numArgs = expr->getNumArgs() - (hasExtraArg ? 1 : 0);
2180:   for (unsigned i = 0, e = numArgs; i != e; i++) {
2181:     if (i == 0) {
2182:       switch (builtinID) {
2183:       case NEON::BI__builtin_neon_vld1_v:
2184:       case NEON::BI__builtin_neon_vld1q_v:
2185:       case NEON::BI__builtin_neon_vld1_dup_v:
2186:       case NEON::BI__builtin_neon_vld1q_dup_v:
2187:       case NEON::BI__builtin_neon_vld1_lane_v:
2188:       case NEON::BI__builtin_neon_vld1q_lane_v:
2189:       case NEON::BI__builtin_neon_vst1_v:
2190:       case NEON::BI__builtin_neon_vst1q_v:
2191:       case NEON::BI__builtin_neon_vst1_lane_v:
2192:       case NEON::BI__builtin_neon_vst1q_lane_v:
2193:       case NEON::BI__builtin_neon_vldap1_lane_s64:
```
- **EN**: This block contains local control-flow decisions or iterative processing. A switch statement is used to dispatch behavior across enumerated cases or kinds. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 2194-2208
```cpp
2194:       case NEON::BI__builtin_neon_vldap1q_lane_s64:
2195:       case NEON::BI__builtin_neon_vstl1_lane_s64:
2196:       case NEON::BI__builtin_neon_vstl1q_lane_s64:
2197:         // Get the alignment for the argument in addition to the value;
2198:         // we'll use it later.
2199:         cgm.errorNYI(
2200:             expr->getSourceRange(),
2201:             std::string("unimplemented AArch64 builtin argument handling ") +
2202:                 getContext().BuiltinInfo.getName(builtinID));
2203:       }
2204:     }
2205:     ops.push_back(
2206:         emitScalarOrConstFoldImmArg(iceArguments, i, expr->getArg(i)));
2207:   }
2208: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `emitScalarOrConstFoldImmArg`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`emitScalarOrConstFoldImmArg`。

### Lines 2209-2213
```cpp
2209:   const ARMVectorIntrinsicInfo *builtin = findARMVectorIntrinsicInMap(
2210:       AArch64SISDIntrinsicMap, builtinID, aarch64SISDIntrinsicsProvenSorted);
2211:   if (builtin)
2212:     return emitCommonNeonSISDBuiltinExpr(*this, *builtin, ops, expr);
2213: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2214-2219
```cpp
2214:   // Not all intrinsics handled by the common case work for AArch64 yet, so only
2215:   // defer to common code if it's been added to our special map.
2216:   assert(!cir::MissingFeatures::aarch64SIMDIntrinsics());
2217: 
2218:   assert(!cir::MissingFeatures::aarch64TblBuiltinExpr());
2219: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2220-2231
```cpp
2220:   const Expr *arg = expr->getArg(expr->getNumArgs() - 1);
2221:   NeonTypeFlags type(0);
2222:   // A trailing constant integer is used for discriminating overloaded builtin
2223:   // calls. Use it to determine the type of this overloaded NEON intrinsic.
2224:   if (std::optional<llvm::APSInt> result =
2225:           arg->getIntegerConstantExpr(getContext()))
2226:     type = NeonTypeFlags(result->getZExtValue());
2227: 
2228:   bool usgn = type.isUnsigned();
2229: 
2230:   mlir::Location loc = getLoc(expr->getExprLoc());
2231: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `type`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `type`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2232-2241
```cpp
2232:   // Not all intrinsics handled by the common case work for AArch64 yet, so only
2233:   // defer to common code if it's been added to our special map.
2234:   builtin = findARMVectorIntrinsicInMap(AArch64SIMDIntrinsicMap, builtinID,
2235:                                         aarch64SIMDIntrinsicsProvenSorted);
2236:   if (builtin)
2237:     return emitCommonNeonBuiltinExpr(
2238:         *this, builtin->BuiltinID, builtin->LLVMIntrinsic,
2239:         builtin->AltLLVMIntrinsic, builtin->NameHint, builtin->TypeModifier,
2240:         expr, ops);
2241: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2242-2259
```cpp
2242:   // Handle non-overloaded intrinsics first.
2243:   switch (builtinID) {
2244:   default:
2245:     break;
2246:   case NEON::BI__builtin_neon_vabsh_f16: {
2247:     return cir::FAbsOp::create(builder, loc, ops);
2248:   }
2249:   case NEON::BI__builtin_neon_vaddq_p128:
2250:   case NEON::BI__builtin_neon_vldrq_p128:
2251:   case NEON::BI__builtin_neon_vstrq_p128:
2252:   case NEON::BI__builtin_neon_vcvts_f32_u32:
2253:   case NEON::BI__builtin_neon_vcvtd_f64_u64:
2254:   case NEON::BI__builtin_neon_vcvts_f32_s32:
2255:   case NEON::BI__builtin_neon_vcvtd_f64_s64:
2256:   case NEON::BI__builtin_neon_vcvth_f16_u16:
2257:   case NEON::BI__builtin_neon_vcvth_f16_u32:
2258:   case NEON::BI__builtin_neon_vcvth_f16_u64:
2259:   case NEON::BI__builtin_neon_vcvth_f16_s16:
```
- **EN**: This block contains local control-flow decisions or iterative processing. A switch statement is used to dispatch behavior across enumerated cases or kinds. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2260-2277
```cpp
2260:   case NEON::BI__builtin_neon_vcvth_f16_s32:
2261:   case NEON::BI__builtin_neon_vcvth_f16_s64:
2262:   case NEON::BI__builtin_neon_vcvtah_u16_f16:
2263:   case NEON::BI__builtin_neon_vcvtmh_u16_f16:
2264:   case NEON::BI__builtin_neon_vcvtnh_u16_f16:
2265:   case NEON::BI__builtin_neon_vcvtph_u16_f16:
2266:   case NEON::BI__builtin_neon_vcvth_u16_f16:
2267:   case NEON::BI__builtin_neon_vcvtah_s16_f16:
2268:   case NEON::BI__builtin_neon_vcvtmh_s16_f16:
2269:   case NEON::BI__builtin_neon_vcvtnh_s16_f16:
2270:   case NEON::BI__builtin_neon_vcvtph_s16_f16:
2271:   case NEON::BI__builtin_neon_vcvth_s16_f16:
2272:   case NEON::BI__builtin_neon_vcaleh_f16:
2273:   case NEON::BI__builtin_neon_vcalth_f16:
2274:   case NEON::BI__builtin_neon_vcageh_f16:
2275:   case NEON::BI__builtin_neon_vcagth_f16:
2276:   case NEON::BI__builtin_neon_vcvth_n_s16_f16:
2277:   case NEON::BI__builtin_neon_vcvth_n_u16_f16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2278-2295
```cpp
2278:   case NEON::BI__builtin_neon_vcvth_n_f16_s16:
2279:   case NEON::BI__builtin_neon_vcvth_n_f16_u16:
2280:   case NEON::BI__builtin_neon_vpaddd_s64:
2281:   case NEON::BI__builtin_neon_vpaddd_f64:
2282:   case NEON::BI__builtin_neon_vpadds_f32:
2283:     cgm.errorNYI(expr->getSourceRange(),
2284:                  std::string("unimplemented AArch64 builtin call: ") +
2285:                      getContext().BuiltinInfo.getName(builtinID));
2286:     return mlir::Value{};
2287:   case NEON::BI__builtin_neon_vceqzd_s64:
2288:   case NEON::BI__builtin_neon_vceqzd_f64:
2289:   case NEON::BI__builtin_neon_vceqzs_f32:
2290:   case NEON::BI__builtin_neon_vceqzh_f16:
2291:     return emitAArch64CompareBuiltinExpr(
2292:         *this, builder, loc, ops[0],
2293:         convertType(expr->getCallReturnType(getContext())), cir::CmpOpKind::eq);
2294:   case NEON::BI__builtin_neon_vcgezd_s64:
2295:   case NEON::BI__builtin_neon_vcgezd_f64:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `emitAArch64CompareBuiltinExpr`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`emitAArch64CompareBuiltinExpr`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2296-2313
```cpp
2296:   case NEON::BI__builtin_neon_vcgezs_f32:
2297:   case NEON::BI__builtin_neon_vcgezh_f16:
2298:   case NEON::BI__builtin_neon_vclezd_s64:
2299:   case NEON::BI__builtin_neon_vclezd_f64:
2300:   case NEON::BI__builtin_neon_vclezs_f32:
2301:   case NEON::BI__builtin_neon_vclezh_f16:
2302:   case NEON::BI__builtin_neon_vcgtzd_s64:
2303:   case NEON::BI__builtin_neon_vcgtzd_f64:
2304:   case NEON::BI__builtin_neon_vcgtzs_f32:
2305:   case NEON::BI__builtin_neon_vcgtzh_f16:
2306:   case NEON::BI__builtin_neon_vcltzd_s64:
2307:   case NEON::BI__builtin_neon_vcltzd_f64:
2308:   case NEON::BI__builtin_neon_vcltzs_f32:
2309:   case NEON::BI__builtin_neon_vcltzh_f16:
2310:   case NEON::BI__builtin_neon_vceqzd_u64: {
2311:     return emitAArch64CompareBuiltinExpr(
2312:         *this, builder, loc, ops[0],
2313:         convertType(expr->getCallReturnType(getContext())), cir::CmpOpKind::eq);
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2314-2331
```cpp
2314:   }
2315:   case NEON::BI__builtin_neon_vceqd_f64:
2316:   case NEON::BI__builtin_neon_vcled_f64:
2317:   case NEON::BI__builtin_neon_vcltd_f64:
2318:   case NEON::BI__builtin_neon_vcged_f64:
2319:   case NEON::BI__builtin_neon_vcgtd_f64:
2320:   case NEON::BI__builtin_neon_vceqs_f32:
2321:   case NEON::BI__builtin_neon_vcles_f32:
2322:   case NEON::BI__builtin_neon_vclts_f32:
2323:   case NEON::BI__builtin_neon_vcges_f32:
2324:   case NEON::BI__builtin_neon_vcgts_f32:
2325:   case NEON::BI__builtin_neon_vceqh_f16:
2326:   case NEON::BI__builtin_neon_vcleh_f16:
2327:   case NEON::BI__builtin_neon_vclth_f16:
2328:   case NEON::BI__builtin_neon_vcgeh_f16:
2329:   case NEON::BI__builtin_neon_vcgth_f16:
2330:   case NEON::BI__builtin_neon_vceqd_s64:
2331:   case NEON::BI__builtin_neon_vceqd_u64:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2332-2349
```cpp
2332:   case NEON::BI__builtin_neon_vcgtd_s64:
2333:   case NEON::BI__builtin_neon_vcgtd_u64:
2334:   case NEON::BI__builtin_neon_vcltd_s64:
2335:   case NEON::BI__builtin_neon_vcltd_u64:
2336:   case NEON::BI__builtin_neon_vcged_u64:
2337:   case NEON::BI__builtin_neon_vcged_s64:
2338:   case NEON::BI__builtin_neon_vcled_u64:
2339:   case NEON::BI__builtin_neon_vcled_s64:
2340:     cgm.errorNYI(expr->getSourceRange(),
2341:                  std::string("unimplemented AArch64 builtin call: ") +
2342:                      getContext().BuiltinInfo.getName(builtinID));
2343:     return mlir::Value{};
2344:   case NEON::BI__builtin_neon_vnegd_s64: {
2345:     return builder.createNeg(ops[0]);
2346:   }
2347:   case NEON::BI__builtin_neon_vnegh_f16: {
2348:     return builder.createFNeg(ops[0]);
2349:   }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2350-2367
```cpp
2350:   case NEON::BI__builtin_neon_vtstd_s64:
2351:   case NEON::BI__builtin_neon_vtstd_u64:
2352:   case NEON::BI__builtin_neon_vset_lane_i8:
2353:   case NEON::BI__builtin_neon_vset_lane_i16:
2354:   case NEON::BI__builtin_neon_vset_lane_i32:
2355:   case NEON::BI__builtin_neon_vset_lane_i64:
2356:   case NEON::BI__builtin_neon_vset_lane_bf16:
2357:   case NEON::BI__builtin_neon_vset_lane_f32:
2358:   case NEON::BI__builtin_neon_vsetq_lane_i8:
2359:   case NEON::BI__builtin_neon_vsetq_lane_i16:
2360:   case NEON::BI__builtin_neon_vsetq_lane_i32:
2361:   case NEON::BI__builtin_neon_vsetq_lane_i64:
2362:   case NEON::BI__builtin_neon_vsetq_lane_bf16:
2363:   case NEON::BI__builtin_neon_vsetq_lane_f32:
2364:   case NEON::BI__builtin_neon_vset_lane_f64:
2365:   case NEON::BI__builtin_neon_vset_lane_mf8:
2366:   case NEON::BI__builtin_neon_vsetq_lane_mf8:
2367:   case NEON::BI__builtin_neon_vsetq_lane_f64:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2368-2385
```cpp
2368:   case NEON::BI__builtin_neon_vget_lane_i8:
2369:   case NEON::BI__builtin_neon_vdupb_lane_i8:
2370:   case NEON::BI__builtin_neon_vgetq_lane_i8:
2371:   case NEON::BI__builtin_neon_vdupb_laneq_i8:
2372:   case NEON::BI__builtin_neon_vget_lane_mf8:
2373:   case NEON::BI__builtin_neon_vdupb_lane_mf8:
2374:   case NEON::BI__builtin_neon_vgetq_lane_mf8:
2375:   case NEON::BI__builtin_neon_vdupb_laneq_mf8:
2376:   case NEON::BI__builtin_neon_vget_lane_i16:
2377:   case NEON::BI__builtin_neon_vduph_lane_i16:
2378:   case NEON::BI__builtin_neon_vgetq_lane_i16:
2379:   case NEON::BI__builtin_neon_vduph_laneq_i16:
2380:   case NEON::BI__builtin_neon_vget_lane_i32:
2381:   case NEON::BI__builtin_neon_vdups_lane_i32:
2382:   case NEON::BI__builtin_neon_vdups_lane_f32:
2383:   case NEON::BI__builtin_neon_vgetq_lane_i32:
2384:   case NEON::BI__builtin_neon_vdups_laneq_i32:
2385:   case NEON::BI__builtin_neon_vget_lane_i64:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2386-2403
```cpp
2386:   case NEON::BI__builtin_neon_vdupd_lane_i64:
2387:   case NEON::BI__builtin_neon_vdupd_lane_f64:
2388:   case NEON::BI__builtin_neon_vgetq_lane_i64:
2389:   case NEON::BI__builtin_neon_vdupd_laneq_i64:
2390:   case NEON::BI__builtin_neon_vget_lane_f32:
2391:   case NEON::BI__builtin_neon_vget_lane_f64:
2392:   case NEON::BI__builtin_neon_vgetq_lane_f32:
2393:   case NEON::BI__builtin_neon_vdups_laneq_f32:
2394:   case NEON::BI__builtin_neon_vgetq_lane_f64:
2395:   case NEON::BI__builtin_neon_vdupd_laneq_f64:
2396:     cgm.errorNYI(expr->getSourceRange(),
2397:                  std::string("unimplemented AArch64 builtin call: ") +
2398:                      getContext().BuiltinInfo.getName(builtinID));
2399:     return mlir::Value{};
2400:   case NEON::BI__builtin_neon_vaddh_f16:
2401:     return builder.createFAdd(loc, ops[0], ops[1]);
2402:   case NEON::BI__builtin_neon_vsubh_f16:
2403:     return builder.createFSub(loc, ops[0], ops[1]);
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2404-2421
```cpp
2404:   case NEON::BI__builtin_neon_vmulh_f16:
2405:     return builder.createFMul(loc, ops[0], ops[1]);
2406:   case NEON::BI__builtin_neon_vdivh_f16:
2407:     return builder.createFDiv(loc, ops[0], ops[1]);
2408:   case NEON::BI__builtin_neon_vfmah_f16:
2409:     // NEON intrinsic puts accumulator first, unlike the LLVM fma.
2410:     std::rotate(ops.begin(), ops.begin() + 1, ops.end());
2411:     return emitCallMaybeConstrainedBuiltin(builder, loc, "fma",
2412:                                            convertType(expr->getType()), ops);
2413:     break;
2414:   case NEON::BI__builtin_neon_vfmsh_f16:
2415:     // NEON intrinsic puts accumulator first, unlike the LLVM fma.
2416:     std::rotate(ops.begin(), ops.begin() + 1, ops.end());
2417:     ops[0] = builder.createFNeg(ops[0]);
2418:     return emitCallMaybeConstrainedBuiltin(builder, loc, "fma",
2419:                                            convertType(expr->getType()), ops);
2420:   case NEON::BI__builtin_neon_vaddd_s64:
2421:   case NEON::BI__builtin_neon_vaddd_u64:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::rotate`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::rotate`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2422-2439
```cpp
2422:   case NEON::BI__builtin_neon_vsubd_s64:
2423:   case NEON::BI__builtin_neon_vsubd_u64:
2424:   case NEON::BI__builtin_neon_vqdmlalh_s16:
2425:   case NEON::BI__builtin_neon_vqdmlslh_s16:
2426:     cgm.errorNYI(expr->getSourceRange(),
2427:                  std::string("unimplemented AArch64 builtin call: ") +
2428:                      getContext().BuiltinInfo.getName(builtinID));
2429:     return mlir::Value{};
2430:   case NEON::BI__builtin_neon_vqshlud_n_s64: {
2431:     cir::IntType int64Type = builder.getSInt64Ty();
2432:     ops[1] = builder.getSInt64(getZExtIntValueFromConstOp(ops[1]), loc);
2433:     return emitNeonCall(cgm, builder, {int64Type, int64Type}, ops,
2434:                         "aarch64.neon.sqshlu", convertType(expr->getType()),
2435:                         loc);
2436:   }
2437:   case NEON::BI__builtin_neon_vqshld_n_u64:
2438:   case NEON::BI__builtin_neon_vqshld_n_s64: {
2439:     cir::IntType int64Type = builtinID == NEON::BI__builtin_neon_vqshld_n_u64
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2440-2457
```cpp
2440:                                  ? builder.getUInt64Ty()
2441:                                  : builder.getSInt64Ty();
2442:     llvm::StringRef intrinsicName =
2443:         builtinID == NEON::BI__builtin_neon_vqshld_n_u64 ? "aarch64.neon.uqshl"
2444:                                                          : "aarch64.neon.sqshl";
2445:     ops[1] = builder.getSInt64(getZExtIntValueFromConstOp(ops[1]), loc);
2446:     return emitNeonCall(cgm, builder, {int64Type, int64Type}, ops,
2447:                         intrinsicName, convertType(expr->getType()), loc);
2448:   }
2449:   case NEON::BI__builtin_neon_vrshrd_n_u64:
2450:   case NEON::BI__builtin_neon_vrshrd_n_s64: {
2451:     llvm::StringRef intrName = builtinID == NEON::BI__builtin_neon_vrshrd_n_s64
2452:                                    ? "aarch64.neon.srshl"
2453:                                    : "aarch64.neon.urshl";
2454:     cir::IntType int64Ty = builtinID == NEON::BI__builtin_neon_vqshld_n_u64
2455:                                ? builder.getUInt64Ty()
2456:                                : builder.getSInt64Ty();
2457:     int64_t sv = -cast<cir::IntAttr>(
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `convertType`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `convertType`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2458-2473
```cpp
2458:                       cast<cir::ConstantOp>(ops[1].getDefiningOp()).getValue())
2459:                       .getSInt();
2460:     ops[1] = builder.getSInt64(sv, loc);
2461:     return emitNeonCall(cgm, builder, {int64Ty, builder.getSInt64Ty()}, ops,
2462:                         intrName, int64Ty, loc);
2463:   }
2464:   case NEON::BI__builtin_neon_vrsrad_n_u64:
2465:   case NEON::BI__builtin_neon_vrsrad_n_s64: {
2466:     cir::IntType int64Type = builtinID == NEON::BI__builtin_neon_vrsrad_n_u64
2467:                                  ? builder.getUInt64Ty()
2468:                                  : builder.getSInt64Ty();
2469:     ops[2] = builder.createNeg(ops[2]);
2470:     const StringRef intrName = builtinID == NEON::BI__builtin_neon_vrsrad_n_u64
2471:                                    ? "aarch64.neon.urshl"
2472:                                    : "aarch64.neon.srshl";
2473: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2474-2491
```cpp
2474:     llvm::SmallVector<mlir::Value, 2> args = {
2475:         ops[1], builder.createIntCast(ops[2], builder.getSInt64Ty())};
2476:     ops[1] = builder.emitIntrinsicCallOp(loc, intrName, int64Type, args);
2477:     return builder.createAdd(loc, ops[0],
2478:                              builder.createBitcast(ops[1], int64Type));
2479:   }
2480:   case NEON::BI__builtin_neon_vshld_n_s64:
2481:   case NEON::BI__builtin_neon_vshld_n_u64: {
2482:     auto loc = getLoc(expr->getExprLoc());
2483:     std::optional<llvm::APSInt> amt =
2484:         expr->getArg(1)->getIntegerConstantExpr(getContext());
2485:     assert(amt && "Expected argument to be a constant");
2486:     return builder.createShiftLeft(loc, ops[0], amt->getZExtValue());
2487:   }
2488:   case NEON::BI__builtin_neon_vshrd_n_s64: {
2489:     std::optional<llvm::APSInt> amt =
2490:         expr->getArg(1)->getIntegerConstantExpr(getContext());
2491:     assert(amt && "Expected argument to be a constant");
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2492-2509
```cpp
2492:     return builder.createShiftRight(
2493:         loc, ops[0], std::min(static_cast<uint64_t>(63), amt->getZExtValue()));
2494:   }
2495:   case NEON::BI__builtin_neon_vshrd_n_u64: {
2496:     std::optional<llvm::APSInt> amt =
2497:         expr->getArg(1)->getIntegerConstantExpr(getContext());
2498:     assert(amt && "Expected argument to be a constant");
2499:     uint64_t shiftAmt = amt->getZExtValue();
2500:     // Right-shifting an unsigned value by its size yields 0.
2501:     if (shiftAmt == 64)
2502:       return builder.getConstInt(loc, builder.getUInt64Ty(), 0);
2503:     return builder.createShiftRight(loc, ops[0], shiftAmt);
2504:   }
2505:   case NEON::BI__builtin_neon_vsrad_n_s64:
2506:   case NEON::BI__builtin_neon_vsrad_n_u64:
2507:   case NEON::BI__builtin_neon_vqdmlalh_lane_s16:
2508:   case NEON::BI__builtin_neon_vqdmlalh_laneq_s16:
2509:   case NEON::BI__builtin_neon_vqdmlslh_lane_s16:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2510-2527
```cpp
2510:   case NEON::BI__builtin_neon_vqdmlslh_laneq_s16:
2511:   case NEON::BI__builtin_neon_vqdmlals_s32:
2512:   case NEON::BI__builtin_neon_vqdmlsls_s32:
2513:   case NEON::BI__builtin_neon_vqdmlals_lane_s32:
2514:   case NEON::BI__builtin_neon_vqdmlals_laneq_s32:
2515:   case NEON::BI__builtin_neon_vqdmlsls_lane_s32:
2516:   case NEON::BI__builtin_neon_vqdmlsls_laneq_s32: {
2517:     cgm.errorNYI(expr->getSourceRange(),
2518:                  std::string("unimplemented AArch64 builtin call: ") +
2519:                      getContext().BuiltinInfo.getName(builtinID));
2520:     return mlir::Value{};
2521:   }
2522:   case NEON::BI__builtin_neon_vget_lane_bf16:
2523:   case NEON::BI__builtin_neon_vduph_lane_bf16:
2524:   case NEON::BI__builtin_neon_vduph_lane_f16:
2525:   case NEON::BI__builtin_neon_vgetq_lane_bf16:
2526:   case NEON::BI__builtin_neon_vduph_laneq_bf16:
2527:   case NEON::BI__builtin_neon_vduph_laneq_f16: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2528-2545
```cpp
2528:     return cir::VecExtractOp::create(builder, loc, ops[0], ops[1]);
2529:   }
2530:   case NEON::BI__builtin_neon_vcvt_bf16_f32:
2531:   case NEON::BI__builtin_neon_vcvtq_low_bf16_f32:
2532:   case NEON::BI__builtin_neon_vcvtq_high_bf16_f32:
2533:   case clang::AArch64::BI_InterlockedAdd:
2534:   case clang::AArch64::BI_InterlockedAdd_acq:
2535:   case clang::AArch64::BI_InterlockedAdd_rel:
2536:   case clang::AArch64::BI_InterlockedAdd_nf:
2537:   case clang::AArch64::BI_InterlockedAdd64:
2538:   case clang::AArch64::BI_InterlockedAdd64_acq:
2539:   case clang::AArch64::BI_InterlockedAdd64_rel:
2540:   case clang::AArch64::BI_InterlockedAdd64_nf:
2541:     cgm.errorNYI(expr->getSourceRange(),
2542:                  std::string("unimplemented AArch64 builtin call: ") +
2543:                      getContext().BuiltinInfo.getName(builtinID));
2544:     return mlir::Value{};
2545:   }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2546-2552
```cpp
2546: 
2547:   cir::VectorType ty = getNeonType(this, type, loc);
2548:   if (!ty)
2549:     return nullptr;
2550: 
2551:   llvm::StringRef intrName;
2552: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2553-2558
```cpp
2553:   switch (builtinID) {
2554:   default:
2555:     return std::nullopt;
2556:   case NEON::BI__builtin_neon_vbsl_v:
2557:   case NEON::BI__builtin_neon_vbslq_v: {
2558: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. A switch statement is used to dispatch behavior across enumerated cases or kinds. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2559-2563
```cpp
2559:     cir::VectorType bitTy = getIntVecFromVecTy(builder, ty);
2560:     ops[0] = builder.createBitcast(ops[0], bitTy);
2561:     ops[1] = builder.createBitcast(ops[1], bitTy);
2562:     ops[2] = builder.createBitcast(ops[2], bitTy);
2563: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2564-2581
```cpp
2564:     ops[1] = builder.createAnd(loc, ops[0], ops[1]);
2565:     ops[2] = builder.createAnd(loc, builder.createNot(ops[0]), ops[2]);
2566:     ops[0] = builder.createOr(loc, ops[1], ops[2]);
2567:     return builder.createBitcast(ops[0], ty);
2568:   }
2569:   case NEON::BI__builtin_neon_vfma_lane_v:
2570:   case NEON::BI__builtin_neon_vfmaq_lane_v:
2571:   case NEON::BI__builtin_neon_vfma_laneq_v:
2572:   case NEON::BI__builtin_neon_vfmaq_laneq_v:
2573:   case NEON::BI__builtin_neon_vfmah_lane_f16:
2574:   case NEON::BI__builtin_neon_vfmas_lane_f32:
2575:   case NEON::BI__builtin_neon_vfmah_laneq_f16:
2576:   case NEON::BI__builtin_neon_vfmas_laneq_f32:
2577:   case NEON::BI__builtin_neon_vfmad_lane_f64:
2578:   case NEON::BI__builtin_neon_vfmad_laneq_f64:
2579:     cgm.errorNYI(expr->getSourceRange(),
2580:                  std::string("unimplemented AArch64 builtin call: ") +
2581:                      getContext().BuiltinInfo.getName(builtinID));
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 2582-2599
```cpp
2582:     return mlir::Value{};
2583:   case NEON::BI__builtin_neon_vmull_v: {
2584:     intrName = usgn ? "aarch64.neon.umull" : "aarch64.neon.smull";
2585:     if (type.isPoly())
2586:       intrName = "aarch64.neon.pmull";
2587:     cir::VectorType argTy = builder.getExtendedOrTruncatedElementVectorType(
2588:         ty, /*isExtended*/ false, !usgn);
2589:     return emitNeonCall(cgm, builder, {argTy, argTy}, ops, intrName, ty, loc);
2590:   }
2591:   case NEON::BI__builtin_neon_vmax_v:
2592:   case NEON::BI__builtin_neon_vmaxq_v:
2593:     intrName = usgn ? "aarch64.neon.umax" : "aarch64.neon.smax";
2594:     if (cir::isFPOrVectorOfFPType(ty))
2595:       intrName = "aarch64.neon.fmax";
2596:     return emitNeonCall(cgm, builder, {ty, ty}, ops, intrName, ty, loc);
2597:   case NEON::BI__builtin_neon_vmaxh_f16:
2598:     cgm.errorNYI(expr->getSourceRange(),
2599:                  std::string("unimplemented AArch64 builtin call: ") +
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2600-2617
```cpp
2600:                      getContext().BuiltinInfo.getName(builtinID));
2601:     return mlir::Value{};
2602:   case NEON::BI__builtin_neon_vmin_v:
2603:   case NEON::BI__builtin_neon_vminq_v:
2604:     intrName = usgn ? "aarch64.neon.umin" : "aarch64.neon.smin";
2605:     if (cir::isFPOrVectorOfFPType(ty))
2606:       intrName = "aarch64.neon.fmin";
2607:     return emitNeonCall(cgm, builder, {ty, ty}, ops, intrName, ty, loc);
2608:   case NEON::BI__builtin_neon_vminh_f16:
2609:     cgm.errorNYI(expr->getSourceRange(),
2610:                  std::string("unimplemented AArch64 builtin call: ") +
2611:                      getContext().BuiltinInfo.getName(builtinID));
2612:     return mlir::Value{};
2613:   case NEON::BI__builtin_neon_vabd_v:
2614:   case NEON::BI__builtin_neon_vabdq_v:
2615:     intrName = usgn ? "aarch64.neon.uabd" : "aarch64.neon.sabd";
2616:     if (cir::isFPOrVectorOfFPType(ty))
2617:       intrName = "aarch64.neon.fabd";
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getContext`, `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getContext`、`std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2618-2635
```cpp
2618:     return emitNeonCall(cgm, builder, {ty, ty}, ops, intrName, ty, loc);
2619:   case NEON::BI__builtin_neon_vpadal_v:
2620:   case NEON::BI__builtin_neon_vpadalq_v: {
2621:     intrName = usgn ? "aarch64.neon.uaddlp" : "aarch64.neon.saddlp";
2622:     llvm::SmallVector<mlir::Value> inputs{ops[1]};
2623:     mlir::Value pairwiseSum =
2624:         emitNeonCall(cgm, builder, {getNeonPairwiseWidenInputType(ty, usgn)},
2625:                      inputs, intrName, ty, loc);
2626:     mlir::Value accumValue = builder.createBitcast(loc, ops[0], ty);
2627:     return cir::AddOp::create(builder, loc, ty, pairwiseSum, accumValue);
2628:   }
2629:   case NEON::BI__builtin_neon_vpmin_v:
2630:   case NEON::BI__builtin_neon_vpminq_v:
2631:     intrName = usgn ? "aarch64.neon.uminp" : "aarch64.neon.sminp";
2632:     if (cir::isFPOrVectorOfFPType(ty))
2633:       intrName = "aarch64.neon.fminp";
2634:     return emitNeonCall(cgm, builder, {ty, ty}, ops, intrName, ty, loc);
2635:   case NEON::BI__builtin_neon_vpmax_v:
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2636-2653
```cpp
2636:   case NEON::BI__builtin_neon_vpmaxq_v:
2637:     cgm.errorNYI(expr->getSourceRange(),
2638:                  std::string("unimplemented AArch64 builtin call: ") +
2639:                      getContext().BuiltinInfo.getName(builtinID));
2640:     return mlir::Value{};
2641:   case NEON::BI__builtin_neon_vminnm_v:
2642:   case NEON::BI__builtin_neon_vminnmq_v:
2643:     intrName = "aarch64.neon.fminnm";
2644:     return emitNeonCall(cgm, builder, {ty, ty}, ops, intrName, ty, loc);
2645:   case NEON::BI__builtin_neon_vminnmh_f16:
2646:     cgm.errorNYI(expr->getSourceRange(),
2647:                  std::string("unimplemented AArch64 builtin call: ") +
2648:                      getContext().BuiltinInfo.getName(builtinID));
2649:     return mlir::Value{};
2650:   case NEON::BI__builtin_neon_vmaxnm_v:
2651:   case NEON::BI__builtin_neon_vmaxnmq_v:
2652:     intrName = "aarch64.neon.fmaxnm";
2653:     return emitNeonCall(cgm, builder, {ty, ty}, ops, intrName, ty, loc);
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2654-2671
```cpp
2654:   case NEON::BI__builtin_neon_vmaxnmh_f16:
2655:   case NEON::BI__builtin_neon_vrecpss_f32:
2656:   case NEON::BI__builtin_neon_vrecpsd_f64:
2657:   case NEON::BI__builtin_neon_vrecpsh_f16:
2658:   case NEON::BI__builtin_neon_vqshrun_n_v:
2659:   case NEON::BI__builtin_neon_vqrshrun_n_v:
2660:   case NEON::BI__builtin_neon_vqshrn_n_v:
2661:   case NEON::BI__builtin_neon_vrshrn_n_v:
2662:   case NEON::BI__builtin_neon_vqrshrn_n_v:
2663:   case NEON::BI__builtin_neon_vrndah_f16:
2664:   case NEON::BI__builtin_neon_vrnda_v:
2665:   case NEON::BI__builtin_neon_vrndaq_v:
2666:   case NEON::BI__builtin_neon_vrndih_f16:
2667:   case NEON::BI__builtin_neon_vrndmh_f16:
2668:   case NEON::BI__builtin_neon_vrndm_v:
2669:   case NEON::BI__builtin_neon_vrndmq_v:
2670:   case NEON::BI__builtin_neon_vrndnh_f16:
2671:   case NEON::BI__builtin_neon_vrndn_v:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2672-2689
```cpp
2672:   case NEON::BI__builtin_neon_vrndnq_v:
2673:   case NEON::BI__builtin_neon_vrndns_f32:
2674:   case NEON::BI__builtin_neon_vrndph_f16:
2675:   case NEON::BI__builtin_neon_vrndp_v:
2676:   case NEON::BI__builtin_neon_vrndpq_v:
2677:   case NEON::BI__builtin_neon_vrndxh_f16:
2678:   case NEON::BI__builtin_neon_vrndx_v:
2679:   case NEON::BI__builtin_neon_vrndxq_v:
2680:   case NEON::BI__builtin_neon_vrndh_f16:
2681:   case NEON::BI__builtin_neon_vrnd32x_f32:
2682:   case NEON::BI__builtin_neon_vrnd32xq_f32:
2683:   case NEON::BI__builtin_neon_vrnd32x_f64:
2684:   case NEON::BI__builtin_neon_vrnd32xq_f64:
2685:   case NEON::BI__builtin_neon_vrnd32z_f32:
2686:   case NEON::BI__builtin_neon_vrnd32zq_f32:
2687:   case NEON::BI__builtin_neon_vrnd32z_f64:
2688:   case NEON::BI__builtin_neon_vrnd32zq_f64:
2689:   case NEON::BI__builtin_neon_vrnd64x_f32:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2690-2707
```cpp
2690:   case NEON::BI__builtin_neon_vrnd64xq_f32:
2691:   case NEON::BI__builtin_neon_vrnd64x_f64:
2692:   case NEON::BI__builtin_neon_vrnd64xq_f64:
2693:   case NEON::BI__builtin_neon_vrnd64z_f32:
2694:   case NEON::BI__builtin_neon_vrnd64zq_f32:
2695:   case NEON::BI__builtin_neon_vrnd64z_f64:
2696:   case NEON::BI__builtin_neon_vrnd64zq_f64:
2697:   case NEON::BI__builtin_neon_vrnd_v:
2698:   case NEON::BI__builtin_neon_vrndq_v:
2699:     cgm.errorNYI(expr->getSourceRange(),
2700:                  std::string("unimplemented AArch64 builtin call: ") +
2701:                      getContext().BuiltinInfo.getName(builtinID));
2702:     return mlir::Value{};
2703:   case NEON::BI__builtin_neon_vcvt_f64_v:
2704:   case NEON::BI__builtin_neon_vcvtq_f64_v:
2705:     ops[0] = builder.createBitcast(ops[0], ty);
2706:     ty = getNeonType(
2707:         this, NeonTypeFlags(NeonTypeFlags::Float64, false, type.isQuad()), loc);
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `NeonTypeFlags`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`NeonTypeFlags`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2708-2725
```cpp
2708:     return builder.createCast(loc, cir::CastKind::int_to_float, ops[0], ty);
2709:   case NEON::BI__builtin_neon_vcvt_f64_f32:
2710:   case NEON::BI__builtin_neon_vcvt_f32_f64:
2711:   case NEON::BI__builtin_neon_vcvt_s32_v:
2712:   case NEON::BI__builtin_neon_vcvt_u32_v:
2713:   case NEON::BI__builtin_neon_vcvt_s64_v:
2714:   case NEON::BI__builtin_neon_vcvt_u64_v:
2715:   case NEON::BI__builtin_neon_vcvt_s16_f16:
2716:   case NEON::BI__builtin_neon_vcvt_u16_f16:
2717:   case NEON::BI__builtin_neon_vcvtq_s32_v:
2718:   case NEON::BI__builtin_neon_vcvtq_u32_v:
2719:   case NEON::BI__builtin_neon_vcvtq_s64_v:
2720:   case NEON::BI__builtin_neon_vcvtq_u64_v:
2721:   case NEON::BI__builtin_neon_vcvtq_s16_f16:
2722:   case NEON::BI__builtin_neon_vcvtq_u16_f16:
2723:   case NEON::BI__builtin_neon_vcvta_s16_f16:
2724:   case NEON::BI__builtin_neon_vcvta_u16_f16:
2725:   case NEON::BI__builtin_neon_vcvta_s32_v:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2726-2743
```cpp
2726:   case NEON::BI__builtin_neon_vcvtaq_s16_f16:
2727:   case NEON::BI__builtin_neon_vcvtaq_s32_v:
2728:   case NEON::BI__builtin_neon_vcvta_u32_v:
2729:   case NEON::BI__builtin_neon_vcvtaq_u16_f16:
2730:   case NEON::BI__builtin_neon_vcvtaq_u32_v:
2731:   case NEON::BI__builtin_neon_vcvta_s64_v:
2732:   case NEON::BI__builtin_neon_vcvtaq_s64_v:
2733:   case NEON::BI__builtin_neon_vcvta_u64_v:
2734:   case NEON::BI__builtin_neon_vcvtaq_u64_v:
2735:   case NEON::BI__builtin_neon_vcvtm_s16_f16:
2736:   case NEON::BI__builtin_neon_vcvtm_s32_v:
2737:   case NEON::BI__builtin_neon_vcvtmq_s16_f16:
2738:   case NEON::BI__builtin_neon_vcvtmq_s32_v:
2739:   case NEON::BI__builtin_neon_vcvtm_u16_f16:
2740:   case NEON::BI__builtin_neon_vcvtm_u32_v:
2741:   case NEON::BI__builtin_neon_vcvtmq_u16_f16:
2742:   case NEON::BI__builtin_neon_vcvtmq_u32_v:
2743:   case NEON::BI__builtin_neon_vcvtm_s64_v:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2744-2761
```cpp
2744:   case NEON::BI__builtin_neon_vcvtmq_s64_v:
2745:   case NEON::BI__builtin_neon_vcvtm_u64_v:
2746:   case NEON::BI__builtin_neon_vcvtmq_u64_v:
2747:   case NEON::BI__builtin_neon_vcvtn_s16_f16:
2748:   case NEON::BI__builtin_neon_vcvtn_s32_v:
2749:   case NEON::BI__builtin_neon_vcvtnq_s16_f16:
2750:   case NEON::BI__builtin_neon_vcvtnq_s32_v:
2751:   case NEON::BI__builtin_neon_vcvtn_u16_f16:
2752:   case NEON::BI__builtin_neon_vcvtn_u32_v:
2753:   case NEON::BI__builtin_neon_vcvtnq_u16_f16:
2754:   case NEON::BI__builtin_neon_vcvtnq_u32_v:
2755:   case NEON::BI__builtin_neon_vcvtn_s64_v:
2756:   case NEON::BI__builtin_neon_vcvtnq_s64_v:
2757:   case NEON::BI__builtin_neon_vcvtn_u64_v:
2758:   case NEON::BI__builtin_neon_vcvtnq_u64_v:
2759:   case NEON::BI__builtin_neon_vcvtp_s16_f16:
2760:   case NEON::BI__builtin_neon_vcvtp_s32_v:
2761:   case NEON::BI__builtin_neon_vcvtpq_s16_f16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2762-2779
```cpp
2762:   case NEON::BI__builtin_neon_vcvtpq_s32_v:
2763:   case NEON::BI__builtin_neon_vcvtp_u16_f16:
2764:   case NEON::BI__builtin_neon_vcvtp_u32_v:
2765:   case NEON::BI__builtin_neon_vcvtpq_u16_f16:
2766:   case NEON::BI__builtin_neon_vcvtpq_u32_v:
2767:   case NEON::BI__builtin_neon_vcvtp_s64_v:
2768:   case NEON::BI__builtin_neon_vcvtpq_s64_v:
2769:   case NEON::BI__builtin_neon_vcvtp_u64_v:
2770:   case NEON::BI__builtin_neon_vcvtpq_u64_v:
2771:   case NEON::BI__builtin_neon_vmulx_v:
2772:   case NEON::BI__builtin_neon_vmulxq_v:
2773:   case NEON::BI__builtin_neon_vmulxh_lane_f16:
2774:   case NEON::BI__builtin_neon_vmulxh_laneq_f16:
2775:   case NEON::BI__builtin_neon_vmul_lane_v:
2776:   case NEON::BI__builtin_neon_vmul_laneq_v:
2777:   case NEON::BI__builtin_neon_vpmaxnm_v:
2778:   case NEON::BI__builtin_neon_vpmaxnmq_v:
2779:     cgm.errorNYI(expr->getSourceRange(),
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2780-2797
```cpp
2780:                  std::string("unimplemented AArch64 builtin call: ") +
2781:                      getContext().BuiltinInfo.getName(builtinID));
2782:     return mlir::Value{};
2783:   case NEON::BI__builtin_neon_vpminnm_v:
2784:   case NEON::BI__builtin_neon_vpminnmq_v:
2785:     intrName = "aarch64.neon.fminnmp";
2786:     return emitNeonCall(cgm, builder, {ty, ty}, ops, intrName, ty, loc);
2787:   case NEON::BI__builtin_neon_vsqrth_f16:
2788:     cgm.errorNYI(expr->getSourceRange(),
2789:                  std::string("unimplemented AArch64 builtin call: ") +
2790:                      getContext().BuiltinInfo.getName(builtinID));
2791:     return mlir::Value{};
2792:   case NEON::BI__builtin_neon_vsqrt_v:
2793:   case NEON::BI__builtin_neon_vsqrtq_v:
2794:     assert(!cir::MissingFeatures::emitConstrainedFPCall());
2795:     return emitNeonCall(cgm, builder, {ty}, ops, "sqrt", ty, loc);
2796:   case NEON::BI__builtin_neon_vrbit_v:
2797:   case NEON::BI__builtin_neon_vrbitq_v:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `assert`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`assert`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2798-2815
```cpp
2798:   case NEON::BI__builtin_neon_vmaxv_f16:
2799:   case NEON::BI__builtin_neon_vmaxvq_f16:
2800:   case NEON::BI__builtin_neon_vminv_f16:
2801:   case NEON::BI__builtin_neon_vminvq_f16:
2802:   case NEON::BI__builtin_neon_vmaxnmv_f16:
2803:   case NEON::BI__builtin_neon_vmaxnmvq_f16:
2804:   case NEON::BI__builtin_neon_vminnmv_f16:
2805:   case NEON::BI__builtin_neon_vminnmvq_f16:
2806:   case NEON::BI__builtin_neon_vmul_n_f64:
2807:     cgm.errorNYI(expr->getSourceRange(),
2808:                  std::string("unimplemented AArch64 builtin call: ") +
2809:                      getContext().BuiltinInfo.getName(builtinID));
2810:     return mlir::Value{};
2811:   case NEON::BI__builtin_neon_vaddlv_u8:
2812:   case NEON::BI__builtin_neon_vaddlvq_u8:
2813:   case NEON::BI__builtin_neon_vaddlv_u16:
2814:   case NEON::BI__builtin_neon_vaddlvq_u16:
2815:   case NEON::BI__builtin_neon_vaddlv_s8:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2816-2833
```cpp
2816:   case NEON::BI__builtin_neon_vaddlvq_s8:
2817:   case NEON::BI__builtin_neon_vaddlv_s16:
2818:   case NEON::BI__builtin_neon_vaddlvq_s16: {
2819:     mlir::Type argTy = convertType(expr->getArg(0)->getType());
2820:     mlir::Type userRetTy = convertType(expr->getType());
2821:     auto eltTy = mlir::cast<cir::IntType>(
2822:         mlir::cast<cir::VectorType>(argTy).getElementType());
2823:     bool isUnsigned = !eltTy.isSigned();
2824:     // These builtins only use 8 and 16-bit element vectors; the intrinsic
2825:     // always produces i32. The C result is i32 for 16-bit elements, but i16
2826:     // for 8-bit elements, so we emit at i32 and narrow only in that case.
2827:     bool needsTrunc = eltTy.getWidth() == 8;
2828:     intrName = isUnsigned ? "aarch64.neon.uaddlv" : "aarch64.neon.saddlv";
2829:     mlir::Type intrRetTy = userRetTy;
2830:     if (needsTrunc)
2831:       intrRetTy = isUnsigned ? builder.getUInt32Ty() : builder.getSInt32Ty();
2832:     mlir::Value result =
2833:         emitNeonCall(cgm, builder, {argTy}, ops, intrName, intrRetTy, loc);
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2834-2851
```cpp
2834:     if (needsTrunc)
2835:       result = builder.createIntCast(result, userRetTy);
2836:     return result;
2837:   }
2838:   case NEON::BI__builtin_neon_vsri_n_v:
2839:   case NEON::BI__builtin_neon_vsriq_n_v:
2840:   case NEON::BI__builtin_neon_vsli_n_v:
2841:   case NEON::BI__builtin_neon_vsliq_n_v:
2842:   case NEON::BI__builtin_neon_vsra_n_v:
2843:   case NEON::BI__builtin_neon_vsraq_n_v:
2844:     cgm.errorNYI(expr->getSourceRange(),
2845:                  std::string("unimplemented AArch64 builtin call: ") +
2846:                      getContext().BuiltinInfo.getName(builtinID));
2847:     return mlir::Value{};
2848:   case NEON::BI__builtin_neon_vrsra_n_v:
2849:   case NEON::BI__builtin_neon_vrsraq_n_v: {
2850:     intrName = usgn ? "aarch64.neon.urshl" : "aarch64.neon.srshl";
2851:     // The llvm intrinsic is expecting negative shift amount for right shift.
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2852-2869
```cpp
2852:     // Thus we have to make shift amount vec type to be signed.
2853:     cir::VectorType shiftAmtVecTy =
2854:         usgn ? getSignChangedVectorType(builder, ty) : ty;
2855:     llvm::SmallVector<mlir::Value, 2> tmpOps = {ops[1], ops[2]};
2856:     mlir::Value tmp = emitNeonCall(cgm, builder, {ty, shiftAmtVecTy}, tmpOps,
2857:                                    intrName, ty, loc,
2858:                                    /*isConstrainedFPIntrinsic=*/false,
2859:                                    /*shift=*/1, /*rightshift=*/true);
2860:     ops[0] = builder.createBitcast(ops[0], ty);
2861:     return builder.createAdd(loc, ops[0], tmp);
2862:   }
2863:   case NEON::BI__builtin_neon_vld1_v:
2864:   case NEON::BI__builtin_neon_vld1q_v:
2865:   case NEON::BI__builtin_neon_vst1_v:
2866:   case NEON::BI__builtin_neon_vst1q_v:
2867:   case NEON::BI__builtin_neon_vld1_lane_v:
2868:   case NEON::BI__builtin_neon_vld1q_lane_v:
2869:   case NEON::BI__builtin_neon_vldap1_lane_s64:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2870-2887
```cpp
2870:   case NEON::BI__builtin_neon_vldap1q_lane_s64:
2871:   case NEON::BI__builtin_neon_vld1_dup_v:
2872:   case NEON::BI__builtin_neon_vld1q_dup_v:
2873:   case NEON::BI__builtin_neon_vst1_lane_v:
2874:   case NEON::BI__builtin_neon_vst1q_lane_v:
2875:   case NEON::BI__builtin_neon_vstl1_lane_s64:
2876:   case NEON::BI__builtin_neon_vstl1q_lane_s64:
2877:   case NEON::BI__builtin_neon_vld2_v:
2878:   case NEON::BI__builtin_neon_vld2q_v:
2879:   case NEON::BI__builtin_neon_vld3_v:
2880:   case NEON::BI__builtin_neon_vld3q_v:
2881:   case NEON::BI__builtin_neon_vld4_v:
2882:   case NEON::BI__builtin_neon_vld4q_v:
2883:   case NEON::BI__builtin_neon_vld2_dup_v:
2884:   case NEON::BI__builtin_neon_vld2q_dup_v:
2885:   case NEON::BI__builtin_neon_vld3_dup_v:
2886:   case NEON::BI__builtin_neon_vld3q_dup_v:
2887:   case NEON::BI__builtin_neon_vld4_dup_v:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2888-2905
```cpp
2888:   case NEON::BI__builtin_neon_vld4q_dup_v:
2889:   case NEON::BI__builtin_neon_vld2_lane_v:
2890:   case NEON::BI__builtin_neon_vld2q_lane_v:
2891:   case NEON::BI__builtin_neon_vld3_lane_v:
2892:   case NEON::BI__builtin_neon_vld3q_lane_v:
2893:   case NEON::BI__builtin_neon_vld4_lane_v:
2894:   case NEON::BI__builtin_neon_vld4q_lane_v:
2895:   case NEON::BI__builtin_neon_vst2_v:
2896:   case NEON::BI__builtin_neon_vst2q_v:
2897:   case NEON::BI__builtin_neon_vst2_lane_v:
2898:   case NEON::BI__builtin_neon_vst2q_lane_v:
2899:   case NEON::BI__builtin_neon_vst3_v:
2900:   case NEON::BI__builtin_neon_vst3q_v:
2901:   case NEON::BI__builtin_neon_vst3_lane_v:
2902:   case NEON::BI__builtin_neon_vst3q_lane_v:
2903:   case NEON::BI__builtin_neon_vst4_v:
2904:   case NEON::BI__builtin_neon_vst4q_v:
2905:   case NEON::BI__builtin_neon_vst4_lane_v:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2906-2919
```cpp
2906:   case NEON::BI__builtin_neon_vst4q_lane_v:
2907:     cgm.errorNYI(expr->getSourceRange(),
2908:                  std::string("unimplemented AArch64 builtin call: ") +
2909:                      getContext().BuiltinInfo.getName(builtinID));
2910:     return mlir::Value{};
2911:   case NEON::BI__builtin_neon_vtrn_v:
2912:   case NEON::BI__builtin_neon_vtrnq_v: {
2913:     ops[1] = builder.createBitcast(ops[1], ty);
2914:     ops[2] = builder.createBitcast(ops[2], ty);
2915:     // Adding a bitcast here as Ops[0] might be a void pointer.
2916:     mlir::Value baseAddr =
2917:         builder.createBitcast(ops[0], builder.getPointerTo(ty));
2918:     mlir::Value sv;
2919: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2920-2937
```cpp
2920:     for (unsigned vi = 0; vi != 2; ++vi) {
2921:       llvm::SmallVector<int64_t, 16> indices;
2922:       for (unsigned i = 0, e = ty.getSize(); i != e; i += 2) {
2923:         indices.push_back(i + vi);
2924:         indices.push_back(i + e + vi);
2925:       }
2926:       cir::ConstantOp idx = builder.getConstInt(loc, builder.getSInt32Ty(), vi);
2927:       mlir::Value addr = builder.createPtrStride(loc, baseAddr, idx);
2928:       sv = builder.createVecShuffle(loc, ops[1], ops[2], indices);
2929:       (void)builder.CIRBaseBuilderTy::createStore(loc, sv, addr);
2930:     }
2931:     return sv;
2932:   }
2933:   case NEON::BI__builtin_neon_vuzp_v:
2934:   case NEON::BI__builtin_neon_vuzpq_v: {
2935:     ops[1] = builder.createBitcast(ops[1], ty);
2936:     ops[2] = builder.createBitcast(ops[2], ty);
2937:     // Adding a bitcast here as Ops[0] might be a void pointer.
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2938-2955
```cpp
2938:     mlir::Value baseAddr =
2939:         builder.createBitcast(ops[0], builder.getPointerTo(ty));
2940:     mlir::Value sv;
2941:     for (unsigned vi = 0; vi != 2; ++vi) {
2942:       llvm::SmallVector<int64_t, 16> indices;
2943:       for (unsigned i = 0, e = ty.getSize(); i != e; ++i) {
2944:         indices.push_back(2 * i + vi);
2945:       }
2946:       cir::ConstantOp idx = builder.getConstInt(loc, builder.getSInt32Ty(), vi);
2947:       mlir::Value addr = builder.createPtrStride(loc, baseAddr, idx);
2948:       sv = builder.createVecShuffle(loc, ops[1], ops[2], indices);
2949:       (void)builder.CIRBaseBuilderTy::createStore(loc, sv, addr);
2950:     }
2951:     return sv;
2952:   }
2953:   case NEON::BI__builtin_neon_vzip_v:
2954:   case NEON::BI__builtin_neon_vzipq_v: {
2955:     ops[1] = builder.createBitcast(ops[1], ty);
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2956-2973
```cpp
2956:     ops[2] = builder.createBitcast(ops[2], ty);
2957:     // Adding a bitcast here as Ops[0] might be a void pointer.
2958:     mlir::Value baseAddr =
2959:         builder.createBitcast(ops[0], builder.getPointerTo(ty));
2960:     mlir::Value sv;
2961:     for (unsigned vi = 0; vi != 2; ++vi) {
2962:       llvm::SmallVector<int64_t, 16> indices;
2963:       for (unsigned i = 0, e = ty.getSize(); i != e; i += 2) {
2964:         indices.push_back((i + vi * e) >> 1);
2965:         indices.push_back(((i + vi * e) >> 1) + e);
2966:       }
2967:       cir::ConstantOp idx = builder.getConstInt(loc, builder.getSInt32Ty(), vi);
2968:       mlir::Value addr = builder.createPtrStride(loc, baseAddr, idx);
2969:       sv = builder.createVecShuffle(loc, ops[1], ops[2], indices);
2970:       (void)builder.CIRBaseBuilderTy::createStore(loc, sv, addr);
2971:     }
2972:     return sv;
2973:   }
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 2974-2991
```cpp
2974:   case NEON::BI__builtin_neon_vqtbl1q_v:
2975:   case NEON::BI__builtin_neon_vqtbl2q_v:
2976:   case NEON::BI__builtin_neon_vqtbl3q_v:
2977:   case NEON::BI__builtin_neon_vqtbl4q_v:
2978:   case NEON::BI__builtin_neon_vqtbx1q_v:
2979:   case NEON::BI__builtin_neon_vqtbx2q_v:
2980:   case NEON::BI__builtin_neon_vqtbx3q_v:
2981:   case NEON::BI__builtin_neon_vqtbx4q_v:
2982:   case NEON::BI__builtin_neon_vsqadd_v:
2983:   case NEON::BI__builtin_neon_vsqaddq_v:
2984:   case NEON::BI__builtin_neon_vuqadd_v:
2985:   case NEON::BI__builtin_neon_vuqaddq_v:
2986:   case NEON::BI__builtin_neon_vluti2_laneq_mf8:
2987:   case NEON::BI__builtin_neon_vluti2_laneq_bf16:
2988:   case NEON::BI__builtin_neon_vluti2_laneq_f16:
2989:   case NEON::BI__builtin_neon_vluti2_laneq_p16:
2990:   case NEON::BI__builtin_neon_vluti2_laneq_p8:
2991:   case NEON::BI__builtin_neon_vluti2_laneq_s16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 2992-3009
```cpp
2992:   case NEON::BI__builtin_neon_vluti2_laneq_s8:
2993:   case NEON::BI__builtin_neon_vluti2_laneq_u16:
2994:   case NEON::BI__builtin_neon_vluti2_laneq_u8:
2995:   case NEON::BI__builtin_neon_vluti2q_laneq_mf8:
2996:   case NEON::BI__builtin_neon_vluti2q_laneq_bf16:
2997:   case NEON::BI__builtin_neon_vluti2q_laneq_f16:
2998:   case NEON::BI__builtin_neon_vluti2q_laneq_p16:
2999:   case NEON::BI__builtin_neon_vluti2q_laneq_p8:
3000:   case NEON::BI__builtin_neon_vluti2q_laneq_s16:
3001:   case NEON::BI__builtin_neon_vluti2q_laneq_s8:
3002:   case NEON::BI__builtin_neon_vluti2q_laneq_u16:
3003:   case NEON::BI__builtin_neon_vluti2q_laneq_u8:
3004:   case NEON::BI__builtin_neon_vluti2_lane_mf8:
3005:   case NEON::BI__builtin_neon_vluti2_lane_bf16:
3006:   case NEON::BI__builtin_neon_vluti2_lane_f16:
3007:   case NEON::BI__builtin_neon_vluti2_lane_p16:
3008:   case NEON::BI__builtin_neon_vluti2_lane_p8:
3009:   case NEON::BI__builtin_neon_vluti2_lane_s16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 3010-3027
```cpp
3010:   case NEON::BI__builtin_neon_vluti2_lane_s8:
3011:   case NEON::BI__builtin_neon_vluti2_lane_u16:
3012:   case NEON::BI__builtin_neon_vluti2_lane_u8:
3013:   case NEON::BI__builtin_neon_vluti2q_lane_mf8:
3014:   case NEON::BI__builtin_neon_vluti2q_lane_bf16:
3015:   case NEON::BI__builtin_neon_vluti2q_lane_f16:
3016:   case NEON::BI__builtin_neon_vluti2q_lane_p16:
3017:   case NEON::BI__builtin_neon_vluti2q_lane_p8:
3018:   case NEON::BI__builtin_neon_vluti2q_lane_s16:
3019:   case NEON::BI__builtin_neon_vluti2q_lane_s8:
3020:   case NEON::BI__builtin_neon_vluti2q_lane_u16:
3021:   case NEON::BI__builtin_neon_vluti2q_lane_u8:
3022:   case NEON::BI__builtin_neon_vluti4q_lane_mf8:
3023:   case NEON::BI__builtin_neon_vluti4q_lane_p8:
3024:   case NEON::BI__builtin_neon_vluti4q_lane_s8:
3025:   case NEON::BI__builtin_neon_vluti4q_lane_u8:
3026:   case NEON::BI__builtin_neon_vluti4q_laneq_mf8:
3027:   case NEON::BI__builtin_neon_vluti4q_laneq_p8:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 3028-3045
```cpp
3028:   case NEON::BI__builtin_neon_vluti4q_laneq_s8:
3029:   case NEON::BI__builtin_neon_vluti4q_laneq_u8:
3030:   case NEON::BI__builtin_neon_vluti4q_lane_bf16_x2:
3031:   case NEON::BI__builtin_neon_vluti4q_lane_f16_x2:
3032:   case NEON::BI__builtin_neon_vluti4q_lane_p16_x2:
3033:   case NEON::BI__builtin_neon_vluti4q_lane_s16_x2:
3034:   case NEON::BI__builtin_neon_vluti4q_lane_u16_x2:
3035:   case NEON::BI__builtin_neon_vluti4q_laneq_bf16_x2:
3036:   case NEON::BI__builtin_neon_vluti4q_laneq_f16_x2:
3037:   case NEON::BI__builtin_neon_vluti4q_laneq_p16_x2:
3038:   case NEON::BI__builtin_neon_vluti4q_laneq_s16_x2:
3039:   case NEON::BI__builtin_neon_vluti4q_laneq_u16_x2:
3040:   case NEON::BI__builtin_neon_vmmlaq_f16_mf8_fpm:
3041:   case NEON::BI__builtin_neon_vmmlaq_f32_mf8_fpm:
3042:   case NEON::BI__builtin_neon_vcvt1_low_bf16_mf8_fpm:
3043:   case NEON::BI__builtin_neon_vcvt1_bf16_mf8_fpm:
3044:   case NEON::BI__builtin_neon_vcvt1_high_bf16_mf8_fpm:
3045:   case NEON::BI__builtin_neon_vcvt2_low_bf16_mf8_fpm:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 3046-3063
```cpp
3046:   case NEON::BI__builtin_neon_vcvt2_bf16_mf8_fpm:
3047:   case NEON::BI__builtin_neon_vcvt2_high_bf16_mf8_fpm:
3048:   case NEON::BI__builtin_neon_vcvt1_low_f16_mf8_fpm:
3049:   case NEON::BI__builtin_neon_vcvt1_f16_mf8_fpm:
3050:   case NEON::BI__builtin_neon_vcvt1_high_f16_mf8_fpm:
3051:   case NEON::BI__builtin_neon_vcvt2_low_f16_mf8_fpm:
3052:   case NEON::BI__builtin_neon_vcvt2_f16_mf8_fpm:
3053:   case NEON::BI__builtin_neon_vcvt2_high_f16_mf8_fpm:
3054:   case NEON::BI__builtin_neon_vcvt_mf8_f32_fpm:
3055:   case NEON::BI__builtin_neon_vcvt_mf8_f16_fpm:
3056:   case NEON::BI__builtin_neon_vcvtq_mf8_f16_fpm:
3057:   case NEON::BI__builtin_neon_vcvt_high_mf8_f32_fpm:
3058:   case NEON::BI__builtin_neon_vdot_f16_mf8_fpm:
3059:   case NEON::BI__builtin_neon_vdotq_f16_mf8_fpm:
3060:   case NEON::BI__builtin_neon_vdot_lane_f16_mf8_fpm:
3061:   case NEON::BI__builtin_neon_vdotq_lane_f16_mf8_fpm:
3062:   case NEON::BI__builtin_neon_vdot_laneq_f16_mf8_fpm:
3063:   case NEON::BI__builtin_neon_vdotq_laneq_f16_mf8_fpm:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 3064-3081
```cpp
3064:   case NEON::BI__builtin_neon_vdot_f32_mf8_fpm:
3065:   case NEON::BI__builtin_neon_vdotq_f32_mf8_fpm:
3066:   case NEON::BI__builtin_neon_vdot_lane_f32_mf8_fpm:
3067:   case NEON::BI__builtin_neon_vdotq_lane_f32_mf8_fpm:
3068:   case NEON::BI__builtin_neon_vdot_laneq_f32_mf8_fpm:
3069:   case NEON::BI__builtin_neon_vdotq_laneq_f32_mf8_fpm:
3070:   case NEON::BI__builtin_neon_vmlalbq_f16_mf8_fpm:
3071:   case NEON::BI__builtin_neon_vmlaltq_f16_mf8_fpm:
3072:   case NEON::BI__builtin_neon_vmlallbbq_f32_mf8_fpm:
3073:   case NEON::BI__builtin_neon_vmlallbtq_f32_mf8_fpm:
3074:   case NEON::BI__builtin_neon_vmlalltbq_f32_mf8_fpm:
3075:   case NEON::BI__builtin_neon_vmlallttq_f32_mf8_fpm:
3076:   case NEON::BI__builtin_neon_vmlalbq_lane_f16_mf8_fpm:
3077:   case NEON::BI__builtin_neon_vmlalbq_laneq_f16_mf8_fpm:
3078:   case NEON::BI__builtin_neon_vmlaltq_lane_f16_mf8_fpm:
3079:   case NEON::BI__builtin_neon_vmlaltq_laneq_f16_mf8_fpm:
3080:   case NEON::BI__builtin_neon_vmlallbbq_lane_f32_mf8_fpm:
3081:   case NEON::BI__builtin_neon_vmlallbbq_laneq_f32_mf8_fpm:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 3082-3099
```cpp
3082:   case NEON::BI__builtin_neon_vmlallbtq_lane_f32_mf8_fpm:
3083:   case NEON::BI__builtin_neon_vmlallbtq_laneq_f32_mf8_fpm:
3084:   case NEON::BI__builtin_neon_vmlalltbq_lane_f32_mf8_fpm:
3085:   case NEON::BI__builtin_neon_vmlalltbq_laneq_f32_mf8_fpm:
3086:   case NEON::BI__builtin_neon_vmlallttq_lane_f32_mf8_fpm:
3087:   case NEON::BI__builtin_neon_vmlallttq_laneq_f32_mf8_fpm:
3088:   case NEON::BI__builtin_neon_vamin_f16:
3089:   case NEON::BI__builtin_neon_vaminq_f16:
3090:   case NEON::BI__builtin_neon_vamin_f32:
3091:   case NEON::BI__builtin_neon_vaminq_f32:
3092:   case NEON::BI__builtin_neon_vaminq_f64:
3093:   case NEON::BI__builtin_neon_vamax_f16:
3094:   case NEON::BI__builtin_neon_vamaxq_f16:
3095:   case NEON::BI__builtin_neon_vamax_f32:
3096:   case NEON::BI__builtin_neon_vamaxq_f32:
3097:   case NEON::BI__builtin_neon_vamaxq_f64:
3098:   case NEON::BI__builtin_neon_vscale_f16:
3099:   case NEON::BI__builtin_neon_vscaleq_f16:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 3100-3110
```cpp
3100:   case NEON::BI__builtin_neon_vscale_f32:
3101:   case NEON::BI__builtin_neon_vscaleq_f32:
3102:   case NEON::BI__builtin_neon_vscaleq_f64:
3103:     cgm.errorNYI(expr->getSourceRange(),
3104:                  std::string("unimplemented AArch64 builtin call: ") +
3105:                      getContext().BuiltinInfo.getName(builtinID));
3106:     return mlir::Value{};
3107:   }
3108: 
3109:   // Unreachable: All cases in the switch above return.
3110: }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

## Key Concepts / 关键概念

- **Clang AST integration / Clang AST 集成**: Reads semantic information from Clang AST data structures. 从 Clang AST 数据结构读取语义信息。
- **Declaration handling / 声明处理**: Works with declaration nodes that describe source-level entities. 处理描述源级实体的声明节点。
- **MLIR/CIR integration / MLIR/CIR 集成**: Bridges Clang logic to MLIR/CIR operations, attributes, or types. 将 Clang 逻辑连接到 MLIR/CIR 的操作、属性或类型。
- **CIR dialect usage / CIR 方言使用**: Manipulates CIR-specific types, attributes, and operations. 操作 CIR 专用的类型、属性和操作。
- **CIR generation pipeline / CIR 生成流水线**: Participates in the lowering pipeline from Clang semantics to CIR constructs. 参与从 Clang 语义到 CIR 构造的 lowering 流水线。
- **`types` / `types`**: `types` is a prominent symbol in this file and helps define its structure or behavior. `types` 是该文件中的关键符号之一，用来界定其结构或行为。

## Dependencies / 依赖关系

- **Clang / Clang**: `clang/Basic/AArch64CodeGenUtils.h`, `clang/Basic/TargetBuiltins.h`, `clang/CIR/Dialect/IR/CIRTypes.h`, `clang/CIR/MissingFeatures.h`, `clang/AST/GlobalDecl.h`, `clang/Basic/Builtins.h`, `clang/Basic/arm_sve_builtin_cg.inc`, `clang/Basic/arm_fp16.inc`, `clang/Basic/arm_neon.inc`
- **LLVM / LLVM**: `llvm/IR/Intrinsics.h`, `llvm/IR/IntrinsicsAArch64.h`
- **MLIR / MLIR**: `mlir/IR/BuiltinTypes.h`, `mlir/IR/Value.h`
- **StdLib/Other / 标准库/其他**: `CIRGenBuilder.h`, `CIRGenFunction.h`
