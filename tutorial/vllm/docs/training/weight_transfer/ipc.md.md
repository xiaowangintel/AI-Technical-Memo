# ipc.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/training/weight_transfer/ipc.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page covers the IPC backend, which uses CUDA IPC handles to let colocated trainer and inference workers share GPU-resident weights directly. The emphasis is efficiency on the same GPU and operational details for Ray or HTTP delivery.
- **CN:** 本文介绍 IPC 后端，它使用 CUDA IPC 句柄让共置的训练端与推理端直接共享位于 GPU 上的权重。重点在于同 GPU 场景下的高效率，以及通过 Ray 或 HTTP 传递句柄的操作细节。

## Key Concepts / 关键概念
- **EN:** IPC is best suited for colocated training and inference on the same GPU or same GPU set.
  **CN:** IPC 最适合同一 GPU 或同一组 GPU 上的训练与推理共置场景。
- **EN:** The trainer serializes CUDA IPC handles, and each inference rank rebuilds only the handle for its own GPU.
  **CN:** 训练端会序列化 CUDA IPC 句柄，而每个推理 rank 只重建属于自己 GPU 的句柄。
- **EN:** Packed transfer chunks weights into bounded buffers so large models do not require all weight memory at once.
  **CN:** 打包传输会把权重切成有界缓冲区，从而避免大模型一次性占满全部权重显存。
- **EN:** HTTP mode requires `VLLM_ALLOW_INSECURE_SERIALIZATION=1` because pickled IPC metadata is transmitted over JSON.
  **CN:** HTTP 模式需要设置 `VLLM_ALLOW_INSECURE_SERIALIZATION=1`，因为 IPC 元数据会以 pickle+JSON 的方式传输。
