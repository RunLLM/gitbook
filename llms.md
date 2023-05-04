---
description: Empower your workflow with LLMs
---

# Built-in LLM support

Aqueduct supports a number of built-in LLMs (large language models) that you can use as part of your workflow. These LLMs are available in the Aqueduct SDK and can be used as follows:

```python
import aqueduct as aq

vicuna = aq.llm_op(
    # the name of the LLM you want to use
    name="vicuna_7b",
    # specify which compute engine to run this LLM operator on
    engine="k8s",
)

response = vicuna(
    "How to generate revenue for my business?",
    {
        "prompt": "Given that we are an open source startup: ",
        "max_new_tokens": 512,
    },
)
response.get()
```

Operator `vicuna` takes in two arguments: an input message and an optional configuration dictionary. Within the dictionary, you can include a key called "prompt" which is used to give LLM some context of the input message. You can also specify any customizable parameters supported by the LLM. In this case, "max_new_tokens" specifies the maximum number of tokens that Vicuna will generate. For more information on the parameters supported by each LLM and their default values, please refer to the [LLM package documentation](https://docs.aqueducthq.com/api-reference/aqueduct-llm-reference/package-aqueduct-llm).

`response` returned by `vicuna()` is an Aqueduct artifact, which can be used as input to other operators in your workflow.

You can also feed a Pandas DataFrame to an LLM operator. In this case, we will send a column of your choice to the LLM, which generates a new column with the prediction, and returns a DataFrame with the new column appended to the original DataFrame.

```python
vicuna = aq.llm_op(
    name="vicuna_7b",
    engine="k8s",
    column_name="review", # the column to send to the LLM
    output_column_name="response", # the column name for the response
)

snowflake = client.resource("snowflake")

reviews_table = snowflake.sql("select * from hotel_reviews;")

updated_table = vicuna(reviews_table)
updated_table.get() # the response column is appended to the original table
```

Please refer to our [SDK documentation](https://docs.aqueducthq.com/api-reference/sdk-reference/package-aqueduct/aqueduct.llm_op) for the detailed usage of `llm_op` and the APIs of the generated LLM operators.

## Specifying compute engine to run LLM operators

As in the example above, you can specify the compute resources to run LLM operators by using the `engine` argument to the `llm_op` function. LLM operators are currently optimized for running on Kubernetes. When you specify the engine to be a Kubernetes resource, we automatically generate the appropriate resource requests (CPU, RAM, GPU, etc.) for the LLM you are using.

If you want to run LLM operators on other type of compute engines, you need to make sure the engine has sufficient hardware resources to run the LLM. Please refer to the [LLM package documentation](https://docs.aqueducthq.com/api-reference/aqueduct-llm-reference/package-aqueduct-llm) for the minimum requirements of each LLM.

## Using LLMs via the Aqueduct decorators

Although `llm_op` is the easiest way to invoke LLMs, you can also use the Aqueduct decorators (`op`, `metric`, `check`)
to combine LLMs with arbitrary Python code, which gives you more flexibility. All you need to do is to
include `aqueduct-llm` as a `requirements` when decorating your function:

```python
from aqueduct import op

@op(
    engine="k8s",
    requirements=["aqueduct-llm"],
    resources={
        'memory': '16GB',
        'gpu_resource_name': 'nvidia.com/gpu',
    }
) 
def vicuna(message):
    from aqueduct_llm import vicuna_7b
    # Your custom code here
    # ...
    # ...
    return vicuna_7b.generate(message, max_new_tokens=512)

response = vicuna("How to generate revenue for my business?")
```

Note that since the operator can contain arbitrary Python code, you need to explicitly specify the
resources required by the operator when running on Kubernetes. Please refer to the [LLM package documentation](https://docs.aqueducthq.com/api-reference/aqueduct-llm-reference/package-aqueduct-llm) for the minimum requirements of each LLM.