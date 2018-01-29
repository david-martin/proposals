# Standarize mobile configuration format

## Introduction

Introduce standard for system interoperability, by providing schemas that will be used to communicate between mobile sdk's and core services.

## Problem Description

Mobile configuration format is not standarized. It contains various fields that have no meaning for the mobile users.
Configuration also exposes some internal fields. 

```json
{
 "config": {
          "headers": {},
          "mysql_db": "unifiedpush",
          "mysql_password": "unifiedpush",
          "mysql_username": "unifiedpush",
          "name": "unified-push-server",
          "type": "unified-push-server",
          "uri": "http://ups-myproject.192.168.37.1.nip.io"
        },
        "name": "unified-push-server"
}
```

Most important parameters often have different names. For example some services contain `url` property and some `uri`.
Service specific `config` prevents us from strict json parsing.

## Expectations

- Provide mobile configuration that has only esential fields for mobile users
- Standarize format so users to not break functionalities once users will start using configuration
 
## Proposed Solution

Provide standard for mobile configuration that will be extendable and will be easy to parse in mobile clients.

### Contract between SDK and AeroGear Services Backend

AeroGear services cli should have a clearly defined format for interaction with the SDK
To do that efficiently we suggest to use JSONSchema format with versioning.

Each backend service will provide set of labels and fields that will be used to configure service.
Labels will conform to schema that can also used as documentation for services. Backend developers can iterate on schema by changing schema versions and notifying SDK developers to adjust SDK to conform with the new format.


### Schema overview

`mobile-service.json` file will aggregate all service definitions.
Each service will have their own specific data (schema) that will be used directly in service SDK.
This schema will be supplied in additional `config` parameter.

### Initial top level json schema

#### Top level object

```json
{
 "version" : "number",
 "clusterName" : "string",
 "namespace": "string",
 "services": { "type": "array", "items":"Service"}
}
```

### Service object

```json
{
  "id": "string",
  "name": "string",
  "type": "string",
  "url": "string",
  "config": "object"
}
```
