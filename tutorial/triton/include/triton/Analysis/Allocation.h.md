# Allocation.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Analysis/Allocation.h`
- **EN:** Declares analysis helpers centered on `Allocation`.
- **CN:** 声明围绕 `Allocation` 的分析辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_ANALYSIS_ALLOCATION_H
   2: #define TRITON_ANALYSIS_ALLOCATION_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-8
```cpp
   4: #include "triton/Analysis/Utility.h"
   5: #include "triton/Tools/GenericSwizzling.h"
   6: #include "llvm/ADT/DenseMap.h"
   7: #include "llvm/ADT/MapVector.h"
   8: #include "llvm/ADT/SetVector.h"
```
**EN:** This block imports the direct dependencies needed here, including triton/Analysis/Utility.h, triton/Tools/GenericSwizzling.h, llvm/ADT/DenseMap.h, llvm/ADT/MapVector.h, and llvm/ADT/SetVector.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 triton/Analysis/Utility.h, triton/Tools/GenericSwizzling.h, llvm/ADT/DenseMap.h, llvm/ADT/MapVector.h, and llvm/ADT/SetVector.h。

### Lines 10-10
```cpp
  10: #include <limits>
```
**EN:** This block imports the direct dependencies needed here, including <limits>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 <limits>。

### Lines 12-12
```cpp
  12: namespace mlir {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir 下。

### Lines 14-15
```cpp
  14: namespace triton {
  15: class AllocationAnalysis;
```
**EN:** This block stores supporting state such as AllocationAnalysis, which other APIs in the file consume.
**CN:** 该代码块声明了 AllocationAnalysis 等支撑状态，供本文件中的其他 API 使用。

### Lines 17-19
```cpp
  17: /// Callback to allow backends to specify target-specific scratch sizes for
  18: /// some operations.
  19: using AllocationAnalysisScratchSizeFn = std::function<unsigned(Operation *)>;
```
**EN:** This block declares or defines callable APIs such as unsigned, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 unsigned 等可调用 API，用来封装这里提供的核心行为。

### Lines 21-21
```cpp
  21: unsigned defaultAllocationAnalysisScratchSizeFn(Operation *op);
```
**EN:** This block declares or defines callable APIs such as defaultAllocationAnalysisScratchSizeFn, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 defaultAllocationAnalysisScratchSizeFn 等可调用 API，用来封装这里提供的核心行为。

### Lines 23-27
```cpp
  23: unsigned getNumScratchElemsSwizzledCvt(const LinearLayout &srcLayout,
  24:                                        const LinearLayout &dstLayout,
  25:                                        int bitwidth, int numBanks = 32,
  26:                                        gpu::LocalMemOpTile srcTile = {},
  27:                                        gpu::LocalMemOpTile dstTile = {});
```
**EN:** This block declares or defines callable APIs such as getNumScratchElemsSwizzledCvt, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumScratchElemsSwizzledCvt 等可调用 API，用来封装这里提供的核心行为。

### Lines 29-33
```cpp
  29: unsigned getNumScratchElemsSwizzledCvt(RankedTensorType srcTy,
  30:                                        RankedTensorType dstTy,
  31:                                        int numBanks = 32,
  32:                                        gpu::LocalMemOpTile srcTile = {},
  33:                                        gpu::LocalMemOpTile dstTile = {});
```
**EN:** This block declares or defines callable APIs such as getNumScratchElemsSwizzledCvt, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumScratchElemsSwizzledCvt 等可调用 API，用来封装这里提供的核心行为。

### Lines 35-35
```cpp
  35: } // namespace triton
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 37-57
```cpp
  37: /// Modified from llvm-15.0: llvm/ADT/AddressRanges.h
  38: /// A class that represents an interval, specified using a start and an end
  39: /// values: [Start, End).
  40: template <typename T> class Interval {
  41: public:
  42:   Interval() {}
  43:   Interval(T S, T E) : Start(S), End(E) { assert(Start <= End); }
  44:   T start() const { return Start; }
  45:   T end() const { return End; }
  46:   T size() const { return End - Start; }
  47:   bool contains(T Addr) const { return Start <= Addr && Addr < End; }
  48:   bool intersects(const Interval &R) const {
  49:     return Start < R.End && R.Start < End;
  50:   }
  51:   bool operator==(const Interval &R) const {
  52:     return Start == R.Start && End == R.End;
  53:   }
  54:   bool operator!=(const Interval &R) const { return !(*this == R); }
  55:   bool operator<(const Interval &R) const {
  56:     return std::make_pair(Start, End) < std::make_pair(R.Start, R.End);
  57:   }
```
**EN:** This block introduces `Interval`, the main class/struct defined here. Within the declaration, methods such as Start, End, start, end, size, and contains expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `Interval`。 其中 Start, End, start, end, size, and contains 等方法构成了它的主要接口。

### Lines 59-62
```cpp
  59: private:
  60:   T Start = std::numeric_limits<T>::min();
  61:   T End = std::numeric_limits<T>::max();
  62: };
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 64-64
```cpp
  64: template <class T> Interval(T, T) -> Interval<T>;
```
**EN:** This block declares or defines callable APIs such as Interval, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 Interval 等可调用 API，用来封装这里提供的核心行为。

### Lines 66-71
```cpp
  66: class Allocation {
  67: public:
  68:   /// A unique identifier for shared memory buffers
  69:   using BufferId = size_t;
  70:   using BufferIdSetT = DenseSet<BufferId>;
  71:   using FuncAllocMapT = triton::CallGraph<Allocation>::FuncDataMapT;
```
**EN:** This block introduces `Allocation`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `Allocation`。

### Lines 73-74
```cpp
  73:   static constexpr BufferId InvalidBufferId =
  74:       std::numeric_limits<BufferId>::max();
```
**EN:** This block defines named compile-time constants such as BufferId, InvalidBufferId, std, numeric_limits, BufferId, and max.
**CN:** 该代码块定义了 BufferId, InvalidBufferId, std, numeric_limits, BufferId, and max 等具名编译期常量。

### Lines 76-79
```cpp
  76:   Allocation() = default;
  77:   /// Creates a new Allocation analysis that computes the shared memory
  78:   /// information for all associated shared memory values.
  79:   explicit Allocation(Operation *operation) : operation(operation) {}
```
**EN:** This block declares or defines callable APIs such as Allocation and operation, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 Allocation and operation 等可调用 API，用来封装这里提供的核心行为。

### Lines 81-86
```cpp
  81:   /// Runs allocation analysis on the given top-level operation.
  82:   /// \param sharedMemoryPartitionSize The size of each shared memory partition
  83:   ///        in bytes. A value of 0 means shared memory is not partitioned.
  84:   void run(FuncAllocMapT &funcAllocMap,
  85:            triton::AllocationAnalysisScratchSizeFn scratchSizeGetter,
  86:            size_t sharedMemoryPartitionSize = 0);
```
**EN:** This block declares or defines callable APIs such as run, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 run 等可调用 API，用来封装这里提供的核心行为。

### Lines 88-89
```cpp
  88:   /// Returns the operation this analysis was constructed from.
  89:   Operation *getOperation() const { return operation; }
```
**EN:** This block declares or defines callable APIs such as getOperation, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOperation 等可调用 API，用来封装这里提供的核心行为。

### Lines 91-94
```cpp
  91:   /// Returns the offset of the given buffer in the shared memory.
  92:   size_t getOffset(BufferId bufferId) const {
  93:     return bufferSet.at(bufferId).offset;
  94:   }
```
**EN:** This block declares or defines callable APIs such as getOffset and at, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOffset and at 等可调用 API，用来封装这里提供的核心行为。

### Lines 96-99
```cpp
  96:   /// Returns the size of the given buffer in the shared memory.
  97:   size_t getAllocatedSize(BufferId bufferId) const {
  98:     return bufferSet.at(bufferId).size;
  99:   }
```
**EN:** This block declares or defines callable APIs such as getAllocatedSize and at, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getAllocatedSize and at 等可调用 API，用来封装这里提供的核心行为。

### Lines 101-105
```cpp
 101:   /// Returns the allocated interval of the given buffer.
 102:   Interval<size_t> getAllocatedInterval(BufferId bufferId) const {
 103:     auto &buffer = bufferSet.at(bufferId);
 104:     return Interval<size_t>(buffer.offset, buffer.offset + buffer.size);
 105:   }
```
**EN:** This block declares or defines callable APIs such as getAllocatedInterval and at, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getAllocatedInterval and at 等可调用 API，用来封装这里提供的核心行为。

### Lines 107-115
```cpp
 107:   /// Returns all buffer ids for a value.
 108:   /// For partitioned tensors, returns all logical piece buffer ids.
 109:   /// For non-partitioned values, returns a single-element vector.
 110:   /// Returns empty vector if value has no associated buffer.
 111:   SmallVector<BufferId> getBufferIds(Value value) const {
 112:     SmallVector<BufferId> bufferIds;
 113:     auto it = valueBuffer.find(value);
 114:     if (it == valueBuffer.end())
 115:       return bufferIds;
```
**EN:** This block declares or defines callable APIs such as getBufferIds, find, and end, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBufferIds, find, and end 等可调用 API，用来封装这里提供的核心行为。

### Lines 117-121
```cpp
 117:     for (auto *buffer : it->second) {
 118:       bufferIds.push_back(buffer->id);
 119:     }
 120:     return bufferIds;
 121:   }
```
**EN:** This block declares or defines callable APIs such as push_back, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 push_back 等可调用 API，用来封装这里提供的核心行为。

### Lines 123-135
```cpp
 123:   /// Returns all buffer ids of the given value, including alias buffers.
 124:   /// This is a superset of getBufferIds that also includes aliased buffers.
 125:   BufferIdSetT getAllBufferIdsWithAliases(Value value) const {
 126:     BufferIdSetT bufferIds;
 127:     for (auto bufferId : getBufferIds(value)) {
 128:       bufferIds.insert(bufferId);
 129:     }
 130:     for (auto *buffer : aliasBuffer.lookup(value)) {
 131:       if (buffer->id != InvalidBufferId)
 132:         bufferIds.insert(buffer->id);
 133:     }
 134:     return bufferIds;
 135:   }
```
**EN:** This block declares or defines callable APIs such as getAllBufferIdsWithAliases, getBufferIds, insert, and lookup, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getAllBufferIdsWithAliases, getBufferIds, insert, and lookup 等可调用 API，用来封装这里提供的核心行为。

### Lines 137-146
```cpp
 137:   /// Returns the scratch buffer id of the given value.
 138:   BufferId getBufferId(Operation *operation) const {
 139:     if (opScratch.count(operation)) {
 140:       return opScratch.lookup(operation)->id;
 141:     } else if (opVirtual.count(operation)) {
 142:       return opVirtual.lookup(operation)->id;
 143:     } else {
 144:       return InvalidBufferId;
 145:     }
 146:   }
```
**EN:** This block declares or defines callable APIs such as getBufferId, count, and lookup, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBufferId, count, and lookup 等可调用 API，用来封装这里提供的核心行为。

### Lines 148-151
```cpp
 148:   /// Returns if the given buffer is a virtual buffer.
 149:   bool isVirtualBuffer(BufferId bufferId) const {
 150:     return bufferSet.at(bufferId).kind == BufferT::BufferKind::Virtual;
 151:   }
```
**EN:** This block declares or defines callable APIs such as isVirtualBuffer and at, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isVirtualBuffer and at 等可调用 API，用来封装这里提供的核心行为。

### Lines 153-156
```cpp
 153:   /// Returns if the given buffer is an explicit buffer.
 154:   bool isExplicitBuffer(BufferId bufferId) const {
 155:     return bufferSet.at(bufferId).kind == BufferT::BufferKind::Explicit;
 156:   }
```
**EN:** This block declares or defines callable APIs such as isExplicitBuffer and at, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isExplicitBuffer and at 等可调用 API，用来封装这里提供的核心行为。

### Lines 158-159
```cpp
 158:   /// Returns the size of total shared memory allocated
 159:   size_t getSharedMemorySize() const { return sharedMemorySize; }
```
**EN:** This block declares or defines callable APIs such as getSharedMemorySize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSharedMemorySize 等可调用 API，用来封装这里提供的核心行为。

### Lines 161-162
```cpp
 161:   /// Returns mapping from operation to list of live LDS buffers
 162:   std::map<Operation *, SmallVector<BufferId>> getLiveBuffers();
```
**EN:** This block declares or defines callable APIs such as getLiveBuffers, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getLiveBuffers 等可调用 API，用来封装这里提供的核心行为。

### Lines 164-170
```cpp
 164: private:
 165:   /// A class that represents a shared memory buffer
 166:   struct BufferT {
 167:     /// Explicit: ttg.local_alloc
 168:     /// Scratch: ttg.convert_layout
 169:     /// Virtual: triton.call
 170:     enum class BufferKind { Explicit, Scratch, Virtual };
```
**EN:** This block defines an enumeration covering values such as private, A, that, represents, a, and shared.
**CN:** 该代码块定义了一个枚举类型，覆盖了 private, A, that, represents, a, and shared 等取值。

### Lines 172-177
```cpp
 172:     BufferKind kind;
 173:     BufferId id;
 174:     Operation *owner;
 175:     size_t size;
 176:     size_t alignment;
 177:     size_t offset;
```
**EN:** This block stores supporting state such as kind, id, owner, size, alignment, and offset, which other APIs in the file consume.
**CN:** 该代码块声明了 kind, id, owner, size, alignment, and offset 等支撑状态，供本文件中的其他 API 使用。

### Lines 179-181
```cpp
 179:     /// For partitioned tensors: buffers that reside in different physical
 180:     /// partitions.
 181:     SmallVector<BufferT *> neighbors;
```
**EN:** This block stores supporting state such as neighbors, which other APIs in the file consume.
**CN:** 该代码块声明了 neighbors 等支撑状态，供本文件中的其他 API 使用。

### Lines 183-184
```cpp
 183:     bool operator==(const BufferT &other) const { return id == other.id; }
 184:     bool operator<(const BufferT &other) const { return id < other.id; }
```
**EN:** This block stores supporting state such as id, which other APIs in the file consume.
**CN:** 该代码块声明了 id 等支撑状态，供本文件中的其他 API 使用。

### Lines 186-189
```cpp
 186:     BufferT(BufferKind kind, BufferId id, Operation *owner, size_t size,
 187:             size_t alignment = 4, size_t offset = 0)
 188:         : kind(kind), id(id), owner(owner), size(size), alignment(alignment),
 189:           offset(offset) {}
```
**EN:** This block declares or defines callable APIs such as BufferT, kind, id, owner, size, alignment, and offset, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 BufferT, kind, id, owner, size, alignment, and offset 等可调用 API，用来封装这里提供的核心行为。

### Lines 191-194
```cpp
 191:     size_t setOffsetAligned(size_t newOffset) {
 192:       return offset = llvm::alignTo(newOffset, alignment);
 193:     }
 194:   };
```
**EN:** This block declares or defines callable APIs such as setOffsetAligned and alignTo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setOffsetAligned and alignTo 等可调用 API，用来封装这里提供的核心行为。

### Lines 196-203
```cpp
 196:   /// Op -> Scratch Buffer
 197:   using OpScratchMapT = llvm::MapVector<Operation *, BufferT *>;
 198:   /// Value -> Explicit Buffers (vector for partitioned tensors)
 199:   using ValueBufferMapT = llvm::MapVector<Value, SmallVector<BufferT *>>;
 200:   /// Value -> Alias Buffer
 201:   using AliasBufferMapT = llvm::MapVector<Value, llvm::SetVector<BufferT *>>;
 202:   /// BufferId -> Buffer
 203:   using BufferSetT = std::map<BufferId, BufferT>;
```
**EN:** This block introduces type aliases such as using, OpScratchMapT, llvm::MapVector<Operation, and BufferT to simplify later declarations.
**CN:** 该代码块引入了 using, OpScratchMapT, llvm::MapVector<Operation, and BufferT 等类型别名，以简化后续声明。

### Lines 205-219
```cpp
 205: private:
 206:   template <BufferT::BufferKind Kind, typename KeyType, typename... Args>
 207:   void addBuffer(KeyType &key, Args &&...args) {
 208:     BufferId nextId = bufferIdCounter++;
 209:     auto [it, inserted] = bufferSet.insert_or_assign(
 210:         nextId, BufferT(Kind, nextId, key, std::forward<Args>(args)...));
 211:     BufferT *buffer = &it->second;
 212:     if constexpr (Kind == BufferT::BufferKind::Explicit) {
 213:       valueBuffer[key].push_back(buffer);
 214:     } else if constexpr (Kind == BufferT::BufferKind::Virtual) {
 215:       opVirtual[key] = buffer;
 216:     } else {
 217:       opScratch[key] = buffer;
 218:     }
 219:   }
```
**EN:** This block declares or defines callable APIs such as addBuffer, insert_or_assign, BufferT, constexpr, and push_back, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addBuffer, insert_or_assign, BufferT, constexpr, and push_back 等可调用 API，用来封装这里提供的核心行为。

### Lines 221-229
```cpp
 221:   /// Create multiple buffers for partitions where all different partitions
 222:   /// are neighbors (must be placed in different physical shared memory slots).
 223:   ///
 224:   /// \param key The value that owns these buffers
 225:   /// \param numPartitions Number of partition buffers to create
 226:   /// \param partitionSize Size of each partition buffer in bytes
 227:   /// \param alignment Required alignment for each buffer
 228:   void addPartitionBuffers(Value key, unsigned numPartitions,
 229:                            size_t partitionSize, size_t alignment);
```
**EN:** This block declares or defines callable APIs such as addPartitionBuffers, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addPartitionBuffers 等可调用 API，用来封装这里提供的核心行为。

### Lines 231-235
```cpp
 231:   void addAlias(Value value, Value alloc) {
 232:     for (auto *buffer : valueBuffer[alloc]) {
 233:       aliasBuffer[value].insert(buffer);
 234:     }
 235:   }
```
**EN:** This block declares or defines callable APIs such as addAlias and insert, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addAlias and insert 等可调用 API，用来封装这里提供的核心行为。

### Lines 237-244
```cpp
 237: private:
 238:   Operation *operation = nullptr;
 239:   OpScratchMapT opScratch;
 240:   OpScratchMapT opVirtual;
 241:   ValueBufferMapT valueBuffer;
 242:   AliasBufferMapT aliasBuffer;
 243:   BufferSetT bufferSet;
 244:   size_t sharedMemorySize = 0;
```
**EN:** This block stores supporting state such as nullptr, opScratch, opVirtual, valueBuffer, aliasBuffer, and bufferSet, which other APIs in the file consume.
**CN:** 该代码块声明了 nullptr, opScratch, opVirtual, valueBuffer, aliasBuffer, and bufferSet 等支撑状态，供本文件中的其他 API 使用。

### Lines 246-246
```cpp
 246:   size_t bufferIdCounter = 0;
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 248-249
```cpp
 248:   friend class triton::AllocationAnalysis;
 249: };
```
**EN:** This block stores supporting state such as AllocationAnalysis, which other APIs in the file consume.
**CN:** 该代码块声明了 AllocationAnalysis 等支撑状态，供本文件中的其他 API 使用。

### Lines 251-259
```cpp
 251: /// Static analysis that computes the allocation of shared memory buffers
 252: /// of the entire call graph.
 253: /// The allocation is performed in a post-order walk of the call graph.
 254: /// Each call op is treated like convert_layout that allocates a scratch buffer.
 255: /// At each call, we compute the start offset of the scratch buffer and pass it
 256: /// as an argument to the callee.
 257: class ModuleAllocation : public triton::CallGraph<Allocation> {
 258: public:
 259:   using FuncOffsetMapT = DenseMap<FunctionOpInterface, Value>;
```
**EN:** This block introduces `ModuleAllocation`, the main class/struct defined here. It also inherits behavior from a base type.
**CN:** 该代码块引入了此文件的核心类/结构体 `ModuleAllocation`。 它还通过继承复用基类能力。

### Lines 261-276
```cpp
 261:   ModuleAllocation(ModuleOp moduleOp,
 262:                    triton::AllocationAnalysisScratchSizeFn scratchSizeGetter =
 263:                        triton::defaultAllocationAnalysisScratchSizeFn,
 264:                    size_t sharedMemoryPartitionSize = 0)
 265:       : triton::CallGraph<Allocation>(moduleOp) {
 266:     walk<WalkOrder::PreOrder, WalkOrder::PostOrder>(
 267:         // Pre-order edge walk callback
 268:         [](CallOpInterface callOp, FunctionOpInterface funcOp) {},
 269:         // Post-order node walk callback
 270:         [&](FunctionOpInterface funcOp) {
 271:           auto [iter, inserted] = funcMap.try_emplace(funcOp, funcOp);
 272:           if (inserted)
 273:             iter->second.run(funcMap, scratchSizeGetter,
 274:                              sharedMemoryPartitionSize);
 275:         });
 276:   }
```
**EN:** This block declares or defines callable APIs such as ModuleAllocation, try_emplace, and run, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ModuleAllocation, try_emplace, and run 等可调用 API，用来封装这里提供的核心行为。

### Lines 278-285
```cpp
 278:   size_t getSharedMemorySize() {
 279:     size_t size = 0;
 280:     for (auto funcOp : getRoots()) {
 281:       auto *alloc = getFuncData(funcOp);
 282:       size = std::max(size, alloc->getSharedMemorySize());
 283:     }
 284:     return size;
 285:   }
```
**EN:** This block declares or defines callable APIs such as getSharedMemorySize, getRoots, getFuncData, and max, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSharedMemorySize, getRoots, getFuncData, and max 等可调用 API，用来封装这里提供的核心行为。

### Lines 287-289
```cpp
 287:   size_t getSharedMemorySize(FunctionOpInterface funcOp) {
 288:     return getFuncData(funcOp)->getSharedMemorySize();
 289:   }
```
**EN:** This block declares or defines callable APIs such as getSharedMemorySize and getFuncData, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSharedMemorySize and getFuncData 等可调用 API，用来封装这里提供的核心行为。

### Lines 291-293
```cpp
 291:   void setFunctionSharedMemoryValue(FunctionOpInterface funcOp, Value value) {
 292:     sharedMemoryValue[funcOp] = value;
 293:   }
```
**EN:** This block declares or defines callable APIs such as setFunctionSharedMemoryValue, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setFunctionSharedMemoryValue 等可调用 API，用来封装这里提供的核心行为。

### Lines 295-297
```cpp
 295:   Value getFunctionSharedMemoryBase(FunctionOpInterface funcOp) {
 296:     return sharedMemoryValue[funcOp];
 297:   }
```
**EN:** This block declares or defines callable APIs such as getFunctionSharedMemoryBase, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getFunctionSharedMemoryBase 等可调用 API，用来封装这里提供的核心行为。

### Lines 299-301
```cpp
 299: private:
 300:   FuncOffsetMapT sharedMemoryValue;
 301: };
```
**EN:** This block stores supporting state such as sharedMemoryValue, which other APIs in the file consume.
**CN:** 该代码块声明了 sharedMemoryValue 等支撑状态，供本文件中的其他 API 使用。

### Lines 303-303
```cpp
 303: } // namespace mlir
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 305-305
```cpp
 305: #endif // TRITON_ANALYSIS_ALLOCATION_H
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** alias analysis  
  **CN:** 别名分析
- **EN:** shared-memory allocation  
  **CN:** 共享内存分配
- **EN:** buffer modeling  
  **CN:** 缓冲区建模
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** linear layout algebra  
  **CN:** 线性布局代数
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** types  
  **CN:** 类型

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `triton/Analysis/Utility.h`
  - `triton/Tools/GenericSwizzling.h`
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/MapVector.h`
  - `llvm/ADT/SetVector.h`
- **System or external includes / 系统或外部依赖:**
  - `<limits>`
