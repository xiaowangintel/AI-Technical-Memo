# passes.h — Code Analysis / 代码分析

## Source / 来源
- **Path:** `python/src/passes.h`
- **EN:** This header defines a small family of preprocessor macros that turn MLIR/Triton pass-construction functions into pybind11 bindings. It centralizes repetitive wrapper patterns so `passes.cc` can register many passes concisely.
- **CN:** 该头文件定义了一组预处理宏，用于把 MLIR/Triton 的 pass 构造函数转换成 pybind11 绑定。它把重复的包装模式集中起来，使 `passes.cc` 可以简洁地注册大量 pass。

## Line-by-Line Analysis / 逐行分析
- **Lines 1-2**
  - **EN:** `ADD_PASS_WRAPPER_0` exposes a pass builder that takes no extra arguments. The generated binding accepts a `mlir::PassManager&` and directly adds a top-level pass via `pm.addPass(builder())`.
  - **CN:** `ADD_PASS_WRAPPER_0` 用于暴露不带额外参数的 pass 构造器。生成的绑定接收 `mlir::PassManager&`，并通过 `pm.addPass(builder())` 直接添加顶层 pass。
- **Lines 4-7**
  - **EN:** `ADD_FUNC_PASS_WRAPPER_0` is similar, but it inserts the pass as a nested pass on `mlir::triton::FuncOp`. This is needed when a pass operates on Triton function operations rather than whole modules.
  - **CN:** `ADD_FUNC_PASS_WRAPPER_0` 与前者类似，但它把 pass 作为 `mlir::triton::FuncOp` 上的嵌套 pass 插入。当 pass 作用于 Triton 函数操作而不是整个模块时，就需要这种形式。
- **Lines 9-11**
  - **EN:** `ADD_PASS_WRAPPER_1` handles builders with one explicit parameter. The lambda forwards the Python value into the pass factory and still registers a top-level pass.
  - **CN:** `ADD_PASS_WRAPPER_1` 处理带一个显式参数的构造器。lambda 会把 Python 传入的值转发给 pass 工厂，并仍然注册为顶层 pass。
- **Lines 13-25**
  - **EN:** `ADD_PASS_WRAPPER_2`, `ADD_PASS_WRAPPER_3`, and `ADD_PASS_WRAPPER_4` extend the same pattern to builders taking two, three, or four positional arguments. The file intentionally stops at four because that is enough for the current pass-registration needs in `passes.cc`.
  - **CN:** `ADD_PASS_WRAPPER_2`、`ADD_PASS_WRAPPER_3` 和 `ADD_PASS_WRAPPER_4` 将同样的模式扩展到接收 2、3、4 个位置参数的构造器。之所以到 4 为止，是因为这已经满足 `passes.cc` 里的当前 pass 注册需求。
- **Lines 27-29**
  - **EN:** `ADD_PASS_OPTION_WRAPPER_1` is for builders that accept an options struct rather than plain positional parameters. It wraps the single value in braces, relying on aggregate initialization of the options type.
  - **CN:** `ADD_PASS_OPTION_WRAPPER_1` 用于接收 options 结构体而不是普通位置参数的构造器。它使用花括号把单个值包起来，依赖 options 类型的聚合初始化。
- **Lines 31-48**
  - **EN:** `ADD_PASS_OPTION_WRAPPER_2` through `ADD_PASS_OPTION_WRAPPER_5` generalize the options-struct pattern for two to five fields. These macros make Python bindings match the pass factory signatures without manually naming the options classes in binding code.
  - **CN:** `ADD_PASS_OPTION_WRAPPER_2` 到 `ADD_PASS_OPTION_WRAPPER_5` 将 options 结构体模式推广到 2 到 5 个字段。这些宏让 Python 绑定能够匹配 pass 工厂签名，而无需在绑定代码中手动写出 options 类名。

## Key Concepts / 关键概念
- **EN:** Macro-based binding generation removes boilerplate when many pass factories have nearly identical pybind11 exposure logic.
- **CN:** 基于宏的绑定生成可以消除大量 pass 工厂在 pybind11 暴露时的样板代码。
- **EN:** The distinction between `addPass` and `addNestedPass<...>` reflects MLIR’s hierarchical pass-manager model.
- **CN:** `addPass` 与 `addNestedPass<...>` 的区别体现了 MLIR 分层 pass manager 的模型。
- **EN:** Brace-init wrappers indicate that several Triton pass factories take option aggregates, not raw scalar argument lists.
- **CN:** 使用花括号初始化说明一些 Triton pass 工厂接收的是 options 聚合体，而不是原始标量参数列表。

## Dependencies / 依赖关系
- **EN:** Depends on `pybind11` naming and lambda syntax, because every macro expands to `m.def(...)` registrations.
- **CN:** 依赖 `pybind11` 的命名与 lambda 语法，因为每个宏最终都会展开成 `m.def(...)` 注册。
- **EN:** Depends on MLIR `PassManager` APIs such as `addPass` and `addNestedPass`.
- **CN:** 依赖 MLIR 的 `PassManager` API，例如 `addPass` 和 `addNestedPass`。
- **EN:** Used directly by `python/src/passes.cc`, which supplies concrete pass-builder functions from Triton and MLIR headers.
- **CN:** 由 `python/src/passes.cc` 直接使用，后者提供来自 Triton 与 MLIR 头文件的具体 pass 构造函数。
