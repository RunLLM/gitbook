---
description: Open-source prediction infrastructure for data scientists
---

# Welcome to Aqueduct

Aqueduct is open-source prediction infrastructure built for data scientists, by data scientists. With Aqueduct, data scientists can instantaneously deploy machine learning models to the cloud, connect those models to data and business systems, and gain visibility into the performance of their prediction pipelines -- all from the comfort of a Python notebook.

For more on why we're building prediction infrastructure for data scientists see [the-aqueduct-philosophy.md](the-aqueduct-philosophy.md "mention").

The core abstraction in Aqueduct is a [Workflow](workflows/), which is a sequence of [Artifacts](artifacts.md) (data) that are transformed by [Operators](operators.md) (compute). The input Artifact(s) for a Workflow is typically loaded from a database, and the output Artifact(s) are typically persisted back to a database. Each Workflow can either be run on a fixed schedule or triggered on-demand.

The 12-line code snippet below is all you need to create your first Aqueduct workflow:

```python
from aqueduct import Client, op, metric, get_apikey

# Create an Aqueduct client. If we're running on the same machine as the 
# Aqueduct server, we can create a client without providing an API key or a
# server address.
client = Client()

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

# With client.integration, we can load a connection to a database.
# Here, we use the Aqueduct demo DB.
demo_db = client.integration("aqueduct_demo")
reviews_table = demo_db.sql("select * from hotel_reviews;")

# Calling .get() allows us to retrieve the underlying data from the TableArtifact and
# returns it to you as a Python object.
print(reviews_table.get())

# Calling a decorated function returns another Aqueduct artifact.
strlen_table = transform_data(reviews_table)

# Artifacts can be saved -- here, we save the table with the appended strlen
# back to the Aqueduct demo DB with the table name `strlen_table`.
strlen_table.save(demo_db.config(table="strlen_table", update_mode="replace")) 

# This publishes the logic needed to create the strlen_table
# to Aqueduct. You will receive a URL below that will take you to the
# Aqueduct UI, which will show you the status of your workflow
# runs and allow you to inspect them.
client.publish_flow(name="review_strlen", artifacts=[strlen_table])

```

For more on this pipeline, check our [Quickstart Guide](quickstart-guide.md).

### Core Concepts

* [Workflows](workflows/)
* [Integrations](integrations/)
* [Operators](operators.md)
* [Artifacts](artifacts.md)
* [Metrics & Checks](metrics-and-checks.md)

### Example Workflows

* [The Demo Data Warehouse](integrations/aqueduct-demo-integration.md)
* [Churn Ensemble](example-workflows/churn-ensemble.md)
* [Wine Quality Prediction](example-workflows/train-and-inference.md)
* [Sentiment Analysis](example-workflows/sentiment-analysis.md)
* [Workflow Parameters](example-workflows/using-parameters.md)
* Product Recommendation (coming soon!)

### Guides

* [Debugging a Prediction Pipeline](guides/debugging-a-failed-workflow.md)
* Running with Airflow (coming soon!)

### API Reference

* [Python SDK](api-reference/sdk-reference/)
* [Aqueduct CLI](api-reference/aqueduct-cli.md)
