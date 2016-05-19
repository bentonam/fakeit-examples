
# Airport Reviews Model

### Example Document

```json
{
  "_id": "airport_2563_review_bd5def2c-9e3b-48fd-94d1-83f95af0a358",
  "doc_type": "airport-review",
  "review_id": "bd5def2c-9e3b-48fd-94d1-83f95af0a358",
  "airport_id": 2563,
  "user_id": 2097,
  "rating": 5,
  "review_title": "Sit est error autem aut vero quo consequatur consequatur.",
  "review_body": "Hic eligendi hic ratione fuga id sit omnis. At nulla eos nihil facere dolorem quidem est officiis. Sit hic voluptatem ut. Distinctio eum omnis at dicta sint.\n \rImpedit hic molestias numquam dolore et eos accusantium recusandae dicta. Veniam reprehenderit expedita. Ipsa ratione quaerat odit.\n \rAb est tempore qui voluptas in. Earum atque veniam libero et odit consequuntur cum iste. Facilis molestias ut. Officiis ducimus magni maxime aut. Voluptatem quo quasi deleniti quod dicta sit sed. Itaque aut error itaque recusandae voluptatem voluptatem accusamus illum.",
  "review_date": 1463617176795
}
```

### Model Definitions

```yaml
type: object
properties:
  _id:
    type: string
    description: The document id
  doc_type:
    type: string
    description: The document type
  review_id:
    type: string
    description: Unique identifier representing a specific review
  airport_id:
    type: integer
    description: The airport_id the review is for
  user_id:
    type: integer
    description: The user_id of the user who wrote the review
  rating:
    type: integer
    description: The review rating
  review_title:
    type: string
    description: The review title
  review_body:
    type: string
    description: The review content
  review_date:
    type: integer
    description: The review content
```



