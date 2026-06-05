# debugging.rst — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/triton/docs/programming-guide/chapter-3/debugging.rst`
- **EN:** Programming-guide chapter page for debugging.rst.
- **CN:** debugging.rst 对应的编程指南章节页面。

## Content Analysis / 内容分析
### Debugging Triton
**EN:** This section introduces the overall debugging toolbox available to Triton users and developers. Notable prose emphasis: This tutorial provides guidance for debugging Triton programs. It is mostly documented for Triton users.
**CN:** 本节总体介绍 Triton 用户和开发者可用的调试工具箱。 其中反复出现的技术关键词包括 Floating-Point Sanitizer (FpSan) <fpsan>、Triton、debugging、code、tutorial、guidance。

### Using Triton's Debugging Operations
**EN:** This section explains Triton's built-in compile-time and runtime debug operators. Key listed points include static_print and static_assert are intended for compile-time debugging. and device_print and device_assert are used for runtime debugging.. Notable prose emphasis: Triton includes four debugging operators that allow users to check and inspect tensor values Other debugging operators execute regardless of the value of TRITON_DEBUG.
**CN:** 本节说明 Triton 内置的编译期与运行期调试操作。 列出的重点包括 static_print and static_assert are intended for compile-time debugging.、device_print and device_assert are used for runtime debugging.。 其中反复出现的技术关键词包括 static_print、static_assert、device_print、device_assert、TRITON_DEBUG、1。

### Using the Interpreter
**EN:** This section explains how interpreter mode can execute kernels on CPU for inspection and step-by-step debugging. Key listed points include Print the intermediate results of each operation using the Python print function. To inspect an entire tensor, use print(tensor). To examine individual tensor values at idx, use print(tensor.handle.data[idx])., Attach pdb for step-by-step debugging of the Triton program, and Import the pdb package and set breakpoints in the Triton program. Notable prose emphasis: The interpreter is a straightforward and helpful tool for debugging Triton programs. It allows Triton users to run Triton programs on the CPU and inspect the intermediate results of each operation.
**CN:** 本节说明如何通过解释器模式在 CPU 上执行内核，以便观察中间结果和逐步调试。 列出的重点包括 Print the intermediate results of each operation using the Python print function. To inspect an entire tensor, use print(tensor). To examine individual tensor values at idx, use print(tensor.handle.data[idx]).、Attach pdb for step-by-step debugging of the Triton program、Import the pdb package and set breakpoints in the Triton program。 其中反复出现的技术关键词包括 TRITON_INTERPRET、1、print、print(tensor)、idx、print(tensor.handle.data[idx])。

### Limitations
**EN:** This section highlights the practical or conceptual limits of the approach under discussion. Key listed points include It does not support operations on bfloat16 numeric types. To perform operations on bfloat16 tensors, use tl.cast(tensor) to convert the tensor to float32. and It does not support indirect memory access patterns such as. Notable prose emphasis: The interpreter has several known limitations
**CN:** 本节强调当前讨论方法在实践或概念层面的局限。 列出的重点包括 It does not support operations on bfloat16 numeric types. To perform operations on bfloat16 tensors, use tl.cast(tensor) to convert the tensor to float32.、It does not support indirect memory access patterns such as。 其中反复出现的技术关键词包括 bfloat16、tl.cast(tensor)、float32、ptr、does、not。

### Using Third-party Tools
**EN:** This section recommends external GPU debugging and visualization tools for NVIDIA, AMD, and general workflows. Notable prose emphasis: For debugging on NVIDIA GPUs, compute-sanitizer is an effective tool for checking data races and memory access issues. To use it, prepend compute-sanitizer to your command to run the Triton program.
**CN:** 本节推荐适用于 NVIDIA、AMD 以及通用工作流的外部调试和可视化工具。 其中反复出现的技术关键词包括 compute-sanitizer、debugging、GPUs、tool、memory、access。

## Key Concepts / 关键概念
- **EN:** Debugging Triton  **CN:** Triton 调试
- **EN:** Using Triton's Debugging Operations  **CN:** 使用 Triton 调试操作
- **EN:** Using the Interpreter  **CN:** 使用解释器
- **EN:** Limitations  **CN:** 局限
- **EN:** Using Third-party Tools  **CN:** 使用第三方工具
- **EN:** static_print and static_assert are intended for compile-time debugging.  **CN:** static_print and static_assert are intended for compile-time debugging.

## Related Files / 相关文件
- `/root/xw/triton/docs/programming-guide/chapter-3/fpsan.rst`
