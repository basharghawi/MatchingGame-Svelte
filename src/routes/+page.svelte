<script lang="ts">
	import { emoji } from './emoji'

  type State = 'start' | 'playing' | 'paused' | 'won' | 'lost';

  let state: State = 'start';
  let size = 20;
  let grid = createGrid();
  let maxMatches = grid.length / 2;
  let selected: number[] = [];
  let matches: string[] = [];
  let timerId: number | null = null;
  let time = 60;

  function startGameTimer() {
    function countDown() {
      state !== 'paused' && (time -= 1)
    }
    timerId = setInterval(countDown, 1000);
  }

  function createGrid() {
    let cards = new Set<string>();
    let maxSize = size / 2;

    while (cards.size < maxSize) {
      const randomIndex = Math.floor(Math.random() * emoji.length);
      cards.add(emoji[randomIndex]);
    }
    return shuffle([...cards, ...cards])
  }

  function shuffle<Items>(array: Items[]) {
    return array.sort(() => Math.random() - 0.5)
  }

  function selectCard(cardIndex: number) {
    selected = selected.concat(cardIndex);
  }
  function matchCards() {
    const [first, second] = selected;

    if (grid[first] === grid[second]) {
      matches = matches.concat(grid[first])
    }

    setTimeout(() => {
      selected = []
    }, 300);

  }
  
  function resetGame() {
    timerId && clearInterval(timerId);
    grid = createGrid();
    maxMatches = grid.length / 2;
    selected = [];
    matches = [];
    timerId = null;
    if (diffVal === 'Eazy') {
      time = 60;
    } else if (diffVal === 'Medium') {
      time = 40;
    } else if (diffVal === 'Hard') {
      time = 25;
    }
  }

  function gameWon() {
    state = 'won';
    resetGame();
  }

  function gameLost() {
    state = 'lost';
    resetGame();
  }

  function pauseGame(e: KeyboardEvent) {
    if (e.key === 'Escape') {
      switch (state) {
        case "playing":
          state = "paused";
          break;
          case 'paused':
            state = "playing"
            break;
      }
    }
  }

  $: if (state === 'playing') {
    !timerId && startGameTimer()
  }

  $: selected.length === 2 && matchCards();

  $: maxMatches === matches.length && gameWon();

  $: time === 0 && gameLost();
  
  $: diffVal;
  // $: console.log(time);

  var diffVal: String = 'Eazy';
  var diffInputVal: number = 1;

  function sliderVal() {
    const difInput = document.getElementById('difficulty-slider') as HTMLInputElement;
    if (difInput.value == '1') {
      diffVal = 'Eazy';
      diffInputVal = 1
      time = 60;
    } else if (difInput.value == '2') {
      diffVal = 'Medium';
      diffInputVal = 2;
      time = 40;
    } else if (difInput.value == '3') {
      diffVal = 'Hard';
      diffInputVal = 3;
      time = 25;
    }
  }
</script>

<svelte:window on:keydown={pauseGame} />



{#if state === 'paused'}
  <h1>Game paused</h1>
{/if}


{#if state === 'start'}
  <h1>Matching Game</h1>
  <button on:click={() => state = 'playing'}>Play</button>
  <div class="difficultly">
    <label for="difficulty-slider">Difficultly: {diffVal}</label>
    <input class="test" type="range" id="difficulty-slider" min="1" max="3" value={diffInputVal}
            on:change={sliderVal}/>
  </div>
{/if}


{#if state === 'playing'}
  <div class="timer-wrapper">
    <h1 class="timer" class:pulse={time <= 10}>{time}</h1>
  </div>

  <div class="matches">
    {#each matches as card}
       <div>{card}</div>
    {/each}
  </div>

  <div class="cards">
  {#each grid as card, cardIndex}
    {@const isSelected = selected.includes(cardIndex)}
    {@const isSelectedOrMatched = selected.includes(cardIndex) || matches.includes(card)}
    {@const match = matches.includes(card)}
    <button class="card"
      disabled={isSelectedOrMatched}
      class:selected={isSelected}
      class:flip={isSelectedOrMatched}
      on:click={() => selectCard(cardIndex)}>
      <div class="back" class:match>{card}</div>
    </button>
  {/each}
  </div>
{/if}


{#if state === 'lost'}
   <h1>You Lost! 💩</h1>
   <button on:click={() => state = 'playing'}>Play again</button>
   <div class="difficultly">
    <label for="difficulty-slider">Difficultly: {diffVal}</label>
    <input class="test" type="range" id="difficulty-slider" min="1" max="3" value={diffInputVal}
            on:change={sliderVal}/>
  </div>
{/if}

{#if state === 'won'}
   <h1>You Won! 🥳</h1>
   <button on:click={() => state = 'playing'}>Play again</button>
   <div class="difficultly">
    <label for="difficulty-slider">Difficultly: {diffVal}</label>
    <input class="test" type="range" id="difficulty-slider" min="1" max="3" value={diffInputVal}
            on:change={sliderVal}/>
  </div>
{/if}


<style>
  .difficultly {
    margin: 3rem auto;
    display: flex;
    align-items: center;
    flex-direction: column;
    & label {
      font-size: 2rem;
    }
    & input {
      width: 250px;
      margin: 1rem 0;
      background: transparent;
      border: 3px solid #d6d6d6;
      border-radius: 15px;
      outline: none;
      &::-webkit-slider-thumb {
        -webkit-appearance: none;
        width: 20px;
        height: 20px;
        cursor: pointer;
        background: rgb(0, 201, 201);
        /* border: 1px solid blue; */
        border-radius: 50%;
      }
    }
  }
  .cards {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 15px;
    user-select: none;

    & .card {
      height: 140px;
      width: 140px;
      font-size: 4rem;
      background-color: var(--bg-2);
      border: 4px solid transparent;
      transition: rotate 0.3s ease-out;
      transform-style: preserve-3d;
      & > div {
        user-select: none;
      }
      @media (max-width: 767px) {
        border-width: 2px;
        height: 60px;
        width: 60px;
        & > div {
          font-size: 33px;
        }
      }

      &.selected {
        border: 4px solid var(--border);
        @media (max-width: 767px) {
          border-width: 2px;
        }
      }

      &.flip {
        rotate: y 180deg;
        pointer-events: none;
      }

      & .back {
        position: absolute;
        inset: 0;
        display: grid;
        place-content: center;
        backface-visibility: hidden;
        rotate: y 180deg;
      }

      & .match {
        transition: opacity .3s ease-out;
        opacity: 0.5;
      }
    }
  }
  .matches {
    display: flex;
    gap: 1rem;
    margin-block: 2rem;
    font-size: 3rem;
    height: 72px;
    user-select: none;
  }
  .timer-wrapper {
    height: 95px;
    overflow: hidden;
    user-select: none;
    @media (max-width: 767px) {
      height: 72px;
    }
  }
  .timer {
    transition: color .3s ease;
    user-select: none;
  }
  .pulse {
    color: var(--pulse);
    animation: pulse 1s infinite ease;
  }
  @keyframes pulse {
    to {
      scale: 1.4;
    }
  }
</style>