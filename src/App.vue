<script setup>
import { ref, reactive, computed, onMounted, onUnmounted, nextTick } from 'vue';

/*
 * Complete A–Z Phonics Dataset matching the early-reader curriculum.
 * Includes phonetic pronunciation strings tuned so TTS engines speak the pure
 * phoneme cleanly instead of spelling out letters.
 */
const PHONICS_DATA = [
  { upper: 'A', lower: 'a', sound: 'ah',   speechSound: 'aah.',  word: 'apple',    emoji: '🍎', group: 'AF' },
  { upper: 'B', lower: 'b', sound: 'buh',  speechSound: 'buh.',  word: 'ball',     emoji: '⚽', group: 'AF' },
  { upper: 'C', lower: 'c', sound: 'kuh',  speechSound: 'kuh.',  word: 'cat',      emoji: '🐱', group: 'AF' },
  { upper: 'D', lower: 'd', sound: 'duh',  speechSound: 'duh.',  word: 'dog',      emoji: '🐶', group: 'AF' },
  { upper: 'E', lower: 'e', sound: 'eh',   speechSound: 'eh.',   word: 'egg',      emoji: '🥚', group: 'AF' },
  { upper: 'F', lower: 'f', sound: 'fff',  speechSound: 'ffff.', word: 'fish',     emoji: '🐟', group: 'AF' },
  { upper: 'G', lower: 'g', sound: 'guh',  speechSound: 'guh.',  word: 'goat',     emoji: '🐐', group: 'GL' },
  { upper: 'H', lower: 'h', sound: 'hhh',  speechSound: 'huh.',  word: 'hat',      emoji: '🎩', group: 'GL' },
  { upper: 'I', lower: 'i', sound: 'ih',   speechSound: 'ih.',   word: 'insect',   emoji: '🐛', group: 'GL' },
  { upper: 'J', lower: 'j', sound: 'juh',  speechSound: 'juh.',  word: 'jar',      emoji: '🫙', group: 'GL' },
  { upper: 'K', lower: 'k', sound: 'kuh',  speechSound: 'kuh.',  word: 'kite',     emoji: '🪁', group: 'GL' },
  { upper: 'L', lower: 'l', sound: 'lll',  speechSound: 'llll.', word: 'lemon',    emoji: '🍋', group: 'GL' },
  { upper: 'M', lower: 'm', sound: 'mmm',  speechSound: 'mmmm.', word: 'moon',     emoji: '🌙', group: 'MR' },
  { upper: 'N', lower: 'n', sound: 'nnn',  speechSound: 'nnnn.', word: 'nest',     emoji: '🪺', group: 'MR' },
  { upper: 'O', lower: 'o', sound: 'ah',   speechSound: 'aww.',  word: 'octopus',  emoji: '🐙', group: 'MR' },
  { upper: 'P', lower: 'p', sound: 'puh',  speechSound: 'puh.',  word: 'pig',      emoji: '🐷', group: 'MR' },
  { upper: 'Q', lower: 'q', sound: 'kwuh', speechSound: 'kwuh.', word: 'queen',    emoji: '👑', group: 'MR' },
  { upper: 'R', lower: 'r', sound: 'rrr',  speechSound: 'rrrr.', word: 'rainbow',  emoji: '🌈', group: 'MR' },
  { upper: 'S', lower: 's', sound: 'sss',  speechSound: 'ssss.', word: 'sun',      emoji: '☀️', group: 'SZ' },
  { upper: 'T', lower: 't', sound: 'tuh',  speechSound: 'tuh.',  word: 'turtle',   emoji: '🐢', group: 'SZ' },
  { upper: 'U', lower: 'u', sound: 'uh',   speechSound: 'uhh.',  word: 'umbrella', emoji: '☂️', group: 'SZ' },
  { upper: 'V', lower: 'v', sound: 'vvv',  speechSound: 'vvvv.', word: 'violin',   emoji: '🎻', group: 'SZ' },
  { upper: 'W', lower: 'w', sound: 'wuh',  speechSound: 'wuh.',  word: 'whale',    emoji: '🐋', group: 'SZ' },
  { upper: 'X', lower: 'x', sound: 'ks',   speechSound: 'kss.',  word: 'box',      emoji: '📦', group: 'SZ' },
  { upper: 'Y', lower: 'y', sound: 'yuh',  speechSound: 'yuh.',  word: 'yo-yo',    emoji: '🪀', group: 'SZ' },
  { upper: 'Z', lower: 'z', sound: 'zzz',  speechSound: 'zzzz.', word: 'zebra',    emoji: '🦓', group: 'SZ' }
];

const BUBBLE_PALETTES = [
  { bg: 'radial-gradient(circle at 30% 30%, #ff8fab, #fb3668)', border: '#be123c', text: '#ffffff' },
  { bg: 'radial-gradient(circle at 30% 30%, #7dd3fc, #0284c7)', border: '#0369a1', text: '#ffffff' },
  { bg: 'radial-gradient(circle at 30% 30%, #fde047, #eab308)', border: '#a16207', text: '#422006' },
  { bg: 'radial-gradient(circle at 30% 30%, #86efac, #16a34a)', border: '#15803d', text: '#ffffff' },
  { bg: 'radial-gradient(circle at 30% 30%, #d8b4fe, #9333ea)', border: '#6b21a8', text: '#ffffff' },
  { bg: 'radial-gradient(circle at 30% 30%, #fdba74, #ea580c)', border: '#c2410c', text: '#ffffff' }
];

const STAGES = [
  { id: 'AF', label: 'A–F', btnClass: 'btn-blue' },
  { id: 'GL', label: 'G–L', btnClass: 'btn-blue' },
  { id: 'MR', label: 'M–R', btnClass: 'btn-blue' },
  { id: 'SZ', label: 'S–Z', btnClass: 'btn-blue' },
  { id: 'ALL', label: 'A–Z', btnClass: 'btn-purple' }
];

// Reactive Application State
const mode = ref('learn');
const currentIndex = ref(0);
const stars = ref(0);
const starBounce = ref(false);
const streak = ref(0);
const isMuted = ref(false);
const isAutoPlaying = ref(false);

// Speaking & Avatar Animation State
const isSpeaking = ref(false);
const speakerFace = ref('😊');
const activeCardPulse = ref('none');
const promptPulse = ref(false);

// Voice Management State
const availableVoices = ref([]);
const selectedVoiceName = ref('');

// Arcade Mode Reactive State
const arcadeStage = ref('AF');
const showPictureHint = ref(true);
const targetItem = ref(null);
const roundScore = ref(0);
const roundGoal = 5;
const stageComplete = ref(false);
const bubbles = reactive([]);
const feedback = reactive({ visible: false, text: 'Great job! ⭐', isSuccess: true });

// Template Refs
const confettiCanvasRef = ref(null);
const bubbleArenaRef = ref(null);
const pillRefs = ref([]);

// Internal Timers & Animation Handles
let audioCtx = null;
let autoPlayTimer = null;
let speechTimers = [];
let feedbackHideTimer = null;
let animFrameId = null;
const confettiParticles = [];
let confettiRunning = false;
let roundLocked = false;

// Computed Properties
const currentItem = computed(() => PHONICS_DATA[currentIndex.value]);
const highlightedWordParts = computed(() => {
  const word = currentItem.value.word;
  const lowerLetter = currentItem.value.lower.toLowerCase();
  const idx = word.toLowerCase().indexOf(lowerLetter);
  if (idx === -1) return { before: word, match: '', after: '' };
  return { before: word.slice(0, idx), match: word.slice(idx, idx + 1), after: word.slice(idx + 1) };
});
const selectedVoiceObj = computed(() => availableVoices.value.find(v => v.name === selectedVoiceName.value) || null);

/* WEB AUDIO SYNTHESIZED ARCADE SFX */
function ensureAudioContext() {
  if (!audioCtx) {
    const AudioContextClass = window.AudioContext || window.webkitAudioContext;
    if (AudioContextClass) audioCtx = new AudioContextClass();
  }
  if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
}
function playSfx(type) {
  if (isMuted.value) return;
  try {
    ensureAudioContext();
    if (!audioCtx) return;
    const now = audioCtx.currentTime;
    if (type === 'pop') {
      const osc = audioCtx.createOscillator(); const gain = audioCtx.createGain();
      osc.type = 'sine'; osc.frequency.setValueAtTime(320, now);
      osc.frequency.exponentialRampToValueAtTime(880, now + 0.14);
      gain.gain.setValueAtTime(0.35, now); gain.gain.exponentialRampToValueAtTime(0.01, now + 0.16);
      osc.connect(gain); gain.connect(audioCtx.destination); osc.start(now); osc.stop(now + 0.17);
      const chime = audioCtx.createOscillator(); const chimeGain = audioCtx.createGain();
      chime.type = 'triangle'; chime.frequency.setValueAtTime(987.77, now + 0.08); chime.frequency.setValueAtTime(1318.51, now + 0.16);
      chimeGain.gain.setValueAtTime(0.25, now + 0.08); chimeGain.gain.exponentialRampToValueAtTime(0.01, now + 0.38);
      chime.connect(chimeGain); chimeGain.connect(audioCtx.destination); chime.start(now + 0.08); chime.stop(now + 0.4);
    } else if (type === 'boing') {
      const osc = audioCtx.createOscillator(); const gain = audioCtx.createGain();
      osc.type = 'sine'; osc.frequency.setValueAtTime(240, now);
      osc.frequency.exponentialRampToValueAtTime(140, now + 0.18); osc.frequency.exponentialRampToValueAtTime(190, now + 0.3);
      gain.gain.setValueAtTime(0.22, now); gain.gain.exponentialRampToValueAtTime(0.01, now + 0.32);
      osc.connect(gain); gain.connect(audioCtx.destination); osc.start(now); osc.stop(now + 0.34);
    } else if (type === 'click') {
      const osc = audioCtx.createOscillator(); const gain = audioCtx.createGain();
      osc.type = 'triangle'; osc.frequency.setValueAtTime(520, now); osc.frequency.exponentialRampToValueAtTime(260, now + 0.06);
      gain.gain.setValueAtTime(0.18, now); gain.gain.exponentialRampToValueAtTime(0.01, now + 0.07);
      osc.connect(gain); gain.connect(audioCtx.destination); osc.start(now); osc.stop(now + 0.08);
    } else if (type === 'fanfare') {
      [523.25, 659.25, 783.99, 1046.50].forEach((freq, idx) => {
        const osc = audioCtx.createOscillator(); const gain = audioCtx.createGain();
        osc.type = 'triangle'; const start = now + idx * 0.11;
        osc.frequency.setValueAtTime(freq, start); gain.gain.setValueAtTime(0.28, start);
        gain.gain.exponentialRampToValueAtTime(0.01, start + 0.35);
        osc.connect(gain); gain.connect(audioCtx.destination); osc.start(start); osc.stop(start + 0.36);
      });
    }
  } catch (err) { console.warn('Audio playback skipped:', err); }
}

/* SPEECH SYNTHESIS & VOICE RANKING */
function scoreVoiceQuality(voice) {
  let score = 0; const name = voice.name.toLowerCase();
  if (!voice.lang.toLowerCase().startsWith('en')) return -100;
  if (name.includes('natural')) score += 60;
  if (name.includes('neural')) score += 55;
  if (name.includes('online')) score += 30;
  if (name.includes('enhanced') || name.includes('premium')) score += 45;
  ['ana','ava','emma','aria','jenny','samantha','google uk english female','google us english','karen','moira','tessa','siri']
    .forEach((fav, idx) => { if (name.includes(fav)) score += (40 - idx); });
  if (['en-us','en-gb','en-au','en-ca'].includes(voice.lang.toLowerCase())) score += 15;
  return score;
}
function formatVoiceLabel(name) {
  return name.replace(/Microsoft |Google |Apple /g, '').replace(/Online (Natural)/g, '★').replace(/ - English.*/g, '');
}
function loadPreferredVoice() {
  if (!('speechSynthesis' in window)) return;
  const voices = window.speechSynthesis.getVoices();
  if (!voices.length) return;
  const sorted = voices.filter(v => v.lang.toLowerCase().startsWith('en')).sort((a, b) => scoreVoiceQuality(b) - scoreVoiceQuality(a));
  availableVoices.value = sorted;
  if (sorted.length > 0 && !selectedVoiceName.value) selectedVoiceName.value = sorted[0].name;
}
function onVoiceChange() { playSfx('click'); if (mode.value === 'learn') playCurrentPhonics(); else speakArcadeTarget(); }
/* =========================================================
 * VOICE AUDIO — bundled TTS clips first, live API hook second,
 * browser speechSynthesis last.
 * Browser speechSynthesis is silent on some phones/tablets, so the
 * app ships pre-generated MP3 clips in public/audio/
 * (sound-a.mp3 … sound-z.mp3, word-a.mp3 … word-z.mp3) and plays
 * them with a plain Audio element, which works everywhere after a tap.
 * LIVE API HOOK: set TTS_API_URL to a proxy endpoint that accepts
 * ?text=... and returns an MP3 (e.g. a Cloudflare Worker / serverless
 * function holding your OpenAI, ElevenLabs, or Google TTS key — never
 * put a raw API key in this file, the site is public). When set, clips
 * are fetched live from that API instead of the bundled files.
 * ========================================================= */
const TTS_API_URL = '';
let currentAudio = null;
function stopCurrentAudio() {
  if (currentAudio) { try { currentAudio.pause(); currentAudio.src = ''; } catch (e) {} currentAudio = null; }
}
function clipUrl(kind, item) { return import.meta.env.BASE_URL + 'audio/' + kind + '-' + item.lower + '.mp3'; }
function playAudioUrl(url, fallbackText, onEndCallback = null) {
  if (isMuted.value) { const t = setTimeout(() => { if (onEndCallback) onEndCallback(); }, 400); speechTimers.push(t); return; }
  const mouthFrames = ['😮','😄','😲','😊']; let frameIdx = 0;
  const mouthInterval = setInterval(() => { speakerFace.value = mouthFrames[frameIdx % mouthFrames.length]; frameIdx++; }, 180);
  let done = false;
  const cleanup = () => { if (done) return; done = true; clearInterval(mouthInterval); if (onEndCallback) onEndCallback(); };
  try {
    stopCurrentAudio();
    const audio = new Audio(url); currentAudio = audio;
    audio.onended = cleanup;
    audio.onerror = () => { clearInterval(mouthInterval); done = true; speakWithBrowserVoice(fallbackText, onEndCallback); };
    const p = audio.play();
    if (p && p.catch) p.catch(() => { clearInterval(mouthInterval); done = true; speakWithBrowserVoice(fallbackText, onEndCallback); });
  } catch (e) { clearInterval(mouthInterval); speakWithBrowserVoice(fallbackText, onEndCallback); }
}
function playClip(kind, item, onEndCallback = null) {
  const fallbackText = kind === 'sound' ? item.speechSound : item.word + '!';
  if (TTS_API_URL) {
    playAudioUrl(TTS_API_URL + '?text=' + encodeURIComponent(fallbackText), fallbackText, onEndCallback);
  } else {
    playAudioUrl(clipUrl(kind, item), fallbackText, onEndCallback);
  }
}
function clearSpeechQueue() {
  speechTimers.forEach(id => clearTimeout(id)); speechTimers = [];
  stopCurrentAudio();
  if ('speechSynthesis' in window) window.speechSynthesis.cancel();
  isSpeaking.value = false; speakerFace.value = '😊'; activeCardPulse.value = 'none';
}
function speakWithBrowserVoice(text, onEndCallback = null) {
  if (isMuted.value || !('speechSynthesis' in window)) {
    const fallbackTimer = setTimeout(() => { if (onEndCallback) onEndCallback(); }, 750);
    speechTimers.push(fallbackTimer); return;
  }
  try {
    const utter = new SpeechSynthesisUtterance(text);
    const voice = selectedVoiceObj.value;
    if (voice) utter.voice = voice;
    const isNeural = voice && /natural|neural|online/i.test(voice.name);
    utter.rate = isNeural ? 0.85 : 0.80; utter.pitch = isNeural ? 1.10 : 1.18; utter.volume = 1.0;
    const mouthFrames = ['😮','😄','😲','😊']; let frameIdx = 0;
    const mouthInterval = setInterval(() => { speakerFace.value = mouthFrames[frameIdx % mouthFrames.length]; frameIdx++; }, 180);
    const cleanup = () => { clearInterval(mouthInterval); if (onEndCallback) onEndCallback(); };
    utter.onend = cleanup; utter.onerror = cleanup;
    window.speechSynthesis.speak(utter);
  } catch (e) { if (onEndCallback) onEndCallback(); }
}
function playCurrentPhonics(onComplete = null) {
  clearSpeechQueue(); const item = currentItem.value;
  isSpeaking.value = true; speakerFace.value = '😮'; activeCardPulse.value = 'letter';
  playClip('sound', item, () => {
    isSpeaking.value = false; speakerFace.value = '👂';
    const t1 = setTimeout(() => {
      isSpeaking.value = true; speakerFace.value = '😮';
      playClip('sound', item, () => {
        isSpeaking.value = false; speakerFace.value = '👂';
        const t2 = setTimeout(() => {
          activeCardPulse.value = 'word'; isSpeaking.value = true; speakerFace.value = '😄';
          playClip('word', item, () => {
            isSpeaking.value = false; speakerFace.value = '😊'; activeCardPulse.value = 'none';
            if (onComplete) onComplete();
          });
        }, 850); speechTimers.push(t2);
      });
    }, 950); speechTimers.push(t1);
  });
}

/* LEARN MODE */
function selectLetter(index, autoSpeak = true) {
  currentIndex.value = (index + PHONICS_DATA.length) % PHONICS_DATA.length;
  nextTick(() => { const el = pillRefs.value[currentIndex.value]; if (el) el.scrollIntoView({ behavior: 'smooth', inline: 'center', block: 'nearest' }); });
  if (autoSpeak) playCurrentPhonics();
}
function handlePillClick(idx) { playSfx('click'); stopAutoPlay(); selectLetter(idx, true); }
function prevLetter() { playSfx('click'); stopAutoPlay(); selectLetter(currentIndex.value - 1, true); }
function nextLetter() { playSfx('click'); stopAutoPlay(); selectLetter(currentIndex.value + 1, true); }
function toggleAutoPlay() { playSfx('click'); isAutoPlaying.value ? stopAutoPlay() : startAutoPlay(); }
function startAutoPlay() {
  isAutoPlaying.value = true;
  const runStep = () => {
    if (!isAutoPlaying.value) return;
    selectLetter(currentIndex.value, false);
    playCurrentPhonics(() => {
      if (!isAutoPlaying.value) return;
      autoPlayTimer = setTimeout(() => {
        if (!isAutoPlaying.value) return;
        if (currentIndex.value < PHONICS_DATA.length - 1) { currentIndex.value++; runStep(); }
        else { stopAutoPlay(); addStars(3); playSfx('fanfare'); launchConfetti(); }
      }, 1100);
    });
  };
  runStep();
}
function stopAutoPlay() { isAutoPlaying.value = false; if (autoPlayTimer) { clearTimeout(autoPlayTimer); autoPlayTimer = null; } }

/* ARCADE MODE */
function setArcadeStage(stage) { playSfx('click'); arcadeStage.value = stage; nextArcadeRound(); }
function togglePictureHint() { playSfx('click'); showPictureHint.value = !showPictureHint.value; }
function getStagePool() { return arcadeStage.value === 'ALL' ? PHONICS_DATA : PHONICS_DATA.filter(i => i.group === arcadeStage.value); }
function showArcadeFeedback(text, isSuccess = true) {
  feedback.text = text; feedback.isSuccess = isSuccess; feedback.visible = true;
  clearTimeout(feedbackHideTimer); feedbackHideTimer = setTimeout(() => { feedback.visible = false; }, 1350);
}
function speakArcadeTarget() {
  if (!targetItem.value) return; clearSpeechQueue(); promptPulse.value = true;
  playClip('sound', targetItem.value, () => {
    const t = setTimeout(() => { playClip('sound', targetItem.value, () => { promptPulse.value = false; }); }, 480);
    speechTimers.push(t);
  });
}
function nextArcadeRound() { roundScore.value = 0; roundLocked = false; stageComplete.value = false; nextTick(() => spawnArcadeTurn()); }
function spawnArcadeTurn() {
  roundLocked = false; bubbles.splice(0, bubbles.length);
  const pool = getStagePool(); const target = pool[Math.floor(Math.random() * pool.length)]; targetItem.value = target;
  const distractorsPool = PHONICS_DATA.filter(i => i.sound !== target.sound);
  const shuffled = [...distractorsPool].sort(() => Math.random() - 0.5);
  const bubbleCount = window.innerWidth < 640 ? 4 : 5;
  const choices = [target, ...shuffled.slice(0, bubbleCount - 1)].sort(() => Math.random() - 0.5);
  const arena = bubbleArenaRef.value; const rect = arena ? arena.getBoundingClientRect() : { width: 600, height: 420 };
  const arenaW = Math.max(rect.width, 300); const arenaH = Math.max(rect.height, 360);
  const bubbleSize = window.innerWidth < 640 ? 92 : 112; const slotWidth = (arenaW - 24) / choices.length;
  choices.forEach((item, idx) => {
    const palette = BUBBLE_PALETTES[(idx + Math.floor(Math.random() * BUBBLE_PALETTES.length)) % BUBBLE_PALETTES.length];
    bubbles.push({
      id: `${Date.now()}-${idx}-${item.upper}`, item, palette,
      x: 12 + idx * slotWidth + Math.max(0, (slotWidth - bubbleSize) / 2),
      y: 64 + Math.random() * Math.max(40, arenaH - bubbleSize - 130),
      vx: (Math.random() < 0.5 ? -1 : 1) * (0.45 + Math.random() * 0.45),
      vy: (Math.random() < 0.5 ? -1 : 1) * (0.45 + Math.random() * 0.45),
      size: bubbleSize, popped: false, faded: false, wobble: false
    });
  });
  speakArcadeTarget();
}
function handleBubbleTap(bubbleObj) {
  if (roundLocked || bubbleObj.popped || !targetItem.value) return;
  if (bubbleObj.item.sound === targetItem.value.sound) {
    roundLocked = true; bubbleObj.popped = true; playSfx('pop'); addStars(1); streak.value++; roundScore.value++;
    showArcadeFeedback(`⭐ /${bubbleObj.item.sound}/ — ${bubbleObj.item.word}! ${bubbleObj.item.emoji}`, true);
    clearSpeechQueue(); playClip('sound', bubbleObj.item, () => { playClip('word', bubbleObj.item); });
    bubbles.forEach(b => { if (!b.popped) b.faded = true; });
    setTimeout(() => {
      if (roundScore.value >= roundGoal) { addStars(5); playSfx('fanfare'); launchConfetti(); stageComplete.value = true; }
      else spawnArcadeTurn();
    }, 1350);
  } else {
    playSfx('boing'); streak.value = 0; bubbleObj.wobble = false;
    requestAnimationFrame(() => { bubbleObj.wobble = true; setTimeout(() => { bubbleObj.wobble = false; }, 460); });
    showArcadeFeedback(`Listen closely: /${targetItem.value.sound}/ 👂`, false); speakArcadeTarget();
  }
}
function updateBubblesLoop() {
  if (mode.value === 'arcade' && bubbleArenaRef.value) {
    const arena = bubbleArenaRef.value; const maxW = arena.clientWidth; const maxH = arena.clientHeight;
    const topBound = 52; const bottomBound = Math.max(topBound + 120, maxH - 54);
    for (let i = 0; i < bubbles.length; i++) {
      const b = bubbles[i]; if (b.popped) continue;
      b.x += b.vx; b.y += b.vy;
      if (b.x <= 8) { b.x = 8; b.vx = Math.abs(b.vx); } else if (b.x + b.size >= maxW - 8) { b.x = Math.max(8, maxW - b.size - 8); b.vx = -Math.abs(b.vx); }
      if (b.y <= topBound) { b.y = topBound; b.vy = Math.abs(b.vy); } else if (b.y + b.size >= bottomBound) { b.y = Math.max(topBound, bottomBound - b.size); b.vy = -Math.abs(b.vy); }
      for (let j = i + 1; j < bubbles.length; j++) {
        const b2 = bubbles[j]; if (b2.popped) continue;
        const dx = (b2.x + b2.size / 2) - (b.x + b.size / 2); const dy = (b2.y + b2.size / 2) - (b.y + b.size / 2);
        const dist = Math.hypot(dx, dy); const minDist = (b.size + b2.size) * 0.52;
        if (dist > 0 && dist < minDist) {
          const nx = dx / dist; const ny = dy / dist; const overlap = (minDist - dist) * 0.5;
          b.x -= nx * overlap; b.y -= ny * overlap; b2.x += nx * overlap; b2.y += ny * overlap;
          const tvx = b.vx; const tvy = b.vy; b.vx = b2.vx; b.vy = b2.vy; b2.vx = tvx; b2.vy = tvy;
        }
      }
    }
  }
  animFrameId = requestAnimationFrame(updateBubblesLoop);
}

/* SHARED HELPERS */
function addStars(amount) { stars.value += amount; starBounce.value = true; setTimeout(() => { starBounce.value = false; }, 240); }
function switchMode(newMode) {
  playSfx('click'); clearSpeechQueue(); stopAutoPlay(); mode.value = newMode;
  if (newMode === 'learn') selectLetter(currentIndex.value, true); else nextArcadeRound();
}
function toggleMute() { isMuted.value = !isMuted.value; if (isMuted.value) clearSpeechQueue(); else playSfx('click'); }
function launchConfetti() {
  const canvas = confettiCanvasRef.value; if (!canvas) return;
  canvas.width = window.innerWidth; canvas.height = window.innerHeight;
  const colors = ['#f43f5e','#facc15','#38bdf8','#4ade80','#c084fc','#fb923c'];
  for (let i = 0; i < 85; i++) confettiParticles.push({ x: Math.random() * canvas.width, y: -20 - Math.random() * 140, w: 10 + Math.random() * 8, h: 6 + Math.random() * 6, color: colors[Math.floor(Math.random() * colors.length)], vx: (Math.random() - 0.5) * 4, vy: 2.5 + Math.random() * 4, rot: Math.random() * Math.PI * 2, vRot: (Math.random() - 0.5) * 0.18 });
  if (!confettiRunning) { confettiRunning = true; animateConfetti(); }
}
function animateConfetti() {
  const canvas = confettiCanvasRef.value; if (!canvas) return; const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  for (let i = confettiParticles.length - 1; i >= 0; i--) {
    const p = confettiParticles[i]; p.x += p.vx; p.y += p.vy; p.rot += p.vRot;
    ctx.save(); ctx.translate(p.x, p.y); ctx.rotate(p.rot); ctx.fillStyle = p.color; ctx.fillRect(-p.w / 2, -p.h / 2, p.w, p.h); ctx.restore();
    if (p.y > canvas.height + 30) confettiParticles.splice(i, 1);
  }
  if (confettiParticles.length > 0) requestAnimationFrame(animateConfetti); else confettiRunning = false;
}
function handleKeyDown(e) {
  if (mode.value === 'learn') {
    if (e.key === 'ArrowRight') nextLetter();
    if (e.key === 'ArrowLeft') prevLetter();
    if (e.key === ' ') { e.preventDefault(); playCurrentPhonics(); }
  }
}
function handleResize() { if (confettiCanvasRef.value) { confettiCanvasRef.value.width = window.innerWidth; confettiCanvasRef.value.height = window.innerHeight; } }

onMounted(() => {
  if ('speechSynthesis' in window) { window.speechSynthesis.onvoiceschanged = loadPreferredVoice; loadPreferredVoice(); }
  window.addEventListener('keydown', handleKeyDown); window.addEventListener('resize', handleResize);
  selectLetter(0, false); updateBubblesLoop();
});
onUnmounted(() => {
  clearSpeechQueue(); stopAutoPlay();
  if (animFrameId) cancelAnimationFrame(animFrameId);
  window.removeEventListener('keydown', handleKeyDown); window.removeEventListener('resize', handleResize);
});
</script>

<template>
  <div class="phonics-app text-slate-800 select-none">
    <div class="cloud cloud-1"></div>
    <div class="cloud cloud-2"></div>
    <canvas ref="confettiCanvasRef" class="confetti-canvas"></canvas>
    <div class="relative z-10 flex flex-col min-h-screen max-w-5xl w-full mx-auto px-3 py-3 sm:px-6 sm:py-4">
      <header class="bg-white/95 backdrop-blur-md rounded-3xl border-4 border-amber-400 shadow-[0_6px_0_#d97706] px-4 py-2.5 sm:px-6 sm:py-3 mb-3 flex flex-wrap items-center justify-between gap-2">
        <div class="flex items-center gap-2.5">
          <div class="w-11 h-11 sm:w-12 sm:h-12 rounded-2xl bg-gradient-to-br from-pink-500 via-amber-400 to-sky-400 flex items-center justify-center text-white text-2xl sm:text-3xl shadow-md border-2 border-white">🎈</div>
          <div>
            <h1 class="text-xl sm:text-3xl font-bold tracking-tight text-amber-500 drop-shadow-[0_2px_0_rgba(180,83,9,0.25)] leading-none">Phonics Pop!</h1>
            <span class="inline-block text-xs sm:text-sm font-extrabold uppercase tracking-wider text-sky-600">A–Z Sound Arcade</span>
          </div>
        </div>
        <div class="flex items-center gap-2 sm:gap-3">
          <button @click="switchMode('learn')" :class="['arcade-btn btn-green px-3.5 py-2 sm:px-5 sm:py-2.5 rounded-2xl text-sm sm:text-lg font-bold flex items-center gap-2', mode !== 'learn' && 'opacity-85']"><i class="fa-solid fa-book-open"></i><span>Learn A–Z</span></button>
          <button @click="switchMode('arcade')" :class="['arcade-btn btn-pink px-3.5 py-2 sm:px-5 sm:py-2.5 rounded-2xl text-sm sm:text-lg font-bold flex items-center gap-2', mode !== 'arcade' && 'opacity-85']"><i class="fa-solid fa-gamepad"></i><span>Pop Game!</span></button>
        </div>
        <div class="flex items-center gap-2">
          <div class="hidden sm:flex items-center bg-sky-100 border-2 border-sky-300 rounded-2xl px-2.5 py-1.5">
            <i class="fa-solid fa-microphone-lines text-sky-600 mr-1.5 text-sm"></i>
            <select v-model="selectedVoiceName" @change="onVoiceChange" aria-label="Choose Voice" class="bg-transparent text-xs font-extrabold text-sky-900 max-w-[130px] truncate focus:outline-none cursor-pointer">
              <option v-if="availableVoices.length === 0" value="">Auto Warm Voice</option>
              <option v-for="voice in availableVoices" :key="voice.name" :value="voice.name">{{ formatVoiceLabel(voice.name) }}</option>
            </select>
          </div>
          <div :class="['bg-amber-100 border-2 border-amber-400 rounded-2xl px-3 py-1.5 flex items-center gap-1.5 shadow-inner transition-transform', starBounce && 'scale-110']"><span class="text-xl sm:text-2xl">⭐</span><span class="text-lg sm:text-2xl font-black text-amber-700">{{ stars }}</span></div>
          <button @click="toggleMute" aria-label="Toggle Sound" class="arcade-btn btn-yellow w-11 h-11 sm:w-12 sm:h-12 rounded-2xl flex items-center justify-center text-lg sm:text-xl"><i :class="isMuted ? 'fa-solid fa-volume-xmark text-red-600' : 'fa-solid fa-volume-high'"></i></button>
        </div>
      </header>

      <main v-show="mode === 'learn'" class="flex-1 flex flex-col justify-between gap-3">
        <section class="flex-1 bg-white/95 rounded-[2rem] border-4 border-sky-400 shadow-[0_8px_0_#0284c7] p-4 sm:p-6 flex flex-col items-center justify-between relative overflow-hidden">
          <div class="w-full flex items-center justify-between gap-2">
            <span class="bg-sky-100 text-sky-800 border-2 border-sky-300 px-3.5 py-1 rounded-full text-sm sm:text-base font-extrabold">{{ currentIndex + 1 }} / 26</span>
            <div :class="['flex items-center gap-2 bg-amber-50 border-2 border-amber-300 px-4 py-1.5 rounded-full', isSpeaking && 'is-speaking']">
              <span class="text-xl sm:text-2xl transition-transform">{{ speakerFace }}</span>
              <div class="flex items-center gap-1 h-8"><div class="sound-bar"></div><div class="sound-bar"></div><div class="sound-bar"></div><div class="sound-bar"></div><div class="sound-bar"></div></div>
            </div>
            <button @click="toggleAutoPlay" :class="['arcade-btn px-3.5 py-1.5 rounded-full text-xs sm:text-sm font-bold flex items-center gap-1.5', isAutoPlaying ? 'btn-pink' : 'btn-purple']"><i :class="isAutoPlaying ? 'fa-solid fa-pause' : 'fa-solid fa-play'"></i><span>{{ isAutoPlaying ? 'Pause' : 'Auto A–Z' }}</span></button>
          </div>
          <div class="my-2 sm:my-4 w-full grid grid-cols-1 md:grid-cols-2 gap-4 sm:gap-6 items-center max-w-3xl">
            <div @click="playCurrentPhonics()" :class="['cursor-pointer bg-gradient-to-b from-amber-50 to-amber-100 border-4 border-amber-400 rounded-[2rem] p-5 sm:p-8 flex flex-col items-center justify-center shadow-[0_6px_0_#f59e0b] hover:scale-[1.02] transition-transform', activeCardPulse === 'letter' && 'speaking-pulse']">
              <div class="text-7xl sm:text-8xl md:text-9xl font-black tracking-wide text-slate-800 leading-none select-none">{{ currentItem.upper }} {{ currentItem.lower }}</div>
              <div class="mt-3 bg-white border-2 border-amber-300 px-4 py-1 rounded-full text-xl sm:text-2xl font-extrabold text-amber-600">/{{ currentItem.sound }}/</div>
            </div>
            <div @click="playCurrentPhonics()" :class="['cursor-pointer bg-gradient-to-b from-sky-50 to-sky-100 border-4 border-sky-400 rounded-[2rem] p-5 sm:p-8 flex flex-col items-center justify-center shadow-[0_6px_0_#38bdf8] hover:scale-[1.02] transition-transform', activeCardPulse === 'word' && 'speaking-pulse']">
              <div class="text-7xl sm:text-8xl md:text-9xl leading-none drop-shadow-sm transition-transform">{{ currentItem.emoji }}</div>
              <div class="mt-3 text-3xl sm:text-4xl font-black text-sky-900 tracking-wide">{{ highlightedWordParts.before }}<span class="text-pink-600 underline decoration-4 decoration-pink-400">{{ highlightedWordParts.match }}</span>{{ highlightedWordParts.after }}</div>
            </div>
          </div>
          <div class="w-full max-w-2xl flex items-center justify-center gap-3 sm:gap-5 pt-1">
            <button @click="prevLetter" aria-label="Previous Letter" class="arcade-btn btn-blue w-14 h-14 sm:w-16 sm:h-16 rounded-2xl text-2xl sm:text-3xl flex items-center justify-center shrink-0"><i class="fa-solid fa-arrow-left"></i></button>
            <button @click="playCurrentPhonics()" class="arcade-btn btn-yellow flex-1 max-w-xs h-14 sm:h-16 rounded-2xl text-lg sm:text-2xl font-black flex items-center justify-center gap-2.5 px-4"><i class="fa-solid fa-volume-high text-xl sm:text-2xl"></i><span>Hear Sound</span></button>
            <button @click="nextLetter" aria-label="Next Letter" class="arcade-btn btn-green w-14 h-14 sm:w-16 sm:h-16 rounded-2xl text-2xl sm:text-3xl flex items-center justify-center shrink-0"><i class="fa-solid fa-arrow-right"></i></button>
          </div>
        </section>
        <nav aria-label="Alphabet Selector" class="bg-white/95 rounded-3xl border-4 border-amber-400 shadow-[0_5px_0_#d97706] p-2.5 sm:p-3">
          <div class="az-scroll flex items-center gap-2 overflow-x-auto pb-1.5 pt-0.5 px-1">
            <button v-for="(item, idx) in PHONICS_DATA" :key="item.upper" :ref="el => { if (el) pillRefs[idx] = el }" @click="handlePillClick(idx)" :class="['arcade-btn shrink-0 rounded-2xl transition-all flex items-center justify-center', idx === currentIndex ? 'w-12 h-12 sm:w-13 sm:h-13 font-black text-lg sm:text-xl bg-amber-400 border-2 border-amber-600 text-amber-950 scale-110 shadow-md' : 'w-11 h-11 sm:w-12 sm:h-12 font-extrabold text-base sm:text-lg bg-sky-100 hover:bg-sky-200 border-2 border-sky-300 text-sky-900']">{{ item.upper }}{{ item.lower }}</button>
          </div>
        </nav>
      </main>

      <main v-show="mode === 'arcade'" class="flex-1 flex flex-col gap-3">
        <section class="bg-white/95 rounded-3xl border-4 border-pink-400 shadow-[0_6px_0_#db2777] p-3 sm:p-4 flex flex-wrap items-center justify-between gap-2.5">
          <div class="flex items-center gap-1.5 flex-wrap">
            <button v-for="stage in STAGES" :key="stage.id" @click="setArcadeStage(stage.id)" :class="['arcade-btn px-3 py-1.5 rounded-xl text-xs sm:text-sm font-extrabold', stage.btnClass, arcadeStage === stage.id ? 'ring-4 ring-amber-300' : 'opacity-75']">{{ stage.label }}</button>
          </div>
          <div class="flex items-center gap-2.5 mx-auto">
            <button v-if="targetItem" @click="speakArcadeTarget" :class="['arcade-btn btn-yellow px-4 py-2 sm:px-6 sm:py-2.5 rounded-2xl flex items-center gap-2.5 text-lg sm:text-2xl font-black', promptPulse && 'speaking-pulse']"><i class="fa-solid fa-ear-listen text-pink-600"></i><span>/{{ targetItem.sound }}/</span><span v-show="showPictureHint" class="text-2xl sm:text-3xl ml-1">{{ targetItem.emoji }}</span></button>
          </div>
          <div class="flex items-center gap-2">
            <button @click="togglePictureHint" title="Show or Hide Picture Clue" :class="['arcade-btn btn-green px-3 py-2 rounded-xl text-xs sm:text-sm font-extrabold flex items-center gap-1.5', !showPictureHint && 'opacity-75']"><i :class="showPictureHint ? 'fa-solid fa-eye' : 'fa-solid fa-eye-slash'"></i><span>Clue</span></button>
            <div class="bg-pink-100 border-2 border-pink-400 px-3 py-1.5 rounded-xl flex items-center gap-1 font-black text-pink-700 text-sm sm:text-base"><span>🔥</span><span>{{ streak }}</span></div>
          </div>
        </section>
        <section ref="bubbleArenaRef" class="relative flex-1 min-h-[360px] sm:min-h-[430px] bg-gradient-to-b from-sky-200/80 via-white/75 to-emerald-100/85 rounded-[2rem] border-4 border-white shadow-[0_8px_0_rgba(2,132,199,0.35)] overflow-hidden">
          <div :class="['pointer-events-none absolute top-3 left-1/2 -translate-x-1/2 z-20 px-5 py-1.5 rounded-full shadow-lg text-base sm:text-xl font-black transition-all duration-200 whitespace-nowrap border-2', feedback.isSuccess ? 'bg-emerald-100 border-emerald-500 text-emerald-900' : 'bg-amber-100 border-amber-500 text-amber-900', feedback.visible ? 'opacity-100 scale-100' : 'opacity-0 scale-90']">{{ feedback.text }}</div>
          <button v-for="b in bubbles" :key="b.id" type="button" @pointerdown.prevent="handleBubbleTap(b)" :class="['phonics-bubble select-none', b.popped && 'bubble-popped', b.wobble && 'wobble-miss']" :style="{ width: `${b.size}px`, height: `${b.size}px`, background: b.palette.bg, border: `4px solid ${b.palette.border}`, color: b.palette.text, opacity: b.faded && !b.popped ? 0.35 : 1, pointerEvents: b.faded || b.popped ? 'none' : 'auto', transform: `translate3d(${b.x}px, ${b.y}px, 0)` }">
            <span class="text-3xl sm:text-4xl font-black leading-none drop-shadow-[0_2px_2px_rgba(0,0,0,0.25)]">{{ b.item.upper }} {{ b.item.lower }}</span>
          </button>
          <div class="absolute bottom-3 left-1/2 -translate-x-1/2 z-20 flex items-center gap-2 bg-white/85 px-4 py-1.5 rounded-full border-2 border-sky-300">
            <span v-for="i in roundGoal" :key="i" :class="i <= roundScore ? 'text-xl sm:text-2xl transition-transform scale-110' : 'text-xl sm:text-2xl opacity-30 grayscale'">⭐</span>
          </div>
          <div v-if="stageComplete" class="absolute inset-0 z-30 bg-sky-900/60 backdrop-blur-sm flex items-center justify-center p-4">
            <div class="bg-white rounded-[2rem] border-4 border-amber-400 shadow-2xl p-6 sm:p-8 max-w-sm w-full text-center flex flex-col items-center gap-4">
              <div class="text-6xl sm:text-7xl animate-bounce">🏆</div>
              <h2 class="text-3xl sm:text-4xl font-black text-amber-500">Super Star!</h2>
              <div class="flex items-center gap-2 text-2xl font-black text-slate-700 bg-amber-100 px-5 py-2 rounded-2xl border-2 border-amber-300"><span>⭐ +5 Stars!</span></div>
              <button @click="nextArcadeRound" class="arcade-btn btn-green w-full py-3.5 rounded-2xl text-xl sm:text-2xl font-black flex items-center justify-center gap-2"><span>Play Again!</span><i class="fa-solid fa-rotate-right"></i></button>
            </div>
          </div>
        </section>
      </main>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Fredoka:wght@500;600;700&family=Nunito:wght@700;800;900&display=swap');
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css');
.phonics-app { font-family: 'Fredoka', 'Nunito', sans-serif; background: linear-gradient(180deg, #0284c7 0%, #38bdf8 45%, #e0f2fe 100%); min-height: 100vh; margin: 0; overflow-x: hidden; display: flex; flex-direction: column; -webkit-tap-highlight-color: transparent; }
.cloud { position: fixed; background: rgba(255,255,255,0.75); border-radius: 9999px; pointer-events: none; z-index: 0; animation: floatCloud linear infinite; }
.cloud::before, .cloud::after { content: ''; position: absolute; background: inherit; border-radius: 50%; }
.cloud-1 { width: 140px; height: 45px; top: 8%; left: -160px; animation-duration: 38s; }
.cloud-1::before { width: 65px; height: 65px; top: -30px; left: 20px; }
.cloud-1::after { width: 50px; height: 50px; top: -20px; left: 70px; }
.cloud-2 { width: 180px; height: 55px; top: 24%; left: -200px; animation-duration: 52s; animation-delay: -15s; opacity: 0.6; }
.cloud-2::before { width: 80px; height: 80px; top: -38px; left: 28px; }
.cloud-2::after { width: 60px; height: 60px; top: -25px; left: 92px; }
@keyframes floatCloud { from { transform: translateX(0); } to { transform: translateX(calc(100vw + 250px)); } }
.arcade-btn { transition: transform 0.1s cubic-bezier(0.34,1.56,0.64,1), box-shadow 0.1s ease; position: relative; }
.arcade-btn:active:not(:disabled) { transform: translateY(5px) scale(0.97); }
.btn-yellow { background: linear-gradient(180deg,#fde047 0%,#facc15 100%); border: 4px solid #ca8a04; box-shadow: 0 6px 0 #a16207, 0 10px 18px rgba(0,0,0,0.18); color: #713f12; }
.btn-yellow:active:not(:disabled) { box-shadow: 0 1px 0 #a16207, 0 3px 6px rgba(0,0,0,0.18); }
.btn-green { background: linear-gradient(180deg,#4ade80 0%,#22c55e 100%); border: 4px solid #15803d; box-shadow: 0 6px 0 #166534, 0 10px 18px rgba(0,0,0,0.18); color: #fff; text-shadow: 0 2px 0 #15803d; }
.btn-green:active:not(:disabled) { box-shadow: 0 1px 0 #166534, 0 3px 6px rgba(0,0,0,0.18); }
.btn-pink { background: linear-gradient(180deg,#f472b6 0%,#ec4899 100%); border: 4px solid #be185d; box-shadow: 0 6px 0 #9d174d, 0 10px 18px rgba(0,0,0,0.18); color: #fff; text-shadow: 0 2px 0 #9d174d; }
.btn-pink:active:not(:disabled) { box-shadow: 0 1px 0 #9d174d, 0 3px 6px rgba(0,0,0,0.18); }
.btn-blue { background: linear-gradient(180deg,#60a5fa 0%,#3b82f6 100%); border: 4px solid #1d4ed8; box-shadow: 0 6px 0 #1e40af, 0 10px 18px rgba(0,0,0,0.18); color: #fff; text-shadow: 0 2px 0 #1e40af; }
.btn-blue:active:not(:disabled) { box-shadow: 0 1px 0 #1e40af, 0 3px 6px rgba(0,0,0,0.18); }
.btn-purple { background: linear-gradient(180deg,#c084fc 0%,#a855f7 100%); border: 4px solid #7e22ce; box-shadow: 0 6px 0 #6b21a8, 0 10px 18px rgba(0,0,0,0.18); color: #fff; text-shadow: 0 2px 0 #6b21a8; }
.btn-purple:active:not(:disabled) { box-shadow: 0 1px 0 #6b21a8, 0 3px 6px rgba(0,0,0,0.18); }
@keyframes speakPulse { 0%,100% { transform: scale(1); } 50% { transform: scale(1.1); } }
.speaking-pulse { animation: speakPulse 0.55s ease-in-out infinite; }
@keyframes waveBar { 0%,100% { height: 10px; } 50% { height: 32px; } }
.sound-bar { width: 6px; height: 10px; border-radius: 999px; background-color: #f59e0b; transition: height 0.15s ease; }
.is-speaking .sound-bar:nth-child(1) { animation: waveBar 0.45s ease-in-out infinite 0s; }
.is-speaking .sound-bar:nth-child(2) { animation: waveBar 0.45s ease-in-out infinite 0.12s; }
.is-speaking .sound-bar:nth-child(3) { animation: waveBar 0.45s ease-in-out infinite 0.24s; }
.is-speaking .sound-bar:nth-child(4) { animation: waveBar 0.45s ease-in-out infinite 0.12s; }
.is-speaking .sound-bar:nth-child(5) { animation: waveBar 0.45s ease-in-out infinite 0s; }
@keyframes gentleWobble { 0%,100% { transform: rotate(0deg) scale(1); } 25% { transform: rotate(-10deg) scale(0.95); } 50% { transform: rotate(10deg) scale(0.95); } 75% { transform: rotate(-6deg) scale(0.98); } }
.wobble-miss { animation: gentleWobble 0.45s ease-in-out; }
@keyframes popOut { 0% { transform: scale(1); opacity: 1; } 45% { transform: scale(1.38); opacity: 0.95; } 100% { transform: scale(0); opacity: 0; } }
.bubble-popped { animation: popOut 0.32s cubic-bezier(0.34,1.56,0.64,1) forwards; pointer-events: none; }
.phonics-bubble { position: absolute; top: 0; left: 0; border-radius: 50%; display: flex; flex-direction: column; align-items: center; justify-content: center; cursor: pointer; box-shadow: inset -8px -10px 18px rgba(0,0,0,0.18), inset 8px 10px 18px rgba(255,255,255,0.65), 0 10px 20px rgba(0,0,0,0.2); transition: opacity 0.2s ease; touch-action: manipulation; }
.phonics-bubble::after { content: ''; position: absolute; top: 14%; left: 18%; width: 26%; height: 15%; border-radius: 50%; background: rgba(255,255,255,0.72); transform: rotate(-32deg); pointer-events: none; }
.az-scroll::-webkit-scrollbar { height: 8px; }
.az-scroll::-webkit-scrollbar-track { background: rgba(255,255,255,0.4); border-radius: 999px; }
.az-scroll::-webkit-scrollbar-thumb { background: #f59e0b; border-radius: 999px; }
.confetti-canvas { position: fixed; inset: 0; width: 100%; height: 100%; pointer-events: none; z-index: 50; }
</style>
