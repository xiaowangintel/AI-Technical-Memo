# sgl_model_gateway.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/sgl_model_gateway.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang Model Gateway is a high-performance model-routing gateway for large-scale LLM deployments. / 该文档围绕 SGLang 模型 Gateway 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang Model Gateway is a high-performance model-routing gateway for large-scale LLM deployments.
**CN:** 本节围绕 Overview 展开，概述了 OpenAI-compatible, LLM, MCP, gateway 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Table of Contents
**EN:** Architecture - Control Plane - Data Plane - Storage and Privacy 3. Deployment Modes - Co-launch Router and Workers - Separate Launch (HTTP) - gRPC Launch - Prefill-Decode Disaggregation - OpenAI Backend Proxy - Multi-Model Inference Gateway 6.
**CN:** 本节围绕 Table of Contents 展开，概述了 Endpoints, Launch, Security, Kubernetes 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overview
**EN:** **Unified control plane** for registering, monitoring, and orchestrating regular, prefill, and decode workers across heterogeneous model fleets.
**CN:** 本节围绕 Overview 展开，概述了 MCP, OpenAI-compatible, gRPC, across 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Control Plane
**EN:** **Worker Manager** discovers capabilities (/server_info, /get_model_info), tracks load, and registers/removes workers in the shared registry. **Job Queue** serializes add/remove requests and exposes status (/workers/ ) so clients can track onboarding progress.
**CN:** 本节围绕 Control Plane 展开，概述了 load, workers, HuggingFace, Job 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Data Plane
**EN:** **HTTP routers** (regular & PD) implement /generate, /v1/chat/completions, /v1/completions, /v1/responses, /v1/embeddings, /v1/rerank, /v1/classify, /v1/tokenize, /v1/detokenize, and associated admin endpoints.
**CN:** 本节围绕 Data Plane 展开，概述了 OpenAI, SRT, Rust, gRPC 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Storage and Privacy
**EN:** Conversation and response history is stored at the router tier (memory, none, Oracle ATP, or PostgreSQL). The same history can power multiple models or MCP loops without sending data to upstream vendors.
**CN:** 本节围绕 Storage and Privacy 展开，概述了 MCP, APIs, same, history 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Docker
**EN:** Pre-built Docker images are available on Docker Hub with multi-architecture support (x86_64 and ARM64): ``bash docker pull lmsysorg/sgl-model-gateway:latest ``
**CN:** 本节围绕 Docker 展开，概述了 ARM64, Docker, Hub, pull 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prerequisites
**EN:** **Rust and Cargo** ``bash curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh source "$HOME/.cargo/env" rustc --version cargo --version ` - **Python** with pip` and virtualenv tooling available.
**CN:** 本节围绕 Prerequisites 展开，概述了 Rust, Cargo, version, --version 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Rust Binary
**EN:** ``bash cd sgl-model-gateway cargo build --release ``
**CN:** 本节围绕 Rust Binary 展开，概述了 cargo, build, release, --release 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Python Package
**EN:** ``bash pip install maturin # Fast development mode cd sgl-model-gateway/bindings/python maturin develop # Production build maturin build --release --out dist --features vendored-openssl pip install --force-reinstall dist/*.whl ``
**CN:** 本节围绕 Python Package 展开，概述了 maturin, pip, Fast, build 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Regular HTTP Routing
**EN:** ``bash # Rust binary ./target/release/sgl-model-gateway \ --worker-urls http://worker1:8000 http://worker2:8000 \ --policy cache_aware # Python launcher python -m sglang_router.launch_router \ --worker-urls http://worker1:8000 http://worker2:8000 \ --policy cache_aware ``
**CN:** 本节围绕 Regular HTTP Routing 展开，概述了 Rust, policy, --policy, worker-urls 等要点，并说明相关配置、流程、示例或限制条件。

### Section: gRPC Routing
**EN:** ``bash python -m sglang_router.launch_router \ --worker-urls grpc://127.0.0.1:20000 \ --model-path meta-llama/Llama-3.1-8B-Instruct \ --reasoning-parser deepseek-r1 \ --tool-call-parser json \ --host 0.0.0.0 --port 8080 ``
**CN:** 本节围绕 gRPC Routing 展开，概述了 host, port, --host, --port 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Co-launch Router and Workers
**EN:** This content focuses on Co-launch Router and Workers and highlights host, port, model, debug.
**CN:** 本节围绕 Co-launch Router and Workers 展开，概述了 host, port, model, debug 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Separate Launch (HTTP)
**EN:** This content focuses on Separate Launch (HTTP) and highlights port, --port, Run, model.
**CN:** 本节围绕 Separate Launch (HTTP) 展开，概述了 port, --port, Run, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: gRPC Launch
**EN:** Provide --tokenizer-path or --model-path` (HuggingFace ID or local directory) whenever connection mode resolves to gRPC.
**CN:** 本节围绕 gRPC Launch 展开，概述了 gRPC, --model-path, port, --port 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prefill-Decode Disaggregation
**EN:** PD mode merges prefill metadata with decode outputs and streams results back to the client.
**CN:** 本节围绕 prefill 阶段-decode 阶段 Disaggregation 展开，概述了 decode, prefill, Split, PD-aware 等要点，并说明相关配置、流程、示例或限制条件。

### Section: OpenAI Backend Proxy
**EN:** Proxy OpenAI-compatible endpoints while keeping history and MCP sessions local: ``bash python -m sglang_router.launch_router \ --backend openai \ --worker-urls https://api.openai.com \ --history-backend memory ` OpenAI backend mode expects exactly one --worker-urls` entry per router instance.
**CN:** 本节围绕 OpenAI Backend Proxy 展开，概述了 MCP, OpenAI, backend, worker-urls 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-Model Inference Gateway
**EN:** Enable IGW mode to route multiple models through a single router: ``bash ./target/release/sgl-model-gateway \ --enable-igw \ --policy cache_aware \ --max-concurrent-requests 512 # Register workers dynamically curl -X POST http://localhost:30000/workers \ -H "Content-Type: application/json" \ -d ' }' ``
**CN:** 本节围绕 Multi-模型 Inference Gateway 展开，概述了 POST, Register, Content-Type, IGW 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Inference Endpoints
**EN:** This section provides a comparison table for Inference Endpoints, covering columns such as Method, Path, Description and examples such as POST, POST, POST, POST.
**CN:** 本节围绕 Inference Endpoints 展开，概述了 POST, OpenAI-compatible, API, Path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tokenization Endpoints
**EN:** This section provides a comparison table for Tokenization Endpoints, covering columns such as Method, Path, Description and examples such as POST, POST, POST, GET.
**CN:** 本节围绕 令牌ization Endpoints 展开，概述了 POST, Hello, status, Tokenize 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Parser Endpoints
**EN:** This section provides a comparison table for Parser Endpoints, covering columns such as Method, Path, Description and examples such as POST, POST.
**CN:** 本节围绕 Parser Endpoints 展开，概述了 POST, Let, text, step 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Classification API
**EN:** This section provides a comparison table for Classification API, covering columns such as Field, Description and examples such as label, probs, num_classes.
**CN:** 本节围绕 Classification API 展开，概述了 label, model, id2label, classification 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Conversation and Response APIs
**EN:** This section provides a comparison table for Conversation and Response APIs, covering columns such as Method, Path, Description and examples such as POST, GET, POST, DELETE.
**CN:** 本节围绕 Conversation and Response APIs 展开，概述了 POST, DELETE, conversation, v1/conversations/ 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Worker Management APIs
**EN:** This section provides a comparison table for Worker Management APIs, covering columns such as Method, Path, Description and examples such as POST, GET, GET, PUT.
**CN:** 本节围绕 Worker Management APIs 展开，概述了 Queue, POST, worker, workers 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Admin and Health Endpoints
**EN:** This section provides a comparison table for Admin and Health Endpoints, covering columns such as Method, Path, Description and examples such as GET, GET, GET, GET.
**CN:** 本节围绕 Admin and Health Endpoints 展开，概述了 POST, Health, List, WASM 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Load Balancing Policies
**EN:** This section provides a comparison table for Load Balancing Policies, covering columns such as Policy, Description, Usage and examples such as random, round_robin, power_of_two, cache_aware.
**CN:** 本节围绕 Load Balancing Policies 展开，概述了 policy, --policy, random, bucket 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Cache-Aware Policy Tuning
**EN:** This section provides a comparison table for Cache-Aware Policy Tuning, covering columns such as Parameter, Default, Description and examples such as --cache-threshold, --balance-abs-threshold, --balance-rel-threshold, --eviction-interval-secs.
**CN:** 本节围绕 缓存-Aware Policy Tuning 展开，概述了 --max-tree-size, --cache-threshold, --balance-abs-threshold, --balance-rel-threshold 等要点，并说明相关配置、流程、示例或限制条件。

### Section: HTTP Client
**EN:** This section provides a comparison table for HTTP Client, covering columns such as Parameter, Default, Description and examples such as --pool-idle-timeout-secs, --connect-timeout-secs, --pool-max-idle-per-host, --tcp-keepalive-secs.
**CN:** 本节围绕 HTTP Client 展开，概述了 upstream, connections, seconds, SMG_TCP_KEEPALIVE_SECS 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Retries
**EN:** This section provides a comparison table for Retries, covering columns such as Parameter, Default, Description and examples such as --retry-max-retries, --retry-initial-backoff-ms, --retry-max-backoff-ms, --retry-backoff-multiplier.
**CN:** 本节围绕 Retries 展开，概述了 Maximum, backoff, --retry-max-retries, --retry-jitter-factor 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Circuit Breaker
**EN:** This section provides a comparison table for Circuit Breaker, covering columns such as Parameter, Default, Description and examples such as --cb-failure-threshold, --cb-success-threshold, --cb-timeout-duration-secs, --cb-window-duration-secs.
**CN:** 本节围绕 Circuit Breaker 展开，概述了 circuit, requests, --cb-failure-threshold, --cb-success-threshold 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Rate Limiting and Queuing
**EN:** ``bash python -m sglang_router.launch_router \ --worker-urls http://worker1:8000 http://worker2:8001 \ --max-concurrent-requests 256 \ --rate-limit-tokens-per-second 512 \ --queue-size 128 \ --queue-timeout-secs 30 ` Requests beyond the concurrency limit wait in a FIFO queue.
**CN:** 本节围绕 Rate Limiting and Queuing 展开，概述了 queue, Requests, FIFO, Returns 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Health Checks
**EN:** ``bash --health-check-interval-secs 30 \ --health-check-timeout-secs 10 \ --health-success-threshold 2 \ --health-failure-threshold 3 \ --health-check-endpoint /health ``
**CN:** 本节围绕 Health Checks 展开，概述了 health, health-check-endpoint, --health-check-endpoint, health-success-threshold 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Reasoning Parser Integration
**EN:** The gateway includes built-in reasoning parsers for models that use Chain-of-Thought (CoT) reasoning with explicit thinking blocks.
**CN:** 本节围绕 Reasoning Parser Integration 展开，概述了 CoT, reasoning, Chain-of-Thought, models 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Parsers
**EN:** This section provides a comparison table for Supported Parsers, covering columns such as Parser ID, Model Family, Think Tokens and examples such as deepseek-r1, qwen3, qwen3-thinking, kimi.
**CN:** 本节围绕 Supported Parsers 展开，概述了 think, <think>...</think>, Qwen-3, kimi 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage
**EN:** ``bash python -m sglang_router.launch_router \ --worker-urls grpc://127.0.0.1:20000 \ --model-path deepseek-ai/DeepSeek-R1 \ --reasoning-parser deepseek-r1 `` The gRPC router automatically: 1. Detects reasoning blocks in streaming output 2.
**CN:** 本节围绕 Usage 展开，概述了 streaming, Detects, Applies, Handles 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tool Call Parsing
**EN:** The gateway supports parsing function/tool calls from LLM outputs in multiple formats.
**CN:** 本节围绕 Tool Call Parsing 展开，概述了 LLM, calls, gateway, parsing 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Formats
**EN:** This section provides a comparison table for Supported Formats, covering columns such as Parser, Format, Description and examples such as json, python, xml.
**CN:** 本节围绕 Supported Formats 展开，概述了 xml, tool, calls, Parser 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage
**EN:** ``bash python -m sglang_router.launch_router \ --worker-urls grpc://127.0.0.1:20000 \ --model-path meta-llama/Llama-3.1-8B-Instruct \ --tool-call-parser json ``
**CN:** 本节围绕 Usage 展开，概述了 model-path, worker-urls, --model-path, --worker-urls 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tokenizer Sources
**EN:** The gateway supports multiple tokenizer backends: - **HuggingFace**: Load from HuggingFace Hub by model ID - **Local**: Load from local tokenizer.json or directory - **Tiktoken**: Auto-detect OpenAI GPT models (gpt-4, davinci, etc.)
**CN:** 本节围绕 令牌izer Sources 展开，概述了 Load, HuggingFace, Local, Tiktoken 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration
**EN:** ``bash # HuggingFace model --model-path meta-llama/Llama-3.1-8B-Instruct # Local tokenizer --tokenizer-path /path/to/tokenizer.json # With chat template override --chat-template /path/to/template.jinja ``
**CN:** 本节围绕 配置 展开，概述了 Local, HuggingFace, chat, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tokenizer Caching
**EN:** This section provides a comparison table for Tokenizer Caching, covering columns such as Cache, Type, Description and examples such as L0, L1.
**CN:** 本节围绕 令牌izer Caching 展开，概述了 Prefix, Type, Cache, Exact 等要点，并说明相关配置、流程、示例或限制条件。

### Section: MCP Integration
**EN:** The gateway provides native Model Context Protocol (MCP) client integration for tool execution.
**CN:** 本节围绕 MCP Integration 展开，概述了 MCP, tool, Model, native 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Transports
**EN:** This section provides a comparison table for Supported Transports, covering columns such as Transport, Description and examples such as STDIO, SSE, Streamable.
**CN:** 本节围绕 Supported Transports 展开，概述了 SSE, STDIO, Local, Transport 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration
**EN:** ``bash python -m sglang_router.launch_router \ --mcp-config-path /path/to/mcp-config.yaml \ --worker-urls http://worker1:8000 ``
**CN:** 本节围绕 配置 展开，概述了 worker-urls, --worker-urls, mcp-config-path, --mcp-config-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: MCP Configuration File
**EN:** This content focuses on MCP Configuration File and highlights name, protocol, required, url.
**CN:** 本节围绕 MCP 配置 File 展开，概述了 name, protocol, required, url 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Service Discovery (Kubernetes)
**EN:** Enable automatic worker discovery via Kubernetes pod selectors: ``bash python -m sglang_router.launch_router \ --service-discovery \ --selector app=sglang-worker role=inference \ --service-discovery-namespace production \ --service-discovery-port 8000 ``
**CN:** 本节围绕 Service Discovery (Kubernetes) 展开，概述了 Enable, Kubernetes, via, pod 等要点，并说明相关配置、流程、示例或限制条件。

### Section: PD Mode Discovery
**EN:** ``bash --pd-disaggregation \ --prefill-selector app=sglang component=prefill \ --decode-selector app=sglang component=decode \ --service-discovery ` Prefill pods can expose bootstrap ports via the sglang.ai/bootstrap-port annotation.
**CN:** 本节围绕 PD Mode Discovery 展开，概述了 app, RBAC, pods, Prefill 等要点，并说明相关配置、流程、示例或限制条件。

### Section: History and Data Connectors
**EN:** This section provides a comparison table for History and Data Connectors, covering columns such as Backend, Description, Usage and examples such as memory, none, oracle, postgres.
**CN:** 本节围绕 History and Data Connectors 展开，概述了 history-backend, --history-backend, redis, memory 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Oracle Configuration
**EN:** This content focuses on Oracle Configuration and highlights export, TNS, ATP_DSN, ATP_USER.
**CN:** 本节围绕 Oracle 配置 展开，概述了 export, TNS, ATP_DSN, ATP_USER 等要点，并说明相关配置、流程、示例或限制条件。

### Section: PostgreSQL Configuration
**EN:** ``bash export POSTGRES_DB_URL="postgres://user:password@host:5432/dbname" python -m sglang_router.launch_router \ --backend openai \ --worker-urls https://api.openai.com \ --history-backend postgres ``
**CN:** 本节围绕 PostgreSQL 配置 展开，概述了 POSTGRES_DB_URL, export, openai, backend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Redis Configuration
**EN:** This content focuses on Redis Configuration and highlights export, REDIS_URL, REDIS_POOL_MAX, REDIS_RETENTION_DAYS.
**CN:** 本节围绕 Redis 配置 展开，概述了 export, REDIS_URL, REDIS_POOL_MAX, REDIS_RETENTION_DAYS 等要点，并说明相关配置、流程、示例或限制条件。

### Section: WASM Middleware
**EN:** The gateway supports WebAssembly (WASM) middleware modules for custom request/response processing. This enables organization-specific logic for authentication, rate limiting, billing, logging, and more—without modifying or recompiling the gateway.
**CN:** 本节围绕 WASM Middleware 展开，概述了 WASM, gateway, WebAssembly, rate 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overview
**EN:** This section provides a comparison table for Overview, covering columns such as Attach Point, When Executed, Use Cases and examples such as OnRequest, OnResponse, Action, Continue.
**CN:** 本节围绕 Overview 展开，概述了 Modify, Reject, status, Continue 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Examples
**EN:** This section provides a comparison table for Examples, covering columns such as Example, Description and examples such as auth/, rate_limit/, logging/.
**CN:** 本节围绕 Examples 展开，概述了 API, auth/, Request, logging/ 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Building Modules
**EN:** ``bash # Prerequisites rustup target add wasm32-wasip2 cargo install wasm-tools # Build cargo build --target wasm32-wasip2 --release # Convert to component format wasm-tools component new \ target/wasm32-wasip2/release/my_middleware.wasm \ -o my_middleware.component.wasm ``
**CN:** 本节围绕 Building Modules 展开，概述了 Build, cargo, target, Convert 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Deploying Modules
**EN:** This content focuses on Deploying Modules and highlights Middleware, curl, POST, List.
**CN:** 本节围绕 Deploying Modules 展开，概述了 Middleware, curl, POST, List 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Runtime Configuration
**EN:** This section provides a comparison table for Runtime Configuration, covering columns such as Parameter, Default, Description and examples such as max_memory_pages, max_execution_time_ms, max_stack_size, module_cache_size.
**CN:** 本节围绕 Runtime 配置 展开，概述了 Note, Rate, Stack, Redis 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Language Bindings
**EN:** SGLang Model Gateway provides official language bindings for Python and Go, enabling integration with different technology stacks and organizational requirements.
**CN:** 本节围绕 Language Bindings 展开，概述了 Model, stacks, Gateway, provides 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Python Bindings
**EN:** The Python bindings provide a PyO3-based wrapper around the Rust gateway library. This is a straightforward binding that calls the gateway server startup from Python.
**CN:** 本节围绕 Python Bindings 展开，概述了 RouterArgs, Router.from_args, See, Key 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Go Bindings
**EN:** This section provides a comparison table for Go Bindings, covering columns such as Example, Description and examples such as simple/, streaming/, oai_server/.
**CN:** 本节围绕 Go Bindings 展开，概述了 FFI, Rust, Integration, Full 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Comparison
**EN:** This section provides a comparison table for Comparison, covering columns such as Feature, Python, Go and examples such as **Primary Use**, **CLI Support**, **K8s Discovery**, **PD Mode**.
**CN:** 本节围绕 Comparison 展开，概述了 client, library, CLI, N/A 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Router API Key
**EN:** ``bash python -m sglang_router.launch_router \ --api-key "your-router-api-key" \ --worker-urls http://worker1:8000 ` Clients must supply Authorization: Bearer <key>` for protected endpoints.
**CN:** 本节围绕 Router API Key 展开，概述了 Bearer, Clients, Authorization, key 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Worker API Keys
**EN:** ``bash # Add worker with explicit key curl -H "Authorization: Bearer router-key" \ -X POST http://localhost:8080/workers \ -H "Content-Type: application/json" \ -d ' ' ``
**CN:** 本节围绕 Worker API Keys 展开，概述了 Add, POST, Bearer, Content-Type 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Security Configurations
**EN:** **No Authentication** (default): Use only in trusted environments 2. **Router-only Authentication**: Clients authenticate to router 3.
**CN:** 本节围绕 Security 配置s 展开，概述了 Authentication, Both, Router, Clients 等要点，并说明相关配置、流程、示例或限制条件。

### Section: TLS (HTTPS) for Gateway Server
**EN:** This section provides a comparison table for TLS (HTTPS) for Gateway Server, covering columns such as Parameter, Description and examples such as --tls-cert-path, --tls-key-path.
**CN:** 本节围绕 TLS (HTTPS) for Gateway 服务端 展开，概述了 TLS, PEM, Path, gateway 等要点，并说明相关配置、流程、示例或限制条件。

### Section: mTLS for Worker Communication
**EN:** This section provides a comparison table for mTLS for Worker Communication, covering columns such as Parameter, Description and examples such as --client-cert-path, --client-key-path, --ca-cert-path.
**CN:** 本节围绕 mTLS for Worker Communication 展开，概述了 TLS, PEM, Path, --ca-cert-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Full TLS Configuration Example
**EN:** This content focuses on Full TLS Configuration Example and highlights Worker, API, Key, mTLS.
**CN:** 本节围绕 Full TLS 配置 Example 展开，概述了 Worker, API, Key, mTLS 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prometheus Metrics
**EN:** This section provides a comparison table for Prometheus Metrics, covering columns such as Layer, Prefix, Metrics and examples such as HTTP, Router, Inference, Worker.
**CN:** 本节围绕 Prometheus 指标 展开，概述了 Histogram, connections_active, Time, requests_total 等要点，并说明相关配置、流程、示例或限制条件。

### Section: OpenTelemetry Tracing
**EN:** This content focuses on OpenTelemetry Tracing and highlights OTLP, Trace, span, Batch.
**CN:** 本节围绕 OpenTelemetry Tracing 展开，概述了 OTLP, Trace, span, Batch 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Logging
**EN:** ``bash python -m sglang_router.launch_router \ --worker-urls http://worker1:8000 \ --log-level debug \ --log-dir ./router_logs ` Structured tracing with optional file sink. Log levels: debug, info, warn, error`.
**CN:** 本节围绕 Logging 展开，概述了 Log, debug, Structured, sink 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Request ID Propagation
**EN:** ``bash --request-id-headers x-request-id x-trace-id x-correlation-id ` Responses include x-request-id` header for correlation.
**CN:** 本节围绕 Request ID Propagation 展开，概述了 Responses, x-request-id, header, include 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Production Recommendations
**EN:** This section provides guidance for deploying SGLang Model Gateway in production environments.
**CN:** 本节围绕 Production Recommendations 展开，概述了 Model, Gateway, provides, guidance 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Security Best Practices
**EN:** This content focuses on Security Best Practices and highlights TLS, Enable, --api-key, API.
**CN:** 本节围绕 Security Best Practices 展开，概述了 TLS, Enable, --api-key, API 等要点，并说明相关配置、流程、示例或限制条件。

### Section: High Availability
**EN:** This section provides a comparison table for High Availability, covering columns such as Component, Shared Across Replicas, Impact and examples such as Worker Registry, Radix Cache Tree, Circuit Breaker State, Rate Limiting.
**CN:** 本节围绕 High Availability 展开，概述了 Worker, Pod, Gateway, Replica 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Performance
**EN:** This section provides a comparison table for Performance, covering columns such as Parameter, Recommendation, Reason and examples such as --policy, --max-concurrent-requests, --queue-size, --request-timeout-secs.
**CN:** 本节围绕 性能 展开，概述了 gRPC, Prevent, mode, --policy 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Kubernetes Deployment
**EN:** This content focuses on Kubernetes Deployment and highlights name, sglang-worker, app, component.
**CN:** 本节围绕 Kubernetes 部署 展开，概述了 name, sglang-worker, app, component 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Monitoring with PromQL
**EN:** Configure Prometheus to scrape the gateway metrics endpoint (default: :29000/metrics). **Essential Dashboards:** **1.
**CN:** 本节围绕 Monitoring with PromQL 展开，概述了 rate, sum, histogram_quantile, P99 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Core Settings
**EN:** This section provides a comparison table for Core Settings, covering columns such as Parameter, Type, Default and examples such as --host, --port, --worker-urls, --policy.
**CN:** 本节围绕 Core Settings 展开，概述了 int, Router, str, Type 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prefill/Decode
**EN:** This section provides a comparison table for Prefill/Decode, covering columns such as Parameter, Type, Default and examples such as --pd-disaggregation, --prefill, --decode, --prefill-policy.
**CN:** 本节围绕 prefill 阶段/decode 阶段 展开，概述了 Override, str, Type, list 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Kubernetes Discovery
**EN:** This section provides a comparison table for Kubernetes Discovery, covering columns such as Parameter, Type, Description and examples such as --service-discovery, --selector, --prefill-selector / --decode-selector, --service-discovery-namespace.
**CN:** 本节围绕 Kubernetes Discovery 展开，概述了 str, Type, list, Label 等要点，并说明相关配置、流程、示例或限制条件。

### Section: TLS Configuration
**EN:** This section provides a comparison table for TLS Configuration, covering columns such as Parameter, Type, Description and examples such as --tls-cert-path, --tls-key-path, --client-cert-path, --client-key-path.
**CN:** 本节围绕 TLS 配置 展开，概述了 PEM, str, Server, Client 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Workers Never Ready
**EN:** Increase --worker-startup-timeout-secs or ensure health probes respond before router startup.
**CN:** 本节围绕 Workers Never Ready 展开，概述了 Increase, --worker-startup-timeout-secs, ensure, health 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Load Imbalance / Hot Workers
**EN:** Inspect smg_router_requests_total by worker and tune cache-aware thresholds (--balance-*, --cache-threshold).
**CN:** 本节围绕 Load Imbalance / Hot Workers 展开，概述了 Inspect, --cache-threshold, smg_router_requests_total, tune 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Circuit Breaker Flapping
**EN:** Increase --cb-failure-threshold or extend the timeout/window durations. Consider temporarily disabling retries.
**CN:** 本节围绕 Circuit Breaker Flapping 展开，概述了 Increase, Consider, --cb-failure-threshold, extend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Queue Overflow (429)
**EN:** Increase --queue-size or reduce client concurrency. Ensure --max-concurrent-requests matches downstream capacity.
**CN:** 本节围绕 Queue Overflow (429) 展开，概述了 Ensure, Increase, --queue-size, --max-concurrent-requests 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Memory Growth
**EN:** Reduce --max-tree-size or lower --eviction-interval-secs for more aggressive cache pruning.
**CN:** 本节围绕 Memory Growth 展开，概述了 Reduce, --max-tree-size, --eviction-interval-secs, lower 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Debugging
**EN:** ``bash python -m sglang_router.launch_router \ --worker-urls http://worker1:8000 \ --log-level debug \ --log-dir ./router_logs ``
**CN:** 本节围绕 Debugging 展开，概述了 debug, log-dir, --log-dir, log-level 等要点，并说明相关配置、流程、示例或限制条件。

### Section: gRPC Connection Issues
**EN:** Ensure workers are started with --grpc-mode and verify --model-path or --tokenizer-path is provided to the router.
**CN:** 本节围绕 gRPC Connection Issues 展开，概述了 Ensure, --grpc-mode, --model-path, --tokenizer-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tokenizer Loading Failures
**EN:** Check HuggingFace Hub credentials (HF_TOKEN environment variable) for private models. Verify local paths are accessible.
**CN:** 本节围绕 令牌izer Loading Failures 展开，概述了 HF_TOKEN, Verify, Hub, CLI 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** POST / **CN:** POST
- **EN:** Description / **CN:** Description
- **EN:** Worker / **CN:** Worker
- **EN:** workers / **CN:** workers
- **EN:** sglang_router.launch_router / **CN:** sglang_router.launch_router
- **EN:** gRPC / **CN:** gRPC
- **EN:** rate / **CN:** rate
- **EN:** gateway / **CN:** gateway

## Dependencies / 依赖关系
- `/path/to/mcp-config.yaml`
