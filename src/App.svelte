<script lang="ts">
  import "bulma/bulma.sass";
  import BlackJack from "./BlackJack.svelte";
</script>

<main>
  <BlackJack />
  <footer>
    <div>
      <a
        href="https://github.com/lineville/break-even"
        target="_blank"
        id="github-link"
      >
        <span class="icon is-large">
          <i class="fab fa-github" />
        </span>
      </a>
    </div>
  </footer>
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
  }

  footer {
    position: fixed;
    bottom: 0;
  }

  #github-link {
    color: #333;
    margin-left: 95vw;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
