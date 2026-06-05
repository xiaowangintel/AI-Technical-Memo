# CMakeLists.txt — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/transformers/hip/flash_attn/ck/CMakeLists.txt`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): Provides tooling or configuration used by the build/runtime flow for ROCm/HIP transformer kernels, centered on CMake Lists with emphasis on flash-attention style fusion.
- 用途（中文）: 提供构建/运行流程使用的工具或配置，属于ROCm/HIP Transformer 内核，核心主题是CMake Lists，重点关注flash-attention 风格融合。

## Line-by-Line Analysis / 逐行分析

- Note (EN): The file is annotated in contiguous line ranges; each numbered line receives a short bilingual explanation.
- 说明（中文）: 为兼顾可读性，以下按连续行区间展示代码，并对每个编号行给出简短的中英双语说明。

### Lines 1-16

```text
   1: # generate a list of kernels, but not actually emit files at config stage
   2: execute_process(
   3:   COMMAND python3 ${CMAKE_SOURCE_DIR}/third_party/composable_kernel/example/ck_tile/01_fmha/generate.py
   4:   --api fwd --receipt 4 --list_blobs ${CMAKE_CURRENT_LIST_DIR}/fwd_blob_list.txt
   5:   RESULT_VARIABLE ret
   6: )
   7: 
   8: if(ret AND NOT ret EQUAL 0)
   9:   message( FATAL_ERROR "CK Tile FMHA FAILED to generate a list of FWD kernels via Python.")
  10: endif()
  11: 
  12: execute_process(
  13:   COMMAND python3 ${CMAKE_SOURCE_DIR}/third_party/composable_kernel/example/ck_tile/01_fmha/generate.py
  14:   --api fwd_splitkv --receipt 4 --list_blobs ${CMAKE_CURRENT_LIST_DIR}/fwd_splitkv_blob_list.txt
  15:   RESULT_VARIABLE ret
  16: )
```
- L1: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L2: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L3: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L4: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L5: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L6: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L8: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L9: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L10: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L12: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L13: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L14: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L15: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L16: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 17-32

```text
  17: 
  18: if(ret AND NOT ret EQUAL 0)
  19:     message( FATAL_ERROR "CK Tile FMHA FAILED to generate a list of FWD_SPLITKV kernels via Python.")
  20: endif()
  21: 
  22: execute_process(
  23:   COMMAND python3 ${CMAKE_SOURCE_DIR}/third_party/composable_kernel/example/ck_tile/01_fmha/generate.py
  24:   --api fwd_appendkv --receipt 4 --list_blobs ${CMAKE_CURRENT_LIST_DIR}/fwd_appendkv_blob_list.txt
  25:   RESULT_VARIABLE ret
  26: )
  27: 
  28: if(ret AND NOT ret EQUAL 0)
  29:     message( FATAL_ERROR "CK Tile FMHA FAILED to generate a list of FWD_APPENDKV kernels via Python.")
  30: endif()
  31: 
  32: execute_process(
```
- L18: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L19: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L20: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L22: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L23: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L24: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L25: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L26: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L28: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L29: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L30: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L32: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 33-48

```text
  33:   COMMAND python3 ${CMAKE_SOURCE_DIR}/third_party/composable_kernel/example/ck_tile/01_fmha/generate.py
  34:   --api bwd --receipt 4 --list_blobs ${CMAKE_CURRENT_LIST_DIR}/bwd_blob_list.txt
  35:   RESULT_VARIABLE ret
  36: )
  37: 
  38: if(ret AND NOT ret EQUAL 0)
  39:   message( FATAL_ERROR "CK Tile FMHA FAILED to generate a list of BWD kernels via Python.")
  40: endif()
  41: 
  42: # Generate the files for both fwd, fwd_splitkv, fwd_appendkv, and bwd
  43: execute_process(COMMAND python3 ${CMAKE_SOURCE_DIR}/third_party/composable_kernel/example/ck_tile/01_fmha/generate.py --api fwd --receipt 4 --output_dir ${CMAKE_CURRENT_LIST_DIR}
  44: )
  45: 
  46: if(ret AND NOT ret EQUAL 0)
  47:   message( FATAL_ERROR "CK Tile FMHA FAILED to generate FWD kernels.")
  48: endif()
```
- L33: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L34: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L35: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L36: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L38: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L39: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L40: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L42: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L43: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L44: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L46: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L47: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L48: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 49-64

```text
  49: 
  50: execute_process(COMMAND python3 ${CMAKE_SOURCE_DIR}/third_party/composable_kernel/example/ck_tile/01_fmha/generate.py --api fwd_splitkv --receipt 4 --output_dir ${CMAKE_CURRENT_LIST_DIR}
  51: )
  52: 
  53: if(ret AND NOT ret EQUAL 0)
  54:     message( FATAL_ERROR "CK Tile FMHA FAILED to generate FWD_SPLITKV kernels.")
  55: endif()
  56: 
  57: execute_process(COMMAND python3 ${CMAKE_SOURCE_DIR}/third_party/composable_kernel/example/ck_tile/01_fmha/generate.py --api fwd_appendkv --receipt 4 --output_dir ${CMAKE_CURRENT_LIST_DIR}
  58: )
  59: 
  60: if(ret AND NOT ret EQUAL 0)
  61:     message( FATAL_ERROR "CK Tile FMHA FAILED to generate FWD_APPENDKV kernels.")
  62: endif()
  63: 
  64: execute_process(COMMAND python3 ${CMAKE_SOURCE_DIR}/third_party/composable_kernel/example/ck_tile/01_fmha/generate.py --api bwd --receipt 4 --output_dir ${CMAKE_CURRENT_LIST_DIR}
```
- L50: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L51: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L53: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L54: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L55: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L57: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L58: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L60: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L61: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L62: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L64: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 65-80

```text
  65:   RESULT_VARIABLE ret
  66: )
  67: 
  68: if(ret AND NOT ret EQUAL 0)
  69:   message( FATAL_ERROR "CK Tile FMHA FAILED to generate BWD kernels.")
  70: endif()
  71: 
  72: # Change make_kernel to make_kernel_pt for fwd
  73: execute_process(
  74:   COMMAND bash -c "${CMAKE_CURRENT_LIST_DIR}/add_make_kernel_pt.sh ${CMAKE_CURRENT_LIST_DIR}/fwd_blob_list.txt"
  75:   RESULT_VARIABLE ret)
  76: 
  77: if(ret AND NOT ret EQUAL 0)
  78:   message( FATAL_ERROR "CK Tile FMHA FAILED to change make_kernel to make_kernel_pt for the fwd pass")
  79: endif()
  80: 
```
- L65: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L66: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L68: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L69: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L70: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L72: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L73: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L74: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L75: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L77: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L78: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L79: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 81-96

```text
  81: execute_process(
  82:   COMMAND bash -c "${CMAKE_CURRENT_LIST_DIR}/add_make_kernel_pt.sh ${CMAKE_CURRENT_LIST_DIR}/fwd_splitkv_blob_list.txt"
  83:   RESULT_VARIABLE ret)
  84: 
  85: if(ret AND NOT ret EQUAL 0)
  86:   message( FATAL_ERROR "CK Tile FMHA FAILED to change make_kernel to make_kernel_pt for the fwd_splitkv pass")
  87: endif()
  88: 
  89: execute_process(
  90:   COMMAND bash -c "${CMAKE_CURRENT_LIST_DIR}/add_make_kernel_pt.sh ${CMAKE_CURRENT_LIST_DIR}/fwd_appendkv_blob_list.txt"
  91:   RESULT_VARIABLE ret)
  92: 
  93: if(ret AND NOT ret EQUAL 0)
  94:   message( FATAL_ERROR "CK Tile FMHA FAILED to change make_kernel to make_kernel_pt for the fwd appendkv pass")
  95: endif()
  96: 
```
- L81: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L82: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L83: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L85: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L86: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L87: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L89: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L90: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L91: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L93: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L94: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L95: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 97-112

```text
  97: # Change make_kernel to make_kernel_pt for bwd
  98: execute_process(
  99:   COMMAND bash -c "${CMAKE_CURRENT_LIST_DIR}/add_make_kernel_pt.sh ${CMAKE_CURRENT_LIST_DIR}/bwd_blob_list.txt"
 100:   RESULT_VARIABLE ret)
 101: 
 102: if(ret AND NOT ret EQUAL 0)
 103:   message( FATAL_ERROR "CK Tile FMHA FAILED to change make_kernel to make_kernel_pt for the bwd pass")
 104: endif()
 105: 
 106: # Change file extensions to .hip
 107: execute_process(COMMAND bash -c "for file in ${CMAKE_CURRENT_LIST_DIR}/*.cpp; do mv -- \"$file\" \"\${file%.cpp}.hip\"; done"
 108:   RESULT_VARIABLE ret
 109: )
 110: 
 111: if(ret AND NOT ret EQUAL 0)
 112:   message( FATAL_ERROR "CK Tile FMHA FAILED to change the generated instances extensions from .cpp to .hpp")
```
- L97: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L98: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L99: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L100: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L102: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L103: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L104: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L106: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L107: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L108: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L109: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L111: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L112: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 113-113

```text
 113: endif()
```
- L113: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

## Key Concepts / 关键概念

- ROCm/HIP transformer kernels / ROCm/HIP Transformer 内核
- Transformer attention operators and helpers / Transformer 注意力算子与辅助逻辑
- HIP/ROCm specialization / HIP/ROCm 特化

## Dependencies / 依赖关系

- No direct `#include` lines; dependencies are expressed through shader resources, build tooling, or neighboring generated context. / 没有直接的 `#include` 语句；依赖通过着色器资源、构建工具或相邻生成上下文体现。
- Subsystem tie-in: transformer attention, masking, scaling, and backend-specific fused kernels. / 子系统关联：Transformer 注意力、掩码、缩放以及后端特化的融合内核。
