# benchmark_runner.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/flash_attention/legacy/flash_attention_prefill/benchmark_runner.hpp`
- **EN:** Legacy prefill benchmark harness that prepares FMHA inputs, verifies results through a reference path, launches the kernel, and reports performance.
- **CN:** 旧版 prefill 基准运行器：准备 FMHA 输入，通过参考路径校验结果，启动内核，并汇报性能。

## Line-by-Line Analysis / 逐行分析

### Lines 1-58 — File prologue
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
  36 | #include "flash_attention_v2/kernel/legacy/tile_scheduler.hpp"
  37 | #include "cutlass/gemm/device/gemm_universal_adapter.h"
  38 | #include "cutlass/util/packed_stride.hpp"
  39 | #include "flash_attention_v2/kernel/legacy/xe_flash_attn_prefill.hpp"
  40 | #include "flash_attention_v2/collective/legacy/xe_flash_attn_prefill_epilogue.hpp"
  41 | #include "flash_attention_v2/collective/legacy/xe_flash_attn_prefill_softmax_epilogue.hpp"
  42 | #include "cutlass/util/GPU_Clock.hpp"
  43 | #include "cutlass/util/sycl_event_manager.hpp"
  44 | 
  45 | #include <cute/tensor.hpp>
  46 | #include <random>
  47 | 
  48 | #include "cutlass/util/command_line.h"
  49 | #include "cutlass/util/device_memory.h"
  50 | #include "cutlass/util/reference/device/gemm_complex.h"
  51 | #include "cutlass/util/reference/device/tensor_compare.h"
  52 | #include "../examples/common/sycl_common.hpp"
  53 | #include "../../../common.hpp"
  54 | 
  55 | using namespace cute;
  56 | 
  57 | namespace cutlass::benchmark {
  58 | 
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 59-103 — FMHAOptions
```cpp
  59 | // Command line options parsing
  60 | struct FMHAOptions {
  61 | 
  62 |   bool error;
  63 | 
  64 |   int batch, num_heads_q, num_heads_kv, seq_len_qo, seq_len_kv, head_size_qk, head_size_vo, iterations;
  65 |   float softmax_scale;
  66 |   std::string bm_name;
  67 | 
  68 |   FMHAOptions()
  69 |       : error(false), batch(32), num_heads_q(16), num_heads_kv(16), seq_len_qo(512), head_size_qk(128),
  70 |         seq_len_kv(512), head_size_vo(128), iterations(100), softmax_scale(1.f), bm_name("Flash Attention v2") {}
  71 | 
  72 |   // Parses the command line
  73 |   void parse(int argc, char const **args) {
  74 |     cutlass::CommandLine cmd(argc, args);
  75 | 
  76 |     cmd.get_cmd_line_argument("batch", batch, 32);
  77 |     cmd.get_cmd_line_argument("num_heads_q", num_heads_q, 16);
  78 |     cmd.get_cmd_line_argument("num_heads_kv", num_heads_kv, num_heads_q);
  79 |     cmd.get_cmd_line_argument("seq_len_qo", seq_len_qo, 512);
  80 |     cmd.get_cmd_line_argument("seq_len_kv", seq_len_kv, seq_len_qo);
  81 |     cmd.get_cmd_line_argument("head_size_vo", head_size_vo, 128);
  82 |     cmd.get_cmd_line_argument("head_size_qk", head_size_qk, head_size_vo);
  83 |     cmd.get_cmd_line_argument("iterations", iterations, 100);
  84 |     cmd.get_cmd_line_argument("bm_name", bm_name, std::string("Flash Attention v2"));
  85 | 
  86 |     softmax_scale = 1 / std::sqrt(static_cast<float>(head_size_qk));
  87 |   }
  88 | 
  89 |   std::string benchmark_name() const {
  90 |     std::stringstream full_name;
  91 |     full_name << bm_name << "/";
  92 |     std::string const test_name_suffix = std::to_string(batch) + "x" +
  93 |                                    std::to_string(num_heads_q) + "x" +
  94 |                                    std::to_string(num_heads_kv) + "x" +
  95 |                                    std::to_string(seq_len_qo) + "x" +
  96 |                                    std::to_string(head_size_qk) + "x" +
  97 |                                    std::to_string(seq_len_kv) + "x" +
  98 |                                    std::to_string(head_size_vo);
  99 |     full_name << test_name_suffix;
 100 | 
 101 |     return full_name.str();
 102 |   }
 103 | };
```
**EN:** Parses prefill benchmark arguments, computes the default softmax scale, and formats the benchmark name.
**CN:** 解析 prefill 基准参数，计算默认 softmax 缩放，并格式化基准名称。

### Lines 107-159 — Runner state
```cpp
 107 | template <class FMHAPrefillConfiguration> struct BenchmarkRunnerFMHA {
 108 | 
 109 |   using GemmKernel = typename FMHAPrefillConfiguration::GemmKernel;
 110 |   
 111 |   using LayoutQ = typename FMHAPrefillConfiguration::LayoutQ;
 112 |   using LayoutK = typename FMHAPrefillConfiguration::LayoutK;
 113 |   using LayoutV = typename FMHAPrefillConfiguration::LayoutV;
 114 |   using LayoutO = typename FMHAPrefillConfiguration::LayoutO;
 115 | 
 116 |   using StrideQ = typename GemmKernel::StrideQ;
 117 |   using StrideK = typename GemmKernel::StrideK;
 118 |   using StrideV = typename GemmKernel::StrideV;
 119 |   using StrideO = typename GemmKernel::StrideO;
 120 | 
 121 |   using ElementQ = typename GemmKernel::ElementQ;
 122 |   using ElementK = typename GemmKernel::ElementK;
 123 |   using ElementV = typename GemmKernel::ElementV;
 124 |   using ElementAcc = typename GemmKernel::ElementAccumulator;
 125 | 
 126 |   using CollectiveEpilogue = typename GemmKernel::CollectiveEpilogue;
 127 |   using ElementOutput = typename CollectiveEpilogue::ElementOutput;
 128 |   using ElementCompute = typename CollectiveEpilogue::ElementCompute;
 129 |   using ElementAccumulator = typename CollectiveEpilogue::ElementAccumulator;
 130 | 
 131 |   using ProblemShapeType = typename GemmKernel::ProblemShape;
 132 |   static constexpr bool Causal = FMHAPrefillConfiguration::Causal;
 133 |   static constexpr bool isVarLen = FMHAPrefillConfiguration::VarLen;
 134 | 
 135 |   int32_t count;
 136 | 
 137 |   //
 138 |   // Data members
 139 |   //
 140 | 
 141 |   /// Initialization
 142 |   StrideQ stride_Q;
 143 |   StrideK stride_K;
 144 |   StrideV stride_V;
 145 |   StrideO stride_O;
 146 |   uint64_t seed = 0;
 147 | 
 148 |   std::vector<cutlass::DeviceAllocation<ElementQ>> block_Q;
 149 |   std::vector<cutlass::DeviceAllocation<ElementK>> block_K;
 150 |   std::vector<cutlass::DeviceAllocation<ElementV>> block_V;
 151 |   cutlass::DeviceAllocation<ElementOutput> block_O;
 152 |   cutlass::DeviceAllocation<ElementOutput> block_ref_O;
 153 | 
 154 |   std::vector<int> cumulative_seqlen_q;
 155 |   std::vector<int> cumulative_seqlen_kv;
 156 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_q;
 157 |   cutlass::DeviceAllocation<int> device_cumulative_seqlen_kv;
 158 | 
 159 |   //
```
**EN:** Defines the prefill runner and stores kernel-dependent types, packed strides, buffers, and cumulative-length arrays.
**CN:** 定义 prefill 运行器，并保存依赖内核的类型、紧凑步长、缓冲区以及累计长度数组。

### Lines 163-330 — Reference verification
```cpp
 163 |   bool verify(ProblemShapeType problem_size) {
 164 |     
 165 |     if constexpr (isVarLen) {
 166 |       int max_seq_len_q = static_cast<int>(get<3>(problem_size));
 167 |       int max_seq_len_kv = static_cast<int>(get<4>(problem_size));
 168 |       get<3>(problem_size) = cutlass::fmha::collective::VariableLength{max_seq_len_q, cumulative_seqlen_q.data()};
 169 |       get<4>(problem_size) = cutlass::fmha::collective::VariableLength{max_seq_len_kv, cumulative_seqlen_kv.data()};
 170 |     }
 171 | 
 172 |     auto [batch, num_heads_q, num_heads_kv, head_size_qk, head_size_vo] = cute::select<0,1,2,5,6>(problem_size);
 173 |     int seq_len_qo, seq_len_kv;
 174 | 
 175 |     int offset_q = 0;
 176 |     int offset_k = 0;
 177 |     int offset_v = 0;
 178 |     int offset_o = 0;
 179 |     // loop over the batch dimension to compute the output
 180 |     // to avoid the risk of running out of device memory
 181 |     int q_group_size = num_heads_q / num_heads_kv;
 182 |     for (int b = 0; b < batch; b++) {
 183 |       if constexpr (isVarLen) {
 184 |         auto logical_problem_shape = cutlass::fmha::collective::apply_variable_length(problem_size, b);
 185 |         seq_len_qo = get<3>(logical_problem_shape);
 186 |         seq_len_kv = get<4>(logical_problem_shape);
 187 |       } else {
 188 |         seq_len_qo = get<3>(problem_size);
 189 |         seq_len_kv = get<4>(problem_size);
 190 |       }
 191 |       int kv_group_update=1;
 192 |       for (int h = 0; h < num_heads_q; h++) {
 193 |         cutlass::DeviceAllocation<ElementAccumulator> block_S;
 194 |         block_S.reset(seq_len_qo * seq_len_kv);
 195 | 
 196 |         cutlass::TensorRef ref_Q(block_Q[0].get() + offset_q, LayoutQ::packed({seq_len_qo, head_size_qk}));
 197 |         cutlass::TensorRef ref_K(block_K[0].get() + offset_k, LayoutK::packed({head_size_qk, seq_len_kv}));
 198 |         cutlass::TensorRef ref_V(block_V[0].get() + offset_v, LayoutV::packed({seq_len_kv, head_size_vo}));
 199 |         cutlass::TensorRef ref_S(block_S.get(), LayoutQ::packed({seq_len_qo, seq_len_kv}));
 200 | 
 201 |         cutlass::reference::device::GemmComplex({seq_len_qo, seq_len_kv, head_size_qk}, ElementAccumulator{1.f}, ref_Q,
 202 |                                                 cutlass::ComplexTransform::kNone, ref_K, cutlass::ComplexTransform::kNone,
 203 |                                                 ElementAccumulator{0}, ref_S, ref_S, ElementAccumulator{0},
 204 |                                                 1,                   // batch_count
 205 |                                                 seq_len_qo * head_size_qk, // batch_stride_Q
 206 |                                                 seq_len_kv * head_size_qk, // batch_stride_K
 207 |                                                 seq_len_qo * seq_len_kv,   // batch_stride_S
 208 |                                                 seq_len_qo * seq_len_kv    // batch_stride_S
 209 |         );
 210 | 
 211 |         compat::wait();
 212 | 
 213 |         std::vector<ElementAccumulator> host_S(block_S.size());
 214 |         compat::memcpy<ElementAccumulator>(host_S.data(), block_S.get(), host_S.size());
 215 | 
 216 |         // delete this memory as it is no longer needed
 217 |         block_S.reset();
 218 |         auto offset = cute::min(seq_len_qo, seq_len_kv);
 219 |         auto discard_seq_coord = seq_len_qo - offset;
 220 |         auto full_tile_offset = seq_len_kv - offset;
 221 |         if constexpr (Causal) {
 222 |           // apply mask to S
 223 |           for (int row = 0; row < seq_len_qo; row++) {
 224 |             for (int col = 0; col < seq_len_kv; col++) {
 225 |               if ((col - full_tile_offset) > (row - discard_seq_coord))
 226 |                 host_S[col + row * seq_len_kv] = ElementAccumulator{-INFINITY};
 227 |             }
 228 |           }
 229 |         }
 230 | 
 231 |         // compute max element per row of S
 232 |         std::vector<ElementAccumulator> max_vec(seq_len_qo, ElementAccumulator{-INFINITY});
 233 |         for (int row = 0; row < seq_len_qo; row++) {
 234 |           int idx = row * seq_len_kv;
 235 |           int max_idx = row;
 236 |           max_vec[max_idx] = host_S[idx++];
 237 |           for (int col = 1; col < seq_len_kv; col++, idx++) {
 238 |             if (max_vec[max_idx] < host_S[idx])
 239 |               max_vec[max_idx] = host_S[idx];
 240 |           }
 241 |         }
 242 | 
 243 |         // compute exp of S
 244 |         for (int row = 0; row < seq_len_qo; row++) {
 245 |           int idx = row * seq_len_kv;
 246 |           int max_idx = row;
 247 |           for (int col = 0; col < seq_len_kv; col++, idx++) {
 248 |             host_S[idx] = expf((host_S[idx] - max_vec[max_idx]) / std::sqrt(static_cast<ElementAccumulator>((head_size_qk))));
 249 |           }
 250 |         }
 251 | 
 252 |         // compute sum per row of S
 253 |         std::vector<ElementAccumulator> sum_vec(seq_len_qo, ElementAccumulator{0});
 254 |         for (int row = 0; row < seq_len_qo; row++) {
 255 |           int idx = row * seq_len_kv;
 256 |           int sum_idx = row;
 257 |           for (int col = 0; col < seq_len_kv; col++, idx++) {
 258 |             sum_vec[sum_idx] += host_S[idx];
 259 |           }
 260 | 
 261 |           // scale each row with the sum to compute softmax
 262 |           idx = row * seq_len_kv;
 263 |           sum_idx = row;
 264 |           for (int col = 0; col < seq_len_kv; col++, idx++) {
 265 |             if(Causal && row < discard_seq_coord) {
 266 |               host_S[idx] = 0;
 267 |             } else {
 268 |               host_S[idx] /= sum_vec[sum_idx];
 269 |             }
 270 |           }
 271 |         }
 272 | 
 273 |         std::vector<ElementV> host_P(host_S.size());
 274 |         for (int p = 0; p < host_P.size(); p++)
 275 |           host_P[p] = static_cast<ElementV>(host_S[p]);
 276 | 
 277 |         cutlass::DeviceAllocation<ElementV> block_P;
 278 |         block_P.reset(host_P.size());
 279 | 
 280 |         compat::memcpy<ElementV>(block_P.get(), host_P.data(), host_P.size());
 281 | 
 282 |         cutlass::TensorRef ref_P(block_P.get(), LayoutQ::packed({seq_len_qo, seq_len_kv}));
 283 | 
 284 |         cutlass::DeviceAllocation<ElementAccumulator> block_acc;
 285 |         block_acc.reset(seq_len_qo * head_size_vo);
 286 |         cutlass::TensorRef ref_acc(block_acc.get(), LayoutO::packed({seq_len_qo, head_size_vo}));
 287 | 
 288 |         cutlass::reference::device::GemmComplex({seq_len_qo, head_size_vo, seq_len_kv}, ElementAccumulator{1}, ref_P,
 289 |                                                 cutlass::ComplexTransform::kNone, ref_V, cutlass::ComplexTransform::kNone,
 290 |                                                 ElementAccumulator{0}, ref_acc, ref_acc, ElementAccumulator{0},
 291 |                                                 1,                   // batch_count
 292 |                                                 seq_len_qo * seq_len_kv,   // batch_stride_P
 293 |                                                 seq_len_kv * head_size_vo, // batch_stride_V
 294 |                                                 seq_len_qo * head_size_vo, // batch_stride_O
 295 |                                                 seq_len_qo * head_size_vo  // batch_stride_O
 296 |         );
 297 | 
 298 |         compat::wait();
 299 |         // delete this memory as it is no longer needed
 300 |         block_P.reset();
 301 | 
 302 |         std::vector<ElementAccumulator> vec_acc(block_acc.size());
 303 |         compat::memcpy<ElementAccumulator>(vec_acc.data(), block_acc.get(), vec_acc.size());
 304 | 
 305 |         // delete this memory as it is no longer needed
 306 |         block_acc.reset();
 307 |         std::vector<ElementOutput> vec_out(vec_acc.size());
 308 |         for(int i = 0; i < vec_out.size(); i++) {
 309 |           vec_out[i] = static_cast<ElementOutput>(vec_acc[i]);
 310 |         }
 311 |         compat::memcpy<ElementOutput>(block_ref_O.get() + offset_o, vec_out.data(), vec_out.size());
 312 | 
 313 |         offset_q += seq_len_qo * head_size_qk;
 314 |         if(kv_group_update % q_group_size==0) {
 315 |           offset_k += seq_len_kv * head_size_qk;
 316 |           offset_v += seq_len_kv * head_size_vo;
 317 |         }
 318 |         kv_group_update++;
 319 |         offset_o += seq_len_qo * head_size_vo;
 320 |       }
 321 |     }
 322 | 
 323 |     compat::wait();
 324 | 
 325 |     // Check if output from CUTLASS kernel and reference kernel are equal or not
 326 |     bool passed = cutlass::reference::device::BlockCompareRelativelyEqual(block_ref_O.get(), block_O.get(),
 327 |                                                                           block_O.size(), ElementOutput{0.5}, ElementOutput{0.5});
 328 | 
 329 |     return passed;
 330 |   }
```
**EN:** Computes a reference QK product, applies causal masking, performs the softmax normalization, multiplies by V, and compares the result to the kernel output.
**CN:** 计算参考 QK 乘积，应用因果 mask，执行 softmax 归一化，再与 V 相乘，并把结果与内核输出比较。

### Lines 332-468 — Initialization helpers
```cpp
 332 |   template<class ProblemShape>
 333 |   auto initialize_varlen(const ProblemShape& problem_size) {
 334 |     int num_batches = get<0>(problem_size);
 335 | 
 336 |     // generate Q as --b times
 337 |     //    gaussian (--Q, --Q / 2) sampled positive
 338 |     //    track cumulative 
 339 |     std::mt19937 rng(0x202305151552ull);
 340 |     std::normal_distribution<double> dist_q(get<3>(problem_size), get<3>(problem_size) / 2);
 341 |     std::normal_distribution<double> dist_kv(get<4>(problem_size), get<4>(problem_size) / 2);
 342 | 
 343 |     // Use Cacheline Size to calculate alignment
 344 |     constexpr int cacheline_bytes = 64;
 345 |     constexpr int AlignmentQ = cacheline_bytes / sizeof(ElementQ);    // Alignment of Q matrix in units of elements
 346 |     constexpr int AlignmentKV = cacheline_bytes / sizeof(ElementK);   // Alignment of Kand V matrix in units of elements
 347 | 
 348 |     auto generate_positive_int = [](auto& dist, auto& gen) {
 349 |       int result = 0;
 350 |       do {
 351 |         result = static_cast<int>(dist(gen));
 352 |       } while (result <= 0);
 353 |       return result;
 354 |     };
 355 | 
 356 |     cumulative_seqlen_q = {0};
 357 |     cumulative_seqlen_kv = {0};
 358 | 
 359 |     int total_seqlen_q = 0;
 360 |     int total_seqlen_kv = 0;
 361 |     int max_seqlen_q = 0;
 362 |     int max_seqlen_kv = 0;
 363 | 
 364 |     for (int i = 0; i < num_batches; i++) {
 365 |       int seqlen_q = cutlass::round_up(generate_positive_int(dist_q, rng), AlignmentQ);
 366 |       int seqlen_kv = cutlass::round_up(generate_positive_int(dist_kv, rng), AlignmentKV);
 367 | 
 368 |       total_seqlen_q += seqlen_q;
 369 |       total_seqlen_kv += seqlen_kv;
 370 | 
 371 |       max_seqlen_q = std::max(max_seqlen_q, seqlen_q);
 372 |       max_seqlen_kv = std::max(max_seqlen_kv, seqlen_kv);
 373 | 
 374 |       cumulative_seqlen_q.push_back(cumulative_seqlen_q.back() + seqlen_q);
 375 |       cumulative_seqlen_kv.push_back(cumulative_seqlen_kv.back() + seqlen_kv);
 376 |     }
 377 | 
 378 |     ProblemShape problem_size_for_init = problem_size;
 379 |     get<0>(problem_size_for_init) = 1;
 380 |     get<3>(problem_size_for_init) = total_seqlen_q;
 381 |     get<4>(problem_size_for_init) = total_seqlen_kv;
 382 | 
 383 |     ProblemShapeType problem_size_for_launch;
 384 | 
 385 |     get<3>(problem_size_for_launch) = cutlass::fmha::collective::VariableLength{max_seqlen_q};
 386 |     get<4>(problem_size_for_launch) = cutlass::fmha::collective::VariableLength{max_seqlen_kv};
 387 |     get<5>(problem_size_for_launch) = get<5>(problem_size);
 388 |     get<6>(problem_size_for_launch) = get<6>(problem_size);
 389 |     get<0>(problem_size_for_launch) = get<0>(problem_size);
 390 |     get<1>(problem_size_for_launch) = get<1>(problem_size);
 391 |     get<2>(problem_size_for_launch) = get<2>(problem_size);
 392 | 
 393 |     return cute::make_tuple(problem_size_for_init, problem_size_for_launch);
 394 |   }
 395 | 
 396 |   /// Initialize operands to be used in the GEMM and reference GEMM
 397 |   ProblemShapeType initialize(const FMHAOptions &options) {
 398 |     auto problem_shape_in =
 399 |         cute::make_tuple(options.batch, options.num_heads_q, options.num_heads_kv, options.seq_len_qo, options.seq_len_kv, options.head_size_qk, options.head_size_vo);
 400 | 
 401 |     ProblemShapeType problem_shape;
 402 |     decltype(problem_shape_in) problem_size;
 403 | 
 404 |     if constexpr (isVarLen) {
 405 |       auto [problem_shape_init, problem_shape_launch] = initialize_varlen(problem_shape_in);
 406 |       problem_shape = problem_shape_launch;
 407 |       problem_size = problem_shape_init;
 408 |     }
 409 |     else {
 410 |       problem_size = problem_shape_in;
 411 |       problem_shape = problem_shape_in;
 412 |     }
 413 | 
 414 |     auto [batch, num_heads_q, num_heads_kv, seq_len_qo, seq_len_kv, head_size_qk, head_size_vo] = problem_size;
 415 | 
 416 |     stride_Q = cutlass::make_cute_packed_stride(StrideQ{}, cute::make_shape(seq_len_qo, head_size_qk, batch * num_heads_q));
 417 |     stride_K = cutlass::make_cute_packed_stride(StrideK{}, cute::make_shape(seq_len_kv, head_size_qk, batch * num_heads_kv));
 418 |     stride_V = cutlass::make_cute_packed_stride(StrideV{}, cute::make_shape(head_size_vo, seq_len_kv, batch * num_heads_kv));
 419 |     stride_O = cutlass::make_cute_packed_stride(StrideO{}, cute::make_shape(seq_len_qo, head_size_vo, batch * num_heads_q));
 420 | 
 421 |     std::size_t mem_size_q = static_cast<std::size_t>(batch) * num_heads_q * seq_len_qo * head_size_qk;
 422 |     std::size_t mem_size_k = static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv * head_size_qk;
 423 |     std::size_t mem_size_v = static_cast<std::size_t>(batch) * num_heads_kv * seq_len_kv * head_size_vo;
 424 |     std::size_t mem_size_o = static_cast<std::size_t>(batch) * num_heads_q * seq_len_qo * head_size_vo;
 425 | 
 426 |     std::size_t mem_occupied_QKV = (mem_size_q * sizeof(ElementQ)) + (mem_size_k * sizeof(ElementK)) + 
 427 |                                    (mem_size_v * sizeof(ElementV));
 428 | 
 429 |     count = std::ceil(static_cast<float>(cutlass::get_llc_size()) / static_cast<float>(mem_occupied_QKV)) + 1;
 430 | 
 431 |     for(int i = 0; i < count; i++) {
 432 |       block_Q.emplace_back();
 433 |       block_K.emplace_back();
 434 |       block_V.emplace_back();      
 435 |     }
 436 | 
 437 |     
 438 |     for(int i = 0; i < count; i++) {
 439 |       block_Q[i].reset(mem_size_q);
 440 |       block_K[i].reset(mem_size_k);
 441 |       block_V[i].reset(mem_size_v);
 442 | 
 443 |       initialize_block(block_Q[i], seed + i);
 444 |       initialize_block(block_K[i], seed + i);
 445 |       initialize_block(block_V[i], seed + i);
 446 |     }
 447 | 
 448 |     block_O.reset(mem_size_o);
 449 |     block_ref_O.reset(mem_size_o);
 450 | 
 451 |     if (!cumulative_seqlen_q.empty()) {
 452 |       device_cumulative_seqlen_q.reset(cumulative_seqlen_q.size());
 453 |       device_cumulative_seqlen_q.copy_from_host(
 454 |         cumulative_seqlen_q.data(), cumulative_seqlen_q.size());
 455 |     }
 456 |     if (!cumulative_seqlen_kv.empty()) {
 457 |       device_cumulative_seqlen_kv.reset(cumulative_seqlen_kv.size());
 458 |       device_cumulative_seqlen_kv.copy_from_host(
 459 |         cumulative_seqlen_kv.data(), cumulative_seqlen_kv.size());
 460 |     }
 461 | 
 462 |     if constexpr (isVarLen) {
 463 |       get<3>(problem_shape).cumulative_length = device_cumulative_seqlen_q.get();
 464 |       get<4>(problem_shape).cumulative_length = device_cumulative_seqlen_kv.get();
 465 |     }
 466 | 
 467 |     return problem_shape;
 468 |   }
```
**EN:** Builds variable-length metadata when requested, allocates tensors, initializes inputs, and uploads cumulative-length arrays to device memory.
**CN:** 在需要时构建变长元数据，分配张量，初始化输入，并把累计长度数组上传到设备内存。

### Lines 470-509 — Launch wrapper
```cpp
 470 |   static void run(typename GemmKernel::Params params) {
 471 |     dim3 const block = GemmKernel::get_block_shape();
 472 |     dim3 const grid = GemmKernel::get_grid_shape(params);
 473 | 
 474 |     // configure smem size and carveout
 475 |     int smem_size = GemmKernel::SharedStorageSize;
 476 | 
 477 |     const auto sycl_block = compat::dim3(block.x, block.y, block.z);
 478 |     const auto sycl_grid = compat::dim3(grid.x, grid.y, grid.z);
 479 | 
 480 | #if !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)
 481 |     using namespace compat::experimental;
 482 |     #if defined(CUTLASS_SYCL_PROFILING_ENABLED)
 483 |     auto event = launch<cutlass::device_kernel<GemmKernel>>(
 484 |         launch_policy{sycl_grid, sycl_block, local_mem_size{static_cast<std::size_t>(smem_size)},
 485 |                       kernel_properties{sycl_exp::sub_group_size<GemmKernel::DispatchPolicy::SubgroupSize>}},
 486 |         params);
 487 |     EventManager::getInstance().addEvent(event);
 488 |     #else
 489 |     launch<cutlass::device_kernel<GemmKernel>, sycl::detail::auto_name, false>(
 490 |         launch_policy{sycl_grid, sycl_block, local_mem_size{static_cast<std::size_t>(smem_size)},
 491 |                       kernel_properties{sycl_exp::sub_group_size<GemmKernel::DispatchPolicy::SubgroupSize>}},
 492 |         params);
 493 |     #endif
 494 | #else
 495 |     compat::experimental::launch_properties launch_props{
 496 |       sycl::ext::oneapi::experimental::work_group_scratch_size(smem_size)
 497 |     };
 498 |     compat::experimental::kernel_properties kernel_props{
 499 |       sycl::ext::oneapi::experimental::sub_group_size<GemmKernel::DispatchPolicy::SubgroupSize>
 500 |     };
 501 |     compat::experimental::launch_policy policy{sycl_grid, sycl_block, launch_props, kernel_props};
 502 |     #if defined(CUTLASS_SYCL_PROFILING_ENABLED)
 503 |     auto event = compat::experimental::launch<cutlass::device_kernel<GemmKernel>, GemmKernel>(policy, params);
 504 |     EventManager::getInstance().addEvent(event);
 505 |     #else
 506 |         compat::experimental::launch<cutlass::device_kernel<GemmKernel>, GemmKernel, false>(policy, params);
 507 |     #endif
 508 | #endif
 509 |   }
```
**EN:** Converts block and grid information into a SYCL launch, including scratch-memory setup and optional profiling event capture.
**CN:** 把 block 与 grid 信息转换为 SYCL 启动过程，包括 scratch memory 设置和可选的 profiling 事件记录。

### Lines 511-623 — Benchmark execution
```cpp
 511 |   void run(::benchmark::State& state, const FMHAOptions &options, const cutlass::KernelHardwareInfo &hw_info) {
 512 | 
 513 |     ProblemShapeType problem_size = initialize(options);
 514 | 
 515 |     typename GemmKernel::Arguments arguments{
 516 |         cutlass::gemm::GemmUniversalMode::kGemm,
 517 |         problem_size,
 518 |         {block_Q[0].get(), stride_Q, block_K[0].get(), stride_K, block_V[0].get(), stride_V},
 519 |         {options.softmax_scale},
 520 |         {block_O.get(), stride_O},
 521 |         hw_info};
 522 | 
 523 |     // GemmKernel gemm_op;
 524 | 
 525 |     size_t workspace_size = GemmKernel::get_workspace_size(arguments);
 526 |     cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
 527 | 
 528 |     GemmKernel::can_implement(arguments);
 529 | 
 530 |     // Initialize the workspace
 531 |     auto status = GemmKernel::initialize_workspace(arguments, workspace.get());
 532 |     if (status != cutlass::Status::kSuccess) {
 533 |       return;
 534 |     }
 535 | 
 536 |     typename GemmKernel::Params params = GemmKernel::to_underlying_arguments(arguments, workspace.get());
 537 | 
 538 |     // Run the GEMM
 539 |     run(params);
 540 | 
 541 |     compat::wait();
 542 | 
 543 |     // Verify that the result is correct
 544 |     bool passed = verify(problem_size);
 545 |     if(not passed) {
 546 |       state.SkipWithError("Disposition Failed.");
 547 |     }
 548 | 
 549 |     state.counters["batch"] = options.batch;
 550 |     state.counters["num_heads_q"] = options.num_heads_q;
 551 |     state.counters["num_heads_kv"] = options.num_heads_kv;
 552 |     state.counters["seq_len_qo"] = options.seq_len_qo;
 553 |     state.counters["seq_len_kv"] = options.seq_len_kv;
 554 |     state.counters["head_size_kv"] = options.head_size_qk;
 555 |     state.counters["head_size_vo"] = options.head_size_vo;
 556 |     state.counters["scale"] = options.softmax_scale;
 557 |     state.counters["causal"] = Causal;
 558 |     state.counters["varlen"] = isVarLen;
 559 | 
 560 |     std::stringstream extra_label;
 561 |     extra_label << "layoutQ=RowMajor ";
 562 |     extra_label << "layoutK=ColumnMajor ";
 563 |     extra_label << "layoutV=RowMajor ";
 564 | 
 565 |     state.SetLabel(extra_label.str());
 566 |     // when seq_len_qo is not equal to seq_len_kv we use bottom up approach for the masking. 
 567 |     // Following changes will adjust the effective_seq_len_kv when masking applied for such cases. 
 568 |     auto offset = cute::min(options.seq_len_qo, options.seq_len_kv);
 569 |     auto discard_seq_coord = options.seq_len_qo - offset;
 570 |     auto full_tile_offset = options.seq_len_kv - offset;
 571 |     auto effective_seq_len_kv = Causal ? full_tile_offset + ((offset + 1) / 2.0): options.seq_len_kv;
 572 |     auto effective_seq_len_qo = Causal ? options.seq_len_qo - discard_seq_coord  : options.seq_len_qo;
 573 |    
 574 |     double flops_qk = 2.0 * options.batch * options.num_heads_q * effective_seq_len_qo * effective_seq_len_kv * options.head_size_qk;
 575 |     double flops_pv = 2.0 * options.batch * options.num_heads_q * effective_seq_len_qo * options.head_size_vo * effective_seq_len_kv;
 576 |     double gflops = (flops_qk + flops_pv) * 1e-9;
 577 | 
 578 |     // TODO: Use sizeof_bits_v instead of sizeof if QKVO is smaller than 8 bits, which avoids incorrect bandwidth calculation
 579 |     double gbps_qk =  options.batch * (sizeof(ElementQ) * options.num_heads_q * effective_seq_len_qo * options.head_size_qk + 
 580 |                       sizeof(ElementK) * options.num_heads_kv * effective_seq_len_kv * options.head_size_qk);    
 581 |     double gbps_pv = sizeof(ElementV) * options.batch * options.num_heads_kv * effective_seq_len_kv * options.head_size_vo +
 582 |                      sizeof(ElementOutput) * options.batch * options.num_heads_q * effective_seq_len_qo * options.head_size_vo;
 583 |     double mega_bytes_transferred = (gbps_qk + gbps_pv) * (1e-6);
 584 | 
 585 |     initialize_counters(state);
 586 |     int32_t counter = 1;
 587 |     for(auto _ : state) {
 588 |       state.PauseTiming();
 589 |       int input_num = std::max(int(0), counter % count);
 590 | 
 591 |       typename GemmKernel::Arguments arguments{
 592 |           cutlass::gemm::GemmUniversalMode::kGemm,
 593 |           problem_size,
 594 |           {block_Q[input_num].get(), stride_Q, block_K[input_num].get(), stride_K, block_V[input_num].get(), stride_V},
 595 |           {options.softmax_scale},
 596 |           {block_O.get(), stride_O},
 597 |           hw_info};
 598 | 
 599 |       size_t workspace_size = GemmKernel::get_workspace_size(arguments);
 600 |       cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
 601 | 
 602 |       GemmKernel::can_implement(arguments);
 603 | 
 604 |       // Initialize the workspace
 605 |       auto status = GemmKernel::initialize_workspace(arguments, workspace.get());
 606 |       if (status != cutlass::Status::kSuccess) {
 607 |         return;
 608 |       }
 609 | 
 610 |       typename GemmKernel::Params params = GemmKernel::to_underlying_arguments(arguments, workspace.get());
 611 | 
 612 |       state.ResumeTiming();
 613 | 
 614 |       GPU_Clock timer;
 615 |       timer.start();
 616 |       run(params);
 617 |       auto ms_elapsed = timer.milliseconds();
 618 |       update_counters(state, ms_elapsed);
 619 |       state.SetIterationTime(ms_elapsed / 1000);
 620 |       counter++;
 621 |     }
 622 |     finalize_counters(state, gflops, mega_bytes_transferred);
 623 |   }
```
**EN:** Builds kernel arguments, performs a validation run, fills benchmark labels and counters, and executes the timed benchmark loop.
**CN:** 构建内核参数，执行一次校验运行，填充基准标签和计数器，并执行计时循环。

### Lines 625-645 — Counter helpers
```cpp
 625 | private:
 626 |   static void initialize_counters(::benchmark::State& state) {
 627 |     state.counters["avg_runtime_ms"] = 0;
 628 |     state.counters["best_runtime_ms"] = std::numeric_limits<double>::max();
 629 |   }
 630 | 
 631 |   static void update_counters(::benchmark::State& state, double ms_elapsed) {
 632 |     state.PauseTiming();
 633 |     state.counters["total_runtime_ms"] += ms_elapsed;
 634 |     state.counters["best_runtime_ms"] = std::min<double>(state.counters["best_runtime_ms"], ms_elapsed);
 635 |     state.ResumeTiming();
 636 |   }
 637 | 
 638 |   static void finalize_counters(::benchmark::State& state,  double gflop, double mega_bytes_transferred) {
 639 |     state.counters["avg_runtime_ms"] =
 640 |       state.counters["total_runtime_ms"] / static_cast<double>(state.iterations());
 641 |     state.counters["avg_tflops"] = gflop / state.counters["avg_runtime_ms"];
 642 |     state.counters["avg_throughput"] = mega_bytes_transferred / state.counters["avg_runtime_ms"];
 643 |     state.counters["best_tflop"] = gflop / state.counters["best_runtime_ms"];
 644 |     state.counters["best_bandwidth"] = mega_bytes_transferred / state.counters["best_runtime_ms"];
 645 |   }
```
**EN:** Aggregates total, average, and best runtime values for later TFLOP/s and bandwidth computation.
**CN:** 聚合总运行时间、平均运行时间和最佳运行时间，以便后续计算 TFLOP/s 与带宽。

### Lines 650-659 — Registration macros
```cpp
 650 | #define CUTLASS_FMHA_PREFILL_BENCHMARK(F) cutlass::benchmark::BenchmarkRegistry<cutlass::benchmark::FMHAOptions>::Register(#F, &F##_func)
 651 | 
 652 | #define CUTLASS_CREATE_FMHA_PREFILL_BENCHMARK(F)                          \
 653 |   static void F##_func(                                           \
 654 |       ::benchmark::State& state,                                  \
 655 |       cutlass::benchmark::FMHAOptions const& options,                 \
 656 |       cutlass::KernelHardwareInfo const& hw_info) {               \
 657 |     auto bench = cutlass::benchmark::BenchmarkRunnerFMHA<F>();    \
 658 |     bench.run(state, options, hw_info);                           \
 659 |   }
```
**EN:** Exposes macro helpers that wrap a configuration type and register it with the benchmark registry.
**CN:** 提供宏辅助函数，用于包装配置类型并将其注册到基准注册表。

## Key Concepts / 关键概念

- Prefill CLI parsing / prefill 命令行解析
- Reference softmax validation / 参考 softmax 校验
- Variable-length setup / 变长设置
- Benchmark timing loop / 基准计时循环

## Dependencies / 依赖关系

- Legacy prefill kernels and epilogues / 旧版 prefill 内核与 epilogue
- `cutlass::DeviceAllocation` — device buffers / 设备缓冲区
- `GPU_Clock` and `benchmark::State` — timing / 计时
