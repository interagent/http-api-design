# Guía de diseño de APIs HTTPGuide

## Introducción

Esta guía describe una serie de buenas prácticas para el diseño de APIs HTTP+JSON originalmente extraído del trabajo en el API de la [plataforma Heroku](https://devcenter.heroku.com/articles/platform-api-reference).

Esta guía incluye añadidos a ese API y también sirve de guía para nuevas APIs internas en Heroku. Esteramos que sea de interés a los diseñadores de APIs que no son de Heroku.

Los objetivos que nos hemos marcado son consistencia y foco en la lógica de negocio, a la vez que evitamos centrarnos en detalles supérfluos (_N.T. el término en ingles es "design bikeshedding" explicado en [este artículo](http://es.wikipedia.org/wiki/Ley_de_Parkinson_de_la_trivialidad))._
Buscamos un _método bueno, consistente y bien documentado_ para el diseño de APIs, aunque no necesariamente _el método único o ideal_.

Asumimos que los conceptos básicos de APIs HTTP+JSON son familiares para tí, así que no entraremos en sus fundamentos en esta guía.

Agradecemos también [contribuciones](CONTRIBUTING.md) a la versión inglesa de esta guía o a su traducción al castellano.

## Contenidos

* [Fundamentos](#fundamentos)
  *  [Separar responsabilidades](#separar-responsabilidades)
  *  [Conexiones seguras requeridas](#conexiones-seguras-requeridas)
  *  [Versionado con la cabecera Accepts requerido](#versionado-con-la-cabecera-accepts-requerido)
  *  [Soportar ETags para el cacheado](#soportar-etags-para-el-cacheado)
  *  [Proporcionar identificadores de petición para introspección](#Proporcionar-identificadores-de-petición-para-introspeccion)
  *  [Dividir respuestas largas en varias peticiones usando rangos](#dividir-respuestas-largas-en-varias-peticiones-usando-rangos)
* [Peticiones](#peticiones)
  *  [Retornar códigos de estado apropiados](#retornar-codigos-de-estado-apropiados)
  *  [Proporcionar recursos completos si están disponibles](#proporcionar-recursos-completos-si-estan-disponibles)
  *  [Aceptar JSON serializado en el cuerpo de las peticiones](#aceptar-json-serializado-en-el-cuerpo-de-las-peticiones)
  *  [Usar formatos de ruta consistentes](#usar-formatos-de-ruta-consistentes)
    *  [Utilizar minúsculas para rutas y atributos](#utilizar-minusculas-para-rutas-y-atributos)
    *  [Soportar referencias por atributos no-identificadores como ayuda](#soportar-referencias-por-atributos-no-identificadores-como-ayuda)
    *  [Minimizar anidado de rutas](#minimizar-anidad-de-rutas)
* [Respuestas](#respuestas)
  *  [Proporcionar identificadores únicos (UUIDs) de recursos](#proporcionar-identificadores-unicos-uuids-de-recursos)
  *  [Proporcionar fechas y horas (timestamps) estándar](#proporcionar-fechas-y-horas-timestamps-estandar)
  *  [Usar horas en UTC formateadas usando ISO8601](#usar-horas-en-utc-formateadas-usando-ISO8601)
  *  [Anidar relaciones de clave foránea](#anidar-relaciones-de-clave-foranea)
  *  [Generar errores estructurados](#generar-errores-estructurados)
  *  [Mostrar el estado del límite de peticiones](#mostrar-el-estado-del-limite-de-peticiones)
  *  [Mantener los JSON minificados en todas las respuestas](#mantener-los-json-minificados-en-todas-las-respuestas)
* [Artefactos](#artifactos)
  *  [Proporcionar un esquema de JSON procesable](#proporcionar-un-esquema-de-json-procesable)
  *  [Proporcionar docmentación para desarrolladores](#proporcionar-docmentacion-para-desarrolladores)
  *  [Proporcionar ejemplos ejecutables](#proporcionar-ejemplos-ejecutables)
  *  [Describir la estabilidad](#describir-la-estabilidad)
* [Traducciones](#tranducciones)

### Fundamentos

#### Separa responsabilidades

Mientras diseñas, simplifica al máximo separando las responsabilidades de las diferentes partes del ciclo de petición y respuesta. Mantener reglas sencillas en esto te permite enfocarte en problemas mayores y más complejos.

Las peticiones y respuestas se realizarán para obtener un recurso o colección en concreto. Usa la ruta (_N.T. el path_) para especificar la entidad, el cuerpo para transferir los contenidos y las cabeceras para especificar metadatos. Los parámetros (_N.T. query params_) pueden ser usados como un medio de pasar información de cabecera en casos extremos, pero es preferible usar las cabeceras ya que son más flexibles y pueden transmitir información más variada.

#### Conexiones seguras requeridas

Haz requerido el acceso seguro al API usando TLS, sin excepciones.
No merece la pena intentar averiguar o explicar cuando se debe permitir acceso con TLS y cuando no. Haz obligatorio TLS para todo.

Idealmente, rechaza cualquier petición sin TLS sin responder a peticiones HTTP o al puerto 80, y así evitar cualquier intercambio de información insegura. En entornos en los que esto no sea posible, retorna con `403 Forbidden`.

Las redirecciones están desaconsejadas ya que permiten a los clientes tener comportamientos incorrectos/chapuceros sin ofrecer ninguna ventaja clara. Los clientes que dependen de las redirecciones duplican el tráfico del servidor and hacen que el TLS sea inutil ya que la información sensible ya ha sido expuesta durante la primera llamada.

#### Versionado con la cabecera Accepts requerido

El versionado y la transición entre versiones puede ser uno de los aspectos más difíciles en el diseño y mantenimiento de un API. Por eso, es mejor empezar con mecanismos para facilitarlo desde el principio.

Para evitar sorpresas, cambios incompatibles para los usuarios, es mejor que la versión sea requerida en todas las peticiones. Las versiones por defecto deben ser evitadas ya que, en el mejor de los casos, son muy difíciles de cambiar en el futuro.

Es mejor especificar la versión en las cabeeras, junto a otros metadatos, usando la cabecera `Accept` junto con un _content type_ personalizado, por ejemplo:

```
Accept: application/vnd.heroku+json; version=3
```

#### Soportar ETags para el cacheado

Incluye una cabecera `ETag` en todas las respuestas, identificando la versión específica del recurso retornado. Esto permite a los usuarios cachear recursos y usar peticiones con este valor en la cabecera `If-None-Match` para especificar si la caché debe ser actualizada.

#### Proporcionar identificadores de petición para introspección

Incluye una cabecear `Request-Id` en cada respuesta del API, junto con un valor UUID. _Logueando_ estos valores en el cliente, en el servidor y cualquier otro servicio de apoyo, se ofrece un mecanismo para _tracear_, diagnosticar y depurar las peticiones.

#### Dividir respuestas largas en varias peticiones usando rangos

Las respuestas largas deben ser troceadas en múltiples peticiones usando la cabecera `Range` para especificar cuando están disponibles más datos y cómo recuperarlos. Consulta en [el debate sobre rangos (en inglés)](https://devcenter.heroku.com/articles/platform-api-reference#ranges) los detalles sobre las cabeceras de las peticiones y respuestas, los códigos de estado, límites, ordenación e iteración.

### Requests

#### Return appropriate status codes

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
  returned: see [above on ranges](#divide-large-responses-across-requests-with-ranges)

Pay attention to the use of authentication and authorization error codes:

* `401 Unauthorized`: Request failed because user is not authenticated
* `403 Forbidden`: Request failed because user does not have authorization to access a specific resource

Return suitable codes to provide additional information when there are errors:

* `422 Unprocessable Entity`: Your request was understood, but contained invalid parameters
* `429 Too Many Requests`: You have been rate-limited, retry later
* `500 Internal Server Error`: Something went wrong on the server, check status site and/or report the issue

Refer to the [HTTP response code spec](https://tools.ietf.org/html/rfc7231#section-6)
for guidance on status codes for user error and server error cases.

#### Provide full resources where available

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

#### Accept serialized JSON in request bodies

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

#### Use consistent path formats

##### Resource names

Use the plural version of a resource name unless the resource in question is a singleton within the system (for example, in most systems a given user would only ever have one account). This keeps it consistent in the way you refer to particular resources.

##### Actions

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

#### Downcase paths and attributes

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

#### Support non-id dereferencing for convenience

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

#### Minimize path nesting

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

### Responses

#### Provide resource (UU)IDs

Give each resource an `id` attribute by default. Use UUIDs unless you
have a very good reason not to. Don’t use IDs that won’t be globally
unique across instances of the service or other resources in the
service, especially auto-incrementing IDs.

Render UUIDs in downcased `8-4-4-4-12` format, e.g.:

```
"id": "01234567-89ab-cdef-0123-456789abcdef"
```

#### Provide standard timestamps

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

#### Use UTC times formatted in ISO8601

Accept and return times in UTC only. Render times in ISO8601 format,
e.g.:

```
"finished_at": "2012-01-01T12:00:00Z"
```

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
    "name": "Alice",
    "email": "alice@heroku.com"
  },
  // ...
}
```

#### Generate structured errors

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

#### Show rate limit status

Rate limit requests from clients to protect the health of the service
and maintain high service quality for other clients. You can use a
[token bucket algorithm](http://en.wikipedia.org/wiki/Token_bucket) to
quantify request limits.

Return the remaining number of request tokens with each request in the
`RateLimit-Remaining` response header.

#### Keep JSON minified in all responses

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

### Artifacts

#### Provide machine-readable JSON schema

Provide a machine-readable schema to exactly specify your API. Use
[prmd](https://github.com/interagent/prmd) to manage your schema, and ensure
it validates with `prmd verify`.

#### Provide human-readable docs

Provide human-readable documentation that client developers can use to
understand your API.

If you create a schema with prmd as described above, you can easily
generate Markdown docs for all endpoints with with `prmd doc`.

In addition to endpoint details, provide an API overview with
information about:

* Authentication, including acquiring and using authentication tokens.
* API stability and versioning, including how to select the desired API
  version.
* Common request and response headers.
* Error serialization format.
* Examples of using the API with clients in different languages.

#### Provide executable examples

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

#### Describe stability

Describe the stability of your API or its various endpoints according to
its maturity and stability, e.g. with prototype/development/production
flags.

See the [Heroku API compatibility policy](https://devcenter.heroku.com/articles/api-compatibility-policy)
for a possible stability and change management approach.

Once your API is declared production-ready and stable, do not make
backwards incompatible changes within that API version. If you need to
make backwards-incompatible changes, create a new API with an
incremented version number.


### Translations
 * [Korean version](https://github.com/yoondo/http-api-design) (based on [f38dba6](https://github.com/interagent/http-api-design/commit/f38dba6fd8e2b229ab3f09cd84a8828987188863)), by [@yoondo](https://github.com/yoondo/)
 * [Simplified Chinese version](https://github.com/ZhangBohan/http-api-design-ZH_CN) (based on [337c4a0](https://github.com/interagent/http-api-design/commit/337c4a05ad08f25c5e232a72638f063925f3228a)), by [@ZhangBohan](https://github.com/ZhangBohan/)
 * [Traditional Chinese version](https://github.com/kcyeu/http-api-design) (based on [232f8dc](https://github.com/interagent/http-api-design/commit/232f8dc6a941d0b25136bf64998242dae5575f66)), by [@kcyeu](https://github.com/kcyeu/)

