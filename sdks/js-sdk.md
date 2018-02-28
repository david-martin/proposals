# Abstract

This document outlines what is needed to provide a set of the SDK's that can be used for JavaScript platforms, specifically Cordova, React Native & Web.

## Goals

### Primary Goals

* Provide SDKs (Core, Metrics, Auth etc..) for Mobile Developers to allow them to integrate Cordova, React Native & Web applications into Mobile Services.

### Secondary Goals

* Reuse as much JavaScript as possible between the 3 different target platforms
* Reuse Native code from Native SDKs when there is a Native code requirement

## Problem Description

The 3 target platforms (Cordova, React Native & Web) all support JavaScript as the development language. To develop Mobile Apps that integrate into Mobile Services, it is necessary to provide SDKs for Mobile Developers for the target platform & language. These SDKs need to be written & maintained. 

## Proposed solution

Provide JavaScript implementations (via a JavaScript SDK). This means the default way of implementing an SDK interface will be in JavaScript. However, this may not always be possible. 
For the Cordova and React Native platforms we will provide a plugin that encapsulates the native code. This plugin conforms to an interface defined in the JS SDK that abstracts all the native platform specific code. Doing this will make Cordova, Web and React Native SDK's more consistent. We will bundle an
implementation of that Interface for the Web Platform in the JS SDK. The plugins for Cordova and React Native will be separate repositories. The JS SDK requires the users to install the plugin for those Platforms and it will either try to auto-detect the platform, or require the user to pass in a `PlatformImpls` module when a service is instantiated.

### Advantages

- Ability to reuse code for Cordova, React Native & Web and work in the single repository
- Can reuse JavaScript implementations for keycloak, push, sync.js etc.
- No need to adjust native code (from Native SDKs) for plugin use. The native wrappers will not depend on the Android or iOS SDKs but provide their own implementations.

## Architecture

### Config parsing 

A Cordova config plugin is not needed due to trivial implementation that can be done in TypeScript/JavaScript.
Networking/Logging should be done on JavaScript side as this is a common aproach for Cordova.

### Individual SDK's

Depending on the SDK, a TypeScript/JavaScript implementation may be possible. In cases where it isn't, existing plugins can be used, or new plugins can be created for services.

## Investigation

POC source code: https://github.com/aerogear/aerogear-js-sdk

### POC for Core

Implementation: https://github.com/aerogear/aerogear-js-sdk/tree/master/packages/core

Two options were investigated
1. Wrapping core into plugin. 
1. Writing TypeScript/JavaScript helper.

Wrapping core into plugin was proven difficult and ineffective:
- JavaScript implementation is trivial
- Dropping configuration into native projects may be bad user experience for Cordova or React Native developers.
- Core Android & IOS providing helpers that are available on cordova/JavaScript
- Only parser needed to be linked back to JavaScript
- The way Android instantiates service will not be compatibile with Cordova 
- Wrapping core into cordova plugin will put some limitations on native implementation we want to avoid.

Wrapping the Android SDK in a React Native library was proven to be difficult
- The Android SDK makes use of Java8 features that are problematic for React Native 
- The Android SDK API requirements are higher than what React Native suggests and implements in their boilerplate code

JavaScript configuration parser/processor was implemented.
This was proven to be the most efficient way to resolve configuration parsing problem.
