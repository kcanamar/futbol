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

  function isSelected(index: number): boolean {
    return selectedIndices.has(index);
  }
</script>

{#if running}
  <!-- Fullscreen color display when running -->
  <div class="fullscreen-color" style:background-color={currentColor}>
    <button class="stop-btn" onclick={toggle}>Stop</button>
  </div>
{:else}
  <!-- Setup UI when stopped -->
  <div class="setup-screen">
    <div class="settings-card">
      <h2>Select {MIN_COLORS}-{MAX_COLORS} colors</h2>
      <p class="selection-count">{selectedIndices.size} selected</p>

      <div class="color-grid">
        {#each COLORS as color, index}
          <button
            class="color-chip"
            class:selected={isSelected(index)}
            style:background-color={color.hex}
            onclick={() => toggleColor(index)}
            aria-label={color.name}
          >
            {#if isSelected(index)}
              <svg viewBox="0 0 24 24" class="check-icon">
                <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z" fill="currentColor"/>
              </svg>
            {/if}
          </button>
        {/each}
      </div>
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
  .fullscreen-color {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 1000;
    display: flex;
    align-items: flex-end;
    justify-content: center;
    padding-bottom: 2rem;
  }

  .stop-btn {
    padding: 1rem 3rem;
    font-size: 1.25rem;
    font-weight: 600;
    background: rgba(0, 0, 0, 0.25);
    color: rgba(255, 255, 255, 0.9);
    border: 2px solid rgba(255, 255, 255, 0.4);
    border-radius: 50px;
    cursor: pointer;
    transition: all 0.2s;
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
  }

  .stop-btn:hover {
    background: rgba(0, 0, 0, 0.4);
  }

  /* Setup screen */
  .setup-screen {
    min-height: calc(100vh - 80px);
    min-height: calc(100dvh - 80px);
    display: flex;
    flex-direction: column;
    padding: 1rem;
    gap: 1rem;
  }

  .settings-card {
    background: #fff;
    border-radius: 16px;
    padding: 1.5rem;
    box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  }

  .settings-card h2 {
    font-size: 0.875rem;
    font-weight: 600;
    color: #666;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    margin-bottom: 0.25rem;
  }

  .selection-count {
    font-size: 0.8rem;
    color: #999;
    margin-bottom: 1rem;
  }

  .color-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 0.75rem;
  }

  .color-chip {
    aspect-ratio: 1;
    border-radius: 12px;
    border: none;
    cursor: pointer;
    transition: transform 0.15s, box-shadow 0.15s;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 60px;
  }

  .color-chip:hover {
    transform: scale(1.05);
  }

  .color-chip:active {
    transform: scale(0.95);
  }

  .color-chip.selected {
    box-shadow: 0 0 0 3px #fff, 0 0 0 5px #333;
  }

  .check-icon {
    width: 28px;
    height: 28px;
    color: white;
    filter: drop-shadow(0 1px 2px rgba(0,0,0,0.3));
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
    margin-top: auto;
  }

  .start-btn:hover {
    background: var(--color-primary-dark);
  }

  .start-btn:active {
    transform: scale(0.98);
  }
</style>
