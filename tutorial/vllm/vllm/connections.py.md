# connections.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `vllm/connections.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable sync/async HTTP helpers with retry, timeout backoff, and download cleanup for media fetching. / 为媒体拉取提供可复用的同步/异步 HTTP 工具，包含重试、超时退避与下载清理逻辑。

## Line-by-Line Analysis / 逐行分析
### Retryable error classification (lines 24-73)
```python
# Multiplier applied to timeout and sleep on each retry attempt.
# Attempt N uses: base_timeout * (_RETRY_BACKOFF_FACTOR ** N) for the
# per-attempt timeout and sleeps _RETRY_BACKOFF_FACTOR ** N seconds.
_RETRY_BACKOFF_FACTOR = 4


def _is_retryable(exc: Exception) -> bool:
    """Return True for transient errors that are worth retrying.

    Retryable:
      - Timeouts (aiohttp, requests, stdlib)
      - Connection-level failures (refused, reset, DNS)
      - Server errors (5xx) -- includes S3 503 SlowDown
    Not retryable:
      - Client errors (4xx) -- bad URL, auth, not-found
      - Programming errors (ValueError, TypeError, ...)
    """
    # Timeouts
    if isinstance(
        exc,
        (
            TimeoutError,
            asyncio.TimeoutError,
            requests.exceptions.Timeout,
            aiohttp.ServerTimeoutError,
        ),
    ):
        return True
    # Connection-level failures
    if isinstance(
        exc,
        (
            ConnectionError,
            aiohttp.ClientConnectionError,
            requests.exceptions.ConnectionError,
        ),
    ):
        return True
    # aiohttp server-side disconnects
    if isinstance(exc, aiohttp.ServerDisconnectedError):
        return True
    # requests 5xx -- raise_for_status() throws HTTPError
    if (
        isinstance(exc, requests.exceptions.HTTPError)
        and exc.response is not None
        and exc.response.status_code >= 500
    ):
        return True
    # aiohttp 5xx -- raise_for_status() throws ClientResponseError
    return isinstance(exc, aiohttp.ClientResponseError) and exc.status >= 500
```
**EN:** `_is_retryable()` encodes the policy boundary: timeouts, connection failures, disconnects, and HTTP 5xx are treated as transient; HTTP 4xx and programming errors are not. That distinction prevents wasteful retries for malformed URLs or authorization failures while still handling bursty object-store and network issues.
**CN:** `_is_retryable()` 定义了重试策略边界：超时、连接失败、断连和 HTTP 5xx 被视为瞬时错误；HTTP 4xx 与编程错误则不会重试。这样可以避免对错误 URL 或鉴权失败做无意义重试，同时保留对对象存储抖动和网络波动的容错能力。

### Sync retry wrapper and logging (lines 76-151)
```python
def _log_retry(
    args: tuple,
    kwargs: dict,
    attempt: int,
    max_retries: int,
    attempt_timeout: float | None,
    exc: Exception,
    backoff: float,
    base_timeout: float | None,
) -> None:
    # args[0] is `self` (bound method), args[1] is the URL
    url = args[1] if len(args) > 1 else kwargs.get("url")
    timeout_info = (
        f"timeout={attempt_timeout:.3f}s" if base_timeout is not None else "no timeout"
    )
    next_timeout = (
        f" with timeout={base_timeout * (_RETRY_BACKOFF_FACTOR ** (attempt + 1)):.3f}s"
        if base_timeout is not None
        else ""
    )
    logger.warning(
        "HTTP fetch failed for %s (attempt %d/%d, %s): %s -- retrying in %.3fs%s",
        url,
        attempt + 1,
        max_retries,
        timeout_info,
        exc,
        backoff,
        next_timeout,
    )


def _sync_retry(
    fn: Callable[_P, _T],
) -> Callable[_P, _T]:
    """Add retry logic with exponential backoff to a sync method.

    The decorated method must accept ``timeout`` as a keyword argument.
    The decorator replaces it with a per-attempt timeout that grows by
    ``_RETRY_BACKOFF_FACTOR`` on each retry so transient slowness on busy
    hosts is absorbed.
    """

    @functools.wraps(fn)
    def wrapper(*args: Any, **kwargs: Any) -> _T:
        base_timeout: float | None = kwargs.get("timeout")
        max_retries = max(envs.VLLM_MEDIA_FETCH_MAX_RETRIES, 1)

        for attempt in range(max_retries):
            attempt_timeout = (
                base_timeout * (_RETRY_BACKOFF_FACTOR**attempt)
                if base_timeout is not None
                else None
            )
            kwargs["timeout"] = attempt_timeout
            try:
                return fn(*args, **kwargs)
            except Exception as e:
                if not _is_retryable(e) or attempt + 1 >= max_retries:
                    raise
                backoff = _RETRY_BACKOFF_FACTOR**attempt
                _log_retry(
                    args,
                    kwargs,
                    attempt,
                    max_retries,
                    attempt_timeout,
                    e,
                    backoff,
                    base_timeout,
                )
                time.sleep(backoff)

        raise AssertionError("unreachable")

    return wrapper  # type: ignore[return-value]
```
**EN:** `_log_retry()` formats a high-signal warning containing URL, attempt index, current timeout, sleep backoff, and next timeout. `_sync_retry()` then multiplies per-attempt timeout and sleep duration by `_RETRY_BACKOFF_FACTOR`, using `envs.VLLM_MEDIA_FETCH_MAX_RETRIES` as the runtime policy knob.
**CN:** `_log_retry()` 会输出高信息密度的告警，包括 URL、重试次数、当前超时、回退等待时间和下次超时值。随后 `_sync_retry()` 按 `_RETRY_BACKOFF_FACTOR` 递增每次尝试的超时与休眠时长，并通过 `envs.VLLM_MEDIA_FETCH_MAX_RETRIES` 读取运行时策略。

### Async retry wrapper (lines 154-197)
```python
def _async_retry(
    fn: Callable[_P, Coroutine[Any, Any, _T]],
) -> Callable[_P, Coroutine[Any, Any, _T]]:
    """Add retry logic with exponential backoff to an async method.

    The decorated method must accept ``timeout`` as a keyword argument.
    The decorator replaces it with a per-attempt timeout that grows by
    ``_RETRY_BACKOFF_FACTOR`` on each retry so transient slowness on busy
    hosts is absorbed.
    """

    @functools.wraps(fn)
    async def wrapper(*args: Any, **kwargs: Any) -> _T:
        base_timeout: float | None = kwargs.get("timeout")
        max_retries = max(envs.VLLM_MEDIA_FETCH_MAX_RETRIES, 1)

        for attempt in range(max_retries):
            attempt_timeout = (
                base_timeout * (_RETRY_BACKOFF_FACTOR**attempt)
                if base_timeout is not None
                else None
            )
            kwargs["timeout"] = attempt_timeout
            try:
                return await fn(*args, **kwargs)
            except Exception as e:
                if not _is_retryable(e) or attempt + 1 >= max_retries:
                    raise
                backoff = _RETRY_BACKOFF_FACTOR**attempt
                _log_retry(
                    args,
                    kwargs,
                    attempt,
                    max_retries,
                    attempt_timeout,
                    e,
                    backoff,
                    base_timeout,
                )
                await asyncio.sleep(backoff)

        raise AssertionError("unreachable")

    return wrapper  # type: ignore[return-value]
```
**EN:** The async decorator mirrors the sync logic but swaps `time.sleep()` for `await asyncio.sleep()`. vLLM keeps the sync and async implementations separate instead of trying to unify them through a generic wrapper, which keeps type signatures and coroutine behavior explicit.
**CN:** 异步装饰器与同步逻辑基本一致，但把 `time.sleep()` 换成了 `await asyncio.sleep()`。vLLM 没有强行用一个通用包装器统一两套实现，而是保留显式的同步/异步版本，从而让类型签名和协程语义更清晰。

### HTTPConnection client lifecycle (lines 200-276)
```python
class HTTPConnection:
    """Helper class to send HTTP requests."""

    def __init__(self, *, reuse_client: bool = True) -> None:
        super().__init__()

        self.reuse_client = reuse_client

        self._sync_client: requests.Session | None = None
        self._async_client: aiohttp.ClientSession | None = None

    def get_sync_client(self) -> requests.Session:
        if self._sync_client is None or not self.reuse_client:
            self._sync_client = requests.Session()

        return self._sync_client

    # NOTE: We intentionally use an async function even though it is not
    # required, so that the client is only accessible inside async event loop
    async def get_async_client(self) -> aiohttp.ClientSession:
        if self._async_client is None or not self.reuse_client:
            self._async_client = aiohttp.ClientSession(trust_env=True)

        return self._async_client

    def _validate_http_url(self, url: str):
        parsed_url = parse_url(url)

        if parsed_url.scheme not in ("http", "https"):
            raise ValueError(
                "Invalid HTTP URL: A valid HTTP URL must have scheme 'http' or 'https'."
            )

    def _headers(self, **extras: str) -> MutableMapping[str, str]:
        return {"User-Agent": f"vLLM/{VLLM_VERSION}", **extras}

    def get_response(
        self,
        url: str,
        *,
        stream: bool = False,
        timeout: float | None = None,
        extra_headers: Mapping[str, str] | None = None,
        allow_redirects: bool = True,
    ):
        self._validate_http_url(url)

        client = self.get_sync_client()
        extra_headers = extra_headers or {}

        return client.get(
            url,
            headers=self._headers(**extra_headers),
            stream=stream,
            timeout=timeout,
            allow_redirects=allow_redirects,
        )

    async def get_async_response(
        self,
        url: str,
        *,
        timeout: float | None = None,
        extra_headers: Mapping[str, str] | None = None,
        allow_redirects: bool = True,
    ):
        self._validate_http_url(url)

        client = await self.get_async_client()
        extra_headers = extra_headers or {}

        return client.get(
            url,
            headers=self._headers(**extra_headers),
            timeout=timeout,
            allow_redirects=allow_redirects,
        )
```
**EN:** `HTTPConnection` caches both a `requests.Session` and an `aiohttp.ClientSession` when `reuse_client=True`, which reduces connection setup overhead for repeated media fetches. `_validate_http_url()` rejects non-HTTP(S) schemes early, and `_headers()` injects a versioned `User-Agent` so remote services can identify vLLM traffic.
**CN:** `HTTPConnection` 在 `reuse_client=True` 时会缓存 `requests.Session` 和 `aiohttp.ClientSession`，以降低重复媒体拉取时的连接建立开销。`_validate_http_url()` 会提前拒绝非 HTTP(S) 协议，`_headers()` 则注入带版本号的 `User-Agent`，便于远端服务识别 vLLM 请求。

### High-level fetch and download helpers (lines 278-394)
```python
    @_sync_retry
    def get_bytes(
        self, url: str, *, timeout: float | None = None, allow_redirects: bool = True
    ) -> bytes:
        with self.get_response(
            url, timeout=timeout, allow_redirects=allow_redirects
        ) as r:
            r.raise_for_status()

            return r.content

    @_async_retry
    async def async_get_bytes(
        self,
        url: str,
        *,
        timeout: float | None = None,
        allow_redirects: bool = True,
    ) -> bytes:
        async with await self.get_async_response(
            url, timeout=timeout, allow_redirects=allow_redirects
        ) as r:
            r.raise_for_status()

            return await r.read()

    def get_text(self, url: str, *, timeout: float | None = None) -> str:
        with self.get_response(url, timeout=timeout) as r:
            r.raise_for_status()

            return r.text

    async def async_get_text(
        self,
        url: str,
        *,
        timeout: float | None = None,
    ) -> str:
        async with await self.get_async_response(url, timeout=timeout) as r:
            r.raise_for_status()

            return await r.text()

    def get_json(self, url: str, *, timeout: float | None = None) -> str:
        with self.get_response(url, timeout=timeout) as r:
            r.raise_for_status()

            return r.json()

    async def async_get_json(
        self,
        url: str,
        *,
        timeout: float | None = None,
    ) -> str:
        async with await self.get_async_response(url, timeout=timeout) as r:
            r.raise_for_status()

            return await r.json()

    @_sync_retry
    def download_file(
        self,
        url: str,
        save_path: Path,
        *,
        timeout: float | None = None,
        chunk_size: int = 128,
    ) -> Path:
        try:
            with self.get_response(url, timeout=timeout) as r:
                r.raise_for_status()

                with save_path.open("wb") as f:
                    for chunk in r.iter_content(chunk_size):
                        f.write(chunk)

            return save_path
        except Exception:
            # Clean up partial downloads before retrying or propagating
            if save_path.exists():
                save_path.unlink()
            raise

    @_async_retry
    async def async_download_file(
        self,
        url: str,
        save_path: Path,
        *,
        timeout: float | None = None,
        chunk_size: int = 128,
    ) -> Path:
        try:
            async with await self.get_async_response(
                url,
                timeout=timeout,
            ) as r:
                r.raise_for_status()

                with save_path.open("wb") as f:
                    async for chunk in r.content.iter_chunked(chunk_size):
                        f.write(chunk)

            return save_path
        except Exception:
            # Clean up partial downloads before retrying or propagating
            if save_path.exists():
                save_path.unlink()
            raise


global_http_connection = HTTPConnection()
"""
The global [`HTTPConnection`][vllm.connections.HTTPConnection] instance used
by vLLM.
"""
```
**EN:** The high-level APIs (`get_bytes`, `get_text`, `get_json`, `download_file`, and async variants) consistently call `raise_for_status()` before consuming content. The download paths are careful to delete partially written files on failure so the next retry never mistakes a truncated artifact for a valid cache hit. The module-level `global_http_connection` provides one shared default instance.
**CN:** 高层 API（`get_bytes`、`get_text`、`get_json`、`download_file` 及其异步版本）都会在读取内容前统一调用 `raise_for_status()`。下载逻辑还会在失败时删除部分写入的文件，避免下一次重试把截断文件误判为有效缓存。模块级的 `global_http_connection` 则提供了一个共享默认实例。

## Key Concepts / 关键概念
- EN: Retry policy is environment-driven, not hard-coded, so operators can tune resilience without code changes.
- CN: 重试策略由环境变量驱动，而不是写死在代码里，运维可以在不改代码的情况下调整容错性。
- EN: Sync and async APIs deliberately expose the same semantics: validated URL, shared headers, `raise_for_status()`, and exponential backoff.
- CN: 同步与异步 API 刻意保持相同语义：URL 校验、统一请求头、`raise_for_status()` 以及指数退避。
- EN: Download cleanup is part of correctness, not just housekeeping, because stale partial files can poison a cache.
- CN: 下载清理属于正确性的一部分，而不仅是卫生工作；残缺文件会污染缓存判断。

## Dependencies / 依赖关系
- EN: Depends on `requests` for synchronous IO and `aiohttp` for asynchronous IO.
- CN: 同步 IO 依赖 `requests`，异步 IO 依赖 `aiohttp`。
- EN: Reads retry count from `vllm.envs.VLLM_MEDIA_FETCH_MAX_RETRIES` and logs through `vllm.logger`.
- CN: 从 `vllm.envs.VLLM_MEDIA_FETCH_MAX_RETRIES` 读取重试次数，并通过 `vllm.logger` 记录日志。
- EN: Uses `urllib3.util.parse_url` and `vllm.version.__version__` for URL validation and User-Agent composition.
- CN: 使用 `urllib3.util.parse_url` 做 URL 校验，并通过 `vllm.version.__version__` 组装 User-Agent。
