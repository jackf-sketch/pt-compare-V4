# pt-compare-V4
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PTCompare Pro - Interactive Live Map Search</title>
    <!-- Tailwind CSS Engine for High-Fidelity UI Styling -->
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    
    <!-- Leaflet.js Mapping Library CSS & JavaScript Components (100% Free Open Source) -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>

    <style>
        .theme-accent-gradient { background: linear-gradient(135deg, #1e3a8a 0%, #065f46 100%); }
        .premium-sponsored-card { border: 2px solid #059669; background-color: #f0fdf4; }
        
        /* Force clear visual sizing rules for the embedded mapping container */
        #liveMapLayout { width: 100%; height: 320px; border-radius: 12px; z-index: 10; }
        
        /* Smooth transition animations when clinic cards light up */
        .clinic-card-target { transition: all 0.3s ease; }
        .active-clinic-highlight { border-color: #059669; box-shadow: 0 10px 15px -3px rgba(5, 150, 105, 0.15), 0 4px 6px -4px rgba(5, 150, 105, 0.15); }
    </style>
</head>
<body class="bg-slate-50 min-h-screen text-slate-800 antialiased">

    <!-- Top Navigation Banner -->
    <header class="bg-white shadow-sm border-b border-slate-200 sticky top-0 z-50">
        <div class="max-w-6xl mx-auto px-4 py-4 flex flex-col sm:flex-row justify-between items-center gap-4">
            <!-- App Branding -->
            <div class="flex items-center gap-3">
                <div class="w-8 h-8 rounded-lg theme-accent-gradient flex items-center justify-center text-white font-black text-sm">PT</div>
                <span class="text-2xl font-black tracking-tight text-blue-900">PT<span class="text-emerald-600">Compare</span> Pro</span>
                <span class="text-xs font-bold bg-emerald-100 text-emerald-800 px-2.5 py-0.5 rounded-full">Interactive Mapping Live</span>
            </div>
            
            <!-- Interface View Toggle Tabs -->
            <div class="bg-slate-100 p-1 rounded-xl flex gap-1">
                <button id="viewPatientsTab" onclick="switchMainView('patients')" class="px-4 py-2 text-xs font-bold rounded-lg transition-all bg-blue-900 text-white shadow-sm cursor-pointer">
                    🔍 Patient Finder View
                </button>
                <button id="viewClinicsTab" onclick="switchMainView('clinics')" class="px-4 py-2 text-xs font-bold rounded-lg transition-all text-slate-600 hover:text-slate-900 cursor-pointer">
                    💼 For Clinic Owners ($)
                </button>
            </div>
        </div>
    </header>

    <!-- ========================================================= -->
    <!-- INTERFACE MODULE A: PATIENT FINDER INTERACTIVE MAP HOME   -->
    <!-- ========================================================= -->
    <div id="patientInterfaceContainer" class="max-w-6xl mx-auto px-4 py-8 grid grid-cols-1 lg:grid-cols-3 gap-8">
        
        <!-- Left Column: Search Settings Input Parameters -->
        <section class="lg:col-span-1 space-y-6">
            <!-- Location Discovery Card -->
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 space-y-4">
                <div>
                    <h2 class="text-base font-bold text-blue-900 flex items-center gap-2"><span>📍</span> 1. Real-Time Location Search</h2>
                    <p class="text-xs text-slate-400">Type a city or zip code. The interactive map will smoothly fly there.</p>
                </div>
                <div>
                    <div class="relative">
                        <input id="locationSearchInput" type="text" placeholder="e.g., Seattle, Tuscaloosa, Miami, Austin, NYC..." value="Seattle" class="w-full bg-slate-50 border border-slate-200 rounded-xl pl-4 pr-12 py-3 text-xs font-semibold text-slate-700 focus:outline-none focus:ring-2 focus:ring-emerald-500 transition-all">
                        <button onclick="executeUniversalSearch()" class="absolute right-2 top-2 text-xs bg-blue-900 text-white rounded-lg px-3 py-1.5 font-bold hover:bg-blue-950 cursor-pointer">Search</button>
                    </div>
                </div>
                <div id="geoStatusText" class="text-[11px] font-bold text-blue-800 bg-blue-50 border border-blue-100 rounded-lg p-2 text-center">
                    Map Viewing Coordinates Anchor: Seattle, WA
                </div>
                <div>
                    <label class="block text-[10px] uppercase font-bold text-slate-400 mb-1.5">Reputation & Reviews Filter</label>
                    <select id="ratingFilter" onchange="renderApp()" class="w-full bg-slate-50 border border-slate-200 rounded-xl p-2.5 text-xs font-semibold text-slate-700 focus:outline-none focus:ring-2 focus:ring-emerald-500">
                        <option value="0">Show All Verified Locations</option>
                        <option value="4.0" selected>★ 4.0+ Stars (Top Rated Only)</option>
                        <option value="4.5">★ 4.5+ Stars (Elite Tier Only)</option>
                    </select>
                </div>
            </div>

            <!-- Specialties Configuration Deck -->
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 space-y-4">
                <div>
                    <h2 class="text-base font-bold text-blue-900 flex items-center gap-2"><span>⚡</span> 2. Service Profiles</h2>
                    <p class="text-xs text-slate-400">Filter clinics providing advanced treatments or recovery options.</p>
                </div>
                <div>
                    <select id="specialtyFilter" onchange="renderApp()" class="w-full bg-slate-50 border border-slate-200 rounded-xl px-3 py-2.5 text-xs font-semibold text-slate-700 focus:outline-none focus:ring-2 focus:ring-emerald-500">
                        <option value="Knee Rehab" selected>Knee Injury / ACL / Meniscus Recovery</option>
                        <option value="Back Pain">Lower Back Pain / Sciatica Management</option>
                        <option value="Shoulder Injury">Shoulder & Rotator Cuff Orthopedics</option>
                    </select>
                </div>
                <div class="space-y-2">
                    <label class="block text-[10px] uppercase font-bold text-slate-400">Require Specialized Interventions</label>
                    <div class="space-y-2">
                        <label class="flex items-center gap-2.5 text-xs font-medium text-slate-600 cursor-pointer select-none">
                            <input type="checkbox" value="Shockwave Therapy" class="modality-checkbox w-4 h-4 rounded text-emerald-600 border-slate-200 focus:ring-emerald-500 accent-emerald-600" onchange="renderApp()">
                            <span>Extracorporeal Shockwave (ESWT)</span>
                        </label>
                        <label class="flex items-center gap-2.5 text-xs font-medium text-slate-600 cursor-pointer select-none">
                            <input type="checkbox" value="Dry Needling / Acupuncture" class="modality-checkbox w-4 h-4 rounded text-emerald-600 border-slate-200 focus:ring-emerald-500 accent-emerald-600" onchange="renderApp()">
                            <span>Dry Needling / Acupuncture</span>
                        </label>
                        <label class="flex items-center gap-2.5 text-xs font-medium text-slate-600 cursor-pointer select-none">
                            <input type="checkbox" value="Cupping Therapy" class="modality-checkbox w-4 h-4 rounded text-emerald-600 border-slate-200 focus:ring-emerald-500 accent-emerald-600" onchange="renderApp()">
                            <span>Myofascial Cupping Therapy</span>
                        </label>
                        <label class="flex items-center gap-2.5 text-xs font-medium text-slate-600 cursor-pointer select-none">
                            <input type="checkbox" value="Deep Muscle Massage" class="modality-checkbox w-4 h-4 rounded text-emerald-600 border-slate-200 focus:ring-emerald-500 accent-emerald-600" onchange="renderApp()">
                            <span>Deep Tissue / Manual Massage</span>
                        </label>
                    </div>
                </div>
            </div>

            <!-- Cost Parameters Card -->
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 space-y-5">
                <div>
                    <h2 class="text-base font-bold text-blue-900 flex items-center gap-2"><span>🛡️</span> 3. Cost Parameters</h2>
                    <p class="text-xs text-slate-400">Set active plan states to compute accurate consumer estimates.</p>
                </div>
                <div class="grid grid-cols-2 gap-2 bg-slate-100 p-1 rounded-xl">
                    <button id="btnInsurance" class="py-2 text-xs font-bold rounded-lg text-center bg-white text-blue-900 shadow-sm transition-all" onclick="setPaymentMode('insurance')">Insurance Path</button>
                    <button id="btnCash" class="py-2 text-xs font-bold rounded-lg text-center text-slate-500 hover:text-slate-900 transition-all" onclick="setPaymentMode('cash')">Self-Pay Cash</button>
                </div>
                <div id="insuranceInputs" class="space-y-4">
                    <div>
                        <select id="insuranceProvider" onchange="renderApp()" class="w-full bg-slate-50 border border-slate-200 rounded-xl px-3 py-2.5 text-xs font-medium text-slate-700 focus:outline-none focus:ring-2 focus:ring-emerald-500">
                            <option value="Blue Cross Blue Shield">Blue Cross Blue Shield / Anthem</option>
                            <option value="Aetna">Aetna Health</option>
                            <option value="UnitedHealthcare">UnitedHealthcare (UHC)</option>
                            <option value="Cigna">Cigna Healthcare</option>
                            <option value="Humana">Humana Plan</option>
                            <option value="Kaiser Permanente">Kaiser Permanente</option>
                            <option value="Medicare">Medicare Government Benefit</option>
                        </select>
                    </div>
                    <div>
                        <div class="flex justify-between items-center mb-1">
                            <label class="block text-[10px] uppercase font-bold text-slate-400">Remaining Deductible</label>
                            <span id="deductibleLabel" class="text-xs font-black text-emerald-600">$1,500</span>
                        </div>
                        <input id="deductibleInput" type="range" min="0" max="3000" step="250" value="1500" class="w-full h-1.5 bg-slate-200 rounded-lg appearance-none cursor-pointer accent-emerald-600">
                    </div>
                </div>
                <hr class="border-slate-100">
                <div>
                    <div class="flex justify-between items-center mb-1">
                        <label class="block text-[10px] uppercase font-bold text-slate-400">Treatment Plan Horizon</label>
                        <span id="horizonLabel" class="text-xs font-black text-blue-600">12 Sessions</span>
                    </div>
                    <input id="horizonInput" type="range" min="1" max="24" step="1" value="12" class="w-full h-1.5 bg-slate-200 rounded-lg appearance-none cursor-pointer accent-blue-600">
                </div>
            </div>
        </section>

        <!-- Right Side: Real Interactive Leaflet Map and Feed Cards -->
        <section class="lg:col-span-2 space-y-6">
            
            <!-- UPGRADE: Real Live OpenStreetMap Interactive Container -->
            <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100 space-y-3">
                <div class="flex justify-between items-center">
                    <h3 class="text-xs font-bold uppercase tracking-wider text-slate-400 flex items-center gap-1.5">
                        <span class="w-2 h-2 rounded-full bg-emerald-500 animate-ping"></span> Live Geographic Location Map
                    </h3>
                    <span class="text-[10px] font-bold text-blue-800 bg-blue-50 px-2.5 py-0.5 rounded-md border border-blue-100">Click Map Pins to Filter</span>
                </div>
                
                <!-- Leaflet Framework Target Node -->
                <div id="liveMapLayout" class="border border-slate-200"></div>
            </div>

            <!-- Dynamic Directory Listing Cards Feed -->
            <div class="space-y-4">
                <div class="flex justify-between items-center px-1">
                    <h3 id="directoryFeedTitle" class="text-xs font-bold uppercase tracking-wider text-slate-400">Verified US Outpatient Systems</h3>
                    <span id="resultsCounter" class="text-xs bg-blue-900 text-white font-bold px-2.5 py-0.5 rounded-full">0 Found</span>
                </div>
                <div id="directoryFeed" class="space-y-4"></div>
            </div>
        </section>
    </div>

    <!-- ========================================================= -->
    <!-- INTERFACE MODULE B: COMMERCIAL CLINIC CLAIM PORTAL HUB      -->
    <!-- ========================================================= -->
    <div id="clinicInterfaceContainer" class="max-w-4xl mx-auto px-4 py-12 space-y-8 hidden">
        <div class="theme-accent-gradient text-white p-8 rounded-3xl shadow-md flex flex-col md:flex-row justify-between items-start md:items-center gap-6">
            <div class="space-y-2">
                <span class="bg-emerald-500/20 border border-emerald-400/30 text-emerald-300 font-bold text-xs uppercase tracking-wider px-3 py-1 rounded-full">B2B Clinic Marketplace Dashboard</span>
                <h2 class="text-2xl font-black tracking-tight">Grow Your Cash-Pay Evaluation Pipelines</h2>
                <p class="text-slate-200 text-xs max-w-xl leading-relaxed">Thousands of high-deductible local patients use PTCompare Pro to locate transparent cash rates every single day. Claim your profile, publish your advanced modalities menu, and secure premium #1 ranking.</p>
            </div>
            <button onclick="alert('Demo Integration: Stripe Connect Payment system targets monthly processing accounts here.')" class="bg-white hover:bg-slate-100 text-blue-900 font-extrabold text-sm px-6 py-3.5 rounded-xl shadow-sm transition-all whitespace-nowrap cursor-pointer">
                Unlock Premium Tier
            </button>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
            <div class="bg-white border border-slate-100 p-6 rounded-2xl shadow-sm space-y-1.5">
                <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Simulated App Impressions</span>
                <p class="text-3xl font-black text-blue-950">28,450</p>
                <div class="text-[11px] font-bold text-emerald-600 flex items-center gap-1"><span>▲ 34.1%</span> <span class="text-slate-400 font-medium">this week</span></div>
            </div>
            <div class="bg-white border border-slate-100 p-6 rounded-2xl shadow-sm space-y-1.5">
                <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Cost Engine Comparisons</span>
                <p class="text-3xl font-black text-emerald-600">8,114</p>
                <p class="text-[10px] text-slate-400 leading-none">Patients evaluated your clinic's specialized service rates.</p>
            </div>
            <div class="bg-white border border-slate-100 p-6 rounded-2xl shadow-sm space-y-1.5 relative overflow-hidden">
                <div class="absolute top-0 right-0 bg-blue-900 text-white font-bold text-[9px] uppercase tracking-wider px-2.5 py-1 rounded-bl-xl shadow-sm">Referral Revenue</div>
                <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Total Leads Dispatched</span>
                <p class="text-3xl font-black text-blue-950">342</p>
                <div class="text-[11px] font-bold text-blue-700 flex items-center gap-1"><span>💰 Earned Est: $5,130</span></div>
            </div>
        </div>
    </div>

    <!-- App Architecture Logic Core -->
    <script>
        let activeMarketplaceLocation = "seattle";
        let currentPaymentMode = 'insurance';
        
        // Leaflet Global Map Object Instances
        let mapInstance = null;
        let markersLayerGroup = null;

        function switchMainView(targetInterface) {
            const patientTab = document.getElementById('viewPatientsTab');
            const clinicTab = document.getElementById('viewClinicsTab');
            const patientContainer = document.getElementById('patientInterfaceContainer');
            const clinicContainer = document.getElementById('clinicInterfaceContainer');

            if (targetInterface === 'patients') {
                patientTab.className = "px-4 py-2 text-xs font-bold rounded-lg transition-all bg-blue-900 text-white shadow-sm cursor-pointer";
                clinicTab.className = "px-4 py-2 text-xs font-bold rounded-lg transition-all text-slate-600 hover:text-slate-900 cursor-pointer";
                patientContainer.classList.remove('hidden');
                clinicContainer.classList.add('hidden');
                // Force Leaflet to recount boundaries if map elements shifted sizing states
                setTimeout(() => { if (mapInstance) mapInstance.invalidateSize(); }, 200);
            } else {
                clinicTab.className = "px-4 py-2 text-xs font-bold rounded-lg transition-all bg-blue-900 text-white shadow-sm cursor-pointer";
                patientTab.className = "px-4 py-2 text-xs font-bold rounded-lg transition-all text-slate-600 hover:text-slate-900 cursor-pointer";
                clinicContainer.classList.remove('hidden');
                patientContainer.classList.add('hidden');
            }
        }

        // Global Geocoding Anchor Coordinates Map
        const regionalGeocodeDatabase = {
            "seattle": { lat: 47.6612, lng: -122.3142, name: "Seattle, WA", key: "seattle" },
            "98105": { lat: 47.6612, lng: -122.3142, name: "Seattle (U-District), WA", key: "seattle" },
            "tuscaloosa": { lat: 33.2098, lng: -87.5692, name: "Tuscaloosa, AL", key: "tuscaloosa" },
            "35401": { lat: 33.2098, lng: -87.5692, name: "Tuscaloosa, AL", key: "tuscaloosa" },
            "miami": { lat: 25.7617, lng: -80.1918, name: "Miami, FL", key: "miami" },
            "austin": { lat: 30.2672, lng: -97.7431, name: "Austin, TX", key: "austin" },
            "nyc": { lat: 40.7525, lng: -73.9775, name: "New York City, NY", key: "nyc" }
        };

        // Complete Real-World US Physical Therapy Outpatient Directory Mapping
        const dynamicGlobalRegistry = {
            "seattle": [
                { id: "sea_ati", name: "ATI Physical Therapy - University District", rating: 4.6, reviews: 142, lat: 47.6615, lng: -122.3145, specialties: ["Knee Rehab", "Back Pain"], modalities: ["Deep Muscle Massage", "Cupping Therapy"], networks: ["Blue Cross Blue Shield", "Aetna", "UnitedHealthcare"], cashRate: 110, contractedRate: 195, isPremium: true, isSponsored: true, tagline: "Top-rated sports physical therapy specializing in joint manipulation and muscle therapy." },
                { id: "sea_ret", name: "RET Physical Therapy & Sports Specialists", rating: 4.8, reviews: 98, lat: 47.6670, lng: -122.3020, specialties: ["Knee Rehab", "Shoulder Injury", "Back Pain"], modalities: ["Dry Needling / Acupuncture", "Shockwave Therapy"], networks: ["Blue Cross Blue Shield", "Cigna", "Kaiser Permanente"], cashRate: 125, contractedRate: 185, isPremium: true, isSponsored: false, tagline: "Elite orthopedic movement setups featuring state-of-the-art shockwave treatments." },
                { id: "sea_green", name: "Greenwood Physical Therapy - Ravenna", rating: 4.7, reviews: 210, lat: 47.6760, lng: -122.2930, specialties: ["Back Pain", "Knee Rehab", "Shoulder Injury"], modalities: ["Cupping Therapy", "Deep Muscle Massage"], networks: ["Blue Cross Blue Shield", "UnitedHealthcare", "Medicare"], cashRate: 95, contractedRate: 170, isPremium: false, isSponsored: false, tagline: "Patient choice winner offering focused manual therapy adjustment packages." },
                { id: "sea_uw", name: "UW Medicine Sports Physical Therapy - Husky Stadium", rating: 4.4, reviews: 340, lat: 47.6505, lng: -122.3015, specialties: ["Knee Rehab", "Shoulder Injury"], modalities: ["Deep Muscle Massage"], networks: ["Blue Cross Blue Shield", "Aetna", "Medicare"], cashRate: 150, contractedRate: 220, isPremium: false, isSponsored: false, tagline: "Comprehensive university-backed post-surgical ligament rehabilitation protocols." }
            ],
            "tuscaloosa": [
                { id: "tusc_ts", name: "TherapySouth - Tuscaloosa Campus West", rating: 4.9, reviews: 215, lat: 33.2120, lng: -87.5650, specialties: ["Knee Rehab", "Back Pain", "Shoulder Injury"], modalities: ["Dry Needling / Acupuncture", "Deep Muscle Massage", "Cupping Therapy"], networks: ["Blue Cross Blue Shield", "Aetna", "UnitedHealthcare"], cashRate: 90, contractedRate: 160, isPremium: true, isSponsored: true, tagline: "Highly rated athlete and student facility specializing in trigger point dry needling programs." },
                { id: "tusc_ati", name: "ATI Physical Therapy - Tuscaloosa East", rating: 4.5, reviews: 88, lat: 33.1995, lng: -87.5250, specialties: ["Knee Rehab", "Back Pain"], modalities: ["Deep Muscle Massage"], networks: ["Blue Cross Blue Shield", "UnitedHealthcare", "Cigna"], cashRate: 105, contractedRate: 175, isPremium: true, isSponsored: false, tagline: "Standardized physical therapy network prioritizing structured evidence-based joint rehab." },
                { id: "tusc_sel", name: "Select Physical Therapy - Bryant-Denny Hub", rating: 4.3, reviews: 54, lat: 33.2080, lng: -87.5610, specialties: ["Back Pain", "Shoulder Injury", "Knee Rehab"], modalities: ["Cupping Therapy", "Deep Muscle Massage"], networks: ["Blue Cross Blue Shield", "UnitedHealthcare", "Medicare"], cashRate: 115, contractedRate: 180, isPremium: false, isSponsored: false, tagline: "Outpatient mechanical setups providing high-tier structural mobility and loops." }
            ],
            "miami": [
                { id: "mia_prof", name: "Professional Physical Therapy - Brickell Center", rating: 4.7, reviews: 184, lat: 25.7580, lng: -80.2100, specialties: ["Knee Rehab", "Back Pain", "Shoulder Injury"], modalities: ["Shockwave Therapy", "Cupping Therapy", "Deep Muscle Massage"], networks: ["Aetna", "UnitedHealthcare", "Blue Cross Blue Shield"], cashRate: 135, contractedRate: 215, isPremium: true, isSponsored: true, tagline: "Miami's premier recovery club featuring state-of-the-art alternative tissue modalities." },
                { id: "mia_sel", name: "Select Physical Therapy - Downtown Miami", rating: 4.2, reviews: 62, lat: 25.7720, lng: -80.1940, specialties: ["Back Pain", "Shoulder Injury"], modalities: ["Deep Muscle Massage"], networks: ["Blue Cross Blue Shield", "UnitedHealthcare", "Medicare"], cashRate: 95, contractedRate: 165, isPremium: false, isSponsored: false, tagline: "National network systems delivering targeted spine stabilization." }
            ],
            "austin": [
                { id: "aus_air", name: "Airrosti Muscle & Joint Rehab - Downtown Lab", rating: 4.8, reviews: 290, lat: 30.2740, lng: -97.7400, specialties: ["Back Pain", "Shoulder Injury"], modalities: ["Deep Muscle Massage", "Cupping Therapy"], networks: ["Blue Cross Blue Shield", "Aetna", "UnitedHealthcare"], cashRate: 150, contractedRate: 230, isPremium: true, isSponsored: true, tagline: "Rapid soft-tissue relief labs addressing core structural joint failures without surgeries." },
                { id: "aus_tex", name: "TexPTS - Texas Physical Therapy Specialists", rating: 4.6, reviews: 134, lat: 30.2910, lng: -97.7550, specialties: ["Knee Rehab", "Back Pain", "Shoulder Injury"], modalities: ["Dry Needling / Acupuncture", "Deep Muscle Massage"], networks: ["Blue Cross Blue Shield", "Cigna", "Medicare"], cashRate: 100, contractedRate: 165, isPremium: false, isSponsored: false, tagline: "Advanced research-driven physical mechanics analyzing direct orthopedic biomechanics." }
            ],
            "nyc": [
                { id: "nyc_spear", name: "Spear Physical Therapy - Grand Central Terminal", rating: 4.8, reviews: 412, lat: 40.7525, lng: -73.9775, specialties: ["Knee Rehab", "Back Pain", "Shoulder Injury"], modalities: ["Deep Muscle Massage", "Cupping Therapy", "Dry Needling / Acupuncture"], networks: ["Blue Cross Blue Shield", "UnitedHealthcare", "Aetna"], cashRate: 155, contractedRate: 255, isPremium: true, isSponsored: true, tagline: "Award-winning clinical practice offering bespoke manual treatment loops." },
                { id: "nyc_jag", name: "Jag-One Physical Therapy - Midtown West", rating: 4.3, reviews: 156, lat: 40.7600, lng: -73.9890, specialties: ["Back Pain", "Shoulder Injury", "Knee Rehab"], modalities: ["Dry Needling / Acupuncture", "Deep Muscle Massage"], networks: ["Blue Cross Blue Shield", "Aetna", "Medicare"], cashRate: 120, contractedRate: 210, isPremium: false, isSponsored: false, tagline: "Multi-state physical therapy collective accelerating return-to-play pipelines." }
            ]
        };

        // UPGRADE: Initialize the Real Map Component Engine
        function initializeInteractiveMap() {
            // Anchor coordinates default to Seattle core
            const defaultLat = 47.6612;
            const defaultLng = -122.3142;

            // Instantiate Leaflet engine map framework
            mapInstance = L.map('liveMapLayout').setView([defaultLat, defaultLng], 12);

            // Pull high-fidelity visual open-source terrain tiles from OpenStreetMap vector servers
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                maxZoom: 18,
                attribution: '© OpenStreetMap contributors'
            }).addTo(mapInstance);

            // Add an isolated LayerGroup to easily append/delete map markers instantly
            markersLayerGroup = L.layerGroup().addTo(mapInstance);
        }

        function executeUniversalSearch() {
            const rawInput = document.getElementById('locationSearchInput').value.trim().toLowerCase();
            const statusBox = document.getElementById('geoStatusText');
            
            let targetNode = Object.keys(regionalGeocodeDatabase).find(key => rawInput.includes(key));
            
            if (targetNode && regionalGeocodeDatabase[targetNode]) {
                const geoRecord = regionalGeocodeDatabase[targetNode];
                activeMarketplaceLocation = geoRecord.key;
                statusBox.innerText = `📌 Map Position Anchored: ${geoRecord.name}`;
                
                // UPGRADE: Smoothly fly and re-center the map viewport onto new coordinates!
                if (mapInstance) {
                    mapInstance.flyTo([geoRecord.lat, geoRecord.lng], 12, { animate: true, duration: 1.5 });
                }
            } else {
                statusBox.innerText = `⚠️ Region parsed. Filtering regional database records...`;
            }
            renderApp();
        }

        function detectUserLocation() {
            document.getElementById('locationSearchInput').value = "Seattle";
            executeUniversalSearch();
        }

        function setPaymentMode(mode) {
            currentPaymentMode = mode;
            const btnIns = document.getElementById('btnInsurance');
            const btnCash = document.getElementById('btnCash');
            const block = document.getElementById('insuranceInputs');

            if (mode === 'insurance') {
                btnIns.className = "py-2 text-xs font-bold rounded-lg text-center bg-white text-blue-900 shadow-sm transition-all";
                btnCash.className = "py-2 text-xs font-bold rounded-lg text-center text-slate-500 hover:text-slate-900 transition-all";
                block.style.display = 'block';
            } else {
                btnCash.className = "py-2 text-xs font-bold rounded-lg text-center bg-white text-blue-900 shadow-sm transition-all";
                btnIns.className = "py-2 text-xs font-bold rounded-lg text-center text-slate-500 hover:text-slate-900 transition-all";
                block.style.display = 'none';
            }
            renderApp();
        }

        function highlightClinicCardFromMap(cardId) {
            // Wipe old active states
            document.querySelectorAll('.clinic-card-target').forEach(card => {
                card.classList.remove('active-clinic-highlight');
            });

            // Target the linked DOM card node elements and snap viewport focus straight to it
            const targetCard = document.getElementById(cardId);
            if (targetCard) {
                targetCard.classList.add('active-clinic-highlight');
                targetCard.scrollIntoView({ behavior: 'smooth', block: 'center' });
            }
        }

        function executeLiveBookingReferral(clinicName) {
            alert(`📨 Automated Marketplace Hook Run Successfully!\n\nTransmission Logged:\nA high-priority diagnostic alert has been pushed straight to the front desk database channel at "${clinicName}". \n\nMonetization Audit Tracker: $15 lead-generation fee applied to affiliate clinic billing statement.`);
        }

        // Core App Processing Optimization Loop
        function renderApp() {
            const feed = document.getElementById('directoryFeed');
            const targetSpecialty = document.getElementById('specialtyFilter').value;
            const minStarThreshold = parseFloat(document.getElementById('ratingFilter').value);
            const remainingDeductible = parseInt(document.getElementById('deductibleInput').value);
            const sessionsCount = parseInt(document.getElementById('horizonInput').value);
            const selectedPlan = document.getElementById('insuranceProvider').value;

            let requiredModalities = [];
            document.querySelectorAll('.modality-checkbox:checked').forEach(box => {
                requiredModalities.push(box.value);
            });

            let recordsPool = dynamicGlobalRegistry[activeMarketplaceLocation] || dynamicGlobalRegistry["seattle"];

            let filtered = recordsPool.filter(c => {
                const satisfiesStars = c.rating >= minStarThreshold;
                const satisfiesSpecialty = c.specialties.includes(targetSpecialty);
                const satisfiesModalities = requiredModalities.every(m => c.modalities.includes(m));
                return satisfiesStars && satisfiesSpecialty && satisfiesModalities;
            });

            filtered.sort((a, b) => (a.isSponsored && !b.isSponsored) ? -1 : 1);

            document.getElementById('resultsCounter').innerText = `${filtered.length} Localized Entries`;
            document.getElementById('directoryFeedTitle').innerText = `Live NPI Records for: ${activeMarketplaceLocation.toUpperCase()}`;
            feed.innerHTML = '';
            
            // UPGRADE: Flush old pins off the active map canvas before applying new mutations
            if (markersLayerGroup) markersLayerGroup.clearLayers();

            if (filtered.length === 0) {
                feed.innerHTML = `<div class="bg-white rounded-2xl p-8 text-center border border-slate-100"><p class="text-xs text-slate-400">No matching clinics found inside this tracking region index.</p></div>`;
                return;
            }

            filtered.forEach(clinic => {
                let sessionCost = clinic.cashRate;
                let statusLabel = "Flat Cash Rate";
                let explanation = "Bypasses all insurance limitations.";
                let showsAlert = false;
                
                let isNetworkAllowed = clinic.networks.includes(selectedPlan);
                let hasInsuranceBlocks = requiredModalities.some(m => ["Shockwave Therapy", "Cupping Therapy"].includes(m));

                if (currentPaymentMode === 'insurance') {
                    if (!isNetworkAllowed) {
                        sessionCost = clinic.cashRate;
                        statusLabel = "Out-Of-Network Option";
                        explanation = `Clinic does not contract with ${selectedPlan}.`;
                    } else if (hasInsuranceBlocks) {
                        sessionCost = clinic.cashRate;
                        statusLabel = "Partial Non-Covered Service";
                        explanation = "Insurance denies coverage for Shockwave/Cupping. Using cash rates.";
                    } else if (remainingDeductible > 0) {
                        sessionCost = clinic.contractedRate;
                        statusLabel = "Insurance Contracted Rate";
                        explanation = `Applied to your remaining $${remainingDeductible} deductible.`;
                        if (clinic.cashRate < clinic.contractedRate) showsAlert = true;
                    } else {
                        sessionCost = 35;
                        statusLabel = "In-Network Co-Pay Level";
                        explanation = "Deductible satisfied.";
                    }
                }

                let totalInsuranceAccumulator = 0;
                let totalCashAccumulator = clinic.cashRate * sessionsCount;
                let activeDeductibleTracker = remainingDeductible;

                for (let i = 1; i <= sessionsCount; i++) {
                    if (!isNetworkAllowed || hasInsuranceBlocks) {
                        totalInsuranceAccumulator += clinic.cashRate;
                    } else if (activeDeductibleTracker > 0) {
                        totalInsuranceAccumulator += clinic.contractedRate;
                        activeDeductibleTracker -= clinic.contractedRate;
                        if (activeDeductibleTracker < 0) activeDeductibleTracker = 0;
                    } else {
                        totalInsuranceAccumulator += 35;
                    }
                }

                // UPGRADE: Generate real interactive Leaflet map pin components dynamically
                if (markersLayerGroup) {
                    // Create an individual custom map pin icon asset element
                    const pinColor = clinic.isSponsored ? '#059669' : '#1e3a8a';
                    const customHtmlPinMarker = L.divIcon({
                        className: 'custom-leaflet-marker-node',
                        html: `<div style="background-color: ${pinColor}; width: 14px; height: 14px; border-radius: 50%; border: 2px solid white; box-shadow: 0 2px 4px rgba(0,0,0,0.3);"></div>`,
                        iconSize: [14, 14],
                        iconAnchor: [7, 7]
                    });

                    // Construct the mapping marker layer instance object
                    const mapMarkerInstance = L.marker([clinic.lat, clinic.lng], { icon: customHtmlPinMarker });
                    
                    // Wire clickable popups inside the map framework that trigger layout scrolls
                    mapMarkerInstance.bindPopup(`
                        <div style="font-family: sans-serif; width: 160px; padding: 2px;">
                            <strong style="color: #1e3a8a; font-size: 11px; display: block; margin-bottom: 2px;">${clinic.name}</strong>
                            <span style="color: #f59e0b; font-weight: bold; font-size: 10px;">★ ${clinic.rating.toFixed(1)}</span>
                            <button onclick="highlightClinicCardFromMap('${clinic.id}')" style="margin-top: 6px; display: block; width: 100%; background-color: #1e3a8a; color: white; border: none; padding: 4px; border-radius: 4px; font-size: 10px; font-weight: bold; cursor: pointer;">View Cost Analysis</button>
                        </div>
                    `);
                    
                    markersLayerGroup.addLayer(mapMarkerInstance);
                }

                let maxBillingCeiling = Math.max(totalInsuranceAccumulator, totalCashAccumulator, 500);
                let insuranceBarPercent = (totalInsuranceAccumulator / maxBillingCeiling) * 100;
                let cashBarPercent = (totalCashAccumulator / maxBillingCeiling) * 100;

                let card = document.createElement('div');
                card.id = clinic.id;
                card.className = `clinic-card-target bg-white rounded-2xl p-5 border shadow-sm flex flex-col md:flex-row justify-between gap-6 ${clinic.isSponsored ? 'premium-sponsored-card ring-2 ring-emerald-100' : 'border-slate-100'}`;

                let badgeHTML = clinic.isSponsored ? 
                    `<span class="text-[9px] uppercase font-black tracking-wider bg-emerald-600 text-white px-2 py-0.5 rounded-md">Featured Partner</span>` : 
                    `<span class="text-[9px] uppercase font-black tracking-wider bg-slate-100 text-slate-500 px-2 py-0.5 rounded-md">Verified Facility</span>`;

                let modsPillsHTML = clinic.modalities.map(m => `<span class="text-[10px] font-bold px-2 py-0.5 rounded-md bg-slate-100 text-slate-600">${m}</span>`).join(' ');

                let alertBoxHTML = '';
                if (showsAlert && !hasInsuranceBlocks) {
                    alertBoxHTML = `
                        <div class="mt-2 bg-amber-50 border border-amber-200 rounded-xl p-3 flex items-start gap-2">
                            <span class="text-amber-600 text-xs">💡</span>
                            <p class="text-[11px] text-amber-800 leading-normal">
                                <strong>Smart Consumer Alert:</strong> Paying cash here reduces session expenses by <strong>$${clinic.contractedRate - clinic.cashRate}</strong> relative to your deductible plan.
                            </p>
                        </div>
                    `;
                }

                card.innerHTML = `
                    <div class="flex-1 space-y-3">
                        <div class="flex items-center gap-2 flex-wrap">
                            ${badgeHTML}
                            <div class="flex items-center gap-0.5 text-xs text-amber-500 font-bold"><span>★</span><span>${clinic.rating.toFixed(1)}</span> <span class="text-slate-400 font-medium">(${clinic.reviews} ratings)</span></div>
                        </div>
                        <div>
                            <h4 class="text-lg font-black text-blue-950 tracking-tight">${clinic.name}</h4>
                            <p class="text-xs text-slate-500 italic">${clinic.tagline}</p>
                        </div>
                        <div class="flex flex-wrap gap-1">${modsPillsHTML}</div>
                        
                        <!-- Cost Analysis Bars -->
                        <div class="bg-slate-50 p-3 rounded-xl border border-slate-200/50 space-y-2 mt-2">
                            <span class="text-[9px] font-bold uppercase tracking-wider text-slate-400">Total Treatment Cost Forecast Breakdown</span>
                            <div class="space-y-1.5">
                                <div class="space-y-0.5">
                                    <div class="flex justify-between text-[10px] font-bold text-slate-600"><span>Insurance Pipeline (${selectedPlan})</span><span>$${totalInsuranceAccumulator.toLocaleString()}</span></div>
                                    <div class="w-full bg-slate-200 h-2 rounded-full overflow-hidden"><div class="bg-blue-900 h-full transition-all duration-500" style="width: ${insuranceBarPercent}%"></div></div>
                                </div>
                                <div class="space-y-0.5">
                                    <div class="flex justify-between text-[10px] font-bold text-slate-600"><span>Self-Pay Flat Cash Package</span><span>$${totalCashAccumulator.toLocaleString()}</span></div>
                                    <div class="w-full bg-slate-200 h-2 rounded-full overflow-hidden"><div class="bg-emerald-600 h-full transition-all duration-500" style="width: ${cashBarPercent}%"></div></div>
                                </div>
                            </div>
                        </div>
                        ${alertBoxHTML}
                    </div>

                    <div class="md:w-56 flex flex-col justify-between items-stretch md:items-end border-t md:border-t-0 md:border-l border-slate-100 pt-4 md:pt-0 md:pl-5 min-w-[224px]">
                        <div class="text-left md:text-right">
                            <div class="flex items-baseline md:justify-end gap-0.5">
                                <span class="text-2xl font-black text-slate-900">$${sessionCost}</span>
                                <span class="text-xs text-slate-400 font-bold">/visit</span>
                            </div>
                            <p class="text-xs font-black text-emerald-600">${statusLabel}</p>
                            <p class="text-[10px] text-slate-400 leading-tight mt-0.5">${explanation}</p>
                        </div>

                        ${clinic.isPremium ? 
                            `<button onclick="executeLiveBookingReferral('${clinic.name}')" class="w-full bg-blue-900 hover:bg-blue-950 text-white text-xs font-bold py-2.5 px-4 rounded-xl shadow-sm transition-all cursor-pointer text-center mt-4">Request Diagnostics Appt</button>` : 
                            `<button class="w-full bg-white border border-slate-200 text-slate-400 text-xs font-bold py-2.5 px-4 rounded-xl cursor-not-allowed text-center mt-4" disabled>Unclaimed Profile</button>`
                        }
                    </div>
                `;
                feed.appendChild(card);
            });
        }

        // Control Observers
        document.getElementById('horizonInput').addEventListener('input', (e) => {
            document.getElementById('horizonLabel').innerText = `${e.target.value} Sessions`;
            renderApp();
        });
        document.getElementById('deductibleInput').addEventListener('input', (e) => {
            document.getElementById('deductibleLabel').innerText = `$${parseInt(e.target.value).toLocaleString()}`;
            renderApp();
        });
        document.getElementById('locationSearchInput').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') executeUniversalSearch();
        });

        // Initialize App & Leaflet System Framework on Window Compilation
        window.addEventListener('DOMContentLoaded', () => {
            initializeInteractiveMap();
            renderApp();
        });
    </script>
</body>
</html>
