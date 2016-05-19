# Airport Airline Queries

These are example N1QL queries that may can performed to retrieve airport airline related data.

---

## Airport Airlines by Code

This query uses the previously created `idx_airport_codes` index.

##### Query 

This query will find the available frequencies by the 3 character IATA / FAA code of the airport

```sql
SELECT airlines.airline_id, airlines.airline_iata, airlines.airline_icao, airlines.airline_name
FROM `flight-data` AS c
INNER JOIN `flight-data` AS airport_lookup ON KEYS 'airport_' || TOSTRING(c.id) || '_airlines'
UNNEST airport_lookup.airlines AS airline_codes
INNER JOIN `flight-data` AS airline_lookup ON KEYS 'airline_code_' || TOSTRING(airline_codes)
INNER JOIN `flight-data` AS airlines ON KEYS 'airline_' || TOSTRING(airline_lookup.id)
WHERE c.code = 'GSO'
    AND c.designation = 'airport' 
    AND c.doc_type = 'code'
ORDER BY airlines.airline_name ASC
```

This query will find the available airlines by the 4 character ICAO code of the airport

```sql
SELECT airlines.airline_id, airlines.airline_iata, airlines.airline_icao, airlines.airline_name
FROM `flight-data` AS c
INNER JOIN `flight-data` AS airport_lookup ON KEYS 'airport_' || TOSTRING(c.id) || '_airlines'
UNNEST airport_lookup.airlines AS airline_codes
INNER JOIN `flight-data` AS airline_lookup ON KEYS 'airline_code_' || TOSTRING(airline_codes)
INNER JOIN `flight-data` AS airlines ON KEYS 'airline_' || TOSTRING(airline_lookup.id)
WHERE c.code = 'KGSO'
    AND c.designation = 'airport' 
    AND c.doc_type = 'code'
ORDER BY airlines.airline_name ASC
```

Both queries will yield the same exact result.

##### Result

```json
[
  {
    "airline_iata": "AF",
    "airline_icao": "AFR",
    "airline_id": 137,
    "airline_name": "Air France"
  },
  {
    "airline_iata": "AS",
    "airline_icao": "ASA",
    "airline_id": 439,
    "airline_name": "Alaska Airlines"
  },
  {
    "airline_iata": "AZ",
    "airline_icao": "AZA",
    "airline_id": 596,
    "airline_name": "Alitalia"
  },
  {
    "airline_iata": "G4",
    "airline_icao": "AAY",
    "airline_id": 35,
    "airline_name": "Allegiant Air"
  },
  {
    "airline_iata": "AA",
    "airline_icao": "AAL",
    "airline_id": 24,
    "airline_name": "American Airlines"
  },
  {
    "airline_iata": "DL",
    "airline_icao": "DAL",
    "airline_id": 2009,
    "airline_name": "Delta Air Lines"
  },
  {
    "airline_iata": "F9",
    "airline_icao": "FFT",
    "airline_id": 2468,
    "airline_name": "Frontier Airlines"
  },
  {
    "airline_iata": "KL",
    "airline_icao": "KLM",
    "airline_id": 3090,
    "airline_name": "KLM Royal Dutch Airlines"
  },
  {
    "airline_iata": "9E",
    "airline_icao": "FLG",
    "airline_id": 3976,
    "airline_name": "Pinnacle Airlines"
  },
  {
    "airline_iata": "US",
    "airline_icao": "USA",
    "airline_id": 5265,
    "airline_name": "US Airways"
  },
  {
    "airline_iata": "UA",
    "airline_icao": "UAL",
    "airline_id": 5209,
    "airline_name": "United Airlines"
  }
]
```