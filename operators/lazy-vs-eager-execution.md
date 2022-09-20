# Lazy vs Eager Executions

Operators in Python have two modes of execution supported by Aqueduct.
By default the mode of execution is `eager`. This can be configured via the `GlobalConfig` as shown below.

* Lazy Execution: The function will not be run when invoked. The return type of the result Artifact will be `untyped`. When requesting the contents of this Artifact via the `.get()` command, Aqueduct will run the function and populate both the concrete type info and the value for the artifact
* Eager Execution: The function will be run when invoked, the resultant artifact will already store the contents and the correct type information

```python
import aqueduct as aq
import pandas as pd

# We can use the a boolean indicated by the key "lazy" to tell Aqueduct how we want execution to proceed
aq.global_config({"lazy" : True})

@aq.op
def my_fn(df):
    return df

# Function will not run on the Aqueduct backend yet so this call is non-blocking
local_result = my_fn.local(pd.DataFrame())

# Aqueduct will now execute my_fn to get the contents
print(local_result.get())
```

Its important to keen in mind that [Metric & Checks](../metrics-and-checks.md) are similiar to operators and so follow the same execution modes as defined by the `GlobalConfig`.
