<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>GAME HUT ARCADE</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Space+Mono:wght@400;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
  :root{
    --bg: #0c0916;
    --panel: #171129;
    --bezel: #241a3d;
    --pink: #ff2e88;
    --cyan: #2be3ff;
    --yellow: #ffd23f;
    --ink: #eae4ff;
    --dim: #8b7fb0;
  }

  *{ box-sizing: border-box; }

  html, body{
    margin:0; padding:0; height:100%;
    background: var(--bg);
    background-image:
      radial-gradient(circle at 20% 10%, rgba(255,46,136,0.08), transparent 40%),
      radial-gradient(circle at 80% 90%, rgba(43,227,255,0.08), transparent 40%);
    color: var(--ink);
    font-family: 'Space Mono', monospace;
    display:flex; align-items:center; justify-content:center;
    min-height: 100vh;
  }

  .marquee{
    text-align:center;
    margin-bottom: 18px;
  }
  .marquee h1{
    font-family:'Press Start 2P', monospace;
    font-size: clamp(20px, 4vw, 34px);
    letter-spacing: 2px;
    margin:0;
    color: var(--yellow);
    text-shadow:
      0 0 6px var(--yellow),
      0 0 18px rgba(255,210,63,0.6);
    animation: flicker 3.2s infinite;
  }
  .marquee p{
    margin: 8px 0 0;
    color: var(--dim);
    font-size: 12px;
    letter-spacing: 3px;
  }
  @keyframes flicker{
    0%, 19%, 21%, 23%, 80%, 100% { opacity: 1; }
    20%, 22%, 79% { opacity: 0.72; }
  }

  .cabinet{
    width: min(560px, 92vw);
    background: linear-gradient(160deg, var(--bezel), #150f26 70%);
    border-radius: 28px;
    padding: 22px;
    box-shadow:
      0 0 0 2px rgba(255,255,255,0.03) inset,
      0 30px 60px rgba(0,0,0,0.55),
      0 0 40px rgba(255,46,136,0.08);
    position: relative;
  }
  .cabinet::before{
    content:"";
    position:absolute; inset: -1px;
    border-radius: 28px;
    padding: 1px;
    background: linear-gradient(160deg, rgba(255,46,136,0.5), rgba(43,227,255,0.3));
    -webkit-mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
    -webkit-mask-composite: xor; mask-composite: exclude;
    pointer-events:none;
  }

  .screen{
    background: radial-gradient(ellipse at center, #0f1a1e 0%, #060a0d 100%);
    border-radius: 16px;
    padding: 26px 22px;
    min-height: 360px;
    position: relative;
    overflow: hidden;
    box-shadow: inset 0 0 40px rgba(0,0,0,0.8), inset 0 0 6px rgba(0,0,0,0.9);
  }
  .screen::after{
    content:"";
    position:absolute; inset:0;
    background: repeating-linear-gradient(
      to bottom,
      rgba(255,255,255,0.035),
      rgba(255,255,255,0.035) 1px,
      transparent 2px,
      transparent 3px
    );
    pointer-events:none;
    mix-blend-mode: overlay;
  }
  .screen-content{
    position: relative;
    z-index: 1;
    animation: powerOn 0.4s ease-out;
  }
  @keyframes powerOn{
    from{ opacity:0; transform: scaleY(0.9); }
    to{ opacity:1; transform: scaleY(1); }
  }

  h2.title{
    font-family:'Press Start 2P', monospace;
    font-size: 15px;
    color: var(--cyan);
    text-shadow: 0 0 10px rgba(43,227,255,0.6);
    margin: 0 0 18px;
    letter-spacing: 1px;
  }

  .menu-row{
    display:flex; flex-direction:column; gap: 14px; margin-top: 8px;
  }

  button{
    font-family:'Space Mono', monospace;
    font-weight: 700;
    cursor: pointer;
    border: none;
    border-radius: 10px;
    padding: 14px 18px;
    font-size: 14px;
    letter-spacing: 1px;
    transition: transform 0.08s ease, filter 0.15s ease;
  }
  button:active{ transform: scale(0.97); }

  .game-btn{
    background: linear-gradient(135deg, #201a38, #171129);
    color: var(--ink);
    border: 1px solid rgba(255,255,255,0.08);
    display:flex; align-items:center; justify-content:space-between;
    text-align:left;
  }
  .game-btn:hover{ border-color: var(--pink); box-shadow: 0 0 16px rgba(255,46,136,0.35); }
  .game-btn .arrow{ color: var(--pink); font-family:'Press Start 2P'; font-size:11px; }

  .diff-grid{ display:grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 10px; }
  .diff-btn{ background: var(--panel); color: var(--ink); border: 1px solid rgba(255,255,255,0.06); }
  .diff-btn small{ display:block; color: var(--dim); font-weight:400; margin-top:4px; font-size: 10px; }
  .diff-btn:hover{ box-shadow: 0 0 14px rgba(43,227,255,0.35); border-color: var(--cyan); }

  .primary{ background: linear-gradient(135deg, var(--pink), #b4176a); color:#fff; }
  .primary:hover{ filter: brightness(1.1); box-shadow: 0 0 18px rgba(255,46,136,0.55); }

  .ghost{ background: transparent; color: var(--dim); border: 1px solid rgba(255,255,255,0.12); }
  .ghost:hover{ color: var(--ink); border-color: var(--ink); }

  .row{ display:flex; gap:10px; }
  .row > *{ flex:1; }

  input[type=text], input[type=number], input[type=password]{
    font-family:'Space Mono', monospace;
    font-size: 16px;
    padding: 12px 14px;
    border-radius: 10px;
    border: 1px solid rgba(255,255,255,0.15);
    background: #0d0818;
    color: var(--ink);
    width: 100%;
  }
  input:focus{ outline: 2px solid var(--cyan); outline-offset: 2px; }

  .hud{
    display:flex; justify-content: space-between;
    font-size: 11px; color: var(--dim);
    margin-bottom: 10px; letter-spacing: 1px;
  }
  .hud span.val{ color: var(--yellow); }

  .lives{ letter-spacing: 4px; font-size: 16px; }

  .feedback{
    margin-top: 16px;
    min-height: 28px;
    font-size: 14px;
  }
  .feedback.win{ color: var(--yellow); text-shadow: 0 0 10px rgba(255,210,63,0.6); }
  .feedback.lose{ color: var(--pink); }
  .feedback.hint{ color: var(--cyan); }

  .rps-grid{ display:grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-top: 14px; }
  .rps-btn{ background: var(--panel); font-size: 26px; padding: 18px 8px; border:1px solid rgba(255,255,255,0.06); }
  .rps-btn:hover{ border-color: var(--pink); box-shadow: 0 0 16px rgba(255,46,136,0.35); transform: translateY(-2px); }
  .rps-label{ display:block; font-size: 10px; margin-top: 6px; color: var(--dim); letter-spacing: 1px; }

  .versus{
    display:flex; align-items:center; justify-content: space-around;
    margin: 18px 0; text-align:center;
  }
  .versus .choice{ font-size: 44px; }
  .versus .vs{ font-family:'Press Start 2P'; font-size: 12px; color: var(--dim); }
  .versus .who{ font-size: 10px; color: var(--dim); letter-spacing: 2px; margin-top: 6px; }

  .score{
    text-align:center; font-size: 12px; color: var(--dim); letter-spacing: 2px; margin-top: 10px;
  }
  .score span{ color: var(--cyan); }

  .back-link{
    margin-top: 20px; display:inline-block;
    font-size: 11px; color: var(--dim); text-decoration: none; cursor:pointer;
    letter-spacing: 1px;
  }
  .back-link:hover{ color: var(--pink); }

  .footer-hint{
    text-align:center; margin-top: 16px; color: var(--dim); font-size: 10px; letter-spacing: 2px;
  }
  .blink{ animation: blink 1.1s steps(1) infinite; }
  @keyframes blink{ 50%{ opacity: 0; } }

  /* --- account additions --- */
  .account-bar{
    display:flex; justify-content:space-between; align-items:center; flex-wrap: wrap;
    background: rgba(255,255,255,0.03);
    border: 1px solid rgba(255,255,255,0.08);
    border-radius: 10px;
    padding: 10px 12px;
    margin-bottom: 16px;
    gap: 6px;
  }
  .account-bar .who{ font-size: 12px; letter-spacing: 1px; }
  .account-bar .who .lvl{ color: var(--yellow); }
  .account-bar .mini-btn{
    background: transparent; border: 1px solid rgba(255,255,255,0.15);
    color: var(--dim); font-size: 10px; padding: 6px 10px; border-radius: 8px;
  }
  .account-bar .mini-btn:hover{ color: var(--ink); border-color: var(--cyan); }
  .xpbar{
    height: 8px; border-radius: 6px; background: rgba(255,255,255,0.08);
    overflow: hidden; margin-top: 6px;
  }
  .xpbar-fill{
    height: 100%; background: linear-gradient(90deg, var(--pink), var(--cyan));
    transition: width 0.3s ease;
  }
  .level-up{
    text-align:center; margin-top: 10px; font-size: 12px;
    color: var(--yellow); text-shadow: 0 0 10px rgba(255,210,63,0.6);
    letter-spacing: 1px;
  }
  .friend-row{
    display:flex; justify-content:space-between; align-items:center;
    padding: 10px 12px; background: var(--panel);
    border: 1px solid rgba(255,255,255,0.06); border-radius: 10px;
    margin-bottom: 8px; font-size: 12px;
  }
  .friend-row .lvl{ color: var(--yellow); font-size: 11px; }

  /* --- coins / shop --- */
  .coin-pill{
    display:inline-flex; align-items:center; gap:5px;
    color: var(--yellow); font-size: 11px; letter-spacing: 1px;
    background: rgba(255,210,63,0.08); border: 1px solid rgba(255,210,63,0.25);
    border-radius: 20px; padding: 4px 10px; white-space:nowrap;
  }
  .points-pill{
    display:inline-flex; align-items:center; gap:5px;
    color: var(--cyan); font-size: 11px; letter-spacing: 1px;
    background: rgba(43,227,255,0.08); border: 1px solid rgba(43,227,255,0.25);
    border-radius: 20px; padding: 4px 10px; white-space:nowrap;
  }
  .shop-list{ display:flex; flex-direction:column; gap:10px; margin-top:12px; }
  .shop-row{
    display:flex; align-items:center; justify-content:space-between; gap:10px;
    background: var(--panel); border: 1px solid rgba(255,255,255,0.06);
    border-radius: 12px; padding: 12px 14px;
  }
  .shop-row.selected{ border-color: var(--yellow); box-shadow: 0 0 14px rgba(255,210,63,0.25); }
  .shop-row .info{ flex:1; }
  .shop-row .cname{ font-size: 12px; letter-spacing: 1px; }
  .shop-row .ctier{ font-size: 9px; color: var(--dim); letter-spacing: 1px; margin-top: 3px; }
  .shop-btn{
    flex: 0 0 auto; font-size: 10px; padding: 8px 12px;
    background: linear-gradient(135deg, var(--pink), #b4176a); color:#fff;
  }
  .shop-btn.owned{ background: transparent; border: 1px solid var(--cyan); color: var(--cyan); }
  .shop-btn.selected-btn{ background: linear-gradient(135deg, var(--yellow), #b48a17); color:#1a1400; }
  .shop-btn:disabled{ opacity: 0.4; cursor:not-allowed; background: rgba(255,255,255,0.05); color: var(--dim); }
  .shop-btn-row{ display:flex; flex-direction:column; gap:6px; flex:0 0 auto; }
  .preview-btn{
    background: transparent; border: 1px solid rgba(255,255,255,0.15);
    color: var(--dim); font-size: 9px; padding: 6px 12px;
  }
  .preview-btn:hover{ color: var(--cyan); border-color: var(--cyan); }

  /* --- leaderboard --- */
  .rank-row{
    display:flex; justify-content:space-between; align-items:center;
    padding: 10px 12px; background: var(--panel);
    border: 1px solid rgba(255,255,255,0.06); border-radius: 10px;
    margin-bottom: 8px; font-size: 12px;
  }
  .rank-row .rnum{ color: var(--dim); margin-right: 8px; font-size: 11px; }
  .rank-row.me{ border-color: var(--yellow); box-shadow: 0 0 14px rgba(255,210,63,0.2); }
  .rank-row .rpts{ color: var(--cyan); font-size: 11px; }
  .rank-summary{
    text-align:center; font-size: 11px; color: var(--dim); margin-top: -6px; margin-bottom: 4px;
  }
  .rank-summary .num{ color: var(--yellow); }

  /* --- celebration overlay --- */
  .celebration-overlay{
    position:absolute; inset:0; overflow:hidden; pointer-events:none; z-index: 5;
  }
  .celeb-particle{
    position:absolute; top:-24px; animation-name: celebFall; animation-timing-function: ease-in;
    animation-fill-mode: forwards; will-change: transform, opacity;
  }
  @keyframes celebFall{
    0%{ transform: translateY(0) rotate(0deg) scale(1); opacity:1; }
    100%{ transform: translateY(360px) rotate(360deg) scale(0.9); opacity:0; }
  }
  .celeb-title{
    text-align:center; font-family:'Press Start 2P', monospace;
    letter-spacing: 1px; margin-top: 4px; position: relative; z-index: 6;
  }
  .celeb-rainbow{
    background: linear-gradient(90deg, var(--pink), var(--yellow), var(--cyan), var(--pink));
    background-size: 250% auto; -webkit-background-clip: text; background-clip: text;
    color: transparent; animation: celebRainbow 1.4s linear infinite;
  }
  @keyframes celebRainbow{ to{ background-position: 250% center; } }
  .celeb-shake{ animation: celebShake 0.5s ease-in-out; }
  @keyframes celebShake{
    0%, 100%{ transform: translateX(0); }
    20%{ transform: translateX(-6px); }
    40%{ transform: translateX(6px); }
    60%{ transform: translateX(-4px); }
    80%{ transform: translateX(4px); }
  }
  .celeb-flash::before{
    content:""; position:absolute; inset:0; background:#fff; opacity:0;
    animation: celebFlashPulse 0.5s ease-out; z-index: 4; pointer-events:none;
  }
  @keyframes celebFlashPulse{ 0%{ opacity:0.85; } 100%{ opacity:0; } }

  /* --- shop celebration preview: fixed to the viewport so it's visible no matter
     how far down the (long) shop list the person has scrolled --- */
  .celeb-preview-wrap{
    position: fixed; inset: 0; z-index: 9999; pointer-events: none; overflow: hidden;
  }
  .celeb-preview-title-wrap{
    position: absolute; inset: 0; display: flex; align-items: center; justify-content: center;
    padding: 0 24px; text-align: center;
  }

  /* --- opponent reactions --- */
  .reaction-bar{
    display:flex; justify-content:center; gap:8px; margin-top:16px; flex-wrap:wrap;
  }
  .reaction-btn{
    background: var(--panel); border: 1px solid rgba(255,255,255,0.08);
    font-size: 20px; line-height:1; padding: 9px 11px; border-radius: 10px;
  }
  .reaction-btn:hover{
    border-color: var(--cyan); box-shadow: 0 0 12px rgba(43,227,255,0.3);
    transform: translateY(-2px);
  }
  .reaction-hint{
    text-align:center; font-size: 9px; color: var(--dim); letter-spacing: 1px; margin-top: 6px;
  }
  .emoji-toast-wrap{
    position: fixed; bottom: 64px; left: 0; right: 0;
    display:flex; justify-content:center; z-index: 9999; pointer-events:none;
  }
  .emoji-toast{
    font-size: 56px;
    animation: emojiPop 1.6s ease-out forwards;
    filter: drop-shadow(0 4px 10px rgba(0,0,0,0.5));
  }
  @keyframes emojiPop{
    0%{ transform: scale(0.3) translateY(20px); opacity:0; }
    15%{ transform: scale(1.2) translateY(0); opacity:1; }
    25%{ transform: scale(1) translateY(0); opacity:1; }
    80%{ transform: scale(1) translateY(0); opacity:1; }
    100%{ transform: scale(0.8) translateY(-30px); opacity:0; }
  }

  @media (prefers-reduced-motion: reduce){
    *{ animation: none !important; transition: none !important; }
  }
</style>
</head>
<body>

<div>
  <div class="marquee">
    <h1>&#9733; GAME HUT &#9733;</h1>
    <p>ARCADE CABINET &mdash; 2 GAMES ON ONE BOARD</p>
  </div>

  <div class="cabinet">
    <div class="screen">
      <div class="screen-content" id="screen"></div>
    </div>
  </div>

  <div class="footer-hint"><span class="blink">&#9679;</span> INSERT SKILL, NOT COINS</div>
</div>

<script>
const screen = document.getElementById('screen');

function beep(freq = 440, dur = 0.08, type = 'square'){
  try{
    const ctx = beep.ctx || (beep.ctx = new (window.AudioContext || window.webkitAudioContext)());
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.type = type;
    osc.frequency.value = freq;
    gain.gain.setValueAtTime(0.05, ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + dur);
    osc.connect(gain).connect(ctx.destination);
    osc.start();
    osc.stop(ctx.currentTime + dur);
  } catch(e){}
}

function render(html, extraClass){
  screen.style.animation = 'none';
  screen.className = 'screen-content' + (extraClass ? (' ' + extraClass) : '');
  void screen.offsetWidth;
  screen.style.animation = 'powerOn 0.35s ease-out';
  screen.innerHTML = html;
}

// ---------- SUPABASE CONFIG ----------
// Fill these in from your Supabase project: Settings -> API
const SUPABASE_URL = 'https://wuabblulcagkvbovmdam.supabase.co';        // e.g. https://xxxxx.supabase.co;        // e.g. https://xxxxx.supabase.co
const SUPABASE_ANON_KEY = 'sb_publishable_dyCUUeWO1Ctu5a25h0li6A__LeI0jnt';

const supabaseClient = (SUPABASE_URL.startsWith('http'))
  ? supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY)
  : null;

async function storageGet(key){
  if(!supabaseClient) return null;
  try{
    const { data, error } = await supabaseClient
      .from('kv_store')
      .select('value')
      .eq('key', key)
      .maybeSingle();
    if(error) return null;
    return data ? data.value : null;
  } catch(e){
    return null; // treat "not found" (or any read failure) as no value
  }
}
async function storageSet(key, value){
  if(!supabaseClient) return false;
  try{
    const { error } = await supabaseClient
      .from('kv_store')
      .upsert({ key, value, updated_at: new Date().toISOString() });
    if(error){ console.error('storage error', error); return false; }
    return true;
  } catch(e){
    console.error('storage error', e);
    return false;
  }
}

// Fetches every stored account row (keys prefixed "acct_") so we can build a
// global leaderboard. Returns an array of parsed account objects.
async function getAllAccounts(){
  if(!supabaseClient) return [];
  try{
    const { data, error } = await supabaseClient
      .from('kv_store')
      .select('key,value')
      .like('key', 'acct\_%');
    if(error || !data) return [];
    return data
      .map(row => {
        try{ return JSON.parse(row.value); } catch(e){ return null; }
      })
      .filter(a => a && typeof a.username === 'string');
  } catch(e){
    return [];
  }
}

// ---------- ACCOUNTS / LEVELING / FRIENDS ----------
let currentAccount = null; // { username, pin, level, xp, wins, friends: [] }

function acctKey(username){ return `acct_${username.toLowerCase()}`; }
function xpForLevel(level){ return (level - 1) * 100; } // xp needed to REACH this level
function levelFromXp(xp){ return Math.floor(xp / 100) + 1; }

async function saveAccount(){
  if(!currentAccount) return;
  await storageSet(acctKey(currentAccount.username), JSON.stringify(currentAccount));
}

// Award XP, coins, points, and match points for a win. Returns true if this win caused a level-up.
// pointsGain feeds the global leaderboard ranking (separate from XP/level).
async function addWin(xpGain, coinsGain, pointsGain){
  if(!currentAccount) return false; // guest mode, no progress tracked
  const prevLevel = currentAccount.level;
  currentAccount.wins += 1;
  currentAccount.xp += xpGain;
  currentAccount.level = levelFromXp(currentAccount.xp);
  if(coinsGain){ currentAccount.coins = (currentAccount.coins || 0) + coinsGain; }
  if(pointsGain){ currentAccount.points = (currentAccount.points || 0) + pointsGain; }
  await saveAccount();
  return currentAccount.level > prevLevel;
}

// Deduct XP, coins, and ranking points for a loss. All are clamped at 0. Level is recalculated (can go down).
async function addLoss(xpLoss, coinsLoss, pointsLoss){
  if(!currentAccount) return; // guest mode, no progress tracked
  currentAccount.xp = Math.max(0, currentAccount.xp - xpLoss);
  currentAccount.level = levelFromXp(currentAccount.xp);
  if(coinsLoss){ currentAccount.coins = Math.max(0, (currentAccount.coins || 0) - coinsLoss); }
  if(pointsLoss){ currentAccount.points = Math.max(0, (currentAccount.points || 0) - pointsLoss); }
  await saveAccount();
}

// Ranking points awarded/lost per match, used for the leaderboard (separate from XP/coins).
const RANK_POINTS = { win: 30, lose: 20 };

// Random coin reward for RPS wins (1-5 coins)
function rpsWinCoins(){
  return Math.floor(Math.random() * 5) + 1;
}

// Fixed coin reward for Number Hunt wins, keyed by difficulty
const NH_COINS = { too_easy: 1, easy: 5, medium: 6, hard: 17 };

// Coins lost on any loss (RPS always uses this flat amount)
const LOSS_COINS = 2;

// Coins lost on a Number Hunt loss, keyed by difficulty (HARD costs much more)
const NH_LOSS_COINS = { too_easy: 2, easy: 2, medium: 5, hard: 15 };

// ---------- OPPONENT REACTIONS (quick emojis sent during an online match) ----------
const QUICK_EMOJIS = ['&#128077;', '&#128514;', '&#128557;', '&#128293;', '&#128548;', '&#128075;'];

let emojiPollTimer = null;
let emojiSeenTs = 0;

function stopEmojiPolling(){
  if(emojiPollTimer){ clearInterval(emojiPollTimer); emojiPollTimer = null; }
}

// Starts (or restarts) listening for reactions sent to me in the given room.
// gameType is 'rps' or 'nh'; code is the room code; myField is 'p1' or 'p2'.
function startEmojiPolling(gameType, code, myField){
  stopEmojiPolling();
  emojiSeenTs = 0;
  if(!code || !myField) return;
  const key = `${gameType}_${code}_emoji_${myField}`;
  emojiPollTimer = setInterval(async () => {
    const raw = await storageGet(key);
    if(!raw) return;
    let msg;
    try{ msg = JSON.parse(raw); } catch(e){ return; }
    if(msg && msg.ts && msg.ts > emojiSeenTs){
      emojiSeenTs = msg.ts;
      showEmojiToast(msg.emoji);
    }
  }, 1200);
}

// Sends a quick emoji reaction to the opponent (targetField is their 'p1'/'p2' slot).
async function sendReaction(gameType, code, targetField, emoji){
  beep(500, 0.05);
  await storageSet(`${gameType}_${code}_emoji_${targetField}`, JSON.stringify({ emoji, ts: Date.now() }));
}

let emojiToastTimer = null;
function showEmojiToast(emoji){
  const old = document.querySelector('.emoji-toast-wrap');
  if(old) old.remove();
  if(emojiToastTimer) clearTimeout(emojiToastTimer);
  const wrap = document.createElement('div');
  wrap.className = 'emoji-toast-wrap';
  wrap.innerHTML = `<div class="emoji-toast">${emoji}</div>`;
  document.body.appendChild(wrap);
  emojiToastTimer = setTimeout(() => wrap.remove(), 1600);
}

function reactionBarHtml(gameType, code, oppField){
  return `
    <div class="reaction-bar">
      ${QUICK_EMOJIS.map(e => `<button class="reaction-btn" onclick="sendReaction('${gameType}','${code}','${oppField}','${e}')">${e}</button>`).join('')}
    </div>
    <div class="reaction-hint">SEND A REACTION</div>
  `;
}

// ---------- CELEBRATIONS (bought with coins, played on any win) ----------
const CELEBRATIONS = [
  { id: 1,  cost: 0,  name: 'CLASSIC FLASH',  tier: 'starter',   emojis: ['&#10022;'],                              count: 8,  dur: 1.1 },
  { id: 2,  cost: 5,  name: 'CONFETTI POP',   tier: 'starter',   emojis: ['&#127881;'],                             count: 12, dur: 1.3 },
  { id: 3,  cost: 8,  name: 'STAR SHOWER',    tier: 'starter',   emojis: ['&#11088;','&#10024;'],                   count: 16, dur: 1.4 },
  { id: 4,  cost: 10, name: 'RAINBOW BURST',  tier: 'mid',       emojis: ['&#127882;','&#10024;','&#11088;'],       count: 20, dur: 1.5, rainbow: true },
  { id: 5,  cost: 12, name: 'SPARKLE GLOW',   tier: 'mid',       emojis: ['&#10024;','&#128171;'],                  count: 24, dur: 1.6, rainbow: true },
  { id: 6,  cost: 16, name: 'FIREWORKS',      tier: 'premium',   emojis: ['&#127878;','&#10024;'],                  count: 34, dur: 1.9, rainbow: true, shake: true },
  { id: 7,  cost: 23, name: 'GALAXY BLAST',   tier: 'premium',   emojis: ['&#127775;','&#128165;','&#10024;'],      count: 46, dur: 2.2, rainbow: true, shake: true, flash: true },
  { id: 8,  cost: 26, name: 'GOLD RUSH',      tier: 'premium',   emojis: ['&#127942;','&#128176;','&#10024;'],      count: 58, dur: 2.5, rainbow: true, shake: true, flash: true, waves: 2 },
  { id: 9,  cost: 29, name: 'SUPERNOVA',      tier: 'premium',   emojis: ['&#128165;','&#127775;','&#9889;'],       count: 72, dur: 2.8, rainbow: true, shake: true, flash: true, waves: 2 },
  { id: 10, cost: 36, name: 'LEGENDARY',      tier: 'premium',   emojis: ['&#128081;','&#127878;','&#128142;','&#9889;','&#10024;'], count: 90, dur: 3.2, rainbow: true, shake: true, flash: true, waves: 3, epic: true },
  { id: 11, cost: 40, name: 'METEOR SHOWER',  tier: 'mythic',    emojis: ['&#127755;','&#128293;','&#10024;'],      count: 98,  dur: 3.4, rainbow: true, shake: true, flash: true, waves: 3, epic: true },
  { id: 12, cost: 43, name: 'COSMIC STORM',   tier: 'mythic',    emojis: ['&#127775;','&#9889;','&#128167;'],       count: 106, dur: 3.6, rainbow: true, shake: true, flash: true, waves: 3, epic: true },
  { id: 13, cost: 47, name: 'PHOENIX RISING', tier: 'mythic',    emojis: ['&#128293;','&#129413;','&#10024;'],      count: 114, dur: 3.8, rainbow: true, shake: true, flash: true, waves: 4, epic: true },
  { id: 14, cost: 50, name: 'DRAGON FLAME',   tier: 'mythic',    emojis: ['&#128009;','&#128293;','&#9889;'],       count: 122, dur: 4.0, rainbow: true, shake: true, flash: true, waves: 4, epic: true },
  { id: 15, cost: 54, name: 'THUNDER GOD',    tier: 'mythic',    emojis: ['&#9889;','&#127785;','&#128168;'],       count: 130, dur: 4.2, rainbow: true, shake: true, flash: true, waves: 4, mega: true },
  { id: 16, cost: 57, name: 'CELESTIAL CROWN',tier: 'mythic',    emojis: ['&#128081;','&#127775;','&#10024;','&#128142;'], count: 138, dur: 4.4, rainbow: true, shake: true, flash: true, waves: 5, mega: true },
  { id: 17, cost: 61, name: 'ETERNAL BLAZE',  tier: 'mythic',    emojis: ['&#128293;','&#127942;','&#9889;','&#10024;'],   count: 146, dur: 4.6, rainbow: true, shake: true, flash: true, waves: 5, mega: true },
  { id: 18, cost: 64, name: 'INFINITY BURST', tier: 'mythic',    emojis: ['&#128165;','&#127775;','&#128142;','&#9889;'],  count: 154, dur: 4.8, rainbow: true, shake: true, flash: true, waves: 5, mega: true },
  { id: 19, cost: 67, name: 'ASCENSION',      tier: 'mythic',    emojis: ['&#128081;','&#128293;','&#127775;','&#9889;','&#10024;'], count: 162, dur: 5.0, rainbow: true, shake: true, flash: true, waves: 6, mega: true },
  { id: 20, cost: 70, name: 'ULTIMATE GODMODE', tier: 'mythic',  emojis: ['&#128081;','&#128165;','&#128293;','&#127775;','&#128142;','&#9889;','&#10024;'], count: 175, dur: 5.5, rainbow: true, shake: true, flash: true, waves: 6, mega: true },
];

function getCelebration(id){
  return CELEBRATIONS.find(c => c.id === id) || CELEBRATIONS[0];
}

// Ensure older saved accounts get the new coin/celebration/points fields
function ensureAccountDefaults(acct){
  if(typeof acct.coins !== 'number') acct.coins = 0;
  if(typeof acct.points !== 'number') acct.points = 0;
  if(!Array.isArray(acct.ownedCelebrations)) acct.ownedCelebrations = [1];
  if(!acct.ownedCelebrations.includes(1)) acct.ownedCelebrations.push(1);
  if(typeof acct.selectedCelebration !== 'number') acct.selectedCelebration = 1;
  return acct;
}

// Builds the HTML for a celebration overlay + title banner, to drop into a win screen.
function renderCelebrationMarkup(celebrationId){
  const c = getCelebration(celebrationId);
  const waves = c.waves || 1;
  let particles = '';
  for(let w = 0; w < waves; w++){
    for(let i = 0; i < c.count; i++){
      const emoji = c.emojis[Math.floor(Math.random() * c.emojis.length)];
      const left = Math.random() * 100;
      const delay = (w * 0.3) + Math.random() * 0.5;
      const dur = c.dur * (0.7 + Math.random() * 0.6);
      const size = c.mega ? (20 + Math.random() * 20) : (c.epic ? (16 + Math.random() * 14) : (12 + Math.random() * 10));
      particles += `<span class="celeb-particle" style="left:${left}%; font-size:${size}px; animation-duration:${dur}s; animation-delay:${delay}s;">${emoji}</span>`;
    }
  }
  const titleClasses = ['celeb-title'];
  if(c.rainbow) titleClasses.push('celeb-rainbow');
  const titleSize = c.mega ? 16 : (c.epic ? 13 : (c.tier === 'premium' ? 11 : 9));
  const titleStyle = c.rainbow ? `font-size:${titleSize}px;` : `font-size:${titleSize}px; color: var(--yellow); text-shadow: 0 0 10px rgba(255,210,63,0.6);`;
  return {
    overlay: `<div class="celebration-overlay">${particles}</div>`,
    title: `<div class="${titleClasses.join(' ')}" style="${titleStyle}">${c.name}</div>`,
    screenExtraClass: `${c.shake ? 'celeb-shake ' : ''}${c.flash ? 'celeb-flash' : ''}`.trim()
  };
}

function levelUpBanner(leveledUp){
  if(!leveledUp) return '';
  return `<div class="level-up">&#9733; LEVEL UP! YOU ARE NOW LEVEL ${currentAccount.level} &#9733;</div>`;
}

function accountBar(){
  if(!currentAccount) return '';
  const inLevelXp = currentAccount.xp - xpForLevel(currentAccount.level);
  const pct = Math.min(100, inLevelXp);
  return `
    <div class="account-bar">
      <div style="flex:1; min-width: 140px;">
        <div class="who">${currentAccount.username} &middot; LV <span class="lvl">${currentAccount.level}</span> &middot; ${currentAccount.wins} WINS</div>
        <div class="xpbar"><div class="xpbar-fill" style="width:${pct}%;"></div></div>
      </div>
      <span class="coin-pill">&#129689; ${currentAccount.coins || 0}</span>
      <span class="points-pill">&#127942; ${currentAccount.points || 0} PTS</span>
      <button class="mini-btn" onclick="beep(300); showLeaderboard()">RANK</button>
      <button class="mini-btn" onclick="beep(300); showShop()">SHOP</button>
      <button class="mini-btn" onclick="beep(300); showFriends()">FRIENDS</button>
    </div>
  `;
}

function showLogin(){
  render(`
    <h2 class="title">GAME HUT ACCOUNT</h2>
    <p style="color:var(--dim); font-size:12px; margin-top:-10px;">
      pick a callsign &amp; 4-digit PIN &mdash; new callsigns are created automatically
    </p>
    <div class="row">
      <input type="text" id="acctUser" placeholder="CALLSIGN" maxlength="14">
      <input type="password" id="acctPin" placeholder="PIN" maxlength="4" inputmode="numeric">
    </div>
    <label style="display:flex; align-items:center; gap:8px; margin-top:10px; font-size:11px; color:var(--dim); cursor:pointer;">
      <input type="checkbox" id="rememberMe" checked style="width:auto; accent-color: var(--pink);">
      remember me on this device (up to 4 accounts)
    </label>
    <div class="menu-row" style="margin-top:12px;">
      <button class="primary" onclick="beep(520); attemptLogin()">LOGIN / CREATE</button>
      <button class="ghost" onclick="beep(300); currentAccount=null; showMenu()">PLAY AS GUEST</button>
    </div>
    <div class="feedback hint" id="acctMsg"></div>
    <p style="color:var(--dim); font-size:9px; margin-top:14px; line-height:1.5;">
      accounts are stored in a shared database so friends can find each other by callsign &mdash;
      it's a lightweight identity for this game, not secure login. don't reuse a real password as your PIN.
      "remember me" saves your callsign and PIN in this browser only, on this device.
    </p>
    ${getRememberedLogins().length ? `<a class="back-link" onclick="beep(300); showAccountSwitcher()">&#8592; BACK TO SAVED ACCOUNTS</a>` : ''}
  `);
  const userInput = document.getElementById('acctUser');
  userInput.focus();
  const go = e => { if(e.key === 'Enter') attemptLogin(); };
  userInput.addEventListener('keydown', go);
  document.getElementById('acctPin').addEventListener('keydown', go);
}

async function attemptLogin(){
  const msg = document.getElementById('acctMsg');
  const username = (document.getElementById('acctUser').value || '').trim();
  const pin = (document.getElementById('acctPin').value || '').trim();
  const rememberMe = document.getElementById('rememberMe')
    ? document.getElementById('rememberMe').checked
    : false;

  if(!username || !/^[A-Za-z0-9_]{2,14}$/.test(username)){
    msg.textContent = 'callsign: 2-14 letters, numbers or _';
    return;
  }
  if(!/^\d{4}$/.test(pin)){
    msg.textContent = 'PIN must be exactly 4 digits';
    return;
  }

  msg.textContent = 'checking...';
  const raw = await storageGet(acctKey(username));

  if(raw){
    let acct;
    try{ acct = JSON.parse(raw); } catch(e){ msg.textContent = 'account data error, try another callsign'; return; }
    if(acct.pin !== pin){
      msg.textContent = 'wrong PIN for that callsign';
      return;
    }
    if(!Array.isArray(acct.friends)) acct.friends = [];
    ensureAccountDefaults(acct);
    currentAccount = acct;
    if(rememberMe) addRememberedLogin(username, pin);
    beep(660, 0.15);
    showMenu();
  } else {
    currentAccount = { username, pin, level: 1, xp: 0, wins: 0, friends: [], coins: 0, points: 0, ownedCelebrations: [1], selectedCelebration: 1 };
    const ok = await saveAccount();
    if(!ok){ msg.textContent = 'network error, try again'; currentAccount = null; return; }
    if(rememberMe) addRememberedLogin(username, pin);
    beep(880, 0.15);
    showMenu();
  }
}

// ---------- SAVED ACCOUNTS / SWITCHER (local to this browser/device only, up to 4) ----------
const REMEMBER_KEY = 'gamehut_remembered_logins';
const MAX_REMEMBERED = 4;

function getRememberedLogins(){
  try{
    const raw = localStorage.getItem(REMEMBER_KEY);
    const list = raw ? JSON.parse(raw) : [];
    return Array.isArray(list) ? list : [];
  } catch(e){
    return [];
  }
}

function saveRememberedLogins(list){
  try{ localStorage.setItem(REMEMBER_KEY, JSON.stringify(list.slice(0, MAX_REMEMBERED))); } catch(e){}
}

function addRememberedLogin(username, pin){
  let list = getRememberedLogins().filter(l => l.username.toLowerCase() !== username.toLowerCase());
  list.unshift({ username, pin });
  saveRememberedLogins(list);
}

function removeSavedAccount(username){
  const list = getRememberedLogins().filter(l => l.username.toLowerCase() !== username.toLowerCase());
  saveRememberedLogins(list);
  showAccountSwitcher();
}

function showAccountSwitcher(){
  const logins = getRememberedLogins();
  if(!logins.length){ showLogin(); return; }

  render(`
    <h2 class="title">WHO'S PLAYING?</h2>
    <p style="color:var(--dim); font-size:12px; margin-top:-10px;">saved accounts on this device (${logins.length}/${MAX_REMEMBERED})</p>
    <div class="menu-row">
      ${logins.map(l => `
        <div class="row" style="align-items:stretch;">
          <button class="game-btn" style="flex:1;" onclick="beep(520); quickLoginAs('${l.username}')">${l.username}</button>
          <button class="ghost" style="flex:0 0 auto; padding:0 14px;" onclick="beep(300); removeSavedAccount('${l.username}')">&#10005;</button>
        </div>
      `).join('')}
      ${logins.length < MAX_REMEMBERED
        ? `<button class="ghost" onclick="beep(300); showLogin()">+ ADD ANOTHER ACCOUNT</button>`
        : `<p style="color:var(--dim); font-size:10px; text-align:center; margin-top:4px;">4-account limit reached &mdash; remove one to add another</p>`}
    </div>
    <a class="back-link" onclick="beep(300); currentAccount=null; showMenu()">&#8592; PLAY AS GUEST</a>
  `);
}

async function quickLoginAs(username){
  const entry = getRememberedLogins().find(l => l.username === username);
  if(!entry){ showAccountSwitcher(); return; }

  render(`
    <h2 class="title">GAME HUT ACCOUNT</h2>
    <p class="feedback hint" style="text-align:center; margin-top:30px;"><span class="blink">&#9679;</span> logging in as ${entry.username}&hellip;</p>
  `);

  const raw = await storageGet(acctKey(entry.username));
  if(!raw){ removeSavedAccount(entry.username); return; }

  let acct;
  try{ acct = JSON.parse(raw); } catch(e){ showAccountSwitcher(); return; }

  if(acct.pin !== entry.pin){ removeSavedAccount(entry.username); return; }

  if(!Array.isArray(acct.friends)) acct.friends = [];
  ensureAccountDefaults(acct);
  currentAccount = acct;
  beep(660, 0.15);
  showMenu();
}

function logout(){
  currentAccount = null;
  stopOnlinePolling();
  stopNHPolling();
  showAccountSwitcher();
}

function showFriends(){
  if(!currentAccount){ showLogin(); return; }
  render(`
    <h2 class="title">FRIENDS</h2>
    ${accountBar()}
    <p style="color:var(--dim); font-size:11px; margin-top:-4px;">
      share your callsign <span style="color:var(--yellow);">${currentAccount.username}</span> so friends can add you
    </p>
    <div class="row">
      <input type="text" id="friendInput" placeholder="ADD BY CALLSIGN" maxlength="14">
      <button class="game-btn" style="flex:0 0 auto;" onclick="beep(440); addFriend()">ADD</button>
    </div>
    <div class="feedback hint" id="friendMsg"></div>
    <div id="friendList" style="margin-top:14px;">
      <p style="color:var(--dim); font-size:11px;">loading friends&hellip;</p>
    </div>
    <a class="back-link" onclick="beep(300); showMenu()">&#8592; BACK TO MENU</a>
  `);
  loadFriendList();
}

async function addFriend(){
  const msg = document.getElementById('friendMsg');
  const raw = (document.getElementById('friendInput').value || '').trim();
  const uname = raw.toLowerCase();
  if(!uname){ msg.textContent = 'enter a callsign'; return; }
  if(uname === currentAccount.username.toLowerCase()){ msg.textContent = "that's you!"; return; }
  if(currentAccount.friends.map(f => f.toLowerCase()).includes(uname)){ msg.textContent = 'already friends'; return; }

  msg.textContent = 'looking them up...';
  const theirRaw = await storageGet(acctKey(uname));
  if(!theirRaw){ msg.textContent = 'no account with that callsign'; return; }

  let theirAcct;
  try{ theirAcct = JSON.parse(theirRaw); } catch(e){ msg.textContent = 'account data error'; return; }

  currentAccount.friends.push(theirAcct.username);
  await saveAccount();
  beep(660);
  document.getElementById('friendInput').value = '';
  msg.textContent = `added ${theirAcct.username}!`;
  loadFriendList();
}

async function loadFriendList(){
  const listEl = document.getElementById('friendList');
  if(!listEl) return;
  if(!currentAccount.friends.length){
    listEl.innerHTML = `<p style="color:var(--dim); font-size:11px;">no friends added yet</p>`;
    return;
  }
  const rows = await Promise.all(currentAccount.friends.map(async (name) => {
    const raw = await storageGet(acctKey(name));
    if(!raw) return `<div class="friend-row"><span>${name}</span><span class="lvl">(no data)</span></div>`;
    try{
      const f = JSON.parse(raw);
      return `<div class="friend-row"><span>${f.username}</span><span class="lvl">LV ${f.level} &middot; ${f.wins}W</span></div>`;
    } catch(e){
      return `<div class="friend-row"><span>${name}</span><span class="lvl">(no data)</span></div>`;
    }
  }));
  listEl.innerHTML = rows.join('');
}

// ---------- LEADERBOARD (ranking points, separate from XP/level) ----------
async function showLeaderboard(){
  if(!currentAccount){ showLogin(); return; }

  render(`
    ${accountBar()}
    <h2 class="title">LEADERBOARD</h2>
    <p class="feedback hint" style="text-align:center; margin-top:20px;"><span class="blink">&#9679;</span> loading rankings&hellip;</p>
    <a class="back-link" onclick="beep(300); showMenu()">&#8592; BACK TO MENU</a>
  `);

  const accounts = await getAllAccounts();
  accounts.forEach(a => { if(typeof a.points !== 'number') a.points = 0; });
  accounts.sort((a, b) => b.points - a.points);

  const myIndex = accounts.findIndex(a => a.username.toLowerCase() === currentAccount.username.toLowerCase());
  const myRank = myIndex >= 0 ? myIndex + 1 : null;
  const total = accounts.length;
  const top = accounts.slice(0, 15);

  const rowsHtml = top.map((a, i) => {
    const rank = i + 1;
    const isMe = a.username.toLowerCase() === currentAccount.username.toLowerCase();
    return `
      <div class="rank-row ${isMe ? 'me' : ''}">
        <span><span class="rnum">#${rank}</span>${a.username}${isMe ? ' (YOU)' : ''}</span>
        <span class="rpts">${a.points || 0} PTS</span>
      </div>
    `;
  }).join('');

  render(`
    ${accountBar()}
    <h2 class="title">LEADERBOARD</h2>
    <div class="rank-summary">
      ${myRank
        ? `you are ranked <span class="num">#${myRank}</span> of <span class="num">${total}</span> players`
        : `play a match to get ranked`}
    </div>
    <div style="margin-top:8px;">
      ${rowsHtml || `<p style="color:var(--dim); font-size:11px; text-align:center;">no ranked players yet &mdash; be the first!</p>`}
    </div>
    <p style="color:var(--dim); font-size:9px; margin-top:10px; text-align:center;">
      win a match: +${RANK_POINTS.win} pts &middot; lose a match: -${RANK_POINTS.lose} pts
    </p>
    <a class="back-link" onclick="beep(300); showMenu()">&#8592; BACK TO MENU</a>
  `);
}

// ---------- SHOP (spend coins on win celebrations) ----------
function showShop(){
  if(!currentAccount){ showLogin(); return; }
  ensureAccountDefaults(currentAccount);
  render(`
    <h2 class="title">CELEBRATION SHOP</h2>
    ${accountBar()}
    <p style="color:var(--dim); font-size:11px; margin-top:-6px;">spend coins on flashier win celebrations &mdash; earn coins by winning</p>
    <div class="shop-list">
      ${CELEBRATIONS.map(c => {
        const owned = currentAccount.ownedCelebrations.includes(c.id);
        const selected = currentAccount.selectedCelebration === c.id;
        const canAfford = (currentAccount.coins || 0) >= c.cost;
        let btn;
        if(selected){
          btn = `<button class="shop-btn selected-btn" disabled>SELECTED</button>`;
        } else if(owned){
          btn = `<button class="shop-btn owned" onclick="beep(440); selectCelebration(${c.id})">SELECT</button>`;
        } else if(canAfford){
          btn = `<button class="shop-btn" onclick="beep(520); buyCelebration(${c.id})">BUY &middot; ${c.cost} &#129689;</button>`;
        } else {
          btn = `<button class="shop-btn" disabled>${c.cost} &#129689;</button>`;
        }
        return `
          <div class="shop-row ${selected ? 'selected' : ''}">
            <div class="info">
              <div class="cname">${c.id}. ${c.name}</div>
              <div class="ctier">${c.cost === 0 ? 'FREE' : c.cost + ' COINS'} &middot; ${c.tier.toUpperCase()}</div>
            </div>
            <div class="shop-btn-row">
              <button class="preview-btn" onclick="beep(400); previewCelebration(${c.id})">&#128065; PREVIEW</button>
              ${btn}
            </div>
          </div>
        `;
      }).join('')}
    </div>
    <a class="back-link" onclick="beep(300); showMenu()">&#8592; BACK TO MENU</a>
  `);
}

async function buyCelebration(id){
  ensureAccountDefaults(currentAccount);
  const c = getCelebration(id);
  if(currentAccount.ownedCelebrations.includes(id)) return;
  if((currentAccount.coins || 0) < c.cost) return;
  currentAccount.coins -= c.cost;
  currentAccount.ownedCelebrations.push(id);
  currentAccount.selectedCelebration = id;
  await saveAccount();
  showShop();
}

async function selectCelebration(id){
  ensureAccountDefaults(currentAccount);
  if(!currentAccount.ownedCelebrations.includes(id)) return;
  currentAccount.selectedCelebration = id;
  await saveAccount();
  showShop();
}

// Plays a celebration's overlay/title/shake/flash as a fixed overlay pinned to the
// current viewport — so it's visible immediately, no matter how far down the shop
// list you've scrolled. Doesn't require ownership. Used by the shop's PREVIEW buttons.
let previewCleanupTimer = null;
function previewCelebration(id){
  const c = getCelebration(id);
  const celeb = renderCelebrationMarkup(id);

  // clear any previous preview still running
  if(previewCleanupTimer) clearTimeout(previewCleanupTimer);
  const oldWrap = document.querySelector('.celeb-preview-wrap');
  if(oldWrap) oldWrap.remove();

  const wrap = document.createElement('div');
  wrap.className = 'celeb-preview-wrap' + (celeb.screenExtraClass ? (' ' + celeb.screenExtraClass) : '');
  wrap.innerHTML = `${celeb.overlay}<div class="celeb-preview-title-wrap">${celeb.title}</div>`;
  document.body.appendChild(wrap);

  const waves = c.waves || 1;
  const totalMs = (c.dur * 1.3 + waves * 0.3) * 1000 + 400;
  previewCleanupTimer = setTimeout(() => {
    wrap.remove();
  }, totalMs);
}

// ---------- MENU ----------
function showMenu(){
  stopNHPolling();
  stopOnlinePolling();
  render(`
    ${accountBar()}
    <div id="inviteBanner"></div>
    <h2 class="title">SELECT GAME</h2>
    <div class="menu-row">
      <button class="game-btn" onclick="beep(520); showNumberHuntHub()">
        <span>NUMBER HUNT<br><small style="color:var(--dim); font-weight:400;">solo or race a friend online</small></span>
        <span class="arrow">&#9654;</span>
      </button>
      <button class="game-btn" onclick="beep(520); showRPSHub()">
        <span>ROCK &middot; PAPER &middot; SCISSORS<br><small style="color:var(--dim); font-weight:400;">vs CPU or a friend online</small></span>
        <span class="arrow">&#9654;</span>
      </button>
    </div>
    ${currentAccount
      ? `<a class="back-link" onclick="beep(300); logout()">&#8592; SWITCH ACCOUNT</a>`
      : `<a class="back-link" onclick="beep(300); showAccountSwitcher()">&#8592; LOGIN TO TRACK LEVELS</a>`}
  `);
  checkAndShowInviteBanner();
}

// ---------- ROOM INVITES (sent by callsign, checked on the main menu) ----------
async function checkAndShowInviteBanner(){
  if(!currentAccount) return;
  const raw = await storageGet(`invite_${currentAccount.username.toLowerCase()}`);
  if(!raw) return;

  let invite;
  try{ invite = JSON.parse(raw); } catch(e){ return; }
  if(!invite || !invite.code || !invite.game) return;

  const host = document.getElementById('inviteBanner');
  if(!host) return;

  const gameLabel = invite.game === 'rps' ? 'ROCK PAPER SCISSORS' : 'NUMBER HUNT';
  host.innerHTML = `
    <div class="account-bar" style="border-color: var(--pink);">
      <div style="flex:1;">
        <div class="who">${invite.from} invited you to ${gameLabel}!</div>
      </div>
      <button class="mini-btn" onclick="beep(520); acceptInvite('${invite.game}', '${invite.code}')">JOIN</button>
      <button class="mini-btn" onclick="beep(300); dismissInvite()">&#10005;</button>
    </div>
  `;
}

async function dismissInvite(){
  if(!currentAccount) return;
  await storageSet(`invite_${currentAccount.username.toLowerCase()}`, '');
  const host = document.getElementById('inviteBanner');
  if(host) host.innerHTML = '';
}

async function acceptInvite(game, code){
  await dismissInvite();

  if(game === 'rps'){
    const ok = await storageSet(`rps_${code}_p2`, 'waiting');
    if(!ok){ showMenu(); return; }
    online = { code, role: 'p2', pollTimer: null };
    startEmojiPolling('rps', code, 'p2');
    showOnlineChoice();
  } else if(game === 'nh'){
    const metaRaw = await storageGet(`nh_${code}_meta`);
    if(!metaRaw){ showMenu(); return; }
    let meta;
    try{ meta = JSON.parse(metaRaw); } catch(e){ showMenu(); return; }
    const ok = await storageSet(`nh_${code}_p2`, 'joined');
    if(!ok){ showMenu(); return; }
    nh = { code, role: 'p2', meta, triesLeft: meta.tries, attempts: 0, pollTimer: null };
    startEmojiPolling('nh', code, 'p2');
    showNHRace();
  }
}

// ---------- NUMBER HUNT ----------
function showNumberHuntHub(){
  stopNHPolling();
  render(`
    ${accountBar()}
    <h2 class="title">NUMBER HUNT</h2>
    <p style="color:var(--dim); font-size:12px; margin-top:-10px;">choose how you want to play</p>
    <div class="menu-row">
      <button class="game-btn" onclick="beep(520); showDifficulty()">
        <span>SOLO PLAY<br><small style="color:var(--dim); font-weight:400;">guess against the clock, no rush</small></span>
        <span class="arrow">&#9654;</span>
      </button>
      <button class="game-btn" onclick="beep(520); showNHOnlineMenu()">
        <span>RACE ONLINE<br><small style="color:var(--dim); font-weight:400;">same number, fewest guesses wins</small></span>
        <span class="arrow">&#9654;</span>
      </button>
    </div>
    <a class="back-link" onclick="beep(300); showMenu()">&#8592; BACK TO MENU</a>
  `);
}

const DIFFS = {
  too_easy: { label: 'ROOKIE',  lo: 1, hi: 10,   tries: 3  },
  easy:     { label: 'EASY',    lo: 1, hi: 100,  tries: 7  },
  medium:   { label: 'MEDIUM',  lo: 1, hi: 500,  tries: 10 },
  hard:     { label: 'HARD',    lo: 1, hi: 1000, tries: 12 },
};

// How close a guess has to be to just say "low"/"high" instead of "TOO low"/"TOO high", per difficulty.
const CLOSE_RANGE = {
  too_easy: 3,
  easy: 10,
  medium: 20,
  hard: 50,
};

// Only say "TOO low/high" when the guess is far outside the close range for this difficulty.
function guessHint(val, target, diffKey){
  const diff = Math.abs(val - target);
  const direction = val < target ? 'LOW' : 'HIGH';
  const arrow = val < target ? '&#8593;' : '&#8595;';
  const threshold = CLOSE_RANGE[diffKey] ?? 1;
  const prefix = diff <= threshold ? '' : 'TOO ';
  return `${prefix}${direction} ${arrow}`;
}

function showDifficulty(){
  render(`
    <h2 class="title">NUMBER HUNT</h2>
    <p style="color:var(--dim); font-size:12px; margin-top:-10px;">choose your difficulty</p>
    <div class="diff-grid">
      ${Object.entries(DIFFS).map(([key, d]) => `
        <button class="diff-btn" onclick="beep(440); startGuess('${key}')">
          ${d.label}<small>${d.lo}&ndash;${d.hi} &bull; ${d.tries} tries &bull; lose ${NH_LOSS_COINS[key]} &#129689; if you fail</small>
        </button>
      `).join('')}
    </div>
    <a class="back-link" onclick="beep(300); showNumberHuntHub()">&#8592; BACK</a>
  `);
}

let guessState = null;

function startGuess(key){
  const d = DIFFS[key];
  guessState = { ...d, diffKey: key, target: Math.floor(Math.random() * (d.hi - d.lo + 1)) + d.lo, triesLeft: d.tries };
  paintGuess('');
}

function paintGuess(feedback, feedbackClass){
  const s = guessState;
  render(`
    <h2 class="title">NUMBER HUNT &mdash; ${s.label}</h2>
    <div class="hud">
      <span>RANGE <span class="val">${s.lo}&ndash;${s.hi}</span></span>
      <span class="lives">${'&#9679;'.repeat(s.triesLeft)}${'&#9675;'.repeat(s.tries - s.triesLeft)}</span>
    </div>
    ${s.diffKey === 'hard' ? `<p style="color:var(--pink); font-size:10px; text-align:center; margin-top:-8px; letter-spacing:1px;">&#9888; HARD MODE: LOSE ${NH_LOSS_COINS.hard} &#129689; IF YOU FAIL</p>` : ''}
    <div class="feedback ${feedbackClass || ''}">${feedback || ''}</div>
    <div class="row">
      <input type="number" id="guessInput" placeholder="your guess" autofocus>
      <button class="primary" style="flex:0 0 auto;" onclick="submitGuess()">GUESS</button>
    </div>
    <a class="back-link" onclick="beep(300); showDifficulty()">&#8592; CHANGE DIFFICULTY</a>
  `);
  const input = document.getElementById('guessInput');
  input.focus();
  input.addEventListener('keydown', e => { if(e.key === 'Enter') submitGuess(); });
}

async function submitGuess(){
  const input = document.getElementById('guessInput');
  const val = parseInt(input.value, 10);
  const s = guessState;
  if(isNaN(val)){
    paintGuess('enter a valid number', 'hint');
    return;
  }
  if(val === s.target){
    beep(880, 0.15);
    const coinsGain = NH_COINS[s.diffKey] || 0;
    const leveledUp = await addWin(20, coinsGain, RANK_POINTS.win);
    const celeb = currentAccount ? renderCelebrationMarkup(currentAccount.selectedCelebration) : null;
    render(`
      ${celeb ? celeb.overlay : ''}
      <h2 class="title">NUMBER HUNT &mdash; ${s.label}</h2>
      <div class="feedback win" style="text-align:center; font-size:18px; margin-top:40px;">
        &#9733; CORRECT! &#9733;<br>
        <span style="font-size:13px; color:var(--ink);">the number was ${s.target}</span>
        ${currentAccount ? `<div style="font-size:11px; color:var(--dim); margin-top:6px;">+20 XP &middot; +${coinsGain} &#129689; &middot; +${RANK_POINTS.win} PTS</div>` : ''}
      </div>
      ${celeb ? celeb.title : ''}
      ${levelUpBanner(leveledUp)}
      <div class="menu-row" style="margin-top:24px;">
        <button class="primary" onclick="beep(520); showDifficulty()">PLAY AGAIN</button>
        <a class="back-link" style="text-align:center;" onclick="beep(300); showMenu()">&#8592; MAIN MENU</a>
      </div>
    `, celeb ? celeb.screenExtraClass : '');
    return;
  }
  s.triesLeft -= 1;
  if(s.triesLeft <= 0){
    beep(140, 0.25, 'sawtooth');
    const coinsLoss = NH_LOSS_COINS[s.diffKey] ?? LOSS_COINS;
    await addLoss(10, coinsLoss, RANK_POINTS.lose);
    render(`
      <h2 class="title">NUMBER HUNT &mdash; ${s.label}</h2>
      <div class="feedback lose" style="text-align:center; font-size:18px; margin-top:40px;">
        GAME OVER<br>
        <span style="font-size:13px; color:var(--ink);">the number was ${s.target}</span>
        ${currentAccount ? `<div style="font-size:11px; color:var(--dim); margin-top:6px;">-10 XP &middot; -${coinsLoss} &#129689; &middot; -${RANK_POINTS.lose} PTS</div>` : ''}
      </div>
      <div class="menu-row" style="margin-top:24px;">
        <button class="primary" onclick="beep(520); showDifficulty()">TRY AGAIN</button>
        <a class="back-link" style="text-align:center;" onclick="beep(300); showMenu()">&#8592; MAIN MENU</a>
      </div>
    `);
    return;
  }
  beep(300);
  const hint = guessHint(val, s.target, s.diffKey);
  paintGuess(hint, 'hint');
}

// ---------- ROCK PAPER SCISSORS ----------
const RPS = {
  rock:     { emoji: '&#129704;', beats: 'scissors' },
  paper:    { emoji: '&#128220;', beats: 'rock' },
  scissors: { emoji: '&#9986;',  beats: 'paper' },
};
let score = { win: 0, lose: 0, tie: 0 };

function showRPSHub(){
  stopOnlinePolling();
  render(`
    ${accountBar()}
    <h2 class="title">ROCK &middot; PAPER &middot; SCISSORS</h2>
    <p style="color:var(--dim); font-size:12px; margin-top:-10px;">choose how you want to play</p>
    <div class="menu-row">
      <button class="game-btn" onclick="beep(520); showRPSCPU()">
        <span>VS CPU<br><small style="color:var(--dim); font-weight:400;">instant, offline</small></span>
        <span class="arrow">&#9654;</span>
      </button>
      <button class="game-btn" onclick="beep(520); showRPSOnlineMenu()">
        <span>PLAY ONLINE<br><small style="color:var(--dim); font-weight:400;">challenge a friend with a room code</small></span>
        <span class="arrow">&#9654;</span>
      </button>
    </div>
    <a class="back-link" onclick="beep(300); showMenu()">&#8592; BACK TO MENU</a>
  `);
}

function showRPSCPU(){
  render(`
    <h2 class="title">ROCK &middot; PAPER &middot; SCISSORS</h2>
    <div class="rps-grid">
      ${Object.entries(RPS).map(([key, r]) => `
        <button class="rps-btn" onclick="playRPS('${key}')">
          ${r.emoji}<span class="rps-label">${key.toUpperCase()}</span>
        </button>
      `).join('')}
    </div>
    <div class="score">W <span>${score.win}</span> &nbsp;&middot;&nbsp; L <span>${score.lose}</span> &nbsp;&middot;&nbsp; T <span>${score.tie}</span></div>
    <a class="back-link" onclick="beep(300); showRPSHub()">&#8592; BACK</a>
  `);
}

async function playRPS(userKey){
  const keys = Object.keys(RPS);
  const cpuKey = keys[Math.floor(Math.random() * keys.length)];
  let result, resultClass, leveledUp = false, xpNote = '', celeb = null;

  if(userKey === cpuKey){
    result = "IT'S A TIE"; resultClass = 'hint'; score.tie++; beep(400);
  } else if(RPS[userKey].beats === cpuKey){
    result = 'YOU WIN'; resultClass = 'win'; score.win++; beep(880, 0.15);
    const coinsGain = rpsWinCoins();
    leveledUp = await addWin(15, coinsGain, RANK_POINTS.win);
    xpNote = ` <span style="font-size:11px; color:var(--dim);">(+15 XP &middot; +${coinsGain} &#129689; &middot; +${RANK_POINTS.win} PTS)</span>`;
    if(currentAccount) celeb = renderCelebrationMarkup(currentAccount.selectedCelebration);
  } else {
    result = 'YOU LOSE'; resultClass = 'lose'; score.lose++; beep(140, 0.25, 'sawtooth');
    await addLoss(10, LOSS_COINS, RANK_POINTS.lose);
    xpNote = ` <span style="font-size:11px; color:var(--dim);">(-10 XP &middot; -${LOSS_COINS} &#129689; &middot; -${RANK_POINTS.lose} PTS)</span>`;
  }

  render(`
    ${celeb ? celeb.overlay : ''}
    <h2 class="title">ROCK &middot; PAPER &middot; SCISSORS</h2>
    <div class="versus">
      <div>
        <div class="choice">${RPS[userKey].emoji}</div>
        <div class="who">YOU</div>
      </div>
      <div class="vs">VS</div>
      <div>
        <div class="choice">${RPS[cpuKey].emoji}</div>
        <div class="who">CPU</div>
      </div>
    </div>
    <div class="feedback ${resultClass}" style="text-align:center; font-size:16px;">
      ${result}${currentAccount ? xpNote : ''}
    </div>
    ${celeb ? celeb.title : ''}
    ${levelUpBanner(leveledUp)}
    <div class="score">W <span>${score.win}</span> &nbsp;&middot;&nbsp; L <span>${score.lose}</span> &nbsp;&middot;&nbsp; T <span>${score.tie}</span></div>
    <div class="menu-row" style="margin-top:16px;">
      <button class="primary" onclick="beep(520); showRPSCPU()">PLAY AGAIN</button>
      <a class="back-link" style="text-align:center;" onclick="beep(300); showMenu()">&#8592; MAIN MENU</a>
    </div>
  `, celeb ? celeb.screenExtraClass : '');
}

// ---------- RPS ONLINE (shared, cross-device via room codes) ----------
const CODE_CHARS = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789'; // no 0/O/1/I, avoids confusion
let online = { code: null, role: null, pollTimer: null };

function randCode(){
  let c = '';
  for(let i = 0; i < 4; i++) c += CODE_CHARS[Math.floor(Math.random() * CODE_CHARS.length)];
  return c;
}

// Clears only the RPS game-state poll (used, join, result checks) without touching
// the separate emoji-reaction poll, so re-arming it on screen changes doesn't
// interrupt incoming reactions.
function stopOnlineGamePoll(){
  if(online.pollTimer){ clearInterval(online.pollTimer); online.pollTimer = null; }
}

function stopOnlinePolling(){
  stopOnlineGamePoll();
  stopEmojiPolling();
}

function showRPSOnlineMenu(){
  stopOnlinePolling();
  online = { code: null, role: null, pollTimer: null };
  render(`
    <h2 class="title">PLAY ONLINE</h2>
    <p style="color:var(--dim); font-size:12px; margin-top:-10px;">
      rooms use shared storage &mdash; anyone with the code can join
    </p>
    <div class="menu-row">
      <button class="primary" onclick="beep(520); quickMatchRPS()">QUICK MATCH<br><small style="font-weight:400;">get paired with a random opponent</small></button>
      <button class="game-btn" onclick="beep(520); createRoom()">CREATE A PRIVATE ROOM</button>
      <div class="row">
        <input type="text" id="joinCodeInput" placeholder="ENTER CODE" maxlength="4" style="text-transform:uppercase;">
        <button class="game-btn" style="flex:0 0 auto;" onclick="beep(440); joinRoom()">JOIN</button>
      </div>
    </div>
    <div class="feedback hint" id="onlineMsg"></div>
    <a class="back-link" onclick="beep(300); showRPSHub()">&#8592; BACK</a>
  `);
}

// ---------- RPS QUICK MATCH (auto-pairs with a random waiting opponent) ----------
async function quickMatchRPS(){
  render(`
    <h2 class="title">QUICK MATCH</h2>
    <p class="feedback hint" style="text-align:center; margin-top:30px;"><span class="blink">&#9679;</span> searching for an opponent&hellip;</p>
  `);

  const queueRaw = await storageGet('rps_quickmatch_queue');
  if(queueRaw){
    // someone already posted a room to the queue - try to claim it
    await storageSet('rps_quickmatch_queue', '');
    const code = queueRaw;
    const p1 = await storageGet(`rps_${code}_p1`);
    const p2existing = await storageGet(`rps_${code}_p2`);
    if(p1 && !p2existing){
      const ok = await storageSet(`rps_${code}_p2`, 'waiting');
      if(ok){
        online = { code, role: 'p2', pollTimer: null };
        startEmojiPolling('rps', code, 'p2');
        beep(520);
        showOnlineChoice();
        return;
      }
    }
    // queued room was stale or already taken - fall through and open a new one
  }

  let code, free = false;
  for(let i = 0; i < 6; i++){
    code = randCode();
    const existing = await storageGet(`rps_${code}_p1`);
    if(!existing){ free = true; break; }
  }
  if(!free){ showRPSOnlineMenu(); return; }

  await storageSet(`rps_${code}_p1`, 'waiting');
  await storageSet(`rps_${code}_p2`, '');
  await storageSet('rps_quickmatch_queue', code);

  online = { code, role: 'p1', pollTimer: null };
  startEmojiPolling('rps', code, 'p1');
  showRPSQuickWaiting();
}

function showRPSQuickWaiting(){
  render(`
    <h2 class="title">QUICK MATCH</h2>
    <p class="feedback hint" style="text-align:center; margin-top:30px;"><span class="blink">&#9679;</span> searching for an opponent&hellip;</p>
    <a class="back-link" onclick="beep(300); cancelRPSQuickMatch()">&#8592; CANCEL</a>
  `);
  stopOnlineGamePoll();
  online.pollTimer = setInterval(async () => {
    const p2 = await storageGet(`rps_${online.code}_p2`);
    if(p2){ stopOnlineGamePoll(); beep(520); showOnlineChoice(); }
  }, 1500);
}

async function cancelRPSQuickMatch(){
  stopOnlinePolling();
  const q = await storageGet('rps_quickmatch_queue');
  if(q === online.code) await storageSet('rps_quickmatch_queue', '');
  showRPSOnlineMenu();
}

async function createRoom(){
  const msg = document.getElementById('onlineMsg');
  msg.textContent = 'creating room...';
  let code, free = false;
  for(let i = 0; i < 6; i++){
    code = randCode();
    const existing = await storageGet(`rps_${code}_p1`);
    if(!existing){ free = true; break; }
  }
  if(!free){ msg.textContent = 'could not create a room, try again'; return; }

  const ok = await storageSet(`rps_${code}_p1`, 'waiting');
  await storageSet(`rps_${code}_p2`, '');
  if(!ok){ msg.textContent = 'network error, try again'; return; }

  online = { code, role: 'p1', pollTimer: null };
  startEmojiPolling('rps', code, 'p1');
  showWaitingForOpponent();
}

async function joinRoom(){
  const input = document.getElementById('joinCodeInput');
  const code = (input.value || '').trim().toUpperCase();
  const msg = document.getElementById('onlineMsg');
  if(code.length !== 4){ msg.textContent = 'enter the 4-character code'; return; }

  msg.textContent = 'joining...';
  const p1 = await storageGet(`rps_${code}_p1`);
  if(!p1){ msg.textContent = 'no room found with that code'; return; }
  const p2 = await storageGet(`rps_${code}_p2`);
  if(p2){ msg.textContent = 'that room is already full'; return; }

  const ok = await storageSet(`rps_${code}_p2`, 'waiting');
  if(!ok){ msg.textContent = 'network error, try again'; return; }

  online = { code, role: 'p2', pollTimer: null };
  startEmojiPolling('rps', code, 'p2');
  showOnlineChoice();
}

function showWaitingForOpponent(){
  render(`
    <h2 class="title">ROOM ${online.code}</h2>
    <div style="text-align:center; margin-top:20px;">
      <div style="font-size:34px; letter-spacing:6px; color:var(--yellow); text-shadow:0 0 10px rgba(255,210,63,0.5);">${online.code}</div>
      <p style="color:var(--dim); font-size:12px; margin-top:14px;">send this to your friends, or enter their callsign to invite them directly</p>
      <button class="ghost" onclick="beep(300); copyRoomCode('${online.code}')">COPY CODE</button>
    </div>
    <div class="row" style="margin-top:16px;">
      <input type="text" id="inviteCallsignInput" placeholder="FRIEND'S CALLSIGN" maxlength="14">
      <button class="game-btn" style="flex:0 0 auto;" onclick="beep(440); sendRPSInvite()">INVITE</button>
    </div>
    <div class="feedback hint" id="inviteMsg"></div>
    <p class="feedback hint" style="text-align:center; margin-top:10px;"><span class="blink">&#9679;</span> waiting for them to join&hellip;</p>
    ${reactionBarHtml('rps', online.code, 'p2')}
    <a class="back-link" onclick="beep(300); stopOnlinePolling(); showRPSOnlineMenu()">&#8592; CANCEL</a>
  `);
  stopOnlineGamePoll();
  online.pollTimer = setInterval(async () => {
    const p2 = await storageGet(`rps_${online.code}_p2`);
    if(p2){ stopOnlineGamePoll(); beep(520); showOnlineChoice(); }
  }, 1500);
}

function copyRoomCode(code){
  try{
    navigator.clipboard.writeText(code);
  } catch(e){}
}

async function sendRPSInvite(){
  const input = document.getElementById('inviteCallsignInput');
  const msg = document.getElementById('inviteMsg');
  const uname = (input.value || '').trim();

  if(!uname){ msg.textContent = 'enter a callsign'; return; }
  if(!currentAccount){ msg.textContent = 'log in to send invites'; return; }
  if(uname.toLowerCase() === currentAccount.username.toLowerCase()){ msg.textContent = "that's you!"; return; }

  msg.textContent = 'looking them up...';
  const theirRaw = await storageGet(acctKey(uname));
  if(!theirRaw){ msg.textContent = 'no account with that callsign'; return; }

  const ok = await storageSet(`invite_${uname.toLowerCase()}`, JSON.stringify({
    from: currentAccount.username, game: 'rps', code: online.code, ts: Date.now()
  }));
  if(!ok){ msg.textContent = 'network error, try again'; return; }

  beep(660);
  msg.textContent = `invite sent to ${uname}!`;
  input.value = '';
}

function showOnlineChoice(){
  stopOnlineGamePoll();
  const oppField = online.role === 'p1' ? 'p2' : 'p1';
  render(`
    <h2 class="title">ROOM ${online.code}</h2>
    <p style="color:var(--dim); font-size:12px; margin-top:-10px;">make your move</p>
    <div class="rps-grid">
      ${Object.entries(RPS).map(([key, r]) => `
        <button class="rps-btn" onclick="submitOnlineChoice('${key}')">
          ${r.emoji}<span class="rps-label">${key.toUpperCase()}</span>
        </button>
      `).join('')}
    </div>
    ${reactionBarHtml('rps', online.code, oppField)}
    <a class="back-link" onclick="beep(300); stopOnlinePolling(); showRPSHub()">&#8592; LEAVE ROOM</a>
  `);
}

async function submitOnlineChoice(myKey){
  beep(440);
  const myField = online.role === 'p1' ? 'p1' : 'p2';
  const oppField = online.role === 'p1' ? 'p2' : 'p1';
  await storageSet(`rps_${online.code}_${myField}`, myKey);

  render(`
    <h2 class="title">ROOM ${online.code}</h2>
    <div class="versus">
      <div>
        <div class="choice">${RPS[myKey].emoji}</div>
        <div class="who">YOU</div>
      </div>
      <div class="vs">VS</div>
      <div>
        <div class="choice">?</div>
        <div class="who">OPPONENT</div>
      </div>
    </div>
    <p class="feedback hint" style="text-align:center;"><span class="blink">&#9679;</span> waiting for opponent&hellip;</p>
    ${reactionBarHtml('rps', online.code, oppField)}
    <a class="back-link" onclick="beep(300); stopOnlinePolling(); showRPSHub()">&#8592; LEAVE ROOM</a>
  `);

  stopOnlineGamePoll();
  online.pollTimer = setInterval(async () => {
    const oppVal = await storageGet(`rps_${online.code}_${oppField}`);
    if(oppVal && RPS[oppVal]){
      stopOnlineGamePoll();
      showOnlineResult(myKey, oppVal, myField, oppField);
    }
  }, 1500);
}

async function showOnlineResult(myKey, oppKey, myField, oppField){
  let result, resultClass, leveledUp = false, xpNote = '', celeb = null;
  if(myKey === oppKey){ result = "IT'S A TIE"; resultClass = 'hint'; beep(400); }
  else if(RPS[myKey].beats === oppKey){
    result = 'YOU WIN'; resultClass = 'win'; beep(880, 0.15);
    const coinsGain = rpsWinCoins();
    leveledUp = await addWin(30, coinsGain, RANK_POINTS.win);
    xpNote = ` <span style="font-size:11px; color:var(--dim);">(+30 XP &middot; +${coinsGain} &#129689; &middot; +${RANK_POINTS.win} PTS)</span>`;
    if(currentAccount) celeb = renderCelebrationMarkup(currentAccount.selectedCelebration);
  }
  else {
    result = 'YOU LOSE'; resultClass = 'lose'; beep(140, 0.25, 'sawtooth');
    await addLoss(10, LOSS_COINS, RANK_POINTS.lose);
    xpNote = ` <span style="font-size:11px; color:var(--dim);">(-10 XP &middot; -${LOSS_COINS} &#129689; &middot; -${RANK_POINTS.lose} PTS)</span>`;
  }

  render(`
    ${celeb ? celeb.overlay : ''}
    <h2 class="title">ROOM ${online.code}</h2>
    <div class="versus">
      <div>
        <div class="choice">${RPS[myKey].emoji}</div>
        <div class="who">YOU</div>
      </div>
      <div class="vs">VS</div>
      <div>
        <div class="choice">${RPS[oppKey].emoji}</div>
        <div class="who">OPPONENT</div>
      </div>
    </div>
    <div class="feedback ${resultClass}" style="text-align:center; font-size:16px;">
      ${result}${currentAccount ? xpNote : ''}
    </div>
    ${celeb ? celeb.title : ''}
    ${levelUpBanner(leveledUp)}
    ${reactionBarHtml('rps', online.code, oppField)}
    <div class="menu-row" style="margin-top:16px;">
      <button class="primary" onclick="beep(520); rematchOnline('${myField}')">REMATCH</button>
      <a class="back-link" style="text-align:center;" onclick="beep(300); stopOnlinePolling(); showRPSHub()">&#8592; LEAVE ROOM</a>
    </div>
  `, celeb ? celeb.screenExtraClass : '');
}

async function rematchOnline(myField){
  await storageSet(`rps_${online.code}_${myField}`, 'waiting');
  const oppField = myField === 'p1' ? 'p2' : 'p1';
  render(`
    <h2 class="title">ROOM ${online.code}</h2>
    <p class="feedback hint" style="text-align:center; margin-top:30px;"><span class="blink">&#9679;</span> waiting for opponent to rematch&hellip;</p>
    ${reactionBarHtml('rps', online.code, oppField)}
    <a class="back-link" onclick="beep(300); stopOnlinePolling(); showRPSHub()">&#8592; LEAVE ROOM</a>
  `);
  stopOnlineGamePoll();
  online.pollTimer = setInterval(async () => {
    const oppVal = await storageGet(`rps_${online.code}_${oppField}`);
    if(oppVal === 'waiting'){ stopOnlineGamePoll(); showOnlineChoice(); }
  }, 1500);
}

// ---------- NUMBER HUNT ONLINE (shared, cross-device via room codes) ----------
let nh = { code: null, role: null, meta: null, triesLeft: 0, attempts: 0, pollTimer: null };

function stopNHGamePoll(){
  if(nh.pollTimer){ clearInterval(nh.pollTimer); nh.pollTimer = null; }
}

function stopNHPolling(){
  stopNHGamePoll();
  stopEmojiPolling();
}

function showNHOnlineMenu(){
  stopNHPolling();
  nh = { code: null, role: null, meta: null, triesLeft: 0, attempts: 0, pollTimer: null };
  render(`
    <h2 class="title">RACE ONLINE</h2>
    <p style="color:var(--dim); font-size:12px; margin-top:-10px;">
      both players guess the same hidden number &mdash; fewest guesses wins
    </p>
    <div class="menu-row">
      <button class="primary" onclick="beep(520); showNHQuickDifficulty()">QUICK MATCH<br><small style="font-weight:400;">get paired with a random opponent</small></button>
      <button class="game-btn" onclick="beep(520); showNHCreateDifficulty()">CREATE A PRIVATE ROOM</button>
      <div class="row">
        <input type="text" id="nhJoinCodeInput" placeholder="ENTER CODE" maxlength="4" style="text-transform:uppercase;">
        <button class="game-btn" style="flex:0 0 auto;" onclick="beep(440); joinNHRoom()">JOIN</button>
      </div>
    </div>
    <div class="feedback hint" id="nhMsg"></div>
    <a class="back-link" onclick="beep(300); showNumberHuntHub()">&#8592; BACK</a>
  `);
}

// ---------- NH QUICK MATCH (auto-pairs with a random waiting opponent, matched by difficulty) ----------
function showNHQuickDifficulty(){
  render(`
    <h2 class="title">QUICK MATCH</h2>
    <p style="color:var(--dim); font-size:12px; margin-top:-10px;">choose a range &mdash; you'll be paired with anyone else searching at that range</p>
    <div class="diff-grid">
      ${Object.entries(DIFFS).map(([key, d]) => `
        <button class="diff-btn" onclick="beep(440); quickMatchNH('${key}')">
          ${d.label}<small>${d.lo}&ndash;${d.hi} &bull; ${d.tries} tries</small>
        </button>
      `).join('')}
    </div>
    <a class="back-link" onclick="beep(300); showNHOnlineMenu()">&#8592; BACK</a>
  `);
}

async function quickMatchNH(diffKey){
  const d = DIFFS[diffKey];
  const queueKey = `nh_quickmatch_${diffKey}`;

  render(`
    <h2 class="title">QUICK MATCH &mdash; ${d.label}</h2>
    <p class="feedback hint" style="text-align:center; margin-top:30px;"><span class="blink">&#9679;</span> searching for an opponent&hellip;</p>
  `);

  const queueRaw = await storageGet(queueKey);
  if(queueRaw){
    await storageSet(queueKey, '');
    const code = queueRaw;
    const metaRaw = await storageGet(`nh_${code}_meta`);
    const p2existing = await storageGet(`nh_${code}_p2`);
    if(metaRaw && !p2existing){
      const meta = JSON.parse(metaRaw);
      const ok = await storageSet(`nh_${code}_p2`, 'joined');
      if(ok){
        nh = { code, role: 'p2', meta, triesLeft: meta.tries, attempts: 0, pollTimer: null };
        startEmojiPolling('nh', code, 'p2');
        beep(520);
        showNHRace();
        return;
      }
    }
    // queued room was stale or already taken - fall through and open a new one
  }

  let code, free = false;
  for(let i = 0; i < 6; i++){
    code = randCode();
    const existing = await storageGet(`nh_${code}_meta`);
    if(!existing){ free = true; break; }
  }
  if(!free){ showNHOnlineMenu(); return; }

  const target = Math.floor(Math.random() * (d.hi - d.lo + 1)) + d.lo;
  const meta = { label: d.label, lo: d.lo, hi: d.hi, tries: d.tries, target, diffKey };
  await storageSet(`nh_${code}_meta`, JSON.stringify(meta));
  await storageSet(`nh_${code}_p1`, 'joined');
  await storageSet(`nh_${code}_p2`, '');
  await storageSet(`nh_${code}_p1_result`, '');
  await storageSet(`nh_${code}_p2_result`, '');
  await storageSet(queueKey, code);

  nh = { code, role: 'p1', meta, triesLeft: d.tries, attempts: 0, pollTimer: null };
  startEmojiPolling('nh', code, 'p1');
  showNHQuickWaiting(queueKey);
}

function showNHQuickWaiting(queueKey){
  render(`
    <h2 class="title">QUICK MATCH &mdash; ${nh.meta.label}</h2>
    <p class="feedback hint" style="text-align:center; margin-top:30px;"><span class="blink">&#9679;</span> searching for an opponent&hellip;</p>
    <a class="back-link" onclick="beep(300); cancelNHQuickMatch('${queueKey}')">&#8592; CANCEL</a>
  `);
  stopNHGamePoll();
  nh.pollTimer = setInterval(async () => {
    const p2 = await storageGet(`nh_${nh.code}_p2`);
    if(p2){ stopNHGamePoll(); beep(520); showNHRace(); }
  }, 1500);
}

async function cancelNHQuickMatch(queueKey){
  stopNHPolling();
  const q = await storageGet(queueKey);
  if(q === nh.code) await storageSet(queueKey, '');
  showNHOnlineMenu();
}

function showNHCreateDifficulty(){
  render(`
    <h2 class="title">RACE ONLINE</h2>
    <p style="color:var(--dim); font-size:12px; margin-top:-10px;">choose the range for this race</p>
    <div class="diff-grid">
      ${Object.entries(DIFFS).map(([key, d]) => `
        <button class="diff-btn" onclick="beep(440); createNHRoom('${key}')">
          ${d.label}<small>${d.lo}&ndash;${d.hi} &bull; ${d.tries} tries</small>
        </button>
      `).join('')}
    </div>
    <a class="back-link" onclick="beep(300); showNHOnlineMenu()">&#8592; BACK</a>
  `);
}

async function createNHRoom(key){
  const d = DIFFS[key];
  let code, free = false;
  for(let i = 0; i < 6; i++){
    code = randCode();
    const existing = await storageGet(`nh_${code}_meta`);
    if(!existing){ free = true; break; }
  }
  if(!free){ return; }

  const target = Math.floor(Math.random() * (d.hi - d.lo + 1)) + d.lo;
  const meta = { label: d.label, lo: d.lo, hi: d.hi, tries: d.tries, target, diffKey: key };
  await storageSet(`nh_${code}_meta`, JSON.stringify(meta));
  await storageSet(`nh_${code}_p1`, 'joined');
  await storageSet(`nh_${code}_p2`, '');
  await storageSet(`nh_${code}_p1_result`, '');
  await storageSet(`nh_${code}_p2_result`, '');

  nh = { code, role: 'p1', meta, triesLeft: d.tries, attempts: 0, pollTimer: null };
  startEmojiPolling('nh', code, 'p1');
  showNHWaitingForOpponent();
}

async function joinNHRoom(){
  const input = document.getElementById('nhJoinCodeInput');
  const code = (input.value || '').trim().toUpperCase();
  const msg = document.getElementById('nhMsg');
  if(code.length !== 4){ msg.textContent = 'enter the 4-character code'; return; }

  msg.textContent = 'joining...';
  const metaRaw = await storageGet(`nh_${code}_meta`);
  if(!metaRaw){ msg.textContent = 'no room found with that code'; return; }
  const p2 = await storageGet(`nh_${code}_p2`);
  if(p2){ msg.textContent = 'that room is already full'; return; }

  const meta = JSON.parse(metaRaw);
  const ok = await storageSet(`nh_${code}_p2`, 'joined');
  if(!ok){ msg.textContent = 'network error, try again'; return; }

  nh = { code, role: 'p2', meta, triesLeft: meta.tries, attempts: 0, pollTimer: null };
  startEmojiPolling('nh', code, 'p2');
  showNHRace();
}

function showNHWaitingForOpponent(){
  render(`
    <h2 class="title">ROOM ${nh.code}</h2>
    <div style="text-align:center; margin-top:20px;">
      <div style="font-size:34px; letter-spacing:6px; color:var(--yellow); text-shadow:0 0 10px rgba(255,210,63,0.5);">${nh.code}</div>
      <p style="color:var(--dim); font-size:12px; margin-top:14px;">send this to your friends, or enter their callsign to invite them directly</p>
      <button class="ghost" onclick="beep(300); copyRoomCode('${nh.code}')">COPY CODE</button>
    </div>
    <div class="row" style="margin-top:16px;">
      <input type="text" id="nhInviteCallsignInput" placeholder="FRIEND'S CALLSIGN" maxlength="14">
      <button class="game-btn" style="flex:0 0 auto;" onclick="beep(440); sendNHInvite()">INVITE</button>
    </div>
    <div class="feedback hint" id="nhInviteMsg"></div>
    <p class="feedback hint" style="text-align:center; margin-top:10px;"><span class="blink">&#9679;</span> waiting for them to join&hellip;</p>
    ${reactionBarHtml('nh', nh.code, 'p2')}
    <a class="back-link" onclick="beep(300); stopNHPolling(); showNHOnlineMenu()">&#8592; CANCEL</a>
  `);
  stopNHGamePoll();
  nh.pollTimer = setInterval(async () => {
    const p2 = await storageGet(`nh_${nh.code}_p2`);
    if(p2){ stopNHGamePoll(); beep(520); showNHRace(); }
  }, 1500);
}

async function sendNHInvite(){
  const input = document.getElementById('nhInviteCallsignInput');
  const msg = document.getElementById('nhInviteMsg');
  const uname = (input.value || '').trim();

  if(!uname){ msg.textContent = 'enter a callsign'; return; }
  if(!currentAccount){ msg.textContent = 'log in to send invites'; return; }
  if(uname.toLowerCase() === currentAccount.username.toLowerCase()){ msg.textContent = "that's you!"; return; }

  msg.textContent = 'looking them up...';
  const theirRaw = await storageGet(acctKey(uname));
  if(!theirRaw){ msg.textContent = 'no account with that callsign'; return; }

  const ok = await storageSet(`invite_${uname.toLowerCase()}`, JSON.stringify({
    from: currentAccount.username, game: 'nh', code: nh.code, ts: Date.now()
  }));
  if(!ok){ msg.textContent = 'network error, try again'; return; }

  beep(660);
  msg.textContent = `invite sent to ${uname}!`;
  input.value = '';
}

function showNHRace(feedback, feedbackClass){
  stopNHGamePoll();
  const m = nh.meta;
  const oppField = nh.role === 'p1' ? 'p2' : 'p1';
  render(`
    <h2 class="title">ROOM ${nh.code} &mdash; ${m.label}</h2>
    <div class="hud">
      <span>RANGE <span class="val">${m.lo}&ndash;${m.hi}</span></span>
      <span class="lives">${'&#9679;'.repeat(nh.triesLeft)}${'&#9675;'.repeat(m.tries - nh.triesLeft)}</span>
    </div>
    ${m.diffKey === 'hard' ? `<p style="color:var(--pink); font-size:10px; text-align:center; margin-top:-8px; letter-spacing:1px;">&#9888; HARD MODE: LOSE ${NH_LOSS_COINS.hard} &#129689; IF YOU FAIL</p>` : ''}
    <div class="feedback ${feedbackClass || ''}">${feedback || ''}</div>
    <div class="row">
      <input type="number" id="nhGuessInput" placeholder="your guess" autofocus>
      <button class="primary" style="flex:0 0 auto;" onclick="submitNHGuess()">GUESS</button>
    </div>
    ${reactionBarHtml('nh', nh.code, oppField)}
    <a class="back-link" onclick="beep(300); stopNHPolling(); showNumberHuntHub()">&#8592; LEAVE ROOM</a>
  `);
  const inputEl = document.getElementById('nhGuessInput');
  inputEl.focus();
  inputEl.addEventListener('keydown', e => { if(e.key === 'Enter') submitNHGuess(); });
}

async function submitNHGuess(){
  const inputEl = document.getElementById('nhGuessInput');
  const val = parseInt(inputEl.value, 10);
  const m = nh.meta;
  if(isNaN(val)){ showNHRace('enter a valid number', 'hint'); return; }

  nh.attempts += 1;
  if(val === m.target){
    beep(880, 0.15);
    await recordNHResult(true, nh.attempts);
    return;
  }
  nh.triesLeft -= 1;
  if(nh.triesLeft <= 0){
    beep(140, 0.25, 'sawtooth');
    await recordNHResult(false, nh.attempts);
    return;
  }
  beep(300);
  const hint = guessHint(val, m.target, m.diffKey);
  showNHRace(hint, 'hint');
}

async function recordNHResult(solved, attempts){
  const myField = nh.role;
  const oppField = nh.role === 'p1' ? 'p2' : 'p1';
  await storageSet(`nh_${nh.code}_${myField}_result`, JSON.stringify({ solved, attempts }));
  showNHWaitingForResult(solved, attempts, oppField);
}

function showNHWaitingForResult(mySolved, myAttempts, oppField){
  render(`
    <h2 class="title">ROOM ${nh.code}</h2>
    <div class="feedback ${mySolved ? 'win' : 'lose'}" style="text-align:center; font-size:16px; margin-top:20px;">
      ${mySolved ? `SOLVED IN ${myAttempts} ${myAttempts === 1 ? 'GUESS' : 'GUESSES'}` : `OUT OF TRIES &mdash; NUMBER WAS ${nh.meta.target}`}
    </div>
    <p class="feedback hint" style="text-align:center;"><span class="blink">&#9679;</span> waiting for opponent to finish&hellip;</p>
    ${reactionBarHtml('nh', nh.code, oppField)}
    <a class="back-link" onclick="beep(300); stopNHPolling(); showNumberHuntHub()">&#8592; LEAVE ROOM</a>
  `);
  stopNHGamePoll();
  nh.pollTimer = setInterval(async () => {
    const oppRaw = await storageGet(`nh_${nh.code}_${oppField}_result`);
    if(oppRaw){
      stopNHGamePoll();
      const opp = JSON.parse(oppRaw);
      showNHFinalResult(mySolved, myAttempts, opp.solved, opp.attempts);
    }
  }, 1500);
}

async function showNHFinalResult(mySolved, myAttempts, oppSolved, oppAttempts){
  let result, resultClass, leveledUp = false, xpNote = '', celeb = null;
  const iWon = (mySolved && oppSolved && myAttempts < oppAttempts) || (mySolved && !oppSolved);
  const isTie = (mySolved && oppSolved && myAttempts === oppAttempts) || (!mySolved && !oppSolved);

  if(isTie){ result = "IT'S A TIE"; resultClass = 'hint'; beep(400); }
  else if(iWon){
    result = 'YOU WIN'; resultClass = 'win'; beep(880, 0.15);
    const coinsGain = NH_COINS[nh.meta.diffKey] || 0;
    leveledUp = await addWin(30, coinsGain, RANK_POINTS.win);
    xpNote = ` <span style="font-size:11px; color:var(--dim);">(+30 XP &middot; +${coinsGain} &#129689; &middot; +${RANK_POINTS.win} PTS)</span>`;
    if(currentAccount) celeb = renderCelebrationMarkup(currentAccount.selectedCelebration);
  } else {
    result = 'YOU LOSE'; resultClass = 'lose'; beep(140, 0.25, 'sawtooth');
    const coinsLoss = NH_LOSS_COINS[nh.meta.diffKey] ?? LOSS_COINS;
    await addLoss(10, coinsLoss, RANK_POINTS.lose);
    xpNote = ` <span style="font-size:11px; color:var(--dim);">(-10 XP &middot; -${coinsLoss} &#129689; &middot; -${RANK_POINTS.lose} PTS)</span>`;
  }

  render(`
    ${celeb ? celeb.overlay : ''}
    <h2 class="title">ROOM ${nh.code} &mdash; RESULT</h2>
    <div class="versus">
      <div>
        <div class="choice" style="font-size:22px;">${mySolved ? myAttempts : '&mdash;'}</div>
        <div class="who">YOU ${mySolved ? 'GUESSES' : '(FAILED)'}</div>
      </div>
      <div class="vs">VS</div>
      <div>
        <div class="choice" style="font-size:22px;">${oppSolved ? oppAttempts : '&mdash;'}</div>
        <div class="who">OPP ${oppSolved ? 'GUESSES' : '(FAILED)'}</div>
      </div>
    </div>
    <div class="feedback ${resultClass}" style="text-align:center; font-size:16px;">
      ${result}${currentAccount ? xpNote : ''}
    </div>
    ${celeb ? celeb.title : ''}
    ${levelUpBanner(leveledUp)}
    ${reactionBarHtml('nh', nh.code, nh.role === 'p1' ? 'p2' : 'p1')}
    <div class="menu-row" style="margin-top:16px;">
      <button class="primary" onclick="beep(520); showNHOnlineMenu()">NEW RACE</button>
      <a class="back-link" style="text-align:center;" onclick="beep(300); stopNHPolling(); showNumberHuntHub()">&#8592; MAIN MENU</a>
    </div>
  `, celeb ? celeb.screenExtraClass : '');
}

showAccountSwitcher();
</script>

</body>
</html>
