<script lang="ts">
  import { fly } from "svelte/transition";

  export let insuranceOpen;
  export let insuranceBet;
  export let bet;
  export let acceptInsurance;
  export let denyInsurance;
  export let wonInsurance;
  export let betOnInsurance;
  export let split;
  export let handleSplitHand;
  export let leftHand;
  export let rightHand;
  export let stayLeft;
  export let stayRight;
  export let leftHandDone;
  export let rightHandDone;
  export let handleStay;
  export let handleDoubleDown;
  export let handleHit;
  export let userCards;
  export let stay;
  export let lockedIn;
  export let handleBust;
  export let handleLeftHandBust;
  export let handleRightHandBust;
  export let canSplit;
  export let isBusted;
  export let hit;
</script>

<div class="is-centered box is-dark" id="controlBar">
  {#if insuranceOpen}
    <div class="field">
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
          class="button is-success is-outlined"
          on:click={acceptInsurance}
        >
          <span class="icon is-small">
            <i class="fas fa-angle-double-left" />
          </span>
          <span class="icon is-small">
            <i class="fas fa-check" />
          </span>
        </button>

        <button class="button is-danger is-outlined" on:click={denyInsurance}>
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
    </div>
  {:else}
    <div class="field" transition:fly={{ x: 2000, duration: 500, delay: 200 }}>
      <div>
        {#if split}
          <div class="split-controls columns mb-2 mt-2">
            <button
              class="button is-danger is-outlined column"
              on:click={() => handleStay(leftHand, stayLeft)}
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
              class="button is-success is-outlined column"
              on:click={() =>
                handleDoubleDown(
                  leftHand,
                  () => hit(leftHand, handleLeftHandBust, "Left"),
                  stayLeft,
                  "Left"
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
              class="button is-info is-outlined column"
              on:click={() =>
                handleHit(
                  leftHand,
                  () => hit(leftHand, handleLeftHandBust, "Left"),
                  "Left"
                )}
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

            <span class="ml-6 mr-6" />

            <button
              class="button is-danger is-outlined column"
              on:click={() => handleStay(rightHand, stayRight)}
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
              class="button is-success is-outlined column"
              on:click={() =>
                handleDoubleDown(
                  rightHand,
                  () => hit(rightHand, handleRightHandBust, "Right"),
                  stayRight,
                  "Right"
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
              class="button is-info is-outlined column"
              on:click={() =>
                handleHit(
                  rightHand,
                  () => hit(rightHand, handleRightHandBust, "Right"),
                  "Right"
                )}
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
        {:else}
          <div class="columns mb-2 mt-2">
            <button
              class="button is-danger is-outlined column"
              on:click={() => handleStay(userCards, stay)}
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

            {#if canSplit && !split}
              <button
                class="button is-warning is-outlined column"
                id="splitButton"
                on:click={handleSplitHand}
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
            {/if}

            <button
              class="button is-success is-outlined column"
              on:click={() =>
                handleDoubleDown(
                  userCards,
                  () => hit(userCards, handleBust, "User"),
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

            <button
              class="button is-info is-outlined column"
              on:click={() =>
                handleHit(
                  userCards,
                  () => hit(userCards, handleBust, "User"),
                  "User"
                )}
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
        {/if}
      </div>
    </div>
  {/if}
</div>

<style>
  #insuranceBet {
    width: 100px;
    margin-top: 5px;
  }

  #controlBar {
    margin-left: 10vw;
    margin-right: 10vw;
    margin-top: 3vh;
    margin-bottom: 3vh;
  }

  #insuranceDollarSign {
    margin-top: 8px;
  }
</style>
