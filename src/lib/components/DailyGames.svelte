<script>
  const games = [
    {
      name: "Połączenia",
      url: "/connections",
      description: "Connections, ale po polsku"
    },
    {
      name: "Connections",
      url: "https://www.nytimes.com/games/connections",
      description: "Grupuj powiązane słowa"
    },
    {
      name: "Wordle",
      url: "https://www.nytimes.com/games/wordle",
      description: "Zgadnij 5-literowe słowo dnia"
    },
    {
      name: "Bandle",
      url: "https://bandle.app",
      description: "Odgadnij piosenkę po instrumentach"
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
    },
    {
      name: "Movie to Movie",
      url: "https://movietomovie.com/",
      description: "Znajdź połączenie z jednego filmu do drugiego"
    },
    {
      name: "Raddle",
      url: "https://raddle.quest/",
      description: "Wstaw słowa w podpowiedzi"
    },
    {
      name: "Wiki Game",
      url: "https://www.thewikigame.com/",
      description: "Połącz ze sobą artykuły z Wikipedii"
    },
    {
      name: "Dordle",
      url: "https://zaratustra.itch.io/dordle",
      description: "Podwójne Wordle"
    },
    {
      name: "Waffle",
      url: "https://wafflegame.net/daily",
      description: "Wordle-krzyżówka w kształcie wafla"
    },
    {
      name: "Woodle",
      url: "https://play.woodle.today/",
      description: "Wordle bez wskazywania konkretnych liter"
    },
    {
      name: "Betweenle",
      url: "https://betweenle.com/",
      description: "Odgadnij wyraz alfabetycznie"
    },
    {
      name: "Chronophoto",
      url: "https://www.chronophoto.app/game.html",
      description: "Podaj rok wykonania zdjęcia"
    },
    {
      name: "TimeGuessr",
      url: "https://timeguessr.com/",
      description: "Zgadnij czas i miejsce wykonania zdjęcia"
    },
    {
      name: "xdle",
      url: "https://zaratustra.itch.io/xdle",
      description: "Odgadnij liczbę x"
    },
    {
      name: "Stacked",
      url: "https://stacked.clevergoat.com/",
      description: "Połącz wyrazy w grupy różnych wielkości"
    },
    {
      name: "23 Words",
      url: "https://wordnerd.co/23words/",
      description: "Odshuffluj litery w 23 wyrazach"
    },
    {
      name: "Disorderly",
      url: "https://playdisorderly.com/",
      description: "Uporządkuj w odpowiedniej kolejności"
    },
    {
      name: "Travle",
      url: "https://travle.earth/?travle-lang=en",
      description: "Połącz dwa kraje na mapie"
    },
    {
      name: "ChessGuessr",
      url: "https://www.chessguessr.com/",
      description: "Podaj najlepszą sekwencję ruchów"
    },
    {
      name: "Word Grid",
      url: "https://wordgrid.clevergoat.com/",
      description: "Uzupełnij pola wyrazami"
    },
    {
      name: "Bracket City",
      url: "https://www.theatlantic.com/games/bracket-city/",
      description: "Odgaduj hasła od środka"
    },
    {
      name: "Contexto",
      url: "https://contexto.me/en/",
      description: "Odgadnij tajemniczy wyraz"
    },
    {
      name: "Semantle",
      url: "https://semantle.com/",
      description: "Odganij tajemniczy wyraz... znowu"
    },
    {
      name: "Nerdle",
      url: "https://nerdlegame.com/",
      description: "Odganij równanie dnia"
    },
    {
      name: "Fill-in-the-blank Mini",
      url: "https://www.nytimes.com/crosswords/game/special/fill-in-the-blank",
      description: "Krzyżówka mini z brakującymi słowami"
    },
    {
      name: "Cinematrix",
      url: "https://www.vulture.com/article/daily-movie-grid-trivia-game-cinematrix.html",
      description: "Uzupełnij pola filmami"
    },
    {
      name: "Movie Grid",
      url: "https://moviegrid.io/",
      description: "Uzupełnij pola filmami... znowu"
    },
    {
      name: "Box Office Game",
      url: "https://boxofficega.me/",
      description: "Odgadnij filmy z box office"
    },
    {
      name: "FoodGuessr",
      url: "https://www.foodguessr.com/",
      description: "Odgadnij kraj pochodzenia potrawy"
    },
    {
      name: "Spellcheck",
      url: "https://spellcheck.xyz/solo_game",
      description: "Czy potrafisz literować?"
    }
  ];
</script>

<section>
  <h2>🎮 Gierki</h2>
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
