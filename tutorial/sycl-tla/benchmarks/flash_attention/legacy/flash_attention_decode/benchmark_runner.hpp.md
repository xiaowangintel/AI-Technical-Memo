# benchmark_runner.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/legacy/flash_attention_decode/benchmark_runner.hpp`
- **EN:** Legacy Flash Attention decode harness that parses options, allocates inputs, verifies outputs against a reference implementation, and records benchmark counters.
- **CN:** 旧版 Flash Attention decode 运行器：负责解析选项、分配输入、用参考实现校验输出，并记录基准计数器。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
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
  36 | #include "flash_attention_v2/kernel/legacy/tile_scheduler.hpp"
  37 | #include "flash_attention_v2/collective/fmha_fusion.hpp"
  38 | #include "cutlass/gemm/device/gemm_universal_adapter.h"
  39 | #include "cutlass/util/packed_stride.hpp"
  40 | #include "flash_attention_v2/kernel/legacy/xe_flash_attn_decode.hpp"
  41 | #include "flash_attention_v2/collective/legacy/xe_flash_attn_decode_epilogue.hpp"
  42 | #include "flash_attention_v2/collective/legacy/xe_flash_attn_decode_softmax_epilogue.hpp"
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

### Lines 61-113 — FMHADecodeOptions
```cpp
  61 | struct FMHADecodeOptions {
  62 | 
  63 |   bool error;
  64 | 
  65 |   int batch, num_heads_q, num_heads_kv, seq_len_qo, seq_len_kv, seq_len_kv_cache, head_size_qk,
  66 |       head_size_vo, iterations, page_size;
  67 |   float softmax_scale;
  68 |   std::string bm_name;
  69 | 
  70 |   FMHADecodeOptions()
  71 |       : error(false), batch(32), num_heads_q(16), num_heads_kv(16), seq_len_qo(1), head_size_qk(128),
  72 |         seq_len_kv(512), seq_len_kv_cache(0), page_size(128), head_size_vo(128), iterations(100), softmax_scale(1.f), bm_name("Flash Attention v2 Decode") {}
  73 | 
  74 |   // Parses the command line
  75 |   void parse(int argc, char const **args) {
  76 |     cutlass::CommandLine cmd(argc, args);
  77 | 
  78 |     cmd.get_cmd_line_argument("batch", batch, 32);
  79 |     cmd.get_cmd_line_argument("num_heads_q", num_heads_q, 16);
  80 |     cmd.get_cmd_line_argument("num_heads_kv", num_heads_kv, num_heads_q);
  81 |     cmd.get_cmd_line_argument("seq_len_qo", seq_len_qo, 1);
  82 |     cmd.get_cmd_line_argument("seq_len_kv", seq_len_kv, seq_len_qo);
  83 |     cmd.get_cmd_line_argument("seq_len_kv_cache", seq_len_kv_cache, 0);
  84 |     cmd.get_cmd_line_argument("page_size", page_size, 128);
  85 |     cmd.get_cmd_line_argument("head_size_vo", head_size_vo, 128);
  86 |     cmd.get_cmd_line_argument("head_size_qk", head_size_qk, head_size_vo);
  87 |     cmd.get_cmd_line_argument("iterations", iterations, 100);
  88 |     cmd.get_cmd_line_argument("bm_name", bm_name, std::string("Flash Attention v2"));
  89 | 
  90 |     softmax_scale = 1 / std::sqrt(static_cast<float>(head_size_qk));
  91 | 
  92 |     if (seq_len_kv_cache % page_size != 0) {
  93 |       std::cerr << "Invalid: seq_len_kv_cache must be divisible by page_size" << std::endl;
  94 |       return;
  95 |     }
  96 |   }
  97 | 
  98 |   std::string benchmark_name() const {
  99 |     std::stringstream full_name;
 100 |     full_name << bm_name << "/";
 101 |     std::string const test_name_suffix = std::to_string(batch) + "x" +
 102 |                                    std::to_string(num_heads_q) + "x" +
 103 |                                    std::to_string(num_heads_kv) + "x" +
 104 |                                    std::to_string(seq_len_qo) + "x" +
 105 |                                    std::to_string(head_size_qk) + "x" +
 106 |                                    std::to_string(seq_len_kv) + "x" +
 107 |                                    std::to_string(seq_len_kv_cache) + "x" +
 108 |                                    std::to_string(head_size_vo);
 109 |     full_name << test_name_suffix;
 110 | 
 111 |     return full_name.str();
 112 |   }
 113 | };
```
**EN:** Parses decode-specific sizes and flags, computes the softmax scale, and formats a descriptive benchmark name.
**CN:** 解析 decode 专用的尺寸与标志，计算 softmax 缩放，并格式化可读的基准名称。

### Lines 117-183 — Runner members
```cpp
 117 | template <class FMHADecodeConfiguration> struct BenchmarkRunnerFMHADecode {
 118 | 
 119 |   using FMHADecodeKernel = typename FMHADecodeConfiguration::FMHADecodeKernel;
 120 |   
 121 |   using LayoutQ = typename FMHADecodeConfiguration::LayoutQ;
 122 |   using LayoutK = typename FMHADecodeConfiguration::LayoutK;
 123 |   using LayoutV = typename FMHADecodeConfiguration::LayoutV;
 124 |   using LayoutO = typename FMHADecodeConfiguration::LayoutO;
 125 | 
 126 |   using StrideQ = typename FMHADecodeKernel::StrideQ;
 127 |   using StrideK = typename FMHADecodeKernel::StrideK;
 128 |   using StrideV = typename FMHADecodeKernel::StrideV;
 129 |   using StrideO = typename FMHADecodeKernel::StrideO;
 130 | 
 131 |   using ElementQ = typename FMHADecodeKernel::ElementQ;
 132 |   using ElementK = typename FMHADecodeKernel::ElementK;
 133 |   using ElementV = typename FMHADecodeKernel::ElementV;
 134 |   using ElementAcc = typename FMHADecodeKernel::ElementAccumulator;
 135 | 
 136 |   using CollectiveEpilogue = typename FMHADecodeKernel::CollectiveEpilogue;
 137 |   using ElementOutput = typename CollectiveEpilogue::ElementOutput;
 138 |   using ElementCompute = typename CollectiveEpilogue::ElementCompute;
 139 |   using ElementAccumulator = typename CollectiveEpilogue::ElementAccumulator;
 140 | 
 141 |   using ProblemShapeType = typename FMHADecodeKernel::ProblemShape;
 142 |   static constexpr bool Causal = FMHADecodeConfiguration::Causal;
 143 |   static constexpr bool isVarLen = FMHADecodeConfiguration::VarLen;
 144 |   static constexpr bool PagedKV = FMHADecodeConfiguration::PagedKV;
 145 | 
 146 |   int32_t count;
 147 | 
 148 |   //
 149 |   // Data members
 150 |   //
 151 | 
 152 |   /// Initialization
 153 |   StrideQ stride_Q;
 154 |   StrideK stride_K;
 155 |   StrideV stride_V;
 156 |   StrideO stride_O;
 157 |   StrideK stride_K_cache;
 158 |   StrideV stride_V_cache;
 159 | 
 160 |   uint64_t seed = 0;
 161 | 
 162 |   std::vector<cutlass::DeviceAllocation<ElementQ>> block_Q;
 163 |   std::vector<cutlass::DeviceAllocation<ElementK>> block_K;
 164 |   std::vector<cutlass::DeviceAllocation<ElementV>> block_V;
 165 |   std::vector<cutlass::DeviceAllocation<ElementK>> block_K_cache;
 166 |   std::vector<cutlass::DeviceAllocation<ElementV>> block_V_cache;
 167 |   cutlass::DeviceAllocation<ElementOutput> block_O;
 168 |   cutlass::DeviceAllocation<ElementOutput> block_ref_O;
 169 | 
 170 |   std::vector<int> cumulative_seqlen_q;
 171 |   std::vector<int> cumulative_seqlen_kv;
 172 |   std::vector<int> cumulative_seqlen_kv_cache;
 173 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_q;
 174 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_kv;
 175 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_kv_cache;
 176 | 
 177 |   struct PagedKVParams {
 178 |       cutlass::DeviceAllocation<int> page_table;
 179 |       int page_size = 0;
 180 |       cutlass::DeviceAllocation<int> num_pages_per_seq;
 181 |   };
 182 |   PagedKVParams paged_kv_cache;
 183 | 
```
**EN:** Defines the templated decode runner and stores kernel-derived types, strides, buffers, and paged-KV metadata.
**CN:** 定义模板化 decode 运行器，并保存内核推导出的类型、步长、缓冲区和分页 KV 元数据。

### Lines 188-409 — Reference verification
```cpp
 188 |   bool verify(ProblemShapeType problem_size) {
 189 |     
 190 |     if constexpr (isVarLen) {
 191 |       int max_seq_len_q = static_cast<int>(get<3>(problem_size));
 192 |       int max_seq_len_kv = static_cast<int>(get<4>(problem_size));
 193 |       int max_seq_len_kv_cache = static_cast<int>(get<5>(problem_size));
 194 |       get<3>(problem_size) = cutlass::fmha::collective::VariableLength{max_seq_len_q, cumulative_seqlen_q.data()};
 195 |       get<4>(problem_size) = cutlass::fmha::collective::VariableLength{max_seq_len_kv, cumulative_seqlen_kv.data()};
 196 |       get<5>(problem_size) = cutlass::fmha::collective::VariableLength{max_seq_len_kv_cache, cumulative_seqlen_kv_cache.data()};
 197 |     }
 198 | 
 199 |     auto [batch, num_heads_q, num_heads_kv, head_size_qk, head_size_vo] = cute::select<0,1,2,6,7>(problem_size);
 200 |     int seq_len_qo, seq_len_kv, seq_len_kv_cache;
 201 | 
 202 |     int offset_q = 0;
 203 |     int offset_k = 0;
 204 |     int offset_v = 0;
 205 |     int offset_k_cache = 0;
 206 |     int offset_v_cache = 0;
 207 |     int offset_o = 0;
 208 |     // loop over the batch dimension to compute the output
 209 |     // to avoid the risk of running out of device memory
 210 |     int q_group_size = num_heads_q / num_heads_kv;
 211 |     for (int b = 0; b < batch; b++) {
 212 |       if constexpr (isVarLen) {
 213 |         auto logical_problem_shape = cutlass::fmha::collective::apply_variable_length(problem_size, b);
 214 |         seq_len_qo = get<3>(logical_problem_shape);
 215 |         seq_len_kv = get<4>(logical_problem_shape);
 216 |         seq_len_kv_cache = get<5>(logical_problem_shape);
 217 |       } else {
 218 |         seq_len_qo = get<3>(problem_size);
 219 |         seq_len_kv = get<4>(problem_size);
 220 |         seq_len_kv_cache = get<5>(problem_size);
 221 |       }
 222 | 
 223 |       int seq_len_kv_total = seq_len_kv_cache + seq_len_kv;
 224 |       int kv_group_update = 1;
 225 |       for (int h = 0; h < num_heads_q; h++) {
 226 |         cutlass::DeviceAllocation<ElementAccumulator> block_S;
 227 |         block_S.reset(seq_len_qo * seq_len_kv_total);
 228 | 
 229 |         ElementK* k_ptr;
 230 |         ElementV* v_ptr;
 231 | 
 232 |         if (seq_len_kv_cache > 0) {
 233 |             cutlass::DeviceAllocation<ElementK> block_K_concat(head_size_qk * seq_len_kv_total);
 234 |             cutlass::DeviceAllocation<ElementV> block_V_concat(seq_len_kv_total * head_size_vo);
 235 | 
 236 |             // Concatenate K_cache and K
 237 |             compat::memcpy<ElementK>(
 238 |                 block_K_concat.get(),
 239 |                 block_K_cache[0].get() + offset_k_cache,
 240 |                 seq_len_kv_cache * head_size_qk
 241 |             );
 242 |             compat::memcpy<ElementK>(
 243 |                 block_K_concat.get() + seq_len_kv_cache * head_size_qk,
 244 |                 block_K[0].get() + offset_k,
 245 |                 seq_len_kv * head_size_qk
 246 |             );
 247 | 
 248 |             // Concatenate V_cache and V
 249 |             compat::memcpy<ElementV>(
 250 |                 block_V_concat.get(),
 251 |                 block_V_cache[0].get() + offset_v_cache,
 252 |                 seq_len_kv_cache * head_size_vo
 253 |             );
 254 |             compat::memcpy<ElementV>(
 255 |                 block_V_concat.get() + seq_len_kv_cache * head_size_vo,
 256 |                 block_V[0].get() + offset_v,
 257 |                 seq_len_kv * head_size_vo
 258 |             );
 259 |             compat::wait();
 260 | 
 261 |             k_ptr = block_K_concat.get();
 262 |             v_ptr = block_V_concat.get();
 263 |         }
 264 |         else {
 265 |             k_ptr = block_K[0].get() + offset_k;
 266 |             v_ptr = block_V[0].get() + offset_v;
 267 |         }
 268 | 
 269 |         cutlass::TensorRef ref_Q(block_Q[0].get() + offset_q, LayoutQ::packed({seq_len_qo, head_size_qk}));
 270 |         cutlass::TensorRef ref_K(k_ptr, LayoutK::packed({head_size_qk, seq_len_kv_total}));
 271 |         cutlass::TensorRef ref_V(v_ptr, LayoutV::packed({seq_len_kv_total, head_size_vo}));
 272 |         cutlass::TensorRef ref_S(block_S.get(), LayoutQ::packed({seq_len_qo, seq_len_kv_total}));
 273 | 
 274 |         cutlass::reference::device::GemmComplex({seq_len_qo, seq_len_kv_total, head_size_qk}, ElementAccumulator{1}, ref_Q,
 275 |                                                 cutlass::ComplexTransform::kNone, ref_K, cutlass::ComplexTransform::kNone,
 276 |                                                 ElementAccumulator{0}, ref_S, ref_S, ElementAccumulator{0},
 277 |                                                 1,                   // batch_count
 278 |                                                 seq_len_qo * head_size_qk, // batch_stride_Q
 279 |                                                 seq_len_kv_total * head_size_qk, // batch_stride_K
 280 |                                                 seq_len_qo * seq_len_kv_total,   // batch_stride_S
 281 |                                                 seq_len_qo * seq_len_kv_total    // batch_stride_S
 282 |         );
 283 | 
 284 |         compat::wait();
 285 | 
 286 |         std::vector<ElementAccumulator> host_S(block_S.size());
 287 |         compat::memcpy<ElementAccumulator>(host_S.data(), block_S.get(), host_S.size());
 288 |         compat::wait();
 289 | 
 290 |         // delete this memory as it is no longer needed
 291 |         block_S.reset();
 292 | 
 293 |         auto offset = cute::min(seq_len_qo, seq_len_kv);
 294 |         auto discard_seq_coord = seq_len_qo - offset;
 295 |         auto full_tile_offset = seq_len_kv - offset;
 296 |         if (Causal) {
 297 |           // apply mask to S
 298 |           for (int row = 0; row < seq_len_qo; row++) {
 299 |             for (int col = seq_len_kv_cache; col < seq_len_kv_total; col++) {
 300 |               if ((col - full_tile_offset) > (row + seq_len_kv_cache - discard_seq_coord))
 301 |                 host_S[col + row * seq_len_kv_total] = ElementAccumulator{-INFINITY};
 302 |             }
 303 |           }
 304 |         }
 305 | 
 306 |         // compute max element per row of S
 307 |         std::vector<ElementAccumulator> max_vec(seq_len_qo, ElementAccumulator{-INFINITY});
 308 |         for (int row = 0; row < seq_len_qo; row++) {
 309 |           int idx = row * seq_len_kv_total;
 310 |           int max_idx = row;
 311 |           max_vec[max_idx] = host_S[idx++];
 312 |           for (int col = 1; col < seq_len_kv_total; col++, idx++) {
 313 |             if (max_vec[max_idx] < host_S[idx])
 314 |               max_vec[max_idx] = host_S[idx];
 315 |           }
 316 |         }
 317 | 
 318 |         // compute exp of S
 319 |         for (int row = 0; row < seq_len_qo; row++) {
 320 |           int idx = row * seq_len_kv_total;
 321 |           int max_idx = row;
 322 |           for (int col = 0; col < seq_len_kv_total; col++, idx++) {
 323 |             host_S[idx] = expf((host_S[idx] - max_vec[max_idx]) / std::sqrt(static_cast<ElementAccumulator>((head_size_qk))));
 324 |           }
 325 |         }
 326 | 
 327 |         // compute sum per row of S
 328 |         std::vector<ElementAccumulator> sum_vec(seq_len_qo, ElementAccumulator{0});
 329 |         for (int row = 0; row < seq_len_qo; row++) {
 330 |           int idx = row * seq_len_kv_total;
 331 |           int sum_idx = row;
 332 |           for (int col = 0; col < seq_len_kv_total; col++, idx++) {
 333 |             sum_vec[sum_idx] += host_S[idx];
 334 |           }
 335 | 
 336 |           // scale each row with the sum to compute softmax
 337 |           idx = row * seq_len_kv_total;
 338 |           sum_idx = row;
 339 |           for (int col = 0; col < seq_len_kv_total; col++, idx++) {
 340 |             if(Causal && row < discard_seq_coord) { 
 341 |               host_S[idx] = 0;
 342 |             } else {
 343 |               host_S[idx] /= sum_vec[sum_idx];
 344 |             }
 345 |           }
 346 |         }
 347 | 
 348 |         std::vector<ElementV> host_P(host_S.size());
 349 |         for (int p = 0; p < host_P.size(); p++)
 350 |           host_P[p] = static_cast<ElementV>(host_S[p]);
 351 | 
 352 |         cutlass::DeviceAllocation<ElementV> block_P;
 353 |         block_P.reset(host_P.size());
 354 | 
 355 |         compat::memcpy<ElementV>(block_P.get(), host_P.data(), host_P.size());
 356 |         compat::wait();
 357 | 
 358 |         cutlass::TensorRef ref_P(block_P.get(), LayoutQ::packed({seq_len_qo, seq_len_kv_total}));
 359 | 
 360 |         cutlass::DeviceAllocation<ElementAccumulator> block_acc;
 361 |         block_acc.reset(seq_len_qo * head_size_vo);
 362 |         cutlass::TensorRef ref_acc(block_acc.get(), LayoutO::packed({seq_len_qo, head_size_vo}));
 363 | 
 364 |         cutlass::reference::device::GemmComplex({seq_len_qo, head_size_vo, seq_len_kv_total}, ElementAccumulator{1}, ref_P,
 365 |                                                 cutlass::ComplexTransform::kNone, ref_V, cutlass::ComplexTransform::kNone,
 366 |                                                 ElementAccumulator{0}, ref_acc, ref_acc, ElementAccumulator{0},
 367 |                                                 1,                   // batch_count
 368 |                                                 seq_len_qo * seq_len_kv_total,   // batch_stride_P
 369 |                                                 seq_len_kv_total * head_size_vo, // batch_stride_V
 370 |                                                 seq_len_qo * head_size_vo, // batch_stride_O
 371 |                                                 seq_len_qo * head_size_vo  // batch_stride_O
 372 |         );
 373 | 
 374 |         compat::wait();
 375 |         // delete this memory as it is no longer needed
 376 |         block_P.reset();
 377 | 
 378 |         std::vector<ElementAccumulator> vec_acc(block_acc.size());
 379 |         compat::memcpy<ElementAccumulator>(vec_acc.data(), block_acc.get(), vec_acc.size());
 380 |         compat::wait();
 381 | 
 382 |         // delete this memory as it is no longer needed
 383 |         block_acc.reset();
 384 |         std::vector<ElementOutput> vec_out(vec_acc.size());
 385 |         for(int i = 0; i < vec_out.size(); i++) {
 386 |           vec_out[i] = static_cast<ElementOutput>(vec_acc[i]);
 387 |         }
 388 |         compat::memcpy<ElementOutput>(block_ref_O.get() + offset_o, vec_out.data(), vec_out.size());
 389 |         compat::wait();
 390 | 
 391 |         offset_q += seq_len_qo * head_size_qk;
 392 |         if(kv_group_update % q_group_size == 0) {
 393 |           offset_k += seq_len_kv * head_size_qk;
 394 |           offset_v += seq_len_kv * head_size_vo;
 395 |           offset_k_cache += seq_len_kv_cache * head_size_qk;
 396 |           offset_v_cache += seq_len_kv_cache * head_size_vo;
 397 |         }
 398 |         kv_group_update++;
 399 |         offset_o += seq_len_qo * head_size_vo;
 400 |       }
 401 |     }
 402 | 
 403 |     compat::wait();
 404 | 
 405 |     // Check if output from CUTLASS kernel and reference kernel are equal or not
 406 |     bool passed = cutlass::reference::device::BlockCompareRelativelyEqual(block_ref_O.get(), block_O.get(),
 407 |                                                                           block_O.size(), ElementOutput{0.5}, ElementOutput{0.5});
 408 | 
 409 |     return passed;
```
**EN:** Expands variable-length shapes, optionally reconstructs paged KV tensors, computes reference QK and PV products with masking and softmax, and compares the result with the kernel output.
**CN:** 展开变长形状，按需重建分页 KV 张量，计算带 mask 与 softmax 的参考 QK/PV 结果，并与内核输出比较。

### Lines 412-607 — Initialization helpers
```cpp
 412 |   template<class ProblemShape>
 413 |   auto initialize_varlen(const ProblemShape& problem_size) {
 414 |     int num_batches = get<0>(problem_size);
 415 | 
 416 |     // generate Q as --b times
 417 |     //    gaussian (--Q, --Q / 2) sampled positive
 418 |     //    track cumulative 
 419 |     std::mt19937 rng(0x202305151552ull);
 420 |     std::normal_distribution<double> dist_q(get<3>(problem_size), get<3>(problem_size) / 2);
 421 |     std::normal_distribution<double> dist_kv(get<4>(problem_size), get<4>(problem_size) / 2);
 422 |     std::normal_distribution<double> dist_kv_cache(get<5>(problem_size), get<5>(problem_size) / 2);
 423 | 
 424 |     // Use Cacheline Size to calculate alignment
 425 |     constexpr int cacheline_bytes = 64;
 426 |     constexpr int AlignmentQ = cacheline_bytes / sizeof(ElementQ);    // Alignment of Q matrix in units of elements
 427 |     constexpr int AlignmentKV = cacheline_bytes / sizeof(ElementK);   // Alignment of Kand V matrix in units of elements
 428 | 
 429 |     auto generate_positive_int = [](auto& dist, auto& gen) {
 430 |       int result = 0;
 431 |       do {
 432 |         result = static_cast<int>(dist(gen));
 433 |       } while (result <= 0);
 434 |       return result;
 435 |     };
 436 | 
 437 |     cumulative_seqlen_q = {0};
 438 |     cumulative_seqlen_kv = {0};
 439 |     cumulative_seqlen_kv_cache = {0};
 440 | 
 441 |     int total_seqlen_q = 0;
 442 |     int total_seqlen_kv = 0;
 443 |     int total_seqlen_kv_cache = 0;
 444 |     int max_seqlen_q = 0;
 445 |     int max_seqlen_kv = 0;
 446 |     int max_seqlen_kv_cache = 0;
 447 | 
 448 |     for (int i = 0; i < num_batches; i++) {
 449 |       //seqlen_q is usually set to 1 for decode.
 450 |       int seqlen_q = cute::get<3>(problem_size) == 1 ? 1 : std::min(cute::get<3>(problem_size), cutlass::round_up(generate_positive_int(dist_q, rng), AlignmentQ));
 451 |       int seqlen_kv = cutlass::round_up(generate_positive_int(dist_kv, rng), AlignmentKV);
 452 |       int seqlen_kv_cache = cute::get<5>(problem_size) == 0 ? 0 : cutlass::round_up(generate_positive_int(dist_kv_cache, rng), AlignmentKV);
 453 | 
 454 |       total_seqlen_q += seqlen_q;
 455 |       total_seqlen_kv += seqlen_kv;
 456 |       total_seqlen_kv_cache += seqlen_kv_cache;
 457 | 
 458 |       max_seqlen_q = std::max(max_seqlen_q, seqlen_q);
 459 |       max_seqlen_kv = std::max(max_seqlen_kv, seqlen_kv);
 460 |       max_seqlen_kv_cache = std::max(max_seqlen_kv_cache, seqlen_kv_cache);
 461 | 
 462 |       cumulative_seqlen_q.push_back(cumulative_seqlen_q.back() + seqlen_q);
 463 |       cumulative_seqlen_kv.push_back(cumulative_seqlen_kv.back() + seqlen_kv);
 464 |       cumulative_seqlen_kv_cache.push_back(cumulative_seqlen_kv_cache.back() + seqlen_kv_cache);
 465 |     }
 466 | 
 467 |     ProblemShape problem_size_for_init = problem_size;
 468 |     get<0>(problem_size_for_init) = 1;
 469 |     get<3>(problem_size_for_init) = total_seqlen_q;
 470 |     get<4>(problem_size_for_init) = total_seqlen_kv;
 471 |     get<5>(problem_size_for_init) = total_seqlen_kv_cache;
 472 | 
 473 |     ProblemShapeType problem_size_for_launch;
 474 | 
 475 |     get<3>(problem_size_for_launch) = cutlass::fmha::collective::VariableLength{max_seqlen_q};
 476 |     get<4>(problem_size_for_launch) = cutlass::fmha::collective::VariableLength{max_seqlen_kv};
 477 |     get<5>(problem_size_for_launch) = cutlass::fmha::collective::VariableLength{max_seqlen_kv_cache};
 478 |     get<0>(problem_size_for_launch) = get<0>(problem_size);
 479 |     get<1>(problem_size_for_launch) = get<1>(problem_size);
 480 |     get<2>(problem_size_for_launch) = get<2>(problem_size);
 481 |     get<6>(problem_size_for_launch) = get<6>(problem_size);
 482 |     get<7>(problem_size_for_launch) = get<7>(problem_size);
 483 | 
 484 |     return cute::make_tuple(problem_size_for_init, problem_size_for_launch);
 485 |   }
 486 | 
 487 |   /// Initialize operands to be used in the Flash Attention
 488 |   ProblemShapeType initialize(const FMHADecodeOptions &options) {
 489 |     auto problem_shape_in =
 490 |         cute::make_tuple(options.batch, options.num_heads_q, options.num_heads_kv, options.seq_len_qo, options.seq_len_kv, options.seq_len_kv_cache, options.head_size_qk, options.head_size_vo);
 491 | 
 492 |     ProblemShapeType problem_shape;
 493 |     decltype(problem_shape_in) problem_size;
 494 | 
 495 |     if constexpr (isVarLen) {
 496 |       auto [problem_shape_init, problem_shape_launch] = initialize_varlen(problem_shape_in);
 497 |       problem_shape = problem_shape_launch;
 498 |       problem_size = problem_shape_init;
 499 |     }
 500 |     else {
 501 |       problem_size = problem_shape_in;
 502 |       problem_shape = problem_shape_in;
 503 |     }
 504 | 
 505 |     auto [batch, num_heads_q, num_heads_kv, seq_len_qo, seq_len_kv, seq_len_kv_cache, head_size_qk, head_size_vo] = problem_size;
 506 | 
 507 |     stride_Q = cutlass::make_cute_packed_stride(StrideQ{}, cute::make_shape(seq_len_qo, head_size_qk, batch * num_heads_q));
 508 |     stride_K = cutlass::make_cute_packed_stride(StrideK{}, cute::make_shape(seq_len_kv, head_size_qk, batch * num_heads_kv));
 509 |     stride_V = cutlass::make_cute_packed_stride(StrideV{}, cute::make_shape(head_size_vo, seq_len_kv, batch * num_heads_kv));
 510 |     stride_K_cache = cutlass::make_cute_packed_stride(StrideK{}, cute::make_shape(seq_len_kv_cache, head_size_qk, batch * num_heads_kv));
 511 |     stride_V_cache = cutlass::make_cute_packed_stride(StrideV{}, cute::make_shape(head_size_vo, seq_len_kv_cache, batch * num_heads_kv));
 512 |     stride_O = cutlass::make_cute_packed_stride(StrideO{}, cute::make_shape(seq_len_qo, head_size_vo, batch * num_heads_q));
 513 | 
 514 |     std::size_t mem_size_q = static_cast<std::size_t>(batch) * num_heads_q * seq_len_qo * head_size_qk;
 515 |     std::size_t mem_size_k = static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv * head_size_qk;
 516 |     std::size_t mem_size_v = static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv * head_size_vo;
 517 |     std::size_t mem_size_k_cache = static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv_cache * head_size_qk;
 518 |     std::size_t mem_size_v_cache = static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv_cache * head_size_vo;
 519 |     std::size_t mem_size_o = static_cast<std::size_t>(batch) * num_heads_q * seq_len_qo * head_size_vo;
 520 | 
 521 |     std::size_t mem_occupied_QKV = (mem_size_q * sizeof(ElementQ)) + ((mem_size_k + mem_size_k_cache) * sizeof(ElementK)) + 
 522 |                                    ((mem_size_v + mem_size_v_cache) * sizeof(ElementV));
 523 | 
 524 |     count = std::ceil(static_cast<float>(cutlass::get_llc_size()) / static_cast<float>(mem_occupied_QKV)) + 1;
 525 | 
 526 |     if (PagedKV) {
 527 |       paged_kv_cache.page_size = options.page_size;
 528 |       std::vector<int> num_pages_per_seq{0};
 529 |       int num_pages = 0;
 530 |       for(int b = 0; b < get<0>(problem_shape); b++) {
 531 |         int seq_len_cache = isVarLen ? cumulative_seqlen_kv_cache[b + 1] - cumulative_seqlen_kv_cache[b] : seq_len_kv_cache;
 532 |         int pages_per_seq = ceil_div(seq_len_cache, paged_kv_cache.page_size);
 533 |         num_pages_per_seq.push_back(num_pages_per_seq.back() + pages_per_seq);
 534 |         num_pages += pages_per_seq;
 535 |       }
 536 |       paged_kv_cache.page_table.reset(num_pages);
 537 | 
 538 |       // initialize block table with random mapping for non-contiguous layout
 539 |       std::vector<int> page_mapping(num_pages);
 540 |       for (int b = 0; b < get<0>(problem_shape); ++b) {
 541 |         std::vector<int> physical_pages(num_pages_per_seq[b + 1] - num_pages_per_seq[b]);
 542 |         std::iota(physical_pages.begin(), physical_pages.end(), 0);
 543 |         // shuffle physical pages
 544 |         std::shuffle(physical_pages.begin(), physical_pages.end(), std::mt19937{ std::random_device{}() });
 545 |         for (int blk = 0; blk < physical_pages.size(); ++blk) {
 546 |           int logical_idx = num_pages_per_seq[b] + blk;
 547 |           page_mapping[logical_idx] = physical_pages[blk];
 548 |         }
 549 |       }
 550 |       compat::memcpy(paged_kv_cache.page_table.get(), page_mapping.data(), page_mapping.size() * sizeof(int));
 551 | 
 552 |       paged_kv_cache.num_pages_per_seq.reset(num_pages_per_seq.size());
 553 |       compat::memcpy(paged_kv_cache.num_pages_per_seq.get(), num_pages_per_seq.data(), num_pages_per_seq.size() * sizeof(int));
 554 |       compat::wait();
 555 |     }
 556 | 
 557 |     for(int i = 0; i < count; i++) {
 558 |       block_Q.emplace_back();
 559 |       block_K.emplace_back();
 560 |       block_V.emplace_back();
 561 |       block_K_cache.emplace_back();
 562 |       block_V_cache.emplace_back();
 563 |     }
 564 | 
 565 |     
 566 |     for(int i = 0; i < count; i++) {
 567 |       block_Q[i].reset(mem_size_q);
 568 |       block_K[i].reset(mem_size_k);
 569 |       block_V[i].reset(mem_size_v);
 570 |       block_K_cache[i].reset(mem_size_k_cache);
 571 |       block_V_cache[i].reset(mem_size_v_cache);
 572 | 
 573 |       initialize_block(block_Q[i], seed + i);
 574 |       initialize_block(block_K[i], seed + i + 100);
 575 |       initialize_block(block_V[i], seed + i + 101);
 576 |       initialize_block(block_K_cache[i], seed + i + 102);
 577 |       initialize_block(block_V_cache[i], seed + i + 103);
 578 |     }
 579 | 
 580 |     block_O.reset(mem_size_o);
 581 |     block_ref_O.reset(mem_size_o);
 582 | 
 583 |     if (!cumulative_seqlen_q.empty()) {
 584 |       device_cumulative_seqlen_q.reset(cumulative_seqlen_q.size());
 585 |       device_cumulative_seqlen_q.copy_from_host(
 586 |         cumulative_seqlen_q.data(), cumulative_seqlen_q.size());
 587 |     }
 588 | 
 589 |     if (!cumulative_seqlen_kv.empty()) {
 590 |       device_cumulative_seqlen_kv.reset(cumulative_seqlen_kv.size());
 591 |       device_cumulative_seqlen_kv.copy_from_host(
 592 |         cumulative_seqlen_kv.data(), cumulative_seqlen_kv.size());
 593 |     }
 594 | 
 595 |     if (!cumulative_seqlen_kv_cache.empty()) {
 596 |       device_cumulative_seqlen_kv_cache.reset(cumulative_seqlen_kv_cache.size());
 597 |       device_cumulative_seqlen_kv_cache.copy_from_host(
 598 |         cumulative_seqlen_kv_cache.data(), cumulative_seqlen_kv_cache.size());
 599 |     }
 600 | 
 601 |     if constexpr (isVarLen) {
 602 |       get<3>(problem_shape).cumulative_length = device_cumulative_seqlen_q.get();
 603 |       get<4>(problem_shape).cumulative_length = device_cumulative_seqlen_kv.get();
 604 |       get<5>(problem_shape).cumulative_length = device_cumulative_seqlen_kv_cache.get();
 605 |     }
 606 | 
 607 |     return problem_shape;
```
**EN:** Generates variable-length sequence layouts, allocates and initializes device memory, builds page tables, and prepares launch-time problem shapes.
**CN:** 生成变长序列布局，分配并初始化设备内存，构建页表，并准备启动阶段使用的问题形状。

### Lines 610-778 — Benchmark execution
```cpp
 610 |   static void run(typename FMHADecodeKernel::Params params) {
 611 |     dim3 const block = FMHADecodeKernel::get_block_shape();
 612 |     dim3 const grid = FMHADecodeKernel::get_grid_shape(params);
 613 | 
 614 |     // configure smem size and carveout
 615 |     int smem_size = FMHADecodeKernel::SharedStorageSize;
 616 | 
 617 |     const auto sycl_block = compat::dim3(block.x, block.y, block.z);
 618 |     const auto sycl_grid = compat::dim3(grid.x, grid.y, grid.z);
 619 | 
 620 | #if !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)
 621 |     using namespace compat::experimental;
 622 |     #if defined(CUTLASS_SYCL_PROFILING_ENABLED)
 623 |     auto event = launch<cutlass::device_kernel<FMHADecodeKernel>, FMHADecodeKernel>(
 624 |         launch_policy{sycl_grid, sycl_block, local_mem_size{static_cast<std::size_t>(smem_size)},
 625 |                       kernel_properties{sycl_exp::sub_group_size<FMHADecodeKernel::DispatchPolicy::SubgroupSize>}},
 626 |         params);
 627 |     EventManager::getInstance().addEvent(event);
 628 |     #else
 629 |     launch<cutlass::device_kernel<FMHADecodeKernel>, FMHADecodeKernel, false>(
 630 |         launch_policy{sycl_grid, sycl_block, local_mem_size{static_cast<std::size_t>(smem_size)},
 631 |                       kernel_properties{sycl_exp::sub_group_size<FMHADecodeKernel::DispatchPolicy::SubgroupSize>}},
 632 |         params);
 633 |     #endif
 634 | #else
 635 |     compat::experimental::launch_properties launch_props{
 636 |       sycl::ext::oneapi::experimental::work_group_scratch_size(smem_size)
 637 |     };
 638 |     compat::experimental::kernel_properties kernel_props{
 639 |       sycl::ext::oneapi::experimental::sub_group_size<FMHADecodeKernel::DispatchPolicy::SubgroupSize>
 640 |     };
 641 |     compat::experimental::launch_policy policy{sycl_grid, sycl_block, launch_props, kernel_props};
 642 |     #if defined(CUTLASS_SYCL_PROFILING_ENABLED)
 643 |     auto event = compat::experimental::launch<cutlass::device_kernel<FMHADecodeKernel>, FMHADecodeKernel>(policy, params);
 644 |     EventManager::getInstance().addEvent(event);
 645 |     #else
 646 |         compat::experimental::launch<cutlass::device_kernel<FMHADecodeKernel>, FMHADecodeKernel, false>(policy, params);
 647 |     #endif
 648 | #endif
 649 |   }
 650 | 
 651 |   void run(::benchmark::State& state, const FMHADecodeOptions &options, const cutlass::KernelHardwareInfo &hw_info) {
 652 | 
 653 |     ProblemShapeType problem_size = initialize(options);
 654 | 
 655 |     typename FMHADecodeKernel::Arguments arguments{
 656 |         cutlass::gemm::GemmUniversalMode::kGemm,
 657 |         problem_size,
 658 |         {block_Q[0].get(), stride_Q,
 659 |         block_K[0].get(), stride_K,
 660 |         block_V[0].get(), stride_V,
 661 |         block_K_cache[0].get(), stride_K_cache,
 662 |         block_V_cache[0].get(), stride_V_cache,
 663 |         PagedKV ? paged_kv_cache.page_table.get() : nullptr,
 664 |         PagedKV ? paged_kv_cache.page_size : 0,
 665 |         PagedKV ? paged_kv_cache.num_pages_per_seq.get() : nullptr},
 666 |         {options.softmax_scale},
 667 |         {block_O.get(), stride_O},
 668 |         hw_info};
 669 | 
 670 |     size_t workspace_size = FMHADecodeKernel::get_workspace_size(arguments);
 671 |     cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
 672 | 
 673 |     FMHADecodeKernel::can_implement(arguments);
 674 | 
 675 |     // Initialize the workspace
 676 |     auto status = FMHADecodeKernel::initialize_workspace(arguments, workspace.get());
 677 |     if (status != cutlass::Status::kSuccess) {
 678 |       return;
 679 |     }
 680 | 
 681 |     typename FMHADecodeKernel::Params params = FMHADecodeKernel::to_underlying_arguments(arguments, workspace.get());
 682 | 
 683 |     // Run the GEMM
 684 |     run(params);
 685 | 
 686 |     compat::wait();
 687 | 
 688 |     // Verify that the result is correct
 689 |     bool passed = verify(problem_size);
 690 |     if(not passed) {
 691 |       state.SkipWithError("Disposition Failed.");
 692 |     }
 693 | 
 694 |     state.counters["batch"] = options.batch;
 695 |     state.counters["num_heads_q"] = options.num_heads_q;
 696 |     state.counters["num_heads_kv"] = options.num_heads_kv;
 697 |     state.counters["seq_len_qo"] = options.seq_len_qo;
 698 |     state.counters["seq_len_kv"] = options.seq_len_kv;
 699 |     state.counters["seq_len_kv_cache"] = options.seq_len_kv_cache;
 700 |     state.counters["head_size_kv"] = options.head_size_qk;
 701 |     state.counters["head_size_vo"] = options.head_size_vo;
 702 |     state.counters["page_size"] = options.page_size;
 703 |     state.counters["scale"] = options.softmax_scale;
 704 |     state.counters["causal"] = Causal;
 705 |     state.counters["varlen"] = isVarLen;
 706 |     state.counters["paged_kv"] = PagedKV;
 707 | 
 708 |     std::stringstream extra_label;
 709 |     extra_label << "layoutQ=RowMajor ";
 710 |     extra_label << "layoutK=ColumnMajor ";
 711 |     extra_label << "layoutV=RowMajor ";
 712 | 
 713 |     state.SetLabel(extra_label.str());
 714 |     // when seq_len_qo is not equal to seq_len_kv we use bottom up approach for the masking. 
 715 |     // Following changes will adjust the effective_seq_len_kv when masking applied for such cases.
 716 |     auto offset = cute::min(options.seq_len_qo, options.seq_len_kv);
 717 |     auto discard_seq_coord = options.seq_len_qo - offset;
 718 |     auto full_tile_offset = options.seq_len_kv - offset;
 719 |     auto effective_seq_len_kv = Causal ? full_tile_offset + ((offset + 1) / 2.0): options.seq_len_kv;
 720 |     auto effective_seq_len_qo = Causal ? options.seq_len_qo - discard_seq_coord  : options.seq_len_qo;
 721 |    
 722 |     double flops_qk = 2.0 * options.batch * options.num_heads_q * effective_seq_len_qo * effective_seq_len_kv * options.head_size_qk;
 723 |     double flops_pv = 2.0 * options.batch * options.num_heads_q * effective_seq_len_qo * options.head_size_vo * effective_seq_len_kv;
 724 |     double gflops = (flops_qk + flops_pv) * 1e-9;
 725 | 
 726 |     // TODO: Use sizeof_bits_v instead of sizeof if QKVO is smaller than 8 bits, which avoids incorrect bandwidth calculation
 727 |     double gbps_qk =  options.batch * (sizeof(ElementQ) * options.num_heads_q * effective_seq_len_qo * options.head_size_qk + 
 728 |                       sizeof(ElementK) * options.num_heads_kv * effective_seq_len_kv * options.head_size_qk);    
 729 |     double gbps_pv = sizeof(ElementV) * options.batch * options.num_heads_kv * effective_seq_len_kv * options.head_size_vo +
 730 |                      sizeof(ElementOutput) * options.batch * options.num_heads_q * effective_seq_len_qo * options.head_size_vo;
 731 |     double mega_bytes_transferred = (gbps_qk + gbps_pv) * (1e-6);
 732 | 
 733 |     initialize_counters(state);
 734 |     int32_t counter = 1;
 735 |     for(auto _ : state) {
 736 |       state.PauseTiming();
 737 |       int input_num = std::max(int(0), counter % count);
 738 | 
 739 |       typename FMHADecodeKernel::Arguments arguments{
 740 |         cutlass::gemm::GemmUniversalMode::kGemm,
 741 |         problem_size,
 742 |         {block_Q[input_num].get(), stride_Q,
 743 |         block_K[input_num].get(), stride_K,
 744 |         block_V[input_num].get(), stride_V,
 745 |         block_K_cache[input_num].get(), stride_K_cache,
 746 |         block_V_cache[input_num].get(), stride_V_cache,
 747 |         PagedKV ? paged_kv_cache.page_table.get() : nullptr,
 748 |         PagedKV ? paged_kv_cache.page_size : 0,
 749 |         PagedKV ? paged_kv_cache.num_pages_per_seq.get() : nullptr},
 750 |         {options.softmax_scale},
 751 |         {block_O.get(), stride_O},
 752 |         hw_info};
 753 | 
 754 |       size_t workspace_size = FMHADecodeKernel::get_workspace_size(arguments);
 755 |       cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
 756 | 
 757 |       FMHADecodeKernel::can_implement(arguments);
 758 | 
 759 |       // Initialize the workspace
 760 |       auto status = FMHADecodeKernel::initialize_workspace(arguments, workspace.get());
 761 |       if (status != cutlass::Status::kSuccess) {
 762 |         return;
 763 |       }
 764 | 
 765 |       typename FMHADecodeKernel::Params params = FMHADecodeKernel::to_underlying_arguments(arguments, workspace.get());
 766 | 
 767 |       state.ResumeTiming();
 768 | 
 769 |       GPU_Clock timer;
 770 |       timer.start();
 771 |       run(params);
 772 |       auto ms_elapsed = timer.milliseconds();
 773 |       update_counters(state, ms_elapsed);
 774 |       state.SetIterationTime(ms_elapsed / 1000);
 775 |       counter++;
 776 |     }
 777 |     finalize_counters(state, gflops, mega_bytes_transferred);
 778 |   }
```
**EN:** Builds arguments, allocates workspace, launches the kernel, validates correctness, records metadata, and runs the timed benchmark loop.
**CN:** 构建参数，分配工作区，启动内核，验证正确性，记录元数据，并执行计时基准循环。

### Lines 781-800 — Counter helpers
```cpp
 781 |   static void initialize_counters(::benchmark::State& state) {
 782 |     state.counters["avg_runtime_ms"] = 0;
 783 |     state.counters["best_runtime_ms"] = std::numeric_limits<double>::max();
 784 |   }
 785 | 
 786 |   static void update_counters(::benchmark::State& state, double ms_elapsed) {
 787 |     state.PauseTiming();
 788 |     state.counters["total_runtime_ms"] += ms_elapsed;
 789 |     state.counters["best_runtime_ms"] = std::min<double>(state.counters["best_runtime_ms"], ms_elapsed);
 790 |     state.ResumeTiming();
 791 |   }
 792 | 
 793 |   static void finalize_counters(::benchmark::State& state,  double gflop, double mega_bytes_transferred) {
 794 |     state.counters["avg_runtime_ms"] =
 795 |       state.counters["total_runtime_ms"] / static_cast<double>(state.iterations());
 796 |     state.counters["avg_tflops"] = gflop / state.counters["avg_runtime_ms"];
 797 |     state.counters["avg_throughput"] = mega_bytes_transferred / state.counters["avg_runtime_ms"];
 798 |     state.counters["best_tflop"] = gflop / state.counters["best_runtime_ms"];
 799 |     state.counters["best_bandwidth"] = mega_bytes_transferred / state.counters["best_runtime_ms"];
 800 |   }
```
**EN:** Maintains accumulated runtime statistics and converts them into throughput-oriented counters.
**CN:** 维护累计运行时间统计，并把它们转换成面向吞吐的计数器。

### Lines 805-814 — Registration macros
```cpp
 805 | #define CUTLASS_FMHA_DECODE_BENCHMARK(F) cutlass::benchmark::BenchmarkRegistry<cutlass::benchmark::FMHADecodeOptions>::Register(#F, &F##_func)
 806 | 
 807 | #define CUTLASS_CREATE_FMHA_DECODE_BENCHMARK(F)                          \
 808 |   static void F##_func(                                           \
 809 |       ::benchmark::State& state,                                  \
 810 |       cutlass::benchmark::FMHADecodeOptions const& options,                 \
 811 |       cutlass::KernelHardwareInfo const& hw_info) {               \
 812 |     auto bench = cutlass::benchmark::BenchmarkRunnerFMHADecode<F>();    \
 813 |     bench.run(state, options, hw_info);                           \
 814 |   }
```
**EN:** Wraps configuration types into registry-friendly functions and exposes a macro to register them by name.
**CN:** 把配置类型包装为可注册的函数，并提供按名称注册它们的宏。

## Key Concepts / 关键概念

- Decode CLI parsing / decode 命令行解析
- Reference decode validation / decode 参考结果校验
- Varlen and paged-KV setup / 变长与分页 KV 设置
- Macro-based benchmark wrapping / 基于宏的基准包装

## Dependencies / 依赖关系

- `flash_attention` decode kernels / Flash Attention decode 内核
- `cutlass::DeviceAllocation` — device buffers / 设备缓冲区
- `cutlass::reference::device::*` — reference math / 参考计算
- `BenchmarkRegistry<FMHADecodeOptions>` — decode benchmark registry / decode 基准注册表
