Cordova Cloud Settings Plugin

# Summary

This plugin makes use of [Android's Data Backup service](http://developer.android.com/guide/topics/data/backup.html) to backup and restore app data. Currently only Android is supported.

# Installation

## Install the plugin

```sh
cordova plugin add cordova-plugin-cloud-settings --variable BACKUP_SERVICE_KEY="<API_KEY>"
```

If you've not yet registered to use Google Data Backup, [get a key here](https://developer.android.com/google/backup/signup.html?csw=1)

## Supported platforms

 - **Android** version 2.2 (API level 8) or higher

# Getting Started

A typical lifecycle is as follows:
 - User installs your app for the first time
 - App starts, sees it has no existing data, makes a request to restore backup
 - No data is found, your app creates stub data
 - Users uses your app, generates data: Your app requests a backup
 - Further use, further backups...
 - User downloads your app onto a new device
 - App starts, sees it has no existing data, makes a request to restore backup
 - Data is found, restored and returned to your app
 - User continues where they left off

# API

## Backup

### `plugins.backup.save(data, [successCallback], [errorCallback]);`

```javascript
var appData = {
  user: {
    id: 1678,
    name: 'Fred',

    preferences: {
      mute: true,
      locale: 'en_GB'
    }
  }
}

plugins.backup.save(appData);
```

## Restore

### `plugins.backup.restore(successCallback, [errorCallback]);`

```javascript
function handleRestore (data) {
  if (data) {
    appData = data;
  }
  else {
    // initialize first-use data
  }
}

plugins.backup.restore(handleRestore);
```

# Testing

## Testing Android

### Test backup
To test backup of settings you need to manually invoke the backup manager (as instructed in [the Android documentation](https://developer.android.com/guide/topics/data/testingbackup)) to force backing up of the updated values:

First make sure the backup manager is enabled and setup for verbose logging:

```bash
    $ adb shell bmgr enabled
    $ adb shell setprop log.tag.GmsBackupTransport VERBOSE
    $ adb shell setprop log.tag.BackupXmlParserLogging VERBOSE
```

#### Android 7.0 and above

Run the following command to perform a backup:

```bash
    $ adb shell bmgr backupnow <APP_PACKAGE_ID>
```

#### Android 6

* Run the following command:
```bash
    $ adb shell bmgr backup @pm@ && adb shell bmgr run
```

* Wait until the command in the previous step finishes by monitoring `adb logcat` for the following output:
```
    I/BackupManagerService: K/V backup pass finished.
```

* Run the following command to perform a backup:
```bash
    $ adb shell bmgr fullbackup <APP_PACKAGE_ID>
```


#### Android 5.1 and below
Run the following commands to perform a backup:

```bash
    $ adb shell bmgr backup <APP_PACKAGE_ID>
    $ adb shell bmgr run
```

### Test restore

To manually initiate a restore, run the following command:
```bash
    $ adb shell bmgr restore <TOKEN> <APP_PACKAGE_ID>
```

* To look up backup tokens run `adb shell dumpsys backup`.
* The token is the hexidecimal string following the labels `Ancestral:` and `Current:`
    * The ancestral token refers to the backup dataset that was used to restore the device when it was initially setup (with the device-setup wizard).
    * The current token refers to the device's current backup dataset (the dataset that the device is currently sending its backup data to).
* You can use a regex to filter the output for just your app ID, for example if your app package ID is `io.cordova.plugin.cloudsettings.test`:
```bash
    $ adb shell dumpsys backup | grep -P '^\S+\: | \: io\.cordova\.plugin\.cloudsettings\.test'
```

You also can test automatic restore for your app by uninstalling and reinstalling your app either with adb or through the Google Play Store app.

### Wipe backup data
To wipe the backup data for your app:

```bash
    $ adb shell bmgr list transports
    # note the one with an * next to it, it is the transport protocol for your backup
    $ adb shell bmgr wipe [* transport-protocol] <APP_PACKAGE_ID>
```

## Testing iOS

TODO

# Authors

[Dave Alden](https://github.com/dpa99c)

Based on the plugin: https://github.com/cloakedninjas/cordova-plugin-backup

# Licence

The MIT License

Copyright (c) 2018, Dave Alden (Working Edge Ltd.)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
