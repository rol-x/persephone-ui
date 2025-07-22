<script>
  import { onMount } from 'svelte';

  const httpUrl = import.meta.env.VITE_API_URL;
  const apiUrl = `${httpUrl}/propositions`;

  let author = '';
  let groups = [
    { color: 'yellow', wordsInput: '', explanation: '' },
    { color: 'green', wordsInput: '', explanation: '' },
    { color: 'blue', wordsInput: '', explanation: '' },
    { color: 'purple', wordsInput: '', explanation: '' },
  ];

  let message = '';

  function validateGroup(group) {
    const words = group.wordsInput.split(',').map(w => w.trim()).filter(Boolean);
    return words.length === 4 && group.explanation.trim().length > 0;
  }

  async function submit() {
    const validGroups = groups.filter(validateGroup);

    if (validGroups.length === 0) {
      message = '❌ Podaj przynajmniej jedną poprawną grupę.';
      return;
    }

    const payload = {
      author,
      groups: validGroups.map(g => ({
        words: g.wordsInput.split(',').map(w => w.trim()),
        explanation: g.explanation,
        color: g.color
      }))
    };

    try {
      const res = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      if (res.ok) {
        message = '✅ Wysłano pomyślnie!';
        groups = groups.map(g => ({ ...g, wordsInput: '', explanation: '' }));
        author = '';
      } else {
        message = '❌ Błąd wysyłania.';
      }
    } catch (err) {
      message = '❌ Wystąpił problem z połączeniem.';
    }
  }
</script>

<div class="container">
  <h2>🧠 Zaproponuj dowolną kategorie</h2>

  <div class="propose-groups">
    {#each groups as group}
      <div style="background-color: var(--group-color-{group.color}); padding: 1rem; border-radius: 6px;">
        <label><strong>Wyrazy (po przecinku)</strong></label>
        <input
          type="text"
          bind:value={group.wordsInput}
          placeholder="zszywacz, klawiatura, zdjęcie w ramce, monitor"
          style="width: 97%; margin-bottom: 0.5rem;" />

        <label><strong>Wyjaśnienie</strong></label>
        <input
          type="text"
          bind:value={group.explanation}
          placeholder="Rzeczy trzymane na biurku"
          style="width: 97%;" />
      </div>
    {/each}
  </div>

  <label><strong>Autor</strong></label>
  <input class="author" type="text" bind:value={author} placeholder="Twoje imię lub nick"/>

  <button on:click={submit} style="padding: 0.5rem 1rem;">Wyślij</button>
</div>

{#if message}
  <p style="margin-top: 1rem;">{message}</p>
{/if}

<style>
  :global(:root) {
    --group-color-blue: #d3e9f8;
    --group-color-green: #d8f0da;
    --group-color-yellow: #fdf3cb;
    --group-color-purple: #e6dbf3;
  }

  .container {
    margin: 10px 10px;
  }

  .propose-groups {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 1rem;
    margin: 1.5rem auto;
  }

  .author {
    width: 20%;
    margin-bottom: 1rem;
  }

  input {
    padding: 0.4rem;
    border: 1px solid #999;
    border-radius: 4px;
  }

  label {
    display: block;
    margin-bottom: 0.2rem;
  }

  @media (max-width: 600px) {
    .propose-groups {
      grid-template-columns: 1fr;
    }

    .author {
      width: 60%;
      margin-right: 5px;
    }
  }
</style>
