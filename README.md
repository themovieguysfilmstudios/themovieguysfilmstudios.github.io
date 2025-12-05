<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>THE MOVIE GUYS FILM STUDIOS LLC</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    :root{--netflix-red:#e50914}
    .bg-netflix-red{background:var(--netflix-red)} .text-netflix-red{color:var(--netflix-red)}
    body{background:#121212;color:#E0E0E0;font-family:Inter,system-ui,Arial}
    .window{animation:float 4s infinite ease-in-out}
    .window.admin-active:hover{border-color:var(--netflix-red);background:#1e1e1e}
    @keyframes float{0%{transform:translateY(0)}50%{transform:translateY(-5px)}100%{transform:translateY(0)}}
  </style>
</head>
<body class="min-h-screen p-4 md:p-8">

  <header class="flex flex-col md:flex-row justify-between items-center mb-8 pb-4 border-b border-gray-700">
    <div class="flex items-center">
      <span class="text-4xl mr-4" role="img" aria-label="film reel">🎬</span>
      <h1 class="text-3xl font-bold text-netflix-red tracking-wider">THE MOVIE GUYS FILM STUDIOS LLC</h1>
    </div>
    <div class="flex space-x-4 mt-4 md:mt-0">
      <button id="loginButton" onclick="openLogin()" class="py-2 px-6 bg-gray-700 hover:bg-gray-600 text-white rounded-lg">Admin Sign In</button>
      <button id="logoutButton" onclick="logout()" class="py-2 px-6 bg-netflix-red text-white rounded-lg hidden">Logout</button>
    </div>
  </header>

  <div id="statusMessage" class="hidden mb-6 p-3 bg-green-900 border border-green-700 text-green-300 rounded-lg shadow-inner">
    <p class="text-sm font-medium">✅ Admin Mode Active: Media will be uploaded to Firebase Storage (if configured).</p>
  </div>

  <main class="container mx-auto">
    <div id="media-windows" class="grid grid-cols-2 lg:grid-cols-4 gap-6"></div>
  </main>

  <!-- Login Modal -->
  <div id="loginModal" class="fixed inset-0 bg-black bg-opacity-75 hidden items-center justify-center z-50">
    <div class="bg-gray-800 p-8 rounded-xl shadow-2xl w-full max-w-sm border border-netflix-red">
      <h2 class="text-2xl font-bold mb-6 text-white text-center">Admin Access</h2>
      <input type="text" id="user" placeholder="Username" class="w-full p-3 mb-4 bg-gray-900 border border-gray-600 rounded-lg text-white placeholder-gray-400">
      <input type="password" id="pass" placeholder="Password" class="w-full p-3 mb-6 bg-gray-900 border border-gray-600 rounded-lg text-white placeholder-gray-400">
      <button id="loginAction" class="w-full py-3 bg-netflix-red text-white font-bold rounded-lg">Sign In</button>
    </div>
  </div>

  <!-- Message Modal (reusable) -->
  <div id="messageModal" class="fixed inset-0 bg-black bg-opacity-75 hidden items-center justify-center z-50">
    <div class="bg-gray-800 p-8 rounded-xl shadow-2xl w-full max-w-md border border-gray-600 text-center">
      <div id="messageText" class="text-white text-lg font-medium mb-6"></div>
      <div id="messageButtons" class="flex justify-center gap-4">
        <button id="messageOk" onclick="closeMessageModal()" class="py-2 px-6 bg-netflix-red text-white rounded-lg">OK</button>
      </div>
    </div>
  </div>

  <!-- Fullscreen video modal -->
  <div id="videoModal" class="fixed inset-0 bg-black bg-opacity-95 hidden items-center justify-center z-50 p-4" onclick="closeFullscreenVideo()">
    <div id="videoContainer" class="w-full h-full max-w-4xl max-h-4xl relative" onclick="event.stopPropagation()"></div>
    <button class="absolute top-4 right-4 text-white text-4xl hover:text-netflix-red" onclick="closeFullscreenVideo()">&times;</button>
  </div>

  <!-- Firebase SDKs + app script -->
  <script type="module">
  // ---- IMPORTS (single app import + named imports) ----
  import { initializeApp, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
  import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
  import { getFirestore, doc, setDoc, onSnapshot, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
  import { getStorage, ref as storageRef, uploadBytes, getDownloadURL, deleteObject } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-storage.js";

  // ---- CONFIG INJECTION (server may define these globals) ----
  const injectedAppId = (typeof __app_id !== 'undefined') ? String(__app_id) : 'default-app-id';
  const injectedFirebaseConfig = (typeof __firebase_config !== 'undefined') ? __firebase_config : null;
  const injectedAuthToken = (typeof __initial_auth_token !== 'undefined') ? String(__initial_auth_token) : null;

  // ---- FEATURE FLAGS / AVAILABILITY ----
  const firebaseAvailable = Boolean(injectedFirebaseConfig && typeof injectedFirebaseConfig === 'object' && Object.keys(injectedFirebaseConfig).length > 0);

  // Globals
  let app = null;
  let auth = null;
  let db = null;
  let storage = null;
  let userId = null;
  window.isAdminLoggedIn = false;
  const localFileMap = new Map(); // temporary previews

  // Firestore location
  // VALID pattern: collection/doc/collection/doc
  const APP_COLLECTION = 'artifacts';
  const APP_DOC_ID = injectedAppId;
  const MEDIA_COLLECTION = 'media';
  const CONFIG_DOC_ID = 'media_layout';

  const CONFIG_DOC_REF = () => {
    if (!db) return null;
    return doc(db, APP_COLLECTION, APP_DOC_ID, MEDIA_COLLECTION, CONFIG_DOC_ID);
  };

  const STORAGE_BASE = `artifacts/${APP_DOC_ID}/public/media`;

  // PLACEHOLDERS
  const PLACEHOLDERS = {
    image: (i) => `https://placehold.co/800x450/1e1e1e/e50914?text=Window+${i}+Image`,
    video: (i) => `https://placehold.co/800x450/1e1e1e/e50914?text=Window+${i}+Video`
  };

  // init firebase if config provided
  function initFirebase() {
    if (!firebaseAvailable) {
      console.warn("Firebase config not provided — running in offline placeholder mode.");
      return;
    }
    try {
      app = initializeApp(injectedFirebaseConfig);
      setLogLevel('warn');
      auth = getAuth(app);
      db = getFirestore(app);
      storage = getStorage(app);
    } catch (e) {
      console.error("Error initializing Firebase:", e);
    }
  }

  // AUTH: prefer injected custom token, else anonymous
  async function initAuth() {
    if (!firebaseAvailable || !auth) return;
    try {
      if (injectedAuthToken) {
        await signInWithCustomToken(auth, injectedAuthToken);
      } else {
        await signInAnonymously(auth);
      }
    } catch (e) {
      console.error("Auth init error:", e);
      // fall back to anonymous attempt
      try { await signInAnonymously(auth); } catch(e2){ console.error(e2); }
    }
  }

  // Listen for auth state and start listening to Firestore when ready
  function setupAuthListener() {
    if (!firebaseAvailable || !auth) return;
    onAuthStateChanged(auth, (user) => {
      if (user) {
        userId = user.uid;
        console.log("Auth ready, uid:", userId);
        // Delay slightly to ensure auth tokens propagate if needed
        setTimeout(initMediaListener, 100);
      } else {
        userId = null;
        console.log("No firebase user");
      }
    });
  }

  // --- UI helpers ---
  function showMessage(text, buttonsHtml = null) {
    const modal = document.getElementById('messageModal');
    const messageText = document.getElementById('messageText');
    const messageButtons = document.getElementById('messageButtons');
    messageText.innerHTML = text;
    if (buttonsHtml) {
      messageButtons.innerHTML = buttonsHtml;
    } else {
      messageButtons.innerHTML = `<button id="messageOk" onclick="closeMessageModal()" class="py-2 px-6 bg-netflix-red text-white rounded-lg">OK</button>`;
    }
    modal.style.display = 'flex';
  }
  window.showAlert = (txt) => showMessage(txt);
  function closeMessageModal() { document.getElementById('messageModal').style.display = 'none'; }
  window.closeMessageModal = closeMessageModal;

  // --- render windows ---
  const defaultWindows = [1,2,3,4].map(i => ({ index: i, type: 'empty', url: '', storagePath: '', classes: '' }));

  function renderWindows(config = defaultWindows) {
    const container = document.getElementById('media-windows');
    container.innerHTML = '';
    config.forEach(item => {
      const div = document.createElement('div');
      const extra = (item.classes || '').split(' ').filter(Boolean);
      div.classList.add('window','relative','w-full','h-64','bg-gray-900','border-2','border-dashed','border-gray-700','rounded-xl','flex','justify-center','items-center','text-gray-500','overflow-hidden','shadow-xl','transition','duration-300', ...extra);
      div.dataset.index = String(item.index);
      // event handlers via addEventListener for module scope safety
      div.addEventListener('dragover', (e) => allowDrop(e));
      div.addEventListener('drop', (e) => drop(e, item.index));

      if (window.isAdminLoggedIn) div.classList.add('admin-active');

      if (!item.url || item.type === 'empty') {
        div.innerHTML = `<div class="drop-area text-center p-4 text-lg">Drag & Drop Media Here (${item.index})</div>`;
      } else if (item.type === 'image') {
        const img = document.createElement('img');
        img.src = localFileMap.get(item.index) || item.url || PLACEHOLDERS.image(item.index);
        img.alt = `Media ${item.index}`;
        img.className = 'w-full h-full object-cover';
        div.appendChild(img);
      } else if (item.type === 'video') {
        const vid = document.createElement('video');
        vid.src = localFileMap.get(item.index) || item.url || PLACEHOLDERS.video(item.index);
        vid.loop = true;
        vid.autoplay = true;
        vid.muted = true; // small previews muted
        vid.className = 'w-full h-full object-cover cursor-pointer';
        vid.addEventListener('click', (e) => { e.stopPropagation(); stopAllSmallVideoAudio(); vid.muted = false; vid.pause(); openFullscreenVideo(vid.src); });
        div.appendChild(vid);
        vid.play().catch(()=>{/* autoplay blocked maybe */});
      }

      // trash button (admin-only)
      const trash = document.createElement('button');
      trash.className = 'trash-button absolute top-2 right-2 p-2 bg-netflix-red text-white rounded-full shadow-lg z-10';
      trash.style.display = window.isAdminLoggedIn ? 'block' : 'none';
      trash.innerHTML = `<svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M6 19c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7H6v12zm2.46-7.12l1.41-1.41L12 12.59l2.12-2.12 1.41 1.41L13.41 14l2.12 2.12-1.41 1.41L12 15.41l-2.12 2.12-1.41-1.41L10.59 14l-2.13-2.12zM15.5 4l-1-1h-5l-1 1H5v2h14V4z"/></svg>`;
      trash.addEventListener('click', (e) => { e.stopPropagation(); if (window.isAdminLoggedIn) confirmRemoval(item.index); else showAlert('Access Denied'); });
      div.appendChild(trash);

      container.appendChild(div);
    });
  }

  function stopAllSmallVideoAudio() {
    document.querySelectorAll('.window video').forEach(v => v.muted = true);
  }

  // Fullscreen
  function openFullscreenVideo(src) {
    const modal = document.getElementById('videoModal');
    const container = document.getElementById('videoContainer');
    container.innerHTML = '';
    const v = document.createElement('video');
    v.src = src;
    v.controls = true;
    v.autoplay = true;
    v.loop = true;
    v.className = 'w-full h-full object-contain rounded-xl shadow-2xl';
    container.appendChild(v);
    modal.style.display = 'flex';
    v.play().catch(()=>{});
  }
  function closeFullscreenVideo() {
    const modal = document.getElementById('videoModal');
    const container = document.getElementById('videoContainer');
    const child = container.querySelector('video');
    if (child) { child.pause(); child.src = ''; }
    container.innerHTML = '';
    modal.style.display = 'none';
  }
  window.openFullscreenVideo = openFullscreenVideo;
  window.closeFullscreenVideo = closeFullscreenVideo;
  window.stopAllSmallVideoAudio = stopAllSmallVideoAudio;

  // Firestore listener
  function initMediaListener() {
    if (!firebaseAvailable || !db) {
      // render placeholders
      renderWindows(defaultWindows);
      return;
    }
    const docRef = CONFIG_DOC_REF();
    if (!docRef) { renderWindows(defaultWindows); return; }

    onSnapshot(docRef, (snap) => {
      if (!snap.exists()) {
        // create default layout only when admin logged in
        if (window.isAdminLoggedIn) {
          const def = defaultWindows.map(d => ({ ...d }));
          setDoc(docRef, { mediaConfig: def }).catch(e => console.error("setDoc failed:", e));
          renderWindows(def);
        } else {
          renderWindows(defaultWindows);
        }
        return;
      }
      const data = snap.data()?.mediaConfig || defaultWindows;
      renderWindows(data);
    }, (err) => {
      console.error("onSnapshot error:", err);
      renderWindows(defaultWindows);
    });
  }

  // Update Firestore media config
  async function updateWindowData(index, type, url, storagePath = '') {
    if (!firebaseAvailable || !db) {
      // If no firebase, keep local only
      // Update local fallback config used to re-render
      const cfg = defaultWindows.map(d => ({ ...d }));
      const idx = cfg.findIndex(i=>i.index===index);
      if (idx>-1) cfg[idx] = {...cfg[idx], type, url, storagePath};
      renderWindows(cfg);
      return;
    }
    const docRef = CONFIG_DOC_REF();
    if (!docRef) { console.error("Invalid docRef"); return; }

    try {
      const snap = await getDoc(docRef);
      let mediaConfig = [];
      if (snap.exists()) mediaConfig = snap.data().mediaConfig || defaultWindows;
      else mediaConfig = defaultWindows.map(d=>({...d}));

      const pos = mediaConfig.findIndex(i => i.index === index);
      if (pos > -1) {
        mediaConfig[pos] = {...mediaConfig[pos], type, url, storagePath};
      } else {
        mediaConfig.push({ index, type, url, storagePath });
      }
      await setDoc(docRef, { mediaConfig }, { merge: true });
    } catch (e) {
      console.error("updateWindowData error:", e);
      showAlert("Error saving media configuration. Check permissions.");
    }
  }

  // --- Drag/drop + upload flow ---
  function allowDrop(ev) {
    if (window.isAdminLoggedIn) ev.preventDefault();
  }

  async function drop(ev, index) {
    ev.preventDefault();
    if (!window.isAdminLoggedIn) { showAlert("Access Denied: Admin only"); return; }
    if (!ev.dataTransfer?.files?.length) return;
    const file = ev.dataTransfer.files[0];
    let fileType = 'empty';
    if (file.type.startsWith('image/')) fileType = 'image';
    else if (file.type.startsWith('video/')) fileType = 'video';
    else { showAlert('Unsupported file type (images/videos only)'); return; }

    // Local preview
    const tempUrl = URL.createObjectURL(file);
    localFileMap.set(index, tempUrl);
    await updateWindowData(index, fileType, tempUrl, ''); // temporary write (optional)

    if (!firebaseAvailable || !storage) {
      showAlert('Firebase not configured — upload disabled. (Preview shown locally)');
      return;
    }

    // Upload to firebase storage
    showMessage(`Uploading ${file.name}... <div class="text-sm mt-2">Please wait</div>`, `<button onclick="closeMessageModal()" class="py-2 px-6 bg-gray-600 text-white rounded-lg">Close</button>`);
    try {
      // create storage ref
      const unique = `${Date.now()}-${file.name.replace(/[^a-zA-Z0-9.\\-_]/g, '_')}`;
      const path = `${STORAGE_BASE}/${unique}`;
      const ref = storageRef(storage, path);
      const snapshot = await uploadBytes(ref, file);
      const permanentUrl = await getDownloadURL(snapshot.ref);

      // persist to firestore
      await updateWindowData(index, fileType, permanentUrl, path);
      localFileMap.delete(index);
      showAlert(`Upload complete: ${file.name}`);
    } catch (err) {
      console.error("Upload error:", err);
      await updateWindowData(index, 'empty', '', '');
      localFileMap.delete(index);
      showAlert(`Upload failed: ${err?.message || err}`);
    }
  }

  // --- Remove media ---
  function confirmRemoval(index) {
    showMessage("Are you sure you want to remove this media? This action is permanent.", `
      <button id="confirmRemoveBtn" class="py-2 px-6 bg-red-700 text-white rounded-lg">Yes, Remove</button>
      <button id="cancelRemoveBtn" class="py-2 px-6 bg-gray-600 text-white rounded-lg">Cancel</button>
    `);
    document.getElementById('confirmRemoveBtn').onclick = async () => {
      closeMessageModal();
      await removeMediaAction(index);
    };
    document.getElementById('cancelRemoveBtn').onclick = () => closeMessageModal();
  }

  async function removeMediaAction(index) {
    if (!window.isAdminLoggedIn) { showAlert("Access Denied"); return; }
    // fetch current config
    if (!firebaseAvailable || !db) {
      showAlert("No server configured — nothing to remove remotely. Local preview cleared.");
      localFileMap.delete(index);
      await updateWindowData(index, 'empty','', '');
      return;
    }
    const docRef = CONFIG_DOC_REF();
    try {
      const snap = await getDoc(docRef);
      if (!snap.exists()) {
        showAlert("Nothing to remove.");
        return;
      }
      const mediaConfig = snap.data().mediaConfig || [];
      const item = mediaConfig.find(i => i.index === index);
      if (item?.storagePath && storage) {
        try {
          const fRef = storageRef(storage, item.storagePath);
          await deleteObject(fRef);
        } catch (e) {
          console.warn("Delete object error (may be missing):", e);
        }
      }
      // mark empty in firestore
      await updateWindowData(index, 'empty','', '');
      localFileMap.delete(index);
      showAlert("Media removed.");
    } catch (e) {
      console.error("removeMediaAction error:", e);
      showAlert("Error removing media. Check console.");
    }
  }

  // --- Admin login/logout (local demo) ---
  function openLogin() { document.getElementById('loginModal').style.display = 'flex'; }
  async function loginAction() {
    const user = document.getElementById('user').value;
    const pass = document.getElementById('pass').value;
    // Demo hard-coded credentials
    if (user === 'admin' && pass === 'password') {
      closeMessageModal();
      showAlert("Logged in as Admin");
      document.getElementById('loginModal').style.display = 'none';
      updateUI(true);
      // ensure firestore listener runs (if firebase available)
      if (firebaseAvailable && db) initMediaListener();
    } else {
      showAlert("Incorrect credentials.");
    }
  }
  function logout() {
    showAlert("Signed out.");
    updateUI(false);
  }
  document.getElementById('loginAction').addEventListener('click', loginAction);

  // Update UI elements
  function updateUI(loggedIn) {
    window.isAdminLoggedIn = !!loggedIn;
    document.getElementById('loginButton').classList.toggle('hidden', loggedIn);
    document.getElementById('logoutButton').classList.toggle('hidden', !loggedIn);
    document.getElementById('statusMessage').classList.toggle('hidden', !loggedIn);
    // show/hide trash icons if windows exist
    document.querySelectorAll('.trash-button').forEach(btn => btn.style.display = loggedIn ? 'block' : 'none');
  }

  // --- Initialization sequence ---
  (async function boot() {
    initFirebase();
    if (firebaseAvailable) {
      await initAuth();
      setupAuthListener();
    } else {
      // render placeholders if no firebase
      renderWindows(defaultWindows);
    }
    updateUI(false);
  })();

  // expose utilities for debugging
  window.renderWindows = renderWindows;
  window.updateWindowData = updateWindowData;
  window.confirmRemoval = confirmRemoval;
  </script>
</body>
</html>
How to test quickly
Drop this index.html into a static host (or open locally).
To test Firebase functionality, set __firebase_config and (optionally) __initial_auth_token in the page before the script runs (server inject or inline <script>window.__firebase_config = {...};</script>). Example:
<script>
  window.__app_id = 'my-app-id';
  window.__firebase_config = {
    apiKey: "...",
    authDomain: "...",
    projectId: "...",
    storageBucket: "...",
    // ...
  };
</script>
