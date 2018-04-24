# Abstract

This document outlines what is needed to provide a set of the SDK's that can be used for JavaScript platforms, specifically Cordova, React Native & Web.

## Goals

### Primary Goals

* Provide SDKs (Core, Metrics, Auth etc..) for Mobile Developers to allow them to integrate Cordova, React Native & Web applications into Mobile Services.

### Secondary Goals

* Reuse as much JavaScript as possible between the 3 different target platforms
* Provide mechanisms for feature-parity with a similar API between all target platforms considering their differences

## Problem Description

The 3 target platforms (Cordova, React Native & Web) all support JavaScript as the development language. To develop Mobile Apps that integrate into Mobile Services, it is necessary to provide SDKs for Mobile Developers for the target platform & language. These SDKs need to be written & maintained. 

## Proposed solution

Provide JavaScript implementations (via a JavaScript SDK). This means the default way of implementing an SDK interface will be in JavaScript. However, this may not always be possible. 
For the Cordova and React Native platforms we will provide a plugin that encapsulates the native code. This plugin conforms to an interface defined in the JS SDK that abstracts all the native platform specific code. Doing this will make Cordova, Web and React Native SDK's more consistent. We will bundle an
implementation of that Interface for the Web Platform in the JS SDK. The plugins for Cordova and React Native will be separate repositories. The JS SDK requires the users to install the plugin for those Platforms and it will either try to auto-detect the platform, or require the user to pass in a `PlatformImpls` module when a service is instantiated.

### Advantages

- Ability to reuse code for Cordova, React Native & Web and work in a single repository
- Can reuse JavaScript implementations for keycloak, push, sync.js etc.
- No need to adjust native code (from Native SDKs) for plugin use. The native wrappers will not depend on the Android or iOS SDKs but provide their own implementations.

## Architecture

### Config parsing 

Despite the built-in `JSON` API to parse the configuration file, the Core SDK will contain a dedicated module to parse the configuration file, this has the following benefits:

- Better documentation of the configuration values via TypeScript interfaces
- Encapsulation of changes in the config file structure
- Possible support for dynamic and asynchronous fetching of future configuration values

### Metrics

As with the other SDKs, Core will contain the implementation for metrics gathering and reporting, providing a JS interface for uploading records which must also be used even by modules that rely heavily on native implementations, such as some of the Self-Defence Security Checks.

The uploading of metrics records utilizes the networking layer provided by Core.

### Individual SDKs

Depending on the SDK, a TypeScript/JavaScript implementation may be possible. In cases where it isn't, existing plugins can be used, or new plugins can be created for services.

## Adding a module to the JS SDK

New modules written JavaScript or Typescript will have the following items:

- A single `index` entry file that organizes and exposes all the exported contents of the module.
- Manually written or compiled files for a Cordova plugin should be present in the `www/` folder by convention
- Native code in the specific platform folder. i.e. `android/`, `ios/`
- A Cordova `plugin.xml` that references the code contents above via `<js-module>` and `<platform>` tags

### Adding native bindings to a plugin for Cordova

Native bindings can be required for some specific features of a 'main' JavaScript module.
These should be exposed as a separate NPM-installable plugin that expose the required features by the 'main' module.

The main module then feature-checks for the existance of the native feature and deal accordingly, allowing for a single codebase to adapt to different
execution environments and for graceful degradation.

The following diagram shows the relationship between native plugins, JavaScript code and client App code:

<!-- TODO: include diagram -->

#### `plugin.xml` example

```xml
<platform name="android">
  <config-file target="res/xml/config.xml" parent="/*">
      <feature name="PlatformVersionModule" >
          <param
              name="android-package"
              value="org.aerogear.mobile.core.PlatformVersionModule"
          />
      </feature>
  </config-file>

  <js-module>
    <clobbers target="aerogear.platformVersion">
  </js-module>

  <config-file target="AndroidManifest.xml" parent="/*">
      <uses-permission android:name="android.permission.INTERNET" />
  </config-file>

  <source-file
      src="src/android/PlatformVersionModule.java"
      target-dir="src/org/aerogear/mobile/core/PlatformVersionModule"
  />
</platform>
```

#### Native plugin usage example

```javascript
// check for plugin presence
if (aerogear.platformVersion) {
    // utilize feature
    const version = ;
    aerogear.platforVersion.get()
        .then(version => metricsService.sendAppMetrics({
            //...
            platforVersion: version
        }));
} else {
    // take appropriate measures
}
```

### Adding native bindings to a plugin for React Native

Native bindings for React Native should also be delivered as a separate NPM package and register their native modules in react-native's `NativeModules` object.
The code in the 'main' JavaScript module should be able to check for the module's presence in order to react accordingly.

Optionally these can include typing declarations or convenience JavaScript to facilitate access, but these should still allow for feature-checking by the main module.

## Investigation

A Proof of Concept was done in the following repository: https://github.com/aerogear/aerogear-js-sdk

### PoC results for the Core Module for the JS SDK

Implementation: https://github.com/aerogear/aerogear-js-sdk/tree/master/packages/core

Two options were investigated
1. Wrapping core into plugin.
1. Writing TypeScript/JavaScript helper.

Wrapping core into a plugin was proven difficult and ineffective:
- JavaScript implementation is trivial
- Dropping configuration into native projects may be bad user experience for Cordova or React Native developers.
- Core Android & IOS providing helpers that are available on Cordova/JavaScript
- Only parser needed to be linked back to JavaScript
- The way Android instantiates service will not be compatibile with Cordova 
- Wrapping Core into Cordova plugin will put some limitations on native implementation we want to avoid.

Wrapping the Android SDK in a React Native library was proven to be difficult
- The Android SDK makes use of Java8 features that are problematic for React Native
- The Android SDK API requirements are higher than what React Native suggests and implements in their boilerplate code

JavaScript configuration parser/processor was implemented.
This was proven to be the most efficient way to resolve configuration parsing problem.
