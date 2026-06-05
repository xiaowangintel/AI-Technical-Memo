# benchmark_runner.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/gemm/benchmark_runner.hpp`
- **EN:** Current GEMM benchmark harness that parses options, allocates tensors, handles mixed-precision dequantization for verification, runs kernels, and reports performance.
- **CN:** 当前 GEMM 基准运行器：解析选项、分配张量、在校验阶段处理混合精度反量化、运行内核，并汇报性能。

## Line-by-Line Analysis / 逐行分析

### Lines 1-64 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
   3 |  * Copyright (C) 2025 Intel Corporation, All rights reserved.
   4 |  * SPDX-License-Identifier: BSD-3-Clause
   5 |  *
   6 |  * Redistribution and use in source and binary forms, with or without
   7 |  * modification, are permitted provided that the following conditions are met:
   8 |  *
   9 |  * 1. Redistributions of source code must retain the above copyright notice, this
  10 |  * list of conditions and the following disclaimer.
  11 |  *
  12 |  * 2. Redistributions in binary form must reproduce the above copyright notice,
  13 |  * this list of conditions and the following disclaimer in the documentation
  14 |  * and/or other materials provided with the distribution.
  15 |  *
  16 |  * 3. Neither the name of the copyright holder nor the names of its
  17 |  * contributors may be used to endorse or promote products derived from
  18 |  * this software without specific prior written permission.
  19 |  *
  20 |  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  21 |  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  22 |  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  23 |  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  24 |  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  25 |  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  26 |  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  27 |  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  28 |  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  29 |  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  30 |  *
  31 |  **************************************************************************************************/
  32 | 
  33 | #pragma once
  34 | 
  35 | #include "cutlass/epilogue/collective/default_epilogue.hpp"
  36 | #include "cutlass/gemm/device/gemm_universal.h"
  37 | #include "cutlass/gemm/device/gemm_universal_adapter.h"
  38 | #include "cutlass/gemm/collective/collective_mma.hpp"
  39 | #include "cutlass/util/GPU_Clock.hpp"
  40 | #include "cutlass/epilogue/fusion/operations.hpp"
  41 | 
  42 | #include "cutlass/util/host_tensor.h"
  43 | #include "cutlass/util/reference/host/tensor_fill.h"
  44 | #include "cute/tensor.hpp"
  45 | 
  46 | #include "cutlass/util/command_line.h"
  47 | #include "cutlass/util/device_memory.h"
  48 | #include "cutlass/util/packed_stride.hpp"
  49 | #include "cutlass/util/reference/device/gemm_complex.h"
  50 | #include "cutlass/util/reference/device/tensor_compare.h"
  51 | #include "cutlass/util/reference/device/tensor_fill.h"
  52 | #include "cutlass/util/reference/device/tensor_silu.h"
  53 | #include "cutlass/util/initialize_block.hpp"
  54 | 
  55 | #include "../common.hpp"
  56 | 
  57 | #include <benchmark/benchmark.h>
  58 | 
  59 | using namespace cute;
  60 | 
  61 | namespace cutlass::benchmark {
  62 | 
  63 | ///////////////////////////////////////////////////////////////////////////////////////////////////
  64 | 
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 65-74 — Mixed-dtype trait
```cpp
  65 | #if defined(SYCL_INTEL_TARGET)
  66 | template <class T, int Stages = 0>
  67 | static constexpr auto is_mixed_dtype = false;
  68 | 
  69 | template <int Stages>
  70 | static constexpr auto is_mixed_dtype<cutlass::gemm::MainloopIntelXeXMX16MixedPrecision<Stages>> = true;
  71 | #else
  72 | template <class T, int Stages = 0>
  73 | static constexpr auto is_mixed_dtype = false;
  74 | #endif
```
**EN:** Defines a small compile-time trait that tells the runner whether the mainloop uses mixed-precision input handling.
**CN:** 定义一个小型编译期 trait，用来告诉运行器 mainloop 是否采用混合精度输入处理。

### Lines 76-110 — Scale and zero helper traits
```cpp
  76 | template <class T, class = void>
  77 | struct ScaleType {
  78 |   using type = int;
  79 | };
  80 | template <class T>
  81 | struct ScaleType<T, cute::void_t<typename T::ElementScale>> {
  82 |   using type = typename T::ElementScale;
  83 | };
  84 | 
  85 | template <class T, class = void>
  86 | struct ZeroType {
  87 |   using type = int;
  88 | };
  89 | template <class T>
  90 | struct ZeroType<T, cute::void_t<typename T::ElementZero>> {
  91 |   using type = typename T::ElementZero;
  92 | };
  93 | 
  94 | template <class T, class = void>
  95 | struct ScaleStride {
  96 |   using type = int;
  97 | };
  98 | template <class T>
  99 | struct ScaleStride<T, cute::void_t<typename T::StrideScale>> {
 100 |   using type = typename T::StrideScale;
 101 | };
 102 | 
 103 | template <class T, class = void>
 104 | struct ZeroStride {
 105 |   using type = int;
 106 | };
 107 | template <class T>
 108 | struct ZeroStride<T, cute::void_t<typename T::StrideZero>> {
 109 |   using type = typename T::StrideZero;
 110 | };
```
**EN:** Extracts optional scale and zero-point element and stride types from mixed-precision mainloops while providing defaults for regular GEMM.
**CN:** 从混合精度 mainloop 中提取可选的 scale、zero-point 及其步长类型，并为常规 GEMM 提供默认值。

### Lines 115-154 — GEMMOptions
```cpp
 115 | struct GEMMOptions {
 116 | 
 117 |   bool error;
 118 | 
 119 |   int m, n, k, l;
 120 |   float alpha, beta;
 121 |   std::string bm_name;
 122 | 
 123 |   GEMMOptions():
 124 |           error(false),
 125 |           m(5120), n(4096), k(4096), l(1),
 126 |           alpha(1.f), beta(0.f),
 127 |           bm_name("GEMM")
 128 |   { }
 129 | 
 130 |   // Parses the command line
 131 |   void parse(int argc, char const **args) {
 132 |     CommandLine cmd(argc, args);
 133 | 
 134 |     cmd.get_cmd_line_argument("m", m, 5120);
 135 |     cmd.get_cmd_line_argument("n", n, 4096);
 136 |     cmd.get_cmd_line_argument("k", k, 4096);
 137 |     cmd.get_cmd_line_argument("l", l, 1);
 138 |     cmd.get_cmd_line_argument("alpha", alpha, 1.f);
 139 |     cmd.get_cmd_line_argument("beta", beta, 0.f);
 140 |     cmd.get_cmd_line_argument("bm_name", bm_name, std::string("GEMM"));
 141 |   }
 142 | 
 143 |   std::string benchmark_name() const {
 144 |     std::stringstream full_name;
 145 |     full_name << bm_name << "/";
 146 |     std::string const test_name_suffix = std::to_string(m) + "x" +
 147 |                                    std::to_string(n) + "x" +
 148 |                                    std::to_string(k) + "x" +
 149 |                                    std::to_string(l);
 150 |     full_name << test_name_suffix;
 151 | 
 152 |     return full_name.str();
 153 |   }
 154 | };
```
**EN:** Parses matrix sizes, alpha/beta coefficients, and the benchmark name, then formats a compact name string.
**CN:** 解析矩阵尺寸、alpha/beta 系数与基准名称，并格式化一个紧凑的名字字符串。

### Lines 158-255 — Runner state and type aliases
```cpp
 158 | template <class GemmConfiguration>
 159 | struct BenchmarkRunnerGemm {
 160 | 
 161 |   using Gemm = typename GemmConfiguration::Gemm;
 162 | 
 163 |   using StrideA = typename Gemm::GemmKernel::StrideA;
 164 |   using StrideB = typename Gemm::GemmKernel::StrideB;
 165 |   using StrideC = typename Gemm::GemmKernel::StrideC;
 166 |   using StrideD = typename Gemm::GemmKernel::StrideD;
 167 | 
 168 |   using LayoutA = typename Gemm::LayoutA;
 169 |   using LayoutB = typename Gemm::LayoutB;
 170 |   using LayoutC = typename Gemm::LayoutC;
 171 |   using LayoutD = typename Gemm::LayoutD;
 172 | 
 173 |   using ElementA = typename Gemm::ElementA;
 174 |   using ElementB = typename Gemm::ElementB;
 175 |   using ElementAccumulator = typename Gemm::ElementAccumulator;
 176 | 
 177 |   using CollectiveMainloop = typename Gemm::GemmKernel::CollectiveMainloop;
 178 |   using DispatchPolicy = typename CollectiveMainloop::DispatchPolicy;
 179 |   using ElementMma = typename CollectiveMainloop::TiledMma::ValTypeA;
 180 | 
 181 |   using ElementScale = typename ScaleType<CollectiveMainloop>::type;
 182 |   using ElementZero = typename ZeroType<CollectiveMainloop>::type;
 183 |   using StrideS = typename ScaleStride<CollectiveMainloop>::type;
 184 |   using StrideZ = typename ZeroStride<CollectiveMainloop>::type;
 185 | 
 186 |   using CollectiveEpilogue = typename Gemm::CollectiveEpilogue;
 187 |   using ElementC = typename Gemm::ElementC;
 188 |   using ElementOutput = typename CollectiveEpilogue::ElementOutput;
 189 |   using ElementCompute = typename CollectiveEpilogue::ElementCompute;
 190 | 
 191 |   using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
 192 | 
 193 |   using FusionOp = typename Gemm::EpilogueOutputOp;
 194 | 
 195 |   // TODO(codeplay): Epilogue detection here should be replaced w/ general solution (see other TODO)
 196 |   using FusionSilu = cutlass::epilogue::fusion::LinCombEltAct<
 197 |       cutlass::epilogue::thread::SiLu, ElementOutput, ElementCompute, ElementAccumulator,
 198 |       ElementAccumulator, cutlass::FloatRoundStyle::round_to_nearest>;
 199 | 
 200 |   using FusionDeEltMul = cutlass::epilogue::fusion::LinCombDeEltAct<LayoutC, std::multiplies,
 201 |                                                                     ElementOutput, ElementCompute>;
 202 |   using FusionLinComb = epilogue::fusion::LinearCombination<
 203 |       ElementAccumulator, ElementCompute, ElementAccumulator, ElementAccumulator,
 204 |       FloatRoundStyle::round_to_nearest>;
 205 | 
 206 |   // Epilogue used in ampere/gemm_configuration.hpp
 207 |   using DefaultEpilogue = epilogue::collective::DefaultEpilogue<
 208 |     float,
 209 |     cutlass::gemm::TagToStrideC_t<LayoutC>,
 210 |     cutlass::gemm::TagToStrideC_t<LayoutC>,
 211 |     epilogue::thread::LinearCombination<float, 1>,
 212 |     cutlass::gemm::EpilogueDefault>;
 213 | 
 214 |   static constexpr bool epi_is_deeltactmul = std::is_same_v<FusionOp, FusionDeEltMul>;
 215 |   static constexpr bool epi_is_silu = std::is_same_v<FusionOp, FusionSilu>;
 216 |   static constexpr bool epi_is_lincomb = std::is_same_v<FusionOp, FusionLinComb>;
 217 |   static constexpr bool epi_is_default = std::is_same_v<CollectiveEpilogue, DefaultEpilogue>;
 218 |   static_assert(cute::is_base_of_v<cutlass::epilogue::fusion::FusionOperation, FusionOp> ||
 219 |                     epi_is_default,
 220 |                 "Failed to determine benchmark epilogue");
 221 |   static_assert(epi_is_default || epi_is_deeltactmul || epi_is_silu || epi_is_lincomb,
 222 |                 "Failed to determine benchmark epilogue");
 223 | 
 224 |   int32_t count;
 225 | 
 226 |   //
 227 |   // Data members
 228 |   //
 229 | 
 230 |   /// Initialization
 231 |   StrideA stride_A;
 232 |   StrideB stride_B;
 233 |   StrideC stride_C;
 234 |   StrideD stride_D;
 235 | 
 236 |   StrideS stride_S;
 237 |   StrideZ stride_Z;
 238 | 
 239 | 
 240 |   uint64_t seed;
 241 | 
 242 |   std::vector<DeviceAllocation<ElementA>> block_A;
 243 |   std::vector<DeviceAllocation<ElementB>> block_B;
 244 |   std::vector<DeviceAllocation<ElementC>> block_C;
 245 |   DeviceAllocation<ElementOutput> block_D;
 246 |   DeviceAllocation<ElementOutput> block_ref_D;
 247 |   std::vector<DeviceAllocation<ElementOutput>> block_Aux;
 248 | 
 249 |   cutlass::DeviceAllocation<ElementScale> block_scale;
 250 |   cutlass::DeviceAllocation<ElementZero> block_zero;
 251 | 
 252 |   DeviceAllocation<ElementMma> block_A_verify;
 253 |   DeviceAllocation<ElementMma> block_B_verify;
 254 | 
 255 |   BenchmarkRunnerGemm() : seed(0) {};
```
**EN:** Defines the templated GEMM runner, resolves all kernel-dependent types, detects epilogue kinds, and stores strides, buffers, and verification scratch space.
**CN:** 定义模板化 GEMM 运行器，解析所有依赖内核的类型，识别 epilogue 类型，并保存步长、缓冲区和校验暂存空间。

### Lines 261-367 — Dequantize A helper
```cpp
 261 |   template <
 262 |   class QuantizedElement,
 263 |   class DequantizedElement,
 264 |   class OperandLayout,
 265 |   class ElementScale,
 266 |   class ElementZero,
 267 |   class ScaleLayout,
 268 |   class ZeroLayout>
 269 |   static auto dequantize_A(DequantizedElement* dq_buffer,
 270 |                        QuantizedElement const* q_buffer,
 271 |                        OperandLayout const operand_layout,
 272 |                        ElementScale const* scale_buffer,
 273 |                        ElementZero const* zero_buffer,
 274 |                        ScaleLayout const scale_layout,
 275 |                        ZeroLayout const zero_layout,
 276 |                        int const group_size) {
 277 |     if constexpr (std::is_same_v<DequantizedElement, QuantizedElement>) {
 278 |       return dq_buffer;
 279 |     }
 280 | 
 281 |     std::vector<uint8_t> dst(size(operand_layout) * sizeof_bits_v<DequantizedElement> / 8, 0);
 282 |     cutlass::device_memory::copy_to_host(dst.data(), (uint8_t*)dq_buffer, dst.size());
 283 | 
 284 |     std::vector<uint8_t> src(size(operand_layout) * sizeof_bits_v<QuantizedElement> / 8, 0);
 285 |     cutlass::device_memory::copy_to_host(src.data(), (uint8_t*)q_buffer, src.size());
 286 | 
 287 |     std::vector<uint8_t> scale(size(scale_layout) * sizeof_bits_v<ElementScale> / 8, 0);
 288 |     cutlass::device_memory::copy_to_host(scale.data(), (uint8_t*)scale_buffer, scale.size());
 289 | 
 290 |     std::vector<uint8_t> zero(size(zero_layout) * sizeof_bits_v<ElementZero> / 8, 0);
 291 |     cutlass::device_memory::copy_to_host(zero.data(), (uint8_t*)zero_buffer, zero.size());
 292 | 
 293 |     compat::wait();
 294 | 
 295 |     auto dst_tensor = make_tensor(make_gmem_ptr(reinterpret_cast<DequantizedElement*>(dst.data())), select<1, 0, 2>(operand_layout));
 296 | 
 297 |     auto src_tensor = [&]() {
 298 |       if constexpr (sizeof_bits_v<QuantizedElement> < 8) {
 299 |         return make_tensor(cute::subbyte_iterator<const QuantizedElement>(src.data()), operand_layout);
 300 |       } else {
 301 |         return make_tensor(make_gmem_ptr(reinterpret_cast<QuantizedElement const *>(src.data())), select<1, 0, 2>(operand_layout));
 302 |       }
 303 |     }();
 304 | 
 305 |     auto scale_tensor = make_tensor(make_gmem_ptr(reinterpret_cast<ElementScale const *>(scale.data())), scale_layout);
 306 | 
 307 |     auto zero_tensor = [&]() {
 308 |       if constexpr (sizeof_bits_v<ElementZero> < 8) {
 309 |         auto flatten_tensor = flatten(make_tensor(cute::subbyte_iterator<const ElementZero>(zero.data()), zero_layout));
 310 |         static_assert(rank(flatten_tensor.layout()) == 4);
 311 |         return make_tensor(flatten_tensor.data(), select<1, 0, 2, 3>(flatten_tensor.layout()));
 312 |       } else {
 313 |         return make_tensor(make_gmem_ptr(reinterpret_cast<ElementZero const *>(zero.data())), zero_layout);
 314 |       }
 315 |     }();
 316 | 
 317 |     auto M = size<1>(src_tensor);
 318 |     auto K = size<0>(src_tensor);
 319 |     auto L = size<2>(src_tensor);
 320 | 
 321 |     static constexpr bool is_qnt = cutlass::platform::numeric_limits<DequantizedElement>::is_integer;
 322 | 
 323 |     for (int l = 0; l < L; l++) {
 324 |       for (int k= 0; k < K; k++) {
 325 |         for (int m = 0; m < M; m++) {
 326 |           auto src_data = [&]() {
 327 |             if constexpr (is_qnt) {
 328 |               if constexpr (sizeof_bits_v<QuantizedElement> >= 8) {
 329 |                 return  src_tensor(k, m, l);
 330 |               } else {
 331 |                 return src_tensor(k, m, l).get();
 332 |               }
 333 |             } else {
 334 |               using ret_type = cute::conditional_t<sizeof_bits_v<ElementZero> >= 8, ElementZero, int8_t>;
 335 |               if constexpr (sizeof_bits_v<QuantizedElement> >= 8) {
 336 |                 return  (ret_type)(src_tensor(k, m, l));
 337 |               } else {
 338 |                 return (ret_type)(src_tensor(k, m, l).get());
 339 |               }
 340 |             }
 341 |           }();
 342 | 
 343 |           auto scale_data = scale_tensor(m, k / group_size, l);
 344 | 
 345 |           using ret_type = cute::conditional_t<sizeof_bits_v<ElementZero> >= 8, ElementZero, int8_t>;
 346 |           ret_type zero_data = [&]() {
 347 |             if constexpr (sizeof_bits_v<ElementZero> >= 8) {
 348 |               return zero_tensor(m, k / group_size, l);
 349 |             } else {
 350 |               auto zero_elements_packed_along_k = get<0>(zero_tensor.shape());
 351 |               return (ret_type)(zero_tensor((k / group_size) % zero_elements_packed_along_k, m, k / group_size / zero_elements_packed_along_k, l).get());
 352 |             }
 353 |           }();
 354 | 
 355 |           if constexpr (is_qnt) {
 356 |             dst_tensor(k, m, l) = ((int)(src_data / scale_data)) + zero_data;
 357 |           } else {
 358 |             dst_tensor(k, m, l) = (src_data - zero_data) * scale_data;
 359 |           }
 360 |         }
 361 |       }
 362 |     }
 363 | 
 364 |     cutlass::device_memory::copy_to_device(dq_buffer, (DequantizedElement*)(raw_pointer_cast(dst_tensor.data())), dst_tensor.size());
 365 |     compat::wait();
 366 |     return dq_buffer;
 367 |   }
```
**EN:** Copies quantized operand A and its scale/zero metadata to host memory, reconstructs dequantized values element by element, and uploads the verification tensor.
**CN:** 把量化后的操作数 A 及其 scale/zero 元数据拷回主机，逐元素重建反量化值，再上传校验张量。

### Lines 369-453 — Dequantize B helper
```cpp
 369 |   template <
 370 |   class QuantizedElement,
 371 |   class DequantizedElement,
 372 |   class OperandLayout,
 373 |   class ElementScale,
 374 |   class ElementZero,
 375 |   class ScaleLayout,
 376 |   class ZeroLayout>
 377 |   static auto dequantize_B(DequantizedElement* dq_buffer,
 378 |                        QuantizedElement const* q_buffer,
 379 |                        OperandLayout const operand_layout,
 380 |                        ElementScale const* scale_buffer,
 381 |                        ElementZero const* zero_buffer,
 382 |                        ScaleLayout const scale_layout,
 383 |                        ZeroLayout const zero_layout,
 384 |                        int const group_size) {
 385 |     std::vector<uint8_t> dst(size(operand_layout) * sizeof_bits_v<DequantizedElement> / 8, 0);
 386 |     cutlass::device_memory::copy_to_host(dst.data(), (uint8_t*)dq_buffer, dst.size());
 387 | 
 388 |     std::vector<uint8_t> src(size(operand_layout) * sizeof_bits_v<QuantizedElement> / 8, 0);
 389 |     cutlass::device_memory::copy_to_host(src.data(), (uint8_t*)q_buffer, src.size());
 390 | 
 391 |     std::vector<uint8_t> scale(size(scale_layout) * sizeof_bits_v<ElementScale> / 8, 0);
 392 |     cutlass::device_memory::copy_to_host(scale.data(), (uint8_t*)scale_buffer, scale.size());
 393 | 
 394 |     std::vector<uint8_t> zero(size(zero_layout) * sizeof_bits_v<ElementZero> / 8, 0);
 395 |     cutlass::device_memory::copy_to_host(zero.data(), (uint8_t*)zero_buffer, zero.size());
 396 | 
 397 |     compat::wait();
 398 | 
 399 |     auto dst_tensor = make_tensor(make_gmem_ptr(reinterpret_cast<DequantizedElement*>(dst.data())), operand_layout);
 400 | 
 401 |     auto src_tensor = [&]() {
 402 |       if constexpr (sizeof_bits_v<QuantizedElement> < 8) {
 403 |         return make_tensor(cute::subbyte_iterator<const QuantizedElement>(src.data()), operand_layout);
 404 |       } else {
 405 |         return make_tensor(make_gmem_ptr(reinterpret_cast<QuantizedElement const *>(src.data())), operand_layout);
 406 |       }
 407 |     }();
 408 | 
 409 |     auto scale_tensor = make_tensor(make_gmem_ptr(reinterpret_cast<ElementScale const *>(scale.data())), scale_layout);
 410 | 
 411 |     auto zero_tensor = [&]() {
 412 |       if constexpr (sizeof_bits_v<ElementZero> < 8) {
 413 |         auto flatten_tensor = flatten(make_tensor(cute::subbyte_iterator<const ElementZero>(zero.data()), zero_layout));
 414 |         static_assert(rank(flatten_tensor.layout()) == 4);
 415 |         return make_tensor(flatten_tensor.data(), select<1, 0, 2, 3>(flatten_tensor.layout()));
 416 |       } else {
 417 |         return make_tensor(make_gmem_ptr(reinterpret_cast<ElementZero const *>(zero.data())), zero_layout);
 418 |       }
 419 |     }();
 420 | 
 421 |     auto N = size<0>(src_tensor);
 422 |     auto K = size<1>(src_tensor);
 423 |     auto L = size<2>(src_tensor);
 424 | 
 425 |     for (int l = 0; l < L; l++) {
 426 |       for (int k= 0; k < K; k++) {
 427 |         for (int n = 0; n < N; n++) {
 428 |           using ret_type = cute::conditional_t<sizeof_bits_v<ElementZero> >= 8, ElementZero, int8_t>;
 429 |           ret_type a = [&]() {
 430 |             if constexpr (sizeof_bits_v<QuantizedElement> >= 8) {
 431 |               return  (ret_type)(src_tensor(n, k, l));
 432 |             } else {
 433 |               return (ret_type)(src_tensor(n, k, l).get());
 434 |             }}();
 435 | 
 436 |           ret_type b = [&]() {
 437 |             if constexpr (sizeof_bits_v<ElementZero> >= 8) {
 438 |               return (ret_type)(zero_tensor(n, k / group_size, l));
 439 |             } else {
 440 |               auto k_packed = get<0>(zero_tensor.shape());
 441 |               return (ret_type)(zero_tensor((k / group_size) % k_packed, n, k / group_size / k_packed, l).get());
 442 |             }
 443 |           }();
 444 | 
 445 |           dst_tensor(n, k, l) = ((ElementScale)(a - b)) * scale_tensor(n, k / group_size, l);
 446 |         }
 447 |       }
 448 |     }
 449 | 
 450 |     cutlass::device_memory::copy_to_device(dq_buffer, (DequantizedElement*)(raw_pointer_cast(dst_tensor.data())), dst_tensor.size());
 451 |     compat::wait();
 452 |     return dq_buffer;
 453 |   }
```
**EN:** Performs the same reconstruction process for operand B, including packed sub-byte zero-point handling.
**CN:** 对操作数 B 执行同样的重建流程，并处理按子字节打包的 zero-point。

### Lines 455-556 — Reference verification
```cpp
 455 |   bool verify(const ProblemShapeType& problem_size, ElementCompute alpha, ElementCompute beta) {
 456 |     auto& M = cute::get<0>(problem_size);
 457 |     auto& N = cute::get<1>(problem_size);
 458 |     auto& K = cute::get<2>(problem_size);
 459 |     auto& L = cute::get<3>(problem_size);
 460 | 
 461 |     TensorRef ref_C(block_C[0].get(), LayoutC::packed({M, N}));
 462 |     TensorRef ref_D(block_ref_D.get(), LayoutD::packed({M, N}));
 463 | 
 464 |     auto [ptr_A, ptr_B] = [&]() {
 465 |       if constexpr (!is_mixed_dtype<DispatchPolicy>) {
 466 |         return make_tuple(block_A[0].get(), block_B[0].get());
 467 |       } else {
 468 |         static constexpr bool IsAQuant = cutlass::platform::numeric_limits<ElementA>::is_integer
 469 |                                     ^ cutlass::platform::numeric_limits<ElementAccumulator>::is_integer;
 470 |         static constexpr bool IsBQuant = cutlass::platform::numeric_limits<ElementB>::is_integer
 471 |                                           ^ cutlass::platform::numeric_limits<ElementAccumulator>::is_integer;
 472 | 
 473 |         static constexpr bool IsATransformed = CollectiveMainloop::IsATransformed;
 474 |         auto dq_mn_size = IsATransformed ? M : N;
 475 | 
 476 |         auto shape_ab = cute::make_shape(dq_mn_size, K, L);
 477 |         auto shape_scale = cute::make_shape(dq_mn_size, K / 128, L);
 478 |         static constexpr auto k_packed = CollectiveMainloop::zero_elements_packed_along_k;
 479 |         auto shape_zero = [&]() {
 480 |           if constexpr (is_tuple_v<std::remove_reference_t<decltype(cute::get<1>(stride_Z))>>) {
 481 |             return cute::make_shape(dq_mn_size, cute::make_shape(k_packed,
 482 |                                                         cute::max(1, K / 128 / k_packed)), L);
 483 |           } else {
 484 |             return shape_scale;
 485 |           }
 486 |         }();
 487 | 
 488 |         auto ptr_A = [&]() {
 489 |           if constexpr (IsAQuant) {
 490 |             return dequantize_A(block_A_verify.get(), block_A[0].get(), make_layout(shape_ab, stride_A), block_scale.get(),
 491 |                                 block_zero.get(), make_layout(shape_scale, stride_S), make_layout(shape_zero, stride_Z), 128);
 492 |           } else {
 493 |             return block_A_verify.get();
 494 |           }
 495 |         }();
 496 | 
 497 |         auto ptr_B = [&]() {
 498 |          if constexpr (IsBQuant) {
 499 |             return dequantize_B(block_B_verify.get(), block_B[0].get(), make_layout(shape_ab, stride_B), block_scale.get(),
 500 |                                 block_zero.get(), make_layout(shape_scale, stride_S), make_layout(shape_zero, stride_Z), 128);
 501 |           } else {
 502 |             return block_B_verify.get();
 503 |           }
 504 |         }();
 505 | 
 506 |         return make_tuple(ptr_A, ptr_B);
 507 |       }
 508 |     }();
 509 | 
 510 |     TensorRef ref_A(ptr_A, LayoutA::packed({M, K}));
 511 |     TensorRef ref_B(ptr_B, LayoutB::packed({K, N}));
 512 | 
 513 |     reference::device::GemmComplex(
 514 |             {M, N, K},
 515 |             alpha,
 516 |             ref_A,
 517 |             ComplexTransform::kNone,
 518 |             ref_B,
 519 |             ComplexTransform::kNone,
 520 |             beta,
 521 |             ref_C,
 522 |             ref_D,
 523 |             ElementAccumulator(0),
 524 |             L,     // batch_count
 525 |             get<2>(stride_A), // batch_stride_A
 526 |             get<2>(stride_B), // batch_stride_B
 527 |             get<2>(stride_C), // batch_stride_C
 528 |             get<2>(stride_D)  // batch_stride_D
 529 |     );
 530 | 
 531 | #if defined(CUTLASS_ENABLE_SYCL)
 532 |     compat::wait();
 533 | #else
 534 |     cudaDeviceSynchronize();
 535 | #endif
 536 | 
 537 |     // TODO(codeplay): Replace this with a general solution (hook up to Testbed3x)
 538 |     if constexpr (epi_is_silu) {
 539 |       using TensorView = cutlass::TensorView<ElementOutput, LayoutD>;
 540 |       for (int batch = 0, offset = 0; batch < L; batch++, offset += M * N) {
 541 |         cutlass::reference::device::TensorSiLu(TensorView(
 542 |             block_ref_D.get() + offset, LayoutD::packed({M, N}), cutlass::make_Coord(M, N)));
 543 |       }
 544 |     } else if constexpr (epi_is_deeltactmul) {
 545 |       cutlass::reference::device::BlockElementwiseOp<std::multiplies>(
 546 |           block_ref_D.get(), block_ref_D.get(), block_Aux[0].get(), block_D.size());
 547 |     }
 548 | 
 549 |     compat::wait();
 550 | 
 551 |     // Check if output from CUTLASS kernel and reference kernel are equal or not
 552 |     bool passed = reference::device::BlockCompareEqual(
 553 |       block_ref_D.get(), block_D.get(), block_D.size());
 554 | 
 555 |     return passed;
 556 |   }
```
**EN:** Selects either the original or dequantized inputs, runs a reference GEMM, applies fused epilogue behavior when needed, and compares the output against the kernel result.
**CN:** 在需要时选择原始输入或反量化输入，运行参考 GEMM，补充执行融合 epilogue 行为，然后把输出与内核结果比较。

### Lines 558-644 — Tensor initialization
```cpp
 558 |   /// Initialize operands to be used in the GEMM and reference GEMM
 559 |   void initialize(::benchmark::State& state, const ProblemShapeType& problem_size) {
 560 |     auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
 561 |     auto [M, N, K, L] = problem_shape_MNKL;
 562 | 
 563 |     stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(M, K, L));
 564 |     stride_B = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(N, K, L));
 565 |     stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(M, N, L));
 566 |     stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(M, N, L));
 567 | 
 568 |     // TODO(codeplay): cute::cosize(some_large_layout) will overflow int32. What can we do about this?
 569 |     std::size_t size_A = cute::cosize(make_layout(cute::make_shape(M, K, L), stride_A));
 570 |     std::size_t size_B = cute::cosize(make_layout(cute::make_shape(N, K, L), stride_B));
 571 |     std::size_t size_C = cute::cosize(make_layout(cute::make_shape(M, N, L), stride_C));
 572 |     std::size_t mem_occupied_ABC = ((size_A * sizeof_bits_v<ElementA>) + (size_B * sizeof_bits_v<ElementB>) +
 573 |                                    (size_C * sizeof_bits_v<ElementC>)) / sizeof_bits_v<int8_t>;
 574 |     count = std::ceil(static_cast<float>(cutlass::get_llc_size()) / static_cast<float>(mem_occupied_ABC)) + 1;
 575 | 
 576 |     if constexpr (is_mixed_dtype<DispatchPolicy>) {
 577 |       static constexpr bool IsATransformed = CollectiveMainloop::IsATransformed;
 578 | 
 579 |       auto dq_mn_size = IsATransformed ? M : N;
 580 |       auto scale_k = K / 128;
 581 | 
 582 |       static constexpr auto k_packed = CollectiveMainloop::zero_elements_packed_along_k;
 583 |       static constexpr auto is_tuple_z = is_tuple_v<std::remove_reference_t<decltype(cute::get<1>(StrideZ{}))>>;
 584 | 
 585 |       auto shape_scale = cute::make_shape(dq_mn_size, scale_k, L);
 586 | 
 587 |       stride_S = cutlass::make_cute_packed_stride(StrideS{}, shape_scale);
 588 |       stride_Z = [&]() {
 589 |         if constexpr (is_tuple_z) {
 590 |           return make_stride(Int<k_packed>{}, make_stride(_1{}, int64_t(k_packed * dq_mn_size)), int64_t(dq_mn_size * scale_k));
 591 |         } else {
 592 |           return stride_S;
 593 |         }
 594 |       }();
 595 | 
 596 |       block_A_verify.reset(size_A);
 597 |       block_B_verify.reset(size_B);
 598 | 
 599 |       block_scale.reset(static_cast<std::size_t>(scale_k) * L * dq_mn_size);
 600 |       block_zero.reset(static_cast<std::size_t>(scale_k) * L * dq_mn_size);
 601 | 
 602 |       initialize_block(block_scale, seed, ElementScale(1), ElementScale(4));
 603 |       initialize_block(block_zero, seed);
 604 |     }
 605 | 
 606 |     for(int i=0; i < count; i++) {
 607 |       block_A.emplace_back();
 608 |       block_B.emplace_back();
 609 |       block_C.emplace_back();
 610 |       if constexpr (epi_is_deeltactmul) {
 611 |         block_Aux.emplace_back();
 612 |       }
 613 |     }
 614 | 
 615 |     try {
 616 |       for (int i = 0; i < count; i++) {
 617 |         block_A[i].reset(size_A);
 618 |         block_B[i].reset(size_B);
 619 |         block_C[i].reset(size_C);
 620 |         if constexpr (is_mixed_dtype<DispatchPolicy>) {
 621 |           if (i == 0) {
 622 |             initialize_mixed_dtype_block(block_A[i], block_A_verify, seed + i);
 623 |             initialize_mixed_dtype_block(block_B[i], block_B_verify, seed + i);
 624 |           } else {
 625 |             initialize_block(block_A[i], seed + i);
 626 |             initialize_block(block_B[i], seed + i);
 627 |           }
 628 |         } else {
 629 |           initialize_block(block_A[i], seed + i);
 630 |           initialize_block(block_B[i], seed + i);
 631 |         }
 632 |         initialize_block(block_C[i], seed + i);
 633 |         if constexpr (epi_is_deeltactmul) {
 634 |           block_Aux[i].reset(size_C);
 635 |           initialize_block(block_Aux[i], seed + i);
 636 |         }
 637 |       }
 638 | 
 639 |       block_D.reset(size_C);
 640 |       block_ref_D.reset(size_C);
 641 |     } catch (std::exception const &e) {
 642 |       state.SkipWithError(e.what());
 643 |     }
 644 |   }
```
**EN:** Builds packed strides, sizes the allocation pool against cache capacity, allocates tensors, initializes input data, and prepares mixed-precision metadata.
**CN:** 构建紧凑步长，按缓存容量估算分配池大小，分配张量，初始化输入数据，并准备混合精度元数据。

### Lines 646-709 — Warmup and validation run
```cpp
 646 |   void run(::benchmark::State& state, const GEMMOptions& options, const KernelHardwareInfo& hw_info) {
 647 |     ProblemShapeType problem_size = ProblemShapeType{options.m, options.n, options.k, options.l};
 648 | 
 649 |     initialize(state, problem_size);
 650 | 
 651 |     typename Gemm::GemmKernel::Arguments arguments = GemmConfiguration::defaultArguments();
 652 |     arguments.mode = gemm::GemmUniversalMode::kGemm;
 653 |     arguments.problem_shape = problem_size;
 654 | 
 655 |     if constexpr (!is_mixed_dtype<DispatchPolicy>) {
 656 |       arguments.mainloop = {block_A[0].get(), stride_A, block_B[0].get(), stride_B};
 657 |     } else {
 658 |       arguments.mainloop = {block_A[0].get(), stride_A, block_B[0].get(), stride_B, block_scale.get(),
 659 |               stride_S, block_zero.get(), stride_Z, 128};
 660 |     }
 661 | 
 662 |     arguments.epilogue = {{ElementAccumulator(options.alpha), ElementAccumulator(options.beta)}, block_C[0].get(), stride_C, block_D.get(), stride_D};
 663 |     arguments.hw_info = hw_info;
 664 | 
 665 |     if constexpr(epi_is_deeltactmul){
 666 |       arguments.epilogue.thread.aux_ptr = block_Aux[0].get();
 667 |       arguments.epilogue.thread.dAux = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.m, options.n, options.l));
 668 |     }
 669 | 
 670 |     Gemm gemm_op;
 671 | 
 672 |     device_memory::allocation<uint8_t> workspace;
 673 |     size_t workspace_size = Gemm::get_workspace_size(arguments);
 674 |     try {
 675 |       workspace.reset(workspace_size);
 676 |     } catch (std::exception const &e) {
 677 |       state.SkipWithError(e.what());
 678 |     }
 679 | 
 680 |     if (gemm_op.can_implement(arguments) != cutlass::Status::kSuccess)
 681 |       state.SkipWithError("GEMM unable to implement given args.");
 682 | 
 683 |     if (gemm_op.initialize(arguments, workspace.get()) != cutlass::Status::kSuccess)
 684 |       state.SkipWithError("GEMM failed to initialize.");
 685 | 
 686 |     if (state.error_occurred()) return;
 687 | 
 688 |     // Run the GEMM
 689 |     gemm_op.run();
 690 | 
 691 | #if defined(CUTLASS_ENABLE_SYCL)
 692 |     compat::wait();
 693 | #else
 694 |     cudaDeviceSynchronize();
 695 | #endif
 696 | 
 697 |     // Verify that the result is correct
 698 |     bool passed = verify(problem_size, options.alpha, options.beta);
 699 |     if(not passed) {
 700 |       state.SkipWithError("Disposition Failed.");
 701 |     }
 702 | 
 703 |     state.counters["m"] = options.m;
 704 |     state.counters["n"] = options.n;
 705 |     state.counters["k"] = options.k;
 706 |     state.counters["l"] = options.l;
 707 |     state.counters["alpha"] = options.alpha;
 708 |     state.counters["beta"] = options.beta;
 709 | 
```
**EN:** Builds kernel arguments, allocates workspace, validates that the kernel can run, performs an initial launch, verifies correctness, and records descriptive counters.
**CN:** 构建内核参数，分配工作区，确认内核可运行，执行首次启动，校验正确性，并记录描述性计数器。

### Lines 710-773 — Labels, metrics, and timed loop
```cpp
 710 |     std::stringstream extra_label;
 711 |     if constexpr (cute::size<0>(StrideA{}) == 1) {
 712 |       extra_label << "layoutA=ColumnMajor ";
 713 |     } else if constexpr (cute::size<1>(StrideA{}) == 1) {
 714 |       extra_label << "layoutA=RowMajor ";
 715 |     }
 716 |     if constexpr (cute::size<0>(StrideB{}) == 1) {
 717 |       extra_label << "layoutB=RowMajor ";
 718 |     } else if constexpr (cute::size<1>(StrideB{}) == 1) {
 719 |       extra_label << "layoutB=ColumnMajor ";
 720 |     }
 721 |     if constexpr (cute::size<0>(StrideC{}) == 1) {
 722 |       extra_label << "layoutC=ColumnMajor ";
 723 |     } else if constexpr (cute::size<1>(StrideC{}) == 1) {
 724 |       extra_label << "layoutC=RowMajor ";
 725 |     }
 726 |     state.SetLabel(extra_label.str());
 727 | 
 728 |     auto gflop = 2.0 * options.m * options.n * options.k * options.l * 1e-9;
 729 | 
 730 |     // Compatible with data types smaller than 8 bits here
 731 |     constexpr double bits_per_byte = static_cast<double>(sizeof_bits_v<char>);
 732 |     constexpr double sizeof_a = sizeof_bits_v<ElementA> / bits_per_byte;
 733 |     constexpr double sizeof_b = sizeof_bits_v<ElementB> / bits_per_byte;
 734 |     constexpr double sizeof_c = sizeof_bits_v<ElementC> / bits_per_byte;
 735 |     auto mega_bytes_transferred = static_cast<double>(
 736 |         options.m * options.k * sizeof_a +
 737 |         options.k * options.n * sizeof_b +
 738 |         (options.beta != 0 ? 2 : 1) * options.m * options.n * sizeof_c
 739 |       ) * 1e-6 * options.l;
 740 | 
 741 |     initialize_counters(state);
 742 |     int32_t counter = 1;
 743 |     for(auto _ : state) {
 744 |       state.PauseTiming();
 745 |       int input_num = std::max(int(0), counter % count);
 746 |       typename Gemm::GemmKernel::Arguments arguments{
 747 |         gemm::GemmUniversalMode::kGemm,
 748 |         problem_size,
 749 |         {block_A[input_num].get(), stride_A, block_B[input_num].get(), stride_B},
 750 |         {{ElementAccumulator(options.alpha), ElementAccumulator(options.beta)}, block_C[input_num].get(), stride_C, block_D.get(), stride_D},
 751 |         hw_info
 752 |       };
 753 |       if constexpr (is_mixed_dtype<DispatchPolicy>) {
 754 |         arguments.mainloop = {block_A[input_num].get(), stride_A, block_B[input_num].get(), stride_B, block_scale.get(),
 755 |                 stride_S, block_zero.get(), stride_Z, 128};
 756 |       }
 757 |       if constexpr(epi_is_deeltactmul){
 758 |         arguments.epilogue.thread.aux_ptr = block_Aux[input_num].get();
 759 |         arguments.epilogue.thread.dAux = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.m, options.n, options.l));
 760 |       }
 761 |       gemm_op.initialize(arguments, workspace.get());
 762 |       state.ResumeTiming();
 763 | 
 764 |       GPU_Clock timer;
 765 |       timer.start();
 766 |       gemm_op.run();
 767 |       auto ms_elapsed = timer.milliseconds();
 768 |       update_counters(state, ms_elapsed);
 769 |       state.SetIterationTime(ms_elapsed / 1000);
 770 |       counter++;
 771 |     }
 772 |     finalize_counters(state, gflop, mega_bytes_transferred);
 773 |   }
```
**EN:** Builds layout labels, computes FLOP and bandwidth estimates, then cycles through cached inputs in the timed benchmark loop.
**CN:** 构建布局标签，计算 FLOP 与带宽估计，然后在计时循环中轮换使用缓存好的输入。

### Lines 775-811 — Counter helpers and macros
```cpp
 775 | private:
 776 |   static void initialize_counters(::benchmark::State& state) {
 777 |     state.counters["avg_runtime_ms"] = 0;
 778 |     state.counters["best_runtime_ms"] = std::numeric_limits<double>::max();
 779 |     state.counters["worst_runtime_ms"] = std::numeric_limits<double>::lowest();
 780 |   }
 781 | 
 782 |   static void update_counters(::benchmark::State& state, double ms_elapsed) {
 783 |     state.PauseTiming();
 784 |     state.counters["total_runtime_ms"] += ms_elapsed;
 785 |     state.counters["best_runtime_ms"] = std::min<double>(state.counters["best_runtime_ms"], ms_elapsed);
 786 |     state.counters["worst_runtime_ms"] = std::max<double>(state.counters["worst_runtime_ms"], ms_elapsed);
 787 |     state.ResumeTiming();
 788 |   }
 789 | 
 790 |   static void finalize_counters(::benchmark::State& state,  double gflop, double mega_bytes_transferred) {
 791 |     state.counters["avg_runtime_ms"] =
 792 |       (state.counters["total_runtime_ms"] -state.counters["best_runtime_ms"] - state.counters["worst_runtime_ms"] ) / static_cast<double>(state.iterations() - 2);
 793 |     state.counters["avg_tflops"] = gflop / state.counters["avg_runtime_ms"];
 794 |     state.counters["avg_throughput"] = mega_bytes_transferred / state.counters["avg_runtime_ms"];
 795 |     state.counters["best_tflop"] = gflop / state.counters["best_runtime_ms"];
 796 |     state.counters["best_bandwidth"] = mega_bytes_transferred / state.counters["best_runtime_ms"];
 797 |   }
 798 | };
 799 | 
 800 | }
 801 | 
 802 | #define CUTLASS_BENCHMARK(F) cutlass::benchmark::BenchmarkRegistry<cutlass::benchmark::GEMMOptions>::Register(#F, &F##_func)
 803 | 
 804 | #define CUTLASS_CREATE_GEMM_BENCHMARK(F)                          \
 805 |   static void F##_func(                                           \
 806 |       ::benchmark::State& state,                                  \
 807 |       cutlass::benchmark::GEMMOptions const& options,                 \
 808 |       cutlass::KernelHardwareInfo const& hw_info) {               \
 809 |     auto bench = cutlass::benchmark::BenchmarkRunnerGemm<F>();    \
 810 |     bench.run(state, options, hw_info);                           \
 811 |   }
```
**EN:** Computes average and best-case metrics from the recorded runtimes and exposes macros that wrap a configuration as a benchmark entry.
**CN:** 根据记录的运行时间计算平均值与最佳值指标，并提供宏把配置包装成基准入口。

## Key Concepts / 关键概念

- GEMM option parsing / GEMM 选项解析
- Mixed-precision verification path / 混合精度校验路径
- Reference GEMM comparison / 参考 GEMM 比较
- Timed performance loop / 计时性能循环

## Dependencies / 依赖关系

- `cutlass::gemm::*` — GEMM kernel machinery / GEMM 内核机制
- `cutlass::reference::device::*` — reference GEMM and compare / 参考 GEMM 与比较
- `GPU_Clock` and `benchmark::State` — timing / 计时
- `BenchmarkRegistry<GEMMOptions>` — benchmark registry / 基准注册表
