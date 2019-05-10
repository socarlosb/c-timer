<script>
  let timerDisplay = "25:00";
  let timerLeftDisplay = "-";
  let isPause = true;
  let initialTime = 25 * 60;
  let countdown;
  let secondsLeft = 0;

  function timer(seconds) {
    const now = Date.now();
    const then = now + seconds * 1000;
    displayTimeLeft(seconds);
    displayEndTime(then);

    countdown = setInterval(() => {
      secondsLeft = Math.round((then - Date.now()) / 1000);
      if (!isPause) {
        if (secondsLeft < 0) {
          clearInterval(countdown);
          secondsLeft = 0
          return;
        }

        displayTimeLeft(secondsLeft);
      }
    }, 1000);
  }

  function displayTimeLeft(seconds) {
    const minutes = Math.floor(seconds / 60);
    const remainSeconds = seconds % 60;
    const display = `${minutes}:${
      remainSeconds < 10 ? "0" : ""
    }${remainSeconds}`;
    timerDisplay = display;
    document.title = `Time left: ${display}`;
  }

  function displayEndTime(timestamp) {
    const end = new Date(timestamp);
    const hour = end.getHours();
    const minutes = end.getMinutes();
    timerLeftDisplay = `Ends at ${hour}:${
      minutes < 10 ? "0" : ""
    }${minutes}`;
  }

  const setState = type => {
    switch (type) {
      case "stop":
        isPause = true;
        const end = new Date();
        const hour = end.getHours();
        const minutes = end.getMinutes();
        const display = `Timer stoped ${hour}:${
          minutes < 10 ? "0" : ""
        }${minutes}!`;
        clearInterval(countdown);

        document.title = display;
        timerLeftDisplay = display;
        break;

      case "start":
        clearInterval(countdown);
        
        isPause = false;
        if (secondsLeft) {
          timer(secondsLeft);
        } else {
          timer(initialTime);
        }
        break;

      case "restart":
        clearInterval(countdown);
        secondsLeft = 0;
        timerDisplay = "25:00";
        timerLeftDisplay = "-";
        document.title = "Timer";
        break;

      default:
        throw new Error("No type found!");
    }
  };
</script>

<style>
  body {
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .container {
    min-width: 300px;
    background-color: #3174aa;
    color: #fff;
    padding: 1em;
    border-radius: 5px;
  }
  .container i{
    color: #fff; 
  }
  .time {
    text-align: center;
    font-size: 4em;
  }
  .time--left{
    text-align: center;
    font-size: 2em;

  }
  .controls {
    display: flex;
    justify-content: space-around;
  }
  button {
    padding: 1em 2em;
    border-radius: 5px;
    background: none;
    border: #fff solid 1px;
    cursor: pointer;
  }
  button:disabled{
    background-color: rgba(157, 202, 238, 0.5);
    cursor: inherit;
  }
</style>

<body>
  <div class="container">
    <h2 class="time">{timerDisplay}</h2>
    <h2 class="time--left">{timerLeftDisplay}</h2>
    <div class="controls">
      <button on:click={() => setState('start')} disabled={!isPause}>
        <i class="fas fa-play" />
      </button>
      <button on:click={() => setState('stop')} disabled={isPause}>
        <i class="fas fa-pause" />
      </button>
      <button on:click={() => setState('restart')} disabled={secondsLeft === 0}>
        <i class="fas fa-redo" />
      </button>
    </div>
  </div>
</body>
