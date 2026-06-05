# leaky-relu.c — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/leaky-relu.c`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `leaky-relu.c`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.."
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `leaky-relu.c` 展开。 文件头部注释也概括了其核心职责。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行

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
0011: #include <stddef.h>
0012: #include <stdint.h>
0013: #include <stdlib.h>
0014: 
```

- **EN:** This block implements local helper logic for `leaky-relu`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `leaky-relu` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 15-31 / 第 15-31 行

```c
0015: #include <pytorch_qnnpack.h>
0016: #include <qnnpack/log.h>
0017: #include <qnnpack/operator.h>
0018: 
0019: enum pytorch_qnnp_status pytorch_qnnp_create_leaky_relu_nc_q8(
0020:     size_t channels,
0021:     float negative_slope,
0022:     uint8_t input_zero_point,
0023:     float input_scale,
0024:     uint8_t output_zero_point,
0025:     float output_scale,
0026:     uint8_t output_min,
0027:     uint8_t output_max,
0028:     uint32_t flags,
0029:     pytorch_qnnp_operator_t* leaky_relu_out) {
0030:   pytorch_qnnp_operator_t leaky_relu_op = NULL;
0031:   enum pytorch_qnnp_status status = pytorch_qnnp_status_uninitialized;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_status`, `pytorch_qnnp_create_leaky_relu_nc_q8`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_status`, `pytorch_qnnp_create_leaky_relu_nc_q8`。

### Lines 32-41 / 第 32-41 行

```c
0032: 
0033:   if (!pytorch_qnnp_params.initialized) {
0034:     pytorch_qnnp_log_error(
0035:         "pytorch_qnnp_create_leaky_relu_nc_q8 failed because QNNPACK is not properly initialized");
0036:     goto error;
0037:   }
0038: 
0039:   status = pytorch_qnnp_status_invalid_parameter;
0040: 
0041:   if (channels == 0) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`pytorch_qnnp_log_error`。

### Lines 42-54 / 第 42-54 行

```c
0042:     pytorch_qnnp_log_error(
0043:         "failed to create Leaky ReLU operator with %zu channels: number of channels must be non-zero",
0044:         channels);
0045:     goto error;
0046:   }
0047: 
0048:   if (negative_slope <= 0.0f || !isnormal(negative_slope)) {
0049:     pytorch_qnnp_log_error(
0050:         "failed to create Leaky ReLU operator with %.7g negative slope: slope must be finite and positive",
0051:         negative_slope);
0052:     goto error;
0053:   }
0054: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`pytorch_qnnp_log_error`。

### Lines 55-68 / 第 55-68 行

```c
0055:   if (negative_slope > 1.0f) {
0056:     pytorch_qnnp_log_error(
0057:         "failed to create Leaky ReLU operator with %.7g negative slope: slope must not exceed 1.0",
0058:         negative_slope);
0059:     goto error;
0060:   }
0061: 
0062:   if (input_scale <= 0.0f || !isnormal(input_scale)) {
0063:     pytorch_qnnp_log_error(
0064:         "failed to create Leaky ReLU operator with %.7g input scale: scale must be finite and positive",
0065:         input_scale);
0066:     goto error;
0067:   }
0068: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`pytorch_qnnp_log_error`。

### Lines 69-84 / 第 69-84 行

```c
0069:   if (output_scale <= 0.0f || !isnormal(output_scale)) {
0070:     pytorch_qnnp_log_error(
0071:         "failed to create Leaky ReLU operator with %.7g output scale: scale must be finite and positive",
0072:         output_scale);
0073:     goto error;
0074:   }
0075: 
0076:   if (output_min >= output_max) {
0077:     pytorch_qnnp_log_error(
0078:         "failed to create Leaky ReLU operator with [%" PRIu8 ", %" PRIu8
0079:         "] output range: range min must be below range max",
0080:         output_min,
0081:         output_max);
0082:     goto error;
0083:   }
0084: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`pytorch_qnnp_log_error`。

### Lines 85-95 / 第 85-95 行

```c
0085:   status = pytorch_qnnp_status_unsupported_parameter;
0086: 
0087:   const float input_output_scale = input_scale / output_scale;
0088:   if (input_output_scale < 0x1.0p-8f || input_output_scale >= 0x1.0p+8f) {
0089:     pytorch_qnnp_log_error(
0090:         "failed to create Leaky ReLU operator with %.7g input-to-output scale ratio: "
0091:         "scale ratio must be in [2**-8, 2**8) range",
0092:         input_output_scale);
0093:     goto error;
0094:   }
0095: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`pytorch_qnnp_log_error`。

### Lines 96-105 / 第 96-105 行

```c
0096:   status = pytorch_qnnp_status_out_of_memory;
0097: 
0098:   leaky_relu_op = calloc(1, sizeof(struct pytorch_qnnp_operator));
0099:   if (leaky_relu_op == NULL) {
0100:     pytorch_qnnp_log_error(
0101:         "failed to allocate %zu bytes for pytorch_qnnp_operator structure",
0102:         sizeof(struct pytorch_qnnp_operator));
0103:     goto error;
0104:   }
0105: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_operator`, `pytorch_qnnp_log_error`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_operator`, `pytorch_qnnp_log_error`。

### Lines 106-118 / 第 106-118 行

```c
0106:   leaky_relu_op->lookup_table = malloc(256 * sizeof(uint8_t));
0107:   if (leaky_relu_op->lookup_table == NULL) {
0108:     pytorch_qnnp_log_error(
0109:         "failed to allocate 256 bytes for Leaky ReLU lookup table");
0110:     goto error;
0111:   }
0112: 
0113:   uint8_t* lookup_table = leaky_relu_op->lookup_table;
0114:   const float scaled_min_less_zero_point =
0115:       (float)((int32_t)output_min - (int32_t)output_zero_point);
0116:   const float scaled_max_less_zero_point =
0117:       (float)((int32_t)output_max - (int32_t)output_zero_point);
0118:   for (int32_t i = 0; i < 256; i++) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manages memory allocation, buffers, or ownership boundaries. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；管理内存分配、缓冲区或所有权边界。关键符号：`pytorch_qnnp_log_error`。

### Lines 119-130 / 第 119-130 行

```c
0119:     const float x =
0120:         input_output_scale * (float)(i - (int32_t)(uint32_t)input_zero_point);
0121:     float y = x < 0.0f ? x * negative_slope : x;
0122:     if (y < scaled_min_less_zero_point) {
0123:       y = scaled_min_less_zero_point;
0124:     }
0125:     if (y > scaled_max_less_zero_point) {
0126:       y = scaled_max_less_zero_point;
0127:     }
0128:     lookup_table[(uint32_t)i] = (uint8_t)(lrintf(y) + (long)output_zero_point);
0129:   }
0130: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：无明显局部符号。

### Lines 131-141 / 第 131-141 行

```c
0131:   leaky_relu_op->channels = channels;
0132: 
0133:   leaky_relu_op->ukernel_type = pytorch_qnnp_ukernel_type_lut;
0134:   leaky_relu_op->format = pytorch_qnnp_format_quint8;
0135: 
0136:   *leaky_relu_out = leaky_relu_op;
0137:   return pytorch_qnnp_status_success;
0138: 
0139: error:
0140:   pytorch_qnnp_delete_operator(leaky_relu_op);
0141:   return status;
```

- **EN:** This block produces a result or forwards a computed value. Key symbols: `pytorch_qnnp_delete_operator`.
- **CN:** 该代码块返回结果或转发已计算的值。关键符号：`pytorch_qnnp_delete_operator`。

### Lines 142-151 / 第 142-151 行

```c
0142: }
0143: 
0144: enum pytorch_qnnp_status pytorch_qnnp_setup_leaky_relu_nc_q8(
0145:     pytorch_qnnp_operator_t leaky_relu,
0146:     size_t batch_size,
0147:     const uint8_t* input,
0148:     size_t input_stride,
0149:     uint8_t* output,
0150:     size_t output_stride) {
0151:   if (!pytorch_qnnp_params.initialized) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_status`, `pytorch_qnnp_setup_leaky_relu_nc_q8`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_status`, `pytorch_qnnp_setup_leaky_relu_nc_q8`。

### Lines 152-161 / 第 152-161 行

```c
0152:     pytorch_qnnp_log_error(
0153:         "pytorch_qnnp_setup_leaky_relu_nc_q8 failed because QNNPACK is not properly initialized");
0154:     return pytorch_qnnp_status_uninitialized;
0155:   }
0156: 
0157:   if (batch_size == 0) {
0158:     leaky_relu->batch_size = 0;
0159:     return pytorch_qnnp_status_success;
0160:   }
0161: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `pytorch_qnnp_log_error`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`pytorch_qnnp_log_error`。

### Lines 162-169 / 第 162-169 行

```c
0162:   leaky_relu->batch_size = batch_size;
0163:   leaky_relu->input = input;
0164:   leaky_relu->input_pixel_stride = input_stride;
0165:   leaky_relu->output = output;
0166:   leaky_relu->output_pixel_stride = output_stride;
0167: 
0168:   return pytorch_qnnp_status_success;
0169: }
```

- **EN:** This block produces a result or forwards a computed value. Key symbols: no prominent local symbols.
- **CN:** 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Memory allocation strategy** — 内存分配策略
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: pytorch_qnnp_operator, pytorch_qnnp_status, pytorch_qnnp_create_leaky_relu_nc_q8, pytorch_qnnp_log_error, pytorch_qnnp_delete_operator, pytorch_qnnp_setup_leaky_relu_nc_q8** — 核心符号：pytorch_qnnp_operator、pytorch_qnnp_status、pytorch_qnnp_create_leaky_relu_nc_q8、pytorch_qnnp_log_error、pytorch_qnnp_delete_operator、pytorch_qnnp_setup_leaky_relu_nc_q8

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `assert.h`, `math.h`, `stddef.h`, `stdint.h`, `stdlib.h`, `pytorch_qnnpack.h`, `qnnpack/log.h`, `qnnpack/operator.h`
- **Namespaces / 命名空间**: 无
- **Representative symbols / 代表性符号**: `pytorch_qnnp_operator`, `pytorch_qnnp_status`, `pytorch_qnnp_create_leaky_relu_nc_q8`, `pytorch_qnnp_log_error`, `pytorch_qnnp_delete_operator`, `pytorch_qnnp_setup_leaky_relu_nc_q8`
