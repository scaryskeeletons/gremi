<script>
  import { onMount } from 'svelte';

  let canvas, ctx;
  let isDrawing = false;
  export let color = '#000000'; // Default color
  export let lineWidth = 4;     // Default line width
  let imageSrc = 'gremlinhanddrawn.png'; // Path to your overlay image

  let lastX, lastY; // Track the last points for smooth curves
  let states = [];
  let currentStateIndex = -1;

  function saveState() {
    if (currentStateIndex + 1 < states.length) {
      states = states.slice(0, currentStateIndex + 1);
    }
    states.push(canvas.toDataURL());
    currentStateIndex++;
    if (states.length > 10) {
      states.shift();
      currentStateIndex--; // Adjust the index after shifting
    }
  }

  function undo() {
    if (currentStateIndex > 0) {
      currentStateIndex--;
      restoreState();
    }
  }

  function redo() {
    if (currentStateIndex < states.length - 1) {
      currentStateIndex++;
      restoreState();
    }
  }

  function download() {
    const link = document.createElement('a');
    const downloadCanvas = document.createElement('canvas');
    const downloadCtx = downloadCanvas.getContext('2d');

    downloadCanvas.width = canvas.width;
    downloadCanvas.height = canvas.height;

    // Fill the background with white
    downloadCtx.fillStyle = 'white';
    downloadCtx.fillRect(0, 0, downloadCanvas.width, downloadCanvas.height);

    // Draw the original canvas on top
    downloadCtx.drawImage(canvas, 0, 0);

    link.download = 'drawing.png';
    link.href = downloadCanvas.toDataURL();
    link.click();
  }

  function restoreState() {
    const img = new Image();
    img.onload = () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(img, 0, 0);
      drawOverlayImage();
    };
    img.src = states[currentStateIndex];
  }

  function startDrawing(event) {
    event.preventDefault();
    isDrawing = true;
    const { x, y } = getPosition(event);
    lastX = x;
    lastY = y;
    ctx.beginPath();
    ctx.moveTo(x, y);
  }

  function draw(event) {
    event.preventDefault();
    if (!isDrawing || !ctx) return;
    const { x, y } = getPosition(event);
    smoothLine(x, y);
  }

  function stopDrawing(event) {
    event.preventDefault();
    if (!isDrawing) return;
    isDrawing = false;
    ctx.lineTo(lastX, lastY);
    ctx.stroke(); // Draw the final segment
    saveState();
  }

  function getPosition(event) {
    let clientX, clientY;
    if (event.touches && event.touches.length) {
      clientX = event.touches[0].clientX;
      clientY = event.touches[0].clientY;
    } else {
      clientX = event.clientX;
      clientY = event.clientY;
    }
    return {
      x: clientX - canvas.getBoundingClientRect().left,
      y: clientY - canvas.getBoundingClientRect().top,
    };
  }

  function smoothLine(x, y) {
    ctx.quadraticCurveTo(lastX, lastY, (lastX + x) / 2, (lastY + y) / 2);
    ctx.stroke();
    lastX = x;
    lastY = y;
  }

  function drawOverlayImage() {
    const img = new Image();
    img.onload = () => {
      const padding = 20; // Adjust padding as needed
      const maxWidth = canvas.width - 2 * padding;
      const maxHeight = canvas.height - 2 * padding;
      let imgWidth = img.width;
      let imgHeight = img.height;

      if (imgWidth > maxWidth || imgHeight > maxHeight) {
        const ratio = Math.min(maxWidth / imgWidth, maxHeight / imgHeight);
        imgWidth *= ratio;
        imgHeight *= ratio;
      }

      const centerX = (canvas.width - imgWidth) / 2;
      const centerY = (canvas.height - imgHeight) / 2;
      ctx.drawImage(img, centerX, centerY, imgWidth, imgHeight);
    };
    img.src = imageSrc;
  }

  function resizeToolbar() {
    const toolbar = document.querySelector('.toolbar');
    const padding = 10; // Adjust padding as needed
    const maxWidth = canvas.width - 2 * padding;
    const maxHeight = 60; // Maximum height for the toolbar
    let toolbarWidth = toolbar.scrollWidth;
    let toolbarHeight = toolbar.scrollHeight;

    if (toolbarWidth > maxWidth) {
      toolbar.style.width = `${maxWidth}px`;
    } else {
      toolbar.style.width = 'auto';
    }

    if (toolbarHeight > maxHeight) {
      toolbar.style.height = `${maxHeight}px`;
    } else {
      toolbar.style.height = 'auto';
    }

    // Center the toolbar
    toolbar.style.left = '50%';
    toolbar.style.transform = 'translateX(-50%)';
  }

  onMount(() => {
    ctx = canvas.getContext('2d');
    setupCanvas();
    drawOverlayImage();
    resizeToolbar();
    saveState();
    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  });

  function setupCanvas() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    canvas.style.width = `${window.innerWidth}px`;
    canvas.style.height = `${window.innerHeight}px`;
    ctx.strokeStyle = color;
    ctx.lineWidth = lineWidth;
    ctx.lineCap = 'round'; // Smooth the end of the line
    ctx.lineJoin = 'round'; // Smooth the joint of the line
  }

  function handleResize() {
    const imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    setupCanvas();
    ctx.putImageData(imgData, 0, 0);
    drawOverlayImage();
    resizeToolbar();
  }

  $: if (ctx) {
    ctx.strokeStyle = color;
    ctx.lineWidth = lineWidth;
  }
</script>

<div class="toolbar">
  <input type="color" bind:value={color} />
  <input type="range" min="1" max="80" bind:value={lineWidth} />
  <button class="toolbar-button" on:click={undo}>
    <i class="fas fa-undo"></i>
  </button>
  <button class="toolbar-button" on:click={redo}>
    <i class="fas fa-redo"></i>
  </button>
  <button class="toolbar-button" on:click={download}>
    <i class="fas fa-download"></i>
  </button>
</div>

<canvas
  bind:this={canvas}
  on:mousedown={startDrawing}
  on:mousemove={draw}
  on:mouseup={stopDrawing}
  on:mouseout={stopDrawing}
  on:blur={stopDrawing}
  on:touchstart={startDrawing}
  on:touchmove={draw}
  on:touchend={stopDrawing}
></canvas>

<style>
  .toolbar {
    position: fixed;
    top: 10px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 10px;
    z-index: 100;
    background-color: rgba(255, 255, 255, 0.9);
    padding: 5px 10px;
    border-radius: 8px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
    transition: width 0.3s ease, height 0.3s ease; /* Smooth transition for resizing */
  }

  .toolbar-button {
    background: none;
    border: none;
    cursor: pointer;
    font-size: 24px; /* Adjust icon size */
  }

  .toolbar-button i {
    color: #000;
  }
</style>
