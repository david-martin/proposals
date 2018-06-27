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

*TODO*: why a resolver per property (see screenshot)? Should it not be one resolver per type?
*TODO*: is the left side editor interactive (should i be able to add/delete nodes in there)?

The [Schema Editor](https://redhat.invisionapp.com/share/4RLVQKKQ2VS#/screens/305421181) will be used to define the types, mutations and subscriptions of the user's data structure. It will be split in two sections:

. left side: a text editor where the user can edit the source of the schema
. right side: an graphical (tree) representation of the schema. This will be used to explore the schema and also add a resolver to a type

### Data Sources

This [view](https://redhat.invisionapp.com/share/4RLVQKKQ2VS#/screens/305423609) allows the user to view and manage data sources. Many data sources can be used in a single schema. There will be a button called `Add Data Source` which opens a [Pop-up](https://redhat.invisionapp.com/share/4RLVQKKQ2VS#/screens/305424878) where the user can specify the type and credentials of the Data source.
Editing and deleting Data sources should also be possible. 

### Resolver Mappings

*TODO*: Screen for resolver mappings list missing?
*TODO*: Why do we need response mappings? Doesn't GraphQl let us define how the response exactly looks like?

This section lets the user define queries for data sources and map them to resolvers.

### Resolver mappings

This view allows the user to create and manage request and response mapping templates. Those templates are always connected to a data source. Request templates are queries (e.g. SQL queries) that retrieve data. Response queries can be used to transform the retrieved data (e.g. string to JSON).

### Resolver editor

A Pop-up shown when the user clicks on `Add Resolver` in the `Data Schema` screen. It is used to combine a data source with queries to retrieve the data and transform the result. The queries are selected from pre-filled dropdowns. For example, there could be a resolver for the `User` type that combines the `Postgres` data source with a query `select {{userId}} from users;`.
The view should have a dropdown to select the data source and to other dropdowns to select:

. the request mapping template. Select a query to retrieve data. The source of the query will be shown below.

. the response mapping template. Select a transformation to convert the retrieved data back into a form that the user is interested in. 

### Schema Playground

*TODO*: probably best to use graphiql and wrap it inside react.

A screen where the user can run queries and inspect the results. We should use the default [Graphiql](https://github.com/graphql/graphiql) view if possible.
