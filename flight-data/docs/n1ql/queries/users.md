# User Queries

These are example N1QL queries that may can performed to retrieve user related data.

---

## Users By ID

The following query will get a User by their ID.

##### Query

```sql
SELECT users.user_id, users.account.username, users.account.`password`
FROM `flight-data` AS users
USE KEYS 'user_197'
```

##### Result

```json
[
  {
    "password": "N8HERvS8btfGbmz",
    "user_id": 197,
    "username": "Eudora43"
  }
]
```

The following query will retrieve multiple Users by their ID.

##### Query

```sql
SELECT users.details.*
FROM `flight-data` AS users
USE KEYS ['user_197', 'user_999']
```

##### Result

```json
[
  {
    "company": null,
    "dob": "2015-09-24",
    "first_name": "Albin",
    "home_country": "GQ",
    "job_title": "Forward Markets Director",
    "last_name": "Price",
    "middle_name": null,
    "prefix": "Dr.",
    "suffix": null
  },
  {
    "company": null,
    "dob": null,
    "first_name": "Dallas",
    "home_country": "KH",
    "job_title": "Central Functionality Executive",
    "last_name": "Kunze",
    "middle_name": "Harriett",
    "prefix": null,
    "suffix": null
  }
]
```

---

## Users By Username

The following query will get a User by their Username.

##### Index

```sql
CREATE INDEX idx_users_username ON `flight-data`(account.username)
WHERE doc_type = 'user'
```

##### Query

```sql
SELECT users.user_id, users.details.first_name, users.details.last_name
FROM `flight-data` AS users
WHERE users.account.username = 'Eudora43'
    AND users.doc_type = 'user'
LIMIT 1
```

##### Result

```json
[
  {
    "first_name": "Albin",
    "last_name": "Price",
    "user_id": 197
  }
]
```

The following query will retrieve a user by their `username` and `password`.

##### Index

We need to update our index from the previous example, to do that we need to drop and recreate it.

```sql
DROP INDEX `flight-data`.idx_users_username
```

```sql
CREATE INDEX idx_users_username ON `flight-data`(account.username, account.`password`)
WHERE doc_type = 'user'
```

##### Query

```sql
SELECT users.user_id, users.details.first_name, users.details.last_name
FROM `flight-data` AS users
WHERE users.account.username = 'Eudora43'
    AND users.account.`password` = 'N8HERvS8btfGbmz'
    AND users.doc_type = 'user'
LIMIT 1
```

##### Result

```json
[
  {
    "first_name": "Albin",
    "last_name": "Price",
    "user_id": 197
  }
]
```

---

## Users Addresses

The following query will get a users addresses by their `user_id`

##### Query

```sql
SELECT users.addresses
FROM `flight-data` AS users
USE KEYS 'user_197'
```

##### Result

```json
[
  {
    "addresses": [
      {
        "address_1": "98527 Tromp Light Lodge",
        "address_2": null,
        "iso_country": "GQ",
        "iso_region": "GQ-CS",
        "locality": "South Selmerhaven",
        "postal_code": "49540-9412",
        "primary": true,
        "type": "Home"
      },
      {
        "address_1": "5783 Mathilde Vista Parkway",
        "address_2": "Apt. 899",
        "iso_country": "GQ",
        "iso_region": "GQ-CS",
        "locality": "Schinnerside",
        "postal_code": "78895",
        "primary": false,
        "type": "Home"
      }
    ]
  }
]
```

However, these results are not friendly to work with as there is actually only 1 record returned.  This is because we only selected a single user document which has a single property `addresses` that is an array of multiple addresses.  We need to flatten this array and return it as separate documents.

##### Query

```sql
SELECT flattened_addresses.*
FROM `flight-data` AS users
USE KEYS 'user_197'
UNNEST users.addresses AS flattened_addresses
```

##### Result

```json
[
  {
    "address_1": "98527 Tromp Light Lodge",
    "address_2": null,
    "iso_country": "GQ",
    "iso_region": "GQ-CS",
    "locality": "South Selmerhaven",
    "postal_code": "49540-9412",
    "primary": true,
    "type": "Home"
  },
  {
    "address_1": "5783 Mathilde Vista Parkway",
    "address_2": "Apt. 899",
    "iso_country": "GQ",
    "iso_region": "GQ-CS",
    "locality": "Schinnerside",
    "postal_code": "78895",
    "primary": false,
    "type": "Home"
  }
]
```

We know that each of our users has a primary address, we need to be able to return just that address.

##### Query

```sql
SELECT flattened_addresses.*
FROM `flight-data` AS users
USE KEYS 'user_197'
UNNEST users.addresses AS flattened_addresses
WHERE flattened_addresses.`primary` = true
```

##### Results

```sql
[
  {
    "address_1": "98527 Tromp Light Lodge",
    "address_2": null,
    "iso_country": "GQ",
    "iso_region": "GQ-CS",
    "locality": "South Selmerhaven",
    "postal_code": "49540-9412",
    "primary": true,
    "type": "Home"
  }
]
```
---

## Users Phones

The following query will get a users phones by their `user_id`

##### Query

```sql
SELECT users.phones
FROM `flight-data` AS users
USE KEYS 'user_197'
```

##### Result

```json
[
  {
    "phones": [
      {
        "extension": null,
        "phone_number": "(570) 615-4605",
        "primary": false,
        "type": "Home"
      },
      {
        "extension": "1735",
        "phone_number": "(923) 578-2435",
        "primary": true,
        "type": "Mobile"
      }
    ]
  }
]
```

Just like the addresses, we need to flatten these results to make them more useful. 

##### Query

```sql
SELECT flattened_phones.*
FROM `flight-data` AS users
USE KEYS 'user_197'
UNNEST users.phones AS flattened_phones
```

##### Result

```json
[
  {
    "extension": null,
    "phone_number": "(570) 615-4605",
    "primary": false,
    "type": "Home"
  },
  {
    "extension": "1735",
    "phone_number": "(923) 578-2435",
    "primary": true,
    "type": "Mobile"
  }
]
```

We know that each of our users has a primary phone, we need to be able to return just that address.

##### Query

```sql
SELECT flattened_phones.*
FROM `flight-data` AS users
USE KEYS 'user_197'
UNNEST users.phones AS flattened_phones
WHERE flattened_phones.`primary` = true
```

##### Results

```json
[
  {
    "extension": "1735",
    "phone_number": "(923) 578-2435",
    "primary": true,
    "type": "Mobile"
  }
]
```

---

## Users Emails

The following query will get a users emails by their `user_id`

##### Query

```sql
SELECT users.emails
FROM `flight-data` AS users
USE KEYS 'user_1997'
```

##### Result

```json
[
  {
    "emails": [
      {
        "email_address": "Chase.Kohler63@gmail.com",
        "primary": false,
        "type": "Home"
      },
      {
        "email_address": "Judah66@gmail.com",
        "primary": true,
        "type": "Home"
      },
      {
        "email_address": "Creola_Little34@gmail.com",
        "primary": false,
        "type": "Work"
      }
    ]
  }
]
```

Just like the addresses and phones, we need to flatten these results to make them more useful.

##### Query

```sql
SELECT flattened_emails.*
FROM `flight-data` AS users
USE KEYS 'user_1997'
UNNEST users.emails AS flattened_emails
```

##### Result

```json
[
  {
    "email_address": "Chase.Kohler63@gmail.com",
    "primary": false,
    "type": "Home"
  },
  {
    "email_address": "Judah66@gmail.com",
    "primary": true,
    "type": "Home"
  },
  {
    "email_address": "Creola_Little34@gmail.com",
    "primary": false,
    "type": "Work"
  }
]
```

We know that each of our users has a primary address, we need to be able to return just that address.

##### Query

```sql
SELECT flattened_emails.*
FROM `flight-data` AS users
USE KEYS 'user_1997'
UNNEST users.emails AS flattened_emails
WHERE flattened_emails.`primary` = true
```

##### Results

```json
[
  {
    "email_address": "Judah66@gmail.com",
    "primary": true,
    "type": "Home"
  }
]
```

The following query will retrieve only the email address from the array of objects, omitting the `type` and `primary` attributes.

##### Query

```sql
SELECT email
FROM `flight-data` AS users
USE KEYS 'user_1997'
UNNEST users.emails[*].email_address AS email
```

##### Results

```json
[
  {
    "email": "Chase.Kohler63@gmail.com"
  },
  {
    "email": "Judah66@gmail.com"
  },
  {
    "email": "Creola_Little34@gmail.com"
  }
]
```

Building on the previous query, what if we wanted to ensure that the first email listed was the primary email address.  We can perform the following query.

```sql
SELECT emails.email_address AS email
FROM `flight-data` AS users
USE KEYS 'user_1997'
UNNEST users.emails AS emails
ORDER BY emails.`primary` DESC
```

```json
[
  {
    "email": "Judah66@gmail.com"
  },
  {
    "email": "Chase.Kohler63@gmail.com"
  },
  {
    "email": "Creola_Little34@gmail.com"
  }
]
```

##### User By ID as Flat Object

Our user model uses nested properties, we need to retrieve a users record as a single level object with just the primary address, phone and email.


##### Query

```sql
SELECT users.account.*, users.details.*, 
    primary_email.email_address, 
    primary_address.address_1, primary_address.address_2, primary_address.iso_country, 
    primary_address.iso_region, primary_address.locality, primary_address.postal_code,
    primary_phone.phone_number, primary_phone.extension AS phone_extension
FROM `flight-data` AS users
USE KEYS 'user_1997'
UNNEST users.emails AS primary_email
UNNEST users.addresses AS primary_address
UNNEST users.phones AS primary_phone
WHERE primary_email.`primary` = true
    AND primary_address.`primary` = true
    AND primary_phone.`primary` = true
```

##### Result

```json
[
  {
    "address_1": "07363 Trantow Garden Crossroad",
    "address_2": "Suite 108",
    "company": null,
    "created_on": 1447034465570,
    "dob": "2016-02-17",
    "email_address": "Judah66@gmail.com",
    "first_name": "Donnell",
    "home_country": "VC",
    "iso_country": "VC",
    "iso_region": "VC-01",
    "job_title": null,
    "last_login": 1463565402328,
    "last_name": "Ortiz",
    "locality": "South Leland",
    "middle_name": "Winifred",
    "modified_on": 1463561681928,
    "password": "cbEsvC1RxKRv0gi",
    "phone_extension": null,
    "phone_number": "(569) 409-9444",
    "postal_code": "26561",
    "prefix": null,
    "suffix": null,
    "username": "Garett31"
  }
]
```

### User Airport Reviews

The following indexes and queries show to retrieve which reviews for an airport a user has submitted.

##### Index

```sql
CREATE INDEX idx_users_airport_reviews ON `flight-data`(user_id, airport_id)
WHERE doc_type = 'airport-review'
```

##### Query

```sql
SELECT reviews.review_id, reviews.review_title, reviews.rating, MILLIS_TO_STR(reviews.review_date) AS review_date,
    IFNULL(airports.airport_iata, airports.airport_icao, airports.airport_ident) AS airport_code, airports.airport_name
FROM `flight-data` AS reviews
INNER JOIN `flight-data` AS airports ON KEYS 'airport_' || TOSTRING(reviews.airport_id)
WHERE reviews.user_id = 2097
    AND reviews.doc_type = 'airport-review'
ORDER BY reviews.review_date DESC
```

##### Results

```json
[
  {
    "airport_code": "LUX",
    "airport_name": "Luxembourg",
    "rating": 5,
    "review_date": "2016-05-19T00:19:36.795Z",
    "review_id": "bd5def2c-9e3b-48fd-94d1-83f95af0a358",
    "review_title": "Sit est error autem aut vero quo consequatur consequatur."
  },
  {
    "airport_code": "FCA",
    "airport_name": "Glacier Park Intl",
    "rating": 4,
    "review_date": "2016-05-18T22:19:01.648Z",
    "review_id": "663b9aed-3622-45d5-931b-d885d5a55edf",
    "review_title": "Quis molestias aut eveniet consequatur omnis."
  },
  {
    "airport_code": "VICX",
    "airport_name": "Kanpur Chakeri",
    "rating": 2,
    "review_date": "2016-05-18T18:01:20.88Z",
    "review_id": "7f9ce067-af32-4d77-9c54-d3b5e1debcd7",
    "review_title": "Debitis est omnis aut quia ut repudiandae pariatur culpa."
  },
  {
    "airport_code": "PLP",
    "airport_name": "Captain Ramon Xatruch Airport",
    "rating": 3,
    "review_date": "2015-06-28T22:48:49.121Z",
    "review_id": "c336ea6c-07b7-4746-9dc5-ddd9e0b89b07",
    "review_title": "Neque sit expedita veritatis quia sed."
  }
]
```

---

### User Airline Reviews

The following indexes and queries show to retrieve which reviews for an airline a user has submitted.

##### Index

```sql
CREATE INDEX idx_users_airline_reviews ON `flight-data`(user_id, airline_id)
WHERE doc_type = 'airline-review'
```

##### Query

```sql
SELECT reviews.review_id, reviews.review_title, reviews.rating, MILLIS_TO_STR(reviews.review_date) AS review_date,
    IFNULL(airlines.airline_iata, airlines.airline_icao) AS airline_code, airlines.airline_name
FROM `flight-data` AS reviews
INNER JOIN `flight-data` AS airlines ON KEYS 'airline_' || TOSTRING(reviews.airline_id)
WHERE reviews.user_id = 2097
    AND reviews.doc_type = 'airline-review'
ORDER BY reviews.review_date DESC
```

##### Results

```json
[
  {
    "airline_code": "MCX",
    "airline_name": "Cargo Express",
    "rating": 1,
    "review_date": "2016-05-18T23:58:07.977Z",
    "review_id": "b01f3273-2940-46a9-9ba6-bd7520eaa047",
    "review_title": "Fuga ex libero rerum magni aliquid est reiciendis."
  },
  {
    "airline_code": "ADP",
    "airline_name": "Aerodiplomatic",
    "rating": 1,
    "review_date": "2016-05-18T13:36:43.956Z",
    "review_id": "8962c9cf-114e-4610-88ee-4d6e373b655b",
    "review_title": "Quasi quod nobis sint."
  },
  {
    "airline_code": "WWI",
    "airline_name": "Worldwide Jet Charter",
    "rating": 2,
    "review_date": "2016-01-23T02:10:54.48Z",
    "review_id": "e7b18254-fe5e-4465-a0a2-56cdce476e77",
    "review_title": "Qui itaque quasi exercitationem suscipit iste ipsum sint est commodi."
  },
  {
    "airline_code": "SAG",
    "airline_name": "SOS Flygambulans",
    "rating": 3,
    "review_date": "2015-06-10T20:35:44.116Z",
    "review_id": "8398fc52-625c-4eb8-a4ae-559c02df9b8f",
    "review_title": "Et sit amet sit voluptate eaque quasi voluptas quia illum."
  }
]
```