# Triton Comprehensive Tutorial / Triton 全面教程

**EN:** This tutorial is a comprehensive bilingual guide to the Triton repository at `/root/xw/triton/`.
**CN:** 本教程是面向 `/root/xw/triton/` 仓库的中英双语全面指南。

**EN:** It is grounded in the repository files requested by the task: `README.md`, `docs/getting-started/installation.rst`, `docs/programming-guide/chapter-1/introduction.rst`, `chapter-2/related-work.rst`, `chapter-3/debugging.rst`, the Python tutorial series, the Gluon tutorial series, `python/triton/language/core.py`, `python/triton/compiler/compiler.py`, `python/triton/runtime/jit.py`, `python/triton/runtime/autotuner.py`, and `CONTRIBUTING.md`.
**CN:** 内容基于任务要求阅读的仓库文件：`README.md`、`docs/getting-started/installation.rst`、`docs/programming-guide/chapter-1/introduction.rst`、`chapter-2/related-work.rst`、`chapter-3/debugging.rst`、Python 教程系列、Gluon 教程系列、`python/triton/language/core.py`、`python/triton/compiler/compiler.py`、`python/triton/runtime/jit.py`、`python/triton/runtime/autotuner.py` 以及 `CONTRIBUTING.md`。

**EN:** The intended audience is a practitioner who wants both a user-level programming guide and a code-reader’s map of Triton internals.
**CN:** 目标读者既包括想学会编写 Triton 内核的使用者，也包括想阅读 Triton 内部实现的工程师。

## Source Files Read / 已阅读源文件

```
README.md
CONTRIBUTING.md
docs/getting-started/installation.rst
docs/programming-guide/chapter-1/introduction.rst
docs/programming-guide/chapter-2/related-work.rst
docs/programming-guide/chapter-3/debugging.rst
docs/programming-guide/chapter-3/fpsan.rst
docs/python-api/triton.testing.rst
python/tutorials/01-vector-add.py
python/tutorials/02-fused-softmax.py
python/tutorials/03-matrix-multiplication.py
python/tutorials/04-low-memory-dropout.py
python/tutorials/05-layer-norm.py
python/tutorials/06-fused-attention.py
python/tutorials/07-extern-functions.py
python/tutorials/08-grouped-gemm.py
python/tutorials/09-persistent-matmul.py
python/tutorials/10-block-scaled-matmul.py
python/tutorials/11-programmatic-dependent-launch.py
python/tutorials/gluon/01-intro.py
python/tutorials/gluon/02-layouts.py
python/tutorials/gluon/03-async-copy.py
python/tutorials/gluon/04-tma.py
python/tutorials/gluon/05-wgmma.py
python/tutorials/gluon/06-tcgen05.py
python/tutorials/gluon/07-persistence.py
python/tutorials/gluon/08-warp-specialization.py
python/tutorials/gluon/09-tma-gather-scatter.py
python/tutorials/gluon/10-tcgen05-copy.py
python/tutorials/gluon/11-tcgen05-mma-scaled.py
python/tutorials/gluon/12-cluster-launch-control.py
python/tutorials/gluon/13-conv-im2col.py
python/tutorials/gluon/14-multicta.py
python/triton/language/core.py
python/triton/compiler/compiler.py
python/triton/runtime/jit.py
python/triton/runtime/autotuner.py
python/triton/runtime/cache.py
python/triton/runtime/driver.py
python/triton/backends/compiler.py
python/triton/backends/__init__.py
python/triton_kernels/triton_kernels/matmul.py
python/triton_kernels/triton_kernels/reduce.py
python/triton_kernels/triton_kernels/topk.py
python/triton_kernels/triton_kernels/numerics.py
python/triton_kernels/triton_kernels/tensor.py
python/triton_kernels/triton_kernels/distributed.py
examples/plugins/README.md
examples/plugins/TritonPlugin.cpp
```

## 1. Introduction / 简介

- **EN:** Triton is described in `README.md` as a language and compiler for writing highly efficient custom Deep Learning primitives.
- **CN:** 在 `README.md` 中，Triton 被描述为一个用于编写高效自定义深度学习原语的语言与编译器。

- **EN:** The project exists because writing every non-standard kernel in CUDA is productive for experts but costly for most model engineers.
- **CN:** Triton 出现的原因在于：用 CUDA 为每个非标准算子手写内核虽然强大，但对大多数模型工程师来说开发成本很高。

- **EN:** Triton tries to provide higher productivity than CUDA while keeping more flexibility than many DSLs and schedule languages.
- **CN:** Triton 试图同时实现比 CUDA 更高的开发效率，以及比许多 DSL 和调度语言更高的灵活性。

- **EN:** The documentation points to a blocked-programming view of GPU execution rather than the traditional scalar-thread mental model.
- **CN:** 文档强调的是“块化程序”的 GPU 执行观，而不是传统的“标量线程”心智模型。

- **EN:** In `docs/programming-guide/chapter-1/introduction.rst`, Triton contrasts CUDA’s “scalar program, blocked threads” with Triton’s “blocked program, scalar threads”.
- **CN:** 在 `docs/programming-guide/chapter-1/introduction.rst` 中，Triton 将 CUDA 的“标量程序、线程分块”与 Triton 的“块化程序、标量线程”进行对比。

- **EN:** That difference matters because matrix multiplication, attention, normalization, and many fused kernels are naturally expressed as tiles.
- **CN:** 这种差异很重要，因为矩阵乘、注意力、归一化和许多融合内核天然就是以 tile 为单位表达的。

- **EN:** Instead of asking each thread to reason about a single scalar, Triton asks each program instance to reason about a block of values.
- **CN:** Triton 不是让每个线程围绕单个标量思考，而是让每个程序实例围绕一块数据思考。

- **EN:** This often makes pointer arithmetic, shared-memory planning, and fusion patterns easier to express in Python.
- **CN:** 这样通常能让指针计算、共享内存规划和融合模式更容易用 Python 表达。

- **EN:** Compared with CUDA, Triton gives up some low-level explicitness in exchange for a language that is closer to tensor algebra and block scheduling.
- **CN:** 与 CUDA 相比，Triton 用一部分底层显式控制，换来了更接近张量代数和块调度的语言抽象。

- **EN:** Compared with pure polyhedral systems, Triton is deliberately less restrictive and better suited to irregular or manually scheduled block computations.
- **CN:** 与纯 polyhedral 系统相比，Triton 有意减少约束，更适合不规则或需要手动调度的块级计算。

- **EN:** Compared with scheduling languages such as Halide and TVM, Triton often exposes the blocked computation directly instead of separating algorithm and schedule into two different languages.
- **CN:** 与 Halide、TVM 这类调度语言相比，Triton 往往直接暴露块化计算本身，而不是把算法和调度拆成两种语言。

- **EN:** The repository also shows Triton’s evolution: regular Triton tutorials teach productive kernels, while `python/tutorials/gluon/` teaches hardware-near control.
- **CN:** 仓库也展示了 Triton 的演进：普通 Triton 教程强调高生产力内核，而 `python/tutorials/gluon/` 则强调接近硬件的精细控制。

- **EN:** A practical way to think about Triton is: use Triton when vendor libraries are too rigid, but you still want a compiler and runtime to handle most lowering work.
- **CN:** 对 Triton 的一个实用理解是：当厂商库太僵硬，但你又希望编译器和运行时替你完成大部分 lowering 工作时，就应该使用 Triton。

- **EN:** The README cites the MAPL 2019 paper “Triton: An Intermediate Language and Compiler for Tiled Neural Network Computations” as the conceptual foundation.
- **CN:** README 引用了 MAPL 2019 论文《Triton: An Intermediate Language and Compiler for Tiled Neural Network Computations》，把它作为项目的理论基础。

- **EN:** Today Triton is not only an OpenAI-origin project but also a broader community repository with documentation, tests, backends, Gluon, and an out-of-tree plugin story.
- **CN:** 今天的 Triton 不仅源自 OpenAI，也已经发展为一个包含文档、测试、后端、Gluon 和树外插件机制的社区型仓库。

- **EN:** This tutorial therefore covers both how to write kernels and how the repository is organized to make those kernels compile and run.
- **CN:** 因此，本教程既讲如何写内核，也讲这个仓库如何组织起来以支持这些内核的编译与运行。

```text
CUDA mental model:
  many scalar threads -> cooperatively cover a tile

Triton mental model:
  one program instance owns a tile -> compiler maps it to threads/warps/SM resources
```

## 2. Architecture Overview / 架构概览

- **EN:** Triton’s user-visible entrypoint is a Python DSL, but the implementation is a multi-stage compiler pipeline.
- **CN:** Triton 对用户暴露的是 Python DSL，但其实现本质上是一个多阶段编译流水线。

- **EN:** The task description summarizes the canonical lowering path as: Python DSL → Triton IR → TritonGPU IR → LLVM IR → PTX/AMDGCN.
- **CN:** 任务描述给出的标准 lowering 路径是：Python DSL → Triton IR → TritonGPU IR → LLVM IR → PTX/AMDGCN。

- **EN:** `python/triton/compiler/compiler.py` turns a kernel into an `ASTSource` or `IRSource`, computes backend options, derives a cache key, and runs stage functions in order.
- **CN:** `python/triton/compiler/compiler.py` 会把内核包装为 `ASTSource` 或 `IRSource`，计算后端选项，生成缓存键，并按顺序运行各阶段函数。

- **EN:** `ASTSource.make_ir()` lowers Python AST to TTIR through `ast_to_ttir`, while `IRSource` allows starting from files such as `.ttir`, `.ttgir`, `.llir`, or `.ptx`.
- **CN:** `ASTSource.make_ir()` 通过 `ast_to_ttir` 把 Python AST 降到 TTIR，而 `IRSource` 则允许从 `.ttir`、`.ttgir`、`.llir`、`.ptx` 等文件直接开始。

- **EN:** The compiler asks the selected backend to populate a `stages` dictionary through `backend.add_stages(stages, options, src.language)`.
- **CN:** 编译器会调用选定后端的 `backend.add_stages(stages, options, src.language)` 来填充 `stages` 字典。

- **EN:** That means the exact pipeline is backend-defined, but the orchestration, caching, dumping, and override logic are centralized in `compiler.py`.
- **CN:** 这意味着具体流水线由后端定义，但调度、缓存、dump 和 override 逻辑集中在 `compiler.py` 中统一处理。

- **EN:** When the source is already an IR file, `compiler.py` deliberately skips passes for that starting stage so that IR-level debugging and tests are easier.
- **CN:** 当输入已经是某级 IR 文件时，`compiler.py` 会刻意跳过该起始阶段的 pass，从而便于做 IR 级调试与测试。

- **EN:** The repository tree confirms dedicated dialect implementations in `lib/Dialect/Triton`, `lib/Dialect/TritonGPU`, `lib/Dialect/TritonNvidiaGPU`, `lib/Dialect/Gluon`, and `lib/Dialect/TritonInstrument`.
- **CN:** 仓库目录清楚表明存在专门的方言实现：`lib/Dialect/Triton`、`lib/Dialect/TritonGPU`、`lib/Dialect/TritonNvidiaGPU`、`lib/Dialect/Gluon` 与 `lib/Dialect/TritonInstrument`。

- **EN:** The main lowering bridges are visible in `lib/Conversion/TritonToTritonGPU`, `lib/Conversion/TritonGPUToLLVM`, and `lib/Conversion/TritonInstrumentToLLVM`.
- **CN:** 主要 lowering 桥接也能在 `lib/Conversion/TritonToTritonGPU`、`lib/Conversion/TritonGPUToLLVM` 和 `lib/Conversion/TritonInstrumentToLLVM` 中看到。

- **EN:** Triton IR is the block-level language closest to the Python DSL.
- **CN:** Triton IR 是最接近 Python DSL 的块级中间表示。

- **EN:** TritonGPU IR adds GPU execution structure, layout, memory movement, and other information required for GPU mapping.
- **CN:** TritonGPU IR 增加了 GPU 映射所需的执行结构、布局、内存移动等信息。

- **EN:** TritonNvidiaGPU holds NVIDIA-specific concepts that do not belong in a generic GPU dialect, such as hardware-specific tensor-core paths.
- **CN:** TritonNvidiaGPU 承载不适合放入通用 GPU 方言的 NVIDIA 专属概念，例如特定张量核路径。

- **EN:** Gluon uses the same broad compiler stack but adds a lower-level programming interface and its own dialect support in `lib/Dialect/Gluon`.
- **CN:** Gluon 复用同一条大体编译链路，但提供更底层的编程接口，并在 `lib/Dialect/Gluon` 中拥有自己的方言支持。

- **EN:** Instrumentation features such as FpSan and global sanitizer are reflected by the `TritonInstrument` dialect and corresponding conversion tests.
- **CN:** FpSan、global sanitizer 等插桩功能则体现为 `TritonInstrument` 方言及其对应转换测试。

- **EN:** The runtime-visible result of compilation is a `CompiledKernel`, which materializes metadata, packed launch metadata, and generated assembly artifacts.
- **CN:** 编译在运行时层面产出的对象是 `CompiledKernel`，它封装了元数据、打包后的 launch metadata 以及生成的汇编产物。

- **EN:** The architecture is therefore not “Python magically becomes PTX”; it is a disciplined stage pipeline with MLIR modules, backend options, cache managers, and launchers.
- **CN:** 因此，Triton 并不是“Python 神奇地变成 PTX”，而是一条由 MLIR 模块、后端选项、缓存管理器和启动器组成的严谨流水线。

- **EN:** Understanding this architecture is the key to understanding why Triton can support both productive kernels and extremely hardware-aware Gluon kernels inside one repository.
- **CN:** 理解这套架构，是理解 Triton 为什么能在同一仓库里同时支持高生产力内核和极度贴近硬件的 Gluon 内核的关键。

```text
Python kernel (@triton.jit / @gluon.jit)
        ↓
frontend AST + specialization
        ↓
TTIR (Triton IR)
        ↓
TTGIR / backend-specific GPU IR
        ↓
LLVM IR
        ↓
PTX (NVIDIA) / AMDGCN (AMD)
        ↓
binary + launch metadata + cache entry
```

## 3. Installation / 安装

- **EN:** The simplest path from `README.md` and `docs/getting-started/installation.rst` is `pip install triton`.
- **CN:** 根据 `README.md` 和 `docs/getting-started/installation.rst`，最简单的安装方式是 `pip install triton`。

- **EN:** Binary wheels are documented as available for CPython 3.10 through 3.14.
- **CN:** 文档说明二进制 wheel 覆盖 CPython 3.10 到 3.14。

- **EN:** For source builds, the standard recipe is: clone the repo, install `python/requirements.txt`, then run `pip install -e .`.
- **CN:** 源码构建的标准步骤是：克隆仓库、安装 `python/requirements.txt`，再执行 `pip install -e .`。

- **EN:** The docs also show a virtual environment workflow using `python -m venv .venv --prompt triton`.
- **CN:** 文档也给出了使用 `python -m venv .venv --prompt triton` 的虚拟环境流程。

- **EN:** If LLVM is not present locally, the build can download official LLVM static libraries automatically.
- **CN:** 如果本机没有 LLVM，构建流程可以自动下载官方 LLVM 静态库。

- **EN:** If you need a custom LLVM, `README.md` recommends `make dev-install-llvm` as the convenient path.
- **CN:** 如果需要自定义 LLVM，`README.md` 推荐使用 `make dev-install-llvm` 作为便捷入口。

- **EN:** The README further explains that Triton is tied to a specific LLVM revision, with the revision recorded in `cmake/llvm-hash.txt`.
- **CN:** README 进一步说明 Triton 依赖特定 LLVM 修订版本，版本号记录在 `cmake/llvm-hash.txt` 中。

- **EN:** Useful build knobs include `TRITON_BUILD_WITH_CLANG_LLD=true` and `TRITON_BUILD_WITH_CCACHE=true`.
- **CN:** 实用的构建开关包括 `TRITON_BUILD_WITH_CLANG_LLD=true` 与 `TRITON_BUILD_WITH_CCACHE=true`。

- **EN:** `TRITON_HOME` changes the location of the `.triton` directory used for cache and downloads.
- **CN:** `TRITON_HOME` 可以修改 `.triton` 目录的位置，该目录用于缓存和下载。

- **EN:** `MAX_JOBS` can reduce build parallelism when memory is limited.
- **CN:** 当机器内存紧张时，可以用 `MAX_JOBS` 限制构建并行度。

- **EN:** The README explicitly notes that `--no-build-isolation` can make repeated local builds faster.
- **CN:** README 明确提到 `--no-build-isolation` 可以让重复本地构建更快。

- **EN:** After building, the repo suggests `make dev-install`, `make test`, and `make test-nogpu` for installation and testing.
- **CN:** 构建完成后，仓库建议使用 `make dev-install`、`make test` 与 `make test-nogpu` 进行安装和测试。

- **EN:** Because Triton targets GPUs and specific backends, installation should always be thought of as “Python package + backend compatibility + driver/toolchain availability”.
- **CN:** 由于 Triton 面向 GPU 和特定后端，安装不能只看 Python 包，还要同时考虑后端兼容性、驱动和工具链。

- **EN:** A practical smoke test after install is to run `python/tutorials/01-vector-add.py` or a short `@triton.jit` kernel from a Python REPL.
- **CN:** 安装后的实用冒烟测试是运行 `python/tutorials/01-vector-add.py`，或者在 Python REPL 中执行一个简短的 `@triton.jit` 内核。

```bash
# Stable package
pip install triton

# From source
git clone https://github.com/triton-lang/triton.git
cd triton
pip install -r python/requirements.txt
pip install -e .

# One common verification path
make dev-install
make test-nogpu
```

## 4. Core Language / 核心语言

- **EN:** The central frontend namespace is `triton.language`, usually imported as `import triton.language as tl`.
- **CN:** 核心前端命名空间是 `triton.language`，通常写作 `import triton.language as tl`。

- **EN:** In `python/triton/language/core.py`, builtins are marked with the `@builtin` decorator and routed through a semantic layer.
- **CN:** 在 `python/triton/language/core.py` 中，内建操作通过 `@builtin` 标记，并经由语义层实现。

- **EN:** The `constexpr` class represents compile-time-known values and powers many meta-parameters such as `BLOCK_SIZE`, `BLOCK_M`, and `ACTIVATION`.
- **CN:** `constexpr` 类表示编译期已知的值，是 `BLOCK_SIZE`、`BLOCK_M`、`ACTIVATION` 等元参数的基础。

- **EN:** `tl.program_id(axis)` returns the program instance index on axis 0, 1, or 2; `tl.num_programs(axis)` tells how many instances exist along that axis.
- **CN:** `tl.program_id(axis)` 返回 0/1/2 轴上的程序实例编号；`tl.num_programs(axis)` 返回该轴上的程序总数。

- **EN:** `tl.arange(start, end)` creates a block of contiguous indices and is one of the most common building blocks in every tutorial.
- **CN:** `tl.arange(start, end)` 生成连续索引块，是几乎每个教程中最常见的构件之一。

- **EN:** `core.py` documents `tl.arange` as requiring power-of-two endpoints within Triton’s maximum tensor-numel constraints.
- **CN:** `core.py` 对 `tl.arange` 的文档说明：其端点要满足 2 的幂等约束，并受 Triton 最大张量元素数限制。

- **EN:** The `dtype`, `pointer_type`, `block_type`, and `tensor` classes in `core.py` are the language-level value system that sits above MLIR handles.
- **CN:** `core.py` 中的 `dtype`、`pointer_type`、`block_type` 和 `tensor` 类构成了位于 MLIR handle 之上的语言值系统。

- **EN:** `tl.load` is overloaded for scalar pointers, blocks of pointers, and block pointers / descriptors.
- **CN:** `tl.load` 同时支持标量指针、指针块，以及 block pointer / descriptor 形式。

- **EN:** For masked loads, `mask` suppresses the access and `other` provides the replacement value.
- **CN:** 在掩码加载中，`mask` 用于抑制访问，而 `other` 提供被屏蔽位置的替代值。

- **EN:** For descriptor or block-pointer style loads, `boundary_check` and `padding_option` control out-of-bounds behavior.
- **CN:** 对于 descriptor 或 block pointer 风格的加载，越界行为由 `boundary_check` 和 `padding_option` 控制。

- **EN:** `tl.store` mirrors `tl.load`, supporting scalar pointers, pointer blocks, and block pointers.
- **CN:** `tl.store` 与 `tl.load` 对称，同样支持标量指针、指针块和 block pointer。

- **EN:** `cache_modifier`, `eviction_policy`, and `volatile` expose low-level backend cache hints, especially relevant on NVIDIA PTX.
- **CN:** `cache_modifier`、`eviction_policy` 和 `volatile` 暴露了底层缓存提示，在 NVIDIA PTX 上尤其有意义。

- **EN:** `tl.dot` performs 2D or batched 3D block matrix products and supports precision controls such as `input_precision="tf32"` or `"ieee"`.
- **CN:** `tl.dot` 支持 2D 或批量 3D 块矩阵乘，并允许通过 `input_precision="tf32"` 或 `"ieee"` 控制精度路径。

- **EN:** `tl.dot_scaled` is the microscaling-oriented variant used in block-scaled matmul examples.
- **CN:** `tl.dot_scaled` 是面向 microscaling 的变体，在 block-scaled matmul 教程中有直接应用。

- **EN:** `tl.reduce(input, axis, combine_fn, keep_dims=False)` lets you define custom reductions; the combine function itself must be a Triton JIT function.
- **CN:** `tl.reduce(input, axis, combine_fn, keep_dims=False)` 用于定义自定义归约；其中 combine 函数本身也必须是 Triton JIT 函数。

- **EN:** Common reductions such as `tl.sum` and `tl.max` are layered on top of this reduction machinery and appear heavily in softmax, layer norm, and attention.
- **CN:** `tl.sum`、`tl.max` 等常见归约就是建立在这套机制之上，在 softmax、layer norm 和 attention 中大量出现。

- **EN:** `tl.make_block_ptr` still exists, but `core.py` now warns that it is deprecated in favor of `TensorDescriptor` or `tl.make_tensor_descriptor`.
- **CN:** `tl.make_block_ptr` 仍然存在，但 `core.py` 已明确警告它正在被 `TensorDescriptor` 或 `tl.make_tensor_descriptor` 取代。

- **EN:** `tl.advance` does not mutate in place; the warning text in `core.py` explicitly says you must assign the returned pointer.
- **CN:** `tl.advance` 不会原地修改；`core.py` 的警告文字明确指出必须接收它返回的新指针。

- **EN:** `tl.make_tensor_descriptor` creates a descriptor object from a base pointer, shape, strides, and block shape.
- **CN:** `tl.make_tensor_descriptor` 可以从基指针、shape、stride 和 block shape 创建 descriptor 对象。

- **EN:** The descriptor documentation in `core.py` notes that the base must be 16-byte aligned, leading dimensions must satisfy alignment constraints, and only 2D–5D tensors are currently supported.
- **CN:** `core.py` 中对 descriptor 的说明指出：base 必须 16 字节对齐，前导维 stride 要满足对齐要求，目前仅支持 2D–5D 张量。

- **EN:** On NVIDIA GPUs with TMA support, tensor-descriptor loads and stores can be backed by TMA hardware.
- **CN:** 在支持 TMA 的 NVIDIA GPU 上，tensor descriptor 的 load/store 可以直接走 TMA 硬件路径。

- **EN:** `load_tensor_descriptor` and `store_tensor_descriptor` are thin helpers that call descriptor methods.
- **CN:** `load_tensor_descriptor` 和 `store_tensor_descriptor` 是调用 descriptor 方法的薄封装。

- **EN:** Debugging builtins in `core.py` include `static_print`, `static_assert`, `device_print`, and `device_assert`.
- **CN:** `core.py` 中的调试内建操作包括 `static_print`、`static_assert`、`device_print` 和 `device_assert`。

- **EN:** A good mental model is that Triton tensors are compile-time-shaped blocks that eventually become MLIR values with attached layout and type information.
- **CN:** 一个很好的心智模型是：Triton tensor 是带有编译期 shape 的数据块，最终会变成带布局与类型信息的 MLIR value。

```python
@triton.jit
def add_kernel(x_ptr, y_ptr, out_ptr, n_elements, BLOCK_SIZE: tl.constexpr):
    pid = tl.program_id(axis=0)
    offsets = pid * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = offsets < n_elements
    x = tl.load(x_ptr + offsets, mask=mask, other=0)
    y = tl.load(y_ptr + offsets, mask=mask, other=0)
    tl.store(out_ptr + offsets, x + y, mask=mask)
```

```python
@triton.jit
def descriptor_example(ptr, M, N, BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr):
    desc = tl.make_tensor_descriptor(
        ptr,
        shape=[M, N],
        strides=[N, 1],
        block_shape=[BLOCK_M, BLOCK_N],
    )
    tile = tl.load_tensor_descriptor(desc, [0, 0])
    tl.store_tensor_descriptor(desc, [0, 0], tile)
```

## 5. Tutorial Walkthrough / 教程详解

- **EN:** The regular tutorial series in `python/tutorials/` teaches Triton by building real kernels rather than toy syntax examples.
- **CN:** `python/tutorials/` 中的常规教程系列是通过真实内核来讲 Triton，而不是只讲玩具级语法。

- **EN:** The progression is deliberate: vector add establishes the model, softmax introduces reductions, matmul introduces tiling and autotune, and later files move into fused transformer-era kernels.
- **CN:** 这一系列的节奏是刻意设计的：vector add 建立模型，softmax 引入归约，matmul 引入分块与 autotune，后续文件逐步进入 Transformer 时代的融合内核。

- **EN:** You should read the tutorial files as both runnable notebooks and style guides for Triton kernel structure.
- **CN:** 这些教程文件既是可运行示例，也可以当作 Triton 内核结构的风格指南来阅读。

- **EN:** A recurring pattern is: compute program id, build offsets, mask tails, load tiles, compute on tiles, then store results.
- **CN:** 贯穿全文的共同模式是：计算 program id，构造 offsets，处理尾部 mask，加载 tile，对 tile 计算，再写回结果。

- **EN:** Another recurring pattern is a Python wrapper that allocates output tensors, computes grid dimensions, launches the kernel, and checks or benchmarks against PyTorch.
- **CN:** 另一个反复出现的模式是：用 Python 包装函数分配输出张量、计算 grid、启动内核，并与 PyTorch 做校验或基准测试。

- **EN:** When you learn Triton, the highest-value habit is to compare the kernel body and the wrapper together, because launch policy is part of the algorithm.
- **CN:** 学习 Triton 时最有价值的习惯之一，就是把内核体和外层包装函数一起读，因为 launch 策略本身就是算法的一部分。

### 5.1 Vector Add / 向量加法

- **EN:** Primary file: `python/tutorials/01-vector-add.py`.
- **CN:** 主要文件：`python/tutorials/01-vector-add.py`。

- **EN:** This file is the minimal Triton kernel: a 1D vector addition used to explain the blocked SPMD model.
- **CN:** 该文件是最小 Triton 内核示例：用一维向量加法解释块化 SPMD 模型。

- **EN:** Key kernels or functions: `add_kernel, add`.
- **CN:** 关键内核/函数：`add_kernel, add`。

- **EN:** Key Triton features: `@triton.jit`, `tl.program_id`, `tl.arange`, `tl.load`, `tl.store`, masking, `triton.cdiv`.
- **CN:** 关键 Triton 特性：`@triton.jit`, `tl.program_id`, `tl.arange`, `tl.load`, `tl.store`, masking, `triton.cdiv`。

- **EN:** The wrapper computes `grid = lambda meta: (triton.cdiv(n_elements, meta["BLOCK_SIZE"]),)` and launches a 1D grid.
- **CN:** 包装函数使用 `grid = lambda meta: (triton.cdiv(n_elements, meta["BLOCK_SIZE"]),)`，即一维 grid 启动。

- **EN:** The file ends with `triton.testing.perf_report` and a simple GB/s benchmark, establishing the repo’s benchmark idiom.
- **CN:** 文件结尾使用 `triton.testing.perf_report` 做 GB/s 基准，建立了本仓库常见的 benchmark 写法。

- **EN:** The notable code detail is the masked tail: `offsets < n_elements` protects the last partial block.
- **CN:** 最值得注意的代码细节是尾块掩码：`offsets < n_elements` 用来保护最后一个不完整块。

- **EN:** If you only remember one Triton pattern, remember this file: program id + arange + mask + load/compute/store.
- **CN:** 如果只记住一个 Triton 模式，就记住这个文件：program id + arange + mask + load/compute/store。

### 5.2 Fused Softmax / 融合 Softmax

- **EN:** Primary file: `python/tutorials/02-fused-softmax.py`.
- **CN:** 主要文件：`python/tutorials/02-fused-softmax.py`。

- **EN:** This tutorial moves from elementwise ops to a bandwidth-bound reduction kernel and explains why fusion matters.
- **CN:** 该教程从逐元素计算提升到带宽受限的归约内核，并解释了为什么融合很重要。

- **EN:** Key kernels or functions: `softmax_kernel, softmax, naive_softmax`.
- **CN:** 关键内核/函数：`softmax_kernel, softmax, naive_softmax`。

- **EN:** Key Triton features: `tl.max`, `tl.sum`, `tl.exp`, masked loads with `other=-inf`, `tl.num_programs`, persistent row stepping.
- **CN:** 关键 Triton 特性：`tl.max`, `tl.sum`, `tl.exp`, masked loads with `other=-inf`, `tl.num_programs`, persistent row stepping。

- **EN:** Each program handles one or more rows, and the kernel advances rows by `row_step = tl.num_programs(0)`.
- **CN:** 每个程序处理一行或多行，内核通过 `row_step = tl.num_programs(0)` 在行维度上前进。

- **EN:** The tutorial contrasts a naive multi-kernel softmax with a fused version that keeps a row in SRAM when possible.
- **CN:** 教程对比了朴素多内核 softmax 与融合版本，后者在可能时把一整行保留在片上 SRAM 中。

- **EN:** A concrete trick is `BLOCK_SIZE = triton.next_power_of_2(n_cols)`, which simplifies vectorized row processing.
- **CN:** 一个具体技巧是 `BLOCK_SIZE = triton.next_power_of_2(n_cols)`，这让按行向量化处理更容易。

- **EN:** Read this file whenever you want to understand reduction structure, numerical stabilization, and persistent looping in Triton.
- **CN:** 如果你想理解 Triton 中的归约结构、数值稳定化与持久化循环，这个文件是首选。

### 5.3 Matrix Multiplication / 矩阵乘法

- **EN:** Primary file: `python/tutorials/03-matrix-multiplication.py`.
- **CN:** 主要文件：`python/tutorials/03-matrix-multiplication.py`。

- **EN:** This is the flagship tutorial for blocked GEMM and the best single file for learning high-performance Triton style.
- **CN:** 这是 Triton 分块 GEMM 的旗舰教程，也是学习高性能 Triton 风格最重要的单个文件。

- **EN:** Key kernels or functions: `matmul_kernel, matmul, leaky_relu`.
- **CN:** 关键内核/函数：`matmul_kernel, matmul, leaky_relu`。

- **EN:** Key Triton features: `tl.dot`, 2D pointer arithmetic, `tl.assume`, masked boundary loads/stores, `@triton.autotune`, fused activation.
- **CN:** 关键 Triton 特性：`tl.dot`, 2D pointer arithmetic, `tl.assume`, masked boundary loads/stores, `@triton.autotune`, fused activation。

- **EN:** The launch grid covers output tiles, while grouped program ordering with `GROUP_SIZE_M` improves L2 reuse.
- **CN:** launch grid 负责覆盖输出 tile，而 `GROUP_SIZE_M` 对程序进行分组排序以提升 L2 复用。

- **EN:** The file contains many autotune configurations for CUDA and HIP, making it the canonical example of Triton meta-parameter search.
- **CN:** 文件内包含大量 CUDA 与 HIP autotune 配置，是 Triton 元参数搜索的标准示例。

- **EN:** The tile pointers are advanced by `BLOCK_SIZE_K` each loop, while the accumulator stays in higher precision until the store.
- **CN:** 在循环中，tile 指针按 `BLOCK_SIZE_K` 推进，而累加器一直保持高精度直到最后写回。

- **EN:** If you want to understand why Triton is compelling, study this file line by line.
- **CN:** 如果你想真正理解 Triton 的吸引力，请逐行阅读这个文件。

### 5.4 Low-Memory Dropout / 低内存 Dropout

- **EN:** Primary file: `python/tutorials/04-low-memory-dropout.py`.
- **CN:** 主要文件：`python/tutorials/04-low-memory-dropout.py`。

- **EN:** This tutorial shows that Triton is not only about GEMM: it can also redesign operator state to reduce memory traffic.
- **CN:** 该教程说明 Triton 不只适合 GEMM：它还可以通过重构算子状态来降低内存流量。

- **EN:** Key kernels or functions: `_dropout, _seeded_dropout, dropout, seeded_dropout`.
- **CN:** 关键内核/函数：`_dropout, _seeded_dropout, dropout, seeded_dropout`。

- **EN:** Key Triton features: `tl.where`, `tl.rand`, blockwise random generation, mask-free seeded dropout.
- **CN:** 关键 Triton 特性：`tl.where`, `tl.rand`, blockwise random generation, mask-free seeded dropout。

- **EN:** Both variants use a simple 1D launch over element blocks.
- **CN:** 两个变体都使用针对元素块的一维启动方式。

- **EN:** The important optimization is conceptual rather than arithmetic: keep only a seed instead of materializing a full dropout mask tensor.
- **CN:** 这里最重要的优化不是算术本身，而是概念层面的：只保存随机种子，而不是显式存储完整 dropout mask 张量。

- **EN:** The seeded version uses `tl.rand(seed, offsets)` and converts random numbers into a keep mask on the fly.
- **CN:** 带种子的版本用 `tl.rand(seed, offsets)` 在线生成随机数，并即时转成 keep mask。

- **EN:** This file teaches a key Triton lesson: sometimes the best optimization is changing data representation, not only instruction choice.
- **CN:** 这个文件传达了 Triton 的一个关键思想：最好的优化有时是改变数据表示，而不只是换一条指令。

### 5.5 Layer Normalization / 层归一化

- **EN:** Primary file: `python/tutorials/05-layer-norm.py`.
- **CN:** 主要文件：`python/tutorials/05-layer-norm.py`。

- **EN:** This file demonstrates a full training-grade operator with fused forward and custom backward kernels.
- **CN:** 该文件展示了一个训练级算子：融合前向加自定义反向内核。

- **EN:** Key kernels or functions: `_layer_norm_fwd_fused, _layer_norm_bwd_dx_fused, _layer_norm_bwd_dwdb, LayerNorm`.
- **CN:** 关键内核/函数：`_layer_norm_fwd_fused, _layer_norm_bwd_dx_fused, _layer_norm_bwd_dwdb, LayerNorm`。

- **EN:** Key Triton features: rowwise reductions, custom autograd integration, atomics, `tl.debug_barrier`, shared-work decomposition.
- **CN:** 关键 Triton 特性：rowwise reductions, custom autograd integration, atomics, `tl.debug_barrier`, shared-work decomposition。

- **EN:** Forward launches one program per row, and the backward path splits work across multiple kernels for `dx`, `dw`, and `db`.
- **CN:** 前向阶段通常一行对应一个程序，而反向则拆成多个内核分别处理 `dx`、`dw` 和 `db`。

- **EN:** The tutorial benchmarks Triton against PyTorch and Apex, showing how fusion can beat general-purpose framework code.
- **CN:** 教程将 Triton 与 PyTorch、Apex 对比，展示了融合实现如何超过通用框架代码。

- **EN:** A memorable implementation detail is the explicit fused-size guard based on `65536 // x.element_size()`.
- **CN:** 一个很有记忆点的实现细节是基于 `65536 // x.element_size()` 的显式 fused-size 上限判断。

- **EN:** Read this file to learn how Triton moves from inference-style demos to fully differentiated training operators.
- **CN:** 想理解 Triton 如何从推理示例走向可训练算子，请认真阅读这个文件。

### 5.6 Fused Attention / 融合注意力

- **EN:** Primary file: `python/tutorials/06-fused-attention.py`.
- **CN:** 主要文件：`python/tutorials/06-fused-attention.py`。

- **EN:** This is the most complex regular tutorial and a direct bridge from Triton basics to FlashAttention-class kernels.
- **CN:** 这是常规教程中最复杂的一个，也是从 Triton 基础迈向 FlashAttention 级内核的直接桥梁。

- **EN:** Key kernels or functions: `_attn_fwd_inner, _attn_fwd, _attn_bwd_preprocess, _attn_bwd_dkdv, _attn_bwd_dq, _attn_bwd`.
- **CN:** 关键内核/函数：`_attn_fwd_inner, _attn_fwd, _attn_bwd_preprocess, _attn_bwd_dkdv, _attn_bwd_dq, _attn_bwd`。

- **EN:** Key Triton features: `TensorDescriptor`, `tl.dot`, online softmax state (`m_i`, `l_i`), staged loops, FP8 paths, warp specialization flags.
- **CN:** 关键 Triton 特性：`TensorDescriptor`, `tl.dot`, online softmax state (`m_i`, `l_i`), staged loops, FP8 paths, warp specialization flags。

- **EN:** The file uses multi-axis `program_id` indexing across tiles and head/batch dimensions, reflecting real transformer kernel structure.
- **CN:** 文件使用多轴 `program_id` 来遍历 tile、head 和 batch 维度，体现了真实 Transformer 内核的结构。

- **EN:** The tutorial contains architecture-specific tuning and pruning logic for Hopper, Blackwell, CUDA, and HIP paths.
- **CN:** 教程中包含针对 Hopper、Blackwell、CUDA 与 HIP 路径的架构特定调优与剪枝逻辑。

- **EN:** The numerically stable softmax update with `m_i`, `l_i`, and `alpha` is one of the most important patterns in modern attention kernels.
- **CN:** 使用 `m_i`、`l_i` 和 `alpha` 进行数值稳定 softmax 更新，是现代注意力内核最关键的模式之一。

- **EN:** This file is where Triton stops feeling like a simple DSL and starts feeling like a serious kernel engineering environment.
- **CN:** 读到这个文件时，Triton 就不再像一个简单 DSL，而开始像真正的内核工程环境。

### 5.7 External Functions / 外部函数调用

- **EN:** Primary file: `python/tutorials/07-extern-functions.py`.
- **CN:** 主要文件：`python/tutorials/07-extern-functions.py`。

- **EN:** This tutorial explains how a Triton kernel can call backend device libraries such as CUDA libdevice or HIP device libs.
- **CN:** 该教程展示 Triton 内核如何调用 CUDA libdevice 或 HIP device libs 等后端设备库。

- **EN:** Key kernels or functions: `asin_kernel`.
- **CN:** 关键内核/函数：`asin_kernel`。

- **EN:** Key Triton features: `triton.language.extra.libdevice`, `extern_libs`, backend-specific library paths.
- **CN:** 关键 Triton 特性：`triton.language.extra.libdevice`, `extern_libs`, backend-specific library paths。

- **EN:** The kernel keeps the launch logic simple so the tutorial can focus on external function integration.
- **CN:** 该内核保持极简 launch 逻辑，从而让教程把重点放在外部函数集成上。

- **EN:** This file is API-oriented rather than performance-oriented, but it matters when your kernel needs transcendental or vendor-supplied functionality.
- **CN:** 这个文件重在 API，而不是性能；但当你的内核需要超越基本算子、调用厂商提供函数时，它非常关键。

- **EN:** The most practical detail is that the library path is selected differently for CUDA and HIP.
- **CN:** 最实用的细节是：CUDA 与 HIP 的库路径选择方式并不相同。

- **EN:** Triton is not closed over its own builtin ops; it can interoperate with external device functionality when needed.
- **CN:** Triton 并不封闭于自身内建算子；在需要时，它可以与外部设备函数协同工作。

```python
# A simplified blocked GEMM idea from 03-matrix-multiplication.py
for m_tile in range(0, M, BLOCK_SIZE_M):
    for n_tile in range(0, N, BLOCK_SIZE_N):
        acc = 0
        for k_tile in range(0, K, BLOCK_SIZE_K):
            a = load(A[m_tile:m_tile+BM, k_tile:k_tile+BK])
            b = load(B[k_tile:k_tile+BK, n_tile:n_tile+BN])
            acc += dot(a, b)
        store(C[m_tile:m_tile+BM, n_tile:n_tile+BN], acc)
```

## 6. Advanced Topics / 高级主题

- **EN:** The later regular tutorials move from “standard fused kernels” to scheduling, persistence, specialized data formats, and hardware launch control.
- **CN:** 后面的常规教程从“标准融合内核”进一步走向调度、持久化、专用数据格式以及硬件级启动控制。

- **EN:** These files are where Triton starts to overlap directly with runtime policy and architecture-specific features.
- **CN:** 在这些文件中，Triton 开始与运行时策略和架构特定特性直接交汇。

- **EN:** If section 5 taught you how to write fast kernels, section 6 teaches you how to think about keeping SMs busy, feeding tensor cores, and reducing launch overhead.
- **CN:** 如果第 5 节教你如何写出快内核，第 6 节则教你如何思考保持 SM 忙碌、喂饱张量核并降低启动开销。

- **EN:** The themes here—persistent scheduling, grouped work queues, microscaling, and dependent launch—are exactly the kinds of ideas you see in production inference and training kernels.
- **CN:** 这里的主题——持久化调度、分组工作队列、microscaling、依赖式启动——正是生产级训练与推理内核中常见的思想。

- **EN:** You should read these files after you are comfortable with basic tiling, masking, and wrapper-launch logic.
- **CN:** 建议在熟悉基本 tiling、mask 和包装层 launch 逻辑之后，再进入这些文件。

- **EN:** They are also a bridge into Gluon, because they naturally raise questions about memory hierarchy and hardware pipelines that Gluon exposes more explicitly.
- **CN:** 它们也是通往 Gluon 的桥梁，因为这些文件自然会引出对内存层次和硬件流水线的思考，而 Gluon 会更显式地暴露这些内容。

- **EN:** In practice, these are the files you revisit when you need to squeeze the last 10–30% of performance or support a new low-precision format.
- **CN:** 在实践中，当你需要再榨出最后 10–30% 的性能，或者支持新的低精度格式时，通常会重新回到这些文件。

- **EN:** They also demonstrate that “advanced Triton” is increasingly about scheduling and data movement, not only arithmetic expressions.
- **CN:** 这些教程也说明：所谓“高级 Triton”越来越多是在处理调度与数据搬运，而不仅仅是算术表达式。

### 6.1 Grouped GEMM / 分组 GEMM

- **EN:** Primary file: `python/tutorials/08-grouped-gemm.py`.
- **CN:** 主要文件：`python/tutorials/08-grouped-gemm.py`。

- **EN:** This tutorial implements multiple GEMMs under a fixed CTA budget and schedules them on device.
- **CN:** 该教程在固定 CTA 预算下执行多个 GEMM，并把调度逻辑放到设备端完成。

- **EN:** Key kernels or functions: `grouped_matmul_kernel, grouped_matmul_tma_kernel, group_gemm_fn`.
- **CN:** 关键内核/函数：`grouped_matmul_kernel, grouped_matmul_tma_kernel, group_gemm_fn`。

- **EN:** Key Triton features: device arrays of pointers and shapes, `tl.dot`, `tl.multiple_of`, optional TMA descriptors, `@triton.autotune`.
- **CN:** 关键 Triton 特性：device arrays of pointers and shapes, `tl.dot`, `tl.multiple_of`, optional TMA descriptors, `@triton.autotune`。

- **EN:** The grid is effectively tied to the number of SMs, and each CTA repeatedly claims more tiles by incrementing tile indices.
- **CN:** grid 基本与 SM 数量绑定，每个 CTA 通过持续递增 tile index 来领取更多工作。

- **EN:** The tutorial demonstrates why grouped scheduling is useful when you cannot or do not want to launch one kernel per GEMM.
- **CN:** 该教程说明：当你不想或不能为每个 GEMM 单独启动一个内核时，分组调度为什么有价值。

- **EN:** A notable detail is that `(M, N, K)` sizes and strides are passed as device tensors, so scheduling decisions are data-driven.
- **CN:** 值得注意的细节是 `(M, N, K)` 尺寸和 stride 以设备张量形式传入，因此调度是数据驱动的。

- **EN:** Grouped GEMM teaches that Triton can manage a work queue, not just a single regular tensor tile space.
- **CN:** 分组 GEMM 说明 Triton 不只会处理规则 tile 空间，也能管理工作队列。

### 6.2 Persistent Matmul / 持久化矩阵乘

- **EN:** Primary file: `python/tutorials/09-persistent-matmul.py`.
- **CN:** 主要文件：`python/tutorials/09-persistent-matmul.py`。

- **EN:** This file explores persistent kernels, TMA variants, profiling metadata, and matmul scheduling at SM granularity.
- **CN:** 该文件探索了持久化内核、TMA 变体、profiling metadata，以及按 SM 粒度思考 matmul 调度。

- **EN:** Key kernels or functions: `matmul_kernel, matmul_kernel_tma, matmul_kernel_persistent, _compute_pid`.
- **CN:** 关键内核/函数：`matmul_kernel, matmul_kernel_tma, matmul_kernel_persistent, _compute_pid`。

- **EN:** Key Triton features: persistent tile traversal, TMA descriptors, profiler launch metadata, warp specialization toggles.
- **CN:** 关键 Triton 特性：persistent tile traversal, TMA descriptors, profiler launch metadata, warp specialization toggles。

- **EN:** The persistent path launches roughly one CTA per SM and lets each CTA iterate over multiple output tiles.
- **CN:** 持久化路径通常按“一张 SM 一个 CTA”的方式启动，然后让每个 CTA 迭代多个输出 tile。

- **EN:** The tutorial exists because launch overhead and tile residency matter once your matmul is already close to hardware throughput.
- **CN:** 之所以需要这类教程，是因为当 matmul 已经接近硬件极限时，启动开销与 tile 驻留行为都会变得重要。

- **EN:** The file also defines launch metadata for Proton profiling, making it a good example of performance instrumentation integration.
- **CN:** 文件还定义了给 Proton profiler 使用的 launch metadata，因此也是性能插桩集成的好例子。

- **EN:** Persistent kernels are a scheduling technique first and a syntax pattern second.
- **CN:** 持久化内核首先是一种调度技术，其次才是一种代码写法。

### 6.3 Block-Scaled Matmul / 块缩放矩阵乘

- **EN:** Primary file: `python/tutorials/10-block-scaled-matmul.py`.
- **CN:** 主要文件：`python/tutorials/10-block-scaled-matmul.py`。

- **EN:** This tutorial introduces microscaling and shows how Triton can target FP4/FP8-style scaled formats.
- **CN:** 该教程引入 microscaling，并展示 Triton 如何支持 FP4/FP8 风格的块缩放格式。

- **EN:** Key kernels or functions: `block_scaled_matmul_kernel, block_scaled_matmul, cublas_block_scaled_matmul`.
- **CN:** 关键内核/函数：`block_scaled_matmul_kernel, block_scaled_matmul, cublas_block_scaled_matmul`。

- **EN:** Key Triton features: `tl.dot_scaled`, `TensorDescriptor`, scale reshaping, format-specific code paths.
- **CN:** 关键 Triton 特性：`tl.dot_scaled`, `TensorDescriptor`, scale reshaping, format-specific code paths。

- **EN:** The kernel still looks like a tile-based GEMM, but the data representation and scale layout become first-class concerns.
- **CN:** 内核整体上仍像一个 tile-based GEMM，但数据表示和 scale 布局成为了一等公民。

- **EN:** The main lesson is that new numeric formats usually force changes in layout, packing, and host-side preprocessing, not just one new math op.
- **CN:** 核心教训是：新数值格式通常会牵涉布局、打包和主机端预处理的改变，而不仅仅是多一条数学指令。

- **EN:** The tutorial explicitly reshapes and preshuffles scale tensors to match what `tl.dot_scaled` expects.
- **CN:** 该教程会显式 reshape 和 preshuffle scale 张量，以满足 `tl.dot_scaled` 的输入要求。

- **EN:** When low-precision kernels arrive, data layout and metadata handling often become half the optimization problem.
- **CN:** 一旦进入低精度内核世界，数据布局与元数据处理往往就占据了优化问题的一半。

### 6.4 Programmatic Dependent Launch / 程序化依赖启动

- **EN:** Primary file: `python/tutorials/11-programmatic-dependent-launch.py`.
- **CN:** 主要文件：`python/tutorials/11-programmatic-dependent-launch.py`。

- **EN:** This tutorial shows how to use CUDA GDC / programmatic dependent launch with a familiar vector-add kernel.
- **CN:** 该教程用一个熟悉的向量加法内核展示如何使用 CUDA GDC / 程序化依赖启动。

- **EN:** Key kernels or functions: `add_kernel, add, validate, benchmark`.
- **CN:** 关键内核/函数：`add_kernel, add, validate, benchmark`。

- **EN:** Key Triton features: `tl.extra.cuda.gdc_wait`, `tl.extra.cuda.gdc_launch_dependents`, `launch_pdl=True`.
- **CN:** 关键 Triton 特性：`tl.extra.cuda.gdc_wait`, `tl.extra.cuda.gdc_launch_dependents`, `launch_pdl=True`。

- **EN:** The grid is intentionally simple so the runtime launch semantics, not the math, stay at center stage.
- **CN:** grid 被刻意保持简单，以便把重点放在运行时启动语义而非数学计算上。

- **EN:** The benchmark compares PDL and non-PDL execution to show when dependent launch can reduce coordination overhead.
- **CN:** 教程通过比较 PDL 与非 PDL 执行，展示依赖启动在何时能降低协调开销。

- **EN:** The placement of `gdc_wait()` before loads and `gdc_launch_dependents()` after loads is the pedagogical core of the file.
- **CN:** 把 `gdc_wait()` 放在 load 之前、把 `gdc_launch_dependents()` 放在 load 之后，是这个文件最关键的教学点。

- **EN:** Advanced Triton is not only about kernel bodies; launch protocol is sometimes part of performance.
- **CN:** 高级 Triton 不仅关乎内核体本身；启动协议有时也是性能的一部分。

## 7. Gluon / Gluon框架

- **EN:** `python/tutorials/gluon/` is the hardware-near counterpart to the regular Triton tutorials.
- **CN:** `python/tutorials/gluon/` 是普通 Triton 教程的“贴近硬件”版本。

- **EN:** `01-intro.py` describes Gluon as a lower-level GPU programming language based on the same compiler stack as Triton.
- **CN:** `01-intro.py` 将 Gluon 描述为建立在同一编译栈之上的、更底层的 GPU 编程语言。

- **EN:** Both Triton and Gluon are tile-based Python DSLs, but Gluon exposes details that Triton intentionally abstracts away.
- **CN:** Triton 与 Gluon 都是基于 tile 的 Python DSL，但 Gluon 暴露了 Triton 刻意隐藏的很多底层细节。

- **EN:** In Gluon you control layouts, shared memory, tensor memory, async proxy ordering, mbarriers, warp specialization, and cluster structure much more explicitly.
- **CN:** 在 Gluon 中，你需要更显式地控制布局、共享内存、tensor memory、async proxy 顺序、mbarrier、warp specialization 和 cluster 结构。

- **EN:** That means Gluon is more powerful for architecture-specific peak tuning, but also demands more hardware knowledge and more correctness discipline.
- **CN:** 这意味着 Gluon 在架构特定峰值调优方面更强，但也要求更多硬件知识和更严格的正确性控制。

- **EN:** A good learning order is exactly the tutorial order in the repo: intro → layouts → async copy → TMA → WGMMA / tcgen05 → persistence → warp specialization → cluster features.
- **CN:** 一个好的学习顺序就是仓库中的原始顺序：intro → layouts → async copy → TMA → WGMMA / tcgen05 → persistence → warp specialization → cluster 特性。

- **EN:** If regular Triton answers “what tile should each program compute?”, Gluon also asks “where is that tile stored, how is it distributed across warps, and how are async producers/consumers synchronized?”.
- **CN:** 如果说常规 Triton 回答的是“每个 program 该算哪块 tile”，那么 Gluon 还会继续追问“tile 存在哪里、如何分布到各个 warp、异步生产者与消费者如何同步”。

- **EN:** The Gluon material is especially valuable on Hopper and Blackwell, where TMA, WGMMA, tcgen05, and cluster control significantly affect performance.
- **CN:** Gluon 材料在 Hopper 和 Blackwell 上尤其有价值，因为 TMA、WGMMA、tcgen05 和 cluster 控制会显著影响性能。

### 7.1 Intro / 入门

- **EN:** Primary file: `python/tutorials/gluon/01-intro.py`.
- **CN:** 主要文件：`python/tutorials/gluon/01-intro.py`。

- **EN:** Introduces `@gluon.jit`, scalar copy, memcpy, and autotuned memcpy as the Gluon starting point.
- **CN:** 以 `@gluon.jit`、标量拷贝、memcpy 和 autotune memcpy 作为 Gluon 的起点。

- **EN:** Hardware focus is light here; the goal is to transfer Triton launch intuition into the Gluon namespace.
- **CN:** 这里的硬件侧重点较轻，主要是把 Triton 的 launch 直觉迁移到 Gluon 命名空间。

- **EN:** Important APIs: `@gluon.jit`, `gl.load`, `gl.store`, `gl.program_id`, `@triton.autotune`.
- **CN:** 重要 API：`@gluon.jit`, `gl.load`, `gl.store`, `gl.program_id`, `@triton.autotune`。

- **EN:** Key kernels or helpers: `copy_scalar_kernel, memcpy_kernel, memcpy_kernel_autotune`.
- **CN:** 关键内核或辅助函数：`copy_scalar_kernel, memcpy_kernel, memcpy_kernel_autotune`。

- **EN:** The memory story is still simple global-memory load/store, but now under Gluon semantics.
- **CN:** 这里的内存模型仍是简单的全局内存 load/store，只是改由 Gluon 语义来表达。

- **EN:** Autotuning already appears, reinforcing that Gluon shares Triton’s runtime and benchmark ecosystem.
- **CN:** 这里已经出现 autotune，说明 Gluon 与 Triton 共享运行时和 benchmark 生态。

- **EN:** Use this file to see what stays familiar when moving from Triton to Gluon.
- **CN:** 如果你想知道从 Triton 切到 Gluon 后哪些东西仍然熟悉，请先看这个文件。

### 7.2 Layouts / 布局

- **EN:** Primary file: `python/tutorials/gluon/02-layouts.py`.
- **CN:** 主要文件：`python/tutorials/gluon/02-layouts.py`。

- **EN:** Teaches layouts as first-class objects describing how tensor elements are distributed across registers, lanes, warps, and CTAs.
- **CN:** 把 layout 作为一等对象来讲，解释张量元素如何在寄存器、lane、warp 和 CTA 之间分布。

- **EN:** This is the conceptual foundation for coalescing, bank behavior, and tensor-core operand organization.
- **CN:** 这是理解 coalescing、bank 行为和张量核操作数组织方式的概念基础。

- **EN:** Important APIs: `gl.BlockedLayout`, `gl.SliceLayout`, `gl.convert_layout`, layout-aware `gl.arange`.
- **CN:** 重要 API：`gl.BlockedLayout`, `gl.SliceLayout`, `gl.convert_layout`, layout-aware `gl.arange`。

- **EN:** Key kernels or helpers: `memcpy_1d_kernel, memcpy_2d_kernel, memcpy_2d_inout_kernel`.
- **CN:** 关键内核或辅助函数：`memcpy_1d_kernel, memcpy_2d_kernel, memcpy_2d_inout_kernel`。

- **EN:** Memory movement becomes layout-aware rather than being treated as an unstructured pointer operation.
- **CN:** 这里的数据搬运不再被看成无结构指针操作，而是明确受 layout 控制。

- **EN:** Performance depends on layout choice, so the tutorial makes “distribution” part of the programming model.
- **CN:** 性能直接依赖布局选择，因此这个教程把“分布方式”纳入了编程模型本身。

- **EN:** If Triton teaches tiles, this file teaches where the tile lives inside a thread block.
- **CN:** 如果说 Triton 教的是 tile，那么这个文件教的是 tile 在 thread block 内部如何落地。

### 7.3 Async Copy / 异步拷贝

- **EN:** Primary file: `python/tutorials/gluon/03-async-copy.py`.
- **CN:** 主要文件：`python/tutorials/gluon/03-async-copy.py`。

- **EN:** Introduces Ampere `cp.async` and shows how to overlap global-memory traffic with compute.
- **CN:** 介绍 Ampere `cp.async`，并展示如何让全局内存流量与计算重叠。

- **EN:** This is the entry point to software pipelining on modern NVIDIA GPUs.
- **CN:** 这是进入现代 NVIDIA GPU 软件流水线世界的入口。

- **EN:** Important APIs: `cp.async_load`, `cp.commit_group`, `cp.wait_group`, shared-memory layouts.
- **CN:** 重要 API：`cp.async_load`, `cp.commit_group`, `cp.wait_group`, shared-memory layouts。

- **EN:** Key kernels or helpers: `memcpy_1d_cpasync_kernel, elementwise_add_cpasync_kernel, elementwise_add_pipelined_kernel`.
- **CN:** 关键内核或辅助函数：`memcpy_1d_cpasync_kernel, elementwise_add_cpasync_kernel, elementwise_add_pipelined_kernel`。

- **EN:** The file makes shared memory an explicit staging area between global memory and registers.
- **CN:** 该文件把共享内存显式变成全局内存与寄存器之间的中转站。

- **EN:** The main win comes from pipelining and reduced exposed latency rather than changing arithmetic.
- **CN:** 其主要收益来自流水化与隐藏延迟，而不是改变算术本身。

- **EN:** Read this file when regular `gl.load`/`gl.store` no longer explain performance behavior.
- **CN:** 当普通 `gl.load`/`gl.store` 已无法解释性能现象时，就该读这个文件。

### 7.4 TMA / TMA

- **EN:** Primary file: `python/tutorials/gluon/04-tma.py`.
- **CN:** 主要文件：`python/tutorials/gluon/04-tma.py`。

- **EN:** Introduces tensor descriptors, TMA loads/stores, and mbarrier synchronization on Hopper and newer.
- **CN:** 介绍张量描述符、TMA load/store 以及 Hopper 及之后架构上的 mbarrier 同步。

- **EN:** TMA is crucial because it moves complex addressing out of registers and onto dedicated hardware.
- **CN:** TMA 很关键，因为它把复杂寻址从寄存器中移走，交给专用硬件完成。

- **EN:** Important APIs: `TensorDescriptor`, `tma.async_load`, `tma.async_copy_shared_to_global`, `mbarrier`, `fence_async_shared`.
- **CN:** 重要 API：`TensorDescriptor`, `tma.async_load`, `tma.async_copy_shared_to_global`, `mbarrier`, `fence_async_shared`。

- **EN:** Key kernels or helpers: `memcpy_1d_tma_kernel, elementwise_add_tma_kernel`.
- **CN:** 关键内核或辅助函数：`memcpy_1d_tma_kernel, elementwise_add_tma_kernel`。

- **EN:** The tutorial revolves around descriptor-based global-memory access and async proxy ordering.
- **CN:** 这个教程围绕 descriptor 驱动的全局内存访问与 async proxy 顺序展开。

- **EN:** Compared with raw global loads, TMA can reduce register pressure and enable larger, cleaner pipelines.
- **CN:** 与普通全局 load 相比，TMA 可以降低寄存器压力，并构造更大、更干净的流水线。

- **EN:** TMA is one of the main reasons Gluon can express Hopper-class kernels naturally.
- **CN:** TMA 是 Gluon 能自然表达 Hopper 级内核的关键原因之一。

### 7.5 WGMMA / Warp-Group MMA

- **EN:** Primary file: `python/tutorials/gluon/05-wgmma.py`.
- **CN:** 主要文件：`python/tutorials/gluon/05-wgmma.py`。

- **EN:** Shows Hopper warp-group MMA and how to build a matmul around asynchronous tensor-core instructions.
- **CN:** 展示 Hopper 的 warp-group MMA，以及如何围绕异步张量核指令构建 matmul。

- **EN:** This is the first tutorial where tensor-core execution shape becomes central instead of incidental.
- **CN:** 这是第一个把张量核执行形状放到核心位置的教程。

- **EN:** Important APIs: `warpgroup_mma_init`, `warpgroup_mma`, `warpgroup_mma_wait`, `NVMMADistributedLayout`.
- **CN:** 重要 API：`warpgroup_mma_init`, `warpgroup_mma`, `warpgroup_mma_wait`, `NVMMADistributedLayout`。

- **EN:** Key kernels or helpers: `small_mma_kernel, blocked_matmul_kernel`.
- **CN:** 关键内核或辅助函数：`small_mma_kernel, blocked_matmul_kernel`。

- **EN:** Operands are carefully placed into layouts compatible with WGMMA expectations.
- **CN:** 操作数会被放入与 WGMMA 要求兼容的布局之中。

- **EN:** The important performance lesson is that async MMA lets compute itself participate in a pipeline.
- **CN:** 关键性能启示是：异步 MMA 让计算本身也成为流水线的一部分。

- **EN:** This file is essential if you want to understand Hopper tensor-core kernels from the inside.
- **CN:** 如果你想从内部理解 Hopper 张量核内核，这个文件必读。

### 7.6 TCGen05 / 第五代 TensorCore

- **EN:** Primary file: `python/tutorials/gluon/06-tcgen05.py`.
- **CN:** 主要文件：`python/tutorials/gluon/06-tcgen05.py`。

- **EN:** Introduces Blackwell tensor memory and `tcgen05` MMA programming.
- **CN:** 介绍 Blackwell 的 tensor memory 以及 `tcgen05` MMA 编程方式。

- **EN:** Blackwell changes the programming model enough that tensor memory becomes a required concept, not an optimization detail.
- **CN:** Blackwell 对编程模型的改变足够大，以至于 tensor memory 成为必学概念，而不是附加优化细节。

- **EN:** Important APIs: `TensorMemoryLayout`, `allocate_tensor_memory`, `tcgen05_mma`, `tcgen05_commit`.
- **CN:** 重要 API：`TensorMemoryLayout`, `allocate_tensor_memory`, `tcgen05_mma`, `tcgen05_commit`。

- **EN:** Key kernels or helpers: `tmem_example_kernel, small_mma_kernel, blocked_matmul_kernel`.
- **CN:** 关键内核或辅助函数：`tmem_example_kernel, small_mma_kernel, blocked_matmul_kernel`。

- **EN:** The file teaches that accumulators may live in tensor memory rather than only in registers or shared memory.
- **CN:** 该文件告诉你：累加器可能生活在 tensor memory 中，而不是只在寄存器或共享内存里。

- **EN:** Programming tensor cores at this level is partly a memory-space problem and only partly a math problem.
- **CN:** 在这个层次上编程张量核，一部分是数学问题，另一大部分其实是内存空间问题。

- **EN:** Read this file to understand why Blackwell programming cannot be explained only with Hopper concepts.
- **CN:** 想理解为什么 Blackwell 不能只用 Hopper 的概念来解释，就读这个文件。

### 7.7 Persistence / 持久化

- **EN:** Primary file: `python/tutorials/gluon/07-persistence.py`.
- **CN:** 主要文件：`python/tutorials/gluon/07-persistence.py`。

- **EN:** Builds persistent matmul kernels and abstracts MMA differences across Hopper and Blackwell.
- **CN:** 构建持久化 matmul 内核，并抽象 Hopper 与 Blackwell 的 MMA 差异。

- **EN:** Persistence here is coupled with deeper pipeline structure and reusable MMA abstractions.
- **CN:** 这里的持久化与更深的流水线结构、可复用 MMA 抽象紧密结合。

- **EN:** Important APIs: `@gluon.aggregate`, persistent outer loops, reusable `WGMMA` / `MMAv5` abstractions.
- **CN:** 重要 API：`@gluon.aggregate`, persistent outer loops, reusable `WGMMA` / `MMAv5` abstractions。

- **EN:** Key kernels or helpers: `matmul_pipelined_kernel, PersistentTileScheduler`.
- **CN:** 关键内核或辅助函数：`matmul_pipelined_kernel, PersistentTileScheduler`。

- **EN:** Persistent kernels turn memory staging and tile reuse into first-class design decisions.
- **CN:** 持久化内核把内存 staging 和 tile 复用提升为一等设计决策。

- **EN:** The file shows how to overlap work not just within a tile pipeline but across the outer schedule itself.
- **CN:** 该文件展示的不只是 tile 内部流水线重叠，还包括跨外层调度的重叠。

- **EN:** Persistence is the natural continuation of TMA and MMA once you start thinking in SM residency terms.
- **CN:** 当你开始以 SM 驻留来思考问题时，持久化就是 TMA 和 MMA 的自然延伸。

### 7.8 Warp Specialization / Warp 专门化

- **EN:** Primary file: `python/tutorials/gluon/08-warp-specialization.py`.
- **CN:** 主要文件：`python/tutorials/gluon/08-warp-specialization.py`。

- **EN:** Splits a kernel into load, compute, and store warp partitions.
- **CN:** 把一个内核拆成 load、compute 和 store 三类 warp 分工。

- **EN:** Warp specialization is a scheduling technique that uses the warp scheduler as a coarse-grained overlap mechanism.
- **CN:** warp specialization 是一种调度技术，它把 warp scheduler 当作粗粒度重叠机制来使用。

- **EN:** Important APIs: `gl.warp_specialize`, partition functions, barriers, async/shared fences.
- **CN:** 重要 API：`gl.warp_specialize`, partition functions, barriers, async/shared fences。

- **EN:** Key kernels or helpers: `load_partition, compute_partition, store_partition, elementwise_add_warp_specialized_kernel`.
- **CN:** 关键内核或辅助函数：`load_partition, compute_partition, store_partition, elementwise_add_warp_specialized_kernel`。

- **EN:** Shared memory becomes a communication medium between specialized warps.
- **CN:** 共享内存成为专门化 warp 之间的数据交换媒介。

- **EN:** The benefit is overlap and critical-path reduction, but the cost is extra synchronization and register pressure.
- **CN:** 收益是重叠执行与缩短关键路径，代价则是更多同步和更高寄存器压力。

- **EN:** This file is one of the clearest demonstrations that scheduling and kernel structure are inseparable on modern GPUs.
- **CN:** 这是最清楚地展示“调度与内核结构无法分离”的文件之一。

### 7.9 TMA Gather/Scatter / TMA Gather/Scatter

- **EN:** Primary file: `python/tutorials/gluon/09-tma-gather-scatter.py`.
- **CN:** 主要文件：`python/tutorials/gluon/09-tma-gather-scatter.py`。

- **EN:** Explains Blackwell-native async gather and scatter built on TMA descriptors.
- **CN:** 解释 Blackwell 原生基于 TMA descriptor 的异步 gather 与 scatter。

- **EN:** The hardware feature matters because irregular access can now still use structured transfer machinery.
- **CN:** 这个硬件特性的重要性在于：即使访问模式不规则，也能利用结构化传输机制。

- **EN:** Important APIs: `tma.async_gather`, `tma.async_scatter`, row-offset layouts, `MBarrierLayout`.
- **CN:** 重要 API：`tma.async_gather`, `tma.async_scatter`, row-offset layouts, `MBarrierLayout`。

- **EN:** Key kernels or helpers: `async_gather_kernel`.
- **CN:** 关键内核或辅助函数：`async_gather_kernel`。

- **EN:** Row offsets themselves become layout-constrained data that must be loaded and synchronized carefully.
- **CN:** 连行偏移本身都变成需要精心布局和同步的数据。

- **EN:** The file’s real lesson is that “irregular” does not mean “unstructured”; the structure just moves into descriptors and offset tensors.
- **CN:** 这个文件真正的启示是：“不规则”不等于“无结构”；结构只是被移动到了 descriptor 和 offset 张量里。

- **EN:** Read it when sparse or gather/scatter style kernels start to dominate your workload.
- **CN:** 当稀疏、gather/scatter 型内核开始主导你的工作负载时，就该回来看这个文件。

### 7.10 TCGen05 Copy / TCGen05 拷贝

- **EN:** Primary file: `python/tutorials/gluon/10-tcgen05-copy.py`.
- **CN:** 主要文件：`python/tutorials/gluon/10-tcgen05-copy.py`。

- **EN:** Shows how to move data from shared memory into tensor memory with `tcgen05_copy`.
- **CN:** 展示如何用 `tcgen05_copy` 把数据从共享内存搬到 tensor memory。

- **EN:** This is important because Blackwell tensor-core pipelines involve more than just MMA instructions.
- **CN:** 这很重要，因为 Blackwell 的张量核流水线不只是 MMA 指令本身。

- **EN:** Important APIs: `tcgen05_copy`, `tcgen05_commit`, tensor-memory allocation, shared-memory fencing.
- **CN:** 重要 API：`tcgen05_copy`, `tcgen05_commit`, tensor-memory allocation, shared-memory fencing。

- **EN:** Key kernels or helpers: `tcgen05_copy_kernel`.
- **CN:** 关键内核或辅助函数：`tcgen05_copy_kernel`。

- **EN:** The tutorial is fundamentally about explicit movement across hardware memory spaces.
- **CN:** 这个教程本质上讲的是跨硬件内存空间的显式数据搬运。

- **EN:** Understanding copy/commit/wait semantics is necessary before you can trust higher-level tensor-core pipelines.
- **CN:** 在真正信任更高层张量核流水线之前，必须先理解 copy/commit/wait 语义。

- **EN:** This file makes Blackwell dataflow feel concrete rather than mystical.
- **CN:** 这个文件让 Blackwell 数据流从“神秘”变得“具体”。

### 7.11 TCGen05 MMA Scaled / 缩放版 TCGen05 MMA

- **EN:** Primary file: `python/tutorials/gluon/11-tcgen05-mma-scaled.py`.
- **CN:** 主要文件：`python/tutorials/gluon/11-tcgen05-mma-scaled.py`。

- **EN:** Demonstrates block-scaled tensor-core instructions that fuse dequantization and MMA.
- **CN:** 展示把反量化与 MMA 融合到一起的 block-scaled 张量核指令。

- **EN:** This is where quantization format, tensor memory, and tensor-core ISA all meet in one place.
- **CN:** 这是量化格式、tensor memory 与 tensor-core ISA 三者真正汇合的地方。

- **EN:** Important APIs: `tcgen05_mma_scaled`, `TensorMemoryScalesLayout`, `tcgen05_copy`, scaled MMA pipelines.
- **CN:** 重要 API：`tcgen05_mma_scaled`, `TensorMemoryScalesLayout`, `tcgen05_copy`, scaled MMA pipelines。

- **EN:** Key kernels or helpers: `simple_mma_scaled_kernel and several scaled MMA variants`.
- **CN:** 关键内核或辅助函数：`simple_mma_scaled_kernel and several scaled MMA variants`。

- **EN:** Scale tensors have their own layouts and movement constraints, not just value tensors.
- **CN:** scale 张量拥有自己独立的布局与搬运约束，而不只是值张量的附属品。

- **EN:** The tutorial teaches how low-precision peak performance depends on marrying math format and memory format.
- **CN:** 该教程说明：低精度峰值性能取决于数值格式与内存格式的耦合。

- **EN:** Use this file as a reference when you need to reason about quantized tensor-core kernels end to end.
- **CN:** 当你需要端到端思考量化张量核内核时，把这个文件当参考。

### 7.12 Cluster Launch Control / Cluster Launch Control

- **EN:** Primary file: `python/tutorials/gluon/12-cluster-launch-control.py`.
- **CN:** 主要文件：`python/tutorials/gluon/12-cluster-launch-control.py`。

- **EN:** Introduces Blackwell dynamic cluster cancellation and work stealing for better load balance.
- **CN:** 介绍 Blackwell 的动态 cluster 取消与工作窃取机制，以改善负载均衡。

- **EN:** CLC turns scheduling itself into a hardware-assisted operation.
- **CN:** CLC 让调度本身变成一种硬件辅助操作。

- **EN:** Important APIs: `clc.try_cancel`, `clc.load_result`, `is_canceled`, persistent matmul scheduling.
- **CN:** 重要 API：`clc.try_cancel`, `clc.load_result`, `is_canceled`, persistent matmul scheduling。

- **EN:** Key kernels or helpers: `ClcTileScheduler, StaticTileScheduler, persistent_matmul_kernel`.
- **CN:** 关键内核或辅助函数：`ClcTileScheduler, StaticTileScheduler, persistent_matmul_kernel`。

- **EN:** The clever part is overlapping CLC with the TMA prologue so scheduling latency is hidden behind memory work.
- **CN:** 最巧妙的地方在于把 CLC 与 TMA prologue 重叠，让调度延迟隐藏在内存准备阶段之后。

- **EN:** This tutorial is about dynamic load balance, not just single-tile speed.
- **CN:** 这个教程关注的是动态负载均衡，而不仅仅是单个 tile 的速度。

- **EN:** When work is irregular, better scheduling can matter as much as faster inner loops.
- **CN:** 当工作负载不规则时，更好的调度往往和更快的内层循环同样重要。

### 7.13 Convolution via im2col / 通过 im2col 做卷积

- **EN:** Primary file: `python/tutorials/gluon/13-conv-im2col.py`.
- **CN:** 主要文件：`python/tutorials/gluon/13-conv-im2col.py`。

- **EN:** Explains TMA im2col mode and uses it to build convolution as implicit GEMM.
- **CN:** 解释 TMA 的 im2col 模式，并据此把卷积写成隐式 GEMM。

- **EN:** This file is a strong example of using hardware-assisted address generation to express a classic algorithmic transform.
- **CN:** 这是一个很好的例子：利用硬件辅助寻址来表达经典算法变换。

- **EN:** Important APIs: `TensorDescriptorIm2Col`, `tma.async_load_im2col`, MMA abstraction reuse.
- **CN:** 重要 API：`TensorDescriptorIm2Col`, `tma.async_load_im2col`, MMA abstraction reuse。

- **EN:** Key kernels or helpers: `tma_im2col_kernel and convolution helpers`.
- **CN:** 关键内核或辅助函数：`tma_im2col_kernel and convolution helpers`。

- **EN:** The tensor descriptor carries convolution window semantics, so address generation moves out of ad hoc pointer arithmetic.
- **CN:** 张量描述符携带卷积窗口语义，因此地址生成不再依赖临时拼凑的指针运算。

- **EN:** The big lesson is that data-layout transforms can be fused into the transfer path rather than materialized as separate tensors.
- **CN:** 最大的启示是：数据布局变换可以融合进传输路径，而不必单独物化成中间张量。

- **EN:** Use this tutorial when you want to see Gluon handle more than GEMM-shaped problems.
- **CN:** 如果你想看 Gluon 如何处理不止 GEMM 形状的问题，请读这个教程。

### 7.14 Multi-CTA / 多 CTA 协作

- **EN:** Primary file: `python/tutorials/gluon/14-multicta.py`.
- **CN:** 主要文件：`python/tutorials/gluon/14-multicta.py`。

- **EN:** Explains CGA / multi-CTA cooperation, distributed shared memory, and cross-CTA layouts.
- **CN:** 解释 CGA / 多 CTA 协作、分布式共享内存以及跨 CTA 布局。

- **EN:** This is the broadest Gluon tutorial because it expands the scope from one CTA to a whole CTA cluster.
- **CN:** 这是范围最广的 Gluon 教程，因为它把问题规模从单个 CTA 扩展到整个 CTA cluster。

- **EN:** Important APIs: cluster-aware `BlockedLayout`, cluster reductions, `mbarrier`, CTA sharding.
- **CN:** 重要 API：cluster-aware `BlockedLayout`, cluster reductions, `mbarrier`, CTA sharding。

- **EN:** Key kernels or helpers: `multicta_softmax_kernel and later multi-CTA examples`.
- **CN:** 关键内核或辅助函数：`multicta_softmax_kernel and later multi-CTA examples`。

- **EN:** Shared memory is no longer purely local; cluster topology becomes part of correctness and cost.
- **CN:** 共享内存不再是纯本地概念；cluster 拓扑本身就成为正确性和成本模型的一部分。

- **EN:** The tutorial repeatedly warns that collaboration across CTAs is powerful but much more expensive than intra-CTA cooperation.
- **CN:** 教程反复强调：跨 CTA 协作虽然强大，但代价远高于 CTA 内协作。

- **EN:** Read this file when single-CTA mental models stop being enough for the hardware you target.
- **CN:** 当单 CTA 心智模型已经不足以解释目标硬件时，就需要读这个文件。

## 8. Compiler Internals / 编译器内部

- **EN:** The compiler front door is `python/triton/compiler/compiler.py`, and the most important function to read is `compile(src, target=None, options=None, _env_vars=None)`.
- **CN:** 编译器入口在 `python/triton/compiler/compiler.py`，其中最重要的函数是 `compile(src, target=None, options=None, _env_vars=None)`。

- **EN:** Compilation begins by selecting a `GPUTarget` and building the backend object with `make_backend(target)`.
- **CN:** 编译一开始会先选择 `GPUTarget`，然后通过 `make_backend(target)` 构建后端对象。

- **EN:** `ASTSource` wraps Python JIT functions, while `IRSource` wraps existing `.ttir`, `.ttgir`, `.llir`, `.ptx`, or `.amdgcn` inputs.
- **CN:** `ASTSource` 用于封装 Python JIT 函数，而 `IRSource` 则封装已有的 `.ttir`、`.ttgir`、`.llir`、`.ptx`、`.amdgcn` 输入。

- **EN:** `compiler.py` computes a cache key from the source hash, backend hash, backend option hash, and cache-invalidating environment variables.
- **CN:** `compiler.py` 会基于源代码哈希、后端哈希、后端选项哈希，以及会影响缓存的环境变量生成缓存键。

- **EN:** This explains why changing backend options or debugging hooks can invalidate previously compiled kernels.
- **CN:** 这解释了为什么修改后端选项或调试 hook 会让已有编译缓存失效。

- **EN:** The compiler stores stage artifacts through cache managers and can also dump or override artifacts depending on knobs and environment variables.
- **CN:** 编译器通过 cache manager 存储各阶段产物，并可根据 knobs/环境变量进行 dump 或 override。

- **EN:** If the kernel is already cached, `CompiledKernel` is reconstructed from metadata without re-running lowering.
- **CN:** 如果内核已命中缓存，就会直接从元数据恢复 `CompiledKernel`，无需再次 lowering。

- **EN:** If not cached, the backend populates the stage pipeline and `compiler.py` runs each stage in insertion order.
- **CN:** 如果没有命中缓存，后端会填充 stage pipeline，然后 `compiler.py` 按插入顺序依次运行。

- **EN:** The repository layout shows the major dialect families: `Triton`, `TritonGPU`, `TritonNvidiaGPU`, `Gluon`, and `TritonInstrument`.
- **CN:** 仓库目录展示了主要方言家族：`Triton`、`TritonGPU`、`TritonNvidiaGPU`、`Gluon` 与 `TritonInstrument`。

- **EN:** The conversion pipeline is correspondingly split into `TritonToTritonGPU`, `TritonGPUToLLVM`, and instrumentation-to-LLVM paths.
- **CN:** 相应地，转换管线也拆分为 `TritonToTritonGPU`、`TritonGPUToLLVM` 以及 instrumentation 到 LLVM 的路径。

- **EN:** `python/triton/backends/compiler.py` defines the backend contract: `supports_target`, `hash`, `parse_options`, `add_stages`, `load_dialects`, and `get_module_map`.
- **CN:** `python/triton/backends/compiler.py` 定义了后端契约：`supports_target`、`hash`、`parse_options`、`add_stages`、`load_dialects` 与 `get_module_map`。

- **EN:** This is why adding a new backend is not “just emit PTX”; it also means defining dialect loading, options, stage order, and runtime/driver integration.
- **CN:** 因此，新增后端绝不是“只要会吐 PTX”这么简单；它还要求定义方言加载、选项处理、阶段顺序以及运行时/驱动集成。

- **EN:** `compiler.py` loads base dialects through `ir.load_dialects(context)` and then lets the backend load additional dialects.
- **CN:** `compiler.py` 先通过 `ir.load_dialects(context)` 加载基础方言，再交给后端加载额外方言。

- **EN:** This layered dialect loading is one reason the same infrastructure can serve both standard Triton and Gluon.
- **CN:** 这种分层方言加载机制，也是同一基础设施能同时服务普通 Triton 和 Gluon 的原因之一。

- **EN:** Warp specialization appears in multiple places in the repo: as a user-facing concept in `python/tutorials/gluon/08-warp-specialization.py`, as NVIDIA/GPU-specific lowering support in dialect/transform directories, and as test coverage under Hopper-focused tests.
- **CN:** warp specialization 在仓库中以多种形式出现：既是 `python/tutorials/gluon/08-warp-specialization.py` 中的用户特性，也是方言/变换目录中的 NVIDIA/GPU lowering 支持，并有 Hopper 相关测试覆盖。

- **EN:** Pipelining also exists at multiple levels: software pipelining within kernels, stage pipelining across compiler phases, and async compile paths in the runtime.
- **CN:** pipelining 也有多层含义：既包括内核内的软件流水线，也包括编译阶段间的 stage pipeline，以及运行时层面的异步编译路径。

- **EN:** `CompileTimer` in `compiler.py` records IR initialization, per-stage lowering durations, and result storage time for listeners.
- **CN:** `compiler.py` 中的 `CompileTimer` 会记录 IR 初始化时间、逐 stage lowering 时间以及结果写回时间，供 listener 使用。

- **EN:** For developers, the key practical lesson is that Triton’s compiler is not hidden magic; it is readable, hookable, cacheable Python orchestration around MLIR/C++ lowering.
- **CN:** 对开发者来说，最重要的实践结论是：Triton 编译器并不是隐藏魔法，而是围绕 MLIR/C++ lowering 构建的、可读、可挂钩、可缓存的 Python 编排层。

- **EN:** When debugging compiler problems, always ask: what is the current stage, what dialect am I in, what backend options are active, and what artifact did the cache save?
- **CN:** 调试编译器问题时，应始终先问：当前处于哪个 stage、现在是哪种方言、激活了哪些后端选项、缓存里保存了什么产物？

- **EN:** This stage-oriented way of reading Triton source code makes the repository much easier to navigate.
- **CN:** 用“stage 导向”的方式去阅读 Triton 源码，会让整个仓库更容易理解。

```text
Key compiler landmarks:
  python/triton/compiler/compiler.py        # orchestration, cache, stage loop
  python/triton/backends/compiler.py        # backend abstract interface
  lib/Dialect/Triton/                       # base Triton dialect
  lib/Dialect/TritonGPU/                    # generic GPU dialect
  lib/Dialect/TritonNvidiaGPU/              # NVIDIA-specific dialect
  lib/Dialect/Gluon/                        # Gluon dialect
  lib/Dialect/TritonInstrument/             # instrumentation dialect
  lib/Conversion/TritonToTritonGPU/         # TTIR -> TTGIR lowering
  lib/Conversion/TritonGPUToLLVM/           # TTGIR -> LLVM lowering
  lib/Conversion/TritonInstrumentToLLVM/    # sanitizer/instrumentation lowering
```

## 9. Runtime / 运行时

- **EN:** `python/triton/runtime/jit.py` is the best file for understanding what `@triton.jit` actually means at runtime.
- **CN:** `python/triton/runtime/jit.py` 是理解 `@triton.jit` 在运行时到底意味着什么的最佳文件。

- **EN:** A JIT function is launched with `fn[grid](*args, **kwargs)`, implemented through `KernelInterface.__getitem__`.
- **CN:** JIT 函数通过 `fn[grid](*args, **kwargs)` 启动，这一语法由 `KernelInterface.__getitem__` 实现。

- **EN:** `JITFunction.run()` obtains the active device and stream from `driver.active`, binds arguments, computes specialization, checks caches, compiles if needed, and finally launches.
- **CN:** `JITFunction.run()` 会从 `driver.active` 获取当前 device 和 stream，绑定参数、计算 specialization、检查缓存、必要时编译，最后发起启动。

- **EN:** Specialization is not just dtype-based; it can include constexpr values, alignment-related attributes, and even custom pipeline hashes.
- **CN:** specialization 不只和 dtype 有关；它还会包含 constexpr 值、对齐相关属性，甚至自定义 pipeline 哈希。

- **EN:** `DependenciesFinder` hashes transitive dependencies and records global values used by the kernel, so changing a referenced global can invalidate correctness assumptions.
- **CN:** `DependenciesFinder` 会对传递依赖做哈希，并记录内核使用的全局值，因此修改被引用的全局变量会影响正确性假设。

- **EN:** This is an underappreciated feature: Triton tries to guard you against silently running a kernel with stale compile-time assumptions.
- **CN:** 这是一个容易被低估的特性：Triton 会尽量防止你在编译期假设已经陈旧的情况下悄悄运行内核。

- **EN:** `python/triton/runtime/autotuner.py` wraps kernels in an `Autotuner` that benchmarks configurations, prunes them, caches results, and remembers the best config per tuning key.
- **CN:** `python/triton/runtime/autotuner.py` 用 `Autotuner` 包装内核，对配置做 benchmark、剪枝、缓存，并按 tuning key 记住最佳配置。

- **EN:** The `Config` object stores meta-parameters such as `num_warps`, `num_stages`, `num_ctas`, `maxnreg`, and optional `ir_override`.
- **CN:** `Config` 对象保存 `num_warps`、`num_stages`、`num_ctas`、`maxnreg` 以及可选的 `ir_override` 等元参数。

- **EN:** Autotune results can also be cached to disk, not only in process memory.
- **CN:** autotune 结果不仅能存在进程内存，也能落盘缓存。

- **EN:** `python/triton/runtime/cache.py` defines file-based and remote cache managers, including a Redis-backed remote cache backend.
- **CN:** `python/triton/runtime/cache.py` 定义了基于文件和远程的缓存管理器，其中还包括基于 Redis 的远程缓存后端。

- **EN:** The file cache stores groups of related artifacts such as TTIR, TTGIR, LLVM IR, PTX, binary, and metadata JSON.
- **CN:** 文件缓存会成组保存相关产物，例如 TTIR、TTGIR、LLVM IR、PTX、二进制以及 metadata JSON。

- **EN:** `python/triton/runtime/driver.py` abstracts backend driver selection and supports `TRITON_DEFAULT_BACKEND` for explicit backend choice.
- **CN:** `python/triton/runtime/driver.py` 抽象了后端驱动选择，并支持通过 `TRITON_DEFAULT_BACKEND` 显式指定后端。

- **EN:** This driver abstraction is why the same Python-facing API can launch CUDA or HIP backends with the same top-level JIT syntax.
- **CN:** 正是这种 driver 抽象，使得同一套 Python API 可以用相同顶层 JIT 语法启动 CUDA 或 HIP 后端。

- **EN:** Warmup compilation is first-class: `KernelInterface.warmup()` exists precisely because kernel compilation and kernel launch are separate concerns.
- **CN:** warmup 编译是一等功能：`KernelInterface.warmup()` 的存在，正是因为“内核编译”和“内核启动”是两个不同问题。

- **EN:** Async compilation is also supported through runtime async-compile helpers, meaning compile latency can itself be pipelined in some workflows.
- **CN:** 运行时还支持异步编译，因此在某些工作流中，连编译延迟本身都可以被流水化。

- **EN:** The runtime is therefore doing three jobs simultaneously: specialization, compilation/cache management, and actual launch orchestration.
- **CN:** 因此，运行时同时承担三项工作：specialization、编译/缓存管理，以及实际 launch 编排。

- **EN:** For users, the important idea is simple: Triton kernels behave like Python callables, but under the hood they are specialized, cached, backend-bound launch objects.
- **CN:** 对用户而言，关键理解很简单：Triton 内核看起来像 Python 可调用对象，但底层其实是带 specialization、缓存与后端绑定的 launch 对象。

- **EN:** For contributors, `jit.py`, `autotuner.py`, `cache.py`, and `driver.py` together are the runtime control plane of the project.
- **CN:** 对贡献者来说，`jit.py`、`autotuner.py`、`cache.py` 与 `driver.py` 共同构成了项目的运行时控制平面。

```python
# Runtime view of a Triton launch
kernel = my_kernel[grid]   # bind grid
kernel(x, y, out, BLOCK_SIZE=1024)

# Under the hood:
# 1) bind args and constexprs
# 2) compute specialization key
# 3) look up compiled kernel in cache
# 4) compile if missing
# 5) launch on active device/stream
```

## 10. Kernel Library / 内核库

- **EN:** The repository contains a separate Python package under `python/triton_kernels/`, which acts as a reusable kernel library and benchmarking/test surface.
- **CN:** 仓库在 `python/triton_kernels/` 下包含一个独立 Python 包，可视为可复用内核库与 benchmark/test 平台。

- **EN:** Its `pyproject.toml` declares the package name `triton_kernels` and keeps dependencies lightweight.
- **CN:** 其 `pyproject.toml` 声明包名为 `triton_kernels`，并保持较轻的依赖集合。

- **EN:** This package is not the same as the core Triton compiler/runtime package; it is where higher-level kernels and utilities live.
- **CN:** 这个包并不等同于 Triton 核心编译器/运行时包；它更多是高层内核和工具函数的归宿。

- **EN:** `triton_kernels/matmul.py` is a substantial module with `PrecisionConfig`, `FusedActivation`, `Epilogue`, allocation helpers, and the main `matmul` entrypoint.
- **CN:** `triton_kernels/matmul.py` 是一个大型模块，其中包含 `PrecisionConfig`、`FusedActivation`、`Epilogue`、分配辅助函数以及主入口 `matmul`。

- **EN:** The same file exposes persistence-related controls such as `matmul_set_idle_sms`, showing how library kernels package scheduling policy.
- **CN:** 同一文件还暴露了 `matmul_set_idle_sms` 这类与持久化相关的控制接口，说明库内核会把调度策略打包成 API。

- **EN:** `triton_kernels/reduce.py` defines reduction-specific option flags, launch metadata, masked-row preprocessing, and a family of reduction kernels.
- **CN:** `triton_kernels/reduce.py` 定义了归约专用 option flag、launch metadata、masked-row 预处理以及一系列归约内核。

- **EN:** `triton_kernels/topk.py` provides forward and backward top-k, plus a `TopK` autograd wrapper returning a `SparseMatrix` abstraction.
- **CN:** `triton_kernels/topk.py` 提供了前向/反向 top-k，以及返回 `SparseMatrix` 抽象的 `TopK` autograd 封装。

- **EN:** That file is also a good example of combining Triton kernels with custom tensor abstractions and optional distributed all-gather support.
- **CN:** 这个文件也是 Triton 内核与自定义张量抽象、可选分布式 all-gather 支持结合的好例子。

- **EN:** `triton_kernels/numerics.py` defines helper dataclasses such as `InFlexData` and `OutFlexData`, alongside constants for low-precision numeric formats.
- **CN:** `triton_kernels/numerics.py` 定义了 `InFlexData`、`OutFlexData` 等辅助数据类，以及多种低精度格式相关常量。

- **EN:** `triton_kernels/tensor.py` defines `Storage`, `Tensor`, `RaggedTensor`, `SparseMatrix`, `wrap_torch_tensor`, `convert_layout`, and TMA-related helpers.
- **CN:** `triton_kernels/tensor.py` 定义了 `Storage`、`Tensor`、`RaggedTensor`、`SparseMatrix`、`wrap_torch_tensor`、`convert_layout` 以及与 TMA 相关的辅助函数。

- **EN:** That means the kernel library is not only a bag of kernels; it also experiments with richer tensor metadata and layout-aware container types.
- **CN:** 这说明内核库不只是“内核集合”，它也在探索更丰富的张量元数据和 layout-aware 容器类型。

- **EN:** `triton_kernels/distributed.py` defines expert assignment helpers and kernels such as `_convert_dp_to_ep` / `_convert_ep_to_dp` for distributed MoE-style data movement.
- **CN:** `triton_kernels/distributed.py` 定义了 expert assignment 辅助结构以及 `_convert_dp_to_ep` / `_convert_ep_to_dp` 等用于分布式 MoE 风格数据搬运的内核。

- **EN:** The subpackages `matmul_details`, `topk_details`, `numerics_details`, `tensor_details`, `distributed_details`, and `compaction_details` hold lower-level implementations.
- **CN:** `matmul_details`、`topk_details`、`numerics_details`、`tensor_details`、`distributed_details` 与 `compaction_details` 等子包保存了更底层的实现。

- **EN:** Other visible library modules include `compaction.py`, `swiglu.py`, `roofline.py`, `specialize.py`, and `testing.py`.
- **CN:** 库中还能看到 `compaction.py`、`swiglu.py`、`roofline.py`、`specialize.py` 和 `testing.py` 等模块。

- **EN:** The `bench/` and `tests/` directories show that `triton_kernels` is meant to be exercised like a real library, not just sample code.
- **CN:** `bench/` 和 `tests/` 目录说明 `triton_kernels` 被当作真正的库来维护，而不只是示例代码。

- **EN:** Conceptually, regular Triton tutorials teach you how to build kernels, while `triton_kernels` shows how to package them into reusable software components.
- **CN:** 从概念上看，常规 Triton 教程教你“如何写内核”，而 `triton_kernels` 则展示了“如何把内核封装成可复用软件组件”。

- **EN:** If you plan to build a project on top of Triton, `triton_kernels` is one of the best places in the repo to study API design and packaging style.
- **CN:** 如果你打算在 Triton 之上构建项目，`triton_kernels` 是研究 API 设计与封装风格的最佳位置之一。

- **EN:** It is also where Triton’s scope expands from individual kernels to subsystems such as distributed routing, specialized numerics, and reusable tensor abstractions.
- **CN:** 在这里，Triton 的作用域也从单个内核扩展到了分布式路由、专用数值格式和可复用张量抽象等更大子系统。

```text
python/triton_kernels/triton_kernels/
  matmul.py
  reduce.py
  topk.py
  numerics.py
  tensor.py
  distributed.py
  compaction.py
  swiglu.py
  roofline.py
  specialize.py
  testing.py
  matmul_details/
  tensor_details/
  topk_details/
  numerics_details/
  distributed_details/
```

## 11. Backend Plugin System / 后端插件系统

- **EN:** Triton supports two different but related extension stories: backend discovery through Python entry points, and MLIR pass plugins that customize compiler pipelines.
- **CN:** Triton 支持两类相关但不同的扩展机制：一类是通过 Python entry point 发现后端，另一类是通过 MLIR pass 插件自定义编译流水线。

- **EN:** Backend discovery lives in `python/triton/backends/__init__.py` and uses the entry-point group `triton.backends`.
- **CN:** 后端发现逻辑位于 `python/triton/backends/__init__.py`，使用 entry-point group `triton.backends`。

- **EN:** The abstract backend contract is defined in `python/triton/backends/compiler.py` by the `BaseBackend` class.
- **CN:** 抽象后端契约由 `python/triton/backends/compiler.py` 中的 `BaseBackend` 类定义。

- **EN:** A backend must answer whether it supports a target, provide a stable hash, parse options, add stages, load dialects, and provide a module map.
- **CN:** 一个后端必须回答自己是否支持某个 target，并提供稳定哈希、选项解析、stage 添加、dialect 加载以及 module map。

- **EN:** This means that adding a backend is a full compiler/runtime integration task, not only a codegen task.
- **CN:** 这意味着新增后端是一项完整的编译器/运行时集成工作，而不仅仅是 codegen 工作。

- **EN:** For pass plugins, `examples/plugins/README.md` explains how to build an out-of-tree shared library and load it at JIT time with `TRITON_PLUGIN_PATHS`.
- **CN:** 对于 pass 插件，`examples/plugins/README.md` 说明了如何构建树外共享库，并通过 `TRITON_PLUGIN_PATHS` 在 JIT 时加载。

- **EN:** The same README shows that plugins can either be inspected through `triton-opt` or inserted into the runtime pipeline with `knobs.runtime.add_stages_inspection_hook`.
- **CN:** 同一个 README 还展示了：插件既可以通过 `triton-opt` 单独观察，也可以借助 `knobs.runtime.add_stages_inspection_hook` 插入运行时编译流水线。

- **EN:** The example C++ plugin in `examples/plugins/TritonPlugin.cpp` registers a pass and exposes `tritonGetPluginInfo()` to Triton.
- **CN:** `examples/plugins/TritonPlugin.cpp` 中的示例 C++ 插件注册了一个 pass，并通过 `tritonGetPluginInfo()` 暴露给 Triton。

- **EN:** In the sample code, the pass simply renames a function, but the mechanism could support custom ops, instrumentation, or downstream optimizations.
- **CN:** 示例代码中的 pass 只是简单重命名函数，但这一机制完全可以承载自定义 op、插桩或下游优化。

- **EN:** The plugin hook model is powerful because it allows per-kernel pipeline modification without rebuilding the whole Triton compiler.
- **CN:** 这种插件 hook 模型非常强大，因为它允许你在不重编整个 Triton 编译器的情况下，对特定内核修改编译流水线。

- **EN:** This is especially valuable for hardware vendors, downstream frameworks, and researchers experimenting with custom passes.
- **CN:** 这对硬件厂商、下游框架以及研究自定义 pass 的开发者尤为有价值。

- **EN:** If you are adding a new backend, study `BaseBackend` first; if you are adding a new pass, study `examples/plugins/README.md` and `TritonPlugin.cpp` first.
- **CN:** 如果你要新增后端，先读 `BaseBackend`；如果你要新增 pass，先读 `examples/plugins/README.md` 和 `TritonPlugin.cpp`。

- **EN:** Together, these mechanisms show that Triton’s extension story is serious and designed for downstream use, not only in-tree development.
- **CN:** 综合来看，这些机制说明 Triton 的扩展体系是面向真实下游使用设计的，而不只是仓库内自用。

- **EN:** A helpful mental split is: backends define whole-codegen targets; plugins surgically modify compiler behavior inside a target.
- **CN:** 一个有用的心智划分是：backend 定义完整 codegen target，而 plugin 则在某个 target 内部对编译器行为做手术式修改。

```python
# Simplified plugin-hook idea from examples/plugins/README.md
from triton import knobs

def inspect_stages_hook(self=None, stages=None, options=None, language=None, capability=None):
    if all(arg is None for arg in (stages, options, language, capability)):
        return "plugin-key", "plugin-hash"

    # Replace or wrap a stage here
    stages["ttir"] = lambda src, metadata: self.make_ttir(src, metadata, options, capability)
    return "plugin-key", "plugin-hash"

knobs.runtime.add_stages_inspection_hook = inspect_stages_hook
```

## 12. Testing & Debugging / 测试与调试

- **EN:** The official user-facing debugging guide is `docs/programming-guide/chapter-3/debugging.rst`.
- **CN:** 面向用户的官方调试指南是 `docs/programming-guide/chapter-3/debugging.rst`。

- **EN:** That document distinguishes compile-time debugging (`static_print`, `static_assert`) from runtime debugging (`device_print`, `device_assert`).
- **CN:** 该文档把编译期调试（`static_print`、`static_assert`）与运行期调试（`device_print`、`device_assert`）区分开来。

- **EN:** `device_assert` only executes when `TRITON_DEBUG=1`, while other debug operators always execute.
- **CN:** `device_assert` 只有在 `TRITON_DEBUG=1` 时才会执行，而其他调试操作始终有效。

- **EN:** The same guide strongly recommends the interpreter mode enabled by `TRITON_INTERPRET=1`.
- **CN:** 同一篇指南强烈推荐通过 `TRITON_INTERPRET=1` 启用解释器模式。

- **EN:** In interpreter mode, Triton kernels bypass GPU compilation and are simulated using NumPy equivalents one operation at a time.
- **CN:** 在解释器模式下，Triton 内核会绕过 GPU 编译，改由 NumPy 等价实现逐操作模拟执行。

- **EN:** This is especially useful because you can use normal Python `print` and `pdb` inside a Triton kernel body.
- **CN:** 这一模式尤其好用，因为你可以在 Triton 内核体里直接使用普通 Python `print` 与 `pdb`。

- **EN:** The docs also list interpreter limitations, including lack of support for `bfloat16` arithmetic and indirect memory access patterns.
- **CN:** 文档同时列出了解释器限制，包括不支持 `bfloat16` 算术和某些间接内存访问模式。

- **EN:** For NVIDIA GPUs, the guide recommends `compute-sanitizer`; for AMD GPUs, ROCm LLVM AddressSanitizer is suggested.
- **CN:** 对 NVIDIA GPU，指南推荐 `compute-sanitizer`；对 AMD GPU，则建议使用 ROCm 的 LLVM AddressSanitizer。

- **EN:** It also points to `triton-viz` for memory-access visualization.
- **CN:** 文档还推荐用 `triton-viz` 做内存访问可视化。

- **EN:** The deeper floating-point instrumentation guide is `docs/programming-guide/chapter-3/fpsan.rst`.
- **CN:** 更深入的浮点插桩文档是 `docs/programming-guide/chapter-3/fpsan.rst`。

- **EN:** FpSan is described there as a compiler instrumentation mode that rewrites selected floating-point ops into deterministic payload algebra over integer bit patterns.
- **CN:** FpSan 在该文档中被描述为一种编译器插桩模式：它把选定浮点操作改写到基于整数比特模式的确定性 payload algebra 上。

- **EN:** It is not an IEEE simulator; it is a structure-preserving kernel-checking tool.
- **CN:** 它不是 IEEE 浮点模拟器，而是一种保结构的内核校验工具。

- **EN:** You can enable FpSan in Python with `triton.knobs.compilation.instrumentation_mode = "fpsan"` or from the shell with `TRITON_INSTRUMENTATION_MODE=fpsan`.
- **CN:** 你可以在 Python 中用 `triton.knobs.compilation.instrumentation_mode = "fpsan"` 启用 FpSan，也可以在 shell 中使用 `TRITON_INSTRUMENTATION_MODE=fpsan`。

- **EN:** The repo also contains low-level MLIR tests for FpSan and global sanitizer, such as `test/TritonGPU/fpsan.mlir`, `test/TritonGPU/gsan.mlir`, and `test/Conversion/tritongpu_to_llvm_gsan.mlir`.
- **CN:** 仓库还包含 FpSan 与 global sanitizer 的底层 MLIR 测试，例如 `test/TritonGPU/fpsan.mlir`、`test/TritonGPU/gsan.mlir` 和 `test/Conversion/tritongpu_to_llvm_gsan.mlir`。

- **EN:** Python-side debug tests are visible in files like `python/test/unit/test_debug.py`, `python/test/unit/test_debug_dump.py`, and `python/test/gluon/test_fpsan.py`.
- **CN:** Python 侧的调试测试可以在 `python/test/unit/test_debug.py`、`python/test/unit/test_debug_dump.py` 和 `python/test/gluon/test_fpsan.py` 等文件中看到。

- **EN:** `docs/python-api/triton.testing.rst` documents utilities such as `Benchmark`, `do_bench`, `do_bench_cudagraph`, `perf_report`, and `assert_close`.
- **CN:** `docs/python-api/triton.testing.rst` 记录了 `Benchmark`、`do_bench`、`do_bench_cudagraph`、`perf_report` 和 `assert_close` 等工具。

- **EN:** From a workflow perspective, Triton debugging usually alternates between three levels: Python/kernel logic, MLIR/LLVM lowering, and actual hardware execution.
- **CN:** 从工作流角度看，Triton 调试通常在三层之间切换：Python/内核逻辑、MLIR/LLVM lowering，以及真实硬件执行。

- **EN:** You should choose the lightest tool that answers the question: interpreter for logic, IR dumps for compiler bugs, sanitizers for memory/synchronization issues.
- **CN:** 应始终选择最轻量、最直接的工具：逻辑问题用解释器，编译器 bug 看 IR dump，内存/同步问题用 sanitizer。

```bash
# Common debugging commands / 常见调试命令
TRITON_INTERPRET=1 python my_script.py
TRITON_DEBUG=1 python my_script.py
MLIR_ENABLE_DUMP=1 python my_script.py
LLVM_IR_ENABLE_DUMP=1 python my_script.py
TRITON_INSTRUMENTATION_MODE=fpsan python my_script.py
compute-sanitizer python my_script.py
```

## 13. Performance Guide / 性能指南

- **EN:** The first performance rule in Triton is to think in tiles, not scalar operations.
- **CN:** Triton 的第一条性能规则是：按 tile 思考，而不是按单个标量操作思考。

- **EN:** The second rule is to minimize HBM traffic by fusion, reuse, and on-chip staging.
- **CN:** 第二条规则是：通过融合、复用和片上 staging 最小化 HBM 访问。

- **EN:** `02-fused-softmax.py`, `05-layer-norm.py`, and `06-fused-attention.py` all exist primarily because memory traffic dominates naive decompositions.
- **CN:** `02-fused-softmax.py`、`05-layer-norm.py` 和 `06-fused-attention.py` 的核心动机，都是因为朴素拆分下内存流量占主导。

- **EN:** Choose block sizes that balance occupancy, register pressure, shared-memory use, and vectorization opportunities.
- **CN:** 选择 block size 时，要在 occupancy、寄存器压力、共享内存占用和向量化机会之间做平衡。

- **EN:** This is why `@triton.autotune` is so central in matmul-family kernels.
- **CN:** 这也正是为什么 `@triton.autotune` 在 matmul 家族内核中如此重要。

- **EN:** Coalescing matters: layouts and pointer arithmetic should make adjacent lanes read adjacent memory whenever possible.
- **CN:** coalescing 很重要：布局和指针运算应尽量让相邻 lane 访问相邻地址。

- **EN:** `03-matrix-multiplication.py` uses grouped ordering (`GROUP_SIZE_M`) to improve L2 locality, which is a reminder that launch order affects cache behavior.
- **CN:** `03-matrix-multiplication.py` 用分组顺序（`GROUP_SIZE_M`）改善 L2 局部性，这提醒我们 launch 顺序会直接影响缓存行为。

- **EN:** Mask tails correctly but do not let masking become your whole boundary strategy if a descriptor or better tiling scheme is available.
- **CN:** 尾块掩码必须正确，但如果 descriptor 或更优 tiling 方案可用，就不要让掩码成为你唯一的边界策略。

- **EN:** Use shared memory or TMA when repeated global loads and address computation become the bottleneck.
- **CN:** 当重复的全局加载与地址计算成为瓶颈时，应考虑共享内存或 TMA。

- **EN:** Use TMA descriptors on Hopper/Blackwell when shape and alignment constraints are satisfied, because they reduce register pressure and enable stronger pipelines.
- **CN:** 在 Hopper/Blackwell 上，如果 shape 与对齐条件满足，应优先考虑 TMA descriptor，因为它们能降低寄存器压力并构造更强流水线。

- **EN:** Pipeline long-latency work: async copies, async MMA, and persistent outer loops are all ways to keep the machine busy.
- **CN:** 要把长延迟工作流水化：异步 copy、异步 MMA 和持久化外层循环，都是让机器持续忙碌的方法。

- **EN:** Warp specialization is powerful when different work types can overlap, but it should be justified because it increases complexity and synchronization cost.
- **CN:** 当不同类型工作能互相重叠时，warp specialization 非常强大；但它会增加复杂度和同步成本，因此必须有充分理由。

- **EN:** Persistent kernels help when launch overhead or device scheduling overhead is material relative to tile work.
- **CN:** 当启动开销或设备调度开销相对于 tile 计算量变得不可忽略时，持久化内核会很有帮助。

- **EN:** For low precision, pay as much attention to data layout, scale layout, and packing as to the actual multiply-accumulate instruction.
- **CN:** 在低精度场景下，对数据布局、scale 布局与打包方式的关注，往往与对乘加指令本身的关注同样重要。

- **EN:** Always benchmark with realistic shapes and compare against a reference such as PyTorch, cuBLAS, or rocBLAS where appropriate.
- **CN:** 务必使用真实形状做 benchmark，并在适当情况下与 PyTorch、cuBLAS 或 rocBLAS 等参考实现比较。

- **EN:** Use `triton.testing.do_bench`, `do_bench_cudagraph`, and `perf_report` to keep benchmarking code reproducible and close to repo style.
- **CN:** 使用 `triton.testing.do_bench`、`do_bench_cudagraph` 与 `perf_report`，让 benchmark 代码保持可复现并贴近仓库风格。

- **EN:** Keep correctness checks in the loop while tuning. Triton’s tutorials repeatedly compare outputs against Torch before reporting performance.
- **CN:** 调优时始终把正确性校验留在回路里。Triton 教程在报告性能前，都会反复与 Torch 输出对比。

- **EN:** Do not overfit one GPU generation: many tutorial files contain CUDA/HIP branches, Hopper/Blackwell branches, and architecture-specific pruning logic for a reason.
- **CN:** 不要对某一代 GPU 过度拟合：很多教程同时包含 CUDA/HIP 分支、Hopper/Blackwell 分支以及架构特定剪枝逻辑，这是有原因的。

- **EN:** Performance engineering in Triton is often “host + kernel” co-design: preprocessing, descriptor construction, and launch policy matter as much as the kernel body.
- **CN:** Triton 的性能工程经常是“主机侧 + 内核侧”的协同设计：预处理、descriptor 构造和 launch 策略与内核体同样重要。

- **EN:** If you need a short checklist, use: coalesce, stage, pipeline, tune, verify, profile.
- **CN:** 如果你需要一个简短清单，可以记住：coalesce、stage、pipeline、tune、verify、profile。

```text
Performance checklist / 性能检查清单
1. Does each program own a useful tile?
2. Are global loads/stores coalesced?
3. Can data be reused from SRAM / shared memory / tensor memory?
4. Are masks only used where unavoidable?
5. Is the launch order cache-friendly?
6. Should num_warps / num_stages / num_ctas be autotuned?
7. Can async copy or async MMA hide latency?
8. Would persistence or warp specialization help?
9. Is the result numerically validated?
10. Is the benchmark realistic?
```

## 14. Project Structure / 项目结构

- **EN:** At the top level, the repo clearly separates docs, Python frontend/runtime code, C++/MLIR compiler code, tests, tutorials, and kernel-library code.
- **CN:** 在顶层目录上，仓库把文档、Python 前端/运行时代码、C++/MLIR 编译器代码、测试、教程和内核库代码清楚地分开。

- **EN:** `python/triton/` is the main Python package containing the language frontend, compiler orchestration, runtime, and utility modules.
- **CN:** `python/triton/` 是主 Python 包，包含语言前端、编译器编排、运行时和工具模块。

- **EN:** `python/tutorials/` and `python/tutorials/gluon/` are the most direct learning resources for end users and contributors alike.
- **CN:** `python/tutorials/` 与 `python/tutorials/gluon/` 是用户和贡献者最直接的学习材料。

- **EN:** `python/triton_kernels/` hosts a reusable kernel library layer on top of core Triton.
- **CN:** `python/triton_kernels/` 承载了构建在核心 Triton 之上的可复用内核库层。

- **EN:** `docs/` is the Sphinx documentation source tree, including getting-started material, programming guide chapters, API docs, and meetup notes.
- **CN:** `docs/` 是 Sphinx 文档源码树，包含入门材料、编程指南章节、API 文档以及 meetup 记录。

- **EN:** `lib/`, `include/`, and `bin/` hold the C++/MLIR dialect definitions, passes, tools, and registration code.
- **CN:** `lib/`、`include/` 与 `bin/` 保存 C++/MLIR 方言定义、pass、工具和注册代码。

- **EN:** `test/` contains MLIR/compiler/backend regression tests, while `python/test/` focuses more on Python/runtime/frontend behavior.
- **CN:** `test/` 主要放 MLIR/编译器/后端回归测试，而 `python/test/` 更偏向 Python/运行时/前端行为测试。

- **EN:** `examples/plugins/` is the best small reference for pass-plugin customization.
- **CN:** `examples/plugins/` 是了解 pass 插件自定义机制的最佳小型参考目录。

- **EN:** The project layout therefore mirrors the mental model of the system: frontend → compiler → runtime → tutorials/libraries/tests.
- **CN:** 因此，项目结构本身就映射了系统心智模型：frontend → compiler → runtime → tutorials/libraries/tests。

- **EN:** `CONTRIBUTING.md` in this snapshot is governance-focused: it defines contributors, module maintainers, core maintainers, and the lead core maintainer.
- **CN:** 当前版本的 `CONTRIBUTING.md` 重点在治理结构：它定义了 contributors、module maintainers、core maintainers 和 lead core maintainer。

- **EN:** That file also communicates an important social fact about the project: high-quality pull requests are encouraged, but review bandwidth is limited and maintainers prioritize selectively.
- **CN:** 该文件还传达了一个重要的项目现实：高质量 PR 虽然受到欢迎，但 review 带宽有限，维护者会选择性优先处理。

- **EN:** The listed in-tree modules include the AMD backend, interpreter, and profiler, while some other backends are described as out-of-tree modules.
- **CN:** 文中列出的树内模块包括 AMD backend、interpreter 和 profiler，而其他一些后端则被描述为树外模块。

- **EN:** This matters for contributors because it tells you where ownership is concentrated and why some areas may have different review patterns.
- **CN:** 这对贡献者很重要，因为它能告诉你哪些区域的责任归属更明确，以及为什么不同区域的 review 模式可能不同。

- **EN:** In other words, project structure in Triton is not just technical layout; it also reflects governance and extension boundaries.
- **CN:** 换句话说，Triton 的项目结构不只是技术目录布局，也反映了治理方式和扩展边界。

- **EN:** When orienting yourself in the repo, start from tutorials, then read `python/triton/*`, then trace into `lib/` and `test/` for the pieces you care about.
- **CN:** 熟悉仓库时，建议先从教程开始，再读 `python/triton/*`，最后根据兴趣深入 `lib/` 与 `test/`。

- **EN:** That reading order matches how most developers learn Triton: first use it, then inspect it, then extend it.
- **CN:** 这个阅读顺序也符合大多数开发者学习 Triton 的方式：先会用，再看懂，最后扩展。

```text
/root/xw/triton/
  README.md
  CONTRIBUTING.md
  docs/
    getting-started/
    programming-guide/
    python-api/
  python/
    triton/
      language/
      compiler/
      runtime/
      backends/
    tutorials/
      01-vector-add.py
      ...
      11-programmatic-dependent-launch.py
      gluon/
        01-intro.py
        ...
        14-multicta.py
    triton_kernels/
  lib/
    Dialect/
    Conversion/
    Target/
    Tools/
  include/
  test/
  python/test/
  examples/plugins/
```

## Appendix A. Environment Variables & Knobs / 附录A：环境变量与 Knobs

- **EN:** `MLIR_ENABLE_DUMP=1` dumps IR before every MLIR pass; the README also notes you can target a specific kernel name.
- **CN:** `MLIR_ENABLE_DUMP=1` 会在每个 MLIR pass 前 dump IR；README 还提到可以只针对指定 kernel 名。

- **EN:** `MLIR_DUMP_PATH` controls where MLIR dumps are written.
- **CN:** `MLIR_DUMP_PATH` 控制 MLIR dump 的输出位置。

- **EN:** `LLVM_IR_ENABLE_DUMP=1` dumps LLVM IR before each LLVM pass.
- **CN:** `LLVM_IR_ENABLE_DUMP=1` 会在每个 LLVM pass 前 dump LLVM IR。

- **EN:** `TRITON_REPRODUCER_PATH=...` asks Triton to emit MLIR reproducer files around compiler stages.
- **CN:** `TRITON_REPRODUCER_PATH=...` 会让 Triton 在编译阶段附近生成 MLIR reproducer 文件。

- **EN:** `TRITON_INTERPRET=1` switches execution to the interpreter.
- **CN:** `TRITON_INTERPRET=1` 会把执行切换到解释器。

- **EN:** `TRITON_ENABLE_LLVM_DEBUG=1` enables noisy LLVM debug output.
- **CN:** `TRITON_ENABLE_LLVM_DEBUG=1` 会打开较为噪声化的 LLVM 调试输出。

- **EN:** `TRITON_LLVM_DEBUG_ONLY=...` narrows LLVM debug output to specific components.
- **CN:** `TRITON_LLVM_DEBUG_ONLY=...` 可以把 LLVM 调试输出限制到指定组件。

- **EN:** `TRITON_ENABLE_ASAN=1` enables address-sanitizer integration on supported paths.
- **CN:** `TRITON_ENABLE_ASAN=1` 会在支持的路径上启用地址消毒相关集成。

- **EN:** `USE_IR_LOC={ttir,ttgir}` remaps location info to IR files for profiling/debugging.
- **CN:** `USE_IR_LOC={ttir,ttgir}` 会把位置信息重映射到 IR 文件，便于分析与调试。

- **EN:** `TRITON_PRINT_AUTOTUNING=1` prints best autotune config and tuning time.
- **CN:** `TRITON_PRINT_AUTOTUNING=1` 会打印最佳 autotune 配置和调优耗时。

- **EN:** `DISABLE_LLVM_OPT` disables LLVM optimization in selected stages.
- **CN:** `DISABLE_LLVM_OPT` 可关闭部分阶段中的 LLVM 优化。

- **EN:** `TRITON_ALWAYS_COMPILE=1` forces recompilation instead of cache reuse.
- **CN:** `TRITON_ALWAYS_COMPILE=1` 会强制重新编译而不是复用缓存。

- **EN:** `TRITON_KERNEL_DUMP=1` together with `TRITON_DUMP_DIR` is useful for saving kernel artifacts.
- **CN:** `TRITON_KERNEL_DUMP=1` 配合 `TRITON_DUMP_DIR` 可用于保存内核产物。

- **EN:** `TRITON_KERNEL_OVERRIDE=1` together with `TRITON_OVERRIDE_DIR` supports overriding stage artifacts.
- **CN:** `TRITON_KERNEL_OVERRIDE=1` 配合 `TRITON_OVERRIDE_DIR` 支持替换阶段产物。

- **EN:** `TRITON_FRONT_END_DEBUGGING=1` keeps frontend traceback details that are otherwise filtered.
- **CN:** `TRITON_FRONT_END_DEBUGGING=1` 会保留原本会被过滤掉的前端 traceback 细节。

- **EN:** `TRITON_DEFAULT_BACKEND` selects the active backend driver explicitly.
- **CN:** `TRITON_DEFAULT_BACKEND` 用于显式指定当前活动后端驱动。

- **EN:** `TRITON_INSTRUMENTATION_MODE=fpsan` enables floating-point sanitizer instrumentation from the shell.
- **CN:** `TRITON_INSTRUMENTATION_MODE=fpsan` 可在 shell 中启用浮点 sanitizer 插桩。

- **EN:** `TRITON_PLUGIN_PATHS` and `TRITON_EXT_ENABLED` are part of the plugin extension story shown in `examples/plugins/README.md`.
- **CN:** `TRITON_PLUGIN_PATHS` 和 `TRITON_EXT_ENABLED` 是 `examples/plugins/README.md` 中展示的插件扩展机制的一部分。

## Appendix B. Glossary / 附录B：术语表

- **EN:** Program instance: Triton’s unit of SPMD execution; each instance usually owns one tile of work.
- **CN:** Program instance：Triton 的 SPMD 执行单位；通常每个实例拥有一个工作 tile。

- **EN:** Tile / block: a rectangular or otherwise structured chunk of tensor elements processed together.
- **CN:** Tile / block：一起处理的一块矩形或结构化张量元素。

- **EN:** TTIR: Triton IR, the MLIR dialect closest to the Python DSL.
- **CN:** TTIR：Triton IR，是最接近 Python DSL 的 MLIR 方言。

- **EN:** TTGIR: TritonGPU IR, the GPU-oriented dialect after Triton IR lowering.
- **CN:** TTGIR：TritonGPU IR，是 Triton IR 之后面向 GPU 的方言。

- **EN:** Tensor Descriptor: a compact object describing multidimensional memory for descriptor-backed loads/stores, especially TMA.
- **CN:** Tensor Descriptor：描述多维内存的紧凑对象，可支持 descriptor-backed load/store，尤其适用于 TMA。

- **EN:** Block pointer: an older block-access abstraction that `core.py` now marks as deprecated in favor of descriptors.
- **CN:** Block pointer：较早期的块访问抽象，`core.py` 现在更推荐用 descriptor 取代。

- **EN:** TMA: Tensor Memory Accelerator, Hopper/Blackwell hardware for descriptor-driven tensor transfer.
- **CN:** TMA：Tensor Memory Accelerator，是 Hopper/Blackwell 上基于 descriptor 的张量传输硬件。

- **EN:** WGMMA: Hopper warp-group matrix multiply-accumulate instruction family.
- **CN:** WGMMA：Hopper 的 warp-group 矩阵乘加指令族。

- **EN:** TCGen05 / MMAv5: Blackwell-era tensor-core programming model using tensor memory.
- **CN:** TCGen05 / MMAv5：Blackwell 时代基于 tensor memory 的张量核编程模型。

- **EN:** Tensor memory (TMEM): a Blackwell memory space used by new tensor-core instructions.
- **CN:** Tensor memory（TMEM）：Blackwell 新型张量核指令使用的内存空间。

- **EN:** Persistent kernel: a kernel where each CTA/program instance processes multiple work tiles over time instead of exiting after one tile.
- **CN:** Persistent kernel：每个 CTA/program instance 会连续处理多个 work tile，而不是处理一个 tile 后立刻退出的内核。

- **EN:** Warp specialization: assigning different warps in one kernel to different roles such as load/compute/store.
- **CN:** Warp specialization：在同一内核中，把不同 warp 分配为 load/compute/store 等不同角色。

- **EN:** CGA / cluster: a multi-CTA cooperation group available on modern NVIDIA architectures.
- **CN:** CGA / cluster：现代 NVIDIA 架构上可用的多 CTA 协作组。

- **EN:** Autotuning: benchmarking multiple meta-parameter configurations and caching the best one for future runs.
- **CN:** Autotuning：对多组元参数配置做 benchmark，并缓存最佳结果以供后续复用。

- **EN:** FpSan: a floating-point sanitizer mode that preserves selected algebraic structure for kernel checking.
- **CN:** FpSan：一种保留特定代数结构的浮点 sanitizer 模式，用于内核校验。

- **EN:** PDL / GDC: programmatic dependent launch / GPU dependency control style runtime coordination shown in tutorial 11.
- **CN:** PDL / GDC：在教程 11 中展示的程序化依赖启动 / GPU 依赖控制风格运行时协调机制。

- **EN:** MX / microscaling: low-precision numeric formats with explicit scale tensors, used by `tl.dot_scaled` and related kernels.
- **CN:** MX / microscaling：带显式 scale 张量的低精度数值格式，被 `tl.dot_scaled` 及相关内核使用。

- **EN:** Epilogue: computation performed after the main matmul accumulation, such as bias, activation, scaling, or communication.
- **CN:** Epilogue：主 matmul 累加完成后执行的计算，例如 bias、activation、scaling 或通信。

## Appendix C. Reading Order Recommendation / 附录C：推荐阅读顺序

- **EN:** Start with `README.md` and `docs/getting-started/installation.rst` to set up the environment.
- **CN:** 先读 `README.md` 与 `docs/getting-started/installation.rst` 完成环境准备。

- **EN:** Then read `python/tutorials/01-vector-add.py` through `03-matrix-multiplication.py` for core language rhythm.
- **CN:** 然后读 `python/tutorials/01-vector-add.py` 到 `03-matrix-multiplication.py`，建立核心语言节奏感。

- **EN:** Next read `05-layer-norm.py` and `06-fused-attention.py` to see realistic fused kernels.
- **CN:** 接着读 `05-layer-norm.py` 和 `06-fused-attention.py`，看真实融合内核。

- **EN:** After that, read `python/triton/language/core.py` selectively around `program_id`, `load`, `store`, `dot`, `reduce`, and tensor descriptors.
- **CN:** 之后选择性阅读 `python/triton/language/core.py` 中与 `program_id`、`load`、`store`、`dot`、`reduce` 和 tensor descriptor 相关的部分。

- **EN:** Then read `python/triton/runtime/jit.py` and `python/triton/compiler/compiler.py` together.
- **CN:** 再把 `python/triton/runtime/jit.py` 和 `python/triton/compiler/compiler.py` 配套阅读。

- **EN:** Finally, enter Gluon once you want to reason directly about layouts, shared memory, TMA, WGMMA, tcgen05, and clusters.
- **CN:** 最后，当你想直接思考布局、共享内存、TMA、WGMMA、tcgen05 和 cluster 时，再进入 Gluon。

