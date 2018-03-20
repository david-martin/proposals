# Unified Push Server Integration with OpenShift 

## Introduction

This proposal outlines some developer experience issues with provisioning, setting up and configuring the Unified Push Server (UPS) on OpenShift, and integrating it with Apps using the Aerogear SDK.

## Problem Description

Rather than highlighting problems, User Stories will be used to capture improvements and value that can be added that makes for a better overall experience when using UPS on OpenShift

*As a developer, I want UPS running in my OpenShift namespace, ready to go without any additional setup so I can start to use it immediately*

The Unified Push Server is made up of a Java application & a database (MySQL).
It also has a dependecy on an auth provider (currently Keycloak) for users of the UPS Administration UI or REST API.
There is a proof of concept [UPS Ansible Playbook Bundle](https://github.com/aerogearcatalog/unifiedpush-apb) (APB) available.
However, the APB just provisions UPS and MySQL currently.
It expects an existing Keycloak instance to be running somewhere with the appropriate Realm, Client & Roles already setup.
This can be improved to eliminate any additional setup or prerequisites required from the developer.

*As a developer, I want a consistent login experience across all Mobile Services so that I can easily login with the same credentials to administer my services*

As detailed in [Sign On Proposal](../auth/developer-single-sign-on-across-mobile-services.md), a goal with all Mobile Services is to have a consistent login experience by logging in using OpenShift credentials.
This currently isn't the case as a separate User in Keycloak is used.

*As a developer, I want a to setup Push variants and link them to Mobile Clients in OpenShift so that I can get Push notifications working as fast & easy as possible with my Mobile Clients*

Currently there is a switch in context and terms between creating Mobile Clients in OpenShift, and setting up Push Applications & Variants in UPS.
The developer creates a 'Mobile Client' in OpenShift as a logical construct to represent the App they are developing outside of OpenShift.
In UPS, the developer creates a 'Push Application' and 1 or more 'Variants' to represent the App they are developing outside of OpenShift.
There is duplication of logical constructs and potential confusion between what Mobile Client is and what a Push Application is.

The developer also has to somehow get the Push configuration from UPS and place it into their App for consumption by the SDK. With the mobile-services.json approach being used by the Mobile CLI/UI and the SDKs for configuration, there's some things that can be done to simplify what the developer has to do here.

*As a developer, I want to write custom logic for sending Push messages so that I can dynamically send messages based on varying factors and inputs*

Push messages can be sent by calling the [sender API](https://aerogear.org/docs/specs/aerogear-unifiedpush-rest/index.html#397083935) with the correct key. There are features available in the OpenShift and the Catalog that can simplify the provisoining of a custom service, and obtaining secret credentials (like a key) from a provisioned service.

## OAuth provider support 

To address the first & second stories, UPS will be modified to support different auth providers.
The 2 providers will be Keycloak (to maintain backwards compatibility for existing users) and a generic OAuth Provider.
The OAuth flow will follow the same pattern & flow as [Prometheus](https://github.com/aerogearcatalog/metrics-apb/blob/9b4cb90988f5f3e7a28a84050cf274355cd36498/roles/provision-metrics-apb/tasks/provision-prometheus.yml#L36-L52) with the [OpenShift OAuth Proxy](https://github.com/openshift/oauth-proxy).
The UPS APB will be modified to include an OAuth ServiceAccount and the OAuth Proxy container.
UPS will be modified to allow running in 'no auth' mode i.e. the UI & API will be uprotected and give 'admin' access to the UPS instance running in the user's namespace. The OAuth Proxy will sit in front of UPS providing authentication and autorisation based on namespace permissions.

A future change to this could look at adding support for individual 'developer' users.
This would be required if running UPS as a shared service.
That would most likely require changes to UPS to call OpenShift to check user permissions based on the token header, and only showing that developers Push Applications.

These changes will mean UPS will no longer have a dependency on Keycloak when running in OpenShift.
Also, the developer doesn't have to know about or be concerned with where or how the auth provider is setup and configured for UPS.

## OpenShift UI Integration

To address the third story, a number of UI screens and integration points will be added to the OpenShift UI & the UPS APB.

Push Variant credentials will be stored in OpenShift as a Secret.
The Secret will have:

* a label to identify it as storing Variant credentials
* an annotation to link to a specific Mobile Client `aerogear.org/mobile-client-id`
* an annotation to store the ID of the Variant in UPS `aerogear.org/variant-id`
* an annotation to store the Variant config from UPS `aerogear.org/variant-config-json`
* a generated name in the format `variant-<variant-type>-<timestamp>`

Example Variant Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  annotations:
    aerogear.org/mobile-client-id: 334e1333b5d4-e114-4332-8332-337c4111
    aerogear.org/variant-id: 337c42c5-f444-4762-89ef-334e1684b5d4
    aerogear.org/variant-config-json: >-
      {"variantId":"some-variant-id","variantSecret":"some-secret","senderId":some-sender-id"}
  labels:
    variant: true
  name: variant-android-334e1333b5d4-e114-4332-8332-337c4111-151222332230
  namespace: myproject
type: Opaque
data:
  googleKey: >-
    \c3VwZXJzZWNyZXQK
```

A sidecar will be used to sync these Push Variant credentials from OpenShift to UPS.
The Variant ID and Config will be set by the sidecar after creation of the Variant in UPS.
OpenShift will be the source of truth for Variants linked to Mobile Clients.
The sycning process in the sidecar will need to consider:

* Syncing *From* OpenShift *to* UPS
  * New Secrets mean a New Variant is created
  * Updates to Secrets overwrite the Variant
  * Deleting a Secret will delete the Variant
  * Deleting a Variant directly in UPS will cause it to be recreated
* Syncing *From* UPS *to* OpenShift
  * Variant ID will be annotated on the Secret after creation
  * Variant Config will be annotated on the Secret and kept up to date (UPS is the source of truth for this config)

Here is an example workflow showing the UI touchpoints:

* developer creates a Cordova App (MobileClient) in OpenShift
* developer provisions UPS from the Catalog
* developer sets up the FCM/Android credentials for push for the Cordova App via *new* UI screens in OpenShift
* this creates a Secret to store those credentials
* the UPS sidecar syncs these credentials from the Secret to a new Android Variant in UPS
* the Variant config is retrieved from UPS and stored in an annotation on the Secret
* the generated mobile-services.json file includes the correct Variant config

```json
{
  "version": 1,
  "clusterName": "https://example.com",
  "namespace": "myproject",
  "clientId": "app-cordova",
  "services": [
    {
      "id": "ups",
      "name": "ups",
      "type": "ups",
      "url": "https://ups.example.com",
      "config": {
        "android": {
          "variantId": "some-variant-id",
          "variantSecret": "some-secret",
          "senderId": "some-sender-id"
        }
      }
    }
  ]
}
```

A follow on flow would be:

* developer sets up iOS credentials for push for the same Codova App via *new* UI screens in OpenShift
* these credentials are stored in a new Secret
* on next sync (in the sidecar), an iOS variant is created in UPS
* the Variant config is retrieved from UPS and stored as annotations in the Secret
* the generated mobile-services.json file now includes both the iOS and Android Variants config

```json
{
  "version": 1,
  "clusterName": "https://example.com",
  "namespace": "myproject",
  "clientId": "app-cordova",
  "services": [
    {
      "id": "ups",
      "name": "ups",
      "type": "ups",
      "url": "https://ups.example.com",
      "config": {
        "android": {
          "variantId": "some-variant-id",
          "variantSecret": "some-secret",
          "senderId": "some-sender-id"
        },
        "ios": {
          "variantId": "some-variant-id",
          "variantSecret": "some-secret"
        }
      }
    }
  ]
}
```

For a first iteration of this, it will be possible to create at most 1 of each variant type that is relevant to the Mobile Client.
For example, the following relations are possible:

* an Android Mobile Client with an Android Variant
* a Cordova Mobile Client with just an iOS Variant
* a Cordova Mobile Client with an iOS Variant and Android Variant
* a Xamarin Mobile Client with an iOS Variant and Android Variant

But the following would not be possible:

* a Cordova Mobile client with 2 Android Variants (The developer should modify the variant to include the credentials they want at that point in time)

Future work could include allowing multiple Variants of a type to be created, and switching between which Variant is currently associated with a Mobile Client.

## Binding to UPS for server-side Integration

To address the fourth story, an APB 'bind' task will be used.
The 'bind' task will create an admin API key in UPS, and pass it back to the catalog to be stored as a secret.
This secret can then be mounted into any custom service running in OpenShift.
This is a standard flow for a 'bind' task when writing APBs.
