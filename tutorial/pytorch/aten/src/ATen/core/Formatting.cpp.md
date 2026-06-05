# Formatting.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/Formatting.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements core ATen/C10 abstractions, with primary focus on `FormatType`, `PrintFormat`, `c10`.
- 用途（中文）: 该文件实现ATen/C10 核心抽象，核心关注对象是 `FormatType`, `PrintFormat`, `c10`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
#include <ATen/core/Formatting.h>
#include <c10/util/irange.h>
#include <fmt/compile.h>
#include <fmt/format.h>
#include <fmt/ostream.h>

#include <cmath>
#include <cstdint>
#include <iostream>
#include <iterator>
#include <string>

```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 13-22
```cpp
namespace c10 {
std::ostream& operator<<(std::ostream& out, Backend b) {
  return out << toString(b);
}

std::ostream& operator<<(std::ostream& out, const Scalar& s) {
  if (s.isFloatingPoint()) {
    return out << s.toDouble();
  }
  if (s.isComplex()) {
```
- EN: Focus symbols: `c10`, `toString`, `isFloatingPoint`, `toDouble`, `isComplex`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`c10`, `toString`, `isFloatingPoint`, `toDouble`, `isComplex`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 23-32
```cpp
    return out << s.toComplexDouble();
  }
  if (s.isBoolean()) {
    return out << (s.toBool() ? "true" : "false");
  }
  if (s.isSymInt()) {
    return out << (s.toSymInt());
  }
  if (s.isSymFloat()) {
    return out << (s.toSymFloat());
```
- EN: Focus symbols: `toComplexDouble`, `isBoolean`, `toBool`, `isSymInt`, `toSymInt`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`toComplexDouble`, `isBoolean`, `toBool`, `isSymInt`, `toSymInt`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 33-44
```cpp
  }
  if (s.isIntegral(false)) {
    return out << s.toLong();
  }
  throw std::logic_error("Unknown type in Scalar");
}

std::string toString(const Scalar& s) {
  return fmt::format("{}", fmt::streamed(s));
}
} // namespace c10

```
- EN: Focus symbols: `c10`, `isIntegral`, `toLong`, `logic_error`, `toString`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`c10`, `isIntegral`, `toLong`, `logic_error`, `toString`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 45-56
```cpp
namespace at {

std::ostream& operator<<(std::ostream& out, const DeprecatedTypeProperties& t) {
  return out << t.toString();
}

enum class FormatType {
  Default, // 'g' format (defaultfloat equivalent)
  Scientific, // 'e' format with precision 4
  Fixed // 'f' format with precision 4
};

```
- EN: Focus symbols: `FormatType`, `at`, `toString`, `format`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`FormatType`, `at`, `toString`, `format`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 57-66
```cpp
struct PrintFormat {
  double scale;
  int width;
  FormatType type;

  PrintFormat(double s, int w, FormatType t = FormatType::Default)
      : scale(s), width(w), type(t) {}
};

static PrintFormat __printFormat(const Tensor& self) {
```
- EN: Focus symbols: `PrintFormat`, `scale`, `width`, `type`, `__printFormat`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`PrintFormat`, `scale`, `width`, `type`, `__printFormat`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 67-76
```cpp
  auto size = self.numel();
  if (size == 0) {
    return PrintFormat(1., 0);
  }

  bool intMode = true;
  auto self_p = self.const_data_ptr<double>();
  for (const auto i : c10::irange(size)) {
    auto z = self_p[i];
    if (std::isfinite(z)) {
```
- EN: Focus symbols: `numel`, `PrintFormat`, `irange`, `isfinite`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`numel`, `PrintFormat`, `irange`, `isfinite`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 77-88
```cpp
      if (z != std::ceil(z)) {
        intMode = false;
        break;
      }
    }
  }

  int64_t offset = 0;
  while (offset < size && !std::isfinite(self_p[offset])) {
    offset = offset + 1;
  }

```
- EN: Focus symbols: `ceil`, `isfinite`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`ceil`, `isfinite`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 89-98
```cpp
  double expMin = 1;
  double expMax = 1;
  if (offset != size) {
    expMin = std::fabs(self_p[offset]);
    expMax = std::fabs(self_p[offset]);
    for (const auto i : c10::irange(offset, size)) {
      double z = std::fabs(self_p[i]);
      if (std::isfinite(z)) {
        expMin = std::min(expMin, z);
        expMax = std::max(expMax, z);
```
- EN: Focus symbols: `fabs`, `irange`, `isfinite`, `min`, `max`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`fabs`, `irange`, `isfinite`, `min`, `max`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 99-112
```cpp
      }
    }
    if (expMin != 0) {
      expMin = std::floor(std::log10(expMin)) + 1;
    } else {
      expMin = 1;
    }
    if (expMax != 0) {
      expMax = std::floor(std::log10(expMax)) + 1;
    } else {
      expMax = 1;
    }
  }

```
- EN: Focus symbols: `floor`, `log10`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`floor`, `log10`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 113-122
```cpp
  double scale = 1;
  int sz = 11;

  if (intMode) {
    if (expMax > 9) {
      sz = 11;
      return PrintFormat(scale, sz, FormatType::Scientific);
    } else {
      sz = static_cast<int>(expMax) + 1;
      return PrintFormat(scale, sz, FormatType::Default);
```
- EN: Focus symbols: `PrintFormat`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`PrintFormat`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 123-132
```cpp
    }
  } else {
    if (expMax - expMin > 4) {
      sz = 11;
      if (std::fabs(expMax) > 99 || std::fabs(expMin) > 99) {
        sz = sz + 1;
      }
      return PrintFormat(scale, sz, FormatType::Scientific);
    } else {
      if (expMax > 5 || expMax < 0) {
```
- EN: Focus symbols: `fabs`, `PrintFormat`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`fabs`, `PrintFormat`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 133-142
```cpp
        sz = 7;
        scale = std::pow(10, expMax - 1);
        return PrintFormat(scale, sz, FormatType::Fixed);
      } else {
        if (expMax == 0) {
          sz = 7;
        } else {
          sz = static_cast<int>(expMax) + 6;
        }
        return PrintFormat(scale, sz, FormatType::Fixed);
```
- EN: Focus symbols: `pow`, `PrintFormat`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`pow`, `PrintFormat`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 143-152
```cpp
      }
    }
  }
}

// Precompiled format specs
static constexpr auto FMT_G = FMT_COMPILE("{:>{}g}");
static constexpr auto FMT_E4 = FMT_COMPILE("{:>{}.4e}");
static constexpr auto FMT_F4 = FMT_COMPILE("{:>{}.4f}");

```
- EN: Focus symbols: `FMT_COMPILE`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`FMT_COMPILE`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 153-162
```cpp
// Print a single value directly into the stream buffer with no temporaries
static void printValue(std::ostream& stream, double v, const PrintFormat& pf) {
  auto out_it = std::ostreambuf_iterator<char>(stream);
  double val = v / pf.scale;
  switch (pf.type) {
    case FormatType::Default:
      fmt::format_to(out_it, FMT_G, val, pf.width);
      break;
    case FormatType::Scientific:
      fmt::format_to(out_it, FMT_E4, val, pf.width);
```
- EN: Focus symbols: `printValue`, `format_to`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`printValue`, `format_to`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 163-176
```cpp
      break;
    case FormatType::Fixed:
      fmt::format_to(out_it, FMT_F4, val, pf.width);
      break;
  }
}

static void __printMatrix(
    std::ostream& stream,
    const Tensor& self,
    int64_t linesize,
    int64_t indent) {
  auto printFmt = __printFormat(self);

```
- EN: Focus symbols: `format_to`, `__printMatrix`, `__printFormat`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`format_to`, `__printMatrix`, `__printFormat`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 177-187
```cpp
  int64_t nColumnPerLine = (linesize - indent) / (printFmt.width + 1);
  int64_t firstColumn = 0;
  int64_t lastColumn = -1;

  while (firstColumn < self.size(1)) {
    if (firstColumn + nColumnPerLine <= self.size(1)) {
      lastColumn = firstColumn + nColumnPerLine - 1;
    } else {
      lastColumn = self.size(1) - 1;
    }

```
- EN: Focus symbols: `size`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`size`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 188-201
```cpp
    if (nColumnPerLine < self.size(1)) {
      if (firstColumn != 0) {
        stream.put('\n');
      }
      fmt::print(
          stream,
          "Columns {} to {}{:>{}s}",
          firstColumn + 1,
          lastColumn + 1,
          "", // empty string to pad
          indent // width to pad to
      );
    }

```
- EN: Focus symbols: `size`, `put`, `print`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`size`, `put`, `print`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 202-212
```cpp
    if (printFmt.scale != 1) {
      fmt::print(stream, "{} *\n{:>{}s}", printFmt.scale, "", indent);
    }

    for (const auto l : c10::irange(self.size(0))) {
      Tensor row = self.select(0, l);
      const double* row_ptr = row.const_data_ptr<double>();

      for (const auto c : c10::irange(firstColumn, lastColumn + 1)) {
        printValue(stream, row_ptr[c], printFmt);

```
- EN: Focus symbols: `print`, `irange`, `size`, `select`, `printValue`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`print`, `irange`, `size`, `select`, `printValue`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 213-222
```cpp
        if (c == lastColumn) {
          stream.put('\n');
          if (l != self.size(0) - 1) {
            if (printFmt.scale != 1) {
              fmt::print(stream, "{:>{}s} ", "", indent);
            } else {
              fmt::print(stream, "{:>{}s}", "", indent);
            }
          }
        } else {
```
- EN: Focus symbols: `put`, `size`, `print`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`put`, `size`, `print`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 223-232
```cpp
          stream.put(' ');
        }
      }
    }
    firstColumn = lastColumn + 1;
  }
}

static void __printTensor(
    std::ostream& stream,
```
- EN: Focus symbols: `put`, `__printTensor`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`put`, `__printTensor`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 233-242
```cpp
    Tensor& self,
    int64_t linesize) {
  std::vector<int64_t> counter(self.ndimension() - 2, 0);
  counter[0] = -1;

  bool start = true;
  bool finished = false;

  while (true) {
    for (int64_t i = 0; self.ndimension() - 2; i++) {
```
- EN: Focus symbols: `counter`, `ndimension`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`counter`, `ndimension`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 243-252
```cpp
      counter[i] = counter[i] + 1;
      if (counter[i] >= self.size(i)) {
        if (i == self.ndimension() - 3) {
          finished = true;
          break;
        }
        counter[i] = 0;
      } else {
        break;
      }
```
- EN: Focus symbols: `size`, `ndimension`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`size`, `ndimension`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 253-262
```cpp
    }
    if (finished) {
      break;
    }
    if (start) {
      start = false;
    } else {
      stream.put('\n');
    }

```
- EN: Focus symbols: `put`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`put`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 263-273
```cpp
    stream.put('(');
    Tensor tensor = self;
    for (const auto i : c10::irange(self.ndimension() - 2)) {
      tensor = tensor.select(0, counter[i]);
      fmt::print(stream, "{},", counter[i] + 1);
    }
    fmt::print(stream, ".,.) = \n");
    __printMatrix(stream, tensor, linesize, 1);
  }
}

```
- EN: Focus symbols: `put`, `irange`, `ndimension`, `select`, `print`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`put`, `irange`, `ndimension`, `select`, `print`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 274-286
```cpp
void print(const Tensor& t, int64_t linesize) {
  print(std::cout, t, linesize);
}

std::ostream& print(
    std::ostream& stream,
    const Tensor& tensor_,
    int64_t linesize) {
  if (!tensor_.defined()) {
    fmt::print(stream, "[ Tensor (undefined) ]");
    return stream;
  }

```
- EN: Focus symbols: `print`, `defined`, `Tensor`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`print`, `defined`, `Tensor`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 287-297
```cpp
  if (tensor_.is_sparse()) {
    fmt::print(stream, "[ {}{{}}\nindices:\n", tensor_.toString());
    print(stream, tensor_._indices(), linesize);
    fmt::print(stream, "\nvalues:\n");
    print(stream, tensor_._values(), linesize);
    fmt::print(stream, "\nsize:\n{}\n]", fmt::streamed(tensor_.sizes()));
    return stream;
  }

  Tensor tensor;

```
- EN: Focus symbols: `is_sparse`, `print`, `toString`, `_indices`, `_values`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`is_sparse`, `print`, `toString`, `_indices`, `_values`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 298-307
```cpp
  if (tensor_.is_quantized()) {
    tensor = tensor_.dequantize().to(kCPU, kDouble).contiguous();
  } else if (tensor_.is_mkldnn()) {
    fmt::print(stream, "MKLDNN Tensor: ");
    tensor = tensor_.to_dense().to(kCPU, kDouble).contiguous();
  } else if (tensor_.is_mps()) {
    // MPS does not support double tensors, so first copy then convert
    tensor = tensor_.to(kCPU).to(kDouble).contiguous();
  } else if (tensor_.is_privateuseone()) {
    // PrivateUseOne backends may not support double tensors
```
- EN: Focus symbols: `is_quantized`, `dequantize`, `to`, `contiguous`, `is_mkldnn`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`is_quantized`, `dequantize`, `to`, `contiguous`, `is_mkldnn`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 308-317
```cpp
    tensor = tensor_.to(kCPU).to(kDouble).contiguous();
  } else {
    tensor = tensor_.to(kCPU, kDouble).contiguous();
  }

  if (tensor.ndimension() == 0) {
    fmt::print(
        stream,
        "{}\n[ {}{{}}",
        tensor.const_data_ptr<double>()[0],
```
- EN: Focus symbols: `to`, `contiguous`, `ndimension`, `print`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`to`, `contiguous`, `ndimension`, `print`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 318-327
```cpp
        tensor_.toString());
  } else if (tensor.ndimension() == 1) {
    if (tensor.numel() > 0) {
      auto printFmt = __printFormat(tensor);
      if (printFmt.scale != 1) {
        fmt::print(stream, "{} *\n", printFmt.scale);
      }
      const double* tensor_p = tensor.const_data_ptr<double>();
      for (const auto i : c10::irange(tensor.size(0))) {
        printValue(stream, tensor_p[i], printFmt);
```
- EN: Focus symbols: `toString`, `ndimension`, `numel`, `__printFormat`, `print`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`toString`, `ndimension`, `numel`, `__printFormat`, `print`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 328-337
```cpp
        stream.put('\n');
      }
    }
    fmt::print(stream, "[ {}{{{}}}", tensor_.toString(), tensor.size(0));
  } else if (tensor.ndimension() == 2) {
    if (tensor.numel() > 0) {
      __printMatrix(stream, tensor, linesize, 0);
    }
    fmt::print(
        stream,
```
- EN: Focus symbols: `put`, `print`, `toString`, `size`, `ndimension`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`put`, `print`, `toString`, `size`, `ndimension`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 338-347
```cpp
        "[ {}{{{},{}}}",
        tensor_.toString(),
        tensor.size(0),
        tensor.size(1));
  } else {
    if (tensor.numel() > 0) {
      __printTensor(stream, tensor, linesize);
    }
    fmt::print(stream, "[ {}{{{}", tensor_.toString(), tensor.size(0));
    for (const auto i : c10::irange(1, tensor.ndimension())) {
```
- EN: Focus symbols: `toString`, `size`, `numel`, `__printTensor`, `print`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`toString`, `size`, `numel`, `__printTensor`, `print`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 348-357
```cpp
      fmt::print(stream, ",{}", tensor.size(i));
    }
    fmt::print(stream, "}}");
  }

  // Add quantization info
  if (tensor_.is_quantized()) {
    fmt::print(stream, ", qscheme: {}", toString(tensor_.qscheme()));
    if (tensor_.qscheme() == c10::kPerTensorAffine) {
      fmt::print(
```
- EN: Focus symbols: `print`, `size`, `is_quantized`, `toString`, `qscheme`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`print`, `size`, `is_quantized`, `toString`, `qscheme`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 358-367
```cpp
          stream,
          ", scale: {}, zero_point: {}",
          tensor_.q_scale(),
          tensor_.q_zero_point());
    } else if (
        tensor_.qscheme() == c10::kPerChannelAffine ||
        tensor_.qscheme() == c10::kPerChannelAffineFloatQParams) {
      fmt::print(stream, ", scales: ");
      print(stream, tensor_.q_per_channel_scales(), linesize);
      fmt::print(stream, ", zero_points: ");
```
- EN: Focus symbols: `q_scale`, `q_zero_point`, `qscheme`, `print`, `q_per_channel_scales`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`q_scale`, `q_zero_point`, `qscheme`, `print`, `q_per_channel_scales`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 368-381
```cpp
      print(stream, tensor_.q_per_channel_zero_points(), linesize);
      fmt::print(stream, ", axis: {}", tensor_.q_per_channel_axis());
    }
  }

  // Proxy check for if autograd was built
  if (tensor.getIntrusivePtr()->autograd_meta()) {
    auto& fw_grad = tensor._fw_grad(/* level */ 0);
    if (fw_grad.defined()) {
      fmt::print(stream, ", tangent:\n");
      print(stream, fw_grad, linesize);
    }
  }

```
- EN: Focus symbols: `print`, `q_per_channel_zero_points`, `q_per_channel_axis`, `getIntrusivePtr`, `autograd_meta`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`print`, `q_per_channel_zero_points`, `q_per_channel_axis`, `getIntrusivePtr`, `autograd_meta`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 382-386
```cpp
  fmt::print(stream, " ]");
  return stream;
}

} // namespace at
```
- EN: Focus symbols: `at`, `print`. This block opens or closes namespaces to keep symbols in the intended API scope. This keeps the implementation aligned with the broader ATen/C10 module hierarchy.
- CN: 关注符号：`at`, `print`。该代码块打开或关闭命名空间以维持符号的预期作用域。这使实现与更大的 ATen/C10 模块层次保持一致。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- Type-system design / 类型系统设计
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/Formatting.h`, `c10/util/irange.h`
- External/system includes / 外部或系统头: `fmt/compile.h`, `fmt/format.h`, `fmt/ostream.h`, `cmath`, `cstdint`, `iostream`, `iterator`, `string`
- Companion translation units / 配套实现或声明文件: `aten/src/ATen/core/Formatting.h`
- Inferred semantic dependencies / 推断出的语义依赖: tensor abstractions / 张量抽象; namespace scoping / 命名空间作用域
