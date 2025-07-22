<script>
  import { onMount } from 'svelte';

  let username = '';
  let password = '';
  let showAuth = true;
  let authorized = false;
  let hideContent = true;

  let error = null;
  let groupedPropositions = {};

  const httpUrl = import.meta.env.VITE_API_URL;

  async function authenticate() {
    authorized = false;
    try {
      const res = await fetch(`${httpUrl}/propositions`, {
        headers: {
          'Authorization': 'Basic ' + btoa(`${username}:${password}`)
        }
      });

      if (!res.ok) throw new Error('Niepoprawne dane logowania lub błąd serwera');

      const data = await res.json();

      groupedPropositions = data.reduce((acc, item) => {
        if (!acc[item.proposedGameId]) {
          acc[item.proposedGameId] = [];
        }
        acc[item.proposedGameId].push(item);
        return acc;
      }, {});

      authorized = true;
      showAuth = false;
    } catch (e) {
      error = e.message;
    }
  }
</script>

{#if showAuth}
  <div class="modal">
    <div class="modal-box">
      <h2>🔐 Logowanie</h2>
      <input placeholder="Użytkownik" bind:value={username} />
      <input placeholder="Hasło" type="password" bind:value={password} />
      <button on:click={authenticate}>Zaloguj</button>
      {#if error}<p class="error">{error}</p>{/if}
    </div>
  </div>
{:else}
  <div class="panel">
    <div class="toggle-bar">
      <label class="switch">
        <input type="checkbox" bind:checked={hideContent} />
        <span class="slider"></span>
      </label>
      <span class="toggle-label">Ukryj słowa</span>
    </div>

    <h1>📋 Proponowane grupy</h1>
    {#each Object.entries(groupedPropositions) as [groupId, groups]}
        <div class="group-box">
            <h2>{groupId}</h2>
            {#each groups as g}
            <div class="group">
                <div><strong>Słowa:</strong> {hideContent ? '******' : g.words}</div>
                <div><strong>Wyjaśnienie:</strong> {hideContent ? '******' : g.explanation}</div>
                <div><strong>Autor:</strong> {g.author}</div>
                <div><strong>Kolor:</strong> {g.color}</div>
            </div>
            {/each}
            {#if groups.length === 4}
            <a class="play-button" href={`/connections/admin/play/${groupId}`}>
                ▶️ Zagraj
            </a>
            {/if}
        </div>
        {/each}
  </div>
{/if}

<style>
  .modal {
    position: fixed;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(0,0,0,0.5);
  }

  .modal-box {
    background: white;
    padding: 2rem;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    width: 300px;
  }

  .modal-box input {
    padding: 0.5rem;
    font-size: 1rem;
  }

  .modal-box button {
    padding: 0.6rem;
    font-weight: bold;
    background: black;
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
  }

  .error {
    color: red;
    font-size: 0.9rem;
  }

  .panel {
    max-width: 800px;
    margin: 2rem auto;
    padding: 1rem;
    font-family: sans-serif;
  }

  .toggle-bar {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }

  .toggle-label {
    font-weight: bold;
    font-size: 0.95rem;
  }

  .group-box {
    border: 1px solid #ccc;
    border-radius: 8px;
    padding: 1rem;
    margin-bottom: 1.5rem;
    background: #f9f9f9;
  }

  .group {
    background: white;
    padding: 0.5rem;
    border-radius: 6px;
    margin-top: 0.5rem;
    font-size: 0.95rem;
    box-shadow: 0 1px 2px rgba(0,0,0,0.05);
  }

  /* Switch styling */
  .switch {
    position: relative;
    display: inline-block;
    width: 42px;
    height: 22px;
  }

  .switch input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  .slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #ccc;
    transition: 0.3s;
    border-radius: 34px;
  }

  .slider:before {
    position: absolute;
    content: "";
    height: 16px;
    width: 16px;
    left: 3px;
    bottom: 3px;
    background-color: white;
    transition: 0.3s;
    border-radius: 50%;
  }

  .switch input:checked + .slider {
    background-color: #2196F3;
  }

  .switch input:checked + .slider:before {
    transform: translateX(20px);
  }

  .play-button {
    margin-top: 0.8rem;
    display: inline-block;
    background: #2b6cb0;
    color: white;
    padding: 0.5rem 1rem;
    border-radius: 6px;
    text-decoration: none;
    font-weight: bold;
  }

  .play-button:hover {
    background: #225a9e;
  }
  
</style>
