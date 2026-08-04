<script>
  import { page } from '$app/stores';
  import { supabase } from '$lib/supabase';
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  const lapakId = $page.params.id;

  let formNama = $state('');
  let formKategori = $state('');
  let formHarga = $state('');
  let formWa = $state('');
  let formDeskripsi = $state(''); 
  
  // Nampung file gambar yang diupload warga
  let fileBanner = $state(null);
  let fileFoto = $state(null);
  
  // Nampung link gambar yang udah ada di database (buat preview)
  let currentBanner = $state('');
  let currentFoto = $state('');

  let loading = $state(true);
  let saving = $state(false);
  let notFound = $state(false);

  onMount(async () => {
    const { data, error } = await supabase
      .from('umkm')
      .select('*')
      .eq('id', lapakId)
      .single();

    if (error || !data) {
      notFound = true;
    } else {
      formNama = data.nama_usaha || '';
      formKategori = data.kategori || '';
      formHarga = data.harga || '';
      formWa = data.nomor_wa || '';
      formDeskripsi = data.deskripsi || '';
      currentBanner = data.banner_url || '';
      currentFoto = data.foto_url || '';
    }
    loading = false;
  });

  // ================= FUNGSI UPLOAD GAMBAR =================
  async function uploadGambar(file) {
    const batasMaksimal = 2 * 1024 * 1024; // 1 MB
    if (file.size > batasMaksimal) {
      throw new Error('Ukuran foto maksimal 2 MB ! Silakan kompres dulu sebelum upload.');
    }

    const fileExt = file.name.split('.').pop();
    const fileName = `umkm_${Date.now()}_${Math.random().toString(36).substring(2)}.${fileExt}`;
    
    // Kita pinjem bucket 'foto_web' yang udah dibikin buat naruh foto warga
    const { error } = await supabase.storage.from('foto_web').upload(fileName, file);
    if (error) throw error;

    const { data } = supabase.storage.from('foto_web').getPublicUrl(fileName);
    return data.publicUrl;
  }

  // ================= FUNGSI UPDATE DATA =================
  async function handleUpdate(e) {
    e.preventDefault();
    saving = true;

    try {
      const cleanPhone = formWa.replace(/\D/g, '');
      
      let bannerUrl = currentBanner;
      let fotoUrl = currentFoto;

      // 1. Kalau warga milih file banner baru, upload!
      if (fileBanner && fileBanner.length > 0) {
        bannerUrl = await uploadGambar(fileBanner[0]);
      }

      // 2. Kalau warga milih file foto baru, upload!
      if (fileFoto && fileFoto.length > 0) {
        fotoUrl = await uploadGambar(fileFoto[0]);
      }

      // 3. Simpan ke database
      const { error } = await supabase
        .from('umkm')
        .update({
          nama_usaha: formNama,
          kategori: formKategori,
          harga: formHarga,
          nomor_wa: cleanPhone,
          deskripsi: formDeskripsi,
          banner_url: bannerUrl,
          foto_url: fotoUrl,
          status: 'pending' // Kembalikan jadi pending biar di-ACC ulang Admin
        })
        .eq('id', lapakId);

      if (error) throw error;
      
      alert('Perubahan berhasil disimpan! Data sedang menunggu ACC ulang dari Admin.');
      
      // Update preview foto di layar
      currentBanner = bannerUrl;
      currentFoto = fotoUrl;
      fileBanner = null;
      fileFoto = null;

    } catch (err) {
      alert('Gagal menyimpan: ' + err.message);
    }
    
    saving = false;
  }
</script>

<div class="min-h-screen bg-slate-50 py-12 px-4 flex justify-center">
  <div class="bg-white p-8 rounded-2xl shadow-sm border border-gray-100 max-w-2xl w-full">
    
    {#if loading}
      <div class="text-center py-10 text-slate-500 font-medium animate-pulse">Mencari data lapak Anda...</div>
    {:else if notFound}
      <div class="text-center py-10">
        <div class="text-4xl mb-4">🕵️‍♂️</div>
        <h2 class="text-xl font-bold text-slate-800 mb-2">Lapak Tidak Ditemukan</h2>
        <p class="text-slate-600 mb-6 text-sm">Pastikan Link Rahasia yang Anda buka sudah benar dan lengkap.</p>
        <button onclick={() => goto('/')} class="bg-slate-800 text-white px-6 py-2.5 rounded-lg font-medium hover:bg-slate-700 transition">
          Kembali ke Beranda
        </button>
      </div>
    {:else}
      <h1 class="text-2xl font-bold text-slate-800 mb-2">Kelola Etalase Lapak</h1>
      <p class="text-slate-500 mb-8 text-sm">Lengkapi foto dan deskripsi agar lapak Anda lebih menarik pembeli.</p>

      <form onsubmit={handleUpdate} class="space-y-6">
        
        <!-- BAGIAN GAMBAR -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6 bg-slate-50 p-4 rounded-xl border border-slate-100">
          <!-- Upload Banner -->
          <div>
            <label class="block text-sm font-semibold text-slate-700 mb-2">Banner (Foto Sampul)</label>
            {#if currentBanner}
              <img src={currentBanner} alt="Banner" class="w-full h-24 object-cover rounded-lg mb-2 border border-slate-200" />
            {/if}
            <input type="file" accept="image/*" bind:files={fileBanner} class="w-full text-sm text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100" />
            <p class="text-xs text-slate-400 mt-1">Format lanskap (melebar) direkomendasikan.</p>
          </div>

          <!-- Upload Foto Produk -->
          <div>
            <label class="block text-sm font-semibold text-slate-700 mb-2">Foto Utama Produk</label>
            {#if currentFoto}
              <img src={currentFoto} alt="Foto Produk" class="w-24 h-24 object-cover rounded-lg mb-2 border border-slate-200" />
            {/if}
            <input type="file" accept="image/*" bind:files={fileFoto} class="w-full text-sm text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100" />
            <p class="text-xs text-slate-400 mt-1">Foto andalan jualan Anda.</p>
          </div>
        </div>

        <!-- BAGIAN TEKS -->
        <div class="space-y-4">
          <div>
            <label class="block text-sm font-semibold text-slate-600 mb-1">Nama Usaha / Lapak</label>
            <input type="text" bind:value={formNama} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none" />
          </div>
          
          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-semibold text-slate-600 mb-1">Kategori Usaha</label>
              <input type="text" bind:value={formKategori} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none" placeholder="Contoh: Makanan, Jasa, Kriya" />
            </div>
            <div>
              <label class="block text-sm font-semibold text-slate-600 mb-1">Nomor WhatsApp</label>
              <input type="tel" bind:value={formWa} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none" />
            </div>
          </div>

          <div>
            <label class="block text-sm font-semibold text-slate-600 mb-1">Kisaran Harga Produk</label>
            <input type="text" bind:value={formHarga} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none" placeholder="Contoh: Rp 10.000 - Rp 50.000" />
          </div>

          <div>
            <label class="block text-sm font-semibold text-slate-600 mb-1">Deskripsi Lapak</label>
            <textarea bind:value={formDeskripsi} rows="4" class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none" placeholder="Ceritakan detail produk, jam buka, atau lokasi lapak Anda..."></textarea>
          </div>
        </div>

        <button type="submit" disabled={saving} class="w-full bg-blue-600 text-white py-3 rounded-lg font-bold text-lg hover:bg-blue-700 transition shadow-md mt-6 disabled:opacity-50">
          {saving ? 'Mengupload & Menyimpan...' : 'Simpan Etalase Lapak'}
        </button>
      </form>
    {/if}

  </div>
</div>