<script>
  let serverTime = null;
  let loading = false;
  let error = null;

  async function fetchTime() {
    loading = true;
    error = null;

    try {
      const res = await fetch('https://api.persephone.run/time/unix');
      if (!res.ok) throw new Error(`Error ${res.status}`);

      const data = await res.json();
      serverTime = data.unix || data.time || JSON.stringify(data);
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }
</script>

<h1>Current Server Time</h1>
<button on:click={fetchTime} disabled={loading}>
  {loading ? 'Loading...' : 'Get Server Time'}
</button>

{#if error}
  <p style="color:red;">⚠️ {error}</p>
{:else if serverTime}
  <p>🕒 Unix Time: {serverTime}</p>
{/if}