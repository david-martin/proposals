# Sync Admin UI

The Admin UI will be written in React, and use Patternfly for the look & feel.
See https://github.com/patternfly/patternfly-react & https://github.com/patternfly/patternfly-react-demo-app for more info.

## Authentication

As the server will be running on OpenShift, a ServiceAccount Oauth Client will be used to leverage OpenShift as an auth provider.
This follows the pattern in the [Auth proposal here](../auth/developer-single-sign-on-across-mobile-services.md).
Any OpenShift user who has 'edit' access to the the `Deployment` object for the Data Sync server will be able to get access to the Data Sync Admin UI.

## Screens

There are 4 main screens in the initial Admin UI:

* Data Schema
* Data Sources
* Resolver Mappings
* Schema Playground

### Data Schema

x

### Data Sources

x

### Resolver Mappings

x

### Schema Playground

x