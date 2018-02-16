# Apache Cordova AeroGear Services

## Motivation

Provide set of the SDK's that can be used for Cordova based development.
Provide best performance and user experience on the platform.

## Proposed solution 1

Pure javascript solution for the mobile configuration and orchiestraction 
that will allow cooperability between JavaScript and Native.
Mixing both existing Native plugins and pure JavaScript based solutions was identified as the most reasonable aproach.
Configuration parser and all AeroGear Services specific business code doesn't require any native implementation and can be reused in web/cordova/react native implementations. Individual SDK will may by default be implemented in pure JavaScript or relay on the cordova plugins. 

## Advantages of the solution 1

- Significantly reduced effort to develop and maintain Cordova Support by reusing available components.
- Aligned with the way cordova works (single plugin to do specific *native* job, integrated with others in JavaScript)
- Ability to reuse existing cordova plugins that perform native tasks (push notifications, sync, auth)
- Ability to wrap existing JavaScript solutions and use them without 
- Ability to select and use only elements that are needed in the app.

See investigation details bellow for more details

## Proposed solution 2

Pure JavaScript implementations (JavaScript SDK) that will abstract from browser support.
JS SDK will be working with Web, Cordova and ReactNative - each platform will provide wrappers for specific apis if needed.
This aproach will not allow us to reuse native implementations, but it will make Cordova, Web and ReactNative SDK's more consistent. Implementation could detect platform 

## Advantages of the solution 2

- Ability to collect and link 3 different platfroms as part of the single repository
- Code still can be reused thanks to keycloak, push, sync js implementations
- No need for specific plugins for React and Cordova (platform specific wrappers can be provided) with single code base
- No need to adjust native code for the plugins
- No challenges related with cross js/native result access.

See investigation details bellow for more details

## Unanswered questions

Questions that needs to be answered before proposal is merged.

### Collecting all plugins into single repository

Since some of the plugins like push are already in separate repository this can be seen as additional effort. 
It may also complicate distribution as community preffers to have 1 plugin per repository and cordova cmd is optimized.
I do not have strong opinions about this one but it feels like leaving things separate and having some aggregator repository with example application that links to all plugins and have parser code will be the best way.

## Architecture for solution 1

### Config parsing 

Cordova config plugin not needed due to trivial implementation that can be done in pure JavaScript.
Networking/Logging should be done on JavaScript side as this is common aproach for cordova.
Due to trivial implementation no need for core module like in Android and IOS.

### Individual SDK's

Using existing cordova plugins or creating new cordova plugins for services.
Following example is using auth module, but behaviour should be the same for the most of the plugins.
Example:

```
// FIXME We need option without using browserify/webpack
var config = require('mobile-config.json')
var ConfigLib = require('aerogear-services')

var keycloakConfig = ConfigLib(config).getKeyCloakConfig()

cordova.auth.init(keycloak)

// Using 
cordova.auth.login()
// Allow to retrieve token in javascript.
cordova.auth.getToken()
```

This code relects how cordova based solutions work in general.

## Investigation process

All investigations and different variations of architecture will be available in following repository:
https://github.com/wtrocki/CordovaSandbox

### POC for Core

Two options were investigated
1. Wrapping core into plugin. 
1. Writing pure JS helper.

Wrapping core into plugin was proven difficult and innefective:
- JavaScript implementation is trivial
- Droping configuration into native projects may be bad user experience for cordova developers.
- Core android IOS providing helpers that are available on cordova/javascript
- Only parser needed to be linked back to javascript
- The way Android instantiates service will not be compatibile with Cordova 
- Wrapping core into cordova plugin will put some limitations on native implementation we want to avoid.

JavaScript configuration parser/processor was implemented.
This was proven to be the most efficient way to resolve configuration parsing problem.

### Interactions between services (binding)

TODO Link github example

### Investigating individual services

### Auth

TODO Document

Source: https://github.com/wtrocki/CordovaSandbox/tree/master/aerogear-auth-plugin

### Push

TODO Document
Evaluating existing plugin: https://github.com/aerogear/aerogear-cordova-push

### Metrics
TODO Document

Suggestion/Spike: metrics done in pure JavaScript/Typescript.
Challenge: How to push from cordova plugins. Can we pull for cordova?

# Possible limitations

Solution will be able to reuse existing cordova plugins like push, 
however to effective manage and build cordova plugins, 
both IOS and Android AeroGear Services SDK should not contain any SDK specific code.
Instead it should just link to the library that can be then also wrapped to the Cordova plugin on different repository.

I identified a couple solutions to extract existing code from SDK, 
but it will be best to apply this aproach across the board, as in most cases code for 
the actuall solution will be comming as dependency to individual module in AeroGear Services SDK.
Cordova will provide JavaScript specific wrapper.

