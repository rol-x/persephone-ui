<script>
  import ConnectionsGame from '$lib/components/ConnectionsGame.svelte';
</script>

<div class="page-container">
  <h1 class="page-title">🧩 Połączenia </h1>
  <ConnectionsGame />

  <a href="/connections/add" class="propose-button">
    ➕ Zaproponuj kategorie
  </a>
</div>

<style>
.page-title {
  margin-bottom: 1rem;
  font-family: sans-serif;
  font-size: 2rem;
  font-weight: bold;
  color: #333;
}

.page-container {
  max-width: 75vw;
  margin: 0 auto;
}

.propose-button {
  margin-top: 2rem;
  display: inline-block;
  font-size: 1.1rem;
  text-decoration: none;
  color: #333;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 0.6rem 0.8rem;
  background: #f3f3f3;
  transition: background 0.2s;
}

@media (max-width: 600px) {
  .page-container {
    max-width: 90vw;
    padding: 0;
    margin: 0 auto;
  }
}

@media (max-width: 480px) {
  .propose-button {
    font-size: 0.9rem;
  }
}

@media (max-width: 340px) {
  .propose-button {
    font-size: 0.75rem;
  }
}
</style>