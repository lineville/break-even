<script lang="ts">
  const links = ['Stats', 'Timeline']
  const emojis = ['🌐', '📊']
  let mobileView = false
</script>

<style>
  button {
    border: none;
  }

  .navbar {
    opacity: 90%;
  }
</style>

<nav class="navbar is-fixed-top is-dark" role="navigation" aria-label="main navigation">
  <div class="navbar-brand">

    <button
      role="button"
      class="navbar-burger burger"
      class:is-active={mobileView}
      aria-label="menu"
      aria-expanded="false"
      data-target="navbarBasicExample"
      on:click={() => {
        mobileView = !mobileView
      }}>
      <span aria-hidden="true" />
      <span aria-hidden="true" />
      <span aria-hidden="true" />
    </button>
  </div>

  <div id="navbarBasicExample" class="navbar-menu" class:is-active={mobileView}>
    <!-- <div class="navbar-start">
      {#each links as link, idx}
        <a class="navbar-item" href="/{link}" rel="noopener">{emojis[idx]} {link}</a>
      {/each}

    </div> -->

    <div class="navbar-end">

      <!-- <div class="navbar-item has-dropdown is-hoverable">
        <a class="navbar-link" href="/">🕹️ Time Wasters</a>

        <div class="navbar-dropdown">
          <a class="navbar-item" href="/colorfill" rel="noopener">🌈 Color Fill</a>
          <a class="navbar-item" href="/candychomper" rel="noopener">🍬 Candy Chomper</a>
          <a class="navbar-item" href="/profitcalculator" rel="noopener">📉 Profit Calculator</a>
          <a class="navbar-item" href="/blackjack" rel="noopener">🃏 BlackJack</a>
        </div>
      </div> -->

      <!-- <div class="navbar-item has-dropdown is-hoverable">
        <a class="navbar-link" href="/">⚙️ Gizmos</a>

        <div class="navbar-dropdown">
          <a class="navbar-item" href="/counter" rel="noopener">➕ Counter</a>
          <a class="navbar-item" href="/squarerootbind" rel="noopener">🟧 Square Root</a>
          <a class="navbar-item" href="/speedtyper" rel="noopener">💻 Speed Typer</a>
        </div>
      </div> -->

      <a class="navbar-item" href="https://medium.com/@lineville" target="_blank" rel="noopener">
        <span class="icon is-medium">
          <i class="fab fa-medium" />
        </span>
        <span>Blog</span>
      </a>
      <hr class="navbar-divider" />
      <a class="navbar-item" href="https://github.com/lineville/svelte-ts" target="_blank" rel="noopener">
        <span class="icon is-medium">
          <i class="fab fa-github" />
        </span>
        <span>GitHub</span>
      </a>

      <a class="navbar-item" href="https://portfolio-6eyf.onrender.com/" target="_blank" rel="noopener">
        <img
          alt="avatar"
          src="https://avataaars.io/?avatarStyle=Transparent&topType=ShortHairShortWaved&accessoriesType=Prescription02&hairColor=Blonde&facialHairType=BeardLight&facialHairColor=Blonde&clotheType=Hoodie&clotheColor=Heather&eyeType=Default&eyebrowType=RaisedExcitedNatural&mouthType=Smile&skinColor=Light" />
        &nbsp Me
      </a>

      <div class="navbar-item tags has-addons">
        <span class="tag">v 1.2.0</span>
        <span class="tag is-primary">Beta</span>
      </div>
    </div>
  </div>
</nav>
