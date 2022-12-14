


<!-- ------------- New Cell ------------ -->


# Quickstart Tutorial

The quickest way to get your first workflow deployed on Aqueduct




<!-- ------------- New Cell ------------ -->


---

### Installation and Setup
First things first, we'll install the Aqueduct pip package and start Aqueduct in your terminal:

```
!pip3 install aqueduct-ml
!aqueduct start
```

Next, we import everything we need and create our Aqueduct client:




<!-- ------------- New Cell ------------ -->


```python
from aqueduct import Client, op, metric, check
import pandas as pd

client = Client()
```




<!-- ------------- New Cell ------------ -->


Note that the API key associated with the server can also be found in the output of the aqueduct start command.

---
### Accessing Data

The base data for our workflow is the [hotel reviews](https://docs.aqueducthq.com/integrations/aqueduct-demo-integration) dataset in the pre-built aqueduct_demo that comes with the Aqueduct server. This code does two things -- (1) it loads a connection to the demo database, and (2) it runs a SQL query against that DB and returns a pointer to the resulting dataset.




<!-- ------------- New Cell ------------ -->


```python
demo_db = client.integration("aqueduct_demo")
reviews_table = demo_db.sql("select * from hotel_reviews;")

# You will see the type of `reviews_table` is an Aqueduct TableArtifact.
print(type(reviews_table))

# Calling .get() allows us to retrieve the underlying data from the TableArtifact and
# returns it to you as a Python object.
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


`reviews_table` is an Artifact -- simply a wrapper around some data -- in Aqueduct terminology and will now serve as the base data for our workflow. We can apply Python functions to it in order to transform it.

---

### Transforming Data

A piece of Python code that transforms an Artifact is called an [Operator](https://docs.aqueducthq.com/operators), which is simply just a decorated Python function. Here, we'll write a simple operator that takes in our reviews table and calculates the length of the review string. It's not too exciting, but it should give you a sense of how Aqueduct works.




<!-- ------------- New Cell ------------ -->


```python
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




<!-- ------------- New Cell ------------ -->


Notice that we added @op above our function definition: This tells Aqueduct that we want to run this function as a part of an Aqueduct workflow. A function decorated with @op can be called like a regular Python function, and Aqueduct takes note of this call to begin constructing a workflow.

Now that we have our string length operator, we can get a preview of our data by calling .get()




<!-- ------------- New Cell ------------ -->


```python
strlen_table.get()
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
      <th>strlen</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>H10 Itaca</td>
      <td>2017-08-03</td>
      <td>Australia</td>
      <td>Damaged bathroom shower screen sealant and ti...</td>
      <td>82</td>
    </tr>
    <tr>
      <th>1</th>
      <td>De Vere Devonport House</td>
      <td>2016-03-28</td>
      <td>United Kingdom</td>
      <td>No Negative The location and the hotel was ver...</td>
      <td>84</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ramada Plaza Milano</td>
      <td>2016-05-15</td>
      <td>Kosovo</td>
      <td>No Negative Im a frequent traveler i visited m...</td>
      <td>292</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aloft London Excel</td>
      <td>2016-11-05</td>
      <td>Canada</td>
      <td>Only tepid water for morning shower They said ...</td>
      <td>368</td>
    </tr>
    <tr>
      <th>4</th>
      <td>The Student Hotel Amsterdam City</td>
      <td>2016-07-31</td>
      <td>Australia</td>
      <td>No Negative The hotel had free gym table tenni...</td>
      <td>167</td>
    </tr>
    <tr>
      <th>...</th>
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
      <td>47</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Hotel V Nesplein</td>
      <td>2015-08-27</td>
      <td>Turkey</td>
      <td>Nothing except the construction going on the s...</td>
      <td>456</td>
    </tr>
    <tr>
      <th>97</th>
      <td>Le Parisis Paris Tour Eiffel</td>
      <td>2015-10-20</td>
      <td>Australia</td>
      <td>When we arrived we had to bring our own baggag...</td>
      <td>672</td>
    </tr>
    <tr>
      <th>98</th>
      <td>NH Amsterdam Museum Quarter</td>
      <td>2016-01-26</td>
      <td>Belgium</td>
      <td>No stairs even to go the first floor Restaura...</td>
      <td>156</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Barcel Raval</td>
      <td>2017-07-07</td>
      <td>United Kingdom</td>
      <td>Air conditioning a little zealous Nice atmosp...</td>
      <td>72</td>
    </tr>
  </tbody>
</table>
<p>100 rows × 5 columns</p>
</div>




<!-- ------------- New Cell ------------ -->


---

### Adding Metrics

We're going to apply a [Metric](https://docs.aqueducthq.com/metrics-and-checks/metrics-measuring-your-predictions) to our strlen_table, which will calculate a numerical summary of our predictions (in this case, just the mean string length).




<!-- ------------- New Cell ------------ -->


```python
@metric
def average_strlen(strlen_table):
    return (strlen_table["strlen"]).mean()

avg_strlen = average_strlen(strlen_table)
avg_strlen.get()
```
**Output:**


```
223.18
```






<!-- ------------- New Cell ------------ -->


Note that metrics are denoted with the @metric decorator. Metrics can be computed over any operator, and even other metrics.

---
### Adding Checks

Let's say that we want to make sure the average strlen of hotel reviews never exceeds 250 characters. We can add a [check](https://docs.aqueducthq.com/metrics-and-checks/checks-ensuring-correctness) over the `avg_strlen` metric.




<!-- ------------- New Cell ------------ -->


```python
@check(severity="error")
def limit_avg_strlen(avg_strlen):
    return avg_strlen < 250

limit_avg_strlen(avg_strlen)
```
**Output:**


```
<aqueduct.artifacts.bool_artifact.BoolArtifact at 0x7f7e65b46ee0>
```






<!-- ------------- New Cell ------------ -->


Note that checks are denoted with the @check decorator. Checks can also computed over any operator or metric. Setting the severity to "error" will automatically fail the workflow if this check is ever violated. Check severity can also be set to "warning" (default), which only print a warning message on any violation.

---
### Saving Data
Finally, we can save the transformed table `strlen_table` back to the Aqueduct demo database. See [here](https://docs.aqueducthq.com/integrations/using-integrations) for more details around using integration objects.




<!-- ------------- New Cell ------------ -->


```python
demo_db.save(strlen_table, table_name="strlen_table", update_mode="replace")
```




<!-- ------------- New Cell ------------ -->


Note that this save is not performed until the flow is actually published.




<!-- ------------- New Cell ------------ -->


---

### Publishing the Flow

This creates the flow in Aqueduct. You will receive a URL below that will take you to the Aqueduct UI which will show you the status of your workflow runs, and allow you to inspect the data.




<!-- ------------- New Cell ------------ -->


```python
client.publish_flow(name="review_strlen", artifacts=[strlen_table])
```
**Output:**


```
<aqueduct.flow.Flow at 0x7f7e61d9cdc0>
```






<!-- ------------- New Cell ------------ -->


And we're done! We've created our first workflow together, and you're off to the races. 

---

There is a lot more you can do with Aqueduct, including having flows run automatically on a cadence, parameterizing flows, and reading to and writing from many different integrations (S3, Postgres, etc.). Check out the other tutorials and examples [here](https://docs.aqueducthq.com/example-workflows) for a deeper dive!

