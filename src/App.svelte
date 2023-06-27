<script lang="ts">
  import 'bulma/bulma.sass'
  import BlackJack from './lib/BlackJack.svelte'
  import Navbar from './lib/Navbar.svelte';

  
  // import { invoke } from '@tauri-apps/api';
  // import { onMount } from 'svelte';

  // let message = "";

  // onMount(async () => {
  //   message = await greet();
  // })
  
  // // Example of passing message from Rust to JS
  // async function greet(): Promise<string> {
  //   const message : string = await invoke('greet', { name: 'Lets play' });
  //   console.log(message);
  //   return message;
  // }
</script>

<main>
  <Navbar />
  <div style="margin-top: 40px;"></div>
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