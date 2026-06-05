# Partition.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/TritonGPU/Transforms/WarpSpecialization/Partition.cpp`
- **Purpose / 作用:** **EN:** Implements the Partition transformation or optimization pass for the TritonGPU pipeline. **CN:** 为 TritonGPU 编译流程实现与 Partition 相关的变换或优化 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11

```cpp
   1: #include "triton/Dialect/TritonGPU/Transforms/Partition.h"
   2: #include "PartitionAttrs.h"
   3: #include "mlir/Dialect/SCF/IR/SCF.h"
   4: #include "mlir/Dialect/UB/IR/UBOps.h"
   5: #include "mlir/IR/BuiltinAttributes.h"
   6: #include "triton/Dialect/Triton/IR/Dialect.h"
   7: #include "triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h"
   8: #include "triton/Dialect/TritonGPU/Transforms/Utility.h"
   9: #include "llvm/ADT/SCCIterator.h"
  10: #include "llvm/ADT/STLExtras.h"
  11: #include "llvm/IR/Use.h"
```

- **EN:** Includes the interfaces this file depends on. Triton headers (`Partition.h`, `Dialect.h`, `PipeliningUtility.h`, `Utility.h`) provide domain-specific IR/support, MLIR headers (`SCF.h`, `UBOps.h`, `BuiltinAttributes.h`) provide rewriting and analysis infrastructure, LLVM headers (`SCCIterator.h`, `STLExtras.h`, `Use.h`) supply low-level utilities, and standard/library headers (`PartitionAttrs.h`) cover general-purpose facilities.
- **CN:** 这里引入了本文件依赖的接口。Triton 头文件（`Partition.h`, `Dialect.h`, `PipeliningUtility.h`, `Utility.h`）提供领域特定 IR/支持逻辑，MLIR 头文件（`SCF.h`, `UBOps.h`, `BuiltinAttributes.h`）提供重写与分析基础设施，LLVM 头文件（`SCCIterator.h`, `STLExtras.h`, `Use.h`）提供底层工具，而标准/通用库头文件（`PartitionAttrs.h`）提供通用能力。
### Lines 13-15

```cpp
  13: using namespace mlir;
  14: using namespace triton;
  15: using namespace triton::gpu;
```

- **EN:** Introduces namespace aliases/imports (`mlir`, `triton`, `triton::gpu`) so the rest of the file can use MLIR/Triton symbols without repetitive qualification.
- **CN:** 这里通过命名空间导入（`mlir`, `triton`, `triton::gpu`）减少后续代码中的重复限定，使 MLIR/Triton 符号使用更简洁。
### Lines 17-17

```cpp
  17: namespace {
```

- **EN:** Opens or closes the namespace nesting for (anonymous), keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 (anonymous) 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 19-25

```cpp
  19: LogicalResult verifyPartitionIdsAttr(Operation *op, StringRef attrName,
  20:                                      Attribute attrValue) {
  21:   auto partitionIdsAttr = dyn_cast<DenseI32ArrayAttr>(attrValue);
  22:   if (!partitionIdsAttr) {
  23:     return op->emitOpError("has invalid attribute ")
  24:            << attrName << "; expected a dense i32 array";
  25:   }
```

- **EN:** Defines `verifyPartitionIdsAttr`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `verifyPartitionIdsAttr`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 27-35

```cpp
  27:   SetVector<int> idSet;
  28:   for (auto id : partitionIdsAttr.asArrayRef()) {
  29:     if (idSet.contains(id))
  30:       return op->emitOpError("has duplicated partition ids in attribute ")
  31:              << attrName;
  32:     idSet.insert(id);
  33:   }
  34:   if (idSet.empty())
  35:     return op->emitOpError("has no partition ids in attribute ") << attrName;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 37-45

```cpp
  37:   auto ids = idSet.takeVector();
  38:   SmallVector<int> sortedIds(ids.begin(), ids.end());
  39:   llvm::sort(sortedIds);
  40:   if (ids != sortedIds) {
  41:     return op->emitOpError("partition ids not in sorted order in attribute ")
  42:            << attrName;
  43:   }
  44:   return success();
  45: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 47-53

```cpp
  47: LogicalResult verifyPartitionAttrs(Operation *op) {
  48:   if (op->hasAttr(kWarpSpecializeAttrName)) {
  49:     if (!isa<scf::ForOp>(op)) {
  50:       return op->emitOpError("has unexpected attribute ")
  51:              << kWarpSpecializeAttrName
  52:              << " which is expected only on `scf.for` ops";
  53:     }
```

- **EN:** Defines `verifyPartitionAttrs`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `verifyPartitionAttrs`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 55-71

```cpp
  55:     Operation *failedOp = nullptr;
  56:     op->walk([&](Operation *childOp) {
  57:       if (isa<ub::PoisonOp>(childOp))
  58:         return WalkResult::advance();
  59:       if (!childOp->hasAttr(kPartitionAttrName)) {
  60:         failedOp = childOp;
  61:         return WalkResult::interrupt();
  62:       }
  63:       return WalkResult::advance();
  64:     });
  65:     if (failedOp) {
  66:       return failedOp->emitOpError("does not have expected attribute ")
  67:              << kPartitionAttrName
  68:              << " which is expected on all child ops of an op with attribute `"
  69:              << kWarpSpecializeAttrName << "`";
  70:     }
  71:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 73-76

```cpp
  73:   if (auto partitionAttr = op->getAttr(kPartitionAttrName)) {
  74:     if (failed(verifyPartitionIdsAttr(op, kPartitionAttrName, partitionAttr)))
  75:       return failure();
  76:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 78-83

```cpp
  78:   if (auto outputsAttr = op->getAttr(kPartitionOutputsAttrName)) {
  79:     auto arrayAttr = dyn_cast<ArrayAttr>(outputsAttr);
  80:     if (!arrayAttr) {
  81:       return op->emitOpError("has invalid attribute ")
  82:              << kPartitionOutputsAttrName << "; expected an array attribute";
  83:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 85-90

```cpp
  85:     for (Attribute attr : arrayAttr) {
  86:       if (failed(verifyPartitionIdsAttr(op, kPartitionOutputsAttrName, attr))) {
  87:         return failure();
  88:       }
  89:     }
  90:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 92-108

```cpp
  92:   if (op->hasAttr(kPartitionAttrName) && op->getNumRegions() != 0) {
  93:     SetVector<int> expectedIds;
  94:     for (Region &region : op->getRegions()) {
  95:       for (Block &block : region.getBlocks()) {
  96:         for (Operation &childOp : block.getOperations()) {
  97:           if (isa<scf::YieldOp, ub::PoisonOp>(childOp))
  98:             continue;
  99:           if (!childOp.hasAttr(kPartitionAttrName)) {
 100:             return childOp.emitOpError("does not have expected attribute ")
 101:                    << kPartitionAttrName
 102:                    << " which is expected for ops whose parent has partitions";
 103:           }
 104:           auto ids = getPartitionIds(&childOp);
 105:           expectedIds.insert(ids.begin(), ids.end());
 106:         }
 107:       }
 108:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 110-118

```cpp
 110:     auto partitionIds = getPartitionIds(op);
 111:     for (auto id : expectedIds) {
 112:       if (!partitionIds.contains(id)) {
 113:         return op->emitOpError("partition ids in attr ")
 114:                << kPartitionAttrName
 115:                << " does not contain partition ids of all child ops";
 116:       }
 117:     }
 118:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 120-123

```cpp
 120:   if (auto outputsAttr = op->getAttr(kPartitionOutputsAttrName)) {
 121:     if (!isa<scf::ForOp, scf::IfOp, triton::ReduceOp>(op))
 122:       return op->emitOpError("has unexpected attribute ")
 123:              << kPartitionOutputsAttrName;
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 125-131

```cpp
 125:     size_t numResults = op->getNumResults();
 126:     auto arrayAttr = cast<ArrayAttr>(outputsAttr);
 127:     if (arrayAttr.size() != numResults) {
 128:       return op->emitOpError("does not have expected number of output "
 129:                              "partition sets in attr ")
 130:              << kPartitionOutputsAttrName << "; should match number of results";
 131:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 133-137

```cpp
 133:     if (!op->hasAttr(kPartitionAttrName)) {
 134:       return op->emitOpError("does not have expected attribute ")
 135:              << kPartitionAttrName << " which is expected for ops with attr "
 136:              << kPartitionOutputsAttrName;
 137:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 139-152

```cpp
 139:     auto partitionIds = getPartitionIds(op);
 140:     SetVector<int> outputPartitionIdsUnion;
 141:     for (auto outputPartitionIds : getPartitionOutputs(op)) {
 142:       outputPartitionIdsUnion.insert(outputPartitionIds.begin(),
 143:                                      outputPartitionIds.end());
 144:     }
 145:     if (!llvm::all_of(outputPartitionIdsUnion,
 146:                       [&](int id) { return partitionIds.contains(id); })) {
 147:       return op->emitOpError("partition ids in attr ")
 148:              << kPartitionAttrName
 149:              << " must be the union of all partition ids in "
 150:              << kPartitionOutputsAttrName;
 151:     }
 152:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 154-155

```cpp
 154:   return success();
 155: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 157-157

```cpp
 157: } // namespace
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 159-161

```cpp
 159: //===----------------------------------------------------------------------===//
 160: // Partition
 161: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 163-169

```cpp
 163: bool Partition::hasOp(Operation *op) const {
 164:   if (!hasPartition(op)) {
 165:     return false;
 166:   }
 167:   auto partitionIds = getPartitionIds(op);
 168:   return partitionIds.contains(getIndex());
 169: }
```

- **EN:** Defines `Partition::hasOp`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Partition::hasOp`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 171-188

```cpp
 171: void Partition::iterateInputs(scf::ForOp loop,
 172:                               function_ref<void(OpOperand &)> callback) const {
 173:   for (Operation *op : getOps()) {
 174:     visitNestedOperands(op, [&](OpOperand &operand) {
 175:       // Ignore implicit captures.
 176:       Value value = operand.get();
 177:       std::optional<SetVector<int>> partitionIds;
 178:       if (hasPartition(value.getDefiningOp()))
 179:         partitionIds = getPartitionIds(value.getDefiningOp());
 180:       if (value.getParentBlock() != loop.getBody())
 181:         return;
 182:       if (auto arg = dyn_cast<BlockArgument>(value)) {
 183:         assert(arg.getOwner() == loop.getBody());
 184:         // Ignore the induction variable.
 185:         if (arg == loop.getInductionVar())
 186:           return;
 187:         // This value originates from a previous iteration.
 188:         assert(llvm::is_contained(loop.getRegionIterArgs(), arg));
```

- **EN:** Defines `Partition::iterateInputs`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic. Assertions document invariants that should already hold at this stage.
- **CN:** 这里定义 `Partition::iterateInputs`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 189-199

```cpp
 189:         callback(operand);
 190:       } else if (!partitionIds ||
 191:                  !llvm::is_contained(*partitionIds, getIndex())) {
 192:         // This value originates from a different partition in the same
 193:         // iteration.
 194:         assert(value.getDefiningOp()->getParentOp() == loop);
 195:         callback(operand);
 196:       }
 197:     });
 198:   }
 199: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 201-218

```cpp
 201: void Partition::iterateOutputs(
 202:     scf::ForOp loop,
 203:     function_ref<void(Operation *, OpOperand &)> callback) const {
 204:   for (Operation *op : getOps()) {
 205:     for (OpOperand &use : op->getUses()) {
 206:       Operation *owner = loop.getBody()->findAncestorOpInBlock(*use.getOwner());
 207:       if (!owner) {
 208:         continue;
 209:       }
 210:       std::optional<SetVector<int>> partitionIds;
 211:       if (hasPartition(owner))
 212:         partitionIds = getPartitionIds(owner);
 213:       if (isa<scf::YieldOp>(owner)) {
 214:         // This value is used in a subsequent iteration.
 215:         callback(owner, use);
 216:       } else if (!partitionIds ||
 217:                  !llvm::is_contained(*partitionIds, getIndex())) {
 218:         // This value is used in a different partition in the same iteration.
```

- **EN:** Defines `Partition::iterateOutputs`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Partition::iterateOutputs`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 219-223

```cpp
 219:         callback(owner, use);
 220:       }
 221:     }
 222:   }
 223: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 225-232

```cpp
 225: void Partition::iterateDefs(
 226:     scf::ForOp loop, function_ref<void(OpResult, unsigned)> callback) const {
 227:   iterateInputs(loop, [&](OpOperand &input) {
 228:     auto [def, distance] = getDefinitionAndDistance(loop, input.get());
 229:     if (def && def.getParentBlock() == loop.getBody())
 230:       callback(def, distance);
 231:   });
 232: }
```

- **EN:** Defines `Partition::iterateDefs`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Partition::iterateDefs`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 234-251

```cpp
 234: void Partition::iterateUses(
 235:     scf::ForOp loop,
 236:     function_ref<void(OpResult, OpOperand &, unsigned)> callback) const {
 237:   SmallVector<std::tuple<OpResult, OpOperand *, unsigned>> uses;
 238:   iterateOutputs(loop, [&](Operation *owner, OpOperand &use) {
 239:     uses.emplace_back(cast<OpResult>(use.get()), &use, 0);
 240:   });
 241:   while (!uses.empty()) {
 242:     auto [output, use, distance] = uses.pop_back_val();
 243:     Operation *owner = loop.getBody()->findAncestorOpInBlock(*use->getOwner());
 244:     if (!owner) {
 245:       continue;
 246:     }
 247:     if (!isa<scf::YieldOp>(owner)) {
 248:       callback(output, *use, distance);
 249:       continue;
 250:     }
 251:     BlockArgument arg = loop.getRegionIterArg(use->getOperandNumber());
```

- **EN:** Defines `Partition::iterateUses`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `Partition::iterateUses`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 252-255

```cpp
 252:     for (OpOperand &use : arg.getUses())
 253:       uses.emplace_back(output, &use, distance + 1);
 254:   }
 255: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 257-259

```cpp
 257: //===----------------------------------------------------------------------===//
 258: // PartitionSet
 259: //===----------------------------------------------------------------------===//
```

- **EN:** This comment block explains intent, constraints, or invariants for the code that follows.
- **CN:** 这段注释用于解释后续代码的意图、约束或不变量。
### Lines 261-264

```cpp
 261: Partition *PartitionSet::addPartition(unsigned stage) {
 262:   partitions.push_back(std::make_unique<Partition>(partitions.size(), stage));
 263:   return partitions.back().get();
 264: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 266-268

```cpp
 266: Partition *PartitionSet::getPartition(unsigned idx) {
 267:   return partitions[idx].get();
 268: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 270-272

```cpp
 270: const Partition *PartitionSet::getPartition(unsigned idx) const {
 271:   return partitions[idx].get();
 272: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 274-278

```cpp
 274: Partition *PartitionSet::getPartition(Operation *op) {
 275:   auto id = getPartitionIds(op);
 276:   assert(id.size() == 1);
 277:   return getPartition(id[0]);
 278: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 280-282

```cpp
 280: FailureOr<PartitionSet> PartitionSet::fromLoop(scf::ForOp loop) {
 281:   if (failed(verifyPartitionedLoop(loop)))
 282:     return failure();
```

- **EN:** Defines `PartitionSet::fromLoop`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `PartitionSet::fromLoop`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 284-286

```cpp
 284:   auto stages = loop->getAttrOfType<ArrayAttr>(kPartitionStagesAttrName);
 285:   if (!stages)
 286:     return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 288-290

```cpp
 288:   auto tag = loop->getAttrOfType<IntegerAttr>(kWarpSpecializeTagAttrName);
 289:   if (!tag)
 290:     return failure();
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 292-299

```cpp
 292:   PartitionSet result;
 293:   result.tag = tag.getInt();
 294:   for (auto [idx, attr] : llvm::enumerate(stages)) {
 295:     auto stage = dyn_cast<IntegerAttr>(attr);
 296:     if (!stage || stage.getInt() < 0) {
 297:       return mlir::emitError(loop.getLoc(), "partition stages attribute '")
 298:              << kPartitionStagesAttrName << "' has invalid element " << attr;
 299:     }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 301-303

```cpp
 301:     result.partitions.push_back(
 302:         std::make_unique<Partition>(idx, stage.getInt()));
 303:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 305-310

```cpp
 305:   SmallVector<Operation *> annotatedOps;
 306:   loop->walk([&](Operation *op) {
 307:     if (hasPartition(op)) {
 308:       annotatedOps.push_back(op);
 309:     }
 310:   });
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 312-319

```cpp
 312:   for (auto op : annotatedOps) {
 313:     auto attrs = getPartitionIds(op);
 314:     for (auto idx : attrs) {
 315:       if (idx < 0 || idx >= result.partitions.size())
 316:         return mlir::emitError(op->getLoc(), "invalid partition index ") << idx;
 317:       result.partitions[idx]->addOp(op);
 318:     }
 319:   }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 321-322

```cpp
 321:   return result;
 322: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。
### Lines 324-335

```cpp
 324: void PartitionSet::dump() const {
 325:   for (auto [i, partition] :
 326:        llvm::enumerate(llvm::make_pointee_range(partitions))) {
 327:     llvm::errs() << "=== PARTITION #" << i << " ===\n";
 328:     for (Operation *op : partition.getOps()) {
 329:       op->print(llvm::errs(), OpPrintingFlags().skipRegions());
 330:       llvm::errs() << "\n";
 331:     }
 332:     llvm::errs() << "\n";
 333:   }
 334:   llvm::errs() << "\n";
 335: }
```

- **EN:** Defines `PartitionSet::dump`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `PartitionSet::dump`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 337-337

```cpp
 337: namespace mlir::triton::gpu {
```

- **EN:** Opens or closes the namespace nesting for mlir::triton::gpu, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 mlir::triton::gpu 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。
### Lines 339-347

```cpp
 339: SetVector<int> getPartitionIds(Operation *op) {
 340:   auto attrs = op->getAttr(kPartitionAttrName);
 341:   SmallVector<int> partitionIds;
 342:   for (auto id : cast<DenseI32ArrayAttr>(attrs).asArrayRef()) {
 343:     partitionIds.push_back(id);
 344:   }
 345:   llvm::sort(partitionIds);
 346:   return SetVector<int>(partitionIds.begin(), partitionIds.end());
 347: }
```

- **EN:** Defines accessor/helper `getPartitionIds` that exposes or updates operation state in a compact, reusable way. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义访问器/辅助函数 `getPartitionIds`，以紧凑且可复用的方式读取或更新操作状态。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 349-352

```cpp
 349: SmallVector<SetVector<int>, 4> getPartitionOutputs(Operation *op) {
 350:   SmallVector<SetVector<int>, 4> partitionOutputsIds;
 351:   if (op->getNumResults() == 0)
 352:     return partitionOutputsIds;
```

- **EN:** Defines accessor/helper `getPartitionOutputs` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getPartitionOutputs`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 354-361

```cpp
 354:   assert(op->hasAttr(kPartitionOutputsAttrName));
 355:   auto arrayAttr = cast<ArrayAttr>(op->getAttr(kPartitionOutputsAttrName));
 356:   for (Attribute attr : arrayAttr) {
 357:     auto ids = cast<DenseI32ArrayAttr>(attr).asArrayRef();
 358:     partitionOutputsIds.push_back(SetVector<int>(ids.begin(), ids.end()));
 359:   }
 360:   return partitionOutputsIds;
 361: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. Assertions document invariants that should already hold at this stage.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 断言用于说明在当前阶段本就应该成立的不变量。
### Lines 363-373

```cpp
 363: SetVector<int> getPartitionIds(OpOperand *use) {
 364:   auto owner = use->getOwner();
 365:   if (isa<scf::YieldOp>(owner)) {
 366:     return getPartitionOutputs(owner->getParentOp())[use->getOperandNumber()];
 367:   }
 368:   if (auto forOp = dyn_cast<scf::ForOp>(owner)) {
 369:     int idx = use->getOperandNumber() - forOp.getNumControlOperands();
 370:     return idx >= 0 ? getPartitionOutputs(owner)[idx] : getPartitionIds(forOp);
 371:   }
 372:   return getPartitionIds(owner);
 373: }
```

- **EN:** Defines accessor/helper `getPartitionIds` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getPartitionIds`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 375-377

```cpp
 375: bool hasPartition(Operation *op) {
 376:   return op && op->hasAttr(kPartitionAttrName);
 377: }
```

- **EN:** Defines `hasPartition`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `hasPartition`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 379-381

```cpp
 379: bool hasWarpSpecializeTag(Operation *op) {
 380:   return op && op->hasAttr(kWarpSpecializeTagAttrName);
 381: }
```

- **EN:** Defines `hasWarpSpecializeTag`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `hasWarpSpecializeTag`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 383-387

```cpp
 383: std::optional<int> getWarpSpecializeTag(Operation *op) {
 384:   if (hasWarpSpecializeTag(op))
 385:     return cast<IntegerAttr>(op->getAttr(kWarpSpecializeTagAttrName)).getInt();
 386:   return std::nullopt;
 387: }
```

- **EN:** Defines accessor/helper `getWarpSpecializeTag` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `getWarpSpecializeTag`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 389-391

```cpp
 389: LogicalResult verifyPartitionedLoop(scf::ForOp loop) {
 390:   if (failed(verifyPartitionAttrs(loop)))
 391:     return failure();
```

- **EN:** Defines `verifyPartitionedLoop`, encapsulating a meaningful piece of this file's analysis, lowering, or IR support logic.
- **CN:** 这里定义 `verifyPartitionedLoop`，封装了本文件中的一段重要分析、降级转换或 IR 支持逻辑。
### Lines 393-402

```cpp
 393:   LogicalResult result = success();
 394:   loop.walk([&](Operation *op) {
 395:     if (failed(verifyPartitionAttrs(op))) {
 396:       result = failure();
 397:       return WalkResult::interrupt();
 398:     }
 399:     return WalkResult::advance();
 400:   });
 401:   return result;
 402: }
```

- **EN:** This block advances the surrounding algorithm with local control flow, bookkeeping, and helper calls. The code uses `LogicalResult` to stay conservative when preconditions are not met. The implementation traverses IR operations to collect facts or apply rewrites globally.
- **CN:** 这段代码通过局部控制流、状态维护和辅助调用推动外围算法继续向前执行。 代码通过 `LogicalResult` 在前提不满足时保持保守处理。 实现会遍历 IR 操作，以便在更大范围内收集信息或应用重写。
### Lines 404-415

```cpp
 404: void setPartition(Operation *op, ArrayRef<int> partitionIds) {
 405:   Builder b(op->getContext());
 406:   auto sorted = llvm::to_vector(partitionIds);
 407:   llvm::sort(sorted);
 408:   op->setAttr(kPartitionAttrName, b.getDenseI32ArrayAttr(sorted));
 409:   for (auto &region : op->getRegions()) {
 410:     for (auto &block : region.getBlocks()) {
 411:       auto terminator = block.getTerminator();
 412:       terminator->setAttr(kPartitionAttrName, b.getDenseI32ArrayAttr(sorted));
 413:     }
 414:   }
 415: }
```

- **EN:** Defines accessor/helper `setPartition` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `setPartition`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 417-431

```cpp
 417: void setPartitionOutputs(Operation *op,
 418:                          ArrayRef<SetVector<int>> partitionOutputsIds) {
 419:   if (partitionOutputsIds.empty()) {
 420:     op->removeAttr(kPartitionOutputsAttrName);
 421:     return;
 422:   }
 423:   SmallVector<Attribute> attrs;
 424:   Builder b(op->getContext());
 425:   for (auto partitionIds : partitionOutputsIds) {
 426:     auto sorted = llvm::to_vector(partitionIds);
 427:     llvm::sort(sorted);
 428:     attrs.push_back(b.getDenseI32ArrayAttr(sorted));
 429:   }
 430:   op->setAttr(kPartitionOutputsAttrName, b.getArrayAttr(attrs));
 431: }
```

- **EN:** Defines accessor/helper `setPartitionOutputs` that exposes or updates operation state in a compact, reusable way. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义访问器/辅助函数 `setPartitionOutputs`，以紧凑且可复用的方式读取或更新操作状态。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 433-436

```cpp
 433: void setPartition(Operation *op, const SetVector<int> &partitionIds) {
 434:   SmallVector<int> partitions(partitionIds.begin(), partitionIds.end());
 435:   setPartition(op, partitions);
 436: }
```

- **EN:** Defines accessor/helper `setPartition` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `setPartition`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 438-442

```cpp
 438: void setPartition(Operation *op, Partition *partition) {
 439:   SmallVector<int> partitions{partition->getIndex()};
 440:   setPartition(op, partitions);
 441:   partition->addOp(op);
 442: }
```

- **EN:** Defines accessor/helper `setPartition` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `setPartition`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 444-451

```cpp
 444: void setPartition(Operation *op, const SetVector<Partition *> &partitions) {
 445:   SmallVector<int> partitionIds;
 446:   for (auto partition : partitions) {
 447:     partitionIds.push_back(partition->getIndex());
 448:     partition->addOp(op);
 449:   }
 450:   setPartition(op, partitionIds);
 451: }
```

- **EN:** Defines accessor/helper `setPartition` that exposes or updates operation state in a compact, reusable way. Small temporary vectors are used to accumulate per-element or per-thread state efficiently.
- **CN:** 这里定义访问器/辅助函数 `setPartition`，以紧凑且可复用的方式读取或更新操作状态。 这里使用 SmallVector 高效累积逐元素或逐线程的临时状态。
### Lines 453-456

```cpp
 453: void setWarpSpecializeTag(Operation *op, int tag) {
 454:   Builder b(op->getContext());
 455:   op->setAttr(kWarpSpecializeTagAttrName, b.getI32IntegerAttr(tag));
 456: }
```

- **EN:** Defines accessor/helper `setWarpSpecializeTag` that exposes or updates operation state in a compact, reusable way.
- **CN:** 这里定义访问器/辅助函数 `setWarpSpecializeTag`，以紧凑且可复用的方式读取或更新操作状态。
### Lines 458-458

```cpp
 458: } // namespace mlir::triton::gpu
```

- **EN:** Opens or closes the namespace nesting for current scopes, keeping the implementation scoped to the intended MLIR/Triton components.
- **CN:** 这里打开或关闭 current scopes 的命名空间层级，使实现被限定在预期的 MLIR/Triton 组件作用域中。

## Key Concepts / 关键概念
- **EN:** The main concern is pass-driven transformation around partition.
  **CN:** 核心关注点是围绕 Partition 的 pass 驱动变换。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/TritonGPU/Transforms/Partition.h`, `triton/Dialect/Triton/IR/Dialect.h`, `triton/Dialect/TritonGPU/Transforms/PipeliningUtility.h`, `triton/Dialect/TritonGPU/Transforms/Utility.h`
- **MLIR headers / MLIR 头文件:** `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/BuiltinAttributes.h`
- **LLVM headers / LLVM 头文件:** `llvm/ADT/SCCIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/IR/Use.h`
- **Standard/library headers / 标准或通用库头文件:** `PartitionAttrs.h`
- **Pipeline role / 流程角色:** This file participates in Triton pass pipelines and usually expects upstream analyses or dialect invariants to have prepared the IR. / 本文件参与 Triton 的 pass 流程，通常依赖上游分析或方言不变量先把 IR 准备好。
