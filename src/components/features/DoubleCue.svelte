<script lang="ts">
  const COLORS = [
    { name: 'Red', hex: '#e53935' },
    { name: 'Blue', hex: '#1e88e5' },
    { name: 'Green', hex: '#43a047' },
    { name: 'Yellow', hex: '#fdd835' },
    { name: 'Orange', hex: '#fb8c00' },
    { name: 'Purple', hex: '#8e24aa' },
  ];

  const MIN_COLORS = 2;
  const MAX_COLORS = 6;
  const MIN_INTERVAL = 0.5;
  const MAX_INTERVAL = 5;
  const MIN_FLASH = 0.2;
  const MAX_FLASH = 2;

  let selectedIndices = $state(new Set([0, 1, 2, 3]));
  let number = $state(0);
  let currentColor = $state<typeof COLORS[0] | null>(null);
  let textColor = $state<typeof COLORS[0] | null>(null);
  let showNumber = $state(true);
  let visible = $state(true);
  let running = $state(false);
  let intervalSec = $state(2);
  let useRandomInterval = $state(false);
  let flashMode = $state(false);
  let flashDuration = $state(0.5);
  let timeoutId: ReturnType<typeof setTimeout> | null = null;
  let flashTimeoutId: ReturnType<typeof setTimeout> | null = null;

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

  function isSelected(index: number): boolean {
    return selectedIndices.has(index);
  }

  function getContrastingColor(bgIndex: number): typeof COLORS[0] {
    const indices = Array.from(selectedIndices);
    const otherIndices = indices.filter(i => i !== bgIndex);
    const textIndex = otherIndices[Math.floor(Math.random() * otherIndices.length)];
    return COLORS[textIndex];
  }

  function generate() {
    number = Math.floor(Math.random() * 100) + 1;
    const indices = Array.from(selectedIndices);
    const randomIdx = indices[Math.floor(Math.random() * indices.length)];
    currentColor = COLORS[randomIdx];
    showNumber = Math.random() < 0.5;
    textColor = showNumber ? null : getContrastingColor(randomIdx);
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
  <div
    class="fullscreen-cue"
    style:background-color={visible ? currentColor?.hex : '#1a1a1a'}
  >
    {#if visible}
      {#if showNumber}
        <span class="cue-number" style:color="#fff">
          {number}
        </span>
      {:else}
        <span class="cue-word" style:color={textColor?.hex}>
          {currentColor?.name}
        </span>
      {/if}
    {:else}
      <span class="blank-indicator">—</span>
    {/if}
    <button class="stop-btn" onclick={toggle}>Stop</button>
  </div>
{:else}
  <div class="setup-screen">
    <div class="preview-area">
      <div class="preview-cue">
        <span class="preview-number">42</span>
      </div>
      <span class="preview-or">or</span>
      <div class="preview-cue preview-word">
        <span class="preview-text">RED</span>
      </div>
    </div>

    <div class="settings-card">
      <h2>How it works</h2>
      <p class="description">
        Randomly shows either a number (1-100) or a color word on a colored background.
        Color words appear in a contrasting color for extra cognitive challenge.
      </p>
    </div>

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

    <div class="settings-card">
      <h2>Flash Mode</h2>

      <label class="toggle-row">
        <span>Brief flash only</span>
        <input type="checkbox" bind:checked={flashMode} />
      </label>

      {#if flashMode}
        <p class="flash-description">Cue disappears after flash duration</p>
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
    transition: background-color 0.05s;
  }

  .cue-number {
    font-size: 45vw;
    font-weight: 800;
    line-height: 1;
    font-variant-numeric: tabular-nums;
  }

  .cue-word {
    font-size: 18vw;
    font-weight: 800;
    line-height: 1;
    text-transform: uppercase;
    letter-spacing: 0.05em;
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
    align-items: center;
    justify-content: center;
    gap: 1rem;
    padding: 1.5rem;
    min-height: 100px;
  }

  .preview-cue {
    width: 70px;
    height: 70px;
    background: #8e24aa;
    border-radius: 14px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .preview-cue.preview-word {
    background: #1e88e5;
  }

  .preview-number {
    font-size: 1.75rem;
    font-weight: 800;
    color: white;
  }

  .preview-text {
    font-size: 0.9rem;
    font-weight: 800;
    color: #fdd835;
  }

  .preview-or {
    color: #999;
    font-size: 0.875rem;
    font-style: italic;
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

  .selection-count {
    font-size: 0.8rem;
    color: #999;
    margin-bottom: 1rem;
  }

  .color-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
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
    margin-top: auto;
  }

  .start-btn:hover {
    background: var(--color-primary-dark);
  }

  .start-btn:active {
    transform: scale(0.98);
  }
</style>
