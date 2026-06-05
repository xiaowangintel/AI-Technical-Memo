# cuda_utils_kernels.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cuda_utils_kernels.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements small runtime helpers that query GPU attributes used by kernel launch code. / [CN] 实现供内核启动代码使用的小型 GPU 运行时属性查询辅助函数。

## Line-by-Line Analysis / 逐行分析
### Cached generic device-attribute query / 带缓存的通用设备属性查询
```cpp
int64_t get_device_attribute(int64_t attribute, int64_t device_id) {
  // Return the cached value on subsequent calls
  static int value = [=]() {
    int device = static_cast<int>(device_id);
    if (device < 0) {
      CUDA_CHECK(cudaGetDevice(&device));
    }
    int value;
    CUDA_CHECK(cudaDeviceGetAttribute(
        &value, static_cast<cudaDeviceAttr>(attribute), device));
    return static_cast<int>(value);
  }();

  return value;
```
**EN:** `get_device_attribute` resolves the target device, calls `cudaDeviceGetAttribute`, and stores the first result in a function-local static variable so later calls avoid the runtime query cost.
**CN:** `get_device_attribute` 会先解析目标设备，再调用 `cudaDeviceGetAttribute`，并把首次结果缓存到函数局部静态变量中，从而避免后续重复查询的运行时开销。

### Max shared-memory selector / 最大共享内存属性选择器
```cpp
int64_t get_max_shared_memory_per_block_device_attribute(int64_t device_id) {
  int64_t attribute;
  // https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__TYPES.html
  // cudaDevAttrMaxSharedMemoryPerBlockOptin = 97 if not is_hip() else 74

#ifdef USE_ROCM
  attribute = hipDeviceAttributeMaxSharedMemoryPerBlock;
#else
  attribute = cudaDevAttrMaxSharedMemoryPerBlockOptin;
#endif

  return get_device_attribute(attribute, device_id);
}
```
**EN:** This wrapper chooses the correct backend-specific enum for “max shared memory per block” and forwards it to the generic cached query above.
**CN:** 这个包装函数选择“每个 block 最大共享内存”在不同后端对应的枚举值，再把它转交给上面的通用缓存查询函数。

## Key Concepts / 关键概念
- **EN:** The file is tiny because it exists only to keep runtime API calls out of headers.
  **CN:** 这个文件很小，因为它的目的只是把运行时 API 调用从头文件中移出去。
- **EN:** Attribute caching matters when launch configuration code is hit repeatedly in high-throughput inference paths.
  **CN:** 在高吞吐推理路径中，启动配置代码会被频繁调用，因此属性缓存是有意义的。

## Dependencies / 依赖关系
- **EN:** Depends on `cuda_utils.h` for `CUDA_CHECK` and function declarations.
  **CN:** 依赖 `cuda_utils.h` 中的 `CUDA_CHECK` 和函数声明。
- **EN:** Uses CUDA/HIP runtime APIs for device selection and attribute lookup.
  **CN:** 使用 CUDA/HIP runtime API 完成设备选择与属性查询。
