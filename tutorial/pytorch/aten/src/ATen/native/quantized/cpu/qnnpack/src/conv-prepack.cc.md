# conv-prepack.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/conv-prepack.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `conv-prepack.cc`.
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `conv-prepack.cc` 展开。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15 / 第 1-15 行

```cpp
0001: #include <pytorch_qnnpack.h>
0002: #include <qnnpack/log.h>
0003: #include <qnnpack/operator.h>
0004: #include <qnnpack/pack.h>
0005: #include <qnnpack_func.h>
0006: #include <cstring>
0007: 
0008: namespace qnnpack {
0009: 
0010: PrePackConvWeights::PrePackConvWeights(
0011:     const pytorch_qnnp_operator_t convolution,
0012:     const uint8_t* kernel_zero_points,
0013:     const uint8_t* kernel,
0014:     const int32_t* bias) {
0015:   enum pytorch_qnnp_ukernel_type ukernel_type = convolution->ukernel_type;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_ukernel_type`, `PrePackConvWeights`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_ukernel_type`, `PrePackConvWeights`。

### Lines 16-37 / 第 16-37 行

```cpp
0016:   const uint32_t kernel_width = convolution->kernel_width;
0017:   const uint32_t kernel_height = convolution->kernel_height;
0018:   // deconvolution leaves this 0 for now, remove when deconvolution supports 3d
0019:   const uint32_t kernel_depth =
0020:       convolution->kernel_depth ? convolution->kernel_depth : 1;
0021:   const uint32_t groups = convolution->groups;
0022: 
0023:   if (convolution->transpose &&
0024:       ukernel_type != pytorch_qnnp_ukernel_type_conv) {
0025:     pytorch_qnnp_log_error("Wrong micro-kernel for deconvolution");
0026:     assert(false && "QNNPACK Runtime Error.");
0027:   }
0028: 
0029:   const size_t kernel_size = kernel_height * kernel_width * kernel_depth;
0030:   switch (ukernel_type) {
0031:     case pytorch_qnnp_ukernel_type_dwconv: {
0032:       const uint32_t cr = pytorch_qnnp_params.q8dw9.cr;
0033:       const uint32_t c_stride = (groups + (cr - 1)) & -cr;
0034:       const size_t packed_weights_size =
0035:           (sizeof(uint8_t) * kernel_size + sizeof(int32_t)) * c_stride;
0036:       packed_weights_ = malloc(packed_weights_size);
0037:       if (packed_weights_ == nullptr) {
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries; validates runtime invariants before continuing. Key symbols: `pytorch_qnnp_log_error`, `assert`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界；在继续执行前校验运行时不变量。关键符号：`pytorch_qnnp_log_error`, `assert`。

### Lines 38-65 / 第 38-65 行

```cpp
0038:         pytorch_qnnp_log_error(
0039:             "failed to allocate %zu bytes for packed weights",
0040:             packed_weights_size);
0041:         assert(false && "QNNPACK Runtime Error.");
0042:       }
0043: 
0044:       switch (kernel_size) {
0045:         case 9:
0046:           pytorch_pack_q8dw_wrq(
0047:               kernel_height,
0048:               kernel_width,
0049:               groups,
0050:               cr,
0051:               kernel,
0052:               bias,
0053:               packed_weights_);
0054:           break;
0055:         case 25:
0056:           /* change this later */
0057:           pytorch_pack_q8dw_2d_w_dilation(
0058:               kernel_height,
0059:               kernel_width,
0060:               groups,
0061:               cr,
0062:               0,
0063:               kernel_height,
0064:               0,
0065:               2,
```

- **EN:** This block validates runtime invariants before continuing. Key symbols: `pytorch_qnnp_log_error`, `assert`, `pytorch_pack_q8dw_wrq`.
- **CN:** 该代码块在继续执行前校验运行时不变量。关键符号：`pytorch_qnnp_log_error`, `assert`, `pytorch_pack_q8dw_wrq`。

### Lines 66-93 / 第 66-93 行

```cpp
0066:               kernel,
0067:               bias,
0068:               packed_weights_,
0069:               true);
0070:           pytorch_pack_q8dw_2d_w_dilation(
0071:               kernel_height,
0072:               kernel_width,
0073:               groups,
0074:               cr,
0075:               0,
0076:               kernel_height,
0077:               2,
0078:               4,
0079:               kernel,
0080:               bias,
0081:               (char*)packed_weights_ +
0082:                   (10 + sizeof(int32_t) / sizeof(uint8_t)) * c_stride,
0083:               false);
0084:           pytorch_pack_q8dw_2d_w_dilation(
0085:               kernel_height,
0086:               kernel_width,
0087:               groups,
0088:               cr,
0089:               0,
0090:               kernel_height,
0091:               4,
0092:               5,
0093:               kernel,
```

- **EN:** This block implements local helper logic for `conv-prepack`. Key symbols: `pytorch_pack_q8dw_2d_w_dilation`.
- **CN:** 该代码块实现与 `conv-prepack` 相关的局部辅助逻辑。关键符号：`pytorch_pack_q8dw_2d_w_dilation`。

### Lines 94-121 / 第 94-121 行

```cpp
0094:               bias,
0095:               (char*)packed_weights_ +
0096:                   (20 + sizeof(int32_t) / sizeof(uint8_t)) * c_stride,
0097:               false);
0098:           break;
0099:         case 27:
0100:           pytorch_pack_q8dw_3d_w_dilation(
0101:               kernel_depth,
0102:               kernel_height,
0103:               kernel_width,
0104:               groups,
0105:               cr,
0106:               0,
0107:               kernel_depth,
0108:               0,
0109:               kernel_height,
0110:               0,
0111:               1,
0112:               kernel,
0113:               bias,
0114:               packed_weights_,
0115:               true);
0116:           pytorch_pack_q8dw_3d_w_dilation(
0117:               kernel_depth,
0118:               kernel_height,
0119:               kernel_width,
0120:               groups,
0121:               cr,
```

- **EN:** This block implements local helper logic for `conv-prepack`. Key symbols: `pytorch_pack_q8dw_3d_w_dilation`.
- **CN:** 该代码块实现与 `conv-prepack` 相关的局部辅助逻辑。关键符号：`pytorch_pack_q8dw_3d_w_dilation`。

### Lines 122-149 / 第 122-149 行

```cpp
0122:               0,
0123:               kernel_depth,
0124:               0,
0125:               kernel_height,
0126:               1,
0127:               2,
0128:               kernel,
0129:               bias,
0130:               (char*)packed_weights_ +
0131:                   (kernel_depth * kernel_height +
0132:                    sizeof(int32_t) / sizeof(uint8_t)) *
0133:                       c_stride,
0134:               false);
0135:           pytorch_pack_q8dw_3d_w_dilation(
0136:               kernel_depth,
0137:               kernel_height,
0138:               kernel_width,
0139:               groups,
0140:               cr,
0141:               0,
0142:               kernel_depth,
0143:               0,
0144:               kernel_height,
0145:               2,
0146:               3,
0147:               kernel,
0148:               bias,
0149:               (char*)packed_weights_ +
```

- **EN:** This block implements local helper logic for `conv-prepack`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `conv-prepack` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 150-168 / 第 150-168 行

```cpp
0150:                   (2 * kernel_depth * kernel_height +
0151:                    sizeof(int32_t) / sizeof(uint8_t)) *
0152:                       c_stride,
0153:               false);
0154:           break;
0155:         default:
0156:           PYTORCH_QNNP_UNREACHABLE;
0157:       }
0158:       break;
0159:     }
0160:     case pytorch_qnnp_ukernel_type_xzp_gemm: {
0161:       const uint32_t nr = pytorch_qnnp_params.q8conv_xzp.nr;
0162:       const uint32_t kr = pytorch_qnnp_params.q8conv_xzp.kr;
0163:       const uint32_t sr = pytorch_qnnp_params.q8conv_xzp.kc;
0164:       const uint32_t n_stride =
0165:           (convolution->group_output_channels + (nr - 1)) & -nr;
0166:       const uint32_t k_stride =
0167:           (convolution->group_input_channels + (kr - 1)) & -kr;
0168: 
```

- **EN:** This block implements local helper logic for `conv-prepack`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `conv-prepack` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 169-182 / 第 169-182 行

```cpp
0169:       const size_t packed_group_weights_size =
0170:           (sizeof(uint8_t) * kernel_size * k_stride + sizeof(int32_t)) *
0171:           n_stride;
0172:       packed_weights_ = malloc(packed_group_weights_size * groups);
0173:       if (packed_weights_ == nullptr) {
0174:         pytorch_qnnp_log_error(
0175:             "failed to allocate %zu bytes for packed weights",
0176:             packed_group_weights_size * groups);
0177:         assert(false && "QNNPACK Runtime Error.");
0178:       }
0179:       /* The XZP ukernel needs the padding to be 0 */
0180:       memset(packed_weights_, 0, packed_group_weights_size * groups);
0181: 
0182:       for (uint32_t group = 0; group < groups; group++) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manages memory allocation, buffers, or ownership boundaries; validates runtime invariants before continuing. Key symbols: `pytorch_qnnp_log_error`, `assert`, `memset`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；管理内存分配、缓冲区或所有权边界；在继续执行前校验运行时不变量。关键符号：`pytorch_qnnp_log_error`, `assert`, `memset`。

### Lines 183-205 / 第 183-205 行

```cpp
0183:         pytorch_pack_swizzle_q8gemm_brq(
0184:             convolution->group_output_channels,
0185:             convolution->group_input_channels,
0186:             nr,
0187:             kr,
0188:             sr,
0189:             kernel +
0190:                 group * convolution->group_output_channels *
0191:                     convolution->group_input_channels,
0192:             bias + group * convolution->group_output_channels,
0193:             (void*)((uintptr_t)packed_weights_ + group * packed_group_weights_size));
0194:       }
0195:       break;
0196:     }
0197:     case pytorch_qnnp_ukernel_type_gemm:
0198:     case pytorch_qnnp_ukernel_type_conv: {
0199:       const uint32_t nr = pytorch_qnnp_params.q8conv.nr;
0200:       const uint32_t kr = pytorch_qnnp_params.q8conv.kr;
0201:       const uint32_t n_stride =
0202:           (convolution->group_output_channels + (nr - 1)) & -nr;
0203:       const uint32_t k_stride =
0204:           (convolution->group_input_channels + (kr - 1)) & -kr;
0205: 
```

- **EN:** This block implements local helper logic for `conv-prepack`. Key symbols: `pytorch_pack_swizzle_q8gemm_brq`.
- **CN:** 该代码块实现与 `conv-prepack` 相关的局部辅助逻辑。关键符号：`pytorch_pack_swizzle_q8gemm_brq`。

### Lines 206-222 / 第 206-222 行

```cpp
0206:       const size_t packed_group_weights_size =
0207:           (sizeof(uint8_t) * kernel_size * k_stride + sizeof(int32_t)) *
0208:           n_stride;
0209:       packed_weights_ = malloc(packed_group_weights_size * groups);
0210:       if (packed_weights_ == nullptr) {
0211:         pytorch_qnnp_log_error(
0212:             "failed to allocate %zu bytes for packed weights",
0213:             packed_group_weights_size * groups);
0214:         assert(false && "QNNPACK Runtime Error.");
0215:       }
0216:       // We likely won't needs this once packing functions are appropriately
0217:       // modified. Remove it then.
0218:       memset(
0219:           packed_weights_,
0220:           kernel_zero_points[0],
0221:           packed_group_weights_size * groups);
0222: 
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries; validates runtime invariants before continuing. Key symbols: `pytorch_qnnp_log_error`, `assert`, `memset`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界；在继续执行前校验运行时不变量。关键符号：`pytorch_qnnp_log_error`, `assert`, `memset`。

### Lines 223-241 / 第 223-241 行

```cpp
0223:       switch (ukernel_type) {
0224:         case pytorch_qnnp_ukernel_type_gemm:
0225:           for (uint32_t group = 0; group < groups; group++) {
0226:             pytorch_pack_q8gemm_wrq(
0227:                 convolution->group_output_channels,
0228:                 convolution->group_input_channels,
0229:                 nr,
0230:                 nr,
0231:                 kr,
0232:                 kernel +
0233:                     group * convolution->group_output_channels *
0234:                         convolution->group_input_channels,
0235:                 bias + group * convolution->group_output_channels,
0236:                 kernel_zero_points + group * convolution->group_output_channels,
0237:                 (void*)((uintptr_t)packed_weights_ + group * packed_group_weights_size));
0238:           }
0239:           break;
0240:         case pytorch_qnnp_ukernel_type_conv:  // The transpose can only be here
0241:           for (uint32_t group = 0; group < groups; group++) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: `pytorch_pack_q8gemm_wrq`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：`pytorch_pack_q8gemm_wrq`。

### Lines 242-269 / 第 242-269 行

```cpp
0242:             const uint8_t* const kernel_p = kernel +
0243:                 group * convolution->group_output_channels * kernel_size *
0244:                     convolution->group_input_channels;
0245:             const int32_t* const bias_p =
0246:                 bias + group * convolution->group_output_channels;
0247:             if (convolution
0248:                     ->transpose) { // Note that only runtime packing is here
0249:               pytorch_pack_q8deconv_wrq(
0250:                   convolution->group_output_channels,
0251:                   kernel_size,
0252:                   convolution->group_input_channels,
0253:                   nr,
0254:                   kr,
0255:                   kernel_p,
0256:                   bias_p,
0257:                   kernel_zero_points +
0258:                       group * convolution->group_output_channels,
0259:                   (void*)((uintptr_t)packed_weights_ + group * packed_group_weights_size));
0260:             } else {
0261:               pytorch_pack_q8conv_wrq(
0262:                   convolution->group_output_channels,
0263:                   kernel_size,
0264:                   convolution->group_input_channels,
0265:                   nr,
0266:                   kr,
0267:                   kernel_p,
0268:                   bias_p,
0269:                   kernel_zero_points +
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `pytorch_pack_q8deconv_wrq`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`pytorch_pack_q8deconv_wrq`。

### Lines 270-284 / 第 270-284 行

```cpp
0270:                       group * convolution->group_output_channels,
0271:                   (void*)((uintptr_t)packed_weights_ + group * packed_group_weights_size));
0272:             }
0273:           }
0274:           break;
0275:         default:
0276:           PYTORCH_QNNP_UNREACHABLE;
0277:       }
0278:       break;
0279:     }
0280:     default:
0281:       PYTORCH_QNNP_UNREACHABLE;
0282:   }
0283: } // namespace qnnpack
0284: } // namespace qnnpack
```

- **EN:** This block implements local helper logic for `conv-prepack`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `conv-prepack` 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Memory allocation strategy** — 内存分配策略
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: pytorch_qnnp_ukernel_type, PrePackConvWeights, pytorch_qnnp_log_error, assert, pytorch_pack_q8dw_wrq, pytorch_pack_q8dw_2d_w_dilation, pytorch_pack_q8dw_3d_w_dilation, memset** — 核心符号：pytorch_qnnp_ukernel_type、PrePackConvWeights、pytorch_qnnp_log_error、assert、pytorch_pack_q8dw_wrq、pytorch_pack_q8dw_2d_w_dilation、pytorch_pack_q8dw_3d_w_dilation、memset

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `pytorch_qnnpack.h`, `qnnpack/log.h`, `qnnpack/operator.h`, `qnnpack/pack.h`, `qnnpack_func.h`, `cstring`
- **Namespaces / 命名空间**: `qnnpack`
- **Representative symbols / 代表性符号**: `pytorch_qnnp_ukernel_type`, `PrePackConvWeights`, `pytorch_qnnp_log_error`, `assert`, `pytorch_pack_q8dw_wrq`, `pytorch_pack_q8dw_2d_w_dilation`, `pytorch_pack_q8dw_3d_w_dilation`, `memset`, `pytorch_pack_swizzle_q8gemm_brq`, `pytorch_pack_q8gemm_wrq`, `pytorch_pack_q8deconv_wrq`, `pytorch_pack_q8conv_wrq`
