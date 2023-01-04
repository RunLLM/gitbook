# Using Parameters

## Workflow Parameters Tutorial

This is a quick tutorial that will demonstrate how workflows can be parameterized with Aqeuduct.

**You can find and download this notebook on GitHub** [**here**](https://github.com/aqueducthq/aqueduct/blob/main/examples/parameterization/Using%20Parameters.ipynb)**.**

**Throughout this notebook, you'll see a decorator (`@aq.op`) above functions. This decorator allows Aqueduct to run your functions as a part of a workflow automatically.**

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

***

### The Basics

A parameter is an argument to a whole workflow that acts exactly like any other Aqueduct artifact. It can be fed as a typical artifact input to any operator. Every parameter must have both a name and a default value, which we will use if not overriding value is provided.

In the example below, we will attempt to filter a table based on the value of a specific column (`reviewer_nationality`). The value we filter on will be parameterized.

```python
db = client.integration("aqueduct_demo")

# reviews_table is an Aqueduct TableArtifact, which is a wrapper around
# a Pandas DataFrame. A TableArtifact can be used as argument to any operator
# in a workflow; you can also call .get() on a TableArtifact to retrieve
# the underlying DataFrame and interact with it directly.
reviews_table = db.sql("select * from hotel_reviews;")
```

```python
# This gets the underlying DataFrame. Note that you can't pass a
# DataFrame as an argument to a workflow; you must use the Aqueduct
# TableArtifact!
reviews_table.get()
```

**Output**

|     | hotel\_name                      | review\_date | reviewer\_nationality | review                                            |
| --- | -------------------------------- | ------------ | --------------------- | ------------------------------------------------- |
| 0   | H10 Itaca                        | 2017-08-03   | Australia             | Damaged bathroom shower screen sealant and ti...  |
| 1   | De Vere Devonport House          | 2016-03-28   | United Kingdom        | No Negative The location and the hotel was ver... |
| 2   | Ramada Plaza Milano              | 2016-05-15   | Kosovo                | No Negative Im a frequent traveler i visited m... |
| 3   | Aloft London Excel               | 2016-11-05   | Canada                | Only tepid water for morning shower They said ... |
| 4   | The Student Hotel Amsterdam City | 2016-07-31   | Australia             | No Negative The hotel had free gym table tenni... |
| ... | ...                              | ...          | ...                   | ...                                               |
| 95  | The Chesterfield Mayfair         | 2015-08-25   | Denmark               | Bad Reading light And light in bathNo Positive    |
| 96  | Hotel V Nesplein                 | 2015-08-27   | Turkey                | Nothing except the construction going on the s... |
| 97  | Le Parisis Paris Tour Eiffel     | 2015-10-20   | Australia             | When we arrived we had to bring our own baggag... |
| 98  | NH Amsterdam Museum Quarter      | 2016-01-26   | Belgium               | No stairs even to go the first floor Restaura...  |
| 99  | Barcel Raval                     | 2017-07-07   | United Kingdom        | Air conditioning a little zealous Nice atmosp...  |

100 rows × 4 columns

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

Here we filter the reviews table to only the rows where `reviewer_nationality` is equal to our parameter value, which currently defaults to "United Kingdom".

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

|   | hotel\_name                       | review\_date | reviewer\_nationality | review                                            |
| - | --------------------------------- | ------------ | --------------------- | ------------------------------------------------- |
| 0 | De Vere Devonport House           | 2016-03-28   | United Kingdom        | No Negative The location and the hotel was ver... |
| 1 | Crowne Plaza London Docklands     | 2017-01-23   | United Kingdom        | Lighting in hotel room wasn t the best was ve...  |
| 2 | London Marriott Hotel Marble Arch | 2016-02-23   | United Kingdom        | No Negative The whole experience was excellent... |
| 3 | Grand Royale London Hyde Park     | 2017-01-04   | United Kingdom        | see above window looking out on the rough sid...  |
| 4 | The Cavendish London              | 2016-02-02   | United Kingdom        | Poor pillows for sleeping good for watching t...  |
| 5 | San Domenico House                | 2016-06-13   | United Kingdom        | The coffee and drinks are quite expensive but ... |
| 6 | Radisson Blu Edwardian Grafton    | 2016-05-18   | United Kingdom        | Pillows hard Evening staff on desk could not ...  |
| 7 | Holiday Inn London Kensington     | 2016-10-04   | United Kingdom        | Put in a disabled room when we weren t disabl...  |
| 8 | The Hoxton Holborn                | 2016-05-25   | United Kingdom        | The loud music in the bar area in the evening...  |
| 9 | Park Plaza County Hall London     | 2016-10-27   | United Kingdom        | Breakfast seating too cramped Not enough spac...  |

***

Since we've already parameterized the workflow, we can provide a different parameter value and see the new results immediately!

```python
# When calling .get() on an artifact, we can provide a map of parametres
# to see how different parametrization affects the execution of our workflow.
# Here, we change the default value ("United Kingdom") to a new value
# ("Australia").
filtered.get(parameters={"nationality": "Australia"}).head(10)
```

**Output**

|   | hotel\_name                      | review\_date | reviewer\_nationality | review                                            |
| - | -------------------------------- | ------------ | --------------------- | ------------------------------------------------- |
| 0 | H10 Itaca                        | 2017-08-03   | Australia             | Damaged bathroom shower screen sealant and ti...  |
| 1 | The Student Hotel Amsterdam City | 2016-07-31   | Australia             | No Negative The hotel had free gym table tenni... |
| 2 | Les Jardins Du Marais            | 2015-10-27   | Australia             | Bathroom is fine but could be improved with a...  |
| 3 | Napoleon Paris                   | 2015-10-07   | Australia             | NOTHING EVERYTHING                                |
| 4 | NH Milano Touring                | 2015-08-09   | Australia             | Check in and check out were extemely slow wit...  |
| 5 | Le Parisis Paris Tour Eiffel     | 2015-10-20   | Australia             | When we arrived we had to bring our own baggag... |

***

### Publishing a Parameterized Flow

When a flow is published on a schedule, the recurring run will continously execute with the default parameters. You may trigger additional runs with different parameters using the `client.trigger()` method. However, in order to change a parameter's value in perpetuity, you must rerun `create_param()` with an updated default value.

```python
flow = client.publish_flow("Parameter Example", artifacts=[filtered])
```

```python
# Wait until the flow has run at least once before triggering a new run.
from time import sleep

while len(flow.list_runs()) == 0:
    sleep(1)

client.trigger(flow.id(), parameters={"nationality": "Australia"})
```

***

#### SQL Query Parameters

SQL queries can also be parameterized. For queries, we'll use the double-bracket syntax to denote the presence of a parameter inline. As long as the name of the parameter matches a previously defined.

Here is the same flow as above, but as a parameterized SQL query instead.

NOTE: unlike parameter names for other operators, multi-part SQL parameter names cannot be separated with spaces. For example, "review-date" is allowed but "review date" is not.

```python
_ = client.create_param("nationality", default="United Kingdom")

table = db.sql("select * from hotel_reviews where reviewer_nationality=' {{nationality}} '")
table.get().head(10)
```

**Output**

|   | hotel\_name                       | review\_date | reviewer\_nationality | review                                            |
| - | --------------------------------- | ------------ | --------------------- | ------------------------------------------------- |
| 0 | De Vere Devonport House           | 2016-03-28   | United Kingdom        | No Negative The location and the hotel was ver... |
| 1 | Crowne Plaza London Docklands     | 2017-01-23   | United Kingdom        | Lighting in hotel room wasn t the best was ve...  |
| 2 | London Marriott Hotel Marble Arch | 2016-02-23   | United Kingdom        | No Negative The whole experience was excellent... |
| 3 | Grand Royale London Hyde Park     | 2017-01-04   | United Kingdom        | see above window looking out on the rough sid...  |
| 4 | The Cavendish London              | 2016-02-02   | United Kingdom        | Poor pillows for sleeping good for watching t...  |
| 5 | San Domenico House                | 2016-06-13   | United Kingdom        | The coffee and drinks are quite expensive but ... |
| 6 | Radisson Blu Edwardian Grafton    | 2016-05-18   | United Kingdom        | Pillows hard Evening staff on desk could not ...  |
| 7 | Holiday Inn London Kensington     | 2016-10-04   | United Kingdom        | Put in a disabled room when we weren t disabl...  |
| 8 | The Hoxton Holborn                | 2016-05-25   | United Kingdom        | The loud music in the bar area in the evening...  |
| 9 | Park Plaza County Hall London     | 2016-10-27   | United Kingdom        | Breakfast seating too cramped Not enough spac...  |

```python
table.get(parameters={"nationality": "Australia"})
```

**Output**

|   | hotel\_name                      | review\_date | reviewer\_nationality | review                                            |
| - | -------------------------------- | ------------ | --------------------- | ------------------------------------------------- |
| 0 | H10 Itaca                        | 2017-08-03   | Australia             | Damaged bathroom shower screen sealant and ti...  |
| 1 | The Student Hotel Amsterdam City | 2016-07-31   | Australia             | No Negative The hotel had free gym table tenni... |
| 2 | Les Jardins Du Marais            | 2015-10-27   | Australia             | Bathroom is fine but could be improved with a...  |
| 3 | Napoleon Paris                   | 2015-10-07   | Australia             | NOTHING EVERYTHING                                |
| 4 | NH Milano Touring                | 2015-08-09   | Australia             | Check in and check out were extemely slow wit...  |
| 5 | Le Parisis Paris Tour Eiffel     | 2015-10-20   | Australia             | When we arrived we had to bring our own baggag... |

***

#### Builtin SQL Parameters

There are also a number of builtin parameter tags that we support for you! See [our documentation](https://docs.aqueducthq.com/workflows/parameterizing-a-workflow) for a list of all of built-in parameters.

Below is an example of the fairly self-explanatory `today` parameter:

```python
# This will be empty because all records are historical.
reviews_after_today = db.sql("select * from hotel_reviews where review_date > {{ today }}")
reviews_after_today.get()
```

**Output**

```python
reviews_before_today = db.sql("select * from hotel_reviews where review_date < {{ today }}")
reviews_before_today.get()
```

**Output**

|     | hotel\_name                      | review\_date | reviewer\_nationality | review                                            |
| --- | -------------------------------- | ------------ | --------------------- | ------------------------------------------------- |
| 0   | H10 Itaca                        | 2017-08-03   | Australia             | Damaged bathroom shower screen sealant and ti...  |
| 1   | De Vere Devonport House          | 2016-03-28   | United Kingdom        | No Negative The location and the hotel was ver... |
| 2   | Ramada Plaza Milano              | 2016-05-15   | Kosovo                | No Negative Im a frequent traveler i visited m... |
| 3   | Aloft London Excel               | 2016-11-05   | Canada                | Only tepid water for morning shower They said ... |
| 4   | The Student Hotel Amsterdam City | 2016-07-31   | Australia             | No Negative The hotel had free gym table tenni... |
| ... | ...                              | ...          | ...                   | ...                                               |
| 95  | The Chesterfield Mayfair         | 2015-08-25   | Denmark               | Bad Reading light And light in bathNo Positive    |
| 96  | Hotel V Nesplein                 | 2015-08-27   | Turkey                | Nothing except the construction going on the s... |
| 97  | Le Parisis Paris Tour Eiffel     | 2015-10-20   | Australia             | When we arrived we had to bring our own baggag... |
| 98  | NH Amsterdam Museum Quarter      | 2016-01-26   | Belgium               | No stairs even to go the first floor Restaura...  |
| 99  | Barcel Raval                     | 2017-07-07   | United Kingdom        | Air conditioning a little zealous Nice atmosp...  |

100 rows × 4 columns

***

#### Implicitly Defined Parameters

If you pass a regular python object into an operator, we will automatically convert it into a parameter for you!

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

|   | hotel\_name                      | review\_date | reviewer\_nationality | review                                            |
| - | -------------------------------- | ------------ | --------------------- | ------------------------------------------------- |
| 0 | H10 Itaca                        | 2017-08-03   | Australia             | Damaged bathroom shower screen sealant and ti...  |
| 1 | The Student Hotel Amsterdam City | 2016-07-31   | Australia             | No Negative The hotel had free gym table tenni... |
| 2 | Les Jardins Du Marais            | 2015-10-27   | Australia             | Bathroom is fine but could be improved with a...  |
| 3 | Napoleon Paris                   | 2015-10-07   | Australia             | NOTHING EVERYTHING                                |
| 4 | NH Milano Touring                | 2015-08-09   | Australia             | Check in and check out were extemely slow wit...  |
| 5 | Le Parisis Paris Tour Eiffel     | 2015-10-20   | Australia             | When we arrived we had to bring our own baggag... |
