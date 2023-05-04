# Table of Contents

* [aqueduct\_llm.dolly\_v2\_3b](#aqueduct_llm.dolly_v2_3b)
  * [generate](#aqueduct_llm.dolly_v2_3b.generate)

<a id="aqueduct_llm.dolly_v2_3b"></a>

# aqueduct\_llm.dolly\_v2\_3b

<a id="aqueduct_llm.dolly_v2_3b.generate"></a>

#### generate

```python
def generate(messages: Union[str, List[str]],
             do_sample: bool = default_do_sample,
             max_new_tokens: int = default_max_new_tokens,
             top_p: float = default_top_p,
             top_k: int = default_top_k) -> Union[str, List[str]]
```

Invoke the Dolly V2 3B model to generate responses.

The model requires a GPU and at least 8GB of RAM.

**Arguments**:

- `messages` _Union[str, List[str]]_ - The message(s) to generate responses for.
- `do_sample` _bool, optional_ - Whether or not to use sampling. Defaults to True. Default: True
- `max_new_tokens` _int, optional_ - Max new tokens after the prompt to generate. Default: 256
- `top_p` _float, optional_ - If set to float < 1, only the smallest set of most probable tokens with
  probabilities that add up to top_p or higher are kept for generation. Default: 0.92.
- `top_k` _int, optional_ - The number of highest probability vocabulary tokens to keep for top-k-filtering. Default: 0.
  

**Examples**:

  ```python
  >>> from aqueduct_llm import dolly_v2_3b
  >>> dolly_v2_3b.generate("What's the best LLM?", do_sample=True, max_new_tokens=256, top_p=0.92, top_k=0)
  "Dolly V2 3B is the best LLM!"
  ```

