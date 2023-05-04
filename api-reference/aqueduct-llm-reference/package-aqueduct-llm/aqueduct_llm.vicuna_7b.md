# Table of Contents

* [aqueduct\_llm.vicuna\_7b](#aqueduct_llm.vicuna_7b)
  * [generate](#aqueduct_llm.vicuna_7b.generate)

<a id="aqueduct_llm.vicuna_7b"></a>

# aqueduct\_llm.vicuna\_7b

<a id="aqueduct_llm.vicuna_7b.generate"></a>

#### generate

```python
@torch.inference_mode()
def generate(
        messages: Union[str, List[str]],
        max_gpu_memory: str = default_max_gpu_memory,
        temperature: float = default_temperature,
        max_new_tokens: int = default_max_new_tokens) -> Union[str, List[str]]
```

Invoke the Vicuna 7B model to generate responses.

The model requires a GPU and at least 32GB of RAM.

**Arguments**:

- `messages` _Union[str, List[str]]_ - The message(s) to generate responses for.
- `max_gpu_memory` _str, optional_ - The maximum amount of GPU memory to use. Default: "13GiB"
- `temperature` _float, optional_ - The temperature to use for sampling. Default: 0.7
- `max_new_tokens` _int, optional_ - The maximum number of tokens to generate. Default: 1024
  

**Examples**:

  >>> from aqueduct_llm import vicuna_7b
  >>> vicuna_7b.generate("What's the best LLM?", max_gpu_memory="13GiB", temperature=0.7, max_new_tokens=1024)
  "Vicuna 7B is the best LLM!"

