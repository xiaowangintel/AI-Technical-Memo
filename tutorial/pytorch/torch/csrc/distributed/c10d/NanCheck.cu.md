# NanCheck.cu — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/NanCheck.cu`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for nan check in the c10d distributed process-group subsystem. Key types include `CheckBytePack`, `HasNanFP8x8`. GPU-oriented code paths are present in this file.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供nan check 的实现逻辑。 关键类型包括 `CheckBytePack`、`HasNanFP8x8`。 该文件还包含面向 GPU 的代码路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #include <ATen/Dispatch.h>
2: #include <ATen/cuda/CUDAContext.h>
3: #include <c10/cuda/CUDAGuard.h>
4: #include <stdint.h>
5: #include <torch/library.h>
6: #include <algorithm>
7: 
8: namespace c10d {
9: namespace {
10: 
11: // CUDA kernel to check if data has NAN, device side assert
12: // is raised if NAN is found
13: 
14: // Using ulong2 as a "byte pack", with 16 bytes, for efficient data load
15: union BytePack16 {
16:   ulong2 ul2;
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；包含面向 CUDA 的声明、内核或启动流程。

### Lines 17-32 / 第 17-32 行

```cpp
17:   uint64_t ul[2];
18: };
19: 
20: typedef union BytePack16 BytePack;
21: 
22: //// Start of templated functions for checking NaNs inside a BytePack
23: 
24: // (i) General implementation (aka fallback)
25: // We use a for loop to iterate over the elements in a BytePack.
26: // EltPerPack would be greater than 8 if falling in this case.
27: 
28: template <typename T, int EltPerPack>
29: struct CheckBytePack {
30:   static __device__ __forceinline__ void check(BytePack* tmp) {
31:     T* data = (T*)tmp;
32: #pragma unroll 8
```

- EN: Lines 17-32 declares or defines types such as `CheckBytePack`; introduces executable logic in routines such as `check`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 17-32 行声明或定义了 `CheckBytePack` 等类型；在 `check` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 33-48 / 第 33-48 行

```cpp
33:     for (int i = 0; i < EltPerPack; i++) {
34:       CUDA_KERNEL_ASSERT(!isnan(data[i]));
35:     }
36:   }
37: };
38: 
39: // (ii) Template Specialization for 8-byte data types, e.g. double
40: // EltPerPack = 16 / 8 = 2
41: 
42: template <typename T>
43: struct CheckBytePack<T, /*EltPerPack*/ 2> {
44:   static __device__ __forceinline__ void check(BytePack* tmp) {
45:     T* data = (T*)tmp;
46:     CUDA_KERNEL_ASSERT(!isnan(data[0]) && !isnan(data[1]));
47:   }
48: };
```

- EN: Lines 33-48 declares or defines types such as `CheckBytePack`; introduces executable logic in routines such as `check`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 33-48 行声明或定义了 `CheckBytePack` 等类型；在 `check` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 49-64 / 第 49-64 行

```cpp
49: 
50: // (iii) Template specialization for 4-byte data types, e.g. float32
51: // EltPerPack = 16 / 4 = 4
52: 
53: template <typename T>
54: struct CheckBytePack<T, /*EltPerPack*/ 4> {
55:   static __device__ __forceinline__ void check(BytePack* tmp) {
56:     T* data = (T*)tmp;
57:     CUDA_KERNEL_ASSERT(
58:         !isnan(data[0]) && !isnan(data[1]) && !isnan(data[2]) &&
59:         !isnan(data[3]));
60:   }
61: };
62: 
63: // (iv) Template specialization for 2-byte data types, e.g. float16, bfloat16,
64: // half. EltPerPack = 16 / 2 = 8
```

- EN: Lines 49-64 declares or defines types such as `CheckBytePack`; introduces executable logic in routines such as `check`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 49-64 行声明或定义了 `CheckBytePack` 等类型；在 `check` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 65-80 / 第 65-80 行

```cpp
65: 
66: template <typename T>
67: struct CheckBytePack<T, /*EltPerPack*/ 8> {
68:   static __device__ __forceinline__ void check(BytePack* tmp) {
69:     T* data = (T*)tmp;
70:     CUDA_KERNEL_ASSERT(
71:         !isnan(data[0]) && !isnan(data[1]) && !isnan(data[2]) &&
72:         !isnan(data[3]) && !isnan(data[4]) && !isnan(data[5]) &&
73:         !isnan(data[6]) && !isnan(data[7]));
74:   }
75: };
76: 
77: // (v) Template specialization for Float8 types.
78: // EltPerPack = 16 / 1 = 16
79: 
80: // We want to check 8 x FP8 simultaneously, hence this template definition.
```

- EN: Lines 65-80 declares or defines types such as `CheckBytePack`; introduces executable logic in routines such as `check`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 65-80 行声明或定义了 `CheckBytePack` 等类型；在 `check` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 81-96 / 第 81-96 行

```cpp
81: template <typename T>
82: struct HasNanFP8x8 {
83:   static __device__ __forceinline__ bool check(uint64_t fp8x8) = delete;
84:   /*
85:   {
86:     // `static_assert` in template definition requires c++23 onwards.
87:     // But the error message still applies if you find yourself here.
88:     static_assert(
89:       false,
90:       "You should never call this template definition because it is empty. You "
91:       "can follow the example of Float8_e4m3fn below to implement the check for
92:   " "your new datatype."
93:     );
94:   }
95:   */
96: };
```

- EN: Lines 81-96 declares or defines types such as `HasNanFP8x8`; introduces executable logic in routines such as `static_assert`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 81-96 行声明或定义了 `HasNanFP8x8` 等类型；在 `static_assert` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 97-112 / 第 97-112 行

```cpp
97: 
98: // isnan condition for Float8_e4m3fn:
99: // (x & 0b01111111) == 0b01111111
100: // i.e.
101: // (x & 0x7f) == 0x7f
102: 
103: // The algorithm is as follows:
104: // (1) Mask out the most significant bit with mask 0x7f.
105: // (2) If the result is 0x7f (is nan), the following arithmetic would cause the
106: //     8th bit to be 1: x[i] = x[i] + 0x01
107: // (3) Only leave the 8th bit by masking with 0x80.
108: // (4) If any x[i] is nan, then the whole x != 0.
109: 
110: template <>
111: struct HasNanFP8x8<c10::Float8_e4m3fn> {
112:   static __device__ __forceinline__ bool check(uint64_t fp8x8) {
```

- EN: Lines 97-112 declares or defines types such as `HasNanFP8x8`; introduces executable logic in routines such as `check`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 97-112 行声明或定义了 `HasNanFP8x8` 等类型；在 `check` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 113-128 / 第 113-128 行

```cpp
113:     auto t = fp8x8 & 0x7F7F7F7F7F7F7F7FULL;
114:     auto incremented = t + 0x0101010101010101ULL;
115:     auto overflow = incremented & 0x8080808080808080ULL;
116:     return overflow != 0;
117:   }
118: };
119: 
120: // isnan condition for Float8_e5m2:
121: // (x & 0x7f) > 0x7c
122: // This case does not overflow: 0x7c + 0x03 == 0x7f but adding 0x03 to anything
123: // greater than 0x7c will overflow.
124: 
125: template <>
126: struct HasNanFP8x8<c10::Float8_e5m2> {
127:   static __device__ __forceinline__ bool check(uint64_t fp8x8) {
128:     auto t = fp8x8 & 0x7F7F7F7F7F7F7F7FULL;
```

- EN: Lines 113-128 declares or defines types such as `HasNanFP8x8`; introduces executable logic in routines such as `check`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 113-128 行声明或定义了 `HasNanFP8x8` 等类型；在 `check` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 129-144 / 第 129-144 行

```cpp
129:     auto incremented = t + 0x0303030303030303ULL;
130:     auto overflow = incremented & 0x8080808080808080ULL;
131:     return overflow != 0;
132:   }
133: };
134: 
135: template <typename T>
136: struct CheckBytePack<T, /*EltPerPack*/ 16> {
137:   static __device__ __forceinline__ void check(BytePack* tmp) {
138:     CUDA_KERNEL_ASSERT(
139:         !HasNanFP8x8<T>::check(tmp->ul[0]) &&
140:         !HasNanFP8x8<T>::check(tmp->ul[1]));
141:   }
142: };
143: 
144: //// End of templated functions for checking NaNs inside a BytePack
```

- EN: Lines 129-144 declares or defines types such as `CheckBytePack`; introduces executable logic in routines such as `check`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 129-144 行声明或定义了 `CheckBytePack` 等类型；在 `check` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 145-160 / 第 145-160 行

```cpp
145: 
146: // Fast-path check routine:
147: // each thread will load and check 8 BytePacks in this routine
148: 
149: // Create a tmp buffer of size 8, also unroll for loop by 8
150: #define UNROLL 8
151: 
152: template <typename T>
153: __device__ __forceinline__ void checkChunk(BytePack* ptr) {
154:   BytePack tmp[UNROLL];
155:   int nWorkers = blockDim.x * gridDim.x;
156: // First load values from global memory into tmp buffer
157: #pragma unroll 8
158:   for (int j = 0; j < UNROLL; j++) {
159:     tmp[j] = ptr[nWorkers * j];
160:   }
```

- EN: Lines 145-160 introduces executable logic in routines such as `checkChunk`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 145-160 行在 `checkChunk` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 161-176 / 第 161-176 行

```cpp
161: // Then check each BytePack in the tmp buffer
162: #pragma unroll 8
163:   for (int j = 0; j < UNROLL; j++) {
164:     CheckBytePack<T, sizeof(BytePack) / sizeof(T)>::check(tmp + j);
165:   }
166:   // Note: we separate the check from the load for efficient loading
167: }
168: 
169: // Align address of `ptr` up, to the alignment of `T`
170: #define ALIGN_UP(ptr, T) \
171:   (((uintptr_t)ptr + sizeof(T) - 1) / sizeof(T) * sizeof(T))
172: 
173: // This is the host-facing kernel
174: 
175: template <typename T>
176: __global__ void checkForNaN(T* data, size_t size) {
```

- EN: Lines 161-176 introduces executable logic in routines such as `checkForNaN`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 161-176 行在 `checkForNaN` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 177-192 / 第 177-192 行

```cpp
177:   constexpr int EltPerPack = sizeof(BytePack) / sizeof(T);
178:   // Offset of current thread
179:   size_t offset = blockIdx.x * blockDim.x + threadIdx.x;
180: 
181:   // Align input address up to BytePack in case it is not
182:   T* ptrAlign = (T*)ALIGN_UP(data, BytePack);
183:   // Pre-process the data before alignment
184:   size_t preProcElts = min(ptrAlign - data, size);
185:   // Read memory by T (slow). One iter is enough bc the number of threads would
186:   // be bigger than `preProcElts`
187:   if (offset < preProcElts) {
188:     CUDA_KERNEL_ASSERT(!isnan(data[offset]));
189:   }
190:   // We have processes this amount of data
191:   size -= preProcElts;
192: 
```

- EN: Lines 177-192 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 177-192 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 193-208 / 第 193-208 行

```cpp
193:   // Start BytePack processing
194:   BytePack* ptr = (BytePack*)ptrAlign;
195:   // Size of input data in unit of BytePack
196:   size_t sizeInBP = size * sizeof(T) / sizeof(BytePack);
197:   // Number of BytePacks processed in one fast-path iteration
198:   size_t loopSize = blockDim.x * gridDim.x * UNROLL;
199: 
200:   // Fast path
201:   // The condition below makes sure there is enough data to process (`loopSize`)
202:   for (; offset + loopSize <= sizeInBP; offset += loopSize) {
203:     checkChunk<T>(ptr + offset);
204:   }
205: 
206:   // The rest data goes on slow path
207:   // We just do regular load and check
208:   for (; offset < sizeInBP; offset += blockDim.x * gridDim.x) {
```

- EN: Lines 193-208 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 193-208 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 209-224 / 第 209-224 行

```cpp
209:     BytePack tmp = ptr[offset];
210:     CheckBytePack<T, EltPerPack>::check(&tmp);
211:   }
212: 
213:   // We can still have a tail smaller than 1 BytePack
214:   // TODO: merge this tail check with head check to make them concurrent
215:   if (threadIdx.x < size % EltPerPack) {
216:     T* tailPtr = (T*)(ptr + sizeInBP);
217:     CUDA_KERNEL_ASSERT(!isnan(tailPtr[threadIdx.x]));
218:   }
219: }
220: 
221: void check_for_nan_cuda(const at::Tensor& tensor) {
222:   if (!tensor.is_floating_point()) {
223:     return;
224:   }
```

- EN: Lines 209-224 introduces executable logic in routines such as `check_for_nan_cuda`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 209-224 行在 `check_for_nan_cuda` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 225-240 / 第 225-240 行

```cpp
225:   const size_t maxNumThreadsPerBlock = 512;
226:   const size_t maxNumBlocks = 24;
227:   const size_t numThreadsPerBlock =
228:       std::min<size_t>(maxNumThreadsPerBlock, tensor.numel());
229: 
230:   if (!(numThreadsPerBlock > 0)) {
231:     return;
232:   }
233: 
234:   const size_t numBlocks = std::min<size_t>(
235:       maxNumBlocks,
236:       (tensor.numel() + numThreadsPerBlock - 1) / numThreadsPerBlock);
237: 
238:   auto stream = at::cuda::getCurrentCUDAStream(tensor.device().index());
239: 
240:   AT_DISPATCH_FLOATING_TYPES_AND4(
```

- EN: Lines 225-240 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 225-240 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 241-256 / 第 241-256 行

```cpp
241:       at::ScalarType::Half,
242:       at::ScalarType::BFloat16,
243:       at::ScalarType::Float8_e4m3fn,
244:       at::ScalarType::Float8_e5m2,
245:       tensor.scalar_type(),
246:       "checkForNaN",
247:       [&] {
248:         checkForNaN<scalar_t><<<numBlocks, numThreadsPerBlock, 0, stream>>>(
249:             tensor.data_ptr<scalar_t>(), tensor.numel());
250:         C10_CUDA_KERNEL_LAUNCH_CHECK();
251:       });
252: }
253: 
254: TORCH_LIBRARY_IMPL(c10d, CUDA, m) {
255:   m.impl("check_for_nan", check_for_nan_cuda);
256: }
```

- EN: Lines 241-256 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 241-256 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 257-259 / 第 257-259 行

```cpp
257: 
258: } // namespace
259: } // namespace c10d
```

- EN: Lines 257-259 opens or closes namespaces to place the code in the correct distributed component; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 257-259 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；包含面向 CUDA 的声明、内核或启动流程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `CheckBytePack`, `HasNanFP8x8`
- CN: 核心符号：`CheckBytePack`、`HasNanFP8x8`
- EN: Notable themes: CUDA paths.
- CN: 值得关注的主题：CUDA 路径。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: None / 无
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/Dispatch.h`, `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDAGuard.h`, `torch/library.h`
- External or system headers / 外部或系统头文件: `stdint.h`, `algorithm`
- Local symbols / 本地符号: `CheckBytePack`, `HasNanFP8x8`