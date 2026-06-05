# TritonAttrDefs.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/Triton/IR/TritonAttrDefs.td`
- **EN:** Declares or defines attribute records and helper APIs used by this subsystem.
- **CN:** 声明或定义该子系统使用的属性记录与辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```tablegen
   1: #ifndef TRITON_ATTR_DEFS
   2: #define TRITON_ATTR_DEFS
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-4
```tablegen
   4: include "mlir/IR/EnumAttr.td"
```
**EN:** This block imports the direct dependencies needed here, including mlir/IR/EnumAttr.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/IR/EnumAttr.td。

### Lines 6-19
```tablegen
   6: // Attributes for LoadOp and StoreOp
   7: def TT_CacheModifierAttr : I32EnumAttr<
   8:     "CacheModifier", "",
   9:     [
  10:         I32EnumAttrCase<"NONE", 1, "none">,
  11:         I32EnumAttrCase<"CA", 2, "ca">,
  12:         I32EnumAttrCase<"CG", 3, "cg">,
  13:         I32EnumAttrCase<"WB", 4, "wb">,
  14:         I32EnumAttrCase<"CS", 5, "cs">,
  15:         I32EnumAttrCase<"WT", 6, "wt">,
  16:         I32EnumAttrCase<"CV", 7, "cv">,
  17:     ]> {
  18:     let cppNamespace = "::mlir::triton";
  19: }
```
**EN:** This TableGen def record defines `TT_CacheModifierAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_CacheModifierAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 21-30
```tablegen
  21: def TT_MemSemanticAttr : I32EnumAttr<
  22:     "MemSemantic", "",
  23:     [
  24:       I32EnumAttrCase<"RELAXED", 1, "relaxed">,
  25:       I32EnumAttrCase<"ACQUIRE", 2, "acquire">,
  26:       I32EnumAttrCase<"RELEASE", 3, "release">,
  27:       I32EnumAttrCase<"ACQUIRE_RELEASE", 4, "acq_rel">,
  28:     ]> {
  29:     let cppNamespace = "::mlir::triton";
  30: }
```
**EN:** This TableGen def record defines `TT_MemSemanticAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_MemSemanticAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 32-40
```tablegen
  32: def TT_EvictionPolicyAttr : I32EnumAttr<
  33:     "EvictionPolicy", "",
  34:     [
  35:         I32EnumAttrCase<"NORMAL", 1, "evict_normal">,
  36:         I32EnumAttrCase<"EVICT_FIRST", 2, "evict_first">,
  37:         I32EnumAttrCase<"EVICT_LAST", 3, "evict_last">
  38:     ]> {
  39:     let cppNamespace = "::mlir::triton";
  40: }
```
**EN:** This TableGen def record defines `TT_EvictionPolicyAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_EvictionPolicyAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 42-50
```tablegen
  42: def TT_PaddingOptionAttr : I32EnumAttr<
  43:     "PaddingOption", "",
  44:     [
  45:         I32EnumAttrCase<"PAD_ZERO", 1, "zero">,
  46:         // We can not set the string value to "NAN" because it is a keyword in C++
  47:         I32EnumAttrCase<"PAD_NAN", 2, "nan">
  48:     ]> {
  49:     let cppNamespace = "::mlir::triton";
  50: }
```
**EN:** This TableGen def record defines `TT_PaddingOptionAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_PaddingOptionAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 52-68
```tablegen
  52: // atomic
  53: def TT_AtomicRMWAttr : I32EnumAttr<
  54:     "RMWOp", "",
  55:     [
  56:         I32EnumAttrCase<"AND", 1, "and">,
  57:         I32EnumAttrCase<"OR", 2, "or">,
  58:         I32EnumAttrCase<"XOR", 3, "xor">,
  59:         I32EnumAttrCase<"ADD", 4, "add">,
  60:         I32EnumAttrCase<"FADD", 5, "fadd">,
  61:         I32EnumAttrCase<"MAX", 6, "max">,
  62:         I32EnumAttrCase<"MIN", 7, "min">,
  63:         I32EnumAttrCase<"UMAX", 8, "umax">,
  64:         I32EnumAttrCase<"UMIN", 9, "umin">,
  65:         I32EnumAttrCase<"XCHG", 10, "exch">
  66:     ]> {
  67:     let cppNamespace = "::mlir::triton";
  68: }
```
**EN:** This TableGen def record defines `TT_AtomicRMWAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_AtomicRMWAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 70-83
```tablegen
  70: def TT_DescriptorReduceKindAttr : I32EnumAttr<
  71:     "DescriptorReduceKind", "",
  72:     [
  73:         I32EnumAttrCase<"ADD", 1, "add">,
  74:         I32EnumAttrCase<"MIN", 2, "min">,
  75:         I32EnumAttrCase<"MAX", 3, "max">,
  76:         I32EnumAttrCase<"INC", 4, "inc">,
  77:         I32EnumAttrCase<"DEC", 5, "dec">,
  78:         I32EnumAttrCase<"AND", 6, "and">,
  79:         I32EnumAttrCase<"OR", 7, "or">,
  80:         I32EnumAttrCase<"XOR", 8, "xor">,
  81:     ]> {
  82:     let cppNamespace = "::mlir::triton";
  83: }
```
**EN:** This TableGen def record defines `TT_DescriptorReduceKindAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_DescriptorReduceKindAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 85-93
```tablegen
  85: def TT_MemSyncScopeAttr : I32EnumAttr<
  86:     "MemSyncScope", "",
  87:     [
  88:       I32EnumAttrCase<"GPU", 1, "gpu">,
  89:       I32EnumAttrCase<"CTA", 2, "cta">,
  90:       I32EnumAttrCase<"SYSTEM", 3, "sys">,
  91:     ]> {
  92:     let cppNamespace = "::mlir::triton";
  93: }
```
**EN:** This TableGen def record defines `TT_MemSyncScopeAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_MemSyncScopeAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 95-104
```tablegen
  95: // Program ID dimensions.
  96: def TT_ProgramDim : I32EnumAttr<
  97:     "ProgramIDDim", "",
  98:     [
  99:         I32EnumAttrCase<"X", 0, "x">,
 100:         I32EnumAttrCase<"Y", 1, "y">,
 101:         I32EnumAttrCase<"Z", 2, "z">,
 102:     ]> {
 103:     let cppNamespace = "::mlir::triton";
 104: }
```
**EN:** This TableGen def record defines `TT_ProgramDim`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_ProgramDim`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 106-114
```tablegen
 106: // Rounding mode.
 107: def TT_RoundingModeAttr : I32EnumAttr<
 108:     "RoundingMode", "",
 109:     [
 110:         I32EnumAttrCase<"RTZ", 0, "rtz">,
 111:         I32EnumAttrCase<"RTNE", 1, "rtne">,
 112:     ]> {
 113:     let cppNamespace = "::mlir::triton";
 114: }
```
**EN:** This TableGen def record defines `TT_RoundingModeAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_RoundingModeAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 116-124
```tablegen
 116: // PropagateNan.
 117: def TT_PropagateNanAttr : I32EnumAttr<
 118:     "PropagateNan", "",
 119:     [
 120:         I32EnumAttrCase<"NONE", 0, "none">,
 121:         I32EnumAttrCase<"ALL", 0xFFFF, "all">,
 122:     ]> {
 123:     let cppNamespace = "::mlir::triton";
 124: }
```
**EN:** This TableGen def record defines `TT_PropagateNanAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_PropagateNanAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 126-137
```tablegen
 126: // InputPrecision
 127: def TT_InputPrecisionAttr : I32EnumAttr<
 128:     "InputPrecision", "",
 129:     [
 130:       I32EnumAttrCase<"TF32", 0, "tf32">,
 131:       I32EnumAttrCase<"TF32x3", 1, "tf32x3">,
 132:       I32EnumAttrCase<"IEEE", 2, "ieee">,
 133:       I32EnumAttrCase<"BF16x3", 3, "bf16x3">,
 134:       I32EnumAttrCase<"BF16x6", 4, "bf16x6">
 135:     ]>{
 136:   let cppNamespace = "::mlir::triton";
 137: }
```
**EN:** This TableGen def record defines `TT_InputPrecisionAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_InputPrecisionAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 139-152
```tablegen
 139: // Type for ScaleDotElemType kind of floats.
 140: def TT_ScaleDotElemTypeAttr : I32EnumAttr<
 141:     "ScaleDotElemType", "",
 142:     [
 143:       I32EnumAttrCase<"E4M3", 0, "e4m3">,
 144:       I32EnumAttrCase<"E5M2", 1, "e5m2">,
 145:       I32EnumAttrCase<"E2M3", 2, "e2m3">,
 146:       I32EnumAttrCase<"E3M2", 3, "e3m2">,
 147:       I32EnumAttrCase<"E2M1", 4, "e2m1">,
 148:       I32EnumAttrCase<"BF16", 5, "bf16">,
 149:       I32EnumAttrCase<"FP16", 6, "fp16">
 150:     ]>{
 151:   let cppNamespace = "::mlir::triton";
 152: }
```
**EN:** This TableGen def record defines `TT_ScaleDotElemTypeAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TT_ScaleDotElemTypeAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 154-154
```tablegen
 154: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** dot-product lowering  
  **CN:** 点积降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** attributes  
  **CN:** 属性
- **EN:** types  
  **CN:** 类型
- **EN:** TableGen-driven code generation  
  **CN:** TableGen 驱动的代码生成
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/IR/EnumAttr.td`
