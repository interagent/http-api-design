#### Provide Request-Ids for Introspection

Include a `Request-Id` header in each API response, populated with a
UUID value. By logging these values on the client, server and any backing
services, it provides a mechanism to trace, diagnose and debug requests.
