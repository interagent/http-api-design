# HTTP API Design Guide

* [Foundations](foundations/README.md)
  * [Separate Concerns](foundations/separate-concerns.md)
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


#### Require Secure Connections

Require secure connections with TLS to access the API, without exception.
It’s not worth trying to figure out or explain when it is OK to use TLS
and when it’s not. Just require TLS for everything.

Ideally, simply reject any non-TLS requests by not responding to requests for
http or port 80 to avoid any insecure data exchange. In environments where this
is not possible, respond with `403 Forbidden`.

Redirects are discouraged since they allow sloppy/bad client behaviour without
providing any clear gain.  Clients that rely on redirects double up on
server traffic and render TLS useless since sensitive data will already
 have been exposed during the first call.


