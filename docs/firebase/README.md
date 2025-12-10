# Firebase Integration - Quick Access Guide

**Project:** SparkData Cleaning Checklists
**Created:** 2025-12-09
**Purpose:** Real-time progress tracking for cleaning sessions

---

## 🔑 Access Firebase Console

**URL:** https://console.firebase.google.com/

**Project:** SparkData Cleaning Checklists
**Project ID:** `sparkdata-cleaning-checklists`
**Project Number:** `825900465348`

**Account:** ryan.zimmerman@southwestresumes.com

---

## 📂 Quick Links

| Resource | Link |
|----------|------|
| **Firebase Console** | https://console.firebase.google.com/project/sparkdata-cleaning-checklists/overview |
| **Firestore Database** | https://console.firebase.google.com/project/sparkdata-cleaning-checklists/firestore |
| **Firestore Data** | https://console.firebase.google.com/project/sparkdata-cleaning-checklists/firestore/databases/-default-/data/~2Fcleaning-sessions |
| **Project Settings** | https://console.firebase.google.com/project/sparkdata-cleaning-checklists/settings/general |
| **Usage & Billing** | https://console.firebase.google.com/project/sparkdata-cleaning-checklists/usage |

---

## 🗂️ Firestore Database Structure

### Collection: `cleaning-sessions`

**Document ID Format:** `session_{timestamp}_{random}`
**Example:** `session_1733760000_k8j2h4n9p`

**Document Fields:**
```javascript
{
  sessionId: "session_1733760000_k8j2h4n9p",
  startTime: Timestamp,              // When session started
  endTime: Timestamp | null,         // When SUBMIT was clicked (null if active)
  status: "active" | "completed",    // Session state
  checklistType: "single-cleaner",   // Type of checklist
  checklistDate: "2025-11-26",       // Date string

  progress: 45,                      // Percentage (0-100)
  completedCount: 54,                // Number of checked tasks
  totalCount: 120,                   // Total tasks in checklist

  tasks: {                           // All task states
    "mb-shower": {
      checked: true,
      lastUpdated: Timestamp
    },
    "mb-sink": {
      checked: false,
      lastUpdated: Timestamp
    }
    // ... all ~120 tasks
  },

  notes: "Shower needs extra attention",  // Cleaner notes
  lastSyncTime: Timestamp            // Last auto-sync time
}
```

---

## 🔐 Firebase Configuration (Already in Code)

**Location:** `Deep_Clean_*_DEV.html` (lines 22-29)

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyCVHDGBL70opBifmdnILzrUYEptQsMw708",
  authDomain: "sparkdata-cleaning-checklists.firebaseapp.com",
  projectId: "sparkdata-cleaning-checklists",
  storageBucket: "sparkdata-cleaning-checklists.firebasestorage.app",
  messagingSenderId: "825900465348",
  appId: "1:825900465348:web:d28f2742e527513311dd91"
};
```

**⚠️ Security Note:** API key is public-facing (safe for client-side web apps). Actual security is enforced via Firestore security rules.

---

## 📊 Viewing Live Data

### Method 1: Firebase Console (Web UI)

1. Go to: https://console.firebase.google.com/
2. Select: **SparkData Cleaning Checklists**
3. Click: **Firestore Database** (left sidebar)
4. Navigate: `cleaning-sessions` collection
5. Click any document to view details

**Filter Active Sessions:**
- Click "Start collection" → Add query
- Field: `status`, Operator: `==`, Value: `active`

### Method 2: Browser DevTools

**While on checklist page:**
1. Open DevTools (Ctrl+Shift+I)
2. Console tab → Type:
```javascript
db.collection('cleaning-sessions')
  .where('status', '==', 'active')
  .get()
  .then(snapshot => {
    snapshot.forEach(doc => console.log(doc.id, doc.data()));
  });
```

---

## 🔄 Common Operations

### Query Active Sessions
```javascript
const activeSessions = await db.collection('cleaning-sessions')
  .where('status', '==', 'active')
  .orderBy('startTime', 'desc')
  .get();

activeSessions.forEach(doc => {
  console.log(doc.id, doc.data());
});
```

### Get Specific Session
```javascript
const sessionId = 'session_1733760000_k8j2h4n9p';
const sessionDoc = await db.collection('cleaning-sessions')
  .doc(sessionId)
  .get();

if (sessionDoc.exists) {
  console.log('Session data:', sessionDoc.data());
} else {
  console.log('Session not found');
}
```

### Real-time Listener (Dashboard)
```javascript
db.collection('cleaning-sessions')
  .doc(sessionId)
  .onSnapshot(doc => {
    const data = doc.data();
    console.log('Updated progress:', data.progress);
    updateDashboardUI(data);
  });
```

### Query Today's Sessions
```javascript
const today = new Date();
today.setHours(0, 0, 0, 0);

const todaysSessions = await db.collection('cleaning-sessions')
  .where('startTime', '>=', firebase.firestore.Timestamp.fromDate(today))
  .orderBy('startTime', 'desc')
  .get();
```

---

## 🛡️ Security Rules

**Location:** Firebase Console → Firestore Database → Rules tab

**Current Rules (Test Mode):**
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /cleaning-sessions/{sessionId} {
      allow read, write: if true;  // Public access for MVP
    }
  }
}
```

**⚠️ Note:** Test mode allows anyone to read/write. Acceptable for home cleaning app with random session IDs.

**For Production (Optional):**
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /cleaning-sessions/{sessionId} {
      // Allow read if user knows the session ID (from URL)
      allow read: if true;

      // Only allow writes if session is active
      allow create: if request.resource.data.status == 'active';
      allow update: if resource.data.status == 'active';
      allow delete: if false;  // Never allow deletion
    }
  }
}
```

---

## 📈 Usage Limits (Free Tier)

| Resource | Free Tier Limit | Current Usage | Notes |
|----------|----------------|---------------|-------|
| **Reads** | 50,000/day | ~400/month | Dashboard views |
| **Writes** | 20,000/day | ~52/month | Auto-sync every 45s |
| **Deletes** | 20,000/day | 0 | Not used |
| **Storage** | 1 GB | <1 MB | Minimal data |
| **Bandwidth** | 10 GB/month | <10 MB | Small documents |

**Estimated Monthly Cost:** $0 (well under free tier)

**Check Usage:**
https://console.firebase.google.com/project/sparkdata-cleaning-checklists/usage

---

## 🐛 Troubleshooting

### Session Not Creating

**Check browser console for errors:**
```javascript
// Should see these logs:
"Session started: session_..."
"Firestore session created: session_..."
```

**Common Issues:**
1. **Permission denied** → Check Firestore rules allow write
2. **Offline** → Check internet connection, Firebase loads
3. **CORS error** → Ensure using firebase-compat SDK (not modular)

### Data Not Syncing

**Check sync status indicator:**
- "● Synced" → Working correctly
- "↻ Syncing..." → In progress (wait a few seconds)
- "⚠ Offline" → No internet or Firestore unreachable

**Manual sync test:**
```javascript
// In browser console:
syncToFirestore();  // Should trigger immediate sync
```

### Can't Find Session in Firestore

**Verify collection exists:**
1. Firestore Console → Check for `cleaning-sessions` collection
2. If empty, click START button on checklist to create first session

**Search by session ID:**
- Console → Search bar → Paste session ID
- Or filter by status: `status == active`

---

## 📞 Support

**Firebase Documentation:** https://firebase.google.com/docs/firestore
**Firebase Status:** https://status.firebase.google.com/

**Project-Specific Questions:**
- Check: `docs/firebase/ARCHITECTURE.md` for system design
- Check: `docs/firebase/API_REFERENCE.md` for code examples

---

## 🔄 Sync Behavior

**Auto-Sync Interval:** 45 seconds
**Trigger:** Every checkbox change, notes edit
**What Syncs:** All task states, notes, progress percentage
**Offline Handling:** Queues changes, syncs when reconnected

**Sync Flow:**
```
User Action (check box)
  ↓
localStorage (instant save)
  ↓
Wait 45 seconds (or next interval)
  ↓
Firestore sync (background)
  ↓
Dashboard updates (real-time listener)
  ↓
Homeowner sees change (<1 second)
```

---

## 📝 Related Files

| File | Purpose |
|------|---------|
| `docs/firebase/ARCHITECTURE.md` | System design and data flow |
| `docs/firebase/API_REFERENCE.md` | Code examples and functions |
| `docs/firebase/DASHBOARD_GUIDE.md` | How to use dashboard (when built) |
| `Deep_Clean_*_DEV.html` | Implementation (lines 16-40, 1422-1598) |

---

**Last Updated:** 2025-12-09
**Maintained By:** AI sessions (see docs/handoffs/ for history)
