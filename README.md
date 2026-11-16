# sinau-privat
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PrivatKu - Administrasi & Analisis Les Privat</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Chart.js untuk grafik perkembangan belajar -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                    colors: {
                        brand: {
                            50: '#eef2ff',
                            100: '#e0e7ff',
                            500: '#6366f1',
                            600: '#4f46e5',
                            700: '#4338ca',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8fafc;
        }
        /* Pengaturan khusus saat mencetak invoice (Print / PDF) */
        @media print {
            body * {
                visibility: hidden;
            }
            #printable-invoice, #printable-invoice * {
                visibility: visible;
            }
            #printable-invoice {
                position: absolute;
                left: 0;
                top: 0;
                width: 100%;
                box-shadow: none !important;
                border: none !important;
            }
            .no-print {
                display: none !important;
            }
        }
        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f5f9; 
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1; 
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8; 
        }
    </style>
</head>
<body class="text-slate-800 antialiased selection:bg-brand-100 selection:text-brand-700">

    <!-- Notifikasi Toast (Pengganti alert) -->
    <div id="toast-container" class="fixed bottom-4 right-4 z-50 flex flex-col gap-2 pointer-events-none"></div>

    <!-- Modal Konfirmasi Custom (Pengganti confirm) -->
    <div id="confirm-modal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 hidden items-center justify-center p-4">
        <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-xl border border-slate-100 transform transition-all">
            <div class="flex items-center gap-3 text-amber-600 mb-3">
                <div class="p-2 bg-amber-50 rounded-xl">
                    <i data-lucide="alert-triangle" class="w-6 h-6"></i>
                </div>
                <h3 id="confirm-title" class="text-lg font-bold text-slate-900">Konfirmasi Tindakan</h3>
            </div>
            <p id="confirm-message" class="text-slate-600 text-sm mb-6 leading-relaxed">Apakah Anda yakin ingin melanjutkan?</p>
            <div class="flex justify-end gap-3">
                <button id="confirm-cancel-btn" class="px-4 py-2.5 rounded-xl text-sm font-medium text-slate-600 hover:bg-slate-100 transition-colors">Batal</button>
                <button id="confirm-ok-btn" class="px-4 py-2.5 rounded-xl text-sm font-medium bg-red-600 text-white hover:bg-red-700 transition-colors shadow-sm">Ya, Lanjutkan</button>
            </div>
        </div>
    </div>

    <!-- Layout Utama -->
    <div class="min-h-screen flex flex-col md:flex-row">
        
        <!-- Sidebar Navigasi (Desktop) & Header (Mobile) -->
        <aside class="w-full md:w-64 bg-white border-b md:border-b-0 md:border-r border-slate-200 flex-shrink-0 z-30 sticky top-0 md:h-screen flex flex-col justify-between">
            <div>
                <!-- Brand Logo -->
                <div class="p-5 flex items-center justify-between border-b border-slate-100">
                    <div class="flex items-center gap-3">
                        <div class="w-10 h-10 rounded-xl bg-brand-600 flex items-center justify-center text-white shadow-md shadow-brand-500/20">
                            <i data-lucide="graduation-cap" class="w-6 h-6"></i>
                        </div>
                        <div>
                            <h1 class="font-bold text-lg text-slate-900 leading-tight">PrivatKu</h1>
                            <p class="text-xs text-slate-500 font-medium">Admin & Evaluasi Les</p>
                        </div>
                    </div>
                    <!-- Tombol Menu Mobile -->
                    <button id="mobile-menu-btn" class="md:hidden p-2 rounded-lg text-slate-600 hover:bg-slate-100">
                        <i data-lucide="menu" class="w-6 h-6"></i>
                    </button>
                </div>

                <!-- Menu Navigasi -->
                <nav id="sidebar-nav" class="hidden md:flex flex-col gap-1.5 p-4">
                    <button onclick="switchTab('dashboard')" data-tab="dashboard" class="nav-btn flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium transition-all bg-brand-50 text-brand-700">
                        <i data-lucide="layout-dashboard" class="w-5 h-5"></i>
                        Dashboard Ringkasan
                    </button>
                    <button onclick="switchTab('jurnal')" data-tab="jurnal" class="nav-btn flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-slate-600 hover:bg-slate-50 transition-all">
                        <i data-lucide="book-open-check" class="w-5 h-5"></i>
                        Jurnal & Kehadiran
                    </button>
                    <button onclick="switchTab('analisis')" data-tab="analisis" class="nav-btn flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-slate-600 hover:bg-slate-50 transition-all">
                        <i data-lucide="line-chart" class="w-5 h-5"></i>
                        Analisis per Mapel
                    </button>
                    <button onclick="switchTab('invoice')" data-tab="invoice" class="nav-btn flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-slate-600 hover:bg-slate-50 transition-all">
                        <i data-lucide="receipt" class="w-5 h-5"></i>
                        Rekap Hadir & Invoice
                    </button>
                    <button onclick="switchTab('siswa')" data-tab="siswa" class="nav-btn flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-slate-600 hover:bg-slate-50 transition-all">
                        <i data-lucide="users" class="w-5 h-5"></i>
                        Data Siswa & Mapel
                    </button>
                    <button onclick="switchTab('backup')" data-tab="backup" class="nav-btn flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-slate-600 hover:bg-slate-50 transition-all">
                        <i data-lucide="database-backup" class="w-5 h-5"></i>
                        Keamanan & Backup Data
                    </button>
                </nav>
            </div>

            <!-- Status Penyimpanan Otomatis -->
            <div class="hidden md:block p-4 m-4 bg-emerald-50/80 border border-emerald-200/80 rounded-xl">
                <div class="flex items-center gap-2 text-emerald-700 font-semibold text-xs mb-1">
                    <span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
                    Penyimpanan Permanen Aktif
                </div>
                <p class="text-[11px] text-emerald-600 leading-relaxed">
                    Semua perubahan otomatis tersimpan di perangkat ini. Gunakan menu Backup untuk cadangan file.
                </p>
            </div>
        </aside>

        <!-- Konten Utama -->
        <main class="flex-1 p-4 sm:p-6 lg:p-8 max-w-7xl mx-auto w-full overflow-x-hidden">
            
            <!-- ==================== TAB 1: DASHBOARD ==================== -->
            <section id="tab-dashboard" class="tab-content space-y-6">
                <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                    <div>
                        <h2 class="text-2xl font-bold text-slate-900">Ringkasan Administrasi Les</h2>
                        <p class="text-sm text-slate-500 mt-1">Pantau jadwal, pertemuan selesai, dan perkembangan belajar siswa Anda.</p>
                    </div>
                    <button onclick="switchTab('jurnal'); openJournalModal();" class="inline-flex items-center justify-center gap-2 bg-brand-600 hover:bg-brand-700 text-white px-4 py-2.5 rounded-xl text-sm font-medium shadow-sm transition-all">
                        <i data-lucide="plus-circle" class="w-4 h-4"></i>
                        Catat Pertemuan Baru
                    </button>
                </div>

                <!-- Kartu Statistik -->
                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                    <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm">
                        <div class="flex items-center justify-between">
                            <span class="text-sm font-medium text-slate-500">Total Siswa Aktif</span>
                            <div class="p-2.5 bg-blue-50 text-blue-600 rounded-xl"><i data-lucide="users" class="w-5 h-5"></i></div>
                        </div>
                        <p id="stat-total-students" class="text-3xl font-bold text-slate-900 mt-3">0</p>
                        <p class="text-xs text-slate-400 mt-1">Siswa terdaftar</p>
                    </div>

                    <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm">
                        <div class="flex items-center justify-between">
                            <span class="text-sm font-medium text-slate-500">Pertemuan Bulan Ini</span>
                            <div class="p-2.5 bg-emerald-50 text-emerald-600 rounded-xl"><i data-lucide="calendar-check" class="w-5 h-5"></i></div>
                        </div>
                        <p id="stat-month-sessions" class="text-3xl font-bold text-slate-900 mt-3">0</p>
                        <p class="text-xs text-emerald-600 mt-1 font-medium">Sesi hadir tercatat</p>
                    </div>

                    <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm">
                        <div class="flex items-center justify-between">
                            <span class="text-sm font-medium text-slate-500">Estimasi Pendapatan</span>
                            <div class="p-2.5 bg-violet-50 text-violet-600 rounded-xl"><i data-lucide="wallet" class="w-5 h-5"></i></div>
                        </div>
                        <p id="stat-month-income" class="text-2xl font-bold text-slate-900 mt-3">Rp 0</p>
                        <p class="text-xs text-slate-400 mt-1">Bulan berjalan (sesi hadir)</p>
                    </div>

                    <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm">
                        <div class="flex items-center justify-between">
                            <span class="text-sm font-medium text-slate-500">Rata-rata Pemahaman</span>
                            <div class="p-2.5 bg-amber-50 text-amber-600 rounded-xl"><i data-lucide="award" class="w-5 h-5"></i></div>
                        </div>
                        <p id="stat-avg-score" class="text-3xl font-bold text-slate-900 mt-3">0%</p>
                        <p class="text-xs text-slate-400 mt-1">Dari seluruh evaluasi mapel</p>
                    </div>
                </div>

                <!-- Aktivitas Terbaru & Ringkasan per Siswa -->
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <!-- Jurnal Terbaru -->
                    <div class="lg:col-span-2 bg-white rounded-2xl border border-slate-200/80 shadow-sm overflow-hidden">
                        <div class="p-5 border-b border-slate-100 flex items-center justify-between">
                            <h3 class="font-bold text-slate-900">Catatan Pertemuan Terbaru</h3>
                            <button onclick="switchTab('jurnal')" class="text-xs font-semibold text-brand-600 hover:text-brand-700">Lihat Semua &rarr;</button>
                        </div>
                        <div id="dashboard-recent-journals" class="divide-y divide-slate-100">
                            <!-- Diisi via JS -->
                        </div>
                    </div>

                    <!-- Daftar Siswa & Mapel -->
                    <div class="bg-white rounded-2xl border border-slate-200/80 shadow-sm overflow-hidden">
                        <div class="p-5 border-b border-slate-100 flex items-center justify-between">
                            <h3 class="font-bold text-slate-900">Siswa & Mapel Diambil</h3>
                            <button onclick="switchTab('siswa')" class="text-xs font-semibold text-brand-600 hover:text-brand-700">Kelola &rarr;</button>
                        </div>
                        <div id="dashboard-student-list" class="p-4 space-y-3">
                            <!-- Diisi via JS -->
                        </div>
                    </div>
                </div>
            </section>


            <!-- ==================== TAB 2: JURNAL HARIAN & ABSENSI ==================== -->
            <section id="tab-jurnal" class="tab-content hidden space-y-6">
                <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                    <div>
                        <h2 class="text-2xl font-bold text-slate-900">Jurnal Harian & Kehadiran Les</h2>
                        <p class="text-sm text-slate-500 mt-1">Catat waktu les, materi, kendala anak, dan status kehadiran terpisah per mata pelajaran.</p>
                    </div>
                    <button onclick="openJournalModal()" class="inline-flex items-center justify-center gap-2 bg-brand-600 hover:bg-brand-700 text-white px-4 py-2.5 rounded-xl text-sm font-medium shadow-sm transition-all">
                        <i data-lucide="plus" class="w-4 h-4"></i>
                        Tambah Catatan Les
                    </button>
                </div>

                <!-- Filter Jurnal -->
                <div class="bg-white p-4 rounded-2xl border border-slate-200/80 shadow-sm grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1">Filter Nama Anak</label>
                        <select id="filter-journal-student" onchange="renderJournals()" class="w-full rounded-xl border border-slate-200 px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                            <option value="ALL">Semua Siswa</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1">Filter Mata Pelajaran</label>
                        <select id="filter-journal-subject" onchange="renderJournals()" class="w-full rounded-xl border border-slate-200 px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                            <option value="ALL">Semua Mata Pelajaran</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1">Filter Bulan</label>
                        <input type="month" id="filter-journal-month" onchange="renderJournals()" class="w-full rounded-xl border border-slate-200 px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1">Status Kehadiran</label>
                        <select id="filter-journal-status" onchange="renderJournals()" class="w-full rounded-xl border border-slate-200 px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                            <option value="ALL">Semua Status</option>
                            <option value="Hadir">Hadir</option>
                            <option value="Izin">Izin</option>
                            <option value="Sakit">Sakit</option>
                            <option value="Alpha">Alpha</option>
                        </select>
                    </div>
                </div>

                <!-- Daftar Kartu/Tabel Jurnal -->
                <div id="journal-list-container" class="space-y-4">
                    <!-- Diisi secara dinamis melalui JavaScript -->
                </div>
            </section>


            <!-- ==================== TAB 3: ANALISIS PEMBELAJARAN PER MAPEL ==================== -->
            <section id="tab-analisis" class="tab-content hidden space-y-6">
                <div>
                    <h2 class="text-2xl font-bold text-slate-900">Analisis Pembelajaran Anak per Mata Pelajaran</h2>
                    <p class="text-sm text-slate-500 mt-1">Evaluasi mendalam setiap mata pelajaran yang ditempuh anak secara terpisah.</p>
                </div>

                <!-- Pemilih Siswa & Tab Mapel -->
                <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm space-y-4">
                    <div class="flex flex-col sm:flex-row sm:items-center gap-4">
                        <div class="sm:w-72">
                            <label class="block text-xs font-semibold text-slate-500 uppercase tracking-wider mb-1.5">Pilih Nama Siswa</label>
                            <select id="analysis-student-select" onchange="handleAnalysisStudentChange()" class="w-full rounded-xl border border-slate-300 bg-slate-50 px-3.5 py-2.5 text-sm font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-brand-500">
                            </select>
                        </div>
                        <div class="flex-1">
                            <label class="block text-xs font-semibold text-slate-500 uppercase tracking-wider mb-1.5">Pilih Mata Pelajaran (Terpisah)</label>
                            <div id="analysis-subject-tabs" class="flex flex-wrap gap-2">
                                <!-- Tombol tab masing-masing mapel yang diambil siswa -->
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Konten Analisis Mapel Terpilih -->
                <div id="analysis-content-area" class="space-y-6">
                    <!-- Ringkasan Metrik Mapel -->
                    <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                        <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm">
                            <span class="text-xs font-semibold text-slate-400 uppercase">Tingkat Penguasaan Materi</span>
                            <div class="flex items-end gap-3 mt-2">
                                <span id="ana-avg-mastery" class="text-3xl font-bold text-brand-600">0%</span>
                                <span id="ana-mastery-badge" class="px-2.5 py-0.5 rounded-full text-xs font-medium bg-slate-100 text-slate-600 mb-1">-</span>
                            </div>
                            <div class="w-full bg-slate-100 h-2 rounded-full mt-3 overflow-hidden">
                                <div id="ana-mastery-bar" class="bg-brand-600 h-full rounded-full transition-all duration-500" style="width: 0%"></div>
                            </div>
                        </div>

                        <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm">
                            <span class="text-xs font-semibold text-slate-400 uppercase">Total Sesi Mapel Ini</span>
                            <div class="flex items-end gap-2 mt-2">
                                <span id="ana-total-sessions" class="text-3xl font-bold text-slate-900">0</span>
                                <span class="text-sm text-slate-500 mb-1">pertemuan hadir</span>
                            </div>
                            <p id="ana-last-date" class="text-xs text-slate-400 mt-3">Pertemuan terakhir: -</p>
                        </div>

                        <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm">
                            <span class="text-xs font-semibold text-slate-400 uppercase">Status & Fokus Mapel</span>
                            <p id="ana-focus-status" class="text-base font-bold text-slate-800 mt-2">Belum ada data evaluasi</p>
                            <p id="ana-focus-desc" class="text-xs text-slate-500 mt-1">Tambahkan jurnal les pada mapel ini untuk melihat analisis.</p>
                        </div>
                    </div>

                    <!-- Grafik Perkembangan & Rincian Kendala/Materi -->
                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                        <!-- Grafik Perkembangan Pemahaman -->
                        <div class="lg:col-span-2 bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm">
                            <div class="flex items-center justify-between mb-4">
                                <div>
                                    <h3 class="font-bold text-slate-900">Grafik Perkembangan Pemahaman</h3>
                                    <p id="ana-chart-subtitle" class="text-xs text-slate-500">Skor pemahaman tiap pertemuan pada mata pelajaran terpilih</p>
                                </div>
                            </div>
                            <div class="relative h-64 w-full">
                                <canvas id="subjectProgressChart"></canvas>
                            </div>
                        </div>

                        <!-- Catatan Kesimpulan & Rekomendasi Guru per Mapel -->
                        <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm flex flex-col justify-between">
                            <div>
                                <div class="flex items-center gap-2 text-brand-600 font-bold text-sm mb-2">
                                    <i data-lucide="sparkles" class="w-4 h-4"></i>
                                    Catatan Evaluasi & Target Mapel
                                </div>
                                <p class="text-xs text-slate-500 mb-3">Tulis ringkasan analisis khusus Anda untuk perkembangan anak pada mapel ini (tersimpan otomatis per siswa & mapel):</p>
                                <textarea id="ana-teacher-summary" rows="6" placeholder="Contoh: Ananda sudah lancar perkalian dasar, namun perlu latihan soal cerita pecahan. Target bulan depan menguasai FPB & KPK..." class="w-full rounded-xl border border-slate-200 p-3 text-sm text-slate-700 focus:outline-none focus:ring-2 focus:ring-brand-500"></textarea>
                            </div>
                            <button onclick="saveSubjectAnalysisNote()" class="mt-3 w-full bg-brand-600 hover:bg-brand-700 text-white py-2.5 rounded-xl text-sm font-medium transition-colors flex items-center justify-center gap-2">
                                <i data-lucide="save" class="w-4 h-4"></i>
                                Simpan Analisis Mapel Ini
                            </button>
                        </div>
                    </div>

                    <!-- Kompilasi Materi & Kendala Spesifik Mapel -->
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm">
                            <h3 class="font-bold text-slate-900 flex items-center gap-2 mb-4">
                                <span class="p-1.5 bg-emerald-100 text-emerald-700 rounded-lg"><i data-lucide="check-circle-2" class="w-4 h-4"></i></span>
                                Riwayat Materi yang Telah Dipelajari
                            </h3>
                            <ul id="ana-materials-list" class="space-y-2.5 text-sm">
                                <!-- Diisi JS -->
                            </ul>
                        </div>

                        <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm">
                            <h3 class="font-bold text-slate-900 flex items-center gap-2 mb-4">
                                <span class="p-1.5 bg-amber-100 text-amber-700 rounded-lg"><i data-lucide="alert-circle" class="w-4 h-4"></i></span>
                                Daftar Kendala Belajar pada Mapel Ini
                            </h3>
                            <ul id="ana-obstacles-list" class="space-y-2.5 text-sm">
                                <!-- Diisi JS -->
                            </ul>
                        </div>
                    </div>
                </div>
            </section>


            <!-- ==================== TAB 4: REKAP KEHADIRAN & INVOICE ==================== -->
            <section id="tab-invoice" class="tab-content hidden space-y-6">
                <div>
                    <h2 class="text-2xl font-bold text-slate-900">Rekap Kehadiran & Generator Invoice</h2>
                    <p class="text-sm text-slate-500 mt-1">Rekap otomatis absensi anak per mata pelajaran dan cetak tagihan (invoice) untuk orang tua.</p>
                </div>

                <!-- Filter Periode & Siswa -->
                <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm grid grid-cols-1 sm:grid-cols-3 gap-4 items-end">
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1.5">Pilih Siswa</label>
                        <select id="invoice-student-select" onchange="renderAttendanceAndInvoice()" class="w-full rounded-xl border border-slate-300 px-3.5 py-2.5 text-sm font-medium focus:outline-none focus:ring-2 focus:ring-brand-500">
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1.5">Periode Bulan Tagihan</label>
                        <input type="month" id="invoice-month-select" onchange="renderAttendanceAndInvoice()" class="w-full rounded-xl border border-slate-300 px-3.5 py-2.5 text-sm font-medium focus:outline-none focus:ring-2 focus:ring-brand-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-500 mb-1.5">Filter Mapel untuk Tagihan</label>
                        <select id="invoice-subject-filter" onchange="renderAttendanceAndInvoice()" class="w-full rounded-xl border border-slate-300 px-3.5 py-2.5 text-sm font-medium focus:outline-none focus:ring-2 focus:ring-brand-500">
                            <option value="ALL">Semua Mapel (Gabungan Rinci)</option>
                        </select>
                    </div>
                </div>

                <!-- Tabel Rekap Kehadiran per Mata Pelajaran -->
                <div class="bg-white rounded-2xl border border-slate-200/80 shadow-sm overflow-hidden">
                    <div class="p-5 border-b border-slate-100">
                        <h3 class="font-bold text-slate-900">Rekapitulasi Kehadiran per Mata Pelajaran</h3>
                        <p class="text-xs text-slate-500">Hanya sesi berstatus <span class="font-semibold text-emerald-600">Hadir</span> yang dihitung ke dalam tagihan invoice.</p>
                    </div>
                    <div class="overflow-x-auto">
                        <table class="w-full text-left border-collapse">
                            <thead>
                                <tr class="bg-slate-50 text-slate-500 text-xs uppercase tracking-wider border-b border-slate-200">
                                    <th class="py-3.5 px-5 font-semibold">Mata Pelajaran</th>
                                    <th class="py-3.5 px-4 font-semibold text-center">Hadir</th>
                                    <th class="py-3.5 px-4 font-semibold text-center">Izin</th>
                                    <th class="py-3.5 px-4 font-semibold text-center">Sakit</th>
                                    <th class="py-3.5 px-4 font-semibold text-center">Alpha</th>
                                    <th class="py-3.5 px-5 font-semibold text-right">Tarif / Sesi</th>
                                    <th class="py-3.5 px-5 font-semibold text-right">Subtotal Tagihan</th>
                                </tr>
                            </thead>
                            <tbody id="attendance-recap-tbody" class="divide-y divide-slate-100 text-sm">
                                <!-- Diisi via JS -->
                            </tbody>
                        </table>
                    </div>
                </div>

                <!-- Pratinjau Dokumen Invoice Siap Cetak -->
                <div class="space-y-3">
                    <div class="flex flex-wrap items-center justify-between gap-3">
                        <h3 class="font-bold text-slate-900">Pratinjau Dokumen Invoice</h3>
                        <div class="flex flex-wrap gap-2">
                            <button onclick="copyInvoiceWhatsApp()" class="inline-flex items-center gap-2 bg-emerald-600 hover:bg-emerald-700 text-white px-4 py-2 rounded-xl text-sm font-medium shadow-sm transition-all">
                                <i data-lucide="message-circle" class="w-4 h-4"></i>
                                Salin Teks WhatsApp
                            </button>
                            <button onclick="window.print()" class="inline-flex items-center gap-2 bg-slate-800 hover:bg-slate-900 text-white px-4 py-2 rounded-xl text-sm font-medium shadow-sm transition-all">
                                <i data-lucide="printer" class="w-4 h-4"></i>
                                Cetak / Simpan PDF
                            </button>
                        </div>
                    </div>

                    <!-- Kartu Invoice -->
                    <div id="printable-invoice" class="bg-white p-6 sm:p-10 rounded-2xl border border-slate-200 shadow-sm max-w-4xl mx-auto">
                        <!-- Header Invoice -->
                        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center border-b border-slate-200 pb-6 gap-4">
                            <div>
                                <div class="flex items-center gap-2.5 text-brand-600 font-bold text-xl">
                                    <i data-lucide="graduation-cap" class="w-7 h-7"></i>
                                    <span id="inv-tutor-brand">Bimbingan Les Privat</span>
                                </div>
                                <p id="inv-tutor-sub" class="text-xs text-slate-500 mt-1">Laporan Kehadiran & Rincian Biaya Pembelajaran</p>
                            </div>
                            <div class="sm:text-right">
                                <span class="inline-block px-3 py-1 bg-brand-50 text-brand-700 font-bold text-xs rounded-lg uppercase tracking-wider">INVOICE TAGIHAN</span>
                                <p id="inv-number" class="text-sm font-bold text-slate-800 mt-1.5">INV/2026/001</p>
                                <p id="inv-date" class="text-xs text-slate-500">Tanggal Cetak: -</p>
                            </div>
                        </div>

                        <!-- Info Siswa & Periode -->
                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-6 py-6 border-b border-slate-100 text-sm">
                            <div>
                                <p class="text-xs font-semibold text-slate-400 uppercase mb-1">Ditagihkan Kepada (Orang Tua / Siswa):</p>
                                <p id="inv-student-name" class="font-bold text-base text-slate-900">-</p>
                                <p id="inv-student-grade" class="text-slate-600">-</p>
                                <p id="inv-student-phone" class="text-slate-500 text-xs mt-0.5">-</p>
                            </div>
                            <div class="sm:text-right">
                                <p class="text-xs font-semibold text-slate-400 uppercase mb-1">Periode Pembelajaran:</p>
                                <p id="inv-period" class="font-bold text-base text-slate-900">-</p>
                                <p class="text-xs text-slate-500 mt-1">Status: <span class="text-amber-600 font-semibold">Menunggu Pembayaran</span></p>
                            </div>
                        </div>

                        <!-- Rincian Sesi per Mapel -->
                        <div class="py-6">
                            <h4 class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-3">Rincian Pertemuan per Mata Pelajaran</h4>
                            <div class="overflow-x-auto">
                                <table class="w-full text-left border-collapse text-sm">
                                    <thead>
                                        <tr class="border-b-2 border-slate-200 text-slate-600 text-xs">
                                            <th class="py-2.5 font-bold">Mata Pelajaran</th>
                                            <th class="py-2.5 font-bold">Tanggal Pertemuan (Hadir)</th>
                                            <th class="py-2.5 font-bold text-center">Jml Sesi</th>
                                            <th class="py-2.5 font-bold text-right">Biaya/Sesi</th>
                                            <th class="py-2.5 font-bold text-right">Total</th>
                                        </tr>
                                    </thead>
                                    <tbody id="inv-items-tbody" class="divide-y divide-slate-100">
                                        <!-- Diisi JS -->
                                    </tbody>
                                </table>
                            </div>
                        </div>

                        <!-- Total Akhir & Catatan Pembayaran -->
                        <div class="border-t-2 border-slate-200 pt-5 flex flex-col sm:flex-row justify-between gap-6">
                            <div class="text-xs text-slate-500 max-w-md space-y-1">
                                <p class="font-semibold text-slate-700">Catatan Pembayaran:</p>
                                <p id="inv-payment-note">Pembayaran dapat dilakukan secara tunai atau transfer. Terima kasih atas kepercayaan Bapak/Ibu.</p>
                            </div>
                            <div class="bg-slate-50 p-4 rounded-xl min-w-[240px] text-right">
                                <p class="text-xs text-slate-500 font-medium">Total Tagihan Keseluruhan</p>
                                <p id="inv-grand-total" class="text-2xl font-bold text-brand-600 mt-1">Rp 0</p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>


            <!-- ==================== TAB 5: DATA SISWA & MAPEL ==================== -->
            <section id="tab-siswa" class="tab-content hidden space-y-6">
                <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                    <div>
                        <h2 class="text-2xl font-bold text-slate-900">Manajemen Siswa & Mata Pelajaran</h2>
                        <p class="text-sm text-slate-500 mt-1">Atur profil siswa, tarif per pertemuan, dan daftar mata pelajaran yang diambil masing-masing anak.</p>
                    </div>
                    <button onclick="openStudentModal()" class="inline-flex items-center justify-center gap-2 bg-brand-600 hover:bg-brand-700 text-white px-4 py-2.5 rounded-xl text-sm font-medium shadow-sm transition-all">
                        <i data-lucide="user-plus" class="w-4 h-4"></i>
                        Tambah Siswa Baru
                    </button>
                </div>

                <!-- Grid Kartu Siswa -->
                <div id="students-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-5">
                    <!-- Diisi via JS -->
                </div>
            </section>


            <!-- ==================== TAB 6: KEAMANAN & BACKUP DATA ==================== -->
            <section id="tab-backup" class="tab-content hidden space-y-6">
                <div>
                    <h2 class="text-2xl font-bold text-slate-900">Keamanan, Pengaturan & Backup Data</h2>
                    <p class="text-sm text-slate-500 mt-1">Jaga agar data administrasi les Anda tidak pernah hilang meskipun berganti perangkat atau membersihkan browser.</p>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <!-- Pengaturan Identitas Pengajar (Untuk Invoice) -->
                    <div class="bg-white p-6 rounded-2xl border border-slate-200/80 shadow-sm space-y-4">
                        <div class="flex items-center gap-3 text-brand-600">
                            <div class="p-2.5 bg-brand-50 rounded-xl"><i data-lucide="settings" class="w-5 h-5"></i></div>
                            <h3 class="font-bold text-slate-900">Pengaturan Identitas & Invoice</h3>
                        </div>
                        <div class="space-y-3">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Bimbingan / Pengajar</label>
                                <input type="text" id="setting-tutor-name" placeholder="Contoh: Les Privat Kak Budi" class="w-full rounded-xl border border-slate-200 px-3.5 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Informasi Rekening / Catatan di Invoice</label>
                                <textarea id="setting-payment-note" rows="3" placeholder="Contoh: Transfer BCA 1234567890 a.n Budi Santoso" class="w-full rounded-xl border border-slate-200 px-3.5 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500"></textarea>
                            </div>
                            <button onclick="saveSettings()" class="bg-brand-600 hover:bg-brand-700 text-white px-4 py-2.5 rounded-xl text-sm font-medium transition-colors">
                                Simpan Pengaturan
                            </button>
                        </div>
                    </div>

                    <!-- Ekspor & Impor Data -->
                    <div class="bg-white p-6 rounded-2xl border border-slate-200/80 shadow-sm space-y-5">
                        <div class="flex items-center gap-3 text-emerald-600">
                            <div class="p-2.5 bg-emerald-50 rounded-xl"><i data-lucide="shield-check" class="w-5 h-5"></i></div>
                            <h3 class="font-bold text-slate-900">Cadangkan & Pulihkan Data (JSON)</h3>
                        </div>
                        <p class="text-xs text-slate-600 leading-relaxed">
                            Data Anda secara otomatis tersimpan di penyimpanan internal browser perangkat ini. Untuk keamanan ekstra, unduh file cadangan (Backup) secara berkala agar bisa dipulihkan kapan saja.
                        </p>
                        
                        <div class="flex flex-col sm:flex-row gap-3">
                            <button onclick="exportBackupData()" class="flex-1 inline-flex items-center justify-center gap-2 bg-emerald-600 hover:bg-emerald-700 text-white px-4 py-2.5 rounded-xl text-sm font-medium transition-colors">
                                <i data-lucide="download" class="w-4 h-4"></i>
                                Unduh Backup Data
                            </button>
                            <label class="flex-1 inline-flex items-center justify-center gap-2 bg-slate-100 hover:bg-slate-200 text-slate-800 px-4 py-2.5 rounded-xl text-sm font-medium cursor-pointer transition-colors">
                                <i data-lucide="upload" class="w-4 h-4"></i>
                                Pulihkan dari File
                                <input type="file" id="import-file-input" accept=".json" onchange="importBackupData(event)" class="hidden">
                            </label>
                        </div>

                        <div class="pt-4 border-t border-slate-100 flex items-center justify-between">
                            <span class="text-xs text-slate-400">Hapus contoh data awal / mulai dari kosong:</span>
                            <button onclick="resetAllDataConfirm()" class="text-xs font-semibold text-red-600 hover:text-red-700 underline">
                                Reset Semua Data
                            </button>
                        </div>
                    </div>
                </div>
            </section>

        </main>
    </div>


    <!-- ==================== MODAL TAMBAH / EDIT SISWA ==================== -->
    <div id="student-modal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 hidden items-center justify-center p-4 overflow-y-auto">
        <div class="bg-white rounded-2xl max-w-lg w-full p-6 shadow-xl border border-slate-100 my-8">
            <div class="flex items-center justify-between pb-4 border-b border-slate-100">
                <h3 id="student-modal-title" class="text-lg font-bold text-slate-900">Tambah Siswa Baru</h3>
                <button onclick="closeStudentModal()" class="p-1.5 text-slate-400 hover:text-slate-600 rounded-lg"><i data-lucide="x" class="w-5 h-5"></i></button>
            </div>
            <form id="student-form" onsubmit="saveStudent(event)" class="space-y-4 mt-4">
                <input type="hidden" id="student-id">
                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">Nama Lengkap Anak *</label>
                    <input type="text" id="student-name" required placeholder="Contoh: Raka Pratama" class="w-full rounded-xl border border-slate-300 px-3.5 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">Kelas / Jenjang *</label>
                        <input type="text" id="student-grade" required placeholder="Contoh: Kelas 5 SD" class="w-full rounded-xl border border-slate-300 px-3.5 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">No. WA Orang Tua</label>
                        <input type="text" id="student-phone" placeholder="Contoh: 08123456789" class="w-full rounded-xl border border-slate-300 px-3.5 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                    </div>
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">Tarif per Pertemuan (Rp) *</label>
                    <input type="number" id="student-fee" required min="0" step="1000" placeholder="Contoh: 50000" class="w-full rounded-xl border border-slate-300 px-3.5 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1.5">Mata Pelajaran yang Ditempuh (Pisahkan setiap mapel) *</label>
                    <div id="subject-checkboxes" class="grid grid-cols-2 gap-2 mb-2">
                        <!-- Pilihan mapel standar -->
                    </div>
                    <!-- Tambah Mapel Custom -->
                    <div class="flex gap-2 mt-2">
                        <input type="text" id="custom-subject-input" placeholder="Tambah mapel lain (misal: Calistung)" class="flex-1 rounded-xl border border-slate-300 px-3 py-2 text-xs focus:outline-none focus:ring-2 focus:ring-brand-500">
                        <button type="button" onclick="addCustomSubjectToModal()" class="px-3 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-semibold">
                            + Tambah
                        </button>
                    </div>
                </div>
                <div class="flex justify-end gap-3 pt-4 border-t border-slate-100">
                    <button type="button" onclick="closeStudentModal()" class="px-4 py-2.5 rounded-xl text-sm font-medium text-slate-600 hover:bg-slate-100">Batal</button>
                    <button type="submit" class="px-5 py-2.5 rounded-xl text-sm font-medium bg-brand-600 text-white hover:bg-brand-700 shadow-sm">Simpan Data Siswa</button>
                </div>
            </form>
        </div>
    </div>


    <!-- ==================== MODAL TAMBAH / EDIT JURNAL PERTEMUAN ==================== -->
    <div id="journal-modal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 hidden items-center justify-center p-4 overflow-y-auto">
        <div class="bg-white rounded-2xl max-w-xl w-full p-6 shadow-xl border border-slate-100 my-8">
            <div class="flex items-center justify-between pb-4 border-b border-slate-100">
                <h3 id="journal-modal-title" class="text-lg font-bold text-slate-900">Catat Pertemuan Les Privat</h3>
                <button onclick="closeJournalModal()" class="p-1.5 text-slate-400 hover:text-slate-600 rounded-lg"><i data-lucide="x" class="w-5 h-5"></i></button>
            </div>
            <form id="journal-form" onsubmit="saveJournal(event)" class="space-y-4 mt-4">
                <input type="hidden" id="journal-id">
                
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">Nama Anak *</label>
                        <select id="journal-student" required onchange="updateJournalSubjectOptions()" class="w-full rounded-xl border border-slate-300 px-3.5 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">Mata Pelajaran (Terpisah) *</label>
                        <select id="journal-subject" required class="w-full rounded-xl border border-slate-300 px-3.5 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                        </select>
                    </div>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">Tanggal Les *</label>
                        <input type="date" id="journal-date" required class="w-full rounded-xl border border-slate-300 px-3 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">Waktu / Jam *</label>
                        <input type="text" id="journal-time" required placeholder="16.00 - 17.30" class="w-full rounded-xl border border-slate-300 px-3 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 mb-1">Kehadiran *</label>
                        <select id="journal-status" required class="w-full rounded-xl border border-slate-300 px-3 py-2.5 text-sm font-medium focus:outline-none focus:ring-2 focus:ring-brand-500">
                            <option value="Hadir">Hadir</option>
                            <option value="Izin">Izin</option>
                            <option value="Sakit">Sakit</option>
                            <option value="Alpha">Alpha</option>
                        </select>
                    </div>
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">Apa Materi yang Diajarkan? *</label>
                    <input type="text" id="journal-material" required placeholder="Contoh: Operasi Hitung Pecahan Campuran & Soal Cerita" class="w-full rounded-xl border border-slate-300 px-3.5 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500">
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">Apa Kendala Anak saat Belajar?</label>
                    <textarea id="journal-obstacle" rows="2" placeholder="Contoh: Masih sering tertukar saat menyamakan penyebut, atau kurang teliti menghitung." class="w-full rounded-xl border border-slate-300 px-3.5 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500"></textarea>
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">Keterangan / Catatan Perkembangan Anak</label>
                    <textarea id="journal-notes" rows="2" placeholder="Contoh: Ananda sangat antusias hari ini, mampu menyelesaikan 8 dari 10 latihan soal dengan benar." class="w-full rounded-xl border border-slate-300 px-3.5 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-brand-500"></textarea>
                </div>

                <!-- Skor Pemahaman untuk Grafik Analisis per Mapel -->
                <div class="bg-slate-50 p-3.5 rounded-xl border border-slate-200">
                    <div class="flex justify-between items-center mb-1">
                        <label class="text-xs font-bold text-slate-700">Tingkat Pemahaman Materi (Untuk Analisis Mapel)</label>
                        <span id="mastery-score-display" class="text-sm font-bold text-brand-600">80%</span>
                    </div>
                    <input type="range" id="journal-score" min="0" max="100" step="5" value="80" oninput="document.getElementById('mastery-score-display').textContent = this.value + '%'" class="w-full accent-brand-600 cursor-pointer">
                    <div class="flex justify-between text-[10px] text-slate-400 mt-1">
                        <span>0% (Perlu Bimbingan Ekstra)</span>
                        <span>50% (Cukup Paham)</span>
                        <span>100% (Sangat Menguasai)</span>
                    </div>
                </div>

                <div class="flex justify-end gap-3 pt-3 border-t border-slate-100">
                    <button type="button" onclick="closeJournalModal()" class="px-4 py-2.5 rounded-xl text-sm font-medium text-slate-600 hover:bg-slate-100">Batal</button>
                    <button type="submit" class="px-5 py-2.5 rounded-xl text-sm font-medium bg-brand-600 text-white hover:bg-brand-700 shadow-sm">Simpan Jurnal</button>
                </div>
            </form>
        </div>
    </div>

    <!-- ==================== SCRIPT LOGIKA APLIKASI ==================== -->
    <script>
        // Kunci Penyimpanan Permanen di LocalStorage
        const STORAGE_KEY = 'privatku_admin_data_v1';

        // Daftar Mata Pelajaran Standar
        const DEFAULT_SUBJECTS = [
            'Matematika',
            'Bahasa Indonesia',
            'IPA (Sains)',
            'Bahasa Inggris',
            'IPS',
            'Bahasa Jawa',
            'Calistung (Baca Tulis Hitung)'
        ];

        // State Utama Aplikasi
        let appData = {
            settings: {
                tutorName: 'Bimbingan Les Privat Mandiri',
                paymentNote: 'Pembayaran dapat dilakukan tunai atau transfer bank. Terima kasih atas kepercayaan Bapak/Ibu.'
            },
            students: [],
            journals: [],
            subjectNotes: {} // Format key: "studentId_subjectName" -> catatan analisis guru
        };

        let progressChartInstance = null;
        let currentAnalysisStudentId = null;
        let currentAnalysisSubject = null;

        // ==================== INISIALISASI & PENYIMPANAN PERMANEN ====================
        function initApp() {
            loadDataFromStorage();
            
            // Set bulan berjalan di filter
            const now = new Date();
            const currentMonthStr = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}`;
            document.getElementById('filter-journal-month').value = currentMonthStr;
            document.getElementById('invoice-month-select').value = currentMonthStr;

            // Load pengaturan ke form
            document.getElementById('setting-tutor-name').value = appData.settings.tutorName || '';
            document.getElementById('setting-payment-note').value = appData.settings.paymentNote || '';

            // Event Listener Menu Mobile
            document.getElementById('mobile-menu-btn').addEventListener('click', () => {
                const nav = document.getElementById('sidebar-nav');
                nav.classList.toggle('hidden');
                nav.classList.toggle('flex');
            });

            refreshAllViews();
            lucide.createIcons();
        }

        function loadDataFromStorage() {
            try {
                const saved = localStorage.getItem(STORAGE_KEY);
                if (saved) {
                    appData = JSON.parse(saved);
                    if (!appData.subjectNotes) appData.subjectNotes = {};
                    if (!appData.settings) appData.settings = { tutorName: 'Bimbingan Les Privat Mandiri', paymentNote: '' };
                } else {
                    // Data contoh awal agar pengguna langsung memahami cara kerja aplikasi
                    seedSampleData();
                    saveDataToStorage();
                }
            } catch (e) {
                console.error("Gagal memuat data:", e);
                showToast("Terjadi kesalahan saat membaca penyimpanan lokal.", "error");
            }
        }

        function saveDataToStorage() {
            try {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(appData));
            } catch (e) {
                console.error("Gagal menyimpan data:", e);
                showToast("Gagal menyimpan data ke browser!", "error");
            }
        }

        function seedSampleData() {
            const now = new Date();
            const year = now.getFullYear();
            const month = String(now.getMonth() + 1).padStart(2, '0');

            appData.students = [
                {
                    id: 'std-1',
                    name: 'Nadia Putri',
                    grade: 'Kelas 5 SD',
                    phone: '081234567890',
                    fee: 50000,
                    subjects: ['Matematika', 'Bahasa Inggris', 'IPA (Sains)']
                },
                {
                    id: 'std-2',
                    name: 'Arkananta Gilang',
                    grade: 'Kelas 8 SMP',
                    phone: '085712345678',
                    fee: 65000,
                    subjects: ['Matematika', 'IPA (Sains)']
                }
            ];

            appData.journals = [
                {
                    id: 'jrn-1',
                    studentId: 'std-1',
                    subject: 'Matematika',
                    date: `${year}-${month}-03`,
                    time: '15.30 - 17.00',
                    status: 'Hadir',
                    material: 'Penjumlahan dan Pengurangan Pecahan Berbeda Penyebut',
                    obstacle: 'Masih sedikit bingung mencari KPK untuk menyamakan penyebut.',
                    notes: 'Nadia fokus menyimak, setelah diberikan trik tabel perkalian mulai bisa mengerjakan mandiri.',
                    score: 70
                },
                {
                    id: 'jrn-2',
                    studentId: 'std-1',
                    subject: 'Matematika',
                    date: `${year}-${month}-10`,
                    time: '15.30 - 17.00',
                    status: 'Hadir',
                    material: 'Perkalian dan Pembagian Pecahan Campuran',
                    obstacle: 'Terkadang lupa mengubah pecahan campuran ke pecahan biasa terlebih dahulu.',
                    notes: 'Perkembangan sangat baik! Dari 10 soal latihan, 9 soal dijawab benar.',
                    score: 85
                },
                {
                    id: 'jrn-3',
                    studentId: 'std-1',
                    subject: 'Bahasa Inggris',
                    date: `${year}-${month}-05`,
                    time: '15.30 - 17.00',
                    status: 'Hadir',
                    material: 'Simple Present Tense & Daily Activities Vocabulary',
                    obstacle: 'Penggunaan akhiran -s/-es pada subjek He, She, It masih sering terlewat.',
                    notes: 'Pelafalan (pronunciation) kosakata bahasa Inggris Nadia sangat bagus dan percaya diri.',
                    score: 80
                },
                {
                    id: 'jrn-4',
                    studentId: 'std-2',
                    subject: 'IPA (Sains)',
                    date: `${year}-${month}-06`,
                    time: '18.30 - 20.00',
                    status: 'Hadir',
                    material: 'Gaya, Gerak, dan Hukum Newton I, II, III',
                    obstacle: 'Kesulitan mengidentifikasi gaya aksi-reaksi pada soal cerita.',
                    notes: 'Sudah paham rumus F = m.a, pertemuan berikutnya akan latihan soal hitungan.',
                    score: 75
                }
            ];

            appData.subjectNotes = {
                'std-1_Matematika': 'Nadia memiliki daya tangkap yang cepat pada konsep visual. Fokus bulan ini adalah memantapkan ketelitian dalam soal cerita pecahan agar siap menghadapi ulangan harian.'
            };
        }

        // ==================== NAVIGASI TAB ====================
        function switchTab(tabId) {
            // Sembunyikan semua tab
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            // Tampilkan tab terpilih
            document.getElementById(`tab-${tabId}`).classList.remove('hidden');

            // Update gaya tombol navigasi
            document.querySelectorAll('.nav-btn').forEach(btn => {
                if (btn.dataset.tab === tabId) {
                    btn.classList.add('bg-brand-50', 'text-brand-700');
                    btn.classList.remove('text-slate-600', 'hover:bg-slate-50');
                } else {
                    btn.classList.remove('bg-brand-50', 'text-brand-700');
                    btn.classList.add('text-slate-600', 'hover:bg-slate-50');
                }
            });

            // Tutup menu mobile setelah klik
            if (window.innerWidth < 768) {
                document.getElementById('sidebar-nav').classList.add('hidden');
                document.getElementById('sidebar-nav').classList.remove('flex');
            }

            // Refresh komponen khusus pada tab tertentu
            if (tabId === 'analisis') {
                initAnalysisView();
            } else if (tabId === 'invoice') {
                initInvoiceView();
            } else if (tabId === 'dashboard') {
                renderDashboard();
            }
        }

        function refreshAllViews() {
            populateStudentDropdowns();
            renderDashboard();
            renderStudents();
            renderJournals();
            initAnalysisView();
            initInvoiceView();
            lucide.createIcons();
        }

        // ==================== UTILITAS FORMAT & NOTIFIKASI ====================
        function formatRupiah(angka) {
            return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(angka || 0);
        }

        function formatDateIndo(dateStr) {
            if (!dateStr) return '-';
            const options = { weekday: 'short', day: 'numeric', month: 'short', year: 'numeric' };
            return new Date(dateStr + 'T00:00:00').toLocaleDateString('id-ID', options);
        }

        function getStudentById(id) {
            return appData.students.find(s => s.id === id);
        }

        function showToast(message, type = 'success') {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            const bgColor = type === 'error' ? 'bg-red-600' : (type === 'info' ? 'bg-slate-800' : 'bg-emerald-600');
            toast.className = `${bgColor} text-white px-4 py-3 rounded-xl shadow-lg text-sm font-medium flex items-center gap-2.5 transform transition-all duration-300 translate-y-2 opacity-0`;
            toast.innerHTML = `<span>${message}</span>`;
            container.appendChild(toast);

            requestAnimationFrame(() => {
                toast.classList.remove('translate-y-2', 'opacity-0');
            });

            setTimeout(() => {
                toast.classList.add('opacity-0', 'translate-y-2');
                setTimeout(() => toast.remove(), 300);
            }, 3200);
        }

        function showCustomConfirm(title, message, onConfirmCallback) {
            const modal = document.getElementById('confirm-modal');
            document.getElementById('confirm-title').textContent = title;
            document.getElementById('confirm-message').textContent = message;
            modal.classList.remove('hidden');
            modal.classList.add('flex');

            const okBtn = document.getElementById('confirm-ok-btn');
            const cancelBtn = document.getElementById('confirm-cancel-btn');

            const cleanup = () => {
                modal.classList.add('hidden');
                modal.classList.remove('flex');
                okBtn.onclick = null;
                cancelBtn.onclick = null;
            };

            cancelBtn.onclick = cleanup;
            okBtn.onclick = () => {
                cleanup();
                onConfirmCallback();
            };
        }

        // ==================== POPULATE DROPDOWNS ====================
        function populateStudentDropdowns() {
            const filterStudent = document.getElementById('filter-journal-student');
            const filterSubject = document.getElementById('filter-journal-subject');
            const analysisStudent = document.getElementById('analysis-student-select');
            const invoiceStudent = document.getElementById('invoice-student-select');

            // Simpan nilai terpilih sebelumnya
            const prevFilterStd = filterStudent.value;
            const prevFilterSubj = filterSubject.value;
            const prevAnaStd = analysisStudent.value;
            const prevInvStd = invoiceStudent.value;

            filterStudent.innerHTML = '<option value="ALL">Semua Siswa</option>';
            analysisStudent.innerHTML = '';
            invoiceStudent.innerHTML = '';

            const allSubjectsSet = new Set();

            appData.students.forEach(s => {
                filterStudent.innerHTML += `<option value="${s.id}">${s.name} (${s.grade})</option>`;
                analysisStudent.innerHTML += `<option value="${s.id}">${s.name} — ${s.grade}</option>`;
                invoiceStudent.innerHTML += `<option value="${s.id}">${s.name} — ${s.grade}</option>`;
                (s.subjects || []).forEach(sub => allSubjectsSet.add(sub));
            });

            filterSubject.innerHTML = '<option value="ALL">Semua Mata Pelajaran</option>';
            allSubjectsSet.forEach(sub => {
                filterSubject.innerHTML += `<option value="${sub}">${sub}</option>`;
            });

            if (prevFilterStd) filterStudent.value = prevFilterStd;
            if (prevFilterSubj) filterSubject.value = prevFilterSubj;
            if (prevAnaStd && getStudentById(prevAnaStd)) analysisStudent.value = prevAnaStd;
            if (prevInvStd && getStudentById(prevInvStd)) invoiceStudent.value = prevInvStd;
        }

        // ==================== 1. DASHBOARD ====================
        function renderDashboard() {
            const now = new Date();
            const currentMonthPrefix = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}`;

            document.getElementById('stat-total-students').textContent = appData.students.length;

            // Hitung sesi hadir bulan ini & pendapatan
            const monthPresentJournals = appData.journals.filter(j => j.date.startsWith(currentMonthPrefix) && j.status === 'Hadir');
            document.getElementById('stat-month-sessions').textContent = monthPresentJournals.length;

            let totalIncome = 0;
            monthPresentJournals.forEach(j => {
                const std = getStudentById(j.studentId);
                if (std) totalIncome += Number(std.fee || 0);
            });
            document.getElementById('stat-month-income').textContent = formatRupiah(totalIncome);

            // Rata-rata skor pemahaman seluruh sesi hadir
            const allPresent = appData.journals.filter(j => j.status === 'Hadir' && typeof j.score === 'number');
            const avgScore = allPresent.length > 0 
                ? Math.round(allPresent.reduce((acc, cur) => acc + cur.score, 0) / allPresent.length) 
                : 0;
            document.getElementById('stat-avg-score').textContent = `${avgScore}%`;

            // Jurnal Terbaru (5 terakhir)
            const recentContainer = document.getElementById('dashboard-recent-journals');
            const sortedJournals = [...appData.journals].sort((a, b) => b.date.localeCompare(a.date));

            if (sortedJournals.length === 0) {
                recentContainer.innerHTML = `<div class="p-8 text-center text-sm text-slate-400">Belum ada catatan pertemuan les.</div>`;
            } else {
                recentContainer.innerHTML = sortedJournals.slice(0, 5).map(j => {
                    const std = getStudentById(j.studentId);
                    const stdName = std ? std.name : 'Siswa Dihapus';
                    const statusBadge = getStatusBadgeHTML(j.status);
                    return `
                        <div class="p-4 hover:bg-slate-50/80 transition-colors flex flex-col sm:flex-row sm:items-center justify-between gap-2">
                            <div>
                                <div class="flex flex-wrap items-center gap-2">
                                    <span class="font-bold text-slate-900 text-sm">${stdName}</span>
                                    <span class="px-2 py-0.5 rounded-md text-xs font-semibold bg-brand-50 text-brand-700">${j.subject}</span>
                                    ${statusBadge}
                                </div>
                                <p class="text-xs text-slate-600 mt-1 line-clamp-1"><span class="font-medium">Materi:</span> ${j.material}</p>
                            </div>
                            <div class="text-xs text-slate-400 sm:text-right flex-shrink-0">
                                <p class="font-medium text-slate-600">${formatDateIndo(j.date)}</p>
                                <p>${j.time}</p>
                            </div>
                        </div>
                    `;
                }).join('');
            }

            // Daftar Siswa & Mapel
            const stdListContainer = document.getElementById('dashboard-student-list');
            if (appData.students.length === 0) {
                stdListContainer.innerHTML = `<p class="text-center text-sm text-slate-400 py-4">Belum ada siswa terdaftar.</p>`;
            } else {
                stdListContainer.innerHTML = appData.students.map(s => `
                    <div class="p-3 rounded-xl border border-slate-100 bg-slate-50/50">
                        <div class="flex justify-between items-start">
                            <div>
                                <p class="font-bold text-sm text-slate-900">${s.name}</p>
                                <p class="text-xs text-slate-500">${s.grade} • ${formatRupiah(s.fee)}/sesi</p>
                            </div>
                        </div>
                        <div class="flex flex-wrap gap-1.5 mt-2">
                            ${(s.subjects || []).map(sub => `<span class="px-2 py-0.5 bg-white border border-slate-200 rounded-md text-[11px] font-medium text-slate-700">${sub}</span>`).join('')}
                        </div>
                    </div>
                `).join('');
            }
        }

        // ==================== 2. MANAJEMEN SISWA & MAPEL ====================
        function renderStudents() {
            const grid = document.getElementById('students-grid');
            if (appData.students.length === 0) {
                grid.innerHTML = `
                    <div class="col-span-full bg-white p-10 rounded-2xl border border-dashed border-slate-300 text-center">
                        <p class="text-slate-500 text-sm mb-3">Belum ada data siswa. Tambahkan siswa pertama Anda sekarang.</p>
                        <button onclick="openStudentModal()" class="bg-brand-600 text-white px-4 py-2 rounded-xl text-sm font-medium">+ Tambah Siswa</button>
                    </div>
                `;
                return;
            }

            grid.innerHTML = appData.students.map(s => {
                const sessionCount = appData.journals.filter(j => j.studentId === s.id && j.status === 'Hadir').length;
                return `
                    <div class="bg-white rounded-2xl border border-slate-200/80 shadow-sm p-5 flex flex-col justify-between">
                        <div>
                            <div class="flex items-start justify-between gap-2">
                                <div class="flex items-center gap-3">
                                    <div class="w-11 h-11 rounded-xl bg-brand-100 text-brand-700 font-bold flex items-center justify-center text-base">
                                        ${s.name.charAt(0).toUpperCase()}
                                    </div>
                                    <div>
                                        <h3 class="font-bold text-slate-900">${s.name}</h3>
                                        <p class="text-xs text-slate-500">${s.grade}</p>
                                    </div>
                                </div>
                                <div class="flex gap-1">
                                    <button onclick="openStudentModal('${s.id}')" title="Edit Siswa" class="p-1.5 text-slate-400 hover:text-brand-600 hover:bg-brand-50 rounded-lg transition-colors">
                                        <i data-lucide="pencil" class="w-4 h-4"></i>
                                    </button>
                                    <button onclick="deleteStudent('${s.id}')" title="Hapus Siswa" class="p-1.5 text-slate-400 hover:text-red-600 hover:bg-red-50 rounded-lg transition-colors">
                                        <i data-lucide="trash-2" class="w-4 h-4"></i>
                                    </button>
                                </div>
                            </div>

                            <div class="mt-4 space-y-2 text-xs text-slate-600">
                                <div class="flex items-center justify-between py-1.5 border-b border-slate-100">
                                    <span class="text-slate-400">Tarif per Sesi:</span>
                                    <span class="font-bold text-slate-800">${formatRupiah(s.fee)}</span>
                                </div>
                                <div class="flex items-center justify-between py-1.5 border-b border-slate-100">
                                    <span class="text-slate-400">WhatsApp Ortu:</span>
                                    <span class="font-medium text-slate-800">${s.phone || '-'}</span>
                                </div>
                                <div class="flex items-center justify-between py-1.5">
                                    <span class="text-slate-400">Total Kehadiran:</span>
                                    <span class="font-semibold text-emerald-600">${sessionCount} pertemuan</span>
                                </div>
                            </div>

                            <div class="mt-4">
                                <p class="text-[11px] font-semibold text-slate-400 uppercase tracking-wider mb-2">Mata Pelajaran Diambil:</p>
                                <div class="flex flex-wrap gap-1.5">
                                    ${(s.subjects || []).map(sub => `
                                        <span class="px-2.5 py-1 rounded-lg bg-brand-50 text-brand-700 text-xs font-medium">${sub}</span>
                                    `).join('')}
                                </div>
                            </div>
                        </div>

                        <div class="mt-5 pt-3 border-t border-slate-100 flex gap-2">
                            <button onclick="goToStudentAnalysis('${s.id}')" class="flex-1 text-center py-2 bg-slate-50 hover:bg-brand-50 text-slate-700 hover:text-brand-700 rounded-xl text-xs font-semibold transition-colors">
                                Analisis Mapel
                            </button>
                            <button onclick="openJournalModalWithStudent('${s.id}')" class="flex-1 text-center py-2 bg-brand-600 hover:bg-brand-700 text-white rounded-xl text-xs font-semibold transition-colors">
                                + Catat Les
                            </button>
                        </div>
                    </div>
                `;
            }).join('');
            lucide.createIcons();
        }

        function openStudentModal(studentId = null) {
            const modal = document.getElementById('student-modal');
            const title = document.getElementById('student-modal-title');
            const form = document.getElementById('student-form');
            form.reset();

            let selectedSubjects = ['Matematika'];
            if (studentId) {
                const s = getStudentById(studentId);
                if (s) {
                    title.textContent = 'Edit Data Siswa';
                    document.getElementById('student-id').value = s.id;
                    document.getElementById('student-name').value = s.name;
                    document.getElementById('student-grade').value = s.grade;
                    document.getElementById('student-phone').value = s.phone || '';
                    document.getElementById('student-fee').value = s.fee;
                    selectedSubjects = s.subjects || [];
                }
            } else {
                title.textContent = 'Tambah Siswa Baru';
                document.getElementById('student-id').value = '';
            }

            // Gabungkan mapel standar dengan mapel custom siswa jika ada
            const allAvailableSubjects = Array.from(new Set([...DEFAULT_SUBJECTS, ...selectedSubjects]));
            renderSubjectCheckboxes(allAvailableSubjects, selectedSubjects);

            modal.classList.remove('hidden');
            modal.classList.add('flex');
        }

        function renderSubjectCheckboxes(allList, selectedList) {
            const container = document.getElementById('subject-checkboxes');
            container.innerHTML = allList.map(sub => {
                const isChecked = selectedList.includes(sub) ? 'checked' : '';
                return `
                    <label class="flex items-center gap-2 p-2 rounded-xl border border-slate-200 hover:bg-slate-50 cursor-pointer text-xs font-medium text-slate-700">
                        <input type="checkbox" name="student-subjects" value="${sub}" ${isChecked} class="rounded text-brand-600 focus:ring-brand-500">
                        <span class="truncate">${sub}</span>
                    </label>
                `;
            }).join('');
        }

        function addCustomSubjectToModal() {
            const input = document.getElementById('custom-subject-input');
            const val = input.value.trim();
            if (!val) return;

            const container = document.getElementById('subject-checkboxes');
            // Cek duplikat
            const existing = Array.from(container.querySelectorAll('input')).map(i => i.value.toLowerCase());
            if (existing.includes(val.toLowerCase())) {
                showToast("Mata pelajaran tersebut sudah ada di daftar.", "info");
                return;
            }

            const label = document.createElement('label');
            label.className = "flex items-center gap-2 p-2 rounded-xl border border-brand-200 bg-brand-50/50 cursor-pointer text-xs font-medium text-brand-800";
            label.innerHTML = `
                <input type="checkbox" name="student-subjects" value="${val}" checked class="rounded text-brand-600 focus:ring-brand-500">
                <span class="truncate">${val}</span>
            `;
            container.appendChild(label);
            input.value = '';
        }

        function closeStudentModal() {
            const modal = document.getElementById('student-modal');
            modal.classList.add('hidden');
            modal.classList.remove('flex');
        }

        function saveStudent(e) {
            e.preventDefault();
            const id = document.getElementById('student-id').value;
            const name = document.getElementById('student-name').value.trim();
            const grade = document.getElementById('student-grade').value.trim();
            const phone = document.getElementById('student-phone').value.trim();
            const fee = Number(document.getElementById('student-fee').value);

            const checkedBoxes = document.querySelectorAll('input[name="student-subjects"]:checked');
            const subjects = Array.from(checkedBoxes).map(cb => cb.value);

            if (subjects.length === 0) {
                showToast("Pilih minimal satu mata pelajaran untuk anak!", "error");
                return;
            }

            if (id) {
                const idx = appData.students.findIndex(s => s.id === id);
                if (idx !== -1) {
                    appData.students[idx] = { id, name, grade, phone, fee, subjects };
                    showToast("Data siswa berhasil diperbarui!");
                }
            } else {
                const newStudent = {
                    id: 'std-' + Date.now(),
                    name,
                    grade,
                    phone,
                    fee,
                    subjects
                };
                appData.students.push(newStudent);
                showToast("Siswa baru berhasil ditambahkan!");
            }

            saveDataToStorage();
            closeStudentModal();
            refreshAllViews();
        }

        function deleteStudent(id) {
            const s = getStudentById(id);
            if (!s) return;
            showCustomConfirm(
                "Hapus Data Siswa?",
                `Menghapus "${s.name}" juga akan menghapus seluruh riwayat jurnal pertemuan siswa tersebut. Lanjutkan?`,
                () => {
                    appData.students = appData.students.filter(st => st.id !== id);
                    appData.journals = appData.journals.filter(j => j.studentId !== id);
                    saveDataToStorage();
                    refreshAllViews();
                    showToast("Data siswa berhasil dihapus.", "info");
                }
            );
        }

        // ==================== 3. JURNAL HARIAN & KEHADIRAN ====================
        function getStatusBadgeHTML(status) {
            const styles = {
                'Hadir': 'bg-emerald-50 text-emerald-700 border-emerald-200',
                'Izin': 'bg-blue-50 text-blue-700 border-blue-200',
                'Sakit': 'bg-amber-50 text-amber-700 border-amber-200',
                'Alpha': 'bg-red-50 text-red-700 border-red-200'
            };
            const cls = styles[status] || 'bg-slate-100 text-slate-700';
            return `<span class="px-2.5 py-0.5 rounded-full text-xs font-semibold border ${cls}">${status}</span>`;
        }

        function renderJournals() {
            const container = document.getElementById('journal-list-container');
            const filterStd = document.getElementById('filter-journal-student').value;
            const filterSubj = document.getElementById('filter-journal-subject').value;
            const filterMonth = document.getElementById('filter-journal-month').value;
            const filterStatus = document.getElementById('filter-journal-status').value;

            let filtered = appData.journals.filter(j => {
                if (filterStd !== 'ALL' && j.studentId !== filterStd) return false;
                if (filterSubj !== 'ALL' && j.subject !== filterSubj) return false;
                if (filterMonth && !j.date.startsWith(filterMonth)) return false;
                if (filterStatus !== 'ALL' && j.status !== filterStatus) return false;
                return true;
            });

            // Urutkan dari tanggal terbaru
            filtered.sort((a, b) => b.date.localeCompare(a.date));

            if (filtered.length === 0) {
                container.innerHTML = `
                    <div class="bg-white p-10 rounded-2xl border border-slate-200/80 text-center">
                        <p class="text-slate-500 text-sm">Tidak ada catatan pertemuan yang sesuai dengan filter saat ini.</p>
                    </div>
                `;
                return;
            }

            container.innerHTML = filtered.map(j => {
                const std = getStudentById(j.studentId);
                const stdName = std ? std.name : 'Siswa Tidak Ditemukan';
                const stdGrade = std ? std.grade : '';
                return `
                    <div class="bg-white p-5 rounded-2xl border border-slate-200/80 shadow-sm hover:border-brand-200 transition-all">
                        <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3 pb-3 border-b border-slate-100">
                            <div class="flex flex-wrap items-center gap-2.5">
                                <span class="font-bold text-base text-slate-900">${stdName}</span>
                                <span class="text-xs text-slate-400">(${stdGrade})</span>
                                <span class="px-3 py-1 rounded-lg bg-brand-50 text-brand-700 text-xs font-bold">${j.subject}</span>
                                ${getStatusBadgeHTML(j.status)}
                            </div>
                            <div class="flex items-center justify-between sm:justify-end gap-4">
                                <div class="text-xs text-slate-500 font-medium flex items-center gap-1.5">
                                    <i data-lucide="calendar" class="w-3.5 h-3.5 text-slate-400"></i>
                                    <span>${formatDateIndo(j.date)}</span>
                                    <span class="text-slate-300">|</span>
                                    <i data-lucide="clock" class="w-3.5 h-3.5 text-slate-400"></i>
                                    <span>${j.time}</span>
                                </div>
                                <div class="flex items-center gap-1">
                                    <button onclick="openJournalModal('${j.id}')" title="Edit Jurnal" class="p-1.5 text-slate-400 hover:text-brand-600 hover:bg-brand-50 rounded-lg">
                                        <i data-lucide="pencil" class="w-4 h-4"></i>
                                    </button>
                                    <button onclick="deleteJournal('${j.id}')" title="Hapus Jurnal" class="p-1.5 text-slate-400 hover:text-red-600 hover:bg-red-50 rounded-lg">
                                        <i data-lucide="trash-2" class="w-4 h-4"></i>
                                    </button>
                                </div>
                            </div>
                        </div>

                        <div class="grid grid-cols-1 md:grid-cols-3 gap-4 pt-3.5 text-sm">
                            <div class="bg-slate-50/80 p-3 rounded-xl">
                                <span class="block text-[11px] font-bold text-slate-400 uppercase mb-1">Materi yang Diajarkan</span>
                                <p class="text-slate-800 font-medium">${j.material || '-'}</p>
                            </div>
                            <div class="bg-amber-50/50 p-3 rounded-xl">
                                <span class="block text-[11px] font-bold text-amber-600/80 uppercase mb-1">Kendala Belajar Anak</span>
                                <p class="text-slate-700">${j.obstacle || 'Tidak ada kendala berarti.'}</p>
                            </div>
                            <div class="bg-emerald-50/50 p-3 rounded-xl flex flex-col justify-between">
                                <div>
                                    <span class="block text-[11px] font-bold text-emerald-700/80 uppercase mb-1">Keterangan & Evaluasi Anak</span>
                                    <p class="text-slate-700">${j.notes || '-'}</p>
                                </div>
                                ${j.status === 'Hadir' ? `
                                    <div class="mt-2 pt-2 border-t border-emerald-100 flex items-center justify-between text-xs">
                                        <span class="text-slate-500">Pemahaman Materi:</span>
                                        <span class="font-bold text-brand-600">${j.score ?? 80}%</span>
                                    </div>
                                ` : ''}
                            </div>
                        </div>
                    </div>
                `;
            }).join('');
            lucide.createIcons();
        }

        function openJournalModal(journalId = null) {
            if (appData.students.length === 0) {
                showToast("Tambahkan data siswa terlebih dahulu sebelum mencatat jurnal!", "error");
                switchTab('siswa');
                openStudentModal();
                return;
            }

            const modal = document.getElementById('journal-modal');
            const title = document.getElementById('journal-modal-title');
            const form = document.getElementById('journal-form');
            form.reset();

            // Isi pilihan siswa
            const stdSelect = document.getElementById('journal-student');
            stdSelect.innerHTML = appData.students.map(s => `<option value="${s.id}">${s.name} (${s.grade})</option>`).join('');

            if (journalId) {
                const j = appData.journals.find(item => item.id === journalId);
                if (j) {
                    title.textContent = 'Edit Catatan Pertemuan';
                    document.getElementById('journal-id').value = j.id;
                    stdSelect.value = j.studentId;
                    updateJournalSubjectOptions(j.subject);
                    document.getElementById('journal-date').value = j.date;
                    document.getElementById('journal-time').value = j.time;
                    document.getElementById('journal-status').value = j.status;
                    document.getElementById('journal-material').value = j.material;
                    document.getElementById('journal-obstacle').value = j.obstacle || '';
                    document.getElementById('journal-notes').value = j.notes || '';
                    const score = j.score ?? 80;
                    document.getElementById('journal-score').value = score;
                    document.getElementById('mastery-score-display').textContent = score + '%';
                }
            } else {
                title.textContent = 'Catat Pertemuan Les Privat';
                document.getElementById('journal-id').value = '';
                document.getElementById('journal-date').value = new Date().toISOString().split('T')[0];
                document.getElementById('journal-time').value = '16.00 - 17.30';
                document.getElementById('journal-score').value = 80;
                document.getElementById('mastery-score-display').textContent = '80%';
                updateJournalSubjectOptions();
            }

            modal.classList.remove('hidden');
            modal.classList.add('flex');
        }

        function openJournalModalWithStudent(studentId) {
            openJournalModal();
            document.getElementById('journal-student').value = studentId;
            updateJournalSubjectOptions();
        }

        function updateJournalSubjectOptions(selectedSubject = null) {
            const stdId = document.getElementById('journal-student').value;
            const std = getStudentById(stdId);
            const subjSelect = document.getElementById('journal-subject');
            const subjects = std && std.subjects ? std.subjects : DEFAULT_SUBJECTS;

            subjSelect.innerHTML = subjects.map(sub => `<option value="${sub}">${sub}</option>`).join('');
            if (selectedSubject && subjects.includes(selectedSubject)) {
                subjSelect.value = selectedSubject;
            }
        }

        function closeJournalModal() {
            const modal = document.getElementById('journal-modal');
            modal.classList.add('hidden');
            modal.classList.remove('flex');
        }

        function saveJournal(e) {
            e.preventDefault();
            const id = document.getElementById('journal-id').value;
            const entry = {
                id: id || 'jrn-' + Date.now(),
                studentId: document.getElementById('journal-student').value,
                subject: document.getElementById('journal-subject').value,
                date: document.getElementById('journal-date').value,
                time: document.getElementById('journal-time').value.trim(),
                status: document.getElementById('journal-status').value,
                material: document.getElementById('journal-material').value.trim(),
                obstacle: document.getElementById('journal-obstacle').value.trim(),
                notes: document.getElementById('journal-notes').value.trim(),
                score: Number(document.getElementById('journal-score').value)
            };

            if (id) {
                const idx = appData.journals.findIndex(j => j.id === id);
                if (idx !== -1) appData.journals[idx] = entry;
                showToast("Catatan pertemuan berhasil diperbarui!");
            } else {
                appData.journals.push(entry);
                showToast("Catatan pertemuan tersimpan permanen!");
            }

            saveDataToStorage();
            closeJournalModal();
            refreshAllViews();
        }

        function deleteJournal(id) {
            showCustomConfirm(
                "Hapus Catatan Jurnal?",
                "Catatan pertemuan ini akan dihapus permanen dari riwayat dan perhitungan analisis. Lanjutkan?",
                () => {
                    appData.journals = appData.journals.filter(j => j.id !== id);
                    saveDataToStorage();
                    refreshAllViews();
                    showToast("Catatan jurnal dihapus.", "info");
                }
            );
        }

        // ==================== 4. ANALISIS PEMBELAJARAN PER MAPEL ====================
        function goToStudentAnalysis(studentId) {
            switchTab('analisis');
            document.getElementById('analysis-student-select').value = studentId;
            handleAnalysisStudentChange();
        }

        function initAnalysisView() {
            const select = document.getElementById('analysis-student-select');
            if (appData.students.length === 0) return;
            if (!select.value || !getStudentById(select.value)) {
                select.value = appData.students[0].id;
            }
            handleAnalysisStudentChange();
        }

        function handleAnalysisStudentChange() {
            const studentId = document.getElementById('analysis-student-select').value;
            const student = getStudentById(studentId);
            if (!student) return;

            currentAnalysisStudentId = studentId;
            const subjects = student.subjects || [];
            const tabsContainer = document.getElementById('analysis-subject-tabs');

            if (subjects.length === 0) {
                tabsContainer.innerHTML = `<span class="text-xs text-slate-400">Siswa ini belum memiliki mata pelajaran terdaftar.</span>`;
                return;
            }

            if (!currentAnalysisSubject || !subjects.includes(currentAnalysisSubject)) {
                currentAnalysisSubject = subjects[0];
            }

            // Render tombol pemisah masing-masing mata pelajaran
            tabsContainer.innerHTML = subjects.map(sub => {
                const isActive = sub === currentAnalysisSubject;
                const activeClass = isActive 
                    ? 'bg-brand-600 text-white shadow-sm border-brand-600' 
                    : 'bg-white text-slate-600 border-slate-200 hover:bg-slate-50';
                return `
                    <button onclick="selectAnalysisSubject('${sub}')" class="px-4 py-2 rounded-xl text-xs font-semibold border transition-all ${activeClass}">
                        ${sub}
                    </button>
                `;
            }).join('');

            renderSubjectAnalysisDetail();
        }

        function selectAnalysisSubject(subject) {
            currentAnalysisSubject = subject;
            handleAnalysisStudentChange();
        }

        function renderSubjectAnalysisDetail() {
            const stdId = currentAnalysisStudentId;
            const subject = currentAnalysisSubject;
            if (!stdId || !subject) return;

            // Filter jurnal khusus anak ini & khusus mata pelajaran ini saja
            const subjectJournals = appData.journals
                .filter(j => j.studentId === stdId && j.subject === subject)
                .sort((a, b) => a.date.localeCompare(b.date)); // Urut kronologis untuk grafik

            const presentJournals = subjectJournals.filter(j => j.status === 'Hadir');

            // 1. Hitung rata-rata penguasaan materi
            const avgScore = presentJournals.length > 0
                ? Math.round(presentJournals.reduce((sum, j) => sum + (j.score ?? 80), 0) / presentJournals.length)
                : 0;

            document.getElementById('ana-avg-mastery').textContent = `${avgScore}%`;
            document.getElementById('ana-mastery-bar').style.width = `${avgScore}%`;

            const badge = document.getElementById('ana-mastery-badge');
            const focusStatus = document.getElementById('ana-focus-status');
            const focusDesc = document.getElementById('ana-focus-desc');

            if (presentJournals.length === 0) {
                badge.textContent = 'Belum Ada Data';
                badge.className = 'px-2.5 py-0.5 rounded-full text-xs font-medium bg-slate-100 text-slate-600 mb-1';
                focusStatus.textContent = `Belum ada sesi ${subject}`;
                focusDesc.textContent = 'Catat pertemuan pada mapel ini untuk melihat analisis otomatis.';
            } else if (avgScore >= 85) {
                badge.textContent = 'Sangat Baik';
                badge.className = 'px-2.5 py-0.5 rounded-full text-xs font-semibold bg-emerald-100 text-emerald-700 mb-1';
                focusStatus.textContent = 'Siap Pengayaan / Materi Lanjutan';
                focusDesc.textContent = `Ananda menguasai materi ${subject} dengan sangat baik.`;
            } else if (avgScore >= 70) {
                badge.textContent = 'Berkembang Baik';
                badge.className = 'px-2.5 py-0.5 rounded-full text-xs font-semibold bg-blue-100 text-blue-700 mb-1';
                focusStatus.textContent = 'Mantapkan Latihan Soal';
                focusDesc.textContent = `Pemahaman dasar ${subject} sudah terbentuk, perbanyak variasi soal.`;
            } else {
                badge.textContent = 'Perlu Pendalaman';
                badge.className = 'px-2.5 py-0.5 rounded-full text-xs font-semibold bg-amber-100 text-amber-700 mb-1';
                focusStatus.textContent = 'Butuh Pengulangan Konsep Dasar';
                focusDesc.textContent = `Perlu bimbingan bertahap untuk mengatasi kendala di ${subject}.`;
            }

            // 2. Total sesi & tanggal terakhir
            document.getElementById('ana-total-sessions').textContent = presentJournals.length;
            const lastSession = presentJournals[presentJournals.length - 1];
            document.getElementById('ana-last-date').textContent = lastSession 
                ? `Pertemuan terakhir: ${formatDateIndo(lastSession.date)}` 
                : 'Pertemuan terakhir: -';

            // 3. Catatan analisis guru tersimpan per mapel
            const noteKey = `${stdId}_${subject}`;
            document.getElementById('ana-teacher-summary').value = appData.subjectNotes[noteKey] || '';

            // 4. Daftar Materi yang sudah dipelajari pada mapel ini
            const matList = document.getElementById('ana-materials-list');
            if (presentJournals.length === 0) {
                matList.innerHTML = `<li class="text-slate-400 italic text-xs">Belum ada materi tercatat untuk mapel ${subject}.</li>`;
            } else {
                matList.innerHTML = [...presentJournals].reverse().map(j => `
                    <li class="p-2.5 rounded-xl bg-slate-50 border border-slate-100 flex items-start justify-between gap-2">
                        <div>
                            <p class="font-medium text-slate-800">${j.material}</p>
                            <span class="text-[11px] text-slate-400">${formatDateIndo(j.date)}</span>
                        </div>
                        <span class="px-2 py-0.5 rounded-md bg-white border border-slate-200 text-xs font-bold text-brand-600 flex-shrink-0">${j.score ?? 80}%</span>
                    </li>
                `).join('');
            }

            // 5. Daftar Kendala spesifik pada mapel ini
            const obsList = document.getElementById('ana-obstacles-list');
            const journalsWithObstacles = [...presentJournals].reverse().filter(j => j.obstacle && j.obstacle.trim() !== '');
            if (journalsWithObstacles.length === 0) {
                obsList.innerHTML = `<li class="text-slate-400 italic text-xs">Tidak ada kendala khusus yang tercatat pada mapel ${subject}.</li>`;
            } else {
                obsList.innerHTML = journalsWithObstacles.map(j => `
                    <li class="p-2.5 rounded-xl bg-amber-50/60 border border-amber-100">
                        <p class="text-slate-800">${j.obstacle}</p>
                        <span class="text-[11px] text-amber-700/80 font-medium mt-1 block">${formatDateIndo(j.date)} • Materi: ${j.material}</span>
                    </li>
                `).join('');
            }

            // 6. Render Grafik Perkembangan Mapel
            document.getElementById('ana-chart-subtitle').textContent = `Grafik skor pemahaman mata pelajaran ${subject}`;
            renderSubjectChart(presentJournals, subject);
        }

        function saveSubjectAnalysisNote() {
            if (!currentAnalysisStudentId || !currentAnalysisSubject) return;
            const noteKey = `${currentAnalysisStudentId}_${currentAnalysisSubject}`;
            appData.subjectNotes[noteKey] = document.getElementById('ana-teacher-summary').value.trim();
            saveDataToStorage();
            showToast(`Analisis untuk mapel ${currentAnalysisSubject} berhasil disimpan!`);
        }

        function renderSubjectChart(presentJournals, subjectName) {
            const ctx = document.getElementById('subjectProgressChart').getContext('2d');
            if (progressChartInstance) {
                progressChartInstance.destroy();
            }

            const labels = presentJournals.map(j => formatDateIndo(j.date));
            const scores = presentJournals.map(j => j.score ?? 80);

            progressChartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: labels.length ? labels : ['Belum ada sesi'],
                    datasets: [{
                        label: `Pemahaman ${subjectName} (%)`,
                        data: scores.length ? scores : [0],
                        borderColor: '#4f46e5',
                        backgroundColor: 'rgba(79, 70, 229, 0.12)',
                        borderWidth: 3,
                        pointBackgroundColor: '#4f46e5',
                        pointRadius: 5,
                        pointHoverRadius: 7,
                        fill: true,
                        tension: 0.3
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true,
                            max: 100,
                            ticks: {
                                callback: value => value + '%'
                            },
                            grid: {
                                color: '#f1f5f9'
                            }
                        },
                        x: {
                            grid: {
                                display: false
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        },
                        tooltip: {
                            callbacks: {
                                afterLabel: function(context) {
                                    const idx = context.dataIndex;
                                    const j = presentJournals[idx];
                                    return j ? `Materi: ${j.material}` : '';
                                }
                            }
                        }
                    }
                }
            });
        }

        // ==================== 5. REKAP KEHADIRAN & INVOICE ====================
        function initInvoiceView() {
            const select = document.getElementById('invoice-student-select');
            if (appData.students.length === 0) return;
            if (!select.value || !getStudentById(select.value)) {
                select.value = appData.students[0].id;
            }
            renderAttendanceAndInvoice();
        }

        function renderAttendanceAndInvoice() {
            const stdId = document.getElementById('invoice-student-select').value;
            const monthStr = document.getElementById('invoice-month-select').value;
            const subjectFilterSelect = document.getElementById('invoice-subject-filter');
            const selectedSubjectFilter = subjectFilterSelect.value;

            const student = getStudentById(stdId);
            if (!student) return;

            // Update opsi filter mapel sesuai mapel milik siswa terpilih
            const studentSubjects = student.subjects || [];
            subjectFilterSelect.innerHTML = '<option value="ALL">Semua Mapel (Rincian Terpisah)</option>' +
                studentSubjects.map(sub => `<option value="${sub}">${sub} Saja</option>`).join('');
            if (studentSubjects.includes(selectedSubjectFilter)) {
                subjectFilterSelect.value = selectedSubjectFilter;
            }

            const activeSubjectFilter = subjectFilterSelect.value;

            // Filter jurnal siswa pada bulan terpilih
            const monthJournals = appData.journals.filter(j => 
                j.studentId === stdId && (!monthStr || j.date.startsWith(monthStr))
            );

            // Kelompokkan berdasarkan mata pelajaran
            const subjectsToProcess = activeSubjectFilter === 'ALL' 
                ? Array.from(new Set([...studentSubjects, ...monthJournals.map(j => j.subject)]))
                : [activeSubjectFilter];

            const recapTbody = document.getElementById('attendance-recap-tbody');
            const invItemsTbody = document.getElementById('inv-items-tbody');

            let recapHTML = '';
            let invHTML = '';
            let grandTotal = 0;
            let totalHadirAll = 0;

            subjectsToProcess.forEach(sub => {
                const subJournals = monthJournals.filter(j => j.subject === sub);
                const hadirList = subJournals.filter(j => j.status === 'Hadir').sort((a, b) => a.date.localeCompare(b.date));
                const izinCount = subJournals.filter(j => j.status === 'Izin').length;
                const sakitCount = subJournals.filter(j => j.status === 'Sakit').length;
                const alphaCount = subJournals.filter(j => j.status === 'Alpha').length;

                const hadirCount = hadirList.length;
                const subtotal = hadirCount * Number(student.fee || 0);
                grandTotal += subtotal;
                totalHadirAll += hadirCount;

                recapHTML += `
                    <tr class="hover:bg-slate-50/80">
                        <td class="py-3.5 px-5 font-bold text-slate-800">${sub}</td>
                        <td class="py-3.5 px-4 text-center"><span class="px-2.5 py-0.5 rounded-full bg-emerald-50 text-emerald-700 font-bold text-xs">${hadirCount}</span></td>
                        <td class="py-3.5 px-4 text-center text-slate-600">${izinCount}</td>
                        <td class="py-3.5 px-4 text-center text-slate-600">${sakitCount}</td>
                        <td class="py-3.5 px-4 text-center text-slate-600">${alphaCount}</td>
                        <td class="py-3.5 px-5 text-right text-slate-600">${formatRupiah(student.fee)}</td>
                        <td class="py-3.5 px-5 text-right font-bold text-slate-900">${formatRupiah(subtotal)}</td>
                    </tr>
                `;

                const datesFormatted = hadirList.length > 0 
                    ? hadirList.map(h => new Date(h.date + 'T00:00:00').toLocaleDateString('id-ID', { day: 'numeric', month: 'short' })).join(', ')
                    : '<span class="text-slate-400 italic">Tidak ada sesi hadir</span>';

                invHTML += `
                    <tr>
                        <td class="py-3.5 font-bold text-slate-800 align-top">${sub}</td>
                        <td class="py-3.5 text-xs text-slate-600 align-top max-w-xs">${datesFormatted}</td>
                        <td class="py-3.5 text-center font-semibold text-slate-800 align-top">${hadirCount}x</td>
                        <td class="py-3.5 text-right text-slate-600 align-top">${formatRupiah(student.fee)}</td>
                        <td class="py-3.5 text-right font-bold text-slate-900 align-top">${formatRupiah(subtotal)}</td>
                    </tr>
                `;
            });

            if (subjectsToProcess.length === 0) {
                recapHTML = `<tr><td colspan="7" class="py-6 text-center text-slate-400 text-sm">Belum ada data mata pelajaran.</td></tr>`;
            }

            recapTbody.innerHTML = recapHTML;
            invItemsTbody.innerHTML = invHTML;

            // Update Informasi Dokumen Invoice
            const periodLabel = monthStr 
                ? new Date(monthStr + '-01T00:00:00').toLocaleDateString('id-ID', { month: 'long', year: 'numeric' })
                : 'Semua Periode';

            document.getElementById('inv-tutor-brand').textContent = appData.settings.tutorName || 'Bimbingan Les Privat';
            document.getElementById('inv-payment-note').textContent = appData.settings.paymentNote || '';
            document.getElementById('inv-number').textContent = `INV/${(monthStr || '2026').replace('-', '')}/${student.id.slice(-4).toUpperCase()}`;
            document.getElementById('inv-date').textContent = `Tanggal Cetak: ${new Date().toLocaleDateString('id-ID', { day: 'numeric', month: 'long', year: 'numeric' })}`;
            
            document.getElementById('inv-student-name').textContent = student.name;
            document.getElementById('inv-student-grade').textContent = student.grade;
            document.getElementById('inv-student-phone').textContent = student.phone ? `WA Ortu: ${student.phone}` : '';
            document.getElementById('inv-period').textContent = periodLabel;
            document.getElementById('inv-grand-total').textContent = formatRupiah(grandTotal);
        }

        function copyInvoiceWhatsApp() {
            const stdId = document.getElementById('invoice-student-select').value;
            const monthStr = document.getElementById('invoice-month-select').value;
            const activeSubjectFilter = document.getElementById('invoice-subject-filter').value;
            const student = getStudentById(stdId);
            if (!student) return;

            const periodLabel = monthStr 
                ? new Date(monthStr + '-01T00:00:00').toLocaleDateString('id-ID', { month: 'long', year: 'numeric' })
                : 'Semua Periode';

            const monthJournals = appData.journals.filter(j => 
                j.studentId === stdId && (!monthStr || j.date.startsWith(monthStr))
            );

            const subjectsToProcess = activeSubjectFilter === 'ALL' 
                ? (student.subjects || [])
                : [activeSubjectFilter];

            let lines = [];
            lines.push(`*INVOICE LES PRIVAT - ${appData.settings.tutorName.toUpperCase()}*`);
            lines.push(`Kepada Yth. Orang Tua dari *${student.name}* (${student.grade})`);
            lines.push(`Periode: *${periodLabel}*`);
            lines.push(`-----------------------------------`);

            let grandTotal = 0;
            subjectsToProcess.forEach(sub => {
                const hadirList = monthJournals
                    .filter(j => j.subject === sub && j.status === 'Hadir')
                    .sort((a, b) => a.date.localeCompare(b.date));
                const count = hadirList.length;
                const subtotal = count * Number(student.fee || 0);
                grandTotal += subtotal;

                const tglStr = hadirList.map(h => new Date(h.date + 'T00:00:00').toLocaleDateString('id-ID', { day: 'numeric', month: 'short' })).join(', ');
                lines.push(`📘 *${sub}*: ${count}x hadir x ${formatRupiah(student.fee)} = *${formatRupiah(subtotal)}*`);
                if (tglStr) lines.push(`   _(Tgl: ${tglStr})_`);
            });

            lines.push(`-----------------------------------`);
            lines.push(`*TOTAL TAGIHAN: ${formatRupiah(grandTotal)}*`);
            if (appData.settings.paymentNote) {
                lines.push(`\nCatatan: ${appData.settings.paymentNote}`);
            }
            lines.push(`\nTerima kasih 🙏`);

            const textToCopy = lines.join('\n');

            // Gunakan textarea temporary & execCommand('copy') agar kompatibel di iframe
            const tempTextArea = document.createElement('textarea');
            tempTextArea.value = textToCopy;
            document.body.appendChild(tempTextArea);
            tempTextArea.select();
            try {
                document.execCommand('copy');
                showToast("Rincian tagihan berhasil disalin! Siap ditempel ke WhatsApp.");
            } catch (err) {
                showToast("Gagal menyalin teks.", "error");
            }
            document.body.removeChild(tempTextArea);
        }

        // ==================== 6. PENGATURAN & BACKUP DATA ====================
        function saveSettings() {
            appData.settings.tutorName = document.getElementById('setting-tutor-name').value.trim() || 'Bimbingan Les Privat';
            appData.settings.paymentNote = document.getElementById('setting-payment-note').value.trim();
            saveDataToStorage();
            renderAttendanceAndInvoice();
            showToast("Pengaturan identitas berhasil disimpan!");
        }

        function exportBackupData() {
            const dataStr = JSON.stringify(appData, null, 2);
            const blob = new Blob([dataStr], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            const dateStamp = new Date().toISOString().split('T')[0];
            a.href = url;
            a.download = `Backup_LesPrivat_${dateStamp}.json`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
            showToast("File cadangan data berhasil diunduh!");
        }

        function importBackupData(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const imported = JSON.parse(e.target.result);
                    if (imported && Array.isArray(imported.students) && Array.isArray(imported.journals)) {
                        showCustomConfirm(
                            "Pulihkan Data dari File Backup?",
                            "Data saat ini akan digantikan dengan data dari file cadangan yang Anda pilih. Lanjutkan?",
                            () => {
                                appData = imported;
                                if (!appData.subjectNotes) appData.subjectNotes = {};
                                if (!appData.settings) appData.settings = { tutorName: 'Bimbingan Les Privat', paymentNote: '' };
                                saveDataToStorage();
                                refreshAllViews();
                                showToast("Data berhasil dipulihkan dari file backup!");
                            }
                        );
                    } else {
                        showToast("Format file backup tidak valid!", "error");
                    }
                } catch (err) {
                    showToast("Gagal membaca file JSON backup.", "error");
                }
                event.target.value = '';
            };
            reader.readAsText(file);
        }

        function resetAllDataConfirm() {
            showCustomConfirm(
                "Hapus Semua Data & Mulai Kosong?",
                "Seluruh data contoh/siswa/jurnal akan dihapus permanen agar Anda bisa mulai mengisi data asli Anda dari nol. Lanjutkan?",
                () => {
                    appData = {
                        settings: appData.settings,
                        students: [],
                        journals: [],
                        subjectNotes: {}
                    };
                    saveDataToStorage();
                    refreshAllViews();
                    showToast("Semua data telah dikosongkan. Silakan tambah siswa baru!", "info");
                }
            );
        }

        // Jalankan saat halaman selesai dimuat
        window.addEventListener('DOMContentLoaded', initApp);
    </script>
</body>
</html>
