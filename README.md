# The Oxygen

বৃক্ষ রোপণ কর্মসূচি — পাবলিক ওয়েবসাইট + এডমিন অ্যাপ

## প্রজেক্ট স্ট্রাকচার

```
public/          → পাবলিক ওয়েবসাইট (GitHub Pages এ হোস্ট হবে)
admin-app/       → এডমিন প্যানেল (Android APK, Termux দিয়ে বিল্ড হবে)
```

দুটোই একই Firebase Realtime Database ব্যবহার করে, তাই এডমিন যা যোগ করবে
সাথে সাথে পাবলিক ওয়েবসাইটে দেখা যাবে।

---

## ১) পাবলিক ওয়েবসাইট লাইভ করা (GitHub Pages)

1. GitHub-এ একটা নতুন রিপোজিটরি বানান (যেমন: `the-oxygen`)
2. `public/` ফোল্ডারের ভেতরের ফাইলগুলো (`index.html`, `logo.png`) রিপোর রুটে আপলোড করুন
   (অথবা পুরো রিপো `public` ফোল্ডার রেখেই GitHub Pages-কে বলে দিতে পারেন `/public` থেকে সার্ভ করতে)
3. রিপোর **Settings → Pages** এ যান → Source: **Deploy from a branch** → branch: `main`, folder: `/ (root)` অথবা `/public` — সিলেক্ট করে Save করুন
4. কিছুক্ষণ পর `https://<আপনার-ইউজারনেম>.github.io/the-oxygen/` এই লিংকে সাইট লাইভ হয়ে যাবে

### bKash/Nagad নম্বর বসানো
`public/index.html` ফাইলে `01XXXXXXXXX` লেখা দুটো জায়গা খুঁজে বের করে
(সার্চ করুন `pay-box`) আসল bKash ও Nagad নম্বর বসিয়ে দিন।

---

## ২) এডমিন অ্যাপ (APK) Termux দিয়ে বিল্ড করা

### ধাপ ১ — Termux প্রস্তুত করা
> ⚠️ Play Store-এর Termux না, **F-Droid** থেকে নামানো Termux ব্যবহার করুন
> (Play Store ভার্সনে `openjdk-17` প্যাকেজ কাজ করে না)

```bash
pkg update -y && pkg upgrade -y
pkg install openjdk-17 wget unzip zip git aapt aapt2 -y
```

### ধাপ ২ — Android SDK কমান্ডলাইন টুলস

```bash
mkdir -p $HOME/android-sdk && cd $HOME/android-sdk
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip -O cmdline-tools.zip
unzip cmdline-tools.zip
mkdir -p cmdline-tools/latest
mv cmdline-tools/bin cmdline-tools/lib cmdline-tools/NOTICE.txt cmdline-tools/source.properties cmdline-tools/latest/ 2>/dev/null
```

`~/.bashrc` ফাইলে যোগ করুন (`nano ~/.bashrc` দিয়ে খুলে নিচের লাইনগুলো শেষে বসান):

```bash
export ANDROID_HOME=$HOME/android-sdk
export PATH=$ANDROID_HOME/cmdline-tools/latest/bin:$PATH
export PATH=$ANDROID_HOME/platform-tools:$PATH
```

তারপর:
```bash
source ~/.bashrc
yes | sdkmanager --licenses
sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"
```

### ধাপ ৩ — Gradle ইনস্টল

```bash
cd $HOME
wget https://services.gradle.org/distributions/gradle-8.7-bin.zip
unzip gradle-8.7-bin.zip -d $HOME/gradle-dist
echo 'export PATH=$HOME/gradle-dist/gradle-8.7/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
gradle -v
```

### ধাপ ৪ — aapt2 ফিক্স (Termux-এ পরিচিত সমস্যা)

Gradle যে aapt2 ডাউনলোড করে সেটা Termux-এ চলে না, তাই লোকাল aapt2 ব্যবহার
করতে হবে:

```bash
which aapt2
```

এই কমান্ড যে পাথ দেখাবে সেটা `admin-app/gradle.properties` ফাইলের শেষ লাইনে
বসান (কমেন্ট `#` চিহ্নটা তুলে দিন):

```
android.aapt2FromMavenOverride=/data/data/com.termux/files/usr/bin/aapt2
```

### ধাপ ৫ — প্রজেক্ট বিল্ড করা

এই পুরো `admin-app` ফোল্ডারটা আপনার ফোনে (Termux-এর হোম ডিরেক্টরিতে বা
`storage/shared` এ) কপি করুন। তারপর:

```bash
cd admin-app
gradle assembleDebug --no-daemon
```

বিল্ড শেষ হলে APK পাবেন এখানে:
```
admin-app/app/build/outputs/apk/debug/app-debug.apk
```

এটা `termux-setup-storage` চালিয়ে `/storage/emulated/0/Download/` এ কপি
করে ফোনের ফাইল ম্যানেজার দিয়ে ইনস্টল করুন:

```bash
termux-setup-storage
cp app/build/outputs/apk/debug/app-debug.apk /storage/emulated/0/Download/
```

> প্রতিবার নতুন বিল্ড করার আগে `gradle clean assembleDebug --no-daemon`
> চালালে পুরনো ক্যাশ-জনিত সমস্যা কম হয়।

---

## ৩) এডমিন পাসকোড সেট করা

Firebase Console → **Authentication → Users → Add user**:
- Email: `admin@theoxygen.app`
- Password: আপনার শেয়ার্ড এডমিন পাসকোড (কমপক্ষে ৬ ক্যারেক্টার)

সব এডমিন এই একই পাসকোড অ্যাপে বসিয়ে লগইন করবেন।

---

## ৪) Firestore/Realtime Database Rules

Realtime Database → Rules ট্যাবে গিয়ে এটা বসান:

```json
{
  "rules": {
    "trees": { ".read": true, ".write": "auth != null" },
    "upcoming": { ".read": true, ".write": "auth != null" },
    "donations": {
      ".read": "auth != null",
      "$donationId": { ".write": "!data.exists() || auth != null" }
    }
  }
}
```

Storage → Rules ট্যাবে:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

---

## অ্যাপের ফিচার সারসংক্ষেপ

**পাবলিক ওয়েবসাইট**
- রোপিত গাছের তালিকা (ছবি, নাম, ঠিকানা, প্রজাতি) + সার্চ
- শীঘ্রই যাদের নামে গাছ লাগবে তার তালিকা
- অনুদান ফর্ম (bKash/Nagad ট্রানজেকশন আইডি জমা দেওয়া)

**এডমিন অ্যাপ (APK)**
- শেয়ার্ড পাসকোড দিয়ে লগইন
- রোপিত গাছ যোগ/সম্পাদনা/মুছে ফেলা (ছবিসহ)
- "শীঘ্রই লাগবে" তালিকা পরিচালনা, এক ক্লিকে "রোপিত" তালিকায় সরানো
- অনুদান অনুরোধ যাচাই করে কনফার্ম/রিজেক্ট করা
- স্প্ল্যাশ স্ক্রিনে লোগো + "Built by HA TECH BD"
- ব্যাক বাটন: এক পেজ পিছনে যায়, হোম পেজে ডাবল-ব্যাক করলে অ্যাপ বন্ধ হয়

## এখনো যা নিজে করতে হবে

- [ ] `public/index.html` এ আসল bKash/Nagad নম্বর বসানো
- [ ] Firebase Authentication-এ এডমিন ইউজার তৈরি করা (উপরে ৩নং ধাপ)
- [ ] Firestore/Storage Rules পাবলিশ করা (৪নং ধাপ)
- [ ] GitHub রিপো বানিয়ে `public/` ফোল্ডার Pages-এ ডিপ্লয় করা
- [ ] Termux-এ APK বিল্ড করে ফোনে ইনস্টল করা
