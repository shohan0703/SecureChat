# SecureChat v5 — Setup Guide

## What's built
- **App PIN lock** — every time you open the app, it asks for a 4-digit PIN
- **Hidden chats** — a secret chat room behind a separate PIN, completely invisible in the main list
- **AES-256 encrypted messages** — all messages encrypted before storing in Firebase
- **Firebase real-time sync** — messages appear instantly on both devices
- **Voice messages** — hold mic button to record, release to send
- **Reply to message** — long press any message → Reply (WhatsApp style)
- **Reactions** — long press any message → pick an emoji
- **Profile photo** — change profile picture (uploaded to Cloudinary)
- **multi-user system** — design

---

## Storage: Cloudinary (not Firebase Storage)

Firebase Storage requires a paid plan in Bangladesh. This app uses **Cloudinary** instead — completely free, no card needed.

| Setting | Value |
|---|---|
| Cloud Name | `dpnfeua6f` |
| Upload Preset | `securechat_upload` (Unsigned) |

Images and voice messages are uploaded to Cloudinary. Only the URL is saved in Firestore.

---

## Step 1: Firebase Setup

1. Go to https://console.firebase.google.com
2. Create a new project (e.g. "SecureChat")
3. Add an **Android app** with package name: `com.securechat`
4. Download `google-services.json` and place it in: `app/google-services.json`
5. Enable **Authentication → Email/Password**
6. Enable **Firestore Database** (start in test mode, then apply the rules below)

### Firestore Rules
Copy the contents of `firestore.rules` into Firebase Console → Firestore → Rules tab.

> **Note:** Firebase Storage is NOT used in this version. No Storage setup needed.

---

## Step 2: Open in Android Studio

1. Open Android Studio
2. File → Open → select the `SecureChat_v3` folder
3. Wait for Gradle sync to finish
4. Run on your Android device (API 24+)

---

## Step 3: First Use

**On your phone:**
1. Open the app → set a 4-digit App PIN
2. Register with your email and password
3. You're now on the main chat screen

**On your friend's phone:**
1. Same steps — register with their email



## Step 4: Hidden Chat

- On the main screen, tap the tiny **"•"** at the very bottom
- First time: set a separate 4-digit Hidden PIN
- Hidden chats are stored with `hidden: true` and `_hidden` suffix in the document ID
- They **never appear** in the main chat list

---

## Step 5: Voice Messages

- In chat screen, **hold** the mic button 🎤 to record
- **Release** to send, **cancel** to discard (swipe away or release within 1 second)
- Voice files are uploaded to Cloudinary as `.m4a`

---

## Step 6: Reply to Message

- **Long press** any message bubble
- Select **"Reply"** from the menu
- A reply bar appears at the bottom showing the quoted message
- Tap ✕ to cancel the reply

---

## Security Notes

| Feature | Implementation |
|---|---|
| App PIN | SHA-256 hashed, stored in EncryptedSharedPreferences |
| Hidden PIN | Same — separate key |
| Message encryption | AES-256-CBC, key derived from conversation ID |
| Reply preview | Also AES-256 encrypted before storing |
| Firestore rules | Only conversation members can read/write |
| File storage | Cloudinary (unsigned preset, authenticated by app) |

---

## Project Structure

```
app/src/main/java/com/securechat/
├── activities/
│   ├── AppLockActivity.java       ← PIN screen (launches first)
│   ├── SetupPinActivity.java      ← First-time PIN setup
│   ├── LoginActivity.java         ← Firebase login
│   ├── RegisterActivity.java      ← Create account
│   ├── MainActivity.java          ← Chat list
│   ├── ChatActivity.java          ← Chat: text, image, voice, reply
│   ├── HiddenChatActivity.java    ← Secret chat behind PIN
│   ├── NewChatActivity.java       ← Start new conversation
│   ├── ProfileActivity.java       ← Change profile photo (Cloudinary)
│   └── ChatPinActivity.java       ← Hidden chat PIN entry
├── adapters/
│   ├── MessageAdapter.java        ← Renders text/image/voice + reply preview
│   ├── ConversationAdapter.java
│   └── UserSearchAdapter.java
├── models/
│   ├── Message.java               ← Includes voice, reply, reaction fields
│   ├── Conversation.java
│   └── User.java
└── utils/
    ├── CryptoUtils.java           ← AES-256 encrypt/decrypt
    └── PinManager.java            ← Secure PIN storage
```

---

## Changelog

### v5
- Replaced Firebase Storage with **Cloudinary** (Bangladesh billing restriction workaround)
- Profile photo upload via Cloudinary
- Voice message upload via Cloudinary

### v4
- Added **voice messages** (hold to record, MediaRecorder → m4a)
- Added **reply to message** (WhatsApp style, encrypted preview)
- Added **emoji reactions** (long press → pick emoji)
- Firebase Storage used for file uploads (replaced in v5)

### v3
- Initial release
- App PIN lock, Hidden chats, AES-256 encryption, Firebase Firestore sync
