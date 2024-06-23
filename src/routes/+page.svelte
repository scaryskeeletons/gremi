<script>
  import { onMount, onDestroy } from 'svelte';
  import { Chart, PieController, ArcElement, Tooltip, Legend } from 'chart.js';
  import { fly } from 'svelte/transition';

  // Register necessary Chart.js components
  Chart.register(PieController, ArcElement, Tooltip, Legend);

  function launch() {
      window.location.href = '/draw';
  }
  let chart;

  onMount(() => {
      const ctx = document.getElementById('tokenomicsChart').getContext('2d');
      chart = new Chart(ctx, {
          type: 'pie',
          data: {
              labels: ['Marketing', 'Supply', 'Dev'],
              datasets: [{
                  label: 'Token Distribution',
                  data: [5, 95, 5],
                  backgroundColor: ['#d12150', '#fcba03', '#218ed1'], // Colors for the slices
                  hoverOffset: 4
              }]
          },
          options: {
              responsive: false, // Make the chart a fixed size
              plugins: {
                  legend: {
                      position: 'top',
                  },
                  tooltip: {
                      callbacks: {
                          label: function(context) {
                              let label = context.label || '';

                              if (label) {
                                  label += ': ';
                              }
                              if (context.parsed !== null) {
                                  label += context.parsed + '%';
                              }
                              return label;
                          }
                      }
                  }
              }
          }
      });
  });

  // Cleanup chart instance when component is destroyed
  onDestroy(() => {
      if (chart) {
          chart.destroy();
      }
  });

  let currentStage = -1;
  const stages = [
      { title: 'Stage 1', description: 'Pump.Fun Fairlaunch & Sniper Shakeout' },
      { title: 'Stage 2', description: 'Begin growing a cult centered around $GIZMO' },
      { title: 'Stage 3', description: '$GIZMO cult holds the floor and we blast off to Raydium' },
      { title: 'Stage 4', description: 'Dev uses marketing allocation to pay for Dexscreener and other sites & trending groups' }
  ];

  function setStage(index) {
      currentStage = index;
  }

  function closeStage() {
      currentStage = -1;
  }

  function handleBackdropClick(event) {
      if (event.target.classList.contains('backdrop')) {
          closeStage();
      }
  }
  
  let telegramUsername = "";
  let userImage = null;
  let overlayImage = null;
  let canvas, ctx;
  let fileChosen = false;
  let isDragging = false;
  let isResizing = false;
  let offsetX, offsetY;
  let resizeHandleSize = 10;
  let overlayX = 50, overlayY = 50, overlayWidth = 100, overlayHeight = 100;

  // Load the overlay image
  onMount(() => {
      overlayImage = new Image();
      overlayImage.src = 'gremears.png'; // Specify the overlay image path here
  });

  function handleImageUpload(event) {
      const file = event.target.files[0];
      if (file) {
          const reader = new FileReader();
          reader.onload = function(e) {
              userImage = new Image();
              userImage.onload = drawMeme;
              userImage.src = e.target.result;
          };
          reader.readAsDataURL(file);
          fileChosen = true;
      }
  }

  function drawMeme() {
      if (!userImage || !overlayImage) return;

      canvas = document.getElementById('memeCanvas');
      ctx = canvas.getContext('2d');

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(userImage, 0, 0, canvas.width, canvas.height);
      ctx.drawImage(overlayImage, overlayX, overlayY, overlayWidth, overlayHeight);

      // Draw resize handle
      ctx.fillStyle = 'red';
      ctx.fillRect(overlayX + overlayWidth - resizeHandleSize, overlayY + overlayHeight - resizeHandleSize, resizeHandleSize, resizeHandleSize);
  }

  function downloadMeme() {
      const downloadCanvas = document.createElement('canvas');
      downloadCanvas.width = 500;
      downloadCanvas.height = 500;
      const downloadCtx = downloadCanvas.getContext('2d');

      downloadCtx.drawImage(userImage, 0, 0, 500, 500);
      downloadCtx.drawImage(overlayImage, overlayX / 300 * 500, overlayY / 300 * 500, overlayWidth / 300 * 500, overlayHeight / 300 * 500);

      const link = document.createElement('a');
      link.href = downloadCanvas.toDataURL('image/png');
      link.download = 'meme.png';
      link.click();
  }

  function onMouseDown(event) {
      const { offsetX: mouseX, offsetY: mouseY } = event;
      if (isOnResizeHandle(mouseX, mouseY)) {
          isResizing = true;
      } else if (isOnOverlay(mouseX, mouseY)) {
          isDragging = true;
          offsetX = mouseX - overlayX;
          offsetY = mouseY - overlayY;
      }
  }

  function onMouseMove(event) {
      if (isDragging) {
          overlayX = event.offsetX - offsetX;
          overlayY = event.offsetY - offsetY;
          drawMeme();
      } else if (isResizing) {
          overlayWidth = event.offsetX - overlayX;
          overlayHeight = event.offsetY - overlayY;
          drawMeme();
      }
  }


  function onMouseUp() {
      isDragging = false;
      isResizing = false;
  }

  function onTouchStart(event) {
      const touch = event.touches[0];
      const mouseX = touch.clientX - canvas.getBoundingClientRect().left;
      const mouseY = touch.clientY - canvas.getBoundingClientRect().top;
      if (isOnResizeHandle(mouseX, mouseY)) {
          isResizing = true;
      } else if (isOnOverlay(mouseX, mouseY)) {
          isDragging = true;
          offsetX = mouseX - overlayX;
          offsetY = mouseY - overlayY;
      }
  }

  function onTouchMove(event) {
      const touch = event.touches[0];
      const mouseX = touch.clientX - canvas.getBoundingClientRect().left;
      const mouseY = touch.clientY - canvas.getBoundingClientRect().top;
      if (isDragging) {
          overlayX = mouseX - offsetX;
          overlayY = mouseY - offsetY;
          drawMeme();
      } else if (isResizing) {
          overlayWidth = mouseX - overlayX;
          overlayHeight = mouseY - overlayY;
          drawMeme();
      }
  }

  function onTouchEnd() {
      isDragging = false;
      isResizing = false;
  }

  function isOnOverlay(x, y) {
      return x >= overlayX && x <= overlayX + overlayWidth && y >= overlayY && y <= overlayY + overlayHeight;
  }

  function isOnResizeHandle(x, y) {
      return x >= overlayX + overlayWidth - resizeHandleSize && x <= overlayX + overlayWidth &&
             y >= overlayY + overlayHeight - resizeHandleSize && y <= overlayY + overlayHeight;
  }
</script>

<div class="container">
  <div class="section" id="intro">
      <h1>$GI<span>ZMO</span></h1>
      <div>
          <img src="gremlin.png" alt="GIZMO">
      </div>
      <p>Solana's cutest little Gremlin</p>
      <span>CA: Coming soon</span>
  </div>

  <div class="section" id="links">
      <h1>$GIZMO Links</h1>
      <div class="social-links">
          <a href="https://twitter.com/" target="_blank" rel="noopener" class="social-link">
              <img src="xlogo.webp" alt="Twitter"/>
          </a>
          <a href="https://pump.fun/" target="_blank" rel="noopener" class="social-link">
              <img src="dexscreener.webp" alt="DexScreener"/>
          </a>
          <a href="https://t.me/" target="_blank" rel="noopener" class="social-link">
              <img src="telegramlogo.webp" alt="Telegram"/>
          </a>
      </div>
  </div>

  <div class="section" id="about">
      <h1>About</h1>
      <p>$GIZMO is a strong community-backed project with a team willing to spend their bag for your success. With an awesome developer there is nothing better on Solana than $GIZMO</p>
  </div>

  <div class="section" id="tokenomics">
      <h1>Tokenomics</h1>
      <p>1,000,000,000 Total Supply</p>
      <p>100% of Liquidity Burnt</p>
      <p>Mint Authority Revoked</p>
      <p>Contract Renounced</p>
      <span></span>
      <canvas id="tokenomicsChart" width="300" height="300"></canvas>
  </div>

  <div class="section" id="roadmap">
      <h1>Roadmap</h1>
      <p>$GIZMO's roadmap for success:</p>
      <div class="roadmap">
          {#each stages as stage, index}
              <div class="roadmap-stage {index === currentStage ? 'active' : ''}" on:click={() => setStage(index)}>
                  <div class="stage-content">
                      <div class="stage-title">{stage.title}</div>
                      <div class="stage-note">Tap to view more</div>
                  </div>
              </div>
          {/each}
      </div>
  </div>

  <div class="section" id="mememaker">
    <h1>MemeMaker</h1>
    <p>Upload any image and get your own gizmo'd meme back.</p>
    {#if !fileChosen}
        <input type="file" accept="image/*" on:change={handleImageUpload} class="upload-button">
    {/if}
    <canvas id="memeCanvas" width="300" height="300"
            on:mousedown={onMouseDown}
            on:mousemove={onMouseMove}
            on:mouseup={onMouseUp}
            on:mouseleave={onMouseUp}
            on:touchstart={onTouchStart}
            on:touchmove={onTouchMove}
            on:touchend={onTouchEnd}></canvas>
    {#if fileChosen}
        <button on:click={downloadMeme} class="download-button">Download Meme</button>
    {/if}
</div>

<div class="section" id="launch">
  <h1>Draw</h1>
  <p>Create your own customized Gizmo!</p>
  <button on:click={launch} class="launch">Launch</button>
</div>

<div class="section" id="credits">
  <a href="https://t.me/fatlzt" target="_blank" rel="noopener" style="color: red;">
      <p>Credits @fatlzt on Telegram for Site</p>
  </a>
</div>

  {#if currentStage !== -1}
      <div class="backdrop" on:click={handleBackdropClick} transition:fly>
          <div class="modal" transition:fly>
              <button class="close-button" on:click={closeStage}>×</button>
              <h2>{stages[currentStage].title}</h2>
              <p>{stages[currentStage].description}</p>
          </div>
      </div>
  {/if}
</div>

<style>
  @import './styles/styles.css';

  #tokenomics {
      display: flex;
      flex-direction: column;
      align-items: center;
  }

  #tokenomicsChart {
      max-width: 300px;
      max-height: 300px;
  }

  .roadmap {
      display: flex;
      justify-content: space-around;
      align-items: center;
      width: 100%;
      margin-top: 20px;
      position: relative;
  }

  .roadmap-stage {
      border: 2px solid #ccc;
      border-radius: 10px;
      padding: 20px;
      width: 150px;
      text-align: center;
      cursor: pointer;
      transition: background-color 0.3s, transform 0.3s;
      position: relative;
  }

  .roadmap-stage.active {
      background-color: #fcba03;
      transform: scale(1.1);
      border-color: #d12150;
  }

  .roadmap-stage .stage-content {
      display: flex;
      flex-direction: column;
      align-items: center;
  }

  .roadmap-stage .stage-title {
      font-size: 18px;
      font-weight: bold;
      margin-bottom: 10px;
  }

  .roadmap-stage .stage-note {
      font-size: 12px;
      color: #ddd;
  }

  .roadmap::before {
      content: '';
      position: absolute;
      top: 50%;
      left: 0;
      right: 0;
      height: 2px;
      background-color: #d12150;
      z-index: -1;
  }

  .backdrop {
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: rgba(0, 0, 0, 0.5);
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 1000;
  }

  .modal {
      background: #342c31;
      border-radius: 10px;
      padding: 20px;
      max-width: 80%;
      width: 500px;
      text-align: center;
      color: white;
      position: relative;
  }

  .close-button {
      position: absolute;
      top: 10px;
      right: 10px;
      background: none;
      border: none;
      font-size: 24px;
      color: white;
      cursor: pointer;
  }

  .close-button:hover {
      color: #d12150;
  }

  #mememaker {
      display: flex;
      flex-direction: column;
      align-items: center;
  }

  #memeCanvas {
      border: 2px solid #d12150;
      margin: 10px 0;
  }

  .upload-button, .download-button {
      background-color: #d12150;
      color: white;
      border: none;
      padding: 10px 20px;
      margin-top: 10px;
      cursor: pointer;
      font-size: 16px;
      border-radius: 5px;
      transition: background-color 0.3s;
  }

  .upload-button:hover, .download-button:hover {
      background-color: #fcba03;
  }
</style>
