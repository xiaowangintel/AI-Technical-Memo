# cumem_allocator.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `csrc/cumem_allocator.cpp`
- **Repository**: `vllm-project/vllm`
- **Purpose**: **EN:** Implements a Python-coordinated CUDAPluggableAllocator backed by cuMem* virtual-memory APIs, with separate CUDA and ROCm handle-management paths. **CN:** 实现一个由 Python 协调、基于 cuMem* 虚拟内存 API 的 CUDAPluggableAllocator，并分别处理 CUDA 与 ROCm 的句柄管理差异。

## Line-by-Line Analysis / 逐行分析

### ROCm chunk sizing and Python ABI selection / ROCm 分块策略与 Python ABI 选择
```cpp
#ifndef USE_ROCM
static const char* PYARGS_PARSE = "KKKK";
#else
  #include <cstdlib>
  #include <cerrno>
  #include <climits>

// Default chunk size 256MB for ROCm. Can be overridden at runtime by the
// environment variable VLLM_ROCM_SLEEP_MEM_CHUNK_SIZE, specified in megabytes
// (MB). The env value is parsed with strtoull as an integer number of MB
// (decimal or 0x hex). The parsed MB value is converted to bytes. If
// parsing fails, the value is 0, or the multiplication would overflow,
// the default (256MB) is used.
static const unsigned long long DEFAULT_MEMCREATE_CHUNK_SIZE =
    (256ULL * 1024ULL * 1024ULL);

static unsigned long long get_memcreate_chunk_size() {
  const char* env = getenv("VLLM_ROCM_SLEEP_MEM_CHUNK_SIZE");
  if (!env) return DEFAULT_MEMCREATE_CHUNK_SIZE;
  char* endptr = nullptr;
  errno = 0;
  unsigned long long val_mb = strtoull(env, &endptr, 0);
  if (endptr == env || errno != 0) {
    // parsing failed, fallback to default
    return DEFAULT_MEMCREATE_CHUNK_SIZE;
  }
  if (val_mb == 0) return DEFAULT_MEMCREATE_CHUNK_SIZE;

  const unsigned long long MB = 1024ULL * 1024ULL;
  // guard against overflow when converting MB -> bytes
  if (val_mb > (ULLONG_MAX / MB)) {
    return DEFAULT_MEMCREATE_CHUNK_SIZE;
  }
  return val_mb * MB;
}

static inline unsigned long long my_min(unsigned long long a,
                                        unsigned long long b) {
  return a < b ? a : b;
}

static const char* PYARGS_PARSE = "KKKO";
#endif
```
**EN:** The ROCm path adds a configurable chunk size (default 256MB, overrideable by VLLM_ROCM_SLEEP_MEM_CHUNK_SIZE). Large virtual allocations are later split into multiple cuMemCreate chunks, and the Python argument parser format changes because the handle metadata becomes a Python object rather than a single integer.
**CN:** ROCm 路径增加了可配置的分块大小（默认 256MB，可由 VLLM_ROCM_SLEEP_MEM_CHUNK_SIZE 覆盖）。之后的大块虚拟分配会被拆成多个 cuMemCreate 分块，而 Python 参数解析格式也随之变化，因为句柄元数据不再是单个整数，而是 Python 对象。

### Global error state and context bootstrap / 全局错误状态与上下文建立
```cpp
char error_msg[10240];  // 10KB buffer to store error messages
CUresult no_error = CUresult(0);
CUresult error_code = no_error;  // store error code

#define CUDA_CHECK(condition)                                           \
  do {                                                                  \
    CUresult error = condition;                                         \
    if (error != 0) {                                                   \
      error_code = error;                                               \
      char* error_string;                                               \
      cuGetErrorString(error, (const char**)&error_string);             \
      snprintf(error_msg, sizeof(error_msg), "CUDA Error: %s at %s:%d", \
               error_string, __FILE__, __LINE__);                       \
      std::cerr << error_msg << std::endl;                              \
    }                                                                   \
  } while (0)

// Global references to Python callables
// NOTE: this is borrowed reference, so we don't need to DECREF them.
// This brings the limitation that the allocator needs to be singleton.
static PyObject* g_python_malloc_callback = nullptr;
static PyObject* g_python_free_callback = nullptr;

// ---------------------------------------------------------------------------
// Helper functions:

void ensure_context(unsigned long long device) {
  CUcontext pctx;
  CUDA_CHECK(cuCtxGetCurrent(&pctx));
  if (!pctx) {
    // Ensure device context.
    CUDA_CHECK(cuDevicePrimaryCtxRetain(&pctx, device));
    CUDA_CHECK(cuCtxSetCurrent(pctx));
  }
}
```
**EN:** The file keeps a shared error buffer and result code for reporting failures back into Python. ensure_context lazily attaches the primary device context so VMM calls always execute against a valid driver context.
**CN:** 该文件维护一个共享错误缓冲区和结果码，用于把底层失败信息回传给 Python。ensure_context 会按需绑定主设备上下文，保证后续 VMM 调用总是在有效的 driver context 中执行。

### Virtual allocation create-and-map helper / 虚拟分配的创建与映射辅助函数
```cpp
void create_and_map(unsigned long long device, ssize_t size, CUdeviceptr d_mem,
#ifndef USE_ROCM
                    CUmemGenericAllocationHandle* p_memHandle) {
#else
                    CUmemGenericAllocationHandle** p_memHandle,
                    unsigned long long* chunk_sizes, size_t num_chunks) {
#endif
  ensure_context(device);
  // Define memory allocation properties
  CUmemAllocationProp prop = {};
  prop.type = CU_MEM_ALLOCATION_TYPE_PINNED;
  prop.location.type = CU_MEM_LOCATION_TYPE_DEVICE;
  prop.location.id = device;
  prop.allocFlags.compressionType = CU_MEM_ALLOCATION_COMP_NONE;

#ifndef USE_ROCM
  int flag = 0;
  CUresult rdma_result = cuDeviceGetAttribute(
      &flag, CU_DEVICE_ATTRIBUTE_GPU_DIRECT_RDMA_WITH_CUDA_VMM_SUPPORTED,
      device);
  if (rdma_result == CUDA_SUCCESS &&
      flag) {  // support GPUDirect RDMA if possible
    prop.allocFlags.gpuDirectRDMACapable = 1;
  }
  int fab_flag = 0;
  CUresult fab_result = cuDeviceGetAttribute(
      &fab_flag, CU_DEVICE_ATTRIBUTE_HANDLE_TYPE_FABRIC_SUPPORTED, device);
  if (fab_result == CUDA_SUCCESS &&
      fab_flag) {  // support fabric handle if possible
    prop.requestedHandleTypes = CU_MEM_HANDLE_TYPE_FABRIC;
  }
#endif

#ifndef USE_ROCM
  // Allocate memory using cuMemCreate
  CUresult ret = (CUresult)cuMemCreate(p_memHandle, size, &prop, 0);
  if (ret) {
    if (fab_flag &&
        (ret == CUDA_ERROR_NOT_PERMITTED || ret == CUDA_ERROR_NOT_SUPPORTED)) {
      // Fabric allocation may fail without multi-node nvlink,
      // fallback to POSIX file descriptor
      prop.requestedHandleTypes = CU_MEM_HANDLE_TYPE_POSIX_FILE_DESCRIPTOR;
      CUDA_CHECK(cuMemCreate(p_memHandle, size, &prop, 0));
    } else {
      CUDA_CHECK(ret);
    }
  }
  if (error_code != 0) {
    return;
  }
  CUDA_CHECK(cuMemMap(d_mem, size, 0, *p_memHandle, 0));
  if (error_code != 0) {
    return;
  }
#else
  for (auto i = 0; i < num_chunks; ++i) {
    CUDA_CHECK(cuMemCreate(p_memHandle[i], chunk_sizes[i], &prop, 0));
    if (error_code != 0) {
      // Clean up previously created handles
      for (auto j = 0; j < i; ++j) {
        cuMemRelease(*(p_memHandle[j]));
      }
      return;
    }
  }
  unsigned long long allocated_size = 0;
  for (auto i = 0; i < num_chunks; ++i) {
    void* map_addr = (void*)((uintptr_t)d_mem + allocated_size);
    CUDA_CHECK(cuMemMap(map_addr, chunk_sizes[i], 0, *(p_memHandle[i]), 0));
    if (error_code != 0) {
      // unmap previously mapped chunks
      unsigned long long unmapped_size = 0;
      for (auto j = 0; j < i; ++j) {
        void* unmap_addr = (void*)((uintptr_t)d_mem + unmapped_size);
        cuMemUnmap(unmap_addr, chunk_sizes[j]);
        unmapped_size += chunk_sizes[j];
      }
      // release all created handles
      for (auto j = 0; j < num_chunks; ++j) {
        cuMemRelease(*(p_memHandle[j]));
      }
      return;
    }
    allocated_size += chunk_sizes[i];
  }
#endif

  CUmemAccessDesc accessDesc = {};
  accessDesc.location.type = CU_MEM_LOCATION_TYPE_DEVICE;
  accessDesc.location.id = device;
  accessDesc.flags = CU_MEM_ACCESS_FLAGS_PROT_READWRITE;

  CUDA_CHECK(cuMemSetAccess(d_mem, size, &accessDesc, 1));
  if (error_code != 0) {
    return;
  }
  // std::cout << "create_and_map: device=" << device << ", size=" << size << ",
  // d_mem=" << d_mem << ", p_memHandle=" << p_memHandle << std::endl;
}
```
**EN:** create_and_map configures CUmemAllocationProp, optionally enables GPUDirect RDMA or fabric handles on NVIDIA, calls cuMemCreate, maps the returned physical allocation into a reserved virtual address range, and finally enables read/write access. The ROCm branch loops over multiple chunks and performs careful rollback on partial failure.
**CN:** create_and_map 会配置 CUmemAllocationProp，在 NVIDIA 上按能力选择 GPUDirect RDMA 或 fabric handle，随后调用 cuMemCreate，把得到的物理分配映射到预留的虚拟地址区间，并最终开启读写权限。ROCm 分支会遍历多个分块，并在部分失败时做完整回滚。

### Unmap/release and Python tuple marshaling helpers / 解除映射、释放以及 Python 元组封装
```cpp
void unmap_and_release(unsigned long long device, ssize_t size,
                       CUdeviceptr d_mem,
#ifndef USE_ROCM
                       CUmemGenericAllocationHandle* p_memHandle) {
#else
                       CUmemGenericAllocationHandle** p_memHandle,
                       unsigned long long* chunk_sizes, size_t num_chunks) {
#endif
  // std::cout << "unmap_and_release: device=" << device << ", size=" << size <<
  // ", d_mem=" << d_mem << ", p_memHandle=" << p_memHandle << std::endl;
  ensure_context(device);
#ifndef USE_ROCM
  CUDA_CHECK(cuMemUnmap(d_mem, size));
  if (error_code != 0) {
    return;
  }
  CUDA_CHECK(cuMemRelease(*p_memHandle));
  if (error_code != 0) {
    return;
  }
#else
  unsigned long long allocated_size = 0;
  CUresult first_error = no_error;

  for (auto i = 0; i < num_chunks; ++i) {
    void* map_addr = (void*)((uintptr_t)d_mem + allocated_size);
    CUresult status = cuMemUnmap(map_addr, chunk_sizes[i]);
    if (status != no_error && first_error == no_error) {
      first_error = status;
    }
    allocated_size += chunk_sizes[i];
  }

  for (auto i = 0; i < num_chunks; ++i) {
    CUresult status = cuMemRelease(*(p_memHandle[i]));
    if (status != no_error && first_error == no_error) {
      first_error = status;
    }
  }

  if (first_error != no_error) {
    CUDA_CHECK(first_error);
  }
#endif
}

PyObject* create_tuple_from_c_integers(unsigned long long a,
                                       unsigned long long b,
                                       unsigned long long c,
                                       unsigned long long d) {
  // Create a new tuple of size 4
  PyObject* tuple = PyTuple_New(4);
  if (!tuple) {
    return NULL;  // Return NULL on failure
  }

  // Convert integers to Python objects and set them in the tuple
  PyTuple_SetItem(
      tuple, 0,
      PyLong_FromUnsignedLongLong(a));  // Steals reference to the PyLong
  PyTuple_SetItem(tuple, 1, PyLong_FromUnsignedLongLong(b));
  PyTuple_SetItem(tuple, 2, PyLong_FromUnsignedLongLong(c));
  PyTuple_SetItem(tuple, 3, PyLong_FromUnsignedLongLong(d));

  // Note: PyTuple_SetItem "steals" a reference to each object,
  // so we do not need to Py_DECREF the PyLong objects explicitly.

  return tuple;  // Return the created tuple
}

PyObject* create_tuple_from_c_mixed(unsigned long long a, unsigned long long b,
                                    unsigned long long c,
                                    CUmemGenericAllocationHandle** vec,
                                    unsigned long long* chunk_sizes,
                                    size_t num_chunks) {
  PyObject* tuple = PyTuple_New(4);
  if (!tuple) {
    return NULL;
  }

  // PyObject* list = PyList_New(vec.size());
  PyObject* list = PyList_New(num_chunks);
  for (auto i = 0; i < num_chunks; ++i) {
    PyObject* addr_size_pair = PyTuple_New(2);
    PyObject* addr = PyLong_FromUnsignedLongLong((unsigned long long)(vec[i]));
    PyObject* size =
        PyLong_FromUnsignedLongLong((unsigned long long)(chunk_sizes[i]));
    PyTuple_SetItem(addr_size_pair, 0, addr);
    PyTuple_SetItem(addr_size_pair, 1, size);
    PyList_SetItem(list, i, addr_size_pair);
  }

  PyTuple_SetItem(tuple, 0, PyLong_FromUnsignedLongLong(a));
  PyTuple_SetItem(tuple, 1, PyLong_FromUnsignedLongLong(b));
  PyTuple_SetItem(tuple, 2, PyLong_FromUnsignedLongLong(c));
  PyTuple_SetItem(tuple, 3, list);

  return tuple;
}
```
**EN:** unmap_and_release is the inverse helper: it unmaps virtual ranges and releases allocation handles, again handling ROCm multi-chunk teardown explicitly. The tuple-building helpers serialize native pointers and sizes into Python-owned metadata so Python can remember how to finish the allocation lifecycle later.
**CN:** unmap_and_release 是相反方向的辅助函数：它负责解除虚拟地址映射并释放分配句柄，同样显式处理 ROCm 多分块的销毁流程。后面的元组构造函数则把原生指针和大小序列化为 Python 持有的元数据，便于 Python 在稍后完成整个生命周期。

### Allocator malloc path / 分配器的 my_malloc 路径
```cpp
void* my_malloc(ssize_t size, int device, CUstream stream) {
  ensure_context(device);

  // first allocation, align the size, and reserve an address, and also allocate
  // a CUmemGenericAllocationHandle

  // Define memory allocation properties
  CUmemAllocationProp prop = {};
  prop.type = CU_MEM_ALLOCATION_TYPE_PINNED;
  prop.location.type = CU_MEM_LOCATION_TYPE_DEVICE;
  prop.location.id = device;
  prop.allocFlags.compressionType = CU_MEM_ALLOCATION_COMP_NONE;

  // Check if the allocation is supported
  size_t granularity;
  CUDA_CHECK(cuMemGetAllocationGranularity(&granularity, &prop,
                                           CU_MEM_ALLOC_GRANULARITY_MINIMUM));
  if (error_code != 0) {
    return nullptr;
  }
  size_t alignedSize = ((size + granularity - 1) / granularity) * granularity;

  CUdeviceptr d_mem;
#ifndef USE_ROCM
  CUDA_CHECK(cuMemAddressReserve(&d_mem, alignedSize, 0, 0, 0));
  if (error_code != 0) {
    return nullptr;
  }
#else
  CUDA_CHECK(cuMemAddressReserve(&d_mem, alignedSize, granularity, 0, 0));
  if (error_code != 0) {
    return nullptr;
  }
#endif

#ifndef USE_ROCM
  // allocate the CUmemGenericAllocationHandle
  CUmemGenericAllocationHandle* p_memHandle =
      (CUmemGenericAllocationHandle*)malloc(
          sizeof(CUmemGenericAllocationHandle));
#else
  // Make sure chunk size is aligned with hardware granularity. The base
  // chunk size can be configured via environment variable
  // ``VLLM_ROCM_SLEEP_MEM_CHUNK_SIZE``; otherwise
  // DEFAULT_MEMCREATE_CHUNK_SIZE is used.
  size_t base_chunk = (size_t)get_memcreate_chunk_size();
  size_t aligned_chunk_size =
      ((base_chunk + granularity - 1) / granularity) * granularity;
  size_t num_chunks =
      (alignedSize + aligned_chunk_size - 1) / aligned_chunk_size;
  CUmemGenericAllocationHandle** p_memHandle =
      (CUmemGenericAllocationHandle**)malloc(
          num_chunks * sizeof(CUmemGenericAllocationHandle*));
  unsigned long long* chunk_sizes =
      (unsigned long long*)malloc(num_chunks * sizeof(unsigned long long));
  for (auto i = 0; i < num_chunks; ++i) {
    p_memHandle[i] = (CUmemGenericAllocationHandle*)malloc(
        sizeof(CUmemGenericAllocationHandle));
    if (p_memHandle[i] == nullptr) {
      std::cerr << "ERROR: malloc failed for p_memHandle[" << i << "].\n";
      for (auto j = 0; j < i; ++j) {
        free(p_memHandle[j]);
      }
      free(p_memHandle);
      free(chunk_sizes);
      return nullptr;
    }
    chunk_sizes[i] = (unsigned long long)my_min(
        (unsigned long long)(alignedSize - i * aligned_chunk_size),
        (unsigned long long)aligned_chunk_size);
  }
#endif

  if (!g_python_malloc_callback) {
    std::cerr << "ERROR: g_python_malloc_callback not set.\n";
    return nullptr;
  }

  // Acquire GIL (not in stable ABI officially, but often works)
  PyGILState_STATE gstate = PyGILState_Ensure();

#ifndef USE_ROCM
  PyObject* arg_tuple = create_tuple_from_c_integers(
      (unsigned long long)device, (unsigned long long)alignedSize,
      (unsigned long long)d_mem, (unsigned long long)p_memHandle);
#else
  PyObject* arg_tuple = create_tuple_from_c_mixed(
      (unsigned long long)device, (unsigned long long)alignedSize,
      (unsigned long long)d_mem, p_memHandle, chunk_sizes, num_chunks);
#endif

  // Call g_python_malloc_callback
  PyObject* py_result =
      PyObject_CallFunctionObjArgs(g_python_malloc_callback, arg_tuple, NULL);
  Py_DECREF(arg_tuple);

  if (!py_result) {
    PyErr_Print();
    PyGILState_Release(gstate);
    return nullptr;
  }

  PyGILState_Release(gstate);

  // do the final mapping
#ifndef USE_ROCM
  create_and_map(device, alignedSize, d_mem, p_memHandle);
#else
  create_and_map(device, alignedSize, d_mem, p_memHandle, chunk_sizes,
                 num_chunks);
  free(chunk_sizes);
#endif

  if (error_code != 0) {
    // free address and the handle
    CUDA_CHECK(cuMemAddressFree(d_mem, alignedSize));
#ifndef USE_ROCM
    free(p_memHandle);
#else
    for (size_t i = 0; i < num_chunks; ++i) {
      free(p_memHandle[i]);
    }
    free(p_memHandle);
#endif
    return nullptr;
  }

  return (void*)d_mem;
}
```
**EN:** my_malloc aligns the requested size to VMM granularity, reserves a virtual address range with cuMemAddressReserve, allocates handle storage, calls back into Python to record the metadata, and only then performs the actual create_and_map step. This separation lets Python keep bookkeeping outside the C++ allocator while C++ retains the real low-level mapping logic.
**CN:** my_malloc 会先把请求大小按 VMM 粒度对齐，用 cuMemAddressReserve 预留虚拟地址区间，再分配句柄存储，回调 Python 记录元数据，最后才真正执行 create_and_map。这样一来，Python 可以负责外部账本管理，而底层映射逻辑仍由 C++ 控制。

### Allocator free path / 分配器的 my_free 路径
```cpp
void my_free(void* ptr, ssize_t size, int device, CUstream stream) {
  // get memory handle from the pointer
  if (!g_python_free_callback) {
    std::cerr << "ERROR: g_python_free_callback not set.\n";
    return;
  }

  // Acquire GIL (not in stable ABI officially, but often works)
  PyGILState_STATE gstate = PyGILState_Ensure();

  PyObject* py_ptr =
      PyLong_FromUnsignedLongLong(reinterpret_cast<unsigned long long>(ptr));

  PyObject* py_result =
      PyObject_CallFunctionObjArgs(g_python_free_callback, py_ptr, NULL);

  if (!py_result || !PyTuple_Check(py_result) || PyTuple_Size(py_result) != 4) {
    PyErr_SetString(PyExc_TypeError, "Expected a tuple of size 4");
    Py_XDECREF(py_result);
    Py_XDECREF(py_ptr);
    return;
  }

  unsigned long long recv_device, recv_size;
  unsigned long long recv_d_mem;
#ifndef USE_ROCM
  unsigned long long recv_p_memHandle;
#else
  PyObject* recv_p_memHandle;
#endif
  // Unpack the tuple into four C integers
  if (!PyArg_ParseTuple(py_result, PYARGS_PARSE, &recv_device, &recv_size,
                        &recv_d_mem, &recv_p_memHandle)) {
    // PyArg_ParseTuple sets an error if it fails
    Py_XDECREF(py_result);
    Py_XDECREF(py_ptr);
    return;
  }

  // For ROCm, copy the Python list of (addr,size) pairs into C arrays while
  // holding the GIL. Then release the GIL and call the unmap/release helper
  // using the copied arrays. This avoids calling PyList_* APIs without the
  // GIL (which is undefined behavior and can crash when called from other
  // threads).
  CUdeviceptr d_mem = (CUdeviceptr)recv_d_mem;
#ifdef USE_ROCM
  Py_ssize_t num_chunks = PyList_Size(recv_p_memHandle);
  CUmemGenericAllocationHandle** p_memHandle =
      (CUmemGenericAllocationHandle**)malloc(
          num_chunks * sizeof(CUmemGenericAllocationHandle*));
  if (p_memHandle == nullptr) {
    Py_DECREF(py_ptr);
    Py_DECREF(py_result);
    PyGILState_Release(gstate);
    std::cerr << "ERROR: malloc failed for p_memHandle in my_free."
              << std::endl;
    return;
  }
  unsigned long long* chunk_sizes =
      (unsigned long long*)malloc(num_chunks * sizeof(unsigned long long));
  if (chunk_sizes == nullptr) {
    free(p_memHandle);
    Py_DECREF(py_ptr);
    Py_DECREF(py_result);
    PyGILState_Release(gstate);
    std::cerr << "ERROR: malloc failed for chunk_sizes in my_free."
              << std::endl;
    return;
  }
  for (Py_ssize_t i = 0; i < num_chunks; ++i) {
    PyObject* item = PyList_GetItem(recv_p_memHandle, i);
    PyObject* addr_py = PyTuple_GetItem(item, 0);
    PyObject* size_py = PyTuple_GetItem(item, 1);
    p_memHandle[i] =
        (CUmemGenericAllocationHandle*)PyLong_AsUnsignedLongLong(addr_py);
    chunk_sizes[i] = (unsigned long long)PyLong_AsUnsignedLongLong(size_py);
  }

  // Drop temporary Python refs, then release the GIL before calling into
  // non-Python APIs.
  Py_DECREF(py_ptr);
  Py_DECREF(py_result);
  PyGILState_Release(gstate);

  unmap_and_release(device, size, d_mem, p_memHandle, chunk_sizes, num_chunks);
#else
  // Non-ROCm path: simple integer handle already extracted; drop temporary
  // Python refs while still holding the GIL, then release it.
  Py_DECREF(py_ptr);
  Py_DECREF(py_result);
  PyGILState_Release(gstate);

  CUmemGenericAllocationHandle* p_memHandle =
      (CUmemGenericAllocationHandle*)recv_p_memHandle;
  unmap_and_release(device, size, d_mem, p_memHandle);
#endif

  // free address and the handle
  CUDA_CHECK(cuMemAddressFree(d_mem, size));
#ifndef USE_ROCM
  free(p_memHandle);
#else
  for (auto i = 0; i < num_chunks; ++i) {
    free(p_memHandle[i]);
  }
  free(p_memHandle);
  free(chunk_sizes);
#endif
}
```
**EN:** my_free asks Python for the metadata associated with a pointer, reconstructs native handle/chunk arrays, releases the GIL before performing non-Python teardown, then unmaps the VMM region, releases the allocation handles, frees the reserved address range, and finally frees the host-side bookkeeping arrays.
**CN:** my_free 会向 Python 查询某个指针对应的元数据，重建本地句柄/分块数组，在进行非 Python 释放逻辑前先释放 GIL，然后解除 VMM 映射、释放分配句柄、释放预留地址区间，最后再释放主机端账本数组。

### Callback registration entry point / 回调注册入口
```cpp
// Python-exposed function: init_module(python_malloc, python_free)
static PyObject* py_init_module(PyObject* self, PyObject* args) {
  PyObject* malloc_callback = nullptr;
  PyObject* free_callback = nullptr;

  if (!PyArg_ParseTuple(args, "OO", &malloc_callback, &free_callback)) {
    return nullptr;
  }

  if (!PyCallable_Check(malloc_callback) || !PyCallable_Check(free_callback)) {
    PyErr_SetString(PyExc_TypeError, "Both arguments must be callables");
    return nullptr;
  }

  // Save the Python callables
  // This module does not handle GC of these objects, so they must be kept alive
  // outside of this module.
  g_python_malloc_callback = malloc_callback;
  g_python_free_callback = free_callback;

  Py_RETURN_NONE;
```
**EN:** py_init_module stores borrowed references to the Python malloc/free callables. The design assumes the callbacks outlive the extension module state, effectively making the allocator behave like a singleton bridge.
**CN:** py_init_module 保存 Python malloc/free 回调的借用引用。这个设计假设回调对象的生命周期长于扩展模块状态，因此整体上把分配器实现成一种单例式桥接层。

### Python-visible create/unmap wrappers and module definition / 面向 Python 的创建/释放包装与模块定义
```cpp
static PyObject* python_unmap_and_release(PyObject* self, PyObject* args) {
  if (!args || !PyTuple_Check(args) || PyTuple_Size(args) != 4) {
    PyErr_SetString(PyExc_TypeError, "Expected a tuple of size 4");
    return nullptr;
  }

  unsigned long long recv_device, recv_size;
  unsigned long long recv_d_mem;
#ifndef USE_ROCM
  unsigned long long recv_p_memHandle;
#else
  PyObject* recv_p_memHandle;
#endif
  // Unpack the tuple into four C integers
  if (!PyArg_ParseTuple(args, PYARGS_PARSE, &recv_device, &recv_size,
                        &recv_d_mem, &recv_p_memHandle)) {
    // PyArg_ParseTuple sets an error if it fails
    return nullptr;
  }

  CUdeviceptr d_mem_ptr = (CUdeviceptr)recv_d_mem;
#ifndef USE_ROCM
  CUmemGenericAllocationHandle* p_memHandle =
      (CUmemGenericAllocationHandle*)recv_p_memHandle;

  unmap_and_release(recv_device, recv_size, d_mem_ptr, p_memHandle);
#else
  if (!PyList_Check(recv_p_memHandle)) {
    PyErr_SetString(PyExc_TypeError,
                    "Expected a list for the 4th argument on ROCm");
    return nullptr;
  }
  Py_ssize_t num_chunks = PyList_Size(recv_p_memHandle);
  if (num_chunks < 0) {
    return nullptr;  // PyList_Size sets an exception on error.
  }
  CUmemGenericAllocationHandle** p_memHandle =
      (CUmemGenericAllocationHandle**)malloc(
          num_chunks * sizeof(CUmemGenericAllocationHandle*));
  if (p_memHandle == nullptr) {
    PyErr_SetString(PyExc_MemoryError, "malloc failed for p_memHandle");
    return nullptr;
  }
  unsigned long long* chunk_sizes =
      (unsigned long long*)malloc(num_chunks * sizeof(unsigned long long));
  if (chunk_sizes == nullptr) {
    free(p_memHandle);
    PyErr_SetString(PyExc_MemoryError, "malloc failed for chunk_sizes");
    return nullptr;
  }
  for (Py_ssize_t i = 0; i < num_chunks; ++i) {
    PyObject* item = PyList_GetItem(recv_p_memHandle, i);
    if (item == nullptr || !PyTuple_Check(item) || PyTuple_Size(item) != 2) {
      free(p_memHandle);
      free(chunk_sizes);
      PyErr_SetString(
          PyExc_TypeError,
          "List items must be tuples of size 2 (handle_addr, size)");
      return nullptr;
    }
    PyObject* addr_py = PyTuple_GetItem(item, 0);
    PyObject* size_py = PyTuple_GetItem(item, 1);
    if (addr_py == nullptr || size_py == nullptr) {
      free(p_memHandle);
      free(chunk_sizes);
      return nullptr;  // PyTuple_GetItem sets an exception
    }
    p_memHandle[i] =
        (CUmemGenericAllocationHandle*)PyLong_AsUnsignedLongLong(addr_py);
    if (PyErr_Occurred()) {
      free(p_memHandle);
      free(chunk_sizes);
      return nullptr;
    }
    chunk_sizes[i] = (unsigned long long)PyLong_AsUnsignedLongLong(size_py);
    if (PyErr_Occurred()) {
      free(p_memHandle);
      free(chunk_sizes);
      return nullptr;
    }
  }

  unmap_and_release(recv_device, recv_size, d_mem_ptr, p_memHandle, chunk_sizes,
                    num_chunks);

  free(p_memHandle);
  free(chunk_sizes);
#endif

  if (error_code != 0) {
    error_code = no_error;
    PyErr_SetString(PyExc_RuntimeError, error_msg);
    return nullptr;
  }

  Py_RETURN_NONE;
}

static PyObject* python_create_and_map(PyObject* self, PyObject* args) {
  if (!args || !PyTuple_Check(args) || PyTuple_Size(args) != 4) {
    PyErr_SetString(PyExc_TypeError, "Expected a tuple of size 4");
    return nullptr;
  }

  unsigned long long recv_device, recv_size;
  unsigned long long recv_d_mem;
#ifndef USE_ROCM
  unsigned long long recv_p_memHandle;
#else
  PyObject* recv_p_memHandle;
#endif
  // Unpack the tuple into four C integers
  if (!PyArg_ParseTuple(args, PYARGS_PARSE, &recv_device, &recv_size,
                        &recv_d_mem, &recv_p_memHandle)) {
    // PyArg_ParseTuple sets an error if it fails
    return nullptr;
  }

  CUdeviceptr d_mem_ptr = (CUdeviceptr)recv_d_mem;
#ifndef USE_ROCM
  CUmemGenericAllocationHandle* p_memHandle =
      (CUmemGenericAllocationHandle*)recv_p_memHandle;

  create_and_map(recv_device, recv_size, d_mem_ptr, p_memHandle);
#else
  Py_ssize_t num_chunks = PyList_Size(recv_p_memHandle);
  CUmemGenericAllocationHandle** p_memHandle =
      (CUmemGenericAllocationHandle**)malloc(
          num_chunks * sizeof(CUmemGenericAllocationHandle*));
  if (p_memHandle == nullptr) {
    PyErr_SetString(PyExc_MemoryError, "malloc failed for p_memHandle");
    return nullptr;
  }
  unsigned long long* chunk_sizes =
      (unsigned long long*)malloc(num_chunks * sizeof(unsigned long long));
  if (chunk_sizes == nullptr) {
    free(p_memHandle);
    PyErr_SetString(PyExc_MemoryError, "malloc failed for chunk_sizes");
    return nullptr;
  }
  for (auto i = 0; i < num_chunks; ++i) {
    PyObject* item = PyList_GetItem(recv_p_memHandle, i);
    PyObject* addr_py = PyTuple_GetItem(item, 0);
    PyObject* size_py = PyTuple_GetItem(item, 1);
    p_memHandle[i] =
        (CUmemGenericAllocationHandle*)PyLong_AsUnsignedLongLong(addr_py);
    chunk_sizes[i] = PyLong_AsUnsignedLongLong(size_py);
  }

  create_and_map(recv_device, recv_size, d_mem_ptr, p_memHandle, chunk_sizes,
                 num_chunks);

  free(p_memHandle);
  free(chunk_sizes);
#endif

  if (error_code != 0) {
    error_code = no_error;
    PyErr_SetString(PyExc_RuntimeError, error_msg);
    return nullptr;
  }

  Py_RETURN_NONE;
}

static PyMethodDef module_methods[] = {
    {"init_module", (PyCFunction)py_init_module, METH_VARARGS,
     "Initialize module with python_malloc and python_free callables."},
    {"python_create_and_map", (PyCFunction)python_create_and_map, METH_VARARGS,
     "Create and map memory on the device."},
    {"python_unmap_and_release", (PyCFunction)python_unmap_and_release,
     METH_VARARGS, "Unmap and release memory on the device."},
    {NULL, NULL, 0, NULL}  // sentinel
};

static struct PyModuleDef cumem_allocator_module = {
    PyModuleDef_HEAD_INIT, "cumem_allocator",
    "cumem-based allocator for CUDAPluggableAllocator", -1, module_methods};

PyMODINIT_FUNC PyInit_cumem_allocator(void) {
  // Initialize the module
  PyObject* module = PyModule_Create(&cumem_allocator_module);
  if (!module) {
    return NULL;
  }
  return module;
}
```
**EN:** python_unmap_and_release and python_create_and_map expose the same VMM helpers to Python code directly. They parse Python tuples back into native pointer metadata, validate ROCm list payloads when needed, propagate stored CUDA errors as Python exceptions, and then register the extension methods in the cumem_allocator module.
**CN:** python_unmap_and_release 与 python_create_and_map 把同一套 VMM 辅助函数直接暴露给 Python。它们把 Python 元组解析回本地指针元数据，在 ROCm 场景下校验列表负载，把缓存的 CUDA 错误提升为 Python 异常，并最终在 cumem_allocator 模块中注册扩展方法。

## Key Concepts / 关键概念

- **EN:** cuMemAddressReserve separates virtual address reservation from backing physical allocation.
  **CN:** cuMemAddressReserve 把虚拟地址预留与底层物理分配解耦。
- **EN:** Python stores allocator metadata while C++ executes the privileged VMM operations.
  **CN:** Python 负责保存分配元数据，而 C++ 执行底层且更敏感的 VMM 操作。
- **EN:** The ROCm path chunks large allocations because a single mem-create handle may not be the desired compatibility strategy there.
  **CN:** ROCm 路径通过分块处理大分配，因为单一 mem-create 句柄并不一定适合作为兼容策略。

## Dependencies / 依赖关系

- **EN:** Relies on csrc/cumem_allocator_compat.h for CUDA/HIP driver API compatibility.
  **CN:** 依赖 csrc/cumem_allocator_compat.h 提供 CUDA/HIP Driver API 兼容层。
- **EN:** Uses Python C API to exchange allocator metadata with higher-level Python code.
  **CN:** 使用 Python C API 与上层 Python 代码交换分配器元数据。
- **EN:** Uses CUDA/ROCm virtual memory APIs such as cuMemCreate, cuMemMap, cuMemSetAccess, and cuMemRelease.
  **CN:** 依赖 cuMemCreate、cuMemMap、cuMemSetAccess、cuMemRelease 等 CUDA/ROCm 虚拟内存 API。
