# utils.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/44_multi_gemm_ir_and_codegen/utils.h`
**Purpose / 用途**: Provides lightweight timing, memory-management, and result-checking helpers for the multi-GEMM example. / 为多 GEMM 示例提供轻量级计时、内存管理与结果校验辅助工具。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1–34 — License Header / 许可证头

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/

#pragma once
#define TI(tag) \
    cudaEvent_t _event_start_ ##tag; \
```
**EN**: Standard NVIDIA BSD-3-Clause copyright block.

**CN**: 标准 NVIDIA BSD-3 条款版权声明。

---

### Line 35 — Include Guard / 头文件保护

```cpp
    cudaEvent_t _event_end_ ##tag; \
```
**EN**: Prevents multiple inclusion. No additional standard headers are included; the file relies on types (`cudaEvent_t`, `cudaMemcpy`, etc.) already available through CUDA runtime headers included by the calling translation unit.

**CN**: 防止重复包含。文件未包含其他标准头文件；依赖调用翻译单元已通过 CUDA 运行时头文件引入的类型（`cudaEvent_t`、`cudaMemcpy` 等）。

---

### Lines 36–44 — `TI` Timing-Start Macro / `TI` 计时开始宏

```cpp
    float _event_time_ ##tag; \
    cudaEventCreate(& _event_start_ ##tag); \
    cudaEventCreate(& _event_end_ ##tag); \
    cudaEventRecord(_event_start_ ##tag);

#define TO(tag, str, times) \
    cudaEventRecord(_event_end_ ##tag); \
    cudaEventSynchronize(_event_end_ ##tag); \
    cudaEventElapsedTime(&_event_time_ ##tag, _event_start_ ##tag, _event_end_ ##tag); \
```
**EN**: `TI(tag)` is a statement-level macro that declares three local variables (`_event_start_tag`, `_event_end_tag`, `_event_time_tag`) and records a start event on the default CUDA stream. The `##` token-paste operator appends the literal `tag` to each identifier name, so multiple `TI`/`TO` pairs in the same scope do not clash. The macro must be placed inside a function body; it cannot appear at file scope.

**CN**: `TI(tag)` 是一个语句级宏，声明三个局部变量（`_event_start_tag`、`_event_end_tag`、`_event_time_tag`），并在默认 CUDA 流上记录起始事件。`##` 标记粘贴运算符将字面量 `tag` 追加到每个标识符名称，因此同一作用域内多对 `TI`/`TO` 不会发生命名冲突。该宏必须置于函数体内，不能出现在文件作用域。

---

### Lines 45–53 — `TO` Timing-End Macro / `TO` 计时结束宏

```cpp
    float _event_time_once_ ##tag = _event_time_ ##tag / times; \
    printf("%20s:\t %10.3fus\t", str, _event_time_once_ ##tag * 1000); \
    cudaDeviceSynchronize(); \
    printf("%20s string: %s\n",str, cudaGetErrorString(cudaGetLastError()));

template<typename T>
struct memory_unit{
    T* host_ptr;
    T* device_ptr;
```
**EN**: `TO(tag, str, times)` records the end event, synchronises the CPU to wait for it (`cudaEventSynchronize`), then computes the elapsed time in milliseconds via `cudaEventElapsedTime`. Dividing by `times` gives an *average* per-launch time. The result is printed in microseconds (`ms * 1000`). A final `cudaDeviceSynchronize` + `cudaGetLastError` check catches any asynchronous kernel errors launched between `TI` and `TO`. The `str` argument labels the output line.

**CN**: `TO(tag, str, times)` 记录结束事件，通过 `cudaEventSynchronize` 同步 CPU 等待其完成，然后用 `cudaEventElapsedTime` 以毫秒为单位计算经过的时间。除以 `times` 得到*平均*每次启动耗时。结果以微秒（`ms * 1000`）打印。最后的 `cudaDeviceSynchronize` + `cudaGetLastError` 检查捕获在 `TI` 和 `TO` 之间启动的所有异步 kernel 错误。`str` 参数用于标注输出行。

---

### Lines 54–80 — `memory_unit<T>` Struct / `memory_unit<T>` 结构体

```cpp
    int size_bytes;
    int elements;
    void h2d(){
        cudaMemcpy(device_ptr, host_ptr, size_bytes, cudaMemcpyHostToDevice);
    }
    void d2h(){
        cudaMemcpy(host_ptr, device_ptr, size_bytes, cudaMemcpyDeviceToHost);
    }
    void free_all(){
        free(host_ptr);
        cudaFree(device_ptr);
    }
    memory_unit(int elements_): size_bytes(elements_ * sizeof(T)), elements(elements_){
        host_ptr = (T*) malloc(elements_ * sizeof(T));
        cudaMalloc((void**)&device_ptr, elements_ * sizeof(T));
    }
    void init(int abs_range = 1){
        for(int i = 0; i < elements; i++){
            host_ptr[i] = T(rand() % 100 / float(100)  * 2 * abs_range - abs_range);
        }
        h2d();
    }
};

template<typename T>
int check_result(T * a, T * b, int N){
    int cnt = 0;
```
**EN**: A simple RAII-like struct for paired CPU/GPU memory. The constructor allocates `elements_ * sizeof(T)` bytes on both the host (via `malloc`) and the device (via `cudaMalloc`), storing both pointers and the byte count. `h2d()` / `d2h()` are thin wrappers around synchronous `cudaMemcpy`. `init(abs_range)` fills the host array with uniform random values in `[-abs_range, abs_range)`, then copies to device — a one-call convenience for test data generation. `free_all()` releases both allocations. Note: there is no destructor, so `free_all()` must be called explicitly; this is consistent with the example's C-style resource management.

**CN**: 一个简单的类 RAII 结构体，用于管理 CPU/GPU 配对内存。构造函数在宿主端（通过 `malloc`）和设备端（通过 `cudaMalloc`）各分配 `elements_ * sizeof(T)` 字节，并存储两个指针和字节数。`h2d()` / `d2h()` 是对同步 `cudaMemcpy` 的轻量封装。`init(abs_range)` 用 `[-abs_range, abs_range)` 范围内的均匀随机值填充宿主数组，然后拷贝到设备端——是测试数据生成的一步到位便捷接口。`free_all()` 释放两端的内存分配。注意：该结构体没有析构函数，因此必须显式调用 `free_all()`；这与示例中 C 风格的资源管理方式一致。

---

### Lines 81–94 — `check_result<T>` Function / `check_result<T>` 函数

```cpp
    for(int i = 0; i < N; i ++){
        float std = float(a[i]);
        float my = float(b[i]);

        if(abs(std - my) / abs(std) > 1e-2)
        {
            // printf("my: %f , std: %f\n", my, std);
            cnt++;
        }

    }
    printf("total err: %d / %d\n", cnt, N);
    return cnt;
}
```
**EN**: A host-side numerical correctness checker. It compares two arrays `a` (reference) and `b` (test) element-by-element, converting each element to `float` before comparison. The relative error threshold is `1e-2` (1%), appropriate for FP16 output where round-trip accuracy is limited. It prints a summary line `"total err: X / N"` and returns the count of failing elements. The commented-out `printf` can be re-enabled for per-element debugging. This function is designed to be called after `d2h()` has brought GPU results back to host memory.

**CN**: 宿主端数值正确性检查函数。它逐元素比较参考数组 `a` 和测试数组 `b`，在比较前将每个元素转换为 `float`。相对误差阈值为 `1e-2`（1%），适用于精度受限的 FP16 输出。函数打印摘要行 `"total err: X / N"` 并返回失败元素的数量。被注释掉的 `printf` 可重新启用以进行逐元素调试。该函数设计为在 `d2h()` 将 GPU 结果传回宿主内存后调用。

---
## Key Concepts / 关键概念
- CUDA event macros provide low-overhead timing around repeated kernel launches. / CUDA event 宏为重复 kernel 启动提供低开销计时。
- A small host/device memory wrapper reduces boilerplate in standalone benchmark code. / 小型主机/设备内存封装减少了独立基准代码中的样板逻辑。
- Result checking converts values to float before comparison to give a simple tolerance-based validation path. / 结果检查会先把值转换为 float，再执行基于容差的简单验证。
## Dependencies / 依赖项
- `cuda_runtime_api.h` — Provides `cudaEvent*`, `cudaMalloc`, and `cudaMemcpy` APIs used by the helpers / 提供这些辅助工具使用的 `cudaEvent*`、`cudaMalloc` 与 `cudaMemcpy` API
- `cstdlib` — Supplies `malloc`, `free`, and `rand` expected by the memory wrapper / 提供内存封装所需的 `malloc`、`free` 与 `rand`
- `cstdio` — Used for `printf`-based timing and error reporting / 用于基于 `printf` 的计时与错误输出
