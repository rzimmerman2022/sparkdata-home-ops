# Firestore Data Structure - December 2025 Implementation

**Project:** SparkData Cleaning Checklists
**Created:** 2025-12-10
**Purpose:** Document the ACTUAL Firestore structure used by the Dec 2025 multi-cleaner sync system

---

## 🚨 CRITICAL: Two Different Implementations

There are **TWO SEPARATE Firestore structures** in this project:

### OLD Structure (November 2025 - Single Collection)

- **Collection:** `cleaning-sessions`
- **Used by:** `Deep_Clean_*_DEV.html` (deprecated)
- **Storage:** All data in single document with nested `tasks` object
- **Status:** ❌ Deprecated

### NEW Structure (December 2025 - Multi-Collection)

- **Collections:** `sessions/dec10-2025/cleaners` and `sessions/dec10-2025/progress`
- **Used by:** `Deep_Clean_2025-12-10_Firestore_Sync.html` (CURRENT)
- **Dashboard:** `dashboard.html` (✅ WORKING - updated for new structure)
- **Storage:** Sessions and tasks separated into different collections
- **Status:** ✅ Active and working

---

## Current Firestore Structure (Dec 2025)

### Collection Path Constants

**Defined in:** `Deep_Clean_2025-12-10_Firestore_Sync.html` (line 1731-1732)

```javascript
const COLLECTION_PATH = 'sessions/dec10-2025/cleaners';
const PROGRESS_PATH = 'sessions/dec10-2025/progress';
```

---

## Collection 1: Session Metadata

**Path:** `sessions/dec10-2025/cleaners/{sessionId}`

**Document ID Format:** `session_{timestamp}_{random}`
**Example:** `session_1733850000_k8j2h4n9p`

### Document Structure

```javascript
{
  sessionId: "session_1733850000_k8j2h4n9p",
  name: "Maria",                          // Cleaner's name
  startTime: Timestamp(2025-12-10 09:35:00),
  endTime: Timestamp | null,              // null if still active
  status: "active" | "completed",
  tasksCompleted: 26,                     // Count only, not individual task states
  createdAt: Timestamp(server),
  lastUpdated: Timestamp(server)
}
```

### Key Differences from Old Structure:
- ✅ Has cleaner `name` field (for multi-cleaner tracking)
- ✅ Has `createdAt` timestamp
- ❌ NO `tasks` object (moved to separate collection)
- ❌ NO `progress` percentage (must be calculated)
- ❌ NO `totalCount` (must be counted from progress collection)
- ❌ NO `notes` field (not implemented yet)

---

## Collection 2: Task Progress

**Path:** `sessions/dec10-2025/progress/{taskId}`

**Document ID:** Task ID from HTML (e.g., `mb-shower`, `kitchen-sink`)

### Document Structure

```javascript
{
  completed: true | false,
  completedBy: "Maria" | null,           // Name of cleaner who checked it
  timestamp: Timestamp(server)            // When it was last modified
}
```

### Important Notes:
- **Shared across all cleaners** - ONE document per task, NOT per session
- Multiple cleaners working simultaneously will update the SAME documents
- `completedBy` tracks who completed it (for accountability)
- Real-time sync means all cleaners see each other's progress instantly

---

## How It Works: Multi-Cleaner Real-Time Sync

### Scenario: Two Cleaners Working Simultaneously

```
Maria starts session → Creates doc: sessions/dec10-2025/cleaners/session_123_maria
Juan starts session  → Creates doc: sessions/dec10-2025/cleaners/session_456_juan

Maria checks "Kitchen - Wipe counters"
  ↓
  Updates: sessions/dec10-2025/progress/kitchen-counters
    { completed: true, completedBy: "Maria", timestamp: now }
  ↓
  Juan's screen updates instantly (via real-time listener)
  ↓
  Juan sees the checkbox is already checked, skips that task

Juan checks "Living Room - Vacuum"
  ↓
  Updates: sessions/dec10-2025/progress/living-vacuum
    { completed: true, completedBy: "Juan", timestamp: now }
  ↓
  Maria's screen updates instantly
  ↓
  Maria sees Juan's progress
```

### Real-Time Listeners

**Progress Listener** (line 1982-2001):
```javascript
progressListener = db.collection(PROGRESS_PATH)
  .onSnapshot(snapshot => {
    snapshot.docChanges().forEach(change => {
      if (change.type === 'added' || change.type === 'modified') {
        const taskId = change.doc.id;
        const data = change.doc.data();

        // Update checkbox to match Firestore state
        const checkbox = document.getElementById(taskId);
        if (checkbox) {
          checkbox.checked = data.completed || false;
          updateProgress();
        }
      }
    });
  });
```

**Cleaners Listener** (line 2003-2012):
```javascript
cleanersListener = db.collection(COLLECTION_PATH)
  .where('status', 'in', ['active', 'completed'])
  .orderBy('startTime', 'desc')
  .onSnapshot(snapshot => {
    updateActiveCleaners(snapshot.docs);
  });
```

---

## Sync Flow

### When Checkbox is Checked

```
1. User clicks checkbox
   ↓
2. handleCheckboxChange() fires immediately (line 2059)
   ↓
3. updateProgress() - Update local UI
   ↓
4. syncToFirestore() - Immediate sync to cloud (line 2076)
   ↓
5. Batch write to Firestore:
   - Update progress/{taskId} with { completed, completedBy, timestamp }
   - Update cleaners/{sessionId} with { tasksCompleted, lastUpdated }
   ↓
6. Firestore broadcasts change to ALL listeners
   ↓
7. Other cleaners' screens update instantly
```

### Sync Function

**Code:** `Deep_Clean_2025-12-10_Firestore_Sync.html` (line 2076-2115)

```javascript
async function syncToFirestore() {
  if (!currentSessionId) return;

  try {
    const checkboxes = document.querySelectorAll('.task-checkbox');
    const batch = db.batch();
    let completedCount = 0;

    // Update each task in progress collection
    checkboxes.forEach(checkbox => {
      const taskId = checkbox.id;
      const isCompleted = checkbox.checked;

      if (isCompleted) completedCount++;

      const taskRef = db.collection(PROGRESS_PATH).doc(taskId);
      batch.set(taskRef, {
        completed: isCompleted,
        completedBy: isCompleted ? currentCleanerName : null,
        timestamp: firebase.firestore.FieldValue.serverTimestamp()
      }, { merge: true });
    });

    // Update session document
    const sessionRef = db.collection(COLLECTION_PATH).doc(currentSessionId);
    batch.update(sessionRef, {
      tasksCompleted: completedCount,
      lastUpdated: firebase.firestore.FieldValue.serverTimestamp()
    });

    await batch.commit();
    console.log('Synced to Firestore:', completedCount, 'tasks completed');

  } catch (error) {
    console.error('Error syncing to Firestore:', error);
    if (error.code === 'unavailable') {
      console.log('Offline - changes will sync when connection is restored');
    }
  }
}
```

---

## Session Lifecycle

### 1. Page Load
```javascript
// Check for existing session in localStorage
const savedSessionId = localStorage.getItem('firestore-session-id');
const savedCleanerName = localStorage.getItem('firestore-cleaner-name');
const savedStartTime = localStorage.getItem('firestore-start-time');

if (savedSessionId) {
  // Resume existing session
  currentSessionId = savedSessionId;
  currentCleanerName = savedCleanerName;
  sessionStartTime = new Date(savedStartTime);

  // Hide modal, show session info
  initializeSession();
}
```

### 2. Start New Session
```javascript
async function startNewSession() {
  const cleanerName = document.getElementById('cleanerNameInput').value.trim();

  currentCleanerName = cleanerName;
  sessionStartTime = new Date();
  currentSessionId = `session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;

  // Save to localStorage
  localStorage.setItem('firestore-session-id', currentSessionId);
  localStorage.setItem('firestore-cleaner-name', currentCleanerName);
  localStorage.setItem('firestore-start-time', sessionStartTime.toISOString());

  // Create session in Firestore
  await db.collection(COLLECTION_PATH).doc(currentSessionId).set({
    sessionId: currentSessionId,
    name: currentCleanerName,
    startTime: firebase.firestore.Timestamp.fromDate(sessionStartTime),
    endTime: null,
    status: 'active',
    tasksCompleted: 0,
    createdAt: firebase.firestore.FieldValue.serverTimestamp(),
    lastUpdated: firebase.firestore.FieldValue.serverTimestamp()
  });

  initializeSession();
}
```

### 3. End Session
```javascript
async function endSession() {
  // Final sync
  await syncToFirestore();

  // Mark session as completed
  await db.collection(COLLECTION_PATH).doc(currentSessionId).update({
    endTime: firebase.firestore.Timestamp.now(),
    status: 'completed',
    lastUpdated: firebase.firestore.FieldValue.serverTimestamp()
  });

  // Clean up
  if (progressListener) progressListener();
  if (cleanersListener) cleanersListener();

  localStorage.removeItem('firestore-session-id');
  localStorage.removeItem('firestore-cleaner-name');
  localStorage.removeItem('firestore-start-time');
}
```

---

## Migration from Old Structure

### Old vs New Comparison

| Feature | Old (cleaning-sessions) | New (sessions/dec10-2025) |
|---------|------------------------|---------------------------|
| **Collections** | 1 collection | 2 collections (cleaners + progress) |
| **Task Storage** | Nested in session doc | Separate documents per task |
| **Multi-Cleaner** | ❌ Not supported | ✅ Fully supported |
| **Real-Time Sync** | Session-level only | Task-level granularity |
| **Cleaner Names** | ❌ Not tracked | ✅ Tracked per session |
| **Who Completed** | ❌ Not tracked | ✅ completedBy field |
| **Scalability** | Limited (doc size) | Better (distributed) |

### Why the New Structure is Better

1. **Multi-Cleaner Support**: Each cleaner has their own session document
2. **Real-Time Collaboration**: Tasks are shared, so cleaners see each other's progress
3. **Better Scalability**: Tasks in separate docs, no 1MB document size limit
4. **Accountability**: Track WHO completed each task
5. **Flexible Querying**: Can query by cleaner, by task, by date range

---

## Common Queries

### Get All Active Cleaners

```javascript
const activeCleaners = await db.collection('sessions/dec10-2025/cleaners')
  .where('status', '==', 'active')
  .orderBy('startTime', 'desc')
  .get();

activeCleaners.forEach(doc => {
  const data = doc.data();
  console.log(`${data.name} started at ${data.startTime.toDate()}`);
  console.log(`  Completed ${data.tasksCompleted} tasks`);
});
```

### Get All Task Progress

```javascript
const progress = await db.collection('sessions/dec10-2025/progress').get();

let completedCount = 0;
progress.forEach(doc => {
  const data = doc.data();
  if (data.completed) {
    completedCount++;
    console.log(`${doc.id} completed by ${data.completedBy}`);
  }
});

console.log(`Total: ${completedCount}/${progress.size} tasks completed`);
```

### Calculate Overall Progress Percentage

```javascript
async function calculateProgress() {
  const progressSnapshot = await db.collection('sessions/dec10-2025/progress').get();

  let completed = 0;
  progressSnapshot.forEach(doc => {
    if (doc.data().completed) completed++;
  });

  const percentage = (completed / progressSnapshot.size) * 100;
  console.log(`Progress: ${Math.round(percentage)}%`);
  return percentage;
}
```

### Get Session by Cleaner Name

```javascript
const marias = await db.collection('sessions/dec10-2025/cleaners')
  .where('name', '==', 'Maria')
  .where('status', '==', 'active')
  .get();

if (!marias.empty) {
  const data = marias.docs[0].data();
  console.log('Maria is currently working');
  console.log(`Started: ${data.startTime.toDate()}`);
}
```

---

## Troubleshooting

### Dashboard Shows 0/0 for All Sections

**Status:** ✅ **RESOLVED** - Dashboard has been updated to read from the new structure.

**Previous Problem:** Dashboard was reading from `cleaning-sessions` collection (old structure)

**Solution Applied:** Dashboard now correctly reads from `sessions/dec10-2025/cleaners` and `sessions/dec10-2025/progress`

### Can't See Other Cleaners' Progress

**Problem:** Real-time listeners not set up correctly

**Check:**

1. Verify `setupProgressListener()` was called (line 1982)
2. Check browser console for listener errors
3. Verify Firestore rules allow read access

### Tasks Not Syncing

**Problem:** Check sync status indicator

**Debug:**

```javascript
// In browser console
console.log('Session ID:', currentSessionId);
console.log('Cleaner name:', currentCleanerName);

// Force sync
syncToFirestore();
```

---

## Firebase Console Links

**Project Console:** https://console.firebase.google.com/project/sparkdata-cleaning-checklists/overview

**Firestore Database:** https://console.firebase.google.com/project/sparkdata-cleaning-checklists/firestore

**View Sessions:** https://console.firebase.google.com/project/sparkdata-cleaning-checklists/firestore/databases/-default-/data/~2Fsessions~2Fdec10-2025~2Fcleaners

**View Progress:** https://console.firebase.google.com/project/sparkdata-cleaning-checklists/firestore/databases/-default-/data/~2Fsessions~2Fdec10-2025~2Fprogress

---

## Related Files

| File | Purpose | Status |
|------|---------|--------|
| `Deep_Clean_2025-12-10_Firestore_Sync.html` | Current multi-cleaner checklist | ✅ WORKING |
| `dashboard.html` | Homeowner progress dashboard | ✅ WORKING (updated for new structure) |
| `cleanup-sessions.html` | Admin tool to close old/duplicate sessions | ✅ WORKING |
| `debug-firestore.html` | Debug tool to view all Firestore sessions | ✅ WORKING |
| `docs/firebase/README.md` | Old structure docs | ⚠️ OUTDATED |
| `docs/firebase/ARCHITECTURE.md` | Old structure design | ⚠️ OUTDATED |
| `docs/firebase/API_REFERENCE.md` | Old structure API | ⚠️ OUTDATED |

---

## Dashboard Features (Updated Dec 2025)

The dashboard has been fully updated and now includes:

1. ✅ **New collection paths** - Reads from `sessions/dec10-2025/cleaners` and `sessions/dec10-2025/progress`

2. ✅ **Multi-cleaner support** - Shows all active cleaners working simultaneously

3. ✅ **Real-time progress calculation** - Calculates progress from the progress collection (not session doc)

4. ✅ **Active cleaner display** - Shows names of all cleaners currently working

5. ✅ **Section breakdowns** - Dynamically loads checklist structure and shows progress per section

6. ✅ **Auto-discovery** - Automatically finds the most recent active session if no session ID is provided in the URL

---

**Last Updated:** 2025-12-10
**Structure Version:** 2.0 (Multi-Collection)
**Implementation:** December 2025 Firestore Sync
