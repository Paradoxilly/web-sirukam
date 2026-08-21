<script>
  import { supabase } from '$lib/supabase';
  import { onMount, onDestroy } from 'svelte';

  let wisataList = $state([]);
  let umkmList = $state([]);
  let artikelList = $state([]);
  let perangkatList = $state([]); // STATE BARU BUAT PERANGKAT
  let loading = $state(true);
  
  let mapInstance; 

  let expandedWisataId = $state(null);
  let carouselIndex = $state(0);
  let sliderInterval;

  // ================= STATE FORM KONTAK BARU =================
  let isContactModalOpen = $state(false);
  let formKontak = $state({ nama: '', email: '', pesan: '' });
  let isSending = $state(false);

  function formatRupiah(harga) {
    if (!harga) return '';
    const strHarga = harga.toString().trim();

    const formatAngka = (angka) => {
      const cleanAngka = angka.trim();
      if (/^\d+$/.test(cleanAngka)) {
        return new Intl.NumberFormat('id-ID', {
          style: 'currency',
          currency: 'IDR',
          minimumFractionDigits: 0
        }).format(cleanAngka);
      }
      return cleanAngka; 
    };

    if (strHarga.includes('-')) {
      const parts = strHarga.split('-');
      return parts.map(p => formatAngka(p)).join(' - ');
    } else {
      return formatAngka(strHarga);
    }
  }

  function toggleWisata(id) {
    if (expandedWisataId === id) {
      expandedWisataId = null; 
    } else {
      expandedWisataId = id; 
    }
  }

  // Fungsi buat bikin inisial otomatis dari nama (Misal: Sullftak Dev -> SD)
  function getInitials(name) {
    if (!name) return 'NK';
    const words = name.replace(/Bapak |Ibu |Pak |Bu /gi, '').trim().split(' ');
    if (words.length === 1) return words[0].substring(0, 2).toUpperCase();
    return (words[0][0] + words[1][0]).toUpperCase();
  }

  // ================= FUNGSI KIRIM PESAN DOUBLE KILL =================
  async function kirimPesan(e) {
    e.preventDefault();
    isSending = true;

    try {
      const { error } = await supabase.from('pesan_masuk').insert([formKontak]);
      if (error) throw error;

      // GANTI EMAIL DI BAWAH INI WOK!
      await fetch('https://formsubmit.co/ajax/email_lu_disini@gmail.com', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'Accept': 'application/json' },
        body: JSON.stringify({
          Nama: formKontak.nama,
          Email_Pengirim: formKontak.email,
          Isi_Pesan: formKontak.pesan,
          _subject: "Ada Pesan Baru dari Web Nagari Sirukam!"
        })
      });

      alert('Terima kasih! Pesan Bapak/Ibu sudah terkirim ke Admin Nagari.');
      isContactModalOpen = false;
      formKontak = { nama: '', email: '', pesan: '' }; 
    } catch (err) {
      alert('Maaf, pesan gagal terkirim: ' + err.message);
    }
    isSending = false;
  }

  onMount(async () => {
    sliderInterval = setInterval(() => {
      carouselIndex++;
    }, 4000); // Muter tiap 4 detik

    const { data: dataWisata } = await supabase.from('wisata').select('*').order('created_at', { ascending: false });
    if (dataWisata) wisataList = dataWisata;

    const { data: dataUmkm } = await supabase.from('umkm').select('*').eq('status', 'approved').order('created_at', { ascending: false }).limit(4);
    if (dataUmkm) umkmList = dataUmkm;

    const { data: dataArtikel } = await supabase.from('artikel').select('*').order('created_at', { ascending: false }).limit(3);
    if (dataArtikel) artikelList = dataArtikel;

    // Tarik data perangkat nagari
    const { data: dataPerangkat } = await supabase.from('perangkat_nagari').select('*').order('created_at', { ascending: true });
    if (dataPerangkat) perangkatList = dataPerangkat;

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

  onDestroy(() => {
    if (sliderInterval) clearInterval(sliderInterval);
  });
</script>

<svelte:head>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
</svelte:head>

<nav class="fixed top-0 left-0 w-full bg-white/90 backdrop-blur-md border-b border-gray-100 z-50 transition-all duration-300">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex justify-between items-center h-20">
      <div class="flex-shrink-0">
        <a href="#profil" class="text-2xl font-bold text-slate-800 tracking-tight flex items-center gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 120" class="h-14 w-auto text-emerald-600" fill="currentColor">
            
            <!-- Atap Utama (4 Gonjong Belakang) -->
            <path d="M 10,5 Q 30,70 60,25 Q 100,75 140,25 Q 170,70 190,5 L 175,65 L 25,65 Z" />
            
            <!-- Badan Rumah Utama -->
            <polygon points="30,67 170,67 155,95 45,95" />
            
            <!-- Anjungan (Bagian Tengah yang Menonjol) -->
            <rect x="85" y="67" width="30" height="33" stroke="white" stroke-width="2" />
            <!-- Atap Anjungan (Gonjong Tengah) -->
            <path d="M 73,67 Q 90,60 100,25 Q 110,60 127,67 Z" stroke="white" stroke-width="2" stroke-linejoin="round" />
            
            <!-- Jendela & Pintu (Warna Putih) -->
            <path d="M 50,75 h 8 v 10 h -8 z 
                    M 68,75 h 8 v 10 h -8 z 
                    M 124,75 h 8 v 10 h -8 z 
                    M 142,75 h 8 v 10 h -8 z 
                    M 93,78 h 14 v 22 h -14 z" fill="white" />
            
            <!-- Tiang-Tiang Pondasi -->
            <path d="M 48,95 v 15 h 3 v -15 
                    M 63,95 v 15 h 3 v -15 
                    M 78,95 v 15 h 3 v -15 
                    M 119,95 v 15 h 3 v -15 
                    M 134,95 v 15 h 3 v -15 
                    M 149,95 v 15 h 3 v -15" />
            
            <!-- Tangga Masuk -->
            <path d="M 92,100 h 16 v 3 h -16 z 
                    M 88,104 h 24 v 3 h -24 z 
                    M 84,108 h 32 v 3 h -32 z" />
          </svg>          Sirukam<span class="text-emerald-600">Smart.</span>
        </a>
      </div>
      
      <div class="hidden md:flex space-x-8">
        <a href="#profil" class="text-sm font-bold text-slate-500 hover:text-emerald-600 transition">Profil</a>
        <a href="#wisata" class="text-sm font-bold text-slate-500 hover:text-emerald-600 transition">Wisata</a>
        <a href="#umkm" class="text-sm font-bold text-slate-500 hover:text-emerald-600 transition">UMKM</a>
        <a href="#berita" class="text-sm font-bold text-slate-500 hover:text-emerald-600 transition">Berita</a>
      </div>

      <div class="hidden md:block">
        <button onclick={() => isContactModalOpen = true} class="flex items-center gap-2 bg-slate-900 hover:bg-emerald-600 text-white px-6 py-2.5 rounded-xl font-bold transition-all duration-300 shadow-sm hover:shadow-md hover:-translate-y-0.5">
          Hubungi Kami
        </button>
      </div>
    </div>
  </div>
</nav>

<section id="profil" class="pt-20">
  <div class="relative min-h-[85vh] flex items-center justify-center bg-slate-900">
    <div class="absolute inset-0">
      <img src="https://knurnsmwprpdfhwenbpo.supabase.co/storage/v1/object/public/Asset/banner.jpg" class="w-full h-full object-cover opacity-50" alt="Jorong Lubuak Pulai" />
      <div class="absolute inset-0 bg-gradient-to-t from-slate-900 via-slate-900/40 to-slate-900/20"></div>
    </div>
    
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
        <div class="mx-auto w-12 h-12 bg-amber-50 text-amber-600 rounded-2xl flex items-center justify-center mb-3"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 9h18v10a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V9Z"/><path d="m3 9 2.45-4.9A2 2 0 0 1 7.24 3h9.52a2 2 0 0 1 1.8 1.1L21 9"/><path d="M12 3v6"/></svg></div>
        <div class="text-3xl md:text-4xl font-extrabold text-slate-800">30+</div><div class="text-xs md:text-sm font-bold text-slate-400 mt-1 uppercase tracking-wider">UMKM Aktif</div>
      </div>
      <div class="text-center p-4 border-t md:border-t-0 md:border-l border-slate-100">
        <div class="mx-auto w-12 h-12 bg-purple-50 text-purple-600 rounded-2xl flex items-center justify-center mb-3"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/></svg></div>
        <div class="text-3xl md:text-4xl font-extrabold text-slate-800">5</div><div class="text-xs md:text-sm font-bold text-slate-400 mt-1 uppercase tracking-wider">Titik Wisata</div>
      </div>
    </div>

    <!-- ================= SECTION SAMBUTAN & CAROUSEL ================= -->
    <div class="mt-24 mb-24 grid grid-cols-1 md:grid-cols-2 gap-16 items-center">
      <div class="relative">
        <svg class="absolute -top-10 -left-10 w-24 h-24 text-slate-100 -z-10" fill="currentColor" viewBox="0 0 32 32" aria-hidden="true"><path d="M9.352 4C4.456 7.456 1 13.12 1 19.36c0 5.088 3.072 8.064 6.624 8.064 3.36 0 5.856-2.688 5.856-5.856 0-3.168-2.208-5.472-5.088-5.472-.576 0-1.344.096-1.536.192.48-3.264 3.552-7.104 6.624-9.024L9.352 4zm16.512 0c-4.8 3.456-8.256 9.12-8.256 15.36 0 5.088 3.072 8.064 6.624 8.064 3.264 0 5.856-2.688 5.856-5.856 0-3.168-2.304-5.472-5.184-5.472-.576 0-1.248.096-1.44.192.48-3.264 3.456-7.104 6.528-9.024L25.864 4z"/></svg>
        <div class="w-16 h-1.5 bg-emerald-500 rounded-full mb-6"></div>
        <h2 class="text-3xl md:text-4xl font-extrabold text-slate-800 mb-6 tracking-tight">Sambutan Perangkat Nagari</h2>
        <p class="text-slate-600 leading-relaxed mb-4 text-justify font-medium">"Selamat datang di portal resmi digital Jorong Lubuak Pulai. Kami berharap platform ini dapat menjadi jembatan informasi yang kuat untuk menghubungkan masyarakat, pelaku usaha UMKM, dan wisatawan dari berbagai daerah."</p>
        <p class="text-slate-600 leading-relaxed text-justify font-medium">Dengan semangat gotong royong, mari kita wujudkan Lubuak Pulai yang maju, mandiri, dan berbasis teknologi, tanpa sedikitpun meninggalkan nilai-nilai adat dan tradisi budaya leluhur kita.</p>
        
        <div class="mt-8 relative h-16">
          {#if perangkatList.length > 0}
            {#each perangkatList as p, i}
              <div class="absolute inset-0 flex items-center gap-4 transition-all duration-700 ease-in-out {i === (carouselIndex % perangkatList.length) ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-4 pointer-events-none'}">
                <div class="w-12 h-12 bg-emerald-100 rounded-full flex items-center justify-center text-emerald-600 font-bold">{getInitials(p.nama)}</div>
                <div>
                  <div class="font-extrabold text-slate-800 text-lg">{p.nama}</div>
                  <div class="text-emerald-600 text-sm font-bold uppercase tracking-wider">{p.jabatan}</div>
                </div>
              </div>
            {/each}
          {:else}
            <!-- Fallback kalau admin belum ngisi -->
            <div class="flex items-center gap-4">
              <div class="w-12 h-12 bg-emerald-100 rounded-full flex items-center justify-center text-emerald-600 font-bold">NK</div>
              <div>
                <div class="font-extrabold text-slate-800 text-lg">Bapak Nama Kades</div>
                <div class="text-emerald-600 text-sm font-bold uppercase tracking-wider">Kepala Jorong Lubuak Pulai</div>
              </div>
            </div>
          {/if}
        </div>
      </div>

      <div class="rounded-3xl overflow-hidden shadow-2xl h-[450px] relative bg-slate-100">
        {#if perangkatList.length > 0}
          {#each perangkatList as p, i}
            <!-- RAHASIA ANTI GEPENG: absolute inset-0 w-full h-full object-cover -->
            <img src={p.foto_url} class="absolute inset-0 w-full h-full object-cover transition-opacity duration-1000 {i === (carouselIndex % perangkatList.length) ? 'opacity-100' : 'opacity-0'}" alt={p.nama} />
          {/each}
          
          <!-- Indikator Bulat-Bulat Carousel -->
          {#if perangkatList.length > 1}
            <div class="absolute bottom-4 left-0 right-0 flex justify-center gap-2 z-20">
              {#each perangkatList as _, i}
                <div class="w-2 h-2 rounded-full transition-colors duration-500 {i === (carouselIndex % perangkatList.length) ? 'bg-white scale-125' : 'bg-white/50'}"></div>
              {/each}
            </div>
          {/if}
        {:else}
          <!-- Fallback foto default -->
          <img src="https://knurnsmwprpdfhwenbpo.supabase.co/storage/v1/object/public/Asset/dont%20even%20joke%20lad.jpg" class="w-full h-full object-cover" alt="Balai Desa" />
        {/if}
        
        <!-- Gradient Biar Elegan -->
        <div class="absolute inset-0 bg-gradient-to-t from-slate-900/40 to-transparent pointer-events-none z-10"></div>
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

  <section id="wisata" class="py-24 bg-slate-50 border-t border-slate-200">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="text-center mb-16">
        <div class="inline-flex items-center justify-center w-16 h-16 rounded-2xl bg-emerald-100 text-emerald-600 mb-6 shadow-sm">
          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="3 6 9 3 15 6 21 3 21 18 15 21 9 18 3 21"/><line x1="9" y1="3" x2="9" y2="18"/><line x1="15" y1="6" x2="15" y2="21"/></svg>
        </div>
        <h2 class="text-3xl md:text-4xl font-extrabold text-slate-800 tracking-tight">Peta Destinasi Wisata</h2>
        <p class="text-slate-500 mt-3 font-medium max-w-2xl mx-auto">Jelajahi surga tersembunyi Sirukam dan temukan keindahan alam langsung dari genggaman Anda.</p>
      </div>
      
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        <div class="lg:col-span-2 h-[550px] bg-white rounded-3xl shadow-lg border border-slate-200 overflow-hidden relative z-0 p-2">
          <div id="map" class="w-full h-full rounded-2xl z-0"></div>
        </div>

        <div class="lg:col-span-1 flex flex-col gap-4 h-[550px] overflow-y-auto pr-3 custom-scrollbar">
          {#each wisataList as wisata}
            {@const fotoArray = wisata.foto_url.split(',')}

            <div onclick={() => toggleWisata(wisata.id)} class="bg-white rounded-2xl shadow-sm border p-6 cursor-pointer transition-all duration-300 {expandedWisataId === wisata.id ? 'border-emerald-500 shadow-lg' : 'border-slate-200 hover:border-emerald-500'}">
              <div class="flex justify-between items-center mb-1">
                <h3 class="font-extrabold text-lg text-slate-800">{wisata.nama_tempat}</h3>
                <span class="text-emerald-500 bg-emerald-50 p-1.5 rounded-lg transition-transform duration-300 {expandedWisataId === wisata.id ? 'rotate-180' : ''}">
                  <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="m6 9 6 6 6-6"/></svg>
                </span>
              </div>
              <p class="text-xs font-bold text-slate-400 flex items-center gap-1 uppercase tracking-wider"><svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/></svg> {wisata.lokasi}</p>
              
              <div class="grid transition-all duration-500 ease-in-out {expandedWisataId === wisata.id ? 'grid-rows-[1fr]' : 'grid-rows-[0fr]'}">
                <div class="overflow-hidden">
                  <div class="pt-4 mt-4 border-t border-slate-100 cursor-default" onclick={(e) => e.stopPropagation()}>
                    
                    <div class="relative w-full h-40 mb-4 rounded-xl overflow-hidden shadow-sm bg-slate-900">
                      {#each fotoArray as foto, index}
                        <img src={foto} loading="lazy" alt={wisata.nama_tempat} class="absolute inset-0 w-full h-full object-cover transition-opacity duration-1000 ease-in-out {index === (carouselIndex % fotoArray.length) ? 'opacity-100' : 'opacity-0'}" />
                      {/each}

                      {#if fotoArray.length > 1}
                        <div class="absolute bottom-2 left-0 right-0 flex justify-center gap-1.5 z-10">
                          {#each fotoArray as _, index}
                            <div class="w-1.5 h-1.5 rounded-full transition-colors duration-500 {index === (carouselIndex % fotoArray.length) ? 'bg-white scale-125' : 'bg-white/40'}"></div>
                          {/each}
                        </div>
                      {/if}
                    </div>
                    
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
              <p class="text-slate-500 font-medium">Belum ada destinasi wisata.</p>
            </div>
          {/each}
        </div>
      </div>
    </div>
  </section>

  <section id="umkm" class="py-24 bg-white border-t border-slate-200">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="text-center mb-16">
        <div class="inline-flex items-center justify-center w-16 h-16 rounded-2xl bg-amber-100 text-amber-600 mb-6 shadow-sm">
          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 9h18v10a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V9Z"/><path d="m3 9 2.45-4.9A2 2 0 0 1 7.24 3h9.52a2 2 0 0 1 1.8 1.1L21 9"/><path d="M12 3v6"/></svg>
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
              <p class="text-slate-500 text-sm mb-6 line-clamp-3 font-medium leading-relaxed">{umkm.deskripsi || 'Lapak ini belum menambahkan deskripsi produk.'}</p>
              <a href="https://wa.me/{umkm.nomor_wa.replace(/^0/, '62')}" target="_blank" class="mt-auto flex items-center justify-center gap-2 w-full bg-slate-900 text-white py-3.5 rounded-xl shadow-sm hover:bg-emerald-600 transition-all duration-300 font-bold hover:shadow-emerald-500/30">
                Pesan via WhatsApp
              </a>
            </div>
          </div>
        {/each}
      </div>

      <div class="mt-20 bg-gradient-to-br from-emerald-50 to-teal-50 rounded-[2rem] p-10 md:p-14 border border-emerald-100 shadow-sm text-center relative overflow-hidden group">
        <div class="absolute top-0 right-0 -mr-16 -mt-16 w-64 h-64 bg-emerald-200/50 rounded-full blur-3xl group-hover:bg-emerald-300/50 transition duration-700"></div>
        <div class="absolute bottom-0 left-0 -ml-16 -mb-16 w-64 h-64 bg-teal-200/50 rounded-full blur-3xl group-hover:bg-teal-300/50 transition duration-700"></div>
        <div class="relative z-10 flex flex-col items-center">
          <div class="w-20 h-20 bg-white rounded-2xl shadow-sm flex items-center justify-center text-emerald-600 mb-6 border border-emerald-100">
            <svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M2 9a3 3 0 0 1 0 6v2a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-2a3 3 0 0 1 0-6V7a2 2 0 0 0-2-2H4a2 2 0 0 0-2 2Z"/><path d="M13 18H7"/><path d="M7 14h.01"/></svg>
          </div>
          <h3 class="text-3xl md:text-4xl font-extrabold text-slate-800 mb-4 tracking-tight">Punya Usaha di Nagari Sirukam?</h3>
          <p class="text-slate-600 mb-10 max-w-xl mx-auto text-base md:text-lg font-medium">Tingkatkan jangkauan pembeli dan majukan perekonomian lokal dengan mendaftarkan lapak Anda ke etalase digital Nagari secara <span class="font-bold text-emerald-600">Gratis!</span></p>
          <a href="/register" class="inline-flex items-center justify-center gap-3 bg-slate-900 text-white font-bold px-10 py-4 rounded-2xl hover:bg-emerald-600 hover:-translate-y-1 hover:shadow-xl hover:shadow-emerald-500/30 transition-all duration-300">
            Daftarkan Lapak Anda <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"/><path d="m12 5 7 7-7 7"/></svg>
          </a>
        </div>
      </div>

    </div>
  </section>

  <section id="berita" class="py-24 bg-slate-50 border-t border-slate-200">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="text-center mb-16">
        <div class="inline-flex items-center justify-center w-16 h-16 rounded-2xl bg-purple-100 text-purple-600 mb-6 shadow-sm">
          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m3 11 18-5v12L3 14v-3z"/><path d="M11.5 13.5c0 2.2-2 4.5-4.5 4.5H5c-1.1 0-2-.9-2-2v-4"/></svg>
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

<footer id="kontak" class="bg-slate-900 text-slate-400 py-16 border-t border-slate-800">
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex flex-col md:flex-row justify-between items-center gap-8">
      <div class="text-center md:text-left">
        <div class="text-3xl font-extrabold text-white tracking-tight mb-3 flex items-center justify-center md:justify-start gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 120" class="h-14 w-auto text-emerald-600" fill="currentColor">
            
            <!-- Atap Utama (4 Gonjong Belakang) -->
            <path d="M 10,5 Q 30,70 60,25 Q 100,75 140,25 Q 170,70 190,5 L 175,65 L 25,65 Z" />
            
            <!-- Badan Rumah Utama -->
            <polygon points="30,67 170,67 155,95 45,95" />
            
            <!-- Anjungan (Bagian Tengah yang Menonjol) -->
            <rect x="85" y="67" width="30" height="33" stroke="white" stroke-width="2" />
            <!-- Atap Anjungan (Gonjong Tengah) -->
            <path d="M 73,67 Q 90,60 100,25 Q 110,60 127,67 Z" stroke="white" stroke-width="2" stroke-linejoin="round" />
            
            <!-- Jendela & Pintu (Warna Putih) -->
            <path d="M 50,75 h 8 v 10 h -8 z 
                    M 68,75 h 8 v 10 h -8 z 
                    M 124,75 h 8 v 10 h -8 z 
                    M 142,75 h 8 v 10 h -8 z 
                    M 93,78 h 14 v 22 h -14 z" fill="white" />
            
            <!-- Tiang-Tiang Pondasi -->
            <path d="M 48,95 v 15 h 3 v -15 
                    M 63,95 v 15 h 3 v -15 
                    M 78,95 v 15 h 3 v -15 
                    M 119,95 v 15 h 3 v -15 
                    M 134,95 v 15 h 3 v -15 
                    M 149,95 v 15 h 3 v -15" />
            
            <!-- Tangga Masuk -->
            <path d="M 92,100 h 16 v 3 h -16 z 
                    M 88,104 h 24 v 3 h -24 z 
                    M 84,108 h 32 v 3 h -32 z" />
          </svg>
            Sirukam<span class="text-emerald-500">Smart.</span>
        </div>
        <p class="text-sm font-medium">Pusat Informasi & Potensi Nagari Sirukam, Kabupaten Solok.</p>
      </div>
      <div class="text-sm text-center md:text-right font-medium">
        &copy; 2026 Pemerintah Nagari Sirukam.<br>Developed By : KKN LP3I Sirukam 2026
      </div>
    </div>
  </div>
</footer>

<!-- ================= POP-UP MODAL KONTAK ================= -->
{#if isContactModalOpen}
  <div class="fixed inset-0 z-[100] flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm">
    <div class="bg-white w-full max-w-md rounded-3xl shadow-2xl overflow-hidden relative">
      
      <div class="bg-slate-50 px-6 py-4 border-b border-slate-100 flex justify-between items-center">
        <h3 class="font-extrabold text-slate-800 text-lg flex items-center gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-emerald-600"><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"/></svg>
          Hubungi Admin
        </h3>
        <button onclick={() => isContactModalOpen = false} class="text-slate-400 hover:text-red-500 hover:bg-red-50 p-2 rounded-xl transition">
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
        </button>
      </div>

      <form onsubmit={kirimPesan} class="p-6 space-y-4">
        <div>
          <label class="block text-sm font-bold text-slate-700 mb-1">Nama Lengkap</label>
          <input type="text" bind:value={formKontak.nama} required placeholder="Contoh: Gastro Adria" class="w-full border border-slate-200 p-3 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition bg-slate-50 focus:bg-white" />
        </div>
        <div>
          <label class="block text-sm font-bold text-slate-700 mb-1">Alamat Email</label>
          <input type="email" bind:value={formKontak.email} required placeholder="email_anda@gmail.com" class="w-full border border-slate-200 p-3 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition bg-slate-50 focus:bg-white" />
        </div>
        <div>
          <label class="block text-sm font-bold text-slate-700 mb-1">Pesan Anda</label>
          <textarea bind:value={formKontak.pesan} required rows="4" placeholder="Tulis pesan, saran, atau pertanyaan di sini..." class="w-full border border-slate-200 p-3 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition bg-slate-50 focus:bg-white"></textarea>
        </div>
        
        <button type="submit" disabled={isSending} class="w-full bg-emerald-600 text-white font-bold py-3.5 rounded-xl hover:bg-emerald-700 transition shadow-sm hover:shadow-emerald-500/30 flex justify-center items-center gap-2 mt-2 disabled:opacity-50">
          {isSending ? 'Mengirim...' : 'Kirim Pesan Sekarang'}
        </button>
      </form>
    </div>
  </div>
{/if}

<style>
  :global(html) {
    scroll-behavior: smooth;
  }
  .custom-scrollbar::-webkit-scrollbar { width: 6px; }
  .custom-scrollbar::-webkit-scrollbar-track { background: #f8fafc; border-radius: 10px; }
  .custom-scrollbar::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
  .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
</style>