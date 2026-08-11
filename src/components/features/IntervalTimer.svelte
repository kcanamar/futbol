<script lang="ts">
  import { playBeep, initAudio } from '../../lib/audio';

  const PRESETS = [15, 30, 45, 60, 90, 120];
  const MIN_INTERVAL = 5;
  const MAX_INTERVAL = 600;
  const RING_RADIUS = 80;
  const RING_CIRCUMFERENCE = 2 * Math.PI * RING_RADIUS;

  let intervalSeconds = $state(30);
  let remainingSeconds = $state(30);
  let running = $state(false);
  let customInput = $state('');
  let audioInitialized = $state(false);
  let timerId: ReturnType<typeof setInterval> | null = null;

  function selectPreset(seconds: number) {
    if (running) return;
    intervalSeconds = seconds;
    remainingSeconds = seconds;
    customInput = '';
  }

  function applyCustom() {
    const value = parseInt(customInput, 10);
    if (isNaN(value) || value < MIN_INTERVAL || value > MAX_INTERVAL) return;
    intervalSeconds = value;
    remainingSeconds = value;
  }

  function formatTime(seconds: number): string {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins}:${secs.toString().padStart(2, '0')}`;
  }

  function tick() {
    remainingSeconds--;
    if (remainingSeconds <= 0) {
      playBeep(880, 300);
      remainingSeconds = intervalSeconds; // auto-reset for next interval
    }
  }

  function start() {
    if (!audioInitialized) {
      initAudio();
      audioInitialized = true;
    }
    running = true;
    timerId = setInterval(tick, 1000);
  }

  function pause() {
    running = false;
    if (timerId) {
      clearInterval(timerId);
      timerId = null;
    }
  }

  function reset() {
    pause();
    remainingSeconds = intervalSeconds;
  }

  function testBeep() {
    if (!audioInitialized) {
      initAudio();
      audioInitialized = true;
    }
    playBeep(880, 300);
  }

  $effect(() => {
    return () => {
      if (timerId) clearInterval(timerId);
    };
  });

  // Progress calculation
  let progress = $derived(1 - remainingSeconds / intervalSeconds);
  let strokeDashoffset = $derived(RING_CIRCUMFERENCE * (1 - progress));
</script>

<div class="interval-timer">
  <div class="timer-display">
    <svg class="timer-ring" width="200" height="200" viewBox="0 0 200 200">
      <circle class="timer-ring-bg" cx="100" cy="100" r={RING_RADIUS} />
      <circle
        class="timer-ring-progress"
        cx="100"
        cy="100"
        r={RING_RADIUS}
        stroke-dasharray={RING_CIRCUMFERENCE}
        stroke-dashoffset={strokeDashoffset}
      />
    </svg>
    <span class="timer-text">{formatTime(remainingSeconds)}</span>
  </div>

  <p class="status" class:running>
    {running ? 'Running' : 'Paused'} — {intervalSeconds}s interval
  </p>

  <div class="preset-grid">
    {#each PRESETS as preset}
      <button
        class="preset-btn"
        class:active={intervalSeconds === preset && !running}
        onclick={() => selectPreset(preset)}
        disabled={running}
      >
        {preset}s
      </button>
    {/each}
  </div>

  <div class="input-group">
    <input
      type="number"
      bind:value={customInput}
      placeholder="Custom (5-600s)"
      min={MIN_INTERVAL}
      max={MAX_INTERVAL}
      disabled={running}
    />
    <button class="btn btn-secondary" onclick={applyCustom} disabled={running}>
      Set
    </button>
  </div>

  <div class="controls">
    {#if running}
      <button class="btn btn-large btn-secondary" onclick={pause}>Pause</button>
    {:else}
      <button class="btn btn-large btn-primary" onclick={start}>Start</button>
    {/if}
    <button class="btn btn-large btn-secondary" onclick={reset}>Reset</button>
  </div>

  <button class="test-beep-btn" onclick={testBeep}>
    Test Beep
  </button>
</div>

<style>
  .test-beep-btn {
    display: block;
    margin: var(--space-lg) auto 0;
    padding: var(--space-sm) var(--space-md);
    background: none;
    border: 1px solid var(--color-border);
    border-radius: var(--border-radius);
    color: var(--color-text-muted);
    font-size: var(--font-size-sm);
    cursor: pointer;
  }

  .test-beep-btn:hover {
    border-color: var(--color-primary);
    color: var(--color-primary);
  }
</style>
