<script lang="ts">
  // TODO bug in double down doesn't check if you bust
  
  import { onMount } from 'svelte'
  import { fly, slide } from 'svelte/transition'
  import { tweened } from 'svelte/motion'
  import { cubicOut } from 'svelte/easing'
  import CardList from './CardList.svelte'
  import { decideMove, computeScore, cardCount } from '../utils/BasicStrategy'

  const localBucket = window.localStorage

  // -------------Types ------------------
  type Suite = '❤️' | '♦' | '♣️' | '♠️'
  type Decision = 'Hit' | 'Stay' | 'DoubleDown' | 'Split'
  type HandType = 'User' | 'Dealer' | 'Left' | 'Right'
  type Card = { name: string; value: number; optionalValue: number | null; suite: Suite }

  // --------BlackJack Actions -----------

  const hit = (hand: Array<Card>, bustFunc: () => void, who: HandType): void => {
    hand = [...hand, drawCard()]
    switch (who) {
      case 'User':
        userCards = hand
        break
      case 'Dealer':
        dealerCards = hand
        break
      case 'Left':
        leftHand = hand
        break
      case 'Right':
        rightHand = hand
        break
    }
    if (isBusted(hand)) {
      bustFunc()
    } else {
      hint = donsHint(hand, dealerCards[0])
    }
  }

  const stay = (): void => {
    lockedIn = true
    dealerCards = playOutDealerHand()

    if (
      isBusted(dealerCards) ||
      computeScore(userCards) > computeScore(dealerCards) ||
      computeScore(userCards) === 21
    ) {
      userWon = true
      push = false
      balance += bet * 2
    } else if (computeScore(userCards) === computeScore(dealerCards)) {
      userWon = false
      push = true
      balance += bet
    }
  }

  const doubleDown = (
    hand: Array<Card>,
    hitFunc: (hand: Array<Card>, bustFunc: () => void, who: HandType) => void,
    stayFunction: () => void,
  ): void => {
    balance -= bet
    bet *= 2
    let bustFunc = split
      ? leftHandDone
        ? () => {
            leftHandDone = true
          }
        : handleSplitBust
      : handleBust
    hitFunc(hand, bustFunc, 'User')
    if (!isBusted(hand)) {
      stayFunction()
    }
    bet /= 2
  }

  const splitHand = (): void => {
    leftHand = [userCards[0], drawCard()]
    rightHand = [userCards[1], drawCard()]
    userCards = []
    split = true
    balance -= bet
    if (leftHand[0].name === 'Ace') {
      leftHandDone = true
      rightHandDone = true
      stayRight()
    } else {
      hint = donsHint(leftHand, dealerCards[0])
    }
  }

  const stayLeft = (): void => {
    leftHandDone = true
    hint = donsHint(rightHand, dealerCards[0])
  }

  const stayRight = (): void => {
    rightHandDone = true
    lockedIn = true
    dealerCards = playOutDealerHand()

    let leftWon = false
    let leftPush = false
    let rightWon = false
    let rightPush = false
    // * Check left hand
    if (
      !isBusted(leftHand) &&
      (isBusted(dealerCards) || computeScore(leftHand) > computeScore(dealerCards) || computeScore(leftHand) === 21)
    ) {
      balance += bet * 2
      leftWon = true
    } else if (computeScore(leftHand) === computeScore(dealerCards)) {
      balance += bet
      leftPush = false
    }

    // * Check right hand
    if (
      isBusted(dealerCards) ||
      computeScore(rightHand) > computeScore(dealerCards) ||
      computeScore(rightHand) === 21
    ) {
      balance += bet * 2
      rightWon = true
    } else if (computeScore(rightHand) === computeScore(dealerCards)) {
      balance += bet
      rightPush = true
    }

    if ((leftWon && rightWon) || (leftWon && rightPush) || (leftPush && rightWon)) {
      userWon = true
      push = false
    } else if ((leftWon && !rightWon) || (!leftWon && rightWon)) {
      userWon = false
      push = true
    }
  }

  // ------ Click Handlers ----------

  const handleHit = (
    hand: Array<Card>,
    hitFunc: (hand: Array<Card>, bustFunc: () => void, who: HandType) => void,
    who: HandType,
  ): void => {
    checkForCorrectMove('Hit', hand)
    hitFunc(hand, handleBust, who)
  }

  const handleStay = (hand: Array<Card>, stayFunc: () => void): void => {
    checkForCorrectMove('Stay', hand)
    stayFunc()
  }

  const handleSplitHand = (): void => {
    checkForCorrectMove('Split', userCards)
    splitHand()
  }

  const handleDoubleDown = (
    hand: Array<Card>,
    hitFunc: (hand: Array<Card>, bustFunc: () => void, who: HandType) => void,
    stayFunc: () => void,
    who: HandType,
  ): void => {
    checkForCorrectMove('DoubleDown', hand)
    doubleDown(hand, () => hitFunc(hand, handleBust, who), stayFunc)
  }

  const handlePeek = (): void => {
    peekDealer = !peekDealer
  }

  const nextHand = (): void => {
    balance -= bet
    lockedIn = false
    split = false
    userWon = false
    push = false
    wonInsurance = false
    betOnInsurance = false
    leftHandDone = false
    rightHandDone = false
    userCards = [drawCard(), drawCard()]
    dealerCards = [drawCard(), drawCard()]
    insuranceOpen = dealerCards[0].name === 'Ace' && computeScore(userCards) !== 21
    canSplit = userCards[0].name === userCards[1].name
    hint = donsHint(userCards, dealerCards[0])

    if (deck.length < 15) {
      deck = shuffle(newDeck())
      deckCount = 0
    }

    // * Write to local storage
    localBucket.setItem('Balance', balance.toString())
  }

  const toggleHint = (): void => {
    hintEnabled = !hintEnabled
    if (hintEnabled) {
      hideInfoMessage = false
    }
  }

  const handleInsurance = (): void => {
    lockedIn = true
    betOnInsurance = true

    if (computeScore(dealerCards) === 21) {
      wonInsurance = true
      userWon = false
      push = false
    } else {
      wonInsurance = false
      balance -= insuranceBet
      lockedIn = false
      setTimeout(() => {
        insuranceOpen = false
        hint = donsHint(userCards, dealerCards[0])
      }, 2000)
    }
    hint = donsHint(userCards, dealerCards[0])
  }

  const denyInsurance = (): void => {
    if (computeScore(dealerCards) === 21) {
      wonInsurance = false
      userWon = false
      push = false
      stay()
    }

    insuranceOpen = false
    hint = donsHint(userCards, dealerCards[0])
  }

  // ------- Utils ------------

  const newDeck = (): Array<Card> => {
    let result = new Array<Card>()
    const suites: Array<Suite> = ['❤️', '♦', '♣️', '♠️']
    suites.forEach(suite => {
      for (let i = 2; i <= 14; i++) {
        result.push(indexToCard(i, suite))
      }
    })

    return result
  }

  const drawCard = (): Card => {
    const card = deck[0]
    deck = deck.slice(1)
    deckCount += cardCount(card)
    return card
  }

  const shuffle = (deck: Array<Card>): Array<Card> => {
    return deck.sort(() => Math.random() - 0.5)
  }

  const indexToCard = (idx: number, suite: Suite): Card => {
    switch (idx) {
      case 2:
        return { name: 'Two', value: idx, optionalValue: null, suite: suite }
      case 3:
        return { name: 'Three', value: idx, optionalValue: null, suite: suite }
      case 4:
        return { name: 'Four', value: idx, optionalValue: null, suite: suite }
      case 5:
        return { name: 'Five', value: idx, optionalValue: null, suite: suite }
      case 6:
        return { name: 'Six', value: idx, optionalValue: null, suite: suite }
      case 7:
        return { name: 'Seven', value: idx, optionalValue: null, suite: suite }
      case 8:
        return { name: 'Eight', value: idx, optionalValue: null, suite: suite }
      case 9:
        return { name: 'Nine', value: idx, optionalValue: null, suite: suite }
      case 10:
        return { name: 'Ten', value: idx, optionalValue: null, suite: suite }
      case 11:
        return { name: 'Jack', value: 10, optionalValue: null, suite: suite }
      case 12:
        return { name: 'Queen', value: 10, optionalValue: null, suite: suite }
      case 13:
        return { name: 'King', value: 10, optionalValue: null, suite: suite }
      case 14:
        return { name: 'Ace', value: 11, optionalValue: 1, suite: suite }
      default:
        throw new Error('Bad card value')
    }
  }

  const cardToImage = (card: Card): string => {
    return card.value < 10 || card.name === 'Ten'
      ? card.value + suitePrefix(card.suite)
      : card.name[0] + suitePrefix(card.suite)
  }

  const suitePrefix = (suite: Suite): string => {
    switch (suite) {
      case '❤️':
        return 'H'
      case '♠️':
        return 'S'
      case '♣️':
        return 'C'
      case '♦':
        return 'D'
      default:
        throw new Error('Bad suite')
    }
  }

  const isBusted = (cards: Array<Card>): boolean => {
    return computeScore(cards) > 21
  }

  const playOutDealerHand = (): Array<Card> => {
    while (!isBusted(dealerCards) && computeScore(dealerCards) < 17) {
      const newCard = drawCard()
      dealerCards = [...dealerCards, newCard]
      deckCount += cardCount(newCard)
    }

    return dealerCards
  }

  const handleBust = (): void => {
    lockedIn = true
    userWon = false
    push = false
  }

  const handleSplitBust = (): void => {
    rightHandDone = true
    lockedIn = true

    if (
      !isBusted(leftHand) &&
      (isBusted(dealerCards) || computeScore(leftHand) > computeScore(dealerCards) || computeScore(leftHand) === 21)
    ) {
      userWon = false
      push = true
    } else if (computeScore(leftHand) === computeScore(dealerCards)) {
      userWon = false
      push = false
    }
  }

  const handleKeydown = (event: any) => {
    switch (event.key) {
      case 'Enter':
        if (lockedIn) {
          nextHand()
        }
        break
      case 'ArrowLeft':
      case 'a':
        if (!lockedIn && !split) {
          handleStay(userCards, stay)
        }
        if (split && !leftHandDone) {
          handleStay(leftHand, stayLeft)
        } else if (split && leftHandDone && !rightHandDone) {
          handleStay(rightHand, stayRight)
        }
        break
      case 'ArrowRight':
      case 'd':
        if (!lockedIn && !split) {
          handleHit(userCards, () => hit(userCards, handleBust, 'User'), 'User')
        }
        if (split && !leftHandDone) {
          handleHit(leftHand, () => hit(leftHand, handleSplitBust, 'Left'), 'Left')
        } else if (split && leftHandDone && !rightHandDone) {
          handleHit(rightHand, () => hit(rightHand, handleSplitBust, 'Right'), 'Right')
        }
        break
      case 'ArrowUp':
      case 'w':
        if (canSplit && !split) {
          handleSplitHand()
        }
        break
      case 'ArrowDown':
      case 's':
        if (!lockedIn && userCards.length === 2 && !split) {
          handleDoubleDown(userCards, () => hit(userCards, handleBust, 'User'), stay, 'User')
        }

        if (split && !leftHandDone) {
          handleDoubleDown(leftHand, () => hit(leftHand, handleSplitBust, 'Left'), stayLeft, 'Left')
        } else if (split && leftHandDone && !rightHandDone) {
          handleDoubleDown(rightHand, () => hit(rightHand, handleSplitBust, 'Right'), stayRight, 'Right')
        }
        break
      default:
        break
    }
  }

  const donsHint = (userCards: Array<Card>, dealerUpCard: Card): string => {
    if (insuranceOpen) {
      hintColor = 'is-info'
      return "Most of the time insurance is a bad bet. Only go for it if you really think they've got 21"
    }
    let decision = decideMove(userCards, dealerUpCard, !split)
    switch (decision) {
      case 'Stay':
        hintColor = 'is-danger'
        return 'Better to play it safe on this one and stay'
      case 'Hit':
        hintColor = 'is-primary'
        return 'The smart thing to do here is to hit!'
      case 'DoubleDown':
        hintColor = 'is-success'
        return "Things are looking good I'd double down on this one 💰!"
      case 'Split':
        hintColor = 'is-warning'
        return 'Splitting looks like your best option!'
    }
  }

  const checkForCorrectMove = (move: Decision, cards: Array<Card>): void => {
    const correctDecision = decideMove(cards, dealerCards[0], !split)
    if (move === correctDecision) {
      correctDecisions += 1
    }
    handsPlayed += 1
  }

  const progress = tweened(0, {
    duration: 400,
    easing: cubicOut,
  })

  // ----------- State -----------

  let balance = 100
  let bet = 10
  let deckCount = 0
  let peekDealer = false
  let lockedIn = false
  let split = false
  let hintEnabled = false
  let userWon = false
  let push = false
  let leftHandDone = false
  let rightHandDone = false
  let hintColor = 'is-info'
  let handsPlayed = 1
  let correctDecisions = 1
  let hideInfoMessage = false
  let hideInfoTip = false
  let insuranceBet = Math.floor(bet / 2)
  let deck = shuffle(newDeck())
  let dealerCards: Array<Card> = [drawCard(), drawCard()]
  let userCards: Array<Card> = [drawCard(), drawCard()]
  let insuranceOpen = dealerCards[0].name === 'Ace' && computeScore(userCards) !== 21
  let canSplit = userCards[0].name === userCards[1].name
  let leftHand: Array<Card> = []
  let rightHand: Array<Card> = []
  let hint = donsHint(userCards, dealerCards[0])
  let wonInsurance = false
  let betOnInsurance = false

  // Load in balance from localStorage on component mounting
  onMount(() => {
    const storageValue = localBucket.getItem('Balance')
    if (storageValue === null) {
      localBucket.setItem('Balance', balance.toString())
    } else {
      balance = parseInt(storageValue || '')
    }
  })
</script>

<style>
  button {
    margin: 5px 5px 5px 5px;
  }

  li {
    margin-left: 15px;
  }

  hr {
    margin-top: 15px;
    width: 85%;
  }

  :disabled {
    pointer-events: none;
  }

  .title {
    margin-top: 20px;
  }

  .subtitle {
    margin-bottom: 0.5rem !important;
  }

  .tag {
    margin-top: 5px;
  }

  .fa-dollar-sign {
    margin-bottom: 20px;
  }

  #bet {
    width: 100px;
    margin-top: 5px;
  }

  #insuranceBet {
    width: 100px;
    margin-top: 5px;
  }

  #upNext {
    margin-top: 80px;
  }

  #correctPct {
    width: 300px;
    margin-left: 10px;
    margin-right: 10px;
    margin-top: 5px;
  }

  #controlBar {
    margin-left: 15vw;
  }
</style>

<svelte:window on:keydown={handleKeydown} />

<div class="columns is-mobile is-centered" id="blackJackContainer">
  <div class="column is-11">

    <h1 class="title is-centered">BlackJack</h1>

    <h2 class="subtitle">
      {peekDealer || lockedIn ? `Dealer's Hand : ${computeScore(dealerCards)}` : `Dealer's Hand`}
    </h2>

    <CardList cards={dealerCards.map(c => cardToImage(c))} visible={peekDealer || lockedIn} />
    <hr />

    {#if split}
      <h2 class="subtitle is-primary">
        Left Hand : {computeScore(leftHand)}
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        Right Hand : {computeScore(rightHand)}
      </h2>
    {:else}
      <h2 class="subtitle">Your Hand : {computeScore(userCards)}</h2>
    {/if}

    {#if split}
      <ul>
        <CardList cards={leftHand.map(c => cardToImage(c))} />
        <span style="display:inline-block; width: 200px;" />
        <CardList cards={rightHand.map(c => cardToImage(c))} />
      </ul>
    {:else}
      <CardList cards={userCards.map(c => cardToImage(c))} />
    {/if}

    <hr />
    <!-- Message Fly-In -->
    {#if lockedIn}
      <div
        class={`notification is-narrow ${userWon ? 'is-success' : push ? 'is-info' : 'is-danger'}`}
        transition:fly={{ x: -1000, duration: 500, delay: 200 }}>
        <span class={`tag is-large ${userWon ? 'is-success' : push ? 'is-info' : 'is-danger'}`} id="wonOrLost">
          <strong>{userWon ? 'You Won!' : push ? 'You Tied!' : 'You Lost!'}</strong>
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
            max={balance} />

          <span class="icon is-small is-left">
            <i class="fa fa-dollar-sign" id="betDollarSign" />
          </span>
        </span>
        <button class="button is-info is-outlined is-light" on:click={nextHand}>
          <span>Next Hand</span>
          <span class="icon is-small">
            <i class="fas fa-angle-double-right" />
          </span>
        </button>

        <span>(or press enter)</span>
      </div>
    {/if}

    <!-- Control Bar -->
    <div class="is-centered box" id="controlBar">

      {#if insuranceOpen}
        <div class="field is-horizontal">
          <div transition:fly={{ x: -1000, duration: 500 }}>
            <span class="tag is-large is-info">Insurance ?</span>
            <span class="control has-icons-left">
              <input
                class="input is-info"
                type="number"
                id="insuranceBet"
                name="insuranceBet"
                bind:value={insuranceBet}
                max={Math.floor(bet / 2)}
                min={1} />

              <span class="icon is-small is-left">
                <i class="fa fa-dollar-sign" />
              </span>
            </span>

            <button class="button is-success is-outlined" on:click={handleInsurance}>
              <span class="icon is-small">
                <i class="fas fa-check" />
              </span>
            </button>

            <button class="button is-danger is-outlined" on:click={denyInsurance}>
              <span class="icon is-small">
                <i class="fas fa-times" />
              </span>
            </button>

            {#if wonInsurance}
              <span
                class="icon is-small"
                in:fly={{ y: -1000, duration: 500 }}
                out:fly={{ y: -1000, duration: 500, delay: 800 }}>
                <i class="fas fa-coins" />
              </span>

              <span
                class={`tag is-light is-success is-medium`}
                id="infoTag"
                transition:fly={{ x: 1000, duration: 500 }}>
                You won ${insuranceBet} from the insurance side bet even though you lost the hand.
              </span>
            {/if}

            {#if betOnInsurance && !wonInsurance}
              <span
                class={`tag is-light is-danger is-medium`}
                id="infoTag"
                transition:fly={{ x: 1000, duration: 500, delay: 500 }}>
                You lost ${insuranceBet} from the insurance side bet. Don't worry you've still got a chance!
              </span>
            {/if}

          </div>

        </div>
      {:else}
        <div class="field is-horizontal" transition:fly={{ x: 2000, duration: 500, delay: 200 }}>
          <div>

            {#if split}
              <button
                class="button is-danger is-outlined"
                on:click={() => handleStay(leftHand, stayLeft)}
                disabled={leftHandDone || isBusted(leftHand)}>
                <span class="icon is-small">
                  <i class="fas fa-chevron-left" />
                </span>
                <span>Stay 1</span>
                <span class="icon is-small">
                  <i class="fas fa-hand-paper" />
                </span>
              </button>
              <button
                class="button is-success is-outlined"
                on:click={() => handleDoubleDown(leftHand, () => hit(leftHand, handleSplitBust, 'Left'), stayLeft, 'Left')}
                disabled={leftHandDone || leftHand.length > 2}>
                <span class="icon is-small">
                  <i class="fas fa-chevron-down" />
                </span>
                <span>Double 1</span>
                <span class="icon is-small">
                  <i class="fas fa-coins" />
                </span>
              </button>
              <button
                class="button is-primary is-outlined"
                on:click={() => handleHit(leftHand, () => hit(leftHand, handleSplitBust, 'Left'), 'Left')}
                disabled={leftHandDone || isBusted(leftHand)}>
                <span class="icon is-small">
                  <i class="fas fa-hand-holding-medical" />
                </span>
                <span>Hit 1</span>
                <span class="icon is-small">
                  <i class="fas fa-chevron-right" />
                </span>
              </button>

              <button
                class="button is-danger is-outlined"
                on:click={() => handleStay(rightHand, stayRight)}
                disabled={!(leftHandDone || isBusted(leftHand)) || rightHandDone || isBusted(rightHand)}>
                <span class="icon is-small">
                  <i class="fas fa-chevron-left" />
                </span>
                <span>Stay 2</span>
                <span class="icon is-small">
                  <i class="fas fa-hand-paper" />
                </span>
              </button>
              <button
                class="button is-success is-outlined"
                on:click={() => handleDoubleDown(rightHand, () => hit(rightHand, handleSplitBust, 'Right'), stayRight, 'Right')}
                disabled={!leftHandDone || rightHand.length > 2 || rightHandDone}>
                <span class="icon is-small">
                  <i class="fas fa-chevron-down" />
                </span>
                <span>Double 2</span>
                <span class="icon is-small">
                  <i class="fas fa-coins" />
                </span>
              </button>
              <button
                class="button is-primary is-outlined"
                on:click={() => handleHit(rightHand, () => hit(rightHand, handleSplitBust, 'Right'), 'Right')}
                disabled={!(leftHandDone || isBusted(leftHand) || isBusted(rightHand)) || rightHandDone}>
                <span class="icon is-small">
                  <i class="fas fa-hand-holding-medical" />
                </span>
                <span>Hit 2</span>
                <span class="icon is-small">
                  <i class="fas fa-chevron-right" />
                </span>
              </button>
            {:else}
              <button
                class="button is-danger is-outlined"
                on:click={() => handleStay(userCards, stay)}
                disabled={lockedIn}>
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
                on:click={() => handleDoubleDown(userCards, () => hit(userCards, handleBust, 'User'), stay, 'User')}
                disabled={lockedIn || userCards.length !== 2}>
                <span class="icon is-small">
                  <i class="fas fa-chevron-down" />
                </span>
                <span>Double</span>
                <span class="icon is-small">
                  <i class="fas fa-coins" />
                </span>
              </button>

              <button
                class="button is-warning is-outlined"
                id="splitButton"
                on:click={handleSplitHand}
                disabled={!canSplit || split}>
                <span class="icon is-small">
                  <i class="fas fa-chevron-up" />
                </span>
                <span>Split</span>
                <span class="icon is-small">
                  <i class="fas fa-expand-alt" />
                </span>
              </button>

              <button
                class="button is-primary is-outlined"
                on:click={() => handleHit(userCards, () => hit(userCards, handleBust, 'User'), 'User')}
                disabled={lockedIn}>
                <span class="icon is-small">
                  <i class="fas fa-hand-holding-medical" />
                </span>
                <span>Hit</span>
                <span class="icon is-small">
                  <i class="fas fa-chevron-right" />
                </span>
              </button>
            {/if}

          </div>

        </div>
      {/if}

      <div class="field is-horizontal">
        <div>

          <span class={`tag is-large ${balance >= 0 ? 'is-success' : 'is-danger'}`}>Balance: $ {balance}</span>

          {#if lockedIn && userWon}
            <span
              class="icon is-small"
              in:fly={{ y: -1000, duration: 500 }}
              out:fly={{ y: -1000, duration: 500, delay: 800 }}>
              <i class="fas fa-coins" />
            </span>
          {/if}

          <button
            class="button is-primary has-tooltip-multiline"
            id="hint"
            on:click={toggleHint}
            data-tooltip="Helpful hint">
            <span>{hintEnabled ? 'Fly Solo' : 'Ask Don'}</span>
            <span class="icon is-small">
              <i class={`fas fa-${hintEnabled ? 'plane' : 'question-circle'}`} />
            </span>
          </button>

          {#if hintEnabled}
            <span class={`tag ${hintColor} is-light is-large subtitle`} transition:fly={{ x: 2000, duration: 500 }}>
              {hint}
            </span>
          {/if}

        </div>

      </div>

      <div class="field is-horizontal">
        <label for="correctPct">Basic Strategy Correctness</label>
        <progress
          id="correctPct"
          class="progress is-primary"
          value={Math.floor((correctDecisions / handsPlayed) * 100)}
          max="100" />
        <span>{Math.floor((correctDecisions / handsPlayed) * 100)}%</span>
      </div>

      <div class="field is-horizontal">
        <span class={`tag is-light is-medium ${deckCount < 0 ? 'is-danger' : deckCount > 0 ? 'is-success' : ''}`}>
          <p class="subtitle">Count: {deckCount}</p>
        </span>
        {#if hintEnabled}
          {#if !hideInfoMessage}
            <span class={`tag is-light is-medium`} id="infoTag" transition:fly={{ x: 2000, duration: 500 }}>
              Don's hints are purely based on basic strategy he can't see any of the cards in the deck!
              <button
                class="delete"
                on:click={() => {
                  hideInfoMessage = true
                }} />
            </span>
          {/if}
        {/if}
      </div>
      <div class="field is-horizontal">
        {#if !hideInfoTip}
          <span class={`tag is-info is-light is-medium`} id="infoTag" transition:fly={{ x: -2000, duration: 500 }}>
            Tip: Use the arrow keys or WASD as an alternative to pressing the buttons (use the arrow icons on the
            buttons as a legend)
            <button
              class="delete"
              on:click={() => {
                hideInfoTip = true
              }} />
          </span>
        {/if}
      </div>
    </div>

  </div>

  <!-- Deck to Peek from -->
  <div class="column is-1">
    {#if peekDealer}
      <ul>
        {#each deck.slice(0, 5).reverse() as card, idx (card)}
          <li transition:slide>
            <figure class="image is-64x64">
              <img src={`./images/${cardToImage(card)}.jpg`} alt="playing card" />
            </figure>
          </li>
        {/each}
      </ul>
    {:else}
      <ul>
        {#each deck.slice(0, 5).reverse() as card, idx (card)}
          <li transition:slide>
            <figure class="image is-64x64">
              <img src={`./images/Gray_back.jpg`} alt="playing card" />
            </figure>
          </li>
        {/each}
      </ul>
    {/if}
    <span class="tag is-primary is-light is-medium" id="upNext">
      <span class="icon">
        <i class="fas fa-arrow-up" />
      </span>
      <p class="subtitle">Up Next</p>
    </span>

    <button class="button is-dark is-outlined" on:click={handlePeek}>

      <span>{peekDealer ? 'Play Clean' : 'Cheat'}</span>
      <span class="icon is-small">
        <i class={`fas fa-eye${peekDealer ? '-slash' : ''}`} />
      </span>
    </button>
  </div>
</div>
