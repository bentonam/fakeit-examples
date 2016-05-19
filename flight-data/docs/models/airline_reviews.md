
# Airport Reviews Model

### Example Document

```json
{
  "_id": "airline_1650_review_b01f3273-2940-46a9-9ba6-bd7520eaa047",
  "doc_type": "airline-review",
  "review_id": "b01f3273-2940-46a9-9ba6-bd7520eaa047",
  "airline_id": 1650,
  "user_id": 2097,
  "rating": 1,
  "review_title": "Fuga ex libero rerum magni aliquid est reiciendis.",
  "review_body": "Suscipit sint et laborum sint sed. Ullam voluptas corrupti natus. Nulla molestias tempore dicta facilis autem dolor et tempora.\n \rVoluptatibus provident laboriosam. Veritatis quis ab neque ratione rerum dolorem. Tenetur numquam expedita. Aut et temporibus. Libero sapiente est voluptas ut.\n \rNumquam natus quae omnis ratione. Ut non et. Pariatur velit architecto occaecati velit nemo harum repudiandae eum. Nemo deserunt voluptatibus.",
  "review_date": 1463615887978
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
  airline_id:
    type: integer
    description: The airline_id the review is for
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



