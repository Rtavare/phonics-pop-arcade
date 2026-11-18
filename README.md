# 🎈 Phonics Pop! Arcade (Vue 3 + Vite)

An interactive, cheerful phonics learning course and bubble-popping arcade game built with **Vue 3** and **Vite** for early readers.

---

## ✨ Features

* **Learn A–Z Mode:** Step through all 26 letters with uppercase and lowercase displays, picture badges, a live soundwave speaker avatar, and slow, clear pronunciation that says each sound twice with a pause before speaking the example word.
* **Pop Game! (Arcade Mode):** Pop floating letter bubbles matching the spoken phonetic sound across selectable letter stages (`A–F`, `G–L`, `M–R`, `S–Z`, or `A–Z`), complete with a toggleable picture clue, star streaks, synthesized Web Audio sound effects, and confetti celebrations.
* **Smart Neural Voice Ranking & Picker:** Automatically prioritizes warm, natural browser voices and includes a voice dropdown selector.

---

## 📚 A–Z Early Reader Phonics Reference

| Letter | Sound | Example Word |
| :--- | :--- | :--- |
| **A a** | /ah/ | Apple 🍎 |
| **B b** | /buh/ | Ball ⚽ |
| **C c** | /kuh/ | Cat 🐱 |
| **D d** | /duh/ | Dog 🐶 |
| **E e** | /eh/ | Egg 🥚 |
| **F f** | /fff/ | Fish 🐟 |
| **G g** | /guh/ | Goat 🐐 |
| **H h** | /hhh/ | Hat 🎩 |
| **I i** | /ih/ | Insect 🐛 |
| **J j** | /juh/ | Jar 🫙 |
| **K k** | /kuh/ | Kite 🪁 |
| **L l** | /lll/ | Lemon 🍋 |
| **M m** | /mmm/ | Moon 🌙 |
| **N n** | /nnn/ | Nest 🪺 |
| **O o** | /ah/ | Octopus 🐙 |
| **P p** | /puh/ | Pig 🐷 |
| **Q q** | /kwuh/ | Queen 👑 |
| **R r** | /rrr/ | Rainbow 🌈 |
| **S s** | /sss/ | Sun ☀️ |
| **T t** | /tuh/ | Turtle 🐢 |
| **U u** | /uh/ | Umbrella ☂️ |
| **V v** | /vvv/ | Violin 🎻 |
| **W w** | /wuh/ | Whale 🐋 |
| **X x** | /ks/ | Box 📦 |
| **Y y** | /yuh/ | Yo-yo 🪀 |
| **Z z** | /zzz/ | Zebra 🦓 |

---

## 🚀 Local Setup & Installation

```bash
npm install
npm run dev
```

Open the URL shown in your terminal (typically `http://localhost:5173`) to launch the game.

Build for production:

```bash
npm run build
```

Project structure:

- `src/App.vue` — the full Phonics Pop component (dataset, Learn mode, Arcade mode, speech, SFX, confetti)
- `src/main.js` — Vue entry point
- `src/style.css` — Tailwind directives
- `tailwind.config.js` / `postcss.config.js` — Tailwind configuration
- `public/audio/` — pre-generated voice clips (`sound-a.mp3` … `sound-z.mp3`, `word-a.mp3` … `word-z.mp3`)

## 🔊 Voice / Audio

The voice does **not** rely on the browser's `speechSynthesis`, which is silent on some phones and tablets. Each phoneme and example word is a bundled MP3 clip played with a plain Audio element after a tap.

Live TTS API hook: in `src/App.vue`, set the `TTS_API_URL` constant to a proxy endpoint that accepts `?text=...` and returns an MP3 (for example a Cloudflare Worker or serverless function holding your OpenAI, ElevenLabs, or Google TTS key — never put a raw API key in the frontend, this site is public). When set, clips are fetched live from that API instead of the bundled files. If a clip fails to load, the app falls back to browser `speechSynthesis`.
