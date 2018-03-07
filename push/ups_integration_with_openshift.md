# Unified Push Server Integration with OpenShift 

## Introduction

This proposal outlines some developer experience issues with provisioning, setting up and configuring the Unified Push Server (UPS) on OpenShift, and integrating it with Apps using the Aerogear SDK.

## Problem Description

Rather than highlighting problems, User Stories will be used to capture improvements and value that can be added that makes for a better overall experience when using UPS on OpenShift

*As a developer, I want UPS running in my OpenShift namespace, ready to go without any additional setup so I can start to use it immediately*

The Unified Push Server is made up of a Java application & a database (MySQL).
It also has a dependecy on an auth provider (currently Keycloak) for users of the UPS Administration UI or REST API.
There is a proof of concept UPS Ansible Playbook Bundle (APB) available.
However, the APB just provisions UPS and MySQL currently.
It expects an existing Keycloak instance to be running somewhere with the appropriate Realm, Client & Roles already setup.
This can be improved to eliminate any additional setup or prerequisites required from the developer.

*As a developer, I want a consistent login experience across all Mobile Services so that I can easily login with the same credentials to administer my services*

As detailed in [Sign On Proposal](../auth/developer-single-sign-on-across-mobile-services.md), a goal with all Mobile Services is to have a consistent login experience by logging in using OpenShift credentials.
This currently isn't the case as a separate User in Keycloak is used.

*As a developer, I want a simplified setup of Push variants in UPS so that I don't have to manually define them and link them back to Mobile Clients in OpenShift*

Currently there is a switch in context and terms between creating Mobile Clients in OpenShift, and setting up Push Applications & Variants in UPS.
The developer creates a 'Mobile Client' in OpenShift as a logical construct to represent the App they are developing outside of OpenShift.
In UPS, the developer creates a 'Push Application' and 1 or more 'Variants' to represent the App they are developing outside of OpenShift.
There is duplication of logical constructs and potential confusion between what Mobile Client is and what a Push Application is.

The developer also has to somehow get the Push configuration from UPS and place it into their App for consumption by the SDK. With the mobile-services.json approach being used by the Mobile CLI/UI and the SDKs for configuration, there's some things that can be done to simplify what the developer has to do here.

*As a developer, I want to write custom logic for sending Push messages so that I can dynamically send messages based on varying factors and inputs*

Push messages can be sent by calling the [sender API](https://aerogear.org/docs/specs/aerogear-unifiedpush-rest/index.html#397083935) with the correct key. There are features available in the OpenShift and the Catalog that can simplify the provisoining of a custom service, and obtaining secret credentials (like a key) from a provisioned service.

## OAuth provider support 

To address the first & second problem, UPS will be modified to support different auth providers.
The 2 providers will be Keycloak (to maintain backwards compatibility for existing users) and a generic OAuth Provider.
The OAuth flow will follow the same pattern & flow used by the [OpenShift OAuth Proxy](https://github.com/openshift/oauth-proxy).
The UPS APB will need to be modified to include an OAuth ServiceAccount, similar to how [Jenkins uses a ServiceAccount](https://github.com/openshift/jenkins-openshift-login-plugin#browser-access).
UPS will use this ServiceAccount as an OAuth Client to verify the User has the necessary permissions in OpenShift to use UPS.
The permissions will be:
* edit access on the namespace in OpenShift => Standard Developer Account in UPS
* admin access on the namespace in OpenShift => Admin Account in UPS

These changes will mean UPS will no longer have a dependency on Keycloak when running in OpenShift.
Minimal changes will be required to the UPS APB.
Also, the developer doesn't have to know about or be concerned with where or how the auth provider is setup and configured for UPS.

## OpenShift Integration

To address the third problem, a sidecar will be added to the UPS APB.
This sidecar will provide background bi-directional syncing of resources and metadata between OpenShift and UPS.
The sidecar will be written in Go and utilise a kubernetes client for watching & updating Kubernetes resources.
It will use a SerivceAccount for auth when watching resources.
The sidecar will also use a ServiceAccount to access the UPS Admin REST API for reading & updating UPS resources.

### Syncing

Syncing will happen *from* OpenShift *to* UPS for the following resources:

* MobileClient => Variant
* Secret (p12 or secret key) => Variant configuration

The Secrets will have an appropriate label so it can be easily filtered when looking for resources to sync.
The sidecar will overwrite the Variant fields and configuration to match what the values are in OpenShift.
This means that any changes to a Variant directly in the UPS UI will be overwritten on next sync.

Syncing will happen *from* UPS *to* OpenShift for the following resources:

* Variant push configuration (JSON) => MobileClient annotation

In UPS, a JSON object with Push configuration for a Variant is available.
However, this JSON object isn't part of the generated mobile-services.json file from the Mobile CLI or UI.
This JSON config will be read from UPS and synced back as an annotation on the associated MobileClient in OpenShift.
Any manual changes this annotation will be overwritten by the sidecar when it next syncs.

```json
{
  // TODO: example mobile-services.json structure with a single variant
}
```

To faciliate this syncing, a single Push Application will be created in UPS for all Mobile Clients in that OpenShift namespace.
This Push Application will be created on provision (in the APB).

### Multi-variant Mobile Clients

A single Mobile Client may not always map to a single Variant in UPS.
For example, a Cordova Mobile Client could have many Variants e.g. Android, iOS, Windows Phone.
Similarly with a React Native Mobile Client, there may be both an Android & iOS Variant.

One solution to this is to create and sync all supported Variants for these kind of Mobile Clients.
For example, if a Cordova Mobile Client is created, it will create both an Android & iOS variant, and keep them synced.

This presents another problem when generating the mobile-services.json file from the Mobile CLI or UI.
A solution to this is to include all variant configs for the Cordova app in the mobile-services.json file.
The SDK would then need to pick out the appropriate variant config from the mobile-services config, depending on the platform, and configure the Cordova SDK.

```json
{
  // TODO: example mobile-services.json structure with multiple variants
}
```

## Binding to UPS for server-side Integration

To address the fourth problem, an APB 'bind' task will be used.
The 'bind' task will create an admin API key in UPS, and pass it back to the catalog to be stored as a secret.
This secret can then be mounted into any custom service running in OpenShift.
This is a standard flow for a 'bind' task when writing APBs.


## Potential future work

Building on the UPS and OpenShfit Integrations in this proposal, there is potential for future work in the Origin Web Console to help with setting up Push variants.
This could include screens for uploading a p12 cert or secret key, which automatically get synced to UPS.
These screens could provide a more cohesive view of a Mobile Client between whats in OpenShift and what's in UPS.
This is very similar to what was done from the [Build Farm proof of concept](https://www.redhat.com/archives/feedhenry-dev/2017-October/msg00069.html) to provide hooks into Jenkins and tying it all back to a single Mobile Client representation.