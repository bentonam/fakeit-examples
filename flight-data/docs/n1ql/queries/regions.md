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
