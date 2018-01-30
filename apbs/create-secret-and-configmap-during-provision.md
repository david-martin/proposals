# Abstract

This document outlines a proposal to create an additional configmap for services accessible via a mobile client, this configmap would compliment the secret we currently create during the provision of each service via an APB 

## Terms

- Mobile Client (a mobile device running a developers application)
- APB (ansible playbook bundle)

## Problem Description

Our current approach is to create a secret during the provision of a new mobile service. This secret contains all of the information required to allow the UI to show the service and how to access the service, it also includes any information needed to create the mobile client configuration. This means we expose too much information and also need to filter out any sensitive information (usernames password etc) when constructing our mobile configuration to avoid it being exposed to the client and ending up in the public domain. As each service is different, it often requires a custom piece of code to filter out the fields that we don't want or need exposed. 

## Proposed Solution

When provisioning a new service instance, we should create two configuration resources:

1) A secret containing private information such as usernames and passwords or anything that we do not want to be exposed to the mobile client.
2) A configmap containing all of the required information needed for a mobile client to use the service.

Note it may be neccessary to duplicate some information across each of these resources (ie the URI of the service for example).

Each of these resources should have at least the following information and labels:

- Labels: 
    - **mobile:enabled**. Will give a mechanism to filter these resources from other resources of the same type from the UI and CLI
    - **serviceName:< serviceName >**. Depending on whether we want to allow more than one service of a particular kind to be provisioned to a namespace, this will allow us to identify which service the information is for without needing to name the resource itself after the service.

- Data
    - **URI** both the configmap and secret will require this. 


### CLI and UI changes

Currently the CLI and UI look for secrets to create the mobile client config, this will need to change to look for configmaps with the label ```mobile:enabled```.
There should be no changes to representing the services as the secret will continue to exist.


### APB Changes

Each APB for a service that needs to be exposed to the mobile client, will need to be changed to create the specified configmap. The APB will also need to be changed to remove this configmap when the service is removed.




