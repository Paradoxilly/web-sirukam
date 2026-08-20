<script>
  import { supabase } from '$lib/supabase';
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  // ================= STATE TAB & DATA =================
  let activeTab = $state('inbox'); // Default langsung buka inbox wok!
  let loading = $state(true);
  let isSubmitting = $state(false);

  // Data UMKM
  let umkmList = $state([]);
  let antreanPending = $derived(umkmList.filter(item => item.status === 'pending'));
  let lapakTayang = $derived(umkmList.filter(item => item.status === 'approved'));

  // Data Wisata & Artikel
  let wisataList = $state([]);
  let artikelList = $state([]);

  // ================= STATE KOTAK MASUK (BARU) =================
  let pesanList = $state([]);
  let pesanBelumDibaca = $derived(pesanList.filter(p => !p.is_read).length);

  // ================= STATE FORM WISATA =================
  let formWisata = $state({ nama_tempat: '', deskripsi: '', lokasi: '', koordinat: '', jalur_akses: '' });
  let fileWisata = $state(null);
  let editWisataId = $state(null); 
  let currentWisataFoto = $state(''); 

  // ================= STATE FORM ARTIKEL =================
  let formArtikel = $state({ judul: '', konten: '', penulis: 'Admin Nagari' });
  let fileArtikel = $state(null);
  let editArtikelId = $state(null); 
  let currentArtikelFoto = $state('');

  // ================= STATE GALERI ARTIKEL (MODAL) =================
  let isGaleriOpen = $state(false);
  let selectedArtikel = $state(null);
  let galeriList = $state([]);
  let isUploadingGaleri = $state(false);

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

    // Tarik Data Pesan Masuk
    const { data: dataPesan } = await supabase.from('pesan_masuk').select('*').order('created_at', { ascending: false });
    if (dataPesan) pesanList = dataPesan;

    const { data: dataUmkm } = await supabase.from('umkm').select('*').order('created_at', { ascending: false });
    if (dataUmkm) umkmList = dataUmkm;

    const { data: dataWisata } = await supabase.from('wisata').select('*').order('created_at', { ascending: false });
    if (dataWisata) wisataList = dataWisata;

    const { data: dataArtikel } = await supabase.from('artikel').select('*').order('created_at', { ascending: false });
    if (dataArtikel) artikelList = dataArtikel;
    
    loading = false;
  }

  // ================= FUNGSI PESAN MASUK (BARU) =================
  async function tandaiDibaca(id) {
    await supabase.from('pesan_masuk').update({ is_read: true }).eq('id', id);
    fetchSemuaData();
  }

  async function hapusPesan(id) {
    if (confirm('Yakin ingin menghapus pesan ini permanen?')) {
      await supabase.from('pesan_masuk').delete().eq('id', id);
      fetchSemuaData();
    }
  }

  // ================= FUNGSI UPLOAD GAMBAR =================
  async function uploadGambar(file) {
    const batasMaksimal = 2 * 1024 * 1024; // Limit 2MB
    if (file.size > batasMaksimal) {
      throw new Error('Maaf Bapak/Ibu Admin, ukuran foto maksimal 2 MB ya. Silakan kompres foto.');
    }
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
    if (confirm('Yakin ingin menghapus lapak ini secara permanen?')) {
      await supabase.from('umkm').delete().eq('id', id);
      fetchSemuaData();
    }
  }
  function copyLinkWarga(id) {
    navigator.clipboard.writeText(`${window.location.origin}/lapak/${id}`);
    alert('Link berhasil disalin! Silakan kirim ke warga.');
  }

  // ================= FUNGSI CRUD WISATA =================
  function setEditWisata(item) {
    editWisataId = item.id;
    currentWisataFoto = item.foto_url;
    formWisata = {
      nama_tempat: item.nama_tempat,
      deskripsi: item.deskripsi,
      lokasi: item.lokasi,
      koordinat: `${item.latitude}, ${item.longitude}`,
      jalur_akses: item.jalur_akses || ''
    };
    fileWisata = null;
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }

  function batalEditWisata() {
    editWisataId = null;
    currentWisataFoto = '';
    formWisata = { nama_tempat: '', deskripsi: '', lokasi: '', koordinat: '', jalur_akses: '' };
    fileWisata = null;
  }

  async function simpanWisata(e) {
    e.preventDefault();
    isSubmitting = true;
    try {
      let lat = '', lng = '';
      if (formWisata.koordinat) {
        const parts = formWisata.koordinat.split(',');
        if (parts.length === 2) {
          lat = parts[0].trim();
          lng = parts[1].trim();
        } else {
          throw new Error('Format koordinat salah! Harus ada tanda koma (,) di tengahnya.');
        }
      }

      let fotoUrl = currentWisataFoto; 
      
      // LOGIKA BARU: BISA UPLOAD BANYAK FOTO (Maks 4)
      if (fileWisata && fileWisata.length > 0) {
        if (fileWisata.length > 4) {
          throw new Error('Maaf wok, maksimal upload 4 foto wisata sekaligus!');
        }
        let urls = [];
        for (let i = 0; i < fileWisata.length; i++) {
          urls.push(await uploadGambar(fileWisata[i]));
        }
        fotoUrl = urls.join(','); // Gabungin link-nya pake koma
      }
      
      const dataPayload = {
        nama_tempat: formWisata.nama_tempat,
        deskripsi: formWisata.deskripsi,
        lokasi: formWisata.lokasi,
        foto_url: fotoUrl,
        latitude: lat,
        longitude: lng,
        jalur_akses: formWisata.jalur_akses
      };

      if (editWisataId) {
        await supabase.from('wisata').update(dataPayload).eq('id', editWisataId);
        alert('Data Wisata berhasil diperbarui!');
      } else {
        await supabase.from('wisata').insert([dataPayload]);
        alert('Wisata baru berhasil ditambahkan!');
      }
      
      batalEditWisata();
      fetchSemuaData();
    } catch (err) {
      alert('Gagal: ' + err.message);
    }
    isSubmitting = false;
  }

  async function hapusWisata(id) {
    if (confirm('Yakin ingin menghapus destinasi wisata ini?')) {
      await supabase.from('wisata').delete().eq('id', id);
      fetchSemuaData();
    }
  }

  // ================= FUNGSI CRUD ARTIKEL =================
  function setEditArtikel(item) {
    editArtikelId = item.id;
    currentArtikelFoto = item.foto_url;
    formArtikel = {
      judul: item.judul,
      konten: item.konten,
      penulis: item.penulis
    };
    fileArtikel = null;
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }

  function batalEditArtikel() {
    editArtikelId = null;
    currentArtikelFoto = '';
    formArtikel = { judul: '', konten: '', penulis: 'Admin Nagari' };
    fileArtikel = null;
  }

  async function simpanArtikel(e) {
    e.preventDefault();
    isSubmitting = true;
    try {
      let fotoUrl = currentArtikelFoto;
      if (fileArtikel && fileArtikel.length > 0) {
        fotoUrl = await uploadGambar(fileArtikel[0]);
      }
      
      const dataPayload = {
        judul: formArtikel.judul,
        konten: formArtikel.konten,
        penulis: formArtikel.penulis,
        foto_url: fotoUrl
      };

      if (editArtikelId) {
        await supabase.from('artikel').update(dataPayload).eq('id', editArtikelId);
        alert('Artikel berhasil diperbarui!');
      } else {
        await supabase.from('artikel').insert([dataPayload]);
        alert('Artikel baru berhasil diterbitkan!');
      }
      
      batalEditArtikel();
      fetchSemuaData();
    } catch (err) {
      alert('Gagal: ' + err.message);
    }
    isSubmitting = false;
  }

  async function hapusArtikel(id) {
    if (confirm('Yakin ingin menghapus artikel ini?')) {
      await supabase.from('artikel').delete().eq('id', id);
      fetchSemuaData();
    }
  }

  // ================= FUNGSI KELOLA GALERI (MODAL) =================
  async function bukaGaleri(artikel) {
    selectedArtikel = artikel;
    isGaleriOpen = true;
    await fetchGaleri(artikel.id);
  }

  function tutupGaleri() {
    isGaleriOpen = false;
    selectedArtikel = null;
    galeriList = [];
  }

  async function fetchGaleri(artikelId) {
    const { data } = await supabase.from('galeri_berita').select('*').eq('artikel_id', artikelId).order('created_at', { ascending: false });
    if (data) galeriList = data;
  }

  async function tambahFotoGaleri(e) {
    const file = e.target.files[0];
    if (!file) return;

    isUploadingGaleri = true;
    try {
      const fotoUrl = await uploadGambar(file); 
      
      const { error } = await supabase.from('galeri_berita').insert({
        artikel_id: selectedArtikel.id,
        foto_url: fotoUrl
      });
      if (error) throw error;
      
      await fetchGaleri(selectedArtikel.id); 
    } catch (err) {
      alert('Gagal upload: ' + err.message);
    }
    isUploadingGaleri = false;
  }

  async function hapusFotoGaleri(idGaleri) {
    if (confirm('Hapus foto ini dari galeri?')) {
      await supabase.from('galeri_berita').delete().eq('id', idGaleri);
      galeriList = galeriList.filter(g => g.id !== idGaleri);
    }
  }

  async function logout() {
    await supabase.auth.signOut();
    goto('/login');
  }
</script>

<div class="min-h-screen bg-slate-50 p-6 relative">
  <div class="max-w-7xl mx-auto">
    
    <!-- HEADER & TAB MENU -->
    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 mb-6">
      <div class="flex justify-between items-center mb-6">
        <h1 class="text-2xl font-bold text-slate-800 flex items-center gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-emerald-600"><rect width="7" height="9" x="3" y="3" rx="1"/><rect width="7" height="5" x="14" y="3" rx="1"/><rect width="7" height="9" x="14" y="12" rx="1"/><rect width="7" height="5" x="3" y="16" rx="1"/></svg>
          Dashboard Admin Nagari
        </h1>
        <button onclick={logout} class="bg-red-50 text-red-600 px-5 py-2 rounded-xl text-sm font-bold hover:bg-red-100 transition flex items-center gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" x2="9" y1="12" y2="12"/></svg>
          Keluar
        </button>
      </div>
      
      <div class="flex gap-2 overflow-x-auto border-b border-slate-100 pb-2">
        <!-- TAB KOTAK MASUK -->
        <button onclick={() => activeTab = 'inbox'} class="flex items-center gap-2 px-6 py-2.5 rounded-xl font-semibold transition {activeTab === 'inbox' ? 'bg-slate-800 text-white shadow-md' : 'text-slate-500 hover:bg-slate-100'}">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/></svg>
          Kotak Masuk
          {#if pesanBelumDibaca > 0}
            <span class="bg-red-500 text-white text-[10px] font-bold px-2 py-0.5 rounded-full">{pesanBelumDibaca}</span>
          {/if}
        </button>

        <!-- TAB UMKM -->
        <button onclick={() => activeTab = 'umkm'} class="flex items-center gap-2 px-6 py-2.5 rounded-xl font-semibold transition {activeTab === 'umkm' ? 'bg-slate-800 text-white shadow-md' : 'text-slate-500 hover:bg-slate-100'}">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M2 9a3 3 0 0 1 0 6v2a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-2a3 3 0 0 1 0-6V7a2 2 0 0 0-2-2H4a2 2 0 0 0-2 2Z"/><path d="M13 18H7"/><path d="M7 14h.01"/></svg>
          Kelola UMKM
        </button>

        <!-- TAB WISATA -->
        <button onclick={() => activeTab = 'wisata'} class="flex items-center gap-2 px-6 py-2.5 rounded-xl font-semibold transition {activeTab === 'wisata' ? 'bg-slate-800 text-white shadow-md' : 'text-slate-500 hover:bg-slate-100'}">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m8 3 4 8 5-5 5 15H2L8 3z"/></svg>
          Destinasi Wisata
        </button>

        <!-- TAB ARTIKEL -->
        <button onclick={() => activeTab = 'artikel'} class="flex items-center gap-2 px-6 py-2.5 rounded-xl font-semibold transition {activeTab === 'artikel' ? 'bg-slate-800 text-white shadow-md' : 'text-slate-500 hover:bg-slate-100'}">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 22h16a2 2 0 0 0 2-2V4a2 2 0 0 0-2-2H8a2 2 0 0 0-2 2v16a2 2 0 0 1-2 2Zm0 0a2 2 0 0 1-2-2v-9c0-1.1.9-2 2-2h2"/><path d="M18 14h-8"/><path d="M15 18h-5"/><path d="M10 6h8v4h-8V6Z"/></svg>
          Berita & Artikel
        </button>
      </div>
    </div>

    {#if loading}
      <div class="text-center py-20 text-slate-500 font-bold animate-pulse text-lg">Sinkronisasi dengan server...</div>
    {:else}

      <!-- ================= TAB KOTAK MASUK (BARU) ================= -->
      {#if activeTab === 'inbox'}
        <div>
          <h2 class="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
            <span class="w-2 h-2 rounded-full {pesanBelumDibaca > 0 ? 'bg-red-500 animate-pulse' : 'bg-slate-300'}"></span>
            Pesan Masuk ({pesanList.length})
          </h2>

          {#if pesanList.length === 0}
            <div class="bg-white p-8 rounded-2xl border border-dashed border-slate-300 text-center text-slate-500 font-medium">Kotak masuk masih kosong.</div>
          {:else}
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
              {#each pesanList as pesan}
                <div class="bg-white p-6 rounded-2xl shadow-sm border transition-all duration-300 relative {pesan.is_read ? 'border-slate-200' : 'border-emerald-500 ring-2 ring-emerald-50'}">
                  
                  {#if !pesan.is_read}
                    <div class="absolute -top-3 -right-3 bg-red-500 text-white text-[10px] font-bold px-3 py-1 rounded-full shadow-md uppercase tracking-wide">Baru</div>
                  {/if}

                  <div class="mb-4 pb-4 border-b border-slate-100">
                    <h3 class="font-bold text-lg text-slate-800">{pesan.nama}</h3>
                    <a href="mailto:{pesan.email}" class="text-sm text-blue-600 hover:underline font-medium break-all">{pesan.email}</a>
                    <p class="text-[11px] text-slate-400 mt-1">{new Date(pesan.created_at).toLocaleString('id-ID')}</p>
                  </div>
                  
                  <div class="mb-6 h-24 overflow-y-auto custom-scrollbar">
                    <p class="text-slate-600 text-sm leading-relaxed whitespace-pre-wrap">{pesan.pesan}</p>
                  </div>

                  <div class="flex gap-2">
                    {#if !pesan.is_read}
                      <button onclick={() => tandaiDibaca(pesan.id)} class="flex-1 bg-emerald-50 text-emerald-600 py-2 rounded-xl text-sm font-bold hover:bg-emerald-100 transition">Tandai Dibaca</button>
                    {/if}
                    <button onclick={() => hapusPesan(pesan.id)} class="flex-1 bg-red-50 text-red-600 py-2 rounded-xl text-sm font-bold hover:bg-red-100 transition">Hapus</button>
                  </div>
                </div>
              {/each}
            </div>
          {/if}
        </div>
      {/if}

      <!-- ================= TAB UMKM ================= -->
      {#if activeTab === 'umkm'}
        <div class="mb-10">
          <h2 class="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2"><span class="w-2 h-2 rounded-full bg-amber-500"></span>Menunggu Persetujuan ({antreanPending.length})</h2>
          {#if antreanPending.length === 0}
            <div class="bg-white p-8 rounded-2xl border border-dashed border-slate-300 text-center text-slate-500 font-medium">Tidak ada antrean lapak baru.</div>
          {:else}
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
              {#each antreanPending as item}
                <div class="bg-white p-6 rounded-2xl shadow-sm border border-amber-200">
                  <h3 class="font-bold text-lg text-slate-800">{item.nama_usaha}</h3>
                  <p class="text-sm text-slate-500 mb-4">{item.kategori} • {item.harga}</p>
                  <div class="flex gap-2">
                    <button onclick={() => accLapak(item.id)} class="flex-1 bg-emerald-600 text-white py-2 rounded-xl font-medium hover:bg-emerald-700 transition">Tayangkan</button>
                    <button onclick={() => hapusLapak(item.id)} class="bg-red-50 text-red-600 px-4 py-2 rounded-xl font-medium hover:bg-red-100 transition">Tolak</button>
                  </div>
                </div>
              {/each}
            </div>
          {/if}
        </div>
        <div>
          <h2 class="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2"><span class="w-2 h-2 rounded-full bg-emerald-500"></span>Lapak Sudah Tayang ({lapakTayang.length})</h2>
          <div class="bg-white rounded-2xl shadow-sm overflow-hidden border border-slate-200">
            <table class="w-full text-left text-sm">
              <thead class="bg-slate-50 border-b text-slate-600">
                <tr><th class="p-4 font-semibold">Nama Usaha</th><th class="p-4 font-semibold">Kategori</th><th class="p-4 font-semibold">WhatsApp</th><th class="p-4 text-center font-semibold w-56">Aksi</th></tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                {#each lapakTayang as item}
                  <tr class="hover:bg-slate-50/50">
                    <td class="p-4 font-bold text-slate-800">{item.nama_usaha}</td>
                    <td class="p-4 text-slate-600">{item.kategori}</td>
                    <td class="p-4 text-slate-600">{item.nomor_wa}</td>
                    <td class="p-4">
                      <div class="flex justify-center gap-2">
                        <button onclick={() => copyLinkWarga(item.id)} class="bg-slate-100 text-slate-700 px-3 py-1.5 rounded-lg text-xs font-semibold hover:bg-slate-200 transition">Copy Link Edit</button>
                        <button onclick={() => hapusLapak(item.id)} class="bg-red-50 text-red-600 px-3 py-1.5 rounded-lg text-xs font-semibold hover:bg-red-100 transition">Hapus</button>
                      </div>
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
        <div class="grid grid-cols-1 xl:grid-cols-3 gap-8">
          <div class="xl:col-span-1 bg-white p-6 rounded-2xl shadow-sm border border-slate-200 self-start">
            <div class="flex justify-between items-center mb-6">
              <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-blue-600"><path d="m8 3 4 8 5-5 5 15H2L8 3z"/></svg>
                {editWisataId ? 'Edit Wisata' : 'Tambah Wisata'}
              </h2>
              {#if editWisataId}<button type="button" onclick={batalEditWisata} class="text-xs font-bold text-slate-500 hover:text-slate-800">Batal Edit</button>{/if}
            </div>
            <form onsubmit={simpanWisata} class="space-y-4">
              <div><label class="block text-sm font-semibold text-slate-700 mb-1">Nama Tempat</label><input type="text" bind:value={formWisata.nama_tempat} required class="w-full border border-slate-300 p-2.5 rounded-xl focus:ring-2 focus:ring-blue-500 outline-none" /></div>
              <div><label class="block text-sm font-semibold text-slate-700 mb-1">Deskripsi Singkat</label><textarea bind:value={formWisata.deskripsi} rows="3" required class="w-full border border-slate-300 p-2.5 rounded-xl focus:ring-2 focus:ring-blue-500 outline-none"></textarea></div>
              <div><label class="block text-sm font-semibold text-slate-700 mb-1">Lokasi / Alamat</label><input type="text" bind:value={formWisata.lokasi} required class="w-full border border-slate-300 p-2.5 rounded-xl focus:ring-2 focus:ring-blue-500 outline-none" /></div>
              <div><label class="block text-sm font-semibold text-slate-700 mb-1">Koordinat Google Maps</label><input type="text" bind:value={formWisata.koordinat} required placeholder="-0.890719, 100.756278" class="w-full border border-slate-300 p-2.5 rounded-xl focus:ring-2 focus:ring-blue-500 outline-none font-mono text-sm" /></div>
              <div><label class="block text-sm font-semibold text-slate-700 mb-1">Panduan Jalur / Rute Akses</label><textarea bind:value={formWisata.jalur_akses} rows="2" class="w-full border border-slate-300 p-2.5 rounded-xl focus:ring-2 focus:ring-blue-500 outline-none"></textarea></div>
              
              <div>
                <label class="block text-sm font-semibold text-slate-700 mb-1">Upload Foto (Maks 4 Foto)</label>
                {#if editWisataId && currentWisataFoto}
                  <div class="mb-2 flex gap-2 overflow-x-auto hide-scrollbar">
                    {#each currentWisataFoto.split(',') as pic}
                      <img src={pic} alt="Current" class="h-16 w-24 rounded-lg border object-cover shrink-0"/>
                    {/each}
                  </div>
                  <p class="text-xs text-amber-600 font-medium mb-1">*Biarkan kosong jika tidak ingin ganti foto.</p>
                {/if}
                <input type="file" accept="image/*" multiple bind:files={fileWisata} required={!editWisataId} class="w-full border border-slate-300 p-2 rounded-xl text-sm file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100" />
                <p class="text-[10px] text-slate-400 mt-1">Tekan CTRL/Tahan Layar untuk memilih lebih dari 1 foto.</p>
              </div>

              <button type="submit" disabled={isSubmitting} class="w-full bg-slate-800 text-white py-3 rounded-xl font-bold hover:bg-slate-700 transition disabled:opacity-50 mt-2">
                {isSubmitting ? 'Menyimpan...' : (editWisataId ? 'Update Data Wisata' : 'Simpan Wisata Baru')}
              </button>
            </form>
          </div>
          <div class="xl:col-span-2 bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
             <table class="w-full text-left text-sm">
              <thead class="bg-slate-50 border-b text-slate-600"><tr><th class="p-4 font-semibold w-24">Foto</th><th class="p-4 font-semibold">Detail Info</th><th class="p-4 text-center font-semibold w-40">Aksi</th></tr></thead>
              <tbody class="divide-y divide-slate-100">
                {#each wisataList as item}
                  <tr class="hover:bg-slate-50/50">
                    <td class="p-4"><img src={item.foto_url.split(',')[0]} alt="wisata" class="w-16 h-16 object-cover rounded-xl shadow-sm border border-slate-100" /></td>
                    <td class="p-4">
                      <p class="font-bold text-slate-800 text-base">{item.nama_tempat}</p>
                      <p class="text-slate-500 mb-1">{item.lokasi}</p>
                      <span class="inline-block bg-slate-100 text-slate-600 text-xs px-2 py-1 rounded font-mono border border-slate-200">Lat: {item.latitude} | Lng: {item.longitude}</span>
                    </td>
                    <td class="p-4">
                      <div class="flex justify-center gap-2">
                        <button onclick={() => setEditWisata(item)} class="bg-blue-50 text-blue-600 px-3 py-1.5 rounded-lg text-xs font-bold hover:bg-blue-100 transition">Edit</button>
                        <button onclick={() => hapusWisata(item.id)} class="bg-red-50 text-red-600 px-3 py-1.5 rounded-lg text-xs font-bold hover:bg-red-100 transition">Hapus</button>
                      </div>
                    </td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        </div>
      {/if}

      <!-- ================= TAB ARTIKEL ================= -->
      {#if activeTab === 'artikel'}
         <div class="grid grid-cols-1 xl:grid-cols-3 gap-8">
          <div class="xl:col-span-1 bg-white p-6 rounded-2xl shadow-sm border border-slate-200 self-start">
            <div class="flex justify-between items-center mb-6">
              <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-purple-600"><path d="M4 22h16a2 2 0 0 0 2-2V4a2 2 0 0 0-2-2H8a2 2 0 0 0-2 2v16a2 2 0 0 1-2 2Zm0 0a2 2 0 0 1-2-2v-9c0-1.1.9-2 2-2h2"/><path d="M18 14h-8"/><path d="M15 18h-5"/><path d="M10 6h8v4h-8V6Z"/></svg>
                {editArtikelId ? 'Edit Berita' : 'Tulis Berita'}
              </h2>
              {#if editArtikelId}<button type="button" onclick={batalEditArtikel} class="text-xs font-bold text-slate-500 hover:text-slate-800">Batal Edit</button>{/if}
            </div>

            <form onsubmit={simpanArtikel} class="space-y-4">
              <div><label class="block text-sm font-semibold text-slate-700 mb-1">Judul Artikel</label><input type="text" bind:value={formArtikel.judul} required class="w-full border border-slate-300 p-2.5 rounded-xl focus:ring-2 focus:ring-purple-500 outline-none" /></div>
              <div><label class="block text-sm font-semibold text-slate-700 mb-1">Penulis</label><input type="text" bind:value={formArtikel.penulis} required class="w-full border border-slate-300 p-2.5 rounded-xl focus:ring-2 focus:ring-purple-500 outline-none" /></div>
              <div><label class="block text-sm font-semibold text-slate-700 mb-1">Konten Berita</label><textarea bind:value={formArtikel.konten} rows="6" required class="w-full border border-slate-300 p-2.5 rounded-xl focus:ring-2 focus:ring-purple-500 outline-none"></textarea></div>
              <div>
                <label class="block text-sm font-semibold text-slate-700 mb-1">Thumbnail Foto</label>
                {#if editArtikelId && currentArtikelFoto}
                  <div class="mb-2"><img src={currentArtikelFoto} alt="Current" class="h-20 rounded-lg border object-cover"/></div>
                  <p class="text-xs text-amber-600 font-medium mb-1">*Biarkan kosong jika tidak ingin ganti foto.</p>
                {/if}
                <input type="file" accept="image/*" bind:files={fileArtikel} required={!editArtikelId} class="w-full border border-slate-300 p-2 rounded-xl text-sm file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-purple-50 file:text-purple-700 hover:file:bg-purple-100" />
              </div>
              <button type="submit" disabled={isSubmitting} class="w-full bg-slate-800 text-white py-3 rounded-xl font-bold hover:bg-slate-700 transition disabled:opacity-50 mt-2">
                {isSubmitting ? 'Mengupload...' : (editArtikelId ? 'Update Berita' : 'Terbitkan Berita')}
              </button>
            </form>
          </div>

          <div class="xl:col-span-2 bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
             <table class="w-full text-left text-sm">
              <thead class="bg-slate-50 border-b text-slate-600">
                <tr><th class="p-4 font-semibold w-24">Thumbnail</th><th class="p-4 font-semibold">Judul & Penulis</th><th class="p-4 text-center font-semibold w-56">Aksi</th></tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                {#each artikelList as item}
                  <tr class="hover:bg-slate-50/50">
                    <td class="p-4"><img src={item.foto_url} alt="artikel" class="w-20 h-14 object-cover rounded-lg shadow-sm border border-slate-100" /></td>
                    <td class="p-4">
                      <p class="font-bold text-slate-800 text-base mb-1 leading-snug">{item.judul}</p>
                      <p class="text-xs text-slate-500 font-semibold uppercase tracking-wider">Oleh: {item.penulis}</p>
                    </td>
                    <td class="p-4">
                      <div class="flex justify-center gap-2">
                        <button onclick={() => bukaGaleri(item)} class="bg-emerald-50 text-emerald-600 px-3 py-1.5 rounded-lg text-xs font-bold hover:bg-emerald-100 transition">Galeri</button>
                        <button onclick={() => setEditArtikel(item)} class="bg-blue-50 text-blue-600 px-3 py-1.5 rounded-lg text-xs font-bold hover:bg-blue-100 transition">Edit</button>
                        <button onclick={() => hapusArtikel(item.id)} class="bg-red-50 text-red-600 px-3 py-1.5 rounded-lg text-xs font-bold hover:bg-red-100 transition">Hapus</button>
                      </div>
                    </td>
                  </tr>
                {:else}
                  <tr><td colspan="3" class="p-8 text-center text-slate-500">Belum ada berita yang diterbitkan.</td></tr>
                {/each}
              </tbody>
            </table>
          </div>
        </div>
      {/if}

    {/if}
  </div>

  <!-- ================= MODAL KELOLA GALERI (POP-UP) ================= -->
  {#if isGaleriOpen && selectedArtikel}
    <div class="fixed inset-0 z-[60] flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm">
      <div class="bg-white w-full max-w-3xl max-h-[90vh] rounded-3xl shadow-2xl flex flex-col overflow-hidden animate-in fade-in zoom-in-95 duration-200">
        
        <div class="px-6 py-4 border-b border-slate-100 flex justify-between items-center bg-slate-50">
          <div>
            <h3 class="text-lg font-bold text-slate-800">Kelola Galeri Dokumentasi</h3>
            <p class="text-xs text-slate-500 mt-1 line-clamp-1">{selectedArtikel.judul}</p>
          </div>
          <button onclick={tutupGaleri} class="p-2 text-slate-400 hover:text-red-500 hover:bg-red-50 rounded-xl transition">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
          </button>
        </div>

        <div class="p-6 overflow-y-auto flex-1 bg-white custom-scrollbar">
          <div class="mb-8 p-6 bg-slate-50 border-2 border-dashed border-slate-200 rounded-2xl text-center relative overflow-hidden group">
            <input type="file" accept="image/*" onchange={tambahFotoGaleri} disabled={isUploadingGaleri} class="absolute inset-0 w-full h-full opacity-0 cursor-pointer z-10" />
            <div class="flex flex-col items-center justify-center text-slate-500 group-hover:text-emerald-600 transition">
              {#if isUploadingGaleri}
                <svg class="animate-spin h-8 w-8 text-emerald-600 mb-2" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
                <span class="font-bold text-emerald-600">Mengupload Foto...</span>
              {:else}
                <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="mb-2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" x2="12" y1="3" y2="15"/></svg>
                <span class="font-bold">Klik disini untuk tambah foto baru</span>
                <span class="text-xs mt-1 text-slate-400">Format JPG/PNG, Maksimal 2 MB</span>
              {/if}
            </div>
          </div>

          <h4 class="font-bold text-slate-800 mb-4 flex items-center gap-2">
            <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-emerald-500"><rect width="18" height="18" x="3" y="3" rx="2" ry="2"/><circle cx="9" cy="9" r="2"/><path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/></svg>
            Koleksi Foto Saat Ini ({galeriList.length})
          </h4>
          
          {#if galeriList.length === 0}
            <div class="text-center py-10 text-slate-400 text-sm border rounded-xl bg-slate-50">Belum ada foto yang diupload.</div>
          {:else}
            <div class="grid grid-cols-2 sm:grid-cols-3 gap-4">
              {#each galeriList as foto}
                <div class="relative group rounded-xl overflow-hidden bg-slate-100 aspect-video shadow-sm border border-slate-200">
                  <img src={foto.foto_url} alt="Galeri" class="w-full h-full object-cover" />
                  <div class="absolute inset-0 bg-black/50 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center">
                    <button onclick={() => hapusFotoGaleri(foto.id)} class="bg-red-500 text-white p-2 rounded-full hover:bg-red-600 transition hover:scale-110 shadow-lg">
                      <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 6h18"/><path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/><path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/></svg>
                    </button>
                  </div>
                </div>
              {/each}
            </div>
          {/if}
        </div>
      </div>
    </div>
  {/if}

</div>

<style>
  .hide-scrollbar::-webkit-scrollbar {
    display: none;
  }
  .hide-scrollbar {
    -ms-overflow-style: none;  /* IE and Edge */
    scrollbar-width: none;  /* Firefox */
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