# Automated E2E tests using Jest and Appium with a React Native app that was ejected

Welcome. This article may be good for a short time as I have solved problems probably related to unstable version of Expo.

This article is the results of my work trying to get Jest/Appium to run tests in Android emulator in my Mac for a React Native app.

## Lets get Expo to Eject and run in the device

### Basic environment

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

### Using the expo-cli to get started with a basic React Native app

```
npm install expo-cli -g
```

[Expo instalation](https://docs.expo.io/versions/latest/introduction/installation/)

### Create your MyTestApp using expo init

```
expo init MyTestApp  
```

And eject it

```
expo eject
```

When expo asked "What should your Android Studio and Xcode projects be called?" I answered 'MyTestApp'.

### Installing extra needed modules

At this point, it seems that some modules were missing when ejecting. I had simply followed errors and got it to work installing:

```
npm i @babel/runtime --save-dev
npm i expo --save-dev
```

### Fixing React Native error

Had to patch React Native line in the packages.json from:

```
"react-native": "https://github.com/expo/react-native/archive/sdk-32.0.0.tar.gz"
```

To

```
"react-native": "0.57.8"
```

Make sure you install after you manually edited:

```
npm install
```

[1](https://github.com/facebook/react-native/issues/22033)

### Updating to React latest

Since the generating the app failed [1](https://github.com/facebook/react/issues/14341) I end up fixing with :

```
npm i react@latest --save
```

### Patching index.js

Expo also is missing index.js generation, so I had to create manually. Notice that the name of the app needs to be the same as the name of your working directory. In my case "MyTestApp".

```
import { AppRegistry } from 'react-native';
import App from './App';
AppRegistry.registerComponent('MyTestApp', () => App);

```  

### Testing the app in the emulator

Bring your emulator. In the Mac OS X I had trouble to launch and had to execute it from the emulator directory:

```
cd ~taboca/Library/Android/sdk/emulators
./emulator -list-avds
./emulator @test #test is name of an emulator in my list
```

Assuming you brought the Android, in the emulator, up; then proceed:

```
npm start&
npm run android
```

If the app appears then you are okay. CTRL+C to the Metro process (the above npm start) and let's move to the Jest/Appium part.

## Part 2 - To hook up Jest

npm install jest --save-dev

### package.json

"jest": {
    "preset": "react-native"
  }

and

  "scripts": {
    "test": "jest",
  },

You may want to test with "npm start&" then later "npm run test" just to find out that it worked while it claims, with an error, that no tests found.

### installing Appium and Appium Doctor

npm install --save-dev  appium appium-doctor

### Check Appium dependencies with Appium-doctor:

First, let's patch scripts in packages.json

```
"scripts": {
  "start": "react-native start",
  "test": "jest",
  "appium": "appium",
  "appium:doctor": "appium-doctor",
  "android": "react-native run-android",
  "ios": "react-native run-ios"
},

```

Now you can use npm to run the checks:

```
npm run appium:doctor
```

As you run the above, it may generate some warnings requesting you to perform actions, such as the necessary environment variables (JAVA_HOME, ANDROID_HOME, etc). According to [E2E with Jest and Appium](https://medium.com/front-end-hacking/how-to-do-end-to-end-e2e-testing-for-react-native-android-using-jest-and-appium-27d75e4d831b), since we are working with Android, we can safely ignore the warning "WARN AppiumDoctor ✖ Carthage was NOT found!".

### Installing the Web Driver

The web driver will be used by Appium.
```
npm install --save-dev wd
```

### Insert script for tests

```
mkdir __tests__
```

And config your initial test file:

```
import wd from 'wd';

jasmine.DEFAULT_TIMEOUT_INTERVAL = 60000;
const PORT = 4723;
const config = {
  platformName: 'Android',
  deviceName: 'test',
  app: './android/app/build/outputs/apk/debug/app-debug.apk' // relative to root of project

};
const driver = wd.promiseChainRemote('localhost', PORT);

beforeAll(async () => {
  await driver.init(config);
  await driver.sleep(20000); // wait for app to load - if you keep this too short you may get problems.
})

test('appium renders', async () => {
  expect(await driver.hasElementByAccessibilityId('testview')).toBe(true);
  expect(await driver.hasElementByAccessibilityId('notthere')).toBe(false);
});


```

Notes: make sure you check the config part so deviceName matches the emulator name and app points to the directory of the apk under android.


### Edit App.js

Notice that the above testing code expects "testview" in the DOM for the React Native app. So you will need to add it. Edit App.js and put the following attribute in some node:

```
accessibilityLabel="testview"
```

Example

```
<View style={styles.container} accessibilityLabel="testview">
  <Text>Open up App.js to start working on your app!</Text>
</View>
```

### Run Appium and the Metro interface

npm run appium&
npm run start&


[1](https://github.com/taboca/doc-js-example-create-react-native-and-e2e-jest-appium)

### Test

```
npm run test
```

If you run into trouble please write, let's help each other.

## References

* https://github.com/taboca/doc-js-example-create-react-native-and-e2e-jest-appium

* https://medium.com/front-end-weekly/how-to-do-end-to-end-e2e-testing-for-react-native-android-using-jest-and-appium-27d75e4d831b
