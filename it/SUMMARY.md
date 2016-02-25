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

Preferisci dei layout per gli endpoint che non prevedono azioni speciali per una determinata risorsa.
Nel caso in cui sia necessario prevedere la possibilità di azioni speciali, specificale sotto un prefisso standard come `actions`, per definirle in modo chiaro:

```
/resources/:resource/actions/:action
```

es.

```
/runs/{run_id}/actions/stop
```

#### Usa un formato consistente per i percorsi (path)

#### Caratteri minuscoli per percorsi e attributi

Usa dei percorsi in minuscolo e separati da trattini, per congruenza con gli hostnames, es:

```
service-api.com/users
service-api.com/app-setups
```

Anche per gli attributi utilizza il minuscolo, ma separando le parole con un underscore, in modo che possano essere scritti
senza l'utilizzo delle virgolette in JavaScript, es:

```
service_class: "first"
```

#### Supporta anche riferimenti non-id per comodita

In alcuni casi potrebbe essere scomodo per gli utenti finali, fornire ID per 
identificare una risorsa. Per esempio, un utente può pensare in termini del nome di una app, ma questa app può essere identificata tramite un UUID. In questi casi, potresti prevedere di accettare entrambi: ID e nome, es:

```bash
$ curl https://service.com/apps/{app_id_or_name}
$ curl https://service.com/apps/97addcf0-c182
$ curl https://service.com/apps/www-prod
```
Non accettare comunque soltanto un nome, senza la possibilità di specificare un ID

#### Minimizza annidamento dei percorsi

Nei model che rappresentano i nostri dati con le relazioni padre/figlio annidate, i percorsi possono diventare
veramente molto lunghi, es:

```
/orgs/{org_id}/apps/{app_id}/dynos/{dyno_id}
```

Limita l'annidamento, preferendo la localizzazione delle risorse nella radice del persorso.
Usa l'annidamento per indicare una raccolta di elementi. Per esempio,
per il caso sopra, dove dyno dipende da app che dipende da org:

```
/orgs/{org_id}
/orgs/{org_id}/apps
/apps/{app_id}
/apps/{app_id}/dynos
/dynos/{dyno_id}
```

### Risposte

La sezione delle risposte fornisce una panoramica sui pattern da utilizzare per le risposte della API

#### Ritorna sempre un codice di stato appropriato

Ritorna un codice di stato HTTP appropriato per ogni risposta.
Le risposte con esito positivo, dovrebbero essere accompagnate da codici di stato come di seguito:

* `200`: Richiesta completata con successo per una chiamata (sincrona) `GET`, `DELETE` o
  `PATCH`, oppure per una chiamata `PUT` (sincrona) che ha completato l'update di una risorsa
* `201`: Richiesta completata con successo per una chiamata (sincrona) `POST`, o `PUT` che ha creato una nuova risorsa
* `202`: Richiesta completata con successo per una chiamata `POST`, `PUT`, `DELETE`, o `PATCH` che viene processata in modo asincrono
* `206`: Richiesta completata con successo per una chiamata `GET`, dove pero viene restituita una risposta parziale: vedi [Dividi risposte molto lunghe in piu richieste con range](#dividi-risposte-molto-lunghe-in-piu-richieste-con-range)

Fai molta attenzione all'utilizzo dei codici di errore per l'autenticazione e l'autorizzazione:

* `401 Unauthorized`: Richiesta fallita per utente non autenticato
* `403 Forbidden`: Richiesta fallita perchè l'utente non è autorizzato ad accedere ad una risorsa

Ritorna codici di errore adatti fornendo informazioni aggiuntive sul tipo di errore:

* `422 Unprocessable Entity`: La tua richiesta è stata compresa, ma contiene dei parametri non validi.
* `429 Too Many Requests`: Hai superato il limite di richieste, riprova più tardi
* `500 Internal Server Error`: Qualcosa è andato storto, controlla lo stato del sito/servizio ed eventualmente segnala l'errore

Fai sempre riferimento alle specifiche [HTTP response code spec](https://tools.ietf.org/html/rfc7231#section-6)
per i codici di stato e per gli errori

#### Fornisci le risorse interamente, quando possibile

Fornisci la rappresentazione dell'intera risorsa (es. oggetto con tutti gli attributi)
quando possible nella risposta. Fornisci sempre la risorsa completa con un codice 200 o 201, incluse nelle richieste 
`PUT`/`PATCH` e `DELETE`, es:

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

le risposte con codice 202 non includono l'intera rappresentazione della risorsa,
es:

```bash
$ curl -X DELETE \
  https://service.com/apps/1f9b/dynos/05bd

HTTP/1.1 202 Accepted
Content-Type: application/json;charset=utf-8
...
{}
```

#### Fornisci gli (UU)IDs delle risorse

Assegna ad ogni risorssa un attributo `id` di default. Usa gli UUIDs a meno che tu
non abbia una buona ragione per non farlo. Non usare gli IDs perchè non sono globalmente 
unici, specialmente quando si hanno più instanze del servizio o delle risorse nel servizio, specialmente
gli IDs auto-incrementali.

Visualizza gli UUIDs in formato `8-4-4-4-12` minuscolo, es:

```
"id": "01234567-89ab-cdef-0123-456789abcdef"
```

#### Fornisci dei timestamps standard

Fornisci di default i timestamp `created_at` e `updated_at` per le risorse, es:

```javascript
{
  // ...
  "created_at": "2012-01-01T12:00:00Z",
  "updated_at": "2012-01-01T13:00:00Z",
  // ...
}
```

Questi timestamp possono non avere senso per alcune risorse, in questo caso possono 
essere omessi.

#### Usa date in formato UTC formattate in ISO8601

Accetta e ritorna le date solo in formato UTC. Visualizza le date nel formato
ISO8601, es:

```
"finished_at": "2012-01-01T12:00:00Z"
```

#### Annida relazioni tramite le chiavi esterne (foreign-key)

Serializza i riferimenti delle chiavi esterne con un oggetto annidato, es:

```javascript
{
  "name": "service-production",
  "owner": {
    "id": "5d8201b0..."
  },
  // ...
}
```

Invece di, es:

```javascript
{
  "name": "service-production",
  "owner_id": "5d8201b0...",
  // ...
}
```

Questo approccio rende possibile la visualizzazione di più informazioni sulla risorsa
in questione senza dover cambiare la struttura della risposta o introdurre altri campi nella risposta stessa, es:

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

Genera i corpi di risposta degli errori in modo consistente e strutturato.
Includi un riferimento `id` all'errore in modo che sia leggibile da una macchina, e un
campo `message` che sia comprensible all'utente e, opzionalmente un campo `url` che porta 
l'utente ad una descrizione più dettagliata dell'errore in questione e come risolverlo, es:

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

Documenta il formato dell'errore e il possibile error `id` che l'utente può incontrare.

#### Visualizza lo stato del limite delle richieste

Misura i limiti di richieste dai client per proteggere la stabilità del servizio e mantenere una
qualita altà per gli altri clients. Puoi usare un 
[token bucket algorithm](http://en.wikipedia.org/wiki/Token_bucket) per misurare e monitorare il
limite delle richieste

In questo caso, ritorna il numero di richieste rimanenti in ogni richiesta nell'header
`RateLimit-Remaining`.

#### Mantieni il JSON minimizzato in tutte le risposte

Gli spazi bianchi extra aumentano inutilmente la dimensione delle richieste/risposte,
inoltre molti clients adottano automaticamente processi di "prettify" sulle risposte JSON. Rimane una scelta
migliore mantenere le risposte JSON minimizzate, es:

```json
{"beta":false,"email":"alice@heroku.com","id":"01234567-89ab-cdef-0123-456789abcdef","last_login":"2012-01-01T12:00:00Z","created_at":"2012-01-01T12:00:00Z","updated_at":"2012-01-01T12:00:00Z"}
```

Invece di, es:

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

Puoi considerare di fornire opzionalmente ai clients l'opportunità di recuperare delle risposte
più verbose (es. `?pretty=true`) o utilizzando un header `Accept` (es. `Accept: application/vnd.heroku+json; version=3; indent=4;`).

### Artefatti

La sezione artefatti descrive gli oggetti che vengono utilizzati per gestire la progettazione delle API

#### Fornisci uno schema JSON interpretabile

Fornisci uno schema JSON interpretabile per descrivere in maniera formale e precisa la tua API.
Puoi usare [prmd](https://github.com/interagent/prmd) per gestire lo schema e assicurarti della sua 
validità usando il comando `prmd verify`.

#### Fornisci una documentazione comprensibile allo sviluppatore

Fornisci una documentazione comprensibile che lo sviluppatore e i clients possono consultare per
usare la tua API.

Se crei uno schema utilizzando prmd come descritto sopra, potrai facilmente generare un documento in formato
Markdown per tutti gli endpoints usando il comando `prmd doc`.

Oltre alle specifiche degli endpoints, fornisci una panoramica della API con le seguenti 
informazioni:

* Autenticazione, specificando come ottenere ed usare un access token
* Stabilità della API e versionamento, specificando come scegliere la versione della API
* I comuni headeer delle richieste e delle risposte
* Errori in formato serializzato
* Esempi di utilizzo della tua API in diversi linguaggi

#### Fornisci degli esempi

Fornisci dei semplici esempi eseguibili che gli utenti possano provare 
velocemente tramite i loro terminali, per vedere il funzionamento delle chiamate alle API.
Per essere più chiaro possibile, descrivi gli esempi in modo molto dettagliato, per diminuire
in maniera considerevole il lavoro dell'utente che dovrà provare ed usare l'API, es:

```bash
$ export TOKEN=... # acquire from dashboard
$ curl -is https://$TOKEN@service.com/users
```

Se usi [prmd](https://github.com/interagent/prmd) per generare la documentazione in formato Markdown,
otterrai automaticamente degli esempi per ogni endpoint.

#### Specifica la stabilità della tua API

Specifica la stabilità della tua API oppure dei vari endpoints per quanto riguarda la maturità e la stabilità
della stessa, es. tramite dei flag prototype/development/production.

Dai un'occhiata a [Heroku API compatibility policy](https://devcenter.heroku.com/articles/api-compatibility-policy)
per possibili cambiamenti nella stabilità o nella gestione delle policy.

Una volta che la tua API è pronta per andare in produzione ed è stabile, non fare cambiamenti non retrocompatibili.
Se hai bisogno di fare dei cambiamenti non retrocompatibili, crea una nuova API con un nuovo numero di versione.



