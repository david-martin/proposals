# End-to-end Aerogear Services SDK test suite

## Introduction

This is proposal that shows how we can write E2E tests for testing Aerogear Services SDKs for Android and iOS. It also recommends test suite architecture and programming language.

## Problem description

Most of testing for Aerogear Services SDKs is currently done in unit-tests only. It's neccessary to have at least some integration testing that is running automatically to check for some regressions.

## Expectations

* The test suite must be easily readable to make it enable those that are not familiar with the test suite to get involved quickly in the maintenance of existing tests and the production of new tests.

* The test suite must be easily maintainable to reduce the time spent on fixing bugs.

* The chosen framework to build the test suite must have the ability to switch between the native application and mobile browser to account for logging into an application using redirect to and from a web form. It definitely needs to be run from UI in real mobile OS environment (real mobile device or emulator)

## Terms

* *Aerogear Services SDKs* - aggregator for SDKs that are mobile focused and designed to work with each other
  * for Android - https://github.com/aerogear/aerogear-android-sdk
  * for iOS - https://github.com/aerogear/aerogear-ios-sdk
* *Appium* - [Appium](http://appium.io/) is an open source test automation framework for use with native, hybrid and mobile web apps.

## Proposal

We will write E2E tests as UI based tests of example applications ([example here](https://github.com/aerogear/aerogear-android-sdk/tree/master/example)). It's the easiest way, but usually there is a big overhead of maintaining the tests when UI changes.

## UI test framework

[Appium](http://appium.io/) is the framework of choice from those explored as it is the only framework explored that provides clients that can handle the redirect to the browser that is an integral test case we must cover. Appium SDK for Java client is likely to be the superior choice as it has a greater number of locator strategies and the annotation selector targeting provided can reduce the size of the codebase.

### Implementation Language

Use [Kotlin](https://kotlinlang.org/) as language for the test suite. It's advanced syntax features allows us to write much more readable code in the tests, but it's still maintaing compatibility with Java without any changes. We can utilize it to create our own DSL (domain-specific language) for the tests. Using Kotlin will also upskill contributors in a safe way and that is important especially in Android development where it quickly replaces Java.

### Page Objects

The test suite should make use of the page object model. This model separates the presentation logic of the associated page objects and the test logic which leads to tests that are less likely to be brittle, and would therefore increase the maintainability of the test suite.
We can utilize page objects to write platform independent tests, because only logic of gathering references to UI views needs to be written separately.
This is implemented in Akow library below.

### Akow library

Even if Appium supports page objects, it's a little bit Java-like. If we want to use Kotlin, it would be better to make some abstraction directly in Kotlin language.
This abstraction would be written in separate [Akow library](https://github.com/aerogear/akow), that will be included in Gradle dependencies of the test suite. This library would be extended by other test contributors to promote this certain pattern of writing UI tests and in future it can be reused for something else than this test suite or by anyone from the community.

### Self-Contained Repository

With use of page objects as platform-independent view and logic holders, it makes sense that the test suite should be its own repository on github rather than nested with the Android SDK code.  It is also worth mentioning, at present there is no Swift client for appium, therefore it is likely iOS E2E tests will not be included with the iOS SDK code. The same pattern should be followed for each SDK, therefore a single unified repository makes the most sense from this point of view.

### Build Automation

With the project implemented in a JVM language, ideally the project should be Gradle based as this matches the build automation that is used for the Android SDK.

### Proposed software and libraries

* [Akow library](https://github.com/aerogear/akow)
* [Appium SDK and server](http://appium.io/)
* [Gradle](https://gradle.org/)
* [Kotlin](https://kotlinlang.org/)

## References

This document follows up on [experimentation and draft proposal](e2e_drafts/Android_SDK_E2E_Test suite_first_proposal.md).
Check out also [proposal for E2E tesing infrastructure](E2E_Test_infrastructure.md) to see where this test suite will be executed.
