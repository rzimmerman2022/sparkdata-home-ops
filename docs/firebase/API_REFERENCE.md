# Firebase API Reference - Code Examples

**Project:** SparkData Cleaning Checklists
**Created:** 2025-12-09

---

## Quick Reference

| Function | Purpose | Location |
|----------|---------|----------|
| `startSession()` | Create new session | Line 1441 |
| `createFirestoreSession()` | Initialize Firestore doc | Line 1473 |
| `syncToFirestore()` | Upload current state | Line 1530 |
| `startAutoSync()` | Begin 45s interval | Line 1509 |
| `stopAutoSync()` | End sync interval | Line 1522 |
| `updateSyncStatus()` | Update UI indicator | Line 1573 |
| `loadState()` | Restore session on load | Line 1615 |

---

## Core Functions

### 1. startSession()

**Purpose:** Initialize a new cleaning session

**Triggered by:** User clicks "Start Session" button

**What it does:**
1. Generates unique session ID
2. Records start time
3. Saves to localStorage
4. Creates Firestore document
5. Updates UI
6. Starts auto-sync

**Code:**
```javascript
function startSession() {
    if (currentSessionId) {
        alert('Session already started!');
        return;
    }

    // Generate session ID
    currentSessionId = generateSessionId();
    sessionStartTime = new Date();

    // Save to localStorage
    localStorage.setItem('currentSessionId', currentSessionId);
    localStorage.setItem('sessionStartTime', sessionStartTime.toISOString());

    // Create session in Firestore
    createFirestoreSession();

    // Update UI
    document.getElementById('startBtn').disabled = true;
    document.getElementById('startBtn').textContent = 'Session Started';
    document.getElementById('sessionInfo').style.display = 'flex';
    document.getElementById('sessionStartTime').textContent =
        sessionStartTime.toLocaleTimeString('en-US', {
            hour: '2-digit', minute: '2-digit'
        });

    // Start auto-sync
    startAutoSync();

    console.log('Session started:', currentSessionId);
}
```

**Example call:**
```javascript
// Called automatically by onclick handler
<button onclick="startSession()">Start Session</button>
```

---

### 2. createFirestoreSession()

**Purpose:** Create initial Firestore document for session

**Triggered by:** `startSession()`

**What it creates:**
```javascript
{
  sessionId: "session_1733760000_k8j2h4n9p",
  startTime: Timestamp(2025-12-09 13:17:00),
  endTime: null,
  status: "active",
  checklistType: "single-cleaner",
  checklistDate: "2025-11-26",
  progress: 0,
  completedCount: 0,
  totalCount: 120,
  lastSyncTime: Timestamp(2025-12-09 13:17:00),
  notes: "",
  tasks: {
    "mb-shower": { checked: false, lastUpdated: Timestamp },
    "mb-sink": { checked: false, lastUpdated: Timestamp },
    // ... all tasks initialized to false
  }
}
```

**Code:**
```javascript
async function createFirestoreSession() {
    try {
        const checkboxes = document.querySelectorAll('.task-checkbox');
        const tasksMap = {};

        // Initialize all tasks as unchecked
        checkboxes.forEach(cb => {
            tasksMap[cb.id] = {
                checked: cb.checked,
                lastUpdated: firebase.firestore.FieldValue.serverTimestamp()
            };
        });

        // Create document
        await db.collection('cleaning-sessions').doc(currentSessionId).set({
            sessionId: currentSessionId,
            startTime: firebase.firestore.Timestamp.fromDate(sessionStartTime),
            endTime: null,
            status: 'active',
            checklistType: 'single-cleaner',
            checklistDate: '2025-11-26',
            progress: 0,
            completedCount: 0,
            totalCount: checkboxes.length,
            lastSyncTime: firebase.firestore.FieldValue.serverTimestamp(),
            notes: '',
            tasks: tasksMap
        });

        updateSyncStatus('success');
        console.log('Firestore session created:', currentSessionId);
    } catch (error) {
        console.error('Error creating Firestore session:', error);
        updateSyncStatus('error');
    }
}
```

---

### 3. syncToFirestore()

**Purpose:** Upload current checkbox states to Firestore

**Triggered by:** Auto-sync interval (every 45s) or manual call

**What it syncs:**
- All checkbox states (checked: true/false)
- Notes textarea content
- Progress percentage
- Completed count

**Code:**
```javascript
async function syncToFirestore() {
    if (!currentSessionId) {
        return;  // No active session
    }

    updateSyncStatus('syncing');

    try {
        const checkboxes = document.querySelectorAll('.task-checkbox');
        const tasksMap = {};
        let completedCount = 0;

        // Build tasks map from current DOM state
        checkboxes.forEach(cb => {
            const isChecked = cb.checked;
            tasksMap[cb.id] = {
                checked: isChecked,
                lastUpdated: firebase.firestore.FieldValue.serverTimestamp()
            };
            if (isChecked) completedCount++;
        });

        const progress = Math.round((completedCount / checkboxes.length) * 100);
        const notes = document.getElementById('cleanerNotes').value;

        // Update Firestore document
        await db.collection('cleaning-sessions').doc(currentSessionId).update({
            tasks: tasksMap,
            progress: progress,
            completedCount: completedCount,
            lastSyncTime: firebase.firestore.FieldValue.serverTimestamp(),
            notes: notes
        });

        lastSyncTime = Date.now();
        updateSyncStatus('success');
        console.log('Synced to Firestore:', { progress, completedCount });

    } catch (error) {
        console.error('Firestore sync error:', error);
        updateSyncStatus('error');
    }
}
```

**Manual call (browser console):**
```javascript
// Force immediate sync
syncToFirestore();
```

---

### 4. startAutoSync() / stopAutoSync()

**Purpose:** Manage auto-sync interval

**Interval:** 45 seconds

**Code:**
```javascript
const SYNC_INTERVAL = 45000; // 45 seconds

function startAutoSync() {
    if (syncInterval) {
        clearInterval(syncInterval);  // Clear existing interval
    }

    syncInterval = setInterval(() => {
        syncToFirestore();
    }, SYNC_INTERVAL);

    console.log('Auto-sync started (every 45 seconds)');
}

function stopAutoSync() {
    if (syncInterval) {
        clearInterval(syncInterval);
        syncInterval = null;
    }
}
```

**When called:**
- **Start:** After START button clicked
- **Stop:** After SUBMIT button clicked or page unload

---

### 5. updateSyncStatus()

**Purpose:** Update sync status indicator UI

**States:**
- `syncing` → "↻ Syncing..." (blue)
- `success` → "● Synced" (green)
- `error` → "⚠ Offline" (yellow/orange)

**Code:**
```javascript
function updateSyncStatus(status) {
    const statusEl = document.getElementById('syncStatus');
    if (!statusEl) return;

    statusEl.className = 'sync-status ' + status;

    const messages = {
        syncing: '↻ Syncing...',
        success: '● Synced',
        error: '⚠ Offline'
    };

    statusEl.textContent = messages[status] || '● Synced';
}
```

**Example calls:**
```javascript
updateSyncStatus('syncing');  // Before sync
updateSyncStatus('success');  // After successful sync
updateSyncStatus('error');    // On error or offline
```

---

### 6. loadState()

**Purpose:** Restore session and checkbox states on page load

**What it restores:**
- Checkbox states (from localStorage)
- Active session (if exists)
- Session UI (START button state, session info)
- Auto-sync (resumes if session active)

**Code:**
```javascript
function loadState() {
    const checkboxes = document.querySelectorAll('.task-checkbox');

    // Restore checkbox states
    checkboxes.forEach(checkbox => {
        const saved = localStorage.getItem('single-cleaner-' + checkbox.id);
        if (saved === 'true') {
            checkbox.checked = true;
        }
    });
    updateProgress();

    // Restore existing session
    const savedSessionId = localStorage.getItem('currentSessionId');
    const savedStartTime = localStorage.getItem('sessionStartTime');

    if (savedSessionId && savedStartTime) {
        currentSessionId = savedSessionId;
        sessionStartTime = new Date(savedStartTime);

        // Restore UI
        document.getElementById('startBtn').disabled = true;
        document.getElementById('startBtn').textContent = 'Session Started';
        document.getElementById('sessionInfo').style.display = 'flex';
        document.getElementById('sessionStartTime').textContent =
            sessionStartTime.toLocaleTimeString('en-US', {
                hour: '2-digit', minute: '2-digit'
            });

        // Resume auto-sync
        startAutoSync();

        console.log('Session restored:', currentSessionId);
    }
}
```

**When called:**
```javascript
// Automatically on page load (line 1646)
loadState();
```

---

## Firebase SDK Methods

### Reading Data

**Get single document:**
```javascript
const sessionId = 'session_1733760000_k8j2h4n9p';

// One-time read
const doc = await db.collection('cleaning-sessions').doc(sessionId).get();
if (doc.exists) {
    const data = doc.data();
    console.log('Progress:', data.progress);
}
```

**Query multiple documents:**
```javascript
// Get all active sessions
const snapshot = await db.collection('cleaning-sessions')
    .where('status', '==', 'active')
    .get();

snapshot.forEach(doc => {
    console.log(doc.id, doc.data());
});
```

**Real-time listener:**
```javascript
// Listen for changes
const unsubscribe = db.collection('cleaning-sessions')
    .doc(sessionId)
    .onSnapshot(doc => {
        const data = doc.data();
        console.log('Updated:', data.progress);
    });

// Stop listening
unsubscribe();
```

---

### Writing Data

**Create document:**
```javascript
await db.collection('cleaning-sessions').doc(sessionId).set({
    sessionId: sessionId,
    startTime: firebase.firestore.Timestamp.now(),
    status: 'active'
});
```

**Update existing document:**
```javascript
await db.collection('cleaning-sessions').doc(sessionId).update({
    progress: 75,
    completedCount: 90
});
```

**Update nested fields:**
```javascript
await db.collection('cleaning-sessions').doc(sessionId).update({
    'tasks.mb-shower.checked': true,
    'tasks.mb-shower.lastUpdated': firebase.firestore.FieldValue.serverTimestamp()
});
```

**Delete document:**
```javascript
await db.collection('cleaning-sessions').doc(sessionId).delete();
```

---

## Common Queries

### Get Active Sessions

```javascript
const activeSessions = await db.collection('cleaning-sessions')
    .where('status', '==', 'active')
    .orderBy('startTime', 'desc')
    .get();

activeSessions.forEach(doc => {
    const data = doc.data();
    console.log(`Session ${doc.id}: ${data.progress}% complete`);
});
```

### Get Today's Sessions

```javascript
const today = new Date();
today.setHours(0, 0, 0, 0);

const todaysSessions = await db.collection('cleaning-sessions')
    .where('startTime', '>=', firebase.firestore.Timestamp.fromDate(today))
    .orderBy('startTime', 'desc')
    .get();
```

### Get Completed Sessions

```javascript
const completedSessions = await db.collection('cleaning-sessions')
    .where('status', '==', 'completed')
    .orderBy('endTime', 'desc')
    .limit(10)  // Last 10 completed
    .get();
```

### Count Sessions

```javascript
const snapshot = await db.collection('cleaning-sessions')
    .where('status', '==', 'active')
    .get();

console.log('Active sessions:', snapshot.size);
```

---

## Firestore Field Values

### Server Timestamp

**Purpose:** Use server time (not client time)

**Usage:**
```javascript
await db.collection('cleaning-sessions').doc(sessionId).update({
    lastSyncTime: firebase.firestore.FieldValue.serverTimestamp()
});
```

**Why?** Server timestamp is authoritative, handles timezone/clock skew issues.

### Timestamp from Date

**Purpose:** Convert JavaScript Date to Firestore Timestamp

**Usage:**
```javascript
const startTime = new Date();

await db.collection('cleaning-sessions').doc(sessionId).set({
    startTime: firebase.firestore.Timestamp.fromDate(startTime)
});
```

### Array Operations

**Add to array:**
```javascript
await db.collection('cleaning-sessions').doc(sessionId).update({
    tags: firebase.firestore.FieldValue.arrayUnion('priority')
});
```

**Remove from array:**
```javascript
await db.collection('cleaning-sessions').doc(sessionId).update({
    tags: firebase.firestore.FieldValue.arrayRemove('priority')
});
```

---

## Error Handling

### Try-Catch Pattern

```javascript
async function safeFirestoreOperation() {
    try {
        await db.collection('cleaning-sessions').doc(sessionId).update({
            progress: 50
        });
        console.log('Success');
    } catch (error) {
        console.error('Firestore error:', error);

        // Handle specific errors
        if (error.code === 'permission-denied') {
            alert('Permission denied. Check Firestore rules.');
        } else if (error.code === 'not-found') {
            console.log('Session not found');
        } else {
            // Network error, quota exceeded, etc.
            updateSyncStatus('error');
        }
    }
}
```

### Common Error Codes

| Code | Meaning | Solution |
|------|---------|----------|
| `permission-denied` | Firestore rules block write | Check security rules |
| `not-found` | Document doesn't exist | Create document first |
| `unavailable` | Network/server error | Retry, check connection |
| `resource-exhausted` | Quota exceeded | Upgrade plan or reduce usage |

---

## Browser Console Helpers

### Debug Current Session

```javascript
// Check if session is active
console.log('Session ID:', currentSessionId);
console.log('Start time:', sessionStartTime);

// View session data
db.collection('cleaning-sessions').doc(currentSessionId).get()
    .then(doc => console.log(doc.data()));
```

### Force Immediate Sync

```javascript
syncToFirestore();
```

### View All Sessions

```javascript
db.collection('cleaning-sessions').get()
    .then(snapshot => {
        snapshot.forEach(doc => {
            console.log(doc.id, doc.data());
        });
    });
```

### Delete All Sessions (Testing Only)

```javascript
// WARNING: This deletes all session data!
db.collection('cleaning-sessions').get()
    .then(snapshot => {
        snapshot.forEach(doc => {
            doc.ref.delete();
        });
    });
```

---

## Testing Offline Mode

### Simulate Offline

**Method 1 (DevTools):**
1. Open DevTools → Network tab
2. Throttling dropdown → "Offline"

**Method 2 (JavaScript):**
```javascript
// Disable Firestore network
firebase.firestore().disableNetwork()
    .then(() => console.log('Network disabled'));

// Re-enable network
firebase.firestore().enableNetwork()
    .then(() => console.log('Network enabled'));
```

### Test Offline Sync

```javascript
// 1. Disable network
firebase.firestore().disableNetwork();

// 2. Make changes (will queue)
syncToFirestore();  // Queued locally

// 3. Re-enable network
firebase.firestore().enableNetwork();  // Auto-syncs queued writes
```

---

## Dashboard Real-time Listener (Preview)

**Future implementation:**

```javascript
// Dashboard page (dashboard.html)
const sessionId = new URLSearchParams(window.location.search).get('session');

// Set up real-time listener
db.collection('cleaning-sessions').doc(sessionId)
    .onSnapshot(doc => {
        if (!doc.exists) {
            console.log('Session not found');
            return;
        }

        const data = doc.data();

        // Update progress bar
        document.getElementById('progressBar').style.width = data.progress + '%';
        document.getElementById('progressText').textContent =
            `${data.completedCount} of ${data.totalCount} tasks`;

        // Update task list
        Object.keys(data.tasks).forEach(taskId => {
            const taskEl = document.getElementById('task-' + taskId);
            if (taskEl) {
                taskEl.classList.toggle('completed', data.tasks[taskId].checked);
            }
        });

        // Update notes
        document.getElementById('cleanerNotes').textContent = data.notes;

        // Show last updated
        const lastUpdated = new Date().toLocaleTimeString();
        document.getElementById('lastUpdated').textContent = lastUpdated;
    }, error => {
        console.error('Listener error:', error);
    });
```

---

## Related Files

| File | Purpose |
|------|---------|
| `docs/firebase/README.md` | Quick access guide |
| `docs/firebase/ARCHITECTURE.md` | System design |
| `Deep_Clean_*_DEV.html` | Implementation |

---

**Last Updated:** 2025-12-09
**API Version:** 1.0
**Firebase SDK:** 10.7.1 (compat)
