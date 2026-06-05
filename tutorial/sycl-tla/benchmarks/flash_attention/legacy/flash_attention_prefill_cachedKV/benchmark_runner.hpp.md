# benchmark_runner.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/legacy/flash_attention_prefill_cachedKV/benchmark_runner.hpp`
- **EN:** Legacy cached-KV prefill harness that extends the prefill runner with cache tensors, cache-aware verification, and cache-length bookkeeping.
- **CN:** 旧版 cached-KV prefill 运行器：在 prefill 运行器基础上增加缓存张量、缓存感知校验和缓存长度管理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
   3 |  * Copyright (C) 2025 - 2026 Intel Corporation, All rights reserved.
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
  32 | #pragma once
  33 | 
  34 | #include "cutlass/epilogue/collective/default_epilogue.hpp"
  35 | #include "cutlass/epilogue/fusion/xe_callbacks.hpp"
  36 | #include "cutlass/gemm/device/gemm_universal_adapter.h"
  37 | #include "cutlass/util/packed_stride.hpp"
  38 | #include "flash_attention_v2/collective/fmha_fusion.hpp"
  39 | #include "flash_attention_v2/kernel/legacy/tile_scheduler_cachedKV.hpp"
  40 | #include "flash_attention_v2/kernel/legacy/xe_flash_attn_prefill_cachedKV.hpp"
  41 | #include "flash_attention_v2/collective/legacy/xe_flash_attn_prefill_epilogue_cachedKV.hpp"
  42 | #include "flash_attention_v2/collective/legacy/xe_flash_attn_prefill_softmax_epilogue.hpp"
  43 | #include "cutlass/util/GPU_Clock.hpp"
  44 | #include "cutlass/util/sycl_event_manager.hpp"
  45 | 
  46 | #include <cute/tensor.hpp>
  47 | #include <random>
  48 | 
  49 | #include "cutlass/util/command_line.h"
  50 | #include "cutlass/util/device_memory.h"
  51 | #include "cutlass/util/reference/device/gemm_complex.h"
  52 | #include "cutlass/util/reference/device/tensor_compare.h"
  53 | #include "../examples/common/sycl_common.hpp"
  54 | #include "../../../common.hpp"
  55 | 
  56 | using namespace cute;
  57 | 
  58 | namespace cutlass::benchmark {
  59 | 
  60 | // Command line options parsing
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 61-105 — FMHAOptions
```cpp
  61 | struct FMHAOptions {
  62 | 
  63 |   bool error;
  64 | 
  65 |   int batch, num_heads_q, num_heads_kv, seq_len_qo, seq_len_kv, seq_len_kv_cache, head_size_qk, head_size_vo, iterations;
  66 |   float softmax_scale;
  67 |   std::string bm_name;
  68 | 
  69 |   FMHAOptions()
  70 |       : error(false), batch(32), num_heads_q(16), num_heads_kv(16), seq_len_qo(512), seq_len_kv(512), seq_len_kv_cache(512),
  71 |         head_size_qk(128), head_size_vo(128), iterations(100), softmax_scale(1.f), bm_name("Flash Attention v2") {}
  72 | 
  73 |   // Parses the command line
  74 |   void parse(int argc, char const **args) {
  75 |     cutlass::CommandLine cmd(argc, args);
  76 | 
  77 |     cmd.get_cmd_line_argument("batch", batch, 32);
  78 |     cmd.get_cmd_line_argument("num_heads_q", num_heads_q, 16);
  79 |     cmd.get_cmd_line_argument("num_heads_kv", num_heads_kv, num_heads_q);
  80 |     cmd.get_cmd_line_argument("seq_len_qo", seq_len_qo, 512);
  81 |     cmd.get_cmd_line_argument("seq_len_kv", seq_len_kv, seq_len_qo);
  82 |     cmd.get_cmd_line_argument("seq_len_kv_cache", seq_len_kv_cache, 512);
  83 |     cmd.get_cmd_line_argument("head_size_vo", head_size_vo, 128);
  84 |     cmd.get_cmd_line_argument("head_size_qk", head_size_qk, head_size_vo);
  85 |     cmd.get_cmd_line_argument("iterations", iterations, 100);
  86 |     cmd.get_cmd_line_argument("bm_name", bm_name, std::string("Flash Attention v2"));
  87 | 
  88 |     softmax_scale = 1 / std::sqrt(static_cast<float>(head_size_qk));
  89 |   }
  90 | 
  91 |   std::string benchmark_name() const {
  92 |     std::stringstream full_name;
  93 |     full_name << bm_name << "/";
  94 |     std::string const test_name_suffix = std::to_string(batch) + "x" +
  95 |                                    std::to_string(num_heads_q) + "x" +
  96 |                                    std::to_string(num_heads_kv) + "x" +
  97 |                                    std::to_string(seq_len_qo) + "x" +
  98 |                                    std::to_string(head_size_qk) + "x" +
  99 |                                    std::to_string(seq_len_kv) + "x" +
 100 |                                    std::to_string(seq_len_kv_cache) + "x" +
 101 |                                    std::to_string(head_size_vo);
 102 |     full_name << test_name_suffix;
 103 | 
 104 |     return full_name.str();
 105 |   }
```
**EN:** Extends the prefill options with `seq_len_kv_cache` and includes the cache length in the benchmark name.
**CN:** 在 prefill 选项的基础上增加 `seq_len_kv_cache`，并把缓存长度编码到基准名称中。

### Lines 110-168 — Runner state
```cpp
 110 | template <class FMHAPrefillConfiguration> struct BenchmarkRunnerFMHA {
 111 | 
 112 |   using GemmKernel = typename FMHAPrefillConfiguration::GemmKernel;
 113 |   
 114 |   using LayoutQ = typename FMHAPrefillConfiguration::LayoutQ;
 115 |   using LayoutK = typename FMHAPrefillConfiguration::LayoutK;
 116 |   using LayoutV = typename FMHAPrefillConfiguration::LayoutV;
 117 |   using LayoutO = typename FMHAPrefillConfiguration::LayoutO;
 118 | 
 119 |   using StrideQ = typename GemmKernel::StrideQ;
 120 |   using StrideK = typename GemmKernel::StrideK;
 121 |   using StrideV = typename GemmKernel::StrideV;
 122 |   using StrideO = typename GemmKernel::StrideO;
 123 | 
 124 |   using ElementQ = typename GemmKernel::ElementQ;
 125 |   using ElementK = typename GemmKernel::ElementK;
 126 |   using ElementV = typename GemmKernel::ElementV;
 127 |   using ElementAcc = typename GemmKernel::ElementAccumulator;
 128 | 
 129 |   using CollectiveEpilogue = typename GemmKernel::CollectiveEpilogue;
 130 |   using ElementOutput = typename CollectiveEpilogue::ElementOutput;
 131 |   using ElementCompute = typename CollectiveEpilogue::ElementCompute;
 132 |   using ElementAccumulator = typename CollectiveEpilogue::ElementAccumulator;
 133 | 
 134 |   using ProblemShapeType = typename GemmKernel::ProblemShape;
 135 |   static constexpr bool Causal = FMHAPrefillConfiguration::Causal;
 136 |   static constexpr bool isVarLen = FMHAPrefillConfiguration::VarLen;
 137 |   static constexpr bool isPagdKV = FMHAPrefillConfiguration::PagdKV;
 138 | 
 139 |   int32_t count;
 140 | 
 141 |   //
 142 |   // Data members
 143 |   //
 144 | 
 145 |   /// Initialization
 146 |   StrideQ stride_Q;
 147 |   StrideK stride_K;
 148 |   StrideK stride_K_cache;
 149 |   StrideV stride_V_cache;
 150 |   StrideV stride_V;
 151 |   StrideO stride_O;
 152 |   uint64_t seed = 0;
 153 | 
 154 |   std::vector<cutlass::DeviceAllocation<ElementQ>> block_Q;
 155 |   std::vector<cutlass::DeviceAllocation<ElementK>> block_K;
 156 |   std::vector<cutlass::DeviceAllocation<ElementV>> block_V;
 157 |   std::vector<cutlass::DeviceAllocation<ElementK>> block_K_cache;
 158 |   std::vector<cutlass::DeviceAllocation<ElementV>> block_V_cache;
 159 |   cutlass::DeviceAllocation<ElementOutput> block_O;
 160 |   cutlass::DeviceAllocation<ElementOutput> block_ref_O;
 161 | 
 162 |   std::vector<int> cumulative_seqlen_q;
 163 |   std::vector<int> cumulative_seqlen_kv;
 164 |   std::vector<int> cumulative_seqlen_kv_cache;
 165 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_q;
 166 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_kv;
 167 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_kv_cache;
 168 | 
```
**EN:** Stores the extra cached K/V strides, buffers, and cumulative-length arrays required by the cached-KV path.
**CN:** 保存 cached-KV 路径所需的额外 K/V 缓存步长、缓冲区与累计长度数组。

### Lines 173-395 — Reference verification
```cpp
 173 |   bool verify(ProblemShapeType problem_size, bool use_kv_cache) {
 174 |     
 175 |     if constexpr (isVarLen) {
 176 |       int max_seq_len_q = static_cast<int>(get<3>(problem_size));
 177 |       int max_seq_len_kv = static_cast<int>(get<4>(problem_size));
 178 |       int max_seq_len_kv_cache = static_cast<int>(get<5>(problem_size));
 179 |       get<3>(problem_size) = cutlass::fmha::collective::VariableLength{max_seq_len_q, cumulative_seqlen_q.data()};
 180 |       get<4>(problem_size) = cutlass::fmha::collective::VariableLength{max_seq_len_kv, cumulative_seqlen_kv.data()};
 181 |       get<5>(problem_size) = cutlass::fmha::collective::VariableLength{max_seq_len_kv_cache, cumulative_seqlen_kv_cache.data()};
 182 |     }
 183 | 
 184 |     auto [batch, num_heads_q, num_heads_kv, head_size_qk, head_size_vo] = cute::select<0,1,2,6,7>(problem_size);
 185 |     int seq_len_qo, seq_len_kv, seq_len_kv_cache;
 186 | 
 187 |     int offset_q = 0;
 188 |     int offset_k = 0;
 189 |     int offset_v = 0;
 190 |     int offset_k_cache = 0;
 191 |     int offset_v_cache = 0;
 192 |     int offset_o = 0;
 193 |     // loop over the batch dimension to compute the output
 194 |     // to avoid the risk of running out of device memory
 195 |     int q_group_size = num_heads_q / num_heads_kv;
 196 |     for (int b = 0; b < batch; b++) {
 197 |       if constexpr (isVarLen) {
 198 |         auto logical_problem_shape = cutlass::fmha::collective::apply_variable_length(problem_size, b);
 199 |         seq_len_qo = get<3>(logical_problem_shape);
 200 |         seq_len_kv = get<4>(logical_problem_shape);
 201 |         seq_len_kv_cache = get<5>(logical_problem_shape);
 202 |       } else {
 203 |         seq_len_qo = get<3>(problem_size);
 204 |         seq_len_kv = get<4>(problem_size);
 205 |         seq_len_kv_cache = get<5>(problem_size);
 206 |       }
 207 |       int seq_len_kv_total = seq_len_kv_cache + seq_len_kv;
 208 |       int kv_group_update=1;
 209 | 
 210 |       for (int h = 0; h < num_heads_q; h++) {
 211 |         cutlass::DeviceAllocation<ElementAccumulator> block_S;
 212 |         block_S.reset(seq_len_qo * seq_len_kv_total);
 213 | 
 214 |         ElementK* k_ptr;
 215 |         ElementV* v_ptr;
 216 | 
 217 |         if (use_kv_cache) {
 218 |           cutlass::DeviceAllocation<ElementK> block_K_concat(head_size_qk * seq_len_kv_total);
 219 |           cutlass::DeviceAllocation<ElementV> block_V_concat(seq_len_kv_total * head_size_vo);
 220 | 
 221 |           // Concatenate K_cache and K
 222 |           compat::memcpy<ElementK>(
 223 |               block_K_concat.get(),
 224 |               block_K_cache[0].get() + offset_k_cache,
 225 |               seq_len_kv_cache * head_size_qk
 226 |           );
 227 |           compat::memcpy<ElementK>(
 228 |               block_K_concat.get() + seq_len_kv_cache * head_size_qk,
 229 |               block_K[0].get() + offset_k,
 230 |               seq_len_kv * head_size_qk
 231 |           );
 232 | 
 233 |           // Concatenate V_cache and V
 234 |           compat::memcpy<ElementV>(
 235 |               block_V_concat.get(),
 236 |               block_V_cache[0].get() + offset_v_cache,
 237 |               seq_len_kv_cache * head_size_vo
 238 |           );
 239 |           compat::memcpy<ElementV>(
 240 |               block_V_concat.get() + seq_len_kv_cache * head_size_vo,
 241 |               block_V[0].get() + offset_v,
 242 |               seq_len_kv * head_size_vo
 243 |           );
 244 |           compat::wait();
 245 | 
 246 |           k_ptr = block_K_concat.get();
 247 |           v_ptr = block_V_concat.get();
 248 |         }
 249 |         else {
 250 |           k_ptr = block_K[0].get() + offset_k;
 251 |           v_ptr = block_V[0].get() + offset_v;
 252 |         }
 253 | 
 254 |         cutlass::TensorRef ref_Q(block_Q[0].get() + offset_q, LayoutQ::packed({seq_len_qo, head_size_qk}));
 255 |         cutlass::TensorRef ref_K(k_ptr, LayoutK::packed({head_size_qk, seq_len_kv_total}));
 256 |         cutlass::TensorRef ref_V(v_ptr, LayoutV::packed({seq_len_kv_total, head_size_vo}));
 257 |         cutlass::TensorRef ref_S(block_S.get(), LayoutQ::packed({seq_len_qo, seq_len_kv_total}));
 258 | 
 259 |         cutlass::reference::device::GemmComplex({seq_len_qo, seq_len_kv_total, head_size_qk}, ElementAccumulator{1}, ref_Q,
 260 |                                                 cutlass::ComplexTransform::kNone, ref_K, cutlass::ComplexTransform::kNone,
 261 |                                                 ElementAccumulator{0}, ref_S, ref_S, ElementAccumulator{0},
 262 |                                                 1,                   // batch_count
 263 |                                                 seq_len_qo * head_size_qk, // batch_stride_Q
 264 |                                                 seq_len_kv_total * head_size_qk, // batch_stride_K
 265 |                                                 seq_len_qo * seq_len_kv_total,   // batch_stride_S
 266 |                                                 seq_len_qo * seq_len_kv_total    // batch_stride_S
 267 |         );
 268 | 
 269 |         compat::wait();
 270 | 
 271 |         std::vector<ElementAccumulator> host_S(block_S.size());
 272 |         compat::memcpy<ElementAccumulator>(host_S.data(), block_S.get(), host_S.size());
 273 |         compat::wait();
 274 | 
 275 |         // delete this memory as it is no longer needed
 276 |         block_S.reset();
 277 |         auto offset = cute::min(seq_len_qo, seq_len_kv);
 278 |         auto discard_seq_coord = seq_len_qo - offset;
 279 |         auto full_tile_offset = seq_len_kv - offset;
 280 |         int start_col = use_kv_cache ? seq_len_kv_cache : 0;
 281 |         if (Causal) {
 282 |           // apply mask to S
 283 |           for (int row = 0; row < seq_len_qo; row++) {
 284 |             for (int col = start_col; col < seq_len_kv_total; col++) {
 285 |               if (col - full_tile_offset > row + start_col - discard_seq_coord)
 286 |                 host_S[col + row * seq_len_kv_total] = ElementAccumulator{-INFINITY};
 287 |             }
 288 |           }
 289 |         }
 290 | 
 291 |         // compute max element per row of S
 292 |         std::vector<ElementAccumulator> max_vec(seq_len_qo, ElementAccumulator{-INFINITY});
 293 |         for (int row = 0; row < seq_len_qo; row++) {
 294 |           int idx = row * seq_len_kv_total;
 295 |           int max_idx = row;
 296 |           max_vec[max_idx] = host_S[idx++];
 297 |           for (int col = 1; col < seq_len_kv_total; col++, idx++) {
 298 |             if (max_vec[max_idx] < host_S[idx])
 299 |               max_vec[max_idx] = host_S[idx];
 300 |           }
 301 |         }
 302 | 
 303 |         // compute exp of S
 304 |         for (int row = 0; row < seq_len_qo; row++) {
 305 |           int idx = row * seq_len_kv_total;
 306 |           int max_idx = row;
 307 |           for (int col = 0; col < seq_len_kv_total; col++, idx++) {
 308 |             host_S[idx] = expf((host_S[idx] - max_vec[max_idx]) / std::sqrt(static_cast<ElementAccumulator>((head_size_qk))));
 309 |           }
 310 |         }
 311 | 
 312 |         // compute sum per row of S
 313 |         std::vector<ElementAccumulator> sum_vec(seq_len_qo, ElementAccumulator{0});
 314 |         for (int row = 0; row < seq_len_qo; row++) {
 315 |           int idx = row * seq_len_kv_total;
 316 |           int sum_idx = row;
 317 |           for (int col = 0; col < seq_len_kv_total; col++, idx++) {
 318 |             sum_vec[sum_idx] += host_S[idx];
 319 |           }
 320 | 
 321 |           // scale each row with the sum to compute softmax
 322 |           idx = row * seq_len_kv_total;
 323 |           sum_idx = row;
 324 |           for (int col = 0; col < seq_len_kv_total; col++, idx++) {
 325 |             if(Causal && row < discard_seq_coord) {
 326 |               host_S[idx] = 0;
 327 |             } else {
 328 |               host_S[idx] /= sum_vec[sum_idx];
 329 |             }
 330 |           }
 331 |         }
 332 | 
 333 |         std::vector<ElementV> host_P(host_S.size());
 334 |         for (int p = 0; p < host_P.size(); p++)
 335 |           host_P[p] = static_cast<ElementV>(host_S[p]);
 336 | 
 337 |         cutlass::DeviceAllocation<ElementV> block_P;
 338 |         block_P.reset(host_P.size());
 339 | 
 340 |         compat::memcpy<ElementV>(block_P.get(), host_P.data(), host_P.size());
 341 |         compat::wait();
 342 | 
 343 |         cutlass::TensorRef ref_P(block_P.get(), LayoutQ::packed({seq_len_qo, seq_len_kv_total}));
 344 | 
 345 |         cutlass::DeviceAllocation<ElementAccumulator> block_acc;
 346 |         block_acc.reset(seq_len_qo * head_size_vo);
 347 |         cutlass::TensorRef ref_acc(block_acc.get(), LayoutO::packed({seq_len_qo, head_size_vo}));
 348 | 
 349 |         cutlass::reference::device::GemmComplex({seq_len_qo, head_size_vo, seq_len_kv_total}, ElementAccumulator{1}, ref_P,
 350 |                                                 cutlass::ComplexTransform::kNone, ref_V, cutlass::ComplexTransform::kNone,
 351 |                                                 ElementAccumulator{0}, ref_acc, ref_acc, ElementAccumulator{0},
 352 |                                                 1,                   // batch_count
 353 |                                                 seq_len_qo * seq_len_kv_total,   // batch_stride_P
 354 |                                                 seq_len_kv_total * head_size_vo, // batch_stride_V
 355 |                                                 seq_len_qo * head_size_vo, // batch_stride_O
 356 |                                                 seq_len_qo * head_size_vo  // batch_stride_O
 357 |         );
 358 | 
 359 |         compat::wait();
 360 |         // delete this memory as it is no longer needed
 361 |         block_P.reset();
 362 | 
 363 |         std::vector<ElementAccumulator> vec_acc(block_acc.size());
 364 |         compat::memcpy<ElementAccumulator>(vec_acc.data(), block_acc.get(), vec_acc.size());
 365 |         compat::wait();
 366 | 
 367 |         // delete this memory as it is no longer needed
 368 |         block_acc.reset();
 369 |         std::vector<ElementOutput> vec_out(vec_acc.size());
 370 |         for(int i = 0; i < vec_out.size(); i++) {
 371 |           vec_out[i] = static_cast<ElementOutput>(vec_acc[i]);
 372 |         }
 373 |         compat::memcpy<ElementOutput>(block_ref_O.get() + offset_o, vec_out.data(), vec_out.size());
 374 |         compat::wait();
 375 | 
 376 |         offset_q += seq_len_qo * head_size_qk;
 377 |         if(kv_group_update % q_group_size==0) {
 378 |           offset_k += seq_len_kv * head_size_qk;
 379 |           offset_v += seq_len_kv * head_size_vo;
 380 |           offset_k_cache += seq_len_kv_cache * head_size_qk;
 381 |           offset_v_cache += seq_len_kv_cache * head_size_vo;
 382 |         }
 383 |         kv_group_update++;
 384 |         offset_o += seq_len_qo * head_size_vo;
 385 |       }
 386 |     }
 387 | 
 388 |     compat::wait();
 389 | 
 390 |     // Check if output from CUTLASS kernel and reference kernel are equal or not
 391 |     bool passed = cutlass::reference::device::BlockCompareRelativelyEqual(block_ref_O.get(), block_O.get(),
 392 |                                                                           block_O.size(), ElementOutput{0.5}, ElementOutput{0.5});
 393 | 
 394 |     return passed;
 395 |   }
```
**EN:** Optionally concatenates cached K/V tensors with the current tensors, computes the reference attention result, and checks it against the kernel output.
**CN:** 按需把缓存 K/V 张量与当前张量拼接起来，计算参考注意力结果，并与内核输出对比。

### Lines 397-470 — Variable-length cache setup
```cpp
 397 |   template<class ProblemShape>
 398 |   auto initialize_varlen(const ProblemShape& problem_size) {
 399 |     int num_batches = get<0>(problem_size);
 400 | 
 401 |     // generate Q as --b times
 402 |     //    gaussian (--Q, --Q / 2) sampled positive
 403 |     //    track cumulative
 404 |     std::mt19937 rng(0x202305151552ull);
 405 |     std::normal_distribution<double> dist_q(get<3>(problem_size), get<3>(problem_size) / 2);
 406 |     std::normal_distribution<double> dist_kv(get<4>(problem_size), get<4>(problem_size) / 2);
 407 |     std::normal_distribution<double> dist_kv_cache(get<5>(problem_size), get<5>(problem_size) / 2);
 408 | 
 409 |     // Use Cacheline Size to calculate alignment
 410 |     constexpr int cacheline_bytes = 64;
 411 |     constexpr int AlignmentQ = cacheline_bytes / sizeof(ElementQ);    // Alignment of Q matrix in units of elements
 412 |     constexpr int AlignmentKV = cacheline_bytes / sizeof(ElementK);   // Alignment of Kand V matrix in units of elements
 413 | 
 414 |     auto generate_positive_int = [](auto& dist, auto& gen) {
 415 |       int result = 0;
 416 |       do {
 417 |         result = static_cast<int>(dist(gen));
 418 |       } while (result <= 0);
 419 |       return result;
 420 |     };
 421 | 
 422 |     cumulative_seqlen_q = {0};
 423 |     cumulative_seqlen_kv = {0};
 424 |     cumulative_seqlen_kv_cache = {0};
 425 | 
 426 |     int total_seqlen_q = 0;
 427 |     int total_seqlen_kv = 0;
 428 |     int total_seqlen_kv_cache = 0;
 429 |     int max_seqlen_q = 0;
 430 |     int max_seqlen_kv = 0;
 431 |     int max_seqlen_kv_cache = 0;
 432 | 
 433 |     for (int i = 0; i < num_batches; i++) {
 434 |       int seqlen_q = cutlass::round_up(generate_positive_int(dist_q, rng), AlignmentQ);
 435 |       int seqlen_kv = cutlass::round_up(generate_positive_int(dist_kv, rng), AlignmentKV);
 436 |       int seqlen_kv_cache = cute::get<5>(problem_size) == 0 ? 0 : cutlass::round_up(generate_positive_int(dist_kv_cache, rng), AlignmentKV);
 437 | 
 438 |       total_seqlen_q += seqlen_q;
 439 |       total_seqlen_kv += seqlen_kv;
 440 |       total_seqlen_kv_cache += seqlen_kv_cache;
 441 | 
 442 |       max_seqlen_q = std::max(max_seqlen_q, seqlen_q);
 443 |       max_seqlen_kv = std::max(max_seqlen_kv, seqlen_kv);
 444 |       max_seqlen_kv_cache = std::max(max_seqlen_kv_cache, seqlen_kv_cache);
 445 | 
 446 |       cumulative_seqlen_q.push_back(cumulative_seqlen_q.back() + seqlen_q);
 447 |       cumulative_seqlen_kv.push_back(cumulative_seqlen_kv.back() + seqlen_kv);
 448 |       cumulative_seqlen_kv_cache.push_back(cumulative_seqlen_kv_cache.back() + seqlen_kv_cache);
 449 |     }
 450 | 
 451 |     ProblemShape problem_size_for_init = problem_size;
 452 |     get<0>(problem_size_for_init) = 1;
 453 |     get<3>(problem_size_for_init) = total_seqlen_q;
 454 |     get<4>(problem_size_for_init) = total_seqlen_kv;
 455 |     get<5>(problem_size_for_init) = total_seqlen_kv_cache;
 456 | 
 457 |     ProblemShapeType problem_size_for_launch;
 458 | 
 459 |     get<3>(problem_size_for_launch) = cutlass::fmha::collective::VariableLength{max_seqlen_q};
 460 |     get<4>(problem_size_for_launch) = cutlass::fmha::collective::VariableLength{max_seqlen_kv};
 461 |     get<5>(problem_size_for_launch) = cutlass::fmha::collective::VariableLength{max_seqlen_kv_cache};
 462 |     get<6>(problem_size_for_launch) = get<6>(problem_size);
 463 |     get<7>(problem_size_for_launch) = get<7>(problem_size);
 464 |     get<0>(problem_size_for_launch) = get<0>(problem_size);
 465 |     get<1>(problem_size_for_launch) = get<1>(problem_size);
 466 |     get<2>(problem_size_for_launch) = get<2>(problem_size);
 467 | 
 468 | 
 469 |     return cute::make_tuple(problem_size_for_init, problem_size_for_launch);
 470 |   }
```
**EN:** Generates random variable lengths for Q, current KV, and cached KV, then produces allocation and launch shapes.
**CN:** 为 Q、当前 KV 和缓存 KV 生成随机变长信息，然后生成分配和启动所需的形状。

### Lines 472-560 — Allocation and initialization
```cpp
 472 |   /// Initialize operands to be used in the GEMM and reference GEMM
 473 |   ProblemShapeType initialize(const FMHAOptions &options) {
 474 |     auto problem_shape_in =
 475 |         cute::make_tuple(options.batch, options.num_heads_q, options.num_heads_kv, options.seq_len_qo, options.seq_len_kv, options.seq_len_kv_cache, options.head_size_qk, options.head_size_vo);
 476 | 
 477 |     ProblemShapeType problem_shape;
 478 |     decltype(problem_shape_in) problem_size;
 479 | 
 480 |     if constexpr (isVarLen) {
 481 |       auto [problem_shape_init, problem_shape_launch] = initialize_varlen(problem_shape_in);
 482 |       problem_shape = problem_shape_launch;
 483 |       problem_size = problem_shape_init;
 484 |     }
 485 |     else {
 486 |       problem_size = problem_shape_in;
 487 |       problem_shape = problem_shape_in;
 488 |     }
 489 | 
 490 |     auto [batch, num_heads_q, num_heads_kv, seq_len_qo, seq_len_kv, seq_len_kv_cache, head_size_qk, head_size_vo] = problem_size;
 491 | 
 492 |     stride_Q = cutlass::make_cute_packed_stride(StrideQ{}, cute::make_shape(seq_len_qo, head_size_qk, batch * num_heads_q));
 493 |     stride_K = cutlass::make_cute_packed_stride(StrideK{}, cute::make_shape(seq_len_kv, head_size_qk, batch * num_heads_kv));
 494 |     stride_V = cutlass::make_cute_packed_stride(StrideV{}, cute::make_shape(head_size_vo, seq_len_kv, batch * num_heads_kv));
 495 |     stride_K_cache = cutlass::make_cute_packed_stride(StrideK{}, cute::make_shape(seq_len_kv_cache, head_size_qk, batch * num_heads_kv));
 496 |     stride_V_cache = cutlass::make_cute_packed_stride(StrideV{}, cute::make_shape(head_size_vo, seq_len_kv_cache, batch * num_heads_kv));
 497 |     stride_O = cutlass::make_cute_packed_stride(StrideO{}, cute::make_shape(seq_len_qo, head_size_vo, batch * num_heads_q));
 498 | 
 499 |     std::size_t mem_size_q = static_cast<std::size_t>(batch) * num_heads_q * seq_len_qo * head_size_qk;
 500 |     std::size_t mem_size_k = static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv * head_size_qk;
 501 |     std::size_t mem_size_v = static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv * head_size_vo;
 502 |     std::size_t mem_size_k_cache = static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv_cache * head_size_qk;
 503 |     std::size_t mem_size_v_cache = static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv_cache * head_size_vo;
 504 |     std::size_t mem_size_o = static_cast<std::size_t>(batch) * num_heads_q * seq_len_qo * head_size_vo;
 505 | 
 506 |     std::size_t mem_occupied_QKV = (mem_size_q * sizeof(ElementQ)) + (mem_size_k * sizeof(ElementK)) +
 507 |                                    (mem_size_v * sizeof(ElementV)) + (mem_size_k_cache * sizeof(ElementK)) +
 508 |                                    (mem_size_v_cache * sizeof(ElementV));
 509 | 
 510 |     count = std::ceil(static_cast<float>(cutlass::get_llc_size()) / static_cast<float>(mem_occupied_QKV)) + 1;
 511 | 
 512 |     for(int i = 0; i < count; i++) {
 513 |       block_Q.emplace_back();
 514 |       block_K.emplace_back();
 515 |       block_V.emplace_back();
 516 |       block_K_cache.emplace_back();
 517 |       block_V_cache.emplace_back();
 518 |     }
 519 | 
 520 | 
 521 |     for(int i = 0; i < count; i++) {
 522 |       block_Q[i].reset(mem_size_q);
 523 |       block_K[i].reset(mem_size_k);
 524 |       block_V[i].reset(mem_size_v);
 525 |       block_K_cache[i].reset(mem_size_k_cache);
 526 |       block_V_cache[i].reset(mem_size_v_cache);
 527 | 
 528 |       initialize_block(block_Q[i], seed + i + 2021);
 529 |       initialize_block(block_K[i], seed + i + 2022);
 530 |       initialize_block(block_V[i], seed + i + 2023);
 531 |       initialize_block(block_K_cache[i], seed + i + 2024);
 532 |       initialize_block(block_V_cache[i], seed + i + 2025);
 533 |     }
 534 | 
 535 |     block_O.reset(mem_size_o);
 536 |     block_ref_O.reset(mem_size_o);
 537 | 
 538 |     if (!cumulative_seqlen_q.empty()) {
 539 |       device_cumulative_seqlen_q.reset(cumulative_seqlen_q.size());
 540 |       device_cumulative_seqlen_q.copy_from_host(
 541 |         cumulative_seqlen_q.data(), cumulative_seqlen_q.size());
 542 |     }
 543 |     if (!cumulative_seqlen_kv.empty()) {
 544 |       device_cumulative_seqlen_kv.reset(cumulative_seqlen_kv.size());
 545 |       device_cumulative_seqlen_kv.copy_from_host(
 546 |         cumulative_seqlen_kv.data(), cumulative_seqlen_kv.size());
 547 |     }
 548 |     if (!cumulative_seqlen_kv_cache.empty()) {
 549 |       device_cumulative_seqlen_kv_cache.reset(cumulative_seqlen_kv_cache.size());
 550 |       device_cumulative_seqlen_kv_cache.copy_from_host(
 551 |         cumulative_seqlen_kv_cache.data(), cumulative_seqlen_kv_cache.size());
 552 |     }
 553 | 
 554 |     if constexpr (isVarLen) {
 555 |       get<3>(problem_shape).cumulative_length = device_cumulative_seqlen_q.get();
 556 |       get<4>(problem_shape).cumulative_length = device_cumulative_seqlen_kv.get();
 557 |       get<5>(problem_shape).cumulative_length = device_cumulative_seqlen_kv_cache.get();
 558 |     }
 559 | 
 560 |     return problem_shape;
```
**EN:** Allocates Q/K/V/cache/output tensors, initializes them, and uploads cumulative-length arrays for the cache-aware kernel path.
**CN:** 为支持缓存感知内核路径分配并初始化 Q/K/V/缓存/输出张量，并上传累计长度数组。

### Lines 563-602 — Launch wrapper
```cpp
 563 |   static void run(typename GemmKernel::Params params) {
 564 |     dim3 const block = GemmKernel::get_block_shape();
 565 |     dim3 const grid = GemmKernel::get_grid_shape(params);
 566 | 
 567 |     // configure smem size and carveout
 568 |     int smem_size = GemmKernel::SharedStorageSize;
 569 | 
 570 |     const auto sycl_block = compat::dim3(block.x, block.y, block.z);
 571 |     const auto sycl_grid = compat::dim3(grid.x, grid.y, grid.z);
 572 | 
 573 | #if !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)
 574 |     using namespace compat::experimental;
 575 |     #if defined(CUTLASS_SYCL_PROFILING_ENABLED)
 576 |     auto event = launch<cutlass::device_kernel<GemmKernel>>(
 577 |         launch_policy{sycl_grid, sycl_block, local_mem_size{static_cast<std::size_t>(smem_size)},
 578 |                       kernel_properties{sycl_exp::sub_group_size<GemmKernel::DispatchPolicy::SubgroupSize>}},
 579 |         params);
 580 |     EventManager::getInstance().addEvent(event);
 581 |     #else
 582 |     launch<cutlass::device_kernel<GemmKernel>, sycl::detail::auto_name, false>(
 583 |         launch_policy{sycl_grid, sycl_block, local_mem_size{static_cast<std::size_t>(smem_size)},
 584 |                       kernel_properties{sycl_exp::sub_group_size<GemmKernel::DispatchPolicy::SubgroupSize>}},
 585 |         params);
 586 |     #endif
 587 | #else
 588 |     compat::experimental::launch_properties launch_props{
 589 |       sycl::ext::oneapi::experimental::work_group_scratch_size(smem_size)
 590 |     };
 591 |     compat::experimental::kernel_properties kernel_props{
 592 |       sycl::ext::oneapi::experimental::sub_group_size<GemmKernel::DispatchPolicy::SubgroupSize>
 593 |     };
 594 |     compat::experimental::launch_policy policy{sycl_grid, sycl_block, launch_props, kernel_props};
 595 |     #if defined(CUTLASS_SYCL_PROFILING_ENABLED)
 596 |     auto event = compat::experimental::launch<cutlass::device_kernel<GemmKernel>, GemmKernel>(policy, params);
 597 |     EventManager::getInstance().addEvent(event);
 598 |     #else
 599 |         compat::experimental::launch<cutlass::device_kernel<GemmKernel>, GemmKernel, false>(policy, params);
 600 |     #endif
 601 | #endif
 602 |   }
```
**EN:** Encapsulates the SYCL launch policy for the cached-KV kernel, including scratch-memory and profiling setup.
**CN:** 封装 cached-KV 内核的 SYCL 启动策略，包括 scratch memory 和 profiling 设置。

### Lines 604-744 — Benchmark execution
```cpp
 604 |   void run(::benchmark::State& state, const FMHAOptions &options, const cutlass::KernelHardwareInfo &hw_info) {
 605 | 
 606 |     ProblemShapeType problem_size = initialize(options);
 607 | 
 608 |     typename GemmKernel::Arguments arguments{
 609 |         cutlass::gemm::GemmUniversalMode::kGemm,
 610 |         problem_size,
 611 |         {
 612 |           block_Q[0].get(), stride_Q,
 613 |           block_K[0].get(), stride_K,
 614 |           block_V[0].get(), stride_V,
 615 |           block_K_cache[0].get(), stride_K_cache,
 616 |           block_V_cache[0].get(), stride_V_cache,
 617 |           //TODO:: the following 3 parameters need to be parametrised when the benchmark for paged KV has been added.
 618 |           // page table
 619 |           nullptr,
 620 |           //page size
 621 |           0,
 622 |           // num pages per seq lengh
 623 |           0
 624 |         },
 625 |         {options.softmax_scale},
 626 |         {block_O.get(), stride_O},
 627 |         hw_info};
 628 | 
 629 |     // GemmKernel gemm_op;
 630 | 
 631 |     size_t workspace_size = GemmKernel::get_workspace_size(arguments);
 632 |     cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
 633 | 
 634 |     GemmKernel::can_implement(arguments);
 635 | 
 636 |     // Initialize the workspace
 637 |     auto status = GemmKernel::initialize_workspace(arguments, workspace.get());
 638 |     if (status != cutlass::Status::kSuccess) {
 639 |       return;
 640 |     }
 641 | 
 642 |     typename GemmKernel::Params params = GemmKernel::to_underlying_arguments(arguments, workspace.get());
 643 | 
 644 |     // Run the GEMM
 645 |     run(params);
 646 | 
 647 |     compat::wait();
 648 | 
 649 |     // Verify that the result is correct
 650 |     bool use_kv_cache = options.seq_len_kv_cache > 0;
 651 |     bool passed = verify(problem_size, use_kv_cache);
 652 |     if(not passed) {
 653 |       state.SkipWithError("Disposition Failed.");
 654 |     }
 655 | 
 656 |     state.counters["batch"] = options.batch;
 657 |     state.counters["num_heads_q"] = options.num_heads_q;
 658 |     state.counters["num_heads_kv"] = options.num_heads_kv;
 659 |     state.counters["seq_len_qo"] = options.seq_len_qo;
 660 |     state.counters["seq_len_kv"] = options.seq_len_kv;
 661 |     state.counters["seq_len_kv_cache"] = options.seq_len_kv_cache;
 662 |     state.counters["head_size_kv"] = options.head_size_qk;
 663 |     state.counters["head_size_vo"] = options.head_size_vo;
 664 |     state.counters["scale"] = options.softmax_scale;
 665 |     state.counters["causal"] = Causal;
 666 |     state.counters["varlen"] = isVarLen;
 667 | 
 668 |     std::stringstream extra_label;
 669 |     extra_label << "layoutQ=RowMajor ";
 670 |     extra_label << "layoutK=ColumnMajor ";
 671 |     extra_label << "layoutV=RowMajor ";
 672 | 
 673 |     state.SetLabel(extra_label.str());
 674 |     // when seq_len_qo is not equal to seq_len_kv we use bottom up approach for the masking.
 675 |     // Following changes will adjust the effective_seq_len_kv when masking applied for such cases.
 676 |     auto offset = cute::min(options.seq_len_qo, options.seq_len_kv);
 677 |     auto discard_seq_coord = options.seq_len_qo - offset;
 678 |     auto full_tile_offset = options.seq_len_kv - offset;
 679 |     auto effective_seq_len_kv = options.seq_len_kv_cache + (Causal ? full_tile_offset + ((offset + 1) / 2.0): options.seq_len_kv);
 680 |     auto effective_seq_len_qo = Causal ? options.seq_len_qo - discard_seq_coord  : options.seq_len_qo;
 681 | 
 682 |     double flops_qk = 2.0 * options.batch * options.num_heads_q * effective_seq_len_qo * effective_seq_len_kv * options.head_size_qk;
 683 |     double flops_pv = 2.0 * options.batch * options.num_heads_q * effective_seq_len_qo * options.head_size_vo * effective_seq_len_kv;
 684 |     double gflops = (flops_qk + flops_pv) * 1e-9;
 685 | 
 686 |     // TODO: Use sizeof_bits_v instead of sizeof if QKVO is smaller than 8 bits, which avoids incorrect bandwidth calculation
 687 |     double gbps_qk =  options.batch * (sizeof(ElementQ) * options.num_heads_q * effective_seq_len_qo * options.head_size_qk +
 688 |                       sizeof(ElementK) * options.num_heads_kv * effective_seq_len_kv * options.head_size_qk);
 689 |     double gbps_pv = sizeof(ElementV) * options.batch * options.num_heads_kv * effective_seq_len_kv * options.head_size_vo +
 690 |                      sizeof(ElementOutput) * options.batch * options.num_heads_q * effective_seq_len_qo * options.head_size_vo;
 691 |     double mega_bytes_transferred = (gbps_qk + gbps_pv) * (1e-6);
 692 | 
 693 |     initialize_counters(state);
 694 |     int32_t counter = 1;
 695 |     for(auto _ : state) {
 696 |       state.PauseTiming();
 697 |       int input_num = std::max(int(0), counter % count);
 698 | 
 699 |       typename GemmKernel::Arguments arguments{
 700 |           cutlass::gemm::GemmUniversalMode::kGemm,
 701 |           problem_size,
 702 |           {
 703 |             block_Q[input_num].get(), stride_Q,
 704 |             block_K[input_num].get(), stride_K,
 705 |             block_V[input_num].get(), stride_V,
 706 |             block_K_cache[input_num].get(), stride_K_cache,
 707 |             block_V_cache[input_num].get(), stride_V_cache,
 708 |             //TODO:: the following 3 parameters need to be parametrised when the benchmark for paged KV has been added.
 709 |             // page table
 710 |             nullptr,
 711 |             //page size
 712 |             0,
 713 |             // num pages per seq lengh
 714 |             0
 715 |           },
 716 |           {options.softmax_scale},
 717 |           {block_O.get(), stride_O},
 718 |           hw_info};
 719 | 
 720 |       size_t workspace_size = GemmKernel::get_workspace_size(arguments);
 721 |       cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
 722 | 
 723 |       GemmKernel::can_implement(arguments);
 724 | 
 725 |       // Initialize the workspace
 726 |       auto status = GemmKernel::initialize_workspace(arguments, workspace.get());
 727 |       if (status != cutlass::Status::kSuccess) {
 728 |         return;
 729 |       }
 730 | 
 731 |       typename GemmKernel::Params params = GemmKernel::to_underlying_arguments(arguments, workspace.get());
 732 | 
 733 |       state.ResumeTiming();
 734 | 
 735 |       GPU_Clock timer;
 736 |       timer.start();
 737 |       run(params);
 738 |       auto ms_elapsed = timer.milliseconds();
 739 |       update_counters(state, ms_elapsed);
 740 |       state.SetIterationTime(ms_elapsed / 1000);
 741 |       counter++;
 742 |     }
 743 |     finalize_counters(state, gflops, mega_bytes_transferred);
 744 |   }
```
**EN:** Builds arguments with cache tensors, runs validation, computes metrics using the total KV length, and executes the timed loop.
**CN:** 构建包含缓存张量的参数，执行结果校验，按总 KV 长度计算性能指标，并运行计时循环。

### Lines 746-766 — Counter helpers
```cpp
 746 | private:
 747 |   static void initialize_counters(::benchmark::State& state) {
 748 |     state.counters["avg_runtime_ms"] = 0;
 749 |     state.counters["best_runtime_ms"] = std::numeric_limits<double>::max();
 750 |   }
 751 | 
 752 |   static void update_counters(::benchmark::State& state, double ms_elapsed) {
 753 |     state.PauseTiming();
 754 |     state.counters["total_runtime_ms"] += ms_elapsed;
 755 |     state.counters["best_runtime_ms"] = std::min<double>(state.counters["best_runtime_ms"], ms_elapsed);
 756 |     state.ResumeTiming();
 757 |   }
 758 | 
 759 |   static void finalize_counters(::benchmark::State& state,  double gflop, double mega_bytes_transferred) {
 760 |     state.counters["avg_runtime_ms"] =
 761 |       state.counters["total_runtime_ms"] / static_cast<double>(state.iterations());
 762 |     state.counters["avg_tflops"] = gflop / state.counters["avg_runtime_ms"];
 763 |     state.counters["avg_throughput"] = mega_bytes_transferred / state.counters["avg_runtime_ms"];
 764 |     state.counters["best_tflop"] = gflop / state.counters["best_runtime_ms"];
 765 |     state.counters["best_bandwidth"] = mega_bytes_transferred / state.counters["best_runtime_ms"];
 766 |   }
```
**EN:** Maintains runtime aggregates used to derive average latency, throughput, and best-case metrics.
**CN:** 维护运行时间聚合值，用于推导平均时延、吞吐量与最佳性能指标。

### Lines 771-780 — Registration macros
```cpp
 771 | #define CUTLASS_FMHA_PREFILL_BENCHMARK(F) cutlass::benchmark::BenchmarkRegistry<cutlass::benchmark::FMHAOptions>::Register(#F, &F##_func)
 772 | 
 773 | #define CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(F)                          \
 774 |   static void F##_func(                                           \
 775 |       ::benchmark::State& state,                                  \
 776 |       cutlass::benchmark::FMHAOptions const& options,                 \
 777 |       cutlass::KernelHardwareInfo const& hw_info) {               \
 778 |     auto bench = cutlass::benchmark::BenchmarkRunnerFMHA<F>();    \
 779 |     bench.run(state, options, hw_info);                           \
 780 |   }
```
**EN:** Wraps cached-KV configurations into benchmark-callable functions and registry entries.
**CN:** 把 cached-KV 配置包装成可由基准框架调用的函数与注册项。

## Key Concepts / 关键概念

- Cached-KV benchmark options / cached-KV 基准选项
- Reference concatenation of cached and live KV / 缓存与实时 KV 的参考拼接
- Variable-length cache metadata / 变长缓存元数据
- Timed execution and metrics / 计时执行与性能指标

## Dependencies / 依赖关系

- Legacy cached-KV kernels and scheduler / 旧版 cached-KV 内核与调度器
- `cutlass::DeviceAllocation` — device buffers / 设备缓冲区
- `BenchmarkRegistry<FMHAOptions>` — benchmark registry / 基准注册表
