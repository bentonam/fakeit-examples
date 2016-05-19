# Airport Navaid Queries

These are example N1QL queries that may can performed to retrieve airport frequency related data.

---

## Airport Navaids by Code

This query uses the previously created `idx_airport_codes` index.

##### Query 

This query will find the available frequencies by the 3 character IATA / FAA code of the airport

```sql
SELECT navaids.navaid_id, navaids.navaid_ident, navaids.navaid_name, navaids.type, 
    navaids.frequency_khz, navaids.geo, navaids.elevation, navaids.usage_type
FROM `flight-data` AS c
INNER JOIN `flight-data` AS lookup ON KEYS 'airport_' || TOSTRING(c.id) || '_navaids'
UNNEST lookup.navaids AS navaid_ids
INNER JOIN `flight-data` AS navaids ON KEYS 'navaid_' || TOSTRING(navaid_ids)
WHERE c.code = 'SLN'
    AND c.designation = 'airport' 
    AND c.doc_type = 'code'
ORDER BY navaids.navaid_name ASC
```

This query will find the available frequencies by the 4 character ICAO code of the airport

```sql
SELECT navaids.navaid_id, navaids.navaid_ident, navaids.navaid_name, navaids.type, 
    navaids.frequency_khz, navaids.geo, navaids.elevation, navaids.usage_type
FROM `flight-data` AS c
INNER JOIN `flight-data` AS lookup ON KEYS 'airport_' || TOSTRING(c.id) || '_navaids'
UNNEST lookup.navaids AS navaid_ids
INNER JOIN `flight-data` AS navaids ON KEYS 'navaid_' || TOSTRING(navaid_ids)
WHERE c.code = 'KSLN'
    AND c.designation = 'airport' 
    AND c.doc_type = 'code'
ORDER BY navaids.navaid_name ASC
```

Both queries will yield the same exact result.

##### Result

```json
[
  {
    "elevation": 1315,
    "frequency_khz": 344,
    "geo": {
      "latitude": 38.68149948,
      "longitude": -97.64510345
    },
    "navaid_id": 93716,
    "navaid_ident": "SL",
    "navaid_name": "Flory",
    "type": "NDB",
    "usage_type": "TERMINAL"
  },
  {
    "elevation": 1310,
    "frequency_khz": 117100,
    "geo": {
      "latitude": 38.92509842,
      "longitude": -97.62139893
    },
    "navaid_id": 93733,
    "navaid_ident": "SLN",
    "navaid_name": "Salina",
    "type": "VORTAC",
    "usage_type": "BOTH"
  }
]
```