# EHABILowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `clang/lib/CIR/Dialect/Transforms/EHABILowering.cpp`
- **Repository**: `/root/xw/llvm-project` (`llvm-project`)
- **Purpose (EN)**: This file implements a pass that lowers ABI-agnostic flattened CIR exception handling operations into an ABI-specific form. Currently only the Itanium C++ ABI is supported.
- **Purpose (CN)**: 实现与 `EHABILowering` 相关的 CIR 方言变换支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
   1: //===- EHABILowering.cpp - Lower flattened CIR EH ops to ABI-specific form ===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements a pass that lowers ABI-agnostic flattened CIR exception
  10: // handling operations into an ABI-specific form. Currently only the Itanium
  11: // C++ ABI is supported.
  12: //
  13: // The Itanium ABI lowering performs these transformations:
  14: //   - cir.eh.initiate            → cir.eh.inflight_exception (landing pad)
  15: //   - cir.eh.dispatch            → cir.eh.typeid + cir.cmp + cir.brcond chains
  16: //   - cir.begin_cleanup          → (removed)
  17: //   - cir.end_cleanup            → (removed)
  18: //   - cir.begin_catch            → call to __cxa_begin_catch
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 19-28
```cpp
  19: //   - cir.end_catch              → call to __cxa_end_catch
  20: //   - cir.eh.terminate           → call to __clang_call_terminate + unreachable
  21: //   - cir.resume                 → cir.resume.flat
  22: //   - !cir.eh_token values       → (!cir.ptr<!void>, !u32i) value pairs
  23: //   - cir.construct_catch_param  → __cxa_get_exception_ptr + inlined
  24: //                                  catch-copy thunk body
  25: //   - personality function set on functions requiring EH
  26: //
  27: //===----------------------------------------------------------------------===//
  28: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 29-45
```cpp
  29: #include "PassDetail.h"
  30: #include "mlir/IR/Builders.h"
  31: #include "mlir/IR/IRMapping.h"
  32: #include "mlir/IR/PatternMatch.h"
  33: #include "clang/CIR/Dialect/IR/CIRDialect.h"
  34: #include "clang/CIR/Dialect/IR/CIROpsEnums.h"
  35: #include "clang/CIR/Dialect/IR/CIRTypes.h"
  36: #include "clang/CIR/Dialect/Passes.h"
  37: #include "clang/CIR/Dialect/Transforms/CIRTransformUtils.h"
  38: #include "clang/CIR/MissingFeatures.h"
  39: #include "llvm/ADT/DenseMap.h"
  40: #include "llvm/ADT/SmallVector.h"
  41: #include "llvm/TargetParser/Triple.h"
  42: 
  43: using namespace mlir;
  44: using namespace cir;
  45: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `PassDetail.h`, `Builders.h`, `IRMapping.h`, `PatternMatch.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `PassDetail.h`, `Builders.h`, `IRMapping.h`, `PatternMatch.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 46-52
```cpp
  46: namespace mlir {
  47: #define GEN_PASS_DEF_CIREHABILOWERING
  48: #include "clang/CIR/Dialect/Passes.h.inc"
  49: } // namespace mlir
  50: 
  51: namespace {
  52: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `Passes.h.inc` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `Passes.h.inc` 这样的头文件说明了该区域依赖的主要 API。

### Lines 53-56
```cpp
  53: //===----------------------------------------------------------------------===//
  54: // Shared utilities
  55: //===----------------------------------------------------------------------===//
  56: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 57-65
```cpp
  57: /// Ensure a function with the given name and type exists in the module. If it
  58: /// does not exist, create a private external declaration.
  59: static cir::FuncOp getOrCreateRuntimeFuncDecl(mlir::ModuleOp mod,
  60:                                               mlir::Location loc,
  61:                                               StringRef name,
  62:                                               cir::FuncType funcTy) {
  63:   if (auto existing = mod.lookupSymbol<cir::FuncOp>(name))
  64:     return existing;
  65: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getOrCreateRuntimeFuncDecl`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getOrCreateRuntimeFuncDecl`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 66-73
```cpp
  66:   mlir::OpBuilder builder(mod.getContext());
  67:   builder.setInsertionPointToEnd(mod.getBody());
  68:   auto funcOp = cir::FuncOp::create(builder, loc, name, funcTy);
  69:   funcOp.setLinkage(cir::GlobalLinkageKind::ExternalLinkage);
  70:   funcOp.setPrivate();
  71:   return funcOp;
  72: }
  73: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `builder`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `builder`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 74-77
```cpp
  74: //===----------------------------------------------------------------------===//
  75: // EH ABI Lowering Base Class
  76: //===----------------------------------------------------------------------===//
  77: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 78-88
```cpp
  78: /// Abstract base class for exception-handling ABI lowering.
  79: /// Each supported ABI (Itanium, Microsoft, etc.) provides a concrete subclass.
  80: class EHABILowering {
  81: public:
  82:   explicit EHABILowering(mlir::ModuleOp mod)
  83:       : mod(mod), ctx(mod.getContext()), builder(ctx) {}
  84:   virtual ~EHABILowering() = default;
  85: 
  86:   /// Lower all EH operations in the module to an ABI-specific form.
  87:   virtual mlir::LogicalResult run() = 0;
  88: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Notable callable symbols here include `EHABILowering`. It introduces or references types such as `for`, `EHABILowering`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 其中值得关注的可调用符号包括 `EHABILowering`。 它引入或引用了诸如 `for`、`EHABILowering` 等类型。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 89-94
```cpp
  89: protected:
  90:   mlir::ModuleOp mod;
  91:   mlir::MLIRContext *ctx;
  92:   mlir::OpBuilder builder;
  93: };
  94: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 95-98
```cpp
  95: //===----------------------------------------------------------------------===//
  96: // Itanium EH ABI Lowering
  97: //===----------------------------------------------------------------------===//
  98: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 99-110
```cpp
  99: /// Lowers flattened CIR EH operations to the Itanium C++ ABI form.
 100: ///
 101: /// The entry point is run(), which iterates over all functions and
 102: /// calls lowerFunc() for each. lowerFunc() drives all lowering from
 103: /// cir.eh.initiate operations: every other EH op (begin/end_cleanup,
 104: /// eh.dispatch, begin/end_catch, resume) is reachable by tracing the
 105: /// eh_token produced by the initiate through its users.
 106: class ItaniumEHLowering : public EHABILowering {
 107: public:
 108:   using EHABILowering::EHABILowering;
 109:   mlir::LogicalResult run() override;
 110: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. It introduces or references types such as `ItaniumEHLowering`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 它引入或引用了诸如 `ItaniumEHLowering` 等类型。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 111-115
```cpp
 111: private:
 112:   /// Maps a !cir.eh_token value to its Itanium ABI replacement pair:
 113:   /// an exception pointer (!cir.ptr<!void>) and a type id (!u32i).
 114:   using EhTokenMap = DenseMap<mlir::Value, std::pair<mlir::Value, mlir::Value>>;
 115: 
```
- **EN**: This block organizes symbols with namespaces or using-declarations for shorter references. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块通过命名空间或 using 声明组织符号并简化后续引用。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 116-120
```cpp
 116:   cir::VoidType voidType;
 117:   cir::PointerType voidPtrType;
 118:   cir::PointerType u8PtrType;
 119:   cir::IntType u32Type;
 120: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 121-133
```cpp
 121:   // Cached runtime function declarations, initialized when needed by
 122:   // ensureRuntimeDecls().
 123:   cir::FuncOp personalityFunc;
 124:   cir::FuncOp beginCatchFunc;
 125:   cir::FuncOp endCatchFunc;
 126:   cir::FuncOp getExceptionPtrFunc;
 127:   cir::FuncOp clangCallTerminateFunc;
 128: 
 129:   DenseMap<mlir::StringAttr, cir::FuncOp> catchCopyThunks;
 130: 
 131:   constexpr const static ::llvm::StringLiteral kGxxPersonality =
 132:       "__gxx_personality_v0";
 133: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 134-150
```cpp
 134:   void ensureRuntimeDecls(mlir::Location loc);
 135:   void ensureClangCallTerminate(mlir::Location loc);
 136:   mlir::Block *buildTerminateBlock(cir::FuncOp funcOp, mlir::Location loc);
 137:   mlir::FailureOr<cir::FuncOp>
 138:   resolveCatchCopyThunk(cir::ConstructCatchParamOp op);
 139:   mlir::LogicalResult lowerFunc(cir::FuncOp funcOp);
 140:   mlir::LogicalResult
 141:   lowerEhInitiate(cir::EhInitiateOp initiateOp, EhTokenMap &ehTokenMap,
 142:                   SmallVectorImpl<mlir::Operation *> &deadOps);
 143:   void lowerDispatch(cir::EhDispatchOp dispatch, mlir::Value exnPtr,
 144:                      mlir::Value typeId,
 145:                      SmallVectorImpl<mlir::Operation *> &deadOps);
 146:   mlir::LogicalResult lowerConstructCatchParam(cir::ConstructCatchParamOp op,
 147:                                                mlir::Value exnPtr);
 148:   void lowerInitCatchParam(cir::InitCatchParamOp op);
 149: };
 150: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ensureRuntimeDecls`, `ensureClangCallTerminate`, `resolveCatchCopyThunk`, `lowerFunc`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ensureRuntimeDecls`、`ensureClangCallTerminate`、`resolveCatchCopyThunk`、`lowerFunc`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 151-160
```cpp
 151: /// Lower all EH operations in the module to the Itanium-specific form.
 152: mlir::LogicalResult ItaniumEHLowering::run() {
 153:   // Pre-compute the common types used throughout all function lowerings.
 154:   // TODO(cir): Move these to the base class if they are also needed for MSVC.
 155:   voidType = cir::VoidType::get(ctx);
 156:   voidPtrType = cir::PointerType::get(voidType);
 157:   auto u8Type = cir::IntType::get(ctx, 8, /*isSigned=*/false);
 158:   u8PtrType = cir::PointerType::get(u8Type);
 159:   u32Type = cir::IntType::get(ctx, 32, /*isSigned=*/false);
 160: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Notable callable symbols here include `ItaniumEHLowering::run`. It introduces or references types such as `if`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 其中值得关注的可调用符号包括 `ItaniumEHLowering::run`。 它引入或引用了诸如 `if` 等类型。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 161-167
```cpp
 161:   for (cir::FuncOp funcOp : mod.getOps<cir::FuncOp>()) {
 162:     if (mlir::failed(lowerFunc(funcOp)))
 163:       return mlir::failure();
 164:   }
 165:   return mlir::success();
 166: }
 167: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 168-179
```cpp
 168: /// Ensure the necessary Itanium runtime function declarations exist in the
 169: /// module.
 170: void ItaniumEHLowering::ensureRuntimeDecls(mlir::Location loc) {
 171:   // TODO(cir): Handle other personality functions. This probably isn't needed
 172:   // here if we fix codegen to always set the personality function.
 173:   if (!personalityFunc) {
 174:     auto s32Type = cir::IntType::get(ctx, 32, /*isSigned=*/true);
 175:     auto personalityFuncTy = cir::FuncType::get({}, s32Type, /*isVarArg=*/true);
 176:     personalityFunc = getOrCreateRuntimeFuncDecl(mod, loc, kGxxPersonality,
 177:                                                  personalityFuncTy);
 178:   }
 179: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ItaniumEHLowering::ensureRuntimeDecls`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ItaniumEHLowering::ensureRuntimeDecls`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 180-186
```cpp
 180:   if (!beginCatchFunc) {
 181:     auto beginCatchFuncTy =
 182:         cir::FuncType::get({voidPtrType}, u8PtrType, /*isVarArg=*/false);
 183:     beginCatchFunc = getOrCreateRuntimeFuncDecl(mod, loc, "__cxa_begin_catch",
 184:                                                 beginCatchFuncTy);
 185:   }
 186: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 187-192
```cpp
 187:   if (!endCatchFunc) {
 188:     auto endCatchFuncTy = cir::FuncType::get({}, voidType, /*isVarArg=*/false);
 189:     endCatchFunc =
 190:         getOrCreateRuntimeFuncDecl(mod, loc, "__cxa_end_catch", endCatchFuncTy);
 191:   }
 192: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getOrCreateRuntimeFuncDecl`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getOrCreateRuntimeFuncDecl`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 193-200
```cpp
 193:   if (!getExceptionPtrFunc) {
 194:     auto getExceptionPtrFuncTy =
 195:         cir::FuncType::get({voidPtrType}, u8PtrType, /*isVarArg=*/false);
 196:     getExceptionPtrFunc = getOrCreateRuntimeFuncDecl(
 197:         mod, loc, "__cxa_get_exception_ptr", getExceptionPtrFuncTy);
 198:   }
 199: }
 200: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 201-215
```cpp
 201: /// Ensure the __clang_call_terminate function exists in the module. This
 202: /// function is defined with a body that calls __cxa_begin_catch followed by
 203: /// std::terminate, matching the behavior of Clang's LLVM IR codegen.
 204: ///
 205: ///   void __clang_call_terminate(void *exn) nounwind noreturn {
 206: ///     __cxa_begin_catch(exn);
 207: ///     std::terminate();
 208: ///     unreachable;
 209: ///   }
 210: void ItaniumEHLowering::ensureClangCallTerminate(mlir::Location loc) {
 211:   if (clangCallTerminateFunc)
 212:     return;
 213: 
 214:   ensureRuntimeDecls(loc);
 215: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ItaniumEHLowering::ensureClangCallTerminate`, `ensureRuntimeDecls`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ItaniumEHLowering::ensureClangCallTerminate`、`ensureRuntimeDecls`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 216-220
```cpp
 216:   if (auto existing = mod.lookupSymbol<cir::FuncOp>("__clang_call_terminate")) {
 217:     clangCallTerminateFunc = existing;
 218:     return;
 219:   }
 220: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 221-227
```cpp
 221:   auto funcTy = cir::FuncType::get({voidPtrType}, voidType, /*isVarArg=*/false);
 222:   builder.setInsertionPointToEnd(mod.getBody());
 223:   auto funcOp =
 224:       cir::FuncOp::create(builder, loc, "__clang_call_terminate", funcTy);
 225:   funcOp.setLinkage(cir::GlobalLinkageKind::LinkOnceODRLinkage);
 226:   funcOp.setGlobalVisibility(cir::VisibilityKind::Hidden);
 227: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::FuncOp::create`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::FuncOp::create`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 228-231
```cpp
 228:   mlir::Block *entryBlock = funcOp.addEntryBlock();
 229:   builder.setInsertionPointToStart(entryBlock);
 230:   mlir::Value exnArg = entryBlock->getArgument(0);
 231: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 232-236
```cpp
 232:   auto catchCall = cir::CallOp::create(
 233:       builder, loc, mlir::FlatSymbolRefAttr::get(beginCatchFunc), u8PtrType,
 234:       mlir::ValueRange{exnArg});
 235:   catchCall.setNothrowAttr(builder.getUnitAttr());
 236: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 237-250
```cpp
 237:   auto terminateFuncDecl = getOrCreateRuntimeFuncDecl(
 238:       mod, loc, "_ZSt9terminatev",
 239:       cir::FuncType::get({}, voidType, /*isVarArg=*/false));
 240:   terminateFuncDecl->setAttr(cir::CIRDialect::getNoReturnAttrName(),
 241:                              builder.getUnitAttr());
 242:   auto terminateCall = cir::CallOp::create(
 243:       builder, loc, mlir::FlatSymbolRefAttr::get(terminateFuncDecl), voidType,
 244:       mlir::ValueRange{});
 245:   terminateCall.setNothrowAttr(builder.getUnitAttr());
 246:   terminateCall->setAttr(cir::CIRDialect::getNoReturnAttrName(),
 247:                          builder.getUnitAttr());
 248: 
 249:   cir::UnreachableOp::create(builder, loc);
 250: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::UnreachableOp::create`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::UnreachableOp::create`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 251-255
```cpp
 251:   funcOp->setAttr(cir::CIRDialect::getNoReturnAttrName(),
 252:                   builder.getUnitAttr());
 253:   clangCallTerminateFunc = funcOp;
 254: }
 255: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 256-275
```cpp
 256: /// Create a terminate landing pad block at the end of the specified function.
 257: mlir::Block *ItaniumEHLowering::buildTerminateBlock(cir::FuncOp funcOp,
 258:                                                     mlir::Location loc) {
 259:   assert(clangCallTerminateFunc &&
 260:          "ensureClangCallTerminate must run before buildTerminateBlock");
 261:   mlir::Region &body = funcOp.getRegion();
 262:   mlir::Block *terminateBlock = builder.createBlock(&body, body.end());
 263:   auto inflight = cir::EhInflightOp::create(
 264:       builder, loc, /*cleanup=*/false, /*catch_all=*/true,
 265:       /*catch_type_list=*/mlir::ArrayAttr{});
 266:   auto terminateCall = cir::CallOp::create(
 267:       builder, loc, mlir::FlatSymbolRefAttr::get(clangCallTerminateFunc),
 268:       voidType, mlir::ValueRange{inflight.getExceptionPtr()});
 269:   terminateCall.setNothrowAttr(builder.getUnitAttr());
 270:   terminateCall->setAttr(cir::CIRDialect::getNoReturnAttrName(),
 271:                          builder.getUnitAttr());
 272:   cir::UnreachableOp::create(builder, loc);
 273:   return terminateBlock;
 274: }
 275: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`, `cir::UnreachableOp::create`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`、`cir::UnreachableOp::create`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 276-280
```cpp
 276: /// Lower all EH operations in a single function.
 277: mlir::LogicalResult ItaniumEHLowering::lowerFunc(cir::FuncOp funcOp) {
 278:   if (funcOp.isDeclaration())
 279:     return mlir::success();
 280: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ItaniumEHLowering::lowerFunc`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ItaniumEHLowering::lowerFunc`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 281-291
```cpp
 281:   // All EH lowering follows from cir.eh.initiate operations. The token each
 282:   // initiate produces connects it to every other EH op in the function
 283:   // (begin/end_cleanup, eh.dispatch, begin/end_catch, resume) through the
 284:   // token graph. A single walk to collect initiates is therefore sufficient.
 285:   SmallVector<cir::EhInitiateOp> initiateOps;
 286:   funcOp.walk([&](cir::EhInitiateOp op) { initiateOps.push_back(op); });
 287:   if (initiateOps.empty())
 288:     return mlir::success();
 289: 
 290:   ensureRuntimeDecls(funcOp.getLoc());
 291: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ensureRuntimeDecls`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ensureRuntimeDecls`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 292-299
```cpp
 292:   // Set the personality function if it is not already set.
 293:   // TODO(cir): The personality function should already have been set by this
 294:   // point. If we've seen a try operation, it will have been set by
 295:   // emitCXXTryStmt. If we only have cleanups, it may not have been set. We
 296:   // need to fix that in CodeGen. This is a placeholder until that is done.
 297:   if (!funcOp.getPersonality())
 298:     funcOp.setPersonality(kGxxPersonality);
 299: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 300-310
```cpp
 300:   // Lower each initiate and all EH ops connected to it. The token map is
 301:   // shared across all initiate operations. Multiple initiates may flow into the
 302:   // same dispatch block, and the map ensures the arguments are registered
 303:   // only once. Dispatch ops are scheduled for deferred removal so that sibling
 304:   // initiates can still read catch types from a shared dispatch.
 305:   EhTokenMap ehTokenMap;
 306:   SmallVector<mlir::Operation *> deadOps;
 307:   for (cir::EhInitiateOp initiateOp : initiateOps)
 308:     if (mlir::failed(lowerEhInitiate(initiateOp, ehTokenMap, deadOps)))
 309:       return mlir::failure();
 310: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 311-315
```cpp
 311:   // Erase operations that were deferred during per-initiate processing
 312:   // (dispatch ops whose catch types were read by multiple initiates).
 313:   for (mlir::Operation *op : deadOps)
 314:     op->erase();
 315: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 316-324
```cpp
 316:   // Remove the !cir.eh_token block arguments that were replaced by (ptr, u32)
 317:   // pairs. Iterate in reverse to preserve argument indices during removal.
 318:   for (mlir::Block &block : funcOp.getBody()) {
 319:     for (int i = block.getNumArguments() - 1; i >= 0; --i) {
 320:       if (mlir::isa<cir::EhTokenType>(block.getArgument(i).getType()))
 321:         block.eraseArgument(i);
 322:     }
 323:   }
 324: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 325-335
```cpp
 325:   // Lower any cir.init_catch_param ops in this function. These materialize
 326:   // the catch parameter local from the (already lowered) begin_catch result,
 327:   // and are independent of the eh_token graph traversal above.
 328:   SmallVector<cir::InitCatchParamOp> initCatchOps;
 329:   funcOp.walk([&](cir::InitCatchParamOp op) { initCatchOps.push_back(op); });
 330:   for (cir::InitCatchParamOp op : initCatchOps)
 331:     lowerInitCatchParam(op);
 332: 
 333:   return mlir::success();
 334: }
 335: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 336-353
```cpp
 336: /// Lower all EH operations connected to a single cir.eh.initiate.
 337: ///
 338: /// The cir.eh.initiate is the root of a token graph. The token it produces
 339: /// flows through branch edges to consuming operations:
 340: ///
 341: ///   cir.eh.initiate → (via cir.br) → cir.begin_cleanup
 342: ///                                     → cir.end_cleanup (via cleanup_token)
 343: ///                                   → (via cir.br) → cir.eh.dispatch
 344: ///                                                     → (successors) →
 345: ///                                                       cir.begin_catch
 346: ///                                                       → cir.end_catch
 347: ///                                                         (via catch_token)
 348: ///                                   → cir.resume
 349: ///
 350: /// A single traversal of the token graph discovers and processes every
 351: /// connected op inline. The inflight_exception is created up-front without
 352: /// a catch_type_list; when the dispatch is encountered during traversal,
 353: /// the catch types are read and set on the inflight op.
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 354-366
```cpp
 354: ///
 355: /// Dispatch ops are not erased during per-initiate processing because they may
 356: /// be used by other initiate ops that haven't yet been lowered. Instead they
 357: /// are added to \p deadOps and erased by the caller after all initiates have
 358: /// been lowered.
 359: ///
 360: /// \p ehTokenMap is shared across all initiates in the function so that block
 361: /// arguments reachable from multiple sibling initiates are registered once.
 362: mlir::LogicalResult ItaniumEHLowering::lowerEhInitiate(
 363:     cir::EhInitiateOp initiateOp, EhTokenMap &ehTokenMap,
 364:     SmallVectorImpl<mlir::Operation *> &deadOps) {
 365:   mlir::Value rootToken = initiateOp.getEhToken();
 366: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ItaniumEHLowering::lowerEhInitiate`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ItaniumEHLowering::lowerEhInitiate`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 367-377
```cpp
 367:   // Create the inflight_exception without a catch_type_list. The catch types
 368:   // will be set once we encounter the dispatch during the traversal below.
 369:   builder.setInsertionPoint(initiateOp);
 370:   auto inflightOp = cir::EhInflightOp::create(
 371:       builder, initiateOp.getLoc(), /*cleanup=*/initiateOp.getCleanup(),
 372:       /*catch_all=*/false,
 373:       /*catch_type_list=*/mlir::ArrayAttr{});
 374: 
 375:   ehTokenMap[rootToken] = {inflightOp.getExceptionPtr(),
 376:                            inflightOp.getTypeId()};
 377: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 378-385
```cpp
 378:   // Single traversal of the token graph. For each token value (the root token
 379:   // or a block argument that carries it), we snapshot its users, register
 380:   // (ptr, u32) replacement arguments on successor blocks, then process every
 381:   // user inline. This avoids collecting ops into separate vectors.
 382:   SmallVector<mlir::Value> worklist;
 383:   SmallPtrSet<mlir::Value, 8> visited;
 384:   worklist.push_back(rootToken);
 385: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 386-390
```cpp
 386:   while (!worklist.empty()) {
 387:     mlir::Value current = worklist.pop_back_val();
 388:     if (!visited.insert(current).second)
 389:       continue;
 390: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 391-396
```cpp
 391:     // Snapshot users before modifying any of them (erasing ops during
 392:     // iteration would invalidate the use-list iterator).
 393:     SmallVector<mlir::Operation *> users;
 394:     for (mlir::OpOperand &use : current.getUses())
 395:       users.push_back(use.getOwner());
 396: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 397-414
```cpp
 397:     // Register replacement block arguments on successor blocks (extending the
 398:     // worklist), then lower the op itself.
 399:     for (mlir::Operation *user : users) {
 400:       // Trace into successor blocks to register (ptr, u32) replacement
 401:       // arguments for any !cir.eh_token block arguments found there.  Even
 402:       // if a block arg was already registered by a sibling initiate, it is
 403:       // still added to the worklist so that the traversal can reach the
 404:       // shared dispatch to read catch types.
 405:       for (unsigned s = 0; s < user->getNumSuccessors(); ++s) {
 406:         mlir::Block *succ = user->getSuccessor(s);
 407:         for (mlir::BlockArgument arg : succ->getArguments()) {
 408:           if (!mlir::isa<cir::EhTokenType>(arg.getType()))
 409:             continue;
 410:           if (!ehTokenMap.count(arg)) {
 411:             mlir::Value ptrArg = succ->addArgument(voidPtrType, arg.getLoc());
 412:             mlir::Value u32Arg = succ->addArgument(u32Type, arg.getLoc());
 413:             ehTokenMap[arg] = {ptrArg, u32Arg};
 414:           }
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 415-418
```cpp
 415:           worklist.push_back(arg);
 416:         }
 417:       }
 418: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 419-436
```cpp
 419:       if (auto op = mlir::dyn_cast<cir::BeginCleanupOp>(user)) {
 420:         // begin_cleanup / end_cleanup are no-ops for Itanium.  Erase the
 421:         // end_cleanup first (drops the cleanup_token use) then the begin.
 422:         for (auto &tokenUsers :
 423:              llvm::make_early_inc_range(op.getCleanupToken().getUses())) {
 424:           if (auto endOp =
 425:                   mlir::dyn_cast<cir::EndCleanupOp>(tokenUsers.getOwner()))
 426:             endOp.erase();
 427:         }
 428:         op.erase();
 429:       } else if (auto op = mlir::dyn_cast<cir::BeginCatchOp>(user)) {
 430:         // Replace end_catch → __cxa_end_catch (drops the catch_token use),
 431:         // then replace begin_catch → __cxa_begin_catch.
 432:         for (auto &tokenUsers :
 433:              llvm::make_early_inc_range(op.getCatchToken().getUses())) {
 434:           if (auto endOp =
 435:                   mlir::dyn_cast<cir::EndCatchOp>(tokenUsers.getOwner())) {
 436:             builder.setInsertionPoint(endOp);
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 437-443
```cpp
 437:             cir::CallOp::create(builder, endOp.getLoc(),
 438:                                 mlir::FlatSymbolRefAttr::get(endCatchFunc),
 439:                                 voidType, mlir::ValueRange{});
 440:             endOp.erase();
 441:           }
 442:         }
 443: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 444-461
```cpp
 444:         auto [exnPtr, typeId] = ehTokenMap.lookup(op.getEhToken());
 445:         builder.setInsertionPoint(op);
 446:         auto callOp = cir::CallOp::create(
 447:             builder, op.getLoc(), mlir::FlatSymbolRefAttr::get(beginCatchFunc),
 448:             u8PtrType, mlir::ValueRange{exnPtr});
 449:         mlir::Value castResult = callOp.getResult();
 450:         mlir::Type expectedPtrType = op.getExnPtr().getType();
 451:         if (castResult.getType() != expectedPtrType)
 452:           castResult =
 453:               cir::CastOp::create(builder, op.getLoc(), expectedPtrType,
 454:                                   cir::CastKind::bitcast, callOp.getResult());
 455:         op.getExnPtr().replaceAllUsesWith(castResult);
 456:         op.erase();
 457:       } else if (auto op = mlir::dyn_cast<cir::ConstructCatchParamOp>(user)) {
 458:         auto [exnPtr, typeId] = ehTokenMap.lookup(op.getEhToken());
 459:         if (mlir::failed(lowerConstructCatchParam(op, exnPtr)))
 460:           return mlir::failure();
 461:       } else if (auto op = mlir::dyn_cast<cir::EhDispatchOp>(user)) {
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 462-479
```cpp
 462:         // Read catch types from the dispatch and set them on the inflight op.
 463:         mlir::ArrayAttr catchTypes = op.getCatchTypesAttr();
 464:         if (catchTypes && catchTypes.size() > 0) {
 465:           SmallVector<mlir::Attribute> typeSymbols;
 466:           for (mlir::Attribute attr : catchTypes)
 467:             typeSymbols.push_back(
 468:                 mlir::cast<cir::GlobalViewAttr>(attr).getSymbol());
 469:           inflightOp.setCatchTypeListAttr(builder.getArrayAttr(typeSymbols));
 470:         }
 471:         if (op.getDefaultIsCatchAll())
 472:           inflightOp.setCatchAllAttr(builder.getUnitAttr());
 473:         // Only lower the dispatch once. A sibling initiate sharing the same
 474:         // dispatch will still read its catch types (above), but the comparison
 475:         // chain and branch replacement are only created the first time.
 476:         if (!llvm::is_contained(deadOps, op.getOperation())) {
 477:           auto [exnPtr, typeId] = ehTokenMap.lookup(op.getEhToken());
 478:           lowerDispatch(op, exnPtr, typeId, deadOps);
 479:         }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `lowerDispatch`. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `lowerDispatch`。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 480-497
```cpp
 480:       } else if (auto op = mlir::dyn_cast<cir::EhTerminateOp>(user)) {
 481:         auto [exnPtr, typeId] = ehTokenMap.lookup(op.getEhToken());
 482:         ensureClangCallTerminate(op.getLoc());
 483:         builder.setInsertionPoint(op);
 484:         auto call = cir::CallOp::create(
 485:             builder, op.getLoc(),
 486:             mlir::FlatSymbolRefAttr::get(clangCallTerminateFunc), voidType,
 487:             mlir::ValueRange{exnPtr});
 488:         call.setNothrowAttr(builder.getUnitAttr());
 489:         call->setAttr(cir::CIRDialect::getNoReturnAttrName(),
 490:                       builder.getUnitAttr());
 491:         cir::UnreachableOp::create(builder, op.getLoc());
 492:         op.erase();
 493:       } else if (auto op = mlir::dyn_cast<cir::ResumeOp>(user)) {
 494:         auto [exnPtr, typeId] = ehTokenMap.lookup(op.getEhToken());
 495:         builder.setInsertionPoint(op);
 496:         cir::ResumeFlatOp::create(builder, op.getLoc(), exnPtr, typeId);
 497:         op.erase();
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ensureClangCallTerminate`, `cir::UnreachableOp::create`, `cir::ResumeFlatOp::create`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ensureClangCallTerminate`、`cir::UnreachableOp::create`、`cir::ResumeFlatOp::create`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 498-515
```cpp
 498:       } else if (auto op = mlir::dyn_cast<cir::BrOp>(user)) {
 499:         // Replace eh_token operands with the (ptr, u32) pair.
 500:         SmallVector<mlir::Value> newOperands;
 501:         bool changed = false;
 502:         for (mlir::Value operand : op.getDestOperands()) {
 503:           auto it = ehTokenMap.find(operand);
 504:           if (it != ehTokenMap.end()) {
 505:             newOperands.push_back(it->second.first);
 506:             newOperands.push_back(it->second.second);
 507:             changed = true;
 508:           } else {
 509:             newOperands.push_back(operand);
 510:           }
 511:         }
 512:         if (changed) {
 513:           builder.setInsertionPoint(op);
 514:           cir::BrOp::create(builder, op.getLoc(), op.getDest(), newOperands);
 515:           op.erase();
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::BrOp::create`. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::BrOp::create`。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 516-520
```cpp
 516:         }
 517:       }
 518:     }
 519:   }
 520: 
```
- **EN**: This block closes scopes, namespaces, or declarations introduced earlier in the file.
- **CN**: 该代码块结束前文引入的作用域、命名空间或声明。

### Lines 521-524
```cpp
 521:   initiateOp.erase();
 522:   return mlir::success();
 523: }
 524: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 525-536
```cpp
 525: /// Lower a cir.eh.dispatch by creating a comparison chain in new blocks.
 526: /// The dispatch itself is replaced with a branch to the first comparison
 527: /// block and added to deadOps for deferred removal.
 528: void ItaniumEHLowering::lowerDispatch(
 529:     cir::EhDispatchOp dispatch, mlir::Value exnPtr, mlir::Value typeId,
 530:     SmallVectorImpl<mlir::Operation *> &deadOps) {
 531:   mlir::Location dispLoc = dispatch.getLoc();
 532:   mlir::Block *defaultDest = dispatch.getDefaultDestination();
 533:   mlir::ArrayAttr catchTypes = dispatch.getCatchTypesAttr();
 534:   mlir::SuccessorRange catchDests = dispatch.getCatchDestinations();
 535:   mlir::Block *dispatchBlock = dispatch->getBlock();
 536: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ItaniumEHLowering::lowerDispatch`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ItaniumEHLowering::lowerDispatch`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 537-547
```cpp
 537:   // Build the comparison chain in new blocks inserted after the dispatch's
 538:   // block. The dispatch itself is replaced with a branch to the first
 539:   // comparison block and scheduled for deferred removal.
 540:   if (!catchTypes || catchTypes.empty()) {
 541:     // No typed catches: replace dispatch with a direct branch.
 542:     builder.setInsertionPoint(dispatch);
 543:     cir::BrOp::create(builder, dispLoc, defaultDest,
 544:                       mlir::ValueRange{exnPtr, typeId});
 545:   } else {
 546:     unsigned numCatches = catchTypes.size();
 547: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 548-561
```cpp
 548:     // Create and populate comparison blocks in reverse order so that each
 549:     // block's false destination (the next comparison block, or defaultDest
 550:     // for the last one) is already available. Each createBlock inserts
 551:     // before the previous one, so the blocks end up in forward order.
 552:     mlir::Block *insertBefore = dispatchBlock->getNextNode();
 553:     mlir::Block *falseDest = defaultDest;
 554:     mlir::Block *firstCmpBlock = nullptr;
 555:     for (int i = numCatches - 1; i >= 0; --i) {
 556:       auto *cmpBlock = builder.createBlock(insertBefore, {voidPtrType, u32Type},
 557:                                            {dispLoc, dispLoc});
 558: 
 559:       mlir::Value cmpExnPtr = cmpBlock->getArgument(0);
 560:       mlir::Value cmpTypeId = cmpBlock->getArgument(1);
 561: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 562-567
```cpp
 562:       auto globalView = mlir::cast<cir::GlobalViewAttr>(catchTypes[i]);
 563:       auto ehTypeIdOp =
 564:           cir::EhTypeIdOp::create(builder, dispLoc, globalView.getSymbol());
 565:       auto cmpOp = cir::CmpOp::create(builder, dispLoc, cir::CmpOpKind::eq,
 566:                                       cmpTypeId, ehTypeIdOp.getTypeId());
 567: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::EhTypeIdOp::create`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::EhTypeIdOp::create`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 568-571
```cpp
 568:       cir::BrCondOp::create(builder, dispLoc, cmpOp, catchDests[i], falseDest,
 569:                             mlir::ValueRange{cmpExnPtr, cmpTypeId},
 570:                             mlir::ValueRange{cmpExnPtr, cmpTypeId});
 571: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 572-576
```cpp
 572:       insertBefore = cmpBlock;
 573:       falseDest = cmpBlock;
 574:       firstCmpBlock = cmpBlock;
 575:     }
 576: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 577-582
```cpp
 577:     // Replace the dispatch with a branch to the first comparison block.
 578:     builder.setInsertionPoint(dispatch);
 579:     cir::BrOp::create(builder, dispLoc, firstCmpBlock,
 580:                       mlir::ValueRange{exnPtr, typeId});
 581:   }
 582: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 583-588
```cpp
 583:   // Schedule the dispatch for deferred removal. We cannot erase it now because
 584:   // a sibling initiate that shares this dispatch may still need to read its
 585:   // catch types.
 586:   deadOps.push_back(dispatch);
 587: }
 588: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 589-596
```cpp
 589: mlir::FailureOr<cir::FuncOp>
 590: ItaniumEHLowering::resolveCatchCopyThunk(cir::ConstructCatchParamOp op) {
 591:   mlir::FlatSymbolRefAttr thunkRef = op.getCopyFnAttr();
 592:   mlir::StringAttr thunkName = thunkRef.getAttr();
 593:   auto cached = catchCopyThunks.find(thunkName);
 594:   if (cached != catchCopyThunks.end())
 595:     return cached->second;
 596: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ItaniumEHLowering::resolveCatchCopyThunk`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ItaniumEHLowering::resolveCatchCopyThunk`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 597-604
```cpp
 597:   cir::FuncOp thunk = mod.lookupSymbol<cir::FuncOp>(thunkRef);
 598:   if (!thunk)
 599:     return op.emitError("could not resolve catch-copy thunk symbol");
 600:   assert(thunk->hasAttr(cir::CIRDialect::getCatchCopyThunkAttrName()) &&
 601:          "verifier should have rejected non-thunk catch-copy reference");
 602:   if (thunk.isDeclaration())
 603:     return op.emitError("catch-copy thunk has no body to inline");
 604: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 605-608
```cpp
 605:   mlir::Region &thunkRegion = thunk.getRegion();
 606:   if (!llvm::hasSingleElement(thunkRegion))
 607:     return op.emitError("multi-block catch-copy thunks are NYI");
 608: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 609-614
```cpp
 609:   mlir::Block &thunkEntry = thunkRegion.front();
 610:   assert(thunkEntry.getNumArguments() == 2 &&
 611:          "catch-copy thunk must have exactly two parameters");
 612:   if (!mlir::isa<cir::ReturnOp>(thunkEntry.getTerminator()))
 613:     return op.emitError("catch-copy thunk must end in cir.return");
 614: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 615-618
```cpp
 615:   catchCopyThunks[thunkName] = thunk;
 616:   return thunk;
 617: }
 618: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 619-629
```cpp
 619: /// Lower a cir.construct_catch_param into the Itanium-specific sequence
 620: /// that runs before `__cxa_begin_catch` to bind the catch parameter to the
 621: /// in-flight exception.
 622: mlir::LogicalResult
 623: ItaniumEHLowering::lowerConstructCatchParam(cir::ConstructCatchParamOp op,
 624:                                             mlir::Value exnPtr) {
 625:   mlir::Location loc = op.getLoc();
 626:   mlir::Value paramAddr = op.getParamAddr();
 627:   cir::PointerType paramAddrType =
 628:       mlir::cast<cir::PointerType>(paramAddr.getType());
 629: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ItaniumEHLowering::lowerConstructCatchParam`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ItaniumEHLowering::lowerConstructCatchParam`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 630-633
```cpp
 630:   if (op.getKind() == cir::InitCatchKind::Reference) {
 631:     assert(!MissingFeatures::sizeOfUnwindException());
 632:     constexpr unsigned headerSize = 32;
 633: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 634-648
```cpp
 634:     builder.setInsertionPoint(op);
 635:     auto index = cir::ConstantOp::create(
 636:         builder, loc, cir::IntAttr::get(u32Type, headerSize));
 637:     assert((exnPtr.getType() == voidPtrType || exnPtr.getType() == u8PtrType) &&
 638:            "lowerConstructCatchParam exn ptr not void* or i8*");
 639:     auto exnObj =
 640:         cir::PtrStrideOp::create(builder, loc, exnPtr.getType(), exnPtr, index);
 641:     mlir::Value casted =
 642:         cir::CastOp::create(builder, loc, paramAddrType.getPointee(),
 643:                             cir::CastKind::bitcast, exnObj);
 644:     cir::StoreOp::create(builder, loc, casted, paramAddr, {}, {}, {}, {});
 645:     op.erase();
 646:     return success();
 647:   }
 648: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::IntAttr::get`, `assert`, `cir::PtrStrideOp::create`, `cir::CastOp::create`. Assertions document invariants that the implementation expects to hold. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::IntAttr::get`、`assert`、`cir::PtrStrideOp::create`、`cir::CastOp::create`。 断言用于说明实现期望始终成立的不变量。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 649-655
```cpp
 649:   if (op.getKind() != cir::InitCatchKind::NonTrivialCopy)
 650:     return op.emitError(
 651:         "ConstructCatchParam: only non_trivial_copy is supported");
 652: 
 653:   ensureRuntimeDecls(loc);
 654:   ensureClangCallTerminate(loc);
 655: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ensureRuntimeDecls`, `ensureClangCallTerminate`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ensureRuntimeDecls`、`ensureClangCallTerminate`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 656-665
```cpp
 656:   // Call __cxa_get_exception_ptr to get the in-flight exception.
 657:   builder.setInsertionPoint(op);
 658:   cir::CallOp getExnCall = cir::CallOp::create(
 659:       builder, loc, mlir::FlatSymbolRefAttr::get(getExceptionPtrFunc),
 660:       u8PtrType, mlir::ValueRange{exnPtr});
 661:   getExnCall.setNothrowAttr(builder.getUnitAttr());
 662:   mlir::Value adjusted =
 663:       cir::CastOp::create(builder, loc, paramAddrType, cir::CastKind::bitcast,
 664:                           getExnCall.getResult());
 665: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::CastOp::create`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::CastOp::create`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 666-671
```cpp
 666:   // Get the thunk function definition.
 667:   mlir::FailureOr<cir::FuncOp> thunkOr = resolveCatchCopyThunk(op);
 668:   if (mlir::failed(thunkOr))
 669:     return mlir::failure();
 670:   cir::FuncOp thunk = *thunkOr;
 671: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 672-676
```cpp
 672:   // This is also verified by resolveCatchCopyThunk, but the loop below is
 673:   // where the constraint is required so let's assert it again here.
 674:   assert(llvm::hasSingleElement(thunk.getRegion()) &&
 675:          "multi-block catch-copy thunks are NYI");
 676: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。

### Lines 677-693
```cpp
 677:   // Clone the thunk function to perform the copy.
 678:   mlir::Block &thunkEntry = thunk.getRegion().front();
 679:   mlir::IRMapping mapping;
 680:   mapping.map(thunkEntry.getArgument(0), paramAddr);
 681:   mapping.map(thunkEntry.getArgument(1), adjusted);
 682:   llvm::SmallVector<cir::CallOp> throwingCalls;
 683:   for (mlir::Operation &thunkOp : thunkEntry.without_terminator()) {
 684:     mlir::Operation *cloned = builder.clone(thunkOp, mapping);
 685:     if (cir::CallOp callOp = mlir::dyn_cast<cir::CallOp>(cloned))
 686:       if (!callOp.getNothrow())
 687:         throwingCalls.push_back(callOp);
 688:   }
 689:   op.erase();
 690: 
 691:   if (throwingCalls.empty())
 692:     return mlir::success();
 693: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 694-706
```cpp
 694:   // All calls in the copy (which is usually just a single call) need to
 695:   // unwind to a terminate block if it throws an exception.
 696:   mlir::IRRewriter rewriter(builder);
 697:   mlir::Block *terminateBlock = nullptr;
 698:   for (cir::CallOp call : throwingCalls) {
 699:     if (!terminateBlock)
 700:       terminateBlock = buildTerminateBlock(call->getParentOfType<cir::FuncOp>(),
 701:                                            call.getLoc());
 702:     cir::replaceCallWithTryCall(call, terminateBlock, call.getLoc(), rewriter);
 703:   }
 704:   return mlir::success();
 705: }
 706: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `rewriter`, `cir::replaceCallWithTryCall`. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `rewriter`、`cir::replaceCallWithTryCall`。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 707-724
```cpp
 707: /// Lower a cir.init_catch_param into the Itanium-specific sequence that
 708: /// materializes the catch parameter's local variable from the exception
 709: /// pointer returned by __cxa_begin_catch. The shape of the lowering
 710: /// depends on the init catch kind:
 711: ///
 712: ///   - Reference: the begin_catch result is
 713: ///     the pointer value itself, so just bitcast and store it into the alloca
 714: ///     except if it reference of pointer of record.
 715: ///   - Pointer: the begin_catch result is
 716: ///     the pointer value itself, so just bitcast and store it into the
 717: ///     alloca.
 718: ///   - Scalar (any other by-value catch): treat the begin_catch result as a
 719: ///     pointer to the value, load it, and store it into the alloca.
 720: ///   - Objc: Handle pointer representation with ObjCLifetime.
 721: ///   - TrivialCopy: copy the exception
 722: ///     object's bytes into the alloca via cir.copy.
 723: ///   - NonTrivialCopy: the construction was already performed by the
 724: ///     companion `cir.construct_catch_param` before `cir.begin_catch`, so
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 725-735
```cpp
 725: ///     this lowering is a no-op.
 726: ///
 727: void ItaniumEHLowering::lowerInitCatchParam(cir::InitCatchParamOp op) {
 728:   builder.setInsertionPoint(op);
 729:   mlir::Location loc = op.getLoc();
 730:   mlir::Value exnPtr = op.getExnPtr();
 731:   mlir::Value paramAddr = op.getParamAddr();
 732:   auto paramAddrType = mlir::cast<cir::PointerType>(paramAddr.getType());
 733:   mlir::Type elementType = paramAddrType.getPointee();
 734:   cir::InitCatchKind kind = op.getKind();
 735: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ItaniumEHLowering::lowerInitCatchParam`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ItaniumEHLowering::lowerInitCatchParam`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 736-750
```cpp
 736:   switch (kind) {
 737:   case InitCatchKind::Reference: {
 738:     // We have no way to tell the personality function that we're
 739:     // catching by reference, so if we're catching a pointer,
 740:     // __cxa_begin_catch will actually return that pointer by value.
 741:     if (const auto ref = mlir::dyn_cast<cir::PointerType>(elementType)) {
 742:       // When catching by reference, generally we should just ignore
 743:       // this by-value pointer and use the exception object instead.
 744:       if (auto ptr = mlir::dyn_cast<cir::PointerType>(ref.getPointee()))
 745:         if (!mlir::isa<cir::RecordType>(ptr.getPointee()))
 746:           // Extracting and storing the actual exception object was performed by
 747:           // cir.construct_catch_param before cir.begin_catch.
 748:           break;
 749:     }
 750: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. A switch statement is used to dispatch behavior across enumerated cases or kinds. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 751-768
```cpp
 751:     mlir::Value casted = cir::CastOp::create(builder, loc, elementType,
 752:                                              cir::CastKind::bitcast, exnPtr);
 753:     cir::StoreOp::create(builder, loc, casted, paramAddr, {}, {}, {}, {});
 754:     break;
 755:   }
 756:   case InitCatchKind::TrivialCopy: {
 757:     mlir::Value srcPtr = cir::CastOp::create(builder, loc, paramAddrType,
 758:                                              cir::CastKind::bitcast, exnPtr);
 759:     cir::CopyOp::create(builder, loc, paramAddr, srcPtr, {}, {});
 760:     break;
 761:   }
 762:   case InitCatchKind::NonTrivialCopy:
 763:     // The non-trivial copy was performed by the matching
 764:     // cir.construct_catch_param before cir.begin_catch.
 765:     break;
 766:   case InitCatchKind::Scalar: {
 767:     // Scalar by-value catch (integer, float, complex, etc.). The begin_catch
 768:     // result points into the exception object; load the value through a
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 769-790
```cpp
 769:     // typed pointer and store it into the alloca.
 770:     mlir::Value srcPtr = cir::CastOp::create(builder, loc, paramAddrType,
 771:                                              cir::CastKind::bitcast, exnPtr);
 772:     auto loadOp = cir::LoadOp::create(builder, loc, elementType, srcPtr);
 773:     cir::StoreOp::create(builder, loc, loadOp.getResult(), paramAddr, {}, {},
 774:                          {}, {});
 775:     break;
 776:   }
 777:   case InitCatchKind::Pointer: {
 778:     mlir::Value casted = cir::CastOp::create(builder, loc, elementType,
 779:                                              cir::CastKind::bitcast, exnPtr);
 780:     cir::StoreOp::create(builder, loc, casted, paramAddr, {}, {}, {}, {});
 781:     break;
 782:   }
 783:   case InitCatchKind::Objc:
 784:     llvm_unreachable("InitCatchParam: ObjCLifetime is NYI");
 785:     break;
 786:   }
 787: 
 788:   op.erase();
 789: }
 790: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 791-794
```cpp
 791: //===----------------------------------------------------------------------===//
 792: // The Pass
 793: //===----------------------------------------------------------------------===//
 794: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 795-800
```cpp
 795: struct CIREHABILoweringPass
 796:     : public impl::CIREHABILoweringBase<CIREHABILoweringPass> {
 797:   CIREHABILoweringPass() = default;
 798:   void runOnOperation() override;
 799: };
 800: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. It introduces or references types such as `CIREHABILoweringPass`.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 它引入或引用了诸如 `CIREHABILoweringPass` 等类型。

### Lines 801-820
```cpp
 801: /// Erase all catch-init thunks after the EHABI lowering. CIRGen emits a thunk
 802: /// for every `cir.construct_catch_param` op, but those uses should all have
 803: /// been replaced during the lowering.
 804: static void eraseCatchCopyThunks(mlir::ModuleOp mod) {
 805:   llvm::StringRef catchHelperAttr =
 806:       cir::CIRDialect::getCatchCopyThunkAttrName();
 807:   for (cir::FuncOp f : llvm::make_early_inc_range(mod.getOps<cir::FuncOp>())) {
 808:     if (!f->hasAttr(catchHelperAttr))
 809:       continue;
 810:     // This is an expensive check, so we need to rely on the implementation
 811:     // to have done the right thing.
 812:     assert(mlir::SymbolTable::symbolKnownUseEmpty(f, mod) &&
 813:            "catch-init helper has remaining users");
 814:     f.erase();
 815:   }
 816: }
 817: 
 818: void CIREHABILoweringPass::runOnOperation() {
 819:   auto mod = mlir::cast<mlir::ModuleOp>(getOperation());
 820: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `eraseCatchCopyThunks`, `cir::CIRDialect::getCatchCopyThunkAttrName`, `assert`, `CIREHABILoweringPass::runOnOperation`. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `eraseCatchCopyThunks`、`cir::CIRDialect::getCatchCopyThunkAttrName`、`assert`、`CIREHABILoweringPass::runOnOperation`。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 821-830
```cpp
 821:   // The target triple is attached to the module as the "cir.triple"
 822:   // attribute. If it is absent (e.g. a CIR module parsed from text without a
 823:   // triple) we cannot determine the ABI and must skip the pass.
 824:   auto tripleAttr = mlir::dyn_cast_if_present<mlir::StringAttr>(
 825:       mod->getAttr(cir::CIRDialect::getTripleAttrName()));
 826:   if (!tripleAttr) {
 827:     mod.emitError("Module has no target triple");
 828:     return;
 829:   }
 830: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 831-846
```cpp
 831:   // Select the ABI-specific lowering handler from the triple. The Microsoft
 832:   // C++ ABI targets a Windows MSVC environment; everything else uses Itanium.
 833:   // Extend this when Microsoft ABI lowering is added.
 834:   llvm::Triple triple(tripleAttr.getValue());
 835:   std::unique_ptr<EHABILowering> lowering;
 836:   if (triple.isWindowsMSVCEnvironment()) {
 837:     mod.emitError(
 838:         "EH ABI lowering is not yet implemented for the Microsoft ABI");
 839:     return signalPassFailure();
 840:   } else {
 841:     lowering = std::make_unique<ItaniumEHLowering>(mod);
 842:   }
 843: 
 844:   if (mlir::failed(lowering->run()))
 845:     return signalPassFailure();
 846: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `triple`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `triple`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 847-852
```cpp
 847:   // Sweep away any the thunk functions. They've been inlined to all users now.
 848:   eraseCatchCopyThunks(mod);
 849: }
 850: 
 851: } // namespace
 852: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `eraseCatchCopyThunks`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `eraseCatchCopyThunks`。

### Lines 853-855
```cpp
 853: std::unique_ptr<Pass> mlir::createCIREHABILoweringPass() {
 854:   return std::make_unique<CIREHABILoweringPass>();
 855: }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `mlir::createCIREHABILoweringPass`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `mlir::createCIREHABILoweringPass`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

## Key Concepts / 关键概念

- **Declaration handling / 声明处理**: Works with declaration nodes that describe source-level entities. 处理描述源级实体的声明节点。
- **Statement handling / 语句处理**: Examines statement-level control flow or expression trees. 分析语句级控制流或表达式树。
- **Symbol tracking / 符号跟踪**: Tracks symbols that represent abstract runtime objects. 跟踪代表抽象运行时对象的符号。
- **MLIR/CIR integration / MLIR/CIR 集成**: Bridges Clang logic to MLIR/CIR operations, attributes, or types. 将 Clang 逻辑连接到 MLIR/CIR 的操作、属性或类型。
- **CIR dialect usage / CIR 方言使用**: Manipulates CIR-specific types, attributes, and operations. 操作 CIR 专用的类型、属性和操作。
- **CIR generation pipeline / CIR 生成流水线**: Participates in the lowering pipeline from Clang semantics to CIR constructs. 参与从 Clang 语义到 CIR 构造的 lowering 流水线。

## Dependencies / 依赖关系

- **Clang / Clang**: `clang/CIR/Dialect/IR/CIRDialect.h`, `clang/CIR/Dialect/IR/CIROpsEnums.h`, `clang/CIR/Dialect/IR/CIRTypes.h`, `clang/CIR/Dialect/Passes.h`, `clang/CIR/Dialect/Transforms/CIRTransformUtils.h`, `clang/CIR/MissingFeatures.h`, `clang/CIR/Dialect/Passes.h.inc`
- **LLVM / LLVM**: `llvm/ADT/DenseMap.h`, `llvm/ADT/SmallVector.h`, `llvm/TargetParser/Triple.h`
- **MLIR / MLIR**: `mlir/IR/Builders.h`, `mlir/IR/IRMapping.h`, `mlir/IR/PatternMatch.h`
- **StdLib/Other / 标准库/其他**: `PassDetail.h`
