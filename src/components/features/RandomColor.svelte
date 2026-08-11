<script lang="ts">
  const COLORS = [
    { name: 'Red', hex: '#e53935' },
    { name: 'Blue', hex: '#1e88e5' },
    { name: 'Green', hex: '#43a047' },
    { name: 'Yellow', hex: '#fdd835' },
    { name: 'Orange', hex: '#fb8c00' },
    { name: 'Purple', hex: '#8e24aa' },
    { name: 'Pink', hex: '#d81b60' },
    { name: 'Cyan', hex: '#00acc1' },
  ];

  const MIN_COLORS = 2;
  const MAX_COLORS = 6;
  const MIN_INTERVAL = 0.5;
  const MAX_INTERVAL = 5;

  let selectedIndices = $state(new Set([0, 1]));
  let currentColor = $state<string | null>(null);
  let running = $state(false);
  let intervalSec = $state(2);
  let useRandomInterval = $state(false);
  let timeoutId: ReturnType<typeof setTimeout> | null = null;

  function toggleColor(index: number) {
    if (running) return;

    const newSet = new Set(selectedIndices);
    if (newSet.has(index)) {
      if (newSet.size > MIN_COLORS) {
        newSet.delete(index);
      }
    } else {
      if (newSet.size < MAX_COLORS) {
        newSet.add(index);
      }
    }
    selectedIndices = newSet;
  }

  function generateColor() {
    const indices = Array.from(selectedIndices);
    const randomIndex = indices[Math.floor(Math.random() * indices.length)];
    currentColor = COLORS[randomIndex].hex;
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
      generateColor();
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
    generateColor();
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

  function isDisabled(index: number): boolean {
    if (running) return true;
    if (selectedIndices.has(index)) {
      return selectedIndices.size <= MIN_COLORS;
    }
    return selectedIndices.size >= MAX_COLORS;
  }

  function formatInterval(): string {
    if (useRandomInterval) return 'random (0.5-5s)';
    return `${intervalSec}s`;
  }
</script>

<div class="random-color">
  <div class="display-box" style:background-color={currentColor || '#ffffff'}>
    {#if !currentColor}
      <span class="display-number" style="font-size: 2rem; color: #999;">Select colors & start</span>
    {/if}
  </div>

  <p class="selection-hint">Select {MIN_COLORS}-{MAX_COLORS} colors ({selectedIndices.size} selected)</p>

  <div class="color-grid">
    {#each COLORS as color, index}
      <button
        class="color-chip"
        class:selected={selectedIndices.has(index)}
        class:disabled={isDisabled(index)}
        style:background-color={color.hex}
        onclick={() => toggleColor(index)}
        disabled={isDisabled(index)}
        aria-label={color.name}
        title={color.name}
      ></button>
    {/each}
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
  .selection-hint {
    text-align: center;
    color: var(--color-text-muted);
    font-size: var(--font-size-sm);
    margin-bottom: var(--space-md);
  }

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
