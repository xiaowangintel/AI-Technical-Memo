# development_jit_kernel_guide.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/developer_guide/development_jit_kernel_guide.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Environment Setup We strongly recommend using clangd as the language server for JIT kernel development. For Ubuntu/Debian, you can download clangd from apt.llvm.org. / 该文档围绕 Development Guide for JIT Kernels 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Environment Setup
**EN:** We strongly recommend using clangd as the language server for JIT kernel development. For Ubuntu/Debian, you can download clangd from apt.llvm.org.
**CN:** 本节围绕 Environment Setup 展开，概述了 clangd, JIT, IDE, AOT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: C++ Implementation
**EN:** C++ source code is located in python/sglang/jit_kernel/csrc. Reusable functions should be placed in python/sglang/jit_kernel/include.
**CN:** 本节围绕 C++ Implementation 展开，概述了 Refer, Reusable, Typically, tvm::ffi::TensorView 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Python Interface
**EN:** Python interfaces are defined in python/sglang/jit_kernel. The load_jit utility function in python/sglang/jit_kernel/utils.py loads and returns the compiled module.
**CN:** 本节围绕 Python Interface 展开，概述了 load_jit, functools.lru_cache, cpp_func, function 等要点，并说明相关配置、流程、示例或限制条件。

### Section: C++ Utilities
**EN:** The following C++ utilities are available: #### Integer Range Similar to PyTorch, we provide an irange function to represent an integer range.
**CN:** 本节围绕 C++ Utilities 展开，概述了 TensorMatcher, auto, LaunchKernel, include 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Add new kernels
**EN:** This section walks through a complete, end-to-end example of adding a new JIT kernel to the system. We use a simple add_constant kernel as a running example, which adds a constant integer value to every element of an input tensor.
**CN:** 本节围绕 Add new kernels 展开，概述了 JIT, src, kernel, Conceptually 等要点，并说明相关配置、流程、示例或限制条件。

### Section: STEP 1: Write the C++ kernel
**EN:** Write your CUDA kernel in jit_kernel/csrc/add_constant.cuh. For demonstration purposes, we pass the constant value as a template parameter.
**CN:** 本节围绕 STEP 1: Write the C++ kernel 展开，概述了 include, int32_t, num_elements, dst 等要点，并说明相关配置、流程、示例或限制条件。

### Section: STEP 2: Create Python Interfaces
**EN:** Next, expose the kernel through a Python wrapper. Create a new file at jit_kernel/add_constant.py and expose the needed interfaces.
**CN:** 本节围绕 STEP 2: Create Python Interfaces 展开，概述了 import, src, Module, constant 等要点，并说明相关配置、流程、示例或限制条件。

### Section: STEP 3: Use your kernel
**EN:** Finally, import and use the kernel like a regular Python function: ``python from sglang.jit_kernel.add_constant import add_constant `` For a complete, runnable example, refer to test_add_constant.py.
**CN:** 本节围绕 STEP 3: Use your kernel 展开，概述了 import, Finally, like, refer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: C++ Include Library Reference
**EN:** The JIT kernel framework provides a set of reusable C++ headers in python/sglang/jit_kernel/include/sgl_kernel/. Each header is designed to be lightweight and self-contained.
**CN:** 本节围绕 C++ Include Library Reference 展开，概述了 APIs, Below, header, python/sglang/jit_kernel/include/sgl_kernel/ 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Core Utilities
**EN:** This section provides a comparison table for Core Utilities, covering columns such as Header, Namespace, Purpose and examples such as utils.h, utils.cuh, source_location.h, runtime.cuh.
**CN:** 本节围绕 Core Utilities 展开，概述了 host, Panic, SGL_DEVICE, RuntimeCheck 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tensor Validation
**EN:** This section provides a comparison table for Tensor Validation, covering columns such as Header, Namespace, Purpose and examples such as tensor.h.
**CN:** 本节围绕 Tensor Validation 展开，概述了 SymbolicSize, TensorMatcher, SymbolicDType, SymbolicDevice 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Math & Type System
**EN:** This section provides a comparison table for Math & Type System, covering columns such as Header, Namespace, Purpose and examples such as math.cuh, type.cuh.
**CN:** 本节围绕 Math & Type System 展开，概述了 max, min, abs, exp 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Memory Access
**EN:** This section provides a comparison table for Memory Access, covering columns such as Header, Namespace, Purpose and examples such as vec.cuh, tile.cuh.
**CN:** 本节围绕 Memory Access 展开，概述了 bit, device, Header, Memory 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Parallel Primitives
**EN:** This section provides a comparison table for Parallel Primitives, covering columns such as Header, Namespace, Purpose and examples such as warp.cuh, cta.cuh, atomic.cuh.
**CN:** 本节围绕 Parallel Primitives 展开，概述了 reduce_max, max, via, CUDA 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Reusable Kernel Templates
**EN:** This section provides a comparison table for Reusable Kernel Templates, covering columns such as Header, Namespace, Purpose and examples such as impl/norm.cuh.
**CN:** 本节围绕 Reusable Kernel Templates 展开，概述了 StorageType, CTA, Header, Purpose 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** kernel / **CN:** kernel
- **EN:** TensorMatcher / **CN:** TensorMatcher
- **EN:** device / **CN:** device
- **EN:** include / **CN:** include
- **EN:** LaunchKernel / **CN:** LaunchKernel
- **EN:** Header / **CN:** Header
- **EN:** Purpose / **CN:** Purpose
- **EN:** Namespace / **CN:** Namespace

## Dependencies / 依赖关系
- `python/sglang/jit_kernel/utils.py`
- `../../python/sglang/jit_kernel/csrc/add_constant.cuh`
- `../../python/sglang/jit_kernel/add_constant.py`
- `../../python/sglang/jit_kernel/tests/test_add_constant.py`
- `jit_kernel/add_constant.py`
- `test_add_constant.py`
