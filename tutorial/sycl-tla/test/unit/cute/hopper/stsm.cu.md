# stsm.cu — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `test/unit/cute/hopper/stsm.cu`
- **EN:** Hopper tests for STSM or store-matrix shared-memory atoms and tiled copies.
- **CN:** 本文件围绕 `stsm` 相关功能编写单元测试或辅助基架。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31
```cpp
   1: /***************************************************************************************************
   2:  * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
   3:  * Copyright (C) 2025 Intel Corporation, All rights reserved.
   4:  * SPDX-License-Identifier: BSD-3-Clause
   5:  *
   6:  * Redistribution and use in source and binary forms, with or without
   7:  * modification, are permitted provided that the following conditions are met:
   8:  *
   9:  * 1. Redistributions of source code must retain the above copyright notice, this
  10:  * list of conditions and the following disclaimer.
  11:  *
  12:  * 2. Redistributions in binary form must reproduce the above copyright notice,
  13:  * this list of conditions and the following disclaimer in the documentation
  14:  * and/or other materials provided with the distribution.
  15:  *
  16:  * 3. Neither the name of the copyright holder nor the names of its
  17:  * contributors may be used to endorse or promote products derived from
  18:  * this software without specific prior written permission.
  19:  *
  20:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  21:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  22:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  23:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  24:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  25:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  26:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  27:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  28:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  29:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  30:  *
  31:  **************************************************************************************************/
```
**EN:** Records copyright ownership and the BSD-3-Clause licensing terms for this source file.
**CN:** 记录该源文件的版权归属以及 BSD-3-Clause 许可条款。

### Lines 33
```cpp
  33: #include "cutlass_unit_test.h"
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 35
```cpp
  35: #include <iostream>
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 37-38
```cpp
  37: #include <cute/tensor.hpp>
  38: #include <cute/arch/copy_sm90.hpp>
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 40-44
```cpp
  40: #if defined(CUTLASS_ENABLE_SYCL)
  41: namespace sc = compat;
  42: namespace sc_exp = compat::experimental;
  43: namespace sycl_ext = sycl::ext::oneapi::experimental;
  44: #endif
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Keeps a SYCL-compatible code path alongside CUDA-specific logic.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 在 CUDA 专用逻辑旁保留兼容 SYCL 的实现路径。

### Lines 46
```cpp
  46: using namespace cute;
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。

### Lines 48-54
```cpp
  48: template<class T>
  49: CUTLASS_GLOBAL void
  50: stsm_test_device(uint16_t* g_in, uint16_t* g_out)
  51: {
  52:   constexpr int count = sizeof(T) / 4;
  53:   int tid = ThreadIdxX();
  54:   int stride = BlockDimX();
```
**EN:** Defines helper type `T` used by the surrounding tests or kernels. Uses per-thread or per-subgroup indices to distribute work across the execution unit.
**CN:** 定义辅助类型 `T`，供周围测试或内核复用。 使用线程或子组索引把工作分配到执行单元。

### Lines 56-60
```cpp
  56:   // load input gmem -> rmem
  57:   uint32_t reg[count];
  58:   for (int i = 0; i < (sizeof(T) / 4); i++) {
  59:     reg[i] = reinterpret_cast<uint32_t*>(g_in)[tid + (stride * i)];
  60:   }
```
**EN:** Continues the procedural logic of the current helper or test case. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 初始化测试场景所需的确定性或随机输入数据。

### Lines 62-70
```cpp
  62:   #if defined(__SYCL_DEVICE_ONLY__)
  63:     auto smem = sycl_ext::get_dynamic_work_group_memory<uint32_t>().get();
  64:   #endif
  65:   #if defined(CUTLASS_ENABLE_SYCL) && !defined(__SYCL_DEVICE_ONLY__)
  66:     uint32_t* smem; // dummy declaration to avoid compilation errors during the host compilation phase
  67:   #endif
  68:   #if !defined(CUTLASS_ENABLE_SYCL)
  69:     CUTLASS_SHARED uint32_t smem[32 * count];
  70:   #endif
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Keeps a SYCL-compatible code path alongside CUDA-specific logic.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 在 CUDA 专用逻辑旁保留兼容 SYCL 的实现路径。

### Lines 72-75
```cpp
  72:   // load rmem -> smem using STSM
  73:   uint128_t* smem_ptr = reinterpret_cast<uint128_t*>(smem) + tid;
  74:   T*         rmem_ptr = reinterpret_cast<T*>(reg);
  75:   cute::copy_stsm(rmem_ptr, smem_ptr);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 77
```cpp
  77:   syncthreads();
```
**EN:** Continues the procedural logic of the current helper or test case. Synchronizes execution so later reads observe completed writes or completed transfers.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 通过同步确保后续读取能够看到已经完成的写入或传输。

### Lines 79-83
```cpp
  79:   // store output smem -> gmem
  80:   for (int i = 0; i < (sizeof(T) / 4); i++) {
  81:     reinterpret_cast<uint32_t*>(g_out)[tid + (stride * i)] = smem[tid + (stride * i)];
  82:   }
  83: }
```
**EN:** Continues the procedural logic of the current helper or test case. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 初始化测试场景所需的确定性或随机输入数据。

### Lines 85-90
```cpp
  85: template <class TiledCopy, class SmemLayout>
  86: CUTLASS_GLOBAL void
  87: stsm_test_device_cute(uint16_t* g_in, uint16_t* g_out,
  88:                       TiledCopy tiled_copy, SmemLayout smem_layout)
  89: {
  90:   using namespace cute;
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。

### Lines 92-100
```cpp
  92:   #if defined(__SYCL_DEVICE_ONLY__)
  93:     auto smem = sycl_ext::get_dynamic_work_group_memory<uint16_t>().get();
  94:   #endif
  95:   #if defined(CUTLASS_ENABLE_SYCL) && !defined(__SYCL_DEVICE_ONLY__)
  96:     char* smem_buf; // dummy declaration to avoid compilation errors during the host compilation phase
  97:   #endif
  98:   #if !defined(CUTLASS_ENABLE_SYCL)
  99:     CUTLASS_SHARED uint16_t smem[size(smem_layout)];
 100:   #endif
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Keeps a SYCL-compatible code path alongside CUDA-specific logic.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 在 CUDA 专用逻辑旁保留兼容 SYCL 的实现路径。

### Lines 102-104
```cpp
 102:   Tensor t_g_in  = make_tensor(make_gmem_ptr(g_in),  smem_layout);
 103:   Tensor t_g_out = make_tensor(make_gmem_ptr(g_out), smem_layout);
 104:   Tensor t_smem  = make_tensor(make_smem_ptr(smem),  smem_layout);
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。

### Lines 106
```cpp
 106:   int tid = ThreadIdxX();
```
**EN:** Continues the procedural logic of the current helper or test case. Uses per-thread or per-subgroup indices to distribute work across the execution unit.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 使用线程或子组索引把工作分配到执行单元。

### Lines 108
```cpp
 108:   auto thr_copy = tiled_copy.get_thread_slice(tid);
```
**EN:** Continues the procedural logic of the current helper or test case. Partitions tensors so each thread, subgroup, or copy atom sees only its own slice.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 对张量进行分块，使每个线程、子组或拷贝原子只处理自己的切片。

### Lines 110-111
```cpp
 110:   Tensor tXgX = thr_copy.partition_S(t_g_in);   // (V,M,N)
 111:   Tensor tXsX = thr_copy.partition_D(t_smem);   // (V,M,N)
```
**EN:** Continues the procedural logic of the current helper or test case. Partitions tensors so each thread, subgroup, or copy atom sees only its own slice.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 对张量进行分块，使每个线程、子组或拷贝原子只处理自己的切片。

### Lines 113-114
```cpp
 113:   Tensor tXrX = make_tensor<uint16_t>(shape(tXgX)); // (V,M,N)
 114:   clear(tXrX);    // Just to make sure
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。

### Lines 116-122
```cpp
 116: /*
 117:   if (thread0()) {
 118:     print("tXsX: " ); print(tXsX.layout()); print("\n");
 119:     print("tXgX: " ); print(tXgX.layout()); print("\n");
 120:     print("tXrX: " ); print(tXrX.layout()); print("\n");
 121:   }
 122: */
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 124-125
```cpp
 124:   // Load input gmem -> rmem
 125:   copy(tXgX, tXrX);
```
**EN:** Continues the procedural logic of the current helper or test case. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 127-128
```cpp
 127:   // Copy rmem -> smem via tiled_copy (STSM, STS)
 128:   copy(tiled_copy, tXrX, tXsX);
```
**EN:** Continues the procedural logic of the current helper or test case. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 130-134
```cpp
 130:   // Output smem -> gmem
 131:   for (int i = tid; i < size(t_smem); i += size(tiled_copy)) {
 132:     t_g_out(i) = t_smem(i);
 133:   }
 134: }
```
**EN:** Continues the procedural logic of the current helper or test case. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 初始化测试场景所需的确定性或随机输入数据。

### Lines 136-139
```cpp
 136: #if CUDA_12_0_SM90_FEATURES_SUPPORTED
 137: TEST(SM90_CuTe_Hopper, Stsm)
 138: {
 139:   constexpr int count = 1024;
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。

### Lines 141-145
```cpp
 141:   host_vector<uint16_t> h_in(count);
 142:   for (int i = 0; i < count; ++i) {
 143:     h_in[i] = uint16_t(i);
 144:   }
 145:   device_vector<uint16_t> d_in = h_in;
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 初始化测试场景所需的确定性或随机输入数据。

### Lines 147-170
```cpp
 147:   //
 148:   // STSM 1x (32b)
 149:   //
 151:   {
 152:   device_vector<uint16_t> d_out(count);
 153:   #if defined(CUTLASS_ENABLE_SYCL)
 154:   sc_exp::launch<stsm_test_device<uint32_t>>
 155:   ( sc::launch_policy{sc::dim3(1), sc::dim3(32),
 156:     sc_exp::launch_properties{sycl_ext::work_group_static_size(sizeof(uint32_t) / 4 * 32)}},
 157:     d_in.data(), d_out.data());
 158:   sc::wait_and_throw();
 159:   #else
 160:   stsm_test_device<uint32_t><<<1, 32>>>(
 161:     thrust::raw_pointer_cast(d_in.data()),
 162:     thrust::raw_pointer_cast(d_out.data()));
 163:   #endif
 164:   host_vector<uint16_t> h_out = d_out;
 165:   for (int i = 0; i < 32; ++i) {
 166:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 167:     EXPECT_EQ(h_out[i], h_in[i]);
 168:   }
 169:   CUTLASS_TRACE_HOST("STSM 1x stsm_test_device SUCCESS\n");
 170:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Launches a device kernel through the SYCL or CUDA execution path. Validates results immediately so the test fails close to the source of an error.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 通过 SYCL 或 CUDA 执行路径启动设备内核。 通过即时断言验证结果，使测试在最接近错误源的位置失败。

### Lines 172-195
```cpp
 172:   //
 173:   // STSM 2x (64b)
 174:   //
 176:   {
 177:   device_vector<uint16_t> d_out(count);
 178:   #if defined(CUTLASS_ENABLE_SYCL)
 179:   sc_exp::launch<stsm_test_device<uint64_t>>
 180:   ( sc::launch_policy{sc::dim3(1), sc::dim3(32),
 181:     sc_exp::launch_properties{sycl_ext::work_group_static_size(sizeof(uint64_t) / 4 * 32)}},
 182:     d_in.data(), d_out.data());
 183:   sc::wait_and_throw();
 184:   #else
 185:   stsm_test_device<uint64_t><<<1, 32>>>(
 186:     thrust::raw_pointer_cast(d_in.data()),
 187:     thrust::raw_pointer_cast(d_out.data()));
 188:   #endif
 189:   host_vector<uint16_t> h_out = d_out;
 190:   for (int i = 0; i < 64; ++i) {
 191:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 192:     EXPECT_EQ(h_out[i], h_in[i]);
 193:   }
 194:   CUTLASS_TRACE_HOST("STSM 2x stsm_test_device SUCCESS\n");
 195:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Launches a device kernel through the SYCL or CUDA execution path. Validates results immediately so the test fails close to the source of an error.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 通过 SYCL 或 CUDA 执行路径启动设备内核。 通过即时断言验证结果，使测试在最接近错误源的位置失败。

### Lines 197-220
```cpp
 197:   //
 198:   // STSM 4x (128b)
 199:   //
 201:   {
 202:   device_vector<uint16_t> d_out(count);
 203:   #if defined(CUTLASS_ENABLE_SYCL)
 204:   sc_exp::launch<stsm_test_device<uint128_t>>
 205:   ( sc_exp::launch_policy{sc::dim3(1), sc::dim3(32),
 206:     sc_exp::launch_properties{sycl_ext::work_group_static_size(sizeof(uint128_t) / 4 * 32)}},
 207:     d_in.data(), d_out.data());
 208:   sc::wait_and_throw();
 209:   #else
 210:   stsm_test_device<uint128_t><<<1, 32>>>(
 211:     thrust::raw_pointer_cast(d_in.data()),
 212:     thrust::raw_pointer_cast(d_out.data()));
 213:   #endif
 214:   host_vector<uint16_t> h_out = d_out;
 215:   for (int i = 0; i < 128; ++i) {
 216:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 217:     EXPECT_EQ(h_out[i], h_in[i]);
 218:   }
 219:   CUTLASS_TRACE_HOST("STSM 4x stsm_test_device SUCCESS\n");
 220:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Launches a device kernel through the SYCL or CUDA execution path. Validates results immediately so the test fails close to the source of an error.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 通过 SYCL 或 CUDA 执行路径启动设备内核。 通过即时断言验证结果，使测试在最接近错误源的位置失败。

### Lines 222-227
```cpp
 222:   //
 223:   // CuTe STSM
 224:   //
 226:   {
 227:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 229-253
```cpp
 229:   auto smem_layout = Layout<Shape <_32,Shape <_2, _4>>,
 230:                             Stride< _2,Stride<_1,_64>>>{};
 231:   auto tiled_copy = make_tiled_copy(Copy_Atom<SM90_U32x1_STSM_N, uint16_t>{},
 232:                                     Layout<Shape<_32,_1>>{},
 233:                                     Layout<Shape< _1,_8>>{});
 234:   #if defined(CUTLASS_ENABLE_SYCL)
 235:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 236:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 237:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 238:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 239:   sc::wait_and_throw();
 240:   #else
 241:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 242:     thrust::raw_pointer_cast(d_in.data()),
 243:     thrust::raw_pointer_cast(d_out.data()),
 244:     tiled_copy,
 245:     smem_layout);
 246:   #endif
 247:   host_vector<uint16_t> h_out = d_out;
 248:   for (int i = 0; i < size(smem_layout); ++i) {
 249:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 250:     EXPECT_EQ(h_out[i], h_in[i]);
 251:   }
 252:   CUTLASS_TRACE_HOST("CuTe 32x8 interleaved U32x1_STSM_N SUCCESS\n");
 253:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 255-256
```cpp
 255:   {
 256:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 258-262
```cpp
 258:   auto smem_layout = Layout<Shape <_32,Shape <_2, _4>>,
 259:                             Stride< _2,Stride<_1,_64>>>{};
 260:   auto tiled_copy = make_tiled_copy(Copy_Atom<SM90_U32x2_STSM_N, uint16_t>{},
 261:                                     Layout<Shape<_32,_1>>{},
 262:                                     Layout<Shape< _1,_8>>{});
```
**EN:** Continues the procedural logic of the current helper or test case. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 264-283
```cpp
 264:   #if defined(CUTLASS_ENABLE_SYCL)
 265:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 266:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 267:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 268:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 269:   sc::wait_and_throw();
 270:   #else
 271:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 272:     thrust::raw_pointer_cast(d_in.data()),
 273:     thrust::raw_pointer_cast(d_out.data()),
 274:     tiled_copy,
 275:     smem_layout);
 276:   #endif
 277:   host_vector<uint16_t> h_out = d_out;
 278:   for (int i = 0; i < size(smem_layout); ++i) {
 279:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 280:     EXPECT_EQ(h_out[i], h_in[i]);
 281:   }
 282:   CUTLASS_TRACE_HOST("CuTe 32x8 interleaved U32x2_STSM_N SUCCESS\n");
 283:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Launches a device kernel through the SYCL or CUDA execution path. Validates results immediately so the test fails close to the source of an error.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 通过 SYCL 或 CUDA 执行路径启动设备内核。 通过即时断言验证结果，使测试在最接近错误源的位置失败。

### Lines 285-286
```cpp
 285:   {
 286:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 288-312
```cpp
 288:   auto smem_layout = Layout<Shape <_32,Shape <_2, _4>>,
 289:                             Stride< _2,Stride<_1,_64>>>{};
 290:   auto tiled_copy = make_tiled_copy(Copy_Atom<SM90_U32x4_STSM_N, uint16_t>{},
 291:                                     Layout<Shape<_32,_1>>{},
 292:                                     Layout<Shape< _1,_8>>{});
 293:   #if defined(CUTLASS_ENABLE_SYCL)
 294:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 295:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 296:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 297:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 298:   sc::wait_and_throw();
 299:   #else
 300:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 301:     thrust::raw_pointer_cast(d_in.data()),
 302:     thrust::raw_pointer_cast(d_out.data()),
 303:     tiled_copy,
 304:     smem_layout);
 305:   #endif
 306:   host_vector<uint16_t> h_out = d_out;
 307:   for (int i = 0; i < size(smem_layout); ++i) {
 308:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 309:     EXPECT_EQ(h_out[i], h_in[i]);
 310:   }
 311:   CUTLASS_TRACE_HOST("CuTe 32x8 interleaved U32x4_STSM_N SUCCESS\n");
 312:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 314-315
```cpp
 314:   {
 315:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 317-321
```cpp
 317:   auto smem_layout = Layout<Shape <_32,Shape <_2, _4>>,
 318:                             Stride< _2,Stride<_1,_64>>>{};
 319:   auto tiled_copy = make_tiled_copy(Copy_Atom<UniversalCopy<uint16_t>, uint16_t>{},
 320:                                     Layout<Shape<_32,_1>>{},
 321:                                     Layout<Shape< _1,_8>>{});
```
**EN:** Continues the procedural logic of the current helper or test case. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 323-342
```cpp
 323:   #if defined(CUTLASS_ENABLE_SYCL)
 324:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 325:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 326:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 327:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 328:   sc::wait_and_throw();
 329:   #else
 330:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 331:     thrust::raw_pointer_cast(d_in.data()),
 332:     thrust::raw_pointer_cast(d_out.data()),
 333:     tiled_copy,
 334:     smem_layout);
 335:   #endif
 336:   host_vector<uint16_t> h_out = d_out;
 337:   for (int i = 0; i < size(smem_layout); ++i) {
 338:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 339:     EXPECT_EQ(h_out[i], h_in[i]);
 340:   }
 341:   CUTLASS_TRACE_HOST("CuTe 32x8 interleaved STSM.U16 SUCCESS\n");
 342:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Launches a device kernel through the SYCL or CUDA execution path. Validates results immediately so the test fails close to the source of an error.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 通过 SYCL 或 CUDA 执行路径启动设备内核。 通过即时断言验证结果，使测试在最接近错误源的位置失败。

### Lines 344-345
```cpp
 344:   {
 345:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 347-371
```cpp
 347:   auto smem_layout = Layout<Shape <_32,_32>,
 348:                             Stride< _1,_32>>{};
 349:   auto tiled_copy = make_tiled_copy(Copy_Atom<SM90_U32x1_STSM_N, uint16_t>{},
 350:                                     Layout<Shape<_16,_2>>{},
 351:                                     Layout<Shape< _2,_4>>{});
 352:   #if defined(CUTLASS_ENABLE_SYCL)
 353:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 354:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 355:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 356:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 357:   sc::wait_and_throw();
 358:   #else
 359:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 360:     thrust::raw_pointer_cast(d_in.data()),
 361:     thrust::raw_pointer_cast(d_out.data()),
 362:     tiled_copy,
 363:     smem_layout);
 364:   #endif
 365:   host_vector<uint16_t> h_out = d_out;
 366:   for (int i = 0; i < size(smem_layout); ++i) {
 367:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 368:     EXPECT_EQ(h_out[i], h_in[i]);
 369:   }
 370:   CUTLASS_TRACE_HOST("CuTe 32x32 U32x1_STSM_N SUCCESS\n");
 371:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 373-374
```cpp
 373:   {
 374:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 376-400
```cpp
 376:   auto smem_layout = Layout<Shape <_32,_32>,
 377:                             Stride< _1,_32>>{};
 378:   auto tiled_copy = make_tiled_copy(Copy_Atom<SM90_U32x2_STSM_N, uint16_t>{},
 379:                                     Layout<Shape<_16,_2>>{},
 380:                                     Layout<Shape< _2,_4>>{});
 381:   #if defined(CUTLASS_ENABLE_SYCL)
 382:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 383:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 384:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 385:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 386:   sc::wait_and_throw();
 387:   #else
 388:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 389:     thrust::raw_pointer_cast(d_in.data()),
 390:     thrust::raw_pointer_cast(d_out.data()),
 391:     tiled_copy,
 392:     smem_layout);
 393:   #endif
 394:   host_vector<uint16_t> h_out = d_out;
 395:   for (int i = 0; i < size(smem_layout); ++i) {
 396:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 397:     EXPECT_EQ(h_out[i], h_in[i]);
 398:   }
 399:   CUTLASS_TRACE_HOST("CuTe 32x32 U32x2_STSM_N SUCCESS\n");
 400:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 402-403
```cpp
 402:   {
 403:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 405-429
```cpp
 405:   auto smem_layout = Layout<Shape <_32,_32>,
 406:                             Stride< _1,_32>>{};
 407:   auto tiled_copy = make_tiled_copy(Copy_Atom<SM90_U32x4_STSM_N, uint16_t>{},
 408:                                     Layout<Shape<_16,_2>>{},
 409:                                     Layout<Shape< _2,_4>>{});
 410:   #if defined(CUTLASS_ENABLE_SYCL)
 411:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 412:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 413:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 414:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 415:   sc::wait_and_throw();
 416:   #else
 417:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 418:     thrust::raw_pointer_cast(d_in.data()),
 419:     thrust::raw_pointer_cast(d_out.data()),
 420:     tiled_copy,
 421:     smem_layout);
 422:   #endif
 423:   host_vector<uint16_t> h_out = d_out;
 424:   for (int i = 0; i < size(smem_layout); ++i) {
 425:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 426:     EXPECT_EQ(h_out[i], h_in[i]);
 427:   }
 428:   CUTLASS_TRACE_HOST("CuTe 32x32 U32x4_STSM_N SUCCESS\n");
 429:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 431-432
```cpp
 431:   {
 432:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 434-458
```cpp
 434:   auto smem_layout = Layout<Shape <_32,_32>,
 435:                             Stride< _1,_32>>{};
 436:   auto tiled_copy = make_tiled_copy(Copy_Atom<UniversalCopy<uint16_t>, uint16_t>{},
 437:                                     Layout<Shape<_16,_2>>{},
 438:                                     Layout<Shape< _2,_4>>{});
 439:   #if defined(CUTLASS_ENABLE_SYCL)
 440:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 441:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 442:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 443:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 444:   sc::wait_and_throw();
 445:   #else
 446:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 447:     thrust::raw_pointer_cast(d_in.data()),
 448:     thrust::raw_pointer_cast(d_out.data()),
 449:     tiled_copy,
 450:     smem_layout);
 451:   #endif
 452:   host_vector<uint16_t> h_out = d_out;
 453:   for (int i = 0; i < size(smem_layout); ++i) {
 454:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 455:     EXPECT_EQ(h_out[i], h_in[i]);
 456:   }
 457:   CUTLASS_TRACE_HOST("CuTe 32x32 STSM.U16 SUCCESS\n");
 458:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 460-461
```cpp
 460:   {
 461:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 463-487
```cpp
 463:   auto smem_layout = Layout<Shape <_32,_32>,
 464:                             Stride<_32, _1>>{};
 465:   auto tiled_copy = make_tiled_copy(Copy_Atom<SM90_U16x2_STSM_T, uint16_t>{},
 466:                                     Layout<Shape<_4,_8>>{},
 467:                                     Layout<Shape<_2,_1>>{});
 468:   #if defined(CUTLASS_ENABLE_SYCL)
 469:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 470:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 471:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 472:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 473:   sc::wait_and_throw();
 474:   #else
 475:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 476:     thrust::raw_pointer_cast(d_in.data()),
 477:     thrust::raw_pointer_cast(d_out.data()),
 478:     tiled_copy,
 479:     smem_layout);
 480:   #endif
 481:   host_vector<uint16_t> h_out = d_out;
 482:   for (int i = 0; i < size(smem_layout); ++i) {
 483:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 484:     EXPECT_EQ(h_out[i], h_in[i]);
 485:   }
 486:   CUTLASS_TRACE_HOST("CuTe 32x32 U16x2_STSM_T SUCCESS\n");
 487:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 489-490
```cpp
 489:   {
 490:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 492-516
```cpp
 492:   auto smem_layout = Layout<Shape <_32,_32>,
 493:                             Stride<_32, _1>>{};
 494:   auto tiled_copy = make_tiled_copy(Copy_Atom<SM90_U16x4_STSM_T, uint16_t>{},
 495:                                     Layout<Shape<_4,_8>>{},
 496:                                     Layout<Shape<_4,_1>>{});
 497:   #if defined(CUTLASS_ENABLE_SYCL)
 498:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 499:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 500:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 501:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 502:   sc::wait_and_throw();
 503:   #else
 504:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 505:     thrust::raw_pointer_cast(d_in.data()),
 506:     thrust::raw_pointer_cast(d_out.data()),
 507:     tiled_copy,
 508:     smem_layout);
 509:   #endif
 510:   host_vector<uint16_t> h_out = d_out;
 511:   for (int i = 0; i < size(smem_layout); ++i) {
 512:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 513:     EXPECT_EQ(h_out[i], h_in[i]);
 514:   }
 515:   CUTLASS_TRACE_HOST("CuTe 32x32 U16x4_STSM_T SUCCESS\n");
 516:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 518-519
```cpp
 518:   {
 519:   device_vector<uint16_t> d_out(count);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 521-545
```cpp
 521:   auto smem_layout = Layout<Shape <_32,_32>,
 522:                             Stride<_32, _1>>{};
 523:   auto tiled_copy = make_tiled_copy(Copy_Atom<SM90_U16x8_STSM_T, uint16_t>{},
 524:                                     Layout<Shape<_4,_8>>{},
 525:                                     Layout<Shape<_8,_1>>{});
 526:   #if defined(CUTLASS_ENABLE_SYCL)
 527:   sc_exp::launch<stsm_test_device_cute<decltype(tiled_copy), decltype(smem_layout)>>
 528:   ( sc::launch_policy{sc::dim3(1), sc::dim3(int(size(tiled_copy))),
 529:     sc_exp::launch_properties{sycl_ext::work_group_static_size(size(smem_layout))}},
 530:     d_in.data(), d_out.data(), tiled_copy, smem_layout);
 531:   sc::wait_and_throw();
 532:   #else
 533:   stsm_test_device_cute<<<1, int(size(tiled_copy))>>>(
 534:     thrust::raw_pointer_cast(d_in.data()),
 535:     thrust::raw_pointer_cast(d_out.data()),
 536:     tiled_copy,
 537:     smem_layout);
 538:   #endif
 539:   host_vector<uint16_t> h_out = d_out;
 540:   for (int i = 0; i < size(smem_layout); ++i) {
 541:     //printf("%d  %d\n", int(h_in[i]), int(h_out[i]));
 542:     EXPECT_EQ(h_out[i], h_in[i]);
 543:   }
 544:   CUTLASS_TRACE_HOST("CuTe 32x32 U16x8_STSM_T SUCCESS\n");
 545:   }
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Builds layout metadata that maps logical coordinates onto physical memory addresses. Issues a CuTe copy operation between global, shared, or register-level tensors.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。

### Lines 547-549
```cpp
 547:   CUTLASS_TRACE_HOST("PASS");
 548: }
 549: #endif
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。

## Key Concepts / 关键概念
- **EN:** Hopper/SM90 backend coverage
  **CN:** 覆盖 Hopper/SM90 架构相关行为。
- **EN:** CuTe tensors and layouts
  **CN:** 使用 CuTe 张量与布局抽象描述数据形状、步幅和坐标映射。
- **EN:** Unit-test driven validation
  **CN:** 通过单元测试断言直接验证行为是否正确。
- **EN:** Copy-atom based data movement
  **CN:** 基于拷贝原子的数据搬运是本文件的重要主题。
- **EN:** SYCL compatibility path
  **CN:** 文件同时覆盖 SYCL 兼容实现路径。
- **EN:** Host/device round-trip checking
  **CN:** 通过主机与设备之间的往返数据检查结果正确性。

## Dependencies / 依赖关系
- **EN:** Direct headers: `"cutlass_unit_test.h"`, `<iostream>`, `<cute/tensor.hpp>`, `<cute/arch/copy_sm90.hpp>`.
  **CN:** 直接头文件依赖：`"cutlass_unit_test.h"`, `<iostream>`, `<cute/tensor.hpp>`, `<cute/arch/copy_sm90.hpp>`。
- **EN:** Primary test harness: `cutlass_unit_test.h` and GoogleTest-style assertion macros.
  **CN:** `cutlass_unit_test.h` 与 GoogleTest 风格断言宏构成主要测试框架。
- **EN:** Conditional feature gates: `CUTLASS_ENABLE_SYCL`, `CUDA_12_0_SM90_FEATURES_SUPPORTED`, `__SYCL_DEVICE_ONLY__`.
  **CN:** 条件特性开关：`CUTLASS_ENABLE_SYCL`, `CUDA_12_0_SM90_FEATURES_SUPPORTED`, `__SYCL_DEVICE_ONLY__`。
- **EN:** Key APIs referenced here: `make_tensor`, `copy`, `partition_S`, `partition_D`, `get_thread_slice`, `EXPECT_EQ`, `sc_exp::launch`, `ThreadIdxX`, `syncthreads`.
  **CN:** 此处反复使用的关键 API：`make_tensor`, `copy`, `partition_S`, `partition_D`, `get_thread_slice`, `EXPECT_EQ`, `sc_exp::launch`, `ThreadIdxX`, `syncthreads`。
