# json_logprobs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/json_logprobs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example demonstrates json logprobs within the SGLang repository. / 该示例展示了在 SGLang 仓库中如何完成 json logprobs。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Import dependencies and runtime symbols
````python
# NOTE: Currently this can only be run through HTTP requests.
from concurrent.futures import ThreadPoolExecutor

from json_decode import character_regex

from sglang.utils import http_request
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 8-12: Set top-level configuration
````python
character_names = ["Hermione Granger", "Ron Weasley", "Harry Potter"]

base_url = "http://localhost:30000"

prompt = "is a character in Harry Potter. Please fill in the following information about this character.\n"
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 15-35: Handle api request
````python
def openai_api_request(name):
    data = {
        "model": "",
        "prompt": name + prompt,
        "temperature": 0,
        "max_tokens": 128,
        "regex": character_regex,
        "logprobs": 3,
    }
    res = http_request(base_url + "/v1/completions", json=data).json()

    # with open(f"json_logprobs_{name.replace(' ', '_')}_tmp.json", "w") as fout:
    #     fout.write(json.dumps(res, indent=4))

    logprobs = res["choices"][0]["logprobs"]
    usage = res["usage"]
    assert len(logprobs["token_logprobs"]) == len(logprobs["tokens"])
    assert len(logprobs["token_logprobs"]) == len(logprobs["top_logprobs"])
    assert len(logprobs["token_logprobs"]) == usage["completion_tokens"] - 1

    return res
````
**EN:** This function encapsulates the “Handle api request” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle api request”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 38-67: Handle api request
````python
def srt_api_request(name):
    data = {
        "text": name + prompt,
        "sampling_params": {
            "temperature": 0,
            "max_new_tokens": 128,
            "regex": character_regex,
        },
        "return_logprob": True,
        "logprob_start_len": 0,
        "top_logprobs_num": 3,
        "return_text_in_logprobs": True,
    }

    res = http_request(base_url + "/generate", json=data).json()

    # with open(f"json_logprobs_{name.replace(' ', '_')}_tmp.json", "w") as fout:
    #     fout.write(json.dumps(res, indent=4))

    meta_info = res["meta_info"]
    assert len(meta_info["input_token_logprobs"]) == len(
        meta_info["input_top_logprobs"]
    )
    assert len(meta_info["output_token_logprobs"]) == len(
        meta_info["output_top_logprobs"]
    )
    assert len(meta_info["input_token_logprobs"]) == meta_info["prompt_tokens"]
    assert len(meta_info["output_token_logprobs"]) == meta_info["completion_tokens"] - 1

    return res
````
**EN:** This function encapsulates the “Handle api request” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle api request”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 70-93: Handle print
````python
def pretty_print(res):
    meta_info = res["meta_info"]

    print("\n\n", "=" * 30, "Prefill", "=" * 30)
    for i in range(len(meta_info["input_token_logprobs"])):
        print(f"{str(meta_info['input_token_logprobs'][i][2].encode()): <20}", end="")
        top_ks = (
            [str(t[2].encode()) for t in meta_info["input_top_logprobs"][i]]
            if meta_info["input_top_logprobs"][i]
            else []
        )
        for top_k in top_ks:
            print(f"{top_k: <15}", end="")
        print()

    print("\n\n", "=" * 30, "Decode", "=" * 30)
    for i in range(len(meta_info["output_token_logprobs"])):
        print(f"{str(meta_info['output_token_logprobs'][i][2].encode()): <20}", end="")
        top_ks = [str(t[2].encode()) for t in meta_info["output_top_logprobs"][i]]
        for top_k in top_ks:
            print(f"{top_k: <15}", end="")
        print()

    print(res["text"])
````
**EN:** This function encapsulates the “Handle print” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle print”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 96-103: Program entry point
````python
if __name__ == "__main__":
    with ThreadPoolExecutor() as executor:
        ress = executor.map(srt_api_request, character_names)

    for res in ress:
        pretty_print(res)

    openai_api_request("Hermione Granger")
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。
- **Structured generation / 结构化生成**: Schemas or constrained decoding keep outputs machine-readable. / 模式约束或受限解码让输出保持机器可读。

## Dependencies / 依赖关系
- **Standard library / 标准库**: concurrent.futures.ThreadPoolExecutor
- **Third-party / 第三方**: json_decode.character_regex
- **Project-specific / 项目相关**: sglang.utils.http_request
