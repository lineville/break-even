<script lang="ts">
  export let balance;
  export let lockedIn = false;
  export let toggleHintEnabled;

  $: progressBarValues = balances(balance);

  $: isTouch = innerWidth <= 768;

  const balances = (balance) => {
    if (Math.abs(balance) <= 100) {
      return [Math.abs(balance)];
    }
    let result = new Array(Math.ceil(Math.abs(balance / 100))).fill(100, 1);
    result[0] = Math.abs(balance) % 100 === 0 ? 100 : Math.abs(balance) % 100;
    return result;
  };
</script>

<div class={`is-centered box balance-bars is-dark ${lockedIn ? "no-gap" : ""}`}>
  <span>
    <span id="corner-left">
      <a href="https://github.com/lineville/break-even" target="_blank" aria-label="GitHub Repo Link">
        <span class="icon is-small" title="GitHub Repo Link">
          <i class="fa-brands fa-github"/>
        </span>
      </a>
      <span class="icon is-small" on:click={toggleHintEnabled} title="Toggle Hint">
        <i class="fas fa-question-circle" />
      </span>
    </span>
    <span id="balance-label">$ {balance}</span>
    <span id="corner-right">
      <span
        class="icon is-small js-modal-trigger"
        data-target="confirm-new-game-modal"
        title="Reset your balance"
      >
        <i class="fas fa-rotate" />
      </span>
    </span>

    {#each progressBarValues as b}
      <progress
        class="progress"
        class:is-small={isTouch}
        class:is-primary={balance > 0}
        class:is-warning={balance <= 0}
        value={b}
        max="100"
      />
    {/each}
  </span>
</div>

<style>
  .icon:hover {
    cursor: pointer;
  }

  .balance-bars {
    margin-left: 15vw;
    margin-right: 15vw;
    margin-top: 2vh;
    margin-bottom: 2vh;
  }

  @media screen and (max-device-width: 768px) {
    .balance-bars {
      margin-left: 2vw;
      margin-right: 2vw;
      margin-top: 2vh;
    }
  }

  .no-gap {
    margin-top: 0;
  }

  #balance-label {
    margin-bottom: 2vh;
    font-size: x-large;
  }

  @media screen and (max-device-width: 768px) {
    #balance-label {
      font-size: medium;
    }
  }

  #corner-right {
    float: right;
    margin-bottom: 1vh;
  }

  #corner-left {
    float: left;
    margin-bottom: 1vh;
  }

  a {
    text-decoration: none;
    color: inherit;
    background: none;
    border: none;
    padding: 0;
    margin: 0;
    margin-right: 0.5rem;
  }
</style>
