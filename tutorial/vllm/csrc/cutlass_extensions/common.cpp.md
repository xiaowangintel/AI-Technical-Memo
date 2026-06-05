# common.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `csrc/cutlass_extensions/common.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements the runtime helper that converts CUDA compute capability into a compact SM version number. / [CN] 实现运行时辅助函数，把 CUDA 计算能力转换成紧凑的 SM 版本号。

## Line-by-Line Analysis / 逐行分析
### SM version query implementation / SM 版本查询实现
```cpp
#include "cutlass_extensions/common.hpp"

int32_t get_sm_version_num() {
  int32_t major_capability, minor_capability;
  cudaDeviceGetAttribute(&major_capability, cudaDevAttrComputeCapabilityMajor,
                         0);
  cudaDeviceGetAttribute(&minor_capability, cudaDevAttrComputeCapabilityMinor,
                         0);
  int32_t version_num = major_capability * 10 + minor_capability;
  return version_num;
```
**EN:** The function queries major and minor compute capability from CUDA device 0, combines them as `major * 10 + minor`, and returns the result. This compact form matches how the header-side architecture wrappers are named and how kernel dispatch code usually reasons about SM versions.
**CN:** 该函数从 CUDA 的 0 号设备读取主、次计算能力，再按 `major * 10 + minor` 拼成一个整数返回。这种紧凑表示与头文件里的架构包装器命名方式一致，也符合 kernel 分发代码常见的 SM 版本判断习惯。

## Key Concepts / 关键概念
- Runtime compute-capability detection / 运行时计算能力检测
- Compact integer encoding of SM versions / SM 版本的紧凑整数编码

## Dependencies / 依赖关系
- Implements the declaration from `cutlass_extensions/common.hpp` / 实现 `cutlass_extensions/common.hpp` 中的声明
- Uses `cudaDeviceGetAttribute` from the CUDA runtime / 使用 CUDA runtime 的 `cudaDeviceGetAttribute`
