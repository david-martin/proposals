MCP Android SDK Goals/Metrics

# Cross Cutting Goals

* Releases (mavencentral, jcenter, jitpack.io etc)
    * Release final builds to to Maven Central
    * Continue using [Sonatype](https://oss.sonatype.org) nexus for snapshots

* CI/CD
    * Setup auto publishing of SNAPSHOT builds

* BOM style dependency management 
    * Source a gradle plugin to support BOMs
    * Ex [https://github.com/aerogear/aerogear-parent](https://github.com/aerogear/aerogear-parent)

* Move everything to aerogear github organization
  * What is included in everything?

* Event driven API (either promise style or listeners. NO CALLBACKS! [They are hard to use in Android]
    * [Art-henry](https://github.com/secondsun/art-henry/blob/master/app/src/main/java/org/feedhenry/apps/arthenry/injection/ApplicationModule.java#L119) and [google api client](https://developers.google.com/android/guides/api-client) are examples

* Android/Java neutral if possible
    * IE Can this run in "pure" Java

* Intelligent but overridable integration with Gradle plugin
* Developer tooling

# Per Module Goals

## Core
* Should HTTP be included at this level?
* Default implementations, providers, interfaces, etc for underlying functionality (networking, file access, settings, etc)
* Hide implementation details (RxJava)
* Return useful config objects instead of Map<string, JSONObject>
* Publish to a central repository
* Modules should be able to consume from the "core" module automatically

## HTTP	

* Maybe some kind of Swagger support?
* HTTP should provide connections that are batchable, provide error handling, automatically refresh authz tokens, etc
    * Batching means that there should be a way to "pend" and “deliver” http requests
    * See here event driven API

## Sync

* Consume HTTP from a object provided by core, currently done by hand
* Test that OAuth Token timeouts are automatically refreshed and retried before an error is thrown
* Currently network errors surface as JSON exceptions

## Auth[z] aka OAuth2/OpenID?

* Need ways to inject/manage refresh tokens in http calls (if the access token expires the refresh token needs to get updated and the request retried before returning a failure to the user)
* Automatically consume config from core

## Push

* Create a lightweight "Matcher" that can send events based on criteria of incoming push messages
    * Will require a meeting with UPS team
* Large payload support

## Demo/Cookbook/Template apps
* Port MemeHenry to MCP ([art-henry](https://github.com/secondsun/art-henry) and [art-henry-cloud-app](https://github.com/secondsun/art-henry-cloud-app))
* Port fehbot to MCP ([fehbot-android](https://github.com/secondsun/fehbot-android) and [fehbot_RHMAP_service](https://github.com/secondsun/fehbot_RHMAP_service))
* PRD for IO style app showing off sync, push, openwhisk, etc etc
* MVVM, MVP, etc usage examples

## File Storage

## Gradle plugin

* Dependency checker/validator
* Scan dependencies of libraries and make sure that they do not interfere with the versions in MCP.  Provide configuration to override as appropriate.
* Android Studio integration
* VS Code integration (bonus feature?)

## Implementation ideas

* Build upon Aerogear
* Move Default implementations of services from fh-sync-android to mcp-core-android
