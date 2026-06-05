# tma_store_testbed.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `test/unit/cute/hopper/tma_store_testbed.hpp`
- **EN:** Helper kernel and host harness for Hopper TMA store tests.
- **CN:** 本文件围绕 `tma_store_testbed` 相关功能编写单元测试或辅助基架。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30
```cpp
   1: /***************************************************************************************************
   2:  * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
   3:  * SPDX-License-Identifier: BSD-3-Clause
   4:  *
   5:  * Redistribution and use in source and binary forms, with or without
   6:  * modification, are permitted provided that the following conditions are met:
   7:  *
   8:  * 1. Redistributions of source code must retain the above copyright notice, this
   9:  * list of conditions and the following disclaimer.
  10:  *
  11:  * 2. Redistributions in binary form must reproduce the above copyright notice,
  12:  * this list of conditions and the following disclaimer in the documentation
  13:  * and/or other materials provided with the distribution.
  14:  *
  15:  * 3. Neither the name of the copyright holder nor the names of its
  16:  * contributors may be used to endorse or promote products derived from
  17:  * this software without specific prior written permission.
  18:  *
  19:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  20:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  29:  *
  30:  **************************************************************************************************/
```
**EN:** Records copyright ownership and the BSD-3-Clause licensing terms for this source file.
**CN:** 记录该源文件的版权归属以及 BSD-3-Clause 许可条款。

### Lines 32
```cpp
  32: #pragma once
```
**EN:** Marks this header as include-once so helper declarations are not processed repeatedly.
**CN:** 通过 include-once 语义避免该头文件被重复处理。

### Lines 34
```cpp
  34: #include "cutlass_unit_test.h"
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 36-37
```cpp
  36: #include <iostream>
  37: #include <cstdint>
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 39
```cpp
  39: #include <cute/tensor.hpp>
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 41
```cpp
  41: namespace cutlass::test {
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。

### Lines 43-47
```cpp
  43: template <class ElementType, class SmemLayout>
  44: struct SharedStorage
  45: {
  46:   cute::ArrayEngine<ElementType, cute::cosize_v<SmemLayout>> smem;
  47: };
```
**EN:** Defines helper type `SharedStorage` used by the surrounding tests or kernels.
**CN:** 定义辅助类型 `SharedStorage`，供周围测试或内核复用。

### Lines 49
```cpp
  49: #if CUDA_12_0_SM90_FEATURES_SUPPORTED
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。

### Lines 51-58
```cpp
  51: template <class T, class TiledCopy, class CTA_Tiler, class GmemLayout, class SmemLayout>
  52: CUTLASS_GLOBAL void
  53: tma_test_device_cute(T const* g_in, T* g_out,
  54:                      CUTE_GRID_CONSTANT TiledCopy const tma, CTA_Tiler cta_tiler,
  55:                      GmemLayout gmem_layout, SmemLayout smem_layout)
  56: {
  57:   using namespace cute;
  58:   CUTE_STATIC_ASSERT_V(product_each(shape(cta_tiler)) == product_each(shape(smem_layout)));
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently. Validates results immediately so the test fails close to the source of an error.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。 通过即时断言验证结果，使测试在最接近错误源的位置失败。

### Lines 60-71
```cpp
  60:   // Use Shared Storage structure to allocate and distribute aligned SMEM addresses
  61:   #if defined(__SYCL_DEVICE_ONLY__)
  62:     auto smem = sycl_ext::get_dynamic_work_group_memory<char>().get();
  63:   #endif
  64:   #if defined(CUTLASS_ENABLE_SYCL) && !defined(__SYCL_DEVICE_ONLY__)
  65:     char* smem; // dummy declaration to avoid compilation errors during the host compilation phase
  66:   #endif
  67:   #if !defined(CUTLASS_ENABLE_SYCL)
  68:     extern CUTLASS_SHARED char shared_memory[];
  69:   #endif
  70:   using SharedStorage = SharedStorage<T, SmemLayout>;
  71:   SharedStorage& shared_storage = *reinterpret_cast<SharedStorage*>(shared_memory);
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Keeps a SYCL-compatible code path alongside CUDA-specific logic.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 在 CUDA 专用逻辑旁保留兼容 SYCL 的实现路径。

### Lines 73-74
```cpp
  73:   // Construct SMEM tensor
  74:   Tensor sB = make_tensor(make_smem_ptr(shared_storage.smem.begin()), smem_layout);  // (CTA_TILE_M,CTA_TILE_N,...)
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。

### Lines 76-79
```cpp
  76:   // TMA requires special handling of strides to deal with coord codomain mapping
  77:   // Represent the full tensors -- get these from TMA
  78:   Tensor mA = make_tensor(make_gmem_ptr<T>(g_in), gmem_layout);
  79:   Tensor mB = tma.get_tma_tensor(shape(gmem_layout));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。

### Lines 81-83
```cpp
  81:   constexpr int R = rank_v<CTA_Tiler>;
  82:   Tensor gA = flat_divide(mA, cta_tiler);                 // (CTA_TILE_M,CTA_TILE_N,...REST_M,REST_N,...)
  83:   Tensor gB = flat_divide(mB, cta_tiler);                 // (CTA_TILE_M,CTA_TILE_N,...REST_M,REST_N,...)
```
**EN:** Continues the procedural logic of the current helper or test case. Applies CuTe layout algebra to tile, regroup, or simplify multidimensional data views.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 利用 CuTe 布局代数对多维数据视图进行分块、重组或简化。

### Lines 85-91
```cpp
  85:   //
  86:   // Prepare the TMA_STORE
  87:   //
  89:   auto cta_tma = tma.get_slice(Int<0>{});                            // CTA slice
  90:   Tensor tBsB_x = cta_tma.partition_S(sB);                           // (TMA,TMA_M,TMA_N)
  91:   Tensor tBgB_x = cta_tma.partition_D(gB);                           // (TMA,TMA_M,TMA_N,REST_M,REST_N)
```
**EN:** Continues the procedural logic of the current helper or test case. Partitions tensors so each thread, subgroup, or copy atom sees only its own slice. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 对张量进行分块，使每个线程、子组或拷贝原子只处理自己的切片。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。

### Lines 93-103
```cpp
  93: #if 0
  94:   if (thread0()) {
  95:     print(tma);
  96:     print("TILE  :  "); print(cta_tiler); print("\n");
  97:     print("  mB  :  "); print(  mB.data());   print(" o "); print(  mB.layout());   print("\n");
  98:     print("  gB  :  "); print(  gB.data());   print(" o "); print(  gB.layout());   print("\n");
  99:     print("tBgB_x:  "); print(tBgB_x.data()); print(" o "); print(tBgB_x.layout()); print("\n");
 100:     print("  sB  :  "); print(  sB.data());   print(" o "); print(  sB.layout());   print("\n");
 101:     print("tBsB_x:  "); print(tBsB_x.data()); print(" o "); print(tBsB_x.layout()); print("\n");
 102:   }
 103: #endif
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。

### Lines 105-110
```cpp
 105:   //
 106:   // Perform the TMA_STORE
 107:   //
 109:   // INPUT: Group the CTA_TILE_X modes and REST_X modes for input
 110:   Tensor tAgA = group_modes<0,R>(group_modes<R,rank(gA)>(gA));       // (CTA_TILE, REST)
```
**EN:** Continues the procedural logic of the current helper or test case. Applies CuTe layout algebra to tile, regroup, or simplify multidimensional data views. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 利用 CuTe 布局代数对多维数据视图进行分块、重组或简化。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。

### Lines 112-115
```cpp
 112:   // OUTPUT: Group the REST_X modes and the TMA_X modes to easily iterate through the tiles
 113:   Tensor tBgB = group_modes<1,rank(tBgB_x)>(tBgB_x);                 // (TMA,REST)
 114:   Tensor tBsB = group_modes<1,rank(tBsB_x)>(tBsB_x);                 // (TMA,REST)
 115:   static_assert(size<1>(tBsB) == 1);
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Applies CuTe layout algebra to tile, regroup, or simplify multidimensional data views. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently. Validates results immediately so the test fails close to the source of an error.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 利用 CuTe 布局代数对多维数据视图进行分块、重组或简化。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。 通过即时断言验证结果，使测试在最接近错误源的位置失败。

### Lines 117-123
```cpp
 117: #if 0
 118:   if (thread0()) {
 119:     print("tAgA  :  "); print(tAgA.data()); print(" o "); print(tAgA.layout()); print("\n");
 120:     print("tBsB  :  "); print(tBsB.data()); print(" o "); print(tBsB.layout()); print("\n");
 121:     print("tBgB  :  "); print(tBgB.data()); print(" o "); print(tBgB.layout()); print("\n");
 122:   }
 123: #endif
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。

### Lines 125-126
```cpp
 125:   // Test L2 prefetch
 126:   cooperative_prefetch<128>(ThreadIdxX(), gA);
```
**EN:** Continues the procedural logic of the current helper or test case. Manages asynchronous data movement and the synchronization needed to consume it safely. Uses per-thread or per-subgroup indices to distribute work across the execution unit.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 管理异步数据传输以及安全消费这些数据所需的同步。 使用线程或子组索引把工作分配到执行单元。

### Lines 128-137
```cpp
 128:   // Loop over the TMA stages, using smem as our buffer
 129:   for (int stage = 0; stage < size<1>(tBgB); ++stage)
 130:   {
 131:     //
 132:     // Read in trivially gmem -> smem
 133:     //
 134:     // Subbyte elements could cause race conditions, so be even more conservative
 135:     if (thread0()) {
 136:       copy(tAgA(_,stage), sB);
 137:     }
```
**EN:** Continues the procedural logic of the current helper or test case. Issues a CuTe copy operation between global, shared, or register-level tensors. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。 初始化测试场景所需的确定性或随机输入数据。

### Lines 139-140
```cpp
 139:     syncthreads();
 140:     cute::cp_async_wait<0>();
```
**EN:** Continues the procedural logic of the current helper or test case. Manages asynchronous data movement and the synchronization needed to consume it safely. Synchronizes execution so later reads observe completed writes or completed transfers.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 管理异步数据传输以及安全消费这些数据所需的同步。 通过同步确保后续读取能够看到已经完成的写入或传输。

### Lines 142-148
```cpp
 142:     //
 143:     // Perform the TMA_STORE
 144:     //
 146:     if (ThreadIdxX() == 0) {
 147:       copy(tma, tBsB(_,0), tBgB(_,stage));
 148:     }
```
**EN:** Continues the procedural logic of the current helper or test case. Issues a CuTe copy operation between global, shared, or register-level tensors. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently. Uses per-thread or per-subgroup indices to distribute work across the execution unit.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。 使用线程或子组索引把工作分配到执行单元。

### Lines 150-153
```cpp
 150:     tma_store_wait<0>();
 151:     syncthreads();
 152:   }
 153: }
```
**EN:** Continues the procedural logic of the current helper or test case. Manages asynchronous data movement and the synchronization needed to consume it safely. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently. Synchronizes execution so later reads observe completed writes or completed transfers.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 管理异步数据传输以及安全消费这些数据所需的同步。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。 通过同步确保后续读取能够看到已经完成的写入或传输。

### Lines 155-162
```cpp
 155: template <class T, class TmaType = T, class CopyOp, class GMEM_Layout, class SMEM_Layout, class CTA_Tile>
 156: void
 157: test_tma_store(CopyOp      const& copy_op,
 158:                GMEM_Layout const& gmem_layout,
 159:                SMEM_Layout const& smem_layout,
 160:                CTA_Tile    const& cta_tile)
 161: {
 162:   using namespace cute;
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。

### Lines 164-170
```cpp
 164:   // Allocate and initialize host test data
 165:   size_t N = ceil_div(cosize(gmem_layout) * sizeof_bits<T>::value, 8);
 166:   host_vector<uint8_t> h_in(N);
 167:   for (size_t i = 0; i < h_in.size(); ++i) {
 168:     h_in[i] = uint8_t(i % 13);
 169:   }
 170:   Tensor hA_in  = make_tensor(recast_ptr<T>(h_in.data()), gmem_layout);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 初始化测试场景所需的确定性或随机输入数据。

### Lines 172-174
```cpp
 172:   // Allocate and initialize device test data
 173:   device_vector<uint8_t> d_in = h_in;
 174:   device_vector<uint8_t> d_out(h_in.size(), uint8_t(-1)); // overflow uint
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 176-179
```cpp
 176:   // Create TMA for this device Tensor
 177:   Tensor gA = make_tensor(make_gmem_ptr<T>(raw_pointer_cast(d_out.data())), gmem_layout);
 178:   auto tma = make_tma_copy<TmaType>(copy_op, gA, smem_layout, cta_tile, Int<1>{});
 179:   //print(tma);
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Issues a CuTe copy operation between global, shared, or register-level tensors. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 发起一次 CuTe 拷贝操作，在全局、共享或寄存器级张量之间传输数据。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。

### Lines 181-205
```cpp
 181:   // Launch
 182:   int smem_size = int(sizeof(SharedStorage<T, decltype(smem_layout)>));
 183:   #if defined(CUTLASS_ENABLE_SYCL)
 184:   auto kernel = tma_test_device_cute<T,
 185:                                     decltype(tma),
 186:                                     CTA_Tile,
 187:                                     GmemLayout,
 188:                                     SmemLayout>;
 189:   sc_exp::launch<kernel>
 190:   ( sc_exp::launch_policy{sc::dim3(1), sc::dim3(128),
 191:     sc_exp::launch_properties{sycl_ext::work_group_static_size(smem_size)}},
 192:     d_in.data(), d_out.data(), tma, cta_tile,
 193:     gmem_layout, smem_layout);
 194:   sc::wait_and_throw();
 195:   #else
 196:   tma_test_device_cute<<<1, 128, smem_size>>>(
 197:     reinterpret_cast<T const*>(raw_pointer_cast(d_in.data())),
 198:     reinterpret_cast<T*>      (raw_pointer_cast(d_out.data())),
 199:     tma, cta_tile,
 200:     gmem_layout,
 201:     smem_layout);
 202:   #endif
 203:   // Copy results back to host
 204:   host_vector<uint8_t> h_out = d_out;
 205:   Tensor hA_out = make_tensor(recast_ptr<T>(h_out.data()), gmem_layout);
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Allocates host/device buffers and moves test data between them. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Uses Hopper TMA descriptors or transactions to move tensor tiles efficiently.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 使用 Hopper 的 TMA 描述符或事务高效搬运张量分块。

### Lines 207-219
```cpp
 207:   // Validate the results. Print only the first 3 errors.
 208:   int count = 3;
 209:   for (int i = 0; i < int(size(hA_out)) && count > 0; ++i) {
 210:     EXPECT_EQ(hA_in(i), hA_out(i));
 211:     if (hA_in(i) != hA_out(i)) {
 212:       --count;
 213:     }
 214:   }
 215: }
 217: #endif
 219: } // end namespace cutlass::test
```
**EN:** Uses conditional compilation to select the correct backend or hardware-feature-specific path. Validates results immediately so the test fails close to the source of an error. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 通过条件编译选择合适的后端路径或硬件特性实现。 通过即时断言验证结果，使测试在最接近错误源的位置失败。 初始化测试场景所需的确定性或随机输入数据。

## Key Concepts / 关键概念
- **EN:** Hopper/SM90 backend coverage
  **CN:** 覆盖 Hopper/SM90 架构相关行为。
- **EN:** CuTe tensors and layouts
  **CN:** 使用 CuTe 张量与布局抽象描述数据形状、步幅和坐标映射。
- **EN:** Unit-test driven validation
  **CN:** 通过单元测试断言直接验证行为是否正确。
- **EN:** Copy-atom based data movement
  **CN:** 基于拷贝原子的数据搬运是本文件的重要主题。
- **EN:** Asynchronous memory pipeline
  **CN:** 涉及异步内存流水线及其同步语义。
- **EN:** Tensor Memory Accelerator usage
  **CN:** 涉及 Tensor Memory Accelerator 的描述符、事务或同步机制。

## Dependencies / 依赖关系
- **EN:** Direct headers: `"cutlass_unit_test.h"`, `<iostream>`, `<cstdint>`, `<cute/tensor.hpp>`.
  **CN:** 直接头文件依赖：`"cutlass_unit_test.h"`, `<iostream>`, `<cstdint>`, `<cute/tensor.hpp>`。
- **EN:** Primary test harness: `cutlass_unit_test.h` and GoogleTest-style assertion macros.
  **CN:** `cutlass_unit_test.h` 与 GoogleTest 风格断言宏构成主要测试框架。
- **EN:** Conditional feature gates: `CUTLASS_ENABLE_SYCL`, `CUDA_12_0_SM90_FEATURES_SUPPORTED`, `__SYCL_DEVICE_ONLY__`.
  **CN:** 条件特性开关：`CUTLASS_ENABLE_SYCL`, `CUDA_12_0_SM90_FEATURES_SUPPORTED`, `__SYCL_DEVICE_ONLY__`。
- **EN:** Key APIs referenced here: `make_tensor`, `copy`, `cp_async`, `prefetch`, `make_tma_copy`, `partition_S`, `partition_D`, `get_slice`, `flat_divide`, `group_modes`.
  **CN:** 此处反复使用的关键 API：`make_tensor`, `copy`, `cp_async`, `prefetch`, `make_tma_copy`, `partition_S`, `partition_D`, `get_slice`, `flat_divide`, `group_modes`。
