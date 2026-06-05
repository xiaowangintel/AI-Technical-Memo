# fully-connected.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/fully-connected.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `fully-connected.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `fully-connected.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15 / 第 1-15 行

```c
0001: /*
0002:  * Copyright (c) Facebook, Inc. and its affiliates.
0003:  * All rights reserved.
0004:  *
0005:  * This source code is licensed under the BSD-style license found in the
0006:  * LICENSE file in the root directory of this source tree.
0007:  */
0008: 
0009: #include <assert.h>
0010: #include <math.h>
0011: #include <stdbool.h>
0012: #include <stddef.h>
0013: #include <stdint.h>
0014: #include <string.h>
0015: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 16-35 / 第 16-35 行

```c
0016: #include <pytorch_qnnpack.h>
0017: #include <qnnpack/log.h>
0018: #include <qnnpack/math.h>
0019: #include <qnnpack/operator.h>
0020: #include <qnnpack/pack.h>
0021: #include <qnnpack/params.h>
0022: #include <qnnpack/requantization.h>
0023: 
0024: enum pytorch_qnnp_status pytorch_qnnp_create_fully_connected_nc_q8(
0025:     size_t input_channels,
0026:     size_t output_channels,
0027:     uint8_t input_zero_point,
0028:     const uint8_t* kernel_zero_points,
0029:     const uint8_t* kernel,
0030:     const int32_t* bias,
0031:     uint8_t output_zero_point,
0032:     uint8_t output_min,
0033:     uint8_t output_max,
0034:     uint32_t flags,
0035:     const float* requantization_scales,
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_status`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_status`。

### Lines 36-45 / 第 36-45 行

```c
0036:     pytorch_qnnp_operator_t* fully_connected_out) {
0037:   pytorch_qnnp_operator_t fully_connected = NULL;
0038:   enum pytorch_qnnp_status status = pytorch_qnnp_status_uninitialized;
0039: 
0040:   if (!pytorch_qnnp_params.initialized) {
0041:     pytorch_qnnp_log_error(
0042:         "pytorch_qnnp_create_fully_connected_nc_q8 failed because QNNPACK is not properly initialized");
0043:     goto error;
0044:   }
0045: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_status`, `pytorch_qnnp_log_error`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_status`, `pytorch_qnnp_log_error`。

### Lines 46-57 / 第 46-57 行

```c
0046:   status = pytorch_qnnp_status_unsupported_parameter;
0047: 
0048:   for (int i = 0; i < output_channels; ++i) {
0049:     if (requantization_scales[i] <= 0.0f ||
0050:         !isnormal(requantization_scales[i])) {
0051:       pytorch_qnnp_log_error(
0052:           "failed to create fully connected operator with %.7g requantization scale: scale must be finite and positive",
0053:           requantization_scales[i]);
0054:       goto error;
0055:     }
0056:   }
0057: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`pytorch_qnnp_log_error`。

### Lines 58-67 / 第 58-67 行

```c
0058:   status = pytorch_qnnp_status_out_of_memory;
0059: 
0060:   fully_connected = calloc(1, sizeof(struct pytorch_qnnp_operator));
0061:   if (fully_connected == NULL) {
0062:     pytorch_qnnp_log_error(
0063:         "failed to allocate %zu bytes for pytorch_qnnp_operator structure",
0064:         sizeof(struct pytorch_qnnp_operator));
0065:     goto error;
0066:   }
0067: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_operator`, `pytorch_qnnp_log_error`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_operator`, `pytorch_qnnp_log_error`。

### Lines 68-86 / 第 68-86 行

```c
0068:   const uint32_t nr = pytorch_qnnp_params.q8conv.nr;
0069:   const uint32_t kr = pytorch_qnnp_params.q8conv.kr;
0070: 
0071:   const uint32_t n_stride = (output_channels + (nr - 1)) & -nr;
0072:   const uint32_t k_stride = (input_channels + (kr - 1)) & -kr;
0073: 
0074:   fully_connected->packed_weights =
0075:       malloc(n_stride * (k_stride * sizeof(uint8_t) + sizeof(int32_t)));
0076:   if (fully_connected->packed_weights == NULL) {
0077:     pytorch_qnnp_log_error(
0078:         "failed to allocate %zu bytes for packed weights",
0079:         n_stride * (k_stride * sizeof(uint8_t) + sizeof(int32_t)));
0080:     goto error;
0081:   }
0082:   memset(
0083:       fully_connected->packed_weights,
0084:       kernel_zero_points[0],
0085:       n_stride * (k_stride * sizeof(uint8_t) + sizeof(int32_t)));
0086: 
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries. Key symbols: `malloc`, `pytorch_qnnp_log_error`, `memset`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界。关键符号：`malloc`, `pytorch_qnnp_log_error`, `memset`。

### Lines 87-96 / 第 87-96 行

```c
0087:   pytorch_pack_q8gemm_w(
0088:       output_channels,
0089:       input_channels,
0090:       nr,
0091:       nr,
0092:       kr,
0093: #if !PYTORCH_QNNPACK_RUNTIME_QUANTIZATION
0094:       input_zero_point,
0095:       kernel_zero_points[0],
0096: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：无明显局部符号。

### Lines 97-107 / 第 97-107 行

```c
0097:       kernel,
0098:       bias,
0099: #if PYTORCH_QNNPACK_RUNTIME_QUANTIZATION
0100:       kernel_zero_points,
0101: #endif
0102:       fully_connected->packed_weights);
0103: 
0104:   fully_connected->groups = 1;
0105:   fully_connected->group_input_channels = input_channels;
0106:   fully_connected->group_output_channels = output_channels;
0107: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：无明显局部符号。

### Lines 108-118 / 第 108-118 行

```c
0108:   fully_connected->kernel_zero_point = kernel_zero_points[0];
0109: 
0110:   fully_connected->conv_quantization_params =
0111:       pytorch_qnnp_compute_conv_quantization_params(
0112:           input_zero_point,
0113:           kernel_zero_points,
0114:           requantization_scales,
0115:           output_zero_point,
0116:           output_min,
0117:           output_max);
0118: 
```

- **EN:** This block implements local helper logic for `fully-connected`. Key symbols: `pytorch_qnnp_compute_conv_quantization_params`.
- **CN:** 该代码块实现与 `fully-connected` 相关的局部辅助逻辑。关键符号：`pytorch_qnnp_compute_conv_quantization_params`。

### Lines 119-129 / 第 119-129 行

```c
0119:   fully_connected->ukernel_type = pytorch_qnnp_ukernel_type_gemm;
0120:   fully_connected->format = pytorch_qnnp_format_quint8;
0121: 
0122:   *fully_connected_out = fully_connected;
0123:   return pytorch_qnnp_status_success;
0124: 
0125: error:
0126:   pytorch_qnnp_delete_operator(fully_connected);
0127:   return status;
0128: }
0129: 
```

- **EN:** This block produces a result or forwards a computed value. Key symbols: `pytorch_qnnp_delete_operator`.
- **CN:** 该代码块返回结果或转发已计算的值。关键符号：`pytorch_qnnp_delete_operator`。

### Lines 130-140 / 第 130-140 行

```c
0130: enum pytorch_qnnp_status pytorch_qnnp_setup_fully_connected_nc_q8(
0131:     pytorch_qnnp_operator_t fully_connected,
0132:     size_t batch_size,
0133:     const uint8_t* input,
0134:     size_t input_stride,
0135:     uint8_t* output,
0136:     size_t output_stride) {
0137:   if (!pytorch_qnnp_params.initialized) {
0138:     pytorch_qnnp_log_error(
0139:         "pytorch_qnnp_setup_fully_connected_nc_q8 failed because QNNPACK is not properly initialized");
0140:     return pytorch_qnnp_status_uninitialized;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_status`, `pytorch_qnnp_setup_fully_connected_nc_q8`, `pytorch_qnnp_log_error`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_status`, `pytorch_qnnp_setup_fully_connected_nc_q8`, `pytorch_qnnp_log_error`。

### Lines 141-153 / 第 141-153 行

```c
0141:   }
0142: 
0143:   if (batch_size == 0) {
0144:     fully_connected->batch_size = 0;
0145:     return pytorch_qnnp_status_success;
0146:   }
0147: 
0148:   fully_connected->batch_size = 1;
0149:   fully_connected->input_height = batch_size;
0150:   fully_connected->input_width = 1;
0151:   fully_connected->input = input;
0152:   fully_connected->input_pixel_stride = input_stride;
0153: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：无明显局部符号。

### Lines 154-160 / 第 154-160 行

```c
0154:   fully_connected->output_height = batch_size;
0155:   fully_connected->output_width = 1;
0156:   fully_connected->output = output;
0157:   fully_connected->output_pixel_stride = output_stride;
0158: 
0159:   return pytorch_qnnp_status_success;
0160: }
```

- **EN:** This block produces a result or forwards a computed value. Key symbols: no prominent local symbols.
- **CN:** 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Memory allocation strategy** — 内存分配策略
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: pytorch_qnnp_operator, pytorch_qnnp_status, pytorch_qnnp_create_fully_connected_nc_q8, pytorch_qnnp_log_error, malloc, memset, pytorch_pack_q8gemm_w, pytorch_qnnp_compute_conv_quantization_params** — 核心符号：pytorch_qnnp_operator、pytorch_qnnp_status、pytorch_qnnp_create_fully_connected_nc_q8、pytorch_qnnp_log_error、malloc、memset、pytorch_pack_q8gemm_w、pytorch_qnnp_compute_conv_quantization_params

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `assert.h`, `math.h`, `stdbool.h`, `stddef.h`, `stdint.h`, `string.h`, `pytorch_qnnpack.h`, `qnnpack/log.h`, `qnnpack/math.h`, `qnnpack/operator.h`, `qnnpack/pack.h`, `qnnpack/params.h`, `...`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_qnnp_operator`, `pytorch_qnnp_status`, `pytorch_qnnp_create_fully_connected_nc_q8`, `pytorch_qnnp_log_error`, `malloc`, `memset`, `pytorch_pack_q8gemm_w`, `pytorch_qnnp_compute_conv_quantization_params`, `pytorch_qnnp_delete_operator`, `pytorch_qnnp_setup_fully_connected_nc_q8`
