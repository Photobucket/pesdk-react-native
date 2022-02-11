<p align="center">
  <a href="https://img.ly/photo-sdk?utm_campaign=Projects&utm_source=Github&utm_medium=PESDK&utm_content=React-Native">
    <img src="https://img.ly/static/logos/PE.SDK_Logo.svg" alt="PhotoEditor SDK Logo"/>
  </a>
</p>
<p align="center">
  <a href="https://npmjs.org/package/react-native-photoeditorsdk">
    <img src="https://img.shields.io/npm/v/react-native-photoeditorsdk.svg" alt="NPM version">
  </a>
  <a href="https://npmjs.org/package/react-native-photoeditorsdk">
    <img src="https://img.shields.io/badge/platforms-android%20|%20ios-lightgrey.svg" alt="Platform support">
  </a>
  <a href="http://twitter.com/PhotoEditorSDK">
    <img src="https://img.shields.io/badge/twitter-@PhotoEditorSDK-blue.svg?style=flat" alt="Twitter">
  </a>
</p>

# React Native module for PhotoEditor SDK

Check out our [video tutorial](https://img.ly/blog/a-photo-and-video-editor-for-your-react-native-apps/) for a step-by-step integration guide which also details advanced SDK features, such as serializing and reusing previously applied editing operations.

## Getting started

### Expo CLI

#### Limitations

This module can **not** be used in the `Expo Go` application because it uses custom native libraries.

#### Usage

In order to use this module with the Expo CLI you can make use of our integrated Expo config plugin:

1. Add our module to your Expo application:

   ```sh
   expo install react-native-photoeditorsdk
   ```
   
   This will automatically install [`react-native-imglysdk`](https://npmjs.org/package/react-native-imglysdk) which you can use to configure your application with our Expo config plugin.

2. Inside your app's `app.json` or `app.config.js` add our config plugin:

   ```json
   {
     "plugins": ["react-native-imglysdk"]
   }
   ```

   If needed, you can also use a specific version of our native library for Android as well as define explicitly the included modules. By default, all modules for both PhotoEditor SDK and VideoEditor SDK are included.

   ```json
   {
     "plugins": [
       [
         "react-native-imglysdk",
         {
           "android": {
             "version": "9.2.0",
             "modules": [
               "ui:core",
               "ui:transform",
               "ui:filter",
               "assets:filter-basic"
             ]
           }
         }
       ]
     ]
   }
   ```

   For further information on the available modules, please refer to step 4 of the React Native CLI [Android](#android) guide below.
   
   **Please note that the `react-native-imglysdk` module manages both VideoEditor SDK as well as PhotoEditor SDK so you only need to add the Expo config plugin once even when using both SDKs.**

3. The changes will be applied on `expo prebuild` or during the prebuild phase of `eas build`.

For further information on how to integrate Expo config plugins please also refer to the official [docs](https://docs.expo.dev/guides/config-plugins/#using-a-plugin-in-your-app).

### React Native CLI

Install the React Native module in your project as follows:

```sh
yarn add react-native-photoeditorsdk
```

In general, [we highly recommend using React Native 0.60 or newer](https://img.ly/blog/react-native-native-modules-made-for-react-developers-59ca93c41541/). If you cannot avoid using an older React Native version you need to [link the native dependencies](https://facebook.github.io/react-native/docs/0.59/linking-libraries-ios#step-2) with:

```sh
yarn react-native link
```

before you continue with the platform-specific guides below.

#### iOS

For React Native 0.60 and newer [autolinking](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md) is used and PhotoEditor SDK for iOS should be automatically installed:

```sh
cd ios && pod install && cd ..
```

and updated:

```sh
cd ios && pod update && cd ..
```

with CocoaPods.

For older React Native versions autolinking is not available and PhotoEditor SDK for iOS needs to be [manually integrated](https://img.ly/docs/pesdk/ios/introduction/getting_started/#manually) in your Xcode project if you don't use [CocoaPods to manage your dependencies](https://facebook.github.io/react-native/docs/0.59/integration-with-existing-apps#configuring-cocoapods-dependencies). Make sure to put `ImglyKit.framework` and `PhotoEditorSDK.framework` in the `ios/` directory of your project.

#### Android

1. Because PhotoEditor SDK for Android is quite large, there is a high chance that you will need to enable [Multidex](https://developer.android.com/studio/build/multidex) for your project as follows:

   1. Open the `android/app/build.gradle` file (**not** `android/build.gradle`) and add these lines at the end:
      ```groovy
      android {
          defaultConfig {
              multiDexEnabled true
          }
      }
      dependencies {
          implementation 'androidx.multidex:multidex:2.0.1'
      }
      ```
   2. Open the `android/app/src/main/java/.../MainApplication.java` file and change the superclass of your `MainApplication` class from `Application` to `androidx.multidex.MultiDexApplication`, e.g.:
      ```java
      public class MainApplication extends androidx.multidex.MultiDexApplication implements ReactApplication {
      ```

2. Add the img.ly repository and plugin by opening the `android/build.gradle` file (**not** `android/app/build.gradle`) and adding these lines at the top:
   ```groovy
   buildscript {
       repositories {
           mavenCentral()
           maven { url "https://artifactory.img.ly/artifactory/imgly" }
       }
       dependencies {
           classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.4.10"
           classpath 'ly.img.android.sdk:plugin:9.2.0'
       }
   }
   ```
   In order to update PhotoEditor SDK for Android replace the version string `9.2.0` with a [newer release](https://github.com/imgly/pesdk-android-demo/releases).

3. Still in the `android/build.gradle` file (**not** `android/app/build.gradle`), add these lines at the bottom:

   ```groovy
   allprojects {
       repositories {
           maven { url 'https://artifactory.img.ly/artifactory/imgly' }
       }
   }
   ```

4. Configure PhotoEditor SDK for Android by opening the `android/app/build.gradle` file  (**not** `android/build.gradle`) and adding the following lines under `apply plugin: "com.android.application"`:
   ```groovy
   apply plugin: 'ly.img.android.sdk'
   apply plugin: 'kotlin-android'

   // Comment out the modules you don't need, to save size.
   imglyConfig {
       modules {
           include 'ui:text'
           include 'ui:focus'
           include 'ui:frame'
           include 'ui:brush'
           include 'ui:filter'
           include 'ui:sticker'
           include 'ui:overlay'
           include 'ui:transform'
           include 'ui:adjustment'
           include 'ui:text-design'

           // This module is big, remove the serializer if you don't need that feature.
           include 'backend:serializer'

           // Remove the asset packs you don't need, these are also big in size.
           include 'assets:font-basic'
           include 'assets:frame-basic'
           include 'assets:filter-basic'
           include 'assets:overlay-basic'
           include 'assets:sticker-shapes'
           include 'assets:sticker-emoticons'

           include 'backend:sticker-smart'
       }
   }
   ```

### Usage

Import the module in your `App.js`:

```js
import {PESDK, PhotoEditorModal, Configuration} from 'react-native-photoeditorsdk';
```

Each platform requires a separate license file. [Unlock PhotoEditor SDK](./index.d.ts#L41-L53) with a single line of code for both platforms via [platform-specific file extensions](https://reactnative.dev/docs/platform-specific-code#platform-specific-extensions):

```js
PESDK.unlockWithLicense(require('./pesdk_license'));
```

Open the editor with an image:

```js
PESDK.openEditor(require('./image.jpg'));
```

Or use the component to open the editor:

```jsx
<PhotoEditorModal visible={true} image={require('./image.jpg')}/>
```

Please see the [code documentation](./index.d.ts) for more details and additional [customization and configuration options](./configuration.ts).

For configuring and customizing PhotoEditor SDK beyond these options exposed to JavaScript the iOS bridge provides an [interface for native customization](./ios/RNPhotoEditorSDK.h). Please refer to [our documentation](https://img.ly/docs/pesdk?utm_campaign=Projects&utm_source=Github&utm_medium=PESDK&utm_content=React-Native) for more details on native customization.

## Example

Please see our [example project](https://github.com/imgly/pesdk-react-native-demo) which demonstrates how to use the React Native module for PhotoEditor SDK.

## License Terms

Make sure you have a [commercial license](https://img.ly/pricing?utm_campaign=Projects&utm_source=Github&utm_medium=PESDK&utm_content=React-Native) for PhotoEditor SDK before releasing your app.
A commercial license is required for any app or service that has any form of monetization: This includes free apps with in-app purchases or ad supported applications. Please contact us if you want to purchase the commercial license.

## Support and License

Use our [service desk](https://support.img.ly) for bug reports or support requests. To request a commercial license, please use the [license request form](https://img.ly/pricing?utm_campaign=Projects&utm_source=Github&utm_medium=PESDK&utm_content=React-Native) on our website.
