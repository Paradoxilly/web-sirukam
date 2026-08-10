<script>
  import { supabase } from '$lib/supabase';
  import { page } from '$app/stores';
  import { onMount } from 'svelte';

  let lapak = $state(null);
  let loading = $state(true);
  let isSubmitting = $state(false);

  // State khusus buat nampung file dan nampilin preview sblm di-upload
  let fileBanner = $state(null);
  let fileFoto = $state(null);
  let previewBanner = $state(null);
  let previewFoto = $state(null);

  onMount(async () => {
    const id = $page.params.id;
    const { data, error } = await supabase.from('umkm').select('*').eq('id', id).single();
    
    if (data) {
      lapak = data;
      // Set preview awal pakai gambar dari database (kalau ada)
      previewBanner = data.banner_url;
      previewFoto = data.foto_url;
    } else {
      console.error(error);
    }
    loading = false;
  });

  // Fungsi buat nangkep file dan langsung nampilin di layar (Preview)
  function handleBannerChange(e) {
    const file = e.target.files[0];
    if (file) {
      fileBanner = file;
      previewBanner = URL.createObjectURL(file);
    }
  }

  function handleFotoChange(e) {
    const file = e.target.files[0];
    if (file) {
      fileFoto = file;
      previewFoto = URL.createObjectURL(file);
    }
  }

  // Fungsi Upload ke Supabase Storage
  async function uploadGambar(file) {
    const fileExt = file.name.split('.').pop();
    const fileName = `umkm_${Date.now()}_${Math.random().toString(36).substring(2)}.${fileExt}`;
    const { error } = await supabase.storage.from('foto_web').upload(fileName, file);
    if (error) throw error;
    
    const { data } = supabase.storage.from('foto_web').getPublicUrl(fileName);
    return data.publicUrl;
  }

  // Fungsi Simpan Semua Perubahan
  async function simpanLapak(e) {
    e.preventDefault();
    isSubmitting = true;
    try {
      let finalBannerUrl = lapak.banner_url;
      let finalFotoUrl = lapak.foto_url;

      // Upload banner baru kalau ada yang diganti
      if (fileBanner) finalBannerUrl = await uploadGambar(fileBanner);
      // Upload foto utama baru kalau ada yang diganti
      if (fileFoto) finalFotoUrl = await uploadGambar(fileFoto);

      const { error } = await supabase.from('umkm').update({
        nama_usaha: lapak.nama_usaha,
        kategori: lapak.kategori,
        nomor_wa: lapak.nomor_wa,
        harga: lapak.harga,
        deskripsi: lapak.deskripsi,
        banner_url: finalBannerUrl,
        foto_url: finalFotoUrl
      }).eq('id', lapak.id);

      if (error) throw error;
      alert('Mantap! Etalase lapak berhasil diperbarui. 🎉');
    } catch (err) {
      alert('Gagal menyimpan: ' + err.message);
    }
    isSubmitting = false;
  }
</script>

<div class="min-h-screen bg-slate-50 py-12 px-4 sm:px-6 lg:px-8 flex justify-center">
  {#if loading}
    <div class="flex flex-col items-center justify-center py-32">
      <svg class="animate-spin h-10 w-10 text-emerald-600 mb-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
      <div class="text-emerald-600 font-bold animate-pulse text-xl">Memuat data lapak...</div>
    </div>
  {:else if lapak}
    <div class="w-full max-w-3xl bg-white rounded-3xl shadow-xl overflow-hidden border border-slate-100">
      <form onsubmit={simpanLapak}>
        
        <!-- ================= BAGIAN HEADER ALA FACEBOOK ================= -->
        <div class="relative h-48 sm:h-64 bg-slate-200 group">
          <!-- Gambar Banner -->
          {#if previewBanner}
            <img src={previewBanner} class="w-full h-full object-cover" alt="Banner Lapak" />
          {:else}
            <div class="w-full h-full bg-gradient-to-tr from-slate-300 to-slate-200 flex items-center justify-center">
              <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" class="text-slate-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><rect width="18" height="18" x="3" y="3" rx="2" ry="2"/><circle cx="9" cy="9" r="2"/><path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/></svg>
            </div>
          {/if}

          <!-- Overlay Ghoib Edit Banner (Muncul pas di Hover) -->
          <div class="absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 transition-opacity duration-300 flex items-center justify-center">
            <label class="cursor-pointer bg-white/20 hover:bg-white/30 backdrop-blur-md border border-white/50 text-white px-5 py-2.5 rounded-xl font-bold flex items-center gap-2 transition shadow-lg">
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3z"/><circle cx="12" cy="13" r="3"/></svg>
              Ganti Foto Sampul
              <input type="file" accept="image/*" class="hidden" onchange={handleBannerChange} />
            </label>
          </div>
        </div>

        <div class="px-6 sm:px-10 relative">
          <!-- Foto Profil (Numpuk di atas Banner) -->
          <div class="relative -mt-16 sm:-mt-20 mb-8 flex flex-col sm:flex-row sm:justify-between sm:items-end gap-4">
            
            <div class="relative group w-32 h-32 sm:w-40 sm:h-40 rounded-2xl border-4 border-white bg-white shadow-lg overflow-hidden flex-shrink-0 z-10">
              {#if previewFoto}
                <img src={previewFoto} class="w-full h-full object-cover" alt="Foto Utama" />
              {:else}
                <div class="w-full h-full bg-slate-100 flex items-center justify-center text-4xl">🛍️</div>
              {/if}

              <!-- Overlay Ghoib Edit Foto Utama -->
              <label class="absolute inset-0 bg-black/50 opacity-0 group-hover:opacity-100 transition-opacity duration-300 flex flex-col items-center justify-center cursor-pointer text-white">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round" class="mb-1"><path d="M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3z"/><circle cx="12" cy="13" r="3"/></svg>
                <span class="text-xs font-bold">Ubah Foto</span>
                <input type="file" accept="image/*" class="hidden" onchange={handleFotoChange} />
              </label>
            </div>

            <!-- Tombol Simpan -->
            <button type="submit" disabled={isSubmitting} class="w-full sm:w-auto bg-slate-900 hover:bg-emerald-600 text-white font-bold px-8 py-3 rounded-xl shadow-md hover:shadow-lg transition-all duration-300 disabled:opacity-50 flex items-center justify-center gap-2 mb-2 sm:mb-6">
              {#if isSubmitting}
                <svg class="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
                Menyimpan...
              {:else}
                <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"/><polyline points="17 21 17 13 7 13 7 21"/><polyline points="7 3 7 8 15 8"/></svg>
                Simpan Etalase
              {/if}
            </button>
          </div>

          <!-- ================= FORM INFO LAPAK ================= -->
          <div class="pb-10">
            <h2 class="text-2xl font-extrabold text-slate-800 mb-6">Informasi Lapak</h2>
            
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-6 mb-6">
              <div>
                <label class="block text-sm font-bold text-slate-700 mb-2">Nama Usaha / Lapak</label>
                <input type="text" bind:value={lapak.nama_usaha} required class="w-full border border-slate-300 bg-slate-50 p-3 rounded-xl focus:bg-white focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition" />
              </div>
              <div>
                <label class="block text-sm font-bold text-slate-700 mb-2">Kategori Usaha</label>
                <input type="text" bind:value={lapak.kategori} required class="w-full border border-slate-300 bg-slate-50 p-3 rounded-xl focus:bg-white focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition" />
              </div>
              <div>
                <label class="block text-sm font-bold text-slate-700 mb-2">Nomor WhatsApp (Aktif)</label>
                <div class="relative">
                  <span class="absolute left-4 top-3.5 text-slate-400">
                    <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"/></svg>
                  </span>
                  <input type="text" bind:value={lapak.nomor_wa} required class="w-full border border-slate-300 bg-slate-50 p-3 pl-11 rounded-xl focus:bg-white focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition" />
                </div>
              </div>
              <div>
                <label class="block text-sm font-bold text-slate-700 mb-2">Kisaran Harga Produk</label>
                <div class="relative">
                  <span class="absolute left-4 top-3 text-slate-500 font-bold">Rp</span>
                  <input type="text" bind:value={lapak.harga} required placeholder="Contoh: 10.000 - 50.000" class="w-full border border-slate-300 bg-slate-50 p-3 pl-11 rounded-xl focus:bg-white focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition" />
                </div>
              </div>
            </div>

            <div>
              <label class="block text-sm font-bold text-slate-700 mb-2">Deskripsi Lapak</label>
              <textarea bind:value={lapak.deskripsi} rows="4" placeholder="Ceritakan keunggulan produk Anda, jam buka, atau alamat detail..." class="w-full border border-slate-300 bg-slate-50 p-4 rounded-xl focus:bg-white focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition leading-relaxed"></textarea>
            </div>
            
          </div>
        </div>

      </form>
    </div>
  {/if}
</div>