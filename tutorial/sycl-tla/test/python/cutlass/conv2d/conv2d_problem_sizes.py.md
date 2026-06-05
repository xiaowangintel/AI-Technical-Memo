# conv2d_problem_sizes.py — Code Analysis / 代码分析
## Source / 来源
- **Path / 路径:** `test/python/cutlass/conv2d/conv2d_problem_sizes.py`
- **EN:** Defines a reusable catalogue of Conv2D problem sizes spanning default cases, ResNet-50 shapes, grouped convolutions, and a few disabled rigorous cases.
- **CN:** 定义一套可复用的 Conv2D 问题规模目录，覆盖默认样例、ResNet-50 形状、分组卷积以及少量被禁用的严格测试样例。

## Line-by-Line Analysis / 逐行分析
### Lines 45-59
```python
class TestbedConv2dProblemSizes:
    def __init__(self, minimum_channel_size: int):
        conv2d_default_sizes = self.initialize_conv2d_default_sizes(minimum_channel_size)
        conv2d_rigorous_sizes = self.initialize_conv2d_rigorous_sizes(minimum_channel_size)
        conv2d_resnet50_sizes = self.initialize_conv2d_resnet50_sizes(1)
        conv2d_resnet50_sizes_perf = self.initialize_conv2d_resnet50_sizes(34)
        grouped_sizes = self.initialize_conv2d_grouped_sizes()

        # Filter all problems
        self.all = []
        for size_list in [conv2d_default_sizes, conv2d_rigorous_sizes, conv2d_resnet50_sizes, conv2d_resnet50_sizes_perf, grouped_sizes]:
            for size in size_list:
                if (size.C // size.groups) % minimum_channel_size == 0:
                    self.all.append(size)

```
**EN:** `TestbedConv2dProblemSizes` builds several named problem lists during construction, then flattens them into `self.all`. The final filter keeps only shapes whose per-group channel count is divisible by `minimum_channel_size`, which is important because many generated kernels assume channel alignment constraints.

**CN:** `TestbedConv2dProblemSizes` 在构造时先生成多个具名问题列表，再把它们汇总到 `self.all`。最后的过滤条件要求“每组通道数”能够被 `minimum_channel_size` 整除，这一点很关键，因为很多生成出来的 kernel 假设通道满足对齐约束。

### Lines 61-129
```python
    def initialize_conv2d_default_sizes(self, minimum_channel_size):
        # Small input size x stride (1,1)
        # C < CTA::K and non-multiples of CTA::K. Typical CTA::K = {32, 64}

        conv2d_default_sizes = []
        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 1, 1, minimum_channel_size,
          8, 1, 1, minimum_channel_size,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 1, 8, minimum_channel_size,
          8, 1, 3, minimum_channel_size,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 7, 8, minimum_channel_size,
          8, 3, 3, minimum_channel_size,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 7, 9, minimum_channel_size,
          8, 4, 4, minimum_channel_size,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          2, 7, 9, minimum_channel_size,
          8, 5, 5, minimum_channel_size,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          3, 7, 9, minimum_channel_size,
          8, 6, 5, minimum_channel_size,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          3, 7, 9, minimum_channel_size,
          8, 6, 6, minimum_channel_size,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          3, 7, 9, minimum_channel_size,
          8, 7, 7, minimum_channel_size,
          1, 1,
          1, 1,
          1, 1,
        ))

```
**EN:** `initialize_conv2d_default_sizes()` begins with tiny stride-1 cases. These `Conv2DProblemSize(...)` calls vary batch size, input height/width, filter size, and channel counts while keeping padding/stride/dilation simple. The goal is to exercise kernels where the input channel count can be smaller than a typical CTA `K` tile and where filter sizes range from 1x1 to 7x7.

**CN:** `initialize_conv2d_default_sizes()` 先从小尺寸、步长为 1 的案例开始。这些 `Conv2DProblemSize(...)` 调用会变化 batch、大/小输入尺寸、滤波器尺寸和通道数，同时让 padding/stride/dilation 保持简单。其目标是覆盖输入通道数小于典型 CTA `K` 分块、以及 1x1 到 7x7 多种滤波器尺寸的情况。

### Lines 134-197
```python
        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 11, 7, minimum_channel_size,
          8, 1, 1, minimum_channel_size,
          0, 0,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 11, 7, minimum_channel_size,
          8, 3, 3, minimum_channel_size,
          1, 1,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 13, 11, minimum_channel_size,
          8, 1, 1, minimum_channel_size,
          1, 1,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 17, 19, minimum_channel_size,
          16, 2, 2, minimum_channel_size,
          1, 1,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 23, 5, minimum_channel_size,
          16, 3, 3, minimum_channel_size,
          1, 1,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 13, 17, 8,
          24, 3, 3, 8,
          0, 0,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 23, 21, 8,
          24, 3, 3, 8,
          1, 1,
          3, 3,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 20, 24, 8,
          40, 3, 3, 8,
          3, 3,
          3, 3,
          1, 1,
        ))

```
**EN:** The next block adds stride-2 and stride-3 style defaults. Several cases use zero padding, others use unit padding, and a few explicitly set channel counts to `8` instead of `minimum_channel_size`. This broadens coverage to downsampling and non-square output patterns that are common in real networks.

**CN:** 接下来的代码块增加了步长为 2 和 3 的默认案例。有些样例使用零填充，有些使用单位填充，还有少数条目显式把通道数写成 `8`，而不是 `minimum_channel_size`。这样可以扩展到下采样以及真实网络中常见的非方形输出模式。

### Lines 201-239
```python
        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 15, 19, 160,
          224, 1, 1, 160,
          0, 0,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 19, 37, 160,
          224, 3, 3, 160,
          1, 1,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 16, 16, 160,
          224, 2, 3, 160,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 23, 21, 128,
          224, 3, 3, 128,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 29, 37, 160,
          224, 5, 5, 160,
          2, 2,
          1, 1,
          1, 1,
        ))
```
**EN:** Here the file shifts to medium-sized tensors with larger channel counts such as 160 and 224. The appended problems vary filter geometry (1x1, 2x3, 3x3, 5x5), padding, and stride, creating shapes that look more like practical CNN layers than synthetic micro-cases.

**CN:** 这一段转向中等规模张量，通道数提升到 160、224 等更大的值。新增问题在滤波器形状（1x1、2x3、3x3、5x5）、padding 和 stride 上做变化，使这些形状更接近真实 CNN 层，而不仅仅是合成的小样例。

### Lines 244-293
```python
        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 15, 19, 32 + minimum_channel_size,
          96, 3, 3, 32 + minimum_channel_size,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 16, 24, 64 + minimum_channel_size,
          96, 3, 3, 64 + minimum_channel_size,
          1, 1,
          1, 1,
          1, 1,
        ))

        ##########################################
        # Medium input size, filter size (1x1, 3,x3, 5x5, 7x7), stride (2, 2)
        ##########################################
        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 13, 16, 288,
          160, 5, 5, 288,
          2, 2,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 55, 51, 256,
          512, 1, 1, 256,
          0, 0,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 71, 80, 32,
          64, 5, 5, 32,
          2, 2,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 224, 224, 8,
          64, 7, 7, 8,
          3, 3,
          2, 2,
          1, 1,
        ))
```
**EN:** These cases explicitly target scenarios where `C > CTA::K` or where the input is much larger, including a 224x224 image-like case. They stress kernels that must handle larger reduction dimensions and more bandwidth-heavy convolutions, especially when stride is greater than one.

**CN:** 这些样例显式针对 `C > CTA::K` 或输入尺寸更大的场景，其中还包含一个类似图像输入的 224x224 案例。它们主要压测需要处理更大归约维度、以及在 stride 大于 1 时更偏带宽密集型的卷积 kernel。

### Lines 298-343
```python
        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 27, 23, 256,
          512, 3, 3, 256,
          0, 0,
          3, 3,
          1, 1,
        ))

        ##########################################
        # Medium input size padding > stride, asymmetric filter, padding and striding
        ##########################################
        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 27, 31, 256,
          512, 3, 3, 256,
          5, 7,
          3, 4,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 27, 35, 256,
          512, 7, 5, 256,
          11, 7,
          3, 5,
          1, 1,
        ))

        ##########################################
        # Medium input size *mixed* stride (1, 2) and (2, 1),
        # filter (3, 3), default padding
        ##########################################
        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 27, 27, 256,
          512, 3, 3, 256,
          1, 1,
          1, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          1, 27, 27, 256,
          512, 3, 3, 256,
          1, 1,
          2, 1,
          1, 1,
        ))
```
**EN:** This section focuses on awkward geometry: non-default padding, asymmetric filters, and mixed strides `(1, 2)` / `(2, 1)`. Such shapes are valuable because they shake out iterator and output-shape calculations that often pass on symmetric, stride-1 cases.

**CN:** 这一段聚焦于“别扭”的几何形状：非默认 padding、非对称滤波器，以及 `(1, 2)` / `(2, 1)` 这样的混合步长。这类问题很有价值，因为很多迭代器与输出尺寸计算错误只会在这些非常规案例中暴露出来，而在对称且 stride=1 的情形下往往不会出错。

### Lines 348-379
```python
        conv2d_default_sizes.append(Conv2DProblemSize(
          3, 28, 28, 256,
          256, 2, 2, 256,
          0, 0,
          2, 2,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
           1, 32, 32, 16,
           32, 3, 3, 16,
           1, 1,
           6, 2,
           1, 1,
         ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          32, 24, 32, 32,
          32, 1, 2, 32,
          0, 0,
          1, 1,
          1, 1,
        ))

        conv2d_default_sizes.append(Conv2DProblemSize(
          4, 2, 3, 256,
          328, 3, 5, 256,
          1, 1,
          1, 1,
          1, 1,
        ))
        return conv2d_default_sizes
```
**EN:** The final default-size additions introduce extra edge cases: higher batch counts, uneven shapes, and unusual filter combinations. Together, they keep the default list from being dominated by one narrow family of tensor layouts.

**CN:** 默认集合的最后一批补充案例引入了更多边界场景：更高的 batch、非均匀尺寸以及不常见的滤波器组合。这样可避免默认测试集被单一类型的张量布局主导。

### Lines 382-395
```python
    def initialize_conv2d_rigorous_sizes(self, minimum_channel_size):
        sizes = []
        if False:
            sizes.append(Conv2DProblemSize.from_sizes(
              (1, 124, 224, 2 * minimum_channel_size),
              (24, 7, 7, 2 * minimum_channel_size),
            ))

            sizes.append(Conv2DProblemSize.from_sizes(
              (1, 233, 35, minimum_channel_size),
              (24, 7, 5, minimum_channel_size),
            ))
        return sizes

```
**EN:** `initialize_conv2d_rigorous_sizes()` currently returns an empty list because the candidate shapes are wrapped in `if False:`. The function remains as a placeholder for future large or expensive cases without affecting today's unit-test runtime.

**CN:** `initialize_conv2d_rigorous_sizes()` 目前会返回空列表，因为候选样例被包在 `if False:` 中。这个函数相当于为未来更大、更昂贵的测试保留了扩展点，但不会影响当前单元测试的运行时间。

### Lines 397-551
```python
    def initialize_conv2d_resnet50_sizes(self, batch_size):
        conv2d_problem_vector = []
        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 56, 56, 64,
          256, 1, 1, 64,
          0, 0,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 56, 56, 64,
          64, 1, 1, 64,
          0, 0,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 56, 56, 64,
          64, 3, 3, 64,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 56, 56, 256,
          64, 1, 1, 256,
          0, 0,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 56, 56, 256,
          512, 1, 1, 256,
          0, 0,
          2, 2,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 56, 56, 256,
          128, 1, 1, 256,
          0, 0,
          2, 2,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 28, 28, 128,
          128, 3, 3, 128,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 28, 28, 128,
          512, 1, 1, 128,
          0, 0,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 28, 28, 512,
          128, 1, 1, 512,
          0, 0,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 28, 28, 512,
          1024, 1, 1, 512,
          0, 0,
          2, 2,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 28, 28, 512,
          256, 1, 1, 512,
          0, 0,
          2, 2,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 14, 14, 256,
          256, 3, 3, 256,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 14, 14, 256,
          1024, 1, 1, 256,
          0, 0,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 14, 14, 1024,
          256, 1, 1, 1024,
          0, 0,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 14, 14, 1024,
          2048, 1, 1, 1024,
          0, 0,
          2, 2,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 14, 14, 1024,
          512, 1, 1, 1024,
          0, 0,
          2, 2,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 7, 7, 512,
          512, 3, 3, 512,
          1, 1,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 7, 7, 512,
          2048, 1, 1, 512,
          0, 0,
          1, 1,
          1, 1,
        ))

        conv2d_problem_vector.append(Conv2DProblemSize(
          batch_size, 7, 7, 2048,
          512, 1, 1, 2048,
          0, 0,
          1, 1,
          1, 1,
        ))

        return conv2d_problem_vector
```
**EN:** `initialize_conv2d_resnet50_sizes()` hard-codes a long sequence of shapes that correspond to ResNet-50 layers. The `batch_size` parameter lets the caller reuse the same layer catalogue for correctness-sized runs (`1`) and performance-oriented runs (`34`), which is why the constructor creates two separate ResNet-derived lists.

**CN:** `initialize_conv2d_resnet50_sizes()` 用一长串硬编码形状描述了 ResNet-50 中的卷积层。`batch_size` 参数让调用方可以复用同一份层级目录：既能生成用于正确性验证的小 batch（`1`），也能生成偏性能测试的较大 batch（`34`），因此构造函数会建立两份不同的 ResNet 派生列表。

### Lines 553-661
```python
    def initialize_conv2d_grouped_sizes(self):
        threadblock_n = 128
        threadblock_k = 32

        sizes = []
        ##########################################
        # One group calculated by one or multiple CTAs: k_per_group % CTA::N = 0
        # One CTA calculates a single group
        ##########################################
        for cta_per_group_k in range(1, 4):
            for groups in range(2, 5):
                conv_k = cta_per_group_k * threadblock_n * groups
                sizes.append(Conv2DProblemSize(
                  1, 8, 8, threadblock_k * 2 * groups,
                  conv_k, 3, 3, threadblock_k * 2,
                  1, 1,
                  1, 1,
                  1, 1,
                  ConvMode.CrossCorrelation,
                  1,
                  groups
                ))

        # Partial gemm_k: k_per_group == CTA::N && channels_per_group < CTA::K
        sizes.append(Conv2DProblemSize(
          1, 8, 8, threadblock_k,
          threadblock_n * 2, 3, 3, threadblock_k // 2,
          1, 1,
          1, 1,
          1, 1,
          ConvMode.CrossCorrelation,
          1,
          2
        ))

        sizes.append(Conv2DProblemSize(
          1, 56, 56, 696,
          768, 3, 3, 232,
          1, 1,
          2, 2,
          1, 1,
          ConvMode.CrossCorrelation,
          1,
          3
        ))
        sizes.append(Conv2DProblemSize(
          1, 14, 14, 1392,
          1536, 3, 3, 232,
          1, 1,
          1, 1,
          1, 1,
          ConvMode.CrossCorrelation,
          1,
          3
        ))

        ##########################################
        # One CTA calculate multiple groups: CTA::N % k_per_group = 0
        ##########################################

        # 2 groups per CTA
        sizes.append(Conv2DProblemSize(
          1, 8, 8, threadblock_k * 4,
          threadblock_n, 3, 3, threadblock_k * 2,
          1, 1,
          1, 1,
          1, 1,
          ConvMode.CrossCorrelation,
          1,
          2
        ))

        # 2 groups per CTA and partial gemm_k
        sizes.append(Conv2DProblemSize(
          1, 8, 8, threadblock_k,
          threadblock_n, 3, 3, threadblock_k // 2,
          1, 1,
          1, 1,
          1, 1,
          ConvMode.CrossCorrelation,
          1,
          2
        ))

        # 4 groups per CTA
        sizes.append(Conv2DProblemSize(
          1, 8, 8, threadblock_k * 8,
          threadblock_n // 2, 3, 3, threadblock_k * 2,
          1, 1,
          1, 1,
          1, 1,
          ConvMode.CrossCorrelation,
          1,
          4
        ))

        # 4 groups per CTA and partial gemm_k
        sizes.append(Conv2DProblemSize(
          1, 8, 8, threadblock_k * 2,
          threadblock_n // 2, 3, 3, threadblock_k // 2,
          1, 1,
          1, 1,
          1, 1,
          ConvMode.CrossCorrelation,
          1,
          4
        ))

        return sizes
```
**EN:** `initialize_conv2d_grouped_sizes()` switches from ordinary convolution to grouped convolution. It uses `threadblock_n` and `threadblock_k` as construction guides, then emits cases for one CTA per group, multiple CTAs per group, and multiple groups per CTA. The final `Conv2DProblemSize` arguments include `ConvMode.CrossCorrelation`, split-k style metadata, and an explicit `groups` count, so these shapes stress grouped-kernel scheduling as well as geometry.

**CN:** `initialize_conv2d_grouped_sizes()` 从普通卷积切换到分组卷积。它先把 `threadblock_n` 和 `threadblock_k` 作为构造参考，再生成“每组一个 CTA”“每组多个 CTA”“一个 CTA 计算多个组”等多种案例。这里 `Conv2DProblemSize` 的末尾参数包含 `ConvMode.CrossCorrelation`、相关元数据以及显式 `groups` 值，因此这些问题不仅覆盖几何尺寸，也会压测分组 kernel 的调度方式。

## Key Concepts / 关键概念
- **EN:** Curated test corpus: the file is not an algorithm implementation but a carefully assembled input-space catalogue.
  **CN:** 精心策划的测试语料：该文件不是算法实现，而是一份经过设计的输入空间目录。
- **EN:** Alignment-aware filtering: `self.all` only keeps shapes compatible with the requested minimum channel granularity.
  **CN:** 对齐感知过滤：`self.all` 只保留满足最小通道粒度要求的形状。
- **EN:** Multiple workload families: synthetic defaults, ResNet-50 layers, grouped convolutions, and disabled rigorous cases coexist in one source.
  **CN:** 多类工作负载并存：合成默认样例、ResNet-50 层、分组卷积以及被禁用的严格样例集中在同一文件中。

## Dependencies / 依赖关系
- **EN:** `cutlass_library.ConvMode` supplies the convolution mode enum used by many `Conv2DProblemSize` constructors.
  **CN:** `cutlass_library.ConvMode` 提供了许多 `Conv2DProblemSize` 构造调用所需的卷积模式枚举。
- **EN:** `cutlass_cppgen.shape.Conv2DProblemSize` is the central data structure; every list in this file is a list of these objects.
  **CN:** `cutlass_cppgen.shape.Conv2DProblemSize` 是核心数据结构；本文件中的所有问题列表都由它组成。
- **EN:** The generated problem lists are consumed downstream by `conv2d_test_utils.py` and `conv2d_sm80.py` to instantiate real CUTLASS Conv2D tests.
  **CN:** 生成的问题列表会被下游的 `conv2d_test_utils.py` 与 `conv2d_sm80.py` 消费，用于实例化真正的 CUTLASS Conv2D 测试。
