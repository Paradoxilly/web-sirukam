<script>
  import { supabase } from '$lib/supabase';
  import { onMount } from 'svelte';

  let wisataList = $state([]);
  let umkmList = $state([]);
  let artikelList = $state([]);
  let loading = $state(true);
  
  let mapInstance; 

  // Fungsi Pintar Format Rupiah
  function formatRupiah(harga) {
    if (!harga) return '';
    if (/^\d+$/.test(harga.toString().trim())) {
      return new Intl.NumberFormat('id-ID', {
        style: 'currency',
        currency: 'IDR',
        minimumFractionDigits: 0
      }).format(harga);
    }
    return harga;
  }

  onMount(async () => {
    const { data: dataWisata } = await supabase.from('wisata').select('*').order('created_at', { ascending: false });
    if (dataWisata) wisataList = dataWisata;

    const { data: dataUmkm } = await supabase.from('umkm').select('*').eq('status', 'approved').order('created_at', { ascending: false }).limit(4);
    if (dataUmkm) umkmList = dataUmkm;

    const { data: dataArtikel } = await supabase.from('artikel').select('*').order('created_at', { ascending: false }).limit(3);
    if (dataArtikel) artikelList = dataArtikel;

    loading = false;

    const initMap = setInterval(() => {
      if (window.L && document.getElementById('map') && !mapInstance) {
        clearInterval(initMap);
        
        mapInstance = window.L.map('map').setView([-0.890719, 100.756278], 12);
        
        window.L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
          maxZoom: 19,
          attribution: '© OpenStreetMap'
        }).addTo(mapInstance);

        wisataList.forEach(w => {
          if(w.latitude && w.longitude) {
            window.L.marker([w.latitude, w.longitude]).addTo(mapInstance)
              .bindPopup(`
                <div class="text-center p-1">
                  <b class="text-sm text-slate-800">${w.nama_tempat}</b><br>
                  <a href="https://www.google.com/maps/dir/?api=1&destination=${w.latitude},${w.longitude}" target="_blank" class="mt-2 inline-block text-xs text-blue-600 font-bold hover:underline">Rute Google Maps &rarr;</a>
                </div>
              `);
          }
        });
      }
    }, 200);
  });
</script>

<svelte:head>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
</svelte:head>

<!-- NAVBAR (Elegan & Modern) -->
<nav class="fixed top-0 left-0 w-full bg-white/90 backdrop-blur-md border-b border-gray-100 z-50 transition-all duration-300">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex justify-between items-center h-20">
      <div class="flex-shrink-0">
        <a href="#profil" class="text-2xl font-bold text-slate-800 tracking-tight flex items-center gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-emerald-600"><path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/></svg>
          Sirukam<span class="text-emerald-600">Smart.</span>
        </a>
      </div>
      
      <div class="hidden md:flex space-x-8">
        <a href="#profil" class="text-sm font-bold text-slate-500 hover:text-emerald-600 transition">Profil</a>
        <a href="#wisata" class="text-sm font-bold text-slate-500 hover:text-emerald-600 transition">Wisata</a>
        <a href="#umkm" class="text-sm font-bold text-slate-500 hover:text-emerald-600 transition">UMKM</a>
        <a href="#berita" class="text-sm font-bold text-slate-500 hover:text-emerald-600 transition">Berita</a>
      </div>

      <div class="hidden md:block">
        <a href="#kontak" class="flex items-center gap-2 bg-slate-900 hover:bg-emerald-600 text-white px-6 py-2.5 rounded-xl font-bold transition-all duration-300 shadow-sm hover:shadow-md hover:-translate-y-0.5">
          Hubungi Kami
        </a>
      </div>
    </div>
  </div>
</nav>

<!-- SECTION: PROFIL -->
<section id="profil" class="pt-20">
<!-- 1. Hero Banner -->
  <div class="relative min-h-[85vh] flex items-center justify-center bg-slate-900">
    <div class="absolute inset-0">
      <!-- Efek horor (mix-blend) dibuang, gambar dibikin lebih terang (opacity-50) -->
      <img src="https://knurnsmwprpdfhwenbpo.supabase.co/storage/v1/object/public/Asset/banner.jpg" class="w-full h-full object-cover opacity-50" alt="Jorong Lubuak Pulai" />
      <!-- Gradient dibikin nyapu dari bawah ke atas biar smooth -->
      <div class="absolute inset-0 bg-gradient-to-t from-slate-900 via-slate-900/40 to-slate-900/20"></div>
    </div>
    
    <!-- Tambahin margin bottom (mb-16) biar kontennya naik dan tombolnya ngehindar dari kotak putih -->
    <div class="relative z-10 text-center px-4 max-w-4xl mx-auto mb-16 mt-10">
      <p class="text-xs md:text-sm text-emerald-400 font-bold mb-4 tracking-[0.2em] uppercase flex items-center justify-center gap-2">
        <span class="w-8 h-px bg-emerald-400"></span> Satu Klik Mengenal Desa <span class="w-8 h-px bg-emerald-400"></span>
      </p>
      <h1 class="text-4xl md:text-6xl lg:text-7xl font-extrabold text-white mb-6 leading-tight drop-shadow-2xl">
        Website<br /> Wisata & UMKM <br /><span class="text-emerald-400">Nagari Sirukam</span>
      </h1>
      <p class="text-lg md:text-xl text-gray-200 mb-10 font-medium drop-shadow-md max-w-2xl mx-auto">
        "Mengenal Potensi, Budaya, dan Kehidupan Masyarakat Nagari Sirukam secara Digital."
      </p>
      
      <!-- Tombol CTA Utama -->
      <div class="flex flex-col sm:flex-row justify-center gap-4">
        <a href="#wisata" class="flex items-center justify-center gap-2 bg-gradient-to-r from-emerald-500 to-teal-500 hover:from-emerald-600 hover:to-teal-600 text-white font-bold py-3.5 px-8 rounded-2xl transition-all duration-300 hover:scale-105 shadow-lg hover:shadow-emerald-500/30">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="m8 3 4 8 5-5 5 15H2L8 3z"/></svg>
          Jelajahi Wisata
        </a>
        <a href="#umkm" class="flex items-center justify-center gap-2 bg-white/10 hover:bg-white/20 backdrop-blur-md border border-white/20 text-white font-bold py-3.5 px-8 rounded-2xl transition-all duration-300 hover:scale-105">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M2 9a3 3 0 0 1 0 6v2a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-2a3 3 0 0 1 0-6V7a2 2 0 0 0-2-2H4a2 2 0 0 0-2 2Z"/><path d="M13 18H7"/><path d="M7 14h.01"/></svg>
          Lihat UMKM
        </a>
      </div>
    </div>
  </div>

  <!-- 2. Konten Profil -->
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 -mt-16 relative z-20">
    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 md:gap-0 bg-white rounded-3xl shadow-xl p-4 md:p-8 border border-slate-100">
      <div class="text-center p-4">
        <div class="mx-auto w-12 h-12 bg-emerald-50 text-emerald-600 rounded-2xl flex items-center justify-center mb-3"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M16 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M22 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg></div>
        <div class="text-3xl md:text-4xl font-extrabold text-slate-800">1.200+</div><div class="text-xs md:text-sm font-bold text-slate-400 mt-1 uppercase tracking-wider">Penduduk</div>
      </div>
      <div class="text-center p-4 md:border-l border-slate-100">
        <div class="mx-auto w-12 h-12 bg-blue-50 text-blue-600 rounded-2xl flex items-center justify-center mb-3"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m3 9 9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/></svg></div>
        <div class="text-3xl md:text-4xl font-extrabold text-slate-800">321</div><div class="text-xs md:text-sm font-bold text-slate-400 mt-1 uppercase tracking-wider">Kepala Keluarga</div>
      </div>
      <div class="text-center p-4 border-t md:border-t-0 md:border-l border-slate-100">
        <div class="mx-auto w-12 h-12 bg-amber-50 text-amber-600 rounded-2xl flex items-center justify-center mb-3"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M2 9a3 3 0 0 1 0 6v2a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-2a3 3 0 0 1 0-6V7a2 2 0 0 0-2-2H4a2 2 0 0 0-2 2Z"/><path d="M13 18H7"/><path d="M7 14h.01"/></svg></div>
        <div class="text-3xl md:text-4xl font-extrabold text-slate-800">30+</div><div class="text-xs md:text-sm font-bold text-slate-400 mt-1 uppercase tracking-wider">UMKM Aktif</div>
      </div>
      <div class="text-center p-4 border-t md:border-t-0 md:border-l border-slate-100">
        <div class="mx-auto w-12 h-12 bg-purple-50 text-purple-600 rounded-2xl flex items-center justify-center mb-3"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/></svg></div>
        <div class="text-3xl md:text-4xl font-extrabold text-slate-800">5</div><div class="text-xs md:text-sm font-bold text-slate-400 mt-1 uppercase tracking-wider">Titik Wisata</div>
      </div>
    </div>

    <!-- Sambutan -->
    <div class="mt-24 mb-24 grid grid-cols-1 md:grid-cols-2 gap-16 items-center">
      <div class="relative">
        <svg class="absolute -top-10 -left-10 w-24 h-24 text-slate-100 -z-10" fill="currentColor" viewBox="0 0 32 32" aria-hidden="true"><path d="M9.352 4C4.456 7.456 1 13.12 1 19.36c0 5.088 3.072 8.064 6.624 8.064 3.36 0 5.856-2.688 5.856-5.856 0-3.168-2.208-5.472-5.088-5.472-.576 0-1.344.096-1.536.192.48-3.264 3.552-7.104 6.624-9.024L9.352 4zm16.512 0c-4.8 3.456-8.256 9.12-8.256 15.36 0 5.088 3.072 8.064 6.624 8.064 3.264 0 5.856-2.688 5.856-5.856 0-3.168-2.304-5.472-5.184-5.472-.576 0-1.248.096-1.44.192.48-3.264 3.456-7.104 6.528-9.024L25.864 4z"/></svg>
        <div class="w-16 h-1.5 bg-emerald-500 rounded-full mb-6"></div>
        <h2 class="text-3xl md:text-4xl font-extrabold text-slate-800 mb-6 tracking-tight">Sambutan Kepala Jorong</h2>
        <p class="text-slate-600 leading-relaxed mb-4 text-justify font-medium">"Selamat datang di portal resmi digital Jorong Lubuak Pulai. Kami berharap platform ini dapat menjadi jembatan informasi yang kuat untuk menghubungkan masyarakat, pelaku usaha UMKM, dan wisatawan dari berbagai daerah."</p>
        <p class="text-slate-600 leading-relaxed text-justify font-medium">Dengan semangat gotong royong, mari kita wujudkan Lubuak Pulai yang maju, mandiri, dan berbasis teknologi, tanpa sedikitpun meninggalkan nilai-nilai adat dan tradisi budaya leluhur kita.</p>
        <div class="mt-8 flex items-center gap-4">
          <div class="w-12 h-12 bg-emerald-100 rounded-full flex items-center justify-center text-emerald-600 font-bold">NK</div>
          <div>
            <div class="font-extrabold text-slate-800 text-lg">Bapak Nama Kades</div>
            <div class="text-emerald-600 text-sm font-bold uppercase tracking-wider">Kepala Jorong Lubuak Pulai</div>
          </div>
        </div>
      </div>
      <div class="rounded-3xl overflow-hidden shadow-2xl h-[450px] relative group">
        <div class="absolute inset-0 bg-emerald-900/10 group-hover:bg-transparent transition duration-500 z-10"></div>
        <img src="https://knurnsmwprpdfhwenbpo.supabase.co/storage/v1/object/public/Asset/dont%20even%20joke%20lad.jpg" class="w-full h-full object-cover group-hover:scale-105 transition duration-700" alt="Balai Desa" />
      </div>
    </div>
  </div>
</section>

{#if loading}
  <div class="flex flex-col items-center justify-center py-20">
    <svg class="animate-spin -ml-1 mr-3 h-10 w-10 text-emerald-600 mb-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
    <div class="text-emerald-600 font-bold animate-pulse text-xl">Menarik data dari server Nagari...</div>
  </div>
{:else}

  <!-- ================= SECTION: WISATA ================= -->
  <section id="wisata" class="py-24 bg-slate-50 border-t border-slate-200">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="text-center mb-16">
        <div class="inline-flex items-center justify-center w-16 h-16 rounded-2xl bg-emerald-100 text-emerald-600 mb-6 shadow-sm">
          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m8 3 4 8 5-5 5 15H2L8 3z"/></svg>
        </div>
        <h2 class="text-3xl md:text-4xl font-extrabold text-slate-800 tracking-tight">Peta Destinasi Wisata</h2>
        <p class="text-slate-500 mt-3 font-medium max-w-2xl mx-auto">Jelajahi surga tersembunyi Sirukam dan temukan keindahan alam langsung dari genggaman Anda.</p>
      </div>
      
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        <!-- PETA -->
        <div class="lg:col-span-2 h-[550px] bg-white rounded-3xl shadow-lg border border-slate-200 overflow-hidden relative z-0 p-2">
          <div id="map" class="w-full h-full rounded-2xl z-0"></div>
        </div>

        <!-- DAFTAR WISATA -->
        <div class="lg:col-span-1 flex flex-col gap-4 h-[550px] overflow-y-auto pr-3 custom-scrollbar">
          {#each wisataList as wisata}
            <div class="group bg-white rounded-2xl shadow-sm border border-slate-200 p-6 cursor-pointer hover:border-emerald-500 hover:shadow-lg transition-all duration-300">
              <div class="flex justify-between items-center mb-1">
                <h3 class="font-extrabold text-lg text-slate-800">{wisata.nama_tempat}</h3>
                <span class="text-emerald-500 bg-emerald-50 p-1.5 rounded-lg group-hover:rotate-180 transition-all duration-300"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="m6 9 6 6 6-6"/></svg></span>
              </div>
              <p class="text-xs font-bold text-slate-400 flex items-center gap-1 uppercase tracking-wider"><svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/></svg> {wisata.lokasi}</p>
              
              <div class="grid grid-rows-[0fr] group-hover:grid-rows-[1fr] transition-all duration-500 ease-in-out">
                <div class="overflow-hidden">
                  <div class="pt-4 mt-4 border-t border-slate-100">
                    <img src={wisata.foto_url} loading="lazy" alt={wisata.nama_tempat} class="w-full h-36 object-cover rounded-xl mb-4 shadow-sm" />
                    <p class="text-sm text-slate-600 line-clamp-3 mb-4 font-medium leading-relaxed">{wisata.deskripsi}</p>
                    <a href="https://www.google.com/maps/dir/?api=1&destination={wisata.latitude},{wisata.longitude}" target="_blank" class="inline-flex items-center gap-1 text-sm font-bold text-emerald-600 hover:text-emerald-700 hover:underline">
                      Rute Maps <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"/><path d="m12 5 7 7-7 7"/></svg>
                    </a>
                  </div>
                </div>
              </div>
            </div>
          {:else}
            <div class="bg-white p-8 rounded-2xl border border-dashed border-slate-300 text-center flex flex-col items-center justify-center h-full">
              <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" class="text-slate-300 mb-3" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
              <p class="text-slate-500 font-medium">Belum ada destinasi wisata.</p>
            </div>
          {/each}
        </div>
      </div>
    </div>
  </section>

  <!-- ================= SECTION: UMKM ================= -->
  <section id="umkm" class="py-24 bg-white border-t border-slate-200">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="text-center mb-16">
        <div class="inline-flex items-center justify-center w-16 h-16 rounded-2xl bg-amber-100 text-amber-600 mb-6 shadow-sm">
          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M2 9a3 3 0 0 1 0 6v2a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-2a3 3 0 0 1 0-6V7a2 2 0 0 0-2-2H4a2 2 0 0 0-2 2Z"/><path d="M13 18H7"/><path d="M7 14h.01"/></svg>
        </div>
        <h2 class="text-3xl md:text-4xl font-extrabold text-slate-800 tracking-tight">Lapak Warga (UMKM)</h2>
        <p class="text-slate-500 mt-3 font-medium max-w-2xl mx-auto">Dukung perekonomian lokal dengan membeli produk karya tangan warga Nagari Sirukam.</p>
      </div>

      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-8">
        {#each umkmList as umkm}
          <div class="bg-white rounded-3xl border border-slate-100 shadow-sm hover:shadow-xl transition-all duration-300 group overflow-hidden flex flex-col hover:-translate-y-1">
            
            <div class="relative h-36 bg-slate-200 overflow-hidden">
              {#if umkm.banner_url}
                <img src={umkm.banner_url} alt="Banner" class="w-full h-full object-cover group-hover:scale-110 transition duration-700" loading="lazy" />
              {:else}
                <div class="w-full h-full bg-gradient-to-br from-amber-400 to-orange-500"></div>
              {/if}
              <div class="absolute inset-0 bg-gradient-to-t from-black/50 to-transparent"></div>
              <span class="absolute top-4 right-4 px-3 py-1 bg-white/90 backdrop-blur-sm text-slate-800 text-xs font-bold uppercase tracking-wider rounded-xl shadow-sm">{umkm.kategori}</span>
            </div>

            <div class="px-6 relative flex justify-between items-end -mt-12 mb-4">
              <div class="w-24 h-24 bg-white rounded-2xl shadow-md p-1.5 border border-slate-100 flex-shrink-0 relative z-10">
                {#if umkm.foto_url}
                  <img src={umkm.foto_url} alt="Produk" class="w-full h-full object-cover rounded-xl" loading="lazy" />
                {:else}
                  <div class="w-full h-full bg-slate-50 rounded-xl flex items-center justify-center text-3xl"><svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-slate-400"><path d="M6 2 3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4Z"/><path d="M3 6h18"/><path d="M16 10a4 4 0 0 1-8 0"/></svg></div>
                {/if}
              </div>
            </div>

            <div class="px-6 pb-6 flex-1 flex flex-col">
              <h3 class="font-extrabold text-xl text-slate-800 mb-1">{umkm.nama_usaha}</h3>
              <p class="text-emerald-600 font-extrabold text-lg mb-4">{formatRupiah(umkm.harga)}</p>
              
              <p class="text-slate-500 text-sm mb-6 line-clamp-3 font-medium leading-relaxed">
                {umkm.deskripsi || 'Lapak ini belum menambahkan deskripsi produk.'}
              </p>

              <a href="https://wa.me/{umkm.nomor_wa.replace(/^0/, '62')}" target="_blank" class="mt-auto flex items-center justify-center gap-2 w-full bg-slate-900 text-white py-3.5 rounded-xl shadow-sm hover:bg-emerald-600 transition-all duration-300 font-bold hover:shadow-emerald-500/30">
                <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" fill="currentColor" viewBox="0 0 16 16"><path d="M13.601 2.326A7.85 7.85 0 0 0 7.994 0C3.627 0 .068 3.558.064 7.926c0 1.399.366 2.76 1.057 3.965L0 16l4.204-1.102a7.9 7.9 0 0 0 3.79.965h.004c4.368 0 7.926-3.558 7.93-7.93A7.9 7.9 0 0 0 13.6 2.326zM7.994 14.521a6.6 6.6 0 0 1-3.356-.92l-.24-.144-2.494.654.666-2.433-.156-.251a6.56 6.56 0 0 1-1.007-3.505c0-3.626 2.957-6.584 6.591-6.584a6.56 6.56 0 0 1 4.66 1.931 6.56 6.56 0 0 1 1.928 4.66c-.004 3.639-2.961 6.592-6.592 6.592m3.615-4.934c-.197-.099-1.17-.578-1.353-.646-.182-.065-.315-.099-.445.099-.133.197-.513.646-.627.775-.114.133-.232.148-.43.05-.197-.1-.836-.308-1.592-.985-.59-.525-.985-1.175-1.103-1.372-.114-.198-.011-.304.088-.403.087-.088.197-.232.296-.346.1-.114.133-.198.198-.33.065-.134.034-.248-.015-.347-.05-.099-.445-1.076-.612-1.47-.16-.389-.323-.335-.445-.34-.114-.007-.247-.007-.38-.007a.73.73 0 0 0-.529.247c-.182.198-.691.677-.691 1.654s.71 1.916.81 2.049c.098.133 1.394 2.132 3.383 2.992.47.205.84.326 1.129.418.475.152.904.129 1.246.08.38-.058 1.171-.48 1.338-.943.164-.464.164-.86.114-.943-.049-.084-.182-.133-.38-.232"/></svg>
                Hubungi via WhatsApp
              </a>
            </div>
          </div>
        {/each}
      </div>

      <!-- CTA DAFTAR LAPAK -->
      <div class="mt-20 bg-gradient-to-br from-emerald-50 to-teal-50 rounded-[2rem] p-10 md:p-14 border border-emerald-100 shadow-sm text-center relative overflow-hidden group">
        <div class="absolute top-0 right-0 -mr-16 -mt-16 w-64 h-64 bg-emerald-200/50 rounded-full blur-3xl group-hover:bg-emerald-300/50 transition duration-700"></div>
        <div class="absolute bottom-0 left-0 -ml-16 -mb-16 w-64 h-64 bg-teal-200/50 rounded-full blur-3xl group-hover:bg-teal-300/50 transition duration-700"></div>
        
        <div class="relative z-10 flex flex-col items-center">
          <div class="w-20 h-20 bg-white rounded-2xl shadow-sm flex items-center justify-center text-emerald-600 mb-6 border border-emerald-100">
            <svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M2 9a3 3 0 0 1 0 6v2a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-2a3 3 0 0 1 0-6V7a2 2 0 0 0-2-2H4a2 2 0 0 0-2 2Z"/><path d="M13 18H7"/><path d="M7 14h.01"/></svg>
          </div>
          <h3 class="text-3xl md:text-4xl font-extrabold text-slate-800 mb-4 tracking-tight">Punya Usaha di Nagari Sirukam?</h3>
          <p class="text-slate-600 mb-10 max-w-xl mx-auto text-base md:text-lg font-medium">
            Tingkatkan jangkauan pembeli dan majukan perekonomian lokal dengan mendaftarkan lapak Anda ke etalase digital Nagari secara <span class="font-bold text-emerald-600">Gratis!</span>
          </p>
          
          <a href="/register" class="inline-flex items-center justify-center gap-3 bg-slate-900 text-white font-bold px-10 py-4 rounded-2xl hover:bg-emerald-600 hover:-translate-y-1 hover:shadow-xl hover:shadow-emerald-500/30 transition-all duration-300">
            Daftarkan Lapak Anda <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"/><path d="m12 5 7 7-7 7"/></svg>
          </a>
        </div>
      </div>

    </div>
  </section>

  <!-- ================= SECTION: BERITA ================= -->
  <section id="berita" class="py-24 bg-slate-50 border-t border-slate-200">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="text-center mb-16">
        <div class="inline-flex items-center justify-center w-16 h-16 rounded-2xl bg-purple-100 text-purple-600 mb-6 shadow-sm">
          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 22h16a2 2 0 0 0 2-2V4a2 2 0 0 0-2-2H8a2 2 0 0 0-2 2v16a2 2 0 0 1-2 2Zm0 0a2 2 0 0 1-2-2v-9c0-1.1.9-2 2-2h2"/><path d="M18 14h-8"/><path d="M15 18h-5"/><path d="M10 6h8v4h-8V6Z"/></svg>
        </div>
        <h2 class="text-3xl md:text-4xl font-extrabold text-slate-800 tracking-tight">Berita & Informasi</h2>
        <p class="text-slate-500 mt-3 font-medium max-w-2xl mx-auto">Kabar terbaru, pengumuman, dan cerita warga langsung dari Balai Nagari Sirukam.</p>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
        {#each artikelList as artikel}
          <div class="bg-white rounded-3xl shadow-sm border border-slate-100 overflow-hidden flex flex-col hover:-translate-y-2 hover:shadow-xl transition-all duration-300 group">
            <div class="overflow-hidden">
              <img src={artikel.foto_url} loading="lazy" alt={artikel.judul} class="w-full h-52 object-cover bg-slate-100 group-hover:scale-105 transition duration-700" />
            </div>
            
            <div class="p-8 flex-1 flex flex-col">
              <h3 class="font-extrabold text-xl text-slate-800 mb-3 leading-snug group-hover:text-emerald-600 transition">{artikel.judul}</h3>
              <p class="text-xs text-slate-400 mb-5 font-bold uppercase tracking-wider flex items-center gap-2">
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M19 21v-2a4 4 0 0 0-4-4H9a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
                Oleh: {artikel.penulis}
              </p>
              
              <p class="text-sm text-slate-600 line-clamp-3 mb-8 font-medium leading-relaxed">{artikel.konten}</p>
              
              <a href="/berita/{artikel.id}" class="mt-auto inline-flex items-center gap-2 text-emerald-600 font-bold text-sm hover:text-emerald-700 transition group/btn">
                Baca Selengkapnya <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round" class="group-hover/btn:translate-x-1 transition-transform"><path d="M5 12h14"/><path d="m12 5 7 7-7 7"/></svg>
              </a>
            </div>
          </div>
        {/each}
      </div>
    </div>
  </section>

{/if}

<!-- FOOTER -->
<footer id="kontak" class="bg-slate-900 text-slate-400 py-16 border-t border-slate-800">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex flex-col md:flex-row justify-between items-center gap-8">
      <div class="text-center md:text-left">
        <div class="text-3xl font-extrabold text-white tracking-tight mb-3 flex items-center justify-center md:justify-start gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round" class="text-emerald-500"><path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/></svg>
          Sirukam<span class="text-emerald-500">Smart.</span>
        </div>
        <p class="text-sm font-medium">Pusat Informasi & Potensi Nagari Sirukam, Kabupaten Solok.</p>
      </div>
      <div class="text-sm text-center md:text-right font-medium">
        &copy; 2026 Pemerintah Nagari Sirukam.<br>Dikelola dengan ❤️ oleh Admin Nagari Sirukam.
      </div>
    </div>
  </div>
</footer>

<style>
  :global(html) {
    scroll-behavior: smooth;
  }

  .custom-scrollbar::-webkit-scrollbar {
    width: 6px;
  }
  .custom-scrollbar::-webkit-scrollbar-track {
    background: #f8fafc;
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