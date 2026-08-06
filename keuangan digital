<!DOCTYPE html>
<html lang="id" class="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SantriPay - Sistem Keuangan Pesantren Realtime & Supabase</title>
  
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          fontFamily: { sans: ['Plus Jakarta Sans', 'sans-serif'] }
        }
      }
    }
  </script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">

  <style>
    body { font-family: 'Plus Jakarta Sans', sans-serif; overflow-x: hidden; }
    @media print {
      body * { visibility: hidden; }
      #printable-area, #printable-area * { visibility: visible; }
      #printable-area { position: absolute; left: 0; top: 0; width: 100%; padding: 20px; background: white; color: black; }
      .printable-section, .printable-section * { visibility: visible !important; }
      .printable-section { position: absolute; left: 0; top: 0; width: 100%; background: white !important; color: black !important; }
    }
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: inherit; }
    ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
  </style>
</head>
<body class="bg-slate-100 dark:bg-slate-900 text-slate-800 dark:text-slate-100 antialiased selection:bg-emerald-500 selection:text-white transition-colors duration-300">
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect, useRef } = React;

    const EMBEDDED_SUPABASE_URL = "https://ygsjtaputrcruxgqfcbb.supabase.co";
    const EMBEDDED_SUPABASE_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Inlnc2p0YXB1dHJjcnV4Z3FmY2JiIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODU5NDI0MDAsImV4cCI6MjEwMTUxODQwMH0.UomgHIAYAkpGZj-B36NHwwhe0jF5BeJWjuJHObNTGuY";

    const DEFAULT_PESANTREN = {
      nama: "Pondok Pesantren Al-Hidayah Nusantara",
      alamat: "Jl. Kyai Haji Hasyim Asy'ari No. 88, Kebayoran Baru, Jakarta Selatan",
      telepon: "0812-3456-7890",
      email: "info@pesantrenalhidayah.sch.id",
      pimpinan: "KH. Ahmad Dahlan, M.Ag",
      bendahara: "Ust. Muhammad Ridwan, S.E.",
      slogan: "Membangun Generasi Rabbani Berakhlakul Karimah"
    };

    const DEFAULT_CONFIG = {
      sppBulanan: 350000,
      uangMakan: 400000,
      uangKitab: 150000,
      uangGedung: 500000,
      infaq: 50000,
      daftarUlang: 750000,
      lainnya: 100000
    };

    const DEFAULT_USERS = [
      { id: 'u1', username: 'admin', password: 'admin123', role: 'admin', name: 'Administrator Utama' },
      { id: 'u2', username: 'bendahara_pusat', password: 'pusat123', role: 'bendahara_pusat', name: 'Ust. Ridwan (Bendahara Pusat)' },
      { id: 'u3', username: 'bendahara_pesantren', password: 'pesantren123', role: 'bendahara_pesantren', name: 'Ust. Ahmad (Bendahara Pesantren)' }
    ];

    function App() {
      const [supabaseClient, setSupabaseClient] = useState(null);
      const [isSyncActive, setIsSyncActive] = useState(false);
      const [darkMode, setDarkMode] = useState(true);

      const [users, setUsers] = useState(() => {
        const saved = localStorage.getItem('santri_app_users');
        return saved ? JSON.parse(saved) : DEFAULT_USERS;
      });

      const [currentUser, setCurrentUser] = useState(() => {
        const saved = localStorage.getItem('santri_current_user');
        return saved ? JSON.parse(saved) : null;
      });

      const [pesantren, setPesantren] = useState(() => {
        const saved = localStorage.getItem('santri_pesantren_info');
        return saved ? JSON.parse(saved) : DEFAULT_PESANTREN;
      });

      const [configTagihan, setConfigTagihan] = useState(() => {
        const saved = localStorage.getItem('santri_config_tagihan');
        return saved ? JSON.parse(saved) : DEFAULT_CONFIG;
      });

      const [santriList, setSantriList] = useState([]);
      const [transaksiList, setTransaksiList] = useState([]);
      
      const [activeTab, setActiveTab] = useState('dashboard');
      const [printedReceipt, setPrintedReceipt] = useState(null);
      const [selectedSantriForPayment, setSelectedSantriForPayment] = useState(null);
      const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);
      const [toast, setToast] = useState({ show: false, message: '', type: 'success' });

      const showToast = (message, type = 'success') => {
        setToast({ show: true, message, type });
        setTimeout(() => setToast({ show: false, message: '', type: 'success' }), 3500);
      };

      useEffect(() => {
        if (darkMode) {
          document.documentElement.classList.add('dark');
        } else {
          document.documentElement.classList.remove('dark');
        }
      }, [darkMode]);

      useEffect(() => { localStorage.setItem('santri_app_users', JSON.stringify(users)); }, [users]);
      useEffect(() => { localStorage.setItem('santri_current_user', JSON.stringify(currentUser)); }, [currentUser]);
      useEffect(() => { localStorage.setItem('santri_pesantren_info', JSON.stringify(pesantren)); }, [pesantren]);
      useEffect(() => { localStorage.setItem('santri_config_tagihan', JSON.stringify(configTagihan)); }, [configTagihan]);

      useEffect(() => {
        let isMounted = true;
        try {
          const client = window.supabase.createClient(EMBEDDED_SUPABASE_URL, EMBEDDED_SUPABASE_KEY);
          if (isMounted) setSupabaseClient(client);
          loadDataFromSupabase(client);

          const pollInterval = setInterval(() => {
            loadDataFromSupabase(client, true);
          }, 4000);

          if (isMounted) setIsSyncActive(true);

          return () => {
            isMounted = false;
            clearInterval(pollInterval);
          };
        } catch (e) {
          console.warn("Supabase init notice:", e.message);
        }
      }, []);

      const loadDataFromSupabase = async (client, silent = false) => {
        const sb = client || supabaseClient;
        if (!sb) return;
        try {
          const { data: santriData, error: sErr } = await sb.from('santri').select('*');
          if (sErr) throw sErr;
          if (santriData) {
            setSantriList(santriData.map(s => ({
              id: s.id, nis: s.nis, nama: s.nama, kelas: s.kelas, kamar: s.kamar, wali: s.wali, noHp: s.no_hp, beasiswa: s.beasiswa, status: s.status
            })));
          }

          const { data: txData, error: tErr } = await sb.from('transaksi').select('*');
          if (tErr) throw tErr;
          if (txData) {
            setTransaksiList(txData.map(t => ({
              id: t.id, tanggal: t.tanggal, noKwitansi: t.no_kwitansi, jenis: t.jenis, kategori: t.kategori, santriId: t.santri_id, nis: t.nis, namaSantri: t.nama_santri, jumlah: t.jumlah, bulan: t.bulan, keterangan: t.keterangan, petugas: t.petugas
            })));
          }
        } catch (err) {
          if (!silent) console.warn("Load Supabase notice:", err.message);
        }
      };

      const handleLogin = (usernameInput, passwordInput) => {
        const found = users.find(u => u.username.trim() === usernameInput.trim() && u.password === passwordInput);
        if (found) {
          setCurrentUser(found);
          if (found.role === 'bendahara_pesantren') setActiveTab('pembayaran');
          else setActiveTab('dashboard');
          showToast(`Selamat datang, ${found.name}!`, 'success');
          return true;
        } else {
          showToast('Username atau password salah!', 'error');
          return false;
        }
      };

      const handleLogout = () => {
        setCurrentUser(null);
        localStorage.removeItem('santri_current_user');
        showToast('Anda telah keluar dari aplikasi', 'info');
      };

      const handleResetData = async () => {
        if (confirm("PERINGATAN: Apakah Anda yakin ingin mengosongkan seluruh data (santri, transaksi, akun kustom) dan mulai dari nol?")) {
          setSantriList([]);
          setTransaksiList([]);
          setUsers(DEFAULT_USERS);
          setPesantren(DEFAULT_PESANTREN);
          setConfigTagihan(DEFAULT_CONFIG);
          localStorage.clear();
          showToast('Seluruh data berhasil dikosongkan (Mulai dari Nol)', 'success');
        }
      };

      const totalPemasukan = transaksiList.filter(t => t.jenis === 'masuk').reduce((sum, t) => sum + Number(t.jumlah), 0);
      const totalPengeluaran = transaksiList.filter(t => t.jenis === 'keluar').reduce((sum, t) => sum + Number(t.jumlah), 0);
      const saldoKas = totalPemasukan - totalPengeluaran;

      const triggerPrintReceipt = (receiptData) => {
        setPrintedReceipt(receiptData);
        setTimeout(() => { window.print(); }, 300);
      };

      const handleSectionDownload = (elementId) => {
        const el = document.getElementById(elementId);
        if (el) {
          el.classList.add('printable-section');
          window.print();
          setTimeout(() => {
            el.classList.remove('printable-section');
          }, 500);
        } else {
          window.print();
        }
      };

      const handleQuickPayFromTunggakan = (santri) => {
        setSelectedSantriForPayment(santri);
        setActiveTab('pembayaran');
      };

      const handleSelectTab = (tabId) => {
        setActiveTab(tabId);
        setIsMobileMenuOpen(false);
      };

      if (!currentUser) {
        return <LoginView onLogin={handleLogin} pesantren={pesantren} darkMode={darkMode} setDarkMode={setDarkMode} />;
      }

      return (
        <div className="min-h-screen flex flex-col md:flex-row bg-slate-100 dark:bg-slate-900 text-slate-800 dark:text-slate-100 overflow-x-hidden">
          {toast.show && (
            <div className={`fixed top-4 right-4 z-50 px-5 py-3.5 rounded-2xl shadow-2xl border flex items-center space-x-3 transition-all transform duration-300 max-w-[90vw] ${
              toast.type === 'success' ? 'bg-gradient-to-r from-emerald-600 to-teal-700 text-white border-emerald-500' : 'bg-gradient-to-r from-rose-600 to-pink-700 text-white border-rose-500'
            }`}>
              <i className={`fa-solid ${toast.type === 'success' ? 'fa-circle-check text-emerald-200' : 'fa-triangle-exclamation text-rose-200'} text-xl`}></i>
              <span className="text-sm font-semibold truncate">{toast.message}</span>
            </div>
          )}

          {isMobileMenuOpen && (
            <div onClick={() => setIsMobileMenuOpen(false)} className="fixed inset-0 bg-slate-950/70 z-40 md:hidden backdrop-blur-sm transition-opacity"></div>
          )}

          <aside className={`fixed md:static top-0 left-0 h-full z-50 w-72 bg-gradient-to-b from-slate-950 via-slate-900 to-emerald-950 text-slate-300 flex-shrink-0 flex flex-col justify-between border-r border-slate-800 shadow-2xl transition-transform duration-300 ${
            isMobileMenuOpen ? 'translate-x-0' : '-translate-x-full md:translate-x-0'
          }`}>
            <div>
              <div className="p-6 border-b border-slate-800/80 flex items-center justify-between">
                <div className="flex items-center space-x-3.5 min-w-0">
                  <div className="w-12 h-12 rounded-2xl bg-gradient-to-tr from-emerald-500 to-teal-400 flex items-center justify-center text-white text-2xl font-black shadow-lg shadow-emerald-500/30 flex-shrink-0">
                    <i className="fa-solid fa-mosque"></i>
                  </div>
                  <div className="min-w-0">
                    <h1 className="font-extrabold text-white text-lg tracking-tight truncate">SantriPay</h1>
                    <div className="flex items-center space-x-1.5 mt-0.5">
                      <span className={`w-2 h-2 rounded-full ${isSyncActive ? 'bg-emerald-400 animate-pulse' : 'bg-amber-400'}`}></span>
                      <p className="text-[10px] text-emerald-400 font-bold tracking-wide uppercase truncate">
                        {isSyncActive ? 'Realtime Sync Aktif' : 'Menghubungkan...'}
                      </p>
                    </div>
                  </div>
                </div>
                <button onClick={() => setIsMobileMenuOpen(false)} className="md:hidden text-slate-400 hover:text-white p-2 flex-shrink-0">
                  <i className="fa-solid fa-xmark text-lg"></i>
                </button>
              </div>

              <nav className="p-4 space-y-1.5 overflow-y-auto max-h-[calc(100vh-180px)]">
                {currentUser.role === 'admin' && (
                  <>
                    {[
                      { id: 'dashboard', label: 'Dashboard Utama', icon: 'fa-chart-pie' },
                      { id: 'pembayaran', label: 'Pembayaran SPP & Lainnya', icon: 'fa-file-invoice-dollar' },
                      { id: 'tunggakan', label: 'Tunggakan Santri', icon: 'fa-triangle-exclamation' },
                      { id: 'santri', label: 'Data & Jumlah Santri', icon: 'fa-users' },
                      { id: 'kas', label: 'Buku Kas Umum', icon: 'fa-wallet' },
                      { id: 'laporan', label: 'Laporan Keuangan', icon: 'fa-file-lines' },
                      { id: 'analisis_pengeluaran', label: 'Analisis Pengeluaran', icon: 'fa-chart-line' },
                      { id: 'profil_pesantren', label: 'Profil Pesantren', icon: 'fa-building-columns' },
                      { id: 'kelola_user', label: 'Kelola Akun & PIN', icon: 'fa-user-gear' },
                      { id: 'pengaturan', label: 'Pengaturan Tarif', icon: 'fa-gear' },
                      { id: 'database_sql', label: 'Integrasi Database SQL', icon: 'fa-database' }
                    ].map(item => (
                      <button
                        key={item.id}
                        onClick={() => handleSelectTab(item.id)}
                        className={`w-full flex items-center space-x-3.5 px-4 py-3 rounded-2xl text-sm font-semibold transition-all duration-200 text-left ${
                          activeTab === item.id 
                            ? 'bg-gradient-to-r from-emerald-600 to-teal-600 text-white shadow-lg shadow-emerald-900/40 md:translate-x-1' 
                            : 'hover:bg-slate-800/80 text-slate-400 hover:text-white'
                        }`}
                      >
                        <i className={`fa-solid ${item.icon} w-5 text-center text-base ${activeTab === item.id ? 'text-white' : 'text-emerald-400'}`}></i>
                        <span className="truncate">{item.label}</span>
                      </button>
                    ))}
                  </>
                )}

                {currentUser.role === 'bendahara_pusat' && (
                  <>
                    {[
                      { id: 'dashboard', label: 'Dashboard Overview', icon: 'fa-chart-pie' },
                      { id: 'pembayaran', label: 'Pembayaran SPP & Tagihan', icon: 'fa-file-invoice-dollar' },
                      { id: 'santri', label: 'Data & Tambah Santri', icon: 'fa-users' },
                      { id: 'tunggakan', label: 'Laporan Tunggakan', icon: 'fa-triangle-exclamation' },
                      { id: 'kas', label: 'Buku Kas Umum', icon: 'fa-wallet' },
                      { id: 'laporan', label: 'Laporan Rekapitulasi', icon: 'fa-file-lines' },
                      { id: 'analisis_pengeluaran', label: 'Analisis Pengeluaran', icon: 'fa-chart-line' }
                    ].map(item => (
                      <button
                        key={item.id}
                        onClick={() => handleSelectTab(item.id)}
                        className={`w-full flex items-center space-x-3.5 px-4 py-3 rounded-2xl text-sm font-semibold transition-all duration-200 text-left ${
                          activeTab === item.id ? 'bg-gradient-to-r from-emerald-600 to-teal-600 text-white shadow-lg shadow-emerald-900/40 md:translate-x-1' : 'hover:bg-slate-800/80 text-slate-400 hover:text-white'
                        }`}
                      >
                        <i className={`fa-solid ${item.icon} w-5 text-center text-base`}></i>
                        <span className="truncate">{item.label}</span>
                      </button>
                    ))}
                  </>
                )}

                {currentUser.role === 'bendahara_pesantren' && (
                  <>
                    {[
                      { id: 'pembayaran', label: 'Pembayaran SPP & Tagihan', icon: 'fa-file-invoice-dollar' },
                      { id: 'tunggakan', label: 'Tunggakan Santri', icon: 'fa-triangle-exclamation' },
                      { id: 'santri', label: 'Data & Tambah Santri', icon: 'fa-users' }
                    ].map(item => (
                      <button
                        key={item.id}
                        onClick={() => handleSelectTab(item.id)}
                        className={`w-full flex items-center space-x-3.5 px-4 py-3 rounded-2xl text-sm font-semibold transition-all duration-200 text-left ${
                          activeTab === item.id ? 'bg-gradient-to-r from-emerald-600 to-teal-600 text-white shadow-lg shadow-emerald-900/40 md:translate-x-1' : 'hover:bg-slate-800/80 text-slate-400 hover:text-white'
                        }`}
                      >
                        <i className={`fa-solid ${item.icon} w-5 text-center text-base`}></i>
                        <span className="truncate">{item.label}</span>
                      </button>
                    ))}
                  </>
                )}
              </nav>
            </div>

            <div className="p-4 border-t border-slate-800/80 space-y-2">
              <div className="bg-slate-800/80 p-3.5 rounded-2xl border border-slate-700/60 flex items-center justify-between shadow-inner">
                <div className="truncate pr-2">
                  <p className="text-xs font-bold text-white truncate">{currentUser.name}</p>
                  <p className="text-[10px] text-emerald-400 uppercase font-bold tracking-wider truncate">{currentUser.role.replace('_', ' ')}</p>
                </div>
              </div>
            </div>
          </aside>

          <main className="flex-1 flex flex-col min-w-0 overflow-y-auto bg-slate-100 dark:bg-slate-900">
            <header className="bg-white/80 dark:bg-slate-900/90 backdrop-blur-md border-b border-slate-200 dark:border-slate-800 px-4 md:px-6 py-4 flex items-center justify-between gap-3 sticky top-0 z-10 shadow-sm">
              <div className="flex items-center space-x-3 min-w-0">
                <button onClick={() => setIsMobileMenuOpen(true)} className="md:hidden p-2.5 rounded-2xl bg-slate-200 dark:bg-slate-800 hover:bg-slate-300 dark:hover:bg-slate-700 text-slate-800 dark:text-white transition text-lg flex-shrink-0">
                  <i className="fa-solid fa-bars"></i>
                </button>
                <div className="min-w-0">
                  <h2 className="text-base md:text-lg font-black text-slate-900 dark:text-white tracking-tight truncate">
                    {activeTab === 'dashboard' && 'Dashboard Overview Keuangan'}
                    {activeTab === 'pembayaran' && 'Pembayaran SPP, Infaq & Daftar Ulang'}
                    {activeTab === 'tunggakan' && 'Laporan Tunggakan Santri'}
                    {activeTab === 'santri' && 'Manajemen & Jumlah Santri'}
                    {activeTab === 'kas' && 'Buku Kas Pemasukan & Pengeluaran'}
                    {activeTab === 'laporan' && 'Laporan & Rekapitulasi Keuangan'}
                    {activeTab === 'analisis_pengeluaran' && 'Analisis & Rata-rata Pengeluaran Bulanan'}
                    {activeTab === 'profil_pesantren' && 'Profil Identitas Pondok Pesantren'}
                    {activeTab === 'kelola_user' && 'Manajemen User & Perubahan PIN'}
                    {activeTab === 'pengaturan' && 'Pengaturan Standar Tarif Tagihan'}
                    {activeTab === 'database_sql' && 'Integrasi Database Supabase SQL'}
                  </h2>
                  <p className="text-[11px] md:text-xs text-emerald-600 dark:text-emerald-400 truncate font-medium">{pesantren.nama}</p>
                </div>
              </div>

              <div className="flex items-center gap-2.5 flex-shrink-0">
                <button 
                  onClick={() => setDarkMode(!darkMode)} 
                  className="p-2.5 rounded-2xl bg-slate-200 dark:bg-slate-800 text-slate-700 dark:text-amber-400 hover:bg-slate-300 dark:hover:bg-slate-700 transition shadow-sm"
                  title="Ganti Mode Gelap / Terang"
                >
                  <i className={`fa-solid ${darkMode ? 'fa-sun' : 'fa-moon'} text-base`}></i>
                </button>
                <div className="bg-gradient-to-r from-emerald-500 to-teal-600 text-white px-3.5 md:px-4 py-2 rounded-2xl text-[11px] md:text-xs font-extrabold flex items-center space-x-1.5 shadow-lg shadow-emerald-500/20 whitespace-nowrap">
                  <i className="fa-solid fa-vault"></i>
                  <span>Rp {saldoKas.toLocaleString('id-ID')}</span>
                </div>
                <button onClick={handleLogout} className="bg-rose-500/20 hover:bg-rose-600 text-rose-600 dark:text-rose-300 hover:text-white px-3.5 py-2 rounded-2xl transition text-xs font-bold shadow flex items-center space-x-1" title="Keluar">
                  <i className="fa-solid fa-right-from-bracket"></i>
                  <span className="hidden sm:inline">Keluar</span>
                </button>
              </div>
            </header>

            <div className="p-4 md:p-6 max-w-full">
              {activeTab === 'dashboard' && (
                <DashboardView totalPemasukan={totalPemasukan} totalPengeluaran={totalPengeluaran} saldoKas={saldoKas} santriList={santriList} transaksiList={transaksiList} setActiveTab={setActiveTab} pesantren={pesantren} />
              )}
              {activeTab === 'pembayaran' && (
                <PembayaranView supabaseClient={supabaseClient} santriList={santriList} configTagihan={configTagihan} transaksiList={transaksiList} setTransaksiList={setTransaksiList} triggerPrintReceipt={triggerPrintReceipt} showToast={showToast} selectedSantriForPayment={selectedSantriForPayment} setSelectedSantriForPayment={setSelectedSantriForPayment} currentUser={currentUser} loadDataFromSupabase={loadDataFromSupabase} />
              )}
              {activeTab === 'tunggakan' && (
                <TunggakanView santriList={santriList} transaksiList={transaksiList} configTagihan={configTagihan} handleQuickPayFromTunggakan={handleQuickPayFromTunggakan} handleSectionDownload={handleSectionDownload} />
              )}
              {activeTab === 'santri' && (
                <SantriView supabaseClient={supabaseClient} santriList={santriList} setSantriList={setSantriList} showToast={showToast} loadDataFromSupabase={loadDataFromSupabase} handleSectionDownload={handleSectionDownload} currentUser={currentUser} />
              )}
              {activeTab === 'kas' && (
                <KasView supabaseClient={supabaseClient} transaksiList={transaksiList} setTransaksiList={setTransaksiList} showToast={showToast} currentUser={currentUser} loadDataFromSupabase={loadDataFromSupabase} handleSectionDownload={handleSectionDownload} />
              )}
              {activeTab === 'laporan' && (
                <LaporanView transaksiList={transaksiList} pesantren={pesantren} handleSectionDownload={handleSectionDownload} />
              )}
              {activeTab === 'analisis_pengeluaran' && (
                <AnalisisPengeluaranView transaksiList={transaksiList} handleSectionDownload={handleSectionDownload} />
              )}
              {activeTab === 'profil_pesantren' && currentUser.role === 'admin' && (
                <ProfilPesantrenView pesantren={pesantren} setPesantren={setPesantren} showToast={showToast} />
              )}
              {activeTab === 'kelola_user' && currentUser.role === 'admin' && (
                <KelolaUserView users={users} setUsers={setUsers} showToast={showToast} currentUser={currentUser} onResetData={handleResetData} />
              )}
              {activeTab === 'pengaturan' && currentUser.role === 'admin' && (
                <PengaturanView configTagihan={configTagihan} setConfigTagihan={setConfigTagihan} showToast={showToast} />
              )}
              {activeTab === 'database_sql' && currentUser.role === 'admin' && (
                <DatabaseSqlView showToast={showToast} />
              )}
            </div>
          </main>

          {printedReceipt && (
            <div id="printable-area" className="hidden print:block font-mono text-slate-900 bg-white p-8 border rounded-2xl max-w-md mx-auto shadow-2xl">
              <div className="text-center border-b-2 border-slate-900 pb-4 mb-4">
                <h2 className="font-extrabold text-lg uppercase tracking-wider">{pesantren.nama}</h2>
                <p className="text-[11px] text-slate-700 mt-0.5">{pesantren.alamat}</p>
                <p className="text-[11px] text-slate-700">Telp: {pesantren.telepon} | Email: {pesantren.email}</p>
              </div>

              <div className="text-center my-4">
                <span className="font-black border-b border-black text-xs uppercase tracking-widest pb-1">BUKTI PEMBAYARAN RESMI</span>
                <p className="text-[11px] text-slate-600 mt-1.5 font-bold">No. Kwitansi: {printedReceipt.noKwitansi}</p>
              </div>

              <table className="w-full text-xs space-y-1.5 mb-6">
                <tbody>
                  <tr><td className="py-1 text-slate-500 font-semibold">Tanggal</td><td className="py-1 font-bold">: {printedReceipt.tanggal}</td></tr>
                  {printedReceipt.namaSantri !== '-' && (
                    <>
                      <tr><td className="py-1 text-slate-500 font-semibold">Nama Santri</td><td className="py-1 font-bold">: {printedReceipt.namaSantri} ({printedReceipt.nis})</td></tr>
                      <tr><td className="py-1 text-slate-500 font-semibold">Periode</td><td className="py-1 font-bold">: {printedReceipt.bulan}</td></tr>
                    </>
                  )}
                  <tr><td className="py-1 text-slate-500 font-semibold">Jenis Pembayaran</td><td className="py-1 font-bold">: {printedReceipt.kategori}</td></tr>
                  <tr><td className="py-1 text-slate-500 font-semibold">Petugas</td><td className="py-1 font-bold">: {printedReceipt.petugas || pesantren.bendahara}</td></tr>
                </tbody>
              </table>

              <div className="border-t-2 border-b-2 border-dashed border-slate-900 py-3 my-3 text-center bg-slate-50 rounded-xl">
                <span className="text-xs font-bold text-slate-600 uppercase">JUMLAH DIBAYAR</span>
                <div className="text-xl font-black text-slate-900 mt-0.5">Rp {Number(printedReceipt.jumlah).toLocaleString('id-ID')}</div>
              </div>

              <div className="flex justify-between text-[11px] mt-8 text-center">
                <div>
                  <p className="font-semibold">Santri / Wali</p>
                  <div className="h-12"></div>
                  <p className="font-bold">( ................................... )</p>
                </div>
                <div>
                  <p className="font-semibold">Bendahara Pesantren</p>
                  <div className="h-12"></div>
                  <p className="font-bold">( {pesantren.bendahara} )</p>
                </div>
              </div>
            </div>
          )}
        </div>
      );
    }

    function LoginView({ onLogin, pesantren, darkMode, setDarkMode }) {
      const [username, setUsername] = useState('');
      const [password, setPassword] = useState('');
      const [showPassword, setShowPassword] = useState(false);

      const handleSubmit = (e) => {
        e.preventDefault();
        onLogin(username, password);
      };

      return (
        <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-emerald-950 flex items-center justify-center p-4 relative">
          <button 
            onClick={() => setDarkMode(!darkMode)} 
            className="absolute top-6 right-6 p-3 rounded-2xl bg-slate-800 text-amber-400 hover:bg-slate-700 transition shadow-lg"
          >
            <i className={`fa-solid ${darkMode ? 'fa-sun' : 'fa-moon'} text-lg`}></i>
          </button>
          <div className="bg-white/95 dark:bg-slate-900/90 backdrop-blur-xl w-full max-w-md rounded-3xl p-6 md:p-8 shadow-2xl border border-slate-200 dark:border-slate-800 space-y-6">
            <div className="text-center">
              <div className="w-20 h-20 bg-gradient-to-tr from-emerald-600 to-teal-400 text-white rounded-3xl flex items-center justify-center text-4xl mx-auto mb-4 shadow-xl shadow-emerald-600/30">
                <i className="fa-solid fa-mosque"></i>
              </div>
              <h2 className="text-2xl font-black text-slate-900 dark:text-white">SantriPay Login</h2>
              <p className="text-xs font-semibold text-emerald-600 dark:text-emerald-400 mt-1 px-2">{pesantren.nama}</p>
            </div>

            <form onSubmit={handleSubmit} className="space-y-4 text-xs">
              <div>
                <label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Username</label>
                <input type="text" required value={username} onChange={e => setUsername(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3.5 focus:ring-2 focus:ring-emerald-500 text-sm bg-slate-50 dark:bg-slate-800 text-slate-900 dark:text-white font-medium" placeholder="Masukkan username..." />
              </div>
              <div>
                <label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Password / PIN</label>
                <div className="relative">
                  <input type={showPassword ? "text" : "password"} required value={password} onChange={e => setPassword(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3.5 pr-12 focus:ring-2 focus:ring-emerald-500 text-sm bg-slate-50 dark:bg-slate-800 text-slate-900 dark:text-white font-medium" placeholder="Masukkan PIN / password..." />
                  <button type="button" onClick={() => setShowPassword(!showPassword)} className="absolute right-3.5 top-1/2 -translate-y-1/2 text-slate-400 hover:text-slate-600 dark:hover:text-white p-1">
                    <i className={`fa-solid ${showPassword ? 'fa-eye-slash' : 'fa-eye'} text-sm`}></i>
                  </button>
                </div>
              </div>
              <button type="submit" className="w-full bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-700 hover:to-teal-700 text-white font-extrabold py-4 rounded-2xl text-sm transition shadow-lg shadow-emerald-600/30 flex items-center justify-center space-x-2">
                <i className="fa-solid fa-right-to-bracket"></i>
                <span>Masuk ke Sistem Keuangan</span>
              </button>
            </form>
          </div>
        </div>
      );
    }

    function DashboardView({ totalPemasukan, totalPengeluaran, saldoKas, santriList, transaksiList, setActiveTab, pesantren }) {
      const chartRef = useRef(null);
      const chartInstance = useRef(null);

      useEffect(() => {
        if (chartRef.current) {
          if (chartInstance.current) chartInstance.current.destroy();
          const ctx = chartRef.current.getContext('2d');
          chartInstance.current = new Chart(ctx, {
            type: 'doughnut',
            data: {
              labels: ['Total Pemasukan', 'Total Pengeluaran'],
              datasets: [{ data: [totalPemasukan, totalPengeluaran], backgroundColor: ['#34d399', '#f43f5e'], borderWidth: 0, hoverOffset: 6 }]
            },
            options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom', labels: { color: document.documentElement.classList.contains('dark') ? '#e2e8f0' : '#334155', font: { weight: 'bold', size: 11 } } } } }
          });
        }
        return () => { if (chartInstance.current) chartInstance.current.destroy(); };
      }, [totalPemasukan, totalPengeluaran]);

      const activeSantriCount = santriList.filter(s => (s.status || 'aktif') === 'aktif').length;

      return (
        <div className="flex flex-col space-y-6">
          <div className="bg-gradient-to-r from-emerald-600 via-teal-600 to-cyan-700 rounded-3xl p-6 md:p-8 text-white shadow-2xl flex flex-col md:flex-row justify-between items-start md:items-center gap-4 relative overflow-hidden">
            <div className="absolute right-0 top-0 translate-x-10 -translate-y-10 w-64 h-64 bg-white/10 rounded-full blur-2xl pointer-events-none"></div>
            <div className="space-y-2 z-10">
              <span className="bg-white/20 px-3 py-1 rounded-full text-[11px] font-extrabold tracking-wider uppercase">Supabase Realtime Cloud</span>
              <h2 className="text-2xl md:text-3xl font-black tracking-tight">{pesantren.nama}</h2>
              <p className="text-xs text-emerald-100 max-w-xl font-medium">{pesantren.slogan || pesantren.alamat}</p>
            </div>
            <div className="flex gap-2.5 z-10">
              <button onClick={() => setActiveTab('pembayaran')} className="bg-white text-emerald-900 hover:bg-emerald-50 px-5 py-3 rounded-2xl text-xs font-black shadow-lg transition transform hover:-translate-y-0.5">
                <i className="fa-solid fa-plus-circle mr-1.5 text-emerald-600"></i> Entri Pembayaran
              </button>
            </div>
          </div>

          <div className="grid grid-cols-2 lg:grid-cols-4 gap-3 md:gap-5">
            <div className="bg-gradient-to-br from-emerald-900 via-teal-900 to-slate-900 p-4 md:p-6 rounded-3xl border border-emerald-500/30 shadow-xl shadow-emerald-950/20 flex flex-col justify-between group hover:border-emerald-400 transition">
              <div className="flex justify-between items-start">
                <span className="text-[10px] md:text-xs font-bold text-emerald-300 uppercase tracking-wider truncate">Saldo Kas</span>
                <div className="w-9 h-9 md:w-12 md:h-12 rounded-2xl bg-gradient-to-tr from-emerald-500 to-teal-400 text-white flex items-center justify-center text-sm md:text-xl shadow-md shadow-emerald-500/30 flex-shrink-0">
                  <i className="fa-solid fa-vault"></i>
                </div>
              </div>
              <h3 className="text-sm md:text-xl font-black text-white mt-3 truncate">Rp {saldoKas.toLocaleString('id-ID')}</h3>
            </div>

            <div className="bg-gradient-to-br from-blue-900 via-indigo-900 to-slate-900 p-4 md:p-6 rounded-3xl border border-blue-500/30 shadow-xl shadow-blue-950/20 flex flex-col justify-between group hover:border-blue-400 transition">
              <div className="flex justify-between items-start">
                <span className="text-[10px] md:text-xs font-bold text-blue-300 uppercase tracking-wider truncate">Santri Aktif</span>
                <div className="w-9 h-9 md:w-12 md:h-12 rounded-2xl bg-gradient-to-tr from-blue-500 to-indigo-500 text-white flex items-center justify-center text-sm md:text-xl shadow-md shadow-blue-500/30 flex-shrink-0">
                  <i className="fa-solid fa-user-graduate"></i>
                </div>
              </div>
              <h3 className="text-sm md:text-xl font-black text-white mt-3 truncate">{activeSantriCount} <span className="text-[10px] font-bold text-blue-300">Santri</span></h3>
            </div>

            <div className="bg-gradient-to-br from-teal-900 via-emerald-950 to-slate-900 p-4 md:p-6 rounded-3xl border border-teal-500/30 shadow-xl shadow-teal-950/20 flex flex-col justify-between group hover:border-teal-400 transition">
              <div className="flex justify-between items-start">
                <span className="text-[10px] md:text-xs font-bold text-teal-300 uppercase tracking-wider truncate">Pemasukan</span>
                <div className="w-9 h-9 md:w-12 md:h-12 rounded-2xl bg-gradient-to-tr from-teal-400 to-emerald-500 text-white flex items-center justify-center text-sm md:text-xl shadow-md shadow-teal-500/30 flex-shrink-0">
                  <i className="fa-solid fa-arrow-trend-up"></i>
                </div>
              </div>
              <h3 className="text-xs md:text-lg font-black text-emerald-400 mt-3 truncate">Rp {totalPemasukan.toLocaleString('id-ID')}</h3>
            </div>

            <div className="bg-gradient-to-br from-rose-950 via-pink-950 to-slate-900 p-4 md:p-6 rounded-3xl border border-rose-500/30 shadow-xl shadow-rose-950/20 flex flex-col justify-between group hover:border-rose-400 transition">
              <div className="flex justify-between items-start">
                <span className="text-[10px] md:text-xs font-bold text-rose-300 uppercase tracking-wider truncate">Pengeluaran</span>
                <div className="w-9 h-9 md:w-12 md:h-12 rounded-2xl bg-gradient-to-tr from-rose-500 to-pink-600 text-white flex items-center justify-center text-sm md:text-xl shadow-md shadow-rose-500/30 flex-shrink-0">
                  <i className="fa-solid fa-arrow-trend-down"></i>
                </div>
              </div>
              <h3 className="text-xs md:text-lg font-black text-rose-400 mt-3 truncate">Rp {totalPengeluaran.toLocaleString('id-ID')}</h3>
            </div>
          </div>

          <div className="grid grid-cols-1 lg:grid-cols-12 gap-6">
            <div className="lg:col-span-4 bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl">
              <h3 className="text-sm font-black text-slate-900 dark:text-white mb-4 border-b border-slate-200 dark:border-slate-700 pb-3 flex items-center justify-between">
                <span>Rasio Keuangan</span>
                <i className="fa-solid fa-chart-pie text-emerald-500"></i>
              </h3>
              <div className="h-64"><canvas ref={chartRef}></canvas></div>
            </div>

            <div className="lg:col-span-8 bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl">
              <div className="flex items-center justify-between mb-4 border-b border-slate-200 dark:border-slate-700 pb-3">
                <h3 className="text-sm font-black text-slate-900 dark:text-white">Transaksi Keuangan Terakhir</h3>
                <button onClick={() => setActiveTab('kas')} className="text-xs font-extrabold text-emerald-600 dark:text-emerald-400 hover:underline whitespace-nowrap">Lihat Semua &rarr;</button>
              </div>
              <div className="overflow-x-auto">
                <table className="w-full text-left text-xs min-w-[500px]">
                  <thead className="bg-slate-100 dark:bg-slate-900/80 text-slate-600 dark:text-slate-400 uppercase font-bold">
                    <tr><th className="p-3">Tanggal</th><th className="p-3">Uraian / Santri</th><th className="p-3">Kategori</th><th className="p-3 text-right">Jumlah</th></tr>
                  </thead>
                  <tbody className="divide-y divide-slate-200 dark:divide-slate-700/60">
                    {transaksiList.slice().reverse().slice(0, 6).map(tx => (
                      <tr key={tx.id} className="hover:bg-slate-50 dark:hover:bg-slate-700/50 transition">
                        <td className="p-3 text-slate-500 dark:text-slate-400 font-medium whitespace-nowrap">{tx.tanggal}</td>
                        <td className="p-3 font-bold text-slate-900 dark:text-white">{tx.namaSantri !== '-' ? tx.namaSantri : (tx.keterangan || 'Umum')}</td>
                        <td className="p-3 font-semibold text-slate-700 dark:text-slate-300">{tx.kategori}</td>
                        <td className={`p-3 text-right font-black whitespace-nowrap ${tx.jenis === 'masuk' ? 'text-emerald-600 dark:text-emerald-400' : 'text-rose-600 dark:text-rose-400'}`}>
                          {tx.jenis === 'masuk' ? '+' : '-'} Rp {Number(tx.jumlah).toLocaleString('id-ID')}
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>
      );
    }

    function PembayaranView({ supabaseClient, santriList, configTagihan, transaksiList, setTransaksiList, triggerPrintReceipt, showToast, selectedSantriForPayment, setSelectedSantriForPayment, currentUser, loadDataFromSupabase }) {
      const activeSantriList = santriList.filter(s => (s.status || 'aktif') === 'aktif');
      const [selectedSantriId, setSelectedSantriId] = useState(selectedSantriForPayment ? selectedSantriForPayment.id : '');
      const [kategori, setKategori] = useState('SPP & Syahriyah');
      const [bulan, setBulan] = useState('Agustus 2026');
      const [nominal, setNominal] = useState(configTagihan.sppBulanan);
      const [keterangan, setKeterangan] = useState('Pembayaran Lunas');

      useEffect(() => {
        if (selectedSantriForPayment) setSelectedSantriId(selectedSantriForPayment.id);
      }, [selectedSantriForPayment]);

      const selectedSantri = santriList.find(s => s.id === selectedSantriId);

      useEffect(() => {
        if (kategori === 'SPP & Syahriyah') setNominal(configTagihan.sppBulanan);
        else if (kategori === 'Uang Makan') setNominal(configTagihan.uangMakan);
        else if (kategori === 'Uang Kitab') setNominal(configTagihan.uangKitab);
        else if (kategori === 'Uang Gedung') setNominal(configTagihan.uangGedung);
        else if (kategori === 'Infaq / Donasi') setNominal(configTagihan.infaq || 50000);
        else if (kategori === 'Daftar Ulang') setNominal(configTagihan.daftarUlang || 750000);
        else setNominal(configTagihan.lainnya || 100000);
      }, [kategori, configTagihan]);

      const handleSaveOnly = async (e) => {
        e.preventDefault();
        await processPayment(false);
      };

      const handleSaveAndPrint = async (e) => {
        e.preventDefault();
        await processPayment(true);
      };

      const processPayment = async (shouldPrint) => {
        if (!selectedSantri) {
          showToast('Pilih santri terlebih dahulu!', 'error');
          return;
        }

        const now = new Date();
        const dateStr = now.toISOString().split('T')[0];
        const kwitansiNo = `KW-${now.getFullYear()}${String(now.getMonth()+1).padStart(2,'0')}${String(now.getDate()).padStart(2,'0')}-${Math.floor(100 + Math.random()*900)}`;

        const newTx = {
          id: 'TX' + Date.now(),
          tanggal: dateStr,
          jenis: 'masuk',
          kategori: kategori,
          santri_id: selectedSantri.id,
          nis: selectedSantri.nis,
          nama_santri: selectedSantri.nama,
          jumlah: Number(nominal),
          bulan: bulan,
          keterangan: keterangan,
          no_kwitansi: kwitansiNo,
          petugas: currentUser.name
        };

        if (supabaseClient) {
          const { error } = await supabaseClient.from('transaksi').insert([newTx]);
          if (error) {
            showToast('Gagal menyimpan ke Supabase: ' + error.message, 'error');
            return;
          }
        }

        const formattedTx = {
          id: newTx.id,
          tanggal: newTx.tanggal,
          jenis: newTx.jenis,
          kategori: newTx.kategori,
          santriId: newTx.santri_id,
          nis: newTx.nis,
          namaSantri: newTx.nama_santri,
          jumlah: newTx.jumlah,
          bulan: newTx.bulan,
          keterangan: newTx.keterangan,
          noKwitansi: newTx.no_kwitansi,
          petugas: newTx.petugas
        };

        setTransaksiList([...transaksiList, formattedTx]);
        loadDataFromSupabase();
        showToast('Pembayaran berhasil disimpan & tersinkronisasi!', 'success');
        if (setSelectedSantriForPayment) setSelectedSantriForPayment(null);
        if (shouldPrint) {
          triggerPrintReceipt(formattedTx);
        }
      };

      const santriHistory = transaksiList.filter(t => t.santriId);

      return (
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          <div className="lg:col-span-1 bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl h-fit">
            <h3 className="text-base font-black text-slate-900 dark:text-white mb-4 border-b border-slate-200 dark:border-slate-700 pb-3 flex items-center space-x-2">
              <i className="fa-solid fa-file-invoice-dollar text-emerald-500"></i>
              <span>Form Pembayaran Santri</span>
            </h3>
            <form className="space-y-4 text-xs">
              <div>
                <label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Pilih Santri</label>
                <select value={selectedSantriId} onChange={e => setSelectedSantriId(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold focus:ring-2 focus:ring-emerald-500" required>
                  <option value="">-- Pilih Santri --</option>
                  {activeSantriList.map(s => <option key={s.id} value={s.id}>{s.nama} ({s.nis}) - {s.kelas}</option>)}
                </select>
              </div>

              {selectedSantri && (
                <div className="bg-emerald-50 dark:bg-emerald-950/60 border border-emerald-200 dark:border-emerald-800 p-3.5 rounded-2xl text-[11px] text-emerald-800 dark:text-emerald-300 space-y-1 font-semibold">
                  <p><span className="font-bold">Kelas:</span> {selectedSantri.kelas} | <span className="font-bold">Kamar:</span> {selectedSantri.kamar}</p>
                  <p><span className="font-bold">Wali:</span> {selectedSantri.wali} ({selectedSantri.noHp})</p>
                </div>
              )}

              <div>
                <label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Jenis Pembayaran</label>
                <select value={kategori} onChange={e => setKategori(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold">
                  <option value="SPP &amp; Syahriyah">1. SPP &amp; Syahriyah Bulanan</option>
                  <option value="Infaq / Donasi">2. Infaq Sukarela Santri</option>
                  <option value="Daftar Ulang">3. Daftar Ulang / Herregistrasi</option>
                  <option value="Uang Makan">4. Uang Makan</option>
                  <option value="Uang Kitab">5. Uang Kitab &amp; Al-Qur'an</option>
                  <option value="Uang Gedung">6. Uang Gedung / Pembangunan</option>
                  <option value="Lainnya">7. Lainnya</option>
                </select>
              </div>

              <div className="grid grid-cols-1 sm:grid-cols-2 gap-2.5">
                <div>
                  <label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Periode Bulan</label>
                  <input type="text" value={bulan} onChange={e => setBulan(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" />
                </div>
                <div>
                  <label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Nominal (Rp)</label>
                  <input type="number" value={nominal} onChange={e => setNominal(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-black" required />
                </div>
              </div>

              <div>
                <label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Keterangan</label>
                <input type="text" value={keterangan} onChange={e => setKeterangan(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-medium" />
              </div>

              <div className="grid grid-cols-2 gap-2.5 pt-2">
                <button type="button" onClick={handleSaveOnly} className="bg-slate-900 hover:bg-slate-950 dark:bg-slate-700 dark:hover:bg-slate-600 text-white font-extrabold py-3.5 rounded-2xl transition shadow text-xs">
                  <i className="fa-solid fa-floppy-disk mr-1"></i> Simpan Saja
                </button>
                <button type="button" onClick={handleSaveAndPrint} className="bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-700 hover:to-teal-700 text-white font-extrabold py-3.5 rounded-2xl transition shadow-lg shadow-emerald-600/30 text-xs flex items-center justify-center space-x-1">
                  <i className="fa-solid fa-print"></i>
                  <span>Simpan &amp; Cetak</span>
                </button>
              </div>
            </form>
          </div>

          <div className="lg:col-span-2 bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl">
            <h3 className="text-base font-black text-slate-900 dark:text-white mb-4 border-b border-slate-200 dark:border-slate-700 pb-3">Riwayat Pembayaran Santri</h3>
            <div className="overflow-x-auto">
              <table className="w-full text-left text-xs min-w-[550px]">
                <thead className="bg-slate-100 dark:bg-slate-900/80 text-slate-600 dark:text-slate-400 uppercase font-bold">
                  <tr><th className="p-3">No Kwitansi</th><th className="p-3">Nama Santri</th><th className="p-3">Jenis</th><th className="p-3">Periode</th><th className="p-3">Jumlah</th><th className="p-3">Aksi</th></tr>
                </thead>
                <tbody className="divide-y divide-slate-200 dark:divide-slate-700/60">
                  {santriHistory.slice().reverse().map(tx => (
                    <tr key={tx.id} className="hover:bg-slate-50 dark:hover:bg-slate-700/50">
                      <td className="p-3 font-mono text-slate-500 dark:text-slate-400 font-semibold whitespace-nowrap">{tx.noKwitansi}</td>
                      <td className="p-3 font-black text-slate-900 dark:text-white">{tx.namaSantri}</td>
                      <td className="p-3 font-semibold text-slate-700 dark:text-slate-300">{tx.kategori}</td>
                      <td className="p-3 text-slate-500 dark:text-slate-400 font-medium whitespace-nowrap">{tx.bulan}</td>
                      <td className="p-3 font-black text-emerald-600 dark:text-emerald-400 whitespace-nowrap">Rp {Number(tx.jumlah).toLocaleString('id-ID')}</td>
                      <td className="p-3 whitespace-nowrap">
                        <button onClick={() => triggerPrintReceipt(tx)} className="bg-slate-200 dark:bg-slate-700 hover:bg-slate-300 dark:hover:bg-slate-600 text-slate-800 dark:text-white px-3 py-1.5 rounded-xl text-[11px] font-bold transition">
                          <i className="fa-solid fa-print mr-1"></i> Cetak
                        </button>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      );
    }

    function TunggakanView({ santriList, transaksiList, configTagihan, handleQuickPayFromTunggakan, handleSectionDownload }) {
      const activeSantri = santriList.filter(s => (s.status || 'aktif') === 'aktif');
      const targetBulan = "Agustus 2026";

      const santriTunggakan = activeSantri.map(s => {
        const hasPaidSPP = transaksiList.some(t => t.santriId === s.id && t.kategori === 'SPP & Syahriyah' && t.bulan === targetBulan);
        return { ...s, isLunasSPP: hasPaidSPP, tunggakanSPP: hasPaidSPP ? 0 : configTagihan.sppBulanan };
      });

      return (
        <div id="tunggakan-report-section" className="space-y-6">
          <div className="bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4">
            <div>
              <h3 className="text-base font-black text-slate-900 dark:text-white flex items-center space-x-2">
                <i className="fa-solid fa-triangle-exclamation text-amber-500"></i>
                <span>Laporan Tunggakan SPP &amp; Syahriyah ({targetBulan})</span>
              </h3>
            </div>
            <button onClick={() => handleSectionDownload('tunggakan-report-section')} className="w-full sm:w-auto bg-slate-900 hover:bg-slate-950 text-white px-5 py-3 rounded-2xl text-xs font-extrabold transition flex items-center justify-center space-x-2 shadow-lg border border-slate-700">
              <i className="fa-solid fa-download"></i>
              <span>Download PDF Laporan</span>
            </button>
          </div>

          <div className="bg-white dark:bg-slate-800/90 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl overflow-hidden">
            <div className="overflow-x-auto">
              <table className="w-full text-left text-xs min-w-[700px]">
                <thead className="bg-slate-100 dark:bg-slate-900/80 text-slate-600 dark:text-slate-400 uppercase font-bold">
                  <tr><th className="p-3.5">NIS</th><th className="p-3.5">Nama Santri</th><th className="p-3.5">Kelas / Kamar</th><th className="p-3.5">Wali</th><th className="p-3.5">Status SPP</th><th className="p-3.5 text-right">Tunggakan</th><th className="p-3.5 text-center">Aksi</th></tr>
                </thead>
                <tbody className="divide-y divide-slate-200 dark:divide-slate-700/60">
                  {santriTunggakan.map(s => (
                    <tr key={s.id} className="hover:bg-slate-50 dark:hover:bg-slate-700/50">
                      <td className="p-3.5 font-mono text-slate-500 dark:text-slate-400 font-semibold whitespace-nowrap">{s.nis}</td>
                      <td className="p-3.5 font-black text-slate-900 dark:text-white">{s.nama}</td>
                      <td className="p-3.5 text-slate-700 dark:text-slate-300 font-medium whitespace-nowrap">{s.kelas} | {s.kamar || '-'}</td>
                      <td className="p-3.5 text-slate-700 dark:text-slate-300 font-medium">{s.wali || '-'} ({s.noHp || '-'})</td>
                      <td className="p-3.5 whitespace-nowrap">
                        {s.isLunasSPP ? <span className="bg-emerald-100 dark:bg-emerald-950 text-emerald-800 dark:text-emerald-300 border border-emerald-300 dark:border-emerald-800 px-3 py-1 rounded-full text-[10px] font-black">LUNAS</span> : <span className="bg-rose-100 dark:bg-rose-950 text-rose-800 dark:text-rose-300 border border-rose-300 dark:border-rose-800 px-3 py-1 rounded-full text-[10px] font-black">BELUM LUNAS</span>}
                      </td>
                      <td className="p-3.5 text-right font-black text-slate-900 dark:text-white whitespace-nowrap">Rp {s.tunggakanSPP.toLocaleString('id-ID')}</td>
                      <td className="p-3.5 text-center whitespace-nowrap">
                        {!s.isLunasSPP && (
                          <button onClick={() => handleQuickPayFromTunggakan(s)} className="bg-gradient-to-r from-emerald-600 to-teal-600 text-white px-3.5 py-1.5 rounded-xl text-[11px] font-extrabold shadow hover:from-emerald-700 hover:to-teal-700">Bayar</button>
                        )}
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      );
    }

    function SantriView({ supabaseClient, santriList, setSantriList, showToast, loadDataFromSupabase, handleSectionDownload, currentUser }) {
      const [showModal, setShowModal] = useState(false);
      const [editingSantri, setEditingSantri] = useState(null);
      const [searchTerm, setSearchTerm] = useState('');
      const [formData, setFormData] = useState({ 
        nis: '', 
        nama: '', 
        kelas: 'Reguler', 
        alamat: '', 
        noHp: '', 
        status: 'aktif', 
        beasiswa: false 
      });

      const handleOpenAdd = () => {
        setEditingSantri(null);
        setFormData({ 
          nis: `202600${santriList.length + 1}`, 
          nama: '', 
          kelas: 'Reguler', 
          alamat: '', 
          noHp: '', 
          status: 'aktif', 
          beasiswa: false 
        });
        setShowModal(true);
      };

      const handleOpenEdit = (s) => {
        setEditingSantri(s);
        setFormData({ 
          nis: s.nis || '',
          nama: s.nama || '',
          kelas: s.kelas || 'Reguler',
          alamat: s.alamat || s.kamar || '',
          noHp: s.noHp || s.no_hp || '',
          status: s.status || 'aktif',
          beasiswa: s.beasiswa || false
        });
        setShowModal(true);
      };

      const handleDeleteSantri = async (id, nama) => {
        if (confirm(`Yakin ingin menghapus santri ${nama}?`)) {
          if (supabaseClient) {
            await supabaseClient.from('santri').delete().eq('id', id);
          }
          setSantriList(santriList.filter(s => s.id !== id));
          loadDataFromSupabase();
          showToast(`Santri ${nama} berhasil dihapus!`, 'info');
        }
      };

      const handleSave = async (e) => {
        e.preventDefault();
        if (editingSantri) {
          const payload = {
            nis: formData.nis,
            nama: formData.nama,
            kelas: formData.kelas,
            kamar: formData.alamat,
            wali: formData.alamat,
            no_hp: formData.noHp,
            beasiswa: formData.beasiswa,
            status: formData.status
          };
          if (supabaseClient) {
            await supabaseClient.from('santri').update(payload).eq('id', editingSantri.id);
          }
          setSantriList(santriList.map(s => s.id === editingSantri.id ? { 
            ...s, 
            nis: formData.nis,
            nama: formData.nama,
            kelas: formData.kelas,
            kamar: formData.alamat,
            wali: formData.alamat,
            noHp: formData.noHp,
            status: formData.status,
            beasiswa: formData.beasiswa
          } : s));
          loadDataFromSupabase();
          showToast(`Data santri ${formData.nama} diperbarui!`, 'success');
        } else {
          const newSantriRecord = {
            id: 'S' + Date.now(),
            nis: formData.nis,
            nama: formData.nama,
            kelas: formData.kelas,
            kamar: formData.alamat,
            wali: formData.alamat,
            no_hp: formData.noHp,
            beasiswa: formData.beasiswa,
            status: formData.status
          };
          if (supabaseClient) {
            await supabaseClient.from('santri').insert([newSantriRecord]);
          }
          const formatted = { 
            id: newSantriRecord.id, 
            nis: newSantriRecord.nis, 
            nama: newSantriRecord.nama, 
            kelas: newSantriRecord.kelas, 
            kamar: newSantriRecord.kamar, 
            wali: newSantriRecord.wali, 
            noHp: newSantriRecord.no_hp, 
            beasiswa: newSantriRecord.beasiswa, 
            status: newSantriRecord.status 
          };
          setSantriList([...santriList, formatted]);
          loadDataFromSupabase();
          showToast(`Santri baru ${formData.nama} ditambahkan!`, 'success');
        }
        setShowModal(false);
      };

      const filtered = santriList.filter(s => s.nama.toLowerCase().includes(searchTerm.toLowerCase()) || s.nis.includes(searchTerm));

      return (
        <div id="santri-list-section" className="space-y-6">
          <div className="bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl flex flex-col sm:flex-row items-center justify-between gap-4">
            <div className="w-full sm:w-80">
              <input type="text" placeholder="Cari nama atau NIS santri..." value={searchTerm} onChange={e => setSearchTerm(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 text-xs bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" />
            </div>
            <div className="flex items-center gap-2.5 w-full sm:w-auto">
              <button onClick={() => handleSectionDownload('santri-list-section')} className="flex-1 sm:flex-none bg-slate-900 hover:bg-slate-950 text-white px-4 py-3 rounded-2xl text-xs font-black shadow transition flex items-center justify-center space-x-2 border border-slate-700">
                <i className="fa-solid fa-download"></i>
                <span>Download PDF</span>
              </button>
              <button onClick={handleOpenAdd} className="flex-1 sm:flex-none bg-gradient-to-r from-emerald-600 to-teal-600 text-white px-5 py-3 rounded-2xl text-xs font-black shadow-lg shadow-emerald-600/30 flex items-center justify-center space-x-2">
                <i className="fa-solid fa-user-plus"></i>
                <span>Tambah Santri Baru</span>
              </button>
            </div>
          </div>

          <div className="bg-white dark:bg-slate-800/95 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl overflow-hidden">
            <div className="overflow-x-auto">
              <table className="w-full text-left text-xs min-w-[650px]">
                <thead className="bg-slate-100 dark:bg-slate-900/80 text-slate-600 dark:text-slate-400 uppercase font-bold">
                  <tr><th className="p-3.5">NIS</th><th className="p-3.5">Nama Santri</th><th className="p-3.5">Kelas</th><th className="p-3.5">Alamat &amp; HP</th><th className="p-3.5">Status / Beasiswa</th><th className="p-3.5 text-center">Aksi</th></tr>
                </thead>
                <tbody className="divide-y divide-slate-200 dark:divide-slate-700/60">
                  {filtered.map(s => (
                    <tr key={s.id} className="hover:bg-slate-50 dark:hover:bg-slate-700/50">
                      <td className="p-3.5 font-mono text-slate-500 dark:text-slate-400 font-semibold whitespace-nowrap">{s.nis}</td>
                      <td className="p-3.5 font-black text-slate-900 dark:text-white">{s.nama}</td>
                      <td className="p-3.5 text-slate-700 dark:text-slate-300 font-semibold whitespace-nowrap">
                        <span className="px-2.5 py-1 rounded-xl bg-slate-100 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 text-emerald-600 dark:text-emerald-400">{s.kelas || 'Reguler'}</span>
                      </td>
                      <td className="p-3.5 text-slate-700 dark:text-slate-300 font-medium">{s.kamar || s.wali || '-'}<br/><span className="text-[10px] text-slate-500 dark:text-slate-400">HP: {s.noHp || '-'}</span></td>
                      <td className="p-3.5 whitespace-nowrap space-x-1">
                        <span className={`px-2.5 py-1 rounded-full text-[10px] font-black ${s.status === 'aktif' ? 'bg-emerald-100 dark:bg-emerald-950 text-emerald-800 dark:text-emerald-300 border border-emerald-300 dark:border-emerald-800' : 'bg-rose-100 dark:bg-rose-950 text-rose-800 dark:text-rose-300 border border-rose-300 dark:border-rose-800'}`}>
                          {s.status === 'aktif' ? 'Aktif' : 'Nonaktif'}
                        </span>
                        {s.beasiswa && (
                          <span className="px-2.5 py-1 rounded-full text-[10px] font-black bg-blue-100 dark:bg-blue-950 text-blue-800 dark:text-blue-300 border border-blue-300 dark:border-blue-800">Beasiswa</span>
                        )}
                      </td>
                      <td className="p-3.5 text-center space-x-1.5 whitespace-nowrap">
                        <button onClick={() => handleOpenEdit(s)} className="bg-slate-200 dark:bg-slate-700 hover:bg-slate-300 dark:hover:bg-slate-600 text-slate-800 dark:text-white px-3 py-1.5 rounded-xl text-[11px] font-bold">Edit</button>
                        <button onClick={() => handleDeleteSantri(s.id, s.nama)} className="bg-rose-100 dark:bg-rose-950 hover:bg-rose-200 dark:hover:bg-rose-900 text-rose-700 dark:text-rose-300 px-3 py-1 rounded-xl text-[11px] font-bold">Hapus</button>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>

          {showModal && (
            <div className="fixed inset-0 bg-slate-950/80 backdrop-blur-sm z-50 flex items-center justify-center p-4">
              <div className="bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-3xl max-w-md w-full p-6 md:p-7 shadow-2xl space-y-4 max-h-[90vh] overflow-y-auto text-slate-900 dark:text-white">
                <h3 className="font-black text-sm border-b border-slate-200 dark:border-slate-700 pb-3">{editingSantri ? 'Edit Data Santri' : 'Tambah Santri Baru'}</h3>
                <form onSubmit={handleSave} className="space-y-3.5 text-xs">
                  <div>
                    <label className="block font-bold mb-1 text-slate-700 dark:text-slate-300">Nama Lengkap</label>
                    <input type="text" required value={formData.nama} onChange={e => setFormData({...formData, nama: e.target.value})} placeholder="Masukkan nama lengkap..." className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 font-semibold" />
                  </div>
                  <div>
                    <label className="block font-bold mb-1 text-slate-700 dark:text-slate-300">Kelas</label>
                    <select value={formData.kelas} onChange={e => setFormData({...formData, kelas: e.target.value})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 font-semibold">
                      <option value="Reguler">Reguler</option>
                      <option value="Tahasus">Tahasus</option>
                    </select>
                  </div>
                  <div>
                    <label className="block font-bold mb-1 text-slate-700 dark:text-slate-300">Alamat</label>
                    <input type="text" required value={formData.alamat} onChange={e => setFormData({...formData, alamat: e.target.value})} placeholder="Masukkan alamat lengkap..." className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 font-semibold" />
                  </div>
                  <div>
                    <label className="block font-bold mb-1 text-slate-700 dark:text-slate-300">No HP Ortu</label>
                    <input type="text" required value={formData.noHp} onChange={e => setFormData({...formData, noHp: e.target.value})} placeholder="Contoh: 08123456789" className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 font-semibold" />
                  </div>
                  <div className="grid grid-cols-2 gap-3">
                    <div>
                      <label className="block font-bold mb-1 text-slate-700 dark:text-slate-300">Status</label>
                      <select value={formData.status} onChange={e => setFormData({...formData, status: e.target.value})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 font-semibold">
                        <option value="aktif">Aktif</option>
                        <option value="nonaktif">Nonaktif</option>
                      </select>
                    </div>
                    <div className="flex items-center pt-5">
                      <label className="flex items-center space-x-2 cursor-pointer">
                        <input type="checkbox" checked={formData.beasiswa} onChange={e => setFormData({...formData, beasiswa: e.target.checked})} className="w-4 h-4 text-emerald-600 rounded border-slate-300 dark:border-slate-700 bg-slate-50 dark:bg-slate-900 focus:ring-emerald-500" />
                        <span className="font-bold text-slate-700 dark:text-slate-300">Beasiswa</span>
                      </label>
                    </div>
                  </div>
                  <div className="flex justify-end space-x-2.5 pt-3 border-t border-slate-200 dark:border-slate-700">
                    <button type="button" onClick={() => setShowModal(false)} className="px-5 py-2.5 border border-slate-300 dark:border-slate-700 rounded-2xl font-bold text-slate-700 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-700">Batal</button>
                    <button type="submit" className="px-5 py-2.5 bg-gradient-to-r from-emerald-600 to-teal-600 text-white rounded-2xl font-black shadow-md">Simpan</button>
                  </div>
                </form>
              </div>
            </div>
          )}
        </div>
      );
    }

    function KasView({ supabaseClient, transaksiList, setTransaksiList, showToast, currentUser, loadDataFromSupabase, handleSectionDownload }) {
      const [jenis, setJenis] = useState('masuk');
      const [kategori, setKategori] = useState('Pemasukan Operasional');
      const [jumlah, setJumlah] = useState('');
      const [keterangan, setKeterangan] = useState('');

      const isAdminOrPusat = currentUser.role === 'admin' || currentUser.role === 'bendahara_pusat';

      const handleSubmit = async (e) => {
        e.preventDefault();
        const now = new Date();
        const dateStr = now.toISOString().split('T')[0];
        const kwitansiNo = `KAS-${now.getFullYear()}${String(now.getMonth()+1).padStart(2,'0')}${String(now.getDate()).padStart(2,'0')}-${Math.floor(100 + Math.random()*900)}`;

        const newTx = {
          id: 'TX' + Date.now(),
          tanggal: dateStr,
          jenis,
          kategori,
          santri_id: null,
          nis: '-',
          nama_santri: '-',
          jumlah: Number(jumlah),
          bulan: '-',
          keterangan,
          no_kwitansi: kwitansiNo,
          petugas: currentUser.name
        };

        if (supabaseClient) {
          await supabaseClient.from('transaksi').insert([newTx]);
        }

        const formatted = {
          id: newTx.id,
          tanggal: newTx.tanggal,
          jenis: newTx.jenis,
          kategori: newTx.kategori,
          santriId: null,
          nis: '-',
          namaSantri: '-',
          jumlah: newTx.jumlah,
          bulan: '-',
          keterangan: newTx.keterangan,
          noKwitansi: newTx.no_kwitansi,
          petugas: newTx.petugas
        };

        setTransaksiList([...transaksiList, formatted]);
        loadDataFromSupabase();
        showToast('Transaksi kas berhasil dicatat!', 'success');
        setJumlah('');
        setKeterangan('');
      };

      return (
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          {isAdminOrPusat && (
            <div className="lg:col-span-1 bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl h-fit">
              <h3 className="text-base font-black text-slate-900 dark:text-white mb-4 border-b border-slate-200 dark:border-slate-700 pb-3 flex items-center space-x-2">
                <i className="fa-solid fa-wallet text-emerald-500"></i>
                <span>Pencatatan Kas Umum</span>
              </h3>
              <form onSubmit={handleSubmit} className="space-y-4 text-xs">
                <div>
                  <label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Jenis Kas</label>
                  <div className="grid grid-cols-2 gap-2">
                    <button type="button" onClick={() => { setJenis('masuk'); setKategori('Pemasukan Operasional'); }} className={`py-2.5 rounded-2xl font-black transition ${jenis === 'masuk' ? 'bg-emerald-600 text-white shadow-md shadow-emerald-600/30' : 'bg-slate-200 dark:bg-slate-900 text-slate-700 dark:text-slate-400'}`}>Masuk</button>
                    <button type="button" onClick={() => { setJenis('keluar'); setKategori('Pengeluaran Operasional'); }} className={`py-2.5 rounded-2xl font-black transition ${jenis === 'keluar' ? 'bg-rose-600 text-white shadow-md shadow-rose-600/30' : 'bg-slate-200 dark:bg-slate-900 text-slate-700 dark:text-slate-400'}`}>Keluar</button>
                  </div>
                </div>
                <div><label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Kategori</label><input type="text" required value={kategori} onChange={e => setKategori(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" /></div>
                <div><label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Nominal (Rp)</label><input type="number" required value={jumlah} onChange={e => setJumlah(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 font-black text-slate-900 dark:text-white bg-slate-50 dark:bg-slate-900" /></div>
                <div><label className="block font-bold text-slate-700 dark:text-slate-300 mb-1.5">Keterangan</label><textarea value={keterangan} onChange={e => setKeterangan(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-medium" rows="3"></textarea></div>
                <button type="submit" className="w-full bg-slate-900 hover:bg-slate-950 text-white font-black py-3.5 rounded-2xl shadow-xl border border-slate-700">Simpan Kas</button>
              </form>
            </div>
          )}
          <div id="kas-list-section" className={`${isAdminOrPusat ? 'lg:col-span-2' : 'lg:col-span-3'} bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl space-y-4`}>
            <div className="flex items-center justify-between border-b border-slate-200 dark:border-slate-700 pb-3">
              <h3 className="text-base font-black text-slate-900 dark:text-white">Riwayat Transaksi Buku Kas</h3>
              <button onClick={() => handleSectionDownload('kas-list-section')} className="bg-slate-200 dark:bg-slate-700 hover:bg-slate-300 dark:hover:bg-slate-600 text-slate-800 dark:text-white px-4 py-2 rounded-xl text-xs font-bold transition flex items-center space-x-1.5">
                <i className="fa-solid fa-download"></i>
                <span>Download PDF</span>
              </button>
            </div>
            <div className="overflow-x-auto">
              <table className="w-full text-left text-xs min-w-[500px]">
                <thead className="bg-slate-100 dark:bg-slate-900/80 text-slate-600 dark:text-slate-400 uppercase font-bold">
                  <tr><th className="p-3.5">Tanggal</th><th className="p-3.5">Uraian / Kategori</th><th className="p-3.5">Petugas</th><th className="p-3.5 text-right">Jumlah</th></tr>
                </thead>
                <tbody className="divide-y divide-slate-200 dark:divide-slate-700/60">
                  {transaksiList.slice().reverse().map(tx => (
                    <tr key={tx.id} className="hover:bg-slate-50 dark:hover:bg-slate-700/50">
                      <td className="p-3.5 text-slate-500 dark:text-slate-400 font-semibold whitespace-nowrap">{tx.tanggal}</td>
                      <td className="p-3.5 font-bold text-slate-900 dark:text-white">{tx.kategori} - <span className="font-normal text-slate-500 dark:text-slate-400">{tx.keterangan || '-'}</span></td>
                      <td className="p-3.5 text-slate-700 dark:text-slate-300 font-medium whitespace-nowrap">{tx.petugas || '-'}</td>
                      <td className={`p-3.5 text-right font-black whitespace-nowrap ${tx.jenis === 'masuk' ? 'text-emerald-600 dark:text-emerald-400' : 'text-rose-600 dark:text-rose-400'}`}>
                        {tx.jenis === 'masuk' ? '+' : '-'} Rp {Number(tx.jumlah).toLocaleString('id-ID')}
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      );
    }

    function LaporanView({ transaksiList, pesantren, handleSectionDownload }) {
      const totalMasuk = transaksiList.filter(t => t.jenis === 'masuk').reduce((s, t) => s + Number(t.jumlah), 0);
      const totalKeluar = transaksiList.filter(t => t.jenis === 'keluar').reduce((s, t) => s + Number(t.jumlah), 0);
      const saldoAkhir = totalMasuk - totalKeluar;

      return (
        <div id="laporan-section" className="space-y-6">
          <div className="bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
            <div>
              <h3 className="text-base font-black text-slate-900 dark:text-white">Laporan &amp; Rekapitulasi Keuangan Pesantren</h3>
              <p className="text-xs text-slate-500 dark:text-slate-400 font-medium">{pesantren.nama}</p>
            </div>
            <button onClick={() => handleSectionDownload('laporan-section')} className="w-full sm:w-auto bg-slate-900 hover:bg-slate-950 text-white px-5 py-3 rounded-2xl text-xs font-black shadow-lg border border-slate-700 flex items-center justify-center space-x-2">
              <i className="fa-solid fa-download"></i>
              <span>Download PDF Laporan</span>
            </button>
          </div>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-5">
            <div className="bg-gradient-to-br from-emerald-50 to-teal-50 dark:from-emerald-950 dark:to-teal-950 border border-emerald-200 dark:border-emerald-800 p-5 rounded-3xl shadow-xl"><p className="text-xs font-black text-emerald-700 dark:text-emerald-300 uppercase tracking-wide">Total Pemasukan</p><h4 className="text-xl font-black text-emerald-600 dark:text-emerald-400 mt-1">Rp {totalMasuk.toLocaleString('id-ID')}</h4></div>
            <div className="bg-gradient-to-br from-rose-50 to-pink-50 dark:from-rose-950 dark:to-pink-950 border border-rose-200 dark:border-rose-800 p-5 rounded-3xl shadow-xl"><p className="text-xs font-black text-rose-700 dark:text-rose-300 uppercase tracking-wide">Total Pengeluaran</p><h4 className="text-xl font-black text-rose-600 dark:text-rose-400 mt-1">Rp {totalKeluar.toLocaleString('id-ID')}</h4></div>
            <div className="bg-gradient-to-br from-slate-900 to-indigo-950 text-white border border-indigo-800 p-5 rounded-3xl shadow-xl"><p className="text-xs font-black text-indigo-300 uppercase tracking-wide">Saldo Akhir</p><h4 className="text-xl font-black text-white mt-1">Rp {saldoAkhir.toLocaleString('id-ID')}</h4></div>
          </div>
        </div>
      );
    }

    function AnalisisPengeluaranView({ transaksiList, handleSectionDownload }) {
      const pengeluaranList = transaksiList.filter(t => t.jenis === 'keluar');

      const uniqueMonths = Array.from(new Set(pengeluaranList.map(t => t.tanggal.substring(0, 7)))).sort().reverse();
      const currentMonthDefault = new Date().toISOString().substring(0, 7);
      
      const [selectedMonth, setSelectedMonth] = useState(uniqueMonths.length > 0 ? uniqueMonths[0] : currentMonthDefault);
      const [customStartDate, setCustomStartDate] = useState('');
      const [customEndDate, setCustomEndDate] = useState('');
      const [useCustomRange, setUseCustomRange] = useState(false);

      const filteredExpenses = pengeluaranList.filter(t => {
        if (useCustomRange) {
          if (!customStartDate || !customEndDate) return true;
          return t.tanggal >= customStartDate && t.tanggal <= customEndDate;
        } else {
          return t.tanggal.startsWith(selectedMonth);
        }
      });

      const totalFilteredExpense = filteredExpenses.reduce((sum, t) => sum + Number(t.jumlah), 0);

      let averageExpense = 0;
      let countLabel = 'Hari Aktif';
      if (useCustomRange && customStartDate && customEndDate) {
        const start = new Date(customStartDate);
        const end = new Date(customEndDate);
        const diffDays = Math.max(1, Math.ceil((end - start) / (1000 * 60 * 60 * 24)) + 1);
        averageExpense = totalFilteredExpense / diffDays;
        countLabel = `${diffDays} Hari`;
      } else {
        const [year, month] = selectedMonth.split('-');
        const daysInMonth = new Date(year, month, 0).getDate();
        averageExpense = totalFilteredExpense / daysInMonth;
        countLabel = `Rata-rata per Hari (${daysInMonth} Hari)`;
      }

      return (
        <div id="analisis-section" className="space-y-6">
          <div className="bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl flex flex-col md:flex-row items-start md:items-center justify-between gap-4">
            <div>
              <h3 className="text-base font-black text-slate-900 dark:text-white flex items-center space-x-2">
                <i className="fa-solid fa-chart-line text-rose-500"></i>
                <span>Analisis &amp; Rata-rata Pengeluaran Bulanan</span>
              </h3>
              <p className="text-xs text-slate-500 dark:text-slate-400 mt-0.5">Pilih bulan atau atur rentang tanggal custom untuk melihat kalkulasi rata-rata pengeluaran.</p>
            </div>
            <button onClick={() => handleSectionDownload('analisis-section')} className="w-full md:w-auto bg-slate-900 hover:bg-slate-950 text-white px-5 py-3 rounded-2xl text-xs font-black shadow-lg border border-slate-700 flex items-center justify-center space-x-2">
              <i className="fa-solid fa-download"></i>
              <span>Download PDF Analisis</span>
            </button>
          </div>

          <div className="bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl space-y-4">
            <div className="flex flex-col md:flex-row items-start md:items-center justify-between gap-4 border-b border-slate-200 dark:border-slate-700 pb-4">
              <div className="flex flex-wrap items-center gap-3">
                <label className="text-xs font-bold text-slate-700 dark:text-slate-300">Mode Filter:</label>
                <div className="flex bg-slate-100 dark:bg-slate-900 p-1 rounded-2xl">
                  <button type="button" onClick={() => setUseCustomRange(false)} className={`px-4 py-2 rounded-xl text-xs font-black transition ${!useCustomRange ? 'bg-emerald-600 text-white shadow' : 'text-slate-600 dark:text-slate-400'}`}>Pilih Bulan</button>
                  <button type="button" onClick={() => setUseCustomRange(true)} className={`px-4 py-2 rounded-xl text-xs font-black transition ${useCustomRange ? 'bg-emerald-600 text-white shadow' : 'text-slate-600 dark:text-slate-400'}`}>Rentang Custom</button>
                </div>
              </div>

              {!useCustomRange ? (
                <div className="flex items-center space-x-2">
                  <label className="text-xs font-bold text-slate-700 dark:text-slate-300">Bulan:</label>
                  <input type="month" value={selectedMonth} onChange={e => setSelectedMonth(e.target.value)} className="border border-slate-300 dark:border-slate-700 rounded-2xl p-2.5 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold text-xs" />
                </div>
              ) : (
                <div className="flex items-center space-x-2 flex-wrap gap-2">
                  <span className="text-xs font-bold text-slate-700 dark:text-slate-300">Dari:</span>
                  <input type="date" value={customStartDate} onChange={e => setCustomStartDate(e.target.value)} className="border border-slate-300 dark:border-slate-700 rounded-2xl p-2.5 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold text-xs" />
                  <span className="text-xs font-bold text-slate-700 dark:text-slate-300">Hingga:</span>
                  <input type="date" value={customEndDate} onChange={e => setCustomEndDate(e.target.value)} className="border border-slate-300 dark:border-slate-700 rounded-2xl p-2.5 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold text-xs" />
                </div>
              )}
            </div>

            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
              <div className="bg-gradient-to-br from-rose-50 to-pink-50 dark:from-rose-950/60 dark:to-pink-950/60 border border-rose-200 dark:border-rose-800 p-5 rounded-3xl shadow-md">
                <span className="text-xs font-extrabold text-rose-700 dark:text-rose-300 uppercase tracking-wide">Total Pengeluaran Periode Ini</span>
                <h4 className="text-2xl font-black text-rose-600 dark:text-rose-400 mt-2">Rp {totalFilteredExpense.toLocaleString('id-ID')}</h4>
                <p className="text-[11px] text-slate-500 dark:text-slate-400 mt-1">Jumlah total transaksi keluar tercatat.</p>
              </div>

              <div className="bg-gradient-to-br from-emerald-50 to-teal-50 dark:from-emerald-950/60 dark:to-teal-950/60 border border-emerald-200 dark:border-emerald-800 p-5 rounded-3xl shadow-md">
                <span className="text-xs font-extrabold text-emerald-700 dark:text-emerald-300 uppercase tracking-wide">Rata-rata Pengeluaran ({countLabel})</span>
                <h4 className="text-2xl font-black text-emerald-600 dark:text-emerald-400 mt-2">Rp {Math.round(averageExpense).toLocaleString('id-ID')}</h4>
                <p className="text-[11px] text-slate-500 dark:text-slate-400 mt-1">Estimasi pengeluaran rata-rata harian.</p>
              </div>
            </div>

            <div className="overflow-x-auto pt-2">
              <h4 className="text-sm font-black text-slate-900 dark:text-white mb-3">Rincian Pengeluaran Terpilih</h4>
              <table className="w-full text-left text-xs min-w-[500px]">
                <thead className="bg-slate-100 dark:bg-slate-900/80 text-slate-600 dark:text-slate-400 uppercase font-bold">
                  <tr><th className="p-3.5">Tanggal</th><th className="p-3.5">Kategori</th><th className="p-3.5">Keterangan</th><th className="p-3.5 text-right">Jumlah</th></tr>
                </thead>
                <tbody className="divide-y divide-slate-200 dark:divide-slate-700/60">
                  {filteredExpenses.length > 0 ? (
                    filteredExpenses.slice().reverse().map(tx => (
                      <tr key={tx.id} className="hover:bg-slate-50 dark:hover:bg-slate-700/50">
                        <td className="p-3.5 text-slate-500 dark:text-slate-400 font-semibold whitespace-nowrap">{tx.tanggal}</td>
                        <td className="p-3.5 font-bold text-slate-900 dark:text-white">{tx.kategori}</td>
                        <td className="p-3.5 text-slate-700 dark:text-slate-300">{tx.keterangan || '-'}</td>
                        <td className="p-3.5 text-right font-black text-rose-600 dark:text-rose-400 whitespace-nowrap">Rp {Number(tx.jumlah).toLocaleString('id-ID')}</td>
                      </tr>
                    ))
                  ) : (
                    <tr><td colSpan="4" className="p-6 text-center text-slate-400 italic">Tidak ada data pengeluaran pada periode ini.</td></tr>
                  )}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      );
    }

    function ProfilPesantrenView({ pesantren, setPesantren, showToast }) {
      const [form, setForm] = useState({ ...pesantren });

      const handleSave = (e) => {
        e.preventDefault();
        setPesantren(form);
        showToast('Profil Pesantren berhasil diperbarui!', 'success');
      };

      return (
        <div className="max-w-3xl">
          <form onSubmit={handleSave} className="bg-white dark:bg-slate-800/90 p-6 md:p-8 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl space-y-5 text-slate-900 dark:text-white">
            <h3 className="text-base font-black border-b border-slate-200 dark:border-slate-700 pb-3 flex items-center space-x-2">
              <i className="fa-solid fa-building-columns text-emerald-500"></i>
              <span>Pengaturan Identitas &amp; Profil Pesantren</span>
            </h3>
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4 text-xs">
              <div><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Nama Pesantren</label><input type="text" required value={form.nama} onChange={e => setForm({...form, nama: e.target.value})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" /></div>
              <div><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">No. Telepon / WhatsApp</label><input type="text" required value={form.telepon} onChange={e => setForm({...form, telepon: e.target.value})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" /></div>
              <div className="md:col-span-2"><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Alamat Lengkap</label><input type="text" required value={form.alamat} onChange={e => setForm({...form, alamat: e.target.value})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" /></div>
              <div><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Email Pesantren</label><input type="email" required value={form.email} onChange={e => setForm({...form, email: e.target.value})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" /></div>
              <div><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Slogan / Motto</label><input type="text" value={form.slogan} onChange={e => setForm({...form, slogan: e.target.value})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" /></div>
              <div><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Pimpinan / Pengasuh</label><input type="text" required value={form.pimpinan} onChange={e => setForm({...form, pimpinan: e.target.value})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" /></div>
              <div><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Bendahara Utama</label><input type="text" required value={form.bendahara} onChange={e => setForm({...form, bendahara: e.target.value})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" /></div>
            </div>
            <button type="submit" className="w-full sm:w-auto bg-gradient-to-r from-emerald-600 to-teal-600 text-white px-6 py-3.5 rounded-2xl text-xs font-black shadow-lg shadow-emerald-600/30">Simpan Profil Pesantren</button>
          </form>
        </div>
      );
    }

    function KelolaUserView({ users, setUsers, showToast, currentUser, onResetData }) {
      const [selectedUserId, setSelectedUserId] = useState(users[0]?.id || '');
      const [newPassword, setNewPassword] = useState('');
      
      const [newUsername, setNewUsername] = useState('');
      const [newPin, setNewPin] = useState('');
      const [newName, setNewName] = useState('');
      const [newRole, setNewRole] = useState('bendahara_pusat');

      const handleUpdatePin = (e) => {
        e.preventDefault();
        if (!newPassword) {
          showToast('Masukkan password/PIN baru!', 'error');
          return;
        }
        const updated = users.map(u => u.id === selectedUserId ? { ...u, password: newPassword } : u);
        setUsers(updated);
        setNewPassword('');
        showToast('PIN / Password akun berhasil diperbarui!', 'success');
      };

      const handleCreateUser = (e) => {
        e.preventDefault();
        if (!newUsername || !newPin || !newName) {
          showToast('Lengkapi semua kolom pembuatan akun!', 'error');
          return;
        }
        if (users.some(u => u.username === newUsername.trim())) {
          showToast('Username sudah terdaftar! Gunakan yang lain.', 'error');
          return;
        }

        const newUserObj = {
          id: 'u' + (users.length + 1) + Date.now(),
          username: newUsername.trim(),
          password: newPin,
          role: newRole,
          name: newName.trim()
        };

        setUsers([...users, newUserObj]);
        setNewUsername('');
        setNewPin('');
        setNewName('');
        showToast(`Akun baru untuk ${newRole === 'bendahara_pusat' ? 'Portal Pusat' : 'Portal Pesantren'} berhasil dibuat!`, 'success');
      };

      return (
        <div className="space-y-6 max-w-4xl">
          <div className="bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl text-slate-900 dark:text-white">
            <h3 className="text-base font-black mb-4 border-b border-slate-200 dark:border-slate-700 pb-3 flex items-center space-x-2">
              <i className="fa-solid fa-user-plus text-emerald-500"></i>
              <span>Buat Username &amp; Sandi Baru (Portal Pusat &amp; Pesantren)</span>
            </h3>
            <form onSubmit={handleCreateUser} className="grid grid-cols-1 md:grid-cols-2 gap-4 text-xs">
              <div>
                <label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Pilih Portal / Peran</label>
                <select value={newRole} onChange={e => setNewRole(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold">
                  <option value="bendahara_pusat">Bendahara / Petugas Portal Pusat</option>
                  <option value="bendahara_pesantren">Bendahara / Petugas Portal Pesantren</option>
                  <option value="admin">Administrator Utama</option>
                </select>
              </div>
              <div>
                <label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Nama Lengkap Petugas</label>
                <input type="text" required value={newName} onChange={e => setNewName(e.target.value)} placeholder="Contoh: Ust. Farhan, S.Pd" className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" />
              </div>
              <div>
                <label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Username Login</label>
                <input type="text" required value={newUsername} onChange={e => setNewUsername(e.target.value)} placeholder="Contoh: pusat_baru" className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" />
              </div>
              <div>
                <label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Sandi / PIN Baru</label>
                <input type="text" required value={newPin} onChange={e => setNewPin(e.target.value)} placeholder="Contoh: rahasia123" className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" />
              </div>
              <div className="md:col-span-2 pt-2">
                <button type="submit" className="w-full bg-gradient-to-r from-emerald-600 to-teal-600 text-white py-3 rounded-2xl font-black shadow-lg">Buat Akun Portal Baru</button>
              </div>
            </form>
          </div>

          <div className="bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl text-slate-900 dark:text-white">
            <h3 className="text-base font-black mb-4 border-b border-slate-200 dark:border-slate-700 pb-3 flex items-center space-x-2">
              <i className="fa-solid fa-key text-emerald-500"></i>
              <span>Perbarui PIN / Password Akun</span>
            </h3>
            <form onSubmit={handleUpdatePin} className="grid grid-cols-1 md:grid-cols-3 gap-4 text-xs items-end">
              <div>
                <label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Pilih Akun / User</label>
                <select value={selectedUserId} onChange={e => setSelectedUserId(e.target.value)} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold">
                  {users.map(u => <option key={u.id} value={u.id}>{u.name} ({u.username} - {u.role})</option>)}
                </select>
              </div>
              <div>
                <label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">PIN / Password Baru</label>
                <input type="text" required value={newPassword} onChange={e => setNewPassword(e.target.value)} placeholder="Masukkan PIN baru..." className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-semibold" />
              </div>
              <div>
                <button type="submit" className="w-full bg-gradient-to-r from-emerald-600 to-teal-600 text-white py-3 rounded-2xl font-black shadow-lg">Simpan PIN Baru</button>
              </div>
            </form>
          </div>

          <div className="bg-white dark:bg-slate-800/90 p-5 md:p-6 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl text-slate-900 dark:text-white">
            <h3 className="text-base font-black mb-4 border-b border-slate-200 dark:border-slate-700 pb-3">Daftar Hak Akses Petugas Aktif</h3>
            <div className="overflow-x-auto">
              <table className="w-full text-left text-xs min-w-[450px]">
                <thead className="bg-slate-100 dark:bg-slate-900/80 text-slate-600 dark:text-slate-400 uppercase font-bold"><tr><th className="p-3.5">Nama</th><th className="p-3.5">Username</th><th className="p-3.5">Role</th><th className="p-3.5">Sandi / PIN</th></tr></thead>
                <tbody className="divide-y divide-slate-200 dark:divide-slate-700/60">
                  {users.map(u => (
                    <tr key={u.id}>
                      <td className="p-3.5 font-black">{u.name}</td>
                      <td className="p-3.5 font-mono text-slate-500 dark:text-slate-300">{u.username}</td>
                      <td className="p-3.5 uppercase font-black text-emerald-600 dark:text-emerald-400">{u.role.replace('_', ' ')}</td>
                      <td className="p-3.5 font-mono text-slate-800 dark:text-slate-200 font-bold">{u.password}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>

          <div className="bg-gradient-to-br from-rose-950 to-slate-900 p-5 md:p-6 rounded-3xl border border-rose-800 shadow-xl text-white">
            <h3 className="text-base font-black mb-2 flex items-center space-x-2 text-rose-300">
              <i className="fa-solid fa-triangle-exclamation"></i>
              <span>Reset &amp; Kosongkan Data Aplikasi</span>
            </h3>
            <p className="text-xs text-slate-300 mb-4">Fitur ini akan menghapus seluruh data santri, transaksi, dan akun kustom untuk memulai pencatatan keuangan dari nol.</p>
            <button onClick={onResetData} className="bg-rose-600 hover:bg-rose-700 text-white px-5 py-3 rounded-2xl text-xs font-black shadow-lg transition">
              <i className="fa-solid fa-trash-can mr-2"></i> Kosongkan Semua Data (Mulai dari Nol)
            </button>
          </div>
        </div>
      );
    }

    function PengaturanView({ configTagihan, setConfigTagihan, showToast }) {
      const [cForm, setCForm] = useState({ ...configTagihan });

      const handleSave = (e) => {
        e.preventDefault();
        setConfigTagihan(cForm);
        showToast('Pengaturan tarif berhasil disimpan!', 'success');
      };

      return (
        <form onSubmit={handleSave} className="max-w-3xl bg-white dark:bg-slate-800/90 p-6 md:p-8 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl space-y-5 text-slate-900 dark:text-white">
          <h3 className="text-base font-black border-b border-slate-200 dark:border-slate-700 pb-3">Pengaturan Standar Tarif Tagihan Santri</h3>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4 text-xs">
            <div><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">SPP Bulanan (Rp)</label><input type="number" value={cForm.sppBulanan} onChange={e => setCForm({...cForm, sppBulanan: Number(e.target.value)})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-black" /></div>
            <div><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Uang Makan (Rp)</label><input type="number" value={cForm.uangMakan} onChange={e => setCForm({...cForm, uangMakan: Number(e.target.value)})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-black" /></div>
            <div><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Uang Kitab (Rp)</label><input type="number" value={cForm.uangKitab} onChange={e => setCForm({...cForm, uangKitab: Number(e.target.value)})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-black" /></div>
            <div><label className="block font-bold mb-1.5 text-slate-700 dark:text-slate-300">Uang Gedung (Rp)</label><input type="number" value={cForm.uangGedung} onChange={e => setCForm({...cForm, uangGedung: Number(e.target.value)})} className="w-full border border-slate-300 dark:border-slate-700 rounded-2xl p-3 bg-slate-50 dark:bg-slate-900 text-slate-900 dark:text-white font-black" /></div>
          </div>
          <button type="submit" className="w-full sm:w-auto bg-gradient-to-r from-emerald-600 to-teal-600 text-white px-6 py-3.5 rounded-2xl text-xs font-black shadow-lg shadow-emerald-600/30">Simpan Tarif Tagihan</button>
        </form>
      );
    }

    function DatabaseSqlView({ showToast }) {
      const sqlCode = `-- 1. Tabel Santri (Data Santri Pesantren)
CREATE TABLE IF NOT EXISTS santri (
    id TEXT PRIMARY KEY,
    nis VARCHAR(50) UNIQUE NOT NULL,
    nama VARCHAR(255) NOT NULL,
    kelas VARCHAR(100) NOT NULL,
    kamar VARCHAR(100) NOT NULL,
    wali VARCHAR(255) NOT NULL,
    no_hp VARCHAR(50) NOT NULL,
    beasiswa BOOLEAN DEFAULT FALSE,
    status VARCHAR(50) DEFAULT 'aktif',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 2. Tabel Transaksi (Terintegrasi secara foreign key ke tabel santri)
CREATE TABLE IF NOT EXISTS transaksi (
    id TEXT PRIMARY KEY,
    tanggal DATE NOT NULL,
    jenis VARCHAR(20) NOT NULL CHECK (jenis IN ('masuk', 'keluar')),
    kategori VARCHAR(100) NOT NULL,
    santri_id TEXT REFERENCES santri(id) ON DELETE CASCADE ON UPDATE CASCADE,
    nis VARCHAR(50) DEFAULT '-',
    nama_santri VARCHAR(255) DEFAULT '-',
    jumlah NUMERIC(15, 2) NOT NULL,
    bulan VARCHAR(50) DEFAULT '-',
    keterangan TEXT,
    no_kwitansi VARCHAR(100) UNIQUE NOT NULL,
    petugas VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);`;

      const handleCopy = () => {
        const textarea = document.createElement('textarea');
        textarea.value = sqlCode;
        document.body.appendChild(textarea);
        textarea.select();
        document.execCommand('copy');
        document.body.removeChild(textarea);
        showToast('Kode SQL berhasil disalin ke clipboard!', 'success');
      };

      return (
        <div className="bg-white dark:bg-slate-800/90 p-6 md:p-8 rounded-3xl border border-slate-200 dark:border-slate-700 shadow-xl space-y-4 max-w-4xl text-slate-900 dark:text-white">
          <div className="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3 border-b border-slate-200 dark:border-slate-700 pb-4">
            <div>
              <h3 className="text-base font-black flex items-center space-x-2">
                <i className="fa-solid fa-database text-emerald-500"></i>
                <span>Integrasi Database Supabase (SQL Setup)</span>
              </h3>
              <p className="text-xs text-slate-500 dark:text-slate-400 font-medium mt-0.5">Salin kode SQL di bawah ini dan jalankan di Supabase SQL Editor Anda.</p>
            </div>
            <button onClick={handleCopy} className="bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-700 hover:to-teal-700 text-white px-5 py-2.5 rounded-2xl text-xs font-black shadow-lg shadow-emerald-600/30 flex items-center space-x-2">
              <i className="fa-solid fa-copy"></i>
              <span>Salin Kode SQL</span>
            </button>
          </div>

          <div className="relative">
            <pre className="bg-slate-900 text-emerald-300 p-5 rounded-2xl font-mono text-xs overflow-x-auto max-h-[450px] shadow-inner leading-relaxed select-all">
              {sqlCode}
            </pre>
          </div>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
