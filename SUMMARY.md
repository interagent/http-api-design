# HTTP API Design Guide

* [Fondamenti](#fondamenti)
  * [Concetti separati](#concetti-separati)
  * [Utlizza una connessione sicura (TLS)](#utlizza-una-connessione-sicura-(tls))
  * [Richiedi il versioning negli Accept Headers](#richiedi-il-versioning-negli-accept-headers)
  * [Supporta ETag per il caching](#supporta-etag-per-il-caching)
  * [Fornisci un parametro Request-Ids per l'analisi](#fornisci-un-parametro-request-ids-per-lanalisi)
  * [Dividi risposte molto lunghe in piu richieste con range](#dividi-risposte-molto-lunghe-in-piu-richieste-con-range)
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
 

#### Richiedi il versioning negli Accept Headers

Un sistema di versioning e transizione tra le versioni può essere uno degli aspetti più difficili da progettare e realizzare nelle tue REST API. Proprio per questo, è meglio cominciare con alcuni accorgimenti che ci aiuteranno a mitigare questo tipo di problemi sin da subito.

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


#### Supporta ETag per il caching

Include an `ETag` header in all responses, identifying the specific
version of the returned resource. This allows users to cache resources
and use requests with this value in the `If-None-Match` header to determine
if the cache should be updated.


#### Fornisci un parametro Request-Ids per l'analisi

Include a `Request-Id` header in each API response, populated with a
UUID value. By logging these values on the client, server and any backing
services, it provides a mechanism to trace, diagnose and debug requests.


#### Dividi risposte molto lunghe in piu richieste con range

Large responses should be broken across multiple requests using `Range` headers
to specify when more data is available and how to retrieve it. See the
[Heroku Platform API discussion of Ranges](https://devcenter.heroku.com/articles/platform-api-reference#ranges)
for the details of request and response headers, status codes, limits,
ordering, and iteration.

### Richieste

The Requests section provides an overview of patterns for API requests.

#### Accetta JSON serializzato nei corpi delle richieste

Accept serialized JSON on `PUT`/`PATCH`/`POST` request bodies, either
instead of or in addition to form-encoded data. This creates symmetry
with JSON-serialized response bodies, e.g.:

```bash
$ curl -X POST https://service.com/apps \
    -H "Content-Type: application/json" \
    -d '{"name": "demoapp"}'

{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "demoapp",
  "owner": {
    "email": "username@example.com",
    "id": "01234567-89ab-cdef-0123-456789abcdef"
  },
  ...
}
```

##### Nomi delle risorse

Use the plural version of a resource name unless the resource in question is a singleton within the system (for example, in most systems a given user would only ever have one account). This keeps it consistent in the way you refer to particular resources.

##### Azioni

Prefer endpoint layouts that don’t need any special actions for
individual resources. In cases where special actions are needed, place
them under a standard `actions` prefix, to clearly delineate them:

```
/resources/:resource/actions/:action
```

e.g.

```
/runs/{run_id}/actions/stop
```

#### Usa un formato consistente per i percorsi (path)

#### Caratteri minuscoli per percorsi e attributi

Use downcased and dash-separated path names, for alignment with
hostnames, e.g:

```
service-api.com/users
service-api.com/app-setups
```

Downcase attributes as well, but use underscore separators so that
attribute names can be typed without quotes in JavaScript, e.g.:

```
service_class: "first"
```

#### Supporta anche riferimenti non-id per comodita

In some cases it may be inconvenient for end-users to provide IDs to
identify a resource. For example, a user may think in terms of a Heroku
app name, but that app may be identified by a UUID. In these cases you
may want to accept both an id or name, e.g.:

```bash
$ curl https://service.com/apps/{app_id_or_name}
$ curl https://service.com/apps/97addcf0-c182
$ curl https://service.com/apps/www-prod
```

Do not accept only names to the exclusion of IDs.

#### Minimizza annidamento dei percorsi

In data models with nested parent/child resource relationships, paths
may become deeply nested, e.g.:

```
/orgs/{org_id}/apps/{app_id}/dynos/{dyno_id}
```

Limit nesting depth by preferring to locate resources at the root
path. Use nesting to indicate scoped collections. For example, for the
case above where a dyno belongs to an app belongs to an org:

```
/orgs/{org_id}
/orgs/{org_id}/apps
/apps/{app_id}
/apps/{app_id}/dynos
/dynos/{dyno_id}
```

### Risposte

The Responses section provides an overview of patterns for API responses.

#### Ritorna sempre un codice di stato appropriato

Return appropriate HTTP status codes with each response. Successful
responses should be coded according to this guide:

* `200`: Request succeeded for a `GET` call, for a `DELETE` or
  `PATCH` call that completed synchronously, or for a `PUT` call that
  synchronously updated an existing resource
* `201`: Request succeeded for a `POST` call that completed
  synchronously, or for a `PUT` call that synchronously created a new
  resource
* `202`: Request accepted for a `POST`, `PUT`, `DELETE`, or `PATCH` call that
  will be processed asynchronously
* `206`: Request succeeded on `GET`, but only a partial response
  returned: see [above on ranges](../foundations/divide-large-responses-across-requests-with-ranges.md)

Pay attention to the use of authentication and authorization error codes:

* `401 Unauthorized`: Request failed because user is not authenticated
* `403 Forbidden`: Request failed because user does not have authorization to access a specific resource

Return suitable codes to provide additional information when there are errors:

* `422 Unprocessable Entity`: Your request was understood, but contained invalid parameters
* `429 Too Many Requests`: You have been rate-limited, retry later
* `500 Internal Server Error`: Something went wrong on the server, check status site and/or report the issue

Refer to the [HTTP response code spec](https://tools.ietf.org/html/rfc7231#section-6)
for guidance on status codes for user error and server error cases.

#### Fornisci le risorse interamente, quando possibile

Provide the full resource representation (i.e. the object with all
attributes) whenever possible in the response. Always provide the full
resource on 200 and 201 responses, including `PUT`/`PATCH` and `DELETE`
requests, e.g.:

```bash
$ curl -X DELETE \
  https://service.com/apps/1f9b/domains/0fd4

HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
...
{
  "created_at": "2012-01-01T12:00:00Z",
  "hostname": "subdomain.example.com",
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "updated_at": "2012-01-01T12:00:00Z"
}
```

202 responses will not include the full resource representation,
e.g.:

```bash
$ curl -X DELETE \
  https://service.com/apps/1f9b/dynos/05bd

HTTP/1.1 202 Accepted
Content-Type: application/json;charset=utf-8
...
{}
```

#### Fornisci gli (UU)IDs delle risorse

Give each resource an `id` attribute by default. Use UUIDs unless you
have a very good reason not to. Don’t use IDs that won’t be globally
unique across instances of the service or other resources in the
service, especially auto-incrementing IDs.

Render UUIDs in downcased `8-4-4-4-12` format, e.g.:

```
"id": "01234567-89ab-cdef-0123-456789abcdef"
```

#### Fornisci dei timestamps standard

Provide `created_at` and `updated_at` timestamps for resources by default,
e.g:

```javascript
{
  // ...
  "created_at": "2012-01-01T12:00:00Z",
  "updated_at": "2012-01-01T13:00:00Z",
  // ...
}
```

These timestamps may not make sense for some resources, in which case
they can be omitted.

#### Usa date in formato UTC formattate in ISO8601

Accept and return times in UTC only. Render times in ISO8601 format,
e.g.:

```
"finished_at": "2012-01-01T12:00:00Z"
```

#### Annida relazioni tramite le chiavi esterne (foreign-key)

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
    "name": "Alice",
    "email": "alice@heroku.com"
  },
  // ...
}
```

#### Genera errori strutturati

Generate consistent, structured response bodies on errors. Include a
machine-readable error `id`, a human-readable error `message`, and
optionally a `url` pointing the client to further information about the
error and how to resolve it, e.g.:

```
HTTP/1.1 429 Too Many Requests
```

```json
{
  "id":      "rate_limit",
  "message": "Account reached its API rate limit.",
  "url":     "https://docs.service.com/rate-limits"
}
```

Document your error format and the possible error `id`s that clients may
encounter.

#### Visualizza lo stato del limite delle richieste

Rate limit requests from clients to protect the health of the service
and maintain high service quality for other clients. You can use a
[token bucket algorithm](http://en.wikipedia.org/wiki/Token_bucket) to
quantify request limits.

Return the remaining number of request tokens with each request in the
`RateLimit-Remaining` response header.

#### Mantieni il JSON minimizzato in tutte le risposte

Extra whitespace adds needless response size to requests, and many
clients for human consumption will automatically "prettify" JSON
output. It is best to keep JSON responses minified e.g.:

```json
{"beta":false,"email":"alice@heroku.com","id":"01234567-89ab-cdef-0123-456789abcdef","last_login":"2012-01-01T12:00:00Z","created_at":"2012-01-01T12:00:00Z","updated_at":"2012-01-01T12:00:00Z"}
```

Instead of e.g.:

```json
{
  "beta": false,
  "email": "alice@heroku.com",
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "last_login": "2012-01-01T12:00:00Z",
  "created_at": "2012-01-01T12:00:00Z",
  "updated_at": "2012-01-01T12:00:00Z"
}
```

You may consider optionally providing a way for clients to retrieve
more verbose response, either via a query parameter (e.g. `?pretty=true`)
or via an `Accept` header param (e.g.
`Accept: application/vnd.heroku+json; version=3; indent=4;`).

### Artefatti

The Artifacts section describes the physical objects we use to manage and
discuss API designs and patterns.

#### Fornisci uno schema JSON interpretabile

Provide a machine-readable schema to exactly specify your API. Use
[prmd](https://github.com/interagent/prmd) to manage your schema, and ensure
it validates with `prmd verify`.

#### Fornisci una documentazione comprensibile allo sviluppatore

Provide human-readable documentation that client developers can use to
understand your API.

If you create a schema with prmd as described above, you can easily
generate Markdown docs for all endpoints with `prmd doc`.

In addition to endpoint details, provide an API overview with
information about:

* Authentication, including acquiring and using authentication tokens.
* API stability and versioning, including how to select the desired API
  version.
* Common request and response headers.
* Error serialization format.
* Examples of using the API with clients in different languages.

#### Fornisci degli esempi eseguibili

Provide executable examples that users can type directly into their
terminals to see working API calls. To the greatest extent possible,
these examples should be usable verbatim, to minimize the amount of
work a user needs to do to try the API, e.g.:

```bash
$ export TOKEN=... # acquire from dashboard
$ curl -is https://$TOKEN@service.com/users
```

If you use [prmd](https://github.com/interagent/prmd) to generate Markdown
docs, you will get examples for each endpoint for free.

#### Specifica la stabilità della tua API

Describe the stability of your API or its various endpoints according to
its maturity and stability, e.g. with prototype/development/production
flags.

See the [Heroku API compatibility policy](https://devcenter.heroku.com/articles/api-compatibility-policy)
for a possible stability and change management approach.

Once your API is declared production-ready and stable, do not make
backwards incompatible changes within that API version. If you need to
make backwards-incompatible changes, create a new API with an
incremented version number.



