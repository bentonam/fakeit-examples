# Airline Review Queries

These are example N1QL queries that may can performed to retrieve airline review related data.

---

## Airline Reviews By Airline ID

The following queries will return review related information for a given `airline_id` 

##### Index

```sql
CREATE INDEX idx_airline_reviews ON `flight-data`(airline_id, rating)
WHERE airline_id IS NOT NULL AND doc_type = 'airline-review'
```

##### Query 

Get all of the reviews for an airline and sort by most recent

```sql
SELECT reviews.review_id, reviews.review_title, reviews.rating, 
    MILLIS_TO_STR(reviews.review_date, 'yyyy-mm-dd') AS review_date,
    users.user_id, users.details.first_name || IFNULL(' ' || SUBSTR(users.details.last_name, 0, 1) || '.', '') AS reviewer_name
FROM `flight-data` AS reviews
INNER JOIN `flight-data` AS users ON KEYS 'user_' || TOSTRING(reviews.user_id)
WHERE reviews.airline_id = 2009
    AND reviews.rating IS NOT NULL
    AND reviews.doc_type = 'airline-review'
ORDER BY reviews.review_date DESC
```


##### Result

```json
[
  {
    "rating": 2,
    "review_date": "2016-05-18T03:40:11.614Z",
    "review_id": "1ff54a30-c8cb-41a7-be72-af2e120cb94d",
    "review_title": "Minima corrupti voluptate et ea sapiente vel vitae sit.",
    "reviewer_name": "Wallace R.",
    "user_id": 6283
  },
  {
    "rating": 2,
    "review_date": "2015-08-15T15:36:17.838Z",
    "review_id": "a4343c3d-570c-4499-a20b-3a624f7038c4",
    "review_title": "Corrupti quasi ea saepe.",
    "reviewer_name": "Ian D.",
    "user_id": 5904
  }
]
```

##### Query 

This query will return the total # of reviews, the average review rating and the best and worst rating.  

```sql
SELECT COUNT( 1 ) AS total_reviews, 
    ROUND( AVG( reviews.rating ), 2 ) AS avg_rating,
    MIN( reviews.rating ) AS worst_rating,
    MAX( reviews.rating ) AS best_rating
FROM `flight-data` AS reviews
WHERE reviews.airline_id = 22
    AND reviews.rating IS NOT NULL
    AND reviews.doc_type = 'airline-review'
GROUP BY reviews.airline_id
```

##### Result

```json
[
  {
    "avg_rating": 2.8,
    "best_rating": 5,
    "total_reviews": 10,
    "worst_rating": 1
  }
]
```
