<script>
  import { supabase } from '$lib/supabase';
  import { onMount } from 'svelte';

  let wisataList = $state([]);
  let umkmList = $state([]);
  let artikelList = $state([]);
  let loading = $state(true);
  
  let mapInstance; // Buat nampung peta biar ga dobel

  onMount(async () => {
    // Tarik data Wisata (semua wisata biar masuk ke peta)
    const { data: dataWisata } = await supabase.from('wisata').select('*').order('created_at', { ascending: false });
    if (dataWisata) wisataList = dataWisata;

    // Tarik data UMKM (limit 4 terbaru)
    const { data: dataUmkm } = await supabase.from('umkm').select('*').eq('status', 'approved').order('created_at', { ascending: false }).limit(4);
    if (dataUmkm) umkmList = dataUmkm;

    // Tarik data Artikel (limit 3 terbaru)
    const { data: dataArtikel } = await supabase.from('artikel').select('*').order('created_at', { ascending: false }).limit(3);
    if (dataArtikel) artikelList = dataArtikel;

    loading = false;

    // Tunggu sampe library peta (Leaflet) ke-load di browser
    const initMap = setInterval(() => {
      if (window.L && document.getElementById('map') && !mapInstance) {
        clearInterval(initMap);
        
        // 1. Bikin peta, fokus ke koordinat Nagari Sirukam (zoom level 13)
        mapInstance = window.L.map('map').setView([-0.890719, 100.756278], 12);
        
        // 2. Pakai desain peta gratis (OpenStreetMap)
        window.L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
          maxZoom: 19,
          attribution: '© OpenStreetMap'
        }).addTo(mapInstance);

        // 3. Looping data wisata, sebar pin ke peta!
        wisataList.forEach(w => {
          if(w.latitude && w.longitude) {
            window.L.marker([w.latitude, w.longitude]).addTo(mapInstance)
              .bindPopup(`
                <div class="text-center">
                  <b class="text-sm">${w.nama_tempat}</b><br>
                  <a href="https://www.google.com/maps/dir/?api=1&destination=${w.latitude},${w.longitude}" target="_blank" class="text-xs text-blue-600 font-bold hover:underline">Google Maps &rarr;</a>
                </div>
              `);
          }
        });
      }
    }, 200);
  });
</script>

<!-- MANTRA WAJIB BUAT PETA GRATISAN -->
<svelte:head>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
</svelte:head>

<!-- NAVBAR (Elegan & Modern) -->
<nav class="fixed top-0 left-0 w-full bg-white/95 backdrop-blur-sm border-b border-gray-100 z-50">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex justify-between items-center h-20">
      <!-- Logo -->
      <div class="flex-shrink-0">
        <a href="#profil" class="text-2xl font-bold text-slate-800 tracking-tight">
          Sirukam<span class="text-emerald-600">Smart.</span>
        </a>
      </div>
      
      <!-- Menu Navigasi -->
      <div class="hidden md:flex space-x-8">
        <a href="#profil" class="text-slate-600 hover:text-emerald-600 font-medium transition">Profil</a>
        <a href="#wisata" class="text-slate-600 hover:text-emerald-600 font-medium transition">Wisata</a>
        <a href="#umkm" class="text-slate-600 hover:text-emerald-600 font-medium transition">UMKM</a>
        <a href="#berita" class="text-slate-600 hover:text-emerald-600 font-medium transition">Berita</a>
      </div>

      <!-- Tombol Kontak -->
      <div class="hidden md:block">
        <a href="#kontak" class="bg-emerald-700 hover:bg-emerald-800 text-white px-6 py-2.5 rounded-md font-medium transition shadow-sm">
          Hubungi Kami
        </a>
      </div>
    </div>
  </div>
</nav>

<!-- SECTION: PROFIL -->
<section id="profil" class="pt-20">
  <!-- 1. Hero Banner -->
  <div class="relative h-[80vh] flex items-center justify-center bg-slate-900">
    <div class="absolute inset-0">
      <img src="https://images.unsplash.com/photo-1542385151-efd9000785a0?q=80&w=2000&auto=format&fit=crop" class="w-full h-full object-cover opacity-40" alt="Jorong Lubuak Pulai" />
    </div>
    <div class="relative z-10 text-center px-4 max-w-4xl mx-auto">
      <p class="text-sm md:text-base text-emerald-400 font-semibold mb-4 tracking-widest uppercase">Satu Klik untuk Mengenal Jorong Lubuak Pulai</p>
      <h1 class="text-4xl md:text-5xl lg:text-6xl font-bold text-white mb-6 leading-tight drop-shadow-lg">Website Resmi <br /><span class="text-emerald-400">Jorong Lubuak Pulai</span></h1>
      <p class="text-lg md:text-xl text-gray-300 mb-10 font-light drop-shadow-md">"Mengenal Potensi, Budaya, dan Kehidupan Masyarakat Nagari Sirukam secara Digital."</p>
      <div class="flex flex-col sm:flex-row justify-center gap-4">
        <a href="#wisata" class="bg-emerald-600 hover:bg-emerald-700 text-white font-medium py-3 px-8 rounded-md transition shadow-md">Jelajahi Wisata</a>
        <a href="#umkm" class="bg-white/10 hover:bg-white/20 backdrop-blur-sm border border-white/30 text-white font-medium py-3 px-8 rounded-md transition">Lihat UMKM</a>
      </div>
    </div>
  </div>

  <!-- 2. Konten Profil -->
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 -mt-12 relative z-20">
    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 md:gap-6 bg-white rounded-xl shadow-xl p-6 md:p-8 border border-gray-100">
      <div class="text-center"><div class="text-3xl md:text-4xl font-bold text-slate-800">1.200+</div><div class="text-sm text-slate-500 mt-1">Penduduk</div></div>
      <div class="text-center border-l border-gray-100"><div class="text-3xl md:text-4xl font-bold text-slate-800">450</div><div class="text-sm text-slate-500 mt-1">Kepala Keluarga</div></div>
      <div class="text-center border-l border-gray-100"><div class="text-3xl md:text-4xl font-bold text-slate-800">30+</div><div class="text-sm text-slate-500 mt-1">UMKM Aktif</div></div>
      <div class="text-center border-l border-gray-100"><div class="text-3xl md:text-4xl font-bold text-slate-800">5</div><div class="text-sm text-slate-500 mt-1">Destinasi Wisata</div></div>
    </div>

    <!-- Sambutan -->
    <div class="mt-20 mb-20 grid grid-cols-1 md:grid-cols-2 gap-12 items-center">
      <div>
        <div class="w-16 h-1 bg-emerald-500 mb-6"></div>
        <h2 class="text-3xl font-bold text-slate-800 mb-6">Sambutan Kepala Jorong</h2>
        <p class="text-slate-600 leading-relaxed mb-4 text-justify">"Selamat datang di portal resmi digital Jorong Lubuak Pulai. Kami berharap platform ini dapat menjadi jembatan informasi yang kuat untuk menghubungkan masyarakat, pelaku usaha UMKM, dan wisatawan dari berbagai daerah."</p>
        <p class="text-slate-600 leading-relaxed text-justify">Dengan semangat gotong royong, mari kita wujudkan Lubuak Pulai yang maju, mandiri, dan berbasis teknologi, tanpa sedikitpun meninggalkan nilai-nilai adat dan tradisi budaya leluhur kita.</p>
        <div class="mt-8">
          <div class="font-bold text-slate-800 text-lg">Bapak Nama Kades</div>
          <div class="text-emerald-600 text-sm font-medium">Kepala Jorong Lubuak Pulai</div>
        </div>
      </div>
      <div class="rounded-xl overflow-hidden shadow-lg h-96 bg-gray-200">
        <img src="https://images.unsplash.com/photo-1577563908411-5077b6dc7624?q=80&w=800&auto=format&fit=crop" class="w-full h-full object-cover" alt="Balai Desa" />
      </div>
    </div>
  </div>
</section>

{#if loading}
  <div class="text-center py-20 text-emerald-600 font-bold animate-pulse text-xl">Memuat data dari brankas Supabase...</div>
{:else}

  <!-- ================= SECTION: WISATA (PETA & LIST HOVER) ================= -->
  <section id="wisata" class="py-20 bg-slate-50 border-t border-gray-200 relative">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="text-center mb-12">
        <div class="w-16 h-1 bg-emerald-500 mx-auto mb-4"></div>
        <h2 class="text-3xl font-bold text-slate-800">Peta Destinasi Wisata</h2>
        <p class="text-slate-500 mt-2">Jelajahi surga tersembunyi Sirukam langsung dari peta di bawah ini</p>
      </div>
      
      <!-- Grid 2 Kolom: Kiri Peta, Kanan List -->
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        
        <!-- PETA (Kiri, ambil 2/3 layar) -->
        <div class="lg:col-span-2 h-[500px] bg-white rounded-xl shadow-md border border-slate-200 overflow-hidden relative z-0">
          <div id="map" class="w-full h-full z-0"></div>
        </div>

        <!-- DAFTAR WISATA (Kanan, List Hover Mekar) -->
        <div class="lg:col-span-1 flex flex-col gap-4 h-[500px] overflow-y-auto pr-2 custom-scrollbar">
          {#each wisataList as wisata}
            <!-- Container Utama Item List (Class "group" ini kuncinya) -->
            <div class="group bg-white rounded-xl shadow-sm border border-slate-200 p-5 cursor-pointer hover:border-emerald-500 hover:shadow-md transition-all duration-300">
              
              <!-- Judul yang selalu keliatan -->
              <div class="flex justify-between items-center">
                <h3 class="font-bold text-lg text-slate-800">{wisata.nama_tempat}</h3>
                <span class="text-emerald-500 group-hover:rotate-180 transition-transform duration-300">▼</span>
              </div>
              <p class="text-xs text-slate-500 mt-1 flex items-center gap-1">📍 {wisata.lokasi}</p>
              
              <!-- Konten Sembunyi (Mekar pas di hover) -->
              <!-- Pakai grid-rows-[0fr] jadi grid-rows-[1fr] buat efek transisi yang super mulus -->
              <div class="grid grid-rows-[0fr] group-hover:grid-rows-[1fr] transition-all duration-500 ease-in-out">
                <div class="overflow-hidden">
                  <div class="pt-4 mt-3 border-t border-slate-100">
                    <img src={wisata.foto_url} loading="lazy" alt={wisata.nama_tempat} class="w-full h-32 object-cover rounded-lg mb-3" />
                    <p class="text-sm text-slate-600 line-clamp-3 mb-3">{wisata.deskripsi}</p>
                    <a href="https://www.google.com/maps/dir/?api=1&destination={wisata.latitude},{wisata.longitude}" target="_blank" class="inline-block text-xs font-bold text-blue-600 hover:underline">
                      Rute Maps &rarr;
                    </a>
                  </div>
                </div>
              </div>

            </div>
          {:else}
            <p class="text-slate-500 italic text-center py-10">Belum ada destinasi wisata.</p>
          {/each}
        </div>

      </div>
    </div>
  </section>

  <!-- ================= SECTION: UMKM ================= -->
  <section id="umkm" class="py-20 bg-white border-t border-gray-200">
    <!-- (Isi UMKM sama persis kayak yang lu desain) -->
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="text-center mb-12">
        <div class="w-16 h-1 bg-emerald-500 mx-auto mb-4"></div>
        <h2 class="text-3xl font-bold text-slate-800">Lapak Warga (UMKM)</h2>
        <p class="text-slate-500 mt-2">Dukung perekonomian lokal dengan membeli produk warga kami</p>
      </div>

      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
        {#each umkmList as umkm}
          <div class="bg-slate-50 p-6 rounded-xl border border-gray-100 hover:shadow-lg transition duration-300 group">
            <span class="inline-block px-3 py-1 bg-emerald-100 text-emerald-700 text-xs font-bold rounded-full mb-4">{umkm.kategori}</span>
            <h3 class="font-bold text-lg text-slate-800 mb-1 group-hover:text-emerald-600 transition">{umkm.nama_usaha}</h3>
            <p class="text-slate-600 font-medium mb-6">{umkm.harga}</p>
            <a href="https://wa.me/{umkm.nomor_wa.replace(/^0/, '62')}" target="_blank" class="flex items-center justify-center gap-2 w-full bg-emerald-600 text-white py-2.5 rounded shadow-sm hover:bg-emerald-700 transition">
              Beli via WhatsApp
            </a>
          </div>
        {/each}
      </div>
    </div>
  </section>

  <!-- ================= SECTION: BERITA ================= -->
  <section id="berita" class="py-20 bg-slate-50 border-t border-gray-200">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="text-center mb-12">
        <div class="w-16 h-1 bg-emerald-500 mx-auto mb-4"></div>
        <h2 class="text-3xl font-bold text-slate-800">Berita & Informasi</h2>
        <p class="text-slate-500 mt-2">Kabar terbaru langsung dari Balai Nagari</p>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
        {#each artikelList as artikel}
          <div class="bg-white rounded-xl shadow-md overflow-hidden flex flex-col hover:-translate-y-1 transition duration-300">
            <img src={artikel.foto_url} loading="lazy" alt={artikel.judul} class="w-full h-48 object-cover bg-slate-100" />
            <div class="p-6 flex-1 flex flex-col">
              <h3 class="font-bold text-lg text-slate-800 mb-2 leading-snug">{artikel.judul}</h3>
              <p class="text-xs text-slate-400 mb-4 font-medium uppercase tracking-wider">Oleh: {artikel.penulis}</p>
              <p class="text-sm text-slate-600 line-clamp-3 mb-6">{artikel.konten}</p>
            </div>
          </div>
        {/each}
      </div>
    </div>
  </section>

{/if}

<!-- FOOTER -->
<footer id="kontak" class="bg-slate-900 text-slate-300 py-12 border-t border-slate-800">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex flex-col md:flex-row justify-between items-center gap-6">
      <div class="text-center md:text-left">
        <div class="text-2xl font-bold text-white tracking-tight mb-2">Sirukam<span class="text-emerald-500">Smart.</span></div>
        <p class="text-sm text-slate-400">Pusat Informasi & Potensi Nagari Sirukam, Kabupaten Solok.</p>
      </div>
      <div class="text-sm text-center md:text-right text-slate-500">
        &copy; 2026 Pemerintah Nagari Sirukam.<br>Dikelola oleh Admin Desa.
      </div>
    </div>
  </div>
</footer>

<style>
  /* Biar scrollbar di list wisata kanannya cakep */
  .custom-scrollbar::-webkit-scrollbar {
    width: 6px;
  }
  .custom-scrollbar::-webkit-scrollbar-track {
    background: #f1f5f9;
    border-radius: 10px;
  }
  .custom-scrollbar::-webkit-scrollbar-thumb {
    background: #cbd5e1;
    border-radius: 10px;
  }
  .custom-scrollbar::-webkit-scrollbar-thumb:hover {
    background: #94a3b8;
  }
</style>