# IndexKernels.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/mps/IndexKernels.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Apple MPS-backed ATen operators, resource management, or execution helpers. This specific file centers on `IndexKernels.h`.
- **Purpose (CN)**: 实现基于 Apple MPS 的 ATen 算子、资源管理或执行辅助逻辑。 该文件具体围绕 `IndexKernels.h` 展开。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21 / 第 1-21 行

```cpp
0001: #pragma once
0002: 
0003: namespace at::mps {
0004: 
0005: static const char* SCATTER_OPS_TEMPLATE = R"METAL_SCATTER(
0006: template<typename Y, typename X>
0007: Y cast(const X x);
0008: 
0009: template<>
0010: {1} cast<{1}, {0}>(const {0} x) {{
0011:  return {2};
0012: }}
0013: 
0014: kernel void scatter_kernel_n(uint linear_index          [[thread_position_in_grid]],
0015:                              constant void * src_       [[buffer(0)]],
0016:                              device void * dst_         [[buffer(1)]],
0017:                              constant uint32_t * size   [[buffer(2)]],
0018:                              constant uint32_t * stride [[buffer(3)]],
0019:                             constant uint32_t & numel   [[buffer(4)]],
0020:                             constant int32_t & ndim     [[buffer(5)]]) {{
0021:     if (linear_index >= numel) return;
```

- **EN:** Builds a reusable template/helper layer around `IndexKernels`. Key symbols: `cast`, `scatter_kernel_n`.
- **CN:** 围绕 `IndexKernels` 构建可复用的模板或辅助层。关键符号：`cast`, `scatter_kernel_n`。

### Lines 22-35 / 第 22-35 行

```cpp
0022: 
0023:     constant {0} * src = (constant {0} *)src_;
0024:     device {1} * dst = (device {1} *)dst_;
0025: 
0026:     uint64_t dst_offs = 0;
0027:     auto dst_idx = linear_index;
0028:     for(int dim = ndim - 1; dim >= 0; --dim) {{
0029:       dst_offs += stride[dim] * (dst_idx % size[dim]);
0030:       dst_idx /= size[dim];
0031:     }}
0032: 
0033:     dst[dst_offs] = cast<{1}>(src[linear_index]);
0034: }}
0035: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 36-52 / 第 36-52 行

```cpp
0036: kernel void scatter_kernel_4(uint linear_index              [[thread_position_in_grid]],
0037:                              constant void * src_           [[buffer(0)]],
0038:                              device void * dst_             [[buffer(1)]],
0039:                              constant packed_uint4 & size   [[buffer(2)]],
0040:                              constant packed_uint4 & stride [[buffer(3)]],
0041:                              constant uint32_t & numel      [[buffer(4)]]) {{
0042:     if (linear_index >= numel) return;
0043: 
0044:     constant {0} * src = (constant {0} *)src_;
0045:     device {1} * dst = (device {1} *)dst_;
0046: 
0047:     packed_uint4 local_index;
0048:     local_index.x = linear_index / (size[3] * size[2] * size[1]) % size[0];
0049:     local_index.y = linear_index / (size[3] * size[2]) % size[1];
0050:     local_index.z = linear_index / size[3] % size[2];
0051:     local_index.w = linear_index % size[3];
0052: 
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries. Key symbols: `scatter_kernel_4`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界。关键符号：`scatter_kernel_4`。

### Lines 53-67 / 第 53-67 行

```cpp
0053:     const packed_uint4 strided_index = local_index * stride;
0054:     dst[strided_index.x + strided_index.y + strided_index.z + strided_index.w] = cast<{1}>(src[linear_index]);
0055: }}
0056: 
0057: kernel void scatter_kernel_3(uint linear_index              [[thread_position_in_grid]],
0058:                              constant void * src_           [[buffer(0)]],
0059:                              device void * dst_             [[buffer(1)]],
0060:                              constant packed_uint3 & size   [[buffer(2)]],
0061:                              constant packed_uint3 & stride [[buffer(3)]],
0062:                              constant uint32_t & numel      [[buffer(4)]]) {{
0063:     if (linear_index >= numel) return;
0064: 
0065:     constant {0} * src = (constant {0} *)src_;
0066:     device {1} * dst = (device {1} *)dst_;
0067: 
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries. Key symbols: `scatter_kernel_3`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界。关键符号：`scatter_kernel_3`。

### Lines 68-83 / 第 68-83 行

```cpp
0068:     packed_uint3 local_index;
0069:     local_index.x = linear_index / (size[2] * size[1]) % size[0];
0070:     local_index.y = linear_index / size[2] % size[1];
0071:     local_index.z = linear_index % size[2];
0072: 
0073:     const packed_uint3 strided_index = local_index * stride;
0074:     dst[strided_index.x + strided_index.y + strided_index.z] = cast<{1}>(src[linear_index]);
0075: }}
0076: 
0077: kernel void scatter_kernel_2(uint linear_index              [[thread_position_in_grid]],
0078:                              constant void * src_           [[buffer(0)]],
0079:                              device void * dst_             [[buffer(1)]],
0080:                              constant packed_uint2 & size   [[buffer(2)]],
0081:                              constant packed_uint2 & stride [[buffer(3)]],
0082:                              constant uint32_t & numel      [[buffer(4)]]) {{
0083:     if (linear_index >= numel) return;
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries. Key symbols: `scatter_kernel_2`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界。关键符号：`scatter_kernel_2`。

### Lines 84-102 / 第 84-102 行

```cpp
0084: 
0085:     constant {0} * src = (constant {0} *)src_;
0086:     device {1} * dst = (device {1} *)dst_;
0087: 
0088:     packed_uint2 local_index;
0089:     local_index.x = linear_index / size[1] % size[0];
0090:     local_index.y = linear_index % size[1];
0091: 
0092:     const packed_uint2 strided_index = local_index * stride;
0093:     dst[strided_index.x + strided_index.y] = cast<{1}>(src[linear_index]);
0094: }}
0095: 
0096: kernel void scatter_kernel_1(uint linear_index              [[thread_position_in_grid]],
0097:                              constant void * src_           [[buffer(0)]],
0098:                              device void * dst_             [[buffer(1)]],
0099:                              constant int & size            [[buffer(2)]],
0100:                              constant int & stride          [[buffer(3)]],
0101:                              constant uint32_t & numel      [[buffer(4)]]) {{
0102:     if (linear_index >= numel) return;
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries. Key symbols: `scatter_kernel_1`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界。关键符号：`scatter_kernel_1`。

### Lines 103-116 / 第 103-116 行

```cpp
0103: 
0104:     constant {0} * src = (constant {0} *)src_;
0105:     device {1} * dst = (device {1} *)dst_;
0106: 
0107:     const int local_index = linear_index % size;
0108:     const int strided_index = local_index * stride;
0109:     dst[strided_index] = cast<{1}>(src[linear_index]);
0110: }}
0111: )METAL_SCATTER";
0112: 
0113: static const char* GATHER_OPS_TEMPLATE = R"METAL_GATHER(
0114: template<typename Y, typename X>
0115: Y cast(const X x);
0116: 
```

- **EN:** Builds a reusable template/helper layer around `IndexKernels`. Key symbols: `cast`.
- **CN:** 围绕 `IndexKernels` 构建可复用的模板或辅助层。关键符号：`cast`。

### Lines 117-130 / 第 117-130 行

```cpp
0117: template<>
0118: {1} cast<{1}, {0}>(const {0} x) {{
0119:  return {2};
0120: }}
0121: 
0122: kernel void gather_kernel_n(uint linear_index           [[thread_position_in_grid]],
0123:                             constant void * src_        [[buffer(0)]],
0124:                             device void * dst_          [[buffer(1)]],
0125:                             constant uint32_t * size    [[buffer(2)]],
0126:                             constant uint32_t * stride  [[buffer(3)]],
0127:                             constant uint32_t & numel   [[buffer(4)]],
0128:                             constant int32_t & ndim     [[buffer(5)]]) {{
0129:     if (linear_index >= numel) return;
0130: 
```

- **EN:** Builds a reusable template/helper layer around `IndexKernels`. Key symbols: `gather_kernel_n`.
- **CN:** 围绕 `IndexKernels` 构建可复用的模板或辅助层。关键符号：`gather_kernel_n`。

### Lines 131-150 / 第 131-150 行

```cpp
0131:     constant {0} * src = (constant {0} *)src_;
0132:     device {1} * dst = (device {1} *)dst_;
0133: 
0134:     uint64_t src_offs = 0;
0135:     auto src_idx = linear_index;
0136:     for(int dim = ndim - 1; dim >= 0; --dim) {{
0137:       src_offs += stride[dim] * (src_idx % size[dim]);
0138:       src_idx /= size[dim];
0139:     }}
0140: 
0141:     dst[linear_index] = cast<{1}>(src[src_offs]);
0142: }}
0143: 
0144: kernel void gather_kernel_4(uint linear_index               [[thread_position_in_grid]],
0145:                             constant void * src_            [[buffer(0)]],
0146:                             device void * dst_              [[buffer(1)]],
0147:                             constant packed_uint4 & size    [[buffer(2)]],
0148:                             constant packed_uint4 & stride  [[buffer(3)]],
0149:                             constant uint32_t & numel       [[buffer(4)]]) {{
0150:     if (linear_index >= numel) return;
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manages memory allocation, buffers, or ownership boundaries. Key symbols: `gather_kernel_4`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；管理内存分配、缓冲区或所有权边界。关键符号：`gather_kernel_4`。

### Lines 151-164 / 第 151-164 行

```cpp
0151: 
0152:     constant {0} * src = (constant {0} *)src_;
0153:     device {1} * dst = (device {1} *)dst_;
0154: 
0155:     packed_uint4 local_index;
0156:     local_index.x = linear_index / (size[3] * size[2] * size[1]) % size[0];
0157:     local_index.y = linear_index / (size[3] * size[2]) % size[1];
0158:     local_index.z = linear_index / size[3] % size[2];
0159:     local_index.w = linear_index % size[3];
0160: 
0161:     const packed_uint4 strided_index = local_index * stride;
0162:     dst[linear_index] = cast<{1}>(src[strided_index.x + strided_index.y + strided_index.z + strided_index.w]);
0163: }}
0164: 
```

- **EN:** This block implements local helper logic for `IndexKernels`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `IndexKernels` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 165-180 / 第 165-180 行

```cpp
0165: kernel void gather_kernel_3(uint linear_index               [[thread_position_in_grid]],
0166:                             constant void * src_            [[buffer(0)]],
0167:                             device void * dst_              [[buffer(1)]],
0168:                             constant packed_uint3 & size    [[buffer(2)]],
0169:                             constant packed_uint3 & stride  [[buffer(3)]],
0170:                             constant uint32_t & numel       [[buffer(4)]]) {{
0171:     if (linear_index >= numel) return;
0172: 
0173:     constant {0} * src = (constant {0} *)src_;
0174:     device {1} * dst = (device {1} *)dst_;
0175: 
0176:     packed_uint3 local_index;
0177:     local_index.x = linear_index / (size[2] * size[1]) % size[0];
0178:     local_index.y = linear_index / size[2] % size[1];
0179:     local_index.z = linear_index % size[2];
0180: 
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries. Key symbols: `gather_kernel_3`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界。关键符号：`gather_kernel_3`。

### Lines 181-195 / 第 181-195 行

```cpp
0181:     const packed_uint3 strided_index = local_index * stride;
0182:     dst[linear_index] = cast<{1}>(src[strided_index.x + strided_index.y + strided_index.z]);
0183: }}
0184: 
0185: kernel void gather_kernel_2(uint linear_index               [[thread_position_in_grid]],
0186:                             constant void * src_            [[buffer(0)]],
0187:                             device void * dst_              [[buffer(1)]],
0188:                             constant packed_uint2 & size    [[buffer(2)]],
0189:                             constant packed_uint2 & stride  [[buffer(3)]],
0190:                             constant uint32_t & numel       [[buffer(4)]]) {{
0191:     if (linear_index >= numel) return;
0192: 
0193:     constant {0} * src = (constant {0} *)src_;
0194:     device {1} * dst = (device {1} *)dst_;
0195: 
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries. Key symbols: `gather_kernel_2`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界。关键符号：`gather_kernel_2`。

### Lines 196-210 / 第 196-210 行

```cpp
0196:     packed_uint2 local_index;
0197:     local_index.x = linear_index / size[1] % size[0];
0198:     local_index.y = linear_index % size[1];
0199: 
0200:     const packed_uint2 strided_index = local_index * stride;
0201:     dst[linear_index] = cast<{1}>(src[strided_index.x + strided_index.y]);
0202: }}
0203: 
0204: kernel void gather_kernel_1(uint linear_index               [[thread_position_in_grid]],
0205:                             constant void * src_            [[buffer(0)]],
0206:                             device void * dst_              [[buffer(1)]],
0207:                             constant int & size             [[buffer(2)]],
0208:                             constant int & stride           [[buffer(3)]],
0209:                             constant uint32_t & numel       [[buffer(4)]]) {{
0210:     if (linear_index >= numel) return;
```

- **EN:** This block handles conditional branches and special cases; manages memory allocation, buffers, or ownership boundaries. Key symbols: `gather_kernel_1`.
- **CN:** 该代码块处理条件分支与特殊情况；管理内存分配、缓冲区或所有权边界。关键符号：`gather_kernel_1`。

### Lines 211-220 / 第 211-220 行

```cpp
0211: 
0212:     constant {0} * src = (constant {0} *)src_;
0213:     device {1} * dst = (device {1} *)dst_;
0214: 
0215:     const int local_index = linear_index % size;
0216:     const int strided_index = local_index * stride;
0217:     dst[linear_index] = cast<{1}>(src[strided_index]);
0218: }}
0219: )METAL_GATHER";
0220: } // namespace at::mps
```

- **EN:** This block connects ATen logic to backend library/resource abstractions. Key symbols: no prominent local symbols.
- **CN:** 该代码块把 ATen 逻辑连接到后端库或资源抽象。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **Apple MPS support** — Apple MPS 支持
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Apple accelerator integration** — Apple 加速后端集成
- **Core symbols: cast, scatter_kernel_n, scatter_kernel_4, scatter_kernel_3, scatter_kernel_2, scatter_kernel_1, gather_kernel_n, gather_kernel_4** — 核心符号：cast、scatter_kernel_n、scatter_kernel_4、scatter_kernel_3、scatter_kernel_2、scatter_kernel_1、gather_kernel_n、gather_kernel_4

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at::mps`
- **Representative symbols / 代表性符号**: `cast`, `scatter_kernel_n`, `scatter_kernel_4`, `scatter_kernel_3`, `scatter_kernel_2`, `scatter_kernel_1`, `gather_kernel_n`, `gather_kernel_4`, `gather_kernel_3`, `gather_kernel_2`, `gather_kernel_1`
