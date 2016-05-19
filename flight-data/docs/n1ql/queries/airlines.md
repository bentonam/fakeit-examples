# Airline Queries

These are example N1QL queries that may can performed to retrieve airline related data.

---

## Airline By ID

The following query will get a Airline by its ID.

##### Query

```sql
SELECT a.*
FROM `flight-data` AS a
USE KEYS 'airline_2009'
```

##### Result

```json
[
  {
    "_id": "airline_2009",
    "active": true,
    "airline_iata": "DL",
    "airline_icao": "DAL",
    "airline_id": 2009,
    "airline_name": "Delta Air Lines",
    "callsign": "DELTA",
    "doc_type": "airline",
    "iso_country": "US"
  }
]
```

The following query will retrieve multiple Airlines by their ID.

##### Query

```sql
SELECT a.*
FROM `flight-data` AS a
USE KEYS ['airline_2009', 'airline_24']
```

##### Result

```json
[
  {
    "_id": "airline_2009",
    "active": true,
    "airline_iata": "DL",
    "airline_icao": "DAL",
    "airline_id": 2009,
    "airline_name": "Delta Air Lines",
    "callsign": "DELTA",
    "doc_type": "airline",
    "iso_country": "US"
  },
  {
    "_id": "airline_24",
    "active": true,
    "airline_iata": "AA",
    "airline_icao": "AAL",
    "airline_id": 24,
    "airline_name": "American Airlines",
    "callsign": "AMERICAN",
    "doc_type": "airline",
    "iso_country": "US"
  }
]
```

---

## Airlines in a Country

The following index and queries allows for finding airlines based in a given country by creating an index on the `iso_country` where the `doc_type` is `airline`

##### Index

```sql
CREATE INDEX idx_airlines_iso_country ON `flight-data`(iso_country)
WHERE doc_type = 'airline' AND iso_country IS NOT NULL
```

##### Query

```sql
SELECT a
FROM `flight-data` AS a
WHERE a.doc_type = 'airline' AND
      a.iso_country IS NOT NULL
LIMIT 1
```

##### Result

```json
[
  {
    "a": {
      "_id": "airline_AAP",
      "active": false,
      "airline_iata": null,
      "airline_icao": "AAP",
      "airline_id": 27,
      "airline_name": "Aerovista Airlines",
      "callsign": "AEROVISTA GROUP",
      "doc_type": "airline",
      "iso_country": "AE"
    }
  }
]
```

In the returned results there is an `active` attribute.  We will want to update our query account for this, so that only active airlines are returned.

##### Query

```sql
SELECT a
FROM `flight-data` AS a
WHERE a.doc_type = 'airline' AND
      a.iso_country IS NOT NULL AND
      a.active = true
LIMIT 1
```

##### Result

```json
[
  {
    "a": {
      "_id": "airline_EK",
      "active": true,
      "airline_iata": "EK",
      "airline_icao": "UAE",
      "airline_id": 2183,
      "airline_name": "Emirates",
      "callsign": "EMIRATES",
      "doc_type": "airline",
      "iso_country": "AE"
    }
  }
]
```

Now we can retrieve all airlines for a given country.

##### Query

```sql
SELECT a._id, a.airline_iata, a.airline_icao, a.airline_name, a.callsign
FROM `flight-data` AS a
WHERE a.doc_type = 'airline' AND
      a.iso_country = 'AE' AND
      a.active = true
ORDER BY a.airline_name ASC
```

##### Result

```json
[
  {
    "_id": "airline_G9",
    "airline_iata": "G9",
    "airline_icao": "ABY",
    "airline_name": "Air Arabia",
    "callsign": "ARABIA"
  },
  {
    "_id": "airline_EK",
    "airline_iata": "EK",
    "airline_icao": "UAE",
    "airline_name": "Emirates",
    "callsign": "EMIRATES"
  },
  {
    "_id": "airline_EY",
    "airline_iata": "EY",
    "airline_icao": "ETD",
    "airline_name": "Etihad Airways",
    "callsign": "ETIHAD"
  },
  {
    "_id": "airline_FZ",
    "airline_iata": "FZ",
    "airline_icao": "FDB",
    "airline_name": "Fly Dubai",
    "callsign": null
  }
]
```

---

## Airline Codes

The following index and queries allows for finding airlines by their IATA, ICAO or Ident Codes. By creating an index on `code` attribute where the `doc_type = 'code'` and the `designation = 'airline'.

##### Index

This is using the previously created `idx_codes` index

```sql
CREATE INDEX idx_codes ON `flight-data`(code, designation)
WHERE doc_type = 'code'
```

##### Query 

This query will find the airline by the 2 character IATA code

```sql
SELECT a._id, a.airline_iata, a.airline_icao, a.airline_name, a.callsign
FROM `flight-data` AS c
INNER JOIN `flight-data` AS a ON KEYS 'airline_' || TOSTRING(c.id)
WHERE c.code = 'DL'
      AND c.designation = 'airline' 
      AND c.doc_type = 'code'
LIMIT 1
```

This query will find the airline by the 3 character ICAO code

```sql
SELECT a._id, a.airline_iata, a.airline_icao, a.airline_name, a.callsign
FROM `flight-data` AS c
INNER JOIN `flight-data` AS a ON KEYS 'airline_' || TOSTRING(c.id)
WHERE c.code = 'DAL'
      AND c.designation = 'airline' 
      AND c.doc_type = 'code'
LIMIT 1
```

Both queries will yield the same exact result.

##### Result

```json
[
  {
    "_id": "airline_2009",
    "airline_iata": "DL",
    "airline_icao": "DAL",
    "airline_name": "Delta Air Lines",
    "callsign": "DELTA"
  }
]
```