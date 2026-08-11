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

  function formatInterval(): string {
    if (useRandomInterval) return 'random (0.5-5s)';
    return `${intervalSec}s`;
  }
</script>

<div class="random-number">
  <div class="display-box">
    <span class="display-number">{number || '—'}</span>
  </div>

  <p class="status" class:running>
    {running ? `Running - ${formatInterval()}` : 'Stopped'}
  </p>

  <div class="interval-controls">
    <label class="random-toggle">
      <input type="checkbox" bind:checked={useRandomInterval} disabled={running} />
      Random interval
    </label>

    {#if !useRandomInterval}
      <div class="slider-row">
        <span class="slider-label">{intervalSec}s</span>
        <input
          type="range"
          min={MIN_INTERVAL}
          max={MAX_INTERVAL}
          step="0.5"
          bind:value={intervalSec}
          disabled={running}
        />
      </div>
    {/if}
  </div>

  <div class="controls">
    <button class="btn btn-large" class:btn-primary={!running} class:btn-secondary={running} onclick={toggle}>
      {running ? 'Stop' : 'Start'}
    </button>
  </div>
</div>

<style>
  .interval-controls {
    margin-bottom: var(--space-lg);
  }

  .random-toggle {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: var(--space-sm);
    margin-bottom: var(--space-md);
    cursor: pointer;
  }

  .random-toggle input {
    width: 20px;
    height: 20px;
    accent-color: var(--color-primary);
  }

  .slider-row {
    display: flex;
    align-items: center;
    gap: var(--space-md);
  }

  .slider-label {
    min-width: 3rem;
    text-align: center;
    font-weight: 600;
    color: var(--color-primary);
  }

  .slider-row input[type="range"] {
    flex: 1;
    height: 8px;
    -webkit-appearance: none;
    appearance: none;
    background: var(--color-border);
    border-radius: 4px;
    outline: none;
  }

  .slider-row input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 24px;
    height: 24px;
    background: var(--color-primary);
    border-radius: 50%;
    cursor: pointer;
  }

  .slider-row input[type="range"]::-moz-range-thumb {
    width: 24px;
    height: 24px;
    background: var(--color-primary);
    border-radius: 50%;
    cursor: pointer;
    border: none;
  }

  .slider-row input[type="range"]:disabled {
    opacity: 0.5;
  }
</style>
