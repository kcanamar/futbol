<script lang="ts">
  const ARROWS = [
    { name: 'Up', rotation: 0 },
    { name: 'Right', rotation: 90 },
    { name: 'Down', rotation: 180 },
    { name: 'Left', rotation: 270 },
  ];

  const MIN_INTERVAL = 0.5;
  const MAX_INTERVAL = 5;
  const MIN_FLASH = 0.2;
  const MAX_FLASH = 2;

  let currentArrow = $state<typeof ARROWS[0] | null>(null);
  let visible = $state(true);
  let running = $state(false);
  let intervalSec = $state(2);
  let useRandomInterval = $state(false);
  let flashMode = $state(false);
  let flashDuration = $state(0.5);
  let timeoutId: ReturnType<typeof setTimeout> | null = null;
  let flashTimeoutId: ReturnType<typeof setTimeout> | null = null;

  function generateArrow() {
    const index = Math.floor(Math.random() * ARROWS.length);
    currentArrow = ARROWS[index];
    visible = true;

    if (flashMode) {
      flashTimeoutId = setTimeout(() => {
        visible = false;
      }, flashDuration * 1000);
    }
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
      generateArrow();
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
    generateArrow();
    running = true;
    scheduleNext();
  }

  function stop() {
    running = false;
    visible = true;
    if (timeoutId) {
      clearTimeout(timeoutId);
      timeoutId = null;
    }
    if (flashTimeoutId) {
      clearTimeout(flashTimeoutId);
      flashTimeoutId = null;
    }
  }
</script>

{#if running}
  <div class="fullscreen-arrow">
    {#if visible}
      <svg
        class="arrow-icon"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2.5"
        stroke-linecap="round"
        stroke-linejoin="round"
        style:transform={`rotate(${currentArrow?.rotation ?? 0}deg)`}
      >
        <path d="M12 19V5"/>
        <path d="M5 12l7-7 7 7"/>
      </svg>
      <span class="direction-label">{currentArrow?.name}</span>
    {:else}
      <span class="blank-indicator">—</span>
    {/if}
    <button class="stop-btn" onclick={toggle}>Stop</button>
  </div>
{:else}
  <div class="setup-screen">
    <div class="preview-area">
      <div class="arrow-preview">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M12 19V5"/>
          <path d="M5 12l7-7 7 7"/>
        </svg>
      </div>
      <p class="preview-label">4 directions</p>
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

    <div class="settings-card">
      <h2>Flash Mode</h2>

      <label class="toggle-row">
        <span>Brief flash only</span>
        <input type="checkbox" bind:checked={flashMode} />
      </label>

      {#if flashMode}
        <p class="flash-description">Arrow disappears after flash duration</p>
        <div class="slider-control">
          <input
            type="range"
            min={MIN_FLASH}
            max={MAX_FLASH}
            step="0.1"
            bind:value={flashDuration}
          />
          <span class="slider-value">{flashDuration.toFixed(1)}s</span>
        </div>
      {/if}
    </div>

    <button class="start-btn" onclick={toggle}>
      Start
    </button>
  </div>
{/if}

<style>
  .fullscreen-arrow {
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

  .arrow-icon {
    width: 60vw;
    height: 60vw;
    max-width: 400px;
    max-height: 400px;
    color: #ffffff;
    transition: transform 0.15s ease-out;
  }

  .direction-label {
    font-size: 2rem;
    font-weight: 700;
    color: rgba(255, 255, 255, 0.6);
    margin-top: 1rem;
    text-transform: uppercase;
    letter-spacing: 0.2em;
  }

  .blank-indicator {
    font-size: 10rem;
    color: rgba(255, 255, 255, 0.15);
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

  .setup-screen {
    min-height: calc(100vh - 80px);
    min-height: calc(100dvh - 80px);
    display: flex;
    flex-direction: column;
    padding: 1rem;
    gap: 1rem;
  }

  .preview-area {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: #1a1a1a;
    border-radius: 20px;
    padding: 2rem;
    min-height: 150px;
  }

  .arrow-preview {
    width: 80px;
    height: 80px;
    color: rgba(255, 255, 255, 0.4);
  }

  .arrow-preview svg {
    width: 100%;
    height: 100%;
  }

  .preview-label {
    color: rgba(255, 255, 255, 0.5);
    margin-top: 1rem;
    font-size: 0.9rem;
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
    margin-bottom: 0.5rem;
  }

  .toggle-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem 0;
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

  .flash-description {
    font-size: 0.8rem;
    color: #888;
    margin-bottom: 0.5rem;
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
