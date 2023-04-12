---
title: Using Sentry
---

import PlatformsSection from '~/components/plugins/PlatformsSection';
import { Collapsible } from '~/ui/components/Collapsible';
import { Terminal } from '~/ui/components/Snippet';

[Sentry](http://getsentry.com/) is a crash reporting platform that provides you with "real-time insight into production deployments with info to reproduce and fix crashes".

It notifies you of exceptions or errors that your users run into while using your app, and organizes them for you on a web dashboard. Reported exceptions include stacktraces, device info, version, and other relevant context automatically; you can also provide additional context that is specific to your application, like the current route and user id.

## Why `sentry-expo`?

- Sentry treats React Native as a first-class citizen and we have collaborated with Sentry to make sure Expo is, too.
- It's very easy to set up and use
- It scales to meet the demands of even the largest projects.
- We trust it for our projects at Expo.
- It is free for up to 5,000 events per month.
- It streamlines your error-reporting code across iOS, Android, and web

> Note: Native crash reporting is not available with the classic build system (`expo build:[ios|android]`), but is available via EAS Build.

<PlatformsSection title="Platform compatibility" android emulator ios simulator web />

## How to add Sentry to your Expo project

### Step 0: Sign up for a Sentry account and create a project

Before getting real-time updates on errors and making your app generally incredible, you'll need to make sure you've created a Sentry project. Here's how to do that:

1. [Sign up for Sentry](https://sentry.io/signup/) (it's free), and create a project in your Dashboard. Take note of your **organization name**, **project name**, and **`DSN`**; you'll need them later.

   - **organization name** is available in your `Organization settings` tab
   - **project name** is available in your project's `Settings` > `Projects` tab (find it in the list)
   - **`DSN`** is avalable in your project's `Settings` > `Projects` > **Project name** > `Client Keys (DSN)` tab

2. Go to the [Sentry API section](https://sentry.io/settings/account/api/auth-tokens/), and create an **auth token**. The token requires the scopes: `org:read`, `project:releases`, and `project:write`. Save this, too.

Once you have each of these: organization name, project name, DSN, and auth token, you're all set!

### Step 1: Installation

In your project directory, run:

<Terminal cmd={['$ expo install sentry-expo']} />

> If you're using SDK 39 or lower, run `yarn add sentry-expo@~3.0.0`

`sentry-expo` also requires some additional dependencies, otherwise it won't work properly. To install them, run:

<Terminal 
  cmd={['$ expo install expo-application expo-constants expo-device expo-updates @sentry/react-native']}
/>

### Step 2: Code

Add the following to your app's main file (usually `App.js`):

```js
import * as Sentry from 'sentry-expo';

Sentry.init({
  dsn: 'YOUR DSN HERE',
  enableInExpoDevelopment: true,
  debug: true, // If `true`, Sentry will try to print out useful debugging information if something goes wrong with sending the event. Set it to `false` in production
});

// Access any @sentry/react-native exports via:
Sentry.Native.*

// Access any @sentry/browser exports via:
Sentry.Browser.*
```

### Step 3: App Config

#### Configure your `postPublish` hook

Add `expo.hooks` to your project's `app.json` (or `app.config.js`) file:

```json
{
  "expo": {
    // ... your existing configuration
    "hooks": {
      "postPublish": [
        {
          "file": "sentry-expo/upload-sourcemaps",
          "config": {
            "organization": "your sentry organization's short name here",
            "project": "your sentry project's name here",
            "authToken": "your auth token here"
          }
        }
      ]
    }
  }
}
```

The correct `authToken` value can be generated from the [Sentry API page ](https://sentry.io/settings/account/api/).

> You can also use environment variables for your config, if you prefer:
>
> - organization = SENTRY_ORG
> - project = SENTRY_PROJECT
> - authToken = SENTRY_AUTH_TOKEN
>
> You can pass them in directly like this:
>
> ```
> SENTRY_PROJECT=myCoolProject expo publish
> ```

<Collapsible summary="Additional configuration options">

In addition to the required config fields above, you can also provide these **optional** fields:

- `setCommits` : boolean value indicating whether or not to tell Sentry about which commits are associated with a new release. This allows Sentry to pinpoint which commits likely caused an issue.
- `deployEnv` : string indicating the deploy environment. This will automatically send an email to Sentry users who have committed to the release that is being deployed.
- `distribution` : The name/value to give your distribution (you can think of this as a sub-release). Expo defaults to using your `version` from app.json. **If you provide a custom `distribution`, you must pass the same value to `dist` in your call to `Sentry.init()`, otherwise you will not see stacktraces in your error reports.**
- `release` : The name you'd like to give your release (e.g. `release-feature-ABC`). This defaults to a unique `revisionId` of your JS bundle. **If you provide a custom `release`, you must pass in the same `release` value to `Sentry.init()`, otherwise you will not see stacktraces in your error reports.**
- `url` : your Sentry URL, only necessary when self-hosting Sentry.

> You can also use environment variables for your config, if you prefer:
>
> - setCommits = SENTRY_SET_COMMITS
> - deployEnv = SENTRY_DEPLOY_ENV
> - distribution = SENTRY_DIST
> - release = SENTRY_RELEASE
> - url = SENTRY_URL

</Collapsible>

#### Add the Config Plugin

> Note: Disregard the following if you're using the classic build system (`expo build:[android|ios]`).

Add `expo.plugins` to your project's `app.json` (or `app.config.js`) file:

```json
{
  "expo": {
    // ... your existing configuration
    "plugins": ["sentry-expo"]
  }
}
```

If you directly edit your native `ios/` and `android/` directories (i.e. you have ejected your project, or have a bare workflow project), **you should not use the above `plugins` property**. Instead, use `yarn sentry-wizard -i reactNative -p ios android` to configure your native projects. This `sentry-wizard` command will add an extra:

```js
import * as Sentry from '@sentry/react-native';

Sentry.init({
  dsn: 'YOUR DSN',
});
```

to your root project file (usually **App.js**), so make sure you remove it (but keep the `sentry-expo` import and original `Sentry.init` call!)

## Sourcemaps

With the `postPublish` hook in place, now all you need to do is run `expo publish` and the sourcemaps will be uploaded automatically. We automatically assign a unique release version for Sentry each time you hit publish, based on the version you specify in **app.json** and a release id on our backend -- this means that if you forget to update the version but hit publish, you will still get a unique Sentry release. If you're not familiar with publishing on Expo, you can [read more about it here](../workflow/publishing.md).

> This hook can also be used as a `postExport` hook if you're [self-hosting your updates](../distribution/hosting-your-app.md).

### "No publish builds"

> Note: Disregard the following if you're using the classic build system (`expo build:[android|ios]`).

With `expo-updates`, release builds of both iOS and Android apps will create and embed a new update from your JavaScript source at build-time. **This new update will not be published automatically** and will exist only in the binary with which it was bundled. Since it isn't published, the sourcemaps aren't uploaded in the usual way like they are when you run `expo publish` (actually, we are relying on Sentry's native scripts to handle that). Because of this you have some extra things to be aware of:

- Your `release` will automatically be set to Sentry's expected value- `${bundleIdentifier}@${version}+${buildNumber}` (iOS) or `${androidPackage}@${version}+${versionCode}` (Android).
- Your `dist` will automatically be set to Sentry's expected value- `${buildNumber}` (iOS) or `${versionCode}` (Android).
- The configuration for build time sourcemaps comes from the `ios/sentry.properties` and `android/sentry.properties` files. For more information, refer to [Sentry's documentation](https://docs.sentry.io/clients/java/config/#configuration-via-properties-file). If you're using the managed workflow, then we handle all of this setup for you via the [`plugin` you added above](#add-the-config-plugin).

> Please note that configuration for `expo publish` and `expo export` in bare and managed is still done via `app.json`.

Skipping or misconfiguring either of these will result in sourcemaps not working, and thus you won't see proper stacktraces in your errors.

### Self-hosting updates?

If you're self-hosting your updates (this means you run `expo export` instead of `expo publish`), you need to:

- replace `hooks.postPublish` in your **app.json** file with `hooks.postExport` (everything else stays the same)
- add the `RewriteFrames` integration to your `Sentry.init` call like so:

```js
Sentry.init({
  dsn: SENTRY_DSN,
  enableInExpoDevelopment: true,
  integrations: [
    new RewriteFrames({
      iteratee: frame => {
        if (frame.filename) {
          // the values depend on what names you give the bundle files you are uploading to Sentry
          frame.filename =
            Platform.OS === 'android' ? 'app:///index.android.bundle' : 'app:///main.jsbundle';
        }
        return frame;
      },
    }),
  ],
});
```

### Testing Sentry

When building tests for your application, you want to assert that the right flow-tracking or error is being sent to Sentry, but without really sending it to Sentry servers. This way you won't swamp Sentry with false reports during test running and other CI operations.

[`sentry-testkit`](https://zivl.github.io/sentry-testkit) enables Sentry to work natively in your application, and by overriding the default Sentry transport mechanism, the report is not really sent but rather logged locally into memory. In this way, the logged reports can be fetched later for your own usage, verification, or any other use you may have in your local developing/testing environment.

See how to get started with `sentry-testkit` in their [documentation site here](https://zivl.github.io/sentry-testkit/)

> If you're using `Jest`, make sure to add `@sentry/.*` and `sentry-expo` to your `transformIgnorePatterns`.

## Error reporting semantics

In order to ensure that errors are reported reliably, Sentry defers reporting the data to their backend until the next time you load the app after a fatal error rather than trying to report it upon catching the exception. It saves the stacktrace and other metadata to `AsyncStorage` and sends it immediately when the app starts.

## Disabled by default in dev

Unless `enableInExpoDevelopment: true` is set, all your dev/local errors will be ignored and only app releases will report errors to Sentry. You can call methods like `Sentry.Native.captureException(new Error('Oops!'))` but these methods will be no-op.

## Learn more about Sentry

Sentry does more than just catch fatal errors, learn more about how to use Sentry from their [JavaScript usage docs](https://docs.sentry.io/platforms/javascript/).
