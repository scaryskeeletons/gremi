<script>
    import { onMount, onDestroy } from 'svelte';
    import { Chart, PieController, ArcElement, Tooltip, Legend } from 'chart.js';
  
    // Register necessary Chart.js components
    Chart.register(PieController, ArcElement, Tooltip, Legend);
  
    let chart;
  
    onMount(() => {
      const ctx = document.getElementById('tokenomicsChart').getContext('2d');
      chart = new Chart(ctx, {
        type: 'pie',
        data: {
          labels: ['Burnt', 'Supply', 'Team'],
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
  </script>
  
  <div class="container">
    <div class="section" id="intro">
      <h1>$GR<span>EMI</span></h1>
      <div>
        <img src="gremlin.png" alt="GREMI">
      </div>
      <p>Solana's cutest little Gremlin</p>
      <span>CA: Coming soon</span>
    </div>
  
    <div class="section" id="links">
      <h1>$GREMI Links</h1>
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
      <p>$GREMI is a strong community-backed project with a team willing to spend their bag for your success. With an awesome developer there is nothing better on Solana than $GREMI</p>
    </div>
  
    <div class="section" id="tokenomics">
      <h1>Tokenomics</h1>
      <p>950,000,000 Total Supply</p>
      <p>100% of Liquidity Burnt</p>
      <p>Mint Authority Revoked</p>
      <span></span>
      <canvas id="tokenomicsChart" width="300" height="300"></canvas>
    </div>
  
    <div class="section" id="credits">
      <a href="https://t.me/fatlzt" target="_blank" rel="noopener">
        <p>Credits @fatlzt on Telegram for Site</p>
      </a>
    </div>
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
  </style>
  