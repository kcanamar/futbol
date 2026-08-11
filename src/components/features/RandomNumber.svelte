<script lang="ts">
  let number = $state(0);
  let running = $state(false);
  let intervalId: ReturnType<typeof setInterval> | null = null;

  function generateNumber() {
    number = Math.floor(Math.random() * 100) + 1;
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
    intervalId = setInterval(generateNumber, 2000);
  }

  function stop() {
    running = false;
    if (intervalId) {
      clearInterval(intervalId);
      intervalId = null;
    }
  }
</script>

<div class="random-number">
  <div class="display-box">
    <span class="display-number">{number || '—'}</span>
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
