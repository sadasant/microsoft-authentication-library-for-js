# Errors

***

**[BrowserConfigurationAuthErrors](#Browserconfigurationautherrors)**

1. [stubbed_public_client_application_called](#stubbed_public_client_application_called)

**[BrowserAuthErrors](#browserautherrors)**

1. [interaction_in_progress](#interaction_in_progress)
1. [block_iframe_reload](#block_iframe_reload)

**[Other](#other)**

1. [Access to fetch at [url] has been blocked by CORS policy](#Access-to-fetch-at-[url]-has-been-blocked-by-CORS-policy)

***

## BrowserConfigurationAuthErrors

### stubbed_public_client_application_called

**Error Message**: Stub instance of Public Client Application was called. If using msal-react, please ensure context is not used without a provider.

See [msal-react errors](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-react/docs/errors.md)

## BrowserAuthErrors

### Interaction_in_progress

**Error Message**: Interaction is currently in progress. Please ensure that this interaction has been completed before calling an interactive API.

This error is thrown when an interactive API (`loginPopup`, `loginRedirect`, `acquireTokenPopup`, `acquireTokenRedirect`) is invoked while another interactive API is still in progress. The login and acquireToken APIs are async so you will need to ensure that the resulting promises have resolved before invoking another one.

#### Using `loginPopup` or `acquireTokenPopup`

Ensure that the promise returned from these APIs has resolved before invoking another one.

❌ The following example will throw this error because `loginPopup` will still be in progress when `acquireTokenPopup` is called:

```javascript
const request = {scopes: ["openid", "profile"]}
loginPopup();
acquireTokenPopup(request);
```

✔️ To resolve this you should ensure all interactive APIs have resolved before invoking another one:

```javascript
const request = {scopes: ["openid", "profile"]}
await msalInstance.loginPopup();
await msalInstance.acquireTokenPopup(request);
```

#### Using `loginRedirect` or `acquireTokenRedirect`

When using redirect APIs, `handleRedirectPromise` must be invoked when returning from the redirect. This ensures that the token response from the server is properly handled and temporary cache entries are cleaned up. This error is thrown when `handleRedirectPromise` has not had a chance to complete before the application invokes `loginRedirect` or `acquireTokenRedirect`.

❌ The following example will throw this error because `handleRedirectPromise` will still be processing the response from a previous `loginRedirect` call when `loginRedirect` is called a 2nd time:

```javascript
msalInstance.handleRedirectPromise();

const accounts = msalInstance.getAllAccounts();
if (accounts.length === 0) {
    // No user signed in
    msalInstance.loginRedirect();
}
```

✔️ To resolve, you should wait for `handleRedirectPromise` to resolve before calling any interactive API:

```javascript
await msalInstance.handleRedirectPromise();

const accounts = msalInstance.getAllAccounts();
if (accounts.length === 0) {
    // No user signed in
    msalInstance.loginRedirect();
}
```

Or alternatively:

```javascript
msalInstance.handleRedirectPromise()
    .then((tokenResponse) => {
        if (!tokenResponse) {
            const accounts = msalInstance.getAllAccounts();
            if (accounts.length === 0) {
                // No user signed in
                msalInstance.loginRedirect();
            }
        } else {
            // Do something with the tokenResponse
        }
    })
    .catch(err => {
        // Handle error
        console.error(err);
    });
```

**Note:** If you are calling `loginRedirect` or `acquireTokenRedirect` from a page that is not your `redirectUri` you will need to ensure `handleRedirectPromise` is called and awaited on both the `redirectUri` page as well as the page that you initiated the redirect from. This is because the `redirectUri` page will initiate a redirect back to the page that originally invoked `loginRedirect` and that page will process the token response.

#### Wrapper Libraries

If you are using one of our wrapper libraries (React or Angular), please see the error docs in those specific libraries for additional reasons you may be receiving this error:

- [msal-react errors](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-react/docs/errors.md)
- [msal-angular errors](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/docs/v2-docs/errors.md)

#### Troubleshooting Steps

- [Enable verbose logging](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/configuration.md#using-the-config-object) and trace the order of events. Verify that `handleRedirectPromise` is called and returns before any `login` or `acquireToken` API is called.

If you are unable to figure out why this error is being thrown please [open an issue](https://github.com/AzureAD/microsoft-authentication-library-for-js/issues/new/choose) and be prepared to share the following information:

- Verbose logs
- A sample app and/or code snippets that we can use to reproduce the issue
- Refresh the page. Does the error go away?
- Open your application in a new tab. Does the error go away?

### block_iframe_reload

**Error Message**: Request was blocked inside an iframe because MSAL detected an authentication response.

This error is thrown when calling `ssoSilent` or `acquireTokenSilent` and the page used as your `redirectUri` is attempting to invoke a login or acquireToken function.
Our recommended mitigation for this is to set your `redirectUri` to a blank page that does not implement MSAL when invoking silent APIs. This will also have the added benefit of improving performance as the hidden iframe doesn't need to render your page.

✔️ You can do this on a per request basis, for example:

```javascript
msalInstance.acquireTokenSilent({
    scopes: ["User.Read"],
    redirectUri: "http://localhost:3000/blank.html"
});
```

Remember that you will need to register this new `redirectUri` on your App Registration.

If you do not want to use a dedicated `redirectUri` for this purpose, you should instead ensure that your `redirectUri` is not attempting to call MSAL APIs when rendered inside the hidden iframe used by the silent APIs.

## Other

Errors not thrown by msal, such as server errors

### Access to fetch at [url] has been blocked by CORS policy

This error occurs with MSAL.js v2.x and is due to improper configuration during **App Registration** on **Azure Portal**. In particular, you should ensure your `redirectUri` is registered as type: `Single-page application` under the **Authentication** blade in your App Registration. If done successfully, you will see a green checkmark that says: 

> Your Redirect URI is eligible for the Authorization Code Flow with PKCE.

![image](https://user-images.githubusercontent.com/5307810/110390912-922fa380-801b-11eb-9e2b-d7aa88ca0687.png)
