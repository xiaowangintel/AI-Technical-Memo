# util.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/util.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-29
```cpp
 1 | /**
 2 |  * Most of the utils in this file is adapted from PyTorch/XLA
 3 |  * https://github.com/pytorch/xla/blob/e0e5f937a0ba8d904f9608137dc8c51ba439df2d/third_party/xla_client/util.h
 4 |  */
 5 | 
 6 | #pragma once
 7 | 
 8 | #include <exception>
 9 | #include <functional>
10 | #include <vector>
11 | 
12 | #include <c10/util/OptionalArrayRef.h>
13 | #include <optional>
14 | 
15 | namespace torch::lazy {
16 | 
17 | // Similar to c10::scope_exit but with a status.
18 | // TODO(alanwaketan): Consolidate it with c10::scope_exit.
19 | template <typename T>
20 | class Cleanup {
21 |  public:
22 |   using StatusType = T;
23 | 
24 |   explicit Cleanup(std::function<void(StatusType&&)>&& func)
25 |       : func_(std::move(func)) {}
26 |   Cleanup(Cleanup&& ref) noexcept
27 |       : func_(std::move(ref.func_)), status_(std::move(ref.status_)) {}
28 |   Cleanup(const Cleanup&) = delete;
29 | 
```
- EN: Brings in project headers such as `<c10/util/OptionalArrayRef.h>` and system or third-party headers such as `<exception>`, `<functional>`, `<vector>`, `<optional>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `Cleanup` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<c10/util/OptionalArrayRef.h>`以及系统或第三方头文件，例如 `<exception>`、`<functional>`、`<vector>`、`<optional>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `Cleanup` 等数据抽象，用来组织本文件处理的状态。

### Lines 30-57
```cpp
30 |   ~Cleanup() {
31 |     if (func_ != nullptr) {
32 |       func_(std::move(status_));
33 |     }
34 |   }
35 | 
36 |   Cleanup& operator=(const Cleanup&) = delete;
37 | 
38 |   Cleanup& operator=(Cleanup&& ref) noexcept {
39 |     if (this != &ref) {
40 |       func_ = std::move(ref.func_);
41 |       status_ = std::move(ref.status_);
42 |     }
43 |     return *this;
44 |   }
45 | 
46 |   void Release() {
47 |     func_ = nullptr;
48 |   }
49 | 
50 |   void SetStatus(StatusType&& status) {
51 |     status_ = std::move(status);
52 |   }
53 | 
54 |   const StatusType& GetStatus() const {
55 |     return status_;
56 |   }
57 | 
```
- EN: Implements routines such as `Release`, `SetStatus`, `GetStatus` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 实现了 `Release`、`SetStatus`、`GetStatus` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 58-89
```cpp
58 |  private:
59 |   std::function<void(StatusType&&)> func_;
60 |   StatusType status_;
61 | };
62 | 
63 | using ExceptionCleanup = Cleanup<std::exception_ptr>;
64 | 
65 | // Allows APIs which might return const references and values, to not be forced
66 | // to return values in the signature.
67 | // TODO(alanwaketan): This is clever, but is there really no std or c10
68 | // supports? Needs more investigations.
69 | template <typename T>
70 | class MaybeRef {
71 |  public:
72 |   /* implicit */ MaybeRef(const T& ref) : ref_(ref) {}
73 |   /* implicit */ MaybeRef(T&& value)
74 |       : storage_(std::move(value)), ref_(*storage_) {}
75 | 
76 |   const T& Get() const {
77 |     return ref_;
78 |   }
79 |   const T& operator*() const {
80 |     return Get();
81 |   }
82 |   operator const T&() const {
83 |     return Get();
84 |   }
85 | 
86 |   bool IsStored() const {
87 |     return storage_.has_value();
88 |   }
89 | 
```
- EN: Defines or extends data abstractions such as `MaybeRef` that structure the state handled by this file. Implements routines such as `Get`, `IsStored` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 定义或扩展了 `MaybeRef` 等数据抽象，用来组织本文件处理的状态。 实现了 `Get`、`IsStored` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 90-119
```cpp
 90 |  private:
 91 |   std::optional<T> storage_;
 92 |   // NOLINTNEXTLINE(cppcoreguidelines-avoid-const-or-ref-data-members)
 93 |   const T& ref_;
 94 | };
 95 | 
 96 | template <typename T>
 97 | std::vector<T> Iota(size_t size, T init = 0, T incr = 1) {
 98 |   std::vector<T> result(size);
 99 |   T value = init;
100 |   for (size_t i = 0; i < size; ++i, value += incr) {
101 |     result[i] = value;
102 |   }
103 |   return result;
104 | }
105 | 
106 | template <typename T, typename S>
107 | std::vector<T> ToVector(const S& input) {
108 |   return std::vector<T>(input.begin(), input.end());
109 | }
110 | 
111 | template <typename T>
112 | std::optional<std::vector<T>> ToOptionalVector(
113 |     c10::OptionalArrayRef<T> arrayRef) {
114 |   if (arrayRef) {
115 |     return arrayRef->vec();
116 |   }
117 |   return std::nullopt;
118 | }
119 | 
```
- EN: Implements routines such as `Iota`, `result`, `ToVector`, `ToOptionalVector` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `Iota`、`result`、`ToVector`、`ToOptionalVector` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 120-125
```cpp
120 | template <typename T>
121 | std::underlying_type_t<T> GetEnumValue(T value) {
122 |   return static_cast<std::underlying_type_t<T>>(value);
123 | }
124 | 
125 | } // namespace torch::lazy
```
- EN: Implements routines such as `GetEnumValue` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `GetEnumValue` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `Cleanup`, `MaybeRef`.
  - CN: `Cleanup`、`MaybeRef`。
- **Important routines / 重要例程**
  - EN: `Cleanup`, `Release`, `SetStatus`, `GetStatus`, `Get`, `IsStored`, `Iota`, `result`.
  - CN: `Cleanup`、`Release`、`SetStatus`、`GetStatus`、`Get`、`IsStored`、`Iota`、`result`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/util/OptionalArrayRef.h>`
- External includes / 外部头文件: `<exception>`, `<functional>`, `<vector>`, `<optional>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
