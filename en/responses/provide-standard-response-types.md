#### Provide standard response types

When a value is a array type, return an empty array rather than `NULL` when
there are no values in the array.

e.g:

```javascript
[
  {
    "child_ids": [1, 2, 3, 4],
    // ...
  },
  {
    "child_ids": [],
    // ...
  }
]
```

For all other values, the response type should always be the same basic type or
`NULL`.

For example, some JSON parsers will return numbers with a precision of over 15
decimal places as strings. If you need precision greater than 15 decimals,
always return a string for that value. If not, convert those strings to numbers
so that consumers of the API always know what value type to expect.

One exception to this rule is that it sometimes makes sense to return `NULL`
rather than an "empty" value of the same basic type, such as `""` for a string
or `0` for an integer.
