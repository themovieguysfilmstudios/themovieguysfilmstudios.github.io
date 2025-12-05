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
    // NOTE: Firebase Storage imports removed as media functionality was disabled.
    
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
/* Custom floating animation for the media windows (kept, but media windows are removed) */
.window {
    animation: float 4s infinite ease-in-out;
    transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
    filter: brightness(0.7); 
}
.window:hover {
    transform: translateY(0); 
    box-shadow: none;
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
    flex-direction: column;
    color: white;
}
.spinner {
    border: 8px solid #333;
    border-top: 8px solid #e50914;
    border-radius: 50%;
    width: 60px;
    height: 60px;
    animation: spin 1s linear infinite;
    margin-bottom: 20px;
}
@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}
.progress-bar {
    width: 200px;
    height: 8px;
    background-color: #333;
    border-radius: 4px;
    overflow: hidden;
}
.progress-fill {
    height: 100%;
    width: 0%;
    background-color: #e50914;
    transition: width 0.1s linear;
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

<!-- Hidden File Input (REMAINS HIDDEN AND UNUSED) -->
<input type="file" id="fileInput" accept="image/*,video/*" class="hidden">

<!-- Loading Indicator -->
<div id="loading-indicator" style="display: none;">
    <div class="spinner"></div>
    <p id="loading-text" class="text-xl font-semibold mb-4">Loading application data...</p>
    <div class="progress-bar">
        <div id="upload-progress" class="progress-fill" style="width: 0%;"></div>
    </div>
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
</header>

<!-- Company Statement and Contact Info Section -->
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
<svg class="w-6 h-6 text-netflix-red" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 0 016 0z"></path></svg>
<div>
<span class="font-semibold text-sm block">Address</span>
<p class="text-white">1713 Indiana St, Bloomington, IL</p>
</div>
</div>
</div>
</div>
</footer>

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
    const rawAppId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
    const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
    const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

    let app, db, auth, userId = 'visitor'; // Default ID
    let currentMediaConfig = {}; // Local cache for media configuration

    // DOM elements
    const loadingIndicator = document.getElementById('loading-indicator');
    const loadingText = document.getElementById('loading-text');
    const uploadProgress = document.getElementById('upload-progress');
    const fileInput = document.getElementById('fileInput'); 
    
    // Reference to the single document storing the media configuration (public data)
    let mediaDocRef = null;
    let appIdForPersistence = '';


    /**
     * Initializes Firebase services (app, db, auth).
     * @returns {void}
     */
    async function initFirebaseServices() {
        try {
            setLogLevel('debug'); // Enable detailed logging
            
            // Clean the raw __app_id for path construction
            const shortIdMatch = rawAppId.match(/c_[a-f0-9]{16}/);
            appIdForPersistence = shortIdMatch ? shortIdMatch[0] : rawAppId; 

            console.log(`App ID used for Persistence (Cleaned): ${appIdForPersistence}`);

            app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);
            
            // Define the public Firestore document path using the cleaned appIdForPersistence
            // Path: /artifacts/{appIdForPersistence}/public/data/media_windows/window_config
            mediaDocRef = doc(db, 'artifacts', appIdForPersistence, 'public', 'data', 'media_windows', 'window_config');
            console.log("Firebase services initialized.");
        } catch (error) {
            console.error("Firebase initialization failed:", error);
            loadingIndicator.style.display = 'none';
        }
    }

    /**
     * Loads and listens to the media configuration from Firestore in real-time.
     * NOTE: This function remains for Firebase initialization/auth stability, 
     * but the media rendering logic it calls is no longer visible in the UI.
     */
    function loadMediaConfig() {
        if (!mediaDocRef) {
            console.error("Firestore document reference not set.");
            return;
        }

        onSnapshot(mediaDocRef, (docSnap) => {
            if (docSnap.exists()) {
                currentMediaConfig = docSnap.data();
                console.log("Media configuration updated from Firestore (No windows to render):", currentMediaConfig);
            } else {
                currentMediaConfig = {};
                console.log("No media configuration found. Initializing empty config.");
            }
            // Hide loading indicator once initial data is loaded
            loadingIndicator.style.display = 'none';
        }, (error) => {
            console.error("Error listening to Firestore updates:", error);
            showAlert("Application data services could not be loaded fully.");
            loadingIndicator.style.display = 'none';
        });
    }

    /**
     * Renders the media elements based on the local currentMediaConfig cache for the public view.
     * NOTE: This is now an empty function as the windows were removed.
     */
    function renderMediaWindows() {
        // No media windows to render.
    }

    /**
     * Handles clicks on the media windows (Visitor Mode Only).
     * NOTE: This is now an empty function as the windows were removed.
     */
    function handleWindowClick(e, index, mediaData) {
        // No windows to click.
    }


    // --- UI/Modal Functions ---

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

    // --- Drag and Drop Logic (DISABLED) ---

    // Deny all drop actions
    window.allowDrop = function(ev) {
        ev.preventDefault();
    }

    // Deny all drop actions
    window.drop = async function(ev, index) {
        ev.preventDefault();
        showAlert("Modification Access Denied. This website is now in public display mode.");
    }

    // --- Initialization ---
    window.addEventListener('load', async () => {

        // Show initial loading screen
        loadingText.textContent = "Initializing security and content.";
        loadingIndicator.style.display = 'flex';
        
        await initFirebaseServices(); // Initialize services (app, db, auth)
        
        // Ensure authentication completes before loading data 
        onAuthStateChanged(auth, async (user) => {
            if (user) {
                // Existing user or custom token sign-in completed
                userId = user.uid;
            } else if (initialAuthToken) {
                // Sign in with the provided custom token (Canvas user)
                try {
                    await signInWithCustomToken(auth, initialAuthToken);
                    userId = auth.currentUser.uid;
                } catch(e) {
                     await signInAnonymously(auth);
                     userId = auth.currentUser.uid;
                }
            } else {
                // Sign in anonymously (Regular visitor)
                const anonUser = await signInAnonymously(auth);
                userId = anonUser.user.uid;
            }
            
            // NOW that we are guaranteed to be authenticated, start the Firestore listener.
            loadMediaConfig(); 
        });
    });

</script>
</body>
</html>
