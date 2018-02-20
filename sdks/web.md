# Web, Cordova and React Native AeroGear Services SDK

## Motivation

Provide set of the SDK's that can be used forWeb, Cordova and React Native based development.
Provide best performance and user experience on the platform.

## Proposed solution for Web, Cordova and React Native AeroGear Services SDK

Provide JavaScript implementations (JavaScript SDK). This means that specific browser functionalities that may not be available for React or Cordova will be provided to enable support for these platforms. JS SDK will be working with Web, Cordova and React Native - each platform will provide wrappers for specific APIs if needed. This approach will not allow us to reuse native implementations, but it will make Cordova, Web and ReactNative SDK's more consistent. Implementation could detect platform and load specific ReactNative/Cordova/Browser functions written in javascript or call native plugins.

## Advantages

- Ability to reuse code for Web, Cordova and ReactNative and work in the single repository
- Availablity of pure JS implementations for keycloak, push, sync.js etc.
- No need to adjust native code for the plugins
- No challenges related with cross js/native communication - fetchnning native stateful data.

## Architecture

### Config parsing 

Cordova config plugin not needed due to trivial implementation that can be done in pure JavaScript.
Networking/Logging should be done on JavaScript side as this is common aproach for cordova.
Due to trivial implementation no need for core module like in Android and IOS.

### Individual SDK's

Using existing cordova plugins or creating new cordova plugins for services.
Following example is using auth module, but behaviour should be the same for the most of the plugins.
Example:

```
var configString = require('mobile-config.json')
var core = require('@aerogearservices/core')
var auth = require('@aerogearservices/auth')

var keycloakConfig = core.config(configString).getKeyCloakConfig()

auth.init(keycloak)

// Using 
auth.login()
// Allow to retrieve token in javascript.
auth.getToken()
```

## Investigation process

Source code: https://github.com/aerogear/aerogear-web-sdk

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
