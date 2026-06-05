# TileAllocation.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/ArmSME/Transforms/TileAllocation.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This transform allocates SME tiles at the 'func.func' op level for ArmSME operations. It roughly implements a linear scan register allocator, similar to the one outlined in [1], but with simplifications and assumptions made for our use case. Note that this is a greedy allocator (so it may not always find the most optimal allocation of tiles).
  - **CN**: 实现 ArmSME 方言与 Arm SME tile/vector 支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```cpp
//===- TileAllocation.cpp - Allocate SME ZA tiles -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This transform allocates SME tiles at the 'func.func' op level for ArmSME
// operations. It roughly implements a linear scan register allocator, similar
// to the one outlined in [1], but with simplifications and assumptions made for
// our use case. Note that this is a greedy allocator (so it may not always find
// the most optimal allocation of tiles).
//
// The allocator operates at the CF dialect level. It is the responsibility of
// users to ensure the IR has been lowered to CF before invoking the tile
// allocator.
//
// The 128-bit tiles overlap with other element tiles as follows (see section
// B2.3.2 of SME spec [2]):
//
//   Tile    Overlaps
//   ---------------------------------------------------------------------------
//   ZA0.B   ZA0.Q, ZA1.Q, ZA2.Q, ZA3.Q, ZA4.Q, ZA5.Q, ZA6.Q, ZA7.Q, ZA8.Q,
//           ZA9.Q, ZA10.Q, ZA11.Q, ZA12.Q, ZA13.Q, ZA14.Q, ZA15.Q
//   ZA0.H   ZA0.Q, ZA2.Q, ZA4.Q, ZA6.Q, ZA8.Q, ZA10.Q, ZA12.Q, ZA14.Q
//   ZA1.H   ZA1.Q, ZA3.Q, ZA5.Q, ZA7.Q, ZA9.Q, ZA11.Q, ZA13.Q, ZA15.Q
//   ZA0.S   ZA0.Q, ZA4.Q, ZA8.Q, ZA12.Q
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 29-46
```cpp
//   ZA1.S   ZA1.Q, ZA5.Q, ZA9.Q, ZA13.Q
//   ZA2.S   ZA2.Q, ZA6.Q, ZA10.Q, ZA14.Q
//   ZA3.S   ZA3.Q, ZA7.Q, ZA11.Q, ZA15.Q
//   ZA0.D   ZA0.Q, ZA8.Q
//   ZA1.D   ZA1.Q, ZA9.Q
//   ZA2.D   ZA2.Q, ZA10.Q
//   ZA3.D   ZA3.Q, ZA11.Q
//   ZA4.D   ZA4.Q, ZA12.Q
//   ZA5.D   ZA5.Q, ZA13.Q
//   ZA6.D   ZA6.Q, ZA14.Q
//   ZA7.D   ZA7.Q, ZA15.Q
//
// [1] "Linear Scan Register Allocation in the Context of SSA Form and Register
//      Constraints" (Hanspeter Mössenböck and Michael Pfeiffer)
//     https://link.springer.com/content/pdf/10.1007/3-540-45937-5_17.pdf
// [2] https://developer.arm.com/documentation/ddi0616/aa
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 47-63
```cpp

#include "mlir/Analysis/Liveness.h"
#include "mlir/Analysis/TopologicalSortUtils.h"
#include "mlir/Dialect/ArmSME/IR/ArmSME.h"
#include "mlir/Dialect/ArmSME/Transforms/Passes.h"
#include "mlir/Dialect/ArmSME/Transforms/Transforms.h"
#include "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "llvm/ADT/IntervalMap.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include "llvm/ADT/TypeSwitch.h"

namespace mlir::arm_sme {
#define GEN_PASS_DEF_TESTTILEALLOCATION
#include "mlir/Dialect/ArmSME/Transforms/Passes.h.inc"
} // namespace mlir::arm_sme

```
- **EN**: Introduces declarations for `mlir::arm_sme`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir::arm_sme` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 64-80
```cpp
using namespace mlir;
using namespace mlir::arm_sme;

namespace {

enum class TileMask : unsigned {
  // clang-format off
  kZA0B  = 0xffff, // 1111 1111 1111 1111

  kZA0H  = 0xaaaa, // 1010 1010 1010 1010
  kZA1H  = 0x5555, // 0101 0101 0101 0101

  kZA0S  = 0x8888, // 1000 1000 1000 1000
  kZA1S  = 0x4444, // 0100 0100 0100 0100
  kZA2S  = 0x2222, // 0010 0010 0010 0010
  kZA3S  = 0x1111, // 0001 0001 0001 0001

```
- **EN**: Introduces declarations for `TileMask`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `TileMask` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 81-106
```cpp
  kZA0D  = 0x8080, // 1000 0000 1000 0000
  kZA1D  = 0x4040, // 0100 0000 0100 0000
  kZA2D  = 0x2020, // 0010 0000 0010 0000
  kZA3D  = 0x1010, // 0001 0000 0001 0000
  kZA4D  = 0x808,  // 0000 1000 0000 1000
  kZA5D  = 0x404,  // 0000 0100 0000 0100
  kZA6D  = 0x202,  // 0000 0010 0000 0010
  kZA7D  = 0x101,  // 0000 0001 0000 0001

  kZA0Q  = 0x8000, // 1000 0000 0000 0000
  kZA1Q  = 0x4000, // 0100 0000 0000 0000
  kZA2Q  = 0x2000, // 0010 0000 0000 0000
  kZA3Q  = 0x1000, // 0001 0000 0000 0000
  kZA4Q  = 0x800,  // 0000 1000 0000 0000
  kZA5Q  = 0x400,  // 0000 0100 0000 0000
  kZA6Q  = 0x200,  // 0000 0010 0000 0000
  kZA7Q  = 0x100,  // 0000 0001 0000 0000
  kZA8Q  = 0x80,   // 0000 0000 1000 0000
  kZA9Q  = 0x40,   // 0000 0000 0100 0000
  kZA10Q = 0x20,   // 0000 0000 0010 0000
  kZA11Q = 0x10,   // 0000 0000 0001 0000
  kZA12Q = 0x8,    // 0000 0000 0000 1000
  kZA13Q = 0x4,    // 0000 0000 0000 0100
  kZA14Q = 0x2,    // 0000 0000 0000 0010
  kZA15Q = 0x1,    // 0000 0000 0000 0001

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 107-134
```cpp
  kNone = 0x0,     // 0000 0000 0000 0000
  // clang-format on

  LLVM_MARK_AS_BITMASK_ENUM(kZA0B)
};

/// Returns the set of masks relevant for the given type.
static ArrayRef<TileMask> getMasks(ArmSMETileType type) {
  static constexpr std::array ZA_B_MASKS = {TileMask::kZA0B};
  static constexpr std::array ZA_H_MASKS = {TileMask::kZA0H, TileMask::kZA1H};
  static constexpr std::array ZA_S_MASKS = {TileMask::kZA0S, TileMask::kZA1S,
                                            TileMask::kZA2S, TileMask::kZA3S};
  static constexpr std::array ZA_D_MASKS = {
      TileMask::kZA0D, TileMask::kZA1D, TileMask::kZA2D, TileMask::kZA3D,
      TileMask::kZA4D, TileMask::kZA5D, TileMask::kZA6D, TileMask::kZA7D};
  static constexpr std::array ZA_Q_MASKS = {
      TileMask::kZA0Q,  TileMask::kZA1Q,  TileMask::kZA2Q,  TileMask::kZA3Q,
      TileMask::kZA4Q,  TileMask::kZA5Q,  TileMask::kZA6Q,  TileMask::kZA7Q,
      TileMask::kZA8Q,  TileMask::kZA9Q,  TileMask::kZA10Q, TileMask::kZA11Q,
      TileMask::kZA12Q, TileMask::kZA13Q, TileMask::kZA14Q, TileMask::kZA15Q};
  switch (type) {
  case ArmSMETileType::ZAB:
    return ZA_B_MASKS;
  case ArmSMETileType::ZAH:
    return ZA_H_MASKS;
  case ArmSMETileType::ZAS:
    return ZA_S_MASKS;
  case ArmSMETileType::ZAD:
```
- **EN**: Implements logic around `getMasks`.
- **CN**: 围绕 `getMasks` 实现具体逻辑。

### Lines 135-155
```cpp
    return ZA_D_MASKS;
  case ArmSMETileType::ZAQ:
    return ZA_Q_MASKS;
  }
  llvm_unreachable("unknown type in getMasks");
}

class TileAllocator {
public:
  /// Allocates and returns a tile ID. Fails if there are no tiles left.
  FailureOr<unsigned> allocateTileId(ArmSMETileType tileType) {
    auto masks = getMasks(tileType);
    for (auto [tileId, tileMask] : llvm::enumerate(masks)) {
      if ((tilesInUse & tileMask) == TileMask::kNone) {
        tilesInUse |= tileMask;
        return tileId;
      }
    }
    return failure();
  }

```
- **EN**: Introduces declarations for `TileAllocator`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `TileAllocator` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 156-171
```cpp
  /// Acquires a specific tile ID. Asserts the tile is initially free.
  void acquireTileId(ArmSMETileType tileType, unsigned tileId) {
    TileMask tileMask = getMasks(tileType)[tileId];
    assert((tilesInUse & tileMask) == TileMask::kNone &&
           "cannot acquire allocated tile!");
    tilesInUse |= tileMask;
  }

  /// Releases a previously allocated tile ID.
  void releaseTileId(ArmSMETileType tileType, unsigned tileId) {
    TileMask tileMask = getMasks(tileType)[tileId];
    assert((tilesInUse & tileMask) == tileMask &&
           "cannot release unallocated tile!");
    tilesInUse ^= tileMask;
  }

```
- **EN**: Implements logic around `acquireTileId`, `getMasks`, `assert`, `releaseTileId`.
- **CN**: 围绕 `acquireTileId`, `getMasks`, `assert`, `releaseTileId` 实现具体逻辑。

### Lines 172-185
```cpp
  /// Allocates an in-memory tile ID.
  unsigned allocateInMemoryTileId() {
    // Note: We never release in-memory tile IDs. We could, which may allow
    // reusing an allocation, but as we _never_ want to spill an SME tile this
    // is not optimized.
    return nextInMemoryTileId++;
  }

private:
  TileMask tilesInUse = TileMask::kNone;
  unsigned nextInMemoryTileId = kInMemoryTileIdBase;
};

/// Add new intermediate blocks for the true and false destinations of
```
- **EN**: Implements logic around `allocateInMemoryTileId`.
- **CN**: 围绕 `allocateInMemoryTileId` 实现具体逻辑。

### Lines 186-199
```cpp
/// `cf.cond_br`s that contain tile operands. This prevents spurious liveness
/// overlaps due to copies at branches.
///
///  BEFORE:
///  ```mlir
///  cf.cond_br %cond, ^bb1(%tile: vector<[4]x[4]xf32>), ^bb2
///  ```
///
///  AFTER:
///  ```mlir
///    cf.cond_br %cond, ^bb1_copy, ^bb2_copy
///  ^bb1_copy:
///    cf.br ^bb1(%tile: vector<[4]x[4]xf32>)
///  ^bb2_copy:
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 200-216
```cpp
///    cf.br ^bb2
///  ```
void splitCondBranches(IRRewriter &rewriter, FunctionOpInterface function) {
  SmallVector<cf::CondBranchOp> worklist;
  function.walk([&](cf::CondBranchOp condBranch) {
    if (llvm::any_of(condBranch->getOperands(), [&](Value value) {
          return isValidSMETileVectorType(value.getType());
        })) {
      worklist.push_back(condBranch);
    }
  });

  auto insertJump = [&](Location loc, Block *source, Block *dest, auto args) {
    rewriter.setInsertionPointToEnd(source);
    cf::BranchOp::create(rewriter, loc, dest, args);
  };

```
- **EN**: Implements logic around `splitCondBranches`, `walk`, `any_of`, `isValidSMETileVectorType`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `splitCondBranches`, `walk`, `any_of`, `isValidSMETileVectorType`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 217-234
```cpp
  for (auto condBranch : worklist) {
    auto loc = condBranch.getLoc();
    Block *block = condBranch->getBlock();
    auto *newTrueBranch = rewriter.splitBlock(block, block->end());
    auto *newFalseBranch = rewriter.splitBlock(block, block->end());
    insertJump(loc, newTrueBranch, condBranch.getTrueDest(),
               condBranch.getTrueDestOperands());
    insertJump(loc, newFalseBranch, condBranch.getFalseDest(),
               condBranch.getFalseDestOperands());
    rewriter.modifyOpInPlace(condBranch, [&] {
      condBranch.getFalseDestOperandsMutable().clear();
      condBranch.getTrueDestOperandsMutable().clear();
      condBranch.setSuccessor(newTrueBranch, 0);
      condBranch.setSuccessor(newFalseBranch, 1);
    });
  }
}

```
- **EN**: Implements logic around `getLoc`, `getBlock`, `splitBlock`, `insertJump`, and 6 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getLoc`, `getBlock`, `splitBlock`, `insertJump`, and 6 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 235-262
```cpp
/// Inserts tile copies at `cf.br` operations.
///
///  BEFORE:
///  ```mlir
///  cf.br ^bb1(%tile: vector<[4]x[4]xf32>)
///  ```
///
///  AFTER:
///  ```mlir
///  %copy = arm_sme.copy_tile %tile : vector<[4]x[4]xf32>
///  cf.br ^bb1(%copy: vector<[4]x[4]xf32>)
///  ```
void insertCopiesAtBranches(IRRewriter &rewriter,
                            FunctionOpInterface function) {
  for (Block &block : function.getBlocks()) {
    Operation *terminator = block.getTerminator();
    if (!isa<cf::BranchOp>(terminator))
      continue;
    rewriter.setInsertionPoint(terminator);
    for (OpOperand &operand : terminator->getOpOperands()) {
      if (isValidSMETileVectorType(operand.get().getType())) {
        auto copy =
            CopyTileOp::create(rewriter, terminator->getLoc(), operand.get());
        rewriter.modifyOpInPlace(terminator, [&] { operand.assign(copy); });
      }
    }
  }
}
```
- **EN**: Implements logic around `insertCopiesAtBranches`, `getBlocks`, `getTerminator`, `BranchOp>`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `insertCopiesAtBranches`, `getBlocks`, `getTerminator`, `BranchOp>`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 263-277
```cpp

/// Prepares the IR for tile allocation. It does this by first 'splitting'
/// conditional branches (see `splitCondBranches`), then inserting tile copies
/// at branch operations. The conditional branches are split to prevent the
/// copies needed for them overlapping between the true and false paths of the
/// branch (see `tile-allocation-copies.mlir` and
/// `tile-allocation-liveness.mlir` for examples). The copies break up live
/// ranges and ensure when moving out of SSA the semantics of the program are
/// preserved.
void preprocessForTileAllocation(IRRewriter &rewriter,
                                 FunctionOpInterface function) {
  splitCondBranches(rewriter, function);
  insertCopiesAtBranches(rewriter, function);
}

```
- **EN**: Implements logic around `preprocessForTileAllocation`, `splitCondBranches`, `insertCopiesAtBranches`; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `preprocessForTileAllocation`, `splitCondBranches`, `insertCopiesAtBranches` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 278-292
```cpp
/// A live range for a (collection of) tile values. A live range is built up of
/// non-overlapping intervals [start, end) which represent parts of the program
/// where a value in the range needs to be live (i.e. in an SME virtual tile).
/// Note that as the intervals are non-overlapping all values within a live
/// range can be allocated to the same SME virtual tile.
struct LiveRange {
  using RangeSet = llvm::IntervalMap<uint64_t, uint8_t, 16,
                                     llvm::IntervalMapHalfOpenInfo<unsigned>>;
  using Allocator = RangeSet::Allocator;
  // Dummy value for the IntervalMap. Only the keys matter (the intervals).
  static constexpr uint8_t kValidLiveRange = 0xff;

  LiveRange(Allocator &allocator)
      : ranges(std::make_unique<RangeSet>(allocator)) {}

```
- **EN**: Introduces declarations for `LiveRange`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LiveRange` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 293-312
```cpp
  /// Returns true if this range overlaps with `otherRange`.
  bool overlaps(LiveRange const &otherRange) const {
    return llvm::IntervalMapOverlaps<RangeSet, RangeSet>(*ranges,
                                                         *otherRange.ranges)
        .valid();
  }

  /// Returns true if this range is active at `point` in the program.
  bool overlaps(uint64_t point) const {
    return ranges->lookup(point) == kValidLiveRange;
  }

  /// Unions this live range with `otherRange`, aborts if the ranges overlap.
  void unionWith(LiveRange const &otherRange) {
    for (auto it = otherRange.ranges->begin(); it != otherRange.ranges->end();
         ++it)
      ranges->insert(it.start(), it.stop(), kValidLiveRange);
    values.set_union(otherRange.values);
  }

```
- **EN**: Implements logic around `overlaps`, `RangeSet>`, `valid`, `lookup`, and 4 more symbols.
- **CN**: 围绕 `overlaps`, `RangeSet>`, `valid`, `lookup`, and 4 more symbols 实现具体逻辑。

### Lines 313-326
```cpp
  /// Inserts an interval [start, end) for `value` into this range.
  void insert(Value value, unsigned start, unsigned end) {
    values.insert(value);
    if (start != end)
      ranges->insert(start, end, kValidLiveRange);
  }

  bool empty() const { return ranges->empty(); }
  unsigned start() const { return ranges->start(); }
  unsigned end() const { return ranges->stop(); }
  bool operator<(LiveRange const &other) const {
    return start() < other.start();
  }

```
- **EN**: Implements logic around `insert`, `empty`, `start`, `end`, and 1 more symbols.
- **CN**: 围绕 `insert`, `empty`, `start`, `end`, and 1 more symbols 实现具体逻辑。

### Lines 327-341
```cpp
  ArmSMETileType getTileType() const {
    return *getSMETileType(cast<VectorType>(values[0].getType()));
  }

  /// The values contained in this live range.
  SetVector<Value> values;

  /// A set of (non-overlapping) intervals that mark where any value in `values`
  /// is live.
  std::unique_ptr<RangeSet> ranges;

  /// The tile ID (or none) assigned to this live range.
  std::optional<unsigned> tileId;
};

```
- **EN**: Implements logic around `getTileType`, `getSMETileType`.
- **CN**: 围绕 `getTileType`, `getSMETileType` 实现具体逻辑。

### Lines 342-369
```cpp
/// Number operations within a function to allow computing live ranges.
/// Operations are numbered consecutively wihin blocks, and the blocks are
/// topologically sorted (using forward edges). This function is only correct if
/// all ArmSME have been converted to CF (which is asserted).
FailureOr<DenseMap<Operation *, unsigned>>
generateOperationNumbering(FunctionOpInterface function) {
  unsigned index = 0;
  SetVector<Block *> blocks =
      getBlocksSortedByDominance(function.getFunctionBody());
  DenseMap<Operation *, unsigned> operationToIndexMap;
  for (Block *block : blocks) {
    index++; // We want block args to have their own number.
    for (Operation &op : block->getOperations()) {
      WalkResult walkResult =
          op.walk([&](ArmSMETileOpInterface nestedOp) -> WalkResult {
            if (&op != nestedOp.getOperation())
              return WalkResult::interrupt();
            return WalkResult::advance();
          });
      if (walkResult.wasInterrupted()) {
        return op.emitError("ArmSME tile allocation requires flattened control "
                            "flow; run -convert-scf-to-cf before this pass "
                            "(e.g. via convert-arm-sme-to-llvm pipeline)");
      }
      operationToIndexMap.try_emplace(&op, index++);
    }
  }

```
- **EN**: Implements logic around `generateOperationNumbering`, `getBlocksSortedByDominance`, `getOperations`, `walk`, and 6 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `generateOperationNumbering`, `getBlocksSortedByDominance`, `getOperations`, `walk`, and 6 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 370-383
```cpp
  return operationToIndexMap;
}

/// Gather live ranges for SME tiles from the MLIR liveness analysis.
DenseMap<Value, LiveRange>
gatherTileLiveRanges(DenseMap<Operation *, unsigned> const &operationToIndexMap,
                     LiveRange::Allocator &liveRangeAllocator,
                     Liveness &liveness, FunctionOpInterface function) {
  assert(!operationToIndexMap.empty() && "expected operation numbering");
  DenseMap<Value, LiveRange> liveRanges;
  /// Defines or updates a live range for an SME tile value. Live-ins may update
  /// an existing live range (rather than define a new one). Note: If
  /// `liveAtBlockEntry` is true then `firstUseOrDef` is the first operation in
  /// the block.
```
- **EN**: Implements logic around `gatherTileLiveRanges`, `assert`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `gatherTileLiveRanges`, `assert` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 384-399
```cpp
  auto defineOrUpdateValueLiveRange = [&](Value value, Operation *firstUseOrDef,
                                          LivenessBlockInfo const &livenessInfo,
                                          bool liveAtBlockEntry = false) {
    if (!isValidSMETileVectorType(value.getType()))
      return;
    // Find or create a live range for `value`.
    auto [it, _] = liveRanges.try_emplace(value, liveRangeAllocator);
    LiveRange &valueLiveRange = it->second;
    auto *lastUseInBlock = livenessInfo.getEndOperation(value, firstUseOrDef);
    // Add the interval [firstUseOrDef, lastUseInBlock) to the live range.
    unsigned startOpIdx =
        operationToIndexMap.at(firstUseOrDef) + (liveAtBlockEntry ? -1 : 0);
    unsigned endOpIdx = operationToIndexMap.at(lastUseInBlock);
    valueLiveRange.insert(value, startOpIdx, endOpIdx);
  };

```
- **EN**: Implements logic around `isValidSMETileVectorType`, `try_emplace`, `getEndOperation`, `at`, and 1 more symbols.
- **CN**: 围绕 `isValidSMETileVectorType`, `try_emplace`, `getEndOperation`, `at`, and 1 more symbols 实现具体逻辑。

### Lines 400-416
```cpp
  for (Block &block : function.getBlocks()) {
    LivenessBlockInfo const *livenessInfo = liveness.getLiveness(&block);
    // Handle block arguments:
    for (Value argument : block.getArguments())
      defineOrUpdateValueLiveRange(argument, &block.front(), *livenessInfo,
                                   /*liveAtBlockEntry=*/true);
    // Handle live-ins:
    for (Value liveIn : livenessInfo->in())
      defineOrUpdateValueLiveRange(liveIn, &block.front(), *livenessInfo,
                                   /*liveAtBlockEntry=*/true);
    // Handle new definitions:
    for (Operation &op : block) {
      for (Value result : op.getResults())
        defineOrUpdateValueLiveRange(result, &op, *livenessInfo);
    }
  }

```
- **EN**: Implements logic around `getBlocks`, `getLiveness`, `getArguments`, `defineOrUpdateValueLiveRange`, and 2 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getBlocks`, `getLiveness`, `getArguments`, `defineOrUpdateValueLiveRange`, and 2 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 417-444
```cpp
  return liveRanges;
}

/// Iterate over all predecessor tile values to a (tile) block argument.
static void forEachPredecessorTileValue(BlockArgument blockArg,
                                        function_ref<void(Value)> callback) {
  Block *block = blockArg.getOwner();
  unsigned argNumber = blockArg.getArgNumber();
  for (Block *pred : block->getPredecessors()) {
    TypeSwitch<Operation *>(pred->getTerminator())
        .Case([&](cf::BranchOp branch) {
          Value predecessorOperand = branch.getDestOperands()[argNumber];
          callback(predecessorOperand);
        })
        .Case([&](cf::CondBranchOp condBranch) {
          if (condBranch.getFalseDest() == block) {
            Value predecessorOperand =
                condBranch.getFalseDestOperands()[argNumber];
            callback(predecessorOperand);
          }
          if (condBranch.getTrueDest() == block) {
            Value predecessorOperand =
                condBranch.getTrueDestOperands()[argNumber];
            callback(predecessorOperand);
          }
        });
  }
}
```
- **EN**: Implements logic around `forEachPredecessorTileValue`, `function_ref`, `getOwner`, `getArgNumber`, and 9 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `forEachPredecessorTileValue`, `function_ref`, `getOwner`, `getArgNumber`, and 9 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 445-466
```cpp

/// Coalesce live ranges where it would prevent unnecessary tile moves.
SmallVector<LiveRange *>
coalesceTileLiveRanges(DenseMap<Value, LiveRange> &initialLiveRanges) {
  DenseMap<Value, LiveRange *> liveRanges;
  for (auto &[value, liveRange] : initialLiveRanges) {
    liveRanges.insert({value, &liveRange});
  }

  // Merge the live ranges of values `a` and `b` into one (if they do not
  // overlap). After this, the values `a` and `b` will both point to the same
  // live range (which will contain multiple values).
  auto mergeValuesIfNonOverlapping = [&](Value a, Value b) {
    LiveRange *aLiveRange = liveRanges.at(a);
    LiveRange *bLiveRange = liveRanges.at(b);
    if (aLiveRange != bLiveRange && !aLiveRange->overlaps(*bLiveRange)) {
      aLiveRange->unionWith(*bLiveRange);
      for (Value value : bLiveRange->values)
        liveRanges[value] = aLiveRange;
    }
  };

```
- **EN**: Implements logic around `coalesceTileLiveRanges`, `insert`, `at`, `overlaps`, and 1 more symbols.
- **CN**: 围绕 `coalesceTileLiveRanges`, `insert`, `at`, `overlaps`, and 1 more symbols 实现具体逻辑。

### Lines 467-487
```cpp
  // Merge the live ranges of new definitions with their tile operands.
  auto unifyDefinitionsWithOperands = [&](Value value) {
    auto armSMEOp = value.getDefiningOp<ArmSMETileOpInterface>();
    if (!armSMEOp)
      return;
    for (auto operand : armSMEOp->getOperands()) {
      if (isValidSMETileVectorType(operand.getType()))
        mergeValuesIfNonOverlapping(value, operand);
    }
  };

  // Merge the live ranges of block arguments with their predecessors.
  auto unifyBlockArgumentsWithPredecessors = [&](Value value) {
    auto blockArg = dyn_cast<BlockArgument>(value);
    if (!blockArg)
      return;
    forEachPredecessorTileValue(blockArg, [&](Value predecessorTile) {
      mergeValuesIfNonOverlapping(blockArg, predecessorTile);
    });
  };

```
- **EN**: Implements logic around `getDefiningOp`, `getOperands`, `isValidSMETileVectorType`, `mergeValuesIfNonOverlapping`, and 2 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getDefiningOp`, `getOperands`, `isValidSMETileVectorType`, `mergeValuesIfNonOverlapping`, and 2 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 488-502
```cpp
  auto applyRule = [&](auto rule) {
    llvm::for_each(llvm::make_first_range(initialLiveRanges), rule);
  };

  // Unify as many live ranges as we can. This prevents unnecessary moves.
  applyRule(unifyBlockArgumentsWithPredecessors);
  applyRule(unifyDefinitionsWithOperands);

  // Remove duplicate live range entries.
  SetVector<LiveRange *> uniqueLiveRanges;
  for (auto [_, liveRange] : liveRanges) {
    if (!liveRange->empty())
      uniqueLiveRanges.insert(liveRange);
  }

```
- **EN**: Implements logic around `for_each`, `applyRule`, `empty`, `insert`.
- **CN**: 围绕 `for_each`, `applyRule`, `empty`, `insert` 实现具体逻辑。

### Lines 503-529
```cpp
  // Sort the new live ranges by starting point (ready for tile allocation).
  auto coalescedLiveRanges = uniqueLiveRanges.takeVector();
  llvm::sort(coalescedLiveRanges,
             [](LiveRange *a, LiveRange *b) { return *a < *b; });
  return std::move(coalescedLiveRanges);
}

/// Choose a live range to spill (via some heuristics). This picks either a live
/// range from `overlappingRanges`, or the new live range `newRange`.
template <typename OverlappingRangesIterator>
LiveRange *
chooseSpillUsingHeuristics(OverlappingRangesIterator overlappingRanges,
                           LiveRange *newRange) {
  // Heuristic: Spill trivially copyable operations (usually free).
  auto isTrivialSpill = [&](LiveRange &allocatedRange) {
    return isTileTypeGreaterOrEqual(allocatedRange.getTileType(),
                                    newRange->getTileType()) &&
           allocatedRange.values.size() == 1 &&
           isTriviallyCloneableTileOp(
               allocatedRange.values[0].getDefiningOp<ArmSMETileOpInterface>());
  };
  if (isTrivialSpill(*newRange))
    return newRange;
  auto trivialSpill = llvm::find_if(overlappingRanges, isTrivialSpill);
  if (trivialSpill != overlappingRanges.end())
    return &*trivialSpill;

```
- **EN**: Implements logic around `takeVector`, `sort`, `move`, `chooseSpillUsingHeuristics`, and 8 more symbols.
- **CN**: 围绕 `takeVector`, `sort`, `move`, `chooseSpillUsingHeuristics`, and 8 more symbols 实现具体逻辑。

### Lines 530-557
```cpp
  // Heuristic: Spill the range that ends last (with a compatible tile type).
  auto isSmallerTileTypeOrEndsEarlier = [](LiveRange &a, LiveRange &b) {
    return !isTileTypeGreaterOrEqual(a.getTileType(), b.getTileType()) ||
           a.end() < b.end();
  };
  LiveRange &latestEndingLiveRange =
      *llvm::max_element(overlappingRanges, isSmallerTileTypeOrEndsEarlier);
  if (!isSmallerTileTypeOrEndsEarlier(latestEndingLiveRange, *newRange))
    return &latestEndingLiveRange;
  return newRange;
}

/// Greedily allocate tile IDs to live ranges. Spill using simple heuristics.
void allocateTilesToLiveRanges(
    ArrayRef<LiveRange *> liveRangesSortedByStartPoint) {
  TileAllocator tileAllocator;
  // `activeRanges` = Live ranges that need to be in a tile at the
  // `currentPoint` in the program.
  SetVector<LiveRange *> activeRanges;
  // `inactiveRanges` = Live ranges that _do not_ need to be in a tile
  // at the `currentPoint` in the program but could become active again later.
  // An inactive section of a live range can be seen as a 'hole' in the live
  // range, where it is possible to reuse the live range's tile ID _before_ it
  // has ended. By identifying 'holes', the allocator can reuse tiles more
  // often, which helps avoid costly tile spills.
  SetVector<LiveRange *> inactiveRanges;
  for (LiveRange *nextRange : liveRangesSortedByStartPoint) {
    auto currentPoint = nextRange->start();
```
- **EN**: Implements logic around `isTileTypeGreaterOrEqual`, `end`, `max_element`, `isSmallerTileTypeOrEndsEarlier`, and 2 more symbols.
- **CN**: 围绕 `isTileTypeGreaterOrEqual`, `end`, `max_element`, `isSmallerTileTypeOrEndsEarlier`, and 2 more symbols 实现具体逻辑。

### Lines 558-585
```cpp
    // 1. Update the `activeRanges` at `currentPoint`.
    activeRanges.remove_if([&](LiveRange *activeRange) {
      // Check for live ranges that have expired.
      if (activeRange->end() <= currentPoint) {
        tileAllocator.releaseTileId(activeRange->getTileType(),
                                    *activeRange->tileId);
        return true;
      }
      // Check for live ranges that have become inactive.
      if (!activeRange->overlaps(currentPoint)) {
        tileAllocator.releaseTileId(activeRange->getTileType(),
                                    *activeRange->tileId);
        inactiveRanges.insert(activeRange);
        return true;
      }
      return false;
    });
    // 2. Update the `inactiveRanges` at `currentPoint`.
    inactiveRanges.remove_if([&](LiveRange *inactiveRange) {
      // Check for live ranges that have expired.
      if (inactiveRange->end() <= currentPoint) {
        return true;
      }
      // Check for live ranges that have become active.
      if (inactiveRange->overlaps(currentPoint)) {
        tileAllocator.acquireTileId(inactiveRange->getTileType(),
                                    *inactiveRange->tileId);
        activeRanges.insert(inactiveRange);
```
- **EN**: Implements logic around `remove_if`, `end`, `releaseTileId`, `overlaps`, and 2 more symbols.
- **CN**: 围绕 `remove_if`, `end`, `releaseTileId`, `overlaps`, and 2 more symbols 实现具体逻辑。

### Lines 586-604
```cpp
        return true;
      }
      return false;
    });

    // 3. Collect inactive live ranges that overlap with the new live range.
    // Note: The overlap checks in steps 1 and 2 only look at the `currentPoint`
    // whereas this checks if there is an overlap at any future point too.
    SmallVector<LiveRange *> overlappingInactiveRanges;
    for (LiveRange *inactiveRange : inactiveRanges) {
      if (inactiveRange->overlaps(*nextRange)) {
        // We need to reserve the tile IDs of overlapping inactive ranges to
        // prevent two (overlapping) live ranges from getting the same tile ID.
        tileAllocator.acquireTileId(inactiveRange->getTileType(),
                                    *inactiveRange->tileId);
        overlappingInactiveRanges.push_back(inactiveRange);
      }
    }

```
- **EN**: Implements logic around `overlaps`, `acquireTileId`, `push_back`.
- **CN**: 围绕 `overlaps`, `acquireTileId`, `push_back` 实现具体逻辑。

### Lines 605-632
```cpp
    // 4. Allocate a tile ID to `nextRange`.
    auto rangeTileType = nextRange->getTileType();
    auto tileId = tileAllocator.allocateTileId(rangeTileType);
    if (succeeded(tileId)) {
      nextRange->tileId = *tileId;
    } else {
      // Create an iterator over all overlapping live ranges.
      auto allOverlappingRanges = llvm::concat<LiveRange>(
          llvm::make_pointee_range(activeRanges.getArrayRef()),
          llvm::make_pointee_range(overlappingInactiveRanges));
      // Choose an overlapping live range to spill.
      LiveRange *rangeToSpill =
          chooseSpillUsingHeuristics(allOverlappingRanges, nextRange);
      if (rangeToSpill != nextRange) {
        // Spill an (in)active live range (so release its tile ID first).
        tileAllocator.releaseTileId(rangeToSpill->getTileType(),
                                    *rangeToSpill->tileId);
        // This will always succeed after a spill (of an active live range).
        nextRange->tileId = *tileAllocator.allocateTileId(rangeTileType);
        // Remove the live range from the active/inactive sets.
        if (!activeRanges.remove(rangeToSpill)) {
          bool removed = inactiveRanges.remove(rangeToSpill);
          assert(removed && "expected a range to be removed!");
          (void)removed;
        }
      }
      rangeToSpill->tileId = tileAllocator.allocateInMemoryTileId();
    }
```
- **EN**: Implements logic around `getTileType`, `allocateTileId`, `succeeded`, `concat`, and 6 more symbols.
- **CN**: 围绕 `getTileType`, `allocateTileId`, `succeeded`, `concat`, and 6 more symbols 实现具体逻辑。

### Lines 633-646
```cpp

    // 5. Insert the live range into the active ranges.
    if (nextRange->tileId < kInMemoryTileIdBase)
      activeRanges.insert(nextRange);

    // 6. Release tiles reserved for inactive live ranges (in step 3).
    for (LiveRange *range : overlappingInactiveRanges) {
      if (*range->tileId < kInMemoryTileIdBase)
        tileAllocator.releaseTileId(range->getTileType(), *range->tileId);
    }
  }
}

/// Assigns a tile ID to an MLIR value.
```
- **EN**: Implements logic around `insert`, `releaseTileId`.
- **CN**: 围绕 `insert`, `releaseTileId` 实现具体逻辑。

### Lines 647-660
```cpp
void assignTileIdToValue(IRRewriter &rewriter, Value value,
                         IntegerAttr tileIdAttr) {
  if (auto tileOp = value.getDefiningOp<ArmSMETileOpInterface>())
    rewriter.modifyOpInPlace(tileOp, [&] { tileOp.setTileId(tileIdAttr); });
  for (Operation *user : value.getUsers()) {
    if (auto tileOp = dyn_cast<ArmSMETileOpInterface>(user)) {
      // Ensure ArmSME ops that don't produce a value still get a tile ID.
      if (!hasTileResult(tileOp))
        rewriter.modifyOpInPlace(tileOp, [&] { tileOp.setTileId(tileIdAttr); });
    }
  }
}

/// Assign tile IDs back to IR and attempt to resolve trivial tile ID conflicts.
```
- **EN**: Implements logic around `assignTileIdToValue`, `getDefiningOp`, `modifyOpInPlace`, `getUsers`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `assignTileIdToValue`, `getDefiningOp`, `modifyOpInPlace`, `getUsers`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 661-681
```cpp
LogicalResult assignTileIdsAndResolveTrivialConflicts(
    IRRewriter &rewriter, FunctionOpInterface function,
    ArrayRef<LiveRange *> allocatedLiveRanges) {
  for (LiveRange const *liveRange : allocatedLiveRanges) {
    auto tileIdAttr = rewriter.getI32IntegerAttr(*liveRange->tileId);
    auto isAllocatedToSameTile = [&](Value value) {
      if (auto tileOp = value.getDefiningOp<ArmSMETileOpInterface>();
          tileOp && tileOp.getTileId() == tileIdAttr)
        return true;
      return liveRange->values.contains(value);
    };

    /// Eliminates copies where the operand has the same tile ID.
    auto foldRedundantCopies = [&](Value value) -> LogicalResult {
      auto copyOp = value.getDefiningOp<CopyTileOp>();
      if (!copyOp || !isAllocatedToSameTile(copyOp.getTile()))
        return failure();
      rewriter.replaceAllUsesWith(copyOp, copyOp.getTile());
      return success();
    };

```
- **EN**: Implements logic around `assignTileIdsAndResolveTrivialConflicts`, `getI32IntegerAttr`, `getDefiningOp`, `getTileId`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `assignTileIdsAndResolveTrivialConflicts`, `getI32IntegerAttr`, `getDefiningOp`, `getTileId`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子。

### Lines 682-703
```cpp
    /// Validates each predecessor to a tile block argument has been assigned
    /// the same tile ID.
    auto validateBlockArguments = [&](Value value) {
      auto blockArg = dyn_cast<BlockArgument>(value);
      if (!blockArg) {
        // Not a block argument (nothing to validate).
        return success();
      }
      bool tileMismatch = false;
      forEachPredecessorTileValue(blockArg, [&](Value predecessorTile) {
        if (tileMismatch)
          return;
        if (!isAllocatedToSameTile(predecessorTile)) {
          blockArg.getOwner()->getParentOp()->emitOpError(
              "block argument not allocated to the same SME virtial tile as "
              "predecessors");
          tileMismatch = true;
        }
      });
      return success(/*isSuccess=*/!tileMismatch);
    };

```
- **EN**: Implements logic around `dyn_cast`, `success`, `forEachPredecessorTileValue`, `isAllocatedToSameTile`, and 1 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `dyn_cast`, `success`, `forEachPredecessorTileValue`, `isAllocatedToSameTile`, and 1 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 704-731
```cpp
    /// Attempts to resolve (trivial) tile ID conflicts.
    auto resolveTrivialTileConflicts = [&](Value value) -> LogicalResult {
      auto tileOp = value.getDefiningOp<ArmSMETileOpInterface>();
      OpOperand *tileOperand = getTileOpOperand(tileOp);
      if (!tileOperand || isAllocatedToSameTile(tileOperand->get())) {
        // Operand already allocated to the correct tile.
        // No conflict to resolve.
        return success();
      }
      auto operandTileOp =
          tileOperand->get().getDefiningOp<ArmSMETileOpInterface>();
      if (!isTriviallyCloneableTileOp(operandTileOp)) {
        auto error =
            tileOp.emitOpError("tile operand allocated to different SME "
                               "virtial tile (move required)");
        error.attachNote(tileOperand->get().getLoc())
            << "tile operand is: " << tileOperand->get();
        return error;
      }
      // Cloning prevents a move/spill (though may require recomputation).
      rewriter.setInsertionPoint(tileOp);
      auto clonedOp = operandTileOp.clone();
      rewriter.modifyOpInPlace(clonedOp,
                               [&] { clonedOp.setTileId(tileOp.getTileId()); });
      rewriter.insert(clonedOp);
      if (isa<CopyTileOp>(tileOp)) {
        rewriter.replaceAllUsesWith(tileOp->getResult(0),
                                    clonedOp->getResult(0));
```
- **EN**: Implements logic around `getDefiningOp`, `getTileOpOperand`, `isAllocatedToSameTile`, `success`, and 13 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getDefiningOp`, `getTileOpOperand`, `isAllocatedToSameTile`, `success`, and 13 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 732-746
```cpp
      } else {
        rewriter.modifyOpInPlace(
            tileOp, [&] { tileOperand->assign(clonedOp->getResult(0)); });
      }
      return success();
    };

    for (Value value : liveRange->values) {
      // 1. Assign the tile ID to the value.
      assignTileIdToValue(rewriter, value, tileIdAttr);

      // 2. Attempt to eliminate redundant tile copies.
      if (succeeded(foldRedundantCopies(value)))
        continue;

```
- **EN**: Implements logic around `modifyOpInPlace`, `assign`, `success`, `assignTileIdToValue`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `modifyOpInPlace`, `assign`, `success`, `assignTileIdToValue`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子。

### Lines 747-774
```cpp
      // 3. Validate tile block arguments.
      if (failed(validateBlockArguments(value)))
        return failure();

      // 4. Attempt to resolve (trivial) tile ID conflicts.
      if (failed(resolveTrivialTileConflicts(value)))
        return failure();
    }
  }
  return success();
}

/// Prints live ranges alongside operation names for debugging.
void dumpLiveRanges(DenseMap<Operation *, unsigned> const &operationToIndexMap,
                    ArrayRef<LiveRange const *> liveRanges,
                    FunctionOpInterface function) {
  llvm::errs() << "SME Tile Liveness: @" << function.getName()
               << "\nKey:\nS - Start\nE - End\n| - Live\n";
  for (auto [blockIdx, block] : llvm::enumerate(function.getBlocks())) {
    llvm::errs() << "^bb" << blockIdx << ":\n";
    for (Operation &op : block.getOperations()) {
      unsigned operationIndex = operationToIndexMap.at(&op);
      for (LiveRange const *range : liveRanges) {
        char liveness = ' ';
        for (auto it = range->ranges->begin(); it != range->ranges->end();
             ++it) {
          if (it.start() == operationIndex)
            liveness = (liveness == 'E' ? '|' : 'S');
```
- **EN**: Implements logic around `failed`, `failure`, `success`, `dumpLiveRanges`, and 6 more symbols; this block implements verifier, folding, parsing, or printing hooks; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `failed`, `failure`, `success`, `dumpLiveRanges`, and 6 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并处理 MLIR region、block 或控制流边。

### Lines 775-788
```cpp
          else if (it.stop() == operationIndex)
            liveness = (liveness == 'S' ? '|' : 'E');
          else if (operationIndex >= it.start() && operationIndex < it.stop())
            liveness = '|';
        }
        llvm::errs() << liveness;
      }
      llvm::errs() << ' ' << op.getName() << '\n';
    }
  }
  llvm::errs() << "==========\n";
}

struct TestTileAllocationPass
```
- **EN**: Introduces declarations for `TestTileAllocationPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `TestTileAllocationPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 789-802
```cpp
    : public arm_sme::impl::TestTileAllocationBase<TestTileAllocationPass> {
  using TestTileAllocationBase::TestTileAllocationBase;
  void runOnOperation() override {
    FunctionOpInterface function = getOperation();
    if (preprocessOnly) {
      IRRewriter rewriter(function);
      return preprocessForTileAllocation(rewriter, function);
    }
    if (failed(arm_sme::allocateSMETiles(function, dumpTileLiveRanges)))
      signalPassFailure();
  }
};
} // namespace

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `rewriter`, `preprocessForTileAllocation`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; participates in pass execution or pass construction.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `rewriter`, `preprocessForTileAllocation`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并参与 pass 执行或 pass 构造。

### Lines 803-826
```cpp
LogicalResult mlir::arm_sme::allocateSMETiles(FunctionOpInterface function,
                                              bool dumpRanges) {
  if (function.empty()) {
    // TODO: Also return early if the function contains no ArmSME ops?
    return success();
  }

  LiveRange::Allocator liveRangeAllocator;
  IRRewriter rewriter(function.getContext());

  // 1. Preprocess the IR for tile allocation.
  preprocessForTileAllocation(rewriter, function);

  // 2. Gather live ranges for each ArmSME tile within the function.
  Liveness liveness(function);
  auto maybeOperationToIndexMap = generateOperationNumbering(function);
  if (failed(maybeOperationToIndexMap))
    return failure();
  auto &operationToIndexMap = *maybeOperationToIndexMap;
  auto initialLiveRanges = gatherTileLiveRanges(
      operationToIndexMap, liveRangeAllocator, liveness, function);
  if (initialLiveRanges.empty())
    return success();

```
- **EN**: Implements logic around `allocateSMETiles`, `empty`, `success`, `rewriter`, and 6 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `allocateSMETiles`, `empty`, `success`, `rewriter`, and 6 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 827-844
```cpp
  if (dumpRanges) {
    // Wrangle initial live ranges into a form suitable for printing.
    auto nonEmpty = llvm::make_filter_range(
        llvm::make_second_range(initialLiveRanges),
        [&](LiveRange const &liveRange) { return !liveRange.empty(); });
    auto initialRanges = llvm::map_to_vector(
        nonEmpty, [](LiveRange const &liveRange) { return &liveRange; });
    llvm::sort(initialRanges,
               [](LiveRange const *a, LiveRange const *b) { return *a < *b; });
    llvm::errs() << "\n========== Initial Live Ranges:\n";
    dumpLiveRanges(operationToIndexMap, initialRanges, function);
  }

  // 3. Coalesce (non-overlapping) live ranges where it would be beneficial
  // for tile allocation. E.g. Unify the result of an operation with its
  // operands.
  auto coalescedLiveRanges = coalesceTileLiveRanges(initialLiveRanges);

```
- **EN**: Implements logic around `make_filter_range`, `make_second_range`, `empty`, `map_to_vector`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `make_filter_range`, `make_second_range`, `empty`, `map_to_vector`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 845-858
```cpp
  if (dumpRanges) {
    llvm::errs() << "\n========== Coalesced Live Ranges:\n";
    dumpLiveRanges(operationToIndexMap, coalescedLiveRanges, function);
  }

  // 4. Allocate tile IDs to live ranges.
  allocateTilesToLiveRanges(coalescedLiveRanges);

  // 5. Assign the tile IDs back to the ArmSME operations.
  if (failed(assignTileIdsAndResolveTrivialConflicts(rewriter, function,
                                                     coalescedLiveRanges))) {
    return failure();
  }

```
- **EN**: Implements logic around `errs`, `dumpLiveRanges`, `allocateTilesToLiveRanges`, `failed`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `errs`, `dumpLiveRanges`, `allocateTilesToLiveRanges`, `failed`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 859-863
```cpp
  // 6. Erase trivially dead tile operations (e.g. a ZeroOp with no
  // users). This prevents the LLVM conversion needlessly inserting spills.
  eraseTriviallyDeadTileOps(rewriter, function);
  return success();
}
```
- **EN**: Implements logic around `eraseTriviallyDeadTileOps`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `eraseTriviallyDeadTileOps`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Analysis/Liveness.h`, `mlir/Analysis/TopologicalSortUtils.h`, `mlir/Dialect/ArmSME/IR/ArmSME.h`, `mlir/Dialect/ArmSME/Transforms/Passes.h`, `mlir/Dialect/ArmSME/Transforms/Transforms.h`, `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `llvm/ADT/IntervalMap.h`, `llvm/ADT/SmallVectorExtras.h`, `llvm/ADT/TypeSwitch.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (6), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (3), MLIR analysis interfaces / MLIR 分析接口 (2)
