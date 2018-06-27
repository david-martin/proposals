# **Android SDK E2E Testing Infrastructure (draft)**

## Executing E2E tests

We need environment where E2E tests can be run automatically requiring no manual interaction. This kind of test doesn’t need to be executed on every PR, but it should be possible to execute it on demand fully without any further manual configuration.

## CI solutions for running tests

### CircleCI

3rd party solution that's already used in most Aerogear projects. With version 2.0 it allows to run custom containers. It also offers possibilty to run tests on device, it's provided by them on their backend. Free plan allows us to run one build/test per organization.  
Easy to set up, but there might be some unforseen things that we can't solve because it's running on 3rd party infrastructure. 

### Jenkins Wendy

Our solution that runs fully on open-source. It builds projects using our Aerogear Digger soulution. Jenkins plugins also allows us to spawn custom containers running in OpenShift. This can allow us to deploy whole Aerogear solution for advanced integration testing. One problem remaining to solve is how to run on device tests. Some of the possibilities were explored (below).

## Explored possibilities

Both Android emulator and on physical device are considered. Most tests would require only emulator, although with Google services included. 

### Running tests on Android Emulator in Kubernetes/OpenShift

This could be promising solution, because it's based on technology many are experienced with. However QEMU based emulator requires access to hardware acceleration with VT-X CPU extensions and GPU acceleration, this is available usally only on non-virtualized machines, such as real physical hardware. Performance is extremely bad without these. Support of VT-X in OpenShift requires access to `/dev/kvm` device, that's available only in privileged container. This container with Android Emulator/Studio was tried both in pure docker and OpenShift [1]. Support for running it in unprivileged container and with access to GPU is probably being developed [2] and may be accessible in future. So it's not very plausible option at the moment. 

### Running tests on Android Emulator/connected phone on common PC

We can configure some computer to serve as slave to Jenkins and have connected devices to it. This option was already used to run some of integration tests so it's proven. We would need this also for iOS tests on device. It would be little bit complicated to upscale for many physical devices.

### Running tests on AWS Device farm

AWS Device farm is 3rd party service with multiple physical devices connected and accessible in cloud. It would be ideal for testing compatibility across many devices. This solution was explored in past and was declined because of quite high costs.

## Proposal

Currently we can do simple E2E tests using infrastucture based on Circle CI, against already deployed services (like Keycloak). On device tests will run directly on CircleCI infrasturcture.

It would be better for advanced integration testing to use our own solution using Jenkins Wendy. We can deploy whole infrastructure in OpenShift and test against it.
For on device tests we should then use physical machine (connected as Jenkins slave) running emulator and having other physical mobile devices connected to it.

## **References**

[1] [https://github.com/ljishen/docker-vnc-android-studio](https://github.com/ljishen/docker-vnc-android-studio)

[2] [https://github.com/kubernetes/kubernetes/issues/5607](https://github.com/kubernetes/kubernetes/issues/5607)