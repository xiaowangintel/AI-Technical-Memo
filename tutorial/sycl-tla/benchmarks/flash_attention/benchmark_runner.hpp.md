# benchmark_runner.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/benchmark_runner.hpp`
- **EN:** Unified Flash Attention benchmark harness covering decode and prefill flows, variable lengths, cached KV, paged KV, validation, and timing.
- **CN:** 统一的 Flash Attention 基准运行器：覆盖 decode 与 prefill、变长、缓存 KV、分页 KV、结果校验和计时。

## Line-by-Line Analysis / 逐行分析

### Lines 1-56 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (C) 2026 Intel Corporation, All rights reserved.
   3 |  * SPDX-License-Identifier: BSD-3-Clause
   4 |  *
   5 |  * Redistribution and use in source and binary forms, with or without
   6 |  * modification, are permitted provided that the following conditions are met:
   7 |  *
   8 |  * 1. Redistributions of source code must retain the above copyright notice, this
   9 |  * list of conditions and the following disclaimer.
  10 |  *
  11 |  * 2. Redistributions in binary form must reproduce the above copyright notice,
  12 |  * this list of conditions and the following disclaimer in the documentation
  13 |  * and/or other materials provided with the distribution.
  14 |  *
  15 |  * 3. Neither the name of the copyright holder nor the names of its
  16 |  * contributors may be used to endorse or promote products derived from
  17 |  * this software without specific prior written permission.
  18 |  *
  19 |  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  20 |  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  21 |  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  22 |  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  23 |  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  24 |  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  25 |  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  26 |  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  27 |  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  28 |  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  29 |  *
  30 |  **************************************************************************************************/
  31 | #pragma once
  32 | 
  33 | #include "cutlass/epilogue/collective/default_epilogue.hpp"
  34 | #include "cutlass/gemm/device/gemm_universal_adapter.h"
  35 | #include "cutlass/util/packed_stride.hpp"
  36 | #include "flash_attention_v2/collective/fmha_fusion.hpp"
  37 | #include "flash_attention_v2/kernel/xe_fmha_fwd_kernel.hpp"
  38 | #include "flash_attention_v2/kernel/xe_tile_scheduler.hpp"
  39 | #include "cutlass/util/GPU_Clock.hpp"
  40 | #include "cutlass/util/sycl_event_manager.hpp"
  41 | #include <cute/tensor.hpp>
  42 | #include <random>
  43 | 
  44 | #include "cutlass/util/command_line.h"
  45 | #include "cutlass/util/device_memory.h"
  46 | #include "cutlass/util/reference/device/gemm_complex.h"
  47 | #include "cutlass/util/reference/device/tensor_compare.h"
  48 | #include "../examples/common/sycl_common.hpp"
  49 | #include "../benchmarks/common.hpp"
  50 | 
  51 | #include <sycl/ext/intel/experimental/grf_size_properties.hpp>
  52 | 
  53 | using namespace cute;
  54 | 
  55 | namespace cutlass::benchmark {
  56 | 
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 57-110 — FMHAOptions
```cpp
  57 | // Command line options parsing
  58 | struct FMHAOptions {
  59 | 
  60 |   bool error;
  61 | 
  62 |   int batch, num_heads_q, num_heads_kv, seq_len_qo, seq_len_kv, seq_len_kv_cache, head_size_qk,
  63 |       head_size_vo, iterations, page_size;
  64 |   float softmax_scale;
  65 |   std::string bm_name;
  66 | 
  67 |   FMHAOptions()
  68 |       : error(false), batch(32), num_heads_q(16), num_heads_kv(16), seq_len_qo(1), head_size_qk(128),
  69 |         seq_len_kv(512), seq_len_kv_cache(0), page_size(128), head_size_vo(128), iterations(100), softmax_scale(1.f), bm_name("Flash Attention v2") {}
  70 | 
  71 |   // Parses the command line
  72 |   void parse(int argc, char const **args) {
  73 |     cutlass::CommandLine cmd(argc, args);
  74 | 
  75 |     cmd.get_cmd_line_argument("batch", batch, 32);
  76 |     cmd.get_cmd_line_argument("num_heads_q", num_heads_q, 16);
  77 |     cmd.get_cmd_line_argument("num_heads_kv", num_heads_kv, num_heads_q);
  78 |     cmd.get_cmd_line_argument("seq_len_qo", seq_len_qo, 1);
  79 |     cmd.get_cmd_line_argument("seq_len_kv", seq_len_kv, seq_len_qo);
  80 |     cmd.get_cmd_line_argument("seq_len_kv_cache", seq_len_kv_cache, 0);
  81 |     cmd.get_cmd_line_argument("page_size", page_size, 128);
  82 |     cmd.get_cmd_line_argument("head_size_vo", head_size_vo, 128);
  83 |     cmd.get_cmd_line_argument("head_size_qk", head_size_qk, head_size_vo);
  84 |     cmd.get_cmd_line_argument("iterations", iterations, 100);
  85 |     cmd.get_cmd_line_argument("bm_name", bm_name, std::string("Flash Attention v2"));
  86 | 
  87 |     softmax_scale = 1 / std::sqrt(static_cast<float>(head_size_qk));
  88 | 
  89 |     if (seq_len_kv_cache % page_size != 0) {
  90 |       std::cerr << "Invalid: seq_len_kv_cache must be divisible by page_size" << std::endl;
  91 |       return;
  92 |     }
  93 |   }
  94 | 
  95 |   std::string benchmark_name() const {
  96 |     std::stringstream full_name;
  97 |     full_name << bm_name << "/";
  98 |     std::string const test_name_suffix = std::to_string(batch) + "x" +
  99 |                                    std::to_string(num_heads_q) + "x" +
 100 |                                    std::to_string(num_heads_kv) + "x" +
 101 |                                    std::to_string(seq_len_qo) + "x" +
 102 |                                    std::to_string(head_size_qk) + "x" +
 103 |                                    std::to_string(seq_len_kv) + "x" +
 104 |                                    std::to_string(seq_len_kv_cache) + "x" +
 105 |                                    std::to_string(head_size_vo);
 106 |     full_name << test_name_suffix;
 107 | 
 108 |     return full_name.str();
 109 |   }
 110 | };
```
**EN:** Parses batch, head, sequence, page, and iteration arguments; derives the default softmax scale; and formats a benchmark name.
**CN:** 解析 batch、head、序列长度、页大小和迭代次数参数；推导默认 softmax 缩放；并格式化基准名称。

### Lines 114-133 — Tensor conversion helpers
```cpp
 114 | template <typename SrcT, typename DstT> class ConvertTensorKernelTag{};
 115 | 
 116 | template <typename SrcT, typename DstT>
 117 | void convert_tensor(const SrcT* d_src, DstT* d_dst, size_t size) {
 118 |   using Tag = ConvertTensorKernelTag<SrcT, DstT>;
 119 |   compat::get_default_queue().parallel_for<Tag>(size, [=](auto indx) {
 120 |     d_dst[indx] = static_cast<DstT>(d_src[indx]);
 121 |   }).wait();
 122 | }
 123 | 
 124 | template <typename InT> inline auto in_memory(cutlass::DeviceAllocation<InT>& in) {
 125 |   using OutT = cute::conditional_t<(sizeof_bits_v<InT> <= 8), half_t, InT>;
 126 |   if constexpr (!is_same_v<InT, OutT>) {
 127 |     cutlass::DeviceAllocation<OutT> out(in.size());
 128 |     convert_tensor<InT, OutT>(in.get(), out.get(), in.size());
 129 |     return out;
 130 |   } else {
 131 |     return in;
 132 |   };
 133 | }
```
**EN:** Provides small utilities that convert device tensors to a host-friendly in-memory type when verification needs wider storage than the kernel input type.
**CN:** 提供小型张量转换工具：当校验阶段需要比内核输入更宽的存储类型时，把设备张量转换为更适合校验的内存表示。

### Lines 137-204 — Benchmark runner state
```cpp
 137 | template <class FMHAConfiguration> struct BenchmarkRunnerFMHA {
 138 | 
 139 |   using FMHAKernel = typename FMHAConfiguration::FMHAKernel;
 140 | 
 141 |   using StrideQ = typename FMHAKernel::StrideQ;
 142 |   using StrideK = typename FMHAKernel::StrideK;
 143 |   using StrideV = typename FMHAKernel::StrideV;
 144 |   using StrideO = typename FMHAKernel::StrideO;
 145 | 
 146 |   using ElementQ = typename FMHAKernel::ElementQ;
 147 |   using ElementK = typename FMHAKernel::ElementK;
 148 |   using ElementV = typename FMHAKernel::ElementV;
 149 |   using ElementO = typename FMHAKernel::ElementO;
 150 | 
 151 |   using LayoutQ = typename FMHAConfiguration::LayoutQ;
 152 |   using LayoutK = typename FMHAConfiguration::LayoutK;
 153 |   using LayoutV = typename FMHAConfiguration::LayoutV;
 154 |   using LayoutO = typename FMHAConfiguration::LayoutO;
 155 | 
 156 |   using CollectiveMainloop = typename FMHAKernel::CollectiveMainloop;
 157 |   using ElementS = typename CollectiveMainloop::ElementS;
 158 | 
 159 |   using ProblemShapeType = typename FMHAConfiguration::ProblemShapeType;
 160 |   static constexpr bool Causal = FMHAConfiguration::Causal;
 161 |   static constexpr bool isVarLen = FMHAConfiguration::VarLen;
 162 |   static constexpr bool CachedKV = FMHAConfiguration::CachedKV;
 163 |   static constexpr bool PagedKV = FMHAConfiguration::PagedKV;
 164 |   static constexpr bool Persistent = FMHAConfiguration::Persistent;
 165 | 
 166 |   int32_t count;
 167 | 
 168 |   //
 169 |   // Data members
 170 |   //
 171 | 
 172 |   /// Initialization
 173 |   StrideQ stride_Q;
 174 |   StrideK stride_K;
 175 |   StrideV stride_V;
 176 |   StrideO stride_O;
 177 | 
 178 |   StrideK stride_K_cache;
 179 |   StrideV stride_V_cache;
 180 | 
 181 |   uint64_t seed = 0;
 182 | 
 183 |   cutlass::DeviceAllocation<ElementQ> block_Q;
 184 |   cutlass::DeviceAllocation<ElementK> block_K;
 185 |   cutlass::DeviceAllocation<ElementV> block_V;
 186 |   cutlass::DeviceAllocation<ElementK> block_K_cache;
 187 |   cutlass::DeviceAllocation<ElementV> block_V_cache;
 188 |   cutlass::DeviceAllocation<ElementO> block_O;
 189 |   cutlass::DeviceAllocation<ElementO> block_ref_O;
 190 | 
 191 |   std::vector<int> cumulative_seqlen_q;
 192 |   std::vector<int> cumulative_seqlen_kv;
 193 |   std::vector<int> cumulative_seqlen_kv_cache;
 194 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_q;
 195 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_kv;
 196 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_kv_cache;
 197 | 
 198 |   struct PagedKVParams {
 199 |       cutlass::DeviceAllocation<int> page_table;
 200 |       int page_size = 0;
 201 |       cutlass::DeviceAllocation<int> num_pages_per_seq;
 202 |   };
 203 |   PagedKVParams paged_kv_cache;
 204 | 
```
**EN:** Defines the templated runner, extracts kernel-dependent types, and stores strides, buffers, cumulative lengths, and paged-KV metadata.
**CN:** 定义模板化运行器，提取依赖内核的类型，并保存步长、缓冲区、累计长度和分页 KV 元数据。

### Lines 205-262 — Verification setup
```cpp
 205 |   bool verify(ProblemShapeType shape, bool is_causal) {
 206 | 
 207 |     if constexpr (isVarLen) {
 208 |       int max_seq_len_q = shape.seq_len_qo;
 209 |       int max_seq_len_kv = shape.seq_len_kv;
 210 |       int max_seq_len_kv_cache = shape.seq_len_kv_cache;
 211 |       shape.seq_len_qo = cutlass::fmha::collective::VariableLength{max_seq_len_q, cumulative_seqlen_q.data()};
 212 |       shape.seq_len_kv = cutlass::fmha::collective::VariableLength{max_seq_len_kv, cumulative_seqlen_kv.data()};
 213 |       shape.seq_len_kv_cache = cutlass::fmha::collective::VariableLength{max_seq_len_kv_cache, cumulative_seqlen_kv_cache.data()};
 214 |     }
 215 | 
 216 |     auto batch = shape.batch;
 217 |     auto num_heads_q = shape.num_heads_q;
 218 |     auto num_heads_kv = shape.num_heads_kv;
 219 |     auto head_size_qk = shape.head_size_qk;
 220 |     auto head_size_vo = shape.head_size_vo;
 221 |     int seq_len_qo, seq_len_kv, seq_len_kv_cache;
 222 | 
 223 |     auto block_Q_ = in_memory(block_Q);
 224 |     auto block_K_ = in_memory(block_K);
 225 |     auto block_V_ = in_memory(block_V);
 226 |     auto block_K_cache_ = in_memory(block_K_cache);
 227 |     auto block_V_cache_ = in_memory(block_V_cache);
 228 |     using ElementV_ = ElementV;
 229 |     using ElementK_ = ElementK;
 230 | 
 231 |     int offset_q = 0;
 232 |     int offset_k = 0;
 233 |     int offset_v = 0;
 234 |     int offset_k_cache = 0;
 235 |     int offset_v_cache = 0;
 236 |     int offset_o = 0;
 237 | 
 238 |     std::vector<int> page_table_host;
 239 |     std::vector<int> num_pages_per_seq_host;
 240 |     if (paged_kv_cache.page_size > 0) {
 241 |       page_table_host.resize(paged_kv_cache.page_table.size());
 242 |       compat::memcpy(page_table_host.data(), paged_kv_cache.page_table.get(), page_table_host.size() * sizeof(int));
 243 |       num_pages_per_seq_host.resize(paged_kv_cache.num_pages_per_seq.size());
 244 |       compat::memcpy(num_pages_per_seq_host.data(), paged_kv_cache.num_pages_per_seq.get(), num_pages_per_seq_host.size() * sizeof(int));
 245 |       compat::wait();
 246 |     }
 247 | 
 248 |     // loop over the batch dimension to compute the output
 249 |     // to avoid the risk of running out of device memory
 250 |     int q_group_size = num_heads_q/num_heads_kv;
 251 |     for (int b = 0; b < batch; b++) {
 252 |       if constexpr (isVarLen) {
 253 |         auto logical_seq_shape = cutlass::fmha::collective::apply_variable_length(make_shape(shape.seq_len_qo, shape.seq_len_kv, shape.seq_len_kv_cache), b);
 254 |         seq_len_qo = get<0>(logical_seq_shape);
 255 |         seq_len_kv = get<1>(logical_seq_shape);
 256 |         seq_len_kv_cache = get<2>(logical_seq_shape);
 257 |       } else {
 258 |         seq_len_qo = shape.seq_len_qo;
 259 |         seq_len_kv = shape.seq_len_kv;
 260 |         seq_len_kv_cache = shape.seq_len_kv_cache;
 261 |       }
 262 |       int seq_len_kv_total = seq_len_kv + seq_len_kv_cache;
```
**EN:** Expands variable-length descriptors, loads sequence parameters, and prepares host-visible state for reference attention computation.
**CN:** 展开变长描述符，读取序列参数，并为参考注意力计算准备可在主机侧访问的状态。

### Lines 264-323 — Cached-KV concatenation path
```cpp
 264 |       int kv_group_update=1;
 265 |       for (int h = 0; h < num_heads_q; h++) {
 266 |         cutlass::DeviceAllocation<ElementS> block_S;
 267 |         block_S.reset(seq_len_qo * seq_len_kv_total);
 268 | 
 269 |         ElementK_* k_ptr;
 270 |         ElementV_* v_ptr;
 271 |         cutlass::DeviceAllocation<ElementK_> block_K_concat;
 272 |         cutlass::DeviceAllocation<ElementV_> block_V_concat;
 273 | 
 274 |         if (seq_len_kv_cache > 0) {
 275 |             block_K_concat.reset(head_size_qk * seq_len_kv_total);
 276 |             block_V_concat.reset(seq_len_kv_total * head_size_vo);
 277 | 
 278 |             if (paged_kv_cache.page_size > 0) {
 279 |               int page_size = paged_kv_cache.page_size;
 280 |               int start_page_idx = isVarLen ? num_pages_per_seq_host[b] : b * (seq_len_kv_cache / page_size);
 281 |               int num_pages = ceil_div(seq_len_kv_cache, page_size);
 282 | 
 283 |               for (int i = 0; i < num_pages; ++i) {
 284 |                 int physical_page_id = page_table_host[start_page_idx + i];
 285 |                 int current_copy_len = std::min(page_size, seq_len_kv_cache - i * page_size);
 286 | 
 287 |                 compat::memcpy<ElementK_>(
 288 |                     block_K_concat.get() + head_size_qk * i * page_size,
 289 |                     block_K_cache_.get() + offset_k_cache + head_size_qk * physical_page_id * page_size,
 290 |                     head_size_qk * current_copy_len);
 291 |                 
 292 |                 compat::memcpy<ElementV_>(
 293 |                     block_V_concat.get() + i * page_size * head_size_vo,
 294 |                     block_V_cache_.get() + offset_v_cache + physical_page_id * page_size * head_size_vo,
 295 |                     current_copy_len * head_size_vo);
 296 |               }
 297 |             } else {
 298 |               compat::memcpy<ElementK_>(
 299 |                     block_K_concat.get(),
 300 |                     block_K_cache_.get() + offset_k_cache,
 301 |                     head_size_qk * seq_len_kv_cache);
 302 |               compat::memcpy<ElementV_>(
 303 |                     block_V_concat.get(),
 304 |                     block_V_cache_.get() + offset_v_cache,
 305 |                     seq_len_kv_cache * head_size_vo);
 306 |             }
 307 | 
 308 |             compat::memcpy<ElementK_>(
 309 |                   block_K_concat.get() + head_size_qk * seq_len_kv_cache,
 310 |                   block_K_.get() + offset_k,
 311 |                   head_size_qk * seq_len_kv);
 312 | 
 313 |             compat::memcpy<ElementV_>(
 314 |                   block_V_concat.get() + seq_len_kv_cache * head_size_vo,
 315 |                   block_V_.get() + offset_v,
 316 |                   seq_len_kv * head_size_vo);
 317 | 
 318 |             k_ptr = block_K_concat.get();
 319 |             v_ptr = block_V_concat.get();
 320 |         } else {
 321 |             k_ptr = block_K_.get() + offset_k;
 322 |             v_ptr = block_V_.get() + offset_v;
 323 |         }
```
**EN:** Builds a logical K/V sequence by combining cached pages or contiguous cache tensors with the current K/V inputs before verification.
**CN:** 在校验前，把分页缓存或连续缓存张量与当前 K/V 输入拼接，构造逻辑上的完整 K/V 序列。

### Lines 325-460 — Reference attention computation
```cpp
 325 |         cutlass::TensorRef ref_Q(block_Q_.get() + offset_q, LayoutQ::packed({seq_len_qo, head_size_qk}));
 326 |         cutlass::TensorRef ref_K(k_ptr, LayoutK::packed({head_size_qk, seq_len_kv_total}));
 327 |         cutlass::TensorRef ref_V(v_ptr, LayoutV::packed({seq_len_kv_total, head_size_vo}));
 328 |         cutlass::TensorRef ref_S(block_S.get(), LayoutQ::packed({seq_len_qo, seq_len_kv_total}));
 329 | 
 330 |         cutlass::reference::device::GemmComplex({seq_len_qo, seq_len_kv_total, head_size_qk}, 1.f, ref_Q,
 331 |                                                 cutlass::ComplexTransform::kNone, ref_K, cutlass::ComplexTransform::kNone,
 332 |                                                 0.f, ref_S, ref_S, ElementS(0),
 333 |                                                 1,                   // batch_count
 334 |                                                 seq_len_qo * head_size_qk, // batch_stride_Q
 335 |                                                 seq_len_kv_total * head_size_qk, // batch_stride_K
 336 |                                                 seq_len_qo * seq_len_kv_total,   // batch_stride_S
 337 |                                                 seq_len_qo * seq_len_kv_total    // batch_stride_S
 338 |         );
 339 | 
 340 |         compat::wait();
 341 | 
 342 |         std::vector<ElementS> host_S(block_S.size());
 343 |         compat::memcpy<ElementS>(host_S.data(), block_S.get(), host_S.size());
 344 | 
 345 |         // delete this memory as it is no longer needed
 346 |         block_S.reset();
 347 |         auto offset = cute::min(seq_len_qo, seq_len_kv);
 348 |         auto discard_seq_coord = seq_len_qo - offset;
 349 |         auto full_tile_offset = seq_len_kv - offset;
 350 |         if (is_causal) {
 351 |           // apply mask to S
 352 |           for (int row = 0; row < seq_len_qo; row++) {
 353 |             for (int col = seq_len_kv_cache; col < seq_len_kv_total; col++) {
 354 |               if ((col - seq_len_kv_cache - full_tile_offset) > (row - discard_seq_coord))
 355 |                 host_S[col + row * seq_len_kv_total] = ElementS{-INFINITY};
 356 |             }
 357 |           }
 358 |         }
 359 | 
 360 |         // compute max element per row of S
 361 |         std::vector<ElementS> max_vec(seq_len_qo, ElementS{-INFINITY});
 362 |         for (int row = 0; row < seq_len_qo; row++) {
 363 |           int idx = row * seq_len_kv_total;
 364 |           int max_idx = row;
 365 |           max_vec[max_idx] = host_S[idx++];
 366 |           for (int col = 1; col < seq_len_kv_total; col++, idx++) {
 367 |             if (max_vec[max_idx] < host_S[idx])
 368 |               max_vec[max_idx] = host_S[idx];
 369 |           }
 370 |         }
 371 | 
 372 |         // compute exp of S
 373 |         for (int row = 0; row < seq_len_qo; row++) {
 374 |           int idx = row * seq_len_kv_total;
 375 |           int max_idx = row;
 376 |           for (int col = 0; col < seq_len_kv_total; col++, idx++) {
 377 |             /* FIXME: use softmax_scale instead of assuming its value here */
 378 |             host_S[idx] = expf((host_S[idx] - max_vec[max_idx]) / std::sqrt(static_cast<ElementS>((head_size_qk))));
 379 |           }
 380 |         }
 381 | 
 382 |         // compute sum per row of S
 383 |         std::vector<ElementS> sum_vec(seq_len_qo, ElementS{0});
 384 |         for (int row = 0; row < seq_len_qo; row++) {
 385 |           int idx = row * seq_len_kv_total;
 386 |           int sum_idx = row;
 387 |           for (int col = 0; col < seq_len_kv_total; col++, idx++) {
 388 |             sum_vec[sum_idx] += host_S[idx];
 389 |           }
 390 | 
 391 |           // scale each row with the sum to compute softmax
 392 |           idx = row * seq_len_kv_total;
 393 |           sum_idx = row;
 394 |           for (int col = 0; col < seq_len_kv_total; col++, idx++) {
 395 |             if(is_causal && row < discard_seq_coord) {
 396 |               host_S[idx] = 0;
 397 |             } else {
 398 |               host_S[idx] /= sum_vec[sum_idx];
 399 |             }
 400 |           }
 401 |         }
 402 | 
 403 |         std::vector<ElementV_> host_P(host_S.size());
 404 |         for (int p = 0; p < host_P.size(); p++)
 405 |           host_P[p] = static_cast<ElementV_>(host_S[p]);
 406 | 
 407 |         cutlass::DeviceAllocation<ElementV_> block_P;
 408 |         block_P.reset(host_P.size());
 409 | 
 410 |         compat::memcpy<ElementV_>(block_P.get(), host_P.data(), host_P.size());
 411 | 
 412 |         cutlass::TensorRef ref_P(block_P.get(), LayoutQ::packed({seq_len_qo, seq_len_kv_total}));
 413 | 
 414 |         cutlass::DeviceAllocation<ElementS> block_acc;
 415 |         block_acc.reset(seq_len_qo * head_size_vo);
 416 |         cutlass::TensorRef ref_acc(block_acc.get(), LayoutO::packed({seq_len_qo, head_size_vo}));
 417 | 
 418 |         cutlass::reference::device::GemmComplex({seq_len_qo, head_size_vo, seq_len_kv_total}, ElementS{1}, ref_P,
 419 |                                                 cutlass::ComplexTransform::kNone, ref_V, cutlass::ComplexTransform::kNone,
 420 |                                                 ElementS{0}, ref_acc, ref_acc, ElementS{0},
 421 |                                                 1,                   // batch_count
 422 |                                                 seq_len_qo * seq_len_kv_total,   // batch_stride_P
 423 |                                                 seq_len_kv_total * head_size_vo, // batch_stride_V
 424 |                                                 seq_len_qo * head_size_vo, // batch_stride_O
 425 |                                                 seq_len_qo * head_size_vo  // batch_stride_O
 426 |         );
 427 | 
 428 |         compat::wait();
 429 |         // delete this memory as it is no longer needed
 430 |         block_P.reset();
 431 | 
 432 |         std::vector<ElementS> vec_acc(block_acc.size());
 433 |         compat::memcpy<ElementS>(vec_acc.data(), block_acc.get(), vec_acc.size());
 434 | 
 435 |         // delete this memory as it is no longer needed
 436 |         block_acc.reset();
 437 |         std::vector<ElementO> vec_out(vec_acc.size());
 438 |         for(int i = 0; i < vec_out.size(); i++) {
 439 |           vec_out[i] = static_cast<ElementO>(vec_acc[i]);
 440 |         }
 441 |         compat::memcpy<ElementO>(block_ref_O.get() + offset_o, vec_out.data(), vec_out.size());
 442 | 
 443 |         offset_q += seq_len_qo * head_size_qk;
 444 |         if(kv_group_update % q_group_size==0) {
 445 |           offset_k += seq_len_kv * head_size_qk;
 446 |           offset_v += seq_len_kv * head_size_vo;
 447 |           offset_k_cache += seq_len_kv_cache * head_size_qk;
 448 |           offset_v_cache += seq_len_kv_cache * head_size_vo;
 449 |         }
 450 |         kv_group_update++;
 451 |         offset_o += seq_len_qo * head_size_vo;
 452 |       }
 453 |     }
 454 | 
 455 |     compat::wait();
 456 | 
 457 |     // Check if output from CUTLASS kernel and reference kernel are equal or not
 458 |     bool passed = cutlass::reference::device::BlockCompareRelativelyEqual(block_ref_O.get(), block_O.get(),
 459 |                                                                           block_O.size(), ElementO{0.05}, ElementO{0.05});
 460 | 
```
**EN:** Computes QK, applies causal masking, performs softmax row by row, multiplies by V, writes the reference output, and compares it with the kernel result.
**CN:** 计算 QK，应用因果 mask，逐行执行 softmax，再与 V 相乘，写出参考输出，并与内核结果比较。

### Lines 464-535 — Variable-length initialization
```cpp
 464 |   template<class ProblemShape>
 465 |   auto initialize_varlen(const ProblemShape& problem_size) {
 466 |     int num_batches = get<0>(problem_size);
 467 | 
 468 |     // generate Q as --b times
 469 |     //    gaussian (--Q, --Q / 2) sampled positive
 470 |     //    track cumulative 
 471 |     std::mt19937 rng(0x202305151552ull);
 472 |     std::normal_distribution<double> dist_q(get<3>(problem_size), get<3>(problem_size) / 2);
 473 |     std::normal_distribution<double> dist_kv(get<4>(problem_size), get<4>(problem_size) / 2);
 474 |     std::normal_distribution<double> dist_kv_cache(get<5>(problem_size), get<5>(problem_size) / 2);
 475 | 
 476 |     // Use Cacheline Size to calculate alignment
 477 |     constexpr int cacheline_bytes = 64;
 478 |     constexpr int AlignmentQ = cacheline_bytes / sizeof(ElementQ);    // Alignment of Q matrix in units of elements
 479 |     constexpr int AlignmentKV = cacheline_bytes / sizeof(ElementK);   // Alignment of Kand V matrix in units of elements
 480 |     constexpr int AlignmentKVCache = 128; // Page size must be a multiple of 128
 481 |     auto generate_positive_int = [](auto& dist, auto& gen) {
 482 |       int result = 0;
 483 |       do {
 484 |         result = static_cast<int>(dist(gen));
 485 |       } while (result <= 0);
 486 |       return result;
 487 |     };
 488 | 
 489 |     cumulative_seqlen_q = {0};
 490 |     cumulative_seqlen_kv = {0};
 491 |     cumulative_seqlen_kv_cache = {0};
 492 | 
 493 |     int total_seqlen_q = 0;
 494 |     int total_seqlen_kv = 0;
 495 |     int total_seqlen_kv_cache = 0;
 496 |     int max_seqlen_q = 0;
 497 |     int max_seqlen_kv = 0;
 498 |     int max_seqlen_kv_cache = 0;
 499 | 
 500 |     for (int i = 0; i < num_batches; i++) {
 501 |       int seqlen_q = cutlass::round_up(generate_positive_int(dist_q, rng), AlignmentQ);
 502 |       int seqlen_kv = cutlass::round_up(generate_positive_int(dist_kv, rng), AlignmentKV);
 503 |       int seqlen_kv_cache = get<5>(problem_size) == 0 ? 0 : cutlass::round_up(generate_positive_int(dist_kv_cache, rng), AlignmentKVCache);
 504 | 
 505 |       total_seqlen_q += seqlen_q;
 506 |       total_seqlen_kv += seqlen_kv;
 507 |       total_seqlen_kv_cache += seqlen_kv_cache;
 508 | 
 509 |       max_seqlen_q = std::max(max_seqlen_q, seqlen_q);
 510 |       max_seqlen_kv = std::max(max_seqlen_kv, seqlen_kv);
 511 |       max_seqlen_kv_cache = std::max(max_seqlen_kv_cache, seqlen_kv_cache);
 512 | 
 513 |       cumulative_seqlen_q.push_back(cumulative_seqlen_q.back() + seqlen_q);
 514 |       cumulative_seqlen_kv.push_back(cumulative_seqlen_kv.back() + seqlen_kv);
 515 |       cumulative_seqlen_kv_cache.push_back(cumulative_seqlen_kv_cache.back() + seqlen_kv_cache);
 516 |     }
 517 | 
 518 |     ProblemShape problem_size_for_init = problem_size;
 519 |     get<0>(problem_size_for_init) = 1;
 520 |     get<3>(problem_size_for_init) = total_seqlen_q;
 521 |     get<4>(problem_size_for_init) = total_seqlen_kv;
 522 |     get<5>(problem_size_for_init) = total_seqlen_kv_cache;
 523 | 
 524 |     ProblemShapeType problem_size_for_launch;
 525 |     problem_size_for_launch.batch = get<0>(problem_size);
 526 |     problem_size_for_launch.num_heads_q = get<1>(problem_size);
 527 |     problem_size_for_launch.num_heads_kv = get<2>(problem_size);
 528 |     problem_size_for_launch.seq_len_qo = cutlass::fmha::collective::VariableLength{max_seqlen_q};
 529 |     problem_size_for_launch.seq_len_kv = cutlass::fmha::collective::VariableLength{max_seqlen_kv};
 530 |     problem_size_for_launch.seq_len_kv_cache = cutlass::fmha::collective::VariableLength{max_seqlen_kv_cache};
 531 |     problem_size_for_launch.head_size_qk = get<6>(problem_size);
 532 |     problem_size_for_launch.head_size_vo = get<7>(problem_size);
 533 | 
 534 |     return cute::make_tuple(problem_size_for_init, problem_size_for_launch);
 535 |   }
```
**EN:** Samples positive random sequence lengths, rounds them for alignment, records cumulative offsets, and produces shapes for allocation versus launch.
**CN:** 采样正的随机序列长度，并按对齐要求向上取整，记录累计偏移，同时生成用于分配和启动的形状。

### Lines 537-642 — Buffer and stride initialization
```cpp
 537 |   ProblemShapeType initialize(const FMHAOptions &options) {
 538 |     auto problem_shape_in = cute::make_tuple(options.batch, options.num_heads_q, options.num_heads_kv, options.seq_len_qo, options.seq_len_kv, options.seq_len_kv_cache,options.head_size_qk, options.head_size_vo);
 539 |     ProblemShapeType shape;
 540 | 
 541 |     decltype(problem_shape_in) problem_size;
 542 | 
 543 |     if constexpr (isVarLen) {
 544 |       auto [problem_shape_init, problem_shape_launch] = initialize_varlen(problem_shape_in);
 545 |       problem_size = problem_shape_init;
 546 |       shape = problem_shape_launch;
 547 |     } else {
 548 |       problem_size = problem_shape_in;
 549 |       shape.batch        = options.batch;
 550 |       shape.num_heads_q  = options.num_heads_q;
 551 |       shape.num_heads_kv = options.num_heads_kv;
 552 |       shape.seq_len_qo   = options.seq_len_qo;
 553 |       shape.seq_len_kv   = options.seq_len_kv;
 554 |       shape.seq_len_kv_cache = options.seq_len_kv_cache;
 555 |       shape.head_size_qk = options.head_size_qk;
 556 |       shape.head_size_vo = options.head_size_vo;
 557 |     }
 558 | 
 559 |     auto [batch, num_heads_q, num_heads_kv, seq_len_qo, seq_len_kv, seq_len_kv_cache, head_size_qk, head_size_vo] = problem_size;
 560 |     auto shape_Q = cute::make_shape(seq_len_qo, head_size_qk, num_heads_q,  batch);
 561 |     auto shape_K = cute::make_shape(seq_len_kv, head_size_qk, num_heads_kv, batch);
 562 |     auto shape_V = cute::make_shape(head_size_vo, seq_len_kv, num_heads_kv, batch);
 563 |     auto shape_K_cache = cute::make_shape(seq_len_kv_cache, head_size_qk, num_heads_kv, batch);
 564 |     auto shape_V_cache = cute::make_shape(head_size_vo, seq_len_kv_cache, num_heads_kv, batch);
 565 |     auto shape_O = cute::make_shape(seq_len_qo, head_size_vo, num_heads_q,  batch);
 566 | 
 567 |     stride_Q = cutlass::make_cute_packed_stride(StrideQ{}, shape_Q);
 568 |     stride_K = cutlass::make_cute_packed_stride(StrideK{}, shape_K);
 569 |     stride_V = cutlass::make_cute_packed_stride(StrideV{}, shape_V);
 570 |     stride_K_cache = cutlass::make_cute_packed_stride(StrideK{}, shape_K_cache);
 571 |     stride_V_cache = cutlass::make_cute_packed_stride(StrideV{}, shape_V_cache);
 572 |     stride_O = cutlass::make_cute_packed_stride(StrideO{}, shape_O);
 573 | 
 574 |     block_Q.reset(static_cast<std::size_t>(batch) * num_heads_q * seq_len_qo * head_size_qk);
 575 |     block_K.reset(static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv * head_size_qk);
 576 |     block_V.reset(static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv * head_size_vo);
 577 |     block_K_cache.reset(static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv_cache * head_size_qk);
 578 |     block_V_cache.reset(static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv_cache * head_size_vo);
 579 |     block_O.reset(static_cast<std::size_t>(batch) * num_heads_q * seq_len_qo * head_size_vo);
 580 |     block_ref_O.reset(static_cast<std::size_t>(batch) * num_heads_q * seq_len_qo * head_size_vo);
 581 | 
 582 |     // Zero-initialize output buffer for the kernel result
 583 |     // block_ref_O is fully written in verify() before being read, so no initialization needed
 584 |     compat::memset(block_O.get(), 0, block_O.size() * sizeof(ElementO));
 585 |     if (PagedKV) {
 586 |       paged_kv_cache.page_size = options.page_size;
 587 |       std::vector<int> num_pages_per_seq{0};
 588 |       int num_pages = 0;
 589 |       for(int b = 0; b < shape.batch; b++) {
 590 |         int seq_len_cache = isVarLen ? cumulative_seqlen_kv_cache[b + 1] - cumulative_seqlen_kv_cache[b] : seq_len_kv_cache;
 591 |         int pages_per_seq = ceil_div(seq_len_cache, paged_kv_cache.page_size);
 592 |         num_pages_per_seq.push_back(num_pages_per_seq.back() + pages_per_seq);
 593 |         num_pages += pages_per_seq;
 594 |       }
 595 |       paged_kv_cache.page_table.reset(num_pages);
 596 | 
 597 |       // initialize block table with random mapping for non-contiguous layout
 598 |       std::vector<int> page_mapping(num_pages);
 599 |       for (int b = 0; b < shape.batch; ++b) {
 600 |         std::vector<int> physical_pages(num_pages_per_seq[b + 1] - num_pages_per_seq[b]);
 601 |         std::iota(physical_pages.begin(), physical_pages.end(), 0);
 602 |         // shuffle physical pages
 603 |         std::shuffle(physical_pages.begin(), physical_pages.end(), std::mt19937{ std::random_device{}() });
 604 |         for (int blk = 0; blk < physical_pages.size(); ++blk) {
 605 |           int logical_idx = num_pages_per_seq[b] + blk;
 606 |           page_mapping[logical_idx] = physical_pages[blk];
 607 |         }
 608 |       }
 609 |       compat::memcpy(paged_kv_cache.page_table.get(), page_mapping.data(), page_mapping.size() * sizeof(int));
 610 | 
 611 |       paged_kv_cache.num_pages_per_seq.reset(num_pages_per_seq.size());
 612 |       compat::memcpy(paged_kv_cache.num_pages_per_seq.get(), num_pages_per_seq.data(), num_pages_per_seq.size() * sizeof(int));
 613 |     }
 614 | 
 615 |     initialize_block(block_Q, seed + 2023);
 616 |     initialize_block(block_K, seed + 2022);
 617 |     initialize_block(block_V, seed + 2021);
 618 |     initialize_block(block_K_cache, seed + 2024);
 619 |     initialize_block(block_V_cache, seed + 2025);
 620 |     
 621 |     if (!cumulative_seqlen_q.empty()) {
 622 |       device_cumulative_seqlen_q.reset(cumulative_seqlen_q.size());
 623 |       device_cumulative_seqlen_q.copy_from_host(cumulative_seqlen_q.data(), cumulative_seqlen_q.size());
 624 |     }
 625 | 
 626 |     if (!cumulative_seqlen_kv.empty()) {
 627 |       device_cumulative_seqlen_kv.reset(cumulative_seqlen_kv.size());
 628 |       device_cumulative_seqlen_kv.copy_from_host(cumulative_seqlen_kv.data(), cumulative_seqlen_kv.size());
 629 |     }
 630 | 
 631 |     if (!cumulative_seqlen_kv_cache.empty()) {
 632 |       device_cumulative_seqlen_kv_cache.reset(cumulative_seqlen_kv_cache.size());
 633 |       device_cumulative_seqlen_kv_cache.copy_from_host(cumulative_seqlen_kv_cache.data(), cumulative_seqlen_kv_cache.size());
 634 |     }
 635 | 
 636 |     if constexpr (isVarLen) {
 637 |       shape.seq_len_qo.cumulative_length = device_cumulative_seqlen_q.get();
 638 |       shape.seq_len_kv.cumulative_length = device_cumulative_seqlen_kv.get();
 639 |       shape.seq_len_kv_cache.cumulative_length = device_cumulative_seqlen_kv_cache.get();
 640 |     }
 641 | 
 642 |     return shape;
```
**EN:** Builds packed strides, allocates Q/K/V/cache/output buffers, initializes page tables for paged KV, fills random inputs, and uploads cumulative-length arrays.
**CN:** 构建紧凑步长，分配 Q/K/V/缓存/输出缓冲区，为分页 KV 初始化页表，填充随机输入，并上传累计长度数组。

### Lines 645-678 — Kernel launch wrapper
```cpp
 645 |   static void run(typename FMHAKernel::Params params) {
 646 |    
 647 |     namespace syclex = sycl::ext::oneapi::experimental;
 648 |     namespace intelex = sycl::ext::intel::experimental;
 649 |    
 650 |     dim3 const block = FMHAKernel::get_block_shape();
 651 |     dim3 const grid = FMHAKernel::get_grid_shape(params);
 652 | 
 653 |     // configure smem size and carveout
 654 |     int smem_size = FMHAKernel::SharedStorageSize;
 655 | 
 656 |     const auto sycl_block = compat::dim3(block.x, block.y, block.z);
 657 |     const auto sycl_grid = compat::dim3(grid.x, grid.y, grid.z);
 658 | 
 659 |     // Launch parameters depend on whether SYCL compiler supports work-group scratch memory extension
 660 |     compat::experimental::launch_properties launch_props {
 661 |       syclex::work_group_scratch_size(smem_size),
 662 |     };
 663 |     compat::experimental::kernel_properties kernel_props{
 664 |       syclex::sub_group_size<cute::intel::sg_size>,
 665 | #if (SYCL_INTEL_TARGET == 35)
 666 |       intelex::grf_size<512>
 667 | #else
 668 |       intelex::grf_size<256>
 669 | #endif
 670 |     };
 671 |     compat::experimental::launch_policy policy{sycl_grid, sycl_block, launch_props, kernel_props};
 672 | #if defined(CUTLASS_SYCL_PROFILING_ENABLED)
 673 |     auto event = compat::experimental::launch<cutlass::device_kernel<FMHAKernel>, FMHAKernel>(policy, params);
 674 |     EventManager::getInstance().addEvent(event);
 675 | #else
 676 |     compat::experimental::launch<cutlass::device_kernel<FMHAKernel>, FMHAKernel, false>(policy, params);
 677 | #endif
 678 |   }
```
**EN:** Translates CUTLASS launch information into SYCL launch properties, configures scratch memory and subgroup/GRF settings, and submits the kernel.
**CN:** 把 CUTLASS 启动信息转换成 SYCL 启动属性，配置 scratch memory 与 subgroup/GRF 设置，并提交内核。

### Lines 680-747 — Warmup and validation run
```cpp
 680 |   void run(::benchmark::State& state, const FMHAOptions &options, const cutlass::KernelHardwareInfo &hw_info) {
 681 | 
 682 |     ProblemShapeType problem_size = initialize(options);
 683 | 
 684 |     typename FMHAKernel::Arguments arguments{
 685 |       {
 686 |       problem_size,
 687 |       block_Q.get(), stride_Q,
 688 |       block_K.get(), stride_K,
 689 |       block_V.get(), stride_V,
 690 |       block_O.get(), stride_O,
 691 |       block_K_cache.get(), stride_K_cache,
 692 |       block_V_cache.get(), stride_V_cache,
 693 |       },
 694 |       {
 695 |       options.softmax_scale,
 696 |       PagedKV ? paged_kv_cache.page_table.get() : nullptr,
 697 |       PagedKV ? paged_kv_cache.page_size : 0,
 698 |       PagedKV ? paged_kv_cache.num_pages_per_seq.get() : nullptr
 699 |       },
 700 |       {},
 701 |       hw_info
 702 |     };
 703 | 
 704 |     size_t workspace_size = FMHAKernel::get_workspace_size(arguments);
 705 |     cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
 706 | 
 707 |     FMHAKernel::can_implement(arguments);
 708 | 
 709 |     // Initialize the workspace
 710 |     auto status = FMHAKernel::initialize_workspace(arguments, workspace.get());
 711 |     if (status != cutlass::Status::kSuccess) {
 712 |       return;
 713 |     }
 714 | 
 715 |     typename FMHAKernel::Params params = FMHAKernel::to_underlying_arguments(arguments, workspace.get());
 716 | 
 717 |     // Run the GEMM
 718 |     run(params);
 719 | 
 720 |     compat::wait();
 721 | 
 722 |     // Verify that the result is correct
 723 |     bool passed = verify(problem_size, Causal);
 724 |     if(not passed) {
 725 |       state.SkipWithError("Disposition Failed.");
 726 |     }
 727 | 
 728 |     state.counters["batch"] = options.batch;
 729 |     state.counters["num_heads_q"] = options.num_heads_q;
 730 |     state.counters["num_heads_kv"] = options.num_heads_kv;
 731 |     state.counters["seq_len_qo"] = options.seq_len_qo;
 732 |     state.counters["seq_len_kv"] = options.seq_len_kv;
 733 |     state.counters["seq_len_kv_cache"] = options.seq_len_kv_cache;
 734 |     state.counters["head_size_kv"] = options.head_size_qk;
 735 |     state.counters["head_size_vo"] = options.head_size_vo;
 736 |     state.counters["page_size"] = options.page_size;
 737 |     state.counters["scale"] = options.softmax_scale;
 738 |     state.counters["causal"] = Causal;
 739 |     state.counters["varlen"] = isVarLen;
 740 |     state.counters["paged_kv"] = PagedKV;
 741 | 
 742 |     std::stringstream extra_label;
 743 |     extra_label << "layoutQ=RowMajor ";
 744 |     extra_label << "layoutK=ColumnMajor ";
 745 |     extra_label << "layoutV=RowMajor ";
 746 | 
 747 |     state.SetLabel(extra_label.str());
```
**EN:** Builds kernel arguments, allocates workspace, initializes the kernel, performs an initial run, verifies correctness, and records descriptive counters.
**CN:** 构建内核参数，分配工作区，初始化内核，执行首次运行，校验正确性，并记录描述性计数器。

### Lines 748-815 — Timed loop and throughput math
```cpp
 748 |     // when seq_len_qo is not equal to seq_len_kv we use bottom up approach for the masking. 
 749 |     // Following changes will adjust the effective_seq_len_kv when masking applied for such cases.
 750 |     auto offset = cute::min(options.seq_len_qo, options.seq_len_kv);
 751 |     auto discard_seq_coord = options.seq_len_qo - offset;
 752 |     auto full_tile_offset = options.seq_len_kv - offset;
 753 |     auto effective_seq_len_kv = Causal ? full_tile_offset + ((offset + 1) / 2.0): options.seq_len_kv;
 754 |     auto effective_seq_len_qo = Causal ? options.seq_len_qo - discard_seq_coord  : options.seq_len_qo;
 755 |    
 756 |     double flops_qk = 2.0 * options.batch * options.num_heads_q * effective_seq_len_qo * effective_seq_len_kv * options.head_size_qk;
 757 |     double flops_pv = 2.0 * options.batch * options.num_heads_q * effective_seq_len_qo * options.head_size_vo * effective_seq_len_kv;
 758 |     double gflops = (flops_qk + flops_pv) * 1e-9;
 759 | 
 760 |     // TODO: Use sizeof_bits_v instead of sizeof if QKVO is smaller than 8 bits, which avoids incorrect bandwidth calculation
 761 |     double gbps_qk =  options.batch * (sizeof(ElementQ) * options.num_heads_q * effective_seq_len_qo * options.head_size_qk + 
 762 |                       sizeof(ElementK) * options.num_heads_kv * effective_seq_len_kv * options.head_size_qk);    
 763 |     double gbps_pv = sizeof(ElementV) * options.batch * options.num_heads_kv * effective_seq_len_kv * options.head_size_vo +
 764 |                      sizeof(ElementO) * options.batch * options.num_heads_q * effective_seq_len_qo * options.head_size_vo;
 765 |     double mega_bytes_transferred = (gbps_qk + gbps_pv) * (1e-6);
 766 | 
 767 |     initialize_counters(state);
 768 |     int32_t counter = 1;
 769 |     for(auto _ : state) {
 770 |       state.PauseTiming();
 771 | 
 772 |       typename FMHAKernel::Arguments arguments{
 773 |         {
 774 |           problem_size,
 775 |           block_Q.get(), stride_Q,
 776 |           block_K.get(), stride_K,
 777 |           block_V.get(), stride_V,
 778 |           block_O.get(), stride_O,
 779 |           block_K_cache.get(), stride_K_cache,
 780 |           block_V_cache.get(), stride_V_cache,
 781 |         },
 782 |         {
 783 |           options.softmax_scale,
 784 |           PagedKV ? paged_kv_cache.page_table.get() : nullptr,
 785 |           PagedKV ? paged_kv_cache.page_size : 0,
 786 |           PagedKV ? paged_kv_cache.num_pages_per_seq.get() : nullptr,
 787 |         },
 788 |         {},
 789 |         hw_info
 790 |       };
 791 | 
 792 |       size_t workspace_size = FMHAKernel::get_workspace_size(arguments);
 793 |       cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
 794 | 
 795 |       FMHAKernel::can_implement(arguments);
 796 | 
 797 |       // Initialize the workspace
 798 |       auto status = FMHAKernel::initialize_workspace(arguments, workspace.get());
 799 |       if (status != cutlass::Status::kSuccess) {
 800 |         return;
 801 |       }
 802 | 
 803 |       typename FMHAKernel::Params params = FMHAKernel::to_underlying_arguments(arguments, workspace.get());
 804 | 
 805 |       state.ResumeTiming();
 806 | 
 807 |       GPU_Clock timer;
 808 |       timer.start();
 809 |       run(params);
 810 |       auto ms_elapsed = timer.milliseconds();
 811 |       update_counters(state, ms_elapsed);
 812 |       state.SetIterationTime(ms_elapsed / 1000);
 813 |       counter++;
 814 |     }
 815 |     finalize_counters(state, gflops, mega_bytes_transferred);
```
**EN:** Computes effective work for causal masking, derives FLOP and bandwidth estimates, and repeats timed launches while updating benchmark counters.
**CN:** 根据因果 mask 计算有效工作量，推导 FLOP 与带宽估计，并在更新计数器的同时重复执行计时启动。

### Lines 818-852 — Counter helpers and macros
```cpp
 818 | private:
 819 |   static void initialize_counters(::benchmark::State& state) {
 820 |     state.counters["avg_runtime_ms"] = 0;
 821 |     state.counters["best_runtime_ms"] = std::numeric_limits<double>::max();
 822 |   }
 823 | 
 824 |   static void update_counters(::benchmark::State& state, double ms_elapsed) {
 825 |     state.PauseTiming();
 826 |     state.counters["total_runtime_ms"] += ms_elapsed;
 827 |     state.counters["best_runtime_ms"] = std::min<double>(state.counters["best_runtime_ms"], ms_elapsed);
 828 |     state.ResumeTiming();
 829 |   }
 830 | 
 831 |   static void finalize_counters(::benchmark::State& state,  double gflop, double mega_bytes_transferred) {
 832 |     state.counters["avg_runtime_ms"] =
 833 |       state.counters["total_runtime_ms"] / static_cast<double>(state.iterations());
 834 |     state.counters["avg_tflops"] = gflop / state.counters["avg_runtime_ms"];
 835 |     state.counters["avg_throughput"] = mega_bytes_transferred / state.counters["avg_runtime_ms"];
 836 |     state.counters["best_tflop"] = gflop / state.counters["best_runtime_ms"];
 837 |     state.counters["best_bandwidth"] = mega_bytes_transferred / state.counters["best_runtime_ms"];
 838 |   }
 839 | };
 840 | 
 841 | }
 842 | 
 843 | #define CUTLASS_FMHA_BENCHMARK(F) cutlass::benchmark::BenchmarkRegistry<cutlass::benchmark::FMHAOptions>::Register(#F, &F##_func)
 844 | 
 845 | #define CUTLASS_CREATE_FMHA_BENCHMARK(F)                          \
 846 |   static void F##_func(                                           \
 847 |       ::benchmark::State& state,                                  \
 848 |       cutlass::benchmark::FMHAOptions const& options,                 \
 849 |       cutlass::KernelHardwareInfo const& hw_info) {               \
 850 |     auto bench = cutlass::benchmark::BenchmarkRunnerFMHA<F>();    \
 851 |     bench.run(state, options, hw_info);                           \
 852 |   }
```
**EN:** Finalizes aggregate timing metrics and exposes macros that wrap a configuration type as a Google Benchmark entry.
**CN:** 收尾生成聚合计时指标，并提供宏把配置类型包装成 Google Benchmark 条目。

## Key Concepts / 关键概念

- FMHA option parsing / FMHA 选项解析
- Reference attention verification / 参考注意力结果校验
- Varlen and cached-KV handling / 变长与缓存 KV 处理
- SYCL kernel launch and counters / SYCL 内核启动与性能计数

## Dependencies / 依赖关系

- `flash_attention_v2/*` — FMHA kernels and scheduler / FMHA 内核与调度器
- `cutlass::DeviceAllocation` — device buffers / 设备缓冲区
- `cutlass::reference::device::GemmComplex` — reference GEMM / 参考 GEMM
- `GPU_Clock` and `benchmark::State` — timing / 计时
