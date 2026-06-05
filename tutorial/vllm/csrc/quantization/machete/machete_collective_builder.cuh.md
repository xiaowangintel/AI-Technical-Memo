# machete_collective_builder.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/machete/machete_collective_builder.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Registers Machete as a custom CUTLASS collective builder specialization for SM90 tensor-op GEMM schedules. / [CN] 将 Machete 注册为 SM90 Tensor Core GEMM 调度下的自定义 CUTLASS collective builder 特化。

## Line-by-Line Analysis / 逐行分析
### Marker tag for dispatch / 用于分发的标记类型
```cpp
namespace cutlass::gemm::collective {
using namespace cute;

struct MacheteKernelTag {};
```
**EN:** `MacheteKernelTag` is a zero-cost type tag. CUTLASS builder logic uses it to choose Machete-specific machinery without adding runtime overhead.
**CN:** `MacheteKernelTag` 是一个零开销的类型标签。CUTLASS 的 builder 逻辑据此切换到 Machete 专用实现，而不会引入运行时开销。

### Partial specialization of `VLLMCollectiveBuilder` / `VLLMCollectiveBuilder` 的偏特化
```cpp
template <class ElementPairA_, class GmemLayoutA_, int AlignmentA,
          class ElementPairB_, class GmemLayoutB_, int AlignmentB,
          class ElementAccumulator, class TileShape_MNK, class ClusterShape_MNK,
          class StageCountType, class KernelScheduleType>
struct VLLMCollectiveBuilder<
    MacheteKernelTag, arch::Sm90, arch::OpClassTensorOp, ...,
    cute::enable_if_t<(
        cute::is_same_v<KernelScheduleType, KernelTmaWarpSpecialized> ||
        cute::is_same_v<KernelScheduleType, KernelTmaWarpSpecializedPingpong> ||
        cute::is_same_v<KernelScheduleType,
                        KernelTmaWarpSpecializedCooperative>)>> {
```
**EN:** The specialization only activates for SM90 Tensor Core kernels and three warp-specialized TMA schedules. That restriction prevents Machete's collective from being instantiated on unsupported architectures or scheduling styles.
**CN:** 这个特化只会在 SM90 Tensor Core 内核以及三种 warp-specialized TMA schedule 下生效。这样的限制能避免在不支持的架构或调度方式上错误实例化 Machete collective。

### Final type selection / 最终类型选择
```cpp
using CollectiveOp = machete::MacheteCollectiveMma<
    ElementPairA_, GmemLayoutA_, AlignmentA, ElementPairB_, GmemLayoutB_,
    AlignmentB, ElementAccumulator, TileShape_MNK, ClusterShape_MNK,
    StageCountType, KernelScheduleType>;
```
**EN:** Once the constraints match, the builder's output is simply aliased to `MacheteCollectiveMma`, which becomes the collective MMA mainloop used by the final GEMM kernel.
**CN:** 一旦约束匹配成功，builder 的输出就直接别名到 `MacheteCollectiveMma`，它会成为最终 GEMM 内核使用的 collective MMA 主循环。

## Key Concepts / 关键概念
- **Compile-time registration**: CUTLASS extensibility is implemented through type specialization rather than runtime plugin lookup. / **编译期注册**：CUTLASS 的可扩展性通过类型特化实现，而不是运行时插件查找。
- **Architecture gating**: the file hard-codes SM90 + TensorOp + specific schedules. / **架构门控**：该文件把可用范围明确限定为 SM90 + TensorOp + 特定 schedule。
- **Collective aliasing**: the builder does not implement logic itself; it redirects to `MacheteCollectiveMma`. / **Collective 别名转发**：builder 本身不实现核心逻辑，而是把选择结果转发到 `MacheteCollectiveMma`。

## Dependencies / 依赖关系
- `cutlass_extensions/vllm_collective_builder.cuh` provides the base builder interface being specialized. / `cutlass_extensions/vllm_collective_builder.cuh` 提供了这里被特化的基础 builder 接口。
- `machete_mainloop.cuh` defines `MacheteCollectiveMma`, the actual collective operator. / `machete_mainloop.cuh` 定义了真正的 collective 算子 `MacheteCollectiveMma`。
- Cute/CUTLASS types such as `KernelTmaWarpSpecialized` and `arch::Sm90` supply the compile-time constraints. / `KernelTmaWarpSpecialized`、`arch::Sm90` 等 Cute/CUTLASS 类型提供编译期约束。
