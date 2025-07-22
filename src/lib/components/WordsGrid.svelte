<script lang="ts">
  import AutoFontSize from './AutoFontSize.svelte';

  export let words: string[] = [];
  export let selected: Set<string> = new Set();
  export let shakeWords: Set<string> = new Set();
  export let isSolved: (word: string) => boolean;
  export let toggle: (word: string) => void;
</script>

<div class="grid">
  {#each words as word}
    {#if !isSolved(word)}
      <button
        class="tile"
        class:selected={selected.has(word)}
        class:shake={shakeWords.has(word)}
        on:click={() => toggle(word)}
      >
        <AutoFontSize text={word} />
      </button>
    {/if}
  {/each}
</div>

<style>
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

  @media (max-width: 600px) {
    .tile {
      height: auto;
      aspect-ratio: 1 / 1;
    }
  }

  @keyframes shake {
    0% { transform: translateX(0); }
    20% { transform: translateX(-4px); }
    40% { transform: translateX(4px); }
    60% { transform: translateX(-3px); }
    80% { transform: translateX(3px); }
    100% { transform: translateX(0); }
  }

  .tile.shake {
    animation: shake 0.4s ease;
  }
</style>
