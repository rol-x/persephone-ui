<script>
  import { browser } from '$app/environment';
  import { onDestroy } from 'svelte';
  import ConnectionStatus from './ConnectionStatus.svelte';
  import { createEventDispatcher } from 'svelte';
  
  const dispatch = createEventDispatcher();

  let socket;
  let serverTime = null;
  let connected = false;
  let error = null;

  const host = 'api.persephone.run';

  function connect() {
    if (!browser) return;

    socket = new WebSocket(`ws://${host}/ws/time/unix`);

    socket.onopen = () => {
      connected = true;
    };

    socket.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        serverTime = data.unix;
        dispatch('time', serverTime);
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
      <ConnectionStatus connected={connected} label="" />
      Czas UNIX 🕒 <div style="font-size: 1.2em">{serverTime}</div>
    </div>

  {#if error}
    <p style="color: red;">⚠️ {error}</p>
  {/if}
</div>
