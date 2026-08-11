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

  let selectedIndices = $state(new Set([0, 1])); // Red and Blue default
  let currentColor = $state<string | null>(null);
  let running = $state(false);
  let intervalId: ReturnType<typeof setInterval> | null = null;

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
    intervalId = setInterval(generateColor, 2000);
  }

  function stop() {
    running = false;
    if (intervalId) {
      clearInterval(intervalId);
      intervalId = null;
    }
  }

  function isDisabled(index: number): boolean {
    if (running) return true;
    if (selectedIndices.has(index)) {
      return selectedIndices.size <= MIN_COLORS;
    }
    return selectedIndices.size >= MAX_COLORS;
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
    {running ? 'Running - 2s interval' : 'Stopped'}
  </p>

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
</style>
