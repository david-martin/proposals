# End-to-end Aerogear Services SDK test infrastructure

## Introduction

This is proposal of setting up testing infrastructure for running E2E tests for Android and iOS. The means of software and hardware required to do so are also described here.

## Problem description

We need environment where E2E tests can be run automatically requiring no manual interaction. This kind of test doesn’t need to be executed on every PR, but it should be possible to execute it on demand fully without any further manual configuration.

## Expectations

* We need a place where we can execute the tests with real running OS on mobile device.
* We need the tests to be executed on demand or periodically in some kind of CI environment.
* We need a place where we can run backend mobile services that are tested.

## Terms

* *Appium* - [Appium](http://appium.io/) is an open source test automation framework for use with native, hybrid and mobile web apps.
* *Jenkins slave* - computer or virtual machine that is controlled by [Jenkins](https://jenkins.io/) server and some tasks are executed on it

## Proposal

As Jenkins CI we will utilize existing [Jenkins Wendy](http://Jenkins-wendy.ci.feedhenry.org).

We need to hook up macOS machine to the network with internet access and connect it as Jenkins slave to a Jenkins CI. Then connect physical mobile devices to it over USB.
Appium will be running on the slave machine. macOS is used because it allows us to test both Android and iOS. Some other software like [Fastlane](https://fastlane.tools/) will be required to do some tasks like signing iOS applications.

In case that some test require specific backend service to run it should be deployed by Jenkins CI on some OpenShift instance. We won't use Wendy's OpenShift, but set up another machine(s) to run OpenShift on them, so performance and stability won't be impacted by running tests.

### Proposed hardware

* one or more recent macOS machines with High Sierra, at least 16GiB RAM (in case of lower value use more computers or test Android on separate computer)
* iOS 11 capable device
* Android 8.1 capable device (Nexus 5X, Google Pixel, Google Pixel 2)
* some hardware or virtual machine in Cloud for running OpenShift

### Proposed software

* [Jenkins CI](https://jenkins.io/)
* [OpenShift](https://www.openshift.com/) for running backend services
* [Appium server](http://appium.io/)
* [Fastlane](https://fastlane.tools/)

## References

This proposal folows up on current [PR testing proposal](pr-testing-guidelines-and-checklists.md) and also [draft proposal and experimentation](e2e_drafts/Android_SDK_E2E_Test_infrastructure_first_proposal.md) ([AGDROID-704](https://issues.jboss.org/browse/AGDROID-704))
