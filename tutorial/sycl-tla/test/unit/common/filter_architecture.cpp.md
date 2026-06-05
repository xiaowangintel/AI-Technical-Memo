# filter_architecture.cpp — Code Analysis / 代码分析

## Source / 来源
- `test/unit/common/filter_architecture.cpp`
- **EN:** Backend-aware test bootstrap code. It detects the active GPU architecture, converts that into GoogleTest exclusion filters, and reads an environment override for problem-count selection.
- **CN:** 这是面向后端的测试启动代码。它检测当前 GPU 架构，把结果转换成 GoogleTest 的排除过滤条件，并读取控制问题规模数量的环境变量。

## Line-by-Line Analysis / 逐行分析

### Lines 32-40 — backend-specific includes / 后端相关头文件
```cpp
#if !defined(CUTLASS_ENABLE_SYCL)
#include <cuda_runtime_api.h>
#endif

#include "cutlass_unit_test.h"

#if defined(CUTLASS_ENABLE_SYCL)
#include <sycl/sycl.hpp>
#endif
```
**EN:** The file is compiled for either CUDA or SYCL. CUDA runtime is needed for device-property queries in native CUDA builds, while `sycl/sycl.hpp` is needed for architecture queries in SYCL mode. `cutlass_unit_test.h` provides the declarations implemented here.

**CN:** 该文件既支持 CUDA，也支持 SYCL。原生 CUDA 构建需要 CUDA Runtime 来查询设备属性；SYCL 模式则需要 `sycl/sycl.hpp` 获取设备架构。`cutlass_unit_test.h` 提供了这里实现的函数声明。

### Lines 46-63 — `GetCudaDevice()` / 获取 CUDA 设备属性
```cpp
cudaDeviceProp GetCudaDevice() {

  cudaError_t err;

  int cudaDeviceId;
  err = cudaGetDevice(&cudaDeviceId);
  if (cudaSuccess != err) {
    std::cerr << "*** Error: Could not detect active GPU device ID"
              << " [" << cudaGetErrorString(err) << "]" << std::endl;
    exit(1);
  }

  cudaDeviceProp deviceProperties;
  err = cudaGetDeviceProperties(&deviceProperties, cudaDeviceId);

  return deviceProperties;
}
```
**EN:** This CUDA-only helper asks the runtime for the currently active device ID and then fetches its `cudaDeviceProp` structure. Failure to discover the active device is treated as fatal and terminates the process, which is reasonable for a test bootstrap path.

**CN:** 这个仅用于 CUDA 的辅助函数先向运行时查询当前活跃设备 ID，再读取对应的 `cudaDeviceProp` 结构。若连当前设备都无法获取，就直接终止进程；对于测试启动阶段来说，这样的失败被视为致命错误。

### Lines 65-83 — stream printer for device properties / 设备属性的输出运算符
```cpp
std::ostream &operator<<(std::ostream &out, cudaDeviceProp const &deviceProperties) {

  int deviceMajorMinor = deviceProperties.major * 10 + deviceProperties.minor;
  if (deviceMajorMinor) {
    int32_t clock_MHz;
    int32_t clock_KHz;
    cudaDeviceGetAttribute(&clock_KHz, cudaDevAttrClockRate, 0);
    clock_MHz = clock_KHz / 1000;
    out << "GPU(compute_"
      << deviceMajorMinor << ", "
      << deviceProperties.multiProcessorCount << " SMs @ " << clock_MHz << " MHz)";
  }
  else {
    out << "No CUDA device.";
  }

  return out;
}
```
**EN:** The overload formats a concise device description: compute capability, SM count, and clock frequency. The `deviceMajorMinor` integer is the common CUTLASS shorthand (`80`, `90`, and so on). A zero capability falls back to the string `No CUDA device.`.

**CN:** 这个输出运算符把设备信息格式化成紧凑字符串：计算能力、SM 数量和时钟频率。`deviceMajorMinor` 是 CUTLASS 常用的简写形式，例如 `80`、`90`。如果组合值为 0，则输出 `No CUDA device.`。

### Lines 89-127 — `FilterArchitecture()` SYCL path / `FilterArchitecture()` 的 SYCL 分支
```cpp
void FilterArchitecture() {
  // Default flags can be overwritten by --gtest_filter from commandline

  int const kMaxDevice = 999;

#if defined(CUTLASS_ENABLE_SYCL)
  using namespace sycl::ext::oneapi::experimental;

  std::map<architecture, int> arch_map {
    {architecture::nvidia_gpu_sm_50, 50},
    {architecture::nvidia_gpu_sm_52, 52},
    {architecture::nvidia_gpu_sm_53, 53},
    {architecture::nvidia_gpu_sm_60, 60},
    {architecture::nvidia_gpu_sm_61, 61},
    {architecture::nvidia_gpu_sm_62, 62},
    {architecture::nvidia_gpu_sm_70, 70},
    {architecture::nvidia_gpu_sm_72, 72},
    {architecture::nvidia_gpu_sm_75, 75},
    {architecture::nvidia_gpu_sm_80, 80},
    {architecture::nvidia_gpu_sm_86, 86},
    {architecture::nvidia_gpu_sm_89, 89},
    {architecture::nvidia_gpu_sm_90, 90},
    {architecture::nvidia_gpu_sm_90a, 90},
    {architecture::intel_gpu_pvc, 0},
    {architecture::intel_gpu_bmg_g21, 1},
    {architecture::intel_gpu_bmg_g31, 1}
  };
  auto device_architecture =
        compat::get_default_queue().get_device().get_info<info::device::architecture>();
  if (device_architecture == architecture::unknown) {
    throw std::runtime_error("Encountered Unknown architecture.");
  }

  if(auto search_result = arch_map.find(device_architecture); search_result == arch_map.end()) {
    throw std::runtime_error("Detected Architecture is not supported.");
  }

  const int deviceMajorMinor = arch_map[device_architecture];
```
**EN:** In SYCL mode the code queries the default queue's device architecture using the OneAPI experimental architecture enum. A lookup table normalizes NVIDIA SM architectures and Intel Xe generations into the same integer space used by the later filter table. Unknown or unmapped architectures trigger exceptions instead of silently running the wrong tests.

**CN:** 在 SYCL 模式下，代码通过默认队列的设备对象读取 OneAPI experimental architecture 枚举。随后用查找表把 NVIDIA 的 SM 架构和 Intel Xe 代际统一映射到后续过滤逻辑使用的整数空间。若架构未知或未映射，函数会抛出异常，而不是静默执行错误的测试集合。

### Lines 129-147 — `FilterArchitecture()` CUDA path / `FilterArchitecture()` 的 CUDA 分支
```cpp
#else
  cudaError_t err;

  int cudaDeviceId;
  err = cudaGetDevice(&cudaDeviceId);
  if (cudaSuccess != err) {
    std::cerr << "*** Error: Could not detect active GPU device ID"
              << " [" << cudaGetErrorString(err) << "]" << std::endl;
    exit(1);
  }
  cudaDeviceProp deviceProperties;
  err = cudaGetDeviceProperties(&deviceProperties, cudaDeviceId);
  if (cudaSuccess != err) {
    std::cerr << "*** Error: Could not get device properties for GPU " << cudaDeviceId << " ["
              << cudaGetErrorString(err) << "]" << std::endl;
    exit(1);
  }

  int deviceMajorMinor = deviceProperties.major * 10 + deviceProperties.minor;
#endif
```
**EN:** The CUDA branch uses the active device's `major.minor` pair to build the same capability number used by CUTLASS test naming. Two error checks guard the bootstrap sequence: one for the device ID, one for the property query.

**CN:** CUDA 分支使用当前设备的 `major.minor` 组合出与 CUTLASS 测试命名一致的能力编号。启动流程里有两次关键错误检查：一次检查设备 ID 获取是否成功，一次检查设备属性读取是否成功。

### Lines 149-175 — static filter table / 静态过滤表
```cpp
  struct {

    /// Unit test filter string
    char const *filter;

    /// Minimum compute capability for the kernels in the named test
    int min_compute_capability;

    /// Maximum compute capability for which the kernels are enabled 
    int max_compute_capability;
  } 
  test_filters[] = {
    { "SM50*",                      50, kMaxDevice},
    { "SM60*",                      60, kMaxDevice},
    { "SM61*",                      61, kMaxDevice},
    { "SM70*",                      70, 75},
    { "SM75*",                      75, kMaxDevice},
    { "SM80*",                      80, kMaxDevice},
    { "SM89*",                      89, 89},
    { "SM90*",                      90, 90},
    { "SM100*",                    100, 100},
    { "XE_*",                        0, 1},
    { "XE2_*",                       1, 1},
    { "*sm100_*",                  100, 100},
    { 0, 0, false }
  };
```
**EN:** This anonymous struct table maps test-name patterns to the compute-capability range in which those tests should run. Patterns such as `SM80*` or `XE_*` correspond to naming conventions used elsewhere in the test suite. The last entry is a sentinel: its null `filter` marks the end of the array for the upcoming loop.

**CN:** 这个匿名结构数组把测试名称模式映射到可运行的计算能力范围。例如 `SM80*`、`XE_*` 等模式，对应测试套件中使用的命名约定。最后一项是哨兵元素：其 `filter` 为空指针，用于通知后面的循环到达数组末尾。

### Lines 178-191 — build a negative GoogleTest filter / 构造 GoogleTest 的负向过滤字符串
```cpp
  std::stringstream ss;
  ss << "-";
  for (int i = 0, j = 0; test_filters[i].filter; ++i) {

    if (deviceMajorMinor < test_filters[i].min_compute_capability ||
        deviceMajorMinor > test_filters[i].max_compute_capability) {

      ss << (j++ ? ":" : "") << test_filters[i].filter;
    }
  }

  ::testing::GTEST_FLAG(filter) = ss.str();
}
```
**EN:** Rather than listing what should run, the function builds a negative filter beginning with `-` and appends every test pattern that does *not* match the active device range. GoogleTest interprets colon-separated entries after the minus sign as excluded tests. This is why the function must execute before the test suite starts.

**CN:** 这里并不是列出“应该运行什么”，而是先构造一个以 `-` 开头的负向过滤字符串，把所有**不适合当前设备范围**的测试模式逐个追加进去。GoogleTest 会把减号后的冒号分隔项解释为“排除的测试”。因此该函数必须在测试真正开始前执行。

### Lines 195-202 — environment-controlled problem count / 从环境变量读取问题规模数量
```cpp
int CutlassUnitTestProblemCount() {
    if(const char* problem_count = std::getenv("CUTLASS_UNIT_TEST_PROBLEM_COUNT")) {

        return std::stoi(problem_count);
    } 

    return 0;
}
```
**EN:** This helper gives the test suite a simple runtime override. If the environment variable is present, it is parsed as an integer with `std::stoi`; otherwise the function returns `0`, which the surrounding tests can interpret as the default behavior.

**CN:** 这个辅助函数为测试套件提供了一个简单的运行时开关。如果环境变量存在，就用 `std::stoi` 把它解析为整数；否则返回 `0`，由外围测试把它解释为默认行为。

## Key Concepts / 关键概念
- **Architecture normalization / 架构归一化:** CUDA and SYCL device identifiers are converted into one comparable integer capability scale / CUDA 与 SYCL 的设备标识最终都被归一化成统一的整数能力等级。
- **Negative test filtering / 负向测试过滤:** The code excludes unsupported test-name patterns instead of enumerating only supported ones / 代码通过排除不支持的测试模式来完成筛选，而不是只枚举支持项。
- **Fail-fast bootstrap / 快速失败启动:** Device-detection errors are surfaced immediately via `exit` or exceptions / 设备检测出错时会立刻通过 `exit` 或异常暴露问题。
- **Runtime tunability / 运行时可调节性:** `CUTLASS_UNIT_TEST_PROBLEM_COUNT` lets test volume be influenced without recompilation / `CUTLASS_UNIT_TEST_PROBLEM_COUNT` 允许在不重新编译的情况下调整测试规模。

## Dependencies / 依赖关系
- **`"cutlass_unit_test.h"` / 本地头文件：** provides the declarations for `GetCudaDevice`, `FilterArchitecture`, and `CutlassUnitTestProblemCount` / 提供 `GetCudaDevice`、`FilterArchitecture`、`CutlassUnitTestProblemCount` 的声明。
- **CUDA Runtime (`<cuda_runtime_api.h>`) / CUDA 运行时：** used for active-device detection, property queries, and attribute lookup in CUDA builds / 在 CUDA 构建中用于当前设备检测、属性查询以及时钟属性读取。
- **SYCL (`<sycl/sycl.hpp>`) / SYCL：** used in SYCL builds to query `info::device::architecture` and distinguish Intel/NVIDIA backends / 在 SYCL 构建中用于查询 `info::device::architecture` 并区分 Intel/NVIDIA 后端。
- **GoogleTest / GoogleTest：** `::testing::GTEST_FLAG(filter)` is the integration point that applies the generated exclusion string / `::testing::GTEST_FLAG(filter)` 是应用生成过滤字符串的关键接口。
- **CUTLASS compat queue / CUTLASS 兼容队列：** the SYCL branch depends on `compat::get_default_queue()` to locate the device actually used by tests / SYCL 分支依赖 `compat::get_default_queue()` 获取测试真正使用的设备。
- **Standard library / 标准库：** uses `std::map`, `std::stringstream`, `std::getenv`, `std::stoi`, and `std::runtime_error` for mapping, string assembly, environment access, and error handling / 通过 `std::map`、`std::stringstream`、`std::getenv`、`std::stoi`、`std::runtime_error` 完成映射、字符串拼接、环境读取与错误处理。
