---
title: Sign users in to your mobile app using the redirect model
excerpt: Configure your Okta org and your mobile app to use Okta's redirect sign-in flow.
layout: Guides
---

Add authentication using the Okta [redirect model](https://developer.okta.com/docs/concepts/redirect-vs-embedded/#redirect-authentication) to your mobile app. This example uses Okta as the user store.

---

**Learning outcomes**

* Create an integration that represents your app in your Okta org.
* Add dependencies and configure your mobile app to use Okta.
* Add a browser-based sign-in flow that is handled by Okta (redirect authentication).
* Load details for the signed in user.
* Check for an existing authenticated session at application startup.
* Refresh tokens to keep the user signed in.
* Make server calls using an access token for the session.
* Validate the integration by signing in as a user.

**Sample code**

<StackSnippet snippet="samplecode" />

---

## Set up Okta

Set up your [Okta org](/docs/concepts/okta-organizations/). The CLI is by far the quickest way to work with your Okta org, so we recommend using it for the first few steps. If you don't want to install the CLI, you can [manually sign up for an org](https://developer.okta.com/signup/) instead. We provide non-CLI instructions along with the CLI steps below.

1. Install the Okta command-line interface: [Okta CLI](https://cli.okta.com/).
2. If you don't already have a free Okta developer account, create one by entering `okta register` on the command line.
3. Make a note of the Okta Domain as you need that later.
4. **IMPORTANT:** Set the password for your Okta developer org by opening the link that's shown after your domain is registered. Look for output similar to this:

```
Your Okta Domain: https://dev-xxxxxxx.okta.com
To set your password open this link:
https://dev-xxxxxxx.okta.com/welcome/xrqyNKPCZcvxL1ouKUoh
```

> **Note**: If you don't receive the confirmation email sent as part of the creation process, check your spam filters for an email from `noreply@okta.com`.

5. Connect to your Okta developer org if you didn't create one in the last step (successfully creating an Okta org also signs you in) by running the following command. You need the URL of your org &mdash; which is your [Okta domain](/docs/guides/find-your-domain/) with `https://` prepended &mdash; and an [API/access token](/docs/guides/create-an-api-token/).

```
okta login
```

## Create an Okta integration for your app

An application integration represents your app in your Okta org. The integration configures how your app integrates with the Okta services including: which users and groups have access, authentication policies, token refresh requirements, redirect URLs, and more. The integration includes configuration information required by the app to access Okta.

To create your app integration in Okta using the CLI:

1. Create the app integration by running:

```
okta apps create native
```

2. Enter **Quickstart** when prompted for the app name.
3. Specify the required redirect URI values:
<StackSnippet snippet="redirectvalues" />
4. Make note of the Redirect URI, Post Logout Redirect URI, and the application configuration printed to the terminal. You'll need these to configure your mobile app.

At this point, you can move to the next step: [Creating your app](#create-app). If you want to set up the integration manually or find out what the CLI just did for you, read on.

1. [Sign in to your Okta organization](https://developer.okta.com/login) with your administrator account.
1. Click the **Admin** button on the top right of the page.
1. Open the Applications configuration pane by selecting **Applications** > **Applications**.
1. Click **Create App Integration**.
1. Select a **Sign-in method** of **OIDC - OpenID Connect**.
1. Select an **Application type** of **Native Application**, then click **Next**.
    > **Note:** If you choose an inappropriate application type, it can break the sign-in or sign-out flows by requiring the verification of a client secret, which is something that public clients don't have.
1. Enter an **App integration name**.
1. Enter the callback route for the **Sign-in redirect URIs**. This is the [full redirect URI](#define-a-callback-route) for your mobile app (for example, `com.okta.example:/callback`).
1. Enter your callback route for the **Sign-out redirect URIs**. This is the [full redirect URI](#define-a-callback-route) for your mobile app (for example, `com.okta.example:/`).
1. Click **Save** to update the Okta app settings.

## Create app

In this section you create a sample mobile app and add redirect authentication using your new Okta app integration.

### Create a new project

<StackSnippet snippet="createproject" />

### Add packages

Add the required dependencies for using the Okta SDK with your app.

<StackSnippet snippet="addconfigpkg" />

### Configure your app

Our app uses information from the Okta integration that we created earlier to configure communication with the API: Redirect URI, Post Logout Redirect URI, Client ID and Issuer.

<StackSnippet snippet="configmid" />

#### Find your config values

If you don't have your configuration values handy, you can find them in the Admin Console (choose **Applications** > **Applications** and find your app integration that you created earlier):

* **Redirect URI**: Found on the **General** tab in the **Login** section.
* **Post Logout Redirect URI**: Found on the **General** tab in the **Login** section.
* **Client ID**: Found on the **General** tab in the **Client Credentials** section.
* **Issuer**: Found in the **Issuer URI** field for the authorization server that appears by selecting **Security** > **API** from the left navigation pane.

### Define a callback route

To sign users in, your application will generally open a browser and display an Okta-hosted sign-in page. Okta then redirects back to your app with information about the user.

To achieve this you need to define how Okta can redirect back to your app. This is called a callback route or redirect URI. In mobile apps, use a custom scheme similar to `your-app:/callback` so that your app can switch back into the foreground after the user is done signing in through the browser. This should be the same value that you used for the Sign-in and Sign-out redirect URIs when you created an app integration in the previous steps.

Your mobile app is responsible for parsing the information Okta sends to the callback route. Our SDKs can help you with this task (covered later in the [Open the sign-in page](#open-the-sign-in-page) section). For now, just define the route itself.

<StackSnippet snippet="definecallback" />

## Open the sign-in page

The SDK signs in the user by opening an Okta-hosted web page. The app may send the SDK sign-in request when a user visits a protected route or when they tap a button.

<StackSnippet snippet="opensignin" />

## Get info about the user

After the user signs in, Okta returns some of their profile information to your app (see [/userinfo response example](/docs/reference/api/oidc/#response-example-success-6)). You can use this information to update your UI, for example, to show the customer's name.

The default profile items (called `claims`) returned by Okta include the user's email address, name, and preferred username. The claims that you see may differ depending on what scopes your app has requested (see [Configure your app](#configure-your-app)).

<StackSnippet snippet="getuserinfo" />

## Sign in a user

Test your integration by signing in a user using your mobile app.

<StackSnippet snippet="testapp" />

## Check for a session at startup

Okta issues an access token when the user signs in. This token allows the user to access the services for a set amount of time. Check for an existing unexpired token when the app launches to find out if the user is still signed in.

<StackSnippet snippet="checkfortoken" />

## Keep the user signed in

Access tokens are short-lived, but for some types of apps users expect to remain signed in for a long time. Granting a refresh token in your app integration enables the client to request an updated access token.

Enable a refresh token in your app integration by following these steps:

1. Launch the Admin Console for your Okta org.
1. Choose **Applications > Applications** to show the app integrations.
1. Click the name of your integration to open the configuration manager.
1. Click **Edit** in the **General Settings** section.
1. Select **Refresh Token** in the **Application** section.
1. Click **Save** at the bottom of the **General Settings** section.

<StackSnippet snippet="refresh" />

## Use the access token

Your own server API (a resource server in OAuth 2.0) uses the Okta-generated access token for both authenticating that the user is signed in and ensuring that the user is authorized to perform the action or access the information.

Use the access token by adding it to the HTTP `Authorization` header of outgoing requests in your mobile or other client using this format:

```
Authorization: Bearer ${token}
```

Your API then checks incoming requests for valid tokens. To learn how to protect your API endpoints and require token authentication, see [Protect your API endpoints](/docs/guides/protect-your-api/).

<StackSnippet snippet="usetoken" />

## Next steps

This guide showed you how to sign users in with their username and password using an Okta themed sign-in page. Here are some ways to extend that functionality:

* [Customize the sign-in page that's presented by Okta](https://developer.okta.com/docs/guides/custom-widget/main/#style-the-okta-hosted-sign-in-widget)
* [Share a sign-in session with native mobile apps](https://developer.okta.com/docs/guides/shared-sso-android-ios/)
* [Add a sign-in flow using biometrics](https://developer.okta.com/docs/guides/unlock-mobile-app-with-biometrics/)
* [Protect your servers' API endpoints](https://developer.okta.com/docs/guides/protect-your-api/)

<StackSnippet snippet="specificlinks" />