# CUDASymmetricMemory.hpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory.hpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides interface and type declarations for cudasymmetric memory in the c10d symmetric-memory support. Key types include `AllocationRef`, `CUDAPeerAllocInfo`, `CUDASymmetricMemory`.
- 用途 (CN): 该文件在c10d 对称内存支持中提供cudasymmetric memory 的接口与类型声明。 关键类型包括 `AllocationRef`、`CUDAPeerAllocInfo`、`CUDASymmetricMemory`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```cpp
1: #pragma once
2: 
3: #include <ATen/ATen.h>
4: #include <c10/cuda/CUDAAllocatorConfig.h>
5: #include <torch/csrc/distributed/c10d/Store.hpp>
6: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryTypes.hpp>
7: #include <torch/csrc/distributed/c10d/symm_mem/SymmetricMemory.hpp>
8: 
9: namespace c10d::symmetric_memory {
10: 
11: // Resource wrapper that owns a (vaddr, allocation handle) pair. Upon
12: // destruction, it unmaps the vaddr and releases the allocation handle.
```

- EN: Lines 1-12 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 1-12 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 13-24 / 第 13-24 行

```cpp
13: struct AllocationRef : public c10::intrusive_ptr_target {
14:   void* ptr;
15:   HandleType handle;
16:   size_t block_size;
17:   int device_idx;
18:   bool is_multicast;
19: 
20:   AllocationRef(
21:       void* ptr,
22:       HandleType handle,
23:       size_t block_size,
24:       int device_idx,
```

- EN: Lines 13-24 declares or defines types such as `AllocationRef`.
- CN: 第 13-24 行声明或定义了 `AllocationRef` 等类型。

### Lines 25-36 / 第 25-36 行

```cpp
25:       bool is_multicast = false);
26: 
27:   ~AllocationRef();
28: };
29: 
30: // Forward declaration of CUDAPeerAllocInfo
31: class CUDAPeerAllocInfo;
32: 
33: class CUDASymmetricMemory : public SymmetricMemory {
34:  public:
35:   // This is mostly a shallow copy that shares the pointer to
36:   // `CUDAPeerAllocInfo` which corresponds to the base Block. The
```

- EN: Lines 25-36 declares or defines types such as `CUDAPeerAllocInfo`, `CUDASymmetricMemory`; introduces executable logic in routines such as `~AllocationRef`.
- CN: 第 25-36 行声明或定义了 `CUDAPeerAllocInfo`、`CUDASymmetricMemory` 等类型；在 `~AllocationRef` 等例程中引入具体执行逻辑。

### Lines 37-48 / 第 37-48 行

```cpp
37:   // CUDASymmetricMemory handle is specified by the offset to the base ptr.
38:   CUDASymmetricMemory(
39:       const c10::intrusive_ptr<CUDAPeerAllocInfo>& pai,
40:       size_t offset);
41: 
42:   ~CUDASymmetricMemory() override {};
43: 
44:   std::vector<void*> get_buffer_ptrs() override;
45:   std::vector<void*> get_signal_pad_ptrs() override;
46:   void** get_buffer_ptrs_dev() override;
47:   void** get_signal_pad_ptrs_dev() override;
48:   size_t get_buffer_size() override;
```

- EN: Lines 37-48 introduces executable logic in routines such as `CUDASymmetricMemory`, `~CUDASymmetricMemory`, `get_buffer_ptrs`.
- CN: 第 37-48 行在 `CUDASymmetricMemory`、`~CUDASymmetricMemory`、`get_buffer_ptrs` 等例程中引入具体执行逻辑。

### Lines 49-60 / 第 49-60 行

```cpp
49:   size_t get_offset() override;
50: 
51:   bool has_multicast_support() override;
52:   void* get_multicast_ptr() override;
53: 
54:   void barrier(int channel, size_t timeout_ms) override;
55:   void put_signal(int dst_rank, int channel, size_t timeout_ms) override;
56:   void wait_signal(int src_rank, int channel, size_t timeout_ms) override;
57: 
58:   int get_rank() override;
59:   int get_world_size() override;
60:   c10::Device get_device() override;
```

- EN: Lines 49-60 introduces executable logic in routines such as `get_offset`, `has_multicast_support`, `get_multicast_ptr`.
- CN: 第 49-60 行在 `get_offset`、`has_multicast_support`、`get_multicast_ptr` 等例程中引入具体执行逻辑。

### Lines 61-72 / 第 61-72 行

```cpp
61:   bool world_within_direct_access() override;
62: 
63:  private:
64:   int local_device_idx_;
65:   int rank_;
66:   int world_size_;
67:   c10::intrusive_ptr<CUDAPeerAllocInfo> pai_;
68:   size_t offset_{0}; // in byte
69: };
70: 
71: // A class to hold the base pointers and signal pad pointers for a group of
72: // peers. One `CUDAPeerAllocInfo` object can be shared by multiple
```

- EN: Lines 61-72 introduces executable logic in routines such as `world_within_direct_access`.
- CN: 第 61-72 行在 `world_within_direct_access` 等例程中引入具体执行逻辑。

### Lines 73-84 / 第 73-84 行

```cpp
73: // `CUDASymmetricMemory` objects when latter reside on the same allocation
74: // and rendezvous over the same group. (The `CUDASymmetricMemory` objects may
75: // have different offsets compared to the base address.)
76: class CUDAPeerAllocInfo : public c10::intrusive_ptr_target {
77:  public:
78:   CUDAPeerAllocInfo(
79:       std::vector<c10::intrusive_ptr<AllocationRef>> alloc_refs,
80:       std::vector<void*> buffers,
81:       std::vector<void*> signal_pads,
82:       HandleType mc_handle,
83:       void* mc_addr,
84:       size_t buffer_size,
```

- EN: Lines 73-84 declares or defines types such as `CUDAPeerAllocInfo`.
- CN: 第 73-84 行声明或定义了 `CUDAPeerAllocInfo` 等类型。

### Lines 85-96 / 第 85-96 行

```cpp
85:       int local_device_idx,
86:       int rank,
87:       int world_size,
88:       std::string group_name);
89: 
90:  private:
91:   std::vector<c10::intrusive_ptr<AllocationRef>> alloc_refs_;
92:   std::vector<void*> buffers_;
93:   std::vector<void*> signal_pads_;
94:   HandleType mc_handle_;
95:   void* mc_addr_;
96:   size_t buffer_size_;
```

- EN: Lines 85-96 continues the local implementation details and data flow for this file.
- CN: 第 85-96 行继续展开本文件的局部实现细节与数据流。

### Lines 97-108 / 第 97-108 行

```cpp
97:   int local_device_idx_;
98:   int rank_;
99:   int world_size_;
100:   void** buffers_dev_;
101:   void** signal_pads_dev_;
102:   std::string group_name_;
103: 
104:   friend class CUDASymmetricMemory;
105: };
106: 
107: // Metadata associated with each allocation performed by
108: // `CUDASymmetricMemoryAllocator`.
```

- EN: Lines 97-108 mainly closes scopes and completes the surrounding declaration or implementation block.
- CN: 第 97-108 行主要用于关闭作用域并结束当前声明或实现块。

### Lines 109-120 / 第 109-120 行

```cpp
109: struct Block : public c10::intrusive_ptr_target {
110:   c10::intrusive_ptr<AllocationRef> alloc_ref;
111:   int device_idx;
112:   size_t block_size;
113:   size_t buffer_size;
114:   size_t signal_pad_offset;
115:   std::optional<std::string> default_group_name;
116:   std::map<std::string, c10::intrusive_ptr<CUDAPeerAllocInfo>> symm_mems;
117: 
118:   Block(
119:       c10::intrusive_ptr<AllocationRef> alloc_ref,
120:       int device_idx,
```

- EN: Lines 109-120 declares or defines types such as `Block`.
- CN: 第 109-120 行声明或定义了 `Block` 等类型。

### Lines 121-132 / 第 121-132 行

```cpp
121:       size_t block_size,
122:       size_t buffer_size,
123:       size_t signal_pad_offset,
124:       const std::optional<std::string>& group_name);
125: };
126: 
127: class CUDASymmetricMemoryAllocator : public SymmetricMemoryAllocator {
128:  public:
129:   void* alloc(
130:       size_t size,
131:       int device_idx,
132:       const std::optional<std::string>& group_name) override;
```

- EN: Lines 121-132 declares or defines types such as `CUDASymmetricMemoryAllocator`; introduces executable logic in routines such as `alloc`.
- CN: 第 121-132 行声明或定义了 `CUDASymmetricMemoryAllocator` 等类型；在 `alloc` 等例程中引入具体执行逻辑。

### Lines 133-144 / 第 133-144 行

```cpp
133: 
134:   void free(void* ptr) override;
135:   size_t get_alloc_size(void* ptr) override;
136:   c10::intrusive_ptr<SymmetricMemory> rendezvous(
137:       void* ptr,
138:       const std::optional<std::string>& group_name) override;
139:   bool has_multicast_support(int device_idx) override;
140:   bool has_allocation(void* ptr) override;
141:   c10::DeviceType supported_device_type() override;
142:   std::string name() override;
143: 
144:  private:
```

- EN: Lines 133-144 introduces executable logic in routines such as `free`, `get_alloc_size`, `rendezvous`.
- CN: 第 133-144 行在 `free`、`get_alloc_size`、`rendezvous` 等例程中引入具体执行逻辑。

### Lines 145-155 / 第 145-155 行

```cpp
145:   c10::intrusive_ptr<Block> find_block(void* ptr);
146:   c10::intrusive_ptr<Block> find_block_covering(void* ptr, size_t& offset);
147: 
148:   std::shared_mutex mutex_;
149:   std::unordered_map<void*, c10::intrusive_ptr<Block>> ptr_to_block_;
150:   c10::cuda::CUDACachingAllocator::Expandable_Segments_Handle_Type
151:       handle_type_ = c10::cuda::CUDACachingAllocator::
152:           Expandable_Segments_Handle_Type::UNSPECIFIED;
153: };
154: 
155: } // namespace c10d::symmetric_memory
```

- EN: Lines 145-155 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `find_block`, `find_block_covering`.
- CN: 第 145-155 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `find_block`、`find_block_covering` 等例程中引入具体执行逻辑。

## Key Concepts / 关键概念

- EN: Subsystem: c10d symmetric-memory support.
- CN: 子系统：c10d 对称内存支持。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `AllocationRef`, `CUDAPeerAllocInfo`, `CUDASymmetricMemory`, `Block`
- CN: 核心符号：`AllocationRef`、`CUDAPeerAllocInfo`、`CUDASymmetricMemory`、`Block`
- EN: Notable themes: store/state coordination.
- CN: 值得关注的主题：存储/状态协调。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/Store.hpp`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryTypes.hpp`, `torch/csrc/distributed/c10d/symm_mem/SymmetricMemory.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/ATen.h`, `c10/cuda/CUDAAllocatorConfig.h`
- External or system headers / 外部或系统头文件: None / 无
- Local symbols / 本地符号: `AllocationRef`, `CUDAPeerAllocInfo`, `CUDASymmetricMemory`, `Block`