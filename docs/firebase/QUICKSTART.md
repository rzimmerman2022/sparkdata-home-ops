# Firebase Quick Start - For AI Models

**If you're an AI model continuing this project, start here!**

---

## 🚀 Fastest Path to Understanding Firebase Setup

### 1. Firebase is Already Configured ✅

**You don't need to set up Firebase - it's done!**

**Configuration location:** `Deep_Clean_*_DEV.html` lines 16-40

**Project details:**
- Project ID: `sparkdata-cleaning-checklists`
- Account: ryan.zimmerman@southwestresumes.com
- Database: Firestore (default)
- Collection: `cleaning-sessions`

---

### 2. Access Firebase Console

**URL:** https://console.firebase.google.com/project/sparkdata-cleaning-checklists/firestore

**What to check:**
1. Click "Firestore Database" → See `cleaning-sessions` collection
2. Click on any document → View session data in real-time
3. Check "Usage" tab → Verify within free tier

**No login needed if user is already authenticated.**

---

### 3. View Live Session Data

**In browser console (on checklist page):**
```javascript
// See current session
console.log('Session ID:', currentSessionId);

// View all active sessions
db.collection('cleaning-sessions')
  .where('status', '==', 'active')
  .get()
  .then(snapshot => snapshot.forEach(doc => console.log(doc.data())));
```

**In Firebase Console:**
- Navigate to: Firestore Database → cleaning-sessions
- See all sessions in real-time
- Filter by `status == active` to see ongoing sessions

---

### 4. How It Works (30-Second Overview)

```
1. User clicks START → Session created in Firestore
2. User checks boxes → Saved to localStorage (instant)
3. Every 45 seconds → Auto-sync to Firestore
4. Dashboard (future) → Real-time listener shows updates
5. User clicks SUBMIT → Mark session complete, sync to Google Sheets
```

**Data flow:** localStorage (primary) → Firestore (sync) → Dashboard (view)

---

### 5. Key Functions (Quick Reference)

| Function | Purpose | When Called |
|----------|---------|-------------|
| `startSession()` | Create new session | START button click |
| `syncToFirestore()` | Upload current state | Every 45s |
| `loadState()` | Restore session | Page load |
| `submitSession()` | Mark complete | SUBMIT button click |

**See full API:** `docs/firebase/API_REFERENCE.md`

---

### 6. Common Tasks

#### View Session in Firestore
1. Open: https://console.firebase.google.com/project/sparkdata-cleaning-checklists/firestore
2. Click: `cleaning-sessions` collection
3. Click: Any document (session ID)
4. See: All task states, progress, notes

#### Test if Firebase is Working
1. Open `Deep_Clean_*_DEV.html` in browser
2. Open DevTools Console (Ctrl+Shift+I)
3. Click "Start Session" button
4. Check console for: "Firestore session created: session_..."
5. Go to Firestore Console → Should see new document

#### Force Immediate Sync
```javascript
// In browser console:
syncToFirestore();
```

#### Check Sync Status
```javascript
// In browser console:
console.log('Last sync:', new Date(lastSyncTime).toLocaleTimeString());
```

---

### 7. Firestore Document Structure (Quick Ref)

```javascript
{
  sessionId: "session_1733760000_abc123",
  startTime: Timestamp,
  endTime: Timestamp | null,
  status: "active" | "completed",

  progress: 45,              // Percentage
  completedCount: 54,        // Number of checked tasks
  totalCount: 120,           // Total tasks

  tasks: {
    "mb-shower": { checked: true, lastUpdated: Timestamp },
    "mb-sink": { checked: false, lastUpdated: Timestamp }
  },

  notes: "Shower needs attention",
  lastSyncTime: Timestamp
}
```

---

### 8. Troubleshooting (Quick Fixes)

#### Session Not Creating
- **Check:** Browser console for errors
- **Check:** Firestore rules allow write: `allow read, write: if true;`
- **Check:** Internet connection

#### Data Not Syncing
- **Check:** Sync status indicator shows "● Synced" (not "⚠ Offline")
- **Check:** Auto-sync running: `console.log(syncInterval)` (should not be null)
- **Try:** Force sync: `syncToFirestore()`

#### Can't Find Session
- **Check:** Firestore Console → cleaning-sessions collection
- **Check:** Session ID matches: `console.log(currentSessionId)`
- **Try:** Query by status: `db.collection('cleaning-sessions').where('status', '==', 'active').get()`

---

### 9. Files to Know About

| File | Purpose | When to Edit |
|------|---------|--------------|
| `Deep_Clean_*_DEV.html` | Implementation | Adding features |
| `docs/firebase/README.md` | Access guide | Never (reference only) |
| `docs/firebase/ARCHITECTURE.md` | System design | Understanding flow |
| `docs/firebase/API_REFERENCE.md` | Code examples | Writing new code |
| `docs/firebase/QUICKSTART.md` | This file | Quick reference |

---

### 10. Next Steps for AI Models

**If continuing this project:**

1. **Read** this file (you're doing it!)
2. **Test** Firebase connection (open DEV file, click START)
3. **View** data in Firestore Console
4. **Read** `docs/firebase/ARCHITECTURE.md` for full understanding
5. **Use** dashboard at `dashboard.html` (no session ID needed - auto-finds latest!)

**Dashboard Usage:**
- **URL:** Just `dashboard.html` - automatically shows most recent active session
- **Homeowner Experience:** Bookmark once, opens every cleaning session
- **No sharing needed** - perfect for single-homeowner use case
- **Real-time updates** via `onSnapshot()` listener
- See: `docs/firebase/API_REFERENCE.md` for implementation details

**If debugging:**
- Check: Browser console for error messages
- Check: Firestore Console → Usage tab for quota issues
- Try: Force sync manually to isolate issue

---

## 📝 Implementation Plan Reference

### Completed ✅
- Phase 1: Firebase setup (API key, SDK integration)
- Phase 2: START button & session management
- Phase 3: Real-time auto-sync (every 45s)
- Phase 4: SUBMIT button enhancement (mark complete)

### TODO 📋
- Phase 5: Dashboard creation (live progress view)
- Phase 6: Offline error handling improvements
- Phase 7: Testing & deployment

**See full plan:** `C:\Users\Ryan\.claude\plans\bubbly-snacking-bird.md`

---

## 🔗 Essential URLs (Bookmark These)

| Resource | URL |
|----------|-----|
| **Firebase Console** | https://console.firebase.google.com/project/sparkdata-cleaning-checklists |
| **Firestore Data** | https://console.firebase.google.com/project/sparkdata-cleaning-checklists/firestore/data |
| **Usage Dashboard** | https://console.firebase.google.com/project/sparkdata-cleaning-checklists/usage |
| **Security Rules** | https://console.firebase.google.com/project/sparkdata-cleaning-checklists/firestore/rules |
| **Project Settings** | https://console.firebase.google.com/project/sparkdata-cleaning-checklists/settings/general |

---

## 💡 Pro Tips for AI Models

1. **Always check Firestore Console first** when debugging - visual data beats console logs
2. **Use browser console liberally** - `currentSessionId`, `syncInterval`, `db` are all global
3. **Firebase SDK docs are your friend** - https://firebase.google.com/docs/firestore
4. **localStorage is primary storage** - Firestore is just for syncing, don't overthink it
5. **Test offline mode** - DevTools → Network → Offline to simulate poor connectivity
6. **Sessions auto-restore on reload** - Check localStorage for `currentSessionId`
7. **45-second sync is intentional** - Don't change without considering costs
8. **Security rules are permissive** - Acceptable for home cleaning app, not production finance app

---

## ⚡ Quick Command Reference

**Browser console shortcuts:**
```javascript
// View current session
currentSessionId

// Force sync now
syncToFirestore()

// Check last sync time
new Date(lastSyncTime).toLocaleTimeString()

// View all active sessions
db.collection('cleaning-sessions').where('status','==','active').get().then(s=>s.forEach(d=>console.log(d.data())))

// Get specific session
db.collection('cleaning-sessions').doc('session_1733760000_abc123').get().then(d=>console.log(d.data()))
```

---

**Last Updated:** 2025-12-09
**For:** AI models continuing this project
**Next:** Build dashboard (see plan file)
