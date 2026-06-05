# CIRGenBuiltinRISCV.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `clang/lib/CIR/CodeGen/CIRGenBuiltinRISCV.cpp`
- **Repository**: `/root/xw/llvm-project` (`llvm-project`)
- **Purpose (EN)**: This contains code to emit RISC-V Builtin calls as CIR or a function call to be later resolved.
- **Purpose (CN)**: 实现与 `CIRGenBuiltinRISCV` 相关的 CIR 代码生成支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```cpp
   1: //===----------------------------------------------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This contains code to emit RISC-V Builtin calls as CIR or a function call
  10: // to be later resolved.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "CIRGenFunction.h"
  15: #include "clang/Basic/TargetBuiltins.h"
  16: 
  17: using namespace clang;
  18: using namespace clang::CIRGen;
  19: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `CIRGenFunction.h`, `TargetBuiltins.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `CIRGenFunction.h`, `TargetBuiltins.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 20-30
```cpp
  20: std::optional<mlir::Value>
  21: CIRGenFunction::emitRISCVBuiltinExpr(unsigned builtinID, const CallExpr *e) {
  22:   if (builtinID == Builtin::BI__builtin_cpu_supports ||
  23:       builtinID == Builtin::BI__builtin_cpu_init ||
  24:       builtinID == Builtin::BI__builtin_cpu_is) {
  25:     cgm.errorNYI(e->getSourceRange(),
  26:                  std::string("unimplemented RISC-V builtin call: ") +
  27:                      getContext().BuiltinInfo.getName(builtinID));
  28:     return mlir::Value{};
  29:   }
  30: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenFunction::emitRISCVBuiltinExpr`, `std::string`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenFunction::emitRISCVBuiltinExpr`、`std::string`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 31-34
```cpp
  31:   StringRef intrinsicName;
  32:   mlir::Type returnType = convertType(e->getType());
  33:   llvm::SmallVector<mlir::Value> ops;
  34: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 35-43
```cpp
  35:   // `iceArguments` is a bitmap indicating whether the argument at the i-th bit
  36:   // is required to be a constant integer expression.
  37:   unsigned iceArguments = 0;
  38:   ASTContext::GetBuiltinTypeError error;
  39:   getContext().GetBuiltinType(builtinID, error, &iceArguments);
  40:   assert(error == ASTContext::GE_None && "Should not codegen an error");
  41:   for (auto [idx, arg] : llvm::enumerate(e->arguments()))
  42:     ops.push_back(emitScalarOrConstFoldImmArg(iceArguments, idx, arg));
  43: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `getContext`, `assert`. Loop constructs indicate repeated processing over collections or state transitions. Assertions document invariants that the implementation expects to hold.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `getContext`、`assert`。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 断言用于说明实现期望始终成立的不变量。

### Lines 44-47
```cpp
  44:   switch (builtinID) {
  45:   default:
  46:     llvm_unreachable("unexpected builtin ID");
  47: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm_unreachable`. A switch statement is used to dispatch behavior across enumerated cases or kinds.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm_unreachable`。 这里使用 `switch` 在多个枚举分支或类别之间进行分派。

### Lines 48-54
```cpp
  48:   // Zbb
  49:   case RISCV::BI__builtin_riscv_orc_b_32:
  50:   case RISCV::BI__builtin_riscv_orc_b_64: {
  51:     intrinsicName = "riscv.orc.b";
  52:     break;
  53:   }
  54: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 55-71
```cpp
  55:   // Zbc
  56:   case RISCV::BI__builtin_riscv_clmul_32:
  57:   case RISCV::BI__builtin_riscv_clmul_64: {
  58:     intrinsicName = "clmul";
  59:     break;
  60:   }
  61:   case RISCV::BI__builtin_riscv_clmulh_32:
  62:   case RISCV::BI__builtin_riscv_clmulh_64: {
  63:     intrinsicName = "riscv.clmulh";
  64:     break;
  65:   }
  66:   case RISCV::BI__builtin_riscv_clmulr_32:
  67:   case RISCV::BI__builtin_riscv_clmulr_64: {
  68:     intrinsicName = "riscv.clmulr";
  69:     break;
  70:   }
  71: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 72-89
```cpp
  72:   // Zbkx
  73:   case RISCV::BI__builtin_riscv_xperm4_32:
  74:   case RISCV::BI__builtin_riscv_xperm4_64: {
  75:     intrinsicName = "riscv.xperm4";
  76:     break;
  77:   }
  78:   case RISCV::BI__builtin_riscv_xperm8_32:
  79:   case RISCV::BI__builtin_riscv_xperm8_64: {
  80:     intrinsicName = "riscv.xperm8";
  81:     break;
  82:   }
  83:   // Zbkb
  84:   case RISCV::BI__builtin_riscv_brev8_32:
  85:   case RISCV::BI__builtin_riscv_brev8_64: {
  86:     intrinsicName = "riscv.brev8";
  87:     break;
  88:   }
  89:   case RISCV::BI__builtin_riscv_zip_32: {
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 90-107
```cpp
  90:     intrinsicName = "riscv.zip";
  91:     break;
  92:   }
  93:   case RISCV::BI__builtin_riscv_unzip_32: {
  94:     intrinsicName = "riscv.unzip";
  95:     break;
  96:   }
  97:   // Zknh
  98:   case RISCV::BI__builtin_riscv_sha256sig0: {
  99:     intrinsicName = "riscv.sha256sig0";
 100:     break;
 101:   }
 102:   case RISCV::BI__builtin_riscv_sha256sig1: {
 103:     intrinsicName = "riscv.sha256sig1";
 104:     break;
 105:   }
 106:   case RISCV::BI__builtin_riscv_sha256sum0: {
 107:     intrinsicName = "riscv.sha256sum0";
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 108-125
```cpp
 108:     break;
 109:   }
 110:   case RISCV::BI__builtin_riscv_sha256sum1: {
 111:     intrinsicName = "riscv.sha256sum1";
 112:     break;
 113:   }
 114:   // Zksed
 115:   case RISCV::BI__builtin_riscv_sm4ks: {
 116:     intrinsicName = "riscv.sm4ks";
 117:     break;
 118:   }
 119:   case RISCV::BI__builtin_riscv_sm4ed: {
 120:     intrinsicName = "riscv.sm4ed";
 121:     break;
 122:   }
 123:   // Zksh
 124:   case RISCV::BI__builtin_riscv_sm3p0: {
 125:     intrinsicName = "riscv.sm3p0";
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 126-143
```cpp
 126:     break;
 127:   }
 128:   case RISCV::BI__builtin_riscv_sm3p1: {
 129:     intrinsicName = "riscv.sm3p1";
 130:     break;
 131:   }
 132:   // Zbb
 133:   case RISCV::BI__builtin_riscv_clz_32:
 134:   case RISCV::BI__builtin_riscv_clz_64: {
 135:     mlir::Location loc = getLoc(e->getSourceRange());
 136:     auto op = cir::BitClzOp::create(builder, loc, ops[0],
 137:                                     /*poison_zero=*/false);
 138:     mlir::Value result = op.getResult();
 139:     if (result.getType() != returnType)
 140:       result = builder.createIntCast(result, returnType);
 141:     return result;
 142:   }
 143:   case RISCV::BI__builtin_riscv_ctz_32:
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 144-153
```cpp
 144:   case RISCV::BI__builtin_riscv_ctz_64: {
 145:     mlir::Location loc = getLoc(e->getSourceRange());
 146:     auto op = cir::BitCtzOp::create(builder, loc, ops[0],
 147:                                     /*poison_zero=*/false);
 148:     mlir::Value result = op.getResult();
 149:     if (result.getType() != returnType)
 150:       result = builder.createIntCast(result, returnType);
 151:     return result;
 152:   }
 153: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 154-162
```cpp
 154:   // Zihintntl
 155:   case RISCV::BI__builtin_riscv_ntl_load:
 156:   case RISCV::BI__builtin_riscv_ntl_store: {
 157:     cgm.errorNYI(e->getSourceRange(),
 158:                  std::string("unimplemented RISC-V builtin call: ") +
 159:                      getContext().BuiltinInfo.getName(builtinID));
 160:     return mlir::Value{};
 161:   }
 162: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 163-169
```cpp
 163:   // Zihintpause
 164:   case RISCV::BI__builtin_riscv_pause: {
 165:     intrinsicName = "riscv.pause";
 166:     returnType = builder.getVoidTy();
 167:     break;
 168:   }
 169: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 170-187
```cpp
 170:   // XCValu
 171:   case RISCV::BI__builtin_riscv_cv_alu_addN:
 172:   case RISCV::BI__builtin_riscv_cv_alu_addRN:
 173:   case RISCV::BI__builtin_riscv_cv_alu_adduN:
 174:   case RISCV::BI__builtin_riscv_cv_alu_adduRN:
 175:   case RISCV::BI__builtin_riscv_cv_alu_clip:
 176:   case RISCV::BI__builtin_riscv_cv_alu_clipu:
 177:   case RISCV::BI__builtin_riscv_cv_alu_extbs:
 178:   case RISCV::BI__builtin_riscv_cv_alu_extbz:
 179:   case RISCV::BI__builtin_riscv_cv_alu_exths:
 180:   case RISCV::BI__builtin_riscv_cv_alu_exthz:
 181:   case RISCV::BI__builtin_riscv_cv_alu_sle:
 182:   case RISCV::BI__builtin_riscv_cv_alu_sleu:
 183:   case RISCV::BI__builtin_riscv_cv_alu_subN:
 184:   case RISCV::BI__builtin_riscv_cv_alu_subRN:
 185:   case RISCV::BI__builtin_riscv_cv_alu_subuN:
 186:   case RISCV::BI__builtin_riscv_cv_alu_subuRN:
 187:   // XAndesPerf
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 188-207
```cpp
 188:   case RISCV::BI__builtin_riscv_nds_ffb_32:
 189:   case RISCV::BI__builtin_riscv_nds_ffb_64:
 190:   case RISCV::BI__builtin_riscv_nds_ffzmism_32:
 191:   case RISCV::BI__builtin_riscv_nds_ffzmism_64:
 192:   case RISCV::BI__builtin_riscv_nds_ffmism_32:
 193:   case RISCV::BI__builtin_riscv_nds_ffmism_64:
 194:   case RISCV::BI__builtin_riscv_nds_flmism_32:
 195:   case RISCV::BI__builtin_riscv_nds_flmism_64:
 196:   // XAndesBFHCvt
 197:   case RISCV::BI__builtin_riscv_nds_fcvt_s_bf16:
 198:   case RISCV::BI__builtin_riscv_nds_fcvt_bf16_s: {
 199:     cgm.errorNYI(e->getSourceRange(),
 200:                  std::string("unimplemented RISC-V builtin call: ") +
 201:                      getContext().BuiltinInfo.getName(builtinID));
 202:     return mlir::Value{};
 203:   }
 204: 
 205:     // TODO: Handle vector builtins in tablegen.
 206:   }
 207: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 208-210
```cpp
 208:   mlir::Location loc = getLoc(e->getSourceRange());
 209:   return builder.emitIntrinsicCallOp(loc, intrinsicName, returnType, ops);
 210: }
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

## Key Concepts / 关键概念

- **Clang AST integration / Clang AST 集成**: Reads semantic information from Clang AST data structures. 从 Clang AST 数据结构读取语义信息。
- **MLIR/CIR integration / MLIR/CIR 集成**: Bridges Clang logic to MLIR/CIR operations, attributes, or types. 将 Clang 逻辑连接到 MLIR/CIR 的操作、属性或类型。
- **CIR dialect usage / CIR 方言使用**: Manipulates CIR-specific types, attributes, and operations. 操作 CIR 专用的类型、属性和操作。
- **CIR generation pipeline / CIR 生成流水线**: Participates in the lowering pipeline from Clang semantics to CIR constructs. 参与从 Clang 语义到 CIR 构造的 lowering 流水线。
- **`CIRGenFunction::emitRISCVBuiltinExpr` / `CIRGenFunction::emitRISCVBuiltinExpr`**: `CIRGenFunction::emitRISCVBuiltinExpr` is a prominent symbol in this file and helps define its structure or behavior. `CIRGenFunction::emitRISCVBuiltinExpr` 是该文件中的关键符号之一，用来界定其结构或行为。
- **`std::string` / `std::string`**: `std::string` is a prominent symbol in this file and helps define its structure or behavior. `std::string` 是该文件中的关键符号之一，用来界定其结构或行为。

## Dependencies / 依赖关系

- **Clang / Clang**: `clang/Basic/TargetBuiltins.h`
- **StdLib/Other / 标准库/其他**: `CIRGenFunction.h`
