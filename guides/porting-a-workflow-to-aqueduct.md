---
description: Going from existing Python code to an Aqueduct workflow
---

# Porting a workflow to Aqueduct

This guide will walk you through the process of taking existing Python code and setting it up to run in Aqueduct. There's multiple ways in which we can do this, and the right one for you will depend on how complicated your workflow is and how much intermediary data you have.&#x20;

At the core of Aqueduct is the concept of [operators.md](../operators.md "mention") and [artifacts.md](../artifacts.md "mention"). An Operator is simply a Python function that (optionally) takes in and outputs some data. An Artifact is a wrapper around your data that allows Aqueduct to track, version, and validate your data. Operators generate Artifacts, which then serve as inputs to downstream Operators. This whole sequence constitutes the concept of [workflows](../workflows/ "mention").

So, how do we get your code running in an Aqueduct workflow?

* _Just get my code running!_ If you simply have some Python code that you want to schedule to run in Aqueduct, you can wrap all of your code into a single Aqueduct operator. This should take only a few minutes.
* _Help me track my inputs and outputs._ If you'd like Aqueduct to track your inputs and outputs without worrying about all the intermediary changes, you'll need to make a few changes to your code. We'll walk you through this below.
* _I want the full experience_. Aqueduct really shines when you can see all of your intermediary code and data and even attach [metrics-and-checks.md](../metrics-and-checks.md "mention") to it. This will take us a little bit longer though.

#### Just get my code running!

If you just want to get your code running in Aqueduct, things should be pretty simple. All you'll have to do is get your Python code into a single function and add Aqueduct's `@op` decorator above it.

```python
from aqueduct import Client, op
client = Client()

@op
def make_predictions():
    # First, read some data.
    # Then clean it up.
    # Next, we'll featurize it.
    # Finally, we'll load our model and make some predictions.
    # And write those predictions to our database.
```

Once we've done, that all we need to do is invoke this function and publish the results as an Aqueduct workflow:&#x20;

```python
none_artf = make_predictions()
client.publish_flow("prediction workflow", artifacts=[none_artf])
```

You might be wondering what `none_artf` is: Aqueduct is built to track your data and code together. We don't force you to return your data via Aqueduct, but we do need an output to track what you executed, and here, we're simply returning `None`.

And we're done! You can see the code that you ran and set it to run on a periodic cadence, but right now, Aqueduct won't track anything for you for the time being. We'll get to that next.

#### Help me track my inputs and outputs

Once you have your code running on Aqueduct, you probably are going to want to see what data is coming into and going out of your workflow. For the most part, this isn't going to require us to change much about our workflows.&#x20;

The first thing we'll need to do is figure out where our data inputs are coming from and where our predictions are going to. You'll need to connect those systems as Aqueduct [integrations](../integrations/ "mention").&#x20;

Once we have our integrations connected, we can get a handle to that integration in our Python code. For our example here, we're going to use the [aqueduct-demo-integration.md](../integrations/aqueduct-demo-integration.md "mention"). Once we have a handle to the demo database, we can then run a SQL query on it (see [aws-s3.md](../integrations/using-integrations/aws-s3.md "mention") for details on using non-relational data systems) to get our input data. You can use any SQL query that works for your underlying database.

```python
from aqueduct import Client, op
client = Client()

db = client.integration('aqueduct_demo') 
input_data = db.sql('SELECT * FROM wine;')
```

To see our input data, we can simply run:&#x20;

```python
input_data.get()
```

This data is now going to serve as the basis for our workflow. The only thing we'll need to change from our code is to remove the steps where we retrieve our data and save our predictions. Our updated `make_predictions` function will have a format like this:

```python
@op
def make_predictions(input_data):
    # Clean my data up.
    # Next, we'll featurize it.
    # Finally, we'll load our model and make some predictions.
    return predictions
```

Now, Aqueduct knows where the input data comes from (the SQL query above) as well as what the output data looks like (the `predictions` data returned from `make_predictions`). To save that data to our database, we can call `.save`:

```python
predictions = make_predictions()
predictions.save(db.config(table='my_predictions', mode='replace))
```

This tells Aqueduct to save `predictions` back to our database (the Aqueduct demo DB in this case) into a table called `my_predictions` and to replace the whole table every time the workflow executes. Once more, we can publish this workflow, and now we have an Aqueduct workflow will all our logic captured but clear visibility into our inputs and outputs:

```python
client.publish_flow("prediction workflow", artifacts=[predictions])
```

#### I want the full experience

You can probably see where we're going with this. To harness the full power of Aqueduct, you can track not just your inputs and outputs but all your intermediary data as well. Aqueduct can help you see where exactly in your workflow an error occurred and give you access to that data to start debugging the issue.&#x20;

To do this, we'll take our code above and split it into multiple stages. Using the steps defined above, we'll have three functions:

```python
from aqueduct import Client, op
client = Client()

db = client.integration('aqueduct_demo')
input_data = db.sql('SELECT * FROM wine;')

@op
def clean_data(input_data):
    # First, clean our data.
    return cleaned_data
    
@op 
def featurize_data(cleaned_data):
    # Next, featurize our data.
    return features
    
@op
def predict(features):
    # Finally, load our model and make some predictions.
    return predictions
```

As before, we can chain the output of each function as the input to the next function to create our workflow:

```python
cleaned_data = clean_data(input_data)
features = featurize_data(cleaned_data)
predictions = predict(features)
```

With these results, we can save and publish a workflow just like before:

```python
predictions.save(db.config(table='my_predictions', mode='replace))
client.publish_flow("prediction workflow", artifacts=[predictions])
```

Just like before, Aqueduct will capture `input_data` and `predictions` on every workflow run, but you will now also be able to see `cleaned_data` and `features` on the Aqueduct UI. This is critical for helping understand when things go wrong -- each function will be shown with its own stack trace and any associated logs.&#x20;

You can also attach [metrics-and-checks.md](../metrics-and-checks.md "mention")to any of the intermediary [artifacts.md](../artifacts.md "mention") to help measure and validate them over time.&#x20;

### Wrapping Up

Okay, you're good to go! You have your full workflow running in Aqueduct. If you have any questions or feedback about this, let us know on [Slack](https://slack.aqueducthq.com) or [GitHub](https://github.com/aqueducthq/aqueduct/issues/new/choose).&#x20;
