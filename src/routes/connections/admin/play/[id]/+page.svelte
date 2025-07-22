<script lang="ts">
  import WordsGrid from '$lib/components/WordsGrid.svelte';
  import { onMount } from 'svelte';
  import { page } from '$app/stores';
  import confetti from 'canvas-confetti';

  let proposedGameId: string;
  let allWords: string[] = [];
  let words: string[] = [];
  let authors: string[] = [];

  let guessedGroups = [];
  let allGroups = [];

  let selected = new Set<string>();
  let shakeWords = new Set<string>();
  let error = null;
  let isComplete = false;

  let scheduleDate: string = '';
  let setSuccess = false;
  let setError: string | null = null;

  const httpUrl = import.meta.env.VITE_API_URL;

  $: proposedGameId = $page.params.id;

  $: words = allWords.filter(w => !guessedGroups.some(g => g.words.includes(w)));

  onMount(async () => {
    try {
      const res = await fetch(`${httpUrl}/propositions/${proposedGameId}`);
      const groups = await res.json();
      authors = groups.map(g => g.author).filter((v, i, a) => a.indexOf(v) === i).join(', ');

      allGroups = groups;
      guessedGroups = [];
      allWords = shuffle(groups.flatMap(g => g.words));
    } catch (e) {
      console.error(e);
      error = 'Błąd ładowania gry testowej';
    }
  });

  async function setAsDaily() {
    setError = null;
    setSuccess = false;

    try {
      const res = await fetch(`${httpUrl}/propositions/approve`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          proposedGameId: proposedGameId,
          date: scheduleDate
        })
      });

      if (res.ok) {
        setSuccess = true;
      } else {
        const msg = await res.text();
        setError = msg || 'Błąd ustawiania dnia';
      }
    } catch (e) {
      setError = 'Nie udało się połączyć z serwerem';
    }
  }

  function shuffle(array: string[]) {
    return array
      .map(v => ({ v, s: Math.random() }))
      .sort((a, b) => a.s - b.s)
      .map(({ v }) => v);
  }

  function toggle(word: string) {
    if (isSolved(word)) return;
    if (!selected.has(word) && selected.size >= 4) return;
    selected.has(word) ? selected.delete(word) : selected.add(word);
    selected = new Set(selected);
  }

  function isSolved(word: string) {
    return guessedGroups.some(g => g.words.includes(word));
  }

  function wordsEqual(w1: string[], w2: string[]) {
    return [...w1].sort().join(',') === [...w2].sort().join(',');
  }

  async function submit() {
    if (selected.size !== 4) return;

    const selectedWords = Array.from(selected);

    const match = allGroups.find(
      g => wordsEqual(g.words, selectedWords) && !guessedGroups.some(gg => gg.proposedGroupId === g.proposedGroupId)
    );

    if (match) {
      guessedGroups = [
        ...guessedGroups,
        {
          words: match.words,
          explanation: match.explanation,
          color: match.color,
          proposedGroupId: match.proposedGroupId
        }
      ];
      selected.clear();

      if (guessedGroups.length === 4) {
        isComplete = true;
        launchConfetti();
      }
    } else {
      shakeWords = new Set(selectedWords);
      setTimeout(() => {
        shakeWords.clear();
      }, 500);
    }
  }

  function launchConfetti() {
    confetti({
      particleCount: 100,
      spread: 70,
      origin: { y: 0.6 }
    });
  }
</script>

<div class="container">
  {#if error}
    <p class="error">⚠️ {error}</p>
  {:else if proposedGameId}
    <div class="inner">
      <h2 style="text-align: center">Tryb testowy</h2>

      {#each guessedGroups as group}
        <div class="solved-group" style="background-color: var(--group-color-{group.color})">
          <div class="solved-title">{group.explanation}</div>
          <div class="solved-words">{group.words.join(', ')}</div>
        </div>
      {/each}

      <WordsGrid
        {words}
        {selected}
        {shakeWords}
        {toggle}
        {isSolved}
      />

      <button class="submit-btn" on:click={submit} disabled={selected.size !== 4}>
        Zatwierdź
      </button>

      <div class="author">Autorzy: {authors}</div>

      <div class="schedule-section">
        <label for="scheduleDate">Ustaw na dzień:</label>
        <input id="scheduleDate" type="date" bind:value={scheduleDate} />

        <button on:click={setAsDaily} disabled={!scheduleDate}>
          Publikuj
        </button>

        {#if setSuccess}
          <p class="success-message">✅ Gra została zaplanowana!</p>
        {/if}
        {#if setError}
          <p class="error-message">❌ {setError}</p>
        {/if}
      </div>

      {#if isComplete}
        <div class="complete-box">
          <h2>🎉 Udało się!</h2>
          <p>Rozwiązałeś wszystkie 4 grupy.</p>
        </div>
      {/if}
    </div>
  {:else}
    <p>⏳ Ładowanie...</p>
  {/if}
</div>

<style>
  .container {
    width: 100%;
    max-width: 620px;
    margin: 0 auto;
    padding: 1rem;
    font-family: sans-serif;
    box-sizing: border-box;
  }

  .inner {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .error {
    color: red;
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
    margin-top: 1rem;
  }

  .tile {
    height: 78px;
    font-size: 1.1em;
    font-weight: 700;
    text-transform: uppercase;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    background: #efefe6;
    color: black;
    transition: background 0.2s, color 0.2s;
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
    width: 100%;
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
      width: 100%;
      padding: 0;
    }

    .tile {
      height: auto;
      aspect-ratio: 1 / 1;
    }
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
