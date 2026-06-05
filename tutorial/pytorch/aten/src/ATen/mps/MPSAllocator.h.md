# MPSAllocator.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/mps/MPSAllocator.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Apple MPS-backed ATen operators, resource management, or execution helpers. This specific file centers on `MPSAllocator.h`. The file header highlights: "this implementation is based on CUDACachingAllocator. It utilizes Metal Heaps to improve the performance with buffer allocation. Do not include this header. Use MPSAllocatorInterface.h instead. TODO: Unify the logic with CUDACachingAlloc...." Descriptor/handle lifecycle management is important here. The implementation pays special attention to memory allocation and ownership boundaries.
- **Purpose (CN)**: 实现基于 Apple MPS 的 ATen 算子、资源管理或执行辅助逻辑。 该文件具体围绕 `MPSAllocator.h` 展开。 文件头部注释也概括了其核心职责。 描述符/句柄的生命周期管理是这里的重要内容。 该实现特别关注内存分配与所有权边界。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行

```cpp
0001: //  Copyright © 2022 Apple Inc.
0002: 
0003: #pragma once
0004: 
0005: #include <ATen/mps/MPSAllocatorInterface.h>
0006: #include <ATen/mps/MPSEvent.h>
0007: #include <ATen/mps/MPSStream.h>
0008: 
0009: #include <c10/util/flat_hash_map.h>
0010: #include <mach/vm_page_size.h>
0011: #include <cstdio>
0012: #include <mutex>
0013: #include <set>
0014: #include <unordered_set>
0015: 
0016: // this implementation is based on CUDACachingAllocator.
0017: // It utilizes Metal Heaps to improve the performance with buffer allocation.
0018: // Do not include this header. Use MPSAllocatorInterface.h instead.
0019: // TODO: Unify the logic with CUDACachingAllocator and remove redundant code.
0020: namespace at::mps::HeapAllocator {
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions; manages memory allocation, buffers, or ownership boundaries. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象；管理内存分配、缓冲区或所有权边界。关键符号：无明显局部符号。

### Lines 21-41 / 第 21-41 行

```cpp
0021: 
0022: static const size_t kMaxSmallAlloc = MB(1); // largest "small" allocation is 1 MiB
0023: static const size_t kMinLargeAlloc = MB(10); // allocations between 1 and 10 MiB may use kLargeHeap
0024: static const size_t kRoundLarge = MB(2); // round up large allocations to 2 MiB
0025: static const size_t kSmallHeap = MB(8); // "small" allocations are packed in 8 MiB heaps
0026: static const size_t kLargeHeap = MB(32); // "large" allocations may be packed in 32 MiB heaps
0027: static const size_t kXLargeHeapD =
0028:     MB(128); // "extra large" allocations on Discrete devices may be packed in 128 MiB heaps
0029: static const size_t kXLargeHeapU =
0030:     MB(1024); // "extra large" allocations on Unified devices may be packed in 1 GiB heaps
0031: static const size_t kMaxScalarAlloc = (sizeof(int64_t)); // largest "scalar" allocation
0032: 
0033: // buffer pools could be customized with a combination of usage flags
0034: enum UsageFlags : uint32_t {
0035:   PRIVATE = 0,
0036:   SMALL = (1 << 0), // small heaps have sizes of kSmallHeap, and large ones kLargeHeap
0037:   SHARED = (1 << 1), // shared pools allocated on devices with unified memory; otherwise, private between host/device
0038:   MANAGED = (1 << 2), // managed storage mode
0039:   HAZARD = (1 << 3), // enables Automatic Hazard Tracking for the resources allocated on the pool
0040:   SCALAR = (1 << 4), // used to import CPU scalar values to GPU and use them in MPS Stream
0041: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `UsageFlags`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`UsageFlags`。

### Lines 42-68 / 第 42-68 行

```cpp
0042: // debug verbosity flags
0043: enum DebugVerbosity : uint32_t {
0044:   SILENT = 0,
0045:   PROFILING = (1 << 0), // print generic profiling data for total system memory usage
0046:   ALLOCATIONS = (1 << 1), // print buffer allocations
0047:   RECYCLES = (1 << 2), // print buffer recycling
0048:   RELEASES = (1 << 3), // print buffer releases
0049:   LARGE_ONLY = (1 << 4), // only log large buffer pool transactions
0050: };
0051: 
0052: struct HeapBlock;
0053: 
0054: struct BufferBlock {
0055:   id<MTLBuffer> buffer;
0056:   void* cpu_ptr = nullptr; // stores the pointer to CPU mapping of a Shared MTLBuffer
0057:   size_t size; // size after alignment
0058:   size_t requested_size; // requested size (before alignment)
0059:   // buffer shape is used for retrieving base of views in cached graphs
0060:   std::vector<int64_t> shape;
0061:   bool in_use = false;
0062:   HeapBlock* heap;
0063:   id_t buf_id;
0064:   // counter to candidate least recently used buffers for garbage collection
0065:   uint32_t gc_count = 0;
0066:   uint32_t use_count = 0;
0067:   // counter to assign unique ids to buffer blocks
0068:   static uint64_t buffer_counter;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `HeapBlock`, `BufferBlock`, `DebugVerbosity`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`HeapBlock`, `BufferBlock`, `DebugVerbosity`。

### Lines 69-88 / 第 69-88 行

```cpp
0069:   // Metal events used to sync GPU/CPU operations on the shared-storage buffers
0070:   MPSEventPtr event;
0071: 
0072:   BufferBlock(size_t Size, size_t RequestedSize = 0, const id<MTLBuffer> Buffer = nullptr, HeapBlock* Heap = nullptr)
0073:       : buffer(Buffer), size(Size), requested_size(RequestedSize), heap(Heap), buf_id(Buffer ? ++buffer_counter : 0) {}
0074: 
0075:   static bool Comparator(const BufferBlock* a, const BufferBlock* b) {
0076:     return (a->size != b->size) ? a->size < b->size : (uintptr_t)a->buffer < (uintptr_t)b->buffer;
0077:   }
0078:   static size_t alignUp(size_t Size, size_t Alignment) {
0079:     assert(((Alignment - 1) & Alignment) == 0);
0080:     return ((Size + Alignment - 1) & ~(Alignment - 1));
0081:   }
0082:   uint32_t retainCount() const {
0083:     return [buffer retainCount];
0084:   }
0085: };
0086: typedef bool (*BufferComparison)(const BufferBlock*, const BufferBlock*);
0087: 
0088: struct BufferPool;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `BufferPool`, `BufferBlock`, `Comparator`, `alignUp`, `assert`, `retainCount`, `bool`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`BufferPool`, `BufferBlock`, `Comparator`, `alignUp`, `assert`, `retainCount`, `bool`。

### Lines 89-109 / 第 89-109 行

```cpp
0089: struct AllocParams {
0090:   AllocParams(size_t Alloc_Size, size_t Requested_Size, BufferPool* Pool)
0091:       : search_key(Alloc_Size), pool(Pool), requested_size(Requested_Size) {}
0092:   size_t size() const {
0093:     return search_key.size;
0094:   }
0095: 
0096:   BufferBlock search_key;
0097:   BufferPool* pool;
0098:   BufferBlock* buffer_block = nullptr;
0099:   size_t requested_size;
0100:   // true if we exceed the low watermark limit. In this case
0101:   // we apply strategies to relieve the pressure before allocation.
0102:   bool has_memory_pressure = false;
0103:   // true if we're allocating on a unified memory device
0104:   bool has_unified_memory = true;
0105: };
0106: 
0107: struct HeapBlock {
0108:   id<MTLHeap> heap;
0109:   struct {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `AllocParams`, `HeapBlock`, `size`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`AllocParams`, `HeapBlock`, `size`。

### Lines 110-130 / 第 110-130 行

```cpp
0110:     size_t total, available;
0111:   } size;
0112:   BufferPool* pool;
0113:   unsigned int n_buffers = 0;
0114:   id_t heap_id;
0115:   // indicates if we split this heap to sub-allocate 'several' buffers (otherwise single buffer)
0116:   bool is_split;
0117:   // counter to assign unique ids to heap blocks
0118:   static uint64_t heap_counter;
0119: 
0120:   HeapBlock(size_t Size, const id<MTLHeap> Heap = nullptr, BufferPool* Pool = nullptr)
0121:       : heap(Heap),
0122:         size({.total = Size, .available = Size}),
0123:         pool(Pool),
0124:         heap_id(Heap ? ++heap_counter : 0),
0125:         is_split(true) {}
0126: 
0127:   static MTLResourceOptions getOptions(uint32_t usage) {
0128:     // TODO: check the caching performance of write-combined mode
0129:     MTLResourceOptions options = MTLResourceCPUCacheModeDefaultCache;
0130: 
```

- **EN:** This block manages memory allocation, buffers, or ownership boundaries. Key symbols: `pool`, `getOptions`.
- **CN:** 该代码块管理内存分配、缓冲区或所有权边界。关键符号：`pool`, `getOptions`。

### Lines 131-151 / 第 131-151 行

```cpp
0131:     if (usage & UsageFlags::MANAGED)
0132:       options |= MTLResourceStorageModeManaged;
0133:     else if (usage & UsageFlags::SHARED)
0134:       options |= MTLResourceStorageModeShared;
0135:     else
0136:       options |= MTLResourceStorageModePrivate;
0137: 
0138:     options |=
0139:         (usage & UsageFlags::HAZARD) ? MTLResourceHazardTrackingModeTracked : MTLResourceHazardTrackingModeUntracked;
0140: 
0141:     return options;
0142:   }
0143: 
0144:   static HeapBlock* createHeapBlock(AllocParams& params, id<MTLDevice> device, uint32_t usage) {
0145:     HeapBlock* heapBlock = nullptr;
0146:     bool is_split = true;
0147:     const size_t size = params.size();
0148:     MTLHeapDescriptor* d = [MTLHeapDescriptor new];
0149:     if (d) {
0150:       const size_t kXLargeHeap = params.has_unified_memory ? kXLargeHeapU : kXLargeHeapD;
0151:       if (size <= kMaxSmallAlloc) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions. Key symbols: `createHeapBlock`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`createHeapBlock`。

### Lines 152-174 / 第 152-174 行

```cpp
0152:         d.size = kSmallHeap;
0153:       } else if (size < kMinLargeAlloc) {
0154:         d.size = kLargeHeap;
0155:       } else if (size < kXLargeHeap / 2 && !params.has_memory_pressure) {
0156:         d.size = kXLargeHeap;
0157:       } else {
0158:         d.size = kRoundLarge * ((size + kRoundLarge - 1) / kRoundLarge);
0159:         is_split = false;
0160:       }
0161:       d.storageMode = (usage & UsageFlags::SHARED) ? MTLStorageModeShared : MTLStorageModePrivate;
0162:       d.cpuCacheMode = MTLCPUCacheModeDefaultCache;
0163:       // this automatically handles Metal buffer access synchronizations at the
0164:       // cost of slightly lower performance.
0165:       d.hazardTrackingMode =
0166:           (usage & UsageFlags::HAZARD) ? MTLHazardTrackingModeTracked : MTLHazardTrackingModeUntracked;
0167:       d.resourceOptions = getOptions(usage);
0168:       d.type = MTLHeapTypeAutomatic;
0169:       id<MTLHeap> heap = [device newHeapWithDescriptor:d];
0170:       if (heap) {
0171:         [heap setPurgeableState:MTLPurgeableStateNonVolatile];
0172:         const size_t heap_size = heapAvailableSize(heap);
0173:         heapBlock = new HeapBlock(heap_size, heap, params.pool);
0174:         if (heapBlock) {
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; manages memory allocation, buffers, or ownership boundaries. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；管理内存分配、缓冲区或所有权边界。关键符号：无明显局部符号。

### Lines 175-194 / 第 175-194 行

```cpp
0175:           heapBlock->is_split = is_split;
0176:         }
0177:       }
0178:       [d release];
0179:     }
0180:     return heapBlock;
0181:   }
0182:   static bool Comparator(const HeapBlock* a, const HeapBlock* b) {
0183:     return (a->size.available != b->size.available) ? a->size.available < b->size.available
0184:                                                     : (uintptr_t)a->heap < (uintptr_t)b->heap;
0185:   }
0186:   static NSUInteger heapAvailableSize(id<MTLHeap> heap, size_t Alignment = vm_page_size) {
0187:     return [heap maxAvailableSizeWithAlignment:Alignment];
0188:   }
0189:   NSUInteger Size() {
0190:     return [heap size];
0191:   }
0192:   id<MTLBuffer> newMTLBuffer(size_t length, uint32_t usage) {
0193:     id<MTLBuffer> buf = [heap newBufferWithLength:length options:getOptions(usage)];
0194:     if (buf) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manages memory allocation, buffers, or ownership boundaries. Key symbols: `Comparator`, `heapAvailableSize`, `Size`, `newMTLBuffer`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；管理内存分配、缓冲区或所有权边界。关键符号：`Comparator`, `heapAvailableSize`, `Size`, `newMTLBuffer`。

### Lines 195-217 / 第 195-217 行

```cpp
0195:       updateAvailableSize();
0196:       n_buffers++;
0197:     }
0198:     return buf;
0199:   }
0200:   // returns the retainCount before releasing the buffer
0201:   uint32_t releaseMTLBuffer(id<MTLBuffer>& buffer) {
0202:     const uint32_t retainCount = [buffer retainCount];
0203:     [buffer release];
0204:     buffer = nil;
0205:     updateAvailableSize();
0206:     n_buffers--;
0207:     return retainCount;
0208:   }
0209:   // returns the retainCount before releasing the heap
0210:   uint32_t releaseMTLHeap() {
0211:     const uint32_t retainCount = [heap retainCount];
0212:     TORCH_INTERNAL_ASSERT(!n_buffers); // assert if heap isn't empty
0213:     [heap setPurgeableState:MTLPurgeableStateEmpty];
0214:     [heap release];
0215:     heap = nil;
0216:     size.available = 0;
0217:     return retainCount;
```

- **EN:** This block produces a result or forwards a computed value; manages memory allocation, buffers, or ownership boundaries; validates runtime invariants before continuing. Key symbols: `updateAvailableSize`, `releaseMTLBuffer`, `releaseMTLHeap`.
- **CN:** 该代码块返回结果或转发已计算的值；管理内存分配、缓冲区或所有权边界；在继续执行前校验运行时不变量。关键符号：`updateAvailableSize`, `releaseMTLBuffer`, `releaseMTLHeap`。

### Lines 218-237 / 第 218-237 行

```cpp
0218:   }
0219:   uint32_t retainCount() const {
0220:     return [heap retainCount];
0221:   }
0222:   void updateAvailableSize() {
0223:     size.available = heapAvailableSize(heap);
0224:   }
0225: };
0226: typedef bool (*HeapComparison)(const HeapBlock*, const HeapBlock*);
0227: 
0228: struct BufferPool {
0229:   enum class Kind {
0230:     SHARED_SMALL,
0231:     SHARED_LARGE,
0232:     SCALAR,
0233:   };
0234: 
0235:   BufferPool(const id<MTLDevice> Device, uint32_t Usage)
0236:       : device(Device), usage(Usage), heaps(HeapBlock::Comparator), available_buffers(BufferBlock::Comparator) {}
0237: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `BufferPool`, `Kind`, `retainCount`, `updateAvailableSize`, `bool`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`BufferPool`, `Kind`, `retainCount`, `updateAvailableSize`, `bool`。

### Lines 238-260 / 第 238-260 行

```cpp
0238:   const id<MTLDevice> device;
0239:   // usage flags to customize the pool for various purposes (see UsageFlags enum)
0240:   const uint32_t usage;
0241:   // total number of buffers in the pool
0242:   uint32_t n_buffers = 0;
0243:   // total allocations size on this pool
0244:   size_t allocated_size = 0;
0245:   // total memory available in the pool
0246:   size_t available_size = 0;
0247:   // list of heaps ordered by their "available" (not total) memory size
0248:   std::set<HeapBlock*, HeapComparison> heaps;
0249:   // list of only "available" buffers in the pool (i.e., buffers not in-use)
0250:   std::set<BufferBlock*, BufferComparison> available_buffers;
0251:   // list of buffers that are in a state of "limbo" where they've already been freed
0252:   // from PyTorch-side, but were not returned to pool due to still being
0253:   // in-use by command buffers with retainCount > 1. In this state, the buffer is
0254:   // neither ready to be recycled, nor could be returned to pool as available.
0255:   // These buffers will be returned to pool once the command buffer's
0256:   // completionHandler callbacks are called.
0257:   std::unordered_set<BufferBlock*> buffers_pending_free;
0258:   // list of heaps pending size update
0259:   std::unordered_set<HeapBlock*> heaps_pending_update;
0260: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: no prominent local symbols.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：无明显局部符号。

### Lines 261-300 / 第 261-300 行

```cpp
0261: 
0262: class MPSHeapAllocatorImpl {
0263:  public:
0264:   explicit MPSHeapAllocatorImpl()
0265:       : m_device(at::mps::MPSDevice::getInstance()->device()),
0266:         m_max_buffer_size([m_device maxBufferLength]),
0267:         m_stream(getDefaultMPSStream()),
0268:         m_event_pool(getMPSEventPool()) {
0269:     init_allocator();
0270:   }
0271:   ~MPSHeapAllocatorImpl() {
0272:     emptyCache();
0273:   }
0274:   // interface exposed to at::Allocator
0275:   id<MTLBuffer> malloc(size_t size, uint32_t usage);
0276:   // frees a buffer and returns it into buffer pool
0277:   void free(void* ptr);
0278:   // releases all the cached buffers and their associated heaps
0279:   void emptyCache();
0280:   // free inactive buffers that are pending to be freed
0281:   void freeInactiveBuffers();
0282:   // returns true if buffer was allocated from the shared pool
0283:   bool isSharedBuffer(const void* ptr);
0284:   // get the requested unaligned size of an MTLBuffer
0285:   ssize_t getUnalignedBufferSize(const void* ptr);
0286:   // set the shape of a base tensor from a view tensor
0287:   void setBufferShape(const void* ptr, const IntArrayRef& shape);
0288:   // retrieve the shape of a base tensor from a view tensor
0289:   IntArrayRef getBufferShape(const void* ptr);
0290:   // get the unique ID of the buffer
0291:   id_t getBufferId(const void* ptr);
0292:   // allocate a buffer from a specialized pool to import CPU scalars into GPU
0293:   id<MTLBuffer> allocScalarBufferWithValue(void* value, size_t size);
0294:   // returns a CPU-mapping of the input buffer and its retainCount,
0295:   // if only it has Shared storage-mode and allocated on MPSAllocator
0296:   std::pair<const void*, uint32_t> getSharedBufferPtr(const void* buffer);
0297:   // records events for a list of MTLBuffers (list is used to lock the mutex once)
0298:   // returns true if records any event (given if passed buffers exist and are shared-storage)
0299:   bool recordEvents(c10::ArrayRef<const void*> buffers);
0300:   // waits for the event to signal the completion of GPU execution
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `MPSHeapAllocatorImpl`, `init_allocator`, `~MPSHeapAllocatorImpl`, `emptyCache`, `malloc`, `free`, `freeInactiveBuffers`, `isSharedBuffer`, `...`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`MPSHeapAllocatorImpl`, `init_allocator`, `~MPSHeapAllocatorImpl`, `emptyCache`, `malloc`, `free`, `freeInactiveBuffers`, `isSharedBuffer`, `...`。

### Lines 301-322 / 第 301-322 行

```cpp
0301:   // on the passed shared buffers (list is used to lock the mutex once)
0302:   // returns true if actually waited on any event
0303:   bool waitForEvents(c10::ArrayRef<const void*> buffers);
0304:   // this indicates how far (in Megabytes) the current total allocations are from the
0305:   // low watermark limit which is used to detect if we're under memory pressure
0306:   // This returns zero if we've reached the low watermark limit
0307:   ssize_t getLowWatermarkValue();
0308:   // (see m_low_watermark_ratio for description)
0309:   void setLowWatermarkRatio(double ratio);
0310:   // (see m_high_watermark_ratio for description)
0311:   void setHighWatermarkRatio(double ratio);
0312:   // (see m_low_watermark_limit for description)
0313:   size_t getLowWatermarkLimit() const {
0314:     return m_low_watermark_limit;
0315:   }
0316:   // (see m_max_total_allowed_size for description)
0317:   size_t getHighWatermarkLimit() const {
0318:     return m_max_total_allowed_size;
0319:   }
0320:   // (see m_total_allocated_memory for description)
0321:   size_t getTotalAllocatedMemory() const {
0322:     return m_total_allocated_memory;
```

- **EN:** This block produces a result or forwards a computed value; manages memory allocation, buffers, or ownership boundaries. Key symbols: `waitForEvents`, `getLowWatermarkValue`, `setLowWatermarkRatio`, `setHighWatermarkRatio`, `getLowWatermarkLimit`, `getHighWatermarkLimit`, `getTotalAllocatedMemory`.
- **CN:** 该代码块返回结果或转发已计算的值；管理内存分配、缓冲区或所有权边界。关键符号：`waitForEvents`, `getLowWatermarkValue`, `setLowWatermarkRatio`, `setHighWatermarkRatio`, `getLowWatermarkLimit`, `getHighWatermarkLimit`, `getTotalAllocatedMemory`。

### Lines 323-342 / 第 323-342 行

```cpp
0323:   }
0324:   // (see m_current_allocated_memory for description)
0325:   size_t getCurrentAllocatedMemory() const {
0326:     return m_current_allocated_memory;
0327:   }
0328:   // total GPU memory allocated in the process by Metal driver; including
0329:   // implicit allocations from MPS/MPSGraph frameworks and MPSHeapAllocatorImpl.
0330:   size_t getDriverAllocatedMemory() const {
0331:     return current_allocated_size();
0332:   }
0333:   // recommended Max memory for Metal
0334:   size_t getRecommendedMaxMemory() const {
0335:     return max_device_size();
0336:   }
0337:   // (see enum DebugVerbosity for description)
0338:   uint32_t getDebugVerbosity() const {
0339:     return m_debug_verbosity;
0340:   }
0341:   // returns the device that we allocate from
0342:   inline id<MTLDevice> Device() const {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `getCurrentAllocatedMemory`, `getDriverAllocatedMemory`, `current_allocated_size`, `getRecommendedMaxMemory`, `max_device_size`, `getDebugVerbosity`, `Device`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`getCurrentAllocatedMemory`, `getDriverAllocatedMemory`, `current_allocated_size`, `getRecommendedMaxMemory`, `max_device_size`, `getDebugVerbosity`, `Device`。

### Lines 343-382 / 第 343-382 行

```cpp
0343:     return m_device;
0344:   }
0345: 
0346:   inline std::string format_size(uint64_t size) const;
0347: 
0348:  private:
0349:   // (see m_high_watermark_ratio for description)
0350:   constexpr static double default_high_watermark_ratio = 1.7;
0351:   // we set the allowed upper bound to twice the size of recommendedMaxWorkingSetSize.
0352:   constexpr static double default_high_watermark_upper_bound = 2.0;
0353:   // (see m_low_watermark_ratio for description)
0354:   // on unified memory, we could allocate beyond the recommendedMaxWorkingSetSize
0355:   constexpr static double default_low_watermark_ratio_unified = 1.4;
0356:   constexpr static double default_low_watermark_ratio_discrete = 1.0;
0357: 
0358:   const id<MTLDevice> m_device;
0359:   std::recursive_mutex m_mutex;
0360:   // allocated buffers by device pointer
0361:   ska::flat_hash_map<const void*, BufferBlock*> m_allocated_buffers;
0362:   // using a container for pools to simplify iterating them
0363:   ska::flat_hash_map<BufferPool::Kind, std::unique_ptr<BufferPool>> m_pools;
0364:   // total memory allocated by HeapAllocator (including blocks in pools)
0365:   size_t m_total_allocated_memory = 0;
0366:   // currently active memory allocations in use (i.e., blocks not in pools)
0367:   size_t m_current_allocated_memory = 0;
0368:   // max buffer size allowed by Metal
0369:   size_t m_max_buffer_size = 0;
0370:   // maximum total size allowed to be allocated
0371:   size_t m_max_total_allowed_size = 0;
0372:   // high watermark ratio is a hard limit for the total allowed allocations
0373:   // 0. : disables high watermark limit (may cause system failure if system-wide OOM occurs)
0374:   // 1. : recommended maximum allocation size (i.e., device.recommendedMaxWorkingSetSize)
0375:   // >1.: allows limits beyond the device.recommendedMaxWorkingSetSize
0376:   // e.g., value 0.95 means we allocate up to 95% of recommended maximum
0377:   // allocation size; beyond that, the allocations would fail with OOM error.
0378:   double m_high_watermark_ratio;
0379:   // low watermark ratio is a soft limit to attempt limiting memory allocations up to the lower watermark
0380:   // level by garbage collection or committing command buffers more frequently (a.k.a, adaptive commit).
0381:   // Value between 0 to m_high_watermark_ratio (setting 0.0 disables adaptive commit and garbage collection)
0382:   // e.g., value 0.9 means we 'attempt' to limit allocations up to 90% of recommended maximum
```

- **EN:** This block produces a result or forwards a computed value; connects ATen logic to backend library/resource abstractions; manages memory allocation, buffers, or ownership boundaries. Key symbols: `format_size`.
- **CN:** 该代码块返回结果或转发已计算的值；把 ATen 逻辑连接到后端库或资源抽象；管理内存分配、缓冲区或所有权边界。关键符号：`format_size`。

### Lines 383-418 / 第 383-418 行

```cpp
0383:   // allocation size.
0384:   double m_low_watermark_ratio;
0385:   // low watermark size limit (in Bytes) at the time we initialize the allocator
0386:   size_t m_low_watermark_limit;
0387:   // use "PYTORCH_DEBUG_MPS_ALLOCATOR" env-var to set debug verbosity
0388:   uint32_t m_debug_verbosity;
0389:   // default MPS stream
0390:   MPSStream* m_stream;
0391:   // we hold a reference to MPSEventPool so it could get destroyed after MPSAllocator
0392:   std::shared_ptr<MPSEventPool> m_event_pool;
0393: 
0394:   void init_allocator();
0395:   void init_buffer_pools();
0396:   HeapBlock* get_free_heap(AllocParams& params);
0397:   bool get_free_buffer(AllocParams& params);
0398:   BufferBlock* get_allocated_buffer_block(const void* ptr);
0399:   BufferBlock* alloc_buffer_block(size_t size, uint32_t usage);
0400:   bool alloc_buffer(AllocParams& params);
0401:   void free_buffer(BufferBlock* buffer_block);
0402:   // returns true if the container heap is also released
0403:   bool release_buffer(BufferBlock* buffer_block, bool remove_empty_heap = true);
0404:   void release_buffers(BufferPool& pool);
0405:   bool release_available_cached_buffers(AllocParams& params);
0406:   bool release_cached_buffers();
0407:   // free unused cached blocks to reclaim GPU memory if memory pressure is high
0408:   void garbage_collect_cached_buffers(AllocParams& params);
0409:   // returns the suitable buffer pool type for the usage or
0410:   // requested/allocated sizes
0411:   BufferPool& get_pool(size_t requested_size, size_t aligned_size, uint32_t usage);
0412:   // returns the aligned allocation size that is optimized
0413:   // for the buffers to get reused frequently
0414:   size_t get_allocation_size(size_t size, uint32_t usage) const;
0415:   // maximum size of device memory available for allocation in current process
0416:   // Note: the recommendedMaxWorkingSetSize is typically 75% of the total system memory.
0417:   size_t max_device_size() const {
0418:     return [m_device recommendedMaxWorkingSetSize];
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions; manages memory allocation, buffers, or ownership boundaries. Key symbols: `init_allocator`, `init_buffer_pools`, `get_free_heap`, `get_free_buffer`, `get_allocated_buffer_block`, `alloc_buffer_block`, `alloc_buffer`, `free_buffer`, `...`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象；管理内存分配、缓冲区或所有权边界。关键符号：`init_allocator`, `init_buffer_pools`, `get_free_heap`, `get_free_buffer`, `get_allocated_buffer_block`, `alloc_buffer_block`, `alloc_buffer`, `free_buffer`, `...`。

### Lines 419-435 / 第 419-435 行

```cpp
0419:   }
0420:   // there are implicit allocations from MPS backend, so we need to query the 'device' for
0421:   // total allocated size instead of manually tracking in MPSAllocator
0422:   size_t current_allocated_size() const {
0423:     return [m_device currentAllocatedSize];
0424:   }
0425: 
0426:   bool trigger_memory_callbacks(BufferBlock* buffer_block, IMpsAllocatorCallback::EventType event) const {
0427:     for (const auto& name : MPSAllocatorCallbacksRegistry()->Keys()) {
0428:       MPSAllocatorCallbacksRegistry()->Create(name)->executeMPSAllocatorCallback(
0429:           buffer_block ? buffer_block->buffer : nullptr, event);
0430:     }
0431:     return true;
0432:   }
0433: };
0434: 
0435: } // namespace at::mps::HeapAllocator
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; connects ATen logic to backend library/resource abstractions; manages memory allocation, buffers, or ownership boundaries. Key symbols: `current_allocated_size`, `trigger_memory_callbacks`, `MPSAllocatorCallbacksRegistry`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；把 ATen 逻辑连接到后端库或资源抽象；管理内存分配、缓冲区或所有权边界。关键符号：`current_allocated_size`, `trigger_memory_callbacks`, `MPSAllocatorCallbacksRegistry`。


## Key Concepts / 关键概念
- **Apple MPS support** — Apple MPS 支持
- **Tensor metadata and value flow** — 张量元数据与数值流
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Apple accelerator integration** — Apple 加速后端集成
- **Memory allocation strategy** — 内存分配策略
- **Device and stream coordination** — 设备与流协调
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: HeapBlock, BufferBlock, BufferPool, AllocParams, Kind, MPSHeapAllocatorImpl, UsageFlags, DebugVerbosity** — 核心符号：HeapBlock、BufferBlock、BufferPool、AllocParams、Kind、MPSHeapAllocatorImpl、UsageFlags、DebugVerbosity

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/mps/MPSAllocatorInterface.h`, `ATen/mps/MPSEvent.h`, `ATen/mps/MPSStream.h`, `c10/util/flat_hash_map.h`
- **External includes / 外部头文件**: `mach/vm_page_size.h`, `cstdio`, `mutex`, `set`, `unordered_set`
- **Namespaces / 命名空间**: `at::mps::HeapAllocator`
- **Representative symbols / 代表性符号**: `HeapBlock`, `BufferBlock`, `BufferPool`, `AllocParams`, `Kind`, `MPSHeapAllocatorImpl`, `UsageFlags`, `DebugVerbosity`, `Comparator`, `alignUp`, `assert`, `retainCount`, `...`
