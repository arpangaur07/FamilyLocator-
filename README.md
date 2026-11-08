# 📍 Family Locator

A simple Android app for families to share **live location**, **battery level** and **messages**. It works anywhere in the world over mobile data or Wi‑Fi.

One phone **shares** its location; the parent's phone **sees it on a map** and can send messages.

> **Built by Arpan** · Kotlin · Firebase Realtime Database · OpenStreetMap

---

## ✨ Features

**On the phone that shares (e.g. a son or daughter)**
- Precise GPS location sent about every minute, and faster when moving
- Battery percentage and charging status
- Keeps working in the background and restarts after a reboot
- Always shows a "Sharing your location" notification, so it's never hidden
- Sharing can be turned **on/off** anytime with one switch
- Messages from parents arrive as notifications with quick replies ("OK", "On my way")

**On the parent's phone**
- Live position on a map with an accuracy circle and street address
- Battery %, charging status and "Updated 2 min ago"
- Warning when the phone has been offline for more than 10 minutes
- **Refresh now** asks the phone for a fresh GPS fix
- **Directions** and **Google Maps** buttons open the exact spot in Google Maps
- Two‑way chat with quick messages ("Come home", "Call me", "Charge your phone")
- "Sent / Delivered / Seen" status on messages

**Costs nothing:** it uses Firebase's free Spark plan and OpenStreetMap, so you need no credit card and no Google Maps API key.

---

## ⚠️ Important: use your OWN Firebase

This repo **does not include** any Firebase project. Each person who builds the app must:

1. Create **their own** Firebase project (free)
2. Download **their own** `google-services.json`
3. Put **their own** database link in `Fb.kt`

**Why?** If you used someone else's Firebase, all your family's locations would be stored in *their* database. They could see them, and you couldn't control or delete them. Your location data should live only in a project **you** own.

`google-services.json` is in `.gitignore`, so it's never uploaded to GitHub. Never commit it and never share it.

---

## 📋 What you need

| Thing | Notes |
|---|---|
| A Google account | For Firebase (free) |
| A Windows / Mac / Linux computer | To build the app |
| [Android Studio](https://developer.android.com/studio) | Free; install with default settings |
| 2 Android phones | Android 8.0 or newer; iPhone is **not** supported |
| USB cable | To install the app (or send the APK file instead) |

Setup takes about **30–45 minutes** the first time.

---

## 🔥 Part 1: Set up Firebase (step by step)

Firebase is a free Google service that sits between the two phones. The sharing phone uploads its location there, and the parent's phone reads it from there.

### Step 1: Create a Firebase project
1. Go to **https://console.firebase.google.com** and sign in with your Google account.
2. Click **Create a project** (or **Add project**).
3. Enter a name, e.g. `my-family-locator`, and click **Continue**.
4. **Turn off Google Analytics** (not needed), then click **Create project**.
5. Wait about 30 seconds, then click **Continue**.

You're now on the **Project Overview** page. The plan badge should say **Spark plan** (free).

### Step 2: Turn on Anonymous sign‑in
The app signs in silently with no email or password, so only real app installs can read or write data.

1. In the left menu, open **Security → Authentication**. In some versions it's directly under **Build → Authentication**.
2. Click **Get started**.
3. Open the **Sign‑in method** tab.
4. Click **Anonymous**, switch it **ON**, and click **Save**.
5. Check that the list now shows: Anonymous ✅ **Enabled**.

> ❗ If you skip this, the app shows **"Sign‑in failed… Is Anonymous sign‑in enabled?"**

### Step 3: Create the Realtime Database
1. In the left menu, open **Databases & Storage → Realtime Database**. In some versions it's **Build → Realtime Database**.
   > Make sure it's **Realtime Database**, not *Firestore*.
2. Click **Create database**.
3. **Location:** choose the one closest to you:
   - India / Asia → **Singapore (asia-southeast1)**
   - Europe → **Belgium (europe-west1)**
   - Americas → **United States (us-central1)**
4. Click **Next**, choose **Start in locked mode**, and click **Enable**.

### Step 4: Set the security rules
The rules decide **who can read and write** your database. These rules say:
- Nobody can read or list the whole database.
- Only signed‑in app users can read or write a family, and only if they know its exact 10‑character family code.

1. In Realtime Database, open the **Rules** tab.
2. **Delete everything** in the box.
3. Paste this. It's the same as `database.rules.json` in this repo:

```json
{
  "rules": {
    ".read": false,
    ".write": false,
    "families": {
      "$code": {
        ".read": "auth != null && $code.length == 10",
        ".write": "auth != null && $code.length == 10"
      }
    }
  }
}
```

4. Click **Publish**.

> ❗ **Never** use `".read": true, ".write": true` ("test mode"). That lets **anyone on the internet** read every family's location.

### Step 5: Copy YOUR database link
1. Open the **Data** tab in Realtime Database.
2. At the top of the data box there's a link with a 📋 copy icon. It looks like **one** of these:
   - `https://YOUR-PROJECT-default-rtdb.firebaseio.com` (USA)
   - `https://YOUR-PROJECT-default-rtdb.asia-southeast1.firebasedatabase.app` (Singapore)
   - `https://YOUR-PROJECT-default-rtdb.europe-west1.firebasedatabase.app` (Belgium)
3. **Copy it** and keep it in Notepad for Part 2.

> ❗ Copy the link from **inside the Data box**, not the browser address bar. The address bar (`console.firebase.google.com/project/...`) is the wrong link.

### Step 6: Register the Android app and download `google-services.json`
1. Go to **Project Overview** (🏠 top left) and click **+ Add app**, then the **Android** icon.
2. **Android package name:** type exactly
   ```
   com.arpan.familylocator
   ```
   *(If you change the package name in the code, use your new one here. They must match exactly.)*
3. App nickname: anything (e.g. `Family Locator`). Leave SHA‑1 empty and click **Register app**.
4. Click **Download google-services.json**.
5. Click **Next** through the remaining steps. You can **skip the "Add Firebase SDK" code**, because this project already includes it. Then click **Continue to console**.

> ⚠️ **Order matters:** create the database (Step 3) **before** downloading `google-services.json`.
> If you downloaded it earlier, get it again: ⚙️ **Project settings → General → Your apps → google-services.json**.

---

## 🛠️ Part 2: Set up the project

### Step 7: Download the code
```bash
git clone https://github.com/<your-username>/FamilyLocator.git
```
Or click the green **Code** button, choose **Download ZIP**, and extract it.

### Step 8: Add YOUR `google-services.json`
Copy the file you downloaded in Step 6 into the **`app`** folder:

```
FamilyLocator/
├── app/
│   ├── google-services.json   ← put it HERE
│   ├── build.gradle.kts
│   └── src/
├── build.gradle.kts
└── settings.gradle.kts
```

✅ Check the file name is **exactly** `google-services.json`. Windows often adds `(1)`, `(2)` and so on when you download twice. Rename it if so.

### Step 9: Put YOUR database link in the code
Open:
```
app/src/main/java/com/arpan/familylocator/Fb.kt
```
Find this line:
```kotlin
private const val DB_URL = "PASTE_YOUR_DATABASE_URL_HERE"
```
Replace the text in quotes with **your** link from Step 5:
```kotlin
private const val DB_URL = "https://my-family-locator-default-rtdb.asia-southeast1.firebasedatabase.app"
```

> ❗ Keep the quotes `" "`. No spaces and no `/` at the end.
> ❗ This must be **your own** link. Don't use one copied from someone else's code or screenshots.

---

## 📱 Part 3: Build and install

### Step 10: Open in Android Studio
1. Open Android Studio and click **File → Open**.
2. Select the **`FamilyLocator`** folder that **directly contains** `app`, `gradle` and `settings.gradle.kts`.
   > If you extracted a ZIP, there may be a folder inside a folder. Pick the inner one.
3. Click **Trust Project**.
4. Wait for **Gradle sync** (progress bar at the bottom right). The first time takes **5–15 minutes**.

When it's done, the green **Run ▶** button at the top is clickable.

### Step 11: Turn on USB debugging on the phone
1. On the phone: **Settings → About phone** (on Samsung: **About phone → Software information**). Tap **Build number** 7 times.
   - On Xiaomi/Redmi/POCO, tap **MIUI / HyperOS version** 7 times.
2. Go back to **Settings → Developer options** and turn on **USB debugging**.
   - On Xiaomi, also turn on **Install via USB**.
3. Connect the phone by USB and tap **Allow** on the phone.

### Step 12: Install
1. In Android Studio, pick **your phone** in the device box at the top, not "Medium Phone API…" (that's a virtual phone).
2. Click **Run ▶**. The app installs and opens.
3. Repeat for the second phone.

**Without a cable:** go to **Build → Build App Bundle(s) / APK(s) → Build APK(s)** and click **locate**. Send `app-debug.apk` to the other phone (WhatsApp as a document, Google Drive, etc.), open it, and allow **Install unknown apps**.

---

## 👨‍👩‍👦 Part 4: Using the app

### On the phone that will share its location
1. Open **Family Locator** and choose **This is MY phone**.
2. Type your name and tap **Set up this phone**.
3. Allow the permissions in this order:
   - 📍 Location → **Precise** → **While using the app**
   - 🔔 Notifications → **Allow**
   - 📍 Location again → **Allow all the time** (keeps working when the app is closed or after a reboot)
   - 🔋 Battery → **Allow** (stops Android from killing the app)
4. You'll see a **family code** like `K7PQX-9M3RT`. Tap **Share** and send it to your parent.
5. "Sharing is ON" shows in green.

### On the parent's phone
1. Open **Family Locator** and choose **I'm a parent**.
2. Type your name (e.g. *Mom*) and the **family code**.
3. Tap **Connect**. The map opens on the shared location.

| Button | What it does |
|---|---|
| **Refresh now** | Asks the phone for a brand‑new GPS location (arrives in a few seconds if it's online) |
| **Directions** | Opens Google Maps navigation to the phone |
| **Google Maps** | Opens the exact spot in Google Maps (satellite, Street View) |
| **Unlink** | Disconnects this phone from the family |

---

## 🔋 Keep it reliable (very important)

Many phone brands kill background apps to save battery. On the **sharing** phone:

| Brand | What to change |
|---|---|
| **All phones** | Settings → Apps → Family Locator → Battery → **Unrestricted / No restrictions** |
| **Samsung** | Settings → Battery → Background usage limits → make sure Family Locator is **not** in "Sleeping apps" or "Deep sleeping apps" |
| **Xiaomi / Redmi / POCO** | Apps → Family Locator → **Autostart ON**, Battery saver → **No restrictions** |
| **Realme / Oppo / OnePlus** | Battery → App battery management → Family Locator → **Allow background activity** |
| **Vivo / iQOO** | Battery → Background power consumption → **Allow** |

More help for each brand: **https://dontkillmyapp.com**

---

## 🌍 How it works

```
 Sharing phone                      Firebase (your project)                Parent's phone
 ─────────────                      ───────────────────────                ──────────────
 GPS + battery  ── every ~1 min ──▶  families/<CODE>/status   ──live──▶   Map + battery
 Notifications  ◀──────────────────  families/<CODE>/messages ◀────────   Chat / quick msgs
 Fresh GPS fix  ◀──────────────────  families/<CODE>/ping     ◀────────   "Refresh now"
```

- It works in **any country** as long as both phones have internet (mobile data, Wi‑Fi or roaming).
- **No signal?** Updates are saved on the phone and uploaded when the signal returns. The parent sees "Last seen X min ago" meanwhile.
- **Phone off / battery dead?** The parent sees the last known location and battery level.
- **Data use** is tiny, a few MB per day.
- **Free plan limits** (Spark): 1 GB stored and 10 GB downloaded per month. A family uses a tiny fraction of this.

---

## 🔒 Privacy and responsible use

This app is for **families who agree** to share their location with each other.

- The person sharing **always sees a notification** while sharing is on, and can **turn it off** anytime.
- Don't install it on anyone's phone **without their knowledge and consent**. Secretly tracking someone is wrong, and in many countries it's illegal.
- The **family code is like a password**: anyone who has it can see the location. Share it only with family.
- To start over with a new code, tap **Unlink** on the sharing phone and set it up again. The old code stops receiving updates.
- To delete all data: Firebase console → Realtime Database → **Data** → hover over `families` → 🗑️ delete.

---

## 🧯 Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Spinner keeps spinning after **Set up this phone** | App can't find your database | Check `DB_URL` in `Fb.kt` is **your** link from the **Data tab** (Step 5/9) |
| "**Sign‑in failed**" | Anonymous sign‑in is off | Step 2: enable **Anonymous** |
| "**Permission denied**" | Rules not published / wrong rules | Step 4: paste the rules and click **Publish** |
| Build error: "**No matching client found for package name**" | `google-services.json` is from another app or project | Download it again for package `com.arpan.familylocator` (Step 6) and replace it |
| Build error about `google-services.json` missing | File not in `app/` or named `google-services (1).json` | Step 8: put it in `app/` and rename it exactly |
| "**No phone found with that code**" | Wrong code, or the two phones use **different Firebase projects** | Check the code in Firebase → **Data → families**. Both phones must be built with the **same** `google-services.json` and `DB_URL` |
| "Gradle project is not linked" | Opened the outer folder | Step 10: open the folder that directly contains `app` |
| Location stops after a while | Battery saver killed the app | See **Keep it reliable** above |
| Accuracy shows ±30–50 m | Phone is indoors | Normal. It's ±3–10 m outdoors |
| Map tiles don't load | No internet on the parent phone | Check data or Wi‑Fi |

---

## 📁 Project structure

```
app/src/main/java/com/arpan/familylocator/
├── App.kt              Starts Firebase (offline support) and notification channels
├── Fb.kt               ⚠️ YOUR database link + sign-in + family codes
├── MainActivity.kt     First screen: "This is MY phone" / "I'm a parent"
├── ChildActivity.kt    Sharing phone: code, on/off switch, permissions, chat
├── ParentActivity.kt   Parent phone: map, battery, address, chat
├── LocationService.kt  Background GPS + battery upload, message notifications
├── ChatController.kt   Chat list + sending messages
├── Notifs.kt           Notifications (sharing + messages)
├── ReplyReceiver.kt    "OK" / "On my way" buttons on notifications
├── BootReceiver.kt     Restarts sharing after the phone reboots
├── Battery.kt          Reads battery % and charging
└── Prefs.kt            Saves role, code and name on the phone
database.rules.json     Security rules to paste into Firebase
```

### Change the package name (optional)
If you want your own app ID (e.g. `com.yourname.familylocator`):
1. In `app/build.gradle.kts`, change both `namespace` and `applicationId`.
2. In Android Studio, right‑click the `com.arpan.familylocator` package → **Refactor → Rename**.
3. Register **that** package name in Firebase (Step 6) and download a new `google-services.json`.

---

## 🛣️ Ideas for the future
- Push notifications on the parent's phone (needs Firebase Cloud Messaging + Cloud Functions, Blaze plan)
- Location history trail for the day
- Low‑battery alert
- Safe zones (alert when arriving home or school)
- iPhone version

---

## 📝 License
MIT. Free to use and modify. Please use it responsibly and only with consent.
