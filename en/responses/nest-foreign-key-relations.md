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

Serializing partial records can cause trouble for consumers. When a consumer has for example 3/4 of the fields within a relationship, that causes confusion - the client may have to subsequently request the full record when it's missing those fields. This is especially troublesome if the partial relationship record varies by endpoint.

To avoid that issue, serialize either:
- the **relationship** only - just unique identifiers the record can be looked up with, like `id`, `slug`, `email`.
- th **full record**, with all fields (this would be an "embedded record")
