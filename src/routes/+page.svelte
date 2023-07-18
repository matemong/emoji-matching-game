<script lang="ts">
  import Game from "./Game.svelte";
  import "../styles.css";
  import Modal from "./Modal.svelte";
  import { levels } from "./levels";
  import { confetti } from "@neoconfetti/svelte";
  let state: "waiting" | "playing" | "paused" | "won" | "lost" = "waiting";

  let game: Game;
</script>

<svelte:head>
	<title>emoji matching game</title>
	<meta name="description" content="emoji matching game" />

</svelte:head>

<main>
  <Game
    bind:this={game}
    on:play={() => {
      state = "playing";
    }}
    on:pause={() => {
      state = "paused";
    }}
    on:win={() => {
      state = "won";
    }}
    on:lose={() => {
      state = "lost";
    }}
  />

  {#if state !== "playing"}
    <Modal>
      <header>
        <h1>emoji <span>match</span>ing game</h1>
      </header>
      {#if state === "won" || state === "lost"}
        <p>you {state}! play again?</p>
      {:else if state === "paused"}
        <p>game paused</p>
      {:else if state === "waiting"}
        <p>choose a difficulty level:</p>
      {/if}
      <div class="buttons">
        {#if state === "paused"}
          <button on:click={() => game.resume()}>resume</button>
          <button on:click={() => (state = "waiting")}> quit </button>
        {:else}
          {#each levels as level}
            <button
              on:click={() => {
                game.start(level);
                state = "playing";
              }}>{level.label}</button
            >
          {/each}
        {/if}
      </div>
    </Modal>
  {/if}

  {#if state === "won"}
    <div
      class="confetti"
      use:confetti={{
        stageWidth: innerWidth,
        stageHeight: innerHeight,
      }}
    />
  {/if}
</main>

<style>
  main {
    text-align: center;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
  header {
    font-size: min(5vw, 2rem);
  }
  
  h1 span {
    color: var(--accent);
  }
  .confetti {
    position: fixed;
    width: 100%;
    height: 100%;
    left: 50%;
    top: 30%;
    pointer-events: none;
  }
  .buttons {
		display: flex;
		justify-content: center;
		gap: 0.5em;
	}
  button {
    background: var(--accent);
    color: white;
    font-size: inherit;
    font-family: inherit;
    border: none;
    padding: 1em;
    border-radius: 0.5em;
  }
</style>
