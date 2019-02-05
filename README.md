# Automated E2E tests using Jest and Appium with a React Native app that was ejected

Welcome. This article is the results of my work trying to get Jest/Appium to run tests in Android emulator in my Mac for a React Native app.

## Basic environment

The following process will involving building your app thus depends on Java SDK, Android SDK, NodeJS, nvm, npm.

```
nvm use 10.15.1
```

Java and Android SDK

```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home
export PATH=/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home/bin:$PATH
export ANDROID_HOME=${HOME}/Library/Android/sdk
export PATH=${PATH}:${ANDROID_HOME}/tools
export PATH=${PATH}:${ANDROID_HOME}/platform-tools
export ANDROID_SDK_ROOT=/Users/taboca/Library/Android/sdk
```

## Using the expo-cli to get started with a basic React Native app

```
npm install expo-cli -g
```

[Expo instalation](https://docs.expo.io/versions/latest/introduction/installation/)
