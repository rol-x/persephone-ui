<script>
  import { onMount } from 'svelte';
  import confetti from 'canvas-confetti';

  let gameId = null;
  let words = [];
  let authors = [];
  let guessedGroups = [];
  let selected = new Set();
  let shakeWords = new Set();
  let error = null;
  let isComplete = false;
  let solvedNumber = null;

  const httpUrl = import.meta.env.VITE_API_URL;
  const getUrl = `${httpUrl}/connections/daily`;

  onMount(async () => {
    try {
      const res = await fetch(getUrl);
      const data = await res.json();
      gameId = data.gameId;
      words = shuffle(Array.from(data.words));
      authors = data.authors;
    } catch (e) {
      console.log(e);
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
    return guessedGroups.some(g => g.words.includes(word));
  }

  async function submit() {
    if (selected.size !== 4 || !gameId) return;

    const postUrl = `${httpUrl}/connections/${gameId}/check`;

    const payload = {
      words: Array.from(selected)
    };

    try {
      const res = await fetch(postUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      if (!res.ok) {
        error = 'Błąd połączenia z serwerem.';
        return;
      }

      const data = await res.json();

      if (data.correct) {
        guessedGroups = [
          ...guessedGroups,
          {
            words: payload.words,
            explanation: data.explanation,
            color: data.color
          }
        ];
        selected.clear();

        if (guessedGroups.length === 4) {
          isComplete = true;
          launchConfetti();
          await updateSolvedCount();
        }
      } else {
        shakeWords = new Set(Array.from(selected));
        setTimeout(() => {
          shakeWords = new Set();
        }, 500);
      }

    } catch (err) {
      error = 'Błąd podczas zgłoszenia grupy.';
    }
  }

  function launchConfetti() {
    confetti({
      particleCount: 100,
      spread: 70,
      origin: { y: 0.6 }
    });
  }

  async function updateSolvedCount() {
    try {
      const res = await fetch(`${httpUrl}/connections/${gameId}/solved`, {
        method: 'POST'
      });

      if (res.ok) {
        const count = await res.json();
        solvedNumber = count;
      }
    } catch (e) {
      console.warn("Nie udało się zaktualizować wyniku.");
    }
  }
</script>

<div class="container">
  {#if error}
    <p style="color: red;">⚠️ {error}</p>
  {:else if gameId}
    <div class="container" style="display: flex; flex-direction: column; gap: 1rem;">
      {#if authors.length}
        <p style="font-size: 0.8em; color: #777;">
          Autorzy: {authors.join(', ')}
        </p>
      {/if}

      {#each guessedGroups as group}
        <div
          style="
            flex: 1 0 100%;
            padding: 1rem;
            text-align: center;
            background-color: var(--group-color-{group.color});
            border-radius: 6px;
            font-weight: bold;
            color: black;
            display: flex;
            flex-direction: column;
            align-items: center;
            text-transform: uppercase;
            gap: 0.5rem;
          ">
          <div style="font-size: 1rem;">
            {group.explanation}
          </div>
          <div style="font-size: 0.85rem; font-weight: normal;">
            {group.words.join(', ')}
          </div>
        </div>
      {/each}

      <div
        style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 0.5rem; margin-top: 1rem;">
        {#each words as word}
          {#if !isSolved(word)}
            <button
              class:shake={shakeWords.has(word)}
              on:click={() => toggle(word)}
              style={`
                height: 60px;
                font-family: sans-serif;
                text-transform: uppercase;
                font-weight: bold;
                font-size: 1rem;
                border: none;
                border-radius: 8px;
                cursor: pointer;
                background: ${selected.has(word) ? '#4a4a4a' : '#f3f3f3'};
                color: ${selected.has(word) ? 'white' : 'black'};
                transition: background 0.2s, color 0.2s;
              `}>
              {word}
            </button>
          {/if}
        {/each}
      </div>

      <button on:click={submit}
        disabled={selected.size !== 4}
        style="
          margin-top: 1rem;
          padding: 0.6rem 1.2rem;
          font-family: sans-serif;
          text-transform: uppercase;
          font-weight: bold;
          background: black;
          color: white;
          border: none;
          border-radius: 8px;
          cursor: pointer;">
        Zatwierdź
      </button>

      {#if isComplete}
        <div style="margin-top: 2rem; padding: 1rem; border: 2px dashed #4caf50; border-radius: 12px; background: #eaffea;">
          <h2 style="color: #2e7d32;">🎉 Gratulacje!</h2>
          <p>Rozwiązałeś wszystkie grupy!</p>
          {#if solvedNumber}
            <p>Jesteś <strong>{solvedNumber}.</strong> osobą, która tego dokonała 💪</p>
          {/if}
        </div>
      {/if}
    </div>
  {:else}
    <p>⏳ Ładowanie gry...</p>
  {/if}
</div>

<style>
  .container {
    max-width: 720px;
    margin: 0 auto;
    padding: 1rem;
    font-family: sans-serif;
  }

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
