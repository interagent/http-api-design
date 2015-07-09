# Gild Platform - HTTP API Design Guide

## Introduction

This guide describes a set of HTTP+JSON API design practices used in 
the development of the [Gild Platform API](https://api.gild.com/). 
The guide is based on the work of [http-api-design](https://github.com/interagent/http-api-design) 
and [JSON API](http://jsonapi.org/format/).

This guide informs additions to that API and also guides new internal APIs at Gild.

We assume you’re familiar with the basics of HTTP+JSON APIs and won’t
cover all of the fundamentals of those in this guide.

This guide is referred in all public and private Pull Requests / code reviews.

## Contents

* [Foundations](#foundations)
  *  [Separation of Concerns](#separation-of-concerns)
  *  [Require Secure Connections](#require-secure-connections)
  *  [Require Versioning in the Accepts Header](#require-api-versioning-in-path)
  *  [Deprecation policy](#deprecation-policy)
  *  [X-Request-Ids Header for Introspection](#x-request-ids-header-for-introspection)
* [Requests](#requests)
  *  [Accept serialized JSON in request bodies](#accept-serialized-json-in-request-bodies)
  *  [Request Content Types](#request-content-types)
  *  [Consistent path formats](#consistent-path-formats)
    *  [Downcase paths and attributes](#downcase-paths-and-attributes)
    *  [Support non-id dereferencing for convenience](#support-non-id-dereferencing-for-convenience)
    *  [Minimize path nesting](#minimize-path-nesting)
* [Responses](#responses)
  *  [Status codes](#status-codes)
  *  [Response Content Types](#response-content-types)
  *  [Full resources where available](#full-resources-where-available)
  *  [Resource (UU)IDs](#resource-uuids)
  *  [Standard timestamps](#standard-timestamps)
  *  [UTC times formatted in ISO8601](#utc-times-formatted-in-iso8601)
  *  [Nested foreign key relations](#nested-foreign-key-relations)
  *  [Show rate limit status](#show-rate-limit-status)
  *  [JSON minified in all responses](#json-minified-in-all-responses)

### Foundations

#### Separation of Concerns

We want to keep things simple by separating the concerns between the
different parts of the request and response cycle. Keeping simple rules here
allows for greater focus on larger and harder problems.

Requests and responses will be made to address a particular resource or
collection. We are using:
 
 * the path to indicate identity
 * the body to transfer the contents
 * headers to communicate metadata

#### Require Secure Connections

We require secure connections with TLS to access the API, without any exception.
It’s not worth trying to figure out or explain when it is OK to use TLS
and when it’s not, so we just require TLS for everything.

We reject any non-TLS requests by not responding to requests for
http or port 80 to avoid any insecure data exchange.

#### Require API Versioning in path

Versioning and the transition between versions can be one of the more
challenging aspects of designing and operating an API. As such, we want to address
this from the start, by including the API version as part of the API identity (i.e. path).

Each API method is prefixed with the version of the API it refers to.

For example:

    .../v1/resources/:id
    
    .../v2/interviews/

#### Deprecation policy

Individual API methods and entire endpoints can be deprecated in favor of new or alternative endpoints.
Most of the changes are due to more solid or better way to access the same data - for example for consistency
or to favor API client patterns. 

When that happens we notify the change on [the deprecation manifest](https://github.com/Gild/gild-api/blob/master/README.md) 
as well as on the API documentation and the [Hiring Success mailing list](https://groups.google.com/forum/#!forum/hiring_success).

You can check the [migration guidelines](https://github.com/Gild/gild-api/blob/master/migration.md) too.
    
When a method or an endpoint gets deprecated, the old method or endpoint will be still supported for 120 days after the 
deprecation notice - unless the change is introduced as response to a security concern, where the safety of the data
prevents us to give a more extended notice.

#### X-Request-Ids Header for Introspection

The Gild API includes a `X-Request-Id` header in each API response, populated with a
UUID value. By logging these values on the client, server and any backing
services, we provides a mechanism to trace, diagnose and debug requests.

### Requests

#### Accept serialized JSON in request bodies

The Gild API accepts serialized JSON on `PUT`/`PATCH`/`POST` request bodies. 
This creates symmetry with JSON-serialized response bodies, e.g.:

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

#### Request Content Types

The Gild API support the following content types in API requests:

* application/json
* application/x-www-form-urlencoded

#### Consistent path formats

##### Resource names

Resource names are in plural version unless the resource in question is a singleton within the system. 
This keeps it consistent in the way as an API client you refer to particular resources.

##### Actions

The Gild API is structured with endpoint layouts that don’t need any special actions for
individual resources. In cases where special actions are needed, they are placed under a standard `actions` prefix, 
to clearly delineate them:

```
/resources/:resource/actions/:action
```

e.g.

```
/interviews/{inteview_id}/actions/stop
```

Prefer actions when the endpoint means to act on the resource atomically and does not requires further specifications/params.

```ruby
POST /articles/1/actions/publish
```

Prefer a restful representation when the endpoint requires extra specifications and you can represent it as a CRUD action over a resource. Use an action if the CRUD operation will have more side effetcs like creating more resources or active on several layer of business logic. 

```ruby
POST /users/invitation
#requires params in the body
```

#### Downcase paths and attributes

The Gild API uses downcased and dash-separated path names, for alignment with
hostnames, e.g:

```
service-api.com/users
service-api.com/app-setups
```

Attributes are downcase as well, but underscore are used as separators so that
attribute names can be typed without quotes in JavaScript, e.g.:

```
service_class: "first"
```

#### Support non-id dereferencing for convenience

In some cases the API accepts multiple types of ids to identify a resource.
For example, a user may think in terms of a company domain name, but that company may be identified by a UUID. In these cases the API
accepts both an id or name, e.g.:

```bash
$ curl https://service.com/companies/{company_id_or_domain}
$ curl https://service.com/companies/97addcf0-c182
$ curl https://service.com/companies/gild.com
```

#### Minimize path nesting

In data models with nested parent/child resource relationships, paths
may become deeply nested, e.g.:

```
/companies/{company_id}/jobs/{job_id}/profiles/{profile_id}
```

The Gild API is designed to limit nesting depth by preferring to locate resources at the root path. 
We use nesting to indicate scoped collections. 

For example, for the case above where a profile belongs to a job that belongs to a company:

```
/companies/{company_id}
/companies/{company_id}/jobs
/jobs/{job_id}
/jobs/{job_id}/profiles
/profiles/{profile_id}
```

### Responses

#### Status codes

The Gild API returns an appropriate HTTP status codes with each response. 

Successful responses are handled according to this guide:

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

Authentication or authorization failures are signalled by:

* `401 Unauthorized`: Request failed because user is not authenticated
* `403 Forbidden`: Request failed because user does not have authorization to access a specific resource

Other failures are handled by the following HTTP status codes:

* `422 Unprocessable Entity`: Your request was understood, but contained invalid parameters
* `429 Too Many Requests`: You have been rate-limited, retry later
* `500 Internal Server Error`: Something went wrong on the server, check status site and/or report the issue

#### Response Content Types

The Gild API support the following content types as API response:

* application/json
* text/xml

#### Full resources where available

The Gild API provides a full resource representation (i.e. the object with all
attributes) whenever possible in the response. The API always provides the full
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

#### Resource (UU)IDs

Each Gild API resource has an `uuid` attribute by default. 
Each UUID is globally unique across every instances and resources of the service.

UUIDs are in downcased `8-4-4-4-12` format, e.g.:

```
"uuid": "01234567-89ab-cdef-0123-456789abcdef"
```

#### Standard timestamps

The API entities contain a `created_at` and `updated_at` timestamps for resources by default,
e.g:

```javascript
{
  // ...
  "created_at": "2012-01-01T12:00:00Z",
  "updated_at": "2012-01-01T13:00:00Z",
  // ...
}
```

These timestamps may not make sense for some resources, in which case they are omitted.

#### UTC times formatted in ISO8601

The API accepts and returns times in UTC only. Times are rendered in ISO8601 format,
e.g.:

```
"finished_at": "2012-01-01T12:00:00Z"
```

#### Nested foreign key relations

Foreign key references are serialized with a nested object, e.g.:

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

#### Rate limit status

The Gild API has a rate limit for requests from clients to protect the health of the service
and maintain high service quality for other clients. 

The API returns the remaining number of request tokens with each request in the
`X-RateLimit-Remaining` response header.

#### JSON minified format

Extra whitespace adds needless response size to requests, and many
clients for human consumption will automatically "prettify" JSON
output. For this reason the Gild API response is minified e.g.:

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

The API supports a query parameter `pretty=true` to retrieve a more human readable response.
