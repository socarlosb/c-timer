<script>
  import Switch from "./Switch.svelte";
  let timerDisplay = "25:00";
  let timerLeftDisplay = "-";
  let isPause = true;
  let initialTime = 25 * 60;
  let countdown;
  let secondsLeft = 0;
  const audio = new Audio("./sound/this-guitar.mp3");
  const vibrate =
    navigator.vibrate ||
    navigator.webkitVibrate ||
    navigator.mozVibrate ||
    navigator.msVibrate;

  let isSound = true;
  let isVibrate = false;

  let currentTask = "";
  let taskList = [];

  function addTask(event) {
    if (currentTask.length > 0 && event.which === 13) {
      taskList = [...taskList, currentTask];
      currentTask = "";
    }
  }

  function endCounter() {
    if (isSound) {
      audio.play();
    }

    if (isVibrate) {
      navigator.vibrate(500);
    }
  }
  function timer(seconds) {
    const now = Date.now();
    const then = now + seconds * 1000;
    displayTimeLeft(seconds);
    displayEndTime(then);

    countdown = setInterval(() => {
      secondsLeft = Math.round((then - Date.now()) / 1000);
      if (!isPause) {
        if (secondsLeft < 0) {
          endCounter();
          clearInterval(countdown);
          secondsLeft = 0;
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
    timerLeftDisplay = `Ends at ${hour}:${minutes < 10 ? "0" : ""}${minutes}`;
  }
  function handleToggle(toggle) {
    if (toggle.name === "isSound") {
      isSound = !toggle.val;
      if (isSound) {
        audio.play();
      }
    } else if (toggle.name === "isVibrate") {
      isVibrate = !toggle.val;
      if (isVibrate) {
        // console.log('is vibrating')
        navigator.vibrate(500);
      }
    } else console.log("not found");
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
        isPause = true;
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
  .container i {
    color: #fff;
  }
  .time {
    text-align: center;
    font-size: 4em;
  }
  .time--left {
    text-align: center;
  }
  .controls {
    display: flex;
    justify-content: space-around;
    margin-bottom: 1.5em;
  }
  button {
    padding: 1em 2em;
    border-radius: 5px;
    background: none;
    border: #fff solid 1px;
    cursor: pointer;
  }
  button:disabled {
    background-color: rgba(157, 202, 238, 0.5);
    cursor: inherit;
  }
  .task {
    width: 100%;
    background: none;
    padding: 0.5em;
    border-radius: 5px;
    border: #fff solid 1px;
    color: #fff;
    text-align: center;
    font-size: 1em;
    font-family: "Viga", sans-serif;
  }
  .task__list {
    border-radius: 5px;
    border: #fff solid 1px;
    font-size: 1.2em;
  }
</style>

<body>
  <div class="container">
    <h2 class="time">{timerDisplay}</h2>
    <input
      class="task"
      type="text"
      bind:value={currentTask}
      on:keyup={addTask} />
    <h3 class="time--left">{timerLeftDisplay}</h3>
    <div class="controls">
      <Switch
        toggle={isSound}
        change={() => handleToggle({ name: 'isSound', val: isSound })}>
        Sound
      </Switch>

      {#if vibrate}
        <Switch
          toggle={isVibrate}
          change={() => handleToggle({ name: 'isVibrate', val: isVibrate })}>
          Vibrate
        </Switch>
      {/if}
    </div>
    <div class="controls">
      <button on:click={() => setState('start')} disabled={!isPause}>
        <i class="fas fa-play" />
      </button>
      <button on:click={() => setState('stop')} disabled={isPause}>
        <i class="fas fa-pause" />
      </button>
      <button
        on:click={() => setState('restart')}
        disabled={secondsLeft !== 0 && isPause === false}>
        <i class="fas fa-redo" />
      </button>
    </div>
    <div class="task__list">
      <ul>
        {#each taskList as task}
          <li>{task}</li>
        {/each}
      </ul>
    </div>
  </div>
</body>
