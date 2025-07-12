<script>
  import { browser } from '$app/environment';
  import { onDestroy } from 'svelte';
  import ConnectionStatus from './ConnectionStatus.svelte';

  let socket;
  let datetime = '';
  let connected = false;
  let error = null;

  const wsUrl = import.meta.env.VITE_API_WS_URL;

  function connect() {
    if (!browser) return;

    socket = new WebSocket(`${wsUrl}/time/datetime`);

    socket.onopen = () => {
      connected = true;
    };

    socket.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        datetime = data.datetime;
      } catch (e) {
        error = 'Błąd formatu wiadomości';
      }
    };

    socket.onerror = () => {
      error = 'Błąd WebSocket';
    };

    socket.onclose = () => {
      connected = false;
    };
  }

  connect();

  onDestroy(() => {
    socket?.close();
  });
</script>

<div style="padding: 0.5rem;">
  <div style="display: flex; align-items: center; gap: 0.5rem;">
    <ConnectionStatus connected={connected} />
    Data i czas 🕓 <div style="font-size: 1.2em;">{datetime}</div>
  </div>

  {#if error}
    <p style="color: red;">⚠️ {error}</p>
  {/if}
</div>
