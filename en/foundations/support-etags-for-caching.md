#### Support ETags for Caching

Include an `ETag` header in all responses, identifying the specific
version of the returned resource. This allows users to cache resources
and use requests with this value in the `If-None-Match` header to determine
if the cache should be updated.

[Back](require-versioning-in-the-accepts-header.md) | [Next](provide-request-ids-for-introspection.md)
