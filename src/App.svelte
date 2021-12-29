<script lang="ts">
  import 'bulma/bulma.sass'
  import BlackJack from './lib/BlackJack.svelte'
  import Navbar from './lib/Navbar.svelte';
</script>

<main>
  <Navbar />
 <BlackJack />
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
  }
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>