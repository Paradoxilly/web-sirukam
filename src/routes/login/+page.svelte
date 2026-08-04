<script>
  import { supabase } from '$lib/supabase';
  import { goto } from '$app/navigation';

  let email = $state('');
  let password = $state('');
  let loading = $state(false);

  async function handleLogin(e) {
    e.preventDefault();
    loading = true;

    // Login normal murni pakai Email & Password
    const { error } = await supabase.auth.signInWithPassword({
      email: email,
      password: password,
    });

    if (error) {
      alert('Gagal login: Pastikan Email dan Password benar.');
    } else {
      goto('/admin');
    }
    loading = false;
  }
</script>

<div class="min-h-screen bg-slate-50 flex items-center justify-center p-4">
  <div class="bg-white p-8 rounded-2xl shadow-sm border border-gray-100 max-w-md w-full">
    <h1 class="text-2xl font-bold text-slate-800 mb-2">Login Admin Nagari</h1>
    <p class="text-slate-500 mb-6 text-sm">Halaman khusus perangkat desa untuk mengelola data website.</p>

    <form onsubmit={handleLogin} class="space-y-4">
      <div>
        <label class="block text-sm font-semibold text-slate-600 mb-1">Email Admin</label>
        <input type="email" bind:value={email} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-slate-800 focus:outline-none" placeholder="admin@sirukam.com" />
      </div>
      <div>
        <label class="block text-sm font-semibold text-slate-600 mb-1">Password</label>
        <input type="password" bind:value={password} required class="w-full border border-gray-300 rounded-lg p-2.5 focus:ring-2 focus:ring-slate-800 focus:outline-none" placeholder="••••••••" />
      </div>

      <button type="submit" disabled={loading} class="w-full bg-slate-800 text-white py-2.5 rounded-lg font-medium hover:bg-slate-900 transition shadow mt-4">
        {loading ? 'Mengecek Akses...' : 'Masuk Dashboard'}
      </button>
    </form>
  </div>
</div>