#### Nest foreign key relations

Serialize foreign key references with a nested object, e.g.:

```javascript
{
  "name": "service-production",
  "owner": {
    "id": "5d8201b0..."
  },
  // ...
}
```

Instead of e.g.:

```javascript
{
  "name": "service-production",
  "owner_id": "5d8201b0...",
  // ...
}
```

This approach makes it possible to inline more information about the
related resource without having to change the structure of the response
or introduce more top-level response fields, e.g.:

```javascript
{
  "name": "service-production",
  "owner": {
    "id": "5d8201b0...",
    "email": "alice@heroku.com"
  },
  // ...
}
```

When nesting foreign key relations, use either the full record or just the foreign keys. Providing a subset of fields can lead to surprises and confusion, makes inconsistencies between different actions and endpoints more likely.

To avoid inconsistency and confusion, serialize either:
- **foreign keys** only - values the full record can be looked up with, like `id`, `slug`, `email`.
- **full record**, all fields (this would be an "embedded record")
