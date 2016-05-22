# Airport Navaid Queries

These are example N1QL queries that may can performed to retrieve airport navaid related data.

---

## Airport Navaids by Code

##### Query

This query will find the available frequencies by the 3 character IATA / FAA code of the airport

[airport_navaids_by_iata_code.n1ql](queries/airport_navaids/airport_navaids_by_iata_code.n1ql)

```sql
SELECT navaids.navaid_id, navaids.navaid_ident, navaids.navaid_name, navaids.type,
    navaids.frequency_khz, navaids.geo, navaids.elevation, navaids.usage_type
FROM `flight-data` AS airport_codes
USE KEYS 'airport_code_SLN'
INNER JOIN `flight-data` AS airport_navaids
    ON KEYS 'airport_' || TOSTRING( airport_codes.id ) || '_navaids'
UNNEST airport_navaids.navaids AS navaids_lookup
INNER JOIN `flight-data` AS navaids
    ON KEYS 'navaid_' || TOSTRING( navaids_lookup )
ORDER BY navaids.navaid_name ASC
```

This query will find the available frequencies by the 4 character ICAO code of the airport

[airport_navaids_by_icao_code.n1ql](queries/airport_navaids/airport_navaids_by_icao_code.n1ql)

```sql
SELECT navaids.navaid_id, navaids.navaid_ident, navaids.navaid_name, navaids.type,
    navaids.frequency_khz, navaids.geo, navaids.elevation, navaids.usage_type
FROM `flight-data` AS airport_codes
USE KEYS 'airport_code_KSLN'
INNER JOIN `flight-data` AS airport_navaids
    ON KEYS 'airport_' || TOSTRING( airport_codes.id ) || '_navaids'
UNNEST airport_navaids.navaids AS navaids_lookup
INNER JOIN `flight-data` AS navaids
    ON KEYS 'navaid_' || TOSTRING( navaids_lookup )
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
