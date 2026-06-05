# SymmetricMemory.hpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/symm_mem/SymmetricMemory.hpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides interface and type declarations for symmetric memory in the c10d symmetric-memory support. Key types include `TORCH_API`, `SymmetricMemoryAllocator`, `GroupInfo`.
- 用途 (CN): 该文件在c10d 对称内存支持中提供symmetric memory 的接口与类型声明。 关键类型包括 `TORCH_API`、`SymmetricMemoryAllocator`、`GroupInfo`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #pragma once
2: 
3: #include <ATen/ATen.h>
4: #include <ATen/core/ivalue.h>
5: #include <torch/csrc/distributed/c10d/Store.hpp>
6: 
7: namespace c10d::symmetric_memory {
8: 
9: // SymmetricMemory represents symmetric allocations across a group of devices.
10: // The allocations represented by a SymmetricMemory object are accessible by
11: // all devices in the group. The class can be used for op-level custom
12: // communication patterns (via the get_buffer APIs and the synchronization
13: // primitives), as well as custom communication kernels (via the buffer and
14: // signal_pad device pointers).
15: //
16: // To acquire a SymmetricMemory object, each rank first allocates
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 17-32 / 第 17-32 行

```cpp
17: // identical-sized memory via SymmetricMemoryAllocator::alloc(), then invokes
18: // SymmetricMemoryAllocator::rendezvous() on the memory to establish the
19: // association across peer buffers. The rendezvous is a one-time process, and
20: // the mapping between a local memory memory and the associated SymmetricMemory
21: // object is unique.
22: //
23: // NOTE [symmetric memory signal pad]
24: // Signal pads are P2P-accessible memory regions designated for
25: // synchronization. SymmetricMemory offers built-in synchronization primitives
26: // such as barriers, put_signal, and wait_signal, which are all based on signal
27: // pads. Users may utilize signal pads for their own synchronization logic,
28: // provided that the signal pads remain zero-filled following successful
29: // synchronization.
30: //
31: // NOTE [symmetric memory synchronization channel]
32: // Synchronization channels allow users to use a single SymmetricMemory object
```

- EN: Lines 17-32 continues the local implementation details and data flow for this file.
- CN: 第 17-32 行继续展开本文件的局部实现细节与数据流。

### Lines 33-48 / 第 33-48 行

```cpp
33: // to perform isolated synchronizations on different streams. For example,
34: // consider the case in which two barriers are issued on two streams for
35: // different purposes. Without the concept of channels, we cannot guarantee the
36: // correctness of the barriers since signals issued from barrier on stream A
37: // can be received by the barrier on stream B. By specifying different channels
38: // for these two barriers, they can operate correctly in parallel.
39: class TORCH_API SymmetricMemory : public torch::CustomClassHolder {
40:  public:
41:   ~SymmetricMemory() override = default;
42: 
43:   virtual std::vector<void*> get_buffer_ptrs() = 0;
44:   virtual std::vector<void*> get_signal_pad_ptrs() = 0;
45: 
46:   // get_buffer_ptrs_dev() and get_signal_pad_ptrs_dev() each return a pointer
47:   // to a device array of size world_size, containing buffer pointers and
48:   // signal pad pointers, respectively.
```

- EN: Lines 33-48 declares or defines types such as `TORCH_API`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 33-48 行声明或定义了 `TORCH_API` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 49-64 / 第 49-64 行

```cpp
49:   virtual void** get_buffer_ptrs_dev() = 0;
50:   virtual void** get_signal_pad_ptrs_dev() = 0;
51:   virtual size_t get_buffer_size() = 0;
52:   size_t get_signal_pad_size();
53: 
54:   virtual size_t get_offset() = 0;
55: 
56:   virtual bool has_multicast_support() = 0;
57:   virtual void* get_multicast_ptr() = 0;
58: 
59:   at::Tensor get_buffer(
60:       int rank,
61:       c10::IntArrayRef sizes,
62:       c10::ScalarType dtype,
63:       int64_t storage_offset);
64: 
```

- EN: Lines 49-64 introduces executable logic in routines such as `get_signal_pad_size`, `get_buffer`.
- CN: 第 49-64 行在 `get_signal_pad_size`、`get_buffer` 等例程中引入具体执行逻辑。

### Lines 65-80 / 第 65-80 行

```cpp
65:   at::Tensor get_signal_pad(
66:       int rank,
67:       c10::IntArrayRef sizes,
68:       std::optional<c10::ScalarType> dtype = std::nullopt,
69:       int64_t storage_offset = 0);
70: 
71:   at::Tensor get_remote_tensor(
72:       int peer,
73:       c10::IntArrayRef sizes,
74:       c10::ScalarType dtype);
75: 
76:   virtual void barrier(int channel, size_t timeout_ms) = 0;
77:   virtual void put_signal(int dst_rank, int channel, size_t timeout_ms) = 0;
78:   virtual void wait_signal(int src_rank, int channel, size_t timeout_ms) = 0;
79: 
80:   virtual int get_rank() = 0;
```

- EN: Lines 65-80 introduces executable logic in routines such as `get_signal_pad`, `get_remote_tensor`.
- CN: 第 65-80 行在 `get_signal_pad`、`get_remote_tensor` 等例程中引入具体执行逻辑。

### Lines 81-96 / 第 81-96 行

```cpp
81:   virtual int get_world_size() = 0;
82:   virtual c10::Device get_device() = 0;
83: 
84:   virtual const std::vector<int>& get_rank_to_global_rank() {
85:     TORCH_CHECK(false, "NYI");
86:   }
87: 
88:   virtual int* get_rank_to_global_rank_dev() {
89:     TORCH_CHECK(false, "NYI");
90:   }
91: 
92:   // Returns true if *all* peers within the group are accessible via direct
93:   // memory load and store.
94:   virtual bool world_within_direct_access() {
95:     TORCH_CHECK(false, "NYI");
96:   }
```

- EN: Lines 81-96 introduces executable logic in routines such as `get_rank_to_global_rank`, `get_rank_to_global_rank_dev`, `world_within_direct_access`; performs validation and error handling to keep distributed state consistent.
- CN: 第 81-96 行在 `get_rank_to_global_rank`、`get_rank_to_global_rank_dev`、`world_within_direct_access` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 97-112 / 第 97-112 行

```cpp
97: };
98: 
99: class SymmetricMemoryAllocator : public c10::intrusive_ptr_target {
100:  public:
101:   ~SymmetricMemoryAllocator() override = default;
102: 
103:   virtual void* alloc(
104:       size_t size,
105:       int device_idx,
106:       const std::optional<std::string>& group_name) = 0;
107: 
108:   virtual void free(void* ptr) = 0;
109:   virtual size_t get_alloc_size(void* ptr) = 0;
110:   virtual c10::intrusive_ptr<SymmetricMemory> rendezvous(
111:       void* ptr,
112:       const std::optional<std::string>& group_name) = 0;
```

- EN: Lines 97-112 declares or defines types such as `SymmetricMemoryAllocator`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 97-112 行声明或定义了 `SymmetricMemoryAllocator` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 113-128 / 第 113-128 行

```cpp
113:   virtual bool has_multicast_support(int device_idx) = 0;
114:   virtual c10::DeviceType supported_device_type() = 0;
115:   virtual std::string name() = 0;
116:   virtual bool has_allocation(void* ptr) {
117:     return false;
118:   }
119: };
120: 
121: C10_EXPORT bool is_finalizing();
122: 
123: C10_EXPORT void register_allocator(
124:     c10::DeviceType device_type,
125:     c10::intrusive_ptr<SymmetricMemoryAllocator> allocator);
126: 
127: C10_EXPORT void register_availability(
128:     const std::string& name,
```

- EN: Lines 113-128 introduces executable logic in routines such as `has_allocation`, `is_finalizing`, `register_allocator`; returns computed state or forwards results to the surrounding caller.
- CN: 第 113-128 行在 `has_allocation`、`is_finalizing`、`register_allocator` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 129-144 / 第 129-144 行

```cpp
129:     c10::intrusive_ptr<SymmetricMemoryAllocator> allocator);
130: 
131: C10_EXPORT bool has_allocator(c10::DeviceType device_type);
132: 
133: C10_EXPORT c10::intrusive_ptr<SymmetricMemoryAllocator> get_allocator(
134:     c10::DeviceType device_type);
135: 
136: // Set a store for rendezvousing symmetric allocations on a group of devices
137: // identified by `group_name`. The concept of groups is logical; users can
138: // utilize predefined groups (e.g., a group of device identified by a
139: // ProcessGroup) or create custom ones. Note that a SymmetricMemoryAllocator
140: // backends might employ a more efficient communication channel for the actual
141: // rendezvous process and only use the store for bootstrapping purposes.
142: TORCH_API void set_group_info(
143:     const std::string& group_name,
144:     int rank,
```

- EN: Lines 129-144 introduces executable logic in routines such as `has_allocator`, `get_allocator`.
- CN: 第 129-144 行在 `has_allocator`、`get_allocator` 等例程中引入具体执行逻辑。

### Lines 145-160 / 第 145-160 行

```cpp
145:     int world_size,
146:     c10::intrusive_ptr<Store> store);
147: 
148: struct GroupInfo {
149:   int rank;
150:   int world_size;
151:   c10::intrusive_ptr<c10d::Store> store;
152: };
153: 
154: C10_EXPORT GroupInfo& get_group_info(const std::string& group_name);
155: 
156: // Identical to empty_strided, but allows symmetric memory access to be
157: // established for the allocated tensor via SymmetricMemory::rendezvous(). This
158: // function itself is not a collective operation. It invokes
159: // SymmetricMemoryAllocator::alloc() for the requested device under the hood.
160: //
```

- EN: Lines 145-160 declares or defines types such as `GroupInfo`; introduces executable logic in routines such as `get_group_info`.
- CN: 第 145-160 行声明或定义了 `GroupInfo` 等类型；在 `get_group_info` 等例程中引入具体执行逻辑。

### Lines 161-176 / 第 161-176 行

```cpp
161: // NOTE [symmetric memory persistent allocation]
162: // If an `alloc_id` is supplied, empty_strided_p2p will perform persistent
163: // allocation. This makes the function cache allocated memory and ensure that
164: // invocations with the same `alloc_id` receive tensors backed by the same
165: // memory address. For safety, if a previous persistent allocation is still
166: // active (i.e., the storage of the returned tensor is still alive), persistent
167: // allocations with the same `alloc_id` will fail. This determinism coupled
168: // with memory planning of communication buffers (e.g., by Inductor) allows
169: // communication algorithms to reliably reuse previously established remote
170: // memory access.
171: TORCH_API at::Tensor empty_strided_p2p(
172:     c10::IntArrayRef size,
173:     c10::IntArrayRef stride,
174:     c10::ScalarType dtype,
175:     c10::Device device,
176:     const std::optional<std::string>& group_name,
```

- EN: Lines 161-176 continues the local implementation details and data flow for this file.
- CN: 第 161-176 行继续展开本文件的局部实现细节与数据流。

### Lines 177-192 / 第 177-192 行

```cpp
177:     std::optional<uint64_t> alloc_id);
178: 
179: // Establishes symmetric memory access on tensors allocated via
180: // empty_strided_p2p() and empty_strided_p2p_persistent(). rendezvous() is a
181: // one-time process, and the mapping between a local memory region and the
182: // associated SymmetricMemory object is unique. Subsequent calls to
183: // rendezvous() with the same tensor, or tensors allocated with
184: // empty_strided_p2p_persistent() using the same alloc_id, will receive the
185: // cached SymmetricMemory object.
186: //
187: // The function has a collective semantic and must be invoked simultaneously
188: // from all rendezvous participants.
189: TORCH_API c10::intrusive_ptr<SymmetricMemory> rendezvous(
190:     const at::Tensor& tensor,
191:     const std::optional<std::string>& group_name = std::nullopt);
192: 
```

- EN: Lines 177-192 introduces executable logic in routines such as `rendezvous`.
- CN: 第 177-192 行在 `rendezvous` 等例程中引入具体执行逻辑。

### Lines 193-208 / 第 193-208 行

```cpp
193: TORCH_API bool has_multicast_support(
194:     c10::DeviceType device_type,
195:     int device_idx);
196: 
197: TORCH_API bool is_symm_mem_tensor(const at::Tensor& tensor);
198: 
199: TORCH_API void set_backend(const std::string& name);
200: 
201: TORCH_API std::optional<std::string> get_backend(c10::Device device);
202: 
203: // Get the current signal pad size for symmetric memory allocations.
204: // Returns the user-configured size if set, otherwise returns the default size.
205: TORCH_API size_t get_signal_pad_size();
206: 
207: // Set the signal pad size for future symmetric memory allocations.
208: // This must be called before any symmetric memory allocations are made.
```

- EN: Lines 193-208 introduces executable logic in routines such as `has_multicast_support`, `is_symm_mem_tensor`, `set_backend`.
- CN: 第 193-208 行在 `has_multicast_support`、`is_symm_mem_tensor`、`set_backend` 等例程中引入具体执行逻辑。

### Lines 209-220 / 第 209-220 行

```cpp
209: // The size should be proportional to the number of blocks the user launches
210: // and the world size.
211: TORCH_API void set_signal_pad_size(size_t size);
212: 
213: C10_EXPORT void register_mempool_allocator(
214:     c10::DeviceType device_type,
215:     std::shared_ptr<c10::Allocator> allocator);
216: 
217: TORCH_API std::shared_ptr<c10::Allocator> get_mempool_allocator(
218:     c10::Device device);
219: 
220: } // namespace c10d::symmetric_memory
```

- EN: Lines 209-220 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `set_signal_pad_size`, `register_mempool_allocator`, `get_mempool_allocator`.
- CN: 第 209-220 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `set_signal_pad_size`、`register_mempool_allocator`、`get_mempool_allocator` 等例程中引入具体执行逻辑。

## Key Concepts / 关键概念

- EN: Subsystem: c10d symmetric-memory support.
- CN: 子系统：c10d 对称内存支持。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `TORCH_API`, `SymmetricMemoryAllocator`, `GroupInfo`
- CN: 核心符号：`TORCH_API`、`SymmetricMemoryAllocator`、`GroupInfo`
- EN: Notable themes: process-group orchestration, store/state coordination, collective communication logic.
- CN: 值得关注的主题：进程组编排、存储/状态协调、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/Store.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/ATen.h`, `ATen/core/ivalue.h`
- External or system headers / 外部或系统头文件: None / 无
- Local symbols / 本地符号: `TORCH_API`, `SymmetricMemoryAllocator`, `GroupInfo`