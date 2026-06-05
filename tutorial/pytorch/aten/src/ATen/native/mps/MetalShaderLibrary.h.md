# MetalShaderLibrary.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mps/MetalShaderLibrary.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the MPS backend in PyTorch ATen native code and focuses on metal shader library; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的MPS 后端，主题聚焦于 metal shader library；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#pragma once
#ifdef __OBJC__
#include <Metal/Metal.h>
typedef id<MTLLibrary> MTLLibrary_t;
typedef id<MTLFunction> MTLFunction_t;
typedef id<MTLComputePipelineState> MTLComputePipelineState_t;
typedef id<MTLComputeCommandEncoder> MTLComputeCommandEncoder_t;
#else
typedef void MTLCompileOptions;
typedef void* MTLLibrary_t;
typedef void* MTLFunction_t;
typedef void* MTLComputePipelineState_t;
typedef void* MTLComputeCommandEncoder_t;
#endif

#include <c10/core/Scalar.h>
#include <c10/util/OptionalArrayRef.h>
#include <functional>
#include <optional>
#include <type_traits>
#include <unordered_map>
#include <utility>
#include <vector>

// Forward declaration of TensorBase and TensorIteratorBase
namespace at {
class TensorBase;
struct TensorIteratorBase;
} // namespace at
```
- EN: Lines 1-30 pull in 9 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-30 行引入了 9 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 31-60
```cpp
namespace at::native::mps {

namespace detail {
template <typename T>
class has_size_type {
  template <typename U>
  static constexpr std::true_type check(typename U::size_type*);
  template <typename>
  static constexpr std::false_type check(...);

 public:
  static constexpr bool value = decltype(check<T>(nullptr))::value;
};

template <typename T>
constexpr bool has_size_type_v = has_size_type<T>::value;

} // namespace detail

// Returns `gpuAddress` of respective `id<MTLBuffer>` plus storage offset
void* get_tensor_gpu_address(const at::TensorBase&);

class MetalKernelFunction {
 public:
  MetalKernelFunction(MTLComputePipelineState_t cps_, MTLFunction_t f_);
  ~MetalKernelFunction();
  MetalKernelFunction(MetalKernelFunction&) = delete;
  // Shader properties
  uint64_t getMaxThreadsPerThreadgroup() const;
  uint64_t getThreadExecutionWidth() const;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp
  uint64_t getStaticThreadGroupMemoryLength() const;
  void runCommandBlock(std::function<void(void)> f);
  // Methods below should be called from runCommandBlock function
  void startEncoding();
  void setArg(unsigned idx, const at::TensorBase& t);
  void setArg(unsigned idx, const void* ptr, uint64_t size);
  void setErrorBufferIndex(unsigned idx);
  template <
      typename T,
      typename = std::enable_if_t<
          std::is_integral_v<T> || std::is_same_v<T, float> ||
          (std::is_class_v<T> && std::is_trivially_copyable_v<T> &&
           !detail::has_size_type_v<T>)>>
  inline void setArg(unsigned idx, const T val) {
    setArg(idx, &val, sizeof(T));
  }

  template <
      typename Container,
      typename = std::enable_if_t<detail::has_size_type_v<Container>>>
  inline void setArg(unsigned idx, const Container& values) {
    setArg(
        idx,
        values.data(),
        values.size() * sizeof(typename Container::value_type));
  }
  void dispatch(
      uint64_t length,
      std::optional<uint64_t> groupSize = std::nullopt);
  void dispatch(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are setArg, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 setArg，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
      c10::ArrayRef<uint64_t> length,
      c10::OptionalArrayRef<uint64_t> groupSize = std::nullopt);

 private:
  MTLComputePipelineState_t cps;
  MTLFunction_t func;
  MTLComputeCommandEncoder_t encoder = nullptr;
};

class MetalShaderLibrary {
 public:
  MetalShaderLibrary(std::string src)
      : shaderSource(std::move(src)), nparams(0), compile_options(nullptr) {}
  MetalShaderLibrary(std::string src, unsigned nparams_)
      : shaderSource(std::move(src)),
        nparams(nparams_),
        compile_options(nullptr) {}
  MetalShaderLibrary(
      std::string src,
      unsigned nparams_,
      MTLCompileOptions* compile_options_)
      : shaderSource(std::move(src)),
        nparams(nparams_),
        compile_options(compile_options_) {}
  MetalShaderLibrary(const MetalShaderLibrary&) = delete;
  virtual ~MetalShaderLibrary();
  std::vector<std::string> getFunctionNames();
  std::shared_ptr<MetalKernelFunction> getKernelFunction(
      const std::string& name);
  // Returns a raw pointer to the kernel function for use in C APIs
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are MetalShaderLibrary, concentrating a specific part of the operator behavior.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 MetalShaderLibrary，它们承载了某一部分算子行为的核心逻辑。

### Lines 121-150
```cpp
  MetalKernelFunction* getCachedKernelFunctionPtr(const std::string& name);
  inline MTLComputePipelineState_t getPipelineStateForFunc(
      const std::string& fname) {
    return getLibraryPipelineState(getLibrary(), fname).first;
  }
  MTLComputePipelineState_t getPipelineStateForFunc(
      const std::string& fname,
      const std::initializer_list<std::string>& params) {
    return getLibraryPipelineState(getLibrary(params), fname).first;
  }
  inline MTLFunction_t getMTLFunction(const std::string& fname) {
    return getLibraryPipelineState(getLibrary(), fname).second;
  }
  MTLFunction_t getMTLFunction(
      const std::string& fname,
      const std::initializer_list<std::string>& params) {
    return getLibraryPipelineState(getLibrary(params), fname).second;
  }
  static MetalShaderLibrary& getBundledLibrary();
  void exec_unary_kernel(
      TensorIteratorBase& iter,
      const std::string& name,
      const std::optional<c10::Scalar> alpha = std::nullopt,
      const std::optional<c10::ScalarType> scalar_arg_type = std::nullopt,
      bool supports_vec4 = false);
  void exec_binary_kernel(
      TensorIteratorBase& iter,
      const std::string& name,
      const std::optional<c10::Scalar> alpha = std::nullopt,
      const std::optional<c10::ScalarType> scalar_arg_type = std::nullopt);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are getMTLFunction, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 getMTLFunction，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 151-180
```cpp
  void exec_ternary_kernel(TensorIteratorBase& iter, const std::string& name);

  template <typename T>
  void exec_unary_kernel_with_params(
      TensorIteratorBase& iter,
      const std::string& name,
      T params,
      const std::string& params_type_name);
  template <typename T>
  void exec_binary_kernel_with_params(
      TensorIteratorBase& iter,
      const std::string& name,
      T params,
      const std::string& params_type_name);

 protected:
  virtual MTLLibrary_t getLibrary();
  virtual MTLLibrary_t getLibrary(
      const std::initializer_list<std::string>& params);
  MTLLibrary_t library = nullptr;

 private:
  std::pair<MTLComputePipelineState_t, MTLFunction_t> getLibraryPipelineState(
      MTLLibrary_t lib,
      const std::string& fname);
  MTLLibrary_t compileLibrary(const std::string& src);
  std::string shaderSource;
  unsigned nparams;
  MTLCompileOptions* compile_options;
  std::unordered_map<std::string, MTLLibrary_t> libMap;
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 181-206
```cpp
  std::unordered_map<
      std::string,
      std::pair<MTLComputePipelineState_t, MTLFunction_t>>
      cplMap;
  // Cache for kernel functions returned by getCachedKernelFunctionPtr
  std::unordered_map<std::string, std::unique_ptr<MetalKernelFunction>>
      kernelCache;
};

class DynamicMetalShaderLibrary : public MetalShaderLibrary {
 public:
  DynamicMetalShaderLibrary(const std::string& src) : MetalShaderLibrary(src) {
    // Compile right away
    getLibrary();
  }
  ~DynamicMetalShaderLibrary() override;
};

class PrecompiledMetalShaderLibrary : public MetalShaderLibrary {
 public:
  explicit PrecompiledMetalShaderLibrary(std::vector<uint8_t> data);
  explicit PrecompiledMetalShaderLibrary(const std::string& path);
  ~PrecompiledMetalShaderLibrary() override;
};

} // namespace at::native::mps
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are DynamicMetalShaderLibrary, concentrating a specific part of the operator behavior.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 DynamicMetalShaderLibrary，它们承载了某一部分算子行为的核心逻辑。

## Key Concepts / 关键概念

- EN: Backend focus: MPS backend.
- CN: 后端重点：MPS 后端。
- EN: TensorIterator is used to describe elementwise or reduction-style iteration.
- CN: 使用 TensorIterator 描述逐元素或归约式遍历。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: setArg, MetalShaderLibrary, getPipelineStateForFunc, getMTLFunction, DynamicMetalShaderLibrary.
- CN: 重要符号：setArg, MetalShaderLibrary, getPipelineStateForFunc, getMTLFunction, DynamicMetalShaderLibrary。

## Dependencies / 依赖关系

- EN: Primary internal headers: `c10/core/Scalar.h, c10/util/OptionalArrayRef.h`.
- CN: 主要内部头文件：`c10/core/Scalar.h, c10/util/OptionalArrayRef.h`。
- EN: External/system headers: `Metal/Metal.h, functional, optional, type_traits, unordered_map, utility, vector`.
- CN: 外部/系统头文件：`Metal/Metal.h, functional, optional, type_traits, unordered_map, utility, vector`。
- EN: The implementation revolves around symbols such as `setArg, MetalShaderLibrary, getPipelineStateForFunc, getMTLFunction, DynamicMetalShaderLibrary`.
- CN: 实现围绕 `setArg, MetalShaderLibrary, getPipelineStateForFunc, getMTLFunction, DynamicMetalShaderLibrary` 等符号展开。
