# Friend Queries

These are example N1QL queries that may can performed to retrieve user auth related data.

---

## User Friends

The following query will get a Users Friends by their `user_id`

##### Query

[user_by_username.n1ql](queries/users/user_by_username.n1ql)

```sql
SELECT users.user_id,
    users.first_name || IFNULL(' ' || users.last_name, '') AS friend
FROM social AS user_friends
USE KEYS 'user_100_friends'
INNER JOIN social AS users ON KEYS
    ARRAY 'user_' || TOSTRING( friend.user_id )
        FOR friend IN user_friends.friends
    END
ORDER BY users.first_name ASC,
    users.last_name ASC
```

##### Result

```json
[
  {
    "friend": "Alfred Erdman",
    "user_id": 470
  },
  {
    "friend": "Blanca Brekke",
    "user_id": 685
  },
  {
    "friend": "Camryn Hodkiewicz",
    "user_id": 487
  },
  {
    "friend": "Daphnee Beahan",
    "user_id": 784
  },
  {
    "friend": "Diamond Konopelski",
    "user_id": 204
  },
  {
    "friend": "Drew Kemmer",
    "user_id": 179
  },
  {
    "friend": "Franz Collins",
    "user_id": 302
  },
  {
    "friend": "Griffin Gulgowski",
    "user_id": 90
  },
  {
    "friend": "Hunter Jast",
    "user_id": 387
  },
  {
    "friend": "Jedediah Armstrong",
    "user_id": 822
  },
  {
    "friend": "Johnny Barton",
    "user_id": 380
  },
  {
    "friend": "Kennith Sipes",
    "user_id": 84
  },
  {
    "friend": "Lia Jakubowski",
    "user_id": 252
  },
  {
    "friend": "Melyna Bernhard",
    "user_id": 350
  },
  {
    "friend": "Monica Schneider",
    "user_id": 608
  },
  {
    "friend": "Myrtie Christiansen",
    "user_id": 268
  },
  {
    "friend": "Nayeli Walter",
    "user_id": 798
  },
  {
    "friend": "Nicolas Koepp",
    "user_id": 898
  },
  {
    "friend": "Niko Aufderhar",
    "user_id": 295
  },
  {
    "friend": "Pasquale Pagac",
    "user_id": 160
  },
  {
    "friend": "Peggie Powlowski",
    "user_id": 964
  },
  {
    "friend": "Rebeka Spencer",
    "user_id": 567
  }
]
```
