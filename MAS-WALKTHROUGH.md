## Electron Mac App Store Walk-Through

Submitting an Electron application to the Mac App Store (MAS) has been made as simple as possible using three packaging tools: [electron-packager](https://github.com/maxogden/electron-packager), [electron-osx-sign](https://github.com/sethlu/electron-osx-sign) and electron-osx-flat (included with electron-osx-sign). 

[sethlu's fork of electron-packager](https://github.com/sethlu/electron-packager) greatly simplifies the process because it downloads the necessary "MAS Build" of Electron instead of having to obtain and override your system-installed version. This fork is [expected to be merged](https://github.com/maxogden/electron-packager/pull/223) soon into the main distribution any day, but at the moment you must explicitly install the fork.

#### Prerequisites

You must be a registered member of the App [Apple Developer Program](https://developer.apple.com/).

* You must have [XCode](https://developer.apple.com/xcode/) installed.
* You must create an OSX app on [iTunes Connect](https://itunesconnect.apple.com). For this walkthrough we will use the name "My App" as the application name.
* You must give your app a unique Bundle Id. For this walkthrough, we will use the Bundle ID "com.mysite.myapp"
* You must give your app a Version Number. For this walkthrough, we will use the Version Number "1.0.0"
* An icon file named "Icon.icns" somewhere in your source path. The Apple utility [iconutil](https://developer.apple.com/library/mac/documentation/GraphicsAnimation/Conceptual/HighResolutionOSX/Optimizing/Optimizing.html) can be used to create the icon files for OSX applications.
* In the Developer Member Center you must create two Production/Distribution Certificates. The first is a `Production > Mac App Store > Mac App Distribution` certificate. The second is a `Production > Mac App Store > Mac Installer Distribution` certificate. After you create these two certifications, download them and double-click them so that they are installed in your Keychain.

#### Install The Tools

We will be installing these tools globally (-g). You may install them locally for your project however if you do you may need to adjust paths.

```
# install sethlu's fork of electron-packager 
npm install https://github.com/sethlu/electron-packager/tarball/master -g

# the official dist of electron-osx-sign is ok
npm install electron-osx-sign -g
```

#### Build The MAS App

The first step is to compile your application into an .app bundle. Notice that the --platform which you would normally expect to be "darwin" is instead "mas" to indicate this build is for the Mac App Store. Currenly only sethlu's fork supports the mas platform. The difference is that MAS builds ommit certain libraries and private API calls that will cause the electron app to be rejected.

```
electron-packager . "My App" --app-bundle-id=com.mysite.myapp --helper-bundle-id=com.mysite.myapp.helper --app-version=1.0.0 --build-version=1.0.100 --platform=mas --arch=x64 --version=0.36.7 --ignore="node_modules/electron-*" --icon=path/to/your/Icon.icns --overwrite
```

###### Parameter Details:

*The arguments are all documented, however below are additional details that apply specifically to building apps for the Mac App Store.*

* **--app-bundle-id** must match your Bundle ID that you assigned in iTunes Connect.
* **--helper-bundle-id** is the same as your app-bundle-id with ".helper" prepended. Unless you have overridden the defaults for the helper bundle this value will be fine.
* **--app-version** must match your Version Number in iTunes Connect.
* **--build-version** the first two numbers must match the app-version (1.0.x). The third number can be any number that is larger than app-version's third number. It must be a unique number each time you submit a new build. For example, the first time submit a binary it is 1.0.100. If you make changes and submit another build, the third number must be incremented by at least 1 number, for example 1.0.101. You must increment the third number for each build that you submit.
* **--version** refers to the version of Electron. At the time of this writing 0.36.7 is the most recent.
* **--ignore** is a regex of files that you do not wish to package into your app bundle.
* **--icon** is the path to the Icon.icns file that you created for your application.

#### Sign The App Bundle

If you have only one set of production certifications from iTunes Connect and your app does not have any special entitlements, then you can sign your app, allowing electron-osx-sign to automatically detect your certificates.

```
electron-osx-sign "My App-mas-x64/My App.app" --verbose
```

###### If you have packaged any additional binaries in your app:

If your app includes any additional binary files (command-line applications for example) then those binaries also need to be signed. You can specify additional binaries after the path to your .app bundle, but before the arguments:

```
electron-osx-sign "My App-mas-x64/My App.app" "My App-mas-x64/My App.app/Contents/Resources/app/bin/mybinary" --verbose
```

###### If you have any custom entitlements:

If your app requires any custom entitlements such as file access, network access, etc then you must create your own parent.plist and child.plist entitlement files. You then specify the path to those plist files. The "parent" is the entitlement used to sign the entire app bundle. The "child" is the entitlement used to sign the frameworks and binaries inside the bundle:

```
electron-osx-sign "My App-mas-x64/My App.app" --entitlements=path/to/parent.plist --entitlements-inherit=path/to/child.plist --verbose
```

###### If you have multiple developer identities in your keychain:

electron-osx-sign searches your keychain for the first signing certificates that it can locate. If you have multiple certificates then it may not know which cert you want to use for signing and you need to explicitly provide the name:

```
electron-osx-sign "My App-mas-x64/My App.app" --identity="3rd Party Mac Developer Application: My Company, Inc (ABCDEFG1234)" --verbose
```

#### Create The Signed Installer Package

The final build step is to "flatten" your app bundle into an installer package. This .pkg file is what you will submit to the app store.

If you only have one signing certification then the following command will produce a signed installer package:

```
electron-osx-flat "My App-mas-x64/My App.app" --verbose
```

###### If you have multiple developer identities in your keychain:

Similar to osx-sign, osx-flat will auto-detect your signing certificate path. If you have multiple certificates then you will need to specify it:

```
electron-osx-flat "My App-mas-x64/My App.app" --identity="3rd Party Mac Developer Installer: My Company, Inc (ABCDEFG1234)" --verbose
```

#### Submit Your Installer Package to the App Store

You should now have a signed .pkg file in the same directory as your .app file. This is your installer package.

*NOTE: It is important that you do not manually rename, or make changes of any kind to these files after signing them. If you need to rename files, do it before you sign them.*

To submit your signed app you must use the [Application Loader](https://developer.apple.com/library/ios/documentation/LanguagesUtilities/Conceptual/iTunesConnect_Guide/Chapters/UploadingBinariesforanApp.html) which is included with Xcode. You can open it using Spotlight, or by opening Xcode and going to the menu: Xcode > Open Developer Tools > Application Loader.

###### App Submission

Application Loader is simple to use. After you login using your Developer Program credentials, double click the "Deliver Your App" button. This will open a file chooser dialog which allows you to select your .pkg file that you created.

###### App Validation

Application Loader will first search your iTunes Connect account for a matching application based on the Bundle ID and Version Number. If a suitable application is found then it will display and allow you to continue.

After you choose to continue, Application Loader will perform validation on your application to ensure that everything is signed properly. If there are any problems they will be displayed for you to correct. Otherwise your binary will be submitted and ready for you to submit for review.

###### Congratulations!

You app is now submitted. Proceed to iTunes connect and ensure that all of the required information about your app has been completed. Your binaries will appear under the "Activity" submenu of iTunes and you will be able to click on the binary to "Select" it as the one you wish to release.

At this point you are ready to submit your app for review. With luck it will be approved and ready for sale!

- - -