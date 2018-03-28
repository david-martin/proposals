# Abstract

In this document we will discuss some of the common problems around network security for mobile developers consuming AeroGear services in their apps, and propose a solution that will help resolve those problems.

We will first look at some of the use cases in order to define the problems, and then look at some of the options to solve them. Finally the proposed solution will be presented.

# Goals of the proposal

## Primary Goal

* A solution that can help mobile developers properly handle network security in their mobile apps that consume AeroGear services

## Secondary Goal

* Provide use cases and try to explain the problems to audiences that may not be familiar with the area, to help them understand and maybe decide the business value of the solution

# Terms

* CA: Certificate Authority. An entity that can issue digital certificates. They are trusted third-parties.
* Self-signed Certificate: A self-signed certificate is an identity certificate that is signed by the same entity whose identity it certifies.
* Man-in-the-middle attack: It is an attack where the attacker secretly relays and possibly alters the communication between two parties who believe they are directly communicating with each other.

# Use Cases

The main use cases that will be discussed in this document are:

1. As a mobile developer, I want to use AeroGear services in my mobile apps. I have the AeroGear service running locally with HTTPS, and I want to test against the local services from my mobile app on an actual device.

2. As an enterprise mobile developer, I need to develop a mobile app for internal use. I need to make sure my mobile app can connect to AeroGear services that are running with the organisation's self-signed certificates.

3. As a mobile developer of a finance institution, I need to develop a mobile app that uses the AeroGear services for the customers of the organisation. However, due to highly sensitive nature of the data, I need to make sure that no one (including the app users themselves) can inspect the network data of the app.

# Problem Description

The above use cases can be categoried into 2 problems:

1. Use case 1 & 2 are about allowing mobile apps consume AeroGear services that use self-signed certificates

2. Use case 3 is about preventing Man-in-the-middle attack when mobile apps communicate with AeroGear services

## Problems Explained

If you already know quite well about the problems, then you can skip this section. 

### Problem 1: Support self-signed certificates

To consume AeroGear services, mobile apps will use the AeroGear SDKs, which use HTTP/HTTPS to communicate with the backend services. In most cases, HTTPS is used to ensure the security of the data during transimission. 

When the HTTPS connection is established, the HTTPS client on the device will need some way to verify the the authencitiy of the server. This is done by checking the server's certificate during the handshake between the client and the server. 

In most cases, the server should have a certificate that is signed by a trusted CA to confirm the identity of the server. The HTTP client can then use the trusted CAs' certificates that are pre-installed on the device to verify the server's certificate.

However, due to various reasons (like cost, and sometimes it is just not necessary), CA-signed certificates are not always being used for HTTPS connections. Instead, self-signed certificates are being used. But in this case, the HTTP client on the mobile device will not be able to verify the authentity of the server, and it will refuse to connect to the server. 

In use case #1, when the AeroGear services are running locally, they will be using self-signed certificates by default. This means developers will not be able to connect to these services from their mobile devices by default.

To solve this problem, all the modern mobile OSes allow users install self-signed CA certificates on their devices and trust them. However, there is a side effect: it is possible for attackers to take advantages of it and perform man-in-the-middle attacks.

### Problem 2: Prevent Man-in-the-middle attack

The whole reason to use HTTPS is to make sure the communication between the client and server are secure and no one can eavesdrop. However, that is not always the case, especially when the users of the app become the attackers.

For example, user A is an experienced hacker and he wants to hack into the backend system of a bank. Initially he doesn't know what kind of backend system the bank is using so he doesn't know where to start. But then he noticed that the bank has a consumer app. So he installed the app on his own device.

Next, he set up a proxy server using tools like [Charles proxy server](https://www.charlesproxy.com/). He install the proxy server's certificate on his own device, and pointed the device to use the proxy server. Since the proxy server's certificate is now trusted on his device, the HTTP client will allow HTTPS connection to the proxy server, and user A can see all the traffic data between the client and the server. With this information, user A is now able to initiate attacks against the bank's backend system.

In this example, the user of the app wants to become the attacker. But more seriously, this type of attacks can be performed without permissions from users. For example, a hacker can setup a malicious WIFI hotspot, acting like a proxy server, and ask users to install the self-signed certficates to their devices in exchange for the free connection. If users don't know what they are doing, they could endup installing the self-signed certificates and connecting to the WIFI, and thus give the hacker the opportunity to eavesdrop all the HTTPS traffic, and steal important data, without them realising it.

To prevent this type of attacks, a technique called "Certificate Pinning" is being used. The idea is that for most of the applications, they will talk to backend servers that are controlled by the same developers. Given the fact that each certificate is unique, at the application build time, the developers can generate "fingerprints" of the certificates of their backend services and "pin" them in the app. Then when the application is trying to establish the HTTPS connection, in addition to the default certificate checks, the HTTP client will also compute the "fingerprints" of the certificates and compare it againsted the "pinned" ones. If they are not matching, then the HTTP client knows that it is actually not talking to the real backend server, and thus refuse to setup the connection.

## Solutions

### Options

Generally speaking, the above problems can be solved by 2 approaches:

1. The first approach is to use the solution provided by the OS. 

    For example, Android provides [Network Security Configurations](https://developer.android.com/training/articles/security-config.html) to allow developers to configure what CA store the apps can trust, and the "fingerprints" of their backend services.

    The advantage of this approach is that there are no code changes required, and provides a center location for all the network security related configurations. Once configured, all the network libaries will respect the settings.

    However, the biggest problem with this approach is the lack of standards accross different OSes. For example, iOS doesn't have the equivalent implementation for this feature. Even for Android itself, this feature is only supported on apps targeting Android 6.0 and above.

2. The second approach is to use the APIs provided by the HTTP libaries. 

    Most of the modern HTTP libaries allow developers to configure many different aspects of the HTTPS connections, and make it possible to support self-signed certificates and certificate pinning.

    The downside of this approach is it requires developers to make code changes to their apps, and changes need to be made to every HTTP libary the apps are using. However, it provides a good foundation to form a solution that can work across different platforms.

### Proposed Solution

Based on the information above, and the fact that we want to find a solution that can work across different platforms for AeroGear services, we propose the following solution:

1. In AeroGear SDKs, the HTTP libaries (there is only one for each platform) will be configured using the SDK config file (`mobile-services.json`). There will be a new section like this:

    ```
    {
      ...,
      "https": {
        "selfSignedCerts": {
          "enabled": true,
          "keystores": [{
            "path": "assets/dev.keystore"
          }]
        },
        "certificatePins": [{
          "host": "example.sync.service",
          "certificateHash": "exampleHash"
        }]
      },
      ...
    }
    ```

2. During the app runtime, when the AeroGear SDK is initialised, it will parse the configuration file, and configure the HTTP client to use the specified certificate pins or support self-signed certificates automatically.

    * For Android, the [okhttp](http://square.github.io/okhttp/) library will be configured to use the provided Keystore files (if any) and the certificate pins.
    * For iOS, if self-signed certificate is enabled, the [Alamofire](https://github.com/Alamofire/Alamofire) library will not validate the certificates against CAs at all. It will just perform certificate pin comparsions.
    * For Cordova, it is likely we will not be able to support self-signed certs from apps, and it has to be done at the OS level. This is mainly due to the lack of customisation support by the XMLHTTPRequest objects in webviews. We might be able to support certificate pinning though, by using a plugin to allow developers to manually invoke a request to a URL using the native HTTP client before starting the request in the Webview.
    * Not sure about what we can do for Xamarin yet.


3. The mobile CLI tool will also be updated to automatically generate the above configurations (and possibly keystore files) based on the CLI flags. Both features will be disabled by default. Once enabled, it will generate certificate pins for all the AeroGear backend services that are enabled for the app. In the future, we will allow developers to select which service should use certificate pinning.

4. Documentations will also be provided to callout some of the risks and pitfalls of using these features to help developers make better decisions.