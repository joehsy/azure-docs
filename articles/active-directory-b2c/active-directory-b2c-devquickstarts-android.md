---
title: 'Azure Active Directory B2C: Acquiring a token using an Android application | Microsoft Docs'
description: This article will show you how to create an Android app that uses AppAuth with Azure Active Directory B2C to manage user identities and authenticate users.
services: active-directory-b2c
documentationcenter: android
author: parakhj
manager: mtillman
editor: ''

ms.assetid: d00947c3-dcaa-4cb3-8c2e-d94e0746d8b2
ms.service: active-directory-b2c
ms.workload: identity
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: article
ms.date: 03/06/2017
ms.author: parakhj

---
# Azure AD B2C: Sign-in using an Android application

The Microsoft identity platform uses open standards such as OAuth2 and OpenID Connect. This allows developers to leverage any library they wish to integrate with our services. To aid developers in using our platform with other libraries, we've written a few walkthroughs like this one to demonstrate how to configure 3rd party libraries to connect to the Microsoft identity platform. Most libraries that implement [the RFC6749 OAuth2 spec](https://tools.ietf.org/html/rfc6749) will be able to connect to the Microsoft Identity platform.

> [!WARNING]
> Microsoft does not provide fixes for 3rd party libraries and has not done a review of those libraries. This sample is using a 3rd party library called AppAuth that has been tested for compatibility in basic scenarios with the Azure AD B2C. Issues and feature requests should be directed to the library's open-source project. Please see [this article](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-libraries) for more information.  
>
>

If you're new to OAuth2 or OpenID Connect much of this sample configuration may not make much sense to you. We recommend you look at a brief [overview of the protocol we've documented here](active-directory-b2c-reference-protocols.md).

## Get an Azure AD B2C directory

Before you can use Azure AD B2C, you must create a directory, or tenant. A directory is a container for all of your users, apps, groups, and more. If you don't have one already, [create a B2C directory](active-directory-b2c-get-started.md) before you continue.

## Create an application

Next, you need to create an app in your B2C directory. This gives Azure AD information that it needs to communicate securely with your app. To create a mobile app, follow [these instructions](active-directory-b2c-app-registration.md). Be sure to:

* Include a **Native Client** in the application.
* Copy the **Application ID** that is assigned to your app. You will need this later.
* Set up a native client **Redirect URI** (e.g. com.onmicrosoft.fabrikamb2c.exampleapp://oauth/redirect). You will also need this later.

[!INCLUDE [active-directory-b2c-devquickstarts-v2-apps](../../includes/active-directory-b2c-devquickstarts-v2-apps.md)]

## Create your policies

In Azure AD B2C, every user experience is defined by a [policy](active-directory-b2c-reference-policies.md). This app contains one identity experience: a combined sign-in and sign-up. You need to create this policy, as described in the
[policy reference article](active-directory-b2c-reference-policies.md#create-a-sign-up-policy). When you create the policy, be sure to:

* Choose the **Display name** as a sign-up attribute in your policy.
* Choose the **Display name** and **Object ID** application claims in every policy. You can choose other claims as well.
* Copy the **Name** of each policy after you create it. It should have the prefix `b2c_1_`.  You'll need the policy name later.

[!INCLUDE [active-directory-b2c-devquickstarts-policy](../../includes/active-directory-b2c-devquickstarts-policy.md)]

After you have created your policies, you're ready to build your app.

## Download the sample code

We have provided a working sample that uses AppAuth with Azure AD B2C [on GitHub](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c). You can download the code and run it. You can quickly get started with your own app using your own Azure AD B2C configuration by following the instructions in the [README.md](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c/blob/master/README.md).

The sample is a modification of the sample provided by [AppAuth](https://openid.github.io/AppAuth-Android/). Please visit their page to learn more about AppAuth and its features.

## Modifying your app to use Azure AD B2C with AppAuth

> [!NOTE]
> AppAuth supports Android API 16 (Jellybean) and above. We recommend using API 23 and above.
>

### Configuration

You can configure communication with Azure AD B2C by either specifying the discovery URI or by specifying both the authorization endpoint and token endpoint URIs. In either case, you will need the following information:

* Tenant ID (e.g. contoso.onmicrosoft.com)
* Policy name (e.g. B2C\_1\_SignUpIn)

If you choose to automatically discover the authorization and token endpoint URIs, you will need to fetch information from the discovery URI. The discovery URI can be generated by replacing the Tenant\_ID and the Policy\_Name in the following URL:

```java
String mDiscoveryURI = "https://login.microsoftonline.com/<Tenant_ID>/v2.0/.well-known/openid-configuration?p=<Policy_Name>";
```

You can then acquire the authorization and token endpoint URIs and create an AuthorizationServiceConfiguration object by running the following:

```java
final Uri issuerUri = Uri.parse(mDiscoveryURI);
AuthorizationServiceConfiguration config;

AuthorizationServiceConfiguration.fetchFromIssuer(
    issuerUri,
    new RetrieveConfigurationCallback() {
      @Override public void onFetchConfigurationCompleted(
          @Nullable AuthorizationServiceConfiguration serviceConfiguration,
          @Nullable AuthorizationException ex) {
        if (ex != null) {
            Log.w(TAG, "Failed to retrieve configuration for " + issuerUri, ex);
        } else {
            // service configuration retrieved, proceed to authorization...
        }
      }
  });
```

Instead of using discovery to obtain the authorization and token endpoint URIs, you can also specify them explicitly by replacing the Tenant\_ID and the Policy\_Name in the URL's below:

```java
String mAuthEndpoint = "https://login.microsoftonline.com/<Tenant_ID>/oauth2/v2.0/authorize?p=<Policy_Name>";

String mTokenEndpoint = "https://login.microsoftonline.com/<Tenant_ID>/oauth2/v2.0/token?p=<Policy_Name>";
```

Run the following code to create your AuthorizationServiceConfiguration object:

```java
AuthorizationServiceConfiguration config =
        new AuthorizationServiceConfiguration(name, mAuthEndpoint, mTokenEndpoint);

// perform the auth request...
```

### Authorizing

After configuring or retrieving an authorization service configuration, an authorization request can be constructed. To create the request, you will need the following information:

* Client ID (e.g. 00000000-0000-0000-0000-000000000000)
* Redirect URI with a custom scheme (e.g. com.onmicrosoft.fabrikamb2c.exampleapp://oauthredirect)

Both items should have been saved when you were [registering your app](#create-an-application).

```java
AuthorizationRequest req = new AuthorizationRequest.Builder(
    config,
    clientId,
    ResponseTypeValues.CODE,
    redirectUri)
    .build();
```

Please refer to the [AppAuth guide](https://openid.github.io/AppAuth-Android/) on how to complete the rest of the process. If you need to quickly get started with a working app, check out [our sample](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c). Follow the steps in the [README.md](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c/blob/master/README.md) to enter your own Azure AD B2C configuration.

We are always open to feedback and suggestions! If you have any difficulties with this topic, or have recommendations for improving this content, we would appreciate your feedback at the bottom of the page. For feature requests, add them to [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160596-b2c).

