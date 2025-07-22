<script>
  import AutoFontSize from './AutoFontSize.svelte'
  import { onMount } from 'svelte';
  import confetti from 'canvas-confetti';

  let gameId = null;
  let words = [];
  let authors = [];
  let gameNumber = 0;
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
      gameNumber = data.gameNumber;
    } catch (e) {
      console.log(e);
      if (e.message === 'Failed to fetch') error = 'Błąd połączenia z serwerem.';
      else error = 'Błąd ładowania danych gry.';
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
    const payload = { words: Array.from(selected) };

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
    <p class="error">⚠️ {error}</p>
  {:else if gameId}
    <div class="inner">
      {#if authors && gameNumber}
        <p class="authors">No. {gameNumber} - Autorzy: {authors}</p>
      {/if}

      {#each guessedGroups as group}
        <div class="solved-group" style="background-color: var(--group-color-{group.color})">
          <div class="solved-title">{group.explanation}</div>
          <div class="solved-words">{group.words.join(', ')}</div>
        </div>
      {/each}

      <div class="grid">
        {#each words as word}
          {#if !isSolved(word)}
            <button
              class:selected={selected.has(word)}
              class:shake={shakeWords.has(word)}
              class="tile"
              on:click={() => toggle(word)}>
              <AutoFontSize text={word}/>
            </button>
          {/if}
        {/each}
      </div>

      <button class="submit-btn" on:click={submit} disabled={selected.size !== 4}>
        Zatwierdź
      </button>

      {#if isComplete}
        <div class="complete-box">
          <h2>🎉 Gratulacje!</h2>
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
    max-width: 620px;
    margin: 0 auto;
    padding: 1rem;
    font-family: sans-serif;
  }

  .inner {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    flex-wrap: wrap;
  }

  .error {
    color: red;
  }

  .authors {
    font-size: 0.75em;
    color: #777;
    margin-bottom: 5px;
  }

  .solved-group {
    padding: 1rem;
    border-radius: 6px;
    font-weight: bold;
    color: black;
    display: flex;
    flex-direction: column;
    align-items: center;
    text-transform: uppercase;
    gap: 0.5rem;
    height: 40px;
  }

  .solved-title {
    font-size: 1rem;
  }

  .solved-words {
    font-size: 0.85rem;
    font-weight: normal;
  }

  .grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 0.5rem;
  }

  .tile {
    height: 78px;
    min-width: 0;
    font-size: 1.15em;
    font-weight: 700;
    text-transform: uppercase;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    background: #efefe6;
    color: black;
    overflow: hidden;
    transition: background 0.2s, color 0.2s;
    padding: 0;
  }

  .tile.selected {
    background: #444;
    color: white;
  }

  .submit-btn {
    margin-top: 1rem;
    padding: 0.6rem 1.2rem;
    text-transform: uppercase;
    font-weight: bold;
    background: black;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
  }

  .complete-box {
    margin-top: 2rem;
    padding: 1rem;
    border: 2px dashed #4caf50;
    border-radius: 12px;
    background: #eaffea;
    text-align: center;
  }

  @media (max-width: 600px) {
    .container {
      max-width: 95vw;
      padding: 0;
      margin: 0;
    }

    .tile {
      height: auto;
      aspect-ratio: 1 / 1;
    }
  }

  :global(:root) {
    --group-color-purple: #ba81c5;
    --group-color-green: #a0c35a;
    --group-color-blue: #b0c4ef;
    --group-color-yellow: #f9df6d;
    font-family: 'Franklin Gothic';
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
