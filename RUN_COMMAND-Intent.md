## RUN_COMMAND Intent

Third-party apps that are not part of Termux world can run commands in Termux app context by either sending an intent to `RunCommandService` or becoming a plugin host for the `termux-tasker` plugin client.

The intent can either be sent with `am startservice` command or with Java. Getting command result back is also possible for intents sent with Java, but not possible with `am startservice` command and it will require Termux app version `>= 0.109`.

You may also want to check out `termux-tasker` [README](https://github.com/termux/termux-tasker).


### Setup Instructions

#### `com.termux.permission.RUN_COMMAND` permission (Mandatory)

The Intent sender/third-party app must request the `com.termux.permission.RUN_COMMAND` permission in its `AndroidManifest.xml` and it should be granted by user to the app through the app's `App Info` `Permissions` page in Android Settings, likely under `Additional Permissions`. This is a security measure to prevent any other apps from running commands in `Termux` context which do not have the required permission granted to them.  

For `Tasker` you can grant it with:  
    `Android Settings` -> `Apps` -> `Tasker` -> `Permissions` -> `Additional permissions` -> `Run commands in Termux environment`.  


#### `allow-external-apps` property (Mandatory)

The `allow-external-apps` property must be set to `true` in `~/.termux/termux.properties` in Termux app, regardless of if the executable path is inside or outside the `~/.termux/tasker/` directory. Check [here](https://github.com/termux/termux-tasker#allow-external-apps-property-optional) for more info.  


#### `Draw Over Apps` permission (Optional)

For android `>= 10` there are new [restrictions](https://developer.android.com/guide/components/activities/background-starts) that prevent activities from starting from the background. This prevents the background `TermuxService` from starting a terminal session in the foreground and running the commands until the user manually clicks `Termux` notification in the status bar drop-down notifications list. This only affects commands that are to be executed in a terminal session and not the background ones. `Termux` version `>= 0.100` requests the `Draw Over Apps` permission so that users can bypass this restriction so that commands can automatically start running without user intervention.  

You can grant `Termux` the `Draw Over Apps` permission from its `App Info` activity:  
    `Android Settings` -> `Apps` -> `Termux` -> `Advanced` -> `Draw over other apps`.  


#### `Storage` permission (Optional)
Termux app must be granted `Storage` permission if the executable is accessing or working directory is set to path in external shared storage. The common paths which usually refer to it are `~/storage`, `/sdcard`, `/storage/emulated/0` etc.  

You can grant `Termux` the `Storage` permission from its `App Info` activity:  
For Android version < 11:  
    `Android Settings` -> `Apps` -> `Termux` -> `Permissions` -> `Storage`.  
For Android version >= 11  
    `Android Settings` -> `Apps` -> `Termux` -> `Permissions` -> `Files and media` -> `Allowed management of all files`.  

**NOTE:** For Android version >= 11, sometimes you will get `Permission Denied` errors for  external shared storage even when you have granted `Files and media` permission. To solve this, Deny the permission and then Allow it again and restart Termux. Also check [termux-setup-storage](https://wiki.termux.com/wiki/Termux-setup-storage).


#### Battery Optimizations (May be mandatory depending on device)

Some devices kill apps aggressively or prevent apps from starting from background. If Termux is running into such problems, then exempt it from such restrictions. The user may also disable battery optimizations for Termux to reduce the chances of Termux being killed by Android even further due to violation of not being able to call `startForeground()` within `~5s` of service start in android >= 8. Check [dontkillmyapp](https://dontkillmyapp.com) for device specific info to opt-out of battery optimizations.
##



### RUN_COMMAND Intent Command Extras

The `RUN_COMMAND` intent expects the following extras that should contain the info of the command to be executed.

The extra constant values are defined by [`TermuxConstants`] class of the [`termux-shared`](https://github.com/termux/termux-app/tree/master/termux-shared) library.

- The `String` `RUN_COMMAND_SERVICE.EXTRA_COMMAND_PATH` extra for absolute path of command. (**mandatory**)  
- The `String[]` `RUN_COMMAND_SERVICE.EXTRA_ARGUMENTS` extra for any arguments to pass to command.  
- The `String` `RUN_COMMAND_SERVICE.EXTRA_WORKDIR` extra for current working directory of command. This defaults to `TermuxConstants.TERMUX_HOME_DIR_PATH`.  
- The `boolean` `RUN_COMMAND_SERVICE.EXTRA_BACKGROUND` extra whether to run command in background or foreground terminal session. This defaults to `false`.  
- The `String` `RUN_COMMAND_SERVICE.EXTRA_SESSION_ACTION` extra for for session action of foreground commands. This defaults to `TERMUX_SERVICE.VALUE_EXTRA_SESSION_ACTION_SWITCH_TO_NEW_SESSION_AND_OPEN_ACTIVITY`. *Requires Termux app version `>= 0.109`*.  
- The `String` `RUN_COMMAND_SERVICE.EXTRA_COMMAND_LABEL` extra for label of the command. *Requires Termux app version `>= 0.109`*.  
- The markdown `String` `RUN_COMMAND_SERVICE.EXTRA_COMMAND_DESCRIPTION` extra for description of the command. This should ideally be get short. *Requires Termux app version `>= 0.109`*.  
- The markdown `String` `RUN_COMMAND_SERVICE.EXTRA_COMMAND_HELP` extra for help of the command. This can add details about the command. 3rd party apps can provide more info to users for setting up commands. Ideally a url link should be provided that goes into full details. *Requires Termux app version `>= 0.109`*.  
- The `Parcelable` `RUN_COMMAND_SERVICE.EXTRA_PENDING_INTENT` extra containing the pending intent with which result of commands should be returned to the caller. The results will be sent in the `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE` bundle. This is optional and only needed if caller wants the results back. *Requires Termux app version `>= 0.109`*.  


The `RUN_COMMAND_SERVICE.EXTRA_COMMAND_PATH` and `RUN_COMMAND_SERVICE.EXTRA_WORKDIR` can optionally be prefixed with `$PREFIX/` or `~/` if an absolute path is not to be given. The `$PREFIX/` will expand to `TermuxConstants.TERMUX_PREFIX_DIR_PATH` and `~/` will expand to `TermuxConstants.TERMUX_HOME_DIR_PATH`.


The `EXTRA_COMMAND_*` extras are used for logging and their values are provided to users in case
of failure in a popup. The popup shown is in [commonmark-spec](https://commonmark.org/) markdown using [markwon](https://github.com/noties/Markwon) library so make sure to follow its formatting rules. Also make sure to end lines with 2 blank spaces to prevent word-wrap wherever needed. It's the users and 3rd party apps responsibility to use them wisely. There are also android internal intent size limits (roughly `500KB`) that must not exceed when sending intents so make sure the combined size of ALL extras is less than that. There are also limits on the arguments size you can pass to commands or the full command string length that can be run, which is likely equal to `131072` bytes or `128KB` on an android device. Check [Arguments and Result Data Limits](https://github.com/termux/termux-tasker#arguments-and-result-data-limits) for more details.
##



### RUN_COMMAND Intent Result Extras

Requires Termux app version `>= 0.109`.

The `RUN_COMMAND` intent returns the following extras in the `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE` [Bundle](https://developer.android.com/reference/android/os/Bundle) if a pending intent is sent by the caller in the [Parcelable](https://developer.android.com/reference/android/os/Parcelable) `RUN_COMMAND_SERVICE.EXTRA_PENDING_INTENT` extra. Getting result of commands back is not possible for intents sent with `am startservice` command and so Java should be used for that case.

For foreground commands (`RUN_COMMAND_SERVICE.EXTRA_BACKGROUND` is `false`):
- `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDOUT` will contain `session transcript`.
- `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDERR` will be `null` since its not used.
- `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_EXIT_CODE` will contain `exit code` of session.

For background commands (`RUN_COMMAND_SERVICE.EXTRA_BACKGROUND` is `true`):
- `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDOUT` will contain `stdout` of commands.
- `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDERR` will contain `stderr` of commands.
- `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_EXIT_CODE` will contain `exit code` of command.


The `session transcript` for foreground commands will contain both `stdout` and `stderr` combined, basically anything sent to the the pseudo terminal `/dev/pts`, including `PS1` prefixes for interactive sessions. Getting separate `stdout` and `stderr` can currently only be done with background commands.

The `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDOUT_ORIGINAL_LENGTH` and `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDERR_ORIGINAL_LENGTH` will contain the original length of `stdout` and `stderr` respectively.

The internal errors raised by termux outside the shell will be sent in the the `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_ERR` (`err` and `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_ERRMSG` (`errmsg`) extras. These will contain errors like if starting a termux command failed or if the user manually exited the termux sessions or android killed the termux service before the commands had finished executing. The `err` value will be `Activity.RESULT_OK` (`-1`) if no internal errors are raised.

Note that if `stdout` or `stderr` are too large in length, then a `android.os.TransactionTooLargeException` exception will be raised when the pending intent is sent back containing the results, But it cannot be caught by the intent sender and intent will silently fail with `logcat` entries for the exception raised internally by android os components. To prevent this, the `stdout` and `stderr` sent back will be truncated from the start to max `100KB` combined. The original length of `stdout` and `stderr` will be provided in `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDOUT_ORIGINAL_LENGTH` and `TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDERR_ORIGINAL_LENGTH` extras respectively, so that the caller can check if either of them were truncated. The `errmsg` will also be truncated from end to max `25KB` to preserve start of stacktraces.
##



### Basic Examples

#### `top` command with Java

```java
intent.setClassName("com.termux", "com.termux.app.RunCommandService");
intent.setAction("com.termux.RUN_COMMAND");
intent.putExtra("com.termux.RUN_COMMAND_PATH", "/data/data/com.termux/files/usr/bin/top");
intent.putExtra("com.termux.RUN_COMMAND_ARGUMENTS", new String[]{"-n", "5"});
intent.putExtra("com.termux.RUN_COMMAND_WORKDIR", "/data/data/com.termux/files/home");
intent.putExtra("com.termux.RUN_COMMAND_BACKGROUND", false);
intent.putExtra("com.termux.RUN_COMMAND_SESSION_ACTION", "0");
startService(intent);
```

#### `top` command with `am startservice` command:
```bash
am startservice --user 0 -n com.termux/com.termux.app.RunCommandService \
-a com.termux.RUN_COMMAND \
--es com.termux.RUN_COMMAND_PATH '/data/data/com.termux/files/usr/bin/top' \
--esa com.termux.RUN_COMMAND_ARGUMENTS '-n,5' \
--es com.termux.RUN_COMMAND_WORKDIR '/data/data/com.termux/files/home' \
--ez com.termux.RUN_COMMAND_BACKGROUND 'false' \
--es com.termux.RUN_COMMAND_SESSION_ACTION '0'
```

### Advance Examples

It's probably wiser for apps to declare the [`termux-shared`] library as a dependency and import the [`TermuxConstants`] class and use the variables provided for actions and extras instead of using hardcoded extra key values.

If your app wants to receive termux session command results, then put the pending intent for your app like for an [IntentService](https://developer.android.com/reference/android/app/IntentService) in the `RUN_COMMAND_SERVICE.EXTRA_PENDING_INTENT` extra.

Include the [`termux-shared`] library as a dependency in the `build.gradle` file.

```
implementation 'com.termux:termux-shared:0.109'
```

Define the `RUN_COMMAND` intent sender code.

```java
import com.termux.shared.termux.TermuxConstants;
import com.termux.shared.termux.TermuxConstants.TERMUX_APP.RUN_COMMAND_SERVICE;

...

Intent intent = new Intent();
intent.setClassName(TermuxConstants.TERMUX_PACKAGE_NAME, TermuxConstants.TERMUX_APP.RUN_COMMAND_SERVICE_NAME);
intent.setAction(RUN_COMMAND_SERVICE.ACTION_RUN_COMMAND);
intent.putExtra(RUN_COMMAND_SERVICE.EXTRA_COMMAND_PATH, "/data/data/com.termux/files/usr/bin/top");
intent.putExtra(RUN_COMMAND_SERVICE.EXTRA_ARGUMENTS, new String[]{"-n", "2"});
intent.putExtra(RUN_COMMAND_SERVICE.EXTRA_WORKDIR, "/data/data/com.termux/files/home");
intent.putExtra(RUN_COMMAND_SERVICE.EXTRA_BACKGROUND, false);
intent.putExtra(RUN_COMMAND_SERVICE.EXTRA_SESSION_ACTION, "0");
intent.putExtra(RUN_COMMAND_SERVICE.EXTRA_COMMAND_LABEL, "top command");
intent.putExtra(RUN_COMMAND_SERVICE.EXTRA_COMMAND_DESCRIPTION, "Runs the top command to show processes using the most resources.");


// Create the intent for the IntentService class that should be sent the result by TermuxService
Intent pluginResultsServiceIntent = new Intent(MainActivity.this, PluginResultsService.class);

// Optional put an extra that uniquely identifies the command internally for your app.
// This can be an Intent extra as well with more extras instead of just an int.
pluginResultsServiceIntent.putExtra(PluginResultsService.EXTRA_EXECUTION_ID, PluginResultsService.getNextExecutionId());

// Create the PendingIntent that will be used by TermuxService to send result of commands back to the IntentService
PendingIntent pendingIntent = PendingIntent.getService(MainActivity.this, 1, pluginResultsServiceIntent, PendingIntent.FLAG_ONE_SHOT);
intent.putExtra(RUN_COMMAND_SERVICE.EXTRA_PENDING_INTENT, pendingIntent);

// Send command intent for execution
startService(intent);
```

Define the `PluginResultsService` class which extends from `IntentService`.

```java
import com.termux.shared.termux.TermuxConstants.TERMUX_APP.TERMUX_SERVICE;

...

public class PluginResultsService extends IntentService {

    public static final String EXTRA_EXECUTION_ID = "execution_id";

    private static int EXECUTION_ID = 1000;

    public static final String PLUGIN_SERVICE_LABEL = "PluginResultsService";

    private static final String LOG_TAG = "PluginResultsService";

    public PluginResultsService(){
        super(PLUGIN_SERVICE_LABEL);
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        if (intent == null) return;

        if(intent.getComponent() != null)
            Log.d(LOG_TAG, PLUGIN_SERVICE_LABEL + " received execution result from " + intent.getComponent().toString());


        final Bundle resultBundle = intent.getBundleExtra(TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE);
        if (resultBundle == null) {
            Log.e(LOG_TAG, "The intent does not contain the result bundle at the \"" + TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE + "\" key.");
            return;
        }

        final int executionId = intent.getIntExtra(EXTRA_EXECUTION_ID, 0);

        Log.d(LOG_TAG, "Execution id " + executionId + " result:\n" +
                "stdout:\n```\n" + resultBundle.getString(TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDOUT, "") + "\n```\n" +
                "stdout_original_length: `" + resultBundle.getString(TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDOUT_ORIGINAL_LENGTH) + "`\n" +
                "stderr:\n```\n" + resultBundle.getString(TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDERR, "") + "\n```\n" +
                "stderr_original_length: `" + resultBundle.getString(TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_STDERR_ORIGINAL_LENGTH) + "`\n" +
                "exitCode: `" + resultBundle.getInt(TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_EXIT_CODE) + "`\n" +
                "errCode: `" + resultBundle.getInt(TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_ERR) + "`\n" +
                "errmsg: `" + resultBundle.getString(TERMUX_SERVICE.EXTRA_PLUGIN_RESULT_BUNDLE_ERRMSG, "") + "`");
    }

    public static synchronized int getNextExecutionId() {
        return EXECUTION_ID++;
    }

}
```

Declare `PluginResultsService` entry in `AndroidManifest.xml`

```xml
<service android:name=".PluginResultsService" />
```
##



### Package Visibility

If your third-party app is targeting sdk `30` (android `11`), then it needs to add `com.termux` package to the `queries` element or request `QUERY_ALL_PACKAGES` permission in its `AndroidManifest.xml`. Otherwise it will get `PackageSetting{...... com.termux/......} BLOCKED` errors in `logcat` and `RUN_COMMAND` won't work.

Check [package-visibility](https://developer.android.com/training/basics/intents/package-visibility#package-name), `QUERY_ALL_PACKAGES` [googleplay policy](https://support.google.com/googleplay/android-developer/answer/10158779) and this [article](https://medium.com/androiddevelopers/working-with-package-visibility-dc252829de2d) for more info.
##



### Privacy

If a third party app ran a termux command for a user, then it can get the session transcript back for the terminal session, and `stdout`/`stderr` for background commands using `PendingIntent`. Even with the dual `RUN_COMMAND` permission and `allow-external-app` requirement, this may not be something that the user wants, since it could give the 3rd party app access to private user data. So use this wisely. In future, a whitelist/blacklist may be implemented to give further control to the user for which app's can get the result back or show prompts before running commands. Although, the 3rd party app can still use physical files or intents inside the commands run to get the result back, but this can likely be solved by approving a script before its run each time or permanently by storing the script hash in an internal termux database.
##

[`TermuxConstants`]: https://github.com/termux/termux-app/tree/master/termux-shared/src/main/java/com/termux/shared/termux/TermuxConstants.java
[`termux-shared`]: https://github.com/termux/termux-app/tree/master/termux-shared

