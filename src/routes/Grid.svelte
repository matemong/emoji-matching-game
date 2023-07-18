<script lang="ts">
  import { createEventDispatcher } from "svelte";
  import Square from "./Square.svelte";

  export let grid: string[];
  export let found: string[];

  const dispatch = createEventDispatcher();

  let a: number = -1;
  let b: number = -1;

  let reset_timeout: number;
</script>

  {#each grid as emoji, i}
    <Square
      {emoji}
      on:click={() => {
        clearTimeout(reset_timeout);
        if (a === -1 && b === -1) {
          a = i;
        } else if (b === -1) {
          b = i;
          if (grid[a] === grid[b]) {
            found = [...found, grid[a]];
            dispatch("found", {
              emoji: grid[a],
            });
          } else {
            reset_timeout = setTimeout(() => {
              a = b = -1;
            }, 1000);
          }
        } else {
          b = -1;
          a = i;
        }
      }}
      selected={a === i || b === i}
      found={found.includes(emoji)}
      group={grid.indexOf(emoji) === i ? "a" : "b"}
    />
  {/each}

