<script>
  import { supabase } from '$lib/supabase';
  import { page } from '$app/stores';
  import { onMount } from 'svelte';

  let artikel = $state(null);
  let loading = $state(true);

  // Fungsi buat ngerubah tanggal kaku dari database jadi estetik (Contoh: 5 Agustus 2026)
  function formatTanggal(isoString) {
    if (!isoString) return '';
    const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
    return new Date(isoString).toLocaleDateString('id-ID', options);
  }

  onMount(async () => {
    // Ngambil ID dari URL (/berita/ini-id-nya)
    const id = $page.params.id;
    
    // Tarik data spesifik artikel berdasarkan ID dari brankas Supabase
    const { data, error } = await supabase
      .from('artikel')
      .select('*')
      .eq('id', id)
      .single();
      
    if (data) {
      artikel = data;
    } else {
      console.error("Not Found:", error);
    }
    
    loading = false;
  });
</script>

<!-- NAVBAR SIMPLE KHUSUS HALAMAN BERITA -->
<nav class="fixed top-0 left-0 w-full bg-white/90 backdrop-blur-md border-b border-gray-100 z-50">
  <div class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex justify-between items-center h-20">
      <div class="flex-shrink-0">
        <a href="/" class="text-2xl font-bold text-slate-800 tracking-tight flex items-center gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-emerald-600"><path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0Z"/><circle cx="12" cy="10" r="3"/></svg>
          Sirukam<span class="text-emerald-600">Smart.</span>
        </a>
      </div>
      <div>
        <a href="/#berita" class="flex items-center gap-2 text-sm font-bold text-slate-500 hover:text-emerald-600 transition">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="m15 18-6-6 6-6"/></svg>
          Kembali ke Beranda
        </a>
      </div>
    </div>
  </div>
</nav>

<div class="min-h-screen bg-slate-50 pt-28 pb-20">
  {#if loading}
    <div class="flex flex-col items-center justify-center py-32">
      <svg class="animate-spin -ml-1 mr-3 h-10 w-10 text-emerald-600 mb-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
      <div class="text-emerald-600 font-bold animate-pulse text-xl">Mencari arsip berita...</div>
    </div>
  {:else if !artikel}
    <!-- Tampilan kalau berita nggak ketemu / ID nya ngasal -->
    <div class="max-w-4xl mx-auto px-4 text-center py-32">
      <h1 class="text-3xl font-bold text-slate-800 mb-4">Berita Tidak Ditemukan</h1>
      <p class="text-slate-500 mb-8">Waduh, sepertinya berita ini udah dihapus atau linknya salah wok.</p>
      <a href="/#berita" class="bg-emerald-600 text-white font-bold px-6 py-3 rounded-xl hover:bg-emerald-700 transition shadow-sm">Kembali ke Beranda</a>
    </div>
  {:else}
    <!-- KONTEN UTAMA ARTIKEL -->
    <main class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
      
      <!-- Breadcrumb (Jejak Navigasi) -->
      <div class="mb-8 flex items-center gap-2 text-sm font-medium text-slate-500">
        <a href="/" class="hover:text-emerald-600 transition">Beranda</a>
        <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m9 18 6-6-6-6"/></svg>
        <a href="/#berita" class="hover:text-emerald-600 transition">Berita</a>
        <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m9 18 6-6-6-6"/></svg>
        <span class="text-slate-800 line-clamp-1">{artikel.judul}</span>
      </div>

      <article class="bg-white rounded-[2rem] shadow-sm border border-slate-100 overflow-hidden">
        
        <!-- Header/Thumbnail Berita Raksasa -->
        <div class="w-full h-[50vh] md:h-[60vh] relative bg-slate-200">
          <img src={artikel.foto_url} alt={artikel.judul} class="w-full h-full object-cover" />
          
          <!-- Efek gelap (gradient) di bawah gambar biar tulisan putihnya kebaca -->
          <div class="absolute inset-0 bg-gradient-to-t from-slate-900/90 via-slate-900/40 to-transparent"></div>
          
          <div class="absolute bottom-0 left-0 w-full p-8 md:p-12">
            <span class="inline-block bg-purple-600 text-white text-xs font-bold uppercase tracking-wider px-3 py-1.5 rounded-lg mb-4 shadow-sm">Kabar Nagari</span>
            <h1 class="text-3xl md:text-5xl font-extrabold text-white leading-tight mb-6 drop-shadow-md">{artikel.judul}</h1>
            
            <div class="flex flex-wrap items-center gap-6 text-slate-200 text-sm font-medium">
              <div class="flex items-center gap-2">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M19 21v-2a4 4 0 0 0-4-4H9a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
                <span class="font-bold text-white">{artikel.penulis}</span>
              </div>
              <div class="flex items-center gap-2">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><rect width="18" height="18" x="3" y="4" rx="2" ry="2"/><line x1="16" x2="16" y1="2" y2="6"/><line x1="8" x2="8" y1="2" y2="6"/><line x1="3" x2="21" y1="10" y2="10"/></svg>
                {formatTanggal(artikel.created_at)}
              </div>
            </div>
          </div>
        </div>

        <!-- Isi Teks (Prose) -->
        <div class="p-8 md:p-12">
          <!-- Textarea nyimpen 'Enter' sebagai \n, jadi kita split biar paragrafnya rapi -->
          <div class="text-lg text-slate-700 leading-relaxed space-y-6">
            {#each artikel.konten.split('\n') as paragraf}
              {#if paragraf.trim() !== ''}
                <p>{paragraf}</p>
              {/if}
            {/each}
          </div>
        </div>
        
        <!-- ================= TEMPAT GALERI DOKUMENTASI ================= -->
        <!-- Nanti kalau lu udah siap bikin tabel "galeri_berita", tinggal di-looping di sini wok -->
        <div class="px-8 md:px-12 pb-12">
          <div class="border-t border-slate-100 pt-10 mt-4">
            <h3 class="text-xl font-extrabold text-slate-800 mb-6 flex items-center gap-2">
              <svg xmlns="http://www.w3.org/2000/svg" width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-emerald-500"><rect width="18" height="18" x="3" y="3" rx="2" ry="2"/><circle cx="9" cy="9" r="2"/><path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/></svg>
              Galeri Dokumentasi
            </h3>
            
            <div class="bg-slate-50 border-2 border-dashed border-slate-200 rounded-3xl p-10 text-center flex flex-col items-center">
              <svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" class="text-slate-300 mb-4"><path d="M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3z"/><circle cx="12" cy="13" r="3"/></svg>
              <p class="text-slate-500 font-bold mb-1">Ruang Dokumentasi</p>
            </div>
          </div>
        </div>

      </article>
    </main>
  {/if}
</div>