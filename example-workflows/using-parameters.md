


<!-- ------------- New Cell ------------ -->


This is a quick tutorial that will demonstrate how workflows can be parameterized with Aqeuduct.

**You can find and download this notebook on GitHub [here](https://github.com/aqueducthq/aqueduct/blob/main/examples/parameterization/Using%20Parameters.ipynb).**

**Throughout this notebook, you'll see a decorator (`@aq.op`) above functions. This decorator allows Aqueduct to run your functions as a part of a workflow automatically.**



<!-- ------------- New Cell ------------ -->


```python
import aqueduct
from aqueduct.decorator import op

# If you're running your notebook on a separate machine from your
# Aqueduct server, change this to the address of your Aqueduct server.
address = "http://localhost:8080"

# If you're running your notebook on a separate machine from your 
# Aqueduct server, you will have to copy your API key here rather than
# using `get_apikey()`.
api_key = aqueduct.get_apikey()
client = aqueduct.Client(api_key, address)
```




<!-- ------------- New Cell ------------ -->


--- 

## The Basics

A parameter is an argument to a whole workflow that acts exactly like any other Aqueduct artifact. It can be fed as a typical artifact input to any operator. Every parameter must have both a name and a default value, which we will use if not overriding value is provided.

In the example below, we will attempt to filter a table based on the value of a specific column (`reviewer_nationality`). The value we filter on will be parameterized.




<!-- ------------- New Cell ------------ -->


```python
db = client.integration("aqueduct_demo")

# reviews_table is an Aqueduct TableArtifact, which is a wrapper around
# a Pandas DataFrame. A TableArtifact can be used as argument to any operator
# in a workflow; you can also call .get() on a TableArtifact to retrieve
# the underlying DataFrame and interact with it directly.
reviews_table = db.sql("select * from hotel_reviews;")
```




<!-- ------------- New Cell ------------ -->


```python
# This gets the underlying DataFrame. Note that you can't pass a
# DataFrame as an argument to a workflow; you must use the Aqueduct
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


```python
import pandas as pd


@op
def strip_whitespace_from_nationality(df: pd.DataFrame):
    """
    This function takes a Pandas DataFrame for hotel_reviews and
    removes the unnecessary whitespace around the reviewer's nationality.
    The reviewer_nationality data is loaded with inconsistent spacing,
    so this is a necessary data cleaning step before further featurization.
    """
    df["reviewer_nationality"] = df["reviewer_nationality"].str.strip(" ")
    return df


@op
def filter_by_nationality(df: pd.DataFrame, target_nationality: str):
    """
    This function takes in a Pandas DataFrame for hotel_reviews and
    filters it by the nationality parameter passed in to this function.
    This filter should only be invoked after the whitespace stripping
    data cleaning operation above, otherwise it will result in inconsisten
    results.
    """
    return df[df["reviewer_nationality"] == target_nationality]
```




<!-- ------------- New Cell ------------ -->


Here we filter the reviews table to only the rows where `reviewer_nationality` is equal to our parameter value, which currently defaults to "United Kingdom".




<!-- ------------- New Cell ------------ -->


```python
# We define a workflow parameter called nationality_param and give
# it a default value of United Kingdom. This parameter can be used
# in any SQL query or Python operator in this workflow.
nationality_param = client.create_param("nationality", default="United Kingdom")

formatted_table = strip_whitespace_from_nationality(reviews_table)

# Here, we use the nationality_param defined above as an argument to
# filter by nationality.
filtered = filter_by_nationality(formatted_table, nationality_param)
filtered.get().head(10)
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
      <td>De Vere Devonport House</td>
      <td>2016-03-28</td>
      <td>United Kingdom</td>
      <td>No Negative The location and the hotel was ver...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Crowne Plaza London Docklands</td>
      <td>2017-01-23</td>
      <td>United Kingdom</td>
      <td>Lighting in hotel room wasn t the best was ve...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>London Marriott Hotel Marble Arch</td>
      <td>2016-02-23</td>
      <td>United Kingdom</td>
      <td>No Negative The whole experience was excellent...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Grand Royale London Hyde Park</td>
      <td>2017-01-04</td>
      <td>United Kingdom</td>
      <td>see above window looking out on the rough sid...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>The Cavendish London</td>
      <td>2016-02-02</td>
      <td>United Kingdom</td>
      <td>Poor pillows for sleeping good for watching t...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>San Domenico House</td>
      <td>2016-06-13</td>
      <td>United Kingdom</td>
      <td>The coffee and drinks are quite expensive but ...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Radisson Blu Edwardian Grafton</td>
      <td>2016-05-18</td>
      <td>United Kingdom</td>
      <td>Pillows hard Evening staff on desk could not ...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Holiday Inn London Kensington</td>
      <td>2016-10-04</td>
      <td>United Kingdom</td>
      <td>Put in a disabled room when we weren t disabl...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>The Hoxton Holborn</td>
      <td>2016-05-25</td>
      <td>United Kingdom</td>
      <td>The loud music in the bar area in the evening...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Park Plaza County Hall London</td>
      <td>2016-10-27</td>
      <td>United Kingdom</td>
      <td>Breakfast seating too cramped Not enough spac...</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


---
Since we've already parameterized the workflow, we can provide a different parameter value and see the new results immediately!




<!-- ------------- New Cell ------------ -->


```python
# When calling .get() on an artifact, we can provide a map of parametres
# to see how different parametrization affects the execution of our workflow.
# Here, we change the default value ("United Kingdom") to a new value
# ("Australia").
filtered.get(parameters={"nationality": "Australia"}).head(10)
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
      <td>The Student Hotel Amsterdam City</td>
      <td>2016-07-31</td>
      <td>Australia</td>
      <td>No Negative The hotel had free gym table tenni...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Les Jardins Du Marais</td>
      <td>2015-10-27</td>
      <td>Australia</td>
      <td>Bathroom is fine but could be improved with a...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Napoleon Paris</td>
      <td>2015-10-07</td>
      <td>Australia</td>
      <td>NOTHING EVERYTHING</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NH Milano Touring</td>
      <td>2015-08-09</td>
      <td>Australia</td>
      <td>Check in and check out were extemely slow wit...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Le Parisis Paris Tour Eiffel</td>
      <td>2015-10-20</td>
      <td>Australia</td>
      <td>When we arrived we had to bring our own baggag...</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


---
## Publishing a Parameterized Flow

When a flow is published on a schedule, the recurring run will continously execute with the default parameters. You may trigger additional runs with different parameters using the `client.trigger()` method. However, in order to change a parameter's value in perpetuity, you must rerun `create_param()` with an updated default value.





<!-- ------------- New Cell ------------ -->


```python
flow = client.publish_flow("Parameter Example", artifacts=[filtered])
print(flow.id())
```




<!-- ------------- New Cell ------------ -->


```python
# Wait until the flow has run at least once before triggering a new run.
from time import sleep

while len(flow.list_runs()) == 0:
    sleep(1)

client.trigger(flow.id(), parameters={"nationality": "Australia"})
```




<!-- ------------- New Cell ------------ -->


---
### SQL Query Parameters

SQL queries can also be parameterized. For queries, we'll use the double-bracket syntax to denote the presence of a parameter inline. As long as the name of the parameter matches a previously defined. 

Here is the same flow as above, but as a parameterized SQL query instead.

NOTE: unlike parameter names for other operators, multi-part SQL parameter names cannot be separated with spaces. For example, "review-date" is allowed but "review date" is not.




<!-- ------------- New Cell ------------ -->


```python
_ = client.create_param("nationality", default="United Kingdom")

table = db.sql("select * from hotel_reviews where reviewer_nationality=' {{nationality}} '")
table.get().head(10)
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
      <td>De Vere Devonport House</td>
      <td>2016-03-28</td>
      <td>United Kingdom</td>
      <td>No Negative The location and the hotel was ver...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Crowne Plaza London Docklands</td>
      <td>2017-01-23</td>
      <td>United Kingdom</td>
      <td>Lighting in hotel room wasn t the best was ve...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>London Marriott Hotel Marble Arch</td>
      <td>2016-02-23</td>
      <td>United Kingdom</td>
      <td>No Negative The whole experience was excellent...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Grand Royale London Hyde Park</td>
      <td>2017-01-04</td>
      <td>United Kingdom</td>
      <td>see above window looking out on the rough sid...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>The Cavendish London</td>
      <td>2016-02-02</td>
      <td>United Kingdom</td>
      <td>Poor pillows for sleeping good for watching t...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>San Domenico House</td>
      <td>2016-06-13</td>
      <td>United Kingdom</td>
      <td>The coffee and drinks are quite expensive but ...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Radisson Blu Edwardian Grafton</td>
      <td>2016-05-18</td>
      <td>United Kingdom</td>
      <td>Pillows hard Evening staff on desk could not ...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Holiday Inn London Kensington</td>
      <td>2016-10-04</td>
      <td>United Kingdom</td>
      <td>Put in a disabled room when we weren t disabl...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>The Hoxton Holborn</td>
      <td>2016-05-25</td>
      <td>United Kingdom</td>
      <td>The loud music in the bar area in the evening...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Park Plaza County Hall London</td>
      <td>2016-10-27</td>
      <td>United Kingdom</td>
      <td>Breakfast seating too cramped Not enough spac...</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


```python
table.get(parameters={"nationality": "Australia"})
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
      <td>The Student Hotel Amsterdam City</td>
      <td>2016-07-31</td>
      <td>Australia</td>
      <td>No Negative The hotel had free gym table tenni...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Les Jardins Du Marais</td>
      <td>2015-10-27</td>
      <td>Australia</td>
      <td>Bathroom is fine but could be improved with a...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Napoleon Paris</td>
      <td>2015-10-07</td>
      <td>Australia</td>
      <td>NOTHING EVERYTHING</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NH Milano Touring</td>
      <td>2015-08-09</td>
      <td>Australia</td>
      <td>Check in and check out were extemely slow wit...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Le Parisis Paris Tour Eiffel</td>
      <td>2015-10-20</td>
      <td>Australia</td>
      <td>When we arrived we had to bring our own baggag...</td>
    </tr>
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


---
### Builtin SQL Parameters

There are also a number of builtin parameter tags that we support for you! See [our documentation](https://docs.aqueducthq.com/workflows/parameterizing-a-workflow) for a list of all of built-in parameters.

Below is an example of the fairly self-explanatory `today` parameter:




<!-- ------------- New Cell ------------ -->


```python
# This will be empty because all records are historical.
reviews_after_today = db.sql("select * from hotel_reviews where review_date > {{ today }}")
reviews_after_today.get()
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
  </tbody>
</table>
</div>




<!-- ------------- New Cell ------------ -->


```python
reviews_before_today = db.sql("select * from hotel_reviews where review_date < {{ today }}")
reviews_before_today.get()
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


---
### Implicitly Defined Parameters

If you pass a regular python object into an operator, we will automatically convert it into a parameter for you!




<!-- ------------- New Cell ------------ -->


```python
# The name of the parameter will be `target_nationality`, since that is the corresponding function name
# in filter_by_nationality()'s signature.
# Note: we currently do not allow you to create implicit parameters with the same name as an existing parameter. If
# the argument name was `nationality` instead of `target_nationality`, this would have failed since we previously
# defined explicit parameter `nationality`.
filtered = filter_by_nationality(formatted_table, "Australia")
filtered.get().head(10)
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
      <td>The Student Hotel Amsterdam City</td>
      <td>2016-07-31</td>
      <td>Australia</td>
      <td>No Negative The hotel had free gym table tenni...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Les Jardins Du Marais</td>
      <td>2015-10-27</td>
      <td>Australia</td>
      <td>Bathroom is fine but could be improved with a...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Napoleon Paris</td>
      <td>2015-10-07</td>
      <td>Australia</td>
      <td>NOTHING EVERYTHING</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NH Milano Touring</td>
      <td>2015-08-09</td>
      <td>Australia</td>
      <td>Check in and check out were extemely slow wit...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Le Parisis Paris Tour Eiffel</td>
      <td>2015-10-20</td>
      <td>Australia</td>
      <td>When we arrived we had to bring our own baggag...</td>
    </tr>
  </tbody>
</table>
</div>

