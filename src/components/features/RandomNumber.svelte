<script lang="ts">
  const MIN_INTERVAL = 0.5;
  const MAX_INTERVAL = 5;

  let number = $state(0);
  let running = $state(false);
  let intervalSec = $state(2);
  let useRandomInterval = $state(false);
  let timeoutId: ReturnType<typeof setTimeout> | null = null;

  function generateNumber() {
    number = Math.floor(Math.random() * 100) + 1;
  }

  function getNextDelay(): number {
    if (useRandomInterval) {
      return (MIN_INTERVAL + Math.random() * (MAX_INTERVAL - MIN_INTERVAL)) * 1000;
    }
    return intervalSec * 1000;
  }

  function scheduleNext() {
    if (!running) return;
    timeoutId = setTimeout(() => {
      generateNumber();
      scheduleNext();
    }, getNextDelay());
  }

  function toggle() {
    if (running) {
      stop();
    } else {
      start();
    }
  }

  function start() {
    generateNumber();
    running = true;
    scheduleNext();
  }

  function stop() {
    running = false;
    if (timeoutId) {
      clearTimeout(timeoutId);
      timeoutId = null;
    }
  }
</script>

{#if running}
  <!-- Fullscreen display when running -->
  <div class="fullscreen-number">
    <span class="giant-number">{number}</span>
    <button class="stop-btn" onclick={toggle}>Stop</button>
  </div>
{:else}
  <!-- Setup UI when stopped -->
  <div class="setup-screen">
    <div class="preview-number">
      <span>{number || '?'}</span>
    </div>

    <div class="settings-card">
      <h2>Interval</h2>

      <label class="toggle-row">
        <span>Random timing</span>
        <input type="checkbox" bind:checked={useRandomInterval} />
      </label>

      {#if !useRandomInterval}
        <div class="slider-control">
          <input
            type="range"
            min={MIN_INTERVAL}
            max={MAX_INTERVAL}
            step="0.5"
            bind:value={intervalSec}
          />
          <span class="slider-value">{intervalSec}s</span>
        </div>
      {:else}
        <p class="random-hint">0.5 - 5 seconds</p>
      {/if}
    </div>

    <button class="start-btn" onclick={toggle}>
      Start
    </button>
  </div>
{/if}

<style>
  /* Fullscreen running state */
  .fullscreen-number {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 1000;
    background: #1a1a1a;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .giant-number {
    font-size: 50vw;
    font-weight: 800;
    color: #ffffff;
    line-height: 1;
    font-variant-numeric: tabular-nums;
  }

  .stop-btn {
    position: absolute;
    bottom: 2rem;
    padding: 1rem 3rem;
    font-size: 1.25rem;
    font-weight: 600;
    background: rgba(255, 255, 255, 0.15);
    color: rgba(255, 255, 255, 0.8);
    border: 2px solid rgba(255, 255, 255, 0.3);
    border-radius: 50px;
    cursor: pointer;
    transition: all 0.2s;
  }

  .stop-btn:hover {
    background: rgba(255, 255, 255, 0.25);
    color: #fff;
  }

  /* Setup screen */
  .setup-screen {
    min-height: calc(100vh - 80px);
    min-height: calc(100dvh - 80px);
    display: flex;
    flex-direction: column;
    padding: 1rem;
  }

  .preview-number {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #1a1a1a;
    border-radius: 20px;
    margin-bottom: 1.5rem;
  }

  .preview-number span {
    font-size: 30vw;
    font-weight: 800;
    color: #ffffff;
    opacity: 0.4;
  }

  .settings-card {
    background: #fff;
    border-radius: 16px;
    padding: 1.5rem;
    margin-bottom: 1rem;
    box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  }

  .settings-card h2 {
    font-size: 0.875rem;
    font-weight: 600;
    color: #666;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    margin-bottom: 1rem;
  }

  .toggle-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.75rem 0;
    cursor: pointer;
  }

  .toggle-row span {
    font-size: 1rem;
    color: #333;
  }

  .toggle-row input[type="checkbox"] {
    width: 48px;
    height: 28px;
    -webkit-appearance: none;
    appearance: none;
    background: #ddd;
    border-radius: 14px;
    position: relative;
    cursor: pointer;
    transition: background 0.2s;
  }

  .toggle-row input[type="checkbox"]:checked {
    background: var(--color-primary);
  }

  .toggle-row input[type="checkbox"]::before {
    content: '';
    position: absolute;
    top: 2px;
    left: 2px;
    width: 24px;
    height: 24px;
    background: #fff;
    border-radius: 50%;
    transition: transform 0.2s;
    box-shadow: 0 1px 3px rgba(0,0,0,0.2);
  }

  .toggle-row input[type="checkbox"]:checked::before {
    transform: translateX(20px);
  }

  .slider-control {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 0.5rem 0;
  }

  .slider-control input[type="range"] {
    flex: 1;
    height: 6px;
    -webkit-appearance: none;
    appearance: none;
    background: #e0e0e0;
    border-radius: 3px;
    outline: none;
  }

  .slider-control input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 28px;
    height: 28px;
    background: var(--color-primary);
    border-radius: 50%;
    cursor: pointer;
    box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  }

  .slider-control input[type="range"]::-moz-range-thumb {
    width: 28px;
    height: 28px;
    background: var(--color-primary);
    border-radius: 50%;
    cursor: pointer;
    border: none;
    box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  }

  .slider-value {
    min-width: 3rem;
    font-size: 1.25rem;
    font-weight: 700;
    color: var(--color-primary);
    text-align: right;
  }

  .random-hint {
    color: #888;
    font-size: 0.9rem;
    padding: 0.5rem 0;
  }

  .start-btn {
    width: 100%;
    padding: 1.25rem;
    font-size: 1.25rem;
    font-weight: 700;
    background: var(--color-primary);
    color: #fff;
    border: none;
    border-radius: 16px;
    cursor: pointer;
    transition: transform 0.1s, background 0.2s;
  }

  .start-btn:hover {
    background: var(--color-primary-dark);
  }

  .start-btn:active {
    transform: scale(0.98);
  }
</style>
