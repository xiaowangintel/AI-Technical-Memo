# TritonOps.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/Triton/IR/TritonOps.td`
- **EN:** Defines TableGen operation records, summaries, operands, results, and assembly syntax.
- **CN:** 定义 TableGen 操作记录，以及摘要、操作数、结果和汇编语法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```tablegen
   1: #ifndef TRITON_OPS
   2: #define TRITON_OPS
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-16
```tablegen
   4: include "triton/Dialect/Triton/IR/TritonDialect.td"
   5: include "triton/Dialect/Triton/IR/TritonTypes.td"
   6: include "triton/Dialect/Triton/IR/TritonAttrDefs.td"
   7: include "triton/Dialect/Triton/IR/TritonInterfaces.td"
   8: include "mlir/IR/OpBase.td"
   9: include "mlir/IR/SymbolInterfaces.td" // SymbolUserOpInterface
  10: include "mlir/IR/OpAsmInterface.td" // OpAsmOpInterface
  11: include "mlir/Interfaces/FunctionInterfaces.td" // FunctionOpInterface
  12: include "mlir/Interfaces/SideEffectInterfaces.td" // Pure
  13: include "mlir/Interfaces/ControlFlowInterfaces.td" // BranchOpInterface
  14: include "mlir/Interfaces/InferTypeOpInterface.td" // SameOperandsAndResultType
  15: include "mlir/Interfaces/CallInterfaces.td" // CallOpInterface
  16: include "triton/Dialect/Triton/IR/TritonOpInterfaces.td"
```
**EN:** This block imports the direct dependencies needed here, including triton/Dialect/Triton/IR/TritonDialect.td, triton/Dialect/Triton/IR/TritonTypes.td, triton/Dialect/Triton/IR/TritonAttrDefs.td, triton/Dialect/Triton/IR/TritonInterfaces.td, mlir/IR/OpBase.td, and mlir/IR/SymbolInterfaces.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 triton/Dialect/Triton/IR/TritonDialect.td, triton/Dialect/Triton/IR/TritonTypes.td, triton/Dialect/Triton/IR/TritonAttrDefs.td, triton/Dialect/Triton/IR/TritonInterfaces.td, mlir/IR/OpBase.td, and mlir/IR/SymbolInterfaces.td。

### Lines 19-22
```tablegen
  19: //
  20: // Interfaces
  21: //
  22: def GlobalMemory : Resource<"::mlir::triton::GlobalMemory">;
```
**EN:** This TableGen def record defines `GlobalMemory`. It is specialized from `Resource<"::mlir::triton::GlobalMemory">;`.
**CN:** 该 TableGen def 记录定义了 `GlobalMemory`。 它基于 `Resource<"::mlir::triton::GlobalMemory">;` 进一步特化。

### Lines 24-30
```tablegen
  24: //
  25: // Op Base
  26: //
  27: class TT_Op<string mnemonic, list<Trait> traits = []> :
  28:     Op<Triton_Dialect, mnemonic,
  29:        !listconcat(traits, [TensorSizeTrait, VerifyTensorLayoutsTrait])> {
  30: }
```
**EN:** This block introduces `TT_Op`, the main class/struct defined here. Within the declaration, methods such as listconcat expose its core API. It also inherits behavior from a base type.
**CN:** 该代码块引入了此文件的核心类/结构体 `TT_Op`。 其中 listconcat 等方法构成了它的主要接口。 它还通过继承复用基类能力。

### Lines 32-44
```tablegen
  32: //
  33: // Cast Ops
  34: //
  35: // Use cast ops in arith:
  36: //   bitcast
  37: //   fptoui, fptosi, uitofp, sitofp,
  38: //   extf, tructf,
  39: //   extui, extsi, tructi
  40: def TT_IntToPtrOp : TT_Op<"int_to_ptr", [Elementwise,
  41:                                          SameOperandsAndResultShape,
  42:                                          SameOperandsAndResultEncoding,
  43:                                          Pure]> {
  44:     let summary = "Cast int64 to pointer";
```
**EN:** This TableGen def record defines `TT_IntToPtrOp` with the summary “Cast int64 to pointer”. It is specialized from `TT_Op<"int_to_ptr", [Elementwise,`.
**CN:** 该 TableGen def 记录定义了 `TT_IntToPtrOp`，其摘要为“Cast int64 to pointer”。 它基于 `TT_Op<"int_to_ptr", [Elementwise,` 进一步特化。

### Lines 46-46
```tablegen
  46:     let arguments = (ins TT_I64Like:$src);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 48-48
```tablegen
  48:     let results = (outs TT_PtrLike:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 50-50
```tablegen
  50:     let assemblyFormat = "$src attr-dict `:` type($src) `->` type($result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 52-53
```tablegen
  52:     let hasCanonicalizer = 1;
  53: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 55-59
```tablegen
  55: def TT_PtrToIntOp : TT_Op<"ptr_to_int", [Elementwise,
  56:                                          SameOperandsAndResultShape,
  57:                                          SameOperandsAndResultEncoding,
  58:                                          Pure]> {
  59:     let summary = "Cast pointer to int64";
```
**EN:** This TableGen def record defines `TT_PtrToIntOp` with the summary “Cast pointer to int64”. It is specialized from `TT_Op<"ptr_to_int", [Elementwise,`.
**CN:** 该 TableGen def 记录定义了 `TT_PtrToIntOp`，其摘要为“Cast pointer to int64”。 它基于 `TT_Op<"ptr_to_int", [Elementwise,` 进一步特化。

### Lines 61-61
```tablegen
  61:     let arguments = (ins TT_PtrLike:$src);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 63-63
```tablegen
  63:     let results = (outs TT_I64Like:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 65-66
```tablegen
  65:     let assemblyFormat = "$src attr-dict `:` type($src) `->` type($result)";
  66: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 68-73
```tablegen
  68: // arith.bitcast doesn't support pointers
  69: def TT_BitcastOp : TT_Op<"bitcast", [Elementwise,
  70:                                      SameOperandsAndResultShape,
  71:                                      SameOperandsAndResultEncoding,
  72:                                      Pure]> {
  73:     let summary = "Cast between types of the same bitwidth";
```
**EN:** This TableGen def record defines `TT_BitcastOp` with the summary “Cast between types of the same bitwidth”. It is specialized from `TT_Op<"bitcast", [Elementwise,`.
**CN:** 该 TableGen def 记录定义了 `TT_BitcastOp`，其摘要为“Cast between types of the same bitwidth”。 它基于 `TT_Op<"bitcast", [Elementwise,` 进一步特化。

### Lines 75-75
```tablegen
  75:     let arguments = (ins TT_Type:$src);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 77-77
```tablegen
  77:     let results = (outs TT_Type:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 79-82
```tablegen
  79:     let assemblyFormat = "$src attr-dict `:` type($src) `->` type($result)";
  80:     let hasFolder = 1;
  81:     let hasVerifier = 1;
  82: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 84-88
```tablegen
  84: def TT_FpToFpOp : TT_Op<"fp_to_fp", [Elementwise,
  85:                                      SameOperandsAndResultShape,
  86:                                      SameOperandsAndResultEncoding,
  87:                                      Pure]> {
  88:     let summary = "Floating point casting for custom types";
```
**EN:** This TableGen def record defines `TT_FpToFpOp` with the summary “Floating point casting for custom types”. It is specialized from `TT_Op<"fp_to_fp", [Elementwise,`.
**CN:** 该 TableGen def 记录定义了 `TT_FpToFpOp`，其摘要为“Floating point casting for custom types”。 它基于 `TT_Op<"fp_to_fp", [Elementwise,` 进一步特化。

### Lines 90-91
```tablegen
  90:     let description = [{
  91:         Floating point casting for custom types (F8), and non-default rounding modes.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 93-94
```tablegen
  93:         F8 <-> FP16, BF16, FP32, FP64
  94:     }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 96-99
```tablegen
  96:     let arguments = (
  97:       ins TT_FloatLike:$src,
  98:       OptionalAttr<TT_RoundingModeAttr>:$rounding
  99:     );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 101-101
```tablegen
 101:     let results = (outs TT_FloatLike:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 103-103
```tablegen
 103:     let assemblyFormat = "$src attr-dict  (`,` `rounding` `=` $rounding^)? `:` type($src) `->` type($result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 105-105
```tablegen
 105:     let hasVerifier = 1;
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 107-108
```tablegen
 107:     let hasFolder = 1;
 108: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 110-112
```tablegen
 110: //
 111: // Arithmetic Ops
 112: //
```
**EN:** This comment block records the intent and constraints of the surrounding code: Arithmetic Ops.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 114-117
```tablegen
 114: def TT_ClampFOp : TT_Op<"clampf", [Elementwise,
 115:                                    SameOperandsAndResultType,
 116:                                    Pure]> {
 117:     let summary = "Clamp operation for floating point types";
```
**EN:** This TableGen def record defines `TT_ClampFOp` with the summary “Clamp operation for floating point types”. It is specialized from `TT_Op<"clampf", [Elementwise,`.
**CN:** 该 TableGen def 记录定义了 `TT_ClampFOp`，其摘要为“Clamp operation for floating point types”。 它基于 `TT_Op<"clampf", [Elementwise,` 进一步特化。

### Lines 119-120
```tablegen
 119:     let description = [{
 120:         Clamp operation for floating point types.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 122-123
```tablegen
 122:         The operation takes three arguments: x, min, and max. It returns a tensor of the same shape as x with its values clamped to the range [min, max].
 123:     }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 125-131
```tablegen
 125:     let arguments = (
 126:       ins
 127:       TT_FloatLike:$x,
 128:       TT_FloatLike:$min,
 129:       TT_FloatLike:$max,
 130:       TT_PropagateNanAttr:$propagateNan
 131:     );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 133-133
```tablegen
 133:     let results = (outs TT_FloatLike:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 135-139
```tablegen
 135:     // List $propagateNan explicitly rather than relying on attr-dict to pick it
 136:     // up, because if it's inside attr-dict, its value will be printed as a
 137:     // number rather than as a meaningful string.
 138:     let assemblyFormat = "$x `,` $min `,` $max `,` `propagateNan` `=` $propagateNan attr-dict `:` type($result)";
 139: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 141-143
```tablegen
 141: //
 142: // Math Ops
 143: //
```
**EN:** This comment block records the intent and constraints of the surrounding code: Math Ops.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 145-148
```tablegen
 145: def TT_PreciseSqrtOp : TT_Op<"precise_sqrt", [Elementwise,
 146:                                               SameOperandsAndResultType,
 147:                                               Pure]> {
 148:     let summary = "Precise sqrt for floating point types";
```
**EN:** This TableGen def record defines `TT_PreciseSqrtOp` with the summary “Precise sqrt for floating point types”. It is specialized from `TT_Op<"precise_sqrt", [Elementwise,`.
**CN:** 该 TableGen def 记录定义了 `TT_PreciseSqrtOp`，其摘要为“Precise sqrt for floating point types”。 它基于 `TT_Op<"precise_sqrt", [Elementwise,` 进一步特化。

### Lines 150-152
```tablegen
 150:     let description = [{
 151:         Precise sqrt for floating point types.
 152:     }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 154-154
```tablegen
 154:     let arguments = (ins TT_FloatLike:$x);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 156-156
```tablegen
 156:     let results = (outs TT_FloatLike:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 158-159
```tablegen
 158:     let assemblyFormat = "$x attr-dict `:` type($x)";
 159: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 161-164
```tablegen
 161: def TT_PreciseDivFOp : TT_Op<"precise_divf", [Elementwise,
 162:                                               SameOperandsAndResultType,
 163:                                               Pure]> {
 164:     let summary = "Precise div for floating point types";
```
**EN:** This TableGen def record defines `TT_PreciseDivFOp` with the summary “Precise div for floating point types”. It is specialized from `TT_Op<"precise_divf", [Elementwise,`.
**CN:** 该 TableGen def 记录定义了 `TT_PreciseDivFOp`，其摘要为“Precise div for floating point types”。 它基于 `TT_Op<"precise_divf", [Elementwise,` 进一步特化。

### Lines 166-168
```tablegen
 166:     let description = [{
 167:         Precise div for floating point types.
 168:     }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 170-170
```tablegen
 170:     let arguments = (ins TT_FloatLike:$x, TT_FloatLike:$y);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 172-172
```tablegen
 172:     let results = (outs TT_FloatLike:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 174-175
```tablegen
 174:     let assemblyFormat = "$x `,` $y attr-dict `:` type($x)";
 175: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 177-180
```tablegen
 177: def TT_MulhiUIOp : TT_Op<"mulhiui", [Elementwise,
 178:                                      SameOperandsAndResultType,
 179:                                      Pure]> {
 180:     let summary = "Most significant N bits of the 2N-bit product of two integers";
```
**EN:** This TableGen def record defines `TT_MulhiUIOp` with the summary “Most significant N bits of the 2N-bit product of two integers”. It is specialized from `TT_Op<"mulhiui", [Elementwise,`.
**CN:** 该 TableGen def 记录定义了 `TT_MulhiUIOp`，其摘要为“Most significant N bits of the 2N-bit product of two integers”。 它基于 `TT_Op<"mulhiui", [Elementwise,` 进一步特化。

### Lines 182-184
```tablegen
 182:     let description = [{
 183:         Most significant N bits of the 2N-bit product of two integers.
 184:     }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 186-186
```tablegen
 186:     let arguments = (ins TT_IntLike:$x, TT_IntLike:$y);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 188-188
```tablegen
 188:     let results = (outs TT_IntLike:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 190-191
```tablegen
 190:     let assemblyFormat = "$x `,` $y attr-dict `:` type($x)";
 191: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 193-203
```tablegen
 193: //
 194: // Pointer Arith Ops
 195: //
 196: def TT_AddPtrOp : TT_Op<"addptr",
 197:                         [Pure,
 198:                          Elementwise,
 199:                          SameOperandsAndResultShape,
 200:                          SameOperandsAndResultEncoding,
 201:                          TypesMatchWith<"result type matches ptr type",
 202:                                         "result", "ptr", "$_self">]> {
 203:     let arguments = (ins TT_PtrLike:$ptr, TT_IntLike:$offset);
```
**EN:** This TableGen def record defines `TT_AddPtrOp`. It is specialized from `TT_Op<"addptr",`.
**CN:** 该 TableGen def 记录定义了 `TT_AddPtrOp`。 它基于 `TT_Op<"addptr",` 进一步特化。

### Lines 205-205
```tablegen
 205:     let results = (outs TT_PtrLike:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 207-209
```tablegen
 207:     let assemblyFormat = "$ptr `,` $offset attr-dict `:` type($result) `,` type($offset)";
 208:     let hasFolder = 1;
 209: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 211-227
```tablegen
 211: //
 212: // Load/Store Ops
 213: //
 214: def TT_LoadOp : TT_Op<"load", [
 215:   SameLoadStoreOperandsAndResultShape,
 216:   SameLoadStoreOperandsAndResultEncoding,
 217:   AttrSizedOperandSegments,
 218:   DeclareOpInterfaceMethods<PredicatedOpInterface>,
 219:   DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
 220:   DeclareOpInterfaceMethods<InferTypeOpInterface>,
 221:   TypesMatchWith<"result matches ptr type", "ptr", "result", "getPointeeType($_self)">,
 222:   TypesMatchWith<"mask type matches ptr type", "ptr", "mask", "getI1SameShape(getPointeeType($_self))",
 223:                  "($_op.getOperands().size() <= 1) || std::equal_to<>()">,
 224:   TypesMatchWith<"other matches ptr type", "ptr", "other", "getPointeeType($_self)",
 225:                  "($_op.getOperands().size() <= 2) || std::equal_to<>()">
 226: ]> {
 227:     let summary = "Load from a pointer or tensor of pointers";
```
**EN:** This TableGen def record defines `TT_LoadOp` with the summary “Load from a pointer or tensor of pointers”. It is specialized from `TT_Op<"load", [`.
**CN:** 该 TableGen def 记录定义了 `TT_LoadOp`，其摘要为“Load from a pointer or tensor of pointers”。 它基于 `TT_Op<"load", [` 进一步特化。

### Lines 229-237
```tablegen
 229:     let arguments = (
 230:       ins
 231:       TT_PtrLike:$ptr,
 232:       Optional<TT_BoolLike>:$mask,
 233:       Optional<TT_Type>:$other,
 234:       DefaultValuedAttr<TT_CacheModifierAttr, "::mlir::triton::CacheModifier::NONE">:$cache,
 235:       DefaultValuedAttr<TT_EvictionPolicyAttr, "::mlir::triton::EvictionPolicy::NORMAL">:$evict,
 236:       DefaultValuedAttr<BoolAttr, "false">:$isVolatile
 237:     );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 239-239
```tablegen
 239:     let results = (outs TT_Type:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 241-248
```tablegen
 241:     let builders = [
 242:         // A tensor of pointers or a pointer to a scalar
 243:         OpBuilder<(ins "Value":$ptr, "triton::CacheModifier":$cache,
 244:                        "triton::EvictionPolicy":$evict, "bool":$isVolatile)>,
 245:         // A tensor of pointers or a pointer to a scalar with mask
 246:         OpBuilder<(ins "Value":$ptr, "Value":$mask, "triton::CacheModifier":$cache,
 247:                        "triton::EvictionPolicy":$evict, "bool":$isVolatile)>
 248:     ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 250-270
```tablegen
 250:     // Specify `cacheModifier` and `evictionPolicy` explicitly in the
 251:     // assemblyFormat instead of as part of attr-dict so that they get printed
 252:     // as strings rather than opaque integers.
 253:     //
 254:     // Note there's no comma between `other` and `cacheModifier` and between
 255:     // `cacheModifier` and `evictionPolicy`.  This is due to an apparent
 256:     // limitation in the MLIR custom-format parser.  In oilist, the initial
 257:     // keywords of each clause have to be unique, so they can't be `,`.
 258:     //
 259:     // Even if we gave up on order-independence and used vanilla optional
 260:     // clauses, the format (`,` `foo` `=` $foo^)? (`,` `bar` `=` $bar^)?  will
 261:     // not match the string ", bar = 0" because after the initial comma (first
 262:     // token of the first optional clause) we expect to see "foo".
 263:     let assemblyFormat = [{
 264:       $ptr (`,` $mask^)? (`,` $other^)?
 265:       oilist(
 266:         `cacheModifier` `=` $cache |
 267:         `evictionPolicy` `=` $evict
 268:       )
 269:       attr-dict `:` type($ptr)
 270:     }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 272-273
```tablegen
 272:     let hasCanonicalizer = 1;
 273: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 275-285
```tablegen
 275: def TT_StoreOp : TT_Op<"store", [
 276:   SameLoadStoreOperandsShape,
 277:   SameLoadStoreOperandsEncoding,
 278:   DeclareOpInterfaceMethods<PredicatedOpInterface>,
 279:   TypesMatchWith<"value type matches ptr type", "ptr", "value",
 280:                  "getPointeeType($_self)">,
 281:   TypesMatchWith<"mask type matches ptr type", "ptr", "mask",
 282:                  "getI1SameShape(getPointeeType($_self))",
 283:                  "($_op.getOperands().size() <= 2) || std::equal_to<>()">
 284: ]> {
 285:     let summary = "Store through a pointer or tensor of pointers";
```
**EN:** This TableGen def record defines `TT_StoreOp` with the summary “Store through a pointer or tensor of pointers”. It is specialized from `TT_Op<"store", [`.
**CN:** 该 TableGen def 记录定义了 `TT_StoreOp`，其摘要为“Store through a pointer or tensor of pointers”。 它基于 `TT_Op<"store", [` 进一步特化。

### Lines 287-294
```tablegen
 287:     let arguments = (ins
 288:       Arg<TT_PtrLike, "", [MemWrite<GlobalMemory>]>:$ptr,
 289:       TT_Type:$value,
 290:       Optional<TT_BoolLike>:$mask,
 291:       DefaultValuedAttr<TT_CacheModifierAttr, "triton::CacheModifier::NONE">:$cache,
 292:       DefaultValuedAttr<TT_EvictionPolicyAttr, "triton::EvictionPolicy::NORMAL">:$evict,
 293:       UnitAttr:$ignore_cta
 294:     );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 296-299
```tablegen
 296:     let builders = [
 297:         // A tensor of pointers or a pointer to a scalar
 298:         OpBuilder<(ins "Value":$ptr, "Value":$value, "triton::CacheModifier":$cache, "triton::EvictionPolicy":$evict)>
 299:     ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 301-311
```tablegen
 301:     // Specify cacheModifier and evictionPolicy explicitly, instead of leaving
 302:     // them in attr-dict, because this way their values get printed as strings,
 303:     // rather than as opaque integers.
 304:     //
 305:     // Note there are no commas between mask, cacheModifier, and evictionPolicy,
 306:     // due to limitations in MLIR's asm parser.
 307:     let assemblyFormat = [{
 308:       $ptr `,` $value (`,` $mask^)?
 309:       oilist(`cacheModifier` `=` $cache | `evictionPolicy` `=` $evict)
 310:       attr-dict `:` type($ptr)
 311:     }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 313-314
```tablegen
 313:     let hasCanonicalizer = 1;
 314: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 316-329
```tablegen
 316: //
 317: // Atomic Ops
 318: //
 319: def TT_AtomicRMWOp : TT_Op<"atomic_rmw", [
 320:   SameOperandsAndResultShape,
 321:   SameOperandsAndResultEncoding,
 322:   DeclareOpInterfaceMethods<PredicatedOpInterface>,
 323:   TypesMatchWith<"ptr type matches value type", "val", "ptr",
 324:                  "getPointerTypeSameShape($_self)">,
 325:   TypesMatchWith<"mask type matches value type",
 326:                  "val", "mask", "getI1SameShape($_self)",
 327:                  "($_op.getOperands().size() <= 2) || std::equal_to<>()">
 328: ]> {
 329:     let summary = "atomic rmw";
```
**EN:** This TableGen def record defines `TT_AtomicRMWOp` with the summary “atomic rmw”. It is specialized from `TT_Op<"atomic_rmw", [`.
**CN:** 该 TableGen def 记录定义了 `TT_AtomicRMWOp`，其摘要为“atomic rmw”。 它基于 `TT_Op<"atomic_rmw", [` 进一步特化。

### Lines 331-332
```tablegen
 331:     let description = [{
 332:         load data at $ptr, do $rmw_op with $val, and store result to $ptr.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 334-335
```tablegen
 334:         return old value at $ptr
 335:     }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 337-344
```tablegen
 337:     let arguments = (ins
 338:       TT_AtomicRMWAttr:$atomic_rmw_op,
 339:       Arg<TT_PtrLike, "", [MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>:$ptr,
 340:       TT_Type:$val,
 341:       Optional<TT_BoolLike>:$mask,
 342:       TT_MemSemanticAttr:$sem,
 343:       TT_MemSyncScopeAttr:$scope
 344:     );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 346-346
```tablegen
 346:     let results = (outs TT_Type:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 348-353
```tablegen
 348:     // Explicitly list $atomic_rmw_op, $sem, and $scope rather than relying on
 349:     // attr-dict so they're printed as strings rather than opaque integers.
 350:     let assemblyFormat = [{
 351:       $atomic_rmw_op `,` $sem `,` $scope `,` $ptr `,` $val (`,` $mask^)?  attr-dict `:`
 352:       functional-type(operands, $result)
 353:     }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 355-355
```tablegen
 355: }
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 357-365
```tablegen
 357: def TT_AtomicCASOp : TT_Op<"atomic_cas", [
 358:   SameOperandsAndResultShape,
 359:   SameOperandsAndResultEncoding,
 360:   TypesMatchWith<"ptr type matches cmp type", "cmp", "ptr",
 361:                   "getPointerTypeSameShape($_self)">,
 362:   TypesMatchWith<"ptr type matches value type", "val", "ptr",
 363:                   "getPointerTypeSameShape($_self)">
 364: ]> {
 365:     let summary = "atomic cas";
```
**EN:** This TableGen def record defines `TT_AtomicCASOp` with the summary “atomic cas”. It is specialized from `TT_Op<"atomic_cas", [`.
**CN:** 该 TableGen def 记录定义了 `TT_AtomicCASOp`，其摘要为“atomic cas”。 它基于 `TT_Op<"atomic_cas", [` 进一步特化。

### Lines 367-368
```tablegen
 367:     let description = [{
 368:         compare $cmp with data $old at location $ptr,
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 370-370
```tablegen
 370:         if $old == $cmp, store $val to $ptr,
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 372-372
```tablegen
 372:         else store $old to $ptr,
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 374-375
```tablegen
 374:         return $old
 375:     }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 377-383
```tablegen
 377:     let arguments = (ins
 378:       Arg<TT_PtrLike, "", [MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>:$ptr,
 379:       TT_Type:$cmp,
 380:       TT_Type:$val,
 381:       TT_MemSemanticAttr:$sem,
 382:       TT_MemSyncScopeAttr:$scope
 383:     );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 385-385
```tablegen
 385:     let results = (outs TT_Type:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 387-393
```tablegen
 387:     // Explicitly list $sem and $scope rather than relying on attr-dict so
 388:     // they're printed as strings rather than opaque integers.
 389:     let assemblyFormat = [{
 390:       $sem `,` $scope `,` $ptr `,` $cmp `,` $val attr-dict `:`
 391:       functional-type(operands, $result)
 392:      }];
 393: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 395-401
```tablegen
 395: //
 396: // Shape Manipulation Ops
 397: //
 398: def TT_SplatOp : TT_Op<"splat", [Pure,
 399:                                  SameOperandsAndResultElementType,
 400:                                  SameOperandsAndResultEncoding]> {
 401:     let summary = "splat";
```
**EN:** This TableGen def record defines `TT_SplatOp` with the summary “splat”. It is specialized from `TT_Op<"splat", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TT_SplatOp`，其摘要为“splat”。 它基于 `TT_Op<"splat", [Pure,` 进一步特化。

### Lines 403-403
```tablegen
 403:     let arguments = (ins TT_Type:$src);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 405-405
```tablegen
 405:     let results = (outs TT_Tensor:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 407-407
```tablegen
 407:     let assemblyFormat = "$src attr-dict `:` type($src) `->` type($result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 409-410
```tablegen
 409:     let hasFolder = 1;
 410: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 412-416
```tablegen
 412: def TT_UnsplatOp : TT_Op<"unsplat", [Pure,
 413:                                      DeclareOpInterfaceMethods<InferTypeOpInterface>]> {
 414:     let summary = "convert a tensor with a single element to a scalar";
 415:     let arguments = (ins TT_Tensor:$src);
 416:     let results = (outs TT_Type:$result);
```
**EN:** This TableGen def record defines `TT_UnsplatOp` with the summary “convert a tensor with a single element to a scalar”. It is specialized from `TT_Op<"unsplat", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TT_UnsplatOp`，其摘要为“convert a tensor with a single element to a scalar”。 它基于 `TT_Op<"unsplat", [Pure,` 进一步特化。

### Lines 418-420
```tablegen
 418:     let assemblyFormat = "$src attr-dict `:` type($src)";
 419:     let hasVerifier = 1;
 420: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 422-425
```tablegen
 422: def TT_ExpandDimsOp : TT_Op<"expand_dims", [Pure,
 423:                                             DeclareOpInterfaceMethods<InferTypeOpInterface>,
 424:                                             SameOperandsAndResultElementType]> {
 425:     let summary = "expand_dims";
```
**EN:** This TableGen def record defines `TT_ExpandDimsOp` with the summary “expand_dims”. It is specialized from `TT_Op<"expand_dims", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TT_ExpandDimsOp`，其摘要为“expand_dims”。 它基于 `TT_Op<"expand_dims", [Pure,` 进一步特化。

### Lines 427-427
```tablegen
 427:     let arguments = (ins TT_Tensor:$src, I32Attr:$axis);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 429-429
```tablegen
 429:     let results = (outs TT_Tensor:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 431-431
```tablegen
 431:     let assemblyFormat = "$src attr-dict `:` type($src) `->` type($result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 433-435
```tablegen
 433:     let hasCanonicalizeMethod = 1;
 434:     let hasFolder = 1;
 435: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 437-441
```tablegen
 437: def TT_ReshapeOp : TT_Op<"reshape", [Pure,
 438:                                      SameOperandsAndResultElementType]> {
 439:     let summary = "reinterpret a tensor to a different shape. It may change elements order if the attribute is set.";
 440:     let description = [{
 441:         reinterpret a tensor to a different shape.
```
**EN:** This TableGen def record defines `TT_ReshapeOp` with the summary “reinterpret a tensor to a different shape. It may change elements order if the attribute is set.”. It is specialized from `TT_Op<"reshape", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TT_ReshapeOp`，其摘要为“reinterpret a tensor to a different shape. It may change elements order if the attribute is set.”。 它基于 `TT_Op<"reshape", [Pure,` 进一步特化。

### Lines 443-444
```tablegen
 443:         If allow_reorder is set the compiler is free to change the order of
 444:         elements to generate more efficient code.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 446-452
```tablegen
 446:         If efficient_layout is set, this is a hint that the destination layout should be kept for performance reason.
 447:         The compiler is still free to change it for better performance.
 448:     }];
 449:     let builders = [
 450:       OpBuilder<(ins "ArrayRef<int64_t>":$shape, "Value":$src,
 451:                      CArg<"bool", "false">:$allowReorder)>
 452:     ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 454-460
```tablegen
 454:     let arguments = (ins TT_Tensor:$src, UnitAttr:$allow_reorder, UnitAttr:$efficient_layout);
 455:     let results = (outs TT_Tensor:$result);
 456:     let assemblyFormat = "$src (`allow_reorder` $allow_reorder^)? (`efficient_layout` $efficient_layout^)? attr-dict `:` type($src) `->` type($result)";
 457:     let hasCanonicalizeMethod = 1;
 458:     let hasFolder = 1;
 459:     let hasVerifier = 1;
 460: }
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 462-465
```tablegen
 462: def TT_BroadcastOp : TT_Op<"broadcast", [Pure,
 463:                                          SameOperandsAndResultElementType,
 464:                                          SameOperandsAndResultEncoding]> {
 465:     let summary = "broadcast a tensor";
```
**EN:** This TableGen def record defines `TT_BroadcastOp` with the summary “broadcast a tensor”. It is specialized from `TT_Op<"broadcast", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TT_BroadcastOp`，其摘要为“broadcast a tensor”。 它基于 `TT_Op<"broadcast", [Pure,` 进一步特化。

### Lines 467-471
```tablegen
 467:     let description = [{
 468:       For a given tensor, broadcast changes one or more dimensions with size 1
 469:       to a new size, e.g. tensor<1x32x1xf32> -> tensor<2x32x4xf32>.  You cannot
 470:       change the size of a non-1 dimension.
 471:     }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 473-473
```tablegen
 473:     let arguments = (ins TT_Tensor:$src);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 475-475
```tablegen
 475:     let results = (outs TT_Tensor:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 477-477
```tablegen
 477:     let assemblyFormat = "$src attr-dict `:` type($src) `->` type($result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 479-482
```tablegen
 479:     let hasCanonicalizer = 1;
 480:     let hasFolder = 1;
 481:     let hasVerifier = 1;
 482: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 484-488
```tablegen
 484: // Cat is not pure because it may reorder elements.
 485: def TT_CatOp : TT_Op<"cat", [NoMemoryEffect,
 486:                              SameTypeOperands,
 487:                              SameOperandsAndResultElementType]> {
 488:     let summary = "concatenate 2 tensors";
```
**EN:** This TableGen def record defines `TT_CatOp` with the summary “concatenate 2 tensors”. It is specialized from `TT_Op<"cat", [NoMemoryEffect,`.
**CN:** 该 TableGen def 记录定义了 `TT_CatOp`，其摘要为“concatenate 2 tensors”。 它基于 `TT_Op<"cat", [NoMemoryEffect,` 进一步特化。

### Lines 490-490
```tablegen
 490:     let arguments = (ins TT_Tensor:$lhs, TT_Tensor:$rhs);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 492-492
```tablegen
 492:     let results = (outs TT_Tensor:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 494-494
```tablegen
 494:     let assemblyFormat = "$lhs `,` $rhs attr-dict `:` type($lhs) `->` type($result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 496-497
```tablegen
 496:     let hasVerifier = 1;
 497: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 499-504
```tablegen
 499: def TT_JoinOp : TT_Op<"join", [
 500:     Pure, SameTypeOperands]> {
 501:     let summary = "join two tensors along a new, minor dimension";
 502:     let description = [{
 503:         For example, if the two input tensors are 4x8xf32, returns a tensor of
 504:         shape 4x8x2xf32.
```
**EN:** This TableGen def record defines `TT_JoinOp` with the summary “join two tensors along a new, minor dimension”. It is specialized from `TT_Op<"join", [`.
**CN:** 该 TableGen def 记录定义了 `TT_JoinOp`，其摘要为“join two tensors along a new, minor dimension”。 它基于 `TT_Op<"join", [` 进一步特化。

### Lines 506-508
```tablegen
 506:         Because Triton tensors always have a power-of-two number of elements,
 507:         the two input tensors must have the same shape.
 508:     }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 510-517
```tablegen
 510:     let builders = [
 511:       OpBuilder<(ins "Value":$lhs, "Value":$rhs)>
 512:     ];
 513:     let arguments = (ins TT_Tensor:$lhs, TT_Tensor:$rhs);
 514:     let results = (outs TT_Tensor:$result);
 515:     let assemblyFormat = "$lhs `,` $rhs attr-dict `:` type($lhs) `->` type($result)";
 516:     let hasVerifier = 1;
 517: }
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 519-528
```tablegen
 519: def TT_SplitOp : TT_Op<"split", [
 520:   Pure,
 521:   InferTensorTypeOpWithLayoutEquivalence,
 522:   TypesMatchWith<"outLHS and outRHS types match",
 523:                   "outLHS", "outRHS", "$_self">,
 524: ]> {
 525:     let summary = "splits a tensor into two, along its last dimension";
 526:     let description = [{
 527:         The input must be a tensor whose last dimension has size 2.  Returns two
 528:         tensors, src[..., 0] and src[..., 1].
```
**EN:** This TableGen def record defines `TT_SplitOp` with the summary “splits a tensor into two, along its last dimension”. It is specialized from `TT_Op<"split", [`.
**CN:** 该 TableGen def 记录定义了 `TT_SplitOp`，其摘要为“splits a tensor into two, along its last dimension”。 它基于 `TT_Op<"split", [` 进一步特化。

### Lines 530-532
```tablegen
 530:         For example, if the input shape is 4x8x2xf32, returns two tensors of
 531:         shape 4x8xf32.
 532:     }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 534-537
```tablegen
 534:     let arguments = (ins TT_Tensor:$src);
 535:     let results = (outs TT_Tensor:$outLHS, TT_Tensor:$outRHS);
 536:     let assemblyFormat = "$src attr-dict `:` type($src) `->` type($outLHS)";
 537: }
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 539-542
```tablegen
 539: def TT_TransOp : TT_Op<"trans", [Pure,
 540:                                  TransposeOpInterface,
 541:                                  InferTensorTypeOpWithLayoutEquivalence,
 542:                                  SameOperandsAndResultElementType]> {
```
**EN:** This TableGen def record defines `TT_TransOp`. It is specialized from `TT_Op<"trans", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TT_TransOp`。 它基于 `TT_Op<"trans", [Pure,` 进一步特化。

### Lines 544-547
```tablegen
 544:     let summary = "rearrange the dimensions of a tensor";
 545:     let description = [{
 546:       For example, given a tensor x with shape [1,2,4], transpose(x) with
 547:       order=[2,0,1] rearranges the tensor to have shape [4,1,2].
```
**EN:** This block assigns the one-line summary used to describe rearrange the dimensions of a tensor in generated documentation and diagnostics.
**CN:** 该代码块设置一行摘要，用于在生成文档和诊断信息中描述 rearrange the dimensions of a tensor。

### Lines 549-551
```tablegen
 549:       Although this op is called "trans", it implements both tl.trans() and
 550:       tl.permute().  ("permute" might be a better name, but it's called "trans"
 551:       because originally it only supported 2D tensors.)
```
**EN:** This block declares or defines callable APIs such as trans and permute, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 trans and permute 等可调用 API，用来封装这里提供的核心行为。

### Lines 553-553
```tablegen
 553:       ## Implementation note on encodings:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 555-556
```tablegen
 555:       In the TritonGPU dialect (and probably others), an encoding is chosen for
 556:       this op's output so it's a nop from the perspective of code generation.
```
**EN:** This block declares or defines callable APIs such as dialect, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 dialect 等可调用 API，用来封装这里提供的核心行为。

### Lines 558-564
```tablegen
 558:       For example, suppose tensor x has an encoding such that GPU thread [i,j,k]
 559:       has a register containing element [i,j,k] of the tensor.  Now we transpose
 560:       x with order [2,1,0], i.e. we reverse the order of its dimensions.  In
 561:       TritonGPU, we will choose a layout for the output of the transpose so that
 562:       GPU thread [i,j,k] has element [k,j,i] of transpose(x).  But this is the
 563:       same element it had before!  All we've done is "rename" the element that
 564:       thread [i,j,k] has.
```
**EN:** This block declares or defines callable APIs such as transpose, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 transpose 等可调用 API，用来封装这里提供的核心行为。

### Lines 566-567
```tablegen
 566:       The "real" transpose -- i.e. moving data between GPU threads -- occurs in
 567:       convertLayout ops that appear before and/or after the operation.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 569-571
```tablegen
 569:       We do this so that you can chain multiple data-movement ops (e.g.
 570:       transpose+reshape+concat) without going to shared memory after each one.
 571:     }];
```
**EN:** This block declares or defines callable APIs such as ops, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ops 等可调用 API，用来封装这里提供的核心行为。

### Lines 573-576
```tablegen
 573:     let arguments = (
 574:       ins TT_Tensor:$src,
 575:       DenseI32ArrayAttr:$order
 576:     );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 578-578
```tablegen
 578:     let results = (outs TT_Tensor:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 580-580
```tablegen
 580:     let assemblyFormat = "$src attr-dict `:` type($src) `->` type($result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 582-584
```tablegen
 582:     let hasFolder = 1;
 583:     let hasVerifier = 1;
 584: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 586-590
```tablegen
 586: //
 587: // SPMD Ops
 588: //
 589: def TT_GetProgramIdOp : TT_Op<"get_program_id", [Pure]> {
 590:     let arguments = (ins TT_ProgramDim:$axis);
```
**EN:** This TableGen def record defines `TT_GetProgramIdOp`. It is specialized from `TT_Op<"get_program_id", [Pure]>`.
**CN:** 该 TableGen def 记录定义了 `TT_GetProgramIdOp`。 它基于 `TT_Op<"get_program_id", [Pure]>` 进一步特化。

### Lines 592-592
```tablegen
 592:     let results = (outs I32:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 594-594
```tablegen
 594:     let assemblyFormat = "$axis attr-dict `:` type($result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 596-600
```tablegen
 596:     let builders = [
 597:       OpBuilder<(ins "int":$axis), [{
 598:         build($_builder, $_state, $_builder.getI32Type(), ProgramIDDimAttr::get($_builder.getContext(), ProgramIDDim(axis)));
 599:       }]>
 600:     ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 602-607
```tablegen
 602:     let extraClassDeclaration = [{
 603:       int32_t getAxisAsInt() {
 604:         return static_cast<int32_t>(getAxis());
 605:       }
 606:     }];
 607: }
```
**EN:** This block injects extra C++ helpers into the generated class, such as getAxisAsInt and getAxis.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getAxisAsInt and getAxis。

### Lines 609-610
```tablegen
 609: def TT_GetNumProgramsOp : TT_Op<"get_num_programs", [Pure]> {
 610:     let arguments = (ins TT_ProgramDim:$axis);
```
**EN:** This TableGen def record defines `TT_GetNumProgramsOp`. It is specialized from `TT_Op<"get_num_programs", [Pure]>`.
**CN:** 该 TableGen def 记录定义了 `TT_GetNumProgramsOp`。 它基于 `TT_Op<"get_num_programs", [Pure]>` 进一步特化。

### Lines 612-612
```tablegen
 612:     let results = (outs I32:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 614-619
```tablegen
 614:     let assemblyFormat = "$axis attr-dict `:` type($result)";
 615:     let builders = [
 616:       OpBuilder<(ins "int":$axis), [{
 617:         build($_builder, $_state, $_builder.getI32Type(), ProgramIDDimAttr::get($_builder.getContext(), ProgramIDDim(axis)));
 618:       }]>
 619:     ];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 621-626
```tablegen
 621:     let extraClassDeclaration = [{
 622:       int32_t getAxisAsInt() {
 623:         return static_cast<int32_t>(getAxis());
 624:       }
 625:     }];
 626: }
```
**EN:** This block injects extra C++ helpers into the generated class, such as getAxisAsInt and getAxis.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getAxisAsInt and getAxis。

### Lines 628-636
```tablegen
 628: //
 629: // Dot Op
 630: //
 631: def TT_DotOp : TT_Op<"dot", [Pure,
 632:                              DeclareOpInterfaceMethods<InferTypeOpInterface>,
 633:                              DeclareOpInterfaceMethods<DotOpInterface>,
 634:                              TypesMatchWith<"result's type matches accumulator's type",
 635:                                             "d", "c", "$_self">]> {
 636:     let summary = "dot";
```
**EN:** This TableGen def record defines `TT_DotOp` with the summary “dot”. It is specialized from `TT_Op<"dot", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TT_DotOp`，其摘要为“dot”。 它基于 `TT_Op<"dot", [Pure,` 进一步特化。

### Lines 638-647
```tablegen
 638:     let description = [{
 639:         $d = matrix_multiply($a, $b) + $c. $inputPrecision describes how to exercise the TC
 640:         when the inputs are f32. It can be one of: tf32, tf32x3, ieee, bf16x3, bf16x6.
 641:         tf32: use TC with tf32 ops.
 642:         tf32x3: implement the 3xTF32 trick. For more info see the pass in F32DotTC.cpp
 643:         bf16x3: implement the 3xBF16 trick. For more info see the pass in F32DotTC.cpp
 644:         bf16x6: implement the 6xBF16 trick. For more info see the pass in F32DotTC.cpp
 645:         ieee: don't use TC, implement dot in software.
 646:         If the GPU does not have Tensor cores or the inputs are not f32, this flag is ignored.
 647:     }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 649-656
```tablegen
 649:     let arguments = (
 650:       ins
 651:       TT_FpIntTensor:$a,
 652:       TT_FpIntTensor:$b,
 653:       TT_FpIntTensor:$c,
 654:       DefaultValuedAttr<TT_InputPrecisionAttr, "::mlir::triton::InputPrecision::IEEE">:$inputPrecision,
 655:       DefaultValuedAttr<I32Attr, "0">:$maxNumImpreciseAcc
 656:     );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 658-658
```tablegen
 658:     let results = (outs TT_FpIntTensor:$d);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 660-667
```tablegen
 660:     // attr-dict prints enums as integers.  To get inputPrecision printed as a
 661:     // string, we need to specify it explicitly.
 662:     let assemblyFormat = [{
 663:       $a`,` $b`,` $c (`,` `inputPrecision` `=` $inputPrecision^)? attr-dict `:`
 664:       type($a) `*` type($b) `->` type($d)
 665:     }];
 666:     let hasVerifier = 1;
 667: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 670-678
```tablegen
 670: //
 671: // DotScaled Op
 672: //
 673: def TT_DotScaledOp : TT_Op<"dot_scaled", [Pure,
 674:                              AttrSizedOperandSegments,
 675:                              DeclareOpInterfaceMethods<DotOpInterface, ["verifyDims", "verifyOutputDims"]>,
 676:                              TypesMatchWith<"result's type matches accumulator's type",
 677:                                             "d", "c", "$_self">]> {
 678:     let summary = "dot_scaled";
```
**EN:** This TableGen def record defines `TT_DotScaledOp` with the summary “dot_scaled”. It is specialized from `TT_Op<"dot_scaled", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TT_DotScaledOp`，其摘要为“dot_scaled”。 它基于 `TT_Op<"dot_scaled", [Pure,` 进一步特化。

### Lines 680-683
```tablegen
 680:     let description = [{
 681:         $d = matrix_multiply(scale($a, $a_scale), scale($b, $b_scale)) + $c.
 682:         Where scale(x, s) is a function that applies the scale per block following microscaling spec.
 683:     }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 685-699
```tablegen
 685:     let arguments = (
 686:       ins
 687:       // inputs are floats if we have a type for them, otherwise (fp4),
 688:       // they are packed in pairs in an I8Tensor
 689:       RankedTensorOf<[TT_Float,I8]>:$a,
 690:       RankedTensorOf<[TT_Float,I8]>:$b,
 691:       TT_FloatTensor:$c,
 692:       Optional<RankedTensorOf<[TT_Float, I8]>>:$a_scale,
 693:       Optional<RankedTensorOf<[TT_Float, I8]>>:$b_scale,
 694:       TT_ScaleDotElemTypeAttr:$a_elem_type,
 695:       TT_ScaleDotElemTypeAttr:$b_elem_type,
 696:       BoolAttr:$fastMath,
 697:       DefaultValuedAttr<BoolAttr, "true">:$lhs_k_pack,
 698:       DefaultValuedAttr<BoolAttr, "true">:$rhs_k_pack
 699:     );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 701-701
```tablegen
 701:     let results = (outs TT_FloatTensor:$d);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 703-718
```tablegen
 703:     let assemblyFormat = [{
 704:       $a (`scale` $a_scale^)? `,` $b (`scale` $b_scale^)? `,` $c
 705:       `lhs` `=` $a_elem_type `rhs` `=` $b_elem_type attr-dict
 706:       `:` type($a) (`,` type($a_scale)^)? `*` type($b) (`,` type($b_scale)^)? `->` type($d)
 707:     }];
 708:     let extraClassDeclaration = [{
 709:       static LogicalResult deduceScaleFactor(Value lhs, Value lhsScale,
 710:                                              ScaleDotElemType lhsFormat,
 711:                                              bool lhsKPack, Value rhs,
 712:                                              Value rhsScale,
 713:                                              ScaleDotElemType rhsFormat,
 714:                                              bool rhsKPack,
 715:                                              int32_t &scaleFactor,
 716:                                              std::string &errMsg);
 717:       int32_t deduceScaleFactor();
 718:     }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 720-721
```tablegen
 720:     let hasVerifier = 1;
 721: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 723-741
```tablegen
 723: //
 724: // Reduce Op
 725: //
 726: def TT_ReduceOp: TT_Op<"reduce",
 727:                        [Pure,
 728:                         SameOperandsShape,
 729:                         SameOperandsEncoding,
 730:                         SingleBlock,
 731:                         DeclareOpInterfaceMethods<InferTypeOpInterface>]> {
 732:     let summary = "Reduction using generic combination algorithm";
 733:     let arguments = (ins Variadic<TT_Tensor>:$srcs, I32Attr:$axis);
 734:     let results = (outs Variadic<TT_Type>:$result);
 735:     let regions = (region SizedRegion<1>:$combineOp);
 736:     let hasVerifier = 1;
 737:     let hasRegionVerifier = 1;
 738:     let extraClassDeclaration = [{
 739:       llvm::SmallVector<RankedTensorType> getInputTypes();
 740:       llvm::SmallVector<Type> getElementTypes();
 741:       unsigned getNumOperands();
```
**EN:** This TableGen def record defines `TT_ReduceOp` with the summary “Reduction using generic combination algorithm”. It is specialized from `TT_Op<"reduce",`.
**CN:** 该 TableGen def 记录定义了 `TT_ReduceOp`，其摘要为“Reduction using generic combination algorithm”。 它基于 `TT_Op<"reduce",` 进一步特化。

### Lines 743-747
```tablegen
 743:       // Returns the CombineOp iff this ReduceOp's region contains only
 744:       // one CombineOp other than the return, or nullptr if not applicable.
 745:       ::mlir::Operation *getSingleCombiner();
 746:     }];
 747: }
```
**EN:** This block declares or defines callable APIs such as getSingleCombiner, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSingleCombiner 等可调用 API，用来封装这里提供的核心行为。

### Lines 749-754
```tablegen
 749: def TT_ReduceReturnOp: TT_Op<"reduce.return",
 750:                              [HasParent<"ReduceOp">, Pure, Terminator, ReturnLike]> {
 751:     let summary = "terminator for reduce operator";
 752:     let arguments = (ins Variadic<AnyType>:$result);
 753:     let assemblyFormat = "$result attr-dict `:` type($result)";
 754: }
```
**EN:** This TableGen def record defines `TT_ReduceReturnOp` with the summary “terminator for reduce operator”. It is specialized from `TT_Op<"reduce.return",`.
**CN:** 该 TableGen def 记录定义了 `TT_ReduceReturnOp`，其摘要为“terminator for reduce operator”。 它基于 `TT_Op<"reduce.return",` 进一步特化。

### Lines 756-779
```tablegen
 756: //
 757: // Scan Op
 758: //
 759: def TT_ScanOp: TT_Op<"scan",
 760:                        [Pure,
 761:                         SameOperandsAndResultEncoding,
 762:                         SameOperandsAndResultShape,
 763:                         SingleBlock,
 764:                         DeclareOpInterfaceMethods<InferTypeOpInterface>]> {
 765:     let summary = "Associative scan using generic combination algorithm";
 766:     let arguments = (ins Variadic<TT_Tensor>:$srcs, I32Attr:$axis, BoolAttr:$reverse);
 767:     let results = (outs Variadic<TT_Tensor>:$result);
 768:     let regions = (region SizedRegion<1>:$combineOp);
 769:     let builders = [
 770:         OpBuilder<(ins "ValueRange":$srcs, "int":$axis, "bool":$reverse)>,
 771:     ];
 772:     let hasVerifier = 1;
 773:     let hasRegionVerifier = 1;
 774:     let extraClassDeclaration = [{
 775:       llvm::SmallVector<RankedTensorType> getInputTypes();
 776:       llvm::SmallVector<Type> getElementTypes();
 777:       unsigned getNumOperands();
 778:     }];
 779: }
```
**EN:** This TableGen def record defines `TT_ScanOp` with the summary “Associative scan using generic combination algorithm”. It is specialized from `TT_Op<"scan",`.
**CN:** 该 TableGen def 记录定义了 `TT_ScanOp`，其摘要为“Associative scan using generic combination algorithm”。 它基于 `TT_Op<"scan",` 进一步特化。

### Lines 781-786
```tablegen
 781: def TT_ScanReturnOp: TT_Op<"scan.return",
 782:                              [HasParent<"ScanOp">, Pure, Terminator, ReturnLike]> {
 783:     let summary = "terminator for scan operator";
 784:     let arguments = (ins Variadic<AnyType>:$result);
 785:     let assemblyFormat = "$result attr-dict `:` type($result)";
 786: }
```
**EN:** This TableGen def record defines `TT_ScanReturnOp` with the summary “terminator for scan operator”. It is specialized from `TT_Op<"scan.return",`.
**CN:** 该 TableGen def 记录定义了 `TT_ScanReturnOp`，其摘要为“terminator for scan operator”。 它基于 `TT_Op<"scan.return",` 进一步特化。

### Lines 788-800
```tablegen
 788: //
 789: // Map Elementwise op
 790: //
 791: def TT_MapElementwiseOp: TT_Op<"map_elementwise", [SameOperandsAndResultEncoding,
 792:                                                    SameOperandsAndResultShape,
 793:                                                    RecursiveMemoryEffects]> {
 794:     let summary = "Map a scalar subregion over a tensor";
 795:     let arguments = (ins Variadic<TT_Tensor>:$srcs, I32Attr:$pack);
 796:     let results = (outs Variadic<TT_Tensor>:$result);
 797:     let regions = (region AnyRegion:$scalarOp);
 798:     let hasVerifier = 1;
 799:     let hasRegionVerifier = 1;
 800: }
```
**EN:** This TableGen def record defines `TT_MapElementwiseOp` with the summary “Map a scalar subregion over a tensor”. It is specialized from `TT_Op<"map_elementwise", [SameOperandsAndResultEncoding,`.
**CN:** 该 TableGen def 记录定义了 `TT_MapElementwiseOp`，其摘要为“Map a scalar subregion over a tensor”。 它基于 `TT_Op<"map_elementwise", [SameOperandsAndResultEncoding,` 进一步特化。

### Lines 802-807
```tablegen
 802: def TT_MapElementwiseReturnOp: TT_Op<"map_elementwise.return",
 803:                                [HasParent<"MapElementwiseOp">, Pure, Terminator, ReturnLike]> {
 804:     let summary = "terminator for map elementwise operator";
 805:     let arguments = (ins Variadic<AnyType>:$result);
 806:     let assemblyFormat = "attr-dict ($result^ `:` type($result))?";
 807: }
```
**EN:** This TableGen def record defines `TT_MapElementwiseReturnOp` with the summary “terminator for map elementwise operator”. It is specialized from `TT_Op<"map_elementwise.return",`.
**CN:** 该 TableGen def 记录定义了 `TT_MapElementwiseReturnOp`，其摘要为“terminator for map elementwise operator”。 它基于 `TT_Op<"map_elementwise.return",` 进一步特化。

### Lines 809-816
```tablegen
 809: //
 810: // External Elementwise op
 811: //
 812: def TT_ExternElementwiseOp : TT_Op<"extern_elementwise", [Elementwise,
 813:                                                           SameOperandsAndResultEncoding,
 814:                                                           SameVariadicOperandSize,
 815:                                                           DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
 816:                                                           ConditionallySpeculatable]> {
```
**EN:** This TableGen def record defines `TT_ExternElementwiseOp`. It is specialized from `TT_Op<"extern_elementwise", [Elementwise,`.
**CN:** 该 TableGen def 记录定义了 `TT_ExternElementwiseOp`。 它基于 `TT_Op<"extern_elementwise", [Elementwise,` 进一步特化。

### Lines 818-821
```tablegen
 818:     let description = [{
 819:         call an external function $symbol implemented in $libpath/$libname with $args
 820:         return $libpath/$libname:$symbol($args...)
 821:     }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 823-823
```tablegen
 823:     let arguments = (ins Variadic<TT_Type>:$srcs, StrAttr:$libname, StrAttr:$libpath, StrAttr:$symbol, BoolAttr:$pure);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 825-825
```tablegen
 825:     let results = (outs TT_Type:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 827-827
```tablegen
 827:     let assemblyFormat = "operands attr-dict `:` functional-type(operands, $result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 829-832
```tablegen
 829:     let extraClassDeclaration = [{
 830:       // Interface method for ConditionallySpeculatable.
 831:       Speculation::Speculatability getSpeculatability();
 832:     }];
```
**EN:** This block injects extra C++ helpers into the generated class, such as getSpeculatability.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getSpeculatability。

### Lines 834-834
```tablegen
 834: }
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 836-840
```tablegen
 836: //
 837: // Make Range Op
 838: //
 839: def TT_MakeRangeOp : TT_Op<"make_range", [Pure]> {
 840:     let summary = "make range";
```
**EN:** This TableGen def record defines `TT_MakeRangeOp` with the summary “make range”. It is specialized from `TT_Op<"make_range", [Pure]>`.
**CN:** 该 TableGen def 记录定义了 `TT_MakeRangeOp`，其摘要为“make range”。 它基于 `TT_Op<"make_range", [Pure]>` 进一步特化。

### Lines 842-843
```tablegen
 842:     let description = [{
 843:         Returns an 1D int32 tensor.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 845-846
```tablegen
 845:         Values span from $start to $end (exclusive), with step = 1
 846:     }];
```
**EN:** This block declares or defines callable APIs such as end, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 end 等可调用 API，用来封装这里提供的核心行为。

### Lines 848-852
```tablegen
 848:     // WARNING: MLIR generates getStart()/getEnd() functions which return
 849:     // uint32_t, even though these arguments are to be interpreted as *signed*
 850:     // int32 values.  If this matters, use get{Start,End}Attr().getInt(), which
 851:     // return int64_t.
 852:     let arguments = (ins I32Attr:$start, I32Attr:$end);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 854-854
```tablegen
 854:     let results = (outs TT_IntTensor:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 856-856
```tablegen
 856:     let assemblyFormat = "attr-dict `:` type($result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 858-860
```tablegen
 858:     let hasFolder = 1;
 859:     let hasVerifier = 1;
 860: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 862-873
```tablegen
 862: //
 863: // ElementwiseInlineAsm Op
 864: //
 865: def TT_ElementwiseInlineAsmOp : TT_Op<"elementwise_inline_asm", [
 866:   Elementwise,
 867:   SameOperandsAndResultEncoding,
 868:   DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
 869:   DeclareOpInterfaceMethods<ConditionallySpeculatable>
 870: ]> {
 871:   let summary = "inline assembly applying an elementwise operation to a group of packed elements.";
 872:   let description = [{
 873:     Runs an inline asm block to generate one or more tensors.
```
**EN:** This TableGen def record defines `TT_ElementwiseInlineAsmOp` with the summary “inline assembly applying an elementwise operation to a group of packed elements.”. It is specialized from `TT_Op<"elementwise_inline_asm", [`.
**CN:** 该 TableGen def 记录定义了 `TT_ElementwiseInlineAsmOp`，其摘要为“inline assembly applying an elementwise operation to a group of packed elements.”。 它基于 `TT_Op<"elementwise_inline_asm", [` 进一步特化。

### Lines 875-877
```tablegen
 875:     The asm block is given `packed_element` elements at a time.  Exactly which
 876:     elems it receives is unspecified.
 877:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 879-880
```tablegen
 879:   let arguments = (ins StrAttr:$asm_string, StrAttr:$constraints, BoolAttr:$pure, I32Attr:$packed_element, Variadic<AnyTypeOf<[TT_Type]>>:$args);
 880:   let results = (outs Variadic<TT_Type>:$result);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 882-884
```tablegen
 882:   let assemblyFormat = [{
 883:     $asm_string attr-dict ($args^ `:` type($args))? `->` type($result)
 884:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 886-887
```tablegen
 886:   let hasVerifier = 1;
 887: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 889-901
```tablegen
 889: //
 890: // Histogram Op
 891: //
 892: def TT_HistogramOp : TT_Op<"histogram", [Pure,
 893:     TypesMatchWith<"mask type matches src type",
 894:                  "src", "mask", "getI1SameShape($_self)",
 895:                  "($_op.getOperands().size() <= 1) || std::equal_to<>()">]> {
 896:   let summary = "return a histogram of the inputs.";
 897:   let description = [{
 898:     Return the histogram of the input tensor. The number of bins is equal to
 899:     the dimension of the output tensor. Each bins has a width of 1 and bins
 900:     start at 0.
 901:   }];
```
**EN:** This TableGen def record defines `TT_HistogramOp` with the summary “return a histogram of the inputs.”. It is specialized from `TT_Op<"histogram", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TT_HistogramOp`，其摘要为“return a histogram of the inputs.”。 它基于 `TT_Op<"histogram", [Pure,` 进一步特化。

### Lines 903-904
```tablegen
 903:   let arguments = (ins TT_IntTensor:$src,
 904:     Optional<TT_BoolLike>:$mask);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 906-906
```tablegen
 906:   let results = (outs TT_IntTensor:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 908-911
```tablegen
 908:   let assemblyFormat = [{
 909:     $src (`,` $mask^)? attr-dict `:` type($src) `->` type($result)
 910:   }];
 911: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 913-925
```tablegen
 913: //
 914: // Gather Op
 915: //
 916: def TT_GatherOp : TT_Op<"gather", [Pure,
 917:     DeclareOpInterfaceMethods<InferTypeOpInterface>]> {
 918:   let summary = "local gather operation";
 919:   let description = [{
 920:     Gather elements from the input tensor using the indices tensor along a
 921:     single specified axis. The output tensor has the same shape as the indices
 922:     tensor. The input and indices tensors must have the same number of
 923:     dimension, and each dimension of the indices tensor that is not the gather
 924:     dimension cannot be greater than the corresponding dimension in the input
 925:     tensor.
```
**EN:** This TableGen def record defines `TT_GatherOp` with the summary “local gather operation”. It is specialized from `TT_Op<"gather", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TT_GatherOp`，其摘要为“local gather operation”。 它基于 `TT_Op<"gather", [Pure,` 进一步特化。

### Lines 927-930
```tablegen
 927:     The `efficient_layout` attribute is set when the compiler has determined an
 928:     optimized layout for the operation, indicating that it should not be
 929:     changed.
 930:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 932-938
```tablegen
 932:   let arguments = (ins
 933:     TT_Tensor:$src,
 934:     TT_IntTensor:$indices,
 935:     I32Attr:$axis,
 936:     UnitAttr:$efficient_layout
 937:   );
 938:   let results = (outs TT_Tensor:$result);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 940-943
```tablegen
 940:   let assemblyFormat = [{
 941:     $src `[` $indices `]` attr-dict `:`
 942:     functional-type(operands, results)
 943:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 945-946
```tablegen
 945:   let hasVerifier = 1;
 946: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 948-967
```tablegen
 948: //
 949: // Print Op
 950: //
 951: def TT_PrintOp : TT_Op<"print", [SameVariadicOperandSize, MemoryEffects<[MemWrite<GlobalMemory>]>]> {
 952:   let arguments = (
 953:     ins
 954:     StrAttr:$prefix,
 955:     BoolAttr:$hex,
 956:     Variadic<AnyTypeOf<[TT_Type]>>:$args,
 957:     DenseI32ArrayAttr:$isSigned
 958:   );
 959:   let summary = "Device-side print, as in CUDA for debugging";
 960:   let description = [{
 961:     `tt.print` takes a literal string prefix and an arbitrary number of scalar or tensor arguments that should be printed.
 962:     format are generated automatically from the arguments.
 963:   }];
 964:   let assemblyFormat = [{
 965:     $prefix attr-dict (`:` $args^ `:` type($args))?
 966:   }];
 967: }
```
**EN:** This TableGen def record defines `TT_PrintOp` with the summary “Device-side print, as in CUDA for debugging”. It is specialized from `TT_Op<"print", [SameVariadicOperandSize, MemoryEffects<[MemWrite<GlobalMemory>]>]>`.
**CN:** 该 TableGen def 记录定义了 `TT_PrintOp`，其摘要为“Device-side print, as in CUDA for debugging”。 它基于 `TT_Op<"print", [SameVariadicOperandSize, MemoryEffects<[MemWrite<GlobalMemory>]>]>` 进一步特化。

### Lines 969-980
```tablegen
 969: //
 970: // Assert Op
 971: //
 972: def TT_AssertOp : TT_Op<"assert", [MemoryEffects<[MemWrite<GlobalMemory>]>]> {
 973:   let summary = "Device-side assert, as in CUDA for correctness checking";
 974:   let description = [{
 975:     `tt.assert` takes a condition tensor and a message string.
 976:     If the condition is false, the message is printed, and the program is aborted.
 977:   }];
 978:   let arguments = (ins AnyTypeOf<[I1, I1Tensor]>:$condition, StrAttr:$message);
 979:   let assemblyFormat = "$condition `,` $message attr-dict `:` type($condition)";
 980: }
```
**EN:** This TableGen def record defines `TT_AssertOp` with the summary “Device-side assert, as in CUDA for correctness checking”. It is specialized from `TT_Op<"assert", [MemoryEffects<[MemWrite<GlobalMemory>]>]>`.
**CN:** 该 TableGen def 记录定义了 `TT_AssertOp`，其摘要为“Device-side assert, as in CUDA for correctness checking”。 它基于 `TT_Op<"assert", [MemoryEffects<[MemWrite<GlobalMemory>]>]>` 进一步特化。

### Lines 982-989
```tablegen
 982: //
 983: // Make Tensor Descriptor Op
 984: //
 985: def TT_MakeTensorDescOp : TT_Op<"make_tensor_descriptor", [
 986:     Pure,
 987:     SameVariadicOperandSize,
 988: ]> {
 989:   let summary = "Make a tensor descriptor type with meta information of the parent tensor and block size";
```
**EN:** This TableGen def record defines `TT_MakeTensorDescOp` with the summary “Make a tensor descriptor type with meta information of the parent tensor and block size”. It is specialized from `TT_Op<"make_tensor_descriptor", [`.
**CN:** 该 TableGen def 记录定义了 `TT_MakeTensorDescOp`，其摘要为“Make a tensor descriptor type with meta information of the parent tensor and block size”。 它基于 `TT_Op<"make_tensor_descriptor", [` 进一步特化。

### Lines 991-994
```tablegen
 991:   let description = [{
 992:       `tt.make_tensor_descriptor` takes both meta information of the parent tensor and the block size,
 993:       and returns a descriptor object which can be used to load/store from the tensor in global memory.
 994:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 996-1001
```tablegen
 996:   let arguments = (ins
 997:     TT_Ptr:$base,
 998:     Variadic<I32>:$shape,
 999:     Variadic<I64>:$strides,
1000:     DefaultValuedAttr<TT_PaddingOptionAttr, "::mlir::triton::PaddingOption::PAD_ZERO">:$padding
1001:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 1003-1003
```tablegen
1003:   let results = (outs TT_TensorDescType:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 1005-1005
```tablegen
1005:   let assemblyFormat = "$base `,` `[` $shape `]` `,` `[` $strides `]` attr-dict `:` type($base) `,` type($result)";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1007-1010
```tablegen
1007:   let builders = [
1008:     OpBuilder<(ins "Value":$base, "ValueRange":$shape, "ValueRange":$strides, "ArrayRef<int32_t>":$blockShape, "bool":$isSignedInteger,
1009:     "triton::PaddingOption":$padding)>
1010:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 1012-1012
```tablegen
1012: }
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 1014-1026
```tablegen
1014: // The following ops, including `call`, `func`, and `return` are copied and modified from
1015: // https://github.com/llvm/llvm-project/blob/main/mlir/include/mlir/Dialect/Func/IR/FuncOps.td
1016: // We could revert it back once MLIR has a better inliner interface.
1017: //
1018: // Function Ops
1019: //
1020: def CallOp : TT_Op<"call", [CallOpInterface, /*MemRefsNormalizable, */DeclareOpInterfaceMethods<SymbolUserOpInterface>]> {
1021:   let summary = "call operation";
1022:   let description = [{
1023:     The `tt.call` operation represents a direct call to a function that is
1024:     within the same symbol scope as the call. The operands and result types of
1025:     the call must match the specified function type. The callee is encoded as a
1026:     symbol reference attribute named "callee".
```
**EN:** This TableGen def record defines `CallOp` with the summary “call operation”. It is specialized from `TT_Op<"call", [CallOpInterface, /*MemRefsNormalizable, */DeclareOpInterfaceMethods<SymbolUserOpInterface>]>`.
**CN:** 该 TableGen def 记录定义了 `CallOp`，其摘要为“call operation”。 它基于 `TT_Op<"call", [CallOpInterface, /*MemRefsNormalizable, */DeclareOpInterfaceMethods<SymbolUserOpInterface>]>` 进一步特化。

### Lines 1028-1028
```tablegen
1028:     Example:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1030-1033
```tablegen
1030:     ```mlir
1031:     %2 = tt.call @my_add(%0, %1) : (f32, f32) -> f32
1032:     ```
1033:   }];
```
**EN:** This block declares or defines callable APIs such as my_add, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 my_add 等可调用 API，用来封装这里提供的核心行为。

### Lines 1035-1039
```tablegen
1035:   let arguments = (ins FlatSymbolRefAttr:$callee,
1036:                    Variadic<AnyType>:$operands,
1037:                    OptionalAttr<DictArrayAttr>:$arg_attrs,
1038:                    OptionalAttr<DictArrayAttr>:$res_attrs);
1039:   let results = (outs Variadic<AnyType>);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 1041-1061
```tablegen
1041:   let builders = [
1042:     OpBuilder<(ins "FuncOp":$callee, CArg<"ValueRange", "{}">:$operands), [{
1043:       $_state.addOperands(operands);
1044:       $_state.addAttribute("callee", SymbolRefAttr::get(callee));
1045:       $_state.addTypes(callee.getFunctionType().getResults());
1046:     }]>,
1047:     OpBuilder<(ins "SymbolRefAttr":$callee, "TypeRange":$results,
1048:       CArg<"ValueRange", "{}">:$operands), [{
1049:       $_state.addOperands(operands);
1050:       $_state.addAttribute("callee", callee);
1051:       $_state.addTypes(results);
1052:     }]>,
1053:     OpBuilder<(ins "StringAttr":$callee, "TypeRange":$results,
1054:       CArg<"ValueRange", "{}">:$operands), [{
1055:       build($_builder, $_state, SymbolRefAttr::get(callee), results, operands);
1056:     }]>,
1057:     OpBuilder<(ins "StringRef":$callee, "TypeRange":$results,
1058:       CArg<"ValueRange", "{}">:$operands), [{
1059:       build($_builder, $_state, StringAttr::get($_builder.getContext(), callee),
1060:             results, operands);
1061:     }]>];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 1063-1066
```tablegen
1063:   let extraClassDeclaration = [{
1064:     FunctionType getCalleeType() {
1065:       return FunctionType::get(getContext(), getOperandTypes(), getResultTypes());
1066:     }
```
**EN:** This block injects extra C++ helpers into the generated class, such as getCalleeType, get, getContext, getOperandTypes, and getResultTypes.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getCalleeType, get, getContext, getOperandTypes, and getResultTypes。

### Lines 1068-1071
```tablegen
1068:     /// Get the argument operands to the called function.
1069:     operand_range getArgOperands() {
1070:       return {arg_operand_begin(), arg_operand_end()};
1071:     }
```
**EN:** This block declares or defines callable APIs such as getArgOperands, arg_operand_begin, and arg_operand_end, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getArgOperands, arg_operand_begin, and arg_operand_end 等可调用 API，用来封装这里提供的核心行为。

### Lines 1073-1074
```tablegen
1073:     operand_iterator arg_operand_begin() { return operand_begin(); }
1074:     operand_iterator arg_operand_end() { return operand_end(); }
```
**EN:** This block declares or defines callable APIs such as arg_operand_begin, operand_begin, arg_operand_end, and operand_end, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 arg_operand_begin, operand_begin, arg_operand_end, and operand_end 等可调用 API，用来封装这里提供的核心行为。

### Lines 1076-1079
```tablegen
1076:     /// Return the callee of this operation.
1077:     CallInterfaceCallable getCallableForCallee() {
1078:       return (*this)->getAttrOfType<SymbolRefAttr>("callee");
1079:     }
```
**EN:** This block declares or defines callable APIs such as getCallableForCallee, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCallableForCallee 等可调用 API，用来封装这里提供的核心行为。

### Lines 1081-1084
```tablegen
1081:     /// Set the callee for this operation.
1082:     void setCalleeFromCallable(CallInterfaceCallable callee) {
1083:       (*this)->setAttr("callee", cast<SymbolRefAttr>(callee));
1084:     }
```
**EN:** This block declares or defines callable APIs such as setCalleeFromCallable and setAttr, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setCalleeFromCallable and setAttr 等可调用 API，用来封装这里提供的核心行为。

### Lines 1086-1089
```tablegen
1086:     // Required by CallOpInterface.
1087:     MutableOperandRange getArgOperandsMutable() {
1088:       return getOperandsMutable();
1089:     }
```
**EN:** This block declares or defines callable APIs such as getArgOperandsMutable and getOperandsMutable, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getArgOperandsMutable and getOperandsMutable 等可调用 API，用来封装这里提供的核心行为。

### Lines 1091-1091
```tablegen
1091:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1093-1096
```tablegen
1093:   let assemblyFormat = [{
1094:     $callee `(` $operands `)` attr-dict `:` functional-type($operands, results)
1095:   }];
1096: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1098-1113
```tablegen
1098: def FuncOp : TT_Op<"func", [
1099:     AffineScope, AutomaticAllocationScope, CallableOpInterface,
1100:     FunctionOpInterface, IsolatedFromAbove, OpAsmOpInterface,
1101:     HasParent<"ModuleOp">
1102: ]> {
1103:   let summary = "An operation with a name containing a single `SSACFG` region";
1104:   let description = [{
1105:     Operations within the function cannot implicitly capture values defined
1106:     outside of the function, i.e. Functions are `IsolatedFromAbove`. All
1107:     external references must use function arguments or attributes that establish
1108:     a symbolic connection (e.g. symbols referenced by name via a string
1109:     attribute like SymbolRefAttr). An external function declaration (used when
1110:     referring to a function declared in some other module) has no body. While
1111:     the MLIR textual form provides a nice inline syntax for function arguments,
1112:     they are internally represented as “block arguments” to the first block in
1113:     the region.
```
**EN:** This TableGen def record defines `FuncOp` with the summary “An operation with a name containing a single `SSACFG` region”. It is specialized from `TT_Op<"func", [`.
**CN:** 该 TableGen def 记录定义了 `FuncOp`，其摘要为“An operation with a name containing a single `SSACFG` region”。 它基于 `TT_Op<"func", [` 进一步特化。

### Lines 1115-1116
```tablegen
1115:     Only dialect attribute names may be specified in the attribute dictionaries
1116:     for function arguments, results, or the function itself.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1118-1118
```tablegen
1118:     Example:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1120-1123
```tablegen
1120:     ```mlir
1121:     // External function definitions.
1122:     tt.func @abort()
1123:     tt.func @scribble(i32, i64, memref<? x 128 x f32, #layout_map0>) -> f64
```
**EN:** This block declares or defines callable APIs such as abort and scribble, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 abort and scribble 等可调用 API，用来封装这里提供的核心行为。

### Lines 1125-1129
```tablegen
1125:     // A function that returns its argument twice:
1126:     tt.func @count(%x: i64) -> (i64, i64)
1127:       attributes {fruit: "banana"} {
1128:       return %x, %x: i64, i64
1129:     }
```
**EN:** This block declares or defines callable APIs such as count, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 count 等可调用 API，用来封装这里提供的核心行为。

### Lines 1131-1132
```tablegen
1131:     // A function with an argument attribute
1132:     tt.func @example_fn_arg(%x: i32 {swift.self = unit})
```
**EN:** This block declares or defines callable APIs such as example_fn_arg, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 example_fn_arg 等可调用 API，用来封装这里提供的核心行为。

### Lines 1134-1135
```tablegen
1134:     // A function with a result attribute
1135:     tt.func @example_fn_result() -> (f64 {dialectName.attrName = 0 : i64})
```
**EN:** This block declares or defines callable APIs such as example_fn_result, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 example_fn_result 等可调用 API，用来封装这里提供的核心行为。

### Lines 1137-1140
```tablegen
1137:     // A function with an attribute
1138:     tt.func @example_fn_attr() attributes {dialectName.attrName = false}
1139:     ```
1140:   }];
```
**EN:** This block declares or defines callable APIs such as example_fn_attr, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 example_fn_attr 等可调用 API，用来封装这里提供的核心行为。

### Lines 1142-1147
```tablegen
1142:   let arguments = (ins SymbolNameAttr:$sym_name,
1143:                        TypeAttrOf<FunctionType>:$function_type,
1144:                        OptionalAttr<StrAttr>:$sym_visibility,
1145:                        OptionalAttr<DictArrayAttr>:$arg_attrs,
1146:                        OptionalAttr<DictArrayAttr>:$res_attrs);
1147:   let regions = (region AnyRegion:$body);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 1149-1157
```tablegen
1149:   let builders = [OpBuilder<(ins
1150:     "StringRef":$name, "FunctionType":$type,
1151:     CArg<"ArrayRef<NamedAttribute>", "{}">:$attrs,
1152:     CArg<"ArrayRef<DictionaryAttr>", "{}">:$argAttrs)
1153:   >];
1154:   let extraClassDeclaration = [{
1155:     //===------------------------------------------------------------------===//
1156:     // CallableOpInterface
1157:     //===------------------------------------------------------------------===//
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 1159-1162
```tablegen
1159:     /// Returns the region on the current operation that is callable. This may
1160:     /// return null in the case of an external callable object, e.g. an external
1161:     /// function.
1162:     ::mlir::Region *getCallableRegion() { return isExternal() ? nullptr : &getBody(); }
```
**EN:** This block declares or defines callable APIs such as getCallableRegion, isExternal, and getBody, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCallableRegion, isExternal, and getBody 等可调用 API，用来封装这里提供的核心行为。

### Lines 1164-1166
```tablegen
1164:     /// Returns the results types that the callable region produces when
1165:     /// executed.
1166:     ArrayRef<Type> getCallableResults() { return getFunctionType().getResults(); }
```
**EN:** This block declares or defines callable APIs such as getCallableResults, getFunctionType, and getResults, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCallableResults, getFunctionType, and getResults 等可调用 API，用来封装这里提供的核心行为。

### Lines 1168-1172
```tablegen
1168:     /// Returns the argument attributes for all callable region arguments or
1169:     /// null if there are none.
1170:     ::mlir::ArrayAttr getCallableArgAttrs() {
1171:       return getArgAttrs().value_or(nullptr);
1172:     }
```
**EN:** This block declares or defines callable APIs such as getCallableArgAttrs, getArgAttrs, and value_or, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCallableArgAttrs, getArgAttrs, and value_or 等可调用 API，用来封装这里提供的核心行为。

### Lines 1174-1178
```tablegen
1174:     /// Returns the result attributes for all callable region results or
1175:     /// null if there are none.
1176:     ::mlir::ArrayAttr getCallableResAttrs() {
1177:       return getResAttrs().value_or(nullptr);
1178:     }
```
**EN:** This block declares or defines callable APIs such as getCallableResAttrs, getResAttrs, and value_or, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCallableResAttrs, getResAttrs, and value_or 等可调用 API，用来封装这里提供的核心行为。

### Lines 1180-1182
```tablegen
1180:     //===------------------------------------------------------------------===//
1181:     // FunctionOpInterface Methods
1182:     //===------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===------------------------------------------------------------------===// FunctionOpInterface Methods ===------------------------------------------------------------------===//.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 1184-1185
```tablegen
1184:     /// Returns the argument types of this function.
1185:     ArrayRef<Type> getArgumentTypes() { return getFunctionType().getInputs(); }
```
**EN:** This block declares or defines callable APIs such as getArgumentTypes, getFunctionType, and getInputs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getArgumentTypes, getFunctionType, and getInputs 等可调用 API，用来封装这里提供的核心行为。

### Lines 1187-1188
```tablegen
1187:     /// Returns the result types of this function.
1188:     ArrayRef<Type> getResultTypes() { return getFunctionType().getResults(); }
```
**EN:** This block declares or defines callable APIs such as getResultTypes, getFunctionType, and getResults, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getResultTypes, getFunctionType, and getResults 等可调用 API，用来封装这里提供的核心行为。

### Lines 1190-1192
```tablegen
1190:     //===------------------------------------------------------------------===//
1191:     // SymbolOpInterface Methods
1192:     //===------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===------------------------------------------------------------------===// SymbolOpInterface Methods ===------------------------------------------------------------------===//.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 1194-1197
```tablegen
1194:     bool isDeclaration() { return isExternal(); }
1195:   }];
1196:   let hasCustomAssemblyFormat = 1;
1197: }
```
**EN:** This block declares or defines callable APIs such as isDeclaration and isExternal, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isDeclaration and isExternal 等可调用 API，用来封装这里提供的核心行为。

### Lines 1199-1205
```tablegen
1199: def ReturnOp : TT_Op<"return", [Pure, HasParent<"FuncOp">, /*MemRefsNormalizable, */ReturnLike, Terminator]> {
1200:   let summary = "Function return operation";
1201:   let description = [{
1202:     The `tt.return` operation represents a return operation within a function.
1203:     The operation takes variable number of operands and produces no results.
1204:     The operand number and types must match the signature of the function
1205:     that contains the operation.
```
**EN:** This TableGen def record defines `ReturnOp` with the summary “Function return operation”. It is specialized from `TT_Op<"return", [Pure, HasParent<"FuncOp">, /*MemRefsNormalizable, */ReturnLike, Terminator]>`.
**CN:** 该 TableGen def 记录定义了 `ReturnOp`，其摘要为“Function return operation”。 它基于 `TT_Op<"return", [Pure, HasParent<"FuncOp">, /*MemRefsNormalizable, */ReturnLike, Terminator]>` 进一步特化。

### Lines 1207-1207
```tablegen
1207:     Example:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1209-1215
```tablegen
1209:     ```mlir
1210:     tt.func @foo() : (i32, f8) {
1211:       ...
1212:       tt.return %0, %1 : i32, f8
1213:     }
1214:     ```
1215:   }];
```
**EN:** This block declares or defines callable APIs such as foo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 foo 等可调用 API，用来封装这里提供的核心行为。

### Lines 1217-1217
```tablegen
1217:   let arguments = (ins Variadic<AnyType>:$srcs);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 1219-1221
```tablegen
1219:   let builders = [OpBuilder<(ins), [{
1220:     build($_builder, $_state, mlir::ValueRange());
1221:   }]>];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 1223-1225
```tablegen
1223:   let assemblyFormat = "attr-dict ($srcs^ `:` type($srcs))?";
1224:   let hasVerifier = 1;
1225: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1228-1240
```tablegen
1228: def TT_DescriptorLoadOp : TT_Op<"descriptor_load", [TT_DescriptorLoadLikeOpInterface]> {
1229:   let summary = "Load from descriptor";
1230:   let description = [{
1231:     This operation will be lowered to Nvidia TMA load operation on targets supporting it.
1232:     `desc` is a tensor descriptor object.
1233:     The destination tensor type and shape must match the descriptor otherwise the result is undefined.
1234:   }];
1235:   let arguments = (ins
1236:     Arg<TT_TensorDescType, "", [MemRead<GlobalMemory>]>:$desc,
1237:     Variadic<I32>:$indices,
1238:     DefaultValuedAttr<TT_CacheModifierAttr, "::mlir::triton::CacheModifier::NONE">:$cache,
1239:     DefaultValuedAttr<TT_EvictionPolicyAttr, "::mlir::triton::EvictionPolicy::NORMAL">:$evict
1240:   );
```
**EN:** This TableGen def record defines `TT_DescriptorLoadOp` with the summary “Load from descriptor”. It is specialized from `TT_Op<"descriptor_load", [TT_DescriptorLoadLikeOpInterface]>`.
**CN:** 该 TableGen def 记录定义了 `TT_DescriptorLoadOp`，其摘要为“Load from descriptor”。 它基于 `TT_Op<"descriptor_load", [TT_DescriptorLoadLikeOpInterface]>` 进一步特化。

### Lines 1242-1242
```tablegen
1242:   let results = (outs TT_Tensor:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 1244-1251
```tablegen
1244:   let assemblyFormat = [{
1245:     $desc `[` $indices `]`
1246:     oilist(
1247:       `cacheModifier` `=` $cache |
1248:       `evictionPolicy` `=` $evict
1249:     )
1250:     attr-dict `:` qualified(type($desc)) `->` type($result)
1251:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1253-1254
```tablegen
1253:   let hasVerifier = 1;
1254: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 1256-1267
```tablegen
1256: def TT_DescriptorStoreOp : TT_Op<"descriptor_store", [TT_DescriptorStoreLikeOpInterface]> {
1257:   let summary = "store value based on descriptor";
1258:   let description = [{
1259:     This operation will be lowered to Nvidia TMA store operation on targets supporting it.
1260:     `desc` is a tensor descriptor object.
1261:     The shape and types of `src` must match the descriptor otherwise the result is undefined.
1262:   }];
1263:   let arguments = (ins
1264:     Arg<TT_TensorDescType, "", [MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>:$desc,
1265:     TT_Tensor:$src,
1266:     Variadic<I32>:$indices
1267:   );
```
**EN:** This TableGen def record defines `TT_DescriptorStoreOp` with the summary “store value based on descriptor”. It is specialized from `TT_Op<"descriptor_store", [TT_DescriptorStoreLikeOpInterface]>`.
**CN:** 该 TableGen def 记录定义了 `TT_DescriptorStoreOp`，其摘要为“store value based on descriptor”。 它基于 `TT_Op<"descriptor_store", [TT_DescriptorStoreLikeOpInterface]>` 进一步特化。

### Lines 1269-1274
```tablegen
1269:   let assemblyFormat = [{
1270:     $desc `[` $indices `]` `,` $src
1271:     attr-dict `:` qualified(type($desc)) `,` type($src)
1272:   }];
1273:   let hasVerifier = 1;
1274: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1276-1288
```tablegen
1276: def TT_DescriptorReduceOp : TT_Op<"descriptor_reduce", [TT_DescriptorStoreLikeOpInterface]> {
1277:   let summary = "performs a reducing store operation based on a descriptor";
1278:   let description = [{
1279:     This operation will be lowered to Nvidia TMA store operation on targets supporting it.
1280:     `desc` is a tensor descriptor object.
1281:     The shape and types of `src` must match the descriptor otherwise the result is undefined.
1282:   }];
1283:   let arguments = (ins
1284:     TT_DescriptorReduceKindAttr:$kind,
1285:     Arg<TT_TensorDescType, "", [MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>:$desc,
1286:     TT_Tensor:$src,
1287:     Variadic<I32>:$indices
1288:   );
```
**EN:** This TableGen def record defines `TT_DescriptorReduceOp` with the summary “performs a reducing store operation based on a descriptor”. It is specialized from `TT_Op<"descriptor_reduce", [TT_DescriptorStoreLikeOpInterface]>`.
**CN:** 该 TableGen def 记录定义了 `TT_DescriptorReduceOp`，其摘要为“performs a reducing store operation based on a descriptor”。 它基于 `TT_Op<"descriptor_reduce", [TT_DescriptorStoreLikeOpInterface]>` 进一步特化。

### Lines 1290-1295
```tablegen
1290:   let assemblyFormat = [{
1291:     $kind `,` $desc `[` $indices `]` `,` $src
1292:     attr-dict `:` qualified(type($desc)) `,` type($src)
1293:   }];
1294:   let hasVerifier = 1;
1295: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1297-1301
```tablegen
1297: def TT_DescriptorGatherOp : TT_Op<"descriptor_gather", [TT_DescriptorLoadLikeOpInterface]> {
1298:   let summary = "gather multiple rows from a descriptor into a single tensor";
1299:   let description = [{
1300:     The `tt.descriptor_gather` op will be lowered to NVIDIA TMA
1301:     gather operations on targets that support it.
```
**EN:** This TableGen def record defines `TT_DescriptorGatherOp` with the summary “gather multiple rows from a descriptor into a single tensor”. It is specialized from `TT_Op<"descriptor_gather", [TT_DescriptorLoadLikeOpInterface]>`.
**CN:** 该 TableGen def 记录定义了 `TT_DescriptorGatherOp`，其摘要为“gather multiple rows from a descriptor into a single tensor”。 它基于 `TT_Op<"descriptor_gather", [TT_DescriptorLoadLikeOpInterface]>` 进一步特化。

### Lines 1303-1306
```tablegen
1303:     `desc_ptr` is a pointer to the TMA descriptor allocated in global memory.
1304:     The descriptor block must have 1 row and the indices must be a 1D tensor.
1305:     Accordingly, the result is a 2D tensor multiple rows.
1306:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1308-1313
```tablegen
1308:   let arguments = (ins
1309:     Arg<TT_TensorDescType, "", [MemRead<GlobalMemory>]>:$desc,
1310:     RankedTensorOf<[I16, I32]>:$x_offsets,
1311:     I32:$y_offset
1312:   );
1313:   let results = (outs TT_Tensor:$result);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 1315-1318
```tablegen
1315:   let assemblyFormat = [{
1316:     $desc `[` $x_offsets `,` $y_offset `]`
1317:     attr-dict `:` functional-type(operands, results)
1318:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1320-1321
```tablegen
1320:   let hasVerifier = 1;
1321: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 1323-1327
```tablegen
1323: def TT_DescriptorScatterOp : TT_Op<"descriptor_scatter", [TT_DescriptorStoreLikeOpInterface]> {
1324:   let summary = "scatter multiple rows to a descriptor from a single tensor";
1325:   let description = [{
1326:     The `tt.descriptor_scatter` op will be lowered to NVIDIA TMA
1327:     scatter operations on targets that support it.
```
**EN:** This TableGen def record defines `TT_DescriptorScatterOp` with the summary “scatter multiple rows to a descriptor from a single tensor”. It is specialized from `TT_Op<"descriptor_scatter", [TT_DescriptorStoreLikeOpInterface]>`.
**CN:** 该 TableGen def 记录定义了 `TT_DescriptorScatterOp`，其摘要为“scatter multiple rows to a descriptor from a single tensor”。 它基于 `TT_Op<"descriptor_scatter", [TT_DescriptorStoreLikeOpInterface]>` 进一步特化。

### Lines 1329-1332
```tablegen
1329:     `desc_ptr` is a pointer to the TMA descriptor allocated in global memory.
1330:     The descriptor block must have 1 row and the indices must be a 1D tensor.
1331:     Accordingly, the result is a 2D tensor multiple rows.
1332:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 1334-1339
```tablegen
1334:   let arguments = (ins
1335:     Arg<TT_TensorDescType, "", [MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>:$desc,
1336:     RankedTensorOf<[I16, I32]>:$x_offsets,
1337:     I32:$y_offset,
1338:     TT_Tensor:$src
1339:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 1341-1344
```tablegen
1341:   let assemblyFormat = [{
1342:     $desc `[` $x_offsets `,` $y_offset `]` `,` $src
1343:     attr-dict `:` type(operands)
1344:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1346-1347
```tablegen
1346:   let hasVerifier = 1;
1347: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 1350-1350
```tablegen
1350: #endif // Triton_OPS
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** layout encodings  
  **CN:** 布局编码
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** interfaces  
  **CN:** 接口
- **EN:** attributes  
  **CN:** 属性
- **EN:** types  
  **CN:** 类型
- **EN:** traits  
  **CN:** 特征约束
- **EN:** TableGen-driven code generation  
  **CN:** TableGen 驱动的代码生成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `triton/Dialect/Triton/IR/TritonDialect.td`
  - `triton/Dialect/Triton/IR/TritonTypes.td`
  - `triton/Dialect/Triton/IR/TritonAttrDefs.td`
  - `triton/Dialect/Triton/IR/TritonInterfaces.td`
  - `mlir/IR/OpBase.td`
  - `mlir/IR/SymbolInterfaces.td`
  - `mlir/IR/OpAsmInterface.td`
  - `mlir/Interfaces/FunctionInterfaces.td`
  - `mlir/Interfaces/SideEffectInterfaces.td`
  - `mlir/Interfaces/ControlFlowInterfaces.td`
  - `mlir/Interfaces/InferTypeOpInterface.td`
  - `mlir/Interfaces/CallInterfaces.td`
  - `triton/Dialect/Triton/IR/TritonOpInterfaces.td`
