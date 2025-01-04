<script lang="ts">
  import { onMount } from "svelte";
  import { fly } from "svelte/transition";
  import CardList from "./CardList.svelte";
  import { decideMove, computeScore, cardCount } from "./BasicStrategy";
  import BalanceBars from "./BalanceBars.svelte";
  import ControlBar from "./ControlBar.svelte";
  import ConfirmNewGameModal from "./ConfirmNewGameModal.svelte";

  const localBucket = window.localStorage;

  // -------------Types ------------------
  type Suite = "❤️" | "♦" | "♣️" | "♠️";
  type HandType = "User" | "Dealer" | "Left" | "Right";
  type Card = {
    name: string;
    value: number;
    optionalValue: number | null;
    suite: Suite;
  };

  // --------BlackJack Actions -----------

  const hit = (
    hand: Array<Card>,
    bustFunc: () => void,
    who: HandType
  ): void => {
    hand = [...hand, drawCard()]; // Add the new card to the hand

    // Update the hand of who ever hit
    switch (who) {
      case "User":
        userCards = hand;
        break;
      case "Dealer":
        dealerCards = hand;
        break;
      case "Left":
        leftHand = hand;
        break;
      case "Right":
        rightHand = hand;
        break;
    }

    // Check if the hand is bust and call the bustFunc callback, otherwise update the hint
    if (isBusted(hand)) {
      bustFunc();
    }
  };

  const stay = (): void => {
    lockedIn = true; // Lock the user in, no more hits at this point
    dealerCards = playOutDealerHand(); // Let the user play out the hand

    if (
      isBusted(dealerCards) ||
      computeScore(userCards) > computeScore(dealerCards) ||
      computeScore(userCards) === 21
    ) {
      // If dealer busts, user has better score, or user has 21, user wins
      userWon = true;
      push = false;
      balance += bet * 2;
    } else if (computeScore(userCards) === computeScore(dealerCards)) {
      // Push
      userWon = false;
      push = true;
      balance += bet;
    }
  };

  const doubleDown = (
    hand: Array<Card>,
    hitFunc: (hand: Array<Card>, bustFunc: () => void, who: HandType) => void,
    stayFunction: () => void
  ): void => {
    balance -= bet; // Bet again
    bet *= 2; // Double the bet

    let bustFunc = isSplit
      ? leftHandDone
        ? leftHandBust
        : rightHandBust
      : bust;
    hitFunc(hand, bustFunc, "User");
    if (!isBusted(userCards)) {
      stayFunction();
    }
    bet /= 2; // Reset the bet
  };

  const split = (): void => {
    leftHand = [userCards[0], drawCard()];
    rightHand = [userCards[1], drawCard()];
    userCards = [];
    isSplit = true;
    balance -= bet;
    if (leftHand[0].name === "Ace") {
      leftHandDone = true;
      rightHandDone = true;
      stayRight();
    }
  };

  const stayLeft = (): void => {
    leftHandDone = true;
  };

  const stayRight = (): void => {
    rightHandDone = true;
    lockedIn = true;
    dealerCards = playOutDealerHand();

    let leftWon = false;
    let leftPush = false;
    let rightWon = false;
    let rightPush = false;
    // * Check left hand
    if (
      !isBusted(leftHand) &&
      (isBusted(dealerCards) ||
        computeScore(leftHand) > computeScore(dealerCards) ||
        computeScore(leftHand) === 21)
    ) {
      balance += bet * 2;
      leftWon = true;
    } else if (computeScore(leftHand) === computeScore(dealerCards)) {
      balance += bet;
      leftPush = false;
    }

    // * Check right hand
    if (
      isBusted(dealerCards) ||
      computeScore(rightHand) > computeScore(dealerCards) ||
      computeScore(rightHand) === 21
    ) {
      balance += bet * 2;
      rightWon = true;
    } else if (computeScore(rightHand) === computeScore(dealerCards)) {
      balance += bet;
      rightPush = true;
    }

    if (
      (leftWon && rightWon) ||
      (leftWon && rightPush) ||
      (leftPush && rightWon)
    ) {
      userWon = true;
      push = false;
    } else if ((leftWon && !rightWon) || (!leftWon && rightWon)) {
      userWon = false;
      push = true;
    }
  };

  const nextHand = (): void => {
    balance -= bet;
    lockedIn = false;
    isSplit = false;
    userWon = false;
    push = false;
    wonInsurance = false;
    betOnInsurance = false;
    leftHandDone = false;
    rightHandDone = false;
    userCards = [drawCard(), drawCard()];
    dealerCards = [drawCard(), drawCard()];

    if (deck.length < 15) {
      deck = shuffle(newDeck());
      deckCount = 0;
    }

    // * Write to local storage
    localBucket.setItem("Balance", balance.toString());
  };

  const acceptInsurance = (): void => {
    lockedIn = true;
    betOnInsurance = true;

    if (computeScore(dealerCards) === 21) {
      wonInsurance = true;
      userWon = false;
      push = false;
    } else {
      wonInsurance = false;
      balance -= insuranceBet;
      lockedIn = false;
      setTimeout(() => {
        insuranceOpen = false;
      }, 2000);
    }
  };

  const denyInsurance = (): void => {
    if (computeScore(dealerCards) === 21) {
      wonInsurance = false;
      userWon = false;
      push = false;
      stay();
    }

    insuranceOpen = false;
  };

  // ------- Utils ------------

  const newDeck = (): Array<Card> => {
    let result = new Array<Card>();
    const suites: Array<Suite> = ["❤️", "♦", "♣️", "♠️"];
    for (let i = 2; i <= 14; i++) {
      suites.forEach((suite) => {
        result.push(indexToCard(i, suite));
      });
    }

    return result;
  };

  const drawCard = (): Card => {
    const card = deck[0];
    deck = deck.slice(1);
    deckCount += cardCount(card);
    return card;
  };

  const shuffle = (deck: Array<Card>): Array<Card> => {
    return deck.sort(() => Math.random() - 0.5);
  };

  const indexToCard = (idx: number, suite: Suite): Card => {
    switch (idx) {
      case 2:
        return { name: "Two", value: idx, optionalValue: null, suite: suite };
      case 3:
        return { name: "Three", value: idx, optionalValue: null, suite: suite };
      case 4:
        return { name: "Four", value: idx, optionalValue: null, suite: suite };
      case 5:
        return { name: "Five", value: idx, optionalValue: null, suite: suite };
      case 6:
        return { name: "Six", value: idx, optionalValue: null, suite: suite };
      case 7:
        return { name: "Seven", value: idx, optionalValue: null, suite: suite };
      case 8:
        return { name: "Eight", value: idx, optionalValue: null, suite: suite };
      case 9:
        return { name: "Nine", value: idx, optionalValue: null, suite: suite };
      case 10:
        return { name: "Ten", value: idx, optionalValue: null, suite: suite };
      case 11:
        return { name: "Jack", value: 10, optionalValue: null, suite: suite };
      case 12:
        return { name: "Queen", value: 10, optionalValue: null, suite: suite };
      case 13:
        return { name: "King", value: 10, optionalValue: null, suite: suite };
      case 14:
        return { name: "Ace", value: 11, optionalValue: 1, suite: suite };
      default:
        throw new Error("Bad card value");
    }
  };

  const cardToImage = (card: Card): string => {
    return card.value < 10 || card.name === "Ten"
      ? card.value + suitePrefix(card.suite)
      : card.name[0] + suitePrefix(card.suite);
  };

  const suitePrefix = (suite: Suite): string => {
    switch (suite) {
      case "❤️":
        return "H";
      case "♠️":
        return "S";
      case "♣️":
        return "C";
      case "♦":
        return "D";
      default:
        throw new Error("Bad suite");
    }
  };

  const isBusted = (cards: Array<Card>): boolean => {
    return computeScore(cards) > 21;
  };

  const playOutDealerHand = (): Array<Card> => {
    while (!isBusted(dealerCards) && computeScore(dealerCards) < 17) {
      const newCard = drawCard();
      dealerCards = [...dealerCards, newCard];
      deckCount += cardCount(newCard);
    }

    return dealerCards;
  };

  const bust = (): void => {
    lockedIn = true;
    userWon = false;
    push = false;
  };

  const leftHandBust = (): void => {
    leftHandDone = true;
  };

  const rightHandBust = (): void => {
    rightHandDone = true;
    lockedIn = true;

    if (
      !isBusted(leftHand) &&
      (isBusted(dealerCards) ||
        computeScore(leftHand) > computeScore(dealerCards) ||
        computeScore(leftHand) === 21)
    ) {
      userWon = false;
      push = true;
    } else if (computeScore(leftHand) === computeScore(dealerCards)) {
      userWon = false;
      push = false;
    }
  };

  const handleKeydown = (event: any) => {
    switch (event.key) {
      case "Enter":
        if (lockedIn) {
          nextHand();
        }
        break;
      case "ArrowLeft":
        if (insuranceOpen) {
          acceptInsurance();
          return;
        }
        if (!lockedIn && !isSplit) {
          stay();
        }
        if (isSplit && !leftHandDone) {
          stayLeft();
        } else if (isSplit && leftHandDone && !rightHandDone) {
          stayRight();
        }
        break;
      case "ArrowRight":
        if (insuranceOpen) {
          denyInsurance();
          return;
        }
        if (!lockedIn && !isSplit) {
          hit(userCards, bust, "User");
        }
        if (isSplit && !leftHandDone) {
          hit(leftHand, leftHandBust, "Left");
        } else if (isSplit && leftHandDone && !rightHandDone) {
          hit(rightHand, rightHandBust, "Right");
        }
        break;
      case "ArrowUp":
        if (canSplit && !isSplit) {
          split();
        }
        break;
      case "ArrowDown":
        if (!lockedIn && userCards.length === 2 && !isSplit) {
          doubleDown(userCards, () => hit(userCards, bust, "User"), stay);
        }

        if (isSplit && !leftHandDone) {
          doubleDown(
            leftHand,
            () => hit(leftHand, leftHandBust, "Left"),
            stayLeft
          );
        } else if (isSplit && leftHandDone && !rightHandDone) {
          doubleDown(
            rightHand,
            () => hit(rightHand, rightHandBust, "Right"),
            stayRight
          );
        }
        break;
      case "h":
        hintEnabled = !hintEnabled;
        break;
      case "r":
        newGameState();
        break;
      default:
        break;
    }
  };

  const donsHint = (
    userCards: Array<Card>,
    dealerUpCard: Card,
    insuranceOpen: boolean,
    splitHand: boolean,
    leftHand: Array<Card>,
    leftHandDone: boolean,
    rightHand: Array<Card>
  ): string => {
    if (insuranceOpen) {
      hintColor = "is-danger";
      return "I never take insurance, but that's just me 😉";
    }

    let decision =
      splitHand && leftHandDone
        ? decideMove(rightHand, dealerUpCard, !isSplit)
        : splitHand
          ? decideMove(leftHand, dealerUpCard, !isSplit)
          : decideMove(userCards, dealerUpCard, !isSplit);
    switch (decision) {
      case "Stay":
        hintColor = "is-danger";
        return "Better to play it safe on this one and stay";
      case "Hit":
        hintColor = "is-primary";
        return "The smart thing to do here is to hit!";
      case "DoubleDown":
        hintColor = "is-success";
        return "Things are looking good I'd double down on this one 💰!";
      case "Split":
        hintColor = "is-warning";
        return "Splitting looks like your best option!";
    }
  };

  const newGameState = () => {
    balance = 100;
    bet = 10;
    deckCount = 0;
    lockedIn = false;
    isSplit = false;
    userWon = false;
    push = false;
    leftHandDone = false;
    rightHandDone = false;
    hintColor = "is-info";
    deck = shuffle(newDeck());
    dealerCards = [drawCard(), drawCard()];
    userCards = [drawCard(), drawCard()];
    leftHand = [];
    rightHand = [];
    wonInsurance = false;
    betOnInsurance = false;

    setTimeout(() => {
      document
        .getElementById("confirm-new-game-modal")
        ?.classList.remove("is-active");
    }, 200);
  };

  const toggleHintEnabled = () => {
    hintEnabled = !hintEnabled;
  };

  // ----------- State -----------

  let balance = 100;
  let bet = 10;
  let deckCount = 0;
  let lockedIn = false;
  let isSplit = false;
  let userWon = false;
  let push = false;
  let leftHandDone = false;
  let rightHandDone = false;
  let hintColor = "is-info";
  let deck = shuffle(newDeck());
  let dealerCards: Array<Card> = [drawCard(), drawCard()];
  let userCards: Array<Card> = [drawCard(), drawCard()];
  let leftHand: Array<Card> = [];
  let rightHand: Array<Card> = [];
  let wonInsurance = false;
  let betOnInsurance = false;
  let hintEnabled = true;

  // ------------ Derived state ------------

  $: insuranceBet = Math.floor(bet / 2);
  $: insuranceOpen =
    dealerCards[0].name === "Ace" &&
    computeScore(userCards) !== 21 &&
    !lockedIn &&
    userCards.length === 2;
  $: canSplit =
    userCards[0]?.name === userCards[1]?.name &&
    !isSplit &&
    userCards.length === 2;
  $: hint = donsHint(
    userCards,
    dealerCards[0],
    insuranceOpen,
    isSplit,
    leftHand,
    leftHandDone,
    rightHand
  );

  // Screen width
  let innerWidth = 0;
  $: isTouch = innerWidth <= 768;

  // Load in balance from localStorage on component mounting
  onMount(() => {
    const storedBalance = localBucket.getItem("Balance");
    if (storedBalance === null) {
      localBucket.setItem("Balance", balance.toString());
    } else {
      balance = parseInt(storedBalance || "");
    }
  });
</script>

<svelte:window on:keydown={handleKeydown} bind:innerWidth />

<div class="columns is-mobile is-centered" id="blackJackContainer">
  <!-- Main central section -->
  <div class="column is-full">
    <!-- Dealer Cards + User Cards -->
    <div class="is-centered" id="cards-section">
      <div>
        <!-- Dealer cards -->
        <CardList
          cards={dealerCards.map((c) => cardToImage(c))}
          visible={lockedIn}
          {isSplit}
          isDealer={true}
        />

        <div id="divider" />

        <!-- Player cards -->
        {#if isSplit}
          <ul>
            <CardList cards={leftHand.map((c) => cardToImage(c))} {isSplit} />
            <span style="display:inline-block; width: 100px;" />
            <CardList cards={rightHand.map((c) => cardToImage(c))} {isSplit} />
          </ul>
        {:else}
          <CardList cards={userCards.map((c) => cardToImage(c))} {isSplit} />
        {/if}
      </div>
    </div>
    <div class={`${isTouch ? "pinned-to-bottom" : ""}`}>
      <!-- Notification Fly-In -->
      {#if lockedIn}
        <div
          class={`notification is-narrow box ${
            userWon ? "is-success" : push ? "is-info" : "is-danger"
          }`}
          transition:fly={{ x: -1000, duration: 500, delay: 200 }}
        >
          <span
            class={`tag is-medium ${
              userWon ? "is-success" : push ? "is-info" : "is-danger"
            }`}
            id="wonOrLost"
          >
            <h1>{userWon ? "Win 🎉" : push ? "Push 🙃" : "Loss 😢"}</h1>
          </span>

          <span class="control has-icons-left">
            <input
              class="input is-info"
              type="number"
              id="bet"
              name="bet"
              bind:value={bet}
              disabled={!lockedIn}
              min={1}
              max={balance}
            />
            <span class="icon is-small is-left">
              <i class="fa fa-dollar-sign" />
            </span>
          </span>
          <button class="button is-info" on:click={nextHand}>
            {#if !isTouch}
              <span>Next Hand</span>
            {/if}
            <span class="icon is-small">
              <i class="fas fa-angle-double-right" />
            </span>
          </button>
        </div>
      {/if}

      <!-- Hint Fly-in -->
      {#if hintEnabled}
        <div class="is-centered">
          <div style="display: flex; justify-content: center;">
            <figure class="image is-32x32 mr-4 mb-8">
              <img
                class="is-rounded is-square"
                alt="Donald J. Neville"
                src="./images/donald_neville_headshot.jpeg"
              />
            </figure>
            <span
              class={`tag is-light mt-4 ${hintColor} ${
                isTouch ? "is-small" : "is-medium"
              }`}
            >
              {hint}
              <button
                class={`delete ${isTouch ? "is-small" : "is-medium"} ml-2`}
                on:click={toggleHintEnabled}
              ></button>
            </span>
          </div>
        </div>
      {/if}

      <!-- Balance Bars -->
      <BalanceBars {balance} {lockedIn} {toggleHintEnabled} />

      <!-- Control Bar -->
      <ControlBar
        {insuranceOpen}
        {insuranceBet}
        {bet}
        {acceptInsurance}
        {denyInsurance}
        {wonInsurance}
        {betOnInsurance}
        {isSplit}
        {leftHand}
        {rightHand}
        {stayLeft}
        {stayRight}
        {stay}
        {leftHandDone}
        {rightHandDone}
        {userCards}
        {lockedIn}
        {canSplit}
        {isBusted}
        {hit}
        {doubleDown}
        {leftHandBust}
        {rightHandBust}
        {bust}
        {split}
      />
    </div>
  </div>
</div>

<ConfirmNewGameModal {newGameState} />

<style>
  :disabled {
    pointer-events: none;
  }

  h1 {
    font-size: x-large;
    margin-top: 10px;
  }

  @media screen and (max-device-width: 768px) {
    h1 {
      font-size: large;
    }
  }

  .tag {
    margin-top: 5px;
    margin-right: 1.5rem;
    margin-bottom: 0.5rem !important;
    font-size: large;
  }

  @media screen and (max-device-width: 768px) {
    .tag {
      font-size: small;
    }
  }

  .fa-dollar-sign {
    margin-bottom: 25px;
  }

  #bet {
    width: 100px;
    margin-top: 5px;
  }

  #cards-section {
    margin-top: 3vh;
    margin-bottom: 4vh;
  }

  .notification {
    margin-left: 15vw;
    margin-right: 15vw;
    margin-top: 10vh;
    margin-bottom: 2vh;
  }

  @media screen and (max-device-width: 768px) {
    .notification {
      margin-left: 2vw;
      margin-right: 2vw;
      margin-top: -45px;
      margin-bottom: 2vw;
    }
  }

  #divider {
    margin-top: 5vh;
    margin-bottom: 5vh;
  }

  .narrow-gap {
    width: 125px !important;
  }

  .pinned-to-bottom {
    position: fixed;
    bottom: 2vh;
    width: 94%;
    margin: auto;
  }
</style>
