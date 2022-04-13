<script lang="ts">
  let mobileView = false;
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
      on:click={() => {
        mobileView = !mobileView;
      }}
    >
      <span aria-hidden="true" />
      <span aria-hidden="true" />
      <span aria-hidden="true" />
    </button>
  </div>

  <div id="navbarBasicExample" class="navbar-menu" class:is-active={mobileView}>
    <div class="navbar-end">
      <a
        class="navbar-item"
        href="https://medium.com/@lineville"
        target="_blank"
        rel="noopener"
      >
        <span class="icon is-medium">
          <i class="fab fa-medium" />
        </span>
        <span>Blog</span>
      </a>
      <hr class="navbar-divider" />
      <a
        class="navbar-item"
        href="https://github.com/lineville/blackjack"
        target="_blank"
        rel="noopener"
      >
        <span class="icon is-medium">
          <i class="fab fa-github" />
        </span>
        <span>GitHub</span>
      </a>

      <a
        class="navbar-item"
        href="https://liamneville.me"
        target="_blank"
        rel="noopener"
      >
        <img
          alt="avatar"
          src="https://avataaars.io/?avatarStyle=Transparent&topType=ShortHairShortWaved&accessoriesType=Prescription02&hairColor=Blonde&facialHairType=BeardLight&facialHairColor=Blonde&clotheType=Hoodie&clotheColor=Heather&eyeType=Default&eyebrowType=RaisedExcitedNatural&mouthType=Smile&skinColor=Light"
        />
        &nbsp Me
      </a>

      <div class="navbar-item tags has-addons">
        <span class="tag">v 1.2.0</span>
        <span class="tag is-primary">Beta</span>
      </div>
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
