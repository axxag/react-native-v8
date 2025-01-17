[![npm version](https://badge.fury.io/js/react-native-v8.svg)](https://badge.fury.io/js/react-native-v8)
[![Build for Android](https://github.com/Kudo/react-native-v8/actions/workflows/android.yml/badge.svg)](https://github.com/Kudo/react-native-v8/actions/workflows/android.yml)

# React Native with V8 Runtime

The aim of this project is to support V8 runtime for React Native. Designed as opt-in package, it should easy to integrate with existing React Native projects.

## Integration with React Native

To make RN integration easier, we publish prebuilt AAR into [npm](https://www.npmjs.com/package/react-native-v8).

The versioning is aligned with React Native with a suffix `-patch.N` in version. E.g., if your React Native version is `0.65.1`, you should use react-native-v8 `>=0.65.1-patch.0 <0.65.1`.

Following steps will take 0.65.1 as an example.

1. Install react-native-v8

```sh
yarn add 'react-native-v8@>=0.65.1-patch.0 <0.65.2'
```

2. Modify your React Native build.gradle

```diff
--- a/android/app/build.gradle
+++ b/android/app/build.gradle
@@ -161,11 +161,18 @@ android {
             }
         }
     }
+
+    packagingOptions {
+        // Make sure libjsc.so does not packed in APK
+        exclude "**/libjsc.so"
+    }
 }

 dependencies {
     implementation fileTree(dir: "libs", include: ["*.jar"])
     implementation "com.facebook.react:react-native:+"  // From node_modules
+    // Add v8-android - prebuilt libv8android.so into APK
+    implementation 'org.chromium:v8-android:9.93.+'

     // JSC from node_modules
     if (useIntlJsc) {
--- a/android/build.gradle
+++ b/android/build.gradle
@@ -24,8 +24,12 @@ allprojects {
     repositories {
         mavenLocal()
         maven {
-            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
-            url("$rootDir/../node_modules/react-native/android")
+            // Replace AAR from original RN with AAR from react-native-v8
+            url("$rootDir/../node_modules/react-native-v8/dist")
+        }
+        maven {
+            // prebuilt libv8android.so
+            url("$rootDir/../node_modules/v8-android-jit/dist")
         }
         maven {
             // Android JSC is installed from npm
```

Since V8 shared library is not ABI safe, the `implementation 'org.chromium:v8-android:9.93.+'` change above requires to pair with `react-native-v8` release version. Please reference [the mappings for the actual gradle dependency you need](#react-native-v8-and-v8-android--mappings).

3. Gradle rebuild or `react-native run-android`

4. Start application and could verify by JS `global._v8runtime().version` which expected to return V8 version.

## `react-native-v8` and `v8-android-*` mappings

| `react-native-v8` version | `v8-android*` version | gradle dependency                                 | Chromium V8 version |
| ------------------------- | --------------------- | ------------------------------------------------- | ------------------- |
| -- 0.66                   |                       |                                                   |                     |
| 0.66.3-patch.1            | 9.93.0                | implementation 'org.chromium:v8-android:`9.93.+`' | 9.3.345.16          |
| 0.66.2-patch.0            | 9.93.0                | implementation 'org.chromium:v8-android:`9.93.+`' | 9.3.345.16          |
| 0.66.1-patch.0            | 9.93.0                | implementation 'org.chromium:v8-android:`9.93.+`' | 9.3.345.16          |
| 0.66.0-patch.0            | 9.93.0                | implementation 'org.chromium:v8-android:`9.93.+`' | 9.3.345.16          |
| -- 0.65                   |                       |                                                   |                     |
| 0.65.2-patch.1            | 9.93.0                | implementation 'org.chromium:v8-android:`9.93.+`' | 9.3.345.16          |
| 0.65.2-patch.0            | 9.93.0                | implementation 'org.chromium:v8-android:`9.93.+`' | 9.3.345.16          |
| 0.65.1-patch.1            | 9.93.0                | implementation 'org.chromium:v8-android:`9.93.+`' | 9.3.345.16          |
| 0.65.1-patch.0            | 9.93.0                | implementation 'org.chromium:v8-android:`9.93.+`' | 9.3.345.16          |
| -- 0.64                   |                       |                                                   |                     |
| 0.64.2-patch.1            | 9.88.0                | implementation 'org.chromium:v8-android:`9.88.+`' | 8.8.278.15          |
| 0.64.2-patch.0            | 9.88.0                | implementation 'org.chromium:v8-android:`9.88.+`' | 8.8.278.15          |
| 0.64.1-patch.0            | 9.88.0                | implementation 'org.chromium:v8-android:`9.88.+`' | 8.8.278.15          |
| 0.64.0-patch.0            | 9.88.0                | implementation 'org.chromium:v8-android:`9.88.+`' | 8.8.278.15          |
| -- 0.63                   |                       |                                                   |                     |
| 0.63.4-patch.1            | 9.88.0                | implementation 'org.chromium:v8-android:`9.88.+`' | 8.8.278.15          |
| 0.63.4-patch.0            | 8.84.0                | implementation 'org.chromium:v8-android:`8.84.+`' | 8.4.371.19          |
| 0.63.3-patch.0            | 8.84.0                | implementation 'org.chromium:v8-android:`8.84.+`' | 8.4.371.19          |
| 0.63.2-patch.1            | 8.84.0                | implementation 'org.chromium:v8-android:`8.84.+`' | 8.4.371.19          |
| 0.63.1-patch.0            | 8.84.0                | implementation 'org.chromium:v8-android:`8.84.+`' | 8.4.371.19          |
| 0.63.0-patch.0            | 8.80.1                | implementation 'org.chromium:v8-android:`8.80.+`' | 8.0.426.16          |
| -- 0.62                   |                       |                                                   |                     |
| 0.62.2-patch.2            | 8.84.0                | implementation 'org.chromium:v8-android:`8.84.+`' | 8.4.371.19          |
| 0.62.2-patch.1            | 8.80.1                | implementation 'org.chromium:v8-android:`8.80.+`' | 8.0.426.16          |
| 0.62.1-patch.0            | 8.80.1                | implementation 'org.chromium:v8-android:`8.80.+`' | 8.0.426.16          |
| 0.62.0-patch.0            | 8.80.1                | implementation 'org.chromium:v8-android:`8.80.+`' | 8.0.426.16          |
| -- 0.59                   |                       |                                                   |                     |
| 0.59.10-patch.6           | 7.8.2                 | implementation 'org.chromium:v8-android:`7.8.+`'  | 7.8.279.23          |


## Builtin JavaScript object

`global._v8runtime()` has some builtin information such as v8 version.

```js
console.log(`V8 version is ${global._v8runtime().version}`);
```

Please note that `global._v8runtime()` existed only for V8 enabled environment but not React Native remote debugging mode.
For remote debugging mode, the JavaScript actually runs on Chrome from your host and there is no V8Runtime.

## V8 Variants

`react-native-v8` uses V8 shared libray from [v8-android-buildscripts](https://github.com/Kudo/v8-android-buildscripts).
For detailed V8 features, please check [there](https://github.com/Kudo/v8-android-buildscripts/blob/master/README.md#v8-feature-flags).

`v8-android-jit` is the default V8 variant we include in `react-native-v8`. This is a full featured V8 with both [JIT](https://en.wikipedia.org/wiki/Just-in-time_compilation) and [Intl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl). We provide other V8 variants to fulfill your needs.

| Package name          | JIT | Intl | maven path                                                   |
| --------------------- | --- | ---- | ------------------------------------------------------------ |
| v8-android-jit        | Yes | Yes  | `url("$rootDir/../node_modules/v8-android-jit/dist")`        |
| v8-android-jit-nointl | Yes | No   | `url("$rootDir/../node_modules/v8-android-jit-nointl/dist")` |
| v8-android            | No  | Yes  | `url("$rootDir/../node_modules/v8-android/dist")`            |
| v8-android-nointl     | No  | No   | `url("$rootDir/../node_modules/v8-android-nointl/dist")`     |

For instance, if you want to reduce memory usage, the non JIT variant, aka [V8 lite mode](https://v8.dev/blog/v8-lite). The following are further steps:

1. Add `v8-android`

```sh
$ yarn add v8-android
```

2. Modify the gradle dependency to use `v8-android` or other variants

```diff
--- a/android/build.gradle
+++ b/android/build.gradle
@@ -29,7 +29,7 @@ allprojects {
         }
         maven {
             // prebuilt libv8android.so
-            url("$rootDir/../node_modules/v8-android-jit/dist")
+            url("$rootDir/../node_modules/v8-android/dist")
         }
         maven {
             // Android JSC is installed from npm
```

## `react-native-v8` version should be

## iOS Support (Experimented)

We did have experimented iOS support. To adopt V8 for Xcodeproj gets a little complicated, so we have a pre-shaped template.
Please check [react-native-template-v8](packages/react-native-template-v8/README.md) for more information.

## FAQ

### How to reduce APK size ?

The V8 currently bundled by default supports [Intl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl) and the ICU data costs about 7MiB per ABI. If you are not going to use `Intl`, you could use no-Intl version to reduce APK size. (jsc-android have no Intl by default)

## TODO

- [x] Performance comparison with JavaScriptCore in React Native

  Please check https://github.com/Kudo/react-native-js-benchmark for the benchmark and result

- [x] V8 inspector integration
- [ ] V8 snapshot integration
- [ ] V8 code cache experiment
