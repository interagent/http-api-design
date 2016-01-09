# HTTP API Design Guide

* [Fondamenti](#fondamenti)
  * [Concetti separati](#concetti-separati)
  * [Require Secure Connections](foundations/require-secure-connections.md)
  * [Require Versioning in the Accepts Header](foundations/require-versioning-in-the-accepts-header.md)
  * [Support ETags for Caching](foundations/support-etags-for-caching.md)
  * [Provide Request-Ids for Introspection](foundations/provide-request-ids-for-introspection.md)
  * [Divide Large Responses Across Requests with Ranges](foundations/divide-large-responses-across-requests-with-ranges.md)
* [Requests](requests/README.md)
  * [Accept serialized JSON in request bodies](requests/accept-serialized-json-in-request-bodies.md)
  * [Resource names](requests/resource-names.md)
  * [Actions](requests/actions.md)
  * [Use consistent path formats](requests/use-consistent-path-formats.md)
    * [Downcase paths and attributes](requests/downcase-paths-and-attributes.md)
    * [Support non-id dereferencing for convenience](requests/support-non-id-dereferencing-for-convenience.md)
    * [Minimize path nesting](requests/minimize-path-nesting.md)
* [Responses](responses/README.md)
  * [Return appropriate status codes](responses/return-appropriate-status-codes.md)
  * [Provide full resources where available](responses/provide-full-resources-where-available.md)
  * [Provide resource (UU)IDs](responses/provide-resource-uuids.md)
  * [Provide standard timestamps](responses/provide-standard-timestamps.md)
  * [Use UTC times formatted in ISO8601](responses/use-utc-times-formatted-in-iso8601.md)
  * [Nest foreign key relations](responses/nest-foreign-key-relations.md)
  * [Generate structured errors](responses/generate-structured-errors.md)
  * [Show rate limit status](responses/show-rate-limit-status.md)
  * [Keep JSON minified in all responses](responses/keep-json-minified-in-all-responses.md)
* [Artifacts](artifacts/README.md)
  *  [Provide machine-readable JSON schema](artifacts/provide-machine-readable-json-schema.md)
  *  [Provide human-readable docs](artifacts/provide-human-readable-docs.md)
  *  [Provide executable examples](artifacts/provide-executable-examples.md)
  *  [Describe stability](artifacts/describe-stability.md)

### Fondamenti

Questa sezione illustra i principi di progettazione, sui quali si basa questa guida


#### Concetti separati

Struttura i componenti in maniera semplice mentre li progetti, separando i concetti tra le varie parti del ciclo di richiesta e risposta. Mantenendo la semplicita sui componenti, avrai possibilità di focalizzarti di più su problemi più grandi e più difficili da risolvere.

La richiesta e la risposta saranno fatte in modo da gestire una particolare risorsa oppure una loro collezione. Usa il percorso (path) per indicare un'identità, il body per trasferire il contenuto e gli headers per comunicare dati aggiuntivi (metadata).


Keep things simple while designing by separating the concerns between the
different parts of the request and response cycle. Keeping simple rules here
allows for greater focus on larger and harder problems.

Requests and responses will be made to address a particular resource or
collection. Use the path to indicate identity, the body to transfer the
contents and headers to communicate metadata. Query params may be used as a
means to pass header information also in edge cases, but headers are preferred
as they are more flexible and can convey more diverse information.


#### Utlizza una connessione sicura (TLS)

Richiedi una connessione sicura con protocollo TLS per accedere alle APIs, senza eccezioni.
Non importa cercare di capire quando è opportuno usare TLS oppure quando non lo è. Semplicemente richiedila sempre.

Idealmente, puoi rifiutare qualsiasi richiesta che non sia fatta utilizzando il protocollo TLS, per evitare scambi di dati ed informazioni non sicuri. Nel caso in cui non possa gestire questo tipo di regola, basta rispondere con un `403 Forbidden`.

Redirects are discouraged since they allow sloppy/bad client behaviour without
providing any clear gain.  Clients that rely on redirects double up on
server traffic and render TLS useless since sensitive data will already
 have been exposed during the first call.
 

#### Require Versioning in the Accepts Header

Versioning and the transition between versions can be one of the more
challenging aspects of designing and operating an API. As such, it is best to
start with some mechanisms in place to mitigate this from the start.

To prevent surprise, breaking changes to users, it is best to require a version
be specified with all requests. Default versions should be avoided as they are
very difficult, at best, to change in the future.

It is best to provide version specification in the headers, with other
metadata, using the `Accept` header with a custom content type, e.g.:

```
Accept: application/vnd.heroku+json; version=3
```


#### Support ETags for Caching

Include an `ETag` header in all responses, identifying the specific
version of the returned resource. This allows users to cache resources
and use requests with this value in the `If-None-Match` header to determine
if the cache should be updated.


#### Provide Request-Ids for Introspection

Include a `Request-Id` header in each API response, populated with a
UUID value. By logging these values on the client, server and any backing
services, it provides a mechanism to trace, diagnose and debug requests.


#### Divide Large Responses Across Requests with Ranges

Large responses should be broken across multiple requests using `Range` headers
to specify when more data is available and how to retrieve it. See the
[Heroku Platform API discussion of Ranges](https://devcenter.heroku.com/articles/platform-api-reference#ranges)
for the details of request and response headers, status codes, limits,
ordering, and iteration.



