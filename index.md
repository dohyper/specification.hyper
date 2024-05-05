# specification.hyper [v1.0.0](https://youtu.be/Gd_70kdfZXw?t=19)-draft


## 1. Status

This is the initial release of the specification.hyper. v1.0.0-draft is a **working draft**. As such, the content on this page is subject to change.

If you have concerns about the changes in this draft, catch an error in the specification’s text, or write an implementation, please let us know by opening an issue or pull request at our [GitHub repository](https://github.com/dohyper/specification.hyper).

You can also propose additions to specification.hyper in our [discussion forum](https://github.com/dohyper/specification.hyper/discussions). Keep in mind, though, that all new versions of specification.hyper **must be backwards compatible** using a _never remove, only add_ strategy.

## 2. Introduction

specification.hyper is a specification for hyper. hyper is a framework for building microservices implementations.

specification.hyper defines how microservices can define, register, and implement resources within a hyper system to participate in its function.

hyper's goal, is to present a structure that allows implementing [hypermedia systems](hypermedia.systems) in an optionally (as in you have a choice) microservices setting.

#### 2.1 dependencies

hyper architecture relies on the following standards and specifications:

- [json-schema](https://json-schema.org/)
- [OpenAPI](https://swagger.io/specification/)
- [JSON:API](https://jsonapi.org/)
- [AsyncAPI](https://www.asyncapi.com/docs/reference/specification/v3.0.0)

## 3. Conventions

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “NOT RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) [[RFC2119](https://tools.ietf.org/html/rfc2119)] [[RFC8174](https://tools.ietf.org/html/rfc8174)] when, and only when, they appear in all capitals, as shown here.

## 4. Services

a hyper service is a logical packaging of resources.

#### 4.1 Services Implementation

This section outlines the key steps involved in implementing a hyper service:

1. Resource Definition: a service **MUST** define its resources according to the resource definition schema.
2. optionally, Configuration Definition: a service MAY register configuration definitions.
3. Resource Discovery: resource definitions **MUST** be registered for discovery. This allows other services and hyper components in the system (e.g. interface.hyper, authorization.hyper, etc..) to locate and interact with your service's resources.
4. Resource Implementation: The service implementation **MUST** comply with its provided resource definitions by handling applicable operation requests for each resource definition. An OpenAPI definition is provided for each possible operation request.

#### 4.2 Service Configuration

A service **MAY** expose updateable configurations, in a key-value setting, by registering configuration definitions according to the configuration json schema dialect.

```json
{
    "$schema": "http://schemas.hyper.mathematikoi.co/configuration-v1.0.0.json"
    ...
}
```

etc.service.hyper provides the `configuration` resource, where each `configuration` resource object correspond to a configuration definition registered by a service on the system, thus manipulating and fetching configuration values can be done through the same uniform interface provided by interface.hyper.

## 5. Resources

Resources represent entities within the hyper system. Each resource models the complete lifecycle of an entity using the CRUD model:

- Create: _it didn’t exist before, and now it does_.
- Read: _it can be observed_.
- Update: _it can be changed_.
- Delete: _it can be destroyed_.

#### 5.1 Resource Definition

a resource definition is a json schema using the [resource json schema dialect](http://schemas.hyper.mathematikoi.co/resource-v1.0.0.json).

```json
{
    "$schema": "http://schemas.hyper.mathematikoi.co/resource-v1.0.0.json"
    ...
}
```

> definition schemas (json schemas and OpenAPI definitions) are maintained in the [schemas.hyper](https://github.com/dohyper/schemas.hyper) repository.

a resource definition specifies the following:

- name: name of the resource.
- applicability: which operations are applicable on the resource.
- properties: resource property fields.
- relations: resource relation fields.
- required: required fields.

#### 5.2 Resource Applicability

describes what kind of operation does the defining service support for the defined resource, and any other features.

```json
{
    "$id": "http://schemas.hyper.mathematikoi.co/resource-v1.0.0.json",
...
    "$defs": {
        "applicability": {
            "type": "object",
            "properties": {
                "create": {
                    "$ref": "#/$defs/operation"
                },
                "read": {
                    "type": "boolean"
                },
                "update": {
                    "$ref": "#/$defs/operation"
                },
                "delete": {
                    "$ref": "#/$defs/operation"
                }
            },
            "unevaluatedProperties": false
        },
        "operation": {
            "anyOf": [
                {
                    "type": "boolean"
                },
                {
                    "type": "object",
                    "properties": {
                        "transactional": {
                            "type": "boolean"
                        }
                    },
                    "required": [
                        "transactional"
                    ],
                    "unevaluatedProperties": false
                }
            ]
        }
    }
...
```

#### 5.3 Resource Properties

A resource **MAY** have any number of properties.

```json
{
    "$id": "http://schemas.hyper.mathematikoi.co/resource-v1.0.0.json",
...
    "$defs": {
        "properties": {
            "type": "object",
            "propertyNames": {
                "pattern": "^[a-zA-Z0-9_]+$",
                "$ref": "https://json-schema.org/draft/2020-12/schema"
            },
            "minProperties": 1
        }
    }
...
```

#### 5.4 Resource Relations

A resource **MAY** have any number of relations to any other resource.

```json
{
    "$id": "http://schemas.hyper.mathematikoi.co/resource-v1.0.0.json",
...
    "$defs": {
         "relations": {
            "type": "object",
            "propertyNames": {
                "pattern": "^[a-zA-Z0-9_]+$",
                "$ref": "#/$defs/relation"
            },
            "minProperties": 1
        },
        "relation": {
            "type": "object",
            "properties": {
                "resource": {
                    "type": "string"
                },
                "cardinality": {
                    "$ref": "#/$defs/cardinality"
                },
                "bidirectionality": {
                    "$ref": "#/$defs/bidirectionality"
                },
                "constraints": {
                    "$ref": "#/$defs/constraints"
                }
            },
            "required": [
                "resource",
                "cardinality"
            ]
        },
        "cardinality": {
            "type": "string",
            "enum": [
                "to-one",
                "to-many"
            ]
        },
        "constraints": {
            "type": "object",
            "properties": {
                "unique": {
                    "type": "boolean"
                }
            },
            "unevaluatedProperties": false
        },
	"operation": {
            "anyOf": [
                {
                    "type": "boolean"
                },
                {
                    "type": "object",
                    "properties": {
                        "transactional": {
                            "type": "boolean"
                        }
                    },
                    "required": [
                        "transactional"
                    ],
                    "unevaluatedProperties": false
                }
            ]
        },
        "projection": {
            "$ref": "#/$defs/relation"
        },
        "bidirectionality": {
            "type": "object",
            "properties": {
                "relation": {
                    "type": "string"
                },
                "projection": {
                    "$ref": "#/$defs/projection"
                }
            },
            "required": [
                "relation"
            ]
        }
    }
...
```

#### 5.5 Resource Controllers

A resource definition MAY specifiy controllers to define operations outside the CRUD model.

```
{
    "$id": "http://schemas.hyper.mathematikoi.co/resource-v1.0.0.json",
...
    "$defs": {
        "controllers": {
            "type": "object",
            "propertyNames": {
                "pattern": "^[a-zA-Z0-9_]+$",
                "$ref": "#/$defs/controller"
            }
        },
        "controller": {
            "type": "object",
            "properties": {
                "method": {
                    "type": "string",
                    "enum": [
                        "GET",
                        "POST",
                        "PUT",
                        "DELETE",
                        "PATCH"
                    ],
                    "default": "POST"
                },
                "type": {
                    "type": "string",
                    "enum": [
                        "collection",
                        "object"
                    ]
                },
                "input": {
                    "$ref": "https://json-schema.org/draft/2020-12/schema"
                },
                "transactional": {
                    "type": "boolean"
                }
            },
            "required": ["type", "input"]
        }
    }
...

```

#### 5.6 Resource Fields

Resource fields are to be defined as described in [JSON:API](https://jsonapi.org/format/1.2/#document-resource-object-fields).

#### 5.7 Resource Operations

This section describes how the resource applicability definition translates into operations for which endpoints need to be implemented.

##### 5.7.1 create resource object operation

a service defining a resource with a truthy `applicability.create` **MUST** implement the `POST /<resource>` endpoint.

###### 5.7.1.1 [definition](http://schemas.hyper.mathematikoi.co/create_resource_object_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/create_resource_object_operation-v1.0.0.json"
}
```

##### 5.7.2 UNDO create resource object operation

a service defining a resource with a truthy `applicability.create.transactional` **MUST** implement the `POST /<resource>/undo` endpoint.

###### 5.7.2.1 [definition](http://schemas.hyper.mathematikoi.co/undo_create_resource_object_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/undo_create_resource_object_operation-v1.0.0.json"
}
```

##### 5.7.3 read resource collection operation

a service defining a resource with a truthy `applicability.read` **MUST** implement the `GET /<resource>` endpoint.

###### 5.7.3.1 [definition](http://schemas.hyper.mathematikoi.co/read_resource_collection_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/read_resource_collection_operation-v1.0.0.json"
}
```

##### 5.7.4 read resource object operation

a service defining a resource with a truthy `applicability.read` **MUST** implement the `GET /<resource>/<id>` endpoint.

###### 5.7.4.1 [definition](http://schemas.hyper.mathematikoi.co/read_resource_object_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/read_resource_object_operation-v1.0.0.json"
}
```

##### 5.7.5 update resource object operation

a service defining a resource with a truthy `applicability.update` **MUST** implement the `PATCH /<resource>/<id>` endpoint.

###### 5.7.5.1 [definition](http://schemas.hyper.mathematikoi.co/update_resource_object_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/update_resource_object_operation-v1.0.0.json"
}
```

##### 5.7.6 UNDO update resource object operation

a service defining a resource with a truthy `applicability.update.transactional` **MUST** implement the `POST /<resource>/<id>/undo` endpoint.

###### 5.7.6.1 [definition](http://schemas.hyper.mathematikoi.co/undo_update_resource_object_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/undo_update_resource_object_operation-v1.0.0.json"
}
```

##### 5.7.7 delete resource object operation

a service defining a resource with a truthy `applicability.delete` **MUST** implement the `DELETE /<resource>/<id>` endpoint.

###### 5.7.7.1 [definition](http://schemas.hyper.mathematikoi.co/delete_resource_object_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/delete_resource_object_operation-v1.0.0.json"
}
```

##### 5.7.8 UNDO delete resource object operation

a service defining a resource with a truthy `applicability.delete.transactional` **MUST** implement the `POST /<resource>/<id>/undo` endpoint.

###### 5.7.8.1 [definition](http://schemas.hyper.mathematikoi.co/undo_delete_resource_object_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/undo_delete_resource_object_operation-v1.0.0.json"
}
```

##### 5.7.9 create relationship operation

a service defining a resource relation with a truthy `applicability.create` **MUST** implement the `POST /<resource>/<id>/relationships/<relations>` endpoint.

###### 5.7.9.1 [definition](http://schemas.hyper.mathematikoi.co/create_relationship_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/create_relationship_operation-v1.0.0.json"
}
```

##### 5.7.10 UNDO create relationship operation

a service defining a resource relation with a truthy `applicability.create.transactional` **MUST** implement the `POST /<resource>/<id>/relationships/<relations>/undo` endpoint.

###### 5.7.10.1 [definition](http://schemas.hyper.mathematikoi.co/undo_create_relationship_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/undo_create_relationship_operation-v1.0.0.json"
}
```

##### 5.7.11 read relationship operation

a service defining a resource relation with a truthy `applicability.read` **MUST** implement the `GET /<resource>/<id>/relationships/<relations>` endpoint.

###### 5.7.11.1 [definition](http://schemas.hyper.mathematikoi.co/read_relationship_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/read_relationship_operation-v1.0.0.json"
}
```

##### 5.7.12 update relationships operation

a service defining a resource relation with a truthy `applicability.update` **MUST** implement the `PATCH /<resource>/<id>/relationships/<relations>` endpoint.

###### 5.7.12.1 [definition](http://schemas.hyper.mathematikoi.co/update_relationship_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/update_relationship_operation-v1.0.0.json"
}
```

##### 5.7.13 UNDO update relationships operation

a service defining a resource relation with a truthy `applicability.update.transactional` **MUST** implement the `POST /<resource>/<id>/relationships/<relations>/undo` endpoint.

###### 5.7.13.1 [definition](http://schemas.hyper.mathematikoi.co/undo_update_relationship_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/undo_update_relationship_operation-v1.0.0.json"
}
```

##### 5.7.14 delete relationships operation

a service defining a resource relation with a truthy `applicability.delete` **MUST** implement the `DELETE /<resource>/<id>/relationships/<relations>` endpoint.

###### 5.7.14.1 [definition](http://schemas.hyper.mathematikoi.co/delete_relationship_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/delete_relationship_operation-v1.0.0.json"
}
```

##### 5.7.15 UNDO delete relationships operation

a service defining a resource relation with a truthy `applicability.delete.transactional` **MUST** implement the `POST /<resource>/<id>/relationships/<relations>/undo` endpoint.

###### 5.7.15.1 [definition](http://schemas.hyper.mathematikoi.co/undo_delete_relationship_operation-v1.0.0.json)

```json
{
  "$ref": "http://schemas.hyper.mathematikoi.co/undo_delete_relationship_operation-v1.0.0.json"
}
```

##### 5.7.16 collection controller operation

a service definition a resource controller of type `collection` **MUST** implement the `<controller.method> /<resource>/<controller.key>`

###### 5.7.16.1 [definition](http://schemas.hyper.mathematikoi.co/collection_controller_operation-v1.0.0.json)

```
{
  "$ref": "http://schemas.hyper.mathematikoi.co/collection_controller_operation-v1.0.0.json"
}
```

##### 5.7.17 UNDO collection controller operation

a service definition a resource controller of type `collection` with a truthy `controller.transactional` **MUST** implement the `<controller.method> /<resource>/<controller.key>/undo`

###### 5.7.17.1 [definition](http://schemas.hyper.mathematikoi.co/undo_collection_controller_operation-v1.0.0.json)

```
{
  "$ref": "http://schemas.hyper.mathematikoi.co/undo_collection_controller_operation-v1.0.0.json"
}
```

##### 5.7.18 object controller operation

a service definition a resource controller of type `collection` **MUST** implement the `<controller.method> /<resource>/<id>/<controller.key>`

###### 5.7.18.1 [definition](http://schemas.hyper.mathematikoi.co/object_controller_operation-v1.0.0.json)

```
{
  "$ref": "http://schemas.hyper.mathematikoi.co/object_controller_operation-v1.0.0.json"
}
```

##### 5.7.19 UNDO object controller operation

a service definition a resource controller of type `object` with a truthy `controller.transactional` **MUST** implement the `<controller.method> /<resource>``/<id>``/<controller.key>/undo`

###### 5.7.19.1 [definition](http://schemas.hyper.mathematikoi.co/undo_object_controller_operation-v1.0.0.json)

```
{
  "$ref": "http://schemas.hyper.mathematikoi.co/undo_object_controller_operation-v1.0.0.json"
}
```

## 6. Appendix

This section links and includes documentation of other hyper components and utilities.

#### 6.1 discovery.hyper

discovery.hyper implements services discovery for a hyper system.

###### 6.1.1 @dohyper/registry.discovery.hyper

`@dohyper/registry.discovery.hyper` is a discovery.hyper client used by services to register definitions.

```javascript
const Discovery = require("@dohyper/registry.discovery.hyper");

const client = new Discovery(process.env.DISCOVERY_HYPER_URL, {
  password: process.env.DISCOVERY_HYPER_PASSWORD,
});

const definitions = [];
const configurations = [];

client
  .register(NAME, { url: process.env.URL, definitions, configurations })
  .catch((error) => {
    console.log(error);
  });
```

#### 6.2 interface.hyper

interface.hyper acts as an api gateway and provides a JSON:API interface to all the resources in a hyper system. interface.hyper offloads the services from dealing with JSON:API specification.
interface.hyper does the following:

- authentication and authorization enforcement.
- requests validation.
- responses serialization.

#### 6.3 identity.hyper

identity.hyper provides identity and access management capabilities to the system.

#### 6.4 authorization.hyper

authorization.hyper watches the resources on the system and creates the necessary authorization configuration.

#### 6.5 compliance.hyper

compliance.hyper is a test suite for hyper systems.

#### 6.6 cli.hyper

cli.hyper is a command-line tool to manage hyper projects, services, and applications.

```
$> npm i -g @dohyper/cli.hyper
```

##### 6.6.1 create a hyper project

```
$> hyper project create -h
```

##### 6.6.2 create a hyper service

```
$> hyper service create -h
```

##### 6.6.3 create a hyper application

```
$> hyper application create -h
```

#### 6.7 templates

##### 6.7.1 hyper project template

[link](https://github.com/dohyper/template.project.hyper)

##### 6.7.2 hyper service template

[link](https://github.com/dohyper/template.service.hyper)

##### 6.7.3 hyper application template

[link](https://github.com/dohyper/template.application.hyper)

#### 6.8 application.hyper

application.hyper is a standalone application that can consume a hyper project interface. application.hyper requires the `presentation.service.hyper`.

#### 6.9 resources.service.hyper

resources.service.hyper provides the `resource` resource.

#### 6.10 governance.service.hyper

governance.service.hyper provides the `identity`, `permission`, and `group` resources.

#### 6.11 transactions.service.hyper

transactions.service.hyper provides the `transaction` resource.

#### 6.12 etc.service.hyper

etc.service.hyper provides the `configuration` resource.

#### 6.13 attributes.service.hyper

attributes.service.hyper provides the `attribute` resource.

#### 6.15 utility hyper services

##### 6.15.1 intelligence.service.hyper

intelligence.service.hyper implements business intelligence capabilities.

##### 6.15.2 automations.service.hyper

automations.service.hyper implements workflow automation capabilities.

##### 6.15.3 objects.service.hyper

objects.service.hyper provides the `object` resource.

##### 6.15.4 mail.service.hyper

mail.service.hyper implements mail capabilities.

##### 6.15.5 chat.service.hyper

chat.service.hyper implements messaging capabilities.
