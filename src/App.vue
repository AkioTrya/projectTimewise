<script setup>
import { ref, computed } from 'vue'

const currentPage = ref('timer');

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
    root.style.setProperty('--border', '#3a3a5a');
  } else {
    root.style.setProperty('--bg', '#f5f5f7');
    root.style.setProperty('--text', '#222222');
    root.style.setProperty('--card-bg', '#ffffff');
    root.style.setProperty('--button-bg', '#e0e0e6');
    root.style.setProperty('--button-hover', '#ccccd6');
    root.style.setProperty('--border', '#e0e0e6');
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
  <div class="layout">
    <aside class="sidebar">
      <div class="logo-area">🍅 Timewise</div>
      <nav>
        <button :class="{ active: currentPage === 'timer' }" @click="currentPage = 'timer'">Timer</button>
        <button :class="{ active: currentPage === 'tasks' }" @click="currentPage = 'tasks'">Tasks</button>
        <button :class="{ active: currentPage === 'history' }" @click="currentPage = 'history'">History</button>
      </nav>
      <div class="sidebar-footer">
        <button class="theme-toggle" @click="toggleTheme">
          {{ isDarkMode ? '☀️ Light Mode' : '🌙 Dark Mode' }}
        </button>
      </div>
    </aside>

    <main class="main-content">
      <div class="center-container">
        
        <div v-if="currentPage === 'timer'" class="view-timer">
          <header class="top-nav">
            <div class="nav-logo">🍅 Timewise</div>
            <button class="icon-btn">⚙️ Settings</button>
          </header>
          
          <div class="timer-center-wrapper">
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
              <div class="stats">Today: {{ todayHistory }} pomodoro(s)</div>
              <div class="status">
                <span v-if="isPaused">Paused</span>
                <span v-else-if="isRunning">Focusing...</span>
                <span v-else>Ready</span>
              </div>
            </div>
          </div>
        </div>

        <div v-if="currentPage === 'tasks'" class="view-tasks">
          <h2>Tasks</h2>
          <div class="tasks-layout">
            <div class="calendar-column">
              <h3>Calendar</h3>
              <div class="placeholder-box">Calendar View Placeholder</div>
            </div>
            <div class="task-list-column">
              <h3>To Do</h3>
              <div class="placeholder-box">Task List Placeholder</div>
              <div class="placeholder-box">Task List Placeholder</div>
              <div class="placeholder-box">Task List Placeholder</div>
            </div>
          </div>
        </div>

        <div v-if="currentPage === 'history'" class="view-history">
          <h2>Session History</h2>
          <div class="history-grid">
            <div class="history-card" v-for="session in sessionHistory.slice().reverse()" :key="session.timestamp">
              <div class="type">{{ session.type === 'work' ? '💪 Work Session' : '☕ Break' }}</div>
              <div class="time">{{ new Date(session.timestamp).toLocaleString() }}</div>
            </div>
            <div v-if="sessionHistory.length === 0" class="empty-state">No history yet.</div>
          </div>
        </div>

      </div>
    </main>
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
  --border: #e0e0e6;
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
  overflow: hidden; /* we manage scroll in .main-content */
}

.layout {
  display: flex;
  height: 100vh;
  width: 100vw;
  background-color: var(--bg);
  color: var(--text);
}

.sidebar {
  width: 260px;
  background: var(--card-bg);
  border-right: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  padding: 2rem 1.5rem;
}

.logo-area {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 3rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.sidebar nav {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  flex: 1;
}

.sidebar nav button {
  text-align: left;
  background: transparent;
  color: var(--text);
  padding: 0.8rem 1.2rem;
  border-radius: 0.5rem;
  font-weight: 500;
  border: 1px solid transparent;
}

.sidebar nav button:hover {
  background: var(--button-hover);
}

.sidebar nav button.active {
  background: var(--button-bg);
  font-weight: bold;
  border: 1px solid var(--border);
}

.sidebar-footer {
  margin-top: auto;
}

.theme-toggle {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.main-content {
  flex: 1;
  overflow-y: auto;
  position: relative;
}

.center-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2.5rem;
  min-height: 100%;
  display: flex;
  flex-direction: column;
}

/* Timer View */
.view-timer {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.top-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 2rem;
  border-bottom: 1px solid var(--border);
  margin-bottom: 2rem;
}

.nav-logo {
  font-size: 1.2rem;
  font-weight: 600;
}

.icon-btn {
  background: transparent;
  border: 1px solid var(--border);
}

.timer-center-wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.timer-card {
  width: 100%;
  max-width: 480px;
  background: var(--card-bg);
  border-radius: 2rem;
  padding: 3rem;
  margin: 2rem 0;
  box-shadow: 0 8px 30px rgba(0,0,0,0.06);
  transition: background 0.3s;
  text-align: center;
  border: 1px solid var(--border);
}

.timer-display {
  font-size: 6rem;
  font-weight: bold;
  font-family: monospace;
  letter-spacing: 4px;
  margin: 1.5rem 0;
}

.buttons {
  display: flex;
  gap: 0.8rem;
  justify-content: center;
  flex-wrap: wrap;
  margin: 2rem 0;
}

button {
  background: var(--button-bg);
  border: none;
  padding: 0.7rem 1.4rem;
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
  font-size: 1.5rem;
  font-weight: bold;
  text-align: center;
}

.status {
  margin-top: 1rem;
  font-style: italic;
  opacity: 0.8;
}

.stats {
  margin-top: 1.5rem;
  font-size: 1.1rem;
  opacity: 0.9;
}

/* Tasks Layout */
.view-tasks h2, .view-history h2 {
  margin-bottom: 2rem;
  font-size: 2rem;
}

.tasks-layout {
  display: flex;
  gap: 3rem;
  height: 100%;
}
.calendar-column {
  flex: 0 0 30%;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}
.task-list-column {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

/* History Layout */
.history-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 1.5rem;
}

.history-card {
  background: var(--card-bg);
  padding: 1.5rem;
  border-radius: 1rem;
  border: 1px solid var(--border);
  box-shadow: 0 4px 10px rgba(0,0,0,0.02);
}

.history-card .type {
  font-weight: bold;
  font-size: 1.1rem;
  margin-bottom: 0.5rem;
}

.history-card .time {
  font-size: 0.95rem;
  opacity: 0.7;
}

.placeholder-box {
  background: var(--card-bg);
  padding: 2rem;
  border-radius: 1rem;
  opacity: 0.7;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 120px;
  border: 1px dashed var(--border);
}

.empty-state {
  grid-column: 1 / -1;
  text-align: center;
  padding: 4rem;
  opacity: 0.6;
  font-size: 1.2rem;
}
</style>
