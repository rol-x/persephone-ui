<script>
  import ConnectionsGame from '$lib/components/ConnectionsGame.svelte';
</script>

<div class="page-container">
<h1 class="page-title">🧩 Połączenia </h1>
<ConnectionsGame />

<a href="/connections/add" style="
  margin-top: 2rem;
  display: inline-block;
  font-size: 1.1rem;
  text-decoration: none;
  color: #333;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 0.4rem 0.8rem;
  background: #f3f3f3;
  transition: background 0.2s;
">
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
</style>