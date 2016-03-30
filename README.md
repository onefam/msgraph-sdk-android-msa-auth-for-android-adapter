# MSGraph SDK Android MSA Auth for Android Adapter

[ ![Download](https://api.bintray.com/packages/microsoftgraph/Maven/msgraph-sdk-android-msa-auth-for-android-adapter/images/download.svg) ](https://bintray.com/microsoftgraph/Maven/msgraph-sdk-android-msa-auth-for-android-adapter/_latestVersion)
[![Build Status](https://travis-ci.org/microsoftgraph/msgraph-sdk-android-msa-auth-for-android-adapter.svg?branch=master)](https://travis-ci.org/microsoftgraph/msgraph-sdk-android-msa-auth-for-android-adapter)

## Overview

The MSGraph SDK Android MSA Auth for Android Adapter library provides an implementation of the Authentication Interface for the Microsoft Graph SDK to use with the V2 Authentication Endpoint.

This authentication providers is limited in scope and are provided to facilitate development with the V2 Authentication Endpoint while development of MSAL for each platform progresses to general availability.  Please see documentation on what is currently supported on the V2 Authentication endpoint [link](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/).

This library provides the following features login, request signing, and logout.  Features not listed are not supported or included and are the responsibility of the developer to support and implement.

It is encouraged to fork or use this implementation as a starting point to develop functionality specific to your needs.

## 1. Installation

### 1.1 Install AAR via Gradle
Add the maven central repository to your projects build.gradle file then add a compile dependency for `com.microsoft.graph:msa-auth-for-android-adapter:0.9.+`

```gradle
repository {
    jcenter()
}

dependency {
    // Include MSGraph SDK Android MSA Auth for Android Adapter as a dependency
    compile 'com.microsoft.graph:msa-auth-for-android-adapter:0.9.+'
}
```

## 2. Getting started

### 2.1 Register your application

Register your application by following [these](http://graph.microsoft.io/en-us/app-registration) steps.

#### 2.1.1 Create a new application in Application Registration Portal

In the [Application Registration Portal](http://apps.dev.microsoft.com/) Create a new application for the converged endpoint, which will function with both MSA and AAD accounts.

#### 2.1.2 Add a mobile application platform to the application

Once the application has been created add platform, selecting the `Mobile Application` option.  This will create a new Mobile Application with a redirect url of `urn:ietf:wg:oauth:2.0:oob`.

**Note:** Make sure you scroll to the button of the page and hit the Save Button to save your platform changes.

### 2.2 Set your application Id and scopes

Note that your _client id_ should look like `00000000-0000-0000-0000-000000000000`.

```java
final IAuthenticationAdapter authenticationAdapter = new MSAAuthAndroidAdapter() {
    @Override
    public String getClientId() {
        return "<client-id>";
    }

    @Override
    protected String[] getScopes() {
        return new String[] {
            // An example set of scopes your application could use
            "https://graph.microsoft.com/Calendars.ReadWrite",
            "https://graph.microsoft.com/Contacts.ReadWrite",
            "https://graph.microsoft.com/Files.ReadWrite",
            "https://graph.microsoft.com/Mail.ReadWrite",
            "https://graph.microsoft.com/Mail.Send",
            "https://graph.microsoft.com/User.ReadBasic.All",
            "https://graph.microsoft.com/User.ReadWrite",
            "offline_access",
            "openid"
        };
    }
}
```

## 3. Using the IAuthenticationAdapter with a GraphService client

### 3.1 Logging In

Once you have set the client-id and scopes, you need to have your application manage the sign in state of the user, so you can make requests against the Graph Service.   Use the login method to force a user login, during the login flow the user will consent to use the application.

Once the authorization flow is completed the callback provided will be called returning the flow of control back to the application.

```java
authenticationAdapter.login(getActivity(), new ICallback<Void>() {
    @Override
    public void success(final Void aVoid) {
        //Handle successful login
    }

    @Override
    public void failure(final ClientException ex) {
        //Handle failed login
    }
};
```

### 3.2 Logging Out

```java
authenticationAdapter.logout(new ICallback<Void>() {
    @Override
    public void success(final Void aVoid) {
        //Handle successful logout
    }

    @Override
    public void failure(final ClientException ex) {
        //Handle failed logout
    }
};
```

### 3.3 Integration with Graph Service client

Once the application has signed in a account to access to Graph Service the application should create a `GraphServiceClient` to issue requests with.

```java
// Use the authentication provider previously defined within the project and create a configuration instance
final IClientConfig config = DefaultClientConfig.createWithAuthenticationProvider(authenticationAdapter);

// Create the service client from the configuration
final IGraphServiceClient client = new GraphServiceClient.Builder()
                                        .fromConfig(config)
                                        .buildClient();
```

### 3.4 Handling Authentication exceptions

While the `IAuthenticationAdapter` is being used to sign requests, it is possible for the account state to become invalid and the user would need to login again.  The recommended way to handle this scenario is to check for the `AuthenticationFailure` error code from a failed request.

```java
// Create a callback that can understand if AuthenticationFailure occurred.
final ICallback<User> callback = new ICallback<User>() {
    @Override
    public void success(final User user) {
        //Handle successful logout
    }

    @Override
    public void failure(final ClientException ex) {
        if (ex.isError(GraphErrorCodes.AuthenticationFailure))
        {
            // Reset application to login again
        }

        // Handle other failed causes
    }
}

// Make a request for the current user object
client
    .getMe()
    .buildRequest()
    .get(callback);
```

## 4. Issues

For known issues, see [issues](https://github.com/microsoftgraph/msgraph-sdk-android-msa-auth-for-android-adapter/issues).

## 5. Contributions

The MSGraph SDK Android MSA Auth for Android Adapter is open for contribution for bug fixes.  This project is intended to help developers get off the ground quickly with the [MSGraph SDK for Android](https://github.com/microsoftgraph/msgraph-sdk-android) and not to serve as an all inclusive authentication library.

## 6. Supported Android Versions
The OneDrive SDK for Android library is supported at runtime for [Android API revision 15](http://source.android.com/source/build-numbers.html) and greater. To build the sdk you need to install Android API revision 23 or greater.

## 7. License

Copyright (c) Microsoft Corporation.  All Rights Reserved.  Licensed under the [MIT license](LICENSE).
