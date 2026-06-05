# optimization_levels.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/optimization_levels.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** The document defines four optimization levels (`-O0` to `-O3`) that trade startup latency against runtime performance, with `-O2` as the default production-oriented setting.  
  **CN:** 本文定义了四个优化等级（`-O0` 到 `-O3`），用于在启动时延与运行性能之间做权衡，其中 `-O2` 是面向生产环境的默认设置。
- **EN:** Rather than being opaque presets, the levels are presented as bundles of lower-level flags for cudagraph behavior, compilation mode, autotuning, and fusion passes; user-specified flags still override the preset defaults.  
  **CN:** 这些等级并不是黑盒预设，而是由 cudagraph 行为、编译模式、自动调优和融合开关等底层参数组合而成；若用户显式设置参数，则会覆盖等级默认值。
- **EN:** `-O0` disables compilation, autotuning, cudagraphs, and fusions for fastest startup, while `-O1` enables fast compilation plus basic fusions and piecewise cudagraphs for development-friendly acceleration.  
  **CN:** `-O0` 关闭编译、自动调优、cudagraph 与融合，以获得最快启动；`-O1` 则启用快速编译、基础融合与分段 cudagraph，在开发场景中提供更友好的加速。
- **EN:** `-O2` extends `-O1` with fuller cudagraph coverage and extra fusion passes, while `-O3` is currently identical to `-O2` but reserved for more aggressive or experimental optimizations later.  
  **CN:** `-O2` 在 `-O1` 基础上增加更完整的 cudagraph 覆盖和额外融合；`-O3` 当前与 `-O2` 相同，但被预留给未来更激进或实验性的优化。

## Key Concepts / 关键概念
- **EN:** **Optimization levels** — coarse presets that simplify choosing between startup speed and steady-state performance.  
  **CN:** **优化等级** —— 用于简化“启动速度 vs 稳态性能”选择的粗粒度预设。
- **EN:** **Cudagraph modes** — the levels map directly to `NONE`, `PIECEWISE`, and `FULL_AND_PIECEWISE` graph-capture behaviors.  
  **CN:** **Cudagraph 模式** —— 各等级直接映射到 `NONE`、`PIECEWISE` 和 `FULL_AND_PIECEWISE` 等图捕获行为。
- **EN:** **Compilation control** — `-cc.mode` and related custom-op behavior are central levers behind the presets.  
  **CN:** **编译控制** —— `-cc.mode` 及相关 custom-op 行为是这些预设背后的核心杠杆。
- **EN:** **Fusion passes** — norm/quant, activation, padding, allreduce-RMS, and RoPE/KV-cache fusions are enabled progressively by level and hardware stack.  
  **CN:** **融合 pass** —— norm/quant、activation、padding、allreduce-RMS 和 RoPE/KV-cache 融合会按等级与硬件栈逐步开启。
- **EN:** **Operational troubleshooting** — the doc links level choice directly to common issues such as long startup, compilation failures, and under-optimized production runs.  
  **CN:** **运维排障** —— 文档把等级选择直接对应到启动过慢、编译失败和生产环境优化不足等常见问题。
