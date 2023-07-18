<script lang="ts">
  import { createEventDispatcher, onMount } from "svelte";
  import Countdown from "./Countdown.svelte";
  import Found from "./Found.svelte";
  import Grid from "./Grid.svelte";
  import { levels } from "./levels";
  import type { Level } from "./levels";
  import { shuffle } from "./utils";

  const dispatch = createEventDispatcher();

  let size: number;
  let grid: string[] = [];
  let found: string[] = [];
  let remaining: number = 0;
  let duration: number = 0;
  let playing: boolean = false;

  export function start(level: Level) {
    size = level.size;
    grid = create_grid(level);
    remaining = duration = level.duration;
    
    //reset the found emoji array after restarting
    found = [];

    resume();
  }

  function resume() {
    playing = true;
    countdown();
    dispatch("play");
  }

  function create_grid(level: Level) {
    const copy = level.emojis.slice();
    const pairs: string[] = [];

    for (let i = 0; i < level.size ** 2 / 2; i++) {
      const index = Math.floor(Math.random() * copy.length);
      const emoji = copy[index];

      copy.splice(index, 1);
      pairs.push(emoji);
    }

    pairs.push(...pairs);

    return shuffle(pairs);
  }

  function countdown() {
    const start = Date.now();
    let remaining_at_start = remaining;

    function loop() {
      if (!playing) return;
      requestAnimationFrame(loop);

      remaining = remaining_at_start - (Date.now() - start);

      if (remaining <= 0) {
        dispatch("lose");
        playing = false;
      }
    }
    loop();
  }
</script>

<div class="game" style="--size: {size}">
  <div class="info">
    {#if playing}
      <Countdown
        {remaining}
        {duration}
        on:click={() => {
          playing = false;
          dispatch("pause");
        }}
      />
    {/if}
  </div>
  <div class="grid-container">
    {#key grid}
      <Grid
        {grid}
        on:found={(e) => {
          found = [...found, e.detail.emoji];
          if (found.length === (size * size) / 2) {
            playing = false;
            setTimeout(() => {
              playing = false;
              dispatch("win");
            }, 1000);
          }
        }}
        {found}
      />
    {/key}
  </div>

  <div class="info">
    <Found {found} />
  </div>
</div>

<style>
  .game {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    font-size: min(1vmin, 0.5em);
    gap: 2em;
    height: 100%;
  }

  .grid-container {
    display: grid;
    grid-template-columns: repeat(var(--size), 1fr);
    grid-template-rows: repeat(var(--size), 1fr);
    grid-gap: 1em;
    width: 80em;
    aspect-ratio: 1;
    perspective: 100vw;
    z-index: 2;
  }

  .info {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 80em;
    height: 10em;
  }

  @media (min-aspect-ratio: 1/1) {
    .game {
      flex-direction: row-reverse;
    }

    .info {
      width: 10em;
      height: 80em;
    }
  }
</style>
