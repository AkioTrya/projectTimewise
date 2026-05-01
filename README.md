# 🍅 projectTimewise – Pomodoro Timer with Personality

A clean, focused Pomodoro timer built with **Vue 3** and **Vite**.  
No paywalls, no ads, no data selling – just a timer that respects your time.

> **Live demo:** [https://projecttimewise.netlify.app](https://projecttimewise.netlify.app) *(replace with your actual link after deploy)*

---

## ✨ Features (v1.0)

- ✅ **Pomodoro timer** – 25 min work / 5 min short break / 15 min long break (after 4 sessions)
- ✅ **Start, Pause, Reset, +5 min**
- ✅ **Session tracking** – Today's pomodoros saved in `localStorage`
- ✅ **Dark / Light mode** – Toggle, preference remembered
- ✅ **Status messages** – "Focusing…", "Paused", "Ready"
- ✅ **Responsive UI** – Works on desktop & mobile
- ❌ **Virtual pet** – Planned for v1.1 (walking companion with encouragement)
- ❌ **Sound notifications** – Planned for v1.1

---

## 🛠️ Tech Stack

| Layer       | Technology |
|-------------|------------|
| Frontend    | Vue 3 (Composition API), Vite |
| Styling     | CSS custom properties (variables) |
| State       | `ref`, `computed`, `localStorage` |
| Deployment  | Static hosting (Netlify / Vercel) |

> No backend yet – all data stays on your device.

---

## 🚀 Getting Started (local)

```bash
git clone https://github.com/yourusername/projectTimewise.git
cd projectTimewise
npm install
npm run dev
