# CIRGenBuiltinAMDGPU.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `clang/lib/CIR/CodeGen/CIRGenBuiltinAMDGPU.cpp`
- **Repository**: `/root/xw/llvm-project` (`llvm-project`)
- **Purpose (EN)**: This contains code to emit AMDGPU Builtin calls.
- **Purpose (CN)**: 实现与 `CIRGenBuiltinAMDGPU` 相关的 CIR 代码生成支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: //===---- CIRGenBuiltinAMDGPU.cpp - Emit CIR for AMDGPU builtins ----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This contains code to emit AMDGPU Builtin calls.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CIRGenFunction.h"
  14: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `CIRGenFunction.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `CIRGenFunction.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 15-18
```cpp
  15: #include "mlir/IR/Value.h"
  16: #include "clang/Basic/TargetBuiltins.h"
  17: #include "llvm/Support/ErrorHandling.h"
  18: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `Value.h`, `TargetBuiltins.h`, `ErrorHandling.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `Value.h`, `TargetBuiltins.h`, `ErrorHandling.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 19-22
```cpp
  19: using namespace clang;
  20: using namespace clang::CIRGen;
  21: using namespace cir;
  22: 
```
- **EN**: This block organizes symbols with namespaces or using-declarations for shorter references.
- **CN**: 该代码块通过命名空间或 using 声明组织符号并简化后续引用。

### Lines 23-33
```cpp
  23: static mlir::Value emitBinaryExpMaybeConstrainedFPBuiltin(
  24:     CIRGenFunction &cgf, const CallExpr *e, llvm::StringRef intrinsicName,
  25:     llvm::StringRef constrainedIntrinsicName) {
  26:   mlir::Value src0 = cgf.emitScalarExpr(e->getArg(0));
  27:   mlir::Value src1 = cgf.emitScalarExpr(e->getArg(1));
  28:   mlir::Location loc = cgf.getLoc(e->getExprLoc());
  29: 
  30:   CIRGenBuilderTy &builder = cgf.getBuilder();
  31: 
  32:   CIRGenFunction::CIRGenFPOptionsRAII fpOptsRAII(cgf, e);
  33: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitBinaryExpMaybeConstrainedFPBuiltin`, `fpOptsRAII`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitBinaryExpMaybeConstrainedFPBuiltin`、`fpOptsRAII`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 34-38
```cpp
  34:   if (builder.getIsFPConstrained()) {
  35:     cgf.cgm.errorNYI(e->getSourceRange(),
  36:                      "constrained FP intrinsic support is NYI.");
  37:   }
  38: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 39-42
```cpp
  39:   return builder.emitIntrinsicCallOp(loc, intrinsicName, src0.getType(),
  40:                                      mlir::ValueRange{src0, src1});
  41: }
  42: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 43-47
```cpp
  43: static mlir::Value emitLogbBuiltin(CIRGenFunction &cgf, const CallExpr *e,
  44:                                    const llvm::fltSemantics &fSem) {
  45:   CIRGenBuilderTy &builder = cgf.getBuilder();
  46:   mlir::Location loc = cgf.getLoc(e->getExprLoc());
  47: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitLogbBuiltin`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitLogbBuiltin`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 48-60
```cpp
  48:   mlir::Value src0 = cgf.emitScalarExpr(e->getArg(0));
  49:   mlir::Type srcTy = src0.getType();
  50:   mlir::Type int32Ty = builder.getSInt32Ty();
  51: 
  52:   cir::RecordType frExpResTy =
  53:       builder.getAnonRecordTy({srcTy, int32Ty}, false, false);
  54: 
  55:   mlir::Value frExpResult = builder.emitIntrinsicCallOp(
  56:       loc, "frexp", frExpResTy, mlir::ValueRange{src0});
  57: 
  58:   mlir::Value exp =
  59:       cir::ExtractMemberOp::create(builder, loc, int32Ty, frExpResult, 1);
  60: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::ExtractMemberOp::create`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::ExtractMemberOp::create`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 61-77
```cpp
  61:   mlir::Value negativeOne =
  62:       builder.getConstant(loc, cir::IntAttr::get(int32Ty, -1));
  63:   mlir::Value expMinus1 = builder.createAdd(loc, exp, negativeOne);
  64: 
  65:   mlir::Value siToFp = cir::CastOp::create(
  66:       builder, loc, srcTy, cir::CastKind::int_to_float, expMinus1);
  67: 
  68:   mlir::Value fabs = cir::FAbsOp::create(builder, loc, srcTy, src0);
  69: 
  70:   llvm::APFloat infVal = llvm::APFloat::getInf(fSem);
  71:   mlir::Value inf = builder.getConstant(loc, cir::FPAttr::get(srcTy, infVal));
  72: 
  73:   mlir::Value fabsNegInf =
  74:       builder.createCompare(loc, cir::CmpOpKind::ne, fabs, inf);
  75: 
  76:   mlir::Value sel = builder.createSelect(loc, fabsNegInf, siToFp, fabs);
  77: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 78-84
```cpp
  78:   llvm::APFloat zeroValue = llvm::APFloat::getZero(fSem);
  79:   mlir::Value zero =
  80:       builder.getConstant(loc, cir::FPAttr::get(srcTy, zeroValue));
  81: 
  82:   mlir::Value srcEqZero =
  83:       builder.createCompare(loc, cir::CmpOpKind::eq, src0, zero);
  84: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 85-93
```cpp
  85:   llvm::APFloat negInfVal = llvm::APFloat::getInf(fSem, true);
  86:   mlir::Value negInf =
  87:       builder.getConstant(loc, cir::FPAttr::get(srcTy, negInfVal));
  88: 
  89:   mlir::Value res = builder.createSelect(loc, srcEqZero, negInf, sel);
  90: 
  91:   return res;
  92: }
  93: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 94-111
```cpp
  94: std::optional<mlir::Value>
  95: CIRGenFunction::emitAMDGPUBuiltinExpr(unsigned builtinId,
  96:                                       const CallExpr *expr) {
  97:   switch (builtinId) {
  98:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_add_u32:
  99:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_sub_u32:
 100:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_min_i32:
 101:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_min_u32:
 102:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_max_i32:
 103:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_max_u32:
 104:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_and_b32:
 105:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_or_b32:
 106:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_xor_b32:
 107:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_add_u64:
 108:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_sub_u64:
 109:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_min_i64:
 110:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_min_u64:
 111:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_max_i64:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenFunction::emitAMDGPUBuiltinExpr`. A switch statement is used to dispatch behavior across enumerated cases or kinds. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenFunction::emitAMDGPUBuiltinExpr`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 112-128
```cpp
 112:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_max_u64:
 113:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_and_b64:
 114:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_or_b64:
 115:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_xor_b64: {
 116:     cgm.errorNYI(expr->getSourceRange(),
 117:                  std::string("unimplemented AMDGPU builtin call: ") +
 118:                      getContext().BuiltinInfo.getName(builtinId));
 119:     return mlir::Value{};
 120:   }
 121:   case AMDGPU::BI__builtin_amdgcn_div_scale:
 122:   case AMDGPU::BI__builtin_amdgcn_div_scalef: {
 123:     Address flagOutPtr = emitPointerWithAlignment(expr->getArg(3));
 124:     llvm::StringRef intrinsicName = "amdgcn.div.scale";
 125:     mlir::Value x = emitScalarExpr(expr->getArg(0));
 126:     mlir::Value y = emitScalarExpr(expr->getArg(1));
 127:     mlir::Value z = emitScalarExpr(expr->getArg(2));
 128: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 129-132
```cpp
 129:     auto i1Ty = builder.getUIntNTy(1);
 130:     cir::RecordType resTy = builder.getAnonRecordTy(
 131:         {x.getType(), i1Ty}, /*packed=*/false, /*padded=*/false);
 132: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 133-138
```cpp
 133:     mlir::Value structResult =
 134:         cir::LLVMIntrinsicCallOp::create(builder, getLoc(expr->getExprLoc()),
 135:                                          builder.getStringAttr(intrinsicName),
 136:                                          resTy, {x, y, z})
 137:             .getResult();
 138: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 139-143
```cpp
 139:     mlir::Value result = cir::ExtractMemberOp::create(
 140:         builder, getLoc(expr->getExprLoc()), x.getType(), structResult, 0);
 141:     mlir::Value flag = cir::ExtractMemberOp::create(
 142:         builder, getLoc(expr->getExprLoc()), i1Ty, structResult, 1);
 143: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getLoc`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getLoc`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 144-161
```cpp
 144:     mlir::Type flagType = flagOutPtr.getElementType();
 145:     mlir::Value flagToStore =
 146:         cir::CastOp::create(builder, getLoc(expr->getExprLoc()), flagType,
 147:                             cir::CastKind::int_to_bool, flag);
 148:     builder.createStore(getLoc(expr->getExprLoc()), flagToStore, flagOutPtr);
 149:     return result;
 150:   }
 151:   case AMDGPU::BI__builtin_amdgcn_div_fmas:
 152:   case AMDGPU::BI__builtin_amdgcn_div_fmasf: {
 153:     mlir::Value src0 = emitScalarExpr(expr->getArg(0));
 154:     mlir::Value src1 = emitScalarExpr(expr->getArg(1));
 155:     mlir::Value src2 = emitScalarExpr(expr->getArg(2));
 156:     mlir::Value src3 = emitScalarExpr(expr->getArg(3));
 157:     mlir::Value result = cir::LLVMIntrinsicCallOp::create(
 158:                              builder, getLoc(expr->getExprLoc()),
 159:                              builder.getStringAttr("amdgcn.div.fmas"),
 160:                              src0.getType(), {src0, src1, src2, src3})
 161:                              .getResult();
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::CastOp::create`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::CastOp::create`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 162-179
```cpp
 162:     return result;
 163:   }
 164:   case AMDGPU::BI__builtin_amdgcn_ds_swizzle: {
 165:     mlir::Value src0 = emitScalarExpr(expr->getArg(0));
 166:     mlir::Value src1 = emitScalarExpr(expr->getArg(1));
 167:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()),
 168:                                        "amdgcn.ds.swizzle", src0.getType(),
 169:                                        mlir::ValueRange{src0, src1});
 170:   }
 171:   case AMDGPU::BI__builtin_amdgcn_mov_dpp8:
 172:   case AMDGPU::BI__builtin_amdgcn_mov_dpp:
 173:   case AMDGPU::BI__builtin_amdgcn_update_dpp: {
 174:     cgm.errorNYI(expr->getSourceRange(),
 175:                  std::string("unimplemented AMDGPU builtin call: ") +
 176:                      getContext().BuiltinInfo.getName(builtinId));
 177:     return mlir::Value{};
 178:   }
 179:   case AMDGPU::BI__builtin_amdgcn_permlane16:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 180-197
```cpp
 180:   case AMDGPU::BI__builtin_amdgcn_permlanex16:
 181:   case AMDGPU::BI__builtin_amdgcn_permlane64: {
 182:     cgm.errorNYI(expr->getSourceRange(),
 183:                  std::string("unimplemented AMDGPU builtin call: ") +
 184:                      getContext().BuiltinInfo.getName(builtinId));
 185:     return mlir::Value{};
 186:   }
 187:   case AMDGPU::BI__builtin_amdgcn_readlane: {
 188:     mlir::Value src0 = emitScalarExpr(expr->getArg(0));
 189:     mlir::Value src1 = emitScalarExpr(expr->getArg(1));
 190:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()),
 191:                                        "amdgcn.readlane", src0.getType(),
 192:                                        mlir::ValueRange{src0, src1});
 193:   }
 194:   case AMDGPU::BI__builtin_amdgcn_readfirstlane: {
 195:     mlir::Value src0 = emitScalarExpr(expr->getArg(0));
 196:     return builder.emitIntrinsicCallOp(getLoc(expr->getExprLoc()),
 197:                                        "amdgcn.readfirstlane", src0.getType(),
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 198-215
```cpp
 198:                                        mlir::ValueRange{src0});
 199:   }
 200:   case AMDGPU::BI__builtin_amdgcn_wave_shuffle: {
 201:     cgm.errorNYI(expr->getSourceRange(),
 202:                  std::string("unimplemented AMDGPU builtin call: ") +
 203:                      getContext().BuiltinInfo.getName(builtinId));
 204:     return mlir::Value{};
 205:   }
 206:   case AMDGPU::BI__builtin_amdgcn_div_fixup:
 207:   case AMDGPU::BI__builtin_amdgcn_div_fixupf:
 208:   case AMDGPU::BI__builtin_amdgcn_div_fixuph: {
 209:     cgm.errorNYI(expr->getSourceRange(),
 210:                  std::string("unimplemented AMDGPU builtin call: ") +
 211:                      getContext().BuiltinInfo.getName(builtinId));
 212:     return mlir::Value{};
 213:   }
 214:   case AMDGPU::BI__builtin_amdgcn_trig_preop:
 215:   case AMDGPU::BI__builtin_amdgcn_trig_preopf: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 216-233
```cpp
 216:     cgm.errorNYI(expr->getSourceRange(),
 217:                  std::string("unimplemented AMDGPU builtin call: ") +
 218:                      getContext().BuiltinInfo.getName(builtinId));
 219:     return mlir::Value{};
 220:   }
 221:   case AMDGPU::BI__builtin_amdgcn_rcp:
 222:   case AMDGPU::BI__builtin_amdgcn_rcpf:
 223:   case AMDGPU::BI__builtin_amdgcn_rcph:
 224:   case AMDGPU::BI__builtin_amdgcn_rcp_bf16: {
 225:     cgm.errorNYI(expr->getSourceRange(),
 226:                  std::string("unimplemented AMDGPU builtin call: ") +
 227:                      getContext().BuiltinInfo.getName(builtinId));
 228:     return mlir::Value{};
 229:   }
 230:   case AMDGPU::BI__builtin_amdgcn_sqrt:
 231:   case AMDGPU::BI__builtin_amdgcn_sqrtf:
 232:   case AMDGPU::BI__builtin_amdgcn_sqrth:
 233:   case AMDGPU::BI__builtin_amdgcn_sqrt_bf16: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 234-251
```cpp
 234:     cgm.errorNYI(expr->getSourceRange(),
 235:                  std::string("unimplemented AMDGPU builtin call: ") +
 236:                      getContext().BuiltinInfo.getName(builtinId));
 237:     return mlir::Value{};
 238:   }
 239:   case AMDGPU::BI__builtin_amdgcn_rsq:
 240:   case AMDGPU::BI__builtin_amdgcn_rsqf:
 241:   case AMDGPU::BI__builtin_amdgcn_rsqh:
 242:   case AMDGPU::BI__builtin_amdgcn_rsq_bf16: {
 243:     cgm.errorNYI(expr->getSourceRange(),
 244:                  std::string("unimplemented AMDGPU builtin call: ") +
 245:                      getContext().BuiltinInfo.getName(builtinId));
 246:     return mlir::Value{};
 247:   }
 248:   case AMDGPU::BI__builtin_amdgcn_rsq_clamp:
 249:   case AMDGPU::BI__builtin_amdgcn_rsq_clampf: {
 250:     cgm.errorNYI(expr->getSourceRange(),
 251:                  std::string("unimplemented AMDGPU builtin call: ") +
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 252-269
```cpp
 252:                      getContext().BuiltinInfo.getName(builtinId));
 253:     return mlir::Value{};
 254:   }
 255:   case AMDGPU::BI__builtin_amdgcn_sinf:
 256:   case AMDGPU::BI__builtin_amdgcn_sinh:
 257:   case AMDGPU::BI__builtin_amdgcn_sin_bf16: {
 258:     cgm.errorNYI(expr->getSourceRange(),
 259:                  std::string("unimplemented AMDGPU builtin call: ") +
 260:                      getContext().BuiltinInfo.getName(builtinId));
 261:     return mlir::Value{};
 262:   }
 263:   case AMDGPU::BI__builtin_amdgcn_cosf:
 264:   case AMDGPU::BI__builtin_amdgcn_cosh:
 265:   case AMDGPU::BI__builtin_amdgcn_cos_bf16: {
 266:     cgm.errorNYI(expr->getSourceRange(),
 267:                  std::string("unimplemented AMDGPU builtin call: ") +
 268:                      getContext().BuiltinInfo.getName(builtinId));
 269:     return mlir::Value{};
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getContext`, `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getContext`、`std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 270-287
```cpp
 270:   }
 271:   case AMDGPU::BI__builtin_amdgcn_dispatch_ptr: {
 272:     cgm.errorNYI(expr->getSourceRange(),
 273:                  std::string("unimplemented AMDGPU builtin call: ") +
 274:                      getContext().BuiltinInfo.getName(builtinId));
 275:     return mlir::Value{};
 276:   }
 277:   case AMDGPU::BI__builtin_amdgcn_logf:
 278:   case AMDGPU::BI__builtin_amdgcn_log_bf16: {
 279:     cgm.errorNYI(expr->getSourceRange(),
 280:                  std::string("unimplemented AMDGPU builtin call: ") +
 281:                      getContext().BuiltinInfo.getName(builtinId));
 282:     return mlir::Value{};
 283:   }
 284:   case AMDGPU::BI__builtin_amdgcn_exp2f:
 285:   case AMDGPU::BI__builtin_amdgcn_exp2_bf16: {
 286:     cgm.errorNYI(expr->getSourceRange(),
 287:                  std::string("unimplemented AMDGPU builtin call: ") +
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 288-305
```cpp
 288:                      getContext().BuiltinInfo.getName(builtinId));
 289:     return mlir::Value{};
 290:   }
 291:   case AMDGPU::BI__builtin_amdgcn_log_clampf: {
 292:     cgm.errorNYI(expr->getSourceRange(),
 293:                  std::string("unimplemented AMDGPU builtin call: ") +
 294:                      getContext().BuiltinInfo.getName(builtinId));
 295:     return mlir::Value{};
 296:   }
 297:   case AMDGPU::BI__builtin_amdgcn_ldexp:
 298:   case AMDGPU::BI__builtin_amdgcn_ldexpf:
 299:   case AMDGPU::BI__builtin_amdgcn_ldexph: {
 300:     cgm.errorNYI(expr->getSourceRange(),
 301:                  std::string("unimplemented AMDGPU builtin call: ") +
 302:                      getContext().BuiltinInfo.getName(builtinId));
 303:     return mlir::Value{};
 304:   }
 305:   case AMDGPU::BI__builtin_amdgcn_frexp_mant:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getContext`, `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getContext`、`std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 306-323
```cpp
 306:   case AMDGPU::BI__builtin_amdgcn_frexp_mantf:
 307:   case AMDGPU::BI__builtin_amdgcn_frexp_manth: {
 308:     cgm.errorNYI(expr->getSourceRange(),
 309:                  std::string("unimplemented AMDGPU builtin call: ") +
 310:                      getContext().BuiltinInfo.getName(builtinId));
 311:     return mlir::Value{};
 312:   }
 313:   case AMDGPU::BI__builtin_amdgcn_frexp_exp:
 314:   case AMDGPU::BI__builtin_amdgcn_frexp_expf:
 315:   case AMDGPU::BI__builtin_amdgcn_frexp_exph: {
 316:     cgm.errorNYI(expr->getSourceRange(),
 317:                  std::string("unimplemented AMDGPU builtin call: ") +
 318:                      getContext().BuiltinInfo.getName(builtinId));
 319:     return mlir::Value{};
 320:   }
 321:   case AMDGPU::BI__builtin_amdgcn_fract:
 322:   case AMDGPU::BI__builtin_amdgcn_fractf:
 323:   case AMDGPU::BI__builtin_amdgcn_fracth: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 324-341
```cpp
 324:     cgm.errorNYI(expr->getSourceRange(),
 325:                  std::string("unimplemented AMDGPU builtin call: ") +
 326:                      getContext().BuiltinInfo.getName(builtinId));
 327:     return mlir::Value{};
 328:   }
 329:   case AMDGPU::BI__builtin_amdgcn_lerp: {
 330:     cgm.errorNYI(expr->getSourceRange(),
 331:                  std::string("unimplemented AMDGPU builtin call: ") +
 332:                      getContext().BuiltinInfo.getName(builtinId));
 333:     return mlir::Value{};
 334:   }
 335:   case AMDGPU::BI__builtin_amdgcn_ubfe: {
 336:     cgm.errorNYI(expr->getSourceRange(),
 337:                  std::string("unimplemented AMDGPU builtin call: ") +
 338:                      getContext().BuiltinInfo.getName(builtinId));
 339:     return mlir::Value{};
 340:   }
 341:   case AMDGPU::BI__builtin_amdgcn_sbfe: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 342-359
```cpp
 342:     cgm.errorNYI(expr->getSourceRange(),
 343:                  std::string("unimplemented AMDGPU builtin call: ") +
 344:                      getContext().BuiltinInfo.getName(builtinId));
 345:     return mlir::Value{};
 346:   }
 347:   case AMDGPU::BI__builtin_amdgcn_ballot_w32:
 348:   case AMDGPU::BI__builtin_amdgcn_ballot_w64: {
 349:     cgm.errorNYI(expr->getSourceRange(),
 350:                  std::string("unimplemented AMDGPU builtin call: ") +
 351:                      getContext().BuiltinInfo.getName(builtinId));
 352:     return mlir::Value{};
 353:   }
 354:   case AMDGPU::BI__builtin_amdgcn_inverse_ballot_w32:
 355:   case AMDGPU::BI__builtin_amdgcn_inverse_ballot_w64: {
 356:     cgm.errorNYI(expr->getSourceRange(),
 357:                  std::string("unimplemented AMDGPU builtin call: ") +
 358:                      getContext().BuiltinInfo.getName(builtinId));
 359:     return mlir::Value{};
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 360-377
```cpp
 360:   }
 361:   case AMDGPU::BI__builtin_amdgcn_tanhf:
 362:   case AMDGPU::BI__builtin_amdgcn_tanhh:
 363:   case AMDGPU::BI__builtin_amdgcn_tanh_bf16: {
 364:     cgm.errorNYI(expr->getSourceRange(),
 365:                  std::string("unimplemented AMDGPU builtin call: ") +
 366:                      getContext().BuiltinInfo.getName(builtinId));
 367:     return mlir::Value{};
 368:   }
 369:   case AMDGPU::BI__builtin_amdgcn_uicmp:
 370:   case AMDGPU::BI__builtin_amdgcn_uicmpl:
 371:   case AMDGPU::BI__builtin_amdgcn_sicmp:
 372:   case AMDGPU::BI__builtin_amdgcn_sicmpl: {
 373:     cgm.errorNYI(expr->getSourceRange(),
 374:                  std::string("unimplemented AMDGPU builtin call: ") +
 375:                      getContext().BuiltinInfo.getName(builtinId));
 376:     return mlir::Value{};
 377:   }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 378-395
```cpp
 378:   case AMDGPU::BI__builtin_amdgcn_fcmp:
 379:   case AMDGPU::BI__builtin_amdgcn_fcmpf: {
 380:     cgm.errorNYI(expr->getSourceRange(),
 381:                  std::string("unimplemented AMDGPU builtin call: ") +
 382:                      getContext().BuiltinInfo.getName(builtinId));
 383:     return mlir::Value{};
 384:   }
 385:   case AMDGPU::BI__builtin_amdgcn_class:
 386:   case AMDGPU::BI__builtin_amdgcn_classf:
 387:   case AMDGPU::BI__builtin_amdgcn_classh: {
 388:     cgm.errorNYI(expr->getSourceRange(),
 389:                  std::string("unimplemented AMDGPU builtin call: ") +
 390:                      getContext().BuiltinInfo.getName(builtinId));
 391:     return mlir::Value{};
 392:   }
 393:   case AMDGPU::BI__builtin_amdgcn_fmed3f:
 394:   case AMDGPU::BI__builtin_amdgcn_fmed3h: {
 395:     cgm.errorNYI(expr->getSourceRange(),
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 396-413
```cpp
 396:                  std::string("unimplemented AMDGPU builtin call: ") +
 397:                      getContext().BuiltinInfo.getName(builtinId));
 398:     return mlir::Value{};
 399:   }
 400:   case AMDGPU::BI__builtin_amdgcn_ds_append:
 401:   case AMDGPU::BI__builtin_amdgcn_ds_consume: {
 402:     cgm.errorNYI(expr->getSourceRange(),
 403:                  std::string("unimplemented AMDGPU builtin call: ") +
 404:                      getContext().BuiltinInfo.getName(builtinId));
 405:     return mlir::Value{};
 406:   }
 407:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b64_i32:
 408:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b64_v2i32:
 409:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v4i16:
 410:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v4f16:
 411:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v4bf16:
 412:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v8i16:
 413:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v8f16:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 414-431
```cpp
 414:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v8bf16:
 415:   case AMDGPU::BI__builtin_amdgcn_global_load_tr4_b64_v2i32:
 416:   case AMDGPU::BI__builtin_amdgcn_global_load_tr8_b64_v2i32:
 417:   case AMDGPU::BI__builtin_amdgcn_global_load_tr6_b96_v3i32:
 418:   case AMDGPU::BI__builtin_amdgcn_global_load_tr16_b128_v8i16:
 419:   case AMDGPU::BI__builtin_amdgcn_global_load_tr16_b128_v8f16:
 420:   case AMDGPU::BI__builtin_amdgcn_global_load_tr16_b128_v8bf16: {
 421:     cgm.errorNYI(expr->getSourceRange(),
 422:                  std::string("unimplemented AMDGPU builtin call: ") +
 423:                      getContext().BuiltinInfo.getName(builtinId));
 424:     return mlir::Value{};
 425:   }
 426:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr4_b64_v2i32:
 427:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr8_b64_v2i32:
 428:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr6_b96_v3i32:
 429:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr16_b128_v8i16:
 430:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr16_b128_v8f16:
 431:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr16_b128_v8bf16: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 432-449
```cpp
 432:     cgm.errorNYI(expr->getSourceRange(),
 433:                  std::string("unimplemented AMDGPU builtin call: ") +
 434:                      getContext().BuiltinInfo.getName(builtinId));
 435:     return mlir::Value{};
 436:   }
 437:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr4_b64_v2i32:
 438:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr8_b64_v2i32:
 439:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr6_b96_v3i32:
 440:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr16_b64_v4f16:
 441:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr16_b64_v4bf16:
 442:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr16_b64_v4i16: {
 443:     cgm.errorNYI(expr->getSourceRange(),
 444:                  std::string("unimplemented AMDGPU builtin call: ") +
 445:                      getContext().BuiltinInfo.getName(builtinId));
 446:     return mlir::Value{};
 447:   }
 448:   case AMDGPU::BI__builtin_amdgcn_global_load_monitor_b32:
 449:   case AMDGPU::BI__builtin_amdgcn_global_load_monitor_b64:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 450-467
```cpp
 450:   case AMDGPU::BI__builtin_amdgcn_global_load_monitor_b128:
 451:   case AMDGPU::BI__builtin_amdgcn_flat_load_monitor_b32:
 452:   case AMDGPU::BI__builtin_amdgcn_flat_load_monitor_b64:
 453:   case AMDGPU::BI__builtin_amdgcn_flat_load_monitor_b128: {
 454:     cgm.errorNYI(expr->getSourceRange(),
 455:                  std::string("unimplemented AMDGPU builtin call: ") +
 456:                      getContext().BuiltinInfo.getName(builtinId));
 457:     return mlir::Value{};
 458:   }
 459:   case AMDGPU::BI__builtin_amdgcn_cluster_load_b32:
 460:   case AMDGPU::BI__builtin_amdgcn_cluster_load_b64:
 461:   case AMDGPU::BI__builtin_amdgcn_cluster_load_b128: {
 462:     cgm.errorNYI(expr->getSourceRange(),
 463:                  std::string("unimplemented AMDGPU builtin call: ") +
 464:                      getContext().BuiltinInfo.getName(builtinId));
 465:     return mlir::Value{};
 466:   }
 467:   case AMDGPU::BI__builtin_amdgcn_load_to_lds: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 468-485
```cpp
 468:     cgm.errorNYI(expr->getSourceRange(),
 469:                  std::string("unimplemented AMDGPU builtin call: ") +
 470:                      getContext().BuiltinInfo.getName(builtinId));
 471:     return mlir::Value{};
 472:   }
 473:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_load_32x4B:
 474:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_store_32x4B:
 475:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_load_16x8B:
 476:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_store_16x8B:
 477:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_load_8x16B:
 478:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_store_8x16B: {
 479:     cgm.errorNYI(expr->getSourceRange(),
 480:                  std::string("unimplemented AMDGPU builtin call: ") +
 481:                      getContext().BuiltinInfo.getName(builtinId));
 482:     return mlir::Value{};
 483:   }
 484:   case AMDGPU::BI__builtin_amdgcn_get_fpenv:
 485:   case AMDGPU::BI__builtin_amdgcn_set_fpenv: {
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 486-503
```cpp
 486:     cgm.errorNYI(expr->getSourceRange(),
 487:                  std::string("unimplemented AMDGPU builtin call: ") +
 488:                      getContext().BuiltinInfo.getName(builtinId));
 489:     return mlir::Value{};
 490:   }
 491:   case AMDGPU::BI__builtin_amdgcn_read_exec:
 492:   case AMDGPU::BI__builtin_amdgcn_read_exec_lo:
 493:   case AMDGPU::BI__builtin_amdgcn_read_exec_hi: {
 494:     cgm.errorNYI(expr->getSourceRange(),
 495:                  std::string("unimplemented AMDGPU builtin call: ") +
 496:                      getContext().BuiltinInfo.getName(builtinId));
 497:     return mlir::Value{};
 498:   }
 499:   case AMDGPU::BI__builtin_amdgcn_image_bvh_intersect_ray:
 500:   case AMDGPU::BI__builtin_amdgcn_image_bvh_intersect_ray_h:
 501:   case AMDGPU::BI__builtin_amdgcn_image_bvh_intersect_ray_l:
 502:   case AMDGPU::BI__builtin_amdgcn_image_bvh_intersect_ray_lh: {
 503:     cgm.errorNYI(expr->getSourceRange(),
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 504-521
```cpp
 504:                  std::string("unimplemented AMDGPU builtin call: ") +
 505:                      getContext().BuiltinInfo.getName(builtinId));
 506:     return mlir::Value{};
 507:   }
 508:   case AMDGPU::BI__builtin_amdgcn_image_bvh8_intersect_ray:
 509:   case AMDGPU::BI__builtin_amdgcn_image_bvh_dual_intersect_ray: {
 510:     cgm.errorNYI(expr->getSourceRange(),
 511:                  std::string("unimplemented AMDGPU builtin call: ") +
 512:                      getContext().BuiltinInfo.getName(builtinId));
 513:     return mlir::Value{};
 514:   }
 515:   case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_rtn:
 516:   case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_push4_pop1_rtn:
 517:   case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_push8_pop1_rtn:
 518:   case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_push8_pop2_rtn: {
 519:     cgm.errorNYI(expr->getSourceRange(),
 520:                  std::string("unimplemented AMDGPU builtin call: ") +
 521:                      getContext().BuiltinInfo.getName(builtinId));
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 522-539
```cpp
 522:     return mlir::Value{};
 523:   }
 524:   case AMDGPU::BI__builtin_amdgcn_image_load_1d_v4f32_i32:
 525:   case AMDGPU::BI__builtin_amdgcn_image_load_1d_v4f16_i32:
 526:   case AMDGPU::BI__builtin_amdgcn_image_load_1darray_v4f32_i32:
 527:   case AMDGPU::BI__builtin_amdgcn_image_load_1darray_v4f16_i32:
 528:   case AMDGPU::BI__builtin_amdgcn_image_load_2d_f32_i32:
 529:   case AMDGPU::BI__builtin_amdgcn_image_load_2d_v4f32_i32:
 530:   case AMDGPU::BI__builtin_amdgcn_image_load_2d_v4f16_i32:
 531:   case AMDGPU::BI__builtin_amdgcn_image_load_2darray_f32_i32:
 532:   case AMDGPU::BI__builtin_amdgcn_image_load_2darray_v4f32_i32:
 533:   case AMDGPU::BI__builtin_amdgcn_image_load_2darray_v4f16_i32:
 534:   case AMDGPU::BI__builtin_amdgcn_image_load_3d_v4f32_i32:
 535:   case AMDGPU::BI__builtin_amdgcn_image_load_3d_v4f16_i32:
 536:   case AMDGPU::BI__builtin_amdgcn_image_load_cube_v4f32_i32:
 537:   case AMDGPU::BI__builtin_amdgcn_image_load_cube_v4f16_i32:
 538:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_1d_v4f32_i32:
 539:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_1d_v4f16_i32:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 540-557
```cpp
 540:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2d_v4f32_i32:
 541:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2d_v4f16_i32:
 542:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2darray_f32_i32:
 543:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2darray_v4f32_i32:
 544:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2darray_v4f16_i32:
 545:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_3d_v4f32_i32:
 546:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_3d_v4f16_i32:
 547:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_cube_v4f32_i32:
 548:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_cube_v4f16_i32: {
 549:     cgm.errorNYI(expr->getSourceRange(),
 550:                  std::string("unimplemented AMDGPU builtin call: ") +
 551:                      getContext().BuiltinInfo.getName(builtinId));
 552:     return mlir::Value{};
 553:   }
 554:   case AMDGPU::BI__builtin_amdgcn_image_store_1d_v4f32_i32:
 555:   case AMDGPU::BI__builtin_amdgcn_image_store_1d_v4f16_i32:
 556:   case AMDGPU::BI__builtin_amdgcn_image_store_1darray_v4f32_i32:
 557:   case AMDGPU::BI__builtin_amdgcn_image_store_1darray_v4f16_i32:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 558-575
```cpp
 558:   case AMDGPU::BI__builtin_amdgcn_image_store_2d_f32_i32:
 559:   case AMDGPU::BI__builtin_amdgcn_image_store_2d_v4f32_i32:
 560:   case AMDGPU::BI__builtin_amdgcn_image_store_2d_v4f16_i32:
 561:   case AMDGPU::BI__builtin_amdgcn_image_store_2darray_f32_i32:
 562:   case AMDGPU::BI__builtin_amdgcn_image_store_2darray_v4f32_i32:
 563:   case AMDGPU::BI__builtin_amdgcn_image_store_2darray_v4f16_i32:
 564:   case AMDGPU::BI__builtin_amdgcn_image_store_3d_v4f32_i32:
 565:   case AMDGPU::BI__builtin_amdgcn_image_store_3d_v4f16_i32:
 566:   case AMDGPU::BI__builtin_amdgcn_image_store_cube_v4f32_i32:
 567:   case AMDGPU::BI__builtin_amdgcn_image_store_cube_v4f16_i32:
 568:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_1d_v4f32_i32:
 569:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_1d_v4f16_i32:
 570:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_1darray_v4f32_i32:
 571:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_1darray_v4f16_i32:
 572:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2d_f32_i32:
 573:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2d_v4f32_i32:
 574:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2d_v4f16_i32:
 575:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2darray_f32_i32:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 576-593
```cpp
 576:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2darray_v4f32_i32:
 577:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2darray_v4f16_i32:
 578:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_3d_v4f32_i32:
 579:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_3d_v4f16_i32:
 580:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_cube_v4f32_i32:
 581:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_cube_v4f16_i32: {
 582:     cgm.errorNYI(expr->getSourceRange(),
 583:                  std::string("unimplemented AMDGPU builtin call: ") +
 584:                      getContext().BuiltinInfo.getName(builtinId));
 585:     return mlir::Value{};
 586:   }
 587:   case AMDGPU::BI__builtin_amdgcn_image_sample_1d_v4f32_f32:
 588:   case AMDGPU::BI__builtin_amdgcn_image_sample_1d_v4f16_f32:
 589:   case AMDGPU::BI__builtin_amdgcn_image_sample_1darray_v4f32_f32:
 590:   case AMDGPU::BI__builtin_amdgcn_image_sample_1darray_v4f16_f32:
 591:   case AMDGPU::BI__builtin_amdgcn_image_sample_2d_f32_f32:
 592:   case AMDGPU::BI__builtin_amdgcn_image_sample_2d_v4f32_f32:
 593:   case AMDGPU::BI__builtin_amdgcn_image_sample_2d_v4f16_f32:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 594-611
```cpp
 594:   case AMDGPU::BI__builtin_amdgcn_image_sample_2darray_f32_f32:
 595:   case AMDGPU::BI__builtin_amdgcn_image_sample_2darray_v4f32_f32:
 596:   case AMDGPU::BI__builtin_amdgcn_image_sample_2darray_v4f16_f32:
 597:   case AMDGPU::BI__builtin_amdgcn_image_sample_3d_v4f32_f32:
 598:   case AMDGPU::BI__builtin_amdgcn_image_sample_3d_v4f16_f32:
 599:   case AMDGPU::BI__builtin_amdgcn_image_sample_cube_v4f32_f32:
 600:   case AMDGPU::BI__builtin_amdgcn_image_sample_cube_v4f16_f32:
 601:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_1d_v4f32_f32:
 602:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_1d_v4f16_f32:
 603:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_1d_v4f32_f32:
 604:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_1d_v4f16_f32:
 605:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_1d_v4f32_f32:
 606:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_1d_v4f16_f32:
 607:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_2d_v4f32_f32:
 608:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_2d_v4f16_f32:
 609:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_2d_f32_f32:
 610:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_2d_v4f32_f32:
 611:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_2d_v4f16_f32:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 612-629
```cpp
 612:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_2d_f32_f32:
 613:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_2d_v4f32_f32:
 614:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_2d_v4f16_f32:
 615:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_2d_f32_f32:
 616:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_3d_v4f32_f32:
 617:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_3d_v4f16_f32:
 618:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_3d_v4f32_f32:
 619:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_3d_v4f16_f32:
 620:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_3d_v4f32_f32:
 621:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_3d_v4f16_f32:
 622:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_cube_v4f32_f32:
 623:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_cube_v4f16_f32:
 624:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_cube_v4f32_f32:
 625:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_cube_v4f16_f32:
 626:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_1darray_v4f32_f32:
 627:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_1darray_v4f16_f32:
 628:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_1darray_v4f32_f32:
 629:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_1darray_v4f16_f32:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 630-647
```cpp
 630:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_1darray_v4f32_f32:
 631:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_1darray_v4f16_f32:
 632:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_2darray_v4f32_f32:
 633:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_2darray_v4f16_f32:
 634:   case AMDGPU::BI__builtin_amdgcn_image_sample_lz_2darray_f32_f32:
 635:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_2darray_v4f32_f32:
 636:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_2darray_v4f16_f32:
 637:   case AMDGPU::BI__builtin_amdgcn_image_sample_l_2darray_f32_f32:
 638:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_2darray_v4f32_f32:
 639:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_2darray_v4f16_f32:
 640:   case AMDGPU::BI__builtin_amdgcn_image_sample_d_2darray_f32_f32: {
 641:     cgm.errorNYI(expr->getSourceRange(),
 642:                  std::string("unimplemented AMDGPU builtin call: ") +
 643:                      getContext().BuiltinInfo.getName(builtinId));
 644:     return mlir::Value{};
 645:   }
 646:   case AMDGPU::BI__builtin_amdgcn_image_gather4_lz_2d_v4f32_f32: {
 647:     cgm.errorNYI(expr->getSourceRange(),
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 648-665
```cpp
 648:                  std::string("unimplemented AMDGPU builtin call: ") +
 649:                      getContext().BuiltinInfo.getName(builtinId));
 650:     return mlir::Value{};
 651:   }
 652:   case AMDGPU::BI__builtin_amdgcn_mfma_scale_f32_16x16x128_f8f6f4:
 653:   case AMDGPU::BI__builtin_amdgcn_mfma_scale_f32_32x32x64_f8f6f4: {
 654:     cgm.errorNYI(expr->getSourceRange(),
 655:                  std::string("unimplemented AMDGPU builtin call: ") +
 656:                      getContext().BuiltinInfo.getName(builtinId));
 657:     return mlir::Value{};
 658:   }
 659:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32:
 660:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w32:
 661:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64:
 662:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w64:
 663:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w32:
 664:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w32:
 665:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w64:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 666-683
```cpp
 666:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w64:
 667:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32:
 668:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w64:
 669:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w32:
 670:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w64:
 671:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32:
 672:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w64:
 673:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32:
 674:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w64:
 675:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32_gfx12:
 676:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64_gfx12:
 677:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w32_gfx12:
 678:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w64_gfx12:
 679:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32_gfx12:
 680:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w64_gfx12:
 681:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w32_gfx12:
 682:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w64_gfx12:
 683:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32_gfx12:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 684-701
```cpp
 684:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w64_gfx12:
 685:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32_gfx12:
 686:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w64_gfx12:
 687:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w32_gfx12:
 688:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w64_gfx12:
 689:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w32_gfx12:
 690:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w64_gfx12:
 691:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w32_gfx12:
 692:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w64_gfx12:
 693:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w32_gfx12:
 694:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w64_gfx12:
 695:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x32_iu4_w32_gfx12:
 696:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x32_iu4_w64_gfx12: {
 697:     cgm.errorNYI(expr->getSourceRange(),
 698:                  std::string("unimplemented AMDGPU builtin call: ") +
 699:                      getContext().BuiltinInfo.getName(builtinId));
 700:     return mlir::Value{};
 701:   }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 702-719
```cpp
 702:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_f16_w32:
 703:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_f16_w64:
 704:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf16_w32:
 705:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf16_w64:
 706:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x32_f16_w32:
 707:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x32_f16_w64:
 708:   case AMDGPU::BI__builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w32:
 709:   case AMDGPU::BI__builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w64:
 710:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu8_w32:
 711:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu8_w64:
 712:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu4_w32:
 713:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu4_w64:
 714:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x64_iu4_w32:
 715:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x64_iu4_w64:
 716:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w32:
 717:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w64:
 718:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w32:
 719:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w64:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 720-737
```cpp
 720:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w32:
 721:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w64:
 722:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w32:
 723:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w64: {
 724:     cgm.errorNYI(expr->getSourceRange(),
 725:                  std::string("unimplemented AMDGPU builtin call: ") +
 726:                      getContext().BuiltinInfo.getName(builtinId));
 727:     return mlir::Value{};
 728:   }
 729:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x4_f32:
 730:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x32_bf16:
 731:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x32_f16:
 732:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x32_f16:
 733:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x32_bf16:
 734:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16f32_16x16x32_bf16:
 735:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_fp8_fp8:
 736:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_fp8_bf8:
 737:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_bf8_fp8:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 738-755
```cpp
 738:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_bf8_bf8:
 739:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_fp8_fp8:
 740:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_fp8_bf8:
 741:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_bf8_fp8:
 742:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_bf8_bf8:
 743:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_fp8_fp8:
 744:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_fp8_bf8:
 745:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_bf8_fp8:
 746:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_bf8_bf8:
 747:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_fp8_fp8:
 748:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_fp8_bf8:
 749:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_bf8_fp8:
 750:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_bf8_bf8:
 751:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x64_iu8:
 752:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_f8f6f4:
 753:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_32x16x128_f4:
 754:   case AMDGPU::BI__builtin_amdgcn_wmma_scale_f32_16x16x128_f8f6f4:
 755:   case AMDGPU::BI__builtin_amdgcn_wmma_scale16_f32_16x16x128_f8f6f4:
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 756-773
```cpp
 756:   case AMDGPU::BI__builtin_amdgcn_wmma_scale_f32_32x16x128_f4:
 757:   case AMDGPU::BI__builtin_amdgcn_wmma_scale16_f32_32x16x128_f4: {
 758:     cgm.errorNYI(expr->getSourceRange(),
 759:                  std::string("unimplemented AMDGPU builtin call: ") +
 760:                      getContext().BuiltinInfo.getName(builtinId));
 761:     return mlir::Value{};
 762:   }
 763:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x64_f16:
 764:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x64_bf16:
 765:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x64_f16:
 766:   case AMDGPU::BI__builtin_amdgcn_swmmac_bf16_16x16x64_bf16:
 767:   case AMDGPU::BI__builtin_amdgcn_swmmac_bf16f32_16x16x64_bf16:
 768:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_fp8_fp8:
 769:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_fp8_bf8:
 770:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_bf8_fp8:
 771:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_bf8_bf8:
 772:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_fp8_fp8:
 773:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_fp8_bf8:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 774-791
```cpp
 774:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_bf8_fp8:
 775:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_bf8_bf8:
 776:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x128_iu8: {
 777:     cgm.errorNYI(expr->getSourceRange(),
 778:                  std::string("unimplemented AMDGPU builtin call: ") +
 779:                      getContext().BuiltinInfo.getName(builtinId));
 780:     return mlir::Value{};
 781:   }
 782:   // amdgcn workgroup size
 783:   case AMDGPU::BI__builtin_amdgcn_workgroup_size_x:
 784:   case AMDGPU::BI__builtin_amdgcn_workgroup_size_y:
 785:   case AMDGPU::BI__builtin_amdgcn_workgroup_size_z: {
 786:     cgm.errorNYI(expr->getSourceRange(),
 787:                  std::string("unimplemented AMDGPU builtin call: ") +
 788:                      getContext().BuiltinInfo.getName(builtinId));
 789:     return mlir::Value{};
 790:   }
 791:   case AMDGPU::BI__builtin_amdgcn_grid_size_x:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 792-809
```cpp
 792:   case AMDGPU::BI__builtin_amdgcn_grid_size_y:
 793:   case AMDGPU::BI__builtin_amdgcn_grid_size_z: {
 794:     cgm.errorNYI(expr->getSourceRange(),
 795:                  std::string("unimplemented AMDGPU builtin call: ") +
 796:                      getContext().BuiltinInfo.getName(builtinId));
 797:     return mlir::Value{};
 798:   }
 799:   case AMDGPU::BI__builtin_r600_recipsqrt_ieee:
 800:   case AMDGPU::BI__builtin_r600_recipsqrt_ieeef: {
 801:     cgm.errorNYI(expr->getSourceRange(),
 802:                  std::string("unimplemented AMDGPU builtin call: ") +
 803:                      getContext().BuiltinInfo.getName(builtinId));
 804:     return mlir::Value{};
 805:   }
 806:   case AMDGPU::BI__builtin_amdgcn_alignbit: {
 807:     cgm.errorNYI(expr->getSourceRange(),
 808:                  std::string("unimplemented AMDGPU builtin call: ") +
 809:                      getContext().BuiltinInfo.getName(builtinId));
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 810-827
```cpp
 810:     return mlir::Value{};
 811:   }
 812:   case AMDGPU::BI__builtin_amdgcn_fence: {
 813:     cgm.errorNYI(expr->getSourceRange(),
 814:                  std::string("unimplemented AMDGPU builtin call: ") +
 815:                      getContext().BuiltinInfo.getName(builtinId));
 816:     return mlir::Value{};
 817:   }
 818:   case AMDGPU::BI__builtin_amdgcn_atomic_inc32:
 819:   case AMDGPU::BI__builtin_amdgcn_atomic_inc64:
 820:   case AMDGPU::BI__builtin_amdgcn_atomic_dec32:
 821:   case AMDGPU::BI__builtin_amdgcn_atomic_dec64:
 822:   case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_f64:
 823:   case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_f32:
 824:   case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_v2f16:
 825:   case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_v2bf16:
 826:   case AMDGPU::BI__builtin_amdgcn_ds_faddf:
 827:   case AMDGPU::BI__builtin_amdgcn_ds_fminf:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 828-845
```cpp
 828:   case AMDGPU::BI__builtin_amdgcn_ds_fmaxf:
 829:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_f32:
 830:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_f64:
 831:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_v2f16:
 832:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_v2f16:
 833:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_f32:
 834:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_f64:
 835:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_v2bf16:
 836:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_v2bf16:
 837:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fmin_f64:
 838:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fmax_f64:
 839:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fmin_f64:
 840:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fmax_f64: {
 841:     cgm.errorNYI(expr->getSourceRange(),
 842:                  std::string("unimplemented AMDGPU builtin call: ") +
 843:                      getContext().BuiltinInfo.getName(builtinId));
 844:     return mlir::Value{};
 845:   }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 846-863
```cpp
 846:   case AMDGPU::BI__builtin_amdgcn_s_sendmsg_rtn:
 847:   case AMDGPU::BI__builtin_amdgcn_s_sendmsg_rtnl: {
 848:     cgm.errorNYI(expr->getSourceRange(),
 849:                  std::string("unimplemented AMDGPU builtin call: ") +
 850:                      getContext().BuiltinInfo.getName(builtinId));
 851:     return mlir::Value{};
 852:   }
 853:   case AMDGPU::BI__builtin_amdgcn_permlane16_swap:
 854:   case AMDGPU::BI__builtin_amdgcn_permlane32_swap: {
 855:     cgm.errorNYI(expr->getSourceRange(),
 856:                  std::string("unimplemented AMDGPU builtin call: ") +
 857:                      getContext().BuiltinInfo.getName(builtinId));
 858:     return mlir::Value{};
 859:   }
 860:   case AMDGPU::BI__builtin_amdgcn_bitop3_b32:
 861:   case AMDGPU::BI__builtin_amdgcn_bitop3_b16: {
 862:     cgm.errorNYI(expr->getSourceRange(),
 863:                  std::string("unimplemented AMDGPU builtin call: ") +
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 864-881
```cpp
 864:                      getContext().BuiltinInfo.getName(builtinId));
 865:     return mlir::Value{};
 866:   }
 867:   case AMDGPU::BI__builtin_amdgcn_make_buffer_rsrc: {
 868:     cgm.errorNYI(expr->getSourceRange(),
 869:                  std::string("unimplemented AMDGPU builtin call: ") +
 870:                      getContext().BuiltinInfo.getName(builtinId));
 871:     return mlir::Value{};
 872:   }
 873:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b8:
 874:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b16:
 875:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b32:
 876:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b64:
 877:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b96:
 878:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b128: {
 879:     cgm.errorNYI(expr->getSourceRange(),
 880:                  std::string("unimplemented AMDGPU builtin call: ") +
 881:                      getContext().BuiltinInfo.getName(builtinId));
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getContext`, `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getContext`、`std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 882-899
```cpp
 882:     return mlir::Value{};
 883:   }
 884:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b8:
 885:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b16:
 886:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b32:
 887:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b64:
 888:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b96:
 889:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b128: {
 890:     cgm.errorNYI(expr->getSourceRange(),
 891:                  std::string("unimplemented AMDGPU builtin call: ") +
 892:                      getContext().BuiltinInfo.getName(builtinId));
 893:     return mlir::Value{};
 894:   }
 895:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_add_i32: {
 896:     cgm.errorNYI(expr->getSourceRange(),
 897:                  std::string("unimplemented AMDGPU builtin call: ") +
 898:                      getContext().BuiltinInfo.getName(builtinId));
 899:     return mlir::Value{};
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 900-917
```cpp
 900:   }
 901:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fadd_f32:
 902:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fadd_v2f16: {
 903:     cgm.errorNYI(expr->getSourceRange(),
 904:                  std::string("unimplemented AMDGPU builtin call: ") +
 905:                      getContext().BuiltinInfo.getName(builtinId));
 906:     return mlir::Value{};
 907:   }
 908:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fmin_f32:
 909:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fmin_f64: {
 910:     cgm.errorNYI(expr->getSourceRange(),
 911:                  std::string("unimplemented AMDGPU builtin call: ") +
 912:                      getContext().BuiltinInfo.getName(builtinId));
 913:     return mlir::Value{};
 914:   }
 915:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fmax_f32:
 916:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fmax_f64: {
 917:     cgm.errorNYI(expr->getSourceRange(),
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 918-935
```cpp
 918:                  std::string("unimplemented AMDGPU builtin call: ") +
 919:                      getContext().BuiltinInfo.getName(builtinId));
 920:     return mlir::Value{};
 921:   }
 922:   case AMDGPU::BI__builtin_amdgcn_s_prefetch_data: {
 923:     cgm.errorNYI(expr->getSourceRange(),
 924:                  std::string("unimplemented AMDGPU builtin call: ") +
 925:                      getContext().BuiltinInfo.getName(builtinId));
 926:     return mlir::Value{};
 927:   }
 928:   case Builtin::BIlogbf:
 929:   case Builtin::BI__builtin_logbf:
 930:     return emitLogbBuiltin(*this, expr, llvm::APFloat::IEEEsingle());
 931:   case Builtin::BIlogb:
 932:   case Builtin::BI__builtin_logb:
 933:     return emitLogbBuiltin(*this, expr, llvm::APFloat::IEEEdouble());
 934:   case Builtin::BIscalbnf:
 935:   case Builtin::BI__builtin_scalbnf:
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`, `emitLogbBuiltin`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`、`emitLogbBuiltin`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 936-944
```cpp
 936:   case Builtin::BIscalbn:
 937:   case Builtin::BI__builtin_scalbn: {
 938:     return emitBinaryExpMaybeConstrainedFPBuiltin(
 939:         *this, expr, "ldexp", "experimental.constrained.ldexp");
 940:   }
 941:   default:
 942:     return std::nullopt;
 943:   }
 944: }
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

## Key Concepts / 关键概念

- **MLIR/CIR integration / MLIR/CIR 集成**: Bridges Clang logic to MLIR/CIR operations, attributes, or types. 将 Clang 逻辑连接到 MLIR/CIR 的操作、属性或类型。
- **CIR dialect usage / CIR 方言使用**: Manipulates CIR-specific types, attributes, and operations. 操作 CIR 专用的类型、属性和操作。
- **CIR generation pipeline / CIR 生成流水线**: Participates in the lowering pipeline from Clang semantics to CIR constructs. 参与从 Clang 语义到 CIR 构造的 lowering 流水线。
- **`emitBinaryExpMaybeConstrainedFPBuiltin` / `emitBinaryExpMaybeConstrainedFPBuiltin`**: `emitBinaryExpMaybeConstrainedFPBuiltin` is a prominent symbol in this file and helps define its structure or behavior. `emitBinaryExpMaybeConstrainedFPBuiltin` 是该文件中的关键符号之一，用来界定其结构或行为。
- **`fpOptsRAII` / `fpOptsRAII`**: `fpOptsRAII` is a prominent symbol in this file and helps define its structure or behavior. `fpOptsRAII` 是该文件中的关键符号之一，用来界定其结构或行为。
- **`emitLogbBuiltin` / `emitLogbBuiltin`**: `emitLogbBuiltin` is a prominent symbol in this file and helps define its structure or behavior. `emitLogbBuiltin` 是该文件中的关键符号之一，用来界定其结构或行为。

## Dependencies / 依赖关系

- **Clang / Clang**: `clang/Basic/TargetBuiltins.h`
- **LLVM / LLVM**: `llvm/Support/ErrorHandling.h`
- **MLIR / MLIR**: `mlir/IR/Value.h`
- **StdLib/Other / 标准库/其他**: `CIRGenFunction.h`
