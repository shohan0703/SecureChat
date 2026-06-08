# SecureChat — Setup Guide

## What's built
- **App PIN lock** — every time you open the app, it asks for a 4-digit PIN
- **Hidden chats** — a secret chat room behind a separate PIN, completely invisible in the main list
- **AES-256 encrypted messages** — all messages encrypted before storing in Firebase
- **Firebase real-time sync** — messages appear instantly on both devices
- **2-user system** — designed for just you and one friend

---

## Step 1: Firebase Setup

1. Go to https://console.firebase.google.com
2. Create a new project (e.g. "SecureChat")
3. Add an **Android app** with package name: `com.securechat`
4. Download `google-services.json` and place it in: `app/google-services.json`
5. Enable **Authentication → Email/Password**
6. Enable **Firestore Database** (start in test mode, then apply the rules below)

### Firestore Rules
Copy the contents of `firestore.rules` into your Firebase Console → Firestore → Rules tab.

---

## Step 2: Open in Android Studio

1. Open Android Studio
2. File → Open → select the `SecureChat` folder
3. Wait for Gradle sync to finish
4. Run on your Android device (API 24+)

---

## Step 3: First Use

**On your phone:**
1. Open the app → you'll be asked to set a 4-digit App PIN
2. Register with your email and a password
3. You're now on the main chat screen

**On your friend's phone:**
1. Same steps — register with their email
2. You now both have accounts

### Starting a chat
Currently the app loads existing conversations. To start the first chat, you need to create it once via Firebase console or add a "New Chat" button. Here's the quick way via Firestore:

1. In Firebase Console → Firestore, create a document in the `conversations` collection:
   - Document ID: `{yourUID}_{friendUID}` (alphabetically sorted UIDs, underscore between)
   - Fields:
     - `members`: array → [yourUID, friendUID]
     - `memberNames`: array → [yourUsername, friendUsername]
     - `hidden`: false
     - `lastMessage`: "Say hello!"
     - `lastTimestamp`: 0

Both of you will see the conversation appear automatically.

---

## Step 4: Hidden Chat

- On the main screen, tap the tiny **"•"** at the very bottom of the screen
- First time: set a separate 4-digit Hidden PIN
- Hidden chats use a completely separate conversation in Firestore with `hidden: true`
- They **never appear** in the main chat list

To create a hidden conversation, use the same Firestore method above but set `hidden: true` and append `_hidden` to the document ID.

---

## Security Notes

| Feature | Implementation |
|---|---|
| App PIN | SHA-256 hashed, stored in EncryptedSharedPreferences |
| Hidden PIN | Same — separate key |
| Message encryption | AES-256-CBC, key derived from conversation ID |
| Firestore rules | Only conversation members can read/write |
| No screenshots | Add `getWindow().setFlags(FLAG_SECURE, FLAG_SECURE)` in activities for production |

### For stronger security in production:
- Use a proper key exchange (Signal Protocol / Diffie-Hellman) instead of deriving the AES key from the conversation ID
- Generate a random IV per message and store it alongside the ciphertext
- Add `FLAG_SECURE` to prevent screenshots
- Add biometric unlock as an alternative to PIN

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
│   ├── ChatActivity.java          ← 1-on-1 chat with encryption
│   └── HiddenChatActivity.java    ← Secret chat behind PIN
├── adapters/
│   ├── MessageAdapter.java
│   └── ConversationAdapter.java
├── models/
│   ├── Message.java
│   └── Conversation.java
└── utils/
    ├── CryptoUtils.java           ← AES-256 encrypt/decrypt
    └── PinManager.java            ← Secure PIN storage
```
