# Route Queries

These are example N1QL queries that may can performed to retrieve route related data.

---


## Routes by Originating Airport

If we want to find all of the routes originating from a given airport.

##### Index

```sql
CREATE INDEX idx_route_source_airport_codes ON `flight-data`(source_airport_code, destination_airport_code)
WHERE doc_type = 'route' 
    AND active = true
    AND source_airport_code IS NOT NULL 
    AND destination_airport_code IS NOT NULL
```

##### Query

```sql
SELECT 
    {
        "airline": {
            "airline_code": IFNULL(airlines.airline_iata, airlines.airline_icao),
            "airline_name": airlines.airline_name
        },
        "destination_airport": {
            "airport_name": airports.airport_name,
            "iso_country": airports.iso_country,
            "iso_region": airports.iso_region,
            "airport_code": IFNULL(airports.airport_iata, airports.airport_icao, airports.airport_ident)
        }
    } AS route
FROM `flight-data` AS routes
USE INDEX (idx_route_source_airport_codes USING GSI)
INNER JOIN `flight-data` AS c1 ON KEYS 'airport_code_' || routes.destination_airport_code
INNER JOIN `flight-data` AS airports ON KEYS 'airport_' || TOSTRING(c1.id)
INNER JOIN `flight-data` AS c2 ON KEYS 'airline_code_' || routes.airline_code
INNER JOIN `flight-data` AS airlines ON KEYS 'airline_' || TOSTRING(c2.id)
WHERE routes.source_airport_code = 'ICT'
    AND routes.destination_airport_code IS NOT NULL
    AND routes.doc_type = 'route'
    AND routes.active = true
ORDER BY airports.airport_name ASC
```

##### Results

```json
[
  {
    "route": {
      "airline": {
        "airline_code": "FL",
        "airline_name": "AirTran Airways"
      },
      "destination_airport": {
        "airport_code": "MDW",
        "airport_name": "Chicago Midway Intl",
        "iso_country": "US",
        "iso_region": "US-IL"
      }
    }
  },
  {
    "route": {
      "airline": {
        "airline_code": "WN",
        "airline_name": "Southwest Airlines"
      },
      "destination_airport": {
        "airport_code": "MDW",
        "airport_name": "Chicago Midway Intl",
        "iso_country": "US",
        "iso_region": "US-IL"
      }
    }
  },
  ...
]
```

We can retrieve an aggregate count of the number of routes originating from a given airport.

##### Query

```sql
SELECT COUNT(1) AS routes
FROM `flight-data` AS r
WHERE r.source_airport_code = 'ATL'
    AND r.destination_airport_code IS NOT NULL
    AND r.doc_type = 'route'
    AND r.active = true
```

##### Result

```json
[
  {
    "airports": 915
  }
]
```

---

## Routes by Destination Airport

If we want to find all of the routes arriving at a given airport.

##### Index

```sql
CREATE INDEX idx_route_destination_airport_codes ON `flight-data`(destination_airport_code, source_airport_code)
WHERE doc_type = 'route' 
    AND active = true
    AND destination_airport_code IS NOT NULL
    AND source_airport_code IS NOT NULL 
```

##### Query

```sql
SELECT 
    {
        "airline": {
            "airline_code": IFNULL(airlines.airline_iata, airlines.airline_icao),
            "airline_name": airlines.airline_name
        },
        "source_airport": {
            "airport_name": airports.airport_name,
            "iso_country": airports.iso_country,
            "iso_region": airports.iso_region,
            "airport_code": IFNULL(airports.airport_iata, airports.airport_icao, airports.airport_ident)
        }
    } AS route
FROM `flight-data` AS routes
USE INDEX (idx_route_destination_airport_codes USING GSI)
INNER JOIN `flight-data` AS c1 ON KEYS 'airport_code_' || routes.source_airport_code
INNER JOIN `flight-data` AS airports ON KEYS 'airport_' || TOSTRING(c1.id)
INNER JOIN `flight-data` AS c2 ON KEYS 'airline_code_' || routes.airline_code
INNER JOIN `flight-data` AS airlines ON KEYS 'airline_' || TOSTRING(c2.id)
WHERE routes.destination_airport_code = 'MRY'
    AND routes.source_airport_code IS NOT NULL
    AND routes.doc_type = 'route'
    AND routes.active = true
ORDER BY airports.airport_name ASC
```

##### Results

```json
[
  {
    "route": {
      "airline": {
        "airline_code": "UA",
        "airline_name": "United Airlines"
      },
      "source_airport": {
        "airport_code": "DEN",
        "airport_name": "Denver Intl",
        "iso_country": "US",
        "iso_region": "US-CO"
      }
    }
  },
  {
    "route": {
      "airline": {
        "airline_code": "AS",
        "airline_name": "Alaska Airlines"
      },
      "source_airport": {
        "airport_code": "LAX",
        "airport_name": "Los Angeles Intl",
        "iso_country": "US",
        "iso_region": "US-CA"
      }
    }
  },
  ...
]
```

Airlines flying into airport 

##### Query

```sql
SELECT results.airline_code, results.airline_name, COUNT(1) AS routes
FROM (
    SELECT IFNULL(airlines.airline_iata, airlines.airline_icao) AS airline_code, airlines.airline_name
    FROM `flight-data` AS routes
    USE INDEX (idx_route_destination_airport_codes USING GSI)
    INNER JOIN `flight-data` AS c1 ON KEYS 'airport_code_' || routes.source_airport_code
    INNER JOIN `flight-data` AS airports ON KEYS 'airport_' || TOSTRING(c1.id)
    INNER JOIN `flight-data` AS c2 ON KEYS 'airline_code_' || routes.airline_code
    INNER JOIN `flight-data` AS airlines ON KEYS 'airline_' || TOSTRING(c2.id)
    WHERE routes.destination_airport_code = 'MRY'
        AND routes.source_airport_code IS NOT NULL
        AND routes.doc_type = 'route'
        AND routes.active = true
) AS results
GROUP BY results.airline_code, results.airline_name
ORDER BY results.airline_code ASC
```

##### Results

```json
[
  {
    "airline_code": "AA",
    "airline_name": "American Airlines",
    "routes": 2
  },
  {
    "airline_code": "AS",
    "airline_name": "Alaska Airlines",
    "routes": 2
  },
  {
    "airline_code": "G4",
    "airline_name": "Allegiant Air",
    "routes": 1
  },
  {
    "airline_code": "UA",
    "airline_name": "United Airlines",
    "routes": 3
  },
  {
    "airline_code": "US",
    "airline_name": "US Airways",
    "routes": 2
  }
]
```

## Routes by within certain distance

If we only want to return routes originating from a given airport that are with a certain distance in miles we would perform the following queries. 


##### Index

This will use the `idx_route_source_airport_codes` index we previously created.

For this query we need to provide it 3 pieces of information which are represented by `{{tokens}}`.

- The airport IATA, ICAO or Ident Code i.e. MCI
- A `distance_unit`
  - Kilometers: 111.045
  - Miles: 69
- A `max_distance` in which to contain results in, i.e. `300` 

##### Base Query

```sql
SELECT results.route.airline, results.route.destination_airport, 
    ROUND( results.route.distance, 2 ) AS distance
FROM (
    SELECT 
        {
            "airline": {
                "airline_code": IFNULL( airlines.airline_iata, airlines.airline_icao ),
                "airline_name": airlines.airline_name
            },
            "source_airport": {
                "airport_name": s_airports.airport_name,
                "iso_country": s_airports.iso_country,
                "iso_region": s_airports.iso_region,
                "airport_code": IFNULL( s_airports.airport_iata, s_airports.airport_icao, s_airports.airport_ident )
            },
            "destination_airport": {
                "airport_name": d_airports.airport_name,
                "iso_country": d_airports.iso_country,
                "iso_region": d_airports.iso_region,
                "airport_code": IFNULL( d_airports.airport_iata, d_airports.airport_icao, d_airports.airport_ident )
            },
            "distance": {{distance_unit}} * DEGREES(ACOS(COS(RADIANS( s_airports.geo.latitude ))
            * COS(RADIANS( d_airports.geo.latitude ))
            * COS(RADIANS( s_airports.geo.longitude ) - RADIANS( d_airports.geo.longitude ))
            + SIN(RADIANS( s_airports.geo.latitude ))
            * SIN(RADIANS( d_airports.geo.latitude ))))
        } AS route
    FROM `flight-data` AS routes
    USE INDEX (idx_route_source_airport_codes USING GSI)
    INNER JOIN `flight-data` AS s_codes ON KEYS 'airport_code_' || routes.source_airport_code
    INNER JOIN `flight-data` AS s_airports ON KEYS 'airport_' || TOSTRING(s_codes.id)
    INNER JOIN `flight-data` AS d_codes ON KEYS 'airport_code_' || routes.destination_airport_code
    INNER JOIN `flight-data` AS d_airports ON KEYS 'airport_' || TOSTRING(d_codes.id)
    INNER JOIN `flight-data` AS a_codes ON KEYS 'airline_code_' || routes.airline_code
    INNER JOIN `flight-data` AS airlines ON KEYS 'airline_' || TOSTRING(a_codes.id)
    WHERE routes.source_airport_code = '{{airport_code}}'
        AND routes.destination_airport_code IS NOT NULL
        AND routes.doc_type = 'route'
        AND routes.active = true
) AS results
WHERE results.route.distance <= {{max_distance}}
ORDER BY results.route.distance ASC
```

##### Airports Routes within a given distance in Miles Query

For our example we want to find any routes within 300 miles of "MCI". Our `{{distance_unit}}` is miles, this value needs to be `69` and our `{{max_distance}}` is `300`.  

```sql
SELECT results.route.airline, results.route.destination_airport, 
    ROUND( results.route.distance, 2 ) AS distance
FROM (
    SELECT 
        {
            "airline": {
                "airline_code": IFNULL( airlines.airline_iata, airlines.airline_icao ),
                "airline_name": airlines.airline_name
            },
            "destination_airport": {
                "airport_name": d_airports.airport_name,
                "iso_country": d_airports.iso_country,
                "iso_region": d_airports.iso_region,
                "airport_code": IFNULL( d_airports.airport_iata, d_airports.airport_icao, d_airports.airport_ident )
            },
            "distance": 69 * DEGREES(ACOS(COS(RADIANS( s_airports.geo.latitude ))
            * COS(RADIANS( d_airports.geo.latitude ))
            * COS(RADIANS( s_airports.geo.longitude ) - RADIANS( d_airports.geo.longitude ))
            + SIN(RADIANS( s_airports.geo.latitude ))
            * SIN(RADIANS( d_airports.geo.latitude ))))
        } AS route
    FROM `flight-data` AS routes
    USE INDEX (idx_route_source_airport_codes USING GSI)
    INNER JOIN `flight-data` AS s_codes ON KEYS 'airport_code_' || routes.source_airport_code
    INNER JOIN `flight-data` AS s_airports ON KEYS 'airport_' || TOSTRING(s_codes.id)
    INNER JOIN `flight-data` AS d_codes ON KEYS 'airport_code_' || routes.destination_airport_code
    INNER JOIN `flight-data` AS d_airports ON KEYS 'airport_' || TOSTRING(d_codes.id)
    INNER JOIN `flight-data` AS a_codes ON KEYS 'airline_code_' || routes.airline_code
    INNER JOIN `flight-data` AS airlines ON KEYS 'airline_' || TOSTRING(a_codes.id)
    WHERE routes.source_airport_code = 'MCI'
        AND routes.destination_airport_code IS NOT NULL
        AND routes.doc_type = 'route'
        AND routes.active = true
) AS results
WHERE results.route.distance <= 300
ORDER BY results.route.distance ASC
```

##### Airports Routes within a given distance in Miles Results

```json
[
  {
    "airline": {
      "airline_code": "K5",
      "airline_name": "SeaPort Airlines"
    },
    "destination_airport": {
      "airport_code": "SLN",
      "airport_name": "Salina Municipal Airport",
      "iso_country": "US",
      "iso_region": "US-KS"
    },
    "distance": 161.29
  },
  {
    "airline": {
      "airline_code": "K5",
      "airline_name": "SeaPort Airlines"
    },
    "destination_airport": {
      "airport_code": "HRO",
      "airport_name": "Boone Co",
      "iso_country": "US",
      "iso_region": "US-AR"
    },
    "distance": 226.08
  },
  {
    "airline": {
      "airline_code": "FL",
      "airline_name": "AirTran Airways"
    },
    "destination_airport": {
      "airport_code": "STL",
      "airport_name": "Lambert St Louis Intl",
      "iso_country": "US",
      "iso_region": "US-MO"
    },
    "distance": 235.89
  },
  {
    "airline": {
      "airline_code": "WN",
      "airline_name": "Southwest Airlines"
    },
    "destination_airport": {
      "airport_code": "STL",
      "airport_name": "Lambert St Louis Intl",
      "iso_country": "US",
      "iso_region": "US-MO"
    },
    "distance": 235.89
  }
]
```

##### Airports Routes within a given distance in Kilometers Query

For our example we want to find any routes within 300 kilometers of "CDG". Our `{{distance_unit}}` is kilometers, this value needs to be `111.045` and our `{{max_distance}}` is `300`.

```sql
SELECT results.route.airline, results.route.destination_airport, 
    ROUND( results.route.distance, 2 ) AS distance
FROM (
    SELECT 
        {
            "airline": {
                "airline_code": IFNULL( airlines.airline_iata, airlines.airline_icao ),
                "airline_name": airlines.airline_name
            },
            "destination_airport": {
                "airport_name": d_airports.airport_name,
                "iso_country": d_airports.iso_country,
                "iso_region": d_airports.iso_region,
                "airport_code": IFNULL( d_airports.airport_iata, d_airports.airport_icao, d_airports.airport_ident )
            },
            "distance": 111.045 * DEGREES(ACOS(COS(RADIANS( s_airports.geo.latitude ))
            * COS(RADIANS( d_airports.geo.latitude ))
            * COS(RADIANS( s_airports.geo.longitude ) - RADIANS( d_airports.geo.longitude ))
            + SIN(RADIANS( s_airports.geo.latitude ))
            * SIN(RADIANS( d_airports.geo.latitude ))))
        } AS route
    FROM `flight-data` AS routes
    USE INDEX (idx_route_source_airport_codes USING GSI)
    INNER JOIN `flight-data` AS s_codes ON KEYS 'airport_code_' || routes.source_airport_code
    INNER JOIN `flight-data` AS s_airports ON KEYS 'airport_' || TOSTRING(s_codes.id)
    INNER JOIN `flight-data` AS d_codes ON KEYS 'airport_code_' || routes.destination_airport_code
    INNER JOIN `flight-data` AS d_airports ON KEYS 'airport_' || TOSTRING(d_codes.id)
    INNER JOIN `flight-data` AS a_codes ON KEYS 'airline_code_' || routes.airline_code
    INNER JOIN `flight-data` AS airlines ON KEYS 'airline_' || TOSTRING(a_codes.id)
    WHERE routes.source_airport_code = 'CDG'
        AND routes.destination_airport_code IS NOT NULL
        AND routes.doc_type = 'route'
        AND routes.active = true
) AS results
WHERE results.route.distance <= 300
ORDER BY results.route.distance ASC
```

##### Airports Routes within a given distance in Miles Results

```json
[
  {
    "airline": {
      "airline_code": "SN",
      "airline_name": "Brussels Airlines"
    },
    "destination_airport": {
      "airport_code": "BRU",
      "airport_name": "Brussels Natl",
      "iso_country": "BE",
      "iso_region": "BE-BRU"
    },
    "distance": 251.14
  },
  {
    "airline": {
      "airline_code": "ET",
      "airline_name": "Ethiopian Airlines"
    },
    "destination_airport": {
      "airport_code": "BRU",
      "airport_name": "Brussels Natl",
      "iso_country": "BE",
      "iso_region": "BE-BRU"
    },
    "distance": 251.14
  },
  {
    "airline": {
      "airline_code": "AF",
      "airline_name": "Air France"
    },
    "destination_airport": {
      "airport_code": "LUX",
      "airport_name": "Luxembourg",
      "iso_country": "LU",
      "iso_region": "LU-L"
    },
    "distance": 273.63
  },
  {
    "airline": {
      "airline_code": "LG",
      "airline_name": "Luxair"
    },
    "destination_airport": {
      "airport_code": "LUX",
      "airport_name": "Luxembourg",
      "iso_country": "LU",
      "iso_region": "LU-L"
    },
    "distance": 273.63
  }
]
```
