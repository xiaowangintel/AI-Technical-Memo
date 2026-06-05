# ssd.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/112_blackwell_ssd/device/ssd.hpp`  
**Purpose / 用途**: CUTLASS device wrapper for launching a Blackwell SSD kernel, handling argument packing, workspace, occupancy, and cluster-aware launch. / 用于启动 Blackwell SSD 内核的 CUTLASS 设备包装器，负责参数打包、工作区、occupancy 和 cluster 感知启动。

---

## Line-by-Line Analysis / 逐行分析

### 1. Template wrapper and public aliases (lines 45-61) / 模板包装器与公开别名（45-61 行）

```cpp
template <class Kernel_>
class SSD {
public:
  using Kernel = Kernel_;
  static int const kThreadCount = Kernel::MaxThreadsPerBlock;
  using Arguments = typename Kernel::Arguments;
  using Params = typename Kernel::Params;
```

**EN** The wrapper is fully generic: any kernel exposing the CUTLASS kernel contract can be used. For the Blackwell SSD example, this lets the higher-level example instantiate `device::SSD<Sm100...::Kernel>` without knowing the internals of the kernel pipeline.

**CN** 这个包装器是完全泛型的：任何满足 CUTLASS 内核契约的内核都能接入。在 Blackwell SSD 示例里，上层代码可以直接实例化 `device::SSD<Sm100...::Kernel>`，而不用关心底层流水线细节。

### 2. Internal state: cached packed params and one-time init flag (lines 63-79) / 内部状态：缓存参数与一次性初始化标志（63-79 行）

```cpp
private:
  Params params_;

  bool is_initialized(bool set = false) {
    static bool initialized = false;
    if (set) initialized = true;
    return initialized;
  }
```

**EN** The handle caches the packed `Params` object so the same SSD kernel can be relaunched without rebuilding arguments. The `is_initialized()` guard is used to avoid repeating the expensive runtime shared-memory attribute setup on every call.

**CN** 该句柄会缓存打包后的 `Params`，这样同一个 SSD 内核可以重复启动而无需每次重建参数。`is_initialized()` 用来避免每次调用都重复执行较昂贵的运行时共享内存属性设置。

### 3. Thin forwarding API for validation, workspace, and grid shape (lines 81-104) / 对校验、workspace 与网格形状的薄封装（81-104 行）

```cpp
static Status can_implement(Arguments const& args) {
  if (Kernel::can_implement(args)) {
    return Status::kSuccess;
  }
  else {
    return Status::kInvalid;
  }
}

static size_t get_workspace_size(Arguments const& args) {
  size_t workspace_bytes = 0;
  workspace_bytes += Kernel::get_workspace_size(args);
  return workspace_bytes;
}
```

**EN** This file intentionally keeps SSD-specific policy out of the wrapper. The real checks and packing live in the selected kernel, builder, mainloop, and epilogue. That separation is important for the Blackwell example because the same wrapper can host other SSD implementations as long as they obey the same interface.

**CN** 这里刻意不把 SSD 专属策略塞进包装器里。真正的检查与参数打包发生在具体的 kernel、builder、mainloop 和 epilogue 中。对 Blackwell 示例来说，这种分层很重要：只要接口一致，同一个包装器就能承载不同 SSD 实现。

### 4. Occupancy query with dynamic smem configuration (lines 106-146) / 带动态共享内存配置的 occupancy 查询（106-146 行）

```cpp
int smem_size = Kernel::SharedStorageSize;
if (smem_size >= (48 << 10)) {
  result = cudaFuncSetAttribute(
      device_kernel<Kernel>,
      cudaFuncAttributeMaxDynamicSharedMemorySize,
      smem_size);
}

result = cudaOccupancyMaxActiveBlocksPerMultiprocessor(
    &max_active_blocks,
    device_kernel<Kernel>,
    Kernel::MaxThreadsPerBlock,
    smem_size);
```

**EN** Blackwell SSD kernels use large shared storage because they coordinate multiple TMA/MMA/epilogue pipelines. This wrapper therefore explicitly requests a larger dynamic-smem allowance before occupancy calculation. Without that step, occupancy results would not match the real launch configuration.

**CN** Blackwell SSD 内核为了协调多个 TMA / MMA / epilogue 流水线，会使用较大的共享内存。因此这个包装器会先申请更大的动态共享内存上限，再去做 occupancy 计算；否则得到的 occupancy 结果就和真实启动配置不一致。

### 5. `initialize()`: workspace setup, argument packing, and first-launch setup (lines 148-183) / `initialize()`：workspace、参数打包与首次启动准备（148-183 行）

```cpp
Status status = Kernel::initialize_workspace(args, workspace, stream);
if (status != Status::kSuccess) {
  return status;
}

params_ = Kernel::to_underlying_arguments(args, workspace);

if (is_initialized()) return Status::kSuccess;
```

**EN** The wrapper first lets the kernel initialize any auxiliary workspace, then converts the public API into launch-ready `Params`. For the SSD example, this is the point where the Blackwell-specific builder output and scheduler parameters become concrete launch data.

**CN** 包装器先让内核初始化辅助 workspace，然后把用户层 `Arguments` 转换成可直接启动的 `Params`。对于 SSD 示例，这一步就是 Blackwell 专用 builder 输出与 scheduler 参数真正落地为启动数据的地方。

### 6. `update()`: lightweight repacking path (lines 185-196) / `update()`：轻量级重新打包路径（185-196 行）

```cpp
size_t workspace_bytes = get_workspace_size(args);
if (workspace_bytes > 0 && nullptr == workspace) {
  return Status::kErrorWorkspaceNull;
}

params_ = Kernel::to_underlying_arguments(args, workspace);
```

**EN** `update()` preserves the CUTLASS handle style: if the user changes tensor pointers or problem sizes, the wrapper can rebuild `Params` without redoing one-time runtime setup.

**CN** `update()` 保留了 CUTLASS 句柄式接口：当用户修改张量指针或问题规模时，可以重新生成 `Params`，而不用重复做一次性运行时设置。

### 7. Static launch path and cluster-aware dispatch (lines 199-233) / 静态启动路径与 cluster 感知分发（199-233 行）

```cpp
dim3 const block = Kernel::get_block_shape();
dim3 const grid = get_grid_shape(params);
int smem_size = Kernel::SharedStorageSize;

if constexpr(Kernel::ArchTag::kMinComputeCapability >= 90) {
  dim3 cluster(cute::size<0>(typename Kernel::ClusterShape{}),
               cute::size<1>(typename Kernel::ClusterShape{}),
               cute::size<2>(typename Kernel::ClusterShape{}));
  launch_result = ClusterLauncher::launch(grid, cluster, block, smem_size, stream, kernel, kernel_params);
}
else {
  device_kernel<Kernel><<<grid, block, smem_size, stream>>>(params);
}
```

**EN** This is the only place where architecture-specific launch behavior appears in the wrapper. For SM100 SSD kernels, the wrapper extracts `ClusterShape` from the kernel type and launches through CUTLASS cluster infrastructure. The wrapper itself stays math-agnostic: it does not know about SSD recurrence, TMA, or TMEM; it only knows how to launch the prepared kernel correctly.

**CN** 这是包装器中唯一明显体现架构差异的地方。对于 SM100 SSD 内核，它会从内核类型中取出 `ClusterShape`，并通过 CUTLASS 的 cluster 启动基础设施发射内核。包装器本身仍然不理解 SSD recurrence、TMA 或 TMEM；它只负责把已经准备好的内核正确启动起来。

### 8. Handle-style convenience overloads (lines 239-265) / 句柄风格的便捷重载（239-265 行）

```cpp
Status run(Arguments const& args, void* workspace = nullptr, cudaStream_t stream = nullptr) {
  Status status = initialize(args, workspace, stream);
  if (Status::kSuccess == status) {
    status = run(params_, stream);
  }
  return status;
}
```

**EN** These overloads make the SSD kernel feel like a normal CUTLASS operation object: you can launch from fresh `Arguments`, or relaunch from cached `params_`. That is useful for benchmarking the Blackwell SSD path because argument packing and kernel execution can be separated.

**CN** 这些重载让 SSD 内核像一个标准 CUTLASS 算子对象一样使用：既可以直接从 `Arguments` 启动，也可以复用缓存的 `params_` 重复启动。这对 Blackwell SSD 路径做 benchmark 很有用，因为可以把参数打包与内核执行分开。

---

## Key Concepts / 关键概念

- `Arguments` vs `Params`
  - **EN** `Arguments` is the user-facing API; `Params` is the packed kernel-facing form.
  - **CN** `Arguments` 面向用户；`Params` 是面向内核执行的打包参数。
- Device wrapper pattern
  - **EN** `SSD<Kernel_>` mirrors CUTLASS GEMM handle style: validate, initialize, update, run.
  - **CN** `SSD<Kernel_>` 复用了 CUTLASS GEMM 风格的句柄模式：校验、初始化、更新、运行。
- Dynamic shared memory and occupancy
  - **EN** The wrapper configures dynamic smem before occupancy queries or first launch.
  - **CN** 包装器会在 occupancy 查询或首次启动前设置动态共享内存属性。
- Cluster launch for modern architectures
  - **EN** For `kMinComputeCapability >= 90`, launch goes through `ClusterLauncher`, so SM90+/SM100 kernels can use cluster-aware launch metadata.
  - **CN** 对于 `kMinComputeCapability >= 90` 的内核，启动路径会走 `ClusterLauncher`，从而支持 SM90+/SM100 的 cluster 感知启动。

## Dependencies / 依赖项

- `cutlass/cutlass.h`
- `cutlass/device_kernel.h`
- `cutlass/cluster_launch.hpp` (non-RTC)
- `cutlass/trace.h` (non-RTC)
- Template contract required from `Kernel`:
  - `Arguments`, `Params`, `ArchTag`, `ClusterShape`
  - `MaxThreadsPerBlock`, `SharedStorageSize`
  - `can_implement`, `get_workspace_size`, `initialize_workspace`
  - `to_underlying_arguments`, `get_grid_shape`, `get_block_shape`
