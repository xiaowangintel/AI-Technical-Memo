# codegen.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/codegen.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
#pragma once

#include <ATen/ATen.h>
#include <torch/csrc/jit/tensorexpr/ir.h>
#include <torch/csrc/jit/tensorexpr/tensor.h>

#include <utility>

namespace torch::jit::tensorexpr {

template <typename T>
class PaddedBuffer;
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/ir.h, torch/csrc/jit/tensorexpr/tensor.h; ATen/c10 facilities such as ATen/ATen.h; standard-library headers such as utility. The preprocessor guard keeps declarations single-instanced when the header is included transitively. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. It introduces or extends PaddedBuffer, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/ir.h、torch/csrc/jit/tensorexpr/tensor.h；ATen/c10 基础设施，如 ATen/ATen.h；标准库头文件，如 utility。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 它引入或扩展了 PaddedBuffer，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 14-27
```cpp
class TORCH_API CodeGen {
 public:
  class BufferArg;
  class CallArg;

  template <typename... Ts>
  CodeGen(StmtPtr stmt, Ts... ts)
      : stmt_(std::move(stmt)), buffer_args_({BufferArg(ts)...}) {}

  CodeGen(
      StmtPtr stmt,
      std::vector<BufferArg> buffer_args,
      at::Device device = at::kCPU,
      std::string kernel_func_name = "func");
```
- **EN**: It introduces or extends TORCH_API, BufferArg, CallArg, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 TORCH_API、BufferArg、CallArg，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 29-41
```cpp
  CodeGen(const CodeGen& rhs);

  CodeGen(CodeGen&& rhs);

  virtual ~CodeGen();

  CodeGen& operator=(const CodeGen& rhs);

  CodeGen& operator=(CodeGen&& rhs);

  StmtPtr stmt() const {
    return stmt_;
  }
```
- **EN**: This chunk defines `stmt`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `stmt`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 43-57
```cpp
  void set_stmt(StmtPtr s) {
    stmt_ = std::move(s);
  }

  void apply_mutator(IRMutator* mutator) {
    stmt_ = stmt_->accept_mutator(mutator);
  }

  void apply_visitor(IRVisitor* visitor) {
    stmt_->accept(visitor);
  }

  std::vector<BufferArg>& buffer_args() {
    return buffer_args_;
  }
```
- **EN**: This chunk defines `buffer_args`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `buffer_args`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 59-72
```cpp
  const std::vector<BufferArg>& buffer_args() const {
    return buffer_args_;
  }

  at::Device device() {
    return device_;
  }

  // This function returns the generated code as
  // a string.
  virtual std::string getCodeText(
      const std::string& attr [[maybe_unused]] = "") {
    return "";
  }
```
- **EN**: This chunk defines `getCodeText`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getCodeText`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 74-88
```cpp
  // TODO: Figure out how to unify these call interfaces.

  /// Call a function with a vector of CallArgs, which are tagged
  /// unions that properly type the arguments.
  virtual void call(const std::vector<CallArg>& args) = 0;

  /// Call a function faster than a regular `call` by assuming that
  /// the generated kernel already knows the type of the arguments, so
  /// they can be type-punned with `void*`s.
  virtual void call_raw(const std::vector<void*>& args) = 0;

  /// Call a function even faster than a regular call, by assuming
  /// that the number of thread blocks can be derived from `numel` via
  /// a simple division, rather than evaluating an expression.
  virtual void call_with_numel(void** args, int64_t numel);
```
- **EN**: This chunk declares `call_with_numel`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段声明了 `call_with_numel`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 90-103
```cpp
  virtual at::Tensor empty_strided(
      c10::IntArrayRef size,
      c10::IntArrayRef stride,
      std::optional<c10::ScalarType> dtype_opt,
      std::optional<c10::Layout> layout_opt,
      std::optional<c10::Device> device_opt,
      std::optional<bool> pin_memory_opt) {
    return at::empty_strided(
        size, stride, dtype_opt, layout_opt, device_opt, pin_memory_opt);
  }

  const std::string& kernel_func_name() const {
    return kernel_func_name_;
  }
```
- **EN**: This chunk defines `kernel_func_name`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `kernel_func_name`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 105-119
```cpp
  void allocIntermediateBufs();

 protected:
  static void* argToPtr(const BufferArg& bufferArg, const CallArg& callArg);

 private:
  StmtPtr stmt_;
  std::vector<BufferArg> buffer_args_;
  at::Device device_ = at::kCPU;
  std::string kernel_func_name_ = "func";
};

class TORCH_API ExtCallMemoryReuse : public IRMutator {
  static std::unordered_map<std::string, std::string> makeExtCallFuncNameMap();
  static const std::unordered_map<std::string, std::string> extCallFuncNameMap_;
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `argToPtr`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `argToPtr`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 121-136
```cpp
 public:
  explicit ExtCallMemoryReuse(
      const std::vector<CodeGen::BufferArg>& bufferArgs);
  ~ExtCallMemoryReuse() override = default;
  StmtPtr mutate(const ExternalCallPtr& v) override;

 private:
  std::unordered_set<BufPtr> bufferArgs_;
};

class CodeGen::BufferArg {
 public:
  BufferArg(const Tensor& tensor) : buf_(tensor.buf()) {}
  BufferArg(const VarHandle& var) : var_(var.node()), isVar_(true) {}
  BufferArg(const BufHandle& buf) : buf_(buf.node()) {}
  BufferArg(BufPtr buf) : buf_(std::move(buf)) {}
```
- **EN**: It introduces or extends CodeGen, which define the primary data structures or interfaces for this portion of the file. This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics.
- **CN**: 它引入或扩展了 CodeGen，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。

### Lines 138-152
```cpp
  VarPtr var() const {
    return isVar_ ? var_ : buf_->base_handle();
  }

  BufPtr buf() const {
    return buf_;
  }

  bool isVar() const {
    return isVar_;
  }

  Dtype dtype() const {
    return isVar_ ? var_->dtype() : buf_->dtype();
  }
```
- **EN**: This chunk defines `dtype`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `dtype`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 154-167
```cpp
 private:
  VarPtr var_ = nullptr;
  BufPtr buf_ = nullptr;
  bool isVar_ = false;
};

class CodeGen::CallArg {
 public:
  template <typename T>
  CallArg(const PaddedBuffer<T>& buffer);

  template <typename T>
  CallArg(const std::vector<T>& buffer)
      : data_(const_cast<T*>(buffer.data())) {}
```
- **EN**: It introduces or extends CodeGen, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 CodeGen，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 169-181
```cpp
  CallArg(void* ptr) : data_(ptr) {}

#define ARG_TYPE_CTOR(Type, Name)      \
  CallArg(Type v) {                    \
    memcpy(buffer_, &v, sizeof(Type)); \
    data_ = (void*)buffer_;            \
  }
  AT_FORALL_SCALAR_TYPES_AND3(Bool, Half, BFloat16, ARG_TYPE_CTOR)
#undef ARG_TYPE_CTOR

  void* data() const {
    return data_;
  }
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk defines `data`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `data`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 183-198
```cpp
  CallArg(const CallArg& rhs) {
    if (rhs.data_ == rhs.buffer_) {
      memcpy(this->buffer_, rhs.buffer_, sizeof(rhs.buffer_));
      this->data_ = (void*)(this->buffer_);
    } else {
      this->data_ = rhs.data_;
    }
  }

  CallArg& operator=(const CallArg& rhs) {
    if (this == &rhs) {
      return *this;
    }
    if (rhs.data_ == rhs.buffer_) {
      memcpy(this->buffer_, rhs.buffer_, sizeof(rhs.buffer_));
      this->data_ = (void*)(this->buffer_);
```
- **EN**: This chunk continues `data` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `data`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 199-211
```cpp
    } else {
      this->data_ = rhs.data_;
    }
    return *this;
  }

#define ARG_PTR_DEFINE(Type, Name)                  \
  Type* Name##Ptr() const {                         \
    TORCH_INTERNAL_ASSERT(data_ == (void*)buffer_); \
    return (Type*)data_;                            \
  }
  AT_FORALL_SCALAR_TYPES_AND3(Bool, Half, BFloat16, ARG_PTR_DEFINE)
#undef ARG_PTR_DEFINE
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk continues `data` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段延续了 `data`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 213-225
```cpp
 private:
  void* data_;
  // Regarding a scalar value, CallArg uses void**=&data_ to store it. But the
  // bit width of a pointer is 32bit on a 32bit platform. It cannot store the
  // scalar if the bit width of the scalar is larger than 32bit, such as double
  // and long. Hence, we add 8 bytes buffer dedicated to storing the scalar
  // value regardless its bit width is less or greater than 32bits.
  char buffer_[8] = {0}; // 64bits
};

class RegisterCodeGenList {
 public:
  TORCH_API static RegisterCodeGenList& GetInstance();
```
- **EN**: It introduces or extends RegisterCodeGenList, which define the primary data structures or interfaces for this portion of the file. This chunk defines `GetInstance`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 它引入或扩展了 RegisterCodeGenList，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `GetInstance`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 227-242
```cpp
  using StmtFactoryMethod = std::function<std::unique_ptr<CodeGen>(
      StmtPtr stmt,
      const std::vector<CodeGen::BufferArg>&,
      at::Device device,
      const std::string& kernel_func_name)>;

  TORCH_API StmtFactoryMethod FindStmtFactoryMethod(const std::string& name);
  RegisterCodeGenList(const RegisterCodeGenList&) = delete;
  RegisterCodeGenList& operator=(const RegisterCodeGenList&) = delete;

 private:
  template <class CodeGenType>
  friend class RegisterCodeGen;
  RegisterCodeGenList() = default;
  TORCH_API void AddStmtFactoryMethod(
      const std::string& name,
```
- **EN**: This chunk declares `FindStmtFactoryMethod`, which looks up previously defined symbols, cached plans, or registry entries.
- **CN**: 这一段声明了 `FindStmtFactoryMethod`，其作用是查找已定义的符号、缓存计划或注册表条目。

### Lines 243-258
```cpp
      const StmtFactoryMethod& stmt_factory_method);

  std::unordered_map<std::string, StmtFactoryMethod> stmt_factory_methods_;
};

template <class CodeGenType>
class RegisterCodeGen {
 public:
  explicit RegisterCodeGen(const std::string& name) {
    RegisterCodeGenList& codegen_list = RegisterCodeGenList::GetInstance();
    codegen_list.AddStmtFactoryMethod(
        name,
        [](const StmtPtr& stmt,
           const std::vector<CodeGen::BufferArg>& params,
           at::Device device,
           const std::string& kernel_func_name) {
```
- **EN**: It introduces or extends RegisterCodeGen, which define the primary data structures or interfaces for this portion of the file. This chunk defines `RegisterCodeGen`, which lowers higher-level IR into a backend-specific executable form.
- **CN**: 它引入或扩展了 RegisterCodeGen，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `RegisterCodeGen`，其作用是把高层 IR 降级为后端特定的可执行形式。

### Lines 259-270
```cpp
          return std::make_unique<CodeGenType>(
              stmt, params, device, kernel_func_name);
        });
  }
};

TORCH_API std::unique_ptr<CodeGen> CreateCodeGen(
    const std::string& name,
    StmtPtr stmt,
    const std::vector<CodeGen::BufferArg>& params,
    at::Device device = at::kCPU,
    const std::string& kernel_func_name = "func");
```
- **EN**: This chunk declares `CreateCodeGen`, which lowers higher-level IR into a backend-specific executable form. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段声明了 `CreateCodeGen`，其作用是把高层 IR 降级为后端特定的可执行形式。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 272-277
```cpp
class TORCH_API GenericIntrinsicsExpander : public IRMutator {
 protected:
  ExprPtr mutate(const IntrinsicsPtr& v) override;
};

} // namespace torch::jit::tensorexpr
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `mutate`, which visits or transforms IR nodes while preserving the intended semantics.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `mutate`，其作用是访问或变换 IR 节点，同时保持预期语义。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **PaddedBuffer**
  - EN: `PaddedBuffer` is a central symbol declared or implemented in this file.
  - CN: `PaddedBuffer` 是本文件声明或实现的核心符号。
- **TORCH_API**
  - EN: `TORCH_API` is a central symbol declared or implemented in this file.
  - CN: `TORCH_API` 是本文件声明或实现的核心符号。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **Backend code generation**
  - EN: Translates IR into backend-specific source, kernels, or machine-level artifacts.
  - CN: 把 IR 翻译为后端特定的源码、内核或机器级产物。
- **Generated code**
  - EN: This file is primarily machine-generated, so the main value is the declared schema or table layout.
  - CN: 该文件主要由机器生成，因此核心价值在于声明的 schema 或表布局。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/ir.h`, `torch/csrc/jit/tensorexpr/tensor.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/ATen.h`
- **Standard library / 标准库**: `utility`
- **Primary symbols in this file / 本文件核心符号**: `PaddedBuffer`, `TORCH_API`, `BufferArg`, `CallArg`, `CodeGen`, `RegisterCodeGenList`, `RegisterCodeGen`, `~CodeGen`
