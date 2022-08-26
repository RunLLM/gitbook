---
description: The quickest way to get your first workflow deployed on Aqueduct
---

# Quickstart Guide

### Installation and Setup

First things first, we'll install the Aqueduct pip package and start Aqueduct:

```bash
pip3 install aqueduct-ml
aqueduct start
```

Once your server is up and running, you can start building your first workflow. We'll use a simple workflow here just to get ourselves acquainted with how Aqueduct works. We prefer to write our workflows in Jupyter notebooks, but this should work fine from a regular Python interpreter as well.

First, we'll import everything we need:

```python
from aqueduct import Client, get_apikey, op, metric
import pandas as pd
```

Next, we'll create our API client:

```python
client = Client(get_apikey(), "localhost:8080")
```

Note that the API key associated with the server can also be found in the output of the `aqueduct start` command.

And access the base data for our workflow, which is the [hotel reviews dataset](integrations/aqueduct-demo-integration.md). This code does two things -- (1) it loads a connection to a database (in this case the pre-build `aqueduct_demo` DB that comes with the Aqueduct server), and (2) it runs a SQL query against that DB and returns a pointer to the resulting dataset.

```python
demo_db = client.integration("aqueduct_demo")
reviews_table = demo_db.sql("select * from hotel_reviews;")

# You will see the type of `reviews_table` is an Aqueduct TableArtifact.
print(type(reviews_table))

# Calling .get() allows us to retrieve the underlying data from the TableArtifact and
# returns it to you as a Python object.
reviews_table.get()
```

`reviews_table` is an [Artifact](artifacts.md) -- simply a wrapper around some data -- in Aqueduct terminology and will now serve as the base data for our workflow. We can apply Python functions to it in order to transform it.

A piece of Python code that transforms an Artifact is called an [Operator](operators.md), which is simply just a decorated Python function. Here, we'll write a simple operator that takes in our reviews table and calculates the length of the review string. It's not too exciting, but it should give you a sense of how Aqueduct works.

```python
# The @op decorator here allows Aqueduct to run this function as 
# a part of an Aqueduct workflow. It tells Aqueduct that when 
# we execute this function, we're defining a step in the workflow.
@op
def transform_data(reviews):
    '''
    This simple Python function takes in a DataFrame with hotel reviews
    and adds a column called strlen that has the string length of the
    review.    
    '''
    reviews['strlen'] = reviews['review'].str.len()
    return reviews

strlen_table = transform_data(reviews_table)
```

Notice that we added `@op` above our function definition: This tells Aqueduct that we want to run this function as a part of an Aqueduct workflow (in the cloud). A function decorated with `@op` can be called like a regular Python function, and Aqueduct takes note of this call to begin constructing a workflow.

Now that we have a our string length operator, we can get a preview of our data by calling `.get()`:

```python
strlen_table.get()
```

Next, we're going to apply a [Metric](metrics-and-checks/metrics-measuring-your-predictions/) to our `strlen_table`, which will calculate a numerical summary of our predictions (in this case, just the mean string length).&#x20;

```python
@metric
def average_strlen(strlen_table):
    return (strlen_table["strlen"]).mean()

avg_strlen = average_strlen(strlen_table)
```

Finally, we're going to save `strlen_table` back to the Aqueduct demo database by calling `.save()` and passing in a configuration for the demo database which indicates the name of the table we will use (see [here](integrations/using-integrations/) for more details), and we save the workflow to the Aqueduct server by calling `publish_flow`.

```python
# Artifacts can be saved -- here, we save the table with the appended strlen
# back to the Aqueduct demo DB with the table name `strlen_table`.
strlen_table.save(demo_db.config(table="strlen_table", update_mode="replace")) 

# This publishes the logic needed to create the strlen_table
# to Aqueduct. You will receive a URL below that will take you to the
# Aqueduct UI, which will show you the status of your workflow
# runs and allow you to inspect them.
client.publish_flow(name="review_strlen", artifacts=[strlen_table])
```

And we're done! 🎉

We've created our first workflow together, and you're off to the races. Here are some next steps:

* For more details on different ways to configure Aqueduct, check out our [Installation Guide](installation-and-deployment.md).
* Check out our [example workflows](example-workflows/) for some more Aqueduct workflows.
* Check out our guide on [Workflows](workflows/) for a deep dive on how to define, preview, and create workflows.
* Check out our documentation on [Operators](operators.md), [Artifacts](artifacts.md), and [Metrics & Checks](metrics-and-checks.md) for deep dives into creating and interacting with each.
