# ConSanTargetHooks.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonInstrument/Transforms/ConSanTargetHooks.h`
- **EN:** Declares transformation support utilities centered on `ConSanTargetHooks`.
- **CN:** 声明围绕 `ConSanTargetHooks` 的变换支持工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITONINSTRUMENT_CONSAN_TARGET_HOOKS_H
   2: #define TRITONINSTRUMENT_CONSAN_TARGET_HOOKS_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-10
```cpp
   4: #include "mlir/IR/BuiltinOps.h"
   5: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   6: #include "triton/Dialect/TritonInstrument/IR/Utility.h"
   7: #include <functional>
   8: #include <memory>
   9: #include <optional>
  10: #include <string>
```
**EN:** This block imports the direct dependencies needed here, including mlir/IR/BuiltinOps.h, triton/Dialect/TritonGPU/IR/Dialect.h, triton/Dialect/TritonInstrument/IR/Utility.h, <functional>, <memory>, and <optional>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/IR/BuiltinOps.h, triton/Dialect/TritonGPU/IR/Dialect.h, triton/Dialect/TritonInstrument/IR/Utility.h, <functional>, <memory>, and <optional>。

### Lines 12-12
```cpp
  12: namespace mlir::triton::instrument {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir::triton::instrument.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir::triton::instrument 下。

### Lines 14-36
```cpp
  14: struct MemEffectsOpInfo {
  15:   // Controls which memory effects become visible to a CTA after it waits on
  16:   // this barrier.
  17:   //
  18:   // Frontier snapshots the issuing thread's current visibility frontier into
  19:   // the barrier. A later wait publishes whatever shared/tensor memory writes
  20:   // and reads were visible to that logical thread before the arrive/commit. Use
  21:   // this for ordering operations whose semantics are a release of prior work.
  22:   //
  23:   // EffectWrites does not snapshot the whole thread frontier. Instead, it
  24:   // attaches only the explicit write effects of this op to the barrier. A later
  25:   // wait publishes those op-local writes and nothing else. Use this for PTX ops
  26:   // that perform the write and also signal the barrier via
  27:   // `mbarrier::complete_tx`.
  28:   enum class BarrierTrackingMode {
  29:     Frontier,
  30:     EffectWrites,
  31:   };
  32:   struct Effects {
  33:     enum RW { Read, Write } rw;
  34:     Value buf;
  35:     std::string operandName = "";
  36:     uint32_t length = 0;
```
**EN:** This block introduces `MemEffectsOpInfo`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `MemEffectsOpInfo`。

### Lines 38-54
```cpp
  38:     Effects(RW rw, Value buf, std::string operandName = "")
  39:         : rw(rw), buf(buf), operandName(operandName),
  40:           length(getMemDescLength(buf)) {}
  41:   };
  42:   struct BarrierInfo {
  43:     Value barrier;
  44:     Value pred;
  45:     int count;
  46:     BarrierTrackingMode trackingMode = BarrierTrackingMode::Frontier;
  47:     int txCount = 0;
  48:   };
  49:   enum class TrackingKind {
  50:     None,
  51:     Barrier,
  52:     wgmmaCommit,
  53:     CommitCount
  54:   } trackingKind = TrackingKind::None;
```
**EN:** This block declares or defines callable APIs such as Effects, rw, buf, operandName, length, and getMemDescLength, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 Effects, rw, buf, operandName, length, and getMemDescLength 等可调用 API，用来封装这里提供的核心行为。

### Lines 56-56
```cpp
  56:   CommitKind::Kind commitKind = CommitKind::None;
```
**EN:** This block stores supporting state such as None, which other APIs in the file consume.
**CN:** 该代码块声明了 None 等支撑状态，供本文件中的其他 API 使用。

### Lines 58-62
```cpp
  58:   SmallVector<BarrierInfo> barriers;
  59:   Value pred;
  60:   SmallVector<Effects> operandEffects;
  61:   bool implicitCommit = false;
  62: };
```
**EN:** This block stores supporting state such as barriers, pred, operandEffects, and false, which other APIs in the file consume.
**CN:** 该代码块声明了 barriers, pred, operandEffects, and false 等支撑状态，供本文件中的其他 API 使用。

### Lines 64-67
```cpp
  64: struct BarrierInitInfo {
  65:   Value alloc;
  66:   uint32_t count;
  67: };
```
**EN:** This block introduces `BarrierInitInfo`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `BarrierInitInfo`。

### Lines 69-73
```cpp
  69: struct BarrierWaitInfo {
  70:   Value alloc;
  71:   Value phase;
  72:   Value pred;
  73: };
```
**EN:** This block introduces `BarrierWaitInfo`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `BarrierWaitInfo`。

### Lines 75-77
```cpp
  75: struct BarrierInvalidateInfo {
  76:   Value alloc;
  77: };
```
**EN:** This block introduces `BarrierInvalidateInfo`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `BarrierInvalidateInfo`。

### Lines 79-84
```cpp
  79: struct WaitOpInfo {
  80:   CommitKind::Kind commitKind;
  81:   int pendingCount;
  82:   bool transferWrites;
  83:   bool transferReads;
  84: };
```
**EN:** This block introduces `WaitOpInfo`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `WaitOpInfo`。

### Lines 86-89
```cpp
  86: struct CommitKindDesc {
  87:   CommitKind::Kind kind;
  88:   std::string operationDesc;
  89: };
```
**EN:** This block introduces `CommitKindDesc`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `CommitKindDesc`。

### Lines 91-93
```cpp
  91: class ConSanTargetHooks {
  92: public:
  93:   virtual ~ConSanTargetHooks() = default;
```
**EN:** This block introduces `ConSanTargetHooks`, the main class/struct defined here. Within the declaration, methods such as ~ConSanTargetHooks expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `ConSanTargetHooks`。 其中 ~ConSanTargetHooks 等方法构成了它的主要接口。

### Lines 95-95
```cpp
  95:   virtual bool isTMAOp(Operation *op) const = 0;
```
**EN:** This block declares or defines callable APIs such as isTMAOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isTMAOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 97-97
```cpp
  97:   virtual bool isCLCOp(Operation *op) const { return false; }
```
**EN:** This block declares or defines callable APIs such as isCLCOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isCLCOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 99-100
```cpp
  99:   virtual std::optional<BarrierInitInfo>
 100:   getBarrierInitInfo(Operation *op) const = 0;
```
**EN:** This block declares or defines callable APIs such as getBarrierInitInfo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBarrierInitInfo 等可调用 API，用来封装这里提供的核心行为。

### Lines 102-103
```cpp
 102:   virtual std::optional<BarrierWaitInfo>
 103:   getBarrierWaitInfo(Operation *op) const = 0;
```
**EN:** This block declares or defines callable APIs such as getBarrierWaitInfo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBarrierWaitInfo 等可调用 API，用来封装这里提供的核心行为。

### Lines 105-106
```cpp
 105:   virtual std::optional<BarrierInvalidateInfo>
 106:   getBarrierInvalidateInfo(Operation *op) const = 0;
```
**EN:** This block declares or defines callable APIs such as getBarrierInvalidateInfo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBarrierInvalidateInfo 等可调用 API，用来封装这里提供的核心行为。

### Lines 108-108
```cpp
 108:   virtual std::optional<WaitOpInfo> getWaitOpInfo(Operation *op) const = 0;
```
**EN:** This block declares or defines callable APIs such as getWaitOpInfo, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getWaitOpInfo 等可调用 API，用来封装这里提供的核心行为。

### Lines 110-111
```cpp
 110:   virtual Value getIssuerCTAPred(ImplicitLocOpBuilder &b,
 111:                                  Operation *op) const = 0;
```
**EN:** This block declares or defines callable APIs such as getIssuerCTAPred, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getIssuerCTAPred 等可调用 API，用来封装这里提供的核心行为。

### Lines 113-145
```cpp
 113:   virtual std::optional<MemEffectsOpInfo>
 114:   getMemEffectsOpInfo(Operation *op) const {
 115:     namespace ttg = triton::gpu;
 116:     std::optional<MemEffectsOpInfo> info;
 117:     if (auto copyOp = dyn_cast<ttg::AsyncCopyGlobalToLocalOp>(op)) {
 118:       info.emplace();
 119:       info->trackingKind = MemEffectsOpInfo::TrackingKind::CommitCount;
 120:       info->commitKind = CommitKind::AsyncCp;
 121:       info->operandEffects.emplace_back(MemEffectsOpInfo::Effects::Write,
 122:                                         copyOp.getResult());
 123:     }
 124:     if (auto loadOp = dyn_cast<ttg::LocalLoadOp>(op)) {
 125:       info.emplace();
 126:       info->trackingKind = MemEffectsOpInfo::TrackingKind::Barrier;
 127:       info->operandEffects.emplace_back(MemEffectsOpInfo::Effects::Read,
 128:                                         loadOp.getSrc());
 129:     }
 130:     if (auto storeOp = dyn_cast<ttg::LocalStoreOp>(op)) {
 131:       info.emplace();
 132:       info->trackingKind = MemEffectsOpInfo::TrackingKind::Barrier;
 133:       info->operandEffects.emplace_back(MemEffectsOpInfo::Effects::Write,
 134:                                         storeOp.getDst());
 135:     }
 136:     if (auto allocOp = dyn_cast<ttg::LocalAllocOp>(op)) {
 137:       if (allocOp.getSrc()) {
 138:         info.emplace();
 139:         info->trackingKind = MemEffectsOpInfo::TrackingKind::Barrier;
 140:         info->operandEffects.emplace_back(MemEffectsOpInfo::Effects::Write,
 141:                                           allocOp.getResult());
 142:       }
 143:     }
 144:     return info;
 145:   }
```
**EN:** This block declares or defines callable APIs such as getMemEffectsOpInfo, emplace, emplace_back, getResult, getSrc, and getDst, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getMemEffectsOpInfo, emplace, emplace_back, getResult, getSrc, and getDst 等可调用 API，用来封装这里提供的核心行为。

### Lines 147-151
```cpp
 147:   // Returns commit kinds used by addWriteChecks to detect outstanding
 148:   // write accesses to shared memory.
 149:   virtual SmallVector<CommitKindDesc> getOutstandingWriteCommitKinds() const {
 150:     return {{CommitKind::AsyncCp, "async_copy_global_to_shared"}};
 151:   }
```
**EN:** This block declares or defines callable APIs such as getOutstandingWriteCommitKinds, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOutstandingWriteCommitKinds 等可调用 API，用来封装这里提供的核心行为。

### Lines 153-157
```cpp
 153:   // Returns commit kinds used by addReadChecks to detect outstanding
 154:   // read accesses to shared memory.
 155:   virtual SmallVector<CommitKindDesc> getOutstandingReadCommitKinds() const {
 156:     return {};
 157:   }
```
**EN:** This block declares or defines callable APIs such as getOutstandingReadCommitKinds, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOutstandingReadCommitKinds 等可调用 API，用来封装这里提供的核心行为。

### Lines 159-168
```cpp
 159:   // Returns true for commit kinds whose ops complete in issue order within a
 160:   // warp. ConSan's thread model tracks one logical
 161:   // thread per WS partition, so it cannot distinguish intra-warp ordering from
 162:   // cross-warp races inside the same partition. For such kinds, the
 163:   // outstanding-commit check excludes the calling thread's own column, avoiding
 164:   // intra-partition false positives while still detecting cross-partition
 165:   // races.
 166:   virtual bool isOrderedCommitKind(CommitKind::Kind kind) const {
 167:     return false;
 168:   }
```
**EN:** This block declares or defines callable APIs such as isOrderedCommitKind, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isOrderedCommitKind 等可调用 API，用来封装这里提供的核心行为。

### Lines 170-172
```cpp
 170:   virtual SmallVector<CommitKind::Kind>
 171:   getRequiredCommitKinds(ModuleOp module) const = 0;
 172: };
```
**EN:** This block declares or defines callable APIs such as getRequiredCommitKinds, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getRequiredCommitKinds 等可调用 API，用来封装这里提供的核心行为。

### Lines 174-175
```cpp
 174: LogicalResult runConcurrencySanitizer(ModuleOp module,
 175:                                       const ConSanTargetHooks *hooks);
```
**EN:** This block declares or defines callable APIs such as runConcurrencySanitizer, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 runConcurrencySanitizer 等可调用 API，用来封装这里提供的核心行为。

### Lines 177-179
```cpp
 177: using ConSanHooksFactory = std::function<std::unique_ptr<ConSanTargetHooks>()>;
 178: void registerConSanHooks(llvm::StringRef key, ConSanHooksFactory factory);
 179: std::unique_ptr<ConSanTargetHooks> createConSanHooks(llvm::StringRef key);
```
**EN:** This block declares or defines callable APIs such as registerConSanHooks and createConSanHooks, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 registerConSanHooks and createConSanHooks 等可调用 API，用来封装这里提供的核心行为。

### Lines 181-181
```cpp
 181: } // namespace mlir::triton::instrument
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 183-183
```cpp
 183: #endif // TRITONINSTRUMENT_CONSAN_TARGET_HOOKS_H
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** warp-level execution  
  **CN:** warp 级执行
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** instrumentation  
  **CN:** 插桩
- **EN:** concurrency sanitizer support  
  **CN:** 并发消毒器支持
- **EN:** barrier semantics  
  **CN:** 屏障语义
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/IR/BuiltinOps.h`
  - `triton/Dialect/TritonGPU/IR/Dialect.h`
  - `triton/Dialect/TritonInstrument/IR/Utility.h`
- **System or external includes / 系统或外部依赖:**
  - `<functional>`
  - `<memory>`
  - `<optional>`
  - `<string>`
