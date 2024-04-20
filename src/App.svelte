<script lang="ts">
  import Chat from "./lib/Chat.svelte";
  import Connect from "./lib/Connect.svelte";
</script>

<main>
  <div class="flex h-screen items-center justify-center">
    {#if window.location.pathname == "/chat"}
      <Chat />
    {:else if window.location.pathname == "/"}
      <Connect />
    {:else}
      {(window.location.pathname = "/")}
    {/if}
  </div>
</main>
