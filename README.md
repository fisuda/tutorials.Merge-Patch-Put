# Concise NGSI-LD[<img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"  align="left" />](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.04.01_60/gs_cim009v010401p.pdf)[<img src="https://fiware.github.io/tutorials.Merge-Patch-Put/img/fiware.png" align="left" width="162">](https://www.fiware.org/)<br/>

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Merge-Patch-Put.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
<br/> [![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/)
[![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

This tutorial introduces the NGSI-LD Merge Patch endpoint. It explains the difference between
Merge Patch (`/entities/<id>`) and Partial Update Patch (`/entities/<id>/attrs`) and demonstrates
the use of this function.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://fiware.github.io/tutorials.Merge-Patch-Put/ngsi-ld.html)

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/d24facc3c430bb5d5aaf)

## Contents

<details>
<summary><strong>Details</strong></summary>

-   [Concise NGSI-LD Payloads](#concise-ngsi-ld-payloads)
    -   [NGSI-LD Payloads](#ngsi-ld-payloads)
        -   [Normalized NGSI-LD](#normalized-ngsi-ld)
        -   [Simplified NGSI-LD](#simplified-ngsi-ld)
        -   [Concise NGSI-LD](#concise-ngsi-ld)
-   [Architecture](#architecture)
-   [Prerequisites](#prerequisites)
    -   [Docker and Docker Compose](#docker-and-docker-compose)
    -   [Cygwin for Windows](#cygwin-for-windows)
-   [Start Up](#start-up)
-   [Concise NGSI-LD Operations](#concise-ngsi-ld-operations)
    -   [Create Operations](#create-operations)
        -   [Create a New Data Entity](#create-a-new-data-entity)
        -   [Create New Attributes](#create-new-attributes)
        -   [Batch Create New Data Entities or Attributes](#batch-create-new-data-entities-or-attributes)
        -   [Batch Create/Overwrite New Data Entities](#batch-createoverwrite-new-data-entities)
    -   [Read Operations](#read-operations)
        -   [Filtering](#filtering)
        -   [Read a Data Entity (concise)](#read-a-data-entity-concise)
        -   [Read an Attribute from a Data Entity](#read-an-attribute-from-a-data-entity)
        -   [Read a Data Entity (concise)](#read-a-data-entity-concise-1)
        -   [Read Multiple attributes values from a Data Entity](#read-multiple-attributes-values-from-a-data-entity)
        -   [List all Data Entities (concise)](#list-all-data-entities-concise)
        -   [List all Data Entities (filtered)](#list-all-data-entities-filtered)
        -   [Filter Data Entities by ID](#filter-data-entities-by-id)
        -   [Returning data as GeoJSON](#returning-data-as-geojson)
    -   [Update Operations](#update-operations)
        -   [Overwrite the value of an Attribute value](#overwrite-the-value-of-an-attribute-value)
        -   [Overwrite Multiple Attributes of a Data Entity](#overwrite-multiple-attributes-of-a-data-entity)
        -   [Batch Update Attributes of Multiple Data Entities](#batch-update-attributes-of-multiple-data-entities)
        -   [Batch Replace Entity Data](#batch-replace-entity-data)
    -   [Setting up concise Subscriptions](#setting-up-concise-subscriptions)
        -   [Concise Notification](#concise-notification)
        -   [Concise GeoJSON Notification](#concise-geojson-notification)

</details>

# Merge-Patch and Put

> "Last night I dreamed about you. What happened in detail I can hardly remember, all I
> know is that we kept merging into one another. I was you, you were me. Finally you somehow
> caught fire"
>
> — Franz Kafka, Letters to Milena


The Merge-Patch is a well defined (IETF specification)[https://www.rfc-editor.org/rfc/rfc7396.html]
used to describe a set of modifications to be made on a resource. It uses a JSON payload as a
forensic knife to create, modify or delete individually specified attributes within an entity.
As defined across the Internet, Merge-Patch typically uses JSON payloads, and is an action usually
assigned to the HTTP **PATCH** method. NGSI-LD extends the concept for use with JSON-LD payloads instead.

## Why two kinds of **PATCH** ?


### Partial Update **PATCH**

Partial Update Patch  is supported under two endpoints - `/entities/<id>/attrs` and `/entities/<id>/attrs/<attr-name>`.
The rules of Partial Update **PATCH** is effectively an overwrite at the attribute level.

Given the following NGSI-LD _Property_:

```json
{
  "temperature": {
    "type" : "Property",
    "value" : 25,
    "unitCode": "CEL",
    "observedAt": "2022-03-14T01:59:26.535Z"
  }
}
```

And applying a Partial Update **PATCH** operation `/entities/<id>/attrs/temperature` with the following payload

```json
{
  "type" : "Property",
  "value" : 100,
  "observedAt": "2022-03-14T13:00:00.000Z"
}
```


#### Result 1

Results in an overwrite of the `value`and `observedAt` sub-Attributes, leaving the `unitCode`
sub-Attribute untouched as shown:

```json
{
  "temperature": {
    "type" : "Property",
    "value" : 100,
    "unitCode": "CEL",
    "observedAt": "2022-03-14T13:00:00.000Z"
  }
}
```


However given the same entity and applying a Partial Update **PATCH** operation at the `/entities/<id>/attrs` level

```json
{
  "temperature": {
    "type" : "Property",
    "value" : 100,
    "observedAt": "2022-03-14T13:00:00.000Z"
  }
}
```


#### Result 2

```json
{
  "temperature": {
    "type" : "Property",
    "value" : 100,
    "observedAt": "2022-03-14T13:00:00.000Z"
  }
}
```

Results in an overwrite of the  whole `temperature` property. Note that in the second case `unitCode` has been removed as well.

The idea of Partial Update **PATCH** at an attribute level is to aim for data consistency.
If a sub-attribute such as `observedAt` is  omitted, then it is **not** removed and the existing value remains.
A user is forced to deliberately delete such data using other means.


The idea of Partial Update **PATCH** at an Entity level is to aim for temporal consistency.
It is necessary to resupply all of the sub-attributes within the first attribute layer for each update.
If a sub-attribute such as `observedAt` is  omitted, then it is removed and the existing temporal record
is not affected.

**PATCH** is appropriate for both of these operation since the effect in both cases is an update of
some (but not all) aspects of a selection of _Properties_ or _Properties of Properties_ the
The entity itself, its `id` and `type` remain unchanged.


### Merge **PATCH**

Merge Patch  is supported under `/entities/<id>` and just upserts the attributes found in a payload.
Again starting from the following NGSI-LD _Property_:


```json
 {
  "temperature": {
    "type" : "Property",
    "value" : 25,
    "unitCode": "CEL",
    "observedAt": "2022-03-14T01:59:26.535Z"
  }
}
```

Applying a merge **PATCH** operation `/entities/<id>` with the following payload

```json
{
  "temperature": {
    "type" : "Property",
    "value" : 100,
    "observedAt": "2022-03-14T13:00:00.000Z"
  }
}
```

Results in the update of the just the `value` and `observedAt` sub-Attributes as shown

#### Result 3

```json
{
  "temperature": {
    "type" : "Property",
    "value" : 100,
    "unitCode": "CEL",
    "observedAt": "2022-03-14T13:00:00.000Z"
  }
}
```


The idea of Merge **PATCH** at an Entity level is to correct existing data without regard for temporal consistency
It is not necessary to resupply unchanged sub-attributes.  If a sub-attribute such as `observedAt` is  omitted, then it is left changed. If not careful, this may give rise to unexpected side-effects on the temporal interface
since it is possible to inadvertently update a `value` without updating the `observedAt`.
is not affected.


In summary, both styles of **PATCH** operation have their place, but Merge **PATCH** is more focussed
and is capable of using smaller payloads. Partial Update **PATCH** forces data to be more consistent
through requiring the payload to be complete and removing second-level attributes which are omitted from
the payload. This is also the case for the `/entityOperations/upsert` endpoint meaning that
second-level _Property-of-a-Property_ meta data attributes must always be include in the payload if
they do not want to be deleted. Therefore _Properties_ within `/entityOperations/upsert` payload
will usually also include `unitCode` and `observedAt` as well as an updated `value`.

### Overwrite **PUT**

HTTO **PUT** is supported under `/entities/<id>` and just overwrites the data within an existing entity.
In this case whole entity is overwritten, a payload such as the one below would result in an entity
with a single `temperature` attribute.


```json
 {
  "temperature": {
    "type" : "Property",
    "value" : 25,
    "unitCode": "CEL",
    "observedAt": "2022-03-14T01:59:26.535Z"
  }
}
```


**PUT** is guaranteed to be an idemponent operation. Calling it several times will result in the same system state, whereas **PATCH** not necessarily idemponent, and a **PATCH** payload of either flavor
(merge patch or partial update patch), does not need supply every attribute from the entire entity data each time.


# Architecture

The required architecture will consist of three elements:

-   The [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
-   The underlying [MongoDB](https://www.mongodb.com/) database :
    -   Used by the Orion Context Broker to hold context data information such as data entities, subscriptions and
        registrations
-   An HTTP **Web-Server** which offers static `@context` files defining the context entities within the system.

Since all interactions between the three elements are initiated by HTTP requests, the elements can be containerized and
run from exposed ports.

![](https://fiware.github.io/tutorials.CRUD-Operations/img/architecture-ld.png)

The necessary configuration information can be seen in the services section of the associated `docker-compose.yml` file.
It has been described in a previous tutorial.

# Prerequisites

## Docker and Docker Compose

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A series of
[YAML files](https://raw.githubusercontent.com/FIWARE/tutorials.Merge-Patch-Put/NGSI-LD/docker-compose.yml) are used to
configure the required services for the application. This means all container services can be brought up in a single
command. Docker Compose is installed by default as part of Docker for Windows and Docker for Mac, however Linux users
will need to follow the instructions found [here](https://docs.docker.com/compose/install/).

You can check your current **Docker** and **Docker Compose** versions using the following commands:

```console
docker-compose -v
docker version
```

Please ensure that you are using Docker version 20.10 or higher and Docker Compose 1.29 or higher and upgrade if
necessary.

## Cygwin for Windows

We will start up our services using a simple Bash script. Windows users should download [cygwin](http://www.cygwin.com/)
to provide a command-line functionality similar to a Linux distribution on Windows.

# Start Up

Before you start, you should ensure that you have obtained or built the necessary Docker images locally. Please clone
the repository and create the necessary images by running the commands as shown:

```console
git clone https://github.com/FIWARE/tutorials.Merge-Patch-Put.git
cd tutorials.Merge-Patch-Put
git checkout NGSI-LD

./services create
```

Thereafter, all services can be initialized from the command-line by running the
[services](https://github.com/FIWARE/tutorials.Merge-Patch-Put/blob/NGSI-LD/services) Bash script provided within the
repository:

```console
./services start
```

This start-up script also preloads two City entities into the context broker.


> :information_source: **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```console
> ./services stop
> ```

---

# Merge Patch Operations



## Preflight

The Orion context broker supports the **OPTIONS** method to enable users to request the supported operations. The two types of **PATCH** operation can be distinguished by reading the `Accept-Patch` header.

#### :one: Request:

```console
curl -iX OPTIONS \
    'http://localhost:1026/ngsi-ld/v1/entities/'
```

#### Response:

The `/entities/` endpoint supports **GET** and **POST** operations only.

```text
HTTP/1.1 200 OK
Date: Tue, 06 Dec 2022 09:36:42 GMT
Allow: GET,POST,OPTIONS
Content-Length: 0
```


#### :two: Request:

```console
curl -iX OPTIONS \
    'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001'
```

#### Response:

The `/entities/<entity-id>` endpoint supports **GET**,**PUT**,**DELETE** and **PATCH** operations only. Because **PATCH** is supported an additional `Accept-Patch` header is returned listing the appropriate payload types. It
can be noted that this list includes `application/merge-patch+json` since `/entities/<entity-id>` is a merge-patch endpoint

```text
HTTP/1.1 200 OK
Date: Tue, 06 Dec 2022 09:49:43 GMT
Accept-Patch: application/json, application/ld+json, application/merge-patch+json
Allow: GET,PUT,DELETE,PATCH,OPTIONS
Content-Length: 0
```

#### :three: Request:


```console
curl -iX OPTIONS \
    'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001/attrs'
```


#### Response:

`/entities/<entity-id>/attrs/temperature` endpoint supports **POST** and **PATCH** operations only. The
additional `Accept-Patch` header does not include  `application/merge-patch+json` since `/entities/<entity-id>/attrs` is a partial update patch endpoint

```text
HTTP/1.1 200 OK
Date: Tue, 06 Dec 2022 09:49:27 GMT
Accept-Patch: application/json, application/ld+json
Allow: POST,PATCH,OPTIONS
Content-Length: 0
```


#### :four: Request:

```console
curl -iX OPTIONS \
    'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001/attrs/temperature'
```

#### Response:

`/entities/<entity-id>/attrs/<attr>` endpoint supports **DELETE** and **PATCH** methods only. The
additional `Accept-Patch` header does not include  `application/merge-patch+json` since `/entities/<entity-id>/attrs/<attribute>` is a partial update patch endpoint

```text
HTTP/1.1 200 OK
Date: Tue, 06 Dec 2022 09:49:10 GMT
Accept-Patch: application/json, application/ld+json
Allow: DELETE,PATCH,OPTIONS
Content-Length:
```


## Merge-Patch Operations


Note that two preexisting entities have been created for amendment by Merge-Patch, the current state of the  entities can be obtained by making a **GET** request to the `/entities/<entity-id>` endpoint.
These requests can be made to see how the entities have changed after each operation.

#### :five: Request:

```console
curl -L -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json'
```

#### Response:

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                28.955,
                41.0136
            ]
        }
    },
    "population": {
        "type": "Property",
        "value": 15840900,
        "observedAt": "2022-12-31T00:00:00.000Z"
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Kanlıca İskele Meydanı",
            "addressRegion": "İstanbul",
            "addressLocality": "Beşiktaş",
            "postalCode": "12345"
        }
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "el": "Κωνσταντινούπολις",
            "en": "Constantinople",
            "tr": "İstanbul"
        }
    },
    "runBy": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Adminstration:Cumhuriyet_Halk_Partisi"
    }
}
```


#### :six: Request:

```console
curl -L -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:002' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'options=concise'
```

#### Response:

```json
{
    "id": "urn:ngsi-ld:City:002",
    "type": "City",
    "temperature": {
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "address": {
        "value": {
            "streetAddress": "Viale di Valle Aurelia",
            "addressRegion": "Lazio",
            "addressLocality": "Roma",
            "postalCode": "00138"
        }
    },
    "location": {
        "type": "Point",
        "coordinates": [
            12.482,
            41.893
        ]
    },
    "population": {
        "value": 4342212,
        "observedAt": "2021-01-01T00:00:00.000Z"
    },
    "name": {
        "languageMap": {
            "el": "Ρώμη",
            "en": "Rome",
            "it": "Roma"
        }
    },
    "runBy": {
        "object": "urn:ngsi-ld:Adminstration:Partito_Democratico"
    }
}
```



### Updating using Merge Patch

This example moves the city `location` to 52.5146 N,13.350 E and amends the `temperature` to 20.
The data here is in normalized format, but concise format is also supported:

#### :seven::A: Request:

```console
curl -L -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "temperature": {
        "type": "Property",
        "value": 25
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                28.955,
                41.0136
            ]
        }
    }
}'
```


#### :seven::B: Request:

```console
curl -L -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "temperature": 20,
    "location": {
        "type": "Point",
        "coordinates": [
            13.3505,
            52.5146
        ]
    }
}'
```

#### :eight: Request:

Re-retrieving the `urn:ngsi-ld:City:001`, you can see that the `location` and `temperature` have changed, but all other _Properties_ and _Properties of Properties_ such as `unitCode` ands `observedAt` remain unchanged:


```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json'
```


#### Response:

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 20,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                13.3505,
                52.5146
            ]
        }
    },
    "population": {
        "type": "Property",
        "value": 15840900,
        "observedAt": "2022-12-31T00:00:00.000Z"
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Kanlıca İskele Meydanı",
            "addressRegion": "İstanbul",
            "addressLocality": "Beşiktaş",
            "postalCode": "12345"
        }
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "el": "Κωνσταντινούπολις",
            "en": "Constantinople",
            "tr": "İstanbul"
        }
    },
    "runBy": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Adminstration:Cumhuriyet_Halk_Partisi"
    }
}
```



### Adding new attributes using Merge Patch

For a merge patch operation, if an attribute is included in the payload but missing on the
entity, it is inserted. In this example, the `temperature` attribute is being updated,
the `value` and `observedAt` have been updated, but the `precision` _Property of a Property_
is being inserted.

As usual, both normalized and concise formats are supported.The default for unknown attributes is _Property_, to insert a concise _Relationship_ or a _LanguageProperty_ include `object` or `languageMap` as expected.

#### :nine::A: Request:

```console
curl -L -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "temperature": {
        "type": "Property",
        "value": 7,
        "observedAt": "2022-03-14T12:51:02.000Z",
        "precision": {
            "value": 0.95,
            "type": "Property",
            "unitCode": "C62"
        }
    }
}'
```

#### :nine::B: Request:

```console
curl -L -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "temperature": {
        "value": 7,
        "observedAt": "2022-03-14T12:51:02.000Z",
        "precision": {
            "value": 0.95,
            "unitCode": "C62"
        }
    }
}'
```

#### :one::zero: Request:

Re-retrieving the `urn:ngsi-ld:City:001`, you can see that the `temperature` have changed, and the
new `precision` _Property of a Property_ has been inserted.


```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'attrs=temperature' \
```

#### Response:

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 7,
        "unitCode": "CEL",
        "observedAt": "2022-03-14T12:51:02.000Z",
        "precision": {
            "type": "Property",
            "value": 0.95,
            "unitCode": "C62"
        }
    }
}
```

### Removing existing attributes using Merge Patch

Usually `null` is used in merge patch operations to indicate deletion. However,
JSON-LD does not support this (since an attribute with a `null` is always removed from the payload
on expansion) and therefore NGSI-LD uses a placeholder value `urn:ngsi-ld:null` instead.
Note that `urn:ngsi-ld:null` is equally valid for deletion of  a _Property_, _Relationship_ or
a _LanguageProperty_. In the concise example below, an insertion, an update and a deletion can be
applied simultaneously.

#### :one::one: Request:

```console
curl -L -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:002' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "humidity": 80,
    "name": {
        "languageMap": {
            "el": "Βερολίνο",
            "en": "Berlin",
            "it": "Berlino"
        }
    },
    "temperature": "urn:ngsi-ld:null"
}'

```

#### :one::two: Request:

Re-retrieving the `urn:ngsi-ld:City:002`, you can see that the `temperature` has been removed and the
new `humidity` _Property_ has been inserted and the `name` updated.

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:002' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'options=concise'
```

#### Response:

```json
{
    "id": "urn:ngsi-ld:City:002",
    "type": "City",
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Viale di Valle Aurelia",
            "addressRegion": "Lazio",
            "addressLocality": "Roma",
            "postalCode": "00138"
        }
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                12.482,
                41.893
            ]
        }
    },
    "population": {
        "type": "Property",
        "value": 4342212,
        "observedAt": "2021-01-01T00:00:00.000Z"
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "el": "Βερολίνο",
            "en": "Berlin",
            "it": "Berlino"
        }
    },
    "runBy": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Adminstration:Partito_Democratico"
    },
    "humidity": {
        "type": "Property",
        "value": 80
    }
}
```


### Amending values of a Property with sub-attributes


#### :one::three: Request:

Using concise format, it is necessary to distinguish between atttibutes of a JSON object and _Properties of Properties_. In this case the use of `value` shows it is the `addressLocality` and  `postalCode`
of the `address` Object which is to be updated and that `verified` the _Property of a Property_
is a meta data attribute.

```console
curl -L -X PATCH \
    'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "address": {
        "value": {
            "addressLocality": "Fenerbahçe",
            "postalCode": "34567"
        },
        "verified": "true"
    }
}'
```

#### :one::two: Request:

Once again retrieving the `urn:ngsi-ld:City:001`, you can see that the `address` and its properties have been updated.

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'attrs=address' \
```

#### Response:

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Kanlıca İskele Meydanı",
            "addressRegion": "İstanbul",
            "addressLocality": "Fenerbahçe",
            "postalCode": "34567"
        },
        "verified": {
            "type": "Property",
            "value": "true"
        }
    }
}
```


#### Updating using key-values format

Merge-Patch also offers some limited support to update `values` using key-values format. In this case
any existing `value` is updated, but no meta data is changed.  Once again Object values need only send
the sub-attributes to be updated, and setting a sub-attribute to `urn:ngsi-ld:null` will cause it to
be deleted.

This means that is is possible to **GET** a key-values entity, amend the values and **PATCH** it back
to the context broker.

#### :one::three: Request:


```console
curl -G -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-d 'options=keyValues' \
-H 'Content-Type: application/json' \
--data-raw '{
    "temperature": 19,
    "location": {
        "type": "Point",
        "coordinates": [
            13.3505,
            52.5146
        ]
    },
    "address": {
        "addressLocality": "Beyoğlu",
        "postalCode": "98765"
    },
    "runBy": "urn:ngsi-ld:Adminstration:Adalet_ve_Kalkınma_Partisi"
}'
```


#### :one::two: Request:

Once again retrieving the `urn:ngsi-ld:City:001` entity, you can see that the attributes have been updated. It should be noted that the _Relationship_ `runBy` is still defined as a  _Relationship_, it is
only the `object` value that has been changed.

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'attrs=address,temperature,location,runBy' \
```


#### Response:

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 19,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                13.3505,
                52.5146
            ]
        }
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Kanlıca İskele Meydanı",
            "addressRegion": "İstanbul",
            "addressLocality": "Beyoğlu",
            "postalCode": "98765"
        }
    },
    "runBy": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Adminstration:Adalet_ve_Kalkınma_Partisi"
    }
}

```


### Updating using key-values with `observedAt`

Since `observedAt` is exceedingly important for maintaining consistent temporal data within the
context broker it is offered as an additional parameter when updating key-values during merge-patch.
If a pre-existing  _Property_ already uses `observedAt` _Property of a Property of a Property_, the
timestamp will also be updated.

The following example updates both the location and temperature attributes


#### :one::three: Request:

```console
curl -G -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
-d 'options=keyValues' \
-d 'observedAt=2022-10-10T10:10:00.000Z' \
--data-raw '{
    "temperature": 19,
    "location": {
        "type": "Point",
        "coordinates": [
            13.3505,
            52.5146
        ]
    }
}'
```


Once again retrieving the `urn:ngsi-ld:City:001` entity, you can see that the attributes have been updated, and this time the timestamp has also changed.


#### :one::four: Request:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'attrs=temperature,location' \
```


#### Response:

Note that `observedAt` is only ever updated. It is not added to a _Property_ where it was not
present previously.

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 19,
        "unitCode": "CEL",
        "observedAt": "2022-10-10T10:10:00.000Z"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                13.3505,
                52.5146
            ]
        }
    }
}
```


### Updating using key-values with `lang`

When retrieving an entity using **GET**, the `lang` parameter switches the attribute type from a `languageMap` to a single string or string array. This is obviously a lossy operation, and in order
for key-values merge patch fully support entities with LanguageProperties, it is necessary to be
able to merge a simple string value back into a `languageMap`.

#### :one::five: Request:

```console
curl -G -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
-d 'options=keyValues' \
-d 'lang=en'
--data-raw '{
    "temperature": 19,
    "population": 15850000,
    "name": "Istanbul, not Constantinople"
}'
```


#### :one::six: Request:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'options=keyValues' \
-d 'attrs=temperature,population,name'
```

#### Response:

As can be seen, `name` attribute in English (`en`) has been updated, whereas the values in Greek (`el`)
and Turkish (`tr`) have been left untouched.

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 19,
        "unitCode": "CEL",
        "observedAt": "2022-10-10T10:10:00.000Z"
    },
    "population": {
        "type": "Property",
        "value": 15850000,
        "observedAt": "2022-12-31T00:00:00.000Z"
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "el": "Κωνσταντινούπολις",
            "en": "Istanbul, not Constantinople",
            "tr": "İstanbul"
        }
    }
}
```

### Overwriting an entity with **PUT**

For an overwrite operation, if an existing attribute is not included in the payload
entity, it is deleted. In this example, the `temperature`, `population` and `name` are being updated,
any other attributes on the entity will be deleted.

As usual, both normalized and concise formats are supported.

#### :one:seven::A: Request:

```console
curl -G -X PUT \
 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:002' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{

    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                12.482,
                41.893
            ]
        }
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "el": "Ρώμη",
            "en": "Rome",
            "it": "Roma"
        }
    }
}'
```


#### :one:seven::B: Request:

```console
curl -G -X PUT \
 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:002' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{

    "type": "City",
    "temperature": {
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "location": {
        "type": "Point",
        "coordinates": [
            12.482,
            41.893
        ]
    },
    "name": {
        "languageMap": {
            "el": "Ρώμη",
            "en": "Rome",
            "it": "Roma"
        }
    }
}'
```


# Next Steps

Want to learn how to add more complexity to your application by adding advanced features? You can find out by reading
the other [tutorials in this series](https://ngsi-ld-tutorials.rtfd.io)

---

## License

[MIT](LICENSE) © 2022 FIWARE Foundation e.V.
