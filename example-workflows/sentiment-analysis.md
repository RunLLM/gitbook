


<!-- ------------- New Cell ------------ -->


# Sentiment Analysis using Deep Learning

This is a short example of how to use Aqueduct to deploy a sentiment analysis model.

**You can find and download this notebook on GitHub [here](https://github.com/aqueducthq/aqueduct/blob/main/examples/sentiment-analysis/Sentiment%20Model.ipynb).**

Note: This example workflow uses HuggingFace's [Transformers](https://huggingface.co/docs/transformers/index) package, which uses large models. If you're running on a resource constrained machine, or if you're running on an M1 Mac using Rosetta, you will likely run out of memory for these models. We recommend using another example workflow if this is the case.

**Throughout this notebook, you'll see a decorator (`@aq.op`) above functions. This decorator allows Aqueduct to run your functions as a part of a workflow automatically.**




<!-- ------------- New Cell ------------ -->


```python
import aqueduct
from aqueduct.decorator import op, check
```




<!-- ------------- New Cell ------------ -->


```python
# If you're running your notebook on a separate machine from your
# Aqueduct server, change this to the address of your Aqueduct server.
address = "http://localhost:8080"

# If you're running youre notebook on a separate machine from your
# Aqueduct server, you will have to copy your API key here rather than
# using `get_apikey()`.
api_key = aqueduct.get_apikey()
```




<!-- ------------- New Cell ------------ -->


```python
client = aqueduct.Client(api_key, address)
```




<!-- ------------- New Cell ------------ -->


## Getting the Input Data




<!-- ------------- New Cell ------------ -->


First, we'll load some test data. Here, we'll use a dataset that has reviews of various hotels; our table has the name of the hotel, the date of the review, the nationality of the reviewer, and the text of the review itself. This data is preloaded for us in the [Aqueduct demo DB](https://docs.aqueducthq.com/example-workflows/demo-data-warehouse).




<!-- ------------- New Cell ------------ -->


```python
warehouse = client.resource("aqueduct_demo")

# reviews_table is an Aqueduct TableArtifact, which is a wrapper around
# a Pandas DataFrame. A TableArtifact can be used as argument to any operator
# in a workflow; you can also call .get() on a TableArtifact to retrieve
# the underlying DataFrame and interact with it directly.
reviews_table = warehouse.sql("select * from hotel_reviews;")
```




<!-- ------------- New Cell ------------ -->


```python
# This gets the head of the underlying DataFrame. Note that you can't
# pass a DataFrame as an argument to a workflow; you must use the Aqueduct
# TableArtifact!
reviews_table.get()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>hotel_name</th>
      <th>review_date</th>
      <th>reviewer_nationality</th>
      <th>review</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>H10 Itaca</td>
      <td>2017-08-03</td>
      <td>Australia</td>
      <td>Damaged bathroom shower screen sealant and ti...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>De Vere Devonport House</td>
      <td>2016-03-28</td>
      <td>United Kingdom</td>
      <td>No Negative The location and the hotel was ver...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ramada Plaza Milano</td>
      <td>2016-05-15</td>
      <td>Kosovo</td>
      <td>No Negative Im a frequent traveler i visited m...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aloft London Excel</td>
      <td>2016-11-05</td>
      <td>Canada</td>
      <td>Only tepid water for morning shower They said ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>The Student Hotel Amsterdam City</td>
      <td>2016-07-31</td>
      <td>Australia</td>
      <td>No Negative The hotel had free gym table tenni...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>95</th>
      <td>The Chesterfield Mayfair</td>
      <td>2015-08-25</td>
      <td>Denmark</td>
      <td>Bad Reading light And light in bathNo Positive</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Hotel V Nesplein</td>
      <td>2015-08-27</td>
      <td>Turkey</td>
      <td>Nothing except the construction going on the s...</td>
    </tr>
    <tr>
      <th>97</th>
      <td>Le Parisis Paris Tour Eiffel</td>
      <td>2015-10-20</td>
      <td>Australia</td>
      <td>When we arrived we had to bring our own baggag...</td>
    </tr>
    <tr>
      <th>98</th>
      <td>NH Amsterdam Museum Quarter</td>
      <td>2016-01-26</td>
      <td>Belgium</td>
      <td>No stairs even to go the first floor Restaura...</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Barcel Raval</td>
      <td>2017-07-07</td>
      <td>United Kingdom</td>
      <td>Air conditioning a little zealous Nice atmosp...</td>
    </tr>
  </tbody>
</table>
<p>100 rows × 4 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


## Applying the Model

Now that we have our data, we'll define an Aqueduct operator called `sentiment_prediction` that takes in our reviews data and appends a positive or negative label to the table as well as a score rating how positive or negative the review was.




<!-- ------------- New Cell ------------ -->


```python
from transformers import pipeline
import pandas as pd
import torch  # this is needed to ensure that pytorch is installed.

# The @op decorator here allows Aqueduct to run this function as
# a part of the Aqueduct workflow. It tells Aqueduct that when
# we execute this function, we're defining a step in the workflow.
# While the results can be retrieved immediately, nothing is
# published until we call `publish_flow()` below.
@op()
def sentiment_prediction(reviews):
    """
    This function uses the HuggingFace transformers library's sentiment-analysis
    model to predict the positive or negative sentiment of the reviews passed in
    to this function. The reviews argument is expected to have a `review` column
    and can have any other additional columns.

    This function will append the sentiment prediction as a column to the original
    DataFrame.
    """
    model = pipeline("sentiment-analysis")
    return reviews.join(pd.DataFrame(model(list(reviews["review"]))))
```




<!-- ------------- New Cell ------------ -->


```python
# This tells Aqueduct to execute sentiment_prediction on reviews_table
# as a part of our workflow. However, nothing is published (yet) until we
# call `publish_flow()` below.
sentiment_table = sentiment_prediction(reviews_table)
```




<!-- ------------- New Cell ------------ -->


We can see all the positive or negative labels as well as the numerical score generated by our sentiment model by calling `.get()` on the `sentiment_table`:




<!-- ------------- New Cell ------------ -->


```python
sentiment_table.get()
```
**Output**
<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>hotel_name</th>
      <th>review_date</th>
      <th>reviewer_nationality</th>
      <th>review</th>
      <th>label</th>
      <th>score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>H10 Itaca</td>
      <td>2017-08-03</td>
      <td>Australia</td>
      <td>Damaged bathroom shower screen sealant and ti...</td>
      <td>POSITIVE</td>
      <td>0.715813</td>
    </tr>
    <tr>
      <th>1</th>
      <td>De Vere Devonport House</td>
      <td>2016-03-28</td>
      <td>United Kingdom</td>
      <td>No Negative The location and the hotel was ver...</td>
      <td>POSITIVE</td>
      <td>0.999741</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ramada Plaza Milano</td>
      <td>2016-05-15</td>
      <td>Kosovo</td>
      <td>No Negative Im a frequent traveler i visited m...</td>
      <td>POSITIVE</td>
      <td>0.999773</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aloft London Excel</td>
      <td>2016-11-05</td>
      <td>Canada</td>
      <td>Only tepid water for morning shower They said ...</td>
      <td>NEGATIVE</td>
      <td>0.999169</td>
    </tr>
    <tr>
      <th>4</th>
      <td>The Student Hotel Amsterdam City</td>
      <td>2016-07-31</td>
      <td>Australia</td>
      <td>No Negative The hotel had free gym table tenni...</td>
      <td>NEGATIVE</td>
      <td>0.931378</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>95</th>
      <td>The Chesterfield Mayfair</td>
      <td>2015-08-25</td>
      <td>Denmark</td>
      <td>Bad Reading light And light in bathNo Positive</td>
      <td>NEGATIVE</td>
      <td>0.999340</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Hotel V Nesplein</td>
      <td>2015-08-27</td>
      <td>Turkey</td>
      <td>Nothing except the construction going on the s...</td>
      <td>POSITIVE</td>
      <td>0.999691</td>
    </tr>
    <tr>
      <th>97</th>
      <td>Le Parisis Paris Tour Eiffel</td>
      <td>2015-10-20</td>
      <td>Australia</td>
      <td>When we arrived we had to bring our own baggag...</td>
      <td>NEGATIVE</td>
      <td>0.999032</td>
    </tr>
    <tr>
      <th>98</th>
      <td>NH Amsterdam Museum Quarter</td>
      <td>2016-01-26</td>
      <td>Belgium</td>
      <td>No stairs even to go the first floor Restaura...</td>
      <td>POSITIVE</td>
      <td>0.996806</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Barcel Raval</td>
      <td>2017-07-07</td>
      <td>United Kingdom</td>
      <td>Air conditioning a little zealous Nice atmosp...</td>
      <td>POSITIVE</td>
      <td>0.999748</td>
    </tr>
  </tbody>
</table>
<p>100 rows × 6 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


It might be helpful to monitor the runtime of this sentiment_predictions operator. Aqueduct comes with a set of pre-built system metrics that allow you to capture system-level metrics like function runtime and memory usage. You can see all available system metrics, you can call `list_system_metrics`. Here, we'll use the `runtime` system metric to track how long it takes to compute the `sentiment_table` artifact.




<!-- ------------- New Cell ------------ -->


```python
sentiment_table.list_system_metrics()
```
**Output:**


```
['runtime', 'max_memory']
```






<!-- ------------- New Cell ------------ -->


```python
# Use an Aqueduct system metric to capture how long it takes to run
# the sentiment_prediction function that generates sentiment_table.
sentiment_table_runtime = sentiment_table.system_metric("runtime")
```




<!-- ------------- New Cell ------------ -->


Now you can view the runtime (in seconds) by retrieving the contents of the sentimment_table_runtime artifact.




<!-- ------------- New Cell ------------ -->


```python
sentiment_table_runtime.get()
```
**Output:**


```
8.313335418701172
```






<!-- ------------- New Cell ------------ -->


## Publishing the Workflow

Now that we've defined our predictions, we can save them back to the data warehouse. Here, we'll simply write them back to the same demo DB that we loaded the data from earlier, but the predictions can be written to any system Aqueduct is connected to.




<!-- ------------- New Cell ------------ -->


```python
# This tells Aqueduct to save the results in sentiment_table
# back to the demo DB we configured earlier.
# NOTE: At this point, no data is actually saved! This is just
# part of a workflow spec that will be executed once the workflow
# is published in the next cell.
warehouse.save(sentiment_table, table_name="sentiment_pred", update_mode="replace")
```




<!-- ------------- New Cell ------------ -->


Finally, we'll publish our workflow to Aqueduct, giving it a name and telling it which artifacts to publish. Optionally, we can also give this workflow a schedule, telling it to run on an hourly basis:




<!-- ------------- New Cell ------------ -->


```python
# This publishes all of the logic needed to create sentiment_table
# to Aqueduct. The URL below will take you to the Aqueduct UI, which
# will show you the status of your workflow runs and allow you to
# inspect them.
sentiment_flow = client.publish_flow(
    name="Demo Customer Sentiment",
    artifacts=[sentiment_table],
    # Uncomment the following line to schedule the workflow on a hourly basis.
    # schedule=aqueduct.hourly(),
)
print(sentiment_flow.id())
```




<!-- ------------- New Cell ------------ -->


Clicking on the URL above will take you to the Aqueudct UI where you can see the workflow that we just created! On the Aqueduct UI, you'll be able to see the DAG of operators we just created, click into any of those operators, and see the data and metadata associated with each stage of the pipeline.

