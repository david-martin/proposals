# Sync Admin UI

The Admin UI will be written in React, and use Patternfly for the look & feel.
See https://github.com/patternfly/patternfly-react & https://github.com/patternfly/patternfly-react-demo-app for more info.

## Authentication

As the server will be running on OpenShift, a ServiceAccount Oauth Client will be used to leverage OpenShift as an auth provider.
This follows the pattern in the [Auth proposal here](../auth/developer-single-sign-on-across-mobile-services.md).
Any OpenShift user who has 'edit' access to the the `Deployment` object for the Data Sync server will be able to get access to the Data Sync Admin UI.

## Architecture

The Admin UI will be located in the same repository as the Server. It will use React with JSX and the JSX will be compiled using link:https://reactjs.org/docs/add-react-to-a-website.html#add-jsx-to-a-project[babel]. We want to
avoid the use of heavyweight build tools (Webpack, Grunt, Gulp) as much as possible and rely on npm scripts. For routing the most popular option (for web projects) is link:https://github.com/ReactTraining/react-router[react-router]. We can investigate others but i would suggest to use this library.
As the Admin UI manages quite a lot of state i would also suggest to use link:https://redux.js.org[redux] as a central state management tool.

## Screens

There are 4 main screens in the initial Admin UI:

* Data Schema
* Data Sources
* Resolver Mappings
* Schema Playground

### Data Schema

The link:https://redhat.invisionapp.com/share/4RLVQKKQ2VS#/screens/305421181[Schema Editor] will be used to define the types, mutations and subscriptions of the user's data structure. It will be split in two sections:

. left side: a text editor where the user can edit the source of the schema
. right side: an graphical (tree) representation of the schema. This will be used to explore the schema and also add a resolver to a type

### Data Sources

x

### Resolver Mappings

x

### Schema Playground

x