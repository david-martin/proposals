# Android Core SDK + Auth, REST, and Sync Service modules Proposal 
This document outlines the features, architecture choices, tooling needs, and processes for developing the Android SDKs for communicating with services connected by Mobile-Core.  It is hoped that This can also be a model for other platforms as we move forward.

## Goals

### Primary Goal
  * Provide a starting point for development of the Android SDKs based on the general architecture described in https://github.com/aerogear/proposals/blob/1b79c6323997a0fa4e9794bb2b91145702cf5a48/sdk/overview.md  
  * Define the relationship between the "Core" SDK and "Service" SDKs
  * Define Release and distribution processes
  * Define processes for dependency management using a Maven BOM
  * Define where documentation will be stored and generated

## Terms
### Core SDK
The base library that provides the mechanisms for setting up mobile-core services.  It will include interfaces for Requests, Responses, Services, Modules, and mechanisms for how they are configured, loaded, and exposed.
### Service Module
A Service Module connects to a OpenShift service that provides some function.  Examples of services are sync, keycloak, push, rest with RHOAR runtimes, etc.  These services will be automatically configured by the CoreSDK based on values provided in mobile-services.json
### Application Module
This is the code which is written by an application developer to bridge our modules and their application. It is explicitly called out so users can define behaviors that depends on Service Modules.

## Problem Description
Mobile core will define services and bindings that represent various aspects of a mobile application.  Examples of these aspects are Metrics, Authentication, and Data Synchronisation and provided by implementations such as Prometheus & Grafana, Keycloak, and fh-sync-server.  The Android core SDK will be responsible for interfacing with mobile core and providing cross cutting functionality (network request building and execution, logging, error handling, etc) while service modules will be responsible for exposing aspect implementaiton specific features and behaviors to the application module as well as consuming deployment details provided by the core SDK.

During the proof of concept phase the Android mcp library and build plugin was written.  The build plugin was responsible for configuring the mcp library to connect to services that were exposed by mcp-standalone.  The application would then configure libraries for services using values that the mcp-library downloaded from mcp-standalone.  The mobile-core SDKs will follow this pattern and expand on it by also automating much of the configuration of service libraries as well as providing service libraries as first class citizens.  In the mcp PoC the developer was responsible for integrating various libraries which may have had different design patterns and goals.

For example, if a KeyCloak and fh-sync-server service were created and bound, the developer was responsible for retrieving tokens from keycloak and configuring fh-sync-server to consume those tokens.  However, fh-sync-server was unaware of tokens beyond the fact that they were to be included in headers of http requests.  If a token expired fh-sync-server would return to the user a synchronization error and the developer would have had to code a system for parsing that error, understanding that it was a token timeout, refreshing the token, and restarting the sync operation.  In the mobile core world the Core SDK would understand that the sync service module connection error was a token error and invoke the keycloak module to resolve it automatically.

## Proposed solution

### Core SDK

The Core SDK will be the package that all service modules inherit from.  It will be responsible for managing network requests, threading, logging, error handling and resolution, loading configuration for mobile-core services, and other cross cutting concerns.  Modules will interface with the core SDK by implementing the necessary interfaces provided and annotating the implementations so that the Core SDK will be able to correctly manage them as well.  The Core SDK will also allow some of its internals to be exposed through interfaces for purposes such as testing, bespoke integrations, and management of additional resources.

The Core SDK will provide a request lifecycle that the pseudocode section will demonstrate.

### Service Modules

Service modules will each represent a singular service that is supportd by mobile-core.  We will use, where available and practical, existing libraries for services and in those cases the modules will provide the necessary implementation of Core APIs to make them compatable with the Core SDK lifecycles and features.  The first Services we are looking at integrations will be Keycloak, fh-sync-service, and a RESTful API service module.

### Implementation notes

Public APIs should always expose native Java Objects and not stringified JSON or JSON Objects.  Our public APIs should not favor one library over another, instead we will abstract away our dependencies behind interfaces so that users may substitute their own and so we can change our implementations without breaking backwards compatability.

### Use cases and psuedocode examples

#### Starting up Mobile Core
MobileCore is the "main" class for the SDK of the Core SDK.  Users instanciate it by placing a client configuration file, `mobile-services.json`, in their assets directory, configuring MobileCore.Builder, and then calling its build method.

```java
MobileCore.Builder builder = new MobileCore.Builder(application);
MobileCore core = builder.build();
```

During the build method, MobileCore will scan the ServiceRegistry and configure them by calling the method `bootstrap` defined in the ServiceModule interface.  MobileCore will pass to bootstrap the core instance and the section of mobile-core.json that corresponds to that service.

#### Registering a Service Module and the ServiceModuleRegistry
ServiceRegistry is a class that modules can register themselves with as well as declare their dependencies.  During build MobileCore will call bootstrap on services in an order that ensures their dependencies are all available during bootstrapping.

Example registration of KeyCloakService
```java
public class KeyCloakService implements ServiceModule {
  static {
    ServiceModuleRegistry.registerServiceModule("keycloak", KeyCloakService.class, "http");
  }
  /*snip*/
}
```

A nonstatic instance of the ServiceRegistry may be provided to a MobileCore.Builder.  In this case MobileCore will not use services that have registered themselves in static blocks.  This is useful for testing and flexibility.

```java
ServiceModuleRegistry registry = new ServiceModuleRegistry();
registry.registerServiceModule("http", MockHttpService.class);
registry.registerServiceModule("keycloak", KeyCloakService.class, "http");

MobileCore core = new MobileCore.Builder(this.getApplicationContext())
        .setRegistryService(registry)
        .setMobileServiceFileName("mobile-core.json")
        .build();

```

#### Writing a Service Module
A service interface will be responsible for building requests, parsing responses, handling errors.  Services may depend on each other and the method of service composition is currently under evaluation. However, services will have a resolution order where a request is passed along a chain until it is ready to be executed.  Mobile Core will provide services for threading, logging, and networking.  ServiceModules will implement the service module class and provide a default no argument constructor.  

Example KeyCloakService
```java
public class KeyCloakService implements ServiceModule {
   /* snip */
   public KeyCloakService() {}

   @Override
    public void bootstrap(MobileCore core, ServiceConfiguration config) {
        this.serverUrl = config.getProperty("auth-server-url");
        this.clientId = config.getProperty("client_id");
        this.audience = config.getProperty("audience");
        this.grantType = config.getProperty("grant_type");
        this.subjectTokenType = config.getProperty("subject_token_type");
        this.requestedTokenType = config.getProperty("requested_token_type");
        this.realm = config.getProperty("realm");
        this.core = core;
    }
}
```


#### Referencing a Service
An instance of the core bassed during bootstrap can be used to reference other services.  MobileCore is responsible for ensuring that ServiceModule.bootstrap is not called until all declared dependencies are bootstrapped.

CustomService.java
```java
public class CustomService implements ServiceModule {

  KeyCloakService keycloakService;
  /*snip*/
  public void bootstrap(MobileCore core, ServiceConfiguration config) {
     this.keycloakService = core.getService("keycloak", KeyCloakService.class);
     this.networkService = core.getService("network", NetworkService.class);
     /*snip*/
  }

  public void performCustomAction() {
    if (keycloakService.isLoggedIn()) {
      Reuqest request = networkService.newRequest();
      request.addSecurityHeaders(keycloakService.getHeaders());//pseudocode method, don't expect this exactly.
      request.setData(this.buildrequestBody());
      Response response = networkService.execute(request);
      response.onError(/*Error handling lambda*/);
      response.onSuccess(/*Success handling lambda*/);
    }
  }

}
```

#### Using Core Threads

The Mobile Core SDK will provide Java Executors that wrap several common blocking tasks.  These executors will be available to any module or part of the application which may need to schedule diskIO work, netowrking work, or work to happen on the main thread.

```java
AppExecutors.networkThread().execute(() -> {
  var result = someRequest.call(); //Perform a HTTP operation off the main thread
  AppExecutors.mainThread().execute(()->{updateDisplay(result);});//Update the Android UI on the main thread.
 });
```

Applications and Services are encouraged to use AppExecutors so that they can inject their own thread pools during testing.  This idea is based on an [example from Google](https://github.com/googlesamples/android-architecture/blob/todo-mvp/todoapp/app/src/main/java/com/example/android/architecture/blueprints/todoapp/util/AppExecutors.java)

#### Using Core Networking
For now, networking will be a service exposed by a http service module.  The definition of this service will be a different proposal.

#### Proposal diagram
![Core Service and Module diagram](./img/diagram.png)



### BOM support and enforcement
Gradle lacks a lockfile mechanism for enforcing dependency versions.  In Android builds this means that it is very easy for projects and their dependencies to have conflicting library versions. We will use a BOM (Bill of Materials) plugin to enforce dependency versions among our code and for end users applications.
We will use the [Dependency Management Plugin](https://github.com/spring-gradle-plugins/dependency-management-plugin) provided by Spring. This plugin can be used in two ways:

1) Defining the dependency versions in the (parent project's) gradle build file
2) Referencing a remote maven BOM

For our use case we are choosing variant 2. The plugin does not allow to inherit from the dependency management configuration, nor does gradle. Thus approach 1 would work for our own SDK but users would not be able to make use of the BOM.
We will instead provide a Maven BOM specifically tailored for the Android SDK located in [Aerogear Parent](https://github.com/aerogear/aerogear-parent). Aerogear Parent will be a dependency of the Android SDK and can also be used by our users and customers.

The BOM itself is a regular `pom.xml` file defining a dependency management section:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            ... (group, artifact and version)
        </dependency>
        ...
    </dependencies>
</dependencyManagement>
```

We then need to apply the BOM to all subprojects (core, sync, keycloak, etc.) in our parent module (build.gradle):

```groovy
plugins {
    id "io.spring.dependency-management" version "1.0.4.RELEASE"
}
...
subprojects {
    apply plugin: 'io.spring.dependency-management'

    dependencyManagement {
        imports {
            mavenBom 'org.jboss.aerogear:aerogear-parent:1.0-SNAPSHOT'
        }
    }
}
```

### CI/CD
Ci/Cd will be provided by Circle CI.  In the past we have used other services, but for Android builds we have had the best luck with Circle CI.  We will improve the value of these services by limiting testing in these environments to automated tests which do not require a emulator.  Before any release is made and during local development we will still provide and run tests that require an Android emulator.

### Repository and Packaging

The Core SDK and service modules will live in a single repository at https://github.com/aerogear/aerogear-android-sdk.  The Core SDK and each service module will be gradle modules within this project.  Service modules will depend on the core SDK.  Modules and the SDK will be published individually as Maven packages.

### Documentation

Public methods and properties will be documented using JavaDoc.  At release JavaDocs will be bundled with the maven artifacts and also published on aerogear.org.

### Future Work


#### Mobile-Core build plugin
Android development relies on many features in Gradle.  Most of these features are abstracted from the end user however a mobile-core build plugin can provide a lot of value to developers.  In the mcp-standalone demo it was responsible for allowing multiple flavors (android term for a configuraiton variation) to have their own configuration files, doing compile time checking and validation of configurations, and providing conveneience features such as pinning self signed certificates.  The compile time validation of the configuration files had the side effect of making the core sdk code simpler by not having to include complicated validaiton and error handling for some parts of the configuration.

The build plugin will invoke the mobile-cli to gather facts about the current state of the server and insure that the correct modules are installed and provide useful configuration.


#### Android Studio Plugin
In order to compete with Firebase's mindshare we should be able to inspect and configure the core SDK and service modules using gui tools.  This idea will be fleshed out in a future proposal.

