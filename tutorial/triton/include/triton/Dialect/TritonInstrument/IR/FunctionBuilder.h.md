# FunctionBuilder.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonInstrument/IR/FunctionBuilder.h`
- **EN:** Declares APIs centered on `FunctionBuilder` inside Triton.
- **CN:** 声明 Triton 中围绕 `FunctionBuilder` 的 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITONINSTRUMENT_FUNCTIONBUILDER_H
   2: #define TRITONINSTRUMENT_FUNCTIONBUILDER_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-4
```cpp
   4: #include "triton/Dialect/TritonInstrument/IR/Utility.h"
```
**EN:** This block imports the direct dependencies needed here, including triton/Dialect/TritonInstrument/IR/Utility.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 triton/Dialect/TritonInstrument/IR/Utility.h。

### Lines 6-7
```cpp
   6: #include <string>
   7: #include <variant>
```
**EN:** This block imports the direct dependencies needed here, including <string> and <variant>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 <string> and <variant>。

### Lines 9-10
```cpp
   9: #include "llvm/ADT/ArrayRef.h"
  10: #include "llvm/ADT/StringRef.h"
```
**EN:** This block imports the direct dependencies needed here, including llvm/ADT/ArrayRef.h and llvm/ADT/StringRef.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 llvm/ADT/ArrayRef.h and llvm/ADT/StringRef.h。

### Lines 12-19
```cpp
  12: namespace mlir {
  13: class ImplicitLocOpBuilder;
  14: class ModuleOp;
  15: class Operation;
  16: class RankedTensorType;
  17: class Type;
  18: class Value;
  19: } // namespace mlir
```
**EN:** This block stores supporting state such as ImplicitLocOpBuilder, ModuleOp, Operation, RankedTensorType, Type, and Value, which other APIs in the file consume.
**CN:** 该代码块声明了 ImplicitLocOpBuilder, ModuleOp, Operation, RankedTensorType, Type, and Value 等支撑状态，供本文件中的其他 API 使用。

### Lines 21-22
```cpp
  21: namespace mlir::triton {
  22: class FuncOp;
```
**EN:** This block stores supporting state such as FuncOp, which other APIs in the file consume.
**CN:** 该代码块声明了 FuncOp 等支撑状态，供本文件中的其他 API 使用。

### Lines 24-25
```cpp
  24: namespace instrument {
  25: std::string mangleType(Type t);
```
**EN:** This block declares or defines callable APIs such as mangleType, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 mangleType 等可调用 API，用来封装这里提供的核心行为。

### Lines 27-29
```cpp
  27: class ManglingArgs {
  28: public:
  29:   using Arg = std::variant<Type, uint64_t, std::string>;
```
**EN:** This block introduces `ManglingArgs`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `ManglingArgs`。

### Lines 31-35
```cpp
  31:   ManglingArgs() = default;
  32:   ManglingArgs(const ManglingArgs &) = default;
  33:   ManglingArgs(ManglingArgs &&) = default;
  34:   ManglingArgs &operator=(const ManglingArgs &) = default;
  35:   ManglingArgs &operator=(ManglingArgs &&) = default;
```
**EN:** This block declares or defines callable APIs such as ManglingArgs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ManglingArgs 等可调用 API，用来封装这里提供的核心行为。

### Lines 37-37
```cpp
  37:   ManglingArgs(std::initializer_list<Arg> args) : args(args) {}
```
**EN:** This block declares or defines callable APIs such as ManglingArgs and args, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ManglingArgs and args 等可调用 API，用来封装这里提供的核心行为。

### Lines 39-39
```cpp
  39:   ~ManglingArgs() = default;
```
**EN:** This block declares or defines callable APIs such as ~ManglingArgs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ~ManglingArgs 等可调用 API，用来封装这里提供的核心行为。

### Lines 41-41
```cpp
  41:   template <typename T> void append(T arg) { args.push_back(arg); }
```
**EN:** This block declares or defines callable APIs such as append and push_back, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 append and push_back 等可调用 API，用来封装这里提供的核心行为。

### Lines 43-47
```cpp
  43:   template <typename T> void append(ArrayRef<T> arg) {
  44:     for (auto &a : arg) {
  45:       args.push_back(a);
  46:     }
  47:   }
```
**EN:** This block declares or defines callable APIs such as append and push_back, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 append and push_back 等可调用 API，用来封装这里提供的核心行为。

### Lines 49-51
```cpp
  49:   void append(ManglingArgs &other) {
  50:     args.append(other.args.begin(), other.args.end());
  51:   }
```
**EN:** This block declares or defines callable APIs such as append, begin, and end, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 append, begin, and end 等可调用 API，用来封装这里提供的核心行为。

### Lines 53-62
```cpp
  53:   std::string mangleArg(Arg arg) const {
  54:     if (auto type = std::get_if<Type>(&arg)) {
  55:       return std::string("_") + mangleType(*type);
  56:     } else if (auto intVal = std::get_if<uint64_t>(&arg)) {
  57:       return std::string("_I") + std::to_string(*intVal);
  58:     } else if (auto stringVal = std::get_if<std::string>(&arg)) {
  59:       return *stringVal;
  60:     }
  61:     llvm_unreachable("Unsupported argument type");
  62:   }
```
**EN:** This block declares or defines callable APIs such as mangleArg, string, mangleType, and to_string, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 mangleArg, string, mangleType, and to_string 等可调用 API，用来封装这里提供的核心行为。

### Lines 64-71
```cpp
  64:   std::string mangle(std::string baseName, int numWarps) const {
  65:     std::string name = "__triton_consan_";
  66:     name += baseName;
  67:     name += "_nw" + std::to_string(numWarps);
  68:     for (auto arg : args)
  69:       name += mangleArg(arg);
  70:     return name;
  71:   }
```
**EN:** This block declares or defines callable APIs such as mangle, to_string, and mangleArg, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 mangle, to_string, and mangleArg 等可调用 API，用来封装这里提供的核心行为。

### Lines 73-75
```cpp
  73: private:
  74:   SmallVector<Arg> args;
  75: };
```
**EN:** This block stores supporting state such as args, which other APIs in the file consume.
**CN:** 该代码块声明了 args 等支撑状态，供本文件中的其他 API 使用。

### Lines 77-80
```cpp
  77: class FunctionBuilder {
  78: public:
  79:   FunctionBuilder(ModuleOp module, AuxDataMap &auxData)
  80:       : module(module), auxData(auxData) {}
```
**EN:** This block introduces `FunctionBuilder`, the main class/struct defined here. Within the declaration, methods such as module and auxData expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `FunctionBuilder`。 其中 module and auxData 等方法构成了它的主要接口。

### Lines 82-277
```cpp
  82:   // Create a function that fills a global tensor with a scalar value.
  83:   void createFillGlobalTensorCall(ImplicitLocOpBuilder &b, Value ptr,
  84:                                   RankedTensorType type, Value scalar);
  85:   // setWaiting: mark the base thread as waiting on the given barrier phase and
  86:   // record that phase for deadlock detection.
  87:   void createSetWaitingCall(ImplicitLocOpBuilder &b, Value mbar, int thread,
  88:                             Value phase, Value pred, Operation *insertPoint);
  89:   // clearWaiting: clear the waiting flag and stored phase for the base thread.
  90:   void createClearWaitingCall(ImplicitLocOpBuilder &b, Value mbar, int thread,
  91:                               Value pred, Operation *insertPoint);
  92:   // setActiveMask: reset the live base-thread mask for the next
  93:   // warp-specialize region.
  94:   void createSetActiveMaskCall(ImplicitLocOpBuilder &b, int activeMask,
  95:                                Operation *insertPoint);
  96:   // retireActiveThread: remove a base thread from the live mask after it
  97:   // reaches its warp-specialize terminator.
  98:   void createRetireActiveThreadCall(ImplicitLocOpBuilder &b, int thread,
  99:                                     Operation *insertPoint);
 100:   // checkAllActiveWaiting: assert that not all unfinished threads across the
 101:   // cluster are waiting on matching barrier phases.
 102:   void createCheckAllActiveWaitingCall(ImplicitLocOpBuilder &b, Value pred,
 103:                                        Operation *insertPoint);
 104:   // verifyBarrierCanInit: ensure the barrier is currently invalidated before
 105:   // initializing it again.
 106:   void createVerifyBarrierCanInitCall(ImplicitLocOpBuilder &b, Value mbar,
 107:                                       Value pred, Operation *insertPoint,
 108:                                       Value recipientCTAs);
 109:   // verifyBarrierInitialized: ensure the barrier has been initialized and not
 110:   // invalidated before it is used.
 111:   void createVerifyBarrierInitializedCall(ImplicitLocOpBuilder &b, Value mbar,
 112:                                           Value pred, Operation *insertPoint,
 113:                                           Value recipientCTAs);
 114:   // initBarrierState: Initialize the tracked barrier state to phase 0 and set
 115:   // both the initial and current arrival counts. A zero state denotes an
 116:   // invalidated/uninitialized barrier.
 117:   void createInitBarrierStateCall(ImplicitLocOpBuilder &b, Value mbar,
 118:                                   int count, Value pred,
 119:                                   Operation *insertPoint);
 120:   // invalidateBarrierState: clear the tracked barrier lifecycle state and any
 121:   // waiting bits for the barrier.
 122:   void createInvalidateBarrierStateCall(ImplicitLocOpBuilder &b, Value mbar,
 123:                                         Value pred, Operation *insertPoint);
 124:   // verifyBarrierArrive: Check that applying the arrive count would not drive
 125:   // the tracked current count negative, and that applying the tx-count delta
 126:   // would keep it in range. Triggers an assertion on failure.
 127:   void createVerifyBarrierArriveCall(ImplicitLocOpBuilder &b, Value mbar,
 128:                                      int count, Value pred,
 129:                                      Operation *insertPoint,
 130:                                      Value recipientCTAs, int txCount = 0);
 131:   // updateBarrierState: Apply an arrive count to the tracked barrier state,
 132:   // apply a tx-count delta, toggling the phase when both counts reach zero and
 133:   // reloading the current count from the initial count.
 134:   void createUpdateBarrierStateCall(ImplicitLocOpBuilder &b, Value mbar,
 135:                                     int count, Value pred,
 136:                                     Operation *insertPoint, Value recipientCTAs,
 137:                                     int txCount = 0);
 138:   // setWriteVisibility: Set the write visibility for a buffer. Marks the buffer
 139:   // as visible to the threads set in threadMask. Clears out any other threads
 140:   // from the visibility bitmask. We know this is safe because there cannot be
 141:   // outstanding writes to this buffer at this point.
 142:   void createSetWriteVisibilityCall(ImplicitLocOpBuilder &b, Value buf,
 143:                                     uint32_t length, uint64_t threadMask,
 144:                                     Value pred, MemType memType,
 145:                                     Operation *insertPoint, Value effectCTAs);
 146:   // setReadVisibility: add the threads set in threadMask to the buffer's read
 147:   // visibility bitmask.
 148:   void createSetReadVisibilityCall(ImplicitLocOpBuilder &b, Value buf,
 149:                                    uint32_t length, uint64_t threadMask,
 150:                                    Value pred, MemType memType,
 151:                                    Operation *insertPoint, Value effectCTAs);
 152:   // clearWriteTracking: clear all the information about threads writing to a
 153:   // buffer.
 154:   void createClearWriteTrackingCall(ImplicitLocOpBuilder &b, Value buf,
 155:                                     uint32_t length, Value pred,
 156:                                     MemType memType, Operation *insertPoint,
 157:                                     Value effectCTAs);
 158:   // clearReadVisibility: clear the read visibility for a buffer.
 159:   void createClearReadVisibilityCall(ImplicitLocOpBuilder &b, Value buf,
 160:                                      uint32_t length, Value pred,
 161:                                      MemType memType, Operation *insertPoint,
 162:                                      Value effectCTAs);
 163:   // clearReadTracking: clear the read tracking for a buffer.
 164:   void createClearReadTrackingCall(ImplicitLocOpBuilder &b, Value buf,
 165:                                    uint32_t length, Value pred, MemType memType,
 166:                                    Operation *insertPoint, Value effectCTAs);
 167:   // trackVisibleWrites: snapshot buffers currently visible to the thread into
 168:   // the tracking table for a barrier.
 169:   void createTrackVisibleWritesCall(ImplicitLocOpBuilder &b, Value mbar,
 170:                                     int thread, Value pred, MemType memType,
 171:                                     Operation *insertPoint, Value barrierCTAs);
 172:   // trackVisibleReads: snapshot buffers currently visible to the thread into
 173:   // the read tracking table for a barrier.
 174:   void createTrackVisibleReadsCall(ImplicitLocOpBuilder &b, Value mbar,
 175:                                    int thread, Value pred, MemType memType,
 176:                                    Operation *insertPoint, Value barrierCTAs);
 177:   // trackBarrierWriteForBuffer: mark a specific buffer as tracked by a
 178:   // barrier in the write-tracking table.
 179:   void createTrackBarrierWriteForBufferCall(ImplicitLocOpBuilder &b, Value mbar,
 180:                                             Value buf, uint32_t length,
 181:                                             Value pred, MemType memType,
 182:                                             Operation *insertPoint,
 183:                                             Value barrierCTAs,
 184:                                             Value effectCTAs);
 185:   // clearBarrierWriteTracking: clear all write tracking associated with the
 186:   // given barrier row.
 187:   void createClearBarrierWriteTrackingCall(ImplicitLocOpBuilder &b, Value mbar,
 188:                                            Value pred, MemType memType,
 189:                                            Operation *insertPoint);
 190:   // clearBarrierReadTracking: clear all read tracking associated with the
 191:   // given barrier row.
 192:   void createClearBarrierReadTrackingCall(ImplicitLocOpBuilder &b, Value mbar,
 193:                                           Value pred, MemType memType,
 194:                                           Operation *insertPoint);
 195:   // transferVisibleWrites: transfer write visibility tracked by a barrier to
 196:   // all threads in threadMask.
 197:   void createTransferVisibleWritesCall(ImplicitLocOpBuilder &b, Value mbar,
 198:                                        uint64_t threadMask, Value pred,
 199:                                        MemType memType, Operation *insertPoint);
 200:   // transferVisibleReads: transfer read visibility tracked by a barrier to all
 201:   // threads in threadMask.
 202:   void createTransferVisibleReadsCall(ImplicitLocOpBuilder &b, Value mbar,
 203:                                       uint64_t threadMask, Value pred,
 204:                                       MemType memType, Operation *insertPoint);
 205:   // verifyWriteVisibility: ensure the thread either sees the latest write or no
 206:   // other thread is writing the buffer.
 207:   void createVerifyWriteVisibilityCall(ImplicitLocOpBuilder &b, Value buf,
 208:                                        uint32_t length, int thread,
 209:                                        StringRef operandName, Value pred,
 210:                                        MemType memType, Operation *insertPoint,
 211:                                        Value effectCTAs);
 212:   // verifyReadVisibility: ensure all reads from the buffer are visible to the
 213:   // thread.
 214:   void createVerifyReadVisibilityCall(ImplicitLocOpBuilder &b, Value buf,
 215:                                       uint32_t length, int thread,
 216:                                       StringRef operandName, Value pred,
 217:                                       MemType memType, Operation *insertPoint,
 218:                                       Value effectCTAs);
 219:   // copyWriteVisibility: replicate the write visibility bit of sourceThread to
 220:   // every destination thread in destMask.
 221:   void createCopyWriteVisibilityCall(ImplicitLocOpBuilder &b, int sourceThread,
 222:                                      uint64_t destMask, Value pred,
 223:                                      MemType memType, Operation *insertPoint);
 224:   // copyReadVisibility: replicate the read visibility row of sourceThread to
 225:   // every destination thread in destMask.
 226:   void createCopyReadVisibilityCall(ImplicitLocOpBuilder &b, int sourceThread,
 227:                                     uint64_t destMask, Value pred,
 228:                                     MemType memType, Operation *insertPoint);
 229:   // publishClusterVisibility: after a non-relaxed cluster barrier, make
 230:   // synchronous facts visible to every CTA in the cluster.
 231:   void createPublishClusterVisibilityCall(ImplicitLocOpBuilder &b, Value pred,
 232:                                           MemType memType,
 233:                                           Operation *insertPoint);
 234:   // stageAccessForCommit: mark the buffer as staged (value -1) in the
 235:   // outstanding commit table for this thread.
 236:   void createStageAccessForCommitCall(ImplicitLocOpBuilder &b, Value buf,
 237:                                       uint32_t length, int thread, Value pred,
 238:                                       MemType memType,
 239:                                       CommitKind::Kind commitKind,
 240:                                       Operation *insertPoint);
 241:   // commitAccesses: convert staged entries to 1 and increment outstanding
 242:   // commits greater than zero for the committing thread.
 243:   void createCommitAccessesCall(ImplicitLocOpBuilder &b, int thread, Value pred,
 244:                                 CommitKind::Kind commitKind,
 245:                                 Operation *insertPoint);
 246:   // clearOutstandingCommitsTransferWrites: clear entries farther than
 247:   // outstandingNum from the thread and set write visibility for threads in
 248:   // transferThreadMask.
 249:   void createClearOutstandingCommitsTransferWritesCall(
 250:       ImplicitLocOpBuilder &b, int thread, uint64_t transferThreadMask,
 251:       int outstandingNum, Value pred, CommitKind::Kind commitKind,
 252:       MemType memType, Operation *insertPoint);
 253:   // clearOutstandingCommitsTransferReads: clear entries farther than
 254:   // outstandingNum from the thread and set read visibility for threads in
 255:   // transferThreadMask.
 256:   void createClearOutstandingCommitsTransferReadsCall(
 257:       ImplicitLocOpBuilder &b, int thread, uint64_t transferThreadMask,
 258:       int outstandingNum, Value pred, CommitKind::Kind commitKind,
 259:       MemType memType, Operation *insertPoint);
 260:   // clearOutstandingCommitsTransferBoth: clear entries farther than
 261:   // outstandingNum from the thread and set both write and read visibility
 262:   // for threads in transferThreadMask. Handles the partial case gracefully:
 263:   // if only one visibility table exists, delegates to the corresponding
 264:   // single-transfer function.
 265:   void createClearOutstandingCommitsTransferBothCall(
 266:       ImplicitLocOpBuilder &b, int thread, uint64_t transferThreadMask,
 267:       int outstandingNum, Value pred, CommitKind::Kind commitKind,
 268:       MemType memType, Operation *insertPoint);
 269:   // checkOutstandingCommits: assert that the outstanding commit row for the
 270:   // buffer is zero before the access described by pendingAccessType.
 271:   // When excludeSelf is true, the calling thread's own column is masked out
 272:   // so that only other partitions' outstanding commits are checked.
 273:   void createCheckOutstandingCommitsCall(
 274:       ImplicitLocOpBuilder &b, Value buf, uint32_t length, int thread,
 275:       StringRef pendingAccessType, Value pred, MemType memType,
 276:       CommitKind::Kind commitKind, Operation *insertPoint, Value effectCTAs,
 277:       bool excludeSelf = false);
```
**EN:** This block declares or defines callable APIs such as createFillGlobalTensorCall, createSetWaitingCall, createClearWaitingCall, createSetActiveMaskCall, createRetireActiveThreadCall, createCheckAllActiveWaitingCall, createVerifyBarrierCanInitCall, and createVerifyBarrierInitializedCall, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createFillGlobalTensorCall, createSetWaitingCall, createClearWaitingCall, createSetActiveMaskCall, createRetireActiveThreadCall, createCheckAllActiveWaitingCall, createVerifyBarrierCanInitCall, and createVerifyBarrierInitializedCall 等可调用 API，用来封装这里提供的核心行为。

### Lines 279-282
```cpp
 279: private:
 280:   ModuleOp module;
 281:   AuxDataMap &auxData;
 282: };
```
**EN:** This block stores supporting state such as module and auxData, which other APIs in the file consume.
**CN:** 该代码块声明了 module and auxData 等支撑状态，供本文件中的其他 API 使用。

### Lines 284-285
```cpp
 284: } // namespace instrument
 285: } // namespace mlir::triton
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 287-287
```cpp
 287: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** warp-level execution  
  **CN:** warp 级执行
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** types  
  **CN:** 类型
- **EN:** instrumentation  
  **CN:** 插桩
- **EN:** concurrency sanitizer support  
  **CN:** 并发消毒器支持
- **EN:** barrier semantics  
  **CN:** 屏障语义

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `triton/Dialect/TritonInstrument/IR/Utility.h`
  - `llvm/ADT/ArrayRef.h`
  - `llvm/ADT/StringRef.h`
- **System or external includes / 系统或外部依赖:**
  - `<string>`
  - `<variant>`
