# College-Project
PBL daa



<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>IITB Maps | Pro Precision Navigator</title>
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Leaflet CSS for Real Maps -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700&display=swap');
        
        :root {
            --primary: #1a73e8;
        }

        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            background: #0f172a; 
            color: #3c4043;
            overflow: hidden;
            overscroll-behavior-y: contain;
            margin: 0;
            padding: 0;
        }

        /* Fullscreen Map Base */
        #map {
            position: absolute;
            inset: 0;
            width: 100vw;
            height: 100vh;
            z-index: 1;
            background: #0f172a !important; /* Beautiful dark void for missing offline tiles */
        }

        /* * ==========================================
         * THE GOOGLE MAPS STYLE BOTTOM SHEET UI 
         * ==========================================
         */
        .ui-panel {
            position: absolute;
            bottom: 0;
            left: 0;
            right: 0;
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(25px);
            -webkit-backdrop-filter: blur(25px);
            border-radius: 32px 32px 0 0;
            height: 55vh; 
            display: flex;
            flex-direction: column;
            z-index: 1000;
            box-shadow: 0 -15px 50px rgba(0,0,0,0.2);
            transition: transform 0.5s cubic-bezier(0.3, 1.05, 0.3, 1);
        }

        .ui-panel.collapsed {
            transform: translateY(calc(100% - 80px));
        }

        @media (min-width: 768px) {
            .ui-panel {
                top: 24px; left: 24px; bottom: 24px; right: auto;
                width: 420px; height: auto; border-radius: 28px;
                box-shadow: 12px 0 40px rgba(0,0,0,0.15);
            }
            .ui-panel.collapsed { transform: translateX(calc(-100% - 24px)); }
        }

        .panel-header {
            padding: 16px 24px 16px 24px; cursor: pointer;
            border-bottom: 1px solid rgba(0,0,0,0.04);
            display: flex; flex-direction: column; align-items: center; flex-shrink: 0;
        }
        
        .drag-handle {
            width: 48px; height: 6px; background: #cbd5e1;
            border-radius: 10px; margin-bottom: 14px;
            transition: background 0.3s;
        }
        .panel-header:hover .drag-handle { background: #94a3b8; }

        /* HUD Styling (Directions overlay at top) */
        .instruction-hud {
            background: rgba(15, 23, 42, 0.95);
            backdrop-filter: blur(15px);
            color: white; border-radius: 24px;
            padding: 18px 24px; display: none; flex-direction: column;
            position: absolute; top: 16px; left: 16px; right: 16px;
            z-index: 1200; box-shadow: 0 20px 40px rgba(0,0,0,0.4);
            animation: slideDown 0.4s cubic-bezier(0.2, 0.8, 0.2, 1); 
            border: 1px solid rgba(255,255,255,0.15);
        }
        @media (min-width: 768px) { .instruction-hud { left: auto; right: 24px; top: 24px; width: 380px; } }

        @keyframes slideDown { from { transform: translateY(-30px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }

        .hud-progress-container {
            width: 100%; height: 6px; background: rgba(255,255,255,0.1);
            border-radius: 3px; margin-top: 18px; overflow: hidden;
        }
        #hudProgressBar {
            height: 100%; width: 0%; background: linear-gradient(90deg, #3b82f6, #60a5fa);
            box-shadow: 0 0 15px rgba(59, 130, 246, 0.8); transition: width 0.3s ease;
        }

        /* Form styling */
        .selection-card { padding: 14px 18px; border-radius: 16px; margin-bottom: 10px; transition: all 0.2s; position: relative; }
        .selection-card:focus-within { transform: scale(1.02); box-shadow: 0 5px 15px rgba(0,0,0,0.05); }
        .card-source { background: #f0fdf4; border: 1px solid #bbf7d0; }
        .card-destination { background: #fef2f2; border: 1px solid #fecaca; }

        .category-chip {
            padding: 10px 18px; border-radius: 20px; font-size: 12px;
            font-weight: 700; background: #f1f5f9; color: #64748b;
            cursor: pointer; transition: all 0.3s; white-space: nowrap; border: 1px solid transparent;
        }
        .category-chip:hover { background: #e2e8f0; }
        .category-chip.active { background: #0f172a; color: white; box-shadow: 0 4px 12px rgba(15, 23, 42, 0.2); }
        
        .no-scrollbar::-webkit-scrollbar { display: none; }

        /* Leaflet Map Customizations */
        .leaflet-popup-content-wrapper { border-radius: 20px; padding: 0; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.3) !important; }
        .leaflet-popup-content { margin: 0; width: 240px !important; }
        .popup-img { width: 100%; height: 130px; object-fit: cover; }
        .popup-text { padding: 16px; }
        
        /* Custom User Marker */
        .user-nav-marker {
            width: 28px; height: 28px; background: white;
            border: 6px solid #3b82f6; border-radius: 50%;
            box-shadow: 0 0 25px rgba(59, 130, 246, 0.9);
            transition: all 0.1s linear;
        }
        .user-nav-marker::after {
            content: ''; position: absolute; top: -10px; left: -10px; right: -10px; bottom: -10px;
            border-radius: 50%; background: rgba(59, 130, 246, 0.2);
            animation: pulse 1.5s infinite;
        }
        @keyframes pulse { 0% { transform: scale(0.8); opacity: 1; } 100% { transform: scale(2); opacity: 0; } }

        /* Custom Pins */
        .custom-pin { display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .custom-pin-inner {
            width: 16px; height: 16px; background: #ea4335;
            border: 3px solid white; border-radius: 50%;
            box-shadow: 0 4px 10px rgba(0,0,0,0.5);
        }
        .custom-pin-label {
            font-family: 'Plus Jakarta Sans', sans-serif; font-size: 11px; font-weight: 800;
            background: rgba(255,255,255,0.95); padding: 4px 8px; border-radius: 6px;
            margin-top: 6px; white-space: nowrap; border: 1px solid #e2e8f0;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1); color: #1e293b;
        }
        
        /* Loader for Routing API */
        .loader {
            border: 3px solid rgba(255,255,255,0.3); border-top: 3px solid white;
            border-radius: 50%; width: 18px; height: 18px; animation: spin 1s linear infinite; display: none;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

        /* GPS Pulse Animation */
        .gps-active { animation: textPulse 2s infinite; color: #16a34a; }
        @keyframes textPulse { 0% { opacity: 1; } 50% { opacity: 0.5; } 100% { opacity: 1; } }
    </style>
</head>
<body>

    <!-- LIVE MAP LAYER -->
    <div id="map"></div>

    <!-- FLOATING MAP ACTIONS -->
    <div class="absolute right-4 top-24 md:top-6 z-[1000] flex flex-col gap-3 pointer-events-none">
        <button onclick="recenterMap()" class="w-12 h-12 bg-white/90 backdrop-blur-md rounded-2xl shadow-xl border border-slate-100 text-slate-700 flex items-center justify-center hover:bg-blue-50 hover:text-blue-600 transition-all pointer-events-auto" title="Recenter to Location">
            <i class="fa-solid fa-location-crosshairs text-xl"></i>
        </button>
        <button onclick="cycleMapStyle()" class="w-12 h-12 bg-white/90 backdrop-blur-md rounded-2xl shadow-xl border border-slate-100 text-slate-700 flex items-center justify-center hover:bg-blue-50 hover:text-blue-600 transition-all pointer-events-auto" title="Switch Map View">
            <i class="fa-solid fa-layer-group text-xl"></i>
        </button>
    </div>

    <!-- TOAST NOTIFICATION -->
    <div id="toastNotif" class="fixed top-8 left-1/2 -translate-x-1/2 bg-slate-800 text-white px-5 py-2.5 rounded-full text-xs font-bold tracking-wide opacity-0 pointer-events-none transition-opacity duration-300 z-[2000] shadow-2xl flex items-center gap-2">
        <i class="fa-solid fa-check-circle text-green-400" id="toastIcon"></i> <span id="toastText">Map Style Updated</span>
    </div>

    <!-- HUD LAYER -->
    <div id="instructionHud" class="instruction-hud">
        <div class="flex items-center gap-4">
            <div class="bg-gradient-to-br from-blue-500 to-blue-600 text-white w-14 h-14 rounded-2xl flex items-center justify-center shrink-0 shadow-inner border border-blue-400/30">
                <i id="hudIcon" class="fa-solid fa-arrow-up text-2xl transition-transform duration-500"></i>
            </div>
            <div class="flex-1">
                <p id="turnLabel" class="text-[10px] font-black uppercase tracking-[0.15em] text-blue-300 mb-0.5">Continue Straight</p>
                <p id="hudMain" class="font-bold text-xl leading-tight truncate tracking-tight">Heading to Destination</p>
                <p id="hudSub" class="text-xs text-slate-300 font-medium mt-1">Starting walk...</p>
            </div>
            <button onclick="stopNavigation()" class="text-slate-400 hover:text-white transition-colors bg-slate-800 p-2.5 rounded-full">
                <i class="fa-solid fa-xmark text-xl w-6 h-6 flex items-center justify-center"></i>
            </button>
        </div>
        <div class="hud-progress-container">
            <div id="hudProgressBar"></div>
        </div>
    </div>

    <!-- UI PANEL LAYER -->
    <div class="ui-panel" id="uiPanel">
        <div class="panel-header" onclick="togglePanel()">
            <div class="drag-handle md:hidden"></div>
            <div class="flex items-center justify-between w-full">
                <h2 class="text-xl font-black tracking-tight text-slate-800 flex items-center gap-3">
                    <div class="bg-gradient-to-br from-blue-600 to-blue-800 w-9 h-9 rounded-xl text-white flex items-center justify-center text-sm shadow-lg shadow-blue-200">
                        <i class="fa-solid fa-satellite-dish"></i>
                    </div>
                    IITB Nav Pro
                </h2>
                <div class="flex items-center gap-3">
                    <i class="fa-solid fa-chevron-down text-slate-400 md:hidden text-sm"></i>
                    <button id="installAppBtn" onclick="event.stopPropagation(); triggerInstall()" class="bg-blue-50 text-blue-600 border border-blue-200 px-3.5 py-1.5 rounded-xl text-xs font-bold hover:bg-blue-100 flex items-center gap-1.5 transition-colors">
                        <i class="fa-solid fa-download"></i> Install
                    </button>
                </div>
            </div>
        </div>

        <div class="flex-1 overflow-y-auto p-6 pt-2 flex flex-col no-scrollbar">
            <!-- Routing Inputs -->
            <div class="space-y-3 mb-5">
                <div class="selection-card card-source flex items-center gap-4">
                    <i class="fa-solid fa-circle-dot text-green-600 text-lg" id="startIcon"></i>
                    <div class="flex-1 pr-10">
                        <label class="text-[10px] font-black text-green-800 uppercase block tracking-widest opacity-80">Starting From</label>
                        <select id="startSelect" onchange="initRoute()" class="w-full bg-transparent border-0 text-[15px] font-bold outline-none text-slate-800 mt-1 cursor-pointer">
                            <option value="gps" id="gpsOption" class="hidden">📍 My Current GPS Location</option>
                        </select>
                    </div>
                    <button onclick="useCurrentLocation()" class="absolute right-4 top-1/2 -translate-y-1/2 w-8 h-8 rounded-full bg-green-100 text-green-600 flex items-center justify-center hover:bg-green-200 transition-colors" title="Use My Location">
                        <i class="fa-solid fa-location-arrow text-sm"></i>
                    </button>
                </div>

                <div class="selection-card card-destination flex items-center gap-4">
                    <i class="fa-solid fa-location-dot text-red-500 text-lg"></i>
                    <div class="flex-1">
                        <label class="text-[10px] font-black text-red-800 uppercase block tracking-widest opacity-80">Going To</label>
                        <select id="destSelect" onchange="initRoute()" class="w-full bg-transparent border-0 text-[15px] font-bold outline-none text-slate-800 mt-1 cursor-pointer">
                        </select>
                    </div>
                </div>
            </div>

            <!-- Stats & Walk Button -->
            <div id="statsPanel" class="hidden mb-6 bg-white border border-slate-200 rounded-2xl p-5 shadow-sm">
                <div class="flex justify-between items-center mb-5 px-1">
                    <div>
                        <span class="text-[10px] font-black text-slate-400 uppercase tracking-widest block mb-1">Actual Route</span>
                        <p id="distVal" class="text-2xl font-black text-slate-800 leading-none">-- km</p>
                    </div>
                    <div class="text-right">
                        <span class="text-[10px] font-black text-slate-400 uppercase tracking-widest block mb-1">Est. Time</span>
                        <p id="etaVal" class="text-2xl font-black text-blue-600 leading-none">-- min</p>
                    </div>
                </div>
                <button id="navBtn" onclick="startNavigation()" class="w-full py-4 bg-gradient-to-r from-blue-600 to-blue-700 text-white rounded-xl font-bold shadow-xl shadow-blue-200/50 hover:shadow-blue-300/50 hover:-translate-y-0.5 transition-all flex items-center justify-center gap-2 text-base tracking-wide">
                    <div class="loader" id="navLoader"></div>
                    <i class="fa-solid fa-person-walking" id="navIcon"></i> <span id="navText">Start Real Navigation</span>
                </button>
            </div>

            <hr class="border-slate-100 mb-5">

            <!-- Search Bar -->
            <div class="relative mb-4 px-1">
                <i class="fa-solid fa-search absolute left-4 top-1/2 -translate-y-1/2 text-slate-400 text-sm"></i>
                <input type="text" id="searchInput" onkeyup="searchDir()" placeholder="Search locations, hostels, departments..." class="w-full bg-slate-100 border border-slate-200 rounded-xl py-3 pl-10 pr-4 text-[13px] font-bold text-slate-700 outline-none focus:ring-2 focus:ring-blue-500 focus:bg-white transition-all">
            </div>

            <!-- Directory Filters -->
            <div class="flex gap-2 mb-5 overflow-x-auto pb-2 no-scrollbar shrink-0 px-1">
                <div class="category-chip active" id="chip-all" onclick="filterDir('all')">All Places</div>
                <div class="category-chip" id="chip-academic" onclick="filterDir('academic')">Academic</div>
                <div class="category-chip" id="chip-hostel" onclick="filterDir('hostel')">Hostels</div>
                <div class="category-chip" id="chip-recreation" onclick="filterDir('recreation')">Facilities</div>
            </div>

            <div class="flex-1 flex flex-col gap-3.5 no-scrollbar" id="directoryList">
                <!-- Directory List Injected via JS -->
            </div>
        </div>
    </div>

    <!-- Leaflet JS -->
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

    <script>
        // --- 1. ACCURATE REAL LAT/LNG POI & OFFLINE GRAPH DATA ---
        // Moved to the top so Service Worker can read image URLs for instant pre-caching
        const CAMPUS_GRAPH = {
            nodes: {
                main_gate: { lat: 19.1254, lng: 72.9164, name: "Main Gate", cat: "gate", desc: "Main Entrance (Powai)", img: "https://images.unsplash.com/photo-1541339907198-e08756dedf3f?w=400&h=300&fit=crop&q=80" },
                hospital_jct: { lat: 19.1265, lng: 72.9160, name: "Hospital Jct", cat: "jct" },
                academic_jct: { lat: 19.1300, lng: 72.9150, name: "Academic Spine", cat: "jct" },
                admin_jct: { lat: 19.1325, lng: 72.9150, name: "Admin Circle", cat: "jct" },
                residential_jct: { lat: 19.1350, lng: 72.9110, name: "Hostel Loop", cat: "jct" },
                kv_gate: { lat: 19.1408, lng: 72.9086, name: "KV Gate", cat: "gate", desc: "North Entry Point", img: "https://images.unsplash.com/photo-1523050854058-8df90110c9f1?w=400&h=300&fit=crop&q=80" },
                hospital: { lat: 19.1264, lng: 72.9157, name: "Campus Hospital", cat: "recreation", desc: "Medical Services", img: "https://images.unsplash.com/photo-1519494026892-80bbd2d6fd0d?w=400&h=300&fit=crop&q=80" },
                convo_hall: { lat: 19.1338, lng: 72.9142, name: "Convocation Hall", cat: "academic", desc: "Main Events", img: "https://images.unsplash.com/photo-1509062522246-3755977927d7?w=400&h=300&fit=crop&q=80" },
                main_bldg: { lat: 19.1332, lng: 72.9135, name: "Main Building", cat: "academic", desc: "Admin HQ", img: "https://images.unsplash.com/photo-1562774053-701939374585?w=400&h=300&fit=crop&q=80" },
                lhc: { lat: 19.1316, lng: 72.9163, name: "LHC Complex", cat: "academic", desc: "Lecture Halls", img: "https://images.unsplash.com/photo-1577416412292-747c6607f055?w=400&h=300&fit=crop&q=80" },
                library: { lat: 19.1318, lng: 72.9155, name: "Central Library", cat: "academic", desc: "Research & Study", img: "https://images.unsplash.com/photo-1541829070764-84a7d30dd3f3?w=400&h=300&fit=crop&q=80" },
                cse: { lat: 19.1311, lng: 72.9154, name: "CSE Dept", cat: "academic", desc: "Computer Science", img: "https://images.unsplash.com/photo-1517694712202-14dd9538aa97?w=400&h=300&fit=crop&q=80" },
                ee: { lat: 19.1315, lng: 72.9147, name: "EE Dept", cat: "academic", desc: "Electrical Eng", img: "https://images.unsplash.com/photo-1532094349884-543bc11b234d?w=400&h=300&fit=crop&q=80" },
                mech: { lat: 19.1328, lng: 72.9140, name: "Mech Dept", cat: "academic", desc: "Mechanical Blocks", img: "https://images.unsplash.com/photo-1581092160562-40aa08e78837?w=400&h=300&fit=crop&q=80" },
                chem_eng: { lat: 19.1338, lng: 72.9152, name: "Chem Eng", cat: "academic", desc: "Chemical Eng", img: "https://images.unsplash.com/photo-1532187863486-abf9dbad1b69?w=400&h=300&fit=crop&q=80" },
                vmcc: { lat: 19.1342, lng: 72.9138, name: "VMCC Center", cat: "academic", desc: "Convention Hub", img: "https://images.unsplash.com/photo-1497366216548-37526070297c?w=400&h=300&fit=crop&q=80" },
                h12: { lat: 19.1352, lng: 72.9061, name: "Hostels 12-14", cat: "hostel", desc: "Hillside Cluster", img: "https://images.unsplash.com/photo-1555854877-bab0e564b8d5?w=400&h=300&fit=crop&q=80" },
                h15: { lat: 19.1367, lng: 72.9080, name: "Hostels 15-16", cat: "hostel", desc: "High-rise Dorms", img: "https://images.unsplash.com/photo-1600585154340-be6161a56a0c?w=400&h=300&fit=crop&q=80" },
                h1: { lat: 19.1265, lng: 72.9120, name: "Hostel 1", cat: "hostel", desc: "Lakeside Living", img: "https://images.unsplash.com/photo-1522771731515-385a53609836?w=400&h=300&fit=crop&q=80" },
                h7: { lat: 19.1287, lng: 72.9092, name: "Hostels 7-9", cat: "hostel", desc: "Old Campus", img: "https://images.unsplash.com/photo-1564013799919-ab600027ffc6?w=400&h=300&fit=crop&q=80" },
                sac: { lat: 19.1354, lng: 72.9123, name: "SAC Hub", cat: "recreation", desc: "Gymkhana", img: "https://images.unsplash.com/photo-1534438327276-14e5300c3a48?w=400&h=300&fit=crop&q=80" },
                pool: { lat: 19.1360, lng: 72.9118, name: "Swimming Pool", cat: "recreation", desc: "Student Recreation", img: "https://images.unsplash.com/photo-1576013551627-1186e0104bf7?w=400&h=300&fit=crop&q=80" },
                gulmohar: { lat: 19.1347, lng: 72.9161, name: "Gulmohar Cafe", cat: "recreation", desc: "Dining Hall", img: "https://images.unsplash.com/photo-1555396273-367ea4eb4db5?w=400&h=300&fit=crop&q=80" }
            },
            edges: [
                ['main_gate', 'hospital_jct'], ['hospital_jct', 'hospital'],
                ['hospital_jct', 'academic_jct'], ['academic_jct', 'convo_hall'],
                ['academic_jct', 'admin_jct'], ['admin_jct', 'main_bldg'],
                ['main_bldg', 'lhc'], ['lhc', 'library'], ['admin_jct', 'cse'],
                ['cse', 'ee'], ['ee', 'mech'], ['admin_jct', 'residential_jct'],
                ['residential_jct', 'kv_gate'], ['residential_jct', 'h12'],
                ['h12', 'h15'], ['residential_jct', 'vmcc'],
                ['vmcc', 'sac'], ['sac', 'pool'],
                ['hospital_jct', 'h7'], ['h7', 'h1'],
                ['admin_jct', 'chem_eng'], ['hospital', 'gulmohar']
            ]
        };

        // --- 2. PWA APP INSTALLATION & BULLETPROOF OFFLINE CACHING ---
        let deferredPrompt;
        const manifest = {
            name: "IITB Satellite Navigator", short_name: "IITB Nav", start_url: ".", display: "standalone",
            background_color: "#e2e8f0", theme_color: "#1a73e8",
            icons: [{ src: "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='%231a73e8'><path d='M12 2L2 22l10-3 10 3L12 2z'/></svg>", sizes: "192x192", type: "image/svg+xml" }]
        };
        const manifestBlob = new Blob([JSON.stringify(manifest)], { type: 'application/json' });
        const link = document.createElement('link'); link.rel = 'manifest'; link.href = URL.createObjectURL(manifestBlob);
        document.head.appendChild(link);

        // Pre-extract all Unsplash image URLs so the SW downloads them instantly
        const locationImages = Object.values(CAMPUS_GRAPH.nodes).filter(n => n.img).map(n => `'${n.img}'`).join(',\n                ');

        const swCode = `
            const CACHE_NAME = 'iitb-nav-pro-v9';
            const CORE_ASSETS = [
                location.href,
                'https://cdn.tailwindcss.com',
                'https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css',
                'https://unpkg.com/leaflet@1.9.4/dist/leaflet.css',
                'https://unpkg.com/leaflet@1.9.4/dist/leaflet.js',
                ${locationImages}
            ];

            self.addEventListener('install', e => { 
                // GUARANTEE all core scripts and location photography is downloaded immediately
                e.waitUntil(caches.open(CACHE_NAME).then(c => c.addAll(CORE_ASSETS))); 
            });

            self.addEventListener('fetch', e => {
                if (e.request.method !== 'GET') return;
                e.respondWith(
                    caches.match(e.request).then(cachedResponse => {
                        if (cachedResponse) return cachedResponse;
                        return fetch(e.request).then(networkResponse => {
                            if(!networkResponse || networkResponse.status !== 200 || (networkResponse.type !== 'basic' && networkResponse.type !== 'cors')) return networkResponse;
                            const url = e.request.url;
                            
                            // FIX 1: Catch ALL Google Subdomains (mt0, mt1, mt2, mt3) and Carto subdomains
                            const isCacheable = url.includes('images.unsplash.com') || 
                                                url.includes('google.com/vt') || 
                                                url.includes('cartocdn.com') || 
                                                url.includes('router.project-osrm.org');

                            if (isCacheable) {
                                const responseToCache = networkResponse.clone();
                                caches.open(CACHE_NAME).then(cache => cache.put(e.request, responseToCache));
                            }
                            return networkResponse;
                        }).catch(() => {
                            console.warn('Offline: Could not fetch', e.request.url);
                        });
                    })
                );
            });
        `;
        const swBlob = new Blob([swCode], { type: 'application/javascript' });
        if ('serviceWorker' in navigator) { try { navigator.serviceWorker.register(URL.createObjectURL(swBlob)).catch(()=>{}); } catch(e){} }

        function triggerInstall() {
            if (deferredPrompt) {
                deferredPrompt.prompt();
                deferredPrompt.userChoice.then((res) => { if (res.outcome === 'accepted') document.getElementById('installAppBtn').style.display = 'none'; deferredPrompt = null; });
            } else {
                alert("To install app:\n• iPhone: Tap 'Share' icon at bottom, then 'Add to Home Screen'.\n• Android: Tap browser menu (⋮), then 'Install App'.");
            }
        }

        // --- 3. LEAFLET MAP & OSRM ROUTING ENGINE ---
        let map, routeLayer, routeGlowLayer, userMarker, pinMarkers = [];
        let activeRouteCoords = []; 
        let userPos = { lat: 19.1254, lng: 72.9164 }; // Default start
        let isUsingLiveGPS = false; 
        let navInterval = null;
        let currentStep = 0;
        let totalDistanceMeters = 0;

        let satelliteGroup, darkLayer, lightLayer;
        let currentStyle = 0; 

        // Transparent pixel fallback for missing tiles
        const emptyTile = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII=';

        function initMap() {
            // FIX 3: GEOFENCING BOUNDS - Locks the camera to IITB to prevent memory waste and focus offline cache perfectly
            const campusBounds = L.latLngBounds(
                [19.1150, 72.8900], // Southwest
                [19.1500, 72.9300]  // Northeast
            );

            map = L.map('map', { 
                zoomControl: false,
                maxBounds: campusBounds,
                maxBoundsViscosity: 1.0,
                minZoom: 14 // Prevent zooming out into empty space
            }).setView([19.1334, 72.9133], 17);
            
            // FIX 1 pt2: Use {s} with explicit subdomains so Leaflet matches the SW exactly
            const satHybrid = L.tileLayer('https://mt{s}.google.com/vt/lyrs=y&x={x}&y={y}&z={z}', { subdomains: '0123', maxZoom: 22, maxNativeZoom: 20, errorTileUrl: emptyTile });
            satelliteGroup = L.layerGroup([satHybrid]);
            lightLayer = L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}.png', { subdomains: 'abcd', maxZoom: 22, maxNativeZoom: 20, errorTileUrl: emptyTile });
            darkLayer = L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png', { subdomains: 'abcd', maxZoom: 22, maxNativeZoom: 20, errorTileUrl: emptyTile });

            satelliteGroup.addTo(map);

            populateSelects();
            renderMapPins();
            filterDir('all');
            
            document.getElementById('startSelect').value = 'main_gate';
            document.getElementById('destSelect').value = 'cse';
            initRoute(); 
        }

        // --- NEW REAL-TIME GPS GEOLOCATION ---
        function useCurrentLocation() {
            if (!navigator.geolocation) {
                showToast("GPS not supported by your browser", "fa-circle-exclamation", "text-red-400");
                return;
            }
            showToast("Locating your GPS signal...", "fa-spinner fa-spin", "text-blue-400");
            
            navigator.geolocation.getCurrentPosition(
                (position) => {
                    isUsingLiveGPS = true;
                    userPos = { lat: position.coords.latitude, lng: position.coords.longitude };
                    document.getElementById('gpsOption').classList.remove('hidden');
                    document.getElementById('startSelect').value = 'gps';
                    document.getElementById('startIcon').classList.add('gps-active');
                    showToast("GPS Location Acquired!");
                    initRoute();
                },
                (error) => {
                    let msg = "Could not get GPS location.";
                    if (error.code === 1) msg = "Please allow location permissions.";
                    showToast(msg, "fa-circle-exclamation", "text-red-400");
                },
                { enableHighAccuracy: true, timeout: 5000, maximumAge: 0 }
            );
        }

        // --- REAL-ROAD ROUTING LOGIC & OFFLINE DIJKSTRA FALLBACK ---
        
        function solveDijkstraLocal(startKey, destKey, customStartCoords) {
            const adj = {};
            Object.keys(CAMPUS_GRAPH.nodes).forEach(k => adj[k] = []);
            CAMPUS_GRAPH.edges.forEach(e => {
                const [u, v] = e;
                if(!adj[u] || !adj[v]) return;
                const d = getDistance(CAMPUS_GRAPH.nodes[u].lat, CAMPUS_GRAPH.nodes[u].lng, CAMPUS_GRAPH.nodes[v].lat, CAMPUS_GRAPH.nodes[v].lng);
                adj[u].push({node: v, weight: d}); adj[v].push({node: u, weight: d});
            });

            if(startKey === 'gps' && customStartCoords) {
                adj['gps'] = []; let nearest = null; let minDist = Infinity;
                Object.keys(CAMPUS_GRAPH.nodes).forEach(k => {
                    const d = getDistance(customStartCoords.lat, customStartCoords.lng, CAMPUS_GRAPH.nodes[k].lat, CAMPUS_GRAPH.nodes[k].lng);
                    if(d < minDist) { minDist = d; nearest = k; }
                });
                adj['gps'].push({node: nearest, weight: minDist}); adj[nearest].push({node: 'gps', weight: minDist});
            }

            const dist = {}; const prev = {}; const pq = new Set();
            const allNodes = Object.keys(CAMPUS_GRAPH.nodes);
            if(startKey === 'gps') allNodes.push('gps');

            allNodes.forEach(n => { dist[n] = Infinity; prev[n] = null; pq.add(n); });
            dist[startKey] = 0;

            while(pq.size > 0) {
                let u = null; let min = Infinity;
                pq.forEach(n => { if(dist[n] < min) { min = dist[n]; u = n; } });
                if(u === null || u === destKey) break;
                pq.delete(u);

                adj[u].forEach(neighbor => {
                    if(!pq.has(neighbor.node)) return;
                    const alt = dist[u] + neighbor.weight;
                    if(alt < dist[neighbor.node]) { dist[neighbor.node] = alt; prev[neighbor.node] = u; }
                });
            }

            const path = []; let curr = destKey;
            while(curr) { path.unshift(curr); curr = prev[curr]; }
            return path;
        }

        async function initRoute() {
            const startKey = document.getElementById('startSelect').value;
            const destKey = document.getElementById('destSelect').value;
            
            if (startKey === destKey) {
                if(routeLayer) map.removeLayer(routeLayer);
                if(routeGlowLayer) map.removeLayer(routeGlowLayer);
                if(userMarker) map.removeLayer(userMarker);
                activeRouteCoords = [];
                document.getElementById('statsPanel').classList.add('hidden');
                return;
            }
            
            let startCoordObj = userPos;
            if (startKey !== 'gps') {
                isUsingLiveGPS = false;
                document.getElementById('startIcon').classList.remove('gps-active');
                const startNode = CAMPUS_GRAPH.nodes[startKey];
                startCoordObj = { lat: startNode.lat, lng: startNode.lng };
                userPos = { ...startCoordObj }; 
            }
            
            const destNode = CAMPUS_GRAPH.nodes[destKey];
            document.getElementById('statsPanel').classList.remove('hidden');
            
            document.getElementById('navIcon').style.display = 'none';
            document.getElementById('navLoader').style.display = 'inline-block';
            document.getElementById('navText').innerText = "Calculating Route...";
            document.getElementById('navBtn').disabled = true;

            try {
                // Fetch exact road geometries from OSRM
                const response = await fetch(`https://router.project-osrm.org/route/v1/foot/${startCoordObj.lng},${startCoordObj.lat};${destNode.lng},${destNode.lat}?geometries=geojson&overview=full`);
                const data = await response.json();

                if(data.routes && data.routes.length > 0) {
                    const route = data.routes[0];
                    activeRouteCoords = route.geometry.coordinates.map(coord => ({ lat: coord[1], lng: coord[0] }));
                    totalDistanceMeters = route.distance;
                    const durationMins = Math.ceil(route.duration / 60);

                    document.getElementById('distVal').innerText = (totalDistanceMeters / 1000).toFixed(2) + " km";
                    document.getElementById('etaVal').innerText = durationMins + " min";
                } else { throw new Error("No route"); }
            } catch (error) {
                console.warn("Routing API offline, falling back to local Dijkstra graph", error);
                showToast("Offline Mode: Using Local Network", "fa-wifi", "text-orange-400");
                
                // PERFECT OFFLINE FALLBACK - Calculate path across known junctions internally
                const pathKeys = solveDijkstraLocal(startKey, destKey, startCoordObj);
                
                activeRouteCoords = pathKeys.map(k => {
                    if(k === 'gps') return {lat: startCoordObj.lat, lng: startCoordObj.lng};
                    return {lat: CAMPUS_GRAPH.nodes[k].lat, lng: CAMPUS_GRAPH.nodes[k].lng};
                });

                totalDistanceMeters = 0;
                for(let i=0; i<activeRouteCoords.length-1; i++) {
                    totalDistanceMeters += getDistance(activeRouteCoords[i].lat, activeRouteCoords[i].lng, activeRouteCoords[i+1].lat, activeRouteCoords[i+1].lng);
                }
                
                document.getElementById('distVal').innerText = (totalDistanceMeters / 1000).toFixed(2) + " km";
                document.getElementById('etaVal').innerText = Math.ceil(totalDistanceMeters / 83) + " min";
            }

            document.getElementById('navIcon').style.display = 'inline-block';
            document.getElementById('navLoader').style.display = 'none';
            document.getElementById('navText').innerText = "Start Navigation";
            document.getElementById('navBtn').disabled = false;

            drawRouteOnMap();
        }

        function drawRouteOnMap() {
            if(routeLayer) map.removeLayer(routeLayer);
            if(routeGlowLayer) map.removeLayer(routeGlowLayer);
            if(userMarker) map.removeLayer(userMarker);

            if (!activeRouteCoords || activeRouteCoords.length === 0) return;

            const latlngs = activeRouteCoords.map(c => [c.lat, c.lng]);

            routeGlowLayer = L.polyline(latlngs, { color: '#3b82f6', weight: 12, opacity: 0.4, lineJoin: 'round', lineCap: 'round' }).addTo(map);
            routeLayer = L.polyline(latlngs, { color: '#ffffff', weight: 4, opacity: 1, dashArray: '8, 12', lineJoin: 'round', lineCap: 'round' }).addTo(map);

            const userIcon = L.divIcon({ className: 'user-nav-marker', iconSize: [28, 28], iconAnchor: [14, 14] });
            userMarker = L.marker([userPos.lat, userPos.lng], {icon: userIcon, zIndexOffset: 1000}).addTo(map);

            map.fitBounds(routeLayer.getBounds(), { paddingBottomRight: [0, window.innerHeight * 0.55], paddingTopLeft: [40, 40], maxZoom: 18 });
        }

        // --- NAVIGATION SIMULATION ---
        function getDistance(lat1, lon1, lat2, lon2) {
            const R = 6371e3;
            const φ1 = lat1 * Math.PI/180, φ2 = lat2 * Math.PI/180;
            const Δφ = (lat2-lat1) * Math.PI/180, Δλ = (lon2-lon1) * Math.PI/180;
            const a = Math.sin(Δφ/2) * Math.sin(Δφ/2) + Math.cos(φ1) * Math.cos(φ2) * Math.sin(Δλ/2) * Math.sin(Δλ/2);
            return R * (2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a))); 
        }

        function getTurnDirection(coords, idx) {
            if (coords.length < 3 || idx >= coords.length - 2) return { icon: "fa-location-dot", text: "Arriving Soon" };
            
            let lookAhead = idx + 1; let distAhead = 0;
            while (lookAhead < coords.length - 1 && distAhead < 15) {
                distAhead += getDistance(coords[lookAhead-1].lat, coords[lookAhead-1].lng, coords[lookAhead].lat, coords[lookAhead].lng); lookAhead++;
            }
            const p1 = coords[idx], p2 = coords[idx+1], p3 = coords[lookAhead]; 
            const cross = (p2.lng - p1.lng) * (p3.lat - p2.lat) - (p2.lat - p1.lat) * (p3.lng - p2.lng);
            if (Math.abs(cross) < 0.0000002) return { icon: "fa-arrow-up", text: "Follow Path Straight" };
            if (cross > 0) return { icon: "fa-arrow-turn-up-left", text: "Turn Left Ahead" };
            return { icon: "fa-arrow-turn-up-right", text: "Turn Right Ahead" };
        }

        function startNavigation() {
            if (!activeRouteCoords || activeRouteCoords.length < 2) return;
            currentStep = 0;
            
            document.getElementById('uiPanel').classList.add('collapsed');
            document.getElementById('instructionHud').style.display = 'flex';
            if(navInterval) clearInterval(navInterval);

            const speedDegrees = 0.000008; 
            map.setView([userPos.lat, userPos.lng], 20, {animate: true});

            navInterval = setInterval(() => {
                const targetCoord = activeRouteCoords[currentStep + 1];
                if (!targetCoord) { stopNavigation(); alert("Arrived at Destination!"); return; }

                const dLat = targetCoord.lat - userPos.lat, dLng = targetCoord.lng - userPos.lng;
                const distDegrees = Math.sqrt(dLat*dLat + dLng*dLng);
                const distMeters = getDistance(userPos.lat, userPos.lng, targetCoord.lat, targetCoord.lng);

                if (distDegrees < speedDegrees) { userPos.lat = targetCoord.lat; userPos.lng = targetCoord.lng; currentStep++; } 
                else { userPos.lat += (dLat/distDegrees) * speedDegrees; userPos.lng += (dLng/distDegrees) * speedDegrees; }

                userMarker.setLatLng([userPos.lat, userPos.lng]);
                map.panTo([userPos.lat, userPos.lng], {animate: true, duration: 0.1});
                
                let coveredDist = 0;
                for(let i=0; i<currentStep; i++) coveredDist += getDistance(activeRouteCoords[i].lat, activeRouteCoords[i].lng, activeRouteCoords[i+1].lat, activeRouteCoords[i+1].lng);
                coveredDist += getDistance(activeRouteCoords[currentStep].lat, activeRouteCoords[currentStep].lng, userPos.lat, userPos.lng);
                
                document.getElementById('hudProgressBar').style.width = Math.min((coveredDist / totalDistanceMeters) * 100, 100) + '%';
                const direction = getTurnDirection(activeRouteCoords, currentStep);
                document.getElementById('hudIcon').className = `fa-solid ${direction.icon} text-2xl transition-transform duration-500`;
                document.getElementById('turnLabel').innerText = direction.text;
                document.getElementById('hudMain').innerText = `Towards ${CAMPUS_GRAPH.nodes[document.getElementById('destSelect').value].name}`;
                document.getElementById('hudSub').innerText = `${Math.round(distMeters)} meters to next maneuver`;
            }, 50); 
        }

        function stopNavigation() {
            if (navInterval) clearInterval(navInterval);
            document.getElementById('instructionHud').style.display = 'none';
            document.getElementById('hudProgressBar').style.width = '0%';
            document.getElementById('uiPanel').classList.remove('collapsed');
            if(routeLayer) map.fitBounds(routeLayer.getBounds(), { paddingBottomRight: [0, window.innerHeight * 0.55], paddingTopLeft: [20, 20] });
        }

        // --- MAP CONTROLS & DIRECTORY UI ---
        function cycleMapStyle() {
            currentStyle = (currentStyle + 1) % 3;
            map.removeLayer(satelliteGroup); map.removeLayer(darkLayer); map.removeLayer(lightLayer);
            let styleName = "";
            if (currentStyle === 0) { satelliteGroup.addTo(map); styleName = "Satellite Pro Mode"; } 
            else if (currentStyle === 1) { lightLayer.addTo(map); styleName = "Clear Street Mode"; } 
            else { darkLayer.addTo(map); styleName = "Dark Navigation Mode"; }
            showToast(styleName);
        }

        function recenterMap() {
            if(userPos && map) { map.flyTo([userPos.lat, userPos.lng], 20, { animate: true, duration: 1 }); showToast("Centered on Location"); }
        }

        function showToast(msg, icon="fa-check-circle", colorClass="text-green-400") {
            const toast = document.getElementById('toastNotif'); 
            document.getElementById('toastText').innerText = msg;
            document.getElementById('toastIcon').className = `fa-solid ${icon} ${colorClass}`;
            toast.style.opacity = '1'; toast.style.transform = 'translate(-50%, 0) scale(1.05)';
            setTimeout(() => { toast.style.opacity = '0'; toast.style.transform = 'translate(-50%, -10px) scale(0.95)'; }, 2500);
        }

        function populateSelects() {
            const startSel = document.getElementById('startSelect'), destSel = document.getElementById('destSelect');
            let optionsHTML = '';
            Object.keys(CAMPUS_GRAPH.nodes).filter(k=>CAMPUS_GRAPH.nodes[k].cat!=='jct').sort((a, b) => CAMPUS_GRAPH.nodes[a].name.localeCompare(CAMPUS_GRAPH.nodes[b].name)).forEach(key => {
                optionsHTML += `<option value="${key}">${CAMPUS_GRAPH.nodes[key].name}</option>`;
            });
            startSel.innerHTML = startSel.innerHTML + optionsHTML; 
            destSel.innerHTML = optionsHTML;
        }

        function togglePanel() { document.getElementById('uiPanel').classList.toggle('collapsed'); }

        function renderMapPins() {
            pinMarkers.forEach(m => map.removeLayer(m)); pinMarkers = [];
            Object.keys(CAMPUS_GRAPH.nodes).forEach(key => {
                const node = CAMPUS_GRAPH.nodes[key];
                if(node.cat === 'jct') return;
                const customIcon = L.divIcon({
                    className: 'custom-pin', html: `<div class="custom-pin-inner bg-${getCatColorClass(node.cat)}"></div><div class="custom-pin-label">${node.name}</div>`, iconSize: [80, 40], iconAnchor: [40, 20]
                });
                const marker = L.marker([node.lat, node.lng], { icon: customIcon }).addTo(map);
                marker.bindPopup(`
                    <div class="flex flex-col">
                        <img src="${node.img}" alt="${node.name}" class="popup-img">
                        <div class="popup-text">
                            <h3 class="font-black text-[15px] text-slate-800 tracking-tight">${node.name}</h3>
                            <p class="text-xs font-semibold text-slate-500 mt-1">${node.desc}</p>
                            <button onclick="setDest('${key}')" class="mt-4 w-full bg-slate-900 text-white py-2.5 rounded-xl text-xs font-bold hover:bg-blue-600 transition-colors shadow-lg">Navigate Here</button>
                        </div>
                    </div>
                `);
                pinMarkers.push(marker);
            });
        }

        // Search and Filter Logic
        let currentFilterCat = 'all';
        function searchDir() { filterDir(currentFilterCat); }

        function filterDir(category) {
            currentFilterCat = category;
            const searchTerm = document.getElementById('searchInput').value.toLowerCase();
            document.querySelectorAll('.category-chip').forEach(c => c.classList.remove('active'));
            document.getElementById('chip-' + category).classList.add('active');
            
            const list = document.getElementById('directoryList'); list.innerHTML = '';
            
            let count = 0;
            Object.keys(CAMPUS_GRAPH.nodes).forEach(key => {
                const node = CAMPUS_GRAPH.nodes[key];
                if (node.cat === 'jct') return;
                if (category !== 'all' && node.cat !== category) return;
                if (searchTerm && !node.name.toLowerCase().includes(searchTerm) && !node.desc.toLowerCase().includes(searchTerm)) return;

                count++;
                list.innerHTML += `
                    <div class="p-3.5 bg-white border border-slate-100 rounded-2xl flex items-center gap-4 hover:border-blue-300 hover:shadow-md cursor-pointer transition-all" onclick="setDest('${key}')">
                        <div class="w-16 h-16 rounded-xl overflow-hidden shrink-0 shadow-inner relative">
                            <img src="${node.img}" class="w-full h-full object-cover" alt="${node.name}">
                            <div class="absolute inset-0 bg-black/10"></div>
                            <div class="absolute bottom-1 right-1 w-6 h-6 rounded-lg flex items-center justify-center text-[11px] text-white shadow-lg ${getCatColor(node.cat)}">
                                <i class="fa-solid ${getCatIcon(node.cat)}"></i>
                            </div>
                        </div>
                        <div class="flex-1">
                            <p class="text-[15px] font-black text-slate-800 leading-tight">${node.name}</p>
                            <p class="text-[10px] font-bold text-slate-400 mt-1 uppercase tracking-widest">${node.desc}</p>
                        </div>
                        <div class="w-9 h-9 rounded-full bg-slate-50 flex items-center justify-center border border-slate-100 shadow-sm text-blue-500">
                            <i class="fa-solid fa-diamond-turn-right text-sm"></i>
                        </div>
                    </div>`;
            });
            
            if(count === 0) {
                list.innerHTML = `<div class="text-center p-6 text-slate-400 font-medium text-sm"><i class="fa-solid fa-ghost mb-3 text-2xl"></i><br>No locations found.</div>`;
            }
        }

        function getCatIcon(cat) { return cat === 'academic' ? 'fa-graduation-cap' : cat === 'hostel' ? 'fa-bed' : cat === 'gate' ? 'fa-door-open' : cat === 'recreation' ? 'fa-mug-hot' : 'fa-location-dot'; }
        function getCatColor(cat) { return cat === 'academic' ? 'bg-blue-600' : cat === 'hostel' ? 'bg-purple-600' : cat === 'gate' ? 'bg-emerald-600' : cat === 'recreation' ? 'bg-orange-600' : 'bg-slate-600'; }
        function getCatColorClass(cat) { return cat === 'academic' ? 'blue-500' : cat === 'hostel' ? 'purple-500' : cat === 'gate' ? 'emerald-500' : cat === 'recreation' ? 'orange-500' : 'slate-500'; }

        function setDest(key) { 
            document.getElementById('destSelect').value = key; 
            initRoute(); map.closePopup();
            document.querySelector('.overflow-y-auto').scrollTo({top: 0, behavior: 'smooth'});
        }

        window.onload = initMap;
    </script>
</body>
</html>
