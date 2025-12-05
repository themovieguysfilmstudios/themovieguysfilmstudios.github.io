<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>THE MOVIE GUYS FILM STUDIOS LLC</title>
<!-- Load Tailwind CSS -->
<script src="https://cdn.tailwindcss.com"></script>
<!-- Firebase Imports (Required for Persistence) -->
<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
    import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
    import { getFirestore, doc, onSnapshot, setDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
    import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
    
    // Global variables for Firebase access
    window.initializeApp = initializeApp;
    window.getAuth = getAuth;
    window.signInAnonymously = signInAnonymously;
    window.signInWithCustomToken = signInWithCustomToken;
    window.onAuthStateChanged = onAuthStateChanged;
    window.getFirestore = getFirestore;
    window.doc = doc;
    window.onSnapshot = onSnapshot;
    window.setDoc = setDoc;
    window.setLogLevel = setLogLevel;
</script>

<!-- Use Inter font for a modern look -->
<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
body {
    font-family: 'Inter', sans-serif;
    min-height: 100vh;
    background-color: #0d0d0d;
}
/* Custom floating animation for the media windows */
.window {
    animation: float 4s infinite ease-in-out;
    transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
    /* Subtle filter indicates content is locked/unavailable when not admin */
    filter: brightness(0.7); 
}
.window:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 20px rgba(229, 9, 20, 0.3); /* Red shadow on hover */
}
.admin-active {
    filter: brightness(1.0); /* Full brightness when admin is active */
    cursor: pointer;
    box-shadow: 0 5px 15px rgba(255, 255, 255, 0.1); /* Subtle white shadow for active state */
}
.admin-active:hover {
    box-shadow: 0 10px 20px rgba(229, 9, 20, 0.5); /* Strong red shadow on hover */
}
@keyframes float {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-5px); }
}
/* Ensure media fills the container */
.window img, .window video {
    width: 100%;
    height: 100%;
    object-fit: cover;
}
/* Style for loading indicator */
#loading-indicator {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.8);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 100;
}
.spinner {
    border: 8px solid #333;
    border-top: 8px solid #e50914;
    border-radius: 50%;
    width: 60px;
    height: 60px;
    animation: spin 1s linear infinite;
}
@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}
</style>
<script>
tailwind.config = {
theme: {
extend: {
colors: {
'netflix-red': '#e50914',
}
}
}
}
</script>
</head>
<body class="bg-gray-900 text-white">

<!-- Loading Indicator -->
<div id="loading-indicator">
    <div class="spinner"></div>
</div>

<!-- Header Section -->
<header class="flex items-center justify-between p-6 bg-gray-950 border-b-4 border-netflix-red shadow-2xl">
    <div class="flex items-center space-x-4">
        <!-- Film Reel SVG/Logo Placeholder -->
        <svg class="w-10 h-10 text-netflix-red" fill="currentColor" viewBox="0 0 24 24">
            <path d="M18 3v2h-2V3H8v2H6V3H4v18h2v-2h2v2h8v-2h2v2h2V3h-2zM8 17H6v-2h2v2zm0-4H6v-2h2v2zm0-4H6V7h2v2zm10 8h-2v-2h2v2zm0-4h-2v-2h2v2zm0-4h-2V7h2v2z"/>
        </svg>
        <h1 class="text-3xl font-extrabold text-white tracking-widest uppercase">THE MOVIE GUYS FILM STUDIOS LLC</h1>
    </div>
    <!-- Admin Buttons: Sign In and Logout -->
    <div id="auth-controls" class="flex items-center">
        <!-- Login Button: calls openLogin to show the modal -->
        <button id="loginButton" onclick="openLogin()" class="px-6 py-2 bg-netflix-red hover:bg-red-700 text-white font-semibold rounded-lg shadow-lg transition duration-300 transform hover:scale-105">
            Admin Sign In
        </button>
        
        <!-- Logout Button: hidden by default -->
        <button id="logoutButton" onclick="logout()" class="hidden px-6 py-2 bg-gray-500 hover:bg-gray-600 text-white font-semibold rounded-lg shadow-lg transition duration-300 transform hover:scale-105">
            Admin Sign Out
        </button>
    </div>
</header>

<!-- Media Windows Section (Drag & Drop) -->
<section class="windows grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-8 p-8 md:p-12 max-w-7xl mx-auto">
<!-- Window 1 -->
<div class="window relative w-full h-64 bg-gray-900 border-2 border-dashed border-gray-700 rounded-xl flex justify-center items-center text-gray-500 overflow-hidden shadow-xl hover:border-netflix-red" data-index="1" ondrop="drop(event,1)" ondragover="allowDrop(event)">
<div class="drop-area text-center p-4 transition duration-300">Drag & Drop Media Here (1)</div>
</div>
<!-- Window 2 -->
<div class="window relative w-full h-64 bg-gray-900 border-2 border-dashed border-gray-700 rounded-xl flex justify-center items-center text-gray-500 overflow-hidden shadow-xl hover:border-netflix-red" data-index="2" ondrop="drop(event,2)" ondragover="allowDrop(event)">
<div class="drop-area text-center p-4 transition duration-300">Drag & Drop Media Here (2)</div>
</div>
<!-- Window 3 -->
<div class="window relative w-full h-64 bg-gray-900 border-2 border-dashed border-gray-700 rounded-xl flex justify-center items-center text-gray-500 overflow-hidden shadow-xl hover:border-netflix-red" data-index="3" ondrop="drop(event,3)" ondragover="allowDrop(event)">
<div class="drop-area text-center p-4 transition duration-300">Drag & Drop Media Here (3)</div>
</div>
<!-- Window 4 -->
<div class="window relative w-full h-64 bg-gray-900 border-2 border-dashed border-gray-700 rounded-xl flex justify-center items-center text-gray-500 overflow-hidden shadow-xl hover:border-netflix-red" data-index="4" ondrop="drop(event,4)" ondragover="allowDrop(event)">
<div class="drop-area text-center p-4 transition duration-300">Drag & Drop Media Here (4)</div>
</div>
<!-- Window 5: First Long Window (spans two columns) -->
<div class="window relative w-full h-64 bg-gray-900 border-2 border-dashed border-gray-700 rounded-xl flex justify-center items-center text-gray-500 overflow-hidden shadow-xl hover:border-netflix-red lg:col-span-2 lg:h-96" data-index="5" ondrop="drop(event,5)" ondragover="allowDrop(event)">
<div class="drop-area text-center p-4 transition duration-300">Long Media Drop Zone (5)</div>
</div>
<!-- Window 6: Second Long Window (new, spans two columns) -->
<div class="window relative w-full h-64 bg-gray-900 border-2 border-dashed border-gray-700 rounded-xl flex justify-center items-center text-gray-500 overflow-hidden shadow-xl hover:border-netflix-red lg:col-span-2 lg:h-96" data-index="6" ondrop="drop(event,6)" ondragover="allowDrop(event)">
<div class="drop-area text-center p-4 transition duration-300">Long Media Drop Zone (6)</div>
</div>
</section>

<!-- Company Statement and Contact Info Section (Unchanged) -->
<footer class="max-w-7xl mx-auto p-8 md:p-12 mt-10 bg-gray-950 rounded-xl shadow-2xl border-t-4 border-gray-700">
<!-- Company Goals/Statement -->
<div class="mb-10">
<h2 class="text-4xl font-extrabold text-netflix-red mb-4">Our Vision. Our Story.</h2>
<p class="text-lg text-gray-300 leading-relaxed border-l-4 border-netflix-red pl-4 py-2 bg-gray-900 rounded-r-lg">
At **THE MOVIE GUYS FILM STUDIOS LLC**, our goal is simple: to redefine cinematic storytelling. We are a passionate, driven film company committed to producing visually stunning, emotionally resonant, and culturally impactful content. We strive to be the definitive hub for independent, high-quality filmmaking, pushing the boundaries of creativity and technical excellence to captivate global audiences.
</p>
</div>

<!-- Contact Information -->
<div>
<h2 class="text-3xl font-bold text-white mb-6">Contact Us</h2>
<div class="grid grid-cols-1 md:grid-cols-3 gap-6 text-gray-400">
<div class="flex items-center space-x-3 p-4 bg-gray-800 rounded-lg shadow-md">
<!-- Email Icon -->
<svg class="w-6 h-6 text-netflix-red" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"></path></svg>
<div>
<span class="font-semibold text-sm block">Email</span>
<a href="mailto:themovieguysfilmstudiosllc@gmail.com" class="text-white hover:text-netflix-red transition">themovieguysfilmstudiosllc@gmail.com</a>
</div>
</div>
<div class="flex items-center space-x-3 p-4 bg-gray-800 rounded-lg shadow-md">
<!-- Phone Icon -->
<svg class="w-6 h-6 text-netflix-red" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-3.96a20.08 20.08 0 01-11.88-11.88V5z"></path></svg>
<div>
<span class="font-semibold text-sm block">Phone</span>
<a href="tel:3092615796" class="text-white hover:text-netflix-red transition">309-261-5796</a>
</div>
</div>

<div class="flex items-center space-x-3 p-4 bg-gray-800 rounded-lg shadow-md">
<!-- Address Icon -->
<svg class="w-6 h-6 text-netflix-red" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
<div>
<span class="font-semibold text-sm block">Address</span>
<p class="text-white">1713 Indiana St, Bloomington, IL</p>
</div>
</div>
</div>
</div>
</footer>

<!-- Admin Login Modal --> 
<div id="loginModal" class="fixed inset-0 bg-black bg-opacity-90 hidden items-center justify-center z-50"> 
<div class="bg-gray-800 p-10 rounded-xl shadow-2xl w-full max-w-sm text-center transform transition-all duration-300 scale-100 border-t-4 border-netflix-red"> 
<h2 class="text-4xl font-black text-white mb-6">Admin Login</h2> 
<input type="text" id="user" placeholder="Username" class="w-full mb-4 p-4 bg-gray-700 text-white border border-gray-600 rounded-lg focus:ring-netflix-red focus:border-netflix-red transition duration-150 placeholder-gray-400"> 
<input type="password" id="pass" placeholder="Password" class="w-full mb-6 p-4 bg-gray-700 text-white border border-gray-600 rounded-lg focus:ring-netflix-red focus:border-netflix-red transition duration-150 placeholder-gray-400"> 
<button onclick="login()" class="w-full py-3 bg-netflix-red hover:bg-red-700 text-white font-bold text-lg rounded-lg shadow-xl transform hover:scale-[1.02] transition duration-200"> 
Sign In 
</button> 
<button onclick="document.getElementById('loginModal').style.display = 'none';" class="mt-4 text-gray-400 hover:text-white transition duration-200"> 
Cancel 
</button> 
</div> 
</div> 

<!-- Custom Message Modal (Replaces alert()) -->
<div id="messageModal" class="fixed inset-0 bg-black bg-opacity-90 hidden items-center justify-center z-[60]">
<div class="bg-gray-800 p-6 rounded-xl shadow-2xl w-full max-w-xs text-center border-t-4 border-netflix-red">
<p id="messageText" class="text-white text-lg font-semibold mb-6"></p>
<button onclick="closeMessageModal()" class="py-2 px-6 bg-netflix-red hover:bg-red-700 text-white font-semibold rounded-lg transition duration-200">
OK
</button>
</div>
</div>

<!-- Full Screen Video Modal (New Element) -->
<div id="videoModal" class="fixed inset-0 bg-black bg-opacity-95 hidden items-center justify-center z-[70] p-4">
<div class="relative w-full h-full max-w-7xl max-h-5xl">
<!-- Close Button (top right corner) -->
<button onclick="closeFullscreenVideo()" class="absolute top-4 right-4 text-white text-4xl font-bold w-10 h-10 flex items-center justify-center rounded-full bg-netflix-red hover:bg-red-700 transition duration-300 z-[80] shadow-xl">
&times;
</button>
<!-- Video Container -->
<div id="videoContainer" class="w-full h-full flex items-center justify-center">
<!-- Video element will be inserted here -->
</div>
</div>
</div>

<script>
    // --- Firebase Global Variables and Initialization ---
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
    const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
    const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

    let app, db, auth, userId = 'visitor'; // Default ID
    let isAdminLoggedIn = false;
    let currentMediaConfig = {}; // Local cache for media configuration

    // DOM elements
    const loadingIndicator = document.getElementById('loading-indicator');
    const loginBtn = document.getElementById('loginButton');
    const logoutBtn = document.getElementById('logoutButton');
    const windows = document.querySelectorAll('.window');

    // Reference to the single document storing the media configuration (public data)
    let mediaDocRef = null;

    /**
     * Initializes Firebase and authenticates the user (anonymously for visitors, custom token for authenticated canvas user).
     */
    async function initFirebase() {
        try {
            setLogLevel('debug'); // Enable detailed logging
            app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);
            
            // Define the public Firestore document path
            mediaDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'media_windows', 'window_config');

            return new Promise((resolve) => {
                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        console.log('Firebase Auth State: Logged in (UID:', userId, ')');
                    } else if (initialAuthToken) {
                        // Use provided custom token for canvas user
                        await signInWithCustomToken(auth, initialAuthToken);
                        userId = auth.currentUser.uid;
                    } else {
                        // Sign in anonymously for regular visitors
                        const anonUser = await signInAnonymously(auth);
                        userId = anonUser.user.uid;
                        console.log('Firebase Auth State: Signed in anonymously (UID:', userId, ')');
                    }
                    resolve();
                });
            });
        } catch (error) {
            console.error("Firebase initialization failed:", error);
            // Hide loading even on error
            loadingIndicator.style.display = 'none';
        }
    }

    // --- Persistence Functions ---

    /**
     * Saves the current media configuration object to Firestore.
     */
    async function saveMediaConfig() {
        // We allow saving even if not admin, as long as it's triggered by a drop action 
        // which is already gated by isAdminLoggedIn. The Firestore rules should handle 
        // actual permission checks if this were a production environment.
        try {
            await setDoc(mediaDocRef, currentMediaConfig, { merge: true });
            console.log("Media configuration saved successfully.");
        } catch (error) {
            console.error("Error saving media configuration:", error);
            showAlert("Error saving media. Check console for details.");
        }
    }

    /**
     * Loads and listens to the media configuration from Firestore in real-time.
     */
    function loadMediaConfig() {
        if (!mediaDocRef) return;

        onSnapshot(mediaDocRef, (docSnap) => {
            if (docSnap.exists()) {
                currentMediaConfig = docSnap.data();
                console.log("Media configuration updated from Firestore:", currentMediaConfig);
            } else {
                currentMediaConfig = {};
                console.log("No media configuration found. Initializing empty config.");
            }
            // Render the UI based on the latest data
            renderMediaWindows();
            // Hide loading indicator once initial data is loaded
            loadingIndicator.style.display = 'none';
        }, (error) => {
            console.error("Error listening to Firestore updates:", error);
            showAlert("Could not load public media content.");
            loadingIndicator.style.display = 'none';
        });
    }

    /**
     * Renders the media elements based on the local currentMediaConfig cache.
     */
    function renderMediaWindows() {
        windows.forEach(windowBox => {
            const index = windowBox.dataset.index;
            const data = currentMediaConfig[index];
            windowBox.innerHTML = ''; // Clear existing content

            if (data && data.url && data.type) {
                // Render media
                if (data.type.startsWith("image/")) {
                    const img = document.createElement("img");
                    img.src = data.url;
                    img.alt = `Media ${index}`;
                    img.classList.add('w-full', 'h-full', 'object-cover');
                    windowBox.appendChild(img);
                } else if (data.type.startsWith("video/")) {
                    const vid = document.createElement("video");
                    vid.src = data.url;
                    vid.controls = true;
                    vid.autoplay = false;
                    vid.loop = true;
                    vid.classList.add('w-full', 'h-full', 'object-cover');
                    // Add click handler for full-screen view
                    vid.onclick = (e) => {
                        e.stopPropagation(); // Prevent container click events
                        openFullscreenVideo(data.url);
                    };
                    windowBox.appendChild(vid);
                }
                
                // Add the "X" delete button if admin is logged in
                if (isAdminLoggedIn) {
                    addDeleteButton(windowBox, index);
                }
                
            } else {
                // Render empty drop zone
                const dropArea = document.createElement("div");
                dropArea.classList.add('drop-area', 'text-center', 'p-4', 'transition', 'duration-300', 'absolute', 'inset-0', 'flex', 'items-center', 'justify-center');
                dropArea.textContent = `Drag & Drop Media Here (${index})`;
                windowBox.appendChild(dropArea);
            }

            // Update admin-active class state based on login status
            if (isAdminLoggedIn) {
                windowBox.classList.add('admin-active');
            } else {
                windowBox.classList.remove('admin-active');
            }
        });
    }

    /**
     * Adds an "X" button to a window for the admin to remove content.
     */
    function addDeleteButton(windowBox, index) {
        const deleteBtn = document.createElement('button');
        // Using an inline SVG for the 'X' icon for crisp, clean display
        deleteBtn.innerHTML = `
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
            </svg>
        `;
        deleteBtn.classList.add(
            'absolute', 'top-2', 'right-2', 'z-20', 'p-2', 
            'bg-netflix-red', 'text-white', 'rounded-full', 'shadow-lg', 
            'opacity-90', 'hover:opacity-100', 'hover:scale-110', 
            'transition', 'duration-200'
        );
        deleteBtn.onclick = (e) => {
            e.stopPropagation(); // Stop event from propagating up
            clearWindow(index);
        };
        windowBox.appendChild(deleteBtn);
    }

    /**
     * Clears the media from a specific window both locally and in Firestore.
     */
    window.clearWindow = function(index) {
        if (!isAdminLoggedIn) return;
        
        // Remove the entry from the local config
        delete currentMediaConfig[index];
        
        // Save the updated configuration to Firestore
        saveMediaConfig();
        
        showAlert(`Window ${index} content removed.`);
    }

    // --- UI/Modal Functions ---

    /**
     * Helper function to manage UI state (login/logout buttons and window appearance)
     */
    function updateUI(loggedIn) {
        isAdminLoggedIn = loggedIn;
        
        if (loggedIn) {
            loginBtn.classList.add('hidden');
            logoutBtn.classList.remove('hidden');
        } else {
            loginBtn.classList.remove('hidden');
            logoutBtn.classList.add('hidden');
        }
        
        // Rerender to show/hide admin controls (like the Delete X button) and effects
        renderMediaWindows();
    }

    /**
     * Custom Alert Implementation
     */
    window.showAlert = function(message) {
        document.getElementById('messageText').textContent = message;
        document.getElementById('messageModal').style.display = 'flex';
    }

    window.closeMessageModal = function() {
        document.getElementById('messageModal').style.display = 'none';
    }

    /**
     * Opens the login modal
     */
    window.openLogin = function() {
        document.getElementById("loginModal").style.display = "flex";
        document.getElementById("user").value = "";
        document.getElementById("pass").value = "";
    }

    /**
     * Hardcoded Admin Login (user=admin, pass=password)
     */
    window.login = function() {
        const user = document.getElementById("user").value;
        const pass = document.getElementById("pass").value;

        if (user === "admin" && pass === "password") {
            showAlert("Logged in successfully! Welcome, Admin.");
            document.getElementById("loginModal").style.display = "none";
            updateUI(true); // Set logged in state
        } else {
            showAlert("Incorrect credentials. Please try again.");
        }
    }
    
    /**
     * Logout function
     */
    window.logout = function() {
        showAlert("You have been successfully signed out.");
        updateUI(false); // Set logged out state
    }

    /**
     * Opens video in fullscreen modal
     */
    window.openFullscreenVideo = function(videoSrc) {
        const modal = document.getElementById('videoModal');
        const videoContainer = document.getElementById('videoContainer');
        videoContainer.innerHTML = '';

        const fullVideo = document.createElement('video');
        fullVideo.src = videoSrc;
        fullVideo.controls = true;
        fullVideo.autoplay = true; 
        fullVideo.loop = true;
        fullVideo.classList.add('w-full', 'h-full', 'object-contain', 'rounded-xl', 'shadow-2xl');

        videoContainer.appendChild(fullVideo);
        modal.style.display = 'flex';
    }

    /**
     * Closes video fullscreen modal
     */
    window.closeFullscreenVideo = function() {
        const modal = document.getElementById('videoModal');
        const videoContainer = document.getElementById('videoContainer');
        const currentVideo = videoContainer.querySelector('video');
        if (currentVideo) {
            currentVideo.pause();
            currentVideo.currentTime = 0;
        }
        videoContainer.innerHTML = '';
        modal.style.display = 'none';
    }

    // --- Drag and Drop Logic (Gated by isAdminLoggedIn) ---

    // Prevent default behavior to allow dropping ONLY if admin is logged in
    window.allowDrop = function(ev) {
        if (isAdminLoggedIn) {
            ev.preventDefault();
        }
    }

    // Handle the drop event
    window.drop = function(ev, index) {
        ev.preventDefault();
        if (!isAdminLoggedIn) {
            showAlert("Access Denied: Only logged-in administrators can drop media files.");
            return;
        }

        if (!ev.dataTransfer.files || ev.dataTransfer.files.length === 0) {
            return;
        }

        const file = ev.dataTransfer.files[0];
        const fileType = file.type;

        if (!fileType.startsWith("image/") && !fileType.startsWith("video/")) {
            showAlert("Unsupported file type (Only images and videos are allowed).");
            return;
        }
        
        // Create a temporary URL for the media
        const mediaURL = URL.createObjectURL(file);
        
        // Update local configuration
        currentMediaConfig[index] = {
            url: mediaURL,
            type: fileType
        };
        
        // Save to database
        saveMediaConfig();
        
        // NOTE: We rely on the Firestore snapshot listener (loadMediaConfig) to call renderMediaWindows()
        // which prevents race conditions with other users/sessions.
    }

    // --- Initialization ---
    // Start the process: Initialize Firebase, then load data, then set initial UI
    window.addEventListener('load', async () => {
        // Initialize Firebase first
        await initFirebase();
        
        // Listen for data changes
        loadMediaConfig(); 
        
        // Set initial UI state (logged out/visitor)
        updateUI(false); 
        
        // Optional: Close modal on outside click (Login Modal only)
        document.getElementById('loginModal').addEventListener('click', function(e) {
            if (e.target.id === 'loginModal') {
                document.getElementById('loginModal').style.display = 'none';
            }
        });
    });

</script>
</body>
</html>
