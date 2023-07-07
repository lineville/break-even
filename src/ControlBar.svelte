<script lang="ts">
  import { fly } from "svelte/transition";

  export let insuranceOpen;
  export let insuranceBet;
  export let bet;
  export let acceptInsurance;
  export let denyInsurance;
  export let wonInsurance;
  export let betOnInsurance;
  export let isSplit;
  export let leftHand;
  export let rightHand;
  export let stayLeft;
  export let stayRight;
  export let leftHandDone;
  export let rightHandDone;
  export let userCards;
  export let stay;
  export let lockedIn;
  export let canSplit;
  export let isBusted;
  export let hit;
  export let split;
  export let leftHandBust;
  export let rightHandBust;
  export let bust;
  export let doubleDown;
</script>

<div class="is-centered box is-dark" id="controlBar">
  {#if insuranceOpen}
    <div transition:fly={{ x: -1000, duration: 500 }}>
      <h3>Insurance ?</h3>
      <span class="control has-icons-left">
        <input
          class="input is-info"
          type="number"
          id="insuranceBet"
          name="insuranceBet"
          bind:value={insuranceBet}
          max={Math.floor(bet / 2)}
          min={1}
        />

        <span class="icon is-small is-left" id="insuranceDollarSign">
          <i class="fa fa-dollar-sign" />
        </span>
      </span>

      <button
        class="button is-success is-outlined insurance-button"
        on:click={acceptInsurance}
      >
        <span class="icon is-small">
          <i class="fas fa-angle-double-left" />
        </span>
        <span class="icon is-small">
          <i class="fas fa-check" />
        </span>
      </button>

      <button
        class="button is-danger is-outlined insurance-button"
        on:click={denyInsurance}
      >
        <span class="icon is-small">
          <i class="fas fa-times" />
        </span>
        <span class="icon is-small">
          <i class="fas fa-angle-double-right" />
        </span>
      </button>

      {#if wonInsurance}
        <span
          class="icon is-small"
          in:fly={{ y: -1000, duration: 500 }}
          out:fly={{ y: -1000, duration: 500, delay: 800 }}
        >
          <i class="fas fa-coins" />
        </span>

        <span
          class={`tag is-light is-success is-medium`}
          id="infoTag"
          transition:fly={{ x: 1000, duration: 500 }}
        >
          You won ${insuranceBet} from the insurance side bet even though you lost
          the hand.
        </span>
      {/if}

      {#if betOnInsurance && !wonInsurance}
        <span
          class={`tag is-light is-danger is-medium`}
          id="infoTag"
          transition:fly={{ x: 1000, duration: 500, delay: 500 }}
        >
          You lost ${insuranceBet} from the insurance side bet. Don't worry you've
          still got a chance!
        </span>
      {/if}
    </div>
  {:else}
    <div transition:fly={{ x: 2000, duration: 500, delay: 200 }}>
      {#if isSplit}
        <div class="split-controls columns is-mobile mb-2 mt-2">
          <div class="column">
            <button
              class="button is-danger is-outlined"
              on:click={stayLeft}
              disabled={leftHandDone || isBusted(leftHand)}
            >
              <span class="icon is-small">
                <i class="fas fa-chevron-left" />
              </span>
              <span>Stay</span>
              <span class="icon is-small">
                <i class="fas fa-hand-paper" />
              </span>
            </button>

            <button
              class="button is-success is-outlined"
              on:click={() =>
                doubleDown(
                  leftHand,
                  () => hit(leftHand, leftHandBust, "Left"),
                  stayLeft
                )}
              disabled={leftHandDone || leftHand.length > 2}
            >
              <span class="icon is-small">
                <i class="fas fa-chevron-down" />
              </span>
              <span>Double</span>
              <span class="icon is-small">
                <i class="fas fa-coins" />
              </span>
            </button>

            <button
              class="button is-info is-outlined"
              on:click={() => hit(leftHand, leftHandBust, "Left")}
              disabled={leftHandDone || isBusted(leftHand)}
            >
              <span class="icon is-small">
                <i class="fas fa-hand-holding-medical" />
              </span>
              <span>Hit</span>
              <span class="icon is-small">
                <i class="fas fa-chevron-right" />
              </span>
            </button>
          </div>

          <span class="ml-6 mr-6" />

          <div class="column">
            <button
              class="button is-danger is-outlined"
              on:click={stayRight}
              disabled={!(leftHandDone || isBusted(leftHand)) ||
                rightHandDone ||
                isBusted(rightHand)}
            >
              <span class="icon is-small">
                <i class="fas fa-chevron-left" />
              </span>
              <span>Stay</span>
              <span class="icon is-small">
                <i class="fas fa-hand-paper" />
              </span>
            </button>

            <button
              class="button is-success is-outlined"
              on:click={() =>
                doubleDown(
                  rightHand,
                  () => hit(rightHand, rightHandBust, "Right"),
                  stayRight
                )}
              disabled={!leftHandDone || rightHand.length > 2 || rightHandDone}
            >
              <span class="icon is-small">
                <i class="fas fa-chevron-down" />
              </span>
              <span>Double</span>
              <span class="icon is-small">
                <i class="fas fa-coins" />
              </span>
            </button>

            <button
              class="button is-info is-outlined"
              on:click={() => hit(rightHand, rightHandBust, "Right")}
              disabled={!(
                leftHandDone ||
                isBusted(leftHand) ||
                isBusted(rightHand)
              ) || rightHandDone}
            >
              <span class="icon is-small">
                <i class="fas fa-hand-holding-medical" />
              </span>
              <span>Hit</span>
              <span class="icon is-small">
                <i class="fas fa-chevron-right" />
              </span>
            </button>
          </div>
        </div>
      {:else}
        <div class="columns is-mobile mb-2 mt-2">
          <div class="column">
            <button
              class="button is-danger is-outlined"
              on:click={stay}
              disabled={lockedIn}
            >
              <span class="icon is-small">
                <i class="fas fa-chevron-left" />
              </span>
              <span>Stay</span>
              <span class="icon is-small">
                <i class="fas fa-hand-paper" />
              </span>
            </button>
          </div>

          {#if canSplit && !isSplit}
            <div class="column">
              <button
                class="button is-warning is-outlined"
                id="splitButton"
                on:click={split}
                transition:fly={{ y: -1000, duration: 500 }}
              >
                <span class="icon is-small">
                  <i class="fas fa-chevron-up" />
                </span>
                <span>Split</span>
                <span class="icon is-small">
                  <i class="fas fa-expand-alt" />
                </span>
              </button>
            </div>
          {/if}

          <div class="column">
            <button
              class="button is-success is-outlined"
              on:click={() =>
                doubleDown(
                  userCards,
                  () => hit(userCards, bust, "User"),
                  stay,
                  "User"
                )}
              disabled={lockedIn || userCards.length !== 2}
            >
              <span class="icon is-small">
                <i class="fas fa-chevron-down" />
              </span>
              <span>Double</span>
              <span class="icon is-small">
                <i class="fas fa-coins" />
              </span>
            </button>
          </div>

          <div class="column">
            <button
              class="button is-info is-outlined"
              on:click={() => hit(userCards, bust, "User")}
              disabled={lockedIn}
            >
              <span class="icon is-small">
                <i class="fas fa-hand-holding-medical" />
              </span>
              <span>Hit</span>
              <span class="icon is-small">
                <i class="fas fa-chevron-right" />
              </span>
            </button>
          </div>
        </div>
      {/if}
    </div>
  {/if}
</div>

<style>
  #insuranceBet {
    width: 100px;
    margin-top: 5px;
  }

  #controlBar {
    margin-left: 15vw;
    margin-right: 15vw;
  }

  #insuranceDollarSign {
    margin-top: 8px;
  }

  @media screen and (max-device-width: 768px) {
    #controlBar {
      margin-left: 2vw;
      margin-right: 2vw;
    }
    .column {
      padding: 0;
    }

    button:not(.insurance-button) > .icon {
      display: none;
    }

    .split-controls button {
      min-width: 85px;
    }

    button {
      min-width: 65px;
    }
  }
</style>
