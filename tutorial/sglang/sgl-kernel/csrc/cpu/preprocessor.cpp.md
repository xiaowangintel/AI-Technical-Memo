# preprocessor.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/preprocessor.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Local implementation details
```cpp
/*****************************************************************************************
 * Copyright (c) 2025 - 2025 Codeplay Software Ltd. All rights reserved.
 * Copyright (C) 2025 Intel Corporation, All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 25-45: Namespace and shared declarations
```cpp
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 ****************************************************************************************/
#include "common.h"
#include "vec.h"

// [NOTE] Preprocessor Optimization
//   1. this file is apple-to-apple to `Qwen2VLImageProcessorFast`.
//   2. `out_dtype` set to torch.bfloat16 skips outplace dtype conversion.
//   3. skip all redundant memory copy and dtype conversion.
//   4. TODO: rewrite `_upsample_bicubic2d_aa`.
//
//   ref: https://github.com/huggingface/transformers/blob/main/src/transformers
//       /models/qwen2_vl/image_processing_qwen2_vl_fast.py
//
namespace {
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 46-69: Runtime integration and dispatch
```cpp
template <typename scalar_t>
inline void normalize(
    scalar_t* __restrict__ out,
    const uint8_t* __restrict__ input,
    const std::vector<float>& image_mean,
    const std::vector<float>& image_std,
    int64_t channel,
    int64_t temporal_patch_size,
    int64_t patch_size,
    int64_t stride_ch,
    int64_t stride_pt,
    int64_t stride_ph) {
  TORCH_CHECK(false, "normalize: scalar path not implemented.");
}

#if defined(CPU_CAPABILITY_AVX512)
template <>
inline void normalize<float>(
    float* __restrict__ out,
    const uint8_t* __restrict__ input,
    const std::vector<float>& image_mean,
    const std::vector<float>& image_std,
    int64_t channel,
    int64_t temporal_patch_size,
```
**EN:** This section uses `normalize`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`normalize`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 70-93: Control flow and branching
```cpp
    int64_t patch_size,
    int64_t stride_ch,
    int64_t stride_pt,
    int64_t stride_ph) {
  // we do vectorization on patch_size dim
  assert(patch_size == 16);

  // loop last 4 dimensions:
  //  {channel, patch_t(repeated), patch_h, patch_w}
  for (int64_t c = 0; c < channel; ++c) {
    __m512 vmean = _mm512_set1_ps(image_mean[c]);
    __m512 vrstd = _mm512_set1_ps(1.f / image_std[c]);

    float* __restrict__ out_ptr = out + c * temporal_patch_size * patch_size * patch_size;
#pragma GCC unroll 4
    for (int64_t ph = 0; ph < patch_size; ++ph) {
      __m128i u8 = _mm_loadu_si128((const __m128i*)(input + c * stride_ch + /* pt */ 0 * stride_pt + ph * stride_ph));
      __m512 x = _mm512_cvtepi32_ps(_mm512_cvtepu8_epi32(u8));
      x = _mm512_mul_ps(_mm512_sub_ps(x, vmean), vrstd);
#pragma GCC unroll 2
      for (int64_t pt = 0; pt < temporal_patch_size; ++pt) {
        _mm512_storeu_ps(out_ptr + pt * patch_size * patch_size + ph * patch_size, x);
      }
    }
```
**EN:** This section drives `assert`, `_mm512_set1_ps`, `_mm_loadu_si128` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`assert`、`_mm512_set1_ps`、`_mm_loadu_si128`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 94-117: Runtime integration and dispatch
```cpp
  }
}

template <>
inline void normalize<at::BFloat16>(
    at::BFloat16* __restrict__ out,
    const uint8_t* __restrict__ input,
    const std::vector<float>& image_mean,
    const std::vector<float>& image_std,
    int64_t channel,
    int64_t temporal_patch_size,
    int64_t patch_size,
    int64_t stride_ch,
    int64_t stride_pt,
    int64_t stride_ph) {
  // we do vectorization on patch_size dim
  assert(patch_size == 16);

  // loop last 4 dimensions:
  //  {channel, patch_t(repeated), patch_h, patch_w}
  for (int64_t c = 0; c < channel; ++c) {
    __m512 vmean = _mm512_set1_ps(image_mean[c]);
    __m512 vrstd = _mm512_set1_ps(1.f / image_std[c]);
```
**EN:** This section uses `assert`, `_mm512_set1_ps` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`assert`、`_mm512_set1_ps`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 118-141: Runtime integration and dispatch
```cpp
    at::BFloat16* __restrict__ out_ptr = out + c * temporal_patch_size * patch_size * patch_size;
#pragma GCC unroll 4
    for (int64_t ph = 0; ph < patch_size; ++ph) {
      __m128i u8 = _mm_loadu_si128((const __m128i*)(input + c * stride_ch + /* pt */ 0 * stride_pt + ph * stride_ph));
      __m512 x = _mm512_cvtepi32_ps(_mm512_cvtepu8_epi32(u8));
      x = _mm512_mul_ps(_mm512_sub_ps(x, vmean), vrstd);
      __m256i x16 = (__m256i)_mm512_cvtneps_pbh(x);
#pragma GCC unroll 2
      for (int64_t pt = 0; pt < temporal_patch_size; ++pt) {
        _mm256_storeu_si256(reinterpret_cast<__m256i*>(out_ptr + pt * patch_size * patch_size + ph * patch_size), x16);
      }
    }
  }
}
#endif

template <typename scalar_t>
void rescale_and_normalize_kernel_impl(
    scalar_t* __restrict__ out,
    const uint8_t* __restrict__ input,
    const std::vector<float>& image_mean,
    const std::vector<float>& image_std,
    int64_t grid_t,
    int64_t grid_h,
```
**EN:** This section uses `_mm_loadu_si128`, `_mm512_cvtepi32_ps`, `_mm512_mul_ps` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_mm_loadu_si128`、`_mm512_cvtepi32_ps`、`_mm512_mul_ps`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 142-164: Local implementation details
```cpp
    int64_t grid_w,
    int64_t merge_size,
    int64_t channel,
    int64_t temporal_patch_size,
    int64_t patch_size) {
  // [NOTE]: temporal patching uses repeat on last image
  //
  //  input : {grid_t, patch_t, channel,  grid_h, merge_h, patch_h,  grid_w, merge_w, patch_w}
  //    out : {grid_t,  grid_h,  grid_w, merge_h, merge_w, channel, patch_t, patch_h, patch_w}
  //
  int64_t height = grid_h * merge_size * patch_size;
  int64_t width = grid_w * merge_size * patch_size;

  int64_t stride_gt = /* temporal_patch_size */ 1 * channel * height * width;
  int64_t stride_gh = merge_size * patch_size * width;
  int64_t stride_gw = merge_size * patch_size;
  int64_t stride_mh = patch_size * width;
  int64_t stride_mw = patch_size;
  int64_t stride_ch = height * width;
  int64_t stride_pt = channel * height * width;
  int64_t stride_ph = width;
  int64_t stride_grid = channel * temporal_patch_size * patch_size * patch_size;
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 165-188: Runtime integration and dispatch
```cpp
  // parallel on first 5 dims, aka, grids
  at::parallel_for(0, grid_t * grid_h * grid_w * merge_size * merge_size, 0, [&](int64_t begin, int64_t end) {
    int64_t gt{0}, gh{0}, gw{0}, mh{0}, mw{0};
    data_index_init(begin, gt, grid_t, gh, grid_h, gw, grid_w, mh, merge_size, mw, merge_size);

    for (int64_t i = begin; i < end; ++i) {
      normalize<scalar_t>(
          out + i * stride_grid,
          input + gt * stride_gt + gh * stride_gh + gw * stride_gw + mh * stride_mh + mw * stride_mw,
          image_mean,
          image_std,
          channel,
          temporal_patch_size,
          patch_size,
          stride_ch,
          stride_pt,
          stride_ph);

      // move to the next index
      data_index_step(gt, grid_t, gh, grid_h, gw, grid_w, mh, merge_size, mw, merge_size);
    }
  });
}
```
**EN:** This section uses `at::parallel_for`, `data_index_init`, `data_index_step` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`data_index_init`、`data_index_step`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 189-212: Runtime integration and dispatch
```cpp
}  // anonymous namespace

void check_input_image(const at::Tensor& image) {
  TORCH_CHECK(image.scalar_type() == at::kByte, "expect image to be uint8.");
  TORCH_CHECK(image.dim() == 3, "expect image to be CHW.");
}

// https://github.com/huggingface/transformers/blob/main/src/transformers/models/qwen2_vl/image_processing_qwen2_vl.py
std::pair<int64_t, int64_t>
smart_resize(int64_t height, int64_t width, int64_t factor, int64_t min_pixels, int64_t max_pixels) {
  // aspect ratio check
  int64_t mx = std::max(height, width);
  int64_t mn = std::min(height, width);

  TORCH_CHECK(static_cast<double>(mx) / mn <= 200.0, "absolute aspect ratio must be smaller than 200");

  // round to nearest multiple of factor
  auto round_to_factor = [&](int64_t x) {
    return static_cast<int64_t>(std::round(static_cast<double>(x) / factor)) * factor;
  };

  int64_t h_bar = round_to_factor(height);
  int64_t w_bar = round_to_factor(width);
```
**EN:** This section uses `check_input_image`, `smart_resize`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`check_input_image`、`smart_resize`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 213-236: Runtime integration and dispatch
```cpp
  int64_t area = h_bar * w_bar;

  if (area > max_pixels) {
    double beta = std::sqrt((1.0 * height * width) / max_pixels);
    h_bar = std::max(factor, (static_cast<int64_t>(std::floor(height / beta / factor)) * factor));
    w_bar = std::max(factor, (static_cast<int64_t>(std::floor(width / beta / factor)) * factor));
  } else if (area < min_pixels) {
    double beta = std::sqrt((double)min_pixels / (height * width));
    h_bar = static_cast<int64_t>(std::ceil(height * beta / factor)) * factor;
    w_bar = static_cast<int64_t>(std::ceil(width * beta / factor)) * factor;
  }

  return {h_bar, w_bar};
}

// do rescale and normalize
// from `resized_image` to `pixel_values`
void rescale_and_normalize_image(
    at::Tensor& pixel_values,
    const at::Tensor& image,
    double rescale_factor,
    c10::ArrayRef<double> image_mean,
    c10::ArrayRef<double> image_std,
    int64_t grid_t,
```
**EN:** This section uses `sqrt`, `max` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sqrt`、`max`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 237-266: Runtime integration and dispatch
```cpp
    int64_t grid_h,
    int64_t grid_w,
    int64_t merge_size,
    int64_t channel,
    int64_t temporal_patch_size,
    int64_t patch_size,
    int64_t grid_offset,
    int64_t grid_stride) {
  // update mean and std
  std::vector<float> mean_vec(channel), std_vec(channel);
  for (int64_t c = 0; c < channel; ++c) {
    mean_vec[c] = static_cast<float>(image_mean[c] * (1 / rescale_factor));
    std_vec[c] = static_cast<float>(image_std[c] * (1 / rescale_factor));
  }

  AT_DISPATCH_FLOATING_TYPES_AND(at::kBFloat16, pixel_values.scalar_type(), "rescale_and_normalize_image", [&] {
    rescale_and_normalize_kernel_impl<scalar_t>(
        pixel_values.data_ptr<scalar_t>() + grid_offset * grid_stride,
        image.data_ptr<uint8_t>(),
        mean_vec,
        std_vec,
        grid_t,
        grid_h / merge_size,
        grid_w / merge_size,
        merge_size,
        channel,
        temporal_patch_size,
        patch_size);
  });
}
```
**EN:** This section uses `mean_vec` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`mean_vec`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 267-289: Runtime integration and dispatch
```cpp

std::tuple<at::Tensor, at::Tensor> image_preprocess_cpu(
    at::TensorList images,
    bool do_convert_rgb,
    bool do_resize,
    int64_t shortest_edge,
    int64_t longest_edge,
    const std::string& interpolation,
    bool do_rescale,
    double rescale_factor,
    bool do_normalize,
    c10::ArrayRef<double> image_mean,
    c10::ArrayRef<double> image_std,
    int64_t patch_size,
    int64_t temporal_patch_size,
    int64_t merge_size,
    bool disable_grouping,
    at::ScalarType out_dtype) {
  // TODO: lift C++ kernel limitations
  TORCH_CHECK(interpolation == "bicubic", "image_preprocess_cpu: support only bicubic mode.");
  TORCH_CHECK(do_rescale && do_normalize, "image_preprocess_cpu: support only do_rescale and do_normalize.");
  TORCH_CHECK(disable_grouping, "image_preprocess_cpu: support only disable_grouping.");
```
**EN:** This section uses `image_preprocess_cpu`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`image_preprocess_cpu`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 290-312: Runtime integration and dispatch
```cpp
  // support only float32 or bfloat16 as output
  TORCH_CHECK(
      out_dtype == at::kFloat || out_dtype == at::kBFloat16,
      "image_preprocess_cpu: support only float32 and bfloat16 as pixel_values dtype.");

  int64_t batch_size = images.size();
  int64_t channel = image_mean.size();
  CHECK_GT(batch_size, 0);
  CHECK_EQ(channel, image_std.size());
  CHECK_EQ(channel, 3);

  const at::Tensor& first_image = images[0];
  const auto options = first_image.options();
  at::Tensor pixel_values = at::empty({}, options.dtype(out_dtype));
  at::Tensor image_grid_thw = at::empty({batch_size, channel}, options.dtype(at::kLong));

  // index type use int64_t
  int64_t* image_grid_thw_data = image_grid_thw.data_ptr<int64_t>();

  // resized image sizes and global grid offset
  std::vector<std::pair<int64_t, int64_t>> image_sizes(batch_size);
  std::vector<int64_t> grid_offsets(batch_size + 1, 0);
```
**EN:** This section uses `TORCH_CHECK`, `CHECK_GT`, `CHECK_EQ` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`CHECK_GT`、`CHECK_EQ`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 313-336: Control flow and branching
```cpp
  // Stage 1: compute resized shapes and fill in `image_grid_thw`
  for (int64_t idx = 0; idx < batch_size; ++idx) {
    const auto& image = images[idx];
    check_input_image(image);

    auto [resized_h, resized_w] =
        smart_resize(image.size(-2), image.size(-1), patch_size * merge_size, shortest_edge, longest_edge);

    image_sizes[idx] = {resized_h, resized_w};

    // temporal dimension for image is 1
    int64_t grid_t = div_up((int64_t)1, temporal_patch_size);
    int64_t grid_h = div_up(resized_h, patch_size);
    int64_t grid_w = div_up(resized_w, patch_size);

    // fill in image_grid_thw
    image_grid_thw_data[idx * 3 + 0] = grid_t;
    image_grid_thw_data[idx * 3 + 1] = grid_h;
    image_grid_thw_data[idx * 3 + 2] = grid_w;

    // fill in global grid offset
    grid_offsets[idx + 1] = grid_offsets[idx] + grid_t * grid_h * grid_w;
  }
```
**EN:** This section drives `check_input_image`, `smart_resize`, `div_up` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`check_input_image`、`smart_resize`、`div_up`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 337-360: Runtime integration and dispatch
```cpp
  // last element holds the total sum of grids
  int64_t grid_size = grid_offsets[batch_size];
  int64_t grid_stride = channel * temporal_patch_size * patch_size * patch_size;
  // allocate memory
  pixel_values.resize_({grid_size, grid_stride});

  // Stage 2: compute `pixel_values`
  for (int64_t idx = 0; idx < batch_size; ++idx) {
    const auto& image = images[idx];
    int64_t resized_h = image_sizes[idx].first;
    int64_t resized_w = image_sizes[idx].second;
    auto resized_image = at::_upsample_bicubic2d_aa(
        image.unsqueeze(0),
        {resized_h, resized_w},
        /* align_corners */ false);

    rescale_and_normalize_image(
        pixel_values,
        resized_image,
        rescale_factor,
        image_mean,
        image_std,
        /* grid_t */ image_grid_thw_data[idx * 3 + 0],
        /* grid_h */ image_grid_thw_data[idx * 3 + 1],
```
**EN:** This section uses `image` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`image`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 361-371: Local implementation details
```cpp
        /* grid_w */ image_grid_thw_data[idx * 3 + 2],
        merge_size,
        channel,
        temporal_patch_size,
        patch_size,
        grid_offsets[idx],
        grid_stride);
  }

  return std::make_tuple(pixel_values, image_grid_thw);
}
```
**EN:** This section fills in the local implementation details around `make_tuple`, completing the behavior required by the file.
**CN:** 本段补充了`make_tuple`周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **Validation / 校验测试**: Contains checks or scaffolding for correctness verification. / 包含正确性验证所需的检查或脚手架。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`, `vec.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / preprocessor.cpp
