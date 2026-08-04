<script>
  import { supabase } from '$lib/supabase';
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  // ================= STATE TAB & DATA =================
  let activeTab = $state('umkm'); // Pilihan: 'umkm', 'wisata', 'artikel'
  let loading = $state(true);
  let isSubmitting = $state(false);

  // Data UMKM
  let umkmList = $state([]);
  let antreanPending = $derived(umkmList.filter(item => item.status === 'pending'));
  let lapakTayang = $derived(umkmList.filter(item => item.status === 'approved'));

  // Data Wisata & Artikel
  let wisataList = $state([]);
  let artikelList = $state([]);

  // Form State Wisata (Sekarang pakai 1 input 'koordinat' aja biar gampang paste)
  let formWisata = $state({ nama_tempat: '', deskripsi: '', lokasi: '', koordinat: '', jalur_akses: '' });
  let fileWisata = $state(null); // Nampung file gambar

  let formArtikel = $state({ judul: '', konten: '', penulis: 'Admin Nagari' });
  let fileArtikel = $state(null);

  // ================= INIT =================
  onMount(async () => {
    const { data: { session } } = await supabase.auth.getSession();
    if (!session) {
      goto('/login');
      return;
    }
    fetchSemuaData();
  });

  async function fetchSemuaData() {
    loading = true;
    
    // Tarik UMKM
    const { data: dataUmkm } = await supabase.from('umkm').select('*').order('created_at', { ascending: false });
    if (dataUmkm) umkmList = dataUmkm;

    // Tarik Wisata
    const { data: dataWisata } = await supabase.from('wisata').select('*').order('created_at', { ascending: false });
    if (dataWisata) wisataList = dataWisata;

    // Tarik Artikel
    const { data: dataArtikel } = await supabase.from('artikel').select('*').order('created_at', { ascending: false });
    if (dataArtikel) artikelList = dataArtikel;

    loading = false;
  }

// ================= FUNGSI UPLOAD GAMBAR =================
  async function uploadGambar(file) {
    // 1. Cek ukuran file (Maksimal 1 MB)
    const batasMaksimal = 1 * 1024 * 1024; // 1 MB dalam satuan bytes
    if (file.size > batasMaksimal) {
      throw new Error('Maaf Bapak/Ibu Admin, ukuran foto maksimal 1 MB ya. Silakan kompres foto atau gunakan foto dari WhatsApp.');
    }

    // 2. Lanjut upload kalau ukuran aman
    const fileExt = file.name.split('.').pop();
    const fileName = `${Date.now()}_${Math.random().toString(36).substring(2)}.${fileExt}`;
    
    const { error } = await supabase.storage.from('foto_web').upload(fileName, file);
    if (error) throw error;

    const { data } = supabase.storage.from('foto_web').getPublicUrl(fileName);
    return data.publicUrl;
  }

  // ================= FUNGSI UMKM =================
  async function accLapak(id) {
    await supabase.from('umkm').update({ status: 'approved' }).eq('id', id);
    fetchSemuaData();
  }
  async function hapusLapak(id) {
    if (confirm('Hapus lapak permanen?')) {
      await supabase.from('umkm').delete().eq('id', id);
      fetchSemuaData();
    }
  }
  function copyLinkWarga(id) {
    navigator.clipboard.writeText(`${window.location.origin}/lapak/${id}`);
    alert('Link berhasil dicopy!');
  }

  // ================= FUNGSI WISATA =================
  async function tambahWisata(e) {
    e.preventDefault();
    isSubmitting = true;
    try {
      // 1. Belah koordinat pakai tanda koma (,)
      let lat = '';
      let lng = '';
      if (formWisata.koordinat) {
        const parts = formWisata.koordinat.split(',');
        if (parts.length === 2) {
          lat = parts[0].trim(); // Ambil angka kiri
          lng = parts[1].trim(); // Ambil angka kanan
        } else {
          alert('Format koordinat salah wok! Harus ada tanda koma (,) di tengahnya.');
          isSubmitting = false;
          return;
        }
      }

      // 2. Upload gambar (kalau ada)
      let fotoUrl = '';
      if (fileWisata && fileWisata.length > 0) {
        fotoUrl = await uploadGambar(fileWisata[0]);
      }
      
      // 3. Masukin ke Supabase (Database tetep nerima lat & lng terpisah)
      const { error } = await supabase.from('wisata').insert([{
        nama_tempat: formWisata.nama_tempat,
        deskripsi: formWisata.deskripsi,
        lokasi: formWisata.lokasi,
        foto_url: fotoUrl,
        latitude: lat,
        longitude: lng,
        jalur_akses: formWisata.jalur_akses
      }]);
      
      if (error) throw error;
      alert('Wisata berhasil ditambahkan!');
      
      // Reset form
      formWisata = { nama_tempat: '', deskripsi: '', lokasi: '', koordinat: '', jalur_akses: '' };
      fileWisata = null;
      fetchSemuaData();
    } catch (err) {
      alert('Gagal: ' + err.message);
    }
    isSubmitting = false;
  }

  async function hapusWisata(id) {
    if (confirm('Hapus wisata ini?')) {
      await supabase.from('wisata').delete().eq('id', id);
      fetchSemuaData();
    }
  }

  // ================= FUNGSI ARTIKEL =================
  async function tambahArtikel(e) {
    e.preventDefault();
    isSubmitting = true;
    try {
      let fotoUrl = '';
      if (fileArtikel && fileArtikel.length > 0) {
        fotoUrl = await uploadGambar(fileArtikel[0]);
      }
      
      const { error } = await supabase.from('artikel').insert([{
        judul: formArtikel.judul,
        konten: formArtikel.konten,
        penulis: formArtikel.penulis,
        foto_url: fotoUrl
      }]);
      
      if (error) throw error;
      alert('Artikel berhasil ditambahkan!');
      formArtikel = { judul: '', konten: '', penulis: 'Admin Nagari' };
      fileArtikel = null;
      fetchSemuaData();
    } catch (err) {
      alert('Gagal: ' + err.message);
    }
    isSubmitting = false;
  }

  async function hapusArtikel(id) {
    if (confirm('Hapus artikel ini?')) {
      await supabase.from('artikel').delete().eq('id', id);
      fetchSemuaData();
    }
  }

  async function logout() {
    await supabase.auth.signOut();
    goto('/login');
  }
</script>

<div class="min-h-screen bg-slate-100 p-6">
  <div class="max-w-6xl mx-auto">
    
    <!-- HEADER & TAB MENU -->
    <div class="bg-white p-4 rounded-xl shadow-sm mb-6">
      <div class="flex justify-between items-center mb-4">
        <h1 class="text-2xl font-bold text-slate-800">🛠️ Dashboard Admin Nagari</h1>
        <button onclick={logout} class="bg-red-50 text-red-600 px-4 py-2 rounded-lg text-sm font-semibold hover:bg-red-100">
          Keluar (Logout)
        </button>
      </div>
      
      <div class="flex gap-2 overflow-x-auto">
        <button onclick={() => activeTab = 'umkm'} class="px-6 py-2 rounded-lg font-medium transition {activeTab === 'umkm' ? 'bg-slate-800 text-white' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'}">🎪 Kelola UMKM</button>
        <button onclick={() => activeTab = 'wisata'} class="px-6 py-2 rounded-lg font-medium transition {activeTab === 'wisata' ? 'bg-slate-800 text-white' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'}">🏞️ Destinasi Wisata</button>
        <button onclick={() => activeTab = 'artikel'} class="px-6 py-2 rounded-lg font-medium transition {activeTab === 'artikel' ? 'bg-slate-800 text-white' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'}">📰 Berita & Artikel</button>
      </div>
    </div>

    {#if loading}
      <div class="text-center py-10 text-slate-500 font-medium">Memuat data Sirukam...</div>
    {:else}

      <!-- ================= TAB UMKM ================= -->
      {#if activeTab === 'umkm'}
        <div class="mb-8">
          <h2 class="text-lg font-bold text-slate-800 mb-4">⏳ Menunggu Persetujuan ({antreanPending.length})</h2>
          {#if antreanPending.length === 0}
            <div class="bg-white p-6 rounded-xl border border-dashed border-slate-300 text-center text-slate-500">Tidak ada antrean baru.</div>
          {:else}
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
              {#each antreanPending as item}
                <div class="bg-white p-5 rounded-xl shadow-sm border-l-4 border-amber-400">
                  <h3 class="font-bold text-lg">{item.nama_usaha}</h3>
                  <p class="text-sm text-slate-600 mb-3">{item.kategori} • {item.harga}</p>
                  <div class="flex gap-2">
                    <button onclick={() => accLapak(item.id)} class="flex-1 bg-emerald-600 text-white py-2 rounded hover:bg-emerald-700">Terima & Tayangkan</button>
                    <button onclick={() => hapusLapak(item.id)} class="bg-slate-100 text-slate-600 px-3 py-2 rounded hover:bg-red-100">Hapus</button>
                  </div>
                </div>
              {/each}
            </div>
          {/if}
        </div>

        <div>
          <h2 class="text-lg font-bold text-slate-800 mb-4">✅ Lapak Sudah Tayang ({lapakTayang.length})</h2>
          <div class="bg-white rounded-xl shadow-sm overflow-hidden border border-slate-200">
            <table class="w-full text-left text-sm">
              <thead class="bg-slate-50 border-b text-slate-600">
                <tr><th class="p-4">Nama Usaha</th><th class="p-4">Kategori</th><th class="p-4">WhatsApp</th><th class="p-4 text-center">Aksi</th></tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                {#each lapakTayang as item}
                  <tr>
                    <td class="p-4 font-medium">{item.nama_usaha}</td>
                    <td class="p-4">{item.kategori}</td>
                    <td class="p-4">{item.nomor_wa}</td>
                    <td class="p-4 text-center">
                      <button onclick={() => copyLinkWarga(item.id)} class="bg-slate-800 text-white px-3 py-1.5 rounded text-xs">Copy Link</button>
                    </td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        </div>
      {/if}

      <!-- ================= TAB WISATA ================= -->
      {#if activeTab === 'wisata'}
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
          
          <!-- Form Tambah Wisata -->
          <div class="lg:col-span-1 bg-white p-6 rounded-xl shadow-sm border border-slate-200 self-start">
            <h2 class="text-lg font-bold text-slate-800 mb-4">➕ Tambah Wisata</h2>
            <form onsubmit={tambahWisata} class="space-y-4">
              <div><label class="block text-sm text-slate-600 mb-1">Nama Tempat</label><input type="text" bind:value={formWisata.nama_tempat} required class="w-full border p-2 rounded focus:ring-2 focus:ring-slate-800 outline-none" /></div>
              <div><label class="block text-sm text-slate-600 mb-1">Deskripsi Singkat</label><textarea bind:value={formWisata.deskripsi} rows="3" required class="w-full border p-2 rounded focus:ring-2 focus:ring-slate-800 outline-none"></textarea></div>
              <div><label class="block text-sm text-slate-600 mb-1">Lokasi / Alamat</label><input type="text" bind:value={formWisata.lokasi} required class="w-full border p-2 rounded focus:ring-2 focus:ring-slate-800 outline-none" /></div>
              
              <!-- KOORDINAT (Garis Lintang & Bujur Gabungan) -->
              <div>
                <label class="block text-sm text-slate-600 mb-1">Koordinat Google Maps</label>
                <input type="text" bind:value={formWisata.koordinat} required placeholder="-0.8907196397905341, 100.75627895696661" class="w-full border p-2 rounded focus:ring-2 focus:ring-slate-800 outline-none" />
                <p class="text-xs text-slate-400 mt-1">Paste langsung angkanya dari Google Maps.</p>
              </div>

              <!-- JALUR AKSES -->
              <div>
                <label class="block text-sm text-slate-600 mb-1">Panduan Jalur / Rute Akses</label>
                <textarea bind:value={formWisata.jalur_akses} rows="2" placeholder="Contoh: Masuk dari pertigaan balai desa, jalan hanya muat motor..." class="w-full border p-2 rounded focus:ring-2 focus:ring-slate-800 outline-none"></textarea>
              </div>

              <div><label class="block text-sm text-slate-600 mb-1">Upload Foto</label><input type="file" accept="image/*" bind:files={fileWisata} required class="w-full border p-2 rounded text-sm" /></div>
              
              <button type="submit" disabled={isSubmitting} class="w-full bg-blue-600 text-white py-2 rounded font-medium hover:bg-blue-700 disabled:opacity-50">
                {isSubmitting ? 'Mengupload...' : 'Simpan Wisata'}
              </button>
            </form>
          </div>

          <!-- Tabel Data Wisata -->
          <div class="lg:col-span-2 bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
             <table class="w-full text-left text-sm">
              <thead class="bg-slate-50 border-b text-slate-600">
                <tr><th class="p-4">Foto</th><th class="p-4">Nama Tempat</th><th class="p-4">Lokasi & Koordinat</th><th class="p-4 text-center">Aksi</th></tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                {#each wisataList as item}
                  <tr>
                    <td class="p-4"><img src={item.foto_url} alt="wisata" class="w-16 h-12 object-cover rounded" /></td>
                    <td class="p-4 font-medium">{item.nama_tempat}</td>
                    <td class="p-4 text-slate-600">
                      {item.lokasi}<br/>
                      <span class="text-xs text-blue-600 font-mono">Lat: {item.latitude} | Lng: {item.longitude}</span>
                    </td>
                    <td class="p-4 text-center"><button onclick={() => hapusWisata(item.id)} class="text-red-500 hover:text-red-700 font-medium">Hapus</button></td>
                  </tr>
                {:else}
                  <tr><td colspan="4" class="p-6 text-center text-slate-500">Belum ada data wisata.</td></tr>
                {/each}
              </tbody>
            </table>
          </div>
        </div>
      {/if}

      <!-- ================= TAB ARTIKEL ================= -->
      {#if activeTab === 'artikel'}
         <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
          <!-- Form Tambah Artikel -->
          <div class="lg:col-span-1 bg-white p-6 rounded-xl shadow-sm border border-slate-200 self-start">
            <h2 class="text-lg font-bold text-slate-800 mb-4">✍️ Tulis Berita</h2>
            <form onsubmit={tambahArtikel} class="space-y-4">
              <div><label class="block text-sm text-slate-600 mb-1">Judul Artikel</label><input type="text" bind:value={formArtikel.judul} required class="w-full border p-2 rounded focus:ring-2 focus:ring-slate-800 outline-none" /></div>
              <div><label class="block text-sm text-slate-600 mb-1">Penulis</label><input type="text" bind:value={formArtikel.penulis} required class="w-full border p-2 rounded focus:ring-2 focus:ring-slate-800 outline-none" /></div>
              <div><label class="block text-sm text-slate-600 mb-1">Konten Berita</label><textarea bind:value={formArtikel.konten} rows="5" required class="w-full border p-2 rounded focus:ring-2 focus:ring-slate-800 outline-none"></textarea></div>
              <div><label class="block text-sm text-slate-600 mb-1">Thumbnail Foto</label><input type="file" accept="image/*" bind:files={fileArtikel} required class="w-full border p-2 rounded text-sm" /></div>
              
              <button type="submit" disabled={isSubmitting} class="w-full bg-purple-600 text-white py-2 rounded font-medium hover:bg-purple-700 disabled:opacity-50">
                {isSubmitting ? 'Mengupload...' : 'Terbitkan Berita'}
              </button>
            </form>
          </div>

          <!-- Tabel Data Artikel -->
          <div class="lg:col-span-2 bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
             <table class="w-full text-left text-sm">
              <thead class="bg-slate-50 border-b text-slate-600">
                <tr><th class="p-4">Thumbnail</th><th class="p-4">Judul & Penulis</th><th class="p-4 text-center">Aksi</th></tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                {#each artikelList as item}
                  <tr>
                    <td class="p-4"><img src={item.foto_url} alt="artikel" class="w-16 h-12 object-cover rounded" /></td>
                    <td class="p-4"><p class="font-bold text-slate-800">{item.judul}</p><p class="text-xs text-slate-500">Oleh: {item.penulis}</p></td>
                    <td class="p-4 text-center"><button onclick={() => hapusArtikel(item.id)} class="text-red-500 hover:text-red-700 font-medium">Hapus</button></td>
                  </tr>
                {:else}
                  <tr><td colspan="3" class="p-6 text-center text-slate-500">Belum ada artikel.</td></tr>
                {/each}
              </tbody>
            </table>
          </div>
        </div>
      {/if}

    {/if}
  </div>
</div>