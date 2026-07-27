<script>
  // Logika javascript nanti di sini
</script>

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
      
      <!-- Menu Navigasi (Beranda Dihapus) -->
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

<!-- SECTION: PROFIL (Hero, Statistik & Sambutan Digabung) -->
<section id="profil" class="pt-20">
  
  <!-- 1. Hero Banner -->
  <div class="relative h-[80vh] flex items-center justify-center bg-slate-900">
    <div class="absolute inset-0">
      <!-- Ganti src nanti dengan foto drone Jorong Lubuak Pulai -->
      <img 
        src="https://images.unsplash.com/photo-1596404987070-d8a4f9104fa2?q=80&w=1920&auto=format&fit=crop" 
        class="w-full h-full object-cover opacity-40" 
        alt="Jorong Lubuak Pulai" 
      />
    </div>

    <div class="relative z-10 text-center px-4 max-w-4xl mx-auto">
      <p class="text-sm md:text-base text-emerald-400 font-semibold mb-4 tracking-widest uppercase">
        Satu Klik untuk Mengenal Jorong Lubuak Pulai
      </p>
      
      <h1 class="text-4xl md:text-5xl lg:text-6xl font-bold text-white mb-6 leading-tight drop-shadow-lg">
        Website Resmi <br />
        <span class="text-emerald-400">Jorong Lubuak Pulai</span>
      </h1>
      
      <p class="text-lg md:text-xl text-gray-300 mb-10 font-light drop-shadow-md">
        "Mengenal Potensi, Budaya, dan Kehidupan Masyarakat Nagari Sirukam secara Digital."
      </p>
      
      <div class="flex flex-col sm:flex-row justify-center gap-4">
        <a href="#wisata" class="bg-emerald-600 hover:bg-emerald-700 text-white font-medium py-3 px-8 rounded-md transition duration-300 shadow-md">
          Jelajahi Wisata
        </a>
        <a href="#umkm" class="bg-white/10 hover:bg-white/20 backdrop-blur-sm border border-white/30 text-white font-medium py-3 px-8 rounded-md transition duration-300">
          Lihat UMKM
        </a>
      </div>
    </div>
  </div>

  <!-- 2. Konten Profil (Nyambung langsung di bawah Hero) -->
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 -mt-12 relative z-20">
    
    <!-- Card Statistik (Desain Numpuk di atas batas foto) -->
    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 md:gap-6 bg-white rounded-xl shadow-xl p-6 md:p-8 border border-gray-100">
      <div class="text-center">
        <div class="text-3xl md:text-4xl font-bold text-slate-800">1.200+</div>
        <div class="text-sm text-slate-500 mt-1">Penduduk</div>
      </div>
      <div class="text-center border-l border-gray-100">
        <div class="text-3xl md:text-4xl font-bold text-slate-800">450</div>
        <div class="text-sm text-slate-500 mt-1">Kepala Keluarga</div>
      </div>
      <div class="text-center border-l border-gray-100">
        <div class="text-3xl md:text-4xl font-bold text-slate-800">30+</div>
        <div class="text-sm text-slate-500 mt-1">UMKM Aktif</div>
      </div>
      <div class="text-center border-l border-gray-100">
        <div class="text-3xl md:text-4xl font-bold text-slate-800">5</div>
        <div class="text-sm text-slate-500 mt-1">Destinasi Wisata</div>
      </div>
    </div>

    <!-- Sambutan Kades -->
    <div class="mt-20 mb-20 grid grid-cols-1 md:grid-cols-2 gap-12 items-center">
      <div>
        <div class="w-16 h-1 bg-emerald-500 mb-6"></div>
        <h2 class="text-3xl font-bold text-slate-800 mb-6">Sambutan Kepala Jorong</h2>
        <p class="text-slate-600 leading-relaxed mb-4 text-justify">
          "Selamat datang di portal resmi digital Jorong Lubuak Pulai. Kami berharap platform ini dapat menjadi jembatan informasi yang kuat untuk menghubungkan masyarakat, pelaku usaha UMKM, dan wisatawan dari berbagai daerah."
        </p>
        <p class="text-slate-600 leading-relaxed text-justify">
          Dengan semangat gotong royong, mari kita wujudkan Lubuak Pulai yang maju, mandiri, dan berbasis teknologi, tanpa sedikitpun meninggalkan nilai-nilai adat dan tradisi budaya leluhur kita.
        </p>
        <div class="mt-8">
          <div class="font-bold text-slate-800 text-lg">Bapak Nama Kades</div>
          <div class="text-emerald-600 text-sm font-medium">Kepala Jorong Lubuak Pulai</div>
        </div>
      </div>
      <div class="rounded-xl overflow-hidden shadow-lg h-96 bg-gray-200">
        <!-- Ganti dengan foto Kepala Jorong -->
        <img src="https://images.unsplash.com/photo-1577563908411-5077b6dc7624?q=80&w=800&auto=format&fit=crop" class="w-full h-full object-cover" alt="Balai Desa / Kepala Jorong" />
      </div>
    </div>

  </div>
</section>

<!-- SPASING SEMENTARA BUKTIIN SCROLL -->
<section id="wisata" class="h-screen bg-slate-50 flex items-center justify-center border-t border-gray-200">
  <div class="text-center">
    <h2 class="text-4xl font-bold text-slate-400">Bagian Wisata (Next)</h2>
  </div>
</section>
<section id="umkm" class="h-screen bg-white flex items-center justify-center border-t border-gray-200">
  <h2 class="text-4xl font-bold text-slate-400">Bagian UMKM (Next)</h2>
</section>