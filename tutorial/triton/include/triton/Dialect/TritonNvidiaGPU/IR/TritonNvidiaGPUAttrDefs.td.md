# TritonNvidiaGPUAttrDefs.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUAttrDefs.td`
- **EN:** Declares or defines attribute records and helper APIs used by this subsystem.
- **CN:** 声明或定义该子系统使用的属性记录与辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```tablegen
   1: #ifndef TRITONNVIDIAGPU_ATTRDEFS
   2: #define TRITONNVIDIAGPU_ATTRDEFS
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-9
```tablegen
   4: include "mlir/IR/AttrTypeBase.td"
   5: include "mlir/IR/EnumAttr.td"
   6: include "triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUDialect.td"
   7: include "triton/Dialect/Triton/IR/TritonInterfaces.td"
   8: include "triton/Dialect/TritonGPU/IR/TritonGPUAttrInterfaces.td"
   9: include "mlir/IR/EnumAttr.td"
```
**EN:** This block imports the direct dependencies needed here, including mlir/IR/AttrTypeBase.td, mlir/IR/EnumAttr.td, triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUDialect.td, triton/Dialect/Triton/IR/TritonInterfaces.td, triton/Dialect/TritonGPU/IR/TritonGPUAttrInterfaces.td, and mlir/IR/EnumAttr.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/IR/AttrTypeBase.td, mlir/IR/EnumAttr.td, triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUDialect.td, triton/Dialect/Triton/IR/TritonInterfaces.td, triton/Dialect/TritonGPU/IR/TritonGPUAttrInterfaces.td, and mlir/IR/EnumAttr.td。

### Lines 11-16
```tablegen
  11: def TTG_TensorMemorySpace : AttrDef<TritonNvidiaGPU_Dialect, "TensorMemorySpace"> {
  12:   let mnemonic = "tensor_memory";
  13:   let description = [{
  14:     Attribute to indicate that the memory descriptor points to tensor memory.
  15:     The memory is laid out in blocks of size blockM x blockN. Each block is distributed
  16:     across TMEM 128 rows.
```
**EN:** This TableGen def record defines `TTG_TensorMemorySpace`. It is specialized from `AttrDef<TritonNvidiaGPU_Dialect, "TensorMemorySpace">`.
**CN:** 该 TableGen def 记录定义了 `TTG_TensorMemorySpace`。 它基于 `AttrDef<TritonNvidiaGPU_Dialect, "TensorMemorySpace">` 进一步特化。

### Lines 18-19
```tablegen
  18:     Blocks are distributed along M dimension first and then N dimension. This is an arbitrary
  19:     convention that needs to be followed by operations reading/writing to TMEM.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 21-21
```tablegen
  21:     a tensor <128x128xf32> with blockM = 64 and blockN = 32 will be distributed as follows:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 23-35
```tablegen
  23:         \ col    0        1            31         32            64            96           127
  24:     rows: 0  ( 0,  0) ( 0,  1) ... ( 0,  31)  ( 0,  32) ... ( 0,  64) ... ( 0,  96) ... ( 0,  127)
  25:           1
  26:          ...
  27:           15 (15,  0) (15,  1) ... (15,  31)  (15,  32) ... (15,  64) ... (15,  96) ... (15,  127)
  28:           16 (64,  0) (64,  1) ... (64,  31)  (64,  32) ... (64,  64) ... (64,  96) ... (64,  127)
  29:          ...
  30:           31 (79,  0) (79,  1) ... (79,  31)  (79,  32) ... (79,  64) ... (79,  96) ... (79,  127)
  31:           32 (16,  0) (16,  1) ... (16,  31)  (16,  32) ... (16,  64) ... (16,  96) ... (16,  127)
  32:          ..
  33:          127 (127, 0) (127, 1) ... (127, 31) (127, 32) ... (127, 64) ... (127, 96) ... (127, 127)
  34:   }];
  35: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 37-48
```tablegen
  37: def TTNG_TMEMLoadReduceModifierAttr : I32EnumAttr<
  38:     "TMEMLoadReduceModifier", "",
  39:     [
  40:         I32EnumAttrCase<"MIN", 1, "min">,
  41:         I32EnumAttrCase<"MAX", 2, "max">,
  42:     ]> {
  43:     let cppNamespace = "::mlir::triton::nvidia_gpu";
  44:     let genSpecializedAttr = 0;
  45: }
  46: def TTNG_TMEMLoadReduceModifierEnum : EnumAttr<TritonNvidiaGPU_Dialect, TTNG_TMEMLoadReduceModifierAttr, "redOp"> {
  47:   let assemblyFormat = "`<` $value `>`";
  48: }
```
**EN:** This TableGen def record defines `TTNG_TMEMLoadReduceModifierAttr`. It is specialized from `I32EnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TMEMLoadReduceModifierAttr`。 它基于 `I32EnumAttr<` 进一步特化。

### Lines 50-63
```tablegen
  50: def TTNG_CGALayoutRankTwoParam
  51:     : AttrOrTypeParameter<"::mlir::triton::gpu::CGAEncodingAttr",
  52:                           "rank-2 CGA layout"> {
  53:   let parser = [{
  54:     [&]() -> ::mlir::FailureOr<::mlir::triton::gpu::CGAEncodingAttr> {
  55:       return ::mlir::triton::nvidia_gpu::parseCGALayoutRankTwo($_parser);
  56:     }()
  57:   }];
  58:   let printer = [{
  59:     ::mlir::triton::nvidia_gpu::printCGALayoutRankTwo($_printer, $_self);
  60:   }];
  61:   let defaultValue =
  62:       "::mlir::triton::gpu::CGAEncodingAttr::get1CTALayout($_ctxt, 2)";
  63: }
```
**EN:** This TableGen def record defines `TTNG_CGALayoutRankTwoParam`.
**CN:** 该 TableGen def 记录定义了 `TTNG_CGALayoutRankTwoParam`。

### Lines 65-89
```tablegen
  65: def TTG_TensorMemoryEncodingAttr
  66:     : AttrDef<TritonNvidiaGPU_Dialect, "TensorMemoryEncoding",
  67:               [LayoutEncodingTrait]> {
  68:   let mnemonic = "tensor_memory_encoding";
  69:   let attrName = "triton.gpu.tensor_memory_encoding";
  70:   let description = [{
  71:     An encoding to represent the different way the tensor memory is laid out.
  72:     `colStride` describes the stride in elements along the column dimension,
  73:     that is, the stride between two elements in the same row.
  74:     When colStride is 1 the tensor memory is packed. When colStride > 1, the
  75:     tensor memory between elements is undefined.
  76:     `twoCTAs` indicates that the tensor memory is laid out for twoCTA mode,
  77:     i.e., `cta_group::2`.
  78:   }];
  79:   let parameters = (
  80:     ins
  81:     "unsigned":$blockM,
  82:     "unsigned":$blockN,
  83:     "unsigned":$colStride,
  84:     TTNG_CGALayoutRankTwoParam:$CGALayout,
  85:     DefaultValuedParameter<"bool", "false">:$twoCTAs
  86:   );
  87:   let genVerifyDecl = 1;
  88:   let assemblyFormat = "`<` struct(params) `>`";
  89: }
```
**EN:** This TableGen def record defines `TTG_TensorMemoryEncodingAttr`.
**CN:** 该 TableGen def 记录定义了 `TTG_TensorMemoryEncodingAttr`。

### Lines 91-109
```tablegen
  91: def TTG_TensorMemoryScalesEncodingAttr
  92:     : AttrDef<TritonNvidiaGPU_Dialect, "TensorMemoryScalesEncoding",
  93:               [LayoutEncodingTrait]> {
  94:   let mnemonic = "tensor_memory_scales_encoding";
  95:   let attrName = "triton.gpu.tensor_memory_scales_encoding";
  96:   let description = [{
  97:     An encoding to represent the layout of tensor memory scales.
  98:     As described in the PTX doc, blocked scales in TMEM must be in a special layout. They are organized
  99:     as a multiple copies of "chunk", each of which having the size 32x4x4B. Moreover, such chunks are duplicated
 100:     over 4 warps to fill entire 128 rows of TMEM. This encoding indicates that a tensor in TMEM is in such a special
 101:     layout.
 102:   }];
 103:   let parameters = (
 104:     ins
 105:     TTNG_CGALayoutRankTwoParam:$CGALayout
 106:   );
 107:   let genVerifyDecl = 1;
 108:   let assemblyFormat = "`<` struct(params) `>`";
 109: }
```
**EN:** This TableGen def record defines `TTG_TensorMemoryScalesEncodingAttr`.
**CN:** 该 TableGen def 记录定义了 `TTG_TensorMemoryScalesEncodingAttr`。

### Lines 111-111
```tablegen
 111: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** layout encodings  
  **CN:** 布局编码
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** interfaces  
  **CN:** 接口
- **EN:** attributes  
  **CN:** 属性
- **EN:** types  
  **CN:** 类型
- **EN:** traits  
  **CN:** 特征约束
- **EN:** NVIDIA backend support  
  **CN:** NVIDIA 后端支持

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/IR/AttrTypeBase.td`
  - `mlir/IR/EnumAttr.td`
  - `triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUDialect.td`
  - `triton/Dialect/Triton/IR/TritonInterfaces.td`
  - `triton/Dialect/TritonGPU/IR/TritonGPUAttrInterfaces.td`
  - `mlir/IR/EnumAttr.td`
