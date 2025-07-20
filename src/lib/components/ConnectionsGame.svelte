<script>
  import { onMount } from 'svelte';

  let gameData = null;
  let allWords = [];
  let guessedGroups = [];
  let selected = new Set();
  let shakeWords = new Set();
  let error = null;

  const httpUrl = import.meta.env.VITE_API_URL;
  const apiUrl = `${httpUrl}/connections/daily`;

  onMount(async () => {
    try {
      const res = await fetch(apiUrl);
      const data = await res.json();
      gameData = data;

      const all = data.groups.flatMap(g => g.wordsList);
      allWords = shuffle(all);
    } catch (e) {
      error = 'Błąd ładowania danych';
    }
  });

  function shuffle(array) {
    return array
      .map(v => ({ v, s: Math.random() }))
      .sort((a, b) => a.s - b.s)
      .map(({ v }) => v);
  }

  function toggle(word) {
    if (isSolved(word)) return;
    if (!selected.has(word) && selected.size >= 4) return;
    selected.has(word) ? selected.delete(word) : selected.add(word);
    selected = new Set(selected);
  }

  function isSolved(word) {
    return guessedGroups.some(g => g.wordsList.includes(word));
  }

  function submit() {
    if (selected.size !== 4) return;

    const current = Array.from(selected).sort();
    const match = gameData.groups.find(g => {
      const sortedGroup = [...g.wordsList].sort();
      return JSON.stringify(sortedGroup) === JSON.stringify(current);
    });

    if (match && !guessedGroups.includes(match)) {
      guessedGroups = [...guessedGroups, match];
      selected.clear();
    } else {
      shakeWords = new Set(selected);
      setTimeout(() => {
        shakeWords.clear();
      }, 500);
    }
  }
</script>

{#if error}
  <p style="color: red;">⚠️ {error}</p>
{:else if gameData}
  <div style="display: flex; flex-direction: column; gap: 1rem;">
    <h3>🧠 Połączenia {gameData.date}</h3>

    <!-- Rozwiązane grupy -->
    {#each guessedGroups as group}
      <div style="margin-bottom: 1rem;">
        <div style="display: flex; gap: 0.5rem; flex-wrap: wrap; margin-bottom: 0.25rem;">
          {#each group.wordsList as word}
            <div
              style="
                flex: 1 0 22%;
                padding: 0.5rem;
                text-align: center;
                background-color: var(--group-color-{group.color});
                border-radius: 4px;
                font-weight: bold;
                color: black;">
              {word}
            </div>
          {/each}
        </div>
        <div style="font-size: 0.9em; font-style: italic; color: #333;">
          {group.explanation}
        </div>
      </div>
    {/each}

    <!-- Pozostałe słowa -->
    <div
      style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 0.5rem; margin-top: 1rem;">
      {#each allWords as word}
        {#if !isSolved(word)}
          <button
            class:shake={shakeWords.has(word)}
            on:click={() => toggle(word)}
            style="
              padding: 0.75rem;
              background: {selected.has(word) ? '#ddd' : '#f9f9f9'};
              border: 1px solid #ccc;
              border-radius: 6px;
              cursor: pointer;
              font-weight: bold;">
            {word}
          </button>
        {/if}
      {/each}
    </div>

    <button on:click={submit}
      disabled={selected.size !== 4}
      style="margin-top: 1rem; padding: 0.5rem 1rem;">
      Zatwierdź
    </button>
  </div>
{:else}
  <p>⏳ Ładowanie gry...</p>
{/if}

<style>
  :global(:root) {
    --group-color-purple: #e6dbf3;
    --group-color-green: #d8f0da;
    --group-color-blue: #d8e9f8;
    --group-color-yellow: #fdf3cb;
  }

  @keyframes shake {
    0% { transform: translateX(0); }
    20% { transform: translateX(-4px); }
    40% { transform: translateX(4px); }
    60% { transform: translateX(-3px); }
    80% { transform: translateX(3px); }
    100% { transform: translateX(0); }
  }

  .shake {
    animation: shake 0.4s ease;
  }
</style>
