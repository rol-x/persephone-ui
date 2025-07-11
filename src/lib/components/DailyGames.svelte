<script>
  const games = [
    {
      name: "Wordle",
      url: "https://www.nytimes.com/games/wordle",
      description: "Zgadnij 5-literowe słowo dnia"
    },
    {
      name: "Connections",
      url: "https://www.nytimes.com/games/connections",
      description: "Grupuj powiązane słowa"
    },
    {
      name: "Bandle",
      url: "https://bandle.app",
      description: "Wordle o bohaterach LoL"
    },
    {
      name: "Globle",
      url: "https://globle-game.com",
      description: "Zgadnij kraj na mapie świata"
    },
    {
      name: "Framed",
      url: "https://framed.wtf",
      description: "Zgadnij film na podstawie kadru"
    }
  ];
</script>

<section>
  <h2>🎮 Zmierz się </h2>
  <ul style="list-style: none; padding-left: 0;">
    {#each games as game}
      <li style="margin-bottom: 0.75rem;">
        <a href={game.url} target="_blank" rel="noopener" style="text-decoration: none; color: inherit;">
          <div style="
            padding: 0.6rem 0.8rem;
            border-radius: 0.5rem;
            border: 1px solid #ccc;
            transition: background 0.2s;
          " on:mouseover={() => event.currentTarget.style.background = '#f9f9f9'}
            on:mouseout={() => event.currentTarget.style.background = 'transparent'}>
            <strong>{game.name}</strong><br>
            <span style="font-size: 0.9em; color: #555;">{game.description}</span>
          </div>
        </a>
      </li>
    {/each}
  </ul>
</section>
