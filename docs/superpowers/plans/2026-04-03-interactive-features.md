# Interactive Features Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add 3 interactive features (stamp collection, love notes, fun fact flip cards) to the existing single-file travel itinerary HTML with Firebase real-time sync.

**Architecture:** Single-file HTML (`01-plan/itinerary.html`). Firebase Realtime DB via CDN for cross-device sync. localStorage as offline fallback. All CSS/JS inline in the same file.

**Tech Stack:** Firebase Realtime Database (Spark free plan), Firebase JS SDK 9 compat via CDN, CSS 3D transforms, vanilla JS.

---

### Task 1: Firebase Project Setup

**Files:**
- Manual: Firebase Console (web)

This is a manual step the user must complete before code can be written.

- [ ] **Step 1: Create Firebase project**

Go to https://console.firebase.google.com/
1. Click "Add project" → name: `beijing-trip-2026`
2. Disable Google Analytics (not needed)
3. Click "Create project"

- [ ] **Step 2: Create Realtime Database**

In Firebase Console:
1. Left sidebar → Build → Realtime Database
2. Click "Create Database"
3. Choose location: `us-central1` (or any)
4. Start in **test mode** (allows read/write without auth for 30 days — sufficient for this trip)

- [ ] **Step 3: Get config values**

In Firebase Console:
1. Project Settings (gear icon) → General → "Your apps" → Web icon (`</>`)
2. Register app, nickname: `trip-web`
3. Copy the `firebaseConfig` object — you'll need `apiKey`, `databaseURL`, `projectId`

Provide these values to the next task. The config looks like:
```js
{
  apiKey: "AIza...",
  authDomain: "beijing-trip-2026.firebaseapp.com",
  databaseURL: "https://beijing-trip-2026-default-rtdb.firebaseio.com",
  projectId: "beijing-trip-2026"
}
```

---

### Task 2: Add Firebase SDK + Bottom Navigation Bar

**Files:**
- Modify: `01-plan/itinerary.html` (add before `</head>`: Firebase CDN scripts; add before `</body>`: nav bar HTML; add to `<style>`: nav bar CSS)

- [ ] **Step 1: Add Firebase CDN scripts**

Add before `</head>`:

```html
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
```

- [ ] **Step 2: Add bottom nav bar CSS**

Add to `<style>`, before the closing `</style>`:

```css
/* Bottom nav */
.bottom-nav {
  position: fixed; bottom: 0; left: 50%; transform: translateX(-50%);
  width: 100%; max-width: 480px;
  display: flex; justify-content: space-around; align-items: center;
  padding: 10px 0 calc(10px + env(safe-area-inset-bottom));
  background: rgba(255, 220, 240, 0.85);
  backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px);
  border-top: 2px solid #ffd6e7;
  border-radius: 18px 18px 0 0;
  z-index: 1000;
  box-shadow: 0 -4px 16px rgba(200, 150, 220, 0.2);
}
.nav-btn {
  background: none; border: none; font-size: 24px;
  display: flex; flex-direction: column; align-items: center; gap: 2px;
  color: #8b4570; cursor: pointer; position: relative;
  -webkit-tap-highlight-color: transparent;
}
.nav-btn span { font-size: 10px; font-weight: 600; }
.nav-btn .badge {
  position: absolute; top: -4px; right: -4px;
  background: #d63031; color: #fff; font-size: 10px; font-weight: 700;
  width: 16px; height: 16px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  line-height: 1;
}

/* Overlay base */
.overlay {
  display: none; position: fixed; top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.4); z-index: 2000;
  justify-content: center; align-items: flex-end;
}
.overlay.show { display: flex; }
.overlay-panel {
  background: linear-gradient(180deg, #fff0f5, #fff);
  width: 100%; max-width: 480px; max-height: 85vh;
  border-radius: 24px 24px 0 0; padding: 20px 16px;
  overflow-y: auto; position: relative;
  animation: slideUp 0.3s ease-out;
}
@keyframes slideUp {
  from { transform: translateY(100%); }
  to { transform: translateY(0); }
}
.overlay-close {
  position: absolute; top: 12px; right: 16px;
  background: none; border: none; font-size: 20px; cursor: pointer; color: #a080b0;
}
.overlay-title {
  font-size: 18px; font-weight: 700; color: #d63384;
  text-align: center; margin-bottom: 16px;
}
```

- [ ] **Step 3: Add body bottom padding and nav bar HTML**

Add to body CSS rule:

```css
padding-bottom: 80px;
```

Add before `</body>` (before any `<script>` tags):

```html
<!-- Bottom Nav -->
<nav class="bottom-nav">
  <button class="nav-btn" onclick="toggleOverlay('stamp-overlay')">
    <span style="font-size:24px">⭐</span>
    <span>集章册</span>
  </button>
  <button class="nav-btn" onclick="toggleOverlay('note-write-overlay')">
    <span style="font-size:24px">✏️</span>
    <span>写纸条</span>
  </button>
  <button class="nav-btn" id="notebox-btn" onclick="toggleOverlay('notebox-overlay')">
    <span style="font-size:24px">📮</span>
    <span>纸条盒</span>
  </button>
</nav>
```

- [ ] **Step 4: Add Firebase init script**

Add before `</body>`, after the nav bar HTML:

```html
<script>
// === Firebase Init ===
const firebaseConfig = {
  // USER: Replace with your Firebase config from Task 1
  apiKey: "YOUR_API_KEY",
  authDomain: "beijing-trip-2026.firebaseapp.com",
  databaseURL: "https://beijing-trip-2026-default-rtdb.firebaseio.com",
  projectId: "beijing-trip-2026"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// === Overlay toggle ===
function toggleOverlay(id) {
  const el = document.getElementById(id);
  el.classList.toggle('show');
}
function closeOverlay(id) {
  document.getElementById(id).classList.remove('show');
}
// Close overlay on backdrop click
document.addEventListener('click', e => {
  if (e.target.classList.contains('overlay')) {
    e.target.classList.remove('show');
  }
});
</script>
```

- [ ] **Step 5: Verify and commit**

Open in browser, confirm: bottom nav bar visible, three buttons render, clicking them does nothing yet (overlays not created yet).

```bash
git add 01-plan/itinerary.html
git commit -m "feat: add Firebase SDK + bottom navigation bar"
```

---

### Task 3: Stamp Collection Feature

**Files:**
- Modify: `01-plan/itinerary.html` (add stamp CSS, stamp overlay HTML, stamp JS)

- [ ] **Step 1: Add stamp CSS**

Add to `<style>`:

```css
/* Stamp collection */
.stamp-grid {
  display: grid; grid-template-columns: repeat(4, 1fr);
  gap: 12px; padding: 8px 0;
}
.stamp-item {
  display: flex; flex-direction: column; align-items: center; gap: 4px;
  cursor: pointer; -webkit-tap-highlight-color: transparent;
}
.stamp-circle {
  width: 60px; height: 60px; border-radius: 50%;
  border: 3px dashed #ddd; background: #fafafa;
  display: flex; align-items: center; justify-content: center;
  font-size: 28px; transition: all 0.3s;
  position: relative;
}
.stamp-circle.done {
  border: 3px solid #ffd700; background: linear-gradient(135deg, #fff0f5, #ffe8cc);
  box-shadow: 0 2px 12px rgba(255, 215, 0, 0.4);
}
.stamp-name {
  font-size: 10px; color: #b0a0b8; font-weight: 600; text-align: center;
}
.stamp-name.done { color: #d63384; }
@keyframes stampIn {
  0% { transform: scale(0) rotate(-180deg); opacity: 0; }
  60% { transform: scale(1.3) rotate(10deg); opacity: 1; }
  100% { transform: scale(1) rotate(0deg); opacity: 1; }
}
.stamp-circle.animating { animation: stampIn 0.8s ease-out; }

/* Stamp confirm dialog */
.stamp-confirm {
  display: none; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);
  background: #fff; border-radius: 20px; padding: 24px 20px; text-align: center;
  box-shadow: 0 8px 32px rgba(0,0,0,0.2); z-index: 3000; min-width: 260px;
}
.stamp-confirm.show { display: block; }
.stamp-confirm h3 { color: #d63384; font-size: 16px; margin-bottom: 12px; }
.stamp-confirm .btn-row { display: flex; gap: 10px; justify-content: center; margin-top: 16px; }
.stamp-confirm .btn-row button {
  padding: 8px 24px; border-radius: 20px; border: none; font-size: 14px;
  font-weight: 600; cursor: pointer;
}
.btn-yes { background: linear-gradient(135deg, #ff7675, #d63384); color: #fff; }
.btn-no { background: #f0e0f0; color: #8b4570; }

/* Celebration */
.celebration {
  display: none; position: fixed; top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(255,200,220,0.95); z-index: 5000;
  flex-direction: column; align-items: center; justify-content: center;
}
.celebration.show { display: flex; }
.celebration .kirby-dance { font-size: 64px; animation: bounce 0.6s ease-in-out infinite; }
.celebration h2 { color: #d63384; font-size: 20px; margin-top: 16px; text-align: center; }
.celebration p { color: #8b4570; font-size: 14px; margin-top: 8px; }
.celebration .close-celebration {
  margin-top: 24px; padding: 10px 32px; border-radius: 20px;
  background: linear-gradient(135deg, #ff7675, #d63384); color: #fff;
  border: none; font-size: 16px; font-weight: 600; cursor: pointer;
}
@keyframes starRain {
  0% { transform: translateY(-20px) rotate(0deg); opacity: 1; }
  100% { transform: translateY(100vh) rotate(720deg); opacity: 0; }
}
.star-particle {
  position: fixed; font-size: 20px; z-index: 5001;
  animation: starRain 2s ease-in forwards; pointer-events: none;
}
```

- [ ] **Step 2: Add stamp overlay HTML and confirm dialog**

Add before `<!-- Bottom Nav -->`:

```html
<!-- Stamp Collection Overlay -->
<div class="overlay" id="stamp-overlay">
  <div class="overlay-panel">
    <button class="overlay-close" onclick="closeOverlay('stamp-overlay')">✕</button>
    <div class="overlay-title">⭐ 暖暖的集章册 ⭐</div>
    <div class="stamp-grid" id="stamp-grid"></div>
    <p style="text-align:center;color:#b898c0;font-size:11px;margin-top:12px">点击盖章 | 长按撤销</p>
  </div>
</div>

<!-- Stamp Confirm Dialog -->
<div class="stamp-confirm" id="stamp-confirm">
  <h3 id="stamp-confirm-text"></h3>
  <div class="btn-row">
    <button class="btn-yes" id="stamp-confirm-yes">到了！盖章！</button>
    <button class="btn-no" id="stamp-confirm-no">还没到~</button>
  </div>
</div>

<!-- Stamp Undo Confirm -->
<div class="stamp-confirm" id="stamp-undo-confirm">
  <h3 id="stamp-undo-text"></h3>
  <div class="btn-row">
    <button class="btn-yes" id="stamp-undo-yes">撤销</button>
    <button class="btn-no" id="stamp-undo-no">不撤了</button>
  </div>
</div>

<!-- Celebration -->
<div class="celebration" id="celebration">
  <div class="kirby-dance">(>'-')> ⭐ <('-'<)</div>
  <h2>暖暖的北京大冒险<br>全部通关！！！</h2>
  <p>集齐 8 个景点印章 🎉</p>
  <button class="close-celebration" onclick="closeCelebration()">太棒了！</button>
</div>
```

- [ ] **Step 3: Add stamp JS logic**

Add inside the existing `<script>` block, after the overlay toggle code:

```js
// === Stamp Collection ===
const STAMPS = [
  { id: 'yiheyuan', name: '颐和园', icon: '⛲' },
  { id: 'mutianyu', name: '慕田峪', icon: '🏯' },
  { id: 'niujie', name: '牛街', icon: '🍢' },
  { id: 'gugong', name: '故宫', icon: '🏛' },
  { id: 'guobo', name: '国博', icon: '🌋' },
  { id: 'tiananmen', name: '天安门', icon: '🇨🇳' },
  { id: 'beida', name: '北大', icon: '🎓' },
  { id: 'yonghegong', name: '雍和宫', icon: '🛕' }
];

let stampData = {};
let pendingStampId = null;
let longPressTimer = null;

// Load from localStorage first
try {
  stampData = JSON.parse(localStorage.getItem('stamps') || '{}');
} catch(e) {}

function saveStampsLocal() {
  localStorage.setItem('stamps', JSON.stringify(stampData));
}

function renderStamps() {
  const grid = document.getElementById('stamp-grid');
  grid.innerHTML = '';
  STAMPS.forEach(s => {
    const done = stampData[s.id] && stampData[s.id].done;
    const item = document.createElement('div');
    item.className = 'stamp-item';
    item.innerHTML = `
      <div class="stamp-circle ${done ? 'done' : ''}" data-id="${s.id}">
        ${done ? s.icon : '<span style="font-size:20px;color:#ddd">?</span>'}
      </div>
      <div class="stamp-name ${done ? 'done' : ''}">${s.name}</div>
    `;
    const circle = item.querySelector('.stamp-circle');

    // Tap to stamp
    circle.addEventListener('click', () => {
      if (!done) {
        pendingStampId = s.id;
        document.getElementById('stamp-confirm-text').textContent =
          `暖暖到${s.name}啦？`;
        document.getElementById('stamp-confirm').classList.add('show');
      }
    });

    // Long press to undo
    circle.addEventListener('touchstart', e => {
      if (done) {
        longPressTimer = setTimeout(() => {
          pendingStampId = s.id;
          document.getElementById('stamp-undo-text').textContent =
            `撤销${s.name}的印章？`;
          document.getElementById('stamp-undo-confirm').classList.add('show');
        }, 1500);
      }
    }, { passive: true });
    circle.addEventListener('touchend', () => clearTimeout(longPressTimer));
    circle.addEventListener('touchmove', () => clearTimeout(longPressTimer));

    grid.appendChild(item);
  });
}

// Confirm stamp
document.getElementById('stamp-confirm-yes').addEventListener('click', () => {
  document.getElementById('stamp-confirm').classList.remove('show');
  if (!pendingStampId) return;
  stampData[pendingStampId] = { done: true, time: new Date().toISOString() };
  saveStampsLocal();
  db.ref('trips/beijing2026/stamps/' + pendingStampId).set(stampData[pendingStampId]);
  renderStamps();
  // Animate the newly stamped circle
  const circle = document.querySelector(`.stamp-circle[data-id="${pendingStampId}"]`);
  if (circle) circle.classList.add('animating');
  // Check all done
  const allDone = STAMPS.every(s => stampData[s.id] && stampData[s.id].done);
  if (allDone) setTimeout(showCelebration, 1000);
  pendingStampId = null;
});

document.getElementById('stamp-confirm-no').addEventListener('click', () => {
  document.getElementById('stamp-confirm').classList.remove('show');
  pendingStampId = null;
});

// Undo stamp
document.getElementById('stamp-undo-yes').addEventListener('click', () => {
  document.getElementById('stamp-undo-confirm').classList.remove('show');
  if (!pendingStampId) return;
  stampData[pendingStampId] = { done: false };
  saveStampsLocal();
  db.ref('trips/beijing2026/stamps/' + pendingStampId).set(stampData[pendingStampId]);
  renderStamps();
  pendingStampId = null;
});

document.getElementById('stamp-undo-no').addEventListener('click', () => {
  document.getElementById('stamp-undo-confirm').classList.remove('show');
  pendingStampId = null;
});

// Celebration
function showCelebration() {
  document.getElementById('celebration').classList.add('show');
  // Star rain
  for (let i = 0; i < 30; i++) {
    const star = document.createElement('div');
    star.className = 'star-particle';
    star.textContent = ['⭐','🌟','✨','💖'][Math.floor(Math.random()*4)];
    star.style.left = Math.random() * 100 + 'vw';
    star.style.animationDelay = Math.random() * 1.5 + 's';
    document.body.appendChild(star);
    setTimeout(() => star.remove(), 3500);
  }
}
function closeCelebration() {
  document.getElementById('celebration').classList.remove('show');
}

// Firebase sync - listen for changes
db.ref('trips/beijing2026/stamps').on('value', snap => {
  const data = snap.val();
  if (data) {
    stampData = data;
    saveStampsLocal();
    renderStamps();
  }
});

renderStamps();
```

- [ ] **Step 4: Verify and commit**

Open in browser:
1. Click "集章册" → overlay appears with 8 grey circles
2. Tap a circle → confirm dialog → tap "到了" → stamp animates in
3. Long press a stamped circle → undo dialog → tap "撤销" → stamp removed
4. Stamp all 8 → celebration screen with star rain

```bash
git add 01-plan/itinerary.html
git commit -m "feat: add stamp collection with Firebase sync + celebration"
```

---

### Task 4: Love Notes Feature

**Files:**
- Modify: `01-plan/itinerary.html` (add notes CSS, overlays HTML, notes JS)

- [ ] **Step 1: Add notes CSS**

Add to `<style>`:

```css
/* Love notes */
.note-write-form { display: flex; flex-direction: column; gap: 12px; }
.note-write-form textarea {
  width: 100%; height: 80px; border: 2px solid #ffd6e7; border-radius: 14px;
  padding: 12px; font-size: 14px; font-family: inherit; resize: none;
  background: #fff; color: #4a3548;
}
.note-write-form textarea:focus { outline: none; border-color: #d63384; }
.note-from-toggle {
  display: flex; gap: 8px; justify-content: center;
}
.note-from-toggle button {
  padding: 6px 20px; border-radius: 16px; border: 2px solid #ffd6e7;
  background: #fff; color: #8b4570; font-size: 13px; font-weight: 600;
  cursor: pointer;
}
.note-from-toggle button.active {
  background: linear-gradient(135deg, #ff7675, #d63384); color: #fff; border-color: transparent;
}
.note-send-btn {
  padding: 10px; border-radius: 20px; border: none;
  background: linear-gradient(135deg, #ff7675, #d63384); color: #fff;
  font-size: 15px; font-weight: 600; cursor: pointer;
}
.note-send-btn:disabled { opacity: 0.5; }
.note-charcount { text-align: right; font-size: 11px; color: #b898c0; }

/* Envelope notification */
.envelope-notify {
  display: none; justify-content: center; margin-bottom: 12px;
  animation: float 2s ease-in-out infinite;
  cursor: pointer;
}
.envelope-notify.show { display: flex; }
.envelope-notify .envelope {
  background: linear-gradient(135deg, #ffc8dd, #ffb8d0);
  padding: 10px 20px; border-radius: 16px;
  box-shadow: 0 4px 16px rgba(255, 150, 180, 0.3);
  font-size: 14px; color: #8b4570; font-weight: 600;
}

/* Note card */
.note-card {
  background: linear-gradient(135deg, #fffde6, #fff5e6);
  border: 1px solid #ffeebb; border-radius: 14px;
  padding: 14px; margin-bottom: 10px; position: relative;
  box-shadow: 0 2px 8px rgba(200,180,100,0.15);
  transform: rotate(-0.5deg);
}
.note-card .note-content { font-size: 14px; color: #5a4830; line-height: 1.6; }
.note-card .note-meta {
  display: flex; justify-content: space-between; margin-top: 8px;
  font-size: 11px; color: #b8a080;
}
.note-day-group { font-size: 13px; color: #d63384; font-weight: 600; margin: 12px 0 6px; }

/* Note open animation */
@keyframes envelopeOpen {
  0% { transform: scale(0.8) rotateX(30deg); opacity: 0; }
  100% { transform: scale(1) rotateX(0deg); opacity: 1; }
}
.note-card.new { animation: envelopeOpen 0.6s ease-out; }
```

- [ ] **Step 2: Add note overlays HTML**

Add before `<!-- Bottom Nav -->`, after the stamp/celebration HTML:

```html
<!-- Envelope Notification (insert after header div) -->

<!-- Write Note Overlay -->
<div class="overlay" id="note-write-overlay">
  <div class="overlay-panel">
    <button class="overlay-close" onclick="closeOverlay('note-write-overlay')">✕</button>
    <div class="overlay-title">✏️ 给暖暖写纸条</div>
    <div class="note-write-form">
      <textarea id="note-input" maxlength="50" placeholder="写点什么给暖暖..."></textarea>
      <div class="note-charcount"><span id="note-charcount">0</span>/50</div>
      <div class="note-from-toggle">
        <button class="active" data-from="爸爸" onclick="selectFrom(this)">👨 爸爸</button>
        <button data-from="妈妈" onclick="selectFrom(this)">👩 妈妈</button>
      </div>
      <button class="note-send-btn" id="note-send-btn" onclick="sendNote()" disabled>💌 送出纸条</button>
    </div>
  </div>
</div>

<!-- Note Box Overlay -->
<div class="overlay" id="notebox-overlay">
  <div class="overlay-panel">
    <button class="overlay-close" onclick="closeOverlay('notebox-overlay')">✕</button>
    <div class="overlay-title">📮 暖暖的纸条盒</div>
    <div id="notebox-list"><p style="text-align:center;color:#b898c0;font-size:13px">还没有纸条哦~</p></div>
  </div>
</div>
```

Also add the envelope notification HTML right after the `<div class="header">...</div>` closing tag:

```html
<!-- Envelope Notification -->
<div class="envelope-notify" id="envelope-notify" onclick="openLatestNote()">
  <div class="envelope">💌 有新纸条！点我打开~</div>
</div>
```

- [ ] **Step 3: Add notes JS logic**

Add inside the `<script>` block, after the stamps code:

```js
// === Love Notes ===
let noteFrom = '爸爸';
let notesData = {};

// Load from localStorage
try {
  notesData = JSON.parse(localStorage.getItem('notes') || '{}');
} catch(e) {}

function saveNotesLocal() {
  localStorage.setItem('notes', JSON.stringify(notesData));
}

// Char count
document.getElementById('note-input').addEventListener('input', e => {
  const len = e.target.value.length;
  document.getElementById('note-charcount').textContent = len;
  document.getElementById('note-send-btn').disabled = len === 0;
});

function selectFrom(btn) {
  document.querySelectorAll('.note-from-toggle button').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  noteFrom = btn.dataset.from;
}

function sendNote() {
  const input = document.getElementById('note-input');
  const content = input.value.trim();
  if (!content) return;

  const noteId = 'note_' + Date.now();
  const note = {
    content: content,
    from: noteFrom,
    time: new Date().toISOString(),
    read: false
  };

  notesData[noteId] = note;
  saveNotesLocal();
  db.ref('trips/beijing2026/notes/' + noteId).set(note);

  input.value = '';
  document.getElementById('note-charcount').textContent = '0';
  document.getElementById('note-send-btn').disabled = true;
  closeOverlay('note-write-overlay');
}

function checkUnreadNotes() {
  const unread = Object.values(notesData).filter(n => !n.read);
  const envelope = document.getElementById('envelope-notify');
  const badge = document.querySelector('#notebox-btn .badge');

  if (unread.length > 0) {
    envelope.classList.add('show');
  } else {
    envelope.classList.remove('show');
  }

  // Update badge
  if (badge) badge.remove();
  if (unread.length > 0) {
    const b = document.createElement('span');
    b.className = 'badge';
    b.textContent = unread.length;
    document.getElementById('notebox-btn').appendChild(b);
  }
}

function openLatestNote() {
  const unread = Object.entries(notesData)
    .filter(([k, v]) => !v.read)
    .sort((a, b) => new Date(b[1].time) - new Date(a[1].time));

  if (unread.length === 0) return;

  const [noteId, note] = unread[0];
  note.read = true;
  notesData[noteId] = note;
  saveNotesLocal();
  db.ref('trips/beijing2026/notes/' + noteId + '/read').set(true);

  // Show in notebox
  toggleOverlay('notebox-overlay');
  renderNotebox();
  checkUnreadNotes();
}

function renderNotebox() {
  const list = document.getElementById('notebox-list');
  const entries = Object.entries(notesData)
    .filter(([k, v]) => v.content)
    .sort((a, b) => new Date(b[1].time) - new Date(a[1].time));

  if (entries.length === 0) {
    list.innerHTML = '<p style="text-align:center;color:#b898c0;font-size:13px">还没有纸条哦~</p>';
    return;
  }

  // Group by date
  const groups = {};
  entries.forEach(([id, note]) => {
    const day = note.time.split('T')[0];
    if (!groups[day]) groups[day] = [];
    groups[day].push({ id, ...note });
  });

  list.innerHTML = '';
  Object.entries(groups).sort((a, b) => b[0].localeCompare(a[0])).forEach(([day, notes]) => {
    const dayEl = document.createElement('div');
    dayEl.className = 'note-day-group';
    dayEl.textContent = '📅 ' + day;
    list.appendChild(dayEl);

    notes.forEach(note => {
      const card = document.createElement('div');
      card.className = 'note-card' + (!note.read ? ' new' : '');
      card.innerHTML = `
        <div class="note-content">${escapeHtml(note.content)}</div>
        <div class="note-meta">
          <span>${note.from === '爸爸' ? '👨' : '👩'} ${note.from}</span>
          <span>${new Date(note.time).toLocaleTimeString('zh-CN', {hour:'2-digit',minute:'2-digit'})}</span>
        </div>
      `;
      // Mark as read on view
      if (!note.read) {
        note.read = true;
        notesData[note.id] = note;
        saveNotesLocal();
        db.ref('trips/beijing2026/notes/' + note.id + '/read').set(true);
      }
      list.appendChild(card);
    });
  });
  checkUnreadNotes();
}

function escapeHtml(text) {
  const d = document.createElement('div');
  d.textContent = text;
  return d.innerHTML;
}

// Firebase sync - notes
db.ref('trips/beijing2026/notes').on('value', snap => {
  const data = snap.val();
  if (data) {
    notesData = data;
    saveNotesLocal();
    checkUnreadNotes();
    // If notebox is open, re-render
    if (document.getElementById('notebox-overlay').classList.contains('show')) {
      renderNotebox();
    }
  }
});

checkUnreadNotes();
```

- [ ] **Step 4: Verify and commit**

Open in two browser tabs (simulating two devices):
1. Tab 1: Click "写纸条" → type text → select "爸爸" → send
2. Tab 2: Envelope appears → click → notebox opens with the note
3. Check notebox shows grouped notes with correct styling

```bash
git add 01-plan/itinerary.html
git commit -m "feat: add love notes with Firebase sync + envelope notification"
```

---

### Task 5: Fun Fact Flip Cards

**Files:**
- Modify: `01-plan/itinerary.html` (add flip CSS, modify timeline rows to add ? bubbles, add flip JS)

- [ ] **Step 1: Add flip card CSS**

Add to `<style>`:

```css
/* Fun fact flip */
.flip-trigger {
  display: inline-flex; align-items: center; justify-content: center;
  width: 22px; height: 22px; border-radius: 50%;
  background: linear-gradient(135deg, #ffc8dd, #ffb8d0);
  color: #d63384; font-size: 12px; font-weight: 700;
  cursor: pointer; vertical-align: middle; margin-left: 4px;
  box-shadow: 0 1px 4px rgba(255,150,180,0.3);
  -webkit-tap-highlight-color: transparent;
}
.flip-container {
  perspective: 800px; display: none; margin: 8px 0;
}
.flip-container.show { display: block; }
.flip-inner {
  position: relative; transition: transform 0.5s;
  transform-style: preserve-3d;
}
.flip-inner.flipped { transform: rotateY(180deg); }
.flip-back {
  background: linear-gradient(135deg, #fff9e6, #fff0f5);
  border: 2px solid #ffd700; border-radius: 14px;
  padding: 14px; font-size: 13px; color: #5a4830; line-height: 1.6;
  text-align: center;
  backface-visibility: hidden;
  transform: rotateY(180deg);
  cursor: pointer;
}
.flip-back::before { content: '💡 '; }
.flip-back .close-flip {
  display: block; margin-top: 8px; font-size: 11px; color: #b898c0;
}
```

- [ ] **Step 2: Add fun facts data and rendering JS**

Add inside the `<script>` block, after the notes code:

```js
// === Fun Fact Flip Cards ===
const FUN_FACTS = {
  '颐和园': '昆明湖的水面有 220 万平方米，相当于 128 个暖暖的学校（红树林外国语小学）那么大！',
  '慕田峪': '长城的砖一块就有 20 斤重，比暖暖的书包还沉！古人要一块一块搬上山～',
  '牛街': '牛街不是因为有牛，是因为回族人把街叫"牛肉街"，后来简称牛街',
  '故宫': '故宫有 9999 间半房间！传说天上有 10000 间，皇帝不敢超过',
  '庞贝': '庞贝古城被火山灰埋了 1700 年才被发现，整座城市像被按了暂停键 ⏸️',
  '天安门': '天安门广场大到可以站 100 万人！是全中国最大的广场',
  '北大': '北大的未名湖是因为大家想不出名字，所以就叫"未名"湖 😂',
  '雍和宫': '雍和宫里有一尊 18 米高的大佛，用一整棵白檀树雕成的，比 6 层楼还高！'
};

// Find timeline cells that match fun fact keywords and add ? bubble
document.querySelectorAll('.timeline td').forEach(td => {
  const text = td.textContent;
  Object.keys(FUN_FACTS).forEach(keyword => {
    if (text.includes(keyword) && !td.classList.contains('time') && !td.classList.contains('note')) {
      // Add ? trigger
      const trigger = document.createElement('span');
      trigger.className = 'flip-trigger';
      trigger.textContent = '?';
      trigger.addEventListener('click', e => {
        e.stopPropagation();
        // Find or create flip container
        const row = td.closest('tr');
        let flipRow = row.nextElementSibling;
        if (!flipRow || !flipRow.classList.contains('flip-row')) {
          flipRow = document.createElement('tr');
          flipRow.className = 'flip-row';
          const flipTd = document.createElement('td');
          flipTd.colSpan = 3;
          flipTd.innerHTML = `
            <div class="flip-container show">
              <div class="flip-back" style="transform:none" onclick="this.closest('.flip-container').classList.remove('show')">
                ${FUN_FACTS[keyword]}
                <span class="close-flip">点击收起</span>
              </div>
            </div>
          `;
          flipRow.appendChild(flipTd);
          row.after(flipRow);
        } else {
          const container = flipRow.querySelector('.flip-container');
          container.classList.toggle('show');
        }
      });
      td.appendChild(trigger);
    }
  });
});
```

- [ ] **Step 3: Verify and commit**

Open in browser:
1. Scroll to any day section → see "?" pink bubbles next to attraction names
2. Tap "?" → fun fact card slides open below
3. Tap card → it closes
4. Check all 8 attractions have their fun facts

```bash
git add 01-plan/itinerary.html
git commit -m "feat: add fun fact flip cards for 8 attractions"
```

---

### Task 6: Final Integration, Copy, and Deploy

**Files:**
- Modify: `01-plan/itinerary.html` (replace Firebase placeholder config with real values)
- Copy: `01-plan/itinerary.html` → `index.html`

- [ ] **Step 1: Replace Firebase config**

Replace `YOUR_API_KEY` and other placeholder values in the `firebaseConfig` object with the real values from Task 1 Step 3.

- [ ] **Step 2: Full end-to-end test**

Open on two different devices/browsers:
1. Device A: Stamp a few attractions → Device B sees stamps update
2. Device A: Write a note → Device B sees envelope notification
3. Both: Tap "?" on attractions → fun facts display correctly
4. Device A: Stamp all 8 → celebration animation fires
5. Device A: Long press a stamp → undo dialog → undo works
6. Open notebox → notes grouped by day, sorted correctly

- [ ] **Step 3: Copy and push**

```bash
cp 01-plan/itinerary.html index.html
git add 01-plan/itinerary.html index.html
git commit -m "feat: complete interactive features - stamps, notes, fun facts

- Stamp collection: 8 attractions, Firebase sync, long-press undo, celebration
- Love notes: write/read with envelope animation, notebox history
- Fun fact flip cards: 8 personalized facts for Nuannuan
- Bottom navigation bar with glassmorphism effect"
git push
```

- [ ] **Step 4: Verify on GitHub Pages**

Open https://bugujun44.github.io/beijing-trip-2026/ on phone and confirm all features work.
