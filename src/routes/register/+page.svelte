<script>
  import { supabase } from '$lib/supabase';
  import { goto } from '$app/navigation';

  let formNama = $state('');
  let formKategori = $state('');
  let formHarga = $state('');
  let formWa = $state('');
  let loading = $state(false);
  let isSuccess = $state(false);
  let magicLink = $state('');
  let copyText = $state('Copy Link Lapak');

  async function handleSubmit(e) {
    e.preventDefault();
    loading = true;

    const cleanPhone = formWa.replace(/\D/g, '');

    // Tembak data dan tambahin .select() di belakang buat ngambil ID uniknya
    const { data, error } = await supabase.from('umkm').insert([
      {
        nama_usaha: formNama,
        kategori: formKategori,
        harga: formHarga,
        nomor_wa: cleanPhone,
        status: 'pending'
      }
    ]).select();

    if (error) {
      alert('Gagal mengirim pendaftaran: ' + error.message);
    } else if (data && data.length > 0) {
      // Bikin Link Rahasia dari ID yang baru aja dibuat Supabase
      const lapakId = data[0].id;
      magicLink = `${window.location.origin}/lapak/${lapakId}`;
      isSuccess = true; 
    }
    
    loading = false;
  }

  // Fungsi buat ngopi link ke clipboard (HP/Laptop)
  function copyToClipboard() {
    navigator.clipboard.writeText(magicLink);
    copyText = 'Tercopy! ✓';
    setTimeout(() => { copyText = 'Copy Link Lapak'; }, 3000);
  }
</script>

<div class="min-h-screen bg-slate-50 flex items-center justify-center p-4">
  <div class="bg-white p-8 rounded-2xl shadow-sm border border-gray-100 max-w-lg w-full">
    
    {#if isSuccess}
      <div class="text-center py-4">
        <div class="w-16 h-16 bg-emerald-100 text-emerald-600 rounded-full flex items-center justify-center mx-auto mb-4 text-3xl">✓</div>
        <h2 class="text-2xl font-bold text-slate-800 mb-2">Data Berhasil Masuk!</h2>
        <p class="text-slate-600 mb-6 text-sm">Data Anda sedang menunggu ACC Admin. <br> <b>PENTING:</b> Simpan Link Rahasia di bawah ini untuk mengedit data lapak Anda kapan saja.</p>
        
        <div class="bg-slate-50 border border-slate-200 p-4 rounded-lg mb-6">
          <p class="text-xs font-semibold text-slate-500 mb-2 text-left">Link Lapak Rahasia Anda:</p>
          <div class="flex items-center gap-2">
            <input type="text" readonly value={magicLink} class="w-full bg-white border border-slate-300 rounded p-2 text-sm text-slate-700 outline-none" />
            <button onclick={copyToClipboard} class="bg-slate-800 text-white px-4 py-2 rounded text-sm font-medium hover:bg-slate-700 transition whitespace-nowrap">
              {copyText}
            </button>
          </div>
          <p class="text-xs text-red-500 text-left mt-2 font-medium">⚠️ Jangan berikan link ini ke sembarang orang!</p>
        </div>

        <button onclick={() => goto('/')} class="w-full bg-emerald-600 text-white px-6 py-2.5 rounded-lg font-medium hover:bg-emerald-700 transition">
          Kembali ke Beranda
        </button>
      </div>
    {:else}
      <h1 class="text-2xl font-bold text-slate-800 mb-2">Daftar UMKM Nagari Sirukam</h1>
      <p class="text-slate-500 mb-6 text-sm">Isi form di bawah ini untuk mempromosikan usaha Anda di website desa. Gratis!</p>

      <form onsubmit={handleSubmit} class="space-y-4">
        <div>
          <label class="block text-sm font-semibold text-slate-600 mb-1">Nama Usaha Anda</label>
          <input type="text" bind:value={formNama} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-emerald-500 focus:outline-none" placeholder="Contoh: Sarapan Pagi Mak Itam" />
        </div>
        <div>
          <label class="block text-sm font-semibold text-slate-600 mb-1">Kategori Usaha</label>
          <input type="text" bind:value={formKategori} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-emerald-500 focus:outline-none" placeholder="Contoh: Kuliner / Kerajinan / Pertanian" />
        </div>
        <div>
          <label class="block text-sm font-semibold text-slate-600 mb-1">Kisaran Harga Produk</label>
          <input type="text" bind:value={formHarga} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-emerald-500 focus:outline-none" placeholder="Contoh: Rp 10.000 - Rp 50.000" />
        </div>
        <div>
          <label class="block text-sm font-semibold text-slate-600 mb-1">Nomor WhatsApp Aktif</label>
          <input type="tel" bind:value={formWa} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-emerald-500 focus:outline-none" placeholder="081234567890" />
        </div>

        <button type="submit" disabled={loading} class="w-full bg-emerald-600 text-white py-3 rounded-lg font-medium hover:bg-emerald-700 transition shadow mt-4">
          {loading ? 'Mengirim Data...' : 'Kirim Pendaftaran'}
        </button>
      </form>
    {/if}

  </div>
</div>