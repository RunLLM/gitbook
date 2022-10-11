


<!-- ------------- New Cell ------------ -->


# Quickstart Tutorial

This is a quick tutorial that will go through the bare minimum needed to set up a useful flow.

Make sure you have an Aqueduct server running locally. If you don't, run `aqueduct start` to start the server.




<!-- ------------- New Cell ------------ -->


---

### Step 1: Set up the Aqueduct client





<!-- ------------- New Cell ------------ -->


```python
from aqueduct import Client, op, get_apikey
client = Client(get_apikey(), "localhost:8080")
```




<!-- ------------- New Cell ------------ -->


### Step 2: Extract some data from the database
Pull some data out of the demo database, which is automatically included with the server.




<!-- ------------- New Cell ------------ -->


```python
demo_db = client.integration("aqueduct_demo")
reviews_table = demo_db.sql("select * from hotel_reviews;")
```




<!-- ------------- New Cell ------------ -->


### Step 3: Perform a transformation on the data
In this case, we'll append a new column 'strlen', which contains the length of the review.




<!-- ------------- New Cell ------------ -->


```python
@op
def transform_data(reviews):
    reviews['strlen'] = reviews['review'].str.len()
    return reviews

table_with_strlen = transform_data(reviews_table)
```




<!-- ------------- New Cell ------------ -->


You can materialize and inspect the resulting table using `.get()`.




<!-- ------------- New Cell ------------ -->


```python
table_with_strlen.get()
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


### Step 4: Save the transformed data back to the database

If you're happy with the result, you can save the data back to the database. This save is not performed until the flow is actually published.




<!-- ------------- New Cell ------------ -->


```python
table_with_strlen.save(demo_db.config(table="reviews_with_strlen", update_mode="replace")) 
```




<!-- ------------- New Cell ------------ -->


### Step 5: Publish the flow




<!-- ------------- New Cell ------------ -->


```python
flow = client.publish_flow(name="Append Strlen to Reviews", artifacts=[table_with_strlen])
print(flow.id())
```




<!-- ------------- New Cell ------------ -->


The published workflow is named "Append Strlen of Reviews". You can view it's runs on the UI (accessible in the browser at `localhost:8080`). It will extract from the reviews data, append a column, and write the result back into the database as a table named "reviews_with_strlen".




<!-- ------------- New Cell ------------ -->


---

There is a lot more you can do with Aqueduct, including having flows run automatically on a cadence, parameterizing flows, and reading to and writing from many different integrations (S3, Postgres, etc.). Check out the other tutorials and examples [here](https://docs.aqueducthq.com/example-workflows) if you're interested!

