# Sync SDK

## Platforms

The initial targeted platforms for a Sync SDK are:

* Android
* iOS
* Cordova

Progressive Web App (PWA) may be targetted at a later stage.

## MemeoList App for 'dogfooding'

During development of the Data Sync Server and the Client SDKs, it will be helpful to have Apps with real functionality and use cases.
2 Apps were considered for this:

* Memeolist App (more info at https://github.com/aerogear/proposals/pull/41)
* Cool Stuff Store App

At this time, only the Memeolist App will be used for this 'dogfooding' purpose. However, it may make sense to convert/write other Apps to work with Data Sync. 

## Binary upload

Binary upload in the context of GraphQL means supporting the upload of binary content from an App to a Mutation resolver.
It also covers the retrieval of binary content from the GraphQL API (via a Query) back to an App.

Some potentially useful references:

* https://github.com/jaydenseric/graphql-multipart-request-spec
* https://github.com/jaydenseric/apollo-upload-client
* https://github.com/jaydenseric/apollo-upload-server

## Realtime Updates

Realtime Updates are provided by Subscriptions.
Subscriptions are implemented as resolvers, which determine when a notification should be sent to connected Clients.
For example, a Subscription could be defined for whenever a new 'Comment' is left on a 'Post' i.e. 'commentCreated'.
If a Client is connected to the server, it can subscribe to this Subscription, and get notified whenever a new 'Comment' is created.
The notification event could contain the new 'Comment' and all its fields.

Subscriptions are supported by the Apollo Clients for Android, iOS and Javascript.
The main work around supporting Subscriptions will be on the server.

## Offline Experience

When using the Apollo GraphQL Clients, there are 2 main factors to providing a good offline experience.
The first factor is having a fast response instead of waiting for a potentially long, failing or timing out network call.
This is handled in 2 ways.
For Queries, a local query cache is used.
The fetch policy can be configured so data is retrieved from the cache immediately, while a network request is also made for the latest data.
See https://www.apollographql.com/docs/react/api/react-apollo.html#graphql-config-options-fetchPolicy for more info on fetch policies.
For Mutations, an optimistic response can be written by the developer.
The optimistic response is the most likely scenario if the mutation was to succeed.
This allows the UI to update immediately without having to wait for the network.
See https://www.apollographql.com/docs/react/features/optimistic-ui.html for more info.

The second factor is ensuring consistency across local data while offline, and when eventually online again.
If a mutation was adding a new entry to a table, then it stands to reason a query that were to list from that table should include the new entry too.
When offline, an optimistic response for a mutation can be followed by an update to the appropriate qeury cache.
See https://www.apollographql.com/docs/react/features/optimistic-ui.html#optimistic-advanced for more info on updating the query cache.

## Pagination

x

## Client Metrics

Client metrics will focus on any metrics that can't be inferred on the Server whenever a Client makes a GraphQL request, or is connected to the server via websocket for Subscriptions.
Based on this defintion, the scope for Client metrics extends to:

* any Sync activity on the client that doesn't result in a request to the server e.g. interactions with the cache, optimistic responses
* any attempted GraphQL requests while offline e.g. number of attempts or timeouts
* client perspective on any successful GraphQL requests e.g. client request/response timings

All these metrics will be maintained on the Client as a set of counters.
Whenever the Client makes a GraphQL request to the server, it will include the latest value for all of these counters.
These values will be exposed on the Server to Prometheus, which will allow them to be visualised in Grafana.

The intial implementation of Client metrics will only look at the latest values received from Clients i.e. whenever they eventually make a successful request to the server.
A future improvement to this could look at storing some amount of historical metrics on the Client.
This would allow for a retroactive look at what a Client was doing while offline.
However, there are timestamp considerations with consilidating the difference between client & server timestamps.
There are also considerations about how much metrics to store, and how much is OK to send, particularly in poor network conditions.


## Authentication

x

## Authorization

x

## Conflict Strategy/Resolution

x