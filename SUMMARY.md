# Guida alla progettazione delle API HTTP

* [Fondamenti](#fondamenti)
  * [Concetti separati](#concetti-separati)
  * [Utlizza una connessione sicura (TLS)](#utlizza-una-connessione-sicura-(tls))
  * [Richiedi il versioning negli Accept Headers](#richiedi-il-versioning-negli-accept-headers)
  * [Supporta ETag per il caching](#supporta-etag-per-il-caching)
  * [Fornisci un parametro Request-Ids per l'analisi](#fornisci-un-parametro-request-ids-per-lanalisi)
  * [Dividi risposte molto lunghe in piu richieste con range](#dividi-risposte-molto-lunghe-in-piu-richieste-con-range)
* [Richieste](#richieste)
  * [Accetta JSON serializzato nei corpi delle richieste](#accetta-json-serializzato-nei-corpi-delle-richieste)
  * [Nomi delle risorse](#nomi-delle-risorse)
  * [Azioni](#azioni)
  * [Usa un formato consistente per i percorsi (path)](#usa-un-formato-consistente-per-i-percorsi-path)
    * [Caratteri minuscoli per percorsi e attributi](#caratteri-minuscoli-per-percorsi-e-attributi)
    * [Supporta anche riferimenti non-id per comodita](#supporta-anche-riferimenti-non-id-per-comodita)
    * [Minimizza annidamento dei percorsi](#minimizza-annidamento-dei-percorsi)
* [Risposte](#risposte)
  * [Ritorna sempre un codice di stato appropriato](#ritorna-sempre-un-codice-di-stato-appropriato)
  * [Fornisci le risorse interamente, quando possibile](#fornisci-le-risorse-interamente-quando-possibile)
  * [Fornisci gli (UU)IDs delle risorse](#fornisci-gli-uuids-delle-risorse)
  * [Fornisci dei timestamps standard](#fornisci-dei-timestamps-standard)
  * [Usa date in formato UTC formattate in ISO8601](#usa-date-in-formato-utc-formattate-in-iso8601)
  * [Annida relazioni tramite le chiavi esterne (foreign-key)](#annida-relazioni-tramite-le-chiavi-esterne-foreign-key)
  * [Genera errori strutturati](#genera-errori-strutturati)
  * [Visualizza lo stato del limite delle richieste](#visualizza-lo-stato-del-limite-delle-richieste)
  * [Mantieni il JSON minimizzato in tutte le risposte](#mantieni-il-json-minimizzato-in-tutte-le-risposte)
* [Artefatti](#artefatti)
  *  [Fornisci uno schema JSON interpretabile](#fornisci-uno-schema-json-interpretabile)
  *  [Fornisci una documentazione comprensibile allo sviluppatore](#fornisci-una-documentazione-comprensibile-allo-sviluppatore)
  *  [Fornisci degli esempi](#fornisci-degli-esempi)
  *  [Specifica la stabilità della tua API](#specifica-la-stabilita-della-tua-api)

### Fondamenti

Questa sezione illustra i principi di progettazione, sui quali si basa questa guida


#### Concetti separati

Struttura i componenti in maniera semplice mentre li progetti, separando i concetti tra le varie parti del ciclo di richiesta e risposta. Mantenendo la semplicita sui componenti, avrai possibilità di focalizzarti di più su problemi più grandi e più difficili da risolvere.

La richiesta e la risposta saranno fatte in modo da gestire una particolare risorsa oppure una loro collezione. Usa il percorso (path) per indicare un'identità, il body per trasferire il contenuto e gli headers per comunicare dati aggiuntivi (metadata). I parametri di query passati nell'url, possono essere usati come alternativa agli headers, solo in rari casi. Gli headers sono sempre preferiti, perche permettono più flessibilità e permettono di inviare informazioni più dettagliate.

#### Utlizza una connessione sicura (TLS)

Richiedi una connessione sicura con protocollo TLS per accedere alle APIs, senza eccezioni.
Non importa cercare di capire quando è opportuno usare TLS oppure quando non lo è. Semplicemente richiedila sempre.

Idealmente, puoi rifiutare qualsiasi richiesta che non sia fatta utilizzando il protocollo TLS, per evitare scambi di dati ed informazioni non sicuri. Nel caso in cui non possa gestire questo tipo di regola, basta rispondere con un `403 Forbidden`.

I redirects sono sconsigliati in quanto non rappresentano un buon approccio. I clients che si imbattono in molti redirects, raddoppiano il traffico sul server e rendono pressocche inutile il protocollo TLS, in quanto le informazioni sensibili vengono esplicitate durante la prima chiamata http

#### Richiedi il versioning negli Accept Headers

Un sistema di versioning e transizione tra le versioni può essere uno degli aspetti più difficili da progettare e realizzare nelle tue REST API. Proprio per questo, è meglio cominciare con alcuni accorgimenti che ci aiuteranno a mitigare questo tipo di problemi sin da subito.

Per evitare sorprese, cambiamenti bruschi agli utenti, è certamente buona norma richiedere di specificare la versione delle APIs in tutte le richieste. Il meccanismo di impostare una versione di default dovrebbe essere evitato, in quanto è molto difficile da cambiare in futuro.

L'approccio migliore sarebbe quello di specificare la versione negli headers http, con altri metadata, utilizzando per esempio `Accept` header con un `Content-Type` specifico, es:

```
Accept: application/vnd.heroku+json; version=3
```


#### Supporta ETag per il caching

Includi un header `ETag` in tutte le risposte, identificando la specifica versione della risorsa restituita.
Questo permetterà gli utenti di aggiungere alla cache le risorse e fare delle richieste con questo valore 
aggiungendo un `If-None-Match` header per determinare se la cache debba essere aggiornata o meno.


#### Fornisci un parametro Request-Ids per l'analisi

Includi un parametro `Request-Id` nell'header per ogni risposta della API, popolato con un valore UUID.
Facendo un log di questi valori nel client, server ed altri servizi ausiliari, è possibile ottenere un meccanismo di tracciabilità, diagnosi e debug delle richieste.

#### Dividi risposte molto lunghe in piu richieste con range

Risposte molto grandi, dovrebbero essere divise in più richieste usando un header `Range` per specificare quando più dati sono disponibili e come recuperarli. Dai un'occhiata alla documentazione [Heroku Platform API discussion of Ranges](https://devcenter.heroku.com/articles/platform-api-reference#ranges) per i dettagli degli headers delle richieste e delle risposte, codici di stato, limiti, ordinamenti e iterazioni

### Richieste

La sezione delle richieste fornisce una panoramica della struttura per le richieste API.

#### Accetta JSON serializzato nei corpi delle richieste

Accetta JSON serializzato nei corpi delle richieste `PUT`/`PATCH`/`POST` oppure in aggiunta ai dati form-encoded. Questo crea simmetria con i il corpo JSON serializzato delle risposte, es:

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

Usa il nome plurale di un nome di risorsa a meno che la risorsa in questione non sia un nome singolare relativo al sistema stesso (per esempio in alcuni sistemi un determinato utente può avere un solo account). Questo mantiene la consistenza quando ti riferisci alle risorse

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

#### Fornisci degli esempi

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



