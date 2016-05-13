## Flight Data Example

The models in this example rely entirely on data made available to the models.  The data being passed to these models came from the following sources:

- [http://ourairports.com/data/](http://ourairports.com/data/) for Airports, Countries, Frequencies, Navaids, Regions, Runways
- [http://openflights.org/data.html](http://openflights.org/data.html) for Airports, Airlines, Routes

These data models demonstrate the following:

- How to use [Model Dependencies](https://github.com/bentonam/fakeit#model-dependencies) with [fakeit](https://github.com/bentonam/fakeit)
- Using the `globals` variable as a counter
- Using the `inputs` to make external data available to the model
- Using the `documents` variable to use data from a previously generated model

There are 12 types of models that will be generated

- [Airlines](#airlines)
- [Airports](#airports)
- [Airport Airlines](#airport-airlines) (has dependencies on Airports and Routes)
- [Airport Frequencies](#airport-frequencies) (has dependencies on Airports and Frequencies)
- [Airport Navaids](#airport-navaids) (has dependencies on Airports and Navaids)
- [Airport Runways](#airport-runways) (has dependencies on Airports and Runways)
- [Countries](#countries)
- [Frequencies](#frequencies)
- [Navaids](#navaids)
- [Regions](#regions)
- [Routes](#routes)
- [Runways](#runways)

> Warning executing this entire example will generate ~150k documents

##### airlines

```json
{
  "_id": "airline_DAL",
  "doc_type": "airline",
  "airline_name": "Delta Air Lines",
  "airline_iata": "DL",
  "airline_icao": "DAL",
  "callsign": "DELTA",
  "iso_country": "US",
  "active": true
}
```

##### airports

```json
{
  "_id": "airport_3384",
  "airport_id": 3384,
  "doc_type": "airport",
  "airport_ident": "KATL",
  "airport_type": "large_airport",
  "airport_name": "Hartsfield Jackson Atlanta Intl",
  "geo": {
    "latitude": 33.63669968,
    "longitude": -84.42810059
  },
  "elevation": 1026,
  "continent": "NA",
  "iso_country": "US",
  "iso_region": "US-GA",
  "municipality": "Atlanta",
  "airport_icao_code": "KATL",
  "airport_iata_code": "ATL",
  "airport_gps_code": "KATL",
  "airport_local_code": "ATL",
  "timezone_offset": -5,
  "dst": "A",
  "timezone": "America/New_York"
}
```

##### airport-airlines

```json
{
  "_id": "airport_3384_airlines",
  "airport_id": 3384,
  "doc_type": "airport-airlines",
  "airport_ident": "KATL",
  "airlines": [
    "3M",
    "9E",
    "AA",
    "AC",
    "AF",
    "AM",
    "AS",
    "AY",
    "AZ",
    "BA",
    "CA",
    "CI",
    "CX",
    "DL",
    "EI",
    "EY",
    "F9",
    "FL",
    "IB",
    "JL",
    "KE",
    "KL",
    "LH",
    "MH",
    "NH",
    "NK",
    "NZ",
    "OZ",
    "QF",
    "QR",
    "SU",
    "TK",
    "UA",
    "US",
    "VA",
    "VS",
    "WN",
    "WS"
  ]
}
```

##### airport-frequencies

```json
{
  "_id": "airport_3384_frequencies",
  "airport_id": 3384,
  "doc_type": "airport-frequencies",
  "airport_ident": "KATL",
  "frequencies": [
    66249,
    66250,
    66251,
    66252,
    66253,
    66254,
    66255,
    66256,
    66257
  ]
}
```

##### airport-navaids

```json
{
  "_id": "airport_3384_navaids",
  "airport_id": 3384,
  "doc_type": "airport-navaids",
  "airport_ident": "KATL",
  "navaids": [
    "5bcbf2fa-7776-52e7-9170-0ff6235e3269",
    "84336733-2663-59d8-8800-4dd66f8d5cab",
    "2413d4de-f37c-5c84-8f7d-da731b47b899"
  ]
}
```

##### airport-runways

```json
{
  "_id": "airport_3384_runways",
  "airport_id": 3384,
  "doc_type": "airport-runways",
  "airport_ident": "KATL",
  "runways": [
    243551,
    243550,
    243553,
    243552,
    243554
  ]
}
```

##### countries

```json
{
  "_id": "country_US",
  "country_code": "US",
  "doc_type": "country",
  "country_name": "United States",
  "continent": "NA"
}
```

##### frequencies

```json
{
  "_id": "frequency_66249",
  "frequency_id": 66249,
  "doc_type": "frequency",
  "airport_id": 3384,
  "airport_ident": "KATL",
  "type": "APP",
  "description": "ATLANTA APP",
  "frequency_mhz": 118.35
}
```

##### navaids

```json
{
  "_id": "navaid_84336733-2663-59d8-8800-4dd66f8d5cab",
  "navaid_id": "84336733-2663-59d8-8800-4dd66f8d5cab",
  "doc_type": "navaid",
  "navaid_ident": "ATL",
  "navaid_name": "Atlanta",
  "type": "VORTAC",
  "frequency_khz": 116900,
  "geo": {
    "latitude": 33.6291008,
    "longitude": -84.43509674
  },
  "elevation": 1000,
  "iso_country": "US",
  "dme": {
    "frequency_khz": 116900,
    "channel": "116X",
    "latitude": 33.6246,
    "longitude": -84.4258,
    "elevation": 1000
  },
  "magnetic_variation": -4.022,
  "usage_type": "BOTH",
  "power": "HIGH",
  "associated_airport_icao_code": "KATL"
}
```

##### regions

```json
{
  "_id": "region_US-GA",
  "region_id": 306086,
  "doc_type": "region",
  "region_code": "US-GA",
  "local_code": "GA",
  "region_name": "Georgia",
  "continent": "NA",
  "iso_country": "US"
}
```

##### routes

```json
{
  "_id": "route_0026d7f9-4453-52c5-8757-e82aaee8c119",
  "route_id": "0026d7f9-4453-52c5-8757-e82aaee8c119",
  "doc_type": "route",
  "airline_code": "DL",
  "source_airport_code": "ATL",
  "destination_airport_code": "JFK",
  "codehsare": false,
  "stops": 0,
  "equipment": "777",
  "active": true
}
```

##### runways

```json
{
  "_id": "runway_243550",
  "runway_id": 243550,
  "doc_type": "runway",
  "airport_id": 3384,
  "airport_ident": "KATL",
  "runway_length": 10000,
  "runway_width": 150,
  "surface": "CON",
  "lighted": true,
  "closed": null,
  "low_bearing": {
    "ident": "08R",
    "latitude": 33.6468,
    "longitude": -84.4384,
    "elevation": 1024,
    "magnetic_heading": 90,
    "displaced_threshold": null
  },
  "high_bearing": {
    "ident": "26L",
    "latitude": 33.6468,
    "longitude": -84.4055,
    "elevation": 995,
    "magnetic_heading": 270,
    "displaced_threshold": null
  }
}
```

### Usage Examples

Below is a variety of commands that can be used on this data model, all of the examples assume that you are in the `flight-data/` directory.  While all of the `fakeit` options will work, for this example we will only demonstrate how adding the data to Couchbase.  Please be aware that this may take several minutes to complete.

```bash
[ecommerce]$ fakeit -m models -i input -d couchbase
Generating 5912 documents for Airlines model
Generating 6922 documents for Airports model
Generating 247 documents for Countries model
Generating 10314 documents for Navaids model
Generating 17137 documents for Frequencies model
Generating 3999 documents for Regions model
Generating 67065 documents for Routes model
Generating 8813 documents for Runways model
Generating 6922 documents for AirportAirlines model
Generating 6922 documents for AirportFrequencies model
Generating 6922 documents for AirportNavaids model
Generating 6922 documents for AirportRunways model
```
