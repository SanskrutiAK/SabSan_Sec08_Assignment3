
# ASSIGNMENT 3 
### NoSQL MongoDB

## Abstract

The aim of this assignment is to convert the SQL Database into NoSQL Database .
The doman of the last assignment was supermarket dataset. The data set was taken from kaggle and had information about the customer transaction for products purchased from a particular Product Line.
Twitter was the social media scraped for tweets about the product line. This assignment converts the previous SQL Database into NoSQL database and stores everything in MongoDB Database.

## Data Source

##### The Link for the data source we have used for this assignment  and the previous assignment- 
#####  https://www.kaggle.com/aungpyaeap/supermarket-sales 

##### What is SQL Database?
SQL are relational databases. They use SQL queries to manipulate the data. Although SQL is a very powwerfull language, it is extremely restrictive. It requires predefined schemas to determine the structure of your data before you start working with it.
SQL requires all the data to follow the same structure for working with it.
Changing the structure of the schema can be disruptive for the whole system.
SQL is veryically scalable. SQL Databases are table based and Follow ACID properties, - Availability, consistency, Isolation and Durability.
Some popular SQL Databases are - PostgreSQL, MySQL, Oracle, Microsoft SQL Server

##### What is NoSQL Database?
NoSQL is primarily called the Non-Relational database or distributed databases. NoSQL has dynamic schemas for non structured data storage. In NoSQL data can be stored in various ways, document-oriented, graph based, column oriented or key value store.
NoSQL database is horizantally scalable , which means it manages traffic by data sharding or adding more servers to your NoSQL Database.
NoSQL databases follow the Brewers CAP theorem ( Consistency, Availability and Partition Tolerance)
Some popoular NoSQL Database are - Redis, RavenDB, Cassandra, MongoDB, CouchDB


## Data Auditing

Data Auditing is performed on the dataset from kaggle for checking for its validity ,completeness and consistency


```python
from pymongo import MongoClient
import sqlite3
import json
import os
from twitter import *
import tweepy 
import pandas as pd
import pprint
import datetime
```

### Reading the CSV File for Sales data


```python
supermarket_data = pd.read_csv("supermarket.csv")
supermarket_data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Invoice ID</th>
      <th>first_name</th>
      <th>last_name</th>
      <th>address</th>
      <th>phone1</th>
      <th>phone2</th>
      <th>email</th>
      <th>Branch</th>
      <th>City</th>
      <th>Customer type</th>
      <th>...</th>
      <th>Total</th>
      <th>Date</th>
      <th>Time</th>
      <th>Payment</th>
      <th>cogs</th>
      <th>gross margin percentage</th>
      <th>gross income</th>
      <th>Rating</th>
      <th>CITY</th>
      <th>STATE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>750-67-8428</td>
      <td>James</td>
      <td>Butt</td>
      <td>6649 N Blue Gum St</td>
      <td>504-621-8927</td>
      <td>504-845-1427</td>
      <td>jbutt@gmail.com</td>
      <td>A</td>
      <td>Yangon</td>
      <td>Member</td>
      <td>...</td>
      <td>548.9715</td>
      <td>1/5/19</td>
      <td>13:08</td>
      <td>Ewallet</td>
      <td>522.83</td>
      <td>4.761905</td>
      <td>26.1415</td>
      <td>9.1</td>
      <td>New Orleans</td>
      <td>LA</td>
    </tr>
    <tr>
      <th>1</th>
      <td>226-31-3081</td>
      <td>Josephine</td>
      <td>Darakjy</td>
      <td>4 B Blue Ridge Blvd</td>
      <td>810-292-9388</td>
      <td>810-374-9840</td>
      <td>josephine_darakjy@darakjy.org</td>
      <td>C</td>
      <td>Naypyitaw</td>
      <td>Normal</td>
      <td>...</td>
      <td>80.2200</td>
      <td>3/8/19</td>
      <td>10:29</td>
      <td>Cash</td>
      <td>76.40</td>
      <td>4.761905</td>
      <td>3.8200</td>
      <td>9.6</td>
      <td>Brighton</td>
      <td>MI</td>
    </tr>
    <tr>
      <th>2</th>
      <td>631-41-3108</td>
      <td>Art</td>
      <td>Venere</td>
      <td>8 W Cerritos Ave #54</td>
      <td>856-636-8749</td>
      <td>856-264-4130</td>
      <td>art@venere.org</td>
      <td>A</td>
      <td>Yangon</td>
      <td>Normal</td>
      <td>...</td>
      <td>340.5255</td>
      <td>3/3/19</td>
      <td>13:23</td>
      <td>Credit card</td>
      <td>324.31</td>
      <td>4.761905</td>
      <td>16.2155</td>
      <td>7.4</td>
      <td>Bridgeport</td>
      <td>NJ</td>
    </tr>
    <tr>
      <th>3</th>
      <td>123-19-1176</td>
      <td>Lenna</td>
      <td>Paprocki</td>
      <td>639 Main St</td>
      <td>907-385-4412</td>
      <td>907-921-2010</td>
      <td>lpaprocki@hotmail.com</td>
      <td>A</td>
      <td>Yangon</td>
      <td>Member</td>
      <td>...</td>
      <td>489.0480</td>
      <td>1/27/19</td>
      <td>20:33</td>
      <td>Ewallet</td>
      <td>465.76</td>
      <td>4.761905</td>
      <td>23.2880</td>
      <td>8.4</td>
      <td>Anchorage</td>
      <td>AK</td>
    </tr>
    <tr>
      <th>4</th>
      <td>373-73-7910</td>
      <td>Donette</td>
      <td>Foller</td>
      <td>34 Center St</td>
      <td>513-570-1893</td>
      <td>513-549-4561</td>
      <td>donette.foller@cox.net</td>
      <td>A</td>
      <td>Yangon</td>
      <td>Normal</td>
      <td>...</td>
      <td>634.3785</td>
      <td>2/8/19</td>
      <td>10:37</td>
      <td>Ewallet</td>
      <td>604.17</td>
      <td>4.761905</td>
      <td>30.2085</td>
      <td>5.3</td>
      <td>Hamilton</td>
      <td>OH</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 25 columns</p>
</div>



### Information about supermarket dataset


```python
supermarket_data.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1000 entries, 0 to 999
    Data columns (total 25 columns):
    Invoice ID                 1000 non-null object
    first_name                 1000 non-null object
    last_name                  1000 non-null object
    address                    1000 non-null object
    phone1                     1000 non-null object
    phone2                     1000 non-null object
    email                      1000 non-null object
    Branch                     1000 non-null object
    City                       1000 non-null object
    Customer type              1000 non-null object
    Gender                     1000 non-null object
    Product line               1000 non-null object
    Unit price                 1000 non-null float64
    Quantity                   1000 non-null int64
    Tax 5%                     1000 non-null float64
    Total                      1000 non-null float64
    Date                       1000 non-null object
    Time                       1000 non-null object
    Payment                    1000 non-null object
    cogs                       1000 non-null float64
    gross margin percentage    1000 non-null float64
    gross income               1000 non-null float64
    Rating                     1000 non-null float64
    CITY                       1000 non-null object
    STATE                      1000 non-null object
    dtypes: float64(7), int64(1), object(17)
    memory usage: 195.4+ KB
    

### Checking for missing values in the data set


```python
supermarket_data.isnull().any()
```




    Invoice ID                 False
    first_name                 False
    last_name                  False
    address                    False
    phone1                     False
    phone2                     False
    email                      False
    Branch                     False
    City                       False
    Customer type              False
    Gender                     False
    Product line               False
    Unit price                 False
    Quantity                   False
    Tax 5%                     False
    Total                      False
    Date                       False
    Time                       False
    Payment                    False
    cogs                       False
    gross margin percentage    False
    gross income               False
    Rating                     False
    CITY                       False
    STATE                      False
    dtype: bool



### Describing the Schema


```python
supermarket_data.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unit price</th>
      <th>Quantity</th>
      <th>Tax 5%</th>
      <th>Total</th>
      <th>cogs</th>
      <th>gross margin percentage</th>
      <th>gross income</th>
      <th>Rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1000.000000</td>
      <td>1000.000000</td>
      <td>1000.000000</td>
      <td>1000.000000</td>
      <td>1000.00000</td>
      <td>1.000000e+03</td>
      <td>1000.000000</td>
      <td>1000.00000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>55.672130</td>
      <td>5.510000</td>
      <td>15.379369</td>
      <td>322.966749</td>
      <td>307.58738</td>
      <td>4.761905e+00</td>
      <td>15.379369</td>
      <td>6.97270</td>
    </tr>
    <tr>
      <th>std</th>
      <td>26.494628</td>
      <td>2.923431</td>
      <td>11.708825</td>
      <td>245.885335</td>
      <td>234.17651</td>
      <td>6.220360e-14</td>
      <td>11.708825</td>
      <td>1.71858</td>
    </tr>
    <tr>
      <th>min</th>
      <td>10.080000</td>
      <td>1.000000</td>
      <td>0.508500</td>
      <td>10.678500</td>
      <td>10.17000</td>
      <td>4.761905e+00</td>
      <td>0.508500</td>
      <td>4.00000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>32.875000</td>
      <td>3.000000</td>
      <td>5.924875</td>
      <td>124.422375</td>
      <td>118.49750</td>
      <td>4.761905e+00</td>
      <td>5.924875</td>
      <td>5.50000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>55.230000</td>
      <td>5.000000</td>
      <td>12.088000</td>
      <td>253.848000</td>
      <td>241.76000</td>
      <td>4.761905e+00</td>
      <td>12.088000</td>
      <td>7.00000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>77.935000</td>
      <td>8.000000</td>
      <td>22.445250</td>
      <td>471.350250</td>
      <td>448.90500</td>
      <td>4.761905e+00</td>
      <td>22.445250</td>
      <td>8.50000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>99.960000</td>
      <td>10.000000</td>
      <td>49.650000</td>
      <td>1042.650000</td>
      <td>993.00000</td>
      <td>4.761905e+00</td>
      <td>49.650000</td>
      <td>10.00000</td>
    </tr>
  </tbody>
</table>
</div>



## Installing the Python Drivers


```python
pip install pymongo
```

    Requirement already satisfied: pymongo in c:\users\sanskruti\anaconda3\lib\site-packages (3.10.1)
    Note: you may need to restart the kernel to use updated packages.
    


```python
pip install twitter
```

    Requirement already satisfied: twitter in c:\users\sanskruti\anaconda3\lib\site-packages (1.18.0)
    Note: you may need to restart the kernel to use updated packages.
    


```python
pip install tweepy
```

    Requirement already satisfied: tweepy in c:\users\sanskruti\anaconda3\lib\site-packages (3.8.0)
    Requirement already satisfied: requests-oauthlib>=0.7.0 in c:\users\sanskruti\anaconda3\lib\site-packages (from tweepy) (1.3.0)
    Requirement already satisfied: PySocks>=1.5.7 in c:\users\sanskruti\anaconda3\lib\site-packages (from tweepy) (1.7.0)
    Requirement already satisfied: six>=1.10.0 in c:\users\sanskruti\anaconda3\lib\site-packages (from tweepy) (1.12.0)
    Requirement already satisfied: requests>=2.11.1 in c:\users\sanskruti\anaconda3\lib\site-packages (from tweepy) (2.22.0)
    Requirement already satisfied: oauthlib>=3.0.0 in c:\users\sanskruti\anaconda3\lib\site-packages (from requests-oauthlib>=0.7.0->tweepy) (3.1.0)
    Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in c:\users\sanskruti\anaconda3\lib\site-packages (from requests>=2.11.1->tweepy) (1.24.2)
    Requirement already satisfied: chardet<3.1.0,>=3.0.2 in c:\users\sanskruti\anaconda3\lib\site-packages (from requests>=2.11.1->tweepy) (3.0.4)
    Requirement already satisfied: certifi>=2017.4.17 in c:\users\sanskruti\anaconda3\lib\site-packages (from requests>=2.11.1->tweepy) (2019.6.16)
    Requirement already satisfied: idna<2.9,>=2.5 in c:\users\sanskruti\anaconda3\lib\site-packages (from requests>=2.11.1->tweepy) (2.8)
    Note: you may need to restart the kernel to use updated packages.
    

## Twitter API


```python
consumer_Key = "44qTzVYo9lBvDDrt7JY2jUc9V"
consumer_Secret_Key = "zINUwbwlRzSguW30I4cR28QH9j0248S5BmRsDVKXlAGgZoFk11"
access_token = "1246250838265335810-HLZHUQ2lfxIJ40OZnEGMBf1fOjy0p5"
access_token_secret = "4OerR3m6TQPleqXUtlvCzzqc1crMXWPKIzupyXRs3C4dn"

auth = tweepy.OAuthHandler(consumer_Key, consumer_Secret_Key)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth,wait_on_rate_limit=True)
```


```python
productLines=set(supermarket_data["Product line"])
```

## Scraping The tweets for ProductLines


```python
productLinesDictionary = {}
def productLineSearch(query):
    tweets = []
    count = 100
    
    searchedTweets = api.search(q=query,count=count,since="2020-04-08", until="2020-04-09")
    for tweet in searchedTweets:
        tags = []
        hashtags = (tweet._json["entities"]["hashtags"])
        screenname = tweet._json["user"]['screen_name']
        for tag in hashtags:
            tags.append(tag["text"])
        if len(tags) == 0:
            tags = ""
        tweet_obj ={
                "product_Line":query,
                "created_at":tweet.created_at,
                "screenname":screenname,
                "id":tweet.id,
                "text":tweet.text,
                "retweet":tweet.retweet_count,
                "favCount":tweet.favorite_count,
                "tags":tags
        }
        tweets.append(tweet_obj)
    productLinesDictionary[query] = tweets
```


```python
# Finds the tweets for all product lines
for product in productLines:
    productLineSearch(product)
```

## Printing all the tweets scraped for all the ProductLines 


```python
productLinesDictionary
```




    {'Food and beverages': [{'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 22, 40, 52),
       'screenname': 'VIRBOY21',
       'id': 1248018000122281984,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 49, 26),
       'screenname': 'RayDubicki',
       'id': 1248005053928992768,
       'text': 'TIL there is an extensive section of the Seattle City Code devoted to milk.\n\nAnd it’s probably against the law for… https://t.co/OCkLCFLinS',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 47, 25),
       'screenname': 'HofNorthwoods',
       'id': 1248004547814903810,
       'text': '@stealthygeek Golfing! Our local course where I have great memories with my dad, who was sadly lost to agent orange… https://t.co/Ia1NJP0EaA',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 40, 12),
       'screenname': 'iseeassociates',
       'id': 1248002728967589890,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 38, 7),
       'screenname': 'cummings_jim2',
       'id': 1248002206277615616,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 33, 20),
       'screenname': 'BobBaileyPC',
       'id': 1248001002797662208,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 33, 9),
       'screenname': 'Roxesays',
       'id': 1248000955821555716,
       'text': '#TheFive\n\nMaking prepared food and beverages tax free for at least a year or for the rest of this year is an EXCELL… https://t.co/ytQphJnvxJ',
       'retweet': 0,
       'favCount': 0,
       'tags': ['TheFive']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 30, 2),
       'screenname': 'Jorgeleon17',
       'id': 1248000171700453376,
       'text': 'Beer and soda isn’t a snack, they’re beverages. All these look disgusting btw. Lol All can go, I can bring my own f… https://t.co/kREpaMJ1O0',
       'retweet': 1,
       'favCount': 4,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 29, 23),
       'screenname': 'ChattyKatherine',
       'id': 1248000007543894021,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 26, 48),
       'screenname': 'jmachadorn',
       'id': 1247999356805890048,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 22, 5),
       'screenname': 'myra539',
       'id': 1247998170937937920,
       'text': '@realDonaldTrump Open us with social distancing!  Tax deductions for food and beverages for businesses again!!!!',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 21, 10, 17),
       'screenname': 'PeterJResistor',
       'id': 1247995203182645249,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 20, 49, 19),
       'screenname': 'TOO_LIVE_MILZ',
       'id': 1247989927729680388,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 20, 27, 20),
       'screenname': 'GahrSEEuh',
       'id': 1247984394838605825,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 20, 15),
       'screenname': 'MABeverage',
       'id': 1247981288616865797,
       'text': 'The #foodindustry provides shoppers with the food, beverages and basic necessities they need to navigate their live… https://t.co/SuIwygegNF',
       'retweet': 0,
       'favCount': 0,
       'tags': ['foodindustry']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 20, 4, 17),
       'screenname': 'DenaliMorrinson',
       'id': 1247978594758012935,
       'text': 'RT @ShannonFreshour: Ohio is making it easier for #SNAP families to get food and get an adequate amount of food through this crisis. \n\nAnd…',
       'retweet': 62,
       'favCount': 0,
       'tags': ['SNAP']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 19, 55, 40),
       'screenname': 'Chelle_Shock3D',
       'id': 1247976424939687936,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 19, 42),
       'screenname': 'gabrielssmith',
       'id': 1247972986868686848,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 19, 41, 3),
       'screenname': 'atamuotaf',
       'id': 1247972744953765892,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 19, 25, 41),
       'screenname': 'brdautremont',
       'id': 1247968879927685120,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 19, 21, 51),
       'screenname': 'JLaCocaina',
       'id': 1247967913472135168,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 19, 8, 35),
       'screenname': 'LizBraunSun',
       'id': 1247964573484843009,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 19, 1, 14),
       'screenname': 'csrlosf35',
       'id': 1247962725281562624,
       'text': 'IRVING, Texas — 7-Eleven Inc. has committed nearly $95 million to support its franchisees in this crucial time as t… https://t.co/wnDTEv0el1',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 45, 6),
       'screenname': 'BlueWaterHL',
       'id': 1247958666688462849,
       'text': "Port Huron's local gem, The Raven Cafe, is still serving up their great food and beverages during this statewide sh… https://t.co/2yixw1gOPV",
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 42, 2),
       'screenname': 'charlessss5',
       'id': 1247957891912552450,
       'text': "@TheOnlyOlogi Food and beverages for my two sons, it's crazy out here broke die here.😢",
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 37, 19),
       'screenname': 'oliviadope',
       'id': 1247956707877289995,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 32),
       'screenname': 'CityBeatCincy',
       'id': 1247955368082198528,
       'text': 'Ohio Gov. Mike DeWine announced that businesses with liquor permits can now sell and deliver two packaged drinks pe… https://t.co/EnNeQjbNjp',
       'retweet': 0,
       'favCount': 8,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 26, 47),
       'screenname': 'ville_siesta',
       'id': 1247954057072787456,
       'text': 'RT @JeetoCheesus: @AITA_reddit “You can live with me. But you are not to enjoy yourself AT ALL for any reason. Do not eat food. Do not cons…',
       'retweet': 6,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 25, 2),
       'screenname': 'JoshSSchutt',
       'id': 1247953616314306561,
       'text': '@ClueHeywood That’s why I stick to @TGIFridays, only the finest food and beverages',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 22, 2),
       'screenname': 'DapaDon',
       'id': 1247952862782554112,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 18, 49),
       'screenname': 'Yanaa_eff',
       'id': 1247952050543112199,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 14, 35),
       'screenname': 'bashdash4',
       'id': 1247950984493428737,
       'text': '15 days until I forego all food and beverages in the name of the Lord',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 11, 49),
       'screenname': 'xShaunieex',
       'id': 1247950291242278913,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 10, 52),
       'screenname': 'joebagod',
       'id': 1247950050694762503,
       'text': '@SpaceCptZemo You’d fun to hang with and enjoy some beverages, food and games. 🤷\u200d♂️',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 18, 10, 1),
       'screenname': 'icicleBsafe',
       'id': 1247949835694555136,
       'text': 'Funding has reopened for the #BritishColumbia Post-Farm Food Safety Program, offering up to $20K to qualifying food… https://t.co/6mmSoWGSoR',
       'retweet': 0,
       'favCount': 1,
       'tags': ['BritishColumbia']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 17, 43, 1),
       'screenname': 'eilispurcell',
       'id': 1247943041953992708,
       'text': '@anaceant @AlanHGoff @jclancy1995 @philipoconnor Give over will ya. They are not going to stay in a caravan for 2 w… https://t.co/TQuD5ntSnr',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 17, 41, 2),
       'screenname': 'h0ney_nee',
       'id': 1247942544572416002,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 17, 26, 8),
       'screenname': 'CoastPackingCo',
       'id': 1247938792872280064,
       'text': 'Consumers may behave more conservatively and cautiously in the months to come, relying on food and beverages that p… https://t.co/BJjZ8ePrM3',
       'retweet': 1,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 17, 20, 45),
       'screenname': 'StaccnBenjis',
       'id': 1247937437969584128,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 17, 14, 23),
       'screenname': 'alfredogiuria',
       'id': 1247935837775552520,
       'text': "Mamma mía, que grande belleza! Italia por Sophia Loren. Barilla pays tribute to 'resilient Italy' in ad narrated by… https://t.co/Ds0Xr7fMaW",
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 17, 4, 43),
       'screenname': 'albertllussa',
       'id': 1247933401870598156,
       'text': '(b) go to an essential retail outlet for the purpose of obtaining items (food, beverages, fuel, medicines, medical… https://t.co/6tRnHAONgY',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 17, 1, 2),
       'screenname': 'centeredgesoft',
       'id': 1247932474845528065,
       'text': 'Industry experts believe your #FEC should stand out through its #team, facility, food/beverages and dedication to c… https://t.co/yrv8sGUz4F',
       'retweet': 0,
       'favCount': 0,
       'tags': ['FEC', 'team']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 50, 56),
       'screenname': 'NancyAl50283383',
       'id': 1247929935601123330,
       'text': 'https://t.co/fZRe2zv7J6\nJoin us online at food packaging 2020 Webinar and gain the latest insights on Food and Beve… https://t.co/OtDHirTCaH',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 45, 55),
       'screenname': 'EuroFoodConf',
       'id': 1247928673555992577,
       'text': 'https://t.co/LRrRdLXkOL\nJoin us online at Euro food 2020 Webinar and gain the latest insights on Food and Beverages… https://t.co/jodlfb0g0n',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 41, 29),
       'screenname': 'cuddlebugg10',
       'id': 1247927557544120322,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 41, 14),
       'screenname': 'TheYazzyB_Show',
       'id': 1247927493492965376,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 39, 54),
       'screenname': 'kwillism',
       'id': 1247927156048609280,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 37, 35),
       'screenname': 'BIGBADREL',
       'id': 1247926576613863424,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 36, 7),
       'screenname': 'riteaparty',
       'id': 1247926206542098432,
       'text': 'RT @RICenterFreedom: Already in Rhode Island, one of the Center’s early recommendations has been enacted: To allow alcoholic beverages to l…',
       'retweet': 2,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 35, 39),
       'screenname': 'shaROCK_',
       'id': 1247926089130946566,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 31, 29),
       'screenname': 'mecredes05',
       'id': 1247925040454066176,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 31, 26),
       'screenname': 'JenneBroughton',
       'id': 1247925027464339457,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 29, 56),
       'screenname': '__NahImGood',
       'id': 1247924650954424327,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 29, 1),
       'screenname': 'sincerelyskye__',
       'id': 1247924418048929792,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 27, 25),
       'screenname': 'OJ_noSimpson',
       'id': 1247924016557539329,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 19, 11),
       'screenname': 'yourmomie',
       'id': 1247921943996592129,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 18, 17),
       'screenname': 'Bselected',
       'id': 1247921719710449667,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 11, 28),
       'screenname': 'YungLost_Rebel',
       'id': 1247920002545704963,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 11, 28),
       'screenname': 'Blue_Nox',
       'id': 1247920001996271617,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 7, 4),
       'screenname': 'YoungstownCP',
       'id': 1247918895136231430,
       'text': 'You wanted it govmikedewine made it possible! We are now able to do with takeout food purchase 2 takeout beverages… https://t.co/zVJ6Q2MQQ7',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 4, 36),
       'screenname': 'ShowTimeRick',
       'id': 1247918275830460416,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 1, 19),
       'screenname': 'Ibnyei',
       'id': 1247917449351245824,
       'text': 'Exempted from the Lockdown in #Liberia are those involved with the production, distribution, and marketing of food… https://t.co/fyPpZolIL6',
       'retweet': 0,
       'favCount': 0,
       'tags': ['Liberia']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 16, 0, 14),
       'screenname': 'MiissV_',
       'id': 1247917175169482754,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 59, 56),
       'screenname': 'onlyfordisplay_',
       'id': 1247917098539548673,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 59, 39),
       'screenname': 'JoellasWorld',
       'id': 1247917028624592897,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 49, 53),
       'screenname': 'NasBlixky63',
       'id': 1247914572759130112,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 45, 17),
       'screenname': 'RaeDiamond',
       'id': 1247913411641585664,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 44, 56),
       'screenname': 'cheesewizbandit',
       'id': 1247913326236930049,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 42, 2),
       'screenname': 'LikkleBITCH_',
       'id': 1247912593987158017,
       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me know…',
       'retweet': 29,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 41, 21),
       'screenname': 'TOO_LIVE_MILZ',
       'id': 1247912424818323457,
       'text': 'If you or someone you know doesn’t have access to FOOD &amp; BEVERAGES during this pandemic and are in BROOKLYN let me… https://t.co/0kxd8afxQ4',
       'retweet': 29,
       'favCount': 3,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 39, 48),
       'screenname': 'hotairtools',
       'id': 1247912034760626179,
       'text': 'Boom! Heat shrink plastic sleeves, film and tubes. Used in the packaging industry for food, beverages and much more… https://t.co/77pzQ3YvKO',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 24, 42),
       'screenname': 'MikeLow55504451',
       'id': 1247908232225587204,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 17, 28),
       'screenname': 'elau122',
       'id': 1247906414670106624,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 17, 10),
       'screenname': 'WesNotWestffs',
       'id': 1247906337226448896,
       'text': 'Dear @LovesTravelStop when you make your beverages "full service" and cordon off the area you have created a food p… https://t.co/qV163knixE',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 7, 36),
       'screenname': '7barbie7c',
       'id': 1247903931176333320,
       'text': 'RT @Shell_Canada: Our priority is the health &amp; safety of everyone affected by this pandemic, but we want to do more. In the coming weeks, a…',
       'retweet': 678,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 7, 26),
       'screenname': 'rokktober',
       'id': 1247903888734208002,
       'text': '@PixelBunny821 Hello!\n\nMy name is October and I’ve been a pixel artist for the past two years!\n\nI like to draw beve… https://t.co/xmR8COVTpX',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 15, 2, 16),
       'screenname': 'WABevAssn',
       'id': 1247902589699547138,
       'text': 'The #foodindustry provides shoppers with the food, beverages and basic necessities they need to navigate their live… https://t.co/EVAUWvuo8N',
       'retweet': 0,
       'favCount': 0,
       'tags': ['foodindustry']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 14, 50, 44),
       'screenname': 'BalinwillinFarm',
       'id': 1247899687429869569,
       'text': 'RT @8degreesbrewing: @NeighbourFoodIE #Doneraile is open until this evening - order online for #contactfree delivery on Friday. Lots of #lo…',
       'retweet': 1,
       'favCount': 0,
       'tags': ['Doneraile', 'contactfree']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 14, 47, 19),
       'screenname': 'sirtaj03',
       'id': 1247898825131347970,
       'text': 'Iont understand how these rich mfs complaining about being “quarantined.” They literally have everything. House wit… https://t.co/HuVhnWmOtw',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 14, 45, 41),
       'screenname': '8degreesbrewing',
       'id': 1247898414232166404,
       'text': '@NeighbourFoodIE #Doneraile is open until this evening - order online for #contactfree delivery on Friday. Lots of… https://t.co/acLGS6lEsP',
       'retweet': 1,
       'favCount': 2,
       'tags': ['Doneraile', 'contactfree']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 14, 34, 56),
       'screenname': 'BMmamarope',
       'id': 1247895710885789697,
       'text': 'Common Industries which uses #Biotechnology are: medical industry, food and beverages, agriculture, nutrition.',
       'retweet': 0,
       'favCount': 0,
       'tags': ['Biotechnology']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 13, 39, 4),
       'screenname': 'ElodieBrassard',
       'id': 1247881648382894080,
       'text': '@Ferocious__B @bigmeepemoppe @CHamiltoChem Also drinking basic beverages alters you digestion because it mixes with… https://t.co/xDLqaXU2IH',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 13, 37, 6),
       'screenname': 'endjonesactPR',
       'id': 1247881154050568193,
       'text': '"The Jones Act adds an estimated\xa0$300\xa0to the annual cost of food and beverages a typical household in Puerto Rico p… https://t.co/EPHK2sYpEX',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 13, 19, 7),
       'screenname': 'MindfuLY',
       'id': 1247876630447603714,
       'text': 'RT @NatLauter: ONroute service centres give free coffee to truck drivers on Wednesday\n\nProviding 24-7 access to fuel, washrooms, food/bever…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 13, 15, 41),
       'screenname': 'sporthiatus',
       'id': 1247875764625772544,
       'text': 'Food and beverages set up at my desk.  Ready to roll, @sbjsbd. https://t.co/Q8eQUiI6w7',
       'retweet': 0,
       'favCount': 3,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 13, 14, 10),
       'screenname': 'SunshineLK10',
       'id': 1247875381572624390,
       'text': "RT @c0lettea: @reneenilseb @SunshineLK10 Illicit fentanyl comes in liquid, powder and counterfeit pill forms. It's not a patch those are pr…",
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 13, 10, 20),
       'screenname': 'c0lettea',
       'id': 1247874419697074176,
       'text': "@reneenilseb @SunshineLK10 Illicit fentanyl comes in liquid, powder and counterfeit pill forms. It's not a patch th… https://t.co/ySQDENbBJe",
       'retweet': 1,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 13, 7, 26),
       'screenname': 'TrumpWarriorJB',
       'id': 1247873690928365568,
       'text': '@LILIFIFIELD @OANN Jamie, Inside sales rep for company with an electrification and automation technology manufactur… https://t.co/5qrUik4gKo',
       'retweet': 0,
       'favCount': 4,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 13, 4, 23),
       'screenname': 'broadstreetlag',
       'id': 1247872919818076160,
       'text': '$cad a #food #beverages and #tobacco #stock listed in #Nigeria has shown its #long play from its N5.15 low closing… https://t.co/yxOiFZUrWo',
       'retweet': 0,
       'favCount': 0,
       'tags': ['food', 'beverages', 'tobacco', 'stock', 'Nigeria', 'long']},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 13, 3, 48),
       'screenname': 'QizBalqis',
       'id': 1247872774678253568,
       'text': '@thekhayalan15 Caregory tu dia automatically akan separate to their own group . Macam tesco starbucks semua under f… https://t.co/rNG2f6P5iG',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 13, 0, 59),
       'screenname': 'hamizan_huge',
       'id': 1247872066977579010,
       'text': '@firzanahbalqis @syhmisamsol Food and beverages are the best bae EvER',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 12, 59, 15),
       'screenname': 'DubaiChronicle',
       'id': 1247871630120009728,
       'text': '@IAmMrBrightside Even in the supermarkets food and beverages are more expensive these days...',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 12, 53, 45),
       'screenname': 'NatLauter',
       'id': 1247870245630550016,
       'text': 'ONroute service centres give free coffee to truck drivers on Wednesday\n\nProviding 24-7 access to fuel, washrooms, f… https://t.co/2tPhqA8T8b',
       'retweet': 1,
       'favCount': 5,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 12, 47, 24),
       'screenname': 'aspire_speaks',
       'id': 1247868648737386498,
       'text': '@ilz_zli Having said this, best source of vitamin C are: \n\n1) Citrus Fruits (Oranges, Grapefruit, Lemon &amp; Lime) and… https://t.co/xm5qy7x4Kg',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 12, 1, 49),
       'screenname': 'MediocrisRegina',
       'id': 1247857177110269952,
       'text': '@LeprechaunCurse As their food and beverages arrive she smiles over at her companion. With Seamus back in her life… https://t.co/iGZ8bG8FQV',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 11, 37),
       'screenname': 'infointuitive',
       'id': 1247850932056064000,
       'text': 'RT @environmentza: The distribution of the food parcels for waste pickers is a partnership between government and Coca-Cola Beverages South…',
       'retweet': 2,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 11, 1, 37),
       'screenname': 'PopRoxas',
       'id': 1247842028324630529,
       'text': '@Fact 1) Food Dyes\n2) Microwave Popcorn\n3) Red Meat\n4) Salt-Cured and Pickled Foods\n5) Refined Carbohydrates and Su… https://t.co/ujay1QiSNQ',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 11, 0, 39),
       'screenname': '_foodmicrobio',
       'id': 1247841781468680192,
       'text': 'RT @FoodIndAsia: FIA and the ASEAN Food and Beverage Alliance (AFBA) are jointly calling upon governments across the region to ensure the u…',
       'retweet': 6,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 10, 57, 39),
       'screenname': 'Sanador',
       'id': 1247841028096372737,
       'text': 'RT @OwlchemyLabs: Sometimes all you need is a staycation. Video games, junk food, and an endless supply of caffeinated sugary beverages. Ga…',
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Food and beverages',
       'created_at': datetime.datetime(2020, 4, 8, 10, 40, 42),
       'screenname': 'institutiGAP',
       'id': 1247836761310539781,
       'text': 'Kosovo’s exports to Albania include manufacturing products from industries such as: alcoholic and non-alcoholic bev… https://t.co/ludTVvCrTt',
       'retweet': 0,
       'favCount': 0,
       'tags': ''}],
     'Electronic accessories': [{'product_Line': 'Electronic accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 28, 41),
       'screenname': 'TicAccessories',
       'id': 1248014932177641473,
       'text': 'TIC Accessories LTD.\n167th Street, Jamaica, New York 11432, USA.\nTIC Accessories LTD is a well-known Electronic pro… https://t.co/DINrF5U25t',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Electronic accessories',
       'created_at': datetime.datetime(2020, 4, 8, 19, 26, 28),
       'screenname': 'GhanaSocialU',
       'id': 1247969073922748420,
       'text': 'RT @IzaicKumy: Get all ur electronic devices :\niPhones 📱 , MacBook 💻, Ps4 , Samsung phones 📲, laptop 💻 at  affordable prices . \nAccessories…',
       'retweet': 92,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Electronic accessories',
       'created_at': datetime.datetime(2020, 4, 8, 19, 2, 16),
       'screenname': 'MightyInkMatt',
       'id': 1247962986922246147,
       'text': "I really wish I could validate this purchase right now, but here's that fancy Black Panther helmet for $54.00 (free… https://t.co/g3y9a6pf3G",
       'retweet': 0,
       'favCount': 4,
       'tags': ''},
      {'product_Line': 'Electronic accessories',
       'created_at': datetime.datetime(2020, 4, 8, 18, 38, 4),
       'screenname': 'digitobacco',
       'id': 1247956894209228801,
       'text': 'RT @perfect_fold: Idea 💡 201:\n\nPLEASE PASS TO ALL VAPE MAKERS\nVAPE ACCESSORIES\n\n©️\nVape Access;\nVPN on electronic version.\nDigital clock.\nP…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Electronic accessories',
       'created_at': datetime.datetime(2020, 4, 8, 18, 35, 17),
       'screenname': 'perfect_fold',
       'id': 1247956197019443201,
       'text': 'Idea 💡 201:\n\nPLEASE PASS TO ALL VAPE MAKERS\nVAPE ACCESSORIES\n\n©️\nVape Access;\nVPN on electronic version.\nDigital cl… https://t.co/2Qlwnjb1pU',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Electronic accessories',
       'created_at': datetime.datetime(2020, 4, 8, 11, 37, 42),
       'screenname': 'WheelersGifts',
       'id': 1247851106232864768,
       'text': 'MB 01 - Keep your eyes peeled because we will soon have them in stock 👀🎧🎶 https://t.co/uC6Ickik4V\n\n-\n\n#montblanc… https://t.co/qinAyGtN0E',
       'retweet': 0,
       'favCount': 0,
       'tags': ['montblanc']},
      {'product_Line': 'Electronic accessories',
       'created_at': datetime.datetime(2020, 4, 8, 10, 46, 31),
       'screenname': 'OShop545',
       'id': 1247838227790286849,
       'text': '#electronic_accessories #phone_case #phone_glass #phone_cables #phone_chargersSpring USB Cable for iPhone Charger F… https://t.co/37SYV4uGnR',
       'retweet': 0,
       'favCount': 0,
       'tags': ['electronic_accessories',
        'phone_case',
        'phone_glass',
        'phone_cables',
        'phone_chargersSpring']},
      {'product_Line': 'Electronic accessories',
       'created_at': datetime.datetime(2020, 4, 8, 3, 33, 42),
       'screenname': 'StoreZentrum',
       'id': 1247729303476994048,
       'text': '#hashtag3 Spacious Bag for MacBook Laptops and Electronic Accessories https://t.co/Ff8VR9QJ7d https://t.co/6zyOK3aZA8',
       'retweet': 0,
       'favCount': 0,
       'tags': ['hashtag3']},
      {'product_Line': 'Electronic accessories',
       'created_at': datetime.datetime(2020, 4, 8, 2, 27, 35),
       'screenname': 'AmroleL',
       'id': 1247712666422042624,
       'text': 'https://t.co/q4qQgEAvCH\nKEYCEO is the professional manufacturer for researching and developing #Gaming peripherals… https://t.co/d0BdImYSVb',
       'retweet': 0,
       'favCount': 0,
       'tags': ['Gaming']},
      {'product_Line': 'Electronic accessories',
       'created_at': datetime.datetime(2020, 4, 8, 2, 10, 24),
       'screenname': 'mkrishnasundar',
       'id': 1247708342039883776,
       'text': '@amazonIN Can you pls enable laptop accessories for purchase/delivery? During lockdown to continue WFH some electro… https://t.co/BRrfEULtKP',
       'retweet': 0,
       'favCount': 0,
       'tags': ''}],
     'Sports and travel': [{'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 22, 48, 27),
       'screenname': 'Lynda63986855',
       'id': 1248019906357940225,
       'text': 'RT @Lynda63986855: @vicksiern @jan_aurora @Shaun_Girk @MarlaineDettlo1 @ChrisPBaconLT @ScottRickhoff @Quin4Trump @davidf4444 @TheWickerhead…',
       'retweet': 47,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 22, 47),
       'screenname': 'AustinEilts',
       'id': 1248019543877795840,
       'text': 'RT @WIPEvenings: Name a moment/event (sports or non sports) you weren’t around for that you’d love to travel back and experience. \n\nhttps:/…',
       'retweet': 2,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 22, 40, 45),
       'screenname': 'rheaecho',
       'id': 1248017967528345601,
       'text': 'RT @KJRMIN4J: which activity would u do with me?\n\n🎉-party \n💃🏼-take a dance class\n💗-cuddle and watch a movie\n🎢-go to an amusement park/zoo\n🏝…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 22, 37, 33),
       'screenname': 'KJRMIN4J',
       'id': 1248017162511368192,
       'text': 'which activity would u do with me?\n\n🎉-party \n💃🏼-take a dance class\n💗-cuddle and watch a movie\n🎢-go to an amusement… https://t.co/lLCk9jwO0q',
       'retweet': 1,
       'favCount': 4,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 22, 33, 17),
       'screenname': 'jeep_sifu',
       'id': 1248016088597553158,
       'text': 'RT @Lynda63986855: @vicksiern @jan_aurora @Shaun_Girk @MarlaineDettlo1 @ChrisPBaconLT @ScottRickhoff @Quin4Trump @davidf4444 @TheWickerhead…',
       'retweet': 47,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 22, 13, 29),
       'screenname': 'ChrisPBaconLT',
       'id': 1248011105420455936,
       'text': 'RT @Lynda63986855: @vicksiern @jan_aurora @Shaun_Girk @MarlaineDettlo1 @ChrisPBaconLT @ScottRickhoff @Quin4Trump @davidf4444 @TheWickerhead…',
       'retweet': 47,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 22, 8, 2),
       'screenname': 'yourboiijames',
       'id': 1248009735397244929,
       'text': 'which activity would u do with me?\n\n🎉-party \n💃🏼-take a dance class\n💗-cuddle and watch a movie\n🎢-go to an amusement… https://t.co/gtkQhucARD',
       'retweet': 0,
       'favCount': 2,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 22, 6, 34),
       'screenname': 'pharsyy',
       'id': 1248009367950995459,
       'text': 'which activity would u do with me?\n\n🎉-party \n💃🏼-take a dance class\n💗-cuddle and watch a movie\n🎢-go to an amusement… https://t.co/zAOfFtrVZW',
       'retweet': 0,
       'favCount': 9,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 22, 5, 34),
       'screenname': 'CoolHandLuke124',
       'id': 1248009114820595713,
       'text': '@KathyLovesPizza @nytimes Not arguing if they should receive help but you could say the same thing about all  enter… https://t.co/DA9QAcI6Iy',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 22, 0, 32),
       'screenname': 'Sewn_apart',
       'id': 1248007846052032512,
       'text': '@bringonthebeer To be fair, i think international travel and sports stadia will later. I can see pubs being earlier… https://t.co/2eD32u5EeR',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 21, 51, 13),
       'screenname': 'JoeGiglioSports',
       'id': 1248005501708668928,
       'text': 'RT @WIPEvenings: Name a moment/event (sports or non sports) you weren’t around for that you’d love to travel back and experience. \n\nhttps:/…',
       'retweet': 2,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 21, 50, 38),
       'screenname': 'WIPEvenings',
       'id': 1248005356690653185,
       'text': 'Name a moment/event (sports or non sports) you weren’t around for that you’d love to travel back and experience. \n\nhttps://t.co/0N55IWC6bz',
       'retweet': 2,
       'favCount': 8,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 21, 47, 11),
       'screenname': 'marymamie5',
       'id': 1248004488650084354,
       'text': 'RT @tax: Travel and tourism—which funnels $83 billion in tax revenue annually to state and local governments—is in a freefall triggered by…',
       'retweet': 3,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 21, 46, 3),
       'screenname': 'ruedelachaise',
       'id': 1248004205010247685,
       'text': 'RT @MylesSuer: .@BillGates levels with Americans on @TheDailyShow re what is in front of us. It will take a month to start to bend the curv…',
       'retweet': 13,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 21, 39, 23),
       'screenname': 'rebelsart',
       'id': 1248002525539610624,
       'text': 'RT @rebelsart: @MickaelleMichel @SportsHochi Thank you for being a wonderful ambassador for horse racing and sports! Have a safe travel bac…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 21, 23, 19),
       'screenname': 'Travel_Iowa',
       'id': 1247998483510026241,
       'text': 'Missing sports? You’re in luck. Tonight, you can catch virtual racing live from the sprint car capital of the world… https://t.co/69vHKMXjGy',
       'retweet': 0,
       'favCount': 2,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 21, 3, 51),
       'screenname': 'step_up_sports',
       'id': 1247993583916986369,
       'text': 'RT @Bakler1: Is now the time to promote all National Prem sides to EFL to make a EFL2 north and EFL2 south, with national league just a nor…',
       'retweet': 116,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 47, 43),
       'screenname': 't_tomochika2014',
       'id': 1247989524103188480,
       'text': 'RT @rebelsart: @MickaelleMichel @SportsHochi Thank you for being a wonderful ambassador for horse racing and sports! Have a safe travel bac…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 39, 37),
       'screenname': 'gdelgado210',
       'id': 1247987483394158592,
       'text': 'RT @prettyybirrd: @davidrocknyc I own my own business in the sports and travel industry. What I need are sponsors who are who are looking f…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 33, 21),
       'screenname': 'Sports_Schlub',
       'id': 1247985909464805383,
       'text': '@CNN Government keeps closing local parks and limiting travel in order to trap brown people in their cramped neighb… https://t.co/BAh1jWYWYa',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 30, 49),
       'screenname': 'luke_h_wright',
       'id': 1247985272093954048,
       'text': 'So the crux of banning/avoiding backcountry travel and sports has been 1) not spreading COVID to new communities an… https://t.co/MG8CvDy7Ao',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 30, 7),
       'screenname': 'mynameistiwani',
       'id': 1247985093685252098,
       'text': 'Sky Sports GOTS making me want to time travel back to 1993, Le Tiss, Shearer and Mark Hughes were on crud',
       'retweet': 0,
       'favCount': 2,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 29, 49),
       'screenname': 'ChabeauMichel',
       'id': 1247985019152408576,
       'text': 'RT @rebelsart: @MickaelleMichel @SportsHochi Thank you for being a wonderful ambassador for horse racing and sports! Have a safe travel bac…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 27, 14),
       'screenname': 'JamesKratch',
       'id': 1247984368531976192,
       'text': "It's interesting how many of the stories about college sports and coronavirus economics reference travel costs as a… https://t.co/S3zgtoc1Al",
       'retweet': 0,
       'favCount': 4,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 26, 51),
       'screenname': 'rabcyr',
       'id': 1247984272482414605,
       'text': 'closing the borders (except essential goods) and stopping all air travel (and sports &amp; concerts) back in january, e… https://t.co/nUuug2X7jJ',
       'retweet': 0,
       'favCount': 2,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 22, 5),
       'screenname': 'KeisersozayHTTC',
       'id': 1247983073364508672,
       'text': '@ClydeSSB Even when this pandemic is under contoll sanctions will remain for a extended period of time  to ensure t… https://t.co/4B1s5IZi2y',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 20, 17),
       'screenname': 'DirectTrip',
       'id': 1247982621260480514,
       'text': 'Travel and hospitality hardest hit as COVID-19 continues to ravage UK job market - Yahoo Sports https://t.co/6HIaYPDfye #UK #Travel',
       'retweet': 0,
       'favCount': 0,
       'tags': ['UK', 'Travel']},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 17, 19),
       'screenname': 'Travel_MSW',
       'id': 1247981874007248898,
       'text': 'RT @ASWISports: The term "identity foreclosure" has become all too relevant during this pandemic and especially in sports. https://t.co/BqH…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 20, 3, 50),
       'screenname': 'jawhoriskey',
       'id': 1247978481578913803,
       'text': 'RT @JzwTravel: JZW Travel not only books #vacations but we also book #business travel as well as #sports, #religious and #club #travels. Co…',
       'retweet': 1,
       'favCount': 0,
       'tags': ['vacations',
        'business',
        'sports',
        'religious',
        'club',
        'travels']},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 19, 59, 1),
       'screenname': 'JzwTravel',
       'id': 1247977265557901314,
       'text': 'JZW Travel not only books #vacations but we also book #business travel as well as #sports, #religious and #club… https://t.co/rLhCABIjUY',
       'retweet': 1,
       'favCount': 0,
       'tags': ['vacations', 'business', 'sports', 'religious', 'club']},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 19, 34, 32),
       'screenname': 'dxgby',
       'id': 1247971106302107653,
       'text': '@AcnhSally " That\'s why I came here.. "\n\n    He pulls out a travel bag, taking out tight black yoga pants and a spo… https://t.co/hvlgQuBMj5',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 18, 37, 45),
       'screenname': 'atic_sports',
       'id': 1247956815582760960,
       'text': '@stewart7pete @CFBKnights @bignizo @Brett_McMurphy @Stadium @CFBPlayoff Nobody outside of Memphis wanted to see Mem… https://t.co/fpu6giWNlH',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 18, 34, 47),
       'screenname': 'sawarnspeaks',
       'id': 1247956067977285633,
       'text': "At this point, we can make changes that wouldn't require any extra travel, unless there is some important lab/sport… https://t.co/AuPA9j4xKE",
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 18, 27, 45),
       'screenname': 'rene4the5',
       'id': 1247954299348488193,
       'text': 'Just like in the world of sports, stupidity, lying and cinicism can always breake records. Freeze Travel to and fro… https://t.co/Xsd4rNs2Fg',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 18, 21, 36),
       'screenname': 'SugdenSteve',
       'id': 1247952751352561665,
       'text': '@DentonClark17 Well to be fair.....sports does make economy go....travel tickets gear food usually stop to eat and… https://t.co/ytOhPCB8IN',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 18, 8, 53),
       'screenname': 'beatsatmonster',
       'id': 1247949553447231488,
       'text': 'DeftGet First Aid Kit – 163 Piece Waterproof Portable Essential Injuries &amp; Red Cross Medical Emergency Equipment Ki… https://t.co/yoWJ4n41n3',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 57, 29),
       'screenname': 'LeeAbbamonte',
       'id': 1247946681158635520,
       'text': 'RT @AmberTheoharis: Love sports and travel? Join me and travel expert @LeeAbbamonte today on IG Live T 1pm PT/ 4pm ET. He’s been to everyon…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 51, 41),
       'screenname': 'ThisIsQuasimodo',
       'id': 1247945224565010432,
       'text': '@NathiMthethwaSA Honourable Minister, please can you tell me why sports administrators are allowed to lead organiza… https://t.co/lrhu3HRtyS',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 29, 44),
       'screenname': 'Billsmafia178',
       'id': 1247939699211026432,
       'text': 'RT @AmberTheoharis: Love sports and travel? Join me and travel expert @LeeAbbamonte today on IG Live T 1pm PT/ 4pm ET. He’s been to everyon…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 28, 5),
       'screenname': 'Michael_Fabiano',
       'id': 1247939283857444864,
       'text': 'RT @AmberTheoharis: Love sports and travel? Join me and travel expert @LeeAbbamonte today on IG Live T 1pm PT/ 4pm ET. He’s been to everyon…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 26, 5),
       'screenname': 'gordwright',
       'id': 1247938780968800257,
       'text': '@llikemoyd Donate those “deposit” bottles guys. Sports teams will need to fundraise soon (and by the sounds of it t… https://t.co/VTHcFRmhKC',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 22, 1),
       'screenname': 'TheShergar',
       'id': 1247937756728262661,
       'text': 'RT @FCurran17: Despite all the doom &amp; gloom and its gonna get worse in new week or two deaths wise I fully believe by end of April/early ma…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 17, 36),
       'screenname': 'FCurran17',
       'id': 1247936646072467467,
       'text': 'Despite all the doom &amp; gloom and its gonna get worse in new week or two deaths wise I fully believe by end of April… https://t.co/NCv7UBJNAl',
       'retweet': 1,
       'favCount': 4,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 17, 21),
       'screenname': 'ManSaffer',
       'id': 1247936584109789184,
       'text': '@MollyJongFast I’m hoping for movie theaters, restaurants, sports stadiums, pubs, beaches, and air travel to name a few',
       'retweet': 0,
       'favCount': 2,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 16, 36),
       'screenname': 'TeamSC11',
       'id': 1247936395794173952,
       'text': '@3ManFront I travel a good bit and every sports bar with 30 to 80 TVs have every most TVs on the games and at the s… https://t.co/FokHVszSV5',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 13, 39),
       'screenname': 'js92879',
       'id': 1247935652345155584,
       'text': 'RT @AmberTheoharis: Love sports and travel? Join me and travel expert @LeeAbbamonte today on IG Live T 1pm PT/ 4pm ET. He’s been to everyon…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 12, 43),
       'screenname': 'VernellGordon',
       'id': 1247935416642256903,
       'text': 'RT @AmberTheoharis: Love sports and travel? Join me and travel expert @LeeAbbamonte today on IG Live T 1pm PT/ 4pm ET. He’s been to everyon…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 12, 6),
       'screenname': 'tinsellinsell',
       'id': 1247935260328935424,
       'text': '@leicspolice Just seen a couple of microlights flying in tandem and messing about over Harbs. Assuming this is esse… https://t.co/2RwcbDcD1X',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 10, 5),
       'screenname': 'AmberTheoharis',
       'id': 1247934752889290752,
       'text': 'Love sports and travel? Join me and travel expert @LeeAbbamonte today on IG Live T 1pm PT/ 4pm ET. He’s been to eve… https://t.co/711ko3G255',
       'retweet': 5,
       'favCount': 29,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 5, 36),
       'screenname': 'MarkWittig',
       'id': 1247933624374214659,
       'text': '@NARNfan @AndrewLeeTCNT Looks bad when all the other sports are not playing and that travel thing',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 17, 1, 2),
       'screenname': 'SwanBoatSteve',
       'id': 1247932477022375936,
       'text': '@regionomics Downtown works and would work better if only (a) there were even more transit and (b) we stopped sched… https://t.co/M2uRUDlBqE',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 16, 49, 12),
       'screenname': 'C4_Sports_',
       'id': 1247929499901100034,
       'text': 'A choice is never simple it’s never easy it’s not supposed to be but those who travel the path have always looked b… https://t.co/4PfNkQpTAV',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 16, 20, 10),
       'screenname': 'ThomasOJaeger',
       'id': 1247922190038773766,
       'text': 'RT @clublasanta: In the middle of the volcanic landscape of Lanzarote, lies a true paradise of sports, wellness and relaxation. Some say it…',
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 16, 7, 24),
       'screenname': 'sebgoursaud',
       'id': 1247918980272140289,
       'text': 'RT @rebelsart: @MickaelleMichel @SportsHochi Thank you for being a wonderful ambassador for horse racing and sports! Have a safe travel bac…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 16, 5, 19),
       'screenname': '___Taekook_',
       'id': 1247918453157216263,
       'text': 'Hey! I discovered the BEST app to practice English! 🇺🇸 It puts you in real life situations for you to learn in prac… https://t.co/xyub7kQ2Sp',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 16, 4, 54),
       'screenname': 'UHennessey',
       'id': 1247918348412878854,
       'text': 'Oh goodness, yes. If this was around when I was writing sports features (which took weeks and hours and cross count… https://t.co/HAffPiKI2h',
       'retweet': 0,
       'favCount': 3,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 15, 41, 35),
       'screenname': 'DaisyDots3',
       'id': 1247912480120156168,
       'text': 'RT @clublasanta: In the middle of the volcanic landscape of Lanzarote, lies a true paradise of sports, wellness and relaxation. Some say it…',
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 15, 33, 43),
       'screenname': 'TheWealthMind',
       'id': 1247910503151800321,
       'text': '@DallenReber Sports, Music, and Travel 🙌🏻',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 15, 27, 35),
       'screenname': 'jennifergorl',
       'id': 1247908958213476356,
       'text': "RT @Jose_Palmtree: All I'm tryna do this summer was turn up, travel and have deep late night talks with my friends and play sports and go t…",
       'retweet': 3,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 15, 19, 33),
       'screenname': 'FStarSolutions',
       'id': 1247906939104899073,
       'text': 'Friends support our efforts in elevating sports and travel hospitality programs by following our small company Firs… https://t.co/MJnjfcuQqX',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 15, 16, 12),
       'screenname': 'superaiwa',
       'id': 1247906092501196802,
       'text': 'Hey! I discovered the BEST app to practice Spanish! 🇪🇸 It puts you in real life situations for you to learn in prac… https://t.co/etbwxa1l32',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 15, 12, 33),
       'screenname': 'bjm262run',
       'id': 1247905175005614084,
       'text': '“By eliminating travel, removing fans, and enacting measures to ensure that athletes can’t easily transmit the viru… https://t.co/hn9w9nkOkj',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 15, 5, 5),
       'screenname': 'sportstravelint',
       'id': 1247903295886118914,
       'text': "So runners, who doesn't love a game of a bingo for a bit of light hearted fun. Here's your Sports Travel Internatio… https://t.co/vNsUbzCDVh",
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 15, 1, 58),
       'screenname': 'issa_reggie',
       'id': 1247902513061232643,
       'text': 'Because of the virus: cut off the cable because no sports. Can’t hoop. Can’t play football. Can’t workout. Can’t sh… https://t.co/Ry5HSdxqzk',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 14, 51, 9),
       'screenname': '_steegs',
       'id': 1247899791645765639,
       'text': 'RT @AmeshAA: "We\'ve got to expect that businesses must reopen and schools must teach again. Whether it\'s travel or sports or live entertain…',
       'retweet': 114,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 14, 39, 6),
       'screenname': 'NetballJo',
       'id': 1247896755955326983,
       'text': 'RT @clublasanta: In the middle of the volcanic landscape of Lanzarote, lies a true paradise of sports, wellness and relaxation. Some say it…',
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 14, 20, 10),
       'screenname': 'SteveODonnell1',
       'id': 1247891994875068418,
       'text': '@MikelSevere @damonbenning Having sports return with a crowd is a uplifting sign both economically and just in gene… https://t.co/MAV6E12mp1',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 13, 53, 21),
       'screenname': 'canariascalida',
       'id': 1247885244792881154,
       'text': 'RT @clublasanta: In the middle of the volcanic landscape of Lanzarote, lies a true paradise of sports, wellness and relaxation. Some say it…',
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 13, 53, 6),
       'screenname': 'theline4two',
       'id': 1247885180896858114,
       'text': '@Trace_AVP @Chuck1one @eepdllc @Rick__War @Woodshed_1914 @Freekeith @stateofthenewy1 @ginisangpepe @AJTheManChild… https://t.co/j4Ivsdknte',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 13, 50, 46),
       'screenname': 'ColleenSerran15',
       'id': 1247884593975267330,
       'text': 'RT @amzjoel: Life in China. Another day. Work and play. https://t.co/v7SC1lHiyH via @YouTube #travel #China #coronavirus #CoronaOutbreak #m…',
       'retweet': 1,
       'favCount': 0,
       'tags': ['travel', 'China', 'coronavirus', 'CoronaOutbreak']},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 13, 45, 19),
       'screenname': 'spanu_patricia',
       'id': 1247883224526868480,
       'text': 'RT @rebelsart: @MickaelleMichel @SportsHochi Thank you for being a wonderful ambassador for horse racing and sports! Have a safe travel bac…',
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 13, 33, 44),
       'screenname': 'EXONDIN',
       'id': 1247880306486435842,
       'text': 'Hobbies &amp; Interests : Animation, art, video games, computers, water sports, racket sports, nature sports, basketbal… https://t.co/l2yav8zEv8',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 13, 29, 56),
       'screenname': 'rebelsart',
       'id': 1247879352911454208,
       'text': '@MickaelleMichel @SportsHochi Thank you for being a wonderful ambassador for horse racing and sports! Have a safe t… https://t.co/ycMS1HnxAB',
       'retweet': 5,
       'favCount': 15,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 13, 23, 10),
       'screenname': 'clublasanta',
       'id': 1247877647629045760,
       'text': 'In the middle of the volcanic landscape of Lanzarote, lies a true paradise of sports, wellness and relaxation. Some… https://t.co/EjhAF48DqW',
       'retweet': 4,
       'favCount': 38,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 13, 21, 45),
       'screenname': 'a_travel_bot',
       'id': 1247877293126680577,
       'text': 'DO\nYou have to bring your own food however. Other water sports like boating and fishing are also plentiful.',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 13, 17, 16),
       'screenname': 'Travel_Cowboy',
       'id': 1247876164380700673,
       'text': '@dmn_cowboys bullshit @TimCowlishaw - sports returning as soon as they safely can (SAFELY) is one of the things tha… https://t.co/8bUIzLCxGy',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 12, 30, 59),
       'screenname': 'KCentral_Sports',
       'id': 1247864517901660163,
       'text': 'Megan Rushlau - Softball\n\nAlso participated in band. I have played travel softball for 6 years and will continue my… https://t.co/77DsSON0aB',
       'retweet': 0,
       'favCount': 4,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 11, 53, 14),
       'screenname': 'lahar1999',
       'id': 1247855014481018880,
       'text': '@MarkFagg The other sports got stuck when state borders closed to non-essential travel.  Racing can be run locally… https://t.co/fJy0UDqhgx',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 10, 24, 44),
       'screenname': 'TheShortBear',
       'id': 1247832744241094656,
       'text': '@szab0r A sports car is one of the dumbest purchases you can make from a purely financial standpoint. There are onl… https://t.co/GcKVzHfa9W',
       'retweet': 0,
       'favCount': 2,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 10, 11, 35),
       'screenname': 'spotlaits',
       'id': 1247829433886609408,
       'text': 'RT @getppcexpo: 7 Industries that are suffering from the #COVID19 Pandemic\n\n1. Travel and tourism\n2. Bars and restaurants\n3. Live entertain…',
       'retweet': 1,
       'favCount': 0,
       'tags': ['COVID19']},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 9, 46, 58),
       'screenname': 'RichCentury',
       'id': 1247823239750897668,
       'text': 'The wireless tour guide system is a rugged tour guide system such as travel guides, multilingual interpretation, he… https://t.co/Snb20SyR7f',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 8, 56, 26),
       'screenname': 'wirelesscamera3',
       'id': 1247810521568407552,
       'text': 'Monocular Telescope High Power, 12×42 Monocular Scope with Smartphone Adapter and Tripod Dual Focus for Outdoor Spo… https://t.co/hZxWwIOLkW',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 7, 38, 44),
       'screenname': 'KogojSlavko',
       'id': 1247790968583016449,
       'text': 'RT @HuginnTravel: Ukrepali so takoj po 2. in 3. primeru.\n\n"With just three confirmed coronavirus cases so far, Lithuania introduced yesterd…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 7, 33, 5),
       'screenname': 'HuginnTravel',
       'id': 1247789545363361796,
       'text': 'Ukrepali so takoj po 2. in 3. primeru.\n\n"With just three confirmed coronavirus cases so far, Lithuania introduced y… https://t.co/G8vG3czHyx',
       'retweet': 1,
       'favCount': 2,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 7, 8, 29),
       'screenname': 'shahidk56970047',
       'id': 1247783356219060227,
       'text': 'SkyGenius 10 x 50 Powerful Binoculars for Adults Durable Full-Size Clear Binoculars for Bird Watching Travel Sights… https://t.co/pzDxZMlk1c',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 6, 50, 46),
       'screenname': 'NassahKapur',
       'id': 1247778896142991361,
       'text': "@rid1kader I'm speculating that schools , religious gatherings and other large gatherings (sports events) will be p… https://t.co/Uet6a2uH8K",
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 5, 26),
       'screenname': 'RonMcGinnis9',
       'id': 1247757567092117504,
       'text': '@dreaminok7 @RealMattCouch Here in Pa. we are under a stay at home advisory until April 30 issued by the governor ,… https://t.co/HgDLsMZxmg',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 5, 25, 9),
       'screenname': 'RobertAlabaster',
       'id': 1247757349911015424,
       'text': "@nytimes True, the man is like a powerful sports car without a steering wheel. But that doesn't meant the WHO is of… https://t.co/ky0Tqyx72D",
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 3, 27, 58),
       'screenname': 'Clarissaarri',
       'id': 1247727863018606592,
       'text': "RT @Jose_Palmtree: All I'm tryna do this summer was turn up, travel and have deep late night talks with my friends and play sports and go t…",
       'retweet': 3,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 3, 9, 52),
       'screenname': 'pistons_eq_man',
       'id': 1247723304728891392,
       'text': 'RT @SamuelScobie77: Another great #EQRoundtable today organized by @NetsGov! Interesting discussion about travel challenges and solutions a…',
       'retweet': 1,
       'favCount': 0,
       'tags': ['EQRoundtable']},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 2, 33, 43),
       'screenname': 'progresivetrend',
       'id': 1247714209972002818,
       'text': 'RT @BLaw: Travel and tourism—which funnels $83 billion in tax revenue annually to state and local governments—is in a freefall triggered by…',
       'retweet': 3,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 2, 22, 55),
       'screenname': 'ARipple_DAsport',
       'id': 1247711492033691652,
       'text': 'RT @BradRosemas: Local travel softball teams effected by the coronavirus are optimistic their teams will remain strong during their breaks.…',
       'retweet': 3,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 2, 20, 49),
       'screenname': 'JoseAnzures6',
       'id': 1247710961336754176,
       'text': "RT @Jose_Palmtree: All I'm tryna do this summer was turn up, travel and have deep late night talks with my friends and play sports and go t…",
       'retweet': 3,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 2, 15, 40),
       'screenname': 'Jose_Palmtree',
       'id': 1247709667217768448,
       'text': "All I'm tryna do this summer was turn up, travel and have deep late night talks with my friends and play sports and… https://t.co/qiqRs6uWMk",
       'retweet': 3,
       'favCount': 7,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 1, 58, 27),
       'screenname': 'DADylanJohnson',
       'id': 1247705333478428672,
       'text': 'RT @BradRosemas: Local travel softball teams effected by the coronavirus are optimistic their teams will remain strong during their breaks.…',
       'retweet': 3,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 1, 45, 24),
       'screenname': 'oscorft',
       'id': 1247702051359821825,
       'text': 'Let me give you a window into the future. You will be denied travel, work, play sports, go to restaurants and any o… https://t.co/6YA23UrVHz',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 1, 37, 10),
       'screenname': 'pphilly10',
       'id': 1247699976635002880,
       'text': "@bobbythebookie @JeffLesson No travel sports for kids I've played more golf in 2 weeks than I do all summer normally and I'm a golf coach.",
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 1, 36, 59),
       'screenname': 'JamesSpenceley',
       'id': 1247699933076926464,
       'text': '@LT3000Lyall @buildthefutures @Mars10387340 @WillKoulouris Yes, and we (generally) no longer have unprotected sex,… https://t.co/FO5nA9Sn9d',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 1, 14, 29),
       'screenname': 'SamuelScobie77',
       'id': 1247694271198699520,
       'text': 'Another great #EQRoundtable today organized by @NetsGov! Interesting discussion about travel challenges and solutio… https://t.co/9zVx0lUfyk',
       'retweet': 1,
       'favCount': 15,
       'tags': ['EQRoundtable']},
      {'product_Line': 'Sports and travel',
       'created_at': datetime.datetime(2020, 4, 8, 1, 12, 53),
       'screenname': 'Cooljenim',
       'id': 1247693865127235584,
       'text': 'RT @BLaw: Travel and tourism—which funnels $83 billion in tax revenue annually to state and local governments—is in a freefall triggered by…',
       'retweet': 3,
       'favCount': 0,
       'tags': ''}],
     'Home and lifestyle': [{'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 49, 20),
       'screenname': 'OlgaSixta',
       'id': 1248020129809444865,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 46, 32),
       'screenname': 'emilyyy307',
       'id': 1248019424390635521,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 46, 13),
       'screenname': '4Everanimalz1',
       'id': 1248019344962883584,
       'text': "RT @SundayTimesZA: Mzansi has been baking up such a storm while stuck at home that there's a now a shortage of yeast on supermarket shelves…",
       'retweet': 13,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 43, 21),
       'screenname': 'ShellieRaygoza',
       'id': 1248018623169302528,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 41, 45),
       'screenname': 'emo__420',
       'id': 1248018219471736832,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 40, 15),
       'screenname': 'mattfromCal1',
       'id': 1248017841191653377,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 40, 13),
       'screenname': 'gornelas20',
       'id': 1248017834476580864,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 39, 23),
       'screenname': 'ChryslerLee',
       'id': 1248017626686574592,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 39, 17),
       'screenname': 'MSNNZ',
       'id': 1248017598295339008,
       'text': "Harry, Meghan 'plan to build home in the Cotswolds' https://t.co/RwnS5dg9u4",
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 38, 35),
       'screenname': '15Stephen15',
       'id': 1248017422759555074,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 37, 40),
       'screenname': 'sally06301',
       'id': 1248017193733771265,
       'text': 'RT @MNMLCase: Minimalists aren’t opposed to spending money. However, we are generally against spending too much of it without a darn good r…',
       'retweet': 156,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 37, 31),
       'screenname': 'dianexav1',
       'id': 1248017156719009793,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 36, 11),
       'screenname': 'itsHemantSharma',
       'id': 1248016817685061633,
       'text': 'RT @Upwork: 😴Sleep later\n💻Set up your desk\n📷Be ready for prime time\n🌿Make your home a pleasant place to be\n👫Maintain your connections and m…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 35, 11),
       'screenname': 'ChristopherVesk',
       'id': 1248016567700340738,
       'text': 'In order to educate the children about Jesus. We as parents have to live in righteousness.\n\nThat they see and know… https://t.co/oUzq86NaPP',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 35, 6),
       'screenname': 'NewCreationCap',
       'id': 1248016545848025088,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 34, 56),
       'screenname': 'luisa71473442',
       'id': 1248016506308321282,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 33, 38),
       'screenname': 'kalvinthedude',
       'id': 1248016178963857410,
       'text': 'The mfs that are shitting on people doing home workouts are the same ones that made fun of the fat kids growing up.… https://t.co/851kqrPWcZ',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 33, 27),
       'screenname': 'GustavoArellano',
       'id': 1248016132075696128,
       'text': 'RT @hbecerraLATimes: I’m going to take a wild guess that \u2066@mrmarkpotts\u2069 asked this question: “If I have no symptoms and I’m at home and I’v…',
       'retweet': 2,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 32, 3),
       'screenname': 'hbecerraLATimes',
       'id': 1248015778227441664,
       'text': 'I’m going to take a wild guess that \u2066@mrmarkpotts\u2069 asked this question: “If I have no symptoms and I’m at home and… https://t.co/4gShCoCAph',
       'retweet': 2,
       'favCount': 4,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 30, 53),
       'screenname': 'Firemonkey991',
       'id': 1248015485951545345,
       'text': '@MagazineAmplify Being accustomed to home life, the 3 of us are handling the lockdown pretty well.\nWe managed to st… https://t.co/1akbXsIAj1',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 30, 11),
       'screenname': 'MEOWFoundation',
       'id': 1248015309795028992,
       'text': "The humans may be tiring of working from home, but it looks like their kitty co-workers ain't! What's better than c… https://t.co/RfXa88WqZ5",
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 27, 15),
       'screenname': 'd7473',
       'id': 1248014573610786822,
       'text': 'RT @MNMLCase: Minimalists aren’t opposed to spending money. However, we are generally against spending too much of it without a darn good r…',
       'retweet': 156,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 24, 55),
       'screenname': 'outdampuff',
       'id': 1248013982436184065,
       'text': 'RT @SenBobCasey: Millions of Americans depend on Meals on Wheels for their nourishment, and three-quarters of the volunteers who deliver th…',
       'retweet': 10,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 24, 53),
       'screenname': 'awfungowie',
       'id': 1248013976224423936,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 24, 20),
       'screenname': 'ameredoux',
       'id': 1248013836717707264,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 24, 18),
       'screenname': 'billymayssteps1',
       'id': 1248013828014522369,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 23, 38),
       'screenname': 'Sixtyminmn',
       'id': 1248013660015833088,
       'text': 'RT @MNMLCase: Minimalists aren’t opposed to spending money. However, we are generally against spending too much of it without a darn good r…',
       'retweet': 156,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 23, 25),
       'screenname': 'MichaelPell',
       'id': 1248013608887283713,
       'text': 'RT @sunriseon7: Police across Australia will be out in force over the Easter long weekend to enforce strict social distancing restrictions…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 20, 51),
       'screenname': 'jeepjohn',
       'id': 1248012961815228416,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 20, 12),
       'screenname': 'valleygirljulie',
       'id': 1248012797440438274,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 20, 2),
       'screenname': 'LattimorePT',
       'id': 1248012755803590657,
       'text': 'Many of us have recently adopted the working from home lifestyle, which could possibly lead to poor posture and bac… https://t.co/WaeUM6ROYu',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 19, 19),
       'screenname': 'moh_choudhury',
       'id': 1248012573284261888,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 19, 17),
       'screenname': 'lindy782',
       'id': 1248012565520637952,
       'text': '@michaelharriot @KassandraSeven You are correct. Until you point out exactly why this is occurring nobody looks at… https://t.co/xPHGdhDUUB',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 18, 42),
       'screenname': 'aleeeganjaaa',
       'id': 1248012418942267392,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 17, 43),
       'screenname': 'Fumb_Ducks',
       'id': 1248012174481448960,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 16, 50),
       'screenname': 'HugoFeijo',
       'id': 1248011948186169346,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 15, 51),
       'screenname': 'KatTompkins',
       'id': 1248011704648101889,
       'text': 'RT @SenBobCasey: Millions of Americans depend on Meals on Wheels for their nourishment, and three-quarters of the volunteers who deliver th…',
       'retweet': 10,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 14, 50),
       'screenname': 'NYBLURBS',
       'id': 1248011446870401029,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 14, 26),
       'screenname': 'TrizzyTray_',
       'id': 1248011347201101824,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 13, 32),
       'screenname': 'JessSargus',
       'id': 1248011120503181313,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 13, 22),
       'screenname': 'nkdec',
       'id': 1248011078253993989,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 12, 57),
       'screenname': 'MainelyLeahy',
       'id': 1248010971668307969,
       'text': 'RT @SenBobCasey: Millions of Americans depend on Meals on Wheels for their nourishment, and three-quarters of the volunteers who deliver th…',
       'retweet': 10,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 12, 15),
       'screenname': 'demetripanos',
       'id': 1248010795981520896,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 11, 22),
       'screenname': 'dj_jiang',
       'id': 1248010574195118086,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 10, 53),
       'screenname': 'Myrtil03',
       'id': 1248010450853224450,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 10, 24),
       'screenname': 'BAULAPARRANTES',
       'id': 1248010329310638081,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 9, 13),
       'screenname': 'theycallmemo_',
       'id': 1248010032148443136,
       'text': 'RT @DeeDeeFlora: I suffer from anxiety and this has been my 4th week at home. It gets easier once you accept the new lifestyle &amp; create you…',
       'retweet': 13,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 8, 38),
       'screenname': 'cliffjorgensen',
       'id': 1248009886073368576,
       'text': 'Check out this American Lifestyle Magazine blog post! The Difference Between Home Staging and Decorating https://t.co/NMFsrf2Uuh',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 8, 21),
       'screenname': 'Sylmois_World',
       'id': 1248009813574815745,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 8, 13),
       'screenname': '_HeyMike',
       'id': 1248009782063034368,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 8, 9),
       'screenname': 'shabana_2018',
       'id': 1248009762844725249,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 7, 10),
       'screenname': 'pandojevito01',
       'id': 1248009516907499520,
       'text': 'RT @MNMLCase: Minimalists aren’t opposed to spending money. However, we are generally against spending too much of it without a darn good r…',
       'retweet': 156,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 6),
       'screenname': 'Upwork',
       'id': 1248009225072070656,
       'text': '😴Sleep later\n💻Set up your desk\n📷Be ready for prime time\n🌿Make your home a pleasant place to be\n👫Maintain your conne… https://t.co/guZo8AzmA3',
       'retweet': 1,
       'favCount': 10,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 5, 7),
       'screenname': 'jrobertsonNM',
       'id': 1248009002224476160,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 4, 58),
       'screenname': 'ChristineIAm',
       'id': 1248008962118565888,
       'text': 'RT @Newsday: “20 years from now, your children will not remember what they learned during the spring of 2020…They WILL remember the time th…',
       'retweet': 7,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 4, 46),
       'screenname': 'Lee_Marlow_Poly',
       'id': 1248008913997271040,
       'text': 'How to successfully work from home and do remote-learning with a full household. https://t.co/nYIPmqrpvk',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 4, 20),
       'screenname': 'VCSTX',
       'id': 1248008806040096769,
       'text': 'RT @VCSTX: First #clean &amp; then #disinfect: \nhow to do it right to keep the #coronavirus at bay 🏠🦠  \n\nPlease respect dwell times and use ade…',
       'retweet': 1,
       'favCount': 0,
       'tags': ['clean', 'disinfect', 'coronavirus']},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 3, 34),
       'screenname': 'theReal_Rebel',
       'id': 1248008612863082497,
       'text': 'RT @SenBobCasey: Millions of Americans depend on Meals on Wheels for their nourishment, and three-quarters of the volunteers who deliver th…',
       'retweet': 10,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 3, 25),
       'screenname': 'ouragami_',
       'id': 1248008575168864256,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 3, 20),
       'screenname': 'Himself3909',
       'id': 1248008554373472256,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 2, 47),
       'screenname': 'ActionSonora',
       'id': 1248008413205778438,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 2, 41),
       'screenname': '30ACTruth',
       'id': 1248008390950842368,
       'text': 'RT @SenBobCasey: Millions of Americans depend on Meals on Wheels for their nourishment, and three-quarters of the volunteers who deliver th…',
       'retweet': 10,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 2, 33),
       'screenname': 'AngeliqueGammon',
       'id': 1248008355269861376,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 2, 14),
       'screenname': 'ibpixiechick',
       'id': 1248008276689575936,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 1, 59),
       'screenname': 'otqvv',
       'id': 1248008214609723392,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 1, 8),
       'screenname': '1citizenpundit',
       'id': 1248007997453819909,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 1, 6),
       'screenname': 'bentoncarolyn25',
       'id': 1248007991913152512,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 1, 4),
       'screenname': 'ZombieJester',
       'id': 1248007984237559808,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 1),
       'screenname': 'cleverhoax',
       'id': 1248007964595597313,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 22, 0, 46),
       'screenname': 'jewelrymavin3',
       'id': 1248007905271402497,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 59, 9),
       'screenname': 'SamLitzinger',
       'id': 1248007501255065603,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 57, 40),
       'screenname': 'FostersDailyDem',
       'id': 1248007126095519744,
       'text': 'Dr. David Itkin of @PortsmouthReg Hospital answers questions we’ve received from readers, including whether you sho… https://t.co/RGAmmnmbHS',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 57, 25),
       'screenname': 'seacoastonline',
       'id': 1248007062786732032,
       'text': 'Dr. David Itkin of @PortsmouthReg Hospital answers questions we’ve received from readers, including whether you sho… https://t.co/POJhbuyc70',
       'retweet': 0,
       'favCount': 3,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 57, 23),
       'screenname': 'Aviv______',
       'id': 1248007056558194690,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 57, 21),
       'screenname': '30ACTruth',
       'id': 1248007046261125122,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 57, 20),
       'screenname': 'rwysocki34',
       'id': 1248007044273061889,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 57, 7),
       'screenname': 'libf9wt',
       'id': 1248006988853686272,
       'text': 'RT @SenBobCasey: Millions of Americans depend on Meals on Wheels for their nourishment, and three-quarters of the volunteers who deliver th…',
       'retweet': 10,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 56, 33),
       'screenname': 'bohring23',
       'id': 1248006847530860544,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 56, 21),
       'screenname': 'GRMSrPa',
       'id': 1248006796817469440,
       'text': 'RT @SenBobCasey: Millions of Americans depend on Meals on Wheels for their nourishment, and three-quarters of the volunteers who deliver th…',
       'retweet': 10,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 56, 12),
       'screenname': 'ArgusC',
       'id': 1248006756686417922,
       'text': 'RT @SenBobCasey: Millions of Americans depend on Meals on Wheels for their nourishment, and three-quarters of the volunteers who deliver th…',
       'retweet': 10,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 55, 53),
       'screenname': 'Harlegator68',
       'id': 1248006679184080896,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 55, 51),
       'screenname': 'ElArturoAleman',
       'id': 1248006668819947520,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 55, 34),
       'screenname': 'mkwmkwmkwmkw',
       'id': 1248006596656893952,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 55, 24),
       'screenname': 'Lynmar2020',
       'id': 1248006556983029760,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 55, 6),
       'screenname': 'FrontierMetrix',
       'id': 1248006482177581056,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 54, 58),
       'screenname': '_yayyyo',
       'id': 1248006447650111489,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 54, 56),
       'screenname': 'LindyLoo515',
       'id': 1248006438149951488,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 54, 53),
       'screenname': 'tra255',
       'id': 1248006425579638785,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 54, 26),
       'screenname': 'twaylondra',
       'id': 1248006312497049600,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 54, 14),
       'screenname': 'willnanken',
       'id': 1248006262085668864,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 53, 56),
       'screenname': 'Jonatha80958138',
       'id': 1248006187833909248,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 53, 51),
       'screenname': 'Emadalayoubi',
       'id': 1248006168103907329,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 53, 43),
       'screenname': 'ParkWardenVoice',
       'id': 1248006132112617472,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 53, 25),
       'screenname': 'pnwrunnerlass',
       'id': 1248006057269608456,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 53, 24),
       'screenname': 'just_jaackiee',
       'id': 1248006054824181760,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 53, 9),
       'screenname': 'Binahsaurus',
       'id': 1248005991158779904,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 53, 7),
       'screenname': 'Rich_Wagner_Esq',
       'id': 1248005980748537857,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 53, 3),
       'screenname': 'KDHcharley',
       'id': 1248005963660976129,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 52, 3),
       'screenname': 'OnyaMag',
       'id': 1248005711100964864,
       'text': "As we all do our part to stay home and flatten the curve, some of @discoverLA's best institutions are committed to… https://t.co/m6Rdl5RFAD",
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Home and lifestyle',
       'created_at': datetime.datetime(2020, 4, 8, 21, 51, 48),
       'screenname': '_valleygirl07',
       'id': 1248005649000099841,
       'text': 'RT @latimes: Gov. Gavin Newsom said the order to stay at home to help slow the spread of the coronavirus will continue “until further notic…',
       'retweet': 103,
       'favCount': 0,
       'tags': ''}],
     'Fashion accessories': [{'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 49, 36),
       'screenname': 'CherylD93415695',
       'id': 1248020196154994689,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/jlve6tkvxd',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 49, 31),
       'screenname': 'LeslieL03210376',
       'id': 1248020176257208321,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/DP8dUHMly5',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 49, 11),
       'screenname': 'liacallahan4',
       'id': 1248020089150017537,
       'text': "RT @SashkaCo: #Giveaway!! We're giving away 3\xa0 Sashka Co. bracelets to 3 #friends!\nTo Enter: \n1. Follow @SashkaCo\n2. Like this post\n3. Tag…",
       'retweet': 77,
       'favCount': 0,
       'tags': ['Giveaway', 'friends']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 48, 40),
       'screenname': 'FashionWarehou2',
       'id': 1248019962305765376,
       'text': 'Fashion Tiny Dainty Heart Initial Necklace Personalized Letter Necklace Name Jewelry for women accessories girlfrie… https://t.co/Iw7X7cL3IQ',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 48, 25),
       'screenname': 'MissyNordone',
       'id': 1248019897386438656,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/mcPgecPT7b',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 47, 8),
       'screenname': 'YvonneM03742581',
       'id': 1248019576220049409,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/gqIZMY263v',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 45, 50),
       'screenname': 'TaraPac53738921',
       'id': 1248019248753987584,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/u5dzN9I7e5',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 44, 40),
       'screenname': 'Kristen44889792',
       'id': 1248018956402610176,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/FUt5V8w1X9',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 44, 14),
       'screenname': 'Kimberl54012436',
       'id': 1248018843961659392,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/MIY6jOP4NS',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 43, 50),
       'screenname': 'DebbieJ65020701',
       'id': 1248018746473476096,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/IGuI9cTlFk',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 43, 34),
       'screenname': 'JacquelineCarf3',
       'id': 1248018679691763714,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/dVYI3GISf5',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 43, 16),
       'screenname': 'AnnieJa64908448',
       'id': 1248018602071961600,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/z50EUoTF7k',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 42, 7),
       'screenname': 'DanielaPatel14',
       'id': 1248018314363719680,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/80m3T9xwiC',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 41, 53),
       'screenname': 'ReginaH94655472',
       'id': 1248018253999271937,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/WNmqv6n8Iw',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 41, 31),
       'screenname': 'Lynnett76799182',
       'id': 1248018162336952320,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/1YhEo2V0Xi',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 40, 53),
       'screenname': 'KatieEd83444316',
       'id': 1248018002794049536,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/NrCYGpUNcj',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 40, 22),
       'screenname': 'MogelAmanda',
       'id': 1248017870824460288,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/vw2ezpTGJY',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 39, 50),
       'screenname': 'AmandaO21831843',
       'id': 1248017739387514881,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/qVfPDUk6uo',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 39, 45),
       'screenname': 'Jessica39352481',
       'id': 1248017718848000001,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/wOs0bQlXcd',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 39, 45),
       'screenname': 'AndreaB70213136',
       'id': 1248017717782691841,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/aTEaedQ3JY',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 39, 42),
       'screenname': 'Monique73298032',
       'id': 1248017706328064002,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/4yXxbbyMJT',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 38, 19),
       'screenname': 'AmberMi50225425',
       'id': 1248017356405633025,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/2xYcHsoaAm',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 37, 57),
       'screenname': 'Shannon61190986',
       'id': 1248017265909350400,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/Fh7BvMY3jw',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 37, 49),
       'screenname': 'Whitney33869058',
       'id': 1248017230450704387,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/jhhMUBDooc',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 37, 48),
       'screenname': 'MonicaB82065975',
       'id': 1248017226072023042,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/fuzxDhBNCL',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 35, 57),
       'screenname': 'FrittsBrandi',
       'id': 1248016761963704321,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/TVTD3Jtss5',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 34, 57),
       'screenname': 'KellyTh80014960',
       'id': 1248016508413865984,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/Cz3oes57nQ',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 34, 7),
       'screenname': 'BidorBuy247',
       'id': 1248016299508133888,
       'text': 'Trending 2020 Rimless Sunglasses Women Fashion Tears Shape Shades Sun Glasses https://t.co/hIqU2Xx9GQ https://t.co/B1oAmiJiRQ',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 33, 43),
       'screenname': 'MissyNordone',
       'id': 1248016200421896192,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/znXDQDi5Dm',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 33, 6),
       'screenname': 'LeslieL03210376',
       'id': 1248016045140406273,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/NabaBHpy7s',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 32, 1),
       'screenname': 'CherylD93415695',
       'id': 1248015772229570560,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/lIb17eeDr0',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 31, 54),
       'screenname': 'GODUSTYLE',
       'id': 1248015741728645120,
       'text': '@oliviaculpo #fashion dior #party #dress #look #style #details #accessories #influencer #outfit #inspiration… https://t.co/oMRk5H2WKM',
       'retweet': 0,
       'favCount': 0,
       'tags': ['fashion',
        'party',
        'dress',
        'look',
        'style',
        'details',
        'accessories',
        'influencer',
        'outfit',
        'inspiration']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 31, 26),
       'screenname': 'YvonneM03742581',
       'id': 1248015624472682497,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/utPhiq2evx',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 31, 22),
       'screenname': 'DebbieJ65020701',
       'id': 1248015607183716352,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/Dy1Lmsqa98',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 30, 45),
       'screenname': 'cudaeducation',
       'id': 1248015450882990081,
       'text': '2020 FASHION BIZ REPORT:  Get real statistics, facts and information about the #fashionbusiness |… https://t.co/sMgHg3PBZV',
       'retweet': 0,
       'favCount': 0,
       'tags': ['fashionbusiness']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 30, 21),
       'screenname': 'myfoodfantasy69',
       'id': 1248015350492348417,
       'text': 'RT @Charlesfrize: Reading about this: #Fashion - What Are The Essential Accessories #Style #FrizeMedia - https://t.co/LShAZPyY8M @Charlesfr…',
       'retweet': 3,
       'favCount': 0,
       'tags': ['Fashion', 'Style', 'FrizeMedia']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 30, 9),
       'screenname': 'JacquelineCarf3',
       'id': 1248015303243513858,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/wioH7vLgDE',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 29, 50),
       'screenname': 'Kristen44889792',
       'id': 1248015223400742915,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/xigYHqsaAt',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 29, 11),
       'screenname': 'AnnieJa64908448',
       'id': 1248015057851568128,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/ITct3qsaGt',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 28, 58),
       'screenname': 'Kimberl54012436',
       'id': 1248015004235747328,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/kHKi7u63nA',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 28, 51),
       'screenname': 'TaraPac53738921',
       'id': 1248014972417789952,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/CLtderPPX9',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 28, 47),
       'screenname': 'Lynnett76799182',
       'id': 1248014958853410816,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/nftNhsbtEC',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 28, 3),
       'screenname': 'KatieEd83444316',
       'id': 1248014774022991872,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/0D54fSPlP1',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 27, 48),
       'screenname': 'ReginaH94655472',
       'id': 1248014710923907072,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/yD8yqi5Xpg',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 27, 32),
       'screenname': 'AndreaB70213136',
       'id': 1248014644884594693,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/gAbLVwhs7v',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 27, 8),
       'screenname': 'BluediamondsGS',
       'id': 1248014540362547200,
       'text': '925 Sterling Silver Ladies Large Oval Ring and Unakite Cabochon Sizes J-R | eBay #handmade #statement… https://t.co/HNGsP482G2',
       'retweet': 0,
       'favCount': 0,
       'tags': ['handmade', 'statement']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 27, 3),
       'screenname': 'Jessica39352481',
       'id': 1248014521056120834,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/vfEjj26Y6T',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 26, 28),
       'screenname': 'maryhigley1',
       'id': 1248014376147116032,
       'text': 'Brooch Pin Broche Clear Rhinestones Fashion Wedding Vintage Jewelry Vendimia Joyeria Bridal Sash Accessories Hollyw… https://t.co/oauBpABRzR',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 25, 9),
       'screenname': 'MogelAmanda',
       'id': 1248014044948127744,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/jrhU1G87Dk',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 23, 33),
       'screenname': 'Monique73298032',
       'id': 1248013638448734208,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/1KsezTFPuj',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 23, 1),
       'screenname': 'YeniExpoSEO',
       'id': 1248013506445631489,
       'text': 'Women Chic Long Sleeve Long Length Jacket  38-48 Jk 01 https://t.co/EDffSnL00p #Toptan #turkeyexport https://t.co/94TEjypOjN',
       'retweet': 0,
       'favCount': 0,
       'tags': ['Toptan', 'turkeyexport']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 22, 58),
       'screenname': 'Whitney33869058',
       'id': 1248013492222746624,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/JECnc49QDJ',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 22, 38),
       'screenname': 'AmandaO21831843',
       'id': 1248013410911932418,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/mwHIMj4bMP',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 22, 35),
       'screenname': 'Shannon61190986',
       'id': 1248013397209149443,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/ORVZpXnVgX',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 22, 27),
       'screenname': 'MonicaB82065975',
       'id': 1248013363948343297,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/e6vC3HcbW3',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 22, 11),
       'screenname': 'DanielaPatel14',
       'id': 1248013298408120320,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/fLqIfOxOoA',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 22, 5),
       'screenname': 'FrittsBrandi',
       'id': 1248013269660360707,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/XEt9TL3L7a',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 21, 29),
       'screenname': 'AmberMi50225425',
       'id': 1248013121496612864,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/j4dhDoYkuy',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 21, 25),
       'screenname': 'hannah15mini',
       'id': 1248013101619802114,
       'text': "RT @SashkaCo: #Giveaway!! We're giving away 3\xa0 Sashka Co. bracelets to 3 #friends!\nTo Enter: \n1. Follow @SashkaCo\n2. Like this post\n3. Tag…",
       'retweet': 77,
       'favCount': 0,
       'tags': ['Giveaway', 'friends']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 21, 14),
       'screenname': 'MissyNordone',
       'id': 1248013056950423552,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/kdZhJD07lD',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 19, 17),
       'screenname': 'DebbieJ65020701',
       'id': 1248012564929232896,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/NfeAGb947M',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 18, 56),
       'screenname': 'LeslieL03210376',
       'id': 1248012476630749186,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/Tc4be9GPCz',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 18, 9),
       'screenname': 'CherylD93415695',
       'id': 1248012280945467393,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/qFuI9wHKE0',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 17, 43),
       'screenname': 'TaraPac53738921',
       'id': 1248012172858241026,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/vlnFfbFc6E',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 17, 18),
       'screenname': 'MissAyoDele',
       'id': 1248012069099565056,
       'text': '#model #style #swag #classy #scarifications #love #beautiful #beautifulpeople #beauty #amazing \n#fashion #woman… https://t.co/JfDybwdUcz',
       'retweet': 0,
       'favCount': 0,
       'tags': ['model',
        'style',
        'swag',
        'classy',
        'scarifications',
        'love',
        'beautiful',
        'beautifulpeople',
        'beauty',
        'amazing',
        'fashion',
        'woman']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 17, 9),
       'screenname': 'KellyTh80014960',
       'id': 1248012031430516736,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/u2GTluyyVY',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 17, 6),
       'screenname': 'Lumpimp',
       'id': 1248012016897232896,
       'text': "RT @acnhivan: 🌸 FASHION ITEMS SALE!! 🌸\nI'm looking to sell some of my clothing and accessories. After an item is sold, it's gone. I will or…",
       'retweet': 5,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 17, 1),
       'screenname': 'CosmoOAaaarty',
       'id': 1248011994315157505,
       'text': 'Accessories - Hair - Fashion - luxe https://t.co/Zsbm9VNool',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 16, 42),
       'screenname': 'AnnieJa64908448',
       'id': 1248011916611473408,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/ANkFo29Kk5',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 16, 37),
       'screenname': 'YvonneM03742581',
       'id': 1248011895660920834,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/I8ZwJCBGJ4',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 16),
       'screenname': 'JacquelineCarf3',
       'id': 1248011740056440833,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/pmnEvvNKAi',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 15, 42),
       'screenname': 'Lynnett76799182',
       'id': 1248011665808871424,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/wYyPxHHlNW',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 14, 45),
       'screenname': 'Kristen44889792',
       'id': 1248011427618549760,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/zBJB2cboX5',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 14, 35),
       'screenname': 'ranjpower',
       'id': 1248011384773722112,
       'text': "RT @SashkaCo: #Giveaway!! We're giving away 3\xa0 Sashka Co. bracelets to 3 #friends!\nTo Enter: \n1. Follow @SashkaCo\n2. Like this post\n3. Tag…",
       'retweet': 77,
       'favCount': 0,
       'tags': ['Giveaway', 'friends']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 13, 38),
       'screenname': 'ZenshyS',
       'id': 1248011146138812416,
       'text': 'Leather Wrap Bracelet (7 Colors) 17.99$\nGet it here: https://t.co/4B59ObPZrn\nLike &amp; Follow Us @zenshys \n(50%SALE &amp;… https://t.co/usmOgQ82xk',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 13, 36),
       'screenname': 'AndreaB70213136',
       'id': 1248011136462508032,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/jINb8saipm',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 13, 35),
       'screenname': 'pharrispenny',
       'id': 1248011134134677505,
       'text': "RT @SashkaCo: #Giveaway!! We're giving away 3\xa0 Sashka Co. bracelets to 3 #friends!\nTo Enter: \n1. Follow @SashkaCo\n2. Like this post\n3. Tag…",
       'retweet': 77,
       'favCount': 0,
       'tags': ['Giveaway', 'friends']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 11, 48),
       'screenname': 'Kimberl54012436',
       'id': 1248010684509515777,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/dmmRNACfFs',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 11, 46),
       'screenname': 'KatieEd83444316',
       'id': 1248010674132770816,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/mT2VEKcE2C',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 11, 21),
       'screenname': 'gossamerfae',
       'id': 1248010571095478274,
       'text': 'Check out Cross Choker Necklace Handmade Adjustable Silver Accessories Fashion Black #Handmade https://t.co/H5tbNBvjrV via @eBay',
       'retweet': 0,
       'favCount': 1,
       'tags': ['Handmade']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 10, 29),
       'screenname': 'BayBlingBeauty',
       'id': 1248010351540461568,
       'text': 'All we can think of is summer ☀️ Body chains are the perfect way to accessorize a simple outfit 😍 Link in bio 👆🏻\n\n•… https://t.co/lYzakUYxfi',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 10, 18),
       'screenname': 'Whitney33869058',
       'id': 1248010303901560832,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/0sIu7cxlNc',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 9, 54),
       'screenname': 'Jessica39352481',
       'id': 1248010205016670210,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/NDoKMSMZth',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 9, 31),
       'screenname': 'Shannon61190986',
       'id': 1248010109579493377,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/ELwLoPY1tY',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 9, 14),
       'screenname': 'ReginaH94655472',
       'id': 1248010039106760707,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/PGI6gkwf8n',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 8, 59),
       'screenname': 'AmberMi50225425',
       'id': 1248009974879367168,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/kauuGdYqeX',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 7, 48),
       'screenname': 'BrendaJBear1',
       'id': 1248009675347394560,
       'text': '@cat01cat01cat01 @GregariousGus @SheilaMSpence1 @Blutospin @OffTheLeashFP @GracieRoadster @basset_bella… https://t.co/KsvZZBLpsx',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 7, 35),
       'screenname': 'DanielaPatel14',
       'id': 1248009620699807745,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/QuppsoqLgO',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 7, 33),
       'screenname': 'Monique73298032',
       'id': 1248009615674970112,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/BYXEAOEXxV',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 5, 53),
       'screenname': 'AmandaO21831843',
       'id': 1248009192448782338,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/x5GGaZbP2v',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 4, 18),
       'screenname': 'RLushington',
       'id': 1248008795227213825,
       'text': '@WhatsupLiz @cookiegigan @ForeverAngel26 #friends #style #beautiful #jewelry #bracelet #fashion #accessories #trend… https://t.co/cutgscebgc',
       'retweet': 0,
       'favCount': 1,
       'tags': ['friends',
        'style',
        'beautiful',
        'jewelry',
        'bracelet',
        'fashion',
        'accessories',
        'trend']},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 4),
       'screenname': 'YvonneM03742581',
       'id': 1248008720442773504,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/6qTkT3l3wL',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 3, 59),
       'screenname': 'CherylD93415695',
       'id': 1248008717829734401,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/10wvTtsYsW',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 3, 3),
       'screenname': 'MogelAmanda',
       'id': 1248008483275849729,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/ZLMxqgMN5Q',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 3, 2),
       'screenname': 'KellyTh80014960',
       'id': 1248008477475139584,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/oIemi7AqXU',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 3),
       'screenname': 'MissyNordone',
       'id': 1248008469455589377,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/90pNvym1BN',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 2, 39),
       'screenname': 'FrittsBrandi',
       'id': 1248008379236139009,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/prKRdMvS0K',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 2, 29),
       'screenname': 'Lynnett76799182',
       'id': 1248008339952291841,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/ts1MBTiuLv',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 2, 29),
       'screenname': 'AnnieJa64908448',
       'id': 1248008337070772227,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/Ry0l2jSsZx',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Fashion accessories',
       'created_at': datetime.datetime(2020, 4, 8, 22, 2, 5),
       'screenname': 'Kristen44889792',
       'id': 1248008236113903618,
       'text': 'Fashion, Beauty, Home Accessories &amp; Baby centrepointstores\n\nBuy gift cards and use them online with ease \norders wi… https://t.co/rPGDkutYRQ',
       'retweet': 0,
       'favCount': 0,
       'tags': ''}],
     'Health and beauty': [{'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 47, 58),
       'screenname': 'GrayskyX',
       'id': 1248019784039411712,
       'text': 'Those who love themselves, by wealth, beauty, or "health" will limit life to themselves, and be complacent and call… https://t.co/SPCBWffDlY',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 47, 24),
       'screenname': 'velocipedus',
       'id': 1248019644289445890,
       'text': '@allpartycycling @bmj_latest The beauty and power of such studies: They provide hard evidence to bring together hea… https://t.co/rRRXrObiLL',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 47, 8),
       'screenname': 'mvsiii71',
       'id': 1248019576043917312,
       'text': 'Come take a look at all Avon has to offer for health and beauty needs! https://t.co/qBqQCTkWuw https://t.co/rqm5chgzJl',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 39, 46),
       'screenname': 'Treuburg',
       'id': 1248017721532416003,
       'text': 'RT @JackStewartCham: Hundreds move into small, run down or boring town and revitalize infrastructure, display health and vitality and inspi…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 34, 51),
       'screenname': 'JackStewartCham',
       'id': 1248016482690199555,
       'text': 'Hundreds move into small, run down or boring town and revitalize infrastructure, display health and vitality and in… https://t.co/NZitR9TqAv',
       'retweet': 1,
       'favCount': 3,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 30, 54),
       'screenname': 'dawn27331807',
       'id': 1248015490057822208,
       'text': '@realDonaldTrump I work in the beauty industry and I touch people all day ... I’m not in any rush to go to work...… https://t.co/mvjjWJ2Cnu',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 30, 43),
       'screenname': 'TheVibeDealer3',
       'id': 1248015444365066243,
       'text': 'RT @tea_natasha: The simplicity of soothing Spearmint and lightly floral Rose Hip blend is a simplistic blend of balance and beauty. Enjoy…',
       'retweet': 2,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 27, 7),
       'screenname': 'HollyThomas61',
       'id': 1248014536809967616,
       'text': 'RT @ladyboarder9669: Natural, easy, at home recipes to boost immuity for yourself and home during Covid-19 #covid19 #immunityboost #home #e…',
       'retweet': 3,
       'favCount': 0,
       'tags': ['covid19', 'immunityboost', 'home']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 12, 42),
       'screenname': 'EmanHabbash',
       'id': 1248010909819142144,
       'text': 'RT @HeightsPodium: It starts from health all the way to media.. we will go through sports, culture, innovation, coffee, Comedy, beauty, art…',
       'retweet': 15,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 10, 19),
       'screenname': 'ewhoma_',
       'id': 1248010309840719873,
       'text': 'RT @KelechiAttamah: Law firms are already issuing opinions on covid-19 and employment contracts to their corporate clients. The beauty of l…',
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 9, 10),
       'screenname': 'USIIV',
       'id': 1248010022266630147,
       'text': 'RT @KelechiAttamah: Law firms are already issuing opinions on covid-19 and employment contracts to their corporate clients. The beauty of l…',
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 8, 5),
       'screenname': 'BidorBuy247',
       'id': 1248009747158073344,
       'text': 'Remedy Antifungal Soap, Helps Wash Away Body Odor Fungus Shower Gel Women/Men https://t.co/OqZRPaDV6n https://t.co/NRKUMVteJ8',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 2, 50),
       'screenname': 'Its_3One4',
       'id': 1248008428703731718,
       'text': 'RT @Ali_yeganeh_s: The twelfth Imam, Imam Mahdi, is not only the Imam of the Shiites, but also the Imam of all human beings With his coming…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 1, 43),
       'screenname': 'Ali_yeganeh_s',
       'id': 1248008144229285893,
       'text': 'The twelfth Imam, Imam Mahdi, is not only the Imam of the Shiites, but also the Imam of all human beings With his c… https://t.co/TVN4l6XP17',
       'retweet': 1,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 22, 0, 8),
       'screenname': 'Gentleman__7',
       'id': 1248007746009493504,
       'text': 'RT @tea_natasha: The simplicity of soothing Spearmint and lightly floral Rose Hip blend is a simplistic blend of balance and beauty. Enjoy…',
       'retweet': 2,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 59, 11),
       'screenname': 'TraceyBenmore',
       'id': 1248007509400379393,
       'text': 'RT @InStyle: A total upheaval of life as we know it — and social distance from the triggers that might push people to drink too much — is l…',
       'retweet': 2,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 57, 34),
       'screenname': 'InStyle',
       'id': 1248007100040531971,
       'text': 'A total upheaval of life as we know it — and social distance from the triggers that might push people to drink too… https://t.co/pTLrEUe2pY',
       'retweet': 2,
       'favCount': 8,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 57, 23),
       'screenname': '4ChrissyO',
       'id': 1248007055178227712,
       'text': '@FLOTUS Schisandra berries fight SARS Coronavirus 19 too. Look it up, they are good for many health and beauty issues, and harmless.',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 56, 18),
       'screenname': 'tea_natasha',
       'id': 1248006781520904192,
       'text': 'The simplicity of soothing Spearmint and lightly floral Rose Hip blend is a simplistic blend of balance and beauty.… https://t.co/EtbGIuqei1',
       'retweet': 2,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 54, 56),
       'screenname': 'Raul38557638',
       'id': 1248006438586167296,
       'text': 'RT @bay_art: It’s official, nature has wide-ranging health benefits, and also it is good for your soul. Famous nature quotes will make you…',
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 51, 36),
       'screenname': 'AALawnLandscape',
       'id': 1248005598072864768,
       'text': 'Easy to Follow Tips on Watering Your Lawn For Optimal Lawn Health, Growth, and Beauty! See Videos…\n\nLEARN MORE...… https://t.co/xz1t7SgJ3w',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 47, 6),
       'screenname': 'Beauty_Rushhhh',
       'id': 1248004465631752192,
       'text': 'They are giving us a health screening and checking our temperature everyday before we begin our shift . Shit is getting real lol',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 44, 23),
       'screenname': 'ah_marachy',
       'id': 1248003782664785921,
       'text': 'RT @KelechiAttamah: Law firms are already issuing opinions on covid-19 and employment contracts to their corporate clients. The beauty of l…',
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 40, 11),
       'screenname': 'tania_hanyu',
       'id': 1248002728082567168,
       'text': 'Price Blackview R6 3/32gb 5.5â€³ android 6 free jelly case and screen protector Cash On delivery Las Pinas Philiphines -  #Health &amp; Beauty',
       'retweet': 0,
       'favCount': 0,
       'tags': ['Health']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 39, 31),
       'screenname': '_ataei',
       'id': 1248002561090539521,
       'text': 'RT @rasuli_alireza: All people around the world are waiting for The Son of Man who brings a world without war, poverty, disease, or polluti…',
       'retweet': 7,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 35, 58),
       'screenname': 'ReGenFriends',
       'id': 1248001664994299905,
       'text': '“Upon this handful of soil, our survival depends. Husband it and it will grow our food, our fuel, and our shelter a… https://t.co/iDlp9Lu1nt',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 28, 24),
       'screenname': 'tmj_rip_adv',
       'id': 1247999760176254976,
       'text': "Want to work at CVS Health? We're hiring in #Woonsocket, RI! Click the link in our bio for details on this job and… https://t.co/GGbVymQGrR",
       'retweet': 0,
       'favCount': 0,
       'tags': ['Woonsocket']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 20),
       'screenname': 'kaelynforde',
       'id': 1247997648554135553,
       'text': '“Everything I use to hold myself accountable: work,  routine, exercise and self-care routines, it was all swept out… https://t.co/3T5k6NQeEK',
       'retweet': 0,
       'favCount': 4,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 14, 6),
       'screenname': 'ccthewriter1',
       'id': 1247996161639813125,
       'text': "RT @CrystalHoshaw: Hi #writingcommunity, I'm new to Twitter and looking to hire seasoned writers for health content. Please add me and shar…",
       'retweet': 15,
       'favCount': 0,
       'tags': ['writingcommunity']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 13, 22),
       'screenname': 'gd959098',
       'id': 1247995978147409920,
       'text': "RT @CrystalHoshaw: Hi #writingcommunity, I'm new to Twitter and looking to hire seasoned writers for health content. Please add me and shar…",
       'retweet': 15,
       'favCount': 0,
       'tags': ['writingcommunity']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 11, 52),
       'screenname': 'Crisp_Mag',
       'id': 1247995599544336385,
       'text': 'Why The Mind and Body Don’t Get Along During Quarantine https://t.co/xwhev2IL8w',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 11, 44),
       'screenname': 'Carols_Garcia',
       'id': 1247995565620723712,
       'text': 'SUPER EXCITED!!🥰 I joined the Arbonne fam when I found out that I could thrive doing something I’m already passiona… https://t.co/SloBzAEX78',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 7, 52),
       'screenname': 'CeciliaDaForno1',
       'id': 1247994595008483328,
       'text': 'What 3 things are you doing for your mental health? Answer / nominate 3 others: @JulieHe10511078  @And_I88… https://t.co/U26LvaS1Ff',
       'retweet': 0,
       'favCount': 2,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 5, 57),
       'screenname': 'BoBAUall',
       'id': 1247994111573078017,
       'text': 'Professional Titanium Microneedle Derma Needle Roller for Acne Scars,Face,B V3U9 https://t.co/4up18DwT2O https://t.co/XcSmAJfZZl',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 1, 46),
       'screenname': 'KatinaParon',
       'id': 1247993059683557380,
       'text': "RT @CrystalHoshaw: Hi #writingcommunity, I'm new to Twitter and looking to hire seasoned writers for health content. Please add me and shar…",
       'retweet': 15,
       'favCount': 0,
       'tags': ['writingcommunity']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 0, 43),
       'screenname': 'Greenhouses1',
       'id': 1247992793198465029,
       'text': 'The Garden is a place for the health of mind, body and soul.  ~ A place of Beauty, a place of celebration and appre… https://t.co/iuvDmJGHyw',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 0, 20),
       'screenname': 'ElleCanada',
       'id': 1247992699661189121,
       'text': 'At-Home Workout Equipment That Won’t Cost a Fortune https://t.co/NHH21nheWX',
       'retweet': 0,
       'favCount': 2,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 21, 0, 8),
       'screenname': 'ArnicareUSA',
       'id': 1247992647517700096,
       'text': '#Diet, #exercise, and #sleep all affect how you look. https://t.co/tuie8x4BZi',
       'retweet': 0,
       'favCount': 0,
       'tags': ['Diet', 'exercise', 'sleep']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 59, 15),
       'screenname': 'honkide',
       'id': 1247992423554281472,
       'text': 'RT @playbill: .@ABTBallet has canceled its 2020 spring season at the @MetOpera, including productions of Romeo and Juliet, Swan Lake, and T…',
       'retweet': 6,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 54, 56),
       'screenname': 'weischoice',
       'id': 1247991340752932865,
       'text': "RT @CrystalHoshaw: Hi #writingcommunity, I'm new to Twitter and looking to hire seasoned writers for health content. Please add me and shar…",
       'retweet': 15,
       'favCount': 0,
       'tags': ['writingcommunity']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 54, 31),
       'screenname': 'tomwlsn31',
       'id': 1247991233374535681,
       'text': 'RT @realTonyDudley: https://t.co/mAL9Jhecjl  Is one of the Best Beauty and Health Products I have ever run across 🙏👍🏻💯💕 #COVID19 #HealthyLi…',
       'retweet': 2,
       'favCount': 0,
       'tags': ['COVID19']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 52, 32),
       'screenname': 'BidorBuy247',
       'id': 1247990734906634245,
       'text': 'Beard Apron Cape Beard Trimming Bib for Men Shaving &amp; Hair Catcher, Non-Stick https://t.co/nL51gxkzQV https://t.co/lDioeDU6JR',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 50, 10),
       'screenname': 'PRJournoRequest',
       'id': 1247990140699643910,
       'text': "RT @RDeevoy: Looking for strong real life health and/or beauty stories for a women's mag. 35+ please! Get in touch if you've got something…",
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 49, 38),
       'screenname': 'Shahinaz6',
       'id': 1247990003579437063,
       'text': 'Guys tell me what do you use for your pets (health and care, food, beauty, everything)',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 49, 34),
       'screenname': 'ladyboarder9669',
       'id': 1247989989075324928,
       'text': 'RT @ladyboarder9669: Natural, easy, at home recipes to boost immuity for yourself and home during Covid-19 #covid19 #immunityboost #home #e…',
       'retweet': 3,
       'favCount': 0,
       'tags': ['covid19', 'immunityboost', 'home']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 48, 13),
       'screenname': 'se23_tweets',
       'id': 1247989648263127040,
       'text': 'RT @FHSoc: On your next health walk look up, refocus and see the beauty around us. Become a #TextureHunterGatherer like local artist @LizAt…',
       'retweet': 5,
       'favCount': 0,
       'tags': ['TextureHunterGatherer']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 47, 4),
       'screenname': 'JournalistJill',
       'id': 1247989360940777473,
       'text': 'RT @lynnettepeck: @JournalistJill @DailyMailUK I wrote about women and hair loss, and offered solutions, for @TheSun in the Nineties. They…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 45, 25),
       'screenname': 'seuoceancleanup',
       'id': 1247988945805287424,
       'text': 'The answer is true! #Microplastics are small plastic pieces less than five millimeters long which can be harmful to… https://t.co/qEg3tF0r01',
       'retweet': 0,
       'favCount': 0,
       'tags': ['Microplastics']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 43, 32),
       'screenname': 'Honeysblood',
       'id': 1247988470938812417,
       'text': "RT @RDeevoy: Looking for strong real life health and/or beauty stories for a women's mag. 35+ please! Get in touch if you've got something…",
       'retweet': 4,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 43, 11),
       'screenname': 'lynnettepeck',
       'id': 1247988382044733451,
       'text': '@JournalistJill @DailyMailUK I wrote about women and hair loss, and offered solutions, for @TheSun in the Nineties.… https://t.co/oF05S4pDkk',
       'retweet': 1,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 41, 3),
       'screenname': 'LizAtkin',
       'id': 1247987846473973762,
       'text': 'RT @FHSoc: On your next health walk look up, refocus and see the beauty around us. Become a #TextureHunterGatherer like local artist @LizAt…',
       'retweet': 5,
       'favCount': 0,
       'tags': ['TextureHunterGatherer']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 38, 42),
       'screenname': 'buynbuyshop',
       'id': 1247987253030408194,
       'text': '#health #clothing #gadget #beauty #watch \nUS$ 52.20 and FREE Shipping BOBO BIRD Men Wooden Mechanical Luminous Watc… https://t.co/xiu9GydX8Q',
       'retweet': 0,
       'favCount': 0,
       'tags': ['health', 'clothing', 'gadget', 'beauty', 'watch']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 35, 2),
       'screenname': 'SomethingDigitl',
       'id': 1247986330782531586,
       'text': 'SD knows the #beauty #health, &amp; #wellness sector. Some time ago we addressed challenges these brands face &amp; we thin… https://t.co/L5APKvTvlF',
       'retweet': 0,
       'favCount': 2,
       'tags': ['beauty', 'health', 'wellness']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 32, 15),
       'screenname': 'drjessigold',
       'id': 1247985629033639936,
       'text': 'RT @kaelynforde: "Having to face such uncertainty in such a raw way has been such a great learning and growth opportunity for me," @Lombard…',
       'retweet': 1,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 31, 35),
       'screenname': 'valiant_beauty',
       'id': 1247985461085298688,
       'text': 'RT @lindahoguttu: A friend has just been released from Mbagathi hospital 👏👏She is now free of #coronavirus\nSo People, this is a happy me an…',
       'retweet': 406,
       'favCount': 0,
       'tags': ['coronavirus']},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 27, 49),
       'screenname': 'biyounaruhodo',
       'id': 1247984515626045440,
       'text': 'ダイエットの正しい知識……\nhttps://t.co/vozL0d7g7l',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 23, 42),
       'screenname': 'AH_Fasihi',
       'id': 1247983477473107968,
       'text': 'O son of Hussein, savior of the world\nThe whole world is waiting for your divine rule, which is full of health, jus… https://t.co/EByv2dMrQ8',
       'retweet': 0,
       'favCount': 1,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 19, 15),
       'screenname': 'HummerSilver',
       'id': 1247982360940986368,
       'text': '8 Weird Things That Can Happen to Your Fingernails—and What They Say About Your Health https://t.co/HL77qN6au7 https://t.co/6kVJpRYccI',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 17, 43),
       'screenname': 'UtahFamilyPharm',
       'id': 1247981973043281922,
       'text': 'Hurricane Family Pharmacy knows that helping you feel better about your health and beauty is a good way to help you… https://t.co/invxk3ZNDU',
       'retweet': 0,
       'favCount': 0,
       'tags': ''},
      {'product_Line': 'Health and beauty',
       'created_at': datetime.datetime(2020, 4, 8, 20, 15, 8),
       'screenname': 'mdad8200',
       'id': 1247981321378500609,
       'text': '@RealWillMunny @FrankLuntz @DiamondandSilk The beauty of trolling is you can say ridiculous with no thought or conc… https://t.co/zCER9ryB2y',
       'retweet': 0,
       'favCount': 0,
       'tags': ''}]}



### Breaking The dataset into tables

### Branch Table


```python
#Branch Main Table
Branch = pd.DataFrame()
Branch[['Branch_Id','City']] = supermarket_data[['Branch','City']]
Branch = Branch.drop_duplicates()
Branch = Branch.reset_index(drop = True)
Branch.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Branch_Id</th>
      <th>City</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>Yangon</td>
    </tr>
    <tr>
      <th>1</th>
      <td>C</td>
      <td>Naypyitaw</td>
    </tr>
    <tr>
      <th>2</th>
      <td>B</td>
      <td>Mandalay</td>
    </tr>
  </tbody>
</table>
</div>



### Product Table


```python
#Product 
Product = pd.DataFrame()
Product[['Invoice ID','Product_Line','Unit_Price','Branch_Id']] = supermarket_data[['Invoice ID','Product line','Unit price','Branch']]
Product.index.name = "Product_Id"
Product.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Invoice ID</th>
      <th>Product_Line</th>
      <th>Unit_Price</th>
      <th>Branch_Id</th>
    </tr>
    <tr>
      <th>Product_Id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>750-67-8428</td>
      <td>Health and beauty</td>
      <td>74.69</td>
      <td>A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>226-31-3081</td>
      <td>Electronic accessories</td>
      <td>15.28</td>
      <td>C</td>
    </tr>
    <tr>
      <th>2</th>
      <td>631-41-3108</td>
      <td>Home and lifestyle</td>
      <td>46.33</td>
      <td>A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>123-19-1176</td>
      <td>Health and beauty</td>
      <td>58.22</td>
      <td>A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>373-73-7910</td>
      <td>Sports and travel</td>
      <td>86.31</td>
      <td>A</td>
    </tr>
  </tbody>
</table>
</div>



### ProductLine Table


```python
#Product Line
ProductLine = pd.DataFrame()
ProductLine[['Product_Line']] = supermarket_data[['Product line']]
ProductLine = ProductLine.drop_duplicates()
ProductLine = ProductLine.reset_index(drop = True)
ProductLine.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Product_Line</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Health and beauty</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Electronic accessories</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Home and lifestyle</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sports and travel</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Food and beverages</td>
    </tr>
  </tbody>
</table>
</div>



### Ratings Table


```python
#Ratings
Ratings = pd.DataFrame()
Ratings[['Product_Line','Customer_Rating','Branch_Id']] = supermarket_data[['Product line','Unit price','Branch']]
Ratings.index.name ="Ratings_Id"
Ratings.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Product_Line</th>
      <th>Customer_Rating</th>
      <th>Branch_Id</th>
    </tr>
    <tr>
      <th>Ratings_Id</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Health and beauty</td>
      <td>74.69</td>
      <td>A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Electronic accessories</td>
      <td>15.28</td>
      <td>C</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Home and lifestyle</td>
      <td>46.33</td>
      <td>A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Health and beauty</td>
      <td>58.22</td>
      <td>A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sports and travel</td>
      <td>86.31</td>
      <td>A</td>
    </tr>
  </tbody>
</table>
</div>



### Customer Table


```python
#Customer
Customer = pd.DataFrame()
#Customer[['First_Name','Last_name','PhoneNumber_1','PhoneNumber_2','Email','Gender','Customer_Type','Amount_Billed','Address','State','City']] = supermarket_data[['first_name','last_name','phone1','phone2','email','Gender','Customer type','Total','Address','STATE','CITY']]

Customer[['Invoice_Id','First_Name','Last_name','PhoneNumber_1','PhoneNumber_2','Email','Gender','Customer_Type','Amount_Billed','Address','STATE','CITY','Product_Line']] = supermarket_data[['Invoice ID','first_name','last_name','phone1','phone2','email','Gender','Customer type','Total','address','STATE','CITY','Product line']]
Customer.index.name ="Customer_Id"
Customer.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Invoice_Id</th>
      <th>First_Name</th>
      <th>Last_name</th>
      <th>PhoneNumber_1</th>
      <th>PhoneNumber_2</th>
      <th>Email</th>
      <th>Gender</th>
      <th>Customer_Type</th>
      <th>Amount_Billed</th>
      <th>Address</th>
      <th>STATE</th>
      <th>CITY</th>
      <th>Product_Line</th>
    </tr>
    <tr>
      <th>Customer_Id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>750-67-8428</td>
      <td>James</td>
      <td>Butt</td>
      <td>504-621-8927</td>
      <td>504-845-1427</td>
      <td>jbutt@gmail.com</td>
      <td>Female</td>
      <td>Member</td>
      <td>548.9715</td>
      <td>6649 N Blue Gum St</td>
      <td>LA</td>
      <td>New Orleans</td>
      <td>Health and beauty</td>
    </tr>
    <tr>
      <th>1</th>
      <td>226-31-3081</td>
      <td>Josephine</td>
      <td>Darakjy</td>
      <td>810-292-9388</td>
      <td>810-374-9840</td>
      <td>josephine_darakjy@darakjy.org</td>
      <td>Female</td>
      <td>Normal</td>
      <td>80.2200</td>
      <td>4 B Blue Ridge Blvd</td>
      <td>MI</td>
      <td>Brighton</td>
      <td>Electronic accessories</td>
    </tr>
    <tr>
      <th>2</th>
      <td>631-41-3108</td>
      <td>Art</td>
      <td>Venere</td>
      <td>856-636-8749</td>
      <td>856-264-4130</td>
      <td>art@venere.org</td>
      <td>Male</td>
      <td>Normal</td>
      <td>340.5255</td>
      <td>8 W Cerritos Ave #54</td>
      <td>NJ</td>
      <td>Bridgeport</td>
      <td>Home and lifestyle</td>
    </tr>
    <tr>
      <th>3</th>
      <td>123-19-1176</td>
      <td>Lenna</td>
      <td>Paprocki</td>
      <td>907-385-4412</td>
      <td>907-921-2010</td>
      <td>lpaprocki@hotmail.com</td>
      <td>Male</td>
      <td>Member</td>
      <td>489.0480</td>
      <td>639 Main St</td>
      <td>AK</td>
      <td>Anchorage</td>
      <td>Health and beauty</td>
    </tr>
    <tr>
      <th>4</th>
      <td>373-73-7910</td>
      <td>Donette</td>
      <td>Foller</td>
      <td>513-570-1893</td>
      <td>513-549-4561</td>
      <td>donette.foller@cox.net</td>
      <td>Male</td>
      <td>Normal</td>
      <td>634.3785</td>
      <td>34 Center St</td>
      <td>OH</td>
      <td>Hamilton</td>
      <td>Sports and travel</td>
    </tr>
  </tbody>
</table>
</div>



### Payment Table


```python
#Payment
Payment = pd.DataFrame()
Payment[['Invoice_Id','Total-Amount','Date_Of_Purchase','Time','Payment_Type']] = supermarket_data[['Invoice ID','Total','Date','Time','Payment']]
Payment.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Invoice_Id</th>
      <th>Total-Amount</th>
      <th>Date_Of_Purchase</th>
      <th>Time</th>
      <th>Payment_Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>750-67-8428</td>
      <td>548.9715</td>
      <td>1/5/19</td>
      <td>13:08</td>
      <td>Ewallet</td>
    </tr>
    <tr>
      <th>1</th>
      <td>226-31-3081</td>
      <td>80.2200</td>
      <td>3/8/19</td>
      <td>10:29</td>
      <td>Cash</td>
    </tr>
    <tr>
      <th>2</th>
      <td>631-41-3108</td>
      <td>340.5255</td>
      <td>3/3/19</td>
      <td>13:23</td>
      <td>Credit card</td>
    </tr>
    <tr>
      <th>3</th>
      <td>123-19-1176</td>
      <td>489.0480</td>
      <td>1/27/19</td>
      <td>20:33</td>
      <td>Ewallet</td>
    </tr>
    <tr>
      <th>4</th>
      <td>373-73-7910</td>
      <td>634.3785</td>
      <td>2/8/19</td>
      <td>10:37</td>
      <td>Ewallet</td>
    </tr>
  </tbody>
</table>
</div>



### Tax Table


```python
#Tax
Tax = pd.DataFrame()
Tax[['Tax','Payment_Invoice']] = supermarket_data[['Tax 5%','Invoice ID']]
Tax.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Tax</th>
      <th>Payment_Invoice</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>26.1415</td>
      <td>750-67-8428</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3.8200</td>
      <td>226-31-3081</td>
    </tr>
    <tr>
      <th>2</th>
      <td>16.2155</td>
      <td>631-41-3108</td>
    </tr>
    <tr>
      <th>3</th>
      <td>23.2880</td>
      <td>123-19-1176</td>
    </tr>
    <tr>
      <th>4</th>
      <td>30.2085</td>
      <td>373-73-7910</td>
    </tr>
  </tbody>
</table>
</div>



### State Table


```python
#State
State = pd.DataFrame()
State[['State']] = supermarket_data[['STATE']]
State = State.drop_duplicates()
State = State.reset_index(drop = True)
State.index.name = "State Id"
State.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>State</th>
    </tr>
    <tr>
      <th>State Id</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>LA</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MI</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NJ</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AK</td>
    </tr>
    <tr>
      <th>4</th>
      <td>OH</td>
    </tr>
  </tbody>
</table>
</div>



### Cost Of Goods Table


```python
#Cost of Goods Sold
COG = pd.DataFrame()
COG[['Total_Billed_Amount']] = supermarket_data[['Total']]
COG.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total_Billed_Amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>548.9715</td>
    </tr>
    <tr>
      <th>1</th>
      <td>80.2200</td>
    </tr>
    <tr>
      <th>2</th>
      <td>340.5255</td>
    </tr>
    <tr>
      <th>3</th>
      <td>489.0480</td>
    </tr>
    <tr>
      <th>4</th>
      <td>634.3785</td>
    </tr>
  </tbody>
</table>
</div>



### Gross Details Table


```python
#Gross Details
Gross_Details = pd.DataFrame()
#Gross_Details[['Cost_Of_Goods_Sold','Gross_Income','Gross_Margine_Percentage','Payment_Invoice_Id']] = supermarket_data[['cogs','gross income','gross margine percentage','Invoice ID']]
Gross_Details[['Cost_Of_Goods_Sold','Gross_Income','Gross_Margin_Percentage','Payment_Invoice_Id']] = supermarket_data[['cogs','gross income','gross margin percentage','Invoice ID']]
Gross_Details.index.name = "Gross_Details_Id"
Gross_Details.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Cost_Of_Goods_Sold</th>
      <th>Gross_Income</th>
      <th>Gross_Margin_Percentage</th>
      <th>Payment_Invoice_Id</th>
    </tr>
    <tr>
      <th>Gross_Details_Id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>522.83</td>
      <td>26.1415</td>
      <td>4.761905</td>
      <td>750-67-8428</td>
    </tr>
    <tr>
      <th>1</th>
      <td>76.40</td>
      <td>3.8200</td>
      <td>4.761905</td>
      <td>226-31-3081</td>
    </tr>
    <tr>
      <th>2</th>
      <td>324.31</td>
      <td>16.2155</td>
      <td>4.761905</td>
      <td>631-41-3108</td>
    </tr>
    <tr>
      <th>3</th>
      <td>465.76</td>
      <td>23.2880</td>
      <td>4.761905</td>
      <td>123-19-1176</td>
    </tr>
    <tr>
      <th>4</th>
      <td>604.17</td>
      <td>30.2085</td>
      <td>4.761905</td>
      <td>373-73-7910</td>
    </tr>
  </tbody>
</table>
</div>



# NoSQL Design explaination

We want to store the Data in a mongoDB database. In mongoDB, data is stored in the form of collections and documents. MongoDb stores the data in the form of Key value pairs.

##### MongoDB
MongoDB is an opensource document database. It is written in C++. MongoDB is cross platformm and provides high perfomance, high availability and easy scalability. MongoDB works on the concept of collection and document.
MongoDB doesnt use table and rows to store data but instead it uses collection of JSON like documents. Documents store embeded fields so that related data can be stored within them. It is schemaless so be do not need to specift the column for the database befor einserting document withing them.

##### Collections
It is a collection of documents and is equivalent to tables in RDBMS table. A collection exists within a single database.
documents within the collection can have documents with different fields however mostly all the documents in the collection are related to eachother. 

##### Documents
Documents have a set of key value pairs. Documents have Dynamic schema which means documents in the same schema do not have same fields or structure. Different fields in the collection document may hold different values. 




##### Design Decisions
The database is designed by considering the productline at the top level of the hierarchy. The ProductLine Database stores documents of every ProductLine. Each of these documents store information about all the customers who have purchased soemthing in the productline category and the tweets that are related to the productLine.

####  ProductLineDoc 
This collection has documents that are segerated according to the productlines.
The structure of each of these documents is - 
   ###### ProductLine: Name of the ProductLine
   ###### customerDictionary :Information about the customers,
   ###### ProductLine_Rating :ProductLine rating by customers
   ###### ProductTweet :Tweets associated with the productline
   
#### CustomerDictionary
This disctionary stores information about the customers who have purchased soemthing in the productLine caregory-
This distionary also contains payment details about the customers.
Things included in this dictionary are - 
'First_Name','Last_name','PhoneNumber_1','PhoneNumber_2','Email','Gender','Customer_Type','Amount_Billed','Address','STATE','CITY','Product_Line','Invoice_Id','Total_amount','Date_Of_Purchase','Time','Payment_Type'

#### ProductTweets
This dictionary stores the tweets that are related to the productline. This dictionary also stores all the hashtags that are associated with the tweet.
Information stored in this dictionary -
"product_Line","created_at","screenname","id","text","retweet","favCount","tags"



```python
ProductLineDoc = []
ProductLine_Dict = {}
ProductLineSalesDict = {}
customerDict = {}
TwitterPosts = {}
ProductLine_rating ={}
productTweet = []
Customer_Ratings = []
#Customer[['First_Name','Last_name','PhoneNumber_1','PhoneNumber_2','Email','Gender','Customer_Type','Amount_Billed','Address','STATE','CITY','Product_Line']] = supermarket_data[['first_name','last_name','phone1','phone2','email','Gender','Customer type','Total','address','STATE','CITY','Product line']]
FinalDictionary = []

for i in range(len(ProductLine)):
    for c in range(len(Customer)):
        #Store customer details and the details of their payments in JSON 
        if Customer["Product_Line"][c] == ProductLine["Product_Line"][i]:
            customerDict = {
                "FirstName":Customer["First_Name"][c],
                "LastName":Customer["Last_name"][c],
                "Address":Customer["Address"][c],
                "PhoneNumber1":Customer["PhoneNumber_1"][c],
                "PhoneNumber2":Customer["PhoneNumber_2"][c],
                "email":Customer["Email"][c],
                "gender":Customer["Gender"][c],
                "customer_type":Customer["Customer_Type"][c],
                "amount_billed":Customer["Amount_Billed"][c],
                "state":Customer["STATE"][c],
                "city":Customer["CITY"][c],
            }
            for  p in range(len(Payment)):
                if Payment["Invoice_Id"][p] == Customer["Invoice_Id"][c]:
                    customerDict["Total_amount"] = Payment["Total-Amount"][p]
                    customerDict["Date_Of_Purchase"] = Payment["Date_Of_Purchase"][p]
                    customerDict["Time"] = Payment["Time"][p]
                    customerDict["Payment_Type"] = Payment["Payment_Type"][p]
                    
    #Store the ratings for each of the Product Lines                
    for r in range(len(Ratings)):
        if Ratings["Product_Line"][r] == ProductLine["Product_Line"][i]:
            Customer_Ratings.append(Ratings["Customer_Rating"][r])
            
    productTweet = productLinesDictionary[ProductLine["Product_Line"][i]]
    mainDict = {
            "ProductLine":ProductLine["Product_Line"][i],
            "customerDictionary":customerDict,
            "ProductLine_Rating":ProductLine_rating,
            "productTweet":productTweet
             }
    ProductLineDoc.append(mainDict)
```

### The Size of the ProductLine Document


```python
len(ProductLineDoc)
```




    6



### Storing the collection created in MongoDB


```python
client = MongoClient('localhost', 27017)
```


```python
print(client.list_database_names())
```

    ['MEANStackDB', 'MoviesDB', 'PleaseDB', 'ProductLine_Database', 'UserdataDB', 'admin', 'config', 'customers', 'local', 'mylib', 'nodeauth', 'userdb']
    

### Deleting the database if it already exists


```python
client["ProductLine_Database"]['ProductLine'].drop()
```

### Checking if the database is deleted


```python
print(client.list_database_names())
```

    ['MEANStackDB', 'MoviesDB', 'PleaseDB', 'UserdataDB', 'admin', 'config', 'customers', 'local', 'mylib', 'nodeauth', 'userdb']
    

### Creating the database -> ProductLine_Database and the database collection ->ProductLine


```python
mydatabase = client["ProductLine_Database"]
mycol = mydatabase["ProductLine"]
x = mycol.insert_many(ProductLineDoc)
print(client.list_database_names())
```

    ['MEANStackDB', 'MoviesDB', 'PleaseDB', 'ProductLine_Database', 'UserdataDB', 'admin', 'config', 'customers', 'local', 'mylib', 'nodeauth', 'userdb']
    

### Retriving all the data stored in MongoDB


```python
for pl in mycol.find({}):
    pprint.pprint(pl)
```

    {'ProductLine': 'Health and beauty',
     'ProductLine_Rating': {},
     '_id': ObjectId('5e8e5532b5101d7d130493c7'),
     'customerDictionary': {'Address': '9166 Devon St #905',
                            'Date_Of_Purchase': '1/29/19',
                            'FirstName': 'Avery',
                            'LastName': 'Veit',
                            'Payment_Type': 'Ewallet',
                            'PhoneNumber1': '01748-625058',
                            'PhoneNumber2': '01369-185737',
                            'Time': '13:46',
                            'Total_amount': 42.3675,
                            'amount_billed': 42.3675,
                            'city': 'Boise',
                            'customer_type': 'Normal',
                            'email': 'avery@veit.co.uk',
                            'gender': 'Male',
                            'state': 'ID'},
     'productTweet': [{'created_at': datetime.datetime(2020, 4, 8, 22, 47, 58),
                       'favCount': 0,
                       'id': 1248019784039411712,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'GrayskyX',
                       'tags': '',
                       'text': 'Those who love themselves, by wealth, beauty, or '
                               '"health" will limit life to themselves, and be '
                               'complacent and call… https://t.co/SPCBWffDlY'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 47, 24),
                       'favCount': 0,
                       'id': 1248019644289445890,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'velocipedus',
                       'tags': '',
                       'text': '@allpartycycling @bmj_latest The beauty and power '
                               'of such studies: They provide hard evidence to '
                               'bring together hea… https://t.co/rRRXrObiLL'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 47, 8),
                       'favCount': 0,
                       'id': 1248019576043917312,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'mvsiii71',
                       'tags': '',
                       'text': 'Come take a look at all Avon has to offer for '
                               'health and beauty needs! https://t.co/qBqQCTkWuw '
                               'https://t.co/rqm5chgzJl'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 39, 46),
                       'favCount': 0,
                       'id': 1248017721532416003,
                       'product_Line': 'Health and beauty',
                       'retweet': 1,
                       'screenname': 'Treuburg',
                       'tags': '',
                       'text': 'RT @JackStewartCham: Hundreds move into small, run '
                               'down or boring town and revitalize infrastructure, '
                               'display health and vitality and inspi…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 34, 51),
                       'favCount': 3,
                       'id': 1248016482690199555,
                       'product_Line': 'Health and beauty',
                       'retweet': 1,
                       'screenname': 'JackStewartCham',
                       'tags': '',
                       'text': 'Hundreds move into small, run down or boring town '
                               'and revitalize infrastructure, display health and '
                               'vitality and in… https://t.co/NZitR9TqAv'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 30, 54),
                       'favCount': 0,
                       'id': 1248015490057822208,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'dawn27331807',
                       'tags': '',
                       'text': '@realDonaldTrump I work in the beauty industry and '
                               'I touch people all day ... I’m not in any rush to '
                               'go to work...… https://t.co/mvjjWJ2Cnu'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 30, 43),
                       'favCount': 0,
                       'id': 1248015444365066243,
                       'product_Line': 'Health and beauty',
                       'retweet': 2,
                       'screenname': 'TheVibeDealer3',
                       'tags': '',
                       'text': 'RT @tea_natasha: The simplicity of soothing '
                               'Spearmint and lightly floral Rose Hip blend is a '
                               'simplistic blend of balance and beauty. Enjoy…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 27, 7),
                       'favCount': 0,
                       'id': 1248014536809967616,
                       'product_Line': 'Health and beauty',
                       'retweet': 3,
                       'screenname': 'HollyThomas61',
                       'tags': ['covid19', 'immunityboost', 'home'],
                       'text': 'RT @ladyboarder9669: Natural, easy, at home '
                               'recipes to boost immuity for yourself and home '
                               'during Covid-19 #covid19 #immunityboost #home #e…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 12, 42),
                       'favCount': 0,
                       'id': 1248010909819142144,
                       'product_Line': 'Health and beauty',
                       'retweet': 15,
                       'screenname': 'EmanHabbash',
                       'tags': '',
                       'text': 'RT @HeightsPodium: It starts from health all the '
                               'way to media.. we will go through sports, culture, '
                               'innovation, coffee, Comedy, beauty, art…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 10, 19),
                       'favCount': 0,
                       'id': 1248010309840719873,
                       'product_Line': 'Health and beauty',
                       'retweet': 4,
                       'screenname': 'ewhoma_',
                       'tags': '',
                       'text': 'RT @KelechiAttamah: Law firms are already issuing '
                               'opinions on covid-19 and employment contracts to '
                               'their corporate clients. The beauty of l…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 9, 10),
                       'favCount': 0,
                       'id': 1248010022266630147,
                       'product_Line': 'Health and beauty',
                       'retweet': 4,
                       'screenname': 'USIIV',
                       'tags': '',
                       'text': 'RT @KelechiAttamah: Law firms are already issuing '
                               'opinions on covid-19 and employment contracts to '
                               'their corporate clients. The beauty of l…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 8, 5),
                       'favCount': 0,
                       'id': 1248009747158073344,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'BidorBuy247',
                       'tags': '',
                       'text': 'Remedy Antifungal Soap, Helps Wash Away Body Odor '
                               'Fungus Shower Gel Women/Men '
                               'https://t.co/OqZRPaDV6n https://t.co/NRKUMVteJ8'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 2, 50),
                       'favCount': 0,
                       'id': 1248008428703731718,
                       'product_Line': 'Health and beauty',
                       'retweet': 1,
                       'screenname': 'Its_3One4',
                       'tags': '',
                       'text': 'RT @Ali_yeganeh_s: The twelfth Imam, Imam Mahdi, '
                               'is not only the Imam of the Shiites, but also the '
                               'Imam of all human beings With his coming…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 1, 43),
                       'favCount': 1,
                       'id': 1248008144229285893,
                       'product_Line': 'Health and beauty',
                       'retweet': 1,
                       'screenname': 'Ali_yeganeh_s',
                       'tags': '',
                       'text': 'The twelfth Imam, Imam Mahdi, is not only the Imam '
                               'of the Shiites, but also the Imam of all human '
                               'beings With his c… https://t.co/TVN4l6XP17'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 0, 8),
                       'favCount': 0,
                       'id': 1248007746009493504,
                       'product_Line': 'Health and beauty',
                       'retweet': 2,
                       'screenname': 'Gentleman__7',
                       'tags': '',
                       'text': 'RT @tea_natasha: The simplicity of soothing '
                               'Spearmint and lightly floral Rose Hip blend is a '
                               'simplistic blend of balance and beauty. Enjoy…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 59, 11),
                       'favCount': 0,
                       'id': 1248007509400379393,
                       'product_Line': 'Health and beauty',
                       'retweet': 2,
                       'screenname': 'TraceyBenmore',
                       'tags': '',
                       'text': 'RT @InStyle: A total upheaval of life as we know '
                               'it — and social distance from the triggers that '
                               'might push people to drink too much — is l…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 57, 34),
                       'favCount': 8,
                       'id': 1248007100040531971,
                       'product_Line': 'Health and beauty',
                       'retweet': 2,
                       'screenname': 'InStyle',
                       'tags': '',
                       'text': 'A total upheaval of life as we know it — and '
                               'social distance from the triggers that might push '
                               'people to drink too… https://t.co/pTLrEUe2pY'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 57, 23),
                       'favCount': 0,
                       'id': 1248007055178227712,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': '4ChrissyO',
                       'tags': '',
                       'text': '@FLOTUS Schisandra berries fight SARS Coronavirus '
                               '19 too. Look it up, they are good for many health '
                               'and beauty issues, and harmless.'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 56, 18),
                       'favCount': 1,
                       'id': 1248006781520904192,
                       'product_Line': 'Health and beauty',
                       'retweet': 2,
                       'screenname': 'tea_natasha',
                       'tags': '',
                       'text': 'The simplicity of soothing Spearmint and lightly '
                               'floral Rose Hip blend is a simplistic blend of '
                               'balance and beauty.… https://t.co/EtbGIuqei1'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 54, 56),
                       'favCount': 0,
                       'id': 1248006438586167296,
                       'product_Line': 'Health and beauty',
                       'retweet': 4,
                       'screenname': 'Raul38557638',
                       'tags': '',
                       'text': 'RT @bay_art: It’s official, nature has '
                               'wide-ranging health benefits, and also it is good '
                               'for your soul. Famous nature quotes will make '
                               'you…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 51, 36),
                       'favCount': 0,
                       'id': 1248005598072864768,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'AALawnLandscape',
                       'tags': '',
                       'text': 'Easy to Follow Tips on Watering Your Lawn For '
                               'Optimal Lawn Health, Growth, and Beauty! See '
                               'Videos…\n'
                               '\n'
                               'LEARN MORE...… https://t.co/xz1t7SgJ3w'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 47, 6),
                       'favCount': 0,
                       'id': 1248004465631752192,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'Beauty_Rushhhh',
                       'tags': '',
                       'text': 'They are giving us a health screening and checking '
                               'our temperature everyday before we begin our shift '
                               '. Shit is getting real lol'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 44, 23),
                       'favCount': 0,
                       'id': 1248003782664785921,
                       'product_Line': 'Health and beauty',
                       'retweet': 4,
                       'screenname': 'ah_marachy',
                       'tags': '',
                       'text': 'RT @KelechiAttamah: Law firms are already issuing '
                               'opinions on covid-19 and employment contracts to '
                               'their corporate clients. The beauty of l…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 40, 11),
                       'favCount': 0,
                       'id': 1248002728082567168,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'tania_hanyu',
                       'tags': ['Health'],
                       'text': 'Price Blackview R6 3/32gb 5.5â€³ android 6 free '
                               'jelly case and screen protector Cash On delivery '
                               'Las Pinas Philiphines -  #Health &amp; Beauty'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 39, 31),
                       'favCount': 0,
                       'id': 1248002561090539521,
                       'product_Line': 'Health and beauty',
                       'retweet': 7,
                       'screenname': '_ataei',
                       'tags': '',
                       'text': 'RT @rasuli_alireza: All people around the world '
                               'are waiting for The Son of Man who brings a world '
                               'without war, poverty, disease, or polluti…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 35, 58),
                       'favCount': 0,
                       'id': 1248001664994299905,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'ReGenFriends',
                       'tags': '',
                       'text': '“Upon this handful of soil, our survival depends. '
                               'Husband it and it will grow our food, our fuel, '
                               'and our shelter a… https://t.co/iDlp9Lu1nt'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 28, 24),
                       'favCount': 0,
                       'id': 1247999760176254976,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'tmj_rip_adv',
                       'tags': ['Woonsocket'],
                       'text': "Want to work at CVS Health? We're hiring in "
                               '#Woonsocket, RI! Click the link in our bio for '
                               'details on this job and… https://t.co/GGbVymQGrR'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 20),
                       'favCount': 4,
                       'id': 1247997648554135553,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'kaelynforde',
                       'tags': '',
                       'text': '“Everything I use to hold myself accountable: '
                               'work,  routine, exercise and self-care routines, '
                               'it was all swept out… https://t.co/3T5k6NQeEK'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 14, 6),
                       'favCount': 0,
                       'id': 1247996161639813125,
                       'product_Line': 'Health and beauty',
                       'retweet': 15,
                       'screenname': 'ccthewriter1',
                       'tags': ['writingcommunity'],
                       'text': "RT @CrystalHoshaw: Hi #writingcommunity, I'm new "
                               'to Twitter and looking to hire seasoned writers '
                               'for health content. Please add me and shar…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 13, 22),
                       'favCount': 0,
                       'id': 1247995978147409920,
                       'product_Line': 'Health and beauty',
                       'retweet': 15,
                       'screenname': 'gd959098',
                       'tags': ['writingcommunity'],
                       'text': "RT @CrystalHoshaw: Hi #writingcommunity, I'm new "
                               'to Twitter and looking to hire seasoned writers '
                               'for health content. Please add me and shar…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 11, 52),
                       'favCount': 1,
                       'id': 1247995599544336385,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'Crisp_Mag',
                       'tags': '',
                       'text': 'Why The Mind and Body Don’t Get Along During '
                               'Quarantine https://t.co/xwhev2IL8w'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 11, 44),
                       'favCount': 1,
                       'id': 1247995565620723712,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'Carols_Garcia',
                       'tags': '',
                       'text': 'SUPER EXCITED!!🥰 I joined the Arbonne fam when I '
                               'found out that I could thrive doing something I’m '
                               'already passiona… https://t.co/SloBzAEX78'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 7, 52),
                       'favCount': 2,
                       'id': 1247994595008483328,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'CeciliaDaForno1',
                       'tags': '',
                       'text': 'What 3 things are you doing for your mental '
                               'health? Answer / nominate 3 others: '
                               '@JulieHe10511078  @And_I88… '
                               'https://t.co/U26LvaS1Ff'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 5, 57),
                       'favCount': 0,
                       'id': 1247994111573078017,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'BoBAUall',
                       'tags': '',
                       'text': 'Professional Titanium Microneedle Derma Needle '
                               'Roller for Acne Scars,Face,B V3U9 '
                               'https://t.co/4up18DwT2O https://t.co/XcSmAJfZZl'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 1, 46),
                       'favCount': 0,
                       'id': 1247993059683557380,
                       'product_Line': 'Health and beauty',
                       'retweet': 15,
                       'screenname': 'KatinaParon',
                       'tags': ['writingcommunity'],
                       'text': "RT @CrystalHoshaw: Hi #writingcommunity, I'm new "
                               'to Twitter and looking to hire seasoned writers '
                               'for health content. Please add me and shar…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 0, 43),
                       'favCount': 0,
                       'id': 1247992793198465029,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'Greenhouses1',
                       'tags': '',
                       'text': 'The Garden is a place for the health of mind, body '
                               'and soul.  ~ A place of Beauty, a place of '
                               'celebration and appre… https://t.co/iuvDmJGHyw'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 0, 20),
                       'favCount': 2,
                       'id': 1247992699661189121,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'ElleCanada',
                       'tags': '',
                       'text': 'At-Home Workout Equipment That Won’t Cost a '
                               'Fortune https://t.co/NHH21nheWX'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 0, 8),
                       'favCount': 0,
                       'id': 1247992647517700096,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'ArnicareUSA',
                       'tags': ['Diet', 'exercise', 'sleep'],
                       'text': '#Diet, #exercise, and #sleep all affect how you '
                               'look. https://t.co/tuie8x4BZi'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 59, 15),
                       'favCount': 0,
                       'id': 1247992423554281472,
                       'product_Line': 'Health and beauty',
                       'retweet': 6,
                       'screenname': 'honkide',
                       'tags': '',
                       'text': 'RT @playbill: .@ABTBallet has canceled its 2020 '
                               'spring season at the @MetOpera, including '
                               'productions of Romeo and Juliet, Swan Lake, and '
                               'T…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 54, 56),
                       'favCount': 0,
                       'id': 1247991340752932865,
                       'product_Line': 'Health and beauty',
                       'retweet': 15,
                       'screenname': 'weischoice',
                       'tags': ['writingcommunity'],
                       'text': "RT @CrystalHoshaw: Hi #writingcommunity, I'm new "
                               'to Twitter and looking to hire seasoned writers '
                               'for health content. Please add me and shar…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 54, 31),
                       'favCount': 0,
                       'id': 1247991233374535681,
                       'product_Line': 'Health and beauty',
                       'retweet': 2,
                       'screenname': 'tomwlsn31',
                       'tags': ['COVID19'],
                       'text': 'RT @realTonyDudley: https://t.co/mAL9Jhecjl  Is '
                               'one of the Best Beauty and Health Products I have '
                               'ever run across 🙏👍🏻💯💕 #COVID19 #HealthyLi…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 52, 32),
                       'favCount': 0,
                       'id': 1247990734906634245,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'BidorBuy247',
                       'tags': '',
                       'text': 'Beard Apron Cape Beard Trimming Bib for Men '
                               'Shaving &amp; Hair Catcher, Non-Stick '
                               'https://t.co/nL51gxkzQV https://t.co/lDioeDU6JR'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 50, 10),
                       'favCount': 0,
                       'id': 1247990140699643910,
                       'product_Line': 'Health and beauty',
                       'retweet': 4,
                       'screenname': 'PRJournoRequest',
                       'tags': '',
                       'text': 'RT @RDeevoy: Looking for strong real life health '
                               "and/or beauty stories for a women's mag. 35+ "
                               "please! Get in touch if you've got something…"},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 49, 38),
                       'favCount': 0,
                       'id': 1247990003579437063,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'Shahinaz6',
                       'tags': '',
                       'text': 'Guys tell me what do you use for your pets (health '
                               'and care, food, beauty, everything)'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 49, 34),
                       'favCount': 0,
                       'id': 1247989989075324928,
                       'product_Line': 'Health and beauty',
                       'retweet': 3,
                       'screenname': 'ladyboarder9669',
                       'tags': ['covid19', 'immunityboost', 'home'],
                       'text': 'RT @ladyboarder9669: Natural, easy, at home '
                               'recipes to boost immuity for yourself and home '
                               'during Covid-19 #covid19 #immunityboost #home #e…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 48, 13),
                       'favCount': 0,
                       'id': 1247989648263127040,
                       'product_Line': 'Health and beauty',
                       'retweet': 5,
                       'screenname': 'se23_tweets',
                       'tags': ['TextureHunterGatherer'],
                       'text': 'RT @FHSoc: On your next health walk look up, '
                               'refocus and see the beauty around us. Become a '
                               '#TextureHunterGatherer like local artist @LizAt…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 47, 4),
                       'favCount': 0,
                       'id': 1247989360940777473,
                       'product_Line': 'Health and beauty',
                       'retweet': 1,
                       'screenname': 'JournalistJill',
                       'tags': '',
                       'text': 'RT @lynnettepeck: @JournalistJill @DailyMailUK I '
                               'wrote about women and hair loss, and offered '
                               'solutions, for @TheSun in the Nineties. They…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 45, 25),
                       'favCount': 0,
                       'id': 1247988945805287424,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'seuoceancleanup',
                       'tags': ['Microplastics'],
                       'text': 'The answer is true! #Microplastics are small '
                               'plastic pieces less than five millimeters long '
                               'which can be harmful to… https://t.co/qEg3tF0r01'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 43, 32),
                       'favCount': 0,
                       'id': 1247988470938812417,
                       'product_Line': 'Health and beauty',
                       'retweet': 4,
                       'screenname': 'Honeysblood',
                       'tags': '',
                       'text': 'RT @RDeevoy: Looking for strong real life health '
                               "and/or beauty stories for a women's mag. 35+ "
                               "please! Get in touch if you've got something…"},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 43, 11),
                       'favCount': 1,
                       'id': 1247988382044733451,
                       'product_Line': 'Health and beauty',
                       'retweet': 1,
                       'screenname': 'lynnettepeck',
                       'tags': '',
                       'text': '@JournalistJill @DailyMailUK I wrote about women '
                               'and hair loss, and offered solutions, for @TheSun '
                               'in the Nineties.… https://t.co/oF05S4pDkk'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 41, 3),
                       'favCount': 0,
                       'id': 1247987846473973762,
                       'product_Line': 'Health and beauty',
                       'retweet': 5,
                       'screenname': 'LizAtkin',
                       'tags': ['TextureHunterGatherer'],
                       'text': 'RT @FHSoc: On your next health walk look up, '
                               'refocus and see the beauty around us. Become a '
                               '#TextureHunterGatherer like local artist @LizAt…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 38, 42),
                       'favCount': 0,
                       'id': 1247987253030408194,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'buynbuyshop',
                       'tags': ['health', 'clothing', 'gadget', 'beauty', 'watch'],
                       'text': '#health #clothing #gadget #beauty #watch \n'
                               'US$ 52.20 and FREE Shipping BOBO BIRD Men Wooden '
                               'Mechanical Luminous Watc… https://t.co/xiu9GydX8Q'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 35, 2),
                       'favCount': 2,
                       'id': 1247986330782531586,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'SomethingDigitl',
                       'tags': ['beauty', 'health', 'wellness'],
                       'text': 'SD knows the #beauty #health, &amp; #wellness '
                               'sector. Some time ago we addressed challenges '
                               'these brands face &amp; we thin… '
                               'https://t.co/L5APKvTvlF'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 32, 15),
                       'favCount': 0,
                       'id': 1247985629033639936,
                       'product_Line': 'Health and beauty',
                       'retweet': 1,
                       'screenname': 'drjessigold',
                       'tags': '',
                       'text': 'RT @kaelynforde: "Having to face such uncertainty '
                               'in such a raw way has been such a great learning '
                               'and growth opportunity for me," @Lombard…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 31, 35),
                       'favCount': 0,
                       'id': 1247985461085298688,
                       'product_Line': 'Health and beauty',
                       'retweet': 406,
                       'screenname': 'valiant_beauty',
                       'tags': ['coronavirus'],
                       'text': 'RT @lindahoguttu: A friend has just been released '
                               'from Mbagathi hospital 👏👏She is now free of '
                               '#coronavirus\n'
                               'So People, this is a happy me an…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 27, 49),
                       'favCount': 0,
                       'id': 1247984515626045440,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'biyounaruhodo',
                       'tags': '',
                       'text': 'ダイエットの正しい知識……\nhttps://t.co/vozL0d7g7l'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 23, 42),
                       'favCount': 1,
                       'id': 1247983477473107968,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'AH_Fasihi',
                       'tags': '',
                       'text': 'O son of Hussein, savior of the world\n'
                               'The whole world is waiting for your divine rule, '
                               'which is full of health, jus… '
                               'https://t.co/EByv2dMrQ8'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 19, 15),
                       'favCount': 0,
                       'id': 1247982360940986368,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'HummerSilver',
                       'tags': '',
                       'text': '8 Weird Things That Can Happen to Your '
                               'Fingernails—and What They Say About Your Health '
                               'https://t.co/HL77qN6au7 https://t.co/6kVJpRYccI'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 17, 43),
                       'favCount': 0,
                       'id': 1247981973043281922,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'UtahFamilyPharm',
                       'tags': '',
                       'text': 'Hurricane Family Pharmacy knows that helping you '
                               'feel better about your health and beauty is a good '
                               'way to help you… https://t.co/invxk3ZNDU'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 15, 8),
                       'favCount': 0,
                       'id': 1247981321378500609,
                       'product_Line': 'Health and beauty',
                       'retweet': 0,
                       'screenname': 'mdad8200',
                       'tags': '',
                       'text': '@RealWillMunny @FrankLuntz @DiamondandSilk The '
                               'beauty of trolling is you can say ridiculous with '
                               'no thought or conc… https://t.co/zCER9ryB2y'}]}
    {'ProductLine': 'Electronic accessories',
     'ProductLine_Rating': {},
     '_id': ObjectId('5e8e5532b5101d7d130493c8'),
     'customerDictionary': {'Address': '4 Covent Garden',
                            'Date_Of_Purchase': '2/18/19',
                            'FirstName': 'Alesia',
                            'LastName': 'Katie',
                            'Payment_Type': 'Ewallet',
                            'PhoneNumber1': '01333-436799',
                            'PhoneNumber2': '01240-614527',
                            'Time': '11:40',
                            'Total_amount': 63.9975,
                            'amount_billed': 63.9975,
                            'city': 'Novato',
                            'customer_type': 'Member',
                            'email': 'alesia_katie@gmail.com',
                            'gender': 'Female',
                            'state': 'CA'},
     'productTweet': [{'created_at': datetime.datetime(2020, 4, 8, 22, 28, 41),
                       'favCount': 0,
                       'id': 1248014932177641473,
                       'product_Line': 'Electronic accessories',
                       'retweet': 0,
                       'screenname': 'TicAccessories',
                       'tags': '',
                       'text': 'TIC Accessories LTD.\n'
                               '167th Street, Jamaica, New York 11432, USA.\n'
                               'TIC Accessories LTD is a well-known Electronic '
                               'pro… https://t.co/DINrF5U25t'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 26, 28),
                       'favCount': 0,
                       'id': 1247969073922748420,
                       'product_Line': 'Electronic accessories',
                       'retweet': 92,
                       'screenname': 'GhanaSocialU',
                       'tags': '',
                       'text': 'RT @IzaicKumy: Get all ur electronic devices :\n'
                               'iPhones 📱 , MacBook 💻, Ps4 , Samsung phones 📲, '
                               'laptop 💻 at  affordable prices . \n'
                               'Accessories…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 2, 16),
                       'favCount': 4,
                       'id': 1247962986922246147,
                       'product_Line': 'Electronic accessories',
                       'retweet': 0,
                       'screenname': 'MightyInkMatt',
                       'tags': '',
                       'text': 'I really wish I could validate this purchase right '
                               "now, but here's that fancy Black Panther helmet "
                               'for $54.00 (free… https://t.co/g3y9a6pf3G'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 38, 4),
                       'favCount': 0,
                       'id': 1247956894209228801,
                       'product_Line': 'Electronic accessories',
                       'retweet': 1,
                       'screenname': 'digitobacco',
                       'tags': '',
                       'text': 'RT @perfect_fold: Idea 💡 201:\n'
                               '\n'
                               'PLEASE PASS TO ALL VAPE MAKERS\n'
                               'VAPE ACCESSORIES\n'
                               '\n'
                               '©️\n'
                               'Vape Access;\n'
                               'VPN on electronic version.\n'
                               'Digital clock.\n'
                               'P…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 35, 17),
                       'favCount': 0,
                       'id': 1247956197019443201,
                       'product_Line': 'Electronic accessories',
                       'retweet': 1,
                       'screenname': 'perfect_fold',
                       'tags': '',
                       'text': 'Idea 💡 201:\n'
                               '\n'
                               'PLEASE PASS TO ALL VAPE MAKERS\n'
                               'VAPE ACCESSORIES\n'
                               '\n'
                               '©️\n'
                               'Vape Access;\n'
                               'VPN on electronic version.\n'
                               'Digital cl… https://t.co/2Qlwnjb1pU'},
                      {'created_at': datetime.datetime(2020, 4, 8, 11, 37, 42),
                       'favCount': 0,
                       'id': 1247851106232864768,
                       'product_Line': 'Electronic accessories',
                       'retweet': 0,
                       'screenname': 'WheelersGifts',
                       'tags': ['montblanc'],
                       'text': 'MB 01 - Keep your eyes peeled because we will soon '
                               'have them in stock 👀🎧🎶 https://t.co/uC6Ickik4V\n'
                               '\n'
                               '-\n'
                               '\n'
                               '#montblanc… https://t.co/qinAyGtN0E'},
                      {'created_at': datetime.datetime(2020, 4, 8, 10, 46, 31),
                       'favCount': 0,
                       'id': 1247838227790286849,
                       'product_Line': 'Electronic accessories',
                       'retweet': 0,
                       'screenname': 'OShop545',
                       'tags': ['electronic_accessories',
                                'phone_case',
                                'phone_glass',
                                'phone_cables',
                                'phone_chargersSpring'],
                       'text': '#electronic_accessories #phone_case #phone_glass '
                               '#phone_cables #phone_chargersSpring USB Cable for '
                               'iPhone Charger F… https://t.co/37SYV4uGnR'},
                      {'created_at': datetime.datetime(2020, 4, 8, 3, 33, 42),
                       'favCount': 0,
                       'id': 1247729303476994048,
                       'product_Line': 'Electronic accessories',
                       'retweet': 0,
                       'screenname': 'StoreZentrum',
                       'tags': ['hashtag3'],
                       'text': '#hashtag3 Spacious Bag for MacBook Laptops and '
                               'Electronic Accessories https://t.co/Ff8VR9QJ7d '
                               'https://t.co/6zyOK3aZA8'},
                      {'created_at': datetime.datetime(2020, 4, 8, 2, 27, 35),
                       'favCount': 0,
                       'id': 1247712666422042624,
                       'product_Line': 'Electronic accessories',
                       'retweet': 0,
                       'screenname': 'AmroleL',
                       'tags': ['Gaming'],
                       'text': 'https://t.co/q4qQgEAvCH\n'
                               'KEYCEO is the professional manufacturer for '
                               'researching and developing #Gaming peripherals… '
                               'https://t.co/d0BdImYSVb'},
                      {'created_at': datetime.datetime(2020, 4, 8, 2, 10, 24),
                       'favCount': 0,
                       'id': 1247708342039883776,
                       'product_Line': 'Electronic accessories',
                       'retweet': 0,
                       'screenname': 'mkrishnasundar',
                       'tags': '',
                       'text': '@amazonIN Can you pls enable laptop accessories '
                               'for purchase/delivery? During lockdown to continue '
                               'WFH some electro… https://t.co/BRrfEULtKP'}]}
    {'ProductLine': 'Home and lifestyle',
     'ProductLine_Rating': {},
     '_id': ObjectId('5e8e5532b5101d7d130493c9'),
     'customerDictionary': {'Address': '9 Milton St',
                            'Date_Of_Purchase': '2/22/19',
                            'FirstName': 'Celestina',
                            'LastName': 'Keeny',
                            'Payment_Type': 'Cash',
                            'PhoneNumber1': '01877-379681',
                            'PhoneNumber2': '01600-463475',
                            'Time': '15:33',
                            'Total_amount': 69.111,
                            'amount_billed': 69.111,
                            'city': 'Seattle',
                            'customer_type': 'Normal',
                            'email': 'celestina_keeny@gmail.com',
                            'gender': 'Male',
                            'state': 'WA'},
     'productTweet': [{'created_at': datetime.datetime(2020, 4, 8, 22, 49, 20),
                       'favCount': 0,
                       'id': 1248020129809444865,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'OlgaSixta',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 46, 32),
                       'favCount': 0,
                       'id': 1248019424390635521,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'emilyyy307',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 46, 13),
                       'favCount': 0,
                       'id': 1248019344962883584,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 13,
                       'screenname': '4Everanimalz1',
                       'tags': '',
                       'text': 'RT @SundayTimesZA: Mzansi has been baking up such '
                               "a storm while stuck at home that there's a now a "
                               'shortage of yeast on supermarket shelves…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 43, 21),
                       'favCount': 0,
                       'id': 1248018623169302528,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'ShellieRaygoza',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 41, 45),
                       'favCount': 0,
                       'id': 1248018219471736832,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'emo__420',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 40, 15),
                       'favCount': 0,
                       'id': 1248017841191653377,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'mattfromCal1',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 40, 13),
                       'favCount': 0,
                       'id': 1248017834476580864,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'gornelas20',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 39, 23),
                       'favCount': 0,
                       'id': 1248017626686574592,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'ChryslerLee',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 39, 17),
                       'favCount': 0,
                       'id': 1248017598295339008,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'MSNNZ',
                       'tags': '',
                       'text': "Harry, Meghan 'plan to build home in the "
                               "Cotswolds' https://t.co/RwnS5dg9u4"},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 38, 35),
                       'favCount': 0,
                       'id': 1248017422759555074,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': '15Stephen15',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 37, 40),
                       'favCount': 0,
                       'id': 1248017193733771265,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 156,
                       'screenname': 'sally06301',
                       'tags': '',
                       'text': 'RT @MNMLCase: Minimalists aren’t opposed to '
                               'spending money. However, we are generally against '
                               'spending too much of it without a darn good r…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 37, 31),
                       'favCount': 0,
                       'id': 1248017156719009793,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'dianexav1',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 36, 11),
                       'favCount': 0,
                       'id': 1248016817685061633,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 1,
                       'screenname': 'itsHemantSharma',
                       'tags': '',
                       'text': 'RT @Upwork: 😴Sleep later\n'
                               '💻Set up your desk\n'
                               '📷Be ready for prime time\n'
                               '🌿Make your home a pleasant place to be\n'
                               '👫Maintain your connections and m…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 35, 11),
                       'favCount': 0,
                       'id': 1248016567700340738,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'ChristopherVesk',
                       'tags': '',
                       'text': 'In order to educate the children about Jesus. We '
                               'as parents have to live in righteousness.\n'
                               '\n'
                               'That they see and know… https://t.co/oUzq86NaPP'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 35, 6),
                       'favCount': 0,
                       'id': 1248016545848025088,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'NewCreationCap',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 34, 56),
                       'favCount': 0,
                       'id': 1248016506308321282,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'luisa71473442',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 33, 38),
                       'favCount': 1,
                       'id': 1248016178963857410,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'kalvinthedude',
                       'tags': '',
                       'text': 'The mfs that are shitting on people doing home '
                               'workouts are the same ones that made fun of the '
                               'fat kids growing up.… https://t.co/851kqrPWcZ'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 33, 27),
                       'favCount': 0,
                       'id': 1248016132075696128,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 2,
                       'screenname': 'GustavoArellano',
                       'tags': '',
                       'text': 'RT @hbecerraLATimes: I’m going to take a wild '
                               'guess that \u2066@mrmarkpotts\u2069 asked this '
                               'question: “If I have no symptoms and I’m at home '
                               'and I’v…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 32, 3),
                       'favCount': 4,
                       'id': 1248015778227441664,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 2,
                       'screenname': 'hbecerraLATimes',
                       'tags': '',
                       'text': 'I’m going to take a wild guess that '
                               '\u2066@mrmarkpotts\u2069 asked this question: “If '
                               'I have no symptoms and I’m at home and… '
                               'https://t.co/4gShCoCAph'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 30, 53),
                       'favCount': 1,
                       'id': 1248015485951545345,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'Firemonkey991',
                       'tags': '',
                       'text': '@MagazineAmplify Being accustomed to home life, '
                               'the 3 of us are handling the lockdown pretty '
                               'well.\n'
                               'We managed to st… https://t.co/1akbXsIAj1'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 30, 11),
                       'favCount': 1,
                       'id': 1248015309795028992,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'MEOWFoundation',
                       'tags': '',
                       'text': 'The humans may be tiring of working from home, but '
                               "it looks like their kitty co-workers ain't! What's "
                               'better than c… https://t.co/RfXa88WqZ5'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 27, 15),
                       'favCount': 0,
                       'id': 1248014573610786822,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 156,
                       'screenname': 'd7473',
                       'tags': '',
                       'text': 'RT @MNMLCase: Minimalists aren’t opposed to '
                               'spending money. However, we are generally against '
                               'spending too much of it without a darn good r…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 24, 55),
                       'favCount': 0,
                       'id': 1248013982436184065,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 10,
                       'screenname': 'outdampuff',
                       'tags': '',
                       'text': 'RT @SenBobCasey: Millions of Americans depend on '
                               'Meals on Wheels for their nourishment, and '
                               'three-quarters of the volunteers who deliver th…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 24, 53),
                       'favCount': 0,
                       'id': 1248013976224423936,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'awfungowie',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 24, 20),
                       'favCount': 0,
                       'id': 1248013836717707264,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'ameredoux',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 24, 18),
                       'favCount': 0,
                       'id': 1248013828014522369,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'billymayssteps1',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 23, 38),
                       'favCount': 0,
                       'id': 1248013660015833088,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 156,
                       'screenname': 'Sixtyminmn',
                       'tags': '',
                       'text': 'RT @MNMLCase: Minimalists aren’t opposed to '
                               'spending money. However, we are generally against '
                               'spending too much of it without a darn good r…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 23, 25),
                       'favCount': 0,
                       'id': 1248013608887283713,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 1,
                       'screenname': 'MichaelPell',
                       'tags': '',
                       'text': 'RT @sunriseon7: Police across Australia will be '
                               'out in force over the Easter long weekend to '
                               'enforce strict social distancing restrictions…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 20, 51),
                       'favCount': 0,
                       'id': 1248012961815228416,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'jeepjohn',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 20, 12),
                       'favCount': 0,
                       'id': 1248012797440438274,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'valleygirljulie',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 20, 2),
                       'favCount': 0,
                       'id': 1248012755803590657,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'LattimorePT',
                       'tags': '',
                       'text': 'Many of us have recently adopted the working from '
                               'home lifestyle, which could possibly lead to poor '
                               'posture and bac… https://t.co/WaeUM6ROYu'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 19, 19),
                       'favCount': 0,
                       'id': 1248012573284261888,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'moh_choudhury',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 19, 17),
                       'favCount': 0,
                       'id': 1248012565520637952,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'lindy782',
                       'tags': '',
                       'text': '@michaelharriot @KassandraSeven You are correct. '
                               'Until you point out exactly why this is occurring '
                               'nobody looks at… https://t.co/xPHGdhDUUB'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 18, 42),
                       'favCount': 0,
                       'id': 1248012418942267392,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'aleeeganjaaa',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 17, 43),
                       'favCount': 0,
                       'id': 1248012174481448960,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Fumb_Ducks',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 16, 50),
                       'favCount': 0,
                       'id': 1248011948186169346,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'HugoFeijo',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 15, 51),
                       'favCount': 0,
                       'id': 1248011704648101889,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 10,
                       'screenname': 'KatTompkins',
                       'tags': '',
                       'text': 'RT @SenBobCasey: Millions of Americans depend on '
                               'Meals on Wheels for their nourishment, and '
                               'three-quarters of the volunteers who deliver th…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 14, 50),
                       'favCount': 0,
                       'id': 1248011446870401029,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'NYBLURBS',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 14, 26),
                       'favCount': 0,
                       'id': 1248011347201101824,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'TrizzyTray_',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 13, 32),
                       'favCount': 0,
                       'id': 1248011120503181313,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'JessSargus',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 13, 22),
                       'favCount': 0,
                       'id': 1248011078253993989,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'nkdec',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 12, 57),
                       'favCount': 0,
                       'id': 1248010971668307969,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 10,
                       'screenname': 'MainelyLeahy',
                       'tags': '',
                       'text': 'RT @SenBobCasey: Millions of Americans depend on '
                               'Meals on Wheels for their nourishment, and '
                               'three-quarters of the volunteers who deliver th…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 12, 15),
                       'favCount': 0,
                       'id': 1248010795981520896,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'demetripanos',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 11, 22),
                       'favCount': 0,
                       'id': 1248010574195118086,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'dj_jiang',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 10, 53),
                       'favCount': 0,
                       'id': 1248010450853224450,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Myrtil03',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 10, 24),
                       'favCount': 0,
                       'id': 1248010329310638081,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'BAULAPARRANTES',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 9, 13),
                       'favCount': 0,
                       'id': 1248010032148443136,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 13,
                       'screenname': 'theycallmemo_',
                       'tags': '',
                       'text': 'RT @DeeDeeFlora: I suffer from anxiety and this '
                               'has been my 4th week at home. It gets easier once '
                               'you accept the new lifestyle &amp; create you…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 8, 38),
                       'favCount': 0,
                       'id': 1248009886073368576,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'cliffjorgensen',
                       'tags': '',
                       'text': 'Check out this American Lifestyle Magazine blog '
                               'post! The Difference Between Home Staging and '
                               'Decorating https://t.co/NMFsrf2Uuh'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 8, 21),
                       'favCount': 0,
                       'id': 1248009813574815745,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Sylmois_World',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 8, 13),
                       'favCount': 0,
                       'id': 1248009782063034368,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': '_HeyMike',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 8, 9),
                       'favCount': 0,
                       'id': 1248009762844725249,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'shabana_2018',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 7, 10),
                       'favCount': 0,
                       'id': 1248009516907499520,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 156,
                       'screenname': 'pandojevito01',
                       'tags': '',
                       'text': 'RT @MNMLCase: Minimalists aren’t opposed to '
                               'spending money. However, we are generally against '
                               'spending too much of it without a darn good r…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 6),
                       'favCount': 10,
                       'id': 1248009225072070656,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 1,
                       'screenname': 'Upwork',
                       'tags': '',
                       'text': '😴Sleep later\n'
                               '💻Set up your desk\n'
                               '📷Be ready for prime time\n'
                               '🌿Make your home a pleasant place to be\n'
                               '👫Maintain your conne… https://t.co/guZo8AzmA3'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 5, 7),
                       'favCount': 0,
                       'id': 1248009002224476160,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'jrobertsonNM',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 4, 58),
                       'favCount': 0,
                       'id': 1248008962118565888,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 7,
                       'screenname': 'ChristineIAm',
                       'tags': '',
                       'text': 'RT @Newsday: “20 years from now, your children '
                               'will not remember what they learned during the '
                               'spring of 2020…They WILL remember the time th…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 4, 46),
                       'favCount': 0,
                       'id': 1248008913997271040,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'Lee_Marlow_Poly',
                       'tags': '',
                       'text': 'How to successfully work from home and do '
                               'remote-learning with a full household. '
                               'https://t.co/nYIPmqrpvk'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 4, 20),
                       'favCount': 0,
                       'id': 1248008806040096769,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 1,
                       'screenname': 'VCSTX',
                       'tags': ['clean', 'disinfect', 'coronavirus'],
                       'text': 'RT @VCSTX: First #clean &amp; then #disinfect: \n'
                               'how to do it right to keep the #coronavirus at bay '
                               '🏠🦠  \n'
                               '\n'
                               'Please respect dwell times and use ade…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 3, 34),
                       'favCount': 0,
                       'id': 1248008612863082497,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 10,
                       'screenname': 'theReal_Rebel',
                       'tags': '',
                       'text': 'RT @SenBobCasey: Millions of Americans depend on '
                               'Meals on Wheels for their nourishment, and '
                               'three-quarters of the volunteers who deliver th…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 3, 25),
                       'favCount': 0,
                       'id': 1248008575168864256,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'ouragami_',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 3, 20),
                       'favCount': 0,
                       'id': 1248008554373472256,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Himself3909',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 2, 47),
                       'favCount': 0,
                       'id': 1248008413205778438,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'ActionSonora',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 2, 41),
                       'favCount': 0,
                       'id': 1248008390950842368,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 10,
                       'screenname': '30ACTruth',
                       'tags': '',
                       'text': 'RT @SenBobCasey: Millions of Americans depend on '
                               'Meals on Wheels for their nourishment, and '
                               'three-quarters of the volunteers who deliver th…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 2, 33),
                       'favCount': 0,
                       'id': 1248008355269861376,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'AngeliqueGammon',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 2, 14),
                       'favCount': 0,
                       'id': 1248008276689575936,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'ibpixiechick',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 1, 59),
                       'favCount': 0,
                       'id': 1248008214609723392,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'otqvv',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 1, 8),
                       'favCount': 0,
                       'id': 1248007997453819909,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': '1citizenpundit',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 1, 6),
                       'favCount': 0,
                       'id': 1248007991913152512,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'bentoncarolyn25',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 1, 4),
                       'favCount': 0,
                       'id': 1248007984237559808,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'ZombieJester',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 1),
                       'favCount': 0,
                       'id': 1248007964595597313,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'cleverhoax',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 0, 46),
                       'favCount': 0,
                       'id': 1248007905271402497,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'jewelrymavin3',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 59, 9),
                       'favCount': 0,
                       'id': 1248007501255065603,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'SamLitzinger',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 57, 40),
                       'favCount': 0,
                       'id': 1248007126095519744,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'FostersDailyDem',
                       'tags': '',
                       'text': 'Dr. David Itkin of @PortsmouthReg Hospital answers '
                               'questions we’ve received from readers, including '
                               'whether you sho… https://t.co/RGAmmnmbHS'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 57, 25),
                       'favCount': 3,
                       'id': 1248007062786732032,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'seacoastonline',
                       'tags': '',
                       'text': 'Dr. David Itkin of @PortsmouthReg Hospital answers '
                               'questions we’ve received from readers, including '
                               'whether you sho… https://t.co/POJhbuyc70'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 57, 23),
                       'favCount': 0,
                       'id': 1248007056558194690,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Aviv______',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 57, 21),
                       'favCount': 0,
                       'id': 1248007046261125122,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': '30ACTruth',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 57, 20),
                       'favCount': 0,
                       'id': 1248007044273061889,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'rwysocki34',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 57, 7),
                       'favCount': 0,
                       'id': 1248006988853686272,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 10,
                       'screenname': 'libf9wt',
                       'tags': '',
                       'text': 'RT @SenBobCasey: Millions of Americans depend on '
                               'Meals on Wheels for their nourishment, and '
                               'three-quarters of the volunteers who deliver th…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 56, 33),
                       'favCount': 0,
                       'id': 1248006847530860544,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'bohring23',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 56, 21),
                       'favCount': 0,
                       'id': 1248006796817469440,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 10,
                       'screenname': 'GRMSrPa',
                       'tags': '',
                       'text': 'RT @SenBobCasey: Millions of Americans depend on '
                               'Meals on Wheels for their nourishment, and '
                               'three-quarters of the volunteers who deliver th…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 56, 12),
                       'favCount': 0,
                       'id': 1248006756686417922,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 10,
                       'screenname': 'ArgusC',
                       'tags': '',
                       'text': 'RT @SenBobCasey: Millions of Americans depend on '
                               'Meals on Wheels for their nourishment, and '
                               'three-quarters of the volunteers who deliver th…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 55, 53),
                       'favCount': 0,
                       'id': 1248006679184080896,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Harlegator68',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 55, 51),
                       'favCount': 0,
                       'id': 1248006668819947520,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'ElArturoAleman',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 55, 34),
                       'favCount': 0,
                       'id': 1248006596656893952,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'mkwmkwmkwmkw',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 55, 24),
                       'favCount': 0,
                       'id': 1248006556983029760,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Lynmar2020',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 55, 6),
                       'favCount': 0,
                       'id': 1248006482177581056,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'FrontierMetrix',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 54, 58),
                       'favCount': 0,
                       'id': 1248006447650111489,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': '_yayyyo',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 54, 56),
                       'favCount': 0,
                       'id': 1248006438149951488,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'LindyLoo515',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 54, 53),
                       'favCount': 0,
                       'id': 1248006425579638785,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'tra255',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 54, 26),
                       'favCount': 0,
                       'id': 1248006312497049600,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'twaylondra',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 54, 14),
                       'favCount': 0,
                       'id': 1248006262085668864,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'willnanken',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 53, 56),
                       'favCount': 0,
                       'id': 1248006187833909248,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Jonatha80958138',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 53, 51),
                       'favCount': 0,
                       'id': 1248006168103907329,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Emadalayoubi',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 53, 43),
                       'favCount': 0,
                       'id': 1248006132112617472,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'ParkWardenVoice',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 53, 25),
                       'favCount': 0,
                       'id': 1248006057269608456,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'pnwrunnerlass',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 53, 24),
                       'favCount': 0,
                       'id': 1248006054824181760,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'just_jaackiee',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 53, 9),
                       'favCount': 0,
                       'id': 1248005991158779904,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Binahsaurus',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 53, 7),
                       'favCount': 0,
                       'id': 1248005980748537857,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'Rich_Wagner_Esq',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 53, 3),
                       'favCount': 0,
                       'id': 1248005963660976129,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': 'KDHcharley',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 52, 3),
                       'favCount': 0,
                       'id': 1248005711100964864,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 0,
                       'screenname': 'OnyaMag',
                       'tags': '',
                       'text': 'As we all do our part to stay home and flatten the '
                               "curve, some of @discoverLA's best institutions are "
                               'committed to… https://t.co/m6Rdl5RFAD'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 51, 48),
                       'favCount': 0,
                       'id': 1248005649000099841,
                       'product_Line': 'Home and lifestyle',
                       'retweet': 103,
                       'screenname': '_valleygirl07',
                       'tags': '',
                       'text': 'RT @latimes: Gov. Gavin Newsom said the order to '
                               'stay at home to help slow the spread of the '
                               'coronavirus will continue “until further notic…'}]}
    {'ProductLine': 'Sports and travel',
     'ProductLine_Rating': {},
     '_id': ObjectId('5e8e5532b5101d7d130493ca'),
     'customerDictionary': {'Address': '275 Peel Sq',
                            'Date_Of_Purchase': '1/24/19',
                            'FirstName': 'Huey',
                            'LastName': 'Stancil',
                            'Payment_Type': 'Ewallet',
                            'PhoneNumber1': '01502-139578',
                            'PhoneNumber2': '01468-195646',
                            'Time': '18:10',
                            'Total_amount': 804.3,
                            'amount_billed': 804.3,
                            'city': 'Roseville',
                            'customer_type': 'Normal',
                            'email': 'hstancil@hotmail.com',
                            'gender': 'Female',
                            'state': 'CA'},
     'productTweet': [{'created_at': datetime.datetime(2020, 4, 8, 22, 48, 27),
                       'favCount': 0,
                       'id': 1248019906357940225,
                       'product_Line': 'Sports and travel',
                       'retweet': 47,
                       'screenname': 'Lynda63986855',
                       'tags': '',
                       'text': 'RT @Lynda63986855: @vicksiern @jan_aurora '
                               '@Shaun_Girk @MarlaineDettlo1 @ChrisPBaconLT '
                               '@ScottRickhoff @Quin4Trump @davidf4444 '
                               '@TheWickerhead…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 47),
                       'favCount': 0,
                       'id': 1248019543877795840,
                       'product_Line': 'Sports and travel',
                       'retweet': 2,
                       'screenname': 'AustinEilts',
                       'tags': '',
                       'text': 'RT @WIPEvenings: Name a moment/event (sports or '
                               'non sports) you weren’t around for that you’d love '
                               'to travel back and experience. \n'
                               '\n'
                               'https:/…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 40, 45),
                       'favCount': 0,
                       'id': 1248017967528345601,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'rheaecho',
                       'tags': '',
                       'text': 'RT @KJRMIN4J: which activity would u do with me?\n'
                               '\n'
                               '🎉-party \n'
                               '💃🏼-take a dance class\n'
                               '💗-cuddle and watch a movie\n'
                               '🎢-go to an amusement park/zoo\n'
                               '🏝…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 37, 33),
                       'favCount': 4,
                       'id': 1248017162511368192,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'KJRMIN4J',
                       'tags': '',
                       'text': 'which activity would u do with me?\n'
                               '\n'
                               '🎉-party \n'
                               '💃🏼-take a dance class\n'
                               '💗-cuddle and watch a movie\n'
                               '🎢-go to an amusement… https://t.co/lLCk9jwO0q'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 33, 17),
                       'favCount': 0,
                       'id': 1248016088597553158,
                       'product_Line': 'Sports and travel',
                       'retweet': 47,
                       'screenname': 'jeep_sifu',
                       'tags': '',
                       'text': 'RT @Lynda63986855: @vicksiern @jan_aurora '
                               '@Shaun_Girk @MarlaineDettlo1 @ChrisPBaconLT '
                               '@ScottRickhoff @Quin4Trump @davidf4444 '
                               '@TheWickerhead…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 13, 29),
                       'favCount': 0,
                       'id': 1248011105420455936,
                       'product_Line': 'Sports and travel',
                       'retweet': 47,
                       'screenname': 'ChrisPBaconLT',
                       'tags': '',
                       'text': 'RT @Lynda63986855: @vicksiern @jan_aurora '
                               '@Shaun_Girk @MarlaineDettlo1 @ChrisPBaconLT '
                               '@ScottRickhoff @Quin4Trump @davidf4444 '
                               '@TheWickerhead…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 8, 2),
                       'favCount': 2,
                       'id': 1248009735397244929,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'yourboiijames',
                       'tags': '',
                       'text': 'which activity would u do with me?\n'
                               '\n'
                               '🎉-party \n'
                               '💃🏼-take a dance class\n'
                               '💗-cuddle and watch a movie\n'
                               '🎢-go to an amusement… https://t.co/gtkQhucARD'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 6, 34),
                       'favCount': 9,
                       'id': 1248009367950995459,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'pharsyy',
                       'tags': '',
                       'text': 'which activity would u do with me?\n'
                               '\n'
                               '🎉-party \n'
                               '💃🏼-take a dance class\n'
                               '💗-cuddle and watch a movie\n'
                               '🎢-go to an amusement… https://t.co/zAOfFtrVZW'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 5, 34),
                       'favCount': 0,
                       'id': 1248009114820595713,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'CoolHandLuke124',
                       'tags': '',
                       'text': '@KathyLovesPizza @nytimes Not arguing if they '
                               'should receive help but you could say the same '
                               'thing about all  enter… https://t.co/DA9QAcI6Iy'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 0, 32),
                       'favCount': 0,
                       'id': 1248007846052032512,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'Sewn_apart',
                       'tags': '',
                       'text': '@bringonthebeer To be fair, i think international '
                               'travel and sports stadia will later. I can see '
                               'pubs being earlier… https://t.co/2eD32u5EeR'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 51, 13),
                       'favCount': 0,
                       'id': 1248005501708668928,
                       'product_Line': 'Sports and travel',
                       'retweet': 2,
                       'screenname': 'JoeGiglioSports',
                       'tags': '',
                       'text': 'RT @WIPEvenings: Name a moment/event (sports or '
                               'non sports) you weren’t around for that you’d love '
                               'to travel back and experience. \n'
                               '\n'
                               'https:/…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 50, 38),
                       'favCount': 8,
                       'id': 1248005356690653185,
                       'product_Line': 'Sports and travel',
                       'retweet': 2,
                       'screenname': 'WIPEvenings',
                       'tags': '',
                       'text': 'Name a moment/event (sports or non sports) you '
                               'weren’t around for that you’d love to travel back '
                               'and experience. \n'
                               '\n'
                               'https://t.co/0N55IWC6bz'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 47, 11),
                       'favCount': 0,
                       'id': 1248004488650084354,
                       'product_Line': 'Sports and travel',
                       'retweet': 3,
                       'screenname': 'marymamie5',
                       'tags': '',
                       'text': 'RT @tax: Travel and tourism—which funnels $83 '
                               'billion in tax revenue annually to state and local '
                               'governments—is in a freefall triggered by…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 46, 3),
                       'favCount': 0,
                       'id': 1248004205010247685,
                       'product_Line': 'Sports and travel',
                       'retweet': 13,
                       'screenname': 'ruedelachaise',
                       'tags': '',
                       'text': 'RT @MylesSuer: .@BillGates levels with Americans '
                               'on @TheDailyShow re what is in front of us. It '
                               'will take a month to start to bend the curv…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 39, 23),
                       'favCount': 0,
                       'id': 1248002525539610624,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'rebelsart',
                       'tags': '',
                       'text': 'RT @rebelsart: @MickaelleMichel @SportsHochi Thank '
                               'you for being a wonderful ambassador for horse '
                               'racing and sports! Have a safe travel bac…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 23, 19),
                       'favCount': 2,
                       'id': 1247998483510026241,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'Travel_Iowa',
                       'tags': '',
                       'text': 'Missing sports? You’re in luck. Tonight, you can '
                               'catch virtual racing live from the sprint car '
                               'capital of the world… https://t.co/69vHKMXjGy'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 3, 51),
                       'favCount': 0,
                       'id': 1247993583916986369,
                       'product_Line': 'Sports and travel',
                       'retweet': 116,
                       'screenname': 'step_up_sports',
                       'tags': '',
                       'text': 'RT @Bakler1: Is now the time to promote all '
                               'National Prem sides to EFL to make a EFL2 north '
                               'and EFL2 south, with national league just a nor…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 47, 43),
                       'favCount': 0,
                       'id': 1247989524103188480,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 't_tomochika2014',
                       'tags': '',
                       'text': 'RT @rebelsart: @MickaelleMichel @SportsHochi Thank '
                               'you for being a wonderful ambassador for horse '
                               'racing and sports! Have a safe travel bac…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 39, 37),
                       'favCount': 0,
                       'id': 1247987483394158592,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'gdelgado210',
                       'tags': '',
                       'text': 'RT @prettyybirrd: @davidrocknyc I own my own '
                               'business in the sports and travel industry. What I '
                               'need are sponsors who are who are looking f…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 33, 21),
                       'favCount': 0,
                       'id': 1247985909464805383,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'Sports_Schlub',
                       'tags': '',
                       'text': '@CNN Government keeps closing local parks and '
                               'limiting travel in order to trap brown people in '
                               'their cramped neighb… https://t.co/BAh1jWYWYa'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 30, 49),
                       'favCount': 0,
                       'id': 1247985272093954048,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'luke_h_wright',
                       'tags': '',
                       'text': 'So the crux of banning/avoiding backcountry travel '
                               'and sports has been 1) not spreading COVID to new '
                               'communities an… https://t.co/MG8CvDy7Ao'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 30, 7),
                       'favCount': 2,
                       'id': 1247985093685252098,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'mynameistiwani',
                       'tags': '',
                       'text': 'Sky Sports GOTS making me want to time travel back '
                               'to 1993, Le Tiss, Shearer and Mark Hughes were on '
                               'crud'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 29, 49),
                       'favCount': 0,
                       'id': 1247985019152408576,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'ChabeauMichel',
                       'tags': '',
                       'text': 'RT @rebelsart: @MickaelleMichel @SportsHochi Thank '
                               'you for being a wonderful ambassador for horse '
                               'racing and sports! Have a safe travel bac…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 27, 14),
                       'favCount': 4,
                       'id': 1247984368531976192,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'JamesKratch',
                       'tags': '',
                       'text': "It's interesting how many of the stories about "
                               'college sports and coronavirus economics reference '
                               'travel costs as a… https://t.co/S3zgtoc1Al'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 26, 51),
                       'favCount': 2,
                       'id': 1247984272482414605,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'rabcyr',
                       'tags': '',
                       'text': 'closing the borders (except essential goods) and '
                               'stopping all air travel (and sports &amp; '
                               'concerts) back in january, e… '
                               'https://t.co/nUuug2X7jJ'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 22, 5),
                       'favCount': 0,
                       'id': 1247983073364508672,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'KeisersozayHTTC',
                       'tags': '',
                       'text': '@ClydeSSB Even when this pandemic is under contoll '
                               'sanctions will remain for a extended period of '
                               'time  to ensure t… https://t.co/4B1s5IZi2y'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 20, 17),
                       'favCount': 0,
                       'id': 1247982621260480514,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'DirectTrip',
                       'tags': ['UK', 'Travel'],
                       'text': 'Travel and hospitality hardest hit as COVID-19 '
                               'continues to ravage UK job market - Yahoo Sports '
                               'https://t.co/6HIaYPDfye #UK #Travel'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 17, 19),
                       'favCount': 0,
                       'id': 1247981874007248898,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'Travel_MSW',
                       'tags': '',
                       'text': 'RT @ASWISports: The term "identity foreclosure" '
                               'has become all too relevant during this pandemic '
                               'and especially in sports. https://t.co/BqH…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 3, 50),
                       'favCount': 0,
                       'id': 1247978481578913803,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'jawhoriskey',
                       'tags': ['vacations',
                                'business',
                                'sports',
                                'religious',
                                'club',
                                'travels'],
                       'text': 'RT @JzwTravel: JZW Travel not only books '
                               '#vacations but we also book #business travel as '
                               'well as #sports, #religious and #club #travels. '
                               'Co…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 59, 1),
                       'favCount': 0,
                       'id': 1247977265557901314,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'JzwTravel',
                       'tags': ['vacations',
                                'business',
                                'sports',
                                'religious',
                                'club'],
                       'text': 'JZW Travel not only books #vacations but we also '
                               'book #business travel as well as #sports, '
                               '#religious and #club… https://t.co/rLhCABIjUY'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 34, 32),
                       'favCount': 0,
                       'id': 1247971106302107653,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'dxgby',
                       'tags': '',
                       'text': '@AcnhSally " That\'s why I came here.. "\n'
                               '\n'
                               '    He pulls out a travel bag, taking out tight '
                               'black yoga pants and a spo… '
                               'https://t.co/hvlgQuBMj5'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 37, 45),
                       'favCount': 0,
                       'id': 1247956815582760960,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'atic_sports',
                       'tags': '',
                       'text': '@stewart7pete @CFBKnights @bignizo @Brett_McMurphy '
                               '@Stadium @CFBPlayoff Nobody outside of Memphis '
                               'wanted to see Mem… https://t.co/fpu6giWNlH'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 34, 47),
                       'favCount': 0,
                       'id': 1247956067977285633,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'sawarnspeaks',
                       'tags': '',
                       'text': "At this point, we can make changes that wouldn't "
                               'require any extra travel, unless there is some '
                               'important lab/sport… https://t.co/AuPA9j4xKE'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 27, 45),
                       'favCount': 0,
                       'id': 1247954299348488193,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'rene4the5',
                       'tags': '',
                       'text': 'Just like in the world of sports, stupidity, lying '
                               'and cinicism can always breake records. Freeze '
                               'Travel to and fro… https://t.co/Xsd4rNs2Fg'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 21, 36),
                       'favCount': 1,
                       'id': 1247952751352561665,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'SugdenSteve',
                       'tags': '',
                       'text': '@DentonClark17 Well to be fair.....sports does '
                               'make economy go....travel tickets gear food '
                               'usually stop to eat and… https://t.co/ytOhPCB8IN'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 8, 53),
                       'favCount': 0,
                       'id': 1247949553447231488,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'beatsatmonster',
                       'tags': '',
                       'text': 'DeftGet First Aid Kit – 163 Piece Waterproof '
                               'Portable Essential Injuries &amp; Red Cross '
                               'Medical Emergency Equipment Ki… '
                               'https://t.co/yoWJ4n41n3'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 57, 29),
                       'favCount': 0,
                       'id': 1247946681158635520,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'LeeAbbamonte',
                       'tags': '',
                       'text': 'RT @AmberTheoharis: Love sports and travel? Join '
                               'me and travel expert @LeeAbbamonte today on IG '
                               'Live T 1pm PT/ 4pm ET. He’s been to everyon…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 51, 41),
                       'favCount': 0,
                       'id': 1247945224565010432,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'ThisIsQuasimodo',
                       'tags': '',
                       'text': '@NathiMthethwaSA Honourable Minister, please can '
                               'you tell me why sports administrators are allowed '
                               'to lead organiza… https://t.co/lrhu3HRtyS'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 29, 44),
                       'favCount': 0,
                       'id': 1247939699211026432,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'Billsmafia178',
                       'tags': '',
                       'text': 'RT @AmberTheoharis: Love sports and travel? Join '
                               'me and travel expert @LeeAbbamonte today on IG '
                               'Live T 1pm PT/ 4pm ET. He’s been to everyon…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 28, 5),
                       'favCount': 0,
                       'id': 1247939283857444864,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'Michael_Fabiano',
                       'tags': '',
                       'text': 'RT @AmberTheoharis: Love sports and travel? Join '
                               'me and travel expert @LeeAbbamonte today on IG '
                               'Live T 1pm PT/ 4pm ET. He’s been to everyon…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 26, 5),
                       'favCount': 0,
                       'id': 1247938780968800257,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'gordwright',
                       'tags': '',
                       'text': '@llikemoyd Donate those “deposit” bottles guys. '
                               'Sports teams will need to fundraise soon (and by '
                               'the sounds of it t… https://t.co/VTHcFRmhKC'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 22, 1),
                       'favCount': 0,
                       'id': 1247937756728262661,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'TheShergar',
                       'tags': '',
                       'text': 'RT @FCurran17: Despite all the doom &amp; gloom '
                               'and its gonna get worse in new week or two deaths '
                               'wise I fully believe by end of April/early ma…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 17, 36),
                       'favCount': 4,
                       'id': 1247936646072467467,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'FCurran17',
                       'tags': '',
                       'text': 'Despite all the doom &amp; gloom and its gonna get '
                               'worse in new week or two deaths wise I fully '
                               'believe by end of April… https://t.co/NCv7UBJNAl'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 17, 21),
                       'favCount': 2,
                       'id': 1247936584109789184,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'ManSaffer',
                       'tags': '',
                       'text': '@MollyJongFast I’m hoping for movie theaters, '
                               'restaurants, sports stadiums, pubs, beaches, and '
                               'air travel to name a few'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 16, 36),
                       'favCount': 1,
                       'id': 1247936395794173952,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'TeamSC11',
                       'tags': '',
                       'text': '@3ManFront I travel a good bit and every sports '
                               'bar with 30 to 80 TVs have every most TVs on the '
                               'games and at the s… https://t.co/FokHVszSV5'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 13, 39),
                       'favCount': 0,
                       'id': 1247935652345155584,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'js92879',
                       'tags': '',
                       'text': 'RT @AmberTheoharis: Love sports and travel? Join '
                               'me and travel expert @LeeAbbamonte today on IG '
                               'Live T 1pm PT/ 4pm ET. He’s been to everyon…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 12, 43),
                       'favCount': 0,
                       'id': 1247935416642256903,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'VernellGordon',
                       'tags': '',
                       'text': 'RT @AmberTheoharis: Love sports and travel? Join '
                               'me and travel expert @LeeAbbamonte today on IG '
                               'Live T 1pm PT/ 4pm ET. He’s been to everyon…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 12, 6),
                       'favCount': 0,
                       'id': 1247935260328935424,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'tinsellinsell',
                       'tags': '',
                       'text': '@leicspolice Just seen a couple of microlights '
                               'flying in tandem and messing about over Harbs. '
                               'Assuming this is esse… https://t.co/2RwcbDcD1X'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 10, 5),
                       'favCount': 29,
                       'id': 1247934752889290752,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'AmberTheoharis',
                       'tags': '',
                       'text': 'Love sports and travel? Join me and travel expert '
                               '@LeeAbbamonte today on IG Live T 1pm PT/ 4pm ET. '
                               'He’s been to eve… https://t.co/711ko3G255'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 5, 36),
                       'favCount': 0,
                       'id': 1247933624374214659,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'MarkWittig',
                       'tags': '',
                       'text': '@NARNfan @AndrewLeeTCNT Looks bad when all the '
                               'other sports are not playing and that travel '
                               'thing'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 1, 2),
                       'favCount': 0,
                       'id': 1247932477022375936,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'SwanBoatSteve',
                       'tags': '',
                       'text': '@regionomics Downtown works and would work better '
                               'if only (a) there were even more transit and (b) '
                               'we stopped sched… https://t.co/M2uRUDlBqE'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 49, 12),
                       'favCount': 1,
                       'id': 1247929499901100034,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'C4_Sports_',
                       'tags': '',
                       'text': 'A choice is never simple it’s never easy it’s not '
                               'supposed to be but those who travel the path have '
                               'always looked b… https://t.co/4PfNkQpTAV'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 20, 10),
                       'favCount': 0,
                       'id': 1247922190038773766,
                       'product_Line': 'Sports and travel',
                       'retweet': 4,
                       'screenname': 'ThomasOJaeger',
                       'tags': '',
                       'text': 'RT @clublasanta: In the middle of the volcanic '
                               'landscape of Lanzarote, lies a true paradise of '
                               'sports, wellness and relaxation. Some say it…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 7, 24),
                       'favCount': 0,
                       'id': 1247918980272140289,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'sebgoursaud',
                       'tags': '',
                       'text': 'RT @rebelsart: @MickaelleMichel @SportsHochi Thank '
                               'you for being a wonderful ambassador for horse '
                               'racing and sports! Have a safe travel bac…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 5, 19),
                       'favCount': 0,
                       'id': 1247918453157216263,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': '___Taekook_',
                       'tags': '',
                       'text': 'Hey! I discovered the BEST app to practice '
                               'English! 🇺🇸 It puts you in real life situations '
                               'for you to learn in prac… https://t.co/xyub7kQ2Sp'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 4, 54),
                       'favCount': 3,
                       'id': 1247918348412878854,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'UHennessey',
                       'tags': '',
                       'text': 'Oh goodness, yes. If this was around when I was '
                               'writing sports features (which took weeks and '
                               'hours and cross count… https://t.co/HAffPiKI2h'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 41, 35),
                       'favCount': 0,
                       'id': 1247912480120156168,
                       'product_Line': 'Sports and travel',
                       'retweet': 4,
                       'screenname': 'DaisyDots3',
                       'tags': '',
                       'text': 'RT @clublasanta: In the middle of the volcanic '
                               'landscape of Lanzarote, lies a true paradise of '
                               'sports, wellness and relaxation. Some say it…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 33, 43),
                       'favCount': 0,
                       'id': 1247910503151800321,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'TheWealthMind',
                       'tags': '',
                       'text': '@DallenReber Sports, Music, and Travel 🙌🏻'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 27, 35),
                       'favCount': 0,
                       'id': 1247908958213476356,
                       'product_Line': 'Sports and travel',
                       'retweet': 3,
                       'screenname': 'jennifergorl',
                       'tags': '',
                       'text': "RT @Jose_Palmtree: All I'm tryna do this summer "
                               'was turn up, travel and have deep late night talks '
                               'with my friends and play sports and go t…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 19, 33),
                       'favCount': 0,
                       'id': 1247906939104899073,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'FStarSolutions',
                       'tags': '',
                       'text': 'Friends support our efforts in elevating sports '
                               'and travel hospitality programs by following our '
                               'small company Firs… https://t.co/MJnjfcuQqX'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 16, 12),
                       'favCount': 0,
                       'id': 1247906092501196802,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'superaiwa',
                       'tags': '',
                       'text': 'Hey! I discovered the BEST app to practice '
                               'Spanish! 🇪🇸 It puts you in real life situations '
                               'for you to learn in prac… https://t.co/etbwxa1l32'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 12, 33),
                       'favCount': 0,
                       'id': 1247905175005614084,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'bjm262run',
                       'tags': '',
                       'text': '“By eliminating travel, removing fans, and '
                               'enacting measures to ensure that athletes can’t '
                               'easily transmit the viru… https://t.co/hn9w9nkOkj'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 5, 5),
                       'favCount': 0,
                       'id': 1247903295886118914,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'sportstravelint',
                       'tags': '',
                       'text': "So runners, who doesn't love a game of a bingo for "
                               "a bit of light hearted fun. Here's your Sports "
                               'Travel Internatio… https://t.co/vNsUbzCDVh'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 1, 58),
                       'favCount': 1,
                       'id': 1247902513061232643,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'issa_reggie',
                       'tags': '',
                       'text': 'Because of the virus: cut off the cable because no '
                               'sports. Can’t hoop. Can’t play football. Can’t '
                               'workout. Can’t sh… https://t.co/Ry5HSdxqzk'},
                      {'created_at': datetime.datetime(2020, 4, 8, 14, 51, 9),
                       'favCount': 0,
                       'id': 1247899791645765639,
                       'product_Line': 'Sports and travel',
                       'retweet': 114,
                       'screenname': '_steegs',
                       'tags': '',
                       'text': 'RT @AmeshAA: "We\'ve got to expect that businesses '
                               'must reopen and schools must teach again. Whether '
                               "it's travel or sports or live entertain…"},
                      {'created_at': datetime.datetime(2020, 4, 8, 14, 39, 6),
                       'favCount': 0,
                       'id': 1247896755955326983,
                       'product_Line': 'Sports and travel',
                       'retweet': 4,
                       'screenname': 'NetballJo',
                       'tags': '',
                       'text': 'RT @clublasanta: In the middle of the volcanic '
                               'landscape of Lanzarote, lies a true paradise of '
                               'sports, wellness and relaxation. Some say it…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 14, 20, 10),
                       'favCount': 0,
                       'id': 1247891994875068418,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'SteveODonnell1',
                       'tags': '',
                       'text': '@MikelSevere @damonbenning Having sports return '
                               'with a crowd is a uplifting sign both economically '
                               'and just in gene… https://t.co/MAV6E12mp1'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 53, 21),
                       'favCount': 0,
                       'id': 1247885244792881154,
                       'product_Line': 'Sports and travel',
                       'retweet': 4,
                       'screenname': 'canariascalida',
                       'tags': '',
                       'text': 'RT @clublasanta: In the middle of the volcanic '
                               'landscape of Lanzarote, lies a true paradise of '
                               'sports, wellness and relaxation. Some say it…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 53, 6),
                       'favCount': 0,
                       'id': 1247885180896858114,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'theline4two',
                       'tags': '',
                       'text': '@Trace_AVP @Chuck1one @eepdllc @Rick__War '
                               '@Woodshed_1914 @Freekeith @stateofthenewy1 '
                               '@ginisangpepe @AJTheManChild… '
                               'https://t.co/j4Ivsdknte'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 50, 46),
                       'favCount': 0,
                       'id': 1247884593975267330,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'ColleenSerran15',
                       'tags': ['travel', 'China', 'coronavirus', 'CoronaOutbreak'],
                       'text': 'RT @amzjoel: Life in China. Another day. Work and '
                               'play. https://t.co/v7SC1lHiyH via @YouTube #travel '
                               '#China #coronavirus #CoronaOutbreak #m…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 45, 19),
                       'favCount': 0,
                       'id': 1247883224526868480,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'spanu_patricia',
                       'tags': '',
                       'text': 'RT @rebelsart: @MickaelleMichel @SportsHochi Thank '
                               'you for being a wonderful ambassador for horse '
                               'racing and sports! Have a safe travel bac…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 33, 44),
                       'favCount': 0,
                       'id': 1247880306486435842,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'EXONDIN',
                       'tags': '',
                       'text': 'Hobbies &amp; Interests : Animation, art, video '
                               'games, computers, water sports, racket sports, '
                               'nature sports, basketbal… https://t.co/l2yav8zEv8'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 29, 56),
                       'favCount': 15,
                       'id': 1247879352911454208,
                       'product_Line': 'Sports and travel',
                       'retweet': 5,
                       'screenname': 'rebelsart',
                       'tags': '',
                       'text': '@MickaelleMichel @SportsHochi Thank you for being '
                               'a wonderful ambassador for horse racing and '
                               'sports! Have a safe t… https://t.co/ycMS1HnxAB'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 23, 10),
                       'favCount': 38,
                       'id': 1247877647629045760,
                       'product_Line': 'Sports and travel',
                       'retweet': 4,
                       'screenname': 'clublasanta',
                       'tags': '',
                       'text': 'In the middle of the volcanic landscape of '
                               'Lanzarote, lies a true paradise of sports, '
                               'wellness and relaxation. Some… '
                               'https://t.co/EjhAF48DqW'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 21, 45),
                       'favCount': 0,
                       'id': 1247877293126680577,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'a_travel_bot',
                       'tags': '',
                       'text': 'DO\n'
                               'You have to bring your own food however. Other '
                               'water sports like boating and fishing are also '
                               'plentiful.'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 17, 16),
                       'favCount': 0,
                       'id': 1247876164380700673,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'Travel_Cowboy',
                       'tags': '',
                       'text': '@dmn_cowboys bullshit @TimCowlishaw - sports '
                               'returning as soon as they safely can (SAFELY) is '
                               'one of the things tha… https://t.co/8bUIzLCxGy'},
                      {'created_at': datetime.datetime(2020, 4, 8, 12, 30, 59),
                       'favCount': 4,
                       'id': 1247864517901660163,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'KCentral_Sports',
                       'tags': '',
                       'text': 'Megan Rushlau - Softball\n'
                               '\n'
                               'Also participated in band. I have played travel '
                               'softball for 6 years and will continue my… '
                               'https://t.co/77DsSON0aB'},
                      {'created_at': datetime.datetime(2020, 4, 8, 11, 53, 14),
                       'favCount': 0,
                       'id': 1247855014481018880,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'lahar1999',
                       'tags': '',
                       'text': '@MarkFagg The other sports got stuck when state '
                               'borders closed to non-essential travel.  Racing '
                               'can be run locally… https://t.co/fJy0UDqhgx'},
                      {'created_at': datetime.datetime(2020, 4, 8, 10, 24, 44),
                       'favCount': 2,
                       'id': 1247832744241094656,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'TheShortBear',
                       'tags': '',
                       'text': '@szab0r A sports car is one of the dumbest '
                               'purchases you can make from a purely financial '
                               'standpoint. There are onl… '
                               'https://t.co/GcKVzHfa9W'},
                      {'created_at': datetime.datetime(2020, 4, 8, 10, 11, 35),
                       'favCount': 0,
                       'id': 1247829433886609408,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'spotlaits',
                       'tags': ['COVID19'],
                       'text': 'RT @getppcexpo: 7 Industries that are suffering '
                               'from the #COVID19 Pandemic\n'
                               '\n'
                               '1. Travel and tourism\n'
                               '2. Bars and restaurants\n'
                               '3. Live entertain…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 9, 46, 58),
                       'favCount': 0,
                       'id': 1247823239750897668,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'RichCentury',
                       'tags': '',
                       'text': 'The wireless tour guide system is a rugged tour '
                               'guide system such as travel guides, multilingual '
                               'interpretation, he… https://t.co/Snb20SyR7f'},
                      {'created_at': datetime.datetime(2020, 4, 8, 8, 56, 26),
                       'favCount': 0,
                       'id': 1247810521568407552,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'wirelesscamera3',
                       'tags': '',
                       'text': 'Monocular Telescope High Power, 12×42 Monocular '
                               'Scope with Smartphone Adapter and Tripod Dual '
                               'Focus for Outdoor Spo… https://t.co/hZxWwIOLkW'},
                      {'created_at': datetime.datetime(2020, 4, 8, 7, 38, 44),
                       'favCount': 0,
                       'id': 1247790968583016449,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'KogojSlavko',
                       'tags': '',
                       'text': 'RT @HuginnTravel: Ukrepali so takoj po 2. in 3. '
                               'primeru.\n'
                               '\n'
                               '"With just three confirmed coronavirus cases so '
                               'far, Lithuania introduced yesterd…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 7, 33, 5),
                       'favCount': 2,
                       'id': 1247789545363361796,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'HuginnTravel',
                       'tags': '',
                       'text': 'Ukrepali so takoj po 2. in 3. primeru.\n'
                               '\n'
                               '"With just three confirmed coronavirus cases so '
                               'far, Lithuania introduced y… '
                               'https://t.co/G8vG3czHyx'},
                      {'created_at': datetime.datetime(2020, 4, 8, 7, 8, 29),
                       'favCount': 0,
                       'id': 1247783356219060227,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'shahidk56970047',
                       'tags': '',
                       'text': 'SkyGenius 10 x 50 Powerful Binoculars for Adults '
                               'Durable Full-Size Clear Binoculars for Bird '
                               'Watching Travel Sights… https://t.co/pzDxZMlk1c'},
                      {'created_at': datetime.datetime(2020, 4, 8, 6, 50, 46),
                       'favCount': 0,
                       'id': 1247778896142991361,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'NassahKapur',
                       'tags': '',
                       'text': "@rid1kader I'm speculating that schools , "
                               'religious gatherings and other large gatherings '
                               '(sports events) will be p… '
                               'https://t.co/Uet6a2uH8K'},
                      {'created_at': datetime.datetime(2020, 4, 8, 5, 26),
                       'favCount': 1,
                       'id': 1247757567092117504,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'RonMcGinnis9',
                       'tags': '',
                       'text': '@dreaminok7 @RealMattCouch Here in Pa. we are '
                               'under a stay at home advisory until April 30 '
                               'issued by the governor ,… https://t.co/HgDLsMZxmg'},
                      {'created_at': datetime.datetime(2020, 4, 8, 5, 25, 9),
                       'favCount': 0,
                       'id': 1247757349911015424,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'RobertAlabaster',
                       'tags': '',
                       'text': '@nytimes True, the man is like a powerful sports '
                               "car without a steering wheel. But that doesn't "
                               'meant the WHO is of… https://t.co/ky0Tqyx72D'},
                      {'created_at': datetime.datetime(2020, 4, 8, 3, 27, 58),
                       'favCount': 0,
                       'id': 1247727863018606592,
                       'product_Line': 'Sports and travel',
                       'retweet': 3,
                       'screenname': 'Clarissaarri',
                       'tags': '',
                       'text': "RT @Jose_Palmtree: All I'm tryna do this summer "
                               'was turn up, travel and have deep late night talks '
                               'with my friends and play sports and go t…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 3, 9, 52),
                       'favCount': 0,
                       'id': 1247723304728891392,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'pistons_eq_man',
                       'tags': ['EQRoundtable'],
                       'text': 'RT @SamuelScobie77: Another great #EQRoundtable '
                               'today organized by @NetsGov! Interesting '
                               'discussion about travel challenges and solutions '
                               'a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 2, 33, 43),
                       'favCount': 0,
                       'id': 1247714209972002818,
                       'product_Line': 'Sports and travel',
                       'retweet': 3,
                       'screenname': 'progresivetrend',
                       'tags': '',
                       'text': 'RT @BLaw: Travel and tourism—which funnels $83 '
                               'billion in tax revenue annually to state and local '
                               'governments—is in a freefall triggered by…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 2, 22, 55),
                       'favCount': 0,
                       'id': 1247711492033691652,
                       'product_Line': 'Sports and travel',
                       'retweet': 3,
                       'screenname': 'ARipple_DAsport',
                       'tags': '',
                       'text': 'RT @BradRosemas: Local travel softball teams '
                               'effected by the coronavirus are optimistic their '
                               'teams will remain strong during their breaks.…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 2, 20, 49),
                       'favCount': 0,
                       'id': 1247710961336754176,
                       'product_Line': 'Sports and travel',
                       'retweet': 3,
                       'screenname': 'JoseAnzures6',
                       'tags': '',
                       'text': "RT @Jose_Palmtree: All I'm tryna do this summer "
                               'was turn up, travel and have deep late night talks '
                               'with my friends and play sports and go t…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 2, 15, 40),
                       'favCount': 7,
                       'id': 1247709667217768448,
                       'product_Line': 'Sports and travel',
                       'retweet': 3,
                       'screenname': 'Jose_Palmtree',
                       'tags': '',
                       'text': "All I'm tryna do this summer was turn up, travel "
                               'and have deep late night talks with my friends and '
                               'play sports and… https://t.co/qiqRs6uWMk'},
                      {'created_at': datetime.datetime(2020, 4, 8, 1, 58, 27),
                       'favCount': 0,
                       'id': 1247705333478428672,
                       'product_Line': 'Sports and travel',
                       'retweet': 3,
                       'screenname': 'DADylanJohnson',
                       'tags': '',
                       'text': 'RT @BradRosemas: Local travel softball teams '
                               'effected by the coronavirus are optimistic their '
                               'teams will remain strong during their breaks.…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 1, 45, 24),
                       'favCount': 0,
                       'id': 1247702051359821825,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'oscorft',
                       'tags': '',
                       'text': 'Let me give you a window into the future. You will '
                               'be denied travel, work, play sports, go to '
                               'restaurants and any o… https://t.co/6YA23UrVHz'},
                      {'created_at': datetime.datetime(2020, 4, 8, 1, 37, 10),
                       'favCount': 1,
                       'id': 1247699976635002880,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'pphilly10',
                       'tags': '',
                       'text': '@bobbythebookie @JeffLesson No travel sports for '
                               "kids I've played more golf in 2 weeks than I do "
                               "all summer normally and I'm a golf coach."},
                      {'created_at': datetime.datetime(2020, 4, 8, 1, 36, 59),
                       'favCount': 0,
                       'id': 1247699933076926464,
                       'product_Line': 'Sports and travel',
                       'retweet': 0,
                       'screenname': 'JamesSpenceley',
                       'tags': '',
                       'text': '@LT3000Lyall @buildthefutures @Mars10387340 '
                               '@WillKoulouris Yes, and we (generally) no longer '
                               'have unprotected sex,… https://t.co/FO5nA9Sn9d'},
                      {'created_at': datetime.datetime(2020, 4, 8, 1, 14, 29),
                       'favCount': 15,
                       'id': 1247694271198699520,
                       'product_Line': 'Sports and travel',
                       'retweet': 1,
                       'screenname': 'SamuelScobie77',
                       'tags': ['EQRoundtable'],
                       'text': 'Another great #EQRoundtable today organized by '
                               '@NetsGov! Interesting discussion about travel '
                               'challenges and solutio… https://t.co/9zVx0lUfyk'},
                      {'created_at': datetime.datetime(2020, 4, 8, 1, 12, 53),
                       'favCount': 0,
                       'id': 1247693865127235584,
                       'product_Line': 'Sports and travel',
                       'retweet': 3,
                       'screenname': 'Cooljenim',
                       'tags': '',
                       'text': 'RT @BLaw: Travel and tourism—which funnels $83 '
                               'billion in tax revenue annually to state and local '
                               'governments—is in a freefall triggered by…'}]}
    {'ProductLine': 'Food and beverages',
     'ProductLine_Rating': {},
     '_id': ObjectId('5e8e5532b5101d7d130493cb'),
     'customerDictionary': {'Address': '714 Fonthill Rd',
                            'Date_Of_Purchase': '2/9/19',
                            'FirstName': 'Charlette',
                            'LastName': 'Brenning',
                            'Payment_Type': 'Cash',
                            'PhoneNumber1': '01888-152110',
                            'PhoneNumber2': '01301-312487',
                            'Time': '13:22',
                            'Total_amount': 33.431999999999995,
                            'amount_billed': 33.431999999999995,
                            'city': 'Alliance',
                            'customer_type': 'Member',
                            'email': 'cbrenning@brenning.co.uk',
                            'gender': 'Male',
                            'state': 'NE'},
     'productTweet': [{'created_at': datetime.datetime(2020, 4, 8, 22, 40, 52),
                       'favCount': 0,
                       'id': 1248018000122281984,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'VIRBOY21',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 49, 26),
                       'favCount': 1,
                       'id': 1248005053928992768,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'RayDubicki',
                       'tags': '',
                       'text': 'TIL there is an extensive section of the Seattle '
                               'City Code devoted to milk.\n'
                               '\n'
                               'And it’s probably against the law for… '
                               'https://t.co/OCkLCFLinS'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 47, 25),
                       'favCount': 0,
                       'id': 1248004547814903810,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'HofNorthwoods',
                       'tags': '',
                       'text': '@stealthygeek Golfing! Our local course where I '
                               'have great memories with my dad, who was sadly '
                               'lost to agent orange… https://t.co/Ia1NJP0EaA'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 40, 12),
                       'favCount': 0,
                       'id': 1248002728967589890,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'iseeassociates',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 38, 7),
                       'favCount': 0,
                       'id': 1248002206277615616,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'cummings_jim2',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 33, 20),
                       'favCount': 0,
                       'id': 1248001002797662208,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'BobBaileyPC',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 33, 9),
                       'favCount': 0,
                       'id': 1248000955821555716,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'Roxesays',
                       'tags': ['TheFive'],
                       'text': '#TheFive\n'
                               '\n'
                               'Making prepared food and beverages tax free for at '
                               'least a year or for the rest of this year is an '
                               'EXCELL… https://t.co/ytQphJnvxJ'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 30, 2),
                       'favCount': 4,
                       'id': 1248000171700453376,
                       'product_Line': 'Food and beverages',
                       'retweet': 1,
                       'screenname': 'Jorgeleon17',
                       'tags': '',
                       'text': 'Beer and soda isn’t a snack, they’re beverages. '
                               'All these look disgusting btw. Lol All can go, I '
                               'can bring my own f… https://t.co/kREpaMJ1O0'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 29, 23),
                       'favCount': 0,
                       'id': 1248000007543894021,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'ChattyKatherine',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 26, 48),
                       'favCount': 0,
                       'id': 1247999356805890048,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'jmachadorn',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 22, 5),
                       'favCount': 1,
                       'id': 1247998170937937920,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'myra539',
                       'tags': '',
                       'text': '@realDonaldTrump Open us with social distancing!  '
                               'Tax deductions for food and beverages for '
                               'businesses again!!!!'},
                      {'created_at': datetime.datetime(2020, 4, 8, 21, 10, 17),
                       'favCount': 0,
                       'id': 1247995203182645249,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'PeterJResistor',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 49, 19),
                       'favCount': 0,
                       'id': 1247989927729680388,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'TOO_LIVE_MILZ',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 27, 20),
                       'favCount': 0,
                       'id': 1247984394838605825,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'GahrSEEuh',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 15),
                       'favCount': 0,
                       'id': 1247981288616865797,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'MABeverage',
                       'tags': ['foodindustry'],
                       'text': 'The #foodindustry provides shoppers with the food, '
                               'beverages and basic necessities they need to '
                               'navigate their live… https://t.co/SuIwygegNF'},
                      {'created_at': datetime.datetime(2020, 4, 8, 20, 4, 17),
                       'favCount': 0,
                       'id': 1247978594758012935,
                       'product_Line': 'Food and beverages',
                       'retweet': 62,
                       'screenname': 'DenaliMorrinson',
                       'tags': ['SNAP'],
                       'text': 'RT @ShannonFreshour: Ohio is making it easier for '
                               '#SNAP families to get food and get an adequate '
                               'amount of food through this crisis. \n'
                               '\n'
                               'And…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 55, 40),
                       'favCount': 0,
                       'id': 1247976424939687936,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'Chelle_Shock3D',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 42),
                       'favCount': 0,
                       'id': 1247972986868686848,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'gabrielssmith',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 41, 3),
                       'favCount': 0,
                       'id': 1247972744953765892,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'atamuotaf',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 25, 41),
                       'favCount': 0,
                       'id': 1247968879927685120,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'brdautremont',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 21, 51),
                       'favCount': 0,
                       'id': 1247967913472135168,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'JLaCocaina',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 8, 35),
                       'favCount': 0,
                       'id': 1247964573484843009,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'LizBraunSun',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 19, 1, 14),
                       'favCount': 0,
                       'id': 1247962725281562624,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'csrlosf35',
                       'tags': '',
                       'text': 'IRVING, Texas — 7-Eleven Inc. has committed nearly '
                               '$95 million to support its franchisees in this '
                               'crucial time as t… https://t.co/wnDTEv0el1'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 45, 6),
                       'favCount': 0,
                       'id': 1247958666688462849,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'BlueWaterHL',
                       'tags': '',
                       'text': "Port Huron's local gem, The Raven Cafe, is still "
                               'serving up their great food and beverages during '
                               'this statewide sh… https://t.co/2yixw1gOPV'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 42, 2),
                       'favCount': 0,
                       'id': 1247957891912552450,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'charlessss5',
                       'tags': '',
                       'text': '@TheOnlyOlogi Food and beverages for my two sons, '
                               "it's crazy out here broke die here.😢"},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 37, 19),
                       'favCount': 0,
                       'id': 1247956707877289995,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'oliviadope',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 32),
                       'favCount': 8,
                       'id': 1247955368082198528,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'CityBeatCincy',
                       'tags': '',
                       'text': 'Ohio Gov. Mike DeWine announced that businesses '
                               'with liquor permits can now sell and deliver two '
                               'packaged drinks pe… https://t.co/EnNeQjbNjp'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 26, 47),
                       'favCount': 0,
                       'id': 1247954057072787456,
                       'product_Line': 'Food and beverages',
                       'retweet': 6,
                       'screenname': 'ville_siesta',
                       'tags': '',
                       'text': 'RT @JeetoCheesus: @AITA_reddit “You can live with '
                               'me. But you are not to enjoy yourself AT ALL for '
                               'any reason. Do not eat food. Do not cons…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 25, 2),
                       'favCount': 1,
                       'id': 1247953616314306561,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'JoshSSchutt',
                       'tags': '',
                       'text': '@ClueHeywood That’s why I stick to @TGIFridays, '
                               'only the finest food and beverages'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 22, 2),
                       'favCount': 0,
                       'id': 1247952862782554112,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'DapaDon',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 18, 49),
                       'favCount': 0,
                       'id': 1247952050543112199,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'Yanaa_eff',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 14, 35),
                       'favCount': 0,
                       'id': 1247950984493428737,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'bashdash4',
                       'tags': '',
                       'text': '15 days until I forego all food and beverages in '
                               'the name of the Lord'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 11, 49),
                       'favCount': 0,
                       'id': 1247950291242278913,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'xShaunieex',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 10, 52),
                       'favCount': 0,
                       'id': 1247950050694762503,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'joebagod',
                       'tags': '',
                       'text': '@SpaceCptZemo You’d fun to hang with and enjoy '
                               'some beverages, food and games. 🤷\u200d♂️'},
                      {'created_at': datetime.datetime(2020, 4, 8, 18, 10, 1),
                       'favCount': 1,
                       'id': 1247949835694555136,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'icicleBsafe',
                       'tags': ['BritishColumbia'],
                       'text': 'Funding has reopened for the #BritishColumbia '
                               'Post-Farm Food Safety Program, offering up to $20K '
                               'to qualifying food… https://t.co/6mmSoWGSoR'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 43, 1),
                       'favCount': 0,
                       'id': 1247943041953992708,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'eilispurcell',
                       'tags': '',
                       'text': '@anaceant @AlanHGoff @jclancy1995 @philipoconnor '
                               'Give over will ya. They are not going to stay in a '
                               'caravan for 2 w… https://t.co/TQuD5ntSnr'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 41, 2),
                       'favCount': 0,
                       'id': 1247942544572416002,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'h0ney_nee',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 26, 8),
                       'favCount': 1,
                       'id': 1247938792872280064,
                       'product_Line': 'Food and beverages',
                       'retweet': 1,
                       'screenname': 'CoastPackingCo',
                       'tags': '',
                       'text': 'Consumers may behave more conservatively and '
                               'cautiously in the months to come, relying on food '
                               'and beverages that p… https://t.co/BJjZ8ePrM3'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 20, 45),
                       'favCount': 0,
                       'id': 1247937437969584128,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'StaccnBenjis',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 14, 23),
                       'favCount': 0,
                       'id': 1247935837775552520,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'alfredogiuria',
                       'tags': '',
                       'text': 'Mamma mía, que grande belleza! Italia por Sophia '
                               "Loren. Barilla pays tribute to 'resilient Italy' "
                               'in ad narrated by… https://t.co/Ds0Xr7fMaW'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 4, 43),
                       'favCount': 0,
                       'id': 1247933401870598156,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'albertllussa',
                       'tags': '',
                       'text': '(b) go to an essential retail outlet for the '
                               'purpose of obtaining items (food, beverages, fuel, '
                               'medicines, medical… https://t.co/6tRnHAONgY'},
                      {'created_at': datetime.datetime(2020, 4, 8, 17, 1, 2),
                       'favCount': 0,
                       'id': 1247932474845528065,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'centeredgesoft',
                       'tags': ['FEC', 'team'],
                       'text': 'Industry experts believe your #FEC should stand '
                               'out through its #team, facility, food/beverages '
                               'and dedication to c… https://t.co/yrv8sGUz4F'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 50, 56),
                       'favCount': 0,
                       'id': 1247929935601123330,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'NancyAl50283383',
                       'tags': '',
                       'text': 'https://t.co/fZRe2zv7J6\n'
                               'Join us online at food packaging 2020 Webinar and '
                               'gain the latest insights on Food and Beve… '
                               'https://t.co/OtDHirTCaH'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 45, 55),
                       'favCount': 0,
                       'id': 1247928673555992577,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'EuroFoodConf',
                       'tags': '',
                       'text': 'https://t.co/LRrRdLXkOL\n'
                               'Join us online at Euro food 2020 Webinar and gain '
                               'the latest insights on Food and Beverages… '
                               'https://t.co/jodlfb0g0n'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 41, 29),
                       'favCount': 0,
                       'id': 1247927557544120322,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'cuddlebugg10',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 41, 14),
                       'favCount': 0,
                       'id': 1247927493492965376,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'TheYazzyB_Show',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 39, 54),
                       'favCount': 0,
                       'id': 1247927156048609280,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'kwillism',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 37, 35),
                       'favCount': 0,
                       'id': 1247926576613863424,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'BIGBADREL',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 36, 7),
                       'favCount': 0,
                       'id': 1247926206542098432,
                       'product_Line': 'Food and beverages',
                       'retweet': 2,
                       'screenname': 'riteaparty',
                       'tags': '',
                       'text': 'RT @RICenterFreedom: Already in Rhode Island, one '
                               'of the Center’s early recommendations has been '
                               'enacted: To allow alcoholic beverages to l…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 35, 39),
                       'favCount': 0,
                       'id': 1247926089130946566,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'shaROCK_',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 31, 29),
                       'favCount': 0,
                       'id': 1247925040454066176,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'mecredes05',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 31, 26),
                       'favCount': 0,
                       'id': 1247925027464339457,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'JenneBroughton',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 29, 56),
                       'favCount': 0,
                       'id': 1247924650954424327,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': '__NahImGood',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 29, 1),
                       'favCount': 0,
                       'id': 1247924418048929792,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'sincerelyskye__',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 27, 25),
                       'favCount': 0,
                       'id': 1247924016557539329,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'OJ_noSimpson',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 19, 11),
                       'favCount': 0,
                       'id': 1247921943996592129,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'yourmomie',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 18, 17),
                       'favCount': 0,
                       'id': 1247921719710449667,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'Bselected',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 11, 28),
                       'favCount': 0,
                       'id': 1247920002545704963,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'YungLost_Rebel',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 11, 28),
                       'favCount': 0,
                       'id': 1247920001996271617,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'Blue_Nox',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 7, 4),
                       'favCount': 1,
                       'id': 1247918895136231430,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'YoungstownCP',
                       'tags': '',
                       'text': 'You wanted it govmikedewine made it possible! We '
                               'are now able to do with takeout food purchase 2 '
                               'takeout beverages… https://t.co/zVJ6Q2MQQ7'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 4, 36),
                       'favCount': 0,
                       'id': 1247918275830460416,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'ShowTimeRick',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 1, 19),
                       'favCount': 0,
                       'id': 1247917449351245824,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'Ibnyei',
                       'tags': ['Liberia'],
                       'text': 'Exempted from the Lockdown in #Liberia are those '
                               'involved with the production, distribution, and '
                               'marketing of food… https://t.co/fyPpZolIL6'},
                      {'created_at': datetime.datetime(2020, 4, 8, 16, 0, 14),
                       'favCount': 0,
                       'id': 1247917175169482754,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'MiissV_',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 59, 56),
                       'favCount': 0,
                       'id': 1247917098539548673,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'onlyfordisplay_',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 59, 39),
                       'favCount': 0,
                       'id': 1247917028624592897,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'JoellasWorld',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 49, 53),
                       'favCount': 0,
                       'id': 1247914572759130112,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'NasBlixky63',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 45, 17),
                       'favCount': 0,
                       'id': 1247913411641585664,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'RaeDiamond',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 44, 56),
                       'favCount': 0,
                       'id': 1247913326236930049,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'cheesewizbandit',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 42, 2),
                       'favCount': 0,
                       'id': 1247912593987158017,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'LikkleBITCH_',
                       'tags': '',
                       'text': 'RT @TOO_LIVE_MILZ: If you or someone you know '
                               'doesn’t have access to FOOD &amp; BEVERAGES during '
                               'this pandemic and are in BROOKLYN let me know…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 41, 21),
                       'favCount': 3,
                       'id': 1247912424818323457,
                       'product_Line': 'Food and beverages',
                       'retweet': 29,
                       'screenname': 'TOO_LIVE_MILZ',
                       'tags': '',
                       'text': 'If you or someone you know doesn’t have access to '
                               'FOOD &amp; BEVERAGES during this pandemic and are '
                               'in BROOKLYN let me… https://t.co/0kxd8afxQ4'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 39, 48),
                       'favCount': 0,
                       'id': 1247912034760626179,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'hotairtools',
                       'tags': '',
                       'text': 'Boom! Heat shrink plastic sleeves, film and tubes. '
                               'Used in the packaging industry for food, beverages '
                               'and much more… https://t.co/77pzQ3YvKO'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 24, 42),
                       'favCount': 0,
                       'id': 1247908232225587204,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'MikeLow55504451',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 17, 28),
                       'favCount': 0,
                       'id': 1247906414670106624,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': 'elau122',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 17, 10),
                       'favCount': 1,
                       'id': 1247906337226448896,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'WesNotWestffs',
                       'tags': '',
                       'text': 'Dear @LovesTravelStop when you make your beverages '
                               '"full service" and cordon off the area you have '
                               'created a food p… https://t.co/qV163knixE'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 7, 36),
                       'favCount': 0,
                       'id': 1247903931176333320,
                       'product_Line': 'Food and beverages',
                       'retweet': 678,
                       'screenname': '7barbie7c',
                       'tags': '',
                       'text': 'RT @Shell_Canada: Our priority is the health &amp; '
                               'safety of everyone affected by this pandemic, but '
                               'we want to do more. In the coming weeks, a…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 7, 26),
                       'favCount': 1,
                       'id': 1247903888734208002,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'rokktober',
                       'tags': '',
                       'text': '@PixelBunny821 Hello!\n'
                               '\n'
                               'My name is October and I’ve been a pixel artist '
                               'for the past two years!\n'
                               '\n'
                               'I like to draw beve… https://t.co/xmR8COVTpX'},
                      {'created_at': datetime.datetime(2020, 4, 8, 15, 2, 16),
                       'favCount': 0,
                       'id': 1247902589699547138,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'WABevAssn',
                       'tags': ['foodindustry'],
                       'text': 'The #foodindustry provides shoppers with the food, '
                               'beverages and basic necessities they need to '
                               'navigate their live… https://t.co/EVAUWvuo8N'},
                      {'created_at': datetime.datetime(2020, 4, 8, 14, 50, 44),
                       'favCount': 0,
                       'id': 1247899687429869569,
                       'product_Line': 'Food and beverages',
                       'retweet': 1,
                       'screenname': 'BalinwillinFarm',
                       'tags': ['Doneraile', 'contactfree'],
                       'text': 'RT @8degreesbrewing: @NeighbourFoodIE #Doneraile '
                               'is open until this evening - order online for '
                               '#contactfree delivery on Friday. Lots of #lo…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 14, 47, 19),
                       'favCount': 0,
                       'id': 1247898825131347970,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'sirtaj03',
                       'tags': '',
                       'text': 'Iont understand how these rich mfs complaining '
                               'about being “quarantined.” They literally have '
                               'everything. House wit… https://t.co/HuVhnWmOtw'},
                      {'created_at': datetime.datetime(2020, 4, 8, 14, 45, 41),
                       'favCount': 2,
                       'id': 1247898414232166404,
                       'product_Line': 'Food and beverages',
                       'retweet': 1,
                       'screenname': '8degreesbrewing',
                       'tags': ['Doneraile', 'contactfree'],
                       'text': '@NeighbourFoodIE #Doneraile is open until this '
                               'evening - order online for #contactfree delivery '
                               'on Friday. Lots of… https://t.co/acLGS6lEsP'},
                      {'created_at': datetime.datetime(2020, 4, 8, 14, 34, 56),
                       'favCount': 0,
                       'id': 1247895710885789697,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'BMmamarope',
                       'tags': ['Biotechnology'],
                       'text': 'Common Industries which uses #Biotechnology are: '
                               'medical industry, food and beverages, agriculture, '
                               'nutrition.'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 39, 4),
                       'favCount': 1,
                       'id': 1247881648382894080,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'ElodieBrassard',
                       'tags': '',
                       'text': '@Ferocious__B @bigmeepemoppe @CHamiltoChem Also '
                               'drinking basic beverages alters you digestion '
                               'because it mixes with… https://t.co/xDLqaXU2IH'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 37, 6),
                       'favCount': 0,
                       'id': 1247881154050568193,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'endjonesactPR',
                       'tags': '',
                       'text': '"The Jones Act adds an estimated\xa0$300\xa0to the '
                               'annual cost of food and beverages a typical '
                               'household in Puerto Rico p… '
                               'https://t.co/EPHK2sYpEX'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 19, 7),
                       'favCount': 0,
                       'id': 1247876630447603714,
                       'product_Line': 'Food and beverages',
                       'retweet': 1,
                       'screenname': 'MindfuLY',
                       'tags': '',
                       'text': 'RT @NatLauter: ONroute service centres give free '
                               'coffee to truck drivers on Wednesday\n'
                               '\n'
                               'Providing 24-7 access to fuel, washrooms, '
                               'food/bever…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 15, 41),
                       'favCount': 3,
                       'id': 1247875764625772544,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'sporthiatus',
                       'tags': '',
                       'text': 'Food and beverages set up at my desk.  Ready to '
                               'roll, @sbjsbd. https://t.co/Q8eQUiI6w7'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 14, 10),
                       'favCount': 0,
                       'id': 1247875381572624390,
                       'product_Line': 'Food and beverages',
                       'retweet': 1,
                       'screenname': 'SunshineLK10',
                       'tags': '',
                       'text': 'RT @c0lettea: @reneenilseb @SunshineLK10 Illicit '
                               'fentanyl comes in liquid, powder and counterfeit '
                               "pill forms. It's not a patch those are pr…"},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 10, 20),
                       'favCount': 1,
                       'id': 1247874419697074176,
                       'product_Line': 'Food and beverages',
                       'retweet': 1,
                       'screenname': 'c0lettea',
                       'tags': '',
                       'text': '@reneenilseb @SunshineLK10 Illicit fentanyl comes '
                               "in liquid, powder and counterfeit pill forms. It's "
                               'not a patch th… https://t.co/ySQDENbBJe'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 7, 26),
                       'favCount': 4,
                       'id': 1247873690928365568,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'TrumpWarriorJB',
                       'tags': '',
                       'text': '@LILIFIFIELD @OANN Jamie, Inside sales rep for '
                               'company with an electrification and automation '
                               'technology manufactur… https://t.co/5qrUik4gKo'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 4, 23),
                       'favCount': 0,
                       'id': 1247872919818076160,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'broadstreetlag',
                       'tags': ['food',
                                'beverages',
                                'tobacco',
                                'stock',
                                'Nigeria',
                                'long'],
                       'text': '$cad a #food #beverages and #tobacco #stock listed '
                               'in #Nigeria has shown its #long play from its '
                               'N5.15 low closing… https://t.co/yxOiFZUrWo'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 3, 48),
                       'favCount': 0,
                       'id': 1247872774678253568,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'QizBalqis',
                       'tags': '',
                       'text': '@thekhayalan15 Caregory tu dia automatically akan '
                               'separate to their own group . Macam tesco '
                               'starbucks semua under f… https://t.co/rNG2f6P5iG'},
                      {'created_at': datetime.datetime(2020, 4, 8, 13, 0, 59),
                       'favCount': 1,
                       'id': 1247872066977579010,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'hamizan_huge',
                       'tags': '',
                       'text': '@firzanahbalqis @syhmisamsol Food and beverages '
                               'are the best bae EvER'},
                      {'created_at': datetime.datetime(2020, 4, 8, 12, 59, 15),
                       'favCount': 0,
                       'id': 1247871630120009728,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'DubaiChronicle',
                       'tags': '',
                       'text': '@IAmMrBrightside Even in the supermarkets food and '
                               'beverages are more expensive these days...'},
                      {'created_at': datetime.datetime(2020, 4, 8, 12, 53, 45),
                       'favCount': 5,
                       'id': 1247870245630550016,
                       'product_Line': 'Food and beverages',
                       'retweet': 1,
                       'screenname': 'NatLauter',
                       'tags': '',
                       'text': 'ONroute service centres give free coffee to truck '
                               'drivers on Wednesday\n'
                               '\n'
                               'Providing 24-7 access to fuel, washrooms, f… '
                               'https://t.co/2tPhqA8T8b'},
                      {'created_at': datetime.datetime(2020, 4, 8, 12, 47, 24),
                       'favCount': 1,
                       'id': 1247868648737386498,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'aspire_speaks',
                       'tags': '',
                       'text': '@ilz_zli Having said this, best source of vitamin '
                               'C are: \n'
                               '\n'
                               '1) Citrus Fruits (Oranges, Grapefruit, Lemon &amp; '
                               'Lime) and… https://t.co/xm5qy7x4Kg'},
                      {'created_at': datetime.datetime(2020, 4, 8, 12, 1, 49),
                       'favCount': 0,
                       'id': 1247857177110269952,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'MediocrisRegina',
                       'tags': '',
                       'text': '@LeprechaunCurse As their food and beverages '
                               'arrive she smiles over at her companion. With '
                               'Seamus back in her life… https://t.co/iGZ8bG8FQV'},
                      {'created_at': datetime.datetime(2020, 4, 8, 11, 37),
                       'favCount': 0,
                       'id': 1247850932056064000,
                       'product_Line': 'Food and beverages',
                       'retweet': 2,
                       'screenname': 'infointuitive',
                       'tags': '',
                       'text': 'RT @environmentza: The distribution of the food '
                               'parcels for waste pickers is a partnership between '
                               'government and Coca-Cola Beverages South…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 11, 1, 37),
                       'favCount': 0,
                       'id': 1247842028324630529,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'PopRoxas',
                       'tags': '',
                       'text': '@Fact 1) Food Dyes\n'
                               '2) Microwave Popcorn\n'
                               '3) Red Meat\n'
                               '4) Salt-Cured and Pickled Foods\n'
                               '5) Refined Carbohydrates and Su… '
                               'https://t.co/ujay1QiSNQ'},
                      {'created_at': datetime.datetime(2020, 4, 8, 11, 0, 39),
                       'favCount': 0,
                       'id': 1247841781468680192,
                       'product_Line': 'Food and beverages',
                       'retweet': 6,
                       'screenname': '_foodmicrobio',
                       'tags': '',
                       'text': 'RT @FoodIndAsia: FIA and the ASEAN Food and '
                               'Beverage Alliance (AFBA) are jointly calling upon '
                               'governments across the region to ensure the u…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 10, 57, 39),
                       'favCount': 0,
                       'id': 1247841028096372737,
                       'product_Line': 'Food and beverages',
                       'retweet': 4,
                       'screenname': 'Sanador',
                       'tags': '',
                       'text': 'RT @OwlchemyLabs: Sometimes all you need is a '
                               'staycation. Video games, junk food, and an endless '
                               'supply of caffeinated sugary beverages. Ga…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 10, 40, 42),
                       'favCount': 0,
                       'id': 1247836761310539781,
                       'product_Line': 'Food and beverages',
                       'retweet': 0,
                       'screenname': 'institutiGAP',
                       'tags': '',
                       'text': 'Kosovo’s exports to Albania include manufacturing '
                               'products from industries such as: alcoholic and '
                               'non-alcoholic bev… https://t.co/ludTVvCrTt'}]}
    {'ProductLine': 'Fashion accessories',
     'ProductLine_Rating': {},
     '_id': ObjectId('5e8e5532b5101d7d130493cc'),
     'customerDictionary': {'Address': '6 Norwood Grove',
                            'Date_Of_Purchase': '2/18/19',
                            'FirstName': 'Mi',
                            'LastName': 'Richan',
                            'Payment_Type': 'Cash',
                            'PhoneNumber1': '01451-785624',
                            'PhoneNumber2': '01202-738406',
                            'Time': '13:28',
                            'Total_amount': 649.299,
                            'amount_billed': 649.299,
                            'city': 'Orlando',
                            'customer_type': 'Member',
                            'email': 'mi@hotmail.com',
                            'gender': 'Female',
                            'state': 'FL'},
     'productTweet': [{'created_at': datetime.datetime(2020, 4, 8, 22, 49, 36),
                       'favCount': 0,
                       'id': 1248020196154994689,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'CherylD93415695',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/jlve6tkvxd'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 49, 31),
                       'favCount': 0,
                       'id': 1248020176257208321,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'LeslieL03210376',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/DP8dUHMly5'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 49, 11),
                       'favCount': 0,
                       'id': 1248020089150017537,
                       'product_Line': 'Fashion accessories',
                       'retweet': 77,
                       'screenname': 'liacallahan4',
                       'tags': ['Giveaway', 'friends'],
                       'text': "RT @SashkaCo: #Giveaway!! We're giving away 3\xa0 "
                               'Sashka Co. bracelets to 3 #friends!\n'
                               'To Enter: \n'
                               '1. Follow @SashkaCo\n'
                               '2. Like this post\n'
                               '3. Tag…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 48, 40),
                       'favCount': 0,
                       'id': 1248019962305765376,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'FashionWarehou2',
                       'tags': '',
                       'text': 'Fashion Tiny Dainty Heart Initial Necklace '
                               'Personalized Letter Necklace Name Jewelry for '
                               'women accessories girlfrie… '
                               'https://t.co/Iw7X7cL3IQ'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 48, 25),
                       'favCount': 0,
                       'id': 1248019897386438656,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'MissyNordone',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/mcPgecPT7b'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 47, 8),
                       'favCount': 0,
                       'id': 1248019576220049409,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'YvonneM03742581',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/gqIZMY263v'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 45, 50),
                       'favCount': 0,
                       'id': 1248019248753987584,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'TaraPac53738921',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/u5dzN9I7e5'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 44, 40),
                       'favCount': 0,
                       'id': 1248018956402610176,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Kristen44889792',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/FUt5V8w1X9'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 44, 14),
                       'favCount': 0,
                       'id': 1248018843961659392,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Kimberl54012436',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/MIY6jOP4NS'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 43, 50),
                       'favCount': 0,
                       'id': 1248018746473476096,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'DebbieJ65020701',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/IGuI9cTlFk'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 43, 34),
                       'favCount': 0,
                       'id': 1248018679691763714,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'JacquelineCarf3',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/dVYI3GISf5'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 43, 16),
                       'favCount': 0,
                       'id': 1248018602071961600,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AnnieJa64908448',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/z50EUoTF7k'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 42, 7),
                       'favCount': 0,
                       'id': 1248018314363719680,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'DanielaPatel14',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/80m3T9xwiC'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 41, 53),
                       'favCount': 0,
                       'id': 1248018253999271937,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'ReginaH94655472',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/WNmqv6n8Iw'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 41, 31),
                       'favCount': 0,
                       'id': 1248018162336952320,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Lynnett76799182',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/1YhEo2V0Xi'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 40, 53),
                       'favCount': 0,
                       'id': 1248018002794049536,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'KatieEd83444316',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/NrCYGpUNcj'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 40, 22),
                       'favCount': 0,
                       'id': 1248017870824460288,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'MogelAmanda',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/vw2ezpTGJY'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 39, 50),
                       'favCount': 0,
                       'id': 1248017739387514881,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AmandaO21831843',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/qVfPDUk6uo'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 39, 45),
                       'favCount': 0,
                       'id': 1248017718848000001,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Jessica39352481',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/wOs0bQlXcd'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 39, 45),
                       'favCount': 0,
                       'id': 1248017717782691841,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AndreaB70213136',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/aTEaedQ3JY'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 39, 42),
                       'favCount': 0,
                       'id': 1248017706328064002,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Monique73298032',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/4yXxbbyMJT'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 38, 19),
                       'favCount': 0,
                       'id': 1248017356405633025,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AmberMi50225425',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/2xYcHsoaAm'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 37, 57),
                       'favCount': 0,
                       'id': 1248017265909350400,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Shannon61190986',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/Fh7BvMY3jw'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 37, 49),
                       'favCount': 0,
                       'id': 1248017230450704387,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Whitney33869058',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/jhhMUBDooc'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 37, 48),
                       'favCount': 0,
                       'id': 1248017226072023042,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'MonicaB82065975',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/fuzxDhBNCL'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 35, 57),
                       'favCount': 0,
                       'id': 1248016761963704321,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'FrittsBrandi',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/TVTD3Jtss5'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 34, 57),
                       'favCount': 0,
                       'id': 1248016508413865984,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'KellyTh80014960',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/Cz3oes57nQ'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 34, 7),
                       'favCount': 0,
                       'id': 1248016299508133888,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'BidorBuy247',
                       'tags': '',
                       'text': 'Trending 2020 Rimless Sunglasses Women Fashion '
                               'Tears Shape Shades Sun Glasses '
                               'https://t.co/hIqU2Xx9GQ https://t.co/B1oAmiJiRQ'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 33, 43),
                       'favCount': 0,
                       'id': 1248016200421896192,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'MissyNordone',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/znXDQDi5Dm'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 33, 6),
                       'favCount': 0,
                       'id': 1248016045140406273,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'LeslieL03210376',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/NabaBHpy7s'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 32, 1),
                       'favCount': 0,
                       'id': 1248015772229570560,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'CherylD93415695',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/lIb17eeDr0'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 31, 54),
                       'favCount': 0,
                       'id': 1248015741728645120,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'GODUSTYLE',
                       'tags': ['fashion',
                                'party',
                                'dress',
                                'look',
                                'style',
                                'details',
                                'accessories',
                                'influencer',
                                'outfit',
                                'inspiration'],
                       'text': '@oliviaculpo #fashion dior #party #dress #look '
                               '#style #details #accessories #influencer #outfit '
                               '#inspiration… https://t.co/oMRk5H2WKM'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 31, 26),
                       'favCount': 0,
                       'id': 1248015624472682497,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'YvonneM03742581',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/utPhiq2evx'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 31, 22),
                       'favCount': 1,
                       'id': 1248015607183716352,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'DebbieJ65020701',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/Dy1Lmsqa98'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 30, 45),
                       'favCount': 0,
                       'id': 1248015450882990081,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'cudaeducation',
                       'tags': ['fashionbusiness'],
                       'text': '2020 FASHION BIZ REPORT:  Get real statistics, '
                               'facts and information about the #fashionbusiness '
                               '|… https://t.co/sMgHg3PBZV'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 30, 21),
                       'favCount': 0,
                       'id': 1248015350492348417,
                       'product_Line': 'Fashion accessories',
                       'retweet': 3,
                       'screenname': 'myfoodfantasy69',
                       'tags': ['Fashion', 'Style', 'FrizeMedia'],
                       'text': 'RT @Charlesfrize: Reading about this: #Fashion - '
                               'What Are The Essential Accessories #Style '
                               '#FrizeMedia - https://t.co/LShAZPyY8M @Charlesfr…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 30, 9),
                       'favCount': 0,
                       'id': 1248015303243513858,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'JacquelineCarf3',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/wioH7vLgDE'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 29, 50),
                       'favCount': 0,
                       'id': 1248015223400742915,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Kristen44889792',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/xigYHqsaAt'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 29, 11),
                       'favCount': 0,
                       'id': 1248015057851568128,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AnnieJa64908448',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/ITct3qsaGt'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 28, 58),
                       'favCount': 0,
                       'id': 1248015004235747328,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Kimberl54012436',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/kHKi7u63nA'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 28, 51),
                       'favCount': 0,
                       'id': 1248014972417789952,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'TaraPac53738921',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/CLtderPPX9'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 28, 47),
                       'favCount': 0,
                       'id': 1248014958853410816,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Lynnett76799182',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/nftNhsbtEC'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 28, 3),
                       'favCount': 0,
                       'id': 1248014774022991872,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'KatieEd83444316',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/0D54fSPlP1'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 27, 48),
                       'favCount': 0,
                       'id': 1248014710923907072,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'ReginaH94655472',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/yD8yqi5Xpg'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 27, 32),
                       'favCount': 0,
                       'id': 1248014644884594693,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AndreaB70213136',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/gAbLVwhs7v'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 27, 8),
                       'favCount': 0,
                       'id': 1248014540362547200,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'BluediamondsGS',
                       'tags': ['handmade', 'statement'],
                       'text': '925 Sterling Silver Ladies Large Oval Ring and '
                               'Unakite Cabochon Sizes J-R | eBay #handmade '
                               '#statement… https://t.co/HNGsP482G2'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 27, 3),
                       'favCount': 0,
                       'id': 1248014521056120834,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Jessica39352481',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/vfEjj26Y6T'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 26, 28),
                       'favCount': 0,
                       'id': 1248014376147116032,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'maryhigley1',
                       'tags': '',
                       'text': 'Brooch Pin Broche Clear Rhinestones Fashion '
                               'Wedding Vintage Jewelry Vendimia Joyeria Bridal '
                               'Sash Accessories Hollyw… https://t.co/oauBpABRzR'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 25, 9),
                       'favCount': 0,
                       'id': 1248014044948127744,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'MogelAmanda',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/jrhU1G87Dk'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 23, 33),
                       'favCount': 0,
                       'id': 1248013638448734208,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Monique73298032',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/1KsezTFPuj'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 23, 1),
                       'favCount': 0,
                       'id': 1248013506445631489,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'YeniExpoSEO',
                       'tags': ['Toptan', 'turkeyexport'],
                       'text': 'Women Chic Long Sleeve Long Length Jacket  38-48 '
                               'Jk 01 https://t.co/EDffSnL00p #Toptan '
                               '#turkeyexport https://t.co/94TEjypOjN'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 22, 58),
                       'favCount': 0,
                       'id': 1248013492222746624,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Whitney33869058',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/JECnc49QDJ'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 22, 38),
                       'favCount': 0,
                       'id': 1248013410911932418,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AmandaO21831843',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/mwHIMj4bMP'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 22, 35),
                       'favCount': 0,
                       'id': 1248013397209149443,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Shannon61190986',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/ORVZpXnVgX'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 22, 27),
                       'favCount': 0,
                       'id': 1248013363948343297,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'MonicaB82065975',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/e6vC3HcbW3'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 22, 11),
                       'favCount': 0,
                       'id': 1248013298408120320,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'DanielaPatel14',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/fLqIfOxOoA'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 22, 5),
                       'favCount': 0,
                       'id': 1248013269660360707,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'FrittsBrandi',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/XEt9TL3L7a'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 21, 29),
                       'favCount': 0,
                       'id': 1248013121496612864,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AmberMi50225425',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/j4dhDoYkuy'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 21, 25),
                       'favCount': 0,
                       'id': 1248013101619802114,
                       'product_Line': 'Fashion accessories',
                       'retweet': 77,
                       'screenname': 'hannah15mini',
                       'tags': ['Giveaway', 'friends'],
                       'text': "RT @SashkaCo: #Giveaway!! We're giving away 3\xa0 "
                               'Sashka Co. bracelets to 3 #friends!\n'
                               'To Enter: \n'
                               '1. Follow @SashkaCo\n'
                               '2. Like this post\n'
                               '3. Tag…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 21, 14),
                       'favCount': 0,
                       'id': 1248013056950423552,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'MissyNordone',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/kdZhJD07lD'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 19, 17),
                       'favCount': 0,
                       'id': 1248012564929232896,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'DebbieJ65020701',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/NfeAGb947M'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 18, 56),
                       'favCount': 0,
                       'id': 1248012476630749186,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'LeslieL03210376',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/Tc4be9GPCz'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 18, 9),
                       'favCount': 0,
                       'id': 1248012280945467393,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'CherylD93415695',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/qFuI9wHKE0'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 17, 43),
                       'favCount': 0,
                       'id': 1248012172858241026,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'TaraPac53738921',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/vlnFfbFc6E'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 17, 18),
                       'favCount': 0,
                       'id': 1248012069099565056,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'MissAyoDele',
                       'tags': ['model',
                                'style',
                                'swag',
                                'classy',
                                'scarifications',
                                'love',
                                'beautiful',
                                'beautifulpeople',
                                'beauty',
                                'amazing',
                                'fashion',
                                'woman'],
                       'text': '#model #style #swag #classy #scarifications #love '
                               '#beautiful #beautifulpeople #beauty #amazing \n'
                               '#fashion #woman… https://t.co/JfDybwdUcz'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 17, 9),
                       'favCount': 0,
                       'id': 1248012031430516736,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'KellyTh80014960',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/u2GTluyyVY'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 17, 6),
                       'favCount': 0,
                       'id': 1248012016897232896,
                       'product_Line': 'Fashion accessories',
                       'retweet': 5,
                       'screenname': 'Lumpimp',
                       'tags': '',
                       'text': 'RT @acnhivan: 🌸 FASHION ITEMS SALE!! 🌸\n'
                               "I'm looking to sell some of my clothing and "
                               "accessories. After an item is sold, it's gone. I "
                               'will or…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 17, 1),
                       'favCount': 0,
                       'id': 1248011994315157505,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'CosmoOAaaarty',
                       'tags': '',
                       'text': 'Accessories - Hair - Fashion - luxe '
                               'https://t.co/Zsbm9VNool'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 16, 42),
                       'favCount': 0,
                       'id': 1248011916611473408,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AnnieJa64908448',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/ANkFo29Kk5'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 16, 37),
                       'favCount': 0,
                       'id': 1248011895660920834,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'YvonneM03742581',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/I8ZwJCBGJ4'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 16),
                       'favCount': 0,
                       'id': 1248011740056440833,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'JacquelineCarf3',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/pmnEvvNKAi'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 15, 42),
                       'favCount': 0,
                       'id': 1248011665808871424,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Lynnett76799182',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/wYyPxHHlNW'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 14, 45),
                       'favCount': 0,
                       'id': 1248011427618549760,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Kristen44889792',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/zBJB2cboX5'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 14, 35),
                       'favCount': 0,
                       'id': 1248011384773722112,
                       'product_Line': 'Fashion accessories',
                       'retweet': 77,
                       'screenname': 'ranjpower',
                       'tags': ['Giveaway', 'friends'],
                       'text': "RT @SashkaCo: #Giveaway!! We're giving away 3\xa0 "
                               'Sashka Co. bracelets to 3 #friends!\n'
                               'To Enter: \n'
                               '1. Follow @SashkaCo\n'
                               '2. Like this post\n'
                               '3. Tag…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 13, 38),
                       'favCount': 0,
                       'id': 1248011146138812416,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'ZenshyS',
                       'tags': '',
                       'text': 'Leather Wrap Bracelet (7 Colors) 17.99$\n'
                               'Get it here: https://t.co/4B59ObPZrn\n'
                               'Like &amp; Follow Us @zenshys \n'
                               '(50%SALE &amp;… https://t.co/usmOgQ82xk'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 13, 36),
                       'favCount': 0,
                       'id': 1248011136462508032,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AndreaB70213136',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/jINb8saipm'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 13, 35),
                       'favCount': 0,
                       'id': 1248011134134677505,
                       'product_Line': 'Fashion accessories',
                       'retweet': 77,
                       'screenname': 'pharrispenny',
                       'tags': ['Giveaway', 'friends'],
                       'text': "RT @SashkaCo: #Giveaway!! We're giving away 3\xa0 "
                               'Sashka Co. bracelets to 3 #friends!\n'
                               'To Enter: \n'
                               '1. Follow @SashkaCo\n'
                               '2. Like this post\n'
                               '3. Tag…'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 11, 48),
                       'favCount': 0,
                       'id': 1248010684509515777,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Kimberl54012436',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/dmmRNACfFs'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 11, 46),
                       'favCount': 0,
                       'id': 1248010674132770816,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'KatieEd83444316',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/mT2VEKcE2C'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 11, 21),
                       'favCount': 1,
                       'id': 1248010571095478274,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'gossamerfae',
                       'tags': ['Handmade'],
                       'text': 'Check out Cross Choker Necklace Handmade '
                               'Adjustable Silver Accessories Fashion Black '
                               '#Handmade https://t.co/H5tbNBvjrV via @eBay'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 10, 29),
                       'favCount': 0,
                       'id': 1248010351540461568,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'BayBlingBeauty',
                       'tags': '',
                       'text': 'All we can think of is summer ☀️ Body chains are '
                               'the perfect way to accessorize a simple outfit 😍 '
                               'Link in bio 👆🏻\n'
                               '\n'
                               '•… https://t.co/lYzakUYxfi'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 10, 18),
                       'favCount': 0,
                       'id': 1248010303901560832,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Whitney33869058',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/0sIu7cxlNc'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 9, 54),
                       'favCount': 0,
                       'id': 1248010205016670210,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Jessica39352481',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/NDoKMSMZth'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 9, 31),
                       'favCount': 0,
                       'id': 1248010109579493377,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Shannon61190986',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/ELwLoPY1tY'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 9, 14),
                       'favCount': 0,
                       'id': 1248010039106760707,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'ReginaH94655472',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/PGI6gkwf8n'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 8, 59),
                       'favCount': 0,
                       'id': 1248009974879367168,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AmberMi50225425',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/kauuGdYqeX'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 7, 48),
                       'favCount': 1,
                       'id': 1248009675347394560,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'BrendaJBear1',
                       'tags': '',
                       'text': '@cat01cat01cat01 @GregariousGus @SheilaMSpence1 '
                               '@Blutospin @OffTheLeashFP @GracieRoadster '
                               '@basset_bella… https://t.co/KsvZZBLpsx'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 7, 35),
                       'favCount': 0,
                       'id': 1248009620699807745,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'DanielaPatel14',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/QuppsoqLgO'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 7, 33),
                       'favCount': 0,
                       'id': 1248009615674970112,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Monique73298032',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/BYXEAOEXxV'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 5, 53),
                       'favCount': 0,
                       'id': 1248009192448782338,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AmandaO21831843',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/x5GGaZbP2v'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 4, 18),
                       'favCount': 1,
                       'id': 1248008795227213825,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'RLushington',
                       'tags': ['friends',
                                'style',
                                'beautiful',
                                'jewelry',
                                'bracelet',
                                'fashion',
                                'accessories',
                                'trend'],
                       'text': '@WhatsupLiz @cookiegigan @ForeverAngel26 #friends '
                               '#style #beautiful #jewelry #bracelet #fashion '
                               '#accessories #trend… https://t.co/cutgscebgc'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 4),
                       'favCount': 0,
                       'id': 1248008720442773504,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'YvonneM03742581',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/6qTkT3l3wL'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 3, 59),
                       'favCount': 0,
                       'id': 1248008717829734401,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'CherylD93415695',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/10wvTtsYsW'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 3, 3),
                       'favCount': 0,
                       'id': 1248008483275849729,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'MogelAmanda',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/ZLMxqgMN5Q'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 3, 2),
                       'favCount': 0,
                       'id': 1248008477475139584,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'KellyTh80014960',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/oIemi7AqXU'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 3),
                       'favCount': 0,
                       'id': 1248008469455589377,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'MissyNordone',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/90pNvym1BN'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 2, 39),
                       'favCount': 0,
                       'id': 1248008379236139009,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'FrittsBrandi',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/prKRdMvS0K'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 2, 29),
                       'favCount': 0,
                       'id': 1248008339952291841,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Lynnett76799182',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/ts1MBTiuLv'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 2, 29),
                       'favCount': 0,
                       'id': 1248008337070772227,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'AnnieJa64908448',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/Ry0l2jSsZx'},
                      {'created_at': datetime.datetime(2020, 4, 8, 22, 2, 5),
                       'favCount': 0,
                       'id': 1248008236113903618,
                       'product_Line': 'Fashion accessories',
                       'retweet': 0,
                       'screenname': 'Kristen44889792',
                       'tags': '',
                       'text': 'Fashion, Beauty, Home Accessories &amp; Baby '
                               'centrepointstores\n'
                               '\n'
                               'Buy gift cards and use them online with ease \n'
                               'orders wi… https://t.co/rPGDkutYRQ'}]}
    

### Answering all the Database questions for the assignment - 


### Q1> - What are tags are associated with a person, place or thing?


```python
def getHashTags():
    tweetCount = set()
    for i in range(len(ProductLineDoc)):
        productLine = mycol.find({})[i]['ProductLine']
        print(productLine)
        ProductLineTweets = mycol.find({})[i]['productTweet']
        for twt in ProductLineTweets:
            if len(twt['tags']) != 0:
                for t in twt['tags']:
                    tweetCount.add(t)
        print(tweetCount)
        tweetCount.clear()
        print()
        print()
getHashTags()
```

    Health and beauty
    {'health', 'Health', 'home', 'coronavirus', 'immunityboost', 'wellness', 'writingcommunity', 'COVID19', 'watch', 'covid19', 'exercise', 'Microplastics', 'beauty', 'gadget', 'clothing', 'sleep', 'TextureHunterGatherer', 'Woonsocket', 'Diet'}
    
    
    Electronic accessories
    {'hashtag3', 'phone_chargersSpring', 'electronic_accessories', 'phone_cables', 'Gaming', 'phone_glass', 'montblanc', 'phone_case'}
    
    
    Home and lifestyle
    {'coronavirus', 'disinfect', 'clean'}
    
    
    Sports and travel
    {'EQRoundtable', 'business', 'club', 'CoronaOutbreak', 'UK', 'sports', 'COVID19', 'religious', 'vacations', 'Travel', 'travels', 'travel', 'China', 'coronavirus'}
    
    
    Food and beverages
    {'beverages', 'contactfree', 'long', 'Biotechnology', 'Doneraile', 'Nigeria', 'SNAP', 'FEC', 'tobacco', 'foodindustry', 'stock', 'food', 'TheFive', 'Liberia', 'team', 'BritishColumbia'}
    
    
    Fashion accessories
    {'influencer', 'classy', 'scarifications', 'model', 'beautiful', 'look', 'fashionbusiness', 'style', 'jewelry', 'bracelet', 'handmade', 'trend', 'accessories', 'Style', 'turkeyexport', 'statement', 'FrizeMedia', 'dress', 'amazing', 'friends', 'beautifulpeople', 'inspiration', 'swag', 'Handmade', 'beauty', 'outfit', 'woman', 'Giveaway', 'Toptan', 'party', 'details', 'Fashion', 'fashion', 'love'}
    
    
    

### Q2>-  What social media users are like other social media users in your domain?


```python
def similarSocialMediaUsers(productLine, user):
    similarUsers = set()
    tweets = mycol.find_one({'ProductLine':productLine})['productTweet']
    for tweet in tweets:
        if tweet["screenname"] == user:
            tags = tweet['tags']
            break
    print(tags)
    for tag in tags:
        for tweet in tweets:
            if tag in tweet['tags']:
                similarUsers.add(tweet['screenname'])
    print(similarUsers)
```


```python
similarSocialMediaUsers("Health and beauty","gd959098")
```

    ['writingcommunity']
    {'ccthewriter1', 'gd959098', 'KatinaParon', 'weischoice'}
    

### Q3>  What people, places or things are popular in your domain?


```python
#Tweet Count within a time range
trending ={}
def trendingDomain(productLine):
    startDate = datetime.datetime(2020, 4, 8, 8, 0, 0)
    endDate =   datetime.datetime(2020, 4, 8, 9, 0, 0)
    numbTweets = 0
    retweets = 0;
    tweets = mycol.find_one({'ProductLine':productLine})['productTweet']
    for tweet in tweets:
        if (tweet['created_at'] > startDate) and (tweet['created_at'] > endDate):
            numbTweets=numbTweets+1
            retweets = retweets+tweet['retweet']
    trend = numbTweets+retweets
    trending[productLine] = trend
```


```python
for product in productLines:
    trendingDomain(product)
```


```python
sortedTrendDictionary = sorted(trending.items(), key=lambda x: x[1], reverse=True)  
```


```python
print(trending)
```

    {'Food and beverages': 13235, 'Electronic accessories': 101, 'Sports and travel': 571, 'Home and lifestyle': 7746, 'Fashion accessories': 416, 'Health and beauty': 613}
    


```python
print(sortedTrendDictionary)
```

    [('Food and beverages', 13235), ('Home and lifestyle', 7746), ('Health and beauty', 613), ('Sports and travel', 571), ('Fashion accessories', 416), ('Electronic accessories', 101)]
    

#### The Most Popular productLine is -


```python
sortedTrendDictionary[0]
```




    ('Food and beverages', 13235)



### UserName and the tags associated with that user


```python
tweets = mycol.find_one({'ProductLine':"Health and beauty"})['productTweet']
for tweet in tweets:
    print(tweet['screenname'],tweet['tags'])
```

    GrayskyX 
    velocipedus 
    mvsiii71 
    Treuburg 
    JackStewartCham 
    dawn27331807 
    TheVibeDealer3 
    HollyThomas61 ['covid19', 'immunityboost', 'home']
    EmanHabbash 
    ewhoma_ 
    USIIV 
    BidorBuy247 
    Its_3One4 
    Ali_yeganeh_s 
    Gentleman__7 
    TraceyBenmore 
    InStyle 
    4ChrissyO 
    tea_natasha 
    Raul38557638 
    AALawnLandscape 
    Beauty_Rushhhh 
    ah_marachy 
    tania_hanyu ['Health']
    _ataei 
    ReGenFriends 
    tmj_rip_adv ['Woonsocket']
    kaelynforde 
    ccthewriter1 ['writingcommunity']
    gd959098 ['writingcommunity']
    Crisp_Mag 
    Carols_Garcia 
    CeciliaDaForno1 
    BoBAUall 
    KatinaParon ['writingcommunity']
    Greenhouses1 
    ElleCanada 
    ArnicareUSA ['Diet', 'exercise', 'sleep']
    honkide 
    weischoice ['writingcommunity']
    tomwlsn31 ['COVID19']
    BidorBuy247 
    PRJournoRequest 
    Shahinaz6 
    ladyboarder9669 ['covid19', 'immunityboost', 'home']
    se23_tweets ['TextureHunterGatherer']
    JournalistJill 
    seuoceancleanup ['Microplastics']
    Honeysblood 
    lynnettepeck 
    LizAtkin ['TextureHunterGatherer']
    buynbuyshop ['health', 'clothing', 'gadget', 'beauty', 'watch']
    SomethingDigitl ['beauty', 'health', 'wellness']
    drjessigold 
    valiant_beauty ['coronavirus']
    biyounaruhodo 
    AH_Fasihi 
    HummerSilver 
    UtahFamilyPharm 
    mdad8200 
    

### Q4> What people, places or things are popular in your domain?


```python
def popularInDomain(productLine):
    print("The domain is -----> "+productLine)
    popularity = {}
    tweets = mycol.find_one({'ProductLine':productLine})['productTweet']
    for tweet in tweets:
        for tag in tweet['tags']:
            if tag in popularity:
                popularity[tag] = popularity[tag]+1
            else:
                popularity[tag] = 1
                
    sortedDictionary = sorted(popularity.items(), key=lambda x: x[1], reverse=True)    
    #Printing top 5 in every domain -
    if len(popularity)<5:
        rg = len(popularity)
    else:
        rg = 5
    for x in range(0,rg):
        print(sortedDictionary[x][0])

```


```python
for i in range(len(ProductLineDoc)):
    productLine = mycol.find({})[i]['ProductLine']
    popularInDomain(productLine)
```

    The domain is -----> Health and beauty
    writingcommunity
    covid19
    immunityboost
    home
    TextureHunterGatherer
    The domain is -----> Electronic accessories
    montblanc
    electronic_accessories
    phone_case
    phone_glass
    phone_cables
    The domain is -----> Home and lifestyle
    clean
    disinfect
    coronavirus
    The domain is -----> Sports and travel
    vacations
    business
    sports
    religious
    club
    The domain is -----> Food and beverages
    foodindustry
    Doneraile
    contactfree
    TheFive
    SNAP
    The domain is -----> Fashion accessories
    friends
    Giveaway
    fashion
    style
    accessories
    

### Report

###### Sample Data -
The data source for this assignment is - https://www.kaggle.com/aungpyaeap/supermarket-sales The source of data that we have used is super store dataset. The data set is taken from Kaggle.

###### Aim
The aim of this assignment was to convert the RDBMS created in the last assignment into NoSQL DBMS. Then answer te questions for the databse.

##### Data Auditing
The data retrived from the kaggle datasource was audited to ensure that the data is all clean, consistent and valid. auditing was done to ensure that there are no null values or duplicate entries in the data source

After t=auditing was done, the data was broken into different tables according to the RDBMS requirements

##### Design Decision Explaination
The design decision explaination have been explained in a very detailed format in the the above section please refer to the design explaination section for more details about this

##### Database Questions 
###### Question 1 - What are tags are associated with a person, place or thing?
The tags have been scraped for each tweet. A set has been used to store all the tags that are related to the productline so that there would be no duplicates in the hash tags retrived

###### Question 2 - What social media users are like other social media users in your domain?
The social media that use same hashtags are similar to other social media users. This is the logic used for finding the users that are similar to each other.

###### Question 3 - What people, places or things are popular in your domain?
The productLines which have maximum tweets and retweets in an hour of a specified date are most popular in my domain.

###### Question 4 - What people, places or things are trending in your domain? (A trend is popularity over time.)
The hashtags which hace been posted the maximum number of times are the things that ateh people are most talking about and are most popular in that domain.


#####  Professionalism
Professionalism has been maintained throughout this assignment 

### Conclusion

The SQL Relational database was converted into NoSQL database - MongoDB. Everything was stored in the form of documents and collection in mySQL database. 
As per the the requirements of this assignmnets, all the 4 question have been answered.
The data is audited and cleaned before using for analysis.
The Database design decisions have been clearly explained
A discrete report has been created for explaianing the work in the assignment.
We have tested that the we can retrive the data for different usecases. We have taken exta efforts to make sure that professionalism has been maintained in the assignment.

### Contributions

Our contributed : 30%

External source (Blogs, wikipedia, sample assignment and youtube videos): 30%

Provided by the professor : 40%

### Citations

https://www.geeksforgeeks.org/difference-between-sql-and-nosql/

https://www.tutorialspoint.com/mongodb/index.htm

https://www.sitepoint.com/an-introduction-to-mongodb/

https://github.com/nikbearbrown/INFO_6210/tree/master/Project/Jobs_DB_Project/IPYNB

