# Creating a Python Operator

Python Operators are simply Python functions annotated with the `@op` decorator, which is provided by the Aqueduct SDK.&#x20;

```python
import pandas as pd
from aqueduct import op

@op
def my_prediction_function(input: pd.DataFrame) -> pd.DataFrame:
    # Make your predictions here.
    return predictions
```

When a Python function is annotated with `@op`, two things happen:

1. Aqueduct packages up this code and infers any library dependencies that it might require, and prepares the code to run as a part of a workflow.&#x20;
2. The annotated function will accept Aqueduct [Artifacts](../artifacts.md) as inputs and return Artifacts as outputs -- the return values can in turn by used by other operators.&#x20; 
   You can also pass raw python objects in, which we will implicitly convert into parameter artifacts for you.

### Executing decorated functions locally

When a function is decorated with `@op`, calling the function will, by default, execute it on the Aqueduct server. However, for testing or validation purposes, you might want to occasionally execute that function locally in your existing Python process.&#x20;

To call an existing function locally, you can simply add `.local` before invoking the function:

```python
import aqueduct as aq
import pandas as pd

@aq.op
def my_fn(df):
    return df

local_result = my_fn.local(pd.DataFrame())
```

### Converting existing Python functions into operators

Sometimes, you will have Python functions defined previously or elsewhere that you would like to convert into Aqueduct operators. Redefining the function can be repetitive and cumbersome; instead, you can use `aqueduct.to_operator` to convert an existing Python function into an Aqueduct operator:

```python
import aqueduct as aq

def my_fn(df):
    return df
    
# my_fn_op is now an Aqueduct operator that can be invoked as
# a part of a workflow and is ready to be run in the cloud.
my_fn_op = aq.to_operator(my_fn)
```
