# DestinationStyleOpInterface.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Interfaces/DestinationStyleOpInterface.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR DestinationStyleOpInterface component, including operation metadata, traits, constraints, and textual assembly rules.
- **用途（CN）**: 为 MLIR 的 DestinationStyleOpInterface 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
````tablegen
//===- DestinationStyleOpInterface.td ----------------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DESTINATIONSTYLEOPINTERFACE
#define MLIR_DESTINATIONSTYLEOPINTERFACE

include "mlir/IR/OpBase.td"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 14-83
````tablegen
def DestinationStyleOpInterface : OpInterface<"DestinationStyleOpInterface"> {
  let description = [{
    Ops that are in destination style have designated "init" operands, which act
    as initial tensor values for the results of the operation or the init
    buffers to which the results of the op will be written.

    Init operands must be tensors or memrefs. Input operands can have any type.
    All non-init operands are DPS inputs.

    The init operands of this op are specified by the MutableOperandRange that
    the `getDpsInitsMutable` interface methods returns. This implies that the
    init operands must be a consecutive range of operands.

    Each tensor init operand is tied to a corresponding tensor OpResult in a
    1-to-1 fashion. The i-th init tensor is tied to the i-th OpResult. The op
    may not have any additional OpResults. Init operands and their tied
    OpResults have the same type. Dynamic dimension sizes also match at runtime.

    Note: This implies that a destination style op without any tensor inits must
    not have any OpResults.

    An op has "pure tensor semantics" if it has at least one tensor operand and
    no buffer (memref) operands. It has "pure buffer semantics" if it has at
    least one buffer (memref) operand and no tensor operands.

    Destination-passing style abstraction makes certain transformations easier.
    For example, tiling implementation can extract/insert slices from/into the
    destination of an op and use the resulting shaped value as an iter_arg in
    the surrounding loop structure. As another example, bufferization does not
    have to allocate new buffers for destinations (in case of in-place
    bufferization) and can directly reuse the existing destination buffer.

    Example of a destination style op: `%r = tensor.insert_slice %t into %d`,
    where `%t` is the single input and `%d` is the single init. `%d` is tied
    to `%r`.

    Example of an op that is not in destination style: `%r = tensor.pad %t`.
    This op is not in destination style because `%r` and `%t` have different
    shape.
  }];

  let cppNamespace = "::mlir";

  let methods = [
    InterfaceMethod<
      /*desc=*/"Return start and end indices of the init operands range.",
      /*retTy=*/"::mlir::MutableOperandRange",
      /*methodName=*/"getDpsInitsMutable",
      /*args=*/(ins)
    >,
  ];

  let extraSharedClassDeclaration = [{
    ::mlir::OperandRange getDpsInits() {
      return $_op.getDpsInitsMutable();
    }

    /// Return the number of DPS inits.
    int64_t getNumDpsInits() { return $_op.getDpsInits().size(); }

    /// Return the `i`-th DPS init.
    ::mlir::OpOperand *getDpsInitOperand(int64_t i) {
      return &$_op.getDpsInitsMutable()[i];
    }

    /// Set the `i`-th DPS init.
    void setDpsInitOperand(int64_t i, Value value) {
      assert(i >= 0 && i < $_op.getNumDpsInits() && "invalid index");
      $_op->setOperand($_op.getDpsInits().getBeginOperandIndex() + i, value);
    }
````
- **EN**: This TableGen block defines `DestinationStyleOpInterface` as a `def` record for `DestinationStyleOpInterface`. It covers semantic documentation, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `DestinationStyleOpInterface` 定义为 `def` 记录，用于描述 `DestinationStyleOpInterface` 相关的声明式信息。 其中涉及 语义文档, trait/接口组合。

### Lines 84-152
````tablegen
    /// Return the number of DPS inputs.
    int64_t getNumDpsInputs() {
      return $_op->getNumOperands() - $_op.getNumDpsInits();
    }

    /// Return the DPS input operands.
    ::llvm::SmallVector<::mlir::OpOperand *> getDpsInputOperands() {
      ::llvm::SmallVector<::mlir::OpOperand *> result;
      int64_t numOperands = $_op->getNumOperands();
      ::mlir::OperandRange range = $_op.getDpsInits();
      if (range.empty()) {
        result.reserve(numOperands);
        for (int64_t i = 0; i < numOperands; ++i)
          result.push_back(&$_op->getOpOperand(i));
        return result;
      }
      int64_t firstInitPos = range.getBeginOperandIndex();
      int64_t numInits = range.size();
      result.reserve(numOperands - numInits);
      for (int64_t i = 0; i < firstInitPos; ++i)
        result.push_back(&$_op->getOpOperand(i));
      for (int64_t i = firstInitPos + numInits; i < numOperands; ++i)
        result.push_back(&$_op->getOpOperand(i));
      return result;
    }

    /// Return the DPS input operands.
    ::llvm::SmallVector<::mlir::Value> getDpsInputs() {
      return ::llvm::to_vector(::llvm::map_range(
          $_op.getDpsInputOperands(), [](OpOperand *o) { return o->get(); }));
    }

    /// Return the `i`-th DPS input operand.
    ::mlir::OpOperand *getDpsInputOperand(int64_t i) {
      ::mlir::OperandRange range = $_op.getDpsInits();
      if (range.empty())
        return &$_op->getOpOperand(i);
      int64_t firstInitPos = range.getBeginOperandIndex();
      int64_t numInits = range.size();
      assert(i >= 0 && i < $_op->getNumOperands() - numInits
             && "invalid index");
      return &$_op->getOpOperand(
          i < firstInitPos ? i : i + firstInitPos + numInits);
    }

    /// Return "true" if `opOperand` is an "input".
    bool isDpsInput(::mlir::OpOperand *opOperand) {
      assert(opOperand->getOwner() == $_op && "invalid operand");
      return !$_op.isDpsInit(opOperand);
    }

    /// Return "true" if `opOperand` is an "init".
    bool isDpsInit(::mlir::OpOperand *opOperand) {
      assert(opOperand->getOwner() == $_op && "invalid operand");
      ::mlir::OperandRange range = $_op.getDpsInits();
      if (range.empty())
        return false;
      auto operandNumber = opOperand->getOperandNumber();
      return operandNumber >= range.getBeginOperandIndex()
          && operandNumber < range.getBeginOperandIndex() + range.size();
    }

    /// Return "true" if `opOperand` is a scalar value. A sclar is defined as
    /// neither a MemRef nor a tensor value.
    bool isScalar(::mlir::OpOperand *opOperand) {
      assert(opOperand->getOwner() == $_op && "invalid operand");
      return !::llvm::isa<BaseMemRefType, TensorType>(
          opOperand->get().getType());
    }
````
- **EN**: This block groups callable interfaces such as `getNumDpsInputs`, `getNumOperands`, `getNumDpsInits`, `getDpsInputOperands`, indicating how `DestinationStyleOpInterface` is queried or updated.
- **CN**: 该代码块聚合了 `getNumDpsInputs`, `getNumOperands`, `getNumDpsInits`, `getDpsInputOperands` 等可调用接口，展示了如何查询或更新 `DestinationStyleOpInterface`。

### Lines 154-206
````tablegen
    /// Return the OpResult that is tied to the given OpOperand.
    ::mlir::OpResult getTiedOpResult(::mlir::OpOperand *opOperand) {
        assert(opOperand->getOwner() == $_op && "invalid operand");
        ::mlir::OperandRange range = $_op.getDpsInits();
        assert(!range.empty() && "op has no inits");
        int64_t resultIndex =
            opOperand->getOperandNumber() - range.getBeginOperandIndex();
        assert(resultIndex >= 0 &&
               resultIndex < $_op->getNumResults());
        return $_op->getResult(resultIndex);
    }

    /// Return the OpOperand that is tied to the given OpResult.
    ::mlir::OpOperand *getTiedOpOperand(::mlir::OpResult opResult) {
      assert(opResult.getDefiningOp() == $_op && "invalid opresult");
      return $_op.getDpsInitOperand(opResult.getResultNumber());
    }

    /// Return whether the op has pure buffer semantics. That is the case if the
    /// op has no tensor operands and at least one memref operand.
    bool hasPureBufferSemantics() {
      // No tensors.
      auto isTensor = [](Value v){
        return ::llvm::isa<::mlir::TensorType>(v.getType());
      };
      if (::llvm::any_of($_op->getOperands(), isTensor))
        return false;
      // At least one memref.
      auto isMemref = [](Value v){
        return ::llvm::isa<::mlir::BaseMemRefType>(v.getType());
      };
      return llvm::any_of($_op->getOperands(), isMemref);
    }

    /// Return whether the op has pure tensor semantics. That is the case if the
    /// op has no memref operands and at least one tensor operand.
    bool hasPureTensorSemantics() {
      // No memrefs.
      auto isMemref = [](Value v){
        return ::llvm::isa<::mlir::BaseMemRefType>(v.getType());
      };
      if (::llvm::any_of($_op->getOperands(), isMemref))
        return false;
      // At least one tensor.
      auto isTensor = [](Value v){
        return ::llvm::isa<::mlir::TensorType>(v.getType());
      };
      return llvm::any_of($_op->getOperands(), isTensor);    }
  }];

  let verify = [{ return detail::verifyDestinationStyleOpInterface($_op); }];
  let verifyWithRegions = 1;
}
````
- **EN**: This block groups callable interfaces such as `getTiedOpResult`, `getOwner`, `getDpsInits`, `empty`, indicating how `DestinationStyleOpInterface` is queried or updated.
- **CN**: 该代码块聚合了 `getTiedOpResult`, `getOwner`, `getDpsInits`, `empty` 等可调用接口，展示了如何查询或更新 `DestinationStyleOpInterface`。

### Lines 208-208
````tablegen
#endif // MLIR_DESTINATIONSTYLEOPINTERFACE
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Trait and interface composition
  **CN**: trait 与接口组合
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/IR/OpBase.td
- DestinationStyleOpInterface builds on OpInterface<"DestinationStyleOpInterface">
