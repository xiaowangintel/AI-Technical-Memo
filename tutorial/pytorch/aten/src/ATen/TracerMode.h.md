# TracerMode.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `aten/src/ATen/TracerMode.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines or implements core ATen tensor abstractions, utilities, and runtime helpers. This file specifically declares the logic associated with `TracerMode.h`. Threading and execution-policy decisions are important to understanding the code. The leading comment summarizes the intent as: "NOTE [Tracing Mode Switches] Historically, tracing function was controlled by two switches: - `AutoDispatchBelowADInplaceOrView` guard Tracing function used to be script-generated inside `VariableType_*.cpp` kernels, sharing the same `Autograd` dispatch key with autograd function. Therefore, before tracing function was moved out of VariableType, `AutoDispatchBelowADInplaceOrView` guard can also disable tracing as a side effect of disabling `Autograd` dispatching. - `setTracingState()` API in `torch/csrc/jit/frontend/tracer.h` It stores tracing data in a `TracingState` object in TLS. If the `TracingState` object in TLS is `null`, then tracing is paused. The `TracingState` object is created in `tracer::trace()` - the main entrance of tracing function. It's temporarily set to `null` inside generated VariableType (now TraceType) to bypass tracing for intermediate ops (ops being called by other ops). After the intermediate op call finishes it's set back to the original `TracingState` object. The `TracingState` object in TLS can also be read/written via its Python binding in `python_tracer.cpp`, and `get/setTracingState()` C++ APIs, which are also exposed as `TORCH_API`. Two new switches were introduced since tracing function was moved out of VariableType: - `tracer::impl::set_dispatch_enabled()` API Unlike the special `Autograd` dispatch key which is included in dispatch key set by default, `Tracer` dispatch key is off by default. The dispatching switch can be toggled via this new API. - `tracer::impl::NoTracerDispatchMode` guard It's used to cover the old semantics of `AutoDispatchBelowADInplaceOrView` after tracing was moved out of VariableType. Before tracing function was moved out of VariableType, tracing was enabled when the following conditions are satisfied: 1) `TracingState` object in TLS != null; - Either inside the execution scope of `tracer::trace()`, or - Eagerly called `setTracingState()` with non-null object. 2) Not inside `AutoDispatchBelowADInplaceOrView` scope; After: 1) `TracingState` object in TLS != null; 2) Has called `tracer::impl::set_dispatch_enabled(true)`; 3) Not inside `tracer::impl::NonDispatchGuard` scope;."
- **Purpose (CN)**: 定义或实现 ATen 的核心张量抽象、工具函数与运行时辅助逻辑。 该文件具体声明与 `TracerMode.h` 相关的逻辑。 线程与执行策略的决策是理解该文件的重要切入点。 文件头部注释给出的意图摘要为：“NOTE [Tracing Mode Switches] Historically, tracing function was controlled by two switches: - `AutoDispatchBelowADInplaceOrView` guard Tracing function used to be script-generated inside `VariableType_*.cpp` kernels, sharing the same `Autograd` dispatch key with autograd function. Therefore, before tracing function was moved out of VariableType, `AutoDispatchBelowADInplaceOrView` guard can also disable tracing as a side effect of disabling `Autograd` dispatching. - `setTracingState()` API in `torch/csrc/jit/frontend/tracer.h` It stores tracing data in a `TracingState` object in TLS. If the `TracingState` object in TLS is `null`, then tracing is paused. The `TracingState` object is created in `tracer::trace()` - the main entrance of tracing function. It's temporarily set to `null` inside generated VariableType (now TraceType) to bypass tracing for intermediate ops (ops being called by other ops). After the intermediate op call finishes it's set back to the original `TracingState` object. The `TracingState` object in TLS can also be read/written via its Python binding in `python_tracer.cpp`, and `get/setTracingState()` C++ APIs, which are also exposed as `TORCH_API`. Two new switches were introduced since tracing function was moved out of VariableType: - `tracer::impl::set_dispatch_enabled()` API Unlike the special `Autograd` dispatch key which is included in dispatch key set by default, `Tracer` dispatch key is off by default. The dispatching switch can be toggled via this new API. - `tracer::impl::NoTracerDispatchMode` guard It's used to cover the old semantics of `AutoDispatchBelowADInplaceOrView` after tracing was moved out of VariableType. Before tracing function was moved out of VariableType, tracing was enabled when the following conditions are satisfied: 1) `TracingState` object in TLS != null; - Either inside the execution scope of `tracer::trace()`, or - Eagerly called `setTracingState()` with non-null object. 2) Not inside `AutoDispatchBelowADInplaceOrView` scope; After: 1) `TracingState` object in TLS != null; 2) Has called `tracer::impl::set_dispatch_enabled(true)`; 3) Not inside `tracer::impl::NonDispatchGuard` scope;”。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6 / 第 1-6 行

```cpp
#pragma once

#include <c10/core/impl/LocalDispatchKeySet.h>
#include <c10/macros/Export.h>
#include <c10/macros/Macros.h>

```

- **EN:** This range is mainly assembling the headers or imports needed by the declarations and runtime logic that follow.
- **CN:** 这一段主要在组织后续声明与运行时逻辑所需的头文件或导入项。
- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Code generation / 代码生成, Header composition / 头文件组织.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Code generation / 代码生成, Header composition / 头文件组织。

### Lines 7-18 / 第 7-18 行

```cpp
// NOTE [Tracing Mode Switches]
//
// Historically, tracing function was controlled by two switches:
//
// - `AutoDispatchBelowADInplaceOrView` guard
//
//    Tracing function used to be script-generated inside `VariableType_*.cpp`
//    kernels, sharing the same `Autograd` dispatch key with autograd function.
//    Therefore, before tracing function was moved out of VariableType,
//    `AutoDispatchBelowADInplaceOrView` guard can also disable tracing as a
//    side effect of disabling `Autograd` dispatching.
//
```

- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Code generation / 代码生成.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Code generation / 代码生成。

### Lines 19-30 / 第 19-30 行

```cpp
// - `setTracingState()` API in `torch/csrc/jit/frontend/tracer.h`
//
//    It stores tracing data in a `TracingState` object in TLS. If the
//    `TracingState` object in TLS is `null`, then tracing is paused.
//
//    The `TracingState` object is created in `tracer::trace()` - the main
//    entrance of tracing function. It's temporarily set to `null` inside
//    generated VariableType (now TraceType) to bypass tracing for intermediate
//    ops (ops being called by other ops). After the intermediate op call
//    finishes it's set back to the original `TracingState` object.
//
//    The `TracingState` object in TLS can also be read/written via its Python
```

- **EN:** Concepts touched here: Sparse tensor / 稀疏张量, Thread-local state / 线程局部状态, Tracing and hooks / 追踪与钩子, Code generation / 代码生成.
- **CN:** 这里涉及的概念包括：Sparse tensor / 稀疏张量, Thread-local state / 线程局部状态, Tracing and hooks / 追踪与钩子, Code generation / 代码生成。

### Lines 31-42 / 第 31-42 行

```cpp
//    binding in `python_tracer.cpp`, and `get/setTracingState()` C++ APIs,
//    which are also exposed as `TORCH_API`.
//
// Two new switches were introduced since tracing function was moved out of
// VariableType:
//
// - `tracer::impl::set_dispatch_enabled()` API
//
//    Unlike the special `Autograd` dispatch key which is included in dispatch
//    key set by default, `Tracer` dispatch key is off by default. The
//    dispatching switch can be toggled via this new API.
//
```

- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Tracing and hooks / 追踪与钩子.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Tracing and hooks / 追踪与钩子。

### Lines 43-54 / 第 43-54 行

```cpp
// - `tracer::impl::NoTracerDispatchMode` guard
//
//    It's used to cover the old semantics of `AutoDispatchBelowADInplaceOrView`
//    after tracing was moved out of VariableType.
//
// Before tracing function was moved out of VariableType, tracing was enabled
// when the following conditions are satisfied:
//
//    1) `TracingState` object in TLS != null;
//       - Either inside the execution scope of `tracer::trace()`, or
//       - Eagerly called `setTracingState()` with non-null object.
//    2) Not inside `AutoDispatchBelowADInplaceOrView` scope;
```

- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Thread-local state / 线程局部状态, Tracing and hooks / 追踪与钩子.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Thread-local state / 线程局部状态, Tracing and hooks / 追踪与钩子。

### Lines 55-66 / 第 55-66 行

```cpp
//
// After:
//
//    1) `TracingState` object in TLS != null;
//    2) Has called `tracer::impl::set_dispatch_enabled(true)`;
//    3) Not inside `tracer::impl::NonDispatchGuard` scope;
//
// [TODOs]
//
// - `setTracingState()` v.s. `tracer::impl::set_dispatch_enabled()`
//
//   Currently `set_dispatch_enabled()` is set/unset inside `setTracingState()`
```

- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Thread-local state / 线程局部状态, Tracing and hooks / 追踪与钩子.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Thread-local state / 线程局部状态, Tracing and hooks / 追踪与钩子。

### Lines 67-78 / 第 67-78 行

```cpp
//   to keep the semantics exactly the same as before - it's confusing to keep
//   both switches, though. We should consider simplifying/limiting the exposed
//   `setTracingState()` Python/C++ APIs (and other APIs calling it) so that
//   these two can be unified.
//
// - `AutoDispatchBelowADInplaceOrView` v.s.
// `tracer::impl::NoTracerDispatchMode`
//
//   We don't need to always set both guards together to keep semantics
//   unchanged. For the follow use cases of `AutoDispatchBelowADInplaceOrView`
//   we don't need set the new tracer guard:
//
```

- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Tracing and hooks / 追踪与钩子.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Tracing and hooks / 追踪与钩子。

### Lines 79-90 / 第 79-90 行

```cpp
//   * Script-generated VariableType kernels. The guard is not necessary as
//     tracing is already disabled explicitly by `setTracingState(null)` in
//     generated TraceType kernels - we could keep it as is or use the new guard
//     instead.
//
//   * Custom ops. Will be handled by fallback kernel for `Tracer`.
//
//   * Functions that are not likely to be called in tracing context (no python
//     binding / not an operator), e.g.: all mobile forward() wrappers, test
//     binaries, and etc.
//
//   * Where new threads are spawned, e.g.: ATen/native/ConvolutionMM2d.cpp.
```

- **EN:** Concurrency-related constructs appear here, showing how work is partitioned, synchronized, or scheduled.
- **CN:** 这里出现了并发相关结构，说明代码在处理任务划分、同步或调度。
- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Parallel runtime / 并行运行时, Tracing and hooks / 追踪与钩子, Code generation / 代码生成.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Parallel runtime / 并行运行时, Tracing and hooks / 追踪与钩子, Code generation / 代码生成。

### Lines 91-102 / 第 91-102 行

```cpp
//     It's not necessary as tracing is off by default.
//
//   For the rest of cases we might need have both:
//
//   * Functions that might be reachable from eager mode python (especially
//     factory methods), e.g.:
//     `internal_new_from_data()` in `torch/csrc/utils/tensor_new.cpp`.
//     Without the new guard it will add `aten::empty` to the traced graph.
//
//   * Some manually maintained functions, e.g.:
//     `torch/csrc/autograd/VariableTypeManual.cpp`.
//     Set the new guard if it's not obvious whether `setTracingState(null)`
```

- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Sparse tensor / 稀疏张量.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Sparse tensor / 稀疏张量。

### Lines 103-110 / 第 103-110 行

```cpp
//     has been called before it reaches the `AutoDispatchBelowADInplaceOrView`
//     guard.
//
//   We might need tweak the usage of the new guard to optimize/fix things.
//   It should only affect the correctness of tracing function, because the
//   guard is essentially no-op when the master `setTracingState()` switch is
//   off.

```

- **EN:** Concepts touched here: Dispatch and registration / 分发与注册.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册。

### Lines 111-116 / 第 111-116 行

```cpp
// TODO: move this from `at::` to `jit::torch::` after
// `aten/src/ATen/cpp_custom_type_hack.h` is removed.

namespace at::tracer::impl {

inline bool is_dispatch_enabled() {
```

- **EN:** It establishes namespace scopes such as at::tracer::impl, placing the code inside the relevant ATen/backend subsystem.
- **CN:** 这里建立了 at::tracer::impl 等命名空间作用域，把代码放入对应的 ATen/后端子系统中。
- **EN:** Important callable entry points in this range include is_dispatch_enabled.
- **CN:** 这一段的重要可调用入口包括 is_dispatch_enabled。
- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Tracing and hooks / 追踪与钩子, Declared symbols / 声明符号, Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Tracing and hooks / 追踪与钩子, Declared symbols / 声明符号, Namespace scoping / 命名空间作用域。

### Lines 117-122 / 第 117-122 行

```cpp
  return c10::impl::tls_is_dispatch_key_included(at::DispatchKey::Tracer) &&
      !c10::impl::tls_is_dispatch_key_excluded(at::DispatchKey::Tracer);
}

inline void set_dispatch_enabled(bool enabled) {
  TORCH_INTERNAL_ASSERT(
```

- **EN:** Important callable entry points in this range include tls_is_dispatch_key_included, set_dispatch_enabled.
- **CN:** 这一段的重要可调用入口包括 tls_is_dispatch_key_included, set_dispatch_enabled。
- **EN:** Return statements finalize the local helper result, state update, or wrapper output for the surrounding API.
- **CN:** 返回语句会为周边 API 给出局部辅助结果、状态更新或封装后的输出。
- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Thread-local state / 线程局部状态, Tracing and hooks / 追踪与钩子, Code generation / 代码生成, Declared symbols / 声明符号.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Thread-local state / 线程局部状态, Tracing and hooks / 追踪与钩子, Code generation / 代码生成, Declared symbols / 声明符号。

### Lines 123-128 / 第 123-128 行

```cpp
      !c10::impl::tls_is_dispatch_key_excluded(at::DispatchKey::Tracer),
      "Cannot enable tracing within the scope of NoTracerDispatchMode!");
  c10::impl::tls_set_dispatch_key_included(at::DispatchKey::Tracer, enabled);
}

struct NoTracerDispatchMode {
```

- **EN:** The block introduces or refines types such as NoTracerDispatchMode.
- **CN:** 该代码块引入或细化了 NoTracerDispatchMode 等类型。
- **EN:** Important callable entry points in this range include tls_set_dispatch_key_included.
- **CN:** 这一段的重要可调用入口包括 tls_set_dispatch_key_included。
- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Thread-local state / 线程局部状态, Tracing and hooks / 追踪与钩子, Code generation / 代码生成, Declared symbols / 声明符号.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Thread-local state / 线程局部状态, Tracing and hooks / 追踪与钩子, Code generation / 代码生成, Declared symbols / 声明符号。

### Lines 129-132 / 第 129-132 行

```cpp
  c10::impl::ExcludeDispatchKeyGuard guard_{at::DispatchKey::Tracer};
};

} // namespace at::tracer::impl
```

- **EN:** Concepts touched here: Dispatch and registration / 分发与注册, Tracing and hooks / 追踪与钩子, Code generation / 代码生成, Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Dispatch and registration / 分发与注册, Tracing and hooks / 追踪与钩子, Code generation / 代码生成, Namespace scoping / 命名空间作用域。

## Key Concepts / 关键概念

- **Core ATen runtime** — ATen 核心运行时
- **Tensor metadata** — 张量元数据
- **Sparse tensor** — 稀疏张量
- **Dispatch and registration** — 分发与注册
- **Parallel runtime** — 并行运行时
- **Thread-local state** — 线程局部状态
- **Tracing and hooks** — 追踪与钩子
- **Code generation** — 代码生成
- **Core symbols: NoTracerDispatchMode, is_dispatch_enabled, tls_is_dispatch_key_included, set_dispatch_enabled, tls_set_dispatch_key_included** — 核心符号：NoTracerDispatchMode、is_dispatch_enabled、tls_is_dispatch_key_included、set_dispatch_enabled、tls_set_dispatch_key_included

## Dependencies / 依赖关系

- `c10/core/impl/LocalDispatchKeySet.h`
- `c10/macros/Export.h`
- `c10/macros/Macros.h`
