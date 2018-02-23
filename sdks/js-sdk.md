# Abstract

This document outlines what is needed to provide a set of the SDK's that can be used for JavaScript platforms, specifically Cordova, React Native & Web.

## Goals

### Primary Goals

* Provide SDKs (Core, Metrics, Auth etc..) for Mobile Developers to allow them to integrat Cordova, React Native & Web applications into Mobile Services.

### Secondary Goals

* Reuse as much JavaScript as possible between the 3 different target platforms
* Reuse Native code from Native SDKs when there is a Native code requirement

## Problem Description

The 3 target platforms (Cordova, React Native & Web) all support JavaScript as the development language. To develop Mobile Apps that integrate into Mobile Services, it is necessary to provide SDKs for Mobile Developers for the target platform & language. These SDKs need to be written & maintained. 

## Proposed solution

Provide JavaScript implementations (via a JavaScript SDK). This means the deault way of implementing an SDK inteface will be in Javascript. However, this may not always be possible. Each platform will provide wrappers for specific APIs if needed. Doing this will make Cordova, Web and React Native SDK's more consistent. Implementation could detect platform and load specific React Native/Cordova/Browser functions written in JavaScript or call native plugins if necessary.

### Advantages

- Ability to reuse code for Cordova, React Native & Web and work in the single repository
- Can reuse pure JS implementations for keycloak, push, sync.js etc.
- No need to adjust native code (from Native SDKs) for plugin use
- No challenges related with cross js/native communication - fetching native stateful data.

## Architecture

### Config parsing 

A Cordova config plugin is not needed due to trivial implementation that can be done in pure JavaScript.
Networking/Logging should be done on JavaScript side as this is a common aproach for Cordova.

### Individual SDK's

Depending on the SDK, a pure javascript implementation may be possible. In cases where it isn't, existing plugins can be used, or new plugins can be created for services.

## Investigation

POC source code: https://github.com/aerogear/aerogear-js-sdk

### POC for Core

Implementation: https://github.com/aerogear/aerogear-js-sdk/tree/master/packages/core

Two options were investigated
1. Wrapping core into plugin. 
1. Writing pure JS helper.

Wrapping core into plugin was proven difficult and innefective:
- JavaScript implementation is trivial
- Dropping configuration into native projects may be bad user experience for Cordova or React Native developers.
- Core Android & IOS providing helpers that are available on cordova/JavaScript
- Only parser needed to be linked back to JavaScript
- The way Android instantiates service will not be compatibile with Cordova 
- Wrapping core into cordova plugin will put some limitations on native implementation we want to avoid.

JavaScript configuration parser/processor was implemented.
This was proven to be the most efficient way to resolve configuration parsing problem.
