# Abstract

This document proposes a solution for Mobile Data Synchronisation in terms of the Sync Mobile SDK API, client/server transport mechanism, server API, server database. 

## Terms

* Mobile Data Synchronisation - The act of saving data on a Mobile device and it being synchronised to a server, and potentially to other Mobile devices. Typically includes the ability to save data while offline and have it synchronise while online.

## Problem Description

This problem can be divided into a number of smaller problems.

### Mobile Storage

A Mobile App typically needs to store data and/or retrieve data from a server.
For example, in a simple ToDo List App the list items will need to be stored so they appear the next time the App is started.
The type of data can vary from simple text data to large binary data.
Mobile/On-Device storage is required in this scenario.

### Remote Storage

There may be a situation where the Mobile App is uninstalled from the device, or the device is damaged/replaced.
If the data was only saved on device, this would lead to data loss.
Additionally, there may need to be other interfaces into the data, besides Mobile Apps.
For example, if the list items in the ToDo List App were set by a administrative worker or manager, this could be done using an administrative (non-mobile) portal that connects to the same remote storage as the Mobile Apps.
Remote Storage is required is these scenarios.

### Synchronisation

Data may need to be shared/synchronised between instances of the Mobile App.
In the ToDo List App example, this would be 2 people with the same App on their Mobile Device, and sharing the same list items.
Data that is modified on one Mobile device should be updated as fast as possible in the remote storage, and on other mobile devices.
Also, if data is modified in the remote storage from a non-mobile interface, the Mobile Apps should be notified of these changes.
Keeping the amount of time a Mobile Device is using stale/inconsistent data to a minimum is essential to the App users.
It will also lower the possibility of data conflicts.

### Offline

Mobile devices connect to networks wirelessly.
Network connectivity can be intermittent or limited based on the many factors, such as location, time of day, power levels, device configuration.
Because of this, Mobile Apps are typically written to provide a good offline experience.
There may also be a hard product/user requirement for offline functionality.
For example, an App may be used by 'field workers' that tend to work in poor network areas (underground, tunnels, rural).
It should be possible for Mobile Apps to continue to store & modify data while offline.
The data should be synchronised to the remote storage (and possibly other Mobile Devices) when network connectivity is available.

### Authentication & Authorization

Due to the distributed nature of Mobile Data Synchronisation, there are multiple ways data can be created or modified.
Only trusted users should be able to create or modify data. Additionally, it should be possible to restrict the type of data access users have.
For example, only Mobile App users who have signed in can read the data. Building on that, only users who has a particular role/permission assigned to them can create/update data.
It should be possible to define the type of authentication and levels of authorisation for data. In practice this could mean locking down access to specific data entities/models, or even specific data entries/rows.

## Proposed Solution

<!-- In no particular order, the following aspects should be present in the solution to the above problems.

* A Mobile SDK/API that makes storing, modifying & querying Data easy and natural for Mobile developers
* Data can be stored, modified & queried while offline
* Data synchronisation should be realtime under optimal network conditions
* Mobile developers shouldn't have to care about how it works (i.e. sensible configuration defaults)
* Conflict resolution should have a sensible default strategy (e.g. last in wins)
* Saving Binary content should be possible, without additional overhead (i.e. raw binary, not base64)
* Data Synchronisation should allow for spotty network conditions (e.g. chunking, pause/resume)
* Network overhead should be minimal (e.g. send diffs only) -->

The proposed solution looks at each smaller problem and how to solve them in a cohesive way that results in a single overall solution.

### Mobile Storage

x

### Remote Storage

x

### Synchronisation

x

### Offline

x

### Authentication & Authorization

x