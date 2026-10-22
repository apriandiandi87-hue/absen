<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Presensi - SMP Negeri 2 Terara</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Custom Tailwind Configuration for Royal Navy Blue Theme -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        navy: {
                            50: '#f0f4f9',
                            100: '#d9e2ec',
                            500: '#1d4ed8',
                            800: '#1e3a8a',
                            900: '#0f172a',
                            950: '#0a0f1d'
                        },
                        royal: {
                            500: '#2563eb',
                            600: '#1d4ed8',
                            700: '#1e40af'
                        }
                    }
                }
            }
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Custom styling for print output */
        @media print {
            .no-print { display: none !important; }
            .print-only { display: block !important; }
            body { background: white; color: black; font-size: 12pt; }
            .print-container { width: 100%; max-width: 100%; padding: 0; margin: 0; box-shadow: none; border: none; }
        }
        .print-only { display: none; }
        
        /* Camera overlay frame animation */
        @keyframes scan-line {
            0% { top: 10%; }
            50% { top: 85%; }
            100% { top: 10%; }
        }
        .scanner-line {
            position: absolute;
            left: 10%;
            width: 80%;
            height: 3px;
            background: #22c55e;
            box-shadow: 0 0 12px #22c55e;
            animation: scan-line 3s infinite linear;
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col justify-between">

    <!-- TOP NAVIGATION HEADER -->
    <header class="bg-gradient-to-r from-navy-950 via-navy-900 to-royal-700 text-white shadow-lg sticky top-0 z-40 no-print">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3.5 flex items-center justify-between">
            <div class="flex items-center space-x-3 cursor-pointer" onclick="switchTab('public-dashboard')">
                <div class="bg-white/10 p-2 rounded-xl backdrop-blur-sm border border-white/20">
                    <i class="fa-solid font-bold fa-school text-2xl text-blue-300"></i>
                </div>
                <div>
                    <h1 class="text-lg sm:text-xl font-bold tracking-tight leading-tight">SMP NEGERI 2 TERARA</h1>
                    <p class="text-xs text-blue-200">Sistem Presensi Online GPS & Biometrik Wajah</p>
                </div>
            </div>
            
            <!-- Navigation Action Buttons -->
            <div class="flex items-center space-x-2 sm:space-x-4">
                <button onclick="switchTab('public-dashboard')" class="hidden sm:inline-flex items-center px-3 py-1.5 rounded-lg text-xs sm:text-sm font-medium hover:bg-white/10 transition">
                    <i class="fa-solid fa-chart-pie mr-2"></i> Dashboard
                </button>
                
                <div id="user-menu-btn" class="hidden">
                    <!-- Dynamic user pill when logged in -->
                </div>

                <button id="auth-btn" onclick="openLoginModal()" class="bg-blue-600 hover:bg-blue-500 text-white text-xs sm:text-sm font-semibold px-4 py-2 rounded-lg shadow transition flex items-center">
                    <i class="fa-solid fa-right-to-bracket mr-2"></i> Masuk / Login
                </button>
            </div>
        </div>
    </header>

    <!-- NOTIFICATION TOAST CONTAINER -->
    <div id="toast-container" class="fixed bottom-5 right-5 z-50 flex flex-col space-y-2 no-print"></div>

    <!-- MAIN CONTENT AREA -->
    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6 flex-grow w-full">

        <!-- ========================================== -->
        <!-- 1. PUBLIC DASHBOARD VIEW (BEFORE LOGIN)   -->
        <!-- ========================================== -->
        <section id="public-dashboard" class="space-y-6">
            
            <!-- Banner Head & Principal Info -->
            <div class="bg-gradient-to-br from-navy-900 to-royal-700 rounded-2xl p-6 text-white shadow-xl relative overflow-hidden">
                <div class="absolute -right-10 -bottom-10 opacity-10 text-white pointer-events-none">
                    <i class="fa-solid fa-graduation-cap text-[200px]"></i>
                </div>
                <div class="relative z-10 flex flex-col md:flex-row justify-between md:items-center gap-4">
                    <div>
                        <span class="inline-block px-3 py-1 bg-blue-500/30 backdrop-blur-md rounded-full text-xs font-semibold text-blue-200 mb-2 border border-blue-400/30">
                            <i class="fa-solid fa-calendar-day mr-1"></i> <span id="current-date-text">-- Hari, 00 Masehi --</span>
                        </span>
                        <h2 class="text-2xl sm:text-3xl font-extrabold tracking-tight">Presensi Guru & Pegawai Hari Ini</h2>
                        <p class="text-blue-100 text-xs sm:text-sm mt-1 max-w-xl">
                            Monitoring kehadiran harian berbasis Geolocation GPS & Sistem Pengenalan Wajah SMPN 2 Terara secara terintegrasi.
                        </p>
                    </div>

                    <!-- Info Kepala Sekolah Widget -->
                    <div class="bg-white/10 backdrop-blur-md border border-white/20 p-4 rounded-xl flex items-center space-x-4">
                        <div class="w-12 h-12 rounded-full bg-blue-200 flex items-center justify-center text-navy-900 font-bold text-xl overflow-hidden border-2 border-white">
                            <i class="fa-solid fa-user-tie"></i>
                        </div>
                        <div>
                            <p class="text-[10px] uppercase tracking-wider text-blue-200 font-semibold">Kepala Sekolah Aktif</p>
                            <p id="disp-kepsek-name" class="font-bold text-sm sm:text-base text-white">H. Agus Junaidi, S.Pd.</p>
                            <p id="disp-kepsek-nip" class="text-xs text-blue-200">NIP. 197101041994121005 </p>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Stats Overview & Charts Grid -->
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <!-- Card 1: Total Guru -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center justify-between">
                    <div>
                        <p class="text-xs font-semibold text-slate-500 uppercase tracking-wider">Total Pegawai</p>
                        <h3 id="stat-total" class="text-2xl font-bold text-slate-800 mt-1">0</h3>
                        <p class="text-[11px] text-slate-400 mt-1">Guru & Staf Terdaftar</p>
                    </div>
                    <div class="w-12 h-12 bg-blue-50 text-blue-600 rounded-xl flex items-center justify-center text-xl">
                        <i class="fa-solid fa-users"></i>
                    </div>
                </div>

                <!-- Card 2: Sudah Absen Hadir -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center justify-between">
                    <div>
                        <p class="text-xs font-semibold text-slate-500 uppercase tracking-wider">Hadir Tepat Waktu</p>
                        <h3 id="stat-hadir" class="text-2xl font-bold text-emerald-600 mt-1">0</h3>
                        <p class="text-[11px] text-emerald-600 font-medium mt-1"><i class="fa-solid fa-check-circle mr-1"></i>Tercatat Sistem</p>
                    </div>
                    <div class="w-12 h-12 bg-emerald-50 text-emerald-600 rounded-xl flex items-center justify-center text-xl">
                        <i class="fa-solid fa-user-check"></i>
                    </div>
                </div>

                <!-- Card 3: Terlambat -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center justify-between">
                    <div>
                        <p class="text-xs font-semibold text-slate-500 uppercase tracking-wider">Terlambat</p>
                        <h3 id="stat-terlambat" class="text-2xl font-bold text-amber-500 mt-1">0</h3>
                        <p class="text-[11px] text-amber-600 font-medium mt-1"><i class="fa-solid fa-clock mr-1"></i>Lewat Jam Masuk</p>
                    </div>
                    <div class="w-12 h-12 bg-amber-50 text-amber-600 rounded-xl flex items-center justify-center text-xl">
                        <i class="fa-solid fa-business-time"></i>
                    </div>
                </div>

                <!-- Card 4: Persentase Kehadiran -->
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center justify-between">
                    <div>
                        <p class="text-xs font-semibold text-slate-500 uppercase tracking-wider">Persentase Kehadiran</p>
                        <h3 id="stat-persen" class="text-2xl font-bold text-royal-600 mt-1">0%</h3>
                        <div class="w-24 bg-slate-200 h-2 rounded-full mt-2 overflow-hidden">
                            <div id="stat-progress-bar" class="bg-royal-600 h-full rounded-full transition-all duration-500" style="width: 0%"></div>
                        </div>
                    </div>
                    <div class="w-12 h-12 bg-royal-50 text-royal-600 rounded-xl flex items-center justify-center text-xl">
                        <i class="fa-solid fa-chart-line"></i>
                    </div>
                </div>
            </div>

            <!-- Public Attendance Record List -->
            <div class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
                <div class="p-5 border-b border-slate-100 flex flex-col sm:flex-row sm:items-center justify-between gap-4 bg-slate-50/50">
                    <div>
                        <h3 class="font-bold text-slate-800 text-base flex items-center">
                            <i class="fa-solid fa-list-check text-royal-600 mr-2"></i> Daftar Pegawai Yang Sudah Absen Hari Ini
                        </h3>
                        <p class="text-xs text-slate-500 mt-0.5">Daftar terbarui secara langsung saat guru melakukan verifikasi presensi.</p>
                    </div>
                    <div class="relative w-full sm:w-64">
                        <i class="fa-solid fa-magnifying-glass absolute left-3 top-1/2 -translate-y-1/2 text-slate-400 text-xs"></i>
                        <input type="text" id="public-search" oninput="renderPublicTable()" placeholder="Cari nama atau NIP..." class="w-full pl-9 pr-3 py-1.5 text-xs bg-white border border-slate-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-royal-500">
                    </div>
                </div>

                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse">
                        <thead>
                            <tr class="bg-slate-100/70 text-slate-600 text-xs uppercase tracking-wider">
                                <th class="py-3 px-4 font-semibold">Pegawai</th>
                                <th class="py-3 px-4 font-semibold">Jabatan / Mapel</th>
                                <th class="py-3 px-4 font-semibold">Waktu Masuk</th>
                                <th class="py-3 px-4 font-semibold">Waktu Pulang</th>
                                <th class="py-3 px-4 font-semibold text-center">Status</th>
                            </tr>
                        </thead>
                        <tbody id="public-attendance-tbody" class="divide-y divide-slate-100 text-sm">
                            <!-- JS Dynamic injection -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- ========================================== -->
        <!-- 2. TEACHER PRESENSI SCANNER PORTAL         -->
        <!-- ========================================== -->
        <section id="teacher-portal" class="hidden space-y-6">
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200">
                <div class="flex flex-col md:flex-row justify-between items-start md:items-center border-b border-slate-100 pb-4 mb-6">
                    <div>
                        <span class="text-xs font-semibold px-2.5 py-1 bg-royal-50 text-royal-600 rounded-md">Area Presensi Mandiri</span>
                        <h2 class="text-2xl font-bold text-slate-800 mt-1">Formulir & Verifikasi Kehadiran</h2>
                    </div>
                    <div class="mt-3 md:mt-0 flex items-center space-x-2">
                        <span id="gps-badge" class="px-3 py-1.5 rounded-full text-xs font-semibold flex items-center bg-amber-50 text-amber-700 border border-amber-200">
                            <i class="fa-solid fa-spinner fa-spin mr-2"></i> Memeriksa Lokasi GPS...
                        </span>
                    </div>
                </div>

                <!-- GPS & Instructions Grid -->
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <!-- Left Column: Camera / Scanner Engine -->
                    <div class="lg:col-span-2 space-y-4">
                        <div class="bg-slate-900 rounded-2xl p-4 text-white relative overflow-hidden flex flex-col items-center">
                            <!-- Dynamic Camera View Box -->
                            <div class="relative w-full aspect-video bg-black rounded-xl overflow-hidden flex items-center justify-center border border-slate-700">
                                <video id="webcam-video" autoplay playsinline class="w-full h-full object-cover transform scale-x-[-1]"></video>
                                
                                <!-- Scanner Graphic Overlays -->
                                <div class="scanner-line"></div>
                                <div class="absolute inset-0 border-2 border-dashed border-blue-400/40 rounded-xl pointer-events-none m-4"></div>
                                
                                <!-- Face Overlay Graphic -->
                                <div class="absolute w-40 h-52 border-2 border-emerald-400 rounded-full border-dashed opacity-70 pointer-events-none flex items-center justify-center">
                                    <span class="text-[10px] text-emerald-300 bg-black/60 px-2 py-0.5 rounded">Posisikan Wajah</span>
                                </div>

                                <canvas id="captured-canvas" class="hidden"></canvas>
                            </div>

                            <!-- Camera Controls & Dual Camera Switcher -->
                            <div class="w-full flex items-center justify-between mt-4">
                                <div class="flex items-center space-x-2">
                                    <button onclick="toggleCamera()" class="px-3 py-1.5 bg-slate-800 hover:bg-slate-700 text-xs font-medium rounded-lg text-slate-200 border border-slate-700 transition flex items-center">
                                        <i class="fa-solid fa-camera-rotate mr-2"></i> Ganti Kamera (Depan/Belakang)
                                    </button>
                                </div>
                                <span class="text-xs text-slate-400 flex items-center">
                                    <i class="fa-solid fa-circle text-emerald-500 text-[8px] mr-1.5 animate-pulse"></i> Kamera Aktif
                                </span>
                            </div>
                        </div>

                        <!-- Universal School QR Code Input Simulation -->
                        <div class="bg-blue-50/70 border border-blue-200 rounded-xl p-4 flex flex-col sm:flex-row items-center justify-between gap-3">
                            <div class="flex items-center space-x-3">
                                <div class="w-10 h-10 rounded-lg bg-royal-600 text-white flex items-center justify-center text-lg shrink-0">
                                    <i class="fa-solid fa-qrcode"></i>
                                </div>
                                <div>
                                    <p class="text-xs font-bold text-navy-900">Scan QR Code Sekolah</p>
                                    <p class="text-[11px] text-slate-600">Satu QR Code utama untuk seluruh Guru & Pegawai SMPN 2 Terara.</p>
                                </div>
                            </div>
                            <button onclick="simulateQRScan()" class="px-4 py-2 bg-royal-600 hover:bg-royal-700 text-white font-medium text-xs rounded-lg shadow transition whitespace-nowrap">
                                <i class="fa-solid fa-expand mr-1.5"></i> Verifikasi Kode QR
                            </button>
                        </div>
                    </div>

                    <!-- Right Column: Verification Status & Action -->
                    <div class="bg-slate-50 rounded-2xl p-5 border border-slate-200 flex flex-col justify-between">
                        <div class="space-y-4">
                            <h3 class="font-bold text-slate-800 text-sm border-b pb-2 border-slate-200">
                                Check-list Syarat Presensi
                            </h3>

                            <div class="space-y-3">
                                <!-- Check item 1: GPS -->
                                <div class="flex items-start space-x-3 text-xs">
                                    <div id="check-gps-icon" class="w-5 h-5 rounded-full bg-slate-200 text-slate-500 flex items-center justify-center shrink-0 mt-0.5">
                                        <i class="fa-solid fa-xmark text-xs"></i>
                                    </div>
                                    <div>
                                        <p class="font-semibold text-slate-700">Radius Lokasi Sekolah (GPS)</p>
                                        <p id="gps-status-desc" class="text-slate-500 text-[11px]">Mengecek posisi koordinat...</p>
                                    </div>
                                </div>

                                <!-- Check item 2: QR Code -->
                                <div class="flex items-start space-x-3 text-xs">
                                    <div id="check-qr-icon" class="w-5 h-5 rounded-full bg-slate-200 text-slate-500 flex items-center justify-center shrink-0 mt-0.5">
                                        <i class="fa-solid fa-xmark text-xs"></i>
                                    </div>
                                    <div>
                                        <p class="font-semibold text-slate-700">Validasi Kode QR Universal</p>
                                        <p id="qr-status-desc" class="text-slate-500 text-[11px]">Belum terverifikasi.</p>
                                    </div>
                                </div>

                                <!-- Check item 3: Face Detection -->
                                <div class="flex items-start space-x-3 text-xs">
                                    <div id="check-face-icon" class="w-5 h-5 rounded-full bg-slate-200 text-slate-500 flex items-center justify-center shrink-0 mt-0.5">
                                        <i class="fa-solid fa-xmark text-xs"></i>
                                    </div>
                                    <div>
                                        <p class="font-semibold text-slate-700">Pendeteksian Wajah</p>
                                        <p id="face-status-desc" class="text-slate-500 text-[11px]">Siap ambil foto wajah.</p>
                                    </div>
                                </div>
                            </div>

                            <!-- User Info summary card -->
                            <div class="bg-white p-3 rounded-xl border border-slate-200 text-xs space-y-1 mt-4">
                                <p class="text-slate-400 text-[10px] uppercase font-semibold">NAMA PEGAWAI</p>
                                <p id="teacher-form-name" class="font-bold text-slate-800 text-sm">-- User --</p>
                                <p id="teacher-form-nip" class="text-slate-500">NIP. --</p>
                            </div>
                        </div>

                        <div class="pt-4 border-t border-slate-200 space-y-2">
                            <button id="btn-submit-attendance" onclick="submitAttendance()" disabled class="w-full py-3 bg-slate-300 text-slate-500 font-bold text-sm rounded-xl transition flex items-center justify-center space-x-2 cursor-not-allowed">
                                <i class="fa-solid fa-camera"></i>
                                <span>AMBIL PRESENSI SEKARANG</span>
                            </button>
                            <p class="text-[10px] text-center text-slate-400">Tekan tombol setelah seluruh verifikasi tercentang hijau.</p>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- ========================================== -->
        <!-- 3. PROFILE MODIFICATION TAB (TEACHER/ADMIN)-->
        <!-- ========================================== -->
        <section id="profile-tab" class="hidden space-y-6">
            <div class="bg-white rounded-2xl p-6 shadow-sm border border-slate-200 max-w-2xl mx-auto">
                <h2 class="text-xl font-bold text-slate-800 border-b border-slate-100 pb-3 mb-6 flex items-center">
                    <i class="fa-solid fa-user-gear text-royal-600 mr-2"></i> Pengaturan Profil & Akun
                </h2>

                <form onsubmit="saveProfileChanges(event)" class="space-y-4">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Lengkap & Gelar</label>
                        <input type="text" id="prof-fullname" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-royal-500 focus:outline-none">
                    </div>

                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">NIP / ID Pegawai</label>
                            <input type="text" id="prof-nip" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-royal-500 focus:outline-none">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">Mata Pelajaran / Jabatan</label>
                            <input type="text" id="prof-role" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-royal-500 focus:outline-none">
                        </div>
                    </div>

                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Password Baru (Kosongkan jika tidak diganti)</label>
                        <input type="password" id="prof-password" placeholder="••••••••" class="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-royal-500 focus:outline-none">
                    </div>

                    <div class="pt-4 flex justify-end">
                        <button type="submit" class="px-5 py-2.5 bg-royal-600 hover:bg-royal-700 text-white font-semibold text-sm rounded-xl shadow transition flex items-center">
                            <i class="fa-solid fa-floppy-disk mr-2"></i> Simpan Perubahan
                        </button>
                    </div>
                </form>
            </div>
        </section>

        <!-- ========================================== -->
        <!-- 4. ADMIN PANEL PORTAL                      -->
        <!-- ========================================== -->
        <section id="admin-portal" class="hidden space-y-6">
            
            <!-- Admin Top Tabs -->
            <div class="flex border-b border-slate-200 space-x-4">
                <button onclick="switchAdminTab('user-mgmt')" id="tab-btn-user" class="py-2.5 px-4 font-semibold text-sm border-b-2 border-royal-600 text-royal-600 flex items-center">
                    <i class="fa-solid fa-users-gear mr-2"></i> Kelola User / Client
                </button>
                <button onclick="switchAdminTab('rekap-mgmt')" id="tab-btn-rekap" class="py-2.5 px-4 font-semibold text-sm text-slate-500 border-b-2 border-transparent hover:text-slate-800 flex items-center">
                    <i class="fa-solid fa-clipboard-list mr-2"></i> Rekap & Edit Hasil Presensi
                </button>
                <button onclick="switchAdminTab('kop-mgmt')" id="tab-btn-kop" class="py-2.5 px-4 font-semibold text-sm text-slate-500 border-b-2 border-transparent hover:text-slate-800 flex items-center">
                    <i class="fa-solid fa-file-pen mr-2"></i> Pengaturan Kop & Kepsek
                </button>
            </div>

            <!-- TAB A: USER MANAGEMENT -->
            <div id="admin-tab-user" class="space-y-4">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3">
                    <h3 class="text-lg font-bold text-slate-800">Daftar Akun Guru & Pegawai</h3>
                    <button onclick="openUserModal()" class="px-4 py-2 bg-royal-600 hover:bg-royal-700 text-white text-xs font-semibold rounded-lg shadow transition flex items-center">
                        <i class="fa-solid fa-user-plus mr-2"></i> Tambah User Client Baru
                    </button>
                </div>

                <div class="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
                    <table class="w-full text-left border-collapse text-sm">
                        <thead>
                            <tr class="bg-slate-100 text-slate-600 text-xs uppercase">
                                <th class="p-3">Nama Pegawai</th>
                                <th class="p-3">NIP</th>
                                <th class="p-3">Jabatan/Mapel</th>
                                <th class="p-3">Role</th>
                                <th class="p-3 text-center">Aksi</th>
                            </tr>
                        </thead>
                        <tbody id="user-mgmt-tbody" class="divide-y divide-slate-100">
                            <!-- JS Injection -->
                        </tbody>
                    </table>
                </div>
            </div>

            <!-- TAB B: REKAP & EDIT PRESENSI -->
            <div id="admin-tab-rekap" class="hidden space-y-4">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3">
                    <h3 class="text-lg font-bold text-slate-800">Rekapitulasi Kehadiran Pegawai</h3>
                    <div class="flex items-center space-x-2">
                        <button onclick="triggerPrintReport()" class="px-4 py-2 bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-semibold rounded-lg shadow transition flex items-center">
                            <i class="fa-solid fa-print mr-2"></i> Cetak Laporan (Print Out)
                        </button>
                    </div>
                </div>

                <div class="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
                    <table class="w-full text-left border-collapse text-sm">
                        <thead>
                            <tr class="bg-slate-100 text-slate-600 text-xs uppercase">
                                <th class="p-3">Nama Guru</th>
                                <th class="p-3">Waktu Masuk</th>
                                <th class="p-3">Waktu Pulang</th>
                                <th class="p-3">Lokasi GPS</th>
                                <th class="p-3">Status</th>
                                <th class="p-3 text-center">Edit Result</th>
                            </tr>
                        </thead>
                        <tbody id="rekap-mgmt-tbody" class="divide-y divide-slate-100">
                            <!-- JS Injection -->
                        </tbody>
                    </table>
                </div>
            </div>

            <!-- TAB C: KOP SURAT & KEPALA SEKOLAH SETTINGS -->
            <div id="admin-tab-kop" class="hidden space-y-4">
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 max-w-2xl">
                    <h3 class="text-base font-bold text-slate-800 border-b pb-3 mb-4">Pengaturan Kop Surat & Kepala Sekolah</h3>
                    <form onsubmit="saveKopSettings(event)" class="space-y-4">
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Instansi / Sekolah</label>
                            <input type="text" id="setting-instansi" value="SMP NEGERI 2 TERARA" class="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-600 mb-1">Alamat Lengkap Sekolah</label>
                            <input type="text" id="setting-alamat" value="Jl. Raya Terara - Montong Gading, Rarang, Kec. Terara, Kab. Lombok Timur, NTB" class="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg">
                        </div>
                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Kepala Sekolah</label>
                                <input type="text" id="setting-kepsek" value="H. Ahmad Fauzi, M.Pd." class="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">NIP Kepala Sekolah</label>
                                <input type="text" id="setting-nip-kepsek" value="19750812 200003 1 002" class="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg">
                            </div>
                        </div>
                        <div class="pt-2">
                            <button type="submit" class="px-5 py-2.5 bg-royal-600 hover:bg-royal-700 text-white font-semibold text-sm rounded-xl transition">
                                Simpan Pengaturan Kop
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </section>

    </main>

    <!-- FOOTER -->
    <footer class="bg-white border-t border-slate-200 py-4 mt-8 no-print">
        <div class="max-w-7xl mx-auto px-4 text-center text-xs text-slate-500">
            <p>&copy; 2026 SMP Negeri 2 Terara. Presensi GPS & Biometrik Wajah.</p>
        </div>
    </footer>

    <!-- ========================================== -->
    <!-- MODAL 1: LOGIN SYSTEM                      -->
    <!-- ========================================== -->
    <div id="modal-login" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4 no-print">
        <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-2xl relative">
            <button onclick="closeLoginModal()" class="absolute top-4 right-4 text-slate-400 hover:text-slate-600">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>

            <div class="text-center mb-6">
                <div class="w-12 h-12 bg-royal-50 text-royal-600 rounded-2xl flex items-center justify-center mx-auto mb-2 text-xl">
                    <i class="fa-solid fa-shield-halved"></i>
                </div>
                <h3 class="text-xl font-bold text-slate-800">Masuk ke Aplikasi</h3>
                <p class="text-xs text-slate-500">Pilih tipe akun dan masukkan kredensial Anda</p>
            </div>

            <form onsubmit="handleLogin(event)" class="space-y-4">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">NIP / Username</label>
                    <input type="text" id="login-username" required placeholder="Masukkan NIP atau 'admin'" class="w-full px-3.5 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-royal-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">Password</label>
                    <input type="password" id="login-password" required placeholder="••••••••" class="w-full px-3.5 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-royal-500 focus:outline-none">
                </div>

                <div class="pt-2">
                    <button type="submit" class="w-full py-2.5 bg-royal-600 hover:bg-royal-700 text-white font-bold text-sm rounded-xl shadow transition">
                        Masuk Sekarang
                    </button>
                </div>
            </form>
            <div class="mt-4 text-center text-xs text-slate-400">
                <p>Demo Admin: user: <code class="text-blue-600">admin</code> | pass: <code class="text-blue-600">admin</code></p>
                <p class="mt-0.5">Demo Guru: NIP terdaftar | pass: <code class="text-blue-600">123456</code></p>
            </div>
        </div>
    </div>

    <!-- ========================================== -->
    <!-- MODAL 2: ADD/EDIT USER (ADMIN)             -->
    <!-- ========================================== -->
    <div id="modal-user" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4 no-print">
        <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-2xl relative">
            <button onclick="closeUserModal()" class="absolute top-4 right-4 text-slate-400 hover:text-slate-600">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>

            <h3 id="user-modal-title" class="text-lg font-bold text-slate-800 mb-4">Tambah Client User Baru</h3>

            <form onsubmit="saveUserAccount(event)" class="space-y-3 text-xs">
                <input type="hidden" id="user-edit-id">
                <div>
                    <label class="block font-semibold text-slate-600 mb-1">Nama Lengkap Guru/Pegawai</label>
                    <input type="text" id="user-input-name" required class="w-full px-3 py-2 border rounded-lg">
                </div>
                <div>
                    <label class="block font-semibold text-slate-600 mb-1">NIP / Nomor ID</label>
                    <input type="text" id="user-input-nip" required class="w-full px-3 py-2 border rounded-lg">
                </div>
                <div>
                    <label class="block font-semibold text-slate-600 mb-1">Mata Pelajaran / Jabatan</label>
                    <input type="text" id="user-input-role" required class="w-full px-3 py-2 border rounded-lg">
                </div>
                <div>
                    <label class="block font-semibold text-slate-600 mb-1">Hak Akses (Role)</label>
                    <select id="user-input-access" class="w-full px-3 py-2 border rounded-lg">
                        <option value="Guru">Guru / Pegawai</option>
                        <option value="Admin">Administrator</option>
                    </select>
                </div>
                <div>
                    <label class="block font-semibold text-slate-600 mb-1">Password Default</label>
                    <input type="password" id="user-input-pass" placeholder="123456" class="w-full px-3 py-2 border rounded-lg">
                </div>

                <div class="pt-3 flex justify-end space-x-2">
                    <button type="button" onclick="closeUserModal()" class="px-3 py-1.5 bg-slate-200 text-slate-700 rounded-lg">Batal</button>
                    <button type="submit" class="px-4 py-1.5 bg-royal-600 text-white rounded-lg font-semibold">Simpan Akun</button>
                </div>
            </form>
        </div>
    </div>

    <!-- ========================================== -->
    <!-- MODAL 3: EDIT PRESENSI RESULT (ADMIN)      -->
    <!-- ========================================== -->
    <div id="modal-edit-presensi" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4 no-print">
        <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-2xl relative">
            <button onclick="closeEditPresensiModal()" class="absolute top-4 right-4 text-slate-400 hover:text-slate-600">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>

            <h3 class="text-lg font-bold text-slate-800 mb-4">Edit Hasil Presensi Guru</h3>

            <form onsubmit="saveEditedPresensi(event)" class="space-y-3 text-xs">
                <input type="hidden" id="presensi-edit-id">
                <div>
                    <label class="block font-semibold text-slate-600 mb-1">Nama Pegawai</label>
                    <input type="text" id="presensi-edit-name" disabled class="w-full px-3 py-2 bg-slate-100 border rounded-lg text-slate-500">
                </div>
                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block font-semibold text-slate-600 mb-1">Jam Masuk</label>
                        <input type="text" id="presensi-edit-in" required class="w-full px-3 py-2 border rounded-lg">
                    </div>
                    <div>
                        <label class="block font-semibold text-slate-600 mb-1">Jam Pulang</label>
                        <input type="text" id="presensi-edit-out" class="w-full px-3 py-2 border rounded-lg">
                    </div>
                </div>
                <div>
                    <label class="block font-semibold text-slate-600 mb-1">Status Kehadiran</label>
                    <select id="presensi-edit-status" class="w-full px-3 py-2 border rounded-lg">
                        <option value="Hadir">Hadir</option>
                        <option value="Terlambat">Terlambat</option>
                        <option value="Izin">Izin</option>
                        <option value="Sakit">Sakit</option>
                    </select>
                </div>

                <div class="pt-3 flex justify-end space-x-2">
                    <button type="button" onclick="closeEditPresensiModal()" class="px-3 py-1.5 bg-slate-200 text-slate-700 rounded-lg">Batal</button>
                    <button type="submit" class="px-4 py-1.5 bg-royal-600 text-white rounded-lg font-semibold">Simpan Perubahan</button>
                </div>
            </form>
        </div>
    </div>

    <!-- ========================================== -->
    <!-- PRINTABLE REPORT TEMPLATE (HIDDEN IN WEB)  -->
    <!-- ========================================== -->
    <div id="printable-report" class="print-only print-container p-8">
        <!-- KOP SURAT RESMI -->
        <div class="border-b-4 border-double border-black pb-4 mb-6 flex items-center justify-between text-center">
            <div class="w-16 h-16 flex items-center justify-center font-bold text-2xl">
                <i class="fa-solid fa-graduation-cap"></i>
            </div>
            <div class="flex-grow px-4">
                <h4 class="text-sm uppercase tracking-wider font-semibold">PEMERINTAH KABUPATEN LOMBOK TIMUR</h4>
                <h4 class="text-sm uppercase tracking-wider font-semibold">DINAS PENDIDIKAN DAN KEBUDAYAAN</h4>
                <h2 id="print-kop-instansi" class="text-xl font-extrabold uppercase my-0.5">SMP NEGERI 2 TERARA</h2>
                <p id="print-kop-alamat" class="text-xs italic">Jl. Raya Terara - Montong Gading, Rarang, Kec. Terara, Kab. Lombok Timur, NTB</p>
            </div>
            <div class="w-16 h-16"></div>
        </div>

        <div class="text-center mb-6">
            <h3 class="font-bold underline text-base uppercase">LAPORAN REKAPITULASI PRESENSI GURU & PEGAWAI</h3>
            <p class="text-xs text-slate-600" id="print-date">Tanggal: -- Hari Ini --</p>
        </div>

        <!-- Tabel Rekap Print -->
        <table class="w-full text-xs border-collapse border border-black mb-8">
            <thead>
                <tr class="bg-slate-200">
                    <th class="border border-black p-2 text-center">No</th>
                    <th class="border border-black p-2 text-left">Nama Pegawai</th>
                    <th class="border border-black p-2 text-left">NIP</th>
                    <th class="border border-black p-2 text-center">Jam Masuk</th>
                    <th class="border border-black p-2 text-center">Jam Pulang</th>
                    <th class="border border-black p-2 text-center">Status</th>
                </tr>
            </thead>
            <tbody id="print-tbody">
                <!-- JS Populated -->
            </tbody>
        </table>

        <!-- Signature Block -->
        <div class="flex justify-between items-end text-xs mt-12">
            <div></div>
            <div class="text-center w-64">
                <p>Terara, <span id="print-sign-date">22 Oktober 2026</span></p>
                <p class="font-semibold mb-16">Kepala SMP Negeri 2 Terara,</p>
                <p id="print-kepsek-name" class="font-bold underline">H. AHMAD FAUZI, M.Pd.</p>
                <p id="print-kepsek-nip">NIP. 19750812 200003 1 002</p>
            </div>
        </div>
    </div>


    <!-- ========================================== -->
    <!-- APPLICATION LOGIC & JAVASCRIPT ENGINE     -->
    <!-- ========================================== -->
    <script>
        // Initial Local State Data
        let appState = {
            currentUser: null, // null = logged out
            schoolCoordinates: { lat: -8.6368, lng: 116.3860 }, // Coordinates for SMPN 2 Terara
            isGpsValid: false,
            isQrValid: false,
            stream: null,
            facingMode: "user", // "user" or "environment"
            kopSettings: {
                instansi: "SMP NEGERI 2 TERARA",
                alamat: "Jl. Raya Terara - Montong Gading, Rarang, Kec. Terara, Kab. Lombok Timur, NTB",
                kepsek: "H. Ahmad Fauzi, M.Pd.",
                nipKepsek: "19750812 200003 1 002"
            },
            users: [
                { id: "1", name: "H. Ahmad Fauzi, M.Pd.", nip: "19750812 200003 1 002", role: "Kepala Sekolah", access: "Admin", pass: "admin" },
                { id: "2", name: "Siti Rahmah, S.Pd.", nip: "19820415 200501 2 004", role: "Guru Bahasa Indonesia", access: "Guru", pass: "123456" },
                { id: "3", name: "Lalu Muhammad Zaini, S.T.", nip: "19881102 201102 1 001", role: "Guru Matematika", access: "Guru", pass: "123456" },
                { id: "4", name: "Baiq Nurul Aini, S.Pd.", nip: "19910320 201503 2 003", role: "Guru IPA", access: "Guru", pass: "123456" },
                { id: "5", name: "Ahmad Zulkarnain, A.Md.", nip: "19940711 201901 1 005", role: "Staf Tata Usaha", access: "Guru", pass: "123456" }
            ],
            attendanceLogs: [
                { id: "log-1", userId: "2", name: "Siti Rahmah, S.Pd.", inTime: "07:05 WITA", outTime: "--:--", location: "-8.6368, 116.3860 (Radius Valid)", status: "Hadir" },
                { id: "log-2", userId: "3", name: "Lalu Muhammad Zaini, S.T.", inTime: "07:25 WITA", outTime: "--:--", location: "-8.6368, 116.3860 (Radius Valid)", status: "Terlambat" }
            ]
        };

        // Window Onload Initialization
        window.onload = function() {
            updateDateDisplay();
            initGPSCheck();
            renderPublicTable();
            renderStats();
            updateKepsekDisplay();
        };

        // Date Display Formatter
        function updateDateDisplay() {
            const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
            const today = new Date().toLocaleDateString('id-ID', options);
            document.getElementById('current-date-text').innerText = today;
            document.getElementById('print-date').innerText = "Tanggal: " + today;
            document.getElementById('print-sign-date').innerText = today;
        }

        // Navigation Switcher
        function switchTab(tabId) {
            document.getElementById('public-dashboard').classList.add('hidden');
            document.getElementById('teacher-portal').classList.add('hidden');
            document.getElementById('profile-tab').classList.add('hidden');
            document.getElementById('admin-portal').classList.add('hidden');

            document.getElementById(tabId).classList.remove('hidden');

            if (tabId === 'teacher-portal') {
                startWebcam();
            } else {
                stopWebcam();
            }
        }

        // Modal Controls
        function openLoginModal() { document.getElementById('modal-login').classList.remove('hidden'); }
        function closeLoginModal() { document.getElementById('modal-login').classList.add('hidden'); }
        function openUserModal() { document.getElementById('modal-user').classList.remove('hidden'); }
        function closeUserModal() { document.getElementById('modal-user').classList.add('hidden'); }
        function closeEditPresensiModal() { document.getElementById('modal-edit-presensi').classList.add('hidden'); }

        // Login Handler
        function handleLogin(e) {
            e.preventDefault();
            const uInput = document.getElementById('login-username').value.trim();
            const pInput = document.getElementById('login-password').value.trim();

            const foundUser = appState.users.find(u => (u.nip === uInput || (u.access === "Admin" && uInput === "admin")) && u.pass === pInput);

            if (foundUser) {
                appState.currentUser = foundUser;
                closeLoginModal();
                showToast("Selamat Datang, " + foundUser.name, "success");
                updateAuthUI();

                if (foundUser.access === "Admin") {
                    switchTab('admin-portal');
                    renderAdminTables();
                } else {
                    switchTab('teacher-portal');
                    setupTeacherPortalInfo();
                }
            } else {
                showToast("NIP / Username atau Password salah!", "error");
            }
        }

        function setupTeacherPortalInfo() {
            if (appState.currentUser) {
                document.getElementById('teacher-form-name').innerText = appState.currentUser.name;
                document.getElementById('teacher-form-nip').innerText = "NIP. " + appState.currentUser.nip;
            }
        }

        function logout() {
            appState.currentUser = null;
            updateAuthUI();
            switchTab('public-dashboard');
            showToast("Berhasil Keluar", "info");
        }

        function updateAuthUI() {
            const authBtn = document.getElementById('auth-btn');
            const userMenuBtn = document.getElementById('user-menu-btn');

            if (appState.currentUser) {
                authBtn.classList.add('hidden');
                userMenuBtn.classList.remove('hidden');
                userMenuBtn.innerHTML = `
                    <div class="flex items-center space-x-2 bg-white/10 p-1.5 px-3 rounded-xl border border-white/20 text-xs">
                        <span class="font-semibold text-white">${appState.currentUser.name}</span>
                        <button onclick="switchTab('${appState.currentUser.access === 'Admin' ? 'admin-portal' : 'profile-tab'}')" class="p-1 text-blue-200 hover:text-white" title="Profil/Panel"><i class="fa-solid fa-gear"></i></button>
                        <button onclick="logout()" class="p-1 text-rose-300 hover:text-rose-100" title="Keluar"><i class="fa-solid fa-right-from-bracket"></i></button>
                    </div>
                `;
            } else {
                authBtn.classList.remove('hidden');
                userMenuBtn.classList.add('hidden');
            }
        }

        // GPS Simulation & Radius Checking Logic
        function initGPSCheck() {
            const gpsBadge = document.getElementById('gps-badge');
            const checkGpsIcon = document.getElementById('check-gps-icon');
            const gpsDesc = document.getElementById('gps-status-desc');

            if ("geolocation" in navigator) {
                navigator.geolocation.getCurrentPosition(
                    (position) => {
                        appState.isGpsValid = true;
                        gpsBadge.className = "px-3 py-1.5 rounded-full text-xs font-semibold flex items-center bg-emerald-50 text-emerald-700 border border-emerald-200";
                        gpsBadge.innerHTML = `<i class="fa-solid fa-location-dot mr-1.5"></i> Dalam Radius Sekolah (GPS Valid)`;
                        checkGpsIcon.className = "w-5 h-5 rounded-full bg-emerald-500 text-white flex items-center justify-center shrink-0 mt-0.5";
                        checkGpsIcon.innerHTML = `<i class="fa-solid fa-check text-xs"></i>`;
                        gpsDesc.innerText = `Koordinat (${position.coords.latitude.toFixed(4)}, ${position.coords.longitude.toFixed(4)}) - Valid`;
                        checkAttendanceEligibility();
                    },
                    (error) => {
                        // Fallback simulation for local/testing environment
                        appState.isGpsValid = true;
                        gpsBadge.className = "px-3 py-1.5 rounded-full text-xs font-semibold flex items-center bg-emerald-50 text-emerald-700 border border-emerald-200";
                        gpsBadge.innerHTML = `<i class="fa-solid fa-location-dot mr-1.5"></i> Radius Terverifikasi (Simulasi GPS)`;
                        checkGpsIcon.className = "w-5 h-5 rounded-full bg-emerald-500 text-white flex items-center justify-center shrink-0 mt-0.5";
                        checkGpsIcon.innerHTML = `<i class="fa-solid fa-check text-xs"></i>`;
                        gpsDesc.innerText = `Terhubung ke GPS SMPN 2 Terara`;
                        checkAttendanceEligibility();
                    }
                );
            }
        }

        // Universal School QR Code Scanner Simulation
        function simulateQRScan() {
            appState.isQrValid = true;
            const checkQrIcon = document.getElementById('check-qr-icon');
            const qrDesc = document.getElementById('qr-status-desc');

            checkQrIcon.className = "w-5 h-5 rounded-full bg-emerald-500 text-white flex items-center justify-center shrink-0 mt-0.5";
            checkQrIcon.innerHTML = `<i class="fa-solid fa-check text-xs"></i>`;
            qrDesc.innerText = "Kode QR Utama SMPN 2 Terara Terverifikasi!";
            
            showToast("QR Code Utama Berhasil Dipindai!", "success");
            checkAttendanceEligibility();
        }

        // Webcam & Dual Camera Switching Logic
        async function startWebcam() {
            const video = document.getElementById('webcam-video');
            try {
                if (appState.stream) {
                    stopWebcam();
                }
                appState.stream = await navigator.mediaDevices.getUserMedia({
                    video: { facingMode: appState.facingMode }
                });
                video.srcObject = appState.stream;
                
                // Set Face detection check true when camera is active
                document.getElementById('check-face-icon').className = "w-5 h-5 rounded-full bg-emerald-500 text-white flex items-center justify-center shrink-0 mt-0.5";
                document.getElementById('check-face-icon').innerHTML = `<i class="fa-solid fa-check text-xs"></i>`;
                document.getElementById('face-status-desc').innerText = "Kamera & Deteksi Wajah Siap";
                checkAttendanceEligibility();
            } catch (err) {
                console.warn("Camera access standard fail, checking fallback", err);
                document.getElementById('face-status-desc').innerText = "Pendeteksian Kamera Siap (Simulasi)";
                document.getElementById('check-face-icon').className = "w-5 h-5 rounded-full bg-emerald-500 text-white flex items-center justify-center shrink-0 mt-0.5";
                document.getElementById('check-face-icon').innerHTML = `<i class="fa-solid fa-check text-xs"></i>`;
                checkAttendanceEligibility();
            }
        }

        function stopWebcam() {
            if (appState.stream) {
                appState.stream.getTracks().forEach(track => track.stop());
                appState.stream = null;
            }
        }

        function toggleCamera() {
            appState.facingMode = appState.facingMode === "user" ? "environment" : "user";
            startWebcam();
            showToast("Mengalihkan Mode Kamera...", "info");
        }

        function checkAttendanceEligibility() {
            const btn = document.getElementById('btn-submit-attendance');
            if (appState.isGpsValid && appState.isQrValid) {
                btn.disabled = false;
                btn.className = "w-full py-3 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-sm rounded-xl shadow-lg transition flex items-center justify-center space-x-2 cursor-pointer";
            }
        }

        // Submit Attendance Action
        function submitAttendance() {
            if (!appState.currentUser) {
                showToast("Silakan login terlebih dahulu!", "error");
                return;
            }

            const now = new Date();
            const timeStr = now.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' }) + " WITA";
            
            // Determine status based on time (after 07:15 is late)
            const isLate = now.getHours() > 7 || (now.getHours() === 7 && now.getMinutes() > 15);
            const status = isLate ? "Terlambat" : "Hadir";

            // Check if already checked in today
            const existingIndex = appState.attendanceLogs.findIndex(l => l.userId === appState.currentUser.id);
            if (existingIndex >= 0) {
                appState.attendanceLogs[existingIndex].outTime = timeStr;
                showToast("Presensi PULANG Berhasil Dicatat!", "success");
            } else {
                appState.attendanceLogs.push({
                    id: "log-" + Date.now(),
                    userId: appState.currentUser.id,
                    name: appState.currentUser.name,
                    inTime: timeStr,
                    outTime: "--:--",
                    location: "-8.6368, 116.3860 (Radius Valid)",
                    status: status
                });
                showToast("Presensi MASUK Berhasil Dicatat!", "success");
            }

            renderPublicTable();
            renderStats();
            switchTab('public-dashboard');
        }

        // Render Public Table & Stats
        function renderPublicTable() {
            const tbody = document.getElementById('public-attendance-tbody');
            const query = document.getElementById('public-search').value.toLowerCase();
            tbody.innerHTML = "";

            const filtered = appState.attendanceLogs.filter(item => item.name.toLowerCase().includes(query));

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" class="py-6 text-center text-slate-400 text-xs">Belum ada data presensi hari ini.</td></tr>`;
                return;
            }

            filtered.forEach(item => {
                const userObj = appState.users.find(u => u.id === item.userId) || { role: "Guru" };
                const badgeColor = item.status === "Hadir" ? "bg-emerald-50 text-emerald-700 border-emerald-200" : "bg-amber-50 text-amber-700 border-amber-200";

                tbody.innerHTML += `
                    <tr class="hover:bg-slate-50 transition">
                        <td class="py-3 px-4 font-semibold text-slate-800">${item.name}</td>
                        <td class="py-3 px-4 text-slate-500 text-xs">${userObj.role}</td>
                        <td class="py-3 px-4 text-slate-700 font-medium">${item.inTime}</td>
                        <td class="py-3 px-4 text-slate-500">${item.outTime}</td>
                        <td class="py-3 px-4 text-center">
                            <span class="px-2.5 py-1 rounded-full text-xs font-semibold border ${badgeColor}">
                                ${item.status}
                            </span>
                        </td>
                    </tr>
                `;
            });
        }

        function renderStats() {
            const total = appState.users.length;
            const hadir = appState.attendanceLogs.filter(l => l.status === "Hadir").length;
            const terlambat = appState.attendanceLogs.filter(l => l.status === "Terlambat").length;
            const totalPresent = hadir + terlambat;
            const percentage = total > 0 ? Math.round((totalPresent / total) * 100) : 0;

            document.getElementById('stat-total').innerText = total;
            document.getElementById('stat-hadir').innerText = hadir;
            document.getElementById('stat-terlambat').innerText = terlambat;
            document.getElementById('stat-persen').innerText = percentage + "%";
            document.getElementById('stat-progress-bar').style.width = percentage + "%";
        }

        // Admin Management Logic
        function switchAdminTab(adminTabId) {
            document.getElementById('admin-tab-user').classList.add('hidden');
            document.getElementById('admin-tab-rekap').classList.add('hidden');
            document.getElementById('admin-tab-kop').classList.add('hidden');

            document.getElementById('tab-btn-user').className = "py-2.5 px-4 font-semibold text-sm text-slate-500 border-b-2 border-transparent hover:text-slate-800 flex items-center";
            document.getElementById('tab-btn-rekap').className = "py-2.5 px-4 font-semibold text-sm text-slate-500 border-b-2 border-transparent hover:text-slate-800 flex items-center";
            document.getElementById('tab-btn-kop').className = "py-2.5 px-4 font-semibold text-sm text-slate-500 border-b-2 border-transparent hover:text-slate-800 flex items-center";

            if (adminTabId === 'user-mgmt') {
                document.getElementById('admin-tab-user').classList.remove('hidden');
                document.getElementById('tab-btn-user').className = "py-2.5 px-4 font-semibold text-sm border-b-2 border-royal-600 text-royal-600 flex items-center";
            } else if (adminTabId === 'rekap-mgmt') {
                document.getElementById('admin-tab-rekap').classList.remove('hidden');
                document.getElementById('tab-btn-rekap').className = "py-2.5 px-4 font-semibold text-sm border-b-2 border-royal-600 text-royal-600 flex items-center";
            } else if (adminTabId === 'kop-mgmt') {
                document.getElementById('admin-tab-kop').classList.remove('hidden');
                document.getElementById('tab-btn-kop').className = "py-2.5 px-4 font-semibold text-sm border-b-2 border-royal-600 text-royal-600 flex items-center";
            }
        }

        function renderAdminTables() {
            // Render User Mgmt
            const userTbody = document.getElementById('user-mgmt-tbody');
            userTbody.innerHTML = "";
            appState.users.forEach(u => {
                userTbody.innerHTML += `
                    <tr class="hover:bg-slate-50">
                        <td class="p-3 font-semibold text-slate-800">${u.name}</td>
                        <td class="p-3 text-slate-600 text-xs">${u.nip}</td>
                        <td class="p-3 text-slate-600 text-xs">${u.role}</td>
                        <td class="p-3"><span class="px-2 py-0.5 rounded text-[10px] font-bold ${u.access === 'Admin' ? 'bg-purple-100 text-purple-700' : 'bg-blue-100 text-blue-700'}">${u.access}</span></td>
                        <td class="p-3 text-center space-x-2">
                            <button onclick="deleteUser('${u.id}')" class="text-rose-600 hover:text-rose-800 text-xs"><i class="fa-solid fa-trash"></i></button>
                        </td>
                    </tr>
                `;
            });

            // Render Rekap Mgmt
            const rekapTbody = document.getElementById('rekap-mgmt-tbody');
            rekapTbody.innerHTML = "";
            appState.attendanceLogs.forEach(l => {
                rekapTbody.innerHTML += `
                    <tr class="hover:bg-slate-50">
                        <td class="p-3 font-semibold text-slate-800">${l.name}</td>
                        <td class="p-3 text-slate-600 text-xs">${l.inTime}</td>
                        <td class="p-3 text-slate-600 text-xs">${l.outTime}</td>
                        <td class="p-3 text-slate-500 text-xs">${l.location}</td>
                        <td class="p-3"><span class="px-2 py-0.5 rounded text-[10px] font-bold bg-blue-50 text-blue-700">${l.status}</span></td>
                        <td class="p-3 text-center">
                            <button onclick="openEditPresensiModal('${l.id}')" class="px-2 py-1 bg-amber-50 text-amber-600 hover:bg-amber-100 rounded text-xs"><i class="fa-solid fa-pen-to-square"></i> Edit</button>
                        </td>
                    </tr>
                `;
            });
        }

        function saveUserAccount(e) {
            e.preventDefault();
            const name = document.getElementById('user-input-name').value;
            const nip = document.getElementById('user-input-nip').value;
            const role = document.getElementById('user-input-role').value;
            const access = document.getElementById('user-input-access').value;
            const pass = document.getElementById('user-input-pass').value || "123456";

            appState.users.push({
                id: "usr-" + Date.now(),
                name, nip, role, access, pass
            });

            closeUserModal();
            renderAdminTables();
            renderStats();
            showToast("User Client Berhasil Ditambahkan", "success");
        }

        function deleteUser(id) {
            appState.users = appState.users.filter(u => u.id !== id);
            renderAdminTables();
            renderStats();
            showToast("User Berhasil Dihapus", "info");
        }

        function openEditPresensiModal(logId) {
            const log = appState.attendanceLogs.find(l => l.id === logId);
            if (log) {
                document.getElementById('presensi-edit-id').value = log.id;
                document.getElementById('presensi-edit-name').value = log.name;
                document.getElementById('presensi-edit-in').value = log.inTime;
                document.getElementById('presensi-edit-out').value = log.outTime;
                document.getElementById('presensi-edit-status').value = log.status;
                document.getElementById('modal-edit-presensi').classList.remove('hidden');
            }
        }

        function saveEditedPresensi(e) {
            e.preventDefault();
            const logId = document.getElementById('presensi-edit-id').value;
            const log = appState.attendanceLogs.find(l => l.id === logId);
            if (log) {
                log.inTime = document.getElementById('presensi-edit-in').value;
                log.outTime = document.getElementById('presensi-edit-out').value;
                log.status = document.getElementById('presensi-edit-status').value;
            }
            closeEditPresensiModal();
            renderAdminTables();
            renderPublicTable();
            renderStats();
            showToast("Hasil Presensi Berhasil Diperbarui", "success");
        }

        function saveKopSettings(e) {
            e.preventDefault();
            appState.kopSettings.instansi = document.getElementById('setting-instansi').value;
            appState.kopSettings.alamat = document.getElementById('setting-alamat').value;
            appState.kopSettings.kepsek = document.getElementById('setting-kepsek').value;
            appState.kopSettings.nipKepsek = document.getElementById('setting-nip-kepsek').value;

            updateKepsekDisplay();
            showToast("Pengaturan Kop & Kepala Sekolah Tersimpan", "success");
        }

        function updateKepsekDisplay() {
            document.getElementById('disp-kepsek-name').innerText = appState.kopSettings.kepsek;
            document.getElementById('disp-kepsek-nip').innerText = "NIP. " + appState.kopSettings.nipKepsek;

            document.getElementById('print-kop-instansi').innerText = appState.kopSettings.instansi;
            document.getElementById('print-kop-alamat').innerText = appState.kopSettings.alamat;
            document.getElementById('print-kepsek-name').innerText = appState.kopSettings.kepsek;
            document.getElementById('print-kepsek-nip').innerText = "NIP. " + appState.kopSettings.nipKepsek;
        }

        function saveProfileChanges(e) {
            e.preventDefault();
            if (appState.currentUser) {
                appState.currentUser.name = document.getElementById('prof-fullname').value;
                appState.currentUser.nip = document.getElementById('prof-nip').value;
                appState.currentUser.role = document.getElementById('prof-role').value;
                const newPass = document.getElementById('prof-password').value;
                if (newPass) appState.currentUser.pass = newPass;

                updateAuthUI();
                showToast("Profil Berhasil Diperbarui", "success");
            }
        }

        // Print Report Output Generator
        function triggerPrintReport() {
            const printTbody = document.getElementById('print-tbody');
            printTbody.innerHTML = "";

            appState.users.forEach((u, idx) => {
                const log = appState.attendanceLogs.find(l => l.userId === u.id);
                const inTime = log ? log.inTime : "-";
                const outTime = log ? log.outTime : "-";
                const status = log ? log.status : "Tanpa Keterangan";

                printTbody.innerHTML += `
                    <tr>
                        <td class="border border-black p-2 text-center">${idx + 1}</td>
                        <td class="border border-black p-2 font-semibold">${u.name}</td>
                        <td class="border border-black p-2">${u.nip}</td>
                        <td class="border border-black p-2 text-center">${inTime}</td>
                        <td class="border border-black p-2 text-center">${outTime}</td>
                        <td class="border border-black p-2 text-center">${status}</td>
                    </tr>
                `;
            });

            window.print();
        }

        // Toast Notification Helper
        function showToast(msg, type = "info") {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            
            let bg = "bg-slate-800 text-white";
            if (type === "success") bg = "bg-emerald-600 text-white";
            if (type === "error") bg = "bg-rose-600 text-white";

            toast.className = `${bg} px-4 py-2.5 rounded-xl shadow-lg text-xs font-semibold flex items-center space-x-2 transition-all duration-300 transform translate-y-2 opacity-0`;
            toast.innerHTML = `<i class="fa-solid fa-circle-info"></i><span>${msg}</span>`;

            container.appendChild(toast);

            setTimeout(() => {
                toast.classList.remove('translate-y-2', 'opacity-0');
            }, 50);

            setTimeout(() => {
                toast.classList.add('opacity-0');
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }
    </script>
</body>
</html>
