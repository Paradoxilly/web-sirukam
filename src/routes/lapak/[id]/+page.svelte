<script>
  import { page } from '$app/stores';
  import { supabase } from '$lib/supabase';
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  // Nangkep ID dari URL Link Rahasia
  const lapakId = $page.params.id;

  let formNama = $state('');
  let formKategori = $state('');
  let formHarga = $state('');
  let formWa = $state('');
  
  let loading = $state(true);
  let saving = $state(false);
  let notFound = $state(false);

  // Pas halaman dibuka, langsung cari data UMKM berdasarkan ID di URL
  onMount(async () => {
    const { data, error } = await supabase
      .from('umkm')
      .select('*')
      .eq('id', lapakId)
      .single();

    if (error || !data) {
      notFound = true;
    } else {
      formNama = data.nama_usaha;
      formKategori = data.kategori;
      formHarga = data.harga;
      formWa = data.nomor_wa;
    }
    loading = false;
  });

  // Fungsi buat nyimpen perubahan data
  async function handleUpdate(e) {
    e.preventDefault();
    saving = true;

    // Bersihin nomor WA jaga-jaga kalau warga ngetik spasi
    const cleanPhone = formWa.replace(/\D/g, '');

    const { error } = await supabase
      .from('umkm')
      .update({
        nama_usaha: formNama,
        kategori: formKategori,
        harga: formHarga,
        nomor_wa: cleanPhone,
        // Status otomatis kita balikin jadi 'pending' kalau diedit, 
        // biar admin nge-ACC ulang (opsional, tapi bagus buat keamanan desa)
        status: 'pending' 
      })
      .eq('id', lapakId);

    if (error) {
      alert('Gagal menyimpan: ' + error.message);
    } else {
      alert('Perubahan berhasil disimpan! Data sedang menunggu ACC ulang dari Admin.');
    }
    saving = false;
  }
</script>

<div class="min-h-screen bg-slate-50 flex items-center justify-center p-4">
  <div class="bg-white p-8 rounded-2xl shadow-sm border border-gray-100 max-w-lg w-full">
    
    {#if loading}
      <div class="text-center py-10 text-slate-500">Mencari data lapak Anda...</div>
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
      <h1 class="text-2xl font-bold text-slate-800 mb-2">Edit Data Lapak</h1>
      <p class="text-slate-500 mb-6 text-sm">Ubah data lapak Anda di bawah ini dan klik Simpan.</p>

      <form onsubmit={handleUpdate} class="space-y-4">
        <div>
          <label class="block text-sm font-semibold text-slate-600 mb-1">Nama Usaha</label>
          <input type="text" bind:value={formNama} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none" />
        </div>
        <div>
          <label class="block text-sm font-semibold text-slate-600 mb-1">Kategori Usaha</label>
          <input type="text" bind:value={formKategori} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none" />
        </div>
        <div>
          <label class="block text-sm font-semibold text-slate-600 mb-1">Kisaran Harga Produk</label>
          <input type="text" bind:value={formHarga} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none" />
        </div>
        <div>
          <label class="block text-sm font-semibold text-slate-600 mb-1">Nomor WhatsApp Aktif</label>
          <input type="tel" bind:value={formWa} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-blue-500 focus:outline-none" />
        </div>

        <button type="submit" disabled={saving} class="w-full bg-blue-600 text-white py-3 rounded-lg font-medium hover:bg-blue-700 transition shadow mt-4">
          {saving ? 'Menyimpan...' : 'Simpan Perubahan'}
        </button>
      </form>
    {/if}

  </div>
</div>