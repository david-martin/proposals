# Android Core SDK + Auth, REST, and Sync Service modules Proposal 
This document outlines the features, architecture choices, tooling needs, and processes for developing the Android SDKs for communicating with services connected by Mobile-Core.  It is hoped that This can also be a model for other platforms as we move forward.

## Goals

### Primary Goal
 Provide a starting point for development of the Android SDKs based on the general architecture described in https://github.com/aerogear/proposals/blob/1b79c6323997a0fa4e9794bb2b91145702cf5a48/sdk/overview.md as well as define for Android what tools we will build (ie build plugins, IDE tooling, scripts, etc), the relationship between the "Core" SDK and "Service" SDKs, release and distribution processes, and processes for ensuring code quality (dependency management, static analysis, CI/CD, etc)

## Terms
### Core SDK
The base library that provides the mechanisms for setting up mobile-core services.  It will include interfaces for Requests, Responses, Services, Modules, and mechanisms for how they become composed.
### Service Module
A Service Module connects to a OpenShift service that provides some function.  Examples of services are sync, keycloak, push, rest with RHOAR runtimes, etc
### Application Module
This is the code which is written by an application developer to bridge our modules and their application. (for example, application specific business logic)

## Problem Description
Mobile core will define services and bindings that represent various aspects of a mobile application.  Examples of these aspects are Metrics, Authentication, and Data Synchronisation and provided by implementations such as Prometheus & Grafana, Keycloak, and fh-sync-server.  The Android core SDK will be responsible for interfacing with mobile core and providing cross cutting functionality (network request building and execution, logging, error handling, etc) while service modules will be responsible for exposing aspect implementaiton specific features and behaviors to the application module as well as consuming deployment details provided by the core SDK.

During the proof of concept phase the Android mcp library and build plugin was written.  The build plugin was responsible for configuring the mcp library to connect to services that were exposed by mcp-standalone.  The application would then configure libraries for services using values that the mcp-library downloaded from mcp-standalone.  The mobile-core SDKs will follow this pattern and expand on it by also automating much of the configuration of service libraries as well as providing service libraries as first class citizens.  In the mcp PoC the developer was responsible for integrating various libraries which may have had different design patterns and goals.

For example, if a KeyCloak and fh-sync-server service were created and bound, the developer was responsible for retrieving tokens from keycloak and configuring fh-sync-server to consume those tokens.  However, fh-sync-server was unaware of tokens beyond the fact that they were to be included in headers of http requests.  If a token expired fh-sync-server would return to the user a synchronization error and the developer would have had to code a system for parsing that error, understanding that it was a token timeout, refreshing the token, and restarting the sync operation.  In the mobile core world the Core SDK would understand that the sync service module connection error was a token error and invoke the keycloak module to resolve it automatically.

## Proposed solution

### Core SDK

The Core SDK will be the package that all service modules inherit from.  It will be responsible for managing network requests, threading, logging, error handling and resolution, loading configuration for mobile-core services, and other cross cutting concerns.  Modules will interface with the core SDK by implementing the necessary interfaces provided and annotating the implementations so that the Core SDK will be able to correctly manage them as well.  The Core SDK will also allow some of its internals to be exposed through interfaces for purposes such as testing, bespoke integrations, and management of additional resources.

The Core SDK will provide a request lifecycle that the Psuedocode section will demonstrate.

### Service Modules

Service modules will each represent a singular service that is supportd by mobile-core.  We will use, where available and practical, existing libraries for services and in those cases the modules will provide the necessary implementation of Core APIs to make them compatable with the Core SDK lifecycles and features.  The first Services we are looking at integrations will be Keycloak, fh-sync-service, and a RESTful API service module.

### Implementation notes

Public APIs should always expose native Java Objects and not stringified JSON or JSON Objects.

### Use cases and psuedo code examples

#### Defining a Service Module
A service interface will be responsible for building requests, parsing responses, handling errors.  Services may depend on each other and the method of service composition is currently under evaluation. However, services will have a resolution order where a request is passed along a chain until it is ready to be executed.
#### Create a Request
A request is a object that is ready to be executed. The request contains in itself any authentication tokens, headers, etc necessary to connect to a service running in a mobile-core environment.
For this example, let’s say the user is writing a photo sharing application.  They will probably in their application have an interface that looks like this : 
```
public interface UploadService {
   public RequestHandle upload(Picture mypicture);
}
```
There will be, in OpenShift, a Mobile application, a sync server, a keycloak server, and a storage server.  An implementation of this interface could load a keycloak token from the device, validate the token, start a new OkHttpRequest object, Upload the file etc etc.  What I am getting at is that this is a very laborious process.  However, let’s annotate this call a bit.
```
@MobileService(requires = {KeycloakService.class, SyncService.class, StorageService.class},
                  appKey=”xyz”)
public interface UploadService {
   @ServiceMethod
   public RequestHandle upload(Picture mypicture);
}
```
In this world, the `@MobileService` annotation would signal to the core SDK that this interface should be implemented by our libraries and it requires a request to include the Keycloak, Sync, and Storage.
The code could then be invoked by the developer as follows : 
```
RequestHandle handle = org.aerogear.android.core.MobileService.from(UploadService.class).upload(myPicture);
```
The ‘handle’ would be how end user code keeps track of the status of the request.  It should be parcelable, non leaking, etc.  RequestHandle will be used to either poll for or register for events from the service as appropriate.


#### Return a response
Responses are’t returned per se.  A handle is returned from a request invocation.  The handle is a wrapper around some serializable ID and the mobile core apis are responsible for providing information about that request.  Eventually (tbd how) a result can be returned from mobilecore for that request handle.

#### Handle an Error
Error handling is defined in the service interface.  Mobile-core will be responsible for trying these error handlers in a sane order (tbd) and retrying a request if approrpriate.
The error handling method on the service is defined as such :
```
 boolean handleError(Request request, Response errorResponse);
 ```
This method is simple.  It will try to resolve the error and return true if it is resolved or false if it is not.  On true mobile core will stop error handling and either retry if appropriate or flag a status in the response.  Activities will see this flag by using the requestHandle.  On false mobilecore will try the next service error handler.  If no services can resolve the error then that is flagged in the requestHandle box.

#### Using Core Threads

The Mobile Core SDK will provide Java Executors that wrap several common blocking tasks.  These executors will be available to any module or part of the application which may need to schedule diskIO work, netowrking work, or work to happen on the main thread.

```
AppExecutors.networkThread.execute(() -> {
  var result = someRequest.call(); //Perform a HTTP operation off the main thread
  AppExecutors.mainThread.execute(()->{updateDisplay(result);});//Update the Android UI on the main thread.
 });
```

### Misc. Bits
The following topics are not required 

#### Mobile-Core build plugin
Android development relies on many features in Gradle.  Most of these features are abstracted from the end user however a mobile-core build plugin can provide a lot of value to developers.  In the mcp-standalone demo it was responsible for allowing multiple flavors (android term for a configuraiton variation) to have their own configuration files, doing compile time checking and validation of configurations, and providing conveneience features such as pinning self signed certificates.  The compile time validation of the configuration files had the side effect of making the core sdk code simpler by not having to include complicated validaiton and error handling for some parts of the configuration.

The build plugin will invoke the mobile-cli to gather facts about the current state of the server and insure that the correct modules are installed and provide useful configuration.

#### BOM support and enforcement
Gradle lacks a lockfile mechanism for enforcing dependency versions.  In Android builds this means that it is very easy for projects and their dependencies to have conflicting library versions.  We will use a BOM (Bill of Materials) plugin to enforce dependency versions among our code and for end users applicaitons.  We are still evaluating plugins to provide this functionality at this time.

#### Android Studio Plugin
In order to compete with Firebase's mindshare we should be able to inspect and configure the core SDK and service modules using gui tools.  This idea will be fleshed out in a future proposal.

#### CI/CD
Ci/Cd will be provided by Circle CI.  In the past we have used other services, but for Android builds we have had the best luck with Circle CI.  We will improve the value of these services by limiting testing in these environments to automated tests which do not require a emulator.  Before any release is made and during local development we will still provide and run tests that require an Android emulator.

