<script>
  import DrawingTool from "./scripts/draw.svelte";

  function backToHome() {
      window.location.href = '/';
  }
</script>

<main class="main-content">
  <DrawingTool />
  <button class="back" on:click={backToHome}>
    Back to Home
</button>
  <div class="social-links">
      <a href="https://x.com/gizmo_onsol" target="_blank" rel="noopener" class="social-link">
          <img src="https://github.com/scaryskeeletons/images/raw/main/twitter.webp" alt="Twitter"/>
      </a>
      <a href="https://dexscreener.com/solana/" target="_blank" rel="noopener" class="social-link">
          <img src="https://github.com/scaryskeeletons/images/raw/main/dexscreener.webp" alt="Dex Screener"/>
      </a>
      <a href="https://t.me/gizmotg" target="_blank" rel="noopener" class="social-link">
          <img src="https://github.com/scaryskeeletons/images/raw/main/telegram.webp" alt="Telegram"/>
      </a>
  </div>
</main>

<style>
  @import "./styles/styles.css";
</style>
