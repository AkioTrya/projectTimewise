<script setup>
import { ref, computed } from 'vue'

let intervalid = null
const startTimer = () => {
  if (!isRunning.value) {
    isRunning.value = true
    intervalid = setInterval(tick, 1000)
  }
}
const minutes = ref(25) //25 minutes default starting point
const seconds = ref(0) //starting the seconds at 0 
const isRunning = ref(false) //checks if the timer whether it's active or not
const isPaused = ref(false) //checks if the timer is paused or not
const isWorkMode = ref(true); //checks if the timer is in work mode or not
const workDuration = ref(25); //default work mode timer
const shortBreakDuration = ref(5); // default short break timer
const longBreakDuration = ref(15); //default long break timer
const workSessionCompleted = ref(0); //default work session completed counter
const workSessionsCompleted = ref(0); //default number of work session completed counter
const sessionHistory = ref([]); //default session history
const todayHistory = ref(0); // default today's history

const loadSessions = () => {
  const stored = localStorage.getItem('timewise_sessions');
  if (stored) {
    sessionHistory.value = JSON.parse(stored);
    updateTodayCount();
  }
};

const updateTodayCount = () => {
  const today = new Date().toDateString();
  todayHistory.value = sessionHistory.value.filter(session =>
    new Date(session.timestamp).toDateString() === today
  ).length;
};
loadSessions();

const saveWorkSession = () => {
  const newSession = {
    timestamp: new Date().toISOString(),
    type: 'work'
  };
  sessionHistory.value.push(newSession);
  localStorage.setItem('timewise_sessions', JSON.stringify(sessionHistory.value));
  updateTodayCount();
};

const formattedTime = computed(() => {
  const mins = minutes.value.toString().padStart(2, '0');
  const secs = seconds.value.toString().padStart(2, '0');
  return `${mins}:${secs}`;
})

const tick = () => {
  if (seconds.value > 0) {
    seconds.value--;
  } else if (minutes.value > 0) {
    minutes.value--;
    seconds.value = 59;
  } else {
    isRunning.value = false; //timer finished here
    if (intervalid) {
      clearInterval(intervalid);
      intervalid = null;
    }
    if (isWorkMode.value) {
      saveWorkSession();
      workSessionCompleted.value++;
      if(workSessionCompleted.value % 4 === 0) {
        isWorkMode.value = false;
        minutes.value = longBreakDuration.value;
        seconds.value = 0;
      } else {
        isWorkMode.value = false;
        minutes.value = shortBreakDuration.value;
        seconds.value = 0;
      }
      startTimer();
    } else {
      isWorkMode.value = true;
      minutes.value = workDuration.value;
      seconds.value = 0;
      isPaused.value = false;
    }
  }
}
const pauseTimer = () => {
  if (isRunning.value) {
    clearInterval(intervalid);
    isRunning.value = false;
    isPaused.value = true;
    intervalid = null;
  }
}

const resetTimer = () => {
  if (intervalid) {
    clearInterval(intervalid);
    intervalid = null;
  }
  if (isWorkMode.value) {
    minutes.value = workDuration;
  } else {
    if (workSessionCompleted.value % 4 == 0 && workSessionCompleted.value !== 0) {
      minutes.value = longBreakDuration;
    } else {
      minutes.value = shortBreakDuration;
    }
  }   
  seconds.value = 0;
  isRunning.value = false;
  isPaused.value = false;
} 
const addFiveMinutes = () => {
  minutes.value += 5;
}
const resetSession = () => {
  workSessionCompleted.value = 0;
  resetTimer(); //also resets timer to work mode
}

const isDarkMode = ref(false);

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value;
  localStorage.setItem('timewise_theme', isDarkMode.value ? 'dark' : 'light');
  applyTheme();
}

const applyTheme = ()=> {
  const root = document.documentElement;
  if (isDarkMode.value) {
  root.style.setProperty('--bg', '#1e1e2f');
    root.style.setProperty('--text', '#ffffff');
    root.style.setProperty('--card-bg', '#2d2d44');
    root.style.setProperty('--button-bg', '#3a3a5a');
    root.style.setProperty('--button-hover', '#4a4a6e');
  } else {
    root.style.setProperty('--bg', '#f5f5f7');
    root.style.setProperty('--text', '#222222');
    root.style.setProperty('--card-bg', '#ffffff');
    root.style.setProperty('--button-bg', '#e0e0e6');
    root.style.setProperty('--button-hover', '#ccccd6');
  };
}

// Load saved theme on startup
const savedTheme = localStorage.getItem('timewise_theme');
if (savedTheme === 'dark') {
  isDarkMode.value = true;
  applyTheme();
} else {
  applyTheme(); // apply default light
}
</script>

<template>
  <div class="app">
    <button class="theme-toggle" @click="toggleTheme">
      {{ isDarkMode ? '☀️' : '🌙' }}
    </button>
    <h1>🍅 Timewise</h1>
    <div class="mode">
      <span v-if="isWorkMode">💪 Work Time ({{ workSessionCompleted }} sessions)</span>
      <span v-else>
        ☕ 
        <span v-if="workSessionCompleted % 4 === 0 && workSessionCompleted !== 0">Long Break</span>
        <span v-else>Short Break</span>
      </span>
    </div>
    <div class="timer-card">
      <div class="timer-display">{{ formattedTime }}</div>
      <div class="buttons">
        <button @click="startTimer">Start</button>
        <button @click="pauseTimer">Pause</button>
        <button @click="resetTimer">Reset</button>
        <button @click="addFiveMinutes">+5 min</button>
        <button @click="resetSession">Reset sessions</button>
      </div>
      <div class="stats">Today: {{ todaySessions }} pomodoro(s)</div>
      <div class="status">
        <span v-if="isPaused">Paused</span>
        <span v-else-if="isRunning">Focusing...</span>
        <span v-else>Ready</span>
      </div>
    </div>
  </div>
</template>

<style>
/* global variables (applies to whole page) */
:root {
  --bg: #f5f5f7;
  --text: #222222;
  --card-bg: #ffffff;
  --button-bg: #e0e0e6;
  --button-hover: #ccccd6;
  --accent: #4caf50;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background-color: var(--bg);
  color: var(--text);
  font-family: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
  transition: background 0.3s, color 0.2s;
}

.app {
  max-width: 600px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
  min-height: 100vh;
}

.timer-card {
  background: var(--card-bg);
  border-radius: 2rem;
  padding: 2rem;
  margin: 2rem 0;
  box-shadow: 0 8px 20px rgba(0,0,0,0.1);
  transition: background 0.3s;
}

.timer-display {
  font-size: 5rem;
  font-weight: bold;
  font-family: monospace;
  letter-spacing: 4px;
  margin: 1rem 0;
}

.buttons {
  display: flex;
  gap: 0.8rem;
  justify-content: center;
  flex-wrap: wrap;
  margin: 1.5rem 0;
}

button {
  background: var(--button-bg);
  border: none;
  padding: 0.6rem 1.2rem;
  border-radius: 2rem;
  font-size: 1rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  color: var(--text);
}
button:hover {
  background: var(--button-hover);
  transform: scale(1.02);
}
button:active {
  transform: scale(0.98);
}

.mode {
  font-size: 1.2rem;
  font-weight: bold;
  margin-bottom: 1rem;
}

.status {
  margin-top: 1rem;
  font-style: italic;
  opacity: 0.8;
}

.stats {
  margin-top: 1rem;
  font-size: 1rem;
}

.theme-toggle {
  position: fixed;
  top: 1rem;
  right: 1rem;
  background: var(--button-bg);
  border-radius: 2rem;
  padding: 0.4rem 1rem;
  font-size: 1.2rem;
}
</style>
