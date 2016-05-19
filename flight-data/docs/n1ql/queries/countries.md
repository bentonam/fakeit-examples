We have a well defined key pattern for our documents, just as you could use the `get()` method of your chosen SDK to retrieve a document, you can do that same operation through N1QL.  Lets say we wanted to get a the country information for Finland, we know that the country code is `FI`, we can execute the following query to retrieve the details:

```sql
SELECT f
FROM `flight-data` AS f
USE KEYS 'country_FI'
```

This query will actually fail with the following output, because we have yet to create a `PRIMARY INDEX` on the bucket.

```json
[
  {
    "code": 4000,
    "msg": "No primary index on keyspace `flight-data`. Use CREATE PRIMARY INDEX to create one.",
    "query_from_user": "SELECT f\rFROM `flight-data` AS f WHERE f.country_code = 'FI'\r;"
  }
]
```

Execute the following N1QL statement to create a primary index on the `flight-data` bucket.  Note that a primary index is not a requirement, and often is not used in production environments as any N1QL query can be executed.  If no primary index is found a secondary index must be used that matches the query criteria.  

```sql
CREATE PRIMARY INDEX idx_primary ON `flight-data`;
```

Now if we execute our query, we will get the following results:

```json
[
  {
    "f": {
      "_id": "country_FI",
      "continent_code": "EU",
      "country_code": "FI",
      "country_name": "Finland",
      "doc_type": "country"
    }
  }
]
```

The `USE KEYS` statement can accept a single document id or an array of document ids.  Lets say we wanted to retrieve the country information for the United States (US), Canada (CA) and Mexico (MX).

```sql
SELECT f
FROM `flight-data` AS f
USE KEYS ['country_US', 'country_CA', 'country_MX']
```

**Result:**

```json
[
  {
    "f": {
      "_id": "country_US",
      "continent_code": "NA",
      "country_code": "US",
      "country_name": "United States",
      "doc_type": "country"
    }
  },
  {
    "f": {
      "_id": "country_CA",
      "continent_code": "NA",
      "country_code": "CA",
      "country_name": "Canada",
      "doc_type": "country"
    }
  },
  {
    "f": {
      "_id": "country_MX",
      "continent_code": "NA",
      "country_code": "MX",
      "country_name": "Mexico",
      "doc_type": "country"
    }
  }
]
```

Using a well defined key pattern, we can derive a documents key, however while useful, this can be very limiting and requires us to be very creative and use lookup documents.  We can retrieve country information by querying the bucket for a specific country code:  

```sql
SELECT f
FROM `flight-data` AS f
WHERE f.doc_type = 'country' AND
      f.country_code = 'US';
```

Now if we run this query, we will get results.  However it will be extremely slow, this is because we are performing a PrimaryIndex scan.

```json
[
  {
    "f": {
      "_id": "country_US",
      "continent_code": "NA",
      "country_code": "US",
      "country_name": "United States",
      "doc_type": "country"
    }
  }
]
```

We can create an index on the country code by executing the following statement:

```sql
CREATE INDEX idx_country_codes ON `flight-data`(country_code)
WHERE country_code IS NOT MISSING
```

This will create an index for all documents that have a `country_code` attribute.  Now by executing our previous query we will get results a faster.

Because we have created an index on the `country_code` attribute, we can now get all of the available country codes and names. To be sure our index is being used we can use the `EXPLAIN` keyword

```sql
EXPLAIN
SELECT f.country_code, f.country_name
FROM `flight-data` AS f
```

We see that our query is still performing a **PrimaryScan** again and is not using our newly created index.  This is because we have not specified the same criteria that was used in when we created the index in the `WHERE` clause.

```sql
EXPLAIN
SELECT f.country_code, f.country_name
FROM `flight-data` AS f
WHERE country_code IS NOT MISSING
```

We now see that our query is using our index.  Executing our query will retrieve the country codes and names much faster.

```sql
SELECT f.country_code, f.country_name
FROM `flight-data` AS f
WHERE f.country_code IS NOT MISSING
```

Next we need to do is to order our results.

```sql
SELECT f.country_code, f.country_name
FROM `flight-data` AS f
WHERE f.country_code IS NOT MISSING
ORDER BY f.country_name ASC
```

Now lets say we also want to return the continent that the country is in as part of the query.  Execute the following query:

```sql
SELECT f.country_code, f.country_name, f.continent
FROM `flight-data` AS f
WHERE f.country_code IS NOT MISSING
ORDER BY f.country_name ASC
```

You will notice that there is no continent information returned as part of the query, and each result only contains the `country_code` and `country_name`, this is because each of our country documents do not contain and attribute for `continent`, it is actually named `continent_code`.  This is a missing attribute that will not be returned as part of your query and will not cause an error.  

```sql
SELECT f.country_code, f.country_name, f.continent_code
FROM `flight-data` AS f
WHERE f.country_code IS NOT MISSING
ORDER BY f.country_name ASC
```

We now are getting the `continent_code` returned with our results, but what if we wanted the name of the continent as well? To do this we need to use a `JOIN` statement.  All of our continent documents use the key pattern `continent_{{code}}` knowing this we can join on those keys by executing the following statement:

```sql
SELECT f.country_code, f.country_name, f.continent_code, c.continent_name
FROM `flight-data` AS f
INNER JOIN `flight-data` AS c ON KEYS 'continent_' || f.continent_code
WHERE f.country_code IS NOT MISSING
ORDER BY f.country_name ASC
```

Another useful query might be to get all of the available regions for a given country.  Lets say we want to find all of the regions / states for the US.  However we cannot simply query on the `iso_country` attribute being equal to a value, because we have 4 different types of documents that use that attribute:

- airline
- airport
- navaid
- region

We need to create a compound index that spans multiple attributes.

```sql
CREATE INDEX idx_regions_iso_country ON `flight-data`(iso_country)
WHERE doc_type = 'region' AND iso_country IS NOT MISSING
```

Now we can retrieve all of the regions / states for the "US".

```sql
SELECT f.region_code, f.region_name
FROM `flight-data` AS f
WHERE f.doc_type = 'region' AND f.iso_country = 'US'
ORDER BY f.region_name ASC
```
