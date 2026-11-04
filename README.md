<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Toko Azzura Jaya Kosmetik - POS & Back Office</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chart.js for Dashboard -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Google Font Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        brand: {
                            50: '#fdf2f8',
                            100: '#fce7f3',
                            500: '#ec4899',
                            600: '#db2777',
                            700: '#be185d',
                            800: '#9d174d',
                        },
                        accent: {
                            500: '#8b5cf6',
                            600: '#7c3aed',
                        }
                    },
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    }
                }
            }
        }
    </script>
    <style>
        body { font-family: 'Inter', sans-serif; }
        .custom-scrollbar::-webkit-scrollbar { width: 6px; height: 6px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: #f1f1f1; border-radius: 10px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
        
        @media print {
            body * { visibility: hidden; }
            #printable-receipt, #printable-receipt * { visibility: visible; }
            #printable-receipt { position: absolute; left: 0; top: 0; width: 100%; }
        }
    </style>
</head>
<body class="bg-gray-50 text-gray-800 antialiased h-screen flex flex-col overflow-hidden">

    <!-- LOGIN CONTAINER (MODAL FULLSCREEN) -->
    <div id="login-screen" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-900 bg-opacity-80 backdrop-blur-sm p-4">
        <div class="bg-white rounded-2xl shadow-2xl max-w-md w-full overflow-hidden border border-pink-100">
            <div class="bg-gradient-to-r from-pink-500 to-purple-600 p-6 text-white text-center relative">
                <div class="w-20 h-20 bg-white rounded-full mx-auto flex items-center justify-center shadow-md p-1 mb-3">
                    <img id="login-logo" src="https://placehold.co/150x150/ec4899/ffffff?text=Azzura" alt="Logo Azzura" class="w-full h-full object-cover rounded-full">
                </div>
                <h1 class="text-2xl font-bold tracking-wide">Azzura Jaya Kosmetik</h1>
                <p class="text-xs text-pink-100 mt-1">Sistem POS & Back Office Terintegrasi</p>
            </div>
            <div class="p-6">
                <!-- Role Selector -->
                <div class="mb-5">
                    <label class="block text-xs font-semibold text-gray-500 uppercase tracking-wider mb-2">Pilih Peran Login</label>
                    <div class="grid grid-cols-2 gap-3 p-1 bg-gray-100 rounded-xl">
                        <button id="role-btn-admin" onclick="selectLoginRole('admin')" class="py-2 rounded-lg text-sm font-semibold transition-all shadow-sm bg-white text-pink-600">
                            <i class="fa-solid font-bold fa-user-shield mr-2"></i>Admin
                        </button>
                        <button id="role-btn-kasir" onclick="selectLoginRole('kasir')" class="py-2 rounded-lg text-sm font-semibold transition-all text-gray-600 hover:text-gray-900">
                            <i class="fa-solid fa-cash-register mr-2"></i>Kasir
                        </button>
                    </div>
                </div>

                <form id="form-login" onsubmit="handleLogin(event)">
                    <div class="space-y-4">
                        <div>
                            <label class="block text-xs font-semibold text-gray-600 mb-1">Username</label>
                            <div class="relative">
                                <i class="fa-solid fa-user absolute left-3 top-3 text-gray-400 text-sm"></i>
                                <input type="text" id="login-username" value="admin" required class="w-full pl-9 pr-4 py-2.5 rounded-lg border border-gray-300 focus:ring-2 focus:ring-pink-500 focus:border-transparent text-sm">
                            </div>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-gray-600 mb-1">Password</label>
                            <div class="relative">
                                <i class="fa-solid fa-lock absolute left-3 top-3 text-gray-400 text-sm"></i>
                                <input type="password" id="login-password" value="admin123" required class="w-full pl-9 pr-4 py-2.5 rounded-lg border border-gray-300 focus:ring-2 focus:ring-pink-500 focus:border-transparent text-sm">
                            </div>
                        </div>
                        <div class="text-xs text-slate-500 bg-pink-50 p-3 rounded-lg border border-pink-100 flex items-start space-x-2">
                            <i class="fa-solid fa-circle-info text-pink-500 mt-0.5"></i>
                            <div>
                                <p class="font-medium text-pink-800">Kredensial Default:</p>
                                <p>Admin: <span class="font-semibold">admin</span> / <span class="font-semibold">admin123</span></p>
                                <p>Kasir: <span class="font-semibold">kasir</span> / <span class="font-semibold">kasir123</span></p>
                            </div>
                        </div>
                        <button type="submit" class="w-full bg-gradient-to-r from-pink-500 to-pink-600 hover:from-pink-600 hover:to-pink-700 text-white font-semibold py-3 rounded-xl shadow-md transition transform active:scale-98">
                            Masuk Ke Sistem
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>

    <!-- MAIN APP STRUCTURE -->
    <div id="main-app" class="flex-1 flex overflow-hidden hidden">
        
        <!-- SIDEBAR -->
        <aside class="w-64 bg-slate-900 text-slate-300 flex flex-col shadow-xl z-20 flex-shrink-0">
            <!-- App Header / Logo -->
            <div class="p-4 bg-slate-950 flex items-center space-x-3 border-b border-slate-800">
                <img id="app-logo" src="https://placehold.co/150x150/ec4899/ffffff?text=Azzura" alt="Logo" class="w-10 h-10 rounded-full object-cover border-2 border-pink-500">
                <div class="overflow-hidden">
                    <h2 id="app-name-display" class="font-bold text-white text-sm tracking-wide truncate">Azzura Jaya Kosmetik</h2>
                    <p class="text-xs text-pink-400 font-medium truncate">Beauty & Skincare POS</p>
                </div>
            </div>

            <!-- User Info Badge -->
            <div class="mx-3 mt-3 p-3 bg-slate-800 rounded-xl flex items-center justify-between border border-slate-700/50">
                <div class="flex items-center space-x-2.5 overflow-hidden">
                    <div class="w-8 h-8 rounded-lg bg-pink-500/20 text-pink-400 flex items-center justify-center font-bold text-sm">
                        <i id="user-role-icon" class="fa-solid fa-user-shield"></i>
                    </div>
                    <div class="truncate">
                        <p id="user-display-name" class="text-xs font-semibold text-white truncate">Administrator</p>
                        <p id="user-display-role" class="text-[10px] uppercase font-bold text-pink-400 tracking-wider">Admin</p>
                    </div>
                </div>
                <button onclick="logout()" title="Keluar" class="text-slate-400 hover:text-red-400 transition p-1">
                    <i class="fa-solid fa-right-from-bracket text-sm"></i>
                </button>
            </div>

            <!-- Navigation Links -->
            <nav class="flex-1 px-3 py-4 space-y-1.5 custom-scrollbar overflow-y-auto text-sm font-medium">
                
                <div class="text-[11px] font-bold text-slate-500 uppercase tracking-wider px-3 mb-1">Kasir / Transaksi</div>
                
                <a href="#" onclick="switchTab('pos')" id="nav-pos" class="nav-item flex items-center space-x-3 px-3 py-2.5 rounded-xl transition bg-pink-600 text-white shadow-md">
                    <i class="fa-solid fa-cash-register w-5 text-center"></i>
                    <span>POS Penjualan</span>
                </a>

                <a href="#" onclick="switchTab('stok-masuk')" id="nav-stok-masuk" class="nav-item flex items-center space-x-3 px-3 py-2.5 rounded-xl text-slate-400 hover:bg-slate-800 hover:text-white transition">
                    <i class="fa-solid fa-boxes-packing w-5 text-center"></i>
                    <span>Stok Masuk & Modal</span>
                </a>

                <a href="#" onclick="switchTab('report-kasir')" id="nav-report-kasir" class="nav-item flex items-center space-x-3 px-3 py-2.5 rounded-xl text-slate-400 hover:bg-slate-800 hover:text-white transition">
                    <i class="fa-solid fa-receipt w-5 text-center"></i>
                    <span>Report Kasir</span>
                </a>

                <div class="admin-only text-[11px] font-bold text-slate-500 uppercase tracking-wider px-3 pt-4 mb-1">Back Office (Admin)</div>

                <a href="#" onclick="switchTab('dashboard')" id="nav-dashboard" class="admin-only nav-item flex items-center space-x-3 px-3 py-2.5 rounded-xl text-slate-400 hover:bg-slate-800 hover:text-white transition">
                    <i class="fa-solid fa-chart-pie w-5 text-center"></i>
                    <span>Dashboard</span>
                </a>

                <a href="#" onclick="switchTab('produk')" id="nav-produk" class="admin-only nav-item flex items-center space-x-3 px-3 py-2.5 rounded-xl text-slate-400 hover:bg-slate-800 hover:text-white transition">
                    <i class="fa-solid fa-spray-can w-5 text-center"></i>
                    <span>Kelola Produk</span>
                </a>

                <a href="#" onclick="switchTab('laporan')" id="nav-laporan" class="admin-only nav-item flex items-center space-x-3 px-3 py-2.5 rounded-xl text-slate-400 hover:bg-slate-800 hover:text-white transition">
                    <i class="fa-solid fa-chart-column w-5 text-center"></i>
                    <span>Laporan Penjualan</span>
                </a>

                <a href="#" onclick="switchTab('pengaturan')" id="nav-pengaturan" class="admin-only nav-item flex items-center space-x-3 px-3 py-2.5 rounded-xl text-slate-400 hover:bg-slate-800 hover:text-white transition">
                    <i class="fa-solid fa-sliders w-5 text-center"></i>
                    <span>Pengaturan Toko</span>
                </a>
            </nav>

            <div class="p-3 bg-slate-950 text-slate-500 text-[11px] text-center border-t border-slate-800">
                v2.5 Pro &bull; Azzura POS
            </div>
        </aside>

        <!-- MAIN CONTENT DISPLAY AREA -->
        <main class="flex-1 flex flex-col overflow-hidden bg-gray-50 relative">

            <!-- TAB 1: POS PENJUALAN -->
            <section id="tab-pos" class="tab-content flex-1 flex flex-col lg:flex-row overflow-hidden">
                <!-- Left: Product Grid area -->
                <div class="flex-1 flex flex-col h-full overflow-hidden border-r border-gray-200">
                    <!-- Search & Filter Header -->
                    <div class="p-4 bg-white border-b border-gray-200 shadow-sm space-y-3">
                        <div class="flex flex-col sm:flex-row gap-3 items-center justify-between">
                            <div class="relative w-full sm:w-80">
                                <i class="fa-solid fa-magnifying-glass absolute left-3 top-3 text-gray-400"></i>
                                <input type="text" id="pos-search" placeholder="Cari nama atau barcode..." onkeyup="renderPOSProducts()" class="w-full pl-9 pr-4 py-2 bg-gray-50 border border-gray-200 rounded-xl text-sm focus:bg-white focus:ring-2 focus:ring-pink-500 focus:outline-none">
                            </div>
                            <div class="flex items-center space-x-2 w-full sm:w-auto">
                                <select id="pos-filter-brand" onchange="renderPOSProducts()" class="w-1/2 sm:w-auto bg-gray-50 border border-gray-200 text-gray-700 text-sm rounded-xl py-2 px-3 focus:outline-none focus:ring-2 focus:ring-pink-500">
                                    <option value="">Semua Brand</option>
                                </select>
                                <select id="pos-filter-kategori" onchange="renderPOSProducts()" class="w-1/2 sm:w-auto bg-gray-50 border border-gray-200 text-gray-700 text-sm rounded-xl py-2 px-3 focus:outline-none focus:ring-2 focus:ring-pink-500">
                                    <option value="">Semua Kategori</option>
                                </select>
                            </div>
                        </div>
                    </div>

                    <!-- Products Grid -->
                    <div class="flex-1 p-4 overflow-y-auto custom-scrollbar">
                        <div id="pos-product-grid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-4 xl:grid-cols-5 gap-4">
                            <!-- Rendered dynamically -->
                        </div>
                    </div>
                </div>

                <!-- Right: Cart & Checkout area -->
                <div class="w-full lg:w-96 bg-white flex flex-col h-full border-t lg:border-t-0 shadow-lg z-10">
                    <div class="p-4 bg-gray-50 border-b border-gray-200 flex justify-between items-center">
                        <h3 class="font-bold text-gray-800 flex items-center">
                            <i class="fa-solid fa-cart-shopping text-pink-500 mr-2"></i> Keranjang Belanja
                        </h3>
                        <button onclick="clearCart()" class="text-xs text-red-500 hover:text-red-700 font-medium">Clear All</button>
                    </div>

                    <!-- Cart Item List -->
                    <div id="pos-cart-items" class="flex-1 p-4 overflow-y-auto custom-scrollbar divide-y divide-gray-100">
                        <!-- Items rendered dynamically -->
                    </div>

                    <!-- Cart Summary & Action -->
                    <div class="p-4 bg-gray-50 border-t border-gray-200 space-y-3">
                        <div class="space-y-1.5 text-sm">
                            <div class="flex justify-between text-gray-600">
                                <span>Subtotal</span>
                                <span id="cart-subtotal" class="font-semibold">Rp 0</span>
                            </div>
                            <div class="flex justify-between text-gray-600">
                                <span>Pajak (0%)</span>
                                <span class="font-semibold">Rp 0</span>
                            </div>
                            <div class="flex justify-between text-base font-bold text-gray-900 pt-2 border-t border-gray-200">
                                <span>Total Tagihan</span>
                                <span id="cart-total" class="text-pink-600 text-lg">Rp 0</span>
                            </div>
                        </div>

                        <!-- Cash Payment Inputs -->
                        <div class="pt-2">
                            <label class="block text-xs font-semibold text-gray-600 mb-1">Nominal Bayar (Rp)</label>
                            <input type="number" id="cart-pay-amount" placeholder="0" oninput="calculateChange()" class="w-full px-3 py-2 border border-gray-300 rounded-xl text-lg font-bold text-gray-800 focus:ring-2 focus:ring-pink-500 focus:outline-none">
                            
                            <!-- Quick Money Buttons -->
                            <div class="grid grid-cols-4 gap-1.5 mt-2">
                                <button onclick="setQuickPay(50000)" class="py-1 bg-white border border-gray-200 rounded text-xs font-medium hover:bg-pink-50 hover:text-pink-600">50k</button>
                                <button onclick="setQuickPay(100000)" class="py-1 bg-white border border-gray-200 rounded text-xs font-medium hover:bg-pink-50 hover:text-pink-600">100k</button>
                                <button onclick="setQuickPay(150000)" class="py-1 bg-white border border-gray-200 rounded text-xs font-medium hover:bg-pink-50 hover:text-pink-600">150k</button>
                                <button onclick="setExactPay()" class="py-1 bg-pink-100 text-pink-700 border border-pink-200 rounded text-xs font-semibold hover:bg-pink-200">Pas</button>
                            </div>

                            <div class="flex justify-between items-center mt-3 text-sm font-semibold">
                                <span class="text-gray-600">Kembalian:</span>
                                <span id="cart-change" class="text-gray-900 font-bold">Rp 0</span>
                            </div>
                        </div>

                        <button onclick="processCheckout()" class="w-full py-3.5 bg-gradient-to-r from-pink-500 to-pink-600 hover:from-pink-600 hover:to-pink-700 text-white font-bold rounded-xl shadow-md transition transform active:scale-98 flex items-center justify-center space-x-2">
                            <i class="fa-solid fa-print"></i>
                            <span>Bayar & Cetak Struk</span>
                        </button>
                    </div>
                </div>
            </section>

            <!-- TAB 2: INPUT STOK MASUK & MODAL (KASIR & ADMIN) -->
            <section id="tab-stok-masuk" class="tab-content flex-1 p-6 overflow-y-auto custom-scrollbar hidden">
                <div class="max-w-4xl mx-auto space-y-6">
                    <div class="bg-white rounded-2xl p-6 shadow-sm border border-gray-100">
                        <div class="flex items-center space-x-3 mb-6 pb-4 border-b border-gray-100">
                            <div class="w-10 h-10 rounded-xl bg-pink-100 text-pink-600 flex items-center justify-center text-lg font-bold">
                                <i class="fa-solid fa-boxes-packing"></i>
                            </div>
                            <div>
                                <h2 class="text-lg font-bold text-gray-800">Input Data Barang Masuk & Modal Supplier</h2>
                                <p class="text-xs text-gray-500">Tambah stok & perbarui harga beli/modal barang dari supplier. <span class="text-pink-600 font-semibold">(Kasir tidak dapat mengubah harga jual)</span></p>
                            </div>
                        </div>

                        <form id="form-stok-masuk" onsubmit="handleStokMasuk(event)" class="space-y-4">
                            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                <div class="md:col-span-2">
                                    <label class="block text-xs font-semibold text-gray-600 mb-1">Pilih Produk *</label>
                                    <select id="stok-masuk-produk-id" required onchange="onSelectStokMasukProduct()" class="w-full border border-gray-300 rounded-xl p-2.5 text-sm focus:ri
