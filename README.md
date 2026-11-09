# My Stock

A simple stock tracker for keeping count of your things, seeing what is running low, and adding or removing items. It runs on Android as an app, offline on a computer, and online as a shared list.

## Files

| File | What it is |
|---|---|
| `MyStock.apk` | The Android app. Works offline, data stays on the phone. |
| `My_Stock_Offline.html` | The same app as one file. Open it on a computer with no internet. |
| `MyStock_Web_App.zip` | The app as an installable web app, for putting online with your own link (see below). |
| `MyStock_Android_source.zip` | Android project source plus the signing key (`mystock.keystore`). Keep this safe. |
| Shared online version | https://claude.ai/artifact/5r1Webj1EiWmPf9vmKQeYr (several people, shared list, needs internet) |

All three versions are the same code. When the app can reach the shared database it works online and shares data. When it can't (the APK, or the HTML file opened from a computer), it saves data on that device only.

## Features

- **Tabs:** group items into your own tabs (for example Kitchen or Garage). Add, rename and remove tabs from the **Tabs** button. The last tab can't be removed.
- **Items:** add an item with a starting amount and a "low when at or below" level. Use the **−** and **+** buttons to change the amount by one, or the **⋯** button to add or use any amount, change the low level, or remove the item.
- **Low stock:** an item turns red when it reaches its low level. **Low only** shows just those items.
- **Buy list (Excel):** the **Buy list (Excel)** button saves a real `.xlsx` file of every low or out item across all tabs, with the columns Tab, Item, In stock, Low level, Status and an empty To buy column to fill in. The file is named `low-stock-DATE.xlsx`. On Android it is saved in the Downloads folder and the phone offers to open it. On a computer it goes to the browser's download folder. In the shared online version you are asked to confirm the save.
- **Password:** the app asks for a password when it opens. The first time, you create it. Use **Change password** and **Lock** at the bottom of the screen.
- **Dark / light mode:** the round button in the top-right corner. Your choice is remembered. On Android it follows the phone's setting until you change it.
- **Icons:** every main button has a picture next to its label (plus, layers, warning triangle, download arrow, key, padlock, pencil, trash can).
- **Backup / restore:** **Backup** copies your data as text. Paste it into **Restore** on another copy of the app to move or restore your list.

## Put it online with your own link, and install it as an app

`MyStock_Web_App.zip` is the same app prepared as an installable web app. Upload its files to a static web host (a service that only serves files) and you get a normal https link. Opening that link in Chrome or Edge shows an **Install app** button in the app, and the browser's own install option, to run it in its own window like an app. It also keeps working with no internet after the first visit.

Files in the zip: `index.html`, `manifest.webmanifest`, `sw.js` and three icons. Keep them together in the same folder.

Publishing with GitHub Pages (free for public repositories):
1. Create a free GitHub account and a new **public** repository, for example `my-stock`.
2. Upload all the files from the zip (unzipped) to the repository's main branch.
3. Go to **Settings → Pages**, choose **Deploy from a branch**, pick `main` and the `/ (root)` folder, and save.
4. After a minute or two your link appears: `https://YOUR-NAME.github.io/my-stock/`.

The link lasts as long as that repository and account exist. Nobody can promise a link forever, so keep the account and repository. Anyone who knows the link can open the page, but their list is only in their own browser.

Data on a web host like this is saved per device and browser, like the offline file. Only the Claude-hosted version shares one list between people.

To publish a new version later, replace `index.html` and raise the number in `CACHE = 'my-stock-v3'` at the top of `sw.js` so browsers pick up the update.

## Install on Android

1. Copy `MyStock.apk` to the phone.
2. Open it. When Android asks, allow installing apps from that source.
3. Open **My Stock** and create your password.

Requires Android 7.0 or newer. The app is built for the latest Android (Android 16), so current phones should not show the "built for an older version" warning. The data is stored inside the app on that phone. Uninstalling the app erases it, so export a backup first.

## Use on a computer, offline

Save `My_Stock_Offline.html` anywhere and double-click it. It opens in your browser and needs no internet. Data is saved in that browser on that computer.

## Shared online use

Open the online link. Everyone who opens it sees the same list, and changes show up for everyone as they happen. The password is shared too.

- For other people to make changes, share the page with them in the share menu as Contributors or Editors. People with a view-only link can look but can't change anything.
- If two people tap + or − on the same item at the same moment, one tap can be lost.

## Moving data between versions

1. In the old version, tap **Backup**, then **Export**. The data appears in the box and is copied.
2. In the new version, tap **Backup**, paste the data into the box, and tap **Restore**.

Backups from the first offline version are accepted too. The first offline version's items and tabs are also picked up automatically when the new offline file or app is opened in the same browser.

## About the password

The password is a lock screen, not encryption. It keeps casual users out, but someone who knows how to look inside the file or the browser's saved data can get past it. Don't use it to protect anything sensitive. There is no way to recover a forgotten password. Clearing the app's or browser's saved data resets it but also erases the list, so keep a backup.

## Signing key

`mystock.keystore` is in the source zip.

- Alias: `mystock`
- Store and key password: `mystock123`

Every future version of the Android app must be signed with this same key. If it isn't, the phone makes you uninstall the old app first, and you lose its data. Keep the file somewhere safe, and change the password if you share it.

## Rebuilding the APK

The source zip holds the raw project files (`AndroidManifest.xml`, `res/`, `src/`, `assets/index.html`). The APK was built with these tools: JDK 17, `aapt2` and `zipalign` (Android build-tools 35.0.2), `dx`, `apksigner`, and the Android 16 (API 36) `android.jar`.

```bash
aapt2 compile --dir res -o build/res.zip
aapt2 link -I android.jar --manifest AndroidManifest.xml -A assets \
  --min-sdk-version 24 --target-sdk-version 36 -o build/base.apk build/res.zip
javac --release 8 -cp android.jar -d obj src/com/mystock/app/MainActivity.java
dx --dex --min-sdk-version=24 --output=build/classes.dex obj
cd build && zip -j base.apk classes.dex && cd ..
zipalign -f -p 4 build/base.apk build/aligned.apk
apksigner sign --ks mystock.keystore --ks-pass pass:mystock123 \
  --key-pass pass:mystock123 --out build/MyStock.apk build/aligned.apk
```

To change the app, edit `assets/index.html` (the whole app is that one file) and rebuild. Raise `versionCode` in `AndroidManifest.xml` each time you release an update (the current version is 3, named 1.2).

## Memory and stability (Android)

- The app pauses its page when you switch away and frees cached data when the phone is short on memory.
- If the system reclaims the page's memory, the app starts a fresh page instead of crashing. It shows the password screen again, and your saved list is untouched.
- It stays clear of the status bar, navigation bar, camera cut-out and keyboard, and the Back button closes an open box first, then sends the app to the background.
- It can't open any outside web pages, has no internet permission, and turns off Android's automatic cloud backup of its data. Use **Backup** to keep a copy of your list.

## How it works

- **App:** one HTML file with its own styles and script, and no outside connections. The Android app is a thin wrapper that shows this file in a full-screen web view and adds two small helpers: copying backup text to the clipboard and reporting whether the phone is in dark mode.
- **Local data:** saved in the device's browser storage under the keys `my-stock-v2` (tabs and items), `my-stock-auth` (password hash) and `my-stock-theme`.
- **Online data:** saved in the page's shared database as `tabs` and `items` collections plus a `meta/auth` document for the password.

## Testing done, and its limits

- An automated test was run 3 times in a computer-style setup and 3 times in an Android-style setup (6 runs, 42 to 44 checks each, all passed). The Excel export was also run 3 times in each setup and the resulting file was read back successfully. It covered the password, Lock, items, low-stock warnings, tabs, dark/light button, backup and restore, keeping data after reopening, and moving over data from the first offline version.
- The tests use a simulated page, not a real browser or phone.
- The APK was checked for correct signing and contents. It has **not** been tried on a real Android phone or emulator.
- The shared online mode has not been tried with a second person.

## Known limits

- Choosing a backup file from inside the Android app doesn't work. Paste the backup text instead.
- On Android 7 to 9, the Excel file is saved in the app's own Downloads folder (inside Android/data) instead of the main Downloads folder, and it isn't opened automatically.
- The Excel export was checked by opening the file with a spreadsheet reader and LibreOffice. It has not been opened in Microsoft Excel.
- The Android app and the offline file can't share a list with other devices. Only the online version does that.
- The monthly history view from an early version was removed at your request.
