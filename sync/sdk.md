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

x

## Pagination

x

## Client Metrics

x

## Authentication

x

## Authorization

x

## Conflict Strategy/Resolution

x