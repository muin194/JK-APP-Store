<!DOCTYPE html>
<html lang="en" data-theme="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App Store | User Panel (Progress Tracking)</title>
    
    <script src='//libtl.com/sdk.js' data-zone='9682755' data-sdk='show_9682755'></script>

    <script src="https://www.gstatic.com/firebasejs/8.6.8/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.6.8/firebase-database.js"></script>

    <style>
        /* ------------------------
           CSS: Global & Theming
        ------------------------ */
        :root {
            --bg-color: #f0f2f5; --card-bg: #ffffff; --text-color: #1c1e21;
            --header-bg: #4267b2; --header-text: #ffffff; --button-primary: #1877f2;
            --button-text: #ffffff; --border-color: #dddfe2; --shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
            --ad-status-color: #dc3545; /* Red for emphasis */
            --progress-bar-bg: #e9ecef;
            --progress-bar-fill: #28a745; /* Green */
        }

        [data-theme="dark"] {
            --bg-color: #1c1e21; --card-bg: #242526; --text-color: #e4e6eb;
            --header-bg: #3a3b3c; --header-text: #e4e6eb; --button-primary: #5879ba;
            --button-text: #ffffff; --border-color: #444950; --shadow: 0 1px 3px rgba(0, 0, 0, 0.5);
            --ad-status-color: #e66b78;
            --progress-bar-bg: #444950;
            --progress-bar-fill: #498859;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; }

        body {
            font-family: Arial, sans-serif; background-color: var(--bg-color);
            color: var(--text-color); transition: background-color 0.3s, color 0.3s;
            min-height: 100vh;
        }

        header {
            background-color: var(--header-bg); color: var(--header-text);
            padding: 15px 20px; display: flex; justify-content: space-between;
            align-items: center; box-shadow: var(--shadow);
        }

        .container {
            max-width: 1200px; margin: 20px auto; padding: 0 20px;
        }

        .search-bar {
            background-color: var(--card-bg); padding: 15px; border-radius: 8px;
            box-shadow: var(--shadow); margin-bottom: 20px;
        }

        input[type="text"] {
            width: 100%; padding: 10px; border: 1px solid var(--border-color);
            border-radius: 4px; background-color: var(--bg-color);
            color: var(--text-color); font-size: 16px;
        }

        button {
            background-color: var(--button-primary); color: var(--button-text);
            border: none; padding: 10px 15px; border-radius: 4px;
            cursor: pointer; transition: background-color 0.2s;
        }

        button:hover:not(:disabled) {
            background-color: #1565c0;
        }
        
        button:disabled {
            background-color: #6c757d;
            cursor: not-allowed;
        }

        #app-list {
            display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 20px;
        }

        .app-card {
            background-color: var(--card-bg); border: 1px solid var(--border-color);
            border-radius: 8px; overflow: hidden; box-shadow: var(--shadow);
        }

        .app-details {
            padding: 15px;
        }

        .app-title {
            font-size: 1.4em; margin-bottom: 10px;
        }
        
        /* Progress Bar Styles */
        .progress-container {
            margin: 10px 0;
            background-color: var(--progress-bar-bg);
            border-radius: 5px;
            overflow: hidden;
            height: 10px;
        }
        
        .progress-bar {
            height: 100%;
            background-color: var(--progress-bar-fill);
            width: 0%; 
            transition: width 0.3s ease-in-out;
        }
        .ad-status {
            font-size: 0.9em; margin-top: 5px; margin-bottom: 10px; min-height: 1.2em;
        }
        
        .click-prompt {
             color: var(--ad-status-color);
             font-weight: bold;
             margin-top: 8px;
        }
        
        .download-ready {
            color: var(--progress-bar-fill) !important;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <header>
        <h1>App Store</h1>
        <div>
            <button id="theme-toggle">🌛 Dark Mode</button>
            <a href="admin_panel.html" target="_blank">
                <button>Admin Login</button>
            </a>
        </div>
    </header>

    <div class="container">
        <div class="search-bar">
            <input type="text" id="search-input" placeholder="Search by App Title..." onkeyup="filterApps()">
        </div>

        <div id="app-list">
            </div>
    </div>

    <script>
        // --- 1. FIREBASE INITIALIZATION ---
        const firebaseConfig = {
            apiKey: "AIzaSyDl127P9rC5x3UGhJxTacWLEv-kRuKBH4w",
            authDomain: "app-store-68289.firebaseapp.com",
            databaseURL: "https://app-store-68289-default-rtdb.firebaseio.com",
            projectId: "app-store-68289",
            storageBucket: "app-store-68289.firebasestorage.app",
            messagingSenderId: "262762683625",
            appId: "1:262762683625:web:51d26fa62b395e45252478"
        };
        if (!firebase.apps.length) {
            firebase.initializeApp(firebaseConfig);
        }
        const database = firebase.database();
        
        let ALL_APPS_DATA = [];
        let CURRENT_SETTINGS = { views: 2, clicks: 1 };
        
        // Local Storage Keys
        const AD_PROGRESS_KEY = 'adProgress_'; 
        const MONETAG_SDK_FUNC_NAME = 'show_9682755';

        // --- 2. SDK Availability Check & Helpers ---

        function isMonetagSdkReady(appId) {
            const adStatusElement = document.getElementById(ad-status-${appId});
            if (typeof window[MONETAG_SDK_FUNC_NAME] !== 'function') {
                if (adStatusElement) {
                    adStatusElement.innerText = 'Monetag SDK not loaded. Cannot proceed.';
                    adStatusElement.style.color = 'var(--ad-status-color)'; 
                }
                return false;
            }
            return true;
        }

        function getProgress(appId) {
            const stored = localStorage.getItem(AD_PROGRESS_KEY + appId);
            return JSON.parse(stored) || { viewsCompleted: 0, clicksCompleted: 0 };
        }

        function setProgress(appId, views, clicks) {
            localStorage.setItem(AD_PROGRESS_KEY + appId, JSON.stringify({
                viewsCompleted: views,
                clicksCompleted: clicks
            }));
        }

        // --- 3. UI Update Logic ---

        function updateProgressUI(appId, settings, progress) {
            const adStatusElement = document.getElementById(ad-status-${appId});
            const progressBarElement = document.getElementById(progress-bar-${appId});
            const downloadButton = document.getElementById(download-btn-${appId});
            
            const requiredViews = settings.views;
            const requiredClicks = settings.clicks;
            
            const viewsProgress = progress.viewsCompleted;
            const clicksProgress = progress.clicksCompleted;

            const isReady = (viewsProgress >= requiredViews) && (clicksProgress >= requiredClicks);
            
            // Calculate progress based on views only for the visual bar
            const viewPercentage = Math.min(100, (viewsProgress / requiredViews) * 100);
            
            progressBarElement.style.width = ${viewPercentage}%;
            if (isReady) {
                adStatusElement.className = 'ad-status download-ready';
                adStatusElement.innerHTML = '✅ Download Ready! Click to get your app.';
                downloadButton.disabled = false;
                downloadButton.innerText = 'Download App';

            } else {
                adStatusElement.className = 'ad-status';
                downloadButton.disabled = false;
                downloadButton.innerText = 'Continue Ad Process';

                if (viewsProgress < requiredViews) {
                    adStatusElement.innerHTML = **Ad View Progress:** ${viewsProgress}/${requiredViews} views completed.;
                } else if (clicksProgress < requiredClicks) {
                    adStatusElement.className = 'ad-status click-prompt';
                    adStatusElement.innerHTML = **Click Required!** ${clicksProgress}/${requiredClicks} clicks completed. Run ad again and click!;
                } else {
                     adStatusElement.innerHTML = Starting Ad Process: ${requiredViews} Views, ${requiredClicks} Clicks needed.;
                }
            }
        }
        
        // --- 4. DATA FETCHERS ---
        function setupAppListener() {
            database.ref('/').on('value', (snapshot) => {
                const data = snapshot.val();
                
                const settings = data?.settings || { requiredAdViews: 2, requiredAdClicks: 1 };
                CURRENT_SETTINGS = {
                    views: parseInt(settings.requiredAdViews || 2),
                    clicks: parseInt(settings.requiredAdClicks || 1)
                };

                const appsObject = data?.apps;
                ALL_APPS_DATA = [];
                if (appsObject) {
                    ALL_APPS_DATA = Object.keys(appsObject).map(key => ({
                        id: key,
                        ...appsObject[key]
                    }));
                }
                renderAppList(ALL_APPS_DATA, CURRENT_SETTINGS);
            });
        }

        // --- 5. AD & DOWNLOAD LOGIC (Strict Sequential Enforcement) ---
        
        function runMonetagAd(appId, adStatusElement) {
            adStatusElement.innerText = 'Running Ad... Please wait.';
            // The actual Monetag SDK call
            return window[MONETAG_SDK_FUNC_NAME]({ 
                type: 'end',
                ymid: download_${appId}_${Date.now()}
            });
        }

        async function downloadApp(appId, downloadLink) {
            // Step 1: SDK Check
            if (!isMonetagSdkReady(appId)) {
                updateProgressUI(appId, CURRENT_SETTINGS, getProgress(appId));
                return;
            }

            const settings = CURRENT_SETTINGS;
            let progress = getProgress(appId);
            const adStatusElement = document.getElementById(ad-status-${appId});
            const requiredViews = settings.views;
            const requiredClicks = settings.clicks;
            
            // Step 2: Final Download Check
            if (progress.viewsCompleted >= requiredViews && progress.clicksCompleted >= requiredClicks) {
                adStatusElement.innerText = 'Download starting...';
                setTimeout(() => {
                    alert(App Download Successful: ${appId});
                    window.location.href = downloadLink;
                }, 500);
                return;
            }
            
            try {
                // --- Phase 1: Views ---
                if (progress.viewsCompleted < requiredViews) {
                    adStatusElement.innerText = Running Ad... View ${progress.viewsCompleted + 1} / ${requiredViews};
                    await runMonetagAd(appId, adStatusElement);
                    
                    // View Success
                    progress.viewsCompleted += 1;
                    setProgress(appId, progress.viewsCompleted, progress.clicksCompleted);
                    updateProgressUI(appId, settings, progress);
                    
                    if (progress.viewsCompleted >= requiredViews) {
                        // Views completed. Transition to the Click Phase alert.
                        if (requiredClicks > 0) {
                            alert(All ${requiredViews} views completed. Now, please run the ad process again and click the advertisement to complete the download requirement.);
                        }
                        // Re-call to update button state and potentially trigger final download if requiredClicks is 0
                        downloadApp(appId, downloadLink); 
                    } else {
                         alert('Ad view complete. Continue the process for the next view.');
                    }
                    return;
                }

                // --- Phase 2: Clicks ---
                if (progress.viewsCompleted >= requiredViews && progress.clicksCompleted < requiredClicks) {
                    
                    // Set status for final click expectation
                    adStatusElement.className = 'ad-status click-prompt';
                    adStatusElement.innerHTML = **Click Remaining:** ${progress.clicksCompleted + 1} / ${requiredClicks} clicks required.;

                    await runMonetagAd(appId, adStatusElement);
                    
                    // Ad finished. Now prompt for user confirmation of the click.
                    const clicked = confirm("DID YOU CLICK THE ADVERTISEMENT? \n\nClick 'OK' to confirm the click for download, or 'Cancel' to try again.");
                    
                    if (clicked) {
                        // Click Success
                        progress.clicksCompleted += 1;
                        setProgress(appId, progress.viewsCompleted, progress.clicksCompleted);
                        updateProgressUI(appId, settings, progress);
                        
                        if (progress.clicksCompleted >= requiredClicks) {
                            alert('All clicks confirmed. Starting download.');
                            downloadApp(appId, downloadLink); // Final call to download
                        } else {
                            alert('Click confirmed. Continue running the ad for remaining clicks.');
                        }
                    } else {
                        // Click Failure/Not confirmed
                        alert('Click not confirmed. You must click the ad and confirm to get the download.');
                        updateProgressUI(appId, settings, progress);
                    }
                    return;
                }

            } catch (error) {
                // Ad Failed or Closed (Monetag .catch block)
                adStatusElement.innerText = Ad Failed or Closed. Click 'Continue Ad Process'.;
                alert('The ad process failed. Please try again.');
                updateProgressUI(appId, settings, progress);
            }
        }


        // --- 6. UI RENDERING & UTILITIES ---
        function renderAppList(appArray, settings) {
            const appListElement = document.getElementById('app-list');
            appListElement.innerHTML = '';
            
            if (appArray.length === 0) {
                 appListElement.innerHTML = '<p style="padding: 15px; text-align: center; color: var(--text-color);">No apps found.</p>';
                 return;
            }

            appArray.forEach(app => {
                const progress = getProgress(app.id);
                const isReady = (progress.viewsCompleted >= settings.views) && (progress.clicksCompleted >= settings.clicks);
                const initialPercentage = Math.min(100, (progress.viewsCompleted / settings.views) * 100);
                
                const appCard = document.createElement('div');
                appCard.className = 'app-card';
                appCard.innerHTML = 
                    <div class="app-banner" style="background-image: url('${app.bannerUrl}'); height: 150px; background-size: cover; background-position: center;">
                        <span class="app-banner-text">Download</span>
                    </div>
                    <div class="app-details">
                        <h3 class="app-title">${app.title || 'Untitled App'}</h3>
                        
                        <div class="progress-container">
                            <div class="progress-bar" id="progress-bar-${app.id}" style="width: ${initialPercentage}%;"></div>
                        </div>

                        <p class="ad-status" id="ad-status-${app.id}">Checking SDK...</p>
                        
                        <button id="download-btn-${app.id}" onclick="downloadApp('${app.id}', '${app.downloadLink}')">
                            ${isReady ? 'Download App' : 'Start Ad Process'}
                        </button>
                    </div>
                ;
                appListElement.appendChild(appCard);
                
                // Initial status update
                updateProgressUI(app.id, settings, progress);
                // Check SDK status on load
                if(!isMonetagSdkReady(app.id)) {
                     document.getElementById(download-btn-${app.id}).disabled = true;
                }
            });
        }
        
        function filterApps() {
            const query = document.getElementById('search-input').value.toLowerCase();
            const filteredApps = ALL_APPS_DATA.filter(app => (app.title || '').toLowerCase().includes(query));
            renderAppList(filteredApps, CURRENT_SETTINGS);
        }
        
        function setupThemeToggle() {
            const toggleButton = document.getElementById('theme-toggle');
            const savedTheme = localStorage.getItem('theme') || 'light';
            document.documentElement.setAttribute('data-theme', savedTheme);
            toggleButton.innerText = savedTheme === 'dark' ? '☀️ Light Mode' : '🌛 Dark Mode';

            toggleButton.addEventListener('click', () => {
                const currentTheme = document.documentElement.getAttribute('data-theme');
                const newTheme = currentTheme === 'light' ? 'dark' : 'light';
                document.documentElement.setAttribute('data-theme', newTheme);
                localStorage.setItem('theme', newTheme);
                toggleButton.innerText = newTheme === 'dark' ? '☀️ Light Mode' : '🌛 Dark Mode';
            });
        }

        // --- 7. INITIALIZATION ---
        
        document.addEventListener('DOMContentLoaded', () => {
            setupThemeToggle();
            setupAppListener(); 
        });
    </script>

</body>
</html>
