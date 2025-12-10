# Firebase Architecture - Technical Design

**Project:** SparkData Cleaning Checklists
**Created:** 2025-12-09

---

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    CLEANER'S CHECKLIST                       │
│  (Deep_Clean_*.html)                                         │
│                                                              │
│  User Action → localStorage (instant) → UI Update (instant) │
│                       ↓                                      │
│              Firebase Sync (every 45s)                       │
│                       ↓                                      │
│         ┌─────────────────────────────────┐                 │
│         │   Firebase Firestore (Cloud)    │                 │
│         │   Collection: cleaning-sessions  │                 │
│         │   - Session metadata            │                 │
│         │   - Task completion states       │                 │
│         │   - Progress percentage          │                 │
│         │   - Notes                        │                 │
│         └─────────────┬───────────────────┘                 │
│                       │                                      │
│                       ↓ Real-time listener (onSnapshot)     │
│         ┌─────────────────────────────────┐                 │
│         │    HOMEOWNER'S DASHBOARD         │                 │
│         │    (dashboard.html - TBD)        │                 │
│         │                                  │                 │
│         │  - Live progress bar             │                 │
│         │  - Section breakdowns            │                 │
│         │  - Task list with checkmarks     │                 │
│         │  - Session timing info           │                 │
│         │  - Updates <1 second             │                 │
│         └──────────────────────────────────┘                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Data Flow

### Session Lifecycle

```
1. PAGE LOAD
   ├─ Load checkboxes from localStorage
   ├─ Check for active session (localStorage.getItem('currentSessionId'))
   ├─ If found → Restore session UI, resume auto-sync
   └─ If not found → Show START button

2. START BUTTON CLICKED
   ├─ Generate session ID: session_{timestamp}_{random}
   ├─ Save sessionId + startTime to localStorage
   ├─ Create Firestore document:
   │   {
   │     sessionId, startTime, endTime: null,
   │     status: 'active', tasks: {}, progress: 0, ...
   │   }
   ├─ Update UI: Disable button, show "Session Active"
   └─ Start auto-sync interval (every 45s)

3. USER CHECKS BOXES
   ├─ Event: checkbox.addEventListener('change')
   ├─ Save to localStorage immediately (PRIMARY STORAGE)
   ├─ Update progress bar visually
   └─ Trigger Firebase sync (debounced/scheduled)

4. AUTO-SYNC (Every 45 seconds)
   ├─ Read all checkbox states from DOM
   ├─ Build tasks map: { taskId: { checked, lastUpdated }, ... }
   ├─ Calculate progress: completedCount / totalCount * 100
   ├─ Read notes from textarea
   ├─ Update Firestore document:
   │   db.collection('cleaning-sessions').doc(sessionId).update({
   │     tasks, progress, completedCount, notes, lastSyncTime
   │   })
   ├─ Update sync status UI: "↻ Syncing..." → "● Synced"
   └─ If offline: Show "⚠ Offline", queue for retry

5. SUBMIT BUTTON CLICKED
   ├─ Mark Firestore session complete:
   │   db.update({ endTime: now, status: 'completed' })
   ├─ Stop auto-sync interval
   ├─ Clear session from localStorage
   ├─ Send final summary to Google Sheets (existing flow)
   └─ Show summary modal

6. DASHBOARD (Real-time)
   ├─ Get session ID from URL: ?session=SESSION_ID
   ├─ Set up Firestore listener:
   │   db.doc(sessionId).onSnapshot(doc => updateUI(doc.data()))
   ├─ Firestore pushes updates whenever cleaner syncs (<1s latency)
   ├─ Update progress bars, task checkmarks, notes
   └─ Show "Last updated: X seconds ago"
```

---

## Technical Implementation

### Firebase SDK Integration

**Type:** Compat SDK (firebase-*-compat.js)
**Reason:** Vanilla JavaScript, no build tools, simpler API

**Loaded via CDN:**
```html
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>
```

**Initialization:**
```javascript
firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();
db.enablePersistence({ synchronizeTabs: true });
```

---

## Storage Strategy: Hybrid localStorage + Firestore

### Why Hybrid?

| Storage | Role | Advantages |
|---------|------|------------|
| **localStorage** | Primary storage | Instant, offline-first, no API limits |
| **Firestore** | Sync target | Real-time, cross-device, homeowner visibility |

### Data Redundancy

**All data exists in 3 places:**
1. **DOM state** (checkbox.checked) → Ephemeral, cleared on reload
2. **localStorage** (single-cleaner-{taskId}) → Persistent, survives reload
3. **Firestore** (cleaning-sessions/{sessionId}) → Cloud, cross-device

**Benefit:** Triple redundancy prevents data loss

---

## Auto-Sync Design

### Throttling Strategy

**Interval:** 45 seconds (balance between real-time and API costs)

**Why 45 seconds?**
- Fast enough: Homeowner sees updates within 1 minute
- Cost effective: 3-hour session = 8 syncs (vs 180 syncs at 1s intervals)
- Free tier safe: 52 writes/month vs 20,000 limit
- Network efficient: Batches multiple checkbox changes into one sync

**Implementation:**
```javascript
const SYNC_INTERVAL = 45000; // 45 seconds

let syncInterval = setInterval(() => {
  syncToFirestore();  // Sync all current state
}, SYNC_INTERVAL);
```

### Debouncing vs Throttling

**Throttling (Used):** Sync every 45s regardless of activity
- Pros: Predictable, regular updates
- Cons: Slight delay if user just checked a box

**Debouncing (Not Used):** Sync X seconds after last change
- Pros: Lower API usage if cleaner works in bursts
- Cons: Unpredictable timing, homeowner may see stale data

**Decision:** Throttling chosen for predictable homeowner experience

---

## Offline Handling

### Firebase Offline Persistence

**Enabled:**
```javascript
db.enablePersistence({ synchronizeTabs: true })
```

**What it does:**
- Caches Firestore data in IndexedDB
- Queues writes when offline
- Auto-syncs when connection restored
- Synchronizes across browser tabs

### Offline Workflow

```
1. Internet connection lost
   ↓
2. window.addEventListener('offline') fires
   ↓
3. Update UI: "⚠ Offline" indicator
   ↓
4. localStorage continues working normally
   ↓
5. Firestore writes queued locally
   ↓
6. Internet connection restored
   ↓
7. window.addEventListener('online') fires
   ↓
8. Firebase auto-syncs queued writes
   ↓
9. Update UI: "● Synced"
```

**User Impact:** Zero. Cleaner continues checking boxes, homeowner sees updates when connection returns.

---

## Session ID Generation

### Format
```
session_{timestamp}_{random}
```

**Example:** `session_1733760000_k8j2h4n9p`

**Components:**
- `session_` - Prefix for easy identification
- `1733760000` - Unix timestamp (milliseconds since 1970)
- `k8j2h4n9p` - Random 9-character base-36 string

**Function:**
```javascript
function generateSessionId() {
  const timestamp = Date.now();
  const random = Math.random().toString(36).substr(2, 9);
  return `session_${timestamp}_${random}`;
}
```

**Why This Format?**
1. **Chronologically sortable** (timestamp first)
2. **Collision-resistant** (timestamp + random = virtually unique)
3. **URL-safe** (only alphanumeric + underscore)
4. **Human-readable** (can extract date from timestamp)
5. **Secure enough** (random component prevents guessing)

**Collision Probability:**
- Timestamp precision: 1 millisecond
- Random space: 36^9 = 101 trillion combinations
- **Probability of collision:** Effectively zero

---

## Security Model

### Current (MVP)

**Firestore Rules:**
```javascript
allow read, write: if true;  // Public access
```

**Why This Is Acceptable:**
1. Home cleaning app (not financial/healthcare data)
2. Session IDs are unpredictable (timestamp + random)
3. Worst case: Someone sees cleaning progress (low risk)
4. No PII (personally identifiable information)
5. No business logic in client (can't "hack" cleaning status)

### Attack Vectors (Low Risk)

| Attack | Risk | Mitigation |
|--------|------|------------|
| Guess session ID | Very Low | 101 trillion combinations |
| View someone's session | Low | Need exact session ID |
| Modify session data | Low | Only cleaning data, no financial impact |
| Delete sessions | Medium | Could be annoying but no data loss (localStorage backup) |

### Production Hardening (Optional)

**If needed in future:**
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /cleaning-sessions/{sessionId} {
      // Rate limit: max 10 writes per minute per session
      allow write: if request.time > resource.data.lastWriteTime + duration.value(6, 's');

      // Validate structure
      allow create: if request.resource.data.keys().hasAll(['sessionId', 'startTime', 'status']);

      // Prevent status tampering
      allow update: if request.resource.data.status == resource.data.status
                    || (resource.data.status == 'active' && request.resource.data.status == 'completed');
    }
  }
}
```

---

## Performance Considerations

### Document Size

**Typical session document:**
- 120 tasks × ~50 bytes each = ~6 KB
- Metadata: ~500 bytes
- Notes: ~500 bytes (average)
- **Total: ~7 KB per session**

**Firestore Limits:**
- Max document size: 1 MB
- **Headroom:** 7 KB / 1 MB = 0.7% of limit

**Conclusion:** Plenty of room for growth

### Network Overhead

**Per sync:**
- Upload: ~7 KB (full document)
- Download (dashboard): ~7 KB

**Per 3-hour session:**
- Syncs: 8 (every 45s)
- Total upload: 56 KB
- Dashboard continuous: ~7 KB initial + delta updates

**Monthly (4 sessions):**
- Total bandwidth: ~500 KB
- **Free tier limit:** 10 GB
- **Usage:** 0.005% of limit

---

## Scalability

### Current Design Limits

| Metric | Current | Limit | Notes |
|--------|---------|-------|-------|
| Tasks per session | 120 | ~20,000 | Document size limited |
| Concurrent sessions | 1-2 | ~100 | Free tier writes |
| Dashboard viewers | 1-5 | ~1,000 | Real-time connections |
| Session history | Unlimited | 1 GB storage | ~140,000 sessions |

### Scaling Strategies (If Needed)

**1. Partition large checklists:**
```
cleaning-sessions/{sessionId}/tasks/{taskId}
  - Subcollections for >500 tasks
  - Avoids document size limit
```

**2. Archive old sessions:**
```javascript
// Move completed sessions older than 30 days
status == 'completed' && completedAt < 30_days_ago
  → Move to 'cleaning-sessions-archive' collection
```

**3. Upgrade to Blaze plan:**
- Still pay-as-you-go
- Remove free tier limits
- ~$5-10/month for moderate usage

---

## Error Handling

### Sync Failures

**Types of errors:**
1. **Network timeout** → Retry automatically (Firebase handles)
2. **Permission denied** → Show error, check Firestore rules
3. **Quota exceeded** → Stop syncing, alert user
4. **Invalid data** → Log error, continue with localStorage

**Strategy: Graceful Degradation**
```javascript
try {
  await syncToFirestore();
  updateSyncStatus('success');
} catch (error) {
  console.error('Sync failed:', error);
  updateSyncStatus('error');
  // localStorage still works, no data loss
}
```

---

## Monitoring & Debugging

### Firebase Console Metrics

**Available Metrics:**
- Reads/writes/deletes per day
- Document count
- Storage size
- Bandwidth usage
- Error rates

**Access:** https://console.firebase.google.com/project/sparkdata-cleaning-checklists/usage

### Browser Console Logging

**Current logs:**
```
"Session started: session_..."
"Firestore session created: session_..."
"Auto-sync started (every 45 seconds)"
"Synced to Firestore: { progress: 45, completedCount: 54 }"
"Session completed: session_..."
```

**Enable detailed logging:**
```javascript
firebase.firestore.setLogLevel('debug');
```

---

## Future Enhancements

### Planned Features

1. **Real-time Dashboard** (Next)
   - Live progress visualization
   - Mobile-responsive
   - URL: `dashboard.html?session=SESSION_ID`

2. **Push Notifications**
   - Firebase Cloud Messaging
   - Notify homeowner when session starts/completes

3. **Photo Uploads**
   - Firebase Storage integration
   - Before/after photos

4. **Analytics**
   - Average completion time
   - Most frequently incomplete tasks
   - Cleaner performance tracking

5. **Multiple Cleaners**
   - Real-time collaboration
   - See who's working on what
   - Conflict resolution

---

## Related Documentation

- **Quick Access:** `docs/firebase/README.md`
- **API Reference:** `docs/firebase/API_REFERENCE.md`
- **Dashboard Guide:** `docs/firebase/DASHBOARD_GUIDE.md` (TBD)
- **Implementation:** `Deep_Clean_*_DEV.html` lines 16-40, 1422-1598

---

**Last Updated:** 2025-12-09
**Architecture Version:** 1.0
**Next Review:** After dashboard implementation
