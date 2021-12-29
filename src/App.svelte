<script lang="ts">
  import 'bulma/bulma.sass'
  import BlackJack from './lib/BlackJack.svelte'
  import Navbar from './lib/Navbar.svelte';
</script>

<main>
  <Navbar />
  <br />
  <BlackJack />
</main>

<style>
  br {
    padding-top: 1rem;
  }
  
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