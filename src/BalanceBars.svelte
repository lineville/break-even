<script lang="ts">
  export let balance;
  export let lockedIn = false;

  $: progressBarValues = balances(balance);

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
  <span class="mb-6" id="balance-label">$ {balance}</span>
  <span
    class="icon is-small js-modal-trigger"
    data-target="settings-modal"
    id="corner-refresh"
  >
    <i class="fas fa-gear" />
  </span>

  {#each progressBarValues as b}
    <progress
      class={`progress is-narrow box ${
        balance > 0 ? "is-primary" : "is-warning"
      }`}
      value={b}
      max="100"
    />
  {/each}
</div>

<style>
  .balance-bars {
    margin-left: 20vw;
    margin-right: 20vw;
    margin-top: 4vh;
  }

  @media screen and (max-device-width: 768px) {
    .balance-bars {
      margin-left: 2vw;
      margin-right: 2vw;
      margin-top: 4vh;
    }
  }

  .no-gap {
    margin-top: 0;
  }

  #balance-label {
    font-size: x-large;
  }

  #corner-refresh {
    float: right;
  }
</style>
