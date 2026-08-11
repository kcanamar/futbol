<script lang="ts">
  const COLORS = [
    { name: 'Red', hex: '#e53935', textColor: '#fff' },
    { name: 'Blue', hex: '#1e88e5', textColor: '#fff' },
    { name: 'Green', hex: '#43a047', textColor: '#fff' },
    { name: 'Yellow', hex: '#fdd835', textColor: '#000' },
    { name: 'Orange', hex: '#fb8c00', textColor: '#fff' },
    { name: 'Purple', hex: '#8e24aa', textColor: '#fff' },
  ];

  const MIN_INTERVAL = 0.5;
  const MAX_INTERVAL = 5;

  let number = $state(0);
  let currentColor = $state<typeof COLORS[0] | null>(null);
  let running = $state(false);
  let intervalSec = $state(2);
  let useRandomInterval = $state(false);
  let timeoutId: ReturnType<typeof setTimeout> | null = null;

  function generate() {
    number = Math.floor(Math.random() * 100) + 1;
    const colorIndex = Math.floor(Math.random() * COLORS.length);
    currentColor = COLORS[colorIndex];
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
      generate();
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
    generate();
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

  // Derive if number is odd or even for display hint
  let isOdd = $derived(number % 2 !== 0);
</script>

{#if running}
  <div
    class="fullscreen-cue"
    style:background-color={currentColor?.hex}
  >
    <span class="cue-number" style:color={currentColor?.textColor}>
      {number}
    </span>
    <div class="cue-hints" style:color={currentColor?.textColor}>
      <span class="hint">{currentColor?.name}</span>
      <span class="hint-dot">•</span>
      <span class="hint">{isOdd ? 'Odd' : 'Even'}</span>
    </div>
    <button class="stop-btn" onclick={toggle}>Stop</button>
  </div>
{:else}
  <div class="setup-screen">
    <div class="preview-area">
      <div class="preview-cue">
        <span class="preview-number">?</span>
      </div>
      <p class="preview-label">Number + Color</p>
    </div>

    <div class="settings-card">
      <h2>How it works</h2>
      <p class="description">
        Shows a random number (1-100) on a random color background.
        Use for complex drills like: "Blue = dribble, Odd = pass"
      </p>
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
  .fullscreen-cue {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 1000;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .cue-number {
    font-size: 45vw;
    font-weight: 800;
    line-height: 1;
    font-variant-numeric: tabular-nums;
  }

  .cue-hints {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    margin-top: 1rem;
    opacity: 0.7;
  }

  .hint {
    font-size: 1.25rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.1em;
  }

  .hint-dot {
    font-size: 0.75rem;
  }

  .stop-btn {
    position: absolute;
    bottom: 2rem;
    padding: 1rem 3rem;
    font-size: 1.25rem;
    font-weight: 600;
    background: rgba(0, 0, 0, 0.2);
    color: rgba(255, 255, 255, 0.9);
    border: 2px solid rgba(255, 255, 255, 0.3);
    border-radius: 50px;
    cursor: pointer;
    transition: all 0.2s;
    backdrop-filter: blur(4px);
    -webkit-backdrop-filter: blur(4px);
  }

  .stop-btn:hover {
    background: rgba(0, 0, 0, 0.35);
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
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 2rem;
    min-height: 180px;
  }

  .preview-cue {
    width: 120px;
    height: 120px;
    background: linear-gradient(135deg, #e53935 0%, #1e88e5 50%, #43a047 100%);
    border-radius: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .preview-number {
    font-size: 3rem;
    font-weight: 800;
    color: white;
  }

  .preview-label {
    color: #888;
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

  .description {
    font-size: 0.9rem;
    color: #555;
    line-height: 1.5;
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
