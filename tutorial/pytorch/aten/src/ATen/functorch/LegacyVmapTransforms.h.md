# LegacyVmapTransforms.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/LegacyVmapTransforms.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `LegacyVmapTransforms.h`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Descriptor/handle lifecycle management is important here. Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `LegacyVmapTransforms.h` 展开。 文件头部注释也概括了其核心职责。 描述符/句柄的生命周期管理是这里的重要内容。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11 / 第 1-11 行

```cpp
0001: // Copyright (c) Facebook, Inc. and its affiliates.
0002: // All rights reserved.
0003: //
0004: // This source code is licensed under the BSD-style license found in the
0005: // LICENSE file in the root directory of this source tree.
0006: 
0007: #pragma once
0008: 
0009: #include <ATen/functorch/Macros.h>
0010: #include <ATen/functorch/BatchedTensorImpl.h>
0011: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 12-31 / 第 12-31 行

```cpp
0012: namespace at::functorch {
0013: 
0014: // This files contains the legacy (now-deprecated) batching rule API.
0015: // Please try to use the new-style batching rule API (see writing_batch_rules.md)
0016: 
0017: // This file contains abstractions used for transforming *logical* vmap arguments
0018: // into *physical* arguments. (Keep reading for definitions of these terms).
0019: 
0020: // NOTE: [Logical vs physical args]
0021: // Consider the following vmap.
0022: //   vmap(vmap(func, in_dims=(2,)), in_dims=(0,))(torch.ones(2, 3, 4))
0023: // This would produce a BatchedTensor wrapping a Tensor of size [2, 3, 4],
0024: // with batch dims 0 and 2:
0025: //   BatchedTensor(ones(2, 3, 4), bdims=[(lvl=1,dim=0),(lvl=2,dim=2)])
0026: //
0027: // We say the *logical* view of the tensor has size [3] -- tensors inside
0028: // `func` appear to have size [3].
0029: // However, the *physical* underlying tensor (the one passed to vmap) has size
0030: // [2, 3, 4].
0031: //
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 32-44 / 第 32-44 行

```cpp
0032: // This notion of logical vs physical also extends to non-tensor arguments.
0033: // Consider the previous tensor; let's assume the user called
0034: // `torch.sum(tensor, dim=0)` inside of `func`. Then the logical
0035: // dimension they are reducing over is dim 0 but the physical dim is dim 1
0036: // (the first non-batch dimension)
0037: 
0038: // Forward declared; see NOTE: [What is a VmapPhysicalView?]
0039: struct VmapPhysicalView;
0040: 
0041: // Most PyTorch operators take 4 or fewer inputs.
0042: constexpr int64_t kVmapTransformStaticInputSize = 4;
0043: using VmapPhysicalViewVec = SmallVector<VmapPhysicalView, kVmapTransformStaticInputSize>;
0044: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `VmapPhysicalView`, `VmapPhysicalViewVec`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`VmapPhysicalView`, `VmapPhysicalViewVec`。

### Lines 45-59 / 第 45-59 行

```cpp
0045: // Pytorch generally advertises good performance for <= 5 dims.
0046: // (see ATen/core/DimVector.h). We add a few extra dims (~3) for vmap
0047: // dimensions to get 8. Adjust this number as necessary
0048: constexpr int64_t kVmapStaticDimVecSize = 8;
0049: using VmapDimVector = SmallVector<int64_t, kVmapStaticDimVecSize>;
0050: using VmapSymDimVector = SmallVector<c10::SymInt, kVmapStaticDimVecSize>;
0051: 
0052: // NOTE: [What is an VmapTransform?]
0053: // An *VmapTransform* converts logical views of tensors to physical views.
0054: //
0055: // Batching rules use VmapTransforms to convert logical arguments to
0056: // physical arguments, then call one or more at:: operator that handles the
0057: // physical arguments, and then converts the physical result back to a logical
0058: // argument.
0059: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; supports transform-aware functorch semantics. Key symbols: `VmapDimVector`, `VmapSymDimVector`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；支持面向变换的 functorch 语义。关键符号：`VmapDimVector`, `VmapSymDimVector`。

### Lines 60-69 / 第 60-69 行

```cpp
0060: // VmapTransform for operators that take tensors with multiple batch dims.
0061: // Given one or more logical views on Tensors, `logicalToPhysical`
0062: // permutes all of the batch dims to the front of the tensor, aligns
0063: // and expands the batch dims to match each other (according to their `level`),
0064: // and returns a VmapPhysicalView on the tensor(s).
0065: struct TORCH_API MultiBatchVmapTransform {
0066:   static VmapPhysicalView logicalToPhysical(const Tensor& logical_tensor);
0067:   static VmapPhysicalViewVec logicalToPhysical(ITensorListRef logical_tensors);
0068: };
0069: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `MultiBatchVmapTransform`, `logicalToPhysical`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`MultiBatchVmapTransform`, `logicalToPhysical`。

### Lines 70-89 / 第 70-89 行

```cpp
0070: // VmapTransform for operators that broadcast all inputs.
0071: // Given some logical views on Tensors, `logicalToPhysical`:
0072: // - permutes all of the batch dims to the front of the tensors
0073: // - aligns all the batch dims to the collective levels of all of the tensors.
0074: //   If a tensor does not have a batch dim for a vmap level, then it receives
0075: //   a size-one dimension for said level.
0076: // - aligns the non-batch dims to have the same dimensionality, adding extra
0077: //   size-1 dimensions in between the batch dimensions and the non-batch dimensions
0078: //   so that the batch dimensions are lined up from the right.
0079: //
0080: // For example: given inputs of size (B, 2) and (B, 3, 2) where B is the batch
0081: // dimension, BroadcastingVmapTransform returns VmapPhysicalViews that wrap tensors
0082: // of size (B, 1, 2) and (B, 3, 2).
0083: //
0084: // Given inputs of size (B, 2) and (2,), BroadcastingVmapTransform returns
0085: // VmapPhysicalViews wrapping tensors of size (B, 2) and (1, 2). We don't
0086: // actually *need* to return a tensor of size (1, 2) for the second tensor
0087: // because the broadcasting operation takes care of that for us, but we do
0088: // it anyways to keep things simple.
0089: struct TORCH_API BroadcastingVmapTransform {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `BroadcastingVmapTransform`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`BroadcastingVmapTransform`。

### Lines 90-109 / 第 90-109 行

```cpp
0090:   static VmapPhysicalViewVec logicalToPhysical(TensorList logical_tensors);
0091: };
0092: 
0093: // Forward declared, if you're reading this file head to toe, don't worry about
0094: // it yet.
0095: struct VmapPhysicalToLogicalMap;
0096: 
0097: // NOTE: [What is a VmapPhysicalView?]
0098: // VmapPhysicalView represents a physical view on a Tensor.
0099: //
0100: // One can use it to further convert logical dimension indices, logical shapes,
0101: // and more to their physical variants, or convert a new (physical) tensor into
0102: // a logical BatchedTensor. (TODO(rzou): some of these are not yet implemented).
0103: //
0104: // VmapPhysicalView stores a physical tensor with all of its batch dimensions at
0105: // the front and some levels that correspond to said batch dimensions.
0106: //
0107: // The levels bitset specifies which vmap levels correspond to the batch
0108: // dimensions at the front of the tensor. In particular, the number of set bits
0109: // corresponds to the number of batch dimensions on `tensor` and the rightmost
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `VmapPhysicalToLogicalMap`, `logicalToPhysical`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`VmapPhysicalToLogicalMap`, `logicalToPhysical`。

### Lines 110-121 / 第 110-121 行

```cpp
0110: // bit of `levels` specifies the maximum number of nested vmaps we are in at
0111: // this point in time.
0112: // For example, given:
0113: //   physical_view = VmapPhysicalView(tensor=ones(2, 3, 4, 5, 6), levels={1, 3})
0114: //
0115: // Rightmost bit of `levels` is 3 indicating the number of nested vmaps less
0116: // than or equal to 3.
0117: //   bitset: 010100
0118: //              ^
0119: //              |
0120: //   levels: 012345
0121: struct TORCH_API VmapPhysicalView {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `VmapPhysicalView`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`VmapPhysicalView`。

### Lines 122-141 / 第 122-141 行

```cpp
0122:   VmapPhysicalView(Tensor&& tensor, std::bitset<kVmapNumLevels> levels)
0123:       : levels_(levels), tensor_(std::move(tensor)) {
0124:     // TORCH_INTERNAL_ASSERT(!isBatchedTensor(tensor));
0125:   }
0126: 
0127:   Tensor& tensor() { return tensor_; }
0128:   const Tensor& tensor() const { return tensor_; }
0129: 
0130:   // Maps logical dim indices to physical dim indices. Also does dim wrapping.
0131:   //
0132:   // For example, given:
0133:   //   physical_view = VmapPhysicalView(tensor=ones(2, 3, 4, 5), levels={1, 3})
0134:   //
0135:   // Then physical_view.getPhysicalDims({0, 1}) returns {2, 3}.
0136:   // This is because the size of levels tell us that the first two dimensions
0137:   // of `tensor_` are batch dimensions, so a logical dim of `n` is actually
0138:   // a physical dim of `n + 2`.
0139:   VmapDimVector getPhysicalDims(IntArrayRef logical_dims) const;
0140:   int64_t getPhysicalDim(int64_t logical_dim) const;
0141: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `VmapPhysicalView`, `tensor`, `getPhysicalDims`, `getPhysicalDim`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`VmapPhysicalView`, `tensor`, `getPhysicalDims`, `getPhysicalDim`。

### Lines 142-152 / 第 142-152 行

```cpp
0142:   // Returns a VmapPhysicalToLogicalMap object. This can be used for
0143:   // mapping a physical tensor to a new logical tensor (BatchedTensor)
0144:   VmapPhysicalToLogicalMap getPhysicalToLogicalMap() const;
0145: 
0146:   // Maps a logical shape to a physical shape by prepending the batch
0147:   // sizes to the logical shape.
0148:   VmapDimVector getPhysicalShape(IntArrayRef logical_shape) const;
0149:   SymDimVector getPhysicalShape(c10::SymIntArrayRef logical_shape) const;
0150: 
0151:   int64_t numBatchDims() const;
0152: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `getPhysicalToLogicalMap`, `getPhysicalShape`, `numBatchDims`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`getPhysicalToLogicalMap`, `getPhysicalShape`, `numBatchDims`。

### Lines 153-164 / 第 153-164 行

```cpp
0153:  private:
0154:   int64_t numLogicalDims() const;
0155: 
0156:   std::bitset<kVmapNumLevels> levels_;
0157:   Tensor tensor_;
0158: };
0159: 
0160: // Convenience struct used for mapping a physical tensor (a non-BatchedTensor)
0161: // to a logical one (BatchedTensor). It holds some levels that are used to do the
0162: // mapping and assumes that the batch dimensions in the physical tensor all
0163: // occur at the front of the tensor.
0164: struct TORCH_API VmapPhysicalToLogicalMap {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `VmapPhysicalToLogicalMap`, `numLogicalDims`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`VmapPhysicalToLogicalMap`, `numLogicalDims`。

### Lines 165-175 / 第 165-175 行

```cpp
0165:   VmapPhysicalToLogicalMap(std::bitset<kVmapNumLevels> levels): levels_(levels) {}
0166: 
0167:   // Maps a physical tensor to a new logical tensor (BatchedTensor).
0168:   // Assumes that all of the "batch dimensions" are at the front
0169:   // of the physical tensor. For example, given:
0170:   // - x = rank-4 Tensor with size 2, 3, 5, 7
0171:   // - levels = (2, 4)
0172:   // Returns:
0173:   // - BatchedTensor(x, bdims=[(dim=0,lvl=2), (dim=1, lvl=4)])
0174:   Tensor apply(const Tensor& physical_tensor) const;
0175: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `VmapPhysicalToLogicalMap`, `apply`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`VmapPhysicalToLogicalMap`, `apply`。

### Lines 176-185 / 第 176-185 行

```cpp
0176:   // Given a vector of physical tensors,
0177:   // 1. maps each tensor to a new logical tensor. Assumes that all of the
0178:   //    "batch dimensions" are at the front of the physical tensors.
0179:   // 2. stores the new logical tensors back into the passed-in vector. This is
0180:   //    to avoid additional dynamic allocations.
0181:   void applyInplace(std::vector<Tensor>& physical_tensors) const;
0182: 
0183:   std::bitset<kVmapNumLevels> levels_;
0184: };
0185: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `applyInplace`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`applyInplace`。

### Lines 186-187 / 第 186-187 行

```cpp
0186: 
0187: } // namespace at::functorch
```

- **EN:** Opens or closes namespace scopes so the remaining declarations stay inside the intended ATen/backend module.
- **CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 ATen/后端模块之内。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Vendor library descriptor management** — 厂商库描述符管理
- **Memory allocation strategy** — 内存分配策略
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: VmapPhysicalView, MultiBatchVmapTransform, BroadcastingVmapTransform, VmapPhysicalToLogicalMap, VmapPhysicalViewVec, VmapDimVector, VmapSymDimVector, logicalToPhysical** — 核心符号：VmapPhysicalView、MultiBatchVmapTransform、BroadcastingVmapTransform、VmapPhysicalToLogicalMap、VmapPhysicalViewVec、VmapDimVector、VmapSymDimVector、logicalToPhysical

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/functorch/Macros.h`, `ATen/functorch/BatchedTensorImpl.h`
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at::functorch`
- **Representative symbols / 代表性符号**: `VmapPhysicalView`, `MultiBatchVmapTransform`, `BroadcastingVmapTransform`, `VmapPhysicalToLogicalMap`, `VmapPhysicalViewVec`, `VmapDimVector`, `VmapSymDimVector`, `logicalToPhysical`, `tensor`, `getPhysicalDims`, `getPhysicalDim`, `getPhysicalToLogicalMap`, `...`
