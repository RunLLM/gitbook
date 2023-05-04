# Table of Contents

* [aqueduct.llm\_wrapper](#aqueduct.llm_wrapper)
  * [llm\_op](#aqueduct.llm_wrapper.llm_op)

<a id="aqueduct.llm_wrapper"></a>

# aqueduct.llm\_wrapper

<a id="aqueduct.llm_wrapper.llm_op"></a>

#### llm\_op

```python
def llm_op(
    name: str,
    op_name: Optional[str] = None,
    column_name: Optional[str] = None,
    output_column_name: Optional[str] = None,
    engine: Optional[Union[str, DynamicK8sResource]] = None
) -> Union[Callable[..., Union[BaseArtifact, List[BaseArtifact]]],
           BaseArtifact, List[BaseArtifact]]
```

Generates an Aqueduct operator to run a LLM. Either both column_name and output_column_name must be provided,
or neither must be provided. Please refer to the `Returns` section below for their differences.

**Arguments**:

  name:
  The name of the LLM to use. Please see aqueduct.supported_llms for a list of supported LLMs.
  
  op_name:
  The name of the operator. If not provided, defaults to the name of the LLM.
  
  column_name:
  The name of the column of the Dataframe to use as input to the LLM. If this field is provided,
  output_column_name must also be provided.
  
  output_column_name:
  The name of the column of the Dataframe to store the output of the LLM. If this field is provided,
  column_name must also be provided.
  
  engine:
  The name of the compute resource this operator will run on. Defaults to the Aqueduct engine.
  We recommend using a Kubernetes engine to run LLM operators, as we have implemented performance
  optimizations for LLMs on Kubernetes.
  

**Returns**:

  If column_name and output_column_name are both provided, returns a function that takes in a
  DataFrame and returns a DataFrame with the output of the LLM appended as a new column:
  
  ```python
  def use_llm_for_table(df: pd.DataFrame, parameters: Dict[str, Any] = {}) -> pd.DataFrame:
  ```
  
  Otherwise, returns a function that takes in a string or list of strings, applies LLM, and
  returns a string or list of strings:
  
  ```python
  def use_llm(messages: Union[str, List[str]], parameters: Dict[str, Any] = {}) -> Union[str, List[str]]:
  ```
  
  In both cases, the function takes in an optional second argument, which is a dictionary of
  parameters to pass to the LLM. Please refer to the documentation for the LLM you are using
  for a list of supported parameters. For all LLMs, we support the "prompt" parameter. If the
  prompt contains {text}, we will replace {text} with the input string(s) before sending to
  the LLM. If the prompt does not contain {text}, we will prepend the prompt to the input
  string(s) before sending to the LLM.
  

**Examples**:

  ```python
  >>> from aqueduct import Client
  >>> client = Client()
  >>> snowflake = client.resource("snowflake")
  >>> reviews_table = snowflake.sql("select * from hotel_reviews;")
  
  >>> from aqueduct import llm_op
  >>> vicuna_table_op = llm_op(
  ...     name="vicuna_7b",
  ...     op_name="my_vicuna_operator",
  ...     column_name="review",
  ...     output_column_name="response",
  ...     engine=ondemand_k8s,
  ... )
  >>> params = client.create_param(
  ...     "vicuna_params",
  ...     default={
  ...         "prompt": "Respond to the following hotel review as a customer service agent: {text} ",
  ...         "max_gpu_memory": "13GiB",
  ...         "temperature": 0.7,
  ...         "max_new_tokens": 512,
  ...     }
  ... )
  >>> review_with_response = vicuna_table_op(reviews_table, params)
  
  `review_with_response` is a Table Artifact with the output of the LLM appended as a new column.
  
  >>> review_with_response.get()
  ```

