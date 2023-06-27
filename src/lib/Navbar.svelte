<script lang="ts">
  let mobileView = false;
  import { hintEnabled } from "../stores";
</script>

<nav
  class="navbar is-fixed-top is-dark"
  role="navigation"
  aria-label="main navigation"
>
  <div class="navbar-brand">
    <button
      role="button"
      class="navbar-burger burger"
      class:is-active={mobileView}
      aria-label="menu"
      aria-expanded="false"
      data-target="navbarBasicExample"
    >
      <span aria-hidden="true" />
      <span aria-hidden="true" />
      <span aria-hidden="true" />
    </button>
  </div>

  <div id="navbarBasicExample" class="navbar-menu" class:is-active={mobileView}>
    <div class="navbar-start">
      <!-- svelte-ignore a11y-missing-attribute -->
      <a
        class="navbar-item"
        rel="noopener"
        on:click={() => {
          hintEnabled.update((n) => !n);
        }}
      >
        <span class="icon is-medium">
          <i class="fas fa-question-circle" />
        </span>
        <span>{$hintEnabled ? "Hide Hint" : "Ask Don"}</span>
      </a>
    </div>
  </div>

  <div id="navbarBasicExample" class="navbar-menu" class:is-active={mobileView}>
    <div class="navbar-end">
      <a
        class="navbar-item"
        href="https://github.com/lineville/break-even"
        target="_blank"
        rel="noopener"
      >
        <span class="icon is-medium">
          <i class="fab fa-github" />
        </span>
        <span>Code</span>
      </a>
    </div>
  </div>
</nav>

<style>
  button {
    border: none;
  }

  .navbar {
    opacity: 90%;
  }
</style>
