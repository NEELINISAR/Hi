## Termux Libraries

### Contents
- [Importing Libraries](#importing-libraries)
	- [Importing Libraries From Jitpack](#jitpack)
	- [Importing Libraries From Github](#github)
- [Target SDK `30` Package Visibility](#target-sdk-30-package-visibility)
- [Forking and Local Development](#forking-and-local-development)
##



### Importing Libraries

The [`termux-app`] repository publishes `3` libraries that are used by Termux plugin apps and can also be used by 3rd party apps as long as they agree to the terms of the relevant [Licenses](https://github.com/termux/termux-app/blob/master/LICENSE.md). The versions of `termux-app` and all libraries are updated together.

- [`termux-shared`]
- [`terminal-view`]
- [`terminal-emulator`]

Termux plugins and 3rd party apps only need to import `termux-shared` library for interacting with the `termux-app`, like for using [`TermuxConstants`] and all other utilities provided by it.

However, if the app needs to use the terminal provided by `termux-app`, then import `terminal-view` library as well. The `terminal-emulator` library will be automatically imported since its a dependency of `terminal-view`. The `termux-shared` library also depends on `terminal-view` and `terminal-emulator` libraries, but an explicit additional import of `terminal-view` is necessary if using the terminal.

#### Jitpack

The libraries are being published on https://jitpack.io for [`termux-app`] version `>= 0.116`.

For importing the libraries in your project, you need to follow the following instructions.

1. Add to `root` level `build.gradle` file.

```
allprojects {
    repositories {
    	...
        //mavenLocal()
        maven { url "https://jitpack.io" }
    }
}
```

2. Add to `app` module level `build.gradle` file if you want to import `termux-shared`.

```
dependencies {
    implementation 'com.termux:termux-shared:0.117'
}
```

3. Run `Sync Gradle with Project Files` in Android Studio in `File` drop down menu or `Sync Now` popup button that may appear at the top of `build.gradle` file.

Note that the first time someone imports a new version, it will take some time to build it on jitpack servers and you may get errors like `Unable to resolve dependency for ':app@debug/compileClasspath'`. You can check or trigger builds and view their logs at https://jitpack.io/#termux/termux-app and the API at e.g https://jitpack.io/api/resolve/com.termux/termux-app/master-SNAPSHOT. It will take a few minutes for builds to be downloadable even after `build.log` shows them to have been succeeded, run `Sync Project with Gradle Files` again to try to re-download.

When [`termux-app`] publishes [github releases](https://github.com/termux/termux-app/releases), a build on jitpack servers is automatically triggered via the [
`Trigger Termux Library Builds on Jitpack
`](https://github.com/termux/termux-app/actions/workflows/trigger_library_builds_on_jitpack.yml) workflow so that imports are instant for that release version when someone tries to import them for the first time. The version build triggered will be for without the `v` prefix, like `0.117`. A version with the `v` prefix like `v0.117`, triggers a separate build on jitpack.

Note that you can use `com.termux` instead of `com.github.termux` as the [`groupId`](https://maven.apache.org/pom.html#maven-coordinates) since https://termux.com has a `DNS TXT` record from `git.termux.com` to https://github.com/termux as per jitpack [custom domain requirements](https://jitpack.io/docs/#custom-domain-name). 

You can confirm by running `dig txt git.termux.com` and should get something like following in the output.

```
;; ANSWER SECTION:
git.termux.com.		300	IN	TXT	"https://github.com/termux"
```

##### Example imports

Check https://github.com/jitpack/jitpack.io#building-with-jitpack for details, like including commit or branch level import.

##### [`termux-shared`]

- `implementation 'com.termux.termux-app:termux-shared:0.117'`
- `implementation 'com.termux.termux-app:termux-shared:master-SNAPSHOT'`
- `implementation 'com.termux.termux-app:termux-shared:9272a757af'`
- `implementation 'com.github.termux.termux-app:termux-shared:0.117'`

##### [`terminal-view`]

- `implementation 'com.termux.termux-app:terminal-view:0.117'`
##



#### Github Packages

The Termux libraries were [published](https://github.com/orgs/termux/packages) on [Github Packages](https://docs.github.com/en/packages/learn-github-packages/introduction-to-github-packages) for versions `0.109-0.114` but has [**since been stopped**](https://github.com/termux/termux-app/commit/b7b12ebe). Github Package hosting is considered a private repository since it requires github API keys if a hosted library needs to be imported as a dependency. Importing from private repositories is not allowed as per [`F-Droid` policy](https://f-droid.org/en/docs/FAQ_-_App_Developers/#which-libraries-and-dependencies-are-good-to-use) so Termux plugin apps can't import Termux libraries as dependencies when building on `F-Droid`, so we moved to [Jitpack](#jitpack) publishing.

There are *plans* to continue publishing on Github Packages as well in future, but requires modifying `build.gradle` `maven-publish` tasks to publish both on Github and Jitpack and also support local publishing. This should also allow comparisons of [`Github`](https://github.com/termux/termux-app#github) and [`F-Droid`](https://github.com/termux/termux-app#f-droid) releases against each other for security reasons (software supply chain attacks), assuming any [problems of reproducible builds](https://f-droid.org/en/docs/Reproducible_Builds/) are solved.


For importing the libraries in your project, you need to follow the following instructions.

1. Create a Github access `token` to download packages. You can create it from your Github account from `Settings` -> `Developer settings` -> `Personal access tokens` -> `Generate new token`. You must enable the `read:packages` scope when creating the token. You can get more details at [AndroidLibraryForGitHubPackagesDemo](https://github.com/enefce/AndroidLibraryForGitHubPackagesDemo).

2. Create `github.properties` file in project `root` directory, and set your username and token. Also optionally add the `github.properties
` entry to `.gitignore` file so that your `token` doesn't accidentally get added to `git`, most of the Termux apps already have it ignored.

```
GH_USERNAME=<username>
GH_TOKEN=<token>
```

3. Add to `app` module level `build.gradle `if you want to import `termux-shared`.

```
def githubProperties = new Properties()
githubProperties.load(new FileInputStream(rootProject.file("github.properties")))

dependencies {
    implementation 'com.termux:termux-shared:0.114'
}

repositories {
    maven {
        name = "GitHubPackages"
        url = uri("https://maven.pkg.github.com/termux/termux-app")

        credentials {
            username = githubProperties['GH_USERNAME'] ?: System.getenv("GH_USERNAME")
            password = githubProperties['GH_TOKEN'] ?: System.getenv("GH_TOKEN")
        }
    }
}
```
##



### Target SDK `30` Package Visibility

If your third-party app has set `targetSdkVersion` to `>=30` (android `>= 11`), then it needs to add `com.termux` package to the `queries` element or request `QUERY_ALL_PACKAGES` permission in its `AndroidManifest.xml`. Otherwise it will get `PackageSetting{...... com.termux/......} BLOCKED` errors in `logcat` when interacting with [`termux-app`], like with [`RUN_COMMAND` intent](https://github.com/termux/termux-app/wiki/RUN_COMMAND-Intent) and it will **not work**.

```
<manifest
    <queries>
        <package android:name="com.termux" />
   </queries>
</manifest>
```

Check [package-visibility](https://developer.android.com/training/basics/intents/package-visibility#package-name), [`QUERY_ALL_PACKAGES` googleplay policy](https://support.google.com/googleplay/android-developer/answer/10158779) and this [article](https://medium.com/androiddevelopers/working-with-package-visibility-dc252829de2d) for more info.
##



### Forking and Local Development

If you are [forking](https://github.com/termux/termux-app#forking) the `termux-app` plugins, then you will need to import your own modified version of the [`termux-shared`] library in which you updated the package name, etc in the `TermuxConstants` class. By default, the `build.gradle` file of plugin apps will import the libraries published for `com.termux` package name and you can't use them in your forked plugins. You can either publish your modified `termux-shared` library via local maven publishing to build your apps or publishing your libraries on Jitpack or Github and import them instead, i.e with your `groupId` `com.github.<user>`.

If you are a maintainer/contributor of the `termux-app` plugins and need to update the `termux-shared` or the other libraries and want to test the updates locally before pushing them upstream, then you can use local maven publishing for that too.

For updating and publishing the libraries locally, you need to follow the following instructions.

1. Firstly `git clone` the [`termux-app`] repo. Make whatever changes you need to make to [`termux-shared`] or other libraries. Then from the `root` directory of `termux-app`, run  `./gradlew publishReleasePublicationToMavenLocal`. This should build a release version of all the libraries and publish them locally. You can view the published files at `~/.m2/repository/com/termux/`. The version with which they are published will be defined by the `versionName` value in the `build.gradle` file.

2. In the `root` level `build.gradle` file of the plugin, uncomment the `mavenLocal()` line or replace the `maven { url "https://jitpack.io" }` line with it.

```
allprojects {
    repositories {
    	...
        mavenLocal()
        //maven { url "https://jitpack.io" }
    }
}
```

3. In the `app` level `build.gradle` file of the plugin, comment out the original `com.termux.termux-app:termux-shared:*` dependency line and replace it with `com.termux:termux-shared:*`. Note that the [`groupId`](https://maven.apache.org/pom.html#maven-coordinates) used by `Jitpack` is `com.termux.termux-app`, but when publishing locally, its `com.termux`.

```
dependencies {
    //implementation 'com.termux.termux-app:termux-shared:0.117'

    // Use if below libraries are published locally by termux-app with `./gradlew publishReleasePublicationToMavenLocal` and used with `mavenLocal()`.
    // If updates are done, republish there and sync project with gradle files here
    implementation 'com.termux:termux-shared:0.117'
}
```

3. Run `Sync Gradle with Project Files` in Android Studio in `File` drop down menu of the plugin app window to import the locally published library. You can check the `Build` tab at the bottom of Android Studio for any import errors.

Now every time you make changes in `termux-app` libraries, run  `./gradlew publishReleasePublicationToMavenLocal` to publish the changes and then in the plugin app, run `Sync Gradle with Project Files` to import the changes.

4. When testing is complete, make appropriate commits in `termux-app` and push them to [`termux-app`] upstream. If you are making a pull request, you will need to wait for changes to be merged with `master` before you will be able to push the plugins to their upstream. Once changes have been committed to `master`, find the commit hash from Github for your commit or the latest commit, then go to https://jitpack.io/#termux/termux-app and trigger a build for that commit under the `Commits` tab with `Get it` button and make sure it succeeds. You necessarily don't need to trigger build manually, importing it in plugin app in `5` will automatically do it, but since it may timeout/fail the first time, best do it before pushing changes to upstream. Note that by default, **Jitpack uses first `10` characters** for versions based on commit hashes.

5. In your plugin app, revert the changes made in `2` and `3` but update the original dependency line with the commit hash from Jitpack, like for example `9272a757af`. If instead of commit hash, a new version `termux-app` was published, like `0.118`, then use that instead of commit hash as version. Then make a commit like [`Changed: Bump termux-shared to 9272a757af`](https://github.com/termux/termux-tasker/commit/dfcdb427) and push changes to plugin upstream.

```
dependencies {
    implementation 'com.termux.termux-app:termux-shared:9272a757af'

    // Use if below libraries are published locally by termux-app with `./gradlew publishReleasePublicationToMavenLocal` and used with `mavenLocal()`.
    // If updates are done, republish there and sync project with gradle files here
    //implementation 'com.termux:termux-shared:0.117'
}
```

Note that making changes to library after dependencies have already been cached without incrementing version number may need deleting `gradle` cache if syncing gradle files doesn't work after publishing changes. This shouldn't normally happen, but sometimes does. Open `Gradle` right sidebar in Android Studio, then right click on top level entry, then select `Refresh Gradle Dependencies`, which will redownload/refresh all dependencies and will take a lot of time. Instead you can run `find ~/.gradle/caches/ -type d -name "*.termux*" -prune -exec rm -rf "{}" \; -print` to just remove `termux` cached libraries, and then running gradle sync again.


[`termux-app`]: https://github.com/termux/termux-app
[`termux-shared`]: https://github.com/termux/termux-app/tree/master/termux-shared
[`terminal-view`]: https://github.com/termux/termux-app/tree/master/terminal-view
[`terminal-emulator`]: https://github.com/termux/termux-app/tree/master/terminal-emulator
[`TermuxConstants`]: https://github.com/termux/termux-app/tree/master/termux-shared/src/main/java/com/termux/shared/termux/TermuxConstants.java
