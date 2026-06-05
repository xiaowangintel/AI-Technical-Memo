# passes.cc — Code Analysis / 代码分析

## Source / 来源
- **Path:** `python/src/passes.cc`
- **EN:** This file builds the Python-facing pass-registration API for Triton. It groups MLIR and Triton passes into pybind11 submodules, exposes a few analysis helpers, and dynamically publishes plugin-provided passes.
- **CN:** 该文件构建了 Triton 面向 Python 的 pass 注册 API。它把 MLIR 与 Triton 的 pass 按 pybind11 子模块组织起来，暴露少量分析辅助类，并动态发布插件提供的 pass。

## Line-by-Line Analysis / 逐行分析
- **Lines 1-19**
  - **EN:** The includes pull in MLIR pass infrastructure, Triton analyses, Triton/TritonGPU/Gluon/instrumentation pass factories, LLVM IR helpers, plugin utilities, and pybind11 support. This signals that the file is a binding layer, not a pass implementation layer.
  - **CN:** 这些头文件引入了 MLIR pass 基础设施、Triton 分析、Triton/TritonGPU/Gluon/插桩 pass 工厂、LLVM IR 辅助工具以及 pybind11 支持。这说明该文件是“绑定层”，而不是 pass 的实现层。
- **Lines 21-23**
  - **EN:** `namespace py = pybind11;` shortens later bindings, and the anonymous namespace keeps helper registration functions local to this translation unit.
  - **CN:** `namespace py = pybind11;` 简化了后续绑定代码，而匿名命名空间则把辅助注册函数限制在当前编译单元内部。
- **Lines 25-31**
  - **EN:** `init_triton_analysis` exposes two analysis objects: `mlir::ModuleAllocation` as `allocation`, and `mlir::ModuleMembarAnalysis` as `membar`. The second depends on an existing allocation analysis object and exposes its `run` method to Python.
  - **CN:** `init_triton_analysis` 暴露了两个分析对象：把 `mlir::ModuleAllocation` 绑定为 `allocation`，把 `mlir::ModuleMembarAnalysis` 绑定为 `membar`。后者依赖已有的 allocation 分析对象，并向 Python 暴露其 `run` 方法。
- **Lines 33-42**
  - **EN:** `init_triton_passes_common` registers general-purpose MLIR optimizations such as SCCP, symbol DCE, inlining, canonicalization, CSE, LICM, and IR printing. These form a backend-agnostic pass toolbox for Python callers.
  - **CN:** `init_triton_passes_common` 注册通用的 MLIR 优化，如 SCCP、符号 DCE、内联、规范化、CSE、LICM 以及 IR 打印。这些为 Python 调用者提供了与后端无关的 pass 工具箱。
- **Lines 44-56**
  - **EN:** `init_triton_passes_ttir` binds TTIR-specific transformations, including combine/reorder/canonical pointer-descriptor rewrites and loop optimizations. `add_convert_to_ttgpuir` is notable because it uses an options-wrapper macro, meaning the pass factory expects a small configuration aggregate.
  - **CN:** `init_triton_passes_ttir` 绑定 TTIR 专用变换，包括 combine、broadcast 重排、张量描述符转指针重写以及循环优化。`add_convert_to_ttgpuir` 很关键，因为它使用了 options-wrapper 宏，说明该 pass 工厂接收的是一个小型配置聚合体。
- **Lines 58-112**
  - **EN:** `init_triton_passes_ttgpuir` is the largest registration block. It exposes TritonGPU optimization, scheduling, memory-allocation, async-copy, warp-specialization, sanitizer, and LLVM-canonicalization passes. Two manual `m.def` bindings stand out: one builds `TritonInstrumentPrepareConSanCapturesOptions` explicitly to set `target`, and the other adds `createCanonicalizeLLVMIR()` as a nested pass on `mlir::LLVM::LLVMFuncOp`.
  - **CN:** `init_triton_passes_ttgpuir` 是最大的注册块。它暴露了 TritonGPU 的优化、调度、内存分配、异步拷贝、warp specialization、sanitizer 以及 LLVM 规范化 pass。其中两个手写 `m.def` 值得注意：一个显式构造 `TritonInstrumentPrepareConSanCapturesOptions` 来设置 `target`；另一个把 `createCanonicalizeLLVMIR()` 作为 `mlir::LLVM::LLVMFuncOp` 上的嵌套 pass 加入。
- **Lines 114-125**
  - **EN:** `init_plugin_passes` enumerates runtime-loaded Triton plugins and publishes each plugin pass into Python by name. The lambda forwards a `std::vector<std::string>` argument list to the plugin’s `addPass` callback, making the Python API extensible without recompiling this file.
  - **CN:** `init_plugin_passes` 枚举运行时加载的 Triton 插件，并按名称把每个插件 pass 发布到 Python。lambda 把 `std::vector<std::string>` 参数列表转发给插件的 `addPass` 回调，从而让 Python API 可以在不重新编译此文件的情况下扩展。
- **Lines 127-134**
  - **EN:** `init_triton_passes_convert` binds core MLIR lowering passes from SCF/CF/Index/Arith/NVVM toward LLVM. This submodule acts as the bridge from higher-level IR into LLVM-compatible IR.
  - **CN:** `init_triton_passes_convert` 绑定从 SCF/CF/Index/Arith/NVVM 向 LLVM 降低的核心 MLIR pass。该子模块充当从高层 IR 过渡到 LLVM 兼容 IR 的桥梁。
- **Lines 136-140**
  - **EN:** `init_triton_passes_llvmir` exposes debug-info related LLVM IR passes (`di_scope` and `di_local_variable`). Their placement here shows the binding layer also covers post-lowering LLVM instrumentation utilities.
  - **CN:** `init_triton_passes_llvmir` 暴露与调试信息相关的 LLVM IR pass（`di_scope` 与 `di_local_variable`）。它们放在这里表明该绑定层也覆盖了 lowering 之后的 LLVM 插桩工具。
- **Lines 142-151**
  - **EN:** `init_gluon_passes` registers passes for the Gluon dialect, including automatic encoding resolution, canonicalization, inlining, and inference of coalesced encodings. The local alias `namespace gluon = mlir::triton::gluon;` keeps the registration block readable.
  - **CN:** `init_gluon_passes` 注册 Gluon 方言相关的 pass，包括自动编码解析、规范化、内联以及合并编码推断。局部别名 `namespace gluon = mlir::triton::gluon;` 让注册代码更易读。
- **Lines 153-164**
  - **EN:** `init_triton_passes` is the public entry point declared in `passes.h`. It assembles all submodules—`analysis`, `common`, `convert`, `ttir`, `ttgpuir`, `llvmir`, `gluon`, and `plugin`—under one Python module tree.
  - **CN:** `init_triton_passes` 是在 `passes.h` 中声明的公共入口。它把 `analysis`、`common`、`convert`、`ttir`、`ttgpuir`、`llvmir`、`gluon` 和 `plugin` 等子模块组装到统一的 Python 模块树下。

## Key Concepts / 关键概念
- **EN:** The file is a pybind11 registration layer that maps C++ pass factories into a structured Python API.
- **CN:** 该文件是一个 pybind11 注册层，用于把 C++ pass 工厂映射为结构化的 Python API。
- **EN:** Passes are grouped by IR level or subsystem, which mirrors Triton’s compilation pipeline from TTIR to TritonGPU to LLVM IR.
- **CN:** 这些 pass 按 IR 层级或子系统分组，映射了 Triton 从 TTIR 到 TritonGPU 再到 LLVM IR 的编译流水线。
- **EN:** Manual bindings are used only when macro wrappers are insufficient, such as when an options object must be populated explicitly or a nested pass target differs from the default.
- **CN:** 只有在宏包装不足以表达时才使用手写绑定，例如需要显式填充 options 对象，或嵌套 pass 的目标类型与默认情况不同。
- **EN:** Plugin loading makes the pass API open-ended, allowing external extensions to inject new Python-visible passes.
- **CN:** 插件加载使 pass API 具备开放性，允许外部扩展注入新的 Python 可见 pass。

## Dependencies / 依赖关系
- **EN:** Depends on `passes.h` macros for concise binding generation.
- **CN:** 依赖 `passes.h` 中的宏来简化绑定生成。
- **EN:** Depends on MLIR pass infrastructure (`PassManager`, standard transform/lowering pass factories).
- **CN:** 依赖 MLIR 的 pass 基础设施（`PassManager`、标准变换/降低 pass 工厂）。
- **EN:** Depends on Triton analysis and pass-factory headers for TTIR, TritonGPU, instrumentation, Gluon, and LLVM IR support.
- **CN:** 依赖 Triton 的分析与 pass 工厂头文件，以支持 TTIR、TritonGPU、插桩、Gluon 和 LLVM IR。
- **EN:** Depends on pybind11 for module/class/function exposure and on plugin utilities for runtime discovery of external passes.
- **CN:** 依赖 pybind11 来暴露模块/类/函数，也依赖插件工具来在运行时发现外部 pass。
