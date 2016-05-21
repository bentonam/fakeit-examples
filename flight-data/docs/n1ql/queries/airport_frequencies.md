# Airport Airline Queries

These are example N1QL queries that may can performed to retrieve airport airline related data.

---

## Airport Frequencies by Code

##### Query

This query will find the available airlines by the 3 character IATA / FAA code of the airport

```sql
SELECT frequencies.frequency_id, frequencies.description, frequencies.frequency_mhz, frequencies.type
FROM `flight-data` AS airport_codes
USE KEYS 'airport_code_SLN'
INNER JOIN `flight-data` AS airport_frequencies
    ON KEYS 'airport_' || TOSTRING( airport_codes.id ) || '_frequencies'
UNNEST airport_frequencies.frequencies AS frequencies_lookup
INNER JOIN `flight-data` AS frequencies
    ON KEYS 'frequency_' || TOSTRING( frequencies_lookup )
ORDER BY frequencies.type ASC
```

This query will find the available airlines by the 4 character ICAO code of the airport

```sql
SELECT frequencies.frequency_id, frequencies.description, frequencies.frequency_mhz, frequencies.type
FROM `flight-data` AS airport_codes
USE KEYS 'airport_code_KSLN'
INNER JOIN `flight-data` AS airport_frequencies
    ON KEYS 'airport_' || TOSTRING( airport_codes.id ) || '_frequencies'
UNNEST airport_frequencies.frequencies AS frequencies_lookup
INNER JOIN `flight-data` AS frequencies
    ON KEYS 'frequency_' || TOSTRING( frequencies_lookup )
ORDER BY frequencies.type ASC
```

Both queries will yield the same exact result.

##### Result

```json
[
  {
    "description": "ATIS",
    "frequency_id": 66133,
    "frequency_mhz": 120.15,
    "type": "ATIS"
  },
  {
    "description": "KANSAS CITY CNTR",
    "frequency_id": 66134,
    "frequency_mhz": 134.9,
    "type": "CNTR"
  },
  {
    "description": "CTAF",
    "frequency_id": 66135,
    "frequency_mhz": 119.3,
    "type": "CTAF"
  },
  {
    "description": "GND",
    "frequency_id": 66136,
    "frequency_mhz": 121.9,
    "type": "GND"
  },
  {
    "description": "ARNG OPS",
    "frequency_id": 66137,
    "frequency_mhz": 49.95,
    "type": "OPS"
  },
  {
    "description": "WICHITA RDO",
    "frequency_id": 66138,
    "frequency_mhz": 122.4,
    "type": "RDO"
  },
  {
    "description": "TWR",
    "frequency_id": 66139,
    "frequency_mhz": 119.3,
    "type": "TWR"
  },
  {
    "description": "UNICOM",
    "frequency_id": 66140,
    "frequency_mhz": 122.95,
    "type": "UNIC"
  }
]
```
