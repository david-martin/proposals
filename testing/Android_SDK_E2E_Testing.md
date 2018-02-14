# **Android SDK E2E Testing Proposal**

[[TOC]]

## Test suite

### Outline of Required Test Suite Features

* The test suite must be easily readable to make it enable those that are not familiar with the test suite to get involved quickly in the maintenance of existing tests and the production of new tests.

* The test suite must be easily maintainable to reduce the time spent on fixing bugs.

* The chosen framework to build the test suite must have the ability to switch between the native application and mobile browser to account for logging into an application using redirect to and from a web form.

### Frameworks Explored

The following frameworks were assessed by creating some proof concept tests against the mobile-security-android-template.

#### Espresso

Espresso is one of the java based frameworks that is favoured and documented on the Android developers website [1].  It provides automatic synchronization of test actions meaning the developer is not required to manage wait times between view transitions.  The framework can be used to test both native applications and hybrid applications and can be written in an easy to read syntax.  An Espresso test recorder is also included in Android studio to help the developer write code quickly.  However, Espresso does not allow any redirects to other apps to happen while testing an apps instead using intents to assert that the correct call was made. 

#### UiAutomator

UiAutomator is another java based framework that is favoured and documented on the Android developers website [2].  This framework is used to build test suites that can perform actions on both user and system apps.  Like the Espresso framework, UiAutomator does not allow for switching between the user application and the browser, again favouring the use of intents to assert that the call was made. 

#### Appium

Appium is a cross platform testing tool that allows for testing of native and hybrid applications on multiple platforms such as iOS and Android.  Appium has a number of clients that can be used to write tests in languages such as NodeJS, Java, Python, Ruby, etc. 

#### WD.js

This client has many positives - there is a large number of locator strategies to target elements in views, the syntax of such tests can be written in a promise chain, and its possible to use mochas transfer promisness to make assertions a natural part of promise chain.  However, despite the positive features, the Wd.js client cannot handle a redirect to the browser at the present time - the problem seems to be that all chromedriver sessions are terminated when attempting to switch context from native to webview which cause the tests to crash. 

#### Webdriverio (nodejs)

Like the WD.js library like the WD.js client, tests using this client can be written in a short chained syntax and can also be used in conjunction with features of tools like mochas transfer promisness to make assertions part of this chained syntax.  Unlike the WD.js client however, webdriverio can handle the redirect to the browser from the application, however there is only two possible locator strategies - xpath and accessibilityId. 

#### Java-Client (Appium Java SDK)

Unlike the javascript libraries the java-client cannot be written in a promise chain syntax, however, there is a large number of locator strategies and it can handle the transition between native application and mobile browser.  Another major feature of the java client is the support for page object classes - using the following annotations, @FindBy(someStrategy), @AndroidFindBy(someStrategy) and @iOSFindBy(someStrategy), allows the creation of a single page object when creating tests for applications on multiple platforms that have the same content and flow but identifiers unique to their platform rather than a separate page object for each [3]. 

### Framework Recommendation

Appium is the framework of choice from those explored as it is the only framework explored that provides clients that can handle the redirect to the browser that is an integral test case we must cover.  Of the two clients, the Java-Client is likely to be the superior choice as it has a greater number of locator strategies and the annotation selector targeting provided can reduce the size of the codebase.

### Test suite Proposal

#### Implementation Language

With the selection of the Appium framework and its Java-Client java, or a JVM based language such as Kotlin, Groovy, Scala, etc. should be used, provided they can support the java based annotations that the appium Java-Client provides for Page Object support.  The use of a JVM language other than Java could increase the readability of the test suite.

#### Page Objects

The test suite should make use of the page object model. This model separates the logic the presentation logic of the associated page objects and the test logic which leads to tests that are less likely to be brittle, and would therefore increase the maintainability of the test suite [4].  As mentioned in the framework recommendation section, the ability of the Appium library to use annotations for targeting selectors that are unique to each platform can help to reduce the codebase size - only one page object is required per platform with the annotations providing the ability to target Android or iOS specific selectors and the tests files themselves can also be generic meaning no separate tests for Android or iOS applications that share the exact same pattern of logic flow through the application.

#### Self-Contained Repository

With the ability of Appium’s page object support able to provide page objects that can target both Android and iOS applications and generic test files, it makes sense that the test suite should be its own repository on github rather than nested with the Android SDK code.  It is also worth mentioning, at present there is no Swift client for appium, therefore it is likely iOS E2E tests will not be included with the iOS SDK code.  The same pattern should be followed for each SDK, therefore a single unified repository makes the most sense from this point of view. 

#### Build Automation

With the project implemented in a JVM language, ideally the project should be Gradle based as this matches the build automation that is used for the Android SDK.  Writing custom task classes for gradle [5] could be explored to ensure that a test, or set of tests can be run against a specified apk (where path to apk can be entered, or a path to a configured folder where an apk should be placed).


## **References**

[1] [https://developer.android.com/training/testing/ui-testing/espresso-testing.html](https://developer.android.com/training/testing/ui-testing/espresso-testing.html)

[2] [https://developer.android.com/training/testing/ui-automator.html](https://developer.android.com/training/testing/ui-automator.html)

[3] [https://github.com/appium/java-client/blob/master/docs/Page-objects.md](https://github.com/appium/java-client/blob/master/docs/Page-objects.md)

[4] [https://martinfowler.com/bliki/PageObject.html](https://martinfowler.com/bliki/PageObject.html)

[5] [https://docs.gradle.org/current/userguide/custom_tasks.html](https://docs.gradle.org/current/userguide/custom_tasks.html)
