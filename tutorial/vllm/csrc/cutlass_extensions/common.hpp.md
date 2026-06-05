# common.hpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `csrc/cutlass_extensions/common.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines shared CUTLASS/CUDA helpers for extension kernels: error checking, shared-memory queries, SM detection hooks, and architecture-gated kernel wrappers. / [CN] 为扩展 kernel 定义共享的 CUTLASS/CUDA 辅助设施：错误检查、共享内存查询、SM 检测入口，以及按架构启用的 kernel 包装器。

## Line-by-Line Analysis / 逐行分析
### CUTLASS check and device helpers / CUTLASS 检查与设备辅助
```cpp
#define CUTLASS_CHECK(status)                           \
  {                                                     \
    cutlass::Status error = status;                     \
    STD_TORCH_CHECK(error == cutlass::Status::kSuccess, \
                    cutlassGetStatusString(error));     \
  }

inline int get_cuda_max_shared_memory_per_block_opt_in(int const device) {
  int max_shared_mem_per_block_opt_in = 0;
  cudaDeviceGetAttribute(&max_shared_mem_per_block_opt_in,
                         cudaDevAttrMaxSharedMemoryPerBlockOptin, device);
  return max_shared_mem_per_block_opt_in;
}

int32_t get_sm_version_num();
```
**EN:** `CUTLASS_CHECK` converts a CUTLASS status code into a Torch-side failure using `STD_TORCH_CHECK`, so extension code can fail with a readable message. The inline helper queries the opt-in shared-memory limit for a device, and `get_sm_version_num()` is declared here for use by callers that need a compact `major * 10 + minor` SM number.
**CN:** `CUTLASS_CHECK` 通过 `STD_TORCH_CHECK` 把 CUTLASS 的状态码转换成 Torch 侧异常，便于扩展代码报出可读错误。内联函数负责查询设备可申请的 opt-in 共享内存上限，而 `get_sm_version_num()` 在此声明，供调用方获取 `major * 10 + minor` 形式的紧凑 SM 版本号。

### SM75 wrapper / SM75 包装器
```cpp
template <typename Kernel>
struct enable_sm75_to_sm80 : Kernel {
  template <typename... Args>
  CUTLASS_DEVICE static void invoke(Args&&... args) {
#if defined __CUDA_ARCH__
  #if __CUDA_ARCH__ >= 750 && __CUDA_ARCH__ < 800
    Kernel::invoke(std::forward<Args>(args)...);
  #else
    printf("This kernel only supports sm[75, 80).\n");
    asm("trap;");
  #endif
#endif
  }
};
```
**EN:** `enable_sm75_to_sm80` inherits the kernel type and exposes a static `invoke`. Device code only forwards to the wrapped kernel when the compiled architecture is in `[75, 80)`. Otherwise it prints a diagnostic and executes `trap`, preventing accidental launches on unsupported GPUs while also keeping host code free of `__CUDA_ARCH__` conditionals.
**CN:** `enable_sm75_to_sm80` 继承原 kernel 类型，并提供静态 `invoke`。只有当编译目标架构位于 `[75, 80)` 时，设备代码才会真正转发到被包装的 kernel；否则会打印诊断信息并执行 `trap`，既避免在不支持的 GPU 上误启动，也把 `__CUDA_ARCH__` 判断封装到了设备代码内部。

### Ampere/Ada wrappers / Ampere 与 Ada 包装器
```cpp
template <typename Kernel>
struct enable_sm80_to_sm89 : Kernel {
  template <typename... Args>
  CUTLASS_DEVICE static void invoke(Args&&... args) {
#if defined __CUDA_ARCH__
  #if __CUDA_ARCH__ >= 800 && __CUDA_ARCH__ < 890
    Kernel::invoke(std::forward<Args>(args)...);
  #else
    printf("This kernel only supports sm[80, 89).\n");
    asm("trap;");
  #endif
#endif
  }
};

template <typename Kernel>
struct enable_sm89_to_sm90 : Kernel {
  template <typename... Args>
  CUTLASS_DEVICE static void invoke(Args&&... args) {
#if defined __CUDA_ARCH__
  #if __CUDA_ARCH__ >= 890 && __CUDA_ARCH__ < 900
    Kernel::invoke(std::forward<Args>(args)...);
  #else
    printf("This kernel only supports sm[89, 90).\n");
    asm("trap;");
  #endif
#endif
  }
};
```
**EN:** The next two wrappers repeat the same pattern for `[80, 89)` and `[89, 90)`. vLLM keeps these architecture bands explicit because different CUTLASS kernels are only valid for certain instruction sets and memory pipelines, and selecting by wrapper helps reduce dead code in the final binary.
**CN:** 后两个包装器把同样的模式扩展到 `[80, 89)` 和 `[89, 90)` 两个区间。vLLM 明确区分这些架构带，是因为不同 CUTLASS kernel 只适用于特定指令集和内存流水线；通过包装器做选择，有助于减少最终二进制中的无效代码。

### SM90+ and SM100 range wrappers / SM90+ 与 SM100 范围包装器
```cpp
template <typename Kernel>
struct enable_sm90_or_later : Kernel {
  template <typename... Args>
  CUTLASS_DEVICE void operator()(Args&&... args) {
#if defined __CUDA_ARCH__
  #if __CUDA_ARCH__ >= 900
    Kernel::operator()(std::forward<Args>(args)...);
  #else
    printf("This kernel only supports sm >= 90.\n");
    asm("trap;");
  #endif
#endif
  }
};

template <typename Kernel>
struct enable_sm100_to_sm120 : Kernel {
  template <typename... Args>
  CUTLASS_DEVICE void operator()(Args&&... args) {
#if defined __CUDA_ARCH__
  #if (__CUDA_ARCH__ >= 1000 && __CUDA_ARCH__ < 1200)
    Kernel::operator()(std::forward<Args>(args)...);
  #else
    printf("This kernel only supports sm[100, 120).\n");
    asm("trap;");
  #endif
#endif
  }
};
```
**EN:** `enable_sm90_or_later` switches from `invoke` to callable-object syntax (`operator()`), matching newer CUTLASS kernel objects. `enable_sm100_to_sm120` applies the same idea to the Blackwell-era range `[1000, 1200)`, again trapping immediately if a mismatched binary path is used.
**CN:** `enable_sm90_or_later` 改为通过可调用对象语法 `operator()` 转发，适配较新的 CUTLASS kernel 对象。`enable_sm100_to_sm120` 则把同样机制应用到 Blackwell 时代的 `[1000, 1200)` 范围，一旦运行到了不匹配的二进制路径，同样会立刻 `trap`。

### SM120 exact vs family wrappers / SM120 精确版与家族版包装器
```cpp
template <typename Kernel>
struct enable_sm120_only : Kernel {
  template <typename... Args>
  CUTLASS_DEVICE void operator()(Args&&... args) {
#if defined __CUDA_ARCH__
  #if __CUDA_ARCH__ == 1200
    Kernel::operator()(std::forward<Args>(args)...);
  #else
    printf("This kernel only supports sm120a.\n");
    asm("trap;");
  #endif
#endif
  }
};

// SM12x family includes SM120 (RTX 5090) and SM121 (DGX Spark GB10)
template <typename Kernel>
struct enable_sm120_family : Kernel {
  template <typename... Args>
  CUTLASS_DEVICE void operator()(Args&&... args) {
#if defined __CUDA_ARCH__
  #if (__CUDA_ARCH__ >= 1200 && __CUDA_ARCH__ < 1300)
    Kernel::operator()(std::forward<Args>(args)...);
  #else
    printf("This kernel only supports sm120f.\n");
    asm("trap;");
  #endif
#endif
  }
};
```
**EN:** The last two wrappers distinguish an exact `sm120a` path from a broader `sm12x` family path. That separation matters when one kernel depends on a very specific ISA variant while another is valid for the wider 12.x family such as SM120 and SM121.
**CN:** 最后两个包装器把精确的 `sm120a` 路径与更宽泛的 `sm12x` 家族路径区分开来。当某个 kernel 依赖特定 ISA 变体，而另一个 kernel 又适用于更广的 12.x 家族（如 SM120、SM121）时，这种区分就非常重要。

## Key Concepts / 关键概念
- Torch-visible error propagation via `CUTLASS_CHECK` / 通过 `CUTLASS_CHECK` 将 CUTLASS 错误提升为 Torch 可见异常
- Runtime querying of opt-in shared memory and SM version / 在运行时查询 opt-in 共享内存和 SM 版本
- Architecture-gated wrappers that trap invalid device launches / 对非法设备架构启动直接 `trap` 的架构门控包装器

## Dependencies / 依赖关系
- `cutlass/cutlass.h` provides `cutlass::Status`, device qualifiers, and status-string helpers / `cutlass/cutlass.h` 提供 `cutlass::Status`、设备修饰符和状态字符串工具
- `cuda_runtime.h` supplies `cudaDeviceGetAttribute` and attribute enums / `cuda_runtime.h` 提供 `cudaDeviceGetAttribute` 及相关属性枚举
- `torch/headeronly/util/shim_utils.h` provides `STD_TORCH_CHECK` for Torch-facing validation / `torch/headeronly/util/shim_utils.h` 提供 Torch 侧校验宏 `STD_TORCH_CHECK`
