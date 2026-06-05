# multiprocessing.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/multiprocessing.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document explains why Python multiprocessing is unusually tricky in vLLM: the project must work both as a CLI-controlled system and as a library embedded inside user applications.  
  **CN:** 本文解释了为什么 Python 多进程在 vLLM 中格外棘手：项目既要支持由 CLI 控制的系统场景，也要支持嵌入用户应用中的库场景。
- **EN:** It reviews the tradeoffs of `spawn`, `fork`, and `forkserver`, highlighting that `fork` is fast but unsafe with thread-using dependencies, while `spawn`/`forkserver` can re-execute unguarded application code and break library usage without a `__main__` guard.  
  **CN:** 文档回顾了 `spawn`、`fork` 与 `forkserver` 的取舍：`fork` 虽快，但对使用线程的依赖不安全；`spawn`/`forkserver` 则可能重新执行未加 `__main__` 保护的应用代码，从而破坏库调用场景。
- **EN:** The current and prior vLLM behavior is documented in detail: the environment variable `VLLM_WORKER_MULTIPROC_METHOD` defaults to `fork`, the `vllm` CLI prefers `spawn`, XPU execution forces `spawn`, and v1 moved toward a best-effort compatibility strategy rather than a single universal rule.  
  **CN:** 文档详细记录了 vLLM 当前与历史行为：环境变量 `VLLM_WORKER_MULTIPROC_METHOD` 默认是 `fork`，`vllm` CLI 倾向使用 `spawn`，XPU 执行强制 `spawn`，而 v1 则转向“尽力兼容”的策略而非单一规则。
- **EN:** The design discussion is pragmatic: it explicitly rejects difficult or disruptive alternatives, documents the warning/exception path when CUDA is already initialized, and suggests future worker-management redesigns as a longer-term fix.  
  **CN:** 文档的设计讨论非常务实：它明确否定了难以实现或破坏兼容性的替代方案，记录了 CUDA 已初始化时的告警/异常路径，并提出未来重构 worker 管理作为长期解决方向。

## Key Concepts / 关键概念
- **EN:** **Start methods** — `spawn`, `fork`, and `forkserver` each trade off startup speed, compatibility, and code re-execution risk.  
  **CN:** **启动方法** —— `spawn`、`fork` 和 `forkserver` 在启动速度、兼容性与代码重复执行风险之间各有取舍。
- **EN:** **`__main__` guard requirement** — library users need safe import boundaries when `spawn`-style methods create fresh Python interpreters.  
  **CN:** **`__main__` 保护要求** —— 当 `spawn` 类方法创建全新 Python 解释器时，库用户需要安全的导入边界。
- **EN:** **Dependency compatibility** — CUDA/PyTorch-related components often prefer or require `spawn`, making pure `fork` solutions unreliable.  
  **CN:** **依赖兼容性** —— 与 CUDA/PyTorch 相关的组件往往偏好或要求 `spawn`，因此纯 `fork` 方案并不可靠。
- **EN:** **Best-effort method selection** — vLLM chooses different defaults depending on context instead of forcing one mode everywhere.  
  **CN:** **尽力而为的方式选择** —— vLLM 会根据上下文选择不同默认值，而不是在所有场景下强制统一模式。
- **EN:** **Future worker manager ideas** — the document points toward a custom manager process or alternative libraries such as `loky`.  
  **CN:** **未来 worker 管理方案** —— 文档把方向指向自定义 manager 进程或 `loky` 等替代库。
