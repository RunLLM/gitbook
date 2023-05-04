# Table of Contents

* [aqueduct\_llm.llama\_7b](#aqueduct_llm.llama_7b)
  * [generate](#aqueduct_llm.llama_7b.generate)

<a id="aqueduct_llm.llama_7b"></a>

# aqueduct\_llm.llama\_7b

<a id="aqueduct_llm.llama_7b.generate"></a>

#### generate

```python
def generate(messages: Union[str, List[str]],
             max_length: int = default_max_length) -> Union[str, List[str]]
```

Invoke the LLaMA 7B model to generate responses.

The model requires a GPU and at least 16GB of RAM.

**Arguments**:

- `messages` _Union[str, List[str]]_ - The message(s) to generate responses for.
- `max_length` _int, optional_ - The maximum length of the generated response. Default: 100
  

**Examples**:

  >>> from aqueduct_llm import llama_7b
  >>> llama_7b.generate("What's the best LLM?", max_length=100)
  "LLaMA 7B is the best LLM!"

