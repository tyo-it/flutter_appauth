# Flutter AppAuth Plugin

A Flutter bridge for AppAuth (https://appauth.io) used authenticating and authorizing users. Note that AppAuth also supports the PKCE extension that is required some providers so this plugin should work with them.

**NOTE**: this library uses AndroidX and there is currently a known issuer with Jetifier that affects the use of Chrome Custom Tabs (see https://issuetracker.google.com/issues/119183822). This means that until a fix for it is released, signing in will direct users to the browser as opposed to using Chrome Custom Tabs.


## Getting Started

Please see the example that demonstrates how to sign into the IdentityServer4 demo site (https://demo.identityserver.io). It has also been tested with Azure B2C and Google Sign-in. API docs can be found [here](https://pub.dartlang.org/documentation/flutter_appauth/latest/)


The first step is to create an instance of the plugin

```
FlutterAppAuth appAuth = FlutterAppAuth();
```

Afterwards, you'll reach a point where end-users need to be authorized and authenticated. A convenience method is provided that do and automatically exchange the authorization code. This can be done in a few different ways, one of which is to use the OpenID Connect Discovery

```
var result = await appAuth.authorizeAndExchangeCode(
                    AuthorizationTokenRequest(
                      '<client_id>',
                      '<redirect_url>',
                      discoveryUrl: '<discovery_url>',
                      scopes: ['openid','profile', 'email', 'offline_access', 'api'],
                    ),
                  );
```

Here the `<client_id>` and `<redirect_url>` should be replaced by the values registered with your identity provider. The `<discovery_url>` would be the URL for the discovery endpoint exposed by your provider that will return a document containing information about the OAuth 2.0 endpoints among other things. This URL is obtained by concatenating the issuer with the path `/.well-known/openid-configuration`. For example, the full URL for the IdentityServer4 demo site is `https://demo.identityserver.io/.well-known/openid-configuration`. As demonstrated in the above sample code, it's also possible specify the `scopes` being requested.

It's possible to perform the above request with just the issuer as well

```
var result = await appAuth.authorizeAndExchangeCode(
                    AuthorizationTokenRequest(
                      '<client_id>',
                      '<redirect_url>',
                      issuer: '<issuer>',
                      scopes: ['openid','profile', 'email', 'offline_access', 'api'],
                    ),
                  );
```

If you already know the authorization and token endpoints, then these could be explicitly specified

```
var result = await appAuth.authorizeAndExchangeCode(
                    AuthorizationTokenRequest(
                      '<client_id>',
                      '<redirect_url>',
                      serviceConfiguration: AuthorizationServiceConfiguration('<authorization_endpoint>', '<token_endpoint>'),
                      scopes: ['openid','profile', 'email', 'offline_access', 'api']
                    ),
                  );
```

Upon completing the request successfully, the method should return an object (the `result` variable in the above sample code is an instance of the `AuthorizationTokenResponse` class) that contain details that should be stored for future use e.g. access token, refresh token etc.

If you would prefer to not have the automatic code exchange to happen then can call the `authorize` method instead of the `authorizeAndExchangeCode` method. This will return an instance of the `AuthorizationResponse` class that will contain the code verifier that AppAuth generated (as part of implementing PKCE) when issuing the authorization request, the authorization code and additional parameters should they exist. Both of the code verifier and authorization code would need to be stored so they can then be reused to exchange the code later on e.g.

```
var result = await appAuth.token(TokenRequest('<client_id>', '<redirect_url>',
        authorizationCode: '<authorization_code>',
        discoveryUrl: '<discovery_url>',
        codeVerifier: '<code_verifier>',
        scopes: ['openid','profile', 'email', 'offline_access', 'api']));
```

# Refreshing tokens

Some providers may return a refresh token that could be used to refresh short-lived access tokens. A request to get a new access token before it expires could be made that would like similar to the following code

```
var result = await appAuth.token(TokenRequest('<client_id>', '<redirect_url>',
        authorizationCode: '<authorization_code>',
        discoveryUrl: '<discovery_url>',
        refreshToken: '<refresh_token>',
        scopes: ['openid','profile', 'email', 'offline_access', 'api']));
```

## Android setup

Go to the `build.gradle` file for your Android app to specify the custom scheme so that there should be a section in it that look similar to the following but replace `<your_custom_scheme>` with the desired value

```
...
android {
    ...
    defaultConfig {
        ...
        manifestPlaceholders = [
                'appAuthRedirectScheme': '<your_custom_scheme>'
        ]
    }
}
```

## iOS setup

Go to the `Info.plist` for your iOS app to specify the custom scheme so that there should be a section in it that look similar to the following but replace `<your_custom_scheme>` with the desired value


```
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string><your_custom_scheme></string>
        </array>
    </dict>
</array>
```