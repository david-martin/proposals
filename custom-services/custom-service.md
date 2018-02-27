# Abstract

This document outlines a proposal to allow the use of custom services as part of your overall mobile solution allowing them to be consumed via a mobile client application.


# Terminology

- **custom service:** This refers to a running service (API Server) that is accessible via a hostname and provides custom business logic.
- **mobile client application:** An instance of an application running on a mobile device.
- **mobile enabled:** A service that has the required objects (configmap and secret) created along with the label "mobile:enabled".

# Problem

Currently there is no simple way for a mobile application developer using OpenShift or Kubernetes to expose the configuration details required to access a custom service running in the cluster to a mobile client application. 
It would require knowledge of configmaps, secrets and our labeling schema to achieve this.
While it would be something a developer familiar with OpenShift/K8s could do, we can provide a better user experience.
Currently we allow configuration to be exposed for the mobile enabled services such as Keycloak that we have created specific APBs for, but most real world mobile client applications require some level of custom business logic. It is important that we allow this business logic to be exposed by the mobile developer to their mobile client applications in a consistent intuitive and simple fashion. If possible once the custom service has been created it should be viewed in the UI as just another mobile service.

## Use cases we hope to solve
- As a mobile developer, I want to expose a service I have already provisioned from the catalog to my mobile clients, so that I can make use of this service in a consistent and simple fashion.

- As a mobile developer, there is a service with custom logic running somewhere that I need to make use of from within my client, I want to easily make this service accessible to my client devices with the needed configuration so that I can consume the custom business logic within my mobile client application.

# Proposed Solution

### Add enable mobile button / link
To allow a developer to expose a service already provisioned from the service catalog, we could modify the UI view in OpenShift to have a call
to action (something like) "enable mobile" that would show up on the overview screen inside the ServiceInstance box if mobile was enabled within the cluster and there was at least one mobile client in the namespace. When clicked they would be presented with a modal form that would allow them to specify needed configuration details (url, API Keys, any other key value pairs needed in order to consume the service). The key value pairs would be dynamic (as in they could add as many as needed) there is already options for this type of field in the OpenShift UI (used to specify env vars).
When saved we would create the required objects (ie the configmap and a secret if needed) in the right format and with the correct labels to allow it to be consumed from the mobile CLI.

**Considerations**

Do we add this button to all services, it doesn't make sense to add it to something like MongoDB for example.

## Create Custom Service Connector APB

To allow mobile developers to expose existing custom logic contained within running services that may or may not be running on an OpenShift cluster, I propose we look at creating a custom service connector APB.

All of our mobile enabled services are provisioned via the catalog. I believe we should follow the same pattern for allowing mobile developers to expose custom services to the mobile application clients. This would give us the following advantages:

- It would create a ServiceInstance that we can then leverage as part of the UI work, ensuring we have a single Object to deal with for all service interactions in the UI and CLI.

- It would provide a familiar flow to developers of going to the catalog and setting up a service. 

- It would give us a simple way to allow setting up a custom service via the CLI

- It would ensure we have control over what and how the service is setup so that we can consume it both from the UI, CLI.

### How it would work

When you provisioned a custom service connector from the catalog, you would be shown a form that would have some required fields such as the host and a name for the service. Then it would accept dynamic key value pairs (in a similar fashion to what is done now for env vars). This would allow a developer to specify important configuration information needed to consume the service such as an API Key etc. As part of the provision we would take this information and create the required objects (likely just a configmap and perhaps a secret) and label them accordingly. As there would be a ServiceInstance created, the developer would also be able to add and exclude clients from the this service and deprovision it. To retrieve the configuration the developer would continue to use the UI screen or the CLI in exactly the same manner as before.

**Considerations**

- I do not believe there is a mechanism currently as part of the catalog ui to have dynamic fields.
https://github.com/openshift/ansible-service-broker/issues/797

- We can work around the above in the short term by allowing a text area field to be used and accepting a JSON definition for the required configuration.

