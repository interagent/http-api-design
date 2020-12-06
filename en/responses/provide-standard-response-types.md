#### Provide standard response types

This document describes the acceptable values for each of JSON's basic data
types.

### String

* Acceptable values:
  * string
  * `null`

e.g:

```javascript
[
  {
    "description": "very descriptive description."
  },
  {
    "description": null
  },
]
```

### Boolean

* Acceptable values:
  * true
  * false

e.g:

```javascript
[
  {
    "provisioned_licenses": true
  },
  {
    "provisioned_licenses": false
  },
]
```

### Number

* Acceptable values:
  * number
  * `null`

Note: some JSON parsers will return numbers with a precision of over 15
decimal places as strings. If you need precision greater than 15 decimals,
always return a string for that value. If not, convert those strings to numbers
so that consumers of the API always know what value type to expect.

e.g:

```javascript
[
  {
    "average": 27.123
  },
  {
    "average": 12.123456789012
  },
]
```

### Array

* Acceptable values:
  * array

Note: Return an empty array rather than `NULL` when there are no values in the
array.

e.g:

```javascript
[
  {
    "child_ids": [1, 2, 3, 4],
  },
  {
    "child_ids": [],
  }
]
```

### Object

* Acceptable values:
  * object
  * null

e.g:

```javascript
[
  {
    "name": "service-production",
    "owner": {
      "id": "5d8201b0..."
     }
  },
  {
    "name": "service-staging",
    "owner": null
  }
]
```
