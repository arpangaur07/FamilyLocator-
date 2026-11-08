# Family Locator

An Android app with two modes:

- **My phone** (Arpan): shares precise GPS location and battery level in the background, with a notification showing while it shares. It gets Mom's messages as notifications that have quick-reply buttons.
- **Parent** (Mom): shows your live spot on a map (OpenStreetMap, free) with an accuracy circle and the street address. Also shows your battery % and whether it's charging, how long ago it last updated, a **Refresh now** button, **Directions**, a **Google Maps** button that opens your exact spot, and a chat with quick messages ("Come home", "Call me", …).

It works anywhere in the world as long as both phones have mobile data or Wi-Fi. It uses Firebase (free tier) for the data.

---

## One-time setup (about 30 minutes)

### 1. Firebase (free)
1. Go to https://console.firebase.google.com, then **Add project** (Analytics can be off).
2. **Build → Authentication → Get started → Sign-in method → Anonymous → Enable.**
3. **Build → Realtime Database → Create database**. Pick a location (e.g. *asia-southeast1*) and start in **locked mode**.
4. Open the **Rules** tab, paste the contents of `database.rules.json` and click **Publish**.
5. **Project settings (gear) → Your apps → Android icon**:
   - Package name: `com.arpan.familylocator`
   - Download **google-services.json** and put it in the `app/` folder.
   - ⚠️ Create the database (step 3) *before* you download this file, so the database URL is included in it.

### 2. Map
No API key or billing needed. The in-app map uses **OpenStreetMap** (free), and the **Google Maps** button opens the exact spot in the Google Maps app.

### 3. Build and install
1. Open the `FamilyLocator` folder in **Android Studio** and let Gradle sync.
2. Plug in a phone with USB debugging on, then press **Run ▶**. Or use **Build → Build APK(s)** and send the APK to the other phone.
3. Install the **same app on both phones**.

---

## Using it

**On your phone**
1. Choose **This is MY phone**, type your name and tap **Set up this phone**.
2. Allow: precise location → notifications → **"Allow all the time"** → no battery restrictions.
3. You'll see a family code like `K7PQX-9M3RT`. Tap **Share** to send it to Mom.

**On Mom's phone**
1. Choose **I'm a parent**, enter her name and the code, then tap **Connect**.
2. Done. The map opens right on your location.

---

## Keeping it reliable (important on Xiaomi / Realme / Oppo / Vivo / Samsung)
Some phone brands kill background apps to save battery. On your phone:
- **Settings → Apps → Family Locator → Battery → Unrestricted / No restrictions**
- On Xiaomi/Redmi/POCO also turn on **Autostart** for the app.
- Don't swipe-clear it from Recents on phones that treat that as "force stop".

## How it behaves
| Thing | Behaviour |
|---|---|
| Location updates | Every ~1 minute (faster while moving), GPS high accuracy |
| Refresh now | Your phone grabs a fresh GPS fix within a few seconds (if online) |
| Phone offline / no signal | Mom sees "Last seen X min ago"; queued updates upload when signal returns |
| Phone off / battery dead | Last known location + battery stays visible |
| Reboot | Sharing restarts automatically (needs "Allow all the time") |
| Sharing turned off | Mom sees "Location sharing is turned off on their phone" |
| Data use | Very small, roughly a few MB per day |

## Limits
- Mom only gets your replies while her app is open, because there are no push notifications for the parent side. Push would need Firebase Cloud Messaging plus a Cloud Function, which requires the paid Blaze plan.
- Indoors, accuracy can drop to ±20–50 m. Outdoors it's usually ±3–10 m.
- Security: the 10-character code works like a password, so share it only with family. Anyone with the code can see the location.

## Files
```
app/src/main/java/com/arpan/familylocator/
  MainActivity.kt     setup screen (choose role, pair with code)
  ChildActivity.kt    your phone: code, sharing switch, permissions, chat
  ParentActivity.kt   Mom's phone: map, battery, address, chat
  LocationService.kt  background GPS + battery upload, message notifications
  ChatController.kt   shared chat UI + Firebase messages
  Notifs.kt, ReplyReceiver.kt, BootReceiver.kt, Battery.kt, Prefs.kt, Fb.kt, App.kt
database.rules.json   paste into Firebase → Realtime Database → Rules
```
