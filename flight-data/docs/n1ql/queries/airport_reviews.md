# Airport Review Queries

These are example N1QL queries that may can performed to retrieve airport review related data.

---

## Airport Reviews By Airport ID

The following queries will return review related information for a given `airport_id` 

##### Index

```sql
CREATE INDEX idx_airport_reviews ON `flight-data`(airport_id, rating)
WHERE airport_id IS NOT NULL AND doc_type = 'airport-review'
```

##### Query 

Get all of the reviews for an airport and sort by most recent

```sql
SELECT reviews.review_id, reviews.review_title, reviews.rating, 
    MILLIS_TO_STR(reviews.review_date, 'yyyy-mm-dd') AS review_date,
    users.user_id, users.details.first_name || IFNULL(' ' || SUBSTR(users.details.last_name, 0, 1) || '.', '') AS reviewer_name
FROM `flight-data` AS reviews
INNER JOIN `flight-data` AS users ON KEYS 'user_' || TOSTRING(reviews.user_id)
WHERE reviews.airport_id = 3878
    AND reviews.rating IS NOT NULL
    AND reviews.doc_type = 'airport-review'
ORDER BY reviews.review_date DESC
```


##### Result

```json
[
  {
    "rating": 1,
    "review_date": "2016-05-19T00:58:36.964Z",
    "review_id": "bf4d5e7f-4a8c-45e5-90d2-c905e4e32e4b",
    "review_title": "Autem voluptatem magni omnis eveniet eos vel esse voluptates ipsa.",
    "reviewer_name": "Joshua C.",
    "user_id": 655
  },
  {
    "rating": 1,
    "review_date": "2016-05-18T18:55:54.292Z",
    "review_id": "eedab2c0-b752-4298-9d82-eeab3c0875ff",
    "review_title": "Sint perferendis reprehenderit voluptatem dolore aliquam dolor doloremque voluptas.",
    "reviewer_name": "Beth D.",
    "user_id": 9216
  },
  {
    "rating": 1,
    "review_date": "2016-05-18T04:18:18.87Z",
    "review_id": "5daa3f1d-0283-43ed-a3c7-47254d0d14f7",
    "review_title": "Error in error accusantium.",
    "reviewer_name": "Barney K.",
    "user_id": 2241
  },
  {
    "rating": 4,
    "review_date": "2015-09-20T21:15:18.71Z",
    "review_id": "afb5cfd9-3c67-4010-9261-c6a4ce833fd8",
    "review_title": "Autem autem reprehenderit ad eveniet id unde rerum.",
    "reviewer_name": "Sherman",
    "user_id": 5317
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
WHERE reviews.airport_id = 3622
    AND reviews.rating IS NOT NULL
    AND reviews.doc_type = 'airport-review'
GROUP BY reviews.airport_id
```

##### Result

```json
[
  {
    "avg_rating": 2.83,
    "best_rating": 5,
    "total_reviews": 6,
    "worst_rating": 1
  }
]
```
